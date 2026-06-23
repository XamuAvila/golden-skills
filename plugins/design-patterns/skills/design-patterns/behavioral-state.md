# State

**Category:** Behavioral

## Intent

Lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.

## Problem

A Document goes through Draft → Moderation → Published. Transition rules differ (only admin can publish from Draft). Managing states with if/switch becomes unmaintainable as states grow.

## Solution

Create State classes for every possible state. The original object (context) stores a reference to the current state object and delegates all state-specific work to it. To transition, replace the context's state object.

## Structure

1. **Context** — stores a reference to a ConcreteState object. Delegates state-specific behavior to it. Exposes a setter for changing state
2. **State** — interface declaring state-specific methods
3. **ConcreteStates** — implement state-specific behavior. Can reference context to get info or trigger transitions

## Pseudocode

```
class AudioPlayer is
    field state: State
    field volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

    method changeState(state: State) is
        this.state = state

    method clickLock() is     state.clickLock()
    method clickPlay() is     state.clickPlay()
    method clickNext() is     state.clickNext()
    method clickPrevious() is state.clickPrevious()

    method startPlayback() is // ...
    method stopPlayback() is  // ...
    method nextSong() is      // ...
    method previousSong() is  // ...
    method fastForward(time) is // ...
    method rewind(time) is    // ...

abstract class State is
    field player: AudioPlayer
    constructor State(player) is
        this.player = player

class LockedState extends State is
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))
    method clickPlay() is      // do nothing — locked
    method clickNext() is      // do nothing — locked
    method clickPrevious() is  // do nothing — locked

class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))
    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))
    method clickNext() is      player.nextSong()
    method clickPrevious() is  player.previousSong()

class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))
    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))
    method clickNext() is
        if (event.doubleclick)  player.nextSong()
        else                    player.fastForward(5)
    method clickPrevious() is
        if (event.doubleclick)  player.previousSong()
        else                    player.rewind(5)
```

## Applicability

- Use when an object behaves differently depending on its current state, the number of states is significant, and state-specific code changes frequently
- Use when a class is polluted with massive conditionals that alter behavior based on field values
- Use when you have a lot of duplicate code across similar states and conditional-based transitions

## Pros and Cons

**Pros:**
- Single Responsibility Principle — organize state-specific code into separate classes
- Open/Closed Principle — add new states without changing existing state classes or context
- Simplify code by eliminating bulky state machine conditionals

**Cons:**
- Overkill if a state machine has only a few states or rarely changes

## Relations with Other Patterns

- **Bridge**, State, **Strategy** (and **Adapter** partly) have very similar structures — all based on composition (delegating work to other objects)
- State can be considered an extension of **Strategy**. Both replace conditionals with object delegation. Difference: in State, concrete states may know about each other and trigger transitions; in Strategy, objects are independent
- State objects may be **Singletons** (if they contain no instance-specific data)

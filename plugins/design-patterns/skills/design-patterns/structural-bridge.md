# Bridge

**Category:** Structural

## Intent

Lets you split a large class or a set of closely related classes into two separate hierarchies — abstraction and implementation — which can be developed independently.

## Problem

Shape hierarchy with Color variants creates a class explosion: RedCircle, BlueCircle, RedSquare, BlueSquare. Adding a new shape or color multiplies the total number of classes.

## Solution

Switch from inheritance to composition. Extract one dimension (Color) into its own hierarchy. Shape holds a reference to a Color object and delegates color-related work.

## Structure

1. **Abstraction** — high-level control layer. Delegates actual work to the implementation object
2. **Implementation** — interface for all implementation classes
3. **ConcreteImplementation** — platform-specific code
4. **RefinedAbstraction** — provides variants of control logic
5. **Client** — links the abstraction with one of the implementations

## Pseudocode

```
class RemoteControl is
    field device: Device

    constructor RemoteControl(device: Device) is
        this.device = device

    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()

    method volumeDown() is
        device.setVolume(device.getVolume() - 10)

    method volumeUp() is
        device.setVolume(device.getVolume() + 10)

    method channelDown() is
        device.setChannel(device.getChannel() - 1)

    method channelUp() is
        device.setChannel(device.getChannel() + 1)

class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)

interface Device is
    method isEnabled(): bool
    method enable()
    method disable()
    method getVolume(): int
    method setVolume(percent)
    method getChannel(): int
    method setChannel(channel)

class Tv implements Device is
    // TV-specific implementation...

class Radio implements Device is
    // Radio-specific implementation...

// Client code
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)
```

## Applicability

- Use when you want to divide and organize a monolithic class that has several variants of some functionality (e.g., multiple DB backends)
- Use when you need to extend a class in several orthogonal (independent) dimensions
- Use when you need to switch implementations at runtime

## Pros and Cons

**Pros:**
- Create platform-independent classes and apps
- Client code works with high-level abstractions, not platform details
- Open/Closed Principle — add new abstractions and implementations independently
- Single Responsibility Principle

**Cons:**
- Can overcomplicate code by applying to a highly cohesive class

## Relations with Other Patterns

- **Bridge** is designed up-front (let abstraction and implementation vary independently); **Adapter** is applied after the fact to make incompatible interfaces work
- **Abstract Factory** can work alongside Bridge — useful when Bridge-defined abstractions only work with specific implementations
- Can combine **Builder** with Bridge: director = abstraction, builders = implementations
- **State**, **Strategy**, and Bridge have very similar structures — all based on composition

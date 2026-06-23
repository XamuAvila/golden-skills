# Iterator

**Also known as:** Cursor
**Category:** Behavioral

## Intent

Lets you traverse elements of a collection without exposing its underlying representation (list, stack, tree, graph, etc.).

## Problem

Collections can store elements in lists, trees, graphs, or other structures. Traversal logic mixed into the collection class bloats its primary responsibility. Client code becomes coupled to specific data structures.

## Solution

Extract the traversal behavior into a separate Iterator object. The iterator encapsulates traversal details (current position, remaining elements). Multiple iterators can traverse the same collection independently.

## Structure

1. **Iterator** — interface declaring traversal operations (getNext, hasMore)
2. **ConcreteIterator** — implements a specific traversal algorithm for a collection
3. **IterableCollection** — interface declaring a method to get an iterator
4. **ConcreteCollection** — returns new iterator instances tied to a particular collection instance

## Pseudocode

```
interface SocialNetwork is
    method createFriendsIterator(profileId): ProfileIterator
    method createCoworkersIterator(profileId): ProfileIterator

class Facebook implements SocialNetwork is
    method createFriendsIterator(profileId): ProfileIterator is
        return new FacebookIterator(this, profileId, "friends")
    method createCoworkersIterator(profileId): ProfileIterator is
        return new FacebookIterator(this, profileId, "coworkers")

    method socialGraphRequest(profileId, type): Profile[] is
        // Send network request to Facebook API

interface ProfileIterator is
    method getNext(): Profile
    method hasMore(): bool

class FacebookIterator implements ProfileIterator is
    field facebook: Facebook
    field profileId, type: string
    field currentPosition: int
    field cache: Profile[]

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    method getNext(): Profile is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore(): bool is
        lazyInit()
        return currentPosition < cache.length

// Client code — doesn't know concrete collection or iterator types
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore()) do
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)

class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, "Hey friend!")

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, "Dear colleague!")
```

## Applicability

- Use when your collection has a complex data structure and you want to hide its complexity from clients
- Use to reduce duplication of traversal code across your app
- Use when you want your code to be able to traverse different data structures, or when the structures are unknown beforehand

## Pros and Cons

**Pros:**
- Single Responsibility Principle — clean up collection by extracting traversal
- Open/Closed Principle — new types of collections and iterators without breaking anything
- Iterate over same collection in parallel (each iterator has its own state)
- Delay an iteration and continue when needed

**Cons:**
- Overkill if your app only works with simple collections
- May be less efficient than going through elements directly for some specialized collections

## Relations with Other Patterns

- Can use to traverse **Composite** trees
- **Factory Method** can be used with Iterator — collection subclasses return different iterator types
- **Memento** + Iterator to capture iteration state and roll back if needed
- **Visitor** + Iterator to traverse a complex structure and apply visitor to each element

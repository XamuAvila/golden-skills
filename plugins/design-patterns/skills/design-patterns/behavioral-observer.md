# Observer

**Also known as:** Event-Subscriber, Listener
**Category:** Behavioral

## Intent

Lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they're observing.

## Problem

Customer visits store daily to check if a product is available — wasted trips. Store could spam every customer about every new product — annoying. Need a targeted notification system.

## Solution

The publisher (subject) maintains a list of subscribers. When something changes, the publisher iterates over subscribers and calls their update method. Subscribers can register and unregister at runtime.

## Structure

1. **Publisher** — issues events. Contains subscription infrastructure (add/remove subscriber, notify)
2. **Subscriber** — interface declaring the update/notification method
3. **ConcreteSubscribers** — perform actions in response to publisher notifications
4. **Client** — creates publishers and subscribers, registers subscribers with publishers

## Pseudocode

```
class EventManager is
    field listeners: hash map of eventType → list of EventListener

    method subscribe(eventType, listener) is
        listeners.get(eventType).add(listener)

    method unsubscribe(eventType, listener) is
        listeners.get(eventType).remove(listener)

    method notify(eventType, data) is
        foreach listener in listeners.get(eventType) do
            listener.update(data)

class Editor is
    field events: EventManager
    field file: File

    constructor Editor() is
        events = new EventManager()

    method openFile(path) is
        this.file = new File(path)
        events.notify("open", file.name)

    method saveFile() is
        file.write()
        events.notify("save", file.name)

interface EventListener is
    method update(filename)

class LoggingListener implements EventListener is
    field log: File

    constructor LoggingListener(logFilename) is
        this.log = new File(logFilename)

    method update(filename) is
        log.write("Someone has opened: " + filename)

class EmailAlertsListener implements EventListener is
    field email: string

    constructor EmailAlertsListener(email) is
        this.email = email

    method update(filename) is
        system.email(email, "Someone has changed: " + filename)

// Client code
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener("/path/to/log.txt")
        editor.events.subscribe("open", logger)

        emailAlerts = new EmailAlertsListener("admin@example.com")
        editor.events.subscribe("save", emailAlerts)
```

## Applicability

- Use when changes to the state of one object may require changing other objects, and the actual set of objects is unknown beforehand or changes dynamically
- Use when some objects in your app must observe others, but only for a limited time or in specific cases

## Pros and Cons

**Pros:**
- Open/Closed Principle — introduce new subscriber classes without changing the publisher
- Establish relations between objects at runtime

**Cons:**
- Subscribers are notified in random order

## Relations with Other Patterns

- **Chain of Responsibility**, **Command**, **Mediator**, and Observer all address different ways of connecting senders and receivers
- Difference between Observer and **Mediator**: Observer = distributed communication; Mediator = centralized communication
- Difference between Observer and Pub/Sub: Observer is often synchronous and direct; Pub/Sub typically involves a message queue middleware

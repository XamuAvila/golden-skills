---
name: design-patterns
description: Use when choosing, understanding, or applying Gang of Four (GoF) design patterns in any language. Triggers on pattern selection questions, code smell symptoms (switch explosions, tight coupling, god classes, rigid hierarchies), OOP design decisions, SOLID violations, or when asked about creational/structural/behavioral patterns. Also use when refactoring to patterns, comparing patterns, or learning design principles. Language-agnostic with pseudocode — complements language-specific skills (design-patterns-python, design-patterns-typescript). Source: "Dive Into Design Patterns" by Alexander Shvets (Refactoring.Guru).
---

# Design Patterns — Language-Agnostic Reference

Complete reference for the 22 Gang of Four (GoF) design patterns with pseudocode examples, plus OOP fundamentals, design principles, and SOLID principles. Based on "Dive Into Design Patterns" by Alexander Shvets (Refactoring.Guru).

## When to use this skill

- Choosing which pattern fits a design problem
- Understanding a pattern's intent, structure, and trade-offs
- Reviewing code for pattern applicability (code smells → pattern mapping)
- Learning or teaching OOP design principles and SOLID
- Refactoring toward a pattern in any language
- Comparing patterns (Factory Method vs Abstract Factory, Strategy vs State, etc.)

## When NOT to use this skill

- For language-specific implementation details → use `design-patterns-python` or `design-patterns-typescript`
- When the language/framework already provides the pattern (Django signals = Observer, ASGI middleware = Chain of Responsibility)
- For one-off scripts where the pattern adds more indirection than it removes

## OOP Fundamentals

### Four Pillars

| Pillar | Definition |
|---|---|
| **Abstraction** | Model only relevant attributes/behaviors; hide the rest |
| **Encapsulation** | Hide internal state; expose behavior through methods |
| **Inheritance** | Build new classes on top of existing ones; reuse code |
| **Polymorphism** | Objects of different classes respond to the same interface |

### Object Relations

| Relation | Strength | Description |
|---|---|---|
| **Dependency** | Weakest | A uses B (parameter, local var, static call) |
| **Association** | Weak | A knows B (field reference) |
| **Aggregation** | Medium | A contains B, but B can exist independently |
| **Composition** | Strong | A owns B; B cannot exist without A |
| **Inheritance** | Strongest | A is-a B (extends/implements) |

## Design Principles

### Encapsulate What Varies

Identify the aspects that vary and separate them from what stays the same. Isolate change at method level (extract to separate method) or class level (delegate to a separate class).

### Program to an Interface, Not an Implementation

Depend on abstractions. Declare collaborators via interfaces/abstract types, not concrete classes. This enables swapping implementations without changing client code.

### Favor Composition Over Inheritance

Inheritance: "is-a" → static, single-parent, exposes parent internals.
Composition: "has-a" → flexible at runtime, delegates to contained objects.

Use composition when behavior must vary at runtime or when inheritance creates rigid, deep hierarchies.

## SOLID Principles

| Principle | Rule | Violation symptom |
|---|---|---|
| **S** — Single Responsibility | A class should have only one reason to change | Class does printing, DB access, and validation |
| **O** — Open/Closed | Open for extension, closed for modification | Adding a feature requires editing existing class |
| **L** — Liskov Substitution | Subclass must be substitutable for its parent | Overridden method throws unexpected exception or changes contract |
| **I** — Interface Segregation | No client should depend on methods it doesn't use | Interface with 20 methods, implementors leave most empty |
| **D** — Dependency Inversion | High-level modules depend on abstractions, not low-level modules | Business logic directly instantiates database driver |

## Pattern Catalog

### Creational Patterns

Provide object creation mechanisms that increase flexibility and reuse.

| Pattern | Intent | File |
|---|---|---|
| Factory Method | Provide an interface for creating objects in a superclass, letting subclasses alter the type created | [creational-factory-method.md](creational-factory-method.md) |
| Abstract Factory | Produce families of related objects without specifying concrete classes | [creational-abstract-factory.md](creational-abstract-factory.md) |
| Builder | Construct complex objects step by step; same construction process, different representations | [creational-builder.md](creational-builder.md) |
| Prototype | Copy existing objects without coupling to their concrete classes | [creational-prototype.md](creational-prototype.md) |
| Singleton | Ensure a class has only one instance with a global access point | [creational-singleton.md](creational-singleton.md) |

### Structural Patterns

Explain how to assemble objects and classes into larger structures while keeping them flexible and efficient.

| Pattern | Intent | File |
|---|---|---|
| Adapter | Convert an incompatible interface into one the client expects | [structural-adapter.md](structural-adapter.md) |
| Bridge | Split an abstraction from its implementation so both vary independently | [structural-bridge.md](structural-bridge.md) |
| Composite | Compose objects into tree structures; treat individual objects and compositions uniformly | [structural-composite.md](structural-composite.md) |
| Decorator | Attach new behaviors to objects by wrapping them, without modifying original class | [structural-decorator.md](structural-decorator.md) |
| Facade | Provide a simplified interface to a complex subsystem | [structural-facade.md](structural-facade.md) |
| Flyweight | Share common state across many objects to reduce memory usage | [structural-flyweight.md](structural-flyweight.md) |
| Proxy | Provide a substitute/placeholder for another object to control access | [structural-proxy.md](structural-proxy.md) |

### Behavioral Patterns

Deal with algorithms and the assignment of responsibilities between objects.

| Pattern | Intent | File |
|---|---|---|
| Chain of Responsibility | Pass a request along a chain of handlers until one processes it | [behavioral-chain-of-responsibility.md](behavioral-chain-of-responsibility.md) |
| Command | Turn a request into a standalone object containing all request info | [behavioral-command.md](behavioral-command.md) |
| Iterator | Traverse a collection without exposing its internal structure | [behavioral-iterator.md](behavioral-iterator.md) |
| Mediator | Reduce chaotic dependencies by routing communication through a mediator object | [behavioral-mediator.md](behavioral-mediator.md) |
| Memento | Save and restore an object's previous state without revealing implementation details | [behavioral-memento.md](behavioral-memento.md) |
| Observer | Define a subscription mechanism to notify multiple objects about state changes | [behavioral-observer.md](behavioral-observer.md) |
| State | Let an object alter its behavior when its internal state changes (appears to change class) | [behavioral-state.md](behavioral-state.md) |
| Strategy | Define a family of algorithms, encapsulate each one, and make them interchangeable | [behavioral-strategy.md](behavioral-strategy.md) |
| Template Method | Define the skeleton of an algorithm in a base class, letting subclasses override specific steps | [behavioral-template-method.md](behavioral-template-method.md) |
| Visitor | Add new operations to a class hierarchy without modifying existing classes | [behavioral-visitor.md](behavioral-visitor.md) |

## Decision Tree — Symptom → Pattern

| Symptom in current code | Candidate patterns |
|---|---|
| `if type == A ... else if type == B ...` dispatch explosion | Strategy, State, Visitor, polymorphism |
| Constructor with 10+ parameters or half-built objects passed around | Builder |
| Concrete class names hard-coded in client code | Factory Method, Abstract Factory, DI |
| Adding a feature requires editing the same class for every variant | Strategy, Decorator, Visitor |
| Object changes behavior based on a `status`/`mode` field | State |
| Adding cross-cutting concerns (logging, caching, auth) | Decorator, Proxy |
| Two class trees with parallel structure (UI vs PDF, MySQL vs Postgres) | Bridge |
| Tree of leaves + groups where operations on groups aggregate leaves | Composite |
| Two libraries with incompatible APIs | Adapter |
| One module exposes 30 classes but clients use only 4 calls | Facade |
| Same data object duplicated millions of times in memory | Flyweight |
| "Undo/redo" or "save snapshot and restore" | Memento, Command |
| N components react to one emitter's state change | Observer |
| 8 components each know the other 7 (N² coupling) | Mediator |
| Algorithm has 5 fixed steps, 2 vary by subclass | Template Method |
| Need a new operation across a stable hierarchy every week | Visitor |
| Request passes through filters until one handles it | Chain of Responsibility |
| Action must be queued, logged, undone, or serialized | Command |
| Custom iteration over non-list structure | Iterator |

## Pattern Evolution

Designs often start simple and evolve toward more complex patterns:

```
Factory Method → Abstract Factory → Prototype → Builder
(less complicated)                          (more flexible)
```

- Many designs start with **Factory Method** and evolve toward **Abstract Factory**, **Prototype**, or **Builder**
- **Abstract Factory** is often based on a set of **Factory Methods**, but can also use **Prototype**
- **Builder** focuses on constructing complex objects step by step; **Abstract Factory** on families of related products
- **Strategy** and **State** look similar but differ in intent: Strategy = interchangeable algorithms; State = behavior changes with internal state
- **Decorator** and **Proxy** have similar structures but different intents: Decorator = add behavior; Proxy = control access

## How to use this skill

1. Identify the **symptom** in the Decision Tree above
2. Open the per-pattern file for full details (intent, structure, pseudocode, pros/cons, relations)
3. For language-specific implementation, cross-reference with `design-patterns-python` or `design-patterns-typescript`
4. Adapt the pattern to your domain — never copy generic names verbatim

## Source

"Dive Into Design Patterns" v2021-2.28 by Alexander Shvets, Refactoring.Guru. All content adapted from [refactoring.guru/design-patterns](https://refactoring.guru/design-patterns).

# Template Method

**Category:** Behavioral

## Intent

Defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps without changing its structure.

## Problem

A data mining app processes DOC, CSV, and PDF files. Each has similar steps (open, extract, parse, analyze, report) but different details. Client code is full of conditionals to handle each format.

## Solution

Break the algorithm into steps. Define the template method in the base class that calls steps in order. Some steps are abstract (must be overridden), some have default implementations (optional override). Subclasses override specific steps without changing the overall structure.

## Structure

1. **AbstractClass** — declares the template method and step methods. Template method calls steps in a defined order. Steps can be abstract or have defaults
2. **ConcreteClass** — overrides specific steps, but NOT the template method itself

## Pseudocode

```
class GameAI is
    // Template method — defines the skeleton
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    method collectResources() is
        foreach s in this.builtStructures do
            s.collect()

    abstract method buildStructures()
    abstract method buildUnits()

    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    method sendScouts(position) is // ...
    method sendWarriors(position) is // ...

class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Build farms, barracks, stronghold

    method buildUnits() is
        if (there are plenty of resources) then
            if (no scouts)
                // Build peon, add to scouts
            else
                // Build grunt, add to warriors

    method attack() is
        if (warriors.length > 5)
            sendRogues(enemy.position)
            sendWarriors(enemy.position)
        else
            super.attack()

class MonstersAI extends GameAI is
    method collectResources() is  // do nothing
    method buildStructures() is   // do nothing
    method buildUnits() is        // do nothing
```

## Applicability

- Use when you want to let clients extend only particular steps of an algorithm, not the whole algorithm or its structure
- Use when you have several classes with nearly identical algorithms — differences are only in some steps

## Pros and Cons

**Pros:**
- Let clients override only certain parts of a large algorithm, reducing impact of changes
- Pull duplicate code into the superclass

**Cons:**
- Some clients may be limited by the provided skeleton
- May violate Liskov Substitution Principle if a subclass suppresses a default step
- Template methods tend to be harder to maintain the more steps they have

## Relations with Other Patterns

- **Factory Method** is a specialization of Template Method — a Factory Method can serve as a step in a Template Method
- Template Method uses inheritance and is class-level (static); **Strategy** uses composition and is object-level (dynamic, switchable at runtime)

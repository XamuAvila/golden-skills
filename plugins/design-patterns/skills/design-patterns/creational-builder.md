# Builder

**Category:** Creational

## Intent

Lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.

## Problem

A complex object (House) requires laborious step-by-step initialization of many fields and nested objects. Constructor with 10+ parameters is ugly. Creating subclasses for every house variant leads to a hierarchy explosion.

## Solution

Extract object construction into separate Builder classes. Organize construction into a series of steps (buildWalls, buildDoor, buildRoof). Call only the steps needed for a particular configuration. An optional Director class defines common construction sequences.

## Structure

1. **Builder** — interface declaring construction steps common to all types of builders
2. **ConcreteBuilder** — implements construction steps. Provides method to retrieve the result
3. **Product** — the resulting object. Products from different builders don't have to share a common interface
4. **Director** — defines the order of construction steps. Optional — client can call builder steps directly
5. **Client** — associates a builder with a director, initiates construction, retrieves result from builder

## Pseudocode

```
class Car is
    // A car can have GPS, trip computer, various seats, etc.

class Manual is
    // Each car has a corresponding user manual

interface Builder is
    method reset()
    method setSeats(number)
    method setEngine(engine: Engine)
    method setTripComputer()
    method setGPS()

class CarBuilder implements Builder is
    field car: Car

    method reset() is
        this.car = new Car()
    method setSeats(number) is  // ...
    method setEngine(engine) is  // ...
    method setTripComputer() is  // ...
    method setGPS() is  // ...
    method getResult(): Car is
        return this.car

class CarManualBuilder implements Builder is
    field manual: Manual

    method reset() is
        this.manual = new Manual()
    method setSeats(number) is
        // Document seat features
    method setEngine(engine) is
        // Add engine instructions
    method setTripComputer() is
        // Add trip computer instructions
    method setGPS() is
        // Add GPS instructions
    method getResult(): Manual is
        return this.manual

class Director is
    method makeSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer()
        builder.setGPS()

    method makeSUV(builder: Builder) is
        builder.reset()
        builder.setSeats(4)
        builder.setEngine(new SUVEngine())
        builder.setGPS()

class Application is
    method makeCar() is
        director = new Director()

        carBuilder = new CarBuilder()
        director.makeSportsCar(carBuilder)
        car = carBuilder.getResult()

        manualBuilder = new CarManualBuilder()
        director.makeSportsCar(manualBuilder)
        manual = manualBuilder.getResult()
```

## Applicability

- Use to get rid of a "telescoping constructor" (constructor with many optional parameters)
- Use when you want the same construction process to create different representations
- Use to construct Composite trees or other complex objects step by step

## How to Implement

1. Define common construction steps for building all available product representations
2. Declare these steps in the base Builder interface
3. Create a concrete builder class for each product representation
4. Consider creating a Director class to encapsulate common construction routines
5. Client creates both builder and director. Director uses builder to construct product. Client retrieves result from builder

## Pros and Cons

**Pros:**
- Construct objects step by step, defer steps, or run steps recursively
- Reuse same construction code for various product representations
- Single Responsibility Principle — isolate complex construction from business logic

**Cons:**
- Code complexity increases since the pattern requires creating multiple new classes

## Relations with Other Patterns

- Many designs start with **Factory Method** and evolve toward **Builder** (more control over construction)
- **Builder** constructs step by step; **Abstract Factory** returns product immediately
- Can use to construct **Composite** trees
- Can combine with **Bridge**: director class = abstraction, builders = implementations

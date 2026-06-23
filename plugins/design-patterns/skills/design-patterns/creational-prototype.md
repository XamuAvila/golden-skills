# Prototype

**Also known as:** Clone
**Category:** Creational

## Intent

Lets you copy existing objects without making your code dependent on their classes.

## Problem

You need an exact copy of an object. Creating via constructor requires knowing the concrete class (coupling). Some fields may be private and not accessible from outside the object.

## Solution

Delegate the cloning process to the actual objects being cloned. Declare a common `clone()` method in a shared interface. Each class implements clone() to copy its own fields, including private ones. The object that supports cloning is called a prototype.

## Structure

1. **Prototype** — interface declaring the `clone()` method
2. **ConcretePrototype** — implements the clone method. Handles edge cases like cloning linked objects, untangling recursive dependencies
3. **Client** — can produce a copy of any object that follows the prototype interface
4. **PrototypeRegistry** — (optional) stores frequently-used prototypes for easy access by name/parameters

## Pseudocode

```
abstract class Shape is
    field x: int
    field y: int
    field color: string

    constructor Shape(source: Shape) is
        this.x = source.x
        this.y = source.y
        this.color = source.color

    abstract method clone(): Shape

class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        super(source)
        this.width = source.width
        this.height = source.height

    method clone(): Shape is
        return new Rectangle(this)

class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone(): Shape is
        return new Circle(this)

// Client code
class Application is
    field shapes: Shape[]

    method init() is
        circle = new Circle()
        circle.x = 10
        circle.y = 10
        circle.radius = 20
        shapes.add(circle)

        anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // anotherCircle is an independent copy

        rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        shapesCopy = new Shape[0]
        foreach s in shapes do
            shapesCopy.add(s.clone())
        // shapesCopy contains independent copies
```

## Applicability

- Use when code shouldn't depend on the concrete classes of objects it needs to copy
- Use to reduce the number of subclasses that only differ in how they initialize their respective objects (a set of pre-configured prototypes can substitute a subclass hierarchy)

## How to Implement

1. Create the prototype interface and declare the `clone` method in it
2. In each class, define a "copy constructor" that accepts an object of the same class. Copy all fields
3. The clone method creates a new object using the copy constructor, passing `this`
4. Optionally, create a centralized prototype registry to store a catalog of frequently used prototypes

## Pros and Cons

**Pros:**
- Clone objects without coupling to their concrete classes
- Remove repeated initialization code — use pre-built prototypes as presets
- Produce complex objects more conveniently
- Alternative to inheritance when dealing with configuration presets

**Cons:**
- Cloning complex objects that have circular references can be tricky

## Relations with Other Patterns

- **Abstract Factory** can use **Prototype** to compose its create methods
- **Prototype** doesn't rely on inheritance (unlike **Factory Method**), but requires a complex `clone` initialization
- Designs that use **Composite** and **Decorator** heavily can benefit from **Prototype** (clone complex trees instead of rebuilding)
- Can help when you need to store copies of **Commands** in history
- **Memento** can sometimes be a simpler alternative when the object's state snapshot is straightforward

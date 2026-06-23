# Visitor

**Category:** Behavioral

## Intent

Lets you define a new operation without changing the classes of the elements on which it operates.

## Problem

A geographic information system has nodes (City, Industry, SightseeingArea). You need to export them to XML. Adding export logic to each node risks breaking existing code. The node classes might not even be yours to modify.

## Solution

Place the new behavior into a separate Visitor class. The original object "accepts" a visitor and calls the visitor's method corresponding to its own type (double dispatch). The visitor has a separate method for each element type.

## Structure

1. **Visitor** — interface with a visit method for each concrete element class
2. **ConcreteVisitor** — implements the same behavior for all concrete element classes. Serves as a template for when the same behavior is adapted for different classes
3. **Element** — interface with `accept(visitor)` method
4. **ConcreteElement** — implements `accept()` by calling the visitor method that matches its own class
5. **Client** — usually represents a collection or complex structure (e.g., Composite tree)

## Pseudocode

```
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

class Dot implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)

class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Export dot's id and center coordinates to XML

    method visitCircle(c: Circle) is
        // Export circle's id, center, radius to XML

    method visitRectangle(r: Rectangle) is
        // Export rectangle's id, coordinates, width, height to XML

    method visitCompoundShape(cs: CompoundShape) is
        // Export shape's id
        // Recursively export each child to XML

// Client code
class Application is
    field allShapes: Shape[]

    method export() is
        exportVisitor = new XMLExportVisitor()
        foreach shape in allShapes do
            shape.accept(exportVisitor)
```

## Applicability

- Use when you need to perform an operation on all elements of a complex object structure (e.g., a Composite tree)
- Use to clean up business logic of auxiliary behaviors
- Use when a behavior makes sense only in some classes of a class hierarchy, not in others — put it in a Visitor and only implement the relevant visit methods

## Pros and Cons

**Pros:**
- Open/Closed Principle — add new behavior without changing existing classes
- Single Responsibility Principle — move multiple versions of the same behavior into one class
- A visitor can accumulate useful information while working with various objects (e.g., totals, aggregations)

**Cons:**
- Must update all visitors whenever a class is added or removed from the element hierarchy
- Visitors might lack access to private fields and methods of the elements they work with

## Relations with Other Patterns

- Visitor can be treated as a more powerful version of **Command** — can execute operations on objects of different classes
- Can use to execute an operation over an entire **Composite** tree
- Can use alongside **Iterator** to traverse a complex data structure and apply visitor operations to each element

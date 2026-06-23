# Composite

**Also known as:** Object Tree
**Category:** Structural

## Intent

Lets you compose objects into tree structures and then work with these structures as if they were individual objects.

## Problem

An ordering system has Products and Boxes. A Box can contain Products and smaller Boxes. To calculate the total price, you'd have to check the type of each element, which creates coupled, fragile code.

## Solution

Work with Products and Boxes through a common interface that declares a `getPrice()` method. A Product returns its price. A Box iterates over its children, sums their prices, and can add packaging cost.

## Structure

1. **Component** — interface describing operations common to simple and complex elements
2. **Leaf** — basic element with no sub-elements. Does the actual work
3. **Composite** — element with sub-elements (children). Delegates work to children, aggregates results
4. **Client** — works with all elements through the Component interface

## Pseudocode

```
interface Graphic is
    method move(x, y)
    method draw()

class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) is // ...

    method move(x, y) is
        this.x += x
        this.y += y

    method draw() is
        // Draw a dot at X and Y

class Circle extends Dot is
    field radius

    method draw() is
        // Draw a circle at X, Y with radius

class CompoundGraphic implements Graphic is
    field children: Graphic[]

    method add(child: Graphic) is
        children.add(child)

    method remove(child: Graphic) is
        children.remove(child)

    method move(x, y) is
        foreach child in children do
            child.move(x, y)

    method draw() is
        // 1. Calculate bounding rectangle of all children
        // 2. For each child, call child.draw()
        foreach child in children do
            child.draw()

// Client code
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))

    method groupSelected(components: Graphic[]) is
        group = new CompoundGraphic()
        foreach component in components do
            group.add(component)
            all.remove(component)
        all.add(group)
        all.draw()
```

## Applicability

- Use when you have to implement a tree-like object structure
- Use when you want client code to treat both simple and complex elements uniformly

## Pros and Cons

**Pros:**
- Work with complex tree structures more conveniently — use polymorphism and recursion
- Open/Closed Principle — add new element types without breaking existing code

**Cons:**
- Can be hard to provide a common interface for classes whose functionality differs too much (overgeneralization)

## Relations with Other Patterns

- **Builder** can construct Composite trees
- **Chain of Responsibility** is often used with Composite — leaf components' parents can be the chain's next handler
- Can traverse Composites using **Iterators**
- Use **Visitor** to execute operations over an entire Composite tree
- Can share leaf nodes as **Flyweights** to save RAM
- **Composite** and **Decorator** have similar structure diagrams (recursive composition), but Decorator adds responsibilities while Composite "sums" results
- Use **Prototype** to clone complex Composite trees instead of reconstructing them

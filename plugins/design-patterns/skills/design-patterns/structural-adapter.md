# Adapter

**Also known as:** Wrapper
**Category:** Structural

## Intent

Allows objects with incompatible interfaces to collaborate by wrapping one object with an adapter that converts its interface.

## Problem

A stock market monitoring app downloads data in XML format, but an analytics library only accepts JSON. You can't change the library code.

## Solution

Create an adapter that wraps the XML data source. The adapter converts XML to JSON on the fly, making the data source compatible with the analytics library.

## Structure

1. **Client** — contains existing business logic
2. **Client Interface** — protocol that other classes must follow to collaborate with client code
3. **Service (Adaptee)** — some useful class with an incompatible interface
4. **Adapter** — implements the client interface, wraps the service object, translates calls

## Pseudocode

```
class RoundHole is
    field radius: double

    constructor RoundHole(radius) is
        this.radius = radius

    method getRadius() is
        return radius

    method fits(peg: RoundPeg): bool is
        return this.getRadius() >= peg.getRadius()

class RoundPeg is
    field radius: double
    method getRadius() is
        return radius

class SquarePeg is
    field width: double
    method getWidth() is
        return width

// Adapter makes SquarePeg compatible with RoundHole
class SquarePegAdapter extends RoundPeg is
    field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // Calculate minimum circle that fits the square peg
        return peg.getWidth() * Math.sqrt(2) / 2

// Client code
hole = new RoundHole(5)
sqPeg = new SquarePeg(5)
// hole.fits(sqPeg)  // Won't compile — incompatible types
sqPegAdapter = new SquarePegAdapter(sqPeg)
hole.fits(sqPegAdapter)  // Works!
```

## Applicability

- Use when you want to use an existing class, but its interface isn't compatible with the rest of your code
- Use when you want to reuse several existing subclasses that lack some common functionality that can't be added to the superclass

## Pros and Cons

**Pros:**
- Single Responsibility Principle — separate interface conversion from business logic
- Open/Closed Principle — add new adapters without changing existing client code

**Cons:**
- Overall complexity increases. Sometimes it's simpler to just change the service class

## Relations with Other Patterns

- **Bridge** is designed up-front; **Adapter** is used with existing classes to make incompatible interfaces work
- **Adapter** changes the interface of an existing object; **Decorator** enhances an object without changing interface; **Proxy** provides the same interface
- **Facade** defines a new interface for an entire subsystem; **Adapter** wraps a single object to make its existing interface usable

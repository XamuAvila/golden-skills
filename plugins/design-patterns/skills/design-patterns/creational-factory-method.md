# Factory Method

**Also known as:** Virtual Constructor
**Category:** Creational

## Intent

Provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.

## Problem

A logistics app initially handles only truck transportation. All code lives in the `Truck` class. Adding `Ship` support would require changing the entire codebase, and every new transport type repeats the problem.

## Solution

Replace direct constructor calls (`new Truck()`) with calls to a special factory method. Subclasses override the factory method to return different product types. All products must implement a common interface.

## Structure

1. **Product** — interface common to all objects the creator and subclasses can produce
2. **ConcreteProduct** — different implementations of the product interface
3. **Creator** — declares the factory method (can be abstract or have a default). Primary responsibility is NOT creating products — it contains core business logic that relies on products
4. **ConcreteCreator** — overrides the factory method to return a different product type

## Pseudocode

```
class Dialog is
    abstract method createButton(): Button

    method render() is
        Button okButton = createButton()
        okButton.onClick(closeDialog)
        okButton.render()

class WindowsDialog extends Dialog is
    method createButton(): Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton(): Button is
        return new HTMLButton()

interface Button is
    method render()
    method onClick(f)

class WindowsButton implements Button is
    method render(a, b) is
        // Render a button in Windows style
    method onClick(f) is
        // Bind a native OS click event

class HTMLButton implements Button is
    method render(a, b) is
        // Return an HTML representation of a button
    method onClick(f) is
        // Bind a web browser click event

class Application is
    field dialog: Dialog

    method initialize() is
        config = readApplicationConfigFile()
        if (config.OS == "Windows") then
            dialog = new WindowsDialog()
        else if (config.OS == "Web") then
            dialog = new WebDialog()
        else
            throw new Exception("Error! Unknown OS.")

    method main() is
        this.initialize()
        dialog.render()
```

## Applicability

- Use when you don't know beforehand the exact types and dependencies of the objects your code should work with
- Use when you want to provide users of your library or framework with a way to extend its internal components
- Use when you want to save system resources by reusing existing objects instead of rebuilding them each time

## How to Implement

1. Make all products follow the same interface
2. Add an empty factory method inside the creator class. Return type should match the common product interface
3. Replace all product constructors with calls to the factory method
4. Create a set of creator subclasses for each product type. Override the factory method in subclasses
5. If too many product types, reuse a control parameter from the base class in subclasses
6. If the base factory method is empty after extractions, make it abstract

## Pros and Cons

**Pros:**
- Avoid tight coupling between creator and concrete products
- Single Responsibility Principle — move product creation into one place
- Open/Closed Principle — introduce new product types without breaking existing client code

**Cons:**
- Code may become more complicated since you need many new subclasses

## Relations with Other Patterns

- Many designs start with **Factory Method** (simpler) and evolve toward **Abstract Factory**, **Prototype**, or **Builder** (more flexible, more complex)
- **Abstract Factory** classes are often based on a set of **Factory Methods**, but you can also use **Prototype** to compose methods on these classes
- You can use **Factory Method** along with **Iterator** to let collection subclasses return different types of iterators
- **Prototype** isn't based on inheritance, so it doesn't have its drawbacks. But Prototype requires complicated initialization. **Factory Method** is based on inheritance but doesn't require initialization
- **Factory Method** is a specialization of **Template Method**. A Factory Method may serve as a step in a large Template Method

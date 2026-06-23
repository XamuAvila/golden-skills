# Abstract Factory

**Also known as:** Kit
**Category:** Creational

## Intent

Lets you produce families of related objects without specifying their concrete classes.

## Problem

A furniture shop simulator needs to create families of related products: Chair + Sofa + CoffeeTable, each in variants: Modern, Victorian, ArtDeco. Products within a family must match. New styles should be addable without changing existing code.

## Solution

Declare interfaces for each product in the family (Chair, Sofa, Table). Then declare the AbstractFactory with create methods for each product. Concrete factories (ModernFurnitureFactory, VictorianFurnitureFactory) return products that match the family.

## Structure

1. **AbstractProduct** — interface for each distinct product type (Chair, Sofa)
2. **ConcreteProduct** — variant-specific implementations (ModernChair, VictorianChair)
3. **AbstractFactory** — interface declaring create methods for each abstract product
4. **ConcreteFactory** — implements create methods returning products of a single variant
5. **Client** — works with factories and products only through abstract interfaces

## Pseudocode

```
interface GUIFactory is
    method createButton(): Button
    method createCheckbox(): Checkbox

class WinFactory implements GUIFactory is
    method createButton(): Button is
        return new WinButton()
    method createCheckbox(): Checkbox is
        return new WinCheckbox()

class MacFactory implements GUIFactory is
    method createButton(): Button is
        return new MacButton()
    method createCheckbox(): Checkbox is
        return new MacCheckbox()

interface Button is
    method paint()

class WinButton implements Button is
    method paint() is
        // Render button in Windows style

class MacButton implements Button is
    method paint() is
        // Render button in macOS style

interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Render checkbox in Windows style

class MacCheckbox implements Checkbox is
    method paint() is
        // Render checkbox in macOS style

class Application is
    field factory: GUIFactory
    field button: Button

    constructor Application(factory: GUIFactory) is
        this.factory = factory

    method createUI() is
        this.button = factory.createButton()

    method paint() is
        button.paint()

// Application picks the factory type based on configuration
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()
        if (config.OS == "Windows") then
            factory = new WinFactory()
        else if (config.OS == "Mac") then
            factory = new MacFactory()
        else
            throw new Exception("Error! Unknown OS.")
        app = new Application(factory)
```

## Applicability

- Use when code must work with various families of related products, but you don't want to depend on concrete classes
- Use when a class has a set of Factory Methods that blur its primary responsibility

## How to Implement

1. Map out a matrix of distinct product types versus variants
2. Declare abstract product interfaces for all product types
3. Declare abstract factory interface with create methods for all abstract products
4. Implement a set of concrete factory classes, one for each product variant
5. Create factory initialization code somewhere in the app (picks concrete factory based on config or environment)

## Pros and Cons

**Pros:**
- Products from the same factory are guaranteed compatible
- Avoid tight coupling between concrete products and client code
- Single Responsibility Principle
- Open/Closed Principle — new variants without breaking existing code

**Cons:**
- Code becomes more complicated with many new interfaces and classes

## Relations with Other Patterns

- Often based on a set of **Factory Methods**, but can also use **Prototype** to compose methods
- Can serve as an alternative to **Facade** when you only want to hide the way subsystem objects are created
- Can use with **Bridge** — useful when Bridge-defined abstractions can only work with specific implementations
- All factories (**Abstract Factory**, **Builder**, **Prototype**) can be implemented as **Singletons**

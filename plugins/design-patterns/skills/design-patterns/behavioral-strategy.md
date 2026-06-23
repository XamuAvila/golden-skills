# Strategy

**Also known as:** Policy
**Category:** Behavioral

## Intent

Lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

## Problem

A navigation app routes only by road initially. Adding walking, public transit, cycling, tourist routes bloats the main class. Each new algorithm risks breaking existing ones.

## Solution

Extract each algorithm into its own class (strategy) sharing a common interface. The context stores a reference to one of the strategies and delegates work to it. The client picks the strategy.

## Structure

1. **Context** — maintains a reference to one of the concrete strategies. Communicates with the strategy via the strategy interface only
2. **Strategy** — interface common to all concrete strategies
3. **ConcreteStrategies** — implement different variants of an algorithm
4. **Client** — creates a specific strategy object and passes it to the context

## Pseudocode

```
interface Strategy is
    method execute(a, b): int

class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

class Context is
    field strategy: Strategy

    method setStrategy(strategy: Strategy) is
        this.strategy = strategy

    method executeStrategy(a, b) is
        return strategy.execute(a, b)

// Client code
class Application is
    method main() is
        context = new Context()
        // Read user input...

        if (action == "addition") then
            context.setStrategy(new ConcreteStrategyAdd())
        if (action == "subtraction") then
            context.setStrategy(new ConcreteStrategySubtract())
        if (action == "multiplication") then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(firstNumber, secondNumber)
```

## Applicability

- Use when you want to use different variants of an algorithm within an object and be able to switch at runtime
- Use when you have many similar classes that only differ in the way they execute some behavior
- Use to isolate business logic from implementation details of algorithms
- Use when your class has a massive conditional that switches between variants of the same algorithm

## Pros and Cons

**Pros:**
- Swap algorithms at runtime
- Isolate the implementation details of an algorithm from the code that uses it
- Replace inheritance with composition
- Open/Closed Principle — add new strategies without changing context

**Cons:**
- If you only have a couple of algorithms that rarely change, there's no real reason to use the pattern
- Clients must be aware of the differences between strategies to pick the right one
- Many modern languages have functional type support (lambdas), letting you achieve the same result without extra classes

## Relations with Other Patterns

- **Bridge**, **State**, Strategy (and **Adapter**) have similar structures — all based on composition
- **Decorator** lets you change the "skin" of an object; Strategy lets you change the "guts"
- **Template Method** uses inheritance and is static; Strategy uses composition and is dynamic (switchable at runtime)
- **Command** and Strategy may look similar — both parameterize an object with an action. Difference: Strategy = different ways of doing the SAME thing; Command = any operation turned into an object
- **State** can be considered an extension of Strategy. In Strategy, objects are independent and unaware of each other; in State, concrete states can trigger transitions between each other

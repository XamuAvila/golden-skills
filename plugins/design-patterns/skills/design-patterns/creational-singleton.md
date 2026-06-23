# Singleton

**Category:** Creational

> **Warning:** Refactoring.guru considers Singleton close to an antipattern. Test pain and global state are the cost. Use with caution.

## Intent

Ensures that a class has only one instance, while providing a global access point to this instance.

## Problem

Two problems solved simultaneously (violates SRP): (1) Ensure a class has a single instance — e.g., shared database connection or config object. (2) Provide a global access point — like a global variable, but write-protected.

## Solution

Make the default constructor private. Create a static creation method that acts as a constructor. On first call, it creates an instance and caches it in a static field. All subsequent calls return the cached instance.

## Structure

1. **Singleton** — declares the static method `getInstance()` that returns the same instance. Constructor must be private. Only way to get the instance is `getInstance()`

## Pseudocode

```
class Database is
    field instance: Database

    private constructor Database() is
        // initialization, e.g., open DB connection

    static method getInstance(): Database is
        if (Database.instance == null) then
            acquireThreadLock() // thread safety
            if (Database.instance == null) then
                Database.instance = new Database()
        return Database.instance

    method query(sql) is
        // All queries go through this single instance

// Client code
class Application is
    method main() is
        foo = Database.getInstance()
        foo.query("SELECT ...")
        // ...
        bar = Database.getInstance()
        bar.query("SELECT ...")
        // foo and bar are the same object
```

## Applicability

- Use when a class in your program should have just a single instance available to all clients (e.g., a single database object shared by different parts of the program)
- Use when you need stricter control over global variables

## How to Implement

1. Add a private static field to the class for storing the singleton instance
2. Declare a public static creation method for getting the singleton instance
3. Implement lazy initialization inside the static method — create a new object on first call, store it in the static field, return it on all subsequent calls
4. Make the constructor of the class private
5. In client code, replace all direct constructor calls with calls to the static creation method

## Pros and Cons

**Pros:**
- Guarantees that a class has only a single instance
- Global access point to that instance
- Singleton object is initialized only when first requested (lazy initialization)

**Cons:**
- Violates the Single Responsibility Principle (solves two problems at once)
- Can mask bad design (components know too much about each other)
- Requires special treatment in multithreaded environments (locking, double-checked locking)
- Difficult to unit test — hard to mock, can't substitute. Tests may share state
- Encourages global mutable state

## Relations with Other Patterns

- A **Facade** class can often be transformed into a **Singleton** since a single facade is usually sufficient
- **Flyweight** would resemble Singleton if you managed to reduce all shared states to just one flyweight object — but there are two key differences: (1) there can be many Flyweight instances; (2) Flyweight objects are immutable, Singleton can be mutable
- **Abstract Factory**, **Builder**, and **Prototype** can all be implemented as Singletons

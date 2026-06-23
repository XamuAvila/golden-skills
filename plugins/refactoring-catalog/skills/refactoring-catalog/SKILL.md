---
name: refactoring-catalog
description: Use when encountering code smells, planning refactoring sessions, reviewing code for structural improvements, or deciding which refactoring technique to apply. Triggers on: long methods, duplicated code, large classes, feature envy, shotgun surgery, data clumps, primitive obsession, switch statements, message chains, speculative generality, or any request to improve code structure without changing external behavior.
---

# Refactoring Catalog

Based on Martin Fowler's "Refactoring: Improving the Design of Existing Code" (1st edition, 1999).

## Overview

**Refactoring (noun):** a change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior. **Refactoring (verb):** to restructure software by applying a series of such changes. The core principle is behavior-preserving structural improvement -- the program produces exactly the same output at every step.

## When to Use

**Triggers:**
- Adding a feature and the code is not structured to accommodate it
- Fixing a bug and the code is hard to understand
- Code review reveals structural problems
- Rule of Three: first time just do it, second time wince, third time refactor
- You feel the need to write a comment explaining what code does
- Copy-paste would be needed to add a variant (e.g., HTML version of a text report)

**Do NOT refactor when:**
- Code is so broken it should be rewritten from scratch (code must mostly work first)
- Very close to a hard deadline (but "no time" usually means you NEED to refactor)
- The design decision is so central that refactoring between alternatives would be very hard -- invest in upfront design instead

## The Refactoring Workflow

1. **Always have tests before refactoring.** Tests must be self-checking (pass/fail automatically). No tests = no refactoring.
2. **Small steps, test after each.** If you make a mistake, it is easy to find. Never take large leaps.
3. **Wear one hat at a time.** Either adding function OR refactoring, never both simultaneously.
4. **Backtrack, do not debug forward.** If a test fails after several refactorings, revert to last known good state and replay changes one by one.
5. **Do not worry about performance during refactoring.** Profile later. Well-factored code is easier to optimize because you have finer granularity and clearer boundaries.
6. **Stop when unsure.** If confidence drops or you cannot prove semantics are preserved, stop. Stopping is the strongest move.

## Code Smells

| # | Smell | Description | Suggested Refactorings |
|---|-------|-------------|----------------------|
| 1 | **Duplicated Code** | Same code structure in more than one place. Number one in the stink parade. | Extract Method, Pull Up Field, Form Template Method, Substitute Algorithm, Extract Class |
| 2 | **Long Method** | Too long, too much buried by complex logic. Almost all problems come from methods that are too long. | Extract Method, Replace Temp with Query, Replace Method with Method Object, Decompose Conditional |
| 3 | **Large Class** | Too many instance variables, too many methods. Doing work that should be done by two or more classes. | Extract Class, Extract Subclass, Extract Interface, Duplicate Observed Data |
| 4 | **Long Parameter List** | Hard to understand, inconsistent, difficult to use, forever changing. | Replace Parameter with Method, Preserve Whole Object, Introduce Parameter Object |
| 5 | **Divergent Change** | One class commonly changed in different ways for different reasons. | Extract Class |
| 6 | **Shotgun Surgery** | One change requires many little changes to many classes. Opposite of Divergent Change. | Move Method, Move Field, Inline Class |
| 7 | **Feature Envy** | A method more interested in another class's data than its own. | Move Method, Extract Method + Move Method |
| 8 | **Data Clumps** | Same three or four data items appearing together repeatedly in fields and parameters. | Extract Class, Introduce Parameter Object, Preserve Whole Object |
| 9 | **Primitive Obsession** | Using primitives instead of small objects for money, ranges, phone numbers, type codes. | Replace Data Value with Object, Replace Type Code with Class/Subclasses/State-Strategy, Extract Class, Introduce Parameter Object |
| 10 | **Switch Statements** | Switch on type code scattered in different places. Adding a new type means finding all switches. | Replace Type Code with Subclasses or State/Strategy, then Replace Conditional with Polymorphism |
| 11 | **Parallel Inheritance Hierarchies** | Every time you subclass one class, you must subclass another. Same prefixes in both hierarchies. | Move Method, Move Field (collapse one hierarchy) |
| 12 | **Lazy Class** | A class not doing enough to pay for itself. | Collapse Hierarchy, Inline Class |
| 13 | **Speculative Generality** | Hooks and special cases for things not required. "Oh, I think we need this someday." | Collapse Hierarchy, Inline Class, Remove Parameter, Rename Method |
| 14 | **Temporary Field** | Instance variable set only in certain circumstances. | Extract Class, Introduce Null Object |
| 15 | **Message Chains** | Client navigates through a chain of objects: a.getB().getC().getD(). | Hide Delegate, Extract Method + Move Method |
| 16 | **Middle Man** | Half the methods just delegate to another class. Encapsulation gone too far. | Remove Middle Man, Inline Method, Replace Delegation with Inheritance |
| 17 | **Inappropriate Intimacy** | Classes delving too much into each other's private parts. | Move Method, Move Field, Change Bidirectional to Unidirectional, Extract Class, Hide Delegate |
| 18 | **Alternative Classes with Different Interfaces** | Two classes do the same thing but have different method signatures. | Rename Method, Move Method, Extract Superclass |
| 19 | **Incomplete Library Class** | Library class needs methods you cannot add because you cannot modify it. | Introduce Foreign Method, Introduce Local Extension |
| 20 | **Data Class** | Classes with only fields and accessors, no behavior. Dumb data holders. | Encapsulate Field, Encapsulate Collection, Remove Setting Method, Move Method, Hide Method |
| 21 | **Refused Bequest** | Subclass inherits methods/data it does not want or need. | Push Down Method, Push Down Field, Replace Inheritance with Delegation |
| 22 | **Comments (as deodorant)** | Comments used to explain bad code instead of fixing it. Try refactoring first. | Extract Method, Rename Method, Introduce Assertion |

> **Note:** Magic numbers (literal values like `0.21`, `500`, `9.81`) are not a named smell in Chapter 3, but are a common sign of Primitive Obsession (#9). Apply **Replace Magic Number with Symbolic Constant** (see Organizing Data catalog).

## Catalog: Composing Methods

### Extract Method

**Motivation:** Turn a code fragment into a method named after its intention. The most common refactoring. Use whenever you feel the need to comment something.

**Mechanics:**
- Identify the fragment; check for local variables (read-only ones become parameters, one modified variable becomes the return value)
- Copy fragment into new method named after WHAT it does, not HOW
- Replace original fragment with a call to the new method
- Compile and test

```
// Before
function printOwing(amount) {
  printBanner()
  // print details
  print("name: " + name)
  print("amount: " + amount)
}

// After
function printOwing(amount) {
  printBanner()
  printDetails(amount)
}
function printDetails(amount) {
  print("name: " + name)
  print("amount: " + amount)
}
```

### Inline Method

**Motivation:** A method's body is just as clear as its name, or methods are badly factored and need reorganizing. Put the body back into the caller.

**Mechanics:**
- Check method is not polymorphic (not overridden by subclasses)
- Replace each call with the method body
- Remove the method definition

```
// Before
getRating() { return moreThanFiveLateDeliveries() ? 2 : 1 }
moreThanFiveLateDeliveries() { return numberOfLateDeliveries > 5 }

// After
getRating() { return (numberOfLateDeliveries > 5) ? 2 : 1 }
```

### Inline Temp

**Motivation:** A temp assigned once with a simple expression is getting in the way of other refactorings. Replace all references with the expression.

**Mechanics:**
- Verify the temp is assigned only once (declare as final/const to check)
- Replace all references with the right-hand side expression
- Remove the declaration and assignment

```
// Before
basePrice = anOrder.basePrice()
return (basePrice > 1000)

// After
return (anOrder.basePrice() > 1000)
```

### Replace Temp with Query

**Motivation:** Temps are local and encourage long methods. Replacing with a query method makes the value accessible to any method in the class and enables Extract Method.

**Mechanics:**
- Find a temp assigned once; extract the right-hand side into a query method
- Replace all references to the temp with calls to the query
- If the temp accumulates in a loop, copy the loop into the query method

```
// Before
function getPrice() {
  basePrice = quantity * itemPrice
  if (basePrice > 1000) discountFactor = 0.95
  else discountFactor = 0.98
  return basePrice * discountFactor
}

// After
function getPrice() { return basePrice() * discountFactor() }
function basePrice() { return quantity * itemPrice }
function discountFactor() { return (basePrice() > 1000) ? 0.95 : 0.98 }
```

### Introduce Explaining Variable

**Motivation:** Complex expressions are hard to read. Use a well-named temp to break the expression into manageable pieces. Prefer Extract Method when possible; use this when temps are too tangled.

**Mechanics:**
- Declare a final/const temp set to part of the complex expression
- Replace that part of the expression with the temp
- Repeat for other parts

```
// Before
if (platform.toUpper().indexOf("MAC") > -1 &&
    browser.toUpper().indexOf("IE") > -1 &&
    wasInitialized() && resize > 0) { ... }

// After
isMacOs = platform.toUpper().indexOf("MAC") > -1
isIE = browser.toUpper().indexOf("IE") > -1
wasResized = resize > 0
if (isMacOs && isIE && wasInitialized() && wasResized) { ... }
```

### Split Temporary Variable

**Motivation:** A temp assigned more than once (not a loop variable or collecting temp) has more than one responsibility. Each responsibility needs its own variable.

**Mechanics:**
- Rename the temp at its first assignment to reflect its first use
- Declare a new temp at the second assignment with a name reflecting its second use
- Repeat for subsequent assignments

```
// Before
temp = 2 * (height + width)
print(temp)
temp = height * width
print(temp)

// After
perimeter = 2 * (height + width)
print(perimeter)
area = height * width
print(area)
```

### Remove Assignments to Parameters

**Motivation:** Reassigning a parameter to a different object causes confusion about pass-by-value vs pass-by-reference. Parameters should only represent what was passed in.

**Mechanics:**
- Create a local temp for the parameter
- Replace all post-assignment references to the parameter with the temp
- Change the assignment to assign to the temp

```
// Before
function discount(inputVal, quantity, yearToDate) {
  if (inputVal > 50) inputVal -= 2
  if (quantity > 100) inputVal -= 1
  return inputVal
}

// After
function discount(inputVal, quantity, yearToDate) {
  result = inputVal
  if (inputVal > 50) result -= 2
  if (quantity > 100) result -= 1
  return result
}
```

### Replace Method with Method Object

**Motivation:** A long method with tangled temps that resist Extract Method. Turn the method into its own class where all temps become fields, then decompose freely.

**Mechanics:**
- Create a new class with a field for each temp, parameter, and the source object
- Add a constructor taking the source object and parameters
- Copy the method body into a `compute` method on the new class
- Replace original method body with instantiation + `compute()` call

```
// Before
class Account {
  gamma(inputVal, quantity, yearToDate) {
    val1 = (inputVal * quantity) + delta()
    val2 = (inputVal * yearToDate) + 100
    if ((yearToDate - val1) > 100) val2 -= 20
    val3 = val2 * 7
    return val3 - 2 * val1
  }
}

// After
class Account {
  gamma(inputVal, quantity, yearToDate) {
    return new Gamma(this, inputVal, quantity, yearToDate).compute()
  }
}
class Gamma {
  // fields: account, inputVal, quantity, yearToDate, val1, val2, val3
  compute() { /* freely decompose with Extract Method */ }
}
```

### Substitute Algorithm

**Motivation:** You find a clearer way to do something. Replace the complicated algorithm with the simpler one.

**Mechanics:**
- Prepare the alternative algorithm
- Run against tests; if results match, you are done
- If results differ, use old algorithm as reference for debugging

```
// Before
function foundPerson(people) {
  for (i = 0; i < people.length; i++) {
    if (people[i] == "Don") return "Don"
    if (people[i] == "John") return "John"
    if (people[i] == "Kent") return "Kent"
  }
  return ""
}

// After
function foundPerson(people) {
  candidates = ["Don", "John", "Kent"]
  for (i = 0; i < people.length; i++)
    if (candidates.contains(people[i])) return people[i]
  return ""
}
```

## Catalog: Moving Features Between Objects

### Move Method

**Motivation:** A method uses more features of another class than its own. Move it to where the data lives. The bread and butter of refactoring.

**Mechanics:**
- Examine features used by the method; consider whether they should also move
- Declare the method in the target class, copy and adjust the body
- Turn the source method into a delegate or remove it and update all callers

```
// Before
class Account {
  overdraftCharge() {
    if (type.isPremium()) {
      result = 10
      if (daysOverdrawn > 7) result += (daysOverdrawn - 7) * 0.85
      return result
    } else return daysOverdrawn * 1.75
  }
}

// After
class AccountType {
  overdraftCharge(daysOverdrawn) {
    if (isPremium()) {
      result = 10
      if (daysOverdrawn > 7) result += (daysOverdrawn - 7) * 0.85
      return result
    } else return daysOverdrawn * 1.75
  }
}
```

### Move Field

**Motivation:** A field is used more by another class than its own. Move state to where it is used most.

**Mechanics:**
- If public, use Encapsulate Field first
- Create the field with accessors in the target class
- Redirect all references from source to target
- Remove the source field

```
// Before
class Account {
  interestRate: double
  interestForAmountDays(amount, days) {
    return interestRate * amount * days / 365
  }
}

// After
class AccountType {
  interestRate: double
  getInterestRate() { return interestRate }
}
class Account {
  interestForAmountDays(amount, days) {
    return type.getInterestRate() * amount * days / 365
  }
}
```

### Extract Class

**Motivation:** A class doing work that should be done by two. Look for subsets of data and methods that go together, or data that changes together.

**Mechanics:**
- Decide how to split responsibilities; create a new class
- Move fields first (Move Field), then methods (Move Method), starting with lower-level ones
- Review and reduce interfaces of both classes

```
// Before
class Person {
  name, officeAreaCode, officeNumber
  getTelephoneNumber() { return "(" + officeAreaCode + ") " + officeNumber }
}

// After
class Person {
  name
  officeTelephone: TelephoneNumber
  getTelephoneNumber() { return officeTelephone.getTelephoneNumber() }
}
class TelephoneNumber {
  areaCode, number
  getTelephoneNumber() { return "(" + areaCode + ") " + number }
}
```

### Inline Class

**Motivation:** A class is not doing enough to earn its existence. Fold it into the class that uses it most. Reverse of Extract Class.

**Mechanics:**
- Declare the source class's public protocol on the absorbing class
- Move all features from source to absorbing class
- Delete the source class

```
// Before
class Person { officeTelephone: TelephoneNumber }
class TelephoneNumber { areaCode, number, getTelephoneNumber() }

// After
class Person {
  areaCode, number
  getTelephoneNumber() { return "(" + areaCode + ") " + number }
}
// TelephoneNumber removed
```

### Hide Delegate

**Motivation:** A client navigates through an object chain. Hide the delegation to reduce coupling.

**Mechanics:**
- For each delegate method used by clients, create a delegating method on the server
- Adjust clients to call the server; remove the delegate accessor if no longer needed

```
// Before
manager = john.getDepartment().getManager()

// After
// Person adds: getManager() { return department.getManager() }
manager = john.getManager()
```

### Remove Middle Man

**Motivation:** Too much simple delegation. Every new delegate feature requires adding another forwarding method. Let clients talk to the delegate directly.

**Mechanics:**
- Create an accessor for the delegate
- Replace delegate method calls on the server with direct delegate calls

```
// Before
manager = john.getManager()  // Person just delegates

// After
manager = john.getDepartment().getManager()
```

### Introduce Foreign Method

**Motivation:** A library class needs one or two methods you cannot add. Create the method on your client class with the library instance as the first parameter.

**Mechanics:**
- Create a method in the client class; make the server instance the first parameter
- Comment it as "foreign method; should be in [ServerClass]"

```
// Before
newStart = new Date(previousEnd.getYear(), previousEnd.getMonth(),
                    previousEnd.getDate() + 1)

// After
newStart = nextDay(previousEnd)
// Foreign method in client:
static nextDay(date) { return new Date(date.getYear(), date.getMonth(), date.getDate() + 1) }
```

### Introduce Local Extension

**Motivation:** A library class needs several additional methods. Create a subclass or wrapper that adds the missing functionality.

**Mechanics:**
- Create extension class as subclass or wrapper of the original
- Add converting constructors (takes original as argument)
- Add new features; move any foreign methods onto the extension

```
// Before (scattered foreign methods in client)

// After (subclass approach)
class MfDate extends Date {
  MfDate(original) { super(original.getTime()) }
  nextDay() { return new Date(getYear(), getMonth(), getDate() + 1) }
}
```

## Catalog: Organizing Data

### Self Encapsulate Field

**Motivation:** Route all field access through getters/setters so subclasses can override variable access with computed values.

**Mechanics:**
- Create getter and setter methods
- Replace all direct field references with accessor calls
- Make the field private

```
// Before
class IntRange {
  _low; _high
  includes(arg) { return arg >= _low && arg <= _high }
}

// After
class IntRange {
  _low; _high
  includes(arg) { return arg >= getLow() && arg <= getHigh() }
  getLow() { return _low }
  getHigh() { return _high }
}
// Subclass can override: getHigh() { return min(super.getHigh(), getCap()) }
```

### Replace Data Value with Object

**Motivation:** A simple data item starts needing formatting, validation, or related attributes. Turn it into an object.

**Mechanics:**
- Create a class for the value with a final field, getter, and constructor
- Change the source class to use the new class

```
// Before
class Order { _customer: string }  // just a name

// After
class Customer { _name: string; getName() { return _name } }
class Order { _customer: Customer }
```

### Change Value to Reference

**Motivation:** Many equal instances should be replaced with a single shared object so changes ripple to all users.

**Mechanics:**
- Replace Constructor with Factory Method
- Decide on a registry/lookup mechanism; alter factory to return the shared instance

```
// Before: each Order creates its own Customer
new Order("Alice")  // two orders for Alice = two Customer objects

// After: shared via registry
Customer.getNamed("Alice")  // both orders share one Customer object
```

### Change Reference to Value

**Motivation:** A reference object is small, immutable, and awkward to manage. Make it a value object.

**Mechanics:**
- Ensure the object is immutable (use Remove Setting Method)
- Implement equals and hashCode based on data values

```
// Before: Currency uses registry, private constructor
Currency.get("USD")

// After: Currency is a value object
new Currency("USD").equals(new Currency("USD"))  // true
```

### Replace Array with Object

**Motivation:** An array where different elements mean different things (e.g., index 0 = name, index 1 = wins).

**Mechanics:**
- Create a class with named fields for each array element
- Replace array access with accessor methods

```
// Before
row[0] = "Liverpool"; row[1] = "15"

// After
perf = new Performance()
perf.setName("Liverpool"); perf.setWins("15")
```

### Duplicate Observed Data

**Motivation:** GUI class contains domain data. Separate it into a domain object and sync via observer pattern.

**Mechanics:**
- Create domain class; make GUI an observer
- Use Self Encapsulate Field on GUI data, then redirect accessors to domain
- Sync GUI from domain in the update method

```
// Before: all data and logic in IntervalWindow (GUI class)

// After
class Interval extends Observable { _end; setEnd(arg) { _end = arg; notify() } }
class IntervalWindow implements Observer {
  update() { endField.setText(subject.getEnd()) }
}
```

### Change Unidirectional Association to Bidirectional

**Motivation:** A client of a referred class needs to navigate back to the referring objects.

**Mechanics:**
- Add a back-pointer field; decide which class controls the association (one-to-many: the "one" side controls)
- Create a helper on the non-controlling side; update the controlling side's modifier to maintain both pointers

```
// Before: Order -> Customer (one-way)
// After: Order <-> Customer (two-way)
// Order.setCustomer(arg) updates both pointers
// Customer.friendOrders() exposes restricted helper
```

### Change Bidirectional Association to Unidirectional

**Motivation:** Bidirectional link is no longer needed. Drop it to reduce complexity, coupling, and zombie-object risk.

**Mechanics:**
- Determine which direction to keep; substitute the removed direction with lookup or parameter passing
- Remove the unneeded field and update code

```
// Before: Order <-> Customer
// After: Order -> Customer only
// Customer no longer needs _orders back-pointer
```

### Replace Magic Number with Symbolic Constant

**Motivation:** Literal numbers with special meaning are hard to understand and dangerous when duplicated.

**Mechanics:**
- Declare a constant; replace all matching occurrences

```
// Before
return mass * 9.81 * height

// After
GRAVITATIONAL_CONSTANT = 9.81
return mass * GRAVITATIONAL_CONSTANT * height
```

### Encapsulate Field

**Motivation:** Public fields violate encapsulation. First step toward a richer class.

**Mechanics:**
- Create getter and setter; make field private; redirect all external access

```
// Before
class Person { public name }

// After
class Person { private _name; getName() { return _name }; setName(arg) { _name = arg } }
```

### Encapsulate Collection

**Motivation:** A getter returns the raw collection, letting clients manipulate contents without the owner knowing.

**Mechanics:**
- Provide add/remove methods for individual elements
- Return a read-only view from the getter; remove the collection setter

```
// Before
person.getCourses().add(newCourse)  // bypasses owner

// After
person.addCourse(newCourse)
person.getCourses()  // returns unmodifiable view
```

### Replace Record with Data Class

**Motivation:** Interface with a traditional record structure (database row, legacy struct). Wrap it in a class as a starting point for adding behavior.

**Mechanics:**
- Create a class with private fields, getters, and setters for each data item

```
// Before
record = { name: "...", age: 30 }

// After
class PersonRecord { private name; private age; getName(); setName(); getAge(); setAge() }
```

### Replace Type Code with Class

**Motivation:** Numeric type code does NOT affect behavior. Provide type safety with a class instead of raw integers.

**Mechanics:**
- Create a class with static instances for each code value
- Replace the type code field with the new class

```
// Before
class Person { static O = 0; static A = 1; bloodGroup: int }

// After
class BloodGroup { static O = new BloodGroup(0); static A = new BloodGroup(1) }
class Person { bloodGroup: BloodGroup }
```

### Replace Type Code with Subclasses

**Motivation:** Immutable type code affects behavior (conditionals switch on it). Create subclasses to enable polymorphism. Cannot use if type code changes after creation or class is already subclassed.

**Mechanics:**
- Self-encapsulate the type code; replace constructor with factory method
- Create a subclass for each value, overriding the type code getter
- Make the superclass accessor abstract

```
// Before
class Employee { type: int; static ENGINEER = 0; static SALESMAN = 1 }

// After
abstract class Employee { abstract getType() }
class Engineer extends Employee { getType() { return ENGINEER } }
class Salesman extends Employee { getType() { return SALESMAN } }
// Factory: Employee.create(type) returns the right subclass
```

### Replace Type Code with State/Strategy

**Motivation:** Type code affects behavior BUT changes during object lifetime or class is already subclassed. Delegate to a state/strategy object that can be swapped.

**Mechanics:**
- Create abstract state class with subclass per type code value
- Replace type code field with state object; delegate getter to state
- Setter creates the appropriate state subclass

```
// Before
class Employee { type: int /* mutable */ }

// After
class Employee {
  employeeType: EmployeeType
  getType() { return employeeType.getTypeCode() }
  setType(arg) { employeeType = EmployeeType.newType(arg) }
}
abstract class EmployeeType { abstract getTypeCode() }
class EngineerType extends EmployeeType { getTypeCode() { return ENGINEER } }
```

### Replace Subclass with Fields

**Motivation:** Subclasses only return different constant values (hard-coded constant methods). The inheritance is not pulling its weight.

**Mechanics:**
- Replace constructors with factory methods
- Move constant values into superclass fields; eliminate subclasses

```
// Before
abstract class Person { abstract isMale(); abstract getCode() }
class Male extends Person { isMale() { return true }; getCode() { return 'M' } }

// After
class Person {
  isMaleFlag; code
  static createMale() { return new Person(true, 'M') }
  static createFemale() { return new Person(false, 'F') }
}
```

## Catalog: Simplifying Conditional Expressions

### Decompose Conditional

**Motivation:** Complex conditional logic obscures what happens and why. Extract the condition, then-part, and else-part into named methods.

**Mechanics:**
- Extract the condition into a method named after the intent
- Extract the then-part and else-part each into their own methods

```
// Before
if (date.isBefore(SUMMER_START) || date.isAfter(SUMMER_END))
  charge = quantity * winterRate + winterServiceCharge
else
  charge = quantity * summerRate

// After
if (isNotSummer(date))
  charge = winterCharge(quantity)
else
  charge = summerCharge(quantity)
```

### Consolidate Conditional Expression

**Motivation:** A series of checks with the same result should be combined into a single check and extracted into a named method.

**Mechanics:**
- Verify no conditionals have side effects
- Combine with logical operators (AND/OR)
- Extract the combined condition into a well-named method

```
// Before
if (seniority < 2) return 0
if (monthsDisabled > 12) return 0
if (isPartTime) return 0

// After
if (isNotEligibleForDisability()) return 0
```

### Consolidate Duplicate Conditional Fragments

**Motivation:** Same code appears in all branches of a conditional. Move it outside.

**Mechanics:**
- Identify code executed the same way regardless of the condition
- Move common code before or after the conditional

```
// Before
if (isSpecialDeal()) { total = price * 0.95; send() }
else { total = price * 0.98; send() }

// After
if (isSpecialDeal()) total = price * 0.95
else total = price * 0.98
send()
```

### Remove Control Flag

**Motivation:** A boolean flag controls a loop with convoluted conditionals. Replace with break, continue, or return.

**Mechanics:**
- Extract the logic into a method if needed
- Replace flag assignments with return (preferred) or break

```
// Before
found = false
for (i = 0; i < people.length; i++) {
  if (!found) {
    if (people[i] == "Don") { sendAlert(); found = true }
  }
}

// After
function foundMiscreant(people) {
  for (i = 0; i < people.length; i++)
    if (people[i] == "Don") { sendAlert(); return "Don" }
  return ""
}
```

### Replace Nested Conditional with Guard Clauses

**Motivation:** Nested if-then-else buries the normal flow. Guard clauses handle unusual cases and exit early, making the normal path clear.

**Mechanics:**
- For each unusual condition, add a guard clause that returns or throws
- The normal path falls through to the end

```
// Before
function getPayAmount() {
  if (isDead) { result = deadAmount() }
  else {
    if (isSeparated) { result = separatedAmount() }
    else {
      if (isRetired) { result = retiredAmount() }
      else { result = normalPayAmount() }
    }
  }
  return result
}

// After
function getPayAmount() {
  if (isDead) return deadAmount()
  if (isSeparated) return separatedAmount()
  if (isRetired) return retiredAmount()
  return normalPayAmount()
}
```

### Replace Conditional with Polymorphism

**Motivation:** Switch on type code should be replaced with polymorphic dispatch. Adding a new type means adding a subclass, not hunting down every conditional.

**Mechanics:**
- Set up inheritance structure (Replace Type Code with Subclasses or State/Strategy)
- For each leg of the conditional, create an overriding method in the appropriate subclass
- Make the superclass method abstract (or provide a default)

```
// Before
function getSpeed() {
  switch (type) {
    case EUROPEAN: return baseSpeed()
    case AFRICAN:  return baseSpeed() - loadFactor() * coconuts
    case NORWEGIAN_BLUE: return isNailed ? 0 : baseSpeed(voltage)
  }
}

// After
class European  { getSpeed() { return baseSpeed() } }
class African   { getSpeed() { return baseSpeed() - loadFactor() * coconuts } }
class NorwegianBlue { getSpeed() { return isNailed ? 0 : baseSpeed(voltage) } }
```

### Introduce Null Object

**Motivation:** Multiple places check for null and do the same default thing. Create a null subclass that provides default behavior polymorphically.

**Mechanics:**
- Create a subclass with isNull() returning true and default behavior methods
- Replace null returns with null object instances; remove null checks from clients

```
// Before
if (customer == null) plan = BillingPlan.basic()
else plan = customer.getPlan()

// After
class NullCustomer extends Customer {
  isNull() { return true }
  getPlan() { return BillingPlan.basic() }
  getName() { return "occupant" }
}
// Site returns NullCustomer instead of null
plan = customer.getPlan()  // no null check needed
```

### Introduce Assertion

**Motivation:** Code assumes certain conditions are true but never states them. Make assumptions explicit with assertions. Assertions are executable documentation.

**Mechanics:**
- Add an assert statement for each implicit assumption
- Assertions are for programmer errors, not user input; they should fail loudly

```
// Before
function getExpenseLimit() {
  // should have either expense limit or primary project
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit : primaryProject.getMemberExpenseLimit()
}

// After
function getExpenseLimit() {
  Assert.isTrue(expenseLimit != NULL_EXPENSE || primaryProject != null)
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit : primaryProject.getMemberExpenseLimit()
}
```

## Catalog: Making Method Calls Simpler

### Rename Method

**Motivation:** The method name does not reveal its purpose. A good name eliminates the need to read the body.

**Mechanics:**
- Create a new method with the better name; copy the body
- Change the old method to delegate to the new one; update all callers; remove the old method

```
// Before
getInvcdtlmt() { ... }

// After
getInvoiceableCreditLimit() { ... }
```

### Add Parameter

**Motivation:** A method needs more information from its caller.

**Mechanics:**
- Declare a new method with the added parameter; copy the body
- Update callers to pass the new argument; remove the old method

```
// Before
getContact() { ... }

// After
getContact(date) { ... }
```

### Remove Parameter

**Motivation:** A parameter is no longer used by the method body. Remove it to simplify.

**Mechanics:**
- Check superclasses/subclasses for usage; if unused, remove from signature and all callers

```
// Before
getContact(date) { /* date unused */ }

// After
getContact() { ... }
```

### Separate Query from Modifier

**Motivation:** A method returns a value AND changes state. Separate them so the query has no side effects (Command-Query Separation).

**Mechanics:**
- Create a query that returns the value (no side effects)
- Create a modifier that performs the side effect (void return)
- Replace callers with two calls: modifier then query

```
// Before
foundMiscreant(people) {
  for each person in people {
    if person == "Don" { sendAlert(); return "Don" }
  }
  return ""
}

// After
foundPerson(people) {  // query only
  for each person in people { if person == "Don" return "Don" }
  return ""
}
checkSecurity(people) { if (foundPerson(people) != "") sendAlert() }  // modifier only
```

### Parameterize Method

**Motivation:** Several methods do similar things with different literal values. Combine into one with a parameter.

**Mechanics:**
- Create a parameterized method; replace each specific method with a call to it

```
// Before
fivePercentRaise() { salary *= 1.05 }
tenPercentRaise() { salary *= 1.10 }

// After
raise(factor) { salary *= (1 + factor) }
```

### Replace Parameter with Explicit Methods

**Motivation:** A method runs different code depending on an enumerated parameter. Create separate methods for clarity and compile-time checking. Do NOT use if parameter values change frequently.

**Mechanics:**
- Create an explicit method for each parameter value
- Replace callers with calls to the specific method

```
// Before
setValue(name, value) {
  if (name == "height") _height = value
  if (name == "width") _width = value
}

// After
setHeight(arg) { _height = arg }
setWidth(arg) { _width = arg }
```

### Preserve Whole Object

**Motivation:** You extract several values from an object and pass them as individual parameters. Pass the whole object instead.

**Mechanics:**
- Add the whole object as a parameter; have the method get what it needs from the object
- Remove the individual parameters

```
// Before
low = daysTempRange().getLow()
high = daysTempRange().getHigh()
withinPlan = plan.withinRange(low, high)

// After
withinPlan = plan.withinRange(daysTempRange())
```

### Replace Parameter with Method

**Motivation:** A caller computes a value and passes it as parameter, but the receiver can compute it itself.

**Mechanics:**
- Remove the parameter; have the method call the appropriate object to get the value

```
// Before
basePrice = quantity * itemPrice
discountLevel = getDiscountLevel()
finalPrice = discountedPrice(basePrice, discountLevel)

// After
finalPrice = discountedPrice(basePrice)
// discountedPrice() calls getDiscountLevel() internally
```

### Introduce Parameter Object

**Motivation:** A group of parameters appears together in many method signatures. Bundle them into an object. Often reveals behavior that can move to the new class.

**Mechanics:**
- Create an immutable class for the parameter group
- Replace parameter groups with the new object in signatures and callers

```
// Before
getFlowBetween(startDate, endDate) {
  for each entry: if entry.date >= startDate && entry.date <= endDate ...
}

// After
class DateRange { start; end; includes(date) { return date >= start && date <= end } }
getFlowBetween(range) {
  for each entry: if range.includes(entry.date) ...
}
```

### Remove Setting Method

**Motivation:** A field should be set at creation time and never altered. Removing the setter communicates immutability intent.

**Mechanics:**
- Verify the setter is called only during construction
- Modify constructor to set the field directly; make the field final/readonly; delete the setter

```
// Before
class Account { _id; constructor(id) { setId(id) }; setId(arg) { _id = arg } }

// After
class Account { final _id; constructor(id) { _id = id } }
```

### Hide Method

**Motivation:** A method is not used by any other class. Reduce visibility to the minimum required.

**Mechanics:**
- Check for external usage; make the method private (or protected)

```
// Before
class Employee { + helperMethod() }  // public but unused externally

// After
class Employee { - helperMethod() }  // now private
```

### Replace Constructor with Factory Method

**Motivation:** You need more than simple construction -- returning subclasses, different creation strategies, or signaling intent.

**Mechanics:**
- Create a static factory method that calls the constructor
- Replace all constructor calls with factory calls; make constructor private

```
// Before
eng = new Employee(ENGINEER)

// After
eng = Employee.create(ENGINEER)
// Or: eng = Employee.createEngineer()
```

### Encapsulate Downcast

**Motivation:** A method returns a generic type that callers must downcast. Move the cast into the method.

**Mechanics:**
- Change the method return type to the specific type; move the cast inside

```
// Before (caller must cast)
reading = (Reading) theSite.readings().lastElement()

// After
Reading lastReading() { return (Reading) readings.lastElement() }
reading = theSite.lastReading()
```

### Replace Error Code with Exception

**Motivation:** A method returns a special code to indicate error. Exceptions clearly separate normal from error processing.

**Mechanics:**
- Decide checked vs unchecked (caller's responsibility to test first = unchecked; method's responsibility = checked)
- Replace error code returns with throw; adjust callers

```
// Before
withdraw(amount) { if (amount > balance) return -1; balance -= amount; return 0 }

// After (unchecked -- caller tests first)
withdraw(amount) { Assert.isTrue(amount <= balance); balance -= amount }
// Caller: if (!account.canWithdraw(amount)) handleOverdrawn()
//         else account.withdraw(amount)
```

### Replace Exception with Test

**Motivation:** An exception is used for a condition the caller could have tested first. Use a conditional instead.

**Mechanics:**
- Add a conditional test before the operation; remove the try/catch

```
// Before
try { result = available.pop() }
catch (EmptyStackException) { result = new Resource() }

// After
if (available.isEmpty()) result = new Resource()
else result = available.pop()
```

## Catalog: Dealing with Generalization

### Pull Up Field

**Motivation:** Two sibling subclasses have the same field. Move it to the common superclass to eliminate duplication and enable moving behavior up.

**Mechanics:**
- Inspect all uses to ensure the fields are used the same way
- If names differ, rename to the target name; move field to superclass
- Delete the subclass fields; compile and test

```
// Before
class Salesman extends Employee { name: string }
class Engineer extends Employee { name: string }

// After
class Employee { name: string }
class Salesman extends Employee {}
class Engineer extends Employee {}
```

### Pull Up Method

**Motivation:** Two sibling subclasses have identical methods. Eliminate duplication by moving to the superclass.

**Mechanics:**
- Inspect methods to ensure they are identical (use Substitute Algorithm if similar but not identical)
- If the method references subclass-only features, use Pull Up Field or declare an abstract method
- Move to superclass; delete one subclass copy at a time, testing after each

```
// Before
class RegularCustomer { createBill(date) { charge = chargeFor(lastBillDate, date); addBill(date, charge) } }
class PreferredCustomer { createBill(date) { charge = chargeFor(lastBillDate, date); addBill(date, charge) } }

// After
class Customer { createBill(date) { charge = chargeFor(lastBillDate, date); addBill(date, charge) }
  abstract chargeFor(start, end) }
```

### Pull Up Constructor Body

**Motivation:** Subclass constructors have mostly identical bodies. You cannot use Pull Up Method on constructors, so extract the common code into a superclass constructor and call it with super().

**Mechanics:**
- Define a superclass constructor
- Move common code from the beginning of the subclass constructor to super()
- Call the superclass constructor as the first step in each subclass constructor
- If there is common code after subclass-specific assignments, use Extract Method + Pull Up Method

```
// Before
class Manager extends Employee {
  constructor(name, id, grade) {
    _name = name
    _id = id
    _grade = grade
  }
}

// After
class Employee {
  constructor(name, id) { _name = name; _id = id }
}
class Manager extends Employee {
  constructor(name, id, grade) {
    super(name, id)
    _grade = grade
  }
}
```

### Push Down Method

**Motivation:** Behavior on a superclass is relevant only to some subclasses. Move it to where it belongs.

**Mechanics:**
- Declare the method in all subclasses that need it; copy the body
- Remove method from superclass; compile and test

```
// Before
class Employee { getQuota() { ... } }  // only Engineers have quotas

// After
class Employee {}
class Engineer extends Employee { getQuota() { ... } }
```

### Push Down Field

**Motivation:** A field is used only by some subclasses, not all.

**Mechanics:**
- Declare the field in the relevant subclasses; remove from superclass

```
// Before
class Employee { quota: int }  // only Salesmen use quota

// After
class Salesman extends Employee { quota: int }
```

### Extract Subclass

**Motivation:** A class has features used only in some instances. Create a subclass for that subset. Simpler than Extract Class but limited to creation-time decisions.

**Mechanics:**
- Create a subclass; provide constructors
- Replace construction calls with the subclass where appropriate; use Replace Constructor with Factory Method to hide the subclass if needed
- Push Down Method and Push Down Field for subset-only features
- Eliminate type-code fields with Self Encapsulate Field + Replace Conditional with Polymorphism

```
// Before
class JobItem { unitPrice; quantity; isLabor; employee }

// After
class JobItem { unitPrice; quantity }
class LaborItem extends JobItem { employee; getUnitPrice() { return employee.getRate() } }
```

### Extract Superclass

**Motivation:** Two classes with similar features. Pull common behavior into a shared superclass to eliminate duplication.

**Mechanics:**
- Create abstract superclass; make both classes extend it
- Use Pull Up Field, Pull Up Method, Pull Up Constructor Body (fields first)
- If methods have different signatures but same intent, use Rename Method first
- If methods have same signature but different bodies, declare abstract on superclass

```
// Before
class Employee { name; annualCost; getId() }
class Department { name; getAnnualCost(); getHeadCount() }

// After
abstract class Party { name; abstract getAnnualCost() }
class Employee extends Party { id; getAnnualCost() { return annualCost } }
class Department extends Party { getAnnualCost() { return staff.sum(getAnnualCost) } }
```

### Extract Interface

**Motivation:** Several clients use the same subset of a class's interface, or two classes share part of their protocol. Make that subset explicit for documentation and flexibility.

**Mechanics:**
- Create an empty interface; declare the common operations
- Have the relevant class(es) implement the interface
- Adjust client type declarations to use the interface

```
// Before
charge(Employee emp, days) { base = emp.getRate() * days; ... }

// After
interface Billable { getRate(); hasSpecialSkill() }
class Employee implements Billable { ... }
charge(Billable emp, days) { base = emp.getRate() * days; ... }
```

### Collapse Hierarchy

**Motivation:** A superclass and subclass are not different enough to justify separate existence. Merge them.

**Mechanics:**
- Choose which to keep; move all fields and methods into the surviving class
- Adjust references; remove the empty class; compile and test

```
// Before
class Employee {}
class Salesman extends Employee {}  // no meaningful difference

// After
class Employee {}  // Salesman merged in
```

### Form Template Method

**Motivation:** Two subclass methods perform similar steps in the same order, but the steps differ. Separate the invariant structure from the varying parts using the Template Method pattern.

**Mechanics:**
- Extract Method in each subclass to isolate differing parts (same signature, different body)
- Pull Up Method on the now-identical overall method
- Declare the differing extracted methods as abstract on the superclass

```
// Before (two subclasses with similar structure)
class TextStatement { value(customer) { result = header + eachRental + footer; return result } }
class HtmlStatement { value(customer) { result = header + eachRental + footer; return result } }

// After
class Statement {
  value(customer) { result = headerString(customer); for each: result += eachRentalString(r); result += footerString(customer) }
  abstract headerString(customer)
  abstract eachRentalString(rental)
  abstract footerString(customer)
}
class TextStatement extends Statement { headerString(c) { return "Record for " + c.name + "\n" } ... }
class HtmlStatement extends Statement { headerString(c) { return "<H1>" + c.name + "</H1>" } ... }
```

### Replace Inheritance with Delegation

**Motivation:** A subclass reuses behavior but does not want to support the parent's interface (refuses the bequest of the interface, not just implementations).

**Mechanics:**
- Create a field in the subclass to hold a delegate instance of the former parent
- Replace inherited method calls with delegation; remove the inheritance relationship

```
// Before
class Stack extends ArrayList {
  push(item) { add(item) }
  pop() { return remove(size() - 1) }
}

// After
class Stack {
  list = new ArrayList()
  push(item) { list.add(item) }
  pop() { return list.remove(list.size() - 1) }
}
```

### Replace Delegation with Inheritance

**Motivation:** You are delegating everything to another class and tired of writing forwarding methods. If appropriate, become a subclass instead.

**Mechanics:**
- Make the delegating class a subclass of the delegate
- Remove the delegation field and forwarding methods

### Tease Apart Inheritance

**Motivation:** An inheritance hierarchy is doing two jobs at once (tangled dimensions). Separate into two hierarchies connected by delegation.

**Mechanics:**
- Create a 2D grid of the jobs; decide which to keep in the current hierarchy
- Extract Class for the subsidiary job; create subclasses of the extracted object
- Move methods for the subsidiary job into the extracted hierarchy

```
// Before: one hierarchy doing deal type + presentation style
class TabularActiveDeal extends ActiveDeal { /* presentation + deal logic */ }

// After: two hierarchies
class ActiveDeal extends Deal { presentationStyle: PresentationStyle }
class TabularPresentation extends PresentationStyle { }
```

## Catalog: Big Refactorings

These take months, not hours. Do them incrementally alongside feature work. Require team agreement on direction.

### Tease Apart Inheritance

See above in Dealing with Generalization. The big-refactoring aspect: untangling deeply entwined hierarchies is a long-term project done step by step.

### Convert Procedural Design to Objects

**Motivation:** Long procedural methods on "manager" classes and dumb data objects with only accessors.

**Strategy:**
- Turn record types into dumb data objects with accessors
- Put all procedural code into a single class
- Apply Extract Method to break down procedures, then Move Method to push behavior into the data objects
- Continue until the procedural class is empty and can be deleted

```
// Before
class OrderCalculator { determinePrice(order) { /* long procedure */ } }
class Order { /* only fields and accessors */ }

// After
class Order { getPrice() { /* behavior lives here */ } }
```

### Separate Domain from Presentation

**Motivation:** GUI classes contain business logic. Separate domain data/behavior from presentation concerns.

**Strategy:**
- Create domain classes for each window
- Move domain data with Duplicate Observed Data; move logic with Extract Method + Move Method
- Drive the process by moving all SQL/data-access calls to domain classes first

```
// Before: OrderWindow has business logic, SQL, and GUI code

// After
class OrderWindow { order: Order /* only UI concerns */ }
class Order { /* all business logic and data access */ }
```

### Extract Hierarchy

**Motivation:** A class doing too much via many conditionals. Flags and conditional expressions accumulated over time.

**Strategy:**
- If variations are unclear: pick one variation, create a subclass, move conditional logic for that case into it, simplify. Repeat for each variation.
- If variations are clear: create all subclasses at once, use Replace Conditional with Polymorphism.
- If the variation changes during object lifetime, use Extract Class first.

```
// Before
class BillingScheme { createBill() { if disability... if lifeline... if business... } }

// After
abstract class BillingScheme { abstract createBill() }
class BusinessBilling extends BillingScheme { createBill() { /* no conditionals */ } }
class ResidentialBilling extends BillingScheme { createBill() { /* no conditionals */ } }
```

## Quick Decision Guide

| I see... | Try... |
|----------|--------|
| Long method with comments marking sections | Extract Method for each section |
| Same code in two places | Extract Method (same class), Extract Superclass / Form Template Method (siblings) |
| Method uses another class's data more than its own | Move Method |
| Field used more by another class | Move Field |
| Several params always appear together | Introduce Parameter Object or Extract Class |
| Primitive used for money, range, phone, type code | Replace Data Value with Object, or Replace Type Code with Class/Subclasses/State-Strategy |
| Switch on type code | Replace Type Code with Subclasses or State/Strategy, then Replace Conditional with Polymorphism |
| Nested if-else with unusual conditions | Replace Nested Conditional with Guard Clauses |
| Complex condition in if statement | Decompose Conditional |
| Temp variable blocking extraction | Replace Temp with Query |
| Multiple temps tangled together | Split Temporary Variable, then Replace Temp with Query |
| Method too tangled for Extract Method | Replace Method with Method Object |
| Null checks scattered everywhere | Introduce Null Object |
| Class with too many fields/methods | Extract Class or Extract Subclass |
| Class with almost no behavior left | Inline Class or Collapse Hierarchy |
| Half the methods just delegate | Remove Middle Man |
| Client navigates a.getB().getC() | Hide Delegate |
| Library class needs extra methods | Introduce Foreign Method (1-2 methods), Introduce Local Extension (many) |
| Data class with only getters/setters | Move Method to give it behavior; Encapsulate Field/Collection |
| Subclass does not want inherited methods | Push Down Method, or Replace Inheritance with Delegation |
| Hierarchy does two jobs at once | Tease Apart Inheritance |
| GUI class has business logic | Separate Domain from Presentation |
| Procedural code with dumb data objects | Convert Procedural Design to Objects |
| Speculative unused abstractions | Collapse Hierarchy, Inline Class, Remove Parameter |
| Method returns value AND has side effects | Separate Query from Modifier |
| Error codes returned from methods | Replace Error Code with Exception |
| Exception used for non-exceptional flow | Replace Exception with Test |
| Magic numbers in code | Replace Magic Number with Symbolic Constant |
| Public fields on a class | Encapsulate Field |
| Getter returns raw mutable collection | Encapsulate Collection |
| Implicit assumptions not documented | Introduce Assertion |
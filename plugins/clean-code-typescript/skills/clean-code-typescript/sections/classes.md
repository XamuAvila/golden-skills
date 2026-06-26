# Classes

### Prefer ES6 classes over ES5 plain functions

The classic prototype-based pattern for inheritance is verbose, error-prone, and hard to read:
you wire up `Object.create`, reset `constructor`, and call the parent by name. ES6 `class`
syntax expresses the same hierarchy declaratively with `extends` and `super()`, and TypeScript
adds typed properties and access modifiers on top.

**Bad:**
```typescript
// ES5 prototype-based inheritance — manual and brittle.
function Animal(age: number) {
  // @ts-expect-error: loosely typed `this`.
  this.age = age;
}
Animal.prototype.move = function (): void { /* ... */ };

function Mammal(age: number, furColor: string) {
  Animal.call(this, age);
  // @ts-expect-error
  this.furColor = furColor;
}
Mammal.prototype = Object.create(Animal.prototype);
Mammal.prototype.constructor = Mammal;
Mammal.prototype.liveBirth = function (): void { /* ... */ };

function Human(age: number, furColor: string, languageSpoken: string) {
  Mammal.call(this, age, furColor);
  // @ts-expect-error
  this.languageSpoken = languageSpoken;
}
Human.prototype = Object.create(Mammal.prototype);
Human.prototype.constructor = Human;
Human.prototype.speak = function (): void { /* ... */ };
```

**Good:**
```typescript
class Animal {
  constructor(protected age: number) {}

  move(): void { /* ... */ }
}

class Mammal extends Animal {
  constructor(age: number, protected furColor: string) {
    super(age);
  }

  liveBirth(): void { /* ... */ }
}

class Human extends Mammal {
  constructor(age: number, furColor: string, private languageSpoken: string) {
    super(age, furColor);
  }

  speak(): void { /* ... */ }
}
```

### Use method chaining (fluent API)

When a class has many configuration steps, returning `this` from each mutating method lets
callers chain calls into a single readable expression instead of repeating the variable name.
In TypeScript, annotate the return type as `this` (the polymorphic *this type*) rather than the
concrete class — that way subclasses keep their own type through the chain, so methods defined
only on the subclass remain callable after a base-class method in the chain.

**Bad:**
```typescript
class QueryBuilder {
  private table = "";
  private columns: string[] = [];
  private wheres: string[] = [];

  from(table: string): void {
    this.table = table;
  }

  select(...columns: string[]): void {
    this.columns = columns;
  }

  where(condition: string): void {
    this.wheres.push(condition);
  }

  build(): string {
    return `SELECT ${this.columns.join(", ")} FROM ${this.table} WHERE ${this.wheres.join(" AND ")}`;
  }
}

// Repetitive: the variable is named on every line.
const query = new QueryBuilder();
query.from("users");
query.select("id", "name");
query.where("active = true");
const sql = query.build();
```

**Good:**
```typescript
class QueryBuilder {
  private table = "";
  private columns: string[] = [];
  private wheres: string[] = [];

  // Returning `this` (the polymorphic this type) keeps chaining subclass-safe.
  from(table: string): this {
    this.table = table;
    return this;
  }

  select(...columns: string[]): this {
    this.columns = columns;
    return this;
  }

  where(condition: string): this {
    this.wheres.push(condition);
    return this;
  }

  build(): string {
    return `SELECT ${this.columns.join(", ")} FROM ${this.table} WHERE ${this.wheres.join(" AND ")}`;
  }
}

const sql = new QueryBuilder()
  .from("users")
  .select("id", "name")
  .where("active = true")
  .build();
```

### Prefer composition over inheritance

Reach for inheritance only when the subtype genuinely *is-a* supertype and you want to inherit
its full interface. When the relationship is really *has-a* — one object owns another as a
collaborator — model it with a property, not `extends`. Inheritance couples you to the base
class's entire surface and forces an artificial "is-a" claim; composition keeps each concern
isolated and recombinable. Use TypeScript interfaces to define the contract of the composed
parts.

**Bad:**
```typescript
// An employee HAS tax data — it is NOT a kind of tax data.
// Inheritance forces a false "is-a" relationship.
class EmployeeTaxData {
  constructor(
    protected ssn: string,
    protected salary: number,
  ) {}
}

class Employee extends EmployeeTaxData {
  constructor(
    private name: string,
    private email: string,
    ssn: string,
    salary: number,
  ) {
    super(ssn, salary);
  }
}
```

**Good:**
```typescript
// "has-a": Employee owns a TaxData collaborator via a property.
interface TaxData {
  readonly ssn: string;
  readonly salary: number;
}

class Employee {
  private taxData?: TaxData;

  constructor(
    private name: string,
    private email: string,
  ) {}

  setTaxData(ssn: string, salary: number): this {
    this.taxData = { ssn, salary };
    return this;
  }
}
```

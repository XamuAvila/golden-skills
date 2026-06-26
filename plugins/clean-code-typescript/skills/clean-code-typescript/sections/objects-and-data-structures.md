# Objects and Data Structures

### Use getters and setters

TypeScript supports native `get`/`set` accessors. Prefer them over manual `getX()`/`setX()`
methods: callers keep a property-like syntax while you retain a hook to add validation,
logging, or lazy computation later — without changing the call sites. Back the accessors with
`private` fields so the raw value can't be reached or mutated directly.

**Bad:**
```typescript
class BankAccount {
  // Public and mutable: anyone can set a negative balance.
  balance = 0;
}

const account = new BankAccount();
account.balance = -100; // No validation — invalid state is now reachable.
```

**Good:**
```typescript
class BankAccount {
  private _balance = 0;

  get balance(): number {
    return this._balance;
  }

  // Validation lives in one place; invalid state is impossible.
  set balance(amount: number) {
    if (amount < 0) {
      throw new Error("Balance cannot be negative.");
    }
    this._balance = amount;
  }
}

const account = new BankAccount();
account.balance = 100; // Goes through the setter.
// account.balance = -100; // Throws — the invariant is enforced.
```

### Make objects have private members

Encapsulation keeps internal state from being read or mutated by outside code. The old ES5
prototype pattern leaves every property public — and therefore deletable and overwritable.
TypeScript gives you two stronger tools: the `private` keyword (compile-time enforcement) and
ES2022 `#` private fields (runtime-enforced, truly inaccessible from outside the class).

**Bad:**
```typescript
// ES5 prototype pattern — `name` is a plain public property.
function Employee(name: string) {
  // @ts-expect-error: `this` is loosely typed in this pattern.
  this.name = name;
}

Employee.prototype.getName = function (): string {
  return this.name;
};

const employee = new Employee("John Doe");
// Nothing protects the state:
delete (employee as { name?: string }).name; // Property silently removed.
console.log(`Employee name: ${(employee as { name?: string }).name}`); // undefined
```

**Good:**
```typescript
// `private` — enforced by the TypeScript compiler.
class Employee {
  constructor(private name: string) {}

  getName(): string {
    return this.name;
  }
}

const employee = new Employee("John Doe");
// employee.name;            // Error: Property 'name' is private.
// delete employee.name;     // Error: cannot delete a private member.
console.log(`Employee name: ${employee.getName()}`); // John Doe
```

```typescript
// `#` private fields — enforced at runtime (ES2022).
// Inaccessible even via bracket access or casting to `any`.
class Account {
  #balance: number;

  constructor(initialBalance: number) {
    this.#balance = initialBalance;
  }

  get balance(): number {
    return this.#balance;
  }
}

const account = new Account(500);
console.log(account.balance);          // 500
// account.#balance;                   // SyntaxError at parse time.
// (account as any)["#balance"];       // undefined — truly hidden, not just hidden by types.
```

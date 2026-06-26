# Functions

### Function arguments (2 or fewer ideally)

Limiting the number of parameters makes functions easier to test and call. When you need more than two, group them into a single object and destructure it with a typed interface.

**Bad:**
```typescript
function createMenu(title: string, body: string, buttonText: string, cancellable: boolean): void {
  // ...
}

createMenu("Foo", "Bar", "Baz", true);
```

**Good:**
```typescript
interface MenuOptions {
  title: string;
  body: string;
  buttonText: string;
  cancellable: boolean;
}

function createMenu({ title, body, buttonText, cancellable }: MenuOptions): void {
  // ...
}

createMenu({
  title: "Foo",
  body: "Bar",
  buttonText: "Baz",
  cancellable: true,
});
```

### Functions should do one thing

A function that filters and acts on data does two things. Split the predicate from the processor so each piece is independently testable and reusable.

**Bad:**
```typescript
function emailClients(clients: Client[]): void {
  clients.forEach((client) => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**Good:**
```typescript
function emailActiveClients(clients: Client[]): void {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client: Client): boolean {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

### Function names should say what they do

A reader should understand a function's behavior from its name and call site alone, without inspecting its body.

**Bad:**
```typescript
function addToDate(date: Date, value: number): Date {
  // unclear: adding days? months? years?
  return new Date(date.getFullYear(), date.getMonth() + value, date.getDate());
}

const date = new Date();
addToDate(date, 1);
```

**Good:**
```typescript
function addMonthToDate(months: number, date: Date): Date {
  return new Date(date.getFullYear(), date.getMonth() + months, date.getDate());
}

const date = new Date();
addMonthToDate(1, date);
```

### Functions should have one level of abstraction

When a function mixes high-level orchestration with low-level details, it becomes hard to read and reuse. Extract each level into its own function with a typed return.

**Bad:**
```typescript
function parseCode(code: string): string[] {
  const REGEXES = [/* ... */];
  const statements = code.split(" ");
  const tokens: string[] = [];

  REGEXES.forEach((regex) => {
    statements.forEach((statement) => {
      // tokenize...
    });
  });

  const ast: string[] = [];
  tokens.forEach((token) => {
    // lex...
  });

  ast.forEach((node) => {
    // parse...
  });

  return ast;
}
```

**Good:**
```typescript
type Token = { type: string; value: string };
type AstNode = { kind: string; children: AstNode[] };

function parseCode(code: string): AstNode[] {
  const tokens = tokenize(code);
  const ast = lexer(tokens);
  return ast.map(parseNode);
}

function tokenize(code: string): Token[] {
  // ...
  return [];
}

function lexer(tokens: Token[]): AstNode[] {
  // ...
  return [];
}

function parseNode(node: AstNode): AstNode {
  // ...
  return node;
}
```

### Remove duplicate code

Two functions that differ only in a few fields should be unified. Model the shared shape with a discriminated union and branch only where the data actually differs.

**Bad:**
```typescript
function showDeveloperList(developers: Developer[]): void {
  developers.forEach((developer) => {
    const expectedSalary = developer.calculateExpectedSalary();
    const experience = developer.getExperience();
    const githubLink = developer.getGithubLink();
    render({ expectedSalary, experience, githubLink });
  });
}

function showManagerList(managers: Manager[]): void {
  managers.forEach((manager) => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();
    render({ expectedSalary, experience, portfolio });
  });
}
```

**Good:**
```typescript
interface Employee {
  type: "developer" | "manager";
  calculateExpectedSalary(): number;
  getExperience(): number;
  getExtraDetails(): Record<string, unknown>;
}

function showEmployeeList(employees: Employee[]): void {
  employees.forEach((employee) => {
    render({
      expectedSalary: employee.calculateExpectedSalary(),
      experience: employee.getExperience(),
      ...employee.getExtraDetails(),
    });
  });
}
```

### Set default objects with Object.assign or spread

Instead of imperatively checking each property, declare defaults once and merge the partial config over them. Typing the input as `Partial<T>` documents which fields are optional.

**Bad:**
```typescript
interface MenuConfig {
  title: string;
  body: string;
  buttonText: string;
  cancellable: boolean;
}

function createMenu(config: MenuConfig): void {
  config.title = config.title || "Foo";
  config.body = config.body || "Bar";
  config.buttonText = config.buttonText || "Baz";
  config.cancellable = config.cancellable !== undefined ? config.cancellable : true;
  // ...
}
```

**Good:**
```typescript
interface MenuConfig {
  title: string;
  body: string;
  buttonText: string;
  cancellable: boolean;
}

const DEFAULT_MENU_CONFIG: MenuConfig = {
  title: "Foo",
  body: "Bar",
  buttonText: "Baz",
  cancellable: true,
};

function createMenu(config: Partial<MenuConfig> = {}): void {
  const finalConfig: MenuConfig = { ...DEFAULT_MENU_CONFIG, ...config };
  // ...
}

createMenu({ body: "Custom body" });
```

### Don't use flags as function parameters

A boolean flag means the function does more than one thing. Split it into two clearly named functions, or model the variants with overloads or a discriminated union.

**Bad:**
```typescript
function createFile(name: string, temp: boolean): void {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

**Good:**
```typescript
function createFile(name: string): void {
  fs.create(name);
}

function createTempFile(name: string): void {
  createFile(`./temp/${name}`);
}
```

### Avoid side effects (part 1)

A function that mutates shared state outside its scope is unpredictable. Take input as `readonly` and return a new value instead of writing to a global.

**Bad:**
```typescript
// Global variable mutated by the function — callers can't predict the result type.
let name: string | string[] = "Robert C. Martin";

function splitIntoFirstAndLastName(): void {
  name = (name as string).split(" ");
}

splitIntoFirstAndLastName();
// name is now ["Robert", "C.", "Martin"] — any code expecting a string will break.
```

**Good:**
```typescript
function splitIntoFirstAndLastName(name: string): readonly string[] {
  return name.split(" ");
}

const name = "Robert C. Martin";
const newName = splitIntoFirstAndLastName(name);
```

### Avoid side effects (part 2)

Mutating an array or object passed as an argument creates hidden coupling. Return a new collection instead, and use `ReadonlyArray<T>` to make the immutability a compile-time guarantee.

**Bad:**
```typescript
function addItemToCart(cart: CartItem[], item: CartItem): void {
  cart.push({ item, date: Date.now() });
}
```

**Good:**
```typescript
function addItemToCart(
  cart: ReadonlyArray<CartItem>,
  item: CartItem,
): ReadonlyArray<CartItem> {
  return [...cart, { item, date: Date.now() }];
}
```

### Don't write to global functions

Polluting built-in prototypes can collide with other libraries and future language features. Extend behavior with your own class or a standalone function; reach for module augmentation only when there is no alternative.

**Bad:**
```typescript
declare global {
  interface Array<T> {
    diff(other: T[]): T[];
  }
}

Array.prototype.diff = function <T>(this: T[], other: T[]): T[] {
  return this.filter((el) => !other.includes(el));
};
```

**Good:**
```typescript
function diff<T>(source: ReadonlyArray<T>, other: ReadonlyArray<T>): T[] {
  return source.filter((el) => !other.includes(el));
}

// Or, if you need a fluent API, wrap it in a class:
class SuperArray<T> {
  constructor(private readonly items: ReadonlyArray<T>) {}

  diff(other: ReadonlyArray<T>): T[] {
    return this.items.filter((el) => !other.includes(el));
  }
}
```

### Favor functional programming over imperative

Declarative pipelines using `map`, `filter`, and `reduce` express intent without manual loop bookkeeping or mutation. Generics keep the utilities reusable and type-safe.

**Bad:**
```typescript
const programmerOutput = [
  { name: "Uncle Bobby", linesOfCode: 500 },
  { name: "Suzie Q", linesOfCode: 1500 },
];

let totalOutput = 0;
for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

**Good:**
```typescript
interface Programmer {
  name: string;
  linesOfCode: number;
}

const programmerOutput: Programmer[] = [
  { name: "Uncle Bobby", linesOfCode: 500 },
  { name: "Suzie Q", linesOfCode: 1500 },
];

const totalOutput = programmerOutput.reduce(
  (total, { linesOfCode }) => total + linesOfCode,
  0,
);
```

### Encapsulate conditionals

A complex boolean expression buried in an `if` forces the reader to decode it. Extract it into a named predicate, and type it as a type guard when it narrows a type.

**Bad:**
```typescript
if (fsm.state === "fetching" && isEmpty(listNode)) {
  // ...
}
```

**Good:**
```typescript
function shouldShowSpinner(fsm: { state: string }, listNode: Node[]): boolean {
  return fsm.state === "fetching" && isEmpty(listNode);
}

if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```

When the predicate refines a type, express it as a type guard:

```typescript
function isErrorResponse(response: ApiResponse): response is ErrorResponse {
  return response.status === "error";
}
```

### Avoid negative conditionals

Double negatives are hard to parse. Name predicates positively so call sites read naturally.

**Bad:**
```typescript
function isDOMNodeNotPresent(node: Node | null): boolean {
  return node === null;
}

if (!isDOMNodeNotPresent(someNode)) {
  // ...
}
```

**Good:**
```typescript
function isDOMNodePresent(node: Node | null): node is Node {
  return node !== null;
}

if (isDOMNodePresent(someNode)) {
  // ...
}
```

### Avoid conditionals (use polymorphism)

A `switch` over a type tag tends to reappear across the codebase and grow with every new variant. Replace it with a class hierarchy that resolves behavior polymorphically.

**Bad:**
```typescript
class Airplane {
  private type: string;

  getCruisingAltitude(): number {
    switch (this.type) {
      case "777":
        return this.getMaxAltitude() - this.getPassengerCount();
      case "Air Force One":
        return this.getMaxAltitude();
      case "Cessna":
        return this.getMaxAltitude() - this.getFuelExpenditure();
      default:
        throw new Error("Unknown airplane type.");
    }
  }

  private getMaxAltitude(): number {
    return 0;
  }
  private getPassengerCount(): number {
    return 0;
  }
  private getFuelExpenditure(): number {
    return 0;
  }
}
```

**Good:**
```typescript
abstract class Airplane {
  protected abstract getMaxAltitude(): number;

  abstract getCruisingAltitude(): number;
}

class Boeing777 extends Airplane {
  protected getMaxAltitude(): number {
    return 45000;
  }

  getCruisingAltitude(): number {
    return this.getMaxAltitude() - this.getPassengerCount();
  }

  private getPassengerCount(): number {
    return 400;
  }
}

class Cessna extends Airplane {
  protected getMaxAltitude(): number {
    return 14000;
  }

  getCruisingAltitude(): number {
    return this.getMaxAltitude() - this.getFuelExpenditure();
  }

  private getFuelExpenditure(): number {
    return 2000;
  }
}
```

### Avoid type-checking

Branching on `typeof` or `instanceof` reintroduces the conditionals polymorphism removes. Prefer a shared interface, or a discriminated union with type narrowing when you genuinely have heterogeneous data.

**Bad:**
```typescript
function travelToTexas(vehicle: Bicycle | Car): void {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(currentLocation, new Location("texas"));
  } else if (vehicle instanceof Car) {
    vehicle.drive(currentLocation, new Location("texas"));
  }
}
```

**Good:**
```typescript
interface Vehicle {
  move(from: Location, to: Location): void;
}

class Bicycle implements Vehicle {
  move(from: Location, to: Location): void {
    // pedal...
  }
}

class Car implements Vehicle {
  move(from: Location, to: Location): void {
    // drive...
  }
}

function travelToTexas(vehicle: Vehicle): void {
  vehicle.move(currentLocation, new Location("texas"));
}
```

When the data is plain (not behavior-bearing), use a discriminated union:

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
  }
}
```

### Don't over-optimize

Modern JavaScript engines already optimize hot paths at runtime. Write clear code and let the runtime handle micro-optimizations; only optimize against a real, measured bottleneck.

**Bad:**
```typescript
// Caching `list.length` was needed on old browsers; the engine handles it now.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

**Good:**
```typescript
for (let i = 0; i < list.length; i++) {
  // ...
}
```

### Remove dead code

Unused code is noise that misleads readers and rots over time. Delete it — version control preserves the history if you ever need it. Enable `noUnusedLocals` and `noUnusedParameters` in `tsconfig.json` so the compiler flags it for you.

**Bad:**
```typescript
function oldRequestModule(url: string): void {
  // no longer called anywhere
}

function newRequestModule(url: string): void {
  // ...
}

newRequestModule("/api/data");
```

**Good:**
```typescript
function requestModule(url: string): void {
  // ...
}

requestModule("/api/data");
```

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

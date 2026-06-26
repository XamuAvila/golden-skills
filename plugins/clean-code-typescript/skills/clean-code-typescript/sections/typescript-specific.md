# TypeScript-Specific Clean Code

TypeScript's type system is a powerful tool for clean code, not just a layer of annotations. These patterns leverage the compiler to catch bugs at compile time and make code self-documenting, so that illegal states become unrepresentable rather than runtime surprises. The principles below go beyond standard JavaScript clean code: they are only possible because of the type system.

### Use strict TypeScript configuration

Strict mode turns the compiler into your first line of defense. Enable it explicitly — relying on defaults silently weakens type safety, and a few extra flags catch entire categories of bugs.

**Bad:**
```typescript
// tsconfig.json — permissive, lets bugs through
{
  "compilerOptions": {
    "strict": false
  }
}

// arr[10] is typed as string even when the index is out of bounds
const value = ["a", "b"][10]; // value: string (but actually undefined at runtime)
value.toUpperCase(); // compiles, throws at runtime
```

**Good:**
```typescript
// tsconfig.json — strict baseline
{
  "compilerOptions": {
    "strict": true,                       // enables strictNullChecks, noImplicitAny, strictFunctionTypes, etc.
    "noUncheckedIndexedAccess": true,     // arr[i] is T | undefined, forcing a presence check
    "exactOptionalPropertyTypes": true    // { x?: number } cannot be assigned `undefined` explicitly
  }
}

const value = ["a", "b"][10]; // value: string | undefined
value?.toUpperCase();         // compiler forces you to handle the undefined case
```

What each flag catches:
- `strict: true` — bundles `strictNullChecks` (null/undefined are distinct types), `noImplicitAny` (no untyped values sneaking in), `strictFunctionTypes`, and more.
- `noUncheckedIndexedAccess` — indexing arrays/records yields `T | undefined`, so off-by-one and missing-key bugs are caught at compile time.
- `exactOptionalPropertyTypes` — distinguishes "property absent" from "property set to `undefined`", preventing subtle bugs in optional-property handling.

### Avoid `any` — use `unknown` for truly unknown types

`any` disables the type checker entirely and spreads silently through your code. `unknown` is the type-safe counterpart: it accepts any value but forces you to narrow before use, so the compiler still protects every access.

**Bad:**
```typescript
function parse(input: any): any {
  return JSON.parse(input); // returns any — every downstream access is unchecked
}

const data = parse(rawInput);
data.user.name.toUpperCase(); // compiles even if user/name don't exist — crashes at runtime
```

**Good:**
```typescript
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };

interface ParsedData {
  user: { name: string };
}

class ParseError extends Error {}

function parse(input: unknown): Result<ParsedData, ParseError> {
  if (typeof input !== "string") {
    return { ok: false, error: new ParseError("input must be a string") };
  }

  const raw: unknown = JSON.parse(input);

  // narrow `unknown` before trusting its shape
  if (
    typeof raw === "object" &&
    raw !== null &&
    "user" in raw &&
    typeof (raw as { user: unknown }).user === "object"
  ) {
    return { ok: true, value: raw as ParsedData };
  }

  return { ok: false, error: new ParseError("unexpected shape") };
}
```

### Use discriminated unions instead of string/number enums for state

Modeling state as a flat set of fields lets impossible combinations exist (`status: 'success'` with no `data`). A discriminated union ties each state to exactly the data it carries, and a `never` check guarantees you handled every case.

**Bad:**
```typescript
interface RequestState<T> {
  status: string;     // "loading" | "success" | "error" — but the compiler doesn't know
  data?: T;           // present only on success, but always allowed
  error?: Error;      // present only on error, but always allowed
}

function render(state: RequestState<string>): string {
  if (state.status === "success") {
    return state.data!.toUpperCase(); // `!` because data is optional — unsafe
  }
  return "loading...";                // forgot the "error" case, no compiler warning
}
```

**Good:**
```typescript
type RequestState<T> =
  | { kind: "loading" }
  | { kind: "success"; data: T }
  | { kind: "error"; error: Error };

function render(state: RequestState<string>): string {
  switch (state.kind) {
    case "loading":
      return "loading...";
    case "success":
      return state.data.toUpperCase(); // data is guaranteed present here
    case "error":
      return `failed: ${state.error.message}`;
    default: {
      // compile error if a new variant is added but not handled
      const exhaustive: never = state;
      return exhaustive;
    }
  }
}
```

### Use branded types for type-safe primitives

When `UserId`, `OrderId`, and `email` are all `string`, the compiler can't stop you from passing one where another is expected. Branded (nominal) types attach a phantom tag so each primitive becomes its own distinct type, while staying a plain string at runtime.

**Bad:**
```typescript
function getUser(id: string): User { /* ... */ }
function getOrder(id: string): Order { /* ... */ }

const orderId = "ord_123";
getUser(orderId); // wrong id, but compiles — runtime bug
```

**Good:**
```typescript
type UserId = string & { readonly __brand: unique symbol };

// the only sanctioned way to produce a UserId — validation lives here
function toUserId(raw: string): UserId {
  if (!/^usr_[a-z0-9]+$/.test(raw)) {
    throw new Error(`invalid user id: ${raw}`);
  }
  return raw as UserId;
}

function getUser(id: UserId): User { /* ... */ }

const userId = toUserId("usr_123");
getUser(userId);        // ok
getUser("ord_123");     // compile error: string is not assignable to UserId
```

### Prefer `readonly` by default

Mutable parameters invite accidental side effects, especially with arrays and objects passed by reference. Mark inputs `readonly` so the compiler rejects mutation, and use `as const` to freeze literals into precise, immutable types.

**Bad:**
```typescript
function total(prices: number[]): number {
  prices.sort();              // mutates the caller's array as a side effect
  return prices.reduce((a, b) => a + b, 0);
}

function applyDefaults(config: { retries: number }): void {
  config.retries = 3;         // silently rewrites the caller's object
}
```

**Good:**
```typescript
function total(prices: ReadonlyArray<number>): number {
  // prices.sort();           // compile error: sort does not exist on ReadonlyArray
  return [...prices].reduce((a, b) => a + b, 0);
}

function describe(config: Readonly<{ retries: number }>): string {
  // config.retries = 3;      // compile error: cannot assign to a read-only property
  return `retries: ${config.retries}`;
}

// `as const` produces a readonly tuple with literal types, not number[]
const RGB = [255, 128, 0] as const; // readonly [255, 128, 0]
```

### Use `satisfies` for type-safe object literals

Annotating a variable with `const config: Config` validates the shape but widens the value to `Config`, erasing the literal types you may need later. `satisfies` validates against the type *without* widening, giving you both safety and precise inference.

**Bad:**
```typescript
type Config = Record<string, string | number>;

const config: Config = {
  host: "localhost",
  port: 8080,
};

// host is typed as string | number, so this needs a cast or guard
const upper = config.host.toUpperCase(); // compile error: toUpperCase not on number
```

**Good:**
```typescript
type Config = Record<string, string | number>;

const config = {
  host: "localhost",
  port: 8080,
} satisfies Config;

// shape is still validated against Config, but literal types are preserved
config.host.toUpperCase(); // ok: host is string
config.port.toFixed(0);    // ok: port is number
// config.missing;         // compile error: property does not exist
```

### Use type guards and assertion functions

When narrowing isn't automatic, a blind `as` cast lies to the compiler and hides real bugs. User-defined type guards (`x is T`) and assertion functions (`asserts x is T`) narrow types safely, encoding the runtime check the compiler can trust.

**Bad:**
```typescript
function handle(value: unknown): void {
  const user = value as { name: string }; // unchecked — crashes if shape is wrong
  console.log(user.name.toUpperCase());
}
```

**Good:**
```typescript
interface User {
  name: string;
}

// type guard: returns a boolean that narrows the argument's type
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    typeof (value as { name: unknown }).name === "string"
  );
}

// assertion function: throws if the check fails, narrows for the rest of the scope
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error("value is not a User");
  }
}

function handle(value: unknown): void {
  assertIsUser(value);
  console.log(value.name.toUpperCase()); // value is User from here on
}
```

### Prefer `interface` for object shapes, `type` for unions/intersections

`interface` and `type` overlap, but each has a sweet spot. Use `interface` for object shapes that may be extended or merged; use `type` for unions, intersections, tuples, mapped types, and other compositions interfaces can't express.

**Bad:**
```typescript
// type for a plain object shape — works, but can't be merged and gives worse error messages
type User = {
  id: string;
  name: string;
};

// interface trying to express a union — not possible
// interface Status = "active" | "inactive"; // syntax error
```

**Good:**
```typescript
// interface for object shapes — extendable and mergeable
interface User {
  id: string;
  name: string;
}

interface Admin extends User {
  permissions: readonly string[];
}

// type for unions, intersections, and other compositions
type Status = "active" | "inactive" | "suspended";
type Identified = User & { createdAt: Date };

// declaration merging: an advantage unique to interface — augment a third-party type
interface Window {
  myAppConfig: { version: string };
}
```

### Use `const` type parameters (TS 5.0+)

By default, generic type parameters infer widened types (`string[]` instead of the exact literals). A `const` type parameter tells the compiler to infer the narrowest possible type, preserving literals without forcing callers to write `as const`.

**Bad:**
```typescript
function defineRoutes<T extends readonly string[]>(routes: T): T {
  return routes;
}

const routes = defineRoutes(["/home", "/about"]);
// routes is string[] — literal values are lost
type Route = (typeof routes)[number]; // string, not "/home" | "/about"
```

**Good:**
```typescript
function defineRoutes<const T extends readonly string[]>(routes: T): T {
  return routes;
}

const routes = defineRoutes(["/home", "/about"]);
// routes is readonly ["/home", "/about"] — literals preserved, no `as const` needed
type Route = (typeof routes)[number]; // "/home" | "/about"
```

### Never use `!` (non-null assertion) without justification

The non-null assertion `!` tells the compiler "trust me, this isn't null" — and removes the very check that would catch the bug. Prefer an explicit null check or optional chaining. Reserve `!` for the rare case where you've already guaranteed presence in a way the compiler can't see, and document why.

**Bad:**
```typescript
const element = document.getElementById("title");
element!.textContent = "Hello"; // throws if the element is missing — `!` hid the risk
```

**Good:**
```typescript
const element = document.getElementById("title");

// optional chaining: no-op if the element is absent
element?.setAttribute("aria-hidden", "false");

// or an explicit guard when presence is required
if (!element) {
  throw new Error("#title is required but was not found");
}
element.textContent = "Hello"; // narrowed to non-null by the guard
```

When `!` is acceptable: immediately after a check the compiler can't track across a boundary — for example, a value validated in a prior loop pass or guaranteed by an external invariant. Even then, add a comment explaining the invariant so the next reader understands why the assertion is safe.

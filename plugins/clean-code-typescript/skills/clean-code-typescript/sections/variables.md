# Variables

### Use meaningful and pronounceable variable names

Names should reveal intent and be easy to say out loud. Cryptic abbreviations force the reader to decode them.

**Bad:**
```typescript
const yyyymmdstr = moment().format("YYYY/MM/DD");
```

**Good:**
```typescript
const currentDate = moment().format("YYYY/MM/DD");
```

### Use the same vocabulary for the same type of variable

Pick one word per concept and stick to it. Three different verbs for the same operation make the codebase harder to search and reason about.

**Bad:**
```typescript
function getUserInfo(): User { /* ... */ }
function getClientData(): User { /* ... */ }
function getCustomerRecord(): User { /* ... */ }
```

**Good:**
```typescript
function getUser(): User { /* ... */ }
```

### Use searchable names

Magic numbers and inline literals are invisible to a text search and carry no meaning. Extract them into named constants. Prefer `as const` objects or `enum`s for related sets of values.

**Bad:**
```typescript
// What is 86400000 for?
setTimeout(blastOff, 86400000);
```

**Good:**
```typescript
const MILLISECONDS_PER_DAY = 86_400_000;

setTimeout(blastOff, MILLISECONDS_PER_DAY);

// Group related literals with `as const` for narrow literal types...
const HttpStatus = {
  Ok: 200,
  NotFound: 404,
  ServerError: 500,
} as const;

type HttpStatus = (typeof HttpStatus)[keyof typeof HttpStatus]; // 200 | 404 | 500

// ...or an enum when you want a named, self-documenting type.
enum UserRole {
  Admin = "ADMIN",
  Member = "MEMBER",
  Guest = "GUEST",
}
```

### Use explanatory variables

Pull intermediate values out of complex expressions into well-named variables. Destructuring with type annotations makes the shape of the data explicit.

**Bad:**
```typescript
declare const users: Map<string, number>;

for (const keyValue of users) {
  console.log(`${keyValue[0]} has ${keyValue[1]} points`);
}
```

**Good:**
```typescript
declare const users: Map<string, number>;

for (const [name, points] of users) {
  console.log(`${name} has ${points} points`);
}
```

```typescript
const address = "One Infinite Loop, Cupertino 95014";
const cityZipCodeRegex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/;

// Bad: indexing into the match result by number.
saveCityZipCode(
  address.match(cityZipCodeRegex)![1],
  address.match(cityZipCodeRegex)![2]
);

// Good: name the captured groups through an explanatory variable.
const match: RegExpMatchArray | null = address.match(cityZipCodeRegex);
const city = match?.[1] ?? "";
const zipCode = match?.[2] ?? "";
saveCityZipCode(city, zipCode);
```

### Avoid mental mapping

Explicit is better than implicit. A single-letter loop variable forces the reader to remember what it stands for.

**Bad:**
```typescript
const locations = ["Austin", "New York", "San Francisco"];

locations.forEach((l) => {
  doStuff();
  doSomeOtherStuff();
  // ...
  // Wait, what was `l` again?
  dispatch(l);
});
```

**Good:**
```typescript
const locations = ["Austin", "New York", "San Francisco"];

locations.forEach((location) => {
  doStuff();
  doSomeOtherStuff();
  // ...
  dispatch(location);
});
```

### Don't add unnecessary context

If the enclosing class or object already names the concept, don't repeat it on every property.

**Bad:**
```typescript
interface Car {
  carMake: string;
  carModel: string;
  carColor: string;
}

function paintCar(car: Car, color: string): void {
  car.carColor = color;
}
```

**Good:**
```typescript
interface Car {
  make: string;
  model: string;
  color: string;
}

function paintCar(car: Car, color: string): void {
  car.color = color;
}
```

### Use default arguments instead of short-circuiting

Default parameters are clearer than `||` short-circuiting and avoid a subtle bug: `||` replaces *every* falsy value (`""`, `0`, `false`), while a default parameter only kicks in for `undefined`. When you want to keep other falsy values but still guard against `null`/`undefined`, use the nullish coalescing operator `??`.

**Bad:**
```typescript
function createMicrobrewery(name?: string): void {
  // `""` and other falsy values are silently replaced too.
  const breweryName = name || "Hipster Brew Co.";
  // ...
}
```

**Good:**
```typescript
function createMicrobrewery(name: string = "Hipster Brew Co."): void {
  // Only triggers when `name` is `undefined`.
  // ...
}

// When the value can be `null`/`undefined` but you want to keep
// other falsy values like `0` or `""`, reach for `??`.
function getPort(configuredPort: number | null | undefined): number {
  return configuredPort ?? 8080; // `0` would be preserved by `??` (it is not nullish)
}
```

### Prefer `const` assertions and literal types

Use `as const` to lock values into their narrowest literal types and make objects deeply `readonly`. Mark properties `readonly` to signal and enforce immutability — never mutate existing objects, return new copies instead.

**Bad:**
```typescript
// `direction` is widened to `string`; the object is mutable.
const movement = { direction: "north", speed: 10 };
movement.speed = 20; // allowed — accidental mutation

function move(direction: string): void { /* ... */ }
```

**Good:**
```typescript
// `as const` narrows literals and freezes the shape.
const movement = { direction: "north", speed: 10 } as const;
// movement.speed = 20; // Error: Cannot assign to 'speed', it is read-only.

type Direction = (typeof movement)["direction"]; // "north"

function move(direction: "north" | "south" | "east" | "west"): void {
  /* ... */
}

// `readonly` properties enforce immutability at the type level.
interface Point {
  readonly x: number;
  readonly y: number;
}

function translate(point: Point, dx: number, dy: number): Point {
  // Return a new object rather than mutating the input.
  return { x: point.x + dx, y: point.y + dy };
}
```

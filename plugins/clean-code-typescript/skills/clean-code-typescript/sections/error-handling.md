# Error Handling

Throwing errors is a good thing — it means the runtime successfully caught something going wrong, stopped execution, and gave you a chance to react instead of silently corrupting state. The problem is never that an error was thrown; it's how (or whether) you handle it. In TypeScript you have an extra advantage: the type system can model failures, so you can decide at compile time which errors are *expected* (model them as values) and which are *exceptional* (let them throw).

## Don't ignore caught errors

Doing nothing with a caught error gives you no ability to react to it. Logging it to the console (`console.log`) is barely better — it gets lost in a sea of output and nobody is alerted. If you wrap code in a `try/catch`, you have a plan for when it fails.

In TypeScript, a caught error is typed as `unknown` (with `useUnknownInCatchVariables`, the default under `strict`). You cannot assume it's an `Error` — anything can be thrown. Narrow it before using it.

**Bad:**

```typescript
try {
  functionThatMightThrow();
} catch (error) {
  // Swallowed. Nobody knows this failed.
  console.log(error);
}
```

**Good:**

```typescript
try {
  functionThatMightThrow();
} catch (error) {
  // 1. Type the error and narrow it — you can't trust it's an Error.
  const message = error instanceof Error ? error.message : String(error);

  // 2. Make it loud and actionable.
  console.error(message);

  // 3. Notify the user, if relevant.
  notifyUserOfError(message);

  // 4. Report to a tracking service.
  reportErrorToService(error);
}
```

## Don't ignore rejected promises

The same reasoning applies to async failures. A rejected promise that you "handle" by logging is just as invisible as a swallowed `catch`.

**Bad:**

```typescript
getUser()
  .then((user: User) => {
    save(user);
  })
  .catch((error) => {
    console.log(error);
  });
```

**Good — promise chain:**

```typescript
getUser()
  .then((user: User) => {
    save(user);
  })
  .catch((error: unknown) => {
    const message = error instanceof Error ? error.message : String(error);
    console.error(message);
    notifyUserOfError(message);
    reportErrorToService(error);
  });
```

**Good — `async/await` with `try/catch`:**

```typescript
async function loadUser(): Promise<void> {
  try {
    const user = await getUser();
    save(user);
  } catch (error) {
    const message = error instanceof Error ? error.message : String(error);
    console.error(message);
    notifyUserOfError(message);
    reportErrorToService(error);
  }
}
```

`async/await` flattens the chain and lets you handle both sync and async failures in one place, which is usually easier to read than `.then().catch()`.

## TypeScript-specific: Use custom error classes

Throwing strings (`throw "not found"`) loses the stack trace and forces every caller to guess what went wrong. Define a typed error hierarchy so callers can narrow with `instanceof` and react differently per error kind.

**Bad:**

```typescript
function fetchOrder(id: string): Order {
  if (!isValidId(id)) {
    throw "invalid id"; // no stack trace, no type, no structure
  }
  // ...
}
```

**Good — typed hierarchy:**

```typescript
class AppError extends Error {
  constructor(message: string) {
    super(message);
    // Restore the prototype chain — required when extending built-ins
    // so that `instanceof` works after transpilation to ES5.
    this.name = new.target.name;
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

class NotFoundError extends AppError {
  constructor(public readonly resource: string, public readonly id: string) {
    super(`${resource} with id "${id}" was not found`);
  }
}

class ValidationError extends AppError {
  constructor(public readonly field: string, message: string) {
    super(message);
  }
}
```

Callers narrow with `instanceof` and get full type information on the matched branch:

```typescript
try {
  return fetchOrder(id);
} catch (error) {
  if (error instanceof NotFoundError) {
    // error.resource and error.id are available and typed here.
    return res.status(404).json({ resource: error.resource, id: error.id });
  }
  if (error instanceof ValidationError) {
    return res.status(400).json({ field: error.field, message: error.message });
  }
  throw error; // Unknown error — let it propagate.
}
```

For an exhaustive, type-checked dispatch, a **discriminated union** of error types makes the compiler enforce that you handled every case:

```typescript
type DomainError =
  | { kind: "not_found"; resource: string; id: string }
  | { kind: "validation"; field: string; message: string }
  | { kind: "conflict"; reason: string };

function describe(error: DomainError): string {
  switch (error.kind) {
    case "not_found":
      return `${error.resource} ${error.id} not found`;
    case "validation":
      return `Invalid ${error.field}: ${error.message}`;
    case "conflict":
      return `Conflict: ${error.reason}`;
    default: {
      // If a new variant is added to DomainError and not handled,
      // this line fails to compile.
      const _exhaustive: never = error;
      return _exhaustive;
    }
  }
}
```

## TypeScript-specific: Result pattern (alternative to try/catch)

`try/catch` is great for *exceptional* situations — bugs, infrastructure failures, things you don't expect. But many failures are *expected*: a user not found, a form field that's invalid, a payment declined. For those, throwing hides the failure from the type system — the caller's signature says it returns a `User`, but it might explode. The **Result pattern** makes the failure explicit and type-checked: the function returns either a success or an error value, and the caller *cannot* access the data without first checking which one it got.

**Bad — throwing for an expected failure:**

```typescript
function parseAge(input: string): number {
  const age = Number(input);
  if (Number.isNaN(age) || age < 0) {
    // "Invalid input" is an expected outcome of parsing user input,
    // not an exceptional event. Throwing hides it from the signature.
    throw new Error("invalid age");
  }
  return age;
}
```

**Good — return a `Result<T, E>`:**

```typescript
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

function err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}

function parseAge(input: string): Result<number, string> {
  const age = Number(input);
  if (Number.isNaN(age) || age < 0) {
    return err("Age must be a non-negative number");
  }
  return ok(age);
}
```

The caller is forced by the type narrowing to handle both outcomes — there's no way to read `.value` on the error branch:

```typescript
const result = parseAge(userInput);

if (!result.ok) {
  // result.error is a string here; result.value does not exist.
  showValidationMessage(result.error);
} else {
  // result.value is a number here.
  saveAge(result.value);
}
```

**Rule of thumb:** use `Result<T, E>` for failures that are part of the normal flow and that every caller should consciously handle; keep `throw` / `try/catch` for genuinely exceptional conditions that most callers can't meaningfully recover from and should bubble up.

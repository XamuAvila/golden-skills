# Testing

Testing is more important than shipping. If you ship without tests, every release is a gamble on whether you broke something. Aim for 100% statement and branch coverage — it is the cheapest insurance you can buy against regressions and the only thing that lets you refactor with confidence. With modern tooling there is no excuse for not writing tests.

### One concept per test

A test that asserts several unrelated things at once is hard to name and harder to diagnose: when it fails, you don't know which behavior broke. Give every concept its own test with a name that describes the behavior, so a red test points straight at the broken case.

**Bad:**
```typescript
import { describe, it, expect } from "vitest";
import { addMonths } from "./dates";

describe("addMonths", () => {
  it("handles date boundaries", () => {
    let date = new Date(2021, 0, 1); // 1 Jan 2021

    date = addMonths(date, 1);
    expect(date.getMonth()).toBe(1); // 30-day-ish month

    date = addMonths(date, 1);
    expect(date.getMonth()).toBe(2);

    // leap year, asserted in the same test
    date = new Date(2020, 1, 29);
    date = addMonths(date, 1);
    expect(date.getMonth()).toBe(2);
  });
});
```

**Good:**
```typescript
import { describe, it, expect } from "vitest";
import { addMonths } from "./dates";

describe("addMonths", () => {
  it("handles 30-day months", () => {
    const result = addMonths(new Date(2021, 3, 30), 1); // 30 Apr -> May
    expect(result.getMonth()).toBe(4);
  });

  it("handles 31-day months", () => {
    const result = addMonths(new Date(2021, 0, 31), 1); // 31 Jan -> Feb
    expect(result.getMonth()).toBe(1);
  });

  it("handles leap year", () => {
    const result = addMonths(new Date(2020, 1, 29), 1); // 29 Feb 2020 -> Mar
    expect(result.getMonth()).toBe(2);
  });
});
```

Each `it` block names exactly one scenario. When `handles leap year` goes red, you know precisely what regressed without reading a single assertion.

### Use typed fixtures and `satisfies` to catch test errors at compile time

In TypeScript, your tests are code the compiler checks too. Lean on it: a typed factory keeps fixtures in sync with the production type, so adding a required field to `User` immediately turns every stale fixture into a compile error instead of a confusing runtime failure. Use `satisfies` to validate a literal against a type *without widening it*, so you still get precise inference on the fixture's fields.

**Bad:**
```typescript
import { describe, it, expect } from "vitest";
import { formatUser } from "./users";

describe("formatUser", () => {
  it("formats the full name", () => {
    // Untyped literal: if `User` gains a required `email` field,
    // this object silently drifts out of sync — no compile error,
    // just a surprise at runtime.
    const user: any = { firstName: "Ada", lastName: "Lovelace" };
    expect(formatUser(user)).toBe("Ada Lovelace");
  });
});
```

**Good:**
```typescript
import { describe, it, expect } from "vitest";
import { formatUser, type User } from "./users";

// A typed factory: defaults live in one place and every fixture
// is checked against `User`. A new required field breaks this once,
// not in every test.
function makeUser(overrides: Partial<User> = {}): User {
  return {
    id: "u_1",
    firstName: "Ada",
    lastName: "Lovelace",
    email: "ada@example.com",
    ...overrides,
  };
}

describe("formatUser", () => {
  it("formats the full name", () => {
    const user = makeUser({ firstName: "Grace", lastName: "Hopper" });
    expect(formatUser(user)).toBe("Grace Hopper");
  });

  it("rejects fixtures that don't match the type", () => {
    // `satisfies` validates the shape against `User` while keeping
    // the literal's precise types. A typo like `firstNam` or a wrong
    // value type fails to compile — the bug never reaches the runner.
    const user = {
      id: "u_2",
      firstName: "Edsger",
      lastName: "Dijkstra",
      email: "edsger@example.com",
    } satisfies User;

    expect(formatUser(user)).toBe("Edsger Dijkstra");
  });
});
```

The payoff is that a whole class of test bugs — stale fixtures, typo'd property names, wrong value types — becomes a red squiggle in your editor instead of a flaky red test in CI.

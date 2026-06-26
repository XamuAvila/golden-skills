# Formatting

Formatting is subjective, and arguing about it in pull requests is a waste of everyone's time. Don't do it by hand and don't litigate it in code review — pick an automated tool, agree on a config once, and let the machine enforce it. **Prettier** handles whitespace, quotes, semicolons, and line breaks; **ESLint** (with `@typescript-eslint`) handles structural and naming rules. The points below are the few things tools can't fully decide for you.

## Use consistent capitalization

Capitalization carries meaning. When the same kind of thing is written three different ways, the reader has to stop and wonder whether the inconsistency is significant. It usually isn't — it's just noise. Pick one convention per category and apply it everywhere.

**Bad:**

```typescript
const DAYS_IN_WEEK = 7;
const daysInMonth = 30;

const songs = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
const Artists = ["ACDC", "Led Zeppelin", "The Beatles"];

function eraseDatabase() {}
function restore_database() {}

type animal = { name: string };
interface Container {}
class Alpaca {}
```

**Good:**

```typescript
const DAYS_IN_WEEK = 7;
const DAYS_IN_MONTH = 30;

const SONGS = ["Back In Black", "Stairway to Heaven", "Hey Jude"];
const ARTISTS = ["ACDC", "Led Zeppelin", "The Beatles"];

function eraseDatabase() {}
function restoreDatabase() {}

type Animal = { name: string };
interface Container {}
class Alpaca {}
```

The conventions, summarized:

| Category | Convention | Example |
|----------|-----------|---------|
| Variables / functions | `camelCase` | `daysInMonth`, `restoreDatabase` |
| Boolean variables | `camelCase` with `is` / `has` / `should` / `can` prefix | `isActive`, `hasAccess` |
| Constants (true compile-time constants) | `UPPER_SNAKE_CASE` | `DAYS_IN_WEEK` |
| Classes | `PascalCase` | `Alpaca`, `UserRepository` |
| Interfaces | `PascalCase`, **no `I` prefix** | `Container`, `User` |
| Type aliases | `PascalCase` | `Animal`, `Result` |
| Enums and their members | `PascalCase` | `enum Direction { Up, Down }` |
| Type parameters (generics) | `PascalCase`, short and meaningful | `T`, `TKey`, `TValue` |

### TypeScript-specific notes

- **Don't prefix interfaces with `I`.** `IUser` is a C#/legacy habit; idiomatic TypeScript uses `User`. The consumer of a type shouldn't have to know whether it's backed by an `interface` or a `type`.
- **Enum members are `PascalCase`, not `UPPER_SNAKE`.** Even though they feel constant-like, the `@typescript-eslint` recommended convention treats them like the type they belong to:

  ```typescript
  enum Direction {
    Up,
    Down,
    Left,
    Right,
  }
  ```

- **Reserve `UPPER_SNAKE_CASE` for genuinely fixed values** — module-level configuration, magic numbers given a name. A `let` you reassign, or a function parameter, is not a constant even if it's currently unchanged.

You can encode all of this in `@typescript-eslint/naming-convention` so the linter — not a reviewer — flags violations.

## Callers and callees should be close

When you read a function that calls a helper, the natural next question is "what does that helper do?" Make the answer easy: put the caller directly **above** the callee, so the file reads top-down like a newspaper — the high-level summary first, the details below. Scattering related methods randomly across a class forces the reader to scroll back and forth hunting for definitions.

**Bad:**

```typescript
class PerformanceReview {
  constructor(private readonly employee: Employee) {}

  private lookupPeers(): Employee[] {
    return db.lookup(this.employee, "peers");
  }

  private lookupManager(): Employee {
    return db.lookup(this.employee, "manager");
  }

  private getPeerReviews(): Review[] {
    const peers = this.lookupPeers();
    // ...
  }

  review(): void {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();
  }

  private getManagerReview(): Review {
    const manager = this.lookupManager();
    // ...
  }

  private getSelfReview(): Review {
    // ...
  }
}
```

**Good** — each method appears just before the methods it depends on, so you can read straight down without jumping around:

```typescript
class PerformanceReview {
  constructor(private readonly employee: Employee) {}

  review(): void {
    this.getPeerReviews();
    this.getManagerReview();
    this.getSelfReview();
  }

  private getPeerReviews(): Review[] {
    const peers = this.lookupPeers();
    // ...
  }

  private lookupPeers(): Employee[] {
    return db.lookup(this.employee, "peers");
  }

  private getManagerReview(): Review {
    const manager = this.lookupManager();
    // ...
  }

  private lookupManager(): Employee {
    return db.lookup(this.employee, "manager");
  }

  private getSelfReview(): Review {
    // ...
  }
}
```

The public entry point (`review`) sits at the top; each private helper follows the method that calls it. The reader descends through levels of detail instead of bouncing around the file.

# Comments

Comments are a fallback, not a goal. Good code mostly explains itself through clear names and small functions; a comment is what you reach for when the *why* genuinely can't be expressed in code. Most comments you're tempted to write are either restating what the code already says, or compensating for code that should have been clearer in the first place. Treat every comment as a small admission of failure to express intent in the code — and then keep the few that earn their place.

## Only comment things with business logic complexity

Don't narrate the code. A reader who knows the language can see *what* each line does; what they can't see is *why* you wrote it that way — a non-obvious business rule, a platform quirk, a deliberate bit-twiddle. Comment that.

**Bad:**

```typescript
function hashIt(data: string): number {
  // The hash
  let hash = 0;

  // Length of string
  const length = data.length;

  // Loop through every character in data
  for (let i = 0; i < length; i++) {
    // Get character code
    const char = data.charCodeAt(i);
    // Make the hash
    hash = (hash << 5) - hash + char;
    // Convert to 32-bit integer
    hash &= hash;
  }

  return hash;
}
```

**Good** — every redundant comment removed; the one comment left explains a non-obvious *why*:

```typescript
function hashIt(data: string): number {
  let hash = 0;

  for (let i = 0; i < data.length; i++) {
    const char = data.charCodeAt(i);
    hash = (hash << 5) - hash + char;

    // Force conversion to a 32-bit integer (overflow is intentional).
    hash &= hash;
  }

  return hash;
}
```

## Don't leave commented-out code

Commented-out code is dead weight that confuses readers: Is it important? Will it come back? Is it broken? Nobody knows, so it lingers forever. Your version control system remembers every line you ever wrote — delete it and recover it from history if you ever truly need it.

**Bad:**

```typescript
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

**Good:**

```typescript
doStuff();
```

## Don't have changelog comments

Files don't need a diary. The history of who changed what, and when, lives in `git log` and `git blame` — far more reliably than a hand-maintained comment block that everyone forgets to update. Keep that noise out of the source.

**Bad:**

```typescript
/**
 * 2024-12-20: Removed monads, didn't understand them (RM)
 * 2024-10-01: Improved using special monads (JP)
 * 2024-02-03: Removed type-checking (LI)
 * 2023-03-14: Added combine with type-checking (JR)
 */
function combine(a: number, b: number): number {
  return a + b;
}
```

**Good:**

```typescript
function combine(a: number, b: number): number {
  return a + b;
}
```

If you need the story behind this function, run `git log -p` on the file.

## Avoid positional markers

ASCII-art banners that divide a file into "sections" add visual clutter without adding meaning. If a class or module is big enough that you feel the need to partition it with comment bars, that's a signal to split it into smaller files or types — not to decorate it. Let well-named functions, fields, and the natural indentation provide the structure.

**Bad:**

```typescript
////////////////////////////////////////////////////////////////////////////////
// Scope Model Instantiation
////////////////////////////////////////////////////////////////////////////////
const model: Model = {
  menu: "foo",
  nav: "bar",
};

////////////////////////////////////////////////////////////////////////////////
// Action setup
////////////////////////////////////////////////////////////////////////////////
function actions(): Actions {
  // ...
}
```

**Good:**

```typescript
const model: Model = {
  menu: "foo",
  nav: "bar",
};

function actions(): Actions {
  // ...
}
```

The names `model` and `actions` already tell you what each block is. The banners were never carrying information — only ink.

# Concurrency

### Use Promises, not callbacks

Callbacks force you to nest each asynchronous step inside the previous one, producing the "callback hell" pyramid that is hard to read and harder to handle errors in. A `Promise` is a first-class value with a typed result, so you can chain steps linearly and funnel every failure through a single `.catch()`.

**Bad:**
```typescript
import { get } from "request";
import { writeFile } from "fs";

get(
  "https://en.wikipedia.org/wiki/Robert_Cecil_Martin",
  (requestErr, response, body) => {
    if (requestErr) {
      console.error(requestErr);
    } else {
      writeFile("article.html", body, (writeErr) => {
        if (writeErr) {
          console.error(writeErr);
        } else {
          console.log("File written");
        }
      });
    }
  }
);
```

**Good:**
```typescript
import { get } from "./http"; // returns Promise<string>
import { writeFile } from "fs/promises";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then((body: string): Promise<void> => writeFile("article.html", body))
  .then(() => console.log("File written"))
  .catch((error: unknown) => console.error(error));
```

Typing the function as `Promise<string>` makes the eventual value explicit, and the single `.catch()` covers every step in the chain.

### Async/Await is cleaner than Promises

`.then()` chains still read inside-out for anything non-trivial — intermediate values are trapped inside callbacks. `async`/`await` lets you write asynchronous code as if it were synchronous, with ordinary `try`/`catch` for errors. Annotate the return type so callers know exactly what they're awaiting: `Promise<void>` for a side effect, `Promise<Result>` when a value comes back.

**Bad:**
```typescript
import { get } from "./http"; // returns Promise<string>
import { writeFile } from "fs/promises";

function downloadArticle(): Promise<void> {
  return get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
    .then((body: string) => writeFile("article.html", body))
    .then(() => console.log("File written"))
    .catch((error: unknown) => console.error(error));
}
```

**Good:**
```typescript
import { get } from "./http"; // returns Promise<string>
import { writeFile, readFile } from "fs/promises";

// `Promise<void>`: the function performs a side effect and returns nothing.
async function downloadArticle(): Promise<void> {
  try {
    const body: string = await get(
      "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
    );
    await writeFile("article.html", body);
    console.log("File written");
  } catch (error: unknown) {
    console.error(error);
  }
}

// `Promise<Result>`: the function resolves with a typed value.
interface Article {
  title: string;
  html: string;
}

async function loadArticle(path: string): Promise<Article> {
  const html: string = await readFile(path, "utf8");
  return { title: "Robert Cecil Martin", html };
}
```

The control flow reads top to bottom, intermediate values are plain typed locals, and the `Promise<void>` vs `Promise<Article>` return types document intent at the signature.

### Reach for the right Promise combinator, and type your errors as `unknown`

Awaiting independent operations one after another wastes time — run them in parallel. `Promise.all` resolves to a tuple whose types line up positionally with its inputs, so you get fully typed destructuring for free; but it rejects as soon as *any* input rejects. When you need every result regardless of individual failures, use `Promise.allSettled`. And in `catch` blocks, type the error as `unknown` (never `any`) so the compiler forces you to narrow before using it.

**Bad:**
```typescript
import { fetchUser, fetchPosts } from "./api";

async function loadProfile(id: string) {
  // Sequential: the second request waits for the first for no reason.
  const user = await fetchUser(id);
  const posts = await fetchPosts(id);

  // `any` disables type checking on the error — `err.mesage` typo
  // compiles and blows up at runtime.
  try {
    return { user, posts };
  } catch (err: any) {
    console.error(err.mesage);
  }
}
```

**Good:**
```typescript
import { fetchUser, fetchPosts, type User, type Post } from "./api";

async function loadProfile(id: string): Promise<{ user: User; posts: Post[] }> {
  // `Promise.all` runs both in parallel and infers a tuple
  // `[User, Post[]]`, so destructuring stays fully typed.
  const [user, posts]: [User, Post[]] = await Promise.all([
    fetchUser(id),
    fetchPosts(id),
  ]);

  return { user, posts };
}

// `Promise.allSettled` keeps going even if some operations reject,
// returning a discriminated union per result you can narrow on `status`.
async function notifyAll(userIds: readonly string[]): Promise<void> {
  const results = await Promise.allSettled(
    userIds.map((id) => fetchUser(id))
  );

  for (const result of results) {
    if (result.status === "fulfilled") {
      console.log(`Loaded ${result.value.id}`);
    } else {
      // `result.reason` is `unknown`; narrow before using it.
      const reason: unknown = result.reason;
      const message =
        reason instanceof Error ? reason.message : String(reason);
      console.error(`Failed: ${message}`);
    }
  }
}
```

Use `Promise.all` when a single failure should abort everything (and you want the tuple types), `Promise.allSettled` when partial success is acceptable, and always narrow `unknown` errors with `instanceof` before touching their properties.

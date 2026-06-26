# SOLID Principles

SOLID is a set of five object-oriented design principles that, applied together, make TypeScript code easier to extend, test, and maintain. The examples below show a "Bad" version that violates the principle and a "Good" version that respects it, using TypeScript interfaces, generics, abstract classes, and dependency injection.

## Single Responsibility Principle (SRP)

A class should have only one reason to change. When a class mixes unrelated responsibilities, a change to one concern risks breaking the other, and the class becomes harder to understand and test. Split distinct concerns into separate classes with focused contracts.

**Bad:**

```typescript
// This class has two reasons to change: authentication rules AND settings persistence.
class UserSettings {
  constructor(private readonly user: { name: string; password: string }) {}

  // Responsibility #1: verifying credentials (auth concern)
  verifyCredentials(password: string): boolean {
    return this.user.password === password;
  }

  // Responsibility #2: changing settings (settings concern)
  changeSettings(settings: Record<string, unknown>): void {
    if (this.verifyCredentials(this.user.password)) {
      // ...persist settings
    }
  }
}
```

**Good:**

```typescript
interface Credentials {
  name: string;
  password: string;
}

// Single responsibility: authentication only.
class UserAuth {
  constructor(private readonly credentials: Credentials) {}

  verifyCredentials(password: string): boolean {
    return this.credentials.password === password;
  }
}

// Single responsibility: settings management only.
// It depends on UserAuth instead of re-implementing auth.
class UserSettings {
  constructor(private readonly auth: UserAuth) {}

  changeSettings(settings: Record<string, unknown>): void {
    if (this.auth.verifyCredentials("...")) {
      // ...persist settings
    }
  }
}
```

## Open/Closed Principle (OCP)

Software entities should be open for extension but closed for modification. Adding a new behavior should not require editing existing, tested code. In TypeScript, model the varying behavior behind an interface and let each variant `implement` it, so new variants are new classes rather than new branches.

**Bad:**

```typescript
// Every new adapter forces a modification of fetch() — it is not closed for modification.
class HttpRequester {
  constructor(private readonly adapterName: string) {}

  fetch(url: string): Promise<unknown> {
    if (this.adapterName === "ajax") {
      return this.makeAjaxCall(url);
    } else if (this.adapterName === "node") {
      return this.makeHttpCall(url);
    }
    // Adding "fetch" or "axios" means editing this method again.
    throw new Error(`Unknown adapter: ${this.adapterName}`);
  }

  private makeAjaxCall(url: string): Promise<unknown> {
    return Promise.resolve({ url });
  }

  private makeHttpCall(url: string): Promise<unknown> {
    return Promise.resolve({ url });
  }
}
```

**Good:**

```typescript
interface Adapter {
  request<T>(url: string): Promise<T>;
}

class AjaxAdapter implements Adapter {
  request<T>(url: string): Promise<T> {
    return Promise.resolve({ url } as T);
  }
}

class NodeAdapter implements Adapter {
  request<T>(url: string): Promise<T> {
    return Promise.resolve({ url } as T);
  }
}

// Closed for modification: HttpRequester never changes when a new adapter appears.
class HttpRequester {
  constructor(private readonly adapter: Adapter) {}

  fetch<T>(url: string): Promise<T> {
    return this.adapter.request<T>(url);
  }
}

// Open for extension: add a brand-new adapter without touching anything above.
class AxiosAdapter implements Adapter {
  request<T>(url: string): Promise<T> {
    return Promise.resolve({ url } as T);
  }
}
```

## Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base type without altering the correctness of the program. A classic violation is `Square extends Rectangle`: a square forces width and height to stay equal, which breaks any code that sets them independently through the rectangle contract. Prefer composition under a shared abstraction over inheritance that breaks the parent's contract.

**Bad:**

```typescript
class Rectangle {
  constructor(protected width = 0, protected height = 0) {}

  setWidth(width: number): void {
    this.width = width;
  }

  setHeight(height: number): void {
    this.height = height;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

// A Square is-NOT-substitutable for a Rectangle: setting one side mutates the other.
class Square extends Rectangle {
  setWidth(width: number): void {
    this.width = width;
    this.height = width;
  }

  setHeight(height: number): void {
    this.width = height;
    this.height = height;
  }
}

// Code written against Rectangle silently breaks for Square:
function expectArea20(rectangle: Rectangle): void {
  rectangle.setWidth(5);
  rectangle.setHeight(4);
  console.log(rectangle.getArea()); // Rectangle -> 20, Square -> 16 (wrong)
}
```

**Good:**

```typescript
// A shared abstraction with a single, honest contract: every shape can report its area.
abstract class Shape {
  abstract getArea(): number;
}

class Rectangle extends Shape {
  constructor(
    private readonly width: number,
    private readonly height: number,
  ) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(private readonly length: number) {
    super();
  }

  getArea(): number {
    return this.length * this.length;
  }
}

// Any Shape is freely substitutable: no setter contract to violate.
function totalArea(shapes: Shape[]): number {
  return shapes.reduce((sum, shape) => sum + shape.getArea(), 0);
}
```

## Interface Segregation Principle (ISP)

Clients should not be forced to depend on methods or properties they do not use. A large "fat" interface couples every consumer to fields it may not care about. Split it into smaller, cohesive interfaces and compose them, using `Partial<T>` and optional members for genuinely optional configuration.

**Bad:**

```typescript
// A fat interface: every caller must supply animation settings even when not animating.
interface DOMTraverserSettings {
  rootNode: HTMLElement;
  animationModule: () => void; // not all clients animate
  onAnimationStart: () => void; // forced even when unused
  onAnimationEnd: () => void; // forced even when unused
}

class DOMTraverser {
  constructor(private readonly settings: DOMTraverserSettings) {}

  traverse(): void {
    // Most clients only need rootNode, yet they must pass the animation hooks too.
  }
}
```

**Good:**

```typescript
// Required, minimal contract.
interface TraverserConfig {
  rootNode: HTMLElement;
}

// Optional, cohesive concern kept separate.
interface AnimationConfig {
  animationModule: () => void;
  onAnimationStart: () => void;
  onAnimationEnd: () => void;
}

// Compose them; the animation half is optional via Partial<T>.
type DOMTraverserSettings = TraverserConfig & Partial<AnimationConfig>;

class DOMTraverser {
  constructor(private readonly settings: DOMTraverserSettings) {}

  traverse(): void {
    if (this.settings.animationModule) {
      this.settings.onAnimationStart?.();
      this.settings.animationModule();
      this.settings.onAnimationEnd?.();
    }
  }
}

// A client that only traverses depends on just what it uses (Pick keeps the contract tight):
const config: Pick<DOMTraverserSettings, "rootNode"> = {
  rootNode: document.body,
};
new DOMTraverser(config).traverse();
```

## Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules; both should depend on abstractions. When a class instantiates its own concrete collaborators, it is tightly coupled to them and hard to test. Depend on an interface and inject the implementation through the constructor.

**Bad:**

```typescript
class InventoryRequester {
  fetchItems(): string[] {
    // Hard-wired to a specific transport (e.g. HTTP).
    return ["apples", "bananas"];
  }
}

class InventoryTracker {
  private readonly requester: InventoryRequester;

  constructor() {
    // High-level module creates its own low-level dependency: impossible to swap or mock.
    this.requester = new InventoryRequester();
  }

  requestItems(): string[] {
    return this.requester.fetchItems();
  }
}
```

**Good:**

```typescript
// The abstraction both layers depend on.
interface ItemRequester {
  fetchItems(): string[];
}

class HttpInventoryRequester implements ItemRequester {
  fetchItems(): string[] {
    return ["apples", "bananas"];
  }
}

class InventoryTracker {
  // Depends on the abstraction, injected via the constructor.
  constructor(private readonly requester: ItemRequester) {}

  requestItems(): string[] {
    return this.requester.fetchItems();
  }
}

// Production wiring:
const tracker = new InventoryTracker(new HttpInventoryRequester());

// Test wiring: substitute a fake with no real I/O.
const fakeRequester: ItemRequester = { fetchItems: () => ["test-item"] };
const testTracker = new InventoryTracker(fakeRequester);
```

For larger applications, a dependency-injection container can wire these abstractions automatically instead of constructing them by hand. Libraries like **tsyringe** (decorator-based, framework-agnostic) and the built-in DI of **NestJS** resolve constructor dependencies from registered providers, keeping the inversion while removing the manual wiring boilerplate.

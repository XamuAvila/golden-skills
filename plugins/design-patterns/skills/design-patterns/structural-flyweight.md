# Flyweight

**Also known as:** Cache
**Category:** Structural

## Intent

Lets you fit more objects into available RAM by sharing common parts of state between multiple objects instead of keeping all data in each object.

## Problem

A particle system in a game creates millions of Particle objects, each with position, velocity, color, and sprite. The app crashes because the objects consume too much RAM.

## Solution

Separate **intrinsic state** (shared — color, sprite) from **extrinsic state** (unique — position, velocity). Store intrinsic state in shared Flyweight objects. Pass extrinsic state to flyweight methods as parameters.

## Structure

1. **Flyweight** — contains the intrinsic state (shareable portion). Accepts extrinsic state via method parameters
2. **Context** — contains the extrinsic state (unique per object) and a reference to a flyweight
3. **FlyweightFactory** — manages a pool of existing flyweights. Returns existing or creates new
4. **Client** — calculates or stores extrinsic state, picks a flyweight from the factory

## Pseudocode

```
class TreeType is
    field name: string
    field color: string
    field texture: string

    constructor TreeType(name, color, texture) is // ...

    method draw(canvas, x, y) is
        // 1. Create bitmap from type, name, color, texture
        // 2. Draw bitmap on canvas at (x, y)

class TreeFactory is
    static field treeTypes: TreeType[]

    static method getTreeType(name, color, texture): TreeType is
        type = treeTypes.find(name, color, texture)
        if (type == null) then
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

class Tree is
    field x: int
    field y: int
    field type: TreeType  // reference to shared flyweight

    constructor Tree(x, y, type) is
        this.x = x
        this.y = y
        this.type = type

    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

class Forest is
    field trees: Tree[]

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach tree in trees do
            tree.draw(canvas)
```

## Applicability

- Use only when your program must support a huge number of objects which barely fit into available RAM
- Verify that the object state can be split into intrinsic (shareable) and extrinsic (unique) parts

## Pros and Cons

**Pros:**
- Save lots of RAM when dealing with millions of similar objects

**Cons:**
- Trading RAM for CPU cycles — extrinsic state may need to be recalculated each time
- Code becomes more complicated
- If flyweight objects are mutable, shared state bugs are extremely subtle

## Relations with Other Patterns

- Can share leaf nodes of **Composite** trees as Flyweights to save RAM
- **Flyweight** shows how to make lots of small objects; **Facade** shows how to make a single object representing an entire subsystem
- **Flyweight** resembles **Singleton** but: Flyweight can have many instances with different intrinsic states; Singleton has exactly one mutable instance

# Memento

**Also known as:** Snapshot
**Category:** Behavioral

## Intent

Lets you save and restore the previous state of an object without revealing the details of its implementation.

## Problem

A text editor needs undo functionality. Saving state from outside the editor requires making all internal fields public, which breaks encapsulation. Future editor changes would break all code that accesses those fields.

## Solution

The object itself (originator) produces snapshots of its own state. The snapshot is stored in a Memento object with a limited interface. A Caretaker stores mementos but can't tamper with their contents — it only knows how to store and retrieve them.

## Structure

1. **Originator** — produces snapshots of its own state and can restore from them
2. **Memento** — value object that acts as a snapshot. Typically immutable, populated via constructor once
3. **Caretaker** — knows "when" and "why" to capture and restore the originator's state. Stores mementos in a stack/list

## Pseudocode

```
class Editor is
    field text: string
    field curX, curY, selectionWidth: int

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method createSnapshot(): Snapshot is
        return new Snapshot(this, text, curX, curY, selectionWidth)

class Snapshot is
    field editor: Editor
    field text: string
    field curX, curY, selectionWidth: int

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = curX
        this.curY = curY
        this.selectionWidth = selectionWidth

    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

class Command is
    field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
```

## Applicability

- Use when you want to produce snapshots of an object's state to be able to restore it later
- Use when direct access to the object's fields/getters/setters violates its encapsulation

## Pros and Cons

**Pros:**
- Produce snapshots without violating encapsulation
- Simplify the originator's code by letting the caretaker maintain history

**Cons:**
- App may consume lots of RAM if clients create mementos too often
- Caretakers must track the originator's lifecycle to destroy obsolete mementos
- Most dynamic languages can't guarantee that the memento's state stays untouched

## Relations with Other Patterns

- **Command** + Memento for undo: command operates, memento saves prior state before the operation
- Memento + **Iterator** to capture current iteration state and roll back if needed
- **Prototype** can sometimes be a simpler alternative to Memento

# Command

**Also known as:** Action, Transaction
**Category:** Behavioral

## Intent

Turns a request into a stand-alone object that contains all information about the request. This lets you parameterize methods with different requests, delay or queue a request's execution, and support undoable operations.

## Problem

GUI toolbar with buttons that trigger various operations. Creating subclasses for each button-action combination is impractical. Some operations (Copy, Cut) are triggered from multiple places (button, menu, keyboard shortcut) — duplicating code.

## Solution

Extract each request into a Command object. The command stores the receiver and all parameters needed to perform the action. GUI elements trigger commands; commands delegate to business logic. Commands decouple UI from business logic.

## Structure

1. **Command** — interface declaring the execution method
2. **ConcreteCommand** — implements execute, delegates to a receiver. Stores parameters and backup for undo
3. **Invoker** — triggers commands. Stores a reference to a command, calls it when needed
4. **Receiver** — contains the actual business logic
5. **Client** — creates commands, associates them with receivers, passes to invokers

## Pseudocode

```
abstract class Command is
    field app: Application
    field editor: Editor
    field backup: string

    constructor Command(app, editor) is
        this.app = app
        this.editor = editor

    method saveBackup() is
        backup = editor.text

    method undo() is
        editor.text = backup

    abstract method execute(): bool

class CopyCommand extends Command is
    method execute(): bool is
        app.clipboard = editor.getSelection()
        return false  // doesn't modify editor — no undo needed

class CutCommand extends Command is
    method execute(): bool is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute(): bool is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

class UndoCommand extends Command is
    method execute(): bool is
        app.undo()
        return false

class CommandHistory is
    field history: Command[]
    method push(c: Command) is
        history.add(c)
    method pop(): Command is
        return history.removeLast()

class Application is
    field clipboard: string
    field editors: Editor[]
    field activeEditor: Editor
    field history: CommandHistory

    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()
```

## Applicability

- Use when you want to parameterize objects with operations
- Use when you want to queue operations, schedule their execution, or execute them remotely
- Use when you want to implement reversible operations (undo/redo)

## Pros and Cons

**Pros:**
- Single Responsibility Principle — decouple invocation from execution
- Open/Closed Principle — new commands without changing existing code
- Implement undo/redo
- Implement deferred execution
- Assemble simple commands into complex ones

**Cons:**
- Code may become more complicated — a new layer between senders and receivers

## Relations with Other Patterns

- **Chain of Responsibility**, Command, **Mediator**, **Observer** all address different ways of connecting senders and receivers
- **Command** + **Memento** for undo: command executes, memento saves prior state
- **Command** and **Strategy** may look similar but differ: Strategy = different ways to do the same thing; Command = any operation turned into object
- **Prototype** can help save copies of Commands for history

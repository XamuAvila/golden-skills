# Mediator

**Also known as:** Intermediary, Controller
**Category:** Behavioral

## Intent

Lets you reduce chaotic dependencies between objects. The pattern restricts direct communications between objects and forces them to collaborate only via a mediator object.

## Problem

A profile editing form has many interdependent elements (checkboxes, text fields, buttons). Changing one element affects several others. Direct coupling between elements makes them impossible to reuse in other forms.

## Solution

Elements communicate only through the mediator (e.g., the dialog itself). When an element changes, it notifies the mediator. The mediator knows which other elements to update and how.

## Structure

1. **Component** — various UI elements or business objects. Each stores a reference to the mediator
2. **Mediator** — interface declaring the notification method
3. **ConcreteMediator** — encapsulates relations between components. Often stores references to all components it manages

## Pseudocode

```
interface Mediator is
    method notify(sender, event)

class AuthenticationDialog implements Mediator is
    field title: string
    field loginCheckbox: Checkbox
    field loginUsername, loginPassword: Textbox
    field regUsername, regPassword, regEmail: Textbox
    field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Create all components
        // Register self as mediator in each component

    method notify(sender, event) is
        if (sender == loginCheckbox and event == "check")
            if (loginCheckbox.checked)
                title = "Log in"
                // Show login fields, hide registration fields
            else
                title = "Register"
                // Show registration fields, hide login fields

        if (sender == okBtn and event == "click")
            if (loginCheckbox.checked)
                found = // try login with credentials
                if (!found)
                    // Show error over login field
            else
                // Create account with registration data
                // Log in with new account

class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, "click")

    method keypress() is
        dialog.notify(this, "keypress")

class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, "check")
```

## Applicability

- Use when it's hard to change classes because they're tightly coupled to many other classes
- Use when you can't reuse a component because it's too dependent on other components
- Use when you're creating tons of component subclasses just to reuse basic behavior in various contexts

## Pros and Cons

**Pros:**
- Single Responsibility Principle — extract communication into one place
- Open/Closed Principle — new mediators without changing components
- Reduce coupling between components
- Reuse individual components more easily

**Cons:**
- Over time, a mediator can evolve into a God Object

## Relations with Other Patterns

- **Chain of Responsibility**, **Command**, Mediator, **Observer** all address sender-receiver decoupling
- **Facade** and Mediator are similar: Facade = simplified unidirectional entry point; Mediator = centralized bidirectional communication
- **Observer** distributes communication; Mediator centralizes it

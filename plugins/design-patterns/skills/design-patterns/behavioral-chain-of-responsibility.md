# Chain of Responsibility

**Also known as:** CoR, Chain of Command
**Category:** Behavioral

## Intent
Chain of Responsibility lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

## Problem
You're building an online ordering system. You want to restrict access so only authenticated users can create orders, and users with administrative permissions get full access to all orders. After a while you realize these checks must be performed sequentially: the system attempts to authenticate the user, and if that fails, there's no point continuing. Over the next months you add more checks: validating raw data before it reaches the system, filtering repeated failed requests from the same IP, and caching results of repeated requests.

Each check, implemented as a sequential block of `if` statements inside one giant function, grew the code into a bloated, rigid mess. Tweaking one check affected the others. Worse, when you wanted to reuse some checks in another part of the system, you had to duplicate code because those components only needed a subset of the checks. The system became very hard to comprehend and expensive to maintain.

## Solution
Like many other behavioral patterns, Chain of Responsibility transforms particular behaviors into stand-alone objects called *handlers*. Each check should be extracted into its own class with a single method that performs the check. The request and its data are passed to this method as arguments.

The pattern suggests that you link these handlers into a chain. Each linked handler has a field for storing a reference to the next handler. In addition to processing a request, handlers pass the request further along the chain. The request travels along the chain until all handlers have had a chance to process it.

Crucially, a handler can decide *not* to pass the request further down the chain — effectively stopping any further processing. For example, if the authentication handler fails, none of the subsequent checks run.

A slightly different but canonical approach: a handler, upon receiving a request, decides whether it can process it. If it can, it doesn't pass the request any further. So it's either one handler that processes the request or none at all. This variant is common when dealing with events in stacks of elements within a graphical user interface.

## Structure
1. **Handler** — declares the interface common to all concrete handlers. It usually contains a single method for handling requests, and sometimes a method for setting the next handler on the chain.
2. **Base Handler** — an optional class where you can put boilerplate code common to all handler classes. Usually it defines a field for storing a reference to the next handler. Clients can build a chain by passing a handler to the constructor or setter of the previous one. The class may also implement the default handling behavior: pass execution to the next handler after checking for its existence.
3. **Concrete Handlers** — contain the actual code for processing requests. Each handler decides whether to process the request and whether to pass it along the chain. Handlers are usually self-contained and immutable, accepting all necessary data once via the constructor.
4. **Client** — composes chains just once or composes them dynamically, depending on the application's logic. A request can be sent to any handler in the chain, not necessarily the first one.

## Pseudocode
This example shows a help system for a graphical interface. The components of the GUI (buttons, panels, dialogs) form a hierarchy. Each component can show contextual help text. If a component has no help text assigned, it forwards the request to its containing component (its parent). The request travels up the containment hierarchy until a component handles it or the chain ends.

```
// The handler interface declares a method for executing a request.
interface ComponentWithContextualHelp is
    method showHelp()

// The base class for simple components.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // The component's container acts as the next link in the
    // chain of handlers.
    protected field container: Container

    // The component shows a tooltip if there's help text assigned
    // to it. Otherwise it forwards the call to the container, if
    // it exists.
    method showHelp() is
        if (tooltipText != null)
            // Show tooltip.
        else
            container.showHelp()

// Containers can contain both simple components and other
// containers as children. The chain relationships are established
// here. The class inherits the showHelp behavior from its parent.
abstract class Container extends Component is
    protected field children: Component[]

    method add(child) is
        children.add(child)
        child.container = this

// Primitive components may be fine with the default help behavior.
class Button extends Component is
    // ...

// But complex components may override the default behavior. If the
// help text can't be provided in a new way, the component can
// always call the base implementation (see Component class).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Show a modal window with the help text.
        else
            super.showHelp()

// ...same as above...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Open the wiki help page.
        else
            super.showHelp()

// Client code.
class Application is
    // Every application configures the chain differently.
    method createUI() is
        dialog = new Dialog("Budget Reports")
        dialog.wikiPageURL = "http://..."

        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = "This panel does..."

        ok = new Button(250, 760, 50, 20, "OK")
        ok.tooltipText = "This is an OK button that..."

        cancel = new Button(320, 760, 50, 20, "Cancel")
        // ...

        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imagine what happens here.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()
```

## Applicability
- Use the Chain of Responsibility pattern when your program is expected to **process different kinds of requests in various ways**, but the exact types of requests and their sequences are unknown beforehand. The pattern lets you link several handlers into one chain and, upon receiving a request, ask each handler whether it can process it. This way all handlers get a chance to process the request.
- Use the pattern when it's essential to **execute several handlers in a particular order**. Since you can link the handlers in the chain in any order, all requests will get through the chain exactly as you planned.
- Use the CoR pattern when the **set of handlers and their order are supposed to change at runtime**. If you provide setters for a reference field inside the handler classes, you'll be able to insert, remove, or reorder handlers dynamically.

## How to Implement
1. Declare the handler interface and describe the signature of a method for handling requests. Decide how the client will pass the request data into the method — the most flexible way is to convert the request into an object and pass it as an argument.
2. To eliminate duplicate boilerplate code in concrete handlers, it might be worth creating an abstract base handler class derived from the handler interface. This class should have a field for storing a reference to the next handler. Make the class immutable; if you need to change the next handler dynamically, define a setter.
3. One by one, create concrete handler subclasses and implement their handling methods. Each handler should make two decisions when receiving a request: whether it'll process the request, and whether it'll pass the request along the chain.
4. The client may either assemble chains on its own or receive pre-built chains from other objects. In the latter case, you must implement some factory classes to build chains according to the configuration or environment settings.
5. The client may trigger any handler in the chain, not just the first one. The request will be passed along the chain until some handler refuses to pass it further or until it reaches the end.
6. Due to the dynamic nature of the chain, the client should be ready to handle these scenarios: the chain may consist of a single link, some requests may not reach the end of the chain, and others may reach the end unhandled.

## Pros and Cons
**Pros:**
- You can control the order of request handling.
- *Single Responsibility Principle.* You can decouple classes that invoke operations from classes that perform operations.
- *Open/Closed Principle.* You can introduce new handlers into the app without breaking the existing client code.

**Cons:**
- Some requests may end up unhandled.

## Relations with Other Patterns
- Chain of Responsibility is often used in conjunction with **Composite**. In this case, when a leaf component gets a request, it may pass it through the chain of all the parent components down to the root of the object tree.
- Handlers in Chain of Responsibility can be implemented as **Commands**. In this case, you can execute a lot of different operations over the same context object, represented by a request.
- Chain of Responsibility and **Decorator** have very similar class structures. Both rely on recursive composition to pass the execution through a series of objects. However, there are several crucial differences. CoR handlers can execute arbitrary operations independently of each other and can stop passing the request further at any point. Decorators can extend the object's behavior while keeping it consistent with the base interface, and they aren't allowed to break the flow of the request.
- Chain of Responsibility, **Command**, **Mediator**, and **Observer** address various ways of connecting senders and receivers of requests. CoR passes a request sequentially along a dynamic chain of potential receivers until one of them handles it. Command establishes unidirectional connections between senders and receivers. Mediator eliminates direct connections, forcing them to communicate via a mediator object. Observer lets receivers dynamically subscribe and unsubscribe from receiving requests.

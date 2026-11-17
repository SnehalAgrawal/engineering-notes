# Behavioral patterns

## Overview

Behavioral patterns are about how objects talk to each other: who does what, and how requests and data move between them.

* **Decoupling sender and receiver**: the object that asks shouldn't need to know who handles the request.
* **Changing behavior at runtime**: swap an algorithm without editing the class that uses it.

## Key concepts

### Observer

A subject keeps a list of subscribers and tells each one when something happens. It doesn't know what they do with the news.

Bad: `Store` must change every time a new service wants to know about sales.

```javascript
class Store {
  saleStarted() {
    new EmailService().send();
    new SMSService().send();
  }
}
```

Better:

```javascript
class Store {
  constructor() { this.observers = []; }

  subscribe(obs) {
    this.observers.push(obs);
    return () => { this.observers = this.observers.filter(o => o !== obs); }; // unsubscribe
  }

  notify(event) { this.observers.forEach(obs => obs.update(event)); }
}
```

Node's `EventEmitter` and the DOM's `addEventListener` are built-in versions. In Java or C#, an event bus often plays this role.

### Strategy

Each version of an algorithm sits behind the same interface. The caller picks one at runtime.

Bad: each new discount type edits this function.

```javascript
function getPrice(price, type) {
  if (type === "member") return price * 0.9;
  if (type === "guest") return price;
}
```

Better: in JavaScript, strategies are often plain functions.

```javascript
const discounts = {
  member: (price) => price * 0.9,
  guest: (price) => price,
};

discounts["member"](100); // 90
```

In Java or C#, each strategy is a class that implements a `DiscountStrategy` interface.

### State

An object hands its behavior to a state object. Swap the state object and the behavior changes.

Bad: every method fills up with checks like `if (this.state === "shipped") ...`.

Better:

```python
class PendingState:
    def cancel(self, order):
        order.state = CancelledState()

class ShippedState:
    def cancel(self, order):
        raise Exception("Cannot cancel a shipped order")

class CancelledState:
    def cancel(self, order):
        pass  # already cancelled

class Order:
    def __init__(self):
        self.state = PendingState()

    def cancel(self):
        self.state.cancel(self)
```

### Command

Wraps a request (the action plus its data) in an object. Now you can queue it, log it, retry it, or undo it.

```javascript
class AddTextCommand {
  constructor(doc, text) {
    this.doc = doc;
    this.text = text;
  }
  execute() { this.doc.content += this.text; }
  undo() { this.doc.content = this.doc.content.slice(0, -this.text.length); }
}

const doc = { content: "" };
const history = [];

function run(cmd) {
  cmd.execute();
  history.push(cmd);
}

run(new AddTextCommand(doc, "Hello"));
run(new AddTextCommand(doc, " world"));
history.pop().undo(); // doc.content is "Hello"
```

In JavaScript a command can be just an object with two functions, `{ execute, undo }`. Java and C# usually use a `Command` interface with one class per command.

### Chain of Responsibility

A request passes through a list of handlers in order. Each handler either deals with it or passes it on. Express middleware, where each function calls `next()`, is the usual example.

### Mediator

Objects talk to one central mediator instead of to each other. Many-to-many links become one-to-many.

```javascript
class ChatRoom {
  constructor() { this.users = []; }

  register(user) {
    user.room = this;
    this.users.push(user);
  }

  send(from, msg) {
    this.users.forEach(u => u !== from && u.receive(msg));
  }
}
```

Common in forms where several fields react to each other, in chat systems, and in games.

## Real-world usage

* **Frontend stores** (Observer): Redux and Vuex notify the views that subscribed.
* **Payment providers** (Strategy): switch between Stripe and PayPal.
* **Undo/redo** (Command): editors keep a stack of commands.
* **Middleware** (Chain of Responsibility): Express and ASP.NET Core pipelines.
* **Order lifecycle** (State): Pending, Paid, Shipped.

## Tradeoffs

* **Observer**: subscribers that never unsubscribe stay in memory (a memory leak).
* **Command**: a class for every small action adds a lot of code.
* **Strategy vs. State**: same structure. The client picks the Strategy. In State, the state objects switch to the next state themselves.
* **Mediator**: can grow into a God Object that knows everything.

## When not to use

* **Chain of Responsibility**: in a long chain it is hard to tell which handler changed or dropped a request.
* **Strategy**: for two options that never change, a simple `if` is enough.

## Interview focus

* **Long `if/else` for shipping cost**: replace it with Strategy.
* **Decouple `CheckoutService` from email and analytics**: Observer, or events.
* **Document approval where allowed actions depend on status**: State.

## Common mistakes

* **Hardcoded chain**: each handler names the next one, instead of the client building the chain.
* **Observer chains**: observers that trigger other observers make side effects hard to follow.
* **Strategy for a value**: if only a number changes, pass the number, not a strategy.

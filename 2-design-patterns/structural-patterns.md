# Structural Design Patterns

### 1. Overview

Structural patterns describe how to put classes and objects together into bigger parts, so a change in one part doesn't spread to the rest.

* **Fitting interfaces together**: make two pieces work together when neither was built for the other.
* **Composition over inheritance**: add features by wrapping an object, not by subclassing it.

### 2. Key Concepts

#### 2.1. Adapter

A wrapper that lets a client use an object through the interface the client expects. Like a travel plug adapter.

Bad: the processor expects an object, but the legacy system sends a string.

```javascript
class PaymentProcessor {
  process(payment) {
    console.log(`Processing payment for user ${payment.userId}`);
  }
}

new PaymentProcessor().process("userId=42"); // prints "user undefined"
```

Better: the adapter takes the old format and calls the new interface.

```javascript
class LegacyPaymentAdapter {
  constructor(processor) { this.processor = processor; }

  process(legacyString) {
    const [key, value] = legacyString.split("=");
    this.processor.process({ [key]: value });
  }
}

new LegacyPaymentAdapter(new PaymentProcessor()).process("userId=42");
```

Neither side changes. Only the adapter knows both formats.

#### 2.2. Decorator

A wrapper with the same interface as the object inside. It adds behavior, then passes the call on. You can stack wrappers in any order at runtime.

Bad: one subclass per combination (`EmailNotifier`, `SMSNotifier`, `EmailSMSNotifier`, ...).

Better:

```javascript
class Notifier {
  send(msg) { console.log(msg); }
}

class EmailDecorator {
  constructor(inner) { this.inner = inner; }
  send(msg) {
    this.inner.send(msg);
    console.log("Email:", msg);
  }
}

class SMSDecorator {
  constructor(inner) { this.inner = inner; }
  send(msg) {
    this.inner.send(msg);
    console.log("SMS:", msg);
  }
}

new SMSDecorator(new EmailDecorator(new Notifier())).send("Hello!");
```

For plain functions, a higher-order function (a function that wraps another function) does the same job. Python's `@decorator` syntax uses this idea.

```javascript
const withLogging = (fn) => (...args) => {
  console.log("Args:", args);
  return fn(...args);
};
```

#### 2.3. Facade

One simple entry point in front of several classes that would otherwise have to be called in the right order by every client.

```python
class OrderFacade:
    def __init__(self):
        self.inventory = Inventory()
        self.payment = Payment()
        self.shipping = Shipping()

    def place_order(self, item, amount):
        self.inventory.check(item)
        self.payment.pay(amount)
        self.shipping.ship(item)
```

Callers use `OrderFacade().place_order("Laptop", 1000)` and never touch the three classes.

#### 2.4. Proxy

A stand-in with the same interface as the real object. It controls access to it: load it later, check permissions, cache results, or call it over the network.

Example: load a video only when someone first presses play.

```javascript
class VideoProxy {
  constructor(filename) {
    this.filename = filename;
    this.real = null;
  }

  play() {
    if (!this.real) this.real = new RealVideo(this.filename); // loads on first use
    this.real.play();
  }
}
```

JavaScript also has a built-in `Proxy` object. It can intercept property reads and writes on any object, which is handy for logging or validation without a wrapper class.

```javascript
const proxy = new Proxy({ msg: "hello" }, {
  get(obj, prop) {
    console.log(`Accessing ${String(prop)}`);
    return obj[prop];
  },
});
```

### 3. Real-World Usage

* **Third-party APIs** (Adapter): map an old SOAP API onto your own types.
* **Function wrappers** (Decorator): Python decorators and JS higher-order functions add logging, auth, or retries.
* **BFF, backend-for-frontend** (Facade): one endpoint combines several microservice calls for a mobile app.
* **Lazy loading** (Proxy): images that load when scrolled into view; ORM relations that load on first access.

### 4. Tradeoffs

* **Decorator**: many nested wrappers make stack traces hard to read.
* **Adapter**: mapping code is one more layer to maintain.
* **Facade**: can hide too much and block advanced features of what it wraps.

### 5. When NOT to Use

* **Decorator**: for one or two fixed variations, a subclass or an `if` is simpler.
* **Adapter**: if you own both sides, change one to match instead.

### 6. Interview Focus

* **Decorator vs. inheritance**: decorators combine at runtime; subclasses need one class per combination (for example, encryption plus compression).
* **Messy third-party library**: wrap it in an Adapter or Facade so its API doesn't spread through your code.
* **Add caching or access checks without changing a class**: use a Proxy.

### 7. Common Mistakes

* **Over-adapting**: adding Adapters for interfaces that already match.
* **Leaky wrappers**: a Proxy or Facade that still makes callers handle errors and types from the hidden classes.
* **Mixing up Proxy and Decorator**: same shape (a wrapper with the same interface), different purpose. Decorator adds behavior. Proxy controls access.

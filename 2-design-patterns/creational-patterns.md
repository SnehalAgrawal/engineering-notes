# Creational Design Patterns

### 1. Overview

Creational patterns control how objects get created. Instead of calling `new ConcreteClass()` all over the code, creation moves behind a function, a class, or a builder.

* **Decoupling**: callers care what an object does, not how it is built.
* **Flexibility**: you can change which class gets created without touching the callers.

Patterns work in any language, but each language has shortcuts. In JavaScript, closures, plain functions, and modules make several of these patterns much smaller than in Java or C#.

### 2. Key Concepts

#### 2.1. Singleton

Only one instance of a class exists, and all code reaches that same instance.

Bad: each `new` opens another connection.

```javascript
class Database {
  constructor() {
    this.connection = Math.random();
  }
}

const db1 = new Database();
const db2 = new Database(); // different connection
```

Better:

```python
class Database:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.connection = "connected"
        return cls._instance
```

In JavaScript, an ES module runs once and is cached. So `export const db = new Database();` is already one instance per process. Older code used the module pattern, a closure with private state:

```javascript
const Counter = (() => {
  let count = 0; // private
  return {
    increment: () => ++count,
    reset: () => (count = 0),
  };
})();
```

#### 2.2. Factory

Moves the choice of which class to create out of the calling code. Two versions:

* **Simple factory**: one function or class returns the right type for an input.
* **Factory Method** (from the GoF book): a base class calls a `create...()` method, and each subclass decides what it returns.

Bad: the caller picks the class, so a new channel (say, push) means editing every caller like this one.

```javascript
function notify(type, msg) {
  let notifier;
  if (type === "email") notifier = new EmailNotification();
  else if (type === "sms") notifier = new SMSNotification();
  notifier.send(msg);
}
```

Better: a simple factory with a registry.

```javascript
class NotificationFactory {
  constructor() { this.creators = {}; }

  register(type, creator) { this.creators[type] = creator; }

  create(type) {
    const Creator = this.creators[type];
    if (!Creator) throw new Error(`Unknown type: ${type}`);
    return new Creator();
  }
}

const factory = new NotificationFactory();
factory.register("email", EmailNotification);
factory.register("sms", SMSNotification);
factory.create("sms").send("Hi!");
```

Factory Method: the subclass decides.

```javascript
class Notifier {
  createChannel() { throw new Error("Not implemented"); }
  notify(msg) { this.createChannel().send(msg); }
}

class EmailNotifier extends Notifier {
  createChannel() { return new EmailNotification(); }
}
```

In JavaScript a factory is often just a function: `const createUser = (name) => ({ name, role: "user" });`. A function can also return a pre-configured function: `const multiplier = (factor) => (n) => n * factor;`.

#### 2.3. Builder

Builds a complex object step by step.

Bad: a long list of positional arguments. A call like `new Pizza(12, true, false, true, true, false)` is hard to read and easy to get wrong.

Better:

```javascript
class Pizza {
  constructor(b) {
    this.size = b.size;
    this.cheese = b.cheese ?? false;
    this.pepperoni = b.pepperoni ?? false;
  }
}

class PizzaBuilder {
  constructor(size) { this.size = size; }
  addCheese() { this.cheese = true; return this; }
  addPepperoni() { this.pepperoni = true; return this; }
  build() {
    if (!this.size) throw new Error("Size is required"); // check before building
    return new Pizza(this);
  }
}

const pizza = new PizzaBuilder(12).addCheese().addPepperoni().build();
```

#### 2.4. Abstract Factory

One factory creates a whole family of related objects. The client never names concrete classes, so it can't mix families by mistake.

Example: a UI factory makes a button and a checkbox, both in macOS style or both in Windows style.

```javascript
class MacFactory {
  createButton() { return new MacButton(); }
  createCheckbox() { return new MacCheckbox(); }
}

class WindowsFactory {
  createButton() { return new WinButton(); }
  createCheckbox() { return new WinCheckbox(); }
}

function renderUI(factory) {
  factory.createButton().render();
  factory.createCheckbox().render();
}

renderUI(new MacFactory()); // switch to WindowsFactory in one place
```

#### 2.5. Prototype

Creates a new object by copying an existing one. Useful when building from scratch is expensive, for example when it needs a slow DB query.

This is not the same as JavaScript's prototype chain, which is how JS does inheritance.

```javascript
class User {
  constructor(name, preferences) {
    this.name = name;
    this.preferences = preferences;
  }

  clone() {
    // Deep copy, so nested objects aren't shared.
    // structuredClone keeps Dates, Maps, and Sets; JSON.parse(JSON.stringify(...)) does not.
    return new User(this.name, structuredClone(this.preferences));
  }
}

const original = new User("Alice", { theme: "dark" });
const copy = original.clone();
copy.preferences.theme = "light";
original.preferences.theme; // still "dark"
```

In Python, `copy.deepcopy(obj)` does the same.

### 3. Real-World Usage

* **Database connections**: one shared connection pool per process, usually handed out by a DI container.
* **UI themes**: Abstract Factory for component families (for example Material vs. iOS styling).
* **Query builders**: Builder for SQL, as in Knex.js or SQLAlchemy Core.
* **Logging**: a factory returns a console, file, or cloud logger based on config.

### 4. Tradeoffs

* **Singleton**: hidden global state, hard to reset between tests. Most teams let a DI container keep one instance instead.
* **Factory**: many small classes can be too much for a simple app.
* **Builder**: an extra class for each complex type.

### 5. When NOT to Use

* **Singleton**: when state must be separate per test, per customer, or per request.
* **Factory**: for a plain `new User()` with no choice to make.

### 6. Interview Focus

* **Singleton vs. DI**: a DI container can still give you one instance. But callers receive it as a parameter, so tests can pass a fake.
* **Builder vs. options object**: in JavaScript, `new Car({ engine: "V8", color: "red" })` is usually enough. Use a Builder when building needs steps, checks across fields (like "engine is required"), or should return an object that can't change later.
* **Abstract Factory**: keeps a family consistent, for example all dark-mode components come from one factory.

### 7. Common Mistakes

* **Singleton not thread-safe**: in multi-threaded languages (Java, C#, C++), two threads can create the instance at the same time. Create it eagerly, use a lock, or use a language idiom (Java's holder class or enum). Single-threaded JavaScript doesn't have this problem.
* **Shallow copy in Prototype**: the clone still shares nested objects with the original.
* **One giant factory**: a factory that builds everything breaks SRP.

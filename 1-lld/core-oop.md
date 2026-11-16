# Core Object-Oriented Programming (OOP)

### 1. Overview

OOP organizes code into objects that hold data and the functions that work on that data. For senior engineers, OOP is about managing complexity: hide details behind clear boundaries and keep parts independent.

### 2. Key Concepts

* **Encapsulation**: keep data private and change it only through methods. Rules like "balance never goes negative" then live in one place.

  ```javascript
  class BankAccount {
    #balance; // private field, enforced by JavaScript

    constructor(balance) { this.#balance = balance; }

    deposit(amount) {
      if (amount > 0) this.#balance += amount;
    }

    getBalance() { return this.#balance; }
  }
  ```

  Without it, anyone can write `account.balance = -50`. In Python, `self._balance` (one underscore) only means "internal" by convention. Python does not enforce it.

* **Abstraction**: show what an object does, hide how. Callers use a simple method and don't need the details.

  ```python
  from abc import ABC, abstractmethod

  class PaymentProcessor(ABC):
      @abstractmethod
      def process_payment(self, amount): ...

  class StripeProcessor(PaymentProcessor):
      def process_payment(self, amount):
          print(f"Processing {amount} via Stripe")
  ```

  JavaScript has no built-in abstract classes or interfaces (TypeScript adds both). A common workaround: the base class method throws `new Error("Must implement")`, and subclasses override it. If the base class also runs shared steps (validate, log) around one overridable step, that is the Template Method pattern.

* **Interface vs. abstract class**:
  * Interface: a contract listing methods, usually with no state. A class can implement many. (Java 8+ allows default method bodies.)
  * Abstract class: a partial implementation that can hold state and shared code. Most languages allow extending only one.

* **Inheritance**: a class reuses and specializes a parent class ("is-a"). Example: `UserService extends BaseService` and gets the shared `db` connection.

* **Polymorphism**: code written for a common type works with any compatible object, and each object brings its own behavior.

  ```javascript
  class CloudLogger { log(msg) { console.log(`Cloud: ${msg}`); } }
  class LocalLogger { log(msg) { console.log(`Local: ${msg}`); } }

  function notifyAdmin(logger, msg) {
    logger.log(msg); // works with either logger
  }
  ```

  Usually this is runtime polymorphism (method overriding). Overloading (same name, different parameters) is the compile-time kind, in languages that have it. JavaScript and Python use duck typing: any object with the right method works. In JavaScript, a method call looks on the object first, then up the prototype chain (child class, then parent class).

* **Composition**: build an object from other objects it holds ("has-a"). Usually more flexible than inheritance, because you can swap parts and test them alone.

  ```python
  class CreateUserUseCase:
      def __init__(self, validator, repository):
          self.validator = validator
          self.repository = repository

      def execute(self, data):
          if self.validator.validate(data):
              self.repository.save(data)
  ```

### 3. Real-World Usage

* **Middleware**: frameworks treat every middleware the same through one signature: `(req, res, next)` in Express, `(context, next)` in ASP.NET Core.
* **SDKs**: a simple `client.upload()` hides multipart uploads, retries, and checksums.
* **ORMs**: classes map to tables, and a base model class adds common fields like `id` and `created_at`.

### 4. Tradeoffs

* **Inheritance vs. composition**: inheritance ties child to parent. A change in the parent can silently break children (the fragile base class problem). Composition lets you swap parts and inject fakes in tests.
* **Too much abstraction**: layers of boilerplate make the flow hard to follow.
* **Performance**: method lookup at runtime costs a little, usually nothing next to I/O.

### 5. When NOT to Use

* **Concurrent or data-pipeline code**: shared objects that change cause race conditions. Immutable data and pure functions often fit better (as in Elixir or most data pipelines).
* **Small scripts**: classes and hierarchies are overkill.

### 6. Interview Focus

* **Composition over inheritance for payment gateways**: each gateway can be swapped and tested alone; no deep class tree.
* **Birds that can and can't fly**: give flying as a separate capability, not a method on every `Bird`.
* **Abstract class vs. interface**: shared state and code vs. a pure contract; single vs. multiple inheritance.

### 7. Common Mistakes

* **Deep inheritance**: many levels of subclasses make any base class change risky.
* **Breaking encapsulation**: public fields everywhere, or a getter and setter for every field.
* **Early abstraction**: an interface for something that will only ever have one implementation.

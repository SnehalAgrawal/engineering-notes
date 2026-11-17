# SOLID principles

## Overview

SOLID is five rules for object-oriented design. They keep code easy to change.

They help you avoid two problems:

* **Rigidity**: a small change forces a big refactor.
* **Fragility**: a change in one place breaks something unrelated.

| Principle | Rule | Benefit |
| --------- | ---- | ------- |
| SRP | One reason to change per class | Changes stay in one place |
| OCP | Add new code instead of editing old code | Tested code stays untouched |
| LSP | A subclass must work wherever the parent works | Safe polymorphism |
| ISP | Small interfaces | Classes don't depend on methods they don't use |
| DIP | Depend on interfaces, not concrete classes | Easy to swap and mock |

## Key concepts

### Single Responsibility (SRP)

A class should have one reason to change. Keep code together when it changes for the same reason, for the same people.

Bad: two reasons to change.

```javascript
class UserService {
  saveUser(user) { /* DB logic */ }
  sendEmail(user) { /* email logic */ }
}
```

A database change and an email template change both edit this class.

Better: split it.

```javascript
class UserRepository {
  save(user) {}
}

class EmailService {
  send(user) {}
}
```

### Open/Closed (OCP)

Open for extension, closed for modification. To add a feature, add new code (a subclass, a strategy, a map entry). Don't edit code that already works.

Bad: every new type edits this function.

```javascript
function getDiscount(type) {
  if (type === "regular") return 10;
  if (type === "premium") return 20;
}
```

Better: one class per type.

```javascript
class Discount {
  get() { throw new Error("Not implemented"); }
}

class RegularDiscount extends Discount {
  get() { return 10; }
}

class PremiumDiscount extends Discount {
  get() { return 20; }
}
```

Something still has to pick the right class, usually a factory or a lookup map. OCP moves the `if` to one place. It does not remove it.

### Liskov Substitution (LSP)

Code that works with the parent class must keep working with any subclass. A subclass must not:

* ask more of the caller (stricter input rules),
* promise less (weaker results),
* throw errors the parent never throws.

Bad:

```javascript
class Bird {
  fly() {}
}

class Penguin extends Bird {
  fly() { throw new Error("Can't fly"); }
}
```

Any code that loops over birds and calls `fly()` now crashes on a penguin.

Better: only flying birds have `fly()`.

```javascript
class Bird {}

class FlyingBird extends Bird {
  fly() {}
}

class Penguin extends Bird {}
```

If a subclass behaves very differently from its parent, use composition instead of inheritance.

### Interface Segregation (ISP)

A class should not depend on methods it doesn't use. Prefer several small interfaces over one big one.

Bad: a robot worker would have to implement `eat()`.

```javascript
class Worker {
  work() {}
  eat() {}
}
```

Better:

```javascript
class Workable {
  work() {}
}

class Eatable {
  eat() {}
}
```

JavaScript has no interfaces. There, ISP means small modules and passing narrow objects. TypeScript interfaces express it directly.

### Dependency Inversion (DIP)

High-level code (business rules) should not depend on low-level code (database, email). Both depend on an interface. The high-level side owns that interface: the service says what it needs, and the database adapter implements it.

Bad: `Notification` is stuck with email and can't be tested without it.

```javascript
class Notification {
  constructor() {
    this.service = new EmailService();
  }
}
```

Better: pass the dependency in.

```javascript
class Notification {
  constructor(service) {
    this.service = service; // anything with send()
  }
}
```

Now you can pass an email service, an SMS service, or a mock in tests.

## Real-world usage

* **Plugins** (OCP): editors and build tools add features without changing the core.
* **Payment types** (OCP): one class per payment method instead of a growing `if/else`.
* **React** (SRP): custom hooks fetch data, components render it.
* **Swappable services** (DIP): change the SMS or email provider; use an in-memory repository in tests.
* **Testing** (ISP + DIP): mock only the small interface a class uses.
* **Microservices** (SRP): one service per business area.

## Tradeoffs

* **Over-engineering**: too many abstractions for a simple problem.
* **Boilerplate**: more files, interfaces, and wiring.
* **Harder debugging**: more layers between the caller and the code that runs.

Apply SOLID where code actually changes, not everywhere up front.

## When not to use

* **Prototypes and MVPs**: speed matters more than structure.
* **Short-lived scripts**: nobody will maintain them.
* **Hot paths**: where profiling shows the extra layers cost too much.

## Interview focus

* **Spot the violation**: you get a snippet and name the broken principle. See the examples below.
* **DIP vs. DI**:
  * DIP is a principle: depend on abstractions, not concrete classes. It says what to achieve.
  * DI is a technique: pass dependencies in from outside. It is one way to follow DIP.

## Common mistakes

* **SRP as "one method per class"**: SRP is about one reason to change, not one function.
* **Throwing "not implemented" in a subclass**: a subclass that can't do what the parent does breaks LSP.
* **OCP everywhere**: making every method extensible "just in case" adds complexity nobody needs.

---

## Spot the violation

### SRP

```javascript
class ReportManager {
  fetchData() { /* DB */ }
  generateReport() { /* formatting */ }
  sendEmail() { /* email */ }
}
```

One class does data access, formatting, and delivery.

```javascript
class User {
  constructor(name) { this.name = name; }
  saveToDatabase() { /* DB */ }
}
```

The entity also saves itself. Active Record frameworks do this on purpose, so it is a tradeoff, not always a bug.

### OCP

```javascript
function calculateSalary(employee) {
  if (employee.type === "fulltime") return employee.salary;
  if (employee.type === "contract") return employee.hourly * employee.hours;
}
```

A new employee type means editing this function. Fix: one class per type.

```javascript
class FullTimeEmployee {
  constructor(salary) { this.salary = salary; }
  calculateSalary() { return this.salary; }
}

class ContractEmployee {
  constructor(hourly, hours) {
    this.hourly = hourly;
    this.hours = hours;
  }
  calculateSalary() { return this.hourly * this.hours; }
}

new ContractEmployee(50, 20).calculateSalary(); // 1000
```

A `PaymentProcessor.process(type)` with `if (type === "card") ... else if (type === "paypal")` breaks OCP the same way.

### LSP

```javascript
class Rectangle {
  setWidth(w) { this.width = w; }
  setHeight(h) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w) { this.width = this.height = w; }
  setHeight(h) { this.width = this.height = h; }
}

function resize(rect) {
  rect.setWidth(5);
  rect.setHeight(4);
  return rect.area(); // expects 20; a Square returns 16
}
```

A rectangle lets width and height change separately. A square silently breaks that. A subclass that throws where the parent works (an `Ostrich.fly()` that throws) is the other classic case.

### ISP

```javascript
class Machine {
  print() {}
  scan() {}
  fax() {}
}

class Printer extends Machine {
  fax() { throw new Error("Not supported"); }
}
```

`Printer` must carry methods it can't support. The throwing method is also an LSP problem; the two often come together. A `Robot` forced to implement `Worker.eat()` is the same issue.

### DIP

```javascript
class UserService {
  constructor() {
    this.db = new MySQLDatabase();
  }
}
```

The service creates a concrete database class itself. Same problem when a method calls `new EmailSender()` inside: it can't be swapped or mocked.

## Quick checks

* **SRP**: does this class have more than one reason to change?
* **OCP**: must I edit existing code to add a new variant?
* **LSP**: can I pass a subclass wherever the parent is expected, with no surprises?
* **ISP**: is this class forced to implement methods it doesn't need?
* **DIP**: does this class create a concrete dependency it should receive instead?

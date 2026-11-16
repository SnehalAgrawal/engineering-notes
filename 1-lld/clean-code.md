# Clean Code & Refactoring

### 1. Overview

Clean code is easy to read, easy to change, and says what it means. Write it for the next person who reads it.

Refactoring means changing how code is organized inside without changing what it does from the outside. Like renovating a house without changing how people use it.

Why it matters for senior engineers:

* **Technical debt**: keep shortcuts from piling up.
* **Steady speed**: the team should move as fast next year as this year.
* **Safe changes**: code you can change without fear.

### 2. Key Concepts

* **Intent-revealing names**: a good name makes a comment unnecessary. `flag` says nothing; `isUserEligibleForDiscount` says everything.

* **Small functions**: a function does one thing, at one level of detail.

  Bad: checks, SQL, and email all inline.

  ```javascript
  function processOrder(order) {
    if (!order.items.length) throw new Error("Empty order");
    db.query("INSERT INTO orders ...", [order.id, order.total]);
    mailer.send(order.email, "Order received", `Order ${order.id} confirmed`);
  }
  ```

  Better: the top function reads like a summary. Small functions are also easier to test and reuse.

  ```javascript
  function processOrder(order) {
    validateOrder(order);
    persistOrder(order);
    notifyCustomer(order);
  }
  ```

* **Code smells**: signs that something deeper may be wrong. Common ones:
  * Long Method.
  * Long Parameter List: `createUser(name, age, address, phone, email)`. Fix: pass one object, `createUser({ name, age, ... })`.
  * Duplicate Code.
  * Shotgun Surgery: one change needs edits in many files.
  * Primitive Obsession: plain strings and numbers where a small type belongs (an `Email` type instead of a string).
  * God Object: one class that does too much (auth, email, DB, UI). Split it into focused services.
  * Spaghetti Code: tangled flow, like one big event handler full of nested `if`s. Pull out named functions and separate layers.

* **Boy Scout Rule**: leave the code a little cleaner than you found it. Even renaming `x` to `totalPrice` counts.

* **Technical debt**: a shortcut today costs more work later.

  ```javascript
  // Quick fix: every new type edits this chain
  if (type === "A") handleA();
  else if (type === "B") handleB();

  // Better: a lookup table (Strategy idea)
  const handlers = { A: handleA, B: handleB };
  handlers[type]?.();
  ```

### 3. Real-World Usage

* **Code reviews**: use shared words ("this has a Long Method smell") instead of "this looks bad".
* **Legacy migrations**: break a huge `processEverything()` into `processUser()`, `processPayment()`, `processNotification()`, one small step at a time: extract, test, move.
* **Onboarding**: `if is_user_eligible_for_discount(user):` needs no comment. Clean code works as documentation.

### 4. Tradeoffs

* **Readability vs. short code**: `arr.filter(x => x.age > 18).map(x => x.name)` is short but the names hide meaning. `users.filter(user => user.age > 18)` is clearer.
* **Refactoring time**: too much delays delivery; too little slows every later change. Refactor the code you are already touching.
* **Too many layers**: `getData() -> processData() -> transformData() -> formatData()`. When each layer adds little, the flow is harder to follow.

### 5. When NOT to Use

* **Legacy code without tests**: add tests that capture current behavior first (characterization tests). Then refactor.
* **Short-lived scripts**: a script that runs once doesn't need careful structure.

### 6. Interview Focus

* **Code review exercise**: turn `function f(x){ return x*0.9 }` into `applyDiscount(price)` with a named `DISCOUNT_RATE` constant.
* **Refactoring a big ball of mud** (a system with no clear structure):
  1. Add tests around current behavior.
  2. Find seams: places where you can cut in and swap behavior.
  3. Extract small functions.
  4. Replace old paths step by step.
* **Naming**: `handleData()` says nothing. `validateAndCacheUserData()` is clear, and the "and" hints it should be two functions.

### 7. Common Mistakes

* **Refactoring for style only**: if code works, rarely changes, and has no bugs, leave it.
* **Comments that repeat the code**: `// add 2 numbers` above `a + b`. Write `const totalPrice = basePrice + tax;` and keep comments for the why.
* **Changing behavior while refactoring**: "while I'm here, I'll fix this too". Keep refactoring commits behavior-neutral. Ship fixes separately.
* **Aiming for perfect**: the goal is to make the next change easy. A good test: someone new can understand the code without asking you.

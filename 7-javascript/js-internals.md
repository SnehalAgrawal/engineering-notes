# JavaScript internals

## Overview
How JavaScript finds variables, what `this` points to, how objects inherit, and how the V8 engine makes code fast.

## Key concepts
*   **Hoisting**: Declarations are set up before the code in their scope runs.
    *   `var`: function-scoped, starts as `undefined`.
    *   `let`, `const`, `class`: block-scoped. Using them before their line throws a `ReferenceError`. That gap is the Temporal Dead Zone (TDZ).
    *   Function declarations are hoisted with their body, so you can call them before the line.
    ```javascript
    console.log(a); // undefined
    var a = 5;
    console.log(b); // ReferenceError
    let b = 5;
    ```
*   **Closure**: A function keeps access to the variables where it was written, even after the outer function has returned.
*   **`this`**: Set by how a function is called, not where it is written. Rules, strongest first:
    1.  `new Fn()`: the new object.
    2.  `fn.call(obj)`, `fn.apply(obj)`, `fn.bind(obj)`: `obj`.
    3.  `obj.fn()`: `obj`.
    4.  Plain `fn()`: `undefined` in strict mode (modules, classes), the global object otherwise.
*   **Arrow functions**: Have no own `this`, `arguments`, or `super`. They use the `this` of the code around them, and `bind` cannot change it.
    ```javascript
    function greet(msg) { return `${msg}, ${this.name}`; }
    const user = { name: 'Alice' };
    greet.call(user, 'Hi');        // 'Hi, Alice'
    greet.apply(user, ['Hi']);     // 'Hi, Alice' (arguments as an array)
    greet.bind(user)('Hi');        // 'Hi, Alice' (returns a new function)
    ```
*   **Prototype chain**: An object can link to another object, its prototype. If a property is missing, JS looks in the prototype, then its prototype, until it reaches `null`. `class` uses the same mechanism. `Object.create(null)` makes an object with no prototype.
    ```javascript
    function Person(name) { this.name = name; }
    Person.prototype.sayHi = function () { return `Hi, ${this.name}`; };
    new Person('Alice').sayHi(); // 'Hi, Alice', found on Person.prototype
    ```
*   **V8 tiers**: V8 first runs bytecode in an interpreter (Ignition). Code that runs often is recompiled by faster compilers (Sparkplug, Maglev, TurboFan) based on the types it has seen.
*   **Hidden class (V8 "map")**: The shape of an object: which properties, in which order. Objects with the same shape share property lookups.
*   **Inline cache**: Each property access remembers the shapes it saw. One shape is fastest. Many shapes ("megamorphic") is slow.
*   **Deoptimization**: If a guess about types turns out wrong, V8 throws away the fast code and goes back to a slower tier.

## Real-world usage
*   **Private state**: Closures hide variables. Today `#private` class fields do the same.
*   **Helpers**: `memoize`, `once`, `debounce` are closures over private state.
*   **Polyfills**: Adding a missing standard method like `Array.prototype.flat` to old browsers. This is the one accepted case of changing a built-in prototype.
*   **Fast objects**: Set all fields in the constructor, always in the same order.

## Tradeoffs
*   **Closures and memory**: Captured variables stay alive as long as the closure. In V8, closures made in the same scope share one context, so one closure can keep another's large variable alive.
*   **Arrow functions**: Handy for callbacks. Cannot be used with `new`, have no `arguments`, and are a bad fit for object methods.
*   **Dynamic types**: Code that always sees the same types gets optimized. Code that mixes types stays slower.

## When not to use
*   **Deep inheritance**: Long class chains are hard to follow. Prefer composition.
*   **Changing built-in prototypes in shared code**: Adding your own methods to `Array.prototype` can clash with other libraries or future standards.

## Interview focus
*   **Counter with a closure**:
    ```javascript
    function createCounter() {
      let count = 0;
      return () => ++count;
    }
    const counter = createCounter();
    counter(); // 1
    counter(); // 2
    ```
*   **`var` vs `let` in a loop**: `for (var i = 0; i < 3; i++) setTimeout(() => console.log(i));` prints `3 3 3`. With `let` it prints `0 1 2`, because `let` makes a new `i` for each loop turn.
*   **Hidden classes**:
    *   Adding properties one by one is fine if every object gets them in the same order. V8 reuses the same shapes.
    *   The slow case is objects of different shapes reaching the same code: different property order, optional fields, or `delete`.
*   **`{}.toString()`**: `{}` has no own `toString`. JS finds it on `Object.prototype` and calls it with `this` set to `{}`. Result: `'[object Object]'`.

## Common mistakes
*   **Losing `this`**: `setTimeout(obj.method, 0)` calls the method without `obj`. Use `() => obj.method()` or `bind`.
*   **Arrow function as a method**: `{ get: () => this.name }` reads `this` from outside the object.
*   **Closure leaks**: A listener that is never removed keeps everything its closure captured.
*   **Different object shapes**: Adding properties in different orders or only sometimes, in hot code.

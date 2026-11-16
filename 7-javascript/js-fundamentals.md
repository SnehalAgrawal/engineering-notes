# JavaScript Fundamentals: Quick Reference

### 1. Overview
Language basics that come up in screening rounds. Scope, hoisting, `this`, and prototypes are in `js-internals.md`. The event loop is in `js-async-event-loop.md`.

### 2. Key Concepts
*   **`var`, `let`, `const`**:

    | | `var` | `let` | `const` |
    | --- | --- | --- | --- |
    | Scope | Function | Block | Block |
    | Before its line | `undefined` | `ReferenceError` | `ReferenceError` |
    | Reassign | Yes | Yes | No |
    | Redeclare | Yes | No | No |

*   **`const` objects**: `const` stops reassignment, not changes. `const arr = []; arr.push(1);` works.
*   **Types**: Seven primitives: `string`, `number`, `bigint`, `boolean`, `undefined`, `null`, `symbol`. They are copied and compared by value. Everything else (objects, arrays, functions) is an object, compared by reference.
*   **`typeof` quirks**: `typeof null` is `'object'`. Use `Array.isArray` for arrays.
*   **`==` vs `===`**: `===` compares without converting types. `==` converts first.
    ```javascript
    0 == false;           // true
    0 === false;          // false
    null == undefined;    // true
    NaN === NaN;          // false, use Number.isNaN(x)
    ```
*   **Falsy values**: `false`, `0`, `-0`, `0n`, `''`, `null`, `undefined`, `NaN`. Everything else is truthy, including `'0'`, `[]`, `{}`.
*   **`||` vs `??`**: `a || b` uses `b` for any falsy `a`. `a ?? b` uses `b` only when `a` is `null` or `undefined`. So `0 || 5` is `5`, but `0 ?? 5` is `0`.
*   **Optional chaining**: `user.address?.city` gives `undefined` instead of throwing when `address` is `null`.
*   **Destructuring, spread, rest**:
    ```javascript
    const { name, ...rest } = { name: 'Alice', age: 30 }; // rest = { age: 30 }
    const [x, , z = 0] = [1, 2];                           // x = 1, z = 0
    const more = [...[1, 2, 3], 4];                        // [1, 2, 3, 4]
    function log(...args) { return args; }                 // args is an array
    ```
*   **Function forms**: Declarations (hoisted), expressions, arrow functions, and IIFEs: `(function () { ... })()`, a function that runs right away. IIFEs were the old way to get a private scope before modules.
*   **Shallow copy**: `{ ...obj }` or `[...arr]`. Nested objects are still shared.
*   **Deep copy**: `structuredClone(obj)`. Handles `Date`, `Map`, and cycles; throws on functions. `JSON.parse(JSON.stringify(obj))` loses `undefined`, functions, and `Date` types, and fails on cycles.
*   **`Map` / `Set`**: `Map` takes any key and keeps insertion order. `Set` keeps unique values: `[...new Set(arr)]` removes duplicates.
*   **`WeakMap` / `WeakSet`**: Keys are objects and do not keep them alive. Good for data attached to objects you do not own.
*   **Array methods**: `map`, `filter`, `reduce` return new values. `sort`, `reverse`, `splice` change the array; `toSorted`, `toReversed`, `toSpliced` return copies.
*   **Default sort**: Compares as strings. `[10, 2, 1].sort()` gives `[1, 10, 2]`. Use `sort((a, b) => a - b)`.
*   **Classes**: Syntax on top of prototypes. Support `#private` fields, `static`, `extends`/`super`. Always strict mode.
*   **ES modules vs CommonJS**: ES modules (`import`/`export`) are static and strict. CommonJS (`require`/`module.exports`) is Node's older, synchronous system.

### 3. Interview Focus
*   **`[] == false` is `true`, but `[]` is truthy**: `==` turns both into numbers (`[]` → `''` → `0`). Truthiness is a separate rule.
*   **`null` vs `undefined`**: `undefined` means not set yet. `null` means set to empty on purpose.
*   **Deep copy**: `structuredClone`. Explain what JSON round-tripping loses.
*   **When to use `WeakMap`**: Data per object that should go away with the object.

### 4. Common Mistakes
*   **Thinking `const` means immutable**: It only blocks reassignment.
*   **`||` for defaults when `0` or `''` are valid**: Use `??`.
*   **Changing nested data after a shallow copy**: The original changes too.
*   **`sort()` on numbers without a comparator**: Sorts as strings.

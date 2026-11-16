# JS Advanced Concepts: Memory & Optimization

### 1. Overview
Keeping memory flat in long sessions, limiting how often expensive handlers run, and writing small pure functions.

### 2. Key Concepts
*   **Memory leak**: Memory held by objects you no longer need, but that something still points to (a global, a timer, a listener, a closure).
*   **Garbage collection in V8**: New objects go to a small "young" area that is cleaned often and fast. Objects that survive move to the "old" area, which is cleaned with mark-sweep and compaction. Much of the work runs in the background. V8 frees what is unreachable, so cycles are not a problem.
*   **Debounce**: Run the function only after calls stop for N ms. Example: search box.
*   **Throttle**: Run the function at most once every N ms. Example: scroll handler. For visual updates, `requestAnimationFrame` often works better.
*   **Higher-order function**: A function that takes or returns a function. Examples: `map`, `filter`, `reduce`, `debounce`.
*   **Pure function**: Same input gives the same output, and nothing else changes.
*   **Currying**: Turn `f(a, b)` into `f(a)(b)`.
*   **Memoization**: Cache results of a pure function by its arguments.
*   **Composition**: `compose(f, g)(x)` means `f(g(x))`.
*   **Factory function vs function factory**: The first returns an object (`createUser(name)`). The second returns a function (`makeMultiplier(2)`).
*   **Event delegation**: One listener on a parent handles events from all its children, using `event.target`.

```javascript
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, interval) {
  let last = 0;
  return function (...args) {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

### 3. Real-World Usage
*   **Search box**: Debounce input (around 300 ms is common), so the API is called when the user pauses, not on every key.
*   **Infinite scroll**: Throttle the scroll handler, or use `IntersectionObserver` and skip the handler.
*   **Finding leaks**: In Chrome DevTools, take a heap snapshot, repeat an action a few times, take another, and compare. Growing counts and "Detached" DOM nodes point to the leak.
*   **Shaping API data**: Turn a messy API response into a UI model with pure functions, without changing the original object.

### 4. Tradeoffs
*   **`map`/`filter` chains vs a `for` loop**: Chains are easier to read but create a new array at each step. In a measured hot loop, one `for` loop can be faster.
*   **Debounce vs throttle**: Debounce can delay feedback for as long as events keep coming. Simple throttle (like the one above) can drop the last event.
*   **Immutability**: Makes change detection and undo easy. Copying big objects on every change costs memory and GC time. `Object.freeze` is shallow.

### 5. When NOT to Use
*   **Copying large arrays on every change in hot code**: `[...arr]` and `toSorted()` copy everything. If you own the data, sorting in place is fine.
*   **Globals by accident**: Assigning to an undeclared variable makes a global in sloppy mode. Top-level `var` in a classic script becomes a `window` property. Use `let`/`const` and ES modules.

### 6. Interview Focus
*   **App gets slower after navigating back and forth**: Repeat the navigation, compare heap snapshots, then look for listeners, intervals, subscriptions, and caches that are never cleaned up.
*   **Write `debounce`**: See above. Mention keeping `this` and arguments, leading vs trailing call, and a `cancel` method.
*   **Why purity matters in React/Redux**: Reducers and render functions must be pure, so changes can be detected by reference and tested in isolation.

### 7. Common Mistakes
*   **Listener not removed**: A `window` listener or `setInterval` added in a component with no `useEffect` cleanup.
*   **Closure holding a DOM node**: Keeps a removed element and its whole subtree in memory.
*   **Throttle interval too long**: For example 1000 ms on drag-and-drop makes the UI feel stuck.
*   **Cache with no limit**: A memoize `Map` grows forever. Limit its size, or use a `WeakMap` when keys are objects.

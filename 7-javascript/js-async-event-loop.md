# JS Async: The Event Loop & Promises

### 1. Overview
JavaScript runs your code on one thread. The browser or Node.js does timers and I/O in the background and puts the callbacks in queues. The event loop takes callbacks from those queues and runs them, one at a time. Most interview questions are about the order in which this happens.

### 2. Key Concepts
*   **Call stack**: The functions running right now. Queued callbacks run only when it is empty.
*   **Task (macrotask)**: A unit of work in a task queue: `setTimeout`/`setInterval` callbacks, I/O callbacks, UI events.
*   **Microtask**: A small follow-up job: promise `then`/`catch`/`finally`, the code after an `await`, `queueMicrotask`, `MutationObserver`.
*   **Browser loop**: Run one task. Then run all microtasks, including new ones added along the way. Then render if a frame is due. Repeat. Rendering is a step of the loop, not a task.
*   **Node.js loop phases**: timers → pending callbacks → poll (I/O) → check (`setImmediate`) → close callbacks.
*   **`process.nextTick`**: Node only. Has its own queue, which runs before the promise queue.
*   **`setTimeout(fn, 0)` vs `setImmediate`**: In Node's main module the order is not fixed. Inside an I/O callback, `setImmediate` runs first.
*   **`async` function**: Runs normally until its first `await`. The rest runs later as a microtask.

```javascript
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('end');
// start, end, promise, timeout

async function f() { console.log('A'); await null; console.log('B'); }
f(); console.log('C');
// A, C, B
```

```javascript
// Node.js, CommonJS file
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
// nextTick, promise, then timeout/immediate in either order
// In an ES module, 'promise' prints before 'nextTick'
```

### 3. Real-World Usage
*   **Responsive UI**: Move heavy work to a Web Worker, or split it into chunks with `setTimeout` so the browser can paint between chunks.
*   **Batched updates**: Frameworks collect state changes and apply them once in a microtask (for example Vue's `nextTick`).
*   **Network calls**: `await` waits for a response without blocking the thread. Wrap it in `try`/`catch` to handle a rejected promise.
*   **Node.js servers**: One process serves many connections at once, because waiting on a socket does not hold a thread.

### 4. Tradeoffs
*   **One thread**: No locks needed, but one long loop freezes everything.
*   **`async`/`await`**: Easy to read, but easy to run independent calls one after another by mistake. Use `Promise.all([a(), b()])`.
*   **Microtask priority**: Microtasks run before rendering. Endless microtasks mean the page never repaints.

### 5. When NOT to Use
*   **CPU-heavy work on the main thread**: Promises do not make it non-blocking. Use Web Workers, Node `worker_threads`, or WebAssembly.

### 6. Interview Focus
*   **Output order**: Sync code first, then all microtasks, then the next task.
*   **Unhandled rejection**: Browsers fire an `unhandledrejection` event. Node.js 15+ crashes the process by default.
*   **Sync throw vs async rejection**: A `throw` stops the current code right away. A rejection only shows up when something handles the promise, or as an unhandled rejection if nothing does.
*   **Starvation**: Microtasks can starve tasks. The loop does not move on until the microtask queue is empty. Recursive `process.nextTick` starves I/O the same way.

### 7. Common Mistakes
*   **`await` in a loop**: N requests take N round trips. Use `Promise.all` when they are independent.
*   **`await` inside `forEach`**: `forEach` does not wait. Use `for...of` (one by one) or `Promise.all` with `map` (together).
*   **Overusing `process.nextTick`**: It runs before I/O and can delay it. Prefer `setImmediate` or `queueMicrotask`.

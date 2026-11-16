# Python Internals: GIL & Memory Management

### 1. Overview
CPython is the standard Python. Its default build has a Global Interpreter Lock (GIL), frees most memory by counting references, and runs a separate collector for reference cycles. These explain why threads do not speed up CPU-heavy Python and when memory is freed.

### 2. Key Concepts
*   **GIL**: A lock in the default CPython build. Only the thread holding it can run Python bytecode, so pure-Python CPU work uses one core per process.
*   **When threads still help**: Blocking I/O releases the GIL. Many C extensions (NumPy, `hashlib`, `zlib`) also release it during heavy work.
*   **Free-threaded build (PEP 703)**: An optional CPython build with no GIL (`python3.13t`, `python3.14t`). Experimental in 3.13, officially supported in 3.14, but not the default. Threads run in parallel, single-thread code is somewhat slower, and C extensions that do not declare support turn the GIL back on.
*   **Subinterpreters**: Separate interpreters in one process. Since 3.12 each can have its own GIL (PEP 684). Python 3.14 adds them to the standard library as `concurrent.interpreters`.
*   **Reference counting**: Each object counts the references to it. At zero, CPython frees it right away. Other Pythons, like PyPy, work differently.
*   **Cycle collector**: Objects that point at each other never reach zero. The `gc` module finds and frees such groups from time to time.
*   **Small int cache and interning**: CPython reuses objects for ints from -5 to 256 and for many strings. So `is` can look like it works on equal values. It is an implementation detail; compare values with `==`.

### 3. Real-World Usage
*   **Picking a model**: Threads or `asyncio` for I/O (calling many APIs). Processes for CPU work (image processing) on the default build.
*   **Finding memory growth**: `tracemalloc` shows which lines allocate memory. `gc.get_referrers()` or `objgraph` show what keeps an object alive.
*   **Native code**: Cython (`with nogil:`), C, or Rust (PyO3) can release the GIL in heavy loops.
*   **Scaling out**: Celery or RQ spread work over many worker processes and machines.

### 4. Tradeoffs
*   **GIL**: Keeps the interpreter and C extensions simple and single-thread code fast. Blocks parallel threads. The free-threaded build flips this trade.
*   **Reference counting**: Frees memory at a predictable moment. Every reference change costs a little, and cycles need the extra collector, which adds short pauses.
*   **Multiprocessing**: Real parallelism, but each process has its own memory, and data between processes must be pickled.

### 5. When NOT to Use
*   **Threads for CPU-heavy Python on the default build**: No speedup, sometimes slower. Use processes, a library that releases the GIL, or a free-threaded build.
*   **Calling `gc.collect()` by hand**: Rarely needed. It forces a full pause. Do it only with a measured reason.

### 6. Interview Focus
*   **Why locks if there is a GIL**: The GIL only protects one bytecode step. `counter += 1` is read, add, write. A thread switch between steps loses updates.
*   **Reference cycle**: Objects pointing at each other. Only the cycle collector frees them, later. `weakref` breaks cycles like parent links or caches.
*   **Fetch 1,000 URLs**: The work is I/O, so use `asyncio`, or a thread pool if the client library blocks. Processes add cost with no gain.
*   **What changes without the GIL**: Threads run in parallel, C extensions must be thread-safe, single-thread speed drops a bit.

### 7. Common Mistakes
*   **"The GIL makes my code thread-safe"**: It protects the interpreter, not your data. `counter += 1` is not atomic in any version.
*   **Growing module-level caches or lists**: Objects in them are never freed.
*   **Blocking I/O in async code**: Stops the whole event loop (see `python-concurrency.md`).
*   **Using `is` on numbers or strings**: Works for small cached values, fails for others.

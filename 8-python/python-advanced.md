# Advanced Python

## Overview
Three everyday tools: generators to loop over big data without loading it all, decorators to add behavior like logging or caching to many functions, and context managers to always clean up resources.

## Key concepts
*   **Generator**: A function with `yield`. Calling it returns an iterator. Each `next()` runs the code up to the next `yield`, then pauses and keeps its local variables. Only one value exists at a time.
*   **Generator expression**: `(x * x for x in data)`, the lazy version of `[x * x for x in data]`.
*   **`yield from`**: Yields every value from another iterable or generator.
*   **Decorator**: A function that takes a function and returns a new one, usually a wrapper. `@deco` above `def f` means `f = deco(f)`. Use `functools.wraps` so the wrapper keeps the original name and docstring.
*   **Context manager**: An object used with `with`. `__enter__` sets up. `__exit__` always runs at the end, even after an error. `contextlib.contextmanager` builds one from a generator.
*   **Dunder methods**: Methods like `__len__`, `__iter__`, `__eq__`, `__repr__` that let your class work with `len()`, `for`, `==`, and `print`.

```python
import functools, time

def time_it(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            print(f"{func.__name__} took {time.perf_counter() - start:.3f}s")
    return wrapper

def read_lines(path):
    with open(path) as f:
        for line in f:
            yield line.rstrip("\n")
```

## Real-world usage
*   **Logging, auth, caching, retries**: Decorators on many functions. `functools.lru_cache` is a built-in one.
*   **Database transactions**: A context manager that commits on success, rolls back on error, and always returns the connection.
*   **Big files**: Chained generators read, filter, and transform a file larger than RAM, one line at a time.
*   **Frameworks**: Flask and FastAPI use decorators for routes (`@app.get("/")`). pytest uses them for fixtures.

## Tradeoffs
*   **Decorators**: Less repeated code, but the behavior is hidden from the caller, and tracebacks get extra frames.
*   **Generators vs lists**: Low memory, but you can loop only once. No `len()`, no indexing.
*   **`with` vs `try`/`finally`**: Same guarantee, but reusable and harder to get wrong.

## When not to use
*   **Many stacked decorators**: Hard to see what runs in which order. Call the logic directly instead.
*   **Generators for small data you reuse**: Use a list.

## Interview focus
*   **Write `@time_it`**: See above. A decorator with arguments, like `@retry(times=3)`, needs one more nested function.
*   **Why `with open(...)` is safer**: If code between `open()` and `close()` raises, `close()` never runs. `with` always closes the file.
*   **`yield` vs `return`**: `return` ends the function with one value. `yield` gives a value, pauses, and continues on the next `next()`.

## Common mistakes
*   **Shared state in a decorator**: A counter or cache in the decorator is shared by every call and every thread.
*   **Swallowing errors in `__exit__`**: If `__exit__` returns a truthy value, the exception disappears. Return `None` unless you mean to suppress it.
*   **Looping over a generator twice**: The second loop gets nothing.
*   **Forgetting `functools.wraps`**: Every decorated function shows up as `wrapper`.

# Python concurrency

## Overview
`asyncio` runs many I/O tasks on one thread using `async`/`await`. With threads, the OS decides when to switch. With `asyncio`, a task runs until it hits an `await` on something not ready yet. Then the event loop switches to another task that can make progress.

## Key concepts
*   **Event loop**: Runs tasks and wakes them up when their I/O or timer is ready. `asyncio.run(main())` starts it.
*   **Coroutine**: A function defined with `async def`. Calling it only creates a coroutine object. Nothing runs until you `await` it or make it a task.
*   **Task**: A coroutine scheduled to run alongside others, made with `asyncio.create_task`.
*   **`asyncio.TaskGroup`** (3.11+): Runs a group of tasks. If one fails, it cancels the rest.
*   **`asyncio.gather`**: Waits for several awaitables and returns their results.
*   **Future**: A low-level placeholder for a result that arrives later. Tasks are futures. You rarely create them yourself.
*   **Awaitable**: Anything you can `await`: coroutines, tasks, futures.

```python
import asyncio

async def fetch(i):
    await asyncio.sleep(1)  # stands in for a network call
    return i * 2

async def main():
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(i)) for i in range(3)]
    print([t.result() for t in tasks])  # [0, 2, 4] after about 1 second, not 3

asyncio.run(main())
```

## Real-world usage
*   **Async web servers**: FastAPI and other ASGI frameworks serve many requests that mostly wait on databases and APIs.
*   **Calling many URLs**: The waits overlap on one thread. This is concurrent, not parallel.
*   **Long-lived connections**: WebSockets, chat, notifications.
*   **Async database drivers**: `asyncpg` and async SQLAlchemy keep the loop free during queries.

## Tradeoffs
*   **No parallelism**: The loop runs on one thread, so CPU-heavy code gets no faster. This comes from the single thread, not from the GIL.
*   **Async spreads**: `await` works only inside `async def`, so async code tends to spread up the call stack.
*   **Debugging**: Tracebacks can be split across tasks. `asyncio.run(main(), debug=True)` reports slow callbacks and coroutines that were never awaited.

## When not to use
*   **CPU-heavy work**: It blocks every task on the loop. Send it to a process pool with `loop.run_in_executor`.
*   **Blocking libraries**: `requests.get` or `time.sleep` inside `async def` freezes the loop. Use `httpx` or `aiohttp`, or `asyncio.to_thread`.

## Interview focus
*   **`time.sleep(10)` in `async def`**: The whole loop stops for 10 seconds. Use `await asyncio.sleep(10)`.
*   **Limit concurrent requests**: Wrap each call in an `asyncio.Semaphore`, or use a few worker tasks reading from an `asyncio.Queue`.
*   **Run blocking code from async code**: `await asyncio.to_thread(fn, *args)` (3.9+) or `loop.run_in_executor(None, fn)`.
*   **Threads vs processes vs asyncio**: Many I/O connections: `asyncio`. I/O with blocking libraries: threads. CPU work: processes, or a free-threaded build (see `python-internals.md`).

## Common mistakes
*   **Calling a coroutine without `await`**: The body never runs. Python warns "coroutine was never awaited".
*   **One blocking call in async code**: A sync `db.commit()` stalls all other requests.
*   **Fire-and-forget tasks**: The loop keeps only a weak reference to a task, so it can be garbage-collected before it finishes. If it fails, the error is only logged. Keep a reference or use a `TaskGroup`, and check results.
*   **Catching `asyncio.CancelledError` and not re-raising**: Breaks timeouts and `TaskGroup` cleanup.

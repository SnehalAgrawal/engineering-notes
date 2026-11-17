# Concurrency fundamentals

## Overview
Concurrency: tasks whose run times overlap. Parallelism: tasks running at the same instant on different cores. The practical work is keeping shared data safe and avoiding races and deadlocks.

## Key concepts
*   **Process**: A running program with its own memory. Isolated, but costly to create and switch.
*   **Thread**: A line of execution inside a process. Threads share the process's memory: cheap, and easy to get wrong.
*   **Race condition**: The result depends on which thread runs first.
*   **Mutex (lock)**: Lets one thread at a time into a critical section.
*   **Semaphore**: A counter that lets up to N threads use something at once.
*   **Deadlock**: Threads wait on each other's locks in a cycle, so none can move.

## Real-world usage
*   **Web servers**: Java and .NET run each request on a thread from a pool. Java 21 virtual threads make a thread per request cheap.
*   **Python workers**: CPython's GIL lets one thread run Python code at a time. CPU-bound work uses processes (Celery's default prefork pool). Threads still help for I/O, since the GIL is released while waiting.
*   **Ruby workers**: Ruby has a similar lock, so Sidekiq runs several processes to use all cores.
*   **Connection pool**: Caps open database connections, like a semaphore.
*   **Concurrent maps**: Java's `ConcurrentHashMap` needs no global lock. Use its atomic methods (`computeIfAbsent`, `merge`) for read-modify-write.

## Tradeoffs
*   **Threads vs. processes**: Threads share memory cheaply, but a fatal bug kills the whole process. Processes are isolated but use more memory and talk through slower IPC.
*   **One lock vs. many**: One big lock is simple but makes threads wait. Many small locks allow more parallel work but invite deadlocks.
*   **Pessimistic vs. optimistic**: Lock first when conflicts are common. Check a version and retry when they are rare.

## When not to use
*   **Shared memory and locks by default**: Prefer passing messages (Go channels, actors, queues). Go's advice: "Don't communicate by sharing memory; share memory by communicating." Deadlocks are still possible.
*   **One OS thread per task at huge scale**: Each thread needs stack memory and scheduling. For tasks that mostly wait, use async I/O, coroutines, or virtual threads.

## Interview focus
*   **`count += 1`**: "Why isn't it thread-safe?"
    * It is three steps: read, add, write.
    * Two threads read 5, both write 6. One update is lost.
    * Fix: a lock or an atomic increment.
*   **Deadlock conditions**: "Name the four and how to break them."
    * Mutual exclusion, hold and wait, no preemption, circular wait.
    * Usual fix: always take locks in one global order. Timeouts (`tryLock`) also help.
*   **Producer-consumer**: "Build it with a thread-safe queue."
    * Use a bounded blocking queue.
    * Producers block when it is full (backpressure); consumers block when it is empty.

## Common mistakes
*   **Check-then-act**: Checking "key is missing" and then inserting, while another thread inserts in between. Make it one atomic step.
*   **Plain flags**: Another thread may never see a plain variable change. Use `volatile`, an atomic, or a lock. `volatile` gives visibility, not atomicity: `count++` is still a race.
*   **Locks during I/O**: Holding a lock across a network call blocks every thread that needs it until the call ends.

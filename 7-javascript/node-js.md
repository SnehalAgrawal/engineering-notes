# Node.js

## Overview
Node.js runs JavaScript on V8. A library called libuv gives it an event loop and non-blocking I/O. Your code runs on one thread per process. The main skills are: keep that thread free, handle big data with streams, and use more than one CPU core.

## Key concepts
*   **Non-blocking I/O**: Network I/O uses the OS's async features (epoll, kqueue, IOCP). File system, `dns.lookup`, and some `crypto`/`zlib` work run on the libuv thread pool (4 threads by default, set with `UV_THREADPOOL_SIZE`).
*   **EventEmitter**: Objects that emit named events (`'data'`, `'error'`) that listeners react to. Servers and streams are built on it. An `'error'` event with no listener crashes the process.
*   **Stream**: Data handled in small chunks, so memory stays small whatever the input size.
    *   *Readable*: a source, like `fs.createReadStream`.
    *   *Writable*: a destination, like an HTTP response.
    *   *Duplex*: both, like a TCP socket. *Transform*: a duplex that changes data, like gzip.
*   **highWaterMark**: The buffer size at which a stream asks the producer to slow down.
*   **Backpressure**: Slowing the fast side down to the speed of the slow side.
*   **Buffer**: Raw bytes. A `Uint8Array` whose memory lives outside the V8 heap.
*   **Cluster**: Several processes of the same app sharing one port. Each has its own memory and event loop.
*   **Worker threads**: Threads inside one process, each with its own V8 instance and event loop. They talk by messages or `SharedArrayBuffer`. Use them for CPU work, not for I/O.

## Real-world usage
*   **Uploads and downloads**: Stream the file to storage with `pipeline`. Memory stays flat even for huge files.
*   **WebSockets**: An idle connection costs some memory but no thread, so one process can hold many.
*   **Data pipelines**: Read CSV → Transform rows → Write to a database.
*   **Using all cores**: One process per CPU (`os.availableParallelism()`) with `cluster` or PM2. In containers, often one process per container and more containers.

## Tradeoffs
*   **Great for I/O, bad for CPU**: One heavy call (a huge `JSON.parse`, a long loop) blocks every request in that process.
*   **Streams vs reading the whole file**: Less memory, but more error handling. Use `pipeline`, not `.pipe()` chains.
*   **Cluster and memory**: Every process has its own heap. For example, 8 processes at 500 MB each need about 4 GB.
*   **Worker threads vs cluster**: Workers are lighter and can share memory. A crash still takes down the whole process; cluster keeps processes separate.

## When not to use
*   **Mostly CPU work**: Heavy math, video encoding, or in-process ML inference fit better in Go, Rust, or Java, or in a separate service.
*   **Shared state in memory across a cluster**: Processes do not share memory. Keep sessions and counters in a store like Redis.

## Interview focus
*   **Read a 10 GB file with 2 GB RAM**: Stream it, one chunk at a time.
    ```javascript
    // inside an async function
    let total = 0;
    for await (const chunk of fs.createReadStream(path)) {
      total += chunk.length; // 64 KiB chunks by default
    }
    ```
*   **`cluster.fork()` vs threads in other languages**: It starts a new process, not a thread. Separate memory and event loop. They talk only through IPC messages. Java or Go threads share one heap.
    ```javascript
    if (cluster.isPrimary) {
      for (let i = 0; i < os.availableParallelism(); i++) cluster.fork();
    } else {
      http.createServer((req, res) => res.end('hi')).listen(8000); // all workers share port 8000
    }
    ```
*   **Backpressure**:
    *   `write()` returns `false` when the writable's buffer is full.
    *   Then stop writing and wait for `'drain'`. `'drain'` fires when the buffer has emptied.
    ```javascript
    readable.on('data', (chunk) => {
      if (!writable.write(chunk)) {
        readable.pause();
        writable.once('drain', () => readable.resume());
      }
    });
    ```
    *   `.pipe()` handles backpressure but not errors. `pipeline` handles both and cleans up all streams.
    ```javascript
    const { pipeline } = require('stream/promises');
    // inside an async function
    await pipeline(fs.createReadStream(src), zlib.createGzip(), fs.createWriteStream(dest));
    ```

## Common mistakes
*   **`fs.readFile` on big files**: Loads everything into memory. Use a stream.
*   **Sync calls in request handlers**: `readFileSync` or `pbkdf2Sync` stall every other request.
*   **Ignoring `write()` returning `false`**: Memory grows until the process crashes.
*   **Restarting crashed workers forever**: Add a restart limit to avoid a crash loop.
*   **Leftover child processes and workers**: Not stopping them when done or on shutdown leaks memory and file handles.

# Operating systems and networking

## Overview
Every service runs on the kernel and the network stack. The useful parts: how the OS waits on many connections, how memory behaves when it runs short, what context switches cost, and what each step of a connection adds to latency.

## Key concepts
*   **Blocking I/O**: The thread waits until data arrives.
*   **Non-blocking I/O**: The call returns at once if no data is ready.
*   **I/O multiplexing (epoll, kqueue)**: One thread watches many sockets and is told which are ready to read or write. Node.js (via libuv) and Go use this.
*   **Completion-based I/O (io_uring, Windows IOCP)**: The kernel does the read or write and reports when it is done.
*   **Virtual memory**: Each process sees its own address space; the OS maps it to physical RAM in pages.
*   **Page fault**: A program touches a page that isn't in RAM yet. A minor fault is fixed in RAM. A major fault reads from disk and is much slower.
*   **Context switch**: The CPU saves one thread's state and loads another's. Many runnable threads waste time switching.
*   **TLS 1.3**: The handshake takes one round trip (TLS 1.2 takes two). Resumed sessions can send 0-RTT data, which can be replayed, so use it only for safe requests.
*   **HTTP/3 (QUIC over UDP)**: A lost packet stalls only its own stream. In HTTP/2 over TCP, it stalls every stream on the connection.

## Real-world usage
*   **Proxies (Nginx, Envoy)**: A few worker processes or threads, each with an epoll event loop, hold huge numbers of connections.
*   **Database servers**: Give them enough RAM to keep hot data in memory and avoid major page faults.
*   **Video calls and games**: UDP (e.g., WebRTC), since a late packet is useless. Video on demand (HLS, DASH) mostly runs over HTTP on TCP.
*   **Container memory limits**: A Kubernetes limit makes the kernel kill the process inside that container, not starve the whole node.

## Tradeoffs
*   **Threads vs. event loop**: Thread-per-connection code is simple but each thread costs memory. Event loops handle many idle connections but are harder to write, and CPU-heavy work blocks the loop.
*   **TCP vs. UDP**: TCP gives a reliable, ordered stream with congestion control, plus handshake delay and head-of-line blocking. UDP sends separate packets; the app handles loss and order (QUIC does this).
*   **TLS cost**: Extra round trips and CPU for key exchange. TLS 1.3, session resumption, and reused connections keep it small.

## When not to use
*   **Raw sockets for business APIs**: Use HTTP, gRPC, or WebSocket libraries.
*   **Kernel tuning by default**: Leave settings like `tcp_max_syn_backlog` alone until measurements show a problem.

## Interview focus
*   **Typing `https://example.com`**: "What happens?"
    * DNS: check browser and OS caches, then ask a resolver (the ISP's, or a public one like `8.8.8.8`), which asks the root, TLD, and domain's servers for the IP.
    * TCP: SYN, SYN-ACK, ACK. One round trip.
    * TLS: agree on keys (ECDHE). The server sends its certificate; the browser checks the chain and the hostname. ALPN picks HTTP/2 or HTTP/1.1.
    * HTTP: send `GET /`, get the page, fetch more resources, often on the same connection.
    * With HTTP/3, one QUIC handshake over UDP replaces TCP + TLS.
    * The kernel handles buffers, resends lost packets, and wakes the browser when data arrives.
*   **Node.js vs. thread-per-request Apache**: "Why can Node hold more connections?"
    * Thread-per-request: each connection holds a thread that sits blocked while waiting. Memory grows with connections.
    * Node: one thread runs an event loop. Sockets are non-blocking; epoll or kqueue says which are ready, and the loop runs their callbacks.
    * File system calls, DNS lookups, and some crypto run on libuv's small thread pool.
    * Idle connections are cheap, but one CPU-heavy callback stalls them all.
*   **Thrashing**: "What is it, and how do you spot it?"
    * Hot data no longer fits in RAM, so the system spends its time moving pages to and from disk.
    * Check major page faults and swap-in/out rates (`vmstat` `si`/`so`, `sar -B`), not just how much swap is used.
    * In containers without swap, it shows as OOM kills or constant re-reading of files.
    * Fix: use less memory (smaller caches, fix leaks, tune GC), add RAM, run fewer processes per host.

## Common mistakes
*   **Free local IPC**: Talking between processes on one machine still costs system calls, copies, and context switches.
*   **HTTP/1.1 queuing**: One request at a time per connection, so requests wait behind slow ones. HTTP/2 multiplexes; HTTP/3 also removes TCP-level blocking.
*   **Trusting `localhost`**: Anything on the host can reach services bound to it, including an SSRF attack through your own app.

# HLD Fundamentals: Scalability, CAP, & Fault Tolerance

### 1. Overview
These are the basic limits every design is judged against: how the system grows, what happens when a part fails, and which data guarantees it drops during a failure.

### 2. Key Concepts
*   **Vertical scaling**: Use a bigger machine. Simple, but there is a size limit and it is still one point of failure.
*   **Horizontal scaling**: Add more machines behind a load balancer. Works best when services are stateless.
*   **Stateless service**: Keeps no per-user data between requests. Sessions live in a shared store or a token, so any instance can serve any request.
*   **Stateful service**: Holds data or connections (databases, WebSocket servers). Needs sharding, sticky routing, or replication to scale.
*   **Latency**: Time for one request. Track p95/p99, not the average. The slowest requests are what users notice.
*   **Throughput**: Requests handled per second (RPS).
*   **L4 load balancer**: Routes TCP connections. Fast, but can't see URLs or headers.
*   **L7 load balancer**: Routes HTTP requests by path, header, or cookie. Can also end TLS and retry. Cloud load balancers can spread traffic across zones and regions.
*   **Balancing algorithms**: Round-robin (take turns), least connections (pick the least busy), consistent hashing (same key goes to the same node).
*   **Health checks**: The load balancer stops sending traffic to instances that fail them.
*   **Session affinity**: Sends a user back to the same instance. Works, but fights statelessness.
*   **Autoscaling**: Add or remove instances based on CPU, request rate, or queue length.
*   **CAP theorem**: During a network partition (nodes can't reach each other), a data store must choose:
    *   *Consistency (C)*: Every read sees the latest write, or gets an error.
    *   *Availability (A)*: Every working node answers, even if the data may be stale.
    *   Partitions are not optional; networks fail. So "pick two of three" is wrong. The real choice is C or A while the partition lasts.
    *   *CP example*: etcd or ZooKeeper refuse writes on the minority side.
    *   *AP example*: Cassandra at low consistency levels keeps answering and fixes differences later.
    *   Many databases let you tune this per request, so CP/AP describes a setup, not a product.
*   **PACELC**: Adds the normal case. If Partition: choose A or C. Else: choose Latency or Consistency. Waiting for replicas on every write costs time even when nothing is broken.
*   **Fault tolerance**: The system keeps working, maybe in a reduced mode, when parts fail. Main tools:
    *   *Redundancy*: Two or more of everything important, across zones.
    *   *Failover*: Detect a dead node and move traffic or leadership to a healthy one.
    *   *Timeouts, retries, circuit breakers*: Stop a slow dependency from dragging callers down. See [reliability patterns](../4-distributed-systems/reliability-patterns.md).
    *   *Idempotency*: Doing an operation twice has the same effect as once, so retries are safe.
    *   *Isolation*: Split the system into independent parts so one failure stays small.
    *   *Consensus*: Raft or Paxos for leader election and agreeing on state. See [distributed core](../4-distributed-systems/distributed-core.md).
    *   *Monitoring*: Heartbeats, metrics, and alerts to catch failures early.
    *   *Graceful degradation*: Show cached data if the DB is down. Turn off recommendations but keep checkout working.
    *   *Costs*: More retries mean more load. More replicas mean more money.

### 3. Real-World Usage
*   **Sale traffic spikes**: Web servers autoscale out. The database is scaled ahead of time and protected by caches and queues.
*   **Social likes and comments**: Prefer availability. A comment showing up 2 seconds late is fine.
*   **Money transfers**: Prefer consistency. If the balance can't be checked, reject the transfer.
*   **Video**: On-demand video is served in large chunks from a CDN for throughput. Live video uses shorter chunks (or WebRTC) for lower delay.

### 4. Tradeoffs
*   **Vertical vs. horizontal**: Vertical is simpler until you hit the size limit. Horizontal needs load balancing and stateless design.
*   **Consistency vs. latency**: Strong consistency waits for other nodes on each write. Eventual consistency is faster but allows stale reads.
*   **Availability vs. complexity**: Running in two regions at once raises uptime but adds replication and conflict handling.

### 5. When NOT to Use
*   **Many microservices for a small app**: One service and a replicated database is enough for a low-traffic CRUD app.
*   **Strong consistency everywhere**: Not needed for profile pictures or view counts.

### 6. Interview Focus
*   **Estimation**: "10M daily users each upload 2 photos. Storage?"
    * 20M photos/day. At ~2 MB each, ~40 TB/day before replication.
    * Photos are viewed many times, so outgoing traffic is a multiple of that. Serve reads from a CDN.
    * Say your assumptions out loud.
*   **Chat app store choice**: "Why lean AP?"
    * A late message is better than a failed send.
    * Order inside one chat can still be kept with per-chat sequence numbers.
    * Don't say "NoSQL = AP, SQL = CP". Many stores are tunable, and SQL read replicas can be stale.
*   **Slow system, healthy load balancer**: "Where do you look?"
    * Use tracing to see which hop is slow.
    * Then check saturation: CPU, memory, DB connection pool, queue depth.
    * Usual causes: slow queries or locks, a dependency with no timeout, GC pauses, network I/O.
*   **Reducing latency**:
    * Cache close to the user (CDN, local cache).
    * Fix slow queries and N+1 query patterns.
    * Move non-urgent work to a queue.
    * Deploy in regions near users.
*   **Keeping data consistent across services**:
    * Strong consistency only where needed: one leader or consensus for that data.
    * Elsewhere: idempotent operations, the outbox pattern, and sagas for multi-step workflows.
*   **Part of the system is down**:
    * Timeouts and circuit breakers stop the failure from spreading.
    * Serve a reduced version (cached data, read-only mode).
    * Queues hold work until the failed part recovers.

### 7. Common Mistakes
*   **Ignoring partitions**: To other nodes, a long GC pause looks the same as a network split.
*   **"Pick two" CAP**: Say C vs. A during a partition, and latency vs. consistency otherwise.
*   **Wrong metric**: Cutting average latency when the real problem is the system falling over under load.
*   **Single point of failure**: Scalable app servers in front of one database with no tested failover.

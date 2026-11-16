# Distributed Core: Consensus & Coordination

### 1. Overview
Coordination is how separate machines agree on a value or an order of operations, even when some crash or the network fails. It is how a cluster keeps one source of truth.

### 2. Key Concepts
*   **Consensus**: Nodes agree on the same ordered list of operations, so every copy applies the same changes. Raft and Paxos keep working while a majority is up.
*   **Leader election**: One node coordinates writes. If its heartbeats stop, the others elect a new one.
*   **Replication**: Copies of data on several nodes. Synchronous waits for the copies (safer, slower). Asynchronous doesn't (faster, can lose recent writes on failover).
*   **Quorum**: How many nodes must agree, usually a majority: `floor(N/2) + 1`. Two majorities always overlap, so decisions don't conflict.
*   **Service discovery**: How a service finds healthy instances of another without hard-coded IPs. Instances register, health checks remove dead ones, clients look them up. Examples: Consul, Kubernetes Services. Needed when autoscaling keeps changing instances. Example: the checkout service asks the registry for a healthy payment instance instead of using a fixed IP.

### 3. Real-World Usage
*   **Config and discovery stores**: etcd and Consul use Raft. Kubernetes keeps all cluster state in etcd.
*   **Distributed locks**: ZooKeeper or etcd so only one worker runs a job at a time.
*   **Database failover**: MongoDB replica sets elect a new primary on their own. PostgreSQL has no built-in automatic failover; tools like Patroni add it.
*   **Kafka**: An elected controller manages partition leaders. Newer versions run controllers as a Raft group (KRaft) instead of ZooKeeper.

### 4. Tradeoffs
*   **Consistency vs. latency**: Each write waits for a majority. Slower than a local write, much slower across regions.
*   **Availability vs. safety**: If a majority is lost (3 of 5 nodes), the cluster stops taking writes rather than risk two leaders.
*   **Raft vs. Paxos**: Raft was designed to be easier to understand and is widely used (etcd, Consul, CockroachDB). Paxos came first and is hard to implement correctly.

### 5. When NOT to Use
*   **Single-node setups**: One database with backups needs no consensus.
*   **Approximate counters**: View counts don't need majority writes.
*   **Writing your own**: Use etcd, ZooKeeper, or the database's built-in replication.

### 6. Interview Focus
*   **Split-brain** (two nodes both think they're leader):
    * Election needs a majority, so only one side can win.
    * Fencing tokens (also called a generation clock or term): each leader gets a higher number; storage rejects writes with an older number.
*   **Why odd cluster sizes**:
    * 3 nodes survive 1 failure. 4 nodes still survive only 1. 5 survive 2.
    * The extra even node adds cost without adding safety.
*   **How a new Raft leader has all committed data**:
    * An entry counts as committed once a majority stores it.
    * Nodes only vote for a candidate whose log is at least as up to date as theirs.
    * Any winning majority overlaps the committing majority.
*   **Client-side vs. server-side discovery**: Client-side means the client picks the instance (fewer hops). Server-side means a load balancer or mesh picks it (simpler clients).

### 7. Common Mistakes
*   **Manual failover**: Waiting for a person to promote a replica at 3 a.m.
*   **Timeouts too short**: Normal network jitter or GC pauses trigger elections over and over.
*   **Replication as backup**: A `DROP TABLE` replicates instantly. Keep real backups.

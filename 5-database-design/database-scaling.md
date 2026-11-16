# Database Scaling: Replicas, Sharding, & Partitioning

### 1. Overview
Scaling a database means handling more data and traffic than one server can. Usual order: tune the single server, buy a bigger one, add read copies, then split the data across servers. Each step adds capacity and also adds complexity.

### 2. Key Concepts
*   **Read replica**: A copy of the primary database that serves reads. All writes still go to the primary.
*   **Sharding**: Splitting rows across separate database servers. Each server (shard) holds part of the data. Built into MongoDB and CockroachDB; Vitess adds it to MySQL.
*   **Table partitioning**: Splitting one big table into parts inside the same database, e.g., one partition per month. Old months are cheap to drop, and queries on a date range skip other partitions.
*   **Partitioning vs. sharding**: Partitions live on one server; shards live on different servers. Partitioning helps manage big tables. Only sharding adds write capacity.
*   **Partition types**: Range (by date), list (by fixed values like `'US'`, `'EU'`), hash (spread by a hash of a key), or a mix.
    ```sql
    CREATE TABLE transactions (
        id         BIGSERIAL,
        user_id    INT NOT NULL,
        amount     NUMERIC(10, 2),
        created_at DATE NOT NULL,
        PRIMARY KEY (id, created_at)  -- must include the partition column
    ) PARTITION BY RANGE (created_at);

    CREATE TABLE transactions_2024_01 PARTITION OF transactions
        FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');  -- upper bound excluded
    ```
*   **Vertical partitioning**: Splitting a table by columns, e.g., moving a large `profile_image` out of `users`.
*   **Shard key**: The column that decides which shard a row goes to. Common choices: user ID, tenant ID, region. Example: `user_id % 4` picks shard 0 to 3.
*   **Federation**: One database per business area: users, orders, products.

### 3. Real-World Usage
*   **Read-heavy feed**: One primary for writes, several replicas for reads.
*   **Multi-tenant SaaS**: Shard by tenant ID, so most queries hit one shard.
*   **Search**: Product text and reviews go to a search engine (e.g., Elasticsearch), keeping order tables small.
*   **Disaster recovery**: A standby replica in another region, promoted if the main region fails.

### 4. Tradeoffs
*   **Replica lag**: Replication is usually asynchronous. A read right after a write can hit a replica that doesn't have the write yet ([Replication Lag](../4-distributed-systems/consistency-models.md)). Synchronous replication fixes this but makes writes slower.
*   **Sharding cost**: Joins across shards become several queries merged in code. Transactions across shards need two-phase commit or sagas. The app or a proxy must route each query to the right shard. Backups, restores, and moving data between shards get harder.
*   **Hot shard**: A bad shard key sends most traffic to one shard while others sit idle.

### 5. When NOT to Use
*   **Sharding**: Last resort. Try query tuning, indexes, a bigger server, caching, and replicas first.
*   **Replicas for write-heavy load**: Replicas only add read capacity. Every replica must apply every write, so they don't help writes and they fall behind.

### 6. Interview Focus
*   **Scaling steps**: "Our SQL database is at its limit. What do you do, easiest first?"
    * Find the bottleneck: slow queries, missing indexes, locks, too many connections.
    * Tune queries and indexes. Add a connection pooler (e.g., PgBouncer).
    * Scale up: more CPU, enough RAM to hold the hot data, faster disks.
    * Cache hot reads in Redis or Memcached.
    * Add read replicas and handle lag.
    * Use materialized views or denormalized tables for heavy repeated queries.
    * Partition tables, split by business area, and shard last.
*   **Hot shard**: "A ride app shards by `city_id`. One big city gets most of the traffic. How do you fix it?"
    * Use a smaller unit than a city, e.g., a map grid cell (geohash).
    * Or split the hot city: key on `(city_id, hash(ride_id) % N)`. City-wide queries then ask N shards.
    * Or keep a lookup table from key to shard, so a hot city can move to its own hardware.
*   **Replication lag**: "A user edits their profile and still sees the old one."
    * The write went to the primary; the read hit a replica that is behind.
    * Read the user's own data from the primary for a short time after they write.
    * Or remember the primary's log position after the write, and only read from a replica that has reached it.
    * Or use synchronous replication for this path, accepting slower writes.
*   **High-read, low-write system**: "How would you design one?"
    * Cache hot reads (Redis) and put static content on a CDN.
    * Add read replicas behind a load balancer.
    * Precompute expensive views (materialized views, denormalized tables).
    * Writes stay on one primary, since there are few of them.

### 7. Common Mistakes
*   **Sharding too late**: Moving to shards while the database is already overloaded is much harder. The move (dual writes, backfill, cutover) needs time and spare capacity.
*   **Untested failover**: Replicas exist, but nobody has practiced promoting one or reconnecting clients.
*   **Modulo sharding**: With `hash(id) % N`, adding a shard moves most keys. Use consistent hashing, or many logical shards mapped onto fewer servers.

# NoSQL databases

## Overview
NoSQL means databases that don't use relational tables as their main model. They differ a lot from each other. Pick one by matching its data model to how you will read and write the data, and know what you give up (joins, wide transactions, ad hoc queries).

## Key concepts
*   **Key-value**: Get a value by its key (Redis, DynamoDB). Very fast lookups; little querying by value.
*   **Document**: JSON-like records with flexible fields and secondary indexes (MongoDB, Couchbase). Good when one entity is read and written as a unit.
*   **Wide-column**: Rows grouped by a partition key and sorted inside it (Cassandra, HBase). High write throughput; fast reads within one partition. Not the same as a columnar store (ClickHouse, Redshift), which stores each column apart for analytics.
*   **Graph**: Nodes and edges (Neo4j). Good for following relationships several hops deep.
*   **NewSQL**: SQL and ACID transactions with automatic sharding across servers (CockroachDB, Google Spanner, YugabyteDB). Costs more latency per write than a single SQL server.

## Real-world usage
*   **Sessions, leaderboards**: Redis keys with TTLs and sorted sets.
*   **Product catalog**: Documents, since a shirt and a laptop have different fields.
*   **Sensor data**: Cassandra, one partition per device per time bucket, read by time range.
*   **Fraud, recommendations**: A graph to find friends of friends, or money paths between accounts.

## Tradeoffs
*   **Flexibility vs. integrity**: No fixed schema or foreign keys, so the app must check structure. MongoDB and DynamoDB have multi-record transactions, with more limits than SQL.
*   **Consistency is a setting**: Cassandra sets it per query. DynamoDB has strong or eventual reads. MongoDB with majority reads and writes acts as CP. "NoSQL means AP" is too simple.
*   **Queries designed up front**: A new query often needs a new index, a new table, or copied data.

## When not to use
*   **Many-record transactions**: Ledgers and similar data are simpler and safer in SQL.
*   **Unknown queries**: Key-value and wide-column stores need query-first design. If you don't know the queries yet, use SQL.

## Interview focus
*   **Graph vs. SQL**: "Why might a graph database beat SQL for a connection graph?"
    * In SQL, each hop is another self-join. Two hops are fine; "within 4 hops" needs a recursive CTE and gets slow.
    * A graph DB stores each node's edges directly, so a hop costs about the number of edges followed, not the total graph size.
    * Results still grow with each hop's fan-out, in both systems.
    ```cypher
    MATCH (p:Person {name: 'Alice'})-[:FRIEND]->()-[:FRIEND]->(fof:Person)
    WHERE fof <> p
    RETURN DISTINCT fof
    ```
*   **DynamoDB hot partition**: "Design a social feed. How do you avoid hot partitions?"
    * Partition key per user, sort key per item: `PK = USER#123`, `SK = POST#<timestamp>`. One query returns a user's items in order.
    * `userId` is usually a good key. One very active account can still exceed a partition's limit.
    * Write sharding: add a suffix, `USER#123#0` to `USER#123#9`. Writes spread over 10 keys; reads query all 10 and merge.
*   **MongoDB write concern**: "Why can MongoDB confirm a write that isn't on all replicas?"
    * With `w: 1`, only the primary confirms. If it fails before copying, the write can be rolled back.
    * Since MongoDB 5.0 the default is usually `w: "majority"`. Older versions and explicit settings may use `w: 1`.
    * Use `{ writeConcern: { w: "majority", wtimeout: 5000 } }` for writes that must survive failover. A timeout returns an error but does not undo the write.
*   **CAP**: "Which side do you pick for a bank ledger?"
    * Partitions will happen. During one, choose consistency (refuse some requests) or availability (answer with maybe-stale data).
    * Ledger: CP. The majority side keeps working; the minority side rejects writes. Some users see errors.
*   **Expiring data**: "How do you remove old MongoDB documents automatically?"
    * TTL index: an index on a date field with `expireAfterSeconds`. A background task deletes expired documents periodically, not instantly.
    * Capped collection: fixed size, insertion order, oldest documents overwritten when full. MongoDB's docs prefer TTL indexes (or time series collections) in most cases.
    ```javascript
    db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
    ```

## Common mistakes
*   **One database for everything**: Forcing relational data into documents, or keeping the only copy of critical data in a cache.
*   **No schema at all**: Many shapes of "user" pile up in one collection. Validate in the app or with schema validation.
*   **Scanning**: The key or index doesn't match the query, so every read scans. Slow, and costly on pay-per-request stores.

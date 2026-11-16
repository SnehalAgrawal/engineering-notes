# Stream Processing: Exactly-Once & Backpressure

### 1. Overview
Stream processing handles data as it arrives, instead of running jobs over a fixed dataset (batch). The hard parts are keeping state correct after crashes, handling late events, and coping when input is faster than processing.

### 2. Key Concepts
*   **Unbounded data**: A stream with no end: clicks, sensor readings, logs, database change events.
*   **Event time vs. processing time**: When it happened vs. when the processor saw it. Correct results use event time.
*   **Tumbling window**: Fixed, non-overlapping blocks (every minute).
*   **Sliding window**: Fixed size, overlapping (last 10 minutes, updated every minute).
*   **Session window**: Ends after a gap of inactivity (one user visit).
*   **Watermark**: The processor's estimate of how far event time has progressed. A watermark at T means "no more events at or before T expected", so windows up to T can close. Events behind it are late.
*   **Stateful processing**: Keeping data between events (counts, averages), split by key and saved by checkpoints.
*   **Exactly-once processing**: After a crash, the results count each event once. Done with checkpoints of state plus input offsets, and sinks that are transactional or idempotent.
*   **Backpressure**: A slow stage makes earlier stages slow down instead of piling up data until memory runs out. With Kafka as input, the backlog shows as consumer lag.
*   **Schema registry**: Stores message formats (Avro, Protobuf) and rejects changes that would break consumers.

### 3. Real-World Usage
*   **Fraud detection**: Flag a card used in several countries within a short time.
*   **Live dashboards and alerts**: Per-minute metrics, error-rate alerts.
*   **IoT**: Aggregate sensor data and spot anomalies (see [worked example](../3-hld/example-high-throughput-ingestion.md)).
*   **Ad tech**: Count impressions and clicks live to adjust bids.

### 4. Tradeoffs
*   **Throughput vs. latency**: Bigger batches process more per second but delay each event.
*   **Exactly-once vs. speed**: Checkpoints and transactions add delay compared with at-least-once.
*   **Complete vs. fast**: Waiting longer for late events gives more accurate results, later.
*   **Large state**: Makes checkpoints, recovery, and rescaling slow.

### 5. When NOT to Use
*   **Historical analysis**: Reports over years of data belong in batch jobs. See [data pipelines](./data-pipelines.md).
*   **Normal CRUD**: Creating a user belongs in a service with a database.

### 6. Interview Focus
*   **Late or out-of-order events**:
    * Watermark = highest event time seen minus a grace period.
    * Late events can update earlier results, go to a side output, or be dropped and counted.
    ```javascript
    maxSeen = Math.max(maxSeen, e.timestamp);
    const watermark = maxSeen - 5000; // 5 s grace period
    if (e.timestamp <= watermark) handleLate(e); else addToWindow(e);
    ```
*   **24-hour moving average for 1M keys** (prompt scale):
    * Key the stream (`keyBy(userId)`) so each instance owns some keys.
    * Use a disk-backed state store (RocksDB in Flink) with regular checkpoints.
    * Keep a sum and count per time bucket instead of every event.
    * Rescale from a savepoint; state is redistributed.
*   **Processor falling behind**:
    * Find the slow stage: source, an operator, or the sink. Flink's UI shows backpressure per operator.
    * Fix it: batch calls to external systems, speed up state access, fix a hot key.
    * Scale out. Parallelism is capped by the Kafka partition count, and adding partitions moves keys.
*   **Pipeline questions** (clickstream design, schema changes, data integrity): see [data pipelines](./data-pipelines.md).

### 7. Common Mistakes
*   **Using processing time**: Events don't arrive in order. Use the event's own timestamp.
*   **No checkpoints**: A crash loses all running totals.
*   **Unbounded state**: Week-long windows with a one-second slide, or keys that never expire. Use running aggregates and state TTLs.

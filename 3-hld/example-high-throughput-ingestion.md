# Example: high-throughput ingestion

Prompt (hypothetical scale): "10 million sensors each send one reading per second. Ingest everything, show live dashboards, keep history for analytics."

That is about 10M messages/sec. Ask for the payload size to get bandwidth and storage. No single node or database handles this, so every layer must be split into partitions.

## Device connections
*   **MQTT**: The usual protocol for small devices. Small headers, long-lived connections, delivery levels (QoS).
*   **HTTPS**: For devices that upload in batches.
*   **WebSockets**: When the device also needs to receive commands in real time.
*   **MQTT broker cluster**: EMQX, HiveMQ, or a managed IoT service holds millions of connections behind a load balancer. Mosquitto suits small setups.
*   **Not straight to Kafka**: Kafka is built for fewer, busy clients, not millions of small ones. The broker forwards into Kafka.
*   **At the edge**: Authenticate each device, drop bad payloads, batch before forwarding.

## Buffer
*   **Kafka** (or Kinesis, Pub/Sub, Event Hubs): Separates ingestion from processing and absorbs bursts.
*   **Partition by device ID**: Keeps each device's readings in order and spreads load evenly. Partitioning by region creates hot partitions.
*   **Schema registry**: Producers and consumers can change message formats safely (Avro, Protobuf).

## Stream processing
*   **Engines**: Flink, Spark Structured Streaming, or Kafka Streams.
*   **Jobs**: Per-minute averages per zone, spike detection, sensors gone silent, adding location to each reading, removing duplicates.
*   **Event time**: Devices that were offline send late data. Use watermarks (see [stream processing](../4-distributed-systems/stream-processing.md)).
*   **Downsampling**: Dashboards rarely need per-second data. Aggregating early is the biggest cost saver.

## Storage
*   **Hot**: A time-series or analytics store (TimescaleDB, InfluxDB, ClickHouse, Druid) for recent data and aggregates.
*   **Cold**: Object storage (S3, GCS) as Parquet files, partitioned by date, then region or device type.
*   **Retention**: Raw data expires fast; aggregates are kept longer.

## Queries
*   **Ops dashboards**: Grafana on the hot store.
*   **Business dashboards**: Superset or Metabase on aggregates.
*   **Ad-hoc**: Trino, Athena, or Spark over the data lake. Druid for fast aggregations.
*   **Logs and text search**: Elasticsearch with Kibana.

## Scaling and operations
*   **Autoscale on lag**: Scale processors on consumer lag, not just CPU.
*   **Partition count**: It caps parallelism. Plan ahead, because adding partitions later moves keys.
*   **Backpressure**: Let Kafka hold spikes; alert on lag instead of dropping data.
*   **Monitoring**: Prometheus and Grafana for connected devices, ingest rate, lag, and processing time.

## Interview talking points
*   **Ordering**: Needed per device, not across devices.
*   **Duplicates**: MQTT QoS 1 and Kafka retries can repeat messages. Deduplicate on (device ID, sequence number) or make writes idempotent.
*   **Processor outage**: Kafka keeps the data; processors resume from their last committed offset.
*   **Cost**: Raw data volume dominates. Aggregate early and move old data to cheap storage.

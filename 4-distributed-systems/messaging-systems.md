# Messaging and queues

## Overview
Messaging lets services talk without calling each other directly. A broker sits in between, absorbs spikes, and holds work until a consumer is ready.

## Key concepts
*   **Pub/sub**: Every subscriber gets each message. For events like `OrderPlaced`.
*   **Queue (point-to-point)**: Each message goes to one worker in a pool. For jobs like `GenerateThumbnail`.
*   **At-most-once**: May lose messages, never duplicates.
*   **At-least-once**: Never loses them, may duplicate. The usual default, so consumers must be idempotent.
*   **Exactly-once**: True exactly-once delivery isn't possible over a network. Systems give exactly-once *effects*: at-least-once delivery plus deduplication or transactions.
*   **Dead letter queue (DLQ)**: Where a message goes after too many failed tries, so it doesn't block the rest.
*   **Log (Kafka)**: Messages stay on disk for a retention period. Each consumer group tracks its own position (offset), so many groups can read the same data and replay it. Order is kept per partition.
*   **Queue broker (RabbitMQ, SQS)**: Messages are removed once acknowledged. Per-message retries, routing, delays, and built-in DLQs. Simple for background jobs.
*   **Picking one**: Kafka for event streams, many consumers, replay, high volume. RabbitMQ/SQS for task queues with per-message retry.

## Real-world usage
*   **Background jobs**: Save the order, queue a message, a worker builds the invoice PDF.
*   **Log pipelines**: Services write to a Kafka topic; search, archive, and alerting each read it.
*   **Slow third parties**: Queue outgoing emails so a slow email provider doesn't slow users down.
*   **Notifications**: One `OrderPlaced` event; SMS, push, and email services each consume it and retry on their own.

## Tradeoffs
*   **Decoupling vs. debugging**: Following a request across queues needs correlation IDs and tracing.
*   **Latency vs. resilience**: A queue adds a hop but keeps requests working when a worker is slow or down.
*   **Ordering vs. parallelism**: Order holds only within a Kafka partition or an SQS FIFO message group. SQS standard queues don't keep order.

## When not to use
*   **Caller needs the answer now**: "Is this username taken?" is a direct query.
*   **Small apps**: A broker is one more thing to run. A jobs table in the database or a Redis-backed job queue (BullMQ in Node) may be enough.

## Interview focus
*   **Consumer crashes mid-processing**:
    * Acknowledge only after the work succeeds.
    * Unacknowledged messages come back (SQS visibility timeout, Kafka re-reads from the last committed offset).
    * Result: no loss, possible duplicates.
*   **Kafka exactly-once**:
    * Idempotent producers drop duplicate sends.
    * Transactions save output and the consumer offset together.
    * Effects outside Kafka (DB writes, emails) still need idempotency.
*   **Ordering and duplicates**:
    * Partition by the thing whose order matters (order ID, account ID).
    * Give each message an ID and record processed IDs in the same transaction as the work.
    * Or make the operation naturally safe to repeat ("set status = paid" instead of "add 1").
*   **DLQ handling**: Alert on DLQ size, find the cause, fix it, replay. Delete only truly invalid messages.

## Common mistakes
*   **Non-idempotent consumers**: Processing the same order twice.
*   **Big payloads**: Put the file in object storage and send its URL.
*   **Not watching consumer lag**: Lag grows until old data expires unread or the disk fills.

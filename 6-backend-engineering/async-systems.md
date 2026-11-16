# Async Systems: Job Queues & Backpressure

### 1. Overview
Async processing moves slow work out of the request. The API saves a job, replies at once, and a worker runs the job later. The hard parts: not losing jobs, safe retries, tracing failures, and coping when jobs arrive faster than workers finish them.

### 2. Key Concepts
*   **Job queue**: A tool that hands work to background workers (Celery, BullMQ, Sidekiq).
*   **Broker**: Where jobs wait (Redis, RabbitMQ, SQS).
*   **Worker**: A separate process that takes a job, runs it, and confirms (acks) it.
*   **At-least-once delivery**: Most queues may deliver a job twice, so handlers must be idempotent (safe to run twice).
*   **Backpressure**: Pushing back when work arrives faster than it is done: slow the producers, reject work (load shedding), or add workers.
*   **Visibility timeout**: SQS hides a taken job for a while. If the worker doesn't ack in time, the job reappears and is retried.

### 3. Real-World Usage
*   **Reports**: The API returns `202 Accepted` and a job ID. A worker builds the PDF; the user polls or gets notified.
*   **Webhooks**: Save the event and return 2xx fast, then process it. Providers like Stripe retry slow or failed deliveries.
*   **Bulk email**: One job per batch. If the email provider throttles, jobs wait and retry with backoff.
*   **Video**: Transcoding is slow and CPU-heavy, so it runs on its own workers.

### 4. Tradeoffs
*   **Fast reply, late result**: Success or failure arrives later and needs a status endpoint or a notification.
*   **Debugging**: One request now spans API, queue, and workers. You need correlation IDs and tracing.
*   **Idle cost**: Workers use memory even with an empty queue. Autoscale on queue size or age.

### 5. When NOT to Use
*   **Small, quick tasks**: Updating one row doesn't need a broker, workers, retries, and monitoring.
*   **Next step needs the result**: If the next screen must show the result, async forces polling or shows missing data.

### 6. Interview Focus
*   **Worker count**: "How many workers? CPU-bound or I/O-bound?"
    * CPU-bound: about one per core.
    * I/O-bound: many more, until a downstream limit (database connections, API rate limits) is hit.
*   **Worker crash**: "A worker dies mid-report. How does the report still get made?"
    * Ack only after the work is done.
    * The unacked job reappears after the visibility timeout and another worker retries it.
    * Make the job idempotent so a retry is safe.
*   **Huge queue**: "The queue hits a million jobs. How do you stop producers?"
    * Bound the queue. Return `429` or `503` from the API when it is full.
    * Rate-limit producers and autoscale workers.
    * Alert on the age of the oldest job, not just queue length.

### 7. Common Mistakes
*   **No dead-letter queue**: A job that always fails retries forever. Cap retries and move it to a dead-letter queue (DLQ).
*   **Not idempotent**: A worker sends an email, crashes before acking, and the retry sends it again. Record finished job IDs.
*   **Too many workers**: Hundreds of workers each open database connections and overload the database. Limit concurrency.

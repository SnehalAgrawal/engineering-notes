# Observability: Logs, Metrics, & Tracing

### 1. Overview
Observability is how well you can tell what a system is doing from the data it emits. Monitoring tells you that something is wrong. Observability helps you find out why, even for failures you have not seen before.

### 2. Key Concepts
*   **Logs**: Records of single events, such as "payment failed for order 42". Rich detail, but expensive to store and slow to search at volume.
*   **Metrics**: Numbers over time, such as request rate or error percentage. Cheap to keep and good for dashboards and alerts.
*   **Traces**: The path of one request through many services. Each step is a span with a start time and duration. Good for finding where latency comes from.
*   **Context propagation**: Services pass a trace ID along with each call (the W3C `traceparent` header) so spans join into one trace and logs can include the same ID.
*   **OpenTelemetry (OTel)**: The vendor-neutral standard for producing traces, metrics, and logs. It has SDKs per language, the OTLP protocol, and the Collector, which receives, processes, and forwards data to any backend.
*   **Prometheus**: An open-source metrics system. It pulls (scrapes) metrics from `/metrics` endpoints, stores them as time series, and evaluates alert rules. Alertmanager routes the alerts.
*   **Grafana**: Dashboards on top of Prometheus, Loki (logs), Tempo or Jaeger (traces), and many other sources.
*   **SLI**: Service Level Indicator. What you measure, for example the share of requests that succeed in under 300 ms.
*   **SLO**: Service Level Objective. The internal target for the SLI over a window, for example 99.9% over 30 days.
*   **Error budget**: The failure the SLO allows (100% minus the SLO). While budget is left, ship features. When it runs out, focus on reliability.
*   **SLA**: Service Level Agreement. A promise to customers with consequences such as service credits. It is usually looser than the internal SLO.

### 3. Real-World Usage
*   **Alerting**: Page on symptoms users feel, such as a high 5xx rate or fast SLO burn, not on every CPU spike.
*   **Latency debugging**: A trace shows a request is slow because Service B waits on a slow query in Service C.
*   **Audit trails**: Structured JSON logs let you find every action by one `user_id` during a security review.
*   **Capacity planning**: Months of memory and disk metrics show when a database needs a bigger instance.

### 4. Tradeoffs
*   **Log level vs. cost**: `DEBUG` logs in production give detail but cost a lot to ship and store. Log at `INFO` and turn on debug for a short time when needed.
*   **Trace sampling**: Keeping every trace is expensive. Head sampling decides at the start (keep a fixed share). Tail sampling decides after the trace ends, so it can keep all errors and slow requests.
*   **Metric cardinality**: Each unique label combination is a new time series. Labels like `user_id` or `request_id` blow up cost and memory. Put those in logs or traces.
*   **Build vs. buy**: Self-hosted stacks (Prometheus, Grafana, Loki, OpenSearch) give control but need people to run them. SaaS (Datadog, New Relic, Grafana Cloud) is faster to start and costs more as volume grows.

### 5. When NOT to Use
*   **Distributed tracing for a single process**: A monolith with one database gets more from logs and a profiler.
*   **Pushing metrics by default**: Prometheus-style pulling scales well for long-running services. Push (Pushgateway, OTLP, StatsD) fits short-lived jobs and places the server cannot reach.

### 6. Interview Focus
*   **Incident debugging**: "The site is slow. You have CPU and error rate dashboards. How do you find the cause?"
    *   Start with user impact: error rate and latency percentiles. Confirm it is real and see when it started.
    *   Ask what changed at that time: deploys, config changes, feature flags, traffic shape.
    *   Check saturation: CPU, memory, connection pools, queue depth. Check restarts and OOM kills in Pod events too.
    *   Check dependencies: database latency, cache hit rate, external APIs.
    *   Use traces and logs from the bad time window to find the slow span or the failing call.
    *   Mitigate first (roll back, scale, shed load), then find the root cause.
*   **Methods**: "What do you measure for a service?"
    *   RED for request-driven services: Rate, Errors, Duration.
    *   USE for resources: Utilization, Saturation, Errors.
    *   Google's four golden signals: latency, traffic, errors, saturation.
*   **Metrics design**: "What would you track for a background job worker?"
    *   Queue depth and the age of the oldest job (best signal of falling behind).
    *   Job duration as a histogram, so you get p50, p95, and p99.
    *   Success, failure, and retry counts, plus dead-letter queue size.
    *   Busy vs. idle workers, and memory per worker.
*   **Monitoring vs. observability**: "What is the difference?"
    *   Monitoring checks known failure modes with fixed thresholds, like a car's check-engine light.
    *   Observability means the data is rich enough (high-cardinality fields, traces, linked logs) to answer new questions, like a mechanic's diagnostic tools.

### 7. Common Mistakes
*   **Alert fatigue**: Too many noisy alerts, so people miss the real one. Every page should need a human action.
*   **Unstructured logs**: `Error in user logic!` instead of `{"event":"login_failed","user_id":123,"reason":"bad_password"}`.
*   **Watching causes, not symptoms**: Tracking CPU but not "login failure rate". Users feel the second one.
*   **Averages only**: An average hides the slow tail. Use percentiles from histograms.
*   **No trace ID in logs**: Without it you cannot jump from a slow trace to its logs.

### 8. Tools
*   **Metrics**: Prometheus with Grafana, or a managed equivalent.
*   **Logs**: ELK/OpenSearch, Loki.
*   **Traces**: Jaeger, Tempo, or a vendor backend fed through OpenTelemetry.
*   **Errors and APM**: Sentry for exceptions; Datadog, New Relic, and Azure Application Insights for full application performance monitoring.

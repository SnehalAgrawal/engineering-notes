# Data pipelines

## Overview
A data pipeline moves data from where it's created (apps, databases, events) to where it's analyzed (warehouse, data lake). It can run in batches on a schedule or continuously as a stream. For streaming details see [stream processing](./stream-processing.md).

## Key concepts
*   **ETL (extract, transform, load)**: Clean and reshape data before loading it into the warehouse. Older pattern, used when the target is expensive or strict.
*   **ELT (extract, load, transform)**: Load raw data first, then transform inside the warehouse with SQL (often using dbt). Common with cloud warehouses like BigQuery, Snowflake, Redshift.
*   **Batch**: Process a fixed chunk on a schedule (nightly sales report). Tools: Spark, warehouse SQL.
*   **Streaming**: Process events as they arrive (fraud checks, live metrics). Tools: Kafka with Flink, Spark Structured Streaming, or managed services (Azure Stream Analytics, Kinesis).
*   **CDC (change data capture)**: Read a database's change log (e.g., with Debezium) and send each insert, update, and delete as an event. Keeps other systems in sync without bulk exports.
*   **Orchestration**: Runs jobs in the right order, on schedule, with retries. Apache Airflow models a pipeline as a DAG (a graph of tasks with no cycles). Managed options: Azure Data Factory, AWS Step Functions, Google Cloud Composer.
*   **Schema evolution**: Handling format changes without breaking the pipeline. Avro or Protobuf plus a schema registry with compatibility rules.
*   **Data lineage**: A record of where each dataset came from and which jobs changed it. Helps debug bad numbers and judge the impact of a change. Tools: OpenLineage, Microsoft Purview, dbt docs.
*   **Pipeline observability**: Freshness (how old is the latest data), volume, job duration, failures, and consumer lag for streams.

## Real-world usage
*   **Nightly reporting**: Airflow runs extract jobs, then dbt models, then refreshes dashboards.
*   **Database to analytics sync**: CDC from the production database into Kafka, then into the warehouse.
*   **Clickstream**: Events into Kafka, live aggregates by a stream job, raw events landed in a data lake for batch analysis.

## Tradeoffs
*   **ETL vs. ELT**: ELT is simpler and keeps raw data for reprocessing, but stores more and needs access controls on raw data. ETL keeps the warehouse clean but makes reprocessing harder.
*   **Batch vs. streaming**: Batch is simpler and cheaper; results are hours old. Streaming gives fresh results but is harder to run and debug.
*   **Managed vs. self-hosted orchestration**: Managed means less ops work but less flexibility and some lock-in.

## When not to use
*   **Streaming**: When a daily report is all anyone needs.
*   **Heavy orchestration**: A single cron job doesn't need Airflow.

## Interview focus
*   **Clickstream pipeline**: Kafka for ingestion, Flink for live aggregates, a data lake or warehouse for queries, a nightly batch job for heavy reports.
*   **Schema changes**:
    * Schema registry with a compatibility mode (backward or full).
    * Add new fields with defaults; don't rename or reuse fields.
    * Consumers should ignore fields they don't know.
*   **Data integrity across stages**:
    * Make each step idempotent so reruns don't duplicate data (overwrite a date partition instead of appending).
    * Checkpoints and retries; exactly-once where the engine supports it.
    * Data checks: row counts against the source, null rates, uniqueness (dbt tests, Great Expectations).
    * Alert on freshness and lag, not only job failures.
*   **Backfills**: Rerun a date range with the same idempotent jobs. Orchestrators like Airflow support this directly.

## Common mistakes
*   **Appending on rerun**: A retried job doubles the data.
*   **No freshness alerts**: The job "succeeded" but loaded nothing.
*   **Silent schema drift**: A source column changes type and breaks reports days later.
*   **No lineage**: Nobody knows which dashboards break when a table changes.

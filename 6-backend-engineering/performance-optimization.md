# Backend performance

## Overview
Most slow backends waste work: extra round trips, queries that read far more rows than they return, and time spent turning data into JSON. These patterns keep response times flat as data grows.

## Key concepts
*   **Offset pagination**: `ORDER BY id LIMIT 10 OFFSET 100`. Simple, but the database still reads and skips the first 100 rows, so deep pages get slow.
*   **Cursor (keyset) pagination**: `WHERE id > :last_id ORDER BY id LIMIT 10`. Starts right after the last row seen, using the index. For a non-unique sort column, add a tie-breaker: `ORDER BY created_at DESC, id DESC`.
*   **N+1 queries**: One query loads N posts, then N more queries load each author. Fix with a join or one `IN (...)` query.
*   **Lazy loading**: Load data only when it is used.
*   **Compression**: Gzip or Brotli on JSON and other text.

## Real-world usage
*   **Feeds**: Cursor pagination on `(created_at, id)`, so new posts don't cause duplicates or skipped items.
*   **ORMs**: Django `select_related` (join) and `prefetch_related` (second `IN` query), or Eloquent `with()`, turn N+1 into one or two queries.
*   **Product pages**: Load reviews only when the user scrolls to them.
*   **Large JSON**: Repetitive JSON compresses well; this helps most on slow mobile networks.

## Tradeoffs
*   **Eager vs. lazy**: Eager may load unused data. Lazy may cause many small queries.
*   **Offset vs. cursor**: Offset allows "jump to page 50". Cursor is fast but only does next and previous.
*   **Pooling**: Reusing connections saves setup time but holds idle resources.

## When not to use
*   **Lazy loading in batch jobs**: A loop over millions of rows becomes millions of queries. Load in chunks, eagerly.
*   **Compressing tiny responses**: CPU cost beats the savings, and output can even grow. Servers usually skip small bodies.

## Interview focus
*   **101 queries per page**: "What's wrong?"
    * N+1: one list query plus one per item.
    * Eager-load with a join or one `IN` query.
*   **`OFFSET 1000000`**: "Why slow? How do you fix it?"
    * The database still walks past a million rows.
    * Use cursor pagination.
*   **Slow in Europe, fast in the US**: "What do you check, besides a CDN?"
    * Number of sequential round trips.
    * Payload size and serialization time.
    * Where the database and other dependencies run compared to the app servers.

## Common mistakes
*   **Ignoring serialization**: The query is fast, but turning thousands of ORM objects into JSON is slow.
*   **Bad page sizes**: Tiny pages force many requests; no maximum lets one request fetch a whole table. Set a default and a cap.
*   **Joining several collections at once**: Eager-loading two one-to-many relations in one query multiplies rows. Load each with its own `IN` query.

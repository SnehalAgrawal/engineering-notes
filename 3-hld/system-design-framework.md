# System design framework

## Overview
A system design interview checks that you can take a vague problem, lead the discussion, and reach a sensible design with clear tradeoffs. This framework keeps you from getting lost in details before the overall shape is clear.

## Key concepts: the six-step blueprint
1.  **Requirements (~5 min)**:
    *   *Functional*: What must it do? ("send a message", "follow a user").
    *   *Non-functional*: Scale (daily users, peak RPS), uptime target, latency target, consistency, durability.
    *   *Constraints*: Existing stack, team, budget, data retention, regulation (GDPR, HIPAA).
    *   *Scope*: Agree on what's in the first version and what's out.
2.  **Estimation (~5 min)**: Requests per second, storage, bandwidth. Only as exact as needed to make a decision.
3.  **API (~5 min)**: The main endpoints, e.g. `POST /v1/messages`. See [API design](./api-design.md).
4.  **Data model (~5 min)**: Entities, how they're read and written, and which store fits.
5.  **High-level design (~10 min)**: Boxes and arrows. Walk one request from client to database and back.
6.  **Deep dive (~15 min)**: The hard parts: scaling the data layer, caching, hot keys, failures.

*   **HLD vs. LLD**: High-level design is the system view: components, how they talk, tech choices. Low-level design is inside one component: classes, data structures, schemas, error handling. "Design a parking lot" is an LLD question (see [1-lld](../1-lld/)).
*   **Real design doc checklist**:
    *   *Requirements*: functional, non-functional, constraints.
    *   *Architecture*: components, data flow, external APIs.
    *   *Component internals*: classes, schemas, retry and error flows.
    *   *Scaling*: stateless services, load balancing, caching, queues, rate limits.
    *   *Data*: SQL vs. NoSQL vs. time-series, partitioning, indexes, retention.
    *   *Reliability*: redundancy, failover, retries with idempotency, circuit breakers, bulkheads.
    *   *Security*: authentication (JWT, OAuth2, mTLS), encryption in transit and at rest, secrets.
    *   *Delivery*: CI/CD, canary or blue-green rollouts, feature flags, config.
    *   *Monitoring*: logs, metrics, traces, alerts.
    *   *Tradeoffs*: latency vs. cost, consistency vs. availability, simple vs. flexible, open source vs. managed.

## Real-world usage
*   **URL shortener**: Create and resolve codes. Generate codes from a unique ID in base62 (0-9, a-z, A-Z). Key-value store, cache for popular codes, rate limiting, click stats sent through a queue. Reads dominate, so focus on caching.
*   **Chat app (WhatsApp-like)**: Ask about group size, read receipts, media. Gateways hold WebSocket connections (compare with long polling); a registry maps user to gateway; messages stored in order per chat; media in object storage; a presence service for "online / last seen"; push notifications and sync for offline users; end-to-end encryption. Deep dive: routing a message to the right gateway and delivery states (sent, delivered, read).
*   **Ride hailing (Uber-like)**: Drivers send location often. A geospatial index (geohash, quadtree, PostGIS) finds nearby drivers. Plus matching, pricing, payments, and notifications. Deep dive: write volume of locations, and making sure one driver gets one ride.
*   **Video platform (Netflix-like)**: Upload and transcoding pipeline separate from playback. Deep dive: multiple bitrates and CDN delivery.
*   **Ticket booking**: Deep dive: stop double-booking a seat with short holds and conditional writes (or a distributed lock); use a waiting room for on-sale spikes.
*   **1M concurrent chat users** (prompt scale): Spread connections over many gateway servers. Route messages between gateways with pub/sub (Redis or Kafka). Plan for everyone reconnecting after a restart.
*   **Worked example**: [High-throughput ingestion (10M sensors)](./example-high-throughput-ingestion.md).

## Tradeoffs
*   **Breadth vs. depth**: Long estimates eat deep-dive time. Estimate only what changes a decision.
*   **Day 1 vs. target scale**: Design for the asked scale, but say what you'd build first (often a monolith and one replicated DB).
*   **More parts, more failures**: Every added component (Kafka, Redis, Kubernetes) must earn its place.

## When not to use
*   **LLD interviews**: Use classes and interfaces, not load balancers.
*   **Focused questions**: If they only want the storage layer, go straight to schema, indexes, and sharding.

## Interview focus
*   **Lead the session**: State your plan and check: "Requirements, estimates, API, data, architecture, then deep dive. OK?"
*   **Limited storage**:
    * Compress, set TTLs, move old data to cheaper storage.
    * Use compact IDs and codes.
    * A Bloom filter can answer "definitely not present" without reading storage.
    * Cost: more CPU, slower writes, slower access to old data.
*   **A database node dies right now**:
    * Health checks detect it within seconds.
    * Single-primary DB: promote a replica. With async replication, the last few writes can be lost.
    * Leaderless DB (Cassandra): other replicas keep serving if enough of them respond.
    * Quorum reads and writes (W + R > N) keep reads current despite one dead replica.
    * Meanwhile serve cached or read-only data; return `503` with `Retry-After` for writes.
    * Circuit breakers stop callers from piling up.
    * Afterwards, replace the node to restore the replica count.
    * Monitor failover time, error rate, and replication lag.

## Common mistakes
*   **Silence**: Talk through your thinking.
*   **Picking tech first**: Naming Kafka before the requirements need it.
*   **Ignoring non-functional needs**: A design that works but can't reach the asked scale.

---

## Bloom filters
A Bloom filter answers "definitely not in the set" or "maybe in the set" using very little memory.

*   **How it works**: A bit array plus k hash functions. Adding a key sets k bits. A lookup checks those k bits: any 0 means definitely absent; all 1 means maybe present.
*   **False positives**: Possible, and they rise as the filter fills. False negatives never happen.
*   **Deletes**: Not supported by the basic version (counting Bloom filters support them).
*   **Cassandra**: One filter per data file (SSTable), on by default. Reads skip files that can't hold the key.
*   **RocksDB**: Supported per file; you turn it on with a filter policy.
*   **Spark**: Can build a filter from one side of a join to skip rows on the other side (a config option).
*   **ORC / Parquet**: Files can store filters for chosen columns. ClickHouse offers them as opt-in skip indexes.
*   **PostgreSQL**: The `bloom` extension indexes many columns at once for equality filters on any mix of them. For one column, a normal B-tree is better.

```sql
CREATE EXTENSION bloom;
CREATE INDEX orders_bloom ON orders USING bloom (customer_id, status, region, channel);
SELECT * FROM orders WHERE status = 'shipped' AND region = 'EU';
```

# Caching Strategies & Invalidation

### 1. Overview
A cache keeps a copy of data in fast storage, usually memory, so repeat reads skip the database. The hard part is invalidation: making the copy stop serving old data after the source changes.

### 2. Key Concepts
*   **Cache-aside**: The app checks the cache. On a miss, it reads the database and stores the result. The most common pattern.
*   **Read-through**: Same idea, but the cache layer loads from the database itself.
*   **Write-through**: Each write updates the cache and the database together. Fresh reads, slower writes. A failure between the two writes can still leave them out of sync.
*   **Write-back**: Writes go to the cache and reach the database later, in batches. Fast, but unflushed writes are lost if the cache node dies.
*   **TTL**: Time to live. The key expires after this time, which caps how stale it can get.
*   **Eviction**: What to drop when memory is full: least recently used (LRU) or least frequently used (LFU).

### 3. Real-World Usage
*   **Sessions**: Stored in Redis with a TTL equal to the session length.
*   **Product pages**: Cache-aside. Products change rarely; a miss is cheap.
*   **View counts**: Write-back. Increment in Redis, save totals to SQL now and then. Losing a few counts is fine.
*   **CDN**: Images, CSS, and JS cached on servers near users (Cloudflare, Akamai).

### 4. Tradeoffs
*   **Freshness vs. load**: A longer TTL means fewer database reads and older data.
*   **Two copies**: Cache and database can disagree. Races and partial failures make sync hard.
*   **Stampede**: A hot key expires and many requests rebuild it at once, all hitting the database.

### 5. When NOT to Use
*   **Little reuse**: If almost every request is different, the hit rate stays near zero and the cache only adds a hop.
*   **Values you act on**: Read balances, stock, and permissions from the source when making the decision. A cached value is fine for display.

### 6. Interview Focus
*   **Invalidation**: "A user changes their name. How do caches in every region drop the old one?"
    * Update the database, then delete the cache key. Don't set it: two writers setting it can race.
    * Publish an event (e.g., Kafka, or change data capture from the database log). Each region deletes its copy.
    * Keep a TTL on every key as a backstop for lost events.
    * Redis Pub/Sub drops messages for disconnected subscribers. Use Kafka or Redis Streams if that matters.
*   **Stampede**: "A celebrity profile key expires. How do you avoid a database spike?"
    * Single-flight: one request rebuilds the key; others wait for it. Across servers, use a short Redis lock (`SET key val NX PX 5000`).
    * Serve the stale value while one request refreshes it in the background.
    * Early refresh: each read has a small chance to refresh before expiry, rising as expiry nears.
    * TTL jitter (random extra seconds) stops many keys expiring together. It does not help one hot key.
    * Warm known hot keys after a deploy or cache flush.
*   **Is it working?**: "How do you know the cache helps?"
    * Hit rate = hits / (hits + misses). Also track p95/p99 latency, evictions, memory, and how much database load dropped.
    * There is no universal good hit rate. Compare per key type and over time. Falling hits with rising evictions means the cache is too small.
*   **Where to cache**: "Which layers can hold a cache?"
    * Browser and CDN: static files and public pages.
    * API gateway or reverse proxy: whole responses for public GETs.
    * In-process memory: tiny, hot data; each server has its own copy.
    * Shared cache (Redis, Memcached): data shared by all app servers.
    * Database: its own buffer pool keeps hot pages in memory.
    * Cache as close to the user as freshness allows.

### 7. Common Mistakes
*   **No limits**: No TTLs and no `maxmemory`, so memory fills up. Set a memory limit and an eviction policy.
*   **Cold start**: A deploy empties the cache and the full load hits the database. Warm hot keys first or ramp traffic slowly.
*   **Sensitive data**: Personal data or tokens in a shared, unencrypted cache other teams can read.

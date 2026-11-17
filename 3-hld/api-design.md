# API design

## Overview
An API is a contract other teams and clients depend on. Good API design means picking the right protocol, changing the API without breaking clients, and making requests safe to retry.

## Key concepts
*   **REST**: URLs name resources; HTTP methods act on them. Works everywhere, easy to cache. The default for public APIs.
*   **GraphQL**: The client asks for exactly the fields it needs in one request. Less over-fetching, more work on the server.
*   **gRPC**: Binary messages (Protocol Buffers) over HTTP/2, with generated clients and streaming. Good for service-to-service calls.
*   **Versioning**: How you ship breaking changes while old clients keep working (`/v1/`, a header, or a query parameter).
*   **Idempotency**: Sending the same request twice has the same effect as once. GET, PUT, DELETE are idempotent. POST needs an idempotency key.
*   **Rate limiting**: Caps how many requests a client can send per period.

## Real-world usage
*   **Public APIs**: Payment APIs like Stripe use REST with clear versions and docs.
*   **Mobile apps**: GraphQL fetches a post, its comments, and the author in one round-trip.
*   **Internal services**: gRPC for typed, fast calls.
*   **Payments**: Each `POST /payments` sends an `Idempotency-Key` header so a retry doesn't charge twice.

## Tradeoffs
*   **GraphQL**: Flexible for clients, but nested queries can cause many DB calls (the N+1 problem) and are hard to cache at a CDN.
*   **Versioning**: Each live version is code to maintain. Prefer adding fields over new versions.
*   **REST vs. gRPC**: REST is readable and works in any browser. gRPC is faster and typed but needs generated clients and a proxy for browsers.

## When not to use
*   **GraphQL**: A simple CRUD API with one client, or a public API where HTTP caching matters and you can't build query cost limits.
*   **gRPC**: Public browser-facing APIs.

## Interview focus
*   **Video platform API**: pagination, uploads, partial updates.
    * Pagination: use a cursor (`next_cursor`) for feeds that change; offsets only for small, stable lists. Set a max page size.
    * Upload: server returns a pre-signed storage URL, client uploads directly, then a background job transcodes.
    * Partial update: `PATCH` with JSON Merge Patch. `PUT` replaces the whole resource.
*   **User clicks Pay twice**:
    * Client sends one unique key for every attempt of the same payment.
    * Server claims the key atomically (unique constraint or Redis `SET NX`), processes, stores the response with a TTL.
    * Same key again: return the stored response. Same key with a different body: reject.
*   **Removing a field many clients use**:
    * Add the replacement first. Keep the old field working.
    * Mark it deprecated (`Deprecation` and `Sunset` headers) and announce a date.
    * Publish a migration guide. Track who still calls it.
    * Remove only when usage is low, or in the next major version.
*   **Feed at scale**: Prompt: "Twitter-like feed, 1M writes/sec, 10M reads/sec." (Numbers come from the prompt.)
    * Fan-out on write: when someone posts, a worker pushes the post ID into each follower's cached timeline (a Redis sorted set of the latest N IDs).
    * Fan-out on read for accounts with huge follower counts: merge their posts at read time instead.
    * Use both. Posts live in a durable store (e.g., Cassandra, partitioned by user ID); timelines are a cache you can rebuild.
    * Watch queue depth, cache hit rate, and read latency. Rate limit posting.
*   **WebSockets vs. plain requests**:
    * WebSockets (two-way) for chat and live collaboration. SSE (server to client) for notifications and live feeds.
    * Plain requests or batch jobs for exports and periodic sync. Easier to scale and retry.
    * Mix: batch for the initial sync, real-time for changes after that.
*   **Rate limiting a public API**:
    * Token bucket: tokens refill at a steady rate; each request spends one. Allows short bursts.
    * Fixed window: count per minute with Redis `INCR` + `EXPIRE`. Simple, but bursts at window edges.
    * A token bucket in Redis needs a Lua script to update tokens and time together.
    * Limit per API key, with different limits per plan.
    * Shared Redis counts across all instances; per-instance limits are faster but rough.
    * Return `429` with a `Retry-After` header.
*   **GraphQL N+1**:
    * DataLoader batches all lookups of one type in a request into one query.
    * Limit query depth and cost; allowlist queries for public clients.
*   **URL shortener REST API**:
    * `POST /v1/urls` creates a code (accepts an `Idempotency-Key`). `GET /{code}` redirects. `GET /v1/urls/{code}/stats` returns clicks. `DELETE` for the owner.
    * 301 is cached by browsers, so you lose click stats. 302 sends every click through you.
    * Resolve code collisions on the server. Return `409` only when a requested custom alias is taken.
    * Optional expiry date; return `410 Gone` after it.
*   **Versioning options**:
    * URI (`/v1/orders`): clear, easy to route and cache. The common choice for major versions.
    * Header (`Accept: application/vnd.example.v2+json`): clean URLs, but caches must vary on the header.
    * Query parameter (`?version=2`): easy to add, easy to forget.
    * Within a version, only make additive changes (new optional fields).
*   **Versioning between microservices**:
    * Keep changes backward compatible. With Protobuf, add fields with new numbers; never reuse old ones.
    * Contract tests check that a provider change doesn't break its consumers.
*   **Ride-hailing API**:
    * `POST /v1/rides` to request a ride (with an idempotency key). `GET /v1/rides/{id}` for status.
    * Drivers send location updates often, over a WebSocket or small frequent requests.
    * Ride status changes are pushed to the rider (WebSocket or push notification).
*   **Securing APIs**: Authentication, scopes and roles, rate limits, input validation. See [gateways and security](../6-backend-engineering/gateways-security.md).
*   **Error handling**:
    * One error shape everywhere: `{ "error": { "code": "invalid_request", "message": "Email is required" } }`.
    * Correct status codes: 400 bad input, 401 not logged in, 403 not allowed, 404, 409 conflict, 429, 5xx server fault.
    * Include a request ID. Never return stack traces.

## Common mistakes
*   **Breaking `v1` in place**: Removing a field or changing its type with no migration path.
*   **`200 OK` for errors**: Breaks client retries, monitoring, and caches.
*   **Returning too much**: Sending the full user object (password hash, address) when only the username is needed.

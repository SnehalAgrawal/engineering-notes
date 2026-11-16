# Architecture Styles: Monolith, Microservices, & EDA

### 1. Overview
An architecture style is how you split a system into deployable parts and how those parts talk. The right choice depends on team size, how complex the domain is, and how much the team can operate. Conway's Law: systems end up shaped like the teams that build them.

### 2. Key Concepts
*   **Monolith**: One app, one deploy. Easy to build, test, and debug early on.
*   **Modular monolith**: One deploy, but with strict module boundaries inside. Easy to split later.
*   **Microservices**: Small services, each owning one business area, its own data, and its own deploys.
*   **Event-driven architecture (EDA)**: Services publish events to a broker; others react. The producer doesn't know who listens. Microservices often use both direct calls and events.
*   **Event sourcing**: Store every change, not just the current state. A bank account is a list like "deposit 500, withdraw 200, deposit 700". Replay it to get the balance.
    *   Gives a full history and lets you rebuild state.
    *   Harder to change event formats and to query.
*   **CQRS**: Use one model for writes and separate models shaped for reads. Read models are usually updated a moment later.
*   **Serverless (FaaS)**: The cloud runs your functions on triggers (HTTP call, file upload, queue message). No servers to manage.

| | Direct calls (REST, gRPC) | Events via a broker |
|---|---|---|
| Coupling | Caller waits for the callee | Producer doesn't know consumers |
| If the other side is down | Caller fails | Broker holds events; consumers catch up |
| Debugging | Simple call chain | Harder to trace |
| Good for | Queries, user actions needing an answer | Fan-out, notifications, streams |

### 3. Real-World Usage
*   **Early startup**: A monolith (Rails, Django) to move fast while the product changes.
*   **Large companies**: Many microservices so teams ship on their own. Netflix and Uber have written publicly about this. Workflows across services often use the [Saga pattern](../2-design-patterns/cloud-native-patterns.md).
*   **Log processing**: One event stream; search, archive, and alerting each consume it.
*   **Ledgers**: Event sourcing for the audit trail, with CQRS read models for balances.
*   **Image resizing**: A serverless function runs when a photo lands in storage.

### 4. Tradeoffs
*   **Microservices vs. monolith**: Team independence and separate scaling, but you pay in network calls, data consistency work, and monitoring.
*   **Events vs. direct calls**: Easy to add new consumers, harder to trace a chain of events.
*   **Serverless cost**: Cheap for low or spiky traffic, often expensive for steady heavy traffic.

### 5. When NOT to Use
*   **Microservices**: Small team or unclear domain. You'll maintain plumbing instead of features, and the boundaries will be wrong.
*   **EDA**: When the user needs an immediate answer that depends on several services.
*   **Event sourcing**: Plain CRUD data where nobody needs history.

### 6. Interview Focus
*   **Why split the monolith?**
    * Only for real pain: blocked deploys, one module forcing the whole app to scale, different reliability needs.
    * Split where it hurts. Start with a modular monolith if unsure.
*   **Joining data across services**: User and Order data live in different services.
    * API composition: call both and merge. Simple, but slower and fails if either is down.
    * CQRS read model: build a combined view from events. Fast reads, slightly stale.
*   **Broker goes down**:
    * Producers can't publish; consumers stop.
    * Use a replicated broker cluster and producer retries.
    * Use the outbox pattern: save the event in the DB in the same transaction, send it later. No event is lost.
*   **When to pick a monolith**: Small team, changing domain, need to move fast, little ops capacity.
*   **When to pick serverless**:
    * Good: spiky traffic, scale to zero, event triggers, MVPs, small data jobs.
    * Bad: steady heavy load, long jobs, paths that can't take cold-start delay.
*   **Events vs. REST**:
    * REST for actions that need an answer now (CRUD, user commands).
    * Events for reactions and fan-out (notifications, logs, IoT).
    * Most systems use both: REST for the command, events for what happens after.
*   **Event-driven stock trading system**:
    * Orders come in as events, partitioned by stock symbol.
    * One matching engine per symbol processes orders in strict order.
    * Trades go out as events to settlement, market data, and notifications.
    * Event sourcing keeps a full, replayable record of every order.

### 7. Common Mistakes
*   **Distributed monolith**: Services chained by sync calls. One goes down, all go down.
*   **Shared database**: Several services writing the same tables. Nobody can change the schema alone.
*   **No tracing**: Moving to microservices without distributed tracing and central logs.

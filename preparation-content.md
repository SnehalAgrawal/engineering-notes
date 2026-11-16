# Topic Index

Every topic, linked to its note. Tick a box when you have revised it (fork the repository or copy this file to keep your own progress).

## 1. Low-Level Design

### Core OOP

- [ ] [Encapsulation](./1-lld/core-oop.md)
- [ ] [Abstraction](./1-lld/core-oop.md)
- [ ] [Inheritance](./1-lld/core-oop.md)
- [ ] [Polymorphism](./1-lld/core-oop.md)
- [ ] [Composition vs inheritance](./1-lld/core-oop.md)
- [ ] [Interfaces vs abstract classes](./1-lld/core-oop.md)
- [ ] [Dependency injection](./1-lld/dependency-injection.md)

### SOLID Principles

- [ ] [Single responsibility](./1-lld/solid-principles.md)
- [ ] [Open/closed](./1-lld/solid-principles.md)
- [ ] [Liskov substitution](./1-lld/solid-principles.md)
- [ ] [Interface segregation](./1-lld/solid-principles.md)
- [ ] [Dependency inversion](./1-lld/solid-principles.md)

### Clean Code

- [ ] [Naming](./1-lld/clean-code.md)
- [ ] [Function design](./1-lld/clean-code.md)
- [ ] [Code smells (God object, spaghetti code)](./1-lld/clean-code.md)
- [ ] [Refactoring](./1-lld/clean-code.md)

---

## 2. Design Patterns

### Creational

- [ ] [Singleton and the module pattern](./2-design-patterns/creational-patterns.md)
- [ ] [Factory, Factory Method, Abstract Factory](./2-design-patterns/creational-patterns.md)
- [ ] [Builder](./2-design-patterns/creational-patterns.md)
- [ ] [Prototype](./2-design-patterns/creational-patterns.md)

### Structural

- [ ] [Adapter](./2-design-patterns/structural-patterns.md)
- [ ] [Decorator](./2-design-patterns/structural-patterns.md)
- [ ] [Facade](./2-design-patterns/structural-patterns.md)
- [ ] [Proxy](./2-design-patterns/structural-patterns.md)

### Behavioral

- [ ] [Observer](./2-design-patterns/behavioral-patterns.md)
- [ ] [Strategy](./2-design-patterns/behavioral-patterns.md)
- [ ] [Command](./2-design-patterns/behavioral-patterns.md)
- [ ] [Chain of responsibility](./2-design-patterns/behavioral-patterns.md)
- [ ] [State](./2-design-patterns/behavioral-patterns.md)
- [ ] [Mediator](./2-design-patterns/behavioral-patterns.md)

### Cloud-Native Patterns

- [ ] [Saga (orchestration vs choreography)](./2-design-patterns/cloud-native-patterns.md)
- [ ] [Transactional outbox](./2-design-patterns/cloud-native-patterns.md)
- [ ] [Circuit breaker and bulkhead](./2-design-patterns/cloud-native-patterns.md)

### UI Architecture

- [ ] [MVC, MVP, MVVM](./2-design-patterns/ui-architecture-patterns.md)

---

## 3. High-Level Design

### Fundamentals

- [ ] [Scalability and load balancing](./3-hld/hld-fundamentals.md)
- [ ] [CAP and PACELC](./3-hld/hld-fundamentals.md)
- [ ] [Latency vs throughput](./3-hld/hld-fundamentals.md)
- [ ] [Fault tolerance](./3-hld/hld-fundamentals.md)

### Architecture

- [ ] [Monolith vs microservices](./3-hld/architecture-styles.md)
- [ ] [Modular monolith](./3-hld/architecture-styles.md)
- [ ] [Event-driven architecture, event sourcing, CQRS](./3-hld/architecture-styles.md)
- [ ] [Serverless](./3-hld/architecture-styles.md)

### Cost Engineering

- [ ] [Serverless vs containers vs reserved capacity](./3-hld/cost-engineering.md)

### API Design

- [ ] [REST, GraphQL, gRPC](./3-hld/api-design.md)
- [ ] [Versioning and deprecation](./3-hld/api-design.md)
- [ ] [Idempotency](./3-hld/api-design.md)
- [ ] [Rate limiting algorithms](./3-hld/api-design.md)

### Approach and Worked Examples

- [ ] [System design framework (URL shortener, ride-hailing, chat)](./3-hld/system-design-framework.md)
- [ ] [High-throughput ingestion (millions of devices)](./3-hld/example-high-throughput-ingestion.md)

---

## 4. Distributed Systems

### Core

- [ ] [Consensus basics](./4-distributed-systems/distributed-core.md)
- [ ] [Leader election and failover](./4-distributed-systems/distributed-core.md)
- [ ] [Service discovery](./4-distributed-systems/distributed-core.md)
- [ ] [Replication](./5-database-design/database-scaling.md)
- [ ] [Sharding](./5-database-design/database-scaling.md)

### Consistency

- [ ] [Strong vs eventual consistency](./4-distributed-systems/consistency-models.md)
- [ ] [Conflict resolution (CRDTs, vector clocks)](./4-distributed-systems/consistency-models.md)

### Reliability Patterns

- [ ] [Timeouts and fallbacks](./4-distributed-systems/reliability-patterns.md)
- [ ] [Circuit breaker](./4-distributed-systems/reliability-patterns.md)
- [ ] [Retry and backoff](./4-distributed-systems/reliability-patterns.md)
- [ ] [Bulkhead](./4-distributed-systems/reliability-patterns.md)
- [ ] [Rate limiting](./4-distributed-systems/reliability-patterns.md)

### Messaging and Data Flow

- [ ] [Pub/sub](./4-distributed-systems/messaging-systems.md)
- [ ] [Queues: Kafka vs RabbitMQ](./4-distributed-systems/messaging-systems.md)
- [ ] [Delivery guarantees](./4-distributed-systems/messaging-systems.md)
- [ ] [Dead letter queues](./4-distributed-systems/messaging-systems.md)
- [ ] [Stream processing (exactly-once effects, backpressure, watermarks)](./4-distributed-systems/stream-processing.md)
- [ ] [Data pipelines (ETL vs ELT, orchestration, lineage)](./4-distributed-systems/data-pipelines.md)

---

## 5. Database Design

### SQL

- [ ] [Indexing](./5-database-design/sql-deep-dive.md)
- [ ] [Joins](./5-database-design/sql-deep-dive.md)
- [ ] [Normalization](./5-database-design/sql-deep-dive.md)
- [ ] [Transactions and isolation levels](./5-database-design/sql-deep-dive.md)
- [ ] [CTEs and window functions](./5-database-design/sql-deep-dive.md)

### NoSQL

- [ ] [Key-value](./5-database-design/nosql-landscape.md)
- [ ] [Document](./5-database-design/nosql-landscape.md)
- [ ] [Wide-column](./5-database-design/nosql-landscape.md)
- [ ] [Graph](./5-database-design/nosql-landscape.md)

### Scaling

- [ ] [Read replicas](./5-database-design/database-scaling.md)
- [ ] [Partitioning vs sharding](./5-database-design/database-scaling.md)

### Caching

- [ ] [Cache-aside](./5-database-design/caching-strategies.md)
- [ ] [Write-through](./5-database-design/caching-strategies.md)
- [ ] [Write-back](./5-database-design/caching-strategies.md)
- [ ] [Invalidation strategies](./5-database-design/caching-strategies.md)
- [ ] [Cache stampede](./5-database-design/caching-strategies.md)

---

## 6. Backend Engineering

### API Layer

- [ ] [Authentication](./6-backend-engineering/gateways-security.md)
- [ ] [Authorization](./6-backend-engineering/gateways-security.md)
- [ ] [Rate limiting](./4-distributed-systems/reliability-patterns.md)

### Performance

- [ ] [Pagination](./6-backend-engineering/performance-optimization.md)
- [ ] [Lazy loading](./6-backend-engineering/performance-optimization.md)
- [ ] [N+1 problem](./6-backend-engineering/performance-optimization.md)

### Async Systems

- [ ] [Job queues](./6-backend-engineering/async-systems.md)
- [ ] [Event loop](./7-javascript/js-async-event-loop.md)
- [ ] [Backpressure](./6-backend-engineering/async-systems.md)

### Testing

- [ ] [Test pyramid, contract tests, load testing](./6-backend-engineering/testing-strategies.md)

---

## 7. JavaScript

### Fundamentals

- [ ] [Declarations, types, equality, modules](./7-javascript/js-fundamentals.md)
- [ ] [TypeScript](./7-javascript/typescript.md)

### Core

- [ ] [Closures](./7-javascript/js-internals.md)
- [ ] [Hoisting and the temporal dead zone](./7-javascript/js-internals.md)
- [ ] [Prototype chain](./7-javascript/js-internals.md)
- [ ] [`this` binding](./7-javascript/js-internals.md)
- [ ] [V8 engine internals](./7-javascript/js-internals.md)

### Async

- [ ] [Event loop](./7-javascript/js-async-event-loop.md)
- [ ] [Microtasks vs macrotasks](./7-javascript/js-async-event-loop.md)
- [ ] [Promises and async/await](./7-javascript/js-async-event-loop.md)

### Advanced

- [ ] [Memory leaks](./7-javascript/js-advanced-concepts.md)
- [ ] [Debounce and throttle](./7-javascript/js-advanced-concepts.md)
- [ ] [Functional programming](./7-javascript/js-advanced-concepts.md)

### Node.js

- [ ] [Event-driven architecture](./7-javascript/node-js.md)
- [ ] [Streams](./7-javascript/node-js.md)
- [ ] [Cluster vs worker threads](./7-javascript/node-js.md)

### Frontend

- [ ] [Micro-frontends](./7-javascript/advanced-fe-patterns.md)
- [ ] [State management (Redux, Zustand, signals)](./7-javascript/advanced-fe-patterns.md)
- [ ] [Core Web Vitals and bundle size](./7-javascript/advanced-fe-patterns.md)
- [ ] [`SharedArrayBuffer` and `Atomics`](./7-javascript/advanced-fe-patterns.md)

---

## 8. Python

### Core

- [ ] [Data structures](./8-python/python-core.md)
- [ ] [Mutable vs immutable](./8-python/python-core.md)
- [ ] [Flask and FastAPI](./8-python/python-core.md)

### Advanced

- [ ] [Generators](./8-python/python-advanced.md)
- [ ] [Decorators](./8-python/python-advanced.md)
- [ ] [Context managers](./8-python/python-advanced.md)

### Internals and Concurrency

- [ ] [GIL and free-threaded Python](./8-python/python-internals.md)
- [ ] [Memory management](./8-python/python-internals.md)
- [ ] [Asyncio](./8-python/python-concurrency.md)

---

## 9. Concurrency

- [ ] [Threads vs processes](./9-concurrency/concurrency-fundamentals.md)
- [ ] [Async vs multithreading](./9-concurrency/concurrency-fundamentals.md)
- [ ] [Race conditions](./9-concurrency/concurrency-fundamentals.md)
- [ ] [Locks and semaphores](./9-concurrency/concurrency-fundamentals.md)
- [ ] [Producer-consumer](./9-concurrency/concurrency-fundamentals.md)

---

## 10. Cloud and Infrastructure

- [ ] [Docker](./10-cloud-infra/containers-orchestration.md)
- [ ] [Kubernetes basics](./10-cloud-infra/containers-orchestration.md)
- [ ] [Kubernetes networking (Services, Ingress, Gateway API)](./10-cloud-infra/kubernetes-networking.md)
- [ ] [Autoscaling](./10-cloud-infra/scaling-deployment.md)
- [ ] [CI/CD](./10-cloud-infra/scaling-deployment.md)
- [ ] [Rolling, blue-green, and canary deployments](./10-cloud-infra/scaling-deployment.md)
- [ ] [Terraform](./10-cloud-infra/terraform.md)
- [ ] [Helm](./10-cloud-infra/helm.md)
- [ ] [NGINX](./10-cloud-infra/nginx.md)
- [ ] [WebSocket tooling (WebSocket API, `ws`, Socket.IO)](./10-cloud-infra/websocket-tooling.md)
- [ ] [Cloud services (Azure, AWS, GCP)](./10-cloud-infra/cloud-services.md)

---

## 11. Observability

- [ ] [Logging](./11-observability/observability-patterns.md)
- [ ] [Metrics](./11-observability/observability-patterns.md)
- [ ] [Tracing](./11-observability/observability-patterns.md)
- [ ] [SLIs, SLOs, and error budgets](./11-observability/observability-patterns.md)
- [ ] [Debugging production issues](./11-observability/observability-patterns.md)

---

## 12. AI Systems

- [ ] [LLM basics (tokens, prompting)](./12-modern-systems/ai-agentic-systems.md)
- [ ] [RAG](./12-modern-systems/ai-agentic-systems.md)
- [ ] [Vector databases](./12-modern-systems/ai-agentic-systems.md)
- [ ] [Agents, memory, and tool orchestration](./12-modern-systems/ai-agentic-systems.md)

---

## 13. Core Fundamentals

### Data Structures and Algorithms

- [ ] [Hash maps, tries, DAGs, LSM trees](./13-core-fundamentals/dsa-for-seniors.md)
- [ ] [Amortized cost and cache locality](./13-core-fundamentals/dsa-for-seniors.md)
- [ ] [Problem patterns (two pointers, sliding window, DP, greedy, graphs)](./13-core-fundamentals/dsa-for-seniors.md)

### Operating Systems and Networking

- [ ] [I/O models (blocking, non-blocking, epoll)](./13-core-fundamentals/os-networking.md)
- [ ] [Virtual memory and page faults](./13-core-fundamentals/os-networking.md)
- [ ] [TCP, TLS 1.3, HTTP/3](./13-core-fundamentals/os-networking.md)

### Security

- [ ] [OWASP Top 10 (injection, broken access control, SSRF)](./13-core-fundamentals/security-fundamentals.md)
- [ ] [Zero trust](./13-core-fundamentals/security-fundamentals.md)

### Tools

- [ ] [Engineering toolbox](./13-core-fundamentals/engineering-toolbox.md)

---

## 14. Leadership and Behavioral

- [ ] [Leadership principles and the STAR method](./14-leadership-and-behavioral/leadership-principles.md)
- [ ] [Engineering execution and ADRs](./14-leadership-and-behavioral/engineering-execution.md)
- [ ] [Incident management and postmortems](./14-leadership-and-behavioral/incident-management.md)

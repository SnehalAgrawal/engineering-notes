# Cloud-native patterns

## Overview

Cloud-native patterns help services survive in distributed systems (Kubernetes, cloud platforms), where the network is unreliable and single instances fail all the time.

* **Containing failures**: one broken service shouldn't take down the rest.
* **Data consistency**: keep data in step across services that each own a database, without two-phase commit (2PC, a protocol that locks all databases until everyone agrees).

## Key concepts

### Circuit Breaker

Stops calling a dependency that keeps failing. While "open", it fails fast or returns a fallback. After a pause, it tries again.

Bad: if the service is down, this hammers it forever and ties up connections on both sides.

```javascript
async function callService() {
  try {
    return await fetch("http://flaky-service/api");
  } catch (e) {
    return callService(); // retries forever, no delay
  }
}
```

Three states:

* **CLOSED**: calls go through; failures are counted.
* **OPEN**: calls fail fast, no request is sent.
* **HALF_OPEN**: after a cooldown, a trial call decides whether to close again. Real libraries also limit how many trial calls get through.

```javascript
class CircuitBreaker {
  constructor({ threshold, cooldownMs }) {
    this.threshold = threshold;
    this.cooldownMs = cooldownMs;
    this.state = "CLOSED";
    this.failures = 0;
    this.openedAt = 0;
  }

  async call(fn, fallback) {
    if (this.state === "OPEN") {
      if (Date.now() - this.openedAt < this.cooldownMs) return fallback();
      this.state = "HALF_OPEN"; // cooldown over: try again
    }
    try {
      const result = await fn();
      this.state = "CLOSED";
      this.failures = 0;
      return result;
    } catch (e) {
      this.failures++;
      if (this.state === "HALF_OPEN" || this.failures >= this.threshold) {
        this.state = "OPEN";
        this.openedAt = Date.now();
      }
      throw e;
    }
  }
}
```

### Outbox

Makes "save to the database" and "publish an event" succeed or fail together.

Bad: the dual-write problem. If the broker is down, or the process crashes between the two lines, the DB has the change but no other service hears about it.

```javascript
await db.save(order);
await kafka.send("order_updated", order); // may never happen
```

Better: write the event to an outbox table in the same transaction.

```javascript
const trx = await db.transaction();
await trx.save(order);
await trx.saveToOutboxTable({ event: "order_updated", data: order });
await trx.commit(); // both rows commit, or neither does
```

A separate relay process reads unsent outbox rows, publishes them, and marks them sent. It can crash after publishing but before marking, so a message can arrive twice (at-least-once delivery). Consumers must handle duplicates safely (be idempotent).

### Saga

Splits one big cross-service transaction into a series of local transactions, one per service. If a step fails, the saga runs compensating actions that undo the earlier steps (refund the payment, release the stock).

Two ways to coordinate:

* **Orchestration**: one central coordinator tells each service what to do.
* **Choreography**: each service reacts to the others' events.

### Bulkhead

Gives each dependency its own limited pool of threads, connections, or instances. If one dependency is slow, it can only use up its own pool. Example: separate thread pools for checkout calls and recommendation calls.

## Real-world usage

* **Checkout** (Saga): Inventory, then Payment, then Shipping.
* **Reliable events** (Outbox): every "UserCreated" event reaches Kafka.
* **Libraries**: Resilience4j (Java), Polly (.NET), and opossum (Node.js) provide circuit breakers, retries, and bulkheads.
* **Resource isolation** (Bulkhead): a connection pool per downstream service, or separate deployments with CPU and memory limits.

## Tradeoffs

* **Saga**: hard to debug. Data is only eventually consistent, and half-finished states are visible to users.
* **Outbox**: events go out a little after the commit, not at the same moment.
* **Circuit Breaker**: a useful fallback (cached data, a reduced response) takes extra work for each call.

## When not to use

* **Saga**: if the data fits in one database (for example a modular monolith), use a normal transaction.
* **Outbox**: if losing an occasional message is fine (some logs or analytics), skip the extra table and relay.

## Interview focus

* **Two services, two databases, must stay consistent**: Saga.
* **DB commit succeeds but the message send fails**: Outbox.
* **Slow downstream service drags everything down**: timeouts, Circuit Breaker, Bulkhead.

## Common mistakes

* **No compensating actions**: a saga step fails and earlier steps are never undone.
* **Flapping breaker**: thresholds or cooldowns so tight that the breaker opens and closes constantly.
* **Wasteful outbox relay**: polling too often or without an index on unsent rows. Change Data Capture (reading the database log, for example with Debezium) avoids polling but adds infrastructure.

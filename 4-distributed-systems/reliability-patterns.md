# Reliability patterns

## Overview
Network calls fail, and slow calls are worse than failed ones because they hold threads and connections. These patterns let a system fail gently and stop one bad dependency from taking everything down.

## Key concepts
*   **Timeout**: A deadline on every outbound call. Without one, a slow service ties up the caller until it fails too.
*   **Retry with exponential backoff**: Retry with growing waits (100 ms, 200 ms, 400 ms) and a max number of tries. Retry only errors that may pass next time (timeouts, 503), never bad input.
*   **Jitter**: Add randomness to each wait so clients don't all retry at the same moment.
*   **Circuit breaker**: After too many failures, stop calling the service for a while and fail fast. Then let a few test calls through.
*   **Fallback**: What to return while the breaker is open: cached data, a default, or a clear error.
*   **Bulkhead**: Separate pools (threads, connections) per dependency, so one full pool doesn't starve the others.
*   **Rate limiting**: Cap how many requests a client can send per period.

App-level circuit breaker in Node with `opossum`:
```javascript
const breaker = new CircuitBreaker(callService, {
  timeout: 3000,                // slower than 3 s counts as a failure
  errorThresholdPercentage: 50, // open when half the calls fail
  resetTimeout: 10000           // try again after 10 s
});
breaker.fallback(() => cachedResponse());
```
A service mesh (Istio, Linkerd, Envoy) can do timeouts, retries, and breaking without code changes, but with less control over fallbacks.

## Real-world usage
*   **API gateways**: Per-key rate limits so one noisy customer doesn't slow everyone.
*   **Payment providers**: Retry short 503 errors with an idempotency key. If the provider keeps failing, open the breaker and show "try again later".
*   **Mixed workloads**: Separate DB pools for user queries and heavy reports.
*   **Cloud SDKs**: AWS SDKs retry with backoff and jitter by default.

## Tradeoffs
*   **Retries add wait time**: 5 tries with a 1 s timeout can keep a user waiting 5 s for an error.
*   **Retries add load**: Retries at several layers multiply. 3 tries at each of 3 layers can mean 27 calls to the bottom service. Retry in one layer only.
*   **Global vs. local rate limits**: A shared Redis counter is accurate but adds a network hop. Per-instance limits are fast but approximate.
*   **Fail fast vs. degrade**: An error is honest. Stale fallback data is friendlier but must be safe for the case.

## When not to use
*   **Retrying non-idempotent calls**: Never auto-retry `POST /charge` without an idempotency key.
*   **Long backoff while a user waits**: Fail quickly with a clear message. Do long retries in background jobs.

## Interview focus
*   **Thundering herd** (everyone retries at once):
    * The spike knocks the recovering service down again.
    * Fix: backoff with jitter, a retry cap, and circuit breakers.
*   **Circuit breaker states**:
    * Closed: calls go through; failures are counted.
    * Open: calls fail immediately for a cool-down period.
    * Half-open: a few test calls. Success closes it; failure opens it again.
*   **Leaky bucket vs. token bucket**:
    * Leaky bucket: requests leave at a fixed rate. Smooth, no bursts. Good for traffic shaping.
    * Token bucket: tokens refill at a fixed rate; each request spends one. Allows short bursts. Common for APIs.

## Common mistakes
*   **Unlimited retries**: Failures loop forever and eat capacity.
*   **No jitter**: All servers reconnect at the same moment after an outage.
*   **No timeouts**: A slow service holds every connection until the caller runs out.

# Testing strategies

## Overview
With many services, most bugs live at the boundaries: a changed API, a real database acting unlike its mock, a slow dependency. Tests must cover each service, the agreements between services, and behavior under failure, and still run fast.

## Key concepts
*   **Test pyramid**: Many unit tests, fewer integration tests, few end-to-end tests. Service teams often use a "diamond" shape with most weight on integration tests.
*   **Unit test**: Tests logic alone. Fast and cheap.
*   **Integration test**: One service with its real database or broker, often started by Testcontainers. Other services are stubbed.
*   **Contract test (Pact)**: The consumer records the requests it sends and the fields it needs. The provider's CI checks it still meets them. Neither side needs the other running.
*   **TDD**: Write a failing test, make it pass, clean up. Tests drive the design.
*   **BDD**: Describe behavior in Given/When/Then terms that non-developers can read.
*   **Chaos engineering**: Break things on purpose (kill instances, add latency) to check the system copes.
*   **Load testing**: Send realistic traffic, raise it step by step, and watch p95/p99 latency and the error rate to find the breaking point. Tools: k6, JMeter, Locust, Gatling.
*   **Common tools**: Jest, Vitest, or Mocha (JavaScript) and pytest (Python) for unit and integration tests; Playwright or Cypress for end-to-end browser tests; Testcontainers for real dependencies; Pact for contracts.

## Real-world usage
*   **Breaking change caught**: The user service changes a date from an ISO 8601 string to a Unix timestamp. Its CI fails against the billing service's contract before deploy.
*   **Real database**: Testcontainers starts PostgreSQL, so queries and migrations run on the real engine.
*   **Resilience**: Gremlin or AWS Fault Injection Service kills pods or adds latency to check retries and timeouts.

## Tradeoffs
*   **Mocks vs. real systems**: Mocks are fast but only do what you assumed. Real systems catch more but are slower and can be flaky.
*   **End-to-end tests**: Most confidence per test, but slow, flaky, and costly. Keep few; push the rest into contract and integration tests.
*   **Async flows**: Testing queue-to-worker paths needs waiting with timeouts, which slows tests.

## When not to use
*   **Full end-to-end suite on every PR**: Run a small smoke set per PR and the full suite before release.
*   **Testing private methods**: Test through the public interface. Tests on private methods break on every refactor.

## Interview focus
*   **Contract testing**: "Service B, which you call, is being rewritten by another team. How do you avoid breakage?"
    * Write consumer contracts for the calls you make.
    * B's CI verifies the new version against them before it ships.
*   **Flaky tests**: "CI fails now and then with no code change."
    * Track failure rate per test; quarantine repeat offenders.
    * Look for shared state, test order, real clocks, and fixed sleeps.
*   **Async**: "How do you test code that publishes to a queue, and the worker that consumes it?"
    * Unit: assert the right message was published, using a fake publisher.
    * Integration: real broker in a container; publish, then wait (with a timeout) for the worker's effect.

## Common mistakes
*   **Over-mocking**: Database, network, and cache all mocked. Tests pass; production fails.
*   **Only the happy path**: No tests for 404s, 500s, timeouts, retries, or bad input.
*   **Slow suites**: If local tests take too long, people stop running them.

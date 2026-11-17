# Engineering toolbox

## Overview
The tools a backend or full-stack engineer is expected to know. Interviews rarely test a tool by name, but "how would you debug, test, deploy, or monitor this?" answers go faster when you know what each tool is for. Learn one tool per row well; recognize the others.

## Key concepts
*   **Version control**: Git. Know branching, rebase vs merge, cherry-pick, `git bisect` to find the commit that broke something, and hooks.
*   **API tools**: Postman, Insomnia, or `curl` to call and mock APIs. OpenAPI (Swagger) to describe an API and generate clients and docs.
*   **Containers and infrastructure**: Docker to package and run services; Kubernetes to deploy and scale them; Terraform, OpenTofu, or Pulumi to define cloud resources as code.
*   **CI/CD**: GitHub Actions, GitLab CI, or Jenkins to run tests and deploy on every change.
*   **Cloud**: AWS, Azure, or GCP. Know identity and access (IAM), compute, managed databases, object storage, queues, and serverless functions on at least one.
*   **Testing**: Jest, Vitest, or Mocha (JavaScript); pytest (Python); Playwright or Cypress (browser end-to-end); k6 or JMeter (load).
*   **Code quality**: ESLint and Prettier (JavaScript), Ruff (Python), SonarQube for wider static analysis.
*   **Data stores**: PostgreSQL or MySQL, MongoDB, Redis, Kafka or RabbitMQ, Elasticsearch or OpenSearch for search and logs.
*   **Observability**: Prometheus and Grafana for metrics; OpenTelemetry for traces; ELK or Loki for logs; Sentry, Datadog, or New Relic for errors and performance.
*   **Diagrams**: Excalidraw or draw.io for whiteboard-style designs; Mermaid or PlantUML for diagrams kept as text next to the code.
*   **Package managers**: npm, pnpm, or yarn (JavaScript); pip, Poetry, or uv (Python). Commit the lock file.
*   **AI tooling**: LLM APIs (OpenAI, Anthropic, Google), frameworks such as LangChain, LlamaIndex, and LangGraph, and vector databases such as pgvector, Weaviate, or Chroma.
*   **Team tools**: An issue tracker (Jira, Linear) and a place for written decisions (Confluence, Notion, or Architecture Decision Records in the repo).

## Real-world usage
*   **A bug that appeared "sometime last week"**: `git bisect` with a test script finds the commit in a few steps.
*   **A new service**: A Dockerfile, a CI pipeline that runs lint and tests, a Terraform module for its database, and a Grafana dashboard before launch.

## Tradeoffs
*   **Managed vs self-hosted**: Managed services (RDS, hosted Kafka) cost more per unit but save on-call time.
*   **One tool per job vs many**: Each extra tool is one more thing to upgrade, secure, and teach.

## When not to use
*   **Kubernetes for one small app**: A managed container service or a single VM is simpler.
*   **A new tool because it is popular**: Adopt it when it solves a problem you have.

## Interview focus
*   **"How do you take a service to production?"**:
    * Container image built in CI, tests and lint on every change.
    * Infrastructure as code, reviewed like application code.
    * Metrics, logs, traces, and alerts in place before launch.
*   **"How do you find what broke?"**:
    * Start from the alert or error tracker, check recent deploys, use traces to find the slow or failing service, then logs for detail.

## Common mistakes
*   **Knowing tools, not concepts**: Naming Kafka without explaining partitions and consumer groups.
*   **No lock files**: Builds change without any code change.
*   **Tools without habits**: The tools help only with the habits behind them: keep services loosely coupled, fail fast with clear errors, measure what matters, automate repeated manual work, and write docs as if you will forget the details tomorrow.

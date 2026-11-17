# Cloud services

## Overview
The big clouds offer the same building blocks under different names: identity, compute, storage, databases, messaging, and monitoring. Learn the categories once and map the names. This note has short summaries of the main Azure services and a table of equivalents.

## Key concepts
*   **IAM (Identity and Access Management)**: Who can do what on which resource. Give people and workloads the least access they need.
*   **Managed identity**: An identity the cloud gives a VM, function, or Pod, so code gets tokens without stored keys. Azure calls it managed identity; AWS uses IAM roles; GCP uses service accounts.
*   **Serverless**: You deploy code or a container. The platform runs, scales (often to zero), and bills per use.
*   **Object storage**: Files ("objects") in buckets or containers, reached over HTTP. Cheap and very durable. Not a file system.

## Azure services
*   **App Service**: Managed hosting for web apps and APIs. Autoscaling, deployment slots, and built-in CI/CD.
*   **Azure Functions**: Event-driven serverless functions, triggered by HTTP, queues, timers, and more. Good for small, stateless tasks.
*   **Azure Container Apps**: Serverless containers with scale-to-zero and KEDA-based autoscaling.
*   **AKS (Azure Kubernetes Service)**: Managed Kubernetes for full control over containers.
*   **Azure Storage**: Blob storage (objects), Queue storage (simple queues), Table storage (key-value), and Azure Files (SMB/NFS shares).
*   **Azure Cosmos DB**: Globally distributed NoSQL database with multi-region writes. It offers several APIs, including NoSQL, MongoDB, Cassandra, Gremlin, and Table.
*   **Azure SQL Database**: Managed SQL Server engine with backups, scaling, and high availability.
*   **Event Hubs**: High-volume event ingestion for telemetry, logs, and streams. It also accepts Kafka clients.
*   **Service Bus**: Enterprise message broker with queues, topics, sessions, and dead-lettering. Use it for commands and business messages between services.
*   **Event Grid**: Routes events (such as "blob created") to handlers by push.
*   **Azure Monitor**: The monitoring platform. Application Insights is its APM part (requests, failures, dependencies, traces). Log Analytics workspaces store logs and are queried with KQL (Kusto Query Language).
*   **Microsoft Entra ID**: The identity provider (formerly Azure Active Directory). Supports OAuth 2.0 and OpenID Connect sign-in for users and apps.
*   **Azure RBAC**: Role assignments on a scope (management group, subscription, resource group, or resource).
*   **Azure Pipelines**: CI/CD in Azure DevOps.

## Service equivalents

| Category | Azure | AWS | GCP |
|---|---|---|---|
| Virtual machines | Virtual Machines | EC2 | Compute Engine |
| Web app hosting | App Service | Elastic Beanstalk | App Engine |
| Serverless functions | Functions | Lambda | Cloud Run functions |
| Serverless containers | Container Apps | ECS on Fargate | Cloud Run |
| Kubernetes | AKS | EKS | GKE |
| Object storage | Blob Storage | S3 | Cloud Storage |
| Relational DB | Azure SQL, Database for PostgreSQL/MySQL | RDS, Aurora | Cloud SQL, AlloyDB |
| NoSQL | Cosmos DB | DynamoDB | Firestore, Bigtable |
| Queue / broker | Service Bus, Queue Storage | SQS, SNS | Pub/Sub |
| Event streaming | Event Hubs | Kinesis, MSK | Pub/Sub, Managed Service for Apache Kafka |
| Event routing | Event Grid | EventBridge | Eventarc |
| Monitoring | Azure Monitor, Application Insights | CloudWatch, X-Ray | Cloud Monitoring, Cloud Logging, Cloud Trace |
| Identity | Entra ID, Azure RBAC | IAM, IAM Identity Center | Cloud IAM, Cloud Identity |
| CI/CD | Azure Pipelines | CodePipeline, CodeBuild | Cloud Build, Cloud Deploy |

## Interview focus
*   **Functions vs. App Service**: "Which do you pick for a given workload?"
    *   Functions: spiky or event-driven work, short tasks, pay per run. Watch cold starts and time limits.
    *   App Service: steady web traffic, long-running requests, WebSockets, predictable cost.
*   **Scalable and cheap on Azure**: "How would you design it?"
    *   Stateless app tier on App Service or Container Apps with autoscaling.
    *   Managed database, a cache (Azure Cache for Redis), and a CDN for static files.
    *   Service Bus for background work, so spikes queue up instead of failing.
    *   Scale to zero where possible, and set budgets and alerts.
*   **Logging and alerting on Azure**: "How do you set it up?"
    *   Send app telemetry to Application Insights (directly or through OpenTelemetry).
    *   Send platform logs to a Log Analytics workspace and query them with KQL.
    *   Create Azure Monitor alerts on error rate and latency, routed through action groups.

## Common mistakes
*   **Long-lived keys in code or config**: Use managed identities and a secret store (Azure Key Vault, AWS Secrets Manager, GCP Secret Manager).
*   **Broad roles**: Owner or Contributor on a whole subscription for an app that needs one storage account.
*   **Queue vs. stream confusion**: Service Bus or SQS for work items processed once. Event Hubs, Kinesis, or Kafka for high-volume streams that several consumers read and can replay.

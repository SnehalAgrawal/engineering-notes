# Cost engineering

## Overview
Cost is a design metric, like latency. You need to know how the cloud charges for compute, storage, and data transfer to build something that stays affordable as it grows.

## Key concepts
*   **On-demand**: No commitment, highest price. For unpredictable load.
*   **Reserved / savings plans**: Commit to 1 or 3 years for a big discount. For the steady base load.
*   **Spot / preemptible**: Spare capacity, very cheap, but the provider can take it back with short notice (2 minutes on AWS). Only for work that can be interrupted.
*   **Serverless pricing**: Pay per request and run time. Nothing when idle.
*   **Storage tiers**: Colder tiers cost less to store but charge to read, and archive tiers can take hours to return data.
*   **Egress**: Data leaving the cloud or crossing regions is billed per GB. Easy to overlook.

## Real-world usage
*   **Batch workers on spot**: CI runners or indexers, with on-demand as backup.
*   **Reserved database**: The main DB runs 24/7, so commit to it.
*   **Dev environments**: Shut them down at night and on weekends.
*   **Same-region reads**: Keep app servers and their read replicas in one region to avoid transfer fees.
*   **Lifecycle rules**: Move old logs to cheaper tiers automatically.

## Tradeoffs
*   **Serverless vs. containers**: Serverless is free when idle but costs more per request once traffic is steady and high. Work out the crossover with your own numbers.
*   **Storage tiers**: Archive is cheap to keep, slow and costly to read. Auto-tiering (S3 Intelligent-Tiering) picks tiers for you for a small monitoring fee.
*   **Spot**: Cheap, but your code must handle being stopped at any time.

## When not to use
*   **Commitments**: Not before you know your normal usage.
*   **Serverless for long jobs**: Video transcoding can exceed the time limit (15 minutes on AWS Lambda) and costs more.

## Interview focus
*   **Build vs. buy auth**:
    * Build cost: engineer time, infra, security fixes, on-call, and features you didn't build.
    * Buy cost: subscription, integration, less control, lock-in.
    * Usually buy what doesn't set you apart; build what does.
*   **Lambda to Kubernetes**:
    * Move when steady traffic makes always-on containers cheaper, and the team can run a cluster.
    * Compare: requests x duration x price vs. nodes needed at good utilization, plus people cost.
*   **Active-active multi-region**:
    * Roughly double the base infra, plus replication traffic and higher egress.
    * Only worth it for strict disaster recovery or users on several continents.

## Common mistakes
*   **Idle resources**: Big staging DBs or GPU nodes running all weekend.
*   **No lifecycle rules**: Years of unread data in the most expensive tier.
*   **Ignoring egress**: Chatty cross-region traffic that costs more than the servers.
*   **Lock-in with no exit plan**: A proprietary service with no alternative if prices rise.

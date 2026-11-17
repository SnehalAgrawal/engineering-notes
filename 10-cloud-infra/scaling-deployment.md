# Deployment and scaling

## Overview
Deployment strategies let you ship new versions without downtime and limit the damage of a bad release. Autoscaling adds and removes capacity as traffic changes.

## Key concepts
*   **Rolling update**: Replace old instances with new ones a few at a time. The Kubernetes default.
*   **Recreate**: Stop all old Pods, then start new ones. Causes downtime, but two versions never run together.
*   **Blue-green**: Two full environments. Blue serves traffic, Green gets the new version, then traffic switches. Switching back is the rollback.
*   **Canary**: Send a small share of traffic to the new version, compare its errors and latency with the old one, then increase the share. Argo Rollouts and Flagger automate this.
*   **HPA (Horizontal Pod Autoscaler)**: Changes the number of Pods to keep a metric near a target. The metric can be CPU or memory use as a percentage of the Pod's requests, or a custom metric.
*   **VPA (Vertical Pod Autoscaler)**: Suggests or sets CPU and memory requests.
*   **Cluster Autoscaler / Karpenter**: Adds a node when Pods are stuck Pending because no node has room for their requests. Removes empty or underused nodes. It looks at requests, not real usage.
*   **CI/CD**: CI builds and tests every change. Continuous delivery keeps every build ready to release. Continuous deployment also releases it automatically.
*   **Pipeline stages**: Build, unit tests, integration tests, end-to-end tests, security scan, deploy. Common tools: GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, Google Cloud Build.
*   **Pipeline secrets**: Keep deploy credentials in the CI tool's secret store or a vault, or use short-lived OIDC tokens instead of long-lived cloud keys.

## Real-world usage
*   **Risky release**: A canary shows the new version to a small slice of traffic first.
*   **Fast rollback**: With blue-green the old version is still running, so rollback is a traffic switch.
*   **Sales peak**: Scale up on a schedule before a known peak, and let metric-based scaling handle the rest.
*   **Quality gates**: Tests, a security scan, and review must pass before deploy.

## Tradeoffs
*   **Blue-green vs. canary**: Blue-green is the fastest to switch and roll back but needs double capacity during the cutover. Canary needs less capacity but takes longer and needs good metrics.
*   **Scaling lag**: New Pods take seconds to start, new nodes take minutes. For sudden spikes, scale ahead on a schedule or keep spare capacity.
*   **Manual approval**: Automatic deploys give faster feedback. Regulated industries often require a recorded human approval.

## When not to use
*   **Blue-green for the database**: Keeping two large databases in sync is risky. Keep one database and make schema changes backward compatible: add the new column, migrate, then remove the old one later (expand/contract).
*   **Autoscaling a tiny, steady app**: Two servers with flat traffic do not need it.

## Interview focus
*   **Rolling updates**: "How does a Kubernetes rolling update work, and what happens to open connections?"
    *   A new Pod template creates a new ReplicaSet. Kubernetes scales it up and the old one down in steps.
    *   `maxSurge` is how many extra Pods are allowed. `maxUnavailable` is how many may be missing. Both default to 25%.
    *   A new Pod counts only after its readiness probe passes.
    *   When an old Pod is deleted, two things start at the same time: it is removed from the Service endpoints, and it gets `SIGTERM`. Load balancers take a moment to notice, so requests can still arrive after the app stops.
    *   Fix: a short `preStop` sleep, then on `SIGTERM` stop taking new requests, finish the current ones, and exit.
    *   Everything must finish within `terminationGracePeriodSeconds` (default 30s). After that the container gets `SIGKILL`.
    *   `SIGTERM` goes to PID 1. A shell-form `CMD node server.js` runs under `sh`, which may not pass the signal on. Use `CMD ["node", "server.js"]` or an init like `tini`.
    *   WebSockets and other long connections are cut, so clients must reconnect.
*   **Rollbacks**: "How do you roll back automatically?"
    *   Kubernetes does not roll back by itself. If new Pods never become ready, the rollout just stalls.
    *   After `progressDeadlineSeconds` (default 600s) the Deployment reports `ProgressDeadlineExceeded`.
    *   Set `maxUnavailable: 0` for critical services, so no old Pod is removed before a new one is ready.
    *   The pipeline runs `kubectl rollout status --timeout=...` and calls `kubectl rollout undo` if it fails. Helm 3 has `helm upgrade --atomic`.
    *   Slow failures, like a memory leak, pass readiness. Catch them with SLO alerts or canary analysis.
    *   With GitOps, also revert the commit, or the tool reapplies the bad version.
*   **Scaling signals**: "What can you scale on besides CPU?"
    *   Queue depth or consumer lag, for workers. This is the most direct measure of backlog.
    *   Requests per second per Pod, for services with steady cost per request.
    *   Active connections, for WebSocket or connection-limited servers.
    *   Business events such as orders per minute, when load follows the business.
    *   Latency, with care. If a shared database is the slow part, more Pods make it worse.
    *   KEDA reads queues and other event sources, feeds the HPA, and can scale to zero.

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>   # CHANGE-CAUSE comes from the kubernetes.io/change-cause annotation
kubectl rollout undo deployment/<name> -n <namespace>      # add --to-revision=<n> for a specific revision
```

## Common mistakes
*   **No readiness probe**: Old Pods are removed before new ones can serve.
*   **Incompatible versions**: During a rollout old and new code run together. New code must not need a column that does not exist yet, and old code must not lose one it reads.
*   **`replicas` fighting the HPA**: A manifest with a fixed `replicas` resets the HPA on every deploy. Leave `replicas` out when an HPA manages the Deployment.
*   **No upper limit**: Set `maxReplicas`, node pool limits, and budget alerts so a spike or retry storm does not create a huge bill.

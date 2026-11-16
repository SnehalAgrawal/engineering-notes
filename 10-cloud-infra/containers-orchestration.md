# Cloud Architecture: Containers & Orchestration

### 1. Overview
A container packages an app with its libraries so it runs the same way on any host. An orchestrator such as Kubernetes runs many containers across many machines, restarts them when they fail, and connects them to each other.

### 2. Key Concepts
*   **Image and container**: An image is the packaged app plus a filesystem. A container is a running image: an isolated process that shares the host's kernel.
*   **CPU architecture**: An image built for amd64 does not run natively on arm64 (and the reverse). Multi-arch images cover both.
*   **Layers and cache**: Each Dockerfile instruction makes a layer. Docker reuses a cached layer if nothing before it changed, so copy dependency files and install them before copying the source code.
*   **Multi-stage build**: Build in one stage with the full toolchain, then copy only the output into a small runtime image.
*   **Volume**: Storage that outlives the container. Use it for databases and uploads.
*   **Container network**: Containers on the same user-defined Docker network reach each other by container name.
*   **Docker Compose**: One YAML file that starts several containers together (app, database, cache) for local development.
*   **Pod**: The smallest unit Kubernetes runs. One or more containers that share a network address and volumes.
*   **Service**: A stable name and IP in front of a group of Pods. See [kubernetes-networking.md](kubernetes-networking.md).
*   **Deployment**: Says what you want, for example "3 Pods of image v2". It manages ReplicaSets to get there.
*   **Other workloads**: StatefulSet (stable names and storage, for databases), DaemonSet (one Pod per node, for log agents), Job and CronJob (run to completion, once or on a schedule).
*   **ConfigMap and Secret**: Settings and credentials given to Pods as environment variables or files.
*   **Namespace**: A named group of objects inside a cluster, used to separate teams or environments.
*   **Control plane**: The API server, etcd (the database), the scheduler, and the controllers. It stores state and makes decisions.
*   **Worker node**: Runs the kubelet (node agent), a container runtime (containerd or CRI-O), and usually kube-proxy.

### 3. Real-World Usage
*   **Microservices**: Each service is an image. Kubernetes places it on a node with enough CPU and memory.
*   **Environment parity**: The same image tag runs in CI, staging, and production. On an Apple silicon Mac, images build as arm64 by default, so build multi-arch images for amd64 servers.
*   **Self-healing**: The kubelet restarts a crashed container. If a node dies, controllers create new Pods on other nodes. Bare Pods with no controller are not recreated.
*   **Secrets**: Kubernetes Secrets pass passwords to containers at runtime. They are only base64-encoded by default, so turn on encryption at rest, limit access with RBAC, or use an external secret manager.

### 4. Tradeoffs
*   **Containers vs. VMs**: Containers start faster and use less memory. They share the host kernel, so isolation is weaker. gVisor or Kata Containers add isolation for untrusted code.
*   **Complexity**: Kubernetes has a real learning and running cost: upgrades, networking, RBAC, monitoring. Managed clusters (EKS, GKE, AKS) remove only the control plane work.
*   **Small images**: Alpine or distroless images are smaller and safer but have no shell. Use `kubectl debug` to attach a temporary debug container.

### 5. When NOT to Use
*   **Static sites**: Use a static host or a single VM.
*   **A few services, no platform team**: Cloud Run, ECS/Fargate, or Azure Container Apps give most of the benefit without running Kubernetes.
*   **Desktop or hardware-bound apps**: GUI apps and special drivers fit poorly. GPUs are the main supported exception.

### 6. Interview Focus
*   **Lifecycle**: "What happens when I run `kubectl apply`?"
    *   The API server checks who you are, runs admission checks, validates the object, and saves it in etcd.
    *   The Deployment controller creates or updates a ReplicaSet.
    *   The ReplicaSet controller sees too few Pods and creates Pod objects.
    *   The scheduler picks a node for each Pod based on requests, affinity, and taints.
    *   The kubelet on that node pulls the image, starts the containers, and reports status.
    *   Controllers keep comparing desired and actual state. If a Pod disappears, a new one is created.
*   **Networking**: "How do Pods find each other if Pod IPs change?"
    *   Callers use a Service name such as `backend.default.svc.cluster.local`.
    *   CoreDNS resolves the name to the Service's stable ClusterIP.
    *   kube-proxy rules on each node forward the connection to one ready Pod behind the Service.
*   **Security**: "Why not run containers as root?"
    *   Root in the container is root on the host unless user namespaces are used.
    *   With a kernel bug, privileged mode, or a host mount, an attacker can take over the node and every Pod on it.
    *   Fix 1: run as a non-root user with a numeric UID (`USER 10001` in the Dockerfile).
    *   Fix 2: in the Pod `securityContext`, set `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, drop all capabilities, and use `readOnlyRootFilesystem: true`.
    *   Fix 3: enforce it with Pod Security Admission (`restricted` profile) or Kyverno/OPA Gatekeeper.
    *   Fix 4: user namespaces (`hostUsers: false` in the Pod spec, on recent Kubernetes versions) map container root to an unprivileged host user.
    *   Fix 5: scan images for known CVEs in CI (Trivy, Grype).

### 7. Common Mistakes
*   **Using `latest`**: Different nodes can pull different images for the same tag. Deploy a version, git SHA, or digest.
*   **Wrong probes**: A readiness probe controls traffic. A liveness probe restarts the container. A liveness probe that checks the database restarts every Pod during a database outage. Use a startup probe for slow starters.
*   **No resource requests**: The scheduler cannot place Pods well, and the HPA cannot compute CPU utilization.
*   **State in the container**: The container filesystem is lost on replacement. Use volumes or external storage.

### 8. Example Dockerfile
A multi-stage Node.js build that runs as a non-root user:

```dockerfile
FROM node:22 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci                       # cached until package files change
COPY . .
RUN npm run build

FROM node:22-slim
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/package*.json ./
RUN npm ci --omit=dev
COPY --from=build /app/dist ./dist
USER 1000                        # the "node" user of the official image, as a numeric UID
CMD ["node", "dist/server.js"]   # exec form, so the app receives SIGTERM
```

### 9. Command Reference

```bash
kubectl config get-contexts
kubectl config use-context <context>
kubectl get ns
kubectl create ns <namespace>
kubectl create deployment <name> -n <namespace> --image=<registry>/<image>:<tag>   # quick test; keep real manifests in Git
kubectl get pods -n <namespace> -o wide
kubectl describe pod <pod> -n <namespace>     # events: scheduling, image pulls, probe failures
kubectl logs <pod> -n <namespace> --previous  # logs from the last crashed container
kubectl get svc -n <namespace>                # EXTERNAL-IP of LoadBalancer Services
```

Rollout and rollback commands are in [scaling-deployment.md](scaling-deployment.md).

GKE access:

```bash
# Write credentials to ~/.kube/config (needs the gke-gcloud-auth-plugin component)
gcloud container clusters get-credentials <cluster> --project <project> --location <region-or-zone>

# Let Docker push to Artifact Registry
gcloud auth configure-docker <region>-docker.pkg.dev
# Image path: <region>-docker.pkg.dev/<project>/<repository>/<image>:<tag>
```

*   **Access**: On GKE, grant access with IAM roles. Binding your user to `cluster-admin` gives full control of the cluster, so avoid it unless needed.

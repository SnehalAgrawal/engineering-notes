# Cloud Architecture: Kubernetes Networking (Services, Ingress, Gateway API)

### 1. Overview
A Service gives a group of Pods one stable name inside the cluster. Ingress or the Gateway API sends outside HTTP(S) traffic to Services based on host name and path. Cluster basics are in [containers-orchestration.md](containers-orchestration.md).

### 2. Key Concepts
*   **ClusterIP Service**: The default. A virtual IP reachable only inside the cluster.
*   **NodePort Service**: Opens the same port on every node.
*   **LoadBalancer Service**: Asks the cloud for an external load balancer.
*   **Headless Service**: `clusterIP: None`. DNS returns the Pod IPs directly. Used by StatefulSets.
*   **Endpoints**: A Service picks Pods by label. Only Pods that pass readiness are listed in its EndpointSlices and get traffic.
*   **Ingress**: Rules like "`app.example.com/api` goes to the `backend` Service". An Ingress does nothing alone. An Ingress controller (a proxy in the cluster or a cloud load balancer) reads it and does the routing. `ingressClassName` picks the controller.
*   **Gateway API**: The successor to Ingress. The platform team owns the `Gateway` (ports, TLS). App teams own `HTTPRoute` objects (paths, backends). Header matching and traffic splitting are built in, not controller-specific annotations.
*   **Ingress status**: The Ingress API is not deprecated, but it is frozen. New features go into Gateway API.

### 3. Ingress NGINX Retirement
*   **Announcement**: In November 2025 Kubernetes SIG Network announced the retirement of the community Ingress NGINX controller (`kubernetes/ingress-nginx`).
*   **Status**: Maintenance ended in March 2026. There are no more releases or security fixes.
*   **Existing clusters**: They keep working but are unpatched. Move to a maintained controller or a Gateway API implementation. None is a drop-in replacement, so migrate and test annotations.
*   **Not affected**: F5's NGINX Ingress Controller is a different project with a similar name.

### 4. TLS from a Certificate File
cert-manager usually issues and renews certificates automatically. For a certificate bought from a CA:

```bash
cat server.crt intermediate-bundle.crt > fullchain.pem   # server certificate first
kubectl create secret tls app-tls -n web --cert=fullchain.pem --key=server.key
```

Ingress using the Secret:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app
  namespace: web
spec:
  ingressClassName: <your-controller-class>
  tls:
    - hosts: [app.example.com]
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix      # matches /api and /api/x, not /apix
            backend:
              service:
                name: backend
                port:
                  number: 80
          - path: /
            pathType: Prefix      # everything else goes to the frontend
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

The same with Gateway API:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web
  namespace: web
spec:
  gatewayClassName: <your-gateway-class>
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: app.example.com
      tls:
        mode: Terminate
        certificateRefs:
          - name: app-tls
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: app
  namespace: web
spec:
  parentRefs:
    - name: web
  rules:
    - matches:
        - path: { type: PathPrefix, value: /api }
      backendRefs:
        - name: backend
          port: 80
    - backendRefs:            # no match means PathPrefix /
        - name: frontend
          port: 80
```

```bash
kubectl get ingress -n web               # ADDRESS shows the load balancer IP
kubectl describe ingress app -n web      # rules, TLS secret, and controller events
kubectl get gateway,httproute -n web     # check the Accepted and Programmed conditions
```

### 5. Interview Focus
*   **Ingress vs. LoadBalancer Service**: Each LoadBalancer Service gets its own cloud load balancer and only routes by port. Ingress or a Gateway puts many hosts and paths behind one entry point and terminates TLS in one place.
*   **Ingress vs. Gateway API**: Ingress puts advanced features in controller-specific annotations. Gateway API has them in the spec and separates platform and app roles.
*   **Service gets no traffic**: Check the selector matches the Pod labels, the Pods are Ready, and `targetPort` matches the container port. `kubectl get endpointslices -l kubernetes.io/service-name=<svc>` shows the endpoints.

### 6. Common Mistakes
*   **Wrong certificate order**: The server certificate must come first, then the intermediates. A missing intermediate may work in browsers but fail in `curl` and API clients.
*   **Installing from a `main` branch URL**: Pin a release version so the install does not change under you.
*   **No `ingressClassName`**: No controller, or the wrong one, picks up the Ingress.
*   **Annotation lock-in**: Many controller-specific annotations make switching controllers expensive.

# Backend Security: AuthN, AuthZ, & API Gateways

### 1. Overview
Backend security has layers. Authentication: who is calling? Authorization: may they do this to this resource? An API gateway applies shared checks at the edge, but each service still checks what it relies on.

### 2. Key Concepts
*   **Authentication (AuthN)**: Proving identity: passwords, SSO, MFA, client certificates.
*   **Authorization (AuthZ)**: Deciding what an identity may do.
    *   *RBAC*: Permissions by role, e.g., "admin", "editor".
    *   *ABAC*: Rules on attributes, e.g., "owner of the document and on a company laptop".
*   **JWT**: A signed token with claims (user, expiry, audience). A service can check it without a database call, so it is often used to pass user identity between services.
*   **API gateway**: One entry point for routing, auth checks, rate limits, and TLS termination.

### 3. Real-World Usage
*   **SSO with OIDC**: Users sign in through Google or another provider; the app stores no passwords.
*   **Ownership check**: Allow editing a post only if the token's user ID equals the post's `author_id`.
*   **Edge checks**: A gateway (Kong, AWS API Gateway, NGINX) rejects unauthenticated or over-limit calls. See [Reliability Patterns](../4-distributed-systems/reliability-patterns.md) for rate limiting.
*   **Service to service**: mTLS (both sides show certificates) proves each service's identity. Separate policies decide which service may call which.

### 4. Tradeoffs
*   **JWT vs. sessions**: JWTs need no session store, but a token stays valid until it expires unless you add server-side state (a denylist). Common fix: short-lived access tokens plus revocable refresh tokens.
*   **Gateway**: One place for shared checks, but every request depends on it. Run several instances; each plugin adds latency.
*   **RBAC vs. ABAC**: RBAC is simple but needs many roles for fine rules. ABAC is flexible but harder to audit.

### 5. When NOT to Use
*   **JWT for one web app**: With one backend, use a server session in a `Secure`, `HttpOnly`, `SameSite` cookie. Easier to revoke. Tokens in `localStorage` can be stolen by XSS.
*   **Business logic in the gateway**: Keep it to routing, auth, and rate limits.

### 6. Interview Focus
*   **Stolen laptop**: "How do you kill a user's tokens now?"
    * Revoke their refresh tokens.
    * Bump a per-user token version that every request checks, or denylist token IDs until they expire.
    * Keep access tokens short-lived.
*   **CSRF**: "What is it and how do you stop it?"
    * Another site makes the browser send a request with the user's cookies.
    * Defenses: `SameSite` cookies, CSRF tokens, checking the `Origin` header, no state changes on `GET`.
*   **OAuth flows**: "Browser SPA vs. backend-to-backend?"
    * SPA: Authorization Code with PKCE.
    * Backend service: Client Credentials.

### 7. Common Mistakes
*   **Secrets in the frontend**: Anyone can read API keys or signing secrets shipped to the browser.
*   **IDOR**: The user is logged in but ownership isn't checked, so changing `/orders/999` to another ID shows someone else's order.
*   **Unverified JWTs**: Always check the signature, pin the expected algorithm, and check `exp`, `iss`, and `aud`.

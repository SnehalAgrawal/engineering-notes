# Security Fundamentals: OWASP & Zero Trust

### 1. Overview
Build security in from the start. Assume any part can be compromised, check every request, and pick defaults that stay safe when someone makes a mistake.

### 2. Key Concepts
*   **OWASP Top 10**: A widely used list of common web risk categories. The 2025 edition, in order: broken access control, security misconfiguration, software supply chain failures, cryptographic failures, injection, insecure design, authentication failures, software or data integrity failures, security logging and alerting failures, and mishandling of exceptional conditions. Three to know well:
    *   *Broken access control*: Users can see or change data that isn't theirs. Since 2025 this also covers SSRF (server-side request forgery): an attacker makes your server call internal addresses. SSRF had its own category in 2021.
    *   *Software supply chain failures*: Risk from dependencies, build systems, and packages you didn't write (new in 2025; replaces "vulnerable and outdated components").
    *   *Injection*: Input is run as code or query text (SQL injection, XSS).
*   **Zero Trust**: Being inside the network grants nothing. Every user and service request is authenticated and authorized.
*   **Encryption at rest**: Data on disk or in fields is encrypted (e.g., AES-256), with keys in a key management service (KMS).
*   **Encryption in transit**: TLS on every hop, internal ones too.
*   **Secrets management**: Keys and passwords live in a vault (AWS Secrets Manager, HashiCorp Vault) and are rotated, not kept in code.

### 3. Real-World Usage
*   **Passwords**: A slow, salted hash (Argon2id, bcrypt, scrypt). The salt stops precomputed tables; the slowness makes guessing expensive.
*   **Link previews**: Fetch user URLs through an egress proxy, allow-list destinations, and block internal and cloud metadata IPs after DNS lookup.
*   **Least privilege**: A service that only reads a bucket gets read access only.
*   **OIDC login**: Let an identity provider handle passwords.

### 4. Tradeoffs
*   **Security vs. convenience**: MFA and short sessions stop account takeover but annoy users.
*   **Dependency scans vs. CI speed**: Snyk, `npm audit`, or Dependabot add time and noise, but catch known-vulnerable versions.
*   **Vault vs. availability**: Safer than env files, but if the vault is down, services may not start. Cache credentials.

### 5. When NOT to Use
*   **Home-made crypto**: Don't invent algorithms or combine primitives yourself. Use vetted libraries (libsodium, your platform's TLS).
*   **Hidden URLs as security**: A secret admin URL gets found. Require real auth.

### 6. Interview Focus
*   **`GET /api/users/:id` review**: "What are the risks?"
    * No authentication: anyone can call it. Require a verified token.
    * IDOR: a logged-in user changes `123` to `456`. Check the caller is that user or has the right role.
    * SQL injection: `"... WHERE id = " + id` lets `123 OR 1=1` change the query. Use parameterized queries; the values are sent apart from the SQL text and never parsed as SQL.
*   **Zero Trust between services**: "How?"
    * mTLS with short-lived certificates per service, often issued by a service mesh (Istio, Linkerd).
    * Explicit rules for which service may call which endpoint.
    * Pass a signed token for the end user, so services authorize on that user's behalf.
*   **Database leaked**: "What limits the damage?"
    * Slow, salted password hashes. Strong passwords stay safe; weak ones can still be cracked, so add MFA.
    * Store only hashes of session tokens, API keys, and reset tokens, so leaked rows can't be reused.
    * Encrypt sensitive fields in the app with KMS keys. Disk encryption doesn't help: the database decrypts for anyone who can query it.
    * Store less data.
    * Revoke sessions and rotate secrets once the breach is found.
    * Audit logs and alerts on unusual bulk reads.

### 7. Common Mistakes
*   **Secrets in logs**: Passwords, tokens, or card numbers sent to Sentry, Splunk, or CloudWatch. Redact in the logger.
*   **Trusting the internal network**: One hacked service can then reach everything else (lateral movement).
*   **Default credentials**: `admin/admin` left on any component.

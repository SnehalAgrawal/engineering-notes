# Incident Management & Post-Mortems

### 1. Overview
Leads are often paged when production breaks. Incident management covers what you do during the outage (restore service) and after it (learn and prevent repeats). Stay calm, split the work, and keep the review blameless.

### 2. Key Concepts
*   **Incident Commander (IC)**: Runs the response. Assigns roles, keeps the timeline, and updates stakeholders. The IC does not debug.
*   **MTTD**: Mean Time to Detect. How long until anyone noticed.
*   **MTTR**: Mean Time to Recover (or Restore). How long until users were fine again.
*   **Blameless post-mortem**: A review that asks how the system allowed the failure, not who made the mistake. Amazon calls its version a COE (Correction of Errors).
*   **Runbook**: Step-by-step instructions for a known problem, such as "Redis is out of memory".

### 3. Real-World Usage
*   **Sev-1 outage**: Payments fail on a peak sales day. The IC opens an incident channel, one engineer checks logs, another prepares a rollback, and support gets updates on a fixed schedule.
*   **5 Whys**: Ask "why" until you reach a fixable cause. Database crashed. Why? Disk full. Why? Logs filled it. Why? Debug logging was left on. Why? Nothing checks the log level before deploy. Real incidents usually have several contributing causes, so do not stop at one chain.
*   **Action items**: Tracked tickets with owners and dates, such as "Add a pipeline check that blocks debug logging in production."

### 4. Tradeoffs
*   **Mitigate vs. investigate**: During the incident, restore service first (roll back, restart, fail over, turn off a feature flag). Finding the exact bug can wait.
*   **Paging**: Paging the whole team for every CPU spike trains people to ignore pages. Page only for user impact.

### 5. When NOT to Use
*   **Blame**: "Bob deployed bad code. Action: Bob must be more careful." This teaches people to hide mistakes.
*   **Full reviews for everything**: Formal post-mortems for Sev-1 and Sev-2. Minor issues get a short note.

### 6. Interview Focus
*   **Outage story**: "Tell me about the worst outage you handled."
    *   Cover detection, mitigation, communication, then the post-mortem and what changed afterward.
*   **Systemic fixes**: "An engineer drops a production table by hand. How do you prevent it?"
    *   Remove standing write access to production. Use break-glass access that is approved and logged.
    *   Run schema changes through reviewed migrations in CI/CD.
    *   Make sure backups and point-in-time restore work, and test them.
*   **Lower MTTR**: "Recovery takes two hours. How do you speed it up?"
    *   Better alerts on symptoms, one-command rollbacks, feature flags, runbooks, and practice drills.

### 7. Common Mistakes
*   **Tunnel vision**: Debugging silently for 40 minutes while leadership waits for an update. The IC owns communication.
*   **"Human error" as root cause**: If one person's mistake can take the system down, the system lacks guardrails.
*   **Forgotten action items**: A good post-mortem whose tickets sink in the backlog, so the same outage happens again.

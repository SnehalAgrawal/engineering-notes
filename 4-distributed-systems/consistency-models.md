# Consistency Models & Conflict Resolution

### 1. Overview
A consistency model says when a write becomes visible to reads. At one end, strong consistency acts like a single copy but is slower. At the other, eventual consistency is fast but reads can be stale. Pick the weakest model each kind of data can safely use.

### 2. Key Concepts
*   **Strong consistency (linearizability)**: Once a write finishes, every later read sees it.
*   **Eventual consistency**: If writes stop, all copies end up the same. No promise on when.
*   **Causal consistency**: Things that depend on each other are seen in order everywhere. A reply never shows up before the post it answers.
*   **Read-your-writes**: You always see your own changes.
*   **Monotonic reads**: Once you've seen a value, you never see an older one.
*   **Link to CAP**: Strong consistency can't stay available during a partition. Causal and the two session guarantees above can. See [HLD fundamentals](../3-hld/hld-fundamentals.md).
*   **Conflict**: Two copies accept different writes to the same data at the same time.
*   **Version vectors / vector clocks**: Tell whether two writes happened in order or at the same time.
*   **Merkle trees**: Quickly find which copies differ, so they can be synced. They find differences; they don't decide the winner.
*   **Last-writer-wins (LWW)**: Keep the write with the latest timestamp. Simple, but the other write is lost silently.
*   **CRDTs**: Data types (counters, sets, text) whose concurrent edits merge automatically with no lost updates.

### 3. Real-World Usage
*   **Inventory and seats**: The final "take one" step must be strongly consistent (a conditional write or transaction). Showing stock levels can be stale.
*   **Likes and view counts**: Eventual is fine. 100 vs. 102 likes for a few seconds hurts nobody.
*   **Collaborative editing**: Google Docs uses Operational Transformation; libraries like Yjs and Automerge use CRDTs.
*   **DNS**: Eventually consistent through caching. A change shows up as cached records expire (their TTL).
*   **Profile edits**: Read-your-writes, so users see their own change right away.

### 4. Tradeoffs
*   **Strong vs. eventual**: Strong avoids confusing results but adds wait time to writes.
*   **LWW**: Easy, but loses updates without any error.
*   **CRDTs**: No lost updates, but extra metadata, and they can't enforce rules like "balance never below zero".

### 5. When NOT to Use
*   **Strong consistency**: Analytics, logs, metrics, feeds. You pay latency for nothing.
*   **Eventual consistency**: Ledgers, unique usernames, stock decrements, permission revokes (a slow revoke is a security hole).

### 6. Interview Focus
*   **Multiplayer leaderboard**: Eventual for the global board, read-your-writes for a player's own score, strong only where rewards or money depend on it.
*   **Two users edit the same bio at once on different nodes**:
    * Version vectors detect that the writes are concurrent.
    * LWW is fine for a bio; or keep both and ask the user.
    * With a single-leader database there's no conflict: the leader orders the writes.
*   **Read-your-writes with async replicas**:
    * Read from the primary for a short time after a user writes, or keep the user on one replica (sticky sessions).
    * Or send the write's version with later reads and only use a replica that has caught up.

### 7. Common Mistakes
*   **Mixing up isolation and replica lag**: Isolation levels (Read Committed, Serializable) are about concurrent transactions. They don't stop stale reads from async replicas.
*   **Trusting the default isolation**: Read Committed (default in PostgreSQL and SQL Server) allows lost updates. Use Serializable or locks where it matters.
*   **Treating eventual as strong**: "Check stock, then decrement" on an eventually consistent store causes races.
*   **LWW for money**: Two withdrawals at once can end with only one recorded.

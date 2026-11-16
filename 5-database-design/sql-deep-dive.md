# SQL Deep Dive: Indexing, Transactions, & Normalization

### 1. Overview
Relational databases are the default for transactional data. The useful skills: know what an index costs, what each isolation level allows, how to read a query plan, and how to avoid locking problems.

### 2. Key Concepts
*   **Index**: A sorted structure (usually a B-tree) that finds rows without reading the whole table. Hash indexes only support equality.
    *   *Clustered index*: The table rows are stored in index order (InnoDB's primary key). PostgreSQL tables are unordered; its `CLUSTER` command sorts once and is not kept up.
    *   *Secondary index*: A separate structure that points back to the row.
    *   *Covering index*: Holds every column the query needs, so the table itself is not read.
*   **ACID**:
    *   *Atomicity*: All statements commit, or none do.
    *   *Consistency*: Constraints hold before and after.
    *   *Isolation*: How much concurrent transactions see of each other. Levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable. PostgreSQL defaults to Read Committed; MySQL InnoDB to Repeatable Read.
    *   *Durability*: Committed data survives a crash.
*   **Normalization (1NF to 3NF)**: Store each fact once, so an update can't leave copies out of sync.
*   **Denormalization**: Copy data on purpose to make some reads cheaper. Writes get more work.

### 3. Real-World Usage
*   **Money transfer**: Debit and credit in one transaction.
*   **Activity feed**: An index on `(user_id, created_at)` returns a user's latest events in one index scan.
*   **Integrity**: Foreign keys stop an order from pointing at a missing user. Check constraints block bad values, e.g., `CHECK (amount > 0)`.
*   **Reporting**: A star schema (one fact table, denormalized dimension tables) keeps queries to a few joins.

### 4. Tradeoffs
*   **Reads vs. writes**: Each index speeds some reads and slows the writes that must update it.
*   **Isolation vs. throughput**: Stronger levels block more anomalies but cost speed. Serializable either blocks (lock-based engines) or aborts conflicts (PostgreSQL), so the app must retry. Read Committed lets a row change between two reads in one transaction.
*   **Normalization vs. joins**: Joins on indexed keys are cheap. Many joins over large data are not.

### 5. When NOT to Use
*   **Shapeless documents**: Records with no stable structure, always read whole, may fit a document store. PostgreSQL `JSONB` covers many such cases.
*   **Huge append-only data**: Logs and metrics usually fit a time-series or columnar store better.

### 6. Interview Focus
*   **Query plans**: "A query that should be fast takes seconds. How do you use `EXPLAIN ANALYZE`?"
    * `EXPLAIN` shows the planned steps. `EXPLAIN ANALYZE` runs the query and shows real times. It really runs, so wrap writes in `BEGIN; ... ROLLBACK;`.
    * Look for full-table scans (`Seq Scan`), row estimates far from actual counts (run `ANALYZE`), nested-loop joins over large inputs where a hash join would be cheaper, and sorts spilling to disk.
    ```
    Seq Scan on users ... Filter: (email = 'a@b.com')                     -- reads every row
    Index Scan using idx_users_email ... Index Cond: (email = 'a@b.com')  -- jumps to the row
    ```
*   **Deadlocks**: "Two transactions update the same two rows in opposite order."
    ```sql
    -- T1: UPDATE users ... WHERE id = 1;  then  UPDATE users ... WHERE id = 2;
    -- T2: UPDATE users ... WHERE id = 2;  then  UPDATE users ... WHERE id = 1;
    ```
    * Each waits for the other. The database detects the cycle and aborts one (PostgreSQL error `40P01`). Nothing waits forever.
    * Prevent it by locking rows in the same order everywhere, e.g., `SELECT ... WHERE id IN (1, 2) ORDER BY id FOR UPDATE`.
    * Keep transactions short and retry the aborted one.
    * `lock_timeout` limits how long a statement waits for a lock. It is not needed to resolve deadlocks.
    * To see deadlocks: PostgreSQL logs them and counts them in `pg_stat_database.deadlocks`; MySQL shows the latest in `SHOW ENGINE INNODB STATUS`.
*   **Many-to-many**: "Students take many classes; classes have many students."
    ```sql
    CREATE TABLE enrollments (
        student_id INT REFERENCES students(id),
        class_id   INT REFERENCES classes(id),
        PRIMARY KEY (student_id, class_id)
    );
    CREATE INDEX ON enrollments (class_id);
    ```
    * The primary key blocks duplicate enrollments and serves "classes for a student". The extra index serves "students in a class".
    * A student's classes: `SELECT c.name FROM enrollments e JOIN classes c ON c.id = e.class_id WHERE e.student_id = 1;`
*   **E-commerce schema**: "Sketch the main tables."
    * `users`, `products`, `orders` (user, status, created_at), `order_items` (order, product, quantity, unit price).
    * Copy the price into `order_items`, so later price changes don't rewrite old orders.
    * Keep stock in its own table and update it in the same transaction as the order.
    * Index `orders (user_id, created_at)` for "my orders".
*   **CTE + window function**: "Each customer's three latest orders."
    ```sql
    WITH ranked AS (
        SELECT customer_id, id, created_at,
               ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
        FROM orders
    )
    SELECT customer_id, id, created_at FROM ranked WHERE rn <= 3;
    ```
    * Window functions run after `WHERE`, so filtering on `rn` needs the CTE. Use `RANK()` to keep ties.

### 7. Common Mistakes
*   **`SELECT *`**: Reads columns you don't need and rules out covering indexes.
*   **Indexing every column**: Slows every write. Check which indexes are actually used.
*   **N+1 queries**: One query per item in a loop instead of one join or one `IN (...)` query.

# Data Structures & Algorithms for Seniors

### 1. Overview
At senior level, data structures are engineering choices. Memory layout, cache behavior, and constant factors decide what is fast in practice, not just big-O.

### 2. Key Concepts
*   **Trie**: A tree keyed by prefixes. Used for autocomplete and longest-prefix match in IP routing.
*   **DAG**: Directed acyclic graph: arrows, no loops. Models dependencies (build systems, Airflow, Git history).
*   **LSM tree**: Buffers writes in memory, flushes sorted files, merges them in the background. Fast writes (RocksDB, LevelDB, Cassandra).
*   **Hash map internals**: Load factor, resizing, and collisions (chaining vs. open addressing).
*   **Amortized cost**: Average cost per operation over a worst-case sequence. Array append is O(1) amortized, though a resize is O(n). Not the same as average case.
*   **Space-time tradeoff**: Use more memory (hash maps, caches, precomputed tables) to save computation.
*   **Cache locality**: Reading memory in order (arrays) is much faster than following pointers (linked lists). CPUs load whole cache lines and prefetch the next ones.

**Problem patterns to recognize** (most coding questions are one of these):
*   **Two pointers**: Two indexes moving through a sorted array or a string. Example: find a pair that sums to a target in O(n).
*   **Sliding window**: Grow and shrink a range over an array or string. Example: the longest substring without repeated characters.
*   **Binary search**: On a sorted array, or on the answer itself ("smallest capacity that ships everything in D days").
*   **BFS / DFS**: BFS finds the shortest path when every edge costs the same; DFS explores and finds cycles. Topological sort orders a DAG.
*   **Dijkstra**: Shortest path when edges have non-negative weights. A min-heap picks the closest unvisited node next. O((V + E) log V).
*   **Heap**: Keep the top K items, or merge K sorted lists. A heap of size K costs O(n log K).
*   **Dynamic programming**: The problem splits into overlapping subproblems. Store each answer once (memoization or a table). Examples: coin change, longest common subsequence.
*   **Greedy**: Take the best local choice each step. Works only when that provably leads to the best overall answer (interval scheduling); otherwise use DP.
*   **Backtracking**: Try a choice, recurse, undo it. Used for permutations, subsets, and puzzles like N-Queens.
*   **Union-find**: Tracks which items are in the same group. Used for connected components and detecting cycles in a graph.
*   **Sorting**: Merge sort is O(n log n) and stable; quicksort is O(n log n) on average, O(n^2) in the worst case, and fast in practice; radix sort is O(n·k) for fixed-size keys.

### 3. Real-World Usage
*   **Build systems (Bazel, Make)**: A topological sort of the DAG gives the build order and what to rebuild.
*   **Rate limiting**: A sliding-window log as a Redis sorted set of request timestamps.
*   **Autocomplete**: A trie, or a compressed form (radix tree) to save memory.
*   **Database indexes**: B-trees have many children per node, so even huge tables need only a few page reads per lookup.

### 4. Tradeoffs
*   **Array vs. linked list**: Arrays have better locality. Linked lists insert in O(1) only if you already hold the node.
*   **Recursion vs. loop**: Recursion reads well for trees but can overflow the stack on deep input.
*   **Bloom filter vs. hash set**: A Bloom filter says "maybe present" or "definitely absent" in far less memory. A hash set is exact but uses much more.

### 5. When NOT to Use
*   **Custom implementations**: Use the standard library's maps and sorts unless profiling shows a need.
*   **Over-engineering**: A linear scan over a hundred items beats building a trie.

### 6. Interview Focus
*   **Top 10 errors in a billion log lines, little memory**:
    * If distinct messages fit: a hash map of counts plus a min-heap of size 10.
    * If not: split lines into files by hash, count each file, merge.
    * Approximate: Count-Min Sketch plus a heap.
*   **Git common ancestor**: `git merge-base` walks the commit DAG back from both branch tips until they meet.
*   **`ArrayList` vs. `LinkedList` inserts**: The linked list must first walk to the spot through scattered nodes. The array shifts one block of memory, which is fast.

### 7. Common Mistakes
*   **Only counting big-O**: For small N, a simple O(N^2) algorithm can beat a "better" one with high overhead.
*   **Unbalanced trees**: A plain binary search tree becomes a list on sorted input: O(N) per lookup. Use AVL or red-black trees.
*   **Many tiny objects**: Billions of small heap objects cause high memory use, long GC pauses, or out-of-memory errors. Use primitive arrays.

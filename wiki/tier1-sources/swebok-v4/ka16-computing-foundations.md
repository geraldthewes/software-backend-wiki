# Computing Foundations (KA16)

> **Tier 1** | Source: SWEBOK V4, Chapter 16 | Authority: immutable

## Summary

Computing Foundations covers the computer science concepts that underpin all software engineering practice: algorithms and data structures, programming paradigms, operating systems, databases, networking, concurrency, and computer architecture. These are not theoretical curiosities — they are the principles that determine whether a system is fast or slow, correct or buggy, secure or vulnerable. SWEBOK V4 adds AI/ML as a new computing foundation topic, reflecting its growing presence in production systems.

For agents, computing foundations determine code quality at the technical level. Choosing the wrong data structure is a performance bug. Ignoring concurrency semantics is a correctness bug. Misunderstanding the network is a reliability bug. Every implementation decision should be grounded in these foundations, not in habit or familiarity.

## Key Concepts

### Algorithms and Data Structures

**Big-O notation**: Describes how time or space requirements grow as input size increases.
- O(1): Constant — hash map lookup, array index access
- O(log n): Logarithmic — binary search, balanced tree operations
- O(n): Linear — array scan, linked list traversal
- O(n log n): Quasi-linear — efficient sorting (merge sort, heap sort)
- O(n²): Quadratic — naive nested loops over collections
- O(2^n): Exponential — brute-force combinatorial search (avoid in production)

**Choosing by access pattern**:
- Random access by index → Array/list
- Frequent insertion/deletion at arbitrary position → Linked list
- Key-based lookup → Hash map (O(1) average)
- Ordered iteration + binary search → Sorted array or balanced BST
- Graph traversal, dependency analysis → Adjacency list + BFS/DFS
- Priority queue / scheduling → Heap

**Sorting**: Know when the built-in sort is sufficient (most of the time) vs. when to use specialized sorts (radix sort for integers, counting sort for small ranges). Default to the language's standard sort; it is almost certainly a well-optimized comparison sort.

### Programming Paradigms

**Imperative**: Step-by-step instructions that modify state. Natural for procedural code, system programming, performance-critical loops.

**Object-Oriented (OOP)**: Models the domain as interacting objects with state (fields) and behavior (methods). Suited to complex domains with many entity types and rich relationships. Apply SOLID principles.

**Functional (FP)**: Computation as evaluation of pure functions over immutable data. Avoids shared mutable state. Excellent for data transformation pipelines, concurrent code, and testable business logic. "Functional Core, Imperative Shell" is a productive hybrid architecture.

**Declarative**: Specifies *what* to compute, not *how*. SQL, HTML, Terraform HCL, CSS are declarative. Prefer declarative where the domain has a natural declarative form — it is often more readable and less error-prone than equivalent imperative code.

**Choosing a paradigm**: Use OOP for stateful domain modeling; FP for data transformation and concurrency; declarative for configuration, queries, and infrastructure.

### Operating Systems

**Processes vs. threads**:
- Process: Independent execution unit with its own memory space. Isolated — crash of one process does not corrupt another.
- Thread: Lightweight execution unit sharing the parent process's memory. Communication is fast (shared memory) but introduces concurrency hazards.

**Memory management**: Stack (fast, limited, automatic) vs. heap (flexible, slower, manual or GC-managed). Memory leaks, use-after-free, and buffer overflows all stem from incorrect heap management.

**File systems**: Durability (fdatasync/fsync before claiming persistence), atomicity (rename for atomic file replacement), file locking for concurrent access.

**IPC (Inter-Process Communication)**: Pipes, sockets, message queues, shared memory. Choose based on locality (same machine vs. network) and coupling requirements.

### Databases

**Relational (SQL)**:
- ACID guarantees: Atomicity (all or nothing), Consistency (invariants maintained), Isolation (transactions do not interfere), Durability (committed data survives crash)
- Strong for complex queries, referential integrity, and transactional workflows
- PostgreSQL is the default choice for relational needs; it is the most capable open-source RDBMS

**NoSQL types and when to use them**:
- **Document** (MongoDB, CouchDB): Schema-flexible JSON documents. Good for heterogeneous content, rapid schema iteration. Weak on joins and cross-document transactions.
- **Key-value** (Redis, DynamoDB): Fastest lookup by key. Good for caching, session storage, rate limiting, feature flags.
- **Column-family** (Cassandra, HBase): Optimized for wide-row scans with high write throughput. Good for time-series data, event logs, IoT.
- **Graph** (Neo4j, Amazon Neptune): Optimized for traversing dense relationship networks. Good for social graphs, recommendation engines, fraud detection.

**ACID vs. BASE**: Most NoSQL systems trade ACID guarantees for BASE properties: Basically Available, Soft-state, Eventually consistent. Choose ACID when correctness requires it; BASE when availability and scale take priority.

### Networking

**TCP/IP stack**: Understand the layers — application (HTTP, gRPC), transport (TCP, UDP), network (IP), link (Ethernet, WiFi). Most backend engineering operates at the application layer, but TCP behaviors (connection setup cost, head-of-line blocking, retransmit delays) affect performance.

**HTTP/HTTPS**: The protocol of APIs. Understand status codes, headers, idempotent methods (GET, PUT, DELETE vs. POST), connection keep-alive, and TLS handshake cost.

**DNS**: Name resolution adds latency on every new connection. DNS caching behavior (TTL) affects service failover speed. Service mesh and internal DNS are common in microservice environments.

**REST vs. gRPC**: REST is simple, human-readable, widely tooled, stateless. gRPC is binary (Protocol Buffers), lower latency, bidirectional streaming, strongly typed — suited for internal service-to-service communication.

**The 8 Fallacies of Distributed Computing**: The network is NOT reliable, NOT zero-latency, NOT infinite bandwidth, NOT secure, NOT consistent topology, NOT administered by one person, NOT zero-cost, and is NOT homogeneous. Design accordingly.

### Concurrency

**Race conditions**: Two threads accessing shared state without synchronization; the outcome depends on scheduling order. Prevent with mutexes, atomic operations, or by eliminating shared mutable state.

**Deadlock**: Two or more threads each waiting for a resource held by the other. Prevent by always acquiring locks in a consistent order and using lock timeouts.

**Livelock**: Threads continuously change state in response to each other but make no progress. Less common than deadlock; harder to detect.

**Starvation**: A thread is perpetually denied the resources it needs. Common with unfair scheduling or priority inversion.

**Synchronization primitives**:
- **Mutex (mutual exclusion lock)**: Only one thread can hold it at a time. Use for protecting shared mutable state.
- **Semaphore**: Generalized mutex allowing N concurrent holders. Use for resource pool limiting.
- **Monitor (condition variable + mutex)**: Used for producer-consumer patterns and wait/notify coordination.

**Async I/O**: Event-loop based concurrency (Python asyncio, Node.js, Go goroutines) handles many concurrent I/O-bound operations with fewer threads than blocking I/O. Not suitable for CPU-bound work — use multiprocessing for that.

### Computer Architecture

**CPU cache hierarchy**: L1 (fastest, ~4 cycles, ~32KB) → L2 (~12 cycles, ~256KB) → L3 (~40 cycles, ~8MB) → RAM (~200 cycles). Cache misses are expensive. Cache-friendly access patterns (sequential array traversal) outperform pointer-chasing (linked list traversal) dramatically.

**Branch prediction**: CPUs predict branches and speculatively execute. Unpredictable branches (random data) cause pipeline stalls. Consistent branch patterns are faster.

**Memory alignment**: Misaligned memory access causes performance penalties on some architectures. Most languages handle this automatically, but it matters in performance-critical serialization code.

### AI/ML (V4 Addition)

**Basic ML pipeline**: Data collection → feature engineering → model training → evaluation → deployment → monitoring. Each step is an engineering problem with failure modes.

**When ML is appropriate**:
- The problem does not have a clear algorithmic solution
- Sufficient labeled training data exists or can be obtained
- The cost of errors in production is acceptable (or mitigated by human review)
- The system can be monitored for distribution shift (training data vs. production data divergence)

**ML system risks**:
- **Data leakage**: Training data includes information not available at inference time → inflated evaluation metrics → poor production performance
- **Distribution shift**: Production data differs from training data → model accuracy degrades silently
- **Algorithmic bias**: Training data encodes historical biases → model discriminates unfairly
- **Opacity**: Complex models are difficult to explain → regulatory and ethical risk

## Agent Guidance

### Do
- Choose data structures by access pattern, not familiarity — look up the complexity characteristics
- Understand time and space complexity of algorithms before selecting them for production code
- Treat the network as unreliable (8 Fallacies) — always add timeouts, retries, and circuit breakers
- Consider concurrency implications when writing any code that accesses shared state
- Use async I/O for I/O-bound concurrent services; use multiprocessing for CPU-bound work
- Assess ML system risks (bias, distribution shift, data leakage) before deploying ML components

### Do Not
- Default to linked lists or trees when a hash map or array would be faster for the access pattern
- Write unbounded concurrent code that can deadlock or starve under contention
- Assume network calls succeed — always handle timeouts and partial failures
- Choose SQL vs. NoSQL based on hype — choose based on access patterns, consistency requirements, and query complexity
- Deploy ML models without a monitoring plan for distribution shift and accuracy degradation

## Checklist
- [ ] Data structures chosen based on access pattern and complexity analysis
- [ ] Network calls have explicit timeouts, retry limits, and circuit breakers
- [ ] Shared mutable state protected by appropriate synchronization
- [ ] Database technology chosen based on ACID vs. BASE requirements and access patterns
- [ ] Async I/O used for I/O-bound concurrency; multiprocessing for CPU-bound concurrency
- [ ] ML pipeline includes evaluation, monitoring, and bias assessment steps

## See Also
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`
- `wiki/tier1-sources/swebok-v4/ka17-mathematical-foundations.md`
- `wiki/tier2-core/distributed-systems/overview.md`
- `wiki/tier3-working/python/async-patterns.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 16: Computing Foundations. IEEE Press, 2024.

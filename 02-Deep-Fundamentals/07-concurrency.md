# Concurrency

## 1. Problem Statement

How do we handle multiple requests or operations happening simultaneously without conflicts or data corruption?

---

## 2. Core Concepts

### Concurrency vs Parallelism

**Concurrency:**
* Multiple tasks making progress (not necessarily at same time)
* Single CPU, time-slicing
* Example: Web server handling multiple requests

**Parallelism:**
* Multiple tasks executing simultaneously
* Multiple CPUs/cores
* Example: Processing 1000 images on 10 cores

**Key Insight**: Concurrency is about structure, parallelism is about execution.

---

## 3. Concurrency Challenges

### Race Conditions
* Multiple threads access shared data
* Outcome depends on timing
* **Solution**: Locks, atomic operations, immutability

### Deadlocks
* Two or more processes waiting for each other
* System hangs
* **Solution**: Lock ordering, timeouts, avoid nested locks

### Data Consistency
* Concurrent writes to same data
* Inconsistent state
* **Solution**: Transactions, locks, versioning

---

## 4. Concurrency Patterns

### Locks
* **Mutex**: Mutual exclusion (one at a time)
* **Read-Write Lock**: Multiple readers, single writer
* **Distributed Lock**: Across multiple servers (Redis, etcd)

**Trade-off**: Safety vs Performance
* Locks ensure correctness but can reduce throughput
* Fine-grained locks: Better performance, more complex
* Coarse-grained locks: Simpler, but lower performance

### Atomic Operations
* Operations that complete entirely or not at all
* No intermediate state visible
* **Example**: Increment counter atomically

### Immutability
* Data never changes after creation
* No locks needed for reads
* **Trade-off**: Memory usage (copy on write)

### Message Passing
* Threads communicate via messages
* No shared state
* **Example**: Actor model, message queues

---

## 5. Database Concurrency

### Isolation Levels
* **Read Uncommitted**: No isolation (dirty reads)
* **Read Committed**: No dirty reads
* **Repeatable Read**: No phantom reads
* **Serializable**: Highest isolation (slowest)

### Optimistic vs Pessimistic Locking
* **Optimistic**: Assume no conflicts, check at commit
* **Pessimistic**: Lock before access
* **Trade-off**: Optimistic is faster when conflicts are rare

### Versioning
* Each update increments version number
* Reject updates with stale version
* **Example**: ETags in HTTP, row versioning in DB

---

## 6. Distributed Concurrency

### Distributed Locks
* **Redis**: SET NX (set if not exists) with expiration
* **etcd/ZooKeeper**: Consensus-based locking
* **Chubby**: Google's distributed lock service

**Challenges:**
* Network partitions
* Clock skew
* Lock expiration

### Vector Clocks
* Track causality in distributed systems
* Each node maintains vector of timestamps
* **Use Case**: Event ordering in distributed systems

### CRDTs (Conflict-Free Replicated Data Types)
* Data structures that merge automatically
* No conflicts, eventual consistency
* **Example**: Distributed counters, sets

---

## 7. How to Explain This in an Interview

> "For handling concurrent requests, I use connection pooling to reuse database connections efficiently. For shared state, I use Redis with atomic operations (INCR) to avoid race conditions. For distributed systems, I use distributed locks (Redis SET NX) when I need mutual exclusion across servers, but I prefer stateless design to avoid locks when possible."

**Key Points:**
* Prefer stateless design (no shared state)
* Use atomic operations when possible
* Use locks only when necessary
* Consider optimistic locking for better performance

---

## 8. Common Mistakes

* **Over-locking**: "Lock everything to be safe"
  * Fix: Lock only what's necessary, use fine-grained locks
* **Deadlocks**: "Multiple locks in different order"
  * Fix: Always acquire locks in same order, use timeouts
* **Ignoring race conditions**: "It works in testing"
  * Fix: Use proper synchronization, test under concurrency
* **Not handling lock failures**: "Lock always succeeds"
  * Fix: Handle timeouts, retries, fallback strategies
* **Shared mutable state**: "Global variables are fine"
  * Fix: Prefer immutability, message passing, or proper locking

# Consistency Models

## 1. Problem Statement

How do we ensure data correctness across distributed systems when nodes can fail or be slow?

---

## 2. Clarifying Questions

* What operations require strong consistency?
* Can users tolerate stale data?
* What's the replication strategy?
* Geographic distribution?

---

## 3. Requirements

### Functional

* Read latest written data (strong consistency)
* OR read eventually consistent data (weak consistency)
* Handle network partitions

### Non-Functional

* Availability vs consistency trade-off
* Performance under consistency model
* Recovery from failures

---

## 4. Scale Assumptions

* Multi-region deployment
* Network partitions possible
* 100K reads/sec, 10K writes/sec
* Replication lag: 10-100ms

---

## 5. High-Level Architecture

**Strong Consistency (CP)**
```
Write → Primary → Sync Replication → Replicas
Read → Primary (or sync replicas)
```

**Eventual Consistency (AP)**
```
Write → Any Node → Async Replication → Other Nodes
Read → Any Node (may be stale)
```

---

## 6. Core Components

* **Primary-Secondary Replication**: Strong consistency
* **Multi-Master Replication**: Eventual consistency
* **Quorum Reads/Writes**: Balance consistency and availability
* **Vector Clocks**: Track causality in distributed systems
* **Conflict Resolution**: Merge strategies for eventual consistency

---

## 7. Data Flow

**Strong Consistency (Synchronous)**
1. Write → Primary DB
2. Primary → Sync replicate to 2 replicas
3. Wait for ACK from replicas
4. Return success to client
5. Read → Primary or sync replicas (guaranteed latest)

**Eventual Consistency (Asynchronous)**
1. Write → Any node
2. Node → Async replicate to others
3. Return success immediately
4. Read → Any node (may return stale data)
5. Eventually all nodes converge

---

## 8. Bottlenecks

* **Synchronous Replication**: Blocks writes until all replicas ACK
  * Solution: Use async for non-critical data
* **Network Partitions**: Can't achieve both C and A (CAP theorem)
  * Solution: Choose based on use case
* **Read Your Writes**: User writes, then reads stale data
  * Solution: Read from primary or use session affinity
* **Conflict Resolution**: Concurrent writes to same key
  * Solution: Last-write-wins, vector clocks, or application logic

---

## 9. Trade-offs

| Model | Consistency | Availability | Latency | Use Cases |
|-------|-------------|--------------|---------|-----------|
| **Strong (CP)** | ✅ Always latest | ❌ Blocks on partition | Higher | Financial transactions, leader election |
| **Eventual (AP)** | ❌ May be stale | ✅ Always available | Lower | Social feeds, counters, analytics |
| **Read Your Writes** | ✅ Own writes visible | ✅ Available | Medium | User profiles, settings |
| **Monotonic Reads** | ✅ Never see older data | ✅ Available | Medium | Timeline views |
| **Causal Consistency** | ✅ Causally related reads consistent | ✅ Available | Medium | Comments, replies |

**CAP Theorem**: You can only guarantee 2 of 3:
* **Consistency**: All nodes see same data
* **Availability**: System responds to requests
* **Partition Tolerance**: System works despite network failures

---

## 10. How to Explain This in an Interview

> "For financial transactions, I choose strong consistency (CP) because correctness is non-negotiable, even if it means blocking during partitions. For social feeds, I choose eventual consistency (AP) because users can tolerate slightly stale data, and availability is more important than perfect consistency."

**Decision Framework:**
* **Money/Transactions**: Strong consistency
* **Social/Feeds**: Eventual consistency
* **User Profiles**: Read-your-writes consistency
* **Analytics**: Eventual consistency

---

## 11. Common Mistakes

* **Over-engineering consistency**: "Everything needs strong consistency"
  * Fix: Most systems can use eventual consistency
* **Ignoring CAP theorem**: "We have both C and A"
  * Fix: During partitions, you must choose
* **Not handling read-your-writes**: "User sees their own stale data"
  * Fix: Route user's reads to primary or use session affinity
* **No conflict resolution**: "What if two users edit simultaneously?"
  * Fix: Define merge strategy (last-write-wins, CRDTs, or app logic)

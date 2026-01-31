# Databases

## 1. Problem Statement

How do we choose and design database systems to store and retrieve data at scale with the right consistency, latency, and availability guarantees?

---

## 2. Clarifying Questions

* What's the data model? (Relational vs document vs key-value)
* What's the access pattern? (Read-heavy vs write-heavy)
* What's the consistency requirement? (Strong vs eventual)
* What's the scale? (QPS, data size)

---

## 3. Requirements

### Functional

* Store and retrieve data
* Support queries efficiently
* Handle transactions (if needed)
* Scale horizontally or vertically

### Non-Functional

* Low latency (<10ms for reads)
* High availability (99.99%)
* Durability (data not lost)
* Scalability (handle growth)

---

## 4. Scale Assumptions

* 1M QPS reads, 100K QPS writes
* 10TB to 1PB data
* Global distribution
* 99.99% availability requirement

---

## 5. High-Level Architecture

**SQL (Relational)**
```
App → Primary DB → Sync Replicas (for reads)
```

**NoSQL (Distributed)**
```
App → Sharded DB Cluster → Multiple Replicas per Shard
```

---

## 6. Core Components

* **Primary/Leader**: Handles writes
* **Replicas/Followers**: Handle reads, backup
* **Shards**: Partition data horizontally
* **Indexes**: Speed up queries
* **Connection Pool**: Reuse connections

---

## 7. Data Flow

**Write Flow (SQL)**
1. App → Primary DB
2. Primary → Write to disk (WAL)
3. Primary → Replicate to replicas (sync/async)
4. Primary → Return success
5. Replicas → Apply changes

**Read Flow (SQL)**
1. App → Read replica (or primary)
2. Replica → Check cache/index
3. Replica → Query data
4. Replica → Return result

**Write Flow (NoSQL Sharded)**
1. App → Determine shard (hash key)
2. App → Write to shard leader
3. Shard leader → Replicate to shard replicas
4. Shard leader → Return success

---

## 8. Bottlenecks

* **Single Primary**: Write bottleneck
  * Solution: Sharding, or read replicas for reads
* **Slow Queries**: No indexes, full table scans
  * Solution: Add indexes, optimize queries
* **Connection Exhaustion**: Too many connections
  * Solution: Connection pooling
* **Hot Shards**: Uneven data distribution
  * Solution: Better sharding key, rebalance
* **Replication Lag**: Reads see stale data
  * Solution: Read from primary for critical reads, or accept lag

---

## 9. Trade-offs

| Database Type | Pros | Cons | When to Use |
|---------------|------|------|-------------|
| **SQL (MySQL/Postgres)** | ACID, complex queries, mature | Hard to scale writes, single primary | Strong consistency needed, complex queries |
| **NoSQL (DynamoDB/Cassandra)** | Horizontal scale, high availability | No joins, eventual consistency | High write volume, simple queries |
| **Document (MongoDB)** | Flexible schema, JSON native | Less mature, scaling challenges | Rapid iteration, document data |
| **Key-Value (Redis)** | Ultra-fast, simple | Memory-limited, no complex queries | Caching, session storage |

**Replication Strategies:**
- **Synchronous**: Strong consistency, but blocks on slow replicas
- **Asynchronous**: Better performance, but eventual consistency

**Sharding Strategies:**
- **Range-based**: Simple, but can create hot shards
- **Hash-based**: Even distribution, but hard to range queries
- **Directory-based**: Flexible, but single point of failure

---

## 10. How to Explain This in an Interview

> "For this system, I choose NoSQL (like DynamoDB) because we need to handle 1M QPS writes with horizontal scalability. The trade-off is we lose ACID transactions and complex joins, but for this use case, we don't need joins and eventual consistency is acceptable. I shard by user_id using consistent hashing to distribute load evenly."

**Decision Framework:**
* **Need ACID + complex queries?** → SQL
* **Need horizontal scale + simple queries?** → NoSQL
* **Read-heavy?** → Read replicas
* **Write-heavy?** → Sharding
* **Need sub-ms latency?** → In-memory (Redis)

---

## 11. Common Mistakes

* **SQL for everything**: "We use MySQL for scale"
  * Fix: Use NoSQL when you need horizontal write scaling
* **No read replicas**: "All reads go to primary"
  * Fix: Use read replicas for read-heavy workloads
* **Wrong sharding key**: "We shard randomly"
  * Fix: Shard by access pattern (user_id, not random)
* **No indexes**: "Queries are slow"
  * Fix: Add indexes on frequently queried fields
* **Ignoring replication lag**: "Reads are always consistent"
  * Fix: Acknowledge eventual consistency, or read from primary
* **Over-normalization**: "We have 20 tables for user data"
  * Fix: Denormalize for read performance when needed

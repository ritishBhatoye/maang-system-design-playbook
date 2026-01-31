# SQL vs NoSQL

## 1. Problem Statement

When do we choose a relational database (SQL) versus a non-relational database (NoSQL) for our system?

---

## 2. Clarifying Questions

* What's the data model? (Structured vs flexible)
* What's the query pattern? (Complex joins vs simple lookups)
* What's the consistency requirement? (ACID vs eventual)
* What's the scale? (QPS, data size)

---

## 3. Requirements

### Functional

* Store and query data efficiently
* Support transactions (if needed)
* Handle relationships (if needed)
* Scale to requirements

### Non-Functional

* Low latency
* High availability
* Horizontal scalability
* Cost-effective

---

## 4. Scale Assumptions

* **SQL Use Case**: 10K QPS, complex queries, ACID needed
* **NoSQL Use Case**: 1M QPS, simple queries, scale needed
* Data size: TBs to PBs
* Global distribution

---

## 5. High-Level Architecture

**SQL Architecture**
```
App → Primary DB (writes) → Sync Replicas (reads)
     Complex joins, ACID transactions
```

**NoSQL Architecture**
```
App → Sharded Cluster → Multiple replicas per shard
     Simple lookups, eventual consistency
```

---

## 6. Core Components

**SQL:**
* Tables with relationships
* ACID transactions
* Complex queries (joins, aggregations)
* Schema enforcement

**NoSQL:**
* Key-value, document, or wide-column stores
* Eventual consistency (typically)
* Simple queries (by key)
* Flexible schema

---

## 7. Data Flow

**SQL Write Flow**
1. App → Begin transaction
2. App → Write to multiple tables (with joins)
3. DB → Validate constraints, enforce ACID
4. DB → Commit transaction (all or nothing)
5. DB → Replicate to read replicas (sync/async)

**NoSQL Write Flow**
1. App → Determine shard (hash key)
2. App → Write to shard leader
3. Shard → Replicate to shard replicas (async)
4. Shard → Return success immediately
5. Replicas → Eventually consistent

---

## 8. Bottlenecks

**SQL Bottlenecks:**
* **Single Primary**: Write bottleneck
  * Solution: Read replicas for reads, but writes still single point
* **Complex Queries**: Joins across tables
  * Solution: Denormalize, or use NoSQL for that use case
* **Vertical Scaling**: Can't scale writes horizontally easily
  * Solution: Sharding (complex), or use NoSQL

**NoSQL Bottlenecks:**
* **No Joins**: Must denormalize or application-level joins
  * Solution: Accept denormalization, or use SQL for that query
* **Eventual Consistency**: May read stale data
  * Solution: Read from leader for critical reads, or accept staleness
* **Sharding Complexity**: Rebalancing, cross-shard queries
  * Solution: Choose good shard key, avoid cross-shard queries

---

## 9. Trade-offs

| Aspect | SQL | NoSQL | Winner |
|--------|-----|-------|--------|
| **ACID Transactions** | ✅ Full support | ❌ Limited/None | SQL |
| **Complex Queries** | ✅ Joins, aggregations | ❌ Simple lookups | SQL |
| **Horizontal Scaling** | ❌ Difficult | ✅ Native | NoSQL |
| **Schema Flexibility** | ❌ Rigid | ✅ Flexible | NoSQL |
| **Consistency** | ✅ Strong | ❌ Eventual (typically) | SQL |
| **Write Throughput** | ❌ Limited by primary | ✅ Scales with shards | NoSQL |
| **Maturity** | ✅ Very mature | ⚠️ Varies | SQL |
| **Cost** | ⚠️ Can be expensive | ✅ Cost-effective at scale | NoSQL |

**Decision Matrix:**

**Choose SQL when:**
* Need ACID transactions (financial, payments)
* Complex queries with joins
* Strong consistency required
* Schema is well-defined and stable
* Moderate scale (<100K QPS writes)

**Choose NoSQL when:**
* Need horizontal write scaling (>100K QPS)
* Simple query patterns (key lookups)
* Flexible schema (rapid iteration)
* Can tolerate eventual consistency
* Global distribution needed

---

## 10. How to Explain This in an Interview

> "I choose NoSQL (DynamoDB) because we need to handle 1M QPS writes with horizontal scalability. The trade-off is we lose ACID transactions and complex joins, but for this use case, we don't need joins and eventual consistency is acceptable. If we needed financial transactions, I'd choose SQL for ACID guarantees."

**Alternative:**
> "I choose SQL (PostgreSQL) because we need ACID transactions for payment processing and complex queries with joins for reporting. The trade-off is we can't scale writes horizontally easily, but for 10K QPS writes, vertical scaling and read replicas are sufficient."

**Key Points:**
* Always mention the specific requirement that drives the choice
* Acknowledge the trade-off explicitly
* Quantify the scale (QPS) to justify the choice

---

## 11. Common Mistakes

* **SQL for everything**: "We use MySQL for scale"
  * Fix: Use NoSQL when you need horizontal write scaling
* **NoSQL for everything**: "NoSQL is modern"
  * Fix: Use SQL when you need ACID or complex queries
* **Ignoring consistency**: "NoSQL is faster" (without considering consistency)
  * Fix: Acknowledge eventual consistency trade-off
* **Not justifying choice**: "We use MongoDB"
  * Fix: Explain why: "We use MongoDB for flexible schema and horizontal scale"
* **Over-normalization in SQL**: "20 tables for user data"
  * Fix: Denormalize for read performance when needed
* **Under-normalization in NoSQL**: "We store everything in one document"
  * Fix: Balance between denormalization and document size limits

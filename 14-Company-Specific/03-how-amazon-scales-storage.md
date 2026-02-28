# 📦 How Amazon Scales Storage (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Amazon pioneered **eventual consistency** and **distributed key-value stores** with Dynamo. Understanding their patterns is essential for any distributed systems interview.

---

## 2. Amazon's Storage Philosophy

```
Core principles:
├── Always available (even during failures)
├── Eventual consistency (accept stale reads)
├── No single point of failure
├── Horizontal scaling
└── Simple key-value interface

"Always writable" is more important than "always consistent"
```

---

## 3. Key Architectural Patterns

### Dynamo Pattern (DynamoDB)

```
Distributed key-value with:
├── Consistent hashing for partition distribution
├── Replication to N nodes (typically 3)
├── Quorum reads/writes (R + W > N for consistency)
├── Vector clocks for conflict resolution
└── Anti-entropy for sync

Write path:
1. Hash(key) → Coordinator node
2. Coordinator replicates to N-1 nodes
3. Wait for W acknowledgments
4. Return success

Read path:
1. Hash(key) → Coordinator
2. Fetch from R nodes
3. Return value (reconcile if divergent)
```

### Consistent Hashing

```
Ring with virtual nodes:

        Node A          Node B
          ●               ●
        /   \           /   \
       /     \         /     \
    ●─────────●─────●─────────●
    Node D         Node C

Key "user:123" → Hash → Position on ring
                     → Stored on next N nodes clockwise
```

---

## 4. Architecture Diagram

```
                    ┌─────────────────────────────────────┐
                    │            Application               │
                    └───────────────┬─────────────────────┘
                                    │
                    ┌───────────────┴─────────────────┐
                    │          DynamoDB               │
                    │        (managed Dynamo)         │
                    └───────────────┬─────────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
   │  Partition 1 │          │  Partition 2 │          │  Partition 3 │
   │  (replica)   │          │  (replica)   │          │  (replica)   │
   └──────────────┘          └──────────────┘          └──────────────┘
         │                          │                          │
         ▼                          ▼                          ▼
   ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
   │  SSD Storage │          │  SSD Storage │          │  SSD Storage │
   └──────────────┘          └──────────────┘          └──────────────┘
```

---

## 5. Key Innovations

### 1. Partition Key Design

```
Good partition key:
├── High cardinality (many unique values)
├── Even distribution (no hot partitions)
├── Access pattern aligned

Examples:
├── user_id: ✓ Good, even distribution
├── date: ✗ Bad, all traffic to today's partition
├── user_id + date: ✓ Good, composite key
```

### 2. Single-Table Design

```
Traditional: Multiple tables with joins
DynamoDB: One table, multiple entity types

Table: Orders
PK              | SK                | Attributes
USER#123        | PROFILE           | name, email
USER#123        | ORDER#001         | total, status
USER#123        | ORDER#001#ITEM#1  | product, qty

Query: PK = USER#123 → Returns all user data in one query
```

### 3. Global Secondary Indexes (GSI)

```
Base table: PK = user_id
GSI: PK = email

Query patterns:
├── Get user by ID → Base table
├── Get user by email → GSI
└── Both supported with different access patterns
```

---

## 6. S3 Storage Architecture

```
S3 Design:
├── Object storage (not block/file)
├── 11 9's durability (99.999999999%)
├── Automatic replication across AZs
├── Infinite scale (no pre-provisioning)
└── Eventually consistent (read after write: immediate)

Characteristics:
├── Latency: 100-200ms first byte
├── Throughput: Limited per prefix (3500 PUT/sec)
├── Storage: Unlimited
└── Cost: $0.023/GB/month
```

---

## 7. Trade-offs Amazon Makes

| Decision | Amazon's Choice | Trade-off |
|----------|-----------------|-----------|
| **Consistency** | Eventual (Dynamo) | Stale reads possible |
| **Flexibility** | Key-value only | No complex queries |
| **Durability** | 3 replicas minimum | Storage cost |
| **Latency** | In-region only | Cross-region requires Global Tables |

---

## 8. Interview Application

> "For a shopping cart system at Amazon scale, I'd use the Dynamo pattern with DynamoDB.
>
> The partition key would be user_id for even distribution. Cart operations are simple key-value: get(user_id), put(user_id, cart).
>
> For availability, I'd use eventually consistent reads for cart display (fast, stale is OK) and strongly consistent reads for checkout (accuracy critical).
>
> The single-table design pattern would store users, carts, and orders in one table with composite sort keys. This eliminates joins and supports all access patterns.
>
> The trade-off is no complex queries. For analytics, I'd stream changes to a data warehouse rather than querying DynamoDB directly."

---

## 9. Senior-Level Phrases

- "I'd apply consistent hashing for even partition distribution."
- "Quorum reads (R + W > N) give us tunable consistency."
- "Single-table design with composite keys eliminates joins."
- "Eventually consistent reads for performance, strongly consistent for checkout."

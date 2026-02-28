# ✍️ Scaling Write-Heavy Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Write-heavy systems are **fundamentally different** from read-heavy. Interviewers test this to see if you can identify write bottlenecks and apply appropriate scaling patterns.

---

## 2. Problem Interviewers Are Testing

- Can you identify write scaling challenges (single primary, locks)?
- Do you know partitioning/sharding strategies?
- Can you use async processing to handle write bursts?

---

## 3. Key Concepts

### Write-Heavy Characteristics

```
Examples:
├── Event/log ingestion: 99% writes
├── IoT sensor data: 95% writes
├── Analytics tracking: 98% writes
├── Gaming leaderboards: 80% writes

Challenge: Single DB primary = single point of write scaling
```

### Scaling Strategies Overview

| Strategy | Write Throughput | Complexity | Consistency |
|----------|----------------:|:----------:|:-----------:|
| Vertical scaling | 2-5x | Low | Strong |
| Sharding | 10-100x | High | Per-shard |
| Write-behind cache | 10x | Medium | Eventual |
| Async queue | 100x+ | Medium | Eventual |
| Time-series DB | 100x+ | Medium | Strong |

---

## 4. Primary Patterns

### 1. Sharding (Horizontal Partitioning)

Distribute writes across multiple database primaries.

```
Write request for user_id = 12345

Shard key = user_id % num_shards
         = 12345 % 4 = 1

Route to Shard 1

┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Shard 0 │ │ Shard 1 │ │ Shard 2 │ │ Shard 3 │
│ (0,4,8) │ │(1,5,9,..)│ │ (2,6,10)│ │ (3,7,11)│
└─────────┘ └─────────┘ └─────────┘ └─────────┘
```

### 2. Async Write Queue

Buffer writes and process in batches.

```
Client → API → Message Queue → Writer Workers → Database

Benefits:
├── Absorbs write bursts
├── Retry failed writes
├── Batch inserts (100x throughput)
└── Backpressure handling
```

### 3. Write-Behind Cache

Write to fast cache, flush to database asynchronously.

```
Write request
     │
     ▼
Redis (immediate ACK)
     │
     │ async (every 1 sec)
     ▼
Database (batched writes)
```

---

## 5. Architecture Pattern

```
                     ┌───────────────┐
                     │   Ingestion   │
                     │     API       │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │    Kafka      │
                     │ (partitioned) │
                     └───────┬───────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
   ┌──────────┐        ┌──────────┐        ┌──────────┐
   │Worker 1  │        │Worker 2  │        │Worker 3  │
   │(batch 1k)│        │(batch 1k)│        │(batch 1k)│
   └────┬─────┘        └────┬─────┘        └────┬─────┘
        │                   │                   │
        ▼                   ▼                   ▼
   ┌──────────┐        ┌──────────┐        ┌──────────┐
   │ Shard 1  │        │ Shard 2  │        │ Shard 3  │
   │ (primary)│        │ (primary)│        │ (primary)│
   └──────────┘        └──────────┘        └──────────┘
```

---

## 6. Batching for Throughput

```
Single insert: 100 inserts/sec per connection
Batched insert (1000 rows): 100,000 inserts/sec per connection

Batch Sizing:
├── Too small: Loses batching benefit
├── Too large: Memory pressure, latency
└── Sweet spot: 1,000-10,000 rows per batch
```

```python
def batch_writer(queue):
    batch = []
    while True:
        event = queue.get(timeout=1.0)
        if event:
            batch.append(event)
        
        if len(batch) >= 1000 or timeout_reached:
            db.bulk_insert(batch)
            batch = []
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Durability** | Sync to disk | Async (faster) | Financial transactions |
| **Ordering** | Strict order | Out of order | Audit logs |
| **Batching** | Real-time | Batched | Low-latency requirements |
| **Sharding** | Single primary | Multi-shard | < 10K writes/sec |

---

## 8. Interview Explanation

> "For a write-heavy system like event tracking at 100K events/sec, I'd design for async batch processing.
>
> Events hit an ingestion API that immediately writes to Kafka—this is fast and durable. The API returns 202 Accepted with eventual consistency.
>
> Kafka partitions by user_id so events for the same user stay ordered. Consumer workers pull batches of 1,000 events and bulk insert into the database. Batching increases throughput by 100x versus single inserts.
>
> The database is sharded by user_id across 10 shards. Each shard handles 10K writes/sec, giving us 100K total capacity with room to grow.
>
> The trade-off is eventual consistency—events might take 1-5 seconds to appear in queries. For analytics tracking, that's perfectly acceptable."

---

## 9. Common Mistakes

- **Single-row inserts**: "Insert one at a time" → 100x slower
- **Sync writes everywhere**: "Wait for DB" → Can't handle bursts
- **Random sharding**: "Round-robin shards" → Can't do user queries
- **Ignoring write amplification**: "Index everything" → Slows writes

---

## 10. Senior-Level Phrases

- "I'd use async processing with Kafka to absorb write bursts."
- "Batching inserts gives us 100x throughput over single inserts."
- "Shard key selection is critical—I'd shard by user_id for query locality."
- "The trade-off is eventual consistency—writes are visible within seconds, not immediately."

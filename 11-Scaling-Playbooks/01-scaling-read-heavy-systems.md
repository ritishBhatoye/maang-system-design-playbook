# 📖 Scaling Read-Heavy Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Most consumer applications are **read-heavy** (100:1 to 1000:1 read/write ratio). Interviewers test this to see if you can optimize for the common case while handling writes correctly.

---

## 2. Problem Interviewers Are Testing

- Can you identify a read-heavy workload and optimize for it?
- Do you understand caching strategies and trade-offs?
- How do you scale reads without sacrificing consistency?

---

## 3. Key Concepts

### Read-Heavy Characteristics

```
Examples:
├── Social feed: 99% reads, 1% writes
├── E-commerce catalog: 99.9% reads
├── Wikipedia: 99.99% reads
└── News sites: 99.99% reads

Goal: Optimize for the 99% case (reads)
Constraint: Writes still need to work correctly
```

### Scaling Strategies Overview

| Strategy | Latency Improvement | Complexity | Cost |
|----------|--------------------:|:----------:|:----:|
| Caching | 10-100x | Low | $ |
| Read replicas | 2-5x | Medium | $$ |
| CDN | 50-100x | Low | $ |
| Denormalization | 5-20x | High | $ |
| Search index | 10-50x | Medium | $$ |

---

## 4. Caching Strategy (Most Important)

### Cache Hierarchy

```
┌──────────────────────────────────────────────────┐
│                    CLIENT                         │
└───────────────────────┬──────────────────────────┘
                        ▼
┌──────────────────────────────────────────────────┐
│           L1: CDN / Edge Cache                    │
│           (Static assets, API responses)          │
│           TTL: 1 min - 1 day                      │
└───────────────────────┬──────────────────────────┘
                        ▼
┌──────────────────────────────────────────────────┐
│           L2: Application Cache (Redis)           │
│           (Hot data, computed results)            │
│           TTL: 5 min - 1 hour                     │
└───────────────────────┬──────────────────────────┘
                        ▼
┌──────────────────────────────────────────────────┐
│           L3: Database (PostgreSQL/MySQL)         │
│           (Source of truth)                       │
└──────────────────────────────────────────────────┘
```

### Cache Hit Rates

```
Target cache hit rates for read-heavy:
├── CDN: 90-99% for static and cacheable API
├── Redis: 80-95% for hot data
└── Database query cache: 50-70%

Impact of hit rate:
├── 80% hit rate: DB load = 20% of total traffic
├── 95% hit rate: DB load = 5% of total traffic
└── 99% hit rate: DB load = 1% of total traffic
```

---

## 5. Architecture Pattern

```
                     ┌───────────────┐
                     │     CDN       │
                     │ (static+API)  │
                     └───────┬───────┘
                             │ Cache MISS
                             ▼
                     ┌───────────────┐
                     │    Gateway    │
                     └───────┬───────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Service  │   │ Service  │   │ Service  │
        │   (1)    │   │   (2)    │   │   (3)    │
        └────┬─────┘   └────┬─────┘   └────┬─────┘
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                     ┌───────────────┐
                     │  Redis Cache  │
                     └───────┬───────┘
                             │ Cache MISS
                             ▼
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │  Primary │   │ Replica  │   │ Replica  │
        │    DB    │   │   (1)    │   │   (2)    │
        └──────────┘   └──────────┘   └──────────┘
```

---

## 6. Implementation Patterns

### Read-Through Cache

```python
def get_user(user_id):
    # Try cache first
    cached = cache.get(f"user:{user_id}")
    if cached:
        return cached
    
    # Cache miss: fetch from DB
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    
    # Populate cache for next time
    cache.set(f"user:{user_id}", user, ttl=3600)
    
    return user
```

### Read Replicas Distribution

```python
def get_connection(query_type):
    if query_type == "write":
        return primary_db
    else:
        # Round-robin across replicas
        return random.choice(read_replicas)
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Consistency** | Strong (read from primary) | Eventual (read from replica) | Financial data |
| **Cache TTL** | Short (fresher data) | Long (higher hit rate) | Rapidly changing data |
| **Invalidation** | TTL-based | Event-based | Simple systems |
| **Denormalization** | Normalize (less storage) | Denormalize (faster reads) | Complex joins needed |

---

## 8. Interview Explanation

> "For a read-heavy system like a product catalog with 1000:1 read/write ratio, I'd implement a multi-layer caching strategy.
>
> At the edge, CDN caches both static assets and API responses for popular products. For a product page that doesn't change often, 5-minute TTL gives us 99% edge hit rate.
>
> For dynamic content, Redis caches query results with 1-hour TTL. For catalog reads, I'd target 90% Redis hit rate, meaning only 10% of requests hit the database.
>
> The database layer uses read replicas—three replicas across availability zones. Writes go to primary, reads are distributed across replicas.
>
> The trade-off is freshness. With 5-minute CDN cache and 1-hour Redis cache, users might see stale data after updates. For a product catalog, that's acceptable. For inventory counts, I'd either use shorter TTL or event-based invalidation."

---

## 9. Common Mistakes

- **Not identifying read/write ratio**: "We need a fast database" → Cache first!
- **Serving all reads from primary**: "Consistency is important" → Replicas for reads
- **Cache without invalidation strategy**: "Cache forever" → Stale data
- **Over-engineering writes**: "Optimize write path" → Writes are 1% of traffic

---

## 10. Senior-Level Phrases

- "With a 100:1 read/write ratio, I'd optimize for reads first—caching is the biggest lever."
- "I'd target 95% cache hit rate to reduce database load by 20x."
- "Read replicas give us horizontal read scaling at the cost of replication lag."
- "For freshness-sensitive data, I'd use event-driven invalidation rather than TTL."

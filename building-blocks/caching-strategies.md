# Caches

## 1. Problem Statement

How do we reduce latency and database load by storing frequently accessed data in fast, in-memory storage?

---

## 2. Clarifying Questions

* What's the cache hit rate target?
* What's the data access pattern? (Read-heavy?)
* What's the TTL strategy?
* What's the consistency requirement?

---

## 3. Requirements

### Functional

* Store frequently accessed data
* Invalidate stale data
* Handle cache misses
* Support high read QPS

### Non-Functional

* Sub-10ms latency
* High hit rate (80%+)
* Horizontal scalability
* Memory efficient

---

## 4. Scale Assumptions

* 1M QPS reads
* 80% cache hit rate
* 20% cache miss → DB
* Cache size: 100GB-1TB

---

## 5. High-Level Architecture

**Cache-Aside Pattern**
```
App → Check Cache → Hit: Return
                  → Miss: DB → Store in Cache → Return
```

**Write-Through Pattern**
```
App → Write to Cache → Write to DB (sync)
```

**Write-Behind Pattern**
```
App → Write to Cache → Async write to DB
```

---

## 6. Core Components

* **Cache Layer**: In-memory storage (Redis, Memcached)
* **Cache Key Strategy**: How to name keys
* **TTL (Time To Live)**: Expiration policy
* **Eviction Policy**: LRU, LFU, FIFO
* **Cache Invalidation**: How to remove stale data

---

## 7. Data Flow

**Cache-Aside (Lazy Loading)**
1. App receives read request
2. App → Check cache with key
3. If hit → Return cached data
4. If miss → Query DB
5. Store result in cache (with TTL)
6. Return data to client

**Write Flow (Cache-Aside)**
1. App receives write request
2. App → Write to DB
3. App → Invalidate cache (delete key)
4. Return success

**Cache-Through (Write-Through)**
1. App receives write request
2. App → Write to cache
3. App → Write to DB (synchronous)
4. Return success

---

## 8. Bottlenecks

* **Cache Stampede**: All requests miss, hit DB simultaneously
  * Solution: Lock on miss, or pre-warm cache
* **Hot Keys**: Single key gets all traffic
  * Solution: Shard by key, or replicate hot keys
* **Memory Limits**: Cache fills up
  * Solution: Eviction policy (LRU), or scale cache
* **Stale Data**: Cache not invalidated
  * Solution: TTL, or explicit invalidation on writes
* **Network Latency**: Cache in different region
  * Solution: Co-locate cache with app, or use local cache (L1) + distributed cache (L2)

---

## 9. Trade-offs

| Pattern | Pros | Cons | When to Use |
|---------|------|------|-------------|
| **Cache-Aside** | Simple, cache only what's needed | Cache miss penalty, possible inconsistency | Most common, read-heavy |
| **Write-Through** | Cache always consistent | Slower writes (sync DB) | When consistency critical |
| **Write-Behind** | Fast writes | Risk of data loss, complexity | High write volume, can tolerate loss |
| **Refresh-Ahead** | Proactive cache warming | Wastes resources if not accessed | Predictable access patterns |

**Eviction Policies:**
- **LRU (Least Recently Used)**: Evicts oldest accessed
- **LFU (Least Frequently Used)**: Evicts least accessed
- **FIFO (First In First Out)**: Evicts oldest inserted
- **TTL (Time To Live)**: Evicts after time expires

**Multi-Level Caching:**
- **L1 (Local)**: In-process cache (fastest, limited size)
- **L2 (Distributed)**: Redis cluster (fast, shared)
- **L3 (DB)**: Persistent storage (slowest, unlimited)

---

## 10. How to Explain This in an Interview

> "I use a cache-aside pattern with Redis because we have a read-heavy workload (10:1 read/write ratio). I set TTL to 5 minutes and use LRU eviction for memory management. The trade-off is we may serve slightly stale data, but for this use case, 5-minute staleness is acceptable and the 80% cache hit rate reduces DB load by 10x."

**Key Points:**
* Always mention the pattern (cache-aside, write-through, etc.)
* Explain TTL and eviction strategy
* Acknowledge consistency trade-off
* Quantify the benefit (hit rate, latency improvement)

---

## 11. Common Mistakes

* **Caching everything**: "We cache all data"
  * Fix: Cache hot data only, measure hit rates
* **No invalidation**: "Cache never expires"
  * Fix: Use TTL or invalidate on writes
* **Cache stampede**: "All requests miss at once"
  * Fix: Lock on miss, or use write-through for critical data
* **Wrong TTL**: "TTL is 1 hour for all data"
  * Fix: Set TTL based on data change frequency
* **Ignoring memory**: "Cache grows forever"
  * Fix: Set eviction policy, monitor memory usage
* **No fallback**: "If cache fails, system fails"
  * Fix: Graceful degradation to DB on cache failure

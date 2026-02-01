# 🏃‍♂️ Cache Stampede (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Cache stampede is one of the most common **production incidents** at scale. Interviewers love this topic because it tests whether you understand cache invalidation beyond the basics.

---

## 2. Problem Interviewers Are Testing

- Do you understand what happens when a popular cache key expires?
- Can you design systems that handle cache misses gracefully?
- Do you know multiple mitigation strategies?

---

## 3. Key Concepts

### What is Cache Stampede?

When a highly-accessed cache key expires, **hundreds or thousands of requests** simultaneously see a cache miss and all try to regenerate the same data from the database. This can overwhelm the database and cause cascading failures.

```
NORMAL OPERATION:
100 requests/sec → Cache HIT → Return cached data

STAMPEDE (cache expires):
100 requests arrive → Cache MISS (all 100)
                    → 100 DB queries for same data
                    → DB overwhelmed
                    → Latency spikes / failures
```

### The Math

```
Cache key expires at T=0
Requests arriving: 1000/sec
Cache regeneration time: 200ms
Requests hitting DB = 1000 × 0.2 = 200 duplicate queries
```

---

## 4. Mitigation Strategies

### 1. Mutex/Lock Pattern

Only one request regenerates the cache; others wait.

```
┌─────────────────────────────────────────┐
│  Request 1: acquire_lock(key)           │
│            → Lock acquired              │
│            → Query DB, write cache      │
│            → Release lock               │
│                                         │
│  Request 2-100: acquire_lock(key)       │
│            → Lock held, wait...         │
│            → Lock released              │
│            → Read from cache            │
└─────────────────────────────────────────┘
```

### 2. Early Expiration (Probabilistic)

Randomly refresh cache before actual expiration.

```python
def get_cached_value(key):
    value, expiry = cache.get_with_expiry(key)
    time_to_expiry = expiry - now()
    
    # Probabilistic early refresh
    if random() < 1 / time_to_expiry:
        refresh_cache_async(key)
    
    return value
```

### 3. Stale-While-Revalidate

Serve stale data while refreshing in background.

```
Request arrives → Check cache
├── Fresh? → Return
├── Stale but within grace period?
│   → Return stale data
│   → Trigger async refresh
└── Expired beyond grace? → Wait for refresh
```

### 4. External Refresh (Never Expire)

Cache never expires; background job refreshes periodically.

```
Cache: key → value (no TTL, or very long TTL)
Background worker: every 5 min, refresh all hot keys
```

---

## 5. Trade-offs

| Strategy | Complexity | Latency Impact | Consistency |
|----------|------------|----------------|-------------|
| **Mutex** | Medium | Waiting requests blocked | Strong |
| **Early expiration** | Low | None (probabilistic) | Eventually |
| **Stale-while-revalidate** | Medium | None (serve stale) | Weak |
| **External refresh** | High | None | Depends on refresh interval |

---

## 6. Interview Explanation

> "Cache stampede happens when a popular key expires and many requests simultaneously try to regenerate it. For a high-traffic system, I'd implement a mutex pattern: the first request acquires a lock, queries the database, and populates the cache. Other requests wait briefly, then read from cache.
>
> For even better performance, I'd use stale-while-revalidate: serve slightly stale data (within a grace period) while refreshing asynchronously. The trade-off is users briefly see stale data, but latency is unaffected."

---

## 7. Common Mistakes

- **No stampede protection**: "Cache has TTL" → But what happens on expiry?
- **Blocking forever**: "Wait for lock" → What if holder crashes?
- **Same TTL everywhere**: All keys expire together → Mass stampede

---

## 8. Senior-Level Phrases

- "I'd add TTL jitter (±10%) to prevent synchronized expiration."
- "Stale-while-revalidate gives us the best latency profile during revalidation."
- "For critical keys, I'd use external refresh so the cache never truly expires."

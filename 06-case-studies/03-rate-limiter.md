# 🚦 Rate Limiter

## 1. Problem Statement

Design a rate limiting system that prevents abuse by limiting the number of requests a user or IP can make within a time window.

---

## 2. Clarifying Questions (Ask First)

* What's the limiting scope? (Per user, per IP, per API key, global)
* What's the rate limit? (100 req/min, 1000 req/hour)
* What's the time window? (Sliding window vs fixed window)
* What happens when limit exceeded? (429 error, queue, throttle)
* Distributed or single server? (Need consistency across servers)

---

## 3. Requirements

### Functional

* Limit requests per user/IP/API key
* Support multiple rate limit rules (different limits per endpoint)
* Return clear error when limit exceeded
* Track usage accurately

### Non-Functional

* Low latency (<1ms overhead)
* High throughput (millions of requests/sec)
* Distributed (works across multiple servers)
* Memory efficient

---

## 4. Scale Assumptions

* 1M requests/second
* 100M unique users/IPs
* Multiple rate limit rules (e.g., 100/min, 1000/hour per user)
* Distributed across 100 servers
* Need sub-millisecond latency

---

## 5. High-Level Architecture

```
Request → Rate Limiter Middleware → Check Limit
                                    │
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
            In-Memory Cache    Redis (Distributed)  Database (Fallback)
            (Local)            (Shared State)
```

---

## 6. Core Components

* **Rate Limiter Service**: Checks and updates counters
* **Storage**: Redis (distributed) or in-memory (local)
* **Algorithm**: Token bucket, sliding window, fixed window
* **Middleware**: Intercepts requests, applies limits
* **Configuration**: Rate limit rules (endpoint, user type, limits)

---

## 7. Data Flow

**Token Bucket Algorithm**
1. Request arrives → Extract identifier (user_id, IP)
2. Rate Limiter → Check bucket for identifier
3. If tokens available → Consume token, allow request
4. If no tokens → Reject with 429 (Too Many Requests)
5. Background → Refill tokens at fixed rate (e.g., 100 tokens/minute)

**Sliding Window Log Algorithm**
1. Request arrives → Extract identifier
2. Rate Limiter → Get all requests in current window (e.g., last 60 seconds)
3. Count requests → If < limit, allow and log timestamp
4. If >= limit → Reject with 429
5. Cleanup → Remove old timestamps outside window

**Fixed Window Algorithm**
1. Request arrives → Extract identifier
2. Rate Limiter → Get counter for current window (e.g., minute: 2024-01-01 10:30)
3. Increment counter → If < limit, allow
4. If >= limit → Reject with 429
5. Window expires → Counter resets (e.g., next minute)

**Distributed Rate Limiting (Redis)**
1. Request → Server A receives request
2. Server A → Check Redis: GET user_id:window
3. Redis → Return current count
4. Server A → If < limit, INCR user_id:window, set TTL
5. Server A → Allow or reject request
6. Other servers → Same Redis, see updated count (consistent)

---

## 8. Bottlenecks

* **Redis Hot Keys**: Single user/IP making many requests
  * Solution: Shard Redis by user_id hash, or use local cache + Redis
* **Memory Usage**: Storing timestamps for sliding window
  * Solution: Use approximate algorithms (count-min sketch), or fixed window
* **Network Latency**: Redis round-trip adds latency
  * Solution: Local cache (L1) + Redis (L2), or in-memory for single server
* **Race Conditions**: Multiple servers incrementing simultaneously
  * Solution: Redis atomic operations (INCR), or distributed locks
* **Configuration Updates**: Changing rate limits dynamically
  * Solution: Store config in database, cache in memory, refresh periodically

---

## 9. Trade-offs

| Algorithm | Pros | Cons | When to Use |
|-----------|------|------|-------------|
| **Fixed Window** | Simple, memory efficient | Burst at window boundary | Simple use cases |
| **Sliding Window Log** | Accurate, smooth limiting | Memory intensive (stores timestamps) | Need precise limiting |
| **Sliding Window Counter** | Memory efficient, approximate | Less accurate (counts overlap) | High throughput, acceptable approximation |
| **Token Bucket** | Allows bursts, smooth refill | More complex | Need burst allowance |

**Storage Trade-offs:**
- **In-Memory (Local)**: Fastest, but not distributed
- **Redis (Distributed)**: Consistent across servers, but network latency
- **Hybrid**: Local cache + Redis for best of both

**Key Trade-off:**
> "I use Redis for distributed rate limiting because we have 100 servers and need consistent limits across all of them. The trade-off is ~1ms network latency, but I mitigate this with a local cache (L1) that caches recent checks. For high-throughput scenarios, I use a sliding window counter algorithm which is memory-efficient and accurate enough."

---

## 10. How to Explain This in an Interview

> "I'll design a distributed rate limiter using Redis for shared state and a sliding window counter algorithm for memory efficiency.
>
> For architecture, I use a middleware that intercepts requests and checks rate limits before processing. I store counters in Redis keyed by identifier (user_id or IP) and time window. For example, 'user_123:2024-01-01-10:30' for fixed window, or 'user_123:sliding' for sliding window.
>
> For the algorithm, I use a sliding window counter which maintains counts for multiple sub-windows (e.g., 6 windows of 10 seconds each for a 60-second window). This is memory-efficient and accurate enough. When a request arrives, I sum the counts in the current window and compare to the limit.
>
> For distributed consistency, I use Redis with atomic operations (INCR) to handle concurrent requests from multiple servers. I also use a local cache (L1) to reduce Redis calls for frequently accessed keys, with Redis as the source of truth (L2).
>
> For high-traffic scenarios, I shard Redis by user_id hash to avoid hot keys. If a single user is making many requests, the sharding distributes the load.
>
> The key trade-off is Redis network latency (~1ms) for distributed consistency, but the local cache mitigates this for hot keys."

**Key Phrases:**
* "Redis for distributed state, local cache for performance"
* "Sliding window counter for memory efficiency"
* "Atomic operations (INCR) for consistency"
* "Shard Redis to avoid hot keys"

---

## 11. Common Mistakes

* **Fixed window only**: "We use fixed 1-minute windows"
  * Fix: Mention sliding window for smoother limiting, or explain why fixed is sufficient
* **No distributed support**: "Each server tracks independently"
  * Fix: Use Redis or shared storage for consistency
* **Storing all timestamps**: "We log every request timestamp"
  * Fix: Use counter-based algorithms for memory efficiency
* **Race conditions**: "Multiple servers increment simultaneously"
  * Fix: Use atomic operations (Redis INCR) or distributed locks
* **No caching**: "Every request hits Redis"
  * Fix: Use local cache for hot keys to reduce latency
* **Hard-coded limits**: "Rate limit is 100/min always"
  * Fix: Store limits in database/config, make them dynamic
* **Not handling Redis failure**: "If Redis is down, system fails"
  * Fix: Fallback to local rate limiting, or fail open/closed based on policy

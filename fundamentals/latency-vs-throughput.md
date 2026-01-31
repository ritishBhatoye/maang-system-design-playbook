# Latency vs Throughput

## 1. Problem Statement

How do we optimize for speed (latency) vs volume (throughput), and when do we choose what?

---

## 2. Clarifying Questions

* What's the SLA requirement? (p50, p95, p99)
* Is this user-facing or batch processing?
* What's the acceptable trade-off?

---

## 3. Requirements

### Functional

* Meet latency SLAs
* Handle peak throughput
* Graceful degradation

### Non-Functional

* p95 latency < 200ms (user-facing)
* Throughput: 10K QPS minimum
* Predictable performance

---

## 4. Scale Assumptions

* User-facing: 50K QPS, p95 < 200ms
* Batch jobs: 1M records/hour, latency not critical
* Mixed workload: Both requirements

---

## 5. High-Level Architecture

**Low Latency System**
```
Client → CDN → Cache → Fast DB (in-memory)
```

**High Throughput System**
```
Client → Queue → Workers (parallel) → Batch DB writes
```

---

## 6. Core Components

* **CDN**: Reduces latency for static content
* **Caching**: Sub-millisecond reads
* **Connection Pooling**: Reuse connections
* **Batching**: Group operations for throughput
* **Async Processing**: Don't block on slow operations

---

## 7. Data Flow

**Low Latency Flow (Read)**
1. Request → Check L1 cache (in-memory)
2. Miss → Check L2 cache (Redis)
3. Miss → DB (with connection pool)
4. Return + warm cache

**High Throughput Flow (Write)**
1. Request → Queue (async)
2. Worker batch processes (1000 at a time)
3. Bulk insert to DB
4. Acknowledge completion

---

## 8. Bottlenecks

* **Network Round Trips**: Each hop adds latency
  * Solution: Co-locate services, use CDN
* **Database Queries**: Slow queries kill both
  * Solution: Indexes, query optimization, caching
* **Synchronous Operations**: Blocking calls
  * Solution: Async, batching, parallelization
* **Serial Processing**: One at a time
  * Solution: Parallel workers, pipelining

---

## 9. Trade-offs

| Optimization | Latency Impact | Throughput Impact | Cost |
|--------------|----------------|-------------------|------|
| **Caching** | ✅ Massive (ms → μs) | ✅ High (offloads DB) | Medium |
| **CDN** | ✅ Huge (global → edge) | ✅ High | Medium |
| **Batching** | ❌ Increases (wait for batch) | ✅ Massive | Low |
| **Async Processing** | ✅ Better (non-blocking) | ✅ Better | Low |
| **Read Replicas** | ✅ Better (closer to users) | ✅ Massive | Medium |
| **Connection Pooling** | ✅ Better (reuse) | ✅ Better | Low |

**Key Insight**: You often optimize for one at the expense of the other.

---

## 10. How to Explain This in an Interview

> "For user-facing reads, I optimize for latency using multi-layer caching (L1 in-memory, L2 Redis) because sub-100ms response time is critical. For batch writes, I optimize for throughput using batching and async queues, accepting higher latency to process millions of records efficiently."

**Decision Framework:**
* **User-facing**: Latency > Throughput
* **Batch/Background**: Throughput > Latency
* **Real-time**: Both matter → parallelize + cache

---

## 11. Common Mistakes

* **Optimizing the wrong metric**: "We batch user requests"
  * Fix: User requests need low latency, not batching
* **Ignoring p95/p99**: "Average is 50ms"
  * Fix: p95/p99 reveal real user experience
* **Over-caching**: "Cache everything"
  * Fix: Cache hot data, measure hit rates
* **Not measuring**: "We think it's fast"
  * Fix: Instrument p50/p95/p99, track trends

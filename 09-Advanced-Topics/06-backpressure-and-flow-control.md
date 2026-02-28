# 🌊 Backpressure and Flow Control (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Backpressure separates engineers who build **reliable systems** from those who build **ticking time bombs**. Interviewers test this to see if you understand how to prevent cascading failures when services are overwhelmed.

---

## 2. Problem Interviewers Are Testing

- **Overload handling**: What happens when traffic 10x's suddenly?
- **Graceful degradation**: How do we fail gracefully instead of crashing?
- **System stability**: How do we prevent cascading failures?

---

## 3. Key Concepts

### What is Backpressure?

Backpressure is the mechanism by which a system signals to upstream producers to **slow down** when it can't keep up. Without backpressure, queues overflow, memory exhausts, and systems crash.

### Flow Control Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Drop** | Discard excess requests | Metrics, logs (can lose some) |
| **Throttle** | Slow down producers | API rate limiting |
| **Buffer** | Queue temporarily | Bursty traffic |
| **Reject** | Return error to caller | Preserve system health |
| **Load Shed** | Drop low-priority work | Protect critical paths |

### Where Backpressure Occurs

- Thread pools exhausted
- Database connections maxed
- Message queue at capacity
- Network buffers full
- Memory/heap exhausted

---

## 4. Typical Architecture

```
WITHOUT BACKPRESSURE:
    Producer (10K/s) → Queue → Consumer (1K/s)
                         │
                   Queue grows unbounded
                         │
                   OOM → System crash

WITH BACKPRESSURE:
    Producer (10K/s) → Queue (bounded) → Consumer (1K/s)
                         │
               Queue full → 429 to producer
                         │
               Producer slows down or drops
```

### Implementation Pattern

```
    ┌──────────┐   bounded    ┌──────────┐
    │ Producer │─────queue───►│ Consumer │
    │          │◄───signal────│          │
    └──────────┘   (slow!)    └──────────┘
```

---

## 5. Scaling Strategy

### Token Bucket for Rate Limiting

```
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second

Request arrives:
├── Token available → Process request, consume token
└── No token → Reject with 429, suggest retry after X
```

### Adaptive Load Shedding

```python
def should_process(request, system_load):
    if system_load < 70%:
        return True  # Process all
    elif system_load < 90%:
        return request.priority >= HIGH  # Only high priority
    else:
        return request.is_critical  # Only critical
```

### Circuit Breaker Integration

```
State Machine:
CLOSED → (errors > threshold) → OPEN → (timeout) → HALF-OPEN
   ↑                                                    │
   └───────────────── (success) ───────────────────────┘
```

---

## 6. Failure Modes

| Failure | Impact | Mitigation |
|---------|--------|------------|
| **Unbounded queues** | OOM crash | Bounded queues with rejection |
| **Slow consumers** | Producer backup | Async processing, scaling |
| **Retry storms** | 10x load on recovery | Exponential backoff with jitter |
| **Cascading failures** | Full system outage | Circuit breakers, bulkheads |

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Overflow behavior** | Drop oldest | Drop newest | Logs, metrics |
| **Signal mechanism** | Explicit (429) | Implicit (latency) | Client can react |
| **Queue size** | Large (more buffering) | Small (faster feedback) | Predictable load |

---

## 8. Interview Explanation

> "For a high-throughput system, I'd implement backpressure at multiple layers. At the API gateway, token bucket rate limiting prevents individual clients from overwhelming us. Each service has bounded thread pools that return 503 when exhausted.
>
> For async processing, I use bounded queues (e.g., Kafka with consumer lag alerting). If consumers fall behind, we scale up consumers or shed low-priority messages.
>
> The trade-off is between throughput and stability—we sacrifice some requests during overload to keep the system responsive for the majority."

---

## 9. Common Mistakes

- **Unbounded queues**: "It'll drain eventually" → OOM
- **No client backoff**: "Just retry immediately" → Amplifies problems
- **Ignoring slow dependencies**: "DB will catch up" → Thread pool exhaustion
- **Load shedding without priority**: "Drop randomly" → Critical requests fail

---

## 10. Senior-Level Phrases

- "I'd implement bounded queues with clear rejection semantics."
- "Client requests get exponential backoff with jitter to prevent thundering herd."
- "We'd use adaptive load shedding based on system CPU and queue depth."
- "Circuit breakers prevent cascading failures to downstream services."

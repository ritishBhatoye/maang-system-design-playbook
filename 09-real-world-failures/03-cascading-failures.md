# 🎯 Cascading Failures (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Cascading failures are the **root cause of major outages** at every major tech company. Interviewers test this to see if you can design resilient systems that fail gracefully.

---

## 2. Problem Interviewers Are Testing

- How does one service failure propagate through the system?
- What mechanisms prevent one failure from taking down everything?
- Can you design for partial failure?

---

## 3. Key Concepts

### What is a Cascading Failure?

A failure in one component that triggers failures in dependent components, which in turn trigger more failures—a domino effect.

```
ONE SERVICE SLOWS DOWN:
Service A (slow) → Service B waits → B's threads exhausted
                                   → Service C calls B → C waits
                                   → Thread pools everywhere exhausted
                                   → Full system outage

A small delay in one service → Complete system failure
```

### Common Triggers

- **Resource exhaustion**: Thread pools, connections, memory
- **Timeout propagation**: Slow service → timeout → retry → more load
- **Circuit without breaker**: Keep calling failing service
- **Shared dependencies**: Shared DB/cache fails → everything fails

---

## 4. Mitigation Strategies

### 1. Circuit Breakers

Stop calling a failing service, fail fast instead.

```
States: CLOSED → OPEN → HALF-OPEN

CLOSED: Normal operation
        │ failure_count > threshold
        ▼
OPEN:   Fail immediately (don't call service)
        │ after timeout
        ▼
HALF-OPEN: Try one request
        │ if success → CLOSED
        │ if failure → OPEN
```

### 2. Bulkheads

Isolate components so failure in one doesn't exhaust shared resources.

```
WITHOUT BULKHEAD:
┌────────────────────────────────┐
│     Shared Thread Pool (100)   │
│  Service A + B + C all share   │
└────────────────────────────────┘
A is slow → uses all 100 → B and C starved

WITH BULKHEAD:
┌────────┐ ┌────────┐ ┌────────┐
│   A    │ │   B    │ │   C    │
│  (33)  │ │  (33)  │ │  (33)  │
└────────┘ └────────┘ └────────┘
A is slow → uses its 33 → B and C unaffected
```

### 3. Timeouts (Aggressive)

Bound the damage from slow dependencies.

```
Rule of Thumb:
├── Database: 500ms timeout
├── Cache: 50ms timeout  
├── Inter-service: 1-2s timeout
└── External API: 3-5s timeout

Timeout = Expected P99 × 2 (with buffer)
```

### 4. Load Shedding

Intentionally drop requests when overloaded.

```python
def handle_request(request):
    if cpu_usage > 80% or queue_depth > 1000:
        if request.priority < HIGH:
            return 503  # Shed load
    
    return process(request)
```

---

## 5. Architecture Pattern

```
┌─────────────────────────────────────────────────┐
│                   API Gateway                    │
│   (rate limit, circuit breaker, timeout: 5s)    │
└─────────────────────┬───────────────────────────┘
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐
    │Service A│  │Service B│  │Service C│
    │ bulkhead│  │ bulkhead│  │ bulkhead│
    │ timeout │  │ timeout │  │ timeout │
    │ circuit │  │ circuit │  │ circuit │
    └────┬────┘  └────┬────┘  └────┬────┘
         │            │            │
         │       ┌────┴────┐       │
         └──────►│   DB    │◄──────┘
                 │ (pooled)│
                 └─────────┘
```

---

## 6. Trade-offs

| Strategy | Pro | Con |
|----------|-----|-----|
| **Circuit breaker** | Prevents hammering | May reject good requests |
| **Bulkhead** | Isolation | Reduced total capacity |
| **Aggressive timeout** | Bounds latency | May timeout valid slow queries |
| **Load shedding** | System survives | Dropped requests |

---

## 7. Interview Explanation

> "To prevent cascading failures, I'd implement defense in depth. First, each service has dedicated thread pools (bulkheads) so a slow dependency can't starve other operations.
>
> Second, all external calls have circuit breakers. If Service B's error rate exceeds 50%, we stop calling it and fail fast with a fallback response. This prevents querying a failing service and making things worse.
>
> Third, aggressive timeouts bound how long any request can wait. P99 plus 20% buffer. If we don't get a response, we timeout and return an error rather than blocking threads indefinitely.
>
> The trade-off is sometimes we fail requests that would have succeeded. But failing fast keeps the system responsive for the majority of users."

---

## 8. Common Mistakes

- **No timeouts**: "Wait for DB" → Threads blocked forever
- **Shared thread pools**: "It's simpler" → One slow service blocks all
- **No circuit breakers**: "Keep trying" → Overloads failing service
- **Retry without backoff**: "Retry immediately" → Amplifies load

---

## 9. Senior-Level Phrases

- "Bulkheads isolate failure domains so a slow dependency only affects its consumers."
- "Circuit breakers prevent cascading load onto already-failing services."
- "I'd set timeouts based on P99 latency with 20% buffer for tail cases."
- "Load shedding protects the system at the cost of some dropped requests."

# 🔄 Retry Storms (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Retry storms turn **small problems into outages**. This topic tests whether you understand how well-intentioned retry logic can amplify failures.

---

## 2. Problem Interviewers Are Testing

- Do you understand how retries can make problems worse?
- Can you design retry policies that help rather than harm?
- Do you know when NOT to retry?

---

## 3. Key Concepts

### What is a Retry Storm?

When a service experiences a brief failure, clients retry. If many clients retry simultaneously, the retry load exceeds the original load, preventing recovery.

```
NORMAL LOAD: 1000 requests/sec → Service handles fine

BRIEF FAILURE (1 second):
├── 1000 requests fail
├── All 1000 retry immediately
├── + 1000 new requests
├── Service sees 2000/sec → Fails again
├── 2000 retry + 1000 new = 3000/sec
└── Exponential amplification → Total outage
```

### Amplification Factor

```
Without backoff:
├── Attempt 1: 1000 requests
├── Attempt 2: 1000 retries + 1000 new = 2000
├── Attempt 3: 2000 retries + 1000 new = 3000
└── Each retry adds load

With 3 retries per request:
├── Worst case: 4x load during outage
└── If synchronized: 4x spike at each retry interval
```

---

## 4. Mitigation Strategies

### 1. Exponential Backoff

Increase delay between retries to give system time to recover.

```python
def get_retry_delay(attempt):
    base = 1.0
    max_delay = 60.0
    return min(base * (2 ** attempt), max_delay)

# Delays: 1s, 2s, 4s, 8s, 16s, 32s, 60s...
```

### 2. Jitter (Critical!)

Random offset prevents synchronized retries.

```python
def get_retry_delay_with_jitter(attempt):
    delay = min(1.0 * (2 ** attempt), 60.0)
    jitter = random.uniform(0, delay)  # Full jitter
    return jitter

# Decorrelated: Each client retries at different times
```

### 3. Retry Budgets

Limit retries at the system level, not per-request.

```python
class RetryBudget:
    def __init__(self, ratio=0.2):
        self.requests = 0
        self.retries = 0
        self.ratio = ratio  # Max 20% of traffic can be retries
    
    def can_retry(self):
        return self.retries < (self.requests * self.ratio)
```

### 4. Idempotency Keys

Ensure retries don't cause duplicate side effects.

```
Request: POST /orders
Header: Idempotency-Key: abc-123

Server checks: Have I seen abc-123?
├── No → Process order, store key
└── Yes → Return previous result (don't reprocess)
```

---

## 5. When NOT to Retry

| Response | Should Retry? | Reason |
|----------|---------------|--------|
| **400 Bad Request** | ❌ No | Request is invalid |
| **401 Unauthorized** | ❌ No | Credentials are wrong |
| **403 Forbidden** | ❌ No | Permission issue |
| **404 Not Found** | ❌ No | Resource doesn't exist |
| **429 Too Many Requests** | ✅ Yes | Rate limited, retry later |
| **500 Internal Error** | ✅ Maybe | Transient server issue |
| **502 Bad Gateway** | ✅ Yes | Upstream temporarily down |
| **503 Service Unavailable** | ✅ Yes | Server overloaded |
| **504 Gateway Timeout** | ✅ Yes | Slow upstream |

---

## 6. Trade-offs

| Strategy | Pro | Con |
|----------|-----|-----|
| **More retries** | Better success rate | More load on failures |
| **Longer backoff** | Less amplification | Slower user experience |
| **Retry budget** | Prevents storms | Some requests never retry |
| **No retries** | Simplest | Single failure = user failure |

---

## 7. Interview Explanation

> "To prevent retry storms, I'd implement a multi-layered approach. First, exponential backoff with jitter: each retry waits 2^n seconds plus random jitter up to that delay. This spreads retries over time instead of spiking.
>
> Second, I'd use retry budgets: at system level, only 10-20% of traffic can be retries. If we hit that limit, new retries are dropped. This caps the amplification factor.
>
> Third, I'd respect response codes: 4xx errors don't retry (client's fault), 5xx errors retry with backoff. 429s respect the Retry-After header.
>
> For idempotency, any write request includes an idempotency key so retries don't cause duplicate orders or payments."

---

## 8. Common Mistakes

- **Immediate retry**: "Retry right away" → Amplifies load
- **Fixed intervals**: "Retry every 1 second" → Synchronized waves
- **Retry on 4xx**: "Retry everything" → Wastes resources
- **Unlimited retries**: "Keep trying forever" → Never gives up load
- **No idempotency**: "Just retry POST" → Duplicate charges

---

## 9. Senior-Level Phrases

- "Exponential backoff with full jitter decorrelates retry timing across clients."
- "I'd use a system-wide retry budget of 10% to cap amplification during failures."
- "Only transient errors (5xx, timeout) should retry—4xx indicates client fault."
- "Idempotency keys ensure retry safety for non-GET requests."

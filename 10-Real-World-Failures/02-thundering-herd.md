# ⚡ Thundering Herd (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Thundering herd is a classic distributed systems problem. Interviewers test this to see if you understand **retry behavior** and can prevent clients from amplifying failures.

---

## 2. Problem Interviewers Are Testing

- What happens when a service comes back online after an outage?
- How do you prevent clients from overwhelming a recovering system?
- Do you understand jitter and backoff?

---

## 3. Key Concepts

### What is Thundering Herd?

When a single event triggers many processes to compete for the same resource simultaneously. Common scenarios:

- Service restart → all clients reconnect at once
- Cache expiration → all requests hit database
- Scheduled job → all workers wake up together
- Blocking operation releases → all waiters proceed

```
SERVICE OUTAGE:
1000 clients waiting for service

SERVICE RECOVERS:
1000 clients ALL reconnect at T=0
→ Service immediately overwhelmed
→ Fails again → thundering herd loop
```

---

## 4. Mitigation Strategies

### 1. Exponential Backoff with Jitter

```python
def retry_with_backoff(attempt):
    base_delay = 1  # second
    max_delay = 60  # seconds
    
    # Exponential: 1, 2, 4, 8, 16...
    delay = min(base_delay * (2 ** attempt), max_delay)
    
    # Add jitter: ±50%
    jitter = delay * random.uniform(-0.5, 0.5)
    
    sleep(delay + jitter)
```

### 2. Admission Control

Limit how many clients can reconnect simultaneously.

```
Connection queue with capacity N
├── Client tries to connect
├── Queue full? → Client backs off
└── Queue has space? → Process connection
```

### 3. Time-Scattered Reconnection

Spread reconnections over a window.

```python
def scheduled_reconnect(client_id, window_seconds=60):
    # Each client gets deterministic offset
    offset = hash(client_id) % window_seconds
    sleep(offset)
    reconnect()
```

### 4. Circuit Breaker at Client

Prevent retry storms by detecting upstream failure.

```
Client states: CLOSED → OPEN → HALF_OPEN
├── CLOSED: Send requests normally
├── OPEN (after failures): Don't send, fail fast
└── HALF_OPEN: Try one request, decide
```

---

## 5. Real-World Examples

| Scenario | Without Mitigation | With Mitigation |
|----------|-------------------|-----------------|
| **Service restart** | 100K connections/sec | 100K connections over 60 sec |
| **Cron at midnight** | All jobs at 00:00:00 | Jobs spread 00:00:00-00:00:59 |
| **Cache expire @10min** | 10K DB queries | 1 query, 9999 wait |

---

## 6. Trade-offs

| Strategy | Latency Impact | Complexity | Effectiveness |
|----------|---------------|------------|---------------|
| **Jitter only** | Minimal | Low | Medium |
| **Exponential backoff** | High for failures | Low | High |
| **Admission control** | Queue wait | Medium | High |
| **Circuit breaker** | Fast failure | Medium | High |

---

## 7. Interview Explanation

> "Thundering herd happens when many clients simultaneously retry after a failure. To prevent this, I'd implement exponential backoff with jitter at the client: each retry waits 2^attempt seconds plus a random offset between 0-50%.
>
> For server-side protection, I'd add admission control—a bounded connection pool that rejects new connections when at capacity. Clients seeing rejection back off, naturally throttling the recovery.
>
> The trade-off is slower recovery for individual clients, but the system as a whole stays stable."

---

## 8. Common Mistakes

- **Immediate retry**: "Retry every 100ms" → Amplifies problem
- **Synchronized schedules**: "Cron at exactly midnight" → All compete
- **Fixed reconnect timing**: "Wait 5 seconds, retry" → All retry together
- **No jitter**: "Wait 2^n seconds exactly" → Waves of retries

---

## 9. Senior-Level Phrases

- "I'd add jitter to decorrelate retry attempts across clients."
- "Exponential backoff prevents a thundering herd during cascading failures."
- "For cron jobs, I'd add random offset based on job ID to spread load."
- "Client-side circuit breakers prevent retry storms from amplifying outages."

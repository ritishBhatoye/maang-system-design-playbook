# 🔌 Partial Outages (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Partial outages are **harder to handle than full outages**. This tests whether you can design systems that degrade gracefully rather than failing completely.

---

## 2. Problem Interviewers Are Testing

- How do you handle a system that's partially working?
- Can you design for graceful degradation?
- Do you understand the complexity of partial failures?

---

## 3. Key Concepts

### What is a Partial Outage?

When some parts of the system fail while others continue working. Much harder to detect and handle than complete failures.

```
COMPLETE OUTAGE:
├── Detection: Easy (everything is down)
├── Response: Failover or wait
└── User experience: Clear error

PARTIAL OUTAGE:
├── Detection: Hard (some requests succeed)
├── Response: Complex (which parts to bypass?)
└── User experience: Confusing (random failures)
```

### Types of Partial Failures

| Type | Example | Detection Difficulty |
|------|---------|---------------------|
| **Geographic** | EU region down, US up | Medium |
| **Functional** | Search works, checkout broken | Medium |
| **Data** | 10% of queries fail | Hard |
| **Latent** | Writes work, reads delayed | Very Hard |
| **Intermittent** | Random 5% failure rate | Very Hard |

---

## 4. Detection Strategies

### 1. Multi-Dimensional Health Checks

```
Traditional: Is service responding? ✓/✗

Better:
├── Liveness: Is process running?
├── Readiness: Can it serve traffic?
├── Dependency health: Are backends OK?
├── Functional: Can it complete key operations?
└── Performance: Is latency acceptable?
```

### 2. Error Rate Thresholds

```python
def is_healthy(service):
    metrics = get_metrics(service, window="5m")
    
    checks = [
        metrics.error_rate < 0.05,      # < 5% errors
        metrics.p99_latency < 500,       # < 500ms
        metrics.success_count > 100,     # Enough traffic
    ]
    
    return all(checks)
```

### 3. Canary Requests

Active health probes that exercise the full flow.

```
Every 10 seconds:
├── Create test order
├── Verify order in database
├── Verify order in search
└── If any step fails → Mark unhealthy
```

---

## 5. Handling Strategies

### 1. Graceful Degradation

Disable non-critical features to preserve core functionality.

```
NORMAL MODE:
Homepage = recommendations + search + trending + ads

DEGRADED MODE (recommendations service down):
Homepage = search + trending + ads + "Recs unavailable"

SURVIVAL MODE (DB at 90% capacity):
Homepage = cached content only, writes queued
```

### 2. Feature Flags for Degradation

```python
def get_recommendations(user_id):
    if not feature_flags.is_enabled("recommendations"):
        return EMPTY_RECOMMENDATIONS  # Graceful degradation
    
    try:
        return recs_service.get(user_id)
    except Timeout:
        feature_flags.disable("recommendations", ttl=300)
        return EMPTY_RECOMMENDATIONS
```

### 3. Fallback Chains

```
Primary: Real-time recommendations API
    │
    ├── Timeout/Error
    ▼
Fallback 1: Cached recommendations (last hour)
    │
    ├── Cache miss
    ▼
Fallback 2: Popular items list
    │
    ├── Error
    ▼
Fallback 3: Static default recommendations
```

---

## 6. Trade-offs

| Strategy | Pro | Con |
|----------|-----|-----|
| **Graceful degradation** | Core functionality preserved | Reduced features |
| **Full failover** | All-or-nothing clarity | May lose working parts |
| **Retry/workaround** | Automatic recovery | Delays, complexity |
| **Fail fast** | Quick error response | No chance for success |

---

## 7. Interview Explanation

> "Partial outages are harder than full outages because they're harder to detect and require nuanced responses. I'd implement multi-layer detection: health checks for liveness, readiness, and actual functional tests that exercise the full path.
>
> For handling, I'd use graceful degradation controlled by feature flags. If the recommendations service has >5% error rate, we automatically disable it and show a static fallback. Users get a slightly worse experience but the page still loads.
>
> The key is classifying features as critical vs non-critical. Checkout must work—if payments are partially failing, we alert and may stop taking orders. But if reviews aren't loading, we show 'Reviews temporarily unavailable' and continue.
>
> The trade-off is complexity: every dependency needs a fallback path and we need to test degraded states regularly."

---

## 8. Common Mistakes

- **Binary health checks**: "Service is up or down" → Misses partial failures
- **No fallback strategy**: "Feature just fails" → Bad user experience
- **All-or-nothing failover**: "Switch to backup" → Loses working parts
- **Missing degradation testing**: "We'll figure it out" → Chaos during real outage

---

## 9. Senior-Level Phrases

- "I'd classify dependencies as critical vs non-critical for degradation planning."
- "Functional health checks exercise the full path, not just TCP ping."
- "Feature flags give us runtime control to disable degraded components."
- "We'd regularly test degraded states in production using chaos engineering."

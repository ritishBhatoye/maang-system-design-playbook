# 🚦 Rate Limiting for AI (2026 MAANG Edition)

## 1. Why This Matters in Interviews

AI rate limiting is **fundamentally different** from traditional API rate limiting. Interviewers test this to see if you understand token-based limits, cost protection, and fair usage.

---

## 2. Problem Interviewers Are Testing

- Do you understand token-based rate limiting?
- Can you protect against cost attacks?
- How do you handle bursty AI workloads?
- Can you implement fair queuing for expensive operations?

---

## 3. Key Concepts

### Why AI Rate Limiting is Different

```
Traditional API:
├── Rate limit: 100 requests/minute
├── Cost per request: ~$0.0001
├── All requests roughly equal

AI/LLM API:
├── Rate limit: 100K tokens/minute
├── Cost per request: $0.001 - $1.00 (1000x variance!)
├── Request "size" varies massively
└── Single abusive request can cost $50+
```

### What to Limit

| Dimension | Why | Limit Example |
|-----------|-----|---------------|
| **Requests/min** | Prevent abuse | 60/min per user |
| **Tokens/min** | Control cost | 100K tokens/min |
| **Concurrent** | Manage capacity | 5 concurrent/user |
| **Daily spend** | Cost protection | $50/day per user |
| **Context size** | Memory protection | 100K tokens max |

---

## 4. Rate Limiting Strategies

### 1. Token Bucket (Bursty-Friendly)

```
Bucket: 100K tokens
Refill: 10K tokens/minute

Request needs 5K tokens:
├── Tokens available? → Consume, proceed
├── Not enough? → Wait or reject (429)

Benefits:
├── Allows bursts up to bucket size
├── Smoothly limits sustained rate
└── Fair for varied request sizes
```

### 2. Sliding Window

```
Track usage in rolling window:

[──────── 1 minute window ────────]
   │5K│ │10K│ │3K│ │8K│    │?│
                              ↑
                        New request: 15K tokens
                        Current usage: 26K
                        Limit: 100K
                        → Allowed (26K + 15K < 100K)
```

### 3. Cost-Based Limiting

```python
def check_rate_limit(user_id, request_tokens):
    # Check multiple dimensions
    checks = [
        rate_limiter.check("requests", user_id, 1),
        rate_limiter.check("tokens", user_id, request_tokens),
        rate_limiter.check("daily_spend", user_id, estimate_cost(request_tokens)),
    ]
    
    if not all(checks):
        return 429, "Rate limit exceeded"
    
    return 200, "OK"
```

---

## 5. Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                      User Request                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  Pre-Request Checks                          │
│  ┌─────────────┐ ┌─────────────┐ ┌────────────────────────┐ │
│  │Request/min  │ │ Token/min   │ │ Daily spend budget     │ │
│  │(60 limit)   │ │(100K limit) │ │($50 limit)             │ │
│  └─────────────┘ └─────────────┘ └────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────┘
                            │ All pass?
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Concurrency Control                        │
│              (Max 5 concurrent LLM calls per user)           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Priority Queue                             │
│              (Paid users > Free users)                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      LLM Provider                            │
│              (Also has its own rate limits!)                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Handling Provider Limits

```
Your app has limits, but so does OpenAI:

OpenAI tier limits (example):
├── Tier 1: 60K TPM, $100/month
├── Tier 2: 80K TPM, $500/month
├── Tier 4: 300K TPM, $5000/month
└── Tier 5: 10M TPM

Strategy:
├── Pool across multiple API keys
├── Distribute load across providers
├── Queue requests during burst
└── Prefer cached responses
```

### Multi-Provider Fallback

```python
providers = ["openai", "anthropic", "azure_openai"]

def call_llm(prompt):
    for provider in providers:
        if rate_limiter.can_use(provider):
            try:
                return provider.generate(prompt)
            except RateLimitError:
                rate_limiter.mark_limited(provider, duration=60)
                continue
    
    # All providers limited
    return queue_for_later(prompt)
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Rejection** | 429 error | Queue request | User can retry |
| **Granularity** | Per-user | Per-org | B2C application |
| **Enforcement** | Hard limit | Soft (degrade) | Cost protection |
| **Measurement** | Input tokens only | Input + output | Accurate cost |

---

## 8. Interview Explanation

> "For an LLM-powered application, I'd implement multi-dimensional rate limiting.
>
> First, request rate: 60 requests per minute per user prevents script abuse. Second, token rate: 100K tokens per minute limits throughput. Third, daily cost budget: $10/day per free user, $50/day per paid user.
>
> Before each request, I estimate token count from prompt length and check all limits. If any exceeds, return 429 with Retry-After header.
>
> For handling provider limits, I'd use a token bucket shared across all users, sized to 80% of our OpenAI tier limit. During bursts, requests queue with priority based on user tier.
>
> Multi-provider fallback gives resilience: if OpenAI is rate-limited, route to Anthropic or Azure OpenAI.
>
> The trade-off is UX for heavy users. They may hit limits and need to wait. We'd show clear messaging and upgrade paths."

---

## 9. Common Mistakes

- **Only count requests**: "60 req/min" → One 100K token request bypasses
- **Ignore output tokens**: "Limit input" → Output can be 10x input
- **No cost protection**: "No spending limit" → $10K bill surprise
- **Single provider**: "OpenAI only" → One outage = full outage
- **No priority**: "FIFO queue" → Free users starve paid users

---

## 10. Senior-Level Phrases

- "Multi-dimensional limits: requests, tokens, and cost budget."
- "Token bucket allows bursts while enforcing sustained rate."
- "Provider-level pooling with fallback ensures resilience."
- "Estimate tokens before the call to fail fast on limits."

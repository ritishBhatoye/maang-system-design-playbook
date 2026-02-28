# 💾 Prompt Caching (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Prompt caching can reduce LLM costs by **10x** and latency by **50x**. In 2026, this is essential for any production AI system.

---

## 2. Problem Interviewers Are Testing

- Do you understand the cost/latency of LLM calls?
- Can you identify cacheable patterns in prompts?
- Do you know exact vs semantic caching strategies?
- Can you design cache invalidation for AI systems?

---

## 3. Key Concepts

### Why Cache Prompts?

```
LLM costs:
├── GPT-4o: $2.50/1M input tokens, $10/1M output tokens
├── Claude Sonnet: $3/1M input, $15/1M output
└── At scale: Millions of dollars/month

LLM latency:
├── GPT-4: 5-30 seconds for complex responses
├── First token latency: 500ms-2s
└── User waiting = bad UX

Caching value:
├── Same question? Return cached answer
├── Cost: $0 (after first call)
├── Latency: 10ms (cache lookup)
```

### Types of Caching

| Type | Match | Hit Rate | Complexity |
|------|-------|----------|------------|
| **Exact** | Identical prompt | 10-30% | Low |
| **Semantic** | Similar meaning | 40-70% | Medium |
| **Prefix** | Same system prompt | 90%+ | Low |
| **KV Cache** | Same conversation | 100% | High |

---

## 4. Caching Strategies

### 1. Exact Match Caching

```
Simple hash-based lookup:

cache_key = hash(prompt)
if cache.exists(cache_key):
    return cache.get(cache_key)
else:
    response = llm.generate(prompt)
    cache.set(cache_key, response, ttl=3600)
    return response

Pros: Simple, guaranteed accuracy
Cons: Low hit rate, minor variations miss
```

### 2. Semantic Caching

```
Embedding-based similarity:

query_embedding = embed(prompt)
similar = vector_db.search(query_embedding, threshold=0.95)

if similar and similar.score > 0.95:
    return similar.cached_response
else:
    response = llm.generate(prompt)
    vector_db.insert(query_embedding, response)
    return response

Pros: High hit rate
Cons: Approximate matches may differ
```

### 3. Prefix Caching (Provider-Level)

```
Many prompts share the same prefix:

┌──────────────────────────────────────────────────┐
│ SYSTEM: You are a helpful assistant for ACME... │ ← Shared prefix
│ (1000 tokens of context)                        │ ← Cached at provider
├──────────────────────────────────────────────────┤
│ USER: What is your return policy?               │ ← Unique per request
└──────────────────────────────────────────────────┘

OpenAI/Anthropic cache the prefix → 50% token cost reduction
```

### 4. KV Cache (Conversation)

```
Multi-turn conversation:

Turn 1: [System] [User1] [Assistant1]
Turn 2: [System] [User1] [Assistant1] [User2] [Assistant2] ← Cache previous KV

Provider caches attention KV from previous turns
→ Only compute new tokens
→ Faster responses in conversations
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
│                     Exact Cache Check                        │
│              hash(prompt) → Redis lookup                     │
│                                                              │
│              HIT? → Return cached response                   │
└───────────────────────────┬─────────────────────────────────┘
                            │ MISS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Semantic Cache Check                       │
│           embed(prompt) → Vector DB similarity               │
│                                                              │
│              SIMILAR? → Return cached response               │
└───────────────────────────┬─────────────────────────────────┘
                            │ MISS
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                        LLM Call                              │
│                                                              │
│              Generate → Store in both caches                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Cache Invalidation

```
When to invalidate:
├── Knowledge update (new docs indexed)
├── Time-based (TTL expired)
├── User feedback ("bad answer" signal)
├── Model update (new model version)
└── Context change (different user permissions)

Strategy:
├── TTL: Set reasonable expiry (1h - 24h)
├── Version key: cache_key + model_version
├── Segment by user: Don't share across tenants
└── Soft invalidation: Flag stale, regenerate async
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Matching** | Exact | Semantic | Deterministic needed |
| **Scope** | Global | Per-user | Privacy concerns |
| **TTL** | Short (1h) | Long (24h) | Frequently changing data |
| **Storage** | Redis | Vector DB | Exact match sufficient |

---

## 8. Interview Explanation

> "For an LLM-powered Q&A system with 10K queries/day, I'd implement tiered prompt caching.
>
> First layer: exact match caching in Redis. Hash the normalized prompt (lowercase, trimmed) and check cache. This catches repeated questions with 20-30% hit rate.
>
> Second layer: semantic caching. Embed the prompt and search the vector DB for similar past queries (cosine similarity > 0.95). If found, return the cached response. This catches paraphrased questions, pushing hit rate to 50-60%.
>
> For the remaining 40%, we call the LLM and store the response in both caches.
>
> Cache invalidation uses TTL (24 hours for FAQ-style content) plus manual invalidation when underlying docs change. We version the cache key by model version so model updates don't serve stale responses.
>
> The trade-off is freshness for cost. With 60% cache hit rate, we reduce LLM costs by 2.5x and average latency by 10x."

---

## 9. Common Mistakes

- **No caching**: "Every query hits LLM" → Expensive at scale
- **Too aggressive semantic**: "0.8 similarity threshold" → Wrong answers
- **No invalidation**: "Cache forever" → Stale/wrong responses
- **Shared across users**: "One global cache" → Permission leaks
- **Ignoring prompt structure**: "Hash entire prompt" → Miss prefix sharing

---

## 10. Senior-Level Phrases

- "Tiered caching: exact match first, semantic similarity second."
- "0.95+ similarity threshold prevents false positive cache hits."
- "Prefix caching at the provider level reduces token costs by 50%."
- "Cache versioned by model ID to handle model updates cleanly."

# 🤖 Designing LLM-Backed Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

LLM-backed systems are the **hottest topic** in 2026 MAANG interviews. Every major tech company is building AI products, and they need engineers who understand the unique challenges.

---

## 2. Problem Interviewers Are Testing

- Do you understand LLM latency, cost, and reliability?
- Can you design around non-deterministic outputs?
- Do you know when to use LLMs vs traditional approaches?
- Can you build production-grade AI systems?

---

## 3. Key Concepts

### LLM Characteristics

```
Different from traditional services:
├── High latency: 1-30 seconds (vs 50ms)
├── High cost: $0.01-0.10 per request
├── Non-deterministic: Same input → different outputs
├── Token limits: Context window constraints
├── Rate limits: Tokens per minute caps
└── Failure modes: Hallucination, refusal, degradation
```

### When to Use LLMs

| Use Case | LLM Good | LLM Bad |
|----------|----------|---------|
| **Text generation** | ✓ Creative content | ✗ Exact templating |
| **Classification** | ✓ Nuanced/fuzzy | ✗ Simple rules |
| **Extraction** | ✓ Unstructured data | ✗ Structured parsing |
| **Search** | ✓ Semantic understanding | ✗ Exact match |
| **Computation** | ✗ Math errors | ✓ Use code |

---

## 4. Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                      User Request                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       Gateway Layer                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐│
│  │Rate Limiter │ │ Auth/Safety │ │ Request Classifier      ││
│  └─────────────┘ └─────────────┘ └─────────────────────────┘│
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Orchestration Layer                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐│
│  │Prompt Cache │ │  Router     │ │   Retry/Fallback        ││
│  └─────────────┘ └─────────────┘ └─────────────────────────┘│
└───────────────────────────┬─────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
       ┌──────────┐  ┌──────────┐  ┌──────────┐
       │ GPT-4    │  │ Claude   │  │ Local    │
       │ (smart)  │  │ (fallback│  │ (cheap)  │
       └──────────┘  └──────────┘  └──────────┘
```

---

## 5. Key Design Patterns

### 1. Tiered Model Strategy

```python
def route_to_model(request):
    complexity = estimate_complexity(request)
    
    if complexity == "simple":
        return "gpt-3.5-turbo"   # Fast, cheap
    elif complexity == "medium":
        return "gpt-4o-mini"     # Balanced
    else:
        return "gpt-4o"          # Best quality
```

### 2. Semantic Caching

```
Cache by meaning, not exact match:

Request: "What's the weather in NYC?"
Cache key: embedding of request

Similar request: "Weather in New York City?"
→ Cache hit (semantically equivalent)
→ Return cached response

Benefit: 10x cost reduction, 50x latency reduction
```

### 3. Streaming Responses

```
User sends query
    │
    ▼
LLM generates (10 seconds)
    │
    ├── Token 1 → Stream to user (50ms)
    ├── Token 2 → Stream to user (100ms)
    ├── Token 3 → Stream to user (150ms)
    └── ...

Perceived latency: 50ms (first token)
Actual latency: 10 seconds (complete response)
```

---

## 6. RAG Architecture

```
Query: "What's our refund policy?"

1. RETRIEVE
   Query → Embedding
   Search vector DB for similar docs
   Return top 5 chunks

2. AUGMENT
   Build prompt:
   "Context: [retrieved docs]
    Question: [user query]
    Answer based on context only."

3. GENERATE
   LLM generates answer grounded in docs
   Cite sources

┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Query     │ →  │ Vector DB   │ →  │    LLM      │
│ Embedding   │    │   Search    │    │  Generate   │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Model** | GPT-4 (smart) | GPT-3.5 (fast) | Quality critical |
| **Caching** | Semantic (flexible) | Exact (simple) | Varied queries |
| **Output** | Streaming | Complete | Latency sensitive |
| **RAG** | Real-time retrieval | Pre-baked context | Fresh data needed |
| **Hosting** | API (OpenAI) | Self-hosted | Data privacy |

---

## 8. Interview Explanation

> "For an LLM-powered customer support system, I'd design for latency, cost, and reliability.
>
> The gateway handles rate limiting and safety filtering—blocking prompt injection attempts. An orchestration layer routes requests: simple FAQs go to GPT-3.5 for speed, complex issues to GPT-4.
>
> For knowledge grounding, I'd use RAG: embed the query, search our documentation in a vector database, and inject relevant context into the prompt. This reduces hallucination and keeps answers accurate.
>
> Semantic caching reduces cost and latency: if we've answered a similar question recently, return the cached response. This can give 10x cost savings.
>
> The response streams back token-by-token, so users see progress immediately rather than waiting 10 seconds for a complete response.
>
> For reliability, I'd have fallback models: if GPT-4 is slow or unavailable, route to Claude or a local model. The trade-off is reduced quality for maintained availability."

---

## 9. Common Mistakes

- **No caching**: "Every request hits the LLM" → Expensive, slow
- **Single model**: "Always use GPT-4" → Over-engineered for simple tasks
- **Sync responses**: "Wait for complete answer" → Bad UX for long responses
- **No fallbacks**: "If OpenAI down, we're down" → Single point of failure
- **Ignoring hallucination**: "LLM knows everything" → Need grounding

---

## 10. Senior-Level Phrases

- "I'd tier models by task complexity: simple → fast model, complex → smart model."
- "Semantic caching reduces redundant LLM calls for similar queries."
- "RAG grounds responses in our data to reduce hallucination."
- "Streaming first token in 100ms gives good perceived latency despite 10s generation."

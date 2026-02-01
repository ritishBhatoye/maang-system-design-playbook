# 💰 Cost Control for AI Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

AI systems can be **10-100x more expensive** than traditional systems. In 2026, cost control is as important as scaling. Interviewers want to see you can build AI systems that don't bankrupt the company.

---

## 2. Problem Interviewers Are Testing

- Do you understand AI cost drivers?
- Can you optimize cost without sacrificing quality?
- How do you prevent cost explosions?
- Can you build predictable AI spending?

---

## 3. Key Concepts

### AI Cost Breakdown

```
Typical AI system costs:
├── LLM API calls: 60-80% of AI spend
├── Embeddings: 5-10%
├── Vector DB: 5-10%
├── Compute (inference): 10-20%
└── Training/fine-tuning: Variable

Cost examples (2026):
├── GPT-4o: $2.50/1M input, $10/1M output
├── Claude Sonnet: $3/1M input, $15/1M output
├── GPT-3.5: $0.50/1M input, $1.50/1M output
└── Embeddings: $0.02/1M tokens
```

### Cost Drivers

| Driver | Impact | Mitigation |
|--------|--------|------------|
| **Model choice** | 10-50x between models | Use smallest effective model |
| **Token count** | Linear cost | Shorter prompts, summarization |
| **Request volume** | Linear cost | Caching, batching |
| **Output length** | 2-4x vs input cost | Constrain max tokens |
| **Retries** | 2-3x on errors | Better error handling |

---

## 4. Cost Optimization Strategies

### 1. Model Tiering

```python
def route_to_model(request):
    # Estimate complexity
    if is_simple_query(request):
        return "gpt-3.5-turbo"  # $0.002/1K tokens
    elif is_standard_task(request):
        return "gpt-4o-mini"     # $0.30/1K tokens
    else:
        return "gpt-4o"          # $5/1K tokens

Cost reduction: 5-10x for 70% of requests
```

### 2. Prompt Optimization

```
BEFORE (verbose):
"You are a helpful assistant. Your task is to help users 
with their questions. Please provide detailed, accurate 
responses. When answering, be sure to..."
→ 200 tokens in system prompt × 10K requests/day = 2M tokens

AFTER (optimized):
"Answer concisely and accurately."
→ 6 tokens × 10K requests = 60K tokens

Reduction: 97%
```

### 3. Output Length Control

```python
response = llm.generate(
    prompt=user_query,
    max_tokens=500,  # Cap output length
    stop_sequences=["---", "END"],  # Stop early
)

# Many requests generate 2000+ tokens when 500 suffices
# Output tokens cost 2-4x input tokens
# Capping saves 50-80% on output costs
```

### 4. Caching (Covered Separately)

```
Cache hit rate vs cost reduction:
├── 20% hit rate → 20% cost reduction
├── 50% hit rate → 50% cost reduction
├── 80% hit rate → 80% cost reduction

Target: 50-60% for conversational
        80%+ for FAQ/documentation
```

---

## 5. Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                   Cost Control Layer                         │
│                                                              │
│ ┌─────────────┐ ┌─────────────┐ ┌───────────────────────┐  │
│ │Budget Check │ │Usage Meter  │ │Cost Estimator         │  │
│ │(pre-flight) │ │(per request)│ │(predict before call)  │  │
│ └─────────────┘ └─────────────┘ └───────────────────────┘  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Cost Optimization                         │
│                                                              │
│ ┌─────────────┐ ┌─────────────┐ ┌───────────────────────┐  │
│ │Prompt Cache │ │Model Router │ │Token Compressor       │  │
│ │(semantic)   │ │(tier based) │ │(summarize history)    │  │
│ └─────────────┘ └─────────────┘ └───────────────────────┘  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     LLM Providers                            │
│              GPT-4 │ Claude │ GPT-3.5 │ Local                │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Cost Governance

### Budget Enforcement

```python
class CostGovernance:
    def check_budget(self, user_id, estimated_cost):
        daily_spend = self.get_daily_spend(user_id)
        budget = self.get_budget(user_id)
        
        if daily_spend + estimated_cost > budget:
            return False, "Daily budget exceeded"
        
        return True, "OK"
    
    def record_spend(self, user_id, actual_cost):
        self.increment_usage(user_id, actual_cost)
        self.check_alerts(user_id)
```

### Alerting Thresholds

```
Alert when:
├── User exceeds 50% of daily budget → Warning
├── User exceeds 80% of daily budget → Alert
├── User exceeds 100% of daily budget → Block
├── Total spend exceeds hourly projection → Page on-call
└── Single request > $1 → Flag for review
```

---

## 7. Conversation History Management

```
Long conversations are expensive:

Turn 1:  Input: 100 tokens
Turn 5:  Input: 500 tokens (includes history)
Turn 10: Input: 1000 tokens
Turn 20: Input: 2000 tokens

Mitigation strategies:
├── Rolling window: Keep last N turns only
├── Summarization: Periodically summarize history
├── Selective memory: Keep only "important" turns
└── Hierarchical: Full recent, summarized older
```

### Summarization Pattern

```python
def compress_history(conversation):
    if len(conversation) > 10:
        old_turns = conversation[:-5]
        recent_turns = conversation[-5:]
        
        summary = llm.summarize(old_turns)  # Cheap model
        
        return [{"role": "system", "content": f"Prior context: {summary}"}] + recent_turns
    
    return conversation
```

---

## 8. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Model** | Best (expensive) | Good enough (cheap) | Quality critical |
| **Caching** | Aggressive | Conservative | Static content |
| **History** | Full context | Summarized | Quality critical |
| **Budget** | Hard limit | Soft (degrade) | Cost predictability |

---

## 9. Interview Explanation

> "For an AI chatbot serving 1M queries/day, I'd implement a multi-layer cost control architecture.
>
> First, model tiering: route 70% of simple queries to GPT-3.5 (20x cheaper), use GPT-4 only for complex reasoning. A classifier determines complexity before calling any model.
>
> Second, caching: semantic cache with 50% hit rate halves our LLM calls. For FAQ-type queries, we can achieve 80%+ cache hit rate.
>
> Third, prompt optimization: compress system prompts, set max_tokens to 500 (most responses don't need more), and summarize long conversation histories.
>
> For governance, each user has a daily budget. We estimate cost before calling the LLM and reject if it would exceed budget. Alerts fire at 80% threshold.
>
> The trade-off is quality for cost. We accept slightly lower quality for simple queries in exchange for 10x cost reduction."

---

## 10. Common Mistakes

- **No budget limits**: "Let it scale" → $100K surprise bill
- **Always best model**: "GPT-4 for everything" → 20x over-spend
- **Unlimited history**: "Keep full context" → Costs explode per turn
- **Ignoring output**: "Only track input" → Output is 2-4x more expensive
- **No monitoring**: "Check monthly" → Runaway costs for weeks

---

## 11. Senior-Level Phrases

- "Model tiering gives 10x cost reduction with minimal quality impact."
- "Semantic caching at 50% hit rate halves LLM spend."
- "Summarizing conversation history caps per-turn cost growth."
- "Budget enforcement with pre-flight cost estimation prevents overruns."

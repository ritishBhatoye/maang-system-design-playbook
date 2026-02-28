# 🤖 Scaling AI/ML Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

AI/ML systems have **unique scaling challenges** around GPU utilization, model serving, and inference latency. This is increasingly tested at MAANG in 2026.

---

## 2. Problem Interviewers Are Testing

- Do you understand ML-specific infrastructure challenges?
- Can you design for GPU utilization and model serving?
- Do you know the trade-offs between latency, throughput, and cost?

---

## 3. Key Concepts

### ML System Characteristics

```
Challenges unique to ML:
├── GPU scarcity: $3-5/hour per GPU
├── Model size: LLMs are 10-100GB
├── Cold start: Loading model = 10-60 seconds
├── Inference latency: User-facing needs < 100ms
└── Batch size: Throughput vs latency trade-off
```

### Inference vs Training

| Aspect | Training | Inference |
|--------|----------|-----------|
| **Latency** | Minutes to hours | Milliseconds |
| **Throughput** | Batch (maximize) | Variable (user-driven) |
| **GPU usage** | 100% (desired) | Varies (often 10-30%) |
| **Scaling** | Horizontal (data parallel) | Horizontal (replicas) |

---

## 4. Model Serving Patterns

### 1. Online Serving (Synchronous)

```
User request → Load balancer → Model server → Response

Latency target: < 100ms
Use case: Real-time recommendations, fraud detection

┌────────────┐    ┌────────────┐    ┌────────────┐
│   Client   │ →  │  Gateway   │ →  │  Model     │
│            │    │            │    │  Server    │
│            │ ←  │            │ ←  │  (GPU)     │
└────────────┘    └────────────┘    └────────────┘
```

### 2. Offline Batch (Asynchronous)

```
Pre-compute predictions for all users

Use case: Recommendations, personalization

┌────────────┐    ┌────────────┐    ┌────────────┐
│  User DB   │ →  │  Batch Job │ →  │  Results   │
│ (all users)│    │  (Spark)   │    │  Cache     │
└────────────┘    └────────────┘    └────────────┘
```

### 3. Streaming (Near Real-Time)

```
Continuous inference on event stream

Use case: Fraud detection, content moderation

┌────────────┐    ┌────────────┐    ┌────────────┐
│   Kafka    │ →  │  Flink +   │ →  │  Output    │
│  (events)  │    │  ML Model  │    │  Kafka     │
└────────────┘    └────────────┘    └────────────┘
```

---

## 5. Architecture Pattern

```
                    ┌─────────────────────────────────┐
                    │           API Gateway           │
                    │    (rate limiting, auth)        │
                    └───────────────┬─────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
              ┌──────────┐   ┌──────────┐   ┌──────────┐
              │ Model    │   │ Model    │   │ Model    │
              │ Server 1 │   │ Server 2 │   │ Server 3 │
              │ (GPU)    │   │ (GPU)    │   │ (GPU)    │
              └────┬─────┘   └────┬─────┘   └────┬─────┘
                   │              │              │
                   └──────────────┼──────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │      Model Registry       │
                    │   (versioning, A/B test)  │
                    └───────────────────────────┘
```

---

## 6. Optimization Techniques

### Batching Requests

```
Single inference: 10ms per request
Batch of 8: 15ms total → 1.9ms per request

Trade-off: Latency vs Throughput
├── Small batch (1-4): Low latency, low throughput
└── Large batch (16-64): High latency, high throughput
```

### Model Optimization

| Technique | Speedup | Accuracy Impact | Use Case |
|-----------|--------:|:---------------:|----------|
| **Quantization** (FP32→INT8) | 2-4x | 1-2% drop | Edge, mobile |
| **Distillation** | 5-10x | 2-5% drop | Student models |
| **Pruning** | 2-3x | < 1% drop | Sparse models |
| **Caching** | 10-100x | 0% | Repeated queries |

### GPU Optimization

```
Problem: GPU idle between requests
Solution: Request batching, multi-model serving

GPU Utilization:
├── Without batching: 10-20%
├── With batching: 50-70%
└── Multi-model: 80-90%
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Serving** | Online (real-time) | Batch (pre-compute) | Fresh data needed |
| **Model size** | Full model (accurate) | Distilled (fast) | Accuracy critical |
| **Hardware** | GPU | CPU | Cost sensitive, low traffic |
| **Batching** | Dynamic (wait) | Immediate | Throughput > latency |

---

## 8. Interview Explanation

> "For a recommendation system serving 10K requests/sec, I'd design a hybrid architecture.
>
> For popular items and users, pre-compute recommendations hourly using batch inference. These cached results serve 80% of traffic with sub-10ms latency from Redis.
>
> For the long tail (new users, fresh items), we fall back to online inference. The model server uses dynamic batching—collecting requests for 5ms, then running batch inference on GPU. This improves throughput 5x at the cost of 5ms latency.
>
> The model itself would be quantized (INT8) for 3x inference speedup with minimal accuracy loss. We'd use model distillation to create a smaller student model for serving.
>
> For GPU utilization, we serve multiple models on the same GPU using Triton Inference Server. This keeps GPU at 70%+ utilization versus 20% with single-model serving.
>
> The trade-off is complexity. We maintain both batch and online paths, with cache-aside pattern for pre-computed recommendations."

---

## 9. Common Mistakes

- **GPU for everything**: "Put model on GPU" → Expensive at low QPS
- **No caching**: "Always run inference" → Repeated work
- **Single-threaded serving**: "One request at a time" → Low utilization
- **Cold model loading**: "Load model per request" → Cold start latency

---

## 10. Senior-Level Phrases

- "I'd use dynamic batching to trade 5ms latency for 5x throughput."
- "Pre-compute predictions for the head of the distribution, online for the tail."
- "INT8 quantization gives 3x speedup with 1-2% accuracy drop."
- "Multi-model serving maximizes GPU utilization across different workloads."

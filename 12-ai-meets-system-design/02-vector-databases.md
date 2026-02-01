# 🔢 Vector Databases (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Vector databases are the **backbone of semantic search and RAG**. In 2026, nearly every AI system uses them. Understanding their internals is essential.

---

## 2. Problem Interviewers Are Testing

- Do you understand embeddings and similarity search?
- Can you choose the right vector DB for your use case?
- Do you know the trade-offs between accuracy and speed?
- Can you scale vector search to billions of vectors?

---

## 3. Key Concepts

### What is a Vector Database?

```
Traditional DB: SELECT * FROM users WHERE name = 'John'
              → Exact match

Vector DB: Find items SIMILAR to this query
         → Semantic/fuzzy match

Embedding: Text/image → [0.1, -0.2, ..., 0.5] (1536 dims)
Distance:  |query - item| → Similarity score
```

### Embedding Models

| Model | Dimensions | Use Case |
|-------|-----------|----------|
| OpenAI text-embedding-3-small | 1536 | General purpose |
| OpenAI text-embedding-3-large | 3072 | High accuracy |
| Cohere embed-v3 | 1024 | Multilingual |
| BGE / E5 (open source) | 768-1024 | Self-hosted |

### Similarity Metrics

```
Cosine Similarity: Angle between vectors
├── Range: -1 to 1 (1 = identical direction)
├── Best for: Normalized embeddings, text
└── Used by: Most applications

Euclidean Distance (L2): Straight-line distance
├── Range: 0 to ∞ (0 = identical)
├── Best for: Magnitude matters
└── Used by: Image similarity

Dot Product: Cosine * magnitudes
├── Range: -∞ to ∞
├── Best for: When magnitude is meaningful
└── Used by: Recommendation systems
```

---

## 4. Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                      Ingestion Pipeline                      │
│                                                              │
│  Document → Chunk → Embed → Store                           │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ Raw Doc │→ │ Chunker │→ │Embedding│→ │Vector DB│        │
│  │         │  │(512 tok)│  │ Model   │  │         │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                       Query Pipeline                         │
│                                                              │
│  Query → Embed → Search → Rerank → Return                   │
│                                                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  Query  │→ │Embedding│→ │ ANN     │→ │ Rerank  │        │
│  │         │  │ Model   │  │ Search  │  │ (top K) │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Indexing Algorithms

### Approximate Nearest Neighbor (ANN)

```
Exact k-NN: Compare query to ALL vectors
          → O(n) per query
          → Too slow for millions of vectors

ANN: Trade accuracy for speed
   → 95-99% accuracy
   → 10-100x faster
```

### Common Algorithms

| Algorithm | Speed | Accuracy | Memory | Use Case |
|-----------|-------|----------|--------|----------|
| **HNSW** | Fast | High | High | Production default |
| **IVF** | Medium | Medium | Medium | Large scale |
| **PQ** | Fast | Lower | Low | Billions of vectors |
| **Flat** | Slow | 100% | Medium | < 100K vectors |

### HNSW (Hierarchical Navigable Small World)

```
Multi-layer graph:

Layer 2:    A ─────────────────────── Z
            │                         │
Layer 1:    A ─── D ─── M ─── R ─── Z
            │     │     │     │     │
Layer 0:    A B C D E F G H I J ... Z

Search:
1. Start at top layer, find approximate region
2. Drop to lower layer, refine
3. Bottom layer gives final candidates
4. Rerank candidates by exact distance

Build params:
├── M: Max connections per node (16-64)
├── efConstruction: Build quality (100-500)
└── Higher = better quality, more memory
```

---

## 6. Vector DB Options

| Database | Type | Scale | Best For |
|----------|------|-------|----------|
| **Pinecone** | Managed | Billions | Production, serverless |
| **Weaviate** | OSS/Managed | Millions | Hybrid search |
| **Milvus** | OSS | Billions | Large scale, self-hosted |
| **Qdrant** | OSS/Managed | Millions | Rust performance |
| **Chroma** | OSS | 100K | Development, embedded |
| **pgvector** | Postgres ext | Millions | Existing Postgres |

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Accuracy** | Exact (flat) | Approximate (HNSW) | < 100K vectors |
| **Hosting** | Managed (Pinecone) | Self-hosted (Milvus) | Speed to market |
| **Hybrid** | Vector only | Vector + keyword | Product search |
| **Reranking** | No | Cross-encoder | Accuracy critical |

---

## 8. Interview Explanation

> "For semantic search over 10 million documents, I'd use a vector database with HNSW indexing.
>
> The ingestion pipeline chunks documents into 512-token segments, embeds with text-embedding-3-small, and stores in Pinecone. Each chunk includes metadata for filtering.
>
> On query, we embed the user query, search for top 100 candidates with ANN (fast but approximate), then rerank with a cross-encoder model for the final top 10. This two-stage approach balances speed and accuracy.
>
> For hybrid search (semantic + keyword), I'd combine vector similarity with BM25 keyword matching using reciprocal rank fusion. This handles both conceptual matches and exact term matches.
>
> The trade-off with ANN is accuracy: HNSW gives ~95% recall at 100x speed improvement. For most applications, that's acceptable."

---

## 9. Common Mistakes

- **Skip reranking**: "Top-10 from ANN is final" → Low precision
- **Chunk too large**: "One embedding per doc" → Loses detail
- **Wrong metric**: "Use Euclidean for text" → Cosine is better
- **No filtering**: "Search all vectors" → Should filter by metadata first
- **Overfit to benchmark**: "90% recall on test set" → Different in production

---

## 10. Senior-Level Phrases

- "HNSW gives sub-linear search at the cost of 95% vs 100% accuracy."
- "Two-stage retrieval: fast ANN for candidates, cross-encoder for final ranking."
- "Hybrid search with reciprocal rank fusion combines semantic and keyword matches."
- "Chunk size of 512 tokens balances context with embedding quality."

# Search Services

## 1. Problem Statement

How do we implement fast, scalable search functionality that can handle complex queries and large datasets?

---

## 2. Search Architecture

### Components
* **Index**: Inverted index mapping terms to documents
* **Query Parser**: Parses user query into search terms
* **Ranking Algorithm**: Scores and ranks results
* **Result Aggregator**: Combines results from multiple shards

### Flow
```
User Query → Query Parser → Search Index → Ranking → Results
```

---

## 3. Inverted Index

### Structure
* **Term → [Document IDs]**: Maps words to documents containing them
* **Example**: "apple" → [doc1, doc5, doc12]

### Building Index
1. Tokenize documents (split into words)
2. Normalize (lowercase, remove punctuation)
3. Build index (term → document mapping)
4. Store metadata (term frequency, positions)

### Trade-offs
* **Build Time**: Indexing is expensive (batch or incremental)
* **Storage**: Index can be larger than original data
* **Update Cost**: Updating index is expensive

---

## 4. Search Types

### Full-Text Search
* Search within document content
* **Use Case**: Search engines, document search
* **Example**: Elasticsearch, Solr

### Keyword Search
* Search by specific keywords/tags
* **Use Case**: Product search, filtering
* **Example**: Database indexes, Redis

### Fuzzy Search
* Handles typos and variations
* **Use Case**: Autocomplete, user queries
* **Example**: Edit distance, n-grams

### Semantic Search
* Understands meaning, not just keywords
* **Use Case**: AI-powered search
* **Example**: Vector embeddings, ML models

---

## 5. Ranking Algorithms

### TF-IDF (Term Frequency-Inverse Document Frequency)
* **TF**: How often term appears in document
* **IDF**: How rare term is across all documents
* **Score**: TF × IDF
* **Use Case**: Basic relevance ranking

### BM25 (Best Matching 25)
* Improved version of TF-IDF
* Handles document length normalization
* **Use Case**: Modern search engines (default in Elasticsearch)

### Learning to Rank (LTR)
* ML model trained on user behavior
* Considers clicks, conversions, time spent
* **Use Case**: Personalized, optimized ranking

### Hybrid Ranking
* Combines multiple signals
* **Example**: Relevance score + recency + popularity + personalization

---

## 6. Distributed Search

### Sharding
* Partition index by document ID or hash
* **Example**: Shard by user_id, or hash(document_id)

### Replication
* Multiple copies of each shard
* **Benefits**: Fault tolerance, read scaling
* **Trade-off**: More storage, write amplification

### Query Flow
1. Query → All shards (broadcast)
2. Each shard → Returns top N results
3. Aggregator → Merges and re-ranks
4. Return → Top K results to user

---

## 7. Search Services

### Elasticsearch
* **Type**: Distributed search and analytics
* **Strengths**: Full-text search, aggregations, real-time
* **Use Case**: Log analysis, product search, analytics

### Solr
* **Type**: Enterprise search platform
* **Strengths**: Mature, feature-rich
* **Use Case**: Enterprise search, e-commerce

### Algolia
* **Type**: Managed search service
* **Strengths**: Fast, easy to use, typo tolerance
* **Use Case**: SaaS products, e-commerce

### AWS CloudSearch / OpenSearch
* **Type**: Managed search service
* **Strengths**: AWS integration, scalable
* **Use Case**: AWS-based applications

---

## 8. Autocomplete vs Search

### Autocomplete
* **Purpose**: Suggest queries as user types
* **Data Structure**: Trie (prefix tree)
* **Latency**: <50ms (critical)
* **Use Case**: Search box suggestions

### Search
* **Purpose**: Find documents matching query
* **Data Structure**: Inverted index
* **Latency**: <200ms (acceptable)
* **Use Case**: Full search results

**Key Difference**: Autocomplete is about query suggestions, search is about finding content.

---

## 9. How to Explain This in an Interview

> "For search, I use Elasticsearch because it provides full-text search, distributed indexing, and ranking out of the box. I shard the index by document type or user_id to distribute load. For ranking, I use BM25 as the base algorithm and add signals like recency and popularity. For autocomplete, I use a separate Trie data structure in Redis for sub-50ms latency, since autocomplete needs to be faster than full search."

**Key Points:**
* Separate autocomplete from full search
* Use appropriate data structures (Trie vs Inverted Index)
* Mention sharding and replication for scale
* Explain ranking strategy

---

## 10. Common Mistakes

* **Using database LIKE queries**: "SELECT * WHERE name LIKE '%query%'"
  * Fix: Use proper search engine (Elasticsearch) for full-text search
* **No ranking**: "Return all matching documents"
  * Fix: Implement ranking (TF-IDF, BM25, or ML-based)
* **Synchronous indexing**: "Index during write"
  * Fix: Async indexing to avoid blocking writes
* **Single shard**: "All data in one index"
  * Fix: Shard index for horizontal scaling
* **Ignoring typos**: "Exact match only"
  * Fix: Use fuzzy matching or typo tolerance
* **No caching**: "Every query hits search engine"
  * Fix: Cache popular queries in Redis

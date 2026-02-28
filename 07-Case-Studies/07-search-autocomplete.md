# 🔍 Search Autocomplete System (Google Search)

## 1. Problem Statement

Design an autocomplete system that suggests search queries as users type, like **Google Search** or **Amazon Search**.

---

## 2. Clarifying Questions (Ask First)

* What's the data source? (Historical queries, product names, user data)
* How many suggestions? (5, 10, 20)
* What's the ranking? (Popularity, relevance, personalization)
* Real-time updates? (New queries appear immediately)
* Global or personalized? (Same for everyone vs per-user)

---

## 3. Requirements

### Functional

* Return suggestions as user types
* Rank suggestions by relevance/popularity
* Support personalization (optional)
* Handle typos (fuzzy matching)

### Non-Functional

* Low latency (<50ms p95)
* High throughput (millions of requests/day)
* Real-time or near-real-time updates
* Handle long-tail queries

---

## 4. Scale Assumptions

* 1B users
* 10B searches/day
* Average: 10 searches/user/day
* Peak QPS: 100K requests/second
* Unique queries: 100M (long tail)
* Top 10K queries: 80% of traffic
* Average query length: 20 characters
* Storage: 100M queries × 20 bytes = 2GB (compressed)

---

## 5. High-Level Architecture

```
User Types → API Gateway → Autocomplete Service
                            │
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
    Trie/Prefix Tree    Cache (Redis)   Analytics Service
    (In-Memory)         (Hot Queries)   (Query Logs)
            │
            ↓
    Query Database
    (Popular Queries)
```

---

## 6. Core Components

* **Autocomplete Service**: Main service that returns suggestions
* **Trie Data Structure**: Fast prefix matching (in-memory)
* **Cache (Redis)**: Caches popular query prefixes
* **Query Database**: Stores query frequencies (NoSQL)
* **Analytics Service**: Tracks queries, updates frequencies
* **Personalization Service**: Ranks by user history (optional)
* **Fuzzy Matching Service**: Handles typos (optional)

---

## 7. Data Flow

**Query Flow (Prefix Matching)**
1. User → Types "app" (3 characters)
2. Client → Sends GET /api/autocomplete?q=app
3. Autocomplete Service → Check cache for prefix "app"
4. If cache hit → Return top 10 suggestions (<10ms)
5. If cache miss → Query Trie for all queries starting with "app"
6. Trie → Returns candidate queries (e.g., "apple", "application", "app store")
7. Autocomplete Service → Rank by frequency/popularity
8. Autocomplete Service → Return top 10, cache result
9. User → Sees suggestions (<50ms)

**Trie Lookup Flow**
1. Autocomplete Service → Traverse Trie with prefix "app"
2. Trie → Finds node for "app"
3. Trie → DFS to collect all queries under "app" node
4. Trie → Returns list: ["apple", "application", "app store", ...]
5. Autocomplete Service → Sorts by frequency, returns top 10

**Cache Strategy**
1. Cache key: prefix (e.g., "app", "appl", "apple")
2. Cache value: top 10 suggestions for that prefix
3. TTL: 1 hour (or until query frequencies update)
4. Popular prefixes (top 1K) → Always cached
5. Long-tail prefixes → Cache on-demand

**Query Frequency Update**
1. User → Submits search query "apple"
2. Analytics Service → Logs query with timestamp
3. Background Job → Aggregates query frequencies (every hour)
4. Background Job → Updates Trie with new frequencies
5. Background Job → Invalidates cache for affected prefixes
6. Next request → Uses updated frequencies

**Personalization Flow (Optional)**
1. User → Types "python"
2. Autocomplete Service → Get user's search history
3. Personalization Service → Boost queries user searched before
4. Autocomplete Service → Merge global + personal rankings
5. Autocomplete Service → Return personalized top 10

---

## 8. Bottlenecks

* **Trie Memory**: 100M queries in memory
  * Solution: Compress Trie, or use disk-backed Trie, or shard by first character
* **Cache Misses**: Long-tail queries not cached
  * Solution: Cache popular prefixes, on-demand caching for others
* **Real-time Updates**: New queries don't appear immediately
  * Solution: Async updates (hourly batch), or stream updates for hot queries
* **Fuzzy Matching**: Handling typos is expensive
  * Solution: Use edit distance algorithm, limit to 1-2 character errors
* **Personalization**: Per-user ranking is slow
  * Solution: Cache personalized results, or use simpler heuristics
* **Global Trie**: Single Trie for all users
  * Solution: Shard Trie by first character, or use distributed cache

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Data Structure** | Trie (in-memory) | Fast prefix matching O(m) where m=prefix length | Memory intensive, but acceptable for 100M queries |
| **Caching Strategy** | Cache popular prefixes, on-demand for others | Reduces Trie lookups | May miss long-tail queries, but they're rare |
| **Update Frequency** | Hourly batch updates | Simple, efficient | Slight delay for new queries, but acceptable |
| **Personalization** | Optional, cache per user | Better UX for returning users | More complex, higher memory usage |
| **Fuzzy Matching** | Limit to 1-2 char errors | Handles common typos | May miss complex typos, but covers 90% of cases |

**Key Trade-off:**
> "I use an in-memory Trie for fast prefix matching because <50ms latency is critical for autocomplete. The trade-off is high memory usage (~10GB for 100M queries), but this is acceptable for the performance gain. I cache popular prefixes in Redis to reduce Trie lookups and handle cache misses with on-demand Trie queries."

---

## 10. How to Explain This in an Interview

> "I'll design an autocomplete system using a Trie data structure for fast prefix matching and Redis cache for popular queries.
>
> For architecture, I use an in-memory Trie that stores all queries. Each node represents a character, and paths from root to leaf represent complete queries. I store query frequencies at leaf nodes for ranking.
>
> When a user types a prefix (e.g., 'app'), I first check the cache. If the prefix is cached, I return the top 10 suggestions immediately (<10ms). If not, I traverse the Trie to find all queries starting with that prefix, rank them by frequency, return the top 10, and cache the result.
>
> For caching, I cache the top 1K most popular prefixes (which cover 80% of traffic) with a 1-hour TTL. For long-tail prefixes, I cache on-demand. This balances memory usage and latency.
>
> For query frequency updates, I log all searches to an analytics service. A background job aggregates frequencies hourly and updates the Trie. I invalidate cache for affected prefixes to ensure fresh results.
>
> For personalization, I optionally boost queries from the user's search history. I cache personalized results per user to avoid recomputing on every request.
>
> For scale, I shard the Trie by first character (a-z, 0-9) across multiple servers if needed. Each server handles queries for its character range.
>
> The key trade-off is memory vs latency: in-memory Trie gives <50ms latency but uses ~10GB memory for 100M queries. This is acceptable because autocomplete latency is critical for user experience."

**Key Phrases:**
* "Trie data structure for O(m) prefix matching"
* "Cache popular prefixes for 80% of traffic"
* "Hourly batch updates for query frequencies"
* "Shard Trie by first character for scale"

---

## 11. Common Mistakes

* **Database lookup for every prefix**: "We query DB for each keystroke"
  * Fix: Use Trie for fast in-memory lookups
* **No caching**: "Every request hits Trie"
  * Fix: Cache popular prefixes to reduce Trie lookups
* **Synchronous frequency updates**: "We update frequencies in real-time"
  * Fix: Use batch updates (hourly) for efficiency
* **Storing all queries in Trie**: "100M queries in single Trie"
  * Fix: Shard by first character, or use disk-backed Trie
* **No ranking**: "Return queries in random order"
  * Fix: Rank by frequency/popularity, return top N
* **Ignoring long-tail**: "Only cache top 100 queries"
  * Fix: Cache top 1K prefixes, on-demand for others
* **Complex fuzzy matching**: "We handle all typos"
  * Fix: Limit to 1-2 character errors for performance

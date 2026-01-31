# 🔗 URL Shortener (2026 MAANG Edition)

## 1. Problem Statement

Design a system like **bit.ly** that shortens long URLs and redirects users at scale.

---

## 2. Clarifying Questions (Ask First)

* Expected read vs write ratio? (Typically 100:1 or 10:1)
* Custom aliases required? (e.g., bit.ly/my-link)
* URL expiration needed? (TTL for short URLs)
* Global or regional traffic? (Affects CDN strategy)
* Analytics required? (Click tracking, geolocation)

---

## 3. Requirements

### Functional

* Generate short URLs from long URLs
* Redirect short URL → original URL
* Handle custom aliases (optional)
* URL expiration (optional)
* Analytics (optional)

### Non-Functional

* Low latency redirects (<50ms p95)
* Highly available (99.99%)
* Horizontally scalable
* Durable (URLs don't get lost)

---

## 4. Scale Assumptions

* 100M URLs created per day
* 10:1 read/write ratio (1B reads/day)
* Peak QPS: ~15K writes, ~150K reads
* Average URL length: 100 characters
* Storage: ~10B URLs × 100 bytes = 1TB (raw), ~3TB with replication
* URLs never deleted (or 5-year retention)

---

## 5. High-Level Architecture

```
Client → Load Balancer → API Service (stateless)
                       → Cache (Redis) - Hot URLs
                       → Database (NoSQL) - All URLs
                       → ID Generator Service
```

---

## 6. Core Components

* **API Service**: Handles create/redirect requests (stateless)
* **ID Generator**: Generates unique short codes (Base62 encoding)
* **Cache (Redis)**: Stores hot URLs for fast redirects
* **Database (NoSQL)**: Persistent storage (DynamoDB/Cassandra)
* **Load Balancer**: Distributes traffic

**ID Generation Options:**
1. **Counter-based**: Single counter service → Base62 encode (collision risk if multiple instances)
2. **Hash-based**: MD5/SHA of long URL → Take first 7 chars (collision possible, need checking)
3. **Distributed ID**: Snowflake/Twitter ID → Base62 encode (best for scale)

---

## 7. Data Flow

**Write Flow (Create Short URL)**
1. Client → POST /api/v1/shorten {long_url}
2. API → Validate URL
3. API → Generate unique ID (from ID generator)
4. API → Check if custom alias (if provided)
5. API → Store mapping: short_code → long_url in DB
6. API → Store in cache (optional, for hot URLs)
7. API → Return short URL: bit.ly/{short_code}

**Read Flow (Redirect)**
1. Client → GET bit.ly/{short_code}
2. Load Balancer → Route to API server
3. API → Check cache (Redis) for short_code
4. If cache hit → Return 302 redirect to long_url (<10ms)
5. If cache miss → Query database
6. If found → Store in cache (with TTL), return 302
7. If not found → Return 404

**ID Generator Flow (Distributed)**
1. Multiple ID generator instances
2. Each instance has unique machine ID
3. Generate ID: timestamp + machine_id + sequence
4. Encode to Base62 (0-9, a-z, A-Z) → 7 characters = 62^7 = 3.5T unique URLs

---

## 8. Bottlenecks

* **ID Generation Collisions**: Multiple instances generating same ID
  * Solution: Distributed ID generator (Snowflake), or single counter with locks
* **Cache Misses**: Hot URLs not in cache
  * Solution: Pre-warm cache for popular URLs, increase cache size
* **Database Write Bottleneck**: 15K writes/sec to single DB
  * Solution: Shard database by short_code hash, or use managed NoSQL (DynamoDB)
* **Hot Keys**: Single short_code gets all traffic (viral URL)
  * Solution: Cache with replication, or CDN for redirects
* **Database Read Bottleneck**: 150K reads/sec
  * Solution: Aggressive caching (80%+ hit rate), read replicas

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Database** | NoSQL (DynamoDB) | Horizontal scale for writes, simple key-value lookups | Lose complex queries, but don't need them |
| **ID Generation** | Distributed ID (Snowflake) | No collisions, scalable | More complex than counter, but necessary at scale |
| **Caching Strategy** | Cache-aside with TTL | Simple, cache hot URLs | May serve stale data if URL updated (rare) |
| **Custom Aliases** | Store in same DB with check | Simple | Need to check uniqueness, but acceptable |
| **URL Expiration** | TTL in DB, background job to delete | Don't block redirects | Eventual cleanup, acceptable |

**Key Trade-off:**
> "I optimize for read latency using aggressive caching (Redis) because traffic is 10:1 read-heavy. I accept eventual consistency between cache and DB for better scalability. The trade-off is a small chance of serving stale redirects, but URL updates are rare."

---

## 10. How to Explain This in an Interview

> "I'll start with a stateless API service behind a load balancer. For ID generation, I use a distributed ID generator (like Snowflake) to avoid collisions at scale. I encode IDs to Base62 to get 7-character short codes, which gives us 3.5 trillion unique URLs - more than enough.
>
> For storage, I use NoSQL (DynamoDB) sharded by short_code because we need horizontal write scaling for 15K writes/sec, and our access pattern is simple key-value lookups. The trade-off is we lose complex queries, but we don't need them.
>
> For redirects, I use Redis cache with cache-aside pattern. Since reads are 10x writes, I optimize for read latency. I target 80%+ cache hit rate, which means only 30K reads/sec hit the database - manageable with read replicas.
>
> If a URL goes viral, I handle hot keys by caching with replication and potentially using CDN for redirects. For custom aliases, I check uniqueness in the database before storing."

**Key Phrases:**
* "Optimize for read latency because traffic is read-heavy"
* "Use NoSQL for horizontal write scaling"
* "Distributed ID generator to avoid collisions"
* "Cache-aside pattern with 80%+ hit rate target"

---

## 11. Common Mistakes

* **Skipping scale assumptions**: "We need a database"
  * Fix: Always state numbers: "15K writes/sec, 150K reads/sec"
* **Not justifying database choice**: "We use MySQL"
  * Fix: "We use NoSQL because we need horizontal write scaling for 15K writes/sec"
* **Hash-based ID without collision handling**: "MD5 hash, take first 7 chars"
  * Fix: Check for collisions, or use distributed ID generator
* **No caching strategy**: "All reads go to database"
  * Fix: "Redis cache with 80% hit rate reduces DB load by 5x"
* **Forgetting custom aliases**: "Only auto-generated codes"
  * Fix: Mention uniqueness check if custom aliases required
* **Not handling hot keys**: "Cache handles everything"
  * Fix: "For viral URLs, we use cache replication or CDN"
* **Single point of failure**: "One ID generator"
  * Fix: "Distributed ID generator with multiple instances"

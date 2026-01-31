# Sample Interviews

## 1. Problem Statement

How do real system design interviews flow, and what does a good answer look like?

---

## 2. Interview Structure

### Typical 45-Minute Interview
* **0-5 min**: Problem clarification
* **5-15 min**: Requirements and scale
* **15-35 min**: Architecture and deep dive
* **35-40 min**: Trade-offs and edge cases
* **40-45 min**: Wrap-up and questions

---

## 3. Sample Interview: URL Shortener

### Opening (0-2 min)

**Interviewer**: "Design a URL shortener like bit.ly"

**Candidate**: "Let me make sure I understand: we're building a system that takes long URLs and generates short URLs, then redirects users when they access the short URL. Is that correct?"

**Interviewer**: "Yes, exactly."

**Candidate**: "Great. Before I start designing, let me ask a few clarifying questions..."

### Clarifying Questions (2-5 min)

**Candidate**: 
1. "What's the expected scale? Users, URLs per day, QPS?"
2. "Do we need custom aliases, or just auto-generated short codes?"
3. "What are the latency requirements for redirects?"
4. "Do URLs expire, or are they permanent?"

**Interviewer**: 
1. "100M URLs per day, 10:1 read/write ratio"
2. "Custom aliases are optional"
3. "Sub-50ms for redirects"
4. "Permanent URLs"

### Requirements (5-10 min)

**Candidate**: "Based on that, here are my requirements:

**Functional:**
- Generate short URLs from long URLs
- Redirect short URL to original URL
- Support custom aliases (optional)

**Non-Functional:**
- Low latency redirects (<50ms p95)
- High availability (99.99%)
- Horizontally scalable
- Durable (URLs don't get lost)

**Scale Assumptions:**
- 100M URLs/day = ~15K writes/sec peak
- 10:1 read/write = ~150K reads/sec peak
- Storage: 10B URLs × 100 bytes = 1TB (raw), ~3TB with replication"

### High-Level Architecture (10-15 min)

**Candidate**: "At a high level, I see:

```
Client → Load Balancer → API Service (stateless)
                       → Cache (Redis) - Hot URLs
                       → Database (NoSQL) - All URLs
                       → ID Generator Service
```

The API service handles both creating short URLs and redirecting. It's stateless so we can scale horizontally. The cache stores hot URLs for fast redirects. The database stores all URL mappings persistently."

### Deep Dive: ID Generation (15-20 min)

**Interviewer**: "How do you generate unique short codes?"

**Candidate**: "I have a few options:

1. **Counter-based**: Single counter service, encode to Base62
   - Pros: Simple, no collisions
   - Cons: Single point of failure, needs coordination

2. **Hash-based**: Hash long URL, take first 7 chars
   - Pros: Deterministic, no coordination
   - Cons: Collision possible, need checking

3. **Distributed ID**: Snowflake-like ID generator
   - Pros: No collisions, scalable
   - Cons: More complex

I'd choose distributed ID generator (like Snowflake) because it's scalable and avoids collisions. I encode the ID to Base62 to get 7-character codes, which gives us 62^7 = 3.5 trillion unique URLs - more than enough."

### Deep Dive: Database Choice (20-25 min)

**Interviewer**: "Why NoSQL over SQL?"

**Candidate**: "I'm choosing NoSQL (like DynamoDB) because:

1. **Horizontal Write Scaling**: We need 15K writes/sec, which is hard with a single SQL primary
2. **Simple Access Pattern**: We're doing simple key-value lookups (short_code → long_url)
3. **No Complex Queries**: We don't need joins or complex queries

The trade-off is we lose ACID transactions and complex queries, but for this use case, we don't need them. If we needed strong consistency or complex queries, I'd choose SQL."

### Deep Dive: Caching (25-30 min)

**Interviewer**: "How does caching work?"

**Candidate**: "I use Redis with cache-aside pattern:

1. **Read Flow**: Check cache first, if miss query DB, store in cache
2. **Write Flow**: Write to DB, invalidate cache (or use TTL)
3. **TTL**: 1 hour for most URLs, longer for popular ones

I target 80% cache hit rate, which means only 30K reads/sec hit the database - manageable with read replicas. The trade-off is we may serve slightly stale URLs, but URL updates are rare, so this is acceptable."

### Trade-offs (30-35 min)

**Interviewer**: "What are the main trade-offs?"

**Candidate**: "The key trade-offs are:

1. **NoSQL vs SQL**: Scalability vs ACID transactions - I chose scalability
2. **Cache Consistency**: Eventual consistency vs strong consistency - I chose eventual for performance
3. **ID Generation**: Distributed ID vs counter - I chose distributed for scalability

For each trade-off, I optimized for the primary requirement: horizontal scalability and low latency."

### Edge Cases (35-40 min)

**Interviewer**: "What if a URL goes viral?"

**Candidate**: "Good question. If a short URL gets millions of requests:

1. **Hot Key Problem**: Single cache key gets all traffic
   - Solution: Cache replication, or CDN for redirects
2. **Database Load**: Even with cache, some requests hit DB
   - Solution: Read replicas, aggressive caching
3. **Cache Stampede**: All requests miss simultaneously
   - Solution: Lock on miss, or pre-warm cache

I'd also monitor for hot keys and proactively cache popular URLs."

### Wrap-up (40-45 min)

**Interviewer**: "Any final thoughts?"

**Candidate**: "To summarize:
- Stateless API services for horizontal scaling
- NoSQL database for write scalability
- Redis cache for read performance
- Distributed ID generator for unique codes

If we had more time, I'd explore:
- Analytics (click tracking)
- URL expiration
- Rate limiting
- Geographic distribution

Does this design address your requirements?"

---

## 4. What Made This Good

### ✅ Strengths
* Asked clarifying questions first
* Stated requirements explicitly
* Explained trade-offs clearly
* Handled edge cases
* Showed system thinking

### ✅ Key Phrases Used
* "Let me make sure I understand..."
* "I'm choosing X because..."
* "The trade-off is..."
* "Does this make sense?"

---

## 5. Common Mistakes to Avoid

### ❌ Jumping to Solution
* **Bad**: "We need Redis and DynamoDB"
* **Good**: "Let me understand requirements first, then choose technologies"

### ❌ No Trade-offs
* **Bad**: "We use NoSQL"
* **Good**: "I choose NoSQL for scalability, trade-off is we lose ACID"

### ❌ Ignoring Scale
* **Bad**: "We need a database"
* **Good**: "We need a horizontally scalable database for 15K writes/sec"

---

## 6. Practice Scenarios

### Scenario 1: Chat System
* Practice explaining WebSocket vs polling
* Practice explaining message ordering
* Practice explaining offline handling

### Scenario 2: News Feed
* Practice explaining push vs pull
* Practice explaining ranking algorithms
* Practice explaining caching strategy

### Scenario 3: Rate Limiter
* Practice explaining algorithms (token bucket, sliding window)
* Practice explaining distributed rate limiting
* Practice explaining trade-offs (accuracy vs memory)

---

## 7. Self-Evaluation

After practicing, ask yourself:
* Did I ask clarifying questions?
* Did I state requirements explicitly?
* Did I explain trade-offs?
* Did I handle edge cases?
* Was I clear and structured?

---

## 8. How to Use This

1. **Read the sample**: Understand the flow
2. **Practice yourself**: Try explaining the same problem
3. **Compare**: What did you miss?
4. **Refine**: Incorporate good phrases and structure
5. **Repeat**: Practice with different problems

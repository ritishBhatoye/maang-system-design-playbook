# 📱 How Meta Handles Feed (2026 MAANG Edition)

## 1. Why This Matters in Interviews

The Facebook/Instagram News Feed is one of the most complex **real-time personalization systems** at scale. Interviewers love this topic for testing fanout strategies.

---

## 2. The Feed Challenge

```
Scale:
├── 3+ billion users
├── Billions of posts per day
├── Each user has 100-1000 friends/follows
├── Feed must render in < 1 second
└── Content must be personalized and ranked

Fundamental question:
How do you show the right content to the right user in real-time?
```

---

## 3. Key Architectural Decisions

### Push vs Pull vs Hybrid

```
PUSH (Write-time fanout):
User posts → Fan out to all followers' feeds
├── Pro: Read is fast (feed pre-computed)
├── Con: Celebrities with 100M followers = disaster
└── Works for users with < 10K followers

PULL (Read-time aggregation):
User opens app → Fetch posts from all followed users
├── Pro: No write amplification
├── Con: Slow reads (aggregate 1000 sources)
└── Works for users following < 100 accounts

META'S HYBRID:
├── Regular users: Push (pre-compute)
├── Celebrities: Pull (aggregate at read time)
└── Combination gives best of both
```

### Architecture Diagram

```
                    ┌─────────────────────────────────────┐
                    │           User Posts                │
                    └───────────────┬─────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                                           ▼
       ┌──────────────┐                          ┌──────────────┐
       │  Fan-out     │                          │  Celebrity   │
       │  Service     │                          │  Post Store  │
       │ (< 10K fans) │                          │ (pull later) │
       └──────┬───────┘                          └──────────────┘
              │
              ▼
       ┌──────────────┐
       │  Feed Cache  │
       │   (Redis)    │ ← User's pre-computed feed
       └──────────────┘
              │
              ▼
       ┌──────────────┐
       │  Ranking     │ ← ML models score posts
       │  Service     │
       └──────────────┘
              │
              ▼
       ┌──────────────┐
       │    User      │
       │   Device     │
       └──────────────┘
```

---

## 4. Ranking System

```
Feed isn't chronological—it's ranked by engagement prediction:

Score = P(like) × W1 + P(comment) × W2 + P(share) × W3 + freshness × W4

ML Features:
├── User-post affinity (past interactions)
├── Content type preference
├── Time since post
├── Social signals (friends liked)
├── Content quality score
└── Session context
```

### Ranking Pipeline

```
1. Candidate generation: ~10,000 eligible posts
2. First-pass ranking: Light model → top 1,000
3. Second-pass ranking: Heavy model → top 100
4. Final ranking: Diversity, freshness → top 50
5. Render: Show first 10, lazy load rest

Latency budget:
├── Candidate fetch: 50ms
├── Light ranking: 20ms
├── Heavy ranking: 100ms
├── Diversification: 10ms
└── Total: ~200ms
```

---

## 5. Key Components

### Feed Cache

```
Each user has a feed cache:
├── Stored in Redis/Memcached
├── Contains post IDs (not full posts)
├── ~1000 recent candidate posts
├── TTL: hours to days
└── Invalidated on new follows/unfollows

Feed read:
1. Get candidate IDs from cache
2. Hydrate with full post content (parallel fetch)
3. Rank with ML model
4. Return top N
```

### Social Graph Service

```
Query: "Who does user X follow?"
├── Stored in TAO (distributed graph store)
├── Billions of edges
├── Sub-millisecond lookups
└── Eventually consistent (follow takes seconds to propagate)
```

---

## 6. Trade-offs Meta Makes

| Decision | Meta's Choice | Trade-off |
|----------|---------------|-----------|
| **Consistency** | Eventual | Stale feeds for seconds |
| **Ranking** | ML-based | Complexity, less control |
| **Fanout** | Hybrid push/pull | Complex invalidation |
| **Storage** | In-memory cache | Cost, but < 1s latency |

---

## 7. Interview Application

> "For a social feed system, I'd use Meta's hybrid push-pull approach. For regular users with < 10K followers, push to their followers' feed caches at write time. For celebrities, store the post and pull at read time.
>
> The feed cache stores post IDs, not full content—around 1000 candidates per user in Redis. On read, I'd hydrate posts in parallel and run a ranking model that predicts engagement.
>
> The ranking pipeline has two stages: a light model for the top 1000, then a heavy model for final ranking. This keeps latency under 200ms.
>
> The trade-off is eventual consistency. A new post might take seconds to appear in all followers' feeds. For a social app, that's acceptable."

---

## 8. Senior-Level Phrases

- "I'd use a hybrid fanout model: push for regular users, pull for high-follower accounts."
- "Feed cache stores post IDs, not full content—hydration happens at read time."
- "Two-stage ranking balances accuracy with latency constraints."
- "The social graph is eventually consistent—follows propagate within seconds."

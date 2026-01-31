# 📰 News Feed System (Facebook/Twitter)

## 1. Problem Statement

Design a news feed system like **Facebook** or **Twitter** that shows personalized content to users from people they follow.

---

## 2. Clarifying Questions (Ask First)

* What's the feed type? (Chronological vs ranked/algorithmic)
* What content types? (Text, images, videos, links)
* Real-time or batch? (New posts appear immediately vs periodic refresh)
* Feed size? (Show last 100 posts vs infinite scroll)
* Personalization? (Ranking algorithm, ML recommendations)

---

## 3. Requirements

### Functional

* Generate personalized feed for each user
* Support multiple content types
* Handle new posts in real-time (optional)
* Pagination (infinite scroll)
* Like/comment/share interactions

### Non-Functional

* Low latency (<200ms to generate feed)
* High availability (99.99%)
* Scalable to billions of users
* Handle viral posts (millions of views)

---

## 4. Scale Assumptions

* 1B users, 500M daily active
* 100M posts/day
* Average user follows 200 people
* Average user views feed 10 times/day
* Feed generation: 5B requests/day = ~60K QPS average, 600K QPS peak
* Storage: 100M posts/day × 1KB = 100GB/day, ~36TB/year

---

## 5. High-Level Architecture

```
User → Load Balancer → Feed Service
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
    Write Path      Read Path      Ranking Service
        │               │               │
        ↓               ↓               ↓
    Post DB      Fan-out Service    ML Service
    (NoSQL)      (Push/Pull)       (Ranking)
                        │
                        ↓
                Feed Cache (Redis)
```

---

## 6. Core Components

* **Feed Service**: Generates feeds for users
* **Post Service**: Handles post creation
* **Fan-out Service**: Distributes posts to followers (push model)
* **Ranking Service**: Ranks posts by relevance (ML algorithm)
* **Post Database**: Stores posts (NoSQL, sharded by user_id)
* **Feed Cache**: Stores pre-computed feeds (Redis)
* **Graph Service**: Manages follow relationships
* **Interaction Service**: Handles likes, comments, shares

---

## 7. Data Flow

**Write Path (Publish Post)**
1. User → Creates post
2. Post Service → Store post in DB (post_id, user_id, content, timestamp)
3. Post Service → Get followers from Graph Service (user follows 200 people)
4. Fan-out Service → For each follower, add post_id to their feed
5. Fan-out Service → Update feed cache (Redis) for online followers
6. Post Service → Return success

**Read Path (Pull Model - Simpler)**
1. User → Requests feed
2. Feed Service → Check cache (Redis) for pre-computed feed
3. If cache hit → Return feed (<50ms)
4. If cache miss → Query Post DB for posts from followed users
5. Feed Service → Merge and sort by timestamp
6. Feed Service → Apply ranking algorithm (optional)
7. Feed Service → Store in cache, return to user
8. User → Sees feed (<200ms)

**Read Path (Push Model - Real-time)**
1. User → Requests feed
2. Feed Service → Check feed cache (Redis) - pre-computed during write
3. Feed Service → Return feed (<50ms) - already contains new posts
4. Background → If cache expired, regenerate from Post DB

**Ranking Flow (Algorithmic Feed)**
1. Feed Service → Gets candidate posts (from cache or DB)
2. Ranking Service → Scores each post (ML model: engagement, recency, relevance)
3. Ranking Service → Sorts by score
4. Feed Service → Returns top N posts (e.g., top 100)
5. User → Sees ranked feed

**Interaction Flow (Like/Comment)**
1. User → Likes a post
2. Interaction Service → Update post metadata (like_count++)
3. Interaction Service → Invalidate feed cache (if post is in cached feeds)
4. Interaction Service → Store like in DB (user_id, post_id, timestamp)
5. Real-time → Push update to post owner (optional)

---

## 8. Bottlenecks

* **Fan-out Cost**: 1 post → 200 followers = 200 writes
  * Solution: Use pull model, or hybrid (push for celebrities, pull for others)
* **Feed Generation**: 60K QPS, complex merge/sort
  * Solution: Pre-compute feeds (push model), or cache aggressively
* **Viral Posts**: Single post viewed by millions
  * Solution: Cache viral posts, CDN for content, read replicas
* **Graph Queries**: Getting 200 followers for every post
  * Solution: Cache follow graph, or use pull model (no fan-out)
* **Ranking Computation**: ML ranking is slow
  * Solution: Pre-compute rankings, cache results, or use simpler heuristics
* **Cache Invalidation**: Post deleted, need to remove from all feeds
  * Solution: TTL-based expiration, or lazy removal on read

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Push vs Pull** | Hybrid (push for celebrities, pull for others) | Balances write cost and read latency | More complex, but necessary at scale |
| **Feed Generation** | Pre-compute (push) for active users, on-demand (pull) for others | Fast reads for active users | Higher write cost, but acceptable |
| **Ranking** | Pre-compute rankings, cache results | Fast feed generation | May be slightly stale, but acceptable |
| **Cache Strategy** | Cache full feeds with TTL (5 minutes) | Reduces DB load | May show slightly stale feeds |
| **Database** | NoSQL (Cassandra) sharded by user_id | Horizontal scale for writes | Eventual consistency, but acceptable |

**Key Trade-off:**
> "I use a hybrid push/pull model: push (fan-out) for users with <1000 followers to keep their feeds fresh, and pull (on-demand) for celebrities with millions of followers to avoid expensive fan-out. The trade-off is slightly higher latency for celebrity feeds, but this reduces write costs by 100x for viral posts."

---

## 10. How to Explain This in an Interview

> "I'll design a news feed system using a hybrid push/pull model to balance write costs and read latency.
>
> For architecture, when a user posts, I store the post in a NoSQL database sharded by user_id. For feed generation, I use a hybrid approach: push (fan-out) for regular users and pull (on-demand) for celebrities.
>
> For regular users (followers <1000), when they post, I fan-out to all followers' feed caches in Redis. This pre-computes feeds, so reads are fast (<50ms). For celebrities (followers >1000), I skip fan-out and generate feeds on-demand by querying posts from followed users.
>
> For feed generation, I check the cache first. If cached, I return immediately. If not, I query the Post DB for posts from users they follow, merge and sort by timestamp, apply ranking (if algorithmic), and cache the result.
>
> For ranking, I use a simple scoring algorithm (engagement × recency) and pre-compute scores during write. For more complex ML ranking, I'd use a separate ranking service that scores posts asynchronously.
>
> For caching, I store full feeds in Redis with a 5-minute TTL. This reduces database load significantly. When a user's feed is requested, I return from cache if available, otherwise regenerate.
>
> For viral posts, I cache the post content in CDN and use read replicas to handle the traffic spike. I also use a separate cache for trending posts.
>
> The key trade-off is push vs pull: push gives fast reads but expensive writes for celebrities, pull gives cheap writes but slower reads. The hybrid approach balances both."

**Key Phrases:**
* "Hybrid push/pull model based on follower count"
* "Pre-compute feeds for active users"
* "Cache full feeds with TTL"
* "Ranking algorithm for relevance"

---

## 11. Common Mistakes

* **Push for everyone**: "Fan-out to all followers always"
  * Fix: Use pull for celebrities to avoid expensive fan-out
* **Pull for everyone**: "Always generate on-demand"
  * Fix: Use push for regular users for fast reads
* **No caching**: "Every feed request hits database"
  * Fix: Cache pre-computed feeds in Redis
* **Synchronous ranking**: "We rank during feed generation"
  * Fix: Pre-compute rankings or use async ranking service
* **Not handling viral posts**: "Single post breaks system"
  * Fix: Cache viral posts, use CDN, read replicas
* **Complex queries**: "JOIN posts with users table"
  * Fix: Denormalize data, or use pull model (simple queries)
* **No pagination**: "Load all posts at once"
  * Fix: Paginate with cursor-based pagination (timestamp/post_id)

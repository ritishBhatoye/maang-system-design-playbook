# 📊 Schema Design Patterns (SQL vs. NoSQL)

> **Staff-Signal**: Can you design a schema that optimizes for the 99% query path while avoiding "Hot Partitions" and "N+1 Queries"?

---

## 1. SQL Optimization: The "Clean" Approach
Used when **Data Integrity (ACID)** is non-negotiable (Payments, Accounting).

### Key Patterns:
- **Normalization**: Reducing redundancy. Good for storage efficiency, bad for read latency due to many JOINs.
- **Indexing**: 
    - **Clustered Index**: The physical order of data (Primary Key).
    - **Non-Clustered Index**: a separate structure pointing to the data.
- **Normalization Tradeoff**: In a MAANG interview, if you're building a News Feed, **Normalize is a mistake**. You need to **Denormalize** to avoid 1,000 JOINs per user request.

---

## 2. NoSQL Mastery: The "Query-First" Approach
Used when **Scale and Latency** are more important than complex relational joins (Chat, Netflix, Twitter).

### Pattern A: Denormalization
Instead of joining tables, we store everything needed for a view in a single row.
- **Example (Chat)**: Store the `sender_name` inside the `messages` table.
- **Tradeoff**: Updates are expensive (must update 1M rows if user changes name), but Reads are sub-millisecond.

### Pattern B: The Partition Key Strategy (Cassandra/DynamoDB)
**This is where L5 candidates fail.**
- **Problem**: You shard by `user_id`, but all users in NYC hit the same shard during a local event (Hot Partition).
- **Solution**: **Composite Keys**. 
    - `Partition Key`: `city_id` + `date`.
    - `Sort Key`: `timestamp`.
    - This spreads the load across the cluster even during bursts.

---

## 3. Real-World Example: Twitter Schema

### SQL Version (Normalized - Fails at scale)
```sql
Users (id, name)
Tweets (id, user_id, body, created_at)
Follows (follower_id, followee_id)
```
*Why it fails*: Generating a feed requires a 3-way JOIN across 1B rows.

### NoSQL Version (Wide-Column - MAANG Level)
```text
Table: UserTimeline
Partition Key: user_id
Sort Key: tweet_id
Attributes: {tweet_body, author_name, author_avatar_url}
```
*Why it wins*: To get the feed, we do **1 GET request** for the `user_id`. The data is already sorted and denormalized.

---

## ⚖️ Decision Matrix

| Requirement | Best Schema | Why |
| :--- | :--- | :--- |
| **Complex Relationships** | SQL (Relational) | Joins are native; ACID is built-in. |
| **High Write Volume** | NoSQL (Append-Only) | No lock contention on indices. |
| **Flexible/Dynamic Schema** | NoSQL (Document) | Easier to iterate on features. |
| **Global Scale** | NoSQL (Wide-Column) | Partitioning is a first-class citizen. |

---

## 💬 How to use this in an Interview

> "I’m choosing a **NoSQL denormalized schema** for the User Feed. Instead of joining a 'Likes' table on every read, I will maintain a `like_count` field directly on the `Tweet` object and update it asynchronously. To avoid **Hot Shards** on celebrity tweets, I'll use a **Sharded Partition Key**—appending a random suffix to the ID to spread the writes across multiple physical nodes."

### Follow-up question: "How do you handle 'Stale Data' in a denormalized schema?"
**Correct Answer**: "Denormalization accepts **Eventual Consistency**. If a user updates their profile picture, it might take a few minutes for old posts to reflect the change. If absolute freshness is required for a specific field, I would separate that into a strictly consistent SQL table, but for 99% of social media content, the performance gain of denormalization outweighs the staleness penalty."

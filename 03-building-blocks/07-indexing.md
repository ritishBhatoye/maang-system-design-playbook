# Indexing

## 1. Problem Statement

How do we design indexes to speed up database queries while managing the trade-offs of storage and write performance?

---

## 2. Database Indexes

### What is an Index?
* **Data Structure**: Additional structure that speeds up lookups
* **Trade-off**: Faster reads, slower writes, more storage
* **Analogy**: Book index (find page number quickly)

### Types of Indexes

**B-Tree Index (Most Common)**
* Balanced tree structure
* **Use Case**: Range queries, equality lookups
* **Example**: Primary key, foreign key indexes

**Hash Index**
* Hash table structure
* **Use Case**: Exact match lookups only
* **Example**: In-memory indexes, Redis

**Bitmap Index**
* Bitmap for each value
* **Use Case**: Low cardinality columns (gender, status)
* **Example**: Data warehousing

**Full-Text Index**
* Inverted index for text search
* **Use Case**: Searching within text content
* **Example**: Elasticsearch, PostgreSQL full-text search

---

## 3. Index Design Principles

### When to Index
* **High Read/Write Ratio**: Reads benefit more than writes hurt
* **Frequently Queried Columns**: WHERE, JOIN, ORDER BY
* **Selective Columns**: High cardinality (many unique values)

### When NOT to Index
* **Low Cardinality**: Few unique values (gender, boolean)
* **Frequently Updated**: Every update requires index update
* **Rarely Queried**: Index overhead not worth it
* **Small Tables**: Sequential scan may be faster

---

## 4. Composite Indexes

### Definition
* Index on multiple columns
* **Order Matters**: (A, B) ≠ (B, A)

### Leftmost Prefix Rule
* Index (A, B, C) can be used for:
  * Queries on A
  * Queries on A, B
  * Queries on A, B, C
* **Cannot** be used for:
  * Queries on B only
  * Queries on B, C

### Example
* Index: (user_id, created_at)
* **Works**: WHERE user_id = X, WHERE user_id = X AND created_at > Y
* **Doesn't Work**: WHERE created_at > Y (without user_id)

---

## 5. Index Strategies

### Covering Index
* Index contains all columns needed for query
* **Benefit**: Query can be answered from index alone (no table lookup)
* **Example**: Index (user_id, name, email) for SELECT name, email WHERE user_id = X

### Partial Index
* Index only subset of rows
* **Benefit**: Smaller index, faster
* **Example**: Index WHERE status = 'active' (only index active users)

### Expression Index
* Index on computed expression
* **Example**: Index on LOWER(email) for case-insensitive search

---

## 6. Index Maintenance

### Rebuilding
* Recreate index from scratch
* **When**: After bulk inserts, index fragmentation
* **Trade-off**: Blocks writes, but improves performance

### Updating
* Index updated on every write
* **Cost**: Write amplification (one write → multiple index updates)
* **Trade-off**: Slower writes, faster reads

### Monitoring
* Track index usage
* **Metrics**: Index hit rate, unused indexes
* **Action**: Drop unused indexes, add missing indexes

---

## 7. Distributed Indexing

### Sharded Indexes
* Index partitioned across shards
* **Challenge**: Cross-shard queries are expensive
* **Solution**: Shard by index key (e.g., user_id)

### Replicated Indexes
* Same index on multiple replicas
* **Benefit**: Read scaling
* **Trade-off**: Write amplification

---

## 8. Search Indexes (Elasticsearch)

### Inverted Index
* Term → [Document IDs]
* **Structure**: Term dictionary + Postings list
* **Use Case**: Full-text search

### Index Lifecycle
1. **Indexing**: Build index from documents
2. **Searching**: Query index for terms
3. **Updating**: Re-index or incremental update
4. **Deleting**: Mark as deleted, periodic cleanup

### Sharding Strategy
* Shard by document ID hash
* **Benefit**: Distribute load
* **Challenge**: Cross-shard queries need aggregation

---

## 9. How to Explain This in an Interview

> "For database queries, I create indexes on frequently queried columns, especially those in WHERE clauses and JOINs. I use composite indexes when queries filter on multiple columns, and I order columns by selectivity (most selective first). For full-text search, I use Elasticsearch with inverted indexes. I monitor index usage and remove unused indexes to reduce write overhead."

**Key Points:**
* Index frequently queried columns
* Composite indexes: order matters (leftmost prefix)
* Trade-off: Faster reads, slower writes
* Monitor and optimize indexes

---

## 10. Common Mistakes

* **Over-indexing**: "Index every column"
  * Fix: Index only what's needed, monitor usage
* **Wrong column order**: "Index (created_at, user_id)" for queries on user_id
  * Fix: Order by selectivity and query pattern
* **Ignoring write cost**: "More indexes = better"
  * Fix: Balance read performance vs write cost
* **No composite indexes**: "Separate indexes for each column"
  * Fix: Use composite indexes for multi-column queries
* **Not monitoring**: "Indexes never change"
  * Fix: Monitor usage, drop unused, add missing
* **Indexing low cardinality**: "Index on gender column"
  * Fix: Don't index columns with few unique values

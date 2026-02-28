# 🗄️ Storage Engines (B-Tree vs. LSM Tree)

> **Staff-Level Signal**: Can you pick the right storage engine for the workload, not just the "trending" database name?

---

## 1. The fundamental Tradeoff
Every storage engine is a tradeoff between:
1.  **Write Throughput**
2.  **Read Latency**
3.  **Space Efficiency**

---

## 2. B-Trees (The "Classic")
Used by: **PostgreSQL, MySQL (InnoDB), Oracle.**
- **Structure**: A balanced tree of sorted blocks (pages).
- **Update Logic**: Overwrites data in place (in-place update).
- **Strengths**: 
    - Consistent **O(log N)** read performance.
    - Excellent for range queries.
    - Robust transactional support (locking is easier on fixed branches).
- **Weaknesses**: 
    - **Random I/O**: Writes require seeking to specific pages.
    - Fragmented over time.

---

## 3. LSM Trees (Log-Structured Merge-Tree)
Used by: **Cassandra, RocksDB, ScyllaDB, BigTable.**
- **Structure**: 
    1.  **MemTable**: In-memory buffer (sorted).
    2.  **WAL**: Write-Ahead Log (for durability).
    3.  **SSTables**: Sorted String Tables (immutable files on disk).
- **Update Logic**: Always appends. Never overwrites.
- **Strengths**: 
    - **Sequential Writes**: Massive write throughput (SSD/HDD friendly).
    - Compacts data in background.
- **Weaknesses**: 
    - **Read Amplification**: Might have to check multiple SSTables for one key.
    - **Write Amplification**: Background compaction consumes I/O.

---

## ⚖️ The Comparison Table

| Feature | B-Tree | LSM Tree |
| :--- | :--- | :--- |
| **Primary Workflow** | Read Heavy | Write Heavy |
| **Write Type** | Random I/O (Overwrite) | Sequential I/O (Append) |
| **Read Type** | Single Seek | Multiple Seeks (Bloom Filter helps) |
| **Fragmentation** | High (Requires VACUUM/Defrag) | Low (Handled by Compaction) |
| **Index Type** | Clustered or Secondary | Segmented Sorted Strings |

---

## 🛠️ Critical Concept: SSTables & Bloom Filters
In LSM Trees, since we have many sorted files (SSTables), how do we avoid checking 100 files for a single key?
1.  **SSTables**: Data is sorted within the file, allowing binary search or sparse indexing.
2.  **Bloom Filters**: A probabilistic data structure that tells you "The key is definitely NOT in this file" or "The key MIGHT be in this file." This saves 99% of unnecessary disk seeks.

---

## 💬 How to use this in an Interview

> "For the Ad-Impression tracking service, I’m selecting **Cassandra** because it uses an **LSM Tree** storage engine. Since we are doing 1M+ writes per second, we need the sequential write performance of the LSM MemTable/SSTable architecture. I’m accepting the **Read-Amplification** penalty, which I will mitigate using **Bloom Filters** to keep P99 read latencies under 50ms."

### Follow-up question: "What happens during Compaction?"
**Correct Answer**: "Compaction merges multiple SSTables, removing deleted items and keeping the data sorted. While it improves read performance, it causes **Write Amplification**, which can lead to latency spikes if the compaction strategy (e.g., Size-Tiered vs Leveled) isn't tuned to the hardware's I/O limit."

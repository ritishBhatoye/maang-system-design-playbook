# 🎲 Probabilistic Data Structures (Scale at Low Cost)

> **Staff-Signal**: How do you check if a billion URLs have been crawled using only 1GB of memory?

---

## 1. The Core Idea: Accuracy vs. Space
In massive systems, it is often too expensive to store every distinct ID (Unique Users, Visited URLs). Probabilistic structures allow you to get "Approximate" answers with a tiny memory footprint.

---

## 2. Bloom Filters (The "Membership" Test)
Used for: **Crawl discovery, Database index seeks, Rate limiting.**
- **Problem**: You want to know if `X` is in a set of `1 Billion` items.
- **How it works**: Uses a bit-array and multiple hash functions.
- **Answers**: 
    1.  **"Definitely NOT in the set"** (No False Negatives ✅).
    2.  **"MIGHT be in the set"** (False Positives exist ⚠️).
- **Space**: 1 Billion items can be checked with ~1GB of RAM.

---

## 3. HyperLogLog (The "Cardinality" King)
Used for: **Unique Daily Active Users (DAU), Count of unique views.**
- **Problem**: You want to count unique visitors to a page. A simple `Set` would grow forever.
- **How it works**: Uses the probability of finding a sequence of leading zeros in hashes.
- **Precision**: Typically 98% accurate.
- **Space**: Can count **billions** of unique items using only **12 KB** of memory.

---

## 4. Count-Min Sketch (The "Frequency" Estimator)
Used for: **Top-K Trending Hashtags, Heavy Hitters.**
- **Problem**: What are the most popular tweets in the last hour?
- **How it works**: A 2D array of counters updated by multiple hashes.
- **Behavior**: Always returns an estimate $\ge$ actual count. Over-estimates frequency but never under-estimates.

---

## ⚖️ Comparison Table

| Structure | Best For | Error Type | Memory Usage |
| :--- | :--- | :--- | :--- |
| **Bloom Filter** | `Contains(X)`? | False Positives | Low |
| **HyperLogLog** | `CountDistinct()`? | $\pm$ 2% Error | Extremely Low |
| **Count-Min Sketch** | `Frequency(X)`? | Over-estimation | Low |

---

## 💬 How to use this in an Interview

> "To prevent our web crawler from cycles, I will utilize a **Bloom Filter** for URL de-duplication. This allows us to perform membership tests for billions of URLs in memory. If the filter says 'Maybe', we can afford a rare duplicate crawl, but if it says 'No', we save a high-latency disk seek to the main database."

---

## 🧩 When to use them?
1.  **Caching**: Check if an item is even in the DB before hitting the cache.
2.  **Streaming**: Count unique users in a real-time Kafka stream.
3.  **Analytics**: Quick 'Estimate' dashboards where 100% precision isn't worth 100x the cost.

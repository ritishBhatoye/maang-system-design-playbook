# 🕒 Distributed Time & Clock Synchronization

> **Staff-Signal**: Why can't you just use `System.currentTimeMillis()` to order events in a distributed system?

---

## 1. The Trap: The Illusion of "Now"
In a distributed system, every machine has its own physical clock. These clocks **drift** due to temperature, hardware quality, and load.
- **Clock Skew**: The difference between two clocks at a point in time.
- **Clock Drift**: The rate at which a clock's skew changes.

---

## 2. Physical Clocks (NTP)
**Network Time Protocol (NTP)** attempts to sync clocks over the internet.
- **Problem**: In a data center, skew can still be 10ms - 100ms. If Service A writes at 10:00:01 and Service B writes at 10:00:02, Service B's clock might actually think it's 10:00:00. This breaks **Linearizability**.

---

## 3. Logical Clocks (Ordering without Time)
If we can't agree on *when* something happened, let's agree on the *order* it happened.

### A. Lamport Timestamps
- Every process maintains a counter.
- For every event, increment the counter.
- When sending a message, include the counter.
- **Limitation**: It tells you if A happened before B, but not if they were concurrent.

### B. Vector Clocks
- Each node maintains an array (vector) of counters for all nodes.
- **Strength**: Detects **Conflicting Writes**. If two vectors are incomparable, a "Split-Brain" conflict occurred.
- **Use Case**: Amazon Dynamo, Riak, CouchDB (Conflict Resolution).

---

## 4. Google's TrueTime (The L6 Winner)
Used by **Google Spanner**.
- Instead of returning a single time, it returns an **Interval**: `[earliest, latest]`.
- **Logic**: If I want to commit a write, I wait until `now().earliest > commit_time.latest`.
- **Result**: Guarantees global **External Consistency** (Linearizability) at the cost of a small "Commit Wait" latency (~7ms).

---

## ⚖️ Tradeoffs

| Mechanism | Precision | Consistency | Cost |
| :--- | :--- | :--- | :--- |
| **NTP** | Low (ms) | None (Ordering fails) | Free |
| **Vector Clocks** | N/A | Causal Consistency | Storage overhead on metadata |
| **TrueTime** | High (us) | Strict Serializability | Expensive (Atomic Clocks + GPS) |

---

## 💬 How to use this in an Interview

> "In a multi-region deployment, we cannot rely on physical timestamps for transaction ordering due to **Clock Skew**. To resolve concurrent edits to the same user profile, I would implement **Vector Clocks** to detect causality. For our financial ledger, however, I’d recommend a service with a centralized sequencer or a **TrueTime-like** interval window to ensure strict linearizability across continents."

### Follow-up question: "What is LWW (Last Write Wins) and why is it dangerous?"
**Correct Answer**: "LWW relies on physical timestamps to discard 'older' data. Because of clock drift, a write that actually happened *later* in real-time might be assigned a *smaller* timestamp and be deleted, leading to silent data loss."

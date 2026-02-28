# ⚖️ Consistency Models Deep-Dive

> **Staff-Signal**: Can you differentiate between "Serializability" and "Linearizability" and explain which one you need for a distributed auction system?

---

## 1. Linearizability (Strongest - Single Object)
**Linearizability** is a guarantee about a **single object** (e.g., one register, one row).
- **Behavior**: Once a write is acknowledged, all subsequent reads must return that value or a later one. It makes the system appears as if there is only one copy of the data.
- **Latency Cost**: High. Usually requires a majority quorum (W+R > N).
- **Interviews**: Necessary for **Leader Election** and **Atomic Updates**.

---

## 2. Serializability (Strongest - Multiple Objects)
**Serializability** is a guarantee about **transactions** (groups of objects).
- **Behavior**: The execution of a set of transactions is equivalent to *some* serial (one-by-one) execution.
- **Problem**: It doesn't define *which* order. A serializable execution could theoretically return a value from 1 hour ago if it satisfies the transaction logic.
- **Conclusion**: You usually want **Strict Serializability** (Serializability + Linearizability).

---

## 3. Causal Consistency (L5 Sweet Spot)
- **Behavior**: If process A has communicated to process B that its write is done, then process B’s subsequent write must be ordered after A’s.
- **Analogy**: A comment replying to a post must appear *after* the post.
- **Performance**: High availability; can survive network partitions (unlike Linearizability).

---

## 4. Eventual Consistency (The Default)
- **Behavior**: If no new updates are made, eventually all accesses will return the same value.
- **Analogy**: DNS, Git, News Feeds.
- **Problem**: "Stuck in the past" reads.

---

## 5. CAP Theorem (The Industry Reality)
In a **Partitioned** (P) network, you must choose:
1.  **CP (Consistency)**: The system returns an error if it cannot guarantee strong consistency.
2.  **AP (Availability)**: The system returns a value, even if it might be stale.
3.  **PACELC**: If there is no partition (E), we choose between **Latency (L)** and **Consistency (C)**.

---

## ⚖️ The Hierarchy of Consistency

| Model | Shorthand | Availability | Use Case |
| :--- | :--- | :--- | :--- |
| **Strict Serializability** | "Atomic" | Low | Banking, Consensus |
| **Linearizability** | "Single-object Strong" | Low | Distributed Locks |
| **Causal** | "Cause-Effect" | High | Social Media, Collaborative Editing |
| **Read-Your-Writes** | "Standard UX" | High | User Profiles |
| **Eventual** | "Just Eventually" | High | Analytics, Cache |

---

## 💬 How to use this in an Interview

> "For the payment ledger, I will require **Strict Serializability** to prevent double-spending, even at the cost of higher latency. However, for the 'Like Count' on the social feed, **Eventual Consistency** is sufficient, as achieving strong consistency for a non-critical counter would negatively impact our P99 throughput without providing significant business value."

### Follow-up question: "How does Spanner achieve global consistency without a heavy performance penalty?"
**Correct Answer**: "Spanner utilizes **TrueTime** (GPS and Atomic Clocks) to assign monotonically increasing timestamps with a known confidence interval (uncertainty). By waiting out the uncertainty period before a write is committed, they achieve **External Consistency** (Linearizability) globally."

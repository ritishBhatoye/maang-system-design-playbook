# ⚖️ Exactly-Once vs. At-Least-Once Delivery

> **Staff-Signal**: Why is 'Exactly-Once' impossible in the presence of network partitions, and how do we fake it?

---

## 1. At-Most-Once
- **Logic**: Send message once. Do not retry.
- **Result**: Data loss is possible. Double-delivery is impossible.
- **Use Case**: UDP Video Streaming, high-velocity metric logs (where 1 lost point doesn't matter).

---

## 2. At-Least-Once (The Industry Standard)
- **Logic**: Retransmit the message until an ACK is received.
- **Result**: No data loss. **Double-delivery is likely** (if ACK is lost).
- **Use Case**: Most APIs, Kafka producers by default.
- **Requirement**: The receiver MUST be **Idempotent**.

---

## 3. Exactly-Once (The "Holy Grail")
- **Logic**: Each message is delivered and processed exactly once.
- **Problem**: **The Two Generals Paradox**. It is theoretically impossible for two nodes to agree on a state over an unreliable network with a fixed number of messages.
- **How we "Fake" it**:
    1.  **Distributed Transactions (2PC)**: Very slow, high lock contention.
    2.  **Idempotency + State Management**: The receiver checks if a message with `ID_123` has already been processed. If yes, it ignores the payload but returns `200 OK`.
    3.  **Transactional Outbox**: Writing to the DB and Sending to Kafka in a single local transaction.

---

## ⚖️ Tradeoffs

| Model | Implementation | Cost | Reliability |
| :--- | :--- | :--- | :--- |
| **At-Most** | Lowest latency | Zero | Low |
| **At-Least** | High latency (retries) | Medium | High |
| **Exactly** | High complexity | Highest (Coordination) | Perfect |

---

## 💬 How to use this in an Interview

> "In my payment system design, I recognize that **Exactly-Once** delivery is impossible over an unreliable network. Therefore, I will implement **At-Least-Once** delivery combined with **Server-Side Idempotency**. By using a unique `idempotency_key` for every transaction, the system can safely retry failed network requests without the risk of double-charging the customer."

---

## 🛠️ The Idempotency Recipe
1.  **Client** generates a UUID.
2.  **Server** stores the UUID in Redis with a TTL.
3.  On every request, Server checks if UUID exists.
4.  If it exists, return the **Cached Response**, don't re-process.

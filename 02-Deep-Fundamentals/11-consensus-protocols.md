# 🤝 Consensus Protocols (Raft, Paxos, ZAB)

> **Staff-Level Signal**: Can you explain how a distributed system agrees on a single value when nodes are failing and the network is slow?

---

## 1. Why They Matter (The Interview "Filter")
Candidates often say, "I'll use a distributed lock." When asked how that lock is implemented, 90% fail. Consensus is the foundation of:
- **Leader Election** (Who is the primary?)
- **Distributed Locking** (ZooKeeper/etcd)
- **Global Configuration**
- **Atomic Commits** (Spanner/TiDB)

---

## 2. The Core Problem: The FLP Impossibility
In an asynchronous network, if even one process can fail, it's impossible to guarantee that a set of processes will ever reach consensus.
- **Consequence**: We must choose between **Safety** (never agree on the wrong value) and **Liveness** (eventually agree on a value). Consensus protocols prioritize **Safety**.

---

## 3. Raft (The Modern Choice)
Raft was designed to be more understandable than Paxos. It is used by **etcd** (Kubernetes) and **CockroachDB**.

### 🛠️ The Three Sub-problems:
1.  **Leader Election**: A leader is elected when an existing leader fails.
2.  **Log Replication**: The leader accepts log entries from clients and replicates them across the cluster.
3.  **Safety**: If any server has applied a particular log entry, then no other server can apply a different command for that same log index.

### 🔄 The Election Flow:
1.  Nodes start as **Followers**.
2.  If they don't hear from a Leader, they become **Candidates**.
3.  A Candidate requests votes. If it gets a **Quorum** (N/2 + 1), it becomes the **Leader**.
4.  The Leader sends "Heartbeats" to maintain authority.

---

## 4. Multi-Paxos (The "Classic")
The original protocol by Leslie Lamport. It's mathematically proven but notoriously hard to implement.
- **Key Idea**: A Proposer suggests a value; Acceptors agree to it if they haven't seen a higher proposal number.
- **Interview Insight**: If asked "Why Raft over Paxos?", the answer is **Understandability and Operational Simplicity**. Raft handles leader election as a first-class citizen; in Paxos, it's often an add-on.

---

## 5. ZAB (ZooKeeper Atomic Broadcast)
Specifically used by **Apache ZooKeeper**.
- **Difference**: Optimized for high-throughput broadcast of state changes. It ensures that changes are applied in the exact order they were sent by the leader (Primary Order).

---

## ⚖️ Tradeoffs (The Senior Mindset)

| Feature | Consensus (CP) | Gossip Protocols (AP) |
| :--- | :--- | :--- |
| **Consistency** | Strong (Strict Quorum) | Eventual (Epidemic) |
| **Availability** | Lower (Blocks on Partition) | High (Always Available) |
| **Latency** | Higher (Requires RTT to Quorum) | Extremely Low |
| **Use Case** | Financials, Metadata, Locking | Membership, Error Detection |

---

## 💬 How to use this in an Interview

> "To manage the primary election in my sharded database, I’ll use a consensus-based coordinator like **etcd** which implements the **Raft** protocol. I chose Raft over a simple heartbeat because it ensures **Safety** during a network partition—preventing 'Split-Brain' scenarios where two nodes think they are both the Leader."

### Follow-up question: "What happens if a Quorum cannot be reached?"
**Correct Answer**: "The system sacrifices **Liveness** to maintain **Safety**. It will stop accepting writes until a majority of nodes can communicate again. This prevents data divergence."

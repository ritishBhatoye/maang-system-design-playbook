# 🛡️ High Availability (HA) vs. Disaster Recovery (DR)

> **Staff-Signal**: Most engineers conflate HA and DR. HA is about surviving a **component** failure with 99.99%+ uptime. DR is about surviving a **catastrophic** failure (entire region down) without losing your business.

---

## 1. The Survival Metrics: RTO and RPO
When a disaster hits (e.g., AWS `us-east-1` goes dark), we measure recovery by two numbers:
- **RTO (Recovery Time Objective)**: How *fast* do we need to be back up? (Minutes, Hours, Days?).
- **RPO (Recovery Point Objective)**: How much *data* can we afford to lose? (0 seconds? 1 hour of transactions?).

---

## 2. The DR Spectrum

### A. Backup & Restore (Cold)
- **Strategy**: Nightly backups to S3/Glacier.
- **RTO**: 24h+ (Must provision all infrastructure and restore DB).
- **RPO**: 24h (Data since last backup is lost).
- **Cost**: Lowest.

### B. Pilot Light
- **Strategy**: Core data is replicated to a secondary region. Application servers are **OFF** but configured.
- **RTO**: 10-30m (Boot up instances/autoscaling groups).
- **RPO**: Seconds (Data replication is async).

### C. Warm Standby
- **Strategy**: A "scaled-down" version of the production stack is always running in Region B.
- **RTO**: Minutes (Scale up instances, flip DNS).
- **RPO**: Seconds.

### D. Multi-Site / Active-Active
- **Strategy**: Both Region A and Region B handle traffic simultaneously.
- **RTO**: Zero (Traffic is just re-routed).
- **RPO**: Zero (If sync replication) or ms (If async).
- **Cost**: Highest.

---

## 3. High Availability Patterns
HA is at the infrastructure level, usually within a single region across multiple **Availability Zones (AZs)**.
- **Redundancy**: No Single Point of Failure (N+1 nodes).
- **Health Checks**: Load balancer stops sending traffic to unhealthy nodes.
- **Auto-failover**: Leader election (Raft/Paxos) happens automatically.

---

## ⚖️ Tradeoffs: The "Price of Resilience"

| Strategy | Cost | RTO/RPO | Complexity |
| :--- | :--- | :--- | :--- |
| **Backup** | $ | Days | Low |
| **Pilot Light** | $$ | Minutes | Medium |
| **Active-Active** | $$$$ | Zero | Extremely High |

---

## 💬 How to use this in an Interview

> "My architecture for the global banking ledger follows a **Multi-Site Active-Active** strategy to achieve a near-zero **RPO**. By utilizing synchronous multi-region replication via distributed consensus, we ensure that a total region loss has a negligible **RTO**, as the secondary region is already warm and serving traffic. This significantly exceeds the 99.99% **High Availability** requirement while maintaining the disaster recovery rigor needed for financial data."

### Follow-up question: "How do you handle 'Data Divergence' in Active-Active?"
**Correct Answer**: "We must choose between **Consistency** and **Availability**. In a 'Split-Brain' scenario where regions can't speak, we use **Conflict-free Replicated Data Types (CRDTs)** for non-critical data or **Last Write Wins (LWW)** with vector clocks. For the ledger, we **sacrifice availability** and stop accepting writes until the partition heals."

# 🔄 Geo-Replication (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Geo-replication is the **backbone of global systems**. Interviewers test this to see if you:
- Understand CAP theorem in practice, not just theory
- Can design for data consistency across continents
- Know when to accept eventual consistency vs demand strong consistency
- Think about replication mechanics beyond "just replicate it"

> **Interview Reality**: At Staff level, you're expected to discuss replication lag, conflict resolution, and data locality—not just mention "we'll replicate."

---

## 2. Problem Interviewers Are Testing

- **Data availability**: How do users in Mumbai read data written in Virginia?
- **Consistency vs performance**: Can we read our own writes? Globally?
- **Failure handling**: What happens when cross-region network fails?
- **Data locality**: Where does the data *physically* live? Who controls it?

---

## 3. Key Concepts (Crisp Bullets)

### Replication Modes

| Mode | Latency | Consistency | Durability | Use Case |
|------|---------|-------------|------------|----------|
| **Synchronous** | High (+100-300ms) | Strong | High | Financial transactions |
| **Asynchronous** | Low | Eventual | Moderate | User content, social |
| **Semi-synchronous** | Medium | Read-your-writes | High | E-commerce carts |

### Replication Topologies

- **Single-leader (Primary-Replica)**: One write region, read anywhere
- **Multi-leader (Multi-Primary)**: Write anywhere, resolve conflicts
- **Leaderless (Quorum)**: Read/write to any N nodes, quorum decides

### Key Metrics

- **Replication lag**: Time for write to appear in replica (target: <1s for async)
- **RPO (Recovery Point Objective)**: Max data loss tolerance on failover
- **RTO (Recovery Time Objective)**: Max downtime tolerance

---

## 4. Typical Architecture

### Single-Leader Geo-Replication

```
                 WRITES                    READS
                   │                         │
                   ▼                         ▼
          ┌───────────────┐         ┌───────────────┐
          │  US-EAST-1    │         │  EU-WEST-1    │
          │   PRIMARY     │────────►│   REPLICA     │
          │  (Read/Write) │  async  │  (Read-only)  │
          └───────────────┘         └───────────────┘
                   │                         
                   │ async                   
                   ▼                         
          ┌───────────────┐
          │  AP-SOUTH-1   │
          │   REPLICA     │
          │  (Read-only)  │
          └───────────────┘
```

### Multi-Leader Geo-Replication

```
          ┌───────────────┐     conflict     ┌───────────────┐
          │  US-EAST-1    │◄───resolution───►│  EU-WEST-1    │
          │   PRIMARY     │                  │   PRIMARY     │
          │  (Read/Write) │                  │  (Read/Write) │
          └───────────────┘                  └───────────────┘
                   ▲                                  ▲
                   │         ┌───────────────┐       │
                   └────────►│  AP-SOUTH-1   │◄──────┘
                             │   PRIMARY     │
                             │  (Read/Write) │
                             └───────────────┘
```

---

## 5. Scaling Strategy

### Read Scaling

- Add read replicas in high-traffic regions
- Use connection pooling (PgBouncer, ProxySQL)
- Implement read-your-writes with session affinity

### Write Scaling

- Shard by tenant/user ID across regions
- Use regional write affinity (user's home region)
- Queue writes and batch replicate

### Data Partitioning Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Geographic sharding** | Users in EU → EU shard | Data sovereignty |
| **Hash sharding** | Hash user_id → region | Even distribution |
| **Tenant sharding** | Enterprise per region | B2B SaaS |

---

## 6. Failure Modes

| Failure | Effect | Mitigation |
|---------|--------|------------|
| **Leader failure** | No writes accepted | Automatic failover with leader election |
| **Network partition** | Regions can't sync | Queue writes, reconcile later |
| **Replication lag spike** | Stale reads | Monitor lag, switch to sync if critical |
| **Split-brain** | Two primaries, data divergence | Fencing tokens, consensus protocols |
| **Conflicting writes** | Same record modified in 2 regions | LWW, vector clocks, CRDTs |

### Conflict Resolution Strategies

| Strategy | Description | Trade-off |
|----------|-------------|-----------|
| **Last-write-wins (LWW)** | Latest timestamp wins | May lose data silently |
| **First-write-wins** | Earliest timestamp wins | Later updates ignored |
| **Application-level merge** | Custom conflict handler | Complex but flexible |
| **CRDTs** | Conflict-free data types | Limited data model |

---

## 7. Trade-offs (Explicit Decisions)

### When to Use Synchronous Replication

✅ Financial transactions (no lost money)
✅ Inventory systems (no overselling)
✅ Authentication tokens (security-critical)

❌ Social feeds (stale is OK)
❌ Session data (can regenerate)
❌ Analytics (eventually consistent is fine)

### Consistency Level Decision Matrix

| Scenario | Consistency Level | Replication Mode |
|----------|-------------------|------------------|
| Bank transfer | Strong | Sync |
| Shopping cart | Session | Semi-sync |
| Likes/comments | Eventual | Async |
| User profile | Read-your-writes | Semi-sync |

---

## 8. How to Explain This in an Interview

> "For a global application, I'd use single-leader async replication for simplicity. The primary in US-EAST handles all writes, and we replicate asynchronously to EU and APAC with 100-200ms lag.
>
> For reads, users hit their local replica for low latency. We accept that a user in EU might not immediately see a write from US—eventual consistency with typical lag under 500ms.
>
> The trade-off is that if the primary fails, we might lose up to 500ms of writes (our RPO). For a social application, that's acceptable. For payments, I'd use synchronous replication with a distributed consensus database like Spanner or CockroachDB.
>
> For conflict resolution in multi-master, I'd use last-write-wins for most data and application-level merge for critical business entities like orders."

---

## 9. Common Candidate Mistakes

- **"Replication is automatic"**: Not explaining lag, consistency, or conflict handling
- **Ignoring CAP**: "Strong consistency AND low latency AND partition tolerance" → Impossible
- **No conflict resolution strategy**: "Just write to both" → What happens on conflict?
- **Forgetting read-your-writes**: User writes, refreshes, doesn't see their data
- **Underestimating cross-region latency**: 200ms RTT across Atlantic is physical limit

---

## 10. Senior-Level Interview Phrases

- "I'd accept eventual consistency with sub-second lag for user content because the business impact of stale data is minimal."
- "For read-your-writes guarantee, I'd use session-sticky routing to the region where the write occurred."
- "The conflict resolution strategy depends on data semantics—LWW for timestamps, but CRDTs for counters."
- "Cross-region replication lag is bounded by physics—100-200ms is the floor for transatlantic."
- "I'd monitor replication lag as a key SLI and alert if it exceeds 1 second."

---

## Database-Specific Implementations

### DynamoDB Global Tables
- Multi-master, last-write-wins conflict resolution
- Async replication, typically <1s lag
- Good for: Session data, user profiles

### CockroachDB / Spanner
- Synchronous replication, serializable consistency
- Higher latency (100-300ms writes)
- Good for: Financial data, inventory

### PostgreSQL + Logical Replication
- Primary-replica, async by default
- Can configure synchronous for specific replicas
- Good for: Traditional OLTP with regional reads

### MongoDB Atlas Global Clusters
- Zone sharding for data locality
- Async replication between zones
- Good for: Mixed workloads with regional requirements

---

## References (Interview Prep Only)

- Designing Data-Intensive Applications, Chapter 5: Replication
- Google Spanner paper (external consistency)
- DynamoDB Global Tables whitepaper
- CockroachDB multi-region patterns

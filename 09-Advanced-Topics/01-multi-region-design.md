# 🌍 Multi-Region Design (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Multi-region architecture is a **Staff+ level signal**. Interviewers use it to test whether you can:
- Think beyond single-datacenter solutions
- Reason about latency, consistency, and failure domains
- Make cost-aware architectural decisions
- Handle regulatory and compliance requirements

> **Interview Reality**: Candidates who only design for one region rarely get "Strong Hire" at L5+.

---

## 2. Problem Interviewers Are Testing

They want to see if you understand:
- **Latency optimization**: Users in Tokyo shouldn't wait for a round-trip to Virginia
- **Failure isolation**: A regional outage shouldn't take down global service
- **Data sovereignty**: GDPR, CCPA, data residency laws
- **Cost vs availability trade-offs**: Multi-region isn't free

---

## 3. Key Concepts (Crisp Bullets)

### Why Go Multi-Region?

- **Latency**: Reduce round-trip time (RTT) from 200ms+ to <50ms
- **Availability**: Survive regional outages (AWS us-east-1 fails ~yearly)
- **Compliance**: GDPR requires EU data in EU, China requires data in China
- **Capacity**: Distribute load across regions during peak

### Deployment Models

| Model | Description | RTO/RPO | Cost |
|-------|-------------|---------|------|
| **Active-Passive** | One region serves traffic, other is standby | Minutes/Minutes | $ |
| **Active-Active** | All regions serve traffic simultaneously | Seconds/Near-zero | $$$$ |
| **Pilot Light** | Minimal infrastructure in standby, scale on failover | 15-60 min/Hours | $ |
| **Warm Standby** | Scaled-down replica running in secondary | Minutes/Seconds | $$ |

### Data Replication Patterns

- **Synchronous replication**: Strong consistency, higher latency (2PC/Paxos)
- **Asynchronous replication**: Eventual consistency, lower latency
- **Conflict resolution**: Last-write-wins, vector clocks, CRDTs

---

## 4. Typical Architecture

```
                    ┌─────────────────────────────────────────┐
                    │            Global DNS (Route 53)        │
                    │         Latency-based routing           │
                    └─────────────┬───────────────────────────┘
                                  │
            ┌─────────────────────┼─────────────────────┐
            ▼                     ▼                     ▼
    ┌───────────────┐     ┌───────────────┐     ┌───────────────┐
    │  US-EAST-1    │     │  EU-WEST-1    │     │  AP-SOUTH-1   │
    │  ┌─────────┐  │     │  ┌─────────┐  │     │  ┌─────────┐  │
    │  │   LB    │  │     │  │   LB    │  │     │  │   LB    │  │
    │  └────┬────┘  │     │  └────┬────┘  │     │  └────┬────┘  │
    │       ▼       │     │       ▼       │     │       ▼       │
    │  ┌─────────┐  │     │  ┌─────────┐  │     │  ┌─────────┐  │
    │  │ App Tier│  │     │  │ App Tier│  │     │  │ App Tier│  │
    │  └────┬────┘  │     │  └────┬────┘  │     │  └────┬────┘  │
    │       ▼       │     │       ▼       │     │       ▼       │
    │  ┌─────────┐  │     │  ┌─────────┐  │     │  ┌─────────┐  │
    │  │ DB      │◄─┼─────┼─►│ DB      │◄─┼─────┼─►│ DB      │  │
    │  │(Primary)│  │     │  │(Replica)│  │     │  │(Replica)│  │
    │  └─────────┘  │     │  └─────────┘  │     │  └─────────┘  │
    └───────────────┘     └───────────────┘     └───────────────┘
                    Async Replication (lag: 50-200ms)
```

---

## 5. Scaling Strategy

### Traffic Distribution

- **Latency-based routing**: Route 53 sends users to nearest healthy region
- **Weighted routing**: Shift traffic gradually during deployments
- **Geolocation routing**: Force traffic to specific regions (compliance)

### Data Tier Scaling

- **Read replicas per region**: Reduce cross-region reads
- **Write routing**: Send all writes to primary or use multi-master
- **Cache per region**: Regional Redis clusters for hot data

### Capacity Planning

```
Rule of Thumb:
- Each region should handle 100% traffic (for failover)
- Keep 30% headroom for failover spike
- Factor in async lag during failover (data loss window)
```

---

## 6. Failure Modes

| Failure | Impact | Mitigation |
|---------|--------|------------|
| **Region outage** | All traffic in region lost | DNS failover to healthy regions |
| **Replication lag spike** | Stale reads during failover | Accept or force sync before failover |
| **Split-brain** | Multiple primaries, data divergence | Consensus protocols, fencing tokens |
| **Cross-region partitions** | Regions can't communicate | Operate independently, reconcile later |
| **DNS propagation delay** | Users still hitting failed region | Lower TTL (30-60s), use Global Accelerator |

---

## 7. Trade-offs (Explicit Decisions)

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Consistency** | Sync replication | Async replication | Financial data, inventory |
| **Failover** | Automatic | Manual | When false positives are costly |
| **Write locality** | Single primary | Multi-master | Most use cases (simpler) |
| **Routing** | DNS-based | Anycast/Global Accelerator | DNS is fine until sub-5s failover needed |

**Cost Impact:**

```
Single region:  $10K/month compute + $2K storage
Multi-region:   $25K/month compute + $6K storage + $3K data transfer
                = 2.5x-3x cost for 99.99% availability
```

---

## 8. How to Explain This in an Interview

> "For a global user base, I'd deploy active-active across US, EU, and APAC regions. Users hit their nearest region via latency-based DNS routing.
>
> For the data layer, I'd use async replication from a single primary—we accept 100-200ms replication lag in exchange for lower write latency. For reads, each region has local replicas.
>
> The trade-off is that during failover, we might serve stale data or lose recent writes (RPO of ~1 minute). For most use cases, that's acceptable. For financial transactions, I'd use synchronous replication with a multi-region distributed database like CockroachDB or Spanner."

---

## 9. Common Candidate Mistakes

- **"Just deploy in multiple regions"**: No mention of data strategy or consistency
- **Ignoring replication lag**: "All regions have the same data" → Wrong
- **Forgetting DNS TTL**: Failover takes forever with 5-minute TTLs
- **Not considering cost**: Multi-region can 3x infrastructure costs
- **Active-active without conflict resolution**: "Both regions write to the same table" → Split-brain

---

## 10. Senior-Level Interview Phrases

- "I'd start with active-passive for simplicity, then evolve to active-active as latency requirements tighten."
- "The trade-off is consistency vs latency—async replication gives us low latency writes but we accept stale reads."
- "For failover, I'd set DNS TTL to 30 seconds and use health checks that probe application health, not just TCP."
- "We need to budget for 2-3x infrastructure cost for true multi-region resilience."
- "I'd use CRDTs or last-write-wins for conflict resolution in multi-master setups."

---

## References (Interview Prep Only)

- AWS Multi-Region Disaster Recovery whitepaper
- Google SRE Book - Chapter on Distributed Systems
- DynamoDB Global Tables architecture
- CockroachDB multi-region patterns

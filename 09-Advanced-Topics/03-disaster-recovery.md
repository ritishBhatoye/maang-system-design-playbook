# 🚨 Disaster Recovery (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Disaster Recovery (DR) separates senior engineers from mid-level. Interviewers test this to see if you:
- Understand business continuity beyond "use multiple AZs"
- Can quantify RTO/RPO and map them to business requirements
- Know the cost-availability trade-off curve
- Think about recovery procedures, not just prevention

> **Interview Reality**: Every MAANG company has had major outages. They want engineers who can design systems that survive them.

---

## 2. Problem Interviewers Are Testing

- **Business continuity**: How fast can we recover from total datacenter loss?
- **Data durability**: How much data can we afford to lose?
- **Cost optimization**: What's the right investment for our risk profile?
- **Automation**: Can we recover without manual intervention at 3 AM?

---

## 3. Key Concepts (Crisp Bullets)

### Core Metrics

| Metric | Definition | Example |
|--------|------------|---------|
| **RTO** (Recovery Time Objective) | Max acceptable downtime | 4 hours for internal tools |
| **RPO** (Recovery Point Objective) | Max acceptable data loss | 1 minute for e-commerce |
| **MTTR** (Mean Time to Recovery) | Average actual recovery time | 15 minutes |
| **MTBF** (Mean Time Between Failures) | Average uptime between failures | 30 days |

### DR Strategies (AWS Terminology)

| Strategy | RTO | RPO | Cost | Description |
|----------|-----|-----|------|-------------|
| **Backup & Restore** | Hours | Hours | $ | Cold storage, restore on demand |
| **Pilot Light** | Minutes | Minutes | $$ | Minimal core running, scale on failover |
| **Warm Standby** | Minutes | Seconds | $$$ | Scaled-down full replica |
| **Multi-Site Active/Active** | Near-zero | Near-zero | $$$$ | Full duplicate, active everywhere |

### What Constitutes a Disaster?

- Regional AWS/GCP outage (happens 1-2x per year)
- Datacenter fire/flood/power failure
- Major security breach requiring isolation
- Massive data corruption (software bug)
- DDoS attack overwhelming single region

---

## 4. Typical Architecture

### Pilot Light Strategy

```
                     NORMAL OPERATION
    ┌─────────────────────────────────────────────────────┐
    │                    PRIMARY REGION                    │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐ │
    │  │   LB    │→ │ App x10 │→ │  Cache  │→ │   DB   │ │
    │  └─────────┘  └─────────┘  └─────────┘  └────────┘ │
    │                                              │      │
    └──────────────────────────────────────────────┼──────┘
                                                   │
                            Continuous Replication │
                                                   ▼
    ┌─────────────────────────────────────────────────────┐
    │                    DR REGION (Cold)                  │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐ │
    │  │ LB(off) │  │ App x0  │  │Cache(0) │  │ DB Rep │ │
    │  │         │  │ (ASG=0) │  │         │  │        │ │
    │  └─────────┘  └─────────┘  └─────────┘  └────────┘ │
    └─────────────────────────────────────────────────────┘

                     DURING FAILOVER
    ┌─────────────────────────────────────────────────────┐
    │                    DR REGION (Hot)                   │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐ │
    │  │   LB    │→ │ App x10 │→ │  Cache  │→ │ DB(Pri)│ │
    │  │         │  │ (scaled)│  │(warmed) │  │        │ │
    │  └─────────┘  └─────────┘  └─────────┘  └────────┘ │
    └─────────────────────────────────────────────────────┘
```

### Multi-Site Active-Active

```
              Global Load Balancer / DNS
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
    ┌─────────┐    ┌─────────┐    ┌─────────┐
    │ US-East │    │ EU-West │    │AP-South │
    │  ACTIVE │◄──►│  ACTIVE │◄──►│  ACTIVE │
    │         │    │         │    │         │
    │ App + DB│    │ App + DB│    │ App + DB│
    └─────────┘    └─────────┘    └─────────┘
         ▲              ▲              ▲
         └──────────────┴──────────────┘
              Multi-master Replication
```

---

## 5. Scaling Strategy

### Infrastructure as Code (Critical)

```
Every DR deployment must be:
- Fully automated (Terraform/CloudFormation)
- Version controlled
- Tested monthly (minimum)
- Deployable in < 15 minutes
```

### Data Replication Strategy

| Data Type | Replication Method | RPO Achievable |
|-----------|-------------------|----------------|
| Transactional DB | Synchronous replication | Near-zero |
| Large datasets | Async replication + WAL shipping | Minutes |
| Object storage | Cross-region replication (S3/GCS) | < 15 minutes |
| Configuration | Git + automated deployment | Near-zero |

### Capacity Planning for DR

```
Rule of Thumb:
- DR region capacity = 100% of primary (for full takeover)
- Keep 20% buffer for failover traffic spike
- Pre-provision critical resources (DB, load balancers)
- Use reserved capacity for predictable cost
```

---

## 6. Failure Modes

| Failure | Detection | Recovery |
|---------|-----------|----------|
| **AZ failure** | Health checks fail for zone | Traffic shifts to other AZs |
| **Region failure** | Multi-AZ health checks fail | DNS failover to DR region |
| **Database corruption** | Data validation alerts | Point-in-time recovery |
| **Network partition** | Cross-region ping failures | Operate independently |
| **Cascading failure** | Spike in error rates | Circuit breaker + manual intervention |

### Runbook Essentials

Every DR plan needs:
1. **Detection**: How do we know there's a disaster?
2. **Decision**: Who decides to failover? What's the threshold?
3. **Execution**: Step-by-step automated + manual procedures
4. **Validation**: How do we know recovery succeeded?
5. **Failback**: How do we return to primary?

---

## 7. Trade-offs (Explicit Decisions)

| Decision | Low-Cost Option | High-Availability Option | Deciding Factor |
|----------|-----------------|-------------------------|-----------------|
| **DR strategy** | Backup & Restore | Active-Active | RTO requirement |
| **Replication** | Async (RPO: minutes) | Sync (RPO: 0) | Data criticality |
| **Failover** | Manual | Automated | Team size, risk tolerance |
| **Testing** | Annual | Monthly/Weekly | Regulatory requirements |

### Cost-Availability Curve

```
  Cost │
       │                        ┌── Active-Active
       │                    ┌───┘
       │               ┌────┘
       │           ┌───┘ Warm Standby
       │      ┌────┘
       │  ┌───┘ Pilot Light
       │──┘ Backup & Restore
       └────────────────────────────────────────
                     Availability (9s)
              99%  99.9%  99.99%  99.999%
```

---

## 8. How to Explain This in an Interview

> "For DR strategy, I'd first establish our RTO and RPO requirements with the business. For an e-commerce platform, I'd target RTO of 15 minutes and RPO of 1 minute.
>
> I'd implement a warm standby approach: a scaled-down replica in a secondary region with continuous database replication. All infrastructure is defined in Terraform, so we can scale up the DR environment in minutes.
>
> The failover process would be semi-automated: monitoring detects the outage, pages on-call, and they confirm failover with a single command. This prevents false positives while keeping RTO low.
>
> We'd test this monthly with game days—simulated regional failures where we actually fail over and measure our real RTO/RPO. The trade-off is warm standby costs about 30% of primary infrastructure, but that's justified for our availability requirements."

---

## 9. Common Candidate Mistakes

- **No RTO/RPO defined**: "We'll have DR" → But how fast? How much data loss?
- **Untested DR**: "We have backups" → Have you restored from them under pressure?
- **Manual-only recovery**: "The team will handle it" → At 3 AM? In 15 minutes?
- **Ignoring dependencies**: "App fails over" → What about third-party APIs? DNS?
- **No failback plan**: "We'll figure it out" → How do you get back to primary?

---

## 10. Senior-Level Interview Phrases

- "Our RTO is 15 minutes and RPO is 1 minute based on business impact analysis."
- "I'd use infrastructure as code so DR deployment is identical to primary and testable."
- "We run game days monthly to validate our actual recovery time versus our target."
- "The trade-off between warm standby and active-active is about 2x cost for 10x faster RTO."
- "I'd implement runbook automation so failover is a single command, not a checklist."

---

## Game Day Checklist

A proper DR test includes:

### Pre-Test
- [ ] Announce test window to stakeholders
- [ ] Confirm monitoring is active
- [ ] Verify DR region capacity

### During Test
- [ ] Simulate failure (DNS cutover, region isolation)
- [ ] Measure time to detection
- [ ] Measure time to failover decision
- [ ] Measure time to service restoration
- [ ] Verify data integrity

### Post-Test
- [ ] Document actual RTO/RPO achieved
- [ ] Identify gaps and bottlenecks
- [ ] Update runbooks
- [ ] File tickets for improvements

---

## References (Interview Prep Only)

- AWS Disaster Recovery Whitepaper
- Netflix Chaos Engineering practices
- Google SRE Book - Chapter on Disaster Recovery
- AWS Well-Architected Framework - Reliability Pillar

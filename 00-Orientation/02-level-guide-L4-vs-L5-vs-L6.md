# Level Guide: L4 vs L5 vs L6 vs L7

> **What you say sounds different at each level. Here's how to calibrate.**

---

## L4 (SDE II) - "The Implementer"

### What They Test
- Can you stay structured?
- Can you build a working happy path?
- Do you know the basic building blocks?

### What Good Sounds Like
- "I'd use a load balancer to distribute traffic"
- "I'd cache frequently accessed data in Redis"
- "I'd use a database with indexes for fast queries"

### What Great Sounds Like
- "I'd use a load balancer with round-robin for stateless services, but stickiness for stateful"
- "I'd use Redis with write-through for this read-heavy workload"
- "I'd add a composite index on (user_id, timestamp) for this query pattern"

### Red Flags
- Jumping to solution without clarifying requirements
- No capacity estimation
- Single point of failure in design
- Can't explain what happens when something fails

### Checklist
- [ ] Complete any case study in 40 minutes
- [ ] Draw clean high-level architecture
- [ ] Pick appropriate databases (SQL vs NoSQL)
- [ ] Add basic caching layer
- [ ] Mention failure handling

---

## L5 (Senior SDE) - "The Architect"

### What They Test
- Can you justify every decision with tradeoffs?
- Can you handle follow-up questions?
- Can you quantify (QPS, storage, latency)?

### What Good Sounds Like
- "I'd use Cassandra because we need write-heavy with time-series data, accepting higher read latency as a tradeoff"
- "We need strong consistency here because this is financial data, accepting 50ms additional latency"
- "Cache-aside works for 90% of reads, but for viral content we'd need write-through"

### What Great Sounds Like
- "Let me estimate: at 100K QPS with 100:1 read:write ratio, we need ~10GB cache to hit 95% hit rate"
- "For the celebrity problem in social feeds, I'd use hybrid push-pull: pre-compute for regular users, on-demand for influencers"
- "The partition key should be user_id for locality, but we'd have hot shards for power users - mitigate with virtual nodes"

### Red Flags
- "I use X because it's better" without justification
- Can't handle "what if traffic 10x?"
- No failure mode discussion
- Designs in isolation (no cross-system thinking)

### Checklist
- [ ] All L4 items, plus:
- [ ] Quantify tradeoffs with numbers
- [ ] Handle 3+ follow-up questions
- [ ] Proactive failure analysis (not just "add redundancy")
- [ ] Discuss database sharding with partition key rationale
- [ ] Add monitoring/metrics to any design

---

## L6 (Staff Engineer) - "The Systems Thinker"

### What They Test
- Can you design for global failure?
- Can you think about cost?
- Can you scope and prioritize?

### What Good Sounds Like
- "For V1, I'd build single-region with manual failover. Multi-region is a V2 feature requiring 3x cost"
- "This design costs ~$50K/month. Is that justified by the business value?"
- "We need to consider GDPR - EU user data stays in EU, but we can have a global search index with hashes"

### What Great Sounds Like
- "For active-active multi-region, I'd use conflict-free replicated data types (CRDTs) for counters, last-writer-wins for mutable data"
- "The chaos engineering approach: we'd inject 50% packet loss to validate circuit breakers actually open"
- "This is a 6-month project for a team of 5. We'd ship MVP in 3 months with reduced consistency guarantees"

### Red Flags
- Over-engineering everything (doesn't scope down to MVP)
- No cost discussion
- Designs in vacuum (no team/timeline consideration)
- Can't discuss data privacy/compliance

### Checklist
- [ ] All L5 items, plus:
- [ ] Propose MVP vs V2 scoping
- [ ] Multi-region active-active with conflict resolution
- [ ] Cost analysis per design
- [ ] Cross-team dependency discussion
- [ ] Chaos engineering approach
- [ ] Data residency/compliance considerations

---

## L7 (Principal Engineer) - "The Strategic Leader"

### What They Test
- Can you influence across organizations?
- Can you make technical strategy decisions?
- Can you balance technical excellence with business value?

### What Good Sounds Like
- "The build-vs hinges-buy decision here on 18-month TCO. Custom build is $2M + $200K/year maintenance vs $500K/year SaaS"
- "This requires organizational alignment - the billing team owns the ledger, we'd need to negotiate API contracts"
- "The industry is moving toward event-driven. Our current monolith blocks us from deploying independently. This migration is existential"

### What Great Sounds Like
- "Looking at our traffic patterns, 80% of our engineering cost is on the write path that affects only 5% of users. We should consider a tiered service"
- "I've benchmarked three approaches. The data suggests Option B has 40% lower TCO with acceptable 10% latency increase. Here's the analysis."
- "This isn't just a technical decision - it affects our relationship with regulators. We need to involve Legal before finalizing"

### Red Flags
- Technical decisions without business context
- No consideration of organizational impact
- Can't defend decisions to non-technical stakeholders

### Checklist
- [ ] All L6 items, plus:
- [ ] Business value justification for all major decisions
- [ ] Organizational influence strategy
- [ ] Technical roadmapping (2-3 year view)
- [ ] Cross-company partnership/ecosystem thinking
- [ ] Risk-adjusted decision making

---

## The Key Differences Summary

| Dimension | L4 | L5 | L6 | L7 |
|-----------|----|----|----|----|
| **Requirements** | Lists features | Prioritizes | Scopes MVP | Business alignment |
| **Tradeoffs** | Mentions | Quantifies | Weighs cost | Links to strategy |
| **Failures** | Adds redundancy | Degradation plan | Chaos validation | Organizational impact |
| **Database** | Uses technology | Justifies choice | Sharding strategy | Build vs buy |
| **Scope** | 45 min design | Complete solution | MVP + V2 | Multi-year roadmap |
| **Communication** | Structured | Confident | Check-in often | Influences others |

---

## How to Level Up

### L4 → L5
1. Practice capacity estimation until it's automatic
2. For every decision, say "I chose X because Y, accepting Z tradeoff"
3. Add monitoring to every design

### L5 → L6
1. Scope everything: "For V1, I'd... then V2 adds..."
2. Add cost analysis: "$X/month for Y users"
3. Study multi-region and chaos engineering
4. Practice 5+ follow-up questions per case study

### L6 → L7
1. Think organizationally: "This requires team X collaboration"
2. Consider business value: "The ROI justifies the cost"
3. Think multi-year: "This sets us up for the next platform evolution"
4. Study company-specific patterns (how Google/Meta/Amazon operate)

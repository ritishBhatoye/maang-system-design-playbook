# 🔥❄️ Hot vs Cold Storage (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Storage tiering shows you understand **cost-performance optimization** at scale. Interviewers test this to see if you:
- Can design systems that are both fast AND cost-effective
- Understand access patterns and data lifecycle
- Know when to optimize for latency vs storage cost
- Think about data growth over years, not just days

> **Interview Reality**: At MAANG scale, storage costs can be 40%+ of infrastructure spend. Tiering is not optional.

---

## 2. Problem Interviewers Are Testing

- **Cost optimization**: How do you avoid paying $0.023/GB for data accessed once per year?
- **Performance**: How do you serve hot data with sub-10ms latency?
- **Data lifecycle**: How does data move between tiers?
- **Access pattern awareness**: Can you identify hot vs cold data?

---

## 3. Key Concepts (Crisp Bullets)

### Storage Temperature Tiers

| Tier | Latency | Cost | Access Pattern | Example |
|------|---------|------|----------------|---------|
| **Hot** | <10ms | $$$$ | Frequent (hourly+) | Active user sessions |
| **Warm** | 10-100ms | $$ | Infrequent (daily-weekly) | Recent orders |
| **Cold** | Seconds | $ | Rare (monthly) | Old logs, archives |
| **Glacier/Archive** | Minutes-hours | ¢ | Very rare (yearly) | Compliance data |

### AWS S3 Storage Classes (2026 Pricing Example)

| Class | Price/GB/Month | Retrieval Cost | Min Duration |
|-------|----------------|----------------|--------------|
| S3 Standard | $0.023 | Free | None |
| S3 Infrequent Access | $0.0125 | $0.01/GB | 30 days |
| S3 Glacier Instant | $0.004 | $0.03/GB | 90 days |
| S3 Glacier Deep Archive | $0.00099 | $0.02/GB | 180 days |

### Data Temperature Indicators

- **Access frequency**: Reads/writes per day
- **Recency**: Last access timestamp
- **Predictability**: Can we predict future access?

---

## 4. Typical Architecture

### Multi-Tier Storage Architecture

```
                         ┌─────────────────────────────────┐
                         │         Application Layer        │
                         └───────────────┬─────────────────┘
                                         │
                         ┌───────────────┼───────────────┐
                         ▼               ▼               ▼
                   ┌──────────┐   ┌──────────┐   ┌──────────┐
                   │   HOT    │   │   WARM   │   │   COLD   │
                   │  Redis   │   │PostgreSQL│   │    S3    │
                   │ In-Memory│   │  / DDB   │   │ Glacier  │
                   └────┬─────┘   └─────┬────┘   └────┬─────┘
                        │               │             │
                        └───────────────┼─────────────┘
                                        │
                         ┌──────────────┴──────────────┐
                         │    Data Lifecycle Manager   │
                         │  (Automatic tier migration) │
                         └─────────────────────────────┘
```

### Time-Based Tiering Example

```
Day 0-7:    HOT    → Redis + SSD-backed DB
            │ Access: Real-time user queries
            │
Day 7-30:   WARM   → Standard DB / S3 Standard
            │ Access: Occasional lookups
            │
Day 30-90:  COOL   → S3 Infrequent Access
            │ Access: Monthly reports
            │
Day 90+:    COLD   → S3 Glacier
            │ Access: Compliance, audits
            │
Year 1+:    ARCHIVE → Glacier Deep Archive
              Access: Legal discovery only
```

---

## 5. Scaling Strategy

### Automated Tiering Rules

```python
# Example tiering policy (pseudo-code)
def determine_tier(object):
    last_access = object.last_accessed
    access_count = object.monthly_access_count
    
    if access_count > 1000 or last_access < 1_hour:
        return "HOT"
    elif access_count > 10 or last_access < 7_days:
        return "WARM"
    elif access_count > 0 or last_access < 90_days:
        return "COLD"
    else:
        return "ARCHIVE"
```

### Database Tiering Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Partitioning by date** | Hot partitions on SSD, cold on HDD | Time-series data |
| **Read replicas on cheaper storage** | Primary on SSD, replicas on HDD | Analytics queries |
| **Archive tables** | Move old rows to separate tables | OLTP systems |
| **Hybrid databases** | S3 + local storage (Athena, BigQuery) | Analytics at scale |

### Capacity Calculation

```
Example: 1TB/day data ingestion
├── Hot (7 days):    7TB × $0.10/GB  = $700/month
├── Warm (23 days):  23TB × $0.03/GB = $690/month
├── Cold (60 days):  60TB × $0.01/GB = $600/month
└── Archive (1 year): 365TB × $0.001/GB = $365/month

Without tiering: 455TB × $0.10 = $45,500/month
With tiering:    $2,355/month → 95% cost reduction
```

---

## 6. Failure Modes

| Failure | Impact | Mitigation |
|---------|--------|------------|
| **Premature archival** | Cold data retrieved frequently ($$) | Analyze access patterns before tiering |
| **Restoration delay** | User waits hours for archived data | Offer "restore in progress" UX |
| **Tier transition failure** | Data stuck in wrong tier | Monitoring + retry logic |
| **Metadata loss** | Can't find archived data | Maintain searchable catalog |
| **Compliance deletion** | Deleted data still needed | Legal hold policies |

---

## 7. Trade-offs (Explicit Decisions)

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Tiering granularity** | Per-object | Per-partition | Small objects with varied access |
| **Transition timing** | Age-based | Access-based | Predictable access patterns |
| **Retrieval speed** | Glacier Instant | Deep Archive | Occasional access needed |
| **Indexing** | Full index in hot tier | Sparse cold index | Search frequency justifies cost |

### When NOT to Tier

- Data accessed unpredictably (random old records)
- Small total data size (<100GB)
- Retrieval latency is business critical
- Compliance requires immediate access

---

## 8. How to Explain This in an Interview

> "For a system generating 1TB/day, I'd implement a multi-tier storage architecture. The last 7 days live in hot storage (Redis cache + SSD-backed database) for sub-10ms access. Data from 7-30 days moves to warm storage with standard S3, which is 4x cheaper but still provides millisecond retrieval.
>
> After 30 days, data moves to S3 Glacier Instant Retrieval—10x cheaper than hot storage but retrieval takes seconds. After 1 year, we archive to Glacier Deep Archive for compliance at $0.001/GB/month.
>
> The trade-off is retrieval latency for older data. For a user dashboard, we'd show 'data loading' for archived records rather than making users wait. This tiering reduces storage costs by 90%+ while maintaining fast access for the common case—recent data."

---

## 9. Common Candidate Mistakes

- **Everything in hot storage**: "'We'll use Redis for all data" → Costs explode at scale
- **No retrieval time consideration**: "Archive everything after 7 days" → Users wait hours
- **Missing lifecycle policies**: "We'll manually move data" → Never actually done
- **Ignoring access patterns**: "30 days is cold" → What if invoices are accessed at month-end?
- **No catalog/index**: "We archived it somewhere" → Can't find the data

---

## 10. Senior-Level Interview Phrases

- "I'd tier based on access patterns, not just age—frequently accessed old data stays warm."
- "The cost-performance trade-off here is 10x cheaper storage vs seconds of retrieval latency."
- "We'd need to analyze historical access patterns before setting tiering thresholds."
- "For compliance data, I'd use legal hold combined with S3 Object Lock to prevent deletion."
- "The metadata index remains in hot storage so we can always answer 'does this data exist?'"

---

## Real-World Examples

### E-commerce Order History
- **Hot**: Last 7 days (active orders, returns)
- **Warm**: 7-90 days (customer support queries)
- **Cold**: 90 days - 7 years (occasional disputes)
- **Archive**: 7+ years (tax compliance)

### Video Streaming Platform
- **Hot**: Currently playing segments (CDN + origin)
- **Warm**: Full movie catalog (S3 Standard)
- **Cold**: Rarely watched titles (S3 IA)
- **Archive**: Master files for re-encoding (Glacier)

### Financial Trading System
- **Hot**: Today's trades (in-memory)
- **Warm**: 30 days (SSD database)
- **Cold**: 1 year (regulated storage)
- **Archive**: 7+ years (compliance vault)

---

## Implementation Checklist

- [ ] Analyze access patterns for past 90 days
- [ ] Define tier thresholds based on access frequency
- [ ] Implement lifecycle policies (S3 Lifecycle, custom jobs)
- [ ] Build metadata catalog for cold/archive data
- [ ] Create retrieval API with latency expectations
- [ ] Set up cost monitoring per tier
- [ ] Test retrieval from each tier
- [ ] Document compliance requirements per tier

---

## References (Interview Prep Only)

- AWS S3 Storage Classes deep dive
- Azure Blob Storage tiers
- GCP Cloud Storage classes
- Designing Data-Intensive Applications - Chapter 3

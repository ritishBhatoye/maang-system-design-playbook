# 🚚 Data Migration at Scale (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Data migration shows you can execute **zero-downtime production changes**. Interviewers test this to see if you can plan large-scale migrations with validation and rollback strategies.

---

## 2. Problem Interviewers Are Testing

- **Zero-downtime migrations**: How do you migrate without users noticing?
- **Data consistency**: How do you ensure no data loss?
- **Rollback strategy**: What if the new system fails?

---

## 3. Key Concepts

### Migration Strategies

| Strategy | Downtime | Risk | Best For |
|----------|----------|------|----------|
| **Big Bang** | Hours | High | Small datasets |
| **Trickle Migration** | Zero | Medium | Large datasets |
| **Dual-Write** | Zero | Medium | Database migrations |
| **Strangler Fig** | Zero | Low | Service rewrites |

### Migration Phases

1. **Preparation**: Schema design, tooling, validation scripts
2. **Historical migration**: Move existing data
3. **Dual-write/sync**: Capture new writes in both systems
4. **Cutover**: Switch reads to new system
5. **Validation**: Verify data integrity
6. **Cleanup**: Remove old system

---

## 4. Typical Architecture

```
DUAL-WRITE PATTERN:

    Application (writes to both)
           │         │
           ▼         ▼
      [OLD DB]   [NEW DB]
           │         │
           └────┬────┘
                │
         [Validator Service]
```

---

## 5. Scaling Strategy

- **Parallel workers**: 100 workers = 100K records/sec
- **Batching**: 10K records per batch
- **Throttling**: Limit during peak hours, max during off-peak

---

## 6. Failure Modes

| Failure | Mitigation |
|---------|------------|
| **Data corruption** | Checksums, validation |
| **Missing records** | Reconciliation reports |
| **Duplicate records** | Idempotent writes |
| **Performance impact** | Adaptive throttling |

---

## 7. Trade-offs

- **Speed vs Impact**: Fast migration = higher production load
- **Consistency**: Strong (locks) vs Eventual (dual-write)
- **Validation**: Full (100%) vs Sampled (1%)

---

## 8. Interview Explanation

> "I'd use CDC (Change Data Capture) to stream changes to Kafka while running historical migration in batches. Once caught up, switch reads to new system while maintaining dual-writes. After a week of validation, stop writes to old system."

---

## 9. Common Mistakes

- No validation strategy
- Underestimating scale
- No rollback plan
- Ignoring schema differences

---

## 10. Senior-Level Phrases

- "I'd use CDC to capture ongoing mutations during historical migration."
- "We'd validate data integrity with shadow queries against both systems."
- "The rollback plan is maintained until 7 days of validated production traffic."

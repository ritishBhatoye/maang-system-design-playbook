# 📊 Scaling Analytics Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Analytics systems have **fundamentally different access patterns** than OLTP. Interviewers test this to see if you can design for large scans, aggregations, and batch processing.

---

## 2. Problem Interviewers Are Testing

- Do you understand OLAP vs OLTP differences?
- Can you design for analytical queries (aggregations, scans)?
- Do you know when to use columnar storage?

---

## 3. Key Concepts

### OLTP vs OLAP

| Characteristic | OLTP | OLAP |
|----------------|------|------|
| **Query pattern** | Point lookups | Full scans, aggregations |
| **Data size** | Row at a time | Billions of rows |
| **Latency** | < 10ms | Seconds to minutes |
| **Freshness** | Real-time | Minutes to hours delayed |
| **Storage** | Row-oriented | Column-oriented |

### Analytics Query Characteristics

```
Typical OLAP query:
SELECT date, country, SUM(revenue), COUNT(DISTINCT user_id)
FROM events
WHERE date BETWEEN '2024-01-01' AND '2024-12-31'
GROUP BY date, country
ORDER BY revenue DESC

Scans: 10 billion rows
Returns: 1,000 rows
Time: 10-60 seconds
```

---

## 4. Storage Patterns

### Columnar Storage

```
Row Storage (OLTP):
┌────────┬──────┬─────────┬────────┐
│   ID   │ Name │ Country │Revenue │
├────────┼──────┼─────────┼────────┤
│   1    │ John │   US    │  100   │
│   2    │ Jane │   UK    │  200   │
└────────┴──────┴─────────┴────────┘

Columnar Storage (OLAP):
┌────────┐  ┌──────┐  ┌─────────┐  ┌────────┐
│   ID   │  │ Name │  │ Country │  │Revenue │
│   1    │  │ John │  │   US    │  │  100   │
│   2    │  │ Jane │  │   UK    │  │  200   │
└────────┘  └──────┘  └─────────┘  └────────┘

Benefit: Read only columns you need
SUM(revenue) → Only scan Revenue column
```

### Partitioning for Analytics

```
Events table partitioned by date:

events/
├── year=2024/
│   ├── month=01/
│   │   ├── day=01/
│   │   ├── day=02/
│   │   └── ...
│   └── month=02/
└── year=2025/

WHERE date = '2024-01-15'
→ Only scans 1 partition (1/365th of data)
```

---

## 5. Architecture Pattern

```
                     Data Sources
                          │
                          ▼
                   ┌─────────────┐
                   │   Kafka     │
                   │ (streaming) │
                   └──────┬──────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
        ┌──────────┐ ┌─────────┐ ┌──────────┐
        │   Flink  │ │  Spark  │ │  S3/GCS  │
        │(real-time)│ │ (batch) │ │  (lake)  │
        └────┬─────┘ └────┬────┘ └────┬─────┘
             │            │           │
             └────────────┼───────────┘
                          ▼
                   ┌─────────────┐
                   │ Data        │
                   │ Warehouse   │
                   │ (Snowflake) │
                   └──────┬──────┘
                          │
                          ▼
                   ┌─────────────┐
                   │ BI / Query  │
                   │   Tools     │
                   └─────────────┘
```

---

## 6. Aggregation Strategies

### Pre-Aggregation (Rollups)

```
Raw events: 10 billion rows
├── Per-minute rollup: 525,600 rows/year
├── Per-hour rollup: 8,760 rows/year
└── Daily rollup: 365 rows/year

Query "daily revenue for 2024":
├── Without rollup: Scan 10B rows → 60 seconds
└── With rollup: Scan 365 rows → 10ms
```

### Materialized Views

```sql
-- Pre-compute common aggregations
CREATE MATERIALIZED VIEW daily_revenue AS
SELECT 
    date,
    country,
    SUM(revenue) as total_revenue,
    COUNT(DISTINCT user_id) as unique_users
FROM events
GROUP BY date, country;

-- Refresh periodically
REFRESH MATERIALIZED VIEW daily_revenue;
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Freshness** | Real-time | Batch (hourly) | Alerting, monitoring |
| **Storage** | Columnar | Row-based | Analytical queries |
| **Query speed** | Pre-aggregated | Raw data | Known query patterns |
| **Cost** | On-demand (Athena) | Provisioned (Redshift) | Variable query volume |

---

## 8. Interview Explanation

> "For an analytics system processing 10 billion events per day, I'd design a Lambda architecture combining batch and stream processing.
>
> Events flow through Kafka into two paths: Flink for real-time aggregations (last 1 hour) and Spark batch jobs for historical data loaded into a data warehouse like Snowflake.
>
> For storage, columnar format (Parquet) partitioned by date. A query for 'revenue by country for 2024' only scans the relevant columns and partitions—reducing I/O by 100x compared to row storage.
>
> For common queries, I'd pre-aggregate with materialized views: daily rollups, weekly rollups. The dashboard query hits the 365-row rollup table instead of 10 billion raw events.
>
> The trade-off is freshness. Batch data is 1 hour delayed, real-time data is within seconds but less complete. We'd use real-time for monitoring/alerting and batch for reporting."

---

## 9. Common Mistakes

- **OLTP database for analytics**: "Just use PostgreSQL" → Scans too slow
- **No partitioning**: "One big table" → Full scan every query
- **Always real-time**: "Stream everything" → Expensive, complex
- **Ignoring pre-aggregation**: "Query raw data" → Slow dashboards

---

## 10. Senior-Level Phrases

- "Columnar storage reduces I/O by only scanning needed columns."
- "Date partitioning with predicate pushdown eliminates 99% of scans."
- "Pre-aggregated rollups trade storage for query performance."
- "Lambda architecture balances real-time freshness with batch completeness."

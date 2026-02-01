# 🔍 How Google Designs Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Understanding Google's architectural philosophy helps you think like a **Staff+ engineer**. Google invented many foundational distributed systems concepts.

---

## 2. Google's Core Principles

### 1. Design for Scale from Day One

```
Google's mindset:
├── Billions of queries per day
├── Petabytes of data
├── Millions of servers
└── Global, always-on

"If it doesn't scale to 1000x, we need a different design."
```

### 2. Embrace Distributed Systems

```
Core assumption: Machines fail
├── Single server MTBF: ~3 years
├── At 1M servers: ~1 server fails per minute
└── Design for failure, not prevention
```

### 3. Data-Driven Everything

```
Every decision backed by data:
├── A/B testing at massive scale
├── Metrics, metrics, metrics
└── SLOs drive engineering priorities
```

---

## 3. Key Architectural Patterns

### Bigtable Pattern (Wide-Column Store)

```
Schema-less, sorted key-value:

Row Key          | Column Family: info | Column Family: metrics
-----------------+--------------------+------------------------
com.google.www   | content: <html>... | visits: 1M, rank: 1
com.youtube.www  | content: <html>... | visits: 500M, rank: 2

Use case: When you need sorted scans and column families
```

### Spanner Pattern (Globally Distributed SQL)

```
Strong consistency across regions via TrueTime:

Region US → Timestamp T1 (atomic clock)
Region EU → Timestamp T2 (atomic clock)
└── TrueTime bounds uncertainty to < 7ms

Use case: Global transactions, financial data
```

### MapReduce → Dataflow (Stream + Batch)

```
Unified model for batch and streaming:

Pipeline:
├── Read from source (batch or streaming)
├── Transform (map, filter, aggregate)
├── Group by key
├── Window (time-based for streaming)
└── Write to sink

Use case: ETL, analytics, ML feature pipelines
```

---

## 4. Google's Infrastructure Stack

```
┌─────────────────────────────────────────────────────────┐
│                    Applications                          │
│              (Gmail, Search, YouTube)                    │
├─────────────────────────────────────────────────────────┤
│                   Serverless/Managed                     │
│          (Cloud Functions, Cloud Run, GKE)               │
├─────────────────────────────────────────────────────────┤
│                     Data Layer                           │
│     (Bigtable, Spanner, BigQuery, Firestore)            │
├─────────────────────────────────────────────────────────┤
│                   Networking/Infra                       │
│        (Borg, Colossus, Stubby/gRPC, Chubby)            │
├─────────────────────────────────────────────────────────┤
│                     Hardware                             │
│      (Custom servers, TPUs, Global fiber network)        │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Key Systems to Know

### Borg (Container Orchestration)

```
Kubernetes' ancestor:
├── Declarative job specification
├── Resource bin-packing
├── Automatic scheduling and rescheduling
└── Cell-based deployment (cluster of 10K machines)
```

### Chubby (Distributed Lock Service)

```
Coordination primitive:
├── Distributed locking
├── Leader election
├── Configuration storage
└── Service discovery

Open-source equivalent: ZooKeeper
```

### Colossus (Distributed File System)

```
GFS v2:
├── Exabyte-scale storage
├── Automatic replication
├── Single namespace across data centers
└── Optimized for large sequential I/O
```

---

## 6. Trade-offs Google Makes

| Decision | Google's Choice | Trade-off Accepted |
|----------|-----------------|-------------------|
| **Consistency** | Strong (Spanner) | Higher latency |
| **Storage** | Custom (Bigtable) | Operational complexity |
| **Compute** | Preemptible VMs | Jobs can be killed |
| **Networking** | Private backbone | Massive infrastructure cost |

---

## 7. Interview Application

> "Google's approach emphasizes horizontal scaling with commodity hardware, accepting failures as normal. For a global system, I'd consider the Spanner pattern: use TrueTime-like mechanisms for global consensus if strong consistency is required.
>
> For high-throughput data, the Bigtable model of sorted key-value with column families works well—similar to HBase or Cassandra.
>
> I'd design with SLOs in mind: define the error budget, then architect to meet it rather than over-engineering for zero failures."

---

## 8. Senior-Level Phrases

- "I'd apply the Bigtable model: sorted keys with column families for flexible schemas."
- "Spanner's TrueTime gives strong consistency globally at the cost of 10-20ms latency."
- "Like Borg, I'd use declarative specifications and let the scheduler handle placement."
- "Following SRE principles, I'd define an error budget before architecting."

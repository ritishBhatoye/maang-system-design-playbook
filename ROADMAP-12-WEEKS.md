# 🗺️ 12-Week MAANG System Design Roadmap (2026 Edition)

> **"If you have 1 hour a day, this is how you crack L5/L6 at Google/Meta/Amazon."**

This roadmap is designed by a Senior Staff Engineer to take you from "knowing the terms" to "designing exabyte-scale systems."

---

## 🟢 Part 1: Foundations (Weeks 1-3)
*Focus: Mastering the 'Why'. Do not skip this. Most L5 rejections happen here.*

### Week 1: The Core Patterns
- **Topics**: Vertical vs Horizontal scaling, Load Balancing (L4 vs L7), Consistent Hashing.
- **Goal**: Explain why round-robin LB is a bad idea for stateful services.
- **Checklist**: Read `02-fundamentals/02-scalability-basics.md`.

### Week 2: Storage Engines & Databases
- **Topics**: B-Trees vs LSM-Trees, SQL vs NoSQL (The real tradeoff), Indexing strategies.
- **Goal**: Choose between Postgres and Cassandra based on write-amplification.
- **Checklist**: Read `03-building-blocks/02-databases-sql-nosql.md`.

### Week 3: Distributed Theory (The L5+ Barrier)
- **Topics**: CAP Theorem (The PIE version), PACELC, Consensus (Raft/Paxos).
- **Goal**: Explain how Spanner achieves "Strong Consistency" globally.
- **Checklist**: Read `02-fundamentals/04-consistency-patterns.md`.

---

## 🟡 Part 2: Building Blocks & API (Weeks 4-6)
*Focus: Designing the interface and the data flow.*

### Week 4: Caching & Message Queues
- **Topics**: Cache-aside vs Write-through, Redis architecture, Kafka partitioning.
- **Goal**: Design a system that handles 1M QPS using only 10% of the DB capacity.
- **Checklist**: Read `03-building-blocks/03-caches.md`.

### Week 5: API Design & Data Modeling
- **Topics**: REST vs gRPC, Protobuf, Schema design for NoSQL.
- **Goal**: Design an API that is backward-compatible for 3 years.
- **Checklist**: Create 3 schemas for your favorite app.

### Week 6: Networking & Security
- **Topics**: HTTP/2 vs HTTP/3, DNS (Anycast/Latency-based), OAuth2 flows.
- **Goal**: Explain how a CDN resolves "Cold Start" latency for global users.

---

## 🟠 Part 3: Architecture Case Studies (Weeks 7-9)
*Focus: Putting it all together. 2 Case studies per week.*

### Week 7: Scaling Communication
- **Projects**: Design a URL Shortener, Design a Notification System.
- **Focus**: Efficiency and durability.

### Week 8: Scaling Social & Media
- **Projects**: Design Twitter/X, Design YouTube.
- **Focus**: Fan-out, Storage optimization, and Ingestion pipelines.

### Week 9: Scaling "Hard" Systems
- **Projects**: Design Uber (Real-time GIS), Design a Payment System (Distributed Transactions).
- **Focus**: High availability and Data integrity.

---

## 🔴 Part 4: Staff-Level & Mocking (Weeks 10-12)
*Focus: Failure, Post-mortems, and Delivery.*

### Week 10: Advanced Failure Modes
- **Topics**: Multi-region Active-Active, Chaos Engineering, Load Shedding.
- **Goal**: Design a system that can lose an entire AWS Region and heal in < 60s.

### Week 11: Real-World Failures & Post-mortems
- **Topics**: Cache Stampede, Thundering Herd, Cascading Failures.
- **Goal**: Walk through a Meta outage and explain the "Fix" on a whiteboard.

### Week 12: Mock Interview Marathon
- **Goal**: Do 3 mocks on Pramp or with a colleague. 
- **Checklist**: Use the `PRACTICE-CHECKLIST.md` to grade yourself.

---

## 🏆 The "Final Boss" Challenge
**Design a system that ranks 1B ads per second across 3 continents with <50ms P99 latency.** 
*If you can answer this without sweating, you are ready for Staff (L6).*

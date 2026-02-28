# ✅ System Design Proficiency Checklist (L4 vs L5 vs L6)

> **"A Senior Engineer builds the correct system. A Staff Engineer builds the correct system that survives the wrong conditions."**

Use this checklist to self-evaluate. If you can't check a box, you aren't ready for that level.

---

## 🟢 L4 (SDE II) — "The Implementer"

### 1. Basics
- [ ] Can explain the difference between vertical and horizontal scaling
- [ ] Can describe what a Load Balancer does (L4 vs L7)
- [ ] Can explain why caching improves performance
- [ ] Knows the difference between SQL and NoSQL databases
- [ ] Can draw a basic 3-tier architecture

### 2. Communication
- [ ] Can stay structured during a 45-minute interview
- [ ] Asks clarifying questions before diving into solution
- [ ] Can explain their design while drawing
- [ ] Can handle simple follow-up questions

### 3. Case Studies (Easy)
- [ ] **URL Shortener**: Can design basic create + redirect flow
- [ ] **Rate Limiter**: Can explain token bucket concept
- [ ] **Cache**: Can describe cache-aside pattern

### 4. Technical Foundation
- [ ] Can estimate simple QPS (requests per second)
- [ ] Understands HTTP methods (GET, POST, PUT, DELETE)
- [ ] Knows basic database indexing concepts
- [ ] Can explain what ACID means

### Red Flags for L4
- No structure or framework
- Doesn't ask questions
- Can't draw basic architecture
- Single point of failure everywhere

## 🟦 Senior SDE (L5) — "The Architect"

### 1. Fundamentals
- [ ] **Capacity Math**: Can calculate QPS, Storage, and Bandwidth (back-of-envelope) in < 3 minutes.
- [ ] **DB Choice**: Can justify NoSQL vs SQL based on access patterns (not just "it scales").
- [ ] **Replication**: Can explain the difference between Synchronous and Asynchronous replication lag.
- [ ] **Consistent Hashing**: Can draw it on a whiteboard and explain how it prevents hot spots during re-sharding.

### 2. Case Study Mastery
- [ ] **URL Shortener**: Can design a collision-free ID generator (Snowflake/Ticket Service).
- [ ] **Rate Limiter**: Can implement Token Bucket vs Sliding Window in a distributed Redis environment.
- [ ] **News Feed**: Can explain Hybrid Push/Pull (The Celebrity Problem).
- [ ] **Notification System**: Can design a multi-channel retry mechanism with idempotency.

### 3. Tradeoffs & Decisiveness
- [ ] **CAP Theorem**: Can explain why "Strong Consistency" is expensive in a global system.
- [ ] **Cache Patterns**: Can decide between Cache-Aside and Write-Through for an E-commerce inventory.
- [ ] **Availability**: Can explain 99.9% vs 99.99% in terms of "downtime per year."

---

## 🟪 Staff Engineer (L6) — "The Systems Thinker"

### 1. Failures & Resilience (Critical)
- [ ] **Cascading Failures**: Can design a "Circuit Breaker" to prevent a slow service from taking down the API Gateway.
- [ ] **Load Shedding**: Can implement priority-based queuing (e.g., drop analytics, keep payments).
- [ ] **Retry Storms**: Can explain why "Exponential Backoff with Jitter" is mandatory for client-side retries.
- [ ] **Chaos Engineering**: Can explain how to inject faults (e.g., simulate 50% packet loss) to validate the design.

### 2. Global Complexity
- [ ] **Multi-Region**: Can design an "Active-Active" architecture for a global payment gateway.
- [ ] **Conflict Resolution**: Can explain **Vector Clocks** or **CRDTs** for resolving "Split-Brain" edits.
- [ ] **Data Residency**: Can architect a system that satisfies GDPR (EU data stays in EU) while keeping a global search index.

### 3. Operational Depth
- [ ] **Observability**: Can design a "Distributed Tracing" strategy for a microservice mesh.
- [ ] **Cost Modeling**: Can justify a $1M/month cloud bill vs a $500K/month design with higher latency.
- [ ] **Migration**: Can explain a "Strangler Fig" migration from a legacy Monolith to Microservices with zero downtime.

---

## 🔴 The "No-Hire" Red Flags (All Levels)
*If you do these, you will be rejected.*

1. **Jumping to Solution**: Drawing a Load Balancer 3 minutes into the interview. (L4-L6)
2. **Ignoring Scale**: Designing a Chat system that only works for 1,000 users. (L4-L6)
3. **Hand-wavy Tradeoffs**: "I use Redis because it's fast." (L5-L6)
4. **No Failure Modes**: Assuming the network and the database are always up. (L5-L6)
5. **Passive Interviewing**: Waiting for the interviewer to tell you what to do next. (L6)

---

## 📈 Scorecards (Staff Level Evaluation)

| **Hire Signal** | **"Strong Hire" (L6)** | **"Lean Hire" (L5)** |
|:---:|:---:|:---:|
| **Scoping** | Defines what *not* to build. | Lists all features. |
| **Failures** | Proactively addresses "Double-Failure" scenarios. | Only mentions single-node failure. |
| **Communication** | Controls the room; checks in every 10 mins. | Follows the interviewer's lead. |
| **Tradeoffs** | Uses "If we do X, we lose Y, which is okay because..." | Uses "Let's use X because it's better." |
| **Production** | Mentions Canary rollouts/Post-mortems. | Only mentions the code/architecture. |

# Scalability Basics

## 1. Problem Statement

How do we design systems that handle growth without breaking?

---

## 2. Clarifying Questions

* What type of growth? Users, data, or traffic?
* Vertical vs horizontal scaling constraints?
* Budget considerations?

---

## 3. Requirements

### Functional

* System must handle increasing load
* Maintain performance under growth
* Support geographic expansion

### Non-Functional

* Linear cost scaling
* No single point of failure
* Graceful degradation

---

## 4. Scale Assumptions

* Start: 1K users, 10K QPS
* Growth: 10M users, 100K QPS
* Data: TBs to PBs

---

## 5. High-Level Architecture

**Vertical Scaling (Scale Up)**
```
Single Server → Bigger Server
```

**Horizontal Scaling (Scale Out)**
```
Single Server → Multiple Servers → Load Balancer
```

---

## 6. Core Components

* **Stateless Services**: Enable horizontal scaling
* **Load Balancers**: Distribute traffic
* **Database Sharding**: Split data across nodes
* **Caching**: Reduce load on backend
* **CDN**: Offload static content

---

## 7. Data Flow

**Stateless Request Flow**
1. Client → Load Balancer
2. Load Balancer → Any Available Server
3. Server processes (no session state)
4. Response returned

**Stateful Request Flow** (Problematic)
1. Client → Server A (session stored)
2. Next request → Server B (session missing)
3. **Problem**: Sticky sessions or shared state needed

---

## 8. Bottlenecks

* **Database**: Single DB becomes bottleneck
  * Solution: Read replicas, sharding, caching
* **Stateful Services**: Can't scale horizontally
  * Solution: Externalize state (Redis, DB)
* **Synchronous Operations**: Blocking calls
  * Solution: Async processing, queues
* **Hot Keys**: Uneven load distribution
  * Solution: Consistent hashing, replication

---

## 9. Trade-offs

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Vertical Scaling** | Simple, no code changes | Limited by hardware, expensive | Small scale, stateful apps |
| **Horizontal Scaling** | Unlimited scale, cost-effective | Requires stateless design | Production systems |
| **Database Sharding** | Scales writes | Complex queries, rebalancing | High write volume |
| **Read Replicas** | Scales reads | Eventual consistency | Read-heavy workloads |

---

## 10. How to Explain This in an Interview

> "I design stateless services from day one because horizontal scaling is non-negotiable at scale. I accept eventual consistency in read replicas to trade off strong consistency for read scalability. I shard by user_id to distribute load evenly and avoid hot partitions."

**Key Points:**
* Always mention statelessness first
* Justify consistency trade-offs explicitly
* Explain sharding strategy (not just "we shard")

---

## 11. Common Mistakes

* **Designing stateful services**: "We store sessions in memory"
  * Fix: Use external session store (Redis)
* **Ignoring database bottlenecks**: "We'll add more app servers"
  * Fix: Address DB scaling first
* **Over-engineering early**: "We need 10 microservices"
  * Fix: Start simple, scale when needed
* **Not considering hot keys**: "We hash randomly"
  * Fix: Use consistent hashing, consider access patterns

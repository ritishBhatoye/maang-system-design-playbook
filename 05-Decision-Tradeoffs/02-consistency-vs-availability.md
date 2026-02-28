# Consistency vs Availability

## 1. Problem Statement

How do we balance data consistency and system availability when network partitions occur, and what does CAP theorem really mean?

---

## 2. CAP Theorem

### The Three Properties

**Consistency (C)**
* All nodes see the same data at the same time
* **Strong Consistency**: Reads always return latest write
* **Example**: Financial transactions, leader election

**Availability (A)**
* System remains operational and responds to requests
* **High Availability**: System responds even if some nodes fail
* **Example**: Social media feeds, content delivery

**Partition Tolerance (P)**
* System continues operating despite network failures
* **Required**: In distributed systems, partitions will happen
* **Reality**: You can't avoid partitions

---

## 3. The CAP Trade-off

### You Can Only Guarantee 2 of 3

**CP (Consistency + Partition Tolerance)**
* **Choose**: Consistency over availability
* **Behavior**: Blocks during partitions to maintain consistency
* **Example**: Financial systems, distributed databases (MongoDB, HBase)

**AP (Availability + Partition Tolerance)**
* **Choose**: Availability over consistency
* **Behavior**: Continues operating, may serve stale data
* **Example**: DNS, Cassandra, DynamoDB

**CA (Consistency + Availability)**
* **Not Possible**: In distributed systems (partitions will happen)
* **Only Works**: Single-node systems (not distributed)

---

## 4. Real-World Implications

### CP Systems
* **Behavior**: During partition, system may reject writes/reads
* **Use Case**: When correctness is critical
* **Example**: Payment processing, leader election
* **Trade-off**: System may be unavailable during partitions

### AP Systems
* **Behavior**: During partition, system continues, may serve stale data
* **Use Case**: When availability is critical
* **Example**: Social feeds, content delivery, DNS
* **Trade-off**: Users may see inconsistent data temporarily

---

## 5. Consistency Models

### Strong Consistency
* **Definition**: All reads see latest write
* **How**: Synchronous replication, read from primary
* **Trade-off**: Higher latency, lower availability
* **Use Case**: Financial transactions, critical data

### Eventual Consistency
* **Definition**: System will become consistent eventually
* **How**: Asynchronous replication
* **Trade-off**: May serve stale data, but high availability
* **Use Case**: Social feeds, user profiles, counters

### Weak Consistency
* **Definition**: No guarantees about when consistency is achieved
* **How**: Best-effort replication
* **Trade-off**: Fastest, but least guarantees
* **Use Case**: Non-critical data, analytics

---

## 6. Availability Levels

### 99% (Two Nines)
* **Downtime**: 3.65 days/year
* **Use Case**: Non-critical systems
* **Not Acceptable**: For most production systems

### 99.9% (Three Nines)
* **Downtime**: 8.76 hours/year
* **Use Case**: Internal tools, non-critical services
* **Acceptable**: For some systems

### 99.99% (Four Nines)
* **Downtime**: 52.56 minutes/year
* **Use Case**: Most production systems
* **Target**: For user-facing services

### 99.999% (Five Nines)
* **Downtime**: 5.26 minutes/year
* **Use Case**: Critical infrastructure
* **Expensive**: Requires significant investment

---

## 7. Choosing the Right Model

### Choose CP (Consistency) When:
* **Correctness Critical**: Financial transactions, leader election
* **Can Accept Downtime**: Brief unavailability is acceptable
* **Strong Consistency Needed**: All reads must see latest write

### Choose AP (Availability) When:
* **Uptime Critical**: System must always respond
* **Staleness Acceptable**: Users can tolerate slightly stale data
* **High Scale**: Need to scale horizontally

### Hybrid Approach
* **Different Models for Different Data**:
  * Critical data: CP (strong consistency)
  * Non-critical data: AP (eventual consistency)
* **Example**: User balance (CP), user feed (AP)

---

## 8. Practical Strategies

### Read Your Writes
* **Problem**: User writes, then reads stale data
* **Solution**: Route user's reads to primary, or use session affinity
* **Trade-off**: Slightly more complex, but better UX

### Monotonic Reads
* **Problem**: User sees data go backwards in time
* **Solution**: Always read from same replica, or use versioning
* **Trade-off**: May read from slower replica

### Causal Consistency
* **Problem**: Causally related events appear out of order
* **Solution**: Vector clocks, or read from same replica
* **Trade-off**: More complex, but maintains causality

---

## 9. How to Explain This in an Interview

> "According to CAP theorem, in a distributed system, I can only guarantee 2 of 3: consistency, availability, or partition tolerance. Since partitions are inevitable, I must choose between CP or AP. For financial transactions, I choose CP - I prioritize consistency even if it means the system may be unavailable during partitions. For social feeds, I choose AP - I prioritize availability and accept that users may see slightly stale data. The key is choosing the right model for each use case, and sometimes using different models for different data within the same system."

**Key Points:**
* CAP theorem: Choose 2 of 3 (P is required)
* CP for correctness-critical, AP for availability-critical
* Different models for different data
* Acknowledge trade-offs explicitly

---

## 10. Common Mistakes

* **Trying to have both C and A**: "We have strong consistency and high availability"
  * Fix: During partitions, you must choose - acknowledge this
* **Ignoring partitions**: "Partitions never happen"
  * Fix: Partitions are inevitable in distributed systems
* **Same model for everything**: "Everything is eventually consistent"
  * Fix: Use different models for different data types
* **Not explaining trade-off**: "We use AP"
  * Fix: Explain why and what you're giving up
* **Over-engineering**: "We need five nines for everything"
  * Fix: Match availability to business needs

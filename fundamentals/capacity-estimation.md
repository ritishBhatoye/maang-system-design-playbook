# Capacity Estimation

## 1. Problem Statement

How do we estimate storage, bandwidth, and compute requirements before building the system?

---

## 2. Clarifying Questions

* What's the expected user base?
* What's the growth projection?
* What's the data retention policy?
* Peak vs average traffic?

---

## 3. Requirements

### Functional

* Estimate storage needs
* Estimate bandwidth requirements
* Estimate compute resources
* Plan for growth

### Non-Functional

* 20% headroom for spikes
* Support 3-5x growth
* Cost-effective provisioning

---

## 4. Scale Assumptions

* 100M daily active users
* 10% peak traffic multiplier
* 5 years data retention
* 50% growth year-over-year

---

## 5. High-Level Architecture

**Estimation Framework**
```
Users → Operations per User → Data per Operation → Total Storage
     → Requests per User → Peak QPS → Compute Needs
     → Data Transfer → Bandwidth Needs
```

---

## 6. Core Components

* **Back-of-envelope Math**: Quick estimates
* **Traffic Patterns**: Peak vs average
* **Data Growth**: Retention + growth rate
* **Replication Factor**: 3x for redundancy
* **Compression**: 2-5x reduction

---

## 7. Data Flow

**Storage Estimation**
1. Define data model (size per record)
2. Calculate records per user
3. Multiply by user count
4. Add replication factor (3x)
5. Add growth projection (5 years)
6. Add 20% headroom

**Bandwidth Estimation**
1. Calculate requests per second
2. Average request size (request + response)
3. Multiply: QPS × size
4. Add peak multiplier (10x)
5. Account for replication traffic

**Compute Estimation**
1. Calculate QPS
2. Estimate processing time per request
3. Calculate required CPU: QPS × time
4. Add overhead (30-50%)
5. Plan for redundancy (2-3x)

---

## 8. Bottlenecks

* **Under-provisioning**: System can't handle load
  * Solution: Always add 20-30% headroom
* **Ignoring peak traffic**: "Average is fine"
  * Solution: Design for 10x peak multiplier
* **Forgetting replication**: "We need 1TB"
  * Solution: Multiply by replication factor (3x)
* **Not accounting for growth**: "Current size only"
  * Solution: Project 3-5 years ahead

---

## 9. Trade-offs

| Approach | Accuracy | Speed | When to Use |
|----------|----------|-------|-------------|
| **Back-of-envelope** | Low-Medium | Fast | Initial design, interviews |
| **Detailed Modeling** | High | Slow | Production planning |
| **Over-provisioning** | Safe | Expensive | Early stage, unknown patterns |
| **Right-sizing** | Optimal | Requires data | Mature systems |

**Common Multipliers:**
* Peak traffic: 10x average
* Replication: 3x (primary + 2 replicas)
* Growth: 50-100% YoY
* Headroom: 20-30%

---

## 10. How to Explain This in an Interview

> "For 100M users posting 5 photos/day, that's 500M photos/day. At 2MB per photo, that's 1PB/day raw. With 3x replication and 5-year retention, we need ~5.5PB storage. For bandwidth, at 150K read QPS with 2MB responses, that's 300GB/s peak, or ~2.4Tbps. I'd provision 3Tbps with CDN to handle edge traffic."

**Key Numbers to Remember:**
* 1M = 10^6
* 1B = 10^9
* 1KB = 10^3 bytes
* 1MB = 10^6 bytes
* 1GB = 10^9 bytes
* 1TB = 10^12 bytes
* 1PB = 10^15 bytes

---

## 11. Common Mistakes

* **Wrong units**: "100M users × 1KB = 100MB"
  * Fix: 100M × 1KB = 100GB
* **Forgetting replication**: "We need 1PB"
  * Fix: 1PB × 3 (replication) = 3PB
* **Ignoring peak**: "Average QPS is 10K"
  * Fix: Design for 100K peak QPS
* **No growth projection**: "Current size is X"
  * Fix: Project 3-5 years: X × (1.5)^years
* **Not rounding**: "Exactly 1,234,567.89"
  * Fix: Round to significant digits: "~1.2M"

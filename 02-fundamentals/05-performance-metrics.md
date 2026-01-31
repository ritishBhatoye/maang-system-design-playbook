# Performance Metrics

## 1. Problem Statement

How do we measure and communicate system performance in a way that matters for both users and business?

---

## 2. Core Metrics

### Latency

**Definition**: Time from request sent to response received

**Key Percentiles:**
* **p50 (Median)**: 50% of requests faster than this
* **p95**: 95% of requests faster than this (most important)
* **p99**: 99% of requests faster than this (tail latency)
* **p99.9**: 99.9% of requests faster than this (worst case)

**Why p95/p99 Matter:**
* p50 can hide tail latency issues
* Users remember slow requests
* p95/p99 reveal real user experience

**Example:**
* p50: 50ms (good)
* p95: 200ms (acceptable)
* p99: 500ms (concerning)
* p99.9: 2s (problem)

---

## 3. Throughput

**Definition**: Requests processed per second (QPS/RPS)

**Key Metrics:**
* **Peak QPS**: Maximum requests/second
* **Average QPS**: Average over time period
* **Sustained QPS**: Can handle continuously

**Relationship with Latency:**
* Higher throughput often increases latency
* Need to balance both
* Measure both under load

---

## 4. Availability

**Definition**: Percentage of time system is operational

**Common SLAs:**
* **99%**: 3.65 days downtime/year (unacceptable for most)
* **99.9%**: 8.76 hours downtime/year (three nines)
* **99.99%**: 52.56 minutes downtime/year (four nines)
* **99.999%**: 5.26 minutes downtime/year (five nines)

**Calculation:**
* Availability = (Total Time - Downtime) / Total Time × 100%

---

## 5. Error Rate

**Definition**: Percentage of requests that fail

**Key Metrics:**
* **4xx Errors**: Client errors (bad requests)
* **5xx Errors**: Server errors (system failures)
* **Timeout Rate**: Requests that exceed timeout

**Target:**
* < 0.1% error rate for most systems
* < 0.01% for critical systems

---

## 6. Capacity Metrics

### Storage
* **Total Capacity**: Maximum data stored
* **Growth Rate**: Data added per day/month
* **Retention**: How long data is kept

### Bandwidth
* **Ingress**: Data coming in (uploads)
* **Egress**: Data going out (downloads)
* **Peak vs Average**: Handle spikes

### Compute
* **CPU Utilization**: Percentage of CPU used
* **Memory Usage**: RAM consumption
* **Connection Count**: Active connections

---

## 7. How to Explain This in an Interview

> "I measure performance using p95 latency because it represents what most users experience. For our system, I target p95 < 200ms for user-facing requests. I also track throughput (QPS) to ensure we can handle peak load. For availability, I design for 99.99% (four nines), which means less than an hour of downtime per year."

**Key Metrics to Mention:**
* p95 latency (not just average)
* Peak QPS (not just average)
* Availability target (99.9% or 99.99%)
* Error rate target (<0.1%)

---

## 8. Common Mistakes

* **Only measuring average**: "Average latency is 50ms"
  * Fix: Always measure p95/p99 (tail latency matters)
* **Ignoring error rate**: "System is fast"
  * Fix: Track error rate separately
* **Not measuring under load**: "Works fine in testing"
  * Fix: Load test to find real bottlenecks
* **Confusing availability with reliability**: "99% uptime is good"
  * Fix: 99% = 3.65 days downtime/year (not good)
* **Not setting SLAs**: "We'll make it fast"
  * Fix: Define specific targets (p95 < 200ms)

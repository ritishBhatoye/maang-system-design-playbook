# Load Balancing at Scale

## 1. Problem Statement

Design a load balancing system that can handle millions of requests per second across thousands of servers with high availability and low latency.

---

## 2. Clarifying Questions (Ask First)

* What's the traffic pattern? (HTTP, TCP, WebSocket)
* What's the scale? (QPS, number of servers)
* Do we need session affinity? (Sticky sessions)
* What's the geographic distribution? (Global vs regional)
* What's the health check strategy? (Active vs passive)

---

## 3. Requirements

### Functional

* Distribute requests evenly across servers
* Handle server failures gracefully
* Support multiple protocols (HTTP, TCP, WebSocket)
* Maintain session affinity (if needed)
* Route based on health status

### Non-Functional

* Low latency (<1ms overhead)
* High availability (99.99%)
* Handle millions of requests/second
* Horizontal scalability
* Real-time health monitoring

---

## 4. Scale Assumptions

* 10M requests/second peak
* 10,000 backend servers
* Global distribution (5+ regions)
* <0.1% server failure rate
* Health check every 5 seconds
* Session affinity for 20% of traffic

---

## 5. High-Level Architecture

```
Clients → DNS (Geo-routing) → Regional Load Balancers
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
            L4 Load Balancer  L7 Load Balancer  Global Load Balancer
                    │         │         │
                    ↓         ↓         ↓
            [Server Pool 1] [Server Pool 2] [Server Pool N]
```

---

## 6. Core Components

* **DNS Load Balancing**: Geographic routing, first-level distribution
* **L4 Load Balancer**: Transport layer (IP/port), fast, simple
* **L7 Load Balancer**: Application layer (HTTP), more features, slower
* **Health Checker**: Monitors server health, removes unhealthy servers
* **Session Affinity**: Sticky sessions (if needed)
* **Global Load Balancer**: Routes across regions

---

## 7. Data Flow

### Request Flow
1. Client → DNS query → Returns IP of nearest regional LB
2. Client → Request to regional LB
3. LB → Health check (is server healthy?)
4. LB → Select server using algorithm (round-robin, least connections, etc.)
5. LB → Forward request to selected server
6. Server → Process request, return response
7. LB → Forward response to client

### Health Check Flow
1. LB → Active health check (HTTP/TCP ping) every 5 seconds
2. Server → Responds with health status
3. LB → Updates server status (healthy/unhealthy)
4. If unhealthy → Remove from pool, stop routing traffic
5. If healthy again → Add back to pool after grace period

### Session Affinity Flow
1. First request → LB selects server, stores mapping (IP hash or cookie)
2. Subsequent requests → LB routes to same server based on mapping
3. If server fails → Route to different server, client reconnects

---

## 8. Load Balancing Algorithms

### Round Robin
* **How**: Distribute requests sequentially
* **Pros**: Simple, even distribution
* **Cons**: Ignores server load, capacity
* **Use Case**: Servers have equal capacity

### Least Connections
* **How**: Route to server with fewest active connections
* **Pros**: Balances active load
* **Cons**: Requires connection tracking
* **Use Case**: Long-lived connections, variable processing time

### Weighted Round Robin
* **How**: Round robin with weights (based on server capacity)
* **Pros**: Handles different server capacities
* **Cons**: Static weights, doesn't adapt to load
* **Use Case**: Servers with different specs

### IP Hash
* **How**: Hash client IP to determine server
* **Pros**: Session affinity, deterministic
* **Cons**: Uneven if IPs are clustered
* **Use Case**: Need sticky sessions

### Least Response Time
* **How**: Route to server with lowest response time
* **Pros**: Routes to fastest server
* **Cons**: Requires latency measurement overhead
* **Use Case**: Latency-sensitive applications

---

## 9. Multi-Layer Load Balancing

### Layer 1: DNS (Geographic)
* Route to nearest region
* **Benefit**: Reduces latency, distributes global load
* **Trade-off**: DNS caching can delay failover

### Layer 2: Global Load Balancer
* Routes across regions
* **Benefit**: Handles regional failures
* **Trade-off**: Additional hop, but necessary for HA

### Layer 3: Regional Load Balancer (L4/L7)
* Routes within region
* **L4**: Fast, simple, IP/port based
* **L7**: Slower, but path-based routing, SSL termination

---

## 10. High Availability

### Multiple Load Balancers
* **Active-Active**: All LBs handle traffic
* **Active-Passive**: One active, others standby
* **Benefit**: No single point of failure

### Health Checks
* **Active**: LB pings servers
* **Passive**: Monitor server responses
* **Frequency**: Balance between responsiveness and overhead

### Failover
* **Automatic**: Remove unhealthy servers
* **Graceful**: Drain connections before removing
* **Recovery**: Add back after health check passes

---

## 11. Bottlenecks

* **Single Load Balancer**: Becomes bottleneck
  * Solution: Multiple LBs, horizontal scaling
* **Health Check Overhead**: Too frequent checks
  * Solution: Tune frequency (5-10s), use passive checks
* **Session Affinity**: Uneven load if sessions are long
  * Solution: Use consistent hashing, or prefer stateless services
* **L7 Performance**: Slower than L4
  * Solution: Use L4 when possible, L7 only when needed
* **DNS Caching**: Slow failover
  * Solution: Low TTL (60s), health-based DNS

---

## 12. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **L4 vs L7** | L4 for performance, L7 for features | L4 is faster, L7 enables path routing | L7 adds latency but more flexible |
| **Algorithm** | Least connections for variable load | Balances active connections | Requires connection tracking |
| **Health Checks** | Active every 5s | Balance responsiveness and overhead | More frequent = more overhead |
| **Session Affinity** | IP hash for sticky sessions | Simple, deterministic | Uneven if IPs clustered |
| **Multiple LBs** | Active-active | No single point of failure | More complex, but necessary |

**Key Trade-off:**
> "I use L4 load balancers for performance-critical paths because they add <1ms overhead. For paths that need routing by URL or SSL termination, I use L7 even though it adds 2-3ms latency. The trade-off is performance vs features, and I choose based on the use case."

---

## 13. How to Explain This in an Interview

> "For load balancing at scale, I use a multi-layer approach: DNS for geographic routing, global load balancers for cross-region routing, and regional L4/L7 load balancers for within-region distribution.
>
> I choose L4 load balancers for most traffic because they're fast (<1ms overhead) and simple. I use L7 only when I need path-based routing or SSL termination. For the algorithm, I use least connections because our requests have variable processing time, so this balances active load better than round-robin.
>
> For high availability, I deploy multiple load balancers in active-active mode with health checks every 5 seconds. If a server fails, I remove it from the pool immediately and add it back after it recovers.
>
> For session affinity, I use IP hash for the 20% of traffic that needs it, but I prefer stateless services to avoid this complexity.
>
> The key trade-off is L4 vs L7: L4 is faster but less flexible, L7 is slower but enables more features. I choose based on the specific needs of each path."

**Key Points:**
* Multi-layer load balancing (DNS → Global → Regional)
* L4 for performance, L7 for features
* Least connections algorithm for variable load
* Active-active LBs for high availability

---

## 14. Common Mistakes

* **Single load balancer**: "We have one LB"
  * Fix: Multiple LBs for HA, or use managed service (AWS ALB)
* **Wrong algorithm**: "Round-robin for everything"
  * Fix: Choose based on use case (least connections for variable load)
* **No health checks**: "Servers just handle traffic"
  * Fix: Always implement health checks, remove unhealthy servers
* **L7 for everything**: "We use L7 for all traffic"
  * Fix: Use L4 when possible, L7 only when needed
* **Ignoring LB capacity**: "LB handles infinite traffic"
  * Fix: Plan for LB scaling or use cloud-managed LB
* **DNS with long TTL**: "DNS TTL is 1 hour"
  * Fix: Use low TTL (60s) for faster failover

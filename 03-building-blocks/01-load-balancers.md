# Load Balancers

## 1. Problem Statement

How do we distribute incoming traffic across multiple servers to handle scale and ensure availability?

---

## 2. Clarifying Questions

* What's the traffic pattern? (HTTP, TCP, WebSocket)
* Do we need session affinity?
* What's the health check strategy?
* Geographic distribution needed?

---

## 3. Requirements

### Functional

* Distribute requests evenly
* Handle server failures
* Support multiple protocols
* Maintain session affinity (if needed)

### Non-Functional

* Low latency (<1ms overhead)
* High availability (99.99%)
* Horizontal scalability
* Health monitoring

---

## 4. Scale Assumptions

* 100K requests/second
* 100 backend servers
* <1% failure rate
* Global traffic distribution

---

## 5. High-Level Architecture

```
Clients → Load Balancer → [Server 1, Server 2, ..., Server N]
         (Health Checks)
```

**Types:**
- **L4 (Transport)**: Routes by IP/port
- **L7 (Application)**: Routes by HTTP headers/path

---

## 6. Core Components

* **Load Balancer**: Distributes traffic
* **Health Checker**: Monitors server health
* **Routing Algorithm**: How to select server
* **Session Affinity**: Sticky sessions (if needed)
* **SSL Termination**: Handle TLS at LB

---

## 7. Data Flow

**Request Flow**
1. Client → Load Balancer (with request)
2. LB → Health check (is server healthy?)
3. LB → Select server (using algorithm)
4. LB → Forward request to server
5. Server → Process request
6. Server → Response to LB
7. LB → Response to client

**Health Check Flow**
1. LB → Ping server (every 5-10s)
2. Server → Respond (healthy/unhealthy)
3. LB → Update server status
4. If unhealthy → Remove from pool
5. If healthy again → Add back to pool

---

## 8. Bottlenecks

* **Single LB**: Becomes single point of failure
  * Solution: Multiple LBs with DNS round-robin
* **Session Affinity**: Uneven load if sessions are long
  * Solution: Use consistent hashing, or stateless services
* **Health Check Overhead**: Too frequent checks
  * Solution: Tune check interval (5-10s is typical)
* **LB Capacity**: LB can't handle traffic
  * Solution: Scale LB horizontally, or use cloud LB (AWS ALB)

---

## 9. Trade-offs

| Algorithm | Pros | Cons | When to Use |
|-----------|------|------|-------------|
| **Round Robin** | Simple, even distribution | Ignores server load | Equal capacity servers |
| **Least Connections** | Balances active load | Requires connection tracking | Long-lived connections |
| **Weighted Round Robin** | Handles different capacities | Static weights | Servers with different specs |
| **IP Hash** | Session affinity | Uneven if IPs clustered | Need sticky sessions |
| **Least Response Time** | Routes to fastest server | Requires latency measurement | Latency-sensitive apps |

**L4 vs L7:**
- **L4 (TCP)**: Faster, simpler, but can't route by path
- **L7 (HTTP)**: More flexible, can route by path/headers, but higher latency

---

## 10. How to Explain This in an Interview

> "I use a Layer 7 load balancer (like AWS ALB) because I need path-based routing and SSL termination. I choose least connections algorithm because our requests have variable processing time, so this balances active load better than round-robin. For high availability, I deploy multiple LBs across zones with DNS round-robin."

**Key Points:**
* Always mention L4 vs L7 choice and why
* Explain routing algorithm choice
* Mention health checks
* Address high availability (multiple LBs)

---

## 11. Common Mistakes

* **Single LB**: "We have one load balancer"
  * Fix: Multiple LBs for HA, or use managed service
* **Wrong algorithm**: "We use round-robin for everything"
  * Fix: Choose based on use case (least connections for variable load)
* **No health checks**: "Servers just handle traffic"
  * Fix: Always implement health checks
* **Session affinity everywhere**: "All requests are sticky"
  * Fix: Only use if needed, prefer stateless services
* **Ignoring LB capacity**: "LB handles infinite traffic"
  * Fix: Plan for LB scaling or use cloud-managed LB

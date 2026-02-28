# Bulkhead Pattern

> **Isolate failures to prevent total system collapse.**

---

## 1. The Problem

```mermaid
graph TB
    Client --> PoolA[Pool A: Critical]
    Client --> PoolB[Pool B: Analytics]
    PoolA --> S1[Service]
    PoolB --> S1
    style S1 fill:#f96
```

One misbehaving service consumes all connections, affecting everything.

---

## 2. The Bulkhead Solution

```mermaid
graph TB
    Client --> P1[Pool A: 10 connections]
    Client --> P2[Pool B: 100 connections]
    Client --> P3[Pool C: 5 connections]
    P1 --> S1[Critical Service]
    P2 --> S2[Analytics Service]
    P3 --> S3[External API]
```

**Isolation Levels**:
- **Connection Pool**: Separate pools per service
- **Thread Pool**: Separate threads per service
- **Process**: Separate processes
- **Region**: Separate deployments

---

## 3. Implementation

### Connection Pool Bulkhead

```python
# Each service has its own pool
user_pool = ConnectionPool(max_connections=10)
order_pool = ConnectionPool(max_connections=50)
analytics_pool = ConnectionPool(max_connections=100)

async def get_user(user_id):
    async with user_pool.connection() as conn:
        return await conn.query("SELECT * FROM users...")

async def get_orders(order_id):
    async with order_pool.connection() as conn:
        return await conn.query("SELECT * FROM orders...")
```

### Thread Pool Bulkhead

```java
// Separate thread pools in Java
@Bulkhead(value = "userService", fallingBack = "fallback")
public User getUser(String id) {
    return userClient.get(id);
}

@Bulkhead(value = "orderService", fallingBack = "fallback")
public Order getOrder(String id) {
    return orderClient.get(id);
}
```

---

## 4. Comparison with Circuit Breaker

| Aspect | Bulkhead | Circuit Breaker |
|--------|----------|-----------------|
| **Purpose** | Isolate resources | Stop failing calls |
| **Trigger** | Pre-allocation | Failure threshold |
| **Failure** | One pool saturates | All calls fail fast |
| **Together** | Yes - use both | Yes - use both |

---

## 5. Design Patterns

### Thread Pool Isolation

```mermaid
graph TB
    subgraph "Request Thread"
        RT[Main Thread]
    end
    subgraph "Bulkhead Pools"
        TP1[User Pool: 5 threads]
        TP2[Order Pool: 10 threads]
        TP3[Payment Pool: 3 threads]
    end
    RT --> TP1
    RT --> TP2
    RT --> TP3
```

### Process Isolation

```mermaid
graph TB
    Client --> LB[Load Balancer]
    LB --> P1[Process 1: Users]
    LB --> P2[Process 2: Orders]
    LB --> P3[Process 3: Payments]
    P1 --> DB1[(User DB)]
    P2 --> DB2[(Order DB)]
    P3 --> DB3[(Payment DB)]
```

---

## 6. Interview Narrative

> "Bulkheads isolate resources so one failing service doesn't consume everything. We'd have separate connection pools: 10 connections for critical payment service, 50 for analytics. If analytics misbehaves, it hits its own limit and doesn't starve payments. Combined with circuit breakers, we get defense in depth - bulkheads prevent resource exhaustion, circuit breakers stop calling failing services."

---

## 7. Follow-up Questions

1. **How do you determine pool sizes?**
   - Measure actual usage under load
   - Reserve capacity for critical paths
   - Monitor for saturation

2. **What's the cost of bulkheads?**
   - More resources (pools, threads)
   - Potential underutilization
   - Complexity in management

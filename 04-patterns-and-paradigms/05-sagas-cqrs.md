# Sagas and CQRS

## 1. Problem Statement

How do we handle distributed transactions and separate read/write models in microservices architectures?

---

## 2. The Problem with Distributed Transactions

### ACID Transactions
* Work well in single database
* **Problem**: Don't work across multiple services/databases
* **Why**: Network partitions, service failures, latency

### Two-Phase Commit (2PC)
* Coordinates transactions across services
* **Problem**: Blocks on failures, poor performance
* **Result**: Not used in modern distributed systems

---

## 3. Saga Pattern

### Concept
* **Long-running transaction** broken into smaller transactions
* Each transaction has **compensating action** (rollback)
* **No ACID**: Eventual consistency instead

### Types

**Choreography (Event-Driven)**
* Services publish events
* Other services react and publish their own events
* **Pros**: Decoupled, no coordinator
* **Cons**: Hard to track, complex flow

**Orchestration (Coordinator)**
* Central orchestrator coordinates steps
* **Pros**: Easier to track, simpler flow
* **Cons**: Single point of failure, coupling

---

## 4. Saga Example: E-Commerce Order

### Steps
1. **Create Order** → Order service
2. **Reserve Inventory** → Inventory service
3. **Process Payment** → Payment service
4. **Ship Order** → Shipping service

### Compensating Actions
* If payment fails → Cancel inventory reservation
* If shipping fails → Refund payment, release inventory

### Flow (Orchestration)
```
Orchestrator → Create Order → Reserve Inventory → Process Payment
                ↓ (if fails)
              Compensate: Cancel Order
```

---

## 5. Saga Trade-offs

### Pros
* **Works Across Services**: No distributed locks
* **Better Performance**: No blocking
* **Resilient**: Handles failures gracefully

### Cons
* **Eventual Consistency**: Not immediately consistent
* **Complexity**: Need to design compensating actions
* **Debugging**: Harder to trace distributed flow

---

## 6. CQRS (Command Query Responsibility Segregation)

### Concept
* **Separate write model** (commands) from **read model** (queries)
* **Write Side**: Commands → Events → Event Store
* **Read Side**: Events → Projections → Read Model

### Why?
* **Different Needs**: Writes and reads have different requirements
* **Optimization**: Optimize each side independently
* **Scaling**: Scale reads and writes independently

---

## 7. CQRS Flow

### Write Flow
```
Command → Command Handler → Domain Logic → Event → Event Store
```

### Read Flow
```
Event → Projection → Read Model (Optimized for Queries)
```

### Example
* **Write**: CreateOrder command → OrderCreated event
* **Read**: Order list query → Reads from optimized read model (not event store)

---

## 8. CQRS Benefits

### Performance
* **Write Model**: Optimized for writes (normalized)
* **Read Model**: Optimized for reads (denormalized)
* **Example**: Write to normalized tables, read from materialized views

### Scalability
* **Independent Scaling**: Scale reads and writes separately
* **Read Replicas**: Multiple read models for different queries

### Flexibility
* **Multiple Read Models**: Different views for different use cases
* **Example**: Order list view, order detail view, analytics view

---

## 9. CQRS Trade-offs

### Pros
* **Performance**: Optimized for each operation
* **Scalability**: Independent scaling
* **Flexibility**: Multiple read models

### Cons
* **Complexity**: More moving parts
* **Eventual Consistency**: Read model may be stale
* **Storage**: Duplicate data (write + read models)

---

## 10. Combining Sagas and CQRS

### How They Work Together
* **Sagas**: Handle distributed transactions (writes)
* **CQRS**: Separate read/write models
* **Event Sourcing**: Store all events (enables both)

### Example Flow
```
Command → Saga Orchestrator → Multiple Services → Events
Events → Event Store (for Saga state)
Events → Projections → Read Models (for queries)
```

---

## 11. How to Explain This in an Interview

> "For distributed transactions across microservices, I use the Saga pattern instead of two-phase commit. For example, when placing an order, I orchestrate steps: create order, reserve inventory, process payment. If any step fails, I execute compensating actions. For read/write separation, I use CQRS: commands write to the event store, and projections build read-optimized models. The trade-off is eventual consistency - reads may be slightly stale, but this enables independent scaling and optimization of reads and writes."

**Key Points:**
* Sagas for distributed transactions (no ACID)
* CQRS for read/write separation
* Event sourcing enables both patterns
* Trade-off: Eventual consistency for scalability

---

## 12. Common Mistakes

* **Trying ACID across services**: "We use distributed transactions"
  * Fix: Use Sagas for distributed transactions
* **Synchronous compensation**: "We rollback synchronously"
  * Fix: Compensating actions can be async
* **No read model optimization**: "We query event store directly"
  * Fix: Build read-optimized projections
* **Ignoring eventual consistency**: "Reads are always consistent"
  * Fix: Acknowledge and handle eventual consistency
* **Over-engineering**: "We use CQRS for everything"
  * Fix: Use CQRS only when read/write needs differ significantly

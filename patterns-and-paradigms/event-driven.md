# Event-Driven Architecture

## 1. Problem Statement

How do we design systems where services communicate through events, enabling loose coupling and scalability?

---

## 2. Core Concepts

### Event
* **Definition**: Something that happened (immutable fact)
* **Examples**: UserCreated, OrderPlaced, PaymentProcessed
* **Characteristics**: Immutable, timestamped, contains data

### Event-Driven Architecture
* Services communicate via events
* **Producer**: Publishes events
* **Consumer**: Subscribes to events and reacts
* **Broker**: Routes events (message queue/stream)

---

## 3. Benefits

### Loose Coupling
* Services don't know about each other
* **Benefit**: Independent development, deployment
* **Trade-off**: Harder to trace, debug

### Scalability
* Services scale independently
* **Benefit**: Handle load spikes per service
* **Trade-off**: More infrastructure to manage

### Resilience
* Service failures don't cascade
* **Benefit**: Isolated failures
* **Trade-off**: Eventual consistency

### Flexibility
* Easy to add new consumers
* **Benefit**: New features without changing producers
* **Trade-off**: More complexity

---

## 4. Patterns

### Event Notification
* **Purpose**: Notify other services of changes
* **Example**: UserCreated event → Send welcome email
* **Data**: Minimal (just notification)

### Event-Carried State Transfer
* **Purpose**: Share state via events
* **Example**: OrderPlaced event contains full order data
* **Data**: Complete state (no need to query source)

### Event Sourcing
* **Purpose**: Store all events, rebuild state
* **Example**: Account balance = sum of all transactions
* **Benefit**: Complete audit trail, time travel

---

## 5. Event Flow

### Simple Flow
```
Producer → Event → Broker → Consumer
```

### Pub/Sub Flow
```
Producer → Event → Broker → [Consumer 1, Consumer 2, Consumer N]
```

### Event Sourcing Flow
```
Command → Event Store → Event → [Projection 1, Projection 2]
                              → Rebuild State
```

---

## 6. Event Store

### What is Event Store?
* Database that stores events (immutable log)
* **Characteristics**: Append-only, ordered, replayable

### Benefits
* **Audit Trail**: Complete history
* **Time Travel**: Rebuild state at any point
* **Debugging**: See exactly what happened

### Challenges
* **Storage**: Events accumulate (need retention policy)
* **Replay**: Rebuilding state can be slow
* **Versioning**: Event schema changes over time

---

## 7. Event Ordering

### Challenges
* Events may arrive out of order
* Network delays, retries
* Multiple producers

### Solutions
* **Partitioning**: Partition by key (e.g., user_id) for ordering
* **Vector Clocks**: Track causality
* **Sequence Numbers**: Per-partition ordering
* **Idempotency**: Handle duplicates gracefully

---

## 8. Event Versioning

### Problem
* Event schema changes over time
* Old consumers may not understand new events

### Solutions
* **Version in Event**: Include version field
* **Backward Compatibility**: Don't remove fields, add new ones
* **Multiple Versions**: Support multiple versions simultaneously
* **Schema Registry**: Centralized schema management

---

## 9. CQRS (Command Query Responsibility Segregation)

### Concept
* Separate write model (commands) from read model (queries)
* **Write Side**: Commands → Events → Event Store
* **Read Side**: Events → Projections → Read Model

### Benefits
* **Optimized Reads**: Read model optimized for queries
* **Scalability**: Scale reads and writes independently
* **Flexibility**: Multiple read models for different views

### Trade-offs
* **Complexity**: More moving parts
* **Eventual Consistency**: Read model may be stale
* **Storage**: Duplicate data (write + read models)

---

## 10. How to Explain This in an Interview

> "I use event-driven architecture when services need to be loosely coupled and scale independently. For example, when a user places an order, I publish an OrderPlaced event. Multiple consumers react independently: inventory service reserves items, payment service processes payment, email service sends confirmation. I use Kafka for event streaming because it provides ordering per partition, replay capability, and high throughput. The trade-off is eventual consistency and added complexity, but it enables services to evolve independently."

**Key Points:**
* Events enable loose coupling
* Use event streaming (Kafka) for high throughput
* Handle ordering via partitioning
* Make consumers idempotent

---

## 11. Common Mistakes

* **Synchronous events**: "We wait for all consumers"
  * Fix: Events should be async, fire-and-forget
* **No idempotency**: "We process events multiple times"
  * Fix: Make consumers idempotent (check if already processed)
* **Ignoring ordering**: "Events arrive randomly"
  * Fix: Partition by key for ordering, or accept eventual consistency
* **No event versioning**: "We change event schema"
  * Fix: Version events, maintain backward compatibility
* **Tight coupling**: "Consumers depend on producer implementation"
  * Fix: Events should be self-contained, not require queries

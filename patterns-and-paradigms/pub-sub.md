# Pub/Sub Pattern

## 1. Problem Statement

How do we implement publish-subscribe messaging to decouple producers and consumers, enabling scalable event-driven systems?

---

## 2. Core Concepts

### Publisher
* **Role**: Publishes messages/events
* **Knowledge**: Doesn't know who consumes
* **Example**: Order service publishes OrderPlaced event

### Subscriber
* **Role**: Consumes messages/events
* **Knowledge**: Subscribes to topics of interest
* **Example**: Email service subscribes to OrderPlaced

### Topic/Channel
* **Role**: Named channel for messages
* **Function**: Routes messages to subscribers
* **Example**: "orders", "payments", "notifications"

### Broker
* **Role**: Manages topics and routing
* **Function**: Receives from publishers, delivers to subscribers
* **Example**: Kafka, RabbitMQ, Google Pub/Sub

---

## 3. Benefits

### Decoupling
* Publishers don't know subscribers
* **Benefit**: Independent development, deployment
* **Trade-off**: Harder to trace message flow

### Scalability
* Add subscribers without changing publishers
* **Benefit**: Horizontal scaling of consumers
* **Trade-off**: More infrastructure to manage

### Flexibility
* Multiple subscribers per topic
* **Benefit**: One event triggers multiple actions
* **Example**: OrderPlaced → Email, Analytics, Inventory

### Resilience
* Messages buffered if subscribers are down
* **Benefit**: System continues operating
* **Trade-off**: Eventual delivery

---

## 4. Message Delivery Models

### At-Most-Once
* Message delivered at most once (may be lost)
* **Use Case**: Non-critical data (metrics, logs)
* **Trade-off**: Simple, but may lose messages

### At-Least-Once
* Message delivered at least once (may duplicate)
* **Use Case**: Most common, make consumers idempotent
* **Trade-off**: May process duplicates, but guaranteed delivery

### Exactly-Once
* Message delivered exactly once
* **Use Case**: Financial transactions (complex, expensive)
* **Trade-off**: Complex implementation, higher cost

---

## 5. Pub/Sub vs Point-to-Point

### Pub/Sub (Topic-Based)
* **Model**: One message → Multiple subscribers
* **Use Case**: Broadcasting events
* **Example**: OrderPlaced → Email, Analytics, Inventory

### Point-to-Point (Queue-Based)
* **Model**: One message → One consumer
* **Use Case**: Task distribution
* **Example**: Process payment → One worker processes

**Key Difference**: Pub/Sub broadcasts, Point-to-Point distributes work.

---

## 6. Implementation Patterns

### Push Model
* Broker pushes messages to subscribers
* **Pros**: Real-time, efficient
* **Cons**: Subscriber must be available

### Pull Model
* Subscribers poll broker for messages
* **Pros**: Subscriber controls rate
* **Cons**: Polling overhead, latency

### Long Polling
* Hybrid: Subscriber opens connection, broker holds until message
* **Pros**: Real-time feel, less overhead than polling
* **Cons**: More complex than pure push/pull

---

## 7. Topic Partitioning

### Why Partition?
* Distribute load across multiple brokers
* Enable parallel processing
* Maintain ordering per partition

### Partitioning Strategies
* **Round-Robin**: Distribute evenly
* **Key-Based**: Partition by message key (e.g., user_id)
* **Hash-Based**: Hash key to determine partition

**Key Insight**: Partitioning by key maintains ordering for that key.

---

## 8. Consumer Groups

### Concept
* Multiple consumers in a group
* Each message delivered to one consumer in group
* **Benefit**: Load balancing, parallel processing

### Example
* Topic: "orders" with 3 partitions
* Consumer Group: 3 consumers
* Each consumer processes one partition

**Scaling**: Add more consumers (up to partition count)

---

## 9. Message Ordering

### Challenges
* Messages may arrive out of order
* Multiple partitions = no global ordering
* Network delays, retries

### Solutions
* **Partition by Key**: Messages with same key → same partition → ordered
* **Single Partition**: Global ordering (but limits throughput)
* **Accept Out-of-Order**: For use cases where order doesn't matter

---

## 10. How to Explain This in an Interview

> "I use pub/sub for event-driven architecture. When an order is placed, the order service publishes an OrderPlaced event to the 'orders' topic. Multiple subscribers consume independently: email service sends confirmation, analytics service tracks metrics, inventory service updates stock. I use Kafka for high-throughput scenarios because it provides partitioning, consumer groups, and message retention. I partition by order_id to maintain ordering per order, and use consumer groups to scale processing horizontally."

**Key Points:**
* Pub/sub enables decoupling and scalability
* Partition by key for ordering
* Use consumer groups for load balancing
* Choose delivery guarantee based on use case

---

## 11. Common Mistakes

* **Synchronous pub/sub**: "We wait for all subscribers"
  * Fix: Pub/sub should be async, fire-and-forget
* **No idempotency**: "We process duplicates"
  * Fix: Make consumers idempotent (check if already processed)
* **Ignoring ordering**: "Messages arrive randomly"
  * Fix: Partition by key for ordering, or accept out-of-order
* **Too many topics**: "One topic per event type"
  * Fix: Group related events, avoid topic explosion
* **No retention policy**: "Messages stored forever"
  * Fix: Set retention (time or size based)

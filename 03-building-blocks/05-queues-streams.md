# Queues and Streams

## 1. Problem Statement

How do we choose between message queues and streams for asynchronous processing and event-driven architectures?

---

## 2. Message Queues

### Characteristics
* **Point-to-Point**: One message → One consumer
* **At-Most-Once or At-Least-Once**: Delivery guarantees
* **Pull Model**: Consumers poll for messages
* **Bounded**: Messages are consumed and removed

### Use Cases
* Task distribution (work queues)
* Decoupling services
* Handling bursts
* Retry logic

### Examples
* **RabbitMQ**: Traditional message broker
* **Amazon SQS**: Managed queue service
* **Azure Service Bus**: Enterprise messaging

---

## 3. Message Streams

### Characteristics
* **Pub/Sub**: One message → Multiple consumers
* **At-Least-Once**: Delivery guarantee
* **Push Model**: Stream pushes to consumers
* **Unbounded**: Messages are retained (time-based or size-based)

### Use Cases
* Event sourcing
* Real-time analytics
* Log aggregation
* Event-driven architecture

### Examples
* **Apache Kafka**: Distributed event streaming
* **Amazon Kinesis**: Managed streaming service
* **Google Pub/Sub**: Global messaging service

---

## 4. Key Differences

| Aspect | Message Queue | Stream |
|--------|---------------|--------|
| **Model** | Point-to-point | Pub/Sub |
| **Retention** | Until consumed | Time/size based |
| **Consumers** | One per message | Multiple per message |
| **Ordering** | Per queue | Per partition |
| **Replay** | No | Yes (within retention) |
| **Throughput** | Medium | Very high |
| **Use Case** | Task processing | Event streaming |

---

## 5. Queue Patterns

### Work Queue
* Multiple workers process jobs
* Load balancing across workers
* **Example**: Image processing, email sending

### Priority Queue
* High-priority messages processed first
* **Example**: Critical alerts before regular notifications

### Dead Letter Queue (DLQ)
* Failed messages go here
* Manual inspection and retry
* **Example**: Messages that failed after max retries

### Delayed Queue
* Messages processed after delay
* **Example**: Scheduled tasks, reminders

---

## 6. Stream Patterns

### Event Sourcing
* Store all events (immutable log)
* Rebuild state by replaying events
* **Example**: Bank account balance from transactions

### CQRS (Command Query Responsibility Segregation)
* Separate write and read models
* Stream updates read model
* **Example**: Write to event stream, read from materialized view

### Change Data Capture (CDC)
* Capture database changes as events
* Stream to other systems
* **Example**: Sync data across microservices

---

## 7. Partitioning

### Queue Partitioning
* Multiple queues for different priorities/workloads
* **Example**: High-priority queue, low-priority queue

### Stream Partitioning
* Partition by key (e.g., user_id)
* Maintains ordering per partition
* **Example**: Kafka partitions by user_id for ordered processing

**Key Insight**: Partitioning enables parallel processing while maintaining ordering where needed.

---

## 8. Delivery Guarantees

### At-Most-Once
* Message may be lost
* **Use Case**: Non-critical data (metrics, logs)

### At-Least-Once
* Message delivered at least once (may duplicate)
* **Use Case**: Most common, make consumers idempotent

### Exactly-Once
* Message delivered exactly once
* **Use Case**: Financial transactions (complex, expensive)

**Trade-off**: Simplicity vs Guarantees
* At-least-once is simpler, make idempotent
* Exactly-once is complex, use only when necessary

---

## 9. How to Explain This in an Interview

> "I use message queues (like SQS) for task distribution and decoupling services. For example, when a user signs up, I enqueue an email job and return immediately. A worker processes it asynchronously. For event streaming (like Kafka), I use it when I need multiple consumers, event replay, or high throughput. For example, user actions are published to a stream, and analytics, recommendations, and notifications all consume from it independently."

**Decision Framework:**
* **Task processing, one consumer**: Message queue
* **Events, multiple consumers, replay needed**: Stream
* **High throughput, ordering important**: Stream with partitioning
* **Simple decoupling**: Message queue

---

## 10. Common Mistakes

* **Queue for everything**: "All async processing uses queues"
  * Fix: Use streams for events that need multiple consumers
* **Stream for simple tasks**: "We use Kafka for everything"
  * Fix: Use queues for simple task distribution
* **Ignoring ordering**: "Order doesn't matter" (when it does)
  * Fix: Use partitioning or single consumer for ordering
* **No idempotency**: "We process duplicates"
  * Fix: Make consumers idempotent for at-least-once delivery
* **Wrong retention**: "Stream keeps all messages forever"
  * Fix: Set retention policy (time or size based)
* **Not handling backpressure**: "Messages pile up"
  * Fix: Monitor queue depth, scale consumers, or throttle producers

# Message Queues

## 1. Problem Statement

How do we decouple services, handle asynchronous processing, and ensure reliable message delivery in distributed systems?

---

## 2. Clarifying Questions

* What's the message volume? (QPS)
* What's the delivery guarantee? (At-least-once vs exactly-once)
* What's the processing time? (Seconds vs hours)
* Do we need ordering? (FIFO vs any order)

---

## 3. Requirements

### Functional

* Send and receive messages
* Guarantee delivery (at-least-once)
* Handle failures gracefully
* Support high throughput

### Non-Functional

* Low latency (<10ms publish)
* High durability (messages not lost)
* Horizontal scalability
* Ordering (if needed)

---

## 4. Scale Assumptions

* 100K messages/second
* 1M messages in queue (peak)
* Processing time: 100ms-10s per message
* 99.99% delivery guarantee

---

## 5. High-Level Architecture

**Basic Queue**
```
Producer → Queue → Consumer
```

**Pub/Sub**
```
Publisher → Topic → [Subscriber 1, Subscriber 2, ...]
```

**Work Queue**
```
Producer → Queue → [Worker 1, Worker 2, ..., Worker N]
```

---

## 6. Core Components

* **Queue/Topic**: Message storage
* **Producer/Publisher**: Sends messages
* **Consumer/Subscriber**: Processes messages
* **Broker**: Manages queue (Kafka, RabbitMQ, SQS)
* **Dead Letter Queue**: Failed messages

---

## 7. Data Flow

**Basic Queue Flow**
1. Producer → Publish message to queue
2. Queue → Store message (durable)
3. Queue → Return ACK to producer
4. Consumer → Poll queue for messages
5. Queue → Deliver message to consumer
6. Consumer → Process message
7. Consumer → Send ACK (delete from queue)
8. If no ACK → Redeliver after timeout

**Pub/Sub Flow**
1. Publisher → Publish to topic
2. Topic → Replicate to all subscribers
3. Each subscriber → Receives copy
4. Each subscriber → Process independently

**Work Queue Flow**
1. Multiple producers → Publish to queue
2. Queue → Distribute to available workers
3. Worker → Process message
4. Worker → ACK when done
5. If worker fails → Redeliver to another worker

---

## 8. Bottlenecks

* **Single Queue**: Becomes bottleneck
  * Solution: Partition queue (Kafka partitions, SQS queues)
* **Slow Consumers**: Messages pile up
  * Solution: Scale consumers, or use multiple queues
* **Message Ordering**: Hard to maintain at scale
  * Solution: Use FIFO queues, or partition by key
* **Exactly-Once Delivery**: Complex, expensive
  * Solution: Use at-least-once, make consumers idempotent
* **Queue Size**: Memory/disk limits
  * Solution: Set retention policy, scale broker

---

## 9. Trade-offs

| Pattern | Pros | Cons | When to Use |
|---------|------|------|-------------|
| **Point-to-Point (Queue)** | Simple, load balancing | One consumer per message | Task distribution |
| **Pub/Sub (Topic)** | Multiple consumers | More complex, all get message | Event broadcasting |
| **At-Least-Once** | Simple, reliable | Duplicates possible | Most use cases |
| **Exactly-Once** | No duplicates | Complex, expensive | Financial transactions |

**Queue Types:**
- **Standard Queue**: High throughput, best-effort ordering
- **FIFO Queue**: Strict ordering, lower throughput
- **Priority Queue**: Process important messages first

**Delivery Guarantees:**
- **At-Most-Once**: May lose messages (rarely used)
- **At-Least-Once**: May duplicate (common, make idempotent)
- **Exactly-Once**: No loss, no duplicates (complex, expensive)

---

## 10. How to Explain This in an Interview

> "I use a message queue (like Kafka) to decouple the API service from the email service. When a user signs up, the API publishes an event to the queue and returns immediately. The email service consumes asynchronously. The trade-off is eventual processing (emails sent within seconds), but this improves API latency and allows the email service to scale independently."

**Key Points:**
* Always mention decoupling benefit
* Explain delivery guarantee (at-least-once vs exactly-once)
* Mention ordering if relevant
* Acknowledge eventual processing trade-off

---

## 11. Common Mistakes

* **Synchronous processing**: "We wait for queue processing"
  * Fix: Queue should be async, return immediately
* **No idempotency**: "We process duplicates"
  * Fix: Make consumers idempotent (check if processed)
* **Single queue for everything**: "All messages go to one queue"
  * Fix: Use separate queues for different priorities/workloads
* **No dead letter queue**: "Failed messages disappear"
  * Fix: Use DLQ for failed messages, monitor and retry
* **Ignoring ordering**: "Order doesn't matter" (when it does)
  * Fix: Use FIFO queue or partition by key
* **Over-engineering**: "We need exactly-once for everything"
  * Fix: Use at-least-once with idempotency (simpler, cheaper)

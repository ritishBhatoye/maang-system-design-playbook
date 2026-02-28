# 💬 Chat System (WhatsApp / Messenger)

## 1. Problem Statement

Design a real-time messaging system like **WhatsApp** or **Facebook Messenger** that supports 1-on-1 and group chats at scale.

---

## 2. Clarifying Questions (Ask First)

* What message types? (Text, images, videos, files)
* Group chat size? (1-on-1 vs groups of 100 vs 1000+)
* Message delivery guarantee? (At-least-once vs exactly-once)
* Offline message handling? (Queue messages, deliver when online)
* Read receipts required? (Seen/delivered status)
* Message history? (How long to store)

---

## 3. Requirements

### Functional

* Send/receive messages (1-on-1 and group)
* Real-time delivery (<100ms latency)
* Offline message queuing
* Message history
* Read receipts (optional)
* Typing indicators (optional)

### Non-Functional

* Low latency (<100ms for online users)
* High availability (99.99%)
* Scalable to billions of messages/day
* Message ordering (per conversation)
* At-least-once delivery

---

## 4. Scale Assumptions

* 1B users, 100M daily active
* 50B messages/day
* Average: 50 messages/user/day
* Peak: 1M messages/second
* Group chats: Average 10 members, max 1000
* Message size: 1KB average (text), 100KB (with media)
* Storage: 50B messages × 1KB = 50TB/day, ~18PB/year (with 1-year retention)

---

## 5. High-Level Architecture

```
Client (Mobile/Web) → Load Balancer → API Gateway
                                          │
                    ┌────────────────────┼────────────────────┐
                    ↓                    ↓                    ↓
            Chat Service          Presence Service      Media Service
                    │                    │                    │
                    ↓                    ↓                    ↓
            Message Queue         WebSocket Manager    Object Storage
            (Kafka/RabbitMQ)      (Connection Mgr)     (S3)
                    │
                    ↓
            Database (Messages)
            (Cassandra/DynamoDB)
```

---

## 6. Core Components

* **API Gateway**: Routes requests, handles auth
* **Chat Service**: Handles message logic, validation
* **Presence Service**: Tracks online/offline status
* **WebSocket Manager**: Manages persistent connections for real-time delivery
* **Message Queue**: Buffers messages for offline users, fan-out for groups
* **Database**: Stores message history (NoSQL, sharded by conversation_id)
* **Media Service**: Handles images/videos (upload to S3, return URL)
* **Notification Service**: Push notifications for offline users

---

## 7. Data Flow

**Send Message Flow (1-on-1)**
1. User A → Sends message to User B
2. Client → POST /api/messages {to_user_id, content}
3. API Gateway → Authenticate, validate
4. Chat Service → Store message in DB (conversation_id, message_id, content, timestamp)
5. Chat Service → Check if User B is online (Presence Service)
6. If online → Push via WebSocket immediately
7. If offline → Enqueue to message queue
8. Chat Service → Return success to User A
9. Notification Service → Send push notification to User B (if offline)

**Receive Message Flow (Real-time)**
1. User B → Connected via WebSocket
2. Chat Service → Receives message from queue or real-time
3. WebSocket Manager → Push message to User B's connection
4. User B → Receives message (<100ms if online)
5. User B → Sends ACK (delivered/read)
6. Chat Service → Update message status in DB

**Group Chat Flow**
1. User → Sends message to group (1000 members)
2. Chat Service → Store message in DB
3. Chat Service → Fan-out to all members:
   - Online members → Push via WebSocket
   - Offline members → Enqueue to message queue
4. Each member → Receives message independently
5. Message Queue → Distributes to offline users when they come online

**Offline Message Delivery**
1. User → Comes online, connects WebSocket
2. Chat Service → Query message queue for pending messages
3. Chat Service → Deliver all queued messages
4. User → Receives messages in order (by timestamp)

---

## 8. Bottlenecks

* **WebSocket Connection Management**: 100M concurrent connections
  * Solution: Horizontal scaling, connection pooling, load balancing by user_id
* **Group Chat Fan-out**: 1 message → 1000 pushes
  * Solution: Message queue for async fan-out, batch processing
* **Message Ordering**: Messages arrive out of order
  * Solution: Timestamp + sequence number, client-side ordering
* **Database Write Bottleneck**: 1M writes/sec
  * Solution: Shard by conversation_id, use NoSQL (Cassandra)
* **Message History Queries**: Loading old messages is slow
  * Solution: Pagination, separate cold storage for old messages
* **Media Storage**: Large files (images, videos)
  * Solution: Store in object storage (S3), return URLs in messages

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Real-time Delivery** | WebSocket for online, queue for offline | Low latency for online users | Connection management complexity |
| **Database** | NoSQL (Cassandra) sharded by conversation_id | Horizontal scale, simple queries | Eventual consistency, no complex queries |
| **Message Ordering** | Timestamp + client-side ordering | Simple, scalable | May need client-side merge logic |
| **Delivery Guarantee** | At-least-once | Simpler than exactly-once | Duplicates possible, make idempotent |
| **Group Fan-out** | Async via message queue | Doesn't block sender | Slight delay for large groups |
| **Message History** | Store in DB, paginate | Simple queries | May need cold storage for very old messages |

**Key Trade-off:**
> "I use WebSocket for real-time delivery to online users because <100ms latency is critical for chat. The trade-off is connection management complexity, but the user experience justifies it. For offline users, I queue messages and deliver when they reconnect, accepting eventual delivery."

---

## 10. How to Explain This in an Interview

> "I'll design a real-time chat system with WebSocket for online delivery and message queues for offline users.
>
> For architecture, I use an API gateway for routing, a stateless chat service for message logic, and a WebSocket manager to handle persistent connections. I shard the database by conversation_id using NoSQL (Cassandra) to handle 1M messages/second writes.
>
> For 1-on-1 chats, when User A sends a message, I store it in the database and check if User B is online. If online, I push immediately via WebSocket. If offline, I enqueue to a message queue and send a push notification.
>
> For group chats, I use async fan-out via message queue. When a message is sent to a 1000-member group, I store it once in the database, then fan out to all members asynchronously. Online members get it via WebSocket, offline members get it from the queue when they reconnect.
>
> For message ordering, I use timestamps and sequence numbers. The client handles ordering to avoid complex server-side coordination. For delivery, I use at-least-once delivery and make message processing idempotent to handle duplicates.
>
> The key trade-off is connection management complexity for real-time delivery, but the <100ms latency for online users justifies it."

**Key Phrases:**
* "WebSocket for real-time, queue for offline"
* "Shard by conversation_id for horizontal scale"
* "Async fan-out for group chats"
* "At-least-once delivery with idempotency"

---

## 11. Common Mistakes

* **Synchronous group fan-out**: "Push to all 1000 members synchronously"
  * Fix: Use async message queue for fan-out
* **No offline handling**: "All messages are real-time"
  * Fix: Queue messages for offline users, deliver on reconnect
* **Single database**: "One database for all messages"
  * Fix: Shard by conversation_id for scale
* **No message ordering**: "Messages arrive randomly"
  * Fix: Use timestamps + sequence numbers, client-side ordering
* **Exactly-once delivery**: "We guarantee no duplicates"
  * Fix: Use at-least-once, make idempotent (simpler, cheaper)
* **WebSocket for everything**: "All 1B users connected"
  * Fix: Only active users, use polling/queue for others
* **No read receipts**: "Can't track delivery"
  * Fix: Store message status (sent/delivered/read) in DB

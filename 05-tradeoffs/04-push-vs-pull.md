# Push vs Pull

## 1. Problem Statement

When do we push data to users (real-time) versus having users pull data (on-demand) in distributed systems?

---

## 2. Clarifying Questions

* What's the update frequency? (Real-time vs periodic)
* What's the user base? (Millions vs thousands)
* What's the data freshness requirement? (Immediate vs eventual)
* What's the fan-out? (One-to-many vs one-to-one)

---

## 3. Requirements

### Functional

* Deliver updates to users
* Handle high user count
* Support real-time or near-real-time updates
* Scale efficiently

### Non-Functional

* Low latency (<100ms for push)
* High availability
* Cost-effective at scale
* Handle connection failures

---

## 4. Scale Assumptions

* **Push Scenario**: 1M users, 10K updates/sec, real-time needed
* **Pull Scenario**: 100M users, 1 update/sec per user, eventual OK
* Fan-out: 1 update → 1M users (push) vs 1M users polling (pull)

---

## 5. High-Level Architecture

**Push Model**
```
Update → Fan-out Service → [User 1, User 2, ..., User N]
         (WebSocket/SSE)
```

**Pull Model**
```
Users → Poll API → Check for updates → Return if available
```

**Hybrid Model**
```
Update → Cache → Users poll cache (frequent, lightweight)
```

---

## 6. Core Components

**Push:**
* **Fan-out Service**: Distributes updates to users
* **WebSocket/SSE**: Persistent connections
* **Connection Manager**: Manages user connections
* **Message Queue**: Buffers updates during connection loss

**Pull:**
* **API Endpoint**: Returns latest data
* **Cache**: Stores recent updates
* **Polling Service**: Clients poll periodically
* **Delta Endpoint**: Returns only changes since last poll

---

## 7. Data Flow

**Push Flow**
1. Event occurs (new message, update)
2. Fan-out service → Determine recipients
3. Fan-out service → Push to connected users (WebSocket)
4. If user offline → Store in queue
5. When user reconnects → Deliver queued messages
6. User → Receives update in real-time

**Pull Flow**
1. User → Polls API every N seconds
2. API → Check for updates since last poll (timestamp/cursor)
3. API → Return updates (or empty if none)
4. User → Processes updates
5. Repeat polling

**Hybrid Flow (Long Polling)**
1. User → Opens long-polling request
2. Server → Holds connection open
3. If update arrives → Return immediately
4. If timeout (30s) → Return empty, client reconnects
5. Reduces polling overhead while maintaining simplicity

---

## 8. Bottlenecks

**Push Bottlenecks:**
* **Connection Management**: 1M WebSocket connections
  * Solution: Connection pooling, load balancing, horizontal scaling
* **Fan-out Cost**: 1 update → 1M pushes
  * Solution: Batch updates, use message queue, or hybrid approach
* **Offline Users**: Messages pile up
  * Solution: Message queue with TTL, or fallback to pull
* **Connection Failures**: Lost messages
  * Solution: Acknowledge delivery, retry mechanism

**Pull Bottlenecks:**
* **Server Load**: 1M users polling every second
  * Solution: Increase poll interval, use cache, or long polling
* **Wasted Bandwidth**: Empty responses
  * Solution: Long polling, or delta endpoints (only changes)
* **Staleness**: Users see updates late
  * Solution: Shorter poll interval (but higher load)

---

## 9. Trade-offs

| Aspect | Push | Pull | When to Use |
|--------|------|------|-------------|
| **Latency** | ✅ Real-time (<100ms) | ❌ Depends on poll interval | Push for real-time |
| **Server Load** | ⚠️ High (maintain connections) | ⚠️ High (many polls) | Depends on update frequency |
| **Scalability** | ❌ Hard (1M connections) | ✅ Easier (stateless) | Pull for high user count |
| **Battery/Resources** | ✅ Efficient (server pushes) | ❌ Wastes resources (polling) | Push for mobile |
| **Complexity** | ❌ High (connection management) | ✅ Simple (HTTP polling) | Pull for simplicity |
| **Offline Handling** | ❌ Complex (queue messages) | ✅ Simple (poll when online) | Pull for offline resilience |
| **Fan-out Efficiency** | ❌ Expensive (1→N pushes) | ✅ Efficient (N polls) | Pull for high fan-out |

**Decision Matrix:**

**Choose Push when:**
* Real-time updates critical (<1s latency)
* Low to medium user count (<100K concurrent)
* High update frequency per user
* Mobile apps (battery efficient)
* Chat, notifications, live feeds

**Choose Pull when:**
* Very high user count (>1M)
* Low update frequency
* Eventual consistency acceptable
* Simpler architecture preferred
* News feeds, social media timelines

**Choose Hybrid (Long Polling) when:**
* Want real-time feel with simpler architecture
* Medium user count
* Can tolerate slight delay

---

## 10. How to Explain This in an Interview

> "For a chat system with 10K concurrent users, I choose push (WebSocket) because real-time delivery is critical and the user count is manageable. The trade-off is connection management complexity, but the low latency (<100ms) justifies it. For a news feed with 100M users, I choose pull with long polling because maintaining 100M WebSocket connections is impractical, and users can tolerate 1-2 second delay."

**Alternative:**
> "I use a hybrid approach: push for online users via WebSocket for real-time updates, and pull for offline users who reconnect and poll for missed messages. This gives us real-time for active users while handling offline gracefully."

**Key Points:**
* Always mention the scale (user count) that drives the decision
* Acknowledge the trade-off (complexity vs latency)
* Consider hybrid approaches when appropriate

---

## 11. Common Mistakes

* **Push for everything**: "WebSocket for all updates"
  * Fix: Use pull for high fan-out or high user count
* **Pull for real-time**: "Users poll every second"
  * Fix: Use push or long polling for real-time requirements
* **Ignoring connection limits**: "1M WebSocket connections"
  * Fix: Consider connection overhead, use pull for scale
* **Not handling offline**: "Push always works"
  * Fix: Queue messages for offline users, or fallback to pull
* **Too frequent polling**: "Poll every 100ms"
  * Fix: Use long polling or push instead
* **No delta optimization**: "Return all data every poll"
  * Fix: Return only changes since last poll (cursor/timestamp)

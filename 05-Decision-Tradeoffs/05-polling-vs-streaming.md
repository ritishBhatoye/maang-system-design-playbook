# Polling vs Streaming

## 1. Problem Statement

When do we have clients poll for updates versus stream updates in real-time?

---

## 2. Clarifying Questions

* What's the update frequency? (Real-time vs periodic)
* What's the user count? (Thousands vs millions)
* What's the data freshness requirement? (Immediate vs eventual)
* What's the connection type? (Web vs mobile)

---

## 3. Requirements

### Functional

* Deliver updates to clients
* Handle connection failures
* Support high user count
* Efficient resource usage

### Non-Functional

* Low latency (<100ms for streaming)
* High availability
* Scalability
* Battery efficient (mobile)

---

## 4. Scale Assumptions

* **Polling Scenario**: 1M users, poll every 5s, eventual consistency OK
* **Streaming Scenario**: 10K users, real-time updates, <100ms latency
* Update frequency: 1-100 updates/sec per user

---

## 5. High-Level Architecture

**Polling**
```
Client → Poll API every N seconds → Check for updates → Return
```

**Streaming (WebSocket/SSE)**
```
Client → Open connection → Server pushes updates → Real-time
```

**Long Polling (Hybrid)**
```
Client → Open connection → Hold until update or timeout → Return
```

---

## 6. Core Components

**Polling:**
* **API Endpoint**: Returns latest data
* **Cache**: Stores recent updates
* **Delta Endpoint**: Returns only changes
* **Polling Interval**: How often clients poll

**Streaming:**
* **WebSocket Server**: Maintains persistent connections
* **Connection Manager**: Tracks connected clients
* **Message Queue**: Buffers updates
* **Heartbeat**: Keeps connection alive

---

## 7. Data Flow

**Polling Flow**
1. Client → Polls API every N seconds (e.g., 5s)
2. API → Check for updates since last poll (timestamp/cursor)
3. API → Return updates (or empty if none)
4. Client → Processes updates
5. Client → Waits N seconds
6. Repeat

**Streaming Flow (WebSocket)**
1. Client → Opens WebSocket connection
2. Server → Accepts connection, stores in connection pool
3. When update occurs → Server pushes to client immediately
4. Client → Receives update in real-time (<100ms)
5. Connection → Stays open, server pushes future updates
6. If connection drops → Client reconnects, server may queue missed messages

**Long Polling Flow**
1. Client → Opens HTTP request (holds connection)
2. Server → Holds connection open (e.g., 30s timeout)
3. If update arrives → Return immediately, close connection
4. If timeout → Return empty, close connection
5. Client → Immediately opens new request
6. Reduces empty responses while maintaining HTTP simplicity

---

## 8. Bottlenecks

**Polling Bottlenecks:**
* **Server Load**: 1M users polling every 5s = 200K req/sec
  * Solution: Increase poll interval, use cache, or long polling
* **Wasted Bandwidth**: Empty responses
  * Solution: Long polling, or delta endpoints (only changes)
* **Staleness**: Users see updates late (up to poll interval)
  * Solution: Shorter interval (but higher load), or use streaming

**Streaming Bottlenecks:**
* **Connection Management**: 10K+ WebSocket connections
  * Solution: Horizontal scaling, load balancing, connection pooling
* **Memory**: Each connection consumes memory
  * Solution: Efficient connection management, or use polling for scale
* **Connection Failures**: Lost messages, reconnection complexity
  * Solution: Message queue for offline users, heartbeat for detection

---

## 9. Trade-offs

| Aspect | Polling | Streaming | When to Use |
|--------|---------|-----------|-------------|
| **Latency** | ❌ Depends on interval (5s+) | ✅ Real-time (<100ms) | Streaming for real-time |
| **Server Load** | ⚠️ High (many requests) | ⚠️ High (maintain connections) | Depends on update frequency |
| **Scalability** | ✅ Easier (stateless) | ❌ Harder (stateful connections) | Polling for high user count |
| **Battery/Resources** | ❌ Wastes resources (polling) | ✅ Efficient (server pushes) | Streaming for mobile |
| **Complexity** | ✅ Simple (HTTP) | ❌ Complex (WebSocket management) | Polling for simplicity |
| **Bandwidth** | ⚠️ Wasted on empty polls | ✅ Efficient (only when updates) | Streaming for efficiency |
| **Connection Resilience** | ✅ Resilient (stateless) | ❌ Complex (reconnection) | Polling for resilience |

**Decision Matrix:**

**Choose Polling when:**
* Very high user count (>100K)
* Low update frequency
* Eventual consistency acceptable
* Simpler architecture preferred
* Stateless is better

**Choose Streaming when:**
* Real-time updates critical (<1s)
* Medium user count (<50K concurrent)
* High update frequency
* Mobile apps (battery efficient)
* Chat, live feeds, notifications

**Choose Long Polling when:**
* Want real-time feel with HTTP simplicity
* Medium user count
* Can tolerate slight delay
* Don't want WebSocket complexity

---

## 10. How to Explain This in an Interview

> "For a chat system with 10K concurrent users, I choose streaming (WebSocket) because real-time message delivery is critical and the user count is manageable. The trade-off is connection management complexity, but the <100ms latency justifies it. For a news feed with 1M users, I choose polling with long polling because maintaining 1M WebSocket connections is impractical, and users can tolerate 1-2 second delay."

**Alternative:**
> "I use a hybrid approach: streaming for active users (real-time), and polling for inactive users who check occasionally. This optimizes resources while providing real-time for engaged users."

**Key Points:**
* Always mention the scale (user count) and latency requirement
* Acknowledge the trade-off (complexity vs latency)
* Consider hybrid approaches when appropriate

---

## 11. Common Mistakes

* **Streaming for everything**: "WebSocket for all updates"
  * Fix: Use polling for high user count or low update frequency
* **Polling for real-time**: "Users poll every 100ms"
  * Fix: Use streaming or long polling for real-time requirements
* **Ignoring connection limits**: "1M WebSocket connections"
  * Fix: Consider connection overhead, use polling for scale
* **Too frequent polling**: "Poll every 100ms"
  * Fix: Use streaming or long polling instead
* **No delta optimization**: "Return all data every poll"
  * Fix: Return only changes since last poll (cursor/timestamp)
* **Not handling reconnection**: "Connection drops, messages lost"
  * Fix: Queue messages for reconnection, or fallback to polling

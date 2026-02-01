# ⚡ Scaling Real-Time Systems (2026 MAANG Edition)

## 1. Why This Matters in Interviews

Real-time systems require **sub-second latency** end-to-end. Interviewers test this to see if you understand the difference between "fast" and "real-time."

---

## 2. Problem Interviewers Are Testing

- Do you understand real-time constraints (latency budgets)?
- Can you design bidirectional, low-latency communication?
- Do you know when to sacrifice consistency for speed?

---

## 3. Key Concepts

### What is "Real-Time"?

```
Soft Real-Time (most systems):
├── Chat messages: < 500ms
├── Stock ticker: < 100ms
├── Gaming: < 50ms
└── Voice/video: < 20ms

Hard Real-Time (specialized):
├── Trading systems: < 1ms
├── Autonomous vehicles: < 10ms
└── Industrial control: < 1ms

Miss deadline = failure or degraded experience
```

### Latency Budget Example

```
User types message → Friend sees it

Total budget: 500ms

Breakdown:
├── Client → Server: 50ms (network)
├── Message queue: 10ms
├── Process + store: 50ms
├── Fanout to recipient: 20ms
├── Push notification: 100ms
├── Server → Client: 50ms (network)
└── Client render: 20ms
─────────────────
Total: 300ms (200ms buffer)
```

---

## 4. Communication Patterns

### WebSocket (Most Common)

```
Client ←→ Server
Full-duplex, persistent connection

Pros:
├── Low latency (no connection overhead)
├── Bidirectional
└── Real-time updates

Cons:
├── Connection state management
├── Horizontal scaling complexity
└── Load balancer sticky sessions
```

### Server-Sent Events (SSE)

```
Client ← Server
Server push only

Pros:
├── Simpler than WebSocket
├── Auto-reconnect built-in
└── Works through HTTP proxies

Cons:
├── Unidirectional only
└── Limited connection count in browser
```

### Long Polling (Fallback)

```
Client → Server: "Any updates?"
Server: Holds connection until update or timeout
Client: Immediately reconnects

Pros:
├── Works everywhere
└── Simple to implement

Cons:
├── Higher latency
└── More connections
```

---

## 5. Architecture Pattern

```
                    ┌─────────────────────────────────┐
                    │       Load Balancer             │
                    │   (sticky sessions/WebSocket)   │
                    └───────────────┬─────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
        ┌──────────┐          ┌──────────┐          ┌──────────┐
        │  WS      │          │  WS      │          │  WS      │
        │ Server 1 │          │ Server 2 │          │ Server 3 │
        └────┬─────┘          └────┬─────┘          └────┬─────┘
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   │
                           ┌───────┴───────┐
                           │  Redis Pub/Sub │
                           │  (fanout)      │
                           └───────────────┘
```

### Message Flow

```
1. User A sends message (WS Server 1)
2. Server 1 publishes to Redis Pub/Sub
3. All servers receive broadcast
4. Server 3 (where User B connected) pushes to B
5. Total latency: < 100ms
```

---

## 6. Scaling Challenges

### Connection State

```
Problem: 1M concurrent connections
├── Each WS connection = memory + file descriptor
├── 1 server can handle ~100K connections
└── Need 10+ servers

Solution: Connection routing layer
├── Track user → server mapping in Redis
├── Route messages through Pub/Sub
└── Stateless servers with shared state
```

### Presence (Who's Online)

```
Pattern: Heartbeat + TTL

Every 30 seconds:
├── Client sends heartbeat
├── Server updates Redis: SET user:123:online EX 60
└── If no heartbeat: TTL expires → offline

Query presence:
├── Check EXISTS user:123:online
└── O(1) lookup
```

---

## 7. Trade-offs

| Decision | Option A | Option B | When to Choose A |
|----------|----------|----------|------------------|
| **Protocol** | WebSocket | HTTP polling | Bidirectional needed |
| **Fanout** | Pub/Sub | Direct message | Broadcast patterns |
| **Persistence** | Write-through | Fire-and-forget | Message durability required |
| **Ordering** | Strict order | Best effort | Chat, collaborative editing |

---

## 8. Interview Explanation

> "For a real-time chat system, I'd use WebSocket for bidirectional, low-latency communication. Target end-to-end latency is under 500ms.
>
> WebSocket servers are stateless—they maintain connections but no application state. When User A sends a message, the server publishes to Redis Pub/Sub. All servers receive the broadcast and push to their connected clients.
>
> For connection routing, Redis tracks which server each user is connected to. For fanout to group chats, we publish once and all servers with group members push locally.
>
> The trade-off is complexity in connection management. We need sticky sessions at the load balancer (or connection migration), and servers must gracefully handle reconnection with message replay.
>
> For presence, heartbeats every 30 seconds with 60-second TTL in Redis. No heartbeat = user offline. This scales to millions of users with O(1) presence checks."

---

## 9. Common Mistakes

- **HTTP for real-time**: "Just poll every second" → High latency, many connections
- **Single server**: "WebSocket to one server" → Can't scale
- **No reconnection logic**: "Connection drops, user refreshes" → Bad UX
- **Blocking operations**: "Query DB on message" → Latency spike

---

## 10. Senior-Level Phrases

- "WebSocket gives us persistent, bidirectional communication with sub-100ms latency."
- "Redis Pub/Sub handles cross-server message fanout without message persistence."
- "Presence is tracked via heartbeat with TTL—no heartbeat means offline."
- "The latency budget breaks down to 50ms network + 50ms processing + 50ms fanout."

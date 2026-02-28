# 🔔 Notification System

## 1. Problem Statement

Design a notification system that delivers push notifications, emails, and SMS to users at scale (like **Firebase Cloud Messaging** or **Apple Push Notification Service**).

---

## 2. Clarifying Questions (Ask First)

* What notification types? (Push, email, SMS, in-app)
* What's the delivery guarantee? (At-least-once vs exactly-once)
* What's the priority? (Real-time vs batch)
* User preferences? (Opt-in/opt-out, frequency limits)
* Global or regional? (Affects provider selection)

---

## 3. Requirements

### Functional

* Send push notifications (iOS, Android, Web)
* Send emails
* Send SMS (optional)
* Track delivery status
* Handle user preferences
* Support batching

### Non-Functional

* High throughput (millions of notifications/day)
* Low latency (<1s for push, <5s for email)
* Reliable delivery (at-least-once)
* Scalable and cost-effective

---

## 4. Scale Assumptions

* 1B users
* 10B notifications/day
* Peak: 100K notifications/second
* Types: 70% push, 20% email, 10% SMS
* Average notification size: 1KB
* Storage: Delivery logs, user preferences

---

## 5. High-Level Architecture

```
Event Trigger → Notification Service → Message Queue
                                        │
                    ┌───────────────────┼───────────────────┐
                    ↓                   ↓                   ↓
            Push Service          Email Service        SMS Service
                    │                   │                   │
                    ↓                   ↓                   ↓
            FCM/APNS Provider    Email Provider      SMS Provider
            (Google/Apple)       (SendGrid/SES)      (Twilio)
```

---

## 6. Core Components

* **Notification Service**: Core logic, routing, batching
* **Message Queue**: Buffers notifications (Kafka/RabbitMQ)
* **Push Service**: Handles iOS (APNS) and Android (FCM) push
* **Email Service**: Sends emails via provider (SendGrid, SES)
* **SMS Service**: Sends SMS via provider (Twilio)
* **User Preference Service**: Stores opt-in/opt-out, frequency limits
* **Delivery Tracking**: Logs delivery status, retries
* **Template Service**: Renders notification content

---

## 7. Data Flow

**Notification Flow (Push)**
1. Event occurs → User action triggers notification
2. Notification Service → Validate user preferences (opt-in, frequency limits)
3. Notification Service → Render template with user data
4. Notification Service → Enqueue to message queue (partitioned by user_id)
5. Push Service Worker → Consumes from queue
6. Push Service → Get device tokens from user service
7. Push Service → Batch notifications (for efficiency)
8. Push Service → Send to FCM/APNS provider
9. Provider → Delivers to device
10. Push Service → Track delivery status (success/failure)
11. If failure → Retry with exponential backoff

**Email Flow**
1. Notification Service → Enqueue email to queue
2. Email Service Worker → Consumes email job
3. Email Service → Render email template (HTML/text)
4. Email Service → Send via provider (SendGrid/SES)
5. Provider → Delivers email
6. Email Service → Track delivery (opened, clicked)

**SMS Flow**
1. Notification Service → Enqueue SMS to queue
2. SMS Service Worker → Consumes SMS job
3. SMS Service → Format message (character limits)
4. SMS Service → Send via provider (Twilio)
5. Provider → Delivers SMS
6. SMS Service → Track delivery status

**User Preference Check**
1. Notification Service → Before sending, check user preferences
2. Preference Service → Query database (cached)
3. Check → Opt-in status, frequency limits, channel preferences
4. If allowed → Proceed with sending
5. If not allowed → Skip or queue for later

---

## 8. Bottlenecks

* **Provider Rate Limits**: FCM/APNS have rate limits
  * Solution: Batch notifications, use multiple provider accounts, queue and throttle
* **Device Token Management**: Storing/updating tokens for 1B users
  * Solution: Shard by user_id, cache hot tokens, background sync
* **Template Rendering**: Slow for millions of notifications
  * Solution: Pre-render common templates, cache rendered content
* **Delivery Tracking**: Logging 10B events/day
  * Solution: Write to time-series DB, aggregate, archive old data
* **User Preference Queries**: Checking preferences for every notification
  * Solution: Cache preferences in Redis, batch checks
* **Queue Backlog**: Notifications pile up faster than processing
  * Solution: Scale workers, prioritize critical notifications, use multiple queues

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Delivery Guarantee** | At-least-once | Simpler than exactly-once | Duplicates possible, make idempotent |
| **Batching** | Batch notifications (100-1000) | Reduces provider API calls | Slight delay (acceptable for most) |
| **Queue Strategy** | Separate queues per channel | Isolate failures, different scaling | More queues to manage |
| **Retry Strategy** | Exponential backoff, max 3 retries | Handles transient failures | May drop notifications after max retries |
| **User Preferences** | Check before sending | Respects user choice | Adds latency, but necessary |
| **Template Rendering** | Pre-render + cache | Fast for high volume | May need to invalidate cache |

**Key Trade-off:**
> "I use message queues for async processing because sending 100K notifications/second synchronously would be too slow. The trade-off is eventual delivery (notifications arrive within seconds), but this allows the system to scale and handle bursts. For critical notifications, I use a priority queue for faster delivery."

---

## 10. How to Explain This in an Interview

> "I'll design a notification system with separate services for push, email, and SMS, using message queues for async processing.
>
> For architecture, when an event triggers a notification, the notification service validates user preferences, renders the template, and enqueues to a message queue partitioned by channel (push, email, SMS). Workers consume from queues and send via external providers (FCM, SendGrid, Twilio).
>
> For push notifications, I batch 100-1000 notifications per API call to FCM/APNS to reduce API calls and stay within rate limits. I store device tokens in a database sharded by user_id and cache hot tokens in Redis for fast access.
>
> For reliability, I use at-least-once delivery with idempotent processing. If a provider fails, I retry with exponential backoff (up to 3 retries). I track delivery status in a time-series database for analytics and debugging.
>
> For user preferences, I check opt-in status and frequency limits before sending. I cache preferences in Redis to avoid database queries for every notification.
>
> For scaling, I use separate queues per channel so I can scale push workers independently from email workers. I also use priority queues for critical notifications (e.g., security alerts) that need immediate delivery.
>
> The key trade-off is async processing for scalability, accepting eventual delivery (seconds) instead of immediate delivery, but this allows the system to handle 100K notifications/second."

**Key Phrases:**
* "Message queues for async processing and scalability"
* "Batch notifications to reduce provider API calls"
* "At-least-once delivery with idempotency"
* "Separate queues per channel for independent scaling"

---

## 11. Common Mistakes

* **Synchronous sending**: "We send notifications immediately"
  * Fix: Use async queues for scalability
* **No batching**: "One API call per notification"
  * Fix: Batch 100-1000 notifications to reduce API calls
* **Ignoring user preferences**: "Send to everyone"
  * Fix: Check opt-in and frequency limits before sending
* **No retry logic**: "If provider fails, notification is lost"
  * Fix: Retry with exponential backoff, dead letter queue
* **Single queue for everything**: "All notifications in one queue"
  * Fix: Separate queues per channel for isolation and scaling
* **Not tracking delivery**: "We don't know if it was delivered"
  * Fix: Log delivery status, track opens/clicks for emails
* **No rate limiting**: "We send unlimited notifications"
  * Fix: Respect provider limits, implement throttling
* **Storing all device tokens in one DB**: "Single database for tokens"
  * Fix: Shard by user_id, cache hot tokens

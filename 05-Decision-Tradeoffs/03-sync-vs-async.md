# Sync vs Async

## 1. Problem Statement

When do we process requests synchronously (wait for completion) versus asynchronously (fire and forget) in distributed systems?

---

## 2. Clarifying Questions

* What's the processing time? (Milliseconds vs seconds/minutes)
* What's the user expectation? (Immediate response vs eventual)
* What happens on failure? (Retry vs user notification)
* What's the dependency? (Blocking vs independent)

---

## 3. Requirements

### Functional

* Process requests reliably
* Handle failures gracefully
* Provide user feedback
* Support high throughput

### Non-Functional

* Low latency for user-facing operations
* High availability
* Scalability
* Cost-effective

---

## 4. Scale Assumptions

* **Sync Use Case**: 10K QPS, <100ms processing, user waits
* **Async Use Case**: 1K QPS, 10s-1hour processing, background job
* Failure rate: 1-5%
* Retry strategy needed

---

## 5. High-Level Architecture

**Synchronous Flow**
```
Client → API → Process → Wait → Response
```

**Asynchronous Flow**
```
Client → API → Queue → Return immediately
                  ↓
              Worker → Process → Notify
```

---

## 6. Core Components

**Synchronous:**
* **API Service**: Handles request end-to-end
* **Database**: Immediate writes
* **External APIs**: Blocking calls
* **Response**: Return result immediately

**Asynchronous:**
* **Message Queue**: Buffers tasks
* **Worker Pool**: Processes tasks
* **Status Endpoint**: Check job status
* **Notification**: Webhook/email when done

---

## 7. Data Flow

**Synchronous Flow**
1. Client → Sends request to API
2. API → Validates request
3. API → Processes request (DB write, computation)
4. API → Waits for completion
5. API → Returns result to client
6. Client → Receives response (success or error)

**Asynchronous Flow**
1. Client → Sends request to API
2. API → Validates request
3. API → Creates job, enqueues to message queue
4. API → Returns job ID immediately (<100ms)
5. Client → Receives job ID
6. Worker → Polls queue, picks up job
7. Worker → Processes job (may take seconds/minutes)
8. Worker → Updates job status (DB)
9. Client → Polls status endpoint or receives webhook
10. Client → Gets final result

**Hybrid Flow (Optimistic Response)**
1. Client → Sends request
2. API → Validates, starts async processing
3. API → Returns optimistic response immediately
4. Background → Completes processing
5. If failure → Notify user, rollback if needed

---

## 8. Bottlenecks

**Synchronous Bottlenecks:**
* **Blocking Operations**: Slow DB query blocks request
  * Solution: Use async for slow operations, or optimize queries
* **Resource Exhaustion**: Threads blocked waiting
  * Solution: Use async, or increase thread pool
* **Cascading Failures**: One slow service blocks all
  * Solution: Timeouts, circuit breakers, or async
* **Poor User Experience**: User waits for slow operation
  * Solution: Use async, return immediately

**Asynchronous Bottlenecks:**
* **Queue Backlog**: Jobs pile up faster than processing
  * Solution: Scale workers, or throttle producers
* **Status Polling**: Clients poll too frequently
  * Solution: Use webhooks, or exponential backoff polling
* **Job Failures**: Need retry and dead letter queue
  * Solution: Implement retry logic, DLQ for failed jobs
* **Complexity**: More moving parts
  * Solution: Use managed queues (SQS, RabbitMQ)

---

## 9. Trade-offs

| Aspect | Synchronous | Asynchronous | When to Use |
|--------|-------------|--------------|-------------|
| **Latency** | ⚠️ Blocks on slowest operation | ✅ Returns immediately | Async for slow operations |
| **Simplicity** | ✅ Simple, linear flow | ❌ Complex (queues, workers) | Sync for simple operations |
| **User Experience** | ⚠️ User waits | ✅ Immediate feedback | Async for long operations |
| **Error Handling** | ✅ Immediate feedback | ❌ Delayed, complex | Sync for critical errors |
| **Throughput** | ❌ Limited by processing time | ✅ High (decoupled) | Async for high throughput |
| **Resource Usage** | ❌ Threads blocked | ✅ Efficient (event-driven) | Async for resource efficiency |
| **Debugging** | ✅ Easier (linear flow) | ❌ Harder (distributed) | Sync for easier debugging |
| **Consistency** | ✅ Immediate consistency | ⚠️ Eventual consistency | Sync for strong consistency |

**Decision Matrix:**

**Choose Synchronous when:**
* Fast operations (<100ms)
* User needs immediate result
* Strong consistency required
* Simple error handling needed
* Low complexity preferred

**Choose Asynchronous when:**
* Slow operations (>1s)
* User can wait for result
* High throughput needed
* Operations can be retried
* Decoupling services needed

**Common Patterns:**
* **User-facing APIs**: Sync for fast, async for slow
* **Background Jobs**: Always async (email, reports, processing)
* **Payment Processing**: Sync (need immediate confirmation)
* **Image Processing**: Async (takes seconds, user can wait)

---

## 10. How to Explain This in an Interview

> "For user login, I use synchronous processing because we need to return the auth token immediately and validate credentials is fast (<50ms). For sending emails, I use asynchronous processing because email delivery takes 1-5 seconds and the user doesn't need to wait. The API enqueues the email job and returns immediately, while a worker processes it in the background."

**Alternative:**
> "I use a hybrid approach: the API validates the request synchronously and returns a job ID immediately, then processes the heavy computation asynchronously. This gives users immediate feedback while handling slow operations in the background."

**Key Points:**
* Always mention the processing time that drives the decision
* Explain user experience impact
* Acknowledge complexity trade-off

---

## 11. Common Mistakes

* **Sync for everything**: "All requests are synchronous"
  * Fix: Use async for slow operations (>1s)
* **Async for everything**: "All requests go to queue"
  * Fix: Use sync for fast, user-facing operations
* **No status tracking**: "User doesn't know if job completed"
  * Fix: Provide job status endpoint or webhooks
* **No retry logic**: "Failed jobs are lost"
  * Fix: Implement retry with exponential backoff, DLQ
* **Blocking on async**: "We wait for queue processing"
  * Fix: Return immediately, process in background
* **Ignoring user experience**: "User waits 30 seconds"
  * Fix: Use async, return immediately, notify when done

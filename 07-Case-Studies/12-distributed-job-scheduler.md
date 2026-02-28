# ⏱️ Distributed Job Scheduler (Distributed Cron)

> **Staff-Signal**: Can you ensure a job runs exactly once across a cluster of 1,000 servers without a single point of failure?

---

## 1. Problem Statement
Design a system that allows users to schedule tasks (jobs) to run at a specific time or on a recurring basis.

---

## 2. Clarifying Questions
*   **Scale**: How many jobs? (100M total, 10K per second execution).
*   **Precision**: Does it have to run at the *exact* millisecond? (No, < 1s delay is fine).
*   **Execution Guarantee**: At-least-once or Exactly-once? (Exactly-once for billing, At-least-once for others).

---

## 3. Requirements
### Functional
*   Submit a job: `scheduler.addJob(taskId, time, payload)`.
*   Execute jobs at the scheduled time.
*   Retry failed jobs with exponential backoff.
*   Check job status.

### Non-Functional
*   **High Scalability**: Handle millions of scheduled tasks.
*   **High Availability**: No single point of failure.
*   **Durable**: Once a job is scheduled, it must not be lost.

---

## 4. Capacity Estimation (Worked Math)
*   **Jobs per day**: 100M.
*   **Storage**: Job metadata (id, schedule, payload, status) ~ 500 bytes. 100M * 500B = **50GB**.
*   **Execution Rate**: 100M / 86400s ≈ **1,200 jobs/sec avg**. Assume peak is **10K jobs/sec**.

---

## 5. API Design
### `POST /v1/jobs`
```json
{
  "job_type": "email_send",
  "schedule_time": "2026-12-31T23:59:59Z",
  "payload": {"user_id": 123, "body": "..."},
  "retries": 3
}
```

---

## 6. Data Model
*   **Table: JobsTable** (Postgres/DynamoDB)
    *   `job_id` (PK)
    *   `execution_time` (Indexed - Crucial!)
    *   `status` (SCHEDULED, RUNNING, COMPLETED, FAILED)
    *   `shard_id` (For horizontal scaling)

---

## 7. High-Level Architecture
```
User → API Gateway → Job Store (DB)
                        │
      ┌─────────────────┴─────────────────┐
      ↓                                   ↓
  Job Masters (Leaders)           Workers (Task Execution)
 (Scanning the DB)                (Stateless)
```

---

## 8. Component Deep Dive: The Scanner & Distributed Lock
The bottleneck is "Who picks up the next job?"
*   **Naive Approach**: All workers scan `SELECT * FROM Jobs WHERE time < now()`. 
    *   **Result**: 1,000 workers try to run the same job. Outage.
*   **Staff Solution**: **Horizontal Sharding & Leader Election**.
    1.  Divide jobs into **100 Shards**.
    2.  Use **ZooKeeper or etcd** to assign Shards to Masters.
    3.  Master for Shard 1 is the ONLY one who scans jobs for that shard.
    4.  Master pushes the Job ID to a **Message Queue** (Kafka/RabbitMQ) for Workers to execute.

---

## 9. Data Flow (Execution)
1.  **Master** finds job J1 in Shard X.
2.  **Master** updates J1 status to `RUNNING` in DB using a **Fencing Token** or Optimistic Lock.
3.  **Master** publishes J1 to a **Message Queue**.
4.  **Worker** picks up the task, executes, and updates status to `COMPLETED`.

---

## 10. Exactly-Once Execution
This is the hardest part of the interview.
1.  Master sends task → Worker (Network fails).
2.  Worker executes task → Updates DB (DB fails).
*   **Conclusion**: Distributed Exactly-Once is impossible. We aim for **At-Least-Once** and make the Worker's logic **Idempotent**.
*   **Interview Tip**: Mention **Idempotency Keys**. If the job is "Charge User $10", the charging service must ignore the same `job_id` if it's already been processed.

---

## 11. Scaling Strategy
*   **Hierarchical Timing Wheels**: For extremely high-precision/high-volume scheduling, using a database `index` on time is slow. A **Timing Wheel** (in-memory) allows O(1) scheduling and execution.

---

## 12. Failure Scenarios
*   **Master Dies**: ZooKeeper detects the loss of heartbeat and reassigns the shards to a standby master.
*   **Worker Dies mid-job**: The Master monitors the `RUNNING` status. If a job is `RUNNING` for too long, it's timed out and re-queued.

---

## 13. Tradeoffs

| Choice | Pro | Con |
| :--- | :--- | :--- |
| **Pull-based (Scanner)** | Robust, simple to reason about | Latency from scan intervals |
| **Push-based (Timing Wheel)** | Sub-millisecond precision | Complex to keep persistent in-memory state |

---

## 14. Monitoring Strategy
*   **Job Lag**: `Now() - scheduled_execution_time`. If lag > 10s, scale Masters/Workers.
*   **Failure Rate**: Percentage of jobs in `FAILED` status.

---

## 15. The Interview Narrative
> "To build a resilient job scheduler, I’ll prioritize horizontal scalability by sharding the job metadata across multiple database partitions. Instead of a global scan which causes contention, I’ll use a consensus-based service like ZooKeeper to assign specific shards to Master nodes. These nodes will utilize a 'Lease' mechanism to fetch and delegate tasks to stateless workers via an idempotent message queue. This ensures that even if a Master fails, its shard is reassigned within seconds, providing high availability with a strong at-least-once delivery guarantee."

---

## 16. Follow-up Questions
1.  **"How do you handle 'Heavy' jobs vs 'Light' jobs?"** (Answer: Separate queues/worker pools).
2.  **"How do you handle Million-member fan-out updates?"** (Answer: Batching and recursive scheduling).

---

## 17. Common Mistakes
1.  **Centralised cron**: Running it on one big server.
2.  **Assuming DB index is enough**: Scanning a DB with 1B rows every second will kill the IO.
3.  **No Idempotency**: Charging a customer twice because the worker retried.

# 🕵️ The Staff-Level Secret Scorecard

> **"What the interviewer is writing in their internal feedback notes while you are drawing."**

---

## 1. Scoping Discipline (The First 5 Minutes)
*   **L4 Signal**: Asks basic questions and dives in.
*   **L5 Signal**: Asks about scale, throughput, and users.
*   **L6+ Signal**: **Aggressively prioritizes**. "We have 45 minutes. I will focus on the Distributed Ledger and the Exactly-Once challenge, as the 'Mobile UI' and 'Login' are standard. Does that work for you?"

---

## 2. Quantitative Reasoning (Back-of-Envelope)
*   **Hire**: Does the math.
*   **Strong Hire**: **Interprets the math**. "10 TB/day tells me we cannot use a single RDS instance. We need a sharded storage engine like Vitess or a NoSQL solution with a 7-day TTL policy to keep the hot-set manageable."

---

## 3. Failure-First Thinking
*   **Hire**: Adds a "Redundant Load Balancer" at the end.
*   **Strong Hire**: **Designs for failure from the start**. "What happens if our primary region's DNS is poisoned? We need a secondary BGP Anycast record that fails over to a static status page."

---

## 4. Communication Pacing
*   **Red Flag**: Monologuing for 15 minutes.
*   **L6+ Signal**: **Checking in constantly**. "I've outlined the high-level data flow. Should I go deep into the caching layer or the database sharding logic next?"

---

## 5. Intellectual Honesty
*   **The Trap**: When the interviewer asks about a technology you don't know well.
*   **The Right Answer**: "I haven't used Kafka Streams in production, but based on my knowledge of message processing, I would look for a solution that handles **Checkpointing** and **Windowing**. Here is how I'd reason about it..."

---

## 📊 The Internal Feedback Rubric

| Competency | L4 (Junior/Mid) | L5 (Senior) | L6 (Staff) |
| :--- | :--- | :--- | :--- |
| **Ambiguity** | Needs a clear prompt. | Handles open-ended Qs. | Defines the problem scope. |
| **Complexity** | Solves for 1,000 users. | Solves for 1M users. | Solves for 1B users + Failures. |
| **Tradeoffs** | Doesn't know them. | Explains them when asked. | **Leads** with them. |
| **Breadth** | One solution. | Two solutions. | Best solution + Operational cost. |

---

## 💬 The "Staff" Phrasebook
*   "Crucially, this design favors **Availability over Consistency** because..."
*   "Operational cost is a concern here; managed S3 is better than running our own Ceph cluster."
*   "To minimize the **Blast Radius** of a deployment error, we will use..."
*   "This is an **L-7 bottleneck**, so a simple Round-Robin LB won't suffice."

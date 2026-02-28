# 🌪️ Chaos Engineering (Designing for Antifragility)

> **Staff-Signal**: How do you prove your multi-region design actually works before the outage happens?

---

## 1. What is Chaos Engineering?
Chaos Engineering is the discipline of experimenting on a system in order to build confidence in the system’s capability to withstand turbulent conditions in production.
- **Goal**: Find vulnerabilities (e.g., a hidden dependency, a missing timeout) before they cause an outage.

---

## 2. The Principles of Chaos
1.  **Steady State**: Define what "Normal" looks like (e.g., 99.9% of requests < 100ms).
2.  **Hypothesis**: "If we kill the Primary Redis node, the Replica will take over in < 5s with no data loss."
3.  **Blast Radius**: Start small. Don't break the whole site. Inject failure for 1% of traffic in one region.
4.  **Verification**: If the hypothesis is wrong, stop the experiment and fix the design.

---

## 3. Common Fault Injections
In a Staff interview, mention these when asked "How do you test this?":
- **Network Latency**: Add 200ms delay between Service A and B.
- **Process Kill**: Randomly kill a microservice instance.
- **Resource Exhaustion**: Fill the disk or max out CPU on a node.
- **Dependency Failure**: Simulate S3 or a Third-Party API being down.

---

## 4. The "GameDay" Strategy
At MAANG, teams run "GameDays" where one engineer intentionally breaks something in a staging environment, and the rest of the team must identify and fix it using only their monitoring tools.

---

## ⚖️ Tradeoffs

| Topic | Pro | Con |
| :--- | :--- | :--- |
| **Testing in Staging** | Safe | Not realistic (traffic patterns differ) |
| **Testing in Production** | 100% realistic | Risky (customer impact) |

---

## 💬 How to use this in an Interview

> "To validate the resilience of my global payment architecture, I’ll implement a **Chaos Engineering** pipeline. We’ll perform regular **Fault Injection** experiments, such as simulating cross-region network partitions and primary database failovers. This allows us to quantify our **Blast Radius** and ensure our automated recovery scripts (e.g., Prometheus alerts and Auto-scaling) trigger correctly before a real incident occurs."

### Follow-up question: "What is a 'Circuit Breaker' and why is it related to Chaos?"
**Correct Answer**: "A Circuit Breaker stops calls to a failing service to prevent a cascading failure. Chaos engineering helps us set the correct **Thresholds** for that circuit breaker by simulating varied latency and error rates."

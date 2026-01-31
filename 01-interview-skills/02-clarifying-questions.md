# How to Ask Clarifying Questions

## 1. Problem Statement

What questions should you ask before designing a system, and how do you ask them like a Staff Engineer?

---

## 2. Clarifying Questions

* What's the scope? (Full system vs feature)
* Who's the interviewer? (Hiring manager vs peer)
* What's their expectation? (High-level vs detailed)

---

## 3. Requirements

### Functional

* Understand problem fully
* Identify ambiguous requirements
* Discover hidden constraints
* Align on success criteria

### Non-Functional

* Ask in 3-5 minutes
* Show structured thinking
* Don't ask obvious questions
* Focus on decisions that matter

---

## 4. Scale Assumptions

* 5-minute clarification phase
* 3-5 targeted questions
* Each question leads to a design decision
* No fishing for answers

---

## 5. High-Level Architecture

**Question Categories**
```
Scale Questions → Capacity planning
Feature Questions → Scope definition
Constraint Questions → Trade-off decisions
Use Case Questions → Data model design
```

---

## 6. Core Components

* **Scale Questions**: Users, QPS, data size
* **Feature Questions**: What's in/out of scope
* **Constraint Questions**: Latency, consistency, availability
* **Use Case Questions**: Read/write patterns, access patterns
* **Geographic Questions**: Global vs regional

---

## 7. Data Flow

**Question Flow**
1. **Scale First**: "How many users? What's the QPS?"
2. **Features**: "Do we need X? Is Y required?"
3. **Constraints**: "What's the latency requirement? Availability?"
4. **Use Cases**: "What's the read/write ratio? Access patterns?"
5. **Geographic**: "Global or single region?"

**Example Sequence:**
- "What's the expected scale? Users and QPS?"
- "What features are must-haves vs nice-to-haves?"
- "What are the latency and availability requirements?"
- "What's the read vs write ratio?"
- "Is this global or regional?"

---

## 8. Bottlenecks

* **Asking too many questions**: Wasting time
  * Solution: 3-5 focused questions max
* **Asking obvious questions**: "Do we need a database?"
  * Solution: Ask questions that affect design decisions
* **Not listening**: Asking questions they already answered
  * Solution: Listen, take notes
* **Fishing for answers**: "Should we use Redis?"
  * Solution: Ask about requirements, not solutions

---

## 9. Trade-offs

| Question Type | Value | Risk | When to Ask |
|---------------|-------|------|-------------|
| **Scale Questions** | Critical for design | May not know | Always ask |
| **Feature Questions** | Defines scope | May over-scope | Early |
| **Constraint Questions** | Drives trade-offs | May be flexible | Always ask |
| **Use Case Questions** | Affects data model | May be obvious | If unclear |
| **Geographic Questions** | Affects architecture | May not matter | If global scale |

**Key Principle**: Every question should lead to a design decision.

---

## 10. How to Explain This in an Interview

> "Before I start designing, let me clarify a few things. What's the expected scale in terms of users and QPS? What are the latency and availability requirements? And what's the read vs write ratio? These will help me make the right trade-offs."

**Good Questions (Lead to Decisions):**
* "What's the expected QPS?" → Determines if we need caching, sharding
* "What's the latency requirement?" → Determines if we need CDN, cache
* "What's the read/write ratio?" → Determines DB strategy (read replicas vs sharding)
* "Do we need strong consistency?" → Determines DB choice (SQL vs NoSQL)
* "Is this global?" → Determines if we need multi-region

**Bad Questions (Don't Help):**
* "Should we use Redis?" → Ask about latency requirements instead
* "Do we need a database?" → Obvious, don't ask
* "What technology should I use?" → You decide based on requirements

---

## 11. Common Mistakes

* **Asking solution questions**: "Should we use microservices?"
  * Fix: Ask about requirements: "What's the team structure? Independent deployability needed?"
* **Too many questions**: Asking 10+ questions
  * Fix: 3-5 focused questions, move on
* **Not taking notes**: Forgetting answers
  * Fix: Write down key numbers (users, QPS, latency)
* **Asking obvious things**: "Do users need to log in?"
  * Fix: Only ask if it affects design significantly
* **Not following up**: Getting vague answers
  * Fix: "So if it's 100M users, that's roughly 10K QPS average, 100K peak?"

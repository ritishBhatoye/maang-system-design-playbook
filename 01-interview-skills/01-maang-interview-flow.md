# MAANG Interview Flow

## 1. Problem Statement

How do you structure a 45-minute system design interview to demonstrate senior engineering judgment?

---

## 2. Clarifying Questions

* What's the scope? (Full system vs component)
* What's the timeline? (45 min vs 60 min)
* What's the interviewer's role? (Hiring manager vs peer)

---

## 3. Requirements

### Functional

* Understand problem fully
* Design scalable solution
* Communicate clearly
* Handle follow-ups

### Non-Functional

* Time management (45 min)
* Show trade-off thinking
* Demonstrate system thinking
* Stay calm under pressure

---

## 4. Scale Assumptions

* 45-minute interview
* 5 min: Clarifying questions
* 10 min: Requirements + scale
* 20 min: Architecture + components
* 5 min: Trade-offs + deep dive
* 5 min: Wrap-up

---

## 5. High-Level Architecture

**Interview Flow**
```
1. Clarify (5 min)
   ↓
2. Requirements + Scale (10 min)
   ↓
3. High-Level Design (5 min)
   ↓
4. Deep Dive Components (15 min)
   ↓
5. Trade-offs + Edge Cases (5 min)
   ↓
6. Wrap-up (5 min)
```

---

## 6. Core Components

* **Opening**: Restate problem, show understanding
* **Clarifying Questions**: Ask 3-5 targeted questions
* **Requirements**: Functional + non-functional, explicit
* **Scale**: Numbers matter (users, QPS, storage)
* **Architecture**: Start high-level, then drill down
* **Trade-offs**: Always explain WHY
* **Edge Cases**: Show you think about failures

---

## 7. Data Flow

**Phase 1: Clarification (0-5 min)**
- Restate problem in your words
- Ask: scale, features, constraints
- Example: "So we're building X for Y users, and Z is critical?"

**Phase 2: Requirements (5-15 min)**
- Functional: What features?
- Non-functional: Latency, availability, consistency
- Scale: Users, QPS, data size
- Write numbers down

**Phase 3: High-Level Design (15-20 min)**
- Draw boxes: Client, LB, Services, DB, Cache
- Show data flow
- Explain at 30,000 feet first

**Phase 4: Deep Dive (20-35 min)**
- Pick 2-3 components to detail
- Database schema, API design, algorithms
- Show you can go deep

**Phase 5: Trade-offs (35-40 min)**
- "I chose X because Y, but trade-off is Z"
- Show you understand alternatives
- Justify decisions

**Phase 6: Wrap-up (40-45 min)**
- Summarize key decisions
- Mention what you'd do next (monitoring, scaling)
- Ask if they want to dive deeper

---

## 8. Bottlenecks

* **Running out of time**: Spending too long on one component
  * Solution: Time-box each phase, move on
* **Not asking questions**: Jumping to solution
  * Solution: Always clarify first (5 min minimum)
* **Over-engineering**: Adding unnecessary complexity
  * Solution: Start simple, add complexity when needed
* **Not justifying decisions**: "We use Redis"
  * Solution: Always explain WHY: "We use Redis for sub-10ms latency"
* **Ignoring scale**: "We need a database"
  * Solution: Always mention scale: "We need a sharded NoSQL DB for 1M QPS"

---

## 9. Trade-offs

| Approach | Pros | Cons | When to Use |
|----------|------|------|-------------|
| **Start High-Level** | Shows system thinking | May miss details | Beginning of interview |
| **Deep Dive Early** | Shows expertise | May lose big picture | When interviewer asks |
| **Draw Diagrams** | Visual, clear | Takes time | Complex systems |
| **Talk Through** | Faster, flexible | Less structured | Simple systems |

**Key Principle**: Show you can think at multiple levels of abstraction.

---

## 10. How to Explain This in an Interview

> "I'll start by clarifying the requirements and scale, then propose a high-level architecture. Once we align on that, I'll deep dive into the critical components and explain the trade-offs I'm making. Sound good?"

**Phrases That Signal Senior Thinking:**
* "Let me clarify the scale first..."
* "I'm making a trade-off here: X for Y..."
* "The bottleneck will be..."
* "If we need to scale further, we could..."
* "The failure mode here is..."

---

## 11. Common Mistakes

* **Jumping to solution**: "We need microservices!"
  * Fix: Clarify first, then design
* **No numbers**: "Lots of users"
  * Fix: "100M users, 10K QPS"
* **Forgetting trade-offs**: "We use Redis"
  * Fix: "We use Redis for latency, trade-off is memory cost"
* **Over-complicating**: "We need 20 services"
  * Fix: Start with 3-4, scale when needed
* **Not handling failures**: "It just works"
  * Fix: "If DB fails, we serve from cache with stale data"
* **Ignoring interviewer cues**: Keep talking when they want to move on
  * Fix: Pause, check: "Should I continue or dive deeper here?"

# Architecture Basics

## 1. Problem Statement

What is system design, and what do MAANG interviewers actually expect from candidates?

---

## 2. Core Concepts

### What is System Design?

System design is the process of defining the architecture, components, modules, interfaces, and data flow for a system to satisfy specified requirements.

**In MAANG interviews, it's not about:**
- Memorizing every technology
- Drawing perfect diagrams
- Knowing every component

**It's about:**
- Making informed decisions under constraints
- Communicating trade-offs clearly
- Thinking like a senior engineer
- Justifying choices with data and reasoning

---

## 3. Core Principles

### Scalability
* **Horizontal**: Add more servers (preferred)
* **Vertical**: Make servers bigger (limited)
* **Key Question**: "How does this scale to 10x, 100x?"

### Reliability
* **Availability**: System is up and operational
* **Durability**: Data is not lost
* **Fault Tolerance**: System handles failures gracefully

### Performance
* **Latency**: Time to complete a request
* **Throughput**: Requests processed per second
* **Efficiency**: Resource usage (CPU, memory, bandwidth)

### Maintainability
* **Simplicity**: Easy to understand and modify
* **Modularity**: Components are independent
* **Observability**: System is monitorable and debuggable

---

## 4. Interview Expectations

### What Interviewers Evaluate

**Decision-Making (40%)**
* Can you make trade-offs?
* Do you justify choices?
* Can you prioritize under constraints?

**Communication (30%)**
* Can you explain clearly?
* Do you ask the right questions?
* Can you discuss trade-offs?

**Technical Depth (20%)**
* Do you understand components?
* Can you go deep when needed?
* Do you know when to stop?

**System Thinking (10%)**
* Can you see the big picture?
* Do you consider edge cases?
* Can you identify bottlenecks?

---

## 5. The 11-Step Framework

Every system design should follow this structure:

1. **Problem Statement** - Understand what you're building
2. **Clarifying Questions** - Ask 3-5 targeted questions
3. **Requirements** - Functional + Non-functional
4. **Scale Assumptions** - Numbers matter (users, QPS, storage)
5. **High-Level Architecture** - Start with boxes and arrows
6. **Core Components** - Detail key components
7. **Data Flow** - Show how requests flow
8. **Bottlenecks** - Identify and address
9. **Trade-offs** - Explain decisions explicitly
10. **How to Explain in Interview** - Practice your narrative
11. **Common Mistakes** - Learn from others' errors

---

## 6. How to Explain This in an Interview

> "System design is about making architectural decisions that balance scalability, reliability, performance, and cost. I start by understanding the problem, asking clarifying questions about scale and requirements, then propose a high-level architecture. I always explain my trade-offs explicitly - for example, choosing NoSQL for horizontal scalability even though we lose ACID transactions."

**Key Phrases:**
* "I'm making a trade-off here..."
* "The bottleneck will be..."
* "I optimize for X because..."
* "The alternative would be Y, but..."

---

## 7. Common Mistakes

* **Jumping to solution**: Drawing diagrams before understanding
  * Fix: Always clarify requirements first
* **No numbers**: "Lots of users" instead of "100M users"
  * Fix: Always quantify scale assumptions
* **Ignoring trade-offs**: "We use Redis" without explaining why
  * Fix: Always explain the decision and trade-off
* **Over-engineering**: "We need 20 microservices"
  * Fix: Start simple, add complexity when needed
* **Not handling failures**: "It just works"
  * Fix: Always consider failure modes and recovery

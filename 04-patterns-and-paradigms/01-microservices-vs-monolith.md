# Microservices vs Monolith

## 1. Problem Statement

When should we choose a microservices architecture over a monolithic architecture, and what are the real trade-offs?

---

## 2. Monolithic Architecture

### Characteristics
* **Single Deployment Unit**: All code in one application
* **Shared Database**: All services use same database
* **Tight Coupling**: Components depend on each other
* **Single Technology Stack**: One language/framework

### Pros
* **Simple**: Easy to develop, test, deploy
* **Performance**: No network calls between components
* **Transactions**: Easy ACID transactions across components
* **Debugging**: Single codebase, easier to trace

### Cons
* **Scaling**: Must scale entire application
* **Deployment**: One change requires full deployment
* **Technology Lock-in**: Hard to adopt new technologies
* **Team Coordination**: Large teams step on each other

---

## 3. Microservices Architecture

### Characteristics
* **Independent Services**: Each service is separate
* **Separate Databases**: Each service has its own database
* **Loose Coupling**: Services communicate via APIs
* **Technology Diversity**: Different services can use different tech

### Pros
* **Independent Scaling**: Scale services independently
* **Independent Deployment**: Deploy services separately
* **Technology Freedom**: Use best tool for each service
* **Team Autonomy**: Teams own services independently

### Cons
* **Complexity**: More moving parts, network calls
* **Distributed Transactions**: Hard to coordinate across services
* **Operational Overhead**: More services to monitor, deploy
* **Network Latency**: Inter-service calls add latency

---

## 4. Decision Framework

### Choose Monolith When:
* **Small Team**: <10 engineers
* **Simple Application**: Clear, well-defined boundaries
* **Rapid Iteration**: Need to move fast, validate product
* **Strong Consistency**: Need ACID transactions across components
* **Low Traffic**: Don't need independent scaling yet

### Choose Microservices When:
* **Large Team**: >50 engineers, multiple teams
* **Different Scaling Needs**: Some services need more scale
* **Technology Diversity**: Need different tech for different services
* **Independent Deployment**: Need to deploy services separately
* **Clear Boundaries**: Well-defined service boundaries

---

## 5. Migration Path

### Start Monolith, Evolve to Microservices
1. **Phase 1**: Build monolith, learn domain
2. **Phase 2**: Identify service boundaries
3. **Phase 3**: Extract services one by one (strangler pattern)
4. **Phase 4**: Fully microservices

**Key Insight**: Don't start with microservices. Start simple, extract when needed.

---

## 6. Service Boundaries

### How to Identify
* **Business Capabilities**: Each service = one business capability
* **Data Ownership**: Each service owns its data
* **Team Structure**: Conway's Law (architecture mirrors organization)
* **Change Frequency**: Services that change together stay together

### Anti-Patterns
* **Database per Service**: Good, but don't overdo it
* **Shared Database**: Breaks service boundaries
* **Chatty Services**: Too many small services, too much communication
* **Distributed Monolith**: Microservices that are tightly coupled

---

## 7. Communication Patterns

### Synchronous (REST/gRPC)
* **Use Case**: Request-response, need immediate answer
* **Trade-off**: Tight coupling, but simple

### Asynchronous (Message Queue)
* **Use Case**: Fire-and-forget, eventual consistency OK
* **Trade-off**: Loose coupling, but more complex

### Event-Driven
* **Use Case**: Multiple consumers, event sourcing
* **Trade-off**: Decoupled, but eventual consistency

---

## 8. Data Management

### Database per Service
* Each service has its own database
* **Benefit**: Independent scaling, technology choice
* **Challenge**: Cross-service queries, data consistency

### Saga Pattern
* Distributed transactions across services
* **Types**: Choreography (events) or Orchestration (coordinator)
* **Trade-off**: Complex, but enables distributed transactions

### API Composition
* Aggregator service calls multiple services
* **Use Case**: Need data from multiple services
* **Trade-off**: Network calls, but keeps services independent

---

## 9. How to Explain This in an Interview

> "I start with a monolith because it's simpler and faster to build. Once we have clear service boundaries and different scaling needs, I extract services using the strangler pattern. I define service boundaries by business capabilities - each service owns its data and represents one business function. For communication, I use REST for synchronous calls and message queues for async operations. The key trade-off is complexity vs flexibility: microservices add operational complexity but enable independent scaling and deployment."

**Key Points:**
* Start simple (monolith), evolve when needed
* Service boundaries = business capabilities
* Database per service (no shared database)
* Choose communication pattern based on needs

---

## 10. Common Mistakes

* **Microservices from day one**: "We need microservices"
  * Fix: Start monolith, extract when boundaries are clear
* **Shared database**: "Services share same database"
  * Fix: Each service has its own database
* **Too many services**: "One service per function"
  * Fix: Group related functions, avoid chatty services
* **Synchronous everything**: "All services call each other directly"
  * Fix: Use async messaging for decoupling
* **No service boundaries**: "Services are tightly coupled"
  * Fix: Define clear boundaries, own data independently

# Common Architecture Patterns (Diagrams as Text)

## 1. Problem Statement

How do we represent common system design architectures in a text format that's interview-friendly?

---

## 2. Three-Tier Architecture

```
┌─────────────┐
│   Client    │
│  (Browser)  │
└──────┬──────┘
       │ HTTP/HTTPS
       ↓
┌─────────────────┐
│  Load Balancer  │
└──────┬──────────┘
       │
       ├──────────┬──────────┐
       ↓          ↓          ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│  App     │ │  App     │ │  App     │
│ Server 1 │ │ Server 2 │ │ Server N │
└────┬─────┘ └────┬─────┘ └────┬─────┘
     │            │            │
     └────────────┼────────────┘
                  ↓
         ┌─────────────────┐
         │    Database     │
         │  (Read Replicas)│
         └─────────────────┘
```

**When to Use:**
* Standard web applications
* Clear separation of concerns
* Easy to scale horizontally (app tier)

---

## 3. Microservices Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ↓
┌─────────────────┐
│   API Gateway   │
└──────┬──────────┘
       │
   ┌───┴───┬──────────┬──────────┐
   ↓       ↓          ↓          ↓
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ User │ │ Order│ │Payment│ │Email │
│Service│ │Service│ │Service│ │Service│
└───┬──┘ └───┬──┘ └───┬──┘ └───┬──┘
    │        │        │        │
    ↓        ↓        ↓        ↓
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│UserDB│ │OrderDB│ │PayDB │ │Queue │
└──────┘ └──────┘ └──────┘ └──────┘
```

**When to Use:**
* Large teams, independent deployment
* Different scaling requirements per service
* Technology diversity needed

---

## 4. Event-Driven Architecture

```
┌──────────┐
│  Service │
│     A    │
└────┬─────┘
     │ Publishes Event
     ↓
┌─────────────────┐
│  Message Queue  │
│   (Kafka/SQS)   │
└────┬────────────┘
     │
     ├──────────┬──────────┐
     ↓          ↓          ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│ Service  │ │ Service  │ │ Service  │
│    B     │ │    C     │ │    D     │
│(Consumer)│ │(Consumer)│ │(Consumer)│
└──────────┘ └──────────┘ └──────────┘
```

**When to Use:**
* Loose coupling between services
* Asynchronous processing
* High throughput event processing

---

## 5. CQRS (Command Query Responsibility Segregation)

```
┌─────────────┐
│   Client    │
└───┬─────┬───┘
    │     │
Write    Read
    │     │
    ↓     ↓
┌──────┐ ┌──────┐
│Write │ │ Read │
│  DB  │ │  DB  │
│(SQL) │ │(NoSQL)│
└───┬──┘ └───┬──┘
    │        │
    │        │ Sync
    └────────┘
```

**When to Use:**
* Read and write patterns differ significantly
* Need to optimize reads and writes independently
* Event sourcing scenarios

---

## 6. Cache-Aside Pattern

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  App Server │
└───┬─────┬───┘
    │     │
    │ 1. Check Cache
    ↓     │
┌─────────┐ │
│  Cache  │ │ Miss
│ (Redis) │ │
└─────────┘ │
            ↓
      ┌──────────┐
      │ Database │
      └──────────┘
            │
            │ 2. Query DB
            │ 3. Store in Cache
            ↓
      ┌─────────┐
      │  Cache  │
      └─────────┘
```

**When to Use:**
* Read-heavy workloads
* Can tolerate eventual consistency
* Want to reduce database load

---

## 7. Write-Through Cache

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ Write
       ↓
┌─────────────┐
│  App Server │
└──────┬──────┘
       │
       ├──────────┬──────────┐
       ↓          ↓          ↓
┌─────────┐  ┌──────────┐
│  Cache  │  │ Database │
│ (Redis) │  │          │
└─────────┘  └──────────┘
    │            │
    └────────────┘
    Write to both
    (synchronous)
```

**When to Use:**
* Need cache and DB always consistent
* Write-heavy with frequent reads
* Can accept write latency

---

## 8. Database Sharding

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  App Server │
└──────┬──────┘
       │ Hash(user_id)
       │
   ┌───┴───┬──────────┬──────────┐
   ↓       ↓          ↓          ↓
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Shard │ │Shard │ │Shard │ │Shard │
│  1   │ │  2   │ │  3   │ │  N   │
│(0-25%)│ │(25-50%)│ │(50-75%)│ │(75-100%)│
└──────┘ └──────┘ └──────┘ └──────┘
```

**When to Use:**
* Need horizontal write scaling
* Data can be partitioned by key
* Single database can't handle load

---

## 9. Read Replicas

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  App Server │
└───┬─────┬───┘
    │     │
Write   Read
    │     │
    ↓     ├──────────┬──────────┐
┌─────────┐          ↓          ↓          ↓
│Primary  │    ┌──────────┐ ┌──────────┐ ┌──────────┐
│  DB     │───→│  Read    │ │  Read    │ │  Read    │
│(Writes) │    │ Replica 1│ │ Replica 2│ │ Replica N│
└─────────┘    └──────────┘ └──────────┘ └──────────┘
    │
    │ Replication (async)
    │
    └──────────────┘
```

**When to Use:**
* Read-heavy workloads
* Can tolerate eventual consistency on reads
* Need to scale reads horizontally

---

## 10. CDN Architecture

```
┌─────────────┐
│User (Asia)  │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│ CDN Edge    │
│  (Asia)     │
└──────┬──────┘
       │ Cache Hit → Return
       │ Cache Miss
       ↓
┌─────────────┐
│Origin Server│
│   (US)      │
│  (S3/Storage)│
└─────────────┘
```

**When to Use:**
* Global user base
* Static or semi-static content
* Need low latency worldwide

---

## 11. How to Explain These in an Interview

> "I'll use a three-tier architecture with load balancers, stateless app servers, and a database with read replicas. This allows horizontal scaling of the app tier and read scaling via replicas. For caching, I use cache-aside pattern with Redis to reduce database load for read-heavy workloads."

**Key Points:**
* Start with high-level (three-tier, microservices)
* Add patterns as needed (caching, sharding, CDN)
* Explain why each pattern fits the use case
* Draw incrementally, don't over-complicate initially

---

## 12. Common Mistakes

* **Over-engineering**: "We need microservices, CQRS, event sourcing..."
  * Fix: Start simple, add complexity when needed
* **Not explaining the diagram**: Just drawing boxes
  * Fix: Explain each component and data flow
* **Missing data flow**: Static diagram
  * Fix: Show how requests flow through the system
* **No scale indicators**: "We have servers"
  * Fix: Mention numbers: "10 app servers, 3 DB replicas"

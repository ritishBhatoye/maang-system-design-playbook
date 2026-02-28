# 📊 Mermaid Diagram Templates (GitHub Native)

> **Pro Tip**: Use these Mermaid snippets in your GitHub issues, PRs, or Markdown files. They render natively on GitHub and are far better for "Live" collaboration than static images.

---

## 1. High-Level Three-Tier Architecture
Perfect for: Web apps, URL Shorteners, simple CRUD.

```mermaid
graph TD
    User((User))
    LB[Load Balancer]
    API1[API Server 1]
    API2[API Server 2]
    Redis[(Redis Cache)]
    DB[(Postgres DB)]
    S3[Object Storage]

    User --> LB
    LB --> API1
    LB --> API2
    API1 --> Redis
    API2 --> Redis
    API1 --> DB
    API2 --> DB
    API1 --> S3
```

---

## 2. Event-Driven (Kafka) Pipeline
Perfect for: Logging, Analytics, Notification systems.

```mermaid
graph LR
    P1[Producer A] --> K[(Kafka Cluster)]
    P2[Producer B] --> K
    K --> C1[Consumer 1]
    K --> C2[Consumer 2]
    C1 --> DB[(Cassandra)]
    C2 --> S3[S3 Data Lake]
```

---

## 3. Microservices & API Gateway
Perfect for: Uber, News Feed, E-commerce.

```mermaid
graph TB
    Client((Mobile App)) --> GW[API Gateway / BFF]
    GW --> US[User Service]
    GW --> OS[Order Service]
    GW --> PS[Payment Service]
    
    US --> UDB[(User SQL)]
    OS --> ODB[(Order SQL)]
    PS --> PDB[(Payment SQL)]

    OS -- Events --> Kafka[(Kafka)]
    Kafka --> Analytics[Analytics Service]
```

---

## 4. Multi-Region Active-Active
Perfect for: Global services, Disaster Recovery.

```mermaid
graph TB
    subgraph Region_US_East
    LB1[Global Accelerator] --> App1[App Tier]
    App1 --> DB1[(Primary DB)]
    end

    subgraph Region_EU_West
    LB2[Global Accelerator] --> App2[App Tier]
    App2 --> DB2[(Read Replica)]
    end

    DB1 -- Async Replication --> DB2
```

---

## 5. Sequence Diagram: OAuth2 Flow
Perfect for: Explaining Auth or complex cross-service handshakes.

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant AuthServer
    participant ResourceServer

    User->>Client: Click Login
    Client->>AuthServer: Request Authorization
    AuthServer->>User: Show Login Page
    User->>AuthServer: Provide Credentials
    AuthServer-->>Client: Authorization Code
    Client->>AuthServer: Exchange Code for Token
    AuthServer-->>Client: Access Token
    Client->>ResourceServer: Fetch Data with Token
    ResourceServer-->>Client: Protected Resource
```

---

## 🎨 Design System for Diagrams

To keep your repo looking premium, follow these Mermaid styling rules:

| Entity | Shape | Syntax |
| :--- | :--- | :--- |
| **User** | Circle | `User((Name))` |
| **Database** | Cylinder | `DB[(Name)]` |
| **Manual Action** | Choice | `Decision{?}` |
| **External API** | Rect | `API[Name]` |

---

## 💡 Visual Learning Strategy
1.  **Don't overcomplicate**: Start with 4-5 boxes. Only add complexity when asked.
2.  **Color Coding**: Use Mermaid's `classDef` to highlight "New" components vs "Legacy".
3.  **Flow Direction**: Use `graph TB` (Top to Bottom) for hierarchy and `graph LR` (Left to Right) for data pipelines.

# Visual Learning Guide

> **使用图表和可视化方式理解系统设计。**

---

## 1. 架构图类型

### 1.1 架构图 (Architecture Diagram)

```
┌─────────────┐     ┌─────────────┐
│   Client    │────▶│Load Balancer│
└─────────────┘     └──────┬──────┘
                          │
           ┌──────────────┼──────────────┐
           │              │              │
           ▼              ▼              ▼
     ┌──────────┐   ┌──────────┐   ┌──────────┐
     │ Service A│   │ Service B│   │ Service C│
     └────┬─────┘   └────┬─────┘   └────┬─────┘
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  ┌──────────┐
                  │ Database │
                  └──────────┘
```

**用途**: 展示系统组件和关系

### 1.2 流程图 (Flowchart)

```
开始 → 接收请求
  ↓
检查缓存?
  ├─ 是 → 返回缓存
  └─ 否 → 查询数据库
  ↓
更新缓存
  ↓
返回响应
  ↓
结束
```

**用途**: 展示处理流程

### 1.3 时序图 (Sequence Diagram)

```
Client      Service       DB
  │            │            │
  │─REQ────────▶│            │
  │            │─SELECT────▶│
  │            │◀───────────│
  │◀───────────│            │
  │            │            │
```

**用途**: 展示交互时序

### 1.4 状态机 (State Machine)

```
    ┌───────┐
    │ start │
    └──┬────┘
       │
       ▼
  ┌─────────┐    error    ┌──────┐
  │processing│ ──────────▶│ error│
  └────┬────┘             └──────┘
       │
    success
       │
       ▼
  ┌────────┐
  │  done  │
  └────────┘
```

**用途**: 展示状态变化

---

## 2. Mermaid 快速参考

### 2.1 流程图

```mermaid
graph TD
    A[Start] --> B{Decision}
    B -->|Yes| C[Action 1]
    B -->|No| D[Action 2]
    C --> E[End]
    D --> E
```

### 2.2 时序图

```mermaid
sequenceDiagram
    Alice->>Bob: Hello
    Bob-->>Alice: Hi there
```

### 2.3 状态图

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Processing: Start
    Processing --> Done: Complete
    Done --> [*]
```

### 2.4 类图

```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    class Animal {
        +String name
        +makeSound()
    }
```

---

## 3. 常见架构模式可视化

### 3.1 微服务架构

```mermaid
graph TB
    subgraph "API Gateway"
        GW[Gateway]
    end
    
    subgraph "Services"
        S1[User]
        S2[Order]
        S3[Payment]
    end
    
    subgraph "Data"
        DB1[(Users)]
        DB2[(Orders)]
        DB3[(Payments)]
    end
    
    GW --> S1
    GW --> S2
    GW --> S3
    
    S1 --> DB1
    S2 --> DB2
    S3 --> DB3
```

### 3.2 事件驱动

```mermaid
graph LR
    Producer[Producer] -->|Event| Queue[Message Queue]
    Queue -->|Subscribe| Consumer1[Consumer 1]
    Queue -->|Subscribe| Consumer2[Consumer 2]
```

### 3.3 CQRS

```mermaid
graph TB
    subgraph "Command Side"
        Cmd[Commands] --> Svc[Service]
        Svc --> CmdDB[(Write DB)]
    end
    
    subgraph "Query Side"
        Query[Queries] --> Svc2[Query Service]
        Svc2 --> ReadDB[(Read DB)]
    end
    
    CmdDB -.->|Sync| ReadDB
```

---

## 4. 容量规划可视化

### 4.1 QPS 分布

```
     ▲
     │                    ████
     │            ██    ████
     │     ██  ████  ██████
     │  ████████████████████
     └──────────────────────▶
       0   100  500  1000  QPS
```

### 4.2 延迟分布

```
     ▲
     │    ┌ P50
     │────┤
     │    │       ┌ P95
     │────┤───────┤
     │    │       │       ┌ P99
     │────┤───────┼───────┤
     │    │       │       │
     └────┴───────┴───────┴────▶
        10ms   50ms   200ms  Latency
```

---

## 5. 故障分析可视化

### 5.1 故障传播

```mermaid
graph LR
    A[DB 故障] --> B[服务 A 慢]
    B --> C[服务 B 超时]
    C --> D[服务 C 熔断]
    D --> E[整个系统不可用]
    
    style A fill:#f96
    style E fill:#f96
```

### 5.2 恢复曲线

```
Availability
    ▲
100%│                      ████
    │                 █████
 99%│            █████
    │       █████
 95%│███████
    │
    └──────────────────────────────▶
       恢复中     恢复完成
```

---

## 6. 画图技巧

### 6.1 颜色含义

| 颜色 | 含义 |
|------|------|
| 绿色 | 正常/健康 |
| 红色 | 故障/错误 |
| 黄色 | 警告/降级 |
| 蓝色 | 存储 |
| 灰色 | 外部系统 |

### 6.2 布局原则

- 从左到右，从上到下
- 核心组件放中间
- 相关组件放一起
- 分层清晰

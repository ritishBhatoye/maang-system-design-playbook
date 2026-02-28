# Data Model Examples

> **各类型系统的完整数据模型示例。**

---

## 1. 用户系统 (SQL)

```sql
-- 用户表
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    avatar_url VARCHAR(500),
    status VARCHAR(20) DEFAULT 'active',
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    INDEX idx_email (email),
    INDEX idx_status (status),
    INDEX idx_created (created_at)
);

-- 用户配置表
CREATE TABLE user_settings (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    language VARCHAR(10) DEFAULT 'en',
    timezone VARCHAR(50) DEFAULT 'UTC',
    notifications JSONB DEFAULT '{}',
    preferences JSONB DEFAULT '{}',
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 地址表（用户可能有多个地址）
CREATE TABLE addresses (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    type VARCHAR(20), -- 'billing', 'shipping'
    line1 VARCHAR(255),
    line2 VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(2),
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 2. 电商订单 (NoSQL - DynamoDB)

```json
// Orders 表
{
  "PK": "ORDER#12345",
  "SK": "METADATA",
  "order_id": "12345",
  "user_id": "user:abc",
  "status": "SHIPPED",
  "total": {
    "amount": 199.99,
    "currency": "USD"
  },
  "items_count": 3,
  "shipping_address": {
    "city": "San Francisco",
    "country": "US"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-16T14:20:00Z"
}

// Order Items (GSI)
{
  "PK": "ORDER#12345",
  "SK": "ITEM#prod_001",
  "order_id": "12345",
  "product_id": "prod_001",
  "quantity": 2,
  "price": {
    "amount": 49.99,
    "currency": "USD"
  }
}

// User Orders (GSI - 按用户查询订单)
{
  "PK": "USER#abc",
  "SK": "ORDER#12345",
  "order_id": "12345",
  "status": "SHIPPED",
  "total": 199.99,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**分区键设计**:
- PK: 订单 ID（高基数）
- GSI1: UserID + 订单状态（查询某用户的所有订单）
- GSI2: 创建时间（按时间范围查询）

---

## 3. 社交 Feed (Wide Column - Cassandra)

```sql
-- 用户时间线表
CREATE TABLE user_timeline (
    user_id UUID,
    posted_at TIMESTAMP,
    post_id UUID,
    author_id UUID,
    content TEXT,
    PRIMARY KEY (user_id, posted_at, post_id)
) WITH CLUSTERING ORDER BY (posted_at DESC, post_id DESC);

-- 全局时间线（趋势）
CREATE TABLE global_timeline (
    bucket INT,  -- 按小时分桶
    posted_at TIMESTAMP,
    post_id UUID,
    author_id UUID,
    content TEXT,
    likes_count INT,
    PRIMARY KEY (bucket, posted_at, post_id)
) WITH CLUSTERING ORDER BY (posted_at DESC);

-- 用户帖子索引
CREATE TABLE user_posts (
    user_id UUID,
    post_id UUID,
    posted_at TIMESTAMP,
    content TEXT,
    PRIMARY KEY (user_id, post_id)
);
```

---

## 4. 消息系统 (时间序列优化)

```sql
-- 消息表
CREATE TABLE messages (
    channel_id UUID,
    message_id TIMEUUID,
    user_id UUID,
    content TEXT,
    attachments JSONB,
    reactions JSONB,
    thread_id UUID,
    created_at TIMESTAMP,
    PRIMARY KEY (channel_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- 用户会话（用于未读计数）
CREATE TABLE user_channels (
    user_id UUID,
    channel_id UUID,
    last_read_message_id TIMEUUID,
    unread_count INT,
    PRIMARY KEY (user_id, channel_id)
);

-- 搜索索引（Elasticsearch）
{
  "index": "messages",
  "body": {
    "mappings": {
      "properties": {
        "message_id": { "type": "keyword" },
        "content": { "type": "text", "analyzer": "english" },
        "author": { "type": "keyword" },
        "channel_id": { "type": "keyword" },
        "created_at": { "type": "date" }
      }
    }
  }
}
```

---

## 5. 支付系统 (ACID + Audit)

```sql
-- 账户表
CREATE TABLE accounts (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    type VARCHAR(20), -- 'checking', 'savings', 'credit'
    balance DECIMAL(19,4) NOT NULL DEFAULT 0,
    currency VARCHAR(3) NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 交易表
CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    account_id UUID NOT NULL,
    type VARCHAR(20), -- 'credit', 'debit'
    amount DECIMAL(19,4) NOT NULL,
    balance_after DECIMAL(19,4) NOT NULL,
    reference_id UUID, -- 关联订单等
    description TEXT,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- 审计日志（不可变）
CREATE TABLE audit_log (
    id UUID PRIMARY KEY,
    entity_type VARCHAR(50),
    entity_id UUID,
    action VARCHAR(50),
    old_value JSONB,
    new_value JSONB,
    performed_by UUID,
    performed_at TIMESTAMP DEFAULT NOW(),
    ip_address INET,
    user_agent TEXT
);

-- 索引
CREATE INDEX idx_transactions_account ON transactions(account_id, created_at DESC);
CREATE INDEX idx_audit_entity ON audit_log(entity_type, entity_id);
```

---

## 6. 面试关键点

### SQL vs NoSQL 选择

| 系统 | 建议 | 原因 |
|------|------|------|
| 用户系统 | SQL | 关系查询、事务 |
| 电商订单 | NoSQL | 高写入、分区键查询 |
| 社交 Feed | Wide Column | 时间线范围查询 |
| 消息系统 | 混合 | 主存储 NoSQL + 搜索 ES |
| 支付系统 | SQL | ACID、审计 |

### 分区键设计原则

1. **高基数**: 避免热分区
2. **查询友好**: 支持主要查询模式
3. **均匀分布**: 数据均匀分布

---

## 7. Interview Narrative

> "对于电商订单系统，我选择 DynamoDB，因为订单写入密集（每秒数万订单），需要通过订单 ID 快速查询。主分区键是订单 ID，按用户查询使用 GSI。设计时考虑了热分区风险 - 超级买家可能有数十万订单，通过虚拟节点或salt 解决。对于支付系统，使用 PostgreSQL 保证 ACID 特性，配合审计日志表记录所有变更，确保合规性。"

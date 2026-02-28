# REST API Design Principles

> **设计可扩展、可维护的 API 接口。**

---

## 1. REST 基础

```
REST = Representational State Transfer

核心原则:
- 资源导向（名词，非动词）
- 无状态（每个请求独立）
- 统一接口（标准方法）
- 分层系统（代理、网关）
```

---

## 2. URL 结构

### 2.1 资源命名

```
❌ 错误: /getUsers, /createOrder, /deleteProduct
✅ 正确: GET /users, POST /orders, DELETE /products
```

### 2.2 层级结构

```
# 集合 → 单个资源
GET    /users              # 用户列表
GET    /users/123          # 单个用户

# 子资源
GET    /users/123/orders   # 用户 123 的订单
GET    /users/123/orders/456 # 特定订单

# 动作资源（必要时）
POST   /orders/456/cancel  # 取消订单
POST   /users/123/activate # 激活用户
```

### 2.3 查询参数

```
GET /users?status=active&sort=name&limit=10&offset=20

参数类型:
- filter: status=active
- sort: sort=name
- pagination: limit=10, offset=20
- fields: fields=id,name,email
```

---

## 3. HTTP 方法

| Method | 语义 | 幂等 | Use Case |
|--------|------|------|----------|
| **GET** | 读取 | Yes | 获取资源 |
| **POST** | 创建 | No | 创建资源 |
| **PUT** | 替换 | Yes | 完整更新 |
| **PATCH** | 修改 | No | 部分更新 |
| **DELETE** | 删除 | Yes | 删除资源 |
| **HEAD** | 元数据 | Yes | 检查资源存在 |
| **OPTIONS** | 能力 | Yes | 支持的方法 |

---

## 4. 状态码

### 2xx 成功

| Code | Meaning |
|------|---------|
| 200 | OK - 成功处理请求 |
| 201 | Created - 资源创建成功 |
| 204 | No Content - 成功无返回 |

### 4xx 客户端错误

| Code | Meaning |
|------|---------|
| 400 | Bad Request - 请求格式错误 |
| 401 | Unauthorized - 需要认证 |
| 403 | Forbidden - 无权限 |
| 404 | Not Found - 资源不存在 |
| 409 | Conflict - 资源冲突 |
| 422 | Unprocessable - 验证失败 |
| 429 | Too Many Requests - 限流 |

### 5xx 服务端错误

| Code | Meaning |
|------|---------|
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

---

## 5. 请求/响应格式

### 5.1 JSON 请求

```http
POST /api/v1/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

### 5.2 JSON 响应

```json
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com",
    "created_at": "2024-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "abc-123"
  }
}
```

### 5.3 错误响应

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  }
}
```

---

## 6. 设计最佳实践

### 6.1 一致性

```yaml
# 保持 URL 结构一致
GET    /users          # 列表
POST   /users          # 创建
GET    /users/:id      # 读取
PUT    /users/:id      # 更新
DELETE /users/:id      # 删除

# 不一致的糟糕设计
GET /getUsers
POST createUser
GET getUserById
PUT updateUserInfo
```

### 6.2 版本控制

```yaml
# URL 版本（推荐）
/api/v1/users
/api/v2/users

# Header 版本
Accept: application/vnd.myapp.v2+json
```

### 6.3 分页

```yaml
GET /users?page=2&per_page=20

响应包含:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 1000,
    "total_pages": 50
  }
}
```

### 6.4 字段过滤

```yaml
# 只返回需要的字段
GET /users?fields=id,name,email

{
  "data": [
    {"id": 1, "name": "John", "email": "john@example.com"}
  ]
}
```

---

## 7. Interview Questions

### Q: PUT 和 PATCH 的区别？
**A**:
- PUT: 完整替换资源（提供所有字段）
- PATCH: 部分更新（只提供要修改的字段）

### Q: 何时使用 POST vs PUT？
**A**:
- POST: 创建新资源，服务器生成 ID
- PUT: 创建或替换，客户端指定 ID

### Q: REST vs GraphQL？
**A**:
- REST: 固定端点，缓存友好
- GraphQL: 客户端选择字段，减少请求数
- REST 更简单，GraphQL 更灵活

---

## 8. Interview Narrative

> "我设计的 REST API 遵循以下原则：资源导向的 URL 结构，使用标准 HTTP 方法（GET 读取、POST 创建、PUT 完整更新、PATCH 部分更新）。响应使用 JSON 格式，包含数据、元信息和分页信息。错误响应遵循 RFC 7807 Problem Details 标准，包含错误码、消息和详细信息。API 使用 URL 版本控制（/v1/、/v2/）实现向后兼容。"

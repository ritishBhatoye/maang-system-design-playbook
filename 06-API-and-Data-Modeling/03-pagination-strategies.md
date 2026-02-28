# Pagination Strategies

> **大规模数据列表的高效分页方法。**

---

## 1. 为什么要分页？

```
假设有 100 万条用户记录:

❌ 一次返回全部: 100MB+，超时
✅ 分页返回: 每页 20-100 条，快速响应
```

---

## 2. 常见分页策略

### 2.1 Offset/Page Based

```http
GET /users?page=2&per_page=20
```

```json
{
  "data": [...],
  "pagination": {
    "current_page": 2,
    "per_page": 20,
    "total": 1000000,
    "total_pages": 50000
  }
}
```

**优点**:
- 简单直观
- 用户友好（可以直接跳转页码）

**缺点**:
- OFFSET 大时性能差
- 数据可能不一致（删除记录后偏移）

```sql
-- 性能问题
SELECT * FROM users ORDER BY id LIMIT 1000000, 20;
-- 需要扫描 1000020 行！
```

### 2.2 Cursor/Keyset Based

```http
GET /users?cursor=abc123&limit=20
```

```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "def456",
    "has_more": true
  }
}
```

**原理**: 使用上一页最后一条记录的 ID

```sql
-- 高效查询
SELECT * FROM users 
WHERE id > :last_seen_id 
ORDER BY id 
LIMIT 20;
```

**优点**:
- 性能稳定（不随页码增加而变慢）
- 一致性（基于 ID，不受删除影响）

**缺点**:
- 不能跳转到任意页
- 只支持单向遍历

### 2.3 Seek Method

```http
GET /users?seek_after=1000000&limit=20
```

类似于 Cursor，但是：
- 可以使用任意排序字段
- 更灵活

```sql
-- 按时间排序的分页
SELECT * FROM orders 
WHERE created_at < :last_timestamp
ORDER BY created_at DESC
LIMIT 20;
```

---

## 3. 对比

| 特性 | Offset/Page | Cursor | Seek |
|------|-------------|--------|------|
| 性能 | O(n) | O(1) | O(1) |
| 跳页 | ✅ | ❌ | ❌ |
| 删除兼容 | ❌ | ✅ | ✅ |
| 复杂度 | 简单 | 中等 | 中等 |
| 适用场景 | 小数据 | 大数据 | 大数据 |

---

## 4. 实现示例

### 4.1 Cursor 实现

```python
class UserRepository:
    async def list_users(self, cursor: str = None, limit: int = 20):
        # 解码 cursor（通常是 base64 编码的时间戳+ID）
        if cursor:
            last_seen = self.decode_cursor(cursor)
            query = query.where(User.id > last_seen['id'])
        
        users = await query.order_by(User.id).limit(limit).fetch_all()
        
        # 生成 next cursor
        next_cursor = None
        if len(users) == limit:
            next_cursor = self.encode_cursor({
                'id': users[-1].id
            })
        
        return {
            'data': users,
            'pagination': {
                'next_cursor': next_cursor,
                'has_more': next_cursor is not None
            }
        }
```

### 4.2 Link Header (RFC 5988)

```http
GET /users?limit=20

Link: <https://api.example.com/users?cursor=abc>; rel="next",
      <https://api.example.com/users?cursor=def>; rel="prev"
```

---

## 5. 面试问题

### Q: Offset 分页的性能问题怎么解决？
**A**: 
- 使用 Cursor/Keyset 分页
- 如果必须用 Offset，设置上限（如最多 1000 页）
- 使用延迟关联（先查 ID，再查详情）

### Q: 数据实时性和分页怎么平衡？
**A**:
- Cursor 分页保证一致性
- 定期同步（如每分钟）给用户看总数
- 实时数据用时间戳而非偏移量

### Q: 如何处理"最后一页"删除的问题？
**A**:
- Cursor 分页天然解决
- Offset 时检测到空结果返回上一页

---

## 6. Interview Narrative

> "对于大规模列表，我推荐使用 Cursor 分页。相比 Offset 分页，Cursor 分页性能恒定（O(1)），不受数据删除影响。实现时使用上一页最后一条记录的 ID 作为游标，Base64 编码后通过 URL 传递。这样即使在高并发场景下，百万级数据的分页也能在 10ms 内完成。"

---

## 7. 选择指南

| 数据量 | 推荐方案 |
|--------|----------|
| < 1000 条 | Offset/Page |
| 1000 - 100,000 | Offset + 限制 |
| > 100,000 | Cursor/Seek |
| 实时流 | Cursor |

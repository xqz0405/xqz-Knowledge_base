---
tags:
  - Node.js
  - RESTful
  - API设计
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# RESTful API设计

## What — 它是什么？

### REST 原则

REST（Representational State Transfer，表述性状态转移）是 Roy Fielding 在 2000 年博士论文中提出的架构风格，不是协议或标准，而是一组设计约束。遵循这些约束的 API 被称为 RESTful API。

**REST 的六大核心约束：**

| 约束 | 含义 | 说明 |
|------|------|------|
| 客户端-服务器分离 | 前端与后端职责分离 | 客户端关注用户界面，服务器关注数据存储与处理，独立演化 |
| 无状态 | 每个请求必须包含所有必要信息 | 服务器不保存客户端会话状态，便于水平扩展 |
| 可缓存 | 响应必须明确标识是否可缓存 | 减少不必要的重复请求，提升性能 |
| 统一接口 | 以统一方式访问资源 | REST 的核心特征，包含资源标识、通过表述操作、自描述消息、超媒体驱动 |
| 分层系统 | 客户端不需要知道直连的是终端服务器还是中间代理 | 支持负载均衡、缓存代理、安全层 |
| 按需代码（可选） | 服务器可以临时扩展客户端功能 | 如返回 JavaScript 让客户端执行，大多数 REST API 不使用 |

**统一接口的四个子约束：**
1. **资源标识**：每个资源通过 URI 标识（如 `/users/123`）
2. **通过表述操作资源**：客户端持有资源的表述（JSON/XML），通过它操作资源
3. **自描述消息**：每条消息包含足够的信息来描述如何处理它（Content-Type、Cache-Control 等）
4. **超媒体作为应用状态引擎（HATEOAS）**：响应中包含指向相关资源的链接

### 资源设计

资源是 REST 的核心概念，是系统中可以被命名和寻址的实体。资源设计的好坏直接决定 API 的可用性和可维护性。

**资源命名规范：**

| 规则 | 正确示例 | 错误示例 |
|------|----------|----------|
| 使用名词而非动词 | `GET /users` | `GET /getUsers` |
| 使用复数形式 | `GET /users/123` | `GET /user/123` |
| 使用小写字母和连字符 | `/user-profiles` | `/userProfiles` |
| 嵌套资源表达关系 | `/users/123/orders` | `/orders?userId=123`（两者都可，前者语义更清晰） |
| 避免过深嵌套（≤3层） | `/users/123/orders/456` | `/users/123/orders/456/items/789/details` |
| 查询参数用于过滤 | `/users?role=admin` | `/admin-users` |

**资源粒度设计原则：**
- 粗粒度：一个资源包含更多关联数据，减少请求次数，适合前端驱动的场景
- 细粒度：资源尽可能独立，请求灵活但可能需要多次请求
- 实际设计中需要根据客户端使用模式在两者之间取舍

### HTTP 方法语义

RESTful API 使用标准 HTTP 方法表达对资源的操作意图，每个方法有明确的语义约定：

| HTTP 方法 | 语义 | 幂等性 | 安全性 | 请求体 | 典型用途 |
|-----------|------|--------|--------|--------|----------|
| GET | 获取资源 | 是 | 是 | 无 | 查询单个或列表 |
| POST | 创建资源/触发处理 | 否 | 否 | 有 | 新建资源、执行操作 |
| PUT | 全量替换资源 | 是 | 否 | 有 | 更新整个资源 |
| PATCH | 部分更新资源 | 否* | 否 | 有 | 修改资源的部分字段 |
| DELETE | 删除资源 | 是 | 否 | 可选 | 删除指定资源 |
| OPTIONS | 查询支持的方法 | 是 | 是 | 无 | CORS 预检请求 |
| HEAD | 获取响应头 | 是 | 是 | 无 | 检查资源是否存在 |

> *PATCH 在 RFC 5789 中定义为非幂等，但实际实现中通常设计为幂等

**RESTful 路由映射表：**

| 请求 | 路由 | 含义 |
|------|------|------|
| `GET /users` | 获取用户列表 | 返回分页后的用户集合 |
| `GET /users/123` | 获取单个用户 | 返回指定 ID 的用户详情 |
| `POST /users` | 创建用户 | 请求体包含用户数据，返回 201 |
| `PUT /users/123` | 全量更新用户 | 请求体包含完整用户数据 |
| `PATCH /users/123` | 部分更新用户 | 请求体仅包含需更新的字段 |
| `DELETE /users/123` | 删除用户 | 返回 204 或 200 |
| `GET /users/123/orders` | 获取用户的订单列表 | 嵌套资源查询 |

### 状态码

HTTP 状态码是 RESTful API 的重要组成部分，正确使用状态码让客户端无需解析响应体即可判断请求结果。

**常用状态码分类：**

| 范围 | 类别 | 常用状态码 |
|------|------|-----------|
| 2xx | 成功 | 200 OK、201 Created、204 No Content |
| 3xx | 重定向 | 301 Moved Permanently、304 Not Modified |
| 4xx | 客户端错误 | 400 Bad Request、401 Unauthorized、403 Forbidden、404 Not Found、409 Conflict、422 Unprocessable Entity、429 Too Many Requests |
| 5xx | 服务端错误 | 500 Internal Server Error、502 Bad Gateway、503 Service Unavailable |

**状态码使用细节：**
- `201 Created`：资源创建成功，应在响应头中包含 `Location` 指向新资源的 URI
- `204 No Content`：操作成功但无返回内容（如 DELETE），不应有响应体
- `401 Unauthorized`：未认证（缺少或无效的身份凭证），不是"未授权"
- `403 Forbidden`：已认证但无权限执行此操作
- `409 Conflict`：请求与服务器当前状态冲突（如唯一约束违反）
- `422 Unprocessable Entity`：格式正确但语义错误（如验证失败）
- `429 Too Many Requests`：请求频率超限，应配合 `Retry-After` 头部

### 分页、过滤与排序

对于集合资源的查询，必须支持分页、过滤和排序以控制返回数据量和顺序。

**分页策略：**

| 策略 | 参数 | 优点 | 缺点 |
|------|------|------|------|
| Offset 分页 | `?page=1&limit=20` | 实现简单，可跳转到任意页 | 大偏移量性能差（OFFSET 扫描）、数据变更时结果不稳定 |
| Cursor 分页 | `?cursor=abc123&limit=20` | 性能稳定（索引查找）、数据变更不影响结果 | 不可跳页、游标需编码、实现稍复杂 |
| Keyset 分页 | `?after_id=100&limit=20` | 基于排序键，性能好 | 仅支持向前翻页 |

**过滤与排序：**
- 精确过滤：`?status=active&role=admin`
- 范围过滤：`?created_after=2025-01-01&created_before=2025-12-31`
- 模糊搜索：`?q=keyword` 或 `?name_like=john`
- 排序：`?sort=-created_at,name`（`-` 表示降序）

### 版本管理

API 版本管理确保在 API 演进过程中不破坏现有客户端。

| 方案 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| URI 路径版本 | `/api/v1/users` | 直观、易理解、缓存友好 | URI 变更，不符合 REST 纯粹主义 |
| 查询参数版本 | `/api/users?version=1` | URI 不变 | 易被忽略、缓存策略复杂 |
| 请求头版本 | `Accept: application/vnd.api.v1+json` | URI 干净、RESTful | 不直观、调试不便 |
| 主机名版本 | `v1.api.example.com/users` | 完全隔离 | 域名管理复杂、CORS 配置繁琐 |

**实践推荐：** URI 路径版本（`/api/v1/`）是最广泛采用的方案，尽管不是最"RESTful"的，但最实用。

**版本策略原则：**
- 向后兼容的修改（新增字段、新增端点）不需要升级版本
- 破坏性修改（删除字段、更改语义、修改数据类型）需要升级版本
- 旧版本应设定废弃时间表，给客户端迁移窗口
- 每个版本有独立文档

### HATEOAS

HATEOAS（Hypermedia as the Engine of Application State，超媒体作为应用状态引擎）是 REST 成熟度模型（Richardson Maturity Model）的最高级别（Level 3）。它要求 API 响应中包含指向相关操作的链接，客户端通过跟随链接发现可用操作，而不是硬编码 URL。

```json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "status": "active",
  "_links": {
    "self": { "href": "/api/v1/users/123" },
    "orders": { "href": "/api/v1/users/123/orders" },
    "deactivate": { "href": "/api/v1/users/123/deactivate", "method": "POST" }
  }
}
```

**HATEOAS 的价值：**
- 客户端无需硬编码 URL，降低耦合
- 服务器可以自由调整 URL 结构
- API 自描述，客户端可以动态发现功能
- 但实现复杂，大多数实际项目停留在 Level 2

### OpenAPI 规范

OpenAPI Specification（OAS，原名 Swagger）是描述 RESTful API 的标准规范，使用 JSON/YAML 格式定义 API 的所有细节：端点、参数、请求/响应格式、认证方式等。

**OpenAPI 文档的核心结构：**
- `openapi`：版本号（3.0.x / 3.1.x）
- `info`：API 元信息（标题、描述、版本）
- `servers`：服务器地址列表
- `paths`：端点与操作定义
- `components`：可复用的 Schema、参数、响应等
- `security`：全局安全方案

### 幂等性

幂等性（Idempotency）是指同一请求执行一次与执行多次的效果相同。这是 HTTP 方法语义的重要属性。

| 方法 | 幂等 | 原因 |
|------|------|------|
| GET | 是 | 多次读取同一资源结果相同 |
| PUT | 是 | 全量替换，多次写入同一值结果一致 |
| DELETE | 是 | 删除一次和多次删除同一资源效果相同（资源不存在） |
| POST | 否 | 每次调用创建新资源，多次调用创建多个 |
| PATCH | 视情况 | 部分更新可能非幂等（如 `increment` 操作） |

**幂等性的实际意义：**
- 网络不稳定时，客户端可以安全重试幂等请求
- 中间代理/负载均衡器可以安全重试幂等请求
- 支付等关键操作通过幂等键（Idempotency Key）保证不会重复处理

---

## Why — 为什么学它？

### 适用场景

1. **CRUD 服务**：资源型应用（用户管理、商品管理、内容管理）天然适合 REST，每个资源对应一组 CRUD 端点
2. **开放 API**：公开给第三方开发者的 API（如支付平台、社交媒体），RESTful 风格最容易被理解和集成
3. **微服务接口**：微服务之间的同步通信，REST 是最主流的选择，工具链完善
4. **移动端后端**：iOS/Android 客户端与后端通信，RESTful API 是事实标准
5. **前端 SPA/SSR 后端**：React/Vue/Angular 应用的数据接口，REST 与前端状态管理配合良好
6. **管理后台**：后台管理系统几乎都是 CRUD 操作，REST 极其适配

### 风格对比

| 对比维度 | REST | GraphQL | gRPC |
|----------|------|---------|------|
| 通信协议 | HTTP/1.1, HTTP/2 | HTTP/1.1, HTTP/2 | HTTP/2（底层） |
| 数据格式 | JSON（为主） | JSON | Protocol Buffers（二进制） |
| 数据获取 | 固定结构，可能过度获取/不足获取 | 客户端指定字段，精确获取 | 固定结构（.proto 定义） |
| API 演化 | 版本管理，可能破坏兼容 | 添加字段不影响客户端 | 向后兼容设计，新增字段安全 |
| 学习成本 | 低，HTTP 语义直观 | 中等，需学习查询语言 | 较高，需学习 Protocol Buffers |
| 工具生态 | 极其成熟（Swagger/Postman等） | 成熟（Apollo/Relay等） | 成熟但偏后端 |
| 浏览器支持 | 原生 Fetch/XHR | 原生 Fetch（需查询构造） | 需要 gRPC-Web 代理 |
| 性能 | 一般（文本序列化开销） | 中等（查询解析开销） | 高（二进制序列化+HTTP/2） |
| 适用场景 | 通用 CRUD、开放 API | 复杂查询、多端聚合 | 微服务间高性能通信 |

### 优缺点

**RESTful API 的优点：**
- 基于标准 HTTP 语义，学习成本低，开发者普遍理解
- 工具链极其丰富：Swagger/OpenAPI 文档生成、Postman 测试、代码生成器
- 无状态设计天然支持水平扩展
- 缓存语义清晰，HTTP 缓存机制直接可用
- 透明性好，URL 和 HTTP 方法即文档
- 几乎所有语言和平台都有成熟的 HTTP 客户端

**RESTful API 的缺点：**
- 过度获取（Over-fetching）：GET 返回资源的所有字段，客户端可能只需要部分
- 获取不足（Under-fetching）：关联资源需要多次请求（N+1 问题）
- 没有统一的标准实现，不同团队的设计差异大
- 多资源关联操作（如事务性操作）不好用 REST 语义表达
- 实时推送需要额外方案（WebSocket/SSE）
- HATEOAS 理论美好但实践中极少完整实现

---

## How — 怎么用？

### 代码示例 1：Express 实现 RESTful API

```javascript
const express = require('express');
const { body, param, query, validationResult } = require('express-validator');

const app = express();
app.use(express.json());

// 模拟数据存储
let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com', role: 'admin', createdAt: '2025-01-15' },
  { id: 2, name: 'Bob', email: 'bob@example.com', role: 'user', createdAt: '2025-02-20' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com', role: 'user', createdAt: '2025-03-10' },
];
let nextId = 4;

// ==================== RESTful API ====================

// GET /api/users — 获取用户列表（支持分页、过滤、排序）
app.get('/api/users', [
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  query('role').optional().isIn(['admin', 'user', 'moderator']),
  query('sort').optional().isString(),
  query('q').optional().isString(),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  let result = [...users];

  // 过滤
  if (req.query.role) {
    result = result.filter(u => u.role === req.query.role);
  }
  if (req.query.q) {
    const keyword = req.query.q.toLowerCase();
    result = result.filter(u =>
      u.name.toLowerCase().includes(keyword) ||
      u.email.toLowerCase().includes(keyword)
    );
  }

  // 排序
  if (req.query.sort) {
    const sortField = req.query.sort.startsWith('-')
      ? req.query.sort.slice(1)
      : req.query.sort;
    const order = req.query.sort.startsWith('-') ? -1 : 1;
    result.sort((a, b) => {
      if (a[sortField] < b[sortField]) return -1 * order;
      if (a[sortField] > b[sortField]) return 1 * order;
      return 0;
    });
  }

  // 分页
  const page = req.query.page || 1;
  const limit = req.query.limit || 20;
  const total = result.length;
  const totalPages = Math.ceil(total / limit);
  const offset = (page - 1) * limit;
  const items = result.slice(offset, offset + limit);

  res.json({
    data: items,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1,
    },
  });
});

// GET /api/users/:id — 获取单个用户
app.get('/api/users/:id', [
  param('id').isInt({ min: 1 }).toInt(),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  const user = users.find(u => u.id === req.params.id);
  if (!user) {
    return res.status(404).json({
      error: 'Not Found',
      message: `用户 ID ${req.params.id} 不存在`,
    });
  }

  res.json({
    data: user,
    _links: {
      self: `/api/users/${user.id}`,
      orders: `/api/users/${user.id}/orders`,
    },
  });
});

// POST /api/users — 创建用户
app.post('/api/users', [
  body('name').isString().trim().isLength({ min: 2, max: 50 }),
  body('email').isEmail().normalizeEmail(),
  body('role').optional().isIn(['admin', 'user', 'moderator']).default('user'),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  // 检查邮箱唯一性
  if (users.some(u => u.email === req.body.email)) {
    return res.status(409).json({
      error: 'Conflict',
      message: '该邮箱已被注册',
    });
  }

  const newUser = {
    id: nextId++,
    name: req.body.name,
    email: req.body.email,
    role: req.body.role || 'user',
    createdAt: new Date().toISOString().split('T')[0],
  };
  users.push(newUser);

  res.status(201)
    .header('Location', `/api/users/${newUser.id}`)
    .json({ data: newUser });
});

// PUT /api/users/:id — 全量更新用户
app.put('/api/users/:id', [
  param('id').isInt({ min: 1 }).toInt(),
  body('name').isString().trim().isLength({ min: 2, max: 50 }),
  body('email').isEmail().normalizeEmail(),
  body('role').isIn(['admin', 'user', 'moderator']),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  const index = users.findIndex(u => u.id === req.params.id);
  if (index === -1) {
    return res.status(404).json({
      error: 'Not Found',
      message: `用户 ID ${req.params.id} 不存在`,
    });
  }

  // 邮箱唯一性检查（排除自身）
  if (users.some(u => u.email === req.body.email && u.id !== req.params.id)) {
    return res.status(409).json({
      error: 'Conflict',
      message: '该邮箱已被其他用户使用',
    });
  }

  users[index] = {
    id: req.params.id,
    name: req.body.name,
    email: req.body.email,
    role: req.body.role,
    createdAt: users[index].createdAt,
    updatedAt: new Date().toISOString().split('T')[0],
  };

  res.json({ data: users[index] });
});

// PATCH /api/users/:id — 部分更新用户
app.patch('/api/users/:id', [
  param('id').isInt({ min: 1 }).toInt(),
  body('name').optional().isString().trim().isLength({ min: 2, max: 50 }),
  body('email').optional().isEmail().normalizeEmail(),
  body('role').optional().isIn(['admin', 'user', 'moderator']),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  const user = users.find(u => u.id === req.params.id);
  if (!user) {
    return res.status(404).json({
      error: 'Not Found',
      message: `用户 ID ${req.params.id} 不存在`,
    });
  }

  // 只更新提供的字段
  const allowedFields = ['name', 'email', 'role'];
  allowedFields.forEach(field => {
    if (req.body[field] !== undefined) {
      user[field] = req.body[field];
    }
  });
  user.updatedAt = new Date().toISOString().split('T')[0];

  res.json({ data: user });
});

// DELETE /api/users/:id — 删除用户
app.delete('/api/users/:id', [
  param('id').isInt({ min: 1 }).toInt(),
], (req, res) => {
  const index = users.findIndex(u => u.id === req.params.id);
  if (index === -1) {
    return res.status(404).json({
      error: 'Not Found',
      message: `用户 ID ${req.params.id} 不存在`,
    });
  }

  users.splice(index, 1);
  res.status(204).send();
});

app.listen(3000, () => {
  console.log('RESTful API 服务运行在 http://localhost:3000/');
});
```

### 代码示例 2：分页过滤中间件 + Cursor 分页

```javascript
// middleware/pagination.js
class PaginationMiddleware {
  // Offset 分页中间件
  static offset() {
    return (req, res, next) => {
      req.pagination = {
        page: Math.max(1, parseInt(req.query.page) || 1),
        limit: Math.min(100, Math.max(1, parseInt(req.query.limit) || 20)),
      };
      req.pagination.offset = (req.pagination.page - 1) * req.pagination.limit;
      next();
    };
  }

  // Cursor 分页中间件
  static cursor(options = {}) {
    const { defaultLimit = 20, maxLimit = 100 } = options;
    return (req, res, next) => {
      req.pagination = {
        cursor: req.query.cursor || null,
        limit: Math.min(maxLimit, Math.max(1, parseInt(req.query.limit) || defaultLimit)),
        direction: req.query.direction === 'prev' ? 'prev' : 'next',
      };
      next();
    };
  }

  // 生成 offset 分页响应
  static offsetResponse(data, total, req) {
    const { page, limit } = req.pagination;
    const totalPages = Math.ceil(total / limit);
    return {
      data,
      pagination: {
        page,
        limit,
        total,
        totalPages,
        hasNext: page < totalPages,
        hasPrev: page > 1,
        nextPage: page < totalPages ? page + 1 : null,
        prevPage: page > 1 ? page - 1 : null,
      },
    };
  }

  // 生成 cursor 分页响应
  static cursorResponse(data, encodeCursor, req) {
    const { limit } = req.pagination;
    // 取 limit+1 条判断是否有下一页
    const hasMore = data.length > limit;
    const items = hasMore ? data.slice(0, limit) : data;

    return {
      data: items,
      pagination: {
        limit,
        nextCursor: hasMore ? encodeCursor(items[items.length - 1]) : null,
        prevCursor: items.length > 0 ? encodeCursor(items[0]) : null,
        hasMore,
      },
    };
  }
}

// 过滤中间件
class FilterMiddleware {
  static apply(allowedFilters) {
    return (req, res, next) => {
      req.filters = {};
      for (const [key, config] of Object.entries(allowedFilters)) {
        const value = req.query[key];
        if (value !== undefined) {
          req.filters[key] = config.transform
            ? config.transform(value)
            : value;
        }
      }
      next();
    };
  }
}

// 排序中间件
class SortMiddleware {
  static apply(allowedFields) {
    return (req, res, next) => {
      req.sort = [];
      if (req.query.sort) {
        const fields = req.query.sort.split(',');
        for (const field of fields) {
          const desc = field.startsWith('-');
          const fieldName = desc ? field.slice(1) : field;
          if (allowedFields.includes(fieldName)) {
            req.sort.push({ field: fieldName, order: desc ? 'DESC' : 'ASC' });
          }
        }
      }
      next();
    };
  }
}

// 使用示例
const express = require('express');
const app = express();
app.use(express.json());

// Offset 分页
app.get('/api/v1/articles',
  PaginationMiddleware.offset(),
  FilterMiddleware.apply({
    status: { transform: v => v },
    author_id: { transform: v => parseInt(v) },
    created_after: { transform: v => new Date(v) },
  }),
  SortMiddleware.apply(['created_at', 'title', 'view_count']),
  (req, res) => {
    // 这里应该是数据库查询，用模拟数据演示
    let articles = [
      { id: 1, title: 'REST 入门', status: 'published', author_id: 1, created_at: '2025-01-01', view_count: 100 },
      { id: 2, title: 'REST 进阶', status: 'draft', author_id: 2, created_at: '2025-02-01', view_count: 200 },
      { id: 3, title: 'GraphQL 指南', status: 'published', author_id: 1, created_at: '2025-03-01', view_count: 150 },
    ];

    // 应用过滤
    if (req.filters.status) {
      articles = articles.filter(a => a.status === req.filters.status);
    }
    if (req.filters.author_id) {
      articles = articles.filter(a => a.author_id === req.filters.author_id);
    }

    // 应用排序
    for (const sort of req.sort) {
      articles.sort((a, b) => {
        const va = a[sort.field], vb = b[sort.field];
        const cmp = va < vb ? -1 : va > vb ? 1 : 0;
        return sort.order === 'DESC' ? -cmp : cmp;
      });
    }

    const total = articles.length;
    const { offset, limit } = req.pagination;
    const paged = articles.slice(offset, offset + limit);

    res.json(PaginationMiddleware.offsetResponse(paged, total, req));
  }
);

// Cursor 分页
app.get('/api/v1/events',
  PaginationMiddleware.cursor({ defaultLimit: 10 }),
  (req, res) => {
    // 模拟数据，实际应从数据库查询
    const events = Array.from({ length: 50 }, (_, i) => ({
      id: i + 1,
      name: `Event ${i + 1}`,
      timestamp: new Date(Date.now() - i * 86400000).toISOString(),
    }));

    let filtered = events;
    if (req.pagination.cursor) {
      const cursorId = parseInt(Buffer.from(req.pagination.cursor, 'base64').toString());
      if (req.pagination.direction === 'next') {
        filtered = events.filter(e => e.id < cursorId);
      } else {
        filtered = events.filter(e => e.id > cursorId);
      }
    }

    // 取 limit+1 条
    const data = filtered.slice(0, req.pagination.limit + 1);

    const encodeCursor = (item) => Buffer.from(String(item.id)).toString('base64');
    res.json(PaginationMiddleware.cursorResponse(data, encodeCursor, req));
  }
);

app.listen(3000);
```

### 代码示例 3：OpenAPI 文档生成 + 错误响应规范

```javascript
// openapi-spec.js — OpenAPI 3.0 规范定义
const openApiSpec = {
  openapi: '3.0.3',
  info: {
    title: 'User Management API',
    description: '用户管理 RESTful API 文档',
    version: '1.0.0',
    contact: {
      name: 'API Support',
      email: 'support@example.com',
    },
  },
  servers: [
    { url: 'http://localhost:3000/api/v1', description: '开发环境' },
    { url: 'https://api.example.com/v1', description: '生产环境' },
  ],
  tags: [
    { name: 'Users', description: '用户管理' },
  ],
  paths: {
    '/users': {
      get: {
        tags: ['Users'],
        summary: '获取用户列表',
        operationId: 'listUsers',
        parameters: [
          { name: 'page', in: 'query', schema: { type: 'integer', minimum: 1, default: 1 } },
          { name: 'limit', in: 'query', schema: { type: 'integer', minimum: 1, maximum: 100, default: 20 } },
          { name: 'role', in: 'query', schema: { type: 'string', enum: ['admin', 'user', 'moderator'] } },
          { name: 'sort', in: 'query', schema: { type: 'string' }, description: '排序字段，- 前缀表示降序' },
          { name: 'q', in: 'query', schema: { type: 'string' }, description: '搜索关键词' },
        ],
        responses: {
          200: {
            description: '用户列表',
            content: {
              'application/json': {
                schema: { $ref: '#/components/schemas/UserListResponse' },
              },
            },
          },
        },
      },
      post: {
        tags: ['Users'],
        summary: '创建用户',
        operationId: 'createUser',
        requestBody: {
          required: true,
          content: {
            'application/json': {
              schema: { $ref: '#/components/schemas/CreateUserRequest' },
            },
          },
        },
        responses: {
          201: {
            description: '用户创建成功',
            headers: {
              Location: { schema: { type: 'string' }, description: '新资源的URI' },
            },
            content: {
              'application/json': {
                schema: { $ref: '#/components/schemas/UserResponse' },
              },
            },
          },
          422: { $ref: '#/components/responses/ValidationError' },
          409: { $ref: '#/components/responses/ConflictError' },
        },
      },
    },
    '/users/{id}': {
      get: {
        tags: ['Users'],
        summary: '获取单个用户',
        operationId: 'getUser',
        parameters: [
          { name: 'id', in: 'path', required: true, schema: { type: 'integer', minimum: 1 } },
        ],
        responses: {
          200: {
            description: '用户详情',
            content: {
              'application/json': {
                schema: { $ref: '#/components/schemas/UserDetailResponse' },
              },
            },
          },
          404: { $ref: '#/components/responses/NotFoundError' },
        },
      },
    },
  },
  components: {
    schemas: {
      User: {
        type: 'object',
        required: ['id', 'name', 'email', 'role', 'createdAt'],
        properties: {
          id: { type: 'integer', example: 1 },
          name: { type: 'string', minLength: 2, maxLength: 50, example: 'Alice' },
          email: { type: 'string', format: 'email', example: 'alice@example.com' },
          role: { type: 'string', enum: ['admin', 'user', 'moderator'], example: 'user' },
          createdAt: { type: 'string', format: 'date', example: '2025-01-15' },
          updatedAt: { type: 'string', format: 'date', nullable: true },
        },
      },
      CreateUserRequest: {
        type: 'object',
        required: ['name', 'email'],
        properties: {
          name: { type: 'string', minLength: 2, maxLength: 50 },
          email: { type: 'string', format: 'email' },
          role: { type: 'string', enum: ['admin', 'user', 'moderator'], default: 'user' },
        },
      },
      UserListResponse: {
        type: 'object',
        properties: {
          data: { type: 'array', items: { $ref: '#/components/schemas/User' } },
          pagination: { $ref: '#/components/schemas/Pagination' },
        },
      },
      UserResponse: {
        type: 'object',
        properties: {
          data: { $ref: '#/components/schemas/User' },
        },
      },
      UserDetailResponse: {
        type: 'object',
        properties: {
          data: { $ref: '#/components/schemas/User' },
          _links: { $ref: '#/components/schemas/Links' },
        },
      },
      Pagination: {
        type: 'object',
        properties: {
          page: { type: 'integer' },
          limit: { type: 'integer' },
          total: { type: 'integer' },
          totalPages: { type: 'integer' },
          hasNext: { type: 'boolean' },
          hasPrev: { type: 'boolean' },
        },
      },
      Links: {
        type: 'object',
        properties: {
          self: { type: 'string' },
          orders: { type: 'string' },
        },
      },
      Error: {
        type: 'object',
        required: ['error', 'message'],
        properties: {
          error: { type: 'string', description: '错误类型' },
          message: { type: 'string', description: '错误描述' },
          details: { type: 'array', items: { type: 'object' }, description: '验证错误详情' },
        },
      },
    },
    responses: {
      ValidationError: {
        description: '请求参数验证失败',
        content: {
          'application/json': {
            schema: { $ref: '#/components/schemas/Error' },
            example: {
              error: 'ValidationError',
              message: '请求参数验证失败',
              details: [
                { field: 'email', message: '必须是有效的邮箱地址' },
              ],
            },
          },
        },
      },
      NotFoundError: {
        description: '资源不存在',
        content: {
          'application/json': {
            schema: { $ref: '#/components/schemas/Error' },
            example: { error: 'NotFound', message: '用户 ID 999 不存在' },
          },
        },
      },
      ConflictError: {
        description: '资源冲突',
        content: {
          'application/json': {
            schema: { $ref: '#/components/schemas/Error' },
            example: { error: 'Conflict', message: '该邮箱已被注册' },
          },
        },
      },
    },
  },
};

// 错误响应规范工具
class ApiError extends Error {
  constructor(status, error, message, details = null) {
    super(message);
    this.status = status;
    this.error = error;
    this.details = details;
  }
}

// 统一错误处理中间件
function errorHandler(err, req, res, next) {
  if (err instanceof ApiError) {
    const response = {
      error: err.error,
      message: err.message,
    };
    if (err.details) {
      response.details = err.details;
    }
    res.status(err.status).json(response);
    return;
  }

  // express-validator 验证错误
  if (err.array && typeof err.array === 'function') {
    res.status(422).json({
      error: 'ValidationError',
      message: '请求参数验证失败',
      details: err.array().map(e => ({
        field: e.path,
        value: e.value,
        message: e.msg,
      })),
    });
    return;
  }

  // 未知错误
  console.error('未处理的错误:', err);
  res.status(500).json({
    error: 'InternalServerError',
    message: '服务器内部错误',
  });
}

// 使用
const express = require('express');
const app = express();
app.use(express.json());

// 提供 OpenAPI 文档
app.get('/api-docs.json', (req, res) => {
  res.json(openApiSpec);
});

// 示例路由
app.get('/api/v1/users/:id', (req, res, next) => {
  const id = parseInt(req.params.id);
  if (isNaN(id) || id < 1) {
    throw new ApiError(400, 'BadRequest', '无效的用户 ID');
  }
  // 模拟查找
  if (id > 100) {
    throw new ApiError(404, 'NotFound', `用户 ID ${id} 不存在`);
  }
  res.json({ data: { id, name: 'User ' + id, email: `user${id}@example.com` } });
});

app.use(errorHandler);
app.listen(3000);

module.exports = { openApiSpec, ApiError, errorHandler };
```

### 代码示例 4：API 版本管理 + 限流中间件

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();
app.use(express.json());

// ==================== 版本管理 ====================

// V1 路由
const v1Router = express.Router();

v1Router.get('/users', (req, res) => {
  // V1 返回简单格式
  res.json({
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ],
  });
});

v1Router.get('/users/:id', (req, res) => {
  res.json({
    user: { id: req.params.id, name: 'User ' + req.params.id },
  });
});

// V2 路由
const v2Router = express.Router();

v2Router.get('/users', (req, res) => {
  // V2 返回增强格式（新增字段、分页、HATEOAS 链接）
  res.json({
    data: [
      { id: 1, name: 'Alice', email: 'alice@example.com', role: 'admin' },
      { id: 2, name: 'Bob', email: 'bob@example.com', role: 'user' },
    ],
    pagination: { page: 1, limit: 20, total: 2 },
    _links: { self: '/api/v2/users' },
  });
});

v2Router.get('/users/:id', (req, res) => {
  res.json({
    data: { id: req.params.id, name: 'User ' + req.params.id, email: `user${req.params.id}@example.com` },
    _links: {
      self: `/api/v2/users/${req.params.id}`,
      orders: `/api/v2/users/${req.params.id}/orders`,
    },
  });
});

// 挂载版本路由
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// 版本废弃警告中间件
app.use('/api/v1', (req, res, next) => {
  res.setHeader('Deprecation', 'true');
  res.setHeader('Sunset', 'Sat, 01 Jan 2027 00:00:00 GMT');
  res.setHeader('Link', '</api/v2/users>; rel="successor-version"');
  next();
});

// ==================== 限流中间件 ====================

// 全局限流
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 1000,                 // 每个 IP 最多 1000 次请求
  standardHeaders: true,     // 返回 RateLimit-* 头部
  legacyHeaders: false,      // 禁用 X-RateLimit-* 头部
  message: {
    error: 'TooManyRequests',
    message: '请求频率超限，请稍后再试',
  },
});

// 登录端点严格限流
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10, // 每个 IP 15 分钟内最多 10 次登录
  message: {
    error: 'TooManyRequests',
    message: '登录尝试次数过多，请 15 分钟后再试',
  },
});

app.use(globalLimiter);
app.post('/api/v2/auth/login', authLimiter, (req, res) => {
  res.json({ token: 'jwt-token-here' });
});

// ==================== 请求日志中间件 ====================

app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`);
  });
  next();
});

app.listen(3000, () => {
  console.log('API 服务运行在 http://localhost:3000/');
  console.log('V1 文档: http://localhost:3000/api/v1/users');
  console.log('V2 文档: http://localhost:3000/api/v2/users');
});
```

### 踩坑指南

| 踩坑场景 | 现象 | 原因 | 解决方案 |
|----------|------|------|----------|
| 动词化 URL | `/createUser`、`/getUserById` | 违反 REST 语义，URL 应表示资源而非操作 | 改用名词 + HTTP 方法：`POST /users`、`GET /users/:id` |
| PUT 用作部分更新 | 只发送修改的字段，未发送的字段被清空 | PUT 是全量替换，缺失字段会被设为 null | 部分更新使用 PATCH，PUT 需要发送完整资源 |
| 忽略 204 状态码 | DELETE 成功后返回 `{ success: true }` | 204 No Content 表示成功且无响应体，是 DELETE 的最佳实践 | 删除成功返回 204，无响应体 |
| 在 GET 中使用请求体 | GET 请求携带 JSON 请求体做过滤 | HTTP 规范不禁止但实际中代理/缓存可能忽略 GET 请求体 | 过滤参数放 URL 查询字符串，复杂查询用 POST /search |
| 分页不返回总数 | 客户端无法知道总页数 | 计算 COUNT 有性能开销，开发者跳过了 | 至少在 offset 分页中返回 total；cursor 分页返回 hasMore |
| 版本号放在子域名 | `v1.api.example.com` | 每个版本需要独立域名和 SSL 证书 | 使用 URI 路径 `/api/v1/`，更简单且易于管理 |
| 4xx 错误无详细信息 | 只返回 `{ error: 'Bad Request' }` | 客户端无法定位具体问题 | 返回结构化错误：`{ error, message, details: [{field, message}] }` |
| 幂等键缺失 | POST 请求网络超时重试导致重复创建 | POST 非幂等，重试会创建多个资源 | 使用幂等键（Idempotency-Key 请求头），服务端记录已处理的请求 |
| 状态码滥用 | 所有错误都返回 500 或 200 + error 字段 | 不遵循 HTTP 语义，客户端难以处理 | 按语义选择状态码：400 参数错误、401 未认证、403 无权限、404 不存在、409 冲突、422 验证失败 |
| 缺少 CORS 头部 | 浏览器端跨域请求被阻止 | 没有配置 Access-Control-Allow-Origin 等响应头 | 使用 cors 中间件，根据环境配置允许的域名和方法 |

### 最佳实践

1. **资源命名使用复数名词**：`/users` 而非 `/user`，`/orders` 而非 `/order`。这是业界最广泛接受的约定，保持一致性比争论单复数更重要

2. **始终返回结构化错误响应**：错误响应应包含 `error`（错误类型）、`message`（人类可读描述）、可选的 `details`（验证错误字段列表），让客户端能程序化地处理错误

3. **为 POST 创建操作返回 201 + Location 头**：创建资源成功时，响应状态码应为 201，响应头中包含 `Location` 指向新创建资源的 URI，响应体包含新资源的完整数据

4. **使用查询参数做过滤，不创建新端点**：`GET /users?role=admin&status=active` 而非 `GET /admin-users` 或 `GET /active-admins`，查询参数是通用的过滤机制

5. **实现分页时返回分页元数据**：包括 `total`、`page`、`limit`、`hasNext` 等，让客户端可以构建分页 UI。对于大数据量场景，优先使用 cursor 分页

6. **版本化 API 从第一天开始**：即使只有 v1，也使用 `/api/v1/` 前缀。这为未来的破坏性变更预留了空间，避免版本升级时需要修改所有端点路径

7. **合理使用 HTTP 缓存头**：对读取频繁且变化不频繁的资源，设置 `Cache-Control`、`ETag`、`Last-Modified` 等头部，利用 HTTP 缓存减少服务器负载

8. **文档先行（Document-First）**：先写 OpenAPI 规范再写代码，确保 API 设计经过评审，也方便前端并行开发。使用 Swagger UI 或 Redoc 提供可交互的文档

---

## 面试题

### 1. REST 的核心约束有哪些？

REST 的六大核心约束：① 客户端-服务器分离（关注点分离）；② 无状态（每个请求包含所有必要信息，服务器不保存会话状态）；③ 可缓存（响应标识是否可缓存）；④ 统一接口（资源标识、表述操作、自描述消息、HATEOAS）；⑤ 分层系统（客户端不需要知道直连的是终端还是代理）；⑥ 按需代码（可选，服务器可以临时扩展客户端功能）。其中统一接口是 REST 区别于其他架构风格的核心。Richardson 成熟度模型将 REST 实现分为 3 级：Level 0（HTTP 隧道）、Level 1（资源）、Level 2（HTTP 方法+状态码）、Level 3（HATEOAS）。

### 2. PUT 和 PATCH 的区别是什么？

PUT 是全量替换，请求体必须包含资源的完整数据，缺失的字段会被设为 null 或默认值。PATCH 是部分更新，请求体仅包含需要修改的字段。关键区别：① 幂等性——PUT 是幂等的（多次执行结果相同），PATCH 理论上非幂等（如 `{"count": +1}` 每次执行都不同），但实践中通常设计为幂等；② 请求体——PUT 必须发送完整资源，PATCH 只发送变更部分；③ 安全性——PUT 可能意外清空未发送的字段，PATCH 更安全；④ HTTP 规范——PUT 定义在 RFC 7231，PATCH 定义在 RFC 5789，部分老旧客户端/代理不支持 PATCH 方法。

### 3. 什么是幂等性？如何保证？

幂等性是指同一请求执行一次和多次的效果相同。GET/PUT/DELETE 天然幂等，POST 天然非幂等。保证幂等性的方法：① 使用 PUT 替代 POST 做更新操作；② 幂等键（Idempotency Key）——客户端生成唯一键放在 `Idempotency-Key` 请求头中，服务端记录已处理的键，重复请求返回缓存结果；③ 数据库唯一约束——如创建订单时用 `user_id + product_id + idempotency_key` 做联合唯一索引；④ 乐观锁——使用版本号字段，更新时检查版本是否一致。

### 4. offset 分页和 cursor 分页各有什么优缺点？

Offset 分页（`?page=2&limit=20`）：优点是实现简单直观，支持跳转到任意页码；缺点是大偏移量时数据库需要扫描并跳过前面所有行（`OFFSET 1000000` 扫描 100 万行），性能随页码增大线性下降；在数据插入/删除时结果不稳定（翻页时可能漏看或重复看到数据）。Cursor 分页（`?cursor=abc123&limit=20`）：优点是性能恒定（基于索引直接定位），不受数据量影响；数据变更时结果稳定；缺点是不能跳转到任意页码，只能上一页/下一页；cursor 需要编码（通常 Base64 编码排序键）；实现稍复杂。推荐：面向用户的前端分页用 offset（需要跳页），API 内部分页/大数据量导出用 cursor。

### 5. API 版本管理有哪些方案？

四种主流方案：① URI 路径版本（`/api/v1/users`）——最常用，直观且缓存友好，缺点是不够"RESTful"；② 查询参数版本（`/api/users?version=1`）——URI 不变但易被忽略，缓存策略复杂；③ 请求头版本（`Accept: application/vnd.api.v1+json`）——RESTful 纯粹主义者的选择，URI 干净但不直观；④ 主机名版本（`v1.api.example.com`）——完全隔离但域名管理复杂。实践推荐 URI 路径版本，配合 `Deprecation` 和 `Sunset` 响应头标记废弃版本。

### 6. HATEOAS 的意义是什么？

HATEOAS（Hypermedia as the Engine of Application State）要求 API 响应中包含指向相关资源和操作的链接。意义：① 解耦——客户端不需要硬编码 URL，服务器可以自由调整 URL 结构；② 自发现——客户端通过跟随链接发现可用操作，类似浏览网页；③ 状态驱动——响应中的链接反映资源当前状态下的可用操作（如订单在"待支付"状态才有"支付"链接）。但 HATEOAS 实现复杂，增加了响应体大小，客户端需要额外的链接解析逻辑，且缺乏标准的链接格式。大多数生产 API 停留在 Richardson 成熟度模型的 Level 2。

### 7. HTTP 状态码的选择原则是什么？

原则：① 2xx 表示成功——200 通用成功、201 创建成功、204 成功无内容；② 3xx 表示重定向——301 永久移动、304 未修改；③ 4xx 表示客户端错误——400 请求格式错误、401 未认证、403 已认证但无权限、404 资源不存在、409 资源冲突、422 语义错误（验证失败）、429 请求频率超限；④ 5xx 表示服务端错误——500 内部错误、502 网关错误、503 服务不可用。常见误区：① 用 200 + error 字段代替 4xx，违反 HTTP 语义；② 混淆 401 和 403（401 是未认证，403 是无权限）；③ 所有错误都返回 500。正确使用状态码让客户端无需解析响应体即可判断结果。

### 8. OpenAPI/Swagger 的作用是什么？

OpenAPI Specification（原名 Swagger）是描述 RESTful API 的标准规范，核心作用：① API 设计先行——在编码前定义 API 契约，方便团队评审和前后端并行开发；② 自动生成文档——配合 Swagger UI/Redoc 生成可交互的 API 文档，支持在线测试；③ 代码生成——根据 OpenAPI 规范自动生成客户端 SDK、服务端桩代码、TypeScript 类型定义；④ 契约测试——验证服务端实现是否符合规范定义；⑤ API 网关集成——许多 API 网关（如 Kong、AWS API Gateway）可直接导入 OpenAPI 规范配置路由和验证。OpenAPI 3.1 是当前最新版本，支持 JSON Schema 完整兼容。

---

## 相关链接

- [[Express]] — 实现 RESTful API 最主流的 Node.js 框架
- [[Fastify]] — 高性能框架，内置 JSON Schema 验证，天然适配 REST API
- [[GraphQL与Apollo]] — REST 的替代方案，解决过度获取/获取不足问题
- [[认证与授权]] — RESTful API 安全的核心：JWT、OAuth2、API Key 等认证方案
- [RESTful API 设计指南](https://restfulapi.net/) — RESTful API 设计最佳实践参考

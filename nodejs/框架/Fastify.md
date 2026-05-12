---
tags:
  - Node.js
  - Fastify
  - Web框架
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Fastify

## 概述

Fastify 是一个高度专注于**性能**和**开发体验**的 Node.js Web 框架，由 Matteo Collina（Node.js TSC 成员）和 Tomas Della Vedova 创建并维护。它的核心设计理念是"在不牺牲开发体验的前提下，提供最快的 HTTP 框架"。Fastify 通过 JSON Schema 验证、高效的路由匹配、日志序列化、插件化架构等机制，在社区基准测试中长期保持领先。截至 2026 年，Fastify GitHub Star 超过 33K，npm 周下载量超过 300 万，已成为 Node.js 生态中最重要的 Web 框架之一。

---

## What — 核心概念

### JSON Schema 验证

Fastify 最大的特色之一是将 JSON Schema 作为一等公民集成到路由定义中。通过为请求的 `body`、`querystring`、`params`、`headers` 以及响应 `response` 定义 Schema，Fastify 可以在路由处理器执行前后自动完成验证和序列化，同时利用 Schema 信息将 JSON 序列化速度提升 2-3 倍。

**核心能力：**
- 请求体验证：自动校验请求体是否符合 Schema，不符则返回 400 错误
- 响应序列化：利用 Schema 将响应对象序列化为 JSON，比 `JSON.stringify` 快约 2-3 倍
- 类型生成：可从 Schema 自动生成 TypeScript 类型
- Swagger 集成：Schema 直接用于生成 API 文档

```javascript
const schema = {
  body: {
    type: 'object',
    required: ['name', 'email'],
    properties: {
      name: { type: 'string', minLength: 1, maxLength: 100 },
      email: { type: 'string', format: 'email' },
      age: { type: 'integer', minimum: 0, maximum: 150 },
    },
    additionalProperties: false,
  },
  querystring: {
    type: 'object',
    properties: {
      page: { type: 'integer', default: 1 },
      limit: { type: 'integer', default: 20 },
    },
  },
  params: {
    type: 'object',
    properties: {
      id: { type: 'string', pattern: '^[0-9a-f]{24}$' },
    },
  },
  headers: {
    type: 'object',
    required: ['authorization'],
    properties: {
      authorization: { type: 'string' },
    },
  },
  response: {
    200: {
      type: 'object',
      properties: {
        id: { type: 'string' },
        name: { type: 'string' },
        email: { type: 'string' },
        createdAt: { type: 'string', format: 'date-time' },
      },
    },
  },
};
```

### 插件系统

Fastify 的插件系统基于 [`avvio`](https://github.com/mcollina/avvio) 库实现，具有以下特点：

- **异步加载**：插件按注册顺序异步初始化，所有插件加载完毕后才启动服务
- **封装作用域**：每个插件拥有独立的命名空间，插件内注册的路由、装饰器、钩子默认不会泄漏到父作用域
- **依赖声明**：插件可以声明对其他插件的依赖，确保加载顺序
- **错误传播**：插件初始化失败会阻止服务器启动

```javascript
// 注册插件
fastify.register(require('@fastify/cors'), {
  origin: ['https://example.com'],
  methods: ['GET', 'POST'],
});

// 封装插件
fastify.register(async function (fastify, opts) {
  // 这里的路由、装饰器只在当前作用域内可见
  fastify.decorate('db', createDbConnection(opts.dbUrl));
  fastify.get('/internal', async () => ({ status: 'ok' }));
});

// 使用 fastify-plugin 打破封装
const fp = require('fastify-plugin');
module.exports = fp(async function (fastify, opts) {
  // 装饰器会泄漏到父作用域
  fastify.decorate('sharedUtil', () => 'available everywhere');
});
```

### 钩子 (Hook)

钩子是 Fastify 生命周期中的拦截点，允许在请求处理的不同阶段插入自定义逻辑。Fastify 提供了丰富的钩子类型，覆盖请求的完整生命周期。

| 钩子类型 | 触发时机 | 用途 | 可修改 |
|---------|---------|------|--------|
| `onRequest` | 请求到达，路由匹配前 | 认证、限流、日志 | request/reply |
| `preParsing` | 请求体解析前 | 修改原始请求体 | request/reply |
| `preValidation` | Schema 验证前 | 自定义验证、数据预处理 | request/reply |
| `preHandler` | 路由处理器执行前 | 权限检查、数据增强 | request/reply |
| `preSerialization` | 响应序列化前 | 修改响应数据 | payload |
| `onError` | 错误发生时 | 错误日志、错误转换 | error |
| `onSend` | 响应发送前 | 响应头修改、压缩 | payload |
| `onResponse` | 响应发送完成 | 性能监控、审计日志 | request/reply |
| `onTimeout` | 请求超时 | 超时日志、降级处理 | request/reply |

### 日志序列化

Fastify 默认使用 [`pino`](https://github.com/pinojs/pino) 作为日志库，这是 Node.js 生态中最快的 JSON 日志库。Fastify 的日志序列化利用 JSON Schema 预编译序列化函数，避免了运行时反射和类型检查，使日志输出几乎零开销。

```javascript
const fastify = require('fastify')({
  logger: {
    level: 'info',
    serializers: {
      req(req) {
        return {
          method: req.method,
          url: req.url,
          headers: req.headers,
          remoteAddress: req.ip,
        };
      },
      res(res) {
        return {
          statusCode: res.statusCode,
        };
      },
    },
  },
});

// 每个请求自动记录
// {"level":30,"time":1715500800000,"req":{"method":"GET","url":"/api/users"},"res":{"statusCode":200},"responseTime":1.5,"msg":"request completed"}
```

### 生命周期

Fastify 的请求生命周期是其性能优势的核心来源。每个请求经过严格的阶段管线，每个阶段都有明确的职责和可预测的行为。

**完整生命周期：**

```
请求到达
  │
  ├─→ onRequest 钩子
  │
  ├─→ 路由匹配（Radix Tree）
  │
  ├─→ preParsing 钩子
  │
  ├─→ 请求体解析（Content-Type 协商）
  │
  ├─→ preValidation 钩子
  │
  ├─→ Schema 验证（body/query/params/headers）
  │     │
  │     └─→ 验证失败 → 400 Bad Request
  │
  ├─→ preHandler 钩子
  │
  ├─→ 路由处理器 (Handler)
  │     │
  │     ├─→ 正常返回 → preSerialization 钩子
  │     └─→ 抛出错误 → onError 钩子
  │
  ├─→ preSerialization 钩子（修改 payload）
  │
  ├─→ 响应序列化（fast-json-stringify）
  │
  ├─→ onSend 钩子
  │
  ├─→ 发送响应
  │
  ├─→ onResponse 钩子
  │
  └─→ 请求结束
```

### 封装上下文

Fastify 的每个插件注册都会创建一个新的作用域（Scope），这称为封装上下文。封装上下文决定了装饰器、钩子、路由的可见范围。

```javascript
// 父作用域
fastify.decorate('version', '1.0.0');

// 子作用域 A
fastify.register(async (scopeA, opts) => {
  scopeA.decorate('feature', 'alpha');
  console.log(scopeA.version);  // '1.0.0' — 继承父作用域
  console.log(scopeA.feature);  // 'alpha' — 本作用域
  // console.log(scopeB.feature);  // ❌ 不可访问 scopeB 的装饰器
});

// 子作用域 B
fastify.register(async (scopeB, opts) => {
  console.log(scopeB.version);  // '1.0.0' — 继承父作用域
  // console.log(scopeB.feature);  // ❌ 不可访问 scopeA 的装饰器
  scopeB.decorate('feature', 'beta');
  console.log(scopeB.feature);  // 'beta'
});
```

作用域继承规则：
- 子作用域可以访问父作用域的装饰器
- 兄弟作用域之间互不可见
- 子作用域的装饰器不会泄漏到父作用域（除非使用 `fastify-plugin`）

---

## 核心架构

### 设计理念

Fastify 的设计理念可以概括为两个核心原则：

**1. 性能优先 (Performance First)**

Fastify 在每一个架构决策中都优先考虑性能：
- 使用 Radix Tree 实现路由匹配，时间复杂度 O(k)（k 为路径长度），与路由数量无关
- 使用 `fast-json-stringify` 预编译 JSON 序列化函数，比 `JSON.stringify` 快 2-3 倍
- 使用 `pino` 日志库，采用子进程写入避免阻塞主线程
- 最小化中间件栈深度，避免 Express 式的洋葱模型开销
- 每个请求的上下文对象复用（请求结束后回收），减少 GC 压力

**2. 开发体验 (Developer Experience)**

性能不以牺牲开发体验为代价：
- 插件系统提供清晰的代码组织方式
- JSON Schema 验证自动拦截无效请求，减少 Handler 中的样板代码
- TypeScript 支持完善，类型推断精确
- 自动生成 Swagger 文档
- 清晰的错误消息和调试信息

### 核心模块

| 模块 | 职责 | 关键实现 |
|------|------|---------|
| **路由引擎** | URL 到 Handler 的映射 | `find-my-way` — Radix Tree 路由 |
| **插件系统** | 插件注册、加载、作用域管理 | `avvio` — 异步插件加载器 |
| **生命周期** | 请求阶段的定义与执行 | 钩子队列 + 阶段管线 |
| **Schema 系统** | 验证与序列化 | `ajv`（验证）+ `fast-json-stringify`（序列化） |
| **日志系统** | 结构化日志输出 | `pino` — 高性能 JSON 日志 |
| **错误系统** | 错误创建、传播、处理 | `@fastify/error` + 错误 Schema |

**路由引擎 — find-my-way：**

`find-my-way` 使用 Radix Tree（压缩前缀树）存储路由，核心优势：
- 路由查找时间与路由总数无关，仅与路径长度相关
- 支持参数化路由（`/users/:id`）、通配符路由（`/files/*`）、约束路由（Host、Method）
- 路由注册时构建树，查找时只需遍历树

**Schema 系统 — ajv + fast-json-stringify：**

- `ajv` 是最快的 JSON Schema 验证器，它将 Schema 编译为验证函数，后续验证直接调用编译后的函数
- `fast-json-stringify` 将 JSON Schema 编译为 JSON 序列化函数，生成的代码直接操作已知属性，跳过 `JSON.stringify` 的类型检查和递归遍历

### 数据流

```
客户端请求
    │
    ▼
┌──────────────────────────────────────────────────┐
│              Node.js HTTP Server                  │
│              (fastify.listen)                     │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              onRequest 钩子                       │
│  [认证、限流、CORS、请求ID]                        │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              路由匹配 (find-my-way)               │
│  Radix Tree → 匹配到 Route Object                │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              preParsing 钩子                      │
│  [请求体预处理、解压]                               │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              请求体解析                            │
│  Content-Type 协商 → JSON/Text/Multipart 解析     │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              preValidation 钩子                   │
│  [自定义验证、数据转换]                             │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              Schema 验证 (ajv)                    │
│  body/query/params/headers → 编译后验证函数        │
│  验证失败 → 400 Error                             │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              preHandler 钩子                      │
│  [权限检查、数据增强、事务管理]                      │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│              路由处理器 (Handler)                   │
│  业务逻辑 → 返回 payload                          │
└──────────────────────────────────────────────────┘
    │
    ├─→ 成功 ──→ preSerialization 钩子
    │                 │
    │                 ▼
    │            响应序列化 (fast-json-stringify)
    │                 │
    │                 ▼
    │            onSend 钩子 [压缩、响应头]
    │                 │
    │                 ▼
    │            发送 HTTP 响应
    │
    └─→ 错误 ──→ onError 钩子
                      │
                      ▼
                 错误 Schema 序列化
                      │
                      ▼
                 onSend 钩子
                      │
                      ▼
                 发送错误响应

    │
    ▼
┌──────────────────────────────────────────────────┐
│              onResponse 钩子                      │
│  [性能监控、审计日志、指标收集]                       │
└──────────────────────────────────────────────────┘
```

---

## 插件生态

### 官方插件

Fastify 官方维护了 40+ 插件，覆盖了 Web 开发的绝大多数需求。以下是核心官方插件：

| 插件 | 功能 | 典型用法 |
|------|------|---------|
| `@fastify/cors` | 跨域资源共享 | 配置允许的 Origin、Methods、Headers |
| `@fastify/swagger` | OpenAPI/Swagger 文档生成 | 从 JSON Schema 自动生成 API 文档 |
| `@fastify/swagger-ui` | Swagger UI 界面 | 提供交互式 API 文档页面 |
| `@fastify/jwt` | JWT 认证 | 签发和验证 JSON Web Token |
| `@fastify/static` | 静态文件服务 | 托管前端构建产物、图片等 |
| `@fastify/multipart` | 文件上传 | 处理 multipart/form-data 请求 |
| `@fastify/cookie` | Cookie 管理 | 解析和设置 Cookie |
| `@fastify/session` | 会话管理 | 服务端 Session 存储 |
| `@fastify/rate-limit` | 请求限流 | 防止 API 滥用 |
| `@fastify/helmet` | 安全头 | 设置安全相关 HTTP 头 |
| `@fastify/compress` | 响应压缩 | gzip/brotli 压缩 |
| `@fastify/formbody` | 表单解析 | 解析 URL 编码的请求体 |
| `@fastify/websocket` | WebSocket | 在 Fastify 中使用 WebSocket |
| `@fastify/redis` | Redis 客户端 | 封装 ioredis 连接管理 |
| `@fastify/mongodb` | MongoDB 客户端 | 封装 MongoDB 连接管理 |
| `@fastify/sequelize` | Sequelize ORM | 集成 Sequelize 数据库 ORM |
| `@fastify/typeorm` | TypeORM | 集成 TypeORM 数据库 ORM |
| `@fastify/env` | 环境变量 | 带验证的环境变量管理 |
| `@fastify/sensible` | HTTP 错误 | 提供 4xx/5xx 错误的便捷方法 |
| `@fastify/circuit-breaker` | 熔断器 | 实现熔断模式防止级联故障 |

### 社区热门插件

| 插件 | 功能 | 说明 |
|------|------|------|
| `fastify-autoload` | 自动加载 | 自动扫描目录注册路由和插件 |
| `fastify-graceful-shutdown` | 优雅关闭 | 处理 SIGTERM/SIGINT 信号 |
| `mercurius` | GraphQL | Fastify 的 GraphQL 实现 |
| `@fastify/auth` | 多策略认证 | 组合多种认证方式 |
| `fastify-plugin` | 插件辅助 | 打破封装作用域 |
| `@fastify/view` | 模板引擎 | 支持多种模板引擎 |

---

## Why — 适用场景与对比

### 适用场景

**高性能 API 服务：**
Fastify 的 JSON Schema 序列化和高效路由匹配使其成为 JSON API 的最佳选择。在基准测试中，Fastify 的吞吐量通常是 Express 的 2-3 倍。

**微服务架构：**
Fastify 的插件化架构天然适合微服务。每个服务可以封装为独立插件，共享基础设施（日志、认证、监控），同时保持业务逻辑的独立性。轻量级的核心（约 20KB gzip）使 Fastify 非常适合容器化部署。

**GraphQL 网关：**
配合 `mercurius` 插件，Fastify 可以作为高性能 GraphQL 网关。Schema Stitching 和 Federation 都有良好支持。

**Serverless 函数：**
Fastify 的快速启动时间（约 50ms 冷启动）使其适合 Serverless 场景。可以使用 `@fastify/aws-lambda` 适配 AWS Lambda。

**不适合场景：**
- 服务端渲染 (SSR) — 更推荐 Next.js / Nuxt.js
- 实时双向通信为主 — 更推荐 Socket.IO / ws
- 简单脚本/工具 — 过于重量级

### Express / Koa / Fastify 四维对比表

| 维度 | Express | Koa | Fastify |
|------|---------|-----|---------|
| **性能** | 基准线（1x） | 略快于 Express（1.2x） | 2-3x 于 Express |
| **中间件模型** | 洋葱模型（callback） | 洋葱模型（async/await） | 钩子模型（生命周期管线） |
| **验证** | 需手动集成（joi/zod） | 需手动集成 | 内置 JSON Schema 验证 |
| **序列化** | `JSON.stringify` | `JSON.stringify` | `fast-json-stringify`（2-3x 更快） |
| **日志** | 需手动集成（morgan/winston） | 需手动集成 | 内置 pino（零开销） |
| **TypeScript** | 社区类型定义 | 社区类型定义 | 原生 TS 支持，类型推断优秀 |
| **插件体系** | 中间件（无作用域） | 中间件（无作用域） | 插件（封装作用域 + 依赖管理） |
| **文档生成** | 需手动集成 swagger-jsdoc | 需手动集成 | 内置 @fastify/swagger |
| **异步错误** | 需 express-async-errors | 原生支持 | 原生支持 |
| **生态规模** | 最大（50K+ 中间件） | 中等 | 快速增长（官方 40+ 插件） |
| **学习曲线** | 低 | 低 | 中等（Schema + 插件概念） |
| **维护状态** | 维护模式 | 维护模式 | 活跃开发 |

### 优缺点

**Fastify 优点：**
- 性能卓越，业界领先
- JSON Schema 验证和序列化一体化
- 插件封装作用域防止命名冲突
- 开箱即用的高性能日志（pino）
- 优秀的 TypeScript 支持
- 自动生成 API 文档
- 清晰的生命周期模型
- 活跃的社区和核心团队

**Fastify 缺点：**
- 学习曲线比 Express 陡峭（需要理解 Schema、插件作用域、生命周期）
- JSON Schema 的表达能力不如 Zod/Joi 灵活
- 社区生态不如 Express 丰富（但官方插件覆盖率高）
- 某些场景下 Schema 定义冗长
- 从 Express 迁移需要重写中间件逻辑

---

## How — 代码示例与最佳实践

### 示例1：基础 API + Schema 验证

```javascript
// server.js
const Fastify = require('fastify');
const sensible = require('@fastify/sensible');
const cors = require('@fastify/cors');

const app = Fastify({ logger: true });

// 注册基础插件
app.register(sensible);
app.register(cors, { origin: true });

// 内存数据存储
const users = new Map();
let nextId = 1;

// 创建用户 — 带 Schema 验证
app.post('/users', {
  schema: {
    body: {
      type: 'object',
      required: ['name', 'email'],
      properties: {
        name: { type: 'string', minLength: 2, maxLength: 50 },
        email: { type: 'string', format: 'email' },
        age: { type: 'integer', minimum: 0, maximum: 150 },
      },
      additionalProperties: false,
    },
    response: {
      201: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          email: { type: 'string' },
          age: { type: ['integer', 'null'] },
          createdAt: { type: 'string' },
        },
      },
    },
  },
}, async (request, reply) => {
  const { name, email, age } = request.body;

  // 业务验证：邮箱唯一性
  for (const user of users.values()) {
    if (user.email === email) {
      return reply.conflict(`Email ${email} already exists`);
    }
  }

  const user = {
    id: nextId++,
    name,
    email,
    age: age ?? null,
    createdAt: new Date().toISOString(),
  };
  users.set(user.id, user);

  reply.code(201);
  return user;
});

// 获取用户列表
app.get('/users', {
  schema: {
    querystring: {
      type: 'object',
      properties: {
        page: { type: 'integer', minimum: 1, default: 1 },
        limit: { type: 'integer', minimum: 1, maximum: 100, default: 20 },
      },
    },
    response: {
      200: {
        type: 'object',
        properties: {
          data: {
            type: 'array',
            items: {
              type: 'object',
              properties: {
                id: { type: 'integer' },
                name: { type: 'string' },
                email: { type: 'string' },
              },
            },
          },
          pagination: {
            type: 'object',
            properties: {
              page: { type: 'integer' },
              limit: { type: 'integer' },
              total: { type: 'integer' },
            },
          },
        },
      },
    },
  },
}, async (request) => {
  const { page, limit } = request.query;
  const allUsers = Array.from(users.values());
  const start = (page - 1) * limit;
  const data = allUsers.slice(start, start + limit);

  return {
    data,
    pagination: { page, limit, total: allUsers.length },
  };
});

// 获取单个用户
app.get('/users/:id', {
  schema: {
    params: {
      type: 'object',
      properties: {
        id: { type: 'integer' },
      },
    },
    response: {
      200: {
        type: 'object',
        properties: {
          id: { type: 'integer' },
          name: { type: 'string' },
          email: { type: 'string' },
          age: { type: ['integer', 'null'] },
          createdAt: { type: 'string' },
        },
      },
    },
  },
}, async (request, reply) => {
  const { id } = request.params;
  const user = users.get(id);
  if (!user) {
    return reply.notFound(`User ${id} not found`);
  }
  return user;
});

// 启动服务
app.listen({ port: 3000, host: '0.0.0.0' })
  .then((address) => console.log(`Server listening at ${address}`))
  .catch((err) => {
    app.log.error(err);
    process.exit(1);
  });
```

### 示例2：插件开发

```javascript
// ===== plugins/db.js — 数据库插件 =====
const fp = require('fastify-plugin');

async function dbPlugin(fastify, options) {
  const { url, dbName, poolSize = 10 } = options;

  // 模拟数据库连接（实际使用 @fastify/mongodb 或 @fastify/sequelize）
  const connection = {
    url,
    dbName,
    poolSize,
    connected: true,
    query: async (sql, params) => {
      fastify.log.debug({ sql, params }, 'Executing query');
      // 模拟查询
      return { rows: [], rowCount: 0 };
    },
    close: async () => {
      connection.connected = false;
      fastify.log.info('Database connection closed');
    },
  };

  // 装饰 fastify 实例（fp 使得装饰泄漏到父作用域）
  fastify.decorate('db', connection);

  // 使用 onClose 钩子清理资源
  fastify.addHook('onClose', async (instance) => {
    await instance.db.close();
  });
}

// 使用 fastify-plugin 导出，打破封装
module.exports = fp(dbPlugin, {
  name: 'app-db',
  fastify: '5.x',
});
```

```javascript
// ===== plugins/auth.js — 认证插件 =====
const fp = require('fastify-plugin');
const jwt = require('@fastify/jwt');

async function authPlugin(fastify, options) {
  // 注册 JWT 插件（在子作用域中）
  fastify.register(jwt, {
    secret: options.secret,
    sign: { expiresIn: '24h' },
  });

  // 添加认证装饰器
  fastify.decorate('authenticate', async (request, reply) => {
    try {
      await request.jwtVerify();
    } catch (err) {
      reply.send(err);
    }
  });

  // 添加 preHandler 钩子（可选的全局认证）
  if (options.globalAuth) {
    fastify.addHook('onRequest', async (request, reply) => {
      // 排除不需要认证的路由
      if (options.publicRoutes?.includes(request.url)) return;
      await request.jwtVerify();
    });
  }
}

module.exports = fp(authPlugin, {
  name: 'app-auth',
  dependencies: ['app-db'],  // 声明依赖，确保 db 插件先加载
  fastify: '5.x',
});
```

```javascript
// ===== app.js — 组合插件 =====
const Fastify = require('fastify');
const dbPlugin = require('./plugins/db');
const authPlugin = require('./plugins/auth');

const app = Fastify({ logger: true });

app.register(dbPlugin, {
  url: 'mongodb://localhost:27017',
  dbName: 'myapp',
  poolSize: 20,
});

app.register(authPlugin, {
  secret: process.env.JWT_SECRET,
  publicRoutes: ['/health', '/auth/login'],
});

// 受保护的路由
app.get('/profile', {
  preHandler: [app.authenticate],
}, async (request) => {
  return { user: request.user };
});

app.listen({ port: 3000 });
```

### 示例3：钩子使用

```javascript
const Fastify = require('fastify');
const crypto = require('crypto');
const app = Fastify({ logger: true });

// ===== onRequest — 请求日志与追踪 =====
app.addHook('onRequest', async (request, reply) => {
  request.startTime = process.hrtime.bigint();
  request.requestId = crypto.randomUUID();
  request.log.info({ requestId: request.requestId }, 'Request received');
});

// ===== preValidation — 数据预处理 =====
app.addHook('preValidation', async (request, reply) => {
  // 将查询参数中的字符串数字转为实际数字
  if (request.query) {
    for (const [key, value] of Object.entries(request.query)) {
      if (/^\d+$/.test(value)) {
        request.query[key] = Number(value);
      }
    }
  }
});

// ===== preHandler — 认证与权限检查 =====
app.addHook('preHandler', async (request, reply) => {
  // 从请求头提取用户信息
  const authHeader = request.headers.authorization;
  if (authHeader?.startsWith('Bearer ')) {
    try {
      const decoded = await request.jwtVerify(authHeader.slice(7));
      request.user = decoded;
    } catch {
      // 不在全局钩子中拒绝，让路由自行处理
      request.user = null;
    }
  }
});

// ===== preSerialization — 响应数据转换 =====
app.addHook('preSerialization', async (request, reply, payload) => {
  // 为所有成功响应添加元数据
  if (payload && typeof payload === 'object' && !payload._meta) {
    payload._meta = {
      requestId: request.requestId,
      timestamp: new Date().toISOString(),
    };
  }
  return payload;
});

// ===== onError — 错误日志与转换 =====
app.addHook('onError', async (request, reply, error) => {
  request.log.error({
    error: {
      message: error.message,
      stack: error.stack,
      statusCode: error.statusCode,
    },
    requestId: request.requestId,
  }, 'Request error');

  // 将内部错误信息隐藏，返回统一格式
  if (error.statusCode >= 500) {
    error.message = 'Internal Server Error';
  }
});

// ===== onResponse — 性能监控 =====
app.addHook('onResponse', async (request, reply) => {
  const duration = Number(process.hrtime.bigint() - request.startTime) / 1e6;
  request.log.info({
    requestId: request.requestId,
    duration: `${duration.toFixed(2)}ms`,
    statusCode: reply.statusCode,
  }, 'Request completed');

  // 可发送到监控系统
  // metrics.histogram('request_duration', duration, { route: request.url });
});
```

### 示例4：错误处理

```javascript
const Fastify = require('fastify');
const createError = require('@fastify/error');

const app = Fastify({ logger: true });

// ===== 自定义错误类型 =====
const UserNotFound = createError('USER_NOT_FOUND', 'User %s not found', 404);
const InvalidCredentials = createError('INVALID_CREDENTIALS', 'Invalid email or password', 401);
const DuplicateEmail = createError('DUPLICATE_EMAIL', 'Email %s is already registered', 409);
const InsufficientPermissions = createError('INSUFFICIENT_PERMISSIONS', 'Insufficient permissions: requires %s', 403);

// ===== 全局错误处理器 =====
app.setErrorHandler((error, request, reply) => {
  // Fastify 验证错误
  if (error.validation) {
    return reply.status(400).send({
      statusCode: 400,
      error: 'Bad Request',
      message: 'Validation error',
      details: error.validation.map((v) => ({
        field: v.instancePath || v.params?.missingProperty,
        message: v.message,
        keyword: v.keyword,
      })),
    });
  }

  // 自定义错误（带 statusCode）
  if (error.statusCode) {
    return reply.status(error.statusCode).send({
      statusCode: error.statusCode,
      error: error.name,
      message: error.message,
    });
  }

  // 未知错误
  request.log.error(error, 'Unhandled error');
  reply.status(500).send({
    statusCode: 500,
    error: 'Internal Server Error',
    message: 'An unexpected error occurred',
  });
});

// ===== 404 处理器 =====
app.setNotFoundHandler((request, reply) => {
  reply.status(404).send({
    statusCode: 404,
    error: 'Not Found',
    message: `Route ${request.method}:${request.url} not found`,
  });
});

// ===== 路由中使用自定义错误 =====
app.get('/users/:id', async (request, reply) => {
  const user = await findUserById(request.params.id);
  if (!user) {
    throw new UserNotFound(request.params.id);  // "User abc123 not found"
  }
  return user;
});

app.post('/auth/login', async (request, reply) => {
  const user = await findUserByEmail(request.body.email);
  if (!user || !verifyPassword(user, request.body.password)) {
    throw new InvalidCredentials();
  }
  const token = app.jwt.sign({ id: user.id, email: user.email });
  return { token };
});
```

### 踩坑表

| 坑点 | 现象 | 原因 | 解决方案 |
|------|------|------|---------|
| 插件装饰器不可用 | `Cannot read property 'xxx' of undefined` | 插件封装作用域，装饰器未泄漏到使用处 | 使用 `fastify-plugin` 包裹插件，或在使用处同一作用域注册 |
| Schema 验证不生效 | 请求体包含额外字段但未报错 | 默认 `ajv` 配置不允许 `additionalProperties` 需显式声明 | 在 Schema 中设置 `"additionalProperties": false` |
| async 错误被吞掉 | Handler 中 throw 的错误未被捕获 | 忘记 await Promise 或使用 `fastify.register` 未正确处理 | 确保所有 async 函数都 await，使用 `setErrorHandler` 统一捕获 |
| 响应与 Schema 不匹配 | 返回了 Schema 中未定义的字段但被截断 | `fast-json-stringify` 按 Schema 序列化，多余字段被忽略 | 更新 Schema 包含所有需要返回的字段，或移除 response Schema |
| 插件加载顺序错误 | 依赖的装饰器/服务尚未注册 | 插件注册是异步的，但未声明依赖 | 使用 `fp` 的 `dependencies` 选项声明依赖关系 |
| preHandler 中 reply.send 后仍执行 Handler | 发送了错误响应但 Handler 仍被调用 | `reply.send()` 不会终止钩子链 | 在 `reply.send()` 后 `return reply`，Fastify 检测到 reply 已发送则跳过后续 |
| JWT verify 抛出未处理错误 | 401 错误返回 500 | `request.jwtVerify()` 抛出的错误未被 setErrorHandler 处理 | 在 preHandler 中 try-catch，或确保 setErrorHandler 覆盖认证错误 |
| onRequest 中修改 request.body 无效 | 在 onRequest 中设置的属性在 Handler 中丢失 | onRequest 阶段 body 尚未解析，修改的是不同阶段的数据 | 在 preHandler 阶段修改，此时 body 已解析 |

### 最佳实践

1. **始终为路由定义 JSON Schema**：不仅用于验证，更重要的是 `fast-json-stringify` 依赖 response Schema 进行高效序列化。缺失 response Schema 时回退到 `JSON.stringify`，性能下降显著。

2. **使用 fastify-plugin 管理作用域**：需要全局共享的装饰器和服务用 `fp` 包裹；仅限特定路由组使用的功能保持封装。不要默认全部用 `fp`，封装是好的。

3. **利用 autoload 自动加载**：将路由和插件按目录组织，使用 `@fastify/autoload` 自动扫描注册，避免手动维护注册列表。

   ```javascript
   // 推荐的项目结构
   // ├── app.js
   // ├── plugins/
   // │   ├── db.js
   // │   ├── auth.js
   // │   └── redis.js
   // ├── routes/
   // │   ├── users.js
   // │   ├── products.js
   // │   └── orders.js

   app.register(require('@fastify/autoload'), {
     dir: path.join(__dirname, 'plugins'),
   });
   app.register(require('@fastify/autoload'), {
     dir: path.join(__dirname, 'routes'),
     routeParams: true,
   });
   ```

4. **声明插件依赖**：使用 `fp` 的 `dependencies` 选项明确声明插件间依赖，让 `avvio` 自动处理加载顺序，避免时序问题。

5. **统一错误处理**：使用 `@fastify/error` 创建自定义错误类，配合 `setErrorHandler` 统一错误响应格式。不要在 Handler 中直接 `reply.send(error)`。

6. **合理使用封装作用域**：为不同的功能模块创建独立作用域（如管理后台 API 和用户端 API），避免装饰器冲突和路由命名冲突。

7. **启用请求超时**：设置合理的 `connectionTimeout` 和 `requestTimeout`，防止慢客户端和长时间运行的请求耗尽服务器资源。

   ```javascript
   const app = Fastify({
     connectionTimeout: 30000,   // 30s 连接超时
     requestTimeout: 15000,      // 15s 请求超时
     keepAliveTimeout: 72000,    // 72s Keep-Alive 超时
   });
   ```

8. **生产环境关闭 logger 或调整级别**：pino 虽快，但在极高并发下仍有开销。生产环境建议设置 `logger: { level: 'warn' }` 或使用子进程写入。

---

## 面试题

### 1. Fastify 的性能优势来源于哪些方面？

**答**：Fastify 的性能优势来自四个核心优化：

1. **JSON 序列化优化**：使用 `fast-json-stringify`，根据 JSON Schema 在启动时预编译序列化函数。生成的代码直接访问已知属性名，跳过了 `JSON.stringify` 的类型检查、hasOwnProperty 检查和递归遍历，序列化速度提升 2-3 倍。

2. **高效路由匹配**：使用 `find-my-way` 库基于 Radix Tree 实现路由查找。Radix Tree 是压缩前缀树，路由查找时间复杂度 O(k)（k 为 URL 路径长度），与注册的路由总数无关。相比之下，Express 的路由匹配是线性遍历正则数组。

3. **高性能日志**：默认使用 pino，它采用极简的 JSON 格式和子进程写入模式，日志开销几乎为零。pino 还支持 redaction（敏感信息脱敏）和自定义序列化器。

4. **最小化开销**：没有 Express 式的中间件洋葱模型遍历开销；请求上下文对象复用减少 GC 压力；异步插件加载避免阻塞事件循环。

### 2. 请描述 Fastify 的 JSON Schema 验证流程。

**答**：Fastify 的 JSON Schema 验证流程如下：

1. **Schema 注册**：在路由定义时通过 `schema` 选项注册 `body`、`querystring`、`params`、`headers` 的验证 Schema 和 `response` 的序列化 Schema。

2. **编译阶段**：Fastify 在服务启动时（`ready` 事件前），使用 `ajv` 将验证 Schema 编译为验证函数。编译后的函数是闭包，直接引用 Schema 中的规则，跳过了解析和构建的开销。

3. **验证阶段**：请求到达后，在 `preValidation` 钩子之后、`preHandler` 钩子之前，Fastify 调用编译好的验证函数对请求数据进行验证。

4. **错误处理**：如果验证失败，Fastify 创建一个包含 `validation` 数组的错误对象（包含失败的 keyword、message、instancePath 等），交给 `setErrorHandler` 处理。默认返回 400 状态码和验证详情。

5. **序列化阶段**：Handler 返回数据后，使用 `fast-json-stringify` 根据 `response` Schema 预编译的序列化函数将数据序列化为 JSON 字符串。如果未定义 response Schema，则回退到 `JSON.stringify`。

关键点：验证和序列化函数的**编译发生在启动时**，请求处理时只调用编译后的函数，这是性能优势的来源。

### 3. Fastify 插件的作用域与封装机制是怎样的？

**答**：Fastify 的插件作用域基于 `avvio` 库实现，核心概念是**封装 (Encapsulation)**：

- 每次调用 `fastify.register(plugin)` 都会创建一个新的作用域（子 Fastify 实例）
- 子作用域可以访问父作用域的装饰器和插件
- 子作用域内注册的装饰器、钩子、路由**默认不会泄漏到父作用域**
- 兄弟作用域之间互不可见

这种封装机制的好处：
- 防止命名冲突：不同插件可以为同名装饰器提供不同实现
- 隔离副作用：插件内的钩子只影响该作用域内的路由
- 安全的代码组织：确保内部实现不会意外影响外部

打破封装的方法：
- 使用 `fastify-plugin (fp)` 包裹插件函数，装饰器会泄漏到父作用域
- `fp` 的第二个参数可以指定 `fastify` 版本范围和 `dependencies` 列表

### 4. Fastify 生命周期中的 Hook 类型有哪些？各自的执行时机是什么？

**答**：Fastify 提供了 9 种请求生命周期 Hook 和 3 种应用生命周期 Hook：

**请求生命周期 Hook（按执行顺序）：**
1. `onRequest` — 请求到达，路由匹配前。适合：认证、限流、请求 ID
2. `preParsing` — 请求体解析前。适合：修改原始请求体（如解压）
3. `preValidation` — Schema 验证前。适合：数据预处理、自定义验证
4. `preHandler` — 路由处理器前。适合：权限检查、事务管理
5. `preSerialization` — 响应序列化前。适合：响应数据转换、脱敏
6. `onError` — 错误发生时。适合：错误日志、错误格式转换（不可修改响应，只能修改 error）
7. `onSend` — 响应发送前。适合：响应头修改、压缩、Cookie 设置
8. `onResponse` — 响应发送完成后。适合：性能监控、审计日志
9. `onTimeout` — 请求超时。适合：超时日志、降级通知

**应用生命周期 Hook：**
1. `onReady` — 服务器就绪（所有插件加载完毕）
2. `onClose` — 服务器关闭
3. `onRegister` — 新插件注册时（仅在注册的作用域内触发）

### 5. fastify-plugin 的作用是什么？

**答**：`fastify-plugin`（简称 `fp`）是 Fastify 插件开发的核心工具，有两个主要作用：

**1. 打破封装作用域**：默认情况下，插件内注册的装饰器只在插件作用域内可用。使用 `fp` 包裹后，装饰器会"泄漏"到父作用域，对所有路由可见。这是共享基础设施（数据库连接、认证服务等）的标准方式。

```javascript
// 不使用 fp — 装饰器仅在本作用域
async function myPlugin(fastify) {
  fastify.decorate('db', connection);  // 只在本插件的路由中可用
}

// 使用 fp — 装饰器泄漏到父作用域
const fp = require('fastify-plugin');
module.exports = fp(async function myPlugin(fastify) {
  fastify.decorate('db', connection);  // 全局可用
});
```

**2. 声明元信息**：`fp` 的第二个参数可以指定：
- `name`：插件名称（用于错误消息和调试）
- `fastify`：兼容的 Fastify 版本范围（如 `'>=3.0.0'`）
- `dependencies`：前置依赖插件列表（确保加载顺序）

```javascript
module.exports = fp(myPlugin, {
  name: 'my-app-db',
  fastify: '5.x',
  dependencies: ['@fastify/redis'],  // 确保 redis 先加载
});
```

### 6. Fastify 的错误处理机制是怎样的？

**答**：Fastify 的错误处理机制包含多个层次：

**1. 自动错误捕获**：Fastify 自动捕获 async Handler 和 Hook 中抛出的错误（以及 rejected Promise），无需像 Express 那样手动 `try-catch` 或使用 `express-async-errors`。

**2. 验证错误**：JSON Schema 验证失败时，Fastify 自动创建 `400 Bad Request` 响应，错误对象包含 `validation` 数组（详细的验证失败信息）。

**3. setErrorHandler**：全局错误处理器，接收 `(error, request, reply)` 三个参数。所有未在 Handler 中自行处理的错误都会进入此处理器。

```javascript
app.setErrorHandler((error, request, reply) => {
  if (error.validation) { /* 验证错误 */ }
  else if (error.statusCode >= 500) { /* 服务器错误 */ }
  else { /* 客户端错误 */ }
  reply.status(error.statusCode || 500).send({ error: error.message });
});
```

**4. setNotFoundHandler**：专门处理 404 路由未找到的情况。

**5. @fastify/error**：创建自定义错误类，自动设置 `statusCode` 和 `name`。

**6. onError Hook**：在错误发生后、发送响应前触发，可以修改错误对象但不能直接发送响应。适合日志记录和错误转换。

**7. fastify.sensible**：提供便捷的错误创建方法，如 `reply.notFound()`、`reply.unauthorized()`、`reply.conflict()` 等。

### 7. Fastify 的日志序列化原理是什么？

**答**：Fastify 日志序列化的高性能原理：

1. **pino 架构**：pino 采用主线程写入策略——日志格式化在主线程完成，但 I/O 写入可以委托给子线程（`pino.worker`），避免磁盘 I/O 阻塞事件循环。

2. **预编译序列化器**：pino 在初始化时为每个日志级别和字段组合编译序列化函数，避免运行时反射。自定义 `serializers` 也只需定义一次。

3. **极简 JSON 格式**：pino 输出的是最简 JSON，不包含多余的格式化（无缩进、无换行），直接拼接字符串而非递归遍历。

4. **子 logger 零开销**：`request.log` 是 pino 的 child logger，通过 `logger.child({ reqId })` 创建。child logger 继承父 logger 的所有配置和序列化器，但只添加一条绑定字段，几乎无性能损耗。

5. **Fastify 集成**：Fastify 在每个请求的 `onRequest` 阶段创建 child logger，在 `onResponse` 阶段自动记录请求完成日志（包含响应时间）。这些都在 pino 的低开销架构内完成。

6. **fast-json-stringify 辅助**：响应体的序列化也使用了 Schema 预编译技术，与日志序列化形成双重性能保障。

### 8. 从 Express 迁移到 Fastify 需要注意哪些要点？

**答**：从 Express 迁移到 Fastify 的核心要点：

**1. 中间件模型转换**：
- Express 的 `app.use(middleware)` 需要转换为 Fastify 的 Hook 或插件
- Express 洋葱模型（next()）→ Fastify 生命周期管线（不需要手动 next()）
- Express 的 `(req, res, next)` → Fastify 的 `(request, reply)` 或 `(request, reply, done)`

**2. Request/Response API 差异**：
- `req.body` → `request.body`（语义相同）
- `req.params` → `request.params`
- `req.query` → `request.query`
- `res.json(data)` → `return data` 或 `reply.send(data)`
- `res.status(code).json(data)` → `reply.code(code).send(data)`
- `res.setHeader()` → `reply.header()` 或 `reply.headers()`

**3. 错误处理转换**：
- 不再需要 `express-async-errors`，Fastify 自动捕获 async 错误
- `next(err)` → 直接 `throw err`
- 全局错误处理从 `(err, req, res, next)` → `setErrorHandler((error, request, reply) => {})`

**4. 添加 JSON Schema**：
- 这是迁移中工作量最大的部分
- 为每个路由的 request/response 定义 Schema
- 建议先迁移路由结构，再逐步添加 Schema

**5. 插件系统**：
- Express 中间件全局共享 → Fastify 插件有作用域
- 共享中间件用 `fastify-plugin` 包裹
- 考虑使用 `@fastify/express` 作为过渡方案，兼容 Express 中间件

**6. 其他差异**：
- `app.listen()` 是异步的，返回 Promise
- `this` 在 Handler 中不可用（不要用箭头函数 + this）
- `__dirname`/`__filename` 在 ESM 项目中需要替换

---

## 相关链接

- [[Express]] — 对比 Fastify 与 Express 的架构差异和迁移路径
- [[Koa中间件机制]] — 对比洋葱模型与 Fastify 生命周期管线的差异
- [[RESTful API设计]] — Fastify 路由设计与 JSON Schema 验证的最佳实践

**外部链接：**
- [Fastify 官方文档](https://fastify.dev/) — Fastify 框架完整文档与教程
- [Fastify Ecosystem](https://fastify.dev/ecosystem/) — 官方与社区插件目录
- [fast-json-stringify](https://github.com/fastify/fast-json-stringify) — Schema 驱动的高性能 JSON 序列化库

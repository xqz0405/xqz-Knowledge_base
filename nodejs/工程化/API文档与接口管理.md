---
tags:
  - Node.js
  - API文档
  - Swagger
  - OpenAPI
date: 2026-05-12
status: 已完成
difficulty: 入门
---

# API 文档与接口管理

## What — 是什么

> API 文档是接口的契约和说明书，OpenAPI（Swagger）是事实标准。在 Node.js 中通过代码注解或声明式定义自动生成交互式文档，减少文档与代码的脱节。

**核心概念：**

- **OpenAPI 3.0**：REST API 描述规范，YAML/JSON 定义路径/参数/请求体/响应
- **Swagger UI**：交互式 API 文档界面，可直接在页面测试接口
- **swagger-jsdoc**：从 JSDoc 注释生成 OpenAPI 规范
- **tsoa**：TypeScript 装饰器自动生成 OpenAPI，类型即文档
- **接口版本**：v1/v2 并行维护，废弃接口标记 `deprecated`

**关键特性：**

- OpenAPI 规范可自动生成客户端 SDK
- Swagger UI 提供"试一试"功能，前后端协作更高效
- 代码注解方式（JSDoc/装饰器）让文档与代码同步更新

## Why — 为什么

**适用场景：**

- 前后端协作：文档即契约，减少沟通成本
- 第三方集成：提供标准化的 API 文档
- API 测试：Swagger UI 直接测试接口
- 接口管理：版本控制和废弃追踪

## How — 怎么用

### 代码示例

```javascript
// Express + swagger-jsdoc + swagger-ui-express
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const specs = swaggerJsdoc({
    definition: {
        openapi: '3.0.0',
        info: { title: 'My API', version: '1.0.0' },
        servers: [{ url: 'http://localhost:3000' }]
    },
    apis: ['./routes/*.js']
});

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

// 路由文件中的 JSDoc 注释
/**
 * @openapi
 * /users:
 *   get:
 *     summary: 获取用户列表
 *     parameters:
 *       - name: page
 *         in: query
 *         schema: { type: integer, default: 1 }
 *     responses:
 *       200:
 *         description: 成功
 */
app.get('/users', listUsers);

// Fastify + @fastify/swagger
fastify.register(require('@fastify/swagger'), {
    openapi: { info: { title: 'My API', version: '1.0.0' } }
});
fastify.register(require('@fastify/swagger-ui'), { routePrefix: '/docs' });
// Schema 自动生成文档
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 文档与代码不同步 | 手动维护 YAML | 用代码注解/Schema 自动生成 |
| JSDoc 注释繁琐 | 每个路由都要写 | 用 tsoa 装饰器或 Fastify Schema |
| 版本管理困难 | 多版本 API 文档混乱 | 按版本分 Swagger 规范文件 |

### 最佳实践

- 用代码注解/Schema 自动生成文档，不手动写 YAML
- Fastify 项目用 `@fastify/swagger`，Express 用 `swagger-jsdoc`
- 接口变更时同步更新文档
- 废弃接口标记 `deprecated: true`

## 面试题

**Q1: OpenAPI 规范的核心结构？**
> 三大核心：① `paths`——定义所有 API 路径和方法（GET/POST/PUT/DELETE），每个路径定义参数、请求体、响应；② `components`——可复用的 Schema（数据模型）、Security Scheme（认证方式）、Parameters（公共参数）；③ `info`/`servers`——API 元信息和服务器地址。好处：机器可读、自动生成文档/SDK/测试、前后端共享契约。

**Q2: 如何保证 API 文档与代码一致？**
> 三种方式：① 代码注解——JSDoc/装饰器写在路由代码旁，改代码即改文档（`swagger-jsdoc`/`tsoa`）；② Schema 驱动——Fastify 的 Schema 同时用于验证和文档生成，天然一致；③ CI 检查——对比 OpenAPI 规范与实际路由，发现不一致则构建失败。最佳实践是 Schema 驱动（Fastify），或代码注解（Express）+ CI 校验。

---

**相关链接：**
- [[RESTful API设计]]
- [[Fastify]]
- [[Express]]
- OpenAPI 规范：https://swagger.io/specification/

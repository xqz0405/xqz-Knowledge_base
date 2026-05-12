---
tags:
  - Node.js
  - Web框架
  - Express
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Express

## What — 是什么

> Express 是 Node.js 最经典的 Web 框架，提供极简的路由、中间件和 HTTP 工具集，是 Node.js Web 开发的事实标准。

**核心概念：**

- **Application**：`express()` 创建的应用实例，管理路由和设置
- **Middleware**：`(req, res, next) => {}` 函数，按注册顺序执行
- **Router**：模块化路由，可挂载到应用的特定路径前缀
- **Request/Response**：对原生 `http.IncomingMessage` 和 `http.ServerResponse` 的增强封装

**核心架构：**

- 设计理念：极简核心，一切皆中间件
- 核心模块：路由系统、中间件栈、请求/响应增强
- 数据流：Request → Middleware Stack → Handler → Response

**插件生态：**

- 官方插件：无（Express 核心极简）
- 社区热门插件：cors、helmet（安全）、morgan（日志）、cookie-parser、express-validator

## Why — 为什么

**适用场景：**

- RESTful API 开发
- SSR Web 应用（配合模板引擎）
- 快速原型和小型服务

**对比同类框架：**

| 维度 | Express | Koa | Fastify |
|------|---------|-----|---------|
| 性能 | 中 | 中 | 极高 |
| 生态 | 最丰富 | 丰富 | 中等 |
| 学习曲线 | 最低 | 低 | 中 |
| 灵活性 | 高 | 极高 | 中 |

**优缺点：**

- ✅ 优点：
  - 极简，5 分钟搭建 API
  - 社区庞大，中间件生态最丰富
  - 文档完善，教程遍地
- ❌ 缺点：
  - 基于 callback，错误处理不够优雅
  - 无内置 TypeScript 支持（需 @types/express）
  - 性能不如新一代框架（Fastify）
  - 缺少内置验证、序列化等

## How — 怎么用

### 快速上手

```javascript
const express = require('express');
const app = express();

app.use(express.json()); // 解析 JSON body

app.get('/users/:id', (req, res) => {
    res.json({ id: req.params.id, name: 'Alice' });
});

app.post('/users', (req, res) => {
    const { name, email } = req.body;
    res.status(201).json({ id: 1, name, email });
});

app.listen(3000);
```

### 代码示例

**模块化路由：**

```javascript
// routes/users.js
const router = express.Router();

router.get('/', listUsers);
router.post('/', createUser);
router.get('/:id', getUser);

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/api/users', usersRouter);
```

**自定义错误处理中间件：**

```javascript
// 必须放最后，4 个参数
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(err.status || 500).json({
        error: err.message || 'Internal Server Error',
    });
});

// 在路由中触发错误
app.get('/broken', (req, res, next) => {
    const err = new Error('Something went wrong');
    err.status = 400;
    next(err); // 跳过后续中间件，进入错误处理
});
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 异步错误丢失 | `async` 函数中 throw 不会被 next 捕获 | 用 `next(err)` 或引入 `express-async-errors` |
| body 解析为 undefined | 未注册 `express.json()` / `express.urlencoded()` | 在路由前 `app.use(express.json())` |
| CORS 报错 | 浏览器跨域限制 | 使用 `cors` 中间件 |

### 最佳实践

- 使用 `helmet` 中间件增强安全性
- 路由按模块拆分到独立文件
- 使用 `express-async-errors` 或手动包装 async 中间件
- 生产环境设置 `NODE_ENV=production`（启用视图缓存等优化）

## 面试题

**Q1: Express 中间件的执行模型是什么？next() 的作用是什么？**
> Express 中间件是线性模型，按注册顺序依次执行。每个中间件接收 (req, res, next) 三个参数，调用 next() 将控制权传递给下一个中间件。如果不调用 next()，请求将挂起（没有中间件再处理）。错误处理中间件接收 4 个参数 (err, req, res, next)，调用 next(err) 会跳过后续普通中间件，直接进入错误处理中间件。

**Q2: Express 的路由机制是如何实现的？Router 和 app 的关系是什么？**
> Express 路由基于路径和 HTTP 方法匹配，内部维护一个路由栈（Layer 数组）。Router 是一个迷你应用实例（mini-app），拥有独立的中间件和路由栈，通过 app.use('/prefix', router) 挂载到主应用。Router 实现了模块化路由，将不同业务模块的路由拆分到独立文件中，便于维护。

**Q3: Express 中 async 函数抛出错误为什么无法被错误处理中间件捕获？如何解决？**
> Express 的 next(err) 机制只能捕获同步错误。async 函数中 throw 或 await 抛出的异常是 Promise rejection，不会被 Express 捕获，导致返回 500 且无自定义响应。解决方案：在 async 中间件内用 try/catch 包裹并手动调用 next(err)；或引入 express-async-errors 包自动包装；或写一个高阶函数包装 async 中间件。

**Q4: Express 和 Koa 的核心区别是什么？**
> 三点核心区别：(1) 中间件模型：Express 是线性模型（顺序执行），Koa 是洋葱模型（支持前后置逻辑）；(2) 异步处理：Express 基于 callback，Koa 原生 async/await；(3) 请求响应封装：Express 使用 (req, res) 双参数，Koa 使用统一的 ctx 对象。Express 内置路由，Koa 核心更精简需额外引入路由中间件。

**Q5: Express 错误处理中间件为什么必须定义 4 个参数？**
> Express 通过函数参数个数（length）来区分普通中间件（3 个参数）和错误处理中间件（4 个参数）。当 next(err) 被调用时，Express 会跳过所有 3 参数中间件，找到第一个 4 参数的中间件执行。如果省略 err 参数，Express 会将其视为普通中间件，导致错误无法被捕获。

---

**相关链接：**
- [[事件循环]]
- [[Koa中间件机制]]
- [[Fastify]]
- [[RESTful API设计]]
- [[认证与授权]]

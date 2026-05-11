---
tags:
  - Node.js
  - Koa
  - 中间件
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Koa中间件机制

## What — 是什么

> Koa 的中间件基于洋葱模型（Onion Model），请求从外层穿入、响应从内层穿出，每个中间件可通过 `await next()` 将控制权交给下一层，完成后恢复执行。

**核心概念：**

- **洋葱模型**：请求依次穿过中间件 1→2→3，响应逆序 3→2→1
- **async/await**：中间件天然是 async 函数，告别回调地狱
- **Context（ctx）**：封装 request 和 response，替代 Express 的 (req, res) 双参数
- **`next()`**：返回 Promise，resolve 后继续执行当前中间件的后置逻辑

**核心架构：**

- 设计理念：更小、更富表现力、更健壮
- 核心模块：Application、Context、compose（koa-compose）
- 数据流：Request → mw1 前置 → mw2 前置 → mw3 → mw2 后置 → mw1 后置 → Response

**插件生态：**

- 官方插件：koa-router、koa-bodyparser、koa-static
- 社区热门插件：koa-jwt、koa-cors、koa-logger

## Why — 为什么

**适用场景：**

- 需要前后置逻辑的中间件（日志、认证、错误处理）
- 更优雅的异步流程控制
- 希望精简核心、按需组装的团队

**对比同类框架：**

| 维度 | Koa | Express | Fastify |
|------|-----|---------|---------|
| 性能 | 中 | 中 | 极高 |
| 生态 | 中等 | 最丰富 | 中等 |
| 学习曲线 | 中（理解洋葱模型） | 最低 | 中 |
| 灵活性 | 极高 | 高 | 中 |

**优缺点：**

- ✅ 优点：
  - 洋葱模型优雅处理前后置逻辑
  - 原生 async/await，代码清晰
  - 核心极简（仅 ~600 行），可定制性强
  - 统一的 Context 对象
- ❌ 缺点：
  - 生态不如 Express 丰富
  - 路由、body 解析等需要额外安装中间件
  - 社区规模较小

## How — 怎么用

### 快速上手

```javascript
const Koa = require('koa');
const app = new Koa();

// 洋葱模型示例
app.use(async (ctx, next) => {
    console.log('1 - 进入外层');
    await next();
    console.log('4 - 离开外层');
    ctx.body = 'Hello';
});

app.use(async (ctx, next) => {
    console.log('2 - 进入内层');
    await next();
    console.log('3 - 离开内层');
});

// 输出顺序：1 → 2 → 3 → 4

app.listen(3000);
```

### 代码示例

**请求计时中间件：**

```javascript
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const ms = Date.now() - start;
    ctx.set('X-Response-Time', `${ms}ms`);
});
```

**统一错误处理：**

```javascript
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        ctx.status = err.status || 500;
        ctx.body = { error: err.message };
        // 触发 app 级别 error 事件
        ctx.app.emit('error', err, ctx);
    }
});

app.on('error', (err, ctx) => {
    console.error('Server error:', err);
});
```

**compose 原理简化版：**

```javascript
function compose(middleware) {
    return (ctx, next) => {
        let index = -1;
        function dispatch(i) {
            if (i <= index) return Promise.reject(new Error('next() called multiple times'));
            index = i;
            const fn = i === middleware.length ? next : middleware[i];
            if (!fn) return Promise.resolve();
            try {
                return Promise.resolve(fn(ctx, () => dispatch(i + 1)));
            } catch (err) {
                return Promise.reject(err);
            }
        }
        return dispatch(0);
    };
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `next()` 调用多次 | 同一中间件内多次 `await next()` | 确保只调用一次，或用标志位保护 |
| 响应体为空 | 忘记设置 `ctx.body` | 在最后执行的中间件中设置响应 |
| 中间件顺序错误 | 认证在路由之后注册 | 按功能正确排列：错误处理 → 日志 → 认证 → 路由 |

### 最佳实践

- 错误处理中间件放最前
- 日志/计时中间件放外层（最先注册）
- 使用 `koa-router` 组织路由
- 用 `koa-bodyparser` 替代手动解析 body

## 面试题

**Q1: 请解释 Koa 洋葱模型的原理，请求和响应是如何流经中间件的？**
> 洋葱模型中，中间件像洋葱的层层结构：请求从最外层中间件进入，依次向内执行前置逻辑；当遇到 await next() 时，控制权交给下一个中间件；到达最内层后，响应逆序返回，从内向外执行各中间件的后置逻辑。整个过程形成"先进后出"的调用栈结构，使每个中间件都能同时处理请求和响应。

**Q2: Koa 的 koa-compose 是如何实现中间件编排的？**
> koa-compose 通过递归的 dispatch 函数实现。dispatch(i) 执行第 i 个中间件，并将 dispatch(i+1) 作为 next 参数传入。中间件内调用 await next() 实际是调用 dispatch(i+1)，返回一个 Promise。当 Promise resolve 后，控制权回到当前中间件继续执行后置逻辑。还通过 index 变量检测 next() 是否被多次调用（防止死循环）。

**Q3: Koa 和 Express 中间件模型有什么本质区别？**
> Express 是线性模型：中间件按顺序执行，next() 调用后控制权交给下一个中间件，不再回来（无法处理响应阶段）。Koa 是洋葱模型：next() 返回 Promise，await next() 后控制权会回到当前中间件，能同时处理请求前和响应后的逻辑。因此 Koa 天然适合日志、计时、错误处理等需要前后置逻辑的场景，而 Express 需要借助 res.on('finish') 等方式实现。

**Q4: Koa 的 ctx 对象是如何实现的？request 和 response 的属性为何能直接通过 ctx 访问？**
> ctx 是通过 Object.create(context) 创建的，内部持有 request 和 response 两个对象。context 原型上通过 getter/setter 委托了 request 和 response 的常用属性（如 ctx.body 实际访问 ctx.response.body，ctx.method 实际访问 ctx.request.method）。这种委托模式让 API 更简洁，无需写 ctx.request.method，直接用 ctx.method 即可。

**Q5: Koa 中 next() 被多次调用会怎样？为什么？**
> Koa 会在 dispatch 中通过 index 变量检测：如果 next()（即 dispatch(i+1)）被调用时发现 i+1 <= index，说明同一层的 next() 已被执行过，直接 reject 抛出 "next() called multiple times" 错误。这是因为多次调用 next() 会导致同一中间件的后置逻辑执行多次，产生不可预期的行为，甚至死循环。

---

**相关链接：**
- [[Express]]
- [[事件循环]]

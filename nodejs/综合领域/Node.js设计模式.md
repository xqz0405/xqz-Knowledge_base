---
tags:
  - Node.js
  - 设计模式
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Node.js 设计模式

## What — 是什么

> Node.js 设计模式是解决 Node.js 开发中常见问题的可复用方案，涵盖观察者、策略、中间件、工厂、单例、发布订阅和装饰器模式。

**核心概念：**

- **观察者模式**：EventEmitter，对象状态变化时通知所有监听者
- **策略模式**：运行时切换算法，如不同支付策略（微信/支付宝/银行卡）
- **中间件模式**：Express/Koa 的请求处理管道，每个中间件处理请求后传递给下一个
- **工厂模式**：封装对象创建逻辑，如 `createConnection('mysql')` 返回不同数据库实例
- **单例模式**：确保类只有一个实例（Node.js 模块缓存天然单例）
- **发布订阅模式**：EventEmitter + 多个订阅者，解耦事件生产和消费
- **装饰器模式**：动态添加功能，如 `@log`/`@cache` 装饰器

**关键特性：**

- Node.js 模块缓存机制天然实现单例
- EventEmitter 是观察者/发布订阅的基础
- 中间件模式是 Express/Koa/Fastify 的核心架构
- 策略模式适合运行时切换行为

## Why — 为什么

**适用场景：**

- 事件驱动：Node.js 核心就是观察者模式（EventEmitter）
- 请求处理：中间件模式串联处理逻辑
- 支付/通知：策略模式切换不同实现
- 数据库连接：工厂模式创建不同类型实例

## How — 怎么用

### 代码示例

```javascript
// 观察者模式（EventEmitter）
const EventEmitter = require('events');
class UserService extends EventEmitter {
    async createUser(data) {
        const user = await db.save(data);
        this.emit('user:created', user); // 通知监听者
    }
}
const userService = new UserService();
userService.on('user:created', (user) => emailService.sendWelcome(user));
userService.on('user:created', (user) => analyticsService.track(user));

// 策略模式
const paymentStrategies = {
    wechat: (amount) => wechatPay(amount),
    alipay: (amount) => alipayPay(amount),
    card: (amount) => cardPay(amount)
};
function processPayment(method, amount) {
    const strategy = paymentStrategies[method];
    if (!strategy) throw new Error(`Unknown payment: ${method}`);
    return strategy(amount);
}

// 中间件模式
function compose(middlewares) {
    return (ctx) => {
        let index = -1;
        function dispatch(i) {
            if (i <= index) throw new Error('next() called multiple times');
            index = i;
            const fn = middlewares[i];
            if (!fn) return Promise.resolve();
            return fn(ctx, () => dispatch(i + 1));
        }
        return dispatch(0);
    };
}

// 工厂模式
function createDatabase(type, config) {
    switch (type) {
        case 'mysql': return new MySQLConnection(config);
        case 'postgres': return new PostgresConnection(config);
        case 'mongo': return new MongoConnection(config);
    }
}

// 单例模式（Node.js 模块缓存）
// database.js
let instance = null;
module.exports = {
    getInstance() {
        if (!instance) instance = new Database(config);
        return instance;
    }
};
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| EventEmitter 内存泄漏 | 监听器未移除 | `removeListener` 或 `once` |
| 中间件忘记 next() | 请求挂起 | 确保每个中间件调用 next |
| 策略模式膨胀 | 策略类过多 | 用配置驱动 + 插件注册 |

### 最佳实践

- 用 `once` 替代 `on` 防止重复监听
- 中间件模式统一错误处理
- 策略模式用对象映射替代 switch
- 单例通过 Node.js 模块缓存实现

## 面试题

**Q1: Node.js 模块缓存如何实现单例？**
> Node.js 的 `require()` 缓存机制：首次 `require('./db')` 时执行模块代码并缓存 `module.exports`，后续 `require('./db')` 直接返回缓存。因此模块中创建的对象（如数据库连接）天然是单例——所有 `require` 引用同一个实例。不需要手动实现单例模式，只需在模块顶层创建实例并 `module.exports` 导出即可。

**Q2: 中间件模式的核心思想？**
> 中间件模式将请求处理拆分为多个独立函数，每个函数处理一个关注点（认证/日志/验证），按顺序执行形成管道。核心：① 每个中间件接收请求和响应对象，处理后调用 `next()` 传递给下一个；② 中间件可以在 `next()` 前后执行逻辑（Koa 洋葱模型）；③ 中间件可以提前终止请求（不调用 `next()`）；④ 错误中间件兜底处理异常。Express 线性管道，Koa 洋葱模型（可执行后置逻辑）。

---

**相关链接：**
- [[事件循环]]
- [[Express]]
- [[Koa中间件机制]]
- [[架构模式与分层]]

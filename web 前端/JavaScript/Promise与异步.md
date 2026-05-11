---
tags:
  - Web前端
  - JavaScript
  - Promise
  - 异步
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Promise与异步

## What — 是什么

> Promise 是 JavaScript 异步编程的核心原语，表示一个异步操作的最终结果。async/await 是 Promise 的语法糖，让异步代码看起来像同步代码。

**核心概念：**

- **三种状态**：`pending` → `fulfilled`（resolved）或 `rejected`，状态一旦改变不可逆
- **`then/catch/finally`**：链式调用处理结果和错误
- **`async function`**：始终返回 Promise，`await` 暂停执行直到 Promise settle
- **静态方法**：`Promise.all`、`Promise.race`、`Promise.allSettled`、`Promise.any`

**关键特性：**

- Promise 链中 `.then()` 返回新 Promise，可链式调用
- `await` 只能在 `async` 函数内使用（或 ES2022 顶层 await）
- 未处理的 rejection 会触发 `unhandledrejection` 事件

## Why — 为什么

**适用场景：**

- 网络请求（fetch）
- 文件读取
- 定时器封装
- 多个异步任务编排

**对比替代方案：**

| 维度 | async/await | Promise 链 | 回调函数 |
|------|------------|-----------|---------|
| 可读性 | 极高 | 中 | 低（回调地狱） |
| 错误处理 | try/catch | .catch() | 手动传递 error |
| 调试 | 容易（像同步） | 中等 | 困难 |
| 控制流 | 直观 | 需技巧 | 混乱 |

**优缺点：**

- ✅ 优点：
  - async/await 代码可读性极高
  - 错误处理统一（try/catch）
  - 调试体验接近同步代码
- ❌ 缺点：
  - `await` 容易被误用为顺序执行（忘记并发）
  - 循环中 `await` 需要特别注意（`for...of` 顺序 vs `Promise.all` 并发）
  - 无法取消（Promise 无 cancel 机制）

## How — 怎么用

### 快速上手

```javascript
// Promise 基础
const fetchUser = (id) =>
    new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) resolve({ id, name: 'Alice' });
            else reject(new Error('Invalid id'));
        }, 1000);
    });

// async/await
async function getUser(id) {
    try {
        const user = await fetchUser(id);
        console.log(user.name);
    } catch (e) {
        console.error(e.message);
    }
}
```

### 代码示例

**并发 vs 顺序：**

```javascript
// ❌ 顺序执行，总耗时 3s
const a = await fetch('/api/a'); // 1s
const b = await fetch('/api/b'); // 1s
const c = await fetch('/api/c'); // 1s

// ✅ 并发执行，总耗时 1s
const [a, b, c] = await Promise.all([
    fetch('/api/a'),
    fetch('/api/b'),
    fetch('/api/c'),
]);
```

**静态方法对比：**

```javascript
// Promise.all：一个失败则全部失败
await Promise.all([p1, p2, p3]);

// Promise.allSettled：等所有完成，不论成功失败
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(r => {
    if (r.status === 'fulfilled') handleSuccess(r.value);
    else handleError(r.reason);
});

// Promise.any：取第一个成功的
await Promise.any([p1, p2, p3]);

// Promise.race：取第一个完成的（无论成败）
await Promise.race([fetch(url), timeout(5000)]);
```

**循环中的异步：**

```javascript
// ❌ forEach 不等 await
items.forEach(async (item) => {
    await process(item); // 不会等！
});

// ✅ for...of 顺序执行
for (const item of items) {
    await process(item);
}

// ✅ Promise.all 并发执行
await Promise.all(items.map(item => process(item)));
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| await 误用为顺序执行 | 每次都等上一个完成 | 互不依赖的用 `Promise.all` 并发 |
| forEach 中 await 无效 | `forEach` 不等回调完成 | 用 `for...of` 或 `Promise.all` + `map` |
| 忘记 catch | `async` 函数 rejection 未处理 | 始终 try/catch 或 `.catch()` |
| Promise 构造反模式 | 用 `new Promise` 包裹已有的 Promise | 直接 return 已有的 Promise |

### 最佳实践

- 互不依赖的异步操作用 `Promise.all` 并发
- 循环中用 `for...of`（顺序）或 `Promise.all` + `map`（并发）
- 错误处理不可省略，全局兜底 `unhandledrejection`
- 避免用 `new Promise` 包裹已有 Promise（反模式）

## 面试题

**Q1: 请描述 JavaScript 事件循环（Event Loop）的执行机制。**
> 事件循环不断从调用栈取出任务执行。调用栈清空后，先清空微任务队列（Promise.then、MutationObserver、queueMicrotask），然后取一个宏任务执行（setTimeout、setInterval、I/O），再清空微任务，如此循环。微任务优先级高于宏任务。

**Q2: Promise 有哪几种状态？状态转换有什么规则？**
> 三种状态：`pending`（等待中）、`fulfilled`（已成功）、`rejected`（已失败）。状态一旦从 `pending` 变为 `fulfilled` 或 `rejected` 就不可逆，称为 resolved（已定型）。`resolve` 和 `reject` 只会执行先调用的那个。

**Q3: 微任务和宏任务有什么区别？各有哪些？**
> 微任务在当前宏任务结束后立即执行，优先级更高；宏任务在下一轮事件循环中执行。微任务：Promise.then/catch/finally、MutationObserver、queueMicrotask。宏任务：setTimeout、setInterval、setImmediate（Node）、I/O、UI 渲染。

**Q4: 如何实现并发控制（如限制同时请求数不超过 3 个）？**
> 核心思路：维护一个执行池和等待队列。当有任务进来时，若执行池未满则直接执行，否则放入等待队列。某个任务完成后从队列取出下一个执行。可以用 `Promise` + 计数器或 `p-limit` 等库实现。注意与 `Promise.all` 的区别：`Promise.all` 全部并发，无法限制并发数。

---

**相关链接：**
- [[作用域与闭包]]
- [[ES6+核心特性]]
- [[DOM与事件机制]]

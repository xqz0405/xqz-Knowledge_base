---
tags:
  - Web前端
  - JavaScript
  - Promise
  - 异步
  - async/await
  - 事件循环
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Promise与异步

## What — 是什么

> Promise 是 JavaScript 异步编程的核心原语，表示一个异步操作的最终结果。async/await 是 Promise 的语法糖，让异步代码看起来像同步代码。理解 Promise 需要从异步编程的演进历程出发，掌握其状态机模型、API 体系以及在事件循环中的调度机制。

### 异步编程的演进

JavaScript 的异步编程经历了四个主要阶段：

| 阶段 | 代表方案 | 核心思想 | 主要问题 |
|------|---------|---------|---------|
| 回调函数 | `fs.readFile(path, callback)` | 将后续逻辑作为函数传入 | 回调地狱、错误处理混乱 |
| Promise | `fetch(url).then().catch()` | 用状态机封装异步结果，支持链式调用 | 链式过长仍不够直观、无法取消 |
| Generator | `function*` + `yield` + `co` | 用同步写法控制异步流程 | 需要执行器（co 库），半成品方案 |
| async/await | `async function` + `await` | Generator 的语法糖，内置执行器 | 无法取消、容易误用为顺序执行 |

**演进路线：**

```
回调地狱 → Promise 链式调用 → Generator + co → async/await（最终形态）
```

> Generator 是 async/await 的前身。`async/await` 本质上就是 Generator + 自动执行器的语法糖。`co` 库曾用于自动执行 Generator，后来被语言原生支持的 `async/await` 取代。

### Promise 的三种状态与状态转换

Promise 是一个有限状态机，只有三种状态：

```
                    resolve(value)
   pending ──────────────────────→ fulfilled（已成功）
      │
      │  reject(reason)
      └──────────────────────────→ rejected（已失败）
```

**状态转换规则：**

- 初始状态为 `pending`
- 只能从 `pending` 转为 `fulfilled` 或 `rejected`，不可逆
- 状态一旦改变就不可再变（称为 settled / 已定型）
- `resolve` 和 `reject` 同时调用时，只有先执行的那个生效
- `resolve` 另一个 Promise 时，状态取决于被 resolve 的 Promise

```javascript
// resolve 另一个 Promise —— 状态传递
const p1 = new Promise((resolve) => {
    setTimeout(() => resolve('p1 完成'), 2000);
});
const p2 = new Promise((resolve) => {
    resolve(p1); // p2 的状态由 p1 决定
});
p2.then(val => console.log(val)); // 2s 后输出 "p1 完成"
```

**注意：`resolved` 不等于 `fulfilled`**

- `fulfilled` 和 `rejected` 是两种终态
- `resolved` 表示"已定型"，可能是 fulfilled 也可能是 rejected
- 更准确地说：`resolve()` 可能还处于 pending（当 resolve 了一个 pending 的 Promise 时）

### Promise 完整 API 列表

| API | 说明 | 返回值 |
|-----|------|--------|
| `new Promise(executor)` | 创建 Promise，executor 立即执行 | Promise 实例 |
| `Promise.resolve(value)` | 创建已 fulfilled 的 Promise | Promise |
| `Promise.reject(reason)` | 创建已 rejected 的 Promise | Promise |
| `Promise.all(iterable)` | 全部成功才成功，一个失败即失败 | Promise |
| `Promise.allSettled(iterable)` | 等所有 settle，不论成功失败 | Promise（数组含 status/value/reason） |
| `Promise.race(iterable)` | 取第一个 settle 的结果 | Promise |
| `Promise.any(iterable)` | 取第一个 fulfilled 的，全失败才失败 | Promise |
| `promise.then(onFulfilled, onRejected)` | 注册成功/失败回调 | 新 Promise |
| `promise.catch(onRejected)` | `.then(null, onRejected)` 的语法糖 | 新 Promise |
| `promise.finally(onFinally)` | 无论成败都执行，不接收参数 | 新 Promise |

**`finally` 的特殊行为：**

```javascript
// finally 不接收参数，且默认透传原值
Promise.resolve(42)
    .finally(() => console.log('清理')) // 输出 "清理"
    .then(val => console.log(val));     // 输出 42（原值透传）

// finally 中抛出异常会覆盖原值
Promise.resolve(42)
    .finally(() => { throw new Error('finally 出错'); })
    .catch(e => console.log(e.message)); // "finally 出错"
```

### 微任务与宏任务

Promise 的回调（`.then()`/`.catch()`/`.finally()`）属于**微任务（Microtask）**，在当前宏任务结束后立即执行，优先级高于宏任务。

**事件循环执行顺序：**

```
1. 执行同步代码（调用栈）
2. 调用栈清空 → 清空微任务队列（全部执行完）
3. 取一个宏任务执行
4. 宏任务执行完 → 再次清空微任务队列
5. 重复 3-4
```

| 分类 | 包含 | 执行时机 |
|------|------|---------|
| 微任务 | `Promise.then/catch/finally`、`MutationObserver`、`queueMicrotask()`、`process.nextTick`（Node） | 当前宏任务结束后立即执行 |
| 宏任务 | `setTimeout`、`setInterval`、`setImmediate`（Node）、`I/O`、UI 渲染、`requestAnimationFrame` | 下一轮事件循环 |

**执行顺序示例：**

```javascript
console.log('1: 同步');

setTimeout(() => console.log('2: 宏任务'), 0);

Promise.resolve()
    .then(() => console.log('3: 微任务1'))
    .then(() => console.log('4: 微任务2'));

queueMicrotask(() => console.log('5: 微任务3'));

console.log('6: 同步');

// 输出顺序：1 → 6 → 3 → 5 → 4 → 2
```

### async/await 的本质

`async/await` 是 Generator + 自动执行器的语法糖：

```javascript
// async/await 写法
async function fetchUserData(id) {
    try {
        const user = await fetchUser(id);
        const posts = await fetchPosts(user.id);
        return { user, posts };
    } catch (e) {
        console.error(e);
    }
}

// 等价的 Generator + co 写法
function* fetchUserData(id) {
    try {
        const user = yield fetchUser(id);
        const posts = yield fetchPosts(user.id);
        return { user, posts };
    } catch (e) {
        console.error(e);
    }
}
// co(fetchUserData, 1); // 需要执行器
```

**关键规则：**

- `async` 函数始终返回 Promise，即使 return 的是普通值
- `await` 右侧的表达式会被 `Promise.resolve()` 包装
- `await` 暂停当前 async 函数的执行，将后续代码注册为微任务
- 顶层 `await`（ES2022）可在模块顶层直接使用，无需包裹 async 函数

### 错误处理机制

**两种错误处理方式的对比：**

| 方式 | 适用场景 | 特点 |
|------|---------|------|
| `try/catch` | async/await | 同步风格，可捕获多个 await 的错误 |
| `.catch()` | Promise 链式调用 | 链式风格，可精确捕获某一步的错误 |

```javascript
// 方式1：try/catch（async/await）
async function loadPage() {
    try {
        const user = await fetchUser();
        const posts = await fetchPosts(user.id);
    } catch (e) {
        // 捕获 fetchUser 或 fetchPosts 的错误
        console.error('加载失败:', e);
    }
}

// 方式2：.catch()（Promise 链）
fetchUser()
    .then(user => fetchPosts(user.id))
    .catch(e => {
        // 捕获链中任意环节的错误
        console.error('加载失败:', e);
    });

// 方式3：混合——精确捕获某一步
async function loadPage() {
    const user = await fetchUser().catch(e => {
        console.error('用户加载失败，使用默认值');
        return { id: 0, name: 'Guest' };
    });
    // 即使用户加载失败，仍继续执行
    const posts = await fetchPosts(user.id);
}
```

**未捕获的 rejection：**

```javascript
// 全局兜底
window.addEventListener('unhandledrejection', (event) => {
    console.error('未处理的 Promise rejection:', event.reason);
    event.preventDefault(); // 阻止控制台报错
});
```

## Why — 为什么

### 回调地狱的问题

在 Promise 出现之前，异步操作只能通过回调函数处理，多个异步操作嵌套形成"回调地狱"：

```javascript
// ❌ 回调地狱：难以阅读、难以维护、难以错误处理
getUser(id, (err, user) => {
    if (err) return handleError(err);
    getPosts(user.id, (err, posts) => {
        if (err) return handleError(err);
        getComments(posts[0].id, (err, comments) => {
            if (err) return handleError(err);
            getAuthor(comments[0].authorId, (err, author) => {
                if (err) return handleError(err);
                // 终于拿到所有数据...
            });
        });
    });
});
```

**回调地狱的核心问题：**

- **可读性差**：深层嵌套，代码呈"金字塔"结构
- **错误处理混乱**：每层都要手动判断 err，容易遗漏
- **无法 return/throw**：回调中的 return 只是退出回调，不是退出外层函数
- **难以调试**：调用栈不完整，异常难以追踪
- **信任问题**：回调可能被调用多次、不调用或调用时机不对

### Promise 解决了什么、还有什么不足

**Promise 解决的问题：**

- **链式调用**：`.then()` 返回新 Promise，扁平化嵌套
- **统一错误传播**：`.catch()` 统一捕获，错误会沿链向下传递
- **状态不可逆**：resolve/reject 只生效一次，解决信任问题
- **组合能力**：`Promise.all`/`race`/`any` 等提供并发控制

**Promise 仍未解决的问题：**

- **无法取消**：一旦创建无法中止（需 AbortController 辅助）
- **无法获取进度**：只有 pending 和 settled，没有 progress
- **链式仍不够直观**：复杂流程中 `.then()` 链过长
- **单个值语义**：每次 `.then()` 只能传一个值（需解构）
- **总是异步**：即使已 settled，`.then()` 回调也是异步执行

### async/await 的优势

- **同步写法**：代码从上到下线性执行，可读性极高
- **try/catch**：用同步风格的错误处理统一异步错误
- **调试友好**：调用栈完整，断点调试体验接近同步代码
- **条件逻辑自然**：`if/else`、`for` 循环中可直接 `await`

### 对比总览：回调 vs Promise vs async/await vs Generator

| 维度 | 回调函数 | Promise 链 | async/await | Generator |
|------|---------|-----------|-------------|-----------|
| 可读性 | 低（嵌套） | 中（链式） | 极高（同步风格） | 高（需 yield） |
| 错误处理 | 手动传递 | .catch() 链式 | try/catch | try/catch |
| 调试体验 | 差（调用栈断裂） | 中等 | 好（接近同步） | 中等 |
| 流程控制 | 困难 | 需技巧 | 直观 | 灵活 |
| 取消能力 | 可控 | 无 | 无 | 可控（return/throw） |
| 惰性求值 | 不适用 | 不适用 | 不支持 | 原生支持 |
| 学习成本 | 低 | 中 | 低 | 高 |
| 运行时支持 | 全部 | ES6+ | ES2017+ | ES6+ |
| 典型用途 | 简单事件回调 | 异步组合/并发 | 业务逻辑主流程 | 迭代/状态机/惰性序列 |

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

#### 示例1：Promise 基本用法（创建、链式调用、错误传播）

```javascript
// 创建 Promise
const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

// 链式调用——每步返回值传给下一步
delay(1000)
    .then(() => {
        console.log('1秒后');
        return fetchUser(1);
    })
    .then(user => {
        console.log('用户:', user.name);
        return fetchPosts(user.id);
    })
    .then(posts => console.log('文章数:', posts.length))
    .catch(err => console.error('链中任意环节出错:', err.message))
    .finally(() => console.log('清理资源'));

// 错误传播——错误会沿链向下传递
Promise.resolve('ok')
    .then(() => { throw new Error('第二步出错'); })
    .then(() => console.log('不会执行'))
    .then(() => console.log('也不会执行'))
    .catch(err => console.log('捕获到:', err.message)); // "第二步出错"
```

#### 示例2：Promise.all 并发控制

```javascript
// Promise.all：全部成功才成功
async function loadDashboard() {
    try {
        const [user, posts, notifications] = await Promise.all([
            fetch('/api/user').then(r => r.json()),
            fetch('/api/posts').then(r => r.json()),
            fetch('/api/notifications').then(r => r.json()),
        ]);
        render({ user, posts, notifications });
    } catch (e) {
        // 任一请求失败，整体失败
        showError('加载失败');
    }
}

// ❌ 常见误区：顺序 await 导致不必要等待
async function loadDashboardSlow() {
    const user = await fetch('/api/user').then(r => r.json());     // 等 1s
    const posts = await fetch('/api/posts').then(r => r.json());   // 再等 1s
    const notifications = await fetch('/api/notifications').then(r => r.json()); // 再等 1s
    // 总耗时 3s，而非 1s！
}
```

#### 示例3：Promise.allSettled 容错并发

```javascript
// allSettled：不因个别失败而中断，适合"尽力而为"场景
async function loadMultipleAPIs() {
    const results = await Promise.allSettled([
        fetch('/api/user').then(r => r.json()),
        fetch('/api/posts').then(r => r.json()),
        fetch('/api/comments').then(r => r.json()),  // 这个可能 404
    ]);

    const data = {};
    results.forEach((result, index) => {
        const keys = ['user', 'posts', 'comments'];
        if (result.status === 'fulfilled') {
            data[keys[index]] = result.value;
        } else {
            console.warn(`${keys[index]} 加载失败:`, result.reason.message);
            data[keys[index]] = null; // 使用默认值
        }
    });
    return data;
}

// allSettled 返回值结构
// [
//   { status: 'fulfilled', value: {...} },
//   { status: 'rejected',  reason: Error },
//   { status: 'fulfilled', value: {...} },
// ]
```

#### 示例4：Promise.race / Promise.any 超时控制

```javascript
// Promise.race：超时控制——取第一个 settle 的结果
function fetchWithTimeout(url, ms = 5000) {
    const timeout = new Promise((_, reject) =>
        setTimeout(() => reject(new Error(`请求超时 ${ms}ms`)), ms)
    );
    return Promise.race([fetch(url), timeout]);
}

// 使用
fetchWithTimeout('/api/data', 3000)
    .then(r => r.json())
    .then(data => console.log(data))
    .catch(e => console.error(e.message)); // 可能输出 "请求超时 3000ms"

// Promise.any：多源竞速——取第一个成功的
async function fetchFromMirrors(url) {
    const mirrors = [
        fetch(`https://mirror1.com${url}`),
        fetch(`https://mirror2.com${url}`),
        fetch(`https://mirror3.com${url}`),
    ];
    try {
        const response = await Promise.any(mirrors);
        return response.json();
    } catch (e) {
        // AggregateError：所有 Promise 都失败时抛出
        console.error('所有镜像均失败:', e.errors);
    }
}
```

#### 示例5：async/await 完整实战（含错误处理、并发控制）

```javascript
// 场景：加载用户页面的完整数据
async function loadUserPage(userId) {
    // 1. 并发加载独立数据
    const [user, settings] = await Promise.all([
        fetchUser(userId).catch(() => ({ id: userId, name: '未知用户' })),
        fetchSettings(userId).catch(() => ({ theme: 'light', lang: 'zh' })),
    ]);

    // 2. 依赖前置结果的请求
    let posts = [];
    try {
        posts = await fetchPosts(user.id);
    } catch (e) {
        console.warn('文章加载失败，显示空列表');
    }

    // 3. 条件请求
    if (posts.length > 0) {
        try {
            const comments = await fetchComments(posts[0].id);
            posts[0].comments = comments;
        } catch (e) {
            posts[0].comments = [];
        }
    }

    return { user, settings, posts };
}

// 使用
loadUserPage(1)
    .then(renderPage)
    .catch(e => showErrorPage(e));
```

#### 示例6：手写 Promise（简易版，含 then 链式调用）

```javascript
class MyPromise {
    constructor(executor) {
        this.status = 'pending';
        this.value = undefined;
        this.reason = undefined;
        this.onFulfilledCallbacks = [];
        this.onRejectedCallbacks = [];

        const resolve = (value) => {
            if (this.status === 'pending') {
                this.status = 'fulfilled';
                this.value = value;
                this.onFulfilledCallbacks.forEach(fn => fn());
            }
        };

        const reject = (reason) => {
            if (this.status === 'pending') {
                this.status = 'rejected';
                this.reason = reason;
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        };

        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }

    then(onFulfilled, onRejected) {
        // 值穿透：如果不是函数，则透传
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (v) => v;
        onRejected = typeof onRejected === 'function' ? onRejected : (e) => { throw e; };

        // then 返回新 Promise 实现链式调用
        const promise2 = new MyPromise((resolve, reject) => {
            const fulfilledTask = () => {
                queueMicrotask(() => {
                    try {
                        const x = onFulfilled(this.value);
                        // 如果返回值是 Promise，等待其 settle
                        if (x instanceof MyPromise) {
                            x.then(resolve, reject);
                        } else {
                            resolve(x);
                        }
                    } catch (e) {
                        reject(e);
                    }
                });
            };

            const rejectedTask = () => {
                queueMicrotask(() => {
                    try {
                        const x = onRejected(this.reason);
                        if (x instanceof MyPromise) {
                            x.then(resolve, reject);
                        } else {
                            resolve(x); // catch 后恢复为 fulfilled
                        }
                    } catch (e) {
                        reject(e);
                    }
                });
            };

            if (this.status === 'fulfilled') fulfilledTask();
            else if (this.status === 'rejected') rejectedTask();
            else {
                // pending 状态：先存起来，状态改变时再执行
                this.onFulfilledCallbacks.push(fulfilledTask);
                this.onRejectedCallbacks.push(rejectedTask);
            }
        });

        return promise2;
    }

    catch(onRejected) {
        return this.then(null, onRejected);
    }

    static resolve(value) {
        if (value instanceof MyPromise) return value;
        return new MyPromise((resolve) => resolve(value));
    }

    static reject(reason) {
        return new MyPromise((_, reject) => reject(reason));
    }
}

// 测试
new MyPromise((resolve) => resolve(1))
    .then(v => v + 1)
    .then(v => { console.log(v); return v * 2; }) // 2
    .then(v => console.log(v))                      // 4
```

#### 示例7：异步并发限制（限制同时请求数量）

```javascript
// 核心思路：维护执行池 + 等待队列
function createConcurrencyLimit(maxConcurrency) {
    const queue = [];       // 等待队列
    let running = 0;        // 当前运行数

    function next() {
        // 还有空闲位 且 队列有任务 → 取出执行
        if (running < maxConcurrency && queue.length > 0) {
            running++;
            const { task, resolve, reject } = queue.shift();
            task()
                .then(resolve)
                .catch(reject)
                .finally(() => {
                    running--;
                    next(); // 完成一个，启动下一个
                });
        }
    }

    return function run(task) {
        return new Promise((resolve, reject) => {
            queue.push({ task, resolve, reject });
            next();
        });
    };
}

// 使用：限制同时请求数不超过 3
const limit = createConcurrencyLimit(3);
const urls = Array.from({ length: 10 }, (_, i) => `/api/item/${i}`);

Promise.all(
    urls.map(url => limit(() => fetch(url).then(r => r.json())))
).then(results => console.log('全部完成:', results.length));
```

#### 示例8：异步重试机制

```javascript
// 带退避的重试函数
async function retry(fn, options = {}) {
    const {
        retries = 3,          // 最大重试次数
        delay = 1000,         // 初始延迟
        backoff = 2,          // 退避倍数
        shouldRetry = () => true,  // 是否重试的判断函数
    } = options;

    let lastError;
    for (let i = 0; i <= retries; i++) {
        try {
            return await fn();
        } catch (e) {
            lastError = e;
            if (i === retries || !shouldRetry(e)) throw e;

            const waitTime = delay * Math.pow(backoff, i);
            console.warn(`第 ${i + 1} 次重试，${waitTime}ms 后执行...`);
            await new Promise(r => setTimeout(r, waitTime));
        }
    }
    throw lastError;
}

// 使用：网络请求重试
const data = await retry(
    () => fetch('/api/unstable').then(r => {
        if (!r.ok) throw new Error(`HTTP ${r.status}`);
        return r.json();
    }),
    {
        retries: 5,
        delay: 500,
        backoff: 2,
        shouldRetry: (e) => e.message.includes('503'), // 仅 503 重试
    }
);
```

#### 示例9：AbortController 取消异步请求

```javascript
// Promise 本身无法取消，但可通过 AbortController 取消 fetch
async function fetchWithAbort(url, signal) {
    const response = await fetch(url, { signal });
    return response.json();
}

// 基本用法
const controller = new AbortController();
fetchWithAbort('/api/data', controller.signal)
    .then(data => console.log(data))
    .catch(e => {
        if (e.name === 'AbortError') {
            console.log('请求已取消');
        } else {
            console.error('请求失败:', e);
        }
    });

// 超时自动取消
function fetchWithAutoAbort(url, ms = 5000) {
    const controller = new AbortController();
    const timer = setTimeout(() => controller.abort(), ms);
    return fetch(url, { signal: controller.signal })
        .then(r => r.json())
        .finally(() => clearTimeout(timer));
}

// 取消多个请求
const controller = new AbortController();
Promise.all([
    fetch('/api/a', { signal: controller.signal }),
    fetch('/api/b', { signal: controller.signal }),
    fetch('/api/c', { signal: controller.signal }),
]).then(results => console.log(results));

// 用户操作触发取消
// controller.abort(); // 三个请求全部取消
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| await 误用为顺序执行 | 每次都等上一个完成 | 互不依赖的用 `Promise.all` 并发 |
| forEach 中 await 无效 | `forEach` 不等回调完成 | 用 `for...of` 或 `Promise.all` + `map` |
| 忘记 catch | `async` 函数 rejection 未处理 | 始终 try/catch 或 `.catch()`，全局兜底 `unhandledrejection` |
| Promise 构造反模式 | 用 `new Promise` 包裹已有的 Promise | 直接 return 已有的 Promise |
| then 中返回 Promise 被嵌套 | `.then(p => p.then(...))` | 直接 `return` Promise，会自动展开 |
| async 函数忘记 await | `const data = asyncFn()` 拿到 Promise 而非值 | 始终 `const data = await asyncFn()` |
| 循环中并发 vs 顺序混淆 | 不清楚何时该用哪种 | 顺序用 `for...of`，并发用 `Promise.all` + `map` |
| then/catch 顺序错误 | `.then().catch()` 中 catch 只捕获前面 | 把 catch 放在正确的位置，或用 try/catch |

**Promise 构造反模式详解：**

```javascript
// ❌ 反模式：用 new Promise 包裹已有的 Promise
function fetchUser(id) {
    return new Promise((resolve, reject) => {
        fetch(`/api/user/${id}`)
            .then(res => resolve(res.json()))
            .catch(err => reject(err));
    });
}

// ✅ 正确写法：直接返回
function fetchUser(id) {
    return fetch(`/api/user/${id}`).then(res => res.json());
}
```

### 最佳实践

- 互不依赖的异步操作用 `Promise.all` 并发
- 循环中用 `for...of`（顺序）或 `Promise.all` + `map`（并发）
- 错误处理不可省略，全局兜底 `unhandledrejection`
- 避免用 `new Promise` 包裹已有 Promise（反模式）
- 需要取消异步操作时使用 `AbortController`
- 不稳定的请求使用重试机制（带指数退避）
- 大量并发请求使用并发限制，避免打爆服务器
- `Promise.allSettled` 适用于"尽力而为"场景，不因个别失败影响整体
- `async` 函数始终 `await` 调用，避免拿到 Promise 对象而非值

## 面试题

**Q1: 请描述 JavaScript 事件循环（Event Loop）的执行机制。**
> 事件循环不断从调用栈取出任务执行。调用栈清空后，先清空微任务队列（Promise.then、MutationObserver、queueMicrotask），然后取一个宏任务执行（setTimeout、setInterval、I/O），再清空微任务，如此循环。微任务优先级高于宏任务。

**Q2: Promise 有哪几种状态？状态转换有什么规则？**
> 三种状态：`pending`（等待中）、`fulfilled`（已成功）、`rejected`（已失败）。状态一旦从 `pending` 变为 `fulfilled` 或 `rejected` 就不可逆，称为 settled（已定型）。`resolve` 和 `reject` 只会执行先调用的那个。`resolve` 另一个 Promise 时，状态取决于被 resolve 的 Promise。

**Q3: 微任务和宏任务有什么区别？各有哪些？Promise 属于哪个？**
> 微任务在当前宏任务结束后立即执行，优先级更高；宏任务在下一轮事件循环中执行。微任务：Promise.then/catch/finally、MutationObserver、queueMicrotask、process.nextTick（Node）。宏任务：setTimeout、setInterval、setImmediate（Node）、I/O、UI 渲染。**Promise 本身的 executor 是同步执行的，只有 `.then()`/`.catch()`/`.finally()` 的回调属于微任务。**

**Q4: Promise.all 和 Promise.allSettled 的区别？**
> `Promise.all`：全部成功才成功，一个失败即短路返回该失败原因（fast-fail）。`Promise.allSettled`：等待所有 Promise 都 settle，不论成功失败，返回数组中每项包含 `{ status, value/reason }`。all 适合"全部必须成功"的场景，allSettled 适合"尽力而为、部分失败可接受"的场景。

**Q5: async 函数的错误处理有哪些方式？**
> 三种主要方式：（1）`try/catch` 包裹 await 代码块，同步风格处理错误；（2）`await promise.catch(e => ...)` 精确捕获单个 await 的错误并恢复；（3）将 async 函数返回的 Promise 用 `.catch()` 统一捕获。另外可用 `window.addEventListener('unhandledrejection', ...)` 做全局兜底。推荐在业务逻辑中用 try/catch，在全局用 unhandledrejection 兜底。

**Q6: 如何实现异步并发限制？**
> 核心思路：维护一个执行池（计数器或数组）和等待队列。当有任务进来时，若执行池未满（`running < maxConcurrency`）则直接执行，否则放入等待队列。某个任务完成后从队列取出下一个执行。可以用 Promise + 计数器手动实现，或使用 `p-limit`、`p-queue` 等库。注意与 `Promise.all` 的区别：`Promise.all` 全部并发，无法限制并发数。

**Q7: 如何取消一个已经发起的异步请求？**
> Promise 本身没有取消机制。在浏览器中使用 `AbortController`：创建 controller 实例，将 `controller.signal` 传入 `fetch` 的 options，调用 `controller.abort()` 即可取消请求，fetch 会抛出 `AbortError`。一个 AbortController 可同时取消多个请求。Node.js 的 http 请求也可通过 `request.destroy()` 取消。

**Q8: 手写 Promise.all 的实现？**
> ```javascript
> Promise.myAll = function(promises) {
>     return new Promise((resolve, reject) => {
>         const results = [];
>         let count = 0;
>         const len = promises.length;
>         if (len === 0) return resolve([]);
>         promises.forEach((p, i) => {
>             Promise.resolve(p).then(val => {
>                 results[i] = val;
>                 if (++count === len) resolve(results);
>             }).catch(reject);
>         });
>     });
> };
> ```
> 关键点：用 `Promise.resolve(p)` 包装确保兼容非 Promise 值；用索引赋值保证结果顺序；任一失败立即 reject；空数组直接 resolve。

---

**相关链接：**
- [[作用域与闭包]]
- [[ES6+核心特性]]
- [[DOM与事件机制]]
- [[迭代器与生成器]]
- [[错误处理与异常模式]]
- [[Web Worker与并行计算]]
- [[事件循环与异步模型]]
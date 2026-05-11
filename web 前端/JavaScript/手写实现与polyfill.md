---
tags:
  - Web前端
  - JavaScript
  - 手写实现
  - 面试
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# 手写实现与polyfill

## What — 是什么

> 手写实现是理解 JavaScript 内置 API 底层原理的方式，通过从零还原 `call/apply/bind`、`Promise`、`new` 等核心方法，深入理解语言机制。

**核心范围：**

- **函数方法**：`call`、`apply`、`bind`
- **面向对象**：`new`、`instanceof`
- **异步**：`Promise`、`Promise.all`、`Promise.race`
- **数组方法**：`map`、`filter`、`reduce`、`flat`
- **工具函数**：防抖、节流、深拷贝、柯里化
- **语言特性**：`Object.create`、`async/await`（generator 模拟）

**关键特性：**

- 手写实现关注核心逻辑，不追求 100% 兼容边界情况
- 每个实现都是对应 API 原理的最佳说明
- 面试高频考点

## Why — 为什么

**适用场景：**

- 面试准备（手写题占比 30%+）
- 深入理解 JS 内部机制
- polyfill 开发（为旧浏览器提供新 API）

**手写的价值：**

| 手写项 | 理解的原理 |
|--------|-----------|
| `call/apply/bind` | this 绑定机制 |
| `new` | 对象创建与原型链 |
| `Promise` | 微任务与状态机 |
| `instanceof` | 原型链查找 |
| `async/await` | generator + 自动执行器 |

## How — 怎么用

### 函数方法

**call：**

```javascript
Function.prototype.myCall = function (context, ...args) {
    context = context ?? window;
    const fn = Symbol('fn'); // 防止覆盖原有属性
    context[fn] = this;
    const result = context[fn](...args);
    delete context[fn];
    return result;
};

// 使用
function greet(greeting) {
    return `${greeting}, ${this.name}`;
}
greet.myCall({ name: 'Alice' }, 'Hello'); // "Hello, Alice"
```

**apply：**

```javascript
Function.prototype.myApply = function (context, args) {
    context = context ?? window;
    const fn = Symbol('fn');
    context[fn] = this;
    const result = context[fn](...args);
    delete context[fn];
    return result;
};
```

**bind：**

```javascript
Function.prototype.myBind = function (context, ...bindArgs) {
    const self = this;
    const bound = function (...callArgs) {
        // new 调用时 this 指向新实例，忽略 context
        return self.apply(
            this instanceof bound ? this : context,
            [...bindArgs, ...callArgs]
        );
    };
    // 继承原型
    bound.prototype = Object.create(self.prototype);
    return bound;
};
```

### 面向对象

**new：**

```javascript
function myNew(Constructor, ...args) {
    const obj = Object.create(Constructor.prototype);
    const result = Constructor.apply(obj, args);
    return result instanceof Object ? result : obj;
}

// 使用
function Person(name) { this.name = name; }
const p = myNew(Person, 'Alice'); // { name: 'Alice' }
```

**instanceof：**

```javascript
function myInstanceOf(obj, Constructor) {
    if (obj === null || typeof obj !== 'object') return false;
    let proto = Object.getPrototypeOf(obj);
    while (proto) {
        if (proto === Constructor.prototype) return true;
        proto = Object.getPrototypeOf(proto);
    }
    return false;
}
```

### 异步

**Promise：**

```javascript
class MyPromise {
    constructor(executor) {
        this.status = 'pending';
        this.value = undefined;
        this.reason = undefined;
        this.onFulfilledCbs = [];
        this.onRejectedCbs = [];

        const resolve = (value) => {
            if (this.status !== 'pending') return;
            this.status = 'fulfilled';
            this.value = value;
            this.onFulfilledCbs.forEach(fn => fn());
        };

        const reject = (reason) => {
            if (this.status !== 'pending') return;
            this.status = 'rejected';
            this.reason = reason;
            this.onRejectedCbs.forEach(fn => fn());
        };

        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }

    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
        onRejected = typeof onRejected === 'function' ? onRejected : e => { throw e; };

        const promise2 = new MyPromise((resolve, reject) => {
            const microTask = (fn) => queueMicrotask(fn);

            if (this.status === 'fulfilled') {
                microTask(() => {
                    try { resolve(onFulfilled(this.value)); }
                    catch (e) { reject(e); }
                });
            }
            if (this.status === 'rejected') {
                microTask(() => {
                    try { resolve(onRejected(this.reason)); }
                    catch (e) { reject(e); }
                });
            }
            if (this.status === 'pending') {
                this.onFulfilledCbs.push(() => {
                    microTask(() => {
                        try { resolve(onFulfilled(this.value)); }
                        catch (e) { reject(e); }
                    });
                });
                this.onRejectedCbs.push(() => {
                    microTask(() => {
                        try { resolve(onRejected(this.reason)); }
                        catch (e) { reject(e); }
                    });
                });
            }
        });
        return promise2;
    }

    catch(onRejected) {
        return this.then(null, onRejected);
    }
}
```

**Promise.all：**

```javascript
MyPromise.all = function (promises) {
    return new MyPromise((resolve, reject) => {
        const results = [];
        let count = 0;
        promises.forEach((p, i) => {
            MyPromise.resolve(p).then(value => {
                results[i] = value;
                if (++count === promises.length) resolve(results);
            }, reject);
        });
    });
};
```

### 工具函数

**深拷贝：**

```javascript
function deepClone(obj, map = new WeakMap()) {
    if (obj === null || typeof obj !== 'object') return obj;
    if (map.has(obj)) return map.get(obj); // 循环引用

    const clone = Array.isArray(obj) ? [] : {};
    map.set(obj, clone);

    // 处理特殊对象
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof RegExp) return new RegExp(obj);
    if (obj instanceof Map) {
        const m = new Map();
        obj.forEach((v, k) => m.set(deepClone(k, map), deepClone(v, map)));
        return m;
    }
    if (obj instanceof Set) {
        const s = new Set();
        obj.forEach(v => s.add(deepClone(v, map)));
        return s;
    }

    for (const key of Reflect.ownKeys(obj)) {
        clone[key] = deepClone(obj[key], map);
    }
    return clone;
}
```

**柯里化：**

```javascript
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) return fn(...args);
        return (...moreArgs) => curried(...args, ...moreArgs);
    };
}

// 使用
const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);   // 6
add(1, 2)(3);   // 6
add(1)(2, 3);   // 6
```

**发布订阅 EventEmitter：**

```javascript
class EventEmitter {
    constructor() { this.events = {}; }

    on(event, fn) {
        (this.events[event] ??= []).push(fn);
        return this;
    }

    emit(event, ...args) {
        (this.events[event] || []).forEach(fn => fn(...args));
        return this;
    }

    off(event, fn) {
        this.events[event] = (this.events[event] || []).filter(f => f !== fn);
        return this;
    }

    once(event, fn) {
        const wrapper = (...args) => {
            fn(...args);
            this.off(event, wrapper);
        };
        this.on(event, wrapper);
        return this;
    }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| bind 后 new 调用 this 不对 | 忘记判断 `this instanceof bound` | bind 实现需区分 new 调用和普通调用 |
| Promise 不支持异步 | then 同步执行了回调 | 用 `queueMicrotask` 确保异步 |
| 深拷贝循环引用栈溢出 | 对象互相引用无限递归 | 用 WeakMap 记录已拷贝对象 |
| 柯里化无法处理不定参 | 依赖 `fn.length` 判断参数个数 | 不定参场景改用占位符方案 |

### 最佳实践

- 手写优先理解核心逻辑，边界情况面试时口头说明即可
- Promise 实现抓住"状态机 + 微任务"两个关键
- bind 必须处理 new 调用和原型继承
- 深拷贝用 WeakMap 解决循环引用
- 练习时对照规范/MDN，理解每个细节

## 面试题

**Q1: 手写 `bind` 实现，关键点是什么？**
> 关键点：1) 保存原函数引用 `self = this`；2) 返回新函数，合并绑定参数和调用参数；3) 区分 `new` 调用和普通调用——`new` 时 `this` 指向新实例（通过 `this instanceof bound` 判断），忽略绑定的 `context`；4) 继承原函数原型 `bound.prototype = Object.create(self.prototype)`，保证 `instanceof` 正常工作。

**Q2: 手写 `new` 的实现过程？如果构造函数返回对象会怎样？**
> 步骤：1) 创建空对象 `obj = Object.create(Constructor.prototype)`；2) 执行构造函数 `result = Constructor.apply(obj, args)`；3) 如果 `result` 是对象（`result instanceof Object`）则返回 `result`，否则返回 `obj`。关键：构造函数显式返回对象时会替代新创建的对象，返回原始值则忽略。

**Q3: 手写 Promise 的核心原理是什么？**
> 核心是"状态机 + 微任务队列"。三个状态 `pending/fulfilled/rejected` 只能单向转换。`then` 返回新 Promise 实现链式调用。回调通过 `queueMicrotask` 异步执行（保证规范要求的异步行为）。`pending` 状态时将回调存入队列，状态变更时依次执行。`then` 的返回值需要透传（值穿透和 Promise 解析）。

**Q4: 深拷贝如何处理循环引用？**
> 用 `WeakMap` 记录已拷贝的对象：每次拷贝前检查 `map.has(obj)`，若存在则直接返回 `map.get(obj)`，否则先创建克隆对象并存入 `map`，再递归拷贝属性。`WeakMap` 用对象做键且是弱引用，不会阻止 GC。还需处理特殊对象：`Date`、`RegExp`、`Map`、`Set` 等，以及用 `Reflect.ownKeys` 拷贝 `Symbol` 属性。

---

**相关链接：**
- [[作用域与闭包]]
- [[原型与继承]]
- [[this指向与绑定]]
- [[Promise与异步]]

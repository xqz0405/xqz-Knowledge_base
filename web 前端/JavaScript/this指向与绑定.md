---
tags:
  - Web前端
  - JavaScript
  - this
  - 绑定
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# this指向与绑定

## What — 是什么

> `this` 是 JavaScript 函数执行时的上下文对象，其值取决于函数的调用方式，而非定义位置（箭头函数除外）。

**核心概念：**

- **默认绑定**：独立调用 `fn()`，非严格模式 `this === window`，严格模式 `this === undefined`
- **隐式绑定**：`obj.fn()`，`this` 指向 `obj`
- **显式绑定**：`fn.call(obj)` / `fn.apply(obj)` / `fn.bind(obj)`
- **new 绑定**：`new Fn()`，`this` 指向新创建的实例
- **箭头函数**：没有自己的 `this`，继承外层词法作用域的 `this`

**关键特性：**

- 优先级：new > 显式 > 隐式 > 默认
- `bind` 返回新函数，`call`/`apply` 立即执行
- 箭头函数的 `this` 在定义时确定，无法被 `call`/`bind` 改变
- 隐式丢失：`const fn = obj.fn; fn()` → `this` 丢失

## Why — 为什么

**适用场景：**

- 对象方法中访问自身属性
- 回调函数中保持外层 `this`
- 类方法中的 `this` 绑定
- 事件处理函数

**对比替代方案：**

| 维度 | this | 闭包变量 | 模块变量 |
|------|------|---------|---------|
| 灵活性 | 高（运行时决定） | 低（定义时确定） | 低（模块作用域） |
| 可读性 | 中（需理解规则） | 高 | 高 |
| 适用场景 | 面向对象 | 函数式 | 模块级状态 |

**优缺点：**

- ✅ 优点：
  - 运行时绑定，面向对象天然支持
  - 同一函数可用于不同对象
- ❌ 缺点：
  - 规则复杂，容易出错
  - 隐式丢失是高频 bug 源
  - 箭头函数与普通函数 `this` 行为不一致

## How — 怎么用

### 快速上手

```javascript
const obj = {
    name: 'Alice',
    greet() {
        console.log(`Hello, ${this.name}`);
    },
};

obj.greet(); // "Hello, Alice" — 隐式绑定

const fn = obj.greet;
fn(); // "Hello, undefined" — 隐式丢失，默认绑定
```

### 代码示例

**隐式丢失与修复：**

```javascript
class Timer {
    constructor() {
        this.seconds = 0;
    }

    // ❌ 回调中 this 丢失
    startWrong() {
        setInterval(function () {
            this.seconds++; // this === window
        }, 1000);
    }

    // ✅ 方案1：箭头函数
    start() {
        setInterval(() => {
            this.seconds++; // 继承 Timer 的 this
        }, 1000);
    }

    // ✅ 方案2：bind
    startBind() {
        setInterval(function () {
            this.seconds++;
        }.bind(this), 1000);
    }
}
```

**显式绑定对比：**

```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const alice = { name: 'Alice' };

greet.call(alice, 'Hello', '!');   // "Hello, Alice!"
greet.apply(alice, ['Hi', '?']);   // "Hi, Alice?"

const boundGreet = greet.bind(alice);
boundGreet('Hey', '~');            // "Hey, Alice~"
```

**new 绑定：**

```javascript
function Person(name) {
    this.name = name;
    // new 调用时：
    // 1. 创建空对象 {}
    // 2. this 指向该对象
    // 3. 执行函数体
    // 4. 返回 this（除非函数返回对象）
}

const alice = new Person('Alice');
alice.name; // "Alice"
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 回调中 this 丢失 | 函数作为参数传递后，隐式绑定丢失 | 用箭头函数或 `bind(this)` |
| 箭头函数不能用 new | 箭头函数没有 `[[Construct]]` | 需要构造函数时用普通函数 |
| React 事件 this 为 undefined | class 方法默认不会绑定 this | 箭头函数类属性或构造函数中 bind |
| 嵌套对象 this 指向 | `a.b.fn()` this 指向 `b` 不是 `a` | 只看最后一层调用对象 |

### 最佳实践

- 回调函数优先用箭头函数
- 类方法用箭头函数属性（`handleClick = () => {}`）或构造函数 bind
- 需要动态 this 的用普通函数
- 理解四条规则优先级：new > 显式 > 隐式 > 默认

## 面试题

**Q1: `this` 的四种绑定规则是什么？优先级如何？**
> 四种绑定规则：1) 默认绑定：独立调用 `fn()`，非严格模式指向 `window`，严格模式为 `undefined`；2) 隐式绑定：`obj.fn()` 指向 `obj`；3) 显式绑定：`fn.call/apply/bind(obj)` 指向 `obj`；4) `new` 绑定：指向新创建的实例。优先级：new > 显式 > 隐式 > 默认。

**Q2: 箭头函数的 `this` 和普通函数有什么不同？**
> 箭头函数没有自己的 `this`，它在定义时继承外层词法作用域的 `this`，且无法被 `call`/`apply`/`bind` 改变。普通函数的 `this` 在运行时根据调用方式确定。因此回调函数中优先用箭头函数保持外层 `this`，需要动态 `this` 时用普通函数。

**Q3: `call`、`apply`、`bind` 有什么区别？**
> `call` 和 `apply` 立即执行函数，`call` 逐个传参（`fn.call(obj, a, b)`），`apply` 传数组参数（`fn.apply(obj, [a, b])`）。`bind` 返回新函数不立即执行，且绑定是永久的（即使 `new` 调用除外）。`bind` 支持偏函数（预设部分参数），`call`/`apply` 不支持。

---

**相关链接：**
- [[作用域与闭包]]
- [[原型与继承]]
- [[ES6+核心特性]]

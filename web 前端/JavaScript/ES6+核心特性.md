---
tags:
  - Web前端
  - JavaScript
  - ES6
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# ES6+核心特性

## What — 是什么

> ES6（ECMAScript 2015）及后续版本引入了现代 JavaScript 的核心特性，包括解构、箭头函数、模块、类、迭代器等，彻底改变了 JS 的编程范式。

**核心概念：**

- **let/const**：块级作用域声明，`const` 不可重赋值（但对象属性可改）
- **箭头函数**：`=>` 简写，没有自己的 `this`/`arguments`
- **解构赋值**：`const { a, b } = obj` / `const [x, y] = arr`
- **模板字符串**：`` `Hello ${name}` ``
- **展开/剩余**：`...arr` 展开，`...args` 收集
- **可选链/空值合并**：`obj?.a?.b` / `value ?? default`
- **ES Module**：`import/export` 静态模块

**关键特性：**

- ES6+ 是年度发布，每年新增少量特性
- 现代 JS 开发的基础，所有框架和工具链都基于 ES6+
- 需 Babel/TypeScript 处理旧浏览器兼容

## Why — 为什么

**适用场景：**

- 所有现代 JavaScript 开发

**对比替代方案：**

| 维度 | ES6+ | TypeScript | CoffeeScript |
|------|------|-----------|-------------|
| 类型安全 | 无 | 有 | 无 |
| 学习成本 | 基础 | 中 | 中（已淘汰） |
| 生态 | 所有 JS 生态 | JS 超集 | 已废弃 |
| 运行时 | 原生支持 | 编译到 JS | 编译到 JS |

**优缺点：**

- ✅ 优点：
  - 代码简洁（解构、箭头函数、展开）
  - 模块化原生支持
  - 每年稳步进化
- ❌ 缺点：
  - 部分新特性需编译才能在旧浏览器运行
  - 特性多，需持续学习

## How — 怎么用

### 快速上手

```javascript
// 解构 + 默认值 + 重命名
const { name, age = 18, city: location = 'Beijing' } = user;

// 数组解构 + 跳过
const [, second, , fourth] = arr;

// 展开运算符
const merged = { ...defaults, ...userConfig };
const combined = [...arr1, ...arr2];

// 可选链 + 空值合并
const street = user?.address?.street ?? '未知';
```

### 代码示例

**解构高级用法：**

```javascript
// 嵌套解构
const { data: { list, total } } = response;

// 函数参数解构
function render({ title, items = [], theme = 'light' }) {
    // ...
}

// 交换变量
let a = 1, b = 2;
[a, b] = [b, a];
```

**Map/Set：**

```javascript
// Set：去重
const unique = [...new Set([1, 2, 2, 3])]; // [1, 2, 3]

// Map：任意类型做键
const cache = new Map();
cache.set({ id: 1 }, 'result');
cache.has(key); // true

// 遍历
for (const [key, value] of cache) {
    console.log(key, value);
}
```

**迭代器与生成器：**

```javascript
// 可迭代协议：[Symbol.iterator]()
const range = {
    from: 1,
    to: 5,
    [Symbol.iterator]() {
        let current = this.from;
        const last = this.to;
        return {
            next() {
                return current <= last
                    ? { value: current++, done: false }
                    : { done: true };
            },
        };
    },
};

for (const num of range) console.log(num); // 1 2 3 4 5

// 生成器
function* idGenerator() {
    let id = 1;
    while (true) yield id++;
}
const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| const 对象被修改 | `const` 只保护引用，不保护内容 | 用 `Object.freeze()` 或 `immutable` 库 |
| 解构重命名混淆 | `const { a: b } = obj` 中 a 是属性名 b 是变量名 | 记住：冒号左边是"从哪来"，右边是"叫什么" |
| `??` 和 `||` 混用 | `\|\|` 对 `0`/`""` 也返回默认值 | 有值可能是 falsy 时用 `??`，否则用 `||` |
| 动态 import | 静态 `import` 必须在顶层 | 用 `import()` 动态导入，返回 Promise |

### 最佳实践

- 优先 `const`，需要重赋值时才用 `let`，不用 `var`
- 解构设置默认值防止 undefined
- 用 `??` 替代 `||` 做空值判断
- 动态导入实现代码分割

## 面试题

**Q1: `let`、`const`、`var` 有什么区别？**
> `var` 是函数作用域，有变量提升（hoisting），可重复声明；`let`/`const` 是块级作用域，有暂时性死区（TDZ），不可重复声明。`const` 声明后不可重赋值，但对象/数组的属性仍可修改（`const` 保护的是引用，不是值）。优先用 `const`，需重赋值时用 `let`，不用 `var`。

**Q2: 解构赋值有哪些常见用法？**
> 对象解构：`const { name, age = 18 } = user`（支持默认值和重命名 `name: userName`）。数组解构：`const [first, , third] = arr`（支持跳过）。函数参数解构：`function render({ title, items = [] })`。交换变量：`[a, b] = [b, a]`。嵌套解构：`const { data: { list } } = response`。

**Q3: 箭头函数和普通函数有什么区别？**
> 箭头函数没有自己的 `this`（继承外层词法作用域的 `this`）、没有 `arguments` 对象、不能用作构造函数（不能 `new`）、没有 `prototype` 属性。因此回调函数优先用箭头函数保持 `this`，需要动态 `this` 或用作构造函数时用普通函数。

**Q4: Symbol 有什么用途？**
> Symbol 是 ES6 新增的原始类型，每次调用 `Symbol()` 返回唯一值。主要用途：1) 定义对象的唯一属性键，避免属性名冲突；2) 内置 Symbol 值如 `Symbol.iterator`（定义迭代器）、`Symbol.toPrimitive`（自定义类型转换）、`Symbol.hasInstance`（自定义 `instanceof` 行为）。

---

**相关链接：**
- [[Promise与异步]]
- [[模块化演进]]
- [[作用域与闭包]]

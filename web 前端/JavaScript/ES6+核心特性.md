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

### ES 版本特性对照表

| 版本 | 年份 | 核心特性 |
|------|------|---------|
| ES2015 | ES6 | let/const、箭头函数、解构、模板字符串、Symbol、迭代器/生成器、Promise、class、Module、Map/Set/WeakMap/WeakSet、Proxy/Reflect |
| ES2016 | ES7 | `**` 指数运算符、`Array.prototype.includes` |
| ES2017 | ES8 | async/await、`Object.entries/values`、字符串 padding、`Object.getOwnPropertyDescriptors` |
| ES2018 | ES9 | 异步迭代器 `for await...of`、`Promise.finally`、对象剩余/展开属性、正则命名捕获组 |
| ES2019 | ES10 | `Array.flat/flatMap`、`Object.fromEntries`、`String.trimStart/trimEnd`、可选 catch 绑定 |
| ES2020 | ES11 | 可选链 `?.`、空值合并 `??`、`BigInt`、`globalThis`、`Promise.allSettled`、`import()` |
| ES2021 | ES12 | 逻辑赋值 `??=/||=/&&=`、`String.replaceAll`、`Promise.any`、WeakRef/FinalizationRegistry、数字分隔符 |
| ES2022 | ES13 | 顶层 await、类字段（`#private`）、`at()`、`Object.hasOwn`、Error cause |
| ES2023 | ES14 | `Array.findLast/findLastIndex`、`toSorted/toReversed/toSpliced/with`（不可变数组方法）、Hashbang 语法 |
| ES2024 | ES15 | `Grouping/Map.groupBy`、`Promise.withResolvers`、`String.isWellFormed/toWellFormed`、`Atomics.waitAsync` |

### 核心特性分类

**变量声明：**
- **let/const**：块级作用域声明，`const` 不可重赋值（但对象属性可改）
- 暂时性死区（TDZ）：从块开始到声明语句之间不可访问

**函数增强：**
- **箭头函数**：`=>` 简写，没有自己的 `this`/`arguments`
- 默认参数：`function foo(x = 10) {}`
- 剩余参数：`function foo(...args) {}`

**解构与展开：**
- **解构赋值**：`const { a, b } = obj` / `const [x, y] = arr`
- **展开/剩余**：`...arr` 展开，`...args` 收集

**字符串增强：**
- **模板字符串**：`` `Hello ${name}` ``
- 标签模板：`tagFunc`Hello ${name}``
- 新增方法：`includes/startsWith/endsWith/repeat/padStart/padEnd/trimStart/trimEnd/replaceAll`

**对象增强：**
- 属性简写：`{ name }` 等价 `{ name: name }`
- 计算属性：`{ [key]: value }`
- 方法简写：`{ foo() {} }`
- 新增方法：`Object.entries/keys/values/fromEntries/assign/is/getOwnPropertyDescriptors/hasOwn`

**数组增强：**
- 新增方法：`find/findIndex/flat/flatMap/includes/at/fill/copyWithin/findLast/findLastIndex`
- 不可变方法（ES2023）：`toSorted/toReversed/toSpliced/with`

**新数据结构：**
- **Map/Set**：键值对集合 / 唯一值集合
- **WeakMap/WeakSet**：弱引用版本，键必须是对象，不阻止 GC

**异步编程：**
- **Promise**：异步操作的标准容器
- **async/await**：Promise 的语法糖
- **异步迭代器**：`for await...of`

**元编程：**
- **Symbol**：唯一标识符，`Symbol.iterator`/`Symbol.toPrimitive` 等
- **Proxy/Reflect**：拦截和自定义对象操作

**其他重要特性：**
- **可选链/空值合并**：`obj?.a?.b` / `value ?? default`
- **ES Module**：`import/export` 静态模块
- **BigInt**：任意精度整数
- **类字段**：`#private` 私有成员
- **顶层 await**：模块顶层直接使用 await

## Why — 为什么

**适用场景：**

- 所有现代 JavaScript 开发

**ES6+ 解决的 ES5 痛点：**

| ES5 痛点              | ES6+ 解决方案             |
| ------------------- | --------------------- |
| var 变量提升和函数作用域      | let/const 块级作用域 + TDZ |
| 回调地狱                | Promise → async/await |
| 无模块系统               | ES Module             |
| 手动遍历                | 迭代器/生成器 + for...of    |
| 无唯一标识符              | Symbol                |
| 对象键只能是字符串           | Map 任意类型键             |
| 字符串拼接不直观            | 模板字符串                 |
| 无私有属性               | #private 类字段          |
| 手动处理 null/undefined | 可选链 + 空值合并            |

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
  - 异步编程体验大幅改善
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

**1. 解构赋值完整用法：**

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

// 解构 + 剩余
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first=1, second=2, rest=[3,4,5]

// 对象解构 + 剩余
const { name, ...others } = user;
// others 包含除 name 外的所有属性

// 解构赋值给已有变量（需括号）
let x, y;
({ x, y } = { x: 1, y: 2 }); // 必须加括号，否则 {x,y} 被视为块

// 计算属性名解构
const prop = 'name';
const { [prop]: userName } = user;
```

**2. 展开运算符和剩余参数：**

```javascript
// 数组展开
const merged = [...arr1, ...arr2, 99];
const clone = [...original]; // 浅拷贝

// 对象展开（后者覆盖前者）
const config = { ...defaults, ...userConfig, extra: true };

// 函数剩余参数
function sum(...nums) {
    return nums.reduce((a, b) => a + b, 0);
}

// 构造函数展开
new Date(...[2024, 0, 1]); // 等价 new Date(2024, 0, 1)

// 展开字符串
const chars = [...'hello']; // ['h','e','l','l','o']

// Set 转数组
const unique = [...new Set([1, 2, 2, 3])]; // [1, 2, 3]

// Map 转数组
const entries = [...map.entries()]; // [[key1,val1], [key2,val2]]
```

**3. Map/Set/WeakMap/WeakSet 实战：**

```javascript
// Set：去重、集合运算
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
const union = new Set([...a, ...b]);          // 并集 {1,2,3,4}
const intersect = new Set([...a].filter(x => b.has(x))); // 交集 {2,3}
const diff = new Set([...a].filter(x => !b.has(x)));     // 差集 {1}

// Map：缓存、配置
const cache = new Map();
cache.set('key', { data: 'value' });
cache.set({ id: 1 }, 'object-key'); // 对象做键
console.log(cache.get('key'));        // { data: 'value' }
console.log(cache.size);              // 2

// Map 遍历
for (const [key, value] of cache) { }
cache.forEach((value, key) => { });

// Map ↔ Object 互转
const obj = Object.fromEntries(cache);    // Map → Object
const map = new Map(Object.entries(obj));  // Object → Map

// WeakMap：私有数据、DOM 关联
const privateData = new WeakMap();
class MyClass {
    constructor() {
        privateData.set(this, { secret: 42 });
    }
    getSecret() {
        return privateData.get(this).secret;
    }
}

// WeakMap 缓存：对象被回收时缓存自动清理
const computedCache = new WeakMap();
function expensiveCompute(obj) {
    if (computedCache.has(obj)) return computedCache.get(obj);
    const result = /* ... */;
    computedCache.set(obj, result);
    return result;
}
```

**4. 可选链和空值合并完整用法：**

```javascript
// 可选链 ?.
const street = user?.address?.street;        // 多层属性
const name = user?.getName?.();              // 可选方法调用
const first = arr?.[0];                      // 可选数组索引
const value = obj?.[prop];                   // 可选计算属性

// 空值合并 ??
const port = config.port ?? 3000;    // 只在 null/undefined 时用默认值
const name = user.name ?? '匿名';

// 配合使用
const city = user?.address?.city ?? '未知';

// ?? vs || 关键区别
const count = 0 ?? 10;    // 0（0 不是 null/undefined）
const count2 = 0 || 10;   // 10（0 是 falsy）
const text = '' ?? '默认'; // ''（空字符串不是 null/undefined）
const text2 = '' || '默认'; // '默认'

// 逻辑赋值运算符（ES2021）
config.port ??= 3000;  // 等价 config.port ?? (config.port = 3000)
config.name ||= '默认'; // 等价 config.name || (config.name = '默认')
config.debug &&= false; // 等价 config.debug && (config.debug = false)
```

**5. Symbol 的用途：**

```javascript
// 唯一属性键，避免冲突
const id = Symbol('id');
const user = { name: 'Alice', [id]: 123 };
console.log(user[id]); // 123
// Symbol 属性不会被 for...in / Object.keys 遍历
// 需用 Object.getOwnPropertySymbols() 获取

// 内置 Symbol
class Range {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
    // 自定义迭代行为
    [Symbol.iterator]() {
        let current = this.from;
        return {
            next: () => current <= this.to
                ? { value: current++, done: false }
                : { done: true }
        };
    }
    // 自定义类型转换
    [Symbol.toPrimitive](hint) {
        if (hint === 'number') return this.to - this.from;
        if (hint === 'string') return `${this.from}..${this.to}`;
        return true;
    }
    // 自定义 instanceof
    static [Symbol.hasInstance](instance) {
        return instance.from !== undefined && instance.to !== undefined;
    }
}

// Symbol.for / Symbol.keyFor：全局注册
const s1 = Symbol.for('app.id');  // 全局注册
const s2 = Symbol.for('app.id');  // 获取同一个
s1 === s2;  // true
Symbol.keyFor(s1); // 'app.id'
```

**6. 模板字面量高级用法：**

```javascript
// 标签模板：自定义模板处理
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => {
        const val = values[i] ? `<mark>${values[i]}</mark>` : '';
        return result + str + val;
    }, '');
}
const keyword = 'JavaScript';
const text = highlight`Learn ${keyword} today`;
// "Learn <mark>JavaScript</mark> today"

// 常见标签模板应用
// 1. HTML 转义
function safeHtml(strings, ...values) {
    const escape = (s) => String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
    return strings.reduce((r, s, i) => r + s + (i < values.length ? escape(values[i]) : ''), '');
}

// 2. 国际化
function i18n(strings, ...values) {
    return strings.reduce((r, s, i) => r + s + (values[i] ?? ''), '')
        .replace('Hello', '你好');
}

// 嵌套模板
const items = list.map(item => `
    <li>${item.name} - ${item.price}</li>
`).join('');

// 原始字符串（处理转义）
const path = String.raw`C:\Users\name\docs`; // 反斜杠不转义
```

**7. 对象和数组新增方法：**

```javascript
// Object 新增方法
const user = { name: 'Alice', age: 25, city: 'Beijing' };

Object.keys(user);      // ['name', 'age', 'city']
Object.values(user);    // ['Alice', 25, 'Beijing']
Object.entries(user);   // [['name','Alice'], ['age',25], ['city','Beijing']]
Object.fromEntries([['name','Bob'], ['age',30]]); // { name: 'Bob', age: 30 }

Object.hasOwn(user, 'name');  // true（替代 obj.hasOwnProperty()）
Object.is(NaN, NaN);          // true（=== 对 NaN 返回 false）
Object.is(-0, 0);             // false（=== 返回 true）

const merged = Object.assign({}, defaults, config); // 浅合并
const frozen = Object.freeze({ x: 1 }); // 冻结，不可修改

// Array 新增方法
const arr = [1, 2, 3, 4, 5];

arr.find(x => x > 3);          // 4（第一个满足的元素）
arr.findIndex(x => x > 3);     // 3（第一个满足的索引）
arr.findLast(x => x > 3);      // 5（最后一个满足的元素，ES2023）
arr.findLastIndex(x => x > 3); // 4
arr.includes(3);                // true
arr.at(-1);                     // 5（支持负索引）

// flat / flatMap
[1, [2, 3], [4, [5]]].flat();       // [1,2,3,4,[5]]
[1, [2, 3], [4, [5]]].flat(2);      // [1,2,3,4,5]
[{items: [1,2]}, {items: [3]}].flatMap(x => x.items); // [1,2,3]

// 不可变方法（ES2023，不修改原数组）
const sorted = arr.toSorted((a, b) => b - a); // [5,4,3,2,1]
const reversed = arr.toReversed();              // [5,4,3,2,1]
const spliced = arr.toSpliced(1, 2, 99);        // [1,99,5]
const replaced = arr.with(2, 99);                // [1,2,99,4,5]
```

**8. 迭代器与生成器：**

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

// 生成器实现懒加载
function* paginate(apiCall) {
    let page = 1;
    while (true) {
        const data = yield apiCall(page++);
        if (data.done) return;
    }
}

// yield* 委托生成器
function* concat(...iterables) {
    for (const iter of iterables) {
        yield* iter;
    }
}

// 异步生成器
async function* fetchPages(url) {
    let page = 1;
    while (true) {
        const res = await fetch(`${url}?page=${page}`);
        const data = await res.json();
        if (data.length === 0) return;
        yield data;
        page++;
    }
}

// for await...of 消费异步迭代器
for await (const page of fetchPages('/api/items')) {
    console.log(page);
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| const 对象被修改 | `const` 只保护引用，不保护内容 | 用 `Object.freeze()` 或 `immutable` 库 |
| 解构重命名混淆 | `const { a: b } = obj` 中 a 是属性名 b 是变量名 | 记住：冒号左边是"从哪来"，右边是"叫什么" |
| `??` 和 `||` 混用 | `||` 对 `0`/`""` 也返回默认值 | 有值可能是 falsy 时用 `??`，否则用 `||` |
| 动态 import | 静态 `import` 必须在顶层 | 用 `import()` 动态导入，返回 Promise |
| Map 的键是对象 | 相同结构的对象是不同引用 | 用字符串/数字做键，或确保用同一引用 |
| WeakMap 不能遍历 | 弱引用设计，可能随时被 GC | WeakMap 只适合关联数据，不适合集合 |
| 解构 null 报错 | `const { a } = null` 抛出 TypeError | 加默认值 `const { a } = obj ?? {}` |
| 展开运算符浅拷贝 | 嵌套对象仍是引用 | 深拷贝用 `structuredClone()` 或 JSON 序列化 |

### 最佳实践

- 优先 `const`，需要重赋值时才用 `let`，不用 `var`
- 解构设置默认值防止 undefined
- 用 `??` 替代 `||` 做空值判断
- 动态导入实现代码分割
- 用 `Object.freeze()` 保护常量对象
- Map 适合需要非字符串键或频繁增删的场景，普通对象适合固定结构
- `structuredClone()` 做深拷贝，取代 JSON.parse(JSON.stringify())
- 用 `Array.from()` + 映射函数替代 `map` 后再 `filter`

## 面试题

**Q1: `let`、`const`、`var` 有什么区别？**
> `var` 是函数作用域，有变量提升（hoisting），可重复声明；`let`/`const` 是块级作用域，有暂时性死区（TDZ），不可重复声明。`const` 声明后不可重赋值，但对象/数组的属性仍可修改（`const` 保护的是引用，不是值）。优先用 `const`，需重赋值时用 `let`，不用 `var`。

**Q2: 解构赋值有哪些常见用法？**
> 对象解构：`const { name, age = 18 } = user`（支持默认值和重命名 `name: userName`）。数组解构：`const [first, , third] = arr`（支持跳过）。函数参数解构：`function render({ title, items = [] })`。交换变量：`[a, b] = [b, a]`。嵌套解构：`const { data: { list } } = response`。剩余解构：`const { name, ...rest } = user`。

**Q3: 箭头函数和普通函数有什么区别？**
> 箭头函数没有自己的 `this`（继承外层词法作用域的 `this`）、没有 `arguments` 对象、不能用作构造函数（不能 `new`）、没有 `prototype` 属性。因此回调函数优先用箭头函数保持 `this`，需要动态 `this` 或用作构造函数时用普通函数。

**Q4: Symbol 有什么用途？**
> Symbol 是 ES6 新增的原始类型，每次调用 `Symbol()` 返回唯一值。主要用途：1) 定义对象的唯一属性键，避免属性名冲突；2) 内置 Symbol 值如 `Symbol.iterator`（定义迭代器）、`Symbol.toPrimitive`（自定义类型转换）、`Symbol.hasInstance`（自定义 `instanceof` 行为）。Symbol 属性不会被 `for...in`/`Object.keys` 遍历，需用 `Object.getOwnPropertySymbols()` 获取。

**Q5: const 声明的对象能修改属性吗？为什么？**
> 能。`const` 保证的是变量绑定（引用）不可变，不是值不可变。对象属性修改不改变引用地址，所以不违反 `const` 约束。如需冻结对象内容，使用 `Object.freeze(obj)`（浅冻结），深冻结需递归。ES2022 的 `#private` 类字段可提供更强的封装。

**Q6: Map 和 Object 有什么区别？什么时候用 Map？**
> 核心区别：1) Map 的键可以是任意类型（对象、函数等），Object 的键只能是字符串/Symbol；2) Map 有 size 属性，可直接获取大小；3) Map 的遍历顺序就是插入顺序；4) Map 频繁增删键值对性能更好。使用 Map 的场景：需要非字符串键、需要频繁增删、需要知道集合大小、用作缓存/映射。使用 Object 的场景：JSON 序列化、固定结构数据、需要原型链方法。

**Q7: 可选链?.和空值合并??的区别和配合使用？**
> 可选链 `?.` 用于安全访问可能为 null/undefined 的属性/方法，短路返回 undefined 而非报错。空值合并 `??` 用于在值为 null/undefined 时提供默认值。配合使用：`user?.address?.city ?? '未知'`，先安全访问，再提供兜底。注意 `??` 只判断 null/undefined，而 `||` 对所有 falsy 值（0、''、false）都生效。`??` 和 `?.` 不能直接混用（需括号），如 `(a ?? b)?.c`。

**Q8: WeakMap 和 Map 有什么区别？有什么用？**
> WeakMap 与 Map 的关键区别：1) 键必须是对象（不可用原始值）；2) 键是弱引用，对象无其他引用时可被 GC 回收，条目自动消失；3) 不可遍历（无 size/keys/forEach）；4) 不阻止垃圾回收。典型用途：1) 为对象关联私有数据（类外部无法访问）；2) DOM 元素关联数据（元素移除后自动清理）；3) 缓存计算结果（对象回收时缓存自动清理）；4) 闭包替代方案（比闭包更不容易内存泄漏）。

---

**相关链接：**
- [[Promise与异步]]
- [[模块化演进]]
- [[作用域与闭包]]
- [[Proxy与Reflect]]
- [[迭代器与生成器]]
- [[TypeScript核心]]

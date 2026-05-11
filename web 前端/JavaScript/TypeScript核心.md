---
tags:
  - Web前端
  - JavaScript
  - TypeScript
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# TypeScript核心

## What — 是什么

> TypeScript 是 JavaScript 的超集，添加了静态类型系统，在编译期捕获错误，提升代码可维护性和开发体验。

**核心概念：**

- **基础类型**：`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`
- **对象类型**：`interface`、`type`、`typeof`、`keyof`
- **高级类型**：泛型（`<T>`）、联合/交叉（`|`/`&`）、条件类型（`extends ? :`）、映射类型
- **工具类型**：`Partial`、`Required`、`Pick`、`Omit`、`Record`、`Exclude`、`Extract`、`ReturnType`
- **类型守卫**：`typeof`、`instanceof`、`in`、自定义守卫（`is`）
- **类型推断**：大部分场景无需手写类型，编译器自动推断

**关键特性：**

- TS 编译为 JS，运行时类型信息被擦除
- `strict: true` 是推荐的起点
- `.d.ts` 文件提供类型声明，`@types/xxx` 提供社区类型

## Why — 为什么

**适用场景：**

- 所有中大型前端项目
- 需要长期维护的代码库
- 团队协作（类型即文档）
- 库/SDK 开发

**对比替代方案：**

| 维度 | TypeScript | JSDoc | Flow |
|------|-----------|-------|------|
| 类型安全 | 编译期检查 | 编辑器提示 | 编译期检查 |
| 生态 | 极好（主流框架支持） | 依赖编辑器 | 较小 |
| 学习成本 | 中等 | 低 | 中等 |
| 社区 | 主流选择 | 辅助方案 | 几乎弃用 |

**优缺点：**

- ✅ 优点：
  - 编译期捕获大量错误
  - IDE 自动补全和重构体验极佳
  - 类型即文档，接口契约清晰
- ❌ 缺点：
  - 学习曲线（泛型、高级类型）
  - 编译增加构建时间
  - 类型体操过度会降低可读性

## How — 怎么用

### 快速上手

```typescript
// 基础类型
const name: string = 'Alice';
const age: number = 25;
const items: string[] = ['a', 'b'];
const tuple: [string, number] = ['id', 1];

// interface vs type
interface User {
    id: number;
    name: string;
    email?: string; // 可选
    readonly createdAt: Date; // 只读
}

type Status = 'active' | 'inactive'; // 联合类型字面量

// 函数类型
function fetchUser(id: number): Promise<User> {
    return fetch(`/api/users/${id}`).then(r => r.json());
}
```

### 代码示例

**泛型：**

```typescript
// 泛型函数
function first<T>(arr: T[]): T | undefined {
    return arr[0];
}

// 泛型约束
function getLength<T extends { length: number }>(item: T): number {
    return item.length;
}

// 泛型与映射
function createMap<K extends string, V>(entries: [K, V][]): Record<K, V> {
    return Object.fromEntries(entries) as Record<K, V>;
}
```

**高级类型：**

```typescript
// 条件类型
type IsString<T> = T extends string ? 'yes' : 'no';
type A = IsString<string>; // 'yes'
type B = IsString<number>; // 'no'

// infer 提取类型
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;
type Result = UnpackPromise<Promise<string>>; // string

// 工具类型组合
type UserUpdate = Partial<Pick<User, 'name' | 'email'>>;
// { name?: string; email?: string }

// discriminated union
type ApiResponse =
    | { status: 'success'; data: User }
    | { status: 'error'; message: string };

function handleResponse(res: ApiResponse) {
    if (res.status === 'success') {
        console.log(res.data); // TypeScript 知道 data 存在
    } else {
        console.log(res.message); // TypeScript 知道 message 存在
    }
}
```

**类型守卫：**

```typescript
// typeof 守卫
function pad(value: string | number): string {
    if (typeof value === 'string') return value.padStart(10);
    return value.toFixed(0).padStart(10);
}

// 自定义守卫
function isUser(obj: unknown): obj is User {
    return typeof obj === 'object' && obj !== null && 'id' in obj;
}

// 使用
const data: unknown = JSON.parse(raw);
if (isUser(data)) {
    console.log(data.name); // 安全访问
}
```

**声明文件：**

```typescript
// env.d.ts — 补充全局类型
declare namespace NodeJS {
    interface ProcessEnv {
        NODE_ENV: 'development' | 'production';
        API_URL: string;
    }
}

// 模块声明
declare module 'some-untyped-lib' {
    export function init(config: { key: string }): void;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `any` 滥用 | 偷懒跳过类型检查 | 用 `unknown` 替代，逐步收窄 |
| 类型断言不安全 | `as` 强制转换绕过检查 | 优先用类型守卫，少用 `as` |
| 第三方库无类型 | 社区未提供 `@types/xxx` | 自己写 `declare module` |
| 泛型推断不出 | 场景过于复杂 | 显式标注泛型参数 |
| 枚举的坑 | `const enum` 和 `enum` 行为不同 | 优先用联合字面量替代 enum |

### 最佳实践

- 开启 `strict: true`，不使用隐式 `any`
- 优先 `interface`（可声明合并），复杂类型用 `type`
- 用 `unknown` 替代 `any`，强制类型收窄
- 避免过度类型体操，可读性优先
- 通用工具类型集中管理（如 `src/types/utils.ts`）

## 面试题

**Q1: `type` 和 `interface` 有什么区别？**
> 两者大部分场景可互换，关键区别：1) `interface` 支持声明合并（同名自动合并），`type` 不行；2) `type` 可表示联合类型、交叉类型、元组等复杂类型，`interface` 不行；3) `interface` 只能描述对象形状，`type` 可以是任意类型别名。建议：对象类型优先 `interface`（可扩展），复杂类型用 `type`。

**Q2: 什么是泛型约束？如何使用？**
> 泛型约束通过 `extends` 限制泛型参数必须满足的条件。例如 `function getLength<T extends { length: number }>(item: T)` 确保 `T` 一定有 `length` 属性。还可以用 `keyof` 约束：`function getProperty<T, K extends keyof T>(obj: T, key: K)`，确保 `key` 是 `T` 的合法属性。

**Q3: `never` 类型有什么用途？**
> `never` 表示永远不会有值的类型，用途：1) 函数永远抛出异常或无限循环时返回 `never`；2) 联合类型收窄到不可能的分支（穷尽检查）；3) 类型运算的底类型（`never` 是所有类型的子类型）。利用 `never` 可实现穷尽检查：`default` 分支将变量赋值给 `never`，遗漏分支时编译报错。

**Q4: TypeScript 的类型收窄（Narrowing）有哪些方式？**
> 类型收窄将宽类型缩小到窄类型：1) `typeof` 守卫（`typeof x === 'string'`）；2) `instanceof` 守卫；3) `in` 操作符（`'name' in obj`）；4) 自定义守卫（`function isUser(obj): obj is User`）；5) 可辨识联合（discriminated union，根据 `status` 字段收窄）；6) 赋值收窄（`let x = 'hello'` 推断为字面量类型）。

---

**相关链接：**
- [[ES6+核心特性]]
- [[作用域与闭包]]
- [[React核心]]

---
tags:
  - Web前端
  - 鸿蒙
  - ArkTS
  - TypeScript
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# ArkTS语言核心

## What — 是什么

> ArkTS 是华为基于 TypeScript 扩展的编程语言，是鸿蒙（HarmonyOS）应用开发的官方语言，通过静态类型增强、AOT 编译和限制动态特性，实现了比标准 TypeScript 更高的运行时性能和更严格的类型安全。

**核心概念：**

- **ArkTS 语言**：基于 TypeScript 的高性能静态类型语言，鸿蒙应用层唯一官方开发语言
- **方舟编译器（Ark Compiler）**：华为自研的 AOT 编译器，将 ArkTS 编译为高效本地代码
- **静态类型增强**：比标准 TS 更严格的类型约束，禁止 `any`、限制动态特性
- **声明式 UI 装饰器**：`@Component`、`@Entry`、`@State` 等装饰器驱动 ArkUI 声明式开发
- **空安全机制**：可选链 `?.`、非空断言 `!`、空值合并 `??` 的系统性空值防护

**ArkTS 在鸿蒙架构中的位置：**

```
┌─────────────────────────────────────────────────────────────────┐
│                      HarmonyOS 应用架构                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 应用层 (Application)                      │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │   │
│  │  │   ArkTS 代码  │  │  ArkUI 声明式 │  │  业务逻辑     │  │   │
│  │  │  (开发语言)    │  │  (UI 框架)    │  │  (数据/网络)  │  │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │   │
│  └─────────┼────────────────┼────────────────┼────────────┘   │
│            │                │                │                 │
│  ┌─────────▼────────────────▼────────────────▼────────────┐   │
│  │             方舟编译器 (Ark Compiler / AOT)              │   │
│  │      静态类型分析 → 优化 → 生成高效本地机器码              │   │
│  └──────────────────────────┬────────────────────────────┘   │
│                             │                                 │
│  ┌──────────────────────────▼────────────────────────────┐   │
│  │               ArkTS 运行时 (Ark Runtime)                │   │
│  │     内存管理 / GC / TaskPool / Worker / 类型系统支持     │   │
│  └──────────────────────────┬────────────────────────────┘   │
│                             │                                 │
│  ┌──────────────────────────▼────────────────────────────┐   │
│  │              HarmonyOS 系统服务层                        │   │
│  │     Ability / 分布式 / 图形 / 网络 / 文件 / 安全         │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**ArkTS 语法体系总览：**

```
┌───────────────────────────────────────────────────────┐
│                   ArkTS 语言体系                        │
├─────────────┬─────────────┬──────────────┬────────────┤
│  类型系统    │  面向对象    │  函数式特性    │  并发模型   │
├─────────────┼─────────────┼──────────────┼────────────┤
│ 基础类型     │ class       │ 箭头函数      │ Promise    │
│ 联合/交叉    │ abstract    │ 函数重载      │ async/await│
│ 字面量类型   │ interface   │ 闭包         │ TaskPool   │
│ 泛型/约束    │ extends     │ 高阶函数      │ Worker     │
│ 枚举         │ implements  │ 剩余参数      │            │
│ 类型守卫     │ 装饰器      │ 可选/默认参数  │            │
│ 空安全       │ 模块系统     │              │            │
└─────────────┴─────────────┴──────────────┴────────────┘
```

## Why — 为什么

**适用场景：**

- 所有鸿蒙应用的开发（ArkTS 是鸿蒙应用层唯一官方开发语言）
- 需要跨设备（手机/平板/手表/车机/PC）运行的鸿蒙应用
- 对性能有较高要求的移动端应用（AOT 编译提供接近原生的性能）
- 需要声明式 UI 开发范式的场景（ArkUI 框架基于 ArkTS）

**ArkTS 与 TypeScript 的关系和差异：**

| 维度 | TypeScript | ArkTS |
|------|-----------|-------|
| 定位 | JavaScript 的超集，通用脚本语言 | 鸿蒙官方开发语言，面向应用开发 |
| 类型严格度 | 可选严格模式，允许 `any` | 强制严格模式，禁止 `any` |
| 动态特性 | 允许 `eval`/`with`/动态属性 | 限制或禁止动态特性 |
| 编译方式 | 转译为 JavaScript（JIT 为主） | AOT 编译为本地机器码 |
| 运行时 | V8 / Node.js 等 JS 引擎 | 方舟运行时（Ark Runtime） |
| 装饰器 | 实验性支持 | 核心特性，UI 框架深度依赖 |
| 空安全 | 可选 `strictNullChecks` | 语言层面强制空安全 |
| 模块系统 | ES Modules + CommonJS | ES Modules（推荐静态导入） |
| 性能 | 依赖 JIT 优化 | AOT 编译，性能接近原生 |
| 生态 | npm 海量包 | 鸿蒙 SDK + ohpm 仓库 |

**ArkTS 对 TypeScript 动态特性的限制：**

| 限制项 | TypeScript 行为 | ArkTS 限制 | 原因 |
|--------|----------------|-----------|------|
| `any` 类型 | 允许使用 | 禁止使用 | `any` 绕过类型检查，AOT 无法优化 |
| `eval()` | 允许执行动态代码 | 禁止使用 | 动态代码无法静态分析 |
| `with` 语句 | 允许使用 | 禁止使用 | 作用域不明确，无法静态分析 |
| 动态属性访问 | `obj[expr]` 任意键 | 限制动态属性 | AOT 需要确定对象形状 |
| 原型链修改 | `Object.prototype.xxx` | 禁止修改内置原型 | 破坏类型稳定性 |
| `delete` 操作符 | 可删除任意属性 | 限制使用 | 改变对象形状影响性能 |
| `as any` 断言 | 允许任意断言 | 禁止 `as any` | 同 `any` 禁用 |
| 索引签名 | `[key: string]: any` | 受限使用 | 运行时类型不确定 |

**优缺点：**

- ✅ 优点：
  - AOT 编译提供接近原生的运行时性能
  - 严格的类型系统减少运行时错误
  - 声明式装饰器让 UI 开发更简洁高效
  - 与鸿蒙系统深度集成，API 访问无障碍
  - TypeScript 开发者上手成本低
- ❌ 缺点：
  - 动态特性受限，部分 TS 写法需改造
  - 生态尚在发展，第三方库不如 npm 丰富
  - 强制类型注解增加编码量
  - 仅限鸿蒙平台，无跨平台能力

**与同类语言的对比：**

| 维度 | ArkTS | Dart (Flutter) | Swift (iOS) | Kotlin (Android) |
|------|-------|---------------|-------------|------------------|
| 基础语言 | TypeScript | 自研 | 自研 | 基于 JVM |
| 类型系统 | 静态强类型+泛型 | 静态强类型+泛型 | 静态强类型+泛型 | 静态强类型+泛型 |
| 编译方式 | AOT | AOT/JIT | AOT | AOT/Kotlin Native |
| 空安全 | 强制 | 强制(Sound Null Safety) | 强制(Optional) | 强制 |
| UI 范式 | 声明式(ArkUI) | 声明式(Widget) | 声明式(SwiftUI) | 声明式(Compose) |
| 跨平台 | 鸿蒙多设备 | 6平台 | Apple 生态 | Android/JVM |
| 动态特性 | 严格限制 | 严格限制 | 无动态特性 | 有限反射 |
| 学习曲线(前端) | 低(TS相似) | 中(需学新语法) | 高(全新语法) | 中(类TS语法) |

## How — 怎么用

### ArkTS 与 TypeScript 的差异详解

#### 1. 禁止 any 类型

```typescript
// ❌ TypeScript 写法 — ArkTS 中不允许
let data: any = fetchData();
let result = (data as any).value;

// ✅ ArkTS 正确写法 — 使用具体类型
interface ApiResponse {
  code: number;
  value: string;
  message: string;
}

let data: ApiResponse = fetchData();
let result: string = data.value;

// ✅ 如果类型真的未知，使用 unknown 并做类型收窄
let data: unknown = parseInput(rawStr);
if (typeof data === 'object' && data !== null && 'value' in data) {
  let result: string = (data as ApiResponse).value;
}
```

#### 2. 禁止 eval 和 with

```typescript
// ❌ TypeScript 写法 — ArkTS 中不允许
eval("console.log('hello')");
with (obj) { console.log(name); } // 动态作用域

// ✅ ArkTS 正确写法 — 使用函数和明确的作用域
function logMessage(msg: string): void {
  console.log(msg);
}
logMessage('hello');

console.log(obj.name); // 明确的属性访问
```

#### 3. 强制类型注解

```typescript
// ❌ TypeScript 写法 — 隐式 any
function process(input) {  // 参数缺少类型注解
  return input.length;
}

// ✅ ArkTS 正确写法 — 显式类型注解
function process(input: string): number {
  return input.length;
}

// ❌ 隐式 any 的变量声明
let items = [];  // TypeScript 推断为 any[]

// ✅ ArkTS 必须提供类型
let items: string[] = [];
let count: number = 0;
```

#### 4. 限制动态属性访问

```typescript
// ❌ TypeScript 写法 — ArkTS 中受限
const key = Math.random() > 0.5 ? 'name' : 'age';
const value = obj[key]; // 动态键访问

// ✅ ArkTS 正确写法 — 使用类型守卫或映射类型
type UserKey = 'name' | 'age';
function getValue(obj: User, key: UserKey): string | number {
  if (key === 'name') {
    return obj.name;
  }
  return obj.age;
}

// ✅ 或使用 Record 类型
const config: Record<string, string> = {
  'host': 'localhost',
  'port': '8080'
};
const host: string = config['host']; // Record 允许索引访问
```

### 变量声明与类型系统

#### 基础类型

```typescript
// ===== 基础类型声明 =====
let username: string = 'Alice';
let age: number = 25;
let isActive: boolean = true;

// ArkTS 基础类型与 TypeScript 一致
let message: string = 'Hello HarmonyOS';
let count: number = 100;
let flag: boolean = false;

// ===== 数组类型 =====
let numbers: number[] = [1, 2, 3, 4, 5];
let names: Array<string> = ['Alice', 'Bob', 'Charlie']; // 泛型写法

// ===== 联合类型 =====
let id: string | number = 'A001';
id = 1001; // 也可以赋值为 number

type Status = 'pending' | 'active' | 'completed' | 'cancelled';
let orderStatus: Status = 'pending';

// ===== 交叉类型 =====
type Timestamped = {
  createdAt: string;
  updatedAt: string;
};

type Identifiable = {
  id: string;
};

type Entity = Identifiable & Timestamped;
// Entity 同时具有 id, createdAt, updatedAt

let user: Entity = {
  id: 'U001',
  createdAt: '2026-01-01',
  updatedAt: '2026-05-11'
};

// ===== 字面量类型 =====
type Direction = 'left' | 'right' | 'up' | 'down';
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;

// 字面量类型与联合类型结合 — 精确约束
type EventName = 'click' | 'hover' | 'focus' | 'blur';
function onEvent(name: EventName, handler: () => void): void {
  console.log(`Registered ${name} event`);
}

// ===== const 与 let =====
const API_BASE: string = 'https://api.harmonyos.com'; // 不可重新赋值
let currentPage: number = 1; // 可重新赋值

// const 推断为字面量类型（与 TS 一致）
const direction = 'left'; // 推断为 'left' 而非 string
```

#### 类型别名与内联类型

```typescript
// ===== type 别名 =====
type Point = {
  x: number;
  y: number;
};

type Callback = (data: string) => void;

type Nullable<T> = T | null; // 泛型类型别名

// ===== 实战：表单字段定义 =====
type FormField = {
  name: string;
  label: string;
  type: 'text' | 'number' | 'email' | 'password' | 'select';
  required: boolean;
  placeholder?: string;       // 可选属性
  readonly options?: string[]; // 仅当 type === 'select' 时需要
};

const loginFields: FormField[] = [
  { name: 'email', label: '邮箱', type: 'email', required: true },
  { name: 'password', label: '密码', type: 'password', required: true },
];
```

### 接口与类

#### 接口（interface）

```typescript
// ===== 基础接口 =====
interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;        // 可选属性
  readonly role: string;  // 只读属性
}

// 接口扩展
interface AdminUser extends User {
  permissions: string[];
  level: number;
}

// 多接口扩展
interface Loggable {
  log(): void;
}

interface Serializable {
  serialize(): string;
}

interface Storable extends Loggable, Serializable {
  storageKey: string;
}

// ===== 接口实现 =====
class ConsoleLogger implements Loggable {
  log(): void {
    console.log('ConsoleLogger active');
  }
}

// ===== 接口定义函数形状 =====
interface SearchFunc {
  (source: string, term: string): boolean;
}

const containsTerm: SearchFunc = (source, term) => {
  return source.includes(term);
};

// ===== 实战：API 响应接口 =====
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
  timestamp: number;
}

interface UserInfo {
  id: string;
  name: string;
  avatar: string;
  vipLevel: number;
}

// 使用泛型接口
function parseResponse(json: string): ApiResponse<UserInfo> {
  const parsed = JSON.parse(json) as ApiResponse<UserInfo>;
  return parsed;
}
```

#### 类（class）

```typescript
// ===== 基础类定义 =====
class Person {
  // ArkTS 中类的属性必须显式声明
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hello, I'm ${this.name}, ${this.age} years old.`;
  }
}

const alice = new Person('Alice', 25);
console.log(alice.greet());

// ===== 访问修饰符 =====
class BankAccount {
  private balance: number;     // 仅类内部访问
  protected owner: string;     // 类及子类访问
  public accountNo: string;    // 任何位置可访问
  readonly createdAt: string;  // 只读，初始化后不可修改

  constructor(owner: string, accountNo: string, balance: number) {
    this.owner = owner;
    this.accountNo = accountNo;
    this.balance = balance;
    this.createdAt = new Date().toISOString();
  }

  // getter / setter
  get currentBalance(): number {
    return this.balance;
  }

  set deposit(amount: number) {
    if (amount > 0) {
      this.balance += amount;
    }
  }

  // 私有方法
  private validateAmount(amount: number): boolean {
    return amount > 0 && this.balance >= amount;
  }

  withdraw(amount: number): boolean {
    if (this.validateAmount(amount)) {
      this.balance -= amount;
      return true;
    }
    return false;
  }
}

// ===== 继承 =====
class Animal {
  constructor(public name: string, public sound: string) {}

  speak(): string {
    return `${this.name} says ${this.sound}`;
  }
}

class Dog extends Animal {
  constructor(name: string) {
    super(name, 'Woof');
  }

  fetch(item: string): string {
    return `${this.name} fetches the ${item}`;
  }
}

class Cat extends Animal {
  constructor(name: string) {
    super(name, 'Meow');
  }

  purr(): string {
    return `${this.name} is purring`;
  }
}

// ===== 抽象类 =====
abstract class Shape {
  abstract area(): number;            // 抽象方法，子类必须实现
  abstract perimeter(): number;

  describe(): string {                // 具体方法，子类可继承
    return `Area: ${this.area()}, Perimeter: ${this.perimeter()}`;
  }
}

class Circle extends Shape {
  constructor(public radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius * this.radius;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  constructor(public width: number, public height: number) {
    super();
  }

  area(): number {
    return this.width * this.height;
  }

  perimeter(): number {
    return 2 * (this.width + this.height);
  }
}

// 多态
const shapes: Shape[] = [
  new Circle(5),
  new Rectangle(4, 6),
];
shapes.forEach(s => console.log(s.describe()));

// ===== implements 接口 =====
interface Printable {
  print(): void;
}

interface Comparable<T> {
  compareTo(other: T): number;
}

class Student implements Printable, Comparable<Student> {
  constructor(
    public id: string,
    public name: string,
    public score: number
  ) {}

  print(): void {
    console.log(`[${this.id}] ${this.name}: ${this.score}`);
  }

  compareTo(other: Student): number {
    return this.score - other.score; // 升序
  }
}
```

### 枚举与类型守卫

#### 枚举（enum）

```typescript
// ===== 数字枚举 =====
enum Direction {
  Up,      // 0
  Down,    // 1
  Left,    // 2
  Right    // 3
}

let dir: Direction = Direction.Up;
console.log(Direction[0]); // 'Up' — 反向映射

// ===== 字符串枚举（推荐） =====
enum HttpStatus {
  OK = '200',
  Created = '201',
  BadRequest = '400',
  Unauthorized = '401',
  NotFound = '404',
  InternalError = '500'
}

function handleStatus(status: HttpStatus): string {
  switch (status) {
    case HttpStatus.OK:
      return '请求成功';
    case HttpStatus.NotFound:
      return '资源未找到';
    case HttpStatus.InternalError:
      return '服务器错误';
    default:
      return '未知状态';
  }
}

// ===== const enum（更高效，编译时内联） =====
const enum Color {
  Red = '#FF0000',
  Green = '#00FF00',
  Blue = '#0000FF'
}

let bgColor: Color = Color.Red; // 编译后直接替换为 '#FF0000'

// ===== 实战：应用状态枚举 =====
enum AppState {
  Initializing = 'INITIALIZING',
  Loading = 'LOADING',
  Ready = 'READY',
  Error = 'ERROR',
  Background = 'BACKGROUND'
}

function onAppStateChanged(newState: AppState): void {
  switch (newState) {
    case AppState.Initializing:
      console.log('应用初始化中...');
      break;
    case AppState.Loading:
      console.log('加载数据...');
      break;
    case AppState.Ready:
      console.log('应用就绪');
      break;
    case AppState.Error:
      console.log('发生错误');
      break;
    case AppState.Background:
      console.log('应用进入后台');
      break;
  }
}

// ===== 枚举与联合字面量的选择 =====
// ArkTS 推荐场景：
// - 需要运行时值和反向映射 → enum
// - 仅需编译期类型约束 → 联合字面量（更轻量）
type Theme = 'light' | 'dark' | 'auto'; // 比 enum 更简洁
```

#### 类型守卫（Type Guard）

```typescript
// ===== typeof 守卫 =====
function formatValue(value: string | number): string {
  if (typeof value === 'string') {
    return value.trim(); // TypeScript 知道这里是 string
  }
  return value.toFixed(2); // TypeScript 知道这里是 number
}

// ===== instanceof 守卫 =====
class SuccessResult {
  constructor(public data: string) {}
}

class ErrorResult {
  constructor(public message: string) {}
}

type Result = SuccessResult | ErrorResult;

function handleResult(result: Result): string {
  if (result instanceof SuccessResult) {
    return `Success: ${result.data}`;
  }
  return `Error: ${result.message}`;
}

// ===== in 操作符守卫 =====
interface Car {
  wheels: number;
  drive(): void;
}

interface Boat {
  sail(): void;
}

function operate(vehicle: Car | Boat): void {
  if ('drive' in vehicle) {
    vehicle.drive();
  } else {
    vehicle.sail();
  }
}

// ===== 自定义类型守卫（is 谓词） =====
interface Dog {
  breed: string;
  bark(): void;
}

interface Cat {
  color: string;
  purr(): void;
}

function isDog(pet: Dog | Cat): pet is Dog {
  return (pet as Dog).bark !== undefined;
}

function interact(pet: Dog | Cat): void {
  if (isDog(pet)) {
    pet.bark();    // TypeScript 知道 pet 是 Dog
  } else {
    pet.purr();    // TypeScript 知道 pet 是 Cat
  }
}

// ===== 可辨识联合（Discriminated Union） =====
interface CircleShape {
  kind: 'circle';   // 可辨识属性
  radius: number;
}

interface RectShape {
  kind: 'rectangle';
  width: number;
  height: number;
}

interface TriangleShape {
  kind: 'triangle';
  base: number;
  height: number;
}

type Shape2D = CircleShape | RectShape | TriangleShape;

function calculateArea(shape: Shape2D): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;  // shape 自动收窄为 CircleShape
    case 'rectangle':
      return shape.width * shape.height;    // shape 自动收窄为 RectShape
    case 'triangle':
      return 0.5 * shape.base * shape.height;
  }
}

// ===== 穷尽检查 =====
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

function describeShape(shape: Shape2D): string {
  switch (shape.kind) {
    case 'circle':
      return `Circle with radius ${shape.radius}`;
    case 'rectangle':
      return `Rectangle ${shape.width}x${shape.height}`;
    case 'triangle':
      return `Triangle base=${shape.base} height=${shape.height}`;
    default:
      return assertNever(shape); // 如果遗漏 case，编译报错
  }
}
```

### 泛型与泛型约束

#### 泛型基础

```typescript
// ===== 泛型函数 =====
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);       // 显式指定类型参数
const str = identity('hello');          // 类型推断为 string

// ===== 泛型接口 =====
interface Repository<T> {
  findById(id: string): T | null;
  findAll(): T[];
  save(entity: T): T;
  delete(id: string): boolean;
}

// ===== 泛型类 =====
class DataStore<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  get(index: number): T | undefined {
    return this.items[index];
  }

  getAll(): T[] {
    return [...this.items]; // 返回副本
  }

  find(predicate: (item: T) => boolean): T | undefined {
    return this.items.find(predicate);
  }

  count(): number {
    return this.items.length;
  }
}

// 使用泛型类
const userStore = new DataStore<User>();
userStore.add({ id: '1', name: 'Alice', email: 'alice@test.com', role: 'admin' });

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);
```

#### 泛型约束

```typescript
// ===== extends 约束 =====
interface HasLength {
  length: number;
}

// 约束 T 必须有 length 属性
function getLength<T extends HasLength>(item: T): number {
  return item.length;
}

getLength('hello');     // string 有 length ✓
getLength([1, 2, 3]);  // Array 有 length ✓
// getLength(123);      // number 无 length ✗

// ===== keyof 约束 =====
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: 'Alice', age: 25, city: 'Shanghai' };
const name = getProperty(person, 'name');   // string
const age = getProperty(person, 'age');     // number
// getProperty(person, 'email');            // 编译错误，'email' 不是 person 的属性

// ===== 多泛型约束 =====
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: number;
}

// T 必须同时满足 Identifiable 和 Timestamped
function mergeAndSort<T extends Identifiable & Timestamped>(items: T[]): T[] {
  return items.sort((a, b) => a.createdAt - b.createdAt);
}

// ===== new() 约束 — 工厂模式 =====
function createInstance<T>(ctor: new () => T): T {
  return new ctor();
}

class Config {
  theme: string = 'light';
  lang: string = 'zh-CN';
}

const config = createInstance(Config); // config: Config

// ===== 实战：通用 API 请求函数 =====
interface ApiError {
  code: number;
  message: string;
}

async function request<T>(
  url: string,
  options?: {
    method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
    body?: Record<string, Object>;
    headers?: Record<string, string>;
  }
): Promise<T> {
  const method = options?.method ?? 'GET';
  // 实际项目中使用 @ohos.net.http 发起请求
  console.log(`${method} ${url}`);
  return {} as T; // 简化示意
}

// 使用
interface ProductList {
  items: Product[];
  total: number;
}

interface Product {
  id: string;
  name: string;
  price: number;
}

const products = await request<ProductList>('/api/products');
```

#### 泛型工具类型

```typescript
// ===== 内置工具类型（与 TypeScript 一致） =====

// Partial — 所有属性变为可选
type PartialUser = Partial<User>;

// Required — 所有属性变为必选
type RequiredConfig = Required<{
  host?: string;
  port?: number;
}>;

// Pick — 选取部分属性
type UserBasic = Pick<User, 'id' | 'name'>;

// Omit — 排除部分属性
type UserWithoutRole = Omit<User, 'role'>;

// Record — 键值映射
type PageMap = Record<string, { title: string; path: string }>;

// Exclude / Extract — 联合类型过滤
type StatusValues = 'active' | 'inactive' | 'pending' | 'deleted';
type ActiveStatus = Extract<StatusValues, 'active' | 'pending'>;
type NonDeleted = Exclude<StatusValues, 'deleted'>;

// ReturnType — 提取函数返回类型
function createUser() {
  return { id: '1', name: 'Alice' };
}
type CreatedUser = ReturnType<typeof createUser>; // { id: string; name: string }

// ===== 自定义工具类型 =====
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

type Nullable<T> = T | null;

type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// 实战：表单数据类型（所有字段变为可选且可空）
type FormData<T> = DeepPartial<Nullable<T>>;
```

### 函数

#### 函数声明与参数

```typescript
// ===== 函数声明方式 =====

// 函数声明
function add(a: number, b: number): number {
  return a + b;
}

// 函数表达式
const multiply = function(a: number, b: number): number {
  return a * b;
};

// 箭头函数（ArkUI 中最常用）
const divide = (a: number, b: number): number => a / b;

// ===== 可选参数 =====
function buildGreeting(name: string, title?: string): string {
  if (title) {
    return `Hello, ${title} ${name}`;
  }
  return `Hello, ${name}`;
}

buildGreeting('Alice');              // 'Hello, Alice'
buildGreeting('Alice', 'Dr.');       // 'Hello, Dr. Alice'

// ===== 默认参数 =====
function createUser(
  name: string,
  role: string = 'user',
  active: boolean = true
): User {
  return { id: Date.now().toString(), name, email: '', role };
}

createUser('Alice');                  // role='user', active=true
createUser('Bob', 'admin');           // role='admin', active=true
createUser('Carol', 'editor', false); // 全部指定

// ===== 剩余参数 =====
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3);       // 6
sum(10, 20, 30, 40); // 100

function formatLog(level: string, ...messages: string[]): string {
  const timestamp = new Date().toISOString();
  return `[${timestamp}] [${level}] ${messages.join(' ')}`;
}

formatLog('INFO', 'Server', 'started', 'on', 'port', '8080');

// ===== 参数解构 =====
interface SearchOptions {
  query: string;
  page?: number;
  pageSize?: number;
  sortBy?: string;
}

function search({ query, page = 1, pageSize = 20, sortBy = 'relevance' }: SearchOptions): string[] {
  console.log(`Searching "${query}", page ${page}, size ${pageSize}, sort by ${sortBy}`);
  return [];
}
```

#### 函数重载

```typescript
// ===== 函数重载 =====
// ArkTS 支持函数重载，先声明重载签名，再实现

function parseInput(input: string): number;
function parseInput(input: number): string;
function parseInput(input: string | number): string | number {
  if (typeof input === 'string') {
    return parseInt(input, 10);
  }
  return input.toString();
}

const numResult: number = parseInput('42');
const strResult: string = parseInput(42);

// ===== 实战：重载实现不同参数组合 =====
function createElement(tag: string): Element;
function createElement(tag: string, attrs: Record<string, string>): Element;
function createElement(tag: string, attrs: Record<string, string>, children: Element[]): Element;
function createElement(
  tag: string,
  attrs?: Record<string, string>,
  children?: Element[]
): Element {
  const el: Element = { tag, attrs: attrs ?? {}, children: children ?? [] };
  return el;
}

// ===== 重载与联合类型的取舍 =====
// 如果只是参数类型不同，联合类型更简洁
function process(input: string | number): string {
  if (typeof input === 'string') return input.toUpperCase();
  return input.toFixed(2);
}

// 如果返回值类型随参数变化，使用重载
function getItem(index: number): string;
function getItem(key: string): string | undefined;
function getItem(keyOrIndex: string | number): string | undefined {
  // 实现...
  return undefined;
}
```

#### 高阶函数与闭包

```typescript
// ===== 高阶函数 =====
type Transformer<T, U> = (item: T) => U;

function mapArray<T, U>(arr: T[], transform: Transformer<T, U>): U[] {
  const result: U[] = [];
  for (const item of arr) {
    result.push(transform(item));
  }
  return result;
}

const lengths = mapArray(['hello', 'world'], s => s.length); // [5, 5]

// ===== 闭包 =====
function createCounter(initial: number = 0): {
  increment: () => number;
  decrement: () => number;
  value: () => number;
} {
  let count = initial;
  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
}

const counter = createCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.decrement(); // 11
counter.value();     // 11

// ===== 函数柯里化 =====
function curry<T, U, V>(fn: (a: T, b: U) => V): (a: T) => (b: U) => V {
  return (a: T) => (b: U) => fn(a, b);
}

const add = (a: number, b: number) => a + b;
const add5 = curry(add)(5);
console.log(add5(3));  // 8
console.log(add5(10)); // 15
```

### 模块系统

#### 静态导入导出

```typescript
// ===== logger.ets — 工具模块 =====

// 命名导出
export enum LogLevel {
  Debug = 'DEBUG',
  Info = 'INFO',
  Warn = 'WARN',
  Error = 'ERROR'
}

export function log(level: LogLevel, message: string): void {
  console.log(`[${level}] ${message}`);
}

export function debug(message: string): void {
  log(LogLevel.Debug, message);
}

export function info(message: string): void {
  log(LogLevel.Info, message);
}

// 默认导出
export default class Logger {
  private prefix: string;

  constructor(prefix: string) {
    this.prefix = prefix;
  }

  log(message: string): void {
    console.log(`[${this.prefix}] ${message}`);
  }
}

// ===== 命名空间内导出（重新导出） =====
export { LogLevel as LevelType } from './logger';
```

```typescript
// ===== app.ets — 使用模块 =====

// 导入默认导出
import Logger from './logger';

// 导入命名导出
import { LogLevel, log, info, debug } from './logger';

// 重命名导入
import { LogLevel as Level } from './logger';

// 全部导入
import * as LoggerModule from './logger';

// 使用
const logger = new Logger('MyApp');
logger.log('Application started');

log(LogLevel.Info, 'System initialized');
debug('Debug mode active');

const currentLevel: Level = Level.Info;
```

#### 动态导入

```typescript
// ===== 动态导入 — 按需加载模块 =====

async function loadHeavyModule(): Promise<void> {
  // 动态 import 返回 Promise
  const heavyModule = await import('./heavy-module');
  heavyModule.doWork();
}

// ===== 实战：路由级别的动态导入 =====
interface RouteConfig {
  path: string;
  load: () => Promise<{ default: object }>;
}

const routes: RouteConfig[] = [
  { path: '/home', load: () => import('./pages/HomePage') },
  { path: '/profile', load: () => import('./pages/ProfilePage') },
  { path: '/settings', load: () => import('./pages/SettingsPage') },
];

async function navigate(path: string): Promise<void> {
  const route = routes.find(r => r.path === path);
  if (route) {
    const page = await route.load();
    console.log(`Loaded page for ${path}`);
  }
}
```

### 装饰器（Decorators）

装饰器是 ArkTS/ArkUI 的核心机制，用于声明式 UI 开发。

#### 装饰器体系总览

```
┌─────────────────────────────────────────────────────────────────┐
│                    ArkUI 装饰器体系                               │
├───────────────┬─────────────────────────────────────────────────┤
│  类别          │  装饰器                                         │
├───────────────┼─────────────────────────────────────────────────┤
│  组件定义      │  @Entry, @Component, @Preview                   │
├───────────────┼─────────────────────────────────────────────────┤
│  状态管理      │  @State, @Prop, @Link, @Provide, @Consume      │
│               │  @StorageLink, @StorageProp                     │
├───────────────┼─────────────────────────────────────────────────┤
│  观察与联动    │  @Watch, @Observed, @ObjectLink                 │
├───────────────┼─────────────────────────────────────────────────┤
│  UI 构建       │  @Builder, @Extend, @Styles                    │
├───────────────┼─────────────────────────────────────────────────┤
│  动画          │  @AnimatableExtend                             │
└───────────────┴─────────────────────────────────────────────────┘
```

#### @Entry 与 @Component

```typescript
// @Entry — 标记页面入口组件，一个页面只有一个 @Entry
// @Component — 标记自定义组件，所有 UI 组件必须加此装饰器

@Entry
@Component
struct IndexPage {
  // 组件状态
  @State message: string = 'Hello HarmonyOS';
  @State count: number = 0;

  // build 方法 — 必须实现，描述 UI 结构
  build() {
    Column() {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      Text(`Count: ${this.count}`)
        .fontSize(18)
        .margin({ top: 10 })

      Button('Increment')
        .onClick(() => {
          this.count++;
        })
        .margin({ top: 10 })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

#### 状态管理装饰器

```typescript
// ===== @State — 组件内部状态，变化触发 UI 刷新 =====
@Component
struct StateDemo {
  @State count: number = 0;
  @State user: User = { id: '1', name: 'Alice', email: 'a@b.com', role: 'admin' };
  @State items: string[] = ['Apple', 'Banana'];

  build() {
    Column() {
      Text(`Count: ${this.count}`)
      Button('+1')
        .onClick(() => this.count++) // @State 变化 → UI 自动更新

      Text(`User: ${this.user.name}`)
      Button('Change Name')
        .onClick(() => {
          this.user.name = 'Bob'; // @State 嵌套属性变化也会触发更新
        })
    }
  }
}

// ===== @Prop — 父组件单向传递，子组件只读 =====
@Component
struct ChildComponent {
  @Prop title: string = '';     // 单向数据流，子组件不能修改
  @Prop count: number = 0;

  build() {
    Column() {
      Text(this.title)
      Text(`Count: ${this.count}`)
    }
  }
}

// ===== @Link — 父子组件双向绑定 =====
@Component
struct CounterComponent {
  @Link value: number;  // 双向绑定，修改会同步回父组件

  build() {
    Row() {
      Button('-').onClick(() => this.value--)
      Text(`${this.value}`).margin({ left: 10, right: 10 })
      Button('+').onClick(() => this.value++)
    }
  }
}

@Entry
@Component
struct ParentComponent {
  @State score: number = 0;

  build() {
    Column() {
      Text(`Total Score: ${this.score}`)
      // @Link 绑定：用 $ 语法传递引用
      CounterComponent({ value: $score })
    }
  }
}

// ===== @Provide / @Consume — 跨层级数据传递 =====
@Entry
@Component
struct GrandParentComponent {
  @Provide theme: string = 'dark';   // 提供数据
  @Provide user: User = { id: '1', name: 'Alice', email: 'a@b.com', role: 'admin' };

  build() {
    Column() {
      Text(`GrandParent theme: ${this.theme}`)
      ParentComponent()
    }
  }
}

@Component
struct ParentComponent {
  // 中间层不需要传递 theme
  build() {
    Column() {
      ChildComponent()
    }
  }
}

@Component
struct ChildComponent {
  @Consume theme: string;   // 消费数据，自动从祖先组件获取
  @Consume user: User;

  build() {
    Column() {
      Text(`Child theme: ${this.theme}`)
      Text(`Child user: ${this.user.name}`)
    }
  }
}

// ===== @Watch — 监听状态变化 =====
@Component
struct WatchDemo {
  @State @Watch('onPriceChange') price: number = 100;
  @State @Watch('onNameChange') name: string = 'Product';

  // 回调方法名，在状态变化时自动触发
  onPriceChange(newValue: number, oldValue: number): void {
    console.log(`Price changed: ${oldValue} → ${newValue}`);
  }

  onNameChange(newValue: string, oldValue: string): void {
    console.log(`Name changed: ${oldValue} → ${newValue}`);
  }

  build() {
    Column() {
      Text(`${this.name}: ¥${this.price}`)
      Button('Change Price')
        .onClick(() => this.price = 200)
    }
  }
}
```

#### UI 构建装饰器

```typescript
// ===== @Builder — 轻量 UI 复用单元 =====
@Component
struct BuilderDemo {
  @State items: string[] = ['Item 1', 'Item 2', 'Item 3'];

  // @Builder 定义可复用的 UI 片段
  @Builder itemCard(title: string, index: number) {
    Row() {
      Text(`${index + 1}. ${title}`)
        .fontSize(16)
        .layoutWeight(1)
      Button('Delete')
        .onClick(() => this.items.splice(index, 1))
    }
    .width('100%')
    .padding(12)
    .backgroundColor('#f5f5f5')
    .borderRadius(8)
    .margin({ bottom: 8 })
  }

  build() {
    Column() {
      ForEach(this.items, (item: string, index: number) => {
        this.itemCard(item, index)  // 调用 @Builder
      })
    }
    .padding(16)
  }
}

// ===== @Extend — 扩展内置组件样式 =====
// @Extend 只能定义在组件外部，全局可用
@Extend(Text)
function highlightText(color: ResourceColor = '#FF0000') {
  .fontColor(color)
  .fontWeight(FontWeight.Bold)
  .fontSize(18)
}

@Extend(Button)
function primaryButton() {
  .backgroundColor('#007DFF')
  .fontColor('#FFFFFF')
  .borderRadius(20)
  .width('80%')
  .height(44)
}

// 使用
@Component
struct ExtendDemo {
  build() {
    Column() {
      Text('Warning').highlightText('#FF9800')   // 使用扩展样式
      Text('Error').highlightText('#F44336')      // 可传参数
      Button('Submit').primaryButton()             // 使用扩展样式
    }
  }
}

// ===== @Styles — 提取通用样式（比 @Extend 更轻量，不绑定特定组件） =====
@Styles function cardStyle() {
  .width('100%')
  .padding(16)
  .backgroundColor('#FFFFFF')
  .borderRadius(12)
  .shadow({ radius: 4, color: '#1a000000', offsetY: 2 })
}

@Styles function flexCenter() {
  .justifyContent(FlexAlign.Center)
  .alignItems(HorizontalAlign.Center)
}

@Component
struct StylesDemo {
  build() {
    Column() {
      // 多个组件复用相同样式
      Column() {
        Text('Card 1')
      }
      .cardStyle()
      .margin({ bottom: 12 })

      Column() {
        Text('Card 2')
      }
      .cardStyle()
    }
    .flexCenter()
    .width('100%')
    .height('100%')
  }
}
```

### 空安全

#### 可选链、非空断言与空值合并

```typescript
// ===== 可选链 ?. — 安全访问可能为 null/undefined 的属性 =====
interface UserProfile {
  name: string;
  address?: {
    city?: string;
    street?: string;
    zipCode?: string;
  };
  company?: {
    name: string;
    department?: {
      manager?: {
        email?: string;
      };
    };
  };
}

const user: UserProfile = { name: 'Alice' };

// 不用可选链 — 需要逐层检查
let city1: string | undefined;
if (user.address !== undefined && user.address !== null) {
  city1 = user.address.city;
}

// 使用可选链 — 一行搞定
const city: string | undefined = user.address?.city;
const managerEmail: string | undefined = user.company?.department?.manager?.email;

// 可选链用于方法调用
interface Service {
  getData?(): string[];
}

const service: Service = {};
const data = service.getData?.() ?? []; // 安全调用 + 空值合并

// 可选链用于数组访问
const arr: (number | undefined)[] = [1, undefined, 3];
const second: number | undefined = arr[1]?.toFixed(2) as number | undefined;

// ===== 非空断言 ! — 告诉编译器"我确定这里不为空" =====
function processValue(value: string | null): string {
  // 如果你能确定 value 不为 null，使用 !
  return value!.toUpperCase(); // 编译通过，但运行时如果 value 为 null 会报错

  // 更安全的做法 — 用类型守卫
  // if (value !== null) return value.toUpperCase();
  // return '';
}

// 实战场景：DOM 元素获取
// const element = document.getElementById('app')!; // 开发者确定元素存在

// ===== 空值合并 ?? — 提供默认值 =====
const port: number = parseInt('') || 3000;   // 3000，|| 对 0 也视为 falsy
const port2: number = parseInt('') ?? 3000;  // 3000，?? 只对 null/undefined 生效
const zero: number = 0 ?? 42;                // 0，0 不是 null/undefined

// 三者组合使用
function getConfigValue(
  config: Record<string, string | undefined>,
  key: string
): string {
  return config[key]?.trim() ?? 'default';
}
```

#### 空安全最佳实践

```typescript
// ===== 类型收窄 — 比 ! 更安全 =====
function processUser(user: User | null): string {
  // ❌ 不推荐 — 非空断言可能崩溃
  // return user!.name;

  // ✅ 推荐 — 类型守卫
  if (user === null) {
    return 'Unknown User';
  }
  return user.name; // TypeScript 自动收窄为 User
}

// ===== 可选参数与空值 =====
interface SearchParams {
  keyword?: string;
  category?: string;
  page?: number;
}

function search(params: SearchParams): string[] {
  const keyword = params.keyword ?? '';           // 空值合并提供默认值
  const category = params.category ?? 'all';
  const page = params.page ?? 1;
  console.log(`Searching "${keyword}" in ${category}, page ${page}`);
  return [];
}

// ===== 嵌套空安全处理 =====
interface AppConfig {
  api?: {
    baseURL?: string;
    timeout?: number;
    headers?: Record<string, string>;
  };
}

function getBaseURL(config: AppConfig): string {
  return config.api?.baseURL ?? 'https://default.api.com';
}

function getTimeout(config: AppConfig): number {
  return config.api?.timeout ?? 5000;
}

function getHeaders(config: AppConfig): Record<string, string> {
  return config.api?.headers ?? { 'Content-Type': 'application/json' };
}
```

### 异步编程

#### Promise

```typescript
// ===== 创建 Promise =====
function fetchUserData(userId: string): Promise<User> {
  return new Promise<User>((resolve, reject) => {
    // 模拟异步操作（实际使用 @ohos.net.http）
    setTimeout(() => {
      if (userId.startsWith('U')) {
        resolve({
          id: userId,
          name: 'Alice',
          email: 'alice@harmonyos.com',
          role: 'admin'
        });
      } else {
        reject(new Error(`Invalid userId: ${userId}`));
      }
    }, 1000);
  });
}

// ===== Promise 链式调用 =====
fetchUserData('U001')
  .then(user => {
    console.log(`Got user: ${user.name}`);
    return fetchOrders(user.id);
  })
  .then(orders => {
    console.log(`Got ${orders.length} orders`);
  })
  .catch(error => {
    console.error(`Error: ${error.message}`);
  })
  .finally(() => {
    console.log('Request completed');
  });

// ===== Promise 组合 =====
// 并行执行
const [users, products, orders] = await Promise.all([
  request<User[]>('/api/users'),
  request<Product[]>('/api/products'),
  request<Order[]>('/api/orders'),
]);

// 竞速 — 取最快返回的
const fastest = await Promise.race([
  request<string>('/api/cdn1/data'),
  request<string>('/api/cdn2/data'),
]);

// 全部完成（无论成功失败）
const results = await Promise.allSettled([
  request<string>('/api/service-a'),
  request<string>('/api/service-b'),
  request<string>('/api/service-c'),
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Service ${index}: ${result.value}`);
  } else {
    console.error(`Service ${index} failed: ${result.reason}`);
  }
});
```

#### async / await

```typescript
// ===== 基础 async/await =====
async function loadUserProfile(userId: string): Promise<User> {
  try {
    const user = await fetchUserData(userId);
    const orders = await fetchOrders(userId);
    const settings = await fetchSettings(userId);
    return { ...user, orders, settings } as User;
  } catch (error) {
    console.error(`Failed to load profile: ${error}`);
    throw error;
  }
}

// ===== 并行 async/await =====
async function loadDashboard(): Promise<void> {
  // 不需要顺序依赖的请求应该并行
  const [users, stats, notifications] = await Promise.all([
    request<User[]>('/api/users'),
    request<DashboardStats>('/api/stats'),
    request<Notification[]>('/api/notifications'),
  ]);

  console.log(`Users: ${users.length}`);
  console.log(`Revenue: ${stats.revenue}`);
  console.log(`Unread: ${notifications.length}`);
}

// ===== async 错误处理 =====
async function safeRequest<T>(url: string): Promise<T | null> {
  try {
    const data = await request<T>(url);
    return data;
  } catch (error) {
    if (error instanceof Error) {
      console.error(`Request failed: ${error.message}`);
    }
    return null;
  }
}

// ===== 实战：带重试的异步请求 =====
async function requestWithRetry<T>(
  url: string,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error | null = null;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await request<T>(url);
    } catch (error) {
      lastError = error as Error;
      console.warn(`Attempt ${attempt} failed: ${lastError.message}`);
      if (attempt < maxRetries) {
        await new Promise<void>(resolve => setTimeout(resolve, delay * attempt));
      }
    }
  }

  throw lastError!;
}
```

#### TaskPool — 鸿蒙并发任务池

```typescript
// TaskPool 是鸿蒙提供的线程池并发 API，适合 CPU 密集型任务
// 将耗时任务放到子线程执行，避免阻塞 UI 主线程

import taskpool from '@ohos.taskpool';

// ===== 定义任务函数（必须是模块级函数，不能是闭包） =====
@Concurrent
function computeFibonacci(n: number): number {
  if (n <= 1) return n;
  let a = 0, b = 1;
  for (let i = 2; i <= n; i++) {
    const temp = a + b;
    a = b;
    b = temp;
  }
  return b;
}

@Concurrent
function processImage(data: number[]): number[] {
  // 模拟图像处理（灰度化）
  return data.map(pixel => {
    const gray = Math.round(pixel * 0.299 + pixel * 0.587 + pixel * 0.114);
    return gray;
  });
}

// ===== 使用 TaskPool 执行任务 =====
async function runTaskPoolDemo(): Promise<void> {
  try {
    // 执行单个任务
    const result = await taskpool.execute(computeFibonacci, 30);
    console.log(`Fibonacci(30) = ${result}`);

    // 执行多个并行任务
    const task1 = taskpool.execute(computeFibonacci, 20);
    const task2 = taskpool.execute(computeFibonacci, 30);
    const task3 = taskpool.execute(computeFibonacci, 40);

    const [r1, r2, r3] = await Promise.all([task1, task2, task3]);
    console.log(`Results: ${r1}, ${r2}, ${r3}`);
  } catch (error) {
    console.error(`TaskPool error: ${error}`);
  }
}
```

#### Worker — 鸿蒙多线程

```typescript
// Worker 适用于长时间运行的后台线程，如持续的数据处理

import worker from '@ohos.worker';

// ===== 主线程：创建 Worker =====
const workerInstance = new worker.ThreadWorker('workers/data-processor.js');

// 接收 Worker 返回的消息
workerInstance.onmessage = (event: MessageEvents) => {
  const result = event.data;
  console.log(`Worker result: ${JSON.stringify(result)}`);
};

// 接收 Worker 错误
workerInstance.onerror = (event: ErrorEvent) => {
  console.error(`Worker error: ${event.message}`);
};

// 向 Worker 发送消息
workerInstance.postMessage({
  type: 'PROCESS',
  data: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
});

// 终止 Worker
// workerInstance.terminate();

// ===== Worker 线程：workers/data-processor.js =====
// const workerPort = worker.workerPort;
//
// workerPort.onmessage = (event) => {
//   const { type, data } = event.data;
//   if (type === 'PROCESS') {
//     const result = data.reduce((sum, val) => sum + val * 2, 0);
//     workerPort.postMessage({ type: 'RESULT', result });
//   }
// };
```

**TaskPool vs Worker 选择指南：**

| 维度 | TaskPool | Worker |
|------|----------|--------|
| 线程管理 | 自动管理线程池 | 手动创建管理 |
| 生命周期 | 任务执行完自动回收 | 长期存在，需手动终止 |
| 通信方式 | 函数参数/返回值 | 消息机制 postMessage |
| 适用场景 | 短时 CPU 密集任务 | 长时后台任务 |
| 数量限制 | 系统调度 | 最多 8 个 Worker |
| 使用复杂度 | 低 | 中 |

### 集合类型

```typescript
// ===== Array — 有序可重复集合 =====
const fruits: string[] = ['Apple', 'Banana', 'Cherry'];
const numbers: number[] = [1, 2, 3, 4, 5];

// 常用方法
fruits.push('Date');              // 末尾添加
fruits.pop();                     // 末尾移除
fruits.unshift('Apricot');        // 开头添加
fruits.shift();                   // 开头移除

// 遍历
fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});

// 函数式操作
const upperFruits = fruits.map(f => f.toUpperCase());
const longFruits = fruits.filter(f => f.length > 5);
const hasCherry = fruits.some(f => f === 'Cherry');
const allLong = fruits.every(f => f.length > 3);
const found = fruits.find(f => f.startsWith('C'));
const totalLength = fruits.reduce((sum, f) => sum + f.length, 0);

// 排序
const sorted = [...fruits].sort((a, b) => a.localeCompare(b));
const nums = [3, 1, 4, 1, 5, 9, 2, 6];
const asc = [...nums].sort((a, b) => a - b);
const desc = [...nums].sort((a, b) => b - a);

// ===== Set — 无序不重复集合 =====
const uniqueIds = new Set<string>();
uniqueIds.add('id-1');
uniqueIds.add('id-2');
uniqueIds.add('id-1'); // 重复添加无效
console.log(uniqueIds.size); // 2

// 数组去重
const dupNumbers = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(dupNumbers)]; // [1, 2, 3, 4]

// Set 操作
const setA = new Set([1, 2, 3]);
const setB = new Set([2, 3, 4]);

// 交集
const intersection = new Set([...setA].filter(x => setB.has(x))); // {2, 3}

// 并集
const union = new Set([...setA, ...setB]); // {1, 2, 3, 4}

// 差集
const difference = new Set([...setA].filter(x => !setB.has(x))); // {1}

// ===== Map — 键值对集合 =====
const userCache = new Map<string, User>();

// 增删改查
userCache.set('U001', { id: 'U001', name: 'Alice', email: 'a@b.com', role: 'admin' });
userCache.set('U002', { id: 'U002', name: 'Bob', email: 'b@c.com', role: 'user' });

const user = userCache.get('U001');          // 获取
const exists = userCache.has('U001');        // 是否存在
userCache.delete('U002');                    // 删除
userCache.clear();                           // 清空

// 遍历
userCache.forEach((user, key) => {
  console.log(`${key}: ${user.name}`);
});

for (const [key, value] of userCache) {
  console.log(`${key} → ${value.name}`);
}

// Map 与 Object 的选择
// - 需要任意键类型、频繁增删 → Map
// - 需要 JSON 序列化、固定键集 → Object

// ===== Record — 键值映射类型（TypeScript 工具类型） =====
type RouteMap = Record<string, {
  title: string;
  icon: string;
  path: string;
}>;

const routes: RouteMap = {
  home: { title: '首页', icon: 'home', path: '/home' },
  profile: { title: '我的', icon: 'person', path: '/profile' },
  settings: { title: '设置', icon: 'settings', path: '/settings' },
};

// 实战：枚举到标签的映射
const StatusLabels: Record<HttpStatus, string> = {
  [HttpStatus.OK]: '成功',
  [HttpStatus.Created]: '已创建',
  [HttpStatus.BadRequest]: '请求错误',
  [HttpStatus.Unauthorized]: '未授权',
  [HttpStatus.NotFound]: '未找到',
  [HttpStatus.InternalError]: '服务器错误',
};
```

### ArkTS 运行时与性能优化

#### 方舟编译器 AOT

```
┌─────────────────────────────────────────────────────────────┐
│              ArkTS 编译执行流程                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ArkTS 源码 (.ets)                                          │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────┐                                           │
│  │  方舟编译器    │  — 静态类型分析                           │
│  │  (Ark Compiler│  — 类型推断与优化                         │
│  │   AOT)       │  — 内联、逃逸分析、死代码消除              │
│  └──────┬───────┘                                           │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────┐      ┌──────────────┐                    │
│  │  优化字节码    │  →   │  本地机器码    │                    │
│  │  (方舟字节码)  │      │  (AOT 生成)   │                    │
│  └──────┬───────┘      └──────┬───────┘                    │
│         │                      │                            │
│         ▼                      ▼                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Ark Runtime (方舟运行时)                   │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐  │  │
│  │  │ GC      │  │ 内存管理 │  │ 并发调度             │  │  │
│  │  │ (分代GC) │  │ (池化)  │  │ (TaskPool/Worker)   │  │  │
│  │  └─────────┘  └─────────┘  └─────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 性能优化要点

```typescript
// ===== 1. 避免动态特性 — 确保类型确定 =====

// ❌ 性能差 — 运行时类型不确定
// let data: any = getResponse();
// let value = data['key'];

// ✅ 性能好 — 编译期类型确定
interface ResponseData {
  key: string;
  value: number;
}
let data: ResponseData = getResponse();
let value: string = data.key;

// ===== 2. 减少对象创建 — 复用对象 =====
@Component
struct OptimizedList {
  @State items: ListItemData[] = [];

  // ❌ 每次 build 创建新对象
  // build() {
  //   List() {
  //     ForEach(this.items, (item) => {
  //       ListItem() {
  //         Row() { Text(item.title) }
  //         .width('100%')
  //         .height(60)
  //         .backgroundColor('#ffffff')
  //       }
  //     })
  //   }
  // }

  // ✅ 使用 @Builder 和 @Styles 复用
  @Builder listItemBuilder(item: ListItemData) {
    ListItem() {
      Row() {
        Text(item.title)
      }
      .width('100%')
      .height(60)
      .cardStyle()  // 复用 @Styles
    }
  }

  build() {
    List() {
      ForEach(this.items, (item: ListItemData) => {
        this.listItemBuilder(item)
      })
    }
  }
}

// ===== 3. 合理使用 @State — 避免不必要的 UI 刷新 =====
@Component
struct StateOptimization {
  // ❌ 整个对象变化会触发全部刷新
  // @State config: AppConfig = { ... };

  // ✅ 拆分细粒度状态
  @State theme: string = 'light';
  @State fontSize: number = 14;
  @State lang: string = 'zh-CN';

  build() {
    // 只有用到 theme 的部分会在 theme 变化时刷新
    Column() {
      Text('Content').fontColor(this.theme === 'dark' ? '#fff' : '#000')
    }
  }
}

// ===== 4. 使用 TaskPool 处理耗时操作 =====
// 参见上文 TaskPool 部分 — 避免阻塞 UI 线程

// ===== 5. 懒加载 — LazyForEach =====
@Component
struct LazyListDemo {
  // LazyForEach 配合 IDataSource 实现按需加载
  private data: MyDataSource = new MyDataSource();

  build() {
    List() {
      // LazyForEach：只渲染可见区域的列表项
      LazyForEach(this.data, (item: ListItemData) => {
        ListItem() {
          Text(item.title).height(60)
        }
      })
    }
    .cachedCount(5) // 缓存额外 5 项，减少滚动时的白块
  }
}
```

### 前端开发者学 ArkTS 的注意事项

#### 从 TypeScript 到 ArkTS 的迁移清单

```
┌─────────────────────────────────────────────────────────────────┐
│              TypeScript → ArkTS 迁移检查清单                      │
├──────┬──────────────────────────────────────────────────────────┤
│ 序号  │ 检查项                                                   │
├──────┼──────────────────────────────────────────────────────────┤
│  1   │ 消除所有 any 类型，替换为具体类型或 unknown                │
│  2   │ 移除所有 eval() 和 with 语句                              │
│  3   │ 为所有函数参数和返回值添加类型注解                          │
│  4   │ 移除 as any 断言，改用 as 具体类型                         │
│  5   │ 确保不修改内置原型（Object.prototype 等）                   │
│  6   │ 动态属性访问改为静态访问或 Record 类型                      │
│  7   │ delete 操作替换为赋值 undefined 或重新构造对象             │
│  8   │ 索引签名 [key: string]: any 改为 Record<string, Type>     │
│  9   │ 模块系统统一使用 ES Modules（import/export）               │
│ 10   │ 了解并使用 ArkUI 装饰器体系替代命令式 UI                   │
└──────┴──────────────────────────────────────────────────────────┘
```

#### 思维方式转变

| 前端思维 | ArkTS 思维 | 说明 |
|---------|-----------|------|
| 命令式 DOM 操作 | 声明式 UI + 状态驱动 | 不要手动操作 UI，修改 @State 即可 |
| npm 生态 | ohpm 生态 | 包管理器从 npm 切换到 ohpm |
| React/Vue 组件 | struct + @Component | 组件定义方式不同 |
| CSS 样式 | 链式调用属性方法 | `.fontSize(16).color('#333')` |
| Webpack/Vite | Hvigor | 构建工具切换 |
| 浏览器运行时 | Ark Runtime | 运行环境完全不同 |
| DOM 事件 | ArkUI 事件回调 | `.onClick(() => {})` |
| React Hooks | @State/@Prop/@Link | 状态管理机制不同 |
| Redux/Pinia | AppStorage/@Provide | 全局状态管理不同 |

```typescript
// ===== React → ArkTS 对照 =====

// React 写法
// function Counter() {
//   const [count, setCount] = useState(0);
//   return (
//     <View>
//       <Text>{count}</Text>
//       <Button onClick={() => setCount(c => c + 1)}>+1</Button>
//     </View>
//   );
// }

// ArkTS 写法
@Component
struct Counter {
  @State count: number = 0;  // 类似 useState

  build() {
    Column() {
      Text(`${this.count}`)
      Button('+1')
        .onClick(() => this.count++)  // 直接修改 @State
    }
  }
}

// ===== Vue → ArkTS 对照 =====

// Vue 写法
// <template>
//   <div>{{ message }}</div>
//   <button @click="reverse">Reverse</button>
// </template>
// <script setup>
// const message = ref('Hello');
// const reverse = () => message.value = message.value.split('').reverse().join('');
// </script>

// ArkTS 写法
@Component
struct MessageReverser {
  @State message: string = 'Hello';  // 类似 ref

  build() {
    Column() {
      Text(this.message)
      Button('Reverse')
        .onClick(() => {
          this.message = this.message.split('').reverse().join('');
        })
    }
  }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `any` 类型报错 | ArkTS 禁止 any | 使用具体类型或 `unknown` + 类型守卫 |
| `eval` 不可用 | ArkTS 禁止动态代码执行 | 改用函数封装、策略模式 |
| `@State` 对象属性不更新 | 嵌套对象修改未触发检测 | 使用 `@Observed` + `@ObjectLink` |
| `@Prop` 无法修改 | 单向数据流设计 | 需要双向绑定改用 `@Link` |
| 父子传值太深 | 层层 Props 传递繁琐 | 使用 `@Provide` / `@Consume` |
| UI 卡顿 | 主线程执行耗时操作 | 使用 TaskPool / Worker |
| 列表性能差 | ForEach 一次性渲染全部 | 改用 LazyForEach 按需加载 |
| 模块循环依赖 | 互相 import 导致初始化异常 | 抽取公共模块，单向依赖 |
| 函数参数缺类型注解 | ArkTS 要求显式注解 | 为所有参数添加类型 |
| `delete` 操作受限 | 删除属性影响对象形状 | 设为 `undefined` 或重建对象 |

### 最佳实践

- 严格遵循 ArkTS 类型约束，不使用 `any`、`eval`、`with` 等动态特性
- 状态管理选择合适的装饰器：内部状态用 `@State`，父子通信用 `@Prop`/`@Link`，跨层级用 `@Provide`/`@Consume`
- 耗时操作（网络请求、计算、I/O）使用 TaskPool 或 Worker，不阻塞 UI 线程
- 长列表使用 `LazyForEach` 代替 `ForEach`，配合 `cachedCount` 优化滚动体验
- 使用 `@Builder` 和 `@Styles` 提取复用 UI 片段，减少重复代码
- 泛型优先于 `any` 或类型断言，在编译期保证类型安全
- 空安全优先使用可选链 `?.` 和空值合并 `??`，减少非空断言 `!` 的使用
- 模块划分清晰，避免循环依赖，静态导入优先于动态导入
- 枚举使用字符串枚举增强可读性，简单场景可用联合字面量替代
- 合理使用 `readonly` 和接口约束，构建不可变数据流

## 面试题

**Q1: ArkTS 和 TypeScript 有什么区别？为什么要限制动态特性？**
> ArkTS 是基于 TypeScript 扩展的语言，核心区别：1) 禁止 `any` 类型，强制类型注解；2) 禁止 `eval`/`with` 等动态特性；3) 限制动态属性访问和原型链修改；4) 使用 AOT 编译而非 JIT。限制动态特性的原因：方舟编译器（Ark Compiler）在编译期需要确定所有类型信息才能进行优化（内联、逃逸分析、死代码消除），动态特性使得编译器无法做静态分析，导致无法生成高效的本地机器码。AOT 编译的优势是运行时性能接近原生，代价是牺牲灵活性。

**Q2: @State、@Prop、@Link、@Provide/@Consume 各自的使用场景是什么？**
> `@State`：组件内部状态，变化触发当前组件 UI 刷新，是最基础的状态装饰器；`@Prop`：父→子单向数据传递，子组件只读，适合展示型子组件；`@Link`：父子双向绑定，子组件可修改并同步回父组件，使用 `$` 语法传递引用（如 `value: $count`）；`@Provide/@Consume`：跨层级数据传递（类似 React Context），祖先组件 `@Provide` 提供数据，后代组件 `@Consume` 消费数据，中间层无需透传。选择原则：内部状态用 @State，父子单向用 @Prop，父子双向用 @Link，跨层级用 @Provide/@Consume。

**Q3: ArkTS 的空安全机制有哪些？分别适用于什么场景？**
> 三大机制：1) 可选链 `?.`：安全访问可能为 null/undefined 的属性/方法，遇到空值短路返回 undefined，适用于深层嵌套属性的访问（如 `user.address?.city`）；2) 非空断言 `!`：告诉编译器"我确定不为空"，编译通过但运行时如果为空会崩溃，适用于开发者能确保非空的场景，应谨慎使用；3) 空值合并 `??`：当左侧为 null/undefined 时返回右侧默认值，与 `||` 的区别是 `??` 只对 null/undefined 生效，0、''、false 等 falsy 值不会触发。最佳实践：优先使用可选链和空值合并，减少非空断言。

**Q4: TaskPool 和 Worker 有什么区别？如何选择？**
> TaskPool 是鸿蒙提供的线程池 API，自动管理线程生命周期，通过 `@Concurrent` 装饰函数后用 `taskpool.execute()` 提交任务，适合短时 CPU 密集型任务（如数据处理、图片压缩），任务完成后线程自动回收；Worker 是手动创建管理的后台线程，通过 `postMessage` 通信，适合长时间运行的后台任务（如 WebSocket 长连接、持续数据处理），需要手动 `terminate()` 终止。选择原则：短时任务用 TaskPool（简单高效），长时任务用 Worker（持续运行），注意 Worker 最多创建 8 个。

**Q5: ArkTS 中的泛型约束有哪些方式？请举例说明。**
> 三种主要方式：1) `extends` 接口约束：`<T extends HasLength>` 确保 T 有 length 属性，函数内可安全访问 `item.length`；2) `keyof` 约束：`<K extends keyof T>` 确保 K 是 T 的合法属性名，常用于安全属性访问函数 `getProperty(obj, key)`；3) 多重约束（交叉类型）：`<T extends Identifiable & Timestamped>` 要求 T 同时满足多个接口。此外还有 `new()` 约束（工厂模式：`ctor: new () => T`）和字面量约束（`<T extends string>` 限定字符串子类型）。泛型约束的核心目的是在编译期保证类型安全，避免运行时错误。

**Q6: @Builder、@Extend、@Styles 三种复用方式有什么区别？**
> `@Builder`：定义可复用的 UI 构建逻辑（可含状态逻辑），在组件内部定义，通过 `this.builderName()` 调用，适合有逻辑的 UI 片段复用（如列表项模板）；`@Extend`：扩展内置组件的样式方法，只能定义在组件外部（全局），绑定特定组件类型，支持传参，如 `@Extend(Text) function highlightText(color)` 只能用于 Text 组件；`@Styles`：提取通用样式集合，不绑定特定组件，所有组件都可使用，不能传参（参数需固定），适合纯粹的无逻辑样式复用。选择：纯样式无参数 → @Styles，样式绑定组件类型 → @Extend，含 UI 逻辑 → @Builder。

**Q7: ArkTS 中如何实现可辨识联合（Discriminated Union）？它解决了什么问题？**
> 可辨识联合是利用一个共同的字面量属性（辨识属性）来区分联合类型中不同成员的模式。实现步骤：1) 定义多个接口，每个接口含一个相同名称但不同字面量值的属性（如 `kind: 'circle'`）；2) 用联合类型组合这些接口；3) 在 switch/if 中根据辨识属性收窄类型。它解决的问题：普通联合类型在收窄时需要用 `in` 或 `instanceof` 判断，不够直观且容易遗漏；可辨识联合让 TypeScript 自动根据辨识属性值收窄到对应接口，保证类型安全。结合 `assertNever` 可实现穷尽检查，遗漏分支时编译报错。

**Q8: 前端开发者从 TypeScript 转到 ArkTS 最大的挑战是什么？如何应对？**
> 最大挑战有三个：1) 动态特性限制 — 不能用 `any`、`eval`、动态属性等，需改用静态类型和类型守卫；2) 声明式 UI 范式转变 — 从 React/Vue 的模板或 JSX 切换到 struct + 装饰器的声明式写法，状态管理从 Hooks/Composition API 切换到 @State/@Prop/@Link 装饰器；3) 生态差异 — 从 npm/Webpack 切换到 ohpm/Hvigor，API 从浏览器 API 切换到鸿蒙 SDK。应对策略：先理解 ArkTS 对 TS 的限制（对照迁移清单逐项修改），再学习装饰器驱动的 ArkUI（从简单组件开始），最后熟悉鸿蒙 SDK 和 ohpm 生态。建议先写一个完整的 Todo 应用练手，覆盖状态管理、列表渲染、网络请求、页面导航等核心场景。

---

**相关链接：**
- [[鸿蒙系统架构与开发入门]]
- [[ArkUI声明式开发]]
- [[TypeScript核心]]
- [[Dart语言核心]]

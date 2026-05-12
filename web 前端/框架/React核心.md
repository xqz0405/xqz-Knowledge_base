---
tags:
  - Web前端
  - React
  - 框架
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# React核心

## What — 是什么

> React 是 Facebook 开发的 UI 库，以组件化、声明式和虚拟 DOM 为核心理念，通过数据驱动视图更新。

### 核心概念

- **JSX**：JavaScript 的语法扩展，编译为 `React.createElement()` 调用
- **虚拟 DOM**：内存中的 DOM 树表示，通过 diff 算法最小化真实 DOM 操作
- **组件**：函数组件（推荐）和类组件，接收 props 返回 UI
- **Hooks**：`useState`、`useEffect`、`useRef` 等，让函数组件拥有状态和副作用

### JSX 编译过程详解

JSX 并不是浏览器可以直接运行的语法，它需要经过编译器（Babel / SWC）转换为 JavaScript：

```
JSX 源码
  ↓ Babel/SWC 编译
React.createElement() 调用
  ↓ 运行时执行
React Element 对象（虚拟 DOM 节点）
```

**编译前后对比：**

```jsx
// JSX 源码
const element = (
    <div className="app">
        <h1 id="title">Hello, {name}</h1>
        <p>World</p>
    </div>
);

// 编译后（Babel 输出）
const element = React.createElement(
    'div',
    { className: 'app' },
    React.createElement('h1', { id: 'title' }, 'Hello, ', name),
    React.createElement('p', null, 'World')
);

// React 17+ 新 JSX 转换（无需手动 import React）
// 编译后自动引入 react/jsx-runtime
import { jsx as _jsx, jsxs as _jsxs } from 'react/jsx-runtime';
const element = _jsxs('div', {
    className: 'app',
    children: [
        _jsxs('h1', { id: 'title', children: ['Hello, ', name] }),
        _jsx('p', { children: 'World' })
    ]
});
```

**`React.createElement(type, props, ...children)` 参数说明：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `string` \| `function` \| `Symbol` | HTML 标签名、函数组件或 React.Fragment |
| `props` | `object` \| `null` | 属性对象（className、onClick 等） |
| `...children` | `any` | 子元素，可以是字符串、数字、Element |

### Virtual DOM 的完整结构

`React.createElement()` 返回的是一个普通的 JavaScript 对象，即 React Element（虚拟 DOM 节点）：

```javascript
// 一个 React Element 的完整结构
const element = {
    $$typeof: Symbol.for('react.element'),  // 类型标识，防止 XSS 注入
    type: 'div',                             // 元素类型（标签名/组件函数/Fragment）
    key: null,                               // 列表渲染的 key，帮助 diff 复用
    ref: null,                               // DOM 引用或组件实例引用
    props: {                                 // 属性和子元素
        className: 'app',
        children: [
            { $$typeof: Symbol.for('react.element'), type: 'h1', key: null, ref: null, props: { children: 'Hello' } },
            { $$typeof: Symbol.for('react.element'), type: 'p', key: null, ref: null, props: { children: 'World' } }
        ]
    },
    _owner: null,      // 创建该元素的组件实例（Fiber）
    _store: {},        // 开发环境校验用
};
```

**关键字段说明：**

| 字段 | 作用 |
|------|------|
| `$$typeof` | 安全标识，React 用 `Symbol` 标记合法 Element，防止伪造对象注入 |
| `type` | 决定渲染方式：字符串渲染原生标签，函数调用组件获取子树 |
| `key` | 在列表 diff 中标识节点身份，影响复用/新建/删除决策 |
| `ref` | 获取 DOM 节点或类组件实例的引用 |
| `props` | 包含所有属性 + `children`，是自顶向下传递的数据载体 |

> 注意：React Element 是**不可变**的（immutable）。一旦创建，其 `type`、`props`、`key` 不可修改。每次渲染都会创建新的 Element 对象。

### Fiber 架构详解

React 16 重写了核心算法，引入 Fiber 架构。Fiber 既是新的协调算法，也是一种数据结构。

**Fiber Node 结构（核心字段）：**

```javascript
// 简化的 Fiber Node 结构
const fiber = {
    // 静态结构 — 描述组件
    tag: FunctionComponent,     // 组件类型（函数/类/原生标签等）
    type: MyComponent,          // 对应的函数/类/标签名
    key: null,                  // 列表 key

    // 实例信息
    stateNode: domNode,         // 对应的真实 DOM 节点或组件实例

    // Fiber 树结构 — 深度优先遍历
    return: parentFiber,        // 父 Fiber
    child: firstChildFiber,     // 第一个子 Fiber
    sibling: nextSiblingFiber,  // 右侧兄弟 Fiber

    // 工作单元 — 增量渲染
    pendingProps: newProps,     // 待处理的新 props
    memoizedProps: oldProps,    // 上次渲染的 props
    memoizedState: hookList,    // 上次渲染的 state（Hooks 链表）
    updateQueue: queue,         // 更新队列

    // 副作用
    flags: Placement,           // 需要执行的操作（插入/更新/删除）
    subtreeFlags: 0,            // 子树副作用标识（优化跳过无变更子树）
    deletions: null,            // 需要删除的子 Fiber 列表

    // 双缓冲
    alternate: currentFiber,    // 指向另一棵树的对应节点
};
```

**双缓冲机制（Double Buffering）：**

React 同时维护两棵 Fiber 树：

```
current 树（当前屏幕上显示的）
    ↕ alternate 指针
workInProgress 树（正在内存中构建的）
```

- `current` 树：对应已渲染到屏幕上的 UI
- `workInProgress` 树：正在内存中构建的新树
- 每个 Fiber 节点通过 `alternate` 指针与另一棵树的对应节点相连
- 渲染完成后，两棵树通过切换 `current` 指针完成"换帧"，实现无闪烁更新

**时间切片（Time Slicing）：**

```
|---渲染 Fiber1---|---渲染 Fiber2---|---让出主线程---|---渲染 Fiber3---|
|<---  一个时间片（~5ms）  --->|           |<--- 下一个时间片 --->|
                               ↓
                        处理用户输入等高优先级任务
```

- 利用 `MessageChannel`（宏任务）实现调度，兼容性优于 `requestIdleCallback`
- 每个时间片约 5ms，超时后将控制权交还主线程
- 高优先级更新（用户输入）可中断低优先级更新（数据请求后的渲染）

### Reconciler 工作流程

Reconciler（协调器）是 React 的核心算法，负责比较新旧 Fiber 树并标记变更。工作流程分为三个阶段：

**1. Render 阶段（可中断）—— `beginWork` → `completeWork`**

```
beginWork：自顶向下递归
  ├─ 根据 type 创建/复用子 Fiber
  ├─ 比较新旧 props，标记副作用（flags）
  └─ 返回 child Fiber 继续递归

completeWork：自底向上回溯
  ├─ 处理 Fiber 的 DOM 操作（创建/更新属性）
  ├─ 收集子树副作用到 subtreeFlags
  └─ 冒泡副作用到父节点
```

**2. Commit 阶段（不可中断）—— `commitWork`**

```
commitWork：同步执行 DOM 操作
  ├─ BeforeMutation：读取 DOM 快照（getSnapshotBeforeUpdate）
  ├─ Mutation：执行 DOM 插入/更新/删除
  └─ Layout：执行 useEffect 清理函数 / useLayoutEffect 回调
```

**整体流程图：**

```
State/Props 变化
    ↓
创建 Update 对象加入 UpdateQueue
    ↓
Scheduler 调度（根据优先级安排执行时机）
    ↓
Render 阶段（可中断）
  ├─ beginWork：自顶向下处理每个 Fiber
  │   └─ bailout 优化：props 未变且 context 未变 → 跳过子树
  └─ completeWork：自底向上收集副作用
    ↓
Commit 阶段（不可中断）
  ├─ BeforeMutation 阶段
  ├─ Mutation 阶段：操作真实 DOM
  └─ Layout 阶段：执行同步副作用
    ↓
屏幕更新完成
    ↓
调度 useEffect（异步执行，不阻塞浏览器绘制）
```

### React 18 并发特性

React 18 引入了并发渲染（Concurrent Rendering），允许 React 同时准备多个版本的 UI。

**useTransition —— 非阻塞状态更新：**

- 将状态更新标记为"过渡"（低优先级），不阻塞用户输入
- 返回 `[isPending, startTransition]`，`isPending` 表示过渡是否在进行中
- 适用于：搜索过滤、标签切换、列表排序等非紧急更新

**useDeferredValue —— 延迟渲染：**

- 接收一个值，返回该值的延迟版本
- 当有紧急更新时，React 先处理紧急更新，再更新延迟值
- 适用于：搜索输入框的实时过滤、大列表的延迟更新

**Suspense —— 声明式加载状态：**

- 声明 fallback UI，当子组件未就绪时显示
- React 18 增强了 Suspense，支持服务端流式渲染
- 可与 `React.lazy`、数据获取库（React Query / Relay）配合使用

**自动批处理（Automatic Batching）：**

- React 18 之前：只有 React 事件处理函数中的更新会自动批处理
- React 18：所有更新（setTimeout、Promise、原生事件）都会自动批处理
- 减少不必要的重渲染次数

### Server Components 概念

React Server Components（RSC）是 React 18+ 引入的服务端组件架构：

| 类型 | 执行位置 | 客户端 JS | 状态/Hooks |
|------|---------|----------|-----------|
| Server Components | 服务端 | 零 JS | 不支持 |
| Client Components | 客户端 | 完整 | 支持 |

- Server Components 在服务端执行，渲染结果序列化后发送到客户端
- 组件代码及其依赖（如 markdown 渲染库）不会增加客户端 bundle
- 通过 `'use client'` 指令声明客户端组件边界
- 详见 [[React Server Components]]

### 合成事件系统

React 实现了一套合成事件（SyntheticEvent）系统，而非直接使用原生 DOM 事件：

**事件委托机制：**

```
原生事件冒泡
    ↓
React 17+：事件绑定到 root 节点（而非 document）
    ↓
根据事件 target 找到对应的 Fiber 节点
    ↓
触发该 Fiber 上的合成事件处理函数
```

**SyntheticEvent 特性：**

| 特性 | 说明 |
|------|------|
| 跨浏览器兼容 | 抹平 IE 和标准浏览器的事件差异 |
| 事件池（React 17 废弃） | 旧版复用事件对象以提高性能，需 `e.persist()` 保存 |
| 自动清理 | 事件回调执行完毕后，事件对象属性被置空 |
| 命名规范化 | 原生 `onclick` → React `onClick`，`class` → `className` |

**合成事件 vs 原生事件执行顺序：**

```jsx
function App() {
    useEffect(() => {
        document.addEventListener('click', () => {
            console.log('1. document 原生事件');
        });
    }, []);

    const handleClick = () => {
        console.log('2. React 合成事件');
    };

    return <div onClick={handleClick}>
        <button onClick={(e) => {
            // e.stopPropagation() 只阻止合成事件冒泡
            // 原生事件不受影响
            console.log('3. button 合成事件');
        }}>Click</button>
    </div>;
}

// 点击按钮输出顺序：1 → 3 → 2
// 原生事件（冒泡到 document）→ 子元素合成事件 → 父元素合成事件
```

### 核心架构

- 设计理念：UI = f(state)，单向数据流
- 核心模块：Fiber 调度器（Scheduler）、Reconciler（协调器）、Renderer（渲染器）
- 数据流：State/Props 变化 → Reconciler diff → Renderer 更新 DOM

### 插件生态

- 官方插件：React DOM、React Native
- 社区热门：React Router（路由）、Redux/Zustand（状态管理）、React Query（数据请求）

## Why — 为什么

### 适用场景

- 单页应用（SPA）
- 复杂交互的 UI（表单、拖拽、动画）
- 跨平台应用（React Native）
- 大型团队协作项目
- 需要丰富组件库的企业级后台系统

### 对比同类框架

| 维度 | React | Vue | Svelte | Solid |
|------|-------|-----|--------|-------|
| 性能 | 高（虚拟 DOM diff） | 高（响应式更新） | 极高（编译时优化） | 极高（信号式细粒度更新） |
| 生态 | 最丰富 | 丰富 | 中等 | 较小 |
| 学习曲线 | 中（Hooks 心智模型） | 低 | 低 | 中（类 JSX 但无 VDOM） |
| 灵活性 | 极高 | 中（约定明确） | 中 | 高 |
| 运行时大小 | ~42KB | ~33KB | ~2KB | ~7KB |
| 更新机制 | VDOM diff 全组件 | Proxy 响应式组件级 | 编译时确定更新路径 | Signal 细粒度 DOM 更新 |
| 状态管理 | useState/useReducer | ref/reactive | 赋值即更新 | createSignal |
| SSR 方案 | Next.js / RSC | Nuxt.js | SvelteKit | SolidStart |
| TS 支持 | 优秀 | 优秀 | 良好 | 优秀 |

**React 的独特优势：**

- **企业级生态**：最丰富的组件库（Ant Design、MUI）、工具链和社区方案
- **RSC 架构**：Server Components 从架构层面减少客户端 JS，是框架级创新
- **跨平台统一**：React Native 实现真正的"Learn once, write anywhere"
- **大厂背书**：Facebook/Meta 主导维护，Netflix、Airbnb 等大厂深度使用
- **人才市场**：React 开发者需求量最大，招聘和求职都更容易

**何时选择 React：**

- 团队规模大，需要成熟的生态和组件库支持
- 需要跨平台（Web + Native），React Native 是最成熟的方案
- 项目复杂度高，需要灵活的架构设计
- 已有 React 技术栈积累，或团队对函数式编程有偏好
- 需要 Server Components 等前沿特性

### 优缺点

- 优点：
  - 生态庞大，组件库丰富
  - 函数式编程思想，UI 可预测
  - 跨平台（Web/Native/SSR）
  - 大厂维护，长期稳定
  - 并发模式提升交互响应性
- 缺点：
  - Hooks 心智负担重（依赖数组、闭包陷阱）
  - 没有官方路由/状态管理（需选型）
  - 重新渲染优化需要手动 memo/useMemo
  - 编译时优化不足（相比 Svelte/Solid）

## How — 怎么用

### 快速上手

```jsx
import { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(c => c + 1)}>+1</button>
        </div>
    );
}
```

### 代码示例

#### 1. 副作用与清理（useEffect + AbortController）

```jsx
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);

    useEffect(() => {
        const controller = new AbortController();

        fetch(`/api/users/${userId}`, { signal: controller.signal })
            .then(res => res.json())
            .then(setUser);

        return () => controller.abort(); // 清理：取消请求
    }, [userId]); // 依赖数组：userId 变化时重新执行

    if (!user) return <div>Loading...</div>;
    return <div>{user.name}</div>;
}
```

#### 2. 性能优化（memo/useMemo/useCallback）

```jsx
import { memo, useMemo, useCallback } from 'react';

// 避免不必要的重渲染
const ExpensiveList = memo(function List({ items, onSelect }) {
    return items.map(item => (
        <div key={item.id} onClick={() => onSelect(item.id)}>
            {item.name}
        </div>
    ));
});

function Parent() {
    const [count, setCount] = useState(0);
    const items = useMemo(() => heavyFilter(rawItems), [rawItems]);
    const handleSelect = useCallback((id) => navigate(id), []);

    return (
        <>
            <button onClick={() => setCount(c => c + 1)}>{count}</button>
            <ExpensiveList items={items} onSelect={handleSelect} />
        </>
    );
}
```

#### 3. useReducer 复杂状态管理

当组件有多个相关联的状态，或下一个状态依赖前一个状态时，`useReducer` 比 `useState` 更合适：

```jsx
import { useReducer } from 'react';

// 定义 reducer 函数
function todoReducer(state, action) {
    switch (action.type) {
        case 'ADD':
            return [...state, { id: Date.now(), text: action.text, done: false }];
        case 'TOGGLE':
            return state.map(todo =>
                todo.id === action.id ? { ...todo, done: !todo.done } : todo
            );
        case 'DELETE':
            return state.filter(todo => todo.id !== action.id);
        case 'CLEAR_COMPLETED':
            return state.filter(todo => !todo.done);
        default:
            return state;
    }
}

function TodoApp() {
    const [todos, dispatch] = useReducer(todoReducer, []);
    const [input, setInput] = useState('');

    const handleAdd = () => {
        if (input.trim()) {
            dispatch({ type: 'ADD', text: input.trim() });
            setInput('');
        }
    };

    return (
        <div>
            <input
                value={input}
                onChange={e => setInput(e.target.value)}
                onKeyDown={e => e.key === 'Enter' && handleAdd()}
            />
            <button onClick={handleAdd}>添加</button>
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>
                        <span
                            style={{ textDecoration: todo.done ? 'line-through' : 'none' }}
                            onClick={() => dispatch({ type: 'TOGGLE', id: todo.id })}
                        >
                            {todo.text}
                        </span>
                        <button onClick={() => dispatch({ type: 'DELETE', id: todo.id })}>
                            删除
                        </button>
                    </li>
                ))}
            </ul>
            <button onClick={() => dispatch({ type: 'CLEAR_COMPLETED' })}>
                清除已完成
            </button>
        </div>
    );
}
```

#### 4. useContext + useReducer 全局状态

将 `useReducer` 与 `useContext` 结合，实现轻量级全局状态管理，无需引入 Redux：

```jsx
import { createContext, useContext, useReducer } from 'react';

// 1. 创建 Context
const AppContext = createContext(null);
const AppDispatchContext = createContext(null);

// 2. 定义 reducer
function appReducer(state, action) {
    switch (action.type) {
        case 'SET_USER':
            return { ...state, user: action.payload };
        case 'SET_THEME':
            return { ...state, theme: action.payload };
        case 'LOGOUT':
            return { ...state, user: null };
        default:
            return state;
    }
}

const initialState = { user: null, theme: 'light' };

// 3. Provider 组件
function AppProvider({ children }) {
    const [state, dispatch] = useReducer(appReducer, initialState);

    return (
        <AppContext.Provider value={state}>
            <AppDispatchContext.Provider value={dispatch}>
                {children}
            </AppDispatchContext.Provider>
        </AppContext.Provider>
    );
}

// 4. 自定义 Hook — 读写分离，避免不必要的重渲染
function useAppState() {
    const context = useContext(AppContext);
    if (context === null) throw new Error('useAppState must be used within AppProvider');
    return context;
}

function useAppDispatch() {
    const context = useContext(AppDispatchContext);
    if (context === null) throw new Error('useAppDispatch must be used within AppProvider');
    return context;
}

// 5. 使用
function Header() {
    const { user, theme } = useAppState();
    const dispatch = useAppDispatch();

    return (
        <header>
            <span>主题: {theme}</span>
            {user ? (
                <>
                    <span>{user.name}</span>
                    <button onClick={() => dispatch({ type: 'LOGOUT' })}>退出</button>
                </>
            ) : (
                <span>未登录</span>
            )}
        </header>
    );
}

// 入口
function App() {
    return (
        <AppProvider>
            <Header />
        </AppProvider>
    );
}
```

#### 5. useTransition 非阻塞更新

`useTransition` 将状态更新标记为低优先级，让紧急更新（如用户输入）不被阻塞：

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
    const [query, setQuery] = useState('');           // 紧急状态：输入框立即响应
    const [searchTerm, setSearchTerm] = useState(''); // 非紧急状态：搜索结果延迟更新
    const [isPending, startTransition] = useTransition();

    const handleChange = (e) => {
        // 紧急更新：输入框立即显示用户输入
        setQuery(e.target.value);

        // 非紧急更新：搜索过滤可以稍后执行
        startTransition(() => {
            setSearchTerm(e.target.value);
        });
    };

    const filteredItems = useMemo(() => {
        if (!searchTerm) return items;
        return items.filter(item =>
            item.name.toLowerCase().includes(searchTerm.toLowerCase())
        );
    }, [searchTerm]);

    return (
        <div>
            <input value={query} onChange={handleChange} placeholder="搜索..." />
            {isPending && <span>搜索中...</span>}
            <ul style={{ opacity: isPending ? 0.7 : 1 }}>
                {filteredItems.map(item => (
                    <li key={item.id}>{item.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

#### 6. useDeferredValue 延迟渲染

`useDeferredValue` 是 `useTransition` 的声明式替代，适用于无法控制 setState 的场景：

```jsx
import { useState, useDeferredValue, useMemo } from 'react';

function DeferredSearch() {
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query); // 延迟版本的 query

    // 使用 deferredQuery 进行昂贵的计算
    // 当用户输入时，React 会优先更新 input，延迟更新列表
    const filteredList = useMemo(() => {
        return expensiveFilter(items, deferredQuery);
    }, [deferredQuery]);

    return (
        <div>
            {/* 输入框始终使用最新的 query，保证即时响应 */}
            <input value={query} onChange={e => setQuery(e.target.value)} />

            {/* 列表使用延迟的 deferredQuery，避免输入卡顿 */}
            <List items={filteredList} />
        </div>
    );
}
```

**useTransition vs useDeferredValue 对比：**

| 维度 | useTransition | useDeferredValue |
|------|--------------|-----------------|
| 控制方式 | 命令式（包裹 setState） | 声明式（包装值） |
| 适用场景 | 可以控制 setState 调用时 | 无法控制 setState（如第三方组件） |
| isPending | 提供 isPending 状态 | 无（需自行判断） |
| 优先级控制 | 显式标记整个更新为过渡 | 自动延迟值的更新 |

#### 7. ErrorBoundary 错误边界

React 的错误边界用于捕获子组件树的渲染错误，防止整个应用崩溃。**错误边界必须是类组件**：

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    // 静态方法：渲染阶段捕获错误，返回新的 state
    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }

    // 生命周期方法：commit 阶段捕获错误，可执行副作用
    componentDidCatch(error, errorInfo) {
        console.error('ErrorBoundary caught:', error, errorInfo);
        // 上报错误到监控服务
        logErrorToService(error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            // 自定义降级 UI
            return this.props.fallback || (
                <div role="alert">
                    <h2>出错了</h2>
                    <p>{this.state.error.message}</p>
                    <button onClick={() => this.setState({ hasError: false })}>
                        重试
                    </button>
                </div>
            );
        }
        return this.props.children;
    }
}

// 使用
function App() {
    return (
        <ErrorBoundary fallback={<div>组件加载失败，请刷新页面</div>}>
            <Header />
            <ErrorBoundary>
                <MainContent />  {/* 独立包裹，局部错误不影响整体 */}
            </ErrorBoundary>
            <Footer />
        </ErrorBoundary>
    );
}
```

> 注意：错误边界无法捕获以下场景的错误：事件处理函数中的错误（需 try/catch）、异步代码（setTimeout/Promise）、服务端渲染错误、错误边界自身抛出的错误。

#### 8. useRef 高级用法

`useRef` 不仅仅是获取 DOM 引用，还有更多实用场景：

```jsx
import { useRef, useEffect, useState } from 'react';

function RefDemo() {
    // 用法 1：DOM 引用 — 聚焦输入框
    const inputRef = useRef(null);
    const focusInput = () => inputRef.current?.focus();

    // 用法 2：实例变量 — 保存不触发渲染的可变值
    const renderCount = useRef(0);
    renderCount.current += 1; // 修改不触发重渲染

    // 用法 3：保存前一次渲染的值（自定义 Hook 基础）
    const prevValueRef = useRef('');
    const [value, setValue] = useState('');
    useEffect(() => {
        prevValueRef.current = value; // 渲染完成后更新
    }, [value]);

    // 用法 4：命令式 API — 暴露特定方法给父组件
    const videoRef = useRef(null);
    const play = () => videoRef.current?.play();
    const pause = () => videoRef.current?.pause();

    // 用法 5：存储定时器 ID，避免闭包陷阱
    const timerRef = useRef(null);
    const startTimer = () => {
        timerRef.current = setInterval(() => {
            console.log('tick');
        }, 1000);
    };
    const stopTimer = () => clearInterval(timerRef.current);

    // 用法 6：标记组件是否已卸载，防止卸载后 setState
    const isMountedRef = useRef(true);
    useEffect(() => {
        return () => { isMountedRef.current = false; };
    }, []);

    return (
        <div>
            <input ref={inputRef} value={value} onChange={e => setValue(e.target.value)} />
            <button onClick={focusInput}>聚焦</button>
            <p>渲染次数: {renderCount.current}</p>
            <p>上次值: {prevValueRef.current}</p>
            <video ref={videoRef} src="video.mp4" />
            <button onClick={play}>播放</button>
            <button onClick={pause}>暂停</button>
        </div>
    );
}
```

#### 9. useImperativeHandle 暴露方法

`useImperativeHandle` 配合 `forwardRef`，让父组件只能访问子组件暴露的特定方法，而非整个 DOM 节点：

```jsx
import { useRef, useImperativeHandle, forwardRef } from 'react';

const Modal = forwardRef(function Modal(props, ref) {
    const dialogRef = useRef(null);

    // 只暴露 open 和 close 方法，隐藏内部 DOM 结构
    useImperativeHandle(ref, () => ({
        open: () => dialogRef.current?.showModal(),
        close: () => dialogRef.current?.close(),
    }), []); // 依赖数组为空，方法引用稳定

    return (
        <dialog ref={dialogRef}>
            <h2>{props.title}</h2>
            <p>{props.children}</p>
            <button onClick={() => dialogRef.current?.close()}>关闭</button>
        </dialog>
    );
});

// 父组件使用
function App() {
    const modalRef = useRef(null);

    return (
        <div>
            <button onClick={() => modalRef.current?.open()}>打开弹窗</button>
            <Modal ref={modalRef} title="提示">这是一个弹窗内容</Modal>
        </div>
    );
}
```

#### 10. 自定义 Hook 示例

**useDebounce —— 防抖 Hook：**

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay = 300) {
    const [debouncedValue, setDebouncedValue] = useState(value);

    useEffect(() => {
        const timer = setTimeout(() => setDebouncedValue(value), delay);
        return () => clearTimeout(timer);
    }, [value, delay]);

    return debouncedValue;
}

// 使用
function SearchInput() {
    const [query, setQuery] = useState('');
    const debouncedQuery = useDebounce(query, 500);

    useEffect(() => {
        if (debouncedQuery) {
            fetchSearchResults(debouncedQuery);
        }
    }, [debouncedQuery]);

    return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

**useLocalStorage —— 持久化状态：**

```jsx
import { useState, useCallback } from 'react';

function useLocalStorage(key, initialValue) {
    const [storedValue, setStoredValue] = useState(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch {
            return initialValue;
        }
    });

    const setValue = useCallback((value) => {
        const valueToStore = value instanceof Function ? value(storedValue) : value;
        setStoredValue(valueToStore);
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
    }, [key, storedValue]);

    const removeValue = useCallback(() => {
        setStoredValue(initialValue);
        window.localStorage.removeItem(key);
    }, [key, initialValue]);

    return [storedValue, setValue, removeValue];
}

// 使用
function Settings() {
    const [theme, setTheme] = useLocalStorage('theme', 'light');
    const [fontSize, setFontSize] = useLocalStorage('fontSize', 14);

    return (
        <div>
            <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
                当前主题: {theme}
            </button>
            <button onClick={() => setFontSize(s => s + 2)}>字号: {fontSize}px</button>
        </div>
    );
}
```

**useFetch —— 数据请求 Hook：**

```jsx
import { useState, useEffect, useCallback } from 'react';

function useFetch(url, options = {}) {
    const [data, setData] = useState(null);
    const [error, setError] = useState(null);
    const [loading, setLoading] = useState(true);

    const fetchData = useCallback(async () => {
        setLoading(true);
        setError(null);
        try {
            const controller = new AbortController();
            const res = await fetch(url, { ...options, signal: controller.signal });
            if (!res.ok) throw new Error(`HTTP ${res.status}`);
            const json = await res.json();
            setData(json);
        } catch (err) {
            if (err.name !== 'AbortError') setError(err.message);
        } finally {
            setLoading(false);
        }
    }, [url]);

    useEffect(() => {
        fetchData();
    }, [fetchData]);

    return { data, error, loading, refetch: fetchData };
}

// 使用
function UserList() {
    const { data: users, error, loading, refetch } = useFetch('/api/users');

    if (loading) return <div>加载中...</div>;
    if (error) return <div>出错了: {error}</div>;

    return (
        <div>
            <button onClick={refetch}>刷新</button>
            <ul>
                {users.map(user => <li key={user.id}>{user.name}</li>)}
            </ul>
        </div>
    );
}
```

#### 11. React.memo 高级用法（自定义比较函数）

`React.memo` 第二个参数允许自定义 props 比较逻辑，实现更精细的重渲染控制：

```jsx
import { memo } from 'react';

// 默认浅比较：props 的每一层引用都变了就重渲染
// 自定义比较函数：返回 true 表示 props 相等，跳过渲染
const UserCard = memo(function UserCard({ user, onSelect }) {
    console.log('UserCard rendered:', user.name);
    return (
        <div onClick={() => onSelect(user.id)}>
            <img src={user.avatar} alt={user.name} />
            <h3>{user.name}</h3>
            <p>{user.email}</p>
        </div>
    );
}, (prevProps, nextProps) => {
    // 自定义比较：只有 id 和 name 变化时才重渲染
    // 忽略 avatar URL 变化（可能只是 CDN 缓存刷新）
    // 忽略 onSelect 引用变化（回调函数通常每次渲染都是新引用）
    return prevProps.user.id === nextProps.user.id
        && prevProps.user.name === nextProps.user.name
        && prevProps.user.email === nextProps.user.email;
});

// 另一个示例：列表项只关心 id 和 status
const TaskItem = memo(function TaskItem({ task, onToggle, onDelete }) {
    return (
        <div>
            <input
                type="checkbox"
                checked={task.done}
                onChange={() => onToggle(task.id)}
            />
            <span style={{ textDecoration: task.done ? 'line-through' : 'none' }}>
                {task.title}
            </span>
            <button onClick={() => onDelete(task.id)}>删除</button>
        </div>
    );
}, (prev, next) => {
    // 只在 task.id 或 task.done 变化时重渲染
    // onToggle/onDelete 引用变化不触发重渲染
    return prev.task.id === next.task.id && prev.task.done === next.task.done;
});
```

> 注意：自定义比较函数应保持简单。如果比较逻辑比重渲染本身还复杂，就失去了 memo 的意义。大多数场景下，默认浅比较 + `useCallback`/`useMemo` 已足够。

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 闭包陷阱 | useEffect 中引用了旧的 state 值 | 依赖数组包含所有使用的变量，或用 ref 存最新值 |
| 无限渲染 | useEffect 依赖导致循环更新 | 检查依赖是否在 effect 内被修改 |
| 子组件不必要渲染 | 父组件重渲染导致所有子组件重渲染 | `React.memo` + `useCallback`/`useMemo` |
| 状态更新批处理 | React 18 自动批处理可能不符合预期 | 使用 `flushSync` 强制同步更新 |
| useState 直接修改对象 | 直接修改 state 对象不会触发渲染 | 始终创建新对象/数组（`{...obj}` / `[...arr]`） |
| useEffect 依赖对象/数组 | 每次渲染创建新引用导致 effect 反复执行 | 用 `useMemo` 缓存对象，或提取原始值作为依赖 |
| key 使用 index | 列表增删时状态错位 | 使用唯一标识（id）作为 key |
| 事件处理函数中 this 丢失 | 类组件事件回调 this 不是组件实例 | 箭头函数、bind、或改用函数组件 |

### 最佳实践

- 依赖数组宁多勿少，用 ESLint `exhaustive-deps` 规则
- 状态尽量下放，避免顶层巨型 state
- 复杂状态用 `useReducer` 替代多个 `useState`
- 数据请求用 React Query/SWR，手写 useEffect 易出 bug
- 大型项目用 `useContext` + `useReducer` 或 Zustand 管理全局状态
- 组件拆分粒度适中，单一职责：一个组件只做一件事
- 自定义 Hook 提取可复用逻辑，命名以 `use` 开头
- `useMemo`/`useCallback` 只在性能瓶颈时使用，不要过度优化

## 面试题

**Q1: Virtual DOM 的工作原理是什么？**
> Virtual DOM 是内存中的 JS 对象树，是对真实 DOM 的轻量表示。状态变化时先生成新的 VNode 树，通过 diff 算法与旧树比较找出最小差异，再批量更新真实 DOM，避免频繁操作 DOM 带来的性能开销。

**Q2: React 的 diff 算法有哪些优化策略？**
> 三大策略：1. 同层比较，不跨层级；2. 不同类型的节点直接替换，不做递归；3. 通过 key 识别同层节点的移动/复用。这使得 diff 从 O(n3) 降为 O(n)。

**Q3: 列表渲染中 key 的作用是什么？为什么不建议用 index 作为 key？**
> key 帮助 diff 算法识别哪些节点可复用。用 index 作 key 时，若列表发生插入/删除/重排，元素顺序变化但 key 不变，导致 diff 错误复用节点，可能出现渲染错乱或状态错位。

**Q4: Fiber 架构解决了什么问题？**
> 旧版 React 递归渲染不可中断，长任务会阻塞主线程导致卡顿。Fiber 将渲染拆分为多个小单元（fiber node），利用空闲时间片逐帧执行，实现可中断、可恢复的增量渲染，优先处理高优先级更新（如用户输入）。

**Q5: React 18 的并发模式（Concurrent Mode）有什么意义？**
> 并发模式让 React 可以同时准备多个版本的 UI，在渲染过程中可以暂停、放弃或重新执行渲染，确保紧急更新（输入、点击）不被慢速渲染阻塞，提升交互响应性。

**Q6: React 18 的自动批处理（Automatic Batching）是什么？**
> React 18 之前，只有 React 事件处理函数中的多次 `setState` 会自动合并为一次渲染（批处理），而 `setTimeout`、Promise、原生事件中的 `setState` 每次都会触发渲染。React 18 将批处理扩展到所有更新场景，无论更新来源是 React 事件、`setTimeout`、Promise 还是原生 DOM 事件，都会自动合并，减少不必要的重渲染。如果确实需要同步更新，可以使用 `flushSync()` 强制立即渲染。

**Q7: useLayoutEffect 和 useEffect 的区别？**
> 两者签名相同，区别在于执行时机：`useEffect` 在浏览器绘制（paint）之后异步执行，不阻塞屏幕更新；`useLayoutEffect` 在 DOM 变更后、浏览器绘制之前同步执行，会阻塞绘制。`useLayoutEffect` 适用于需要读取 DOM 布局信息并同步修改的场景（如测量元素尺寸后调整样式、防止闪烁），但会阻塞渲染应谨慎使用。服务端渲染时 `useLayoutEffect` 会报警告，可用 `useIsomorphicLayoutEffect` 兼容。

**Q8: React 合成事件和原生事件有什么区别？**
> 区别包括：(1) 合成事件是 React 跨浏览器封装的 SyntheticEvent 对象，抹平了 IE 等浏览器差异；(2) React 17+ 事件委托到 root 节点而非 document，方便微前端集成；(3) 执行顺序上，原生事件先于合成事件触发（因为事件冒泡到 root 后 React 才开始处理合成事件）；(4) `e.stopPropagation()` 只阻止合成事件冒泡，不阻止原生事件冒泡到 document；(5) 合成事件命名采用驼峰（onClick），原生事件全小写（onclick）；(6) React 17 废弃了事件池机制，不再需要 `e.persist()`。

**Q9: useRef 和 useState 有什么区别？各自适用场景？**
> 核心区别是修改是否触发重渲染：`useState` 的 setter 调用会触发组件重渲染，而修改 `useRef.current` 不会触发任何渲染。`useState` 适合需要在 UI 中展示的数据（计数器、表单值、开关状态）；`useRef` 适合不参与渲染但需要跨渲染持久化的值（DOM 引用、定时器 ID、前一次渲染值、是否已卸载标记）。此外，`useRef.current` 可以在渲染过程中直接修改（不推荐），`useState` 在渲染过程中调用 setter 会导致无限循环。

---

**相关链接：**
- [[Vue核心]]
- [[React Hooks详解]]
- [[React生态Router与Query]]
- [[React Server Components]]
- [[状态管理方案]]
- [[Next.js与SSR]]
- [[Webpack与Vite]]
- React 官方文档：https://react.dev/

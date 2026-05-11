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

**核心概念：**

- **JSX**：JavaScript 的语法扩展，编译为 `React.createElement()` 调用
- **虚拟 DOM**：内存中的 DOM 树表示，通过 diff 算法最小化真实 DOM 操作
- **组件**：函数组件（推荐）和类组件，接收 props 返回 UI
- **Hooks**：`useState`、`useEffect`、`useRef` 等，让函数组件拥有状态和副作用

**核心架构：**

- 设计理念：UI = f(state)，单向数据流
- 核心模块：Fiber 调度器、Reconciler（协调器）、Renderer（渲染器）
- 数据流：State/Props 变化 → Reconciler diff → Renderer 更新 DOM

**插件生态：**

- 官方插件：React DOM、React Native
- 社区热门：React Router（路由）、Redux/Zustand（状态管理）、React Query（数据请求）

## Why — 为什么

**适用场景：**

- 单页应用（SPA）
- 复杂交互的 UI（表单、拖拽、动画）
- 跨平台应用（React Native）
- 大型团队协作项目

**对比同类框架：**

| 维度 | React | Vue | Svelte |
|------|-------|-----|--------|
| 性能 | 高（虚拟 DOM diff） | 高（响应式更新） | 极高（编译时优化） |
| 生态 | 最丰富 | 丰富 | 中等 |
| 学习曲线 | 中（Hooks 心智模型） | 低 | 低 |
| 灵活性 | 极高 | 中（约定明确） | 中 |

**优缺点：**

- ✅ 优点：
  - 生态庞大，组件库丰富
  - 函数式编程思想，UI 可预测
  - 跨平台（Web/Native/SSR）
  - 大厂维护，长期稳定
- ❌ 缺点：
  - Hooks 心智负担重（依赖数组、闭包陷阱）
  - 没有官方路由/状态管理（需选型）
  - 重新渲染优化需要手动 memo/useMemo

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

**副作用与清理：**

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

**性能优化：**

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

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 闭包陷阱 | useEffect 中引用了旧的 state 值 | 依赖数组包含所有使用的变量，或用 ref 存最新值 |
| 无限渲染 | useEffect 依赖导致循环更新 | 检查依赖是否在 effect 内被修改 |
| 子组件不必要渲染 | 父组件重渲染导致所有子组件重渲染 | `React.memo` + `useCallback`/`useMemo` |
| 状态更新批处理 | React 18 自动批处理可能不符合预期 | 使用 `flushSync` 强制同步更新 |

### 最佳实践

- 依赖数组宁多勿少，用 ESLint `exhaustive-deps` 规则
- 状态尽量下放，避免顶层巨型 state
- 复杂状态用 `useReducer` 替代多个 `useState`
- 数据请求用 React Query/SWR，手写 useEffect 易出 bug

## 面试题

**Q1: Virtual DOM 的工作原理是什么？**
> Virtual DOM 是内存中的 JS 对象树，是对真实 DOM 的轻量表示。状态变化时先生成新的 VNode 树，通过 diff 算法与旧树比较找出最小差异，再批量更新真实 DOM，避免频繁操作 DOM 带来的性能开销。

**Q2: React 的 diff 算法有哪些优化策略？**
> 三大策略：① 同层比较，不跨层级；② 不同类型的节点直接替换，不做递归；③ 通过 key 识别同层节点的移动/复用。这使得 diff 从 O(n³) 降为 O(n)。

**Q3: 列表渲染中 key 的作用是什么？为什么不建议用 index 作为 key？**
> key 帮助 diff 算法识别哪些节点可复用。用 index 作 key 时，若列表发生插入/删除/重排，元素顺序变化但 key 不变，导致 diff 错误复用节点，可能出现渲染错乱或状态错位。

**Q4: Fiber 架构解决了什么问题？**
> 旧版 React 递归渲染不可中断，长任务会阻塞主线程导致卡顿。Fiber 将渲染拆分为多个小单元（fiber node），利用空闲时间片逐帧执行，实现可中断、可恢复的增量渲染，优先处理高优先级更新（如用户输入）。

**Q5: React 18 的并发模式（Concurrent Mode）有什么意义？**
> 并发模式让 React 可以同时准备多个版本的 UI，在渲染过程中可以暂停、放弃或重新执行渲染，确保紧急更新（输入、点击）不被慢速渲染阻塞，提升交互响应性。

---

**相关链接：**
- [[Vue核心]]
- [[Webpack与Vite]]
- React 官方文档：https://react.dev/

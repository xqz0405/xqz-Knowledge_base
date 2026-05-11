---
tags:
  - Web前端
  - React
  - Hooks
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# React Hooks详解

## What — 是什么

> Hooks 是 React 16.8 引入的函数，让函数组件拥有状态管理、副作用处理和性能优化能力，替代类组件的 this/state 生命周期。

**核心概念：**

- **useState**：`[value, setter] = useState(initial)` 声明状态
- **useEffect**：`useEffect(fn, deps)` 处理副作用（数据请求、订阅、DOM 操作）
- **useRef**：`useRef(initial)` 持有可变引用，不触发渲染
- **useMemo/useCallback**：缓存计算结果/函数，避免不必要渲染
- **useContext**：读取 Context 值，跨层级传递数据
- **自定义 Hook**：`useXxx` 命名的函数，封装可复用逻辑

**关键特性：**

- Hooks 每次渲染按顺序调用，**不能在条件/循环中使用**
- `useEffect` 依赖数组为 `[]` 等价于 `componentDidMount`
- `useRef` 的 `.current` 变化不触发渲染
- `useCallback(fn, deps)` = `useMemo(() => fn, deps)`

## Why — 为什么

**适用场景：**

- 所有 React 函数组件
- 逻辑复用（替代 HOC 和 renderProps）
- 副作用管理

**对比替代方案：**

| 维度 | Hooks | 类组件生命周期 | HOC/RenderProps |
|------|-------|-------------|----------------|
| 逻辑复用 | 自定义 Hook | mixin（已废弃） | HOC/RenderProps |
| 代码量 | 少 | 多 | 多 |
| 心智模型 | 按功能组织 | 按生命周期组织 | 嵌套地狱 |
| TypeScript | 友好 | 不友好 | 中等 |

**优缺点：**

- ✅ 优点：
  - 逻辑按功能聚合，而非分散在多个生命周期
  - 自定义 Hook 实现优雅复用
  - 函数组件更简洁
- ❌ 缺点：
  - 依赖数组心智负担重
  - 闭包陷阱（stale closure）
  - 不能条件调用

## How — 怎么用

### 快速上手

```jsx
function useCounter(initial = 0) {
    const [count, setCount] = useState(initial);
    const increment = useCallback(() => setCount(c => c + 1), []);
    const decrement = useCallback(() => setCount(c => c - 1), []);
    return { count, increment, decrement };
}

function App() {
    const { count, increment, decrement } = useCounter();
    return (
        <>
            <button onClick={decrement}>-</button>
            <span>{count}</span>
            <button onClick={increment}>+</button>
        </>
    );
}
```

### 代码示例

**useEffect 常见模式：**

```jsx
// 1. 数据请求
useEffect(() => {
    let cancelled = false;
    fetchUser(id).then(data => {
        if (!cancelled) setUser(data);
    });
    return () => { cancelled = true; };
}, [id]);

// 2. 事件订阅
useEffect(() => {
    const handler = (e) => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
}, []);

// 3. 同步外部状态
useEffect(() => {
    document.title = `Count: ${count}`;
}, [count]);
```

**闭包陷阱与修复：**

```jsx
// ❌ 闭包陷阱：setInterval 中 count 永远是初始值 0
useEffect(() => {
    const id = setInterval(() => {
        setCount(count + 1); // count 始终为 0
    }, 1000);
    return () => clearInterval(id);
}, []);

// ✅ 修复1：函数式更新
setCount(c => c + 1);

// ✅ 修复2：useRef 保存最新值
const countRef = useRef(count);
countRef.current = count;
setInterval(() => {
    setCount(countRef.current + 1);
}, 1000);
```

**useReducer 复杂状态：**

```jsx
const reducer = (state, action) => {
    switch (action.type) {
        case 'increment': return { count: state.count + 1 };
        case 'decrement': return { count: state.count - 1 };
        case 'reset': return { count: 0 };
        default: return state;
    }
};

function Counter() {
    const [state, dispatch] = useReducer(reducer, { count: 0 });
    return (
        <>
            <span>{state.count}</span>
            <button onClick={() => dispatch({ type: 'increment' })}>+</button>
            <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
            <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
        </>
    );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 闭包中读到旧值 | useEffect 捕获了渲染时的变量 | 函数式更新 `setX(prev => ...)` 或 useRef |
| 无限请求 | useEffect 依赖包含对象/函数引用 | 用 useMemo 包装对象，useCallback 包装函数 |
| 渲染卡顿 | 子组件不必要重渲染 | React.memo + useMemo/useCallback |
| 依赖数组遗漏 | ESLint 规则未开启 | 启用 `eslint-plugin-react-hooks` 的 `exhaustive-deps` |

### 最佳实践

- 开启 ESLint `exhaustive-deps` 规则
- 复杂状态用 `useReducer` 替代多个 `useState`
- 数据请求用 React Query/SWR，不用手写 useEffect
- 自定义 Hook 提取复用逻辑，以 `use` 开头命名

## 面试题

**Q1: 为什么 Hooks 不能在条件语句或循环中使用？**
> Hooks 依靠调用顺序建立与 Fiber 节点的映射关系。若在条件/循环中调用，顺序可能变化导致映射错位，state 和 effect 对应到错误的 Hook 实例，产生难以调试的 bug。

**Q2: useEffect 的依赖数组有什么作用？遗漏依赖会怎样？**
> 依赖数组决定 effect 何时重新执行。省略依赖会导致 effect 中读到旧的闭包值（stale closure），可能引发数据不一致或无限请求。推荐开启 ESLint `exhaustive-deps` 规则自动检测遗漏。

**Q3: 什么是 Hooks 的闭包陷阱？如何解决？**
> useEffect 内部函数捕获的是渲染时的 state 快照，后续更新不会自动同步。解决方式：① 用函数式更新 `setState(prev => ...)` 基于前值计算；② 用 useRef 保存最新值，在 effect 中读取 ref.current。

**Q4: useMemo 和 useCallback 的区别是什么？**
> `useMemo` 缓存计算结果（值），`useMemo(() => compute(a, b), [a, b])`；`useCallback` 缓存函数引用，`useCallback(fn, deps)` 等价于 `useMemo(() => fn, deps)`。两者目的相同——避免子组件不必要重渲染，只是缓存对象不同。

**Q5: useEffect 和 useLayoutEffect 有什么区别？**
> `useEffect` 在浏览器绘制后异步执行，不阻塞画面更新；`useLayoutEffect` 在 DOM 更新后、浏览器绘制前同步执行，适合需要读取/修改 DOM 布局的场景。服务端渲染时 `useLayoutEffect` 会被警告，可用 `useIsomorphicLayoutEffect` 替代。

---

**相关链接：**
- [[React核心]]
- [[Vue3响应式原理]]

---
tags:
  - Web前端
  - 框架
  - SolidJS
  - Signal
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# SolidJS 与信号式响应

## What — 什么是 SolidJS 与 Signal

SolidJS 是一个细粒度响应式前端框架，核心思想是：**用 Signal（信号）作为响应式原语，在运行时追踪依赖，只更新真正变化的 DOM 节点，完全不需要 Virtual DOM**。它使用 JSX 语法，但编译产物是直接操作真实 DOM 的命令式代码。

Signal 是一种响应式原语，封装一个可变值并在读取时自动追踪依赖、在写入时自动通知订阅者。Signal 不是 SolidJS 的发明——它来自 Knockout.js 的 observable、Svelte 的 store、Vue 的 ref 等多年演化，但 SolidJS 将 Signal 作为整个框架的核心，构建了最纯粹的信号式响应体系。

### 核心概念

| 概念 | 说明 |
|------|------|
| Signal（信号） | 响应式原语，`createSignal` 创建，读取自动追踪，写入自动通知 |
| 细粒度响应式 | 更新粒度到单个 DOM 节点/表达式，而非整个组件 |
| 无 Virtual DOM | 不做 VDOM Diff，直接操作真实 DOM |
| 编译时优化 | JSX 编译为 DOM 表达式，去掉组件函数调用的运行时开销 |
| 一次性组件函数 | 组件函数只执行一次（初始化），不因状态变化重新执行 |

### 框架对比

| 维度 | SolidJS | React | Vue 3 | Svelte |
|------|---------|-------|-------|--------|
| 响应式粒度 | Signal 级（最细） | 组件级 | 组件级（ref 可细粒度） | 编译时确定 |
| Virtual DOM | 无 | 有 | 有 | 无 |
| 模板语法 | JSX | JSX | Vue Template | Svelte 模板 |
| 更新机制 | 信号通知 → 精准 DOM 更新 | setState → VDOM Diff → Patch | Proxy → VDOM Diff → Patch | 编译时生成更新代码 |
| 运行时大小 | ~7KB | ~42KB | ~33KB | ~2KB（增量） |
| 学习曲线 | 中等（需理解 Signal） | 低 | 低 | 最低 |
| 组件重渲染 | 从不重渲染（只更新 DOM） | 整个组件重渲染 | 组件级重渲染 | 从不重渲染 |
| 响应式原语 | createSignal | useState | ref / reactive | $: / $state |
| 服务端渲染 | @solidjs/start | Next.js | Nuxt.js | SvelteKit |

相关链接：[[React核心]] [[Vue3响应式原理]] [[Svelte与编译时框架]]

---

## Why — 为什么选择 SolidJS 与 Signal

### 1. Virtual DOM Diff 仍有开销

React 和 Vue 通过 Virtual DOM Diff 找出最小变更集，但 Diff 本身是运行时开销——需要遍历虚拟树、比较新旧节点、生成 Patch。对于频繁更新的场景（动画、实时数据、拖拽），Diff 开销不可忽视。SolidJS 完全跳过 Diff：Signal 变化时直接执行对应的 DOM 更新函数，路径确定、无比较开销。

### 2. 细粒度更新只更新真正变化的部分

React 的 `setState` 触发组件重渲染——组件函数重新执行，所有子组件也可能重渲染（除非 memo）。Vue 的响应式在组件级触发重渲染，组件函数重新执行。SolidJS 中，Signal 变化只触发读取了该 Signal 的 DOM 表达式更新，组件函数不会重新执行，其他 DOM 节点完全不受影响。

```
// React: count 变化 → Counter 组件重渲染 → 所有 JSX 重新执行
// Vue: count 变化 → Counter 组件重渲染 → 模板重新执行
// SolidJS: count 变化 → 只更新读取 count() 的那个 DOM 节点
```

### 3. Signal 是响应式的未来方向

Signal 模式正在成为前端响应式的统一方向：

- **Angular Signals**（2023）：Angular 引入 `signal()` 和 `computed()`，替代 Zone.js
- **Preact Signals**：`@preact/signals` 为 Preact/React 提供 Signal 支持
- **Vue ref 细粒度**：Vue 的 `ref` 本质就是 Signal，`effectScope` 提供细粒度控制
- **TC39 Signal 提案**：Signal 正在进入 ECMAScript 标准化流程，有望成为语言级原语
- **Svelte 5 Runes**：`$state()` 和 `$derived()` 本质是编译时的 Signal

Signal 代表了从"组件级重渲染"到"值级精准更新"的范式转变。

### 优缺点

- ✅ 优点：性能极佳、更新精准、心智模型一致（Signal 贯穿全局）、JSX 熟悉度高、无 Hook 规则限制
- ❌ 缺点：生态较小、社区资源少、JSX 有限制（不能解构 props）、调试工具不如 React 成熟

---

## How — 怎么用

### 1. 创建项目

```bash
# 使用 Vite 模板
npx degit solidjs/templates/ts my-solid-app
cd my-solid-app
npm install
npm run dev

# 或使用创建工具
npm create solid@latest my-app
```

项目结构：

```
my-solid-app/
├── src/
│   ├── App.tsx          # 根组件
│   ├── index.tsx        # 入口
│   ├── index.module.css # CSS Modules
│   └── ...
├── index.html
├── vite.config.ts
├── tsconfig.json
└── package.json
```

### 2. Signal 基础

`createSignal` 是 SolidJS 的核心原语，返回一个 getter/setter 元组。

```tsx
import { createSignal } from 'solid-js'

// 基础用法
const [count, setCount] = createSignal(0)

// 读取值（getter 函数调用）
console.log(count())  // 0

// 设置值
setCount(1)
console.log(count())  // 1

// 函数式更新（基于前值）
setCount(prev => prev + 1)
console.log(count())  // 2
```

**自动追踪依赖**：在响应式上下文（组件 JSX、createEffect、createMemo）中读取 Signal 时，SolidJS 自动追踪依赖关系。

```tsx
import { createSignal } from 'solid-js'

function Counter() {
  const [count, setCount] = createSignal(0)

  // JSX 中读取 count() 自动追踪依赖
  // count 变化时，只有 <p> 的文本节点更新，<button> 不受影响
  return (
    <div>
      <p>Count: {count()}</p>
      <p>Double: {count() * 2}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
    </div>
  )
}
```

**Signal 是函数而非值**：这是与 React `useState` 的关键区别。`count()` 是函数调用，SolidJS 通过拦截函数调用追踪依赖。这也意味着不能解构 Signal——解构后丢失追踪能力。

```tsx
// 错误：解构丢失响应性
const [count, setCount] = createSignal(0)
const c = count()  // c 是普通数字，不再有响应性

// 正确：始终通过 getter 读取
const getCount = count  // 保留 getter 函数
console.log(getCount()) // 响应式
```

### 3. 派生计算（createMemo）

`createMemo` 创建派生 Signal——只有依赖变化时才重新计算，结果被缓存。

```tsx
import { createSignal, createMemo } from 'solid-js'

function FibonacciDemo() {
  const [n, setN] = createSignal(1)

  // createMemo：惰性求值 + 缓存
  // 只有 n() 变化时才重新计算，多次读取不重复计算
  const fib = createMemo(() => {
    console.log('computing fib...')  // 只在 n 变化时打印
    if (n() <= 1) return n()
    let a = 0, b = 1
    for (let i = 2; i <= n(); i++) {
      [a, b] = [b, a + b]
    }
    return b
  })

  return (
    <div>
      <input
        type="number"
        value={n()}
        onInput={e => setN(Number(e.currentTarget.value))}
      />
      <p>fib({n()}) = {fib()}</p>
      <p>Again: {fib()}</p>  {/* 不会重新计算，直接用缓存 */}
    </div>
  )
}
```

**createMemo vs 普通派生表达式**：

```tsx
// 普通派生：每次渲染上下文执行时都计算
const double = count() * 2  // 简单计算，开销小

// createMemo：依赖不变时直接返回缓存值
const expensive = createMemo(() => heavyComputation(data()))  // 开销大，用 Memo

// 规则：简单表达式用内联派生，昂贵计算用 createMemo
```

### 4. 副作用（createEffect）

`createEffect` 在依赖变化时自动执行副作用函数，自动追踪内部读取的所有 Signal。

```tsx
import { createSignal, createEffect, onCleanup } from 'solid-js'

function Timer() {
  const [seconds, setSeconds] = createSignal(0)

  // 自动追踪 seconds 依赖
  createEffect(() => {
    console.log('Seconds:', seconds())
  })

  // 带清理的副作用
  createEffect(() => {
    const timer = setInterval(() => {
      setSeconds(prev => prev + 1)
    }, 1000)

    // onCleanup：副作用清理（类似 React useEffect 的 return）
    onCleanup(() => clearInterval(timer))
  })

  // 显式依赖声明（不常用，通常自动追踪就够了）
  // on() 可以精确控制依赖
  createEffect(() => {
    console.log('Only tracks seconds:', seconds())
  })

  return <p>Timer: {seconds()}s</p>
}
```

**createEffect 的执行时机**：

```tsx
// createEffect 是异步执行的（在 DOM 更新后）
// 如果需要同步执行，使用 createRenderEffect
import { createRenderEffect } from 'solid-js'

createRenderEffect(() => {
  // 在 DOM 更新后、浏览器绘制前同步执行
  console.log('Render effect:', count())
})

// createComputed：同步执行，立即计算
import { createComputed } from 'solid-js'

createComputed(() => {
  // 立即同步执行，适合需要同步派生的场景
  console.log('Computed:', count())
})
```

**避免无限循环**：

```tsx
// 错误：在 createEffect 中写入自己读取的 Signal
createEffect(() => {
  setCount(count() + 1)  // 读取 count → 写入 count → 触发 effect → 无限循环！
})

// 正确：副作用中只做"读→写其他Signal"或"读→DOM操作/API调用"
createEffect(() => {
  console.log(count())     // 读取
  setDocumentTitle(count()) // 写入不相关的目标
})
```

### 5. 组件与 JSX

SolidJS 的组件是函数，但与 React 有本质区别：**组件函数只执行一次**，不会因状态变化重新执行。

```tsx
import { createSignal } from 'solid-js'

// 组件函数只执行一次（初始化时）
function Greeting(props) {
  console.log('Greeting rendered')  // 只打印一次！

  const [showDetail, setShowDetail] = createSignal(false)

  return (
    <div>
      {/* props.name 是响应式的 Proxy 属性 */}
      <h1>Hello, {props.name}</h1>
      <button onClick={() => setShowDetail(!showDetail())}>
        Toggle Detail
      </button>
      {showDetail() && <p>Detail for {props.name}</p>}
    </div>
  )
}

function App() {
  const [name, setName] = createSignal('Alice')

  return (
    <div>
      <Greeting name={name()} />
      <button onClick={() => setName('Bob')}>Change Name</button>
    </div>
  )
}
```

**Props 是 Proxy 对象**：不能解构 props，否则丢失响应性。

```tsx
// 错误：解构 props 丢失响应性
function BadComponent({ name, age }) {
  return <p>{name} - {age}</p>  // 不再响应式更新！
}

// 正确：通过 props.xxx 访问
function GoodComponent(props) {
  return <p>{props.name} - {props.age}</p>
}

// 正确：用 splitProps 分割 props（保留响应性）
import { splitProps } from 'solid-js'

function Card(props) {
  const [local, rest] = splitProps(props, ['class', 'style'])
  return (
    <div class={local.class} style={local.style}>
      <div {...rest} />
    </div>
  )
}

// 正确：用 mergeProps 合并 props
import { mergeProps } from 'solid-js'

const merged = mergeProps({ color: 'blue' }, props)
```

**Children 处理**：

```tsx
import { children, createSignal } from 'solid-js'

function Wrapper(props) {
  // children() 创建一个响应式的 children Signal
  const resolved = children(() => props.children)

  return (
    <div class="wrapper">
      <h2>Children:</h2>
      {resolved()}
    </div>
  )
}

// 使用
function App() {
  const [show, setShow] = createSignal(true)
  return (
    <Wrapper>
      {show() && <p>Conditional child</p>}
      <p>Always shown</p>
    </Wrapper>
  )
}
```

**无 VDOM 的直接 DOM 操作**：SolidJS 编译 JSX 为真实 DOM 表达式，可安全使用 ref 直接操作 DOM。

```tsx
import { createSignal, onMount } from 'solid-js'

function Canvas() {
  let canvasRef  // 直接 DOM 引用，不需要 useRef

  onMount(() => {
    const ctx = canvasRef.getContext('2d')
    ctx.fillStyle = 'red'
    ctx.fillRect(10, 10, 100, 100)
  })

  return <canvas ref={canvasRef} width={300} height={200} />
}
```

### 6. 控制流

SolidJS 提供专用控制流组件，**不能**用 `map` 和三元表达式替代——因为 SolidJS 组件只执行一次，控制流组件管理 DOM 节点的创建和销毁。

```tsx
import { Show, For, Switch, Match, Dynamic } from 'solid-js'

function ControlFlowDemo() {
  const [items, setItems] = createSignal([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' },
  ])
  const [loggedIn, setLoggedIn] = createSignal(false)
  const [status, setStatus] = createSignal<'loading' | 'success' | 'error'>('loading')

  return (
    <div>
      {/* Show：条件渲染 */}
      <Show
        when={loggedIn()}
        fallback={<p>Please log in</p>}
      >
        <p>Welcome back!</p>
      </Show>

      {/* For：列表渲染（带 key） */}
      <For each={items()}>
        {(item, index) => (
          <div>
            #{index()} - {item.name}
          </div>
        )}
      </For>

      {/* Switch/Match：多条件匹配 */}
      <Switch fallback={<p>Unknown status</p>}>
        <Match when={status() === 'loading'}>
          <p>Loading...</p>
        </Match>
        <Match when={status() === 'success'}>
          <p>Success!</p>
        </Match>
        <Match when={status() === 'error'}>
          <p>Error occurred</p>
        </Match>
      </Switch>

      {/* Dynamic：动态组件渲染 */}
      <Dynamic component={loggedIn() ? UserPanel : GuestPanel} />
    </div>
  )
}
```

**为什么不能用 `map` 和三元表达式**：

```tsx
// 错误方式 1：用 map 渲染列表
// 问题：items 变化时，整个 map 重新执行，所有 DOM 节点重建
{items().map(item => <div>{item.name}</div>)}

// 正确：<For> 组件追踪每个 item，只更新/创建/删除变化的项
<For each={items()}>
  {(item) => <div>{item.name}</div>}
</For>

// 错误方式 2：三元表达式条件渲染
// 问题：条件切换时，旧节点被销毁、新节点被创建，无法保留状态
{loggedIn() ? <UserPanel /> : <GuestPanel />}

// 正确：<Show> 组件可以正确管理 DOM 节点的挂载/卸载
<Show when={loggedIn()} fallback={<GuestPanel />}>
  <UserPanel />
</Show>

// <For> 的 key 稳定性
<For each={items()} fallback={<p>No items</p>}>
  {(item) => <ItemCard id={item.id} name={item.name} />}
</For>
// item.id 是默认 key，确保列表更新时复用 DOM 而非全部重建
```

**Index 组件**：当需要索引作为 key 时使用 `<Index>`。

```tsx
import { Index } from 'solid-js'

// Index 按索引追踪，适合项不需要唯一 key 的场景
<Index each={items()}>
  {(item, index) => (
    <div>
      {index}: {item().name}  {/* 注意：Index 中 item 是 getter */}
    </div>
  )}
</Index>
```

### 7. Store 深度响应

`createStore` 创建深度响应式对象，嵌套属性变化也能触发精准更新。

```tsx
import { createStore, produce, reconcile } from 'solid-js/store'

function TodoApp() {
  // 创建 Store
  const [state, setState] = createStore({
    user: { name: 'Alice', age: 25 },
    todos: [
      { id: 1, text: 'Learn SolidJS', done: false },
      { id: 2, text: 'Build an app', done: false },
    ],
    filter: 'all' as 'all' | 'active' | 'completed',
  })

  // 深层属性更新
  setState('user', 'name', 'Bob')         // state.user.name = 'Bob'
  setState('user', 'age', prev => prev + 1) // 函数式更新

  // 数组操作
  setState('todos', state.todos.length, {
    id: 3, text: 'Ship it', done: false
  })  // 添加新项

  setState('todos', 0, 'done', true)  // 更新第一项的 done

  // produce：类似 Immer 的语法
  setState(produce(s => {
    s.todos.push({ id: 4, text: 'Deploy', done: false })
    s.todos[0].done = true
    s.filter = 'active'
  }))

  // reconcile：整体替换并最小化更新
  const newData = await fetchTodos()
  setState(reconcile(newData))  // 智能对比新旧数据，只更新变化的部分

  return (
    <div>
      <h1>{state.user.name}'s Todos</h1>
      <For each={state.todos}>
        {(todo) => (
          <div style={{ 'text-decoration': todo.done ? 'line-through' : 'none' }}>
            {todo.text}
          </div>
        )}
      </For>
    </div>
  )
}
```

**Store vs Signal 选择**：

```tsx
// Signal：适合独立的基础类型值
const [count, setCount] = createSignal(0)
const [name, setName] = createSignal('Alice')

// Store：适合关联的复合数据、嵌套对象、数组
const [user, setUser] = createStore({ name: 'Alice', address: { city: 'NYC' } })

// 混合使用：顶层用 Signal 管理切换，Store 管理数据
const [currentId, setCurrentId] = createSignal(1)
const [users, setUsers] = createStore({})

// 规则：
// 1. 基础类型 → createSignal
// 2. 对象/数组 → createStore
// 3. 需要深层更新 → createStore
// 4. 需要整体替换 → createSignal + reconcile 或 createStore + reconcile
```

**Store 的嵌套更新路径**：

```tsx
const [state, setState] = createStore({
  team: {
    members: [
      { id: 1, name: 'Alice', skills: ['JS', 'TS'] },
      { id: 2, name: 'Bob', skills: ['Python'] },
    ],
  },
})

// 路径语法
setState('team', 'members', 0, 'name', 'Alice Wang')
setState('team', 'members', 1, 'skills', 1, 'Go')  // 添加 Bob 的第二技能

// 函数式路径
setState(
  'team',
  'members',
  m => m.id === 1,
  'name',
  'Alice W.'
)
```

### 8. Context 与依赖注入

```tsx
import { createContext, useContext } from 'solid-js'

// 创建 Context
const ThemeContext = createContext<{ color: string; size: string }>()

// 提供者组件
function ThemeProvider(props) {
  const [theme, setTheme] = createStore({
    color: 'blue',
    size: 'medium',
  })

  // 传递响应式 Store 到 Context
  return (
    <ThemeContext.Provider value={theme}>
      {props.children}
    </ThemeContext.Provider>
  )
}

// 消费者组件
function ThemedButton() {
  const theme = useContext(ThemeContext)

  // theme 是响应式的 Store
  return (
    <button style={{ color: theme.color, 'font-size': theme.size }}>
      Themed Button
    </button>
  )
}

// 使用
function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  )
}
```

**Context 与 Signal 组合**：

```tsx
// 将 Signal 通过 Context 传递，实现跨组件状态共享
const CounterContext = createContext<{
  count: () => number
  increment: () => void
}>()

function CounterProvider(props) {
  const [count, setCount] = createSignal(0)
  const increment = () => setCount(prev => prev + 1)

  return (
    <CounterContext.Provider value={{ count, increment }}>
      {props.children}
    </CounterContext.Provider>
  )
}

function DeepChild() {
  const { count, increment } = useContext(CounterContext)
  return (
    <button onClick={increment}>
      Count: {count()}
    </button>
  )
}
```

### 9. 资源与数据获取

`createResource` 是 SolidJS 的异步数据获取原语，配合 Suspense 和 ErrorBoundary 使用。

```tsx
import { createResource, Suspense, ErrorBoundary, Show } from 'solid-js'

// 数据获取函数
async function fetchUser(id: number) {
  const res = await fetch(`/api/users/${id}`)
  if (!res.ok) throw new Error('Failed to fetch user')
  return res.json()
}

function UserProfile(props) {
  // createResource：Signal + async
  const [user, { refetch, mutate }] = createResource(
    () => props.id,  // 源 Signal（变化时自动重新获取）
    fetchUser        // 数据获取函数
  )

  // mutate：乐观更新
  const handleNameChange = (newName: string) => {
    mutate(u => ({ ...u, name: newName }))  // 立即更新本地数据
  }

  return (
    <div>
      <Show when={user.loading}>
        <p>Loading...</p>
      </Show>
      <Show when={user.error}>
        <p>Error: {user.error.message}</p>
      </Show>
      <Show when={user()}>
        <div>
          <h1>{user().name}</h1>
          <p>{user().email}</p>
          <button onClick={() => refetch()}>Refresh</button>
        </div>
      </Show>
    </div>
  )
}
```

**Suspense + ErrorBoundary**：

```tsx
function App() {
  return (
    <ErrorBoundary
      fallback={(err, reset) => (
        <div>
          <p>Something went wrong: {err.message}</p>
          <button onClick={reset}>Retry</button>
        </div>
      )}
    >
      <Suspense
        fallback={
          <div class="suspense-loading">
            <div class="spinner" />
            <p>Loading profile...</p>
          </div>
        }
      >
        <UserProfile id={1} />
        <UserPosts id={1} />
      </Suspense>
    </ErrorBoundary>
  )
}
```

**多个 Resource 并行加载**：

```tsx
function Dashboard() {
  const [stats] = createResource(fetchStats)
  const [recent] = createResource(fetchRecentActivity)
  const [notifications] = createResource(fetchNotifications)

  // Suspense 等待所有 Resource 就绪
  return (
    <Suspense fallback={<p>Loading dashboard...</p>}>
      <StatsPanel data={stats()} />
      <ActivityList items={recent()} />
      <NotificationList items={notifications()} />
    </Suspense>
  )
}
```

### 10. 路由

`@solidjs/router` 是 SolidJS 官方路由库，支持嵌套路由、数据路由、动态路由。

```bash
npm install @solidjs/router
```

```tsx
import { Router, Route, Link, useParams, useSearchParams } from '@solidjs/router'

// 页面组件
function Home() {
  return <h1>Home</h1>
}

function About() {
  return <h1>About</h1>
}

// 动态路由
function UserPage() {
  const params = useParams()  // 响应式 params
  const [user] = createResource(() => params.id, fetchUser)

  return (
    <Suspense fallback={<p>Loading user...</p>}>
      <h1>User: {user()?.name}</h1>
    </Suspense>
  )
}

// 查询参数
function Search() {
  const [searchParams, setSearchParams] = useSearchParams()
  return (
    <div>
      <input
        value={searchParams.q || ''}
        onInput={e => setSearchParams({ q: e.currentTarget.value })}
      />
      <p>Searching: {searchParams.q}</p>
    </div>
  )
}

// 路由配置
function App() {
  return (
    <Router root={Layout}>
      <Route path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/users/:id" component={UserPage} />
      <Route path="/search" component={Search} />
    </Router>
  )
}

// 布局组件
function Layout(props) {
  return (
    <div>
      <nav>
        <Link href="/">Home</Link>
        <Link href="/about">About</Link>
      </nav>
      <main>{props.children}</main>
    </div>
  )
}
```

**数据路由（Data Router）**：

```tsx
import { Route } from '@solidjs/router'

// routeData 在导航时预加载，与 Suspense 配合
function UserPage() {
  const user = createRouteData(async () => {
    const res = await fetch('/api/user')
    return res.json()
  })

  return (
    <Suspense fallback={<p>Loading...</p>}>
      <h1>{user()?.name}</h1>
    </Suspense>
  )
}

// 路由配置中声明 data
<Route
  path="/users/:id"
  component={UserPage}
  loadData={async ({ params }) => {
    const user = await fetchUser(params.id)
    return { user }
  }}
/>
```

### 11. 服务端渲染

`@solidjs/start` 是 SolidJS 的全栈框架（类似 Next.js/Nuxt.js），支持 SSR 和 SSG。

```bash
npx create-solid@latest
# 选择 SSR 模板
```

```tsx
// src/entry-client.tsx — 客户端入口
import { hydrate } from 'solid-js/web'
import { StartClient } from '@solidjs/start'
import { router } from './router'

hydrate(() => <StartClient router={router} />, document.getElementById('app')!)

// src/entry-server.tsx — 服务端入口
import { renderToString } from 'solid-js/web'
import { StartServer } from '@solidjs/start'

export default function ({ url }) {
  const html = renderToString(() => <StartServer url={url} />)
  return html
}
```

**SSR vs SSG 模式**：

```tsx
// 路由级渲染模式控制
// src/routes/index.tsx
export const ssr = true      // 启用 SSR（默认）
export const prerender = true // 预渲染为静态 HTML（SSG）

// API Routes（类似 Next.js API Routes）
// src/routes/api/users.ts
export function GET() {
  return new Response(JSON.stringify([{ id: 1, name: 'Alice' }]), {
    headers: { 'Content-Type': 'application/json' },
  })
}

// Server Functions
// src/routes/todos.tsx
import { createServerData$ } from 'solid-start/server'

function Todos() {
  const todos = createServerData$(async () => {
    // 此函数只在服务端执行
    return await db.todo.findMany()
  })

  return (
    <Suspense fallback={<p>Loading...</p>}>
      <For each={todos()}>
        {(todo) => <div>{todo.text}</div>}
      </For>
    </Suspense>
  )
}
```

**SSR 中的 isServer 判断**：

```tsx
import { isServer } from 'solid-js/web'

if (isServer) {
  // 服务端逻辑（访问数据库等）
} else {
  // 客户端逻辑（访问浏览器 API 等）
}
```

### 12. 信号模式的跨框架趋势

Signal 不只是 SolidJS 的特性，它正在成为整个前端生态的统一方向：

**Angular Signals（2023+）**：

```ts
// Angular 16+ 引入 Signals，替代 Zone.js
import { signal, computed, effect } from '@angular/core'

@Component({})
class MyComponent {
  count = signal(0)
  double = computed(() => this.count() * 2)

  constructor() {
    effect(() => console.log('Count:', this.count()))
  }

  increment() {
    this.count.update(v => v + 1)
  }
}
```

**Preact Signals**：

```tsx
// @preact/signals — 可在 Preact 和 React 中使用
import { signal, computed, effect } from '@preact/signals'

const count = signal(0)
const double = computed(() => count.value * 2)

effect(() => console.log('Count:', count.value))

// React 集成
import { useSignal } from '@preact/signals-react'

function Counter() {
  const count = useSignal(0)
  return <button onClick={() => count.value++}>{count.value}</button>
}
```

**Vue ref 的细粒度本质**：

```ts
// Vue 的 ref 本质就是 Signal
import { ref, computed, watchEffect } from 'vue'

const count = ref(0)                   // Signal
const double = computed(() => count.value * 2)  // createMemo

watchEffect(() => {                     // createEffect
  console.log('Count:', count.value)
})

// Vue 3.3+ 的 effectScope 提供细粒度控制
import { effectScope, onScopeDispose } from 'vue'

const scope = effectScope(() => {
  const state = ref(0)
  watchEffect(() => console.log(state.value))
  onScopeDispose(() => console.log('cleaned'))
})

scope.run()   // 激活
scope.stop()  // 清理所有 effect
```

**TC39 Signal 提案**：

```ts
// TC39 Signal 提案（Stage 1-2，仍在推进中）
// 目标：成为 JavaScript 语言级标准

// 提案中的 API（可能变化）
const count = new Signal.State(0)                // 可写信号
const double = new Signal.Computed(() => count.get() * 2)  // 计算信号

// 自动追踪 effect
effect(() => {
  console.log('Count:', count.get())
})

count.set(1)  // 触发 effect

// 意义：
// 1. 跨框架共享响应式基础设施
// 2. 框架只需实现"Signal → DOM 更新"的绑定层
// 3. DevTools 可以原生理解 Signal 调试
// 4. 引擎级优化（V8 可以针对 Signal 做 JIT 优化）
```

---

## 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 解构 Signal 丢失响应性 | `const val = signal()` 取出的是值不是 getter | 始终通过 `signal()` 函数调用读取 |
| 解构 props 丢失响应性 | props 是 Proxy 对象 | 使用 `props.xxx` 或 `splitProps` |
| `map()` 列表全量重建 | `map` 不是响应式控制流 | 使用 `<For>` 组件 |
| createEffect 无限循环 | effect 中读写同一个 Signal | effect 中只读取，不反向写入 |
| Store 直接赋值不触发更新 | `state.obj = newObj` 绕过 Proxy | 使用 `setState` 更新 |
| 组件不重渲染不是 bug | SolidJS 组件只执行一次是设计 | 组件内用 Signal 驱动 DOM 更新 |
| 异步回调中 Signal 不追踪 | 自动追踪只在同步执行时有效 | 在同步作用域读取 Signal，传值给回调 |
| JSX 中 `{...props}` 展开有问题 | props 是 Proxy，展开行为特殊 | 使用 `splitProps` 分割后展开 |

---

## 面试题

### 1. Signal 的原理是什么？它是如何实现自动依赖追踪的？

**答**：Signal 内部维护三个核心数据：当前值（value）、订阅者集合（subscribers）、全局追踪栈（currentObserver）。依赖追踪的机制是：当 Signal 的 getter 被调用时，检查全局追踪栈上是否有正在执行的副作用函数（effect），如果有，将这个 effect 添加到 Signal 的 subscribers 集合中，同时将 Signal 添加到 effect 的 dependencies 集合中——这就是"读取即追踪"。当 Signal 的 setter 被调用时，遍历 subscribers 集合，按优先级（computed > effect）依次执行所有订阅者——这就是"写入即通知"。这个机制与 Vue 3 的 `ref` + `effect` 原理类似，但 SolidJS 在框架层全量使用，做到了 DOM 级别的细粒度更新。

---

### 2. 细粒度响应式与 VDOM Diff 的核心区别是什么？各自优劣？

**答**：细粒度响应式（SolidJS Signal）在值变化时直接通知对应的 DOM 更新函数执行，路径是确定的：`Signal.set() → 通知订阅者 → 执行 DOM 操作`，没有比较过程。VDOM Diff（React/Vue）在状态变化时重新构建虚拟 DOM 树，然后 Diff 新旧树找出差异，最后 Patch 到真实 DOM，路径是：`setState → 构建新 VDOM → Diff → Patch`。细粒度响应式的优势是更新效率高（无 Diff 开销，只更新真正变化的节点），劣势是初始化时需要建立依赖追踪的订阅关系（少量开销）。VDOM Diff 的优势是通用性强（不关心数据如何变化，统一 Diff），劣势是每次更新都有构建+比较的开销。在频繁更新场景（动画、实时数据），细粒度响应式优势明显；在大型列表全量重渲染场景，VDOM Diff 可能更简单高效。

---

### 3. SolidJS 为什么快？从编译和运行时两个角度分析。

**答**：编译层面：(1) JSX 编译为真实 DOM 创建表达式——`<div class="x">{name()}</div>` 编译为 `const _el$ = document.createElement("div"); _el$.className = "x"; _el$.textContent = name();`，没有组件函数调用的运行时开销；(2) 模板分析——编译器识别静态部分和动态部分，静态部分只创建一次，动态部分只更新变化的表达式。运行时层面：(1) 无 VDOM Diff——Signal 变化直接执行 DOM 更新，跳过构建+比较阶段；(2) 组件只执行一次——初始化时建立 Signal → DOM 的订阅关系，之后只执行最小粒度的 DOM 更新，不重新执行组件函数；(3) 细粒度更新——一个 Signal 只通知读取它的 DOM 表达式，其他 DOM 节点完全不受影响。综合来看，SolidJS 同时消除了编译冗余（组件函数调用、VDOM 创建）和运行时冗余（Diff、组件重渲染）。

---

### 4. createMemo 和 createEffect 的区别是什么？分别在什么场景使用？

**答**：`createMemo` 创建派生 Signal——返回一个只读 getter，值被缓存，依赖不变时直接返回缓存。`createEffect` 创建副作用——不返回值，依赖变化时执行副作用（DOM 操作、日志、API 调用等）。区别：(1) **用途**——`createMemo` 用于计算派生数据，`createEffect` 用于执行副作用；(2) **返回值**——`createMemo` 返回 getter 函数，`createEffect` 返回清理函数；(3) **执行时机**——`createMemo` 同步执行（立即计算），`createEffect` 延迟执行（DOM 更新后）；(4) **缓存**——`createMemo` 有缓存（依赖不变不重算），`createEffect` 无缓存（每次都执行）。使用原则：需要用计算结果 → `createMemo`；需要做副作用 → `createEffect`；简单表达式直接内联（如 `count() * 2`），无需 `createMemo`。

---

### 5. Store 的 produce 有什么用途？与 Immer 的关系是什么？

**答**：`produce` 让你在 `setState` 中以可变语法编写更新逻辑，同时保持不可变的语义——与 Immer 的 `produce` 原理一致：在内部创建一个 Proxy 包装的草稿对象，允许直接修改草稿，修改完成后生成新的不可变状态。区别在于：Immer 是独立库，生成全新的不可变对象；SolidJS 的 `produce` 与 Store 的响应式系统深度集成——`setState(produce(s => { s.todos.push(newTodo) }))` 中，SolidJS 只会触发 `todos` 数组相关订阅者的更新，而不是整个 Store 的订阅者。`produce` 适合批量更新 Store 中的多个嵌套属性，比多次调用 `setState('a', 'b', 'c', value)` 更简洁。注意：`produce` 只能在 `setState` 中使用，不能独立使用。

---

### 6. SolidJS 的控制流组件（Show/For/Switch）存在意义是什么？为什么不能用 map 和三元表达式？

**答**：核心原因是 SolidJS 的组件只执行一次。在 React 中，`{list.map(item => <Item />)}` 每次渲染都重新执行 map，这没问题因为 React 本来就重渲染。但 SolidJS 组件函数只执行一次——如果用 `map`，列表变化时 map 不会重新执行，DOM 不会更新。`<For>` 组件内部管理响应式订阅，当 `each` 的 Signal 变化时，只创建/更新/销毁变化的项对应的 DOM 节点。同样，三元表达式 `condition ? <A /> : <B />` 在初始化时求值后就固定了，条件变化不会切换 DOM——`<Show>` 组件内部订阅 `when` 的 Signal，条件变化时正确挂载/卸载 DOM。控制流组件本质是"响应式订阅 + DOM 生命周期管理"的封装，是 SolidJS 无 VDOM 架构的必然选择。

---

### 7. TC39 Signal 提案的意义是什么？对前端框架生态有什么影响？

**答**：TC39 Signal 提案旨在将 Signal 成为 JavaScript 语言级标准原语，类似于 Promise 对异步的标准化。意义：(1) **跨框架共享基础设施**——目前每个框架各自实现 Signal（SolidJS、Angular、Vue、Preact），提案标准化后框架可以复用同一套底层实现，减少重复工作；(2) **互操作性**——不同框架的组件可以共享 Signal 数据，Signal 作为通用接口连接不同生态；(3) **引擎级优化**——Signal 成为语言标准后，V8 等引擎可以做专门的 JIT 优化（如更高效的订阅通知机制），所有框架受益；(4) **DevTools 原生支持**——浏览器 DevTools 可以原生展示 Signal 的值、依赖关系、更新历史，调试体验大幅提升；(5) **降低框架开发门槛**——新框架只需实现"Signal → DOM 更新"的绑定层，不用重新实现响应式核心。潜在风险：标准化可能限制创新（提案是最低公共集），框架特有的优化可能无法纳入标准。

---

### 8. SolidJS 和 React 的 API 看起来相似（JSX、组件、Hooks），但行为完全不同，请详细对比。

**答**：

| 维度 | React | SolidJS |
|------|-------|---------|
| 组件执行 | 每次状态变化重新执行 | 只执行一次（初始化） |
| 状态原语 | `useState` 返回 `[value, setter]` | `createSignal` 返回 `[getter, setter]` |
| 状态读取 | 值类型 `count` | 函数调用 `count()` |
| 副作用 | `useEffect(cb, deps)` 需手动依赖数组 | `createEffect(cb)` 自动追踪依赖 |
| 依赖数组 | 手动声明，容易出错 | 无需声明，自动追踪 |
| Hooks 规则 | 必须在组件顶层调用，不能条件调用 | 无此限制（Signal 可在任何地方创建） |
| 闭包陷阱 | `useEffect` 中读取旧值（stale closure） | 无闭包陷阱（getter 始终返回最新值） |
| 列表渲染 | `map()` + `key` | `<For each={}>` 组件 |
| 条件渲染 | 三元表达式 / `&&` | `<Show when={}>` 组件 |
| Props | 普通对象，可解构 | Proxy 对象，不能解构 |
| Ref | `useRef` 返回 `{ current: value }` | 直接 `let ref`，赋值给 `ref={ref}` |
| 子组件通信 | `useImperativeHandle` + `forwardRef` | 直接使用 `ref` 调用子组件方法 |
| 更新粒度 | 组件级 | DOM 节点级 |
| 渲染模型 | Pull（调度 → 渲染 → Diff → Patch） | Push（Signal 通知 → 直接 DOM 更新） |

最根本的区别是渲染模型：React 是 Pull 模型——状态变化后调度重渲染，组件函数重新执行，VDOM Diff 后 Patch；SolidJS 是 Push 模型——Signal 变化直接通知订阅者执行 DOM 更新。这导致 API 虽然看起来相似（都是 JSX + 函数组件 + Hook 风格 API），但心智模型完全不同：React 是"声明 UI = f(state)，框架负责重渲染"，SolidJS 是"初始化时建立响应式连接，之后精准推送更新"。

---

## 相关链接

- [[React核心]]
- [[Vue3响应式原理]]
- [[Svelte与编译时框架]]
- [SolidJS 官方文档](https://www.solidjs.com/)
- [SolidJS GitHub](https://github.com/solidjs/solid)
- [Solid Playground](https://playground.solidjs.com/)
- [TC39 Signal 提案](https://github.com/tc39/proposal-signals)
- [Angular Signals](https://angular.dev/guide/signals)
- [Preact Signals](https://preactjs.com/guide/v10/signals/)

---
tags:
  - Web前端
  - Vue
  - 框架
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Vue核心

## What — 是什么

> Vue 是渐进式 JavaScript 框架，核心是响应式数据绑定和组件化系统，通过 Proxy 拦截实现数据变化自动驱动视图更新。

### 核心设计理念

- **渐进式框架**：Vue 被设计为自底向上逐层应用。核心库只关注视图层，通过搭配 Vue Router、Pinia 等官方生态逐步扩展为完整 SPA。你可以在现有页面中用一行 `<script>` 引入 Vue，也可以用 Vite 构建大型单页应用
- **声明式渲染**：通过模板语法声明式地将数据绑定到 DOM，Vue 自动追踪依赖并在数据变化时更新视图，开发者只需关注"数据是什么"而非"如何操作 DOM"
- **组件化系统**：将 UI 拆分为独立、可复用的组件，每个组件封装自己的模板、逻辑和样式，通过 Props/Emit/Slot 等机制组合

### 核心模块

- **响应式系统（Reactivity）**：Vue 3 用 `Proxy` 拦截对象操作，Vue 2 用 `Object.defineProperty`
- **编译器（Compiler）**：将模板编译为渲染函数，同时执行静态分析优化
- **渲染器（Renderer）**：生成 VNode 树，通过 diff 算法最小化 DOM 操作

**数据流：**

```
数据变化 → 响应式追踪（Proxy get/set）→ 调度器（nextTick 批量更新）→ diff → DOM patch
```

### Vue 3 完整 API 分类

**响应式 API：**

| API | 用途 | 说明 |
|-----|------|------|
| `ref()` | 基本类型/任意值响应式 | 模板自动解包，JS 需要 `.value` |
| `reactive()` | 对象类型响应式 | 返回 Proxy 代理对象 |
| `computed()` | 计算属性 | 依赖缓存，惰性求值 |
| `readonly()` | 只读代理 | 防止修改响应式数据 |
| `watch()` | 侦听器 | 显式指定数据源，可获取新旧值 |
| `watchEffect()` | 副作用侦听 | 自动追踪依赖，立即执行 |
| `shallowRef()` | 浅层 ref | 只追踪 `.value` 的替换，不深度追踪 |
| `shallowReactive()` | 浅层 reactive | 只代理根级属性 |
| `toRef()` / `toRefs()` | 解构保持响应性 | 将 reactive 属性转为 ref |
| `triggerRef()` | 强制触发 shallowRef | 手动通知浅层 ref 更新 |
| `customRef()` | 自定义 ref | 显式控制追踪与触发 |
| `markRaw()` | 标记不代理 | 让对象永远不会转为响应式 |
| `toRaw()` | 获取原始对象 | 从 Proxy 中取回原始对象 |

**组合式 API：**

| API | 用途 |
|-----|------|
| `setup()` | 组合式 API 入口函数 |
| `<script setup>` | setup 语法糖，编译时自动处理 |
| `defineProps()` | 声明组件 Props |
| `defineEmits()` | 声明组件事件 |
| `defineExpose()` | 暴露组件方法/属性给父组件 |
| `defineModel()` | 声明 v-model 绑定（3.4+） |
| `defineOptions()` | 声明组件选项（3.3+） |
| `defineSlots()` | 声明插槽类型（3.3+） |
| `useSlots()` / `useAttrs()` | 访问插槽/透传属性 |

**生命周期钩子：**

| 选项式 API | 组合式 API | 触发时机 |
|-----------|-----------|---------|
| `beforeCreate` | 不需要（setup 替代） | 实例初始化之后，数据观测之前 |
| `created` | 不需要（setup 替代） | 实例创建完成，可访问数据 |
| `beforeMount` | `onBeforeMount` | 挂载到 DOM 之前 |
| `mounted` | `onMounted` | DOM 挂载完成，可访问 DOM |
| `beforeUpdate` | `onBeforeUpdate` | 数据变化，DOM 更新之前 |
| `updated` | `onUpdated` | DOM 更新完成 |
| `beforeUnmount` | `onBeforeUnmount` | 组件卸载之前 |
| `unmounted` | `onUnmounted` | 组件卸载完成 |
| `errorCaptured` | `onErrorCaptured` | 捕获后代组件错误 |
| `renderTracked` | `onRenderTracked` | 响应式依赖被追踪时（调试） |
| `renderTriggered` | `onRenderTriggered` | 响应式依赖触发更新时（调试） |
| `activated` | `onActivated` | 被 keep-alive 缓存的组件激活 |
| `deactivated` | `onDeactivated` | 被 keep-alive 缓存的组件停用 |

**依赖注入 API：**

| API | 用途 |
|-----|------|
| `provide()` | 提供数据到后代组件 |
| `inject()` | 从祖先组件注入数据 |

**内置组件：**

| 组件 | 用途 |
|------|------|
| `<Transition>` | 单元素/组件过渡动画 |
| `<TransitionGroup>` | 列表过渡动画 |
| `<KeepAlive>` | 缓存组件实例，避免重复渲染 |
| `<Teleport>` | 将内容传送到 DOM 其他位置 |
| `<Suspense>` | 协调异步依赖（实验性） |
| `<Component>` | 动态组件渲染 |
| `<Slot>` | 插槽内容分发 |

### 选项式 API vs 组合式 API 详细对比

| 维度 | 选项式 API（Options API） | 组合式 API（Composition API） |
|------|--------------------------|------------------------------|
| 代码组织 | 按选项类型分散（data/methods/computed/watch 分开） | 按逻辑功能聚合（相关代码集中） |
| 逻辑复用 | Mixins（有命名冲突、来源不明问题） | Composable 函数（`useXxx`，清晰明确） |
| 类型推断 | 依赖 `this`，TS 推断困难 | 无 `this`，TS 原生推断 |
| 响应式机制 | 初始化时递归遍历 `data` 返回值 | 按需声明 `ref`/`reactive` |
| Tree-shaking | 整个组件无法摇树 | 组合式函数可独立摇树 |
| 学习曲线 | 低（对象结构直观） | 中（需理解 ref/reactive 概念） |
| 适用场景 | 简单组件、小型项目 | 复杂逻辑、大型项目 |
| this 访问 | 可用 `this` 访问组件实例 | 无 `this`，通过闭包引用 |

### Vue 的编译时优化

Vue 3 编译器在模板编译阶段做了大量优化，使运行时只需做最少的工作：

**1. 静态提升（Static Hoisting）**

编译器将不会变化的节点/属性提升到渲染函数之外，每次更新无需重新创建：

```javascript
// 编译前模板
// <div><span class="label">Hello</span>{{ msg }}</div>

// 编译后
const _hoisted_1 = /*#__PURE__*/_createElementVNode("span", { class: "label" }, "Hello", -1 /* HOISTED */)

function render() {
  return _createElementBlock("div", null, [_hoisted_1, _toDisplayString(_ctx.msg)])
}
```

**2. PatchFlag（补丁标记）**

编译器为动态节点标记类型，diff 时只检查标记的部分：

| Flag 值 | 含义 | diff 行为 |
|---------|------|-----------|
| `TEXT = 1` | 动态文本内容 | 只比较 textContent |
| `CLASS = 2` | 动态 class 绑定 | 只比较 class |
| `STYLE = 4` | 动态 style 绑定 | 只比较 style |
| `PROPS = 8` | 动态非 class/style 属性 | 只比较标记的 props |
| `FULL_PROPS = 16` | 有动态 key | 完整 diff 所有 props |
| `STABLE_FRAGMENT = 64` | 稳定序列片段 | 按顺序 diff 子节点 |

**3. Block Tree（块级树）**

组件模板根节点作为 Block，收集所有动态后代节点到 `dynamicChildren` 数组。diff 时跳过静态节点，只遍历动态节点列表，将 diff 复杂度从 O(模板大小) 降到 O(动态节点数量)。

`v-if`/`v-for` 会创建子 Block，分支切换时丢弃旧 Block 的动态节点集合。

**4. 缓存事件处理器**

编译器为事件处理器加缓存标记，避免每次渲染创建新函数导致子组件无谓更新：

```javascript
// 无缓存：每次渲染创建新函数
onClick: $event => (_ctx.count++)

// 缓存后：同一个函数引用
onClick: _cache[0] || (_setVNodeBinding(_cache, 0), $event => (_ctx.count++))
```

### Vue 组件系统

**组件通信机制总览：**

```
父组件
  ├── Props → 子组件（单向数据流）
  ├── 子组件 Emit → 父组件（事件通信）
  ├── Slot → 子组件（内容分发）
  ├── Provide → 后代组件 Inject（跨层级注入）
  ├── ref + defineExpose → 子组件（命令式访问）
  ├── $attrs → 子组件（属性透传）
  └── Pinia → 任意组件（全局状态）
```

**Props 规则：**
- 单向数据流：父 → 子，子不能直接修改 Props
- 类型校验：支持 String/Number/Boolean/Array/Object/Function/Symbol 等类型
- 必填与默认值：`required: true` / `default: xxx`
- 验证函数：`validator(value) { ... }`

**Emit 规则：**
- 子组件通过 `defineEmits` 声明可触发的事件
- 父组件通过 `@event-name` 监听
- 支持携带参数：`emit('update:count', newValue)`

**Slot 插槽：**
- 默认插槽：`<slot></slot>`
- 具名插槽：`<slot name="header"></slot>` + `<template #header>`
- 作用域插槽：`<slot :data="item"></slot>` + `<template #default="{ data }">`

**Provide/Inject：**
- 祖先组件 `provide(key, value)` 提供数据
- 后代组件 `inject(key, defaultValue)` 注入数据
- 跨越中间任意层级，无需逐层 Props 传递
- 默认非响应式，传入 `ref`/`reactive` 才有响应性

### 插件生态

- 官方插件：Vue Router、Pinia、VueUse
- 社区热门：Element Plus、Vuetify（UI 库）、Vue I18n（国际化）
- 开发工具：Vite（构建）、Vitest（单元测试）、Playwright（E2E）

## Why — 为什么

### 适用场景

- 中小型 SPA 项目
- 快速原型开发
- 需要模板语法的团队（比 JSX 更贴近 HTML）
- 渐进式迁移老项目
- 表单密集型应用（v-model 双向绑定天然优势）

### Vue vs React vs Svelte vs Angular 对比

| 维度 | Vue 3 | React 18 | Svelte 5 | Angular 17+ |
|------|-------|----------|----------|-------------|
| 响应式 | Proxy 自动追踪 | 手动 useState/useReducer | 编译时 Runes（$state） | Zone.js 信号（Signal） |
| 模板 | HTML 模板语法 | JSX | 增强 HTML | HTML 模板 |
| 更新策略 | 细粒度依赖追踪 + 虚拟 DOM | 虚拟 DOM diff | 编译时生成命令式更新 | 虚拟 DOM + Signal |
| 类型系统 | 渐进式 TS 支持 | 渐进式 TS 支持 | 渐进式 TS 支持 | 深度集成 TS |
| 状态管理 | Pinia（官方） | Redux/Zustand/Jotai | 内置 Store | NgRx/Signal Store |
| 路由 | Vue Router | React Router | SvelteKit | Angular Router |
| SSR | Nuxt | Next.js | SvelteKit | Angular Universal |
| 包体积 | ~33KB gzip | ~42KB gzip | 编译后极小 | ~65KB gzip |
| 学习曲线 | 低 | 中 | 低 | 高 |
| 生态丰富度 | 丰富 | 最丰富 | 中等 | 丰富（企业级） |
| 适用规模 | 中小型 → 大型 | 中大型 | 中小型 | 大型企业级 |
| 框架理念 | 渐进式 | 声明式 UI 库 | 编译时消失 | 完整平台 |

### 选项式 API vs 组合式 API 的取舍

**选择选项式 API 的场景：**
- 小型项目或简单组件，逻辑不多
- 团队新手较多，对象式写法更容易理解
- 从 Vue 2 迁移，保持代码风格一致
- 不需要复杂逻辑复用

**选择组合式 API 的场景：**
- 大型项目，组件逻辑复杂
- 需要跨组件复用逻辑（Composable）
- 深度使用 TypeScript
- 需要更好的 Tree-shaking 体积优化

**实际建议：** 新项目推荐组合式 API + `<script setup>`，这是 Vue 3 的推荐写法。简单展示型组件可以用选项式 API，但混合使用时保持一致性。

### 优缺点

- ✅ 优点：
  - 上手门槛最低，模板语法直观
  - 响应式自动追踪，无需手动优化
  - 官方全家桶（Router + Pinia）开箱即用
  - SFC 单文件组件，关注点分离清晰
  - 编译时优化减少运行时开销
  - 组合式 API 提供灵活的逻辑复用
  - 渐进式设计，可从简单脚本逐步扩展
- ❌ 缺点：
  - 响应式有边界情况（reactive 对象解构丢失响应性）
  - 大型项目不如 React 灵活
  - 国际化程度不如 React
  - 模板语法不如 JSX 表达力强（复杂渲染逻辑受限）
  - 响应式系统理解成本（ref 自动解包、shallow 深度等）

## How — 怎么用

### 快速上手

```vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <button @click="increment">+1</button>
    <p>Double: {{ double }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const count = ref(0)
const double = computed(() => count.value * 2)

function increment() {
  count.value++
}
</script>
```

### 响应式陷阱与解决

```javascript
import { reactive, ref, toRefs } from 'vue'

// ❌ 解构丢失响应性
const state = reactive({ name: 'Alice', age: 25 })
let { name } = state // name 是普通字符串，失去响应性

// ✅ 方案 1：toRefs
const { name, age } = toRefs(state) // 变成 ref，保持响应性

// ✅ 方案 2：直接用 ref
const name = ref('Alice')
```

### 组合式 API 组件完整示例

```vue
<template>
  <div class="user-profile">
    <h2>{{ user.name }}</h2>
    <p>Email: {{ user.email }}</p>
    <p>Role: {{ roleLabel }}</p>
    <button @click="updateRole('admin')">设为管理员</button>
    <button @click="updateRole('user')">设为普通用户</button>
    <p v-if="loading">加载中...</p>
    <p v-if="error" class="error">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'

// Props 定义
const props = defineProps({
  userId: { type: Number, required: true }
})

// Emit 定义
const emit = defineEmits(['role-changed'])

// 响应式状态
const user = reactive({ name: '', email: '', role: '' })
const loading = ref(false)
const error = ref(null)

// 计算属性
const roleLabel = computed(() => {
  const map = { admin: '管理员', user: '普通用户', guest: '访客' }
  return map[user.role] || '未知'
})

// 方法
async function fetchUser(id) {
  loading.value = true
  error.value = null
  try {
    const res = await fetch(`/api/users/${id}`)
    if (!res.ok) throw new Error('用户不存在')
    const data = await res.json()
    Object.assign(user, data)
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}

function updateRole(newRole) {
  user.role = newRole
  emit('role-changed', newRole)
}

// 侦听器
watch(() => props.userId, (newId) => {
  fetchUser(newId)
})

// 生命周期
onMounted(() => {
  fetchUser(props.userId)
})

// 暴露方法给父组件
defineExpose({ fetchUser, updateRole })
</script>

<style scoped>
.error { color: red; }
</style>
```

### Props 和 Emit 类型定义

```vue
<script setup>
// 方式一：运行时声明
const props = defineProps({
  title: { type: String, required: true },
  count: { type: Number, default: 0 },
  items: { type: Array, default: () => [] },
  callback: { type: Function, default: () => {} }
})

const emit = defineEmits(['update', 'delete'])

// 方式二：基于类型的声明（推荐 TS 项目）
interface Props {
  title: string
  count?: number
  items?: string[]
}

const props = defineProps<Props>()
const emit = defineEmits<{
  update: [value: string]
  delete: [id: number]
}>()

// 使用 emit
emit('update', 'new value')
emit('delete', 42)
</script>
```

### Slot 插槽（默认/具名/作用域）

```vue
<!-- Card.vue — 定义插槽的组件 -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">
        <h3>默认标题</h3>  <!-- 默认内容 -->
      </slot>
    </div>
    <div class="card-body">
      <slot></slot>  <!-- 默认插槽 -->
    </div>
    <div class="card-footer">
      <slot name="footer" :year="2026" :author="'Vue Team'">
        <!-- 作用域插槽：向父组件暴露数据 -->
        <p>&copy; 2026</p>
      </slot>
    </div>
  </div>
</template>
```

```vue
<!-- Parent.vue — 使用插槽 -->
<template>
  <Card>
    <!-- 具名插槽 -->
    <template #header>
      <h1>自定义标题</h1>
    </template>

    <!-- 默认插槽 -->
    <p>这是卡片正文内容</p>

    <!-- 作用域插槽：接收子组件暴露的数据 -->
    <template #footer="{ year, author }">
      <p>&copy; {{ year }} by {{ author }}</p>
    </template>
  </Card>
</template>
```

### Provide/Inject 依赖注入

```vue
<!-- 祖先组件：App.vue -->
<script setup>
import { provide, ref, readonly } from 'vue'

const theme = ref('dark')
const user = reactive({ name: 'Alice', role: 'admin' })

// 提供 ref/reactive 保持响应性
// 用 readonly 防止后代随意修改
provide('theme', readonly(theme))
provide('user', readonly(user))
provide('setTheme', (val) => { theme.value = val }) // 通过方法修改
</script>
```

```vue
<!-- 深层后代组件：任意层级 -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme', 'light')        // 第二个参数为默认值
const setTheme = inject('setTheme', () => {})  // 注入修改方法
const user = inject('user')

// 使用
console.log(theme.value) // 'dark'
setTheme('light')        // 通过方法安全修改
</script>
```

### 自定义指令

```vue
<script setup>
// 局部自定义指令（v- 前缀自动识别）
const vFocus = {
  mounted(el) {
    el.focus()
  }
}

// 带参数的自定义指令
const vPermission = {
  mounted(el, binding) {
    const userRole = 'user' // 从 store 获取
    const requiredRole = binding.value
    if (userRole !== requiredRole) {
      el.parentNode?.removeChild(el) // 无权限则移除元素
    }
  }
}

// 指令钩子完整列表
const vDemo = {
  created(el, binding, vnode) { /* 绑定前 */ },
  beforeMount(el, binding) { /* 挂载前 */ },
  mounted(el, binding) { /* 挂载后 */ },
  beforeUpdate(el, binding) { /* 更新前 */ },
  updated(el, binding) { /* 更新后 */ },
  beforeUnmount(el, binding) { /* 卸载前 */ },
  unmounted(el, binding) { /* 卸载后 */ }
}
</script>

<template>
  <input v-focus />
  <button v-permission="'admin'">删除用户</button>
</template>
```

```javascript
// 全局注册：main.js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

app.directive('lazy', {
  mounted(el, binding) {
    // 简易懒加载图片
    const observer = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) {
        el.src = binding.value
        observer.disconnect()
      }
    })
    observer.observe(el)
  }
})

app.mount('#app')
```

### Teleport / Suspense / Transition 内置组件

**Teleport — 传送门：**

```vue
<template>
  <div class="page">
    <!-- Modal 渲染到 body 下，避免 z-index/overflow 问题 -->
    <Teleport to="body">
      <div v-if="showModal" class="modal-overlay" @click="showModal = false">
        <div class="modal-content" @click.stop>
          <h2>弹窗标题</h2>
          <p>弹窗内容</p>
          <button @click="showModal = false">关闭</button>
        </div>
      </div>
    </Teleport>

    <!-- 多个 Teleport 可传送到同一目标，按顺序追加 -->
    <Teleport to="#modals">
      <p>另一个传送内容</p>
    </Teleport>
  </div>
</template>
```

**Suspense — 异步依赖协调：**

```vue
<template>
  <Suspense>
    <!-- 默认内容：等待异步组件解析 -->
    <template #default>
      <AsyncUserList />
    </template>
    <!-- 加载中回退内容 -->
    <template #fallback>
      <div class="loading">加载中...</div>
    </template>
  </Suspense>
</template>

<script setup>
import { defineAsyncComponent } from 'vue'

const AsyncUserList = defineAsyncComponent(() =>
  import('./UserList.vue')
)
</script>
```

**Transition — 过渡动画：**

```vue
<template>
  <!-- 单元素过渡 -->
  <Transition name="fade" mode="out-in">
    <p v-if="show" :key="current">内容 {{ current }}</p>
  </Transition>

  <!-- 列表过渡 -->
  <TransitionGroup name="list" tag="ul">
    <li v-for="item in items" :key="item.id">{{ item.text }}</li>
  </TransitionGroup>
</template>

<style>
/* fade 过渡 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* list 过渡 */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-active {
  position: absolute;
}
.list-move {
  transition: transform 0.5s ease;
}
</style>
```

### 动态组件与异步组件

```vue
<template>
  <div>
    <!-- 动态组件：通过 is 切换 -->
    <button
      v-for="tab in tabs"
      :key="tab"
      @click="currentTab = tab"
      :class="{ active: currentTab === tab }"
    >
      {{ tab }}
    </button>

    <!-- keep-alive 缓存组件实例，避免重复渲染 -->
    <KeepAlive include="TabHome,TabProfile" :max="5">
      <component :is="currentComponent" />
    </KeepAlive>
  </div>
</template>

<script setup>
import { ref, computed, defineAsyncComponent } from 'vue'
import TabHome from './TabHome.vue'

// 同步导入
const tabs = ['home', 'profile', 'settings']
const currentTab = ref('home')

// 异步组件：按需加载
const TabProfile = defineAsyncComponent({
  loader: () => import('./TabProfile.vue'),
  loadingComponent: () => import('./LoadingSpinner.vue'),
  errorComponent: () => import('./ErrorDisplay.vue'),
  delay: 200,     // 延迟显示 loading
  timeout: 5000   // 超时显示 error
})

const TabSettings = defineAsyncComponent(() => import('./TabSettings.vue'))

const componentMap = {
  home: TabHome,
  profile: TabProfile,
  settings: TabSettings
}

const currentComponent = computed(() => componentMap[currentTab.value])
</script>
```

### v-model 自定义组件

```vue
<!-- CustomInput.vue -->
<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<script setup>
// Vue 3.4+ 推荐用 defineModel
const model = defineModel() // 自动创建 props + emit
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <!-- 单个 v-model -->
  <CustomInput v-model="username" />

  <!-- 多个 v-model（命名） -->
  <UserForm v-model:first-name="first" v-model:last-name="last" />
</template>
```

```vue
<!-- UserForm.vue — 多个 v-model -->
<script setup>
const firstName = defineModel('firstName')
const lastName = defineModel('lastName')
</script>

<template>
  <input v-model="firstName" placeholder="名" />
  <input v-model="lastName" placeholder="姓" />
</template>
```

### 组合式函数（Composable）

```javascript
// composables/useFetch.js
import { ref, watchEffect, toValue } from 'vue'

export function useFetch(url) {
    const data = ref(null)
    const error = ref(null)
    const loading = ref(true)

    watchEffect(async () => {
        try {
            loading.value = true
            // toValue 支持 ref/reactive/getter/原始值
            const resolvedUrl = toValue(url)
            const res = await fetch(resolvedUrl)
            if (!res.ok) throw new Error(`HTTP ${res.status}`)
            data.value = await res.json()
        } catch (e) {
            error.value = e
        } finally {
            loading.value = false
        }
    })

    return { data, error, loading }
}
```

```vue
<script setup>
import { useFetch } from '../composables/useFetch'
import { ref } from 'vue'

const userId = ref(1)
const { data, loading, error } = useFetch(() => `/api/users/${userId.value}`)
// URL 变化时自动重新请求
</script>
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| reactive 解构丢失响应性 | 解构得到的是原始值，不再被 Proxy 追踪 | 用 `toRefs` 或直接使用 `ref` |
| ref 自动解包 | 模板中 ref 自动解包，JS 中需要 `.value` | 模板不加 `.value`，JS 必须加 |
| v-for 没有加 key | 列表渲染无法正确 diff | 始终添加唯一 `:key` |
| watch 监听 reactive 对象属性 | 直接监听 `state.count` 无效 | 用 getter `() => state.count` 或 `watchEffect` |
| nextTick 前读取 DOM | Vue 异步更新 DOM，同步获取不到最新值 | 在 `nextTick` 回调中操作 DOM |
| Teleport 目标不存在 | `to` 选择器匹配不到 DOM 元素 | 确保目标元素在 Teleport 挂载前存在 |
| v-if 与 v-for 同时使用 | 优先级冲突（v-if 优先级高于 v-for） | 用计算属性过滤或嵌套 `<template>` |
| KeepAlive 组件 name 不匹配 | include/exclude 匹配的是组件 name | 用 `defineOptions({ name: 'Xxx' })` 声明 |
| shallowRef 对象内部修改不触发 | shallowRef 只追踪 .value 引用替换 | 用 `triggerRef()` 强制触发或改用 `ref` |

### 最佳实践

- 简单值用 `ref`，对象/数组用 `reactive`
- 逻辑复用用 Composable（`useXxx`）
- 用 `<script setup>` 语法糖，更简洁
- Pinia 替代 Vuex 做状态管理
- 组件 Props 始终定义类型和默认值
- 大列表用虚拟滚动（`vue-virtual-scroller`）
- 避免在模板中使用复杂表达式，提取为 `computed`
- 使用 `defineOptions` 声明组件 name 配合 KeepAlive
- 异步组件使用 `defineAsyncComponent` 懒加载
- 样式用 `scoped` 防止全局污染，深度选择器用 `:deep()`

## 面试题

**Q1: Vue 的响应式原理是什么？**
> Vue 3 使用 Proxy 拦截对象的 get/set 操作：get 时收集当前依赖（effect），set 时触发所有依赖重新执行。Vue 2 使用 Object.defineProperty，只能拦截已有属性的读写，无法检测新增/删除属性。Proxy 的优势：可以拦截所有操作（包括属性新增/删除/has/ownKeys 等），性能更好（懒递归，访问到才代理嵌套对象），不需要 `$set`/`$delete` 补丁方法。

**Q2: v-if 和 v-show 的区别是什么？**
> `v-if` 是条件渲染，条件为 false 时 DOM 节点不创建，切换开销高；`v-show` 通过 CSS `display` 控制显隐，DOM 姫终存在，初次渲染开销高。频繁切换用 v-show，条件很少变化用 v-if。v-if 还会触发组件的创建/销毁生命周期，v-show 不会。

**Q3: computed 和 watch 的区别是什么？**
> `computed` 是计算属性，基于依赖缓存结果，依赖不变不重新计算，必须有返回值；`watch` 是侦听器，数据变化时执行副作用（如 API 请求），不需要返回值。需要派生值用 computed，需要响应变化执行操作用 watch。`watchEffect` 是 watch 的简化版，自动追踪依赖并立即执行。

**Q4: Vue 组件通信有哪些方式？**
> ① props/emit — 父子通信；② provide/inject — 跨层级传递；③ ref + defineExpose — 父访问子方法；④ Pinia — 全局状态共享；⑤ EventBus/mitt — 任意组件事件通信（不推荐滥用）；⑥ $attrs — 透传属性。

**Q5: Vue 3 Composition API 相比 Options API 有什么优势？**
> 逻辑按功能聚合而非分散在 data/methods/computed 中，相关代码集中在一处；逻辑复用更灵活，通过 Composable（useXxx）替代 mixins（避免命名冲突和来源不明）；TypeScript 类型推断更好（无 this 依赖）；Tree-shaking 更友好（未使用的组合式 API 不会被打包）。

**Q6: Vue 3 的组合式 API 和选项式 API 有什么区别？**
> 核心区别在于代码组织方式：选项式 API 按选项类型（data/methods/computed/watch）分散组织，同一功能的代码散落在不同选项中；组合式 API 按逻辑功能聚合，相关代码集中在一处。组合式 API 通过 `ref`/`reactive` 声明响应式数据，通过 Composable 函数复用逻辑，替代 mixins 避免命名冲突和来源不明问题。组合式 API 无 this 绑定，TypeScript 类型推断更自然。选项式 API 学习门槛低，适合简单组件；组合式 API 适合复杂逻辑和大型项目。

**Q7: Vue 的编译时有哪些优化？**
> Vue 3 编译器做了四项核心优化：① 静态提升 — 不会变化的节点/属性提升到渲染函数外，避免每次重新创建；② PatchFlag — 为动态节点标记类型（TEXT/CLASS/STYLE/PROPS 等），diff 时只比较标记的属性；③ Block Tree — 模板根节点作为 Block 收集动态后代节点，diff 时只遍历动态节点列表，跳过所有静态节点；④ 缓存事件处理器 — 缓存内联事件函数引用，避免每次渲染创建新函数导致子组件无谓更新。这些优化让 Vue 3 的更新性能从 O(模板大小) 降到 O(动态节点数量)。

**Q8: Provide/Inject 和 Props 有什么区别？**
> Props 是父子组件间逐层传递数据，必须每一层都显式声明，适合明确的父子关系。Provide/Inject 是跨层级传递，祖先组件 provide 数据后，任意后代组件都可以 inject 获取，中间层无需感知，适合深层嵌套组件（如主题、国际化、表单上下文）。区别：① 传递层级不同 — Props 逐层传递，Provide/Inject 跨层级；② 数据流方向 — Props 单向数据流更明确，Provide/Inject 来源不直观；③ 默认非响应式 — Provide/Inject 传入普通值不具响应性，需传入 ref/reactive 才行；④ 适用场景 — Props 适合组件接口定义，Provide/Inject 适合共享上下文。

**Q9: Vue 的 nextTick 原理是什么？**
> Vue 的 DOM 更新是异步的：数据变化后不会立即更新 DOM，而是将 watcher 推入队列，在下一个"微任务"中批量执行更新。`nextTick` 就是在这次 DOM 更新完成后执行回调。原理：Vue 内部维护一个回调队列，数据变化时将 flush 作业入队，然后通过微任务（Promise.then → MutationObserver → setImmediate → setTimeout 降级方案）异步执行队列。`nextTick(fn)` 将回调追加到队列末尾，确保在所有 DOM 更新后执行。使用场景：数据修改后需要操作更新后的 DOM，如获取元素高度、聚焦输入框等。

---

**相关链接：**
- [[React核心]]
- [[Webpack与Vite]]
- [[TypeScript核心]]
- [[Vue Router]]
- [[Pinia状态管理]]
- Vue 官方文档：https://vuejs.org/
- Vue 3 迁移指南：https://v3-migration.vuejs.org/

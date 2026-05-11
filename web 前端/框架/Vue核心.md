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

**核心概念：**

- **响应式系统**：Vue 3 用 `Proxy` 拦截对象操作，Vue 2 用 `Object.defineProperty`
- **虚拟 DOM**：模板编译为渲染函数，生成 VNode 树 diff 更新
- **组件**：SFC（单文件组件）`<template>` + `<script>` + `<style>`
- **Composition API**：`setup()` + `ref`/`reactive`/`computed`，逻辑复用更灵活

**核心架构：**

- 设计理念：渐进式框架，可从简单脚本逐步扩展到完整 SPA
- 核心模块：响应式系统（Reactivity）、编译器（Compiler）、渲染器（Renderer）
- 数据流：数据变化 → 响应式追踪 → 调度更新 → diff → DOM patch

**插件生态：**

- 官方插件：Vue Router、Pinia、VueUse
- 社区热门：Element Plus、Vuetify（UI 库）、Vue I18n（国际化）

## Why — 为什么

**适用场景：**

- 中小型 SPA 项目
- 快速原型开发
- 需要模板语法的团队（比 JSX 更贴近 HTML）
- 渐进式迁移老项目

**对比同类框架：**

| 维度 | Vue | React | Svelte |
|------|-----|-------|--------|
| 性能 | 高（细粒度更新） | 高（虚拟 DOM diff） | 极高（编译时优化） |
| 生态 | 丰富 | 最丰富 | 中等 |
| 学习曲线 | 最低 | 中 | 低 |
| 灵活性 | 中（约定明确） | 极高 | 中 |

**优缺点：**

- ✅ 优点：
  - 上手门槛最低，模板语法直观
  - 响应式自动追踪，无需手动优化
  - 官方全家桶（Router + Pinia）开箱即用
  - SFC 单文件组件，关注点分离清晰
- ❌ 缺点：
  - 响应式有边界情况（reactive 对象解构丢失响应性）
  - 大型项目不如 React 灵活
  - 国际化程度不如 React

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

### 代码示例

**响应式陷阱与解决：**

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

**组合式函数（Composable）：**

```javascript
// composables/useFetch.js
import { ref, watchEffect } from 'vue'

export function useFetch(url) {
    const data = ref(null)
    const error = ref(null)
    const loading = ref(true)

    watchEffect(async () => {
        try {
            loading.value = true
            const res = await fetch(url.value)
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
const { data, loading, error } = useFetch('/api/users')
</script>
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| reactive 解构丢失响应性 | 解构得到的是原始值，不再被 Proxy 追踪 | 用 `toRefs` 或直接使用 `ref` |
| ref 自动解包 | 模板中 ref 自动解包，JS 中需要 `.value` | 模板不加 `.value`，JS 必须加 |
| v-for 没有加 key | 列表渲染无法正确 diff | 始终添加唯一 `:key` |
| watch 监听 reactive 对象属性 | 直接监听 `state.count` 无效 | 用 getter `() => state.count` 或 `watchEffect` |

### 最佳实践

- 简单值用 `ref`，对象/数组用 `reactive`
- 逻辑复用用 Composable（`useXxx`）
- 用 `<script setup>` 语法糖，更简洁
- Pinia 替代 Vuex 做状态管理

## 面试题

**Q1: Vue 的响应式原理是什么？**
> Vue 3 使用 Proxy 拦截对象的 get/set 操作：get 时收集当前依赖（effect），set 时触发所有依赖重新执行。Vue 2 使用 Object.defineProperty，只能拦截已有属性的读写，无法检测新增/删除属性。

**Q2: v-if 和 v-show 的区别是什么？**
> `v-if` 是条件渲染，条件为 false 时 DOM 节点不创建，切换开销高；`v-show` 通过 CSS `display` 控制显隐，DOM 始终存在，初次渲染开销高。频繁切换用 v-show，条件很少变化用 v-if。

**Q3: computed 和 watch 的区别是什么？**
> `computed` 是计算属性，基于依赖缓存结果，依赖不变不重新计算，必须有返回值；`watch` 是监听器，数据变化时执行副作用（如 API 请求），不需要返回值。需要派生值用 computed，需要响应变化执行操作用 watch。

**Q4: Vue 组件通信有哪些方式？**
> ① props/emit — 父子通信；② provide/inject — 跨层级传递；③ ref + defineExpose — 父访问子方法；④ Pinia — 全局状态共享；⑤ EventBus/mitt — 任意组件事件通信（不推荐滥用）；⑥ $attrs — 透传属性。

**Q5: Vue 3 Composition API 相比 Options API 有什么优势？**
> 逻辑按功能聚合而非分散在 data/methods/computed 中，相关代码集中在一处；逻辑复用更灵活，通过 Composable（useXxx）替代 mixins（避免命名冲突和来源不明）；TypeScript 类型推断更好。

---

**相关链接：**
- [[React核心]]
- [[Webpack与Vite]]
- Vue 官方文档：https://vuejs.org/

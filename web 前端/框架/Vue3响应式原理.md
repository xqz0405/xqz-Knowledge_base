---
tags:
  - Web前端
  - Vue
  - 响应式
  - Proxy
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Vue3响应式原理

## What — 是什么

> Vue 3 使用 Proxy 实现响应式系统，拦截对象的读取和修改操作，自动追踪依赖并在数据变化时触发更新。

**核心概念：**

- **Proxy**：拦截 `get`/`set`/`deleteProperty` 等操作
- **依赖收集**：`get` 时将当前副作用函数（effect）收集为依赖
- **触发更新**：`set` 时通知所有依赖的副作用重新执行
- **effect**：Vue 内部的响应式副作用（组件渲染函数就是 effect）
- **ref vs reactive**：`ref` 包装基础类型（通过 `.value`），`reactive` 包装对象（深层 Proxy）

**核心架构：**

- 设计理念：数据驱动，自动追踪，精确更新
- 核心模块：reactivity（响应式）、runtime（调度渲染）
- 数据流：数据读取 → 收集依赖 → 数据修改 → 触发 effect → 重新渲染

**关键特性：**

- Vue 3 用 Proxy 替代 Vue 2 的 `Object.defineProperty`
- 支持 Map/Set/WeakMap 等新数据结构
- 惰性深层代理（访问嵌套属性时才代理）
- `shallowRef`/`shallowReactive` 可选浅层响应

### Vue2 响应式（Object.defineProperty）的局限性

Vue 2 的响应式基于 `Object.defineProperty`，它通过劫持对象已有属性的 getter/setter 实现数据追踪。这种方式有以下核心局限：

**1. 无法检测属性新增和删除**

```javascript
// Vue 2 中
const vm = new Vue({
  data: { obj: { name: 'Alice' } }
})

vm.obj.age = 25     // ❌ 不是响应式的，界面不会更新
delete vm.obj.name  // ❌ 不是响应式的，界面不会更新

// 必须使用 Vue.set / this.$set
Vue.set(vm.obj, 'age', 25)      // ✅ 响应式
this.$delete(vm.obj, 'name')     // ✅ 响应式
```

**2. 数组变更检测需要 hack**

Vue 2 无法通过 defineProperty 拦截数组的索引修改和长度变化，因此重写了 7 个数组变异方法：

```javascript
// Vue 2 内部重写的方法
const methodsToPatch = [
  'push', 'pop', 'shift', 'unshift',
  'splice', 'sort', 'reverse'
]

vm.items[0] = 'new'      // ❌ 不是响应式
vm.items.length = 0      // ❌ 不是响应式
vm.items.push('new')     // ✅ 响应式（方法被重写）
vm.items.splice(0, 1)    // ✅ 响应式（方法被重写）
```

**3. 深层监听需要递归初始化，性能差**

Vue 2 在初始化时就必须递归遍历整个对象的所有属性，为每个属性设置 defineProperty，即使某些嵌套属性永远不会被访问：

```javascript
// Vue 2 初始化时：一次性递归所有层级
const data = {
  user: {          // 第 1 层 → defineProperty
    profile: {     // 第 2 层 → defineProperty
      settings: {  // 第 3 层 → defineProperty
        theme: 'dark'  // 第 4 层 → defineProperty
      }
    }
  }
}
// 即使 never 访问 user.profile.settings，也已经被劫持
```

**4. 不支持 Map/Set/WeakMap/WeakSet**

defineProperty 无法拦截这些集合类型的方法调用，Vue 2 完全无法对它们做响应式处理。

### Vue3 响应式（Proxy）的完整机制

Vue 3 使用 ES6 Proxy 完全重新实现了响应式系统，从属性级别的劫持升级为对象级别的代理：

**Proxy 能拦截的 13 种操作：**

| 拦截操作 | 触发场景 | 响应式用途 |
|----------|----------|-----------|
| `get` | 读取属性 | 依赖收集 |
| `set` | 设置属性 | 触发更新 |
| `has` | `in` 操作符 | 依赖收集 |
| `deleteProperty` | `delete` 操作 | 触发更新 |
| `ownKeys` | `Object.keys` 等 | 依赖收集 |
| `getPrototypeOf` | 获取原型 | — |
| `setPrototypeOf` | 设置原型 | — |
| `isExtensible` | 判断可扩展性 | — |
| `preventExtensions` | 禁止扩展 | — |
| `getOwnPropertyDescriptor` | 获取属性描述符 | — |
| `defineProperty` | 定义属性 | — |
| `apply` | 函数调用 | — |
| `construct` | `new` 操作 | — |

**Vue 3 实际使用的拦截器：**

```javascript
// reactiveHandlers — 普通对象的处理器
const reactiveHandlers = {
  get,           // 依赖收集 + 惰性深层代理
  set,           // 触发更新
  deleteProperty,// 触发更新
  has,           // 依赖收集（in 操作符）
  ownKeys        // 依赖收集（Object.keys 等）
}

// collectionHandlers — Map/Set/WeakMap/WeakSet 的处理器
// 通过拦截方法调用实现响应式（如 get/has/add/delete/forEach 等）

// mutableReactiveHandler / readonlyHandlers / shallowReactiveHandlers
// 分别对应不同的响应式策略
```

**惰性深层代理的工作原理：**

```javascript
// 访问嵌套对象时才递归代理，而非初始化时
const state = reactive({
  user: {           // 初始化时不代理
    profile: {      // 初始化时不代理
      name: 'Alice' // 初始化时不代理
    }
  }
})

// 只有实际访问时才会触发代理
state.user           // get → 发现是对象 → reactive(user) → 代理
state.user.profile   // get → 发现是对象 → reactive(profile) → 代理
state.user.profile.name // get → 基础类型 → 直接返回
```

### reactive vs ref vs shallowReactive vs shallowRef 的区别

| 特性 | `reactive` | `ref` | `shallowReactive` | `shallowRef` |
|------|-----------|-------|-------------------|-------------|
| 接收类型 | 对象/数组 | 任意类型 | 对象/数组 | 任意类型 |
| 深层响应 | ✅ 是 | ✅ 是（.value 是对象时） | ❌ 只有根级 | ❌ 只有 .value 替换 |
| 访问方式 | 直接 `state.name` | `.value` | 直接 `state.name` | `.value` |
| 模板自动解包 | — | ✅ 自动 | — | ✅ 自动 |
| 解构丢失 | ✅ 会丢失 | ❌ 不会 | ✅ 会丢失 | ❌ 不会 |
| 适用场景 | 复杂对象 | 简单值/不确定类型 | 大对象性能优化 | 大数据结构性能优化 |

```javascript
import { reactive, ref, shallowReactive, shallowRef } from 'vue'

// reactive：深层响应式
const state = reactive({
  user: { name: 'Alice' },
  list: [1, 2, 3]
})
state.user.name = 'Bob'  // ✅ 触发更新

// ref：可包装任意类型
const count = ref(0)
count.value++            // ✅ 触发更新

const obj = ref({ name: 'Alice' })
obj.value.name = 'Bob'   // ✅ 触发更新（.value 是对象时深层响应）

// shallowReactive：只有根级属性是响应式的
const shallow = shallowReactive({
  user: { name: 'Alice' },
  count: 0
})
shallow.count++           // ✅ 触发更新（根级）
shallow.user.name = 'Bob' // ❌ 不触发更新（嵌套属性）

// shallowRef：只有 .value 替换触发更新
const shallowList = shallowRef([1, 2, 3])
shallowList.value.push(4)        // ❌ 不触发更新
shallowList.value = [1, 2, 3, 4] // ✅ 触发更新
```

### 依赖收集与派发更新的完整流程

Vue 3 响应式系统的核心数据结构：

```javascript
// 三层映射关系
// targetMap: WeakMap<target, Map<key, Set<effect>>>
//              ↓           ↓         ↓
//           目标对象    属性名    依赖集合

const targetMap = new WeakMap()  // 全局依赖映射表
let activeEffect = null          // 当前正在执行的副作用函数

// 依赖收集 —— get 时调用
function track(target, key) {
  if (!activeEffect) return  // 没有正在运行的 effect，无需收集

  let depsMap = targetMap.get(target)
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }

  let dep = depsMap.get(key)
  if (!dep) {
    dep = new Set()  // 用 Set 存储 effect，自动去重
    depsMap.set(key, dep)
  }

  dep.add(activeEffect)         // 将当前 effect 加入依赖集合
  activeEffect.deps.push(dep)   // effect 也记录自己的依赖（用于清理）
}

// 派发更新 —— set 时调用
function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return

  const dep = depsMap.get(key)
  if (!dep) return

  const effectsToRun = new Set(dep)  // 避免无限循环
  effectsToRun.forEach(effect => {
    // 如果 effect 有调度器，则由调度器决定执行时机
    if (effect.options.scheduler) {
      effect.options.scheduler(effect)
    } else {
      effect()  // 直接执行
    }
  })
}
```

**完整流程图：**

```
组件挂载
    ↓
执行组件渲染函数（本质是 effect）
    ↓
读取响应式数据 state.count
    ↓
触发 Proxy get 拦截器
    ↓
调用 track(target, 'count')
    ↓
将当前 effect 存入 targetMap[target]['count'] 的依赖集合
    ↓
【依赖收集完成】

---数据变更---

修改响应式数据 state.count = 1
    ↓
触发 Proxy set 拦截器
    ↓
调用 trigger(target, 'count')
    ↓
从 targetMap 中取出 'count' 对应的所有 effect
    ↓
根据调度策略执行 effect（同步 / 异步队列 / 自定义 scheduler）
    ↓
effect 重新执行 → 组件重新渲染
    ↓
【派发更新完成】
```

### effect 副作用函数与调度器

`effect` 是 Vue 3 响应式系统的基石，所有响应式行为的起点：

```javascript
import { effect } from '@vue/reactivity'

// 基本用法：立即执行一次，数据变化时自动重新执行
effect(() => {
  console.log(state.count)
})

// effect 返回值就是一个副作用函数，可以手动调用
const runner = effect(() => {
  console.log(state.count)
})
runner() // 手动触发

// effect 接受第二个参数 —— 选项对象
effect(() => {
  console.log(state.count)
}, {
  // lazy: 不立即执行，需要手动调用
  lazy: true,

  // scheduler: 数据变化时不直接执行 effect，而是执行调度器
  scheduler(fn) {
    // 典型用法：将更新放到微任务队列，实现批量异步更新
    queueMicrotask(fn)

    // 或者实现防抖
    // clearTimeout(timer)
    // timer = setTimeout(fn, 300)
  },

  // onTrack: 依赖被追踪时调用（调试用）
  onTrack(e) {
    console.log('tracked:', e)
  },

  // onTrigger: 依赖触发更新时调用（调试用）
  onTrigger(e) {
    console.log('triggered:', e)
  },

  // onStop: effect 被停止时调用
  onStop() {
    console.log('effect stopped')
  }
})
```

**effect 的内部实现简化版：**

```javascript
function effect(fn, options = {}) {
  const effectFn = () => {
    // 清理旧依赖（避免分支切换导致的无效依赖）
    cleanup(effectFn)
    // 设置当前 effect 为活跃状态
    activeEffect = effectFn
    // 执行原始函数，触发依赖收集
    const result = fn()
    // 恢复
    activeEffect = null
    return result
  }

  effectFn.deps = []           // 存储所有依赖的集合
  effectFn.options = options   // 保存选项

  if (!options.lazy) {
    effectFn()  // 立即执行一次
  }

  return effectFn
}

function cleanup(effectFn) {
  // 从所有依赖集合中移除该 effect
  for (let i = 0; i < effectFn.deps.length; i++) {
    effectFn.deps[i].delete(effectFn)
  }
  effectFn.deps.length = 0
}
```

**分支切换与依赖清理：**

```javascript
// 条件分支切换时，需要清理不再需要的依赖
const state = reactive({ ok: true, text: 'hello' })

effect(() => {
  // 当 ok 为 true 时，ok 和 text 都是依赖
  // 当 ok 为 false 时，只有 ok 是依赖，text 不应再被追踪
  console.log(state.ok ? state.text : 'default')
})

state.ok = false
state.text = 'changed' // 不应触发 effect，因为 ok 为 false 后 text 不再被读取
```

### computed 的缓存机制

`computed` 是基于 effect 之上的高层封装，核心特性是缓存：

```javascript
import { computed, reactive } from 'vue'

const state = reactive({ count: 0 })

const double = computed(() => {
  console.log('computed 执行') // 只有依赖变化时才执行
  return state.count * 2
})

// 多次读取，只计算一次
console.log(double.value) // "computed 执行"，输出 0
console.log(double.value) // 缓存命中，不执行计算，直接返回 0

state.count = 1
console.log(double.value) // "computed 执行"，输出 2
```

**computed 的缓存实现原理：**

```javascript
function computed(getter) {
  let value        // 缓存值
  let dirty = true // 脏标记：是否需要重新计算

  const runner = effect(getter, {
    lazy: true,  // 不立即执行
    scheduler: () => {
      dirty = true  // 依赖变化时标记为脏，但不立即重算
      trigger(obj, 'value') // 通知依赖 computed 的 effect
    }
  })

  const obj = {
    get value() {
      if (dirty) {
        value = runner()  // 脏时才重新计算
        dirty = false     // 计算后标记为干净
        track(obj, 'value') // 让其他 effect 依赖 computed
      }
      return value
    }
  }

  return obj
}
```

**缓存机制的关键点：**

1. **dirty 标记**：初始为 true，首次访问时计算并置为 false
2. **scheduler**：依赖变化时不直接重算，只标记 dirty = true
3. **惰性求值**：只有被读取时才判断是否需要重新计算
4. **避免重复计算**：连续多次读取，只要 dirty 为 false 就返回缓存值

### watch vs watchEffect 的区别

| 特性 | `watch` | `watchEffect` |
|------|---------|---------------|
| 依赖声明 | 显式指定监听源 | 自动追踪回调中的响应式数据 |
| 旧值访问 | ✅ 可获取 oldValue | ❌ 无法获取 |
| 首次执行 | 默认不执行（lazy） | 立即执行 |
| 精确控制 | 可监听特定属性 | 追踪所有访问的响应式数据 |
| 回调参数 | `(newVal, oldVal)` | `onCleanup` |
| 深层监听 | 需要设置 `deep: true` | 自动追踪深层 |
| 暂停/恢复 | 支持 | 支持 |

```javascript
import { watch, watchEffect, ref, reactive } from 'vue'

// watchEffect：立即执行，自动追踪
const stop = watchEffect((onCleanup) => {
  console.log(state.count)
  // 自动追踪 state.count

  // 清理副作用
  const timer = setInterval(() => {}, 1000)
  onCleanup(() => clearInterval(timer))
})

stop() // 停止监听

// watch：显式指定监听源
const count = ref(0)

// 监听单个 ref
watch(count, (newVal, oldVal) => {
  console.log(`count: ${oldVal} → ${newVal}`)
})

// 监听 getter 函数
watch(
  () => state.count,
  (newVal, oldVal) => {
    console.log(`count: ${oldVal} → ${newVal}`)
  }
)

// 监听多个源
watch(
  [() => state.count, () => state.name],
  ([newCount, newName], [oldCount, oldName]) => {
    console.log('changed')
  }
)

// 监听 reactive 对象的深层变化
const obj = reactive({ nested: { count: 0 } })

// 方式一：deep 选项
watch(obj, (newVal) => {
  // 注意：newVal 和 reactive 对象是同一个引用
  console.log('deep change detected')
}, { deep: true })

// 方式二：getter + deep
watch(
  () => obj.nested,
  (newVal) => {
    console.log('nested changed')
  },
  { deep: true }
)

// watch 的完整选项
watch(source, callback, {
  immediate: true,  // 创建时立即执行一次
  deep: true,       // 深层监听
  flush: 'post',    // 回调执行时机：'pre' | 'post' | 'sync'
  once: true,       // Vue 3.4+：只触发一次
})
```

**flush 选项的执行时机：**

| flush 值 | 执行时机 | 适用场景 |
|----------|---------|---------|
| `pre`（默认） | 组件更新前 | 修改数据，在 DOM 更新前做处理 |
| `post` | 组件更新后 | 需要访问更新后的 DOM |
| `sync` | 同步执行 | 需要立即响应（谨慎使用，性能差） |

### 响应式数据的解构丢失问题（toRefs/toRef）

**为什么解构会丢失响应式：**

```javascript
const state = reactive({ name: 'Alice', age: 25 })

// 解构时，get 拦截器返回的是原始值
let { name } = state
// 等价于：
// const name = state.name  → Proxy get → 返回 'Alice'（字符串原始值）
// name 变量存储的是 'Alice'，与 Proxy 无关

name = 'Bob'  // 只是修改了局部变量，与 state 无关
```

**解决方案：**

```javascript
import { toRefs, toRef, reactive } from 'vue'

const state = reactive({ name: 'Alice', age: 25 })

// ✅ toRefs：将 reactive 对象的所有属性转为 ref
const { name, age } = toRefs(state)
name.value = 'Bob'  // 触发更新，state.name 也变为 'Bob'

// ✅ toRef：将单个属性转为 ref
const nameRef = toRef(state, 'name')
nameRef.value = 'Bob'  // 触发更新

// ✅ toRefs 在组合式函数中的典型用法
function useUser() {
  const state = reactive({
    name: 'Alice',
    age: 25,
    update() { /* ... */ }
  })

  // 返回时用 toRefs 保持响应式
  return toRefs(state)
}

// 使用
const { name, age } = useUser()
name.value = 'Bob' // ✅ 响应式
```

**toRefs 的实现原理：**

```javascript
function toRefs(object) {
  const result = {}
  for (const key in object) {
    result[key] = toRef(object, key)
  }
  return result
}

function toRef(object, key) {
  return {
    get value() {
      return object[key]  // 读取时走 Proxy get，触发依赖收集
    },
    set value(newVal) {
      object[key] = newVal // 设置时走 Proxy set，触发更新
    }
  }
}
```

## Why — 为什么

**适用场景：**

- 理解 Vue 响应式行为（为什么解构丢失响应性）
- 调试响应式问题
- 编写自定义响应式逻辑

### Vue2 defineProperty vs Vue3 Proxy 的详细对比

| 对比维度 | Vue 2 (defineProperty) | Vue 3 (Proxy) |
|----------|----------------------|---------------|
| 拦截级别 | 属性级别（逐个劫持） | 对象级别（整体代理） |
| 新增属性 | ❌ 无法检测（需 `Vue.set`） | ✅ 自动检测 |
| 删除属性 | ❌ 无法检测（需 `Vue.delete`） | ✅ 自动检测（`deleteProperty`） |
| 数组索引修改 | ❌ 无法检测 | ✅ 自动检测 |
| 数组长度修改 | ❌ 无法检测 | ✅ 自动检测 |
| Map/Set 支持 | ❌ 不支持 | ✅ 完整支持 |
| 深层监听策略 | 初始化时递归遍历（全量） | 惰性递归（按需） |
| 初始化性能 | 差（大型对象启动慢） | 好（只代理最外层） |
| 原型链属性 | 可能意外劫持 | 不会（Proxy 不影响原型） |
| 兼容性 | IE8+ | IE 不支持（需 ES6） |

### 为什么 Vue3 从 defineProperty 切换到 Proxy

**1. 语言层面的根本缺陷**

defineProperty 是属性级别的劫持，必须预先知道有哪些属性才能劫持。而 Proxy 是对象级别的代理，无论对象后续如何变化（增删属性），都能被拦截到。这是从"被动防御"到"主动代理"的根本性升级。

**2. 性能优化**

```javascript
// Vue 2：初始化时必须递归遍历所有属性
function observe(obj) {
  if (typeof obj !== 'object') return
  Object.keys(obj).forEach(key => {
    defineReactive(obj, key, obj[key]) // 每个属性都调用 defineProperty
  })
}

function defineReactive(obj, key, val) {
  observe(val) // 递归处理嵌套对象 → 启动时全部遍历
  Object.defineProperty(obj, key, {
    get() { /* ... */ },
    set() { /* ... */ }
  })
}

// Vue 3：只代理外层，内层按需
function reactive(target) {
  return new Proxy(target, handlers) // 一次 Proxy 创建，完成
  // 嵌套对象在 get 时才递归代理
}
```

**3. 更好的开发者体验**

- 不再需要 `Vue.set()` / `Vue.delete()`
- 数组可以直接通过索引修改
- Map/Set 可以正常使用
- 减少了大量"魔法 API"的记忆负担

**4. 代码量减少**

Vue 3 的 reactivity 模块代码量约 1000 行，而 Vue 2 的 Observer 模块约 1500 行，功能却更完整。

### Proxy 还有什么局限性

Proxy 虽然比 defineProperty 强大得多，但也有一些局限：

**1. 原始值无法代理**

```javascript
// Proxy 只能代理对象，不能代理原始值
const str = 'hello'
// new Proxy(str, {}) → TypeError

// 所以 Vue 3 需要 ref 来包装原始值
const count = ref(0) // 通过 { value: 0 } 的对象包装来间接实现
```

**2. 性能损耗**

Proxy 本身有运行时开销，虽然惰性代理改善了初始化性能，但每次属性访问都会经过 Proxy 拦截器，对高频访问的属性可能产生性能影响。这也是 `shallowReactive`/`shallowRef` 存在的意义。

**3. 无法穿透内置对象的部分内部插槽**

```javascript
// 某些内置对象有内部插槽（internal slots），Proxy 无法正确代理
const date = new Date()
const proxy = new Proxy(date, {})
proxy.getTime() // TypeError: this is not a Date object

// Vue 3 的解决方案：get 拦截器中绑定 this
get(target, key, receiver) {
  const result = Reflect.get(target, key, receiver)
  if (typeof result === 'function') {
    return result.bind(target) // 绑定原始对象
  }
  return result
}
```

**4. 不支持 IE11**

Proxy 无法被 polyfill，这意味着 Vue 3 无法支持 IE11。Vue 官方已放弃 IE11 支持，这在 2026 年已不是问题。

**5. JSON.stringify 行为**

```javascript
const state = reactive({ name: 'Alice' })
JSON.stringify(state) // 结果正常，因为 Vue 3 处理了 toJSON

// 但在某些边缘情况下，Proxy 对象的序列化可能不符合预期
// Vue 3 通过 toRaw() 获取原始对象来解决
```

**对比其他方案：**

| 维度 | Vue 3 Proxy | Vue 2 defineProperty | React setState | Solid Signals |
|------|------------|---------------------|---------------|---------------|
| 拦截能力 | 全面（增删改查） | 有限（无法拦截新增属性） | 手动触发 | 全面 |
| 深层监听 | 惰性代理 | 递归初始化（性能差） | 手动 | 惰性 |
| 数组支持 | 完美 | 需要 hack | 正常 | 完美 |
| 学习成本 | 中等 | 中等 | 低 | 低 |
| 运行时开销 | 中等（Proxy 拦截） | 低（直接 getter/setter） | 低 | 极低（编译时优化） |
| 调试体验 | 较差（Proxy 包装） | 较好 | 好 | 好 |

**优缺点：**

- ✅ 优点：
  - 自动追踪，无需手动声明依赖
  - 精确更新，只更新变化的组件
  - 支持完整 JS 对象操作
  - 惰性深层代理，初始化性能好
  - 原生支持 Map/Set 等集合类型
- ❌ 缺点：
  - 解构 reactive 对象丢失响应性
  - Proxy 无法被 IE11 支持（已不是问题）
  - 调试时 Proxy 包装增加阅读复杂度
  - 原始值必须通过 ref 包装

## How — 怎么用

### 快速上手

```javascript
import { reactive, ref, effect, computed } from '@vue/reactivity'

const state = reactive({ count: 0, name: 'Alice' })
const double = computed(() => state.count * 2)

effect(() => {
    console.log(`count is ${state.count}, double is ${double.value}`)
})
// 立即输出: "count is 0, double is 0"

state.count++
// 自动输出: "count is 1, double is 2"
```

### 代码示例 1：reactive/ref/computed 基本用法

```javascript
import { reactive, ref, computed } from 'vue'

// ===== reactive =====
const user = reactive({
  name: 'Alice',
  age: 25,
  hobbies: ['reading', 'coding']
})
user.name = 'Bob'          // ✅ 直接修改
user.hobbies.push('gaming') // ✅ 数组方法正常使用
delete user.age             // ✅ 删除属性也是响应式的

// ===== ref =====
const count = ref(0)
const message = ref('hello')
const list = ref([1, 2, 3])

count.value++              // ✅ 修改需要 .value
message.value = 'world'    // ✅ 字符串
list.value.push(4)         // ✅ .value 是数组时，深层响应

// 在模板中自动解包，无需 .value
// <template>{{ count }}</template>  ← 等价于 count.value

// ===== computed =====
const fullName = computed(() => {
  return `${user.firstName} ${user.lastName}`
})

// 可写 computed
const firstName = computed({
  get: () => user.name.split(' ')[0],
  set: (val) => {
    user.name = `${val} ${user.name.split(' ')[1]}`
  }
})
firstName.value = 'Charlie' // 触发 set

// computed 的 stop
const stoppable = computed(() => count.value * 2)
// Vue 3.3+ 可以停止 computed
// stoppable.effect.stop()
```

### 代码示例 2：手写简易 reactive（Proxy + effect）

```javascript
// 完整的迷你响应式系统实现

// ===== 全局状态 =====
const targetMap = new WeakMap()  // 依赖映射表
let activeEffect = null          // 当前活跃的 effect
const effectStack = []           // effect 栈（支持嵌套）

// ===== effect =====
function effect(fn, options = {}) {
  const effectFn = () => {
    cleanup(effectFn)          // 清理旧依赖
    activeEffect = effectFn
    effectStack.push(effectFn)
    const result = fn()        // 执行函数，触发依赖收集
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1] || null
    return result
  }

  effectFn.deps = []           // 存储依赖集合的引用
  effectFn.options = options

  if (!options.lazy) {
    effectFn()
  }

  return effectFn
}

function cleanup(effectFn) {
  for (const dep of effectFn.deps) {
    dep.delete(effectFn)
  }
  effectFn.deps.length = 0
}

// ===== 依赖收集 =====
function track(target, key) {
  if (!activeEffect) return

  let depsMap = targetMap.get(target)
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }

  let dep = depsMap.get(key)
  if (!dep) {
    dep = new Set()
    depsMap.set(key, dep)
  }

  if (!dep.has(activeEffect)) {
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
  }
}

// ===== 派发更新 =====
function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return

  const dep = depsMap.get(key)
  if (!dep) return

  const effectsToRun = new Set()
  dep.forEach(effect => {
    if (effect !== activeEffect) { // 避免无限递归
      effectsToRun.add(effect)
    }
  })

  effectsToRun.forEach(effectFn => {
    if (effectFn.options.scheduler) {
      effectFn.options.scheduler(effectFn)
    } else {
      effectFn()
    }
  })
}

// ===== reactive =====
const reactiveMap = new WeakMap() // 缓存已代理的对象

function reactive(target) {
  if (typeof target !== 'object' || target === null) return target

  // 避免重复代理
  if (reactiveMap.has(target)) {
    return reactiveMap.get(target)
  }

  const proxy = new Proxy(target, {
    get(obj, key, receiver) {
      // 如果是内置 Symbol，不追踪
      if (key === '__v_raw') return obj
      if (typeof key === 'symbol' && key in builtInSymbols) return Reflect.get(obj, key, receiver)

      track(obj, key)  // 依赖收集
      const result = Reflect.get(obj, key, receiver)

      // 惰性深层代理
      if (typeof result === 'object' && result !== null) {
        return reactive(result)
      }
      return result
    },

    set(obj, key, value, receiver) {
      const oldValue = obj[key]
      const result = Reflect.set(obj, key, value, receiver)

      // 只有值确实发生变化才触发更新
      if (oldValue !== value) {
        trigger(obj, key)
      }
      return result
    },

    deleteProperty(obj, key) {
      const hadKey = key in obj
      const result = Reflect.deleteProperty(obj, key)
      if (hadKey && result) {
        trigger(obj, key)
      }
      return result
    },

    has(obj, key) {
      track(obj, key)
      return Reflect.has(obj, key)
    },

    ownKeys(obj) {
      track(obj, 'iterate')
      return Reflect.ownKeys(obj)
    }
  })

  reactiveMap.set(target, proxy)
  return proxy
}

// ===== 测试 =====
const state = reactive({ count: 0, name: 'Alice' })

effect(() => {
  console.log(`count = ${state.count}`)
})
// 输出: count = 0

state.count = 1
// 输出: count = 1
```

### 代码示例 3：手写简易 ref

```javascript
function ref(initialValue) {
  return customRef((track, trigger) => {
    let value = initialValue

    return {
      get value() {
        track()     // 收集依赖
        return value
      },
      set value(newVal) {
        if (newVal !== value) {
          value = newVal
          trigger()  // 触发更新
        }
      }
    }
  })
}

// 更直接的实现（不用 customRef）
function simpleRef(initialValue) {
  const r = reactive({ value: initialValue })
  return {
    get value() { return r.value },
    set value(newVal) { r.value = newVal }
  }
}

// ===== 测试 =====
const count = ref(0)

effect(() => {
  console.log(`ref count = ${count.value}`)
})
// 输出: ref count = 0

count.value = 10
// 输出: ref count = 10

// ref 包装对象时，深层也是响应式的
const obj = ref({ name: 'Alice' })
obj.value.name = 'Bob' // ✅ 触发更新
```

### 代码示例 4：toRefs 解构保持响应式

```javascript
import { reactive, toRefs, toRef } from 'vue'

// ===== 组合式函数中的典型用法 =====
function useCounter(initialValue = 0) {
  const state = reactive({
    count: initialValue,
    doubled: computed(() => state.count * 2)
  })

  function increment() {
    state.count++
  }

  function decrement() {
    state.count--
  }

  function reset() {
    state.count = initialValue
  }

  // 返回 toRefs 保持响应式
  return {
    ...toRefs(state),
    increment,
    decrement,
    reset
  }
}

// 使用
const { count, doubled, increment } = useCounter(10)
// count 和 doubled 都是 ref，保持响应式

// ===== toRef 单个属性 =====
const state = reactive({ name: 'Alice', age: 25, city: 'Beijing' })

// 只转换某个属性
const nameRef = toRef(state, 'name')
nameRef.value = 'Bob' // state.name 也变为 'Bob'

// toRef 对不存在的属性也能创建 ref（避免报错）
const phoneRef = toRef(state, 'phone') // 不会报错
console.log(phoneRef.value) // undefined

// ===== 在 props 中的使用 =====
export default {
  props: ['title', 'count'],
  setup(props) {
    // ❌ 不要解构 props，会丢失响应式
    // const { title } = props

    // ✅ 使用 toRef 保持响应式
    const title = toRef(props, 'title')
    const count = toRef(props, 'count')

    return { title, count }
  }
}
```

### 代码示例 5：watch/watchEffect 完整用法

```javascript
import {
  watch, watchEffect, ref, reactive,
  onWatcherCleanup // Vue 3.5+
} from 'vue'

// ===== watchEffect：自动追踪依赖 =====
const count = ref(0)
const name = ref('Alice')

watchEffect(() => {
  console.log(`count: ${count.value}, name: ${name.value}`)
  // 自动追踪 count 和 name
})

// 带清理函数
watchEffect((onCleanup) => {
  const controller = new AbortController()

  fetch(`/api/data?count=${count.value}`, {
    signal: controller.signal
  })
    .then(res => res.json())
    .then(data => {
      // 处理数据
    })

  // 下次 effect 重新执行前调用
  onCleanup(() => {
    controller.abort() // 取消上一次请求
  })
})

// 返回停止函数
const stop = watchEffect(() => {
  console.log(count.value)
})
stop() // 停止监听

// ===== watch：精确控制 =====

// 1. 监听 ref
watch(count, (newVal, oldVal) => {
  console.log(`count: ${oldVal} → ${newVal}`)
})

// 2. 监听 getter
watch(
  () => state.user.name,
  (newVal, oldVal) => {
    console.log(`name: ${oldVal} → ${newVal}`)
  }
)

// 3. 监听多个源
watch(
  [count, () => state.user.name],
  ([newCount, newName], [oldCount, oldName]) => {
    console.log('multiple sources changed')
  }
)

// 4. 深层监听
const obj = reactive({
  nested: { deep: { value: 0 } }
})

watch(
  () => obj,
  (newVal) => {
    console.log('deep change')
  },
  { deep: true }
)

// 5. immediate：立即执行
watch(count, (newVal, oldVal) => {
  console.log(`immediate: ${newVal}`)
}, { immediate: true })

// 6. flush: 'post' — DOM 更新后执行
watch(count, () => {
  // 可以安全访问更新后的 DOM
  const el = document.getElementById('counter')
  console.log(el.textContent)
}, { flush: 'post' })

// 7. once：只触发一次（Vue 3.4+）
watch(count, (newVal) => {
  console.log('only once')
}, { once: true })

// 8. 暂停和恢复（Vue 3.5+）
const watcher = watch(count, (newVal) => {
  console.log(newVal)
})
watcher.pause()   // 暂停
watcher.resume()  // 恢复

// 9. watch 中的清理回调
watch(id, (newVal, oldVal, onCleanup) => {
  const controller = new AbortController()

  fetch(`/api/user/${newVal}`, { signal: controller.signal })
    .then(res => res.json())
    .then(data => {
      // ...
    })

  onCleanup(() => {
    controller.abort()
  })
})
```

### 代码示例 6：自定义 ref（customRef 防抖示例）

```javascript
import { customRef, watchEffect } from 'vue'

// ===== 防抖 ref =====
function useDebouncedRef(value, delay = 300) {
  let timeout
  return customRef((track, trigger) => {
    return {
      get value() {
        track()  // 追踪依赖
        return value
      },
      set value(newVal) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newVal
          trigger()  // 延迟触发更新
        }, delay)
      }
    }
  })
}

// 使用
const keyword = useDebouncedRef('', 500)

watchEffect(() => {
  console.log('搜索关键词:', keyword.value)
})

// 快速输入时不会频繁触发更新
keyword.value = 'a'
keyword.value = 'ab'
keyword.value = 'abc' // 只有最后一次在 500ms 后触发更新

// ===== 自定义 ref：本地存储同步 =====
function useLocalStorageRef(key, defaultValue) {
  let value
  try {
    const stored = localStorage.getItem(key)
    value = stored !== null ? JSON.parse(stored) : defaultValue
  } catch {
    value = defaultValue
  }

  return customRef((track, trigger) => {
    return {
      get value() {
        track()
        return value
      },
      set value(newVal) {
        value = newVal
        localStorage.setItem(key, JSON.stringify(newVal))
        trigger()
      }
    }
  })
}

// 使用
const theme = useLocalStorageRef('app-theme', 'light')
theme.value = 'dark' // 自动同步到 localStorage

// ===== 自定义 ref：验证 ref =====
function useValidatedRef(getter, setter, validator) {
  let value = getter()
  let error = ''

  return customRef((track, trigger) => {
    return {
      get value() {
        track()
        return value
      },
      set value(newVal) {
        const validation = validator(newVal)
        if (validation === true) {
          value = setter(newVal)
          error = ''
        } else {
          error = validation
        }
        trigger()
      }
    }
  })
}

const email = useValidatedRef(
  () => '',
  (v) => v,
  (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) || '请输入有效邮箱'
)
```

### 代码示例 7：shallowReactive/shallowRef 性能优化

```javascript
import {
  shallowReactive, shallowRef,
  triggerRef, isReactive, isShallow
} from 'vue'

// ===== shallowReactive：只代理根级属性 =====
const state = shallowReactive({
  name: 'Alice',         // ✅ 响应式（根级）
  profile: {             // ❌ 非响应式（嵌套）
    age: 25,
    address: {
      city: 'Beijing'
    }
  }
})

state.name = 'Bob'              // ✅ 触发更新
state.profile.age = 30          // ❌ 不触发更新
state.profile = { age: 30 }     // ✅ 触发更新（替换根级属性）

// 适用场景：大型表单，只关心表单整体提交
const form = shallowReactive({
  username: '',
  password: '',
  // ... 100 个字段，但不需要逐字段响应式
})
form.username = 'admin'  // ✅ 触发更新
form.password = '123456' // ✅ 触发更新

// ===== shallowRef：只追踪 .value 替换 =====
const bigList = shallowRef([])

// 从 API 获取大量数据
async function fetchList() {
  const data = await api.getList() // 10000 条数据
  bigList.value = data // ✅ 整体替换触发更新
}

// ❌ 修改内部不会触发更新
bigList.value.push('new item')      // 不触发
bigList.value[0].name = 'changed'   // 不触发

// ✅ 方式一：替换整个值
bigList.value = [...bigList.value, 'new item']

// ✅ 方式二：手动触发
bigList.value.push('new item')
triggerRef(bigList) // 强制通知依赖更新

// ===== 判断响应式类型 =====
console.log(isReactive(state))  // true
console.log(isShallow(state))   // true

const deepState = reactive({ name: 'Alice' })
console.log(isReactive(deepState))  // true
console.log(isShallow(deepState))   // false

// ===== 典型性能优化场景 =====

// 1. 不可变的配置对象
const config = shallowRef({
  apiEndpoint: 'https://api.example.com',
  timeout: 5000,
  // ... 更多配置
})

// 2. ECharts 实例
const chartInstance = shallowRef(null)
chartInstance.value = echarts.init(el) // 不需要深层响应式

// 3. DOM 元素引用
const canvasRef = shallowRef(null)
// template: <canvas ref="canvasRef"></canvas>
```

### 代码示例 8：effectScope 副作用作用域

```javascript
import { effectScope, onScopeDispose, ref, watch, computed } from 'vue'

// ===== 基本用法 =====
const scope = effectScope(() => {
  const count = ref(0)

  // 这些副作用都会被 scope 管理
  watch(count, (val) => {
    console.log('count changed:', val)
  })

  const doubled = computed(() => count.value * 2)

  onScopeDispose(() => {
    console.log('scope 被销毁了')
  })
})

// 当不再需要时，一次性停止所有副作用
scope.stop() // 停止所有 watch、computed、effect

// ===== 在组合式函数中使用 =====
function useMousePosition() {
  const x = ref(0)
  const y = ref(0)

  const scope = effectScope()

  scope.run(() => {
    // 这些事件监听器会在 scope.stop() 时自动清理
    watchEffect(() => {
      window.addEventListener('mousemove', (e) => {
        x.value = e.clientX
        y.value = e.clientY
      })
    })

    onScopeDispose(() => {
      window.removeEventListener('mousemove', handler)
    })
  })

  // 返回数据和清理函数
  return {
    x,
    y,
    stop: () => scope.stop()
  }
}

// ===== 嵌套作用域 =====
const parent = effectScope()

parent.run(() => {
  const child = effectScope()

  child.run(() => {
    watch(someRef, () => {
      console.log('child watch')
    })
  })

  // 如果 child 没有设置 detached: true
  // 当 parent.stop() 时，child 也会被停止
})

// 独立作用域（不受父级影响）
const detached = effectScope(true) // detached: true
detached.run(() => {
  watch(someRef, () => {
    console.log('detached watch')
  })
})
// parent.stop() 不会影响 detached

// ===== getCurrentScope =====
import { getCurrentScope } from 'vue'

function useFeature() {
  const scope = getCurrentScope()
  if (!scope) {
    console.warn('请在 effectScope 中使用')
    return
  }

  onScopeDispose(() => {
    // 清理逻辑
  })
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| reactive 解构丢失响应性 | 解构得到原始值脱离 Proxy | `toRefs` 或改用 `ref` |
| 直接修改 reactive 数组索引 | Vue 3 Proxy 已支持 | Vue 2 不支持，Vue 3 没问题 |
| `ref` 模板自动解包但 JS 不行 | 模板编译时自动加 `.value` | JS 中必须写 `.value` |
| `reactive` 重新赋值整个对象 | 替换引用后不再被代理 | 用 `Object.assign(state, newObj)` 或 `ref` |
| computed 中使用异步操作 | computed 不支持异步（必须同步返回值） | 改用 `watch` + `ref` 手动管理 |
| watch 监听 reactive 对象得到相同引用 | reactive 返回的是 Proxy 代理 | 监听 getter 或使用 `deep: true` |
| 修改 props 的 ref 报错 | 单向数据流，props 不应被修改 | emit 事件通知父组件修改 |
| shallowRef 修改内部不更新 | shallowRef 只追踪 .value 替换 | 替换整个值或 `triggerRef()` |
| 嵌套 ref 自动解包失效 | 只有顶层 ref 在 reactive 中自动解包 | 手动 `.value` 访问 |
| reactive 的 Map/Set 方法调用未触发更新 | 需要使用 Vue 提供的集合代理方法 | 确保对象经过 `reactive()` 包装 |

### 最佳实践

- 简单值用 `ref`，对象/数组用 `reactive`
- 解构 reactive 时用 `toRefs`
- 不需要深层响应时用 `shallowRef`/`shallowReactive` 提升性能
- 只读数据用 `readonly()` 或 `shallowReadonly()`
- 组合式函数中用 `effectScope` 管理副作用生命周期
- `watch` 需要旧值时使用，`watchEffect` 用于"执行并追踪"的副作用
- 使用 `toRaw()` 获取原始对象以减少 Proxy 开销
- 避免在 computed 中执行异步操作或产生副作用
- 大型数据结构（如 ECharts 实例、大量数据列表）使用 `shallowRef`
- 使用 `markRaw()` 标记永远不需要响应式的对象

## 面试题

**Q1: Vue 3 为什么用 Proxy 替代 Object.defineProperty？**

> Proxy 可以拦截对象的所有操作（属性新增、删除、修改、查询），无需提前遍历属性；而 Object.defineProperty 只能劫持已有属性，新增/删除属性无法检测，必须使用 `Vue.set()`/`Vue.delete()`。此外，Proxy 还原生支持数组索引修改和 Map/Set 等集合类型，初始化性能更好（惰性代理 vs 递归遍历），代码实现也更简洁。

**Q2: reactive 和 ref 有什么区别？什么时候用哪个？**

> `reactive` 接收对象/数组，返回 Proxy 代理，直接访问属性无需 `.value`，但解构会丢失响应性；`ref` 接收任意类型（包括原始值），通过 `.value` 访问，在模板中自动解包。选择建议：简单值（数字、字符串、布尔）用 ref，确定结构的对象/数组用 reactive，不确定类型时优先用 ref。实际项目中 ref 使用更广泛，因为解构安全、重新赋值安全。

**Q3: 为什么解构 reactive 对象会丢失响应式？如何解决？**

> 解构时 Proxy 的 get 拦截器返回的是原始值（如字符串、数字），新变量脱离了 Proxy 代理，修改普通变量不会触发 set 拦截器。解决方案有三种：① `toRefs(state)` 将每个属性转为 ref，通过 `.value` 保持响应式连接；② 改用 `ref` 包装对象，通过 `.value` 访问整个对象；③ 不解构，直接使用 `state.name` 访问。推荐方案一，在组合式函数返回值时尤其常用。

**Q4: computed 是如何实现缓存的？**

> computed 内部使用 `dirty` 标记 + `scheduler` 实现缓存。首次访问时 dirty 为 true，执行 getter 计算值并缓存结果，将 dirty 置为 false；后续访问时 dirty 为 false，直接返回缓存值不重新计算。当依赖数据变化时，scheduler 被调用，只将 dirty 重新置为 true，不立即重算（惰性求值）。下次有人读取 computed 时才真正重新计算。这种设计避免了无谓的计算开销。

**Q5: Vue 3 依赖收集的流程是怎样的？**

> ① 组件渲染时执行 effect 函数；② 读取响应式数据触发 Proxy get 拦截；③ get 中调用 track()，将当前 effect 记录到该属性的依赖集合（Dep）中；④ 数据修改触发 set 拦截；⑤ set 中调用 trigger()，遍历 Dep 通知所有 effect 重新执行。

**Q6: watch 和 watchEffect 有什么区别？**

> `watchEffect` 立即执行回调并自动追踪其中使用的响应式数据，无法获取旧值，适合"执行并响应"的场景。`watch` 显式指定监听源，默认不立即执行，可获取新旧值，支持 deep/immediate/flush 等选项，适合需要精确控制监听范围和获取变化前值的场景。简单来说：watchEffect 像自动追踪的 effect，watch 像精确的观察者。

**Q7: shallowReactive 和 shallowRef 有什么用？**

> 它们是浅层响应式 API，只追踪第一层/根级变化。`shallowReactive` 只代理对象的根级属性，嵌套对象不会被 reactive 包装；`shallowRef` 只追踪 `.value` 的替换，内部属性变化不触发更新。适用场景：大型对象/数组不需要逐属性响应式时，使用浅层 API 减少代理开销提升性能；不可变数据（如 ECharts 实例、大型配置对象）也适合用 shallowRef。

**Q8: Vue 3 的 effectScope 是什么？有什么用？**

> `effectScope` 创建一个副作用作用域，统一管理其内部创建的所有 effect、watch、computed。调用 `scope.stop()` 可以一次性停止所有副作用，避免手动逐个清理。典型场景：组合式函数中创建多个 watch/eventListener，在组件卸载时需要统一清理。嵌套的 effectScope 会被父级一起停止（除非设置 `detached: true`）。这是 Vue 3.2+ 提供的底层 API，解决副作用生命周期管理问题。

---

**相关链接：**
- [[Vue核心]]
- [[React Hooks详解]]
- [[Proxy与Reflect]]
- [[JavaScript响应式编程]]
- [[Vue组合式API]]

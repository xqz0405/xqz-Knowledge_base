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

## Why — 为什么

**适用场景：**

- 理解 Vue 响应式行为（为什么解构丢失响应性）
- 调试响应式问题
- 编写自定义响应式逻辑

**对比其他方案：**

| 维度 | Vue 3 Proxy | Vue 2 defineProperty | React setState | Solid Signals |
|------|------------|---------------------|---------------|---------------|
| 拦截能力 | 全面（增删改查） | 有限（无法拦截新增属性） | 手动触发 | 全面 |
| 深层监听 | 惰性代理 | 递归初始化（性能差） | 手动 | 惰性 |
| 数组支持 | 完美 | 需要 hack | 正常 | 完美 |

**优缺点：**

- ✅ 优点：
  - 自动追踪，无需手动声明依赖
  - 精确更新，只更新变化的组件
  - 支持完整 JS 对象操作
- ❌ 缺点：
  - 解构 reactive 对象丢失响应性
  - Proxy 无法被 IE11 支持（已不是问题）
  - 调试时 Proxy 包装增加阅读复杂度

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

### 代码示例

**Proxy 拦截原理简化版：**

```javascript
function reactive(target) {
    return new Proxy(target, {
        get(obj, key, receiver) {
            // 依赖收集：记录"谁在读取"
            track(obj, key);
            const result = Reflect.get(obj, key, receiver);
            // 惰性深层代理
            if (typeof result === 'object' && result !== null) {
                return reactive(result);
            }
            return result;
        },
        set(obj, key, value, receiver) {
            const oldValue = obj[key];
            const result = Reflect.set(obj, key, value, receiver);
            if (oldValue !== value) {
                // 触发更新：通知依赖的 effect
                trigger(obj, key);
            }
            return result;
        },
    });
}
```

**为什么解构丢失响应性：**

```javascript
const state = reactive({ name: 'Alice', age: 25 })

// ❌ 解构得到原始值，不是 Proxy 代理
let { name } = state
name = 'Bob' // 只修改了局部变量，state.name 不变

// ✅ toRefs 将每个属性变成 ref
const { name, age } = toRefs(state)
name.value = 'Bob' // 触发响应式更新

// ✅ 直接用 ref
const name = ref('Alice')
name.value = 'Bob' // 触发更新
```

**shallowRef 与 triggerRef：**

```javascript
// shallowRef：只追踪 .value 本身的替换
const list = shallowRef([1, 2, 3])
list.value.push(4)       // ❌ 不触发更新（数组没变引用）
list.value = [...list.value, 4] // ✅ 替换整个 .value 触发更新

// 或手动触发
list.value.push(4);
triggerRef(list); // 强制通知更新
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| reactive 解构丢失响应性 | 解构得到原始值脱离 Proxy | `toRefs` 或改用 `ref` |
| 直接修改 reactive 数组索引 | Vue 3 Proxy 已支持 | Vue 2 不支持，Vue 3 没问题 |
| `ref` 模板自动解包但 JS 不行 | 模板编译时自动加 `.value` | JS 中必须写 `.value` |
| `reactive` 重新赋值整个对象 | 替换引用后不再被代理 | 用 `Object.assign(state, newObj)` 或 `ref` |

### 最佳实践

- 简单值用 `ref`，对象/数组用 `reactive`
- 解构 reactive 时用 `toRefs`
- 不需要深层响应时用 `shallowRef`/`shallowReactive` 提升性能
- 只读数据用 `readonly()` 或 `shallowReadonly()`

## 面试题

**Q1: Vue 3 为什么用 Proxy 替代 Object.defineProperty？**
> Proxy 可拦截对象的所有操作（增删改查），无需提前遍历属性；Object.defineProperty 只能劫持已有属性，新增/删除属性无法检测（需 $set/$delete）。Proxy 还原生支持数组变化和 Map/Set 等新数据结构。

**Q2: reactive 和 ref 的区别是什么？分别适用什么场景？**
> `reactive` 接收对象/数组，返回 Proxy 代理，直接访问属性无需 `.value`，但解构会丢失响应性；`ref` 接收任意类型，通过 `.value` 访问，在模板中自动解包。推荐简单值用 ref，对象/数组用 reactive，不确定时优先用 ref。

**Q3: Vue 3 依赖收集的流程是怎样的？**
> ① 组件渲染时执行 effect 函数；② 读取响应式数据触发 Proxy get 拦截；③ get 中调用 track()，将当前 effect 记录到该属性的依赖集合（Dep）中；④ 数据修改触发 set 拦截；⑤ set 中调用 trigger()，遍历 Dep 通知所有 effect 重新执行。

**Q4: 为什么 reactive 对象解构会丢失响应性？如何解决？**
> 解构得到的是原始值，脱离了 Proxy 代理，修改原始值不会触发追踪。解决方案：① `toRefs(state)` 将每个属性转为 ref；② 直接用 ref 代替 reactive；③ 不解构，直接使用 `state.name`。

---

**相关链接：**
- [[Vue核心]]
- [[React Hooks详解]]

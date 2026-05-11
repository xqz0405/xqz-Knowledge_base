---
tags:
  - Web前端
  - Intersection Observer
  - 懒加载
  - 性能优化
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Intersection Observer 与懒加载

## What — 什么是 Intersection Observer

Intersection Observer API 是浏览器提供的异步观察元素与视口（或祖先元素）交叉状态的 API。它替代了传统的 `scroll` 事件监听 + `getBoundingClientRect()` 方案，在性能和易用性上有质的飞跃。

### 核心概念

| 概念 | 说明 |
|------|------|
| target | 被观察的目标元素 |
| root | 视口或指定的祖先元素（默认为浏览器视口） |
| rootMargin | root 的边距，扩大/缩小观察区域 |
| threshold | 交叉比例阈值，触发回调的时机 |
| isIntersecting | 目标是否与 root 交叉 |
| intersectionRatio | 当前交叉比例（0~1） |

### 懒加载的定义

懒加载（Lazy Loading）是指推迟非关键资源的加载，直到它们真正需要时才加载。常见场景：图片、组件、数据列表。

### Intersection Observer vs scroll 事件

| 维度 | scroll + getBoundingClientRect | Intersection Observer |
|------|-------------------------------|----------------------|
| 执行线程 | 主线程（同步） | 观察者线程（异步） |
| 触发频率 | 每帧都可能触发 | 仅在交叉状态变化时触发 |
| 性能 | 滚动卡顿常见 | 不影响滚动性能 |
| 代码复杂度 | 手动计算位置 + 节流 | 声明式配置 |
| 兼容性 | 全浏览器 | IE 不支持，其余全支持 |

---

## Why — 为什么需要 Intersection Observer

### 1. 性能问题的根源

传统懒加载用 `scroll` 事件 + `getBoundingClientRect()` 计算。`getBoundingClientRect()` 触发强制同步布局（Forced Reflow），在滚动中频繁调用会导致严重卡顿。一个页面有 100 张懒加载图片，每次滚动都可能触发 100 次布局计算。

### 2. Intersection Observer 的性能优势

Intersection Observer 的观察逻辑运行在浏览器引擎层，不占用主线程。只有交叉状态真正变化时才回调 JavaScript，回调次数极低。

### 3. 不止懒加载

Intersection Observer 的用途远超懒加载：

| 用途 | 说明 |
|------|------|
| 图片懒加载 | 进入视口再加载 src |
| 无限滚动 | 滚动到底部加载更多 |
| 组件懒渲染 | 不可见时不渲染 DOM |
| 曝光统计 | 元素是否被用户看到 |
| 视频自动播放 | 进入视口自动播放，离开暂停 |
| 动画触发 | 滚动到元素时触发入场动画 |
| 吸顶效果 | 检测元素是否离开视口 |

---

## How — 怎么用

### 1. 基础用法

```js
// 创建观察者
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        console.log('元素进入视口:', entry.target)
        console.log('交叉比例:', entry.intersectionRatio)
      } else {
        console.log('元素离开视口:', entry.target)
      }
    })
  },
  {
    root: null,          // 默认视口
    rootMargin: '0px',   // 无边距
    threshold: 0,        // 0% 交叉时触发（刚进入/刚离开）
  }
)

// 观察目标元素
const target = document.querySelector('.lazy-image')
observer.observe(target)

// 停止观察
observer.unobserve(target)

// 断开所有观察
observer.disconnect()
```

### 2. 图片懒加载

```html
<!-- data-src 存储真实 URL，src 初始为空或占位图 -->
<img class="lazy-image" data-src="photo.jpg" src="placeholder.svg" alt="..." />
<img class="lazy-image" data-src="photo2.jpg" src="placeholder.svg" alt="..." />
```

```js
const imageObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target
        img.src = img.dataset.src
        img.classList.add('loaded')
        // 加载后停止观察
        imageObserver.unobserve(img)
      }
    })
  },
  {
    rootMargin: '200px 0px', // 提前 200px 开始加载
    threshold: 0,
  }
)

// 观察所有懒加载图片
document.querySelectorAll('.lazy-image').forEach((img) => {
  imageObserver.observe(img)
})
```

```css
.lazy-image {
  opacity: 0;
  transition: opacity 0.3s;
}

.lazy-image.loaded {
  opacity: 1;
}
```

**原生懒加载（无需 JS）**：

```html
<!-- loading="lazy" 是浏览器原生懒加载 -->
<img src="photo.jpg" loading="lazy" alt="..." />

<!-- eager = 立即加载（默认） -->
<img src="hero.jpg" loading="eager" alt="..." />
```

| 方式 | 优点 | 缺点 |
|------|------|------|
| `loading="lazy"` | 零代码、零 JS | 无法控制加载距离、无回调 |
| Intersection Observer | 精确控制、有回调 | 需要 JS |

---

### 3. 无限滚动（Infinite Scroll）

```html
<div id="list">
  <!-- 数据项 -->
</div>
<div id="sentinel">Loading...</div>
```

```js
let page = 1
let loading = false

const sentinelObserver = new IntersectionObserver(
  async (entries) => {
    if (entries[0].isIntersecting && !loading) {
      loading = true
      const items = await fetchMoreData(page)
      renderItems(items)
      page++
      loading = false
    }
  },
  {
    rootMargin: '400px', // 提前 400px 触发加载
    threshold: 0,
  }
)

sentinelObserver.observe(document.getElementById('sentinel'))

async function fetchMoreData(page) {
  const res = await fetch(`/api/items?page=${page}`)
  return res.json()
}

function renderItems(items) {
  const list = document.getElementById('list')
  items.forEach((item) => {
    const el = document.createElement('div')
    el.className = 'item'
    el.textContent = item.name
    list.appendChild(el)
  })
}
```

---

### 4. 组件懒渲染

```js
// 不可见的组件不渲染 DOM，进入视口时才渲染
class LazyComponent {
  constructor(el, renderFn) {
    this.el = el
    this.renderFn = renderFn
    this.rendered = false

    this.observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !this.rendered) {
          this.render()
          this.observer.unobserve(this.el)
        }
      },
      { rootMargin: '100px' }
    )

    this.observer.observe(this.el)
  }

  render() {
    this.el.innerHTML = this.renderFn()
    this.rendered = true
  }
}

// 使用
new LazyComponent(
  document.getElementById('heavy-chart'),
  () => renderExpensiveChart()
)
```

**Vue 组件懒渲染**：

```vue
<template>
  <div ref="container">
    <slot v-if="visible" />
    <div v-else class="placeholder" :style="{ minHeight }" />
  </div>
</template>

<script setup>
const props = defineProps({
  minHeight: { type: String, default: '200px' },
  rootMargin: { type: String, default: '200px' },
})

const container = ref(null)
const visible = ref(false)

onMounted(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) {
        visible.value = true
        observer.disconnect()
      }
    },
    { rootMargin: props.rootMargin }
  )
  observer.observe(container.value)
})
</script>
```

---

### 5. 曝光统计

```js
// 元素被用户看到超过 50% 持续 1 秒，计为一次有效曝光
const exposureObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      const el = entry.target
      const id = el.dataset.trackId

      if (entry.isIntersecting) {
        // 开始计时
        el._exposureTimer = setTimeout(() => {
          trackExposure(id)
        }, 1000)
      } else {
        // 离开视口，取消计时
        clearTimeout(el._exposureTimer)
      }
    })
  },
  {
    threshold: 0.5, // 50% 可见时触发
  }
)

function trackExposure(id) {
  // 上报曝光数据
  navigator.sendBeacon('/api/exposure', JSON.stringify({ id, time: Date.now() }))
}
```

---

### 6. 视频自动播放/暂停

```js
const videoObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      const video = entry.target
      if (entry.isIntersecting) {
        video.play().catch(() => {}) // 自动播放可能被浏览器阻止
      } else {
        video.pause()
      }
    })
  },
  {
    threshold: 0.5, // 50% 可见时播放
  }
)

document.querySelectorAll('video[data-autoplay]').forEach((video) => {
  videoObserver.observe(video)
})
```

---

### 7. 滚动入场动画

```js
const revealObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('revealed')
        revealObserver.unobserve(entry.target)
      }
    })
  },
  {
    threshold: 0.1,
    rootMargin: '0px 0px -50px 0px', // 元素进入视口 50px 后才触发
  }
)

document.querySelectorAll('.reveal-on-scroll').forEach((el) => {
  revealObserver.observe(el)
})
```

```css
.reveal-on-scroll {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.6s ease-out, transform 0.6s ease-out;
}

.reveal-on-scroll.revealed {
  opacity: 1;
  transform: translateY(0);
}

/* 尊重用户的动画偏好 */
@media (prefers-reduced-motion: reduce) {
  .reveal-on-scroll {
    opacity: 1;
    transform: none;
    transition: none;
  }
}
```

---

### 8. 吸顶效果

```js
const stickyObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      const navbar = document.querySelector('.navbar')
      if (!entry.isIntersecting) {
        navbar.classList.add('sticky')
      } else {
        navbar.classList.remove('sticky')
      }
    })
  },
  {
    threshold: 0,
    rootMargin: '-1px 0px 0px 0px', // 微调，确保触发
  }
)

// 观察一个哨兵元素（在 navbar 上方）
stickyObserver.observe(document.getElementById('sticky-sentinel'))
```

```html
<div id="sticky-sentinel"></div>
<nav class="navbar">
  <!-- 导航栏 -->
</nav>
```

---

### 9. threshold 和 rootMargin 深度理解

**threshold — 交叉比例阈值**：

```js
// threshold: 0    — 刚进入/刚离开视口时触发
// threshold: 0.5  — 50% 可见时触发
// threshold: 1    — 100% 可见时触发
// threshold: [0, 0.25, 0.5, 0.75, 1] — 多个阈值

const observer = new IntersectionObserver(callback, {
  threshold: [0, 0.25, 0.5, 0.75, 1],
})
// 元素从 0% → 25% → 50% → 75% → 100% 每个阶段都触发回调
```

**rootMargin — 扩大/缩小观察区域**：

```js
// 提前加载：在元素距离视口还有 200px 时触发
rootMargin: '200px 0px 0px 0px'  // 上方扩展 200px

// 四个方向都扩展
rootMargin: '200px'  // 等同于 '200px 200px 200px 200px'

// 缩小观察区域：元素进入视口 100px 后才触发
rootMargin: '-100px 0px 0px 0px'  // 上方缩小 100px
```

**rootMargin 语法与 CSS margin 一致**：

```
rootMargin: '10px'                  → 四边 10px
rootMargin: '10px 20px'             → 上下 10px，左右 20px
rootMargin: '10px 20px 30px'        → 上 10px，左右 20px，下 30px
rootMargin: '10px 20px 30px 40px'   → 上 10px，右 20px，下 30px，左 40px
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 回调不触发 | 元素被 `display: none` 隐藏 | 隐藏元素无法被观察，改用 `visibility: hidden` 或 `opacity: 0` |
| 首屏图片不加载 | Observer 回调有微任务延迟 | 首屏图片用 `loading="eager"` 或直接设置 src |
| 重复触发 | threshold=0 时，元素在边界会反复进入/离开 | 使用 `unobserve` 或加防抖 |
| 动态内容不观察 | 新增的 DOM 元素未被 observe | MutationObserver 监听 DOM 变化，自动观察新元素 |
| rootMargin 不生效 | 值格式错误（如 `200` 不带单位） | 必须带单位：`200px` 或 `20%` |
| SSR 中报错 | 服务端没有 IntersectionObserver | 用 `typeof IntersectionObserver !== 'undefined'` 检测 |

### 最佳实践

1. **加载后立即 unobserve**：避免持续观察已处理元素，减少回调开销。
2. **rootMargin 预加载**：图片设 `200px` rootMargin，在元素进入视口前就开始加载。
3. **首屏不懒加载**：首屏图片直接加载，避免 Observer 延迟导致白屏。
4. **尊重 prefers-reduced-motion**：动画场景中检查用户偏好。
5. **SSR 兼容**：检测 API 是否存在，降级到直接加载。
6. **使用单一 Observer**：多个元素共用一个 Observer 实例，而非每个元素创建一个。

---

## 面试题

### 1. Intersection Observer 相比 scroll 事件监听有什么性能优势？

**答**：三个核心优势：(1) **不占用主线程**——Intersection Observer 的交叉计算在浏览器引擎层（合成器线程）完成，不触发主线程的布局计算；`scroll` 事件 + `getBoundingClientRect()` 必须在主线程同步执行，每次调用都触发强制回流（Forced Reflow）；(2) **回调频率低**——Observer 只在交叉状态变化时触发回调（元素进入/离开视口），`scroll` 事件每帧触发一次（60fps = 每秒 60 次）；(3) **无需手动节流**——Observer 天然"节流"，不需要 `throttle`/`debounce`，而 `scroll` 不加节流会严重卡顿。

---

### 2. rootMargin 和 threshold 各自控制什么？如何配合使用实现"提前加载"？

**答**：`rootMargin` 扩大或缩小观察区域的边界，`threshold` 控制交叉比例达到多少时触发回调。实现"提前加载"：设置 `rootMargin: '200px 0px'`（向上扩展 200px），`threshold: 0`（刚进入扩展区域就触发）。这样元素距离视口还有 200px 时就会触发回调，提前开始加载资源。用户滚动到时资源已经加载完成，体验更流畅。

---

### 3. `loading="lazy"` 和 Intersection Observer 实现的懒加载有什么区别？

**答**：`loading="lazy"` 是浏览器原生的图片懒加载，零 JS 代码，浏览器自动决定加载时机（通常距离视口 1250-2500px 时加载）。Intersection Observer 需要手写 JS，但可以精确控制加载时机（通过 rootMargin）、监听回调、支持非图片元素（组件、视频等）。`loading="lazy"` 只适用于 `<img>` 和 `<iframe>`，Observer 适用于任何 DOM 元素。最佳实践：简单图片用 `loading="lazy"`，需要精确控制或非图片元素用 Observer。

---

### 4. 如何用 Intersection Observer 实现"有效曝光"统计（元素被看到超过 2 秒才算一次曝光）？

**答**：分两步实现：(1) 在 Observer 回调中，元素进入视口时（`isIntersecting = true`）启动一个 `setTimeout`，2 秒后执行上报逻辑；(2) 元素离开视口时（`isIntersecting = false`）清除 timer，不上报。关键细节：threshold 设为 0.5（50% 可见才算"看到"），确保不是擦边而过；用 `sendBeacon` 上报以确保页面卸载时数据不丢失；对频繁进出的场景（如快速滚动），只有持续可见超过 2 秒的才会被上报。

---

### 5. Intersection Observer 能检测 `display: none` 的元素吗？为什么？

**答**：不能。`display: none` 的元素不参与布局，没有盒模型（零尺寸、零位置），浏览器无法计算它与 root 的交叉区域。Intersection Observer 的 `isIntersecting` 永远为 false。如果需要"隐藏但可观察"的元素，用 `visibility: hidden`（保持布局占位）或 `opacity: 0`（保持布局和交互占位），这两种方式下元素仍有尺寸，可以被 Observer 正常检测。

---

### 6. 多个元素应该共用一个 Observer 还是各自创建 Observer？为什么？

**答**：应该共用一个 Observer 实例。原因：(1) 每个 Observer 实例会在浏览器引擎中注册一个观察者，多个实例意味着多次交叉计算；(2) 共用一个实例时，浏览器可以对同一 root 下的所有 target 进行批量计算，效率更高；(3) 减少 GC 压力——一个实例 vs N 个实例。创建方式：一个 Observer 观察所有目标元素，回调中通过 `entry.target` 区分不同元素。只有当不同组元素需要不同的 `root`/`rootMargin`/`threshold` 配置时，才需要创建多个 Observer。

---

### 7. Intersection Observer 在 SPA 中有什么特殊问题？如何解决？

**答**：SPA 中的特殊问题：(1) **路由切换后观察失效**——切换页面后旧 Observer 仍在观察已不存在的 DOM 元素，新页面的元素未被观察。解决：路由切换时 `disconnect()` 旧 Observer，在新页面 `mounted` 时重新 `observe`；(2) **虚拟滚动兼容**——虚拟滚动中 DOM 元素频繁创建/销毁，需要在元素挂载时 observe、卸载时 unobserve；(3) **Tab 切换**——隐藏的 Tab 中元素不在 DOM 中，无法观察。解决：Tab 激活时重新 observe。通用方案：配合 MutationObserver 自动观察新增 DOM 元素。

---

### 8. 如何实现一个通用的懒加载指令（Vue directive / React hook）？

**答**：核心思路是封装 Observer 为可复用的指令/hook，自动处理生命周期。Vue directive：`bind` 时创建 Observer 并 observe 元素，`unbind` 时 unobserve；支持 `rootMargin` 和 `threshold` 通过 binding value 传入。React hook：`useLazyLoad(ref, options)` 内部用 `useEffect` 创建 Observer，依赖 `ref.current`，cleanup 时 disconnect。关键细节：(1) 单例模式——所有使用同一配置的元素共享一个 Observer 实例；(2) SSR 安全——检测 `typeof IntersectionObserver !== 'undefined'`；(3) 首屏优化——对初始可见的元素立即触发回调，不等待 Observer。

---

## 相关链接

- [Intersection Observer — MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
- [Lazy Loading — web.dev](https://web.dev/articles/lazy-loading-images)
- [loading="lazy" — MDN](https://developer.mozilla.org/zh-CN/docs/Web/Performance/Lazy_loading)
- [IntersectionObserver Polyfill — W3C](https://github.com/w3c/IntersectionObserver)

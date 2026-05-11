---
tags:
  - Web前端
  - View Transitions
  - SPA
  - MPA
  - 动画
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# View Transitions API

## What — 什么是 View Transitions API

View Transitions API 是浏览器原生的视图过渡动画 API，让页面切换、DOM 更新时的动画效果变得极其简单。它自动截图旧状态的 DOM、更新 DOM、截图新状态，然后自动计算差异并生成过渡动画。

### 核心概念

| 概念 | 说明 |
|------|------|
| View Transition | 一次完整的视图过渡过程 |
| Old Snapshot | 过渡前的 DOM 快照 |
| New Snapshot | DOM 更新后的快照 |
| `view-transition-name` | 为元素命名，使新旧状态中同名元素做动画 |
| Pseudo-elements | `::view-transition-*` 系列伪元素控制动画 |

### 两种模式

| 模式 | 适用 | 触发方式 |
|------|------|----------|
| Same-document（SPA） | 单页应用内状态切换 | `document.startViewTransition()` |
| Cross-document（MPA） | 多页应用页面跳转 | `@view-transition { navigation: auto; }` |

### 与传统方案对比

| 维度 | Framer Motion / GSAP | View Transitions API |
|------|---------------------|---------------------|
| 依赖 | 第三方库（~30KB） | 浏览器原生 |
| 原理 | JS 计算动画帧 | 浏览器合成器层 |
| 性能 | 主线程 | 合成器线程（更流畅） |
| 跨页面 | 不支持 | MPA 模式原生支持 |
| 浏览器兼容 | 全浏览器 | Chrome 111+ / Safari 18+ |

---

## Why — 为什么需要 View Transitions API

### 1. 视图过渡的痛点

传统方案中，页面切换动画需要：(1) 手动计算新旧元素的位置和尺寸差异；(2) 用 `position: absolute` + `transform` 做位移动画；(3) 处理 z-index、opacity 等中间状态；(4) 在 SPA 中还要手动管理路由切换时机。这些代码复杂且容易出 bug。

### 2. 原生方案的优势

View Transitions API 只需告诉浏览器"哪些元素是同一个"，浏览器自动处理截图、差异计算、动画生成。开发者只需关注"哪个元素在过渡中保持连续"，无需手动写动画。

### 3. 跨页面过渡成为可能

SPA 的视图过渡已有库支持，但 MPA（传统多页应用）的跨页面动画几乎不可能——两个独立的 HTML 文档无法共享动画状态。View Transitions API 的 MPA 模式首次让跨页面过渡成为现实。

### 优缺点

- ✅ 优点：零依赖、合成器线程执行、跨页面过渡、代码极简
- ❌ 缺点：浏览器兼容性有限、自定义动画需了解伪元素结构、MPA 模式需浏览器支持

---

## How — 怎么用

### 1. SPA 模式 — 基础用法

```js
// 最简单的用法：淡入淡出
document.startViewTransition(() => {
  // 在回调中更新 DOM
  updateTheDOMSomehow()
})
```

浏览器执行流程：(1) 截取当前 DOM 快照（Old Snapshot）；(2) 执行回调，更新 DOM；(3) 截取新 DOM 快照（New Snapshot）；(4) 对比差异，播放过渡动画。

**列表切换示例**：

```html
<div class="container">
  <nav>
    <button onclick="switchTab('photos')">Photos</button>
    <button onclick="switchTab('albums')">Albums</button>
  </nav>
  <div id="content">
    <!-- 动态内容 -->
  </div>
</div>

<script>
function switchTab(tab) {
  // 包裹 DOM 更新
  const transition = document.startViewTransition(() => {
    document.getElementById('content').innerHTML =
      tab === 'photos' ? getPhotosHTML() : getAlbumsHTML()
  })
}
</script>
```

**异步更新（如 fetch 数据）**：

```js
async function switchView(userId) {
  const transition = document.startViewTransition(async () => {
    // 回调可以是 async
    const data = await fetch(`/api/users/${userId}`).then(r => r.json())
    document.getElementById('content').innerHTML = renderUser(data)
  })
}
```

---

### 2. SPA 模式 — 共享元素过渡（Cross-fade）

为元素命名后，新旧状态中同名的元素会做"共享过渡"动画（如图片从列表放大到详情页）。

```css
/* 给需要做过渡动画的元素命名 */
.photo-card img {
  view-transition-name: var(--photo-id);
}

/* 每张图片用不同的名字 */
.photo-card:nth-child(1) img { view-transition-name: photo-1; }
.photo-card:nth-child(2) img { view-transition-name: photo-2; }
.photo-card:nth-child(3) img { view-transition-name: photo-3; }
```

```js
function openPhoto(photoId) {
  // 设置当前图片的过渡名
  document.querySelector(`[data-id="${photoId}"] img`)
    .style.viewTransitionName = 'photo-active'

  const transition = document.startViewTransition(() => {
    // 更新 DOM 到详情页
    document.getElementById('content').innerHTML = getDetailHTML(photoId)
    // 详情页的大图也用相同的过渡名
    document.querySelector('.detail-image')
      .style.viewTransitionName = 'photo-active'
  })
}
```

**动态设置 view-transition-name**：

```js
// 用 CSS 变量动态传递
function openCard(cardId) {
  // 列表中的卡片
  const card = document.querySelector(`[data-id="${cardId}"]`)
  card.style.setProperty('--vt-name', `card-${cardId}`)

  const transition = document.startViewTransition(() => {
    updateToDetailPage(cardId)
    // 详情页中对应的元素也设置相同名称
    const detail = document.querySelector('.detail-hero')
    detail.style.setProperty('--vt-name', `card-${cardId}`)
  })
}
```

```css
/* 用 CSS 变量做过渡名 */
[data-id] {
  view-transition-name: var(--vt-name, none);
}

.detail-hero {
  view-transition-name: var(--vt-name, none);
}
```

---

### 3. SPA 模式 — 自定义动画

View Transitions API 生成一组伪元素树，可以自定义每个部分的动画：

```
::view-transition
├── ::view-transition-group(root)
│   ├── ::view-transition-old(root)     — 旧快照
│   └── ::view-transition-new(root)     — 新快照
├── ::view-transition-group(photo-active)
│   ├── ::view-transition-old(photo-active)
│   └── ::view-transition-new(photo-active)
```

```css
/* 默认过渡动画是 cross-fade + 250ms */

/* 自定义根容器过渡 — 改为淡入淡出 */
::view-transition-old(root) {
  animation: 0.3s ease-out both fade-out;
}

::view-transition-new(root) {
  animation: 0.3s ease-in both fade-in;
}

@keyframes fade-out {
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
}

/* 自定义共享元素的过渡 — 改为缩放+移动 */
::view-transition-group(photo-active) {
  animation-duration: 0.5s;
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

::view-transition-old(photo-active) {
  animation: 0.3s ease-out both scale-down;
}

::view-transition-new(photo-active) {
  animation: 0.3s ease-in both scale-up;
  animation-delay: 0.2s; /* 延迟，让旧图先缩小 */
}

@keyframes scale-down {
  to { opacity: 0; transform: scale(0.8); }
}

@keyframes scale-up {
  from { opacity: 0; transform: scale(1.1); }
}

/* 暗色模式下的过渡 */
.dark::view-transition-old(root) {
  animation: 0.3s ease-out both fade-out-dark;
}

.dark::view-transition-new(root) {
  animation: 0.3s ease-in both fade-in-dark;
}
```

---

### 4. MPA 模式 — 跨页面过渡

```css
/* 在全局 CSS 中启用 */
@view-transition {
  navigation: auto;
}
```

只需这一行 CSS，所有同源页面跳转都会自动做淡入淡出过渡。

**跨页面的共享元素过渡**：

```css
/* 页面 A 中的元素 */
.hero-image {
  view-transition-name: hero;
}

.page-title {
  view-transition-name: title;
}

/* 页面 B 中同名元素自动做共享过渡 */
.hero-banner {
  view-transition-name: hero;
}

.detail-title {
  view-transition-name: title;
}
```

**自定义跨页面动画**：

```css
@view-transition {
  navigation: auto;
}

/* 阻止特定导航的过渡 */
@view-transition {
  navigation: auto;
}

/* 返回导航使用不同的动画 */
::view-transition-old(root) {
  animation: 0.25s ease-out both slide-out-left;
}

::view-transition-new(root) {
  animation: 0.25s ease-in both slide-in-right;
}

/* 可以用 :active-view-transition-type 区分前进/后退 */
@keyframes slide-out-left {
  to { transform: translateX(-30%); opacity: 0; }
}

@keyframes slide-in-right {
  from { transform: translateX(30%); opacity: 0; }
}
```

---

### 5. 与 React/Vue 集成

```tsx
// React 集成
import { useViewTransition } from './hooks/useViewTransition'

function useViewTransition() {
  const startTransition = useCallback((updateDOM: () => void) => {
    if (!document.startViewTransition) {
      updateDOM()
      return
    }
    document.startViewTransition(() => {
      // React 需要等待状态更新完成
      flushSync(() => {
        updateDOM()
      })
    })
  }, [])

  return startTransition
}

// 使用
function PhotoGrid() {
  const [selected, setSelected] = useState(null)
  const transition = useViewTransition()

  function openPhoto(id) {
    transition(() => setSelected(id))
  }

  return (
    <div>
      {photos.map(photo => (
        <img
          key={photo.id}
          src={photo.url}
          style={{ viewTransitionName: selected === photo.id ? 'active' : 'none' }}
          onClick={() => openPhoto(photo.id)}
        />
      ))}
    </div>
  )
}
```

```vue
<!-- Vue 集成 -->
<template>
  <div>
    <div v-for="photo in photos" :key="photo.id">
      <img
        :src="photo.url"
        :style="{ viewTransitionName: selected === photo.id ? 'active' : 'none' }"
        @click="selectPhoto(photo.id)"
      />
    </div>
  </div>
</template>

<script setup>
import { nextTick } from 'vue'

const selected = ref(null)

async function selectPhoto(id) {
  if (!document.startViewTransition) {
    selected.value = id
    return
  }

  const transition = document.startViewTransition(async () => {
    selected.value = id
    await nextTick() // 等待 Vue 更新 DOM
  })
}
</script>
```

---

### 6. 高级技巧

**阻止过渡**：

```js
// 条件性跳过过渡
if (shouldSkipTransition) {
  updateDOM() // 直接更新，不触发过渡
  return
}

document.startViewTransition(() => updateDOM())
```

**过渡类型（Transition Types）**：

```js
// 设置过渡类型，用于区分不同方向的动画
document.startViewTransition({
  update: () => updateDOM(),
  types: ['forward'],  // 或 ['back']
})
```

```css
/* 前进：新页面从右滑入 */
:active-view-transition-type(forward)::view-transition-new(root) {
  animation: 0.3s ease-in both slide-in-right;
}

:active-view-transition-type(forward)::view-transition-old(root) {
  animation: 0.3s ease-out both slide-out-left;
}

/* 后退：新页面从左滑入 */
:active-view-transition-type(back)::view-transition-new(root) {
  animation: 0.3s ease-in both slide-in-left;
}

:active-view-transition-type(back)::view-transition-old(root) {
  animation: 0.3s ease-out both slide-out-right;
}
```

**等待过渡完成**：

```js
const transition = document.startViewTransition(() => {
  updateDOM()
})

// 过渡完成后执行
transition.finished.then(() => {
  console.log('Transition complete')
  // 清理、重置状态
})

// 过渡准备就绪（截图完成，动画即将开始）
transition.ready.then(() => {
  console.log('Transition ready')
})
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 过渡不生效 | 浏览器不支持 | 检测 `document.startViewTransition` 再降级 |
| React 状态更新白屏 | React 异步更新，截图时机不对 | 用 `flushSync` 强制同步更新 |
| 共享元素闪烁 | 两个同名元素同时存在 | 确保任何时刻只有一个元素有某个 `view-transition-name` |
| 图片过渡变形 | 图片宽高比不同 | 设置 `object-fit: cover` 保持比例 |
| MPA 过渡不生效 | 服务器未返回正确 Content-Type | 确认页面是同源的 HTML 文档 |
| 性能卡顿 | 过渡元素过多 | 只给关键元素设置 `view-transition-name` |

### 最佳实践

1. **降级方案必须**：`if (!document.startViewTransition) { updateDOM(); return }`
2. **React 用 flushSync**：确保 DOM 同步更新后再截图。
3. **view-transition-name 唯一**：同一时刻不能有两个同名元素。
4. **控制过渡元素数量**：只给视觉上需要连续的元素命名，其余用默认淡入淡出。
5. **MPA 模式先测试**：跨页面过渡浏览器支持尚不完全，需充分测试。

---

## 面试题

### 1. View Transitions API 的工作原理是什么？

**答**：View Transitions API 的工作流程分四步：(1) **旧快照**——浏览器对当前 DOM 进行截图（像素级），记录每个有 `view-transition-name` 的元素位置和样式；(2) **DOM 更新**——执行回调函数，更新 DOM 到新状态；(3) **新快照**——对新 DOM 截图；(4) **动画合成**——浏览器对比新旧快照中同名元素的位置/尺寸/样式差异，自动生成过渡动画。整个动画运行在合成器线程，不阻塞主线程。对于没有命名的元素，默认做整体淡入淡出。

---

### 2. SPA 模式下 View Transitions 与 React/Vue 的状态更新有什么冲突？如何解决？

**答**：冲突在于 React/Vue 的状态更新是异步的（批量更新），而 `startViewTransition` 的回调执行后浏览器立即截图——如果 DOM 还没更新，截图的就是旧状态。解决方法：React 中使用 `flushSync()` 包裹状态更新，强制同步刷新 DOM；Vue 中使用 `nextTick()` 等待 DOM 更新完成。核心原则是确保回调函数返回时 DOM 已经是新状态。

---

### 3. `view-transition-name` 的唯一性规则是什么？为什么必须唯一？

**答**：任何时刻，页面中只能有一个元素拥有某个 `view-transition-name` 值。如果有两个元素同名，浏览器会忽略该名称（不做过渡动画）。原因：浏览器通过名称匹配新旧快照中的元素，如果旧快照有 2 个 `hero`、新快照有 1 个 `hero`，浏览器无法确定哪个旧元素应该过渡到新元素。解决方案：动态设置名称——列表中只有被点击的卡片设置名称，其余设为 `none`；或在 `startViewTransition` 回调中修改名称。

---

### 4. MPA 模式的 View Transitions 是如何实现跨页面动画的？

**答**：MPA 模式下，当导航发生时：(1) 浏览器在离开前截取当前页面的快照；(2) 正常加载新页面；(3) 新页面渲染完成后截取快照；(4) 在两个快照之间播放过渡动画。关键挑战是两个页面是独立的文档，无法共享 JavaScript 状态。解决方案是通过 `view-transition-name` 匹配——两个页面中同名元素自动做共享过渡。CSS 中的 `@view-transition { navigation: auto; }` 一行启用，无需任何 JS。这是之前任何库都无法做到的——跨文档的动画过渡。

---

### 5. View Transitions API 的伪元素树结构是什么？如何自定义动画？

**答**：伪元素树结构为 `::view-transition` → `::view-transition-group(name)` → `::view-transition-old(name)` + `::view-transition-new(name)`。`::view-transition` 是容器，`group` 是单个过渡元素的容器（处理位置和尺寸变化），`old` 是旧快照，`new` 是新快照。自定义动画：对 `old` 设置退场动画（如淡出、缩小），对 `new` 设置入场动画（如淡入、放大），对 `group` 设置位置/尺寸过渡的时长和缓动。默认动画是 250ms cross-fade。

---

### 6. 如何实现前进/后退不同方向的过渡动画？

**答**：使用 Transition Types 区分方向：(1) 调用 `document.startViewTransition({ update, types: ['forward'] })` 设置类型；(2) CSS 中用 `:active-view-transition-type(forward)` 选择器定制前进动画（如新页面从右滑入），用 `:active-view-transition-type(back)` 定制后退动画（如新页面从左滑入）；(3) 在路由拦截中判断是前进还是后退（通过 history 索引或导航方向），设置不同 type。没有 type 时使用默认动画。

---

### 7. View Transitions API 和 FLIP 动画技术有什么关系？

**答**：View Transitions API 可以理解为浏览器原生实现的 FLIP（First-Last-Invert-Play）。FLIP 技术的手动步骤：(1) 记录元素初始位置（First）；(2) 更新 DOM，记录最终位置（Last）；(3) 计算差值，用 transform 反向偏移回初始位置（Invert）；(4) 移除 transform 让元素动画到最终位置（Play）。View Transitions API 自动完成了这四步：截图 = First/Last，差异计算 = Invert，过渡动画 = Play。API 还提供了 FLIP 不具备的能力——跨页面过渡、像素级快照（而非基于布局计算）。

---

### 8. View Transitions API 在生产环境中有哪些注意事项？

**答**：六个注意事项：(1) **降级必须**——不支持的浏览器直接跳过过渡，用 `if (!document.startViewTransition)` 检测；(2) **避免 `view-transition-name` 冲突**——列表项动态命名，未选中项设为 `none`；(3) **控制过渡范围**——不要给大量元素设置 name，只给视觉关键元素设置；(4) **尊重用户偏好**——`@media (prefers-reduced-motion: reduce)` 时禁用过渡；(5) **SSR 场景**——SPA 模式需要客户端 JS，MPA 模式的跨页面过渡需要两个页面都声明 `@view-transition`；(6) **性能监控**——过渡动画虽然运行在合成器线程，但截图操作有开销，大量 DOM 变更时注意 `transition.ready` 的延迟。

---

## 相关链接

- [View Transitions API — MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/View_Transitions_API)
- [View Transitions — Chrome Developers](https://developer.chrome.com/docs/web-platform/view-transitions/)
- [Cross-document View Transitions](https://developer.chrome.com/docs/web-platform/view-transitions/cross-document)
- [Smooth transitions with View Transitions API](https://web.dev/articles/view-transitions)

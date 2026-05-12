---
tags:
  - Web前端
  - CSS
  - 容器查询
  - 响应式
  - 组件化
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# CSS容器查询

## What — 是什么

> CSS 容器查询（Container Queries）允许组件根据其父容器的尺寸（而非视口宽度）来调整样式，是实现真正的组件级响应式设计的核心特性。与传统 `@media` 媒体查询基于视口不同，容器查询让同一个组件在不同宽度的容器中自动适配，无需依赖页面整体布局。

**核心概念：**

- **容器上下文（Container Context）**：通过 `container-type` 声明一个元素为查询容器，使其尺寸可被后代查询
- **容器名称（Container Name）**：通过 `container-name` 给容器命名，后代可指定查询哪个容器
- **容器查询（Container Query）**：使用 `@container` 规则，根据容器尺寸条件应用样式
- **容器查询单位（Container Query Units）**：`cqw`/`cqh`/`cqi`/`cqb`/`cqmin`/`cqmax`，相对于容器尺寸计算
- **样式查询（Style Queries）**：`@container style(--xxx)` 根据容器自定义属性值应用样式（实验性）

**与媒体查询对比：**

```
┌─────────────────────────────────────────────────────────────┐
│                    媒体查询 @media                            │
│  基于视口宽度（window.innerWidth）                            │
│                                                             │
│  ┌──────────────────── Viewport ──────────────────────────┐ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │ │
│  │  │ 组件A     │  │ 组件B     │  │ 组件C     │            │ │
│  │  │ 宽300px  │  │ 宽600px  │  │ 宽300px  │            │ │
│  │  │ 竖排 ❌  │  │ 竖排 ❌  │  │ 竖排 ❌  │            │ │
│  │  └──────────┘  └──────────┘  └──────────┘            │ │
│  │  视口宽度1024px → 所有组件统一响应 → 组件C空间不足但仍横向│ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    容器查询 @container                        │
│  基于父容器尺寸                                              │
│                                                             │
│  ┌──────────────────── Viewport ──────────────────────────┐ │
│  │  ┌──────────┐  ┌──────────────────────────┐           │ │
│  │  │ 容器1     │  │ 容器2                     │           │ │
│  │  │ 宽300px  │  │ 宽600px                   │           │ │
│  │  │┌────────┐│  │┌────────────┬───────────┐ │           │ │
│  │  ││组件A   ││  ││ 组件B      │ 组件C      │ │           │ │
│  │  ││竖排 ✅ ││  ││ 宽400px   │ 宽150px   │ │           │ │
│  │  │└────────┘│  ││ 横排 ✅   │ 竖排 ✅   │ │           │ │
│  │  └──────────┘  │└────────────┴───────────┘ │           │ │
│  │                 └──────────────────────────┘           │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

**浏览器支持：**

| 浏览器 | 最低版本 | 支持状态 |
|--------|---------|---------|
| Chrome | 105+ | 完整支持 |
| Firefox | 110+ | 完整支持 |
| Safari | 16+ | 完整支持 |
| Edge | 105+ | 完整支持 |
| iOS Safari | 16+ | 完整支持 |
| Samsung Internet | 16+ | 完整支持 |

## Why — 为什么

**适用场景：**

- 组件库/设计系统中，组件需要在不同布局位置自适应
- 侧边栏 + 主内容区中，同一组件在窄侧边栏竖排、宽主内容区横排
- 响应式卡片：宽容器横向布局、窄容器纵向堆叠
- Dashboard 仪表盘：同一组件在网格的不同单元格中适配
- CMS 内容区：组件不知道自己会被放在多宽的容器中

**媒体查询 vs 容器查询：**

| 维度 | @media | @container |
|------|--------|-----------|
| 查询基准 | 视口宽度 | 父容器尺寸 |
| 组件独立性 | 差（依赖页面全局布局） | 强（组件自包含） |
| 复用性 | 低（同一组件不同位置需不同断点） | 高（同一组件自动适配） |
| 嵌套场景 | 无法处理 | 天然支持 |
| 性能 | 基于视口，触发少 | 基于容器，需监听尺寸变化 |
| 浏览器支持 | 全部 | 2022年主流（Chrome 105+） |

**优缺点：**

- ✅ 优点：
  - 真正的组件级响应式，不再依赖视口
  - 组件完全自包含，可自由放置在页面任何位置
  - 减少为不同布局编写重复样式
  - 与设计系统/组件库理念完美契合
  - 容器查询单位实现流式排版
- ❌ 缺点：
  - 旧浏览器不支持（IE 全不支持，需渐进增强）
  - 性能开销：容器尺寸变化需浏览器持续计算
  - 不支持查询非祖先元素（只能向上查找容器）
  - 样式查询（Style Queries）仍为实验性特性
  - 嵌套容器查询可能导致性能问题

## How — 怎么用

### 定义容器

```css
/* 方式1：简写属性 */
.card-wrapper {
  container: card / inline-size;
  /* container-name: card */
  /* container-type: inline-size */
}

/* 方式2：分开写 */
.sidebar {
  container-name: sidebar;
  container-type: inline-size;
}

.main-content {
  container-name: main;
  container-type: inline-size;
}

/* 方式3：只声明容器类型（不命名） */
.layout {
  container-type: inline-size;
}
```

**container-type 取值：**

| 类型 | 说明 | 可查询的尺寸 |
|------|------|------------|
| `inline-size` | 查询行内方向尺寸（宽度） | `inline-size` |
| `size` | 查询宽度和高度 | `inline-size` + `block-size` |
| `normal` | 默认值，不建立容器上下文 | 无 |

> 建议：大多数场景用 `inline-size` 即可，`size` 会同时监听两个方向的尺寸变化，性能开销更大。

### 基础容器查询

```css
/* 定义容器 */
.card-container {
  container-type: inline-size;
}

/* 基础用法：窄容器竖排，宽容器横排 */
.card {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* 容器宽度 ≥ 400px 时切换为横向布局 */
@container (min-width: 400px) {
  .card {
    flex-direction: row;
    align-items: center;
  }

  .card__image {
    width: 200px;
    flex-shrink: 0;
  }

  .card__content {
    flex: 1;
  }
}

/* 容器宽度 ≥ 600px 时更宽松的布局 */
@container (min-width: 600px) {
  .card {
    gap: 24px;
    padding: 32px;
  }

  .card__image {
    width: 280px;
  }

  .card__title {
    font-size: 1.5rem;
  }
}
```

```html
<div class="sidebar">
  <!-- sidebar 宽度 280px → card 竖排 -->
  <div class="card-container">
    <div class="card">
      <img class="card__image" src="photo.jpg" alt="">
      <div class="card__content">
        <h3 class="card__title">标题</h3>
        <p class="card__desc">描述文字</p>
      </div>
    </div>
  </div>
</div>

<div class="main-content">
  <!-- main 宽度 700px → card 横排 -->
  <div class="card-container">
    <div class="card">
      <img class="card__image" src="photo.jpg" alt="">
      <div class="card__content">
        <h3 class="card__title">标题</h3>
        <p class="card__desc">描述文字</p>
      </div>
    </div>
  </div>
</div>
```

### 命名容器查询

```css
/* 当页面有多个容器时，用名称指定查询目标 */
.sidebar {
  container-name: sidebar;
  container-type: inline-size;
}

.main {
  container-name: main;
  container-type: inline-size;
}

/* 查询 sidebar 容器 */
@container sidebar (min-width: 300px) {
  .widget {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}

/* 查询 main 容器 */
@container main (min-width: 600px) {
  .article-card {
    display: grid;
    grid-template-columns: 200px 1fr;
    gap: 24px;
  }
}
```

**查找规则：** `@container` 会向上查找最近的匹配 `container-name` 的祖先元素。如果未指定名称，查找最近的声明了 `container-type` 的祖先。

### 容器查询单位

```css
/* 容器查询单位相对于查询容器的尺寸计算 */

/* cqw — 容器宽度的 1% */
/* cqh — 容器高度的 1% */
/* cqi — 容器内联尺寸的 1%（水平流 = 宽度） */
/* cqb — 容器块尺寸的 1%（水平流 = 高度） */
/* cqmin — min(cqi, cqb) */
/* cqmax — max(cqi, cqb) */

.card-container {
  container-type: inline-size;
}

.card {
  /* 字体大小随容器宽度流式缩放 */
  font-size: clamp(0.875rem, 2cqi, 1.25rem);

  /* 内边距随容器宽度调整 */
  padding: clamp(8px, 3cqi, 24px);

  /* 标题字号流式 */
  --title-size: clamp(1rem, 3cqi, 2rem);
}

.card__title {
  font-size: var(--title-size);
}

.card__gap {
  /* 间距随容器宽度 */
  gap: clamp(8px, 2cqi, 16px);
}
```

**单位对比：**

| 单位 | 基准 | 典型用途 |
|------|------|---------|
| `vw` / `vh` | 视口宽度/高度 | 全局布局 |
| `%` | 父元素 | 相对布局 |
| `cqw` / `cqh` | 容器宽度/高度 | 组件内流式排版 |
| `cqi` / `cqb` | 容器内联/块尺寸 | 方向无关的流式排版 |
| `cqmin` / `cqmax` | 容器较小/较大尺寸 | 自适应极限值 |

### 实战：响应式卡片组件

```css
/* 完整的响应式卡片：从紧凑到宽松四档 */
.card-container {
  container: card / inline-size;
}

.card {
  display: flex;
  flex-direction: column;
  border-radius: 12px;
  overflow: hidden;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card__image {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}

.card__body {
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.card__tag {
  display: inline-block;
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 4px;
  background: #e6f4ff;
  color: #1677ff;
  align-self: flex-start;
}

.card__title {
  font-size: 1rem;
  font-weight: 600;
  line-height: 1.4;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card__desc {
  font-size: 0.875rem;
  color: #666;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card__meta {
  display: flex;
  gap: 12px;
  font-size: 0.75rem;
  color: #999;
  margin-top: auto;
}

/* 第一档：≥ 350px — 增加内边距和字号 */
@container card (min-width: 350px) {
  .card__body {
    padding: 16px;
    gap: 10px;
  }

  .card__title {
    font-size: 1.125rem;
  }

  .card__desc {
    -webkit-line-clamp: 4;
  }
}

/* 第二档：≥ 500px — 横向布局 */
@container card (min-width: 500px) {
  .card {
    flex-direction: row;
  }

  .card__image {
    width: 200px;
    aspect-ratio: auto;
    height: 100%;
  }

  .card__body {
    padding: 20px;
    gap: 12px;
  }
}

/* 第三档：≥ 700px — 更宽的图片和更宽松的间距 */
@container card (min-width: 700px) {
  .card__image {
    width: 300px;
  }

  .card__body {
    padding: 28px;
    gap: 16px;
  }

  .card__title {
    font-size: 1.375rem;
  }

  .card__desc {
    -webkit-line-clamp: unset;
  }
}
```

### 实战：响应式导航

```css
.nav-container {
  container: nav / inline-size;
}

.nav {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
}

.nav__logo {
  font-size: 1.25rem;
  font-weight: 700;
  margin-right: auto;
}

/* 窄容器：汉堡菜单 */
.nav__menu-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border: none;
  background: none;
  cursor: pointer;
}

.nav__links {
  display: none;
  flex-direction: column;
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  padding: 16px;
  gap: 12px;
}

.nav__links.open {
  display: flex;
}

.nav__link {
  font-size: 0.875rem;
  color: #333;
  padding: 8px 12px;
  border-radius: 8px;
  text-decoration: none;
}

/* 宽容器：水平导航 */
@container nav (min-width: 600px) {
  .nav__menu-toggle {
    display: none;
  }

  .nav__links {
    display: flex;
    flex-direction: row;
    position: static;
    background: none;
    box-shadow: none;
    padding: 0;
    gap: 4px;
  }

  .nav__link {
    font-size: 0.875rem;
  }

  .nav__link:hover {
    background: #f5f5f5;
  }
}

/* 更宽容器：增加间距和用户信息 */
@container nav (min-width: 900px) {
  .nav {
    gap: 16px;
    padding: 12px 24px;
  }

  .nav__links {
    gap: 8px;
  }

  .nav__link {
    font-size: 1rem;
    padding: 8px 16px;
  }
}
```

### 实战：Dashboard 网格自适应

```css
.dashboard {
  container: dashboard / inline-size;
  display: grid;
  gap: 16px;
}

/* 窄容器：单列 */
.dashboard {
  grid-template-columns: 1fr;
}

/* ≥ 600px：两列 */
@container dashboard (min-width: 600px) {
  .dashboard {
    grid-template-columns: 1fr 1fr;
  }

  .widget--wide {
    grid-column: span 2;
  }
}

/* ≥ 900px：三列 */
@container dashboard (min-width: 900px) {
  .dashboard {
    grid-template-columns: repeat(3, 1fr);
  }

  .widget--wide {
    grid-column: span 2;
  }

  .widget--full {
    grid-column: span 3;
  }
}

/* ≥ 1200px：四列 + 紧凑间距 */
@container dashboard (min-width: 1200px) {
  .dashboard {
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
  }

  .widget--wide {
    grid-column: span 2;
  }

  .widget--full {
    grid-column: span 4;
  }
}
```

### 实战：表单布局自适应

```css
.form-container {
  container: form / inline-size;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-row {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.form-field {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.form-field label {
  font-size: 0.875rem;
  font-weight: 500;
  color: #333;
}

.form-field input,
.form-field select {
  padding: 8px 12px;
  border: 1px solid #d9d9d9;
  border-radius: 6px;
  font-size: 0.875rem;
}

/* ≥ 480px：同行字段并排 */
@container form (min-width: 480px) {
  .form-row {
    flex-direction: row;
  }

  .form-row .form-field {
    flex: 1;
  }
}

/* ≥ 640px：标签和输入框同行 */
@container form (min-width: 640px) {
  .form-field {
    flex-direction: row;
    align-items: center;
  }

  .form-field label {
    width: 120px;
    flex-shrink: 0;
    text-align: right;
    padding-right: 12px;
  }

  .form-field input,
  .form-field select {
    flex: 1;
  }
}
```

### 样式查询（Style Queries）

```css
/* 样式查询：根据容器的自定义属性值应用样式 */
/* 注意：仍为实验性特性，Chrome 111+ 部分支持 */

.theme-container {
  --theme: light;
  container-type: inline-size;
}

/* 容器 --theme 为 dark 时 */
@container style(--theme: dark) {
  .card {
    background: #1a1a2e;
    color: #e0e0e0;
  }

  .card__title {
    color: #fff;
  }
}

/* 容器 --theme 为 light 时 */
@container style(--theme: light) {
  .card {
    background: #fff;
    color: #333;
  }
}

/* 根据容器 --layout 属性切换布局 */
.grid-container {
  --layout: list;
  container-type: inline-size;
}

@container style(--layout: grid) {
  .items {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 16px;
  }
}

@container style(--layout: list) {
  .items {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
}
```

```html
<!-- 动态切换布局：只需改 CSS 变量 -->
<div class="grid-container" style="--layout: grid">
  <div class="items">...</div>
</div>

<div class="grid-container" style="--layout: list">
  <div class="items">...</div>
</div>
```

### 容器查询 + 媒体查询配合

```css
/* 容器查询处理组件内部布局 */
/* 媒体查询处理全局页面结构 */

.page {
  display: grid;
  gap: 24px;
}

/* 全局：窄视口单列，宽视口侧边栏+主内容 */
@media (max-width: 768px) {
  .page {
    grid-template-columns: 1fr;
  }
}

@media (min-width: 769px) {
  .page {
    grid-template-columns: 280px 1fr;
  }
}

/* 侧边栏：作为容器 */
.sidebar {
  container-type: inline-size;
}

/* 侧边栏内的组件自适应 */
@container (min-width: 260px) {
  .sidebar-widget {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}

/* 主内容区：作为容器 */
.main {
  container-type: inline-size;
}

/* 主内容区的卡片自适应 */
@container (min-width: 500px) {
  .article-card {
    flex-direction: row;
  }
}
```

### React/Vue 中使用容器查询

```tsx
// React 组件中使用容器查询
import './Card.css';

interface CardProps {
  title: string;
  description: string;
  image: string;
  tag?: string;
}

export function Card({ title, description, image, tag }: CardProps) {
  return (
    <div className="card-container">
      <div className="card">
        <img className="card__image" src={image} alt={title} />
        <div className="card__body">
          {tag && <span className="card__tag">{tag}</span>}
          <h3 className="card__title">{title}</h3>
          <p className="card__desc">{description}</p>
        </div>
      </div>
    </div>
  );
}

// 使用：Card 自动根据父容器宽度适配
// <div style={{ width: '300px' }}><Card ... /></div>  → 竖排
// <div style={{ width: '600px' }}><Card ... /></div>  → 横排
```

```vue
<!-- Vue 组件 -->
<template>
  <div class="card-container">
    <div class="card">
      <img class="card__image" :src="image" :alt="title" />
      <div class="card__body">
        <span v-if="tag" class="card__tag">{{ tag }}</span>
        <h3 class="card__title">{{ title }}</h3>
        <p class="card__desc">{{ description }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  title: string;
  description: string;
  image: string;
  tag?: string;
}>();
</script>
```

### 渐进增强方案

```css
/* 方案1：先写容器查询，再用媒体查询兜底 */
.card-container {
  container-type: inline-size;
}

/* 容器查询（现代浏览器） */
@container (min-width: 400px) {
  .card {
    flex-direction: row;
  }
}

/* 媒体查询兜底（旧浏览器） */
@supports not (container-type: inline-size) {
  @media (min-width: 768px) {
    .card {
      flex-direction: row;
    }
  }
}

/* 方案2：使用 CSS @layer 管理优先级 */
@layer base {
  .card {
    flex-direction: column;
  }
}

@layer container-queries {
  @container (min-width: 400px) {
    .card {
      flex-direction: row;
    }
  }
}

@layer fallback {
  @supports not (container-type: inline-size) {
    @media (min-width: 768px) {
      .card {
        flex-direction: row;
      }
    }
  }
}
```

**JS ResizeObserver 兜底：**

```ts
// 旧浏览器兼容：用 ResizeObserver 模拟容器查询
function polyfillContainerQueries() {
  if (CSS.supports('container-type', 'inline-size')) return;

  const containers = document.querySelectorAll('[data-container]');

  const observer = new ResizeObserver((entries) => {
    entries.forEach((entry) => {
      const width = entry.contentBoxSize?.[0]?.inlineSize ?? entry.contentRect.width;
      const breakpoints = JSON.parse(entry.target.getAttribute('data-container') || '{}');

      Object.entries(breakpoints).forEach(([className, minWidth]) => {
        if (width >= (minWidth as number)) {
          entry.target.classList.add(className);
        } else {
          entry.target.classList.remove(className);
        }
      });
    });
  });

  containers.forEach((el) => observer.observe(el));
}

// 使用
// <div data-container='{"container-md": 400, "container-lg": 700}' class="card-container">
//   <div class="card container-md:flex-row container-lg:p-8">...</div>
// </div>
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| @container 不生效 | 未声明 `container-type` | 父元素必须设置 `container-type: inline-size` |
| 查询了错误的容器 | 多层嵌套时向上查找到非目标容器 | 使用 `container-name` 命名容器，`@container name (…)` |
| 容器查询内无法修改容器自身尺寸 | 容器不能查询自身 | 在容器外再包一层，查询外层容器 |
| `cqw` 单位不生效 | 父元素未声明 `container-type` | 容器查询单位依赖容器上下文 |
| 性能卡顿 | 嵌套多层容器查询 + 复杂布局 | 减少容器层级，避免 `container-type: size` |
| 容器查询内使用 `height` 不生效 | `container-type: inline-size` 只允许查询宽度 | 改用 `container-type: size` |
| 样式查询不生效 | Style Queries 仍为实验性 | 仅 Chrome 111+ 支持，生产环境用 JS 方案 |
| `container` 简写顺序错误 | `container: name / type` 顺序 | 名称在前，斜杠分隔，类型在后 |

### 最佳实践

- 优先使用 `container-type: inline-size`，避免 `size` 的性能开销
- 给容器命名（`container-name`），避免多层嵌套时查询到错误容器
- 容器查询断点从组件设计出发（如卡片横向布局需要多少宽度），而非沿用媒体查询断点
- 容器查询单位（`cqw`/`cqi`）配合 `clamp()` 实现流式排版
- 容器查询 + 媒体查询配合使用：容器查询管组件内部，媒体查询管页面结构
- 组件库中统一声明容器断点变量：`--cq-sm: 350px; --cq-md: 500px; --cq-lg: 700px;`
- 渐进增强：现代浏览器用容器查询，旧浏览器用媒体查询兜底
- 避免在容器查询中改变容器自身尺寸，防止循环计算

## 面试题

**Q1: 容器查询和媒体查询的核心区别是什么？什么场景下必须用容器查询？**
> 核心区别：媒体查询基于视口宽度，容器查询基于父容器尺寸。必须用容器查询的场景：① 组件库中同一组件在不同宽度容器中需要不同布局（侧边栏 vs 主内容区）；② 组件不知道自己会被放在多宽的容器中；③ 同一页面中多个同类组件位于不同宽度的区域。媒体查询无法处理这些场景，因为它只关心视口宽度，而视口宽度相同时不同区域的容器宽度可能完全不同。

**Q2: container-type 的 inline-size 和 size 有什么区别？应该怎么选？**
> `inline-size` 只允许查询容器的行内方向尺寸（水平流中即宽度），浏览器只监听宽度变化；`size` 允许查询宽度和高度，浏览器同时监听两个方向。选择：90% 场景用 `inline-size` 即可（大多数响应式只需关心宽度），因为 `size` 会监听两个方向的变化，导致更多重排计算，性能开销更大。只有在需要根据容器高度变化布局时（如宽高比不同的网格）才用 `size`。

**Q3: 容器查询单位（cqw/cqh/cqi/cqb）和视口单位（vw/vh）有什么区别？**
> `vw`/`vh` 相对于视口尺寸（`window.innerWidth`/`innerHeight`），`cqw`/`cqh` 相对于查询容器尺寸。`cqi`/`cqb` 是方向无关版本：`cqi` = 容器内联方向的 1%（水平流 = 宽度），`cqb` = 容器块方向的 1%（水平流 = 高度）。核心优势：组件内使用 `cqi` 替代 `vw`，字体大小、间距等能随容器宽度流式缩放，而非随视口。典型用法：`font-size: clamp(0.875rem, 2cqi, 1.25rem)`。

**Q4: 容器查询在组件库中如何应用？有什么注意事项？**
> 组件库中每个组件的根容器声明 `container-type: inline-size`，组件内部用 `@container` 处理响应式布局。注意事项：① 组件必须自包含，不能依赖外部容器宽度；② 使用 `container-name` 避免嵌套时查询到错误容器；③ 容器断点基于组件设计（卡片横向需要多宽），不沿用媒体查询断点；④ 不能在容器查询中修改容器自身尺寸（防止循环计算）；⑤ 导出组件时同时导出容器样式，确保消费者不需要额外设置。

**Q5: 样式查询（Style Queries）是什么？能解决什么问题？**
> 样式查询允许根据容器的自定义属性值（CSS 变量）应用样式：`@container style(--theme: dark) { ... }`。解决的问题：① 主题切换：容器设置 `--theme: dark`，后代组件自动适配暗色；② 布局模式切换：`--layout: grid/list`，同一组数据展示不同布局；③ 变体控制：`--variant: compact/comfortable`，一套样式代码支持多种变体。优势：只需改 CSS 变量就能切换样式，无需修改 class 或 DOM 结构。但目前仍为实验性特性（Chrome 111+ 部分支持），生产环境需 JS 兜底。

**Q6: 容器查询的性能影响是什么？如何优化？**
> 性能开销：浏览器需持续监听容器尺寸变化，每次变化触发后代样式重新计算和可能的布局重排。优化：① 优先使用 `container-type: inline-size`（只监听宽度）而非 `size`（监听宽高）；② 减少容器嵌套层级，避免多层容器同时触发更新；③ 容器查询内只修改视觉属性（color、font-size）而非布局属性（width、height）；④ 避免在容器查询中修改容器自身尺寸（导致循环计算）；⑤ 大量元素时考虑虚拟滚动减少容器数量。

**Q7: 如何在旧浏览器中实现容器查询的渐进增强？**
> 两种方案：① `@supports` 检测：`@supports not (container-type: inline-size) { @media (...) { ... } }`，旧浏览器走媒体查询，新浏览器走容器查询；② JS ResizeObserver 兜底：监听容器尺寸变化，动态添加/移除 class（如 `.container-md`），CSS 中用 `.container-md .card { flex-direction: row }` 替代 `@container`。推荐：组件库中同时提供容器查询和 class 变体两套方案，消费者按浏览器支持情况选择。

**Q8: 容器查询能否替代媒体查询？**
> 不能完全替代。两者解决不同层面的问题：容器查询解决组件级响应式（同一页面中不同位置的组件自适应）；媒体查询解决页面级响应式（整体布局结构切换，如侧边栏显隐、网格列数变化）。最佳实践是配合使用：媒体查询管理页面结构（单列/双列/三列），容器查询管理组件内部布局（竖排/横排/紧凑/宽松）。这样页面结构变化时组件自动适配，组件也能独立于页面使用。

---

**相关链接：**
- [[CSS新特性与现代布局]]
- [[CSS布局Flex与Grid]]
- [[CSS变量与主题系统]]
- [[响应式设计]]
- [[移动端适配方案]]
- [[CSS架构方法论]]

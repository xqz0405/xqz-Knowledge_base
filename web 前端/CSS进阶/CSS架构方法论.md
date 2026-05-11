---
tags:
  - Web前端
  - CSS
  - BEM
  - ITCSS
  - SMACSS
  - 架构
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS 架构方法论

## What — 什么是 CSS 架构方法论

CSS 架构方法论是一套组织 CSS 代码的约定和分层规则，解决全局作用域、命名冲突、样式覆盖混乱等问题。在没有 CSS Modules / CSS-in-JS 等工具化方案之前，这些方法论是大型项目维持 CSS 可维护性的主要手段。

### 核心方法论一览

| 方法论 | 全称 | 核心思想 | 诞生年份 |
|--------|------|----------|----------|
| BEM | Block Element Modifier | 命名约定，通过类名表达结构关系 | 2010 |
| OOCSS | Object-Oriented CSS | 分离结构与外观，容器与内容 | 2009 |
| SMACSS | Scalable and Modular Architecture for CSS | 分类分层，按职责组织 | 2011 |
| ITCSS | Inverted Triangle CSS | 倒三角分层，从通用到具体 | 2014 |
| CUBE CSS | Composition Utility Block Exception | 组合优先，工具类辅助 | 2020 |

---

## Why — 为什么需要 CSS 架构方法论

### 1. CSS 的三个根本问题

| 问题 | 表现 | 根因 |
|------|------|------|
| 全局作用域 | 写 `.title` 影响所有标题 | CSS 天生全局 |
| 层叠冲突 | 后加载的样式覆盖前面的 | 源码顺序决定优先级 |
| 样式腐化 | 不敢删旧样式，怕影响未知页面 | 无从追踪选择器使用范围 |

### 2. 方法论 vs 工具化方案

方法论是约定，靠人遵守；工具化方案（CSS Modules、CSS-in-JS）是机制，靠工具保障。两者不冲突，现代项目通常结合使用。

| 维度 | 方法论 | 工具化方案 |
|------|--------|------------|
| 保障方式 | 人工 Code Review | 自动化构建 |
| 学习成本 | 低 | 中高 |
| 灵活性 | 高 | 受工具限制 |
| 可靠性 | 依赖团队执行 | 机制保证 |
| 适用范围 | 所有 CSS 环境 | 需构建工具支持 |

---

## How — 各方法论详解

### 1. BEM — Block Element Modifier

BEM 是最广泛使用的 CSS 命名约定。通过类名约定表达组件结构，避免嵌套选择器和命名冲突。

**命名规则**：

```
.block           — 组件块
.block__element   — 块内的元素
.block--modifier  — 块的变体
.block__element--modifier — 元素的变体
```

```html
<!-- 一个搜索框组件 -->
<form class="search-form">
  <input class="search-form__input" type="text" />
  <button class="search-form__button search-form__button--primary">
    Search
  </button>
  <button class="search-form__button search-form__button--secondary">
    Cancel
  </button>
</form>
```

```css
/* Block */
.search-form {
  display: flex;
  gap: 8px;
}

/* Element */
.search-form__input {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* Element */
.search-form__button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Modifier */
.search-form__button--primary {
  background: #3b82f6;
  color: white;
}

.search-form__button--secondary {
  background: #e5e7eb;
  color: #374151;
}
```

**BEM 的扁平化原则**：

```css
/* ❌ 错误：嵌套选择器 */
.search-form .input { ... }
.search-form .button.primary { ... }

/* ✅ 正确：扁平类名 */
.search-form__input { ... }
.search-form__button--primary { ... }
```

**BEM 变体写法**：

```css
/* 方式一：完整修饰符（推荐） */
.btn--primary { ... }
.btn--large { ... }

/* 方式二：键值对修饰符（适合多维度） */
.btn--size-sm { ... }
.btn--size-lg { ... }
.btn--color-primary { ... }
.btn--color-danger { ... }

/* 方式三：Molecule 变体（Vue/React 中更实用） */
<button :class="['btn', `btn--${variant}`, `btn--${size}`]">
```

**BEM 与现代框架结合**：

```vue
<!-- Vue + BEM -->
<template>
  <div :class="[
    'card',
    { 'card--featured': featured },
    `card--${theme}`,
  ]">
    <div class="card__header">
      <h3 class="card__title">{{ title }}</h3>
    </div>
    <div class="card__body">
      <slot />
    </div>
  </div>
</template>
```

---

### 2. OOCSS — 面向对象 CSS

OOCSS 的两大原则：

**原则一：分离结构与外观**

```css
/* ❌ 结构与外观耦合 */
.btn-primary {
  padding: 8px 16px;
  border-radius: 4px;
  background: #3b82f6;
  color: white;
}

.btn-secondary {
  padding: 8px 16px;
  border-radius: 4px;
  background: #6b7280;
  color: white;
}

/* ✅ 结构（尺寸/间距）与外观（颜色）分离 */
.btn {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
}

.btn-primary {
  background: #3b82f6;
  color: white;
}

.btn-secondary {
  background: #6b7280;
  color: white;
}
```

```html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
```

**原则二：分离容器与内容**

```css
/* ❌ 依赖容器 */
.sidebar .title { font-size: 14px; }
.main .title { font-size: 24px; }

/* ✅ 独立于容器 */
.title-sm { font-size: 14px; }
.title-lg { font-size: 24px; }
```

OOCSS 是 Tailwind 等原子化 CSS 的思想源头。

---

### 3. SMACSS — 可扩展模块化架构

SMACSS 将 CSS 分为 5 个类别：

| 类别 | 职责 | 命名约定 | 示例 |
|------|------|----------|------|
| Base | 元素默认样式 | 元素选择器 | `body`, `a`, `h1` |
| Layout | 页面布局结构 | `l-` 前缀 | `.l-header`, `.l-sidebar` |
| Module | 可复用组件 | 无前缀 / 语义名 | `.card`, `.nav` |
| State | 状态样式 | `is-` / `has-` 前缀 | `.is-active`, `.has-error` |
| Theme | 主题覆盖 | `theme-` 前缀 | `.theme-dark` |

```css
/* ===== Base ===== */
body {
  font-family: system-ui;
  line-height: 1.6;
  color: #1f2937;
}

a {
  color: #3b82f6;
  text-decoration: none;
}

/* ===== Layout ===== */
.l-page {
  display: grid;
  grid-template-columns: 240px 1fr;
  grid-template-rows: 56px 1fr;
  min-height: 100vh;
}

.l-header {
  grid-column: 1 / -1;
  display: flex;
  align-items: center;
  padding: 0 24px;
  border-bottom: 1px solid #e5e7eb;
}

.l-sidebar {
  padding: 16px;
  border-right: 1px solid #e5e7eb;
}

.l-main {
  padding: 24px;
  overflow: auto;
}

/* ===== Module ===== */
.card {
  border-radius: 8px;
  background: white;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 600;
}

.card-body {
  padding: 16px;
}

/* ===== State ===== */
.is-active {
  font-weight: 600;
  color: #3b82f6;
}

.is-hidden {
  display: none;
}

.is-loading {
  opacity: 0.6;
  pointer-events: none;
}

.has-error {
  border-color: #ef4444;
}

/* ===== Theme ===== */
.theme-dark .l-header {
  background: #1f2937;
  border-color: #374151;
}

.theme-dark .card {
  background: #374151;
  color: #e5e7eb;
}
```

---

### 4. ITCSS — 倒三角 CSS

ITCSS 按照从通用到具体的顺序组织 CSS，形成一个倒三角结构：

```
┌─────────────────────────────────┐  Settings    — 变量、设计令牌
│          Settings               │
├─────────────────────────────────┤  Tools       — Mixins、函数
│          Tools                  │
├─────────────────────────────────┤  Generic     — Reset、Normalize
│         Generic                 │
├─────────────────────────────────┤  Elements    — HTML 元素默认样式
│        Elements                 │
├─────────────────────────────────┤  Objects     — 无装饰的布局模式
│       Objects                   │
├─────────────────────────────────┤  Components  — 具体 UI 组件
│      Components                 │
├─────────────────────────────────┤  Utilities   — 工具类（最高优先级）
│       Utilities                 │
└─────────────────────────────────┘
```

**每层的特点**：

| 层级 | 特异性 | 选择器数量 | 覆盖能力 |
|------|--------|------------|----------|
| Settings | — | — | — |
| Tools | — | — | — |
| Generic | 最低 | 最多 | 最弱 |
| Elements | 低 | 较多 | 较弱 |
| Objects | 中 | 中等 | 中等 |
| Components | 较高 | 较少 | 较强 |
| Utilities | 最高 | 最少 | 最强 |

**目录结构**：

```
styles/
├── settings/
│   ├── _colors.scss
│   ├── _spacing.scss
│   └── _typography.scss
├── tools/
│   ├── _mixins.scss
│   └── _functions.scss
├── generic/
│   ├── _reset.scss
│   └── _normalize.scss
├── elements/
│   ├── _headings.scss
│   ├── _links.scss
│   └── _forms.scss
├── objects/
│   ├── _grid.scss
│   ├── _media.scss
│   └── _layout.scss
├── components/
│   ├── _card.scss
│   ├── _button.scss
│   └── _navbar.scss
├── utilities/
│   ├── _spacing.scss
│   ├── _display.scss
│   └── _text.scss
└── main.scss          — 按层序导入
```

```scss
// main.scss — 严格按层级顺序导入
@import 'settings/colors';
@import 'settings/spacing';
@import 'settings/typography';

@import 'tools/mixins';
@import 'tools/functions';

@import 'generic/reset';
@import 'generic/normalize';

@import 'elements/headings';
@import 'elements/links';
@import 'elements/forms';

@import 'objects/grid';
@import 'objects/media';
@import 'objects/layout';

@import 'components/card';
@import 'components/button';
@import 'components/navbar';

@import 'utilities/spacing';
@import 'utilities/display';
@import 'utilities/text';
```

**Objects 层详解——无装饰布局模式**：

```scss
// _media.scss — 经典 Media Object 模式
.o-media {
  display: flex;
  align-items: flex-start;
}

.o-media__img {
  flex-shrink: 0;
  margin-right: 16px;
}

.o-media__body {
  flex: 1;
}

// _grid.scss — 通用网格
.o-grid {
  display: grid;
  gap: var(--grid-gap, 16px);
  grid-template-columns: repeat(var(--grid-cols, 12), 1fr);
}

.o-grid__item {
  grid-column: span var(--grid-span, 12);
}
```

---

### 5. CUBE CSS — 组合优先的新方法论

CUBE CSS 是现代 CSS 方法论，拥抱 CSS 原生能力（自定义属性、Grid、Flexbox），不依赖预处理器的嵌套。

| 层级 | 含义 | 说明 |
|------|------|------|
| **C**omposition | 组合布局 | 用 Flex/Grid 做布局，不添加装饰 |
| **U**tility | 工具类 | 单一职责的小样式（间距、颜色） |
| **B**lock | 块 | 等同 BEM 的 Block，可复用组件 |
| **E**xception | 异常 | 上下文覆盖，偶尔需要的一刀切样式 |

```html
<!-- Composition：布局 -->
<div class="cluster">
  <!-- Utility：间距、颜色 -->
  <span class="bg-blue-500 text-white px-4 py-2 rounded">Tag 1</span>
  <span class="bg-blue-500 text-white px-4 py-2 rounded">Tag 2</span>
</div>

<!-- Block：组件 -->
<article class="card">
  <img class="card__image" src="..." alt="" />
  <div class="card__content stack">
    <h3 class="card__title">Title</h3>
    <p class="card__desc">Description</p>
  </div>
</article>

<!-- Exception：上下文覆盖 -->
<div class="card card--sidebar">
  <!-- 侧边栏中的卡片样式不同 -->
</div>
```

**CUBE CSS 的布局原语**：

```css
/* Stack — 垂直间距 */
.stack {
  display: flex;
  flex-direction: column;
  gap: var(--stack-space, 1rem);
}

/* Cluster — 水平间距 */
.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--cluster-space, 0.5rem);
  align-items: center;
}

/* Sidebar — 侧边栏布局 */
.with-sidebar {
  display: flex;
  flex-wrap: wrap;
  gap: var(--sidebar-gap, 1rem);
}

.with-sidebar > :first-child {
  flex-basis: var(--sidebar-width, 240px);
  flex-grow: 1;
}

.with-sidebar > :last-child {
  flex-basis: 0;
  flex-grow: 999;
  min-width: calc(50% - var(--sidebar-gap));
}

/* Switcher — 自动换行 */
.switcher {
  display: flex;
  flex-wrap: wrap;
  gap: var(--switcher-gap, 1rem);
}

.switcher > * {
  flex-basis: calc(var(--switcher-threshold, 30rem) - var(--switcher-gap));
  flex-grow: 1;
}

/* Center — 居中容器 */
.center {
  max-width: var(--center-max, 65ch);
  margin-inline: auto;
  padding-inline: var(--center-padding, 1rem);
}
```

---

### 6. @layer 与 ITCSS 的融合

CSS 原生 `@layer` 可以完美替代 ITCSS 的导入顺序约定，用机制保证层叠顺序：

```css
@layer reset, base, layout, components, utilities;

@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
  }
}

@layer base {
  body {
    font-family: system-ui;
    line-height: 1.6;
  }
}

@layer layout {
  .page-grid {
    display: grid;
    grid-template-columns: 240px 1fr;
    min-height: 100vh;
  }
}

@layer components {
  .card {
    border-radius: 8px;
    background: white;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  }
}

@layer utilities {
  .mt-4 { margin-top: 1rem; }
  .text-center { text-align: center; }
  .hidden { display: none; }
}
```

**@layer vs 传统 ITCSS 导入顺序**：

| 维度 | 传统导入顺序 | @layer |
|------|-------------|--------|
| 保障方式 | 约定（靠人） | 机制（靠浏览器） |
| 覆盖风险 | 第三方库可能意外覆盖 | 层顺序保证优先级 |
| 灵活性 | 可随时调整导入顺序 | 层声明必须在最前 |
| 浏览器兼容 | 全部 | Chrome 99+、Safari 15.4+ |

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| BEM 类名过长 | 多层嵌套导致 `block__elem--mod` | 不要超过 Element 一层，用组件拆分代替深层嵌套 |
| ITCSS 层边界模糊 | Object 和 Component 难区分 | Object 无装饰只做布局，Component 有视觉样式 |
| SMACSS State 类被覆盖 | State 类特异性不够 | `.is-active` 特异性低于 `.nav .item`，需加 `!important` |
| OOCSS 组合爆炸 | 大量外观类排列组合 | 用 CSS 变量代替组合类 |
| 方法论间混合冲突 | BEM 命名 + SMACSS 分类混用 | 选一个方法论为主，其他作为补充 |

### 最佳实践

1. **BEM 是基础**：即使使用 CSS Modules / Tailwind，BEM 的命名思维仍有价值。
2. **@layer 替代导入顺序**：现代项目用 `@layer` 机制化保证层叠顺序。
3. **CUBE CSS 适合新项目**：拥抱 CSS 原生能力，不依赖预处理器。
4. **方法论与工具结合**：BEM 命名 + CSS Modules 隔离 + @layer 分层 = 最强组合。
5. **团队统一比选择更重要**：任何方法论，全员一致执行才有效。

---

## 面试题

### 1. BEM 的命名规则是什么？为什么不推荐嵌套选择器？

**答**：BEM 命名规则：`.block` 表示组件块，`.block__element` 表示块内元素，`.block--modifier` 表示块/元素的变体。双下划线连接块与元素，双中划线连接修饰符。不推荐嵌套选择器的原因：(1) 嵌套选择器增加特异性，导致样式难以覆盖和维护；(2) 嵌套选择器与 DOM 结构耦合，DOM 变化会破坏样式；(3) BEM 的扁平类名保证了特异性一致（都是单类名），任何样式都可以通过修改 Modifier 轻松覆盖。

---

### 2. ITCSS 的倒三角分层逻辑是什么？为什么 Utilities 放在最底层？

**答**：ITCSS 的分层逻辑是"从通用到具体、从低特异性到高特异性"：Settings 和 Tools 不产出 CSS；Generic 做全局重置，特异性最低；Elements 设置标签默认样式；Objects 提供无装饰布局模式；Components 是具体 UI 组件；Utilities 是工具类。Utilities 放在最后（最底层/最具体）是因为它们需要能覆盖前面所有层的样式——当你说 `mt-4` 时，无论组件内部怎么设置 margin，都应该生效。这保证了工具类的绝对优先权。

---

### 3. OOCSS 的"分离容器与内容"原则是什么意思？与现代原子化 CSS 有什么关系？

**答**："分离容器与内容"是指样式不应依赖元素所在的容器。传统写法 `.sidebar .title { font-size: 14px; }` 中标题样式依赖侧边栏容器；OOCSS 建议用 `.title-sm { font-size: 14px; }` 让样式独立于位置。这正好是原子化 CSS 的核心理念：每个工具类做一件事，不依赖 DOM 结构，可以在任何地方复用。可以说 Tailwind / UnoCSS 是 OOCSS 思想在工具层面的终极实现——用工具类替代所有依赖容器的样式组合。

---

### 4. CSS 原生 @layer 如何替代 ITCSS 的导入顺序约定？

**答**：ITCSS 传统上依赖 SCSS 导入顺序来保证层叠优先级——先导入的文件特异性低，后导入的高。但这只是约定，第三方库可能不遵守。`@layer` 在 CSS 规范层面保证层顺序：先声明的层优先级最低，后声明的最高，且未分层的样式优先级高于所有分层样式。使用 `@layer reset, base, layout, components, utilities;` 一行声明就锁定了优先级顺序，无论 CSS 文件的实际加载顺序如何。

---

### 5. CUBE CSS 和 BEM 的核心区别是什么？

**答**：CUBE CSS 和 BEM 的核心区别在于对 CSS 原生能力的态度。BEM 诞生于 CSS 能力有限的时代（无 Grid、无自定义属性），通过严格的命名约定来弥补 CSS 的不足。CUBE CSS 拥抱现代 CSS：(1) 用 Flex/Grid 的 Composition 层做布局，而非嵌套 BEM 结构；(2) 用 Utility 层（类似 Tailwind）处理间距、颜色等简单样式，不需要每个都定义 BEM 类名；(3) Block 只处理组件特有的复杂样式；(4) Exception 处理上下文覆盖，而非用 BEM Modifier 暴力枚举所有变体。CUBE CSS 本质上是 BEM + OOCSS + 原子化的融合。

---

### 6. SMACSS 的 State 类为什么要用 `!important`？

**答**：SMACSS 的 State 类（如 `.is-active`、`.is-hidden`）是单个类名，特异性为 `0,1,0`。但组件样式通常也是单类名（`.nav-item`），如果组件样式中设置了 `display: block`，State 类的 `.is-hidden { display: none }` 特异性相同，后加载者胜出——如果加载顺序不对，State 类可能不生效。用 `!important` 可以确保状态类始终覆盖组件样式，因为状态类应该表示"无论组件当前样式是什么，这个状态必须生效"。替代方案是用 `@layer` 将状态层放在组件层之后。

---

### 7. 在现代项目中，BEM + CSS Modules + @layer 的组合怎么用？

**答**：三者各司其职：(1) **BEM 提供命名思维**——CSS Modules 自动哈希类名，但组件内部的类名仍用 BEM 思维组织（如 `card`、`card__title`、`card--featured`），让代码可读；(2) **CSS Modules 提供隔离**——编译时哈希类名保证组件间样式零泄漏，比 BEM 命名约定更可靠；(3) **@layer 管理全局优先级**——将全局 reset、主题、组件、工具类分层，保证工具类可覆盖组件样式。具体做法：全局样式用 `@layer` 分层；组件样式用 CSS Modules + BEM 命名；跨组件共享的布局模式放 `@layer layout`。

---

### 8. 如何处理第三方 UI 库与自有 CSS 的层叠冲突？

**答**：四种策略：(1) **@layer 隔离**——将第三方库样式放入低优先级层：`@layer third-party, components, utilities;`，自有样式自然覆盖；(2) **CSS Modules 隔离**——自有样式用 CSS Modules，哈希类名与第三方类名不可能冲突；(3) **提高特异性**——`.my-app .ant-btn` 比 `.ant-btn` 特异性高，确保覆盖；(4) **CSS 变量覆盖**——现代 UI 库（如 Ant Design 5、Element Plus）使用 CSS 变量，直接覆盖变量值即可：`--el-color-primary: #3b82f6;`。推荐方式(1)或(4)，最干净。

---

## 相关链接

- [BEM 官方文档](https://getbem.com/)
- [OOCSS — Nicole Sullivan](https://github.com/stubbornella/oocss)
- [SMACSS 官方文档](https://smacss.com/)
- [ITCSS — Harry Roberts](https://itcss.io/)
- [CUBE CSS — Andy Bell](https://cube.fyi/)
- [CSS @layer — MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@layer)

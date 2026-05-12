---
tags:
  - Web前端
  - CSS
  - Flexbox
  - Grid
  - 布局
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS布局Flex与Grid

## What — 是什么

> Flexbox 是一维布局模型（行或列），Grid 是二维布局模型（行和列同时控制），两者是现代 CSS 布局的核心方案。

### Flexbox 完整属性表

**容器属性（设在父元素上）：**

| 属性 | 说明 | 可选值 |
|------|------|--------|
| `display` | 定义弹性容器 | `flex` / `inline-flex` |
| `flex-direction` | 主轴方向 | `row`（默认）/ `row-reverse` / `column` / `column-reverse` |
| `flex-wrap` | 是否换行 | `nowrap`（默认）/ `wrap` / `wrap-reverse` |
| `flex-flow` | direction + wrap 简写 | 如 `row wrap` |
| `justify-content` | 主轴对齐 | `flex-start` / `flex-end` / `center` / `space-between` / `space-around` / `space-evenly` |
| `align-items` | 交叉轴对齐（单行） | `stretch`（默认）/ `flex-start` / `flex-end` / `center` / `baseline` |
| `align-content` | 交叉轴对齐（多行） | `stretch`（默认）/ `flex-start` / `flex-end` / `center` / `space-between` / `space-around` / `space-evenly` |
| `gap` | 项目间距 | `row-gap column-gap`，如 `16px` / `8px 16px` |

**项目属性（设在子元素上）：**

| 属性 | 说明 | 可选值 |
|------|------|--------|
| `flex-grow` | 放大比例 | 数字，默认 `0`（不放大） |
| `flex-shrink` | 缩小比例 | 数字，默认 `1`（可缩小） |
| `flex-basis` | 初始主轴尺寸 | `auto`（默认）/ 长度值 / `0` / `content` |
| `flex` | grow shrink basis 简写 | `0 1 auto`（默认）/ `1` / `auto` / `none` |
| `order` | 排列顺序 | 整数，默认 `0`，值小在前 |
| `align-self` | 单个项目交叉轴对齐 | 同 `align-items` 值 + `auto`（默认） |

### Grid 完整属性表

**容器属性（设在父元素上）：**

| 属性 | 说明 | 可选值 / 示例 |
|------|------|---------------|
| `display` | 定义网格容器 | `grid` / `inline-grid` |
| `grid-template-columns` | 定义列轨道 | `200px 1fr 1fr` / `repeat(3, 1fr)` / `repeat(auto-fill, minmax(200px, 1fr))` |
| `grid-template-rows` | 定义行轨道 | 同上语法 |
| `grid-template-areas` | 命名区域 | `"header header" "sidebar content" "footer footer"` |
| `grid-template` | rows + columns + areas 简写 | — |
| `grid-auto-columns` | 隐式列轨道尺寸 | `1fr` / `minmax(100px, auto)` |
| `grid-auto-rows` | 隐式行轨道尺寸 | 同上 |
| `grid-auto-flow` | 自动放置方向 | `row`（默认）/ `column` / `dense` |
| `gap` / `row-gap` / `column-gap` | 间距 | 长度值 |
| `justify-items` | 单元格内水平对齐 | `stretch`（默认）/ `start` / `end` / `center` |
| `align-items` | 单元格内垂直对齐 | 同上 |
| `place-items` | align-items + justify-items | `center` / `start end` |
| `justify-content` | 网格整体水平对齐 | `start` / `end` / `center` / `stretch` / `space-between` / `space-around` / `space-evenly` |
| `align-content` | 网格整体垂直对齐 | 同上 |

**项目属性（设在子元素上）：**

| 属性 | 说明 | 可选值 / 示例 |
|------|------|---------------|
| `grid-column-start` | 起始列线 | 数字 / 命名线 / `span N` |
| `grid-column-end` | 结束列线 | 同上 |
| `grid-column` | start / end 简写 | `1 / 3` / `1 / span 2` / `span 2` |
| `grid-row-start` | 起始行线 | 同 column-start |
| `grid-row-end` | 结束行线 | 同上 |
| `grid-row` | start / end 简写 | 同 grid-column |
| `grid-area` | 区域名 / 行列简写 | `header` / `1 / 1 / 3 / 3` |
| `justify-self` | 单个项目水平对齐 | 同 justify-items |
| `align-self` | 单个项目垂直对齐 | 同 align-items |
| `place-self` | align-self + justify-self | `center` / `start end` |

### Flex 轴模型详解

Flexbox 的核心是**轴模型**，理解轴是掌握 Flex 的关键：

```
┌─────────────────────────────────────────────┐
│  主轴（Main Axis）───→                       │  flex-direction: row
│  ┌──────┐  ┌──────┐  ┌──────┐              │
│  │ Item │  │ Item │  │ Item │  ← 交叉轴     │
│  │  1   │  │  2   │  │  3   │  （Cross Axis）│
│  └──────┘  └──────┘  └──────┘  ↓            │
│                                              │
│  justify-content → 控制主轴对齐              │
│  align-items     → 控制交叉轴对齐            │
└─────────────────────────────────────────────┘
```

- **主轴（Main Axis）**：由 `flex-direction` 决定方向
  - `row`：从左到右（默认）
  - `row-reverse`：从右到左
  - `column`：从上到下
  - `column-reverse`：从下到上
- **交叉轴（Cross Axis）**：与主轴垂直的方向
  - 主轴水平时，交叉轴垂直
  - 主轴垂直时，交叉轴水平
- **主轴尺寸（Main Size）**：项目在主轴方向上的尺寸（width 或 height）
- **交叉轴尺寸（Cross Size）**：项目在交叉轴方向上的尺寸（height 或 width）

> 注意：`justify-content` 控制主轴，`align-items` 控制交叉轴，方向随 `flex-direction` 变化而变化，不要死记"水平/垂直"。

### Grid 网格模型详解

Grid 的核心概念比 Flex 更丰富：

```
列线1  列线2  列线3  列线4
  ↓      ↓      ↓      ↓
  ┌──────┬──────┬──────┐ ← 行线1
  │ Cell │ Cell │ Cell │
  ├──────┼──────┼──────┤ ← 行线2
  │ Cell │ Cell │ Cell │
  ├──────┼──────┼──────┤ ← 行线3
  │ Cell │ Cell │ Cell │
  └──────┴──────┴──────┘ ← 行线4

  ← 轨道1 →← 轨道2 →← 轨道3 →
```

- **网格线（Grid Line）**：构成网格结构的分界线，从 1 开始编号，也可以命名
- **轨道（Track）**：两条相邻网格线之间的空间，即行或列
- **单元格（Cell）**：行和列交叉形成的最小单元
- **区域（Area）**：由一个或多个单元格组成的矩形范围
- **间隙（Gap）**：轨道之间的间距，`row-gap` 和 `column-gap`
- **显式网格**：由 `grid-template-columns/rows` 明确定义的网格
- **隐式网格**：项目超出显式网格范围时自动生成的轨道，尺寸由 `grid-auto-columns/rows` 控制

### Flex vs Grid 适用场景

| 场景 | 推荐方案 | 原因 |
|------|----------|------|
| 导航栏（logo左、菜单右） | Flex | 一维方向分布，space-between 天然适配 |
| 居中元素 | Flex/Grid | 两者都可以，Flex 更常见 |
| 等分卡片列表 | Flex | `flex: 1` 简单直观 |
| 响应式卡片网格 | Grid | `auto-fill + minmax` 无媒体查询换行 |
| 页面整体布局（header/sidebar/main/footer） | Grid | 二维命名区域，语义清晰 |
| 仪表盘 | Grid | 不同大小面板跨行跨列 |
| 表单对齐 | Grid | 行列对齐，标签和输入框齐整 |
| 工具栏 | Flex | 一维排列，间距控制简单 |
| 瀑布流 | Grid + masonry | Grid 的 masonry 实验性特性 |
| 等高列 | Flex/Grid | 两者都天然支持等高 |

### Grid 的高级特性

**auto-fill vs auto-fit：**

```css
/* auto-fill：尽可能多地创建列，空列保留空间 */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));

/* auto-fit：尽可能多地创建列，空列折叠为0 */
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
```

- `auto-fill`：填充尽可能多的列轨道，即使有些是空的。项目少时会有空白轨道。
- `auto-fit`：同上，但空轨道折叠为 0 宽度，让现有项目拉伸填满。项目少时看起来更满。

**minmax()：**

```css
/* 最小 200px，最大 1fr（占满剩余空间） */
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));

/* 最小 auto（内容撑开），最大 300px */
grid-template-columns: repeat(3, minmax(auto, 300px));
```

`minmax()` 定义轨道的最小和最大尺寸范围，是实现无媒体查询响应式的核心。

**命名线（Named Lines）：**

```css
.grid {
    grid-template-columns: [start] 200px [main-start] 1fr [main-end] 200px [end];
    grid-template-rows: [top] auto [content-top] 1fr [content-bottom] auto [bottom];
}

.item {
    grid-column: main-start / main-end;
    grid-row: content-top / content-bottom;
}
```

命名线让网格放置更语义化，避免硬编码数字。

**命名区域（Named Areas）：**

```css
.layout {
    grid-template-areas:
        "header header  header"
        "sidebar content aside"
        "footer footer  footer";
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.aside   { grid-area: aside; }
.footer  { grid-area: footer; }
```

命名区域是最直观的 Grid 布局方式，ASCII 艺术般的声明。

**subgrid：**

```css
.card-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
}

.card {
    display: grid;
    grid-row: span 3;
    /* 继承父网格的行轨道定义 */
    grid-template-rows: subgrid;
}
```

`subgrid` 让子网格继承父网格的轨道定义，解决嵌套网格对齐问题。目前 Chrome 117+、Firefox 71+ 已支持。

## Why — 为什么

### Flex vs Grid vs Float vs Position 对比

| 维度 | Flexbox | Grid | Float | Position |
|------|---------|------|-------|----------|
| 布局维度 | 一维（行或列） | 二维（行和列） | 一维（仅水平） | 脱离文档流 |
| 居中难度 | 极简单 | 极简单 | 困难（需 hack） | 需手动计算 |
| 等高列 | 自动 | 自动 | 需 hack（伪等高） | 不支持 |
| 源码顺序 | 可用 order 改变 | 可自由放置 | 由 HTML 顺序决定 | 不影响 |
| 响应式能力 | 中等 | 极佳（auto-fill/auto-fit） | 差 | 差 |
| 语义化 | 一般 | 强（命名区域） | 无 | 无 |
| 浏览器支持 | 极佳 | 极佳 | 全兼容 | 全兼容 |
| 内容驱动 vs 布局驱动 | 内容驱动（项目大小影响布局） | 布局驱动（先定义网格再放项目） | 内容驱动 | N/A |
| 典型用途 | 导航栏、卡片列表、居中 | 页面布局、仪表盘、表单 | 文字环绕图片 | 弹窗、固定头、悬浮按钮 |
| 学习曲线 | 低 | 中 | 低（但坑多） | 低 |

### 什么时候用 Flex，什么时候用 Grid

**选 Flex 的场景：**
- 只需要在一个方向上排列元素（一行或一列）
- 项目的大小由内容决定
- 需要灵活地分配剩余空间（如导航栏的剩余空间给搜索框）
- 简单的居中需求
- 不确定项目数量，需要自动换行

**选 Grid 的场景：**
- 需要同时控制行和列
- 需要项目跨越多行多列
- 需要精确的二维对齐
- 页面级别的宏观布局
- 需要无媒体查询的响应式网格

**混合使用的最佳实践：**
- 外层用 Grid 做页面框架（header/sidebar/main/footer）
- 内层用 Flex 做组件内部排列（导航项、按钮组、卡片内容）
- Grid 嵌套 Flex 是最常见的组合

> 记忆口诀：**一维 Flex，二维 Grid，外 Grid 内 Flex 是套路。**

## How — 怎么用

### 快速上手

**Flexbox 居中：**

```css
.container {
    display: flex;
    justify-content: center; /* 主轴居中 */
    align-items: center;     /* 交叉轴居中 */
    gap: 16px;               /* 项目间距 */
}
```

**Grid 基础：**

```css
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3等分 */
    gap: 20px;
}
```

### 示例 1：Flex 居中（多种方案对比）

```css
/* 方案一：Flex 居中（最推荐） */
.center-flex {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* 方案二：Flex + margin auto */
.center-flex-margin {
    display: flex;
}
.center-flex-margin .child {
    margin: auto; /* 四个方向自动 */
}

/* 方案三：Grid 居中 */
.center-grid {
    display: grid;
    place-items: center; /* align-items + justify-items 简写 */
}

/* 方案四：绝对定位 + transform（传统方案） */
.center-absolute {
    position: relative;
}
.center-absolute .child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

### 示例 2：Flex 导航栏

```css
/* 导航栏：logo左 菜单中 按钮右 */
.navbar {
    display: flex;
    align-items: center;
    padding: 0 20px;
    height: 60px;
    background: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.navbar .logo {
    flex-shrink: 0; /* logo 不缩小 */
    margin-right: 24px;
}

.navbar .menu {
    display: flex;
    gap: 8px;
    flex: 1; /* 菜单占据剩余空间 */
    justify-content: center;
}

.navbar .actions {
    display: flex;
    gap: 12px;
    flex-shrink: 0; /* 按钮不缩小 */
}

/* 移动端：菜单换行到下一行 */
@media (max-width: 768px) {
    .navbar {
        flex-wrap: wrap;
        height: auto;
        padding: 12px 20px;
    }
    .navbar .menu {
        order: 3; /* 排到最后 */
        flex-basis: 100%; /* 占满整行 */
        justify-content: flex-start;
    }
}
```

### 示例 3：Flex 卡片列表 + 等高列

```css
.card-list {
    display: flex;
    gap: 20px;
    /* 多行换行 */
    flex-wrap: wrap;
}

.card {
    flex: 1 1 300px; /* 最小300px，可放大可缩小 */
    min-width: 0;     /* 防止内容撑破 */
    display: flex;
    flex-direction: column; /* 卡片内容纵向排列 */
    gap: 12px;
    padding: 20px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
}

.card .title {
    font-size: 18px;
    font-weight: 600;
}

.card .description {
    flex: 1; /* 描述区域弹性填充，保证等高 */
    color: #666;
}

.card .footer {
    margin-top: auto; /* 底部按钮始终贴底 */
    display: flex;
    gap: 8px;
}
```

### 示例 4：Flex 圣杯布局

```css
/* 圣杯布局：Header + (Sidebar | Main | Aside) + Footer */
.holy-grail {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.holy-grail .header,
.holy-grail .footer {
    flex-shrink: 0;
    padding: 16px;
    background: #333;
    color: #fff;
}

.holy-grail .body {
    display: flex;
    flex: 1; /* 填充剩余高度 */
}

.holy-grail .sidebar {
    flex: 0 0 200px; /* 固定宽度 */
    background: #f5f5f5;
    padding: 16px;
}

.holy-grail .main {
    flex: 1; /* 占据中间剩余空间 */
    padding: 16px;
}

.holy-grail .aside {
    flex: 0 0 150px; /* 固定宽度 */
    background: #f5f5f5;
    padding: 16px;
}

/* 移动端：侧边栏变为纵向排列 */
@media (max-width: 768px) {
    .holy-grail .body {
        flex-direction: column;
    }
    .holy-grail .sidebar,
    .holy-grail .aside {
        flex: 0 0 auto; /* 自适应高度 */
    }
}
```

### 示例 5：Flex 粘性页脚

```css
/* 内容不足一屏时，页脚贴底；内容超出一屏时，页脚正常跟随 */
.page {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.page .content {
    flex: 1; /* 填充剩余空间，把页脚推到底部 */
}

.page .footer {
    flex-shrink: 0;
    padding: 20px;
    text-align: center;
    background: #f0f0f0;
}
```

### 示例 6：Grid 命名区域布局（圣杯布局）

```css
.layout {
    display: grid;
    grid-template-areas:
        "header header  header"
        "sidebar content aside"
        "footer footer  footer";
    grid-template-columns: 200px 1fr 150px;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
    gap: 0;
}

.header  { grid-area: header;  background: #333; color: #fff; padding: 16px; }
.sidebar { grid-area: sidebar; background: #f5f5f5; padding: 16px; }
.content { grid-area: content; padding: 24px; }
.aside   { grid-area: aside;   background: #f5f5f5; padding: 16px; }
.footer  { grid-area: footer;  background: #333; color: #fff; padding: 16px; }

/* 移动端：重新定义区域 */
@media (max-width: 768px) {
    .layout {
        grid-template-areas:
            "header"
            "content"
            "sidebar"
            "aside"
            "footer";
        grid-template-columns: 1fr;
        grid-template-rows: auto auto auto auto auto;
    }
}
```

### 示例 7：Grid auto-fit + minmax 响应式（无媒体查询）

```css
/* 自适应卡片网格：卡片最小 280px，自动增减列数 */
.auto-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 20px;
    padding: 20px;
}

.card {
    padding: 20px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
}

/*
 * 原理解析：
 * - auto-fit：自动计算能放多少列
 * - minmax(280px, 1fr)：每列最小 280px，最大平分剩余空间
 * - 屏幕宽 1200px：(1200 - 40) / 280 ≈ 4列
 * - 屏幕宽 600px：2列
 * - 屏幕宽 300px：1列
 * - 无需任何媒体查询！
 */
```

### 示例 8：Grid 杂志布局

```css
.magazine {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-auto-rows: 200px;
    gap: 16px;
}

/* 头条：跨2列2行 */
.magazine .featured {
    grid-column: span 2;
    grid-row: span 2;
}

/* 宽文章：跨2列 */
.magazine .wide {
    grid-column: span 2;
}

/* 高文章：跨2行 */
.magazine .tall {
    grid-row: span 2;
}

/* 普通文章：1列1行（默认，无需设置） */
```

### 示例 9：Grid 仪表盘布局

```css
.dashboard {
    display: grid;
    grid-template-columns: 240px repeat(3, 1fr);
    grid-template-rows: 60px 1fr 1fr 40px;
    grid-template-areas:
        "nav    nav     nav     nav"
        "sidebar stats   stats   chart"
        "sidebar table   table   chart"
        "footer footer  footer  footer";
    gap: 12px;
    min-height: 100vh;
    padding: 0;
}

.nav     { grid-area: nav;     background: #1a1a2e; color: #fff; display: flex; align-items: center; padding: 0 20px; }
.sidebar { grid-area: sidebar; background: #16213e; color: #fff; padding: 16px; }
.stats   { grid-area: stats;   background: #fff; padding: 16px; }
.table   { grid-area: table;   background: #fff; padding: 16px; overflow: auto; }
.chart   { grid-area: chart;   background: #fff; padding: 16px; }
.footer  { grid-area: footer;  background: #f0f0f0; display: flex; align-items: center; justify-content: center; }
```

### 示例 10：Grid 命名线布局

```css
.layout {
    display: grid;
    grid-template-columns:
        [full-start] 1fr
        [main-start] minmax(0, 800px)
        [main-end] 1fr
        [full-end];
    grid-template-rows:
        [header-start] auto
        [header-end content-start] 1fr
        [content-end footer-start] auto
        [footer-end];
    min-height: 100vh;
    row-gap: 0;
}

/* 全宽元素（header、footer） */
.header { grid-column: full-start / full-end; }
.footer { grid-column: full-start / full-end; }

/* 居中内容区（最大宽度 800px） */
.content { grid-column: main-start / main-end; }
```

### 示例 11：Grid subgrid 对齐

```css
/* 父网格定义行轨道 */
.card-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    grid-template-rows: auto 1fr auto; /* 标题 / 描述 / 按钮 */
    gap: 16px;
}

/* 子网格继承父网格的行轨道 */
.card {
    display: grid;
    grid-row: span 3;         /* 跨3行（与父网格行轨道对齐） */
    grid-template-rows: subgrid; /* 继承父网格的行定义 */
    gap: 8px;
    padding: 16px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
}

.card .title   { /* 自动对齐到父网格第1行 */ }
.card .desc    { /* 自动对齐到父网格第2行 */ }
.card .actions { /* 自动对齐到父网格第3行，所有卡片按钮对齐 */ }

/*
 * 没有 subgrid 时：
 * 每个卡片的行轨道独立计算，描述文字少的卡片按钮靠上，
 * 描述文字多的卡片按钮靠下，视觉不齐。
 *
 * 有了 subgrid：
 * 所有卡片的行轨道共享父网格定义，标题/描述/按钮三行对齐。
 */
```

### 示例 12：Flex 嵌套 Grid 混合布局

```css
/* 外层 Flex 做页面框架 */
.app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.app .header {
    flex-shrink: 0;
    display: flex;           /* Flex：logo + nav 横排 */
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
    height: 60px;
    background: #fff;
}

.app .main {
    flex: 1;
    display: grid;           /* Grid：侧边栏 + 内容区 */
    grid-template-columns: 220px 1fr;
    gap: 0;
}

.app .sidebar {
    background: #f8f9fa;
    padding: 16px;
}

.app .content {
    padding: 24px;
    display: grid;           /* Grid：内容区卡片网格 */
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 16px;
    align-content: start;    /* 内容少时从顶部开始 */
}

.card {
    display: flex;           /* Flex：卡片内部纵向排列 */
    flex-direction: column;
    gap: 12px;
    padding: 16px;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
}

.card .actions {
    display: flex;           /* Flex：按钮横排 */
    gap: 8px;
    margin-top: auto;        /* 按钮贴底 */
}

.app .footer {
    flex-shrink: 0;
    padding: 16px;
    text-align: center;
    background: #f0f0f0;
}
```

### 示例 13：Flex wrap 和间距管理

```css
/* 方案一：gap 属性（推荐，现代浏览器支持） */
.tag-list {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;   /* 行间距和列间距统一 */
}

/* 方案二：gap 分别控制行间距和列间距 */
.tag-list {
    display: flex;
    flex-wrap: wrap;
    row-gap: 12px;
    column-gap: 8px;
}

/* 方案三：旧方案，margin + 负 margin（兼容旧浏览器） */
.tag-list-legacy {
    display: flex;
    flex-wrap: wrap;
    margin: -4px; /* 抵消最外层 margin */
}
.tag-list-legacy .tag {
    margin: 4px;
}

/* 方案四：justify-content + 固定宽度（不推荐，末行左对齐有问题） */
.tag-list-justify {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Flex 项目溢出容器 | `flex-shrink: 0` 或 `min-width` 未设置 | 设置 `min-width: 0` 或 `overflow: hidden` |
| Grid 间距不均匀 | 混用 `gap` 和 `margin` | 统一用 `gap`，去掉 margin |
| `flex: 1` 不等分 | `flex-basis` 默认 auto，内容长度差异导致 | 加 `min-width: 0` 或 `flex-basis: 0` |
| 图片撑破布局 | 图片原始尺寸未限制 | `img { max-width: 100%; height: auto; }` |
| Grid 项目超出网格范围 | 未定义足够轨道 | 用 `grid-auto-rows/columns` 定义隐式轨道 |
| `align-content` 无效 | 单行 Flex 项目不触发 | 需要 `flex-wrap: wrap` 且多行才生效 |
| Flex 换行后间距不一致 | 用了 margin 而非 gap | 用 `gap` 替代 margin |
| Grid 命名区域拼写错误 | 区域名不匹配 | 确保声明和引用完全一致 |
| 子元素不参与 Flex 布局 | 子元素是绝对定位 | 绝对定位元素脱离 Flex 流，不参与分配 |
| `flex-basis` 与 `width` 冲突 | 同时设置两者 | `flex-basis` 优先级高于 `width`（在主轴方向） |

### 最佳实践

1. **一维用 Flex，二维用 Grid**，不要纠结，两者可嵌套组合
2. **间距统一用 `gap`**，不用 margin hack，更简洁可控
3. **Flex 项目加 `min-width: 0`** 防止内容溢出，这是最常见的坑
4. **Grid 响应式优先用 `auto-fill`/`auto-fit` + `minmax`**，减少媒体查询
5. **命名区域优先于数字线**，代码可读性更高
6. **外 Grid 内 Flex 是黄金组合**，Grid 做宏观框架，Flex 做微观对齐
7. **`flex: 1` 等价于 `flex: 1 1 0%`**，不是 `flex: 1 1 auto`，理解差异
8. **Grid `place-items: center`** 是最简居中写法，比 Flex 少一行

## 面试题

**Q1: Flexbox 和 Grid 的核心区别是什么？**
> Flexbox 是一维布局模型，沿主轴（行或列）分布项目；Grid 是二维布局模型，可同时控制行和列。Flex 是内容驱动的——项目大小影响布局；Grid 是布局驱动的——先定义网格再放置项目。一维分布用 Flex，二维精确布局用 Grid，两者可嵌套组合使用。

**Q2: Flexbox 的 `flex: 1` 是什么意思？**
> `flex: 1` 是 `flex-grow: 1; flex-shrink: 1; flex-basis: 0%` 的简写。表示项目可放大填充剩余空间（grow: 1）、可缩小（shrink: 1）、初始基准为 0%（basis: 0%）。注意 `flex: 1` 与 `flex: auto`（`1 1 auto`）的区别：`flex: 1` 的 basis 为 0%，先平分空间再放大，等分效果更可靠；`flex: auto` 的 basis 为 auto，先按内容分配再放大，内容差异大时不等分。

**Q3: Grid 的 auto-fill 和 auto-fit 有什么区别？**
> 两者都会自动计算能放多少列。区别在于：`auto-fill` 会创建尽可能多的列轨道，即使有些是空的——项目少时会有空白轨道占位；`auto-fit` 同样创建尽可能多的列，但空轨道会折叠为 0 宽度，让现有项目拉伸填满剩余空间。项目数量充足时两者表现一致，项目少时 `auto-fit` 视觉上更满，`auto-fill` 保留空间。常见用法：`repeat(auto-fit, minmax(250px, 1fr))` 实现无媒体查询响应式。

**Q4: 如何用 Flex 实现居中？有哪些方案？**
> 方案一（最推荐）：`display: flex; justify-content: center; align-items: center;`。方案二：`display: flex;` 子元素 `margin: auto;`。方案三（Grid）：`display: grid; place-items: center;`。方案四（传统）：`position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);`。方案五（行内元素）：父 `text-align: center; line-height` 等于高度。现代开发首选 Flex 或 Grid 方案，代码最少、兼容最好。

**Q5: Grid 的 subgrid 解决什么问题？**
> subgrid 解决嵌套网格与父网格轨道对齐的问题。没有 subgrid 时，子网格独立定义自己的轨道，无法与父网格的行或列对齐。典型场景：卡片列表中，每个卡片内部有标题、描述、按钮三行，不同卡片描述文字长度不同，导致按钮位置参差不齐。使用 `grid-template-rows: subgrid` 后，子网格继承父网格的行轨道定义，所有卡片的标题行、描述行、按钮行各自对齐。目前 Chrome 117+、Firefox 71+、Safari 16+ 已支持。

**Q6: CSS 居中有哪些方案？**
> Flex 居中：`display: flex; justify-content: center; align-items: center;`。Grid 居中：`display: grid; place-items: center;`。绝对定位：`position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);`。行内元素：`text-align: center` + `line-height` 等于父高度。现代开发推荐 Flex 或 Grid 方案。Grid 的 `place-items: center` 是最简写法。

**Q7: `flex: 1` 为什么有时不等分？如何解决？**
> 因为 `flex: 1` 等价于 `flex: 1 1 0%`，虽然 basis 为 0%，但当项目内容很长且没有 `min-width: 0` 时，浏览器不会让项目缩小到小于内容的最小宽度，导致视觉上不等分。解决方法：添加 `min-width: 0` 允许项目缩小到小于内容宽度，或添加 `overflow: hidden` 截断溢出内容。同理纵向布局用 `min-height: 0`。

**Q8: Flexbox 的 `flex-grow`、`flex-shrink`、`flex-basis` 分别是什么？**
> `flex-grow` 定义项目的放大比例，默认 0（不放大）。当容器有剩余空间时，按 grow 比例分配。`flex-shrink` 定义项目的缩小比例，默认 1（可缩小）。当空间不足时，按 shrink 比例缩小。`flex-basis` 定义项目在主轴上的初始尺寸，默认 auto（取 width/height 或内容大小），分配前先按 basis 分配空间，剩余空间再按 grow 分配。三者简写：`flex: grow shrink basis`，常见值：`flex: 1`（`1 1 0%`）、`flex: auto`（`1 1 auto`）、`flex: none`（`0 0 auto`，不放大不缩小）。

---

**相关链接：**
- [[HTML5语义化]]
- [[响应式设计]]
- [[CSS动画与过渡]]
- [[CSS选择器与优先级]]
- [[CSS变量与自定义属性]]

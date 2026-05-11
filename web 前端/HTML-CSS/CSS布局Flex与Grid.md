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

**核心概念：**

**Flexbox：**
- **主轴/交叉轴**：`flex-direction` 决定主轴方向
- **容器属性**：`display: flex`、`justify-content`、`align-items`、`flex-wrap`
- **项目属性**：`flex-grow`、`flex-shrink`、`flex-basis`、`order`

**Grid：**
- **网格容器**：`display: grid`、`grid-template-columns/rows`
- **间距**：`gap`（行列通用）
- **项目放置**：`grid-column`、`grid-row`、`grid-area`
- **命名区域**：`grid-template-areas`

**关键特性：**

- Flexbox 擅长项目分布和对齐（导航栏、卡片列表）
- Grid 擅长二维布局（整体页面、复杂网格）
- 两者可以嵌套使用

## Why — 为什么

**适用场景：**

- Flexbox：导航栏、居中对齐、等分空间、卡片横排
- Grid：页面整体布局、仪表盘、瀑布流、复杂表单

**对比替代方案：**

| 维度 | Flexbox | Grid | Float | Position |
|------|---------|------|-------|----------|
| 维度 | 一维 | 二维 | 一维 | 脱离文档流 |
| 居中 | 简单 | 简单 | 困难 | 需计算 |
| 等高列 | 自动 | 自动 | 需 hack | 不支持 |
| 响应式 | 中等 | 极佳（auto-fill） | 差 | 差 |

**优缺点：**

- ✅ Flexbox 优点：
  - 一维分布极其简单
  - 自动处理项目大小和对齐
  - 浏览器支持极好
- ✅ Grid 优点：
  - 二维控制精确
  - `auto-fill`/`auto-fit` 天然响应式
  - 命名区域语义化布局
- ❌ 共同缺点：
  - 旧项目兼容需注意（IE 已淘汰，基本无影响）

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

### 代码示例

**Flexbox 常见布局：**

```css
/* 导航栏：logo左 按钮右 */
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
}

/* 等分卡片 */
.card-list {
    display: flex;
    gap: 16px;
}
.card {
    flex: 1; /* 等分空间 */
    min-width: 0; /* 防止内容撑破 */
}

/* 底部固定 */
.page {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}
.page .content {
    flex: 1; /* 填充剩余空间 */
}
```

**Grid 命名区域布局：**

```css
.layout {
    display: grid;
    grid-template-areas:
        "header header  header"
        "sidebar content aside"
        "footer footer  footer";
    grid-template-columns: 200px 1fr 200px;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.content { grid-area: content; }
.aside   { grid-area: aside; }
.footer  { grid-area: footer; }
```

**Grid 响应式自动换行：**

```css
.auto-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 20px;
}
/* 自动填充，屏幕窄时列数减少，无需媒体查询 */
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Flex 项目溢出 | `flex-shrink: 0` 或 `min-width` 未设置 | 设置 `min-width: 0` 或 `overflow: hidden` |
| Grid 间距不均匀 | 混用 `gap` 和 `margin` | 统一用 `gap`，去掉 margin |
| flex: 1 不等分 | 内容长度差异导致 | 加 `min-width: 0` 或 `flex-basis: 0` |
| 图片撑破布局 | 图片原始尺寸未限制 | `img { max-width: 100%; height: auto; }` |

### 最佳实践

- 一维用 Flex，二维用 Grid，不要纠结
- 间距统一用 `gap`，不用 margin hack
- Flex 项目加 `min-width: 0` 防止溢出
- Grid 响应式优先用 `auto-fill` + `minmax`

## 面试题

**Q1: Flexbox和Grid的核心区别是什么？**
> Flexbox是一维布局模型，沿主轴（行或列）分布项目；Grid是二维布局模型，可同时控制行和列。一维分布用Flex，二维精确布局用Grid，两者可嵌套组合使用。

**Q2: flex: 1 是什么含义？**
> `flex: 1` 是 `flex-grow: 1; flex-shrink: 1; flex-basis: 0%` 的简写，表示项目可放大填充剩余空间、可缩小、初始基准为0。注意flex-basis为0%时等分效果更可靠，但内容较长时需配合`min-width: 0`防止溢出。

**Q3: CSS居中有哪些方案？**
> Flex居中：`display: flex; justify-content: center; align-items: center;`。Grid居中：`display: grid; place-items: center;`。绝对定位：`position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);`。行内元素：`text-align: center` + `line-height`等于父高度。现代开发推荐Flex或Grid方案。

**Q4: flex: 1 为什么有时不等分？如何解决？**
> 因为flex-basis默认为auto，项目会先按内容大小分配空间再按flex-grow分配剩余，内容长度差异大时视觉上不等分。解决方法：设置`flex-basis: 0`（即`flex: 1 1 0%`）或添加`min-width: 0`防止内容撑破。

---

**相关链接：**
- [[HTML5语义化]]
- [[响应式设计]]
- [[CSS动画与过渡]]

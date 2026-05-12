---
tags:
  - Web前端
  - CSS
  - 逻辑属性
  - 国际化
  - RTL
  - 书写模式
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# CSS逻辑属性与国际化布局

## What — 是什么

> CSS 逻辑属性（Logical Properties）是相对于书写方向（writing mode）定义的属性体系，替代传统的物理方向属性（left/right/top/bottom）。在从左到右（LTR）的语言中，逻辑属性与物理属性一致；在从右到左（RTL）的语言（阿拉伯语、希伯来语）或竖排书写模式（日文、蒙古文）中，逻辑属性会自动适配方向，无需为不同语言重写样式。

**核心概念：**

- **物理属性**：基于屏幕方向的属性（`left`/`right`/`top`/`bottom`/`width`/`height`），不随书写方向变化
- **逻辑属性**：基于书写方向的属性（`inline-start`/`inline-end`/`block-start`/`block-end`/`inline-size`/`block-size`），自动适配不同语言和方向
- **书写模式（Writing Mode）**：`writing-mode` 定义文本排列方向（水平/垂直）和行进方向
- **方向（Direction）**：`direction` 定义行内方向（LTR/RTL）

**方向映射关系：**

```
┌────────────────────────────────────────────────────────┐
│              水平 LTR（英文、中文）                      │
│                                                        │
│  block-start (top)                                     │
│  ┌──────────────────────────────────────┐              │
│  │ inline-start    →    inline-end      │              │
│  │ (left)               (right)         │              │
│  │                                      │              │
│  │  文字从左到右排列 →                    │              │
│  │                                      │              │
│  └──────────────────────────────────────┘              │
│  block-end (bottom)                                    │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│              水平 RTL（阿拉伯语、希伯来语）              │
│                                                        │
│  block-start (top)                                     │
│  ┌──────────────────────────────────────┐              │
│  │ inline-end    ←    inline-start      │              │
│  │ (left)              (right)          │              │
│  │                                      │              │
│  │  ← 文字从右到左排列                    │              │
│  │                                      │              │
│  └──────────────────────────────────────┘              │
│  block-end (bottom)                                    │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│              垂直书写（日文竖排、蒙古文）                │
│                                                        │
│  inline-start ← block-start                            │
│  (top)          (right)                                 │
│  ┌────┬────┬────┬────┐                                 │
│  │ 文 │ 从 │ 上 │ 下 │                                 │
│  │ 字 │ 上 │ 到 │ 排 │                                 │
│  │    │ 到 │ 列 │ 列 │                                 │
│  │    │ 下 │    │    │                                 │
│  └────┴────┴────┴────┘                                 │
│  inline-end (bottom)  block-end (left)                  │
└────────────────────────────────────────────────────────┘
```

**逻辑属性 vs 物理属性对照：**

| 类别 | 物理属性 | 逻辑属性 | LTR 水平对应 |
|------|---------|---------|-------------|
| 尺寸 | `width` | `inline-size` | width |
| 尺寸 | `height` | `block-size` | height |
| 外边距 | `margin-left` | `margin-inline-start` | margin-left |
| 外边距 | `margin-right` | `margin-inline-end` | margin-right |
| 外边距 | `margin-top` | `margin-block-start` | margin-top |
| 外边距 | `margin-bottom` | `margin-block-end` | margin-bottom |
| 内边距 | `padding-left` | `padding-inline-start` | padding-left |
| 内边距 | `padding-right` | `padding-inline-end` | padding-right |
| 内边距 | `padding-top` | `padding-block-start` | padding-top |
| 内边距 | `padding-bottom` | `padding-block-end` | padding-bottom |
| 定位 | `left` | `inset-inline-start` | left |
| 定位 | `right` | `inset-inline-end` | right |
| 定位 | `top` | `inset-block-start` | top |
| 定位 | `bottom` | `inset-block-end` | bottom |
| 边框 | `border-left` | `border-inline-start` | border-left |
| 边框 | `border-right` | `border-inline-end` | border-right |
| 圆角 | `border-top-left-radius` | `border-start-start-radius` | border-top-left-radius |
| 圆角 | `border-top-right-radius` | `border-start-end-radius` | border-top-right-radius |
| 圆角 | `border-bottom-left-radius` | `border-end-start-radius` | border-bottom-left-radius |
| 圆角 | `border-bottom-right-radius` | `border-end-end-radius` | border-bottom-right-radius |
| 浮动 | `float: left` | `float: inline-start` | float: left |
| 清除 | `clear: left` | `clear: inline-start` | clear: left |
| 文本对齐 | `text-align: left` | `text-align: start` | text-align: left |
| 调整大小 | `resize: horizontal` | `resize: inline` | resize: horizontal |

**简写属性：**

| 物理简写 | 逻辑简写 | 说明 |
|---------|---------|------|
| `margin: 10px 20px` | `margin: 10px 20px` | 物理简写仍用物理值 |
| `margin-inline: 20px` | — | `margin-inline-start` + `margin-inline-end` |
| `margin-block: 10px` | — | `margin-block-start` + `margin-block-end` |
| `padding-inline: 16px` | — | `padding-inline-start` + `padding-inline-end` |
| `padding-block: 8px` | — | `padding-block-start` + `padding-block-end` |
| `inset: 0` | — | `top right bottom left` 统一 |
| `inset-inline: 0` | — | `inset-inline-start` + `inset-inline-end` |
| `inset-block: 0` | — | `inset-block-start` + `inset-block-end` |

## Why — 为什么

**适用场景：**

- 多语言网站/应用，需支持 LTR 和 RTL 语言
- 国际化产品（如出海 App、跨境电商）
- 设计系统/组件库需天然支持多语言
- 竖排书写场景（日文、中文竖排、蒙古文）
- 未来维护性：逻辑属性让布局与方向解耦

**传统 RTL 适配 vs 逻辑属性：**

```css
/* 传统方式：用 CSS 覆盖物理属性 */
.card {
  margin-left: 16px;
  padding-right: 24px;
  text-align: left;
  border-left: 3px solid #1677ff;
}

/* RTL 时逐条覆盖 */
[dir="rtl"] .card {
  margin-left: 0;
  margin-right: 16px;
  padding-right: 0;
  padding-left: 24px;
  text-align: right;
  border-left: none;
  border-right: 3px solid #1677ff;
}

/* 逻辑属性：一套样式自动适配 */
.card {
  margin-inline-start: 16px;
  padding-inline-end: 24px;
  text-align: start;
  border-inline-start: 3px solid #1677ff;
  /* RTL 时无需任何覆盖！ */
}
```

**优缺点：**

- ✅ 优点：
  - 一套样式自动适配 LTR/RTL，无需额外 CSS
  - 代码量减少，维护成本低
  - 天然支持竖排书写模式
  - 语义更准确（"起始边"比"左边"更通用）
  - 与 CSS Flex/Grid 的逻辑方向一致
- ❌ 缺点：
  - 学习曲线：需要从物理思维转为逻辑思维
  - 旧浏览器不支持（IE 全不支持，需要 Polyfill）
  - 部分属性仍无逻辑版本（如 `box-shadow` 偏移量）
  - 开发者工具中显示的是计算后的物理值，调试时需注意
  - 第三方 CSS 库可能未适配逻辑属性

## How — 怎么用

### 基础：方向与书写模式

```css
/* 设置页面方向 */
html {
  direction: ltr;  /* 左到右（英文、中文） */
  /* direction: rtl; */  /* 右到左（阿拉伯语、希伯来语） */
}

/* 设置书写模式 */
.vertical-text {
  writing-mode: vertical-rl;  /* 竖排从右到左（中文竖排、日文） */
  /* writing-mode: vertical-lr; */  /* 竖排从左到右（蒙古文） */
  /* writing-mode: horizontal-tb; */  /* 横排从上到下（默认） */
}
```

### 外边距与内边距

```css
/* 物理属性 → 逻辑属性 */
.element {
  /* margin */
  margin-top: 8px;           → margin-block-start: 8px;
  margin-bottom: 16px;       → margin-block-end: 16px;
  margin-left: 12px;         → margin-inline-start: 12px;
  margin-right: 24px;        → margin-inline-end: 24px;

  /* 简写 */
  margin-block: 8px 16px;    /* block-start block-end */
  margin-inline: 12px 24px;  /* inline-start inline-end */
  margin-block: 8px;         /* 上下都是 8px */
  margin-inline: auto;       /* 左右自动居中 */

  /* padding 同理 */
  padding-block-start: 8px;
  padding-block-end: 16px;
  padding-inline-start: 12px;
  padding-inline-end: 24px;
  padding-block: 8px;
  padding-inline: 16px;
}
```

### 定位

```css
/* 绝对定位 */
.badge {
  position: absolute;
  /* 物理属性 */
  top: 0;
  right: 0;
  /* → 逻辑属性 */
  inset-block-start: 0;
  inset-inline-end: 0;
}

/* 固定定位 */
.fixed-panel {
  position: fixed;
  /* 物理属性 */
  top: 64px;
  left: 0;
  bottom: 0;
  width: 280px;
  /* → 逻辑属性 */
  inset-block-start: 64px;
  inset-inline-start: 0;
  inset-block-end: 0;
  inline-size: 280px;
}

/* inset 简写 */
.overlay {
  position: fixed;
  inset: 0;              /* 四边都是 0 */
  inset-block: 0;        /* block-start + block-end = 0 */
  inset-inline: 0;       /* inline-start + inline-end = 0 */
}

.tooltip {
  position: absolute;
  inset-block-start: 100%;  /* 定位在父元素下方 */
  inset-inline-start: 0;
}
```

### 尺寸

```css
/* width → inline-size, height → block-size */
.card {
  inline-size: 300px;      /* 横排时 = width */
  block-size: 200px;       /* 横排时 = height */
  min-inline-size: 200px;  /* 横排时 = min-width */
  max-inline-size: 100%;   /* 横排时 = max-width */
  min-block-size: 100vh;   /* 横排时 = min-height */
  max-block-size: 80vh;    /* 横排时 = max-height */
}

/* 竖排书写模式下，inline-size 和 block-size 含义互换 */
.vertical-mode {
  writing-mode: vertical-rl;
  /* inline-size = height（竖排时行内方向是垂直的） */
  /* block-size = width（竖排时块方向是水平的） */
}
```

### 边框

```css
/* 边框 */
.alert {
  border-inline-start: 4px solid #1677ff;     /* LTR: 左边框 */
  border-inline-end: 1px solid #e0e0e0;       /* LTR: 右边框 */
  border-block-start: 1px solid #e0e0e0;      /* 上边框 */
  border-block-end: 1px solid #e0e0e0;        /* 下边框 */

  /* 边框圆角 */
  border-start-start-radius: 12px;   /* LTR: 左上 */
  border-start-end-radius: 12px;     /* LTR: 右上 */
  border-end-start-radius: 0;        /* LTR: 左下 */
  border-end-end-radius: 0;          /* LTR: 右下 */
}

/* 通知条：左侧色条指示 */
.notification {
  padding-inline-start: 16px;
  border-inline-start: 3px solid;
}

.notification--success { border-inline-start-color: #52c41a; }
.notification--warning { border-inline-start-color: #faad14; }
.notification--error   { border-inline-start-color: #ff4d4f; }
```

### 文本对齐

```css
/* text-align: left/right → start/end */
.content {
  text-align: start;    /* LTR: left, RTL: right */
}

.price {
  text-align: end;      /* LTR: right, RTL: left */
}

.center {
  text-align: center;   /* 居中不受方向影响 */
}
```

### 浮动与清除

```css
/* float: left/right → inline-start/inline-end */
.icon-start {
  float: inline-start;   /* LTR: float left */
  margin-inline-end: 12px;
}

.icon-end {
  float: inline-end;     /* LTR: float right */
  margin-inline-start: 12px;
}

.clearfix::after {
  content: '';
  display: table;
  clear: inline-start;   /* LTR: clear left */
}
```

### 实战：卡片组件 RTL 适配

```css
/* 物理属性版本（需要 RTL 覆盖） */
.card-physical {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px 24px;
  border-left: 3px solid #1677ff;
  border-radius: 0 8px 8px 0;
}

.card-physical .card__icon {
  margin-right: 16px;
}

.card-physical .card__arrow {
  margin-left: auto;
}

[dir="rtl"] .card-physical {
  padding: 16px 24px;
  border-left: none;
  border-right: 3px solid #1677ff;
  border-radius: 8px 0 0 8px;
}

[dir="rtl"] .card-physical .card__icon {
  margin-right: 0;
  margin-left: 16px;
}

[dir="rtl"] .card-physical .card__arrow {
  margin-left: 0;
  margin-right: auto;
  transform: scaleX(-1);
}

/* 逻辑属性版本（自动适配，无需覆盖） */
.card-logical {
  display: flex;
  align-items: center;
  gap: 16px;
  padding-inline: 24px;
  padding-block: 16px;
  border-inline-start: 3px solid #1677ff;
  border-start-start-radius: 0;
  border-start-end-radius: 8px;
  border-end-start-radius: 0;
  border-end-end-radius: 8px;
}

.card-logical .card__icon {
  margin-inline-end: 16px;
}

.card-logical .card__arrow {
  margin-inline-start: auto;
}
```

### 实战：导航栏 RTL 适配

```css
.nav {
  display: flex;
  align-items: center;
  padding-inline: 24px;
  block-size: 64px;
}

.nav__logo {
  margin-inline-end: auto;  /* LTR: 推到最右边；RTL: 推到最左边 */
}

.nav__links {
  display: flex;
  gap: 8px;
}

.nav__link {
  padding-inline: 12px;
  padding-block: 8px;
  border-radius: 8px;
  text-align: start;
}

.nav__avatar {
  margin-inline-start: 16px;
  border-start-start-radius: 50%;
  border-start-end-radius: 50%;
  border-end-start-radius: 50%;
  border-end-end-radius: 50%;
}
```

### 实战：表单布局

```css
.form-field {
  display: flex;
  align-items: center;
  gap: 12px;
}

.form-field__label {
  inline-size: 120px;
  flex-shrink: 0;
  text-align: end;  /* LTR: 右对齐；RTL: 左对齐 */
  padding-inline-end: 8px;
}

.form-field__input {
  flex: 1;
  padding-inline: 12px;
  padding-block: 8px;
  border: 1px solid #d9d9d9;
  border-start-start-radius: 6px;
  border-start-end-radius: 6px;
  border-end-start-radius: 6px;
  border-end-end-radius: 6px;
  text-align: start;
}

/* 必填标记：字段标签后面 */
.form-field__required {
  margin-inline-start: 4px;
  color: #ff4d4f;
}

/* 错误提示：字段下方起始边 */
.form-field__error {
  margin-inline-start: calc(120px + 12px); /* label宽度 + gap */
  margin-block-start: 4px;
  color: #ff4d4f;
  font-size: 0.75rem;
}
```

### 实战：聊天消息气泡

```css
.chat-messages {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
}

/* 自己的消息：靠末尾 */
.message--self {
  align-self: inline-end;  /* LTR: 靠右；RTL: 靠左 */
  background: #1677ff;
  color: #fff;
  border-start-start-radius: 16px;
  border-start-end-radius: 16px;
  border-end-start-radius: 16px;
  border-end-end-radius: 4px;  /* 末尾角小圆角 */
  padding-inline-start: 16px;
  padding-inline-end: 12px;
}

/* 对方的消息：靠起始 */
.message--other {
  align-self: inline-start;  /* LTR: 靠左；RTL: 靠右 */
  background: #f0f0f0;
  color: #333;
  border-start-start-radius: 16px;
  border-start-end-radius: 16px;
  border-end-start-radius: 4px;  /* 起始角小圆角 */
  border-end-end-radius: 16px;
  padding-inline-start: 12px;
  padding-inline-end: 16px;
}
```

### Flex 和 Grid 中的逻辑方向

```css
/* Flex 布局本身是逻辑方向 */
.flex-row {
  display: flex;
  flex-direction: row;     /* 行内方向（自动适配RTL） */
  flex-direction: column;  /* 块方向 */
}

.flex-row-reverse {
  display: flex;
  flex-direction: row-reverse;  /* 行内反方向 */
}

/* Grid 布局的 gap 等也是逻辑的 */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;           /* 行 gap + 列 gap */
  column-gap: 16px;    /* 列间距 = 行内间距 */
  row-gap: 8px;        /* 行间距 = 块间距 */
}

/* Grid 区域定位用逻辑属性 */
.grid-item {
  grid-column-start: 1;
  grid-column-end: 3;
  /* 或简写 */
  grid-column: 1 / 3;
  grid-row: 2 / 4;
}
```

### 竖排书写模式适配

```css
/* 竖排文本（中文竖排、日文竖排） */
.vertical-text {
  writing-mode: vertical-rl;
  text-orientation: mixed;   /* 默认：CJK 竖排，拉丁字母旋转 */
  /* text-orientation: upright; */  /* 所有字符直立 */
}

/* 竖排标题 */
.vertical-heading {
  writing-mode: vertical-rl;
  inline-size: auto;
  block-size: 100%;
  margin-inline: 0 16px;  /* 行内间距（竖排时=水平间距） */
}

/* 横排卡片中的竖排标签 */
.vertical-label {
  writing-mode: vertical-rl;
  padding-block: 8px;
  padding-inline: 4px;
  background: #1677ff;
  color: #fff;
  font-size: 0.75rem;
  letter-spacing: 0.1em;
}

/* 全页竖排布局 */
.vertical-page {
  writing-mode: vertical-rl;
  /* 所有 inline-size/block-size 自动互换 */
  /* margin-inline = 水平方向 */
  /* margin-block = 垂直方向 */
}
```

### CSS 自定义属性结合逻辑属性

```css
:root {
  /* 设计 Token 用逻辑属性 */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;

  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;
}

.component {
  padding-inline: var(--space-md);
  padding-block: var(--space-sm);
  margin-block-end: var(--space-md);
  border-inline-start: 3px solid #1677ff;
  border-start-end-radius: var(--radius-md);
  border-end-end-radius: var(--radius-md);
}

/* 全局间距方案 */
.flow > * + * {
  margin-block-start: var(--space-md);
}
```

### PostCSS 逻辑属性插件

```bash
# 安装 PostCSS 逻辑属性插件
npm install postcss-logical postcss-dir-pseudo-class
```

```js
// postcss.config.js
module.exports = {
  plugins: [
    // 逻辑属性 → 物理属性（兼容旧浏览器）
    require('postcss-logical')({
      dir: 'ltr',             // 默认方向
      preserve: true,         // 保留逻辑属性 + 生成物理属性
    }),
    // [dir="rtl"] 选择器支持
    require('postcss-dir-pseudo-class')(),
  ],
};
```

**编译前后对比：**

```css
/* 源码（逻辑属性） */
.card {
  margin-inline-start: 16px;
  padding-inline: 24px;
  text-align: start;
  border-inline-start: 3px solid #1677ff;
  inset-inline-end: 0;
}

/* 编译后（物理属性 + 逻辑属性，preserve: true） */
.card {
  margin-left: 16px;
  margin-inline-start: 16px;
  padding-left: 24px;
  padding-right: 24px;
  padding-inline: 24px;
  text-align: left;
  text-align: start;
  border-left: 3px solid #1677ff;
  border-inline-start: 3px solid #1677ff;
  right: 0;
  inset-inline-end: 0;
}

[dir="rtl"] .card {
  margin-right: 16px;
  margin-left: 0;
  padding-right: 24px;
  padding-left: 24px;
  text-align: right;
  border-right: 3px solid #1677ff;
  border-left: none;
  left: 0;
  right: auto;
}
```

### RTL 适配最佳实践

```html
<!-- 方式1：HTML lang + dir 属性 -->
<html lang="ar" dir="rtl">

<!-- 方式2：动态切换 -->
<script>
  function setDirection(dir) {
    document.documentElement.dir = dir;
    document.documentElement.lang = dir === 'rtl' ? 'ar' : 'en';
  }
</script>
```

```ts
// React 中动态 RTL
function App() {
  const [dir, setDir] = useState<'ltr' | 'rtl'>('ltr');

  return (
    <div dir={dir} lang={dir === 'rtl' ? 'ar' : 'en'}>
      <button onClick={() => setDir(dir === 'ltr' ? 'rtl' : 'ltr')}>
        切换方向
      </button>
      {/* 组件内容 */}
    </div>
  );
}
```

**图标翻转：**

```css
/* 方向性图标在 RTL 中需翻转 */
/* 如箭头、返回、前进等 */
[dir="rtl"] .icon-directional {
  transform: scaleX(-1);
}

/* 非方向性图标不翻转（搜索、设置、用户等） */
```

### 需要特殊处理的场景

```css
/* 1. box-shadow 偏移量（无逻辑属性版本） */
/* 需要用 CSS 变量 + 条件处理 */
.card {
  --shadow-x: 4px;
  --shadow-y: 2px;
  box-shadow: var(--shadow-x) var(--shadow-y) 8px rgba(0, 0, 0, 0.1);
}

[dir="rtl"] .card {
  --shadow-x: -4px;
}

/* 2. transform: translateX（无逻辑版本） */
/* 使用 CSS 变量 */
.tooltip {
  --translate-x: 10px;
  transform: translateX(var(--translate-x));
}

[dir="rtl"] .tooltip {
  --translate-x: -10px;
}

/* 3. background-position（无逻辑版本） */
.icon-start {
  background-position: left center;
}

[dir="rtl"] .icon-start {
  background-position: right center;
}

/* 4. gradient 方向 */
/* LTR: 从左到右 */
.gradient-lr {
  background: linear-gradient(to right, #667eea, #764ba2);
}

[dir="rtl"] .gradient-lr {
  background: linear-gradient(to left, #667eea, #764ba2);
}

/* 或使用 CSS 变量 */
.gradient-auto {
  --gradient-dir: to right;
  background: linear-gradient(var(--gradient-dir), #667eea, #764ba2);
}

[dir="rtl"] .gradient-auto {
  --gradient-dir: to left;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 逻辑属性不生效 | 旧浏览器不支持 | PostCSS 插件编译为物理属性 |
| RTL 下图标方向反了 | 方向性图标（箭头）未翻转 | `[dir="rtl"]` 下 `transform: scaleX(-1)` |
| `margin-inline` 不生效 | 误用 `margin: 0 inline 0 inline` | 逻辑简写是 `margin-inline: 12px`，不是在 `margin` 中替换 |
| Flex 布局 RTL 不翻转 | `flex-direction: row` 已是逻辑方向 | 检查是否误用 `justify-content: flex-start`（应为 `start`） |
| `text-align: start` 不生效 | 旧浏览器不支持 | 降级为 `text-align: left` + `[dir="rtl"]` 覆盖 |
| 圆角逻辑属性名记不住 | `border-start-end-radius` 命名反直觉 | 规则：第一个是块方向（start/end），第二个是行内方向（start/end） |
| `box-shadow` 方向不翻转 | 无逻辑属性版本 | CSS 变量 + `[dir="rtl"]` 覆盖 |
| 开发者工具显示物理值 | 计算值始终是物理属性 | 开发时用逻辑属性写，DevTools 中看到的计算值是物理的 |
| Google Fonts RTL 不生效 | 字体未加载 RTL 字形 | 确保引入了支持阿拉伯/希伯来字形的字体 |
| `overflow-inline` 不生效 | 较新的逻辑属性，浏览器支持有限 | 用 `overflow-x/y` + 条件覆盖 |

### 最佳实践

- 新项目默认使用逻辑属性，老项目渐进迁移
- `text-align` 优先用 `start`/`end` 替代 `left`/`right`
- 外边距/内边距优先用 `margin-inline`/`padding-inline` 简写
- 边框用 `border-inline-start` 替代 `border-left`
- 定位用 `inset-inline-start`/`inset-block-start` 替代 `left`/`top`
- 设计 Token 使用逻辑属性命名
- 方向性图标（箭头、返回）在 RTL 下翻转 `scaleX(-1)`
- `box-shadow`/`transform`/`gradient` 等无逻辑版本的属性用 CSS 变量 + `[dir="rtl"]` 处理
- 使用 PostCSS 逻辑属性插件兼容旧浏览器
- 测试时至少覆盖 LTR 英文、RTL 阿拉伯语两种方向

## 面试题

**Q1: CSS 逻辑属性和物理属性的核心区别是什么？为什么要用逻辑属性？**
> 物理属性基于屏幕方向（left/right/top/bottom），不随书写方向变化；逻辑属性基于书写方向（inline-start/inline-end/block-start/block-end），在 LTR/RTL/竖排中自动适配。使用逻辑属性的原因：① 一套样式自动适配 LTR 和 RTL，无需 `[dir="rtl"]` 覆盖；② 代码量减少，维护成本低；③ 天然支持竖排书写模式；④ 与 Flex/Grid 的逻辑方向一致，语义更准确。

**Q2: inline/block 方向在 LTR 和 RTL 中分别对应什么物理方向？**
> LTR 水平模式：inline = 左→右（left→right），block = 上→下（top→bottom）；inline-start = left，inline-end = right。RTL 水平模式：inline = 右→左（right→left），block = 上→下；inline-start = right，inline-end = left。竖排模式（vertical-rl）：inline = 上→下（top→bottom），block = 右→左（right→left）；inline-start = top，inline-end = bottom，block-start = right，block-end = left。关键：inline 是文字行进方向，block 是段落换行方向。

**Q3: 如何实现网站的 RTL 适配？有哪些方案？**
> 三种方案：① CSS 逻辑属性（推荐）：用 `margin-inline-start`/`text-align: start`/`border-inline-start` 等替代物理属性，一套样式自动适配；② `[dir="rtl"]` 选择器覆盖：对每个需要翻转的属性写 RTL 覆盖规则，工作量大但兼容性好；③ PostCSS 插件自动转换：源码写逻辑属性，编译时生成物理属性 + RTL 覆盖，兼容旧浏览器。最佳实践：新项目直接用逻辑属性 + PostCSS 降级；老项目逐步迁移，优先处理 text-align 和 margin/padding。

**Q4: 哪些 CSS 属性没有逻辑版本？如何处理？**
> 没有逻辑版本的属性：① `box-shadow` 偏移量；② `transform: translateX/Y()`；③ `background-position`；④ `linear-gradient` 方向（`to right`/`to left`）；⑤ `clip-path` 坐标。处理方式：用 CSS 变量封装方向性值，在 `[dir="rtl"]` 下修改变量值：`.card { --shadow-x: 4px; box-shadow: var(--shadow-x) 2px 8px rgba(0,0,0,0.1); } [dir="rtl"] .card { --shadow-x: -4px; }`。这种方式只需覆盖变量，不需要重复整个属性。

**Q5: border-start-start-radius 命名规则是什么？**
> 规则：`border-{block方向}-{inline方向}-radius`。第一个词是块方向（start = 上/下中的"上"，end = "下"），第二个词是行内方向（start = 左/右中的"起始"，end = "末尾"）。在 LTR 水平模式中：`border-start-start-radius` = 左上角（block-start + inline-start = top + left）；`border-start-end-radius` = 右上角（block-start + inline-end = top + right）；`border-end-start-radius` = 左下角；`border-end-end-radius` = 右下角。

**Q6: Flex 和 Grid 布局是否天然支持 RTL？**
> Flex 布局的 `flex-direction: row` 是逻辑方向（inline 方向），在 RTL 中自动翻转，天然支持。但 `justify-content: flex-start` 中的 `flex-start` 是逻辑值（等同于 `start`），也自动适配。Grid 布局的列方向也是逻辑方向。需要注意：`justify-content: left/right` 是物理值，不会自动翻转，应使用 `start/end`。Grid 的 `grid-template-columns` 中用 `1fr` 等比例值是逻辑安全的，但 `grid-column: 1 / 3` 是物理列号，RTL 下不会自动翻转列顺序。

**Q7: PostCSS 逻辑属性插件的工作原理是什么？**
> PostCSS 逻辑属性插件在构建阶段将 CSS 逻辑属性编译为物理属性：① `margin-inline-start: 16px` → `margin-left: 16px`（LTR 默认）；② 如果 `preserve: true`，同时保留逻辑属性和生成物理属性，现代浏览器用逻辑属性（优先级更高），旧浏览器用物理属性；③ 同时生成 `[dir="rtl"]` 选择器下的覆盖规则：`[dir="rtl"] .card { margin-right: 16px; margin-left: 0; }`。这样开发者只需写逻辑属性，插件自动处理兼容性。

**Q8: 竖排书写模式下逻辑属性有什么特殊行为？**
> 竖排模式（`writing-mode: vertical-rl`）下，inline 方向变为垂直（上→下），block 方向变为水平（右→左），因此：`inline-size` = height（行内方向尺寸 = 高度）；`block-size` = width（块方向尺寸 = 宽度）；`margin-inline-start` = margin-top（行内起始 = 上方）；`margin-block-start` = margin-right（块起始 = 右侧）；`padding-inline` = 上下内边距；`padding-block` = 左右内边距。逻辑属性的优势在于：同一套 CSS 在横排和竖排下自动适配，无需为竖排模式写特殊覆盖。

---

**相关链接：**
- [[CSS新特性与现代布局]]
- [[CSS布局Flex与Grid]]
- [[CSS变量与主题系统]]
- [[CSS架构方法论]]
- [[国际化i18n]]
- [[CSS预处理器]]

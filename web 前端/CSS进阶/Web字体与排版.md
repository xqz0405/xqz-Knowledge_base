---
tags:
  - Web前端
  - CSS进阶
  - 字体
  - 排版
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Web 字体与排版

## What — 什么是 Web 字体与排版

Web 字体与排版是前端视觉呈现的基石。它涵盖从字体加载渲染、CSS 排版属性到中文排版特殊需求的全链路技术体系，主要包含三大领域：

### 1. Web 字体技术

| 技术 | 核心能力 | 解决问题 |
|------|----------|----------|
| `@font-face` | 自定义字体声明与加载 | 突破系统字体限制，实现品牌字体 |
| WOFF2 格式 | 高压缩比 Web 字体格式 | 减少字体文件体积，加速加载 |
| 可变字体（Variable Font） | 单文件包含多字重/宽度变体 | 减少 HTTP 请求，支持字体动画 |
| Font Face API | JS 控制字体加载生命周期 | 精细控制字体加载时机与回退 |

### 2. CSS 排版属性

| 属性 | 核心能力 | 适用场景 |
|------|----------|----------|
| `line-height` | 行高控制 | 段落可读性、垂直居中 |
| `letter-spacing` / `word-spacing` | 字/词间距 | 标题装饰、排版微调 |
| `text-align` / `text-indent` | 对齐与缩进 | 段落排版、中文首行缩进 |
| `hanging-punctuation` | 标点悬挂 | 中文排版标点处理 |
| `writing-mode` | 书写模式 | 竖排中文、混排布局 |

### 3. 中文排版特点

| 特点 | 说明 | 对应方案 |
|------|------|----------|
| 字符集庞大 | 常用汉字 6000+，完整字库 8 万+ | 字体子集化、CDN 动态加载 |
| 标点规则复杂 | 首行缩进、标点悬挂、避头尾 | `text-indent`、`hanging-punctuation` |
| 竖排需求 | 传统中文竖排阅读习惯 | `writing-mode: vertical-rl` |
| 中西文混排 | 中英文间距、字号搭配 | `text-spacing`、`unicode-range` |

---

## Why — 为什么需要 Web 字体与排版技术

### 1. 系统字体有限，品牌设计需要自定义字体

不同操作系统内置字体差异巨大。Windows 有微软雅黑、macOS 有苹方、Linux 有文泉驿，同一页面在不同平台呈现效果不一致。品牌设计往往要求使用特定字体，`@font-face` 让设计师的字体选择不再受限于用户系统。

### 2. 可变字体减少请求，提升性能

传统方案中，一个字体的 Regular / Bold / Italic / Bold Italic 四种样式需要四个独立文件。可变字体将所有变体打包进单个文件，通过轴（axis）参数控制样式，大幅减少 HTTP 请求和总传输体积。

### 3. 中文排版有独特需求

中文排版与西文排版差异显著：中文需要首行缩进两个字符、标点避头尾规则、竖排阅读模式、中西文混排间距处理。CSS 排版属性（如 `hanging-punctuation`、`text-spacing`、`writing-mode`）正是为满足这些需求而生。

### 4. 字体加载影响用户体验

未优化的字体加载会导致 FOIT（Flash of Invisible Text）和 FOUT（Flash of Unstyled Text），严重影响页面可读性和用户体验。`font-display`、Font Face API、preload 等技术让开发者精细控制字体加载策略。

---

## 对比：传统字体 vs 可变字体

| 对比项 | 传统字体 | 可变字体 |
|--------|----------|----------|
| 文件数量 | 每种样式一个文件（Regular/Bold/Italic...） | 单文件包含所有变体 |
| 样式数量 | 有限（通常 4-6 个文件） | 几乎无限（轴参数连续变化） |
| HTTP 请求 | 多个（N 个样式 = N 个请求） | 1 个 |
| 动画支持 | 不支持（样式间无法过渡） | 支持（轴值可动画化） |
| 总文件体积 | 各文件累加，总体较大 | 单文件略大，但远小于多文件总和 |
| 兼容性 | 全部浏览器支持 | Chrome 69+、Firefox 62+、Safari 12+ |
| 设计灵活性 | 固定样式，无法中间态 | 可在任意字重/宽度间插值 |
| 典型文件大小（英文） | 每个 30-80KB | 单个 100-200KB |
| 典型文件大小（中文） | 每个 2-8MB | 单个 5-15MB |

---

## How — 各技术详解与实战

### 1. @font-face 详解

#### 完整语法

```css
@font-face {
  font-family: "MyFont";         /* 字体名称，使用时引用 */
  src: local("MyFont"),          /* 优先查找本地安装的字体 */
       url("myfont.woff2") format("woff2"),  /* WOFF2 格式（优先） */
       url("myfont.woff") format("woff"),    /* WOFF 格式（回退） */
       url("myfont.ttf") format("truetype"); /* TTF 格式（最终回退） */
  font-weight: 400;              /* 字重范围：100-900 或 normal/bold */
  font-style: normal;            /* normal / italic / oblique */
  font-display: swap;            /* 字体加载策略 */
  font-stretch: normal;          /* 字宽：normal / condensed / expanded */
  unicode-range: U+0020-007E;    /* 仅加载指定 Unicode 范围的字符 */
  font-variant: normal;          /* 字体变体 */
  font-feature-settings: "kern" 1; /* OpenType 特性 */
}
```

#### 使用示例

```css
/* 基础使用 */
body {
  font-family: "MyFont", sans-serif;
}

/* 多字重声明 — 每个字重单独声明 */
@font-face {
  font-family: "MyFont";
  src: url("myfont-regular.woff2") format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "MyFont";
  src: url("myfont-bold.woff2") format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "MyFont";
  src: url("myfont-italic.woff2") format("woff2");
  font-weight: 400;
  font-style: italic;
  font-display: swap;
}

/* 使用不同字重和样式 */
h1 { font-family: "MyFont", sans-serif; font-weight: 700; }
p  { font-family: "MyFont", sans-serif; font-weight: 400; }
em { font-family: "MyFont", sans-serif; font-style: italic; }
```

#### local() — 本地字体优先

```css
@font-face {
  font-family: "SystemFont";
  /* 优先使用用户系统已安装的字体，避免重复下载 */
  src: local("PingFang SC"),          /* macOS 苹方 */
       local("Microsoft YaHei"),       /* Windows 微软雅黑 */
       local("Noto Sans SC"),          /* Linux 思源黑体 */
       url("fallback.woff2") format("woff2"); /* 均不存在时才下载 */
}
```

#### format 格式说明

```css
@font-face {
  src: url("font.woff2") format("woff2"),    /* WOFF2：最佳压缩，优先使用 */
       url("font.woff")  format("woff"),     /* WOFF：中等压缩，广泛支持 */
       url("font.ttf")   format("truetype"), /* TTF：无压缩，兼容性好 */
       url("font.otf")   format("opentype"), /* OTF：无压缩，支持 OpenType 特性 */
       url("font.eot")   format("embedded-opentype"); /* EOT：仅 IE 支持 */
}
```

**浏览器按 `src` 列表顺序查找**，遇到第一个支持的格式即下载。因此应将 WOFF2 放在前面。

---

### 2. 字体格式对比

| 格式 | 全称 | 压缩 | 典型体积 | 浏览器支持 | 推荐度 |
|------|------|------|----------|------------|--------|
| WOFF2 | Web Open Font Format 2 | Brotli 厃缩 | 最小（比 WOFF 小 30%） | Chrome 36+、Firefox 39+、Safari 12+ | 必须提供 |
| WOFF | Web Open Font Format | zlib 压缩 | 较小 | 几乎所有现代浏览器 | 建议提供 |
| TTF | TrueType Font | 无 | 较大 | 所有浏览器 | 作为回退 |
| OTF | OpenType Font | 无 | 较大 | 所有浏览器 | 作为回退 |
| EOT | Embedded OpenType | 私有压缩 | 较小 | 仅 IE | 已弃用 |

**格式选择策略**：

1. **必须提供 WOFF2**：压缩率最高，现代浏览器均支持
2. **建议提供 WOFF 作为回退**：覆盖极少数不支持 WOFF2 的场景
3. **可选 TTF/OTF**：如需兼容极旧浏览器
4. **EOT 已弃用**：IE 已退出历史舞台，无需再提供
5. **生产环境中通常只需 WOFF2 + WOFF 即可**

```css
/* 生产环境推荐写法 */
@font-face {
  font-family: "MyFont";
  src: url("myfont.woff2") format("woff2"),
       url("myfont.woff") format("woff");
  font-weight: 400;
  font-display: swap;
}
```

---

### 3. font-display 策略详解

`font-display` 控制 Web 字体加载前后的文字渲染行为，核心问题是处理 FOIT 和 FOUT：

- **FOIT**（Flash of Invisible Text）：字体加载期间文字不可见，加载完成后突然出现
- **FOUT**（Flash of Unstyled Text）：字体加载期间先显示回退字体，加载完成后切换为自定义字体

#### 五种策略对比

| 策略 | 阻塞期 | 交换期 | 行为描述 | 适用场景 |
|------|--------|--------|----------|----------|
| `auto` | 由浏览器决定 | 由浏览器决定 | 默认值，多数浏览器表现为 FOIT | 不推荐 |
| `block` | 最长 3s | 无限 | 阻塞期文字不可见，之后用回退字体；字体加载完后切换 | 图标字体（文字不可用回退替代） |
| `swap` | 0s（无阻塞） | 无限 | 立即显示回退字体，加载完后切换 | 品牌展示字体（正文不建议） |
| `fallback` | 约 100ms | 约 3s | 极短阻塞后显示回退字体，若 3s 内加载完则切换 | 大多数正文内容的推荐选择 |
| `optional` | 约 100ms | 0s（无交换） | 极短阻塞后显示回退字体，仅当字体已缓存时使用自定义字体 | 性能敏感的正文内容 |

#### 各策略时间线

```
auto:     [---阻塞期(浏览器决定)---][---交换期(浏览器决定)---]
block:    [-------阻塞期(3s)-------][----------交换期----------]
swap:     [阻塞期(0s)][---------------交换期----------------]
fallback: [阻塞(100ms)][-------交换期(~3s)-------]
optional: [阻塞(100ms)][无交换期]
```

#### 选择建议

```css
/* 图标字体 — 必须加载完成才可用，否则显示方块 */
@font-face {
  font-family: "IconFont";
  src: url("iconfont.woff2") format("woff2");
  font-display: block;
}

/* 品牌标题 — 立即可读，加载完后切换为品牌字体 */
@font-face {
  font-family: "BrandFont";
  src: url("brand.woff2") format("woff2");
  font-display: swap;
}

/* 正文内容 — 平衡可读性与视觉一致性 */
@font-face {
  font-family: "BodyFont";
  src: url("body.woff2") format("woff2");
  font-display: fallback;
}

/* 性能敏感场景 — 弱网环境不等待 */
@font-face {
  font-family: "OptionalFont";
  src: url("optional.woff2") format("woff2");
  font-display: optional;
}
```

---

### 4. 可变字体（Variable Font）

可变字体是 OpenType 字体规范的一个扩展，允许一个字体文件包含多种样式变体，通过"轴"（axis）参数来控制。

#### 注册轴（Registered Axes）— 规范定义的五轴

| 轴名 | 全称 | CSS 属性 | 值范围 | 说明 |
|------|------|----------|--------|------|
| `wght` | Weight | `font-weight` | 1-999 | 字重（粗细） |
| `wdth` | Width | `font-stretch` | 50%-200% | 字宽（宽窄） |
| `ital` | Italic | `font-style` | 0/1 | 斜体开关 |
| `slnt` | Slant | `font-style` | -90~90 | 倾斜角度 |
| `opsz` | Optical Size | `font-optical-sizing` | 8-144 | 光学尺寸 |

#### 基础使用

```css
/* 声明可变字体 */
@font-face {
  font-family: "VariableFont";
  src: url("variable-font.woff2") format("woff2");
  /* 支持的字重范围 */
  font-weight: 100 900;
  font-stretch: 75% 125%;
  font-display: swap;
}

/* 使用标准 CSS 属性控制 */
.light-text {
  font-family: "VariableFont", sans-serif;
  font-weight: 300;        /* 通过 wght 轴控制 */
  font-stretch: 100%;      /* 通过 wdth 轴控制 */
}

.bold-condensed {
  font-family: "VariableFont", sans-serif;
  font-weight: 800;
  font-stretch: 80%;       /* 压窄 */
}
```

#### font-variation-settings — 底层轴控制

```css
/* 标准属性无法覆盖自定义轴时，使用 font-variation-settings */
.custom-variation {
  font-family: "VariableFont", sans-serif;
  font-variation-settings:
    "wght" 450,       /* 字重 450 — 标准属性只有 100 的整数倍 */
    "wdth" 85,        /* 宽度 85% */
    "opsz" 32;        /* 光学尺寸 32 */
}

/* 自定义轴 — 以大写字母开头 */
.decorative {
  font-variation-settings:
    "wght" 700,
    "GRAD" 50,        /* 自定义轴：等级 */
    "XTRA" 468;       /* 自定义轴：额外宽度 */
}
```

#### 可变字体动画

```css
/* 字重动画 — hover 时字重从 400 渐变到 800 */
.animate-weight {
  font-family: "VariableFont", sans-serif;
  font-variation-settings: "wght" 400;
  transition: font-variation-settings 0.3s ease;
}

.animate-weight:hover {
  font-variation-settings: "wght" 800;
}

/* 宽度动画 — 点击时字宽变化 */
.animate-width {
  font-family: "VariableFont", sans-serif;
  font-variation-settings: "wdth" 100;
  transition: font-variation-settings 0.4s ease-in-out;
}

.animate-width:active {
  font-variation-settings: "wdth" 75;
}

/* 多轴组合动画 */
@keyframes variable-dance {
  0%   { font-variation-settings: "wght" 200, "wdth" 125; }
  25%  { font-variation-settings: "wght" 400, "wdth" 100; }
  50%  { font-variation-settings: "wght" 800, "wdth" 75;  }
  75%  { font-variation-settings: "wght" 400, "wdth" 100; }
  100% { font-variation-settings: "wght" 200, "wdth" 125; }
}

.dance-text {
  font-family: "VariableFont", sans-serif;
  animation: variable-dance 4s ease-in-out infinite;
}

/* 光学尺寸随字号自适应 */
.responsive-optical {
  font-family: "VariableFont", sans-serif;
  font-optical-sizing: auto; /* 自动根据字号调整 opsz 轴 */
}
```

#### 可变字体的 JS 检测

```js
// 检测浏览器是否支持可变字体
const supportsVariableFonts = CSS.supports(
  "font-variation-settings", '"wght" 400'
);

if (supportsVariableFonts) {
  document.documentElement.classList.add("variable-fonts");
} else {
  document.documentElement.classList.add("static-fonts");
}

// 检测字体是否包含特定轴
async function checkFontAxes(fontFamily) {
  try {
    const fonts = await document.fonts.values();
    for (const font of fonts) {
      if (font.family === fontFamily && font.variationAxes) {
        console.log('可用轴:', Object.keys(font.variationAxes));
        return font.variationAxes;
      }
    }
  } catch (e) {
    console.warn('无法检测字体轴');
  }
  return null;
}
```

---

### 5. 字体加载优化

#### 5.1 preload 预加载关键字体

```html
<!-- 在 <head> 中预加载关键字体，尽早发起请求 -->
<link
  rel="preload"
  href="/fonts/main-regular.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>
<!-- crossorigin 必须添加，否则字体请求不会携带凭证 -->
```

```css
/* 配合 preload，字体声明仍然需要 */
@font-face {
  font-family: "MainFont";
  src: url("/fonts/main-regular.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
}
```

#### 5.2 FontFace API — JS 控制字体加载

```js
// 方式一：使用 FontFace 构造函数
const fontFace = new FontFace(
  "MyFont",
  "url(/fonts/myfont.woff2)",
  {
    weight: "400",
    style: "normal",
    display: "swap"
  }
);

// 加载字体
fontFace.load().then((loadedFont) => {
  // 添加到文档字体集
  document.fonts.add(loadedFont);
  // 字体已就绪，可以安全使用
  document.body.style.fontFamily = '"MyFont", sans-serif';
}).catch((error) => {
  console.error("字体加载失败:", error);
  // 回退到系统字体
  document.body.style.fontFamily = 'sans-serif';
});

// 方式二：监听所有字体加载完成
document.fonts.ready.then(() => {
  document.documentElement.classList.add("fonts-loaded");
});

// 方式三：检测特定字体是否已加载
async function isFontLoaded(fontFamily) {
  await document.fonts.load(`16px "${fontFamily}"`);
  return document.fonts.check(`16px "${fontFamily}"`);
}

// 方式四：使用 check() 检查（不触发加载）
if (document.fonts.check('16px "MyFont"')) {
  console.log('字体已加载');
} else {
  console.log('字体未加载');
}
```

#### 5.3 size-adjust — 字体度量对齐

`size-adjust` 用于调整自定义字体的度量，使其与回退字体视觉对齐，减少 FOUT 时的布局偏移（CLS）。

```css
/* 回退字体 — 调整度量使其与 Web 字体对齐 */
@font-face {
  font-family: "FallbackFont";
  src: local("Arial");
  size-adjust: 105.2%;       /* 放大 5.2% 以匹配 Web 字体 */
  ascent-override: 98%;      /* 覆盖上升度量 */
  descent-override: 25%;     /* 覆盖下降度量 */
  line-gap-override: 0%;     /* 覆盖行间距 */
}

/* Web 字体 */
@font-face {
  font-family: "WebFont";
  src: url("webfont.woff2") format("woff2");
  font-display: swap;
}

/* 使用 — 回退字体与 Web 字体视觉对齐 */
body {
  font-family: "WebFont", "FallbackFont", sans-serif;
}
```

**计算 size-adjust 的方法**：

```js
// 工具函数：计算两种字体间的 size-adjust 比例
function calculateSizeAdjust(webFontMetrics, fallbackFontMetrics) {
  const webRatio = webFontMetrics.ascent / webFontMetrics.unitsPerEm;
  const fallbackRatio = fallbackFontMetrics.ascent / fallbackFontMetrics.unitsPerEm;
  return (webRatio / fallbackRatio) * 100;
}

// 实际操作：使用在线工具更方便
// https://sf-design-system.vercel.app/font-matcher
// https://kwLTon.github.io/font-style-matcher/
```

#### 5.4 unicode-range 子集化

```css
/* 仅加载拉丁字母字符 — 大幅减小文件 */
@font-face {
  font-family: "MyFont";
  src: url("myfont-latin.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
  unicode-range: U+0020-007E; /* 基本拉丁字母 */
}

/* 仅加载中文常用字符 */
@font-face {
  font-family: "MyFont";
  src: url("myfont-cjk.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
  unicode-range: U+4E00-9FFF; /* CJK 统一汉字 */
}

/* 仅加载日文假名 */
@font-face {
  font-family: "MyFont";
  src: url("myfont-kana.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
  unicode-range: U+3040-30FF; /* 平假名 + 片假名 */
}

/* 浏览器仅在页面包含对应 Unicode 范围的字符时才会下载对应字体文件 */
```

**常用 Unicode 范围**：

| 范围 | 说明 |
|------|------|
| `U+0020-007E` | 基本拉丁字母（ASCII） |
| `U+00A0-00FF` | 拉丁字母补充 |
| `U+2000-206F` | 通用标点 |
| `U+4E00-9FFF` | CJK 统一汉字（常用中文） |
| `U+3400-4DBF` | CJK 统一汉字扩展 A |
| `U+3000-303F` | CJK 符号和标点 |
| `U+FF00-FFEF` | 半角及全角字符 |
| `U+3040-309F` | 平假名 |
| `U+30A0-30FF` | 片假名 |

#### 5.5 Critical FOFT 策略

Critical FOFT（Flash of Faux Text）是一种进阶字体加载策略，分两阶段加载：

```
第一阶段（关键）：加载可变字体的最小子集（仅包含页面首屏需要的字符）
第二阶段（增强）：异步加载完整字符集
```

```js
// Critical FOFT 实现
class FontLoader {
  constructor() {
    this.criticalFont = "CriticalFont";
    this.fullFont = "FullFont";
  }

  // 第一阶段：加载关键子集
  async loadCritical() {
    const criticalFont = new FontFace(
      this.criticalFont,
      "url(/fonts/critical-subset.woff2)",
      { display: "swap", weight: "100 900" }
    );
    const loaded = await criticalFont.load();
    document.fonts.add(loaded);
    document.documentElement.classList.add("critical-font-loaded");
  }

  // 第二阶段：加载完整字体
  async loadFull() {
    // 使用 requestIdleCallback 在浏览器空闲时加载
    const loadFullFont = () => {
      const fullFont = new FontFace(
        this.fullFont,
        "url(/fonts/full-font.woff2)",
        { display: "swap", weight: "100 900" }
      );
      fullFont.load().then((loaded) => {
        document.fonts.add(loaded);
        document.documentElement.classList.add("full-font-loaded");
      });
    };

    if ("requestIdleCallback" in window) {
      requestIdleCallback(loadFullFont);
    } else {
      setTimeout(loadFullFont, 1000);
    }
  }

  async init() {
    await this.loadCritical();
    this.loadFull(); // 不 await，异步加载
  }
}

// 使用
const loader = new FontLoader();
loader.init();
```

```css
/* 配合 CSS 阶段切换 */
body {
  font-family: system-ui, sans-serif;
}

.critical-font-loaded body {
  font-family: "CriticalFont", system-ui, sans-serif;
}

.full-font-loaded body {
  font-family: "FullFont", "CriticalFont", system-ui, sans-serif;
}
```

---

### 6. 中文 Web 字体方案

中文字体的核心挑战是**体量大**：一个完整的中文字体文件通常 2-8MB，远大于英文的 30-80KB。

#### 6.1 字体子集化

```bash
# 使用 fontmin（Node.js 工具）提取指定字符的子集
npm install fontmin --save-dev
```

```js
// fontmin 配置
const Fontmin = require("fontmin");

const fontmin = new Fontmin()
  .src("src/fonts/NotoSansSC-Regular.ttf")
  .use(Fontmin.glyph({
    text: "需要提取的中文字符",  // 仅包含这些字符
    hinting: false                 // 关闭 hinting 减小体积
  }))
  .use(Fontmin.ttf2woff2())       // 转为 WOFF2
  .dest("dist/fonts/");

fontmin.run((err, files) => {
  if (err) throw err;
  console.log("子集化完成:", files);
});
```

```bash
# 使用 pyftsubset（Python 工具）
pip install fonttools brotli

# 基础子集化
pyftsubset NotoSansSC-Regular.ttf \
  --text-file=chars.txt \
  --output-file=NotoSansSC-subset.woff2 \
  --flavor=woff2 \
  --layout-features='*'

# 基于 Unicode 范围子集化
pyftsubset NotoSansSC-Regular.ttf \
  --unicodes="U+4E00-9FFF" \
  --output-file=NotoSansSC-cjk.woff2 \
  --flavor=woff2
```

#### 6.2 CDN 动态加载

```html
<!-- 有字库 — 国内常用的中文 Web 字体 CDN -->
<link
  rel="stylesheet"
  href="https://cdn.fontcdn.org/2024/css?family=Noto+Sans_SC:wght@400;700&display=swap"
/>

<!-- Google Fonts — 需要访问 Google 服务 -->
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap"
/>
```

#### 6.3 分片加载策略

```js
// 将大字体文件按 Unicode 范围分片，按需加载
const fontSlices = [
  { name: "latin",     range: "U+0020-007E", url: "/fonts/latin.woff2"     },
  { name: "cjk-basic", range: "U+4E00-6FFF", url: "/fonts/cjk-basic.woff2" },
  { name: "cjk-ext1",  range: "U+7000-8FFF", url: "/fonts/cjk-ext1.woff2"  },
  { name: "cjk-ext2",  range: "U+9000-9FFF", url: "/fonts/cjk-ext2.woff2"  },
  { name: "symbols",   range: "U+3000-303F", url: "/fonts/symbols.woff2"   },
];

// 检测页面使用了哪些 Unicode 范围
function detectUsedRanges() {
  const text = document.body.innerText;
  const ranges = new Set();

  for (const char of text) {
    const code = char.codePointAt(0);
    if (code >= 0x0020 && code <= 0x007E) ranges.add("latin");
    else if (code >= 0x4E00 && code <= 0x6FFF) ranges.add("cjk-basic");
    else if (code >= 0x7000 && code <= 0x8FFF) ranges.add("cjk-ext1");
    else if (code >= 0x9000 && code <= 0x9FFF) ranges.add("cjk-ext2");
    else if (code >= 0x3000 && code <= 0x303F) ranges.add("symbols");
  }

  return [...ranges];
}

// 按需加载
async function loadSlicedFonts() {
  const usedRanges = detectUsedRanges();
  const neededSlices = fontSlices.filter(s => usedRanges.includes(s.name));

  // 优先加载基础拉丁和常用汉字
  const criticalSlices = neededSlices.filter(s =>
    s.name === "latin" || s.name === "cjk-basic"
  );
  const deferredSlices = neededSlices.filter(s =>
    s.name !== "latin" && s.name !== "cjk-basic"
  );

  // 关键字体同步加载
  for (const slice of criticalSlices) {
    const font = new FontFace("MyCJKFont", `url(${slice.url})`, {
      display: "swap",
      unicodeRange: slice.range
    });
    const loaded = await font.load();
    document.fonts.add(loaded);
  }

  // 其余字体空闲时加载
  requestIdleCallback(() => {
    for (const slice of deferredSlices) {
      const font = new FontFace("MyCJKFont", `url(${slice.url})`, {
        display: "swap",
        unicodeRange: slice.range
      });
      font.load().then(loaded => document.fonts.add(loaded));
    }
  });
}

loadSlicedFonts();
```

#### 6.4 系统字体栈 Fallback 策略

```css
/* 中文系统字体栈 — 按平台排列 */
:root {
  --font-sans: "PingFang SC",       /* macOS */
               "Microsoft YaHei",    /* Windows */
               "Noto Sans SC",       /* Linux / Android */
               "Source Han Sans SC", /* Adobe 思源黑体 */
               "WenQuanYi Micro Hei", /* Linux 旧版 */
               sans-serif;

  --font-serif: "Songti SC",         /* macOS 宋体 */
                "SimSun",            /* Windows 宋体 */
                "Noto Serif SC",     /* Linux 思源宋体 */
                serif;

  --font-mono: "SF Mono",            /* macOS */
               "Cascadia Code",      /* Windows */
               "Fira Code",          /* 跨平台 */
               "Source Code Pro",    /* 跨平台 */
               monospace;
}

/* 带自定义 Web 字体的完整字体栈 */
body {
  font-family: var(--font-sans);
}

/* Web 字体加载后替换 */
.fonts-loaded body {
  font-family: "MyWebFont", var(--font-sans);
}

/* 中英文分别使用不同字体 — 利用 unicode-range */
@font-face {
  font-family: "MixedFont";
  src: local("PingFang SC"), local("Microsoft YaHei");
  unicode-range: U+4E00-9FFF; /* 中文使用系统字体 */
}

@font-face {
  font-family: "MixedFont";
  src: url("inter-latin.woff2") format("woff2");
  unicode-range: U+0020-007E; /* 英文使用 Web 字体 */
}

body {
  font-family: "MixedFont", sans-serif;
}
```

---

### 7. CSS 排版属性详解

#### 7.1 line-height — 行高

```css
/* 行高的取值方式 */
.line-height-demo {
  /* 无单位值 — 相对于元素自身 font-size（推荐） */
  line-height: 1.5;  /* 16px * 1.5 = 24px */

  /* 带单位值 — 固定值，不继承计算结果 */
  line-height: 24px; /* 固定 24px，子元素不会重新计算 */

  /* 百分比 — 继承计算后的固定值 */
  line-height: 150%; /* 同 1.5，但子元素继承的是计算值 */

  /* normal — 浏览器默认，通常约 1.2 */
  line-height: normal;
}

/* 关键区别：无单位 vs 百分比的继承差异 */
.parent {
  font-size: 16px;
  line-height: 1.5;  /* 计算 = 24px，但子元素继承 1.5 这个系数 */
}

.parent-percent {
  font-size: 16px;
  line-height: 150%; /* 计算 = 24px，子元素继承 24px 这个固定值 */
}

.child {
  font-size: 32px;
  /* 继承无单位 1.5 → 32 * 1.5 = 48px（正确） */
  /* 继承百分比 24px → 行高仅 24px，文字溢出（错误） */
}
```

**行高推荐值**：

| 内容类型 | 推荐 line-height | 说明 |
|----------|-----------------|------|
| 正文段落 | 1.5 - 1.8 | 中文正文需要较大行高 |
| 标题 | 1.2 - 1.4 | 标题字大，行高可紧凑 |
| 代码 | 1.4 - 1.6 | 代码需要行间清晰 |
| 移动端 | 1.6 - 1.8 | 移动端阅读需要更大行高 |

#### 7.2 letter-spacing & word-spacing

```css
/* letter-spacing — 字间距 */
.title {
  letter-spacing: 0.1em;  /* 标题常加字间距增加装饰感 */
}

.caption {
  letter-spacing: 0.05em;
}

/* 极端值用于特殊设计 */
.logo-text {
  letter-spacing: 0.3em;
  text-transform: uppercase;
}

/* word-spacing — 词间距（主要影响西文） */
.english-text {
  word-spacing: 0.2em;    /* 增大英文词间距 */
}

/* 中英文混排时间距调整 */
.mixed-text {
  letter-spacing: 0.02em; /* 中文微调字间距 */
  word-spacing: 0.1em;    /* 英文微调词间距 */
}
```

#### 7.3 text-align & text-indent

```css
/* text-align — 文本对齐 */
.text-left    { text-align: left;       } /* 左对齐（默认） */
.text-right   { text-align: right;      } /* 右对齐 */
.text-center  { text-align: center;     } /* 居中 */
.text-justify { text-align: justify;    } /* 两端对齐 */
.text-match-parent {
  text-align: match-parent;              /* 继承父元素方向 */
}

/* 中文排版 — 首行缩进两个字符 */
p {
  text-indent: 2em;       /* 缩进两个全角字符 */
  text-align: justify;    /* 两端对齐 */
}

/* 首行不缩进（如紧跟标题后的段落） */
h2 + p {
  text-indent: 0;
}

/* 悬挂缩进 */
.hanging-indent {
  text-indent: -2em;     /* 首行向左偏移 */
  padding-left: 2em;     /* 整体右移保持对齐 */
}
```

#### 7.4 hanging-punctuation — 标点悬挂

```css
/* 标点悬挂 — 让行首/行尾标点悬挂到边距外 */
.chinese-text {
  hanging-punctuation: first;     /* 行首标点悬挂 */
  hanging-punctuation: last;      /* 行尾标点悬挂 */
  hanging-punctuation: first last; /* 两端都悬挂 */
  hanging-punctuation: allow-end;  /* 行尾允许悬挂（非强制） */
  hanging-punctuation: force-end;  /* 行尾强制悬挂 */
}

/* 完整中文排版 */
.article p {
  text-indent: 2em;
  text-align: justify;
  hanging-punctuation: first last allow-end;
}
```

#### 7.5 text-spacing（CSS Text Level 4）

```css
/* 中西文混排自动间距 — 减少手动调整 */
.auto-spacing {
  text-spacing: auto;  /* 浏览器自动处理中西文间距 */
}

/* 精细控制 */
.text-spacing-detail {
  text-spacing:
    trim-start allow-end,   /* 行首标点压缩 */
    trim-end allow-end,     /* 行尾标点压缩 */
    trim-adjacent allow,    /* 相邻标点压缩 */
    space-start allow,      /* 行首空白 */
    space-end allow,        /* 行尾空白 */
    space-adjacent allow;   /* 标点间空白 */
}

/* 禁止标点压缩 — 保持标点原始间距 */
.no-trim {
  text-spacing: trim-start no-trim trim-adjacent no-trim;
}
```

#### 7.6 orphans & widows — 孤行控制

```css
/* orphans — 页面/列底部至少保留的行数 */
.article {
  orphans: 3;  /* 段落最后一部分至少 3 行留在当前页/列 */
}

/* widows — 页面/列顶部至少保留的行数 */
.article {
  widows: 3;   /* 段落开始部分至少 3 行移到新页/列 */
}

/* 打印排版 */
@media print {
  p {
    orphans: 3;
    widows: 3;
  }
}
```

---

### 8. 垂直排版

```css
/* writing-mode — 书写模式 */
.vertical-text {
  writing-mode: vertical-rl;  /* 竖排，从右到左（传统中文） */
}

.vertical-lr {
  writing-mode: vertical-lr;  /* 竖排，从左到右（蒙古文等） */
}

.horizontal-tb {
  writing-mode: horizontal-tb; /* 横排，从上到下（默认） */
}

/* 竖排中文排版 */
.vertical-article {
  writing-mode: vertical-rl;
  text-orientation: mixed;      /* 混合方向：汉字直立，字母旋转 */
  line-height: 1.8;
  letter-spacing: 0.05em;
  height: 100vh;                /* 竖排高度 = 横排宽度 */
  overflow-x: auto;             /* 横向滚动（竖排的"向下"） */
}

/* 纯汉字直立 */
.vertical-cjk {
  writing-mode: vertical-rl;
  text-orientation: upright;    /* 所有字符直立 */
}

/* 竖排中的标题 */
.vertical-article h1 {
  writing-mode: vertical-rl;
  font-size: 2rem;
  letter-spacing: 0.2em;
  margin-left: 1em;             /* 竖排中 margin-left = 上一列的间距 */
}

/* 横竖混排 — 标题横排、正文竖排 */
.mixed-layout {
  display: flex;
  flex-direction: row-reverse;  /* 竖排阅读顺序从右到左 */
}

.mixed-layout .title {
  writing-mode: horizontal-tb;
}

.mixed-layout .content {
  writing-mode: vertical-rl;
  flex: 1;
}
```

```html
<div class="vertical-article">
  <h1>古诗标题</h1>
  <p>
    床前明月光，疑是地上霜。
    举头望明月，低头思故乡。
  </p>
</div>
```

**竖排中的文本方向属性**：

| 属性 | 值 | 说明 |
|------|-----|------|
| `text-orientation` | `mixed` | 汉字直立，字母旋转（默认） |
| `text-orientation` | `upright` | 所有字符直立 |
| `text-orientation` | `sideways` | 所有字符旋转 90 度 |
| `text-combine-upright` | `all` | 竖排中的数字/字母压缩为一字宽 |
| `text-combine-upright` | `digits 2` | 最多 2 位数字压缩 |

```css
/* 竖排中的数字处理 — 年份、日期等 */
.vertical-text .year {
  text-combine-upright: all;     /* "2024" 压缩为一个字宽 */
}

.vertical-text .date {
  text-combine-upright: digits 2; /* 最多 2 位数字压缩 */
}
```

---

### 9. 响应式排版

#### 9.1 clamp() — 流式字号

```css
/* clamp(最小值, 首选值, 最大值) */
h1 {
  /* 字号在 1.5rem ~ 3rem 之间，随视口宽度平滑过渡 */
  font-size: clamp(1.5rem, 4vw, 3rem);
}

h2 {
  font-size: clamp(1.25rem, 3vw, 2rem);
}

p {
  font-size: clamp(0.875rem, 1.5vw, 1.125rem);
  line-height: clamp(1.5, 1.5 + 0.2vw, 1.8);
}

/* 流式行高配合流式字号 */
.hero-title {
  font-size: clamp(2rem, 5vw + 1rem, 5rem);
  line-height: 1.1;
  letter-spacing: clamp(-0.02em, -0.5vw, -0.05em);
}
```

#### 9.2 视口单位

```css
/* 视口单位 */
.vw-text {
  font-size: 3vw;     /* 视口宽度的 3% */
}

.vh-text {
  font-size: 2vh;     /* 视口高度的 2% */
}

.vmin-text {
  font-size: 2vmin;   /* 视口较小边的 2% */
}

/* 实际应用 — 混合固定值与视口值 */
.fluid-text {
  /* 16px + 视口宽度的 0.5%，确保最小可读 */
  font-size: calc(16px + 0.5vw);
}

/* 更精确的流式计算 — Sass 风格的响应式字号 */
:root {
  --min-size: 1rem;
  --max-size: 2rem;
  --min-width: 320;
  --max-width: 1200;
}

/* 原理：font-size = min + (max - min) * (viewport - min-vp) / (max-vp - min-vp) */
.fluid {
  font-size: clamp(
    var(--min-size),
    calc(var(--min-size) + (var(--max-size) - var(--min-size)) *
      ((100vw - var(--min-width) * 1px) / (var(--max-width) - var(--min-width))),
    var(--max-size)
  );
}
```

#### 9.3 容器查询排版

```css
/* 容器查询实现排版自适应 — 组件根据容器宽度调整字号 */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* 默认排版 */
.card-title {
  font-size: 1rem;
  line-height: 1.4;
}

.card-body {
  font-size: 0.875rem;
  line-height: 1.5;
}

/* 容器宽度 > 400px */
@container card (min-width: 400px) {
  .card-title {
    font-size: 1.5rem;
    line-height: 1.3;
  }

  .card-body {
    font-size: 1rem;
    line-height: 1.6;
  }
}

/* 容器宽度 > 600px */
@container card (min-width: 600px) {
  .card-title {
    font-size: 2rem;
    line-height: 1.2;
  }

  .card-body {
    font-size: 1.125rem;
    line-height: 1.7;
  }
}
```

#### 9.4 排版的响应式断点建议

```css
/* 排版断点系统 */
:root {
  /* 移动端 (< 640px) */
  --font-size-base: 1rem;
  --line-height-base: 1.6;
  --spacing-paragraph: 1em;

  /* 平板 (>= 640px) */
  --font-size-heading: 1.5rem;
}

@media (min-width: 640px) {
  :root {
    --font-size-base: 1.0625rem; /* 17px */
    --line-height-base: 1.65;
    --spacing-paragraph: 1.2em;
  }
}

@media (min-width: 1024px) {
  :root {
    --font-size-base: 1.125rem; /* 18px */
    --line-height-base: 1.7;
    --spacing-paragraph: 1.4em;
  }
}

body {
  font-size: var(--font-size-base);
  line-height: var(--line-height-base);
}
```

---

### 10. 排版最佳实践

#### 10.1 字体栈设计

```css
/* 完善的字体栈 */
:root {
  /* 无衬线 — 日常界面 */
  --font-sans: "Inter",
               "PingFang SC",
               "Microsoft YaHei",
               "Noto Sans SC",
               system-ui,
               -apple-system,
               sans-serif;

  /* 衬线 — 长文阅读 */
  --font-serif: "Noto Serif SC",
                "Songti SC",
                "SimSun",
                Georgia,
                serif;

  /* 等宽 — 代码 */
  --font-mono: "JetBrains Mono",
               "Fira Code",
               "Cascadia Code",
               "Source Code Pro",
               "SF Mono",
               monospace;
}
```

**字体栈设计原则**：

1. 自定义字体在前，系统字体在后
2. 覆盖三大平台（macOS / Windows / Linux）
3. 最后以通用族名结尾（sans-serif / serif / monospace）
4. 同一族内西文字体在中文字体前面（西文优先使用西文字体渲染）
5. 字体名称含空格需加引号

#### 10.2 行高与字号的黄金比例

```css
/* 基于比例尺的字号系统 — 1.25 比例 */
:root {
  --step--2: 0.64rem;   /* 10.24px */
  --step--1: 0.8rem;    /* 12.8px  */
  --step-0:  1rem;      /* 16px    — 基准 */
  --step-1:  1.25rem;   /* 20px    */
  --step-2:  1.563rem;  /* 25px    */
  --step-3:  1.953rem;  /* 31.25px */
  --step-4:  2.441rem;  /* 39.06px */
  --step-5:  3.052rem;  /* 48.83px */
}

/* 行高随字号递减 — 大字号用紧凑行高 */
:root {
  --line-height-5: 1.1;
  --line-height-4: 1.15;
  --line-height-3: 1.2;
  --line-height-2: 1.25;
  --line-height-1: 1.3;
  --line-height-0: 1.5;  /* 正文基准 */
}

h1 { font-size: var(--step-5); line-height: var(--line-height-5); }
h2 { font-size: var(--step-4); line-height: var(--line-height-4); }
h3 { font-size: var(--step-3); line-height: var(--line-height-3); }
h4 { font-size: var(--step-2); line-height: var(--line-height-2); }
h5 { font-size: var(--step-1); line-height: var(--line-height-1); }
p  { font-size: var(--step-0); line-height: var(--line-height-0); }
```

#### 10.3 段间距

```css
/* 段间距 — 使用行高相关的间距 */
.article p {
  line-height: 1.6;
  margin-bottom: 1em;      /* 段间距 = 一个字号 */
}

/* 更好的方案 — 使用行高倍数 */
.article p + p {
  margin-top: 1.5em;       /* 段间距 = 1.5 倍字号 */
}

/* 或使用 CSS 变量统一管理 */
:root {
  --rhythm: 1.5rem;        /* 基础排版节奏单位 */
}

h1 { margin-bottom: calc(var(--rhythm) * 2); }
h2 { margin-bottom: calc(var(--rhythm) * 1.5); }
p  { margin-bottom: var(--rhythm); }
ul, ol { margin-bottom: var(--rhythm); }
```

#### 10.4 中文排版规范

```css
/* 中文排版综合规范 */
.chinese-article {
  font-family: var(--font-serif);     /* 中文长文推荐衬线体 */
  font-size: 1.0625rem;              /* 中文正文 17px 较舒适 */
  line-height: 1.8;                  /* 中文行高需比西文大 */
  text-align: justify;               /* 两端对齐 */
  text-indent: 2em;                  /* 首行缩进两字符 */
  letter-spacing: 0.02em;            /* 微调字间距 */
  hanging-punctuation: first allow-end; /* 行首标点悬挂 */
  orphans: 3;                        /* 避免孤行 */
  widows: 3;
  word-break: break-all;             /* 允许单词内断行（中文） */
  overflow-wrap: break-word;         /* 溢出换行 */
}

.chinese-article p + p {
  text-indent: 2em;
  margin-top: 0;                     /* 缩进段落间不加间距 */
}

.chinese-article h2 + p {
  text-indent: 0;                    /* 标题后首段不缩进 */
}

/* 中英文之间自动空格 — 浏览器正逐步原生支持 */
.chinese-article {
  text-spacing: auto;                /* CSS Text Level 4 */
}

/* 中文引号 */
.chinese-article q {
  quotes: "「" "」" "『" "』";       /* 中文直角引号 */
}

.chinese-article q::before { content: open-quote; }
.chinese-article q::after  { content: close-quote; }
```

---

## 常见问题

### 问题 1：字体闪烁 FOIT / FOUT

**现象**：页面加载时文字短暂不可见（FOIT）或先显示回退字体后突然跳动（FOUT）。

**解决方案**：

```css
/* 1. 使用 font-display 控制行为 */
@font-face {
  font-display: swap;     /* 正文：立即显示回退字体 */
}

/* 2. 使用 size-adjust 对齐回退字体度量 */
@font-face {
  font-family: "Fallback";
  src: local("Arial");
  size-adjust: 106%;
}

/* 3. preload 关键字体 */
/* <link rel="preload" href="font.woff2" as="font" crossorigin> */

/* 4. JS 监听字体加载完成后再应用 */
```

```js
// 方案 4：字体加载完成后添加类名
document.fonts.ready.then(() => {
  document.documentElement.classList.add("fonts-loaded");
});
```

### 问题 2：中文字体加载慢

**现象**：中文字体文件 2-8MB，加载时间数秒甚至数十秒。

**解决方案**：

1. **字体子集化**：只包含页面实际使用的字符，体积可从 5MB 降到 50-200KB
2. **unicode-range 分片**：按 Unicode 范围拆分，按需加载
3. **CDN 加速**：使用 Google Fonts 或国内 CDN
4. **系统字体优先**：`local()` 优先查找系统已安装字体
5. **Critical FOFT**：先加载关键子集，再异步加载完整字体

### 问题 3：可变字体兼容性

**现象**：旧浏览器不支持可变字体，显示为默认样式。

**解决方案**：

```css
/* 渐进增强：同时声明可变字体和静态字体 */
@font-face {
  font-family: "MyFont";
  src: url("myfont-var.woff2") format("woff2-variations"),
       url("myfont-var.woff2") format("woff2"),
       url("myfont-regular.woff2") format("woff2");
  font-weight: 100 900;
  font-display: swap;
}

/* JS 特性检测 */
if (CSS.supports("font-variation-settings", '"wght" 400')) {
  // 可变字体可用
} else {
  // 加载静态字体
}
```

### 问题 4：跨平台字体渲染差异

**现象**：同一字体在 macOS（亚像素渲染）和 Windows（灰度渲染）上粗细和清晰度差异明显。

**解决方案**：

```css
/* 1. 使用 -webkit-font-smoothing 控制渲染 */
body {
  -webkit-font-smoothing: antialiased;     /* 灰度抗锯齿，macOS 更细 */
  -moz-osx-font-smoothing: grayscale;      /* Firefox 灰度渲染 */
}

/* 2. 针对平台微调 */
@supports (-webkit-font-smoothing: antialiased) {
  body { font-weight: 400; }  /* macOS 字体偏粗，用细字重 */
}

/* 3. Windows 上适当加粗 */
@media (-ms-high-contrast: active), (-ms-high-contrast: none) {
  body { font-weight: 400; }
}

/* 4. 选择跨平台渲染差异小的字体 — Inter / Roboto / Noto Sans SC */
```

---

## 面试题

### 1. @font-face 的 font-display 各值有什么区别？如何选择？

**答**：`font-display` 控制字体加载期间的文字渲染策略，有五个值：

- `auto`：浏览器默认行为，多数表现为 FOIT
- `block`：最长 3s 阻塞期（文字不可见），之后用回退字体，加载完后切换。适用于**图标字体**，因为回退字体会显示方块
- `swap`：无阻塞期，立即显示回退字体，加载完后切换。适用于**品牌展示字体**，确保内容立即可读
- `fallback`：约 100ms 阻塞期后显示回退字体，约 3s 交换期。适用于**正文内容**，平衡可读性与视觉一致性
- `optional`：约 100ms 阻塞期后显示回退字体，无交换期（字体加载完也不切换，除非已缓存）。适用于**性能敏感场景**，弱网下不等待

选择建议：图标用 `block`、品牌用 `swap`、正文用 `fallback`、性能敏感用 `optional`。

---

### 2. FOIT 和 FOUT 是什么？如何解决？

**答**：

- **FOIT**（Flash of Invisible Text）：字体加载期间文字不可见，加载完成后突然出现。大多数浏览器默认超时约 3s 后显示回退字体
- **FOUT**（Flash of Unstyled Text）：字体加载期间先显示回退字体，加载完成后文字突然跳动切换为自定义字体

FOIT 的危害是页面短暂空白，用户看不到内容；FOUT 的危害是布局偏移（CLS），影响用户体验和 Core Web Vitals 评分。

解决方案：(1) 使用 `font-display: swap` 避免 FOIT；(2) 使用 `size-adjust` + `ascent-override` 等属性对齐回退字体度量，减少 FOUT 时的布局偏移；(3) 使用 `<link rel="preload">` 提前加载关键字体；(4) 使用 Font Face API 监听加载完成后再应用字体类名。

---

### 3. 可变字体相比传统字体有什么优势？

**答**：四大核心优势：

1. **减少请求**：一个文件包含所有字重/宽度变体，替代多个独立文件，从 N 个 HTTP 请求降为 1 个
2. **文件更小**：单文件略大于一个静态字体，但远小于多个静态字体总和。例如 Inter 可变字体 300KB vs 四个静态字体 800KB
3. **设计灵活性**：可在任意字重/宽度间连续插值，不局限于几个固定档位。设计稿要求 350 字重，传统字体无法实现，可变字体直接设 `"wght" 350`
4. **动画支持**：轴值可平滑过渡，实现字重/宽度动画效果，传统字体无法做到

此外，可变字体还支持注册轴（wght/wdth/ital/slnt/opsz）和自定义轴，提供更精细的排版控制。

---

### 4. 中文 Web 字体有哪些优化策略？

**答**：中文字体核心问题是体量大（2-8MB），优化策略分五层：

1. **字体子集化**：使用 fontmin / pyftsubset 提取页面实际使用的字符，体积可从 5MB 降到 50-200KB
2. **unicode-range 分片**：按 Unicode 范围拆分字体文件（基础拉丁、常用汉字、扩展汉字），浏览器只下载页面用到的分片
3. **CDN 动态加载**：使用 Google Fonts 或有字库等 CDN，利用边缘缓存加速
4. **Critical FOFT 策略**：先加载仅含首屏字符的小子集，浏览器空闲时再异步加载完整字符集
5. **系统字体回退**：通过 `local()` 优先使用系统已安装字体（苹方/微软雅黑），减少下载需求；配合 `unicode-range` 对中英文分别使用不同字体

---

### 5. unicode-range 的作用是什么？如何使用？

**答**：`unicode-range` 是 `@font-face` 的一个属性，用于声明该字体文件覆盖的 Unicode 字符范围。浏览器仅在页面包含该范围内的字符时才会下载对应的字体文件。

使用场景：

```css
/* 英文字体 — 仅在页面包含拉丁字符时加载 */
@font-face {
  font-family: "MyFont";
  src: url("latin.woff2") format("woff2");
  unicode-range: U+0020-007E;
}

/* 中文字体 — 仅在页面包含汉字时加载 */
@font-face {
  font-family: "MyFont";
  src: url("cjk.woff2") format("woff2");
  unicode-range: U+4E00-9FFF;
}
```

核心价值：(1) 按需加载，减少不必要的网络请求；(2) 同一 font-family 可按 Unicode 范围拆分为多个文件，实现对中英文、不同语言使用不同字体文件；(3) 配合字体子集化，是中文 Web 字体优化的关键手段。

---

### 6. size-adjust 的用途是什么？

**答**：`size-adjust` 是 CSS Fonts Level 5 新增的 `@font-face` 描述符，用于等比缩放回退字体的视觉尺寸和度量，使其与 Web 字体视觉对齐，减少 FOUT 时的布局偏移（CLS）。

当 `font-display: swap` 导致先显示回退字体、后切换为 Web 字体时，两种字体的 x-height、ascender、descender 等度量不同，切换时会产生布局跳动。`size-adjust` 通过调整回退字体的全局缩放比例，使其行高、字宽等视觉参数与 Web 字体尽可能一致。

配合 `ascent-override`、`descent-override`、`line-gap-override` 可以进一步精细对齐单个度量值。这组属性是优化 CLS 指标的关键工具。

---

### 7. 如何实现中文垂直排版？需要注意什么？

**答**：使用 `writing-mode: vertical-rl` 实现传统中文竖排：

```css
.vertical-text {
  writing-mode: vertical-rl;
  text-orientation: mixed;
}
```

需要注意的问题：

1. **`text-orientation`**：`mixed` 让汉字直立、字母旋转；`upright` 让所有字符直立；`sideways` 让所有字符旋转 90 度
2. **数字处理**：竖排中的年份、日期等需用 `text-combine-upright: all` 将多位数字压缩为一字宽
3. **布局方向反转**：竖排中 `margin-left` 变为上一列的间距，`width` 和 `height` 的含义互换
4. **滚动方向**：竖排内容需要横向滚动（`overflow-x: auto`），而非纵向
5. **Flex/Grid 适配**：竖排中 `flex-direction: row` 变为纵向排列，需注意主轴方向变化
6. **浏览器差异**：各浏览器对 `hanging-punctuation`、`text-spacing` 在竖排下的支持程度不一

---

### 8. 字体加载性能优化有哪些方案？

**答**：从请求、加载、渲染三阶段优化：

**请求阶段**：
- `<link rel="preload">` 预加载关键字体，提前发起请求
- `unicode-range` 子集化，仅下载页面用到的字符范围
- 优先提供 WOFF2 格式，压缩率最高

**加载阶段**：
- `font-display` 控制加载行为，避免 FOIT
- Font Face API 的 `document.fonts.load()` 精细控制加载时机
- Critical FOFT：先加载小子集，空闲时加载完整字体
- `requestIdleCallback` 延迟加载非关键字体

**渲染阶段**：
- `size-adjust` + `ascent-override` 对齐回退字体度量，减少 CLS
- JS 监听 `document.fonts.ready` 后切换字体类名
- 可变字体替代多静态字体，减少总请求数
- CDN 缓存 + `Cache-Control` 长缓存策略

---

## 相关链接

- [[CSS新特性与现代布局]]
- [[CSS变量与主题系统]]
- [[无障碍访问A11y]]
- [MDN — @font-face](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face)
- [MDN — font-display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face/font-display)
- [MDN — font-variation-settings](https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-variation-settings)
- [MDN — writing-mode](https://developer.mozilla.org/zh-CN/docs/Web/CSS/writing-mode)
- [MDN — FontFace API](https://developer.mozilla.org/zh-CN/docs/Web/API/FontFace)
- [W3C — CSS Fonts Level 5](https://www.w3.org/TR/css-fonts-5/)
- [W3C — CSS Text Level 4](https://www.w3.org/TR/css-text-4/)
- [Google Fonts](https://fonts.google.com/)
- [Variable Fonts — MDN 指南](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_fonts/Variable_fonts_guide)
- [Web 字体性能优化 — web.dev](https://web.dev/articles/font-best-practices)
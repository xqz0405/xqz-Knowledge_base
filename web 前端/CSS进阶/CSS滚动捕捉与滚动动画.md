---
tags:
  - Web前端
  - CSS
  - 滚动捕捉
  - 滚动动画
  - Scroll-driven
  - 动画
date: 2026-05-12
status: 已完成
difficulty: 中高
---

# CSS滚动捕捉与滚动动画

## What — 是什么

> CSS 滚动捕捉（Scroll Snap）让滚动容器在滚动停止时自动吸附到指定位置，实现全屏轮播、分页滚动等效果。CSS 滚动动画（Scroll-driven Animations）让动画进度与滚动位置绑定，无需 JS 即可实现视差滚动、滚动揭示、进度条等效果。两者结合，纯 CSS 即可打造丰富的滚动交互体验。

**核心概念：**

**滚动捕捉（Scroll Snap）：**

- **Scroll Snap Container**：通过 `scroll-snap-type` 声明为捕捉容器，定义捕捉方向和严格度
- **Scroll Snap Area**：子元素通过 `scroll-snap-align` 声明自身对齐方式（start/center/end）
- **Scroll Snap Stop**：通过 `scroll-snap-stop: always` 阻止跳过捕捉点
- **Scroll Padding/Margin**：容器/子元素的内边距补偿，处理固定导航栏遮挡

**滚动动画（Scroll-driven Animations）：**

- **Scroll Timeline**：`animation-timeline: scroll()` 将动画进度与滚动位置绑定
- **View Timeline**：`animation-timeline: view()` 将动画进度与元素进入视口的过程绑定
- **Animation Range**：`animation-range` 指定动画在滚动中的起止范围
- **Timeline Scope**：`timeline-scope` 让嵌套元素访问祖先的滚动时间线

**浏览器支持：**

| 特性 | Chrome | Firefox | Safari |
|------|--------|---------|--------|
| Scroll Snap | 69+ | 68+ | 14.1+ |
| Scroll-driven Animations | 115+ | 部分支持 (126+) | 部分支持 (18+) |
| View Timeline | 115+ | 实验性 | 实验性 |

## Why — 为什么

**Scroll Snap 适用场景：**

- 全屏轮播/引导页（Snap mandatory + horizontal）
- 图片画廊/卡片横向滚动（Snap proximity）
- 分页内容（如阅读器、步骤引导）
- 日期选择器/时间轴滚动

**Scroll-driven Animations 适用场景：**

- 视差滚动效果（背景层速度不同）
- 滚动揭示动画（元素滚入视口时淡入/滑入）
- 页面阅读进度条
- 固定导航栏滚动变色
- 元素随滚动旋转/缩放

**对比传统 JS 方案：**

| 维度 | CSS Scroll-driven | JS scroll 监听 |
|------|-------------------|----------------|
| 性能 | 合成器线程运行，不掉帧 | 主线程运行，易卡顿 |
| 代码量 | 纯 CSS，~10行 | IntersectionObserver + scroll 事件，~50行 |
| 兼容性 | Chrome 115+（2023） | 全部浏览器 |
| 灵活性 | 有限（CSS 声明式） | 完全自由 |
| 调试 | DevTools Animations 面板 | console.log |

## How — 怎么用

### Scroll Snap 基础

**水平轮播：**

```css
.snap-carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  gap: 16px;
  padding: 16px;

  /* 隐藏滚动条 */
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}

.snap-carousel__item {
  flex: 0 0 80%;          /* 每项占 80% 宽度 */
  scroll-snap-align: center;
  border-radius: 12px;
  background: #f0f0f0;
  aspect-ratio: 16 / 9;
}
```

```html
<div class="snap-carousel">
  <div class="snap-carousel__item">Slide 1</div>
  <div class="snap-carousel__item">Slide 2</div>
  <div class="snap-carousel__item">Slide 3</div>
</div>
```

**全屏垂直分页：**

```css
.snap-pages {
  height: 100vh;
  overflow-y: auto;
  scroll-snap-type: y mandatory;
}

.snap-pages__section {
  height: 100vh;
  scroll-snap-align: start;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
}

/* 每个分页不同背景色 */
.snap-pages__section:nth-child(1) { background: #ff6b6b; color: #fff; }
.snap-pages__section:nth-child(2) { background: #4ecdc4; color: #fff; }
.snap-pages__section:nth-child(3) { background: #45b7d1; color: #fff; }
.snap-pages__section:nth-child(4) { background: #96ceb4; color: #fff; }
```

### scroll-snap-type 详解

```css
/* 语法：scroll-snap-type: [方向] [严格度] */

/* 方向 */
scroll-snap-type: x mandatory;     /* 水平捕捉，必须吸附 */
scroll-snap-type: y mandatory;     /* 垂直捕捉，必须吸附 */
scroll-snap-type: both mandatory;  /* 双向捕捉（网格场景） */
scroll-snap-type: inline mandatory; /* 行内方向（考虑书写模式） */
scroll-snap-type: block mandatory;  /* 块方向（考虑书写模式） */

/* 严格度 */
scroll-snap-type: x mandatory;     /* mandatory：必须吸附到最近的捕捉点 */
scroll-snap-type: x proximity;     /* proximity：接近时吸附，可停在非捕捉点 */
```

**mandatory vs proximity：**

| 严格度 | 行为 | 适用场景 |
|--------|------|---------|
| `mandatory` | 滚动结束后必须吸附到最近的捕捉点 | 全屏分页、引导页、日期选择 |
| `proximity` | 接近捕捉点时吸附，远离时不强制 | 卡片列表、图片画廊、随意浏览 |

```css
/* proximity 示例：卡片列表可自由滚动，靠近时吸附 */
.card-list {
  overflow-x: auto;
  scroll-snap-type: x proximity;
  display: flex;
  gap: 16px;
  padding: 16px;
}

.card-list__item {
  flex: 0 0 280px;
  scroll-snap-align: center;
}
```

### scroll-snap-align 详解

```css
/* 子元素对齐方式 */
.scroll-snap-align: start;    /* 吸附到容器起始边 */
.scroll-snap-align: center;   /* 吸附到容器中心 */
.scroll-snap-align: end;      /* 吸附到容器末尾 */

/* 不同对齐方式效果 */
.snap-container {
  scroll-snap-type: x mandatory;
  display: flex;
  overflow-x: auto;
}

/* start 对齐：元素左边缘对齐容器左边缘 */
.item--start {
  scroll-snap-align: start;
  flex: 0 0 75%;
}

/* center 对齐：元素居中显示 */
.item--center {
  scroll-snap-align: center;
  flex: 0 0 75%;
}

/* end 对齐：元素右边缘对齐容器右边缘 */
.item--end {
  scroll-snap-align: end;
  flex: 0 0 75%;
}
```

### scroll-snap-stop 阻止跳过

```css
/* always：不允许跳过此捕捉点 */
/* normal：允许快速滚动跳过 */

/* 适用于：必须逐步查看的内容（如条款页、步骤引导） */
.step-item {
  scroll-snap-align: start;
  scroll-snap-stop: always;  /* 必须停在此页，不能跳过 */
}

/* 适用于：可快速浏览的内容（如图片画廊） */
.gallery-item {
  scroll-snap-align: center;
  scroll-snap-stop: normal;  /* 快速滑动可跳过 */
}
```

### scroll-padding 补偿固定导航

```css
/* 当页面有固定导航栏时，捕捉点会被遮挡 */
/* scroll-padding 补偿偏移量 */

.page-container {
  scroll-snap-type: y mandatory;
  scroll-padding-top: 64px;  /* 导航栏高度 */
}

.page-section {
  scroll-snap-align: start;
  min-height: 100vh;
  padding-top: 64px;  /* 视觉补偿 */
}

/* 子元素用 scroll-snap-margin（即 scroll-margin）补偿 */
.sticky-header-page {
  scroll-snap-type: y proximity;
}

.section-with-offset {
  scroll-snap-align: start;
  scroll-margin-top: 64px;  /* 子元素级别补偿 */
}
```

### 实战：全屏引导页

```css
.onboarding {
  height: 100vh;
  overflow-y: auto;
  scroll-snap-type: y mandatory;
  scroll-behavior: smooth;
}

.onboarding__step {
  height: 100vh;
  scroll-snap-align: start;
  scroll-snap-stop: always;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 48px 32px;
  text-align: center;
}

.onboarding__step:nth-child(1) { background: linear-gradient(135deg, #667eea, #764ba2); }
.onboarding__step:nth-child(2) { background: linear-gradient(135deg, #f093fb, #f5576c); }
.onboarding__step:nth-child(3) { background: linear-gradient(135deg, #4facfe, #00f2fe); }
.onboarding__step:nth-child(4) { background: linear-gradient(135deg, #43e97b, #38f9d7); }

.onboarding__icon {
  font-size: 4rem;
  margin-bottom: 24px;
}

.onboarding__title {
  font-size: 1.75rem;
  font-weight: 700;
  color: #fff;
  margin-bottom: 16px;
}

.onboarding__desc {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.9);
  max-width: 320px;
  line-height: 1.6;
}

/* 指示器 */
.onboarding__dots {
  position: fixed;
  bottom: 48px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  z-index: 10;
}

.onboarding__dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  transition: all 0.3s;
}

.onboarding__dot.active {
  width: 24px;
  border-radius: 4px;
  background: #fff;
}
```

### 实战：图片画廊

```css
.gallery {
  position: relative;
  overflow: hidden;
}

.gallery__track {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}

.gallery__item {
  flex: 0 0 100%;
  scroll-snap-align: center;
  position: relative;
}

.gallery__item img {
  width: 100%;
  height: 400px;
  object-fit: cover;
}

.gallery__item-caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 24px 16px 16px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  color: #fff;
  font-size: 0.875rem;
}

/* 左右箭头 */
.gallery__nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.gallery__nav--prev { left: 12px; }
.gallery__nav--next { right: 12px; }

/* 计数器 */
.gallery__counter {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 4px 12px;
  border-radius: 20px;
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  font-size: 0.75rem;
  z-index: 2;
}
```

```ts
// 箭头导航逻辑
function scrollGallery(direction: 'prev' | 'next') {
  const track = document.querySelector('.gallery__track')!;
  const itemWidth = track.clientWidth;
  const scrollAmount = direction === 'next' ? itemWidth : -itemWidth;
  track.scrollBy({ left: scrollAmount, behavior: 'smooth' });
}
```

### Scroll-driven Animations 基础

**滚动进度时间线：**

```css
/* 页面滚动进度条 */
.progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: #1677ff;
  transform-origin: left;
  transform: scaleX(0);
  animation: progress linear;
  animation-timeline: scroll();
}

@keyframes progress {
  to { transform: scaleX(1); }
}
```

**指定滚动容器：**

```css
/* 容器内滚动驱动动画 */
.scroll-container {
  overflow-y: auto;
  timeline-scope: --my-scroll;
}

.scroll-container {
  animation-timeline: --my-scroll;
}

/* 或者使用 scroll() 指定容器 */
.element {
  animation: fadeIn linear;
  animation-timeline: scroll(nearest block);
  /* scroll(nearest) — 最近的滚动祖先 */
  /* scroll(root) — 文档滚动 */
  /* scroll(self) — 元素自身滚动 */
}
```

### View Timeline — 元素进入视口动画

```css
/* 元素进入视口时淡入上移 */
.reveal {
  opacity: 0;
  transform: translateY(40px);
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
  /* entry — 元素开始进入视口
     exit — 元素开始离开视口
     cover — 元素完全在视口内
     contain — 元素完全进入视口到完全离开 */
}

@keyframes reveal {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

**animation-range 详解：**

```css
/* 动画范围：控制动画在滚动中的哪个阶段播放 */
/* 语法：animation-range: [范围名] [偏移] [范围名] [偏移] */

/* 元素刚进入视口时播放动画 */
.fade-in {
  animation: fadeIn linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

/* 元素完全在视口内时播放 */
.highlight {
  animation: highlight linear both;
  animation-timeline: view();
  animation-range: contain 0% contain 100%;
}

/* 元素离开视口时播放 */
.fade-out {
  animation: fadeOut linear both;
  animation-timeline: view();
  animation-range: exit 0% exit 100%;
}

/* 完整范围：从进入视口到离开视口 */
.full-range {
  animation: scaleIn linear both;
  animation-timeline: view();
  animation-range: entry 0% exit 100%;
}

/* 偏移：提前/延后触发 */
.early-trigger {
  animation: slideIn linear both;
  animation-timeline: view();
  animation-range: entry -10% entry 80%; /* 视口上方10%处开始 */
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes fadeOut { from { opacity: 1; } to { opacity: 0; } }
@keyframes highlight { from { background: transparent; } to { background: #ff0; } }
@keyframes scaleIn { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
@keyframes slideIn { from { transform: translateX(-100px); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
```

### 实战：视差滚动

```css
.parallax-container {
  height: 100vh;
  overflow-y: auto;
}

/* 背景层：慢速移动 */
.parallax-bg {
  position: fixed;
  inset: 0;
  background: url('mountains.jpg') center / cover;
  animation: parallax-slow linear;
  animation-timeline: scroll();
}

@keyframes parallax-slow {
  from { transform: translateY(0); }
  to { transform: translateY(-200px); }
}

/* 中景层：中速移动 */
.parallax-mid {
  position: relative;
  animation: parallax-mid linear;
  animation-timeline: scroll();
}

@keyframes parallax-mid {
  from { transform: translateY(0); }
  to { transform: translateY(-100px); }
}

/* 前景层：正常滚动（不设置动画） */
.parallax-fg {
  position: relative;
}
```

### 实战：滚动揭示动画组

```css
/* 不同方向的揭示动画 */
.reveal-up {
  opacity: 0;
  transform: translateY(60px);
  animation: reveal-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}

@keyframes reveal-up {
  to { opacity: 1; transform: translateY(0); }
}

.reveal-left {
  opacity: 0;
  transform: translateX(-60px);
  animation: reveal-left linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}

@keyframes reveal-left {
  to { opacity: 1; transform: translateX(0); }
}

.reveal-right {
  opacity: 0;
  transform: translateX(60px);
  animation: reveal-right linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}

@keyframes reveal-right {
  to { opacity: 1; transform: translateX(0); }
}

.reveal-scale {
  opacity: 0;
  transform: scale(0.8);
  animation: reveal-scale linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}

@keyframes reveal-scale {
  to { opacity: 1; transform: scale(1); }
}

.reveal-rotate {
  opacity: 0;
  transform: rotate(-10deg);
  animation: reveal-rotate linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}

@keyframes reveal-rotate {
  to { opacity: 1; transform: rotate(0); }
}
```

```html
<section class="content">
  <h2 class="reveal-up">标题从下方滑入</h2>
  <div class="grid">
    <div class="card reveal-left">卡片从左侧滑入</div>
    <div class="card reveal-right">卡片从右侧滑入</div>
    <div class="card reveal-scale">卡片缩放出现</div>
  </div>
  <p class="reveal-up">段落从下方淡入</p>
</section>
```

### 实战：导航栏滚动变色

```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  padding: 16px 24px;
  z-index: 100;
  transition: background 0.3s, padding 0.3s;

  /* 滚动驱动动画 */
  animation: navbar-scroll linear both;
  animation-timeline: scroll();
  animation-range: 0px 100px; /* 滚动0-100px范围内变化 */
}

@keyframes navbar-scroll {
  from {
    background: transparent;
    padding: 20px 24px;
    box-shadow: none;
  }
  to {
    background: rgba(255, 255, 255, 0.95);
    padding: 12px 24px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(10px);
  }
}

.navbar__logo {
  font-size: 1.25rem;
  font-weight: 700;
  animation: logo-scroll linear both;
  animation-timeline: scroll();
  animation-range: 0px 100px;
}

@keyframes logo-scroll {
  from { color: #fff; }
  to { color: #333; }
}
```

### 实战：水平滚动时间轴

```css
.timeline {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  gap: 0;
  padding: 48px 24px;
  position: relative;
}

/* 连接线 */
.timeline::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 2px;
  background: #e0e0e0;
}

.timeline__item {
  flex: 0 0 300px;
  scroll-snap-align: center;
  position: relative;
  padding: 0 24px;
}

.timeline__dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #1677ff;
  border: 3px solid #fff;
  box-shadow: 0 0 0 2px #1677ff;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
}

.timeline__content {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  margin-bottom: 60px;
}

.timeline__year {
  font-size: 0.75rem;
  color: #1677ff;
  font-weight: 600;
}

.timeline__title {
  font-size: 1rem;
  font-weight: 600;
  margin-top: 8px;
}

.timeline__desc {
  font-size: 0.875rem;
  color: #666;
  margin-top: 8px;
  line-height: 1.5;
}
```

### Scroll Snap + Scroll-driven Animation 组合

```css
/* 全屏展示页：Snap 分页 + 每页滚动揭示 */
.showcase {
  height: 100vh;
  overflow-y: auto;
  scroll-snap-type: y mandatory;
}

.showcase__page {
  height: 100vh;
  scroll-snap-align: start;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 48px;
}

.showcase__text {
  animation: text-reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 60%;
}

.showcase__image {
  animation: image-reveal linear both;
  animation-timeline: view();
  animation-range: entry 10% entry 70%;
}

@keyframes text-reveal {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes image-reveal {
  from { opacity: 0; transform: scale(0.9); }
  to { opacity: 1; transform: scale(1); }
}
```

### JS 交互增强

```ts
// Scroll Snap 导航：点击指示器跳转
function setupSnapNavigation(containerSelector: string, dotSelector: string) {
  const container = document.querySelector(containerSelector)!;
  const items = container.children;
  const dots = document.querySelectorAll(dotSelector);

  // 点击指示器跳转
  dots.forEach((dot, index) => {
    dot.addEventListener('click', () => {
      items[index].scrollIntoView({ behavior: 'smooth' });
    });
  });

  // 滚动时更新指示器
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const index = Array.from(items).indexOf(entry.target);
          dots.forEach((d, i) => d.classList.toggle('active', i === index));
        }
      });
    },
    { root: container, threshold: 0.5 }
  );

  Array.from(items).forEach((item) => observer.observe(item));
}

// Scroll-driven Animation 降级：旧浏览器用 IntersectionObserver
function progressiveReveal() {
  // 检测是否支持 scroll-driven animations
  if (CSS.supports('animation-timeline', 'scroll()')) return;

  // 降级方案
  const reveals = document.querySelectorAll('.reveal-up, .reveal-left, .reveal-right');
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          entry.target.classList.add('revealed');
          observer.unobserve(entry.target);
        }
      });
    },
    { threshold: 0.1, rootMargin: '0px 0px -50px 0px' }
  );

  reveals.forEach((el) => observer.observe(el));
}

// 降级 CSS
// .reveal-up.revealed { opacity: 1; transform: translateY(0); transition: all 0.6s ease; }
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Snap 不生效 | 容器未设 `overflow: auto/scroll` | 必须设置 overflow 才能滚动+捕捉 |
| 捕捉点位置偏移 | 固定导航遮挡 | 使用 `scroll-padding-top` 补偿 |
| 快速滑动跳过多个捕捉点 | `scroll-snap-stop: normal` | 设置 `scroll-snap-stop: always` |
| iOS 弹性滚动干扰 | iOS rubber-band 效果 | CSS `overscroll-behavior: contain` |
| `animation-timeline: scroll()` 不生效 | 浏览器不支持 | Chrome 115+，旧浏览器用 IntersectionObserver |
| View Timeline 动画播放一次就停 | `animation-fill-mode` 未设置 | 使用 `animation: xxx linear both` |
| 容器内 scroll() 时间线不触发 | 未设置 `timeline-scope` | 容器上添加 `timeline-scope: --name` |
| 横向 Snap 在 Firefox 行为不同 | Firefox 弹性滚动实现差异 | 测试兼容性，必要时用 JS 控制 |
| 嵌套滚动容器冲突 | 内外容器都设置了 snap | 只在最内层滚动容器设置 snap |
| 动画卡顿 | `animation-timeline` 配合了重排属性 | 只动画 `transform`/`opacity`，避免 `width`/`height` |

### 最佳实践

- 全屏分页用 `mandatory`，自由浏览用 `proximity`
- 有固定导航栏时必须设置 `scroll-padding-top`
- 必须逐步查看的内容用 `scroll-snap-stop: always`
- Scroll-driven Animations 只动画 `transform` 和 `opacity`，避免重排
- `animation-range: entry 0% entry 80%` 提前触发，体验更自然
- 旧浏览器用 IntersectionObserver 渐进增强
- Snap 容器嵌套时只设一层 snap，避免冲突
- 使用 `overscroll-behavior: contain` 防止滚动穿透
- 移动端优先测试 iOS Safari，其 Snap 行为有差异
- Scroll-driven Animations 配合 `will-change` 提示浏览器优化合成

## 面试题

**Q1: scroll-snap-type 的 mandatory 和 proximity 有什么区别？怎么选择？**
> `mandatory`：滚动结束后必须吸附到最近的捕捉点，不会停在两个捕捉点之间。`proximity`：接近捕捉点时吸附，距离远则不强制。选择：必须逐页查看的内容（引导页、全屏展示、日期选择器）用 `mandatory`；可自由浏览的内容（卡片列表、图片画廊）用 `proximity`。`mandatory` 的风险是如果捕捉点间距不合理，用户会感觉被"强制拉回"，`proximity` 更自然但不够精确。

**Q2: 如何解决固定导航栏遮挡 Scroll Snap 捕捉点的问题？**
> 两种方式：① 容器级 `scroll-padding-top: 64px`（推荐），偏移所有捕捉点，使其不被导航栏遮挡；② 子元素级 `scroll-margin-top: 64px`，只对特定子元素偏移。两者的区别：`scroll-padding` 作用于容器的所有子元素，`scroll-margin` 只作用于声明的子元素。通常用 `scroll-padding` 更简单，特殊子元素再用 `scroll-margin` 覆盖。

**Q3: CSS Scroll-driven Animations 和 JS scroll 事件监听有什么性能差异？**
> CSS Scroll-driven Animations 运行在浏览器合成器线程，不阻塞主线程，动画帧率稳定在 60fps；JS scroll 事件运行在主线程，高频触发时会导致布局抖动（layout thrashing），滚动时可能掉帧。原理：CSS 动画只需声明 `animation-timeline: scroll()`，浏览器在合成阶段直接根据滚动偏移计算动画进度，不经过 JS 引擎。但 CSS 方案灵活性有限，复杂交互逻辑仍需 JS。

**Q4: animation-range 的 entry/exit/cover/contain 分别是什么意思？**
> 以 View Timeline 为例：`entry` = 元素开始进入视口（底部刚出现）→ 完全进入视口；`exit` = 元素开始离开视口 → 完全离开视口（顶部消失）；`cover` = entry 开始 → exit 结束，覆盖元素从进入到离开的全过程；`contain` = 元素完全在视口内（entry 结束 → exit 开始）。典型用法：滚动揭示用 `entry 0% entry 100%`（进入时动画），滚动高亮用 `contain 0% contain 100%`（完全可见时动画）。

**Q5: 如何实现纯 CSS 的视差滚动效果？**
> 使用 `animation-timeline: scroll()` 让不同层以不同速度移动：背景层 `animation: slow-scroll linear; animation-timeline: scroll()`，位移量较大（如 `translateY(-200px)`）；中景层位移量中等（`translateY(-100px)`）；前景层不设动画（正常滚动速度）。原理：`scroll()` 时间线的进度 0-1 对应页面从顶部到底部，每层的 `@keyframes` 中设置不同的 `translateY` 偏移量，偏移量越大滚动越快，形成视差。

**Q6: Scroll Snap 在移动端有什么兼容性问题？**
> ① iOS Safari 弹性滚动：iOS 的 rubber-band 效果可能干扰 snap，用 `overscroll-behavior: contain` 禁止；② iOS momentum scrolling：快速滑动时 iOS 可能跳过多个 snap 点，用 `scroll-snap-stop: always` 阻止；③ Android Chrome 嵌套滚动：外层 snap 容器可能拦截内层滚动事件，需仔细设计 DOM 层级；④ 触摸延迟：某些低端设备 snap 回弹有延迟，用 `scroll-behavior: smooth` 优化；⑤ Firefox 移动端横向 snap 行为与 Chrome 不一致，需真机测试。

**Q7: View Timeline 和 IntersectionObserver 在滚动揭示场景下有什么区别？**
> IntersectionObserver：JS API，元素进入/离开视口时触发回调，通常配合 class 切换 + CSS transition 实现动画，动画只播放一次（unobserve 后不再触发）。View Timeline：CSS 原生，动画进度与元素在视口中的位置线性映射，滚动回去动画会反向播放。选择：一次性揭示（进入视口后不再变化）用 IntersectionObserver + transition 更简单；持续映射（元素位置与动画状态始终关联，如进度条、视差）用 View Timeline。

**Q8: Scroll Snap 和 Scroll-driven Animation 如何配合使用？**
> Snap 处理滚动停止位置，Scroll-driven Animation 处理滚动过程中的视觉效果。典型配合：全屏展示页用 Snap 分页（每页吸附到顶部），每页内容用 View Timeline 在进入视口时播放揭示动画。注意：Snap 的 `mandatory` 模式下，动画应在 `entry` 范围内完成（元素进入视口的过程中），避免在吸附静止状态下播放不完整的动画。`proximity` 模式下更灵活，因为用户可能停在任意位置。

---

**相关链接：**
- [[CSS动画与过渡]]
- [[CSS新特性与现代布局]]
- [[View Transitions API]]
- [[Intersection Observer与懒加载]]
- [[CSS Houdini与自定义绘制]]

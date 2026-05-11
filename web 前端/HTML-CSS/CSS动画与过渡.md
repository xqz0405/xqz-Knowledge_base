---
tags:
  - Web前端
  - CSS
  - 动画
  - 过渡
  - 性能
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS动画与过渡

## What — 是什么

> CSS Transition 用于状态间的平滑过渡，CSS Animation 用于定义关键帧动画。两者都是声明式、GPU 加速的高性能动画方案。

**核心概念：**

- **transition**：`transition: property duration timing-function delay`
- **animation**：`animation: name duration timing-function delay iteration-count direction fill-mode`
- **`@keyframes`**：定义动画的关键帧序列
- **timing-function**：`ease`、`linear`、`ease-in-out`、`cubic-bezier()`、`steps()`

**关键特性：**

- 只动画 `transform` 和 `opacity` 可触发 GPU 合成层，性能最佳
- `will-change` 提前告知浏览器哪些属性会变
- `animation-fill-mode: forwards` 保持动画结束状态
- `prefers-reduced-motion` 尊重用户减少动画偏好

## Why — 为什么

**适用场景：**

- 按钮悬浮/点击反馈（transition）
- 加载动画（animation + keyframes）
- 页面过渡和元素入场
- 交互微动效

**对比替代方案：**

| 维度 | CSS 动画 | JS 动画（GSAP/WAAPI） | Lottie |
|------|---------|----------------------|--------|
| 性能 | 极高（GPU 加速） | 高 | 中（Canvas/SVG） |
| 控制力 | 低（声明式） | 极高（可暂停/反转/seek） | 中 |
| 复杂度 | 简单 | 中 | 需设计导出 |
| 兼容性 | 极好 | 好 | 好 |

**优缺点：**

- ✅ 优点：
  - 声明式，代码简洁
  - GPU 加速，不阻塞主线程
  - 浏览器自动优化（跳帧处理）
- ❌ 缺点：
  - 无法精确控制（暂停/seek/动态参数）
  - 复杂动画链难以编排
  - 与 JS 状态同步困难

## How — 怎么用

### 快速上手

```css
/* 过渡：hover 效果 */
.button {
    background: #3b82f6;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.button:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.4);
}

/* 关键帧动画：加载旋转 */
@keyframes spin {
    to { transform: rotate(360deg); }
}
.spinner {
    animation: spin 1s linear infinite;
}
```

### 代码示例

**入场动画：**

```css
@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.card {
    animation: fadeInUp 0.4s ease-out forwards;
    opacity: 0; /* 初始隐藏 */
}

/* 交错延迟 */
.card:nth-child(1) { animation-delay: 0.1s; }
.card:nth-child(2) { animation-delay: 0.2s; }
.card:nth-child(3) { animation-delay: 0.3s; }
```

**性能优化：只动画 transform/opacity：**

```css
/* ❌ 触发 layout + paint */
.box {
    transition: width 0.3s, height 0.3s;
}

/* ✅ 只触发 composite，GPU 加速 */
.box {
    transition: transform 0.3s;
    /* 用 scaleX/scaleY 替代 width/height */
}
```

**尊重用户偏好：**

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}
```

**弹性效果（cubic-bezier）：**

```css
/* 弹性弹出 */
.bounce {
    transition: transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

/* 常用贝塞尔曲线 */
/* ease-out: cubic-bezier(0, 0, 0.2, 1) */
/* ease-in-out: cubic-bezier(0.4, 0, 0.2, 1) */
/* 弹性: cubic-bezier(0.34, 1.56, 0.64, 1) */
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 动画卡顿 | 动画了 layout 属性（width/height/margin） | 只动画 `transform` 和 `opacity` |
| 动画结束状态丢失 | 默认 `fill-mode: none` | 用 `animation-fill-mode: forwards` |
| height: auto 无法过渡 | `auto` 不是数值，无法插值 | 用 `max-height` hack 或 `grid-template-rows: 0fr → 1fr` |
| 移动端动画闪烁 | 层合成问题 | 加 `transform: translateZ(0)` 或 `will-change: transform` |

### 最佳实践

- 只动画 `transform` 和 `opacity`
- 用 `will-change` 提示浏览器，用完移除
- 始终添加 `prefers-reduced-motion` 降级
- 交互反馈用 transition（hover/click），循环动效用 animation

## 面试题

**Q1: transition和animation的区别是什么？**
> transition用于两个状态间的平滑过渡，需要触发条件（如hover），只能定义开始和结束状态；animation通过@keyframes定义关键帧序列，可自动播放、循环、暂停，支持多个关键帧和复杂动画路径。交互反馈用transition，循环/入场动效用animation。

**Q2: 为什么CSS动画要只用transform和opacity？**
> transform和opacity的动画只触发合成层（composite）操作，由GPU加速处理，不触发布局重排（layout）和重绘（paint），性能最佳。动画width/height/margin等属性会触发布局重排，导致主线程阻塞造成卡顿。

**Q3: 什么是GPU加速（合成层）？如何触发？**
> GPU加速是指浏览器将元素提升到独立的合成层，由GPU直接处理渲染，跳过CPU布局和绘制。触发方式：transform 3D变换（translateZ/translate3d）、will-change: transform/opacity、opacity动画、fixed定位。注意不要过度创建合成层，会增加内存占用。

**Q4: height: auto如何实现过渡动画？**
> auto不是数值无法插值，直接transition无效。方案：用max-height hack（设一个足够大的max-height值，从0过渡到该值，但时间曲线不精确）；或用grid-template-rows从0fr过渡到1fr（现代方案，动画精确）；或用JS读取实际高度后设置。

---

**相关链接：**
- [[CSS布局Flex与Grid]]
- [[CSS选择器与优先级]]

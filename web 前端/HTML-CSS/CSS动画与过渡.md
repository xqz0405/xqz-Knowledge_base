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

### transition 完整属性

`transition` 是以下四个属性的简写：

| 属性 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `transition-property` | 指定过渡的 CSS 属性 | `all` | `transform`, `opacity`, `background-color` |
| `transition-duration` | 过渡持续时间 | `0s` | `0.3s`, `200ms` |
| `transition-timing-function` | 时间函数（速度曲线） | `ease` | `linear`, `cubic-bezier(0.4,0,0.2,1)` |
| `transition-delay` | 延迟时间 | `0s` | `0.1s`, `100ms` |

```css
/* 简写 */
transition: property duration timing-function delay;

/* 示例 */
transition: transform 0.3s ease 0.1s;

/* 多属性过渡 */
transition: transform 0.3s ease, opacity 0.3s ease, box-shadow 0.2s ease-in;

/* 分写 */
transition-property: transform, opacity;
transition-duration: 0.3s;
transition-timing-function: ease;
transition-delay: 0.1s;
```

**注意：** `transition-property: all` 虽然方便，但会让浏览器对所有可动画属性做过渡准备，影响性能。建议明确指定需要过渡的属性。

### timing-function 详解

时间函数决定动画在持续时间内的速度变化曲线：

| 函数 | 曲线描述 | 典型场景 |
|------|----------|----------|
| `linear` | 匀速，无加减速 | 进度条、匀速旋转 |
| `ease` | 慢→快→慢（默认） | 通用过渡 |
| `ease-in` | 慢→快（加速） | 元素退出、下落 |
| `ease-out` | 快→慢（减速） | 元素进入、弹窗出现 |
| `ease-in-out` | 慢→快→慢（对称） | 颜色变化、淡入淡出 |
| `cubic-bezier(x1,y1,x2,y2)` | 自定义贝塞尔曲线 | 弹性效果、特殊曲线 |
| `steps(n, <jumpterm>)` | 逐帧离散跳变 | 打字效果、精灵图动画 |

```css
/* 标准缓动 */
ease:            cubic-bezier(0.25, 0.1, 0.25, 1.0)
ease-in:         cubic-bezier(0.42, 0.0, 1.0, 1.0)
ease-out:        cubic-bezier(0.0, 0.0, 0.58, 1.0)
ease-in-out:     cubic-bezier(0.42, 0.0, 0.58, 1.0)

/* Material Design 推荐曲线 */
standard:        cubic-bezier(0.2, 0.0, 0, 1.0)   /* 通用 */
decelerate:      cubic-bezier(0.0, 0.0, 0, 1.0)   /* 进入 */
accelerate:      cubic-bezier(0.3, 0.0, 1, 1.0)   /* 退出 */

/* 弹性曲线 */
spring:          cubic-bezier(0.34, 1.56, 0.64, 1) /* 轻微过冲回弹 */
bounce-in:       cubic-bezier(0.6, -0.28, 0.74, 0.05) /* 大幅过冲 */
```

**steps() 参数详解：**

```css
/* steps(number_of_steps, direction) */
/* direction: jump-start / jump-end / jump-none / jump-both */
/* jump-end（原 start）: 间隔末尾跳变，默认 */
/* jump-start（原 end）: 间隔开头跳变 */

/* 打字效果：10步，末尾跳变 */
animation: typing 1s steps(10, jump-end) forwards;

/* 精灵图：8帧，开头跳变 */
animation: sprite 0.8s steps(8, jump-start) infinite;
```

### animation 完整属性

`animation` 是以下八个属性的简写：

| 属性 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `animation-name` | @keyframes 名称 | `none` | `fadeIn`, `spin` |
| `animation-duration` | 动画持续时间 | `0s` | `0.5s`, `1s` |
| `animation-timing-function` | 时间函数 | `ease` | `linear`, `cubic-bezier(...)` |
| `animation-delay` | 延迟时间 | `0s` | `0.2s` |
| `animation-iteration-count` | 播放次数 | `1` | `3`, `infinite` |
| `animation-direction` | 播放方向 | `normal` | `reverse`, `alternate`, `alternate-reverse` |
| `animation-fill-mode` | 填充模式 | `none` | `forwards`, `backwards`, `both` |
| `animation-play-state` | 播放状态 | `running` | `paused` |

```css
/* 简写 */
animation: name duration timing-function delay iteration-count direction fill-mode play-state;

/* 示例 */
animation: fadeInUp 0.4s ease-out 0.1s 1 normal forwards running;

/* 常用简写（省略使用默认值） */
animation: spin 1s linear infinite;          /* 旋转 */
animation: pulse 2s ease-in-out alternate;   /* 来回脉冲 */
animation: slideIn 0.3s ease-out forwards;   /* 入场保持 */
```

**animation-direction 取值：**

| 值 | 说明 |
|----|------|
| `normal` | 每次从头到尾播放 |
| `reverse` | 每次从尾到头播放 |
| `alternate` | 正→反→正→反交替 |
| `alternate-reverse` | 反→正→反→正交替 |

**animation-fill-mode 取值：**

| 值 | 说明 |
|----|------|
| `none` | 默认，动画结束后回到原始样式 |
| `forwards` | 动画结束后保持最后一帧状态 |
| `backwards` | 动画延迟期间应用第一帧状态 |
| `both` | 同时应用 forwards 和 backwards 行为 |

### @keyframes 高级用法

```css
/* 基础：from/to */
@keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* 多关键帧百分比 */
@keyframes bounce {
    0%   { transform: translateY(0); }
    20%  { transform: translateY(-30px); }
    40%  { transform: translateY(0); }
    60%  { transform: translateY(-15px); }
    80%  { transform: translateY(0); }
    100% { transform: translateY(0); }
}

/* 同一关键帧多个属性 */
@keyframes complexEntry {
    0% {
        opacity: 0;
        transform: translateY(40px) scale(0.9);
        filter: blur(4px);
    }
    60% {
        opacity: 1;
        transform: translateY(-5px) scale(1.02);
        filter: blur(0);
    }
    100% {
        opacity: 1;
        transform: translateY(0) scale(1);
        filter: blur(0);
    }
}

/* 多选择器复用同一关键帧 */
@keyframes slideIn {
    from { transform: translateX(-100%); opacity: 0; }
    to   { transform: translateX(0); opacity: 1; }
}

.item-1 { animation: slideIn 0.4s ease-out 0.0s both; }
.item-2 { animation: slideIn 0.4s ease-out 0.1s both; }
.item-3 { animation: slideIn 0.4s ease-out 0.2s both; }
```

### transform 完整函数

| 函数 | 说明 | 示例 |
|------|------|------|
| `translate(x, y)` | 2D 平移 | `translate(10px, 20px)` |
| `translateX(x)` | 水平平移 | `translateX(50px)` |
| `translateY(y)` | 垂直平移 | `translateY(-10px)` |
| `translate3d(x, y, z)` | 3D 平移 | `translate3d(0, 0, 100px)` |
| `translateZ(z)` | Z 轴平移 | `translateZ(50px)` |
| `scale(x, y)` | 2D 缩放 | `scale(1.5, 1.2)` |
| `scaleX(x)` | 水平缩放 | `scaleX(2)` |
| `scaleY(y)` | 垂直缩放 | `scaleY(0.5)` |
| `scale3d(x, y, z)` | 3D 缩放 | `scale3d(1, 1, 1.5)` |
| `rotate(angle)` | 2D 旋转 | `rotate(45deg)` |
| `rotateX(angle)` | 绕 X 轴旋转 | `rotateX(60deg)` |
| `rotateY(angle)` | 绕 Y 轴旋转 | `rotateY(45deg)` |
| `rotateZ(angle)` | 绕 Z 轴旋转 | `rotateZ(90deg)` |
| `rotate3d(x, y, z, angle)` | 3D 旋转 | `rotate3d(1, 1, 0, 45deg)` |
| `skew(x-angle, y-angle)` | 2D 倾斜 | `skew(10deg, 5deg)` |
| `skewX(angle)` | 水平倾斜 | `skewX(15deg)` |
| `skewY(angle)` | 垂直倾斜 | `skewY(10deg)` |
| `matrix(a,b,c,d,e,f)` | 2D 矩阵变换 | `matrix(1,0,0,1,10,20)` |
| `matrix3d(...)` | 3D 矩阵变换（16值） | 高级用法 |
| `perspective(n)` | 设置透视距离 | `perspective(500px)` |

```css
/* transform 组合：从右往左依次应用 */
.transform-example {
    /* 先平移 → 再旋转 → 再缩放 */
    transform: translate(50px, 20px) rotate(30deg) scale(1.2);
}

/* transform-origin：变换原点 */
.rotate-center  { transform-origin: center center; transform: rotate(45deg); }  /* 默认 */
.rotate-top-left { transform-origin: top left; transform: rotate(45deg); }
.rotate-custom   { transform-origin: 30% 70%; transform: rotate(45deg); }

/* perspective：透视（在父元素上设置） */
.perspective-container {
    perspective: 800px;       /* 透视距离，值越小透视效果越强 */
    perspective-origin: 50% 50%; /* 视点位置 */
}

/* transform-style：保留 3D 空间 */
.preserve-3d {
    transform-style: preserve-3d;  /* 子元素保持 3D 变换 */
    /* flat（默认）：子元素扁平化到 2D 平面 */
}

/* backface-visibility：背面可见性 */
.card {
    backface-visibility: hidden; /* 隐藏翻转后的背面 */
}
```

### 性能优化原理：GPU 合成层

**浏览器渲染管线：**

```
JavaScript → Style → Layout → Paint → Composite
                        ↑        ↑        ↑
                     重排      重绘     合成
                   (慢)      (中)     (快/GPU)
```

| 操作 | 触发阶段 | 性能影响 |
|------|----------|----------|
| 修改 `width`/`height`/`margin` | Layout → Paint → Composite | 差（重排） |
| 修改 `color`/`background`/`box-shadow` | Paint → Composite | 中（重绘） |
| 修改 `transform`/`opacity` | Composite | 好（仅合成） |

**will-change 的作用：**

```css
/* 提前告知浏览器该属性会变化，让浏览器优化准备 */
.will-animate {
    will-change: transform, opacity;
}

/* ⚠️ 使用原则：不要滥用！ */
/* ❌ 错误：全局设置，浪费内存 */
* { will-change: transform; }

/* ✅ 正确：在需要时添加，动画结束后移除 */
/* 方式1：hover 时设置（此时浏览器有足够时间准备） */
.card { transition: transform 0.3s; }
.card:hover { will-change: transform; }

/* 方式2：JS 动态添加和移除 */
// element.style.willChange = 'transform';
// 动画结束后：element.style.willChange = 'auto';
```

**触发合成层的条件：**

- 3D 变换：`translate3d`、`translateZ`
- `will-change: transform` / `will-change: opacity`
- `opacity` 动画（值 < 1）
- `position: fixed`（部分浏览器）
- `filter`、`backdrop-filter`
- `video`、`canvas`、`iframe` 元素

**注意：** 每个合成层都会占用额外内存（约 width x height x 4 bytes），过度创建会导致内存暴涨。

### CSS 动画 vs JS 动画 vs Web Animations API

| 维度 | CSS 动画 | JS 动画（requestAnimationFrame/GSAP） | Web Animations API |
|------|---------|--------------------------------------|-------------------|
| 性能 | 极高（GPU 加速） | 高（需手动优化） | 高（CSS 动画的 JS 接口） |
| 控制力 | 低（声明式） | 极高（可暂停/反转/seek） | 高（可暂停/反转/seek） |
| 复杂度 | 简单 | 中 | 中 |
| 兼容性 | 极好 | 极好 | 好（现代浏览器） |
| 状态同步 | 难（无法读取中间状态） | 易 | 易 |
| 动态参数 | 不支持 | 支持 | 支持 |
| 编排链 | 困难 | 简单（Timeline） | 中等 |

**Web Animations API 示例：**

```javascript
// 用 JS 调用 CSS 级别的动画能力
const animation = element.animate(
    [
        { transform: 'translateY(20px)', opacity: 0 },
        { transform: 'translateY(0)', opacity: 1 }
    ],
    {
        duration: 300,
        easing: 'ease-out',
        fill: 'forwards'
    }
);

// 完整控制
animation.pause();          // 暂停
animation.play();           // 恢复
animation.reverse();        // 反转
animation.finish();         // 跳到结束
animation.currentTime = 150; // seek 到中间
animation.onfinish = () => {}; // 完成回调
```

### View Transitions API 概念

> View Transitions API 是浏览器原生的页面/元素过渡方案，可以让 DOM 变化时有平滑的视觉过渡。

```css
/* 定义过渡动画 */
::view-transition-old(root) {
    animation: fade-out 0.25s ease-out;
}
::view-transition-new(root) {
    animation: fade-in 0.25s ease-in;
}

/* 自定义元素过渡 */
::view-transition-old(card) {
    animation: slide-out 0.3s ease-in;
}
::view-transition-new(card) {
    animation: slide-in 0.3s ease-out;
}
```

```javascript
// 基本用法
document.startViewTransition(() => {
    // 在这里更新 DOM
    updateDOM();
});

// SPA 路由切换
async function navigate(url) {
    const transition = document.startViewTransition(async () => {
        await loadPage(url);
        updateContent();
    });
    await transition.finished;
}
```

**浏览器支持：** Chrome 111+、Edge 111+，Safari 和 Firefox 逐步跟进。

## Why — 为什么

### 适用场景

- 按钮悬浮/点击反馈（transition）
- 加载动画（animation + keyframes）
- 页面过渡和元素入场
- 交互微动效
- 数据可视化动画
- 3D 卡片翻转效果

### CSS 动画 vs JS 动画对比

| 对比维度 | CSS 动画 | JS 动画 |
|----------|---------|---------|
| 渲染性能 | 高（浏览器自动优化，可跳帧） | 需手动优化（rAF 节奏控制） |
| 主线程 | 不阻塞（合成层动画） | 可能阻塞（在主线程执行） |
| 控制能力 | 弱（声明式，无法暂停/seek/反转） | 强（可精确控制每个参数） |
| 动态参数 | 不支持（需预定义） | 支持（运行时计算） |
| 动画链编排 | 困难（animation-delay hack） | 简单（GSAP Timeline、Promise） |
| 代码量 | 少（声明式） | 多（命令式） |
| 调试 | DevTools Animations 面板 | 断点 / console.log |
| 兼容性 | 极好 | 极好 |
| 减少动画 | `prefers-reduced-motion` 原生支持 | 需手动检测 |

**选择建议：**

- 简单交互反馈（hover/click） → **CSS transition**
- 循环/入场/退场动画 → **CSS animation**
- 复杂编排/需要控制 → **JS 动画（GSAP/WAAPI）**
- 路由切换过渡 → **View Transitions API**

### 什么时候用 transition vs animation

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| hover/click 状态切换 | transition | 两态切换，简洁 |
| 页面加载入场 | animation | 自动播放，可设 fill-mode |
| 循环旋转/脉冲 | animation | iteration-count: infinite |
| 折叠/展开 | transition | 状态驱动，两态 |
| 多步复杂路径 | animation | 多关键帧 |
| 交错入场 | animation + delay | 多元素编排 |
| 用户拖拽跟随 | JS + transition | 需动态参数 |

### 性能对比

```
属性动画性能排序（从好到差）：

1. transform / opacity     → 仅 Composite（GPU 加速）
2. filter / backdrop-filter → Composite + 部分 Paint
3. color / background      → Paint + Composite
4. width / height / margin → Layout + Paint + Composite（重排）
5. top / left              → Layout + Paint + Composite（重排）
```

**GPU 加速原理：**

- 浏览器将元素提升到独立的合成层（Compositing Layer）
- 合成层由 GPU 直接处理，不经过 CPU 的 Layout 和 Paint 阶段
- 动画时只需在 GPU 上做矩阵运算，极快
- 但每个合成层都要占内存，不能滥用

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

### 示例1：常用过渡效果

```css
/* ---- 颜色过渡 ---- */
.color-btn {
    background-color: #3b82f6;
    color: white;
    transition: background-color 0.3s ease, color 0.3s ease;
}
.color-btn:hover {
    background-color: #1d4ed8;
    color: #e0e7ff;
}

/* ---- 尺寸过渡（用 transform 替代 width/height） ---- */
.scale-card {
    transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.scale-card:hover {
    transform: scale(1.05);
}

/* ---- 位置过渡 ---- */
.slide-box {
    transition: transform 0.4s ease-out;
}
.slide-box.active {
    transform: translateX(200px);
}

/* ---- 变形过渡（旋转+缩放组合） ---- */
.icon-btn {
    transition: transform 0.2s ease;
}
.icon-btn:hover {
    transform: rotate(15deg) scale(1.1);
}

/* ---- 多属性组合过渡 ---- */
.fancy-card {
    background: white;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease, background 0.3s ease;
}
.fancy-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 12px 24px rgba(0,0,0,0.15);
    background: #f0f9ff;
}
```

### 示例2：贝塞尔曲线调试与自定义

```css
/* Material Design 标准缓动曲线 */
.md-standard {
    transition: transform 0.3s cubic-bezier(0.2, 0, 0, 1);
}

/* iOS 弹性曲线 */
.ios-spring {
    transition: transform 0.5s cubic-bezier(0.28, 0.11, 0.32, 1);
    /* 或更强的弹跳 */
    transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

/* 过冲回弹（值超过1会产生过冲效果） */
.overshoot {
    /* Y值 > 1 会让动画超过目标值再回来 */
    transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

/* 快出慢停 */
.quick-out {
    transition: transform 0.3s cubic-bezier(0.0, 0.0, 0.2, 1);
}

/* 在线调试工具：cubic-bezier.com */
/* Chrome DevTools：点击时间函数可直接拖拽调参 */
```

### 示例3：steps() 逐帧动画

```css
/* ---- 打字效果 ---- */
@keyframes typing {
    from { width: 0; }
    to   { width: 20ch; }
}

.typing-text {
    font-family: monospace;
    white-space: nowrap;
    overflow: hidden;
    border-right: 2px solid currentColor;
    width: 0;
    animation: typing 2s steps(20) forwards,
               blink 0.5s step-end infinite alternate;
}

@keyframes blink {
    50% { border-color: transparent; }
}

/* ---- 精灵图动画 ---- */
/*
 * 假设精灵图：8帧，横向排列
 * 每帧 64x64px，总宽 512px
 */
@keyframes walk {
    from { background-position: 0 0; }
    to   { background-position: -512px 0; }
}

.sprite-character {
    width: 64px;
    height: 64px;
    background-image: url('sprite-walk.png');
    animation: walk 0.8s steps(8) infinite;
}

/* ---- 进度条分段 ---- */
@keyframes progress {
    from { width: 0; }
    to   { width: 100%; }
}

.step-progress {
    animation: progress 4s steps(10) forwards;
    /* 10段，每段10%，离散跳变 */
}
```

### 示例4：animation-fill-mode 详解

```css
/* ---- fill-mode: none（默认） ---- */
/* 动画结束后回到原始样式，延迟期间也是原始样式 */
.demo-none {
    opacity: 1;
    animation: fadeOut 1s ease 0.5s none;
    /* 0-0.5s: opacity:1（原始样式）
       0.5-1.5s: 动画中
       1.5s后: opacity:1（回到原始样式） */
}

/* ---- fill-mode: forwards ---- */
/* 动画结束后保持最后一帧状态 */
.demo-forwards {
    opacity: 0;
    animation: fadeIn 1s ease 0.5s forwards;
    /* 0-0.5s: opacity:0（原始样式）
       0.5-1.5s: 动画中
       1.5s后: opacity:1（保持最后一帧） */
}

/* ---- fill-mode: backwards ---- */
/* 延迟期间应用第一帧状态，结束后回到原始样式 */
.demo-backwards {
    opacity: 1;
    animation: fadeOut 1s ease 0.5s backwards;
    /* 0-0.5s: opacity:0（第一帧状态覆盖）
       0.5-1.5s: 动画中
       1.5s后: opacity:1（回到原始样式） */
}

/* ---- fill-mode: both ---- */
/* 延迟期间应用第一帧，结束后保持最后一帧 */
.demo-both {
    opacity: 0;
    animation: fadeIn 1s ease 0.5s both;
    /* 0-0.5s: opacity:0（第一帧）
       0.5-1.5s: 动画中
       1.5s后: opacity:1（最后一帧） */
}

@keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
}
@keyframes fadeOut {
    from { opacity: 1; }
    to   { opacity: 0; }
}
```

### 示例5：复杂关键帧动画

```css
/* ---- 弹跳小球 ---- */
@keyframes bounce {
    0%, 100% {
        transform: translateY(0);
        animation-timing-function: cubic-bezier(0.8, 0, 1, 1);
    }
    50% {
        transform: translateY(-80px);
        animation-timing-function: cubic-bezier(0, 0, 0.2, 1);
    }
}
.bouncing-ball {
    animation: bounce 1s infinite;
}

/* ---- 脉冲光环 ---- */
@keyframes pulse-ring {
    0% {
        transform: scale(0.8);
        opacity: 1;
    }
    100% {
        transform: scale(2.2);
        opacity: 0;
    }
}
.pulse-dot {
    position: relative;
}
.pulse-dot::before {
    content: '';
    position: absolute;
    inset: -4px;
    border-radius: 50%;
    border: 2px solid currentColor;
    animation: pulse-ring 1.5s ease-out infinite;
}

/* ---- 摇晃提示（输入错误） ---- */
@keyframes shake {
    0%, 100% { transform: translateX(0); }
    10%, 50%, 90% { transform: translateX(-4px); }
    30%, 70% { transform: translateX(4px); }
}
.shake {
    animation: shake 0.5s ease-in-out;
}

/* ---- 数字翻转（计数器效果） ---- */
@keyframes countUp {
    from { transform: translateY(100%); opacity: 0; }
    to   { transform: translateY(0); opacity: 1; }
}
.counter-digit {
    display: inline-block;
    overflow: hidden;
}
.counter-digit span {
    display: block;
    animation: countUp 0.4s ease-out both;
}

/* ---- 交错入场（配合 JS 设置 delay） ---- */
.stagger-item {
    opacity: 0;
    transform: translateY(20px);
    animation: fadeInUp 0.5s ease-out both;
    /* JS: element.style.animationDelay = `${index * 0.05}s` */
}
```

### 示例6：3D 变换与透视

```css
/* ---- 3D 卡片翻转 ---- */
.card-container {
    perspective: 800px;
}
.card-inner {
    position: relative;
    width: 300px;
    height: 200px;
    transform-style: preserve-3d;
    transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}
.card-container:hover .card-inner {
    transform: rotateY(180deg);
}
.card-front, .card-back {
    position: absolute;
    inset: 0;
    backface-visibility: hidden;
    border-radius: 12px;
}
.card-back {
    transform: rotateY(180deg);
}

/* ---- 3D 倾斜（鼠标跟踪） ---- */
.tilt-card {
    transform-style: preserve-3d;
    transition: transform 0.1s ease-out;
    /* JS 配合：
       card.style.transform =
         `rotateY(${mouseX}deg) rotateX(${mouseY}deg)` */
}

/* ---- 3D 立方体 ---- */
.cube-wrapper {
    perspective: 600px;
}
.cube {
    width: 100px;
    height: 100px;
    transform-style: preserve-3d;
    animation: cubeRotate 6s linear infinite;
}
.cube-face {
    position: absolute;
    width: 100px;
    height: 100px;
}
.cube-face.front  { transform: translateZ(50px); }
.cube-face.back   { transform: translateZ(-50px) rotateY(180deg); }
.cube-face.left   { transform: translateX(-50px) rotateY(-90deg); }
.cube-face.right  { transform: translateX(50px) rotateY(90deg); }
.cube-face.top    { transform: translateY(-50px) rotateX(90deg); }
.cube-face.bottom { transform: translateY(50px) rotateX(-90deg); }

@keyframes cubeRotate {
    0%   { transform: rotateX(0) rotateY(0); }
    100% { transform: rotateX(360deg) rotateY(360deg); }
}

/* ---- 视差滚动效果 ---- */
.parallax-container {
    perspective: 1px;
    height: 100vh;
    overflow-x: hidden;
    overflow-y: auto;
}
.parallax-layer-back {
    transform: translateZ(-2px) scale(3);
}
.parallax-layer-front {
    transform: translateZ(0);
}
```

### 示例7：动画性能优化实战

```css
/* ---- 1. 只动画 transform 和 opacity ---- */

/* ❌ 差：触发 Layout */
.bad-animate {
    transition: width 0.3s, height 0.3s, top 0.3s, left 0.3s;
}

/* ✅ 好：只触发 Composite */
.good-animate {
    transition: transform 0.3s, opacity 0.3s;
    /* 用 scaleX/Y 代替 width/height */
    /* 用 translate 代替 top/left */
}

/* ---- 2. contain 属性隔离布局 ---- */
.isolated-component {
    contain: layout style paint;
    /* 限制重排范围，避免影响其他元素 */
}

/* ---- 3. content-visibility 延迟渲染 ---- */
.lazy-section {
    content-visibility: auto;
    contain-intrinsic-size: 0 500px;
    /* 不在视口内时跳过渲染 */
}

/* ---- 4. will-change 谨慎使用 ---- */
/* ✅ hover 时添加 */
.card {
    transition: transform 0.3s ease;
}
.card:hover {
    will-change: transform;
}

/* ✅ 入场动画前添加，结束后移除（JS） */
// element.style.willChange = 'transform, opacity';
// requestAnimationFrame(() => {
//     element.classList.add('animate');
//     element.addEventListener('animationend', () => {
//         element.style.willChange = 'auto';
//     }, { once: true });
// });

/* ---- 5. 避免同时动画大量元素 ---- */
/* ❌ 100个元素同时入场 */
.list-item {
    animation: fadeIn 0.3s ease-out both;
}

/* ✅ 分批入场，减少同时合成层数量 */
.list-item {
    opacity: 0;
    animation: fadeIn 0.3s ease-out both;
    animation-delay: calc(var(--i) * 30ms);
    /* 或使用 IntersectionObserver 分批触发 */
}

/* ---- 6. 动画结束后移除合成层 ---- */
.animated-element {
    animation: slideIn 0.3s ease-out both;
}
/* JS: 动画结束后可以移除 animation，
   让浏览器回收合成层内存 */

/* ---- 7. 减少阴影和滤镜在动画中的使用 ---- */
/* ❌ 阴影变化触发 Paint */
.bad-shadow { transition: box-shadow 0.3s; }

/* ✅ 用伪元素 + opacity 模拟阴影 */
.good-shadow {
    position: relative;
}
.good-shadow::after {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: inherit;
    box-shadow: 0 8px 24px rgba(0,0,0,0.2);
    opacity: 0;
    transition: opacity 0.3s;
}
.good-shadow:hover::after {
    opacity: 1;
}
```

### 减少动画偏好适配

```css
/* ---- 基础适配 ---- */
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
    }
}

/* ---- 精细适配：保留必要反馈 ---- */
@media (prefers-reduced-motion: reduce) {
    /* 移除装饰性动画 */
    .decoration,
    .background-animation {
        animation: none !important;
    }

    /* 保留交互反馈，但缩短时间 */
    .button,
    .link {
        transition-duration: 0.01ms !important;
    }

    /* 入场改为直接显示 */
    .fade-in-up {
        animation: none !important;
        opacity: 1 !important;
        transform: none !important;
    }
}

/* ---- JS 检测 ---- */
// const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
// if (prefersReducedMotion.matches) {
//     // 跳过动画或使用极简方案
// }
// prefersReducedMotion.addEventListener('change', (e) => {
//     // 动态切换
// });
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 动画卡顿 | 动画了 layout 属性（width/height/margin） | 只动画 `transform` 和 `opacity` |
| 动画结束状态丢失 | 默认 `fill-mode: none` | 用 `animation-fill-mode: forwards` 或 `both` |
| `height: auto` 无法过渡 | `auto` 不是数值，无法插值 | 用 `max-height` hack 或 `grid-template-rows: 0fr → 1fr` |
| 移动端动画闪烁 | 层合成问题 | 加 `transform: translateZ(0)` 或 `will-change: transform` |
| `transition` 不生效 | `display: none → block` 不可过渡 | 用 `visibility` + `opacity` 或 JS 两帧延迟 |
| 动画闪烁/抖动 | 亚像素渲染问题 | 加 `transform: translateZ(0)` 或 `backface-visibility: hidden` |
| `transform` 覆盖问题 | 多次赋值 `transform` 会覆盖 | 合并到一条 `transform` 声明中 |
| `animation-delay` 占用周期 | delay 不算在 iteration 内 | 注意 `iteration-count` 不包含 delay 时间 |
| `border-radius` + `overflow: hidden` + 动画 | 合成层裁剪异常 | 加 `will-change: transform` 或用 `clip-path` 替代 |
| `transition` 自动反向 | 只在属性变化时过渡 | 正常行为：hover 进入和离开都会触发过渡 |
| `animation` 不自动反向 | direction 默认 `normal` | 用 `alternate` 实现来回动画 |
| 多动画冲突 | 同一属性被多个 animation 操作 | 避免多个动画操作同一属性 |

### 最佳实践

- 只动画 `transform` 和 `opacity`
- 用 `will-change` 提示浏览器，用完移除
- 始终添加 `prefers-reduced-motion` 降级
- 交互反馈用 transition（hover/click），循环动效用 animation
- `animation-fill-mode: both` 是最安全的默认选择
- 大量元素动画时分批进行，减少同时合成层
- 用 `contain` 和 `content-visibility` 限制重排范围
- 复杂动画优先考虑 GSAP 或 Web Animations API
- 阴影过渡用伪元素 + opacity 模拟，避免 Paint
- 用 CSS 自定义属性（`--delay`）实现交错动画

## 面试题

**Q1: CSS transition 和 animation 的区别是什么？**
> transition 用于两个状态间的平滑过渡，需要触发条件（如 hover），只能定义开始和结束状态，执行一次后停止；animation 通过 @keyframes 定义关键帧序列，可自动播放、循环、暂停，支持多个关键帧和复杂动画路径，不需要触发条件。交互反馈用 transition，循环/入场动效用 animation。

**Q2: 为什么推荐用 transform 和 opacity 做动画？**
> transform 和 opacity 的动画只触发合成层（Composite）操作，由 GPU 加速处理，跳过 CPU 的 Layout 和 Paint 阶段，性能最佳。动画 width/height/margin/top/left 等属性会触发布局重排（Layout + Paint + Composite），导致主线程阻塞造成卡顿。

**Q3: will-change 的作用和使用原则？**
> will-change 提前告知浏览器哪些属性即将变化，让浏览器可以提前创建合成层、优化渲染管线，减少动画启动时的卡顿。使用原则：(1) 不要全局滥用，每个合成层都占额外内存；(2) 在需要时添加，动画结束后移除；(3) 优先在 hover/focus 时设置，给浏览器准备时间；(4) 如果页面已经很流畅，不需要加 will-change。

**Q4: 如何实现动画的暂停和恢复？**
> CSS 方案：通过 `animation-play-state: paused / running` 控制，可配合 JS 切换类名。Web Animations API 方案：调用 `animation.pause()` 和 `animation.play()` 方法，还支持 `reverse()`、`finish()`、`seek(currentTime)` 等更精细的控制。WAAPI 方案功能更强，推荐在需要动态控制时使用。

**Q5: 什么是GPU加速（合成层）？如何触发？**
> GPU 加速是指浏览器将元素提升到独立的合成层，由 GPU 直接处理渲染，跳过 CPU 布局和绘制。触发方式：3D 变换（translateZ/translate3d）、will-change: transform/opacity、opacity 动画（值 < 1）、position: fixed（部分浏览器）、filter、backdrop-filter。注意不要过度创建合成层，每个合成层约消耗 width x height x 4 bytes 内存。

**Q6: height: auto 如何实现过渡动画？**
> auto 不是数值无法插值，直接 transition 无效。方案：(1) max-height hack：设一个足够大的 max-height 值，从 0 过渡到该值，但时间曲线不精确；(2) grid-template-rows 从 0fr 过渡到 1fr（现代方案，动画精确）；(3) JS 读取实际高度后设置具体像素值过渡（最精确但需要 JS）。

**Q7: animation-fill-mode 各值有什么区别？**
> none（默认）：动画结束后回到原始样式；forwards：结束后保持最后一帧状态；backwards：延迟期间应用第一帧状态；both：同时应用 forwards 和 backwards 行为。入场动画推荐 `both`，确保延迟期间显示初始状态、结束后保持最终状态。

**Q8: CSS 动画和 JS 动画如何选择？**
> 简单的交互反馈（hover/click 状态切换）用 CSS transition；自动播放的循环/入场动画用 CSS animation；需要暂停/恢复/反转/动态参数等精细控制时用 Web Animations API 或 GSAP；复杂动画编排（Timeline 链式动画）用 GSAP。CSS 动画性能更好（GPU 加速、不阻塞主线程），JS 动画控制力更强，根据场景权衡选择。

---

**相关链接：**
- [[CSS布局Flex与Grid]]
- [[CSS选择器与优先级]]
- [[CSS变量与自定义属性]]
- [[Web APIs与新特性/Web Animations API]]

---
tags:
  - Web前端
  - CSS
  - Canvas
  - SVG
  - 图形
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Canvas与SVG

## What — 是什么

> Canvas 是基于像素的 2D 绘图 API，通过 JS 逐帧绘制；SVG 是基于矢量的标记语言，通过 XML 描述图形。两者是 Web 图形渲染的两大方案。

**Canvas 核心概念：**

- **2D 上下文**：`canvas.getContext('2d')`，提供绑制路径、矩形、圆弧、文本等 API
- **逐帧绘制**：每次绘制覆盖上一帧，适合动画和游戏
- **像素操作**：`getImageData`/`putImageData` 直接操作像素
- **WebGL**：`canvas.getContext('webgl')`，GPU 加速的 3D 渲染

**SVG 核心概念：**

- **矢量图形**：放大不失真，基于 XML 的 DOM 元素
- **基本形状**：`<rect>`、`<circle>`、`<line>`、`<path>`、`<text>`
- **CSS 样式**：SVG 元素可用 CSS 控制样式和动画
- **交互**：SVG 元素可绑定事件，每个形状独立可操作

**关键特性：**

- Canvas 适合像素级操控、高频动画、图像处理
- SVG 适合图标、图表、地图、可交互图形
- Canvas 输出位图，SVG 输出矢量图

## Why — 为什么

**适用场景：**

- Canvas：游戏、数据可视化热力图、图像编辑器、粒子效果
- SVG：图标系统、数据图表、地图、Logo、动画插画

**对比：**

| 维度 | Canvas | SVG |
|------|--------|-----|
| 渲染方式 | 位图（像素） | 矢量（路径） |
| 缩放 | 模糊 | 清晰 |
| DOM | 无（单一元素） | 每个图形是 DOM 节点 |
| 事件 | 需手动计算碰撞 | 原生 DOM 事件 |
| 性能 | 大量元素更优 | 元素多时 DOM 开销大 |
| 动画 | JS 逐帧控制 | CSS/SMIL 动画 |
| 文件格式 | PNG/JPEG | SVG/XML |
| 可访问性 | 差（无结构） | 好（有语义） |

**优缺点：**

- ✅ Canvas 优点：
  - 像素级操控，适合图像处理
  - 大量图形时性能优于 SVG
  - WebGL 支持 3D 渲染
- ❌ Canvas 缺点：
  - 无 DOM 结构，事件处理复杂
  - 缩放模糊
  - 无障碍性差
- ✅ SVG 优点：
  - 矢量不失真
  - DOM 事件原生支持
  - CSS 可控样式和动画
- ❌ SVG 缺点：
  - 复杂图形 DOM 节点多，性能差
  - 像素级操作不便

## How — 怎么用

### Canvas 基础

```html
<canvas id="canvas" width="800" height="600"></canvas>
```

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

// 高清屏适配
function setupHiDPI(canvas, ctx, width, height) {
    const dpr = window.devicePixelRatio || 1;
    canvas.width = width * dpr;
    canvas.height = height * dpr;
    canvas.style.width = width + 'px';
    canvas.style.height = height + 'px';
    ctx.scale(dpr, dpr);
}

// 绘制矩形
ctx.fillStyle = '#3b82f6';
ctx.fillRect(10, 10, 100, 50);

// 绘制圆
ctx.beginPath();
ctx.arc(200, 100, 40, 0, Math.PI * 2);
ctx.fillStyle = '#ef4444';
ctx.fill();

// 绘制文本
ctx.font = '16px sans-serif';
ctx.fillStyle = '#1f2937';
ctx.fillText('Hello Canvas', 300, 100);

// 绘制路径
ctx.beginPath();
ctx.moveTo(400, 50);
ctx.lineTo(450, 100);
ctx.lineTo(350, 100);
ctx.closePath();
ctx.strokeStyle = '#10b981';
ctx.lineWidth = 2;
ctx.stroke();
```

**动画循环：**

```javascript
function drawFrame(time) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 更新位置
    const x = (time / 10) % canvas.width;

    // 绘制
    ctx.beginPath();
    ctx.arc(x, canvas.height / 2, 20, 0, Math.PI * 2);
    ctx.fillStyle = '#3b82f6';
    ctx.fill();

    requestAnimationFrame(drawFrame);
}
requestAnimationFrame(drawFrame);
```

**图像处理：**

```javascript
// 灰度化
async function grayscale(imageSrc) {
    const img = new Image();
    img.src = imageSrc;
    await img.decode();

    ctx.drawImage(img, 0, 0);
    const imageData = ctx.getImageData(0, 0, img.width, img.height);
    const data = imageData.data;

    for (let i = 0; i < data.length; i += 4) {
        const avg = data[i] * 0.299 + data[i + 1] * 0.587 + data[i + 2] * 0.114;
        data[i] = data[i + 1] = data[i + 2] = avg;
    }

    ctx.putImageData(imageData, 0, 0);
}
```

### SVG 基础

**内联 SVG：**

```html
<!-- 图标 -->
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <circle cx="12" cy="12" r="10"/>
    <path d="M8 12l3 3 5-5"/>
</svg>

<!-- 渐变 -->
<svg width="200" height="100">
    <defs>
        <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="0%">
            <stop offset="0%" style="stop-color:#3b82f6"/>
            <stop offset="100%" style="stop-color:#8b5cf6"/>
        </linearGradient>
    </defs>
    <rect width="200" height="100" rx="8" fill="url(#grad)"/>
</svg>
```

**SVG 图标系统：**

```html
<!-- sprite.svg（合并所有图标） -->
<svg xmlns="http://www.w3.org/2000/svg" style="display:none">
    <symbol id="icon-home" viewBox="0 0 24 24">
        <path d="M3 12l9-9 9 9M5 10v10a1 1 0 001 1h3m10-11v10a1 1 0 01-1 1h-3m-4 0v-6a1 1 0 011-1h2a1 1 0 011 1v6"/>
    </symbol>
    <symbol id="icon-user" viewBox="0 0 24 24">
        <circle cx="12" cy="8" r="4"/>
        <path d="M4 21v-1a6 6 0 0112 0v1"/>
    </symbol>
</svg>
```

```html
<!-- 使用图标 -->
<svg class="icon" width="20" height="20">
    <use href="sprite.svg#icon-home"/>
</svg>
```

**SVG CSS 动画：**

```html
<svg viewBox="0 0 100 100" width="100" height="100">
    <circle cx="50" cy="50" r="40" fill="none" stroke="#3b82f6" stroke-width="4"
            stroke-dasharray="251" stroke-dashoffset="251" class="spinner"/>
</svg>

<style>
.spinner {
    animation: spin 1.5s linear infinite;
}
@keyframes spin {
    to {
        stroke-dashoffset: 0;
        transform: rotate(360deg);
    }
}
</style>
```

**SVG + React：**

```tsx
// 可配置的图标组件
interface IconProps {
    name: string;
    size?: number;
    color?: string;
    className?: string;
}

function Icon({ name, size = 20, color = 'currentColor', className }: IconProps) {
    return (
        <svg
            width={size}
            height={size}
            fill="none"
            stroke={color}
            strokeWidth={2}
            strokeLinecap="round"
            strokeLinejoin="round"
            className={className}
        >
            <use href={`sprite.svg#icon-${name}`} />
        </svg>
    );
}

// 使用
<Icon name="home" size={24} color="#3b82f6" />
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Canvas 模糊 | 高清屏 DPR 未处理 | canvas 尺寸 × DPR + ctx.scale(dpr) |
| Canvas resize 丢失内容 | 改变 canvas 尺寸会清空 | resize 前保存 imageData 或重新绘制 |
| SVG 图标加载闪烁 | sprite 文件未预加载 | `<link rel="preload">` 或内联关键图标 |
| SVG 复杂图形卡顿 | DOM 节点太多 | 超过 1000 个元素考虑 Canvas |
| Canvas 事件难做 | 只有一个 DOM 元素 | 记录图形位置，点击时遍历判断碰撞 |

### 最佳实践

- 图标用 SVG（矢量清晰 + CSS 可控），动画/游戏/图像处理用 Canvas
- Canvas 高清屏必做 DPR 适配
- SVG 图标用 sprite 合并，减少请求
- SVG 图标用 `currentColor` 继承颜色，方便主题切换
- 大量图形（>1000）场景选 Canvas，交互丰富的选 SVG

## 面试题

**Q1: Canvas和SVG的核心区别是什么？**
> Canvas是基于像素的位图绘图，通过JS逐帧绘制，输出位图，无DOM结构，事件需手动计算碰撞；SVG是基于矢量的XML标记，每个图形是DOM节点，原生支持事件和CSS样式，缩放不失真。Canvas适合大量图形和高频动画，SVG适合图标和交互丰富的图形。

**Q2: Canvas如何适配高清屏？**
> 高清屏设备像素比(DPR)>1时，需将canvas的width/height属性设为CSS尺寸乘以DPR，再通过style设置CSS显示尺寸，最后调用`ctx.scale(dpr, dpr)`缩放绘制上下文。否则canvas会在高清屏上模糊，因为1个CSS像素对应多个物理像素。

**Q3: 什么时候选Canvas，什么时候选SVG？**
> 选型原则：需要像素级操作、图像处理、大量图形（>1000个元素）、高频动画/游戏选Canvas；需要缩放不失真、CSS样式控制、DOM事件交互、图标/图表/地图选SVG。实际项目中常结合使用，如图标用SVG，动画效果用Canvas。

**Q4: Canvas的事件处理为什么复杂？如何解决？**
> Canvas只是一个DOM元素，内部绘制的图形不是独立DOM节点，无法直接绑定事件。解决方案：1) 维护图形列表记录位置，点击时遍历判断碰撞（hit testing）；2) 使用`isPointInPath()`检测点击是否在路径内；3) 复杂场景可用Canvas库（Fabric.js/Konva.js）提供对象模型和事件系统。

---

**相关链接：**
- [[CSS动画与过渡]]
- [[CSS变量与主题系统]]
- [[DOM与事件机制]]

---
tags:
  - Web前端
  - CSS Houdini
  - 自定义绘制
  - Paint API
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS Houdini 与自定义绘制

## What — 什么是 CSS Houdini

CSS Houdini 是一组**底层 API**，将 CSS 引擎内部渲染管线暴露给开发者，使其能够通过 JavaScript 直接介入浏览器的样式计算、布局、绘制和动画流程。与传统的"绕过 CSS 用 JS 模拟"不同，Houdini 让扩展代码运行在渲染管线内部，浏览器将其视为原生 CSS 的一部分。

### 核心 API 一览

| API | 作用 | 成熟度 |
|-----|------|--------|
| **Paint API** | 自定义绘制逻辑，替代 background / border-image | Chrome/Edge 稳定，Firefox 部分支持 |
| **Animation API** | 自定义动画逻辑，支持滚动驱动动画 | Chrome/Edge 稳定 |
| **Layout API** | 自定义布局算法（如瀑布流、环形布局） | Chrome Origin Trial |
| **Typed OM** | 类型化的 CSS 值读写，替代字符串操作 | Chrome/Edge/Firefox 支持 |
| **Properties & Values API** | 注册自定义属性、类型约束与动画能力 | Chrome/Edge/Safari 支持 |
| **Worklets** | 轻量级 JS 执行环境，运行在渲染线程 | 与各 API 配合使用 |

### Houdini 扩展 CSS 渲染管线

浏览器渲染管线的每个阶段均可被 Houdini Worklet 拦截：

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌───────────┐
│  Style   │───▶│  Layout  │───▶│   Paint  │───▶│ Composite │
└──────────┘    └──────────┘    └──────────┘    └───────────┘
     ▲               ▲               ▲
     │               │               │
 Properties &     Layout API      Paint API
 Values API       (registerLayout) (registerPaint)
     │
 Typed OM
     │
 Animation API (跨阶段控制)
```

- **Style 阶段**：`Properties & Values API` + `Typed OM` 让自定义属性参与样式计算
- **Layout 阶段**：`Layout API` 替换或增强浏览器的布局算法
- **Paint 阶段**：`Paint API` 在绘制环节插入自定义 Canvas 绘制逻辑
- **Composite 阶段**：`Animation API` 控制帧级动画更新，无需主线程参与

## Why — 为什么需要 CSS Houdini

### 技术方案对比

| 维度 | 纯 CSS | JS 动画 (requestAnimationFrame) | Canvas/SVG | CSS Houdini |
|------|--------|-------------------------------|------------|-------------|
| 性能 | 高（GPU 加速） | 中（主线程阻塞风险） | 中高 | 高（Worklet 线程） |
| 可交互性 | 有限 | 强 | 强 | 强（inputProperties） |
| 样式集成 | 原生 | 需手动同步 | 需手动同步 | 原生（CSS 属性驱动） |
| 动画能力 | 声明式 | 命令式 | 命令式 | 声明式 + 命令式 |
| 降级处理 | 无需 | 无需 | 需 fallback | 需 `@supports` 检测 |
| 复杂视觉效果 | 受限 | 灵活 | 灵活 | 灵活 + 管线级优化 |
| 开发成本 | 低 | 中 | 高 | 中 |

### 典型使用场景

1. **自定义背景与边框**：波纹效果、点阵图案、跟随鼠标的渐变
2. **创意视觉特效**：粒子背景、噪声纹理、动态边框动画
3. **性能优先的动画**：滚动驱动动画、离屏 Worklet 动画，避免主线程卡顿
4. **非标准布局**：瀑布流、环形布局、自适应网格

## How — 如何使用 CSS Houdini

### 1. Typed OM — 类型化 CSS 值操作

Typed OM 用结构化对象替代字符串，提供类型安全的 CSS 值读写。

```js
// 传统方式：字符串拼接
element.style.opacity = '0.5';
element.style.width = '100px';

// Typed OM 方式：类型化赋值
element.attributeStyleMap.set('opacity', CSS.number(0.5));
element.attributeStyleMap.set('width', CSS.px(100));

// 读取
const opacity = element.attributeStyleMap.get('opacity'); // CSSUnitValue { value: 0.5, unit: 'number' }
const width = element.attributeStyleMap.get('width');     // CSSUnitValue { value: 100, unit: 'px' }

// 数学运算（无需手动解析字符串）
const newWidth = CSS.px(width.value * 2);
element.attributeStyleMap.set('width', newWidth);
```

```js
// computedStyleMap() 获取计算后的样式
const computed = element.computedStyleMap();
const fontSize = computed.get('font-size'); // CSSUnitValue { value: 16, unit: 'px' }
```

```js
// CSS 数学函数
const calcValue = CSS.calc(CSS.px(100).add(CSS.percent(50)));
element.attributeStyleMap.set('width', calcValue);
```

**常用工厂方法**：

| 方法 | 示例 | 结果 |
|------|------|------|
| `CSS.number(n)` | `CSS.number(0.8)` | `{ value: 0.8, unit: 'number' }` |
| `CSS.px(n)` | `CSS.px(100)` | `{ value: 100, unit: 'px' }` |
| `CSS.percent(n)` | `CSS.percent(50)` | `{ value: 50, unit: 'percent' }` |
| `CSS.deg(n)` | `CSS.deg(45)` | `{ value: 45, unit: 'deg' }` |
| `CSS.em(n)` | `CSS.em(2)` | `{ value: 2, unit: 'em' }` |

### 2. Paint API — 自定义绘制

Paint API 是 Houdini 中最成熟的 API，允许开发者在 CSS 渲染管线的绘制阶段插入自定义 Canvas 2D 绘制逻辑。

#### 基本流程

```
1. 编写 PaintWorklet 类（含 paint() 方法）
2. 注册 Worklet：registerPaint('name', WorkletClass)
3. 在 CSS 中使用：paint(name)
4. 可选：通过 inputProperties / inputArguments 接收外部参数
```

#### 示例：波纹效果

```js
// ripple-worklet.js
class RipplePainter {
  // 声明需要监听的 CSS 属性（变化时自动重绘）
  static get inputProperties() {
    return ['--ripple-x', '--ripple-y', '--ripple-color', '--ripple-size'];
  }

  paint(ctx, size, props) {
    const x = parseFloat(props.get('--ripple-x').toString()) || 0;
    const y = parseFloat(props.get('--ripple-y').toString()) || 0;
    const color = props.get('--ripple-color').toString() || 'rgba(0, 150, 255, 0.3)';
    const rippleSize = parseFloat(props.get('--ripple-size').toString()) || 0;

    ctx.clearRect(0, 0, size.width, size.height);

    // 绘制波纹
    const gradient = ctx.createRadialGradient(x, y, 0, x, y, rippleSize);
    gradient.addColorStop(0, color);
    gradient.addColorStop(1, 'transparent');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, size.width, size.height);
  }
}

registerPaint('ripple', RipplePainter);
```

```css
/* style.css */
.ripple-btn {
  --ripple-x: 0;
  --ripple-y: 0;
  --ripple-color: rgba(0, 150, 255, 0.3);
  --ripple-size: 0;
  background: paint(ripple);
  transition: --ripple-size 0.4s ease-out;
}
```

```js
// main.js
CSS.paintWorklet.addModule('./ripple-worklet.js');

document.querySelectorAll('.ripple-btn').forEach(btn => {
  btn.addEventListener('click', e => {
    const rect = btn.getBoundingClientRect();
    btn.attributeStyleMap.set('--ripple-x', CSS.px(e.clientX - rect.left));
    btn.attributeStyleMap.set('--ripple-y', CSS.px(e.clientY - rect.top));
    btn.attributeStyleMap.set('--ripple-size', CSS.px(Math.max(rect.width, rect.height) * 2));
  });
});
```

#### 示例：自定义虚线边框

```js
// dash-border-worklet.js
class DashBorderPainter {
  static get inputProperties() {
    return ['--dash-length', '--dash-gap', '--dash-color', '--dash-width'];
  }

  paint(ctx, size, props) {
    const dashLength = parseFloat(props.get('--dash-length')) || 8;
    const gap = parseFloat(props.get('--dash-gap')) || 4;
    const color = props.get('--dash-color').toString() || '#333';
    const lineWidth = parseFloat(props.get('--dash-width')) || 2;

    ctx.lineWidth = lineWidth;
    ctx.strokeStyle = color;
    ctx.setLineDash([dashLength, gap]);

    // 绘制矩形边框
    const offset = lineWidth / 2;
    ctx.strokeRect(offset, offset, size.width - lineWidth, size.height - lineWidth);
  }
}

registerPaint('dash-border', DashBorderPainter);
```

```css
.dash-box {
  --dash-length: 12;
  --dash-gap: 6;
  --dash-color: #e74c3c;
  --dash-width: 3;
  background: paint(dash-border);
}
```

#### 示例：点阵图案背景

```js
// dot-pattern-worklet.js
class DotPatternPainter {
  static get inputProperties() {
    return ['--dot-spacing', '--dot-radius', '--dot-color'];
  }

  paint(ctx, size, props) {
    const spacing = parseFloat(props.get('--dot-spacing')) || 20;
    const radius = parseFloat(props.get('--dot-radius')) || 2;
    const color = props.get('--dot-color').toString() || '#ccc';

    ctx.fillStyle = color;
    for (let y = radius; y < size.height; y += spacing) {
      for (let x = radius; x < size.width; x += spacing) {
        ctx.beginPath();
        ctx.arc(x, y, radius, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }
}

registerPaint('dot-pattern', DotPatternPainter);
```

#### 示例：跟随鼠标的渐变

```js
// mouse-gradient-worklet.js
class MouseGradientPainter {
  static get inputProperties() {
    return ['--mouse-x', '--mouse-y'];
  }

  paint(ctx, size, props) {
    const x = parseFloat(props.get('--mouse-x')) || size.width / 2;
    const y = parseFloat(props.get('--mouse-y')) || size.height / 2;

    const gradient = ctx.createRadialGradient(x, y, 0, x, y, 300);
    gradient.addColorStop(0, 'rgba(99, 102, 241, 0.4)');
    gradient.addColorStop(1, 'rgba(99, 102, 241, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, size.width, size.height);
  }
}

registerPaint('mouse-gradient', MouseGradientPainter);
```

```css
.card {
  --mouse-x: 50%;
  --mouse-y: 50%;
  background: paint(mouse-gradient);
}
```

```js
CSS.paintWorklet.addModule('./mouse-gradient-worklet.js');

document.querySelectorAll('.card').forEach(card => {
  card.addEventListener('mousemove', e => {
    const rect = card.getBoundingClientRect();
    card.attributeStyleMap.set('--mouse-x', CSS.px(e.clientX - rect.left));
    card.attributeStyleMap.set('--mouse-y', CSS.px(e.clientY - rect.top));
  });
});
```

### 3. Animation API — 自定义动画

Animation API 允许开发者编写运行在合成器线程上的自定义动画逻辑，实现滚动驱动动画等高性能效果。

```js
// scroll-animator.js
class ScrollFadeAnimator {
  // 构造函数接收 options（来自 new WorkletAnimation 时的参数）
  constructor(options) {
    this.fadeDirection = options.fadeDirection || 'out';
  }

  // animate() 在每一帧调用
  // currentTime: 当前时间线位置
  // effect: 关联的 KeyframeEffect
  animate(currentTime, effect) {
    const progress = currentTime / 1000; // 归一化到 0~1
    const clampedProgress = Math.min(1, Math.max(0, progress));

    if (this.fadeDirection === 'out') {
      effect.localTime = (1 - clampedProgress) * 1000;
    } else {
      effect.localTime = clampedProgress * 1000;
    }
  }
}

registerAnimator('scroll-fade', ScrollFadeAnimator);
```

```js
// main.js — 滚动驱动动画
CSS.animationWorklet.addModule('./scroll-animator.js');

const scrollTimeline = new ScrollTimeline({
  source: document.scrollingElement,
  orientation: 'block'
});

const fadeEffect = new KeyframeEffect(
  document.querySelector('.fade-element'),
  [
    { opacity: 1, transform: 'translateY(0)' },
    { opacity: 0, transform: 'translateY(-30px)' }
  ],
  { duration: 1000 }
);

new WorkletAnimation('scroll-fade', fadeEffect, scrollTimeline, {
  fadeDirection: 'out'
}).play();
```

### 4. Layout API — 自定义布局

Layout API 允许替换浏览器的布局算法，实现非标准布局。

```js
// masonry-layout.js
class MasonryLayout {
  static get inputProperties() {
    return ['--masonry-gap'];
  }

  // 声明子元素可使用的 CSS 属性
  static get childInputProperties() {
    return [];
  }

  async layout(children, edges, constraints, styleMap) {
    const gap = parseFloat(styleMap.get('--masonry-gap')) || 10;
    const inlineSize = constraints.fixedInlineSize;
    const columnCount = 3;
    const columnWidth = (inlineSize - gap * (columnCount - 1)) / columnCount;
    const columnHeights = new Array(columnCount).fill(0);

    const childFragments = [];
    const childLayouts = await Promise.all(children.map(child => {
      // 找到最短列
      const shortestColumn = columnHeights.indexOf(Math.min(...columnHeights));
      const x = shortestColumn * (columnWidth + gap);
      const y = columnHeights[shortestColumn];

      return child.layoutNextFragment({
        fixedInlineSize: columnWidth
      }).then(fragment => {
        fragment.inlineOffset = x;
        fragment.blockOffset = y;
        columnHeights[shortestColumn] += fragment.blockSize + gap;
        childFragments.push(fragment);
      });
    }));

    const blockSize = Math.max(...columnHeights);
    return { childFragments, autoBlockSize: blockSize };
  }
}

registerLayout('masonry', MasonryLayout);
```

```css
.masonry-container {
  --masonry-gap: 16;
  display: layout(masonry);
}
```

### 5. Properties & Values API — 自定义属性注册

让自定义属性具备类型约束、初始值和动画能力。

#### CSS 方式：@property

```css
@property --hue {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}

@property --gradient-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@property --glow-color {
  syntax: '<color>';
  inherits: true;
  initial-value: #6366f1;
}

/* 注册后即可对自定义属性使用 transition / animation */
.animated-border {
  --hue: 0;
  --gradient-angle: 0deg;
  border: 2px solid;
  border-image: conic-gradient(from var(--gradient-angle), hsl(var(--hue), 100%, 60%), #6366f1) 1;
  transition: --gradient-angle 0.3s, --hue 0.3s;
}

.animated-border:hover {
  --gradient-angle: 360deg;
  --hue: 360;
}
```

#### JS 方式：CSS.registerProperty

```js
CSS.registerProperty({
  name: '--ripple-size',
  syntax: '<length>',
  inherits: false,
  initialValue: '0px'
});

CSS.registerProperty({
  name: '--progress',
  syntax: '<percentage>',
  inherits: true,
  initialValue: '0%'
});
```

**syntax 常用类型**：

| syntax 值 | 含义 | 示例初始值 |
|-----------|------|-----------|
| `<length>` | 长度 | `0px` |
| `<percentage>` | 百分比 | `0%` |
| `<number>` | 数字 | `0` |
| `<angle>` | 角度 | `0deg` |
| `<color>` | 颜色 | `#000000` |
| `<url>` | URL | 无 |
| `<integer>` | 整数 | `0` |
| `*` | 任意值 | 空 |

### 6. Worklet 生命周期与性能模型

```
┌───────────────────────────────────────────┐
│              主线程 (Main Thread)          │
│                                           │
│  1. CSS.paintWorklet.addModule(url)       │
│  2. 浏览器加载 & 编译 Worklet 脚本        │
│  3. 样式变化 → 触发重绘/重排              │
│  4. 将绘制参数传给 Worklet 线程           │
└──────────────┬────────────────────────────┘
               │ 异步通信
┌──────────────▼────────────────────────────┐
│           Worklet 线程 (Render Thread)     │
│                                           │
│  5. 接收参数，执行 paint() / animate()    │
│  6. 输出绘制指令 → 合成器                  │
│  7. 不阻塞主线程                          │
└───────────────────────────────────────────┘
```

**关键约束**：

- Worklet 运行在独立线程，**无法访问 DOM、BOM、主线程变量**
- Worklet 代码必须是无状态的（`paint()` 每次调用都是独立的）
- 每个 Worklet 实例的生命周期由浏览器管理，可随时被回收重建
- `inputProperties` 变化时自动触发重绘，无需手动监听
- 通信只能通过 CSS 属性值传递，避免序列化开销

## 常见陷阱

| # | 陷阱 | 说明 | 解决方案 |
|---|------|------|---------|
| 1 | **Worklet 中使用 DOM API** | `document`、`window`、`localStorage` 在 Worklet 中不可用 | 仅使用 `ctx`（Canvas 2D 上下文）和传入的参数 |
| 2 | **Worklet 中使用外部状态** | Worklet 必须无状态，不能依赖闭包变量 | 所有数据通过 `inputProperties` 传入 |
| 3 | **忘记注册 @property** | 未注册的自定义属性是字符串，无法参与 transition | 使用 `@property` 或 `CSS.registerProperty()` 注册类型 |
| 4 | **syntax 类型不匹配** | `syntax: '<number>'` 但初始值写 `'0px'` | 初始值必须与 syntax 声明的类型严格一致 |
| 5 | **Paint Worklet 中创建 ImageBitmap** | `createImageBitmap()` 在部分浏览器中不可用 | 预加载图片并通过 `inputArguments` 传入 |
| 6 | **对不支持 Houdini 的浏览器无降级** | 旧浏览器直接忽略 `paint()` 声明 | 使用 `@supports (background: paint(id))` 提供回退样式 |
| 7 | **inputProperties 值解析错误** | `props.get()` 返回 `CSSStyleValue` 而非原始值 | 使用 `.toString()` 或 `parseFloat()` 正确解析 |
| 8 | **过度绘制导致性能下降** | 在 `paint()` 中做大量复杂计算 | 保持 paint() 逻辑轻量，复杂计算放在主线程 |
| 9 | **Worklet 文件跨域加载失败** | `addModule()` 要求同源或正确 CORS 头 | 确保 Worklet 文件与页面同源或配置 CORS |
| 10 | **在 registerPaint 前使用 paint()** | CSS 中先于 Worklet 加载使用 `paint()` | 确保 `addModule()` 完成后再应用样式 |

## 最佳实践

1. **渐进增强**：始终提供 CSS 回退方案，用 `@supports` 检测支持情况
2. **保持 paint() 轻量**：绘制逻辑应尽量简单，复杂运算放在主线程，仅传递结果
3. **利用 inputProperties 驱动响应式**：将交互状态（鼠标位置、进度等）映射为 CSS 自定义属性
4. **注册自定义属性**：凡是需要 transition 的自定义属性都必须通过 `@property` 注册
5. **模块化管理 Worklet**：每个 Worklet 单独文件，通过 `addModule()` 加载
6. **避免在 Worklet 中做 I/O**：不发起网络请求、不读取文件，保持纯计算
7. **使用 Typed OM 读写样式**：避免字符串拼接，用 `CSS.px()` 等工厂方法保证类型正确
8. **性能监控**：使用 DevTools Performance 面板检查 Worklet 执行耗时

## 面试题

### 1. CSS Houdini 解决了传统 CSS 扩展的什么痛点？

> 传统 CSS 扩展依赖浏览器实现，开发者无法介入渲染管线。Houdini 暴露了 Style/Layout/Paint/Composite 各阶段扩展点，使自定义逻辑成为浏览器渲染流程的一部分，获得性能和集成度的双重优势。

### 2. Paint API 和 Canvas 有什么本质区别？

> Canvas 是独立的渲染表面，需要手动管理 DOM 同步和尺寸响应；Paint API 运行在浏览器渲染管线内部，由浏览器自动调度重绘，通过 `inputProperties` 与 CSS 属性联动，无需手动同步。Paint Worklet 还运行在独立线程，不阻塞主线程。

### 3. Worklet 和 Web Worker 有什么区别？

> Web Worker 用于通用计算，可访问部分 Web API；Worklet 是轻量级、专用的执行环境，仅暴露特定 API（如 Canvas 2D 上下文），运行在渲染线程，生命周期由浏览器管理，要求无状态，可被浏览器随时创建和销毁。

### 4. 为什么自定义属性需要 @property 注册才能做 transition？

> 未注册的自定义属性值是字符串类型，浏览器无法确定如何在两个值之间插值。`@property` 通过 `syntax` 声明值类型，浏览器才知道用何种插值算法（如 `<number>` 线性插值、`<color>` 颜色插值、`<angle>` 角度插值）。

### 5. 如何实现滚动驱动的视差动画？

> 使用 Animation API 的 `ScrollTimeline` + `WorkletAnimation`：创建 ScrollTimeline 关联滚动容器，在 WorkletAnimator 的 `animate()` 方法中根据 currentTime 计算进度，控制 effect.localTime 驱动关键帧。整个动画运行在合成器线程，不阻塞主线程滚动。

### 6. Paint Worklet 中为什么不能使用 DOM？

> Worklet 运行在渲染线程（合成器线程），与主线程隔离。允许访问 DOM 会导致线程安全问题（竞态条件），且 DOM 操作可能阻塞渲染帧。因此 Worklet 只接收纯数据和 Canvas 上下文，确保线程安全和高性能。

### 7. 如何为不支持 Houdini 的浏览器提供降级方案？

> 两种方式：一是使用 `@supports (background: paint(id))` 条件样式提供回退；二是使用 CSS.paintWorklet 判断 API 是否存在，在 JS 中做特性检测后决定是否加载 Worklet。对于关键视觉，可提供纯 CSS 替代方案。

### 8. Houdini 的 Typed OM 相比 style 属性操作有什么优势？

> Typed OM 提供类型安全：`CSS.px(100)` 返回结构化对象而非字符串 `'100px'`，避免字符串拼接和解析错误；支持数学运算（`CSS.calc()`、`add()`/`sub()`）；`computedStyleMap()` 直接获取计算值对象；与 Houdini 其他 API（inputProperties 返回值）无缝衔接。

## 相关链接

- [CSS Houdini — MDN](https://developer.mozilla.org/en-US/docs/Web/Houdini)
- [CSS Paint API — W3C Spec](https://www.w3.org/TR/css-paint-api-1/)
- [CSS Properties and Values API — W3C Spec](https://www.w3.org/TR/css-properties-values-api-1/)
- [CSS Typed OM — W3C Spec](https://www.w3.org/TR/css-typed-om-1/)
- [CSS Animation Worklet — W3C Draft](https://www.w3.org/TR/css-animation-worklet-1/)
- [Houdini Samples — Google Chrome Labs](https://github.com/GoogleChromeLabs/houdini-samples)
- [Is Houdini Ready Yet?](https://ishoudinireadyyet.com/)
- [CSS Houdini 完全指南 — CSS-Tricks](https://css-tricks.com/an-introduction-to-css-houdini/)

---
tags:
  - Web前端
  - Web Components
  - Custom Elements
  - Shadow DOM
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Web Components

## What — 什么是 Web Components

Web Components 是浏览器原生支持的组件化标准，由三个核心 API 组成：Custom Elements（自定义元素）、Shadow DOM（影子 DOM）、HTML Templates（HTML 模板）。它让开发者可以创建跨框架复用的原生组件，不依赖 React/Vue 等库。

### 三大核心 API

| API | 职责 | 类比 |
|-----|------|------|
| Custom Elements | 注册自定义 HTML 标签 | Vue.component / React.Component |
| Shadow DOM | 样式与 DOM 隔离 | CSS Modules + scoped styles |
| HTML Templates | 声明可复用的 DOM 模板 | Vue `<template>` / JSX |
| ES Modules | 组件导入导出 | import / export |

### 与框架组件的区别

| 维度 | Web Components | React/Vue 组件 |
|------|---------------|----------------|
| 运行依赖 | 无（浏览器原生） | 需框架运行时 |
| 跨框架复用 | 天然支持 | 需要适配层 |
| 样式隔离 | Shadow DOM | CSS Modules / scoped |
| 数据绑定 | 手动（属性/事件） | 响应式系统 |
| 状态管理 | 手动 | 框架内置 |
| 生态 | 较小 | 成熟丰富 |

---

## Why — 为什么需要 Web Components

### 1. 跨框架复用

一个 `<my-button>` 组件可以在 React、Vue、Angular、Svelte 甚至纯 HTML 中直接使用，无需任何适配。

### 2. 样式真正隔离

Shadow DOM 内部的样式不会泄漏到外部，外部样式也不会影响内部。这是 CSS-in-JS、CSS Modules 都无法做到的彻底隔离。

### 3. 无依赖的原生能力

不引入任何框架，0KB 运行时。适合嵌入式组件、微前端场景、设计系统的基础组件。

### 4. 与标准对齐

Web Components 是 W3C 标准，不是某个公司的库。浏览器原生支持，不会被"弃坑"。

### 优缺点

- ✅ 优点：跨框架、样式隔离、零依赖、标准稳定
- ❌ 缺点：无响应式、手动 DOM 操作、SSR 复杂、生态小

---

## How — 怎么用

### 1. Custom Elements — 自定义元素

```js
// 定义自定义元素
class MyButton extends HTMLElement {
  // 观察的属性列表
  static get observedAttributes() {
    return ['variant', 'disabled']
  }

  constructor() {
    super()
    this._variant = 'primary'
    this._disabled = false
  }

  // 元素被插入 DOM
  connectedCallback() {
    this.render()
    this.addEventListener('click', this._handleClick)
  }

  // 元素被移除 DOM
  disconnectedCallback() {
    this.removeEventListener('click', this._handleClick)
  }

  // 属性变化
  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) return

    switch (name) {
      case 'variant':
        this._variant = newValue
        break
      case 'disabled':
        this._disabled = newValue !== null
        break
    }

    this.render()
  }

  _handleClick(e) {
    if (this._disabled) {
      e.preventDefault()
      e.stopPropagation()
      return
    }
    this.dispatchEvent(new CustomEvent('my-click', {
      bubbles: true,
      composed: true,
      detail: { variant: this._variant },
    }))
  }

  render() {
    this.innerHTML = `
      <button class="btn btn--${this._variant}"
        ${this._disabled ? 'disabled' : ''}>
        <slot></slot>
      </button>
    `
  }
}

// 注册自定义元素（必须含短横线）
customElements.define('my-button', MyButton)
```

```html
<!-- 使用 -->
<my-button variant="primary">Click Me</my-button>
<my-button variant="danger" disabled>Disabled</my-button>

<script>
  document.querySelector('my-button').addEventListener('my-click', (e) => {
    console.log('Clicked!', e.detail.variant)
  })
</script>
```

**生命周期回调**：

| 回调 | 触发时机 | 用途 |
|------|----------|------|
| `constructor()` | 元素创建 | 初始化状态、创建 Shadow DOM |
| `connectedCallback()` | 插入 DOM | 渲染、绑定事件、启动定时器 |
| `disconnectedCallback()` | 移除 DOM | 清理事件、停止定时器 |
| `attributeChangedCallback()` | observed 属性变化 | 更新渲染 |
| `adoptedCallback()` | 移动到新 document | 跨 iframe 场景 |

---

### 2. Shadow DOM — 影子 DOM

Shadow DOM 创建一个与外部隔离的 DOM 子树，内部样式不会泄漏。

```js
class MyCard extends HTMLElement {
  constructor() {
    super()
    // 创建开放式 Shadow DOM
    this.attachShadow({ mode: 'open' })
  }

  connectedCallback() {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
          border-radius: 8px;
          background: white;
          box-shadow: 0 1px 3px rgba(0,0,0,0.1);
          overflow: hidden;
        }

        :host([featured]) {
          border: 2px solid #3b82f6;
        }

        .card-header {
          padding: 16px;
          border-bottom: 1px solid #e5e7eb;
          font-weight: 600;
        }

        .card-body {
          padding: 16px;
        }

        /* ::slotted 选择外部传入的内容 */
        ::slotted(h3) {
          margin: 0;
          font-size: 1.125rem;
        }

        ::slotted(p) {
          margin: 0;
          color: #6b7280;
        }

        /* slot 默认内容 */
        .card-footer {
          padding: 12px 16px;
          border-top: 1px solid #e5e7eb;
          background: #f9fafb;
        }
      </style>

      <div class="card-header">
        <slot name="header">Default Header</slot>
      </div>
      <div class="card-body">
        <slot>Default content</slot>
      </div>
      <div class="card-footer">
        <slot name="footer"></slot>
      </div>
    `
  }
}

customElements.define('my-card', MyCard)
```

```html
<!-- 使用 -->
<my-card featured>
  <h3 slot="header">Card Title</h3>
  <p>This is the card body content.</p>
  <div slot="footer">
    <button>Action</button>
  </div>
</my-card>
```

**Shadow DOM 选择器**：

| 选择器 | 含义 |
|--------|------|
| `:host` | 选择宿主元素本身 |
| `:host(.active)` | 宿主元素有 `.active` 类时 |
| `:host([disabled])` | 宿主元素有 `disabled` 属性时 |
| `:host-context(.dark)` | 祖先元素有 `.dark` 类时 |
| `::slotted(*)` | 选择插槽中传入的内容 |
| `::part(name)` | 外部通过 `part` 属性选择内部元素 |

**open vs closed 模式**：

| 维度 | open | closed |
|------|------|--------|
| 外部访问 `shadowRoot` | 可以 | 返回 null |
| DevTools 调试 | 可见 | 不可见 |
| 外部样式查询 | 可以 | 不可以 |
| 实际使用 | 99% 场景 | 极少使用 |

---

### 3. HTML Templates — 模板

```html
<!-- <template> 内容不渲染，但可被 JS 克隆 -->
<template id="user-card-template">
  <style>
    .user-card {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 12px;
      border-radius: 8px;
      background: #f9fafb;
    }
    .avatar {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: #3b82f6;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .info { flex: 1; }
    .name { font-weight: 600; }
    .email { font-size: 0.875rem; color: #6b7280; }
  </style>
  <div class="user-card">
    <div class="avatar"></div>
    <div class="info">
      <div class="name"></div>
      <div class="email"></div>
    </div>
  </div>
</template>
```

```js
class UserCard extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
  }

  connectedCallback() {
    const template = document.getElementById('user-card-template')
    const clone = template.content.cloneNode(true)

    // 填充数据
    const name = this.getAttribute('name') || 'Unknown'
    const email = this.getAttribute('email') || ''
    const initials = name.split(' ').map(w => w[0]).join('').toUpperCase()

    clone.querySelector('.name').textContent = name
    clone.querySelector('.email').textContent = email
    clone.querySelector('.avatar').textContent = initials

    this.shadowRoot.appendChild(clone)
  }
}

customElements.define('user-card', UserCard)
```

```html
<user-card name="John Doe" email="john@example.com"></user-card>
```

---

### 4. ::part() — 外部样式穿透

Shadow DOM 默认阻止外部样式影响内部，但 `::part()` 允许组件作者主动暴露可样式化的部分。

```js
// 组件内部声明 part
this.shadowRoot.innerHTML = `
  <style>
    .container { padding: 16px; background: white; border-radius: 8px; }
    .header { font-size: 1.25rem; font-weight: bold; padding-bottom: 8px; }
    .body { color: #555; }
  </style>
  <div class="container">
    <div class="header" part="header">
      <slot name="header"></slot>
    </div>
    <div class="body" part="body">
      <slot></slot>
    </div>
  </div>
`
```

```css
/* 外部样式可以穿透到 part */
my-card::part(header) {
  color: #3b82f6;
  border-bottom: 2px solid #3b82f6;
}

my-card::part(body) {
  font-size: 0.95rem;
  line-height: 1.8;
}
```

---

### 5. CSS ::theme() — 全局主题穿透

```css
/* ::theme() 影响所有 Shadow Root 中的 part */
:root {
  --card-bg: white;
  --card-header-color: #1f2937;
}

my-card::theme(header) {
  color: var(--card-header-color);
}

my-card::theme(body) {
  color: #555;
}
```

---

### 6. 与 React/Vue 集成

```jsx
// React 中使用 Web Components
function App() {
  const handleClick = (e) => {
    console.log('Custom event:', e.detail)
  }

  return (
    <my-card featured onMyClick={handleClick}>
      <h3 slot="header">React + Web Components</h3>
      <p>Seamless integration</p>
    </my-card>
  )
}

// React 中创建 Web Component 的桥接
function WebComponent({ tag, props, children, ...events }) {
  const ref = useRef()

  useEffect(() => {
    const el = ref.current
    Object.entries(events).forEach(([name, handler]) => {
      el.addEventListener(name.replace(/^on/, '').toLowerCase(), handler)
    })
    return () => {
      Object.entries(events).forEach(([name, handler]) => {
        el.removeEventListener(name.replace(/^on/, '').toLowerCase(), handler)
      })
    }
  }, [events])

  return React.createElement(tag, { ref, ...props }, children)
}
```

```vue
<!-- Vue 中使用 Web Components -->
<template>
  <my-card :featured="isFeatured" @my-click="handleClick">
    <h3 slot="header">{{ title }}</h3>
    <p>{{ content }}</p>
  </my-card>
</template>

<script setup>
import './components/my-card.js'

const isFeatured = ref(true)
const title = ref('Vue + Web Components')

function handleClick(e) {
  console.log('Custom event:', e.detail)
}
</script>
```

```js
// vue.config.js — 配置 Vue 忽略自定义元素
export default {
  compilerOptions: {
    isCustomElement: (tag) => tag.startsWith('my-')
  }
}
```

---

### 7. Lit — Web Components 的最佳拍档

原生 Web Components 写法繁琐，Lit 是 Google 出品的轻量库（~5KB），极大简化开发：

```bash
npm install lit
```

```ts
import { LitElement, html, css } from 'lit'
import { customElement, property, query } from 'lit/decorators.js'

@customElement('my-button')
export class MyButton extends LitElement {
  static styles = css`
    :host {
      display: inline-block;
    }
    button {
      padding: 8px 16px;
      border-radius: 4px;
      border: none;
      cursor: pointer;
      font-weight: 500;
      transition: all 0.2s;
    }
    button:hover { opacity: 0.9; }
    button:disabled { opacity: 0.5; cursor: not-allowed; }
    button.primary { background: #3b82f6; color: white; }
    button.danger { background: #ef4444; color: white; }
    button.outline {
      background: transparent;
      border: 1px solid #3b82f6;
      color: #3b82f6;
    }
  `

  @property({ type: String }) variant = 'primary'
  @property({ type: Boolean, reflect: true }) disabled = false
  @property({ type: String }) label = ''

  @query('button') _button

  render() {
    return html`
      <button
        class=${this.variant}
        ?disabled=${this.disabled}
        @click=${this._onClick}
      >
        ${this.label || html`<slot></slot>`}
      </button>
    `
  }

  _onClick(e) {
    if (this.disabled) return
    this.dispatchEvent(new CustomEvent('my-click', {
      bubbles: true,
      composed: true,
      detail: { variant: this.variant },
    }))
  }
}
```

**Lit 核心特性**：

| 特性 | 说明 |
|------|------|
| `@property` | 声明响应式属性，变化自动重渲染 |
| `html` 模板 | Tagged Template，只更新变化的部分 |
| `css` 集成 | 组件级样式，编译时提取 |
| 装饰器 | 简化 Custom Elements 定义 |
| ~5KB | 极小运行时 |

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 样式无法穿透 Shadow DOM | Shadow DOM 的设计目标就是隔离 | 使用 `::part()` 暴露、CSS 变量穿透 |
| 表单元素无法提交 | Shadow DOM 内的 input 不在表单作用域 | 使用 `ElementInternals` API |
| React 事件不工作 | React 合成事件不识别自定义元素 | 用 `ref` + `addEventListener` |
| SSR 不支持 | Shadow DOM 依赖浏览器 API | 使用 Declarative Shadow DOM（`<template shadowroot>`） |
| 插槽内容样式不生效 | `::slotted()` 只能选择直接子元素 | 用 CSS 变量或 `::part()` 传递样式 |
| 性能差 | 频繁 `innerHTML` 重建 DOM | 使用 Lit 的响应式渲染 |

### 最佳实践

1. **用 Lit 而非原生 API**：原生写法过于繁琐，Lit 是事实标准。
2. **CSS 变量做主题穿透**：`:host` 中定义 CSS 变量，外部覆盖变量即可定制主题。
3. **`::part()` 精确暴露**：只暴露需要定制的部分，不要全部暴露。
4. **属性 vs 属性（Attribute vs Property）**：复杂值用 Property（JS 赋值），简单值用 Attribute（HTML 属性）。
5. **事件使用 `composed: true`**：让自定义事件穿透 Shadow DOM 边界。

---

## 面试题

### 1. Shadow DOM 的 open 和 closed 模式有什么区别？实际应该用哪个？

**答**：open 模式下外部可以通过 `element.shadowRoot` 访问内部 DOM，closed 模式下返回 `null`。理论上 closed 更安全，但实际中几乎没有使用场景：(1) DevTools 在 closed 模式下无法查看内部结构，调试困难；(2) `element.shadowRoot` 返回 null 意味着测试代码也无法访问；(3) 安全性是伪命题——浏览器 DevTools 仍可查看。99% 的场景用 open 模式。

---

### 2. Custom Elements 的 `connectedCallback` 和 `constructor` 有什么区别？各适合做什么？

**答**：`constructor` 在元素创建时调用（`document.createElement` 时就触发），此时元素还未插入 DOM，不能访问 `this.parentNode`、`this.getAttribute()` 可能返回 null。`connectedCallback` 在元素被插入 DOM 后调用，可以安全访问属性、父元素、执行 DOM 操作。分工：`constructor` 做初始化（创建 Shadow DOM、设置内部状态），`connectedCallback` 做渲染和事件绑定。注意 `connectedCallback` 可能被多次调用（元素移除后重新插入），所以要确保幂等性。

---

### 3. Web Components 如何实现样式主题化？

**答**：三种方式：(1) **CSS 变量穿透**——在 `:host` 中用 CSS 变量定义主题值，外部通过覆盖变量定制主题：`:host { --btn-bg: #3b82f6; }`，外部 `.dark my-button { --btn-bg: #1d4ed8; }`；(2) **`::part()` 暴露**——组件内部用 `part="header"` 标记元素，外部用 `my-card::part(header) { color: red; }` 定制；(3) **`:host-context()` 上下文感知**——`:host-context(.dark-theme) { background: #1f2937; }`，当祖先元素有 `.dark-theme` 时自动应用暗色样式。推荐 CSS 变量做整体主题，`::part()` 做局部定制。

---

### 4. Web Components 为什么不适合做整个应用？适合什么场景？

**答**：不适合做整个应用的原因：(1) 无响应式系统——手动 DOM 操作，开发效率远低于 React/Vue；(2) 无内置状态管理——复杂应用的状态流转难以维护；(3) SSR 复杂——Shadow DOM 依赖浏览器 API，服务端渲染需要 Declarative Shadow DOM 方案；(4) 生态匮乏——路由、表单验证、国际化都要自己造。适合的场景：(1) 跨框架共享的基础组件（按钮、输入框、弹窗）；(2) 设计系统的底层组件库；(3) 微前端中的独立功能模块；(4) 嵌入第三方网站的组件（如支付按钮、评论组件）。

---

### 5. `::slotted()` 选择器有什么限制？如何绕过？

**答**：`::slotted()` 有三个限制：(1) **只能选择插槽的直接子元素**——`::slotted(.inner)` 无法选中 `<slot>` 传入的 `<div><span class="inner">` 中的 span；(2) **特异性较低**——`::slotted(p)` 的特异性不如外部直接设置的 `p { color: red }`；(3) **不能用于组合选择器**——`::slotted(.a .b)` 无效。绕过方式：(1) 使用 CSS 变量——`::slotted(.card) { --text-color: red; }`，子元素继承变量；(2) 使用 `::part()` 代替——组件内部给元素添加 `part` 属性，外部用 `::part()` 精确控制；(3) 在传入内容自身的样式中设置样式（不依赖 Shadow DOM 内部）。

---

### 6. Lit 相比原生 Web Components 有哪些改进？

**答**：Lit 在原生 API 基础上做了四个关键改进：(1) **响应式属性**——`@property` 装饰器声明属性，值变化时自动触发重渲染，无需手动 `attributeChangedCallback` + `render()`；(2) **高效模板更新**——`html` tagged template 只更新变化的部分，而非整个 `innerHTML` 重建，性能接近虚拟 DOM；(3) **声明式模板**——`html\`<button @click=${handler}>${label}</button>\`` 替代手动 DOM 操作；(4) **CSS 集成**——`static styles` 在编译时提取为 `<style>` 标签，支持 CSS 变量和 `::part()`。代价是 ~5KB 运行时。

---

### 7. Declarative Shadow DOM 是什么？解决了什么问题？

**答**：Declarative Shadow DOM 允许在 HTML 中直接声明 Shadow DOM，不需要 JS 执行：`<template shadowrootmode="open"><style>...</style><div>content</div></template>`。它解决了两个问题：(1) **SSR**——服务端可以直接在 HTML 中输出 Shadow DOM 结构，浏览器解析时自动创建 Shadow Root，无需等待 JS 执行；(2) **FOUC 闪烁**——传统 Web Components 在 JS 执行后才创建 Shadow DOM 和注入样式，页面会闪烁。Declarative Shadow DOM 在 HTML 解析阶段就完成样式注入。浏览器兼容：Chrome 111+、Safari 16.4+。

---

### 8. Web Components 如何与微前端架构结合？

**答**：Web Components 天然适合微前端：(1) **隔离性**——Shadow DOM 确保各子应用的样式互不干扰；(2) **独立性**——每个子应用可以独立部署为一个 Custom Element；(3) **通信**——通过属性传值和 Custom Event 通信，与框架无关。实现模式：(1) 主应用加载各子应用的 JS 入口，子应用注册为 `<sub-app-a>`、`<sub-app-b>`；(2) 主应用通过属性传递上下文（如用户信息、路由状态），子应用通过 Custom Event 通知主应用；(3) 框架集成——子应用内部可以用 React/Vue，对外暴露为 Web Component。缺点是子应用间的 JS 沙箱隔离需要额外处理（Shadow DOM 只隔离样式和 DOM，不隔离 JS 全局变量）。

---

## 相关链接

- [Web Components — MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components)
- [Custom Elements — MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomElementRegistry)
- [Shadow DOM — MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/ShadowRoot)
- [Lit 官方文档](https://lit.dev/)
- [Declarative Shadow DOM — Chrome Blog](https://developer.chrome.com/docs/css-ui/declarative-shadow-dom)

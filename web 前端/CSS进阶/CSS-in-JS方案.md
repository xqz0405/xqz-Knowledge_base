---
tags:
  - Web前端
  - CSS
  - CSS-in-JS
  - Styled Components
  - CSS Modules
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS-in-JS 方案

## What — 什么是 CSS-in-JS

CSS-in-JS 是一种将 CSS 写在 JavaScript 中的样式方案。组件的样式与逻辑在同一个文件中定义，样式可以访问组件的 props 和 state，实现真正的组件级样式隔离。

### 核心流派

| 流派 | 代表 | 原理 | 运行时 |
|------|------|------|--------|
| 运行时 CSS-in-JS | Styled Components、Emotion | 运行时动态生成 `<style>` 标签 | 有 |
| 零运行时 CSS-in-JS | Vanilla Extract、Panda CSS | 编译时提取为 .css 文件 | 无 |
| 编译时原子化 | StyleX、Linaria | 编译时生成原子类 | 无 |
| CSS Modules | webpack/Vite 内置 | 编译时生成局部作用域类名 | 无 |

### 关键概念

| 概念 | 说明 |
|------|------|
| 样式隔离 | 每个组件的样式不会泄漏到其他组件 |
| 动态样式 | 根据组件 props/state 计算样式值 |
| 主题注入 | 通过 Context/Provider 共享主题变量 |
| SSR 兼容 | 服务端渲染时正确提取关键 CSS |
| Colocation | 样式与组件逻辑放在同一文件 |

---

## Why — 为什么需要 CSS-in-JS

### 1. 样式隔离是刚需

全局 CSS 的命名冲突是大项目的顽疾。BEM、命名空间都是人工约定，CSS-in-JS 从机制层面保证隔离——每个样式哈希后生成唯一类名，不可能冲突。

### 2. 样式跟随组件

传统方案中，组件的模板在 `.vue`/`.jsx`，样式在 `.scss`，改一个按钮要跨文件编辑。CSS-in-JS 让样式和逻辑同位，组件自包含，方便复用和删除。

### 3. 动态样式零成本

根据主题、状态、props 计算样式是前端常见需求。传统方案要预定义所有状态的 class 再切换；CSS-in-JS 直接在样式中写 JavaScript 表达式。

### 对比各方案

| 维度 | CSS Modules | Styled Components | Vanilla Extract | Tailwind |
|------|-------------|-------------------|-----------------|----------|
| 样式隔离 | 编译时哈希 | 运行时哈希 | 编译时哈希 | 原子类复用 |
| 动态样式 | 需切换 class | 原生支持 | 通过 CSS 变量 | 需切换 class |
| 运行时开销 | 无 | 有（~15KB） | 无 | 无 |
| SSR | 原生支持 | 需配置提取 | 原生支持 | 原生支持 |
| TypeScript | 无类型 | 模板字符串无检查 | 完整类型 | 需插件 |
| 学习成本 | 低 | 中 | 中高 | 中 |

### 优缺点

- ✅ 优点：样式隔离、动态样式、组件自包含、主题系统
- ❌ 缺点：运行时开销（运行时方案）、调试困难、构建复杂度增加

---

## How — 各方案详解

### 1. CSS Modules — 最轻量的样式隔离

CSS Modules 不是库，是构建工具提供的功能。它将 CSS 类名编译为带哈希的唯一名，实现局部作用域。

```css
/* Button.module.css */
.btn {
  padding: 8px 16px;
  border-radius: 4px;
  font-weight: 500;
}

.primary {
  composes: btn;
  background: #3b82f6;
  color: white;
}

.danger {
  composes: btn;
  background: #ef4444;
  color: white;
}
```

```jsx
import styles from './Button.module.css'

function Button({ variant = 'primary', children }) {
  return (
    <button className={styles[variant]}>
      {children}
    </button>
  )
}

export default Button
```

编译产物：`<button class="Button_primary_a3x9k">Click</button>`

**composes 组合**：

```css
/* 基础类 */
.base {
  padding: 8px 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* 组合基础类 + 额外样式 */
.large {
  composes: base;
  padding: 12px 24px;
  font-size: 1.125rem;
}
```

**TypeScript 支持**：

```ts
// Button.module.css.d.ts
declare const styles: {
  readonly btn: string
  readonly primary: string
  readonly danger: string
}
export default styles
```

---

### 2. Styled Components — 运行时经典方案

```bash
npm install styled-components
```

```jsx
import styled, { css, keyframes, createGlobalStyle, ThemeProvider } from 'styled-components'

// 基础样式组件
const Button = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: background 0.2s;

  /* 基于 props 动态样式 */
  background: ${props => props.$variant === 'primary' ? '#3b82f6' : '#6b7280'};
  color: white;

  &:hover {
    opacity: 0.9;
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`

// 继承与扩展
const LargeButton = styled(Button)`
  padding: 12px 24px;
  font-size: 1.125rem;
`

// 使用
function App() {
  return (
    <div>
      <Button $variant="primary">Primary</Button>
      <Button $variant="secondary">Secondary</Button>
      <Button disabled>Disabled</Button>
      <LargeButton $variant="primary">Large</LargeButton>
    </div>
  )
}
```

**共享样式片段 — css 辅助函数**：

```jsx
const flexCenter = css`
  display: flex;
  align-items: center;
  justify-content: center;
`

const Card = styled.div`
  ${flexCenter}
  flex-direction: column;
  padding: 24px;
  border-radius: 8px;
  background: white;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
`
```

**动画 — keyframes**：

```jsx
const fadeIn = keyframes`
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
`

const FadeInBox = styled.div`
  animation: ${fadeIn} 0.5s ease-out;
`
```

**主题系统**：

```jsx
const theme = {
  colors: {
    primary: '#3b82f6',
    secondary: '#6b7280',
    success: '#10b981',
    danger: '#ef4444',
    background: '#ffffff',
    text: '#1f2937',
  },
  spacing: {
    sm: '8px',
    md: '16px',
    lg: '24px',
  },
  borderRadius: {
    sm: '4px',
    md: '8px',
    lg: '12px',
  },
}

const ThemedButton = styled.button`
  padding: ${props => props.theme.spacing.sm} ${props => props.theme.spacing.md};
  background: ${props => props.theme.colors.primary};
  color: white;
  border-radius: ${props => props.theme.borderRadius.sm};
  border: none;
  cursor: pointer;
`

function App() {
  return (
    <ThemeProvider theme={theme}>
      <ThemedButton>Themed Button</ThemedButton>
    </ThemeProvider>
  )
}
```

**全局样式**：

```jsx
const GlobalStyle = createGlobalStyle`
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    font-family: system-ui, -apple-system, sans-serif;
    background: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
    line-height: 1.6;
  }
`
```

---

### 3. Emotion — 更灵活的运行时方案

```bash
npm install @emotion/react @emotion/styled
```

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react'
import styled from '@emotion/styled'

// css prop 方式（推荐，最简洁）
function Card({ title, highlighted }) {
  return (
    <div css={css`
      padding: 24px;
      border-radius: 8px;
      background: ${highlighted ? '#fef3c7' : 'white'};
      border: 1px solid ${highlighted ? '#f59e0b' : '#e5e7eb'};
      transition: all 0.2s;
    `}>
      <h3 css={css` font-size: 1.125rem; font-weight: 600; `}>
        {title}
      </h3>
    </div>
  )
}

// styled API（与 Styled Components 兼容）
const Button = styled.button`
  padding: 8px 16px;
  border-radius: 4px;
  background: ${props => props.primary ? '#3b82f6' : '#e5e7eb'};
  color: ${props => props.primary ? 'white' : '#374151'};
`

// 对象样式（类型安全）
const boxStyles = css({
  padding: '16px',
  borderRadius: '8px',
  backgroundColor: '#f9fafb',
  border: '1px solid #e5e7eb',
})
```

**与 Styled Components 的区别**：

| 维度 | Styled Components | Emotion |
|------|-------------------|---------|
| css prop | 不支持 | 支持（推荐方式） |
| 对象样式 | 不支持 | 支持 |
| 包体积 | ~16KB | ~12KB |
| SSR | 需 babel 插件 | 零配置 |
| 速度 | 略慢 | 更快（缓存优化） |

---

### 4. Vanilla Extract — 零运行时方案

```bash
npm install @vanilla-extract/css @vanilla-extract/recipes
```

```ts
// styles.css.ts — 注意是 .css.ts 后缀
import { style, styleVariants, createTheme, globalStyle } from '@vanilla-extract/css'
import { recipe } from '@vanilla-extract/recipes'

// 主题定义
const [themeClass, vars] = createTheme({
  color: {
    primary: '#3b82f6',
    secondary: '#6b7280',
    background: '#ffffff',
    text: '#1f2937',
  },
  space: {
    sm: '8px',
    md: '16px',
    lg: '24px',
  },
})

// 基础样式（编译时生成唯一类名）
export const buttonBase = style({
  padding: `${vars.space.sm} ${vars.space.md}`,
  borderRadius: '4px',
  fontWeight: 500,
  border: 'none',
  cursor: 'pointer',
  transition: 'background 0.2s',
  ':hover': {
    opacity: 0.9,
  },
  ':disabled': {
    opacity: 0.5,
    cursor: 'not-allowed',
  },
})

// 变体样式
export const buttonVariant = styleVariants({
  primary: { background: vars.color.primary, color: 'white' },
  secondary: { background: vars.color.secondary, color: 'white' },
  outline: {
    background: 'transparent',
    color: vars.color.primary,
    border: `1px solid ${vars.color.primary}`,
  },
})

// Recipe 模式（类似 cva）
export const button = recipe({
  base: {
    padding: `${vars.space.sm} ${vars.space.md}`,
    borderRadius: '4px',
    fontWeight: 500,
    border: 'none',
    cursor: 'pointer',
  },
  variants: {
    variant: {
      primary: { background: vars.color.primary, color: 'white' },
      secondary: { background: '#e5e7eb', color: '#374151' },
      danger: { background: '#ef4444', color: 'white' },
    },
    size: {
      sm: { padding: '4px 12px', fontSize: '0.875rem' },
      md: { padding: '8px 16px' },
      lg: { padding: '12px 24px', fontSize: '1.125rem' },
    },
  },
  defaultVariants: {
    variant: 'primary',
    size: 'md',
  },
})
```

```tsx
// Button.tsx
import { vars } from './styles.css'
import { button, themeClass } from './styles.css'

function Button({ variant, size, children }) {
  return (
    <button className={button({ variant, size })}>
      {children}
    </button>
  )
}

function App() {
  return (
    <div className={themeClass}>
      <Button variant="primary" size="md">Primary</Button>
      <Button variant="danger" size="lg">Danger</Button>
    </div>
  )
}
```

**Vanilla Extract 的优势**：

| 特性 | 说明 |
|------|------|
| 零运行时 | 编译产物是纯 .css 文件 |
| 完整 TypeScript | 类型检查样式属性和值 |
| Sprinkles | 类型安全的原子类系统 |
| 主题类型安全 | `createTheme` 返回强类型变量 |

---

### 5. StyleX — Meta 出品的编译时原子化方案

```bash
npm install @stylexjs/stylex
```

```tsx
import stylex from '@stylexjs/stylex'

const styles = stylex.create({
  base: {
    padding: '8px 16px',
    borderRadius: '4px',
    fontWeight: 500,
    border: 'none',
    cursor: 'pointer',
  },
  primary: {
    backgroundColor: '#3b82f6',
    color: 'white',
  },
  secondary: {
    backgroundColor: '#e5e7eb',
    color: '#374151',
  },
  disabled: {
    opacity: 0.5,
    cursor: 'not-allowed',
  },
})

function Button({ variant = 'primary', disabled, children }) {
  return (
    <button
      {...stylex.props(
        styles.base,
        styles[variant],
        disabled && styles.disabled,
      )}
      disabled={disabled}
    >
      {children}
    </button>
  )
}
```

**StyleX 的编译原理**：

```tsx
// 编译前
<button {...stylex.props(styles.base, styles.primary)}>

// 编译后——每个属性变成原子类
<button class="x1y0q6nb x78zum5 x1l90r2v">
```

编译产物是一个极小的 CSS 文件，所有样式被拆解为原子类，相同属性值共享同一个类名。

**StyleX 核心特性**：

| 特性 | 说明 |
|------|------|
| 编译时原子化 | 样式被拆解为原子类，产物极小 |
| 类型安全 | 完整 TypeScript 支持 |
| 条件样式 | `stylex.props()` 支持条件合并 |
| 主题支持 | `stylex.defineVars()` / `stylex.createTheme()` |
| 无运行时 | 编译后只有 CSS 类名 |

---

### 6. Panda CSS — 新一代零运行时方案

```bash
npm install -D @pandacss/dev
npx panda init
```

```tsx
import { css } from '../styled-system/css'

// 原子化写法
function Card() {
  return (
    <div className={css({
      padding: '6',
      borderRadius: 'lg',
      bg: 'white',
      boxShadow: 'sm',
      _hover: { boxShadow: 'md' },
    })}>
      <h3 className={css({ fontSize: 'lg', fontWeight: 'bold' })}>
        Title
      </h3>
    </div>
  )
}

// Recipe 模式（变体）
import { cva } from '../styled-system/css'

const button = cva({
  base: {
    padding: '2 4',
    borderRadius: 'md',
    fontWeight: 'semibold',
  },
  variants: {
    variant: {
      primary: { bg: 'blue.500', color: 'white' },
      secondary: { bg: 'gray.100', color: 'gray.800' },
    },
    size: {
      sm: { padding: '1 3', fontSize: 'sm' },
      md: { padding: '2 4', fontSize: 'md' },
    },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
})

// 使用
<button className={button({ variant: 'primary', size: 'md' })}>
  Click
</button>
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| SSR 样式闪烁 | 运行时 CSS-in-JS 的样式注入时机不对 | 使用 `ServerStyleSheet`（Styled Components）或 `extractCritical`（Emotion） |
| 运行时性能差 | 频繁 re-render 时动态样式计算开销大 | 用 `useMemo` 缓存样式，或迁移到零运行时方案 |
| DevTools 调试难 | 生成的类名是哈希 | 配置 `displayName`（开发模式显示组件名） |
| 包体积大 | 运行时 CSS-in-JS 库本身 ~15KB | 考虑 Vanilla Extract / StyleX 等零运行时方案 |
| CSS Modules 不支持动态样式 | 编译时无法访问运行时值 | 结合 CSS 变量：`style={{ '--color': props.color }}` + `.btn { color: var(--color); }` |
| 样式优先级问题 | 多个样式源竞争 | 使用 `&& { }` 增加优先级（Styled Components） |

### 最佳实践

1. **新项目优先零运行时**：Vanilla Extract、Panda CSS、StyleX 性能更优。
2. **运行时方案优化 SSR**：确保关键 CSS 在服务端正确提取和内联。
3. **控制动态样式粒度**：只对真正需要动态计算的属性使用 JS 表达式，静态值用 CSS 变量。
4. **样式与逻辑分离**：即使样式写在 JS 中，也应提取到 `.css.ts` 文件，保持组件文件整洁。
5. **主题用 CSS 变量**：零运行时方案中用 CSS 变量传递主题值，避免 JS 运行时依赖。
6. **开发环境开启 displayName**：方便调试，生产环境自动移除。

---

## 面试题

### 1. CSS Modules 和 CSS-in-JS 的核心区别是什么？

**答**：CSS Modules 在编译时通过哈希类名实现作用域隔离，产物是普通 CSS 文件，无运行时开销；CSS-in-JS（运行时方案如 Styled Components）在运行时动态生成 `<style>` 标签注入样式，支持基于 props 的动态样式计算。核心区别：(1) CSS Modules 不支持真正的动态样式（只能切换预定义的 class），CSS-in-JS 可以在样式中写 JS 表达式；(2) CSS Modules 零运行时开销，CSS-in-JS 有 10-15KB 运行时且每次渲染都需计算样式；(3) CSS Modules 的样式写在 .css 文件中，CSS-in-JS 写在 .js/.ts 文件中。

---

### 2. 运行时 CSS-in-JS 的性能问题有哪些？如何优化？

**答**：三大性能问题：(1) **样式计算开销**——每次渲染都要执行模板字符串生成 CSS，大量动态组件时显著增加主线程负担；(2) **样式注入开销**——运行时创建和更新 `<style>` 标签，触发浏览器样式重计算；(3) **SSR 闪烁**—— hydration 前后样式可能不一致（FOUC）。优化方案：(1) 用 `useMemo` / `useCallback` 缓存样式对象，避免每次渲染重新计算；(2) 减少动态样式——能用 CSS 变量 + 静态类解决的不要用 JS 表达式；(3) 启用 Styled Components 的 `scss` 预处理加速；(4) 终极方案：迁移到零运行时方案（Vanilla Extract / StyleX）。

---

### 3. Vanilla Extract 和 Styled Components 的核心架构差异是什么？

**答**：Styled Components 是运行时方案：组件渲染时动态生成 CSS 字符串，通过 `<style>` 标签注入页面，支持基于 props 的实时样式计算。Vanilla Extract 是零运行时方案：样式定义在 `.css.ts` 文件中，构建时编译为纯 `.css` 文件，产物中没有任何 JS 运行时代码。这意味着 Vanilla Extract：(1) 没有运行时开销，首屏渲染更快；(2) 无法在样式中直接使用 props（需要通过 CSS 变量桥接）；(3) 提供完整 TypeScript 类型检查（属性名、属性值都有类型提示）；(4) SSR 无需额外配置。

---

### 4. StyleX 是如何实现"编译时原子化"的？

**答**：StyleX 的编译过程分两步：(1) **属性拆解**——`stylex.create()` 中定义的每个样式对象，其每个 CSS 属性被拆解为独立的原子规则，如 `{ padding: '8px', color: 'red' }` 变成两条原子规则；(2) **原子去重**——相同属性值的所有样式共享同一个原子类名，如多个组件都使用 `color: red`，最终只生成一个 `.x1a2b3c { color: red; }`。编译产物是一个极小的 CSS 文件，类名在 JS 中通过 `stylex.props()` 合并。这实现了零运行时 + 原子化的双重优势。

---

### 5. CSS-in-JS 如何实现主题切换？

**答**：三种方式：(1) **运行时方案（Styled Components/Emotion）**——通过 React Context 传递主题对象，样式函数中用 `props.theme` 访问主题值，切换主题只需更新 Provider 的 value，所有组件自动响应；(2) **零运行时方案（Vanilla Extract）**——用 `createTheme()` 生成主题类名和 CSS 变量，切换主题时替换根元素的 `className`，CSS 变量值随之变化；(3) **CSS 变量方案（通用）**——在 `:root` / `[data-theme="dark"]` 中定义 CSS 变量，样式直接引用 `var(--color-primary)`，切换主题只需修改根元素属性。零运行时方案推荐方式(2)或(3)。

---

### 6. 如何在 CSS Modules 中实现动态样式？

**答**：CSS Modules 本身是编译时方案，不支持在样式中写 JS 表达式。实现动态样式有三种方式：(1) **组合变体类**——预定义所有状态的 class（`.primary`、`.danger`），通过 `className={styles[variant]}` 切换；(2) **CSS 变量桥接**——在行内 style 中设置 CSS 变量值，CSS Modules 中引用该变量：`style={{ '--progress': percent }}` + `.bar { width: var(--progress); }`；(3) **composes 组合**——用 `composes` 复用基础样式，减少重复代码。方式(2)最适合连续值动态场景（如进度条、滑块），方式(1)适合离散状态切换。

---

### 7. 为什么 Meta 弃用 Styled Components 转向 StyleX？

**答**：核心原因是性能。Facebook 的页面组件树极其庞大，运行时 CSS-in-JS 的问题在规模放大后变得严重：(1) 每次渲染都要执行 JS 计算 CSS 字符串，主线程压力大；(2) 大量动态 `<style>` 标签注入导致样式重计算频繁；(3) SSR 样式提取和 hydration 的复杂度高，偶尔出现样式闪烁。StyleX 通过编译时原子化解决了这些问题：产物只有纯 CSS 类名和一个小 CSS 文件，运行时零开销。Meta 报告迁移后 CSS 产物体积减少约 80%，首屏渲染速度显著提升。

---

### 8. 如何选择 CSS-in-JS 方案？给出决策依据。

**答**：决策树：(1) **项目是否极度重视性能？**（如电商、社交 feed）→ 零运行时方案（Vanilla Extract / StyleX / Panda CSS）；(2) **团队是否习惯模板字符串写样式？** → Styled Components / Emotion（开发体验好，但接受运行时开销）；(3) **是否需要完整 TypeScript 类型安全？** → Vanilla Extract / StyleX / Panda CSS；(4) **是否需要兼容现有 CSS 生态？** → CSS Modules（最轻量，渐进增强）；(5) **是否使用 React Server Components？** → 零运行时方案（运行时 CSS-in-JS 与 RSC 不兼容）。通用建议：新项目首选 Vanilla Extract 或 Panda CSS；已有 Styled Components 的项目不必急于迁移，但应避免在性能关键路径上使用动态样式。

---

## 相关链接

- [Styled Components 官方文档](https://styled-components.com/)
- [Emotion 官方文档](https://emotion.sh/)
- [Vanilla Extract 官方文档](https://vanilla-extract.style/)
- [StyleX 官方文档](https://stylexjs.com/)
- [Panda CSS 官方文档](https://panda-css.com/)
- [CSS Modules — GitHub](https://github.com/css-modules/css-modules)

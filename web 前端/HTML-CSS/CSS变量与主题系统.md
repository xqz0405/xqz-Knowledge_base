---
tags:
  - Web前端
  - CSS
  - 变量
  - 主题
  - 暗色模式
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS变量与主题系统

## What — 是什么

> CSS 自定义属性（CSS Variables）是原生 CSS 的变量机制，是实现主题系统、暗色模式和设计令牌的基础。

**核心概念：**

- **定义与使用**：`--name: value` 定义，`var(--name)` 使用
- **作用域**：变量遵循 CSS 层叠规则，可在任何选择器上定义
- **默认值**：`var(--name, fallback)` 第二参数为默认值
- **全局变量**：在 `:root` 上定义，全文档可用
- **运行时动态**：JS 可通过 `style.setProperty` 实时修改

**关键特性：**

- CSS 变量是运行时的，不是编译时的（与 Less/Sass 变量不同）
- 变量可继承、可覆盖、可组合
- 是实现 CSS 设计系统（Design Tokens）的原生方案

## Why — 为什么

**适用场景：**

- 主题系统（亮色/暗色/品牌色）
- 设计令牌（间距、颜色、字号统一管理）
- 组件样式定制（通过 CSS 变量暴露样式接口）
- 运行时动态样式（用户偏好、A/B 测试）

**对比替代方案：**

| 维度 | CSS 变量 | Sass 变量 | Less 变量 | CSS-in-JS |
|------|---------|----------|----------|-----------|
| 运行时 | ✅ 动态 | ❌ 编译时 | ❌ 编译时 | ✅ 动态 |
| 主题切换 | 原生支持 | 需多套 CSS | 需多套 CSS | 运行时切换 |
| 浏览器支持 | 全部（IE 除外） | 无关（编译后） | 无关（编译后） | 全部 |
| 性能 | 无额外开销 | 无 | 无 | 运行时开销 |

**优缺点：**

- ✅ 优点：
  - 原生支持，无构建依赖
  - 运行时可变，主题切换零成本
  - 与层叠规则结合，灵活覆盖
- ❌ 缺点：
  - IE 不支持（已淘汰，基本无影响）
  - 无法做编译时计算（需 Sass/PostCSS 辅助）

## How — 怎么用

### 快速上手

```css
/* 全局设计令牌 */
:root {
    /* 颜色 */
    --color-primary: #3b82f6;
    --color-primary-hover: #2563eb;
    --color-bg: #ffffff;
    --color-text: #1f2937;
    --color-border: #e5e7eb;

    /* 间距 */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;

    /* 字号 */
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.25rem;

    /* 圆角 */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;

    /* 阴影 */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* 使用 */
.card {
    background: var(--color-bg);
    color: var(--color-text);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    padding: var(--spacing-md);
    box-shadow: var(--shadow-sm);
}
```

### 代码示例

**暗色模式：**

```css
/* 方式1：media query（跟随系统） */
@media (prefers-color-scheme: dark) {
    :root {
        --color-bg: #1a1a2e;
        --color-text: #e5e7eb;
        --color-border: #374151;
        --color-primary: #60a5fa;
        --color-primary-hover: #93bbfd;
        --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
    }
}

/* 方式2：class 切换（用户手动选择） */
[data-theme="dark"] {
    --color-bg: #1a1a2e;
    --color-text: #e5e7eb;
    --color-border: #374151;
    --color-primary: #60a5fa;
}

/* 方式3：两者结合 */
:root { /* 亮色 */ }
:root[data-theme="dark"],
[data-theme="dark"] { /* 暗色 */ }
@media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) { /* 系统暗色但用户未选亮色 */ }
}
```

**JS 切换主题：**

```typescript
// theme.ts
type Theme = 'light' | 'dark' | 'system';

function getSystemTheme(): 'light' | 'dark' {
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

function applyTheme(theme: Theme) {
    const effective = theme === 'system' ? getSystemTheme() : theme;
    document.documentElement.setAttribute('data-theme', effective);
    // 同步 <meta name="theme-color"> 让浏览器 UI 配合
    document.querySelector('meta[name="theme-color"]')
        ?.setAttribute('content', effective === 'dark' ? '#1a1a2e' : '#ffffff');
    localStorage.setItem('theme', theme);
}

function initTheme() {
    const saved = (localStorage.getItem('theme') ?? 'system') as Theme;
    applyTheme(saved);

    // 监听系统主题变化
    window.matchMedia('(prefers-color-scheme: dark)')
        .addEventListener('change', () => {
            if (localStorage.getItem('theme') === 'system') {
                applyTheme('system');
            }
        });
}

// 防止闪烁：在 <head> 中内联执行
// <script>document.documentElement.setAttribute('data-theme',
//   localStorage.getItem('theme') === 'dark' ||
//   (!localStorage.getItem('theme') && matchMedia('(prefers-color-scheme:dark)').matches)
//     ? 'dark' : 'light')</script>
```

**Vue 主题 Composable：**

```typescript
function useTheme() {
    const theme = ref<Theme>((localStorage.getItem('theme') as Theme) ?? 'system');

    const effectiveTheme = computed(() =>
        theme.value === 'system'
            ? window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
            : theme.value
    );

    watchEffect(() => {
        document.documentElement.setAttribute('data-theme', effectiveTheme.value);
        localStorage.setItem('theme', theme.value);
    });

    function toggle() {
        const next = { light: 'dark', dark: 'system', system: 'light' };
        theme.value = next[theme.value];
    }

    return { theme, effectiveTheme, toggle };
}
```

**React 主题 Hook：**

```typescript
function useTheme() {
    const [theme, setTheme] = useState<Theme>(() =>
        (localStorage.getItem('theme') as Theme) ?? 'system'
    );

    const effectiveTheme = useMemo(() => {
        if (theme !== 'system') return theme;
        return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }, [theme]);

    useEffect(() => {
        document.documentElement.setAttribute('data-theme', effectiveTheme);
        localStorage.setItem('theme', theme);
    }, [theme, effectiveTheme]);

    return { theme, setTheme, effectiveTheme };
}
```

**组件级样式定制：**

```css
/* 通过 CSS 变量暴露组件样式接口 */
.button {
    --btn-bg: var(--color-primary);
    --btn-color: white;
    --btn-radius: var(--radius-md);
    --btn-padding: var(--spacing-sm) var(--spacing-md);

    background: var(--btn-bg);
    color: var(--btn-color);
    border-radius: var(--btn-radius);
    padding: var(--btn-padding);
}

/* 使用时覆盖 */
.button-danger {
    --btn-bg: #ef4444;
}

.button-outline {
    --btn-bg: transparent;
    --btn-color: var(--color-primary);
    --btn-radius: var(--radius-lg);
}

/* 父级作用域覆盖 */
.card .button {
    --btn-radius: var(--radius-sm);
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 暗色模式闪烁 | JS 在 body 之后才设置 data-theme | `<head>` 内联脚本提前设置 |
| var() 回调不生效 | 默认值语法错误 | `var(--x, #fff)` 注意逗号后是完整值 |
| 变量未继承 | 定义在选择器内部 | 全局变量放在 `:root` |
| 组件库变量冲突 | 命名无前缀 | 组件变量加前缀 `--btn-*` |
| 计算不生效 | `var()` 不能直接参与计算 | 用 `calc(var(--x) * 2)` 包裹 |

### 最佳实践

- 全局设计令牌放 `:root`，组件变量加前缀 `--btn-*`
- 暗色模式用 `[data-theme]` class 切换 + `prefers-color-scheme` 系统跟随
- `<head>` 内联脚本防止暗色闪烁
- 组件通过 CSS 变量暴露样式接口，而非不断增加 props
- 用 CSS 变量管理设计令牌，Sass 处理编译时计算

## 面试题

**Q1: CSS变量和Sass变量有什么区别？**
> CSS变量是运行时变量，浏览器原生支持，可通过JS动态修改，遵循层叠和继承规则；Sass变量是编译时变量，构建后变成固定值，无法运行时改变，但支持编译时计算和条件逻辑。两者可配合使用：Sass处理编译时逻辑，CSS变量处理运行时主题。

**Q2: 如何实现暗色模式？**
> 三种方式：1) `@media (prefers-color-scheme: dark)`跟随系统主题；2) `[data-theme="dark"]`通过class切换支持用户手动选择；3) 两者结合：默认跟随系统，用户选择后用class覆盖。关键：在`<head>`中内联脚本提前设置data-theme防止闪烁，配合CSS变量切换颜色令牌。

**Q3: 如何用JS动态修改CSS变量？**
> 使用`element.style.setProperty('--name', 'value')`修改，如`document.documentElement.style.setProperty('--color-primary', '#ff0000')`。读取用`getComputedStyle(element).getPropertyValue('--name')`。Vue/React中可通过响应式变量+watchEffect/useEffect自动同步。

**Q4: CSS变量的默认值语法是什么？常见坑有哪些？**
> 语法：`var(--name, fallback)`，第二参数为默认值，当变量未定义时使用。常见坑：1) 默认值中的逗号不会被当作参数分隔符，如`var(--font, 16px, sans-serif)`的默认值是`16px, sans-serif`整段；2) 变量设为空串不会触发fallback；3) 不能直接参与计算，需用calc()包裹。

---

**相关链接：**
- [[CSS选择器与优先级]]
- [[CSS动画与过渡]]
- [[响应式设计]]

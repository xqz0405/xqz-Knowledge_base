---
tags:
  - Web前端
  - CSS
  - 选择器
  - 优先级
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CSS选择器与优先级

## What — 是什么

> CSS 选择器决定样式应用到哪些元素，优先级（Specificity）决定冲突时哪条规则生效。掌握选择器和优先级是写出可维护 CSS 的基础。

### 选择器分类

**基础选择器：**

| 类型 | 语法 | 优先级 | 示例 |
|------|------|--------|------|
| 通配符 | `*` | (0,0,0) | `* { box-sizing: border-box; }` |
| 元素选择器 | `tag` | (0,0,1) | `div { margin: 0; }` |
| 类选择器 | `.class` | (0,1,0) | `.card { padding: 16px; }` |
| ID 选择器 | `#id` | (1,0,0) | `#header { height: 60px; }` |
| 属性选择器 | `[attr]` | (0,1,0) | `[disabled] { opacity: 0.5; }` |

**组合选择器：**

| 类型 | 语法 | 含义 |
|------|------|------|
| 后代 | `A B` | A 后代中的 B |
| 子代 | `A > B` | A 直接子元素 B |
| 相邻兄弟 | `A + B` | A 紧邻的下一个兄弟 B |
| 通用兄弟 | `A ~ B` | A 后面所有兄弟 B |

**伪类选择器（单冒号 `:`）：**

| 类别 | 选择器 | 说明 |
|------|--------|------|
| 交互状态 | `:hover` / `:focus` / `:active` / `:visited` | 用户交互 |
| 表单状态 | `:checked` / `:disabled` / `:enabled` / `:read-only` | 表单控件 |
| 结构位置 | `:first-child` / `:last-child` / `:nth-child(n)` / `:only-child` | 子元素位置 |
| 类型位置 | `:first-of-type` / `:last-of-type` / `:nth-of-type(n)` | 同类型位置 |
| 逻辑组合 | `:not()` / `:is()` / `:where()` / `:has()` | 条件匹配 |
| 输入验证 | `:valid` / `:invalid` / `:in-range` / `:out-of-range` | 表单验证 |

**伪元素选择器（双冒号 `::`）：**

| 选择器 | 说明 | 优先级 |
|--------|------|--------|
| `::before` / `::after` | 元素前后插入内容 | (0,0,1) |
| `::first-line` / `::first-letter` | 首行/首字母样式 | (0,0,1) |
| `::selection` | 文本选中样式 | (0,0,1) |
| `::placeholder` | 输入框占位符样式 | (0,0,1) |
| `::backdrop` | 全屏元素背景 | (0,0,1) |

### 优先级计算

**权重规则：**

```
优先级从低到高：
(0,0,0)  * / :where()
(0,0,1)  元素选择器 / 伪元素
(0,1,0)  类选择器 / 属性选择器 / 伪类
(1,0,0)  ID 选择器
(内联)   style="" 属性
(终极)   !important
```

**关键规则：**

- 同优先级时，后声明的覆盖先声明的（源码顺序）
- `!important` 覆盖所有优先级（最后手段）
- `:not()` / `:is()` / `:has()` 本身不计权重，参数中的选择器计
- `:where()` 优先级强制为 0
- `@layer` 可将样式降级到低优先级层
- 内联样式 `style=""` 权重高于所有选择器

**优先级计算示例：**

```
*                           → (0,0,0)
li                          → (0,0,1)
ul li                       → (0,0,2)
ul > li + p                 → (0,0,3)
.item                       → (0,1,0)
div.item                    → (0,1,1)
.item:hover                 → (0,2,0)
#nav                        → (1,0,0)
#nav .item                  → (1,1,0)
#nav .item:hover::before    → (1,2,1)
style="color: red"          → 内联，最高
!important                  → 终极覆盖
```

## Why — 为什么

**适用场景：**

- 理解样式覆盖规则，调试 CSS 冲突
- 设计可维护的 CSS 架构（避免 !important 泛滥）
- 编写可预测的组件样式
- 实现主题切换和样式层叠

**对比替代方案：**

| 维度 | CSS 优先级 | CSS-in-JS | Tailwind | CSS Modules |
|------|-----------|-----------|----------|-------------|
| 可预测性 | 中（需理解权重） | 高（作用域隔离） | 高（原子化无冲突） | 高（哈希类名） |
| 调试 | 中 | 中 | 容易 | 容易 |
| 学习成本 | 低 | 中 | 中 | 低 |
| 冲突风险 | 高 | 低 | 低 | 低 |
| 运行时开销 | 无 | 有（部分方案） | 无 | 无 |

**优缺点：**

- ✅ 优点：
  - 原生机制，无额外依赖
  - 精确控制样式层级
  - 浏览器原生支持，性能最优
- ❌ 缺点：
  - 优先级计算容易出错
  - 嵌套选择器权重累积，后期难以覆盖
  - `!important` 滥用导致维护困难
  - 全局作用域易冲突

## How — 怎么用

### 快速上手

```css
/* 优先级从低到高 */
*              {}  /* (0,0,0) */
li             {}  /* (0,0,1) */
ul li          {}  /* (0,0,2) */
.item          {}  /* (0,1,0) */
ul .item       {}  /* (0,1,1) */
#nav           {}  /* (1,0,0) */
#nav .item     {}  /* (1,1,0) */
style="..."        /* 内联，最高 */
!important         /* 终极覆盖 */
```

### 代码示例

**1. `:where()` 降权：**

```css
/* :where() 优先级为 0，方便被覆盖 */
:where(.card) {
    padding: 16px;
    border-radius: 8px;
    background: #f5f5f5;
}

/* 任何选择器都能轻松覆盖 */
.card { padding: 24px; } /* (0,1,0) > (0,0,0) 生效 */

/* 实际应用：基础样式层 */
:where(h1, h2, h3) {
    margin: 0;
    line-height: 1.3;
}
```

**2. `:is()` 简化选择器：**

```css
/* 之前：冗长重复 */
.card h2,
.panel h2,
.modal h2 {
    font-size: 1.5rem;
}

/* 之后：:is() 简化 */
:is(.card, .panel, .modal) h2 {
    font-size: 1.5rem;
}

/* :is() 取参数最高优先级 */
:is(#nav, .list) .item { }  /* 优先级 = (1,1,0)，因为 #nav 是 ID */
```

**3. `:has()` 父选择器：**

```css
/* 只有包含图片的卡片才有特殊样式 */
.card:has(img) {
    padding: 0;
}

/* 表单验证：输入框无效时标签变红 */
label:has(+ input:invalid) {
    color: red;
}

/* 没有子项的菜单隐藏 */
.menu:has(> .item:nth-child(1):not(:only-child)) {
    display: flex;
}

/* 图片悬停时容器效果 */
.card:has(img:hover) {
    transform: scale(1.02);
}
```

**4. `@layer` 分层：**

```css
/* 声明层顺序（先声明的层优先级低） */
@layer reset, base, components, utilities;

/* 重置层（最低优先级） */
@layer reset {
    *, *::before, *::after { box-sizing: border-box; margin: 0; }
    h1, h2, h3 { font-weight: normal; }
}

/* 基础层 */
@layer base {
    a { color: blue; text-decoration: none; }
    body { font-family: system-ui; }
}

/* 组件层 */
@layer components {
    .card { padding: 16px; border-radius: 8px; }
    .card:hover { box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
}

/* 工具层（最高优先级） */
@layer utilities {
    .hidden { display: none; }
    .mt-4 { margin-top: 1rem; }
}

/* 未分层的样式优先级最高（覆盖所有层） */
.card { border: 1px solid #ddd; }
```

**5. BEM 命名避免权重问题：**

```css
/* ❌ 嵌套选择器，权重累积 */
.nav .nav-item .nav-link { color: red; }      /* (0,3,0) */
.nav .nav-item .nav-link:hover { color: blue; } /* (0,4,0) */

/* ✅ BEM：单 class，权重一致 */
.nav__link { color: red; }            /* (0,1,0) */
.nav__link--active { color: blue; }   /* (0,1,0)，同权重后声明覆盖 */

/* ✅ 更完整的 BEM 示例 */
.card { }
.card__header { }
.card__body { }
.card__footer { }
.card--featured { }       /* 修饰符 */
.card__title--lg { }     /* 元素修饰符 */
```

**6. 属性选择器高级用法：**

```css
/* 精确匹配 */
[type="text"] { border: 1px solid #ccc; }

/* 以某值开头 */
[href^="https"] { color: green; }        /* 外部链接 */
[class^="icon-"] { font-family: icons; } /* 图标类 */

/* 以某值结尾 */
[href$=".pdf"] { background: url(pdf-icon.png); }
[src$=".webp"] { }  /* WebP 图片 */

/* 包含某值 */
[class*="col-"] { float: left; }  /* 栅格列 */

/* 大小写不敏感 */
[href="README" i] { }  /* 匹配 readme, README, ReadMe 等 */

/* 组合属性选择器 */
a[href^="https"][target="_blank"]::after {
    content: "↗";
}
```

**7. `:nth-child()` 高级模式：**

```css
/* 基础模式 */
:nth-child(3)        /* 第 3 个 */
:nth-child(odd)      /* 奇数（2n+1） */
:nth-child(even)     /* 偶数（2n） */
:nth-child(3n)       /* 每 3 个 */
:nth-child(3n+1)     /* 第 1, 4, 7... 个 */
:nth-child(-n+3)     /* 前 3 个 */
:nth-child(n+4):nth-child(-n+6)  /* 第 4-6 个 */

/* 实际应用 */
.table-row:nth-child(even) { background: #f9f9f9; } /* 斑马纹 */
.grid-item:nth-child(3n+1) { clear: left; }          /* 每 3 个换行 */
li:last-child { border-bottom: none; }               /* 最后一项无下边框 */

/* :nth-of-type 区别 */
.item:nth-of-type(2)    /* 同类型元素中第 2 个 */
.item:nth-child(2)      /* 所有子元素中第 2 个（如果不是 .item 则不匹配） */
```

**8. CSS Modules 自动隔离：**

```css
/* Button.module.css */
.btn {              /* 编译为 .Button_btn_1a2b3 */
    padding: 8px 16px;
    border-radius: 4px;
}
.primary {          /* 编译为 .Button_primary_4c5d6 */
    background: blue;
    color: white;
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ primary, children }) {
    return (
        <button className={`${styles.btn} ${primary ? styles.primary : ''}`}>
            {children}
        </button>
    );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 样式不生效 | 被更高优先级覆盖 | DevTools 查看被划线的规则，找到覆盖源 |
| !important 泛滥 | 优先级失控 | 用 BEM/CSS Modules 从根本解决，`:where()` 降权 |
| 第三方库样式难覆盖 | 库用了高优先级选择器 | 用更具体选择器或 `:where()` 反向操作 |
| `:not()` 权重计算 | `:not(.a)` 权重是 (0,1,0) | 把 `:not()` 的参数权重加上去 |
| `:nth-child` 不生效 | 选择器类型和位置不匹配 | 用 `:nth-of-type` 替代，确认 HTML 结构 |
| `:has()` 兼容性 | 旧浏览器不支持 | 检查 caniuse，提供降级方案 |
| CSS Modules 和全局样式冲突 | 全局选择器权重更高 | 避免 ID 选择器，统一用 CSS Modules |
| `@layer` 内 !important | !important 在层内反转优先级 | 了解层中 !important 的反转规则：层越低，!important 优先级越高 |

### 最佳实践

- 选择器嵌套不超过 3 层
- 用 BEM / CSS Modules 控制权重在 (0,1,0)
- 永远不要用 `!important`（除了覆盖第三方库）
- 用 `@layer` 管理全局样式层级
- 用 `:where()` 定义可覆盖的基础样式
- 用 `:is()` 简化冗长的选择器列表
- 优先用类选择器，避免 ID 选择器和元素选择器组合
- 使用 CSS Modules 或 Scoped CSS 避免全局冲突

## 面试题

**Q1: CSS优先级如何计算？**
> 优先级按 (ID, Class, Element) 三位权重比较：#id 为 (1,0,0)，.class / 属性选择器 / 伪类为 (0,1,0)，元素选择器 / 伪元素为 (0,0,1)。同权重时后声明覆盖先声明，内联样式优先级最高，!important 覆盖一切。注意这不是十进制，(0,1,0) 优先级高于 (0,0,11)。

**Q2: !important的使用原则是什么？**
> !important 应作为最后手段，仅用于覆盖第三方库样式。滥用会导致样式无法被正常覆盖，形成优先级军备竞赛。应通过 BEM 命名、CSS Modules、@layer 等机制从源头控制权重，而非依赖 !important。在 @layer 中，!important 的优先级规则会反转：越低层的 !important 优先级越高。

**Q3: :where()和:is()的区别是什么？**
> 两者功能相似，都接受选择器列表作为参数。关键区别在优先级：:where() 优先级强制为 0（方便被覆盖），:is() 取参数中最高优先级。:where() 适合基础样式定义（如 CSS Reset），:is() 适合简化选择器写法同时保留权重。两者都不支持 `::before`/`::after` 作为参数。

**Q4: 选择器嵌套过深有什么问题？**
> 嵌套过深导致权重累积（如 `.nav .nav-item .nav-link` 为 (0,3,0)），后期难以覆盖；增加 CSS 文件体积；降低浏览器匹配性能（选择器从右向左匹配，每增加一层多一轮遍历）；增加维护成本。建议嵌套不超过 3 层，用 BEM 单 class 方案控制权重。

**Q5: :has() 选择器有什么用？以前 CSS 能实现类似功能吗？**
> `:has()` 被称为"CSS 父选择器"，允许根据子元素/后续元素的状态来选择父元素/前驱元素。例如 `.card:has(img)` 选中包含图片的卡片。以前 CSS 无法实现，只能通过 JS 添加类名。常见用法：表单标签根据输入状态变色（`label:has(+ input:invalid)`）、空状态隐藏（`.menu:has(> :empty)`）、悬停联动（`.card:has(img:hover)`）。

**Q6: @layer 是什么？解决了什么问题？**
> `@layer` 是 CSS 层叠层（Cascade Layers），允许开发者显式声明样式层的优先级顺序，解决样式冲突不可控的问题。先声明的层优先级低，后声明的高，未分层的样式优先级最高。这让 CSS 架构可以按 reset → base → components → utilities 的层级组织，即使 utilities 使用低优先级选择器也能覆盖 components 的高优先级选择器，从根源上避免优先级军备竞赛。

**Q7: CSS Modules 是如何解决样式冲突的？**
> CSS Modules 在构建时将类名编译为唯一的哈希字符串（如 `.btn` → `.Button_btn_1a2b3`），确保每个组件的样式不会影响其他组件。它本质上是自动化的 BEM，不需要手动命名约定。好处：零运行时开销、完全隔离、可和 TypeScript 配合类型检查。局限：动态样式需要用 :global 或 CSS Variables 辅助。

**Q8: 属性选择器有哪些匹配模式？实际开发中怎么用？**
> 属性选择器有 7 种匹配模式：`[attr]`（存在）、`[attr=val]`（精确）、`[attr^=val]`（开头）、`[attr$=val]`（结尾）、`[attr*=val]`（包含）、`[attr~=val]`（词列表中含）、`[attr|=val]`（值或值-开头）。实战用途：`[type="email"]` 自定义邮箱输入框样式；`[href^="https"]` 区分外部链接；`[href$=".pdf"]` 添加文件类型图标；`[data-theme="dark"]` 主题切换；`[aria-disabled="true"]` 无障碍样式。加 `i` 标志可大小写不敏感。

---

**相关链接：**
- [[CSS布局Flex与Grid]]
- [[CSS动画与过渡]]
- [[CSS架构方法论]]
- [[Tailwind CSS与原子化CSS]]

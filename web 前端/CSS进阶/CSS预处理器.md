---
tags:
  - Web前端
  - CSS进阶
  - CSS预处理器
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# CSS预处理器

## What — 是什么

> CSS 预处理器是一种 CSS 的扩展语言，它在 CSS 基础上增加了变量、嵌套、混入、函数、模块化等编程能力，编写后通过编译器输出标准 CSS，让样式开发具备工程化能力。

**核心概念：**

- **预处理语言**：在 CSS 语法之上扩展了编程特性，源码不能被浏览器直接识别，需编译为标准 CSS
- **编译器**：将预处理器语法翻译为标准 CSS 的工具（如 Dart Sass、Less.js）
- **变量系统**：用符号声明可复用的值，一处修改全局生效
- **嵌套规则**：让选择器层级关系与 DOM 结构对应，消除重复书写
- **混入（Mixin）**：可复用的样式片段，支持参数化，类似函数
- **模块化机制**：将样式拆分为多个文件，按需引入，避免单文件臃肿

**关键特性：**

- 变量与数据类型（数字、字符串、颜色、布尔值、列表、Map）
- 选择器嵌套与父选择器引用 `&`
- 混入（Mixin）与继承（Extend）
- 控制流：条件判断 `@if`、循环 `@for` / `@each` / `@while`
- 丰富的内置函数（颜色运算、字符串处理、数学计算、Map/List 操作）
- 模块系统 `@use` / `@forward`（取代 `@import`）
- 占位符选择器 `%placeholder`（配合 `@extend` 减少冗余输出）

---

## Why — 为什么

原生 CSS 缺少变量、嵌套、混入、函数等编程能力，导致大型项目中样式难以维护：重复值散落各处、选择器层级靠人工缩进、无法抽象公共样式、媒体查询反复书写。预处理器通过引入编程能力弥补这些缺陷。

**适用场景：**

- 中大型项目需要统一管理颜色、字号、间距等设计令牌
- 组件化开发中需要按模块拆分样式文件
- 主题切换、响应式布局等需要大量复用样式片段
- 团队协作需要一致的样式编写规范
- 需要自动生成前缀、计算值等工具函数

**对比替代方案：**

| 维度 | Sass/SCSS | Less | Stylus |
|------|-----------|------|--------|
| 语法 | SCSS（兼容CSS）+ 缩进语法 | 兼容CSS语法 | 灵活（缩进/括号/省略冒号分号） |
| 编译方式 | Dart Sass（Dart，官方推荐） | Less.js（JavaScript） | Stylus（Node.js） |
| 变量符号 | `$var` | `@var` | 直接写变量名或 `$var` |
| 生态 | 最成熟，Compass/Bourbon/社区庞大 | 较成熟，Bootstrap 4 及之前使用 | 较小众，Expressive 社区 |
| 功能丰富度 | 最强（Map、@use、@forward、!default） | 中等（无 Map、模块系统较弱） | 强（函数式风格、灵活语法） |
| 使用率 | 最高，行业事实标准 | 较高（Bootstrap 5 已转SCSS） | 较低 |
| 混入 | `@mixin` / `@include` | `.mixin()` 直接调用 | 函数即混入 |
| 继承 | `@extend` + `%placeholder` | `:extend()` 伪类 | `@extend` |
| 条件循环 | `@if` / `@for` / `@each` / `@while` | When 守卫 + 递归 | `if` / `for` / `for ... in` |
| 模块化 | `@use` / `@forward`（新） | `@import`（旧） | `@import` / `@require` |
| 学习曲线 | 中等（功能多但SCSS语法接近CSS） | 低（接近CSS，JS开发者友好） | 中等（语法灵活但风格独特） |

**优缺点：**

- ✅ 优点：
  - 变量统一管理设计令牌，一处修改全局生效
  - 嵌套让选择器层级清晰，减少重复书写
  - 混入和继承大幅提升样式复用率
  - 函数和循环可以程序化生成样式（如生成工具类）
  - 模块化拆分让大型项目样式可维护
  - 编译产物是标准 CSS，兼容所有浏览器
- ❌ 缺点：
  - 引入编译环节，增加构建复杂度
  - 调试时看到的行号是编译后 CSS，需 source map 辅助
  - 过度使用嵌套会导致选择器优先级膨胀
  - `@extend` 可能产生意料之外的选择器合并
  - 变量是编译时确定的，无法运行时动态切换（需 CSS 变量补充）

---

## How — 怎么用

### 快速上手

**安装 Dart Sass（官方推荐编译器）：**

```bash
# npm
npm install -g sass

# 监听文件变化自动编译
sass --watch src/styles/main.scss dist/styles/main.css

# 指定输出格式（compressed 压缩）
sass src/styles/main.scss dist/styles/main.css --style compressed
```

**安装 Less：**

```bash
npm install -g less
lessc src/styles/main.less dist/styles/main.css
```

**最小示例：**

```scss
// variables.scss
$primary-color: #3498db;
$font-size-base: 16px;

// main.scss
@use 'variables' as *;

.button {
  font-size: $font-size-base;
  background: $primary-color;
  color: #fff;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;

  &:hover {
    background: darken($primary-color, 10%);
  }
}
```

编译输出：

```css
.button {
  font-size: 16px;
  background: #3498db;
  color: #fff;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
}
.button:hover {
  background: #2176ae;
}
```

### 1. 变量

**Sass 变量：**

```scss
// 基本变量
$font-stack: 'Helvetica Neue', Arial, sans-serif;
$primary: #3498db;
$spacing-unit: 8px;
$border-radius: 4px;
$header-height: 64px;

// 变量作用域
$global-var: 'I am global';

.container {
  $local-var: 'I am local'; // 局部变量，仅在此块内有效
  content: $local-var;
}

// !default — 默认值，仅在变量未定义或为 null 时生效
$brand-color: #3498db !default; // 如果 $brand-color 已定义则忽略
$brand-color: #e74c3c; // 此赋值先生效，上面的 !default 被忽略

// !global — 将局部变量提升为全局
.sidebar {
  $sidebar-width: 280px !global;
}
```

**Less 变量：**

```less
// Less 用 @ 符号
@primary: #3498db;
@font-size-base: 16px;
@spacing: 8px;

// Less 变量延迟求值（整个作用域内最后赋值生效）
@var: 0;
.class1 {
  @var: 1;
  .class2 {
    @var: 2;
    three: @var; // 输出 3，因为下面还有 @var: 3
    @var: 3;
  }
  one: @var; // 输出 1
}
```

**变量的实际应用 — 设计令牌管理：**

```scss
// _tokens.scss
$colors: (
  'primary':    #3498db,
  'secondary':  #2ecc71,
  'accent':     #e74c3c,
  'background': #f5f6fa,
  'surface':    #ffffff,
  'text':       #2c3e50,
  'text-light': #7f8c8d,
);

$font-sizes: (
  'xs':  12px,
  'sm':  14px,
  'base': 16px,
  'lg':  18px,
  'xl':  20px,
  '2xl': 24px,
  '3xl': 30px,
  '4xl': 36px,
);

$spacing: (
  0: 0,
  1: 4px,
  2: 8px,
  3: 12px,
  4: 16px,
  5: 20px,
  6: 24px,
  8: 32px,
  10: 40px,
  12: 48px,
);
```

### 2. 嵌套规则

```scss
// 基本嵌套
.nav {
  background: #fff;
  padding: 16px;

  .nav-item {
    display: inline-block;
    margin-right: 12px;

    a {
      color: #333;
      text-decoration: none;

      &:hover { // & 代表父选择器 .nav .nav-item a
        color: #3498db;
      }

      &.active { // &.active => a.active
        color: #3498db;
        font-weight: bold;
      }
    }
  }
}

// & 的高级用法
.button {
  color: blue;

  // & 拼接前缀
  &--primary { color: white; }     // .button--primary
  &--danger  { color: red; }       // .button--danger
  &--large   { font-size: 24px; }  // .button--large

  // & 拼接修饰符状态
  &.is-disabled { opacity: 0.5; }  // .button.is-disabled

  // & 嵌套到父级前面
  .no-js & { display: block; }     // .no-js .button

  // & 用于伪元素
  &::before {
    content: '';
    display: inline-block;
  }
}

// BEM 配合嵌套
.block {
  &__element {
    color: #333;

    &--modifier {
      color: #3498db;
      font-weight: bold;
    }
  }
}
// 输出: .block__element { }  .block__element--modifier { }
```

**嵌套媒体查询：**

```scss
.container {
  width: 100%;
  padding: 16px;

  @media (min-width: 768px) {
    width: 720px;
    padding: 24px;
  }

  @media (min-width: 1024px) {
    width: 960px;
    padding: 32px;
  }
}

// 嵌套的层数警告 — 不要超过 3-4 层
// ❌ 过度嵌套
.page {
  .content {
    .article {
      .body {
        .paragraph {
          span { // 选择器过长，优先级过高
            color: red;
          }
        }
      }
    }
  }
}
```

### 3. 混入（Mixin）

**基本用法：**

```scss
// 定义
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

// 使用
.hero {
  @include flex-center;
  height: 400px;
}
```

**参数与默认参数：**

```scss
@mixin button($bg: #3498db, $color: #fff, $radius: 4px, $padding: 8px 16px) {
  background: $bg;
  color: $color;
  border: none;
  border-radius: $radius;
  padding: $padding;
  cursor: pointer;
  transition: background 0.2s;

  &:hover {
    background: darken($bg, 10%);
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.btn-primary {
  @include button; // 使用所有默认值
}

.btn-danger {
  @include button($bg: #e74c3c); // 命名参数，其他用默认值
}

.btn-outline {
  @include button($bg: transparent, $color: #3498db, $radius: 20px);
}
```

**@content 内容块：**

```scss
// 封装媒体查询
@mixin respond-to($breakpoint) {
  $breakpoints: (
    'sm': 576px,
    'md': 768px,
    'lg': 1024px,
    'xl': 1280px,
  );

  @media (min-width: map-get($breakpoints, $breakpoint)) {
    @content;
  }
}

.sidebar {
  width: 100%;

  @include respond-to('md') {
    width: 280px;
    float: left;
  }

  @include respond-to('lg') {
    width: 320px;
  }
}

// 封装 hover 效果（支持无障碍）
@mixin hover-supported {
  @media (hover: hover) {
    &:hover {
      @content;
    }
  }
}

.card {
  @include hover-supported {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    transform: translateY(-2px);
  }
}
```

**复杂 Mixin 示例 — 三角形生成器：**

```scss
@mixin triangle($direction, $size, $color) {
  width: 0;
  height: 0;
  border-style: solid;

  @if $direction == 'up' {
    border-width: 0 $size $size $size;
    border-color: transparent transparent $color transparent;
  } @else if $direction == 'down' {
    border-width: $size $size 0 $size;
    border-color: $color transparent transparent transparent;
  } @else if $direction == 'left' {
    border-width: $size $size $size 0;
    border-color: transparent $color transparent transparent;
  } @else if $direction == 'right' {
    border-width: $size 0 $size $size;
    border-color: transparent transparent transparent $color;
  }
}

.tooltip::after {
  @include triangle('up', 6px, #333);
}
```

**Less 混入：**

```less
// Less 混入直接用类选择器定义
.flex-center() {  // () 表示不输出到 CSS
  display: flex;
  justify-content: center;
  align-items: center;
}

.hero {
  .flex-center(); // 调用
  height: 400px;
}

// 带参数
.button(@bg: #3498db; @color: #fff) {
  background: @bg;
  color: @color;
  border: none;
  padding: 8px 16px;
}

.btn-primary { .button(); }
.btn-danger  { .button(@bg: #e74c3c); }
```

### 4. 继承 @extend

```scss
// 基础样式
%button-base { // % 占位符选择器，不会单独输出到 CSS
  display: inline-block;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  text-align: center;
  transition: all 0.2s;
}

.btn-primary {
  @extend %button-base;
  background: #3498db;
  color: #fff;
}

.btn-secondary {
  @extend %button-base;
  background: #95a5a6;
  color: #fff;
}

// 编译输出 — 选择器合并，减少重复
// .btn-primary, .btn-secondary {
//   display: inline-block; padding: 8px 16px; ...
// }
// .btn-primary { background: #3498db; color: #fff; }
// .btn-secondary { background: #95a5a6; color: #fff; }
```

**@extend vs @mixin 的区别：**

```scss
// @mixin — 复制样式到每个调用处
@mixin clearfix {
  &::after {
    content: '';
    display: table;
    clear: both;
  }
}

.card   { @include clearfix; } // .card::after { ... }
.panel  { @include clearfix; } // .panel::after { ... }  重复输出

// @extend — 合并选择器，共享样式声明
%clearfix {
  &::after {
    content: '';
    display: table;
    clear: both;
  }
}

.card   { @extend %clearfix; }
.panel  { @extend %clearfix; }
// .card, .panel { &::after { ... } }  选择器合并，不重复

// 选择建议：
// - 无参数的复用样式片段 → @extend（减少CSS体积）
// - 需要参数化 → @mixin（灵活但会重复输出）
// - 需要传入 @content → @mixin
```

### 5. 条件与循环

**@if 条件判断：**

```scss
@mixin theme-color($theme) {
  @if $theme == 'dark' {
    background: #1a1a2e;
    color: #e0e0e0;
  } @else if $theme == 'light' {
    background: #ffffff;
    color: #333333;
  } @else {
    background: #f5f5f5;
    color: #666666;
  }
}

.dark-mode  { @include theme-color('dark'); }
.light-mode { @include theme-color('light'); }
```

**@for 循环：**

```scss
// 生成间距工具类
@for $i from 1 through 10 {
  .mt-#{$i} { margin-top: #{$i * 4}px; }
  .mb-#{$i} { margin-bottom: #{$i * 4}px; }
  .ml-#{$i} { margin-left: #{$i * 4}px; }
  .mr-#{$i} { margin-right: #{$i * 4}px; }
  .p-#{$i}  { padding: #{$i * 4}px; }
}

// 输出: .mt-1 { margin-top: 4px; }  .mt-2 { margin-top: 8px; } ...
```

**@each 循环：**

```scss
// 遍历颜色 Map 生成工具类
$theme-colors: (
  'primary':   #3498db,
  'success':   #2ecc71,
  'warning':   #f39c12,
  'danger':    #e74c3c,
  'info':      #1abc9c,
);

@each $name, $color in $theme-colors {
  .text-#{$name} { color: $color; }
  .bg-#{$name}   { background-color: $color; }
  .border-#{$name} { border-color: $color; }

  .btn-#{$name} {
    background: $color;
    color: #fff;
    &:hover { background: darken($color, 10%); }
  }
}

// 遍历列表
$sizes: 'sm', 'md', 'lg', 'xl';
@each $size in $sizes {
  .icon-#{$size} {
    @if $size == 'sm'  { width: 16px; height: 16px; }
    @if $size == 'md'  { width: 24px; height: 24px; }
    @if $size == 'lg'  { width: 32px; height: 32px; }
    @if $size == 'xl'  { width: 48px; height: 48px; }
  }
}
```

**@while 循环：**

```scss
$col-count: 12;
$i: 1;
@while $i <= $col-count {
  .col-#{$i} {
    width: percentage($i / $col-count);
  }
  $i: $i + 1;
}
```

**Less 的条件与循环：**

```less
// Less 用 when 守卫实现条件
.mixin(@color) when (lightness(@color) >= 50%) {
  color: black; // 浅色背景用深色文字
}
.mixin(@color) when (lightness(@color) < 50%) {
  color: white; // 深色背景用浅色文字
}

.button-danger { .mixin(#e74c3c); } // 输出 color: white

// Less 用递归实现循环
.generate-spacing(@n, @i: 1) when (@i =< @n) {
  .mt-@{i} { margin-top: (@i * 4px); }
  .generate-spacing(@n, (@i + 1));
}

.generate-spacing(10);
```

### 6. 内置函数

**颜色函数：**

```scss
$base: #3498db;

// 调整明暗
darken($base, 20%);   // #1a5276  变暗20%
lighten($base, 20%);  // #6cb4e8  变亮20%

// 调整饱和度
saturate($base, 20%);   // 增加饱和度
desaturate($base, 20%); // 降低饱和度

// 调整色相
adjust-hue($base, 30deg); // 色相偏移30度

// 透明度
rgba($base, 0.5);       // 添加透明度
opacify(rgba($base, 0.5), 0.3); // 增加不透明度
transparentize($base, 0.3);      // 增加透明度

// 混合颜色
mix(#3498db, #e74c3c); // 混合两种颜色，默认50%/50%
mix(#3498db, #e74c3c, 30%); // 30%第一种 + 70%第二种

// 取反
complement($base);    // 互补色
invert($base);        // 反色

// 获取颜色分量
red($base);     // 52
green($base);   // 152
blue($base);    // 219
hue($base);     // 204deg
saturation($base); // 69.9%
lightness($base);  // 52.9%
```

**字符串函数：**

```scss
$str: "hello world";

to-upper-case($str);  // "HELLO WORLD"
to-lower-case($str);  // "hello world"
str-length($str);     // 11
str-index($str, "world"); // 7
str-insert($str, " beautiful", 6); // "hello beautiful world"
str-slice($str, 1, 5); // "hello"

// 插值 #{} — 变量嵌入字符串
$name: "primary";
.class-#{$name} { color: #3498db; } // .class-primary { ... }
```

**数学函数：**

```scss
// 基本运算
10px + 20px;   // 30px
10px * 2;      // 20px（乘法只能一个带单位）
(100px / 2);   // 50px（除法需括号或变量参与）

// 数学函数
abs(-15px);        // 15px
ceil(4.2px);       // 5px
floor(4.8px);      // 4px
round(4.5px);      // 5px
percentage(0.75);  // 75%
max(10px, 20px);   // 20px
min(10px, 20px);   // 10px
random();          // 0-1 随机数
random(10);        // 1-10 随机整数
```

**Map 和 List 操作：**

```scss
// Map 操作
$breakpoints: (
  'sm': 576px,
  'md': 768px,
  'lg': 1024px,
  'xl': 1280px,
);

map-get($breakpoints, 'md');    // 768px
map-has-key($breakpoints, 'xxl'); // false
map-keys($breakpoints);   // ('sm', 'md', 'lg', 'xl')
map-values($breakpoints); // (576px, 768px, 1024px, 1280px)
map-merge($breakpoints, ('xxl': 1536px)); // 合并

// List 操作
$list: 10px 20px 30px;

length($list);      // 3
nth($list, 2);      // 20px（索引从1开始）
index($list, 20px); // 2
append($list, 40px); // (10px 20px 30px 40px)
join(10px 20px, 30px 40px); // (10px 20px 30px 40px)
```

**自定义函数：**

```scss
@function px-to-rem($px, $base: 16px) {
  @return ($px / $base) * 1rem;
}

@function z-index($layer) {
  $z-layers: (
    'default': 1,
    'dropdown': 10,
    'sticky':   50,
    'modal':    100,
    'popover':  200,
    'tooltip':  300,
    'toast':    400,
  );
  @return map-get($z-layers, $layer);
}

// 使用
h1 { font-size: px-to-rem(32px); }      // 2rem
.modal { z-index: z-index('modal'); }    // 100
.tooltip { z-index: z-index('tooltip'); } // 300
```

### 7. 模块化 @use / @forward

**@use — 替代 @import 的现代模块系统：**

```scss
// _colors.scss
$primary: #3498db;
$secondary: #2ecc71;
$danger: #e74c3c;

// _typography.scss
@use 'colors' as c; // 命名空间 c

$font-base: 16px;
$heading-color: c.$primary; // 通过命名空间访问

// _mixins.scss
@use 'colors' as c;

@mixin button-base($bg: c.$primary) {
  background: $bg;
  color: #fff;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
}

// main.scss
@use 'colors';       // 默认命名空间 colors
@use 'mixins' as m;  // 自定义命名空间 m

.btn {
  @include m.button-base(colors.$primary);
}
```

**@use vs @import 的关键区别：**

```scss
// ❌ @import（已废弃）— 全局污染
@import 'colors';  // 所有变量、mixin 进入全局命名空间
@import 'colors';  // 可以重复导入，文件会被多次编译

// ✅ @use — 模块化，命名空间隔离
@use 'colors';     // 变量通过 colors.$primary 访问
@use 'colors';     // 多次 @use 只加载一次（单例模式）
@use 'colors' as *; // 无命名空间（但不推荐，失去隔离优势）
@use 'colors' as c; // 自定义命名空间 c.$primary
```

**@forward — 转发模块：**

```scss
// _foundation.scss — 统一转发子模块
@forward 'colors';
@forward 'typography';
@forward 'spacing';
@forward 'mixins';

// 使用者只需引入一个文件
// main.scss
@use 'foundation' as f;

.btn {
  background: f.$primary;
  @include f.button-base;
}

// @forward 控制导出范围
@forward 'colors' hide $internal-color; // 隐藏内部变量
@forward 'colors' show $primary, $secondary; // 只导出指定成员
```

**典型项目目录结构：**

```
styles/
├── main.scss            # 入口文件，@use 各模块
├── _foundation/
│   ├── _index.scss      # @forward 所有子模块
│   ├── _colors.scss     # 颜色变量
│   ├── _typography.scss # 排版变量
│   ├── _spacing.scss    # 间距变量
│   └── _breakpoints.scss
├── _mixins/
│   ├── _index.scss
│   ├── _responsive.scss # 响应式 mixin
│   ├── _layout.scss     # 布局 mixin
│   └── _utilities.scss  # 工具 mixin
├── _functions/
│   ├── _index.scss
│   ├── _math.scss       # 数学函数
│   └── _color.scss      # 颜色函数
├── base/
│   ├── _reset.scss
│   └── _base.scss
├── components/
│   ├── _button.scss
│   ├── _card.scss
│   └── _modal.scss
└── utilities/
    ├── _spacing.scss
    └── _text.scss
```

### 8. Less 特有用法

**变量插值（Variable Interpolation）：**

```less
// 属性名插值
@prop: color;
.my-class {
  @{prop}: #3498db;  // 等同于 color: #3498db
}

// 选择器插值
@selector: banner;
.@{selector} {
  font-size: 24px;
}

// URL 插值
@images: "../img";
.logo {
  background: url("@{images}/logo.png");
}

// import 语句插值
@theme: "dark";
@import "@{theme}/variables.less";
```

**when 条件守卫：**

```less
// 类型检查守卫
.mixin(@val) when (iscolor(@val)) {
  color: @val;
}
.mixin(@val) when (isnumber(@val)) {
  width: @val;
}
.mixin(@val) when (isstring(@val)) {
  content: @val;
}

.test {
  .mixin(#3498db);  // 触发 iscolor
  .mixin(100px);    // 触发 isnumber
  .mixin("hello");  // 触发 isstring
}

// 组合条件
.mixin(@a) when (@a > 10) and (@a < 100) { ... }
.mixin(@a) when (@a = 10), (@a = 20) { ... }  // 逗号表示 or
.mixin(@a) when not (@a = 0) { ... }
```

**Less 递归实现循环：**

```less
// 生成网格列
.generate-columns(@n, @i: 1) when (@i =< @n) {
  .col-@{i} {
    width: (@i * 100% / @n);
  }
  .generate-columns(@n, (@i + 1));
}

.generate-columns(12);
// .col-1 { width: 8.3333% } .col-2 { width: 16.6667% } ...
```

### 9. 实战案例

**实战一：主题系统**

```scss
// _themes.scss
$themes: (
  'light': (
    'bg-primary':   #ffffff,
    'bg-secondary': #f5f6fa,
    'text-primary': #2c3e50,
    'text-secondary': #7f8c8d,
    'border':       #dcdde1,
    'accent':       #3498db,
  ),
  'dark': (
    'bg-primary':   #1a1a2e,
    'bg-secondary': #16213e,
    'text-primary': #e0e0e0,
    'text-secondary': #a0a0a0,
    'border':       #2d2d44,
    'accent':       #4da6ff,
  ),
);

// 主题读取函数
@function theme-value($theme, $key) {
  @return map-get(map-get($themes, $theme), $key);
}

// 主题 Mixin
@mixin themed {
  @each $theme-name, $theme-map in $themes {
    $theme-selector: if($theme-name == 'light', ':root', '[data-theme="#{$theme-name}"]');
    #{$theme-selector} & {
      @content($theme-map);
    }
  }
}

// 使用
.card {
  @include themed using ($theme) {
    background: map-get($theme, 'bg-primary');
    color: map-get($theme, 'text-primary');
    border: 1px solid map-get($theme, 'border');
  }
}

// 编译输出：
// :root .card { background: #fff; color: #2c3e50; border: 1px solid #dcdde1; }
// [data-theme="dark"] .card { background: #1a1a2e; color: #e0e0e0; border: 1px solid #2d2d44; }
```

**实战二：响应式 Mixin 库**

```scss
// _responsive.scss
$breakpoints: (
  'xs': 0,
  'sm': 576px,
  'md': 768px,
  'lg': 1024px,
  'xl': 1280px,
  'xxl': 1536px,
) !default;

// min-width 媒体查询
@mixin above($bp) {
  $value: map-get($breakpoints, $bp);
  @if $value {
    @media (min-width: $value) { @content; }
  } @else {
    @warn "Breakpoint `#{$bp}` not found.";
  }
}

// max-width 媒体查询
@mixin below($bp) {
  $value: map-get($breakpoints, $bp);
  @if $value {
    @media (max-width: $value - 1px) { @content; }
  }
}

// 区间媒体查询
@mixin between($lower, $upper) {
  $min: map-get($breakpoints, $lower);
  $max: map-get($breakpoints, $upper);
  @media (min-width: $min) and (max-width: $max - 1px) { @content; }
}

// 使用
.grid {
  display: grid;
  gap: 16px;
  grid-template-columns: 1fr;

  @include above('sm') { grid-template-columns: repeat(2, 1fr); }
  @include above('md') { grid-template-columns: repeat(3, 1fr); }
  @include above('lg') { grid-template-columns: repeat(4, 1fr); }
}
```

**实战三：工具函数库**

```scss
// _functions.scss

// px 转 rem
@function rem($px, $base: 16px) {
  @return math.div($px, $base) * 1rem; // Dart Sass 用 math.div
}

// 行高计算
@function line-height($font-size, $ratio: 1.5) {
  @return $font-size * $ratio;
}

// z-index 管理
$z-layers: (
  'default': 1,
  'dropdown': 100,
  'sticky':   200,
  'overlay':  300,
  'modal':    400,
  'popover':  500,
  'tooltip':  600,
  'toast':    700,
) !default;

@function z($layer) {
  @if not map-has-key($z-layers, $layer) {
    @warn "No z-index layer found for `#{$layer}`.";
    @return 1;
  }
  @return map-get($z-layers, $layer);
}

// 灰度生成
@function gray($level) {
  @return rgb($level, $level, $level);
}

// 阴影生成
@function shadow($level: 1) {
  $shadows: (
    1: 0 1px 3px rgba(0,0,0,0.12),
    2: 0 3px 6px rgba(0,0,0,0.16),
    3: 0 10px 20px rgba(0,0,0,0.19),
    4: 0 14px 28px rgba(0,0,0,0.25),
    5: 0 19px 38px rgba(0,0,0,0.30),
  );
  @return map-get($shadows, $level);
}

// 使用
h1 {
  font-size: rem(32px);
  line-height: line-height(32px, 1.3);
}

.modal {
  z-index: z('modal');
  box-shadow: shadow(3);
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `@import` 全局污染 | 所有导入的变量/mixin 进入全局命名空间，可能冲突 | 使用 `@use` 替代，通过命名空间隔离 |
| `@extend` 选择器膨胀 | `@extend` 会合并所有相关选择器，导致编译后 CSS 选择器列表极长 | 无参数复用用 `%placeholder`；需参数时用 `@mixin` |
| 编译性能慢 | 大量 `@import` 和复杂计算导致编译耗时 | 迁移到 `@use`（单例加载）；减少嵌套层级；缓存编译结果 |
| 预处理器变量 vs CSS 变量如何选 | 预处理器变量编译时确定，无法运行时切换；CSS 变量运行时可变 | 设计令牌用预处理器变量保证编译时安全；主题切换用 CSS 变量实现运行时动态 |
| `@import` 重复加载 | 同一文件被多次 `@import` 会重复编译输出 | 改用 `@use`（只加载一次）；或用 `@import` 前检查是否已导入 |
| 除法报错 | Dart Sass 2.0 废弃 `/` 做除法 | 使用 `math.div()` 函数替代，需 `@use 'sass:math'` |
| 嵌套过深 | 选择器优先级过高，难以覆盖 | 限制嵌套不超过 3-4 层；使用 BEM 扁平化选择器 |
| 调试困难 | 编译后 CSS 行号与源码不对应 | 开启 source map（`sass --source-map`）；DevTools 中映射到源文件 |

### 最佳实践

- 优先使用 `@use` / `@forward` 替代已废弃的 `@import`，利用命名空间避免全局污染
- 变量命名遵循分类前缀：`$color-primary`、`$spacing-md`、`$font-size-lg`，方便检索和分组
- 嵌套层级控制在 3-4 层以内，避免选择器优先级过高和编译产物膨胀
- 无参数的样式复用用 `@extend %placeholder`，需要参数化用 `@mixin`
- 设计令牌（颜色、字号、间距）集中管理在独立文件中，用 Map 结构组织
- 善用 `!default` 让变量可被覆盖，便于主题定制和第三方扩展
- 自定义函数封装常用计算（rem 转换、z-index 管理、阴影生成），保持业务样式代码简洁
- 预处理器变量用于编译时确定的静态值，CSS 变量用于运行时动态切换（如主题）
- 文件命名以 `_` 开头表示局部文件（partial），不会被单独编译
- 项目初期搭建好模块目录结构（foundation / mixins / functions / components / utilities），避免后期重构

---

## 面试题

**Q1: Sass 和 Less 的核心区别是什么？**
> 1) 变量符号：Sass 用 `$`，Less 用 `@`；2) 编译器：Sass 用 Dart Sass（Dart 实现），Less 用 Less.js（JavaScript 实现）；3) 功能：Sass 支持 Map 数据类型、`@use`/`@forward` 模块系统、`@content` 内容块、`!default` 默认值，Less 不支持或需要变通实现；4) 混入语法：Sass 用 `@mixin`/`@include` 关键字，Less 用类选择器直接调用；5) 循环：Sass 有 `@for`/`@each`/`@while`，Less 靠递归实现；6) 生态：Sass 生态更大，是行业事实标准。

**Q2: @mixin 和 @extend 有什么区别？如何选择？**
> 核心区别在于编译输出方式不同。`@mixin` 将样式复制到每个调用处（重复输出声明），`@extend` 将选择器合并到一起共享声明（不重复输出）。选择依据：1) 需要传参数 → `@mixin`；2) 需要传 `@content` → `@mixin`；3) 无参数的纯样式复用 → `@extend %placeholder`（减少 CSS 体积）；4) 注意 `@extend` 会影响继承链上所有选择器，可能产生意料之外的合并，复杂场景优先 `@mixin`。

**Q3: @use 和 @import 有什么区别？为什么要迁移？**
> 1) 命名空间：`@use` 通过命名空间隔离成员（`colors.$primary`），`@import` 全部注入全局；2) 单例加载：`@use` 同一模块只加载一次，`@import` 每次都执行（可能重复输出 CSS）；3) 私有成员：`@use` 中以 `-` 或 `_` 开头的变量/mixin 是私有的，外部不可访问；4) `@import` 已被 Sass 官方废弃，将在未来版本移除。迁移方法：用 `@use` 替代 `@import`，用 `@forward` 统一转发子模块，加命名空间访问成员。

**Q4: Sass 有哪些数据类型？**
> 7 种数据类型：1) Numbers（数字）：`16px`、`0.8`，可带单位；2) Strings（字符串）：`"Helvetica"` 或 `sans-serif`（有引号和无引号）；3) Colors（颜色）：`#3498db`、`red`、`rgba(0,0,0,0.5)`；4) Booleans（布尔值）：`true`、`false`；5) Null：`null`（表示空值，`!default` 依赖 null 判断）；6) Lists（列表）：`10px 20px 30px`（空格或逗号分隔，类似数组）；7) Maps（映射）：`('key': 'value')`（键值对集合，类似对象）。此外函数 `calc()` 返回的是特殊的 Calculation 类型。

**Q5: 如何用 Sass 实现主题切换？**
> 方案一：Map + `@each` 循环，为每个主题生成 `[data-theme="xxx"]` 选择器下的样式。方案二：Mixin 接收主题参数，遍历主题 Map 输出对应选择器。方案三（推荐）：预处理器变量定义静态设计令牌，CSS 变量实现运行时切换。通过 `:root` 定义默认 CSS 变量值，`[data-theme="dark"]` 覆盖变量，JS 切换 `data-theme` 属性即可。预处理器负责编译时计算和生成变量声明，CSS 变量负责运行时响应切换。

**Q6: CSS 预处理器变量和 CSS 自定义属性（CSS 变量）如何选择？**
> 两者不是替代关系而是互补。预处理器变量：编译时确定，无法运行时修改，用于设计令牌管理、数学计算、条件逻辑；CSS 变量：运行时可变，可被 JS 动态修改，支持继承和层叠，用于主题切换、响应式值变化、组件状态样式。最佳实践：用预处理器变量组织设计令牌并生成 CSS 变量声明，运行时切换用 CSS 变量。例如：`$colors` Map 遍历生成 `--color-primary` 等 CSS 变量，主题切换只需覆盖 CSS 变量。

**Q7: Sass 中 `!default` 的作用是什么？**
> `!default` 是 Sass 变量的默认值标志，含义是"如果此变量未定义或值为 `null`，则赋值；否则保留已有值"。典型用途：1) 编写可配置的库/框架，用户可在引入前自定义变量；2) `_variables.scss` 中定义默认值，项目层覆盖特定变量；3) 结合 `@use ... with (...)` 语法，在引入模块时传入配置。例如 Bootstrap 的源码大量使用 `!default`，用户可以在引入前重新赋值来自定义主题。

**Q8: CSS 预处理器的发展趋势是什么？**
> 1) CSS 原生能力增强：CSS 嵌套（Nesting）、CSS 自定义属性、`@layer` 层叠控制、容器查询等特性逐步普及，部分预处理器功能被原生替代；2) `@use` 模块化：Sass 废弃 `@import` 推向模块系统，代码组织更规范；3) 与 PostCSS 结合：预处理器负责逻辑生成，PostCSS 负责 autoprefixer 等后处理；4) 原子化 CSS 冲击：Tailwind / UnoCSS 等方案减少了对预处理器混入/函数的依赖；5) 预处理器不会消亡：CSS 原生仍不支持条件、循环、Map 操作、函数定义，复杂样式工程化仍需预处理器；6) 最佳实践是预处理器 + CSS 变量 + PostCSS 三者配合。

---

**相关链接：**
- [[CSS变量与主题系统]]
- [[Tailwind CSS与原子化CSS]]

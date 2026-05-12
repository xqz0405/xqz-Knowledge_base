---
tags:
  - Web前端
  - 工程化
  - Babel
  - AST
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Babel与AST

## What — 是什么

> Babel 是 JavaScript 编译器，将新语法/JSX/TypeScript 转换为低版本 JS。AST（抽象语法树）是代码的结构化表示，Babel 和大多数代码工具都基于 AST 工作。

### Babel 的完整处理流程

Babel 的编译过程分为三个阶段，每个阶段由不同的核心包负责：

```
源代码 ──→ Parse（解析）──→ AST ──→ Transform（转换）──→ 新 AST ──→ Generate（生成）──→ 目标代码
              ↑                            ↑                              ↑
        @babel/parser               @babel/traverse                  @babel/generator
```

**1. Parse（解析阶段）**

解析阶段又分为两个子步骤：

- **词法分析（Lexical Analysis）**：将源代码字符串拆分为 `Token` 流。例如 `const x = 1;` 会被拆分为 `const`、`x`、`=`、`1`、`;` 五个 Token
- **语法分析（Syntax Analysis）**：将 Token 流根据语法规则组装成 AST 树结构

```javascript
// Token 流示例
// 源码：const x = 1 + 2;
[
  { type: 'Keyword',    value: 'const' },
  { type: 'Identifier', value: 'x' },
  { type: 'Punctuator', value: '=' },
  { type: 'Numeric',    value: '1' },
  { type: 'Punctuator', value: '+' },
  { type: 'Numeric',    value: '2' },
  { type: 'Punctuator', value: ';' },
]
```

**2. Transform（转换阶段）**

遍历 AST，通过插件中的 visitor 函数对节点进行增删改查。这是 Babel 最核心的阶段，所有插件都在此阶段工作。

**3. Generate（生成阶段）**

将修改后的 AST 重新生成目标代码字符串，同时可选生成 Source Map。

### AST 节点类型详解

AST 中的每个节点都有 `type` 字段标识节点类型，不同类型的节点有不同的属性：

| 类别 | 节点类型 | 说明 | 示例代码 |
|------|---------|------|---------|
| 根节点 | `Program` | 整个文件的根节点 | — |
| 声明 | `VariableDeclaration` | 变量声明 | `let x = 1` |
| 声明 | `FunctionDeclaration` | 函数声明 | `function foo() {}` |
| 声明 | `ClassDeclaration` | 类声明 | `class A {}` |
| 声明 | `ImportDeclaration` | 导入声明 | `import x from 'a'` |
| 声明 | `ExportDefaultDeclaration` | 默认导出 | `export default {}` |
| 语句 | `ExpressionStatement` | 表达式语句 | `foo()` |
| 语句 | `BlockStatement` | 块语句 | `{ ... }` |
| 语句 | `IfStatement` | if 语句 | `if (x) {}` |
| 语句 | `ReturnStatement` | return 语句 | `return x` |
| 语句 | `ForStatement` | for 循环 | `for (;;) {}` |
| 表达式 | `CallExpression` | 函数调用 | `foo(1, 2)` |
| 表达式 | `MemberExpression` | 成员访问 | `obj.prop` |
| 表达式 | `ArrowFunctionExpression` | 箭头函数 | `() => {}` |
| 表达式 | `BinaryExpression` | 二元运算 | `a + b` |
| 表达式 | `AssignmentExpression` | 赋值 | `x = 1` |
| 字面量 | `StringLiteral` | 字符串字面量 | `"hello"` |
| 字面量 | `NumericLiteral` | 数字字面量 | `42` |
| 字面量 | `BooleanLiteral` | 布尔字面量 | `true` |
| 字面量 | `NullLiteral` | null | `null` |
| 字面量 | `TemplateLiteral` | 模板字符串 | `` `hi ${name}` `` |
| 标识符 | `Identifier` | 标识符 | 变量名、函数名等 |

**常见 AST 节点结构示例：**

```javascript
// const x = 1 + 2; 的 AST 结构（简化）
{
  type: "Program",
  body: [{
    type: "VariableDeclaration",
    kind: "const",
    declarations: [{
      type: "VariableDeclarator",
      id: { type: "Identifier", name: "x" },
      init: {
        type: "BinaryExpression",
        operator: "+",
        left: { type: "NumericLiteral", value: 1 },
        right: { type: "NumericLiteral", value: 2 }
      }
    }]
  }]
}
```

```javascript
// function greet(name) { return 'Hello ' + name; } 的 AST 结构（简化）
{
  type: "FunctionDeclaration",
  id: { type: "Identifier", name: "greet" },
  params: [{ type: "Identifier", name: "name" }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "+",
        left: { type: "StringLiteral", value: "Hello " },
        right: { type: "Identifier", name: "name" }
      }
    }]
  }
}
```

### Babel 核心包详解

| 包名 | 职责 | 常用 API |
|------|------|---------|
| `@babel/core` | 编排整个编译流程 | `transformSync()`、`transformAsync()`、`parse()` |
| `@babel/parser` | 将源码解析为 AST | `parser.parse(code, options)` |
| `@babel/traverse` | 遍历和修改 AST | `traverse(ast, visitor)` |
| `@babel/generator` | 将 AST 生成回代码 | `generate(ast, options, code)` |
| `@babel/types` | AST 节点的判断和创建工具 | `t.isIdentifier()`、`t.identifier()`、`t.callExpression()` |
| `@babel/template` | 模板方式创建 AST | `template.ast()`、`template.expression()` |
| `@babel/helpers` | 编译时的辅助函数 | 自动注入，如 `_classCallCheck` |
| `@babel/runtime` | 运行时辅助函数集 | 与 `transform-runtime` 配合使用 |

**@babel/core 核心用法：**

```javascript
const babel = require('@babel/core');

// 同步转换
const result = babel.transformSync('const x = 1;', {
  presets: ['@babel/preset-env'],
});
console.log(result.code);

// 异步转换
const resultAsync = await babel.transformAsync('const x = 1;', {
  presets: ['@babel/preset-env'],
});

// 只解析不转换
const ast = babel.parseSync('const x = 1;');
```

**@babel/types 核心用法：**

```javascript
const t = require('@babel/types');

// 判断节点类型
t.isIdentifier({ type: 'Identifier', name: 'x' }); // true
t.isCallExpression(node); // 判断是否函数调用

// 创建 AST 节点
const id = t.identifier('myFunc');
const str = t.stringLiteral('hello');
const num = t.numericLiteral(42);
const call = t.callExpression(
  t.identifier('console'),
  [t.identifier('log'), str]
);
const ret = t.returnStatement(num);
const func = t.functionDeclaration(
  t.identifier('myFunc'),
  [], // params
  t.blockStatement([ret])
);
```

### Babel 预设（Preset）

Preset 是一组插件的集合，简化配置：

| 预设 | 包含的功能 | 典型配置项 |
|------|-----------|-----------|
| `@babel/preset-env` | 按目标环境转换 ES6+ 语法 | `targets`、`useBuiltIns`、`corejs`、`modules` |
| `@babel/preset-react` | JSX 编译 + React 专用转换 | `runtime`、`development`、`importSource` |
| `@babel/preset-typescript` | 去除 TypeScript 类型注解 | `isTSX`、`allowNamespaces`、`allowDeclareFields` |
| `@babel/preset-flow` | 去除 Flow 类型注解 | — |

**@babel/preset-env 详解：**

```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "chrome": "60",
        "firefox": "60",
        "ie": "11",
        "node": "12"
      },
      "useBuiltIns": "usage",
      "corejs": { "version": 3, "proposals": true },
      "modules": "auto",
      "bugfixes": true,
      "shippedProposals": true,
      "debug": false,
      "include": [],
      "exclude": []
    }]
  ]
}
```

| `useBuiltIns` 值 | 行为 | 体积影响 |
|------------------|------|---------|
| `false` | 不自动注入 polyfill | 无影响，需手动引入 |
| `"entry"` | 在入口文件处根据 targets 引入全量 polyfill | 中等 |
| `"usage"` | 按每个文件实际使用的 API 注入 polyfill | 最小 |

**@babel/preset-react 详解：**

```json
{
  "presets": [
    ["@babel/preset-react", {
      "runtime": "automatic",
      "development": true,
      "importSource": "react",
      "throwIfNamespace": true
    }]
  ]
}
```

| `runtime` 值 | 行为 |
|-------------|------|
| `"classic"` | JSX 编译为 `React.createElement()`，需手动引入 React |
| `"automatic"` | JSX 编译为自动导入的 `jsx()` 函数，无需手动引入 React |

### Babel 插件执行顺序

Babel 的插件和预设遵循严格的执行顺序规则：

```
1. Parser 插件（解析阶段，影响 AST 生成）
   ↓
2. 转换插件（Transform 插件，从前到后执行）
   ↓
3. 预设（Preset，从后到前执行）
```

**具体规则：**

- 插件（Plugins）在预设（Presets）之前执行
- 插件之间：按配置顺序从前往后执行（先配置的先执行）
- 预设之间：按配置顺序从后往前执行（后配置的先执行）
- 同一预设内的插件：按预设内部的顺序执行

```json
{
  "plugins": ["A", "B", "C"],   // 执行顺序：A → B → C
  "presets": ["P1", "P2", "P3"] // 执行顺序：P3 → P2 → P1
}
```

> 为什么 Preset 从后往前？因为通常 `@babel/preset-env` 要最后处理（最先执行），而 `@babel/preset-react` 和 `@babel/preset-typescript` 需要先处理（后配置）。所以通常把 `preset-env` 放在最后。

### AST Explorer 工具介绍

[AST Explorer](https://astexplorer.net/) 是学习和调试 AST 的核心工具：

- 支持多种解析器（Babel、TypeScript ESLint Parser、Acorn 等）
- 实时预览：左侧输入代码，右侧展示 AST 树
- 点击 AST 节点，高亮对应源码位置
- 支持自定义 Babel 插件实时测试
- 支持多种语言（JS/TS/JSX/CSS/HTML/JSON）

**使用技巧：**

1. 选择 `@babel/parser` 作为解析器
2. 开启 `Transform` 面板测试自定义插件
3. 选中节点后在底部看到节点路径（Path）信息
4. 复制 AST JSON 用于文档或调试

---

## Why — 为什么

### 为什么需要 Babel

**1. 浏览器兼容性问题**

JavaScript 语言在不断演进（ES6/ES2016+/提案），但浏览器对新语法的支持参差不齐。Babel 将新语法转换为等价的旧语法，确保代码在所有目标浏览器中正常运行。

```javascript
// 输入：ES2020 可选链
const name = user?.profile?.name;

// 输出：ES5 兼容代码
var _user$profile;
var name = user === null || user === void 0
  ? void 0
  : (_user$profile = user.profile) === null || _user$profile === void 0
    ? void 0
    : _user$profile.name;
```

**2. 使用新语法提升开发体验**

开发者可以使用最新的语法特性（如箭头函数、解构、类、可选链、空值合并等），无需等待浏览器支持。

**3. 代码转换能力**

Babel 的插件机制使其不仅能做语法降级，还能执行各种代码转换：JSX 编译、TypeScript 类型剥离、自动注入 polyfill、自动国际化、代码注入等。

**4. 统一构建管道**

在一个项目中同时使用 JSX、TypeScript、新 ES 语法时，Babel 可以统一处理所有转换，而非分别使用多个工具。

### Babel vs SWC vs esbuild 对比

| 维度 | Babel | SWC | esbuild |
|------|-------|-----|---------|
| 实现语言 | JavaScript | Rust | Go |
| 编译速度 | 慢（基准） | 快 20-70 倍 | 快 10-100 倍 |
| 插件生态 | 最丰富（数千个） | 增长中（兼容部分 Babel 插件） | 少（插件 API 有限） |
| 自定义转换 | 非常灵活 | 灵活（WASM 插件） | 有限（无 AST 插件 API） |
| JSX 支持 | 完整 | 完整 | 完整 |
| TypeScript | 仅剥离类型 | 仅剥离类型 | 仅剥离类型 |
| Source Map | 支持 | 支持 | 支持 |
| 配置复杂度 | 高 | 中 | 低 |
| 稳定性 | 非常稳定 | 快速迭代中 | 稳定 |
| 适用场景 | 需要复杂自定义转换的项目 | 大型项目加速构建 | 纯构建速度优先 |
| 框架采用 | 通用 | Next.js（默认）、Deno | Vite（开发模式） |

```javascript
// SWC 配置示例（.swcrc）
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "transform": {
      "react": {
        "runtime": "automatic"
      }
    },
    "target": "es2015"
  },
  "env": {
    "targets": "> 0.25%, not dead"
  }
}
```

```javascript
// esbuild 配置示例
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  minify: true,
  target: ['es2015'],
  outdir: 'dist',
  jsx: 'automatic',
});
```

### Babel vs TypeScript 编译器对比

| 维度 | Babel | TypeScript Compiler (tsc) |
|------|-------|--------------------------|
| 类型检查 | 不支持 | 支持（核心功能） |
| 类型剥离 | 支持（`@babel/preset-typescript`） | 支持 |
| 语法转换 | 按目标环境按需转换 | 转换到指定 ES 版本 |
| 常量枚举 | 不支持（需避免使用） | 支持（内联替换） |
| 命名空间 | 有限支持 | 完整支持 |
| 装饰器 | 支持（需配置提案版本） | 支持（旧版标准） |
| 类型导入 | 不处理（需 `transform-typescript`） | 自动剥离 |
| 构建速度 | 较慢 | 中等 |
| 增量编译 | 不支持 | 支持（`--incremental`） |
| 项目引用 | 不支持 | 支持 |
| Source Map | 支持 | 支持 |
| 输出控制 | 灵活（per-file 转换） | 严格（按 tsconfig 输出） |

**推荐协作方案：**

- Babel 负责：语法转换 + JSX + polyfill 注入
- tsc 负责：类型检查（`tsc --noEmit`）
- 两者各司其职，互不冲突

---

## How — 怎么用

### 示例 1：Babel 基础配置

**babel.config.js（项目根目录，推荐方式）：**

```javascript
module.exports = function (api) {
  const isProduction = api.env('production');
  const isDevelopment = api.env('development');
  const isTest = api.env('test');

  return {
    presets: [
      ['@babel/preset-env', {
        targets: isTest
          ? { node: 'current' }
          : '> 0.25%, not dead',
        useBuiltIns: 'usage',
        corejs: 3,
        modules: isTest ? 'commonjs' : false,
      }],
      ['@babel/preset-react', {
        runtime: 'automatic',
        development: isDevelopment,
      }],
      '@babel/preset-typescript',
    ],
    plugins: [
      // 开发环境启用 React Fast Refresh
      isDevelopment && 'react-refresh/babel',
      // 装饰器支持
      ['@babel/plugin-proposal-decorators', { version: '2023-05' }],
      ['@babel/plugin-proposal-class-properties', { loose: true }],
    ].filter(Boolean),
    // 忽略 node_modules（提升构建速度）
    ignore: [
      /node_modules/,
    ],
  };
};
```

**.babelrc.json（旧式配置，适合简单项目）：**

```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "> 0.25%, not dead",
      "useBuiltIns": "usage",
      "corejs": 3
    }],
    "@babel/preset-react",
    "@babel/preset-typescript"
  ],
  "plugins": [
    "@babel/plugin-proposal-optional-chaining",
    "@babel/plugin-proposal-nullish-coalescing-operator"
  ],
  "env": {
    "development": {
      "plugins": ["react-refresh/babel"]
    }
  }
}
```

**两种配置方式对比：**

| 特性 | `babel.config.js` | `.babelrc.json` |
|------|-------------------|-----------------|
| 作用范围 | 整个项目（含 node_modules） | 当前目录及子目录 |
| 动态配置 | 支持（JS 函数） | 不支持（纯 JSON） |
| monorepo 支持 | 好 | 需每个包单独配置 |
| 推荐场景 | 大多数项目 | 简单项目或库 |

### 示例 2：@babel/preset-env + browserslist 配置

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      // 方式一：直接指定 targets
      targets: {
        chrome: '60',
        firefox: '60',
        safari: '12',
        edge: '16',
        ios: '12',
      },

      // 方式二：使用 browserslist 查询字符串
      // targets: '> 0.5%, last 2 versions, not dead, not ie 11',

      // 方式三：使用 browserslist 配置文件（推荐）
      // 不指定 targets，自动读取 .browserslistrc 或 package.json 中的 browserslist

      useBuiltIns: 'usage',  // 按需注入 polyfill
      corejs: { version: 3, proposals: true },  // 包含提案阶段的 polyfill
      modules: false,  // 保留 ES 模块（让 Webpack 做tree shaking）
      bugfixes: true,  // 精确修复，而非降级到 ES5
    }],
  ],
};
```

```text
// .browserslistrc 文件
> 0.5%
last 2 versions
not dead
not ie 11
iOS >= 12
Android >= 5
```

```json
// 或在 package.json 中配置
{
  "browserslist": [
    "> 0.5%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```

**browserslist 常用查询语法：**

| 查询 | 含义 |
|------|------|
| `> 1%` | 全球使用率大于 1% 的浏览器 |
| `last 2 versions` | 每个浏览器的最近 2 个版本 |
| `not dead` | 排除官方不再维护的浏览器 |
| `not ie 11` | 排除 IE 11 |
| `iOS >= 12` | iOS 12 及以上 |
| `since 2020` | 2020 年以来发布的版本 |
| `defaults` | 等同于 `> 0.5%, last 2 versions, not dead` |

### 示例 3：Babel 编译流程（完整版）

```javascript
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generate = require('@babel/generator').default;
const t = require('@babel/types');

const sourceCode = `
  const greet = (name: string): string => {
    return \`Hello \${name}\`;
  };
`;

// 1. 解析（Parse）
const ast = parser.parse(sourceCode, {
  sourceType: 'module',       // 支持 ES 模块语法
  plugins: ['typescript'],    // 支持 TypeScript 语法
  // 其他常用插件：
  // 'jsx' — 支持 JSX
  // 'decorators-legacy' — 支持旧版装饰器
  // 'classProperties' — 支持类属性
  // 'optionalChaining' — 支持可选链
  // 'nullishCoalescingOperator' — 支持空值合并
});

// 2. 遍历 + 转换（Transform）
traverse(ast, {
  // 移除 TypeScript 类型注解
  TSTypeAnnotation(path) {
    path.remove();
  },
  // 移除 TypeScript 类型参数
  TSTypeParameterInstantiation(path) {
    path.remove();
  },
  // 箭头函数转普通函数
  ArrowFunctionExpression(path) {
    // 跳过有 this 引用的箭头函数
    let skipTransform = false;
    path.traverse({
      ThisExpression() { skipTransform = true; },
    });
    if (skipTransform) return;

    const func = t.functionExpression(
      null,               // 函数名（匿名）
      path.node.params,   // 参数
      path.node.body,     // 函数体
      path.node.generator, // 是否 generator
      path.node.async      // 是否 async
    );
    path.replaceWith(func);
  },
});

// 3. 生成（Generate）
const output = generate(ast, {
  retainLines: true,     // 尽量保持行号
  comments: true,        // 保留注释
  compact: false,        // 不压缩
  sourceMaps: true,      // 生成 Source Map
}, sourceCode);

console.log(output.code);
// const greet = function (name) {
//   return `Hello ${name}`;
// };
```

### 示例 4：编写自定义 Babel 插件（自动日志插件）

```javascript
// babel-plugin-auto-logger.js
// 自动在函数入口注入 console.log，用于调试

module.exports = function ({ types: t }) {
  return {
    name: 'auto-logger',
    visitor: {
      // 匹配所有函数声明
      FunctionDeclaration(path) {
        const functionName = path.node.id?.name;
        if (!functionName) return;

        // 跳过已有 console.log 的函数
        let hasLogger = false;
        path.traverse({
          CallExpression(innerPath) {
            const callee = innerPath.node.callee;
            if (
              t.isMemberExpression(callee) &&
              t.isIdentifier(callee.object, { name: 'console' }) &&
              t.isIdentifier(callee.property, { name: 'log' })
            ) {
              hasLogger = true;
            }
          },
        });
        if (hasLogger) return;

        // 创建 console.log('函数名 被调用') 语句
        const logStatement = t.expressionStatement(
          t.callExpression(
            t.memberExpression(
              t.identifier('console'),
              t.identifier('log')
            ),
            [t.stringLiteral(`${functionName} 被调用`)]
          )
        );

        // 插入到函数体开头
        path.node.body.body.unshift(logStatement);
      },
    },
  };
};
```

```javascript
// 使用插件
// babel.config.js
module.exports = {
  plugins: [
    ['./babel-plugin-auto-logger.js', {
      // 可扩展：支持配置日志级别
      // level: 'debug',
    }],
  ],
};
```

**转换效果：**

```javascript
// 输入
function add(a, b) {
  return a + b;
}

// 输出
function add(a, b) {
  console.log("add 被调用");
  return a + b;
}
```

### 示例 5：编写自定义 Babel 插件（自动国际化插件）

```javascript
// babel-plugin-auto-i18n.js
// 自动提取代码中的中文字符串，替换为 i18n 函数调用

const generateI18nKey = (text) => {
  // 简单的 key 生成策略：取拼音首字母或哈希
  return 'i18n.' + text.split('').map(c => c.charCodeAt(0)).join('_');
};

module.exports = function ({ types: t }) {
  return {
    name: 'auto-i18n',
    visitor: {
      // 处理字符串字面量
      StringLiteral(path) {
        const { value } = path.node;
        if (!/[一-龥]/.test(value)) return; // 跳过非中文

        // 避免重复处理
        if (path.findParent(p => t.isCallExpression(p.node) &&
          t.isIdentifier(p.node.callee, { name: 't' }))) {
          return;
        }

        const key = generateI18nKey(value);
        path.replaceWith(
          t.callExpression(t.identifier('t'), [t.stringLiteral(key)])
        );
      },

      // 处理模板字符串中的中文
      TemplateLiteral(path) {
        path.get('quasis').forEach(quasi => {
          const value = quasi.node.value.raw;
          if (!/[一-龥]/.test(value)) return;

          const key = generateI18nKey(value);
          quasi.node.value.raw = `{t('${key}')}`;
          quasi.node.value.cooked = `{t('${key}')}`;
        });
      },

      // 处理 JSX 文本
      JSXText(path) {
        const { value } = path.node;
        if (!/[一-龥]/.test(value)) return;

        const key = generateI18nKey(value);
        path.replaceWith(
          t.jsxExpressionContainer(
            t.callExpression(t.identifier('t'), [t.stringLiteral(key)])
          )
        );
      },
    },
  };
};
```

**转换效果：**

```javascript
// 输入
const msg = "你好世界";
alert("操作成功");

// 输出
const msg = t("i18n.20320_22909_19990_30028");
alert(t("i18n.25820_20316_25104_21151"));
```

```jsx
// 输入 JSX
<div>欢迎登录</div>

// 输出 JSX
<div>{t("i18n.27426_36814_30331_24405")}</div>
```

### 示例 6：AST 遍历与修改（@babel/traverse 用法）

```javascript
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const t = require('@babel/types');

const code = `
  import { Button, Input, Modal } from 'antd';
  import React, { useState } from 'react';

  function App() {
    const [visible, setVisible] = useState(false);
    return <Modal visible={visible}><Input /></Modal>;
  }
`;

const ast = parser.parse(code, {
  sourceType: 'module',
  plugins: ['jsx'],
});

// 1. 收集所有导入的模块
const imports = [];
traverse(ast, {
  ImportDeclaration(path) {
    imports.push({
      source: path.node.source.value,
      specifiers: path.node.specifiers.map(s => s.local.name),
    });
  },
});
console.log('导入分析:', imports);
// [{ source: 'antd', specifiers: ['Button', 'Input', 'Modal'] },
//  { source: 'react', specifiers: ['React', 'useState'] }]

// 2. 检测未使用的导入
traverse(ast, {
  ImportDeclaration(path) {
    const source = path.node.source.value;
    path.node.specifiers.forEach((spec, index) => {
      const localName = spec.local.name;
      // 绑定检查：变量是否被引用
      const binding = path.scope.getBinding(localName);
      if (!binding || binding.references === 0) {
        // 如果是唯一的 specifier，移除整条 import 语句
        if (path.node.specifiers.length === 1) {
          path.remove();
        } else {
          // 否则只移除该 specifier
          path.node.specifiers.splice(index, 1);
        }
        console.log(`移除未使用的导入: ${localName} from ${source}`);
      }
    });
  },
});

// 3. 重命名变量
traverse(ast, {
  Identifier(path) {
    if (path.node.name === 'visible') {
      path.node.name = 'isOpen';  // 重命名
    }
  },
});

// 4. 查找所有 React Hook 调用
const hookCalls = [];
traverse(ast, {
  CallExpression(path) {
    const callee = path.node.callee;
    if (t.isIdentifier(callee) && /^use[A-Z]/.test(callee.name)) {
      hookCalls.push({
        name: callee.name,
        loc: path.node.loc,
      });
    }
  },
});
console.log('Hook 调用:', hookCalls);
// [{ name: 'useState', loc: ... }]
```

### 示例 7：AST 节点创建（@babel/types + @babel/template）

```javascript
const t = require('@babel/types');
const template = require('@babel/template').default;

// ===== 使用 @babel/types 手动创建 =====

// 创建 const x = 42;
const varDecl = t.variableDeclaration('const', [
  t.variableDeclarator(
    t.identifier('x'),
    t.numericLiteral(42)
  )
]);

// 创建 function add(a, b) { return a + b; }
const funcDecl = t.functionDeclaration(
  t.identifier('add'),
  [t.identifier('a'), t.identifier('b')],
  t.blockStatement([
    t.returnStatement(
      t.binaryExpression('+', t.identifier('a'), t.identifier('b'))
    )
  ])
);

// 创建 export default function
const exportDefault = t.exportDefaultDeclaration(funcDecl);

// 创建 import 语句
const importDecl = t.importDeclaration(
  [t.importDefaultSpecifier(t.identifier('React'))],
  t.stringLiteral('react')
);

// ===== 使用 @babel/template 模板创建（更简洁）=====

// 模板创建语句
const logStatement = template.statement(`
  console.log(MESSAGE);
`);

const ast1 = logStatement({ MESSAGE: t.stringLiteral('Hello') });

// 模板创建表达式
const callExpr = template.expression(`
  fn(...args)
`);

const ast2 = callExpr({
  fn: t.identifier('sum'),
  args: [t.numericLiteral(1), t.numericLiteral(2)],
});

// 模板创建整个函数
const createSetter = template.statement(`
  function SETTER_NAME(value) {
    STATE_NAME = value;
  }
`);

const ast3 = createSetter({
  SETTER_NAME: t.identifier('setName'),
  STATE_NAME: t.identifier('name'),
});

// template.ast() 直接返回 AST 节点（非函数包装）
const programAst = template.ast(`
  import React from 'react';
  export default function App() {
    return <div>Hello</div>;
  }
`, { plugins: ['jsx'] });
```

### 示例 8：Babel 宏（babel-plugin-macros）

Babel 宏允许在编译时执行 JavaScript 逻辑，避免了编写复杂 Babel 插件的门槛。

```javascript
// 安装：npm install babel-plugin-macros

// babel.config.js 只需配置一次
module.exports = {
  plugins: ['babel-plugin-macros'],
};
```

```javascript
// 定义宏：features.macro.js
const { createMacro } = require('babel-plugin-macros');

module.exports = createMacro(function featureMacro({ references, state, babel }) {
  const { types: t } = babel;

  references.default.forEach((referencePath) => {
    // 获取宏调用的参数
    const callExpr = referencePath.parentPath;
    if (!callExpr.isCallExpression()) return;

    const args = callExpr.get('arguments');
    const featureName = args[0]?.node.value;

    // 根据编译时环境变量决定是否启用功能
    if (process.env.FEATURES?.includes(featureName)) {
      // 功能启用，替换为 true
      callExpr.replaceWith(t.booleanLiteral(true));
    } else {
      // 功能禁用，替换为 false
      callExpr.replaceWith(t.booleanLiteral(false));
    }
  });
});
```

```javascript
// 使用宏
import feature from './features.macro';

function App() {
  if (feature('darkMode')) {
    // 这段代码在编译时决定是否保留
    enableDarkMode();
  }

  if (feature('newDashboard')) {
    renderNewDashboard();
  } else {
    renderOldDashboard();
  }
}

// 编译时 FEATURES=darkMode npm run build 后：
function App() {
  if (true) {
    enableDarkMode();
  }
  if (false) {
    renderNewDashboard();
  } else {
    renderOldDashboard();
  }
}
```

### 示例 9：代码压缩与优化（@babel/plugin-transform-* 系列）

```javascript
// babel.config.js — 生产环境优化配置
module.exports = function (api) {
  const isProd = api.env('production');

  return {
    presets: [
      ['@babel/preset-env', {
        targets: '> 0.25%, not dead',
        useBuiltIns: 'usage',
        corejs: 3,
        bugfixes: isProd,   // 生产环境启用精确 bug 修复
        forceAllTransforms: isProd,  // 强制所有转换（确保兼容）
      }],
    ],
    plugins: [
      // 常用 transform 插件
      '@babel/plugin-transform-runtime',  // 提取辅助函数，减少体积

      // 生产环境专用优化
      isProd && [
        '@babel/plugin-transform-react-constant-elements',
        // 提升不变的 JSX 元素为常量
      ],
      isProd && [
        '@babel/plugin-transform-react-pure-annotations',
        // 标记纯函数组件，便于 Webpack tree shaking
      ],
      isProd && [
        'babel-plugin-transform-react-remove-prop-types',
        // 移除 PropTypes（生产环境不需要）
        { removeImport: true },
      ],
    ].filter(Boolean),
  };
};
```

**@babel/plugin-transform-runtime 详解：**

```javascript
// 不使用 transform-runtime：每个文件都会内联辅助函数
// file1.js
function _classCallCheck(instance, Constructor) { ... }
function _defineProperties(target, props) { ... }
var MyClass = /* ... */;

// file2.js
function _classCallCheck(instance, Constructor) { ... }  // 重复！
function _defineProperties(target, props) { ... }          // 重复！
var AnotherClass = /* ... */;
```

```javascript
// 使用 transform-runtime：辅助函数从 @babel/runtime 引入
// file1.js
var _classCallCheck = require("@babel/runtime/helpers/classCallCheck");
var _defineProperties = require("@babel/runtime/helpers/defineProperties");
var MyClass = /* ... */;

// file2.js
var _classCallCheck2 = require("@babel/runtime/helpers/classCallCheck");  // 复用！
var MyClass2 = /* ... */;
```

**常用 @babel/plugin-transform-* 插件列表：**

| 插件 | 功能 | preset-env 是否包含 |
|------|------|-------------------|
| `transform-arrow-functions` | 箭头函数转普通函数 | 是 |
| `transform-classes` | class 转构造函数 | 是 |
| `transform-template-literals` | 模板字符串转字符串拼接 | 是 |
| `transform-destructuring` | 解构转变量赋值 | 是 |
| `transform-spread` | 展开运算符转 apply | 是 |
| `transform-optional-chaining` | 可选链转条件判断 | 是 |
| `transform-nullish-coalescing-operator` | 空值合并转条件判断 | 是 |
| `transform-react-jsx` | JSX 转 createElement | preset-react |
| `transform-react-constant-elements` | 提升不变的 JSX 元素 | 否 |
| `transform-react-pure-annotations` | 标记纯组件 | 否 |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| polyfill 体积过大 | 全量引入 corejs | 用 `useBuiltIns: "usage"` 按需引入 |
| 装饰器编译错误 | 装饰器提案多个版本 | 配置 `@babel/plugin-proposal-decorators` 的 `version: "2023-05"` |
| Babel 和 TS Compiler 冲突 | 两者都处理 TS | 用 `@babel/preset-typescript` 编译，`tsc --noEmit` 检查类型 |
| 编译慢 | JS 实现，项目大 | 使用 SWC 替代（Vite/Next.js 已内置） |
| `regeneratorRuntime is not defined` | async/await 缺少运行时 | 添加 `@babel/plugin-transform-runtime` 或引入 `regenerator-runtime` |
| class 属性编译结果不一致 | loose 模式 vs 标准模式 | 统一配置 `loose: true` 或 `loose: false` |
| 常量枚举（const enum）不工作 | Babel 不支持 TS 常量枚举内联 | 避免使用 `const enum`，改用普通 `enum` |
| JSX 中自动导入不生效 | 未配置 automatic runtime | `@babel/preset-react` 设置 `runtime: "automatic"` |
| 第三方库被 Babel 编译 | 默认不忽略 node_modules | 配置 `ignore: [/node_modules/]` 或使用 `exclude` |
| 动态导入 `import()` 报错 | 缺少动态导入插件 | `@babel/preset-env` 的 `modules: false` + 确保支持 `import()` |

### 最佳实践

- 新项目考虑用 SWC/esbuild 替代 Babel（Vite/Next.js 已内置）
- `useBuiltIns: "usage"` 按需 polyfill，避免体积膨胀
- Babel 负责 JS 转换，`tsc --noEmit` 负责类型检查，各司其职
- 生产环境使用 `@babel/plugin-transform-runtime` 避免辅助函数重复
- `modules: false` 保留 ES 模块语法，让打包器做 tree shaking
- 使用 `babel.config.js` 而非 `.babelrc`，支持动态配置和 monorepo
- 编写自定义插件时善用 `@babel/template` 简化 AST 构造
- 了解 AST 是理解所有前端工具的基础（ESLint、Prettier、Webpack、Vite 等）

---

## 面试题

**Q1: Babel 的编译流程是什么？**
> Babel 编译分三个阶段：**Parse**（`@babel/parser` 将源码字符串解析为 AST，包括词法分析和语法分析两步） → **Transform**（`@babel/traverse` 遍历 AST，通过插件中的 visitor 函数对节点进行增删改查） → **Generate**（`@babel/generator` 将修改后的 AST 生成目标代码字符串和 Source Map）。整个流程是"源码 → AST → 修改后的 AST → 目标代码"。

**Q2: AST 是什么？为什么前端工具都基于它？**
> AST（抽象语法树）是源代码的树形结构化表示，每个节点对应一个语法单元（变量声明、函数调用、表达式等）。ESLint 检查、Babel 转换、Prettier 格式化、代码压缩、自动重构等工具都基于 AST，因为树结构便于程序化地定位和修改代码，比正则字符串替换更精确可靠，不会出现误替换的情况。

**Q3: Babel 的 preset 和 plugin 有什么区别？**
> Plugin 是单个转换功能的最小单元（如 `@babel/plugin-transform-arrow-functions` 只转换箭头函数）；Preset 是一组 plugin 的集合（如 `@babel/preset-env` 包含所有 ES6+ 转换插件），用于简化配置。执行顺序：Plugin 先于 Preset 执行，Plugin 按配置顺序从前往后执行，Preset 按配置顺序从后往前执行。

**Q4: @babel/polyfill 和 @babel/plugin-transform-runtime 有什么区别？**
> `@babel/polyfill` 直接修改全局对象和原型（如 `Array.prototype.includes`），会污染全局作用域，适合应用开发但已废弃；`@babel/plugin-transform-runtime` 以 helper 方式引入 polyfill，通过模块变量引用而非修改全局，不会污染全局作用域，适合库开发。推荐方案：应用用 `core-js` + `useBuiltIns: "usage"`，库用 `transform-runtime`。

**Q5: 如何编写一个 Babel 插件？**
> Babel 插件是一个函数，接收 `babel` 参数（含 `types`），返回包含 `visitor` 的对象。visitor 中每个方法名对应一个 AST 节点类型，方法接收 `path` 参数（包含节点信息和操作方法）。基本结构：
> ```javascript
> module.exports = function({ types: t }) {
>   return {
>     name: 'my-plugin',
>     visitor: {
>       Identifier(path) {
>         // path.node — 当前节点
>         // path.parent — 父节点
>         // path.scope — 作用域
>         // path.replaceWith() — 替换节点
>         // path.remove() — 删除节点
>         // path.insertBefore() — 前插入
>         // path.insertAfter() — 后插入
>       }
>     }
>   }
> }
> ```

**Q6: preset 和 plugin 的执行顺序是什么？**
> 执行顺序规则：1）Plugins 在 Presets 之前执行；2）Plugins 之间按配置顺序从前往后执行（先写的先执行）；3）Presets 之间按配置顺序从后往前执行（后写的先执行）。这样设计是因为通常 `preset-env` 需要最后处理（所以放在 presets 数组最后），而 `preset-typescript` 和 `preset-react` 需要先处理（放在数组前面）。

**Q7: Babel 和 TypeScript 编译器有什么区别？**
> 核心区别：Babel 不做类型检查，只做语法转换（stripping types）；TypeScript 编译器既做类型检查又做语法转换。Babel 的优势是插件生态丰富、配置灵活，可与 JSX/新 ES 语法统一处理；TypeScript 的优势是类型检查、增量编译、项目引用、常量枚举内联等。推荐协作方式：Babel 负责代码转换，`tsc --noEmit` 负责类型检查。此外，Babel 不支持 `const enum` 内联和 `namespace` 的完整转换。

**Q8: Babel 如何实现按需 polyfill？useBuiltIns 三种模式有什么区别？**
> `useBuiltIns` 是 `@babel/preset-env` 的配置项，控制 polyfill 注入方式。`false`：不自动注入，需手动引入全部 core-js；`"entry"`：在入口文件处根据 targets 注入所有需要的 polyfill（全量但按目标环境筛选）；`"usage"`：按每个文件中实际使用的 API 精确注入 polyfill（最小体积）。推荐使用 `"usage"` + `corejs: 3`，体积最小且无需手动管理。注意 `"usage"` 模式无法处理原型方法在第三方库中的调用。

---

**相关链接：**
- [[Webpack与Vite]]
- [[代码规范ESLint与Prettier]]
- [[TypeScript基础]]
- [[模块化规范]]

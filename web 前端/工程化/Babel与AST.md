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

**核心概念：**

- **AST**：源代码解析成的树结构，每个节点代表一个语法单元
- **@babel/parser**：将源码解析为 AST
- **@babel/traverse**：遍历和修改 AST 节点
- **@babel/generator**：将 AST 生成回代码
- **Preset**：一组插件的集合（`@babel/preset-env`、`@babel/preset-react`、`@babel/preset-typescript`）

**关键特性：**

- Babel 只做语法转换，不包含 API polyfill（需 `core-js`）
- `@babel/preset-env` + `browserslist` 按目标浏览器按需转换
- JSX → `React.createElement()`（或自动 JSX Runtime）
- 插件顺序：先 parser 插件，再转换插件

## Why — 为什么

**适用场景：**

- 新语法兼容旧浏览器
- JSX 编译
- TypeScript 编译（Babel 仅 stripping types，不做类型检查）
- 自定义代码转换（自动化重构、国际化提取）

**对比方案：**

| 维度 | Babel | TypeScript Compiler | SWC |
|------|-------|-------------------|-----|
| 速度 | 慢（JS 实现） | 中 | 极快（Rust） |
| 类型检查 | 不支持 | 支持 | 不支持 |
| 插件生态 | 最丰富 | 少 | 增长中 |
| 配置复杂度 | 高 | 中 | 低 |

**优缺点：**

- ✅ 优点：
  - 插件生态极其丰富
  - 高度可定制
  - JSX/TS 统一处理
- ❌ 缺点：
  - 编译速度慢（大项目明显）
  - 配置复杂
  - 不做类型检查

## How — 怎么用

### 快速上手

**babel.config.json：**

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
    ]
}
```

### 代码示例

**Babel 编译流程：**

```javascript
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generate = require('@babel/generator').default;

// 1. 解析
const ast = parser.parse('const x: number = 1;', {
    sourceType: 'module',
    plugins: ['typescript'],
});

// 2. 遍历 + 转换
traverse(ast, {
    // 移除 TypeScript 类型注解
    TSTypeAnnotation(path) {
        path.remove();
    },
});

// 3. 生成
const output = generate(ast);
console.log(output.code); // "const x = 1;"
```

**自定义 Babel 插件：**

```javascript
// 自动添加 displayName 到 React 组件
module.exports = function autoDisplayName() {
    return {
        visitor: {
            FunctionDeclaration(path) {
                if (isReactComponent(path)) {
                    path.node.id = path.node.id || t.identifier(
                        path.parent.id?.name || 'Anonymous'
                    );
                }
            },
        },
    };
};
```

**AST 实际应用场景：**

```javascript
// 自动国际化：提取代码中的中文字符串
traverse(ast, {
    StringLiteral(path) {
        if (/[一-龥]/.test(path.node.value)) {
            const key = generateI18nKey(path.node.value);
            path.replaceWith(
                t.callExpression(t.identifier('t'), [t.stringLiteral(key)])
            );
        }
    },
});
// "你好" → t("hello")
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| polyfill 体积过大 | 全量引入 corejs | 用 `useBuiltIns: "usage"` 按需引入 |
| 装饰器编译错误 | 装饰器提案多个版本 | 配置 `@babel/plugin-proposal-decorators` 的 `version` |
| Babel 和 TS Compiler 冲突 | 两者都处理 TS | 用 `@babel/preset-typescript` 编译，`tsc --noEmit` 检查类型 |
| 编译慢 | JS 实现，项目大 | 使用 SWC 替代（Vite/Next.js 已内置） |

### 最佳实践

- 新项目考虑用 SWC/esbuild 替代 Babel（Vite 内置）
- `useBuiltIns: "usage"` 按需 polyfill
- Babel 负责 JS 转换，`tsc --noEmit` 负责类型检查
- 了解 AST 是理解所有前端工具的基础

## 面试题

**Q1: Babel 的编译流程是怎样的？**
> Babel 编译分三步：Parse（`@babel/parser` 将源码解析为 AST）→ Transform（`@babel/traverse` 遍历 AST 并通过插件修改节点）→ Generate（`@babel/generator` 将修改后的 AST 生成目标代码和 SourceMap）。

**Q2: AST 是什么？为什么前端工具都基于它？**
> AST（抽象语法树）是源代码的树形结构化表示，每个节点对应一个语法单元。ESLint 检查、Babel 转换、Prettier 格式化、代码压缩等工具都基于 AST，因为树结构便于程序化地定位和修改代码，比正则字符串替换更精确可靠。

**Q3: Babel 的 preset 和 plugin 有什么区别？**
> Plugin 是单个转换功能的最小单元（如 `@babel/plugin-transform-arrow-functions`）；Preset 是一组 plugin 的集合（如 `@babel/preset-env` 包含所有 ES6+ 转换插件）。执行顺序：Plugin 先于 Preset，Plugin 从前到后，Preset 从后到前。

**Q4: @babel/polyfill 和 @babel/plugin-transform-runtime 有什么区别？**
> `@babel/polyfill` 直接修改全局对象和原型（如 `Array.prototype.includes`），会污染全局作用域；`@babel/plugin-transform-runtime` 以 `helpers` 方式引入 polyfill，通过变量引用而非修改全局，适合库开发避免污染使用者环境。前者已废弃，推荐 `core-js` + `useBuiltIns` 或 `transform-runtime` 方案。

---

**相关链接：**
- [[Webpack与Vite]]
- [[代码规范ESLint与Prettier]]

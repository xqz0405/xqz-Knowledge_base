---
tags:
  - Web前端
  - 工程化
  - ESLint
  - Prettier
  - 代码规范
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# 代码规范ESLint与Prettier

## What — 是什么

> ESLint 检查代码质量和逻辑错误，Prettier 统一代码格式。两者配合实现代码风格一致 + 逻辑正确。

**核心概念：**

- **ESLint**：基于 AST 的可插拔 lint 工具，规则可配置
- **Prettier**：代码格式化工具，极简配置，opinionated
- **冲突问题**：ESLint 格式规则与 Prettier 冲突，需 `eslint-config-prettier` 关闭 ESLint 格式规则
- **共享配置**：团队统一规则（`eslint-config-xxx`）

**关键特性：**

- ESLint 规则：`off` / `warn` / `error`
- Prettier 核心配置：`printWidth`、`tabWidth`、`semi`、`singleQuote`、`trailingComma`
- `eslint-config-prettier`：关闭所有与 Prettier 冲突的 ESLint 规则
- `husky` + `lint-staged`：Git 提交时自动 lint

## Why — 为什么

**适用场景：**

- 所有团队项目
- CI/CD 质量门禁
- 代码评审减少风格争议

**对比方案：**

| 维度 | ESLint + Prettier | Biome | ESLint Alone |
|------|-------------------|-------|-------------|
| 格式化 | Prettier 负责 | 内置 | 部分支持 |
| 代码检查 | ESLint 负责 | 内置 | 完整 |
| 速度 | 中 | 极快（Rust） | 中 |
| 生态 | 最丰富 | 增长中 | 丰富 |

**优缺点：**

- ✅ 优点：
  - ESLint 生态成熟，规则丰富
  - Prettier 格式化一致，零争论
  - 团队统一风格
- ❌ 缺点：
  - 两个工具配合配置复杂
  - 大项目 lint 速度慢
  - 规则冲突需额外处理

## How — 怎么用

### 快速上手

```bash
npm install -D eslint prettier eslint-config-prettier
npx eslint --init
```

**ESLint 配置（Flat Config，ESLint 9+）：**

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import prettier from 'eslint-config-prettier';

export default tseslint.config(
    js.configs.recommended,
    ...tseslint.configs.recommended,
    reactHooks.configs.recommended,
    prettier, // 必须放最后，关闭冲突规则
    {
        rules: {
            'no-console': 'warn',
            '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
        },
    },
);
```

**Prettier 配置：**

```json
// .prettierrc
{
    "printWidth": 100,
    "tabWidth": 2,
    "semi": true,
    "singleQuote": true,
    "trailingComma": "all",
    "arrowParens": "always",
    "endOfLine": "lf"
}
```

### 代码示例

**Git 提交自动 lint（husky + lint-staged）：**

```bash
npm install -D husky lint-staged
npx husky init
```

```json
// package.json
{
    "lint-staged": {
        "*.{js,ts,tsx}": [
            "eslint --fix",
            "prettier --write"
        ],
        "*.{json,md,css}": [
            "prettier --write"
        ]
    }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

**VSCode 设置（自动保存格式化）：**

```json
// .vscode/settings.json
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit"
    }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| ESLint 和 Prettier 规则冲突 | ESLint 格式规则与 Prettier 不一致 | `eslint-config-prettier` 放配置最后 |
| lint-staged 卡住 | 文件太多或规则太多 | 只对暂存文件 lint，精简规则 |
| ESLint 找不到 TS 类型 | 缺少 parser 配置 | 使用 `typescript-eslint` parser |
| 保存格式化抖动 | ESLint 和 Prettier 交替格式化 | Prettier 负责格式化，ESLint 只检查逻辑 |

### 最佳实践

- Prettier 管格式，ESLint 管逻辑，职责不交叉
- `eslint-config-prettier` 必须放配置最后
- Git hook 自动 lint，不让不合规代码进入仓库
- `.vscode/settings.json` 纳入版本控制，团队统一

## 面试题

**Q1: ESLint 和 Prettier 规则冲突怎么解决？**
> 使用 `eslint-config-prettier` 关闭所有与 Prettier 冲突的 ESLint 格式规则，且该配置必须放在 ESLint 配置数组的最后，确保覆盖前面的格式规则。核心原则：Prettier 管格式，ESLint 管逻辑，职责不交叉。

**Q2: `eslint-disable` 注释有哪些使用方式？有什么风险？**
> 支持行级 `// eslint-disable-line`、块级 `/* eslint-disable */`、单规则 `// eslint-disable-line no-console` 三种方式。风险是滥用会掩盖真正的代码问题，应仅在必要场景使用（如第三方代码约束），且推荐指定具体规则名而非全量禁用。

**Q3: 如何自定义 ESLint 规则？**
> 编写自定义规则需创建一个插件，规则函数接收 `context` 对象，通过 `context.report()` 上报问题。规则基于 AST 节点访问器（`create` 函数中的 `Selector`）匹配代码模式，类似 Babel 插件的 visitor 机制。

**Q4: EditorConfig 的作用是什么？和 Prettier 有什么区别？**
> EditorConfig 跨编辑器统一基本编辑行为（缩进风格、缩进宽度、行尾符、字符编码等），作用于编辑器层面；Prettier 是代码格式化引擎，控制更细粒度的格式（分号、引号、行宽等）。两者有重叠配置需保持一致，EditorConfig 先生效，Prettier 再覆盖。

---

**相关链接：**
- [[Webpack与Vite]]
- [[Babel与AST]]

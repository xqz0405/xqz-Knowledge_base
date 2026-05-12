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

### ESLint 完整架构

ESLint 由四大核心模块组成，形成完整的 lint 管道：

```
源代码 → Parser(解析) → AST → Rules(规则检查) → Formatter(格式化输出) → CLI(命令行交互)
```

| 模块 | 职责 | 可替换性 |
|------|------|----------|
| **Parser** | 将源代码解析为 AST | 可替换（如 `@typescript-eslint/parser`、`@babel/eslint-parser`） |
| **Rules** | 基于 AST 节点匹配，检查代码问题 | 可扩展（自定义规则、插件） |
| **Formatter** | 将检查结果格式化为可读输出 | 可替换（stylish、json、junit 等） |
| **CLI** | 命令行入口，提供 `--fix`、`--cache` 等 | 不可替换 |

**Parser 详解：**

- 默认 Parser 是 `espree`（基于 Acorn），只支持标准 ECMAScript
- TypeScript 项目需替换为 `@typescript-eslint/parser`，它先将 TS 代码转换为 ESTree 兼容的 AST
- 实验性语法（装饰器等）可使用 `@babel/eslint-parser`

**Rules 详解：**

- 每条规则是一个对象，包含 `meta`（元信息）和 `create`（访问器函数）
- `create` 函数返回 AST 节点的访问器（visitor），类似 Babel 插件机制
- 规则通过 `context.report()` 上报问题，支持 `fix` 自动修复

**Formatter 详解：**

```bash
# 常用格式化器
eslint -f stylish .    # 默认，彩色终端输出
eslint -f json .       # JSON 格式，适合机器读取
eslint -f junit .      # JUnit XML，适合 CI 集成
eslint -f html .       # HTML 报告，适合团队审查
```

### ESLint 配置文件详解

ESLint 经历了两个配置体系：传统配置（`.eslintrc.*`）和 Flat Config（`eslint.config.js`）。

**传统配置（.eslintrc.js / .eslintrc.json / .eslintrc.yaml）：**

```javascript
// .eslintrc.js
module.exports = {
    root: true,                          // 停止向上查找配置
    env: {
        browser: true,                   // 浏览器全局变量
        node: true,                      // Node.js 全局变量
        es2022: true,                    // ES2022 语法
    },
    parser: '@typescript-eslint/parser',  // 指定解析器
    parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module',
        ecmaFeatures: { jsx: true },
    },
    plugins: ['@typescript-eslint', 'react-hooks'],
    extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
        'plugin:react-hooks/recommended',
        'prettier',                      // eslint-config-prettier
    ],
    rules: {
        'no-console': 'warn',
        '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    },
    overrides: [
        {
            files: ['*.test.ts'],
            rules: { '@typescript-eslint/no-explicit-any': 'off' },
        },
    ],
    ignorePatterns: ['dist/', 'node_modules/'],
};
```

**传统配置核心字段说明：**

| 字段 | 作用 | 说明 |
|------|------|------|
| `root` | 停止配置合并 | 避免继承上层目录的配置 |
| `env` | 声明运行环境 | 预设全局变量（browser、node、jest 等） |
| `parser` | 指定解析器 | 默认 espree，TS 用 `@typescript-eslint/parser` |
| `parserOptions` | 解析器选项 | ECMAScript 版本、模块类型、JSX 等 |
| `plugins` | 加载插件 | 只加载，不启用规则 |
| `extends` | 继承配置 | 共享配置 + 插件推荐配置，按顺序覆盖 |
| `rules` | 自定义规则 | 覆盖 extends 中的规则 |
| `overrides` | 文件级覆盖 | 对特定文件应用不同规则 |
| `ignorePatterns` | 忽略文件 | 不检查的文件/目录 |

**Flat Config（eslint.config.js，ESLint 9+ 默认）：**

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import prettier from 'eslint-config-prettier';
import globals from 'globals';

export default tseslint.config(
    // 全局忽略
    {
        ignores: ['dist/', 'node_modules/', '**/*.d.ts'],
    },
    // 基础推荐规则
    js.configs.recommended,
    // TypeScript 推荐
    ...tseslint.configs.recommended,
    // React Hooks
    reactHooks.configs.recommended,
    // 自定义规则
    {
        files: ['**/*.{ts,tsx}'],
        languageOptions: {
            ecmaVersion: 'latest',
            sourceType: 'module',
            globals: {
                ...globals.browser,
                ...globals.node,
            },
        },
        rules: {
            'no-console': 'warn',
            '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
        },
    },
    // 测试文件覆盖
    {
        files: ['**/*.test.ts'],
        rules: {
            '@typescript-eslint/no-explicit-any': 'off',
        },
    },
    // Prettier 必须放最后
    prettier,
);
```

**传统配置 vs Flat Config 对比：**

| 维度 | 传统配置 (.eslintrc) | Flat Config (eslint.config.js) |
|------|----------------------|-------------------------------|
| 格式 | JSON / YAML / JS | 仅 JS（ESM） |
| 继承机制 | `extends` 字符串合并 | 直接导入配置对象 |
| 插件引用 | 字符串名 `plugins: ['react']` | 导入对象 `import react from 'eslint-plugin-react'` |
| 环境 | `env` 字段 | `languageOptions.globals` |
| 解析器 | `parser` 字符串 | `languageOptions.parser` 对象 |
| 忽略 | `.eslintignore` / `ignorePatterns` | `ignores` 配置项 |
| 覆盖 | `overrides` 数组 | 独立配置对象 + `files` 匹配 |
| 模块系统 | CommonJS | ESM（推荐） |
| 状态 | ESLint 9+ 废弃 | ESLint 9+ 默认 |

### ESLint 规则级别与常用规则分类

**规则级别：**

| 级别 | 值 | 含义 | 退出码 |
|------|----|------|--------|
| 关闭 | `"off"` 或 `0` | 不检查 | - |
| 警告 | `"warn"` 或 `1` | 报警告，不阻断 | 0 |
| 错误 | `"error"` 或 `2` | 报错误，阻断 CI | 1 |

规则可配置选项：`'rule-name': 'level'` 或 `'rule-name': ['level', { options }]`

**常用规则分类：**

| 分类 | 典型规则 | 说明 |
|------|----------|------|
| **可能的错误** | `no-unused-vars`、`no-undef`、`no-dupe-keys`、`no-unreachable` | 逻辑错误，必须开启 |
| **最佳实践** | `no-console`、`no-eval`、`no-implicit-coercion`、`curly` | 代码质量 |
| **严格模式** | `strict` | 控制严格模式声明 |
| **变量** | `no-shadow`、`no-use-before-define`、`init-declarations` | 变量声明规范 |
| **风格** | `quotes`、`semi`、`indent`、`comma-dangle` | 代码风格（交给 Prettier） |
| **ES6+** | `prefer-const`、`prefer-arrow-callback`、`no-var` | 现代 JS 规范 |
| **TypeScript** | `@typescript-eslint/no-explicit-any`、`@typescript-eslint/consistent-type-imports` | TS 特有规范 |

**规则配置示例：**

```javascript
rules: {
    // 简单级别
    'no-eval': 'error',
    // 带选项
    'no-unused-vars': ['error', {
        vars: 'all',
        args: 'after-used',
        argsIgnorePattern: '^_',    // _ 前缀参数忽略
        varsIgnorePattern: '^_',    // _ 前缀变量忽略
    }],
    // 对象选项
    'max-lines': ['warn', {
        max: 300,
        skipBlankLines: true,
        skipComments: true,
    }],
}
```

### Prettier 核心选项和配置

**Prettier 配置文件支持格式：**

- `.prettierrc`（JSON/YAML）
- `.prettierrc.js` / `.prettierrc.cjs`（JS）
- `.prettierrc.json5`
- `prettier.config.js` / `prettier.config.cjs`
- `package.json` 中的 `prettier` 字段

**核心选项详解：**

```json
{
    "printWidth": 100,
    "tabWidth": 2,
    "useTabs": false,
    "semi": true,
    "singleQuote": true,
    "quoteProps": "as-needed",
    "jsxSingleQuote": false,
    "trailingComma": "all",
    "bracketSpacing": true,
    "bracketSameLine": false,
    "arrowParens": "always",
    "endOfLine": "lf",
    "singleAttributePerLine": false,
    "embeddedLanguageFormatting": "auto",
    "htmlWhitespaceSensitivity": "css"
}
```

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `printWidth` | 80 | 行宽限制，超出行将换行 |
| `tabWidth` | 2 | 缩进空格数 |
| `useTabs` | false | 使用 tab 而非空格缩进 |
| `semi` | true | 语句末尾加分号 |
| `singleQuote` | false | 使用单引号 |
| `quoteProps` | "as-needed" | 对象属性引号策略 |
| `jsxSingleQuote` | false | JSX 中使用单引号 |
| `trailingComma` | "all" | 尾逗号：`none` / `es5` / `all` |
| `bracketSpacing` | true | 对象花括号内加空格 `{ foo }` |
| `bracketSameLine` | false | JSX `>` 是否另起一行 |
| `arrowParens` | "always" | 箭头函数参数括号：`always` / `avoid` |
| `endOfLine` | "lf" | 换行符：`lf` / `crlf` / `cr` / `auto` |
| `singleAttributePerLine` | false | 单属性独占一行 |
| `htmlWhitespaceSensitivity` | "css" | HTML 空白敏感度 |

**Prettier 忽略文件（.prettierignore）：**

```
dist/
node_modules/
build/
coverage/
*.min.js
package-lock.json
```

### ESLint + Prettier 冲突与解决

**冲突根源：** ESLint 的风格规则（`quotes`、`semi`、`indent`、`comma-dangle` 等）和 Prettier 的格式化功能重叠，两者可能给出不同结果。

**典型冲突示例：**

```javascript
// ESLint 规则：semi: ['error', 'always']  要求分号
// Prettier 配置：semi: false              不加分号
// 结果：ESLint 报错要加分号，Prettier 格式化又去掉分号 → 死循环
```

**解决方案演进：**

| 阶段 | 方案 | 说明 |
|------|------|------|
| 早期 | `eslint-plugin-prettier` | 将 Prettier 规则作为 ESLint 规则运行 |
| 早期 | `eslint-config-prettier` | 关闭 ESLint 中与 Prettier 冲突的规则 |
| 早期 | 两者配合 | `eslint-config-prettier` + `eslint-plugin-prettier` |
| 当前推荐 | 仅 `eslint-config-prettier` | Prettier 独立格式化，ESLint 只管逻辑，不再将 Prettier 嵌入 ESLint |

**为什么不再推荐 `eslint-plugin-prettier`？**

- 将 Prettier 作为 ESLint 规则运行会显著降低 lint 速度
- 格式化问题被标记为 ESLint 错误，容易混淆
- Prettier 应该独立运行（编辑器保存时、lint-staged 中），不需要通过 ESLint

**`eslint-config-prettier` 关闭的规则列表（部分）：**

- `quotes`、`semi`、`indent`、`comma-dangle`、`comma-spacing`
- `no-mixed-spaces-and-tabs`、`no-tabs`、`object-curly-spacing`
- `max-len`、`wrap-iife`、`function-call-argument-newline`
- 以及 `@typescript-eslint/*` 中对应格式规则

### TypeScript ESLint 配置

TypeScript 项目需要特殊的 ESLint 配置，核心是 `typescript-eslint` 包（v8+ 整合了原 `@typescript-eslint/parser` 和 `@typescript-eslint/eslint-plugin`）。

```bash
npm install -D typescript-eslint @eslint/js eslint typescript
```

**关键配置项：**

| 配置 | 作用 | 推荐值 |
|------|------|--------|
| `parser` | TS 解析器 | `typescript-eslint` 内置 |
| `tseslint.configs.recommended` | 推荐 TS 规则集 | 基础必备 |
| `tseslint.configs.strict` | 更严格的 TS 规则 | 团队规范高时使用 |
| `tseslint.configs.stylistic` | TS 风格规则 | 不推荐（交给 Prettier） |

**类型感知规则（Type-Aware Rules）：**

部分规则需要完整的 TypeScript 类型信息，需配置 `tsconfig`：

```javascript
import tseslint from 'typescript-eslint';

export default tseslint.config(
    ...tseslint.configs.recommendedTypeChecked,  // 类型感知推荐规则
    {
        languageOptions: {
            parserOptions: {
                projectService: true,  // 自动发现 tsconfig
                tsconfigRootDir: import.meta.dirname,
            },
        },
        rules: {
            '@typescript-eslint/no-floating-promises': 'error',
            '@typescript-eslint/no-misused-promises': 'error',
            '@typescript-eslint/no-unnecessary-type-assertion': 'warn',
        },
    },
);
```

> 注意：类型感知规则比普通规则慢，因为需要完整的类型检查。CI 中可用 `--cache` 加速。

### husky + lint-staged 的 Git Hook 机制

**Git Hook 机制：**

```
代码修改 → git add → git commit → 触发 pre-commit hook → lint-staged → 通过 → 生成 commit
                                                          ↓ 失败
                                                     阻止提交
```

**husky 的工作原理：**

- Git 本身支持钩子（`.git/hooks/` 目录），但不会被版本控制
- husky 在 `.husky/` 目录创建钩子脚本，并通过 `core.hooksPath` 指向该目录
- 安装时执行 `git config core.hooksPath .husky`，使 Git 使用 `.husky/` 作为钩子目录

**lint-staged 的工作原理：**

- `lint-staged` 不检查所有文件，只检查 `git add` 后暂存区（staged）中的文件
- 通过 `git diff --cached --name-only --diff-filter=ACM` 获取暂存文件列表
- 对匹配 glob 模式的文件执行对应的 lint/format 命令
- 执行完后自动 `git add` 修复后的文件

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

### 为什么需要代码规范

**团队协作层面：**

- **统一代码风格**：消除代码评审中的风格争议，将精力集中在逻辑和架构上
- **降低认知负担**：风格一致的代码更易阅读和理解，新人上手更快
- **减少合并冲突**：格式化规则统一后，因格式差异产生的冲突大幅减少
- **知识传递**：规则即文档，代码规范隐含了团队的最佳实践

**代码质量层面：**

- **提前发现错误**：ESLint 可捕获 `no-undef`、`no-unreachable`、`no-dupe-keys` 等逻辑错误
- **避免反模式**：`no-eval`、`no-implicit-globals`、`no-with` 等规则防止危险写法
- **强制最佳实践**：`prefer-const`、`no-var`、`prefer-arrow-callback` 等推动现代 JS 写法
- **TypeScript 补充**：TS 类型检查无法覆盖的运行时问题，ESLint 规则可以

**可维护性层面：**

- **长期项目健康**：规范是项目的技术债务防火墙
- **自动化执行**：Git Hook + CI 确保规范被持续执行，不依赖人治
- **渐进式改进**：`warn` 级别允许渐进式修复，不阻断开发流程

### ESLint vs Prettier 的分工

| 维度 | ESLint | Prettier |
|------|--------|----------|
| **核心职责** | 代码质量检查 | 代码格式化 |
| **工作方式** | AST 分析，模式匹配 | AST 分析，重新打印 |
| **输出** | 错误/警告列表 | 格式化后的代码 |
| **可配置性** | 极高（数百条规则） | 极低（约 20 个选项） |
| **自动修复** | 部分规则支持 `--fix` | 全部格式化 |
| **哲学** | 可配置，灵活性 | Opinionated，零争论 |
| **典型检查** | 未使用变量、全局泄漏、eval 调用 | 缩进、引号、分号、换行 |
| **重叠区域** | 风格规则（应关闭） | 格式化（应独占） |

**一句话总结：ESLint 管"对不对"，Prettier 管"好不好看"。**

### Lint vs Format vs Type Check 的边界

| 维度 | Lint（ESLint） | Format（Prettier） | Type Check（TypeScript） |
|------|----------------|--------------------|-------------------------|
| **关注点** | 代码逻辑和模式 | 代码外观和排版 | 类型安全 |
| **检查内容** | 未使用变量、危险写法、最佳实践 | 缩进、引号、分号、行宽 | 类型兼容、空安全、泛型约束 |
| **能否自动修复** | 部分可以 | 全部可以 | 否（需手动修改） |
| **运行时机** | 编辑时/保存时/提交时 | 保存时/提交时 | 编译时/编辑时 |
| **必要性** | 必须 | 推荐 | TS 项目必须 |

**三者互补关系：**

```
Lint    → 发现逻辑问题（no-undef、no-eval、no-unused-vars）
Format  → 统一视觉风格（缩进、引号、分号）
Type    → 保证类型安全（类型推断、泛型约束、接口实现）
```

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
| 配置复杂度 | 高（两工具） | 低（单工具） | 中 |

**优缺点：**

- ✅ 优点：
  - ESLint 生态成熟，规则丰富
  - Prettier 格式化一致，零争论
  - 团队统一风格
  - 社区共享配置丰富
  - VSCode 集成良好
- ❌ 缺点：
  - 两个工具配合配置复杂
  - 大项目 lint 速度慢
  - 规则冲突需额外处理
  - Prettier 不够灵活，少数场景无法满足

## How — 怎么用

### 快速上手

```bash
npm install -D eslint prettier eslint-config-prettier
npx eslint --init
```

### 代码示例 1：ESLint 传统配置（.eslintrc.js）

```javascript
// .eslintrc.js
module.exports = {
    root: true,
    env: {
        browser: true,
        es2022: true,
        node: true,
    },
    parser: '@typescript-eslint/parser',
    parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module',
        ecmaFeatures: { jsx: true },
    },
    plugins: ['@typescript-eslint', 'react-hooks'],
    extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
        'plugin:react-hooks/recommended',
        'prettier',
    ],
    rules: {
        'no-console': ['warn', { allow: ['warn', 'error'] }],
        '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
        '@typescript-eslint/explicit-function-return-type': 'off',
        '@typescript-eslint/no-explicit-any': 'warn',
        'prefer-const': 'error',
        'no-var': 'error',
        'curly': ['error', 'multi-line'],
        'eqeqeq': ['error', 'always'],
    },
    overrides: [
        {
            files: ['**/*.test.{ts,tsx}'],
            env: { jest: true },
            rules: {
                '@typescript-eslint/no-explicit-any': 'off',
            },
        },
        {
            files: ['scripts/**/*.js'],
            rules: {
                '@typescript-eslint/no-var-requires': 'off',
            },
        },
    ],
    ignorePatterns: ['dist/', 'node_modules/', 'coverage/', '*.min.js'],
};
```

### 代码示例 2：ESLint Flat Config（eslint.config.js，ESLint 9+）

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactHooks from 'eslint-plugin-react-hooks';
import prettier from 'eslint-config-prettier';
import globals from 'globals';

export default tseslint.config(
    {
        ignores: ['dist/', 'node_modules/', '**/*.d.ts', 'coverage/'],
    },
    js.configs.recommended,
    ...tseslint.configs.recommended,
    reactHooks.configs.recommended,
    {
        files: ['**/*.{ts,tsx}'],
        languageOptions: {
            ecmaVersion: 'latest',
            sourceType: 'module',
            globals: {
                ...globals.browser,
                ...globals.node,
            },
        },
        rules: {
            'no-console': ['warn', { allow: ['warn', 'error'] }],
            '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
            '@typescript-eslint/no-explicit-any': 'warn',
            'prefer-const': 'error',
            'no-var': 'error',
            'eqeqeq': ['error', 'always'],
        },
    },
    {
        files: ['**/*.test.{ts,tsx}'],
        rules: {
            '@typescript-eslint/no-explicit-any': 'off',
        },
    },
    prettier,
);
```

### 代码示例 3：Prettier 配置文件

**.prettierrc.json（推荐）：**

```json
{
    "printWidth": 100,
    "tabWidth": 2,
    "useTabs": false,
    "semi": true,
    "singleQuote": true,
    "quoteProps": "as-needed",
    "jsxSingleQuote": false,
    "trailingComma": "all",
    "bracketSpacing": true,
    "bracketSameLine": false,
    "arrowParens": "always",
    "endOfLine": "lf",
    "singleAttributePerLine": true,
    "embeddedLanguageFormatting": "auto",
    "htmlWhitespaceSensitivity": "css"
}
```

**.prettierignore：**

```
dist/
node_modules/
build/
coverage/
*.min.js
*.min.css
package-lock.json
pnpm-lock.yaml
```

**Prettier CLI 常用命令：**

```bash
# 检查格式（不修改，仅报告）
npx prettier --check .

# 格式化所有文件
npx prettier --write .

# 只格式化特定文件
npx prettier --write "src/**/*.{ts,tsx}"

# 忽略某个目录
npx prettier --write . --ignore-path .prettierignore
```

### 代码示例 4：ESLint + Prettier 整合配置

**安装依赖：**

```bash
npm install -D eslint prettier eslint-config-prettier
```

**整合要点：**

1. `eslint-config-prettier` 必须放在 extends/config 数组最后
2. Prettier 负责格式化（保存时 + lint-staged）
3. ESLint 只负责代码质量检查
4. 不要使用 `eslint-plugin-prettier`

**eslintrc 整合：**

```javascript
// .eslintrc.js
module.exports = {
    extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
        // prettier 必须放最后
        'prettier',
    ],
};
```

**Flat Config 整合：**

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
import prettier from 'eslint-config-prettier';

export default tseslint.config(
    js.configs.recommended,
    ...tseslint.configs.recommended,
    // prettier 必须放最后
    prettier,
);
```

**package.json 脚本：**

```json
{
    "scripts": {
        "lint": "eslint . --report-unused-disable-directives --max-warnings 0",
        "lint:fix": "eslint . --fix",
        "format": "prettier --write .",
        "format:check": "prettier --check ."
    }
}
```

### 代码示例 5：TypeScript ESLint 配置

```bash
npm install -D typescript-eslint @eslint/js typescript
```

**基础配置（推荐规则集）：**

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
    { ignores: ['dist/'] },
    js.configs.recommended,
    ...tseslint.configs.recommended,
);
```

**严格配置（类型感知规则）：**

```javascript
// eslint.config.js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
    { ignores: ['dist/', '**/*.d.ts'] },
    js.configs.recommended,
    ...tseslint.configs.recommendedTypeChecked,
    {
        languageOptions: {
            parserOptions: {
                projectService: true,
                tsconfigRootDir: import.meta.dirname,
            },
        },
        rules: {
            // 类型安全
            '@typescript-eslint/no-floating-promises': 'error',
            '@typescript-eslint/no-misused-promises': 'error',
            '@typescript-eslint/no-unnecessary-type-assertion': 'warn',
            '@typescript-eslint/no-unnecessary-condition': 'warn',
            '@typescript-eslint/prefer-nullish-coalescing': 'warn',
            '@typescript-eslint/prefer-optional-chain': 'warn',
            // 代码质量
            '@typescript-eslint/consistent-type-imports': ['error', {
                prefer: 'type-imports',
                fixStyle: 'inline-type-imports',
            }],
            '@typescript-eslint/no-unused-vars': ['error', {
                argsIgnorePattern: '^_',
                varsIgnorePattern: '^_',
            }],
        },
    },
);
```

**tsconfig.json 配合：**

```json
{
    "compilerOptions": {
        "strict": true,
        "noUncheckedIndexedAccess": true,
        "noImplicitOverride": true
    },
    "include": ["src"],
    "exclude": ["node_modules", "dist"]
}
```

### 代码示例 6：husky + lint-staged 配置

**安装和初始化：**

```bash
npm install -D husky lint-staged
npx husky init
```

**husky 9+ 初始化后目录结构：**

```
.husky/
├── _
│   └── .gitignore
└── pre-commit       # Git pre-commit 钩子
```

**pre-commit 钩子（.husky/pre-commit）：**

```bash
npx lint-staged
```

**lint-staged 配置（package.json）：**

```json
{
    "lint-staged": {
        "*.{js,jsx,ts,tsx}": [
            "eslint --fix",
            "prettier --write"
        ],
        "*.{json,md,mdx,css,scss,less,yaml,yml}": [
            "prettier --write"
        ],
        "*.{svg}": [
            "prettier --write"
        ]
    }
}
```

**或独立配置文件（.lintstagedrc.js）：**

```javascript
// .lintstagedrc.js
export default {
    '*.{js,jsx,ts,tsx}': [
        'eslint --fix',
        'prettier --write',
    ],
    '*.{json,md,css,scss,yaml,yml}': [
        'prettier --write',
    ],
};
```

**commit-msg 钩子（配合 commitlint）：**

```bash
# .husky/commit-msg
npx --no -- commitlint --edit "$1"
```

**跳过钩子（紧急情况）：**

```bash
# 跳过所有钩子
git commit --no-verify -m "紧急修复"

# 仅跳过 pre-commit
HUSKY=0 git commit -m "紧急修复"
```

> 注意：`--no-verify` 应仅在紧急情况下使用，滥用会使规范形同虚设。

### 代码示例 7：自定义 ESLint 规则

```javascript
// rules/no-todo-comments.js
export default {
    meta: {
        type: 'suggestion',
        docs: {
            description: '禁止 TODO 注释（提醒处理待办事项）',
            category: 'Best Practices',
            recommended: false,
        },
        schema: [
            {
                type: 'object',
                properties: {
                    allowPattern: { type: 'string' },
                },
                additionalProperties: false,
            },
        ],
        messages: {
            unexpected: '发现 TODO 注释：{{ comment }}，请尽快处理。',
        },
    },
    create(context) {
        const options = context.options[0] || {};
        const allowPattern = options.allowPattern ? new RegExp(options.allowPattern) : null;

        return {
            // 匹配所有包含 TODO 的注释
            Program(node) {
                const sourceCode = context.sourceCode;
                const comments = sourceCode.getAllComments();

                comments.forEach((comment) => {
                    const value = comment.value.trim();
                    if (value.match(/\bTODO\b/i)) {
                        if (allowPattern && allowPattern.test(value)) return;
                        context.report({
                            loc: comment.loc,
                            messageId: 'unexpected',
                            data: { comment: value },
                        });
                    }
                });
            },
        };
    },
};
```

**在 Flat Config 中使用自定义规则：**

```javascript
// eslint.config.js
import noTodoComments from './rules/no-todo-comments.js';

export default [
    {
        plugins: {
            custom: {
                rules: {
                    'no-todo-comments': noTodoComments,
                },
            },
        },
        rules: {
            'custom/no-todo-comments': ['warn', { allowPattern: 'PERF-\\d+' }],
        },
    },
];
```

### 代码示例 8：VSCode 配置（settings.json）

**.vscode/settings.json（项目级，纳入版本控制）：**

```json
{
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit",
        "source.organizeImports": "never"
    },
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "typescript",
        "typescriptreact",
        "vue",
        "json",
        "jsonc"
    ],
    "eslint.useFlatConfig": true,
    "prettier.requireConfig": true,
    "typescript.tsdk": "node_modules/typescript/lib",
    "typescript.enablePromptUseWorkspaceTsdk": true,
    "files.eol": "\n",
    "files.insertFinalNewline": true,
    "files.trimTrailingWhitespace": true
}
```

**.vscode/extensions.json（推荐扩展）：**

```json
{
    "recommendations": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "editorconfig.editorconfig"
    ]
}
```

**EditorConfig 配置（.editorconfig）：**

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false
```

### 代码示例 9：commitlint 提交信息规范

**安装：**

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

**配置文件（commitlint.config.js）：**

```javascript
// commitlint.config.js
export default {
    extends: ['@commitlint/config-conventional'],
    rules: {
        'type-enum': [
            2,
            'always',
            [
                'feat',     // 新功能
                'fix',      // 修复 bug
                'docs',     // 文档变更
                'style',    // 代码格式（不影响代码运行的变动）
                'refactor', // 重构
                'perf',     // 性能优化
                'test',     // 测试
                'build',    // 构建或辅助工具变动
                'ci',       // CI 变动
                'chore',    // 其他变动
                'revert',   // 回退
            ],
        ],
        'subject-max-length': [2, 'always', 100],
        'subject-case': [0],  // 不限制大小写
    },
};
```

**提交信息格式：**

```
<type>(<scope>): <subject>

<body>

<footer>
```

**示例：**

```
feat(user): 添加用户头像上传功能

- 支持 JPG/PNG/WebP 格式
- 最大文件大小 5MB
- 自动压缩到 200x200

Closes #123
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| ESLint 和 Prettier 规则冲突 | ESLint 格式规则与 Prettier 不一致 | `eslint-config-prettier` 放配置最后 |
| lint-staged 卡住 | 文件太多或规则太多 | 只对暂存文件 lint，精简规则 |
| ESLint 找不到 TS 类型 | 缺少 parser 配置 | 使用 `typescript-eslint` parser |
| 保存格式化抖动 | ESLint 和 Prettier 交替格式化 | Prettier 负责格式化，ESLint 只检查逻辑 |
| Flat Config 不识别插件 | 插件引用方式变了 | 使用 `import` 导入插件对象，不再用字符串 |
| `eslint-config-prettier` 不生效 | 放置顺序不对 | 必须放在配置数组最后，确保覆盖前面规则 |
| husky 钩子不触发 | core.hooksPath 配置丢失 | 重新执行 `npx husky init` |
| Prettier 格式化后 ESLint 报错 | Prettier 配置与 ESLint 风格规则不一致 | 用 `eslint-config-prettier` 关闭 ESLint 风格规则 |
| CI 中 lint 超时 | 全量 lint 太慢 | 使用 `--cache` 和 `--cache-location` |
| `no-unused-vars` 误报 | 解构或回调参数 | 使用 `argsIgnorePattern: '^_'` 忽略下划线前缀 |
| 类型感知规则报错 `projectService` | tsconfig 不匹配 | 配置 `projectService: true` 或指定 `project` |
| monorepo 中 ESLint 配置重复 | 每个包独立配置 | 使用共享 ESLint 配置包 |

### 最佳实践

- Prettier 管格式，ESLint 管逻辑，职责不交叉
- `eslint-config-prettier` 必须放配置最后
- Git hook 自动 lint，不让不合规代码进入仓库
- `.vscode/settings.json` 纳入版本控制，团队统一
- 使用 Flat Config（ESLint 9+），不再使用传统 .eslintrc
- 不要使用 `eslint-plugin-prettier`，Prettier 独立运行
- 代码中使用 `// eslint-disable-next-line` 而非文件级 disable
- CI 中开启 `--max-warnings 0`，warn 也不允许
- 使用 `--cache` 加速 CI 中的 lint
- monorepo 中提取共享 ESLint 配置包
- commitlint 规范提交信息，配合 changelog 生成

## 面试题

**Q1: ESLint 和 Prettier 规则冲突怎么解决？**
> 使用 `eslint-config-prettier` 关闭所有与 Prettier 冲突的 ESLint 格式规则，且该配置必须放在 ESLint 配置数组的最后，确保覆盖前面的格式规则。核心原则：Prettier 管格式，ESLint 管逻辑，职责不交叉。不再推荐 `eslint-plugin-prettier`，因为它将 Prettier 作为 ESLint 规则运行，性能差且容易混淆。

**Q2: `eslint-disable` 注释有哪些使用方式？有什么风险？**
> 支持行级 `// eslint-disable-line`、块级 `/* eslint-disable */`、单规则 `// eslint-disable-line no-console` 三种方式。风险是滥用会掩盖真正的代码问题，应仅在必要场景使用（如第三方代码约束），且推荐指定具体规则名而非全量禁用。

**Q3: 如何自定义 ESLint 规则？**
> 编写自定义规则需创建一个插件，规则函数接收 `context` 对象，通过 `context.report()` 上报问题。规则基于 AST 节点访问器（`create` 函数中的 `Selector`）匹配代码模式，类似 Babel 插件的 visitor 机制。在 Flat Config 中，自定义规则通过 `plugins` 字段注册，使用 `plugin-name/rule-name` 引用。

**Q4: EditorConfig 的作用是什么？和 Prettier 有什么区别？**
> EditorConfig 跨编辑器统一基本编辑行为（缩进风格、缩进宽度、行尾符、字符编码等），作用于编辑器层面；Prettier 是代码格式化引擎，控制更细粒度的格式（分号、引号、行宽等）。两者有重叠配置需保持一致，EditorConfig 先生效，Prettier 再覆盖。

**Q5: ESLint 和 Prettier 的职责分别是什么？为什么需要配合使用？**
> ESLint 负责**代码质量检查**：检测逻辑错误、反模式、最佳实践违反等（如 `no-undef`、`no-unused-vars`、`no-eval`）。Prettier 负责**代码格式化**：统一缩进、引号、分号、行宽等视觉风格。需要配合使用是因为 ESLint 虽然有风格规则但不够完善且配置复杂，Prettier 的 opinionated 格式化零争论、效果一致。配合时用 `eslint-config-prettier` 关闭 ESLint 的格式规则，避免冲突。

**Q6: ESLint 的 Flat Config 是什么？和旧配置有什么区别？**
> Flat Config 是 ESLint 9+ 引入的新配置体系，使用 `eslint.config.js` 文件替代 `.eslintrc.*`。核心区别：（1）**配置格式**：仅支持 JS（ESM），不再支持 JSON/YAML；（2）**插件引用**：从字符串名变为直接导入对象；（3）**继承机制**：从 `extends` 字符串合并变为直接导入配置对象数组；（4）**环境**：从 `env` 字段变为 `languageOptions.globals`；（5）**覆盖**：从 `overrides` 变为独立配置对象 + `files` 匹配。Flat Config 更显式、更灵活、更易于 TypeScript 支持。

**Q7: lint-staged 的作用是什么？和全局 lint 有什么区别？**
> lint-staged 只对 Git 暂存区的文件执行 lint 和格式化，而全局 lint（如 `eslint .`）会检查整个项目。核心区别：（1）**速度**：lint-staged 只检查改动的文件，速度快几个数量级；（2）**时机**：lint-staged 在 git commit 时自动运行，全局 lint 通常在 CI 或手动执行；（3）**范围**：lint-staged 不检查无关文件，避免历史代码问题阻断当前提交。推荐两者结合：lint-staged 用于日常开发体验，全局 lint 用于 CI 质量门禁。

**Q8: 如何在 CI 中集成代码规范检查？**
> CI 中通常在 lint 和 format 检查阶段集成：（1）**ESLint 检查**：`npx eslint . --max-warnings 0`，`--max-warnings 0` 确保 warn 也阻断流水线；（2）**Prettier 检查**：`npx prettier --check .`，只检查不修改；（3）**TypeScript 检查**：`npx tsc --noEmit`，纯类型检查不生成文件；（4）**加速**：使用 `--cache` 和 `--cache-location` 缓存未修改文件的检查结果；（5）**GitHub Actions 示例**：在 `pull_request` 事件中运行上述命令，任一失败则阻止合并。

---

**相关链接：**
- [[Webpack与Vite]]
- [[Babel与AST]]
- [[TypeScript类型系统]]
- [[Git工作流与规范]]
- [[CI与CD持续集成]]

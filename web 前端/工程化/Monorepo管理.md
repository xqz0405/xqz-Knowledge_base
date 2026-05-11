---
tags:
  - Web前端
  - 工程化
  - Monorepo
  - pnpm
  - Turborepo
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Monorepo 管理

## What — 什么是 Monorepo

Monorepo 是将多个相关项目（包、应用、库）放在同一个 Git 仓库中管理的策略，共享工具链、依赖和 CI 配置。

### Monorepo vs Multirepo

| 维度 | Monorepo | Multirepo |
|------|----------|-----------|
| 代码位置 | 一个仓库 | 多个仓库 |
| 包共享 | 直接引用 | 发布 npm |
| 原子提交 | 一个 PR 改多个包 | 多个 PR 跨仓库 |
| 构建缓存 | 跨包共享 | 各自独立 |
| CI/CD | 统一流水线 | 每个仓库独立 |
| 权限控制 | 统一 | 精细 |

### 核心工具

| 工具 | 定位 | 特点 |
|------|------|------|
| pnpm workspace | 包管理 + 工作区 | 硬链接节省磁盘，workspace 协议 |
| Turborepo | 构建编排 | 增量构建 + 本地/远程缓存 |
| Nx | 全功能平台 | 依赖图分析 + 增量构建 + 插件 |
| Lerna | 历史方案 | 发布管理（已被 Nx 接管） |
| Changesets | 版本管理 | 多包联动发版 |

---

## Why — 为什么选择 Monorepo

### 1. 代码共享零延迟

包之间直接 `import { Button } from '@my/ui'`，无需 `npm publish` 再 `npm install`，修改即生效。

### 2. 原子提交

一个 PR 同时修改组件库和消费应用，确保兼容性，不会出现"组件库更新了但应用还没适配"的问题。

### 3. 统一工具链

ESLint、TypeScript、Prettier、CI 配置全仓库共享，无需每个仓库重复配置。

### 4. 增量构建

Turborepo / Nx 自动分析依赖图，只构建变化的包及其下游，大型仓库构建时间从 30 分钟降到 2 分钟。

---

## How — 怎么用

### 1. pnpm Workspace 搭建

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

```json
// package.json（根目录）
{
  "name": "my-monorepo",
  "private": true,
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "turbo lint"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.5.0"
  }
}
```

```
my-monorepo/
├── apps/
│   ├── web/            — Web 应用
│   ├── admin/          — 后台管理
│   └── mobile/         — 移动端
├── packages/
│   ├── ui/             — 组件库
│   ├── utils/          — 工具库
│   ├── config/         — 共享配置（ESLint / TS）
│   └── api-client/     — API 客户端
├── pnpm-workspace.yaml
├── turbo.json
├── package.json
└── tsconfig.json
```

### 2. 包之间的引用

```json
// apps/web/package.json
{
  "name": "@my/web",
  "dependencies": {
    "@my/ui": "workspace:*",
    "@my/utils": "workspace:*",
    "@my/api-client": "workspace:*"
  }
}
```

```json
// packages/ui/package.json
{
  "name": "@my/ui",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": { "import": "./dist/index.mjs", "require": "./dist/index.js", "types": "./dist/index.d.ts" },
    "./button": { "import": "./dist/button.mjs", "types": "./dist/button.d.ts" }
  },
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "dev": "tsup src/index.ts --format esm,cjs --watch"
  },
  "peerDependencies": {
    "react": ">=18"
  }
}
```

`workspace:*` 在开发时链接到本地包，发布时自动替换为真实版本号。

### 3. Turborepo 配置

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"]
    },
    "type-check": {
      "dependsOn": ["^build"]
    }
  }
}
```

**任务依赖说明**：

| 配置 | 含义 |
|------|------|
| `"dependsOn": ["^build"]` | 先构建所有依赖包，再构建当前包 |
| `"dependsOn": ["build"]` | 先执行当前包的 build，再执行当前任务 |
| `"outputs": ["dist/**"]` | 缓存输出的文件 |
| `"persistent": true` | 长驻进程（如 dev server） |
| `"cache": false` | 不缓存（如 dev） |

**Turborepo 缓存原理**：

```
输入哈希 = hash(源文件 + 依赖包输出 + 环境变量 + turbo.json)
→ 缓存命中？→ 直接复制缓存输出
→ 缓存未命中？→ 执行任务 → 存入缓存
```

### 4. 共享 TypeScript 配置

```json
// packages/config/tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

```json
// packages/ui/tsconfig.json
{
  "extends": "@my/config/tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

### 5. 共享 ESLint 配置

```js
// packages/config/eslint/index.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/no-explicit-any': 'warn',
  },
}
```

### 6. Changesets — 多包发版

```bash
pnpm add -Dw @changesets/cli
npx changeset init
```

```json
// .changeset/config.json
{
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch"
}
```

```bash
# 1. 创建变更记录
npx changeset
# 交互式选择：哪些包有变更？是 patch/minor/major？变更说明？

# 2. 消费变更记录，更新版本号和 CHANGELOG
npx changeset version

# 3. 发布
pnpm changeset publish
```

**CI 自动发版**：

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'pnpm' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - run: pnpm test
      - name: Create Release Pull Request or Publish
        uses: changesets/action@v1
        with: { publish: pnpm changeset publish }
        env: { GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}, NPM_TOKEN: ${{ secrets.NPM_TOKEN }} }
```

### 7. Turborepo 远程缓存

```bash
# 启用 Vercel 远程缓存（免费）
npx turbo login
npx turbo link

# 现在 CI 的构建结果会缓存到 Vercel
# 本地开发也能命中 CI 的缓存
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 循环依赖 | 包 A 依赖包 B，包 B 也依赖包 A | 重构提取共享部分到包 C |
| 构建顺序错 | 没有正确配置 `dependsOn` | 确保 `^build` 声明依赖 |
| 缓存失效 | 源文件没变但输出不同 | 检查 `globalDependencies`，是否遗漏环境变量 |
| 发布版本不一致 | workspace 协议未正确解析 | 用 Changesets 管理版本 |
| IDE 跳转不对 | TypeScript 项目引用未配置 | 配置 `tsconfig.json` 的 `references` |

### 最佳实践

1. **workspace 协议**：包间依赖用 `workspace:*`，确保本地链接。
2. **构建顺序交给 Turborepo**：不要手动编排构建顺序，用 `dependsOn`。
3. **Changesets 管理发版**：多包联动发版，避免手动改版本号。
4. **共享配置包**：TypeScript、ESLint、Prettier 配置提取到 `packages/config`。
5. **远程缓存**：开启 Turborepo 远程缓存，CI 和本地共享构建结果。

---

## 面试题

### 1. pnpm 的 workspace 协议是什么？和 npm link 有什么区别？

**答**：`workspace:*` 是 pnpm workspace 的协议，在开发时将依赖链接到本地工作区包，发布时自动替换为真实版本号。与 `npm link` 的区别：(1) **自动化**——`workspace:*` 声明在 package.json 中，安装即链接，无需手动执行命令；(2) **发布安全**——发布时 pnpm 自动将 `workspace:*` 替换为实际版本号（如 `1.2.3`），而 `npm link` 创建的是符号链接，发布时可能导致包引用了一个本地路径；(3) **一致性**——所有开发者 clone 后 `pnpm install` 即获得正确的链接，无需额外操作。

---

### 2. Turborepo 的增量构建是怎么实现的？

**答**：基于输入哈希的缓存机制：(1) 为每个 task 计算输入哈希——源文件内容（git hash）、依赖包的 task 输出（通过 `dependsOn: ["^build"]` 递归追踪）、环境变量、turbo.json 配置；(2) 哈希相同 → 缓存命中 → 直接复制 `outputs` 目录中的文件，跳过执行；(3) 哈希不同 → 执行任务 → 将 outputs 存入缓存。远程缓存（Vercel）让 CI 的构建结果可以共享给本地开发者。效果：修改 `@my/ui` 只会重建 `@my/ui` + 依赖它的 `@my/web`，其他包直接用缓存。

---

### 3. Monorepo 中如何避免循环依赖？

**答**：循环依赖（A 依赖 B，B 依赖 A）会导致构建失败和不可预测的行为。避免方法：(1) **架构分层**——明确定义依赖方向（apps → features → entities → shared），不允许反向依赖；(2) **提取共享模块**——如果 A 和 B 都需要对方的某个功能，将共享部分提取到 C 包；(3) **工具检测**——使用 `madge` 或 `dependency-cruiser` 自动检测循环依赖，集成到 CI；(4) **Nx 的模块边界规则**——配置 `enforce-module-boundaries` ESLint 规则，在开发时阻止非法依赖。

---

### 4. Changesets 和 lerna version 有什么区别？

**答**：Changesets 和 lerna version 都管理 Monorepo 的多包发版，但机制不同：(1) **记录方式**——Changesets 在 `.changeset/` 目录中创建独立的 Markdown 文件记录变更，lerna version 通过 git commit 信息推断版本变化；(2) **灵活性**——Changesets 允许每次变更独立选择包和版本级别（patch/minor/major），lerna version 通常统一处理；(3) **CI 集成**——Changesets 有官方 GitHub Action，自动创建 Release PR，审核后自动发布；lerna 需要 `--conventional-commits` 配合；(4) **维护状态**——Changesets 活跃维护，lerna 已停止维护（被 Nx 接管）。新项目推荐 Changesets。

---

### 5. pnpm 的硬链接机制为什么能节省磁盘空间？

**答**：pnpm 将所有包安装在全局存储（`~/.pnpm-store/`），项目中的 `node_modules` 通过硬链接指向全局存储。同一个包（如 React 18.2.0）在全局只存一份，10 个项目使用 React 也只占一份空间。而 npm/yarn 每个项目独立安装，10 个项目 = 10 份 React。硬链接是文件系统级别的——多个目录条目指向同一个磁盘数据块，不占用额外空间。pnpm 还使用符号链接创建非扁平的 `node_modules` 结构，解决幽灵依赖问题（只能 import 声明过的依赖）。

---

## 相关链接

- [pnpm Workspace](https://pnpm.io/workspaces)
- [Turborepo 官方文档](https://turbo.build/repo)
- [Nx 官方文档](https://nx.dev/)
- [Changesets](https://github.com/changesets/changesets)

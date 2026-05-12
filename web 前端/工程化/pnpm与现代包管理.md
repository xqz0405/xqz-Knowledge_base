---
tags:
  - Web前端
  - 工程化
  - pnpm
  - 包管理
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# pnpm与现代包管理

## What — 是什么

> pnpm（performant npm）是高性能的 Node.js 包管理器，通过硬链接 + 符号链接机制解决 npm/yarn 的幽灵依赖、磁盘浪费和安装速度问题，已成为 Monorepo 场景下的首选方案。

### 包管理器演进

| 阶段 | 工具 | 年份 | 核心突破 |
|------|------|------|----------|
| 第一代 | npm v1-v2 | 2010 | 嵌套安装，每个依赖独立 node_modules |
| 第二代 | npm v3+ | 2015 | 扁平化安装，缓解路径过长问题 |
| 第二代 | yarn v1 | 2016 | 离线缓存、确定性安装（lockfile）、并行下载 |
| 第三代 | pnpm | 2017 | 硬链接 + 符号链接、严格隔离、内容寻址存储 |
| 第三代 | yarn Berry (v2+) | 2019 | Plug'n'Play、Zero-Install |
| 官方方案 | Corepack | 2022 | Node.js 内置包管理器管理器 |

### node_modules 结构演进

**1. 嵌套结构（npm v1-v2）**

```
node_modules/
├── A/
│   └── node_modules/
│       └── lodash@3.0.0    — A 自己的 lodash
└── B/
    └── node_modules/
        └── lodash@4.0.0    — B 自己的 lodash
```

问题：依赖嵌套过深导致 Windows 路径超 260 字符限制；同一个包重复安装。

**2. 扁平结构（npm v3+ / yarn v1）**

```
node_modules/
├── A/
├── B/
└── lodash@4.0.0            — 提升到顶层，A 和 B 共享
    └── node_modules/
        └── lodash@3.0.0    — 版本冲突时才嵌套
```

问题：解决了路径过长，但引入了**幽灵依赖**——未声明的包因为提升而可以被访问。

**3. pnpm 符号链接结构**

```
node_modules/
├── .pnpm/                          — 硬链接指向全局存储
│   ├── lodash@4.0.0/
│   │   └── node_modules/
│   │       └── lodash/             — 真实文件（硬链接）
│   └── A@1.0.0/
│       └── node_modules/
│           ├── A/                  — 硬链接
│           └── lodash/ → ../../lodash@4.0.0/node_modules/lodash  — 符号链接
├── A/ → .pnpm/A@1.0.0/node_modules/A      — 符号链接
└── B/ → .pnpm/B@1.0.0/node_modules/B      — 符号链接
```

关键区别：`node_modules/A` 只是符号链接，A 的真实依赖只能通过 `.pnpm/A@1.0.0/node_modules/` 访问，实现了严格隔离。

### pnpm 核心机制

**1. 硬链接（Hard Link）**

硬链接是文件系统级别的概念——多个目录条目指向同一个磁盘数据块。pnpm 将包文件硬链接到全局存储，不复制文件内容，因此：

- 多个项目共用同一份包文件，磁盘占用接近零增量
- 修改一个项目的 node_modules 中的文件不会影响其他项目（因为硬链接共享 inode，写时需要特殊处理）

**2. 内容寻址存储（Content-Addressable Store）**

```
~/.pnpm-store/v3/
├── files/
│   ├── 00/                          — 基于文件内容哈希的目录
│   │   ├── d4a1b2c3e4f5...          — 文件内容（硬链接源）
│   │   └── ...
│   ├── 01/
│   └── ...
└── metadata/
```

- 存储路径基于文件内容的 SHA512 哈希
- 相同内容的文件只存储一份（去重）
- 不同版本的包如果共享相同文件（如 README、LICENSE），也只存一份

**3. .pnpm 目录**

`.pnpm` 是 pnpm 的"真实"依赖图存储区：

- 每个包以其 `name@version` 创建独立目录
- 目录内的 `node_modules` 包含该包的所有依赖（符号链接指向 `.pnpm` 中对应位置）
- 顶层的 `node_modules` 只包含直接依赖的符号链接

**4. 符号链接图（Symlink Graph）**

符号链接建立两层映射关系：

- **外层**：`node_modules/pkg` → `.pnpm/pkg@ver/node_modules/pkg`（项目可访问的入口）
- **内层**：`.pnpm/pkg@ver/node_modules/dep` → `.pnpm/dep@ver/node_modules/dep`（包的依赖指向）

这种双层结构确保了：只能 import 声明过的依赖，但依赖的依赖也能正确解析。

---

## Why — 为什么

### 1. 幽灵依赖（Phantom Dependencies）

**问题描述**：在 npm/yarn 的扁平结构中，未在 `package.json` 中声明的包也可以被 import。

```javascript
// package.json 只声明了 A
{
  "dependencies": {
    "A": "^1.0.0"    // A 依赖了 lodash
  }
}

// 但代码中可以直接使用 lodash（因为提升到了顶层）
import _ from 'lodash';  // 能运行，但 A 升级或移除后就报错
```

**危害**：
- A 升级后可能改用不同的 lodash 版本，代码静默使用错误版本
- A 移除后代码直接崩溃，但 `package.json` 中从未声明过 lodash
- 依赖关系不透明，难以排查问题来源

**pnpm 的解决**：符号链接结构使得 `node_modules` 根目录只有直接声明的依赖，`import 'lodash'` 会直接报错 `Cannot find module 'lodash'`。

### 2. npm 扁平化的缺陷

| 缺陷 | 说明 |
|------|------|
| 不确定性 | 相同 `package.json` 可能产生不同拓扑（依赖安装顺序影响提升结果） |
| 幽灵依赖 | 提升导致未声明包可访问 |
| 版本冲突 | 多个版本只能提升一个，其他嵌套 |
| 磁盘浪费 | 同一个包在不同项目中重复存储 |

### 3. 磁盘空间节省

```
场景：10 个项目都使用 React 18.2.0 + Lodash 4.17.21

npm/yarn：  10 × (React + Lodash) ≈ 10 × 5MB = 50MB
pnpm：      1 × (React + Lodash) ≈ 5MB（硬链接，10 个项目共享）
节省比例：  约 90%
```

实际数据：一个中型团队迁移到 pnpm 后，全局存储约 2GB，但支撑了 30+ 项目总计约 45GB 的 `node_modules`。

### 4. 安装速度

pnpm 的速度优势来自三个方面：

| 阶段 | npm/yarn | pnpm |
|------|----------|------|
| 下载 | 下载到缓存，复制到 node_modules | 下载到全局存储，硬链接到 node_modules |
| 写入 | 复制文件（慢，涉及磁盘 I/O） | 硬链接（快，只创建目录条目） |
| 重复包 | 每个项目都要复制 | 直接硬链接，跳过下载和写入 |

硬链接的创建速度接近 `ln` 系统调用，远快于文件复制。

### 5. Monorepo 原生支持

pnpm workspace 提供一等公民级别的 Monorepo 支持：

- `pnpm-workspace.yaml` 声明工作区
- `workspace:*` 协议链接本地包
- `--filter` 过滤执行命令的包
- 原生支持包间的依赖拓扑排序

### 包管理器对比总表

| 维度 | npm | yarn v1 | pnpm | yarn Berry (v2+) |
|------|-----|---------|------|-------------------|
| 安装速度 | 慢 | 中 | 快 | 快 |
| 磁盘占用 | 高 | 高 | 低（硬链接去重） | 低（PnP 无 node_modules） |
| 幽灵依赖 | 有 | 有 | 无（严格隔离） | 无（PnP 模式） |
| Workspaces | 支持 | 支持 | 支持（原生优先） | 支持 |
| Plug'n'Play | 不支持 | 不支持 | 不支持 | 支持 |
| Node 兼容 | 官方标准 | 完全兼容 | 完全兼容 | 需要适配（PnP） |
| 确定性安装 | npm v5+（lockfile） | 是（lockfile） | 是（lockfile） | 是 |
| 离线安装 | 缓存支持 | 支持 | 支持 | 支持（Zero-Install） |
| Monorepo 支持 | 基础 | 基础 | 强（filter/workspace 协议） | 强 |
| 学习成本 | 低 | 低 | 中 | 高（PnP 概念） |

---

## How — 怎么用

### 1. pnpm 安装与配置

```bash
# 方式一：npm 全局安装
npm install -g pnpm

# 方式二：Corepack 启用（Node.js 16.9+ 自带）
corepack enable
corepack prepare pnpm@latest --activate

# 方式三：独立安装脚本
# Windows (PowerShell)
iwr https://get.pnpm.io/install.ps1 -useb | iex
# macOS/Linux
curl -fsSL https://get.pnpm.io/install.sh | sh -

# 验证安装
pnpm --version
```

**.npmrc 全局配置**：

```ini
# ~/.npmrc（全局）

# 全局存储路径（默认 ~/.pnpm-store）
store-dir=~/.pnpm-store

# 国内镜像
registry=https://registry.npmmirror.com

# 严格 peer 依赖
strict-peer-dependencies=true
```

**查看和清理全局存储**：

```bash
# 查看存储路径
pnpm store path
# 输出：/home/user/.pnpm-store/v3

# 清理未被任何项目引用的包
pnpm store prune

# 查看存储统计
pnpm store status
```

### 2. pnpm add / install / remove

```bash
# 安装所有依赖（等同 npm install）
pnpm install

# 添加生产依赖
pnpm add react react-dom

# 添加开发依赖
pnpm add -D typescript @types/node

# 添加 peer 依赖
pnpm add --save-peer react

# 添加全局包
pnpm add -g pm2

# 安装特定版本
pnpm add lodash@4.17.21

# 移除依赖
pnpm remove lodash

# 移除开发依赖
pnpm remove -D eslint
```

**--filter 过滤执行**：

```bash
# 只在 @my/web 包中安装依赖
pnpm add lodash --filter @my/web

# 在 @my/web 及其所有依赖包中执行 build
pnpm build --filter @my/web...

# 在 @my/web 的所有依赖包中执行 build（不含自身）
pnpm build --filter ...@my/web

# 在匹配 glob 的包中执行
pnpm build --filter "./packages/**"

# 排除特定包
pnpm build --filter "!@my/mobile"

# 组合使用：@my/web 及其依赖，但排除 @my/config
pnpm build --filter @my/web... --filter "!@my/config"
```

**filter 语法速查**：

| 语法 | 含义 |
|------|------|
| `--filter @my/web` | 只在 @my/web 执行 |
| `--filter @my/web...` | @my/web + 它的所有依赖 |
| `--filter ...@my/web` | @my/web 的所有依赖者（被谁依赖） |
| `--filter "./packages/**"` | glob 匹配包路径 |
| `--filter "!@my/mobile"` | 排除 |

### 3. pnpm workspace

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
  - 'utils/*'
```

**多包项目结构**：

```
my-project/
├── apps/
│   ├── web/
│   │   └── package.json          — "@my/web"
│   └── admin/
│       └── package.json          — "@my/admin"
├── packages/
│   ├── ui/
│   │   └── package.json          — "@my/ui"
│   └── utils/
│       └── package.json          — "@my/utils"
├── pnpm-workspace.yaml
└── package.json                   — 根目录（private: true）
```

**workspace 协议**：

```json
// apps/web/package.json
{
  "name": "@my/web",
  "dependencies": {
    "@my/ui": "workspace:*",
    "@my/utils": "workspace:^"
  }
}
```

| 协议 | 开发时 | 发布时替换为 |
|------|--------|-------------|
| `workspace:*` | 链接到本地最新版本 | 精确版本 `1.2.3` |
| `workspace:^` | 链接到本地最新版本 | 兼容范围 `^1.2.3` |
| `workspace:~` | 链接到本地最新版本 | 补丁范围 `~1.2.3` |
| `workspace:^1.2.3` | 本地版本需满足范围，否则链接 | 保持 `^1.2.3` |

**workspace 常用命令**：

```bash
# 递归执行所有包的 build
pnpm -r build

# 递归执行，带拓扑排序（依赖先构建）
pnpm -r --sort build

# 只在某个包及其依赖中执行
pnpm -r --filter @my/web... build

# 查看工作区所有包
pnpm ls -r --depth 0

# 递归安装所有包的依赖
pnpm install
```

### 4. pnpm 的 node_modules 结构解析

以一个安装了 `express@4.18.2` 的项目为例：

```
node_modules/
├── .pnpm/
│   ├── express@4.18.2/
│   │   └── node_modules/
│   │       ├── express/          ← 硬链接到全局存储
│   │       ├── body-parser/ → ../../body-parser@1.20.1/node_modules/body-parser
│   │       ├── cookie/ → ../../cookie@0.5.0/node_modules/cookie
│   │       └── ...               ← express 的所有依赖（符号链接）
│   ├── body-parser@1.20.1/
│   │   └── node_modules/
│   │       ├── body-parser/      ← 硬链接
│   │       └── ...
│   └── cookie@0.5.0/
│       └── node_modules/
│           └── cookie/           ← 硬链接
├── express/ → .pnpm/express@4.18.2/node_modules/express   ← 符号链接
└── .modules.yaml                  ← pnpm 元数据
```

**严格隔离的效果**：

```javascript
// 项目只声明了 express
{
  "dependencies": { "express": "^4.18.2" }
}

// 可以正常访问
const express = require('express');   // OK

// 无法访问 express 的依赖（幽灵依赖被阻止）
const cookie = require('cookie');     // Error: Cannot find module 'cookie'

// 必须显式声明
// pnpm add cookie
const cookie = require('cookie');     // OK
```

### 5. .npmrc 常用配置

```ini
# 项目级 .npmrc（放在项目根目录）

# 提升所有依赖到根目录（类似 npm 扁平化，牺牲严格隔离换取兼容性）
shamefully-hoist=true

# 严格检查 peer 依赖（默认 false，建议 true）
strict-peer-dependencies=true

# 自动安装 peer 依赖（pnpm v7+ 默认 true）
auto-install-peers=true

# 只提升指定包（精确控制，比 shamefully-hoist 粒度更细）
hoist-pattern[]=*eslint*
hoist-pattern[]=*prettier*

# 不提升指定包到根目录
public-hoist-pattern[]=*types*

# 使用国内镜像
registry=https://registry.npmmirror.com

# 指定 Node.js 版本范围（不满足时警告）
use-node-version=18

# 保存时使用精确版本号（不用 ^/~）
save-exact=true

# 忽略 scripts（安全场景，如 CI 中防范恶意 postinstall）
ignore-scripts=true
```

**shamefully-hoist 详解**：

```ini
# 不提升（默认，严格隔离）
shamefully-hoist=false
# node_modules/ 只有直接依赖

# 全部提升
shamefully-hoist=true
# node_modules/ 结构接近 npm，所有包可见
# 适用于兼容性差的旧项目

# 精确控制（推荐）
public-hoist-pattern[]=*eslint*
public-hoist-pattern[]=*prettier*
# 只有 eslint/prettier 相关包提升，其他保持严格隔离
```

### 6. pnpm patch 修改第三方包

```bash
# 1. 编辑包的补丁
pnpm patch lodash@4.17.21

# 输出：
# You can now edit the following folder: /tmp/abc123/lodash@4.17.21
#
# Once you're done with your changes, run:
# pnpm patch-commit /tmp/abc123/lodash@4.17.21
```

```javascript
// 2. 修改临时目录中的文件
// /tmp/abc123/lodash@4.17.21/lodash.js

// 原始代码
function debounce(func, wait) {
  // ...
}

// 修改为
function debounce(func, wait, immediate) {
  // 添加 immediate 参数支持
  // ...
}
```

```bash
# 3. 提交补丁
pnpm patch-commit /tmp/abc123/lodash@4.17.21
```

补丁提交后，pnpm 会在 `package.json` 中添加 `pnpm.patchedDependencies` 字段：

```json
{
  "pnpm": {
    "patchedDependencies": {
      "lodash@4.17.21": "patches/lodash@4.17.21.patch"
    }
  }
}
```

同时生成补丁文件 `patches/lodash@4.17.21.patch`，可以提交到 Git，团队成员 `pnpm install` 时自动应用。

### 7. pnpm deploy 部署子集

在 Monorepo 中，只部署某个应用及其依赖，不部署整个仓库：

```bash
# 将 @my/web 及其所有依赖导出到 dist 目录
pnpm deploy --filter @my/web ./dist
```

```json
// apps/web/package.json 中可以指定要包含的依赖
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild"]
  }
}
```

```bash
# deploy 配合 Docker 使用
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY dist/ .
RUN pnpm install --prod --frozen-lockfile
CMD ["node", "server.js"]
```

```bash
# 构建并部署
pnpm deploy --filter @my/web ./deploy
cd deploy
docker build -t my-web .
```

### 8. pnpm store 管理

```bash
# 查看存储路径
pnpm store path
# 输出：C:\Users\user\AppData\Local\pnpm-store\v3
# 或：/home/user/.local/share/pnpm-store/v3

# 清理未被引用的包
pnpm store prune
# 扫描所有项目，移除不被任何项目引用的包

# 检查存储中包的完整性
pnpm store status
# 检查硬链接是否有效，文件是否被篡改

# 修改存储路径（.npmrc）
# store-dir=/custom/path/pnpm-store
```

**存储路径的跨平台差异**：

| 平台 | 默认路径 |
|------|----------|
| macOS | `~/Library/pnpm/store/v3` |
| Linux | `~/.local/share/pnpm/store/v3` |
| Windows | `%LOCALAPPDATA%/pnpm/store/v3` |

### 9. Corepack 使用

Corepack 是 Node.js 16.9+ 自带的包管理器管理器，无需全局安装 pnpm/yarn。

```bash
# 启用 Corepack
corepack enable

# 指定项目使用特定版本的 pnpm
corepack use pnpm@9
# 会在 package.json 中写入 "packageManager": "pnpm@9.x.x"

# 准备特定版本（预下载）
corepack prepare pnpm@9.1.0 --activate

# 禁用 Corepack（回退到全局安装的包管理器）
corepack disable

# 查看当前激活的包管理器
corepack --version
```

**package.json 中声明包管理器版本**：

```json
{
  "name": "my-project",
  "packageManager": "pnpm@9.1.0+sha256.86d4189345374d72f7d5e9e0e375e6e9e9f2848e6e6e6e6e6e6e6e6e6e6e6e6"
}
```

Corepack 会根据 `packageManager` 字段自动使用对应版本的 pnpm，确保团队版本一致性。

**CI 环境中使用 Corepack**：

```yaml
# GitHub Actions
- uses: actions/setup-node@v4
  with:
    node-version: 20
- run: corepack enable
- run: pnpm install --frozen-lockfile
```

### 10. 从 npm/yarn 迁移到 pnpm

**Step 1：安装 pnpm 并初始化**

```bash
npm install -g pnpm

# 在项目根目录执行
pnpm import
# 自动根据 package-lock.json 或 yarn.lock 生成 pnpm-lock.yaml
```

**Step 2：清理旧文件**

```bash
# 删除旧的 node_modules 和 lockfile
rm -rf node_modules package-lock.json yarn.lock
```

**Step 3：安装依赖**

```bash
pnpm install
```

**Step 4：处理兼容性问题**

```ini
# .npmrc — 如果遇到幽灵依赖报错
shamefully-hoist=true

# 如果只是个别包有问题，用精确提升
public-hoist-pattern[]=*problematic-package*
```

**package.json scripts 兼容**：

```json
{
  "scripts": {
    "preinstall": "npx only-allow pnpm",
    "dev": "pnpm -r --parallel dev",
    "build": "pnpm -r --sort build",
    "lint": "pnpm -r lint",
    "test": "pnpm -r test"
  },
  "engines": {
    "pnpm": ">=9.0.0"
  }
}
```

`preinstall` 脚本确保团队成员只能使用 pnpm 安装依赖，防止混用 npm。

**从 yarn v1 迁移的注意事项**：

| 差异点 | yarn v1 | pnpm |
|--------|---------|------|
| 命令 | `yarn` | `pnpm install` |
| 添加依赖 | `yarn add pkg` | `pnpm add pkg` |
| 运行脚本 | `yarn dev` | `pnpm dev` |
| 全局安装 | `yarn global add` | `pnpm add -g` |
| 升级交互 | `yarn upgrade-interactive` | `pnpm update -i` |

### 11. Monorepo 实战（Turborepo + pnpm）

**项目结构**：

```
my-monorepo/
├── apps/
│   ├── web/                    — Next.js 应用
│   │   ├── package.json
│   │   ├── next.config.js
│   │   └── src/
│   ├── docs/                   — 文档站点
│   │   ├── package.json
│   │   └── src/
│   └── admin/                  — 后台管理
│       ├── package.json
│       └── src/
├── packages/
│   ├── ui/                     — 共享组件库
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   ├── utils/                  — 工具函数库
│   │   ├── package.json
│   │   └── src/
│   ├── tsconfig/               — 共享 TS 配置
│   │   ├── base.json
│   │   ├── nextjs.json
│   │   └── react-library.json
│   └── eslint-config/          — 共享 ESLint 配置
│       ├── index.js
│       └── next.js
├── pnpm-workspace.yaml
├── turbo.json
├── package.json
└── .npmrc
```

**配置文件**：

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
    "lint": "turbo lint",
    "clean": "turbo clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.5.0"
  },
  "packageManager": "pnpm@9.1.0"
}
```

```ini
# .npmrc
shamefully-hoist=true
strict-peer-dependencies=false
```

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
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
    }
  }
}
```

**包间引用示例**：

```json
// packages/ui/package.json
{
  "name": "@my/ui",
  "version": "0.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "dev": "tsup src/index.ts --format esm,cjs --watch"
  },
  "dependencies": {
    "@my/utils": "workspace:*",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0"
  },
  "peerDependencies": {
    "react": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "react": "^18.2.0",
    "tsup": "^8.0.0",
    "typescript": "^5.5.0"
  }
}
```

```json
// apps/web/package.json
{
  "name": "@my/web",
  "dependencies": {
    "@my/ui": "workspace:*",
    "@my/utils": "workspace:*",
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@my/eslint-config": "workspace:*",
    "@my/tsconfig": "workspace:*",
    "typescript": "^5.5.0"
  }
}
```

**常用开发命令**：

```bash
# 启动所有应用的 dev server
pnpm dev

# 只启动 web 应用
pnpm dev --filter @my/web

# 构建（自动拓扑排序）
pnpm build

# 构建 web 及其依赖
pnpm build --filter @my/web...

# 只构建变化的包（增量构建）
pnpm build

# 清理所有构建产物
pnpm clean

# 添加依赖到指定包
pnpm add zod --filter @my/web

# 在根目录添加开发依赖（-w 标志）
pnpm add -Dw turbo

# 查看依赖图
pnpm ls -r --depth 0

# 更新交互式选择
pnpm update -i
```

---

## 常见问题

### 1. shamefully-hoist 的取舍

**什么时候需要开启**：

- 旧项目迁移到 pnpm 时，大量幽灵依赖导致报错
- 某些工具（如 `eslint`、`jest`）依赖提升才能找到插件
- 使用了不兼容 pnpm 严格模式的第三方库

**什么时候不应该开启**：

- 新项目——应该从一开始就养成声明依赖的习惯
- Monorepo——严格隔离是 pnpm 的核心优势
- 库开发——幽灵依赖会导致发布后用户安装失败

**推荐做法**：先用默认模式，遇到问题逐个用 `public-hoist-pattern` 精确提升，而不是一刀切 `shamefully-hoist=true`。

### 2. peer 依赖警告

```bash
# 常见警告
WARN  Issues with peer dependencies found
.
├─┬ @my/ui 0.0.0
│ └── ✕ unmet peer react@^18.0.0: found 17.0.2
```

**解决方案**：

```ini
# 方式一：严格模式（默认），peer 不满足会报错
strict-peer-dependencies=true

# 方式二：自动安装 peer 依赖
auto-install-peers=true

# 方式三：忽略 peer 依赖警告（不推荐）
# 不配置 strict-peer-dependencies 即可
```

```bash
# 临时忽略 peer 依赖检查
pnpm install --no-strict-peer-dependencies
```

### 3. 不兼容的包

某些包假设了扁平的 `node_modules` 结构，在 pnpm 下可能报错：

```bash
# 报错：Cannot find module 'some-dep'
# 原因：该包未声明 some-dep 为依赖，但 npm 下因为提升可以访问
```

**解决方案优先级**：

1. 向包作者提 Issue / PR，补全缺失的依赖声明
2. 使用 `pnpm patch` 临时修复
3. 使用 `public-hoist-pattern` 精确提升
4. 最后才考虑 `shamefully-hoist=true`

### 4. CI 环境缓存

```yaml
# GitHub Actions 缓存配置
- name: Checkout
  uses: actions/checkout@v4

- name: Install pnpm
  uses: pnpm/action-setup@v2
  with:
    version: 9

- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'pnpm'                  # 自动缓存 pnpm-store

- name: Install dependencies
  run: pnpm install --frozen-lockfile

- name: Build
  run: pnpm build
```

**Docker 中使用 pnpm**：

```dockerfile
FROM node:20-alpine AS builder

RUN corepack enable

WORKDIR /app

# 先复制依赖文件，利用 Docker 缓存层
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY apps/web/package.json ./apps/web/
COPY packages/ui/package.json ./packages/ui/

RUN pnpm install --frozen-lockfile

# 再复制源码
COPY . .

RUN pnpm build --filter @my/web

# 生产镜像
FROM node:20-alpine

RUN corepack enable

WORKDIR /app

COPY --from=builder /app/apps/web/dist ./dist
COPY --from=builder /app/apps/web/package.json ./package.json
COPY --from=builder /app/pnpm-lock.yaml ./pnpm-lock.yaml

RUN pnpm install --prod --frozen-lockfile

CMD ["node", "dist/server.js"]
```

---

## 面试题

### Q1: pnpm 为什么比 npm/yarn 快？

> pnpm 的速度优势来自三层机制：(1) **内容寻址存储**——全局存储中同一个包只存一份，新项目安装时通过硬链接指向已有文件，跳过下载和磁盘写入；(2) **硬链接替代复制**——创建硬链接是文件系统级别的操作（创建目录条目），比复制文件快一个数量级；(3) **并发下载**——pnpm 使用多连接并发下载包的 tarball，并利用智能网络策略优化下载顺序。实测中，在已有全局缓存的情况下，pnpm install 比 npm install 快 2-3 倍。

### Q2: 幽灵依赖是什么？有什么危害？

> 幽灵依赖（Phantom Dependencies）是指在 `package.json` 中未声明，但代码中可以直接 `require`/`import` 使用的依赖。产生原因是 npm/yarn 的扁平化安装将依赖提升到 `node_modules` 根目录，使得未声明的传递依赖也可被访问。危害包括：(1) **隐式耦合**——代码依赖了不确定的包版本，随时可能因上游升级而变化；(2) **运行时崩溃**——被依赖的包升级或移除后，项目代码直接报错 `Cannot find module`；(3) **发布失败**——库发布后用户安装时缺少幽灵依赖导致运行失败。pnpm 通过符号链接的严格隔离结构彻底解决了这个问题。

### Q3: pnpm 的 node_modules 结构是怎样的？为什么能防止幽灵依赖？

> pnpm 的 `node_modules` 采用三层结构：(1) **`.pnpm/` 目录**——按 `name@version` 组织，每个包有自己的 `node_modules` 子目录，其中真实文件通过硬链接指向全局存储，依赖通过符号链接指向 `.pnpm/` 中的对应位置；(2) **`.pnpm/pkg@ver/node_modules/` 内层**——包含该包的所有依赖（符号链接），确保包可以正确解析自己的依赖；(3) **`node_modules/` 根目录**——只包含直接依赖的符号链接（指向 `.pnpm/` 中的对应包）。防止幽灵依赖的原理：根目录只有 `package.json` 中声明的依赖，未声明的包不存在于根目录，`require('undeclared-pkg')` 会直接报错。

### Q4: shamefully-hoist 什么时候用？有什么代价？

> `shamefully-hoist` 将所有依赖提升到 `node_modules` 根目录，模拟 npm 的扁平结构。适用场景：(1) **旧项目迁移**——大量幽灵依赖存在，逐个修复成本过高；(2) **不兼容的第三方工具**——某些 ESLint 插件、Babel 插件依赖提升才能找到依赖；(3) **React Native 项目**——Metro bundler 与 pnpm 严格模式兼容性较差。代价：(1) **失去严格隔离**——重新引入幽灵依赖问题；(2) **违背 pnpm 设计初衷**——扁平化带来的确定性、安全性优势丧失。推荐做法：优先使用 `public-hoist-pattern` 精确提升特定包，而不是全局开启 `shamefully-hoist`。

### Q5: workspace 协议是什么？`workspace:*` 和 `workspace:^` 有什么区别？

> `workspace:` 协议是 pnpm 提供的 Monorepo 包间依赖机制，开发时将依赖链接到本地工作区包，发布时自动替换为真实版本号。区别：(1) `workspace:*`——发布时替换为精确版本号（如 `1.2.3`），适用于需要精确控制版本的场景；(2) `workspace:^`——发布时替换为兼容范围（如 `^1.2.3`），允许补丁和小版本更新；(3) `workspace:~`——发布时替换为补丁范围（如 `~1.2.3`）。开发时三者行为相同，都是链接到本地最新版本。选择建议：内部包之间用 `workspace:*` 确保版本一致；对外发布的库用 `workspace:^` 保持灵活性。

### Q6: pnpm patch 的原理是什么？和 patch-package 有什么区别？

> `pnpm patch` 的原理：(1) 执行 `pnpm patch pkg@ver` 时，pnpm 将该包从全局存储复制到临时目录；(2) 开发者在临时目录中修改文件；(3) 执行 `pnpm patch-commit` 时，pnpm 计算修改前后的 diff，生成 patch 文件保存到 `patches/` 目录；(4) 后续 `pnpm install` 时，pnpm 在硬链接完成后自动应用 patch。与 `patch-package` 的区别：(1) **内置 vs 外部**——pnpm patch 是内置功能，无需安装额外包；(2) **硬链接安全**——patch-package 直接修改 `node_modules` 中的文件，可能影响硬链接的其他项目；pnpm patch 先复制再打补丁，不影响全局存储；(3) **格式**——pnpm patch 生成标准 unified diff 格式，`patch-package` 使用自有格式。

### Q7: Corepack 是什么？解决了什么问题？

> Corepack 是 Node.js 16.9+ 自带的包管理器管理器（package manager manager），解决了团队中包管理器版本不一致的问题。它充当代理——当执行 `pnpm install` 时，Corepack 拦截命令，根据 `package.json` 中的 `packageManager` 字段自动下载和使用指定版本的 pnpm。解决的问题：(1) **版本一致性**——不同开发者本地的 pnpm 版本可能不同，导致 `pnpm-lock.yaml` 格式差异和行为不一致；(2) **无需全局安装**——团队成员无需手动 `npm install -g pnpm`，Corepack 自动管理；(3) **CI 一致性**——CI 环境中通过 `corepack enable` 即可使用正确版本。使用方式：在 `package.json` 中声明 `"packageManager": "pnpm@9.1.0"`，团队成员执行 `corepack enable` 后即可。

### Q8: pnpm vs yarn Berry（v2+）各有什么优劣？

> pnpm 和 yarn Berry 是第三代包管理器的两大方案，核心差异在于解决幽灵依赖的路径不同。(1) **pnpm**——保留 `node_modules` 结构，通过符号链接的严格隔离防止幽灵依赖。优势：兼容性好（Node 生态完全兼容）、迁移成本低、Monorepo 支持完善；劣势：仍占用 `node_modules` 空间（虽然文件通过硬链接共享）、`shamefully-hoist` 场景下会退化为扁平结构。(2) **yarn Berry**——使用 Plug'n'Play（PnP）彻底消除 `node_modules`，通过 `.pnp.cjs` 文件告诉 Node.js 如何解析依赖。优势：零 `node_modules`（安装极快、磁盘极省）、Zero-Install（可提交到 Git 实现离线安装）；劣势：兼容性差（很多工具假设 `node_modules` 存在，需要 `node-modules` linker 回退）、学习成本高（PnP 概念、.pnp.cjs 调试困难）。选择建议：新项目、团队协作、Monorepo 场景选 pnpm；追求极致安装速度、愿意处理兼容性问题的团队可尝试 yarn Berry。

---

**相关链接：** [[Webpack与Vite]] [[Monorepo管理]] [[npm与包管理]]

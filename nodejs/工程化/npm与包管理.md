---
tags:
  - Node.js
  - 工具
  - npm
  - 包管理
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# npm与包管理

## What — 是什么

> npm（Node Package Manager）是 Node.js 的默认包管理器，管理项目依赖、脚本和发布流程。与 yarn、pnpm 构成三大包管理工具。

**核心概念：**

- **package.json**：项目元数据，声明依赖、脚本和配置
- **package-lock.json**：锁定依赖树的确切版本，保证可复现构建
- **node_modules**：依赖安装目录（嵌套结构）
- **npx**：执行 npm 包中的命令，无需全局安装

**关键特性：**

- 语义化版本（SemVer）：`^1.2.3`（兼容次版本）、`~1.2.3`（兼容补丁）
- workspaces：monorepo 支持（npm 7+）
- 生命周期脚本：`preinstall`、`postbuild` 等

## Why — 为什么

**适用场景：**

- 项目依赖管理
- 开发脚本编排（build/test/lint）
- 包发布与分发

**对比替代工具：**

| 维度 | npm | yarn | pnpm |
|------|-----|------|------|
| 安装速度 | 中 | 快 | 极快 |
| 磁盘占用 | 高 | 高 | 低（硬链接共享） |
| 幽灵依赖 | 有 | 有 | 无（严格隔离） |
| monorepo | workspaces | workspaces | workspace（内置） |

**优缺点：**

- ✅ 优点：
  - Node.js 内置，零配置开箱即用
  - 生态最大，所有包都支持
  - lockfile 保证可复现性
- ❌ 缺点：
  - 安装速度较慢
  - 扁平化 node_modules 导致幽灵依赖
  - 磁盘占用大（每个项目独立安装）

## How — 怎么用

### 安装配置

```bash
# 初始化项目
npm init -y

# 安装依赖
npm install express          # 生产依赖
npm install -D jest          # 开发依赖
npm install -g typescript    # 全局安装

# 版本管理
npm outdated                 # 检查过时依赖
npm update                   # 更新依赖（遵循 SemVer 范围）
npm audit                    # 安全审计
```

### 快速上手

**package.json 脚本：**

```json
{
  "scripts": {
    "dev": "nodemon src/index.js",
    "build": "tsc && node dist/index.js",
    "test": "jest --coverage",
    "lint": "eslint src/"
  }
}
```

```bash
npm run dev    # 执行开发服务器
npm test       # test 可省略 run
```

### 代码示例

**npx 使用：**

```bash
# 一次性执行，无需全局安装
npx create-react-app my-app
npx ts-node script.ts

# 指定版本
npx -p typescript@4.9 tsc --init
```

**workspaces 配置：**

```json
{
  "workspaces": ["packages/*"]
}
```

```bash
npm install -w packages/utils   # 安装到指定 workspace
npm run build -w packages/api   # 在指定 workspace 执行脚本
```

### 性能调优

| 参数 | 默认值 | 调优建议 | 说明 |
|------|--------|----------|------|
| `--prefer-offline` | false | 启用 | 优先使用缓存 |
| `--legacy-peer-deps` | false | 按需 | 跳过 peer 依赖检查 |
| `registry` | npmjs.org | 使用镜像 | 国内用 `npmmirror` |
| `cache` | ~/.npm | SSD 分区 | 加速缓存读取 |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 幽灵依赖 | 扁平化安装，间接依赖被意外引用 | 使用 pnpm 或启用 `--install-strategy=nested` |
| peer 依赖冲突 | 多个包要求不同版本 | `--legacy-peer-deps` 或升级依赖 |
| lockfile 不一致 | 团队成员手动修改 package.json | 始终通过 `npm install/remove` 操作 |
| 安装慢 | 网络问题 | 使用国内镜像 `npm config set registry https://registry.npmmirror.com` |

## 面试题

**Q1: package-lock.json 的作用是什么？能否提交到 Git？**
> package-lock.json 锁定整个依赖树的确切版本号（包括间接依赖），确保团队成员和 CI 环境安装完全相同的依赖版本，避免"我本地能跑"的问题。应该提交到 Git，尤其在应用项目中。只有在库（library）项目中，某些团队选择不提交 lockfile，让使用者获取最新兼容版本。

**Q2: SemVer 中 ^ 和 ~ 的区别是什么？还有哪些版本范围写法？**
> ^1.2.3 表示兼容次版本更新（允许 1.x.x，不升到 2.0.0），即左边第一个非零数字不变。~1.2.3 表示兼容补丁更新（允许 1.2.x，不升到 1.3.0），只改最右边的版本号。其他写法：1.2.3（精确版本）、* 或空（任意版本）、>=1.2.3（大于等于）、1.2.3 - 2.0.0（范围）、||（或关系）。

**Q3: npm 和 pnpm 的核心区别是什么？pnpm 如何解决幽灵依赖问题？**
> pnpm 使用硬链接 + 符号链接策略：所有包存储在全局 store 中，项目 node_modules/.pnpm 下通过硬链接指向 store，再通过符号链接建立依赖拓扑。这带来两个优势：(1) 磁盘节省，多项目共享同一份包文件；(2) 严格的依赖隔离，只能访问 package.json 中声明的依赖，杜绝幽灵依赖。npm/yarn 的扁平化安装会将间接依赖提升到顶层，导致引用未声明的包成为可能。

**Q4: 什么是幽灵依赖？如何产生的？怎么解决？**
> 幽灵依赖是指代码中引用了 package.json 未声明的包。原因是 npm/yarn 的扁平化（hoisting）机制会将间接依赖提升到 node_modules 顶层，使其可被意外引用。当该间接依赖被上游包升级或移除时，项目会突然崩溃。解决方案：使用 pnpm 的严格隔离模式；或在 npm 中启用 --install-strategy=nested 禁用扁平化；也可配合 eslint-plugin-import 检测未声明的依赖引用。

**Q5: 依赖版本冲突（peer dependency conflict）如何解决？**
> 当多个包要求同一 peer dependency 的不同版本时会产生冲突。解决方法：(1) 升级依赖使版本一致；(2) npm 7+ 中使用 --legacy-peer-deps 忽略 peer 依赖检查（回退到 npm 6 行为）；(3) 使用 pnpm 的 overrides 或 npm 的 overrides 字段强制指定版本；(4) 使用 resolutions（yarn）覆盖特定依赖版本。优先推荐升级依赖统一版本。

---

**相关链接：**
- npm 官方文档：https://docs.npmjs.com/

---
tags:
  - Web前端
  - 工程化
  - Webpack
  - Vite
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Webpack与Vite

## What — 是什么

> Webpack 是老牌模块打包器，基于配置和 loader/plugin 体系处理各种资源；Vite 是新一代构建工具，开发时利用浏览器原生 ESM 实现极速冷启动，生产构建基于 Rollup。

**核心概念：**

- **Webpack**：Entry → Loader（转换） → Plugin（扩展） → Output
- **Vite**：Dev Server（原生 ESM 按需编译） + Build（Rollup 打包）
- **HMR**：热模块替换，修改代码后局部更新，不刷新页面
- **Tree Shaking**：移除未使用的导出代码

**关键特性：**

- Webpack：成熟的代码分割、懒加载、loader 生态
- Vite：毫秒级冷启动、按需编译、原生 ESM
- 两者都支持 HMR 和 Tree Shaking

## Why — 为什么

**适用场景：**

- Webpack：复杂企业级项目、需要丰富 loader/plugin 生态
- Vite：新项目、追求开发体验、SPA/SSR

**对比替代工具：**

| 维度 | Webpack | Vite | Rollup |
|------|---------|------|--------|
| 开发体验 | 慢冷启动 | 极快冷启动 | 无 dev server |
| 生态 | 最丰富 | 快速增长 | 丰富（库开发） |
| 学习曲线 | 高（配置复杂） | 低（开箱即用） | 中 |
| 生产构建 | 自有打包器 | Rollup | 自有 |

**优缺点：**

- ✅ Webpack 优点：
  - 生态最成熟，几乎所有场景有现成方案
  - 强大的代码分割和懒加载
- ❌ Webpack 缺点：
  - 冷启动慢（全量打包）
  - 配置复杂，学习成本高
- ✅ Vite 优点：
  - 冷启动极快（按需编译）
  - 配置极简，零配置可用
- ❌ Vite 缺点：
  - 生态不如 Webpack 完善
  - CommonJS 依赖兼容偶尔有问题

## How — 怎么用

### 安装配置

**Webpack 最小配置：**

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
    },
    module: {
        rules: [
            { test: /\.css$/, use: ['style-loader', 'css-loader'] },
            { test: /\.(js|jsx)$/, exclude: /node_modules/, use: 'babel-loader' },
        ],
    },
    devServer: {
        hot: true,
        port: 3000,
    },
};
```

**Vite 最小配置：**

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    server: {
        port: 3000,
    },
});
```

### 快速上手

```bash
# Webpack 项目
npm install webpack webpack-cli webpack-dev-server -D

# Vite 项目（脚手架）
npm create vite@latest my-app -- --template react
```

### 代码示例

**Webpack 代码分割：**

```javascript
// 动态导入
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));

// webpack.config.js
optimization: {
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
            vendor: {
                test: /node_modules/,
                name: 'vendors',
                chunks: 'all',
            },
        },
    },
}
```

**Vite 代理配置：**

```javascript
// vite.config.js
export default defineConfig({
    server: {
        proxy: {
            '/api': {
                target: 'http://localhost:8080',
                changeOrigin: true,
                rewrite: (path) => path.replace(/^\/api/, ''),
            },
        },
    },
});
```

### 性能调优

| 参数 | Webpack | Vite | 说明 |
|------|---------|------|------|
| 缓存 | `cache: { type: 'filesystem' }` | 默认启用 | 加速二次构建 |
| Source Map | `devtool: 'eval-cheap-module-source-map'` | 默认启用 | 开发调试 |
| 构建分析 | `webpack-bundle-analyzer` | `rollup-plugin-visualizer` | 查看包大小 |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Webpack 冷启动慢 | 全量打包所有模块 | 开启 filesystem cache 或迁移到 Vite |
| Vite CJS 兼容 | 部分包只提供 CommonJS 格式 | 用 `vite-plugin-commonjs` 或 `optimizeDeps.include` 预构建 |
| HMR 失效 | 组件缺少 HMR 边界 | 确保组件有默认导出 |
| 生产包过大 | 未做 Tree Shaking | 确保 ESM 导入、配置 `sideEffects: false` |

### 最佳实践

- 新项目优先用 Vite
- 老项目可先用 `@vitejs/plugin-legacy` 渐进迁移
- 生产构建前用 bundle analyzer 检查包大小
- 开启持久化缓存加速开发体验

## 面试题

**Q1: Vite 为什么比 Webpack 冷启动快？**
> Vite 开发时利用浏览器原生 ESM，按需编译当前请求的模块，无需全量打包；而 Webpack 冷启动时需要从 Entry 出发递归解析所有依赖并全量打包，项目越大差距越明显。

**Q2: HMR（热模块替换）的原理是什么？**
> 开发服务器监听文件变化后，只重新编译变更模块，通过 WebSocket 通知浏览器替换对应模块，保留应用状态不刷新页面。Webpack 通过 `module.hot.accept` 圈定 HMR 边界，Vite 基于原生 ESM 的 `import` 动态重载实现。

**Q3: Tree Shaking 生效的条件是什么？**
> 必须使用 ESM 的静态导入（`import/export`），因为 Tree Shaking 依赖静态分析识别未使用的导出；同时需要在 `package.json` 中配置 `sideEffects: false` 告知打包器哪些文件无副作用可安全移除。

**Q4: Loader 和 Plugin 的区别是什么？**
> Loader 是文件转换器，在 `module.rules` 中配置，负责将非 JS 资源（CSS/图片/TS）转换为 Webpack 能处理的模块；Plugin 是功能扩展，基于事件钩子机制工作，贯穿整个构建生命周期，可执行打包优化、资源注入等更广泛的任务。

---

**相关链接：**
- [[React核心]]
- [[Vue核心]]

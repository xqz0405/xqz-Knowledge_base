---
tags:
  - Web前端
  - 工程化
  - 代码分割
  - Tree Shaking
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# 代码分割与Tree Shaking

## What — 是什么

> 代码分割（Code Splitting）将应用代码按需拆分为多个小包，实现按需加载；Tree Shaking 通过静态分析消除未使用的导出代码。两者都是减少最终交付包体积的核心手段。

**核心概念：**

- **代码分割**：将单个大包拆分为多个小包，按路由/组件/功能按需加载
- **Tree Shaking**：基于 ES Module 静态结构，标记并移除未使用的导出（dead code elimination）
- **动态导入**：`import()` 语法实现运行时按需加载模块
- **副作用（Side Effects）**：模块执行时会对全局产生影响的代码，影响 Tree Shaking 安全性

**关键特性：**

- 代码分割关注"何时加载"——首屏只加载必要代码，其余按需拉取
- Tree Shaking 关注"加载什么"——确保打包产物不含未使用代码
- 两者配合：先分割减小单次加载量，再摇树移除冗余代码

## Why — 为什么

**核心动机：**

- **首屏加载性能**：单页应用全量打包可达数 MB，首屏只需其中一小部分
- **减少无效代码**：第三方库通常只用到部分功能，未用代码不应进入产物
- **按需加载**：用户访问特定路由/功能时才加载对应代码，节省带宽
- **HTTP/2 多路复用**：小包并行加载比单个大包更快，代码分割在 HTTP/2 下优势更明显

**数据支撑：**

- 首屏 JS 体积每减少 100KB，LCP 平均下降 200-400ms
- lodash 全量 72KB gzip，按需引入仅 4-8KB
- moment.js 含全部 locale 约 67KB gzip，只保留中文约 3KB

**对比其他优化手段：**

| 手段 | 减少体积 | 减少请求数 | 加速首屏 | 实现成本 |
|------|---------|-----------|---------|---------|
| 代码分割 | 间接（按需加载） | 可能增加 | 显著 | 低 |
| Tree Shaking | 直接 | 不变 | 间接 | 低 |
| 压缩（gzip/brotli） | 直接 | 不变 | 间接 | 极低 |
| CDN | 不变 | 不变 | 显著 | 低 |

## 对比：代码分割 vs Tree Shaking

| 维度 | 代码分割 | Tree Shaking |
|------|---------|-------------|
| **目标** | 按需加载，减少首屏体积 | 移除未使用代码，减少总体积 |
| **原理** | 将代码拆分为多个 chunk，运行时动态加载 | 静态分析 ES Module 依赖图，标记未使用 export |
| **时机** | 运行时（动态 import 触发加载） | 构建时（打包阶段分析 + 压缩阶段消除） |
| **效果** | 减少首屏加载体积，总体积不变 | 减少总产物体积 |
| **前提** | 浏览器支持动态导入 / HTTP/2 | 必须使用 ES Module 静态导入导出 |
| **工具支持** | Webpack SplitChunks / Vite 自动 | Webpack usedExports / Rollup / esbuild |

## How — 怎么用

### 一、Webpack 代码分割

#### 1. Entry 分割

```javascript
// webpack.config.js — 多入口分割
module.exports = {
    entry: {
        app: './src/app.js',
        admin: './src/admin.js',
        vendor: './src/vendor.js',
    },
    output: {
        filename: '[name].[contenthash:8].js',
        path: path.resolve(__dirname, 'dist'),
    },
};
```

**局限**：如果多个 entry 引入相同模块，会重复打包。需要配合 SplitChunksPlugin 去重。

#### 2. SplitChunksPlugin

```javascript
// webpack.config.js
module.exports = {
    optimization: {
        splitChunks: {
            // chunks: 'async'   — 只分割异步导入（默认）
            // chunks: 'initial' — 只分割同步导入
            // chunks: 'all'     — 同步+异步都分割（推荐）
            chunks: 'all',
            minSize: 20000,          // 最小分割体积（20KB），小于此值不分割
            maxSize: 244000,         // 超过此值尝试进一步分割
            minChunks: 1,            // 被引用次数 >= 1 才分割
            maxAsyncRequests: 30,    // 异步加载最大并行请求数
            maxInitialRequests: 30,  // 入口点最大并行请求数
            automaticNameDelimiter: '~',
            cacheGroups: {
                // 第三方库单独打包
                vendors: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    priority: -10,
                    reuseExistingChunk: true,
                },
                // 公共模块提取
                common: {
                    minChunks: 2,
                    name: 'common',
                    priority: -20,
                    reuseExistingChunk: true,
                },
                // 将 React 全家桶单独拆包
                react: {
                    test: /[\\/]node_modules[\\/](react|react-dom|react-router)[\\/]/,
                    name: 'react-vendor',
                    priority: 0,
                    chunks: 'all',
                },
                // UI 库单独拆包
                ui: {
                    test: /[\\/]node_modules[\\/](antd|@ant-design)[\\/]/,
                    name: 'ui-vendor',
                    priority: 0,
                    chunks: 'all',
                },
            },
        },
    },
};
```

**cacheGroups 核心参数说明：**

| 参数 | 说明 |
|------|------|
| `test` | 匹配模块路径的正则或函数 |
| `name` | 输出 chunk 名称 |
| `priority` | 优先级，数值越大越优先匹配 |
| `minSize` | 覆盖外层 minSize |
| `reuseExistingChunk` | 如果模块已被提取到已有 chunk，复用而非重新创建 |
| `enforce` | 忽略 minSize/minChunks 等限制强制分割 |

#### 3. 动态 import 与魔法注释

```javascript
// 基础动态导入
const module = await import('./utils/heavy');

// 魔法注释：命名 chunk
const Dashboard = await import(
    /* webpackChunkName: "dashboard" */
    './pages/Dashboard'
);

// 魔法注释：预获取（浏览器空闲时下载）
const Settings = await import(
    /* webpackChunkName: "settings" */
    /* webpackPrefetch: true */
    './pages/Settings'
);

// 魔法注释：预加载（与父 chunk 并行下载）
const Modal = await import(
    /* webpackChunkName: "modal" */
    /* webpackPreload: true */
    './components/Modal'
);
```

**魔法注释对比：**

| 注释 | 作用 | 加载时机 | 适用场景 |
|------|------|---------|---------|
| `webpackChunkName` | 指定 chunk 名称 | 同动态 import | 调试 & 控制 chunk 分组 |
| `webpackPrefetch` | 空闲时预获取 | 父 chunk 加载完成后，浏览器空闲时 | 未来可能访问的页面 |
| `webpackPreload` | 并行预加载 | 与父 chunk 同时请求 | 当前页面一定会用的模块 |

### 二、Vite 代码分割

Vite 生产构建基于 Rollup，自动处理代码分割：

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
    build: {
        rollupOptions: {
            output: {
                // 手动配置 chunk 分割
                manualChunks(id) {
                    // node_modules 全部打入 vendors
                    if (id.includes('node_modules')) {
                        // 按包名进一步拆分
                        if (id.includes('react') || id.includes('react-dom')) {
                            return 'react-vendor';
                        }
                        if (id.includes('antd') || id.includes('@ant-design')) {
                            return 'ui-vendor';
                        }
                        return 'vendor';
                    }
                },
                // 或使用对象形式
                // manualChunks: {
                //     'react-vendor': ['react', 'react-dom'],
                //     'ui-vendor': ['antd'],
                //     'utils': ['lodash-es', 'dayjs'],
                // },
            },
        },
        // 小于此阈值的资源内联为 base64
        assetsInlineLimit: 4096,
        // chunk 大小警告阈值
        chunkSizeWarningLimit: 1000,
    },
});
```

**Vite 动态 import 自动分割：**

```javascript
// Vite 自动将动态 import 拆分为独立 chunk
const AdminPanel = lazy(() => import('./pages/AdminPanel'));
// 产物：AdminPanel.[hash].js（自动分割，无需额外配置）
```

**Vite vs Webpack 代码分割对比：**

| 维度 | Webpack | Vite |
|------|---------|------|
| 分割方式 | SplitChunksPlugin + cacheGroups | Rollup manualChunks |
| 动态 import | 自动分割 + 魔法注释 | 自动分割 |
| 配置复杂度 | 高（大量参数） | 低（开箱即用） |
| 预加载 | webpackPrefetch / webpackPreload | 原生 `<link>` 标签 |

### 三、路由级别代码分割

#### React：lazy + Suspense

```javascript
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import Loading from './components/Loading';

// 路由懒加载
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
    return (
        <Suspense fallback={<Loading />}>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/dashboard" element={<Dashboard />} />
                <Route path="/settings" element={<Settings />} />
                <Route path="/profile" element={<Profile />} />
            </Routes>
        </Suspense>
    );
}

// 自定义 Suspense fallback（骨架屏）
function AppWithSkeleton() {
    return (
        <Suspense fallback={<DashboardSkeleton />}>
            <Dashboard />
        </Suspense>
    );
}
```

#### Vue：defineAsyncComponent

```javascript
// Vue 3 路由懒加载
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    {
        path: '/',
        component: () => import('./pages/Home.vue'),
    },
    {
        path: '/dashboard',
        component: () => import('./pages/Dashboard.vue'),
    },
    {
        path: '/settings',
        component: () => import('./pages/Settings.vue'),
    },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

// defineAsyncComponent — 更细粒度的控制
import { defineAsyncComponent } from 'vue';

const AsyncModal = defineAsyncComponent({
    loader: () => import('./components/HeavyModal.vue'),
    loadingComponent: LoadingSpinner,
    errorComponent: ErrorDisplay,
    delay: 200,       // 延迟显示 loading，避免闪烁
    timeout: 10000,   // 超时显示 error 组件
});
```

### 四、组件级别懒加载

```javascript
// React — 条件渲染时懒加载
function App() {
    const [showChart, setShowChart] = useState(false);

    // 只有点击后才加载图表组件
    const ChartModal = showChart
        ? lazy(() => import('./components/ChartModal'))
        : null;

    return (
        <div>
            <button onClick={() => setShowChart(true)}>打开图表</button>
            {showChart && (
                <Suspense fallback={<Spinner />}>
                    <ChartModal />
                </Suspense>
            )}
        </div>
    );
}

// Vue — defineAsyncComponent 懒加载重型组件
const RichTextEditor = defineAsyncComponent(() =>
    import('./components/RichTextEditor.vue')
);

// 适合懒加载的组件类型：
// - 图表库（ECharts / D3）
// - 富文本编辑器
// - 大型弹窗/抽屉
// - 代码高亮组件
// - 地图组件
```

### 五、Tree Shaking 原理

#### 核心原理：ES Module 静态分析

```javascript
// ES Module — 静态导入，可分析
import { used, unused } from './utils';  // 编译时确定依赖

// CommonJS — 动态导入，不可分析
const utils = require('./utils');  // 运行时才能确定
const fn = require(condition ? './a' : './b');  // 动态路径
```

**Tree Shaking 工作流程：**

1. **静态分析阶段**：构建工具遍历 ES Module 依赖图，标记每个 export 是否被 import
2. **标记阶段**：未使用的 export 被标记为 `unused harmony export`
3. **压缩阶段**：Terser/esbuild 在压缩时移除标记为 unused 的代码

```javascript
// utils.js
export function used() { return 'used'; }
export function unused() { return 'unused'; }  // 将被标记为 unused

// app.js
import { used } from './utils';  // 只导入 used
used();  // unused() 从未引用，产物中被移除

// 产物（压缩后）
function used(){return"used"}used();
// unused 完全消失
```

#### sideEffects 字段

```json
// package.json — 声明无副作用，允许安全 Tree Shaking
{
    "sideEffects": false
}

// 有副作用的文件需要排除
{
    "sideEffects": [
        "*.css",
        "*.scss",
        "*.less",
        "./src/polyfills.js",
        "./src/global-setup.js"
    ]
}
```

**副作用场景举例：**

```javascript
// 副作用代码：修改全局变量、注册事件、修改原型
import './polyfills';           // 修改了 Array.prototype
import './global.css';          // 注入了全局样式
import './analytics';           // 注册了全局埋点

// 即使没有显式 import 任何导出，这些文件的副作用必须保留
// 如果 sideEffects: false，这些文件会被错误移除
```

### 六、Webpack Tree Shaking 配置

```javascript
// webpack.config.js
module.exports = {
    mode: 'production',  // 自动启用 Tree Shaking

    optimization: {
        usedExports: true,    // 标记未使用的 export（production 默认 true）
        minimize: true,       // 启用压缩（production 默认 true）
        sideEffects: true,    // 读取 package.json 的 sideEffects 字段

        // 更精细的 Tree Shaking 配置
        // providedExports: true,   // 收集模块提供的 export
        // usedExports: 'global',   // 全局标记使用情况
        // innerGraph: true,        // 内部图分析（Webpack 5）
    },

    module: {
        rules: [
            {
                test: /\.jsx?$/,
                sideEffects: false,  // 覆盖该类文件的副作用标记
            },
        ],
    },
};
```

**Webpack 5 内部图优化（innerGraph）：**

```javascript
// 开启 innerGraph 后，Webpack 能分析更细粒度的未使用代码
const a = 1;
const b = 2;  // 如果 b 未被使用，整个变量声明被移除
export { a, b };
```

### 七、Vite Tree Shaking

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
    build: {
        // Vite 使用 esbuild 做开发时预构建
        // 生产构建使用 Rollup 做 Tree Shaking（自动启用）
        minify: 'esbuild',  // 或 'terser'
    },
    optimizeDeps: {
        // 预构建时排除有问题的包
        exclude: ['some-cjs-only-package'],
    },
});

// Vite Tree Shaking 特点：
// 1. 开发时 esbuild 快速预构建
// 2. 生产时 Rollup 做完整 Tree Shaking
// 3. 零配置自动启用
// 4. 兼容 package.json 的 sideEffects 字段
```

### 八、哪些代码无法 Tree Shake

#### 1. 副作用代码

```javascript
// 模块顶层有副作用执行
window.myGlobal = 'value';           // 修改全局对象
document.addEventListener(...);       // 注册事件
Array.prototype.myMethod = () => {};  // 修改原型

// 即使只 import 了某个函数，副作用代码也必须保留
// 打包器保守策略：保留整个模块
```

#### 2. CommonJS 模块

```javascript
// require 是动态的，无法静态分析
const _ = require('lodash');
module.exports = { ... };

// 即使是 lodash-es（ESM 版）可以 Tree Shake
// 但 lodash（CJS 版）无法 Tree Shake
import _ from 'lodash';           // 全量引入，无法摇树
import { debounce } from 'lodash-es';  // ESM，可以摇树
```

#### 3. 动态属性访问

```javascript
// 静态访问 — 可以 Tree Shake
import { debounce } from 'lodash-es';
debounce(fn, 300);

// 动态访问 — 无法静态确定使用哪个导出
const methodName = 'debounce';
lodash[methodName](fn, 300);

// 解构后批量导出 — 也可能影响
export * from './utils';  // 重新导出所有，无法确定哪些被使用
```

#### 4. prototype 赋值

```javascript
// 这种写法无法被静态分析
function MyClass() {}
MyClass.prototype.method = function () {};

// 改用 class 语法则可以
class MyClass {
    method() {}  // 静态可分析
}
```

#### 5. IIFE（立即执行函数）

```javascript
// IIFE 有副作用，打包器无法安全移除
const result = (function () {
    // 复杂逻辑...
    return something;
})();

// 全局副作用
(function (global) {
    global.polyfill = ...;
})(window);
```

**不可 Tree Shake 汇总：**

| 代码模式 | 原因 | 替代方案 |
|---------|------|---------|
| CommonJS | 动态 require，无法静态分析 | 使用 ESM 版本（`lodash-es`） |
| 副作用代码 | 必须保留执行效果 | 隔离副作用到独立文件，配置 sideEffects |
| 动态属性访问 | 运行时才能确定引用 | 使用具名静态导入 |
| prototype 赋值 | 动态修改原型链 | 使用 class 语法 |
| IIFE | 可能包含副作用 | 避免，或用 `/*@__PURE__*/` 标记 |
| 全局变量修改 | 影响外部作用域 | 隔离到副作用文件 |

### 九、人工标记纯函数（/@__PURE__/）

```javascript
// @__PURE__ 告诉打包器：此调用无副作用，如果返回值未使用可安全移除
const result = /* @__PURE__ */ createHeavyUtility();

// 如果 result 未被使用，整行代码被移除
// 不加 @__PURE__，打包器保守保留 createHeavyUtility() 调用

// 常见使用场景
const store = /* @__PURE__ */ createStore(reducer);
const i18n = /* @__PURE__ */ setupI18n(config);
const styles = /* @__PURE__ */ css({ color: 'red' });

// React 中常见
export default /* @__PURE__ */ React.memo(function MyComponent(props) {
    return <div>{props.name}</div>;
});

// 以下场景必须加 @__PURE__
const computed = /* @__PURE__ */ expensiveCalculation(data);
if (computed > threshold) { ... }
// 如果 computed 被使用则保留，否则移除整个调用
```

**@__PURE__ 注意事项：**

```javascript
// 只标记调用表达式，不标记声明
const x = /* @__PURE__ */ factory();  // 正确

// 不会传播到后续使用
const a = /* @__PURE__ */ create();
const b = a.transform();  // a.transform() 没有被标记，不会被移除

// 只在赋值处标记
const c = /* @__PURE__ */ create().transform();  // 整条链路标记
```

### 十、包体积分析

#### webpack-bundle-analyzer

```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
    plugins: [
        new BundleAnalyzerPlugin({
            analyzerMode: 'server',       // 'server' | 'static' | 'json'
            analyzerHost: '127.0.0.1',
            analyzerPort: 8888,
            reportFilename: 'report.html',
            openAnalyzer: true,
            generateStatsFile: false,
            statsOptions: null,
        }),
    ],
};

// 也可通过 CLI 临时分析
// npx webpack --analyze
```

#### rollup-plugin-visualizer（Vite）

```javascript
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
    plugins: [
        visualizer({
            filename: './dist/stats.html',  // 输出文件
            open: true,                      // 自动打开
            gzipSize: true,                  // 显示 gzip 大小
            brotliSize: true,                // 显示 brotli 大小
            template: 'treemap',             // 'treemap' | 'sunburst' | 'network'
        }),
    ],
});
```

#### vite-plugin-visualizer

```javascript
// vite.config.js
import { visualizer } from 'vite-plugin-visualizer';

export default defineConfig({
    plugins: [
        visualizer({
            emitFile: true,
            filename: 'stats.html',
        }),
    ],
});
```

**分析流程：**

1. 构建项目，生成可视化报告
2. 查看哪些包占用体积最大
3. 识别可 Tree Shake 的第三方库（检查是否使用 ESM 版本）
4. 检查是否有重复打包的模块
5. 优化后重新构建对比

### 十一、预加载策略

#### Prefetch vs Preload

```html
<!-- Prefetch：预获取，低优先级，浏览器空闲时下载 -->
<link rel="prefetch" href="/settings.js" as="script">
<!-- 适合：下一页可能访问的资源 -->

<!-- Preload：预加载，高优先级，当前页面必须用的资源 -->
<link rel="preload" href="/critical-font.woff2" as="font" crossorigin>
<!-- 适合：当前页面渲染必需但不在关键路径的资源 -->
```

| 维度 | Prefetch | Preload |
|------|---------|---------|
| **优先级** | 低（空闲时） | 高（立即） |
| **用途** | 未来可能需要的资源 | 当前页面必需的资源 |
| **时机** | 页面加载完成后 | 与页面资源并行 |
| **缓存** | 缓存到磁盘 | 缓存到内存 |
| **滥用后果** | 浪费带宽 | 阻塞其他资源 |

#### 在 Webpack 中使用

```javascript
// Prefetch — 浏览器空闲时下载
const Settings = import(
    /* webpackPrefetch: true */
    './pages/Settings'
);
// 产物：<link rel="prefetch" href="settings.[hash].js">

// Preload — 与父 chunk 并行下载
const Modal = import(
    /* webpackPreload: true */
    './components/Modal'
);
// 产物：<link rel="preload" href="modal.[hash].js">
```

#### 在 Vite 中使用

```javascript
// Vite 支持通过原生 link 标签或插件实现
// 推荐使用 vite-plugin-preload 手动控制

// 或在 HTML 中直接声明
// <link rel="prefetch" href="/assets/settings-[hash].js">
// <link rel="preload" href="/assets/critical-[hash].js" as="script">
```

#### 预加载最佳实践

```javascript
// 好的预加载：首屏关键字体
<link rel="preload" href="/fonts/main.woff2" as="font" crossorigin>

// 好的预获取：用户大概率访问的下一页
<link rel="prefetch" href="/dashboard.[hash].js">

// 不好的预加载：所有路由都 preload
// 会导致首屏竞争带宽，适得其反

// 不好的预获取：低概率页面也 prefetch
// 浪费用户带宽
```

## 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Tree Shaking 不生效 | 使用了 CJS 导入或动态属性访问 | 改用 ESM 版本（如 `lodash-es`），使用具名静态导入 |
| 动态 import 在 SSR 报错 | 服务端不支持动态导入 | 使用 `@loadable/component`（React）或 `vue3-lazy`（Vue）做 SSR 兼容 |
| SplitChunks 配置过于复杂 | cacheGroups 规则冲突 | 先用默认配置分析产物，逐步调优；用 bundle analyzer 可视化验证 |
| 第三方库不可摇树 | 库只提供 CJS 格式 | 寻找 ESM 替代品；或用 `babel-plugin-lodash` 做按需转换 |
| CSS 被错误 Tree Shake | `sideEffects: false` 移除了 CSS | 在 sideEffects 中排除 `*.css` 文件 |
| 代码分割后 chunk 过多 | cacheGroups 规则过细 | 合并小 chunk，设置 `minSize` 阈值 |
| Prefetch 浪费带宽 | 低概率页面也预获取 | 仅对高转化率页面做 prefetch |
| 懒加载组件闪烁 | Suspense fallback 延迟不够 | 设置 `delay`（Vue）或骨架屏（React） |

**Tree Shaking 不生效排查清单：**

```javascript
// 1. 检查是否使用 ESM 导入
import { debounce } from 'lodash';      // CJS — 无法摇树
import { debounce } from 'lodash-es';   // ESM — 可以摇树

// 2. 检查 mode 是否为 production
// webpack --mode=production  或  mode: 'production'

// 3. 检查 sideEffects 配置
// package.json 中设置 "sideEffects": false
// 或排除有副作用的文件

// 4. 检查是否使用了具名导入
import _ from 'lodash-es';              // 默认导入 — 无法摇树
import { debounce } from 'lodash-es';   // 具名导入 — 可以摇树

// 5. 用 Bundle Analyzer 检查产物
// npx webpack --analyze
```

## 面试题

**Q1: 代码分割有哪些方式？分别适用什么场景？**
> 三种方式：① Entry 分割 — 多入口应用（如前台/后台），不同入口独立打包；② SplitChunksPlugin — 提取公共模块和第三方库，避免重复打包，适合所有项目；③ 动态 import — 路由和组件级别按需加载，首屏只加载必要代码。实践中三者配合使用：Entry 拆大模块，SplitChunks 提公共依赖，动态 import 做懒加载。

**Q2: Tree Shaking 的原理是什么？为什么依赖 ES Module？**
> Tree Shaking 分三步：① 静态分析阶段 — 构建工具从入口递归遍历 ES Module 的 `import/export`，构建完整的模块依赖图；② 标记阶段 — 对每个 export 检查是否有对应的 import 引用，未引用的标记为 `unused`；③ 消除阶段 — 压缩器（Terser/esbuild）移除标记为 unused 的代码。依赖 ES Module 是因为 `import/export` 是静态声明，编译时就能确定模块依赖关系，而 CommonJS 的 `require` 是动态调用，只有运行时才能确定加载什么，无法做静态分析。

**Q3: 为什么 Tree Shaking 需要 ES Module？CommonJS 为什么不行？**
> ES Module 的 `import/export` 是编译时静态声明的，模块依赖关系在代码执行前就确定了，打包器可以精确构建依赖图并识别哪些导出从未被引用。CommonJS 的 `require` 是运行时动态调用，可以写在条件语句中（`if (x) require('./a')`），甚至可以用变量拼接路径（`require('./' + name)`），这些场景只有运行时才能确定实际依赖，打包器无法在构建时安全地移除任何代码。

**Q4: sideEffects 字段的作用是什么？怎么配置？**
> `sideEffects` 告诉打包器哪些文件没有副作用、可以安全 Tree Shake。设为 `false` 表示整个包无副作用，打包器可移除未被使用的模块；设为数组则排除有副作用的文件（如 CSS、polyfill、全局初始化代码）。配置方式：在 `package.json` 中 `"sideEffects": false` 或 `"sideEffects": ["*.css", "./src/polyfills.js"]`；Webpack 还可在 `module.rules` 中对特定文件类型设置 `sideEffects: false`。错误配置会导致 CSS 丢失或 polyfill 不生效，需要仔细排除副作用文件。

**Q5: prefetch 和 preload 有什么区别？各自的使用场景？**
> Prefetch 是低优先级预获取，浏览器空闲时下载资源并缓存到磁盘，适合未来可能访问的页面（如用户大概率会点击的路由）；Preload 是高优先级预加载，与当前页面资源并行请求并缓存到内存，适合当前页面必需但不在关键解析路径上的资源（如关键字体、首屏大图）。核心区别：Prefetch 不阻塞当前页面加载，Preload 会竞争当前页面带宽。Preload 必须谨慎使用，滥用会导致首屏变慢。

**Q6: 动态 import() 的原理是什么？和静态 import 有什么区别？**
> 动态 `import()` 返回一个 Promise，运行时才发起模块请求，打包器将其拆分为独立 chunk。静态 `import` 是编译时声明，模块在代码执行前就已加载并绑定。区别：① 时机 — 静态编译时，动态运行时；② 返回值 — 静态直接绑定到导出，动态返回 Promise；③ 代码分割 — 静态导入的模块打入同一 chunk，动态导入自动拆分为独立 chunk；④ 条件加载 — 静态不能写在条件语句中，动态可以。

**Q7: 如何分析和优化包体积？**
> 使用打包分析工具可视化产物组成：Webpack 用 `webpack-bundle-analyzer`（`npx webpack --analyze`），Vite 用 `rollup-plugin-visualizer`。分析步骤：① 识别占用体积最大的模块；② 检查第三方库是否使用了 ESM 版本（如 `lodash` → `lodash-es`）；③ 检查是否有重复打包的模块（多版本共存）；④ 验证 Tree Shaking 是否生效（对比源码与产物）；⑤ 优化后重新构建对比体积变化。还可以用 `source-map-explorer` 基于 Source Map 分析。

**Q8: 哪些代码无法被 Tree Shake？如何解决？**
> 五种情况：① CommonJS 模块 — `require` 动态调用无法静态分析，解决方案是使用 ESM 版本（`lodash-es`、`@mui/material`）；② 副作用代码 — 全局变量修改、事件注册、原型修改必须保留，解决方案是隔离到独立文件并在 `sideEffects` 中声明；③ 动态属性访问 — `obj[dynamicKey]` 运行时才能确定，解决方案是使用具名静态导入；④ IIFE — 打包器无法确定是否有副作用，解决方案是避免 IIFE 或用 `/*@__PURE__*/` 标记；⑤ 类的 prototype 赋值 — 动态修改无法分析，解决方案是使用 class 语法。核心原则：代码越"静态"，Tree Shaking 效果越好。

---

**相关链接：**
- [[Webpack与Vite]]
- [[前端性能优化]]
- [[Babel与AST]]

---
tags:
  - Web前端
  - Electron
  - 桌面开发
  - Node.js
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Electron桌面开发

## What — 是什么

> Electron 是 GitHub 推出的桌面应用开发框架，将 Chromium 和 Node.js 合并到同一个运行时中，让开发者用 Web 技术（HTML/CSS/JS）构建跨平台桌面应用。

**核心概念：**

- **主进程（Main Process）**：运行 Node.js，管理窗口和系统交互，每个应用只有一个
- **渲染进程（Renderer Process）**：运行 Chromium，负责页面渲染，可有多个
- **IPC 通信**：主进程与渲染进程通过 `ipcMain` / `ipcRenderer` 通信
- **Preload 脚本**：在渲染进程加载前运行，安全地桥接主进程和渲染进程

**核心架构：**

- 设计理念：Web 技术 + Node.js 能力 = 桌面应用
- 核心模块：BrowserWindow、ipcMain/ipcRenderer、BrowserView、Tray、net
- 数据流：用户操作 → 渲染进程 UI 事件 → IPC → 主进程 Node.js 处理 → IPC → 渲染进程更新

**关键特性：**

- 跨平台：一套代码编译 Windows / macOS / Linux
- 完整 Node.js 能力：文件系统、原生模块、系统调用
- Chromium 渲染：CSS3、WebGL、DevTools 全部支持
- 自动更新：`electron-updater` 支持 GitHub/自定义服务器

## Why — 为什么

**适用场景：**

- 需要 Node.js 原生能力的桌面应用（文件操作、系统调用）
- Web 应用需要离线运行或系统集成
- 开发工具类应用（VS Code、Figma 都基于 Electron）
- 内部工具，团队以 Web 技术栈为主

**对比同类框架：**

| 维度 | Electron | Tauri | NW.js | Flutter Desktop | Qt |
|------|----------|-------|-------|-----------------|-----|
| 语言 | JS/TS | Rust + Web前端 | JS | Dart | C++ |
| 底层 | Chromium + Node | 系统 WebView | Chromium + Node | Skia | 自绘 |
| 包体积 | 大（~100MB+） | 小（~5MB） | 大 | 中 | 小 |
| 内存占用 | 高 | 低 | 高 | 中 | 低 |
| 原生能力 | Node.js | Rust FFI | Node.js | FFI | 原生 |
| 学习曲线 | 低 | 中 | 低 | 中 | 高 |
| 生态 | 最成熟 | 快速增长 | 一般 | 增长中 | 成熟 |

**优缺点：**

- ✅ 优点：
  - Web 技术栈直接复用，前端团队零门槛
  - 生态最成熟，社区资源丰富
  - Node.js 完整能力，原生模块支持好
  - DevTools 调试方便
  - 知名案例多（VS Code、Discord、Slack）
- ❌ 缺点：
  - 包体积大，Chromium 内核打包进去
  - 内存占用高，多进程架构开销大
  - 性能不如原生，CPU 密集型任务吃力
  - 安全风险，需严格限制渲染进程权限
  - 启动速度较慢

## How — 怎么用

### 快速上手

```bash
# 创建项目（推荐 electron-vite）
npm create @quick-start/electron my-app -- --template vue-ts
cd my-app
npm install
npm run dev
```

**项目结构：**

```
my-app/
├── src/
│   ├── main/          # 主进程代码
│   │   └── index.ts
│   ├── preload/       # Preload 脚本
│   │   └── index.ts
│   └── renderer/      # 渲染进程（Vue/React）
│       └── src/
├── electron.vite.config.ts
└── package.json
```

### 代码示例

**主进程 — 创建窗口：**

```ts
// src/main/index.ts
import { app, BrowserWindow, ipcMain } from 'electron';
import { join } from 'path';

let mainWindow: BrowserWindow | null = null;

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
    frame: false,              // 无边框窗口
    titleBarStyle: 'hidden',   // macOS 隐藏标题栏
    webPreferences: {
      preload: join(__dirname, '../preload/index.js'),
      sandbox: true,           // 开启沙箱
      contextIsolation: true,  // 上下文隔离（必须开启）
      nodeIntegration: false,  // 禁止渲染进程直接用 Node（必须）
    },
  });

  // 开发环境加载 dev server
  if (process.env.ELECTRON_RENDERER_URL) {
    mainWindow.loadURL(process.env.ELECTRON_RENDERER_URL);
  } else {
    mainWindow.loadFile(join(__dirname, '../renderer/index.html'));
  }
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

**Preload — 安全桥接：**

```ts
// src/preload/index.ts
import { contextBridge, ipcRenderer } from 'electron';

// 通过 contextBridge 暴露安全 API 到渲染进程
contextBridge.exposeInMainWorld('electronAPI', {
  // 文件操作
  selectFile: () => ipcRenderer.invoke('dialog:selectFile'),
  readFile: (path: string) => ipcRenderer.invoke('fs:readFile', path),

  // 系统信息
  getPlatform: () => process.platform,

  // 事件监听（单向）
  onUpdateAvailable: (callback: (info: any) => void) =>
    ipcRenderer.on('update:available', (_e, info) => callback(info)),
});
```

**IPC 通信 — 双向 invoke：**

```ts
// 主进程 — 注册 handler
ipcMain.handle('dialog:selectFile', async () => {
  const { canceled, filePaths } = await dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [{ name: 'Images', extensions: ['png', 'jpg'] }],
  });
  if (canceled) return null;
  return filePaths[0];
});

ipcMain.handle('fs:readFile', async (_e, filePath: string) => {
  const buffer = await fs.promises.readFile(filePath);
  return buffer.toString('base64');
});
```

```ts
// 渲染进程 — 调用（通过 preload 暴露的 API）
const filePath = await window.electronAPI.selectFile();
if (filePath) {
  const base64 = await window.electronAPI.readFile(filePath);
}
```

**IPC 通信 — 单向 send/on：**

```ts
// 渲染进程 → 主进程（不需要返回值）
ipcRenderer.send('notification:show', { title: '提示', body: '操作完成' });

// 主进程监听
ipcMain.on('notification:show', (_e, data) => {
  new Notification({ title: data.title, body: data.body }).show();
});

// 主进程 → 渲染进程
mainWindow.webContents.send('update:available', { version: '1.2.0' });
```

**系统托盘：**

```ts
import { Tray, Menu, nativeImage } from 'electron';

let tray: Tray | null = null;

app.whenReady().then(() => {
  const icon = nativeImage.createFromPath(join(__dirname, '../../resources/tray.png'));
  tray = new Tray(icon.resize({ width: 16, height: 16 }));

  const contextMenu = Menu.buildFromTemplate([
    { label: '显示窗口', click: () => mainWindow?.show() },
    { label: '退出', click: () => app.quit() },
  ]);

  tray.setToolTip('My App');
  tray.setContextMenu(contextMenu);
  tray.on('click', () => mainWindow?.show());
});
```

**自动更新：**

```ts
import { autoUpdater } from 'electron-updater';

autoUpdater.autoDownload = false;
autoUpdater.setFeedURL({ provider: 'github', owner: 'my-org', repo: 'my-app' });

app.whenReady().then(() => {
  autoUpdater.checkForUpdates();
});

autoUpdater.on('update-available', (info) => {
  mainWindow?.webContents.send('update:available', info);
  autoUpdater.downloadUpdate();
});

autoUpdater.on('update-downloaded', () => {
  mainWindow?.webContents.send('update:downloaded');
  // 可以提示用户重启
});

// 用户确认后安装
ipcMain.handle('update:install', () => {
  autoUpdater.quitAndInstall(false, true);
});
```

**无边框窗口拖拽：**

```css
/* 渲染进程 CSS — 自定义标题栏 */
.title-bar {
  -webkit-app-region: drag;      /* 可拖拽区域 */
  height: 32px;
  display: flex;
  align-items: center;
}

.title-bar button {
  -webkit-app-region: no-drag;   /* 按钮不可拖拽 */
}
```

```vue
<!-- 自定义标题栏 -->
<div class="title-bar">
  <span class="title">My App</span>
  <div class="window-controls">
    <button @click="minimize">─</button>
    <button @click="maximize">□</button>
    <button @click="close">×</button>
  </div>
</div>
```

```ts
// 主进程 — 窗口控制
ipcMain.handle('window:minimize', (e) => {
  BrowserWindow.fromWebContents(e.sender)?.minimize();
});
ipcMain.handle('window:maximize', (e) => {
  const win = BrowserWindow.fromWebContents(e.sender);
  win?.isMaximized() ? win.unmaximize() : win?.maximize();
});
ipcMain.handle('window:close', (e) => {
  BrowserWindow.fromWebContents(e.sender)?.close();
});
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 包体积过大 | 打包了完整 Chromium | 用 electron-builder 配置压缩/NSIS，或考虑 Tauri |
| 内存占用高 | 每个窗口独立渲染进程 | 减少窗口数，用 BrowserView 复用，及时销毁 |
| 渲染进程被 XSS 攻击 | 开启了 nodeIntegration | 必须关闭 nodeIntegration + 开启 contextIsolation |
| contextBridge 传函数失败 | 序列化限制 | 只暴露函数引用，实际逻辑在 preload 中执行 |
| 原生模块编译失败 | node-gyp 依赖问题 | 用 electron-rebuild 重新编译，或用 @electron/remote 替代 |
| macOS 签名公证失败 | 缺少 Apple 开发者证书 | 配置 CSC_LINK/CSC_KEY_PASSWORD，用 notarize |
| 窗口闪烁白屏 | 加载时背景默认白色 | 创建窗口时设置 `backgroundColor: '#1a1a1a'` |
| 启动慢 | Chromium 初始化开销 | 用 `app.whenReady()` 优化加载顺序，延迟创建窗口 |

### 最佳实践

- **安全第一**：永远关闭 `nodeIntegration`，开启 `contextIsolation` + `sandbox`
- **IPC 优先用 invoke/handle**：双向通信用 invoke，比 send/on 更安全（有返回值、错误处理）
- **Preload 最小暴露**：只暴露渲染进程真正需要的 API，不暴露整个 ipcRenderer
- **主进程做重活**：文件 I/O、加密、数据库操作放主进程，渲染进程只管 UI
- **拆分进程逻辑**：主进程只做窗口管理和系统交互，业务逻辑尽量放渲染进程
- **打包优化**：用 asar 打包、tree-shaking 去除无用代码、按平台打包

## 面试题

**Q1: Electron 的主进程和渲染进程有什么区别？它们如何通信？**
> 主进程运行 Node.js，负责创建窗口（BrowserWindow）、管理系统资源和原生交互，每个应用只有一个。渲染进程运行 Chromium，负责页面渲染和 UI 交互，可以有多个（多窗口）。通信通过 IPC：`ipcMain.handle` / `ipcRenderer.invoke` 做双向通信，`ipcRenderer.send` / `ipcMain.on` 做单向推送。

**Q2: Electron 中 contextIsolation 和 nodeIntegration 为什么重要？**
> `nodeIntegration: false` 禁止渲染进程直接使用 Node.js API，防止 XSS 攻击获得文件系统等能力；`contextIsolation: true` 让 preload 脚本和渲染页面的 JS 运行在隔离的 V8 上下文中，防止页面脚本篡改 preload 暴露的 API。两者结合是 Electron 最重要的安全防线，关闭任一项都会导致严重安全漏洞。

**Q3: Electron 和 Tauri 的核心区别是什么？**
> Electron 打包 Chromium + Node.js，体积大（~100MB+）但兼容性好、生态成熟；Tauri 用系统自带的 WebView + Rust 后端，体积小（~5MB）性能好但 WebView 兼容性依赖系统、Rust 学习成本高。选 Electron：需要最强兼容性和成熟生态；选 Tauri：追求小体积、低内存和安全性。

**Q4: 如何优化 Electron 应用的启动速度和内存占用？**
> 启动优化：延迟创建非首屏窗口、用 `show: false` 创建窗口等 `ready-to-show` 再显示、预加载必要资源。内存优化：减少窗口数量、及时销毁关闭的窗口（`win = null`）、用 BrowserView 替代多窗口、避免在渲染进程中缓存大量数据。还可以用 `app.commandLine.appendSwitch` 关闭不需要的 Chromium 特性。

**Q5: Electron 的 Preload 脚本作用是什么？为什么不直接在渲染进程中使用 Node.js？**
> Preload 在渲染进程的页面加载前执行，通过 `contextBridge.exposeInMainWorld` 安全地向渲染进程暴露有限 API。不在渲染进程直接使用 Node.js 是因为：渲染进程加载不可信的网页内容，如果直接拥有 Node.js 能力，XSS 攻击就能读写文件、执行系统命令。Preload 作为中间层，只暴露必要的、经过验证的接口，实现最小权限原则。

---

**相关链接：**
- [[Vue核心]]
- [[Webpack与Vite]]
- [[Web安全XSS与CSRF]]
- [[浏览器渲染原理]]

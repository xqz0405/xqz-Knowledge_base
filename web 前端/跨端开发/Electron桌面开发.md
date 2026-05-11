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

> Electron 是 GitHub 推出的桌面应用开发框架，将 Chromium 和 Node.js 合并到同一个运行时中，让开发者用 Web 技术（HTML/CSS/JS）构建跨平台桌面应用。VS Code、Discord、Slack、Figma 等知名应用均基于 Electron。

**核心概念：**

- **主进程（Main Process）**：运行 Node.js，管理窗口和系统交互，每个应用只有一个
- **渲染进程（Renderer Process）**：运行 Chromium，负责页面渲染，可有多个
- **IPC 通信**：主进程与渲染进程通过 `ipcMain` / `ipcRenderer` 通信
- **Preload 脚本**：在渲染进程加载前运行，安全地桥接主进程和渲染进程
- **BrowserWindow**：窗口管理类，创建和控制应用窗口
- **Tray**：系统托盘，常驻后台运行

**核心架构：**

```
┌─────────────────────────────────────────────┐
│                  Electron App                │
├──────────────────┬──────────────────────────┤
│   Main Process   │   Renderer Process(es)   │
│   (Node.js)      │   (Chromium)             │
│                  │                          │
│  - BrowserWindow │  - Vue/React UI          │
│  - IPC Handler   │  - User Interaction      │
│  - System API    │  - DOM Rendering         │
│  - File System   │                          │
│  - Native Module │  Preload Bridge          │
│  - Tray/Menu     │  - contextBridge         │
│                  │  - ipcRenderer.invoke    │
├──────────────────┴──────────────────────────┤
│            IPC (Inter-Process Communication) │
└─────────────────────────────────────────────┘
```

- 设计理念：Web 技术 + Node.js 能力 = 桌面应用
- 核心模块：BrowserWindow、ipcMain/ipcRenderer、BrowserView、Tray、Menu、net、shell
- 数据流：用户操作 → 渲染进程 UI 事件 → IPC → 主进程 Node.js 处理 → IPC → 渲染进程更新

**进程模型：**

| 进程 | 运行环境 | 数量 | 职责 |
|------|---------|------|------|
| 主进程 | Node.js | 1 | 窗口管理、系统交互、原生 API |
| 渲染进程 | Chromium | N | 页面渲染、用户交互 |
| GPU 进程 | Chromium | 1 | GPU 加速渲染 |
| Preload | Node.js(受限) | N/窗口 | 安全桥接 |
| Utility | Node.js | N | 网络请求等辅助任务 |

**关键特性：**

- 跨平台：一套代码编译 Windows / macOS / Linux
- 完整 Node.js 能力：文件系统、原生模块、系统调用、子进程
- Chromium 渲染：CSS3、WebGL、DevTools 全部支持
- 自动更新：`electron-updater` 支持 GitHub/自定义服务器
- 原生集成：系统托盘、通知、菜单、快捷键、剪贴板

## Why — 为什么

**适用场景：**

- 需要 Node.js 原生能力的桌面应用（文件操作、系统调用）
- Web 应用需要离线运行或系统集成
- 开发工具类应用（VS Code、Figma 都基于 Electron）
- 内部工具，团队以 Web 技术栈为主
- 需要跨平台但不想维护三套原生代码

**知名应用案例：**

| 应用 | 说明 | 特色 |
|------|------|------|
| VS Code | 微软代码编辑器 | 插件生态、性能优化标杆 |
| Discord | 语音通讯 | 原生集成、自动更新 |
| Slack | 企业协作 | 多窗口、系统托盘 |
| Figma | 设计工具 | WebGL 渲染、高性能 |
| Notion | 笔记工具 | 离线缓存、跨平台 |
| Obsidian | 知识管理 | 本地文件、插件系统 |

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
| WebView 一致性 | 极好（自带 Chromium） | 依赖系统 | 极好 | N/A | N/A |

**优缺点：**

- ✅ 优点：
  - Web 技术栈直接复用，前端团队零门槛
  - 生态最成熟，社区资源丰富
  - Node.js 完整能力，原生模块支持好
  - DevTools 调试方便，开发体验好
  - 知名案例多，最佳实践丰富
  - Chromium 渲染一致性，无兼容性问题
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

# 或使用 Electron Forge
npm init electron-app@latest my-app -- --template=vite-typescript

# 或手动搭建
mkdir my-app && cd my-app
npm init -y
npm install electron --save-dev
npm install electron-builder --save-dev
```

**项目结构：**

```
my-app/
├── src/
│   ├── main/              # 主进程代码
│   │   ├── index.ts       # 入口
│   │   ├── window.ts      # 窗口管理
│   │   ├── ipc.ts         # IPC 注册
│   │   ├── tray.ts        # 托盘
│   │   └── menu.ts        # 菜单
│   ├── preload/           # Preload 脚本
│   │   └── index.ts
│   └── renderer/          # 渲染进程（Vue/React）
│       └── src/
├── resources/             # 静态资源（图标、安装包素材）
│   ├── icon.ico           # Windows 图标
│   ├── icon.icns          # macOS 图标
│   └── tray.png           # 托盘图标
├── electron.vite.config.ts
├── electron-builder.yml   # 打包配置
└── package.json
```

### 主进程 — 窗口管理

**创建窗口：**

```ts
// src/main/window.ts
import { app, BrowserWindow, screen } from 'electron';
import { join } from 'path';
import isDev from 'electron-is-dev';

interface WindowOptions {
  width?: number;
  height?: number;
  url?: string;
  file?: string;
}

const windows = new Map<string, BrowserWindow>();

export function createWindow(name: string, options: WindowOptions = {}) {
  const { width = 1200, height = 800 } = options;

  // 获取窗口上次位置
  const bounds = loadWindowBounds(name);
  const win = new BrowserWindow({
    width: bounds?.width || width,
    height: bounds?.height || height,
    x: bounds?.x,
    y: bounds?.y,
    minWidth: 800,
    minHeight: 600,
    frame: false,              // 无边框窗口
    titleBarStyle: 'hidden',   // macOS 隐藏标题栏（保留交通灯按钮）
    backgroundColor: '#1a1a1a', // 防止白屏闪烁
    show: false,               // 先隐藏，ready-to-show 后再显示
    webPreferences: {
      preload: join(__dirname, '../preload/index.js'),
      sandbox: true,
      contextIsolation: true,
      nodeIntegration: false,
    },
  });

  // 窗口准备好再显示，避免白屏
  win.once('ready-to-show', () => {
    win.show();
  });

  // 记住窗口位置
  win.on('close', () => {
    saveWindowBounds(name, win.getBounds());
    windows.delete(name);
  });

  // 加载内容
  if (isDev && process.env.ELECTRON_RENDERER_URL) {
    win.loadURL(process.env.ELECTRON_RENDERER_URL);
    win.webContents.openDevTools();
  } else {
    win.loadFile(join(__dirname, `../renderer/${options.file || 'index'}.html`));
  }

  windows.set(name, win);
  return win;
}

// 窗口位置持久化
function loadWindowBounds(name: string) {
  try {
    return JSON.parse(localStorage.getItem(`window-bounds-${name}`) || 'null');
  } catch { return null; }
}

function saveWindowBounds(name: string, bounds: Electron.Rectangle) {
  localStorage.setItem(`window-bounds-${name}`, JSON.stringify(bounds));
}

export function getWindow(name: string) {
  return windows.get(name);
}
```

**多窗口管理：**

```ts
// src/main/index.ts
import { app } from 'electron';
import { createWindow } from './window';

app.whenReady().then(() => {
  createWindow('main', { width: 1200, height: 800 });
});

// macOS 点击 dock 图标时重新创建窗口
app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow('main');
  }
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});
```

**父子窗口与模态窗口：**

```ts
// 模态窗口（阻塞父窗口）
const parent = getWindow('main');
const modal = new BrowserWindow({
  width: 500,
  height: 400,
  parent,                    // 指定父窗口
  modal: true,               // 模态窗口
  show: false,
});

modal.once('ready-to-show', () => modal.show());
modal.loadFile(join(__dirname, '../renderer/modal.html'));

// 子窗口跟随父窗口
parent.on('move', () => {
  const [x, y] = parent.getPosition();
  child.setPosition(x + 20, y + 20);
});
```

**BrowserView — 嵌入网页：**

```ts
// 在主窗口中嵌入另一个网页（如帮助文档）
import { BrowserView } from 'electron';

const view = new BrowserView({
  webPreferences: {
    nodeIntegration: false,
    contextIsolation: true,
  }
});

mainWindow.setBrowserView(view);
view.setBounds({ x: 0, y: 40, width: 1200, height: 760 });
view.webContents.loadURL('https://docs.example.com');

// 移除
mainWindow.setBrowserView(null);
view.webContents.destroy();
```

### Preload — 安全桥接

```ts
// src/preload/index.ts
import { contextBridge, ipcRenderer } from 'electron';

// 通过 contextBridge 暴露安全 API 到渲染进程
contextBridge.exposeInMainWorld('electronAPI', {
  // === 文件操作 ===
  selectFile: () => ipcRenderer.invoke('dialog:selectFile'),
  selectFolder: () => ipcRenderer.invoke('dialog:selectFolder'),
  readFile: (path: string) => ipcRenderer.invoke('fs:readFile', path),
  writeFile: (path: string, content: string) =>
    ipcRenderer.invoke('fs:writeFile', path, content),

  // === 系统信息 ===
  getPlatform: () => process.platform,
  getAppVersion: () => ipcRenderer.invoke('app:getVersion'),
  getSystemInfo: () => ipcRenderer.invoke('system:getInfo'),

  // === 窗口控制 ===
  minimize: () => ipcRenderer.invoke('window:minimize'),
  maximize: () => ipcRenderer.invoke('window:maximize'),
  close: () => ipcRenderer.invoke('window:close'),
  isMaximized: () => ipcRenderer.invoke('window:isMaximized'),

  // === 事件监听（主进程 → 渲染进程） ===
  onUpdateAvailable: (callback: (info: any) => void) =>
    ipcRenderer.on('update:available', (_e, info) => callback(info)),
  onUpdateDownloaded: (callback: () => void) =>
    ipcRenderer.on('update:downloaded', () => callback()),
  onDeepLink: (callback: (url: string) => void) =>
    ipcRenderer.on('deep-link', (_e, url) => callback(url)),

  // === 清理监听 ===
  removeAllListeners: (channel: string) => ipcRenderer.removeAllListeners(channel),
});
```

**TypeScript 类型声明：**

```ts
// src/preload/index.d.ts
export interface ElectronAPI {
  selectFile: () => Promise<string | null>;
  selectFolder: () => Promise<string | null>;
  readFile: (path: string) => Promise<string>;
  writeFile: (path: string, content: string) => Promise<void>;
  getPlatform: () => string;
  getAppVersion: () => Promise<string>;
  getSystemInfo: () => Promise<SystemInfo>;
  minimize: () => Promise<void>;
  maximize: () => Promise<void>;
  close: () => Promise<void>;
  isMaximized: () => Promise<boolean>;
  onUpdateAvailable: (callback: (info: any) => void) => void;
  onUpdateDownloaded: (callback: () => void) => void;
  onDeepLink: (callback: (url: string) => void) => void;
  removeAllListeners: (channel: string) => void;
}

declare global {
  interface Window {
    electronAPI: ElectronAPI;
  }
}
```

### IPC 通信详解

**三种通信方式对比：**

| 方式 | 方向 | 返回值 | 使用场景 |
|------|------|--------|----------|
| `invoke/handle` | 双向 | Promise | 请求-响应模式（最常用） |
| `send/on` | 单向 | 无 | 事件通知 |
| `postMessage` | 双向 | 无 | 跨上下文通信 |

**双向通信 — invoke/handle（推荐）：**

```ts
// 主进程 — 注册 handler
ipcMain.handle('dialog:selectFile', async () => {
  const { canceled, filePaths } = await dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [
      { name: 'Images', extensions: ['png', 'jpg', 'gif'] },
      { name: 'Documents', extensions: ['pdf', 'doc', 'docx'] },
      { name: 'All Files', extensions: ['*'] },
    ],
  });
  return canceled ? null : filePaths[0];
});

ipcMain.handle('dialog:selectFolder', async () => {
  const { canceled, filePaths } = await dialog.showOpenDialog({
    properties: ['openDirectory'],
  });
  return canceled ? null : filePaths[0];
});

ipcMain.handle('fs:readFile', async (_e, filePath: string) => {
  try {
    const buffer = await fs.promises.readFile(filePath);
    return buffer.toString('base64');
  } catch (err) {
    throw new Error(`读取文件失败: ${err.message}`);
  }
});

ipcMain.handle('fs:writeFile', async (_e, filePath: string, content: string) => {
  await fs.promises.writeFile(filePath, content, 'utf-8');
});
```

```ts
// 渲染进程 — 调用
const filePath = await window.electronAPI.selectFile();
if (filePath) {
  const base64 = await window.electronAPI.readFile(filePath);
}
```

**单向通信 — send/on：**

```ts
// 渲染进程 → 主进程（不需要返回值）
// 在 preload 中
contextBridge.exposeInMainWorld('electronAPI', {
  notify: (data: any) => ipcRenderer.send('notification:show', data),
});

// 主进程监听
ipcMain.on('notification:show', (_e, data) => {
  new Notification({ title: data.title, body: data.body }).show();
});

// 主进程 → 渲染进程
mainWindow.webContents.send('update:available', { version: '1.2.0' });
```

**MessagePort — 双向流式通信：**

```ts
// 主进程 — 创建 MessageChannel
ipcMain.handle('chat:createChannel', (event) => {
  const { port1, port2 } = new MessageChannelMain();
  // port1 给主进程用
  port1.on('message', (e) => {
    console.log('渲染进程消息:', e.data);
    port1.postMessage({ reply: '收到' });
  });
  port1.start();
  // port2 发给渲染进程
  event.senderFrame.postMessage('chat:port', null, [port2]);
  return true;
});
```

**安全注意事项：**

```ts
// ❌ 危险：暴露整个 ipcRenderer
contextBridge.exposeInMainWorld('ipc', ipcRenderer);

// ✅ 安全：只暴露特定方法
contextBridge.exposeInMainWorld('electronAPI', {
  selectFile: () => ipcRenderer.invoke('dialog:selectFile'),
});

// ❌ 危险：直接在 handler 中信任渲染进程传来的路径
ipcMain.handle('fs:readFile', async (_e, filePath: string) => {
  return fs.readFileSync(filePath); // 路径遍历攻击！
});

// ✅ 安全：验证路径
ipcMain.handle('fs:readFile', async (_e, filePath: string) => {
  const resolved = path.resolve(filePath);
  if (!resolved.startsWith(APP_DATA_DIR)) {
    throw new Error('路径越权');
  }
  return fs.readFileSync(resolved);
});
```

### 数据持久化

**electron-store — 轻量 JSON 存储：**

```ts
// src/main/store.ts
import Store from 'electron-store';

interface StoreSchema {
  windowBounds: Record<string, Electron.Rectangle>;
  recentFiles: string[];
  settings: {
    theme: 'light' | 'dark' | 'system';
    language: string;
    autoUpdate: boolean;
    minimizeToTray: boolean;
  };
}

const store = new Store<StoreSchema>({
  defaults: {
    windowBounds: {},
    recentFiles: [],
    settings: {
      theme: 'system',
      language: 'zh-CN',
      autoUpdate: true,
      minimizeToTray: false,
    },
  },
  encryptionKey: 'your-encryption-key', // 加密敏感数据
});

export default store;

// 使用
store.set('settings.theme', 'dark');
store.get('settings.theme'); // 'dark'
store.delete('recentFiles');
store.has('settings'); // true
```

**better-sqlite3 — SQLite 数据库：**

```ts
// src/main/database.ts
import Database from 'better-sqlite3';
import { app } from 'electron';
import { join } from 'path';

const dbPath = join(app.getPath('userData'), 'app.db');
const db = new Database(dbPath);

// 启用 WAL 模式（提升并发性能）
db.pragma('journal_mode = WAL');

// 创建表
db.exec(`
  CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT DEFAULT '',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
  );

  CREATE INDEX IF NOT EXISTS idx_notes_updated ON notes(updated_at);
`);

// 预编译语句（性能优化）
const insertNote = db.prepare('INSERT INTO notes (title, content) VALUES (?, ?)');
const getNote = db.prepare('SELECT * FROM notes WHERE id = ?');
const listNotes = db.prepare('SELECT * FROM notes ORDER BY updated_at DESC LIMIT ? OFFSET ?');
const updateNote = db.prepare('UPDATE notes SET title = ?, content = ?, updated_at = CURRENT_TIMESTAMP WHERE id = ?');
const deleteNote = db.prepare('DELETE FROM notes WHERE id = ?');

// 事务
const insertMany = db.transaction((notes: { title: string; content: string }[]) => {
  for (const note of notes) {
    insertNote.run(note.title, note.content);
  }
});

// IPC 注册
ipcMain.handle('db:insertNote', (_e, title: string, content: string) => {
  const result = insertNote.run(title, content);
  return result.lastInsertRowid;
});

ipcMain.handle('db:getNote', (_e, id: number) => {
  return getNote.get(id);
});

ipcMain.handle('db:listNotes', (_e, limit: number, offset: number) => {
  return listNotes.all(limit, offset);
});

ipcMain.handle('db:updateNote', (_e, id: number, title: string, content: string) => {
  return updateNote.run(title, content, id);
});

ipcMain.handle('db:deleteNote', (_e, id: number) => {
  return deleteNote.run(id);
});

// 应用退出时关闭
app.on('before-quit', () => db.close());
```

**文件存储 — 大文件/二进制：**

```ts
// src/main/fileStorage.ts
import { app } from 'electron';
import { join } from 'path';
import fs from 'fs/promises';

const DATA_DIR = join(app.getPath('userData'), 'data');

export async function saveBuffer(filename: string, buffer: Buffer) {
  await fs.mkdir(DATA_DIR, { recursive: true });
  const filePath = join(DATA_DIR, filename);
  await fs.writeFile(filePath, buffer);
  return filePath;
}

export async function readBuffer(filename: string) {
  return fs.readFile(join(DATA_DIR, filename));
}

export async function deleteFile(filename: string) {
  return fs.unlink(join(DATA_DIR, filename));
}

export async function listFiles() {
  await fs.mkdir(DATA_DIR, { recursive: true });
  return fs.readdir(DATA_DIR);
}
```

### 原生菜单与快捷键

**应用菜单：**

```ts
// src/main/menu.ts
import { Menu, shell, app, BrowserWindow } from 'electron';

const isMac = process.platform === 'darwin';

export function createAppMenu() {
  const template: Electron.MenuItemConstructorOptions[] = [
    // macOS 应用菜单
    ...(isMac ? [{
      label: app.name,
      submenu: [
        { label: `关于 ${app.name}`, role: 'about' },
        { type: 'separator' },
        { label: '偏好设置', accelerator: 'CmdOrCtrl+,', click: openSettings },
        { type: 'separator' },
        { label: '隐藏', role: 'hide' },
        { label: '隐藏其他', role: 'hideOthers' },
        { label: '显示全部', role: 'unhide' },
        { type: 'separator' },
        { label: '退出', accelerator: 'CmdOrCtrl+Q', role: 'quit' },
      ],
    }] : []),
    {
      label: '文件',
      submenu: [
        { label: '新建', accelerator: 'CmdOrCtrl+N', click: handleNew },
        { label: '打开', accelerator: 'CmdOrCtrl+O', click: handleOpen },
        { label: '保存', accelerator: 'CmdOrCtrl+S', click: handleSave },
        { label: '另存为', accelerator: 'CmdOrCtrl+Shift+S', click: handleSaveAs },
        { type: 'separator' },
        { label: '最近打开', role: 'recentDocuments' },
        { type: 'separator' },
        isMac ? { label: '关闭窗口', role: 'close' } : { label: '退出', role: 'quit' },
      ],
    },
    {
      label: '编辑',
      submenu: [
        { label: '撤销', role: 'undo' },
        { label: '重做', role: 'redo' },
        { type: 'separator' },
        { label: '剪切', role: 'cut' },
        { label: '复制', role: 'copy' },
        { label: '粘贴', role: 'paste' },
        { label: '全选', role: 'selectAll' },
      ],
    },
    {
      label: '视图',
      submenu: [
        { label: '重新加载', role: 'reload' },
        { label: '强制重新加载', role: 'forceReload' },
        { label: '开发者工具', role: 'toggleDevTools' },
        { type: 'separator' },
        { label: '放大', role: 'zoomIn' },
        { label: '缩小', role: 'zoomOut' },
        { label: '重置缩放', role: 'resetZoom' },
        { type: 'separator' },
        { label: '全屏', role: 'togglefullscreen' },
      ],
    },
    {
      label: '帮助',
      submenu: [
        { label: '文档', click: () => shell.openExternal('https://docs.example.com') },
        { label: '检查更新', click: checkForUpdates },
        { type: 'separator' },
        { label: '关于', click: showAboutDialog },
      ],
    },
  ];

  const menu = Menu.buildFromTemplate(template);
  Menu.setApplicationMenu(menu);
}
```

**右键上下文菜单：**

```ts
import { Menu } from 'electron';

// 主进程监听右键菜单请求
ipcMain.handle('context-menu:show', (event, options) => {
  const menu = Menu.buildFromTemplate([
    { label: '复制', role: 'copy' },
    { label: '粘贴', role: 'paste' },
    { type: 'separator' },
    { label: '剪切', role: 'cut' },
    { type: 'separator' },
    {
      label: '另存为...',
      click: () => {
        BrowserWindow.fromWebContents(event.sender)?.webContents.send('context-menu:save-as');
      }
    },
  ]);

  menu.popup({ window: BrowserWindow.fromWebContents(event.sender)! });
});
```

**全局快捷键：**

```ts
import { globalShortcut, app } from 'electron';

app.whenReady().then(() => {
  // 注册全局快捷键（应用未聚焦时也生效）
  const ret = globalShortcut.register('CommandOrControl+Shift+V', () => {
    mainWindow.show();
    mainWindow.webContents.send('shortcut:paste-special');
  });

  if (!ret) console.error('快捷键注册失败');

  // 检查是否注册成功
  console.log(globalShortcut.isRegistered('CommandOrControl+Shift+V'));
});

// 应用退出时注销所有快捷键
app.on('will-quit', () => {
  globalShortcut.unregisterAll();
});
```

### 系统托盘

```ts
// src/main/tray.ts
import { Tray, Menu, nativeImage, app, BrowserWindow } from 'electron';
import { join } from 'path';
import store from './store';

let tray: Tray | null = null;

export function createTray(mainWindow: BrowserWindow) {
  const iconPath = join(__dirname, '../../resources/tray.png');
  const icon = nativeImage.createFromPath(iconPath).resize({ width: 16, height: 16 });

  tray = new Tray(icon);
  tray.setToolTip('My App');

  const contextMenu = Menu.buildFromTemplate([
    { label: '显示主窗口', click: () => {
      mainWindow.show();
      mainWindow.focus();
    }},
    { type: 'separator' },
    { label: '新建', accelerator: 'CmdOrCtrl+N', click: () => {
      mainWindow.show();
      mainWindow.webContents.send('action:new');
    }},
    { type: 'separator' },
    { label: '偏好设置', click: () => {
      mainWindow.show();
      mainWindow.webContents.send('navigate:settings');
    }},
    { type: 'separator' },
    { label: '退出', click: () => app.quit() },
  ]);

  tray.setContextMenu(contextMenu);

  // 点击托盘图标显示窗口（Windows/Linux）
  tray.on('click', () => {
    if (mainWindow.isVisible()) {
      mainWindow.hide();
    } else {
      mainWindow.show();
      mainWindow.focus();
    }
  });

  // 最小化到托盘
  mainWindow.on('close', (event) => {
    if (store.get('settings.minimizeToTray')) {
      event.preventDefault();
      mainWindow.hide();
    }
  });
}
```

### 系统对话框与剪贴板

```ts
import { dialog, clipboard, nativeImage, shell } from 'electron';
import fs from 'fs/promises';

// === 文件对话框 ===

// 打开文件
ipcMain.handle('dialog:selectFile', async () => {
  const { canceled, filePaths } = await dialog.showOpenDialog({
    title: '选择文件',
    properties: ['openFile', 'multiSelections'],
    filters: [
      { name: '图片', extensions: ['png', 'jpg', 'gif', 'webp'] },
      { name: '文档', extensions: ['pdf', 'doc', 'docx', 'txt'] },
      { name: '所有文件', extensions: ['*'] },
    ],
  });
  return canceled ? null : filePaths;
});

// 保存文件
ipcMain.handle('dialog:saveFile', async (_e, defaultName: string, content: string) => {
  const { canceled, filePath } = await dialog.showSaveDialog({
    title: '保存文件',
    defaultPath: defaultName,
    filters: [{ name: 'Text', extensions: ['txt'] }],
  });
  if (canceled || !filePath) return null;
  await fs.writeFile(filePath, content, 'utf-8');
  return filePath;
});

// 错误对话框
ipcMain.handle('dialog:showError', (_e, title: string, message: string) => {
  dialog.showErrorBox(title, message);
});

// === 剪贴板 ===

// 读取剪贴板文本
ipcMain.handle('clipboard:readText', () => {
  return clipboard.readText();
});

// 写入剪贴板文本
ipcMain.handle('clipboard:writeText', (_e, text: string) => {
  clipboard.writeText(text);
});

// 读取剪贴板图片
ipcMain.handle('clipboard:readImage', () => {
  const image = clipboard.readImage();
  return image.isEmpty() ? null : image.toDataURL();
});

// 写入剪贴板图片
ipcMain.handle('clipboard:writeImage', (_e, dataUrl: string) => {
  const image = nativeImage.createFromDataURL(dataUrl);
  clipboard.writeImage(image);
});

// === shell ===

// 在默认浏览器打开链接
ipcMain.handle('shell:openExternal', (_e, url: string) => {
  // 安全检查：只允许 http/https 协议
  if (!url.startsWith('http://') && !url.startsWith('https://')) {
    throw new Error('不允许的协议');
  }
  shell.openExternal(url);
});

// 在文件管理器中显示
ipcMain.handle('shell:showItemInFolder', (_e, fullPath: string) => {
  shell.showItemInFolder(fullPath);
});

// 打开文件
ipcMain.handle('shell:openPath', async (_e, path: string) => {
  return shell.openPath(path);
});
```

### 无边框窗口与自定义标题栏

**主进程窗口控制：**

```ts
ipcMain.handle('window:minimize', (e) => {
  BrowserWindow.fromWebContents(e.sender)?.minimize();
});

ipcMain.handle('window:maximize', (e) => {
  const win = BrowserWindow.fromWebContents(e.sender);
  if (!win) return;
  win.isMaximized() ? win.unmaximize() : win.maximize();
});

ipcMain.handle('window:close', (e) => {
  BrowserWindow.fromWebContents(e.sender)?.close();
});

ipcMain.handle('window:isMaximized', (e) => {
  return BrowserWindow.fromWebContents(e.sender)?.isMaximized();
});

// 监听最大化状态变化
mainWindow.on('maximize', () => {
  mainWindow.webContents.send('window:maximized-changed', true);
});
mainWindow.on('unmaximize', () => {
  mainWindow.webContents.send('window:maximized-changed', false);
});
```

**渲染进程 — 自定义标题栏组件：**

```vue
<!-- TitleBar.vue -->
<template>
  <div class="title-bar">
    <div class="drag-area">
      <img src="/logo.png" class="app-icon" />
      <span class="app-title">My App</span>
    </div>
    <div class="window-controls">
      <button class="control-btn" @click="minimize" title="最小化">
        <svg width="12" height="12"><line x1="0" y1="6" x2="12" y2="6" /></svg>
      </button>
      <button class="control-btn" @click="toggleMaximize" title="最大化">
        <svg v-if="!isMaximized" width="12" height="12"><rect x="1" y="1" width="10" height="10" /></svg>
        <svg v-else width="12" height="12"><rect x="3" y="0" width="9" height="9" /><rect x="0" y="3" width="9" height="9" /></svg>
      </button>
      <button class="control-btn close" @click="close" title="关闭">
        <svg width="12" height="12"><line x1="0" y1="0" x2="12" y2="12" /><line x1="12" y1="0" x2="0" y2="12" /></svg>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

const isMaximized = ref(false);

const minimize = () => window.electronAPI.minimize();
const toggleMaximize = () => window.electronAPI.maximize();
const close = () => window.electronAPI.close();

onMounted(async () => {
  isMaximized.value = await window.electronAPI.isMaximized();
  window.electronAPI.onMaximizedChanged?.((maximized: boolean) => {
    isMaximized.value = maximized;
  });
});
</script>

<style scoped>
.title-bar {
  -webkit-app-region: drag;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #1e1e1e;
  user-select: none;
}

.drag-area {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-left: 12px;
}

.app-icon { width: 16px; height: 16px; }
.app-title { font-size: 13px; color: #ccc; }

.window-controls {
  -webkit-app-region: no-drag;
  display: flex;
  height: 100%;
}

.control-btn {
  width: 46px;
  height: 100%;
  border: none;
  background: transparent;
  color: #ccc;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.control-btn:hover { background: rgba(255,255,255,0.1); }
.control-btn.close:hover { background: #e81123; color: white; }
</style>
```

### 自动更新

```ts
// src/main/updater.ts
import { autoUpdater } from 'electron-updater';
import { BrowserWindow } from 'electron';

export function setupAutoUpdater(mainWindow: BrowserWindow) {
  autoUpdater.autoDownload = false;
  autoUpdater.autoInstallOnAppQuit = true;

  // 配置更新源
  autoUpdater.setFeedURL({
    provider: 'github',
    owner: 'my-org',
    repo: 'my-app',
  });

  // 检查更新（启动时 + 定时）
  app.whenReady().then(() => {
    autoUpdater.checkForUpdates();
    // 每 4 小时检查一次
    setInterval(() => autoUpdater.checkForUpdates(), 4 * 60 * 60 * 1000);
  });

  autoUpdater.on('checking-for-update', () => {
    mainWindow.webContents.send('update:checking');
  });

  autoUpdater.on('update-available', (info) => {
    mainWindow.webContents.send('update:available', {
      version: info.version,
      releaseNotes: info.releaseNotes,
    });
  });

  autoUpdater.on('update-not-available', () => {
    mainWindow.webContents.send('update:not-available');
  });

  autoUpdater.on('error', (err) => {
    mainWindow.webContents.send('update:error', err.message);
  });

  autoUpdater.on('download-progress', (progress) => {
    mainWindow.webContents.send('update:progress', {
      percent: progress.percent,
      transferred: progress.transferred,
      total: progress.total,
    });
  });

  autoUpdater.on('update-downloaded', () => {
    mainWindow.webContents.send('update:downloaded');
  });

  // IPC — 手动检查更新
  ipcMain.handle('update:check', () => autoUpdater.checkForUpdates());

  // IPC — 下载更新
  ipcMain.handle('update:download', () => autoUpdater.downloadUpdate());

  // IPC — 安装更新并重启
  ipcMain.handle('update:install', () => {
    autoUpdater.quitAndInstall(false, true);
  });
}
```

**渲染进程更新 UI：**

```vue
<!-- UpdateNotification.vue -->
<template>
  <Transition name="slide">
    <div v-if="updateInfo" class="update-banner">
      <span>新版本 v{{ updateInfo.version }} 可用</span>
      <button v-if="!downloading" @click="downloadUpdate">立即更新</button>
      <div v-else class="progress-bar">
        <div class="progress-fill" :style="{ width: progress + '%' }" />
      </div>
      <button v-if="readyToInstall" @click="installUpdate">重启安装</button>
      <button class="dismiss" @click="dismiss">稍后</button>
    </div>
  </Transition>
</template>
```

### 安全最佳实践

**安全配置清单：**

```ts
// src/main/security.ts
import { app, session } from 'electron';

app.whenReady().then(() => {
  // 1. 设置 CSP（内容安全策略）
  session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
    callback({
      responseHeaders: {
        ...details.responseHeaders,
        'Content-Security-Policy': [
          "default-src 'self'; " +
          "script-src 'self'; " +
          "style-src 'self' 'unsafe-inline'; " +
          "img-src 'self' data: https:; " +
          "connect-src 'self' https://api.example.com; " +
          "font-src 'self';"
        ],
      },
    });
  });

  // 2. 禁止导航到外部网站
  session.defaultSession.webRequest.onBeforeRequest((details, callback) => {
    const url = new URL(details.url);
    if (url.protocol === 'http:' || url.protocol === 'https:') {
      if (!url.hostname.endsWith('example.com')) {
        // 在外部浏览器打开
        shell.openExternal(details.url);
        callback({ cancel: true });
        return;
      }
    }
    callback({});
  });

  // 3. 禁止新窗口打开
  mainWindow.webContents.setWindowOpenHandler(() => {
    return { action: 'deny' };
  });

  // 4. 禁用 remote 模块
  // 不安装 @electron/remote（已废弃）
});

// 5. 验证 IPC 来源
ipcMain.handle('sensitive:action', (event) => {
  // 验证请求来自受信任的渲染进程
  const frameUrl = event.senderFrame.url;
  if (!frameUrl.startsWith('file://') && !frameUrl.includes('localhost')) {
    throw new Error('未授权的请求来源');
  }
  // 执行敏感操作
});
```

**webPreferences 安全配置：**

```ts
const secureWebPreferences: Electron.WebPreferences = {
  nodeIntegration: false,       // ❗ 必须关闭
  contextIsolation: true,       // ❗ 必须开启
  sandbox: true,                // ❗ 推荐：沙箱隔离
  webSecurity: true,            // 同源策略
  allowRunningInsecureContent: false,
  enableRemoteModule: false,    // 禁用 remote
  preload: join(__dirname, '../preload/index.js'),
};
```

### 打包与分发

**electron-builder 配置：**

```yaml
# electron-builder.yml
appId: com.example.myapp
productName: My App
copyright: Copyright © 2026

directories:
  output: dist
  buildResources: resources

files:
  - src/main/**/*
  - src/preload/**/*
  - src/renderer/dist/**/*
  - package.json
  - "!**/*.map"

# Windows
win:
  target:
    - target: nsis
      arch: [x64]
  icon: resources/icon.ico
  artifactName: ${name}-${version}-setup.${ext}

nsis:
  oneClick: false
  allowToChangeInstallationDirectory: true
  installerIcon: resources/icon.ico
  uninstallerIcon: resources/icon.ico
  installerHeaderIcon: resources/icon.ico
  createDesktopShortcut: true
  createStartMenuShortcut: true
  shortcutName: My App

# macOS
mac:
  target:
    - target: dmg
      arch: [x64, arm64]
  icon: resources/icon.icns
  category: public.app-category.productivity
  hardenedRuntime: true
  gatekeeperAssess: false
  entitlements: resources/entitlements.mac.plist
  entitlementsInherit: resources/entitlements.mac.plist

dmg:
  title: ${name} ${version}
  artifactName: ${name}-${version}-${arch}.${ext}

# Linux
linux:
  target:
    - target: AppImage
      arch: [x64]
    - target: deb
      arch: [x64]
  icon: resources
  category: Development

# 自动更新
publish:
  provider: github
  owner: my-org
  repo: my-app
```

**打包命令：**

```bash
# 打包当前平台
npm run build

# 打包指定平台
npx electron-builder --win
npx electron-builder --mac
npx electron-builder --linux

# 打包所有平台
npx electron-builder --win --mac --linux

# 只生成不打包（调试）
npx electron-builder --dir
```

**代码签名（macOS）：**

```bash
# 设置环境变量
export CSC_LINK=/path/to/certificate.p12
export CSC_KEY_PASSWORD=your-password
export APPLE_ID=your@email.com
export APPLE_APP_SPECIFIC_PASSWORD=xxxx-xxxx-xxxx-xxxx
export APPLE_TEAM_ID=XXXXXXXXXX

# electron-builder 会自动签名和公证
npx electron-builder --mac
```

**electron-forge 配置（替代方案）：**

```ts
// forge.config.ts
import { VitePlugin } from '@electron-forge/plugin-vite';

export default {
  packagerConfig: {
    name: 'My App',
    appBundleId: 'com.example.myapp',
    icon: 'resources/icon',
    asar: true,
  },
  makers: [
    { name: '@electron-forge/maker-squirrel', config: { name: 'my_app' } },  // Windows
    { name: '@electron-forge/maker-dmg', config: { format: 'ULFO' } },        // macOS
    { name: '@electron-forge/maker-deb', config: { options: { maintainer: 'Me' } } }, // Linux
    { name: '@electron-forge/maker-appx' },                                     // Windows Store
  ],
  plugins: [new VitePlugin()],
};
```

### 调试与性能分析

**开发调试：**

```ts
// 主进程调试
// 方法1：VS Code launch.json
// 方法2：命令行
// electron --inspect=5858 .

// 主进程日志
import log from 'electron-log';
log.info('应用启动');
log.error('出错了', error);

// 渲染进程调试
mainWindow.webContents.openDevTools();
```

**性能分析：**

```ts
// 渲染进程性能
mainWindow.webContents.session.setPreloads([
  join(__dirname, '../preload/perf.js'),
]);

// CPU Profile
mainWindow.webContents.cpuProfiler.start();
// ... 操作 ...
const profile = await mainWindow.webContents.cpuProfiler.stop();
fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));

// 内存快照
const heap = await mainWindow.webContents.takeHeapSnapshot();
```

**启动速度优化：**

```ts
// src/main/index.ts
import { app } from 'electron';

// 1. 延迟加载非必要模块
app.whenReady().then(async () => {
  // 2. 首窗口尽快显示
  const win = createWindow('main', { show: false });
  win.once('ready-to-show', () => win.show());

  // 3. 非关键初始化延后
  setImmediate(() => {
    setupAutoUpdater(win);
    createTray(win);
    createAppMenu();
  });
});

// 4. 关闭不需要的 Chromium 特性
app.commandLine.appendSwitch('disable-features', 'HardwareMediaKeyHandling');
app.commandLine.appendSwitch('disable-background-timer-throttling');
```

**内存优化：**

```ts
// 及时销毁窗口
win.on('closed', () => {
  win.removeAllListeners();
  win = null;
});

// 限制渲染进程缓存
session.defaultSession.clearCache();

// 检查内存泄漏
setInterval(() => {
  const mem = process.memoryUsage();
  log.info(`RSS: ${(mem.rss / 1024 / 1024).toFixed(1)}MB, Heap: ${(mem.heapUsed / 1024 / 1024).toFixed(1)}MB`);
}, 60000);
```

### Deep Link 与协议注册

```ts
// 注册自定义协议
app.setAsDefaultProtocolClient('myapp');

// macOS 处理 URL 事件
app.on('open-url', (event, url) => {
  event.preventDefault();
  handleDeepLink(url);
});

// Windows 处理命令行参数
const gotTheLock = app.requestSingleInstanceLock();
if (!gotTheLock) {
  app.quit();
} else {
  app.on('second-instance', (_event, argv) => {
    const url = argv.find(arg => arg.startsWith('myapp://'));
    if (url) handleDeepLink(url);
    mainWindow?.show();
  });
}

function handleDeepLink(url: string) {
  // myapp://open?id=123
  const parsed = new URL(url);
  const action = parsed.hostname;
  const params = Object.fromEntries(parsed.searchParams);

  mainWindow?.webContents.send('deep-link', { action, params });
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 包体积过大 | 打包了完整 Chromium | electron-builder 配置压缩/NSIS，或考虑 Tauri |
| 内存占用高 | 每个窗口独立渲染进程 | 减少窗口数，用 BrowserView 复用，及时销毁 |
| 渲染进程被 XSS 攻击 | 开启了 nodeIntegration | 必须关闭 nodeIntegration + 开启 contextIsolation |
| contextBridge 传函数失败 | 序列化限制 | 只暴露函数引用，实际逻辑在 preload 中执行 |
| 原生模块编译失败 | node-gyp 依赖问题 | 用 electron-rebuild 重新编译 |
| macOS 签名公证失败 | 缺少 Apple 开发者证书 | 配置 CSC_LINK/CSC_KEY_PASSWORD，用 notarize |
| 窗口闪烁白屏 | 加载时背景默认白色 | `show: false` + `backgroundColor` + `ready-to-show` |
| 启动慢 | Chromium 初始化开销 | 延迟创建窗口，`app.whenReady()` 优化加载顺序 |
| 多窗口通信复杂 | 各渲染进程独立 | 主进程做消息中转，或用 MessagePort |
| 渲染进程崩溃 | 内存泄漏/JS 错误 | `webContents.on('crashed')` 监听，自动重启 |
| 文件监听失效 | asar 打包后路径变化 | `process.resourcesPath` 获取资源路径 |
| 中文路径乱码 | Windows 编码问题 | 用 `path.normalize` 处理，避免 GBK 路径 |
| macOS Dock 点击无反应 | 窗口已隐藏 | 监听 `activate` 事件重新显示窗口 |
| 打包后白屏 | 路径引用错误 | 使用 `__dirname` + `path.join` 拼接路径 |
| asar 内文件无法读取 | 某些原生模块不支持 asar | `unpackDirName` 排除，或用 `process.noAsar` |

### 最佳实践

- **安全第一**：永远关闭 `nodeIntegration`，开启 `contextIsolation` + `sandbox`
- **IPC 优先用 invoke/handle**：双向通信用 invoke，比 send/on 更安全（有返回值、错误处理）
- **Preload 最小暴露**：只暴露渲染进程真正需要的 API，不暴露整个 ipcRenderer
- **主进程做重活**：文件 I/O、加密、数据库操作放主进程，渲染进程只管 UI
- **窗口生命周期管理**：`show: false` + `ready-to-show` 避免白屏，关闭时 `win = null` 释放内存
- **打包优化**：asar 打包、tree-shaking、按平台打包、排除 devDependencies
- **自动更新**：electron-updater + GitHub Releases，启动时检查 + 定时轮询
- **错误监控**：`webContents.on('crashed')` + `process.on('uncaughtException')` 全局捕获
- **单实例锁**：`app.requestSingleInstanceLock()` 防止多开
- **日志记录**：electron-log 统一管理主进程和渲染进程日志

## 面试题

**Q1: Electron 的主进程和渲染进程有什么区别？它们如何通信？**
> 主进程运行 Node.js，负责创建窗口（BrowserWindow）、管理系统资源和原生交互，每个应用只有一个。渲染进程运行 Chromium，负责页面渲染和 UI 交互，可以有多个（多窗口）。通信通过 IPC：`ipcMain.handle` / `ipcRenderer.invoke` 做双向通信，`ipcRenderer.send` / `ipcMain.on` 做单向推送，`MessagePort` 做流式双向通信。

**Q2: Electron 中 contextIsolation 和 nodeIntegration 为什么重要？**
> `nodeIntegration: false` 禁止渲染进程直接使用 Node.js API，防止 XSS 攻击获得文件系统等能力；`contextIsolation: true` 让 preload 脚本和渲染页面的 JS 运行在隔离的 V8 上下文中，防止页面脚本篡改 preload 暴露的 API。两者结合是 Electron 最重要的安全防线，关闭任一项都会导致严重安全漏洞。

**Q3: Electron 和 Tauri 的核心区别是什么？**
> Electron 打包 Chromium + Node.js，体积大（~100MB+）但兼容性好、生态成熟；Tauri 用系统自带的 WebView + Rust 后端，体积小（~5MB）性能好但 WebView 兼容性依赖系统、Rust 学习成本高。选 Electron：需要最强兼容性和成熟生态；选 Tauri：追求小体积、低内存和安全性。

**Q4: 如何优化 Electron 应用的启动速度和内存占用？**
> 启动优化：`show: false` 创建窗口等 `ready-to-show` 再显示、延迟创建非首屏窗口、非关键模块 `setImmediate` 延后初始化、关闭不需要的 Chromium 特性。内存优化：减少窗口数量、及时销毁关闭的窗口（`win = null`）、用 BrowserView 替代多窗口、避免渲染进程缓存大量数据、定期 `clearCache`。

**Q5: Electron 的 Preload 脚本作用是什么？为什么不直接在渲染进程中使用 Node.js？**
> Preload 在渲染进程的页面加载前执行，通过 `contextBridge.exposeInMainWorld` 安全地向渲染进程暴露有限 API。不在渲染进程直接使用 Node.js 是因为：渲染进程加载不可信的网页内容，如果直接拥有 Node.js 能力，XSS 攻击就能读写文件、执行系统命令。Preload 作为中间层，只暴露必要的、经过验证的接口，实现最小权限原则。

**Q6: Electron 中 invoke/handle 和 send/on 两种 IPC 方式有什么区别？**
> `invoke/handle` 是双向请求-响应模式，返回 Promise，支持错误传播（handler 抛出的异常会传到渲染进程），适合需要返回值的操作（如读取文件）。`send/on` 是单向事件模式，无返回值，适合通知类场景（如显示通知、状态推送）。推荐：需要返回值用 invoke，纯事件通知用 send + webContents.send。

**Q7: Electron 打包后的应用如何实现自动更新？**
> 使用 `electron-updater` 库。流程：① 在 `electron-builder.yml` 配置 `publish` 指向更新源（GitHub Releases/自定义服务器）；② 主进程启动时调用 `autoUpdater.checkForUpdates()`；③ 检测到新版本后 `autoUpdater.downloadUpdate()` 下载；④ 下载完成后通知用户，调用 `autoUpdater.quitAndInstall()` 重启安装。支持全量更新和差量更新（Windows NSIS）。

**Q8: 如何处理 Electron 应用的多窗口架构？**
> 用 Map 管理多个 BrowserWindow 实例，每个窗口有唯一标识。窗口间通信通过主进程中转：A 窗口 → IPC → 主进程 → IPC → B 窗口。也可用 `MessagePort` 建立直接通道。注意：每个窗口独立渲染进程，要及时销毁（`win.close()` + `win = null`），避免内存泄漏。模态窗口通过 `parent` 和 `modal` 属性实现。

---

**相关链接：**
- [[Vue核心]]
- [[Webpack与Vite]]
- [[Web安全XSS与CSRF]]
- [[浏览器渲染原理]]
- [[Cookie与认证]]

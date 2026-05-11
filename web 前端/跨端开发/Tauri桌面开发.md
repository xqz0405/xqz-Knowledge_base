---
tags:
  - Web前端
  - Tauri
  - 桌面开发
  - Rust
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Tauri 桌面开发

## What — 什么是 Tauri

Tauri 是一个用于构建跨平台桌面（及移动端）应用的框架，核心思想是 **Rust 后端 + Web 前端**，利用操作系统的原生 WebView 渲染界面，而非打包 Chromium。

### 核心架构

```
┌─────────────────────────────────────────┐
│              Tauri Application          │
│                                         │
│  ┌───────────────┐   ┌───────────────┐  │
│  │  Web Frontend │   │  Rust Core    │  │
│  │  (HTML/CSS/JS)│   │  (业务逻辑)    │  │
│  │               │   │               │  │
│  │  React/Vue/   │   │  文件系统      │  │
│  │  Svelte/Vanilla│  │  系统API      │  │
│  └───────┬───────┘   └───────┬───────┘  │
│          │    IPC (invoke)   │          │
│          └───────────────────┘          │
│                  │                      │
│         ┌────────▼────────┐             │
│         │   System WebView│             │
│         │   (系统原生)     │             │
│         └─────────────────┘             │
└─────────────────────────────────────────┘
```

### 核心组成

| 组件 | 说明 |
|------|------|
| **Rust Core** | 后端逻辑、系统调用、安全策略，编译为原生二进制 |
| **WebView** | 使用操作系统自带 WebView（Windows: WebView2, macOS: WKWebView, Linux: WebKitGTK） |
| **IPC 通信** | 前端通过 `invoke` 调用 Rust Command，Rust 通过 `emit` 推送事件到前端 |

### 关键特性

- **极小体积**：打包后通常 2-10 MB（Electron 通常 100+ MB）
- **低内存占用**：复用系统 WebView，不额外打包 Chromium
- **Rust 安全性**：内存安全、线程安全、零成本抽象
- **Tauri 2.0**：统一桌面端与移动端（iOS/Android），共享 Rust 核心逻辑

---

## Why — 为什么选择 Tauri

### 框架对比

| 特性 | Tauri 2.0 | Electron | NW.js | Qt |
|------|-----------|----------|-------|----|
| **后端语言** | Rust | Node.js | Node.js | C++ |
| **渲染引擎** | 系统 WebView | 内置 Chromium | 内置 Chromium | Qt 自绘 |
| **打包体积** | 2-10 MB | 100-200 MB | 80-150 MB | 30-80 MB |
| **内存占用** | 较低 | 较高 | 较高 | 中等 |
| **启动速度** | 快 | 较慢 | 较慢 | 快 |
| **前端框架** | 任意 | 任意 | 任意 | QML/Qt Quick |
| **移动端支持** | iOS/Android（2.0） | Capacitor | 无 | 有 |
| **自动更新** | 内置插件 | 内置 | 需自建 | 需自建 |
| **系统API** | 插件化 | 完整 | 完整 | 原生 |
| **安全模型** | 权限白名单+Scope | 较弱（全访问） | 较弱 | 中等 |
| **跨平台** | Win/Mac/Linux/Mobile | Win/Mac/Linux | Win/Mac/Linux | 全平台 |
| **学习曲线** | 需学 Rust 基础 | 前端友好 | 前端友好 | 需学 C++/QML |
| **生态成熟度** | 快速增长 | 非常成熟 | 一般 | 成熟 |

### 适用场景

- 需要轻量安装包的工具类应用（编辑器、CLI GUI、系统工具）
- 对性能和内存敏感的应用
- 需要移动端+桌面端共享逻辑的项目
- 安全性要求较高的应用（权限控制、CSP）
- 已有 Web 前端，希望快速封装为原生应用

### 不适用场景

- 需要精确控制渲染引擎版本（依赖系统 WebView 版本）
- 团队完全无 Rust 经验且项目时间紧迫
- 需要大量 npm 原生模块（Node.js 生态依赖）

---

## How — 如何使用 Tauri

### 1. 项目初始化

```bash
# 使用 create-tauri-app 脚手架
npm create tauri-app@latest

# 交互式选择：
# - 应用名称
# - 前端语言（TS/JS）
# - 前端框架（React/Vue/Svelte/Vanilla 等）
# - 包管理器（npm/pnpm/yarn/bun）

# 或在已有前端项目中添加 Tauri
cd existing-frontend-project
npm install -D @tauri-apps/cli@latest
npx tauri init
```

开发命令：

```bash
# 启动开发服务器（热更新）
npm run tauri dev

# 构建生产版本
npm run tauri build
```

### 2. 项目结构

```
my-tauri-app/
├── src/                      # 前端源码
│   ├── App.tsx
│   ├── main.tsx
│   └── ...
├── src-tauri/                # Rust 后端
│   ├── Cargo.toml            # Rust 依赖配置
│   ├── tauri.conf.json       # Tauri 核心配置
│   ├── capabilities/         # 权限配置（Tauri 2.0）
│   │   └── default.json
│   ├── icons/                # 应用图标
│   ├── src/
│   │   ├── main.rs           # 入口文件
│   │   └── lib.rs            # 库入口（Tauri 2.0）
│   └── build.rs              # 构建脚本
├── package.json
└── ...
```

**tauri.conf.json 核心配置：**

```json
{
  "productName": "my-app",
  "version": "1.0.0",
  "identifier": "com.example.my-app",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:1420",
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build"
  },
  "app": {
    "windows": [
      {
        "title": "My App",
        "width": 800,
        "height": 600,
        "resizable": true,
        "fullscreen": false,
        "decorations": true
      }
    ],
    "security": {
      "csp": "default-src 'self'; script-src 'self'"
    }
  }
}
```

**Cargo.toml 关键依赖：**

```toml
[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-shell = "2"
tauri-plugin-dialog = "2"
tauri-plugin-fs = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"

[build-dependencies]
tauri-build = { version = "2", features = [] }
```

### 3. Tauri Commands（IPC 通信）

Tauri Commands 是前后端通信的核心机制。前端通过 `invoke` 调用 Rust 函数。

**Rust 端定义 Command：**

```rust
// src-tauri/src/lib.rs
use tauri::command;
use serde::{Deserialize, Serialize};

// 基本 Command
#[command]
fn greet(name: &str) -> String {
    format!("Hello, {}! Welcome to Tauri.", name)
}

// 返回复杂结构体
#[derive(Serialize, Deserialize, Debug)]
struct UserInfo {
    id: u32,
    name: String,
    email: String,
}

#[command]
fn get_user(id: u32) -> Result<UserInfo, String> {
    if id == 0 {
        Err("Invalid user id".into())
    } else {
        Ok(UserInfo {
            id,
            name: "张三".into(),
            email: "zhangsan@example.com".into(),
        })
    }
}

// 异步 Command
#[command]
async fn fetch_data(url: String) -> Result<String, String> {
    // Rust 异步操作
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    Ok(format!("Fetched from {}", url))
}

// 访问 App 状态
#[command]
fn read_counter(state: tauri::State<'_, Counter>) -> i32 {
    state.0.load(std::sync::atomic::Ordering::SeqCst)
}

use std::sync::atomic::AtomicI32;
struct Counter(AtomicI32);

// 注册所有 Command
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .manage(Counter(AtomicI32::new(0)))
        .invoke_handler(tauri::generate_handler![
            greet,
            get_user,
            fetch_data,
            read_counter,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

**前端调用 Command：**

```typescript
// src/lib/api.ts
import { invoke } from '@tauri-apps/api/core';

// 基本调用
async function greet(name: string): Promise<string> {
  return await invoke<string>('greet', { name });
}

// 带错误处理
async function getUser(id: number): Promise<UserInfo> {
  try {
    return await invoke<UserInfo>('get_user', { id });
  } catch (error) {
    console.error('获取用户失败:', error);
    throw error;
  }
}

// 异步调用
async function fetchData(url: string): Promise<string> {
  return await invoke<string>('fetch_data', { url });
}
```

**事件通信（Rust 推送到前端）：**

```rust
// Rust 端发送事件
use tauri::Emitter;

#[command]
fn start_task(app: tauri::AppHandle) -> Result<(), String> {
    let handle = app.clone();
    std::thread::spawn(move || {
        for i in 1..=100 {
            handle.emit("progress", i).unwrap();
            std::thread::sleep(std::time::Duration::from_millis(50));
        }
        handle.emit("task-complete", ()).unwrap();
    });
    Ok(())
}
```

```typescript
// 前端监听事件
import { listen } from '@tauri-apps/api/event';

const unlisten = await listen<number>('progress', (event) => {
  console.log(`进度: ${event.payload}%`);
  updateProgressBar(event.payload);
});

await listen('task-complete', () => {
  console.log('任务完成!');
});

// 清理监听
unlisten();
```

### 4. 窗口管理

```typescript
// src/lib/window.ts
import { Window } from '@tauri-apps/api/window';
import { getCurrent } from '@tauri-apps/api/window';

// 获取当前窗口
const appWindow = getCurrent();

// 窗口操作
await appWindow.minimize();
await appWindow.maximize();
await appWindow.toggleMaximize();
await appWindow.close();
await appWindow.setFocus();

// 设置窗口属性
await appWindow.setTitle('新标题');
await appWindow.setSize(new LogicalSize(1024, 768));
await appWindow.setPosition(new LogicalPosition(100, 100));
await appWindow.setFullscreen(true);
await appWindow.setAlwaysOnTop(true);

// 监听窗口事件
appWindow.onResized(({ payload: size }) => {
  console.log(`窗口大小: ${size.width} x ${size.height}`);
});

appWindow.onCloseRequested(async (event) => {
  const confirmed = await ask('确定要关闭吗？', {
    title: '确认',
    kind: 'warning',
  });
  if (!confirmed) {
    event.preventDefault();
  }
});
```

**创建多窗口：**

```rust
// Rust 端创建窗口
use tauri::{Manager, WebviewWindowBuilder, WebviewUrl};

#[command]
fn open_settings(app: tauri::AppHandle) -> Result<(), String> {
    if let Some(window) = app.get_webview_window("settings") {
        window.set_focus().map_err(|e| e.to_string())?;
        return Ok(());
    }

    WebviewWindowBuilder::new(
        &app,
        "settings",
        WebviewUrl::App("settings.html".into()),
    )
    .title("设置")
    .inner_size(600.0, 400.0)
    .center()
    .build()
    .map_err(|e| e.to_string())?;

    Ok(())
}
```

```typescript
// 前端创建窗口
import { WebviewWindow } from '@tauri-apps/api/webviewWindow';

const settingsWin = new WebviewWindow('settings', {
  url: 'settings.html',
  title: '设置',
  width: 600,
  height: 400,
  center: true,
});

settingsWin.once('tauri://created', () => {
  console.log('设置窗口已创建');
});

settingsWin.once('tauri://error', (e) => {
  console.error('创建窗口失败:', e);
});
```

### 5. 文件系统

```typescript
// src/lib/files.ts
import {
  readTextFile,
  writeTextFile,
  readDir,
  createDir,
  removeDir,
  renameFile,
  exists,
  BaseDirectory,
} from '@tauri-apps/plugin-fs';

// 读取文本文件
async function loadConfig(): Promise<string> {
  return await readTextFile('config.json', {
    baseDir: BaseDirectory.AppConfig,
  });
}

// 写入文件
async function saveConfig(content: string): Promise<void> {
  await writeTextFile('config.json', content, {
    baseDir: BaseDirectory.AppConfig,
  });
}

// 列出目录内容
async function listFiles(path: string): Promise<string[]> {
  const entries = await readDir(path, {
    baseDir: BaseDirectory.AppData,
  });
  return entries
    .filter((e) => !e.isDirectory)
    .map((e) => e.name);
}

// 检查文件是否存在
async function configExists(): Promise<boolean> {
  return await exists('config.json', {
    baseDir: BaseDirectory.AppConfig,
  });
}
```

**文件对话框：**

```typescript
import { open, save, ask, message } from '@tauri-apps/plugin-dialog';

// 打开文件选择
const selected = await open({
  multiple: false,
  directory: false,
  filters: [
    { name: '图片', extensions: ['png', 'jpg', 'gif'] },
    { name: '所有文件', extensions: ['*'] },
  ],
});
if (selected) {
  console.log('选择了:', selected);
}

// 保存文件对话框
const savePath = await save({
  filters: [{ name: '文本', extensions: ['txt'] }],
});

// 确认对话框
const yes = await ask('是否覆盖已有文件？', {
  title: '确认覆盖',
  kind: 'warning',
  okLabel: '覆盖',
  cancelLabel: '取消',
});
```

### 6. 系统托盘

```rust
// src-tauri/src/lib.rs
use tauri::{
    menu::{MenuBuilder, MenuItemBuilder},
    tray::{MouseButton, MouseButtonState, TrayIconBuilder, TrayIconEvent},
    Manager,
};

pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            let quit = MenuItemBuilder::with_id("quit", "退出").build(app)?;
            let show = MenuItemBuilder::with_id("show", "显示窗口").build(app)?;

            let menu = MenuBuilder::new(app)
                .item(&show)
                .separator()
                .item(&quit)
                .build()?;

            let _tray = TrayIconBuilder::new()
                .icon(app.default_window_icon().unwrap().clone())
                .menu(&menu)
                .menu_on_left_click(false)
                .on_menu_event(|app, event| {
                    match event.id().as_ref() {
                        "quit" => app.exit(0),
                        "show" => {
                            if let Some(w) = app.get_webview_window("main") {
                                let _ = w.show();
                                let _ = w.set_focus();
                            }
                        }
                        _ => {}
                    }
                })
                .on_tray_icon_event(|tray, event| {
                    if let TrayIconEvent::Click {
                        button: MouseButton::Left,
                        button_state: MouseButtonState::Up,
                        ..
                    } = event
                    {
                        let app = tray.app_handle();
                        if let Some(w) = app.get_webview_window("main") {
                            let _ = w.show();
                            let _ = w.set_focus();
                        }
                    }
                })
                .build(app)?;

            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### 7. 菜单与快捷键

```rust
use tauri::{
    menu::{MenuBuilder, MenuItemBuilder, PredefinedMenuItem, SubmenuBuilder},
    keyboard::{KeyCode, Modifiers},
    Accelerator,
};

pub fn create_menu(app: &tauri::AppHandle) -> Result<tauri::menu::Menu, tauri::Error> {
    let quit = MenuItemBuilder::with_id("quit", "退出")
        .accelerator("CmdOrCtrl+Q")
        .build(app)?;

    let new_file = MenuItemBuilder::with_id("new_file", "新建文件")
        .accelerator("CmdOrCtrl+N")
        .build(app)?;

    let open_file = MenuItemBuilder::with_id("open_file", "打开文件")
        .accelerator("CmdOrCtrl+O")
        .build(app)?;

    let save_file = MenuItemBuilder::with_id("save_file", "保存")
        .accelerator("CmdOrCtrl+S")
        .build(app)?;

    let file_menu = SubmenuBuilder::new(app, "文件")
        .item(&new_file)
        .item(&open_file)
        .separator()
        .item(&save_file)
        .separator()
        .item(&quit)
        .build()?;

    let edit_menu = SubmenuBuilder::new(app, "编辑")
        .undo()
        .redo()
        .separator()
        .cut()
        .copy()
        .paste()
        .separator()
        .select_all()
        .build()?;

    MenuBuilder::new(app)
        .item(&file_menu)
        .item(&edit_menu)
        .build()
}

// 菜单事件处理（在 setup 或 invoke_handler 中）
app.on_menu_event(move |app, event| {
    match event.id().as_ref() {
        "new_file" => { /* 新建文件逻辑 */ }
        "open_file" => { /* 打开文件逻辑 */ }
        "save_file" => { /* 保存文件逻辑 */ }
        "quit" => app.exit(0),
        _ => {}
    }
});
```

**全局快捷键：**

```rust
use tauri::GlobalShortcut;

pub fn register_shortcuts(app: &tauri::AppHandle) -> Result<(), Box<dyn std::error::Error>> {
    app.plugin(tauri_plugin_global_shortcut::Builder::new())?;

    // 在 setup 中注册
    // 使用 on_shortcut 回调处理
    Ok(())
}
```

```typescript
// 前端注册快捷键
import { register, unregister } from '@tauri-apps/plugin-global-shortcut';

await register('CommandOrControl+Shift+I', (event) => {
  if (event.state === 'Pressed') {
    console.log('快捷键被按下');
    toggleDevTools();
  }
});
```

### 8. 自动更新

```rust
// Cargo.toml 添加依赖
// tauri-plugin-updater = "2"

// src-tauri/src/lib.rs
use tauri_plugin_updater::UpdaterExt;

#[command]
async fn check_update(app: tauri::AppHandle) -> Result<Option<String>, String> {
    let updater = app.updater_builder().build().map_err(|e| e.to_string())?;
    let update = updater.check().await.map_err(|e| e.to_string())?;

    match update {
        Some(update) => {
            Ok(Some(format!(
                "新版本 {} 可用，当前版本 {}",
                update.version,
                update.current_version
            )))
        }
        None => Ok(None),
    }
}

#[command]
async fn install_update(app: tauri::AppHandle) -> Result<(), String> {
    let updater = app.updater_builder().build().map_err(|e| e.to_string())?;
    let update = updater.check().await.map_err(|e| e.to_string())?;

    if let Some(update) = update {
        update.download_and_install(
            |chunk_len, content_len| {
                println!("下载进度: {} / {:?}", chunk_len, content_len);
            },
            || {
                println!("下载完成，正在安装...");
            },
        )
        .await
        .map_err(|e| e.to_string())?;
    }

    Ok(())
}
```

```json
// tauri.conf.json 添加更新配置
{
  "plugins": {
    "updater": {
      "endpoints": [
        "https://releases.myapp.com/{{target}}/{{arch}}/{{current_version}}"
      ],
      "pubkey": "PUBLIC_KEY_HERE"
    }
  }
}
```

### 9. Shell 与进程

```typescript
// 执行系统命令
import { Command } from '@tauri-apps/plugin-shell';

// 执行命令并获取输出
async function runCommand() {
  const command = Command.create('git', ['status']);
  const output = await command.execute();

  if (output.code === 0) {
    console.log('stdout:', output.stdout);
  } else {
    console.error('stderr:', output.stderr);
  }
}

// 流式输出（长时间运行命令）
async function streamCommand() {
  const command = Command.create('ping', ['localhost', '-c', '4']);

  command.stdout.on('data', (line) => {
    console.log('输出:', line);
  });

  command.stderr.on('data', (line) => {
    console.error('错误:', line);
  });

  const child = await command.spawn();
  console.log('进程 PID:', child.pid);
}

// 打开 URL
import { open } from '@tauri-apps/plugin-opener';
await open('https://tauri.app');
```

### 10. 安全机制

Tauri 的安全模型是多层防护的，核心原则是 **最小权限**。

#### CSP（内容安全策略）

```json
// tauri.conf.json
{
  "app": {
    "security": {
      "csp": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' asset: https://data:; connect-src 'self' https://api.example.com"
    }
  }
}
```

#### 权限配置（Tauri 2.0 Capabilities）

```json
// src-tauri/capabilities/default.json
{
  "identifier": "default",
  "description": "默认权限",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:default",
    "core:window:allow-close",
    "core:window:allow-set-title",
    "shell:allow-open",
    "dialog:allow-open",
    "dialog:allow-save",
    "dialog:allow-ask",
    "fs:default",
    {
      "identifier": "fs:allow-read-text-file",
      "allow": [
        { "path": "$APPDATA/**" },
        { "path": "$APPCONFIG/**" }
      ]
    },
    {
      "identifier": "fs:allow-write-text-file",
      "allow": [
        { "path": "$APPDATA/**" }
      ]
    }
  ]
}
```

#### 安全要点

| 机制 | 说明 |
|------|------|
| **Allowlist** | 仅声明需要的 API 权限，未声明则不可调用 |
| **Scope** | 限定文件系统等 API 可访问的路径范围 |
| **CSP** | 限制前端可加载的资源来源 |
| **IPC 白名单** | 只有注册的 Command 可被前端调用 |
| **沙箱** | WebView 运行在沙箱环境中 |

### 11. SQLite 插件

```rust
// Cargo.toml
// tauri-plugin-sql = { version = "2", features = ["sqlite"] }
```

```typescript
// src/lib/database.ts
import Database from '@tauri-apps/plugin-sql';

let db: Database | null = null;

async function getDb(): Promise<Database> {
  if (!db) {
    db = await Database.load('sqlite:myapp.db');
  }
  return db;
}

// 初始化表结构
async function initDb(): Promise<void> {
  const db = await getDb();
  await db.execute(`
    CREATE TABLE IF NOT EXISTS notes (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      content TEXT,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
      updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `);
}

// 增删改查
async function createNote(title: string, content: string): Promise<void> {
  const db = await getDb();
  await db.execute('INSERT INTO notes (title, content) VALUES ($1, $2)', [
    title,
    content,
  ]);
}

async function getNotes(): Promise<Note[]> {
  const db = await getDb();
  return await db.select<Note[]>('SELECT * FROM notes ORDER BY updated_at DESC');
}

async function deleteNote(id: number): Promise<void> {
  const db = await getDb();
  await db.execute('DELETE FROM notes WHERE id = $1', [id]);
}
```

### 12. HTTP 客户端插件

```rust
// Cargo.toml
// tauri-plugin-http = "2"
```

```typescript
// src/lib/http.ts
import { fetch, Body, ResponseType } from '@tauri-apps/plugin-http';

// GET 请求
async function getData(url: string): Promise<any> {
  const response = await fetch(url, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer token123',
    },
  });
  return await response.json();
}

// POST 请求
async function postData(url: string, data: object): Promise<any> {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: Body.json(data),
  });
  return await response.json();
}

// 下载文件（流式）
async function downloadFile(url: string, savePath: string): Promise<void> {
  const response = await fetch(url, {
    method: 'GET',
    responseType: ResponseType.Binary,
  });
  const bytes = await response.arrayBuffer();
  await writeBinaryFile(savePath, new Uint8Array(bytes));
}
```

### 13. 构建与打包

```bash
# 构建当前平台安装包
npm run tauri build

# 指定目标
npm run tauri build -- --target x86_64-pc-windows-msvc
npm run tauri build -- --target aarch64-apple-darwin
npm run tauri build -- --target x86_64-unknown-linux-gnu
```

**各平台输出格式：**

| 平台 | 输出格式 | 说明 |
|------|---------|------|
| Windows | `.msi` / `.exe` (NSIS) | 安装程序，可在 tauri.conf.json 中配置 |
| macOS | `.dmg` / `.app` | 应用包和磁盘映像 |
| Linux | `.deb` / `.AppImage` | Debian 包和便携应用 |

```json
// tauri.conf.json 打包配置
{
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": {
      "webviewInstallMode": {
        "type": "downloadBootstrapper"
      },
      "nsis": {
        "installerIcon": "icons/icon.ico",
        "installMode": "currentUser"
      }
    }
  }
}
```

**CI/CD 跨平台构建：**

```yaml
# .github/workflows/build.yml
name: Build
on: push

jobs:
  build:
    strategy:
      matrix:
        include:
          - platform: windows-latest
            target: x86_64-pc-windows-msvc
          - platform: macos-latest
            target: aarch64-apple-darwin
          - platform: ubuntu-22.04
            target: x86_64-unknown-linux-gnu

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}
      - name: Install Linux dependencies
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf
      - run: npm install
      - run: npm run tauri build -- --target ${{ matrix.target }}
```

### 14. Tauri 2.0 移动端支持

Tauri 2.0 统一了桌面端和移动端的架构，共享 Rust 核心代码。

```bash
# 初始化移动端
npm install @tauri-apps/cli@next
npx tauri android init
npx tauri ios init

# 开发
npx tauri android dev
npx tauri ios dev

# 构建
npx tauri android build
npx tauri ios build
```

**移动端特定配置：**

```json
// tauri.conf.json
{
  "app": {
    "windows": [
      {
        "title": "My App",
        "width": 800,
        "height": 600
      }
    ]
  },
  "bundle": {
    "android": {},
    "ios": {}
  }
}
```

**移动端注意事项：**

- iOS 需要 Xcode 和 macOS 开发环境
- Android 需要 Android SDK 和 NDK
- 移动端 WebView 版本：iOS 使用 WKWebView，Android 使用 System WebView
- 部分插件不支持移动端，需查阅文档确认
- 触摸交互需单独处理，与桌面端鼠标事件不同

---

## 常见踩坑

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| Windows 上 WebView2 缺失 | Windows 10 旧版本未预装 WebView2 | 在 tauri.conf.json 配置 `webviewInstallMode` 为自动下载安装 |
| 前端调用 Command 返回 404 | Command 未注册到 `invoke_handler` | 检查 `generate_handler![]` 宏是否包含该函数名 |
| IPC 参数名不匹配 | Rust 惯用 snake_case，JS 惯用 camelCase | Rust 端使用 `#[command(rename_all = "camelCase")]` 或前端传 snake_case |
| 文件系统操作被拒绝 | 未在 Capabilities 中声明 `fs` 权限 | 在 `capabilities/default.json` 中添加 `fs:allow-*` 及对应 scope |
| CSP 阻止外部资源加载 | CSP 策略过于严格 | 在 `tauri.conf.json` 的 `security.csp` 中添加对应域名 |
| Linux 构建缺少依赖 | WebKitGTK 等系统库未安装 | `sudo apt install libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf` |
| 中文路径乱码 | Rust 字符串编码问题 | 确保使用 `String` 而非 `&str` 传递路径，使用 `std::path::PathBuf` |
| 热更新不生效 | `beforeDevCommand` 配置错误 | 确保 devUrl 与前端开发服务器端口一致 |
| 打包体积仍然较大 | 引入了过多 Rust 依赖或前端资源 | 检查 Cargo.toml 依赖，优化前端资源（压缩图片、Tree-shaking） |
| 多窗口通信失败 | 窗口间需要通过事件通信 | 使用 `app.emit()` 广播事件，各窗口用 `listen()` 监听 |
| macOS 上签名问题 | 缺少开发者证书 | 在 CI 中使用 `apple-certificate` 配置或本地配置签名 |
| 移动端插件不兼容 | 部分插件仅支持桌面端 | 查阅插件文档，确认 `android` / `ios` 支持 |

---

## 最佳实践

1. **最小权限原则**：仅在 Capabilities 中声明实际需要的权限，避免使用通配符权限
2. **敏感逻辑放 Rust 层**：加密解密、API 密钥、核心业务逻辑放在 Rust 端，前端只负责 UI
3. **合理使用 Scope**：对文件系统、HTTP 等操作设置路径/域名白名单
4. **错误处理要完整**：Rust Command 使用 `Result<T, String>` 返回错误，前端必须 catch
5. **避免频繁 IPC**：批量传递数据而非逐条调用，减少序列化/反序列化开销
6. **状态管理选型**：简单状态用 `tauri::State`，复杂状态用前端状态管理（Zustand/Pinia）
7. **构建优化**：Rust 侧使用 `release` profile 优化二进制体积，开启 LTO
8. **CI 自动化**：使用 GitHub Actions 实现三平台自动构建，避免本地交叉编译
9. **WebView 兼容性**：避免使用最新 CSS/JS 特性，注意不同平台 WebView 的差异
10. **日志与调试**：Rust 端使用 `log` + `tauri-plugin-log`，前端使用 `console` + 开发者工具

---

## 面试题

### 1. Tauri 与 Electron 的核心区别是什么？各自适用什么场景？

**核心区别：**
- **渲染引擎**：Tauri 使用系统 WebView，Electron 内置 Chromium
- **后端语言**：Tauri 使用 Rust，Electron 使用 Node.js
- **打包体积**：Tauri 通常 2-10 MB，Electron 通常 100+ MB
- **安全模型**：Tauri 基于权限白名单和 Scope，Electron 默认可访问全部 Node.js API
- **内存占用**：Tauri 复用系统 WebView，内存占用更低

**Tauri 适用**：轻量工具、性能敏感应用、安全要求高、需要移动端支持
**Electron 适用**：需要完整 Chromium 特性、大量 npm 原生模块、团队纯前端

### 2. Tauri 的 IPC 通信机制是怎样的？有哪些通信方式？

Tauri 提供两种 IPC 方式：
- **Command（invoke）**：前端主动调用 Rust 函数，类似 RPC。前端用 `invoke('command_name', { args })`，Rust 用 `#[command]` 宏标记函数。支持同步和异步，参数自动序列化/反序列化。
- **Event（emit/listen）**：基于发布-订阅模式。Rust 用 `app.emit("event", payload)` 推送，前端用 `listen("event", callback)` 监听。适合进度推送、状态变更等场景。

选择原则：请求-响应模式用 Command，推送通知模式用 Event。

### 3. Tauri 2.0 的权限系统（Capabilities）如何工作？

Tauri 2.0 引入 Capabilities 机制替代 1.x 的 Allowlist：
- **Capabilities 文件**：`src-tauri/capabilities/` 下定义，声明窗口可使用的权限
- **权限粒度**：从 `core:default` 到具体操作如 `fs:allow-read-text-file`
- **Scope 限定**：对文件系统等 API 可指定允许的路径范围，如 `{ "path": "$APPDATA/**" }`
- **窗口绑定**：`windows` 字段指定该权限应用于哪些窗口
- **默认最小权限**：`core:default` 仅包含基本权限，其余需显式声明

### 4. 如何在 Tauri 中处理文件系统操作？有哪些安全限制？

通过 `tauri-plugin-fs` 插件操作文件系统：
- **API**：`readTextFile`、`writeTextFile`、`readDir`、`createDir`、`exists` 等
- **BaseDirectory**：使用 `$APPDATA`、`$APPCONFIG` 等路径变量代替硬编码路径
- **安全限制**：必须在 Capabilities 中声明 `fs:allow-*` 权限；Scope 限定可访问路径范围；CSP 可能影响文件加载
- **最佳实践**：敏感文件放在 `$APPDATA` 下，配置文件放 `$APPCONFIG`，使用 Scope 限制写入路径

### 5. Tauri 如何实现自动更新？更新流程是怎样的？

通过 `tauri-plugin-updater` 实现：
1. **配置**：在 `tauri.conf.json` 中指定更新服务器 endpoint 和公钥
2. **检查更新**：调用 `updater.check()` 向服务器查询新版本
3. **下载安装**：调用 `update.download_and_install()` 下载并安装
4. **签名验证**：使用公钥验证更新包完整性，防止篡改
5. **更新服务器**：需自建或使用 GitHub Releases，返回 JSON 格式的版本信息

注意：Windows 上安装后可能需要重启应用；macOS 需要 Apple 签名才能静默更新。

### 6. Tauri 应用在 Windows 上提示缺少 WebView2 怎么办？

WebView2 是 Tauri 在 Windows 上的渲染引擎，Windows 10 (1803+) 和 Windows 11 预装，旧版本需额外安装：
- **方案一**：配置 `webviewInstallMode` 为 `downloadBootstrapper`，自动下载安装
- **方案二**：配置为 `embedBootstrapper`，将安装器嵌入应用包
- **方案三**：配置为 `offlineInstaller`，嵌入离线安装包（体积增大约 100MB）
- **方案四**：配置为 `fixedRuntime`，嵌入固定版本 WebView2（体积增加约 150MB）

推荐方案一，体积最小，大多数用户已有 WebView2。

### 7. Tauri 如何实现系统托盘？托盘与窗口如何交互？

使用 `TrayIconBuilder` 创建系统托盘：
- **图标**：设置托盘图标（支持 ico/png）
- **菜单**：通过 `MenuBuilder` 构建右键菜单
- **事件**：`on_menu_event` 处理菜单点击，`on_tray_icon_event` 处理托盘图标点击
- **交互模式**：点击托盘图标显示/隐藏窗口，关闭窗口时隐藏到托盘而非退出
- **生命周期**：在 `setup` 回调中创建托盘，确保 app handle 可用

关键点：关闭窗口时用 `event.preventDefault()` 阻止退出，调用 `window.hide()` 隐藏到托盘。

### 8. Tauri 2.0 如何支持移动端开发？桌面端和移动端代码如何复用？

Tauri 2.0 统一了桌面和移动架构：
- **代码复用**：Rust 核心逻辑（Commands、状态管理、业务逻辑）完全复用，前端代码可复用 UI 组件
- **平台差异**：通过条件编译 `#[cfg(desktop)]` / `#[cfg(mobile)]` 处理平台特有逻辑
- **插件兼容**：部分插件支持全平台（fs、http、dialog），部分仅桌面端（updater、全局快捷键）
- **UI 适配**：前端需做响应式设计，处理触摸交互、安全区域、软键盘等移动端特性
- **构建**：`npx tauri android build` / `npx tauri ios build`，需对应 SDK 环境

复用策略：抽取平台无关的 Rust 模块到 `lib.rs`，平台特有代码分文件组织。

---

## 相关链接

- [Tauri 官方文档](https://v2.tauri.app/)
- [Tauri GitHub 仓库](https://github.com/tauri-apps/tauri)
- [Tauri 2.0 迁移指南](https://v2.tauri.app/start/migrate/)
- [Tauri 插件列表](https://v2.tauri.app/plugin/)
- [Rust 官方网站](https://www.rust-lang.org/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [WebView2 官方文档](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)
- [Tauri Awesome 列表](https://github.com/tauri-apps/awesome-tauri)

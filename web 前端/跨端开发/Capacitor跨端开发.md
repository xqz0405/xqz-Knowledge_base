---
tags:
  - Web前端
  - Capacitor
  - 跨端开发
  - Ionic
  - 移动开发
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Capacitor跨端开发

## What — 是什么

> Capacitor 是 Ionic 团队推出的跨平台运行时（Cross-platform runtime），让 Web 应用（HTML/CSS/JS）可以直接部署到 iOS、Android、Electron 和 PWA 等平台。与 Cordova 的 WebView 嵌套不同，Capacitor 提供了更现代的 Bridge 层和原生 API 访问机制，被 Auth0、BBC、NASA、Amtrak 等组织采用。

**核心概念：**

- **Cross-platform Runtime**：不是一个 UI 框架，而是 Web 应用的跨平台运行时，任何前端框架（React/Vue/Angular/Svelte）都可作为上层
- **Bridge 层**：JS 与原生平台之间的标准化通信层，支持同步和异步调用，统一的消息格式替代 Cordova 的 `exec()` 机制
- **Plugin 系统**：官方提供 30+ 核心插件（Camera、Geolocation、Filesystem、Push Notifications 等），社区 200+ 插件，同时完全兼容 Cordova 插件
- **Native Project**：Capacitor 不生成原生代码，而是管理真实的 Xcode/Android Studio 工程，开发者可随时打开原生工程修改
- **WebView 渲染**：App 内嵌系统 WebView 渲染 Web 内容，原生 Shell 提供导航栏、启动屏、权限等原生体验
- **Capacitor Config**：统一的 `capacitor.config.ts` 配置文件，管理 App ID、显示名、服务器、插件配置等

**核心架构：**

```
┌──────────────────────────────────────────────────┐
│                 Capacitor App                     │
├──────────────┬───────────────────────────────────┤
│              │         Native Shell               │
│   Web App    │  ┌─────────┐  ┌────────────────┐  │
│  (React/Vue  │  │ iOS App │  │ Android App    │  │
│   /Angular/  │  │ (Swift) │  │ (Kotlin/Java)  │  │
│   Svelte)    │  ├─────────┤  ├────────────────┤  │
│              │  │ WKWeb-  │  │ WebView        │  │
│  HTML/CSS/JS │  │ View    │  │ (Chrome-based) │  │
│  ├───────────┤  └────┬────┘  └───────┬────────┘  │
│  │ Plugin JS │       │               │            │
│  │ @capacitor│       │    Bridge     │            │
│  │ /core     │───────┴───────────────┘            │
│  │ @capacitor│       │               │            │
│  │ /xxx      │  ┌────┴────┐   ┌──────┴────────┐  │
│  └───────────┘  │ Plugin  │   │ Plugin        │  │
│                 │ (Swift) │   │ (Kotlin/Java) │  │
├─────────────────┴─────────┴───┴───────────────┤  │
│               Platform APIs                     │
│          iOS (Cocoa) │ Android (SDK)            │
├─────────────────────────────────────────────────┤
│              Electron / PWA (Web)               │
└─────────────────────────────────────────────────┘
```

- 设计理念：Web First — 先写 Web，再部署到原生平台
- 核心模块：WebView 容器 + Bridge 通信层 + Plugin 原生插件 + CLI 工具链
- 数据流：JS 调用 Plugin API → Bridge 序列化消息 → 原生 Plugin 处理 → 回调返回 JS
- 与 Cordova 的关键区别：不生成原生代码、管理真实原生工程、现代 Plugin API、支持 Swift/Kotlin

**Capacitor 与 Cordova 对比：**

| 对比项 | Capacitor | Cordova |
|--------|-----------|---------|
| 原生工程 | 管理真实 Xcode/Android 工程 | 生成平台工程（platforms/） |
| Bridge 通信 | 现代 Promise API，类型安全 | `cordova.exec()` 回调式 |
| 插件开发 | Swift/Kotlin，一等公民 | Objective-C/Java，传统方式 |
| 插件安装 | `npm install`，自动链接 | `cordova plugin add`，需 config.xml |
| Cordova 兼容 | 完全兼容 Cordova 插件 | — |
| 原生修改 | 随时打开 Xcode/AS 修改 | 每次 build 会覆盖修改 |
| Web 调试 | Chrome/Safari DevTools 直连 | 需配置远程调试 |
| 热更新 | 官方支持 Capacitor Live Update | 需第三方（cordova-hot-code-push） |
| 项目状态 | 活跃维护（Ionic 团队） | 维护模式（Apache） |
| 学习曲线 | 低 | 中等 |

**支持平台：**

| 平台 | 包名 | 说明 |
|------|------|------|
| iOS | `@capacitor/ios` | WKWebView，Swift 原生 |
| Android | `@capacitor/android` | System WebView，Kotlin 原生 |
| Web/PWA | `@capacitor/core` 内置 | 浏览器直接运行 |
| Electron | `@capacitor/electron` | 桌面应用（社区维护） |

## Why — 为什么

**适用场景：**

- 已有 Web 应用（React/Vue/Angular），需要快速发布到 App Store 和 Google Play
- 中小型应用，对性能要求不极致，追求开发效率
- 企业内部应用、工具类应用、内容展示类应用
- 已有 Ionic/Cordova 项目，需要迁移到更现代的方案
- 需要同时覆盖 Web + iOS + Android，且团队以 Web 技术为主

**对比同类框架：**

| 维度 | Capacitor | React Native | Flutter | Tauri | Electron |
|------|-----------|-------------|---------|-------|----------|
| UI 技术 | Web (HTML/CSS/JS) | React Native 组件 | Flutter Widget | Web (HTML/CSS/JS) | Web (HTML/CSS/JS) |
| 渲染方式 | WebView | 原生组件 | Skia 自绘 | 系统 WebView | Chromium |
| 移动端 | iOS + Android | iOS + Android | iOS + Android | 仅 Android | 不支持 |
| 桌面端 | Electron 社区支持 | Windows/macOS（0.76+） | Windows/macOS/Linux | Windows/macOS/Linux | Windows/macOS/Linux |
| 语言 | JS/TS | JS/TS | Dart | JS/TS + Rust | JS/TS |
| 包体积 | 中等（15-30MB） | 中等（10-25MB） | 较大（15-40MB） | 极小（3-10MB） | 大（80-150MB） |
| 性能 | 中等（WebView） | 高（原生渲染） | 高（Skia） | 高（原生 WebView） | 中等 |
| 原生访问 | Plugin Bridge | Native Module/Turbo | Platform Channel | Tauri Command | IPC |
| 学习曲线 | 低 | 中 | 中高 | 中 | 低 |

**优缺点：**

- ✅ 优点：
  - Web 优先，前端团队零学习成本上手
  - 管理真实原生工程，可随时添加原生代码
  - 现代插件系统，TypeScript 一等公民
  - 完全兼容 Cordova 插件生态
  - 热更新支持（Capacitor Live Update）
  - 支持 PWA，同一套代码可做 Web 应用
  - Ionic 团队活跃维护，文档完善
  - 原生工程不被覆盖，可深度定制
- ❌ 缺点：
  - WebView 渲染性能不如原生和 Flutter
  - 复杂动画和交互体验受限
  - 部分原生功能需要自己写 Plugin
  - 社区规模不如 React Native 和 Flutter
  - iOS 上 WKWebView 有内存限制
  - 不适合游戏、AR/VR 等高性能场景

## How — 怎么用

### 快速上手

```bash
# 创建项目（以 React + Vite 为例）
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install

# 安装 Capacitor
npm install @capacitor/core @capacitor/cli
npx cap init "My App" "com.example.myapp" --web-dir dist

# 构建 Web 资源
npm run build

# 添加平台
npm install @capacitor/ios @capacitor/android
npx cap add ios
npx cap add android

# 同步 Web 资源到原生工程
npx cap sync

# 打开原生 IDE
npx cap open ios      # 打开 Xcode
npx cap open android   # 打开 Android Studio
```

**项目结构：**

```
my-app/
├── src/                     # Web 前端源码
│   ├── App.tsx
│   ├── main.tsx
│   ├── pages/
│   ├── components/
│   └── utils/
├── ios/                     # iOS 原生工程（Xcode 管理）
│   ├── App/
│   │   ├── App.swift
│   │   └── Info.plist
│   └── App.xcworkspace
├── android/                 # Android 原生工程（AS 管理）
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── java/.../MainActivity.java
│   │   │   └── AndroidManifest.xml
│   │   └── build.gradle
│   └── build.gradle
├── public/                  # Web 静态资源
├── capacitor.config.ts      # Capacitor 配置
├── vite.config.ts
├── package.json
└── tsconfig.json
```

### Capacitor 配置详解

```ts
// capacitor.config.ts
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist',
  server: {
    // 开发时连接本地 dev server（Live Reload）
    url: 'http://192.168.1.100:5173',
    cleartext: true,
  },
  plugins: {
    SplashScreen: {
      launchShowDuration: 2000,
      launchAutoHide: true,
      backgroundColor: '#FFFFFF',
      showSpinner: true,
      spinnerColor: '#007AFF',
      androidScaleType: 'CENTER_CROP',
    },
    StatusBar: {
      style: 'LIGHT',        // LIGHT | DARK
      backgroundColor: '#FFFFFF',
    },
    Keyboard: {
      resize: 'body',        // none | body | ionic
      resizeOnFullScreen: true,
    },
    LocalNotifications: {
      smallIcon: 'ic_stat_icon_config_sample',
      iconColor: '#488AFF',
      sound: 'beep.wav',
    },
    PushNotifications: {
      presentationOptions: ['badge', 'sound', 'alert'],
    },
  },
  ios: {
    contentInset: 'automatic',
    backgroundColor: '#FFFFFF',
    prefersHomeIndicatorAutoHidden: true,
    // 自定义 WKWebView 配置
    scrollEnabled: false,
    // 允许混合内容
    allowsLinkPreview: false,
  },
  android: {
    backgroundColor: '#FFFFFF',
    allowMixedContent: true,
    // 捕获原生返回键
    captureInput: true,
    webContentsDebuggingEnabled: true, // 开发时开启 Chrome 调试
  },
};

export default config;
```

### 常用核心插件

```bash
# 安装常用插件
npm install @capacitor/camera
npm install @capacitor/filesystem
npm install @capacitor/geolocation
npm install @capacitor/push-notifications
npm install @capacitor/local-notifications
npm install @capacitor/share
npm install @capacitor/haptics
npm install @capacitor/network
npm install @capacitor/preferences
npm install @capacitor/app
npm install @capacitor/app-launcher
npm install @capacitor/clipboard
npm install @capacitor/device
npm install @capacitor/toast

npx cap sync
```

### 相机与图片选择

```ts
// utils/camera.ts
import { Camera, CameraResultType, CameraSource, Photo } from '@capacitor/camera';

// 拍照
async function takePhoto(): Promise<Photo> {
  const photo = await Camera.getPhoto({
    quality: 90,
    allowEditing: false,
    resultType: CameraResultType.Uri,
    source: CameraSource.Camera,
    width: 1200,
    height: 1200,
    correctOrientation: true,
    saveToGallery: true,
    presentationStyle: 'fullscreen',
  });
  return photo;
}

// 从相册选择
async function pickImage(): Promise<Photo> {
  const photo = await Camera.getPhoto({
    quality: 90,
    resultType: CameraResultType.Uri,
    source: CameraSource.Photos,
    width: 1200,
    height: 1200,
    correctOrientation: true,
  });
  return photo;
}

// 选择多张图片
async function pickMultipleImages(): Promise<Photo[]> {
  const photos = await Camera.pickImages({
    quality: 90,
    limit: 9,
    presentationStyle: 'fullscreen',
  });
  return photos.photos;
}

// React 组件中使用
import { useState } from 'react';

function PhotoCapture() {
  const [photoUrl, setPhotoUrl] = useState<string | null>(null);

  const handleCapture = async () => {
    try {
      const photo = await takePhoto();
      // photo.webPath 可直接用于 <img src>
      // photo.path 是文件系统路径
      setPhotoUrl(photo.webPath || null);
    } catch (error) {
      console.error('Camera error:', error);
    }
  };

  return (
    <div>
      {photoUrl && <img src={photoUrl} alt="captured" />}
      <button onClick={handleCapture}>拍照</button>
      <button onClick={async () => {
        const photo = await pickImage();
        setPhotoUrl(photo.webPath || null);
      }}>从相册选择</button>
    </div>
  );
}
```

### 文件系统操作

```ts
// utils/filesystem.ts
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';

// 写入文本文件
async function writeFile(path: string, data: string) {
  await Filesystem.writeFile({
    path,
    data,
    directory: Directory.Documents,
    encoding: Encoding.UTF8,
    recursive: true, // 自动创建父目录
  });
}

// 读取文本文件
async function readFile(path: string): Promise<string> {
  const result = await Filesystem.readFile({
    path,
    directory: Directory.Documents,
    encoding: Encoding.UTF8,
  });
  return result.data as string;
}

// 追加内容
async function appendFile(path: string, data: string) {
  await Filesystem.appendFile({
    path,
    data,
    directory: Directory.Documents,
    encoding: Encoding.UTF8,
  });
}

// 删除文件
async function deleteFile(path: string) {
  await Filesystem.deleteFile({
    path,
    directory: Directory.Documents,
  });
}

// 检查文件是否存在
async function fileExists(path: string): Promise<boolean> {
  try {
    const stat = await Filesystem.stat({
      path,
      directory: Directory.Documents,
    });
    return stat.type === 'file';
  } catch {
    return false;
  }
}

// 列出目录内容
async function listDir(path: string) {
  const result = await Filesystem.readdir({
    path,
    directory: Directory.Documents,
  });
  return result.files;
}

// 获取文件 URI（用于分享等）
async function getUri(path: string): Promise<string> {
  const result = await Filesystem.getUri({
    path,
    directory: Directory.Documents,
  });
  return result.uri;
}

// 下载文件并保存到本地
async function downloadAndSave(url: string, filename: string) {
  const response = await fetch(url);
  const blob = await response.blob();
  const base64 = await blobToBase64(blob);

  await Filesystem.writeFile({
    path: filename,
    data: base64,
    directory: Directory.Documents,
    recursive: true,
  });
}

function blobToBase64(blob: Blob): Promise<string> {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onloadend = () => {
      const base64 = (reader.result as string).split(',')[1];
      resolve(base64);
    };
    reader.readAsDataURL(blob);
  });
}
```

**Directory 枚举：**

| 目录 | 说明 | iOS 路径 | Android 路径 |
|------|------|----------|-------------|
| `Documents` | 用户文档 | NSDocumentDirectory | Internal Storage |
| `Data` | 应用数据 | NSLibraryDirectory | Internal Files |
| `Library` | 库目录 | NSLibraryDirectory | — |
| `Cache` | 缓存 | NSCachesDirectory | Cache |
| `External` | 外部存储 | — | External Storage |
| `ExternalStorage` | 外部存储 | — | External Storage |

### 地理位置与地图

```ts
// utils/geolocation.ts
import { Geolocation, Position } from '@capacitor/geolocation';

// 获取当前位置
async function getCurrentPosition(): Promise<Position> {
  const position = await Geolocation.getCurrentPosition({
    enableHighAccuracy: true,
    timeout: 10000,
  });
  return position;
}

// 实时监听位置变化
async function watchPosition(
  callback: (position: Position) => void,
  errorCallback?: (error: any) => void
) {
  const watchId = await Geolocation.watchPosition(
    { enableHighAccuracy: true, timeout: 10000 },
    (position, err) => {
      if (err) {
        errorCallback?.(err);
        return;
      }
      if (position) callback(position);
    }
  );
  return watchId; // 用于停止监听
}

// 停止监听
function clearWatch(watchId: string) {
  Geolocation.clearWatch({ id: watchId });
}

// 检查权限
async function checkLocationPermission() {
  const status = await Geolocation.checkPermissions();
  // status.location: 'prompt' | 'granted' | 'denied'
  return status;
}

// 请求权限
async function requestLocationPermission() {
  const status = await Geolocation.requestPermissions();
  return status;
}

// React Hook 封装
function useGeolocation() {
  const [position, setPosition] = useState<Position | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let watchId: string;

    (async () => {
      try {
        const perm = await Geolocation.requestPermissions();
        if (perm.location === 'denied') {
          setError('位置权限被拒绝');
          return;
        }
        watchId = await watchPosition(setPosition, (err) =>
          setError(err.message)
        );
      } catch (e: any) {
        setError(e.message);
      }
    })();

    return () => {
      if (watchId) clearWatch(watchId);
    };
  }, []);

  return { position, error };
}
```

### 推送通知

```ts
// utils/push.ts
import { PushNotifications } from '@capacitor/push-notifications';
import { LocalNotifications } from '@capacitor/local-notifications';

// 远程推送初始化
async function initPushNotifications() {
  // 请求权限
  let permStatus = await PushNotifications.checkPermissions();

  if (permStatus.receive === 'prompt') {
    permStatus = await PushNotifications.requestPermissions();
  }

  if (permStatus.receive !== 'granted') {
    console.warn('推送权限未授予');
    return;
  }

  // 注册推送
  await PushNotifications.register();

  // 监听注册成功，获取 device token
  PushNotifications.addListener('registration', (token) => {
    console.log('Push registration success, token:', token.value);
    // 将 token 发送到后端
    // await api.post('/devices/register', { token: token.value, platform: 'ios' });
  });

  // 监听注册失败
  PushNotifications.addListener('registrationError', (error) => {
    console.error('Push registration error:', error);
  });

  // 前台收到推送
  PushNotifications.addListener('pushNotificationReceived', (notification) => {
    console.log('Push received in foreground:', notification);
    // 可选：显示本地通知
    showLocalNotification(notification.title || '新消息', notification.body || '');
  });

  // 点击推送打开 App
  PushNotifications.addListener(
    'pushNotificationActionPerformed',
    (action) => {
      console.log('Push action:', action.notification.data);
      const data = action.notification.data;
      // 根据推送数据跳转页面
      if (data.type === 'chat') {
        // navigateTo(`/chat/${data.id}`);
      }
    }
  );
}

// 本地通知
async function showLocalNotification(title: string, body: string) {
  await LocalNotifications.schedule({
    notifications: [
      {
        title,
        body,
        id: Date.now(),
        schedule: { at: new Date(Date.now() + 1000) },
        sound: undefined,
        attachments: undefined,
        actionTypeId: '',
        extra: null,
      },
    ],
  });
}

// 定时通知
async function scheduleReminder(title: string, body: string, at: Date) {
  await LocalNotifications.schedule({
    notifications: [
      {
        title,
        body,
        id: Math.floor(Math.random() * 100000),
        schedule: { at },
        extra: { type: 'reminder' },
      },
    ],
  });
}

// 取消通知
async function cancelNotification(id: number) {
  await LocalNotifications.cancel({ notifications: [{ id }] });
}
```

### 网络状态与偏好存储

```ts
// 网络状态监听
import { Network, ConnectionStatus } from '@capacitor/network';

async function initNetworkListener() {
  // 获取当前状态
  const status = await Network.getStatus();
  console.log('当前网络:', status.connectionType);
  // wifi | cellular | none | unknown | ethernet | bluetooth

  // 监听网络变化
  Network.addListener('networkStatusChange', (status: ConnectionStatus) => {
    if (status.connected) {
      console.log('网络已连接:', status.connectionType);
    } else {
      console.log('网络已断开');
      // 提示用户切换到离线模式
    }
  });
}

// 偏好存储（类似 localStorage，但跨平台统一）
import { Preferences } from '@capacitor/preferences';

async function storageExample() {
  // 写入
  await Preferences.set({ key: 'token', value: 'jwt-xxx' });
  await Preferences.set({ key: 'theme', value: 'dark' });
  await Preferences.set({
    key: 'userInfo',
    value: JSON.stringify({ name: '张三', role: 'admin' }),
  });

  // 读取
  const { value: token } = await Preferences.get({ key: 'token' });
  const { value: userInfoStr } = await Preferences.get({ key: 'userInfo' });
  const userInfo = userInfoStr ? JSON.parse(userInfoStr) : null;

  // 删除
  await Preferences.remove({ key: 'token' });

  // 清空
  await Preferences.clear();

  // 获取所有键
  const { keys } = await Preferences.keys();
}

// 封装类型安全的 Storage
class TypedStorage {
  async get<T>(key: string, defaultValue: T): Promise<T> {
    const { value } = await Preferences.get({ key });
    return value ? JSON.parse(value) : defaultValue;
  }

  async set<T>(key: string, value: T): Promise<void> {
    await Preferences.set({ key, value: JSON.stringify(value) });
  }

  async remove(key: string): Promise<void> {
    await Preferences.remove({ key });
  }
}

const storage = new TypedStorage();
```

### 分享与剪贴板

```ts
import { Share, ShareOptions } from '@capacitor/share';
import { Clipboard } from '@capacitor/clipboard';

// 分享文本/链接/文件
async function shareContent() {
  // 分享文本
  await Share.share({
    title: '查看这篇文章',
    text: '一篇关于 Capacitor 的好文章',
    url: 'https://capacitorjs.com',
    dialogTitle: '分享到',
  });

  // 分享图片
  await Share.share({
    title: '分享图片',
    files: [photoUrl],  // 本地文件路径
    dialogTitle: '分享图片到',
  });
}

// 剪贴板操作
async function clipboardExample() {
  // 写入剪贴板
  await Clipboard.write({
    string: 'https://example.com/invite?code=ABC123',
  });

  // 写入图片（base64）
  await Clipboard.write({
    image: 'data:image/png;base64,iVBOR...',
  });

  // 读取剪贴板
  const { value, type } = await Clipboard.read();
  if (type === 'string') {
    console.log('剪贴板文本:', value);
  }
}
```

### App 生命周期与深度链接

```ts
import { App, AppState } from '@capacitor/app';
import { AppLauncher } from '@capacitor/app-launcher';

// App 状态监听
App.addListener('appStateChange', (state: AppState) => {
  if (state.isActive) {
    console.log('App 进入前台');
  } else {
    console.log('App 进入后台');
    // 保存未提交的数据
  }
});

// 返回按钮（Android）
App.addListener('backButton', ({ canGoBack }) => {
  if (canGoBack) {
    window.history.back();
  } else {
    // 确认退出
    App.exitApp();
  }
});

// 深度链接（Deep Link / URL Scheme）
App.addListener('appUrlOpen', (event) => {
  console.log('Deep link:', event.url);
  // 例如: myapp://product/detail?id=123
  const url = new URL(event.url);
  const path = url.pathname; // /product/detail
  const id = url.searchParams.get('id'); // 123
  // 根据路径导航到对应页面
});

// 通用链接配置（iOS）
// ios/App/App/Entitlements.plist 中配置:
// <key>com.apple.developer.associated-domains</key>
// <array>
//   <string>applinks:example.com</string>
// </array>

// Android App Links 配置
// android/app/src/main/AndroidManifest.xml 中配置:
// <intent-filter android:autoVerify="true">
//   <action android:name="android.intent.action.VIEW" />
//   <category android:name="android.intent.category.DEFAULT" />
//   <category android:name="android.intent.category.BROWSABLE" />
//   <data android:scheme="https" android:host="example.com" />
// </intent-filter>

// 打开其他 App
async function openExternalApp() {
  // 检查是否安装
  const { value } = await AppLauncher.canOpenUrl({ url: 'com.google.maps' });

  if (value) {
    await AppLauncher.openUrl({
      url: 'comgooglemaps://?center=40.765819,-73.975866&zoom=14',
    });
  }
}
```

### 自定义 Plugin 开发

**JS 接口定义：**

```ts
// plugins/echo.ts
import { registerPlugin } from '@capacitor/core';

export interface EchoPlugin {
  echo(options: { value: string }): Promise<{ value: string }>;
  getDeviceModel(): Promise<{ model: string }>;
  vibrate(duration: number): Promise<void>;
}

const Echo = registerPlugin<EchoPlugin>('Echo');

export default Echo;
```

**iOS 原生实现（Swift）：**

```swift
// ios/App/App/EchoPlugin.swift
import Capacitor

@objc(EchoPlugin)
public class EchoPlugin: CAPPlugin {
  private let implementation = Echo()

  @objc func echo(_ call: CAPPluginCall) {
    let value = call.getString("value") ?? ""
    call.resolve(["value": implementation.echo(value)])
  }

  @objc func getDeviceModel(_ call: CAPPluginCall) {
    let model = UIDevice.current.model
    call.resolve(["model": model])
  }

  @objc func vibrate(_ call: CAPPluginCall) {
    let duration = call.getInt("duration") ?? 100
    DispatchQueue.main.async {
      let generator = UIImpactFeedbackGenerator(style: .medium)
      generator.impactOccurred()
    }
    call.resolve()
  }
}

// 注册插件方法
@objc(EchoPlugin)
public class EchoPlugin: CAPPlugin {
  // ... 上面的实现
}

// 在 AppDelegate 中注册
// AppDelegate.swift
import Capacitor

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    return true
  }

  // 注册自定义插件
  func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    return ApplicationDelegateProxy.shared.application(app, open: url, options: options)
  }
}
```

**Android 原生实现（Kotlin）：**

```kotlin
// android/app/src/main/java/com/example/myapp/EchoPlugin.kt
package com.example.myapp

import com.getcapacitor.JSObject
import com.getcapacitor.Plugin
import com.getcapacitor.PluginCall
import com.getcapacitor.PluginMethod
import com.getcapacitor.annotation.CapacitorPlugin
import android.os.Build
import android.os.VibrationEffect
import android.os.Vibrator

@CapacitorPlugin(name = "Echo")
class EchoPlugin : Plugin() {

    @PluginMethod
    fun echo(call: PluginCall) {
        val value = call.getString("value") ?: ""
        val ret = JSObject()
        ret.put("value", value)
        call.resolve(ret)
    }

    @PluginMethod
    fun getDeviceModel(call: PluginCall) {
        val ret = JSObject()
        ret.put("model", Build.MODEL)
        call.resolve(ret)
    }

    @PluginMethod
    fun vibrate(call: PluginCall) {
        val duration = call.getInt("duration") ?: 100
        val vibrator = context.getSystemService(android.content.Context.VIBRATOR_SERVICE) as Vibrator
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            vibrator.vibrate(VibrationEffect.createOneShot(duration.toLong(), VibrationEffect.DEFAULT_AMPLITUDE))
        } else {
            vibrator.vibrate(duration.toLong())
        }
        call.resolve()
    }
}

// 在 MainActivity 中注册
// MainActivity.java
package com.example.myapp;

import android.os.Bundle;
import com.getcapacitor.BridgeActivity;

public class MainActivity extends BridgeActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        registerPlugin(EchoPlugin.class);
        super.onCreate(savedInstanceState);
    }
}
```

**在 Web 端使用：**

```ts
// src/utils/echo.ts
import Echo from '../plugins/echo';

async function useEcho() {
  const result = await Echo.echo({ value: 'Hello Capacitor!' });
  console.log(result.value); // "Hello Capacitor!"

  const device = await Echo.getDeviceModel();
  console.log(device.model); // "iPhone" or "Pixel 7"

  await Echo.vibrate(200);
}
```

### Live Reload 开发调试

```ts
// capacitor.config.ts — 开发时配置
const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist',
  server: {
    // 方法1：指定本地 IP + 端口
    url: 'http://192.168.1.100:5173',
    cleartext: true, // Android 允许 HTTP
  },
};
```

```bash
# 方法2：使用 CLI 自动发现（推荐）
npx cap run ios --livereload --external
npx cap run android --livereload --external

# 指定端口
npx cap run ios --livereload --port 5173 --external
```

**调试技巧：**

| 平台 | 调试方式 | 步骤 |
|------|---------|------|
| iOS | Safari Web Inspector | Safari → 开发 → 模拟器/设备 → 选择 WebView |
| Android | Chrome DevTools | chrome://inspect → 选择设备和 WebView |
| Web | 浏览器 DevTools | 直接 F12 |
| iOS 原生 | Xcode Console | Xcode → 运行 → Console 输出 |
| Android 原生 | Logcat | Android Studio → Logcat 过滤 `Capacitor` |

### 热更新（Capacitor Live Update）

```bash
# 安装 Live Update 插件
npm install @capacitor/live-update
npx cap sync
```

```ts
// capacitor.config.ts
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  // ...
  plugins: {
    LiveUpdate: {
      config: {
        appId: 'your-app-id',       // Appflow 中的应用 ID
        channel: 'production',       // 更新通道
        autoDeleteBundles: true,      // 自动删除旧包
      },
    },
  },
};

// 代码中手动检查更新
import { LiveUpdate } from '@capacitor/live-update';

async function checkForUpdates() {
  try {
    const result = await LiveUpdate.sync();
    if (result.activeBundlePath) {
      console.log('有新版本可用，下次启动生效');
    }
  } catch (error) {
    console.error('检查更新失败:', error);
  }
}
```

### Ionic UI 组件配合使用

```bash
# 安装 Ionic 框架（可选，Capacitor 不强制依赖）
npm install @ionic/react @ionic/react-router react-router-dom
```

```tsx
// 使用 Ionic UI 组件 + Capacitor 原生能力
import {
  IonApp,
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonImg,
  IonToast,
} from '@ionic/react';
import { Camera, CameraResultType } from '@capacitor/camera';
import { Share } from '@capacitor/share';
import { useState } from 'react';

function App() {
  const [photo, setPhoto] = useState<string | null>(null);
  const [showToast, setShowToast] = useState(false);

  const takePhoto = async () => {
    const image = await Camera.getPhoto({
      quality: 90,
      allowEditing: false,
      resultType: CameraResultType.Uri,
    });
    setPhoto(image.webPath || null);
  };

  const sharePhoto = async () => {
    if (photo) {
      await Share.share({
        title: '看看我的照片',
        files: [photo],
        dialogTitle: '分享照片',
      });
    }
  };

  return (
    <IonApp>
      <IonPage>
        <IonHeader>
          <IonToolbar>
            <IonTitle>Capacitor 示例</IonTitle>
          </IonToolbar>
        </IonHeader>
        <IonContent className="ion-padding">
          <IonCard>
            <IonCardHeader>
              <IonCardTitle>相机示例</IonCardTitle>
            </IonCardHeader>
            <IonCardContent>
              {photo && <IonImg src={photo} />}
              <IonButton expand="block" onClick={takePhoto}>
                拍照
              </IonButton>
              {photo && (
                <IonButton expand="block" fill="outline" onClick={sharePhoto}>
                  分享照片
                </IonButton>
              )}
            </IonCardContent>
          </IonCard>

          <IonToast
            isOpen={showToast}
            onDidDismiss={() => setShowToast(false)}
            message="操作成功"
            duration={2000}
          />
        </IonContent>
      </IonPage>
    </IonApp>
  );
}
```

### Cordova 插件迁移

```bash
# 安装 Cordova 插件（Capacitor 直接兼容）
npm install cordova-plugin-device
npm install cordova-plugin-qrscanner
npx cap sync

# 自动迁移 cordova-plugin 中的配置到原生工程
npx cap sync
```

```ts
// 使用 Cordova 插件
declare global {
  interface Window {
    device: any;
    QRScanner: any;
  }
}

// Cordova 插件通过 window 对象访问
document.addEventListener('deviceready', () => {
  console.log(window.device.platform, window.device.version);
}, false);

// 推荐做法：逐步替换为 Capacitor 原生插件
// @capacitor/device 替代 cordova-plugin-device
// @capacitor/barcode-scanner 替代 cordova-plugin-qrscanner
```

**Cordova → Capacitor 插件映射：**

| Cordova 插件 | Capacitor 替代 |
|-------------|---------------|
| `cordova-plugin-camera` | `@capacitor/camera` |
| `cordova-plugin-file` | `@capacitor/filesystem` |
| `cordova-plugin-geolocation` | `@capacitor/geolocation` |
| `cordova-plugin-device` | `@capacitor/device` |
| `cordova-plugin-network-information` | `@capacitor/network` |
| `cordova-plugin-splashscreen` | `@capacitor/splash-screen` |
| `cordova-plugin-statusbar` | `@capacitor/status-bar` |
| `cordova-plugin-local-notification` | `@capacitor/local-notifications` |
| `cordova-plugin-push` | `@capacitor/push-notifications` |
| `cordova-plugin-inappbrowser` | `@capacitor/browser` |

### 发布与打包

**iOS 发布：**

```bash
# 1. 构建 Web 资源
npm run build

# 2. 同步到 iOS 工程
npx cap sync ios

# 3. 打开 Xcode
npx cap open ios

# 4. 在 Xcode 中：
#    - 选择 Team（开发者账号）
#    - 设置 Version 和 Build
#    - Product → Archive → Distribute App
```

**Android 发布：**

```bash
# 1. 构建 + 同步
npm run build && npx cap sync android

# 2. 打开 Android Studio
npx cap open android

# 3. 在 Android Studio 中：
#    - Build → Generate Signed Bundle / APK
#    - 选择 APK 或 AAB（推荐 AAB 上传 Google Play）
#    - 创建或选择 keystore
#    - 选择 release 构建类型
```

**版本号管理：**

```ts
// capacitor.config.ts
const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'My App',
  webDir: 'dist',
};

// 使用 npm version 同步版本号
// package.json 添加 scripts:
// "version:sync": "npx cap copy && npx cap sync"
```

```bash
# 更新版本号（自动修改 package.json）
npm version patch  # 1.0.0 → 1.0.1
npm version minor  # 1.0.0 → 1.1.0
npm version major  # 1.0.0 → 2.0.0

# 同步到原生工程
npx cap sync
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| iOS WKWebView 白屏 | CSP 策略限制或混合内容 | 配置 `server.allowNavigation`，修改 CSP 允许本地资源 |
| Android WebView 不加载 | 明文 HTTP 被拦截 | 配置 `android.allowMixedContent: true`，或使用 HTTPS |
| 插件调用报 not available | Web 端不支持该插件 | 检查 `Capacitor.isNativePlatform()` 后再调用 |
| 相机权限被拒 | 未配置权限描述 | iOS: Info.plist 添加 `NSCameraUsageDescription`；Android: AndroidManifest.xml 添加 `CAMERA` 权限 |
| 热更新不生效 | `webDir` 配置错误 | 确保 `webDir` 指向构建输出目录，执行 `npx cap sync` |
| iOS 状态栏遮挡内容 | 安全区域未处理 | CSS 添加 `padding-top: env(safe-area-inset-top)` |
| Android 返回键直接退出 | 未处理返回逻辑 | 监听 `App.addListener('backButton')` 处理导航 |
| 深度链接不生效 | 配置缺失 | iOS: 配置 Associated Domains；Android: 配置 App Links + assetlinks.json |
| 相机图片显示不出来 | 跨域问题 | 使用 `CameraResultType.Uri` 或将图片转 base64 |
| `npx cap add ios` 报错 | Xcode CLI Tools 未安装 | 运行 `xcode-select --install` |
| 启动屏闪白 | 启动屏配置不当 | 配置 `SplashScreen.launchShowDuration` + 背景色与 Web 一致 |
| 文件路径在 Web 端无效 | Web 没有 Filesystem | 使用 `Capacitor.convertFileSrc()` 将原生路径转为 Web 可访问的 URL |

### 最佳实践

- 使用 `Capacitor.isNativePlatform()` 判断运行环境，Web 端提供降级方案
- 相机/文件等涉及权限的功能，先检查权限再调用，避免直接崩溃
- iOS 安全区域用 `env(safe-area-inset-*)` CSS 变量处理
- 开发时配置 `server.url` 连接 dev server，发布时删除此配置
- 自定义 Plugin 时 JS 端用 TypeScript 定义接口，确保类型安全
- Cordova 插件优先替换为 Capacitor 原生插件，性能和类型更好
- 图片等大文件使用 `CameraResultType.Uri` 而非 Base64，避免内存溢出
- 使用 `@capacitor/preferences` 替代 `localStorage`，后者在 WebView 清理时可能丢失
- 原生工程修改后不要手动 build，用 `npx cap sync` 保持同步
- 深度链接和通用链接需要后端配合配置 apple-app-site-association 和 assetlinks.json

## 面试题

**Q1: Capacitor 和 Cordova 的核心区别是什么？为什么要从 Cordova 迁移？**
> 核心区别：① Capacitor 管理真实原生工程，Cordova 生成平台工程；② Capacitor 用现代 Promise/TS 插件 API，Cordova 用回调式 `exec()`；③ Capacitor 的原生工程不被 build 覆盖，Cordova 每次 build 会覆盖修改；④ Capacitor 原生支持 Swift/Kotlin。迁移原因：Cordova 已进入维护模式，Capacitor 由 Ionic 团队活跃维护；插件开发体验更好；兼容 Cordova 插件可渐进迁移。

**Q2: Capacitor 的 Bridge 通信机制是怎样的？和 React Native 的 Bridge 有什么区别？**
> Capacitor Bridge：JS 调用 `Capacitor.Plugins.Xxx.method()` → 消息序列化为 JSON → 通过 WKWebView 的 `postMessage` 或 Android 的 `evaluateJavascript` 传递到原生层 → 原生 Plugin 处理 → 结果回传 JS。与 RN Bridge 的区别：① Capacitor 是 WebView 内嵌通信，RN 是独立 JS 引擎与原生通信；② Capacitor 的 Bridge 主要是异步的（同步场景有限），RN 新架构通过 JSI 实现同步调用；③ Capacitor 通信开销小（同进程），RN 旧架构 Bridge 有序列化瓶颈。

**Q3: Capacitor 中如何实现深度链接（Deep Link）？**
> 两种方式：① URL Scheme（如 `myapp://product/123`）：iOS 在 Info.plist 配置 `CFBundleURLSchemes`，Android 在 AndroidManifest.xml 配置 `<intent-filter>` 的 `<data android:scheme>`。② 通用链接/应用链接（如 `https://example.com/product/123`）：iOS 配置 Associated Domains + 服务器放置 `apple-app-site-association` 文件；Android 配置 App Links + 服务器放置 `assetlinks.json`。代码中通过 `App.addListener('appUrlOpen')` 监听并解析跳转。

**Q4: Capacitor 的 WebView 渲染有什么性能限制？如何优化？**
> 限制：① JS 执行速度不如原生（JIT 编译 vs AOT）；② DOM 渲染受 WebView 引擎限制，复杂列表滚动有卡顿；③ 动画帧率不如原生（CSS 动画 vs 原生动画驱动）；④ iOS WKWebView 有内存上限（约 1.5GB），大图片可能崩溃。优化：① 长列表使用虚拟滚动（react-window/virtuoso）；② 动画用 `transform/opacity` 触发 GPU 合成；③ 图片压缩 + 懒加载 + 使用 `Uri` 而非 Base64；④ 限制 DOM 节点数量；⑤ 高性能场景考虑用原生 Plugin 实现。

**Q5: Capacitor 自定义 Plugin 的开发流程是什么？**
> 四步：① JS 层用 `registerPlugin<T>()` 定义接口（TypeScript 类型约束）；② iOS 层用 Swift 继承 `CAPPlugin`，用 `@objc` 标注方法，通过 `CAPPluginCall` 接收参数和返回结果；③ Android 层用 Kotlin 继承 `Plugin`，用 `@CapacitorPlugin` + `@PluginMethod` 标注，通过 `PluginCall` 交互；④ 在 `AppDelegate`（iOS）或 `MainActivity`（Android）中注册插件。关键点：JS 接口是统一的，各平台分别实现，Web 端可以提供 mock 降级。

**Q6: Capacitor 中如何处理 iOS 安全区域（Safe Area）？**
> iOS 刘海屏和底部横条会遮挡内容，需用 CSS 环境变量处理：`padding-top: env(safe-area-inset-top)` 处理顶部状态栏；`padding-bottom: env(safe-area-inset-bottom)` 处理底部横条；`padding-left/right: env(safe-area-inset-left/right)` 处理横屏左右安全区。同时需在 viewport meta 中添加 `viewport-fit=cover`。Capacitor 的 StatusBar 插件可控制状态栏样式（LIGHT/DARK），配合 `contentInset` 配置调整 WebView 内边距。

**Q7: Capacitor 项目如何实现热更新？有哪些方案？**
> 三种方案：① 官方 Live Update 插件（`@capacitor/live-update`）：通过 Appflow 服务分发 Web 资源包，App 启动时检查更新并下载，下次启动生效，支持灰度发布和回滚；② 自建方案：将 Web 资源上传到 CDN，App 启动时请求版本清单，下载差异包后替换本地文件，重启 WebView 加载；③ CodePush 方案：利用微软 App Center 的 CodePush 服务（需适配）。注意：热更新只能更新 Web 资源，原生代码变更必须通过应用商店发版。

**Q8: Capacitor 项目中 Web 调试和原生调试分别怎么做？**
> Web 调试：iOS 用 Safari → 开发 → 选择模拟器/设备 → 选择 WebView 打开 Web Inspector；Android 开启 `webContentsDebuggingEnabled: true` 后用 Chrome 访问 `chrome://inspect` 连接 WebView。原生调试：iOS 用 Xcode 运行项目查看 Console 和断点调试；Android 用 Android Studio 的 Logcat 过滤 `Capacitor` 关键字，或在原生代码中打断点。混合调试时可同时打开 Web Inspector 和原生 IDE Console，排查 Bridge 通信问题。

---

**相关链接：**
- [[Electron桌面开发]]
- [[Tauri桌面开发]]
- [[React Native移动开发]]
- [[uni-app跨端开发]]
- [[小程序开发]]
- [[Web安全XSS与CSRF]]
- [[Service Worker与PWA]]

---
tags:
  - Web前端
  - ServiceWorker
  - PWA
  - 离线
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Service Worker与PWA

## What — 是什么

> Service Worker 是浏览器在后台独立线程运行的脚本，可拦截网络请求、管理缓存、接收推送通知，是 PWA（渐进式 Web 应用）的核心技术。

**核心概念：**

- **生命周期**：`install` → `activated` → `fetch` 事件拦截，`terminate`（空闲时终止以节省内存）
- **缓存策略**：Cache-First、Network-First、Stale-While-Revalidate、Cache-Only、Network-Only
- **PWA 三要素**：HTTPS + Service Worker + Web App Manifest
- **Manifest**：`manifest.json` 定义应用名、图标、启动 URL、显示模式

**关键特性：**

- Service Worker 线程无法访问 DOM，通过 `postMessage` 与主线程通信
- 作用域限于注册路径及其子路径
- 必须在 HTTPS 下运行（localhost 除外）
- 一旦激活，页面刷新后仍然控制页面

## Why — 为什么

**适用场景：**

- 离线可用（文档编辑器、阅读器）
- 秒开体验（App Shell 缓存）
- 推送通知（消息提醒）
- 后台同步（断网时排队，联网后发送）

**对比替代方案：**

| 维度 | Service Worker | AppCache（已废弃） | localStorage 缓存 |
|------|---------------|-------------------|-------------------|
| 缓存粒度 | 请求级（可编程） | 文件级（声明式） | 字符串（手动序列化） |
| 离线能力 | 完整（可拦截请求） | 简单（仅缓存列表） | 需手动实现 |
| 推送通知 | 支持 | 不支持 | 不支持 |
| 灵活性 | 极高（编程控制） | 低（配置文件） | 中 |
| 状态 | 标准 | 已废弃 | 标准 |

**优缺点：**

- ✅ 优点：
  - 完全可编程的缓存控制
  - 离线体验接近原生 App
  - 推送通知增加用户粘性
- ❌ 缺点：
  - 调试困难（生命周期复杂）
  - 缓存更新策略需仔细设计
  - iOS 支持不完整（推送通知有限制）

## How — 怎么用

### 快速上手

**注册 Service Worker：**

```javascript
// main.js
if ('serviceWorker' in navigator) {
    window.addEventListener('load', async () => {
        try {
            const reg = await navigator.serviceWorker.register('/sw.js');
            console.log('SW 注册成功，作用域:', reg.scope);
        } catch (err) {
            console.error('SW 注册失败:', err);
        }
    });
}
```

**基础 Service Worker：**

```javascript
// sw.js
const CACHE_NAME = 'app-v1';
const ASSETS = [
    '/',
    '/index.html',
    '/styles/main.css',
    '/scripts/app.js',
    '/offline.html',
];

// 安装：预缓存核心资源
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then(cache => cache.addAll(ASSETS))
            .then(() => self.skipWaiting()) // 立即激活新 SW
    );
});

// 激活：清理旧缓存
self.addEventListener('activate', (event) => {
    event.waitUntil(
        caches.keys().then(keys =>
            Promise.all(
                keys.filter(key => key !== CACHE_NAME)
                    .map(key => caches.delete(key))
            )
        ).then(() => self.clients.claim()) // 立即控制所有页面
    );
});

// 拦截请求
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request)
            .then(cached => cached || fetch(event.request))
    );
});
```

### 代码示例

**缓存策略实现：**

```javascript
// Cache-First：优先缓存，适合不常变化的资源
async function cacheFirst(request) {
    const cached = await caches.match(request);
    if (cached) return cached;
    const response = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, response.clone());
    return response;
}

// Network-First：优先网络，适合频繁更新的内容
async function networkFirst(request) {
    try {
        const response = await fetch(request);
        const cache = await caches.open(CACHE_NAME);
        cache.put(request, response.clone());
        return response;
    } catch {
        const cached = await caches.match(request);
        return cached || caches.match('/offline.html');
    }
}

// Stale-While-Revalidate：先返回缓存，后台更新
async function staleWhileRevalidate(request) {
    const cache = await caches.open(CACHE_NAME);
    const cached = await cache.match(request);
    const fetchPromise = fetch(request).then(response => {
        cache.put(request, response.clone());
        return response;
    });
    return cached || fetchPromise;
}

// 路由分发
self.addEventListener('fetch', (event) => {
    const url = new URL(event.request.url);

    if (url.pathname.startsWith('/api/')) {
        event.respondWith(networkFirst(event.request));
    } else if (url.pathname.match(/\.(js|css|woff2)$/)) {
        event.respondWith(cacheFirst(event.request));
    } else {
        event.respondWith(staleWhileRevalidate(event.request));
    }
});
```

**Web App Manifest：**

```json
{
    "name": "My App",
    "short_name": "MyApp",
    "description": "A progressive web app",
    "start_url": "/",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#3b82f6",
    "icons": [
        { "src": "/icons/192.png", "sizes": "192x192", "type": "image/png" },
        { "src": "/icons/512.png", "sizes": "512x512", "type": "image/png" },
        { "src": "/icons/512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
    ]
}
```

```html
<!-- index.html -->
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#3b82f6">
<!-- iOS 兼容 -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<link rel="apple-touch-icon" href="/icons/192.png">
```

**推送通知：**

```javascript
// 前端：请求权限 + 订阅
async function subscribePush() {
    const permission = await Notification.requestPermission();
    if (permission !== 'granted') return;

    const reg = await navigator.serviceWorker.ready;
    const subscription = await reg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
    });

    // 将 subscription 发送到后端保存
    await fetch('/api/push/subscribe', {
        method: 'POST',
        body: JSON.stringify(subscription),
    });
}

// SW：接收推送
self.addEventListener('push', (event) => {
    const data = event.data?.json() ?? { title: '新消息', body: '' };
    event.waitUntil(
        self.registration.showNotification(data.title, {
            body: data.body,
            icon: '/icons/192.png',
            badge: '/icons/badge.png',
            data: { url: data.url },
        })
    );
});

// SW：通知点击
self.addEventListener('notificationclick', (event) => {
    event.notification.close();
    event.waitUntil(
        clients.openWindow(event.notification.data.url || '/')
    );
});
```

**后台同步：**

```javascript
// 前端：注册同步任务
async function submitOffline(formData) {
    // 保存到 IndexedDB
    await saveToOutbox(formData);
    const reg = await navigator.serviceWorker.ready;
    reg.sync.register('submit-data');
}

// SW：执行同步
self.addEventListener('sync', (event) => {
    if (event.tag === 'submit-data') {
        event.waitUntil(submitOutbox());
    }
});

async function submitOutbox() {
    const items = await getOutboxItems();
    await Promise.all(items.map(item =>
        fetch('/api/submit', {
            method: 'POST',
            body: JSON.stringify(item),
        }).then(() => removeFromOutbox(item.id))
    ));
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| SW 更新不生效 | 浏览器缓存了旧 SW | 修改 CACHE_NAME 版本号 + skipWaiting + clients.claim |
| 缓存雪崩 | 大量资源同时过期 | 分层缓存，按更新频率分组 |
| 调试困难 | SW 生命周期复杂 | Chrome DevTools → Application → Service Workers 面板 |
| iOS 推送限制 | iOS 16.4+ 才支持 | 降级为轮询或短信通知 |
| 作用域过大 | SW 注册在根路径 | 在子目录注册或设置 `Scope` 头 |

### 最佳实践

- 静态资源 Cache-First，API 请求 Network-First
- 缓存版本化，激活时清理旧版本
- 开发时勾选 "Update on reload" 方便调试
- Manifest 提供 maskable 图标，适配不同设备
- 离线页面提供基本导航，不要完全白屏

## 面试题

**Q1: Service Worker 的生命周期有哪些阶段？**
> 生命周期：`install`（安装，可预缓存资源）→ `waiting`（等待旧 SW 释放控制权）→ `activate`（激活，可清理旧缓存）→ `fetch` 事件拦截请求 → 空闲时 `terminate`（终止以节省内存，有事件时重新唤醒）。`skipWaiting()` 可跳过等待立即激活，`clients.claim()` 可立即控制所有页面。

**Q2: Service Worker 的缓存策略如何选择？**
> Cache-First：优先缓存，适合不常变化的静态资源（字体、图片）；Network-First：优先网络，适合频繁更新的内容（API、HTML）；Stale-While-Revalidate：先返回缓存再后台更新，兼顾速度与时效性（非关键 API）；Cache-Only / Network-Only 分别用于纯离线资源和必须实时的请求。按资源类型路由分发不同策略。

**Q3: PWA 的三要素是什么？**
> HTTPS（安全上下文，SW 运行的前提）、Service Worker（拦截请求、管理缓存、推送通知）、Web App Manifest（`manifest.json` 定义应用名、图标、启动 URL、显示模式，使 Web 应用可安装到桌面）。三者缺一不可，缺少任一项都无法成为完整的 PWA。

**Q4: Service Worker 和 Web Worker 有什么区别？**
> Service Worker 是特殊的 Web Worker，可拦截网络请求、管理缓存、接收推送通知，生命周期独立于页面（页面关闭后仍可运行），必须 HTTPS 运行；普通 Web Worker 仅用于计算密集型任务，与页面绑定（页面关闭即销毁），无法拦截请求和接收推送。两者都不能直接操作 DOM，需通过 `postMessage` 与主线程通信。

---

**相关链接：**
- [[HTTP与缓存策略]]
- [[浏览器渲染原理]]
- [[前端性能优化]]

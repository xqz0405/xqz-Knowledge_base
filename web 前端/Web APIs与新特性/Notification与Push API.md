---
tags:
  - Web前端
  - Notification
  - Push API
  - 推送通知
  - PWA
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Notification与Push API

## What — 是什么

> Notification API 让 Web 应用发送系统级通知（即使页面在后台），Push API 让服务端主动向客户端推送消息（即使浏览器未打开页面）。两者配合，Web 应用可实现类似原生 App 的推送通知体验，是 PWA 的核心能力。

**核心概念：**

- **Notification API**：浏览器端 API，显示系统通知栏消息，支持图标、操作按钮、进度条等
- **Push API**：订阅推送服务（Push Service），接收服务端推送的消息，配合 Service Worker 在后台处理
- **Service Worker**：Push 消息的接收者，即使页面关闭也能处理推送并显示通知
- **Push Subscription**：客户端订阅后获得的端点（endpoint）+ 密钥，服务端用其发送推送
- **VAPID**：Voluntary Application Server Identification，应用服务器身份认证，无需第三方推送服务注册
- **Web Push 协议**：基于 HTTP/2 的推送协议，服务端 → Push Service → 浏览器

**推送流程：**

```
┌──────────┐    ① subscribe     ┌──────────────┐    ③ push message    ┌──────────┐
│  Browser  │ ─────────────────→ │  Push Service │ ←────────────────── │  App     │
│  (Client) │                    │  (FCM/APNs/   │                     │  Server  │
│           │ ←───────────────── │   Mozilla)    │ ──────────────────→ │          │
│           │    ② endpoint      │               │    � deliver         │          │
│           │    + keys          │               │                     │          │
└─────┬─────┘                    └───────┬───────┘                     └──────────┘
      │                                  │
      │  ④ push event                    │
      ▼                                  │
┌──────────────┐                         │
│Service Worker│ ←───────────────────────┘
│  onpush      │    ⑤ show notification
│              │ ──────────────────→ 系统通知栏
└──────────────┘
```

**浏览器支持：**

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| Notification API | 22+ | 22+ | 16+（macOS only） | 17+ |
| Push API | 50+ | 44+ | 16.5+（macOS only） | 17+ |
| VAPID | 52+ | 46+ | 16.5+ | 17+ |

> iOS 16.4+ 支持 Web Push（需添加到主屏幕），Android Chrome 完整支持。

## Why — 为什么

**适用场景：**

- 即时通讯：新消息通知
- 电商：订单状态、促销活动
- 内容平台：新内容、评论回复
- 协作工具：任务分配、@提醒
- 金融：交易确认、行情预警
- 物流：配送状态更新

**对比其他通知方式：**

| 方式 | 页面关闭可用 | 系统级 | 实时性 | 兼容性 |
|------|------------|--------|--------|--------|
| Notification + Push | ✅ | ✅ | 高 | Chrome/Firefox/Safari |
| WebSocket + 页面内提示 | ❌ | ❌ | 高 | 全部 |
| SSE + 页面内提示 | ❌ | ❌ | 中 | 全部 |
| 轮询 + 页面内提示 | ❌ | ❌ | 低 | 全部 |
| 邮件/短信 | ✅ | ✅ | 中 | 全部 |

## How — 怎么用

### Notification API 基础

```ts
// 1. 检查支持
if (!('Notification' in window)) {
  console.log('浏览器不支持通知');
}

// 2. 请求权限
async function requestPermission(): Promise<boolean> {
  if (Notification.permission === 'granted') return true;
  if (Notification.permission === 'denied') return false;

  const result = await Notification.requestPermission();
  return result === 'granted';
}

// 3. 显示通知
async function showNotification(title: string, options?: NotificationOptions) {
  const granted = await requestPermission();
  if (!granted) return;

  const notification = new Notification(title, {
    body: '这是一条通知内容',
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    image: '/images/preview.jpg',
    tag: 'message-1',          // 相同 tag 会替换旧通知
    requireInteraction: true,  // 不自动消失，需手动关闭
    silent: false,
    vibrate: [200, 100, 200], // 振动模式 [振动, 暂停, 振动]
    ...options,
  });

  // 点击通知
  notification.onclick = (event) => {
    event.preventDefault();
    window.focus();  // 聚焦窗口
    window.location.href = '/messages/1';
    notification.close();
  };

  // 通知关闭
  notification.onclose = () => {
    console.log('通知已关闭');
  };

  // 通知出错
  notification.onerror = (error) => {
    console.error('通知错误:', error);
  };

  // 自动关闭（5秒后）
  setTimeout(() => notification.close(), 5000);
}
```

**Notification 选项详解：**

| 选项 | 类型 | 说明 |
|------|------|------|
| `body` | string | 通知正文 |
| `icon` | string | 通知图标 URL |
| `image` | string | 大图预览（Android 支持） |
| `badge` | string | 状态栏小图标（Android） |
| `tag` | string | 通知标识，相同 tag 替换旧通知 |
| `renotify` | boolean | tag 相同时是否再次提醒（默认 false） |
| `requireInteraction` | boolean | 不自动消失，需用户操作（默认 false） |
| `silent` | boolean | 静默通知，无声音无振动 |
| `vibrate` | number[] | 振动模式 |
| `timestamp` | number | 通知时间戳 |
| `data` | any | 附加数据 |
| `actions` | NotificationAction[] | 操作按钮（最多 2 个） |

### 通知操作按钮

```ts
// 带操作按钮的通知
function showActionNotification() {
  const notification = new Notification('新消息', {
    body: '张三: 今天开会吗？',
    icon: '/icons/icon-192x192.png',
    tag: 'chat-1',
    actions: [
      { action: 'reply', title: '回复', icon: '/icons/reply.png' },
      { action: 'ignore', title: '忽略', icon: '/icons/ignore.png' },
    ],
    data: { messageId: '123', chatId: '456' },
  });

  notification.onclick = (event: any) => {
    const action = event.action;
    const data = event.notification.data;

    if (action === 'reply') {
      window.focus();
      openReplyDialog(data.chatId);
    } else if (action === 'ignore') {
      markAsRead(data.messageId);
    } else {
      // 点击通知主体（非按钮）
      window.focus();
      navigateToChat(data.chatId);
    }

    event.notification.close();
  };
}
```

### Service Worker 中显示通知

```ts
// sw.js — Service Worker 中处理通知

// 监听推送事件
self.addEventListener('push', (event) => {
  const data = event.json();

  const options: NotificationOptions = {
    body: data.body,
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    image: data.image,
    tag: data.tag || `notification-${Date.now()}`,
    data: {
      url: data.url || '/',
      type: data.type,
      id: data.id,
    },
    actions: data.actions || [],
    vibrate: [200, 100, 200],
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

// 通知点击
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  const data = event.notification.data;
  const action = event.action;

  if (action === 'reply') {
    // 打开回复窗口
    event.waitUntil(
      clients.openWindow(`/chat/${data.id}?action=reply`)
    );
  } else {
    // 打开对应页面
    event.waitUntil(
      clients.matchAll({ type: 'window', includeUncontrolled: true }).then((clientList) => {
        // 如果已有打开的窗口，聚焦并导航
        for (const client of clientList) {
          if (client.url.includes(data.url) && 'focus' in client) {
            return client.focus();
          }
        }
        // 没有打开的窗口，新建
        return clients.openWindow(data.url);
      })
    );
  }
});

// 通知关闭
self.addEventListener('notificationclose', (event) => {
  const data = event.notification.data;
  // 上报通知关闭事件
  fetch('/api/notification/dismiss', {
    method: 'POST',
    body: JSON.stringify({ id: data.id }),
    keepalive: true,
  });
});
```

### Push API 订阅

```ts
// utils/push.ts

// VAPID 密钥对（服务端生成，公钥给客户端）
const VAPID_PUBLIC_KEY = 'BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkA...';

function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding).replace(/-/g, '+').replace(/_/g, '/');
  const rawData = atob(base64);
  return Uint8Array.from([...rawData].map((char) => char.charCodeAt(0)));
}

// 1. 注册 Service Worker
async function registerSW(): Promise<ServiceWorkerRegistration> {
  if (!('serviceWorker' in navigator)) {
    throw new Error('Service Worker 不支持');
  }

  const registration = await navigator.serviceWorker.register('/sw.js', {
    scope: '/',
  });
  return registration;
}

// 2. 订阅推送
async function subscribePush(): Promise<PushSubscription | null> {
  const registration = await registerSW();

  // 检查是否已订阅
  const existing = await registration.pushManager.getSubscription();
  if (existing) return existing;

  // 请求通知权限
  const permission = await Notification.requestPermission();
  if (permission !== 'granted') {
    console.warn('通知权限被拒绝');
    return null;
  }

  // 创建订阅
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,  // 必须为 true，推送必须显示通知
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
  });

  // 将订阅信息发送到服务端
  await sendSubscriptionToServer(subscription);

  return subscription;
}

// 3. 发送订阅到服务端
async function sendSubscriptionToServer(subscription: PushSubscription) {
  const subscriptionJson = subscription.toJSON();

  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      endpoint: subscriptionJson.endpoint,
      keys: {
        p256dh: subscriptionJson.keys!.p256dh,
        auth: subscriptionJson.keys!.auth,
      },
    }),
  });
}

// 4. 取消订阅
async function unsubscribePush(): Promise<boolean> {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();

  if (subscription) {
    const result = await subscription.unsubscribe();
    // 通知服务端删除订阅
    await fetch('/api/push/unsubscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ endpoint: subscription.endpoint }),
    });
    return result;
  }

  return false;
}
```

### 服务端推送实现

```ts
// server/push.ts — Node.js 服务端推送
import webpush from 'web-push';
import { readFileSync } from 'fs';

// 配置 VAPID
webpush.setVapidDetails(
  'mailto:admin@example.com',
  readFileSync('./vapid-public.pem', 'utf8'),
  readFileSync('./vapid-private.pem', 'utf8'),
);

// 生成 VAPID 密钥对（只需执行一次）
// const vapidKeys = webpush.generateVAPIDKeys();
// console.log('Public Key:', vapidKeys.publicKey);
// console.log('Private Key:', vapidKeys.privateKey);

interface PushSubscriptionData {
  endpoint: string;
  keys: { p256dh: string; auth: string };
}

// 发送推送
async function sendPushNotification(
  subscription: PushSubscriptionData,
  payload: {
    title: string;
    body: string;
    icon?: string;
    image?: string;
    url?: string;
    tag?: string;
    actions?: { action: string; title: string }[];
  }
) {
  try {
    const result = await webpush.sendNotification(
      subscription,
      JSON.stringify(payload),
      {
        TTL: 86400,           // 消息存活时间（秒）
        urgency: 'normal',   // very-low | low | normal | high
        topic: 'chat',       // 相同 topic 只保留最新一条
      }
    );
    console.log('Push sent:', result.statusCode);
  } catch (error: any) {
    if (error.statusCode === 410) {
      // 订阅已失效，从数据库删除
      console.log('Subscription expired, removing...');
      await removeSubscription(subscription.endpoint);
    } else {
      console.error('Push failed:', error.statusCode, error.message);
    }
  }
}

// 批量推送
async function broadcastPush(
  subscriptions: PushSubscriptionData[],
  payload: any
) {
  const results = await Promise.allSettled(
    subscriptions.map((sub) => sendPushNotification(sub, payload))
  );

  const succeeded = results.filter((r) => r.status === 'fulfilled').length;
  const failed = results.filter((r) => r.status === 'rejected').length;
  console.log(`Broadcast: ${succeeded} succeeded, ${failed} failed`);
}
```

### 服务端 API 路由

```ts
// routes/push.ts — Express 路由

// 存储订阅（生产环境用数据库）
const subscriptions = new Map<string, PushSubscriptionData>();

// 客户端注册推送
app.post('/api/push/subscribe', (req, res) => {
  const { endpoint, keys } = req.body;
  subscriptions.set(endpoint, { endpoint, keys });
  res.json({ success: true });
});

// 客户端取消推送
app.post('/api/push/unsubscribe', (req, res) => {
  const { endpoint } = req.body;
  subscriptions.delete(endpoint);
  res.json({ success: true });
});

// 服务端触发推送（如新消息、订单状态变更）
app.post('/api/push/send', async (req, res) => {
  const { userId, title, body, url } = req.body;

  // 查找用户订阅
  const userSubs = getUserSubscriptions(userId); // 从数据库获取
  if (!userSubs.length) {
    return res.json({ success: false, message: '用户未订阅' });
  }

  await broadcastPush(userSubs, { title, body, url });
  res.json({ success: true });
});
```

### React Hook 封装

```tsx
// hooks/usePushNotification.ts
function usePushNotification() {
  const [permission, setPermission] = useState<NotificationPermission>(
    'Notification' in window ? Notification.permission : 'denied'
  );
  const [subscription, setSubscription] = useState<PushSubscription | null>(null);

  useEffect(() => {
    (async () => {
      if (!('serviceWorker' in navigator)) return;
      const reg = await navigator.serviceWorker.ready;
      const sub = await reg.pushManager.getSubscription();
      setSubscription(sub);
    })();
  }, []);

  const requestPermission = useCallback(async () => {
    const result = await Notification.requestPermission();
    setPermission(result);
    return result === 'granted';
  }, []);

  const subscribe = useCallback(async () => {
    try {
      const sub = await subscribePush();
      setSubscription(sub);
      return !!sub;
    } catch (error) {
      console.error('Subscribe failed:', error);
      return false;
    }
  }, []);

  const unsubscribe = useCallback(async () => {
    const result = await unsubscribePush();
    if (result) setSubscription(null);
    return result;
  }, []);

  const showNotification = useCallback(async (title: string, options?: NotificationOptions) => {
    if (permission !== 'granted') {
      const granted = await requestPermission();
      if (!granted) return;
    }

    if ('serviceWorker' in navigator) {
      const reg = await navigator.serviceWorker.ready;
      reg.showNotification(title, options);
    } else {
      new Notification(title, options);
    }
  }, [permission, requestPermission]);

  return {
    permission,
    isSubscribed: !!subscription,
    requestPermission,
    subscribe,
    unsubscribe,
    showNotification,
  };
}

// 使用
function NotificationSettings() {
  const { permission, isSubscribed, subscribe, unsubscribe } = usePushNotification();

  return (
    <div>
      <h3>通知设置</h3>
      <p>权限状态: {permission}</p>
      {permission === 'default' && (
        <button onClick={subscribe}>开启通知</button>
      )}
      {permission === 'granted' && (
        <button onClick={isSubscribed ? unsubscribe : subscribe}>
          {isSubscribed ? '关闭推送' : '开启推送'}
        </button>
      )}
      {permission === 'denied' && (
        <p>通知权限已被拒绝，请在浏览器设置中手动开启</p>
      )}
    </div>
  );
}
```

### 推送权限最佳实践

```tsx
// 延迟请求权限：不要一进页面就弹权限请求
// 先解释通知价值，用户主动点击后再请求

function NotificationPrompt() {
  const [showExplain, setShowExplain] = useState(false);

  return (
    <div>
      {!showExplain ? (
        <button onClick={() => setShowExplain(true)}>
          开启消息通知
        </button>
      ) : (
        <div className="notification-explain">
          <h4>开启通知后，您可以：</h4>
          <ul>
            <li>及时收到新消息提醒</li>
            <li>订单状态变更第一时间通知</li>
            <li>重要活动不错过</li>
          </ul>
          <button onClick={async () => {
            const granted = await subscribePush();
            if (!granted) {
              // 权限被拒，引导用户到设置
              alert('请在浏览器设置中允许通知');
            }
          }}>
            确认开启
          </button>
          <button onClick={() => setShowExplain(false)}>暂不开启</button>
        </div>
      )}
    </div>
  );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 通知不显示 | 权限未授予或被拒绝 | 先 `requestPermission()`，检查返回值 |
| Push 消息收不到 | Service Worker 未注册或订阅过期 | 检查 SW 注册状态，处理 410 过期 |
| iOS 不支持 | iOS 16.4+ 需添加到主屏幕 | 引导用户"添加到主屏幕" |
| 通知无操作按钮 | 不在 Service Worker 中显示 | 操作按钮需在 SW 的 `showNotification` 中使用 |
| 通知点击不跳转 | 未处理 `notificationclick` 事件 | SW 中监听 `notificationclick` 并 `clients.openWindow` |
| VAPID 认证失败 | 公钥格式错误 | 使用 `urlBase64ToUint8Array` 转换 |
| 推送延迟 | Push Service 排队 | 设置 `urgency: 'high'` 和合理的 TTL |
| 通知重复显示 | 未设置 tag | 用 `tag` 标识同类通知，自动替换 |
| 订阅突然失效 | 浏览器更新/清理数据 | 监听 `pushsubscriptionchange` 事件重新订阅 |

### 最佳实践

- 延迟请求权限：先解释通知价值，用户主动操作后再弹权限
- 设置 `tag` 避免同类通知堆叠，相同 tag 自动替换
- 关键通知用 `requireInteraction: true`，确保用户看到
- 推送数据最小化（载荷 < 4KB），大数据引导用户打开页面获取
- 处理订阅失效：监听 `pushsubscriptionchange` 自动重新订阅
- 服务端处理 410 状态码，清理失效订阅
- 通知点击后 `window.focus()` + 页面导航，确保窗口激活
- iOS 用户需引导"添加到主屏幕"才能使用推送
- 通知图标用 192x192+ PNG，badge 用 72x72 单色
- 使用 `data` 字段携带路由信息，点击后精准跳转

## 面试题

**Q1: Notification API 和 Push API 的区别是什么？**
> Notification API 是客户端 API，用于在本地显示系统通知，只需页面打开且有权限即可。Push API 是客户端+服务端配合的推送机制，服务端通过 Push Service 将消息推送到浏览器，由 Service Worker 接收并显示通知，即使页面关闭也能工作。核心区别：Notification = 本地触发显示；Push = 服务端远程触发。Push 必须配合 Service Worker 和 Notification 使用（`userVisibleOnly: true` 要求推送必须显示通知）。

**Q2: Web Push 的完整流程是什么？**
> ① 客户端注册 Service Worker；② 客户端调用 `pushManager.subscribe()` 获取 PushSubscription（含 endpoint 和加密密钥）；③ 客户端将 subscription 发送到应用服务端存储；④ 服务端需要推送时，用 VAPID 私钥签名，通过 Web Push 协议向 Push Service（endpoint 指向的服务器）发送加密消息；⑤ Push Service 将消息推送到浏览器；⑥ 浏览器唤醒 Service Worker，触发 `push` 事件；⑦ Service Worker 调用 `showNotification()` 显示通知；⑧ 用户点击通知，触发 `notificationclick` 事件，打开对应页面。

**Q3: VAPID 是什么？为什么需要它？**
> VAPID（Voluntary Application Server Identification）是应用服务器身份认证协议，让 Push Service 识别推送消息的来源应用，无需向每个浏览器厂商单独注册。原理：应用服务器用 VAPID 私钥对推送请求签名，Push Service 用公钥验证。好处：① 自主管控：无需注册 GCM/APNs 等第三方推送服务；② 安全性：Push Service 可验证推送来源，拒绝未授权的推送；③ 多浏览器统一：同一套 VAPID 密钥适用于 Chrome/Firefox/Safari 所有浏览器。密钥生成：`web-push generate-vapid-keys` 或代码 `webpush.generateVAPIDKeys()`。

**Q4: 为什么 Push API 要求 `userVisibleOnly: true`？**
> `userVisibleOnly: true` 表示每次推送必须显示可见的通知。这是浏览器的隐私保护措施：防止应用在用户不知情的情况下在后台接收数据（静默追踪）。如果没有这个限制，网站可以在用户关闭页面后持续接收推送数据，用户完全无感知。有了 `userVisibleOnly`，每条推送都必须以通知形式告知用户，保证透明性。如果需要静默数据同步，应使用 Background Sync API 而非 Push API。

**Q5: iOS Safari 的 Web Push 有什么限制？**
> iOS 16.4+ 支持 Web Push，但有限制：① 必须添加到主屏幕（"添加到主屏幕"后的 PWA 才能接收推送），Safari 浏览器标签页不支持；② 需用户主动触发 `requestPermission()`，不能自动弹窗；③ 通知点击行为由系统处理，`notificationclick` 事件的支持有限；④ 不支持 `actions` 通知按钮；⑤ Badge（角标）通过 `navigator.setAppBadge()` 设置，不通过 Notification；⑥ 后台刷新间隔由系统控制，iOS 可能限制推送频率。建议：iOS 用户引导添加到主屏幕，并做兼容性检测降级。

**Q6: 如何处理 Push 订阅失效？**
> 订阅失效原因：① 用户清除浏览器数据；② 浏览器更新导致密钥变化；③ Push Service 清理长期未活跃的订阅。处理方式：① 监听 `pushsubscriptionchange` 事件（Firefox 支持，Chrome 不触发）：SW 中自动重新订阅并通知服务端；② 页面打开时检查订阅是否仍有效：`pushManager.getSubscription()` 后与本地缓存的 endpoint 对比；③ 服务端发送推送时处理 410 Gone 响应，从数据库删除失效订阅；④ 定期重新订阅策略：每次用户打开页面时，取消旧订阅、创建新订阅、更新服务端。

**Q7: Web Push 消息的加密机制是什么？**
> Web Push 使用 ECDH（椭圆曲线 Diffie-Hellman）+ AES-128-GCM 加密。流程：① 客户端订阅时生成一对 ECDH 密钥（p256dh 公钥 + auth 认证密钥），随 subscription 发送给服务端；② 服务端发送推送时，生成临时 ECDH 密钥对，用客户端公钥 + 临时私钥计算共享密钥，再用 auth 密钥派生加密密钥，最后用 AES-128-GCM 加密载荷；③ 加密后的消息 + 临时公钥一起发送给 Push Service；④ 浏览器收到后，用客户端私钥 + 临时公钥还原共享密钥，解密载荷。Push Service 只负责转发加密消息，无法看到内容。

**Q8: 如何设计一个支持百万用户的推送系统？**
> 四个层面：① 订阅管理：数据库存储用户订阅，按用户/设备索引，支持批量查询；定期清理 410 失效订阅；② 推送调度：消息队列（Kafka/RabbitMQ）缓冲推送任务，Worker 池并发发送；按 Push Service 分组（FCM/Mozilla），复用 HTTP/2 连接；③ 降级策略：推送失败回退到 WebSocket/轮询；频率限制避免被 Push Service 封禁；高优先级消息（交易确认）用 `urgency: 'high'`；④ 监控：推送成功率、送达延迟、订阅活跃度、410 失效率。关键优化：HTTP/2 多路复用减少连接数、批量推送合并同用户多设备、消息压缩减小载荷。

---

**相关链接：**
- [[Service Worker与PWA]]
- [[WebRTC实时通信]]
- [[WebTransport实时通信]]
- [[Fetch API与请求模式]]
- [[浏览器存储方案]]

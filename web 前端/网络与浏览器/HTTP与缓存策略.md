---
tags:
  - Web前端
  - HTTP
  - 缓存
  - 网络协议
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# HTTP与缓存策略

## What — 是什么

> HTTP 是 Web 通信的基础协议，缓存策略决定资源是否使用本地副本，直接影响页面加载速度和服务器压力。

### HTTP 协议演进

**HTTP/0.9（1991）**：仅支持 GET，无 Header，响应只有 HTML。

**HTTP/1.0（1996）**：
- 增加 POST/HEAD 方法
- 引入 Header（Content-Type、Status Code 等）
- 每次请求需新建 TCP 连接（Connection: close）

**HTTP/1.1（1997，最广泛使用）**：
- 持久连接（`Connection: keep-alive`），默认复用 TCP 连接
- 管线化（Pipelining）：允许连续发送多个请求，但响应必须按序返回（队头阻塞）
- 新增方法：PUT、DELETE、OPTIONS、PATCH
- 新增 Host Header（支持虚拟主机）
- 分块传输编码（`Transfer-Encoding: chunked`）

**HTTP/2（2015）**：
- 多路复用：单连接并行多个请求/响应，解决 HTTP 层队头阻塞
- 头部压缩：HPACK 算法，静态表 + 动态表 + 哈夫曼编码
- 服务器推送：主动推送关联资源
- 请求优先级：权重和依赖关系
- 二进制分帧：将消息拆分为更小的帧

**HTTP/3（2022）**：
- 传输层从 TCP 切换为 QUIC（基于 UDP）
- 彻底解决 TCP 层队头阻塞
- 0-RTT / 1-RTT 连接建立
- 连接迁移：网络切换不断连（基于 Connection ID）
- 内置 TLS 1.3

### HTTP 状态码详解

| 状态码 | 类别 | 常见码 | 含义 |
|--------|------|--------|------|
| 1xx | 信息 | 100 | Continue，继续发送请求体 |
| 2xx | 成功 | 200 | OK，请求成功 |
| | | 201 | Created，资源创建成功 |
| | | 204 | No Content，成功但无返回体 |
| | | 206 | Partial Content，范围请求 |
| 3xx | 重定向 | 301 | 永久重定向（浏览器缓存） |
| | | 302 | 临时重定向（不缓存） |
| | | 304 | Not Modified，协商缓存命中 |
| | | 307 | 临时重定向，保持方法不变 |
| | | 308 | 永久重定向，保持方法不变 |
| 4xx | 客户端错误 | 400 | Bad Request，请求格式错误 |
| | | 401 | Unauthorized，未认证 |
| | | 403 | Forbidden，无权限 |
| | | 404 | Not Found，资源不存在 |
| | | 405 | Method Not Allowed |
| | | 429 | Too Many Requests，限流 |
| 5xx | 服务端错误 | 500 | Internal Server Error |
| | | 502 | Bad Gateway，网关错误 |
| | | 503 | Service Unavailable，服务不可用 |
| | | 504 | Gateway Timeout，网关超时 |

### 缓存核心概念

- **强缓存**：`Cache-Control` / `Expires`，不发请求直接用本地缓存
- **协商缓存**：`ETag` / `Last-Modified`，发请求验证资源是否变化
- **缓存位置优先级**：Service Worker → Memory Cache → Disk Cache → Push Cache

**关键特性：**

- `Cache-Control: max-age=3600` 强缓存 1 小时
- `Cache-Control: no-cache` 跳过强缓存，走协商缓存
- `Cache-Control: no-store` 完全不缓存
- `Cache-Control: public` 允许中间代理缓存
- `Cache-Control: private` 只允许浏览器缓存
- `Cache-Control: immutable` 资源不会变化，无需协商验证
- `Expires` 是 HTTP/1.0 的绝对时间，优先级低于 `Cache-Control`
- HTTP/2 多路复用解决队头阻塞，单连接并行请求

## Why — 为什么

**适用场景：**

- 静态资源缓存（JS/CSS/图片）
- API 响应缓存
- 性能优化
- 离线应用支持
- CDN 分发加速

**对比方案：**

| 维度 | 强缓存 | 协商缓存 | Service Worker |
|------|--------|---------|---------------|
| 请求次数 | 0 | 1（验证请求） | 0（可拦截） |
| 时效性 | 过期前不更新 | 每次验证 | 代码控制 |
| 适用资源 | 带 hash 的静态资源 | HTML/频繁更新的资源 | 离线/复杂策略 |
| 灵活性 | 低 | 中 | 高 |
| 存储位置 | 浏览器缓存 | 浏览器缓存 | Cache API |
| 缓存更新 | 等 max-age 过期 | 服务器判断 | 代码主动更新 |

**优缺点：**

- ✅ 优点：
  - 强缓存零请求，极致性能
  - 协商缓存保证时效性
  - 分层策略覆盖不同资源类型
  - CDN 缓存减少源站压力
- ❌ 缺点：
  - 缓存策略配置不当导致更新不及时
  - 用户强制刷新绕过缓存
  - CDN 缓存与源站不一致
  - 协商缓存每次仍需发请求

## How — 怎么用

### 快速上手

**静态资源缓存策略（推荐）：**

```
# HTML 文件：不缓存或短缓存（确保用户拿到最新版）
Cache-Control: no-cache（配合 ETag 走协商缓存）

# JS/CSS/图片（带 hash 文件名）：长期强缓存
Cache-Control: public, max-age=31536000, immutable
# 文件名含 hash，内容变了文件名也变，缓存自动失效

# API 响应：按业务设置
Cache-Control: no-store  # 敏感数据
Cache-Control: max-age=60 # 实时性要求低的数据
```

### 代码示例

**1. 完整缓存流程：**

```
第一次请求：
  浏览器 → 服务器：GET /app.js
  服务器 → 浏览器：200 OK + 资源 + Cache-Control: max-age=3600 + ETag: "abc123"

1小时内再次请求（强缓存命中）：
  浏览器直接使用本地缓存，不发请求

1小时后请求（强缓存过期，走协商缓存）：
  浏览器 → 服务器：GET /app.js + If-None-Match: "abc123"
  服务器 → 浏览器：304 Not Modified（资源未变）或 200 OK + 新资源
```

**2. Nginx 缓存配置：**

```nginx
# 带 hash 的静态资源：1 年强缓存
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# HTML：协商缓存
location / {
    try_files $uri $uri/ /index.html;
    add_header Cache-Control "no-cache";
    etag on;
}

# API 代理：不缓存
location /api/ {
    proxy_pass http://backend;
    add_header Cache-Control "no-store";
}

# 图片资源：30 天缓存
location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
    expires 30d;
    add_header Cache-Control "public";
    access_log off;
}
```

**3. Express 服务器缓存配置：**

```javascript
const express = require('express');
const app = express();

// 静态资源：1 年强缓存
app.use('/assets', express.static('dist/assets', {
    maxAge: '1y',
    immutable: true,
    etag: false, // 带 hash 不需要 ETag
}));

// HTML：不缓存
app.get('*', (req, res) => {
    res.setHeader('Cache-Control', 'no-cache');
    res.sendFile('index.html');
});

// API 数据缓存
app.get('/api/articles', (req, res) => {
    res.setHeader('Cache-Control', 'public, max-age=60, s-maxage=300');
    // s-maxage: CDN 缓存 5 分钟，浏览器缓存 1 分钟
    res.json(articles);
});
```

**4. Service Worker 缓存策略：**

```javascript
// sw.js - Service Worker 缓存策略
const CACHE_NAME = 'app-v1';
const STATIC_ASSETS = [
    '/',
    '/styles/main.css',
    '/app.js',
];

// 安装：预缓存静态资源
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then(cache => cache.addAll(STATIC_ASSETS))
    );
});

// 请求拦截：缓存优先，网络回退
self.addEventListener('fetch', (event) => {
    const { request } = event;

    // API 请求：网络优先，缓存回退
    if (request.url.includes('/api/')) {
        event.respondWith(
            fetch(request)
                .then(response => {
                    const clone = response.clone();
                    caches.open(CACHE_NAME).then(cache => cache.put(request, clone));
                    return response;
                })
                .catch(() => caches.match(request))
        );
        return;
    }

    // 静态资源：缓存优先
    event.respondWith(
        caches.match(request).then(cached => {
            return cached || fetch(request).then(response => {
                const clone = response.clone();
                caches.open(CACHE_NAME).then(cache => cache.put(request, clone));
                return response;
            });
        })
    );
});

// 激活：清理旧缓存
self.addEventListener('activate', (event) => {
    event.waitUntil(
        caches.keys().then(keys =>
            Promise.all(keys
                .filter(key => key !== CACHE_NAME)
                .map(key => caches.delete(key))
            )
        )
    );
});
```

**5. CDN 缓存策略配置：**

```javascript
// CDN 回源配置要点
// 1. 尊源：CDN 遵循源站的 Cache-Control Header
// 2. 自定义 TTL：不遵循源站时设置 CDN 层缓存时间
// 3. 缓存键：默认 URL 作为键，可添加 Query String / Header / Cookie 维度

// 强制刷新 CDN 缓存
// 方式1：URL 加版本号 /app.js?v=2
// 方式2：CDN 控制台手动刷新
// 方式3：CDN API 刷新
```

```bash
# 阿里云 CDN 刷新示例
aliyun cdn RefreshObjectCaches --ObjectPath https://cdn.example.com/assets/app.js
```

**6. HTTP/2 Server Push 配置：**

```nginx
# Nginx HTTP/2 推送
http2_push /assets/style.css;
http2_push /assets/app.js;

# 或用 Link Header 自动推送
add_header Link "</assets/style.css>; rel=preload; as=style";
add_header Link "</assets/app.js>; rel=preload; as=script";
```

```javascript
// Node.js HTTP/2 推送
const http2 = require('http2');
const server = http2.createSecureServer(options);

server.on('stream', (stream, headers) => {
    stream.respond({ 'content-type': 'text/html', ':status': 200 });
    stream.end('<h1>Hello</h1>');

    // 推送关联资源
    stream.pushStream({ ':path': '/style.css' }, (pushStream) => {
        pushStream.respond({ 'content-type': 'text/css' });
        pushStream.end('body { margin: 0; }');
    });
});
```

**7. 浏览器缓存检测工具：**

```javascript
// Performance API 检查缓存命中情况
const [nav] = performance.getEntriesByType('navigation');
console.log('传输协议:', nav.nextHopProtocol); // h2 = HTTP/2
console.log('传输大小:', nav.transferSize);     // 0 = 强缓存命中
console.log('解码大小:', nav.decodedBodySize);
console.log('缓存命中:', nav.transferSize === 0 ? '强缓存' : '未命中');

// 检查资源缓存状态
performance.getEntriesByType('resource').forEach(entry => {
    if (entry.transferSize === 0 && entry.decodedBodySize > 0) {
        console.log(entry.name, '命中缓存');
    }
});
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 发版后用户看到旧页面 | HTML 被强缓存，引用旧 JS | HTML 不强缓存，JS 文件名带 hash |
| CDN 缓存不更新 | CDN 回源策略太长 | CDN 刷新 API 或缩短 TTL |
| 304 太多影响性能 | 协商缓存每次都发请求 | 静态资源用 hash + 长期强缓存 |
| HTTP/2 推送浪费带宽 | 推送了用户已有的资源 | 保守使用推送，优先用 `<link rel="preload">` |
| ETag 计算不一致 | 多台服务器文件时间/inode 不同 | 使用弱 ETag 或基于内容哈希的强 ETag |
| 301 重定向被缓存 | 浏览器永久缓存 301 目标 | 临时重定向用 302/307，301 只在确认永久迁移时用 |
| Cookie 影响缓存 | 请求带 Cookie 时 CDN 可能不缓存 | 静态资源用独立无 Cookie 域名 |
| 跨域资源缓存失败 | 缺少 CORS Header | 响应加 `Access-Control-Allow-Origin` |
| Service Worker 缓存更新 | SW 更新后旧缓存仍生效 | 监听 `activate` 事件清理旧缓存 |
| Vary Header 误用 | `Vary: *` 导致缓存完全失效 | 只对确实需要区分的 Header 设置 Vary |

### 最佳实践

- HTML：`no-cache`（协商缓存）
- 带 hash 的 JS/CSS/图片：`max-age=31536000, immutable`
- API 响应：按业务设置，通常 `no-store` 或短缓存
- 优先用 HTTP/2，避免域分片
- 静态资源使用独立域名（无 Cookie，CDN 友好）
- ETag 在多服务器部署时注意一致性
- 使用 Service Worker 实现离线可用和精细缓存控制
- 利用 `s-maxage` 区分 CDN 缓存和浏览器缓存

## 面试题

**Q1: HTTP/1.1 和 HTTP/2 的主要区别是什么？**
> HTTP/2 核心改进：多路复用（单连接并行请求，解决 HTTP 层队头阻塞）、头部压缩（HPACK 算法，静态表+动态表+哈夫曼编码）、服务器推送、请求优先级、二进制分帧。HTTP/1.1 每个请求需单独 TCP 连接或管线化（有队头阻塞问题），且头部重复传输浪费带宽。

**Q2: 强缓存和协商缓存的区别是什么？各自用什么 Header 控制？**
> 强缓存：不发请求，直接用本地副本，由 `Cache-Control`（`max-age`）和 `Expires` 控制；协商缓存：发请求验证资源是否变化，由 `ETag`/`If-None-Match` 和 `Last-Modified`/`If-Modified-Since` 控制，命中返回 304。强缓存优先级高于协商缓存。`Cache-Control` 优先级高于 `Expires`。

**Q3: ETag 的原理是什么？强 ETag 和弱 ETag 有什么区别？**
> ETag 是服务器为资源生成的唯一标识符（通常基于内容哈希）。请求时客户端通过 `If-None-Match` 携带上次 ETag，服务器比对决定返回 304 还是新资源。强 ETag（默认）要求资源字节级一致才相同；弱 ETag（`W/"..."`）语义等价即可，适用于可接受轻微差异的场景。多服务器部署时强 ETag 可能不一致（inode/时间戳差异），此时建议用弱 ETag 或基于内容哈希。

**Q4: HTTPS 的握手过程是怎样的？TLS 1.2 和 TLS 1.3 有什么区别？**
> TLS 1.2：客户端发送 ClientHello（支持的加密套件、随机数）→ 服务端回 ServerHello（选定套件、证书、随机数）→ 客户端验证证书，生成预主密钥，用服务端公钥加密发送 → 双方基于三个随机数生成会话密钥 → 完成加密通信（2-RTT）。TLS 1.3 简化为 1-RTT：客户端在 ClientHello 中同时发送密钥共享（Key Share），服务端在 ServerHello 中直接完成密钥协商；支持 0-RTT 恢复；移除了不安全的加密算法（RSA 密钥交换、CBC 模式等）。

**Q5: HTTP/3 为什么弃用 TCP 改用 QUIC？解决了什么问题？**
> TCP 的队头阻塞问题：一个 TCP 连接中某个包丢失，后续所有数据都要等待重传，即使它们属于不同的 HTTP 请求。QUIC 基于 UDP，每个流独立传输，一个流的丢包不影响其他流。此外 QUIC 还提供：0-RTT 连接建立（TLS 握手和传输握手合并）、连接迁移（基于 Connection ID 而非四元组，网络切换不断连）、更快的拥塞控制恢复。

**Q6: 如何设计一个完整的前端缓存方案？**
> 分层策略：1) HTML 用协商缓存（`no-cache` + ETag），确保入口最新；2) 带 hash 的 JS/CSS/图片用长期强缓存（`max-age=31536000, immutable`），文件名变则缓存自动失效；3) API 响应根据实时性要求设置，敏感数据 `no-store`，可缓存数据配 `max-age` + `s-maxage` 区分 CDN 和浏览器；4) Service Worker 做精细缓存控制和离线支持；5) 静态资源独立域名避免 Cookie 污染；6) CDN 边缘缓存减少回源。

**Q7: Cache-Control 的 public 和 private 有什么区别？什么时候用？**
> `public` 允许中间代理（CDN）缓存响应，`private` 只允许浏览器缓存。默认情况下，带 Authorization Header 或 Set-Cookie 的响应会被视为 private。静态资源（JS/CSS/图片）用 `public` 让 CDN 缓存；包含用户个人数据的 API 响应用 `private` 防止 CDN 缓存敏感信息。

**Q8: Last-Modified 和 ETag 哪个优先级更高？各有什么局限性？**
> 服务器同时收到 `If-Modified-Since` 和 `If-None-Match` 时，ETag 优先级更高（Nginx/Apache 均如此）。Last-Modified 局限性：1) 精度只到秒，1 秒内多次修改无法区分；2) 文件内容未变但修改时间变了会误判为更新；3) 不同服务器时间可能不一致。ETag 局限性：1) 默认基于 inode 生成，分布式服务器 inode 不同导致相同文件 ETag 不同；2) 计算开销比时间戳高。建议：静态资源用基于内容哈希的 ETag，或用带 hash 文件名 + 强缓存完全跳过协商。

---

**相关链接：**
- [[浏览器渲染原理]]
- [[跨域解决方案]]
- [[Webpack与Vite]]
- [[Service Worker与PWA]]
- [[HTTP2与HTTP3]]

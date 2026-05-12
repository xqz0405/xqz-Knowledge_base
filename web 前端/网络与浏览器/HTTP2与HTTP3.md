---
tags:
  - Web前端
  - HTTP/2
  - HTTP/3
  - QUIC
  - 网络协议
  - 性能优化
date: 2026-05-12
status: 已完成
difficulty: 中高
---

# HTTP/2与HTTP/3

## What — 是什么

> HTTP/2 通过多路复用、头部压缩、服务器推送等特性，在 HTTP/1.1 基础上大幅提升传输效率；HTTP/3 基于 QUIC 协议（UDP），彻底解决 TCP 层的队头阻塞和握手延迟，是 Web 传输协议的最新演进。

**协议演进：**

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP 协议演进                              │
├──────────────┬──────────────┬───────────────────────────────┤
│  HTTP/1.1    │  HTTP/2      │  HTTP/3                       │
│  1997        │  2015        │  2022 (RFC 9114)              │
├──────────────┼──────────────┼───────────────────────────────┤
│  串行请求    │  多路复用    │  无队头阻塞                    │
│  纯文本头部  │  HPACK 压缩  │  QPACK 压缩                   │
│  无推送      │  Server Push │  无推送（简化）                │
│  TCP         │  TCP         │  QUIC (UDP)                   │
│  1.5-RTTL    │  1-RTT       │  0-RTT 重连                   │
│  队头阻塞    │  TCP层队阻   │  无队头阻塞                    │
│  明文传输    │  可明文      │  强制 TLS 1.3                 │
└──────────────┴──────────────┴───────────────────────────────┘
```

**HTTP/2 核心概念：**

- **多路复用（Multiplexing）**：单 TCP 连接上并行交错发送多个请求/响应，通过 Stream ID 区分
- **帧（Frame）**：HTTP/2 最小通信单位，包括 HEADERS、DATA、SETTINGS、PING、GOAWAY 等类型
- **流（Stream）**：双向的字节流，承载一对请求/响应，用 31 位无符号整数标识
- **HPACK 头部压缩**：静态表（61 个常用头部）+ 动态表 + Huffman 编码，压缩率 ~85%
- **服务器推送（Server Push）**：服务端主动推送与请求关联的资源（如 HTML 请求后自动推送 CSS/JS）
- **流量控制（Flow Control）**：流级别和连接级别的流量控制，类似 TCP 滑动窗口
- **优先级（Priority）**：客户端可指定流的权重和依赖关系，指导服务端资源分配

**HTTP/3 核心概念：**

- **QUIC 协议**：Google 设计的基于 UDP 的传输协议，集成 TLS 1.3，替代 TCP+TLS
- **0-RTT 连接**：重连时首包即携带应用数据，无需等待握手完成
- **连接迁移（Connection Migration）**：网络切换（WiFi→4G）时连接不断，用 Connection ID 而非四元组标识
- **无队头阻塞**：每个 QUIC Stream 独立拥塞控制，一个包丢失只阻塞对应 Stream
- **QPACK 头部压缩**：QUIC 版 HPACK，解决 HPACK 在乱序交付时的队头阻塞问题
- **内置加密**：QUIC 强制 TLS 1.3，所有数据加密传输，无明文选项

**HTTP/2 帧类型：**

| 帧类型 | 说明 |
|--------|------|
| DATA | 传输请求/响应体 |
| HEADERS | 传输压缩后的头部字段 |
| PRIORITY | 指定流的优先级 |
| RST_STREAM | 异常终止流 |
| SETTINGS | 通信配置参数（最大并发流、初始窗口等） |
| PUSH_PROMISE | 服务器推送声明 |
| PING | 连接保活和 RTT 测量 |
| GOAWAY | 优雅关闭连接 |
| WINDOW_UPDATE | 流量控制窗口更新 |
| CONTINUATION | 头部块的续传 |

## Why — 为什么

**HTTP/1.1 的性能瓶颈：**

| 问题 | 说明 | HTTP/2 解决方案 |
|------|------|----------------|
| 队头阻塞 | 一个请求未完成，后续请求排队 | 多路复用，单连接并行 |
| 头部冗余 | 每次请求携带完整 Header（Cookie 可达数 KB） | HPACK 压缩，增量编码 |
| TCP 连接开销 | 并行请求需要多个 TCP 连接（浏览器限制 6 个/域名） | 单连接多路复用 |
| 无推送 | 客户端必须解析 HTML 后才请求 CSS/JS | Server Push 主动推送 |
| 优先级缺失 | 无法告诉服务端哪个资源更重要 | Stream Priority |

**HTTP/2 的残留问题（HTTP/3 解决）：**

| 问题 | HTTP/2 现状 | HTTP/3 解决方案 |
|------|------------|----------------|
| TCP 队头阻塞 | 一个 TCP 包丢失，所有 Stream 等待重传 | QUIC 独立 Stream，丢包只影响对应流 |
| 握手延迟 | TCP 1-RTT + TLS 1-RTT = 2-RTT | QUIC 合并握手，首次 1-RTT，重连 0-RTT |
| 网络切换断连 | IP/端口变化导致 TCP 连接中断 | Connection ID 标识连接，无缝迁移 |
| 中间设备干扰 | NAT/防火墙修改 TCP 行为 | 基于 UDP，中间设备不干预 |

**浏览器支持：**

| 协议 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| HTTP/2 | 41+ | 36+ | 9+ | 12+ |
| HTTP/3 | 87+ (默认启用) | 89+ | 16+ | 87+ |

> HTTP/3 使用率已超过 30%（Google 统计），主流 CDN（Cloudflare/AWS/Fastly）均已支持。

## How — 怎么用

### HTTP/2 服务端配置

**Nginx 配置 HTTP/2：**

```nginx
server {
    listen 443 ssl http2;           # 启用 HTTP/2
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.pem;
    ssl_certificate_key /etc/ssl/private/example.key;
    ssl_protocols       TLSv1.2 TLSv1.3;

    # HTTP/2 Server Push（Nginx 1.25+ 已弃用，推荐用 preload）
    # http2_push /css/main.css;
    # http2_push /js/app.js;

    # 推荐：Link preload 替代 Server Push
    add_header Link "</css/main.css>; rel=preload; as=style";
    add_header Link "</js/app.js>; rel=preload; as=script";

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

**Nginx 配置 HTTP/3（QUIC）：**

```nginx
server {
    listen 443 quic reuseport;      # HTTP/3 (QUIC)
    listen 443 ssl http2;           # HTTP/2 回退
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.pem;
    ssl_certificate_key /etc/ssl/private/example.key;
    ssl_protocols       TLSv1.3;    # QUIC 要求 TLS 1.3

    # Alt-Svc 头：告诉浏览器支持 HTTP/3
    add_header Alt-Svc 'h3=":443"; ma=86400';

    # 启用 0-RTT
    ssl_early_data on;

    location / {
        proxy_pass http://backend;
    }
}
```

**Node.js 启用 HTTP/2：**

```ts
import { createSecureServer } from 'http2';
import { readFileSync } from 'fs';

const server = createSecureServer({
  key: readFileSync('./server.key'),
  cert: readFileSync('./server.crt'),
  allowHTTP1: true,  // 兼容 HTTP/1.1
});

server.on('stream', (stream, headers) => {
  const path = headers[':path'];

  // Server Push
  if (path === '/') {
    stream.pushStream({ ':path': '/css/main.css' }, (err, pushStream) => {
      if (!err) {
        pushStream.respond({ 'content-type': 'text/css' });
        pushStream.end('body { margin: 0; }');
      }
    });

    stream.pushStream({ ':path': '/js/app.js' }, (err, pushStream) => {
      if (!err) {
        pushStream.respond({ 'content-type': 'application/javascript' });
        pushStream.end('console.log("hello");');
      }
    });
  }

  stream.respond({ 'content-type': 'text/html' });
  stream.end('<!DOCTYPE html><html><head><link rel="stylesheet" href="/css/main.css"></head><body><script src="/js/app.js"></script></body></html>');
});

server.listen(8443);
```

**Node.js 启用 HTTP/3（quic库）：**

```bash
npm install @fails-components/webtransport
```

### 前端适配 HTTP/2

**域名分片不再需要：**

```html
<!-- HTTP/1.1 时代：多域名突破 6 连接限制 -->
<link rel="stylesheet" href="https://cdn1.example.com/style.css">
<link rel="stylesheet" href="https://cdn2.example.com/theme.css">
<script src="https://cdn1.example.com/app.js"></script>
<script src="https://cdn2.example.com/vendor.js"></script>

<!-- HTTP/2 时代：单域名即可，多路复用无连接限制 -->
<link rel="stylesheet" href="https://cdn.example.com/style.css">
<link rel="stylesheet" href="https://cdn.example.com/theme.css">
<script src="https://cdn.example.com/app.js"></script>
<script src="https://cdn.example.com/vendor.js"></script>
```

**资源合并策略变化：**

```ts
// HTTP/1.1 时代：合并文件减少请求
// bundle.js = jquery.js + lodash.js + app.js (1个请求)

// HTTP/2 时代：拆分文件更好利用缓存
// jquery.js  → 几乎不变，强缓存命中
// lodash.js  → 几乎不变，强缓存命中
// app.js     → 频繁更新，只重新下载这个
// 多路复用下 3 个请求的开销 ≈ 1 个请求

// Vite/Webpack 代码分割配置
// vite.config.ts — HTTP/2 友好的分包策略
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-utils': ['lodash-es', 'dayjs'],
          'vendor-chart': ['echarts'],
        },
      },
    },
  },
});
```

**Preload 替代 Server Push：**

```html
<!-- Server Push 在 HTTP/2 中已被主流浏览器弱化 -->
<!-- Chrome 106+ 移除了对 Server Push 的支持 -->
<!-- 推荐使用 <link rel="preload"> -->

<head>
  <!-- 关键 CSS 预加载 -->
  <link rel="preload" href="/css/critical.css" as="style">
  <!-- 关键 JS 预加载 -->
  <link rel="preload" href="/js/app.js" as="script">
  <!-- 字体预加载 -->
  <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
  <!-- 图片预加载 -->
  <link rel="preload" href="/img/hero.webp" as="image" type="image/webp">

  <!-- Preconnect：提前建立连接 -->
  <link rel="preconnect" href="https://api.example.com">
  <link rel="preconnect" href="https://cdn.example.com" crossorigin>
</head>
```

### HTTP/2 流优先级

```ts
// HTTP/2 Stream Priority 告诉服务端资源的重要性
// 浏览器自动处理，但开发者可通过加载顺序影响

// 关键资源在前
<head>
  <!-- 最高优先级：阻塞渲染的 CSS -->
  <link rel="stylesheet" href="/css/critical.css">
  <!-- 高优先级：首屏 JS -->
  <script src="/js/hero.js"></script>
</head>
<body>
  <!-- 中优先级：非首屏 JS -->
  <script src="/js/analytics.js" defer></script>
  <!-- 低优先级：图片等 -->
  <img src="/img/lazy.jpg" loading="lazy">
</body>

// Fetch API 设置优先级（Chrome 101+）
fetch('/api/critical', { priority: 'high' });
fetch('/api/analytics', { priority: 'low' });
```

### HTTP/3 连接迁移

```ts
// HTTP/3 连接迁移：网络切换不断连
// 场景：WiFi → 4G，IP 变化但连接保持

// 服务端：QUIC 用 Connection ID 标识连接
// 客户端自动处理，无需前端代码

// 但前端可检测网络变化，优化体验
const connection = navigator.connection;

if (connection) {
  connection.addEventListener('change', () => {
    console.log('网络变化:', connection.type, connection.effectiveType);

    // 网络降级时暂停非关键请求
    if (connection.effectiveType === '2g' || connection.saveData) {
      pauseNonCriticalRequests();
    }
  });
}
```

### HTTP/2 调试

```bash
# 检查网站是否支持 HTTP/2
curl -I -s --http2 https://example.com | grep -i "HTTP/"

# 检查是否支持 HTTP/3 (Alt-Svc 头)
curl -I -s https://example.com | grep -i "alt-svc"

# Chrome 查看 HTTP 协议版本
# chrome://net-internals/#http2
# DevTools → Network → Protocol 列

# 启用 Chrome HTTP/3 指标
# chrome://flags/#enable-quic
```

**Chrome DevTools 查看：**

```
Network 面板 → 右键列头 → 勾选 Protocol
显示: h2 (HTTP/2) / h3 (HTTP/3) / http/1.1
```

### 性能对比实战

```ts
// HTTP/1.1 vs HTTP/2 加载对比

// 场景：加载 100 个小图片
// HTTP/1.1：6 个并发连接 × 17 轮 ≈ 1.7s
// HTTP/2：1 个连接 × 100 流 ≈ 0.3s

// 场景：首屏加载（HTML + CSS + JS + API）
// HTTP/1.1：串行 TCP + TLS 握手，多连接开销
// HTTP/2：1 个连接，并行加载

// 场景：弱网重连
// HTTP/2 over TCP：TCP 重新握手 + TLS 重新握手 = 2-3 RTT
// HTTP/3 over QUIC：0-RTT，首包即带数据

// 使用 Performance API 对比
function measureProtocolPerformance() {
  const entries = performance.getEntriesByType('resource');
  const h2Entries = entries.filter(e => (e as any).nextHopProtocol === 'h2');
  const h3Entries = entries.filter(e => (e as any).nextHopProtocol === 'h3');
  const http1Entries = entries.filter(e => (e as any).nextHopProtocol === 'http/1.1');

  console.table({
    'HTTP/1.1': { count: http1Entries.length, avgDuration: avg(http1Entries.map(e => e.duration)) },
    'HTTP/2':   { count: h2Entries.length, avgDuration: avg(h2Entries.map(e => e.duration)) },
    'HTTP/3':   { count: h3Entries.length, avgDuration: avg(h3Entries.map(e => e.duration)) },
  });
}
```

### HTTP/2 Server Push 的替代方案

```ts
// Server Push 已被 Chrome 弃用，替代方案：

// 方案1: <link rel="preload">（最推荐）
// 提前加载关键资源，浏览器自动决策是否需要
<link rel="preload" href="/css/main.css" as="style">
<link rel="preload" href="/js/app.js" as="script">

// 方案2: 103 Early Hints（HTTP 状态码）
// 服务端在正式响应前发送预加载提示
// Nginx 配置:
// add_header Link "</css/main.css>; rel=preload; as=style";
// 返回 103 状态码，浏览器提前加载

// 方案3: Service Worker 预缓存
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => cache.addAll([
      '/css/main.css',
      '/js/app.js',
      '/fonts/inter.woff2',
    ]))
  );
});

// 方案4: 内联关键资源
// 将首屏关键 CSS/JS 内联到 HTML 中，减少一个请求
<style>/* critical CSS inline */</style>
<script>/* critical JS inline */</script>
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| HTTP/2 未生效 | 未配置 HTTPS | HTTP/2 要求 TLS（浏览器强制） |
| 多域名反而变慢 | HTTP/2 单连接最优，多域名增加 DNS+TCP 开销 | 合并为单域名 + CDN |
| 大文件合并后缓存差 | 整个 bundle 一改全重下载 | HTTP/2 下拆分小文件，细粒度缓存 |
| Server Push 不生效 | Chrome 106+ 已移除 | 使用 preload / 103 Early Hints |
| HTTP/3 连接回退 HTTP/2 | 服务端/CDN 未配置 QUIC | 检查 Alt-Svc 头和 QUIC 监听 |
| HTTP/2 下雪崩效应 | 多路复用单连接，服务端慢响应影响所有流 | 设置合理的流优先级 + 超时 |
| QUIC 被 UDP 防火墙拦截 | 企业防火墙可能封锁 UDP | 回退到 HTTP/2 over TCP |
| 0-RTT 重放攻击 | 首包数据可能被重放 | 服务端限流 + 非幂等操作不用 0-RTT |

### 最佳实践

- 合并域名：HTTP/2 下单域名 + CDN 优于多域名分片
- 拆分资源：小文件细粒度缓存优于大文件合并
- 关键资源用 `<link rel="preload">` 预加载，不用 Server Push
- 启用 HTTP/3（QUIC）减少握手延迟和队头阻塞
- 配置 `Alt-Svc` 头让浏览器自动升级到 HTTP/3
- 0-RTT 只用于幂等请求（GET），非幂等请求（POST）走 1-RTT
- 弱网场景 HTTP/3 优势更明显，优先启用
- 监控 `nextHopProtocol` 指标，确认用户实际使用的协议版本
- 使用 `fetch priority` API 为关键请求设置高优先级
- Service Worker 预缓存关键资源，离线/弱网兜底

## 面试题

**Q1: HTTP/2 的多路复用是怎么实现的？和 HTTP/1.1 的长连接有什么区别？**
> HTTP/2 多路复用通过 Stream + Frame 实现：一个 TCP 连接上承载多个 Stream（用 Stream ID 标识），每个 Stream 承载一对请求/响应，数据被拆分为 Frame（HEADERS/DATA 等）交错传输。HTTP/1.1 长连接（Keep-Alive）虽然复用 TCP 连接，但必须串行处理：前一个响应完成后才能发送下一个请求，否则会产生队头阻塞。关键区别：HTTP/1.1 长连接 = 单车道轮流通行；HTTP/2 多路复用 = 多车道同时通行。HTTP/2 还支持流优先级，浏览器可指定关键资源优先加载。

**Q2: HTTP/2 为什么还有队头阻塞？HTTP/3 怎么解决的？**
> HTTP/2 解决了应用层（HTTP）的队头阻塞，但 TCP 层仍然存在：当一个 TCP 包丢失时，TCP 协议要求按序交付，后续已到达的包必须在缓冲区等待重传完成，导致该连接上所有 Stream 都被阻塞。HTTP/3 基于 QUIC（UDP），每个 Stream 独立拥塞控制：Stream A 的包丢失只阻塞 Stream A 的数据组装，Stream B/C 不受影响，可以继续读取已到达的数据。本质区别：TCP 是"一损俱损"，QUIC 是"各管各的"。

**Q3: QUIC 协议为什么选择 UDP 而不是改进 TCP？**
> 三个原因：① TCP 的中间设备固化：NAT/防火墙/路由器对 TCP 行为有固定预期，修改 TCP 协议（如拥塞控制算法）会被中间设备干扰或丢弃，而 UDP 通常直接放行；② 内核升级困难：TCP 实现在操作系统内核中，全球升级需数年，QUIC 在用户态实现，应用可自行升级；③ 灵活性：QUIC 在 UDP 之上实现了可靠的传输（重传、拥塞控制、流量控制），同时可以自由添加 TCP 没有的特性（连接迁移、0-RTT、独立流控）。代价：UDP 在某些企业网络被封锁，QUIC 需要回退到 TCP。

**Q4: HTTP/3 的 0-RTT 是什么？有什么安全风险？**
> 0-RTT（Zero Round Trip Time）重连优化：客户端首次连接完成握手后缓存服务端的传输参数，重连时首包直接携带加密的应用数据，无需等待握手完成。过程：首次 1-RTT 握手 → 缓存 Session Ticket → 重连时 0-RTT 发送请求。安全风险：**重放攻击** — 攻击者可截获 0-RTT 数据并重放，因为服务端无法区分是首次还是重放。缓解：① 0-RTT 只用于幂等请求（GET/HEAD），不用于状态修改（POST/PUT）；② 服务端限制 0-RTT 的有效期（Ticket 有效期）；③ 服务端对 0-RTT 请求做幂等性校验。

**Q5: HTTP/2 下前端资源合并策略应该怎么调整？**
> HTTP/1.1 时代：合并文件减少请求（1 个大 bundle 优于 10 个小文件），因为每个请求需要独立的 TCP 连接/排队。HTTP/2 时代：拆分文件更好 — ① 缓存粒度更细：app.js 频繁更新只需重下载 app.js，vendor.js 缓存命中；② 多路复用下多个小请求开销极低；③ 优先级控制更精确：关键 JS 可高优先级加载，非关键的低优先级。但不要过度拆分：每个文件有 HTTP 头部开销和解析成本，一般按功能模块拆分（vendor-react、vendor-utils、app、route-xxx），单个文件 50-200KB 为宜。

**Q6: HTTP/2 Server Push 为什么被弃用？替代方案是什么？**
> Server Push 被弃用的原因：① 推送过度：服务端难以准确判断客户端是否需要该资源（可能已有缓存），导致带宽浪费；② 优先级冲突：推送的资源可能抢占客户端主动请求的关键资源；③ 调试困难：推送资源在 DevTools 中不直观；④ 浏览器差异：各浏览器实现不一致，Chrome 106 移除支持。替代方案：① `<link rel="preload">`：客户端声明式预加载，浏览器自行决策（最推荐）；② `103 Early Hints`：服务端在正式响应前发送预加载提示；③ Service Worker 预缓存：离线缓存关键资源；④ 内联关键 CSS/JS：首屏关键资源直接嵌入 HTML。

**Q7: QUIC 的连接迁移是怎么工作的？**
> TCP 用四元组（源IP、源端口、目标IP、目标端口）标识连接，网络切换（WiFi→4G）时 IP 变化，TCP 连接必须断开重建。QUIC 用 Connection ID（CID）标识连接，CID 由客户端生成，不依赖 IP/端口。工作流程：① 客户端连接时生成 CID 包含在 QUIC 包中；② 网络切换后，客户端从新 IP 发送包含相同 CID 的 QUIC 包；③ 服务端根据 CID 识别这是已有连接，无需重新握手，直接继续传输。优势：移动设备上 WiFi↔蜂窝切换无缝衔接，下载/通话不断。限制：CID 长度有限，且网络切换可能导致路径 MTU 变化，需要重新探测。

**Q8: 如何判断和监控网站用户使用的 HTTP 协议版本？**
> 三种方式：① Performance API：`performance.getEntriesByType('resource')` 获取每个资源的 `nextHopProtocol`，值为 `h2`（HTTP/2）、`h3`（HTTP/3）、`http/1.1`；② Navigation Timing：`performance.getEntriesByType('navigation')[0].nextHopProtocol` 获取主文档的协议版本；③ 服务端日志：Nginx 的 `$protocol` 变量或 `$server_protocol` 记录协商的协议。监控建议：定期上报协议分布（h1/h2/h3 占比），关注 HTTP/3 升级率和回退率，按地区/运营商分析 QUIC 可用性，发现 UDP 被封的网段。

---

**相关链接：**
- [[HTTP与缓存策略]]
- [[WebSocket与实时通信]]
- [[前端性能优化]]
- [[前端性能监控体系]]
- [[浏览器渲染原理]]

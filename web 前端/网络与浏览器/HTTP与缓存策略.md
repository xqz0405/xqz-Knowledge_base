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

**核心概念：**

- **HTTP/1.1 → HTTP/2 → HTTP/3**：协议演进（多路复用、头部压缩、QUIC）
- **强缓存**：`Cache-Control` / `Expires`，不发请求直接用本地缓存
- **协商缓存**：`ETag` / `Last-Modified`，发请求验证资源是否变化
- **状态码**：`200`（成功）、`301/302`（重定向）、`304`（协商缓存命中）、`401`（未认证）、`403`（禁止）、`500`（服务端错误）

**关键特性：**

- `Cache-Control: max-age=3600` 强缓存 1 小时
- `Cache-Control: no-cache` 跳过强缓存，走协商缓存
- `Cache-Control: no-store` 完全不缓存
- HTTP/2 多路复用解决队头阻塞，单连接并行请求

## Why — 为什么

**适用场景：**

- 静态资源缓存（JS/CSS/图片）
- API 响应缓存
- 性能优化

**对比方案：**

| 维度 | 强缓存 | 协商缓存 | Service Worker |
|------|--------|---------|---------------|
| 请求次数 | 0 | 1（验证请求） | 0（可拦截） |
| 时效性 | 过期前不更新 | 每次验证 | 代码控制 |
| 适用资源 | 带 hash 的静态资源 | HTML/频繁更新的资源 | 离线/复杂策略 |

**优缺点：**

- ✅ 优点：
  - 强缓存零请求，极致性能
  - 协商缓存保证时效性
  - 分层策略覆盖不同资源类型
- ❌ 缺点：
  - 缓存策略配置不当导致更新不及时
  - 用户强制刷新绕过缓存
  - CDN 缓存与源站不一致

## How — 怎么用

### 快速上手

**静态资源缓存策略（推荐）：**

```
# HTML 文件：不缓存或短缓存（确保用户拿到最新版）
Cache-Control: no-cache（配合 ETag 走协商缓存）

# JS/CSS/图片（带 hash 文件名）：长期强缓存
Cache-Control: public, max-age=31536000, immutable
# 文件名含 hash，内容变了文件名也变，缓存自动失效
```

### 代码示例

**完整缓存流程：**

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

**Nginx 缓存配置：**

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
```

**HTTP/2 关键特性：**

```
# 多路复用：一个 TCP 连接并行多个请求
# 头部压缩：HPACK 算法压缩重复 Header
# 服务器推送：主动推送关联资源
# 优先级：请求可设置权重和依赖
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 发版后用户看到旧页面 | HTML 被强缓存，引用旧 JS | HTML 不强缓存，JS 文件名带 hash |
| CDN 缓存不更新 | CDN 回源策略太长 | CDN 刷新 API 或缩短 TTL |
| 304 太多影响性能 | 协商缓存每次都发请求 | 静态资源用 hash + 长期强缓存 |
| HTTP/2 推送浪费带宽 | 推送了用户已有的资源 | 保守使用推送，优先用 `<link rel="preload">` |

### 最佳实践

- HTML：`no-cache`（协商缓存）
- 带 hash 的 JS/CSS/图片：`max-age=31536000, immutable`
- API 响应：按业务设置，通常 `no-store` 或短缓存
- 优先用 HTTP/2，避免域分片

## 面试题

**Q1: HTTP/1.1 和 HTTP/2 的主要区别是什么？**
> HTTP/2 核心改进：多路复用（单连接并行请求，解决队头阻塞）、头部压缩（HPACK 算法）、服务器推送、请求优先级。HTTP/1.1 每个请求需单独 TCP 连接或管线化（有队头阻塞问题）。

**Q2: 强缓存和协商缓存的区别是什么？各自用什么 Header 控制？**
> 强缓存：不发请求，直接用本地副本，由 `Cache-Control`（`max-age`）和 `Expires` 控制；协商缓存：发请求验证资源是否变化，由 `ETag`/`If-None-Match` 和 `Last-Modified`/`If-Modified-Since` 控制，命中返回 304。强缓存优先级高于协商缓存。

**Q3: ETag 的原理是什么？强 ETag 和弱 ETag 有什么区别？**
> ETag 是服务器为资源生成的唯一标识符（通常基于内容哈希）。请求时客户端通过 `If-None-Match` 携带上次 ETag，服务器比对决定返回 304 还是新资源。强 ETag（默认）要求资源字节级一致才相同；弱 ETag（`W/"..."`）语义等价即可，适用于可接受轻微差异的场景。

**Q4: HTTPS 的握手过程是怎样的？**
> TLS 1.2：客户端发送 ClientHello（支持的加密套件、随机数）→ 服务端回 ServerHello（选定套件、证书、随机数）→ 客户端验证证书，生成预主密钥，用服务端公钥加密发送 → 双方基于三个随机数生成会话密钥 → 完成加密通信。TLS 1.3 简化为 1-RTT，并移除了不安全的加密算法。

---

**相关链接：**
- [[浏览器渲染原理]]
- [[跨域解决方案]]
- [[Webpack与Vite]]

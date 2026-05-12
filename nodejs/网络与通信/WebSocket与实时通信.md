---
tags:
  - Node.js
  - WebSocket
  - Socket.IO
  - 实时通信
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# WebSocket 与实时通信

## What — 是什么

> WebSocket 是全双工通信协议，允许服务端主动向客户端推送数据。Socket.IO 是最流行的 WebSocket 库，提供房间、命名空间、自动降级、心跳检测等高级功能。

**核心概念：**

- **ws**：Node.js 最轻量的 WebSocket 库，性能高，API 简洁
- **Socket.IO**：功能丰富的实时通信框架，自动降级（WebSocket→HTTP 长轮询）、房间、命名空间
- **房间（Room）**：Socket.IO 的分组机制，向一组连接广播消息（如聊天室、游戏房间）
- **命名空间（Namespace）**：Socket.IO 的逻辑隔离，同一端口多个独立通信通道
- **心跳检测**：定期 ping/pong 确认连接存活，超时自动断开
- **二进制帧**：WebSocket 支持传输 ArrayBuffer/Blob，用于音视频、文件传输

**核心架构：**

- 设计理念：ws 追求极致性能和简洁，Socket.IO 追求功能完善和兼容性
- 核心模块：**连接管理**（握手/升级/心跳）、**消息路由**（房间/命名空间）、**降级策略**（WebSocket→Long Polling）
- 数据流：客户端连接 → HTTP Upgrade 握手 → WebSocket 全双工通道 → 双向消息帧 → 心跳维持

**插件生态：**

- Socket.IO 官方：`socket.io-redis`（集群适配器）、`socket.io-emitter`（外部广播）
- 社区：`socket.io-sticky`（集群会话粘滞）、`@socket.io/admin-ui`（管理界面）

## Why — 为什么

**适用场景：**

- 实时聊天：多房间、消息广播、在线状态
- 协作编辑：多人实时编辑文档/白板
- 实时数据推送：股票行情、体育比分、通知
- 游戏：多人在线游戏状态同步
- IoT：设备状态监控和远程控制

**对比实时通信方案：**

| 维度 | ws | Socket.IO | Server-Sent Events | HTTP 长轮询 |
|------|-----|-----------|-------------------|------------|
| 通信方向 | 全双工 | 全双工 | 服务端→客户端 | 客户端发起 |
| 协议 | WebSocket | WebSocket + 降级 | HTTP | HTTP |
| 兼容性 | 现代浏览器 | 全浏览器（自动降级） | 现代浏览器 | 全浏览器 |
| 功能 | 基础 | 房间/命名空间/ACK | 仅推送 | 无 |
| 性能 | 最高 | 中（协议开销） | 高 | 低 |
| 复杂度 | 低 | 中 | 低 | 高 |

**优缺点：**

- ✅ 优点：
  - 全双工通信，实时性最好
  - ws 性能极高，内存占用低
  - Socket.IO 功能完善，自动降级
  - 支持二进制数据传输
- ❌ 缺点：
  - Socket.IO 协议与原生 WebSocket 不兼容
  - 长连接占用服务器资源
  - 集群模式需 Redis 适配器
  - 网络环境差时连接不稳定

## How — 怎么用

### 快速上手

```javascript
// ws 基础
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    ws.on('message', (data) => {
        ws.send(`Echo: ${data}`);
    });
});

// Socket.IO 基础
const { Server } = require('socket.io');
const io = new Server(3000, { cors: { origin: '*' } });

io.on('connection', (socket) => {
    console.log(`用户 ${socket.id} 连接`);
    socket.on('chat:message', (msg) => {
        io.emit('chat:message', msg); // 广播
    });
    socket.on('disconnect', () => console.log(`用户 ${socket.id} 断开`));
});
```

### 代码示例

**Socket.IO 聊天室：**

```javascript
const { Server } = require('socket.io');

const io = new Server(httpServer, {
    cors: { origin: 'http://localhost:5173' },
    pingInterval: 25000,
    pingTimeout: 20000
});

// 认证中间件
io.use(async (socket, next) => {
    const token = socket.handshake.auth.token;
    try {
        const user = await verifyToken(token);
        socket.user = user;
        next();
    } catch (err) {
        next(new Error('Authentication failed'));
    }
});

io.on('connection', (socket) => {
    // 加入房间
    socket.on('room:join', (roomId) => {
        socket.join(roomId);
        socket.to(roomId).emit('user:joined', { userId: socket.user.id, name: socket.user.name });
    });

    // 房间消息
    socket.on('room:message', ({ roomId, content }) => {
        const message = { id: Date.now(), userId: socket.user.id, content, createdAt: new Date() };
        io.to(roomId).emit('room:message', message);
    });

    // 离开房间
    socket.on('room:leave', (roomId) => {
        socket.leave(roomId);
        socket.to(roomId).emit('user:left', { userId: socket.user.id });
    });

    // 私信
    socket.on('dm:send', ({ to, content }) => {
        socket.to(to).emit('dm:receive', { from: socket.user.id, content });
    });

    // 正在输入
    socket.on('typing:start', (roomId) => {
        socket.to(roomId).emit('typing:update', { userId: socket.user.id, typing: true });
    });
    socket.on('typing:stop', (roomId) => {
        socket.to(roomId).emit('typing:update', { userId: socket.user.id, typing: false });
    });

    // 断开连接
    socket.on('disconnect', (reason) => {
        console.log(`${socket.user.name} 断开: ${reason}`);
    });
});
```

**ws + 心跳 + 重连：**

```javascript
// 服务端
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

const clients = new Map();

wss.on('connection', (ws, req) => {
    const clientId = Date.now().toString(36);
    clients.set(clientId, { ws, isAlive: true, lastPing: Date.now() });

    ws.on('pong', () => {
        const client = clients.get(clientId);
        if (client) { client.isAlive = true; client.lastPing = Date.now(); }
    });

    ws.on('message', (data) => {
        try {
            const msg = JSON.parse(data);
            handleMessage(clientId, msg);
        } catch {}
    });

    ws.on('close', () => clients.delete(clientId));
});

// 心跳检测
const heartbeat = setInterval(() => {
    wss.clients.forEach((ws) => {
        if (!ws.isAlive) return ws.terminate();
        ws.isAlive = false;
        ws.ping();
    });
}, 30000);

wss.on('close', () => clearInterval(heartbeat));

// 客户端重连逻辑
class ReconnectingWebSocket {
    constructor(url, options = {}) {
        this.url = url;
        this.reconnectInterval = options.reconnectInterval || 1000;
        this.maxReconnectInterval = options.maxReconnectInterval || 30000;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = options.maxReconnectAttempts || Infinity;
        this.connect();
    }

    connect() {
        this.ws = new WebSocket(this.url);
        this.ws.onopen = () => { this.reconnectAttempts = 0; this.onopen?.(); };
        this.ws.onmessage = (e) => this.onmessage?.(e);
        this.ws.onclose = () => this.reconnect();
        this.ws.onerror = () => {};
    }

    reconnect() {
        if (this.reconnectAttempts >= this.maxReconnectAttempts) return;
        const delay = Math.min(this.reconnectInterval * 2 ** this.reconnectAttempts, this.maxReconnectInterval);
        this.reconnectAttempts++;
        setTimeout(() => this.connect(), delay);
    }

    send(data) { this.ws?.send(data); }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Socket.IO 客户端连不上 ws 服务 | 协议不兼容 | 客户端必须用 socket.io-client |
| 集群消息不互通 | 多进程内存独立 | 用 `socket.io-redis` 适配器 |
| 连接频繁断开重连 | 心跳超时或网络不稳定 | 调整 `pingInterval`/`pingTimeout` |
| 内存泄漏 | 未清理断开连接的引用 | `disconnect` 时从 Map 中删除引用 |
| 跨域被拒 | 未配置 CORS | `new Server(httpServer, { cors: {...} })` |
| 消息丢失 | 连接不稳定时发送 | 用 ACK 确认机制，超时重发 |

### 最佳实践

- 生产环境使用心跳检测（ws 手动 / Socket.IO 内置）
- 集群模式使用 Redis 适配器实现跨进程广播
- 客户端实现指数退避重连
- 使用房间分组，避免全局广播浪费带宽
- 消息加 ACK 确认机制，防止关键消息丢失
- 连接数监控，设置上限防止资源耗尽

## 面试题

**Q1: WebSocket 与 HTTP 的区别？**
> 核心区别：① HTTP 是请求-响应模式（客户端发起），WebSocket 是全双工（双方随时发送）；② HTTP 每次请求需携带 Header（数百字节），WebSocket 握手后帧头仅 2-14 字节；③ HTTP 短连接（HTTP/1.1 Keep-Alive 复用连接），WebSocket 长连接持久化；④ WebSocket 通过 HTTP Upgrade 握手升级协议。适用场景：WebSocket 适合实时推送/双向通信，HTTP 适合请求-响应/资源获取。

**Q2: Socket.IO 与原生 WebSocket 的区别？**
> 核心区别：① Socket.IO 自动降级——WebSocket 不可用时降级为 HTTP 长轮询，原生 ws 不行；② Socket.IO 有房间和命名空间——逻辑分组和隔离，原生 ws 需自行实现；③ Socket.IO 有 ACK 机制——发送消息后等待确认，原生 ws 无；④ Socket.IO 协议与 WebSocket 不兼容——必须用 socket.io-client，不能用浏览器原生 WebSocket；⑤ Socket.IO 有自动重连——客户端断开后自动重连，原生 ws 需手动实现；⑥ ws 性能更高，Socket.IO 功能更全。

**Q3: 心跳检测的原理是什么？为什么需要？**
> 原理：服务端定期向客户端发送 ping 帧，客户端自动回复 pong 帧。如果超时未收到 pong，认为连接已断开，关闭并清理资源。ws 中需手动实现（`ws.ping()` + 监听 `pong`），Socket.IO 内置（`pingInterval`/`pingTimeout` 配置）。必要性：① 网络中间件（NAT/代理/负载均衡）会关闭空闲连接，心跳保持连接活跃；② 客户端异常关闭（崩溃/断网）不会发送 close 帧，服务端需心跳检测才能发现；③ 清理僵尸连接释放服务器资源。

**Q4: Socket.IO 房间（Room）的实现原理？**
> 房间是 Socket.IO 的逻辑分组机制。实现原理：服务端维护一个 `Map<roomId, Set<socketId>>` 映射表。`socket.join(roomId)` 将 socket ID 加入对应房间的 Set；`socket.leave(roomId)` 移除；`io.to(roomId).emit()` 遍历房间内所有 socket ID 发送消息。房间不是持久化的——服务端重启后房间丢失。集群模式下，房间信息通过 Redis 适配器在进程间同步，广播消息通过 Redis Pub/Sub 跨进程传递。

**Q5: 如何在集群模式下使用 Socket.IO？**
> 集群模式下多个工作进程各自维护独立的连接，进程间消息需要通过 Redis 适配器同步。步骤：① 安装 `@socket.io/redis-adapter` + `redis`；② 创建两个 Redis 客户端（Pub 和 Sub）；③ `io.adapter(createAdapter(pubClient, subClient))`；④ 广播/房间消息自动通过 Redis Pub/Sub 跨进程传递。还需处理会话粘滞（Sticky Session）：同一客户端的请求始终路由到同一工作进程，可通过 cookie-based 路由或 `socket.io-sticky` 中间件实现。

**Q6: WebSocket 的安全问题有哪些？如何防范？**
> 安全问题：① 跨站 WebSocket 劫持（CSWSH）——恶意网页连接受害者的 WebSocket，防范：验证 `Origin` Header；② 未认证连接——任何人可连接，防范：在 `io.use()` 中间件验证 token；③ 注入攻击——消息内容未过滤，防范：服务端验证和清洗消息；④ DDoS——大量连接耗尽资源，防范：连接数限制 + 限流。生产环境必须使用 WSS（WebSocket over TLS），配置 CORS 白名单，验证连接身份。

**Q7: SSE（Server-Sent Events）与 WebSocket 如何选择？**
> SSE 适合：① 只需服务端→客户端推送（通知/股票行情/日志流）；② 需要自动重连（SSE 内置）；③ 需要标准 HTTP 协议（易穿透代理/防火墙）；④ 简单场景不想引入 WebSocket 复杂度。WebSocket 适合：① 双向实时通信（聊天/游戏/协作）；② 需要传输二进制数据；③ 高频消息（SSE 每次请求有 HTTP 开销）。经验法则：单向推送用 SSE，双向通信用 WebSocket。

**Q8: 如何保证 WebSocket 消息的可靠性？**
> WebSocket 基于TCP，保证帧的有序到达但不保证业务层可靠。保证方式：① ACK 确认——发送消息后等待确认回执，超时重发；Socket.IO 的 callback 参数就是 ACK；② 消息序号——每条消息带递增 ID，接收方检测丢消息请求重发；③ 消息持久化——关键消息先写数据库，连接恢复后按序号拉取；④ 幂等设计——同一条消息处理多次结果一致。折中方案：普通消息发后即忘，关键消息（支付/交易）用 ACK + 持久化。

---

**相关链接：**
- [[HTTP服务构建]]
- [[事件循环]]
- [[消息队列与事件驱动]]
- Socket.IO 文档：https://socket.io/

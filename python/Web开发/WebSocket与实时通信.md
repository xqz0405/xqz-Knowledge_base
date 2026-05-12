---
tags:
  - Python
  - WebSocket
  - 实时通信
  - SSE
  - Socket.IO
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# WebSocket与实时通信

## What — 是什么

> WebSocket 是全双工通信协议，在单个 TCP 连接上实现客户端与服务器的实时双向数据传输。Python 生态中，FastAPI 原生支持 WebSocket，Django 通过 Channels 扩展，Flask 通过 Flask-SocketIO。实时通信还包括 SSE（Server-Sent Events）和长轮询等替代方案。

**核心概念：**

- **WebSocket**：全双工协议，一次握手后持久连接，双方随时发送数据
- **SSE**：服务端单向推送，基于 HTTP，比 WebSocket 轻量
- **Socket.IO**：WebSocket 的封装库，自动降级到长轮询，提供房间/广播等高级功能
- **ASGI**：异步服务器网关接口，Django Channels 和 FastAPI WebSocket 的底层

**关键特性：**

- WebSocket 协议开销极小（帧头 2-10 字节），适合高频消息
- 连接建立需要 HTTP 握手（Upgrade 请求），之后切换为 WebSocket 协议
- 支持文本和二进制帧，可传输 JSON/Protobuf/二进制数据
- 心跳机制（ping/pong）检测连接存活
- Socket.IO 提供自动重连、房间、命名空间等 WebSocket 缺失的功能

**实时通信方案对比：**

| 维度 | WebSocket | SSE | 长轮询 |
|------|-----------|-----|--------|
| 方向 | 双向 | 服务端→客户端 | 客户端→服务端 |
| 协议 | ws/wss | HTTP | HTTP |
| 连接 | 持久 | 持久 | 每次新建 |
| 延迟 | 极低 | 低 | 中 |
| 浏览器支持 | 广泛 | 广泛 | 全部 |
| 复杂度 | 中 | 低 | 低 |
| 适合场景 | 聊天/游戏/协作 | 通知/股票行情 | 兼容降级 |

**运行机制：**

- **握手**：客户端发送 `Upgrade: websocket` HTTP 请求，服务器返回 101 Switching
- **帧格式**：opcode（text/binary/ping/pong/close）+ payload
- **连接管理**：服务端维护连接池，处理断连/重连
- **背压（Backpressure）**：生产速度 > 消费速度时，需要流量控制

## Why — 为什么

**适用场景：**

- 即时聊天、在线协作
- 实时仪表盘、股票行情
- 多人游戏、实时投票
- IoT 设备数据推送
- 日志/监控实时流

**优缺点：**

- ✅ 优点：
  - 全双工，实时性极高
  - 协议开销小，适合高频消息
  - 支持二进制数据
- ❌ 缺点：
  - 连接管理复杂（断连/重连/心跳）
  - 无 HTTP 缓存/CDN 支持
  - 调试困难（浏览器 DevTools 支持不如 HTTP）
  - 水平扩展需要消息代理（Redis Pub/Sub）

## How — 怎么用

### 快速上手：FastAPI WebSocket

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            try:
                await connection.send_text(message)
            except:
                pass

manager = ConnectionManager()

@app.websocket("/ws/chat")
async def chat(websocket: WebSocket):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(data)
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        await manager.broadcast("有人离开了聊天")
```

### 代码示例1：房间与私聊

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from collections import defaultdict

app = FastAPI()

class RoomManager:
    def __init__(self):
        self.rooms: dict[str, list[WebSocket]] = defaultdict(list)
        self.user_room: dict[WebSocket, str] = {}

    async def join(self, room: str, ws: WebSocket):
        await ws.accept()
        self.rooms[room].append(ws)
        self.user_room[ws] = room
        await self.broadcast_to_room(room, f"用户加入了房间")

    def leave(self, ws: WebSocket):
        room = self.user_room.pop(ws, None)
        if room:
            self.rooms[room].remove(ws)
            if not self.rooms[room]:
                del self.rooms[room]

    async def broadcast_to_room(self, room: str, message: str):
        for ws in self.rooms.get(room, []):
            try:
                await ws.send_text(message)
            except:
                pass

    async def send_to_user(self, ws: WebSocket, message: str):
        try:
            await ws.send_text(message)
        except:
            pass

manager = RoomManager()

@app.websocket("/ws/chat/{room}")
async def room_chat(websocket: WebSocket, room: str):
    await manager.join(room, websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast_to_room(room, data)
    except WebSocketDisconnect:
        manager.leave(websocket)
```

### 代码示例2：Django Channels + Redis

```python
# pip install channels channels-redis

# settings.py
INSTALLED_APPS = [
    'daphne',       # ASGI 服务器
    'channels',
    ...
]
ASGI_APPLICATION = 'myproject.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {"hosts": [("127.0.0.1", 6379)]},
    },
}

# consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room']
        self.room_group = f'chat_{self.room_name}'

        await self.channel_layer.group_add(self.room_group, self.channel_name)
        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(self.room_group, self.channel_name)

    async def receive(self, text_data):
        data = json.loads(text_data)
        await self.channel_layer.group_send(
            self.room_group,
            {
                'type': 'chat_message',
                'message': data['message'],
                'username': data['username'],
            }
        )

    async def chat_message(self, event):
        await self.send(text_data=json.dumps({
            'message': event['message'],
            'username': event['username'],
        }))

# routing.py
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room>\w+)/$', consumers.ChatConsumer.as_asgi()),
]
```

### 代码示例3：SSE 服务端推送

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio
import json

app = FastAPI()

async def event_generator():
    """SSE 事件生成器"""
    count = 0
    while True:
        count += 1
        data = json.dumps({"count": count, "timestamp": asyncio.get_event_loop().time()})
        yield f"data: {data}\n\n"
        await asyncio.sleep(1)

@app.get("/sse/counter")
async def sse_counter():
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
            "X-Accel-Buffering": "no",  # Nginx 不缓冲
        }
    )

# 带事件类型和 ID 的 SSE
async def notification_generator():
    count = 0
    while True:
        count += 1
        yield f"id: {count}\nevent: notification\ndata: {{'msg': '通知 {count}'}}\n\n"
        await asyncio.sleep(5)

@app.get("/sse/notifications")
async def sse_notifications():
    return StreamingResponse(
        notification_generator(),
        media_type="text/event-stream",
    )
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 连接频繁断开 | 网络不稳定/代理超时 | 心跳机制（30s ping/pong） |
| 多实例消息不同步 | 多进程间连接不共享 | Redis Pub/Sub 做消息代理 |
| 内存泄漏 | 断连未清理连接池 | try/finally 确保 disconnect 清理 |
| Nginx 阻断 WebSocket | 默认配置不转发 Upgrade | 加 `proxy_set_header Upgrade` |
| SSE 被代理缓冲 | Nginx/CDN 缓冲 SSE 流 | `X-Accel-Buffering: no` |
| 二进制数据解析错误 | WebSocket 帧类型处理 | `receive_bytes()` 处理二进制帧 |

### 最佳实践

- 连接管理用专门类（ConnectionManager），统一 accept/disconnect/broadcast
- 生产环境必须用 Redis Pub/Sub 做消息代理（多实例同步）
- 心跳机制防止连接假死（30-60s 间隔）
- 断连自动重连（客户端指数退避重试）
- Nginx 配置 WebSocket 代理头
- 单向推送用 SSE（更简单），双向通信用 WebSocket

## 面试题

**Q1: WebSocket 和 HTTP 长轮询有什么区别？**
> HTTP 长轮询：客户端发请求，服务端不立即响应，等有数据时才返回。每次数据传输都需要新建 HTTP 请求/响应，开销大。WebSocket：一次握手建立持久连接，双方随时发送数据，帧头仅 2-10 字节，无重复 HTTP 头开销。WebSocket 延迟更低（无需反复建立连接）、带宽更省，但连接管理更复杂。长轮询是 WebSocket 不可用时的降级方案。

**Q2: 如何实现 WebSocket 的水平扩展？**
> 单实例 WebSocket 连接存在本机内存中，多实例间无法直接通信。解决方案：1) Redis Pub/Sub——实例收到消息后发布到 Redis 频道，其他实例订阅并转发给本地连接；2) 消息队列（RabbitMQ/Kafka）——类似 Redis 但更可靠；3) 分布式 KV（etcd/Consul）——服务发现 + 消息路由。Django Channels 用 `CHANNEL_LAYERS` 配置（channels-redis），FastAPI 需自建 Redis 发布订阅。

**Q3: SSE 和 WebSocket 如何选择？**
> SSE 适合服务端单向推送：通知、股票行情、日志流、进度更新。优势是简单（基于 HTTP）、自动重连、浏览器原生 `EventSource` API、支持 HTTP 缓存/CDN。WebSocket 适合双向实时通信：聊天、游戏、协作编辑。优势是全双工、低延迟、支持二进制。如果只需要服务端推数据，SSE 更简单可靠；需要客户端也发数据时，必须 WebSocket。

**Q4: Django Channels 的工作原理是什么？**
> Channels 将 Django 从同步 WSGI 扩展到异步 ASGI，支持 WebSocket/HTTP 长连接/后台任务。核心概念：Consumer（类似 View，处理 WebSocket 事件）、Channel Layer（消息传递层，Redis 实现）、Group（广播组）。流程：ASGI 服务器（Daphne）接收连接 → 路由到 Consumer → Consumer 通过 Channel Layer 与其他 Consumer 通信。Channel Layer 的 `group_send` 实现跨实例广播。

**Q5: WebSocket 连接的认证如何实现？**
> WebSocket 不支持自定义 HTTP 头（浏览器限制），认证方案：1) URL 参数传 token（`ws://host/ws?token=xxx`），在 accept 前验证；2) Cookie/Session（同域情况下浏览器自动携带）；3) 先 HTTP 验证获取 ticket，再用 ticket 建立 WebSocket。FastAPI 中在 `websocket.accept()` 之前验证 token，无效则 `await websocket.close(code=4001)`。不推荐在 URL 中传敏感 token（会被日志记录）。

**Q6: 如何处理 WebSocket 的背压（Backpressure）？**
> 生产速度 > 消费速度时，未发送的消息堆积在发送缓冲区，最终 OOM。解决方案：1) 监控发送缓冲区大小，超过阈值暂停生产或断开慢消费者；2) 消息队列 + 消费者速率控制；3) 丢弃旧消息（保留最新 N 条）；4) 限流（限制每秒消息数）。FastAPI 中 `await websocket.send_text()` 是异步的，发送缓冲区满时会 await 阻塞，需设置超时或监控。

**Q7: Socket.IO 和原生 WebSocket 有什么区别？**
> Socket.IO 是 WebSocket 的上层封装：1) 自动降级——WebSocket 不可用时降级到长轮询；2) 自动重连——断连后指数退避重试；3) 房间和命名空间——原生 WebSocket 无此概念；4) 事件系统——`emit('event', data)` 代替原始帧；5) 心跳和超时——自动管理。代价：协议不兼容原生 WebSocket，必须两端都用 Socket.IO 客户端。如果控制两端，Socket.IO 更方便；需要标准协议则用原生 WebSocket。

**Q8: 如何测试 WebSocket 应用？**
> 工具层面：1) Postman 支持 WebSocket 测试；2) `websocat` 命令行工具（`websocat ws://localhost:8000/ws`）；3) Python 的 `websockets` 库编写测试客户端；4) 浏览器 DevTools 的 Network → WS 面板查看帧数据。代码层面：FastAPI 用 `TestClient` 的 `websocket_connect` 上下文管理器；Django Channels 用 `ChannelsLiveServerTestCase`。测试要点：连接建立/断开、消息收发、广播、并发连接、异常恢复。

---

**相关链接：**
- [[FastAPI]]
- [[Flask]]
- [[异步编程]]
- [FastAPI WebSocket 文档](https://fastapi.tiangolo.com/advanced/websockets/)

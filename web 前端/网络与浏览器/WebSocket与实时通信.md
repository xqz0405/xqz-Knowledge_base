---
tags:
  - Web前端
  - WebSocket
  - SSE
  - 实时通信
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# WebSocket与实时通信

## What — 是什么

> WebSocket 是全双工持久连接协议，SSE（Server-Sent Events）是服务端单向推送协议，两者是 Web 实时通信的核心方案。

**WebSocket 核心概念：**

- **全双工通信**：客户端和服务端可同时收发消息
- **持久连接**：一次握手，长期保持，无需反复建连
- **协议升级**：HTTP → WS，通过 `Upgrade: websocket` 头完成
- **帧格式**：文本帧（JSON/字符串）和二进制帧（Blob/ArrayBuffer）

**SSE 核心概念：**

- **单向推送**：服务端 → 客户端，客户端无法通过同一连接发送
- **基于 HTTP**：普通 GET 请求，响应 `Content-Type: text/event-stream`
- **自动重连**：浏览器断线自动重连，`Last-Event-ID` 续传
- **事件格式**：`data:` + `event:` + `id:` 字段

**关键特性：**

- WebSocket 适合双向实时通信（聊天、协作、游戏）
- SSE 适合单向推送（通知、股票行情、日志流）
- 两者都比轮询高效得多

## Why — 为什么

**适用场景：**

- WebSocket：即时聊天、多人协作、在线游戏、实时仪表盘
- SSE：消息通知、实时日志、股票行情、AI 流式输出

**对比替代方案：**

| 维度 | WebSocket | SSE | 短轮询 | 长轮询 |
|------|-----------|-----|--------|--------|
| 方向 | 双向 | 单向（服务端→客户端） | 单向 | 单向 |
| 协议 | WS（独立协议） | HTTP | HTTP | HTTP |
| 实时性 | 极高（毫秒） | 高 | 低（延迟=轮询间隔） | 中 |
| 连接开销 | 低（持久） | 低（持久） | 高（频繁建连） | 中 |
| 断线重连 | 需手动实现 | 浏览器自动 | 不涉及 | 需手动 |
| 浏览器支持 | 全部 | 全部（IE 除外） | 全部 | 全部 |
| 代理/CDN | 可能不支持 | 兼容 HTTP | 兼容 | 兼容 |

**优缺点：**

- ✅ WebSocket 优点：
  - 全双工，延迟最低
  - 二进制数据支持好
- ❌ WebSocket 缺点：
  - 运维复杂（心跳、重连、状态管理）
  - 部分代理/防火墙不支持
- ✅ SSE 优点：
  - 基于 HTTP，兼容性好
  - 浏览器自动重连
  - 实现简单
- ❌ SSE 缺点：
  - 单向通信
  - 不支持二进制

## How — 怎么用

### WebSocket

**基础连接：**

```javascript
const ws = new WebSocket('wss://api.example.com/ws');

ws.onopen = () => {
    console.log('连接建立');
    ws.send(JSON.stringify({ type: 'join', room: 'general' }));
};

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('收到消息:', data);
};

ws.onerror = (error) => {
    console.error('连接错误:', error);
};

ws.onclose = (event) => {
    console.log('连接关闭:', event.code, event.reason);
};
```

**生产级封装（心跳 + 自动重连）：**

```javascript
class ReconnectingWebSocket {
    constructor(url, options = {}) {
        this.url = url;
        this.reconnectInterval = options.reconnectInterval ?? 3000;
        this.heartbeatInterval = options.heartbeatInterval ?? 30000;
        this.maxRetries = options.maxRetries ?? Infinity;
        this.retries = 0;
        this.ws = null;
        this.handlers = {};
        this.shouldReconnect = true;
    }

    connect() {
        this.ws = new WebSocket(this.url);

        this.ws.onopen = () => {
            this.retries = 0;
            this.emit('open');
            this.startHeartbeat();
        };

        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            if (data.type === 'pong') return; // 心跳响应，忽略
            this.emit('message', data);
        };

        this.ws.onclose = () => {
            this.stopHeartbeat();
            this.emit('close');
            if (this.shouldReconnect && this.retries < this.maxRetries) {
                this.retries++;
                setTimeout(() => this.connect(), this.reconnectInterval);
            }
        };

        this.ws.onerror = (error) => this.emit('error', error);
    }

    startHeartbeat() {
        this.heartbeatTimer = setInterval(() => {
            if (this.ws?.readyState === WebSocket.OPEN) {
                this.ws.send(JSON.stringify({ type: 'ping' }));
            }
        }, this.heartbeatInterval);
    }

    stopHeartbeat() {
        clearInterval(this.heartbeatTimer);
    }

    send(data) {
        this.ws?.send(JSON.stringify(data));
    }

    close() {
        this.shouldReconnect = false;
        this.ws?.close();
    }

    on(event, handler) {
        (this.handlers[event] ??= []).push(handler);
    }

    emit(event, ...args) {
        (this.handlers[event] || []).forEach(fn => fn(...args));
    }
}

// 使用
const ws = new ReconnectingWebSocket('wss://api.example.com/ws');
ws.on('message', (data) => updateChat(data));
ws.connect();
```

**React Hook 封装：**

```typescript
function useWebSocket(url: string) {
    const wsRef = useRef<ReconnectingWebSocket | null>(null);
    const [lastMessage, setLastMessage] = useState<any>(null);
    const [status, setStatus] = useState<'connecting' | 'open' | 'closed'>('connecting');

    useEffect(() => {
        const ws = new ReconnectingWebSocket(url);
        wsRef.current = ws;

        ws.on('open', () => setStatus('open'));
        ws.on('close', () => setStatus('closed'));
        ws.on('message', (data) => setLastMessage(data));

        ws.connect();
        return () => ws.close();
    }, [url]);

    const sendMessage = useCallback((data: any) => {
        wsRef.current?.send(data);
    }, []);

    return { lastMessage, status, sendMessage };
}
```

### SSE

**基础用法：**

```javascript
const eventSource = new EventSource('/api/stream');

eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('推送数据:', data);
};

eventSource.onerror = () => {
    console.log('连接断开，浏览器自动重连');
};

// 监听命名事件
eventSource.addEventListener('notification', (event) => {
    showToast(JSON.parse(event.data));
});

// 关闭
eventSource.close();
```

**AI 流式输出（SSE 典型场景）：**

```typescript
async function streamChat(messages: Message[]) {
    const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages }),
    });

    const reader = response.body!.getReader();
    const decoder = new TextDecoder();
    let fullText = '';

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        // 解析 SSE 格式：data: {...}\n\n
        const lines = chunk.split('\n').filter(l => l.startsWith('data: '));
        for (const line of lines) {
            const data = line.slice(6);
            if (data === '[DONE]') break;
            const parsed = JSON.parse(data);
            fullText += parsed.content;
            renderMarkdown(fullText); // 逐步渲染
        }
    }
}
```

**Vue Composable：**

```typescript
function useSSE(url: string) {
    const data = ref<any>(null);
    const status = ref<'connecting' | 'open' | 'closed'>('connecting');
    let eventSource: EventSource | null = null;

    function connect() {
        eventSource = new EventSource(url);
        status.value = 'connecting';

        eventSource.onopen = () => { status.value = 'open'; };
        eventSource.onmessage = (e) => { data.value = JSON.parse(e.data); };
        eventSource.onerror = () => { status.value = 'closed'; };
    }

    function close() {
        eventSource?.close();
        eventSource = null;
    }

    onMounted(connect);
    onUnmounted(close);

    return { data, status, connect, close };
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| WebSocket 频繁断连 | 代理超时或网络不稳定 | 心跳保活 + 自动重连 + 指数退避 |
| SSE 连接被代理缓冲 | Nginx 默认缓冲响应 | `proxy_buffering off` + `X-Accel-Buffering: no` |
| 消息丢失 | 断连期间服务端发送了消息 | 重连时用 `Last-Event-ID` 或时间戳补发 |
| 并发连接数限制 | 浏览器限制同域 6 个 | 复用连接，消息多路复用 |
| 乱码 | 二进制帧未正确处理 | 设置 `ws.binaryType = 'arraybuffer'` |

### 最佳实践

- 双向实时通信用 WebSocket，单向推送优先 SSE
- WebSocket 必须实现心跳保活 + 自动重连
- SSE 用于 AI 流式输出/通知推送，比 WebSocket 轻量
- 消息加 type 字段做路由，避免单连接逻辑混乱
- 断线重连后主动请求缺失数据，不要只依赖连接恢复

## 面试题

**Q1: WebSocket 和 HTTP 的区别是什么？**
> HTTP 是请求-响应模式，每次通信需客户端发起，单向且短连接（HTTP/1.1 keep-alive 可复用但仍为请求-响应）；WebSocket 通过 HTTP 升级握手建立持久连接，之后双方可随时收发数据，是真正的全双工通信。WebSocket 开销更低（无重复 Header），实时性更高。

**Q2: SSE 适用于什么场景？和 WebSocket 如何选择？**
> SSE 适用于服务端单向推送场景：消息通知、股票行情、AI 流式输出、实时日志。选择依据：双向通信用 WebSocket（聊天、协作、游戏），仅服务端推送用 SSE 更轻量。SSE 基于标准 HTTP、浏览器自动重连、实现简单；WebSocket 需手动处理心跳重连，但支持双向和二进制数据。

**Q3: WebSocket 心跳机制的目的是什么？如何实现？**
> 心跳机制用于检测连接是否存活，防止代理/防火墙因长时间无数据传输而关闭连接（空闲超时），同时及时发现断线以触发重连。实现方式：客户端定时（如 30s）发送 `ping` 消息，服务端回复 `pong`；若超时未收到响应则判定断连，触发自动重连。

**Q4: WebSocket 断线重连策略有哪些？**
> 常见策略：固定间隔重连（简单但可能加剧服务器压力）、指数退避重连（1s → 2s → 4s → 8s，逐渐增大间隔，推荐）、限制最大重试次数（避免无限重连）。重连成功后应主动请求缺失数据（通过时间戳或序号），而非仅依赖连接恢复。还需注意服务端可能需要消息缓冲机制以补发断线期间的消息。

---

**相关链接：**
- [[HTTP与缓存策略]]
- [[Cookie与认证]]
- [[跨域解决方案]]

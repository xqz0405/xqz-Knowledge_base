---
tags:
  - Python
  - WSGI
  - ASGI
  - 中间件
  - 请求处理
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Web中间件与请求处理

## What — 是什么

> Web 中间件是请求/响应处理链中的拦截层，在视图逻辑前后执行通用逻辑（认证、日志、CORS、限流等）。Python Web 生态有两个服务器网关协议：WSGI（同步）和 ASGI（异步），中间件机制也因框架而异。理解中间件模型是掌握 Python Web 请求处理的关键。

**核心概念：**

- **WSGI**：Web Server Gateway Interface，Python 同步 Web 的标准协议（PEP 3333）
- **ASGI**：Asynchronous Server Gateway Interface，WSGI 的异步扩展
- **中间件**：请求/响应的拦截器链，按顺序执行前处理，逆序执行后处理
- **请求上下文**：请求生命周期内的状态容器

**关键特性：**

- WSGI 中间件是可调用对象（callable），包装 app 形成洋葱模型
- ASGI 中间件支持异步，可处理 WebSocket 和长连接
- Django 中间件基于钩子方法（`process_request`/`process_response`）
- FastAPI/Starlette 中间件基于 ASGI 协议，纯异步
- Flask 使用 `before_request`/`after_request` 钩子而非中间件类

**请求处理流程：**

```
HTTP 请求
  ↓
WSGI/ASGI 服务器（Gunicorn/Uvicorn）
  ↓
中间件链（洋葱模型）
  ↓ 外层 → 内层
  [CORS] → [Auth] → [Logging] → [App]
  ↓ 内层 → 外层
  [CORS] ← [Auth] ← [Logging] ← [Response]
  ↓
HTTP 响应
```

**WSGI vs ASGI 对比：**

| 维度 | WSGI | ASGI |
|------|------|------|
| 协议 | 同步 | 异步 |
| 请求类型 | HTTP | HTTP + WebSocket + 长轮询 |
| 接口 | `app(environ, start_response)` | `async app(scope, receive, send)` |
| 服务器 | Gunicorn/uWSGI | Uvicorn/Daphne/Hypercorn |
| 并发模型 | 多线程/多进程 | 事件循环 + 协程 |
| 框架 | Flask/Django（传统） | FastAPI/Django 3.1+/Starlette |

## Why — 为什么

**适用场景：**

- 认证与授权（JWT/Session 验证）
- 跨域处理（CORS）
- 请求日志与性能监控
- 限流（Rate Limiting）
- 请求 ID 追踪（分布式追踪）
- 响应压缩（Gzip/Brotli）

**各框架中间件机制对比：**

| 框架 | 机制 | 特点 |
|------|------|------|
| FastAPI | ASGI 中间件 + 依赖注入 | 纯异步，Starlette 基础 |
| Django | Middleware 类 | 钩子方法，同步+异步 |
| Flask | before/after_request | 函数式钩子，非标准中间件 |

**优缺点：**

- ✅ 优点：
  - 横切关注点与业务逻辑解耦
  - 可插拔，按需组合
  - 统一处理认证/日志/CORS 等
- ❌ 缺点：
  - 执行顺序需要仔细管理
  - 过多中间件增加请求延迟
  - 调试困难（多层包装）

## How — 怎么用

### 快速上手

```python
# FastAPI 中间件
from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware
import time

app = FastAPI()

# 内置 CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 自定义中间件
@app.middleware("http")
async def timing_middleware(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    elapsed = time.time() - start
    response.headers["X-Process-Time"] = f"{elapsed:.3f}s"
    return response

@app.get("/api/data")
async def get_data():
    return {"data": "hello"}
```

### 代码示例1：FastAPI 完整中间件链

```python
from fastapi import FastAPI, Request, Response, HTTPException
from fastapi.middleware.gzip import GZipMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
import time
import uuid
import logging

logger = logging.getLogger("api")

app = FastAPI()

# Gzip 压缩
app.add_middleware(GZipMiddleware, minimum_size=1000)

# 请求 ID 追踪
class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response

# 认证中间件
class AuthMiddleware(BaseHTTPMiddleware):
    PUBLIC_PATHS = {"/docs", "/openapi.json", "/health"}

    async def dispatch(self, request: Request, call_next):
        if request.url.path in self.PUBLIC_PATHS:
            return await call_next(request)

        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return Response(
                content='{"detail":"未提供认证令牌"}',
                status_code=401,
                media_type="application/json"
            )

        try:
            # JWT 验证逻辑
            user = verify_token(token)
            request.state.user = user
        except Exception:
            return Response(
                content='{"detail":"认证令牌无效"}',
                status_code=401,
                media_type="application/json"
            )

        return await call_next(request)

# 限流中间件
class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, max_requests=100, window_seconds=60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window = window_seconds
        self.requests: dict[str, list[float]] = {}

    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        now = time.time()

        if client_ip not in self.requests:
            self.requests[client_ip] = []

        # 清理过期记录
        self.requests[client_ip] = [
            t for t in self.requests[client_ip]
            if now - t < self.window
        ]

        if len(self.requests[client_ip]) >= self.max_requests:
            return Response(
                content='{"detail":"请求过于频繁"}',
                status_code=429,
                media_type="application/json"
            )

        self.requests[client_ip].append(now)
        return await call_next(request)

# 注册中间件（注意顺序：最后注册的最先执行）
app.add_middleware(RateLimitMiddleware, max_requests=100)
app.add_middleware(AuthMiddleware)
app.add_middleware(RequestIDMiddleware)
```

### 代码示例2：Django 中间件

```python
# middleware.py
import time
import uuid
import logging

logger = logging.getLogger(__name__)

class RequestIDMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        request.request_id = request.headers.get(
            'X-Request-ID', str(uuid.uuid4())
        )
        response = self.get_response(request)
        response['X-Request-ID'] = request.request_id
        return response

class TimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start = time.time()
        response = self.get_response(request)
        elapsed = time.time() - start
        response['X-Response-Time'] = f"{elapsed:.3f}s"

        if elapsed > 1.0:
            logger.warning(
                f"慢请求: {request.method} {request.path} "
                f"耗时 {elapsed:.3f}s [rid={request.request_id}]"
            )
        return response

class ExceptionLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        return self.get_response(request)

    def process_exception(self, request, exception):
        logger.error(
            f"未捕获异常: {request.method} {request.path} "
            f"[rid={getattr(request, 'request_id', 'N/A')}]",
            exc_info=exception
        )
        return None  # 返回 None 让 Django 默认错误处理继续

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'myapp.middleware.RequestIDMiddleware',
    'myapp.middleware.TimingMiddleware',
    'myapp.middleware.ExceptionLoggingMiddleware',
]
```

### 代码示例3：ASGI 中间件与 Flask 钩子

```python
# 纯 ASGI 中间件（不依赖框架）
class ASGICORSMiddleware:
    def __init__(self, app, allow_origins=None):
        self.app = app
        self.allow_origins = allow_origins or ["*"]

    async def __call__(self, scope, receive, send):
        if scope["type"] == "http":
            origin = None
            for name, value in scope.get("headers", []):
                if name == b"origin":
                    origin = value.decode()
                    break

            async def send_with_cors(message):
                if message["type"] == "http.response.start":
                    headers = dict(message.get("headers", []))
                    if origin and (
                        "*" in self.allow_origins
                        or origin in self.allow_origins
                    ):
                        headers[b"access-control-allow-origin"] = origin.encode()
                        headers[b"access-control-allow-credentials"] = b"true"
                    message["headers"] = list(headers.items())
                await send(message)

            # 处理预检请求
            if scope["method"] == "OPTIONS":
                await send_with_cors({
                    "type": "http.response.start",
                    "status": 204,
                    "headers": [],
                })
                await send_with_cors({"type": "http.response.body", "body": b""})
                return

            await self.app(scope, receive, send_with_cors)
        else:
            await self.app(scope, receive, send)


# Flask 钩子（非标准中间件）
from flask import Flask, request, g
import time

app = Flask(__name__)

@app.before_request
def before_request():
    g.start_time = time.time()
    g.request_id = request.headers.get('X-Request-ID', str(uuid.uuid4()))

@app.after_request
def after_request(response):
    elapsed = time.time() - g.start_time
    response.headers['X-Request-ID'] = g.request_id
    response.headers['X-Response-Time'] = f"{elapsed:.3f}s"
    return response

@app.teardown_request
def teardown_request(exception=None):
    if exception:
        logger.error(f"请求异常: {exception}")
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 中间件顺序错误 | 后注册先执行（FastAPI 洋葱模型） | 理解注册顺序与执行顺序的关系 |
| 认证中间件跳过 | 路径白名单遗漏 | 明确列出公开路径，默认需要认证 |
| 同步中间件阻塞 ASGI | Django 同步中间件在异步视图中阻塞 | 用 `async def` 或 `SyncToAsync` |
| 限流内存泄漏 | IP 记录永不清理 | 定期清理过期记录或用 Redis |
| 异常被中间件吞掉 | 中间件捕获异常但返回正常响应 | 只做日志，不吞异常 |

### 最佳实践

- 中间件保持单一职责（认证/日志/CORS 各自独立）
- FastAPI 注册顺序注意：最后注册最先执行
- 限流用 Redis 存储（分布式环境），不用内存
- 认证中间件放在 CORS 之后（预检请求不需要认证）
- 请求 ID 贯穿整个请求链（日志/追踪/调试）
- 异常处理中间件只做日志，不修改响应（让框架错误处理器工作）

## 面试题

**Q1: WSGI 和 ASGI 的核心区别是什么？**
> WSGI 是同步协议，接口 `app(environ, start_response)`，每次请求由独立线程/进程处理，不支持 WebSocket 和长连接。ASGI 是异步协议，接口 `async app(scope, receive, send)`，支持 HTTP/WebSocket/长轮询，单个事件循环处理多个连接。ASGI 向后兼容 WSGI（可通过转换器包装 WSGI app）。新项目推荐 ASGI（FastAPI/Django 3.1+）。

**Q2: 洋葱模型的中间件执行顺序是什么？**
> 请求从外到内穿过中间件链（最后注册的最外层），响应从内到外返回。FastAPI 中 `add_middleware(A)` 然后 `add_middleware(B)`，执行顺序是 B → A → App → A → B。Django 中间件按 `MIDDLEWARE` 列表顺序执行请求阶段，逆序执行响应阶段。理解顺序的关键：中间件像洋葱皮层层包裹，请求穿入，响应穿出。

**Q3: FastAPI 的中间件和依赖注入（Depends）有什么区别？**
> 中间件作用于每个请求，无法按路由选择性应用，适合全局逻辑（CORS/日志/压缩）。依赖注入作用于特定路由，`Depends(get_db)` 只在声明了的路由上执行，适合路由级逻辑（数据库会话/当前用户）。中间件在路由匹配之前执行，依赖注入在路由匹配之后执行。能用依赖注入解决的不用中间件（更精确、更易测试）。

**Q4: Django 中间件的 `process_exception` 和 `process_template_response` 有什么区别？**
> `process_exception` 在视图抛出异常时调用，接收 request 和 exception，可以返回 Response（覆盖错误页面）或 None（让 Django 默认处理继续）。`process_template_response` 在视图返回 `TemplateResponse` 时调用，可以修改模板上下文或响应。两者都是可选钩子，不需要就不定义。

**Q5: 如何在 ASGI 中间件中处理 WebSocket？**
> ASGI 中间件的 `scope["type"]` 区分请求类型：`"http"` 是 HTTP 请求，`"websocket"` 是 WebSocket。中间件可以检查 scope 类型并分别处理。例如认证中间件在 WebSocket 连接时验证 token（从 query string 获取），HTTP 请求时从 Header 获取。注意 WebSocket 的 send/receive 消息格式与 HTTP 不同（`websocket.accept`/`websocket.send` 等）。

**Q6: CORS 中间件为什么要处理 OPTIONS 预检请求？**
> 浏览器跨域请求时，对于非简单请求（自定义 Header/PUT/DELETE 等）会先发送 OPTIONS 预检请求。CORS 中间件必须拦截 OPTIONS 并返回正确的 CORS 头（`Access-Control-Allow-Origin`/`Methods`/`Headers`），浏览器才会发送实际请求。如果中间件不对 OPTIONS 做特殊处理，预检请求会走到路由层，可能返回 405 Method Not Allowed，导致跨域请求失败。

**Q7: 如何实现分布式限流？**
> 单机限流用内存存储（字典/令牌桶），多实例不共享状态。分布式限流方案：1) Redis + 滑动窗口——`INCR` + `EXPIRE` 实现固定窗口，`EVALUA` Lua 脚本实现滑动窗口；2) Redis + 令牌桶——`CL.THROTTLE`（Redis Cell 模块）；3) 专用服务——Sentinel/RateLimiter。生产推荐 Redis 滑动窗口，精度和性能平衡。

**Q8: 请求 ID（Request ID）在微服务中有什么作用？**
> 请求 ID 是分配给每个请求的唯一标识，贯穿整个请求链路。作用：1) 日志关联——所有服务的日志通过请求 ID 串联，快速定位问题；2) 分布式追踪——与 OpenTelemetry/Jaeger 集成，可视化请求链路；3) 问题排查——用户反馈时提供请求 ID，直接搜索相关日志。生成方式：UUID4 或 Snowflake。入口网关生成，下游服务通过 Header 传递。

---

**相关链接：**
- [[FastAPI]]
- [[Django]]
- [[Flask]]
- [[WebSocket与实时通信]]
- [ASGI 规范](https://asgi.readthedocs.io/)

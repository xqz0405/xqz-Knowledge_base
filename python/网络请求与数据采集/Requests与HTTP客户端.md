---
tags:
  - Python
  - 网络请求与数据采集
  - HTTP
  - requests
  - httpx
date: 2026-05-14
status: 已完成
difficulty: 中等
---

# Requests与HTTP客户端

## What — 是什么

> Python HTTP 客户端库是网络请求的基础工具，核心三件为 requests（同步标准库）、httpx（同步+异步现代库）和 aiohttp（纯异步库），覆盖从简单 API 调用到高并发爬虫的全部场景。

**核心概念：**

- **requests**：Python 最流行的 HTTP 库，API 简洁优雅（"HTTP for Humans"），同步阻塞模型，覆盖 99% 的日常 HTTP 需求
- **httpx**：下一代 HTTP 客户端，API 兼容 requests 同时支持 HTTP/2、async/await、超时控制更精细，被称为"异步版 requests"
- **aiohttp**：纯异步 HTTP 客户端/服务端框架，基于 asyncio，适用于高并发爬虫和实时通信场景
- **会话（Session）**：复用 TCP 连接、Cookie、认证信息的请求上下文，避免每次请求重新握手，性能提升显著
- **连接池**：底层管理多个 TCP 连接的池化机制，控制并发数、复用连接、自动清理

**核心架构：**

- requests 架构：`requests.Request` → `requests.PreparedRequest` → `urllib3` 连接池 → HTTP 响应 → `requests.Response`
- httpx 架构：`httpx.Request` → `httpcore` 多路复用连接池 → HTTP/1.1 或 HTTP/2 → `httpx.Response`
- aiohttp 架构：`aiohttp.ClientSession` → asyncio 事件循环 → `aiohttp.ClientResponse`

**关键特性：**

- requests：自动解码、Cookie 持久化、Basic/Digest 认证、代理、超时、重定向
- httpx：兼容 requests API + async/await + HTTP/2 + 严格超时 + 环境变量代理
- aiohttp：纯异步、连接池、WebSocket 客户端、中间件、Cookie Jar

## Why — 为什么

**适用场景：**

- 调用 REST API——GitHub、Stripe、微信等第三方接口
- Web 爬虫数据采集——抓取网页内容、API 数据
- 微服务间通信——服务 A 调用服务 B 的 HTTP 接口
- 文件上传下载——大文件断点续传、流式下载
- 自动化测试——对 HTTP 接口做集成测试
- OAuth 认证流程——获取 token、刷新 token

**对比同类库：**

| 维度 | requests | httpx | aiohttp | urllib |
|------|----------|-------|---------|--------|
| 模型 | 同步阻塞 | 同步+异步 | 纯异步 | 同步阻塞 |
| API 风格 | 简洁优雅 | 兼容 requests | 偏底层 | 冗长繁琐 |
| HTTP/2 | 不支持 | 支持 | 不支持 | 不支持 |
| 异步支持 | 不支持 | 原生 async/await | 原生 async/await | 不支持 |
| WebSocket | 不支持 | 不支持 | 支持 | 不支持 |
| 连接池 | 内置（urllib3） | 内置（httpcore） | 内置 | 需手动管理 |
| 超时控制 | 基础 | 精细（连接/读取/写入/池） | 基础 | 基础 |
| 类型提示 | 第三方 stub | 原生支持 | 原生支持 | 标准库 |
| 学习曲线 | 极低 | 低（会 requests 即可） | 中 | 高 |
| 适用场景 | 日常 HTTP、脚本、爬虫 | 全场景、现代项目 | 高并发爬虫、实时通信 | 标准库无依赖 |

**优缺点：**

- ✅ requests 优点：
  - API 最简洁，Python 社区事实标准
  - 文档和教程极其丰富
  - 生态兼容性最好（几乎所有 SDK 基于 requests）
- ❌ requests 缺点：
  - 同步阻塞，不适用于高并发
  - 不支持 HTTP/2
  - 超时控制不够精细
- ✅ httpx 优点：
  - 兼容 requests API，迁移成本极低
  - 原生 async/await
  - HTTP/2 支持
  - 类型提示完善
- ❌ httpx 缺点：
  - 生态和第三方集成不如 requests 成熟
  - async 模式需要事件循环支持

## How — 怎么用

### 1. requests 基础

```bash
pip install requests
```

**基本请求：**

```python
import requests

# GET 请求
response = requests.get('https://api.github.com/users/psf')
print(response.status_code)    # 200
print(response.headers['content-type'])  # application/json
print(response.encoding)       # utf-8
print(response.text)           # 响应文本
print(response.json())         # 解析 JSON（自动）

# 带参数的 GET
params = {
    'q': 'python',
    'sort': 'stars',
    'order': 'desc',
    'per_page': 10,
}
response = requests.get('https://api.github.com/search/repositories', params=params)
data = response.json()
for repo in data['items'][:3]:
    print(f"{repo['full_name']}: {repo['stargazers_count']} stars")

# POST 请求
response = requests.post(
    'https://httpbin.org/post',
    data={'key': 'value'},  # 表单数据
)

# JSON POST
response = requests.post(
    'https://api.example.com/items',
    json={'name': 'test', 'price': 9.99},  # 自动设置 Content-Type: application/json
)

# PUT / PATCH / DELETE
requests.put('https://api.example.com/items/1', json={'name': 'updated'})
requests.patch('https://api.example.com/items/1', json={'price': 19.99})
requests.delete('https://api.example.com/items/1')
```

**响应对象详解：**

```python
response = requests.get('https://httpbin.org/get')

# 状态码
response.status_code          # 200
response.ok                   # True (2xx)
response.reason               # 'OK'
response.raise_for_status()   # 非 2xx 抛出 HTTPError

# 响应内容
response.text                 # 字符串（自动解码）
response.content              # 字节（用于二进制：图片、文件）
response.json()               # 解析 JSON（自动）
response.raw                  # 原始套接字响应（需 stream=True）
response.encoding             # 编码（可手动覆盖）

# 响应头
response.headers              # 大小写不敏感的字典
response.headers['Content-Type']
response.cookies              # CookieJar

# 请求信息（调试用）
response.request.method       # 'GET'
response.request.url          # 完整 URL
response.request.headers      # 请求头
response.request.body         # 请求体
response.history              # 重定向历史（列表）
response.elapsed              # 请求耗时（timedelta）
```

### 2. Session 会话管理

Session 是 requests 最重要的特性——复用 TCP 连接和 Cookie，大幅提升性能。

```python
import requests

# 不用 Session：每次请求都新建 TCP 连接 + 不共享 Cookie
r1 = requests.get('https://httpbin.org/cookies/set?session_id=abc123')
r2 = requests.get('https://httpbin.org/cookies')  # Cookie 丢失！

# 用 Session：复用连接 + Cookie 持久化
session = requests.Session()

# 设置全局默认值
session.headers.update({'Authorization': 'Bearer my-token'})
session.verify = True           # SSL 验证
session.proxies = {'https': 'http://proxy:8080'}

# 请求自动共享 Session 的 headers、cookies、auth
r1 = session.get('https://httpbin.org/cookies/set?session_id=abc123')
r2 = session.get('https://httpbin.org/cookies')
print(r2.json())  # {'cookies': {'session_id': 'abc123'}} ✓

# 也支持请求级覆盖
r3 = session.get(
    'https://api.example.com/data',
    headers={'X-Custom': 'override'},  # 合并到 Session headers
    timeout=10,
)

# 用完关闭（或用 with 语句）
session.close()

# 推荐：with 语句自动关闭
with requests.Session() as session:
    session.auth = ('user', 'pass')
    r = session.get('https://api.example.com/protected')
```

**Session 性能对比：**

```python
import requests
import time

url = 'https://httpbin.org/get'

# 无 Session：每次新建 TCP 连接 + TLS 握手
start = time.perf_counter()
for _ in range(10):
    requests.get(url)
print(f'无 Session: {time.perf_counter() - start:.2f}s')  # ~3-5s

# 有 Session：复用 TCP 连接
start = time.perf_counter()
with requests.Session() as s:
    for _ in range(10):
        s.get(url)
print(f'有 Session: {time.perf_counter() - start:.2f}s')  # ~0.5-1s
```

### 3. 认证与安全

```python
import requests
from requests.auth import HTTPBasicAuth, HTTPDigestAuth

# Basic Auth
response = requests.get(
    'https://api.example.com/protected',
    auth=HTTPBasicAuth('username', 'password'),
    # 简写：auth=('username', 'password')
)

# Bearer Token
headers = {'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...'}
response = requests.get('https://api.example.com/data', headers=headers)

# OAuth2 流程
session = requests.Session()

# 步骤1：获取 token
token_response = session.post('https://auth.example.com/oauth/token', data={
    'grant_type': 'client_credentials',
    'client_id': 'your_client_id',
    'client_secret': 'your_client_secret',
})
access_token = token_response.json()['access_token']

# 步骤2：使用 token 访问 API
session.headers.update({'Authorization': f'Bearer {access_token}'})
data = session.get('https://api.example.com/protected').json()

# 步骤3：刷新 token（token 过期时）
def refresh_token(session, client_id, client_secret, token_url):
    response = session.post(token_url, data={
        'grant_type': 'refresh_token',
        'refresh_token': session.refresh_token,
        'client_id': client_id,
        'client_secret': client_secret,
    })
    new_token = response.json()['access_token']
    session.headers.update({'Authorization': f'Bearer {new_token}'})
    return new_token

# Digest Auth（比 Basic 更安全）
response = requests.get(
    'https://api.example.com/digest',
    auth=HTTPDigestAuth('user', 'pass'),
)

# SSL 证书验证
response = requests.get('https://example.com', verify=True)      # 默认验证
response = requests.get('https://example.com', verify=False)     # 跳过验证（不推荐）
response = requests.get('https://example.com', verify='/path/to/ca-bundle.crt')  # 自定义 CA
```

### 4. 代理与超时

```python
import requests

# HTTP/HTTPS 代理
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'http://proxy.example.com:8080',
    # 或 SOCKS5 代理（需 pip install requests[socks]）
    # 'https': 'socks5://user:pass@proxy:1080',
}
response = requests.get('https://httpbin.org/ip', proxies=proxies)

# Session 级代理
session = requests.Session()
session.proxies.update(proxies)

# 环境变量代理（自动读取 HTTP_PROXY / HTTPS_PROXY）
session = requests.Session()
session.trust_env = True  # 默认就是 True

# 超时控制
# 单值：连接 + 读取总超时
requests.get('https://example.com', timeout=5)

# 元组：连接超时 + 读取超时（推荐）
requests.get('https://example.com', timeout=(3.05, 27))

# 永不超时（危险！）
# requests.get('https://example.com', timeout=None)

# 超时重试
from urllib3.util.retry import Retry
from requests.adapters import HTTPAdapter

session = requests.Session()

retry_strategy = Retry(
    total=3,                      # 最多重试 3 次
    backoff_factor=1,             # 重试间隔：1s, 2s, 4s
    status_forcelist=[429, 500, 502, 503, 504],  # 这些状态码触发重试
    allowed_methods=['GET', 'HEAD'],  # 仅 GET/HEAD 重试（POST 不重试）
    raise_on_status=False,        # 不抛异常，返回响应
)

adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount('https://', adapter)
session.mount('http://', adapter)

# 现在请求自动重试
response = session.get('https://api.example.com/data', timeout=5)
```

### 5. 流式请求与文件操作

```python
import requests

# 流式下载大文件（不一次性加载到内存）
response = requests.get('https://example.com/large-file.zip', stream=True)

# 方式一：写入文件
with open('large-file.zip', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)

# 方式二：带进度条
from tqdm import tqdm

total_size = int(response.headers.get('content-length', 0))
with open('large-file.zip', 'wb') as f:
    with tqdm(total=total_size, unit='B', unit_scale=True) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
            pbar.update(len(chunk))

# 断点续传
def download_with_resume(url, filepath):
    import os
    headers = {}

    # 检查已有文件大小
    if os.path.exists(filepath):
        downloaded = os.path.getsize(filepath)
        headers['Range'] = f'bytes={downloaded}-'
        mode = 'ab'
    else:
        mode = 'wb'

    response = requests.get(url, headers=headers, stream=True)
    with open(filepath, mode) as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)

# 流式上传
with open('data.json', 'rb') as f:
    response = requests.post(
        'https://api.example.com/upload',
        data=f,
        headers={'Content-Type': 'application/octet-stream'},
    )

# multipart/form-data 上传
files = {
    'file': ('report.pdf', open('report.pdf', 'rb'), 'application/pdf'),
    'metadata': (None, '{"title": "Report"}', 'application/json'),
}
response = requests.post('https://api.example.com/upload', files=files)
```

### 6. httpx 现代客户端

```bash
pip install httpx
# HTTP/2 支持
pip install httpx[http2]
```

```python
import httpx

# 同步 API（和 requests 几乎一样）
response = httpx.get('https://httpbin.org/get')
response = httpx.post('https://httpbin.org/post', json={'key': 'value'})
print(response.status_code)
print(response.json())

# Client（等价于 requests.Session）
with httpx.Client() as client:
    # 全局配置
    client.headers['Authorization'] = 'Bearer token'

    response = client.get('https://httpbin.org/get')
    response = client.post('https://httpbin.org/post', json={'data': 'test'})

# 超时控制（比 requests 更精细）
timeout = httpx.Timeout(
    connect=5.0,     # 连接超时
    read=10.0,       # 读取超时
    write=10.0,      # 写入超时
    pool=5.0,        # 连接池等待超时
)
with httpx.Client(timeout=timeout) as client:
    response = client.get('https://example.com')

# 异步请求
import asyncio

async def fetch_data():
    async with httpx.AsyncClient() as client:
        # 单个请求
        response = await client.get('https://httpbin.org/get')
        data = response.json()

        # 并发多个请求
        urls = [
            'https://httpbin.org/get?id=1',
            'https://httpbin.org/get?id=2',
            'https://httpbin.org/get?id=3',
        ]
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        results = [r.json() for r in responses]

        return results

results = asyncio.run(fetch_data())

# 异步流式下载
async def download_large_file(url, filepath):
    async with httpx.AsyncClient() as client:
        async with client.stream('GET', url) as response:
            with open(filepath, 'wb') as f:
                async for chunk in response.aiter_bytes(chunk_size=8192):
                    f.write(chunk)

# HTTP/2
with httpx.Client(http2=True) as client:
    response = client.get('https://httpbin.org/get')
    print(response.http_version)  # 'HTTP/2'
```

**requests 迁移到 httpx 的差异：**

| requests | httpx | 说明 |
|----------|-------|------|
| `requests.Session()` | `httpx.Client()` | 会话/客户端 |
| `response.text` | `response.text` | 相同 |
| `response.content` | `response.content` | 相同 |
| `response.json()` | `response.json()` | 相同 |
| `response.ok` | `response.is_success` | httpx 更明确 |
| `response.raise_for_status()` | `response.raise_for_status()` | 相同 |
| `timeout=5` | `timeout=5.0` | httpx 推荐浮点数 |
| `auth=('user', 'pass')` | `auth=('user', 'pass')` | 相同 |
| `verify=False` | `verify=False` | 相同 |
| 无原生异步 | `AsyncClient()` | httpx 独有 |
| 无 HTTP/2 | `http2=True` | httpx 独有 |

### 7. aiohttp 高并发异步

```bash
pip install aiohttp
```

```python
import aiohttp
import asyncio

# 基础异步请求
async def fetch_one():
    async with aiohttp.ClientSession() as session:
        async with session.get('https://httpbin.org/get') as response:
            # 状态码和头
            print(response.status)
            print(response.headers['Content-Type'])

            # 读取内容
            text = await response.text()
            data = await response.json()
            content = await response.read()  # 字节

            return data

# 高并发爬虫
async def fetch_many(urls, max_concurrency=10):
    semaphore = asyncio.Semaphore(max_concurrency)

    async def fetch_one(session, url):
        async with semaphore:
            async with session.get(url) as response:
                response.raise_for_status()
                return await response.json()

    async with aiohttp.ClientSession() as session:
        tasks = [fetch_one(session, url) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)

        # 处理结果和异常
        success = []
        failures = []
        for url, result in zip(urls, results):
            if isinstance(result, Exception):
                failures.append((url, result))
            else:
                success.append(result)

        return success, failures

# 运行
urls = [f'https://httpbin.org/get?id={i}' for i in range(50)]
results, failures = asyncio.run(fetch_many(urls))
print(f'成功: {len(results)}, 失败: {len(failures)}')

# 异步 POST
async def post_data():
    async with aiohttp.ClientSession() as session:
        # JSON POST
        async with session.post(
            'https://httpbin.org/post',
            json={'name': 'test', 'value': 42},
        ) as response:
            data = await response.json()

        # 表单 POST
        async with session.post(
            'https://httpbin.org/post',
            data={'username': 'admin', 'password': 'secret'},
        ) as response:
            data = await response.json()

# 异步文件上传
async def upload_file():
    async with aiohttp.ClientSession() as session:
        data = aiohttp.FormData()
        data.add_field('file', open('data.csv', 'rb'),
                       filename='data.csv',
                       content_type='text/csv')

        async with session.post(
            'https://api.example.com/upload',
            data=data,
        ) as response:
            return await response.json()

# 超时控制
timeout = aiohttp.ClientTimeout(
    total=30,      # 总超时
    connect=5,     # 连接超时
    sock_read=10,  # 读取超时
)
async with aiohttp.ClientSession(timeout=timeout) as session:
    async with session.get('https://example.com') as response:
        data = await response.json()

# aiohttp WebSocket 客户端
async def websocket_client():
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect('wss://echo.websocket.org') as ws:
            # 发送消息
            await ws.send_str('Hello WebSocket')
            await ws.send_json({'type': 'ping', 'data': 'test'})

            # 接收消息
            async for msg in ws:
                if msg.type == aiohttp.WSMsgType.TEXT:
                    print(f'收到: {msg.data}')
                elif msg.type == aiohttp.WSMsgType.ERROR:
                    print(f'错误: {ws.exception()}')
                    break
```

### 8. 实战：封装健壮的 HTTP 客户端

```python
"""生产级 HTTP 客户端封装，支持重试、限流、日志、错误处理"""
import time
import logging
from typing import Any, Optional
from functools import wraps

import httpx
from httpx._transports.default import HTTPTransport

logger = logging.getLogger(__name__)


class HttpClient:
    """健壮的 HTTP 客户端，封装 httpx"""

    def __init__(
        self,
        base_url: str = '',
        headers: Optional[dict] = None,
        timeout: float = 30.0,
        max_retries: int = 3,
        retry_delay: float = 1.0,
        rate_limit: Optional[float] = None,  # 每秒最多请求数
    ):
        self.base_url = base_url
        self.max_retries = max_retries
        self.retry_delay = retry_delay
        self.rate_limit = rate_limit
        self._last_request_time = 0.0

        transport = HTTPTransport(retries=max_retries)

        self.client = httpx.Client(
            base_url=base_url,
            headers=headers or {},
            timeout=timeout,
            transport=transport,
            follow_redirects=True,
        )

    def _rate_limit_wait(self):
        """限流等待"""
        if self.rate_limit:
            elapsed = time.monotonic() - self._last_request_time
            min_interval = 1.0 / self.rate_limit
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
        self._last_request_time = time.monotonic()

    def request(self, method: str, url: str, **kwargs) -> Any:
        """发送请求，自动重试和错误处理"""
        self._rate_limit_wait()

        last_exception = None
        for attempt in range(1, self.max_retries + 1):
            try:
                response = self.client.request(method, url, **kwargs)
                response.raise_for_status()

                logger.info(f'{method} {url} → {response.status_code}')
                return response.json() if 'json' in response.headers.get('content-type', '') else response.text

            except httpx.TimeoutException as e:
                last_exception = e
                logger.warning(f'请求超时 (尝试 {attempt}/{self.max_retries}): {url}')
            except httpx.HTTPStatusError as e:
                if e.response.status_code in (429, 502, 503, 504):
                    last_exception = e
                    retry_after = e.response.headers.get('Retry-After')
                    wait = float(retry_after) if retry_after else self.retry_delay * attempt
                    logger.warning(f'服务器错误 {e.response.status_code}，{wait}s 后重试')
                    time.sleep(wait)
                else:
                    raise  # 4xx 错误不重试

            except httpx.RequestError as e:
                last_exception = e
                logger.warning(f'请求失败 (尝试 {attempt}/{self.max_retries}): {e}')

            if attempt < self.max_retries:
                time.sleep(self.retry_delay * attempt)  # 退避

        raise ConnectionError(f'请求失败，已重试 {self.max_retries} 次') from last_exception

    def get(self, url: str, **kwargs) -> Any:
        return self.request('GET', url, **kwargs)

    def post(self, url: str, **kwargs) -> Any:
        return self.request('POST', url, **kwargs)

    def put(self, url: str, **kwargs) -> Any:
        return self.request('PUT', url, **kwargs)

    def delete(self, url: str, **kwargs) -> Any:
        return self.request('DELETE', url, **kwargs)

    def close(self):
        self.client.close()

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.close()


# 使用示例
if __name__ == '__main__':
    with HttpClient(
        base_url='https://api.github.com',
        headers={'Authorization': 'Bearer ghp_xxx'},
        rate_limit=2.0,  # 每秒最多 2 个请求
    ) as client:
        # GET 请求
        user = client.get('/users/psf')
        print(f"User: {user['login']}")

        # 搜索仓库
        repos = client.get('/search/repositories', params={
            'q': 'python',
            'sort': 'stars',
            'per_page': 5,
        })
        for repo in repos['items']:
            print(f"  {repo['full_name']}: {repo['stargazers_count']} stars")
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 连接超时频繁 | 网络不稳定或服务器响应慢 | 增大 timeout，配置 Retry 自动重试 |
| SSL 证书验证失败 | 自签名证书或系统 CA 过期 | `verify='/path/to/ca.pem'` 指定 CA，不要 `verify=False` |
| 内存溢出（大文件） | `response.content` 一次性加载全部内容 | 使用 `stream=True` + `iter_content()` 分块读取 |
| Cookie 丢失 | 没有使用 Session | 改用 `requests.Session()` 共享 Cookie |
| 429 Too Many Requests | 请求频率超限 | 添加限流（sleep/semaphore），检查 Retry-After 头 |
| 中文乱码 | 编码检测错误 | 手动设置 `response.encoding = 'utf-8'` 或 `response.content.decode('gbk')` |
| 代理不生效 | 环境变量或配置错误 | 检查 `proxies` 参数格式，SOCKS5 需额外安装 |
| 异步请求卡死 | 未正确使用 `await` 或事件循环 | 确保所有 async 调用都有 `await`，`asyncio.run()` 启动 |

---

## 面试题

**Q1: requests 的 Session 和直接调用 requests.get 有什么区别？为什么爬虫要用 Session？**

> 核心区别：(1) **连接复用**——`requests.get()` 每次创建新的 TCP 连接（三次握手 + TLS 握手），Session 在同一主机上复用 TCP 连接，省去重复握手开销；(2) **Cookie 持久化**——直接调用不保留 Cookie，Session 自动管理 Cookie Jar，登录后后续请求自动携带；(3) **全局配置**——Session 可设置默认 headers、auth、proxies、verify 等，避免每个请求重复传参。爬虫必须用 Session 的原因：(1) 性能——10 次请求，无 Session 约 3-5s，有 Session 约 0.5-1s；(2) 状态维护——登录态、Session ID 等依赖 Cookie 传递；(3) 连接池管理——Session 底层 urllib3 连接池自动管理连接数和生命周期。

**Q2: requests、httpx、aiohttp 如何选择？**

> 选择标准：(1) **日常脚本/简单 API 调用**——用 requests，API 最简洁，生态最成熟，99% 场景够用；(2) **新项目/需要异步**——用 httpx，API 兼容 requests，同步异步一体，类型提示好，是 requests 的现代替代品；(3) **高并发爬虫/实时通信**——用 aiohttp，纯异步，WebSocket 支持，适合万级并发场景。迁移建议：现有项目不急着换，新项目优先 httpx。httpx 的优势是"会 requests 就会 httpx"，迁移成本极低，同时获得 async + HTTP/2 + 精细超时控制。aiohttp 的优势是更底层、更灵活、WebSocket 客户端/服务端一体，但 API 偏繁琐。

**Q3: 如何处理 HTTP 请求的超时和重试？**

> 超时：(1) **必须设置超时**——永远不要用默认的无限等待，生产环境至少设置 `timeout=(3.05, 27)`（连接 3s + 读取 27s）；(2) **区分超时类型**——连接超时（DNS + TCP + TLS）和读取超时（等待服务器响应）应分别设置；(3) **httpx 更精细**——支持 connect/read/write/pool 四种超时。重试：(1) **仅对幂等请求重试**——GET/HEAD/PUT/DELETE 可重试，POST 通常不重试（可能重复创建资源）；(2) **按状态码重试**——429（限流）、502/503/504（服务端临时故障）可重试，4xx 客户端错误不重试；(3) **指数退避**——重试间隔递增（1s, 2s, 4s），避免雪崩；(4) **Retry-After 头**——429 响应通常带 `Retry-After` 头，应遵守服务器指定的等待时间。

**Q4: 流式请求（stream=True）适用于什么场景？原理是什么？**

> 适用场景：(1) **大文件下载**——避免将整个文件加载到内存，逐块写入磁盘；(2) **流式 API**——Server-Sent Events (SSE)、ChatGPT 流式输出等持续产生数据的接口；(3) **大响应体**——不确定响应大小时，流式读取比一次性加载更安全。原理：默认情况下 `requests.get()` 会等待整个响应体下载完毕再返回，`response.content` 包含全部数据。设置 `stream=True` 后，requests 只下载响应头就返回 Response 对象，响应体在网络到达时通过 `iter_content()` / `iter_lines()` 逐块提供，内存中始终只保持一个 chunk 大小的数据。注意：`stream=True` 时必须消费响应体或调用 `response.close()`，否则连接不会归还连接池。

**Q5: 如何实现高并发爬虫？并发控制有哪些手段？**

> 三层并发控制：(1) **异步 I/O**——使用 httpx.AsyncClient 或 aiohttp，单线程即可并发数百请求（I/O 密集型不需要多线程）；(2) **Semaphore 限流**——`asyncio.Semaphore(N)` 控制同时进行的请求数，防止打爆目标服务器或本地资源耗尽；(3) **速率限制**——每秒最多 N 个请求，通过令牌桶或漏桶算法实现。完整方案：`httpx.AsyncClient` + `Semaphore` + 请求间隔控制 + 重试。典型配置：Semaphore(20) + 每秒 10 个请求 + 3 次重试。多线程方案（`concurrent.futures.ThreadPoolExecutor`）也可行，但线程开销大（每个线程 ~8MB 栈），不如异步高效。多进程仅在 CPU 密集型（如解析大量 HTML）时有用。

**Q6: requests 如何配置代理？SOCKS5 代理和 HTTP 代理有什么区别？**

> HTTP 代理：`proxies={'https': 'http://proxy:8080'}`，代理服务器转发 HTTP 请求，对 HTTPS 使用 CONNECT 隧道。SOCKS5 代理：`proxies={'https': 'socks5://proxy:1080'}`，需 `pip install requests[socks]`（依赖 PySocks），SOCKS5 是通用代理协议，支持任意 TCP 流量。区别：(1) **协议层**——HTTP 代理理解 HTTP 协议，可缓存/修改请求头；SOCKS5 在传输层代理，不理解也不修改应用层协议；(2) **安全性**——HTTP 代理对 HTTPS 使用 CONNECT 隧道，代理看不到加密内容；SOCKS5 同样不查看内容；(3) **适用范围**——HTTP 代理仅支持 HTTP/HTTPS；SOCKS5 支持任何 TCP 流量（FTP、SMTP 等）；(4) **认证**——两者都支持用户名密码认证。爬虫常用 SOCKS5 代理池来轮换 IP。

**Q7: httpx 相比 requests 的核心改进是什么？**

> (1) **原生异步支持**——`AsyncClient` + `await` 实现异步请求，requests 只能同步；(2) **HTTP/2**——`httpx.Client(http2=True)` 启用 HTTP/2 多路复用，一个 TCP 连接并发多个请求，requests 仅 HTTP/1.1；(3) **精细超时**——`httpx.Timeout(connect=5, read=10, write=10, pool=5)` 四种超时独立控制，requests 只能设置连接+读取总超时；(4) **类型提示**——httpx 原生支持 type hints，IDE 自动补全和类型检查更好，requests 需要第三方 `types-requests` stub；(5) **ASGI 传输**——httpx 可以直接调用 ASGI 应用（如 FastAPI）而不经过网络，测试更方便；(6) **环境感知**——自动读取环境变量代理、SSL 配置。API 设计上 httpx 刻意兼容 requests，`requests.get()` → `httpx.get()`，`Session` → `Client`，迁移几乎只需替换导入。

**Q8: 如何封装一个生产级 HTTP 客户端？需要考虑哪些方面？**

> 六大方面：(1) **重试策略**——指数退避 + 可配置重试次数 + 按 status code 区分重试（429/5xx 重试，4xx 不重试）；(2) **超时控制**——区分连接/读取/总超时，默认值 + 请求级覆盖；(3) **限流**——令牌桶或 Semaphore 控制并发和 QPS，避免触发目标服务限流；(4) **日志**——记录请求 URL、方法、状态码、耗时，异常时记录完整错误信息；(5) **错误处理**——区分网络错误（重试）、客户端错误（4xx 不重试）、服务端错误（5xx 重试），统一异常类型；(6) **资源管理**——`with` 语句自动关闭连接，Session/Client 复用。额外考虑：请求/响应拦截器（添加 trace ID、统一认证）、熔断（连续失败后暂停请求）、指标采集（Prometheus 统计 QPS/延迟/错误率）。

---

**相关链接：** [[BeautifulSoup与HTML解析]] [[Scrapy框架]] [[异步编程]] [[WebSocket与实时通信]] [[Web中间件与请求处理]]

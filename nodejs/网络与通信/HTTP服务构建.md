---
tags:
  - Node.js
  - HTTP
  - Web服务器
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# HTTP服务构建

## What — 它是什么？

### 原生 http 模块

Node.js 内置的 `http` 模块是构建 Web 服务的基础设施，无需安装任何第三方依赖即可创建 HTTP 服务器与客户端。它基于 C++ 的 libuv 和 llhttp 实现，提供极低层级的事件驱动网络 I/O 能力。`http` 模块是 Node.js 诞生的核心动机之一——Ryan Dahl 最初设计 Node.js 的目标就是构建高性能的 HTTP 服务。

```javascript
const http = require('http');

// 最简单的 HTTP 服务器
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain; charset=utf-8');
  res.end('你好，Node.js HTTP 服务！');
});

server.listen(3000, () => {
  console.log('服务器运行在 http://localhost:3000/');
});
```

### http.createServer 详解

`http.createServer([options][, requestListener])` 方法创建并返回一个 `http.Server` 实例。`requestListener` 是一个自动绑定到 `request` 事件的函数，接收两个参数：请求对象 `IncomingMessage` 和响应对象 `ServerResponse`。

**options 参数**（Node.js 18+）支持以下关键配置：

| 选项 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `requestTimeout` | number | 300000 (5min) | 接收完整请求的超时时间（毫秒） |
| `headersTimeout` | number | 60000 (1min) | 解析请求头的超时时间（毫秒） |
| `keepAliveTimeout` | number | 5000 (5s) | Keep-Alive 连接空闲超时时间 |
| `keepAlive` | boolean | true | 是否启用 TCP Keep-Alive |
| `maxHeaderSize` | number | 16384 (16KB) | 请求头最大字节数 |

```javascript
const server = http.createServer({
  requestTimeout: 30000,   // 30秒请求超时
  headersTimeout: 10000,   // 10秒头部超时
  keepAliveTimeout: 10000, // 10秒Keep-Alive超时
  maxHeaderSize: 32768,    // 32KB请求头上限
}, (req, res) => {
  res.end('OK');
});
```

**http.Server 关键事件：**

- `request`：每次收到 HTTP 请求触发，最常用
- `connection`：新的 TCP 连接建立时触发（socket 对象）
- `close`：服务器关闭时触发
- `checkContinue`：请求带 `Expect: 100-continue` 时触发
- `upgrade`：客户端请求协议升级（如 WebSocket）时触发
- `clientError`：客户端发送了无效请求时触发

### 请求对象 IncomingMessage

`http.IncomingMessage` 继承自 `stream.Readable`，是对 HTTP 请求的完整封装。它同时实现了 `stream.Readable` 接口用于读取请求体，以及包含丰富的请求元数据属性。

**核心属性与方法：**

| 属性/方法 | 类型 | 说明 |
|-----------|------|------|
| `req.method` | string | 请求方法（GET/POST/PUT/DELETE等） |
| `req.url` | string | 完整请求路径（含查询字符串） |
| `req.headers` | object | 请求头对象（全小写键名） |
| `req.httpVersion` | string | HTTP 版本（'1.0'/'1.1'/'2.0'） |
| `req.socket` | Socket | 底层 TCP socket 对象 |
| `req.rawHeaders` | string[] | 原始请求头列表（交替键值） |
| `req.trailers` | object | 尾部头部（chunked 编码） |
| `req.destroy()` | function | 销毁请求流，中断连接 |

```javascript
const server = http.createServer((req, res) => {
  // 解析 URL
  const parsedUrl = new URL(req.url, `http://${req.headers.host}`);

  console.log('方法:', req.method);
  console.log('路径:', parsedUrl.pathname);
  console.log('查询参数:', Object.fromEntries(parsedUrl.searchParams));
  console.log('请求头:', req.headers);
  console.log('远程IP:', req.socket.remoteAddress);
  console.log('HTTP版本:', req.httpVersion);

  res.end(JSON.stringify({
    method: req.method,
    path: parsedUrl.pathname,
    query: Object.fromEntries(parsedUrl.searchParams),
  }, null, 2));
});
```

**注意事项：**
- `req.headers` 中的键名自动转为小写（如 `Content-Type` → `content-type`）
- `req.url` 仅包含路径和查询字符串，不含协议和主机名
- 请求体需要通过流式读取获取，不能直接像框架中那样 `req.body`

### 响应对象 ServerResponse

`http.ServerResponse` 继承自 `stream.Writable`，是对 HTTP 响应的封装。开发者通过它设置状态码、响应头和响应体。

**核心方法与属性：**

| 方法/属性 | 说明 |
|-----------|------|
| `res.statusCode` | 获取/设置 HTTP 状态码 |
| `res.statusMessage` | 获取/设置状态描述文本 |
| `res.setHeader(name, value)` | 设置单个响应头 |
| `res.getHeader(name)` | 获取已设置的响应头 |
| `res.removeHeader(name)` | 移除已设置的响应头 |
| `res.writeHead(statusCode, headers)` | 一次性写入状态码和多个响应头 |
| `res.write(chunk[, encoding])` | 写入响应体数据块 |
| `res.end([data[, encoding]])` | 结束响应（可同时发送最后一块数据） |
| `res.flushHeaders()` | 立即发送已排队等待的响应头 |
| `res.socket` | 底层 TCP socket 对象 |

**关键行为规则：**
1. `res.writeHead()` 必须在 `res.write()` / `res.end()` 之前调用
2. 调用 `res.end()` 后不能再写入数据，否则触发 `ERR_STREAM_WRITE_AFTER_END`
3. 如果没有手动调用 `res.writeHead()`，第一次 `res.write()` 或 `res.end()` 会隐式发送头部，状态码默认 200
4. `setHeader()` 与 `writeHead()` 不能冲突：如果调用了 `writeHead()`，则之前通过 `setHeader()` 设置的同名头部会被 `writeHead()` 中的覆盖

```javascript
// writeHead 与 setHeader 的区别
res.setHeader('Content-Type', 'text/html');
res.setHeader('X-Custom', 'value');
// writeHead 会发送头部，之后不能再 setHeader
res.writeHead(200, { 'Content-Type': 'text/plain' }); // 覆盖上面设置的 Content-Type
// 最终响应头: Content-Type: text/plain, X-Custom: value
```

### 路由实现

原生 `http` 模块没有内置路由功能，需要手动解析 URL 和方法来实现。路由的本质是根据请求的 Method + Path 映射到对应的处理函数。

```javascript
// 简易路由实现
const router = {
  GET: {},
  POST: {},
  PUT: {},
  DELETE: {},
};

function addRoute(method, path, handler) {
  router[method][path] = handler;
}

function matchRoute(req, res) {
  const parsedUrl = new URL(req.url, `http://${req.headers.host}`);
  const method = req.method.toUpperCase();
  const pathname = parsedUrl.pathname;
  const handler = router[method]?.[pathname];

  if (handler) {
    handler(req, res, parsedUrl);
  } else {
    res.writeHead(404, { 'Content-Type': 'application/json; charset=utf-8' });
    res.end(JSON.stringify({ error: 'Not Found', path: pathname }));
  }
}

// 注册路由
addRoute('GET', '/api/users', (req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ users: ['Alice', 'Bob', 'Charlie'] }));
});

addRoute('GET', '/api/users/:id', (req, res, url) => {
  // 注意：原生路由不支持自动参数提取，需要正则匹配
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ id: 'param-extraction-needed' }));
});

addRoute('POST', '/api/users', (req, res) => {
  let body = '';
  req.on('data', chunk => { body += chunk; });
  req.on('end', () => {
    res.writeHead(201, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ created: true, data: body }));
  });
});

const server = http.createServer(matchRoute);
server.listen(3000);
```

**进阶：支持路径参数的路由**

```javascript
class Router {
  constructor() {
    this.routes = [];
  }

  add(method, pattern, handler) {
    // 将 /api/users/:id 转为正则 /\/api\/users\/([^/]+)/
    const paramNames = [];
    const regexStr = pattern.replace(/:([^/]+)/g, (_, name) => {
      paramNames.push(name);
      return '([^/]+)';
    });
    const regex = new RegExp(`^${regexStr}$`);
    this.routes.push({ method, regex, paramNames, handler });
  }

  match(req) {
    const parsedUrl = new URL(req.url, `http://${req.headers.host}`);
    for (const route of this.routes) {
      if (route.method !== req.method.toUpperCase()) continue;
      const match = parsedUrl.pathname.match(route.regex);
      if (match) {
        const params = {};
        route.paramNames.forEach((name, i) => {
          params[name] = match[i + 1];
        });
        return { handler: route.handler, params, query: parsedUrl.searchParams };
      }
    }
    return null;
  }
}

// 使用
const router = new Router();
router.add('GET', '/api/users/:id', (req, res, { params, query }) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ userId: params.id }));
});
```

### 静态文件服务

静态文件服务需要根据文件扩展名设置正确的 MIME 类型，并处理文件读取、缓存、范围请求等问题。

**常见 MIME 类型映射：**

| 扩展名 | MIME 类型 |
|--------|-----------|
| `.html` | text/html; charset=utf-8 |
| `.css` | text/css; charset=utf-8 |
| `.js` | application/javascript; charset=utf-8 |
| `.json` | application/json; charset=utf-8 |
| `.png` | image/png |
| `.jpg` | image/jpeg |
| `.gif` | image/gif |
| `.svg` | image/svg+xml |
| `.ico` | image/x-icon |
| `.woff2` | font/woff2 |
| `.pdf` | application/pdf |

### 请求体解析

由于 `IncomingMessage` 是可读流，请求体不能像 Express 中 `req.body` 那样直接获取，必须手动从流中读取数据并拼接。

### Keep-Alive

HTTP Keep-Alive（持久连接）允许同一个 TCP 连接上发送多个请求和响应，避免每次请求都重新建立 TCP 连接（三次握手）和 TLS 握手，显著降低延迟。HTTP/1.1 默认启用 Keep-Alive。

**工作原理：**
1. 客户端发送请求时携带 `Connection: keep-alive` 头部
2. 服务器响应同样携带 `Connection: keep-alive` 表示同意保持连接
3. 连接在空闲 `keepAliveTimeout` 毫秒后自动断开
4. 客户端也可以通过 `Connection: close` 主动关闭连接

**Node.js 中的配置：**

```javascript
const server = http.createServer({ keepAliveTimeout: 10000 }, (req, res) => {
  res.end('OK');
});

// 全局 socket 级别配置
server.on('connection', (socket) => {
  socket.setKeepAlive(true, 5000); // 每5秒发送TCP Keep-Alive探测包
});

server.listen(3000);
```

### HTTPS

HTTPS 是 HTTP over TLS，通过 TLS/SSL 加密传输数据。Node.js 通过 `https` 模块提供 HTTPS 服务器功能，需要提供证书和私钥。

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
  ca: fs.readFileSync('ca-cert.pem'),         // 可选：CA证书链
  passphrase: 'your-passphrase',               // 可选：私钥密码
  minVersion: 'TLSv1.2',                       // 最低TLS版本
  ciphers: 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256',
  honorCipherOrder: true,                       // 优先使用服务端密码套件
};

const server = https.createServer(options, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('HTTPS 安全连接！');
});

server.listen(443, () => {
  console.log('HTTPS 服务器运行在 https://localhost:443/');
});
```

**自签名证书开发环境快速生成：**
```bash
# 生成私钥
openssl genrsa -out server-key.pem 2048
# 生成自签名证书（有效期365天）
openssl req -new -x509 -key server-key.pem -out server-cert.pem -days 365
```

### HTTP/2

HTTP/2 引入了多路复用、头部压缩、服务器推送等重大改进。Node.js 通过 `http2` 模块原生支持 HTTP/2。

**核心特性：**
- **多路复用**：单一 TCP 连接上并行交错发送多个请求/响应，消除 HTTP/1.x 的队头阻塞
- **头部压缩**：使用 HPACK 算法压缩请求/响应头，大幅减少开销
- **服务器推送**：服务器可以主动向客户端推送资源
- **二进制分帧**：所有通信在单 TCP 连接上完成，信息分割为更小的帧

```javascript
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
});

server.on('stream', (stream, headers) => {
  const method = headers[':method'];
  const path = headers[':path'];

  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200,
  });

  // 服务器推送
  stream.pushStream({ ':path': '/style.css' }, (err, pushStream) => {
    if (err) throw err;
    pushStream.respond({
      'content-type': 'text/css; charset=utf-8',
      ':status': 200,
    });
    pushStream.end('body { font-family: sans-serif; }');
  });

  stream.end('<link rel="stylesheet" href="/style.css"><h1>HTTP/2 推送</h1>');
});

server.listen(8443, () => {
  console.log('HTTP/2 服务器运行在 https://localhost:8443/');
});
```

---

## Why — 为什么学它？

### 适用场景

1. **理解底层原理**：无论是使用 Express、Koa 还是 Fastify，底层都是 `http` 模块。理解原生 API 有助于排查框架无法解决的问题，如请求挂起、连接泄漏、超时配置等
2. **轻量级 API 服务**：对于极简的内网工具、健康检查端点、Webhook 接收器等，不需要框架的额外开销，原生 `http` 足够
3. **反向代理/网关**：构建自定义代理逻辑时，需要直接操作 socket、处理 `upgrade` 事件、管理连接池，框架的抽象反而成为阻碍
4. **WebSocket 服务**：WebSocket 建立在 HTTP 升级机制之上，需要监听 `upgrade` 事件获取原始 socket
5. **性能基准测试**：原生 `http` 是 Node.js Web 性能的上限，理解它能帮助你评估框架的额外开销
6. **教学与面试**：掌握原生 HTTP 模块是理解 Node.js 网络编程的必经之路

### 框架对比

| 对比维度 | 原生 http | Express | Fastify |
|----------|-----------|---------|---------|
| 性能（req/s） | ~50,000+ | ~15,000 | ~45,000+ |
| 代码量 | 大，手动实现所有功能 | 中等，中间件丰富 | 中等，插件生态完善 |
| 路由 | 无内置，需手动实现 | 内置，支持参数/正则 | 内置，支持约束路由 |
| 中间件/插件 | 无，需自行实现洋葱模型 | 中间件模式（线性） | 插件系统（支持异步） |
| 请求体解析 | 手动读取流 | 需 body-parser 中间件 | 内置，基于 schema 解析 |
| 类型安全 | 无 | 弱（需 @types） | 强（JSON Schema → TS） |
| 适用场景 | 底层理解/极简服务/代理 | 快速开发/全栈应用 | 高性能API/微服务 |

### 优缺点

**原生 http 的优点：**
- 零依赖，无额外安全风险
- 性能最高，没有框架抽象的开销
- 完全掌控连接生命周期、超时行为
- 理解底层有助于使用和调试框架
- 适合学习 HTTP 协议本身

**原生 http 的缺点：**
- 功能极其原始，路由/中间件/请求体解析等全部需要手动实现
- 代码冗长，维护成本高
- 缺少生态系统支持（验证、序列化、鉴权等）
- 容易出现安全漏洞（路径遍历、CRLF注入、未处理异常等）
- 团队协作时没有统一的代码结构约定

---

## How — 怎么用？

### 代码示例 1：原生 HTTP 服务器 + 错误处理

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // 全局错误处理，防止未捕获异常导致进程崩溃
  try {
    const parsedUrl = new URL(req.url, `http://${req.headers.host}`);
    const pathname = parsedUrl.pathname;

    // 健康检查端点
    if (pathname === '/health' && req.method === 'GET') {
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ status: 'ok', timestamp: Date.now() }));
      return;
    }

    // 根路径
    if (pathname === '/' && req.method === 'GET') {
      res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
      res.end('<h1>Welcome to Node.js HTTP Server</h1>');
      return;
    }

    // 404
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not Found', path: pathname }));
  } catch (err) {
    console.error('请求处理错误:', err);
    if (!res.headersSent) {
      res.writeHead(500, { 'Content-Type': 'application/json' });
    }
    res.end(JSON.stringify({ error: 'Internal Server Error' }));
  }
});

// 处理客户端错误（如发送格式错误的请求）
server.on('clientError', (err, socket) => {
  console.error('客户端错误:', err.message);
  if (!socket.destroyed) {
    socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
  }
});

// 优雅关闭
function gracefulShutdown(signal) {
  console.log(`收到 ${signal}，正在关闭服务器...`);
  server.close(() => {
    console.log('服务器已关闭所有连接');
    process.exit(0);
  });
  // 强制退出超时
  setTimeout(() => {
    console.error('强制关闭，等待超时');
    process.exit(1);
  }, 10000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

server.listen(3000, () => {
  console.log('HTTP 服务器运行在 http://localhost:3000/');
});
```

### 代码示例 2：静态文件服务 + MIME 类型 + 缓存

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const MIME_TYPES = {
  '.html': 'text/html; charset=utf-8',
  '.css': 'text/css; charset=utf-8',
  '.js': 'application/javascript; charset=utf-8',
  '.json': 'application/json; charset=utf-8',
  '.png': 'image/png',
  '.jpg': 'image/jpeg',
  '.jpeg': 'image/jpeg',
  '.gif': 'image/gif',
  '.svg': 'image/svg+xml',
  '.ico': 'image/x-icon',
  '.woff': 'font/woff',
  '.woff2': 'font/woff2',
  '.ttf': 'font/ttf',
  '.pdf': 'application/pdf',
  '.txt': 'text/plain; charset=utf-8',
};

const STATIC_DIR = path.join(__dirname, 'public');

const server = http.createServer((req, res) => {
  const parsedUrl = new URL(req.url, `http://${req.headers.host}`);
  let filePath = path.join(STATIC_DIR, parsedUrl.pathname);

  // 安全检查：防止路径遍历攻击
  if (!filePath.startsWith(STATIC_DIR)) {
    res.writeHead(403, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Forbidden' }));
    return;
  }

  // 默认访问 index.html
  if (parsedUrl.pathname === '/') {
    filePath = path.join(STATIC_DIR, 'index.html');
  }

  const ext = path.extname(filePath).toLowerCase();
  const contentType = MIME_TYPES[ext] || 'application/octet-stream';

  fs.stat(filePath, (err, stats) => {
    if (err) {
      if (err.code === 'ENOENT') {
        res.writeHead(404, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'File Not Found' }));
      } else {
        res.writeHead(500, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Internal Server Error' }));
      }
      return;
    }

    // 缓存控制
    const lastModified = stats.mtime.toUTCString();
    const ifModifiedSince = req.headers['if-modified-since'];

    if (ifModifiedSince && new Date(ifModifiedSince) >= stats.mtime) {
      res.writeHead(304); // Not Modified
      res.end();
      return;
    }

    // 范围请求支持（大文件/视频）
    const range = req.headers.range;
    if (range) {
      const total = stats.size;
      const parts = range.replace(/bytes=/, '').split('-');
      const start = parseInt(parts[0], 10);
      const end = parts[1] ? parseInt(parts[1], 10) : total - 1;

      res.writeHead(206, {
        'Content-Range': `bytes ${start}-${end}/${total}`,
        'Accept-Ranges': 'bytes',
        'Content-Length': end - start + 1,
        'Content-Type': contentType,
      });

      fs.createReadStream(filePath, { start, end }).pipe(res);
      return;
    }

    // 普通请求
    res.writeHead(200, {
      'Content-Type': contentType,
      'Content-Length': stats.size,
      'Cache-Control': 'public, max-age=3600',
      'Last-Modified': lastModified,
    });

    fs.createReadStream(filePath).pipe(res);
  });
});

server.listen(8080, () => {
  console.log('静态文件服务运行在 http://localhost:8080/');
});
```

### 代码示例 3：请求体解析 + 大文件上传流式处理

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

// JSON 请求体解析
function parseJSONBody(req) {
  return new Promise((resolve, reject) => {
    const chunks = [];
    let size = 0;
    const MAX_SIZE = 10 * 1024 * 1024; // 10MB 上限

    req.on('data', (chunk) => {
      size += chunk.length;
      if (size > MAX_SIZE) {
        reject(new Error('请求体超过大小限制'));
        req.destroy(); // 中断连接
        return;
      }
      chunks.push(chunk);
    });

    req.on('end', () => {
      try {
        const body = Buffer.concat(chunks).toString('utf-8');
        if (!body) {
          resolve(null);
          return;
        }
        resolve(JSON.parse(body));
      } catch (err) {
        reject(new Error('无效的 JSON 格式'));
      }
    });

    req.on('error', reject);
  });
}

// 大文件流式上传处理
function handleFileUpload(req, res) {
  const uploadDir = path.join(__dirname, 'uploads');
  if (!fs.existsSync(uploadDir)) {
    fs.mkdirSync(uploadDir, { recursive: true });
  }

  const tmpName = `upload_${Date.now()}_${crypto.randomBytes(6).toString('hex')}`;
  const tmpPath = path.join(uploadDir, tmpName);
  const writeStream = fs.createWriteStream(tmpPath);
  let receivedBytes = 0;
  const MAX_FILE_SIZE = 500 * 1024 * 1024; // 500MB

  writeStream.on('error', (err) => {
    console.error('写入文件错误:', err);
    fs.unlinkSync(tmpPath); // 清理临时文件
    if (!res.headersSent) {
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: '文件写入失败' }));
    }
  });

  req.on('data', (chunk) => {
    receivedBytes += chunk.length;
    if (receivedBytes > MAX_FILE_SIZE) {
      writeStream.destroy();
      fs.unlinkSync(tmpPath);
      res.writeHead(413, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: '文件超过大小限制' }));
      return;
    }
    writeStream.write(chunk);
  });

  req.on('end', () => {
    writeStream.end(() => {
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        message: '上传成功',
        size: receivedBytes,
        filename: tmpName,
      }));
    });
  });

  req.on('error', (err) => {
    writeStream.destroy();
    fs.unlinkSync(tmpPath);
    console.error('上传请求错误:', err);
  });
}

const server = http.createServer(async (req, res) => {
  const parsedUrl = new URL(req.url, `http://${req.headers.host}`);

  try {
    // JSON API
    if (parsedUrl.pathname === '/api/data' && req.method === 'POST') {
      const body = await parseJSONBody(req);
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ received: body }));
      return;
    }

    // 文件上传
    if (parsedUrl.pathname === '/api/upload' && req.method === 'POST') {
      handleFileUpload(req, res);
      return;
    }

    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not Found' }));
  } catch (err) {
    console.error('请求处理失败:', err.message);
    if (!res.headersSent) {
      const status = err.message.includes('超过大小') ? 413
        : err.message.includes('JSON') ? 400 : 500;
      res.writeHead(status, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: err.message }));
    }
  }
});

server.listen(3000, () => {
  console.log('服务运行在 http://localhost:3000/');
});
```

### 代码示例 4：HTTPS + HTTP/2 配置

```javascript
const https = require('https');
const http2 = require('http2');
const fs = require('fs');
const path = require('path');

const tlsOptions = {
  key: fs.readFileSync(path.join(__dirname, 'certs/server-key.pem')),
  cert: fs.readFileSync(path.join(__dirname, 'certs/server-cert.pem')),
  // 安全配置
  minVersion: 'TLSv1.2',
  ciphers: [
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-ECDSA-CHACHA20-POLY1305',
    'ECDHE-RSA-CHACHA20-POLY1305',
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
  ].join(':'),
  honorCipherOrder: true,
};

// HTTP/1.1 HTTPS 服务（回退）
const httpsServer = https.createServer(tlsOptions, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('HTTPS (HTTP/1.1) 连接');
});

// HTTP/2 服务
const h2Server = http2.createSecureServer(tlsOptions);

h2Server.on('stream', (stream, headers) => {
  const path = headers[':path'];

  if (path === '/') {
    stream.respond({
      'content-type': 'text/html; charset=utf-8',
      ':status': 200,
    });
    stream.end('<h1>HTTP/2 Server</h1>');
  } else {
    stream.respond({ ':status': 404 });
    stream.end('Not Found');
  }
});

// HTTP → HTTPS 重定向
const httpServer = http.createServer((req, res) => {
  const host = req.headers.host || 'localhost';
  res.writeHead(301, { Location: `https://${host}${req.url}` });
  res.end();
});

httpsServer.listen(443, () => {
  console.log('HTTPS 服务运行在端口 443');
});

h2Server.listen(8443, () => {
  console.log('HTTP/2 服务运行在端口 8443');
});

httpServer.listen(80, () => {
  console.log('HTTP 重定向服务运行在端口 80');
});
```

### 踩坑指南

| 踩坑场景 | 现象 | 原因 | 解决方案 |
|----------|------|------|----------|
| 未调用 `res.end()` | 请求一直挂起，浏览器持续 loading | 响应未结束，连接保持 | 每个请求处理分支都必须调用 `res.end()`，包括错误分支 |
| `writeHead` 在 `write` 之后 | 抛出 `ERR_HTTP_HEADERS_SENT` | 响应头已发送后不能再修改 | 确保 `writeHead` 在 `write`/`end` 之前调用 |
| 请求体不完整 | `data` 事件丢失数据 | 在异步回调中才读取流，但流已结束 | 使用同步收集或确保在 `request` 事件中立即注册 `data` 监听 |
| 未处理 `clientError` | 进程崩溃或请求卡住 | 客户端发送格式错误的请求时，默认行为不确定 | 监听 `server.on('clientError')` 优雅处理 |
| 路径遍历攻击 | 任意文件可被读取 | 直接拼接 URL 路径到文件系统，`../` 可跳出目录 | 使用 `path.resolve` 后检查 `startsWith(STATIC_DIR)`，或使用 `path.normalize` |
| 大请求体 OOM | 服务器内存溢出 | 将整个请求体存入内存再处理 | 使用流式处理，设置大小限制，及时 `destroy()` |
| Keep-Alive 连接泄漏 | 服务器连接数持续增长 | 长连接未正确管理超时和关闭 | 合理配置 `keepAliveTimeout`，监听 `connection` 事件统计连接 |
| 多个 `res.end` | `ERR_STREAM_WRITE_AFTER_END` | 在某处调用 `end` 后，其他逻辑分支又调用 | 使用 `return` 确保单次返回，或检查 `res.writableEnded` |
| CRLF 注入 | 响应头被注入恶意内容 | 用户输入直接写入响应头，包含 `\r\n` | 对所有用户输入进行头部值消毒，过滤 `\r` 和 `\n` |
| ECONNRESET | 请求中断报错 | 客户端提前关闭连接，服务端还在写入 | 监听 `req.on('error')` 和 `res.on('error')`，忽略已关闭连接的写入错误 |

### 最佳实践

1. **始终处理错误事件**：`req`/`res`/`server` 都可能触发 `error` 事件，未处理的 `error` 事件会导致 Node.js 进程崩溃。务必监听 `server.on('clientError')`、`req.on('error')` 和 `res.on('error')`

2. **设置超时配置**：不要依赖默认超时值，生产环境应根据业务需要显式配置 `requestTimeout`、`headersTimeout`、`keepAliveTimeout`，防止慢速攻击（Slowloris）

3. **优雅关闭**：监听 `SIGTERM`/`SIGINT` 信号，调用 `server.close()` 停止接受新连接，等待现有请求完成后退出，设置强制退出超时兜底

4. **请求体大小限制**：始终设置请求体大小上限，防止恶意客户端发送超大请求导致 OOM。对于文件上传使用流式写入磁盘

5. **安全头部**：至少设置 `X-Content-Type-Options: nosniff`、`X-Frame-Options: DENY`、`Strict-Transport-Security` 等安全响应头，防止 MIME 嗅探和点击劫持

6. **连接数限制**：通过 `server.maxConnections` 限制最大并发连接数，防止连接风暴压垮服务器

7. **使用 `pipe()` 传输文件**：`fs.createReadStream().pipe(res)` 自动处理背压（backpressure），比手动 `readFile` + `res.end()` 更高效且内存友好

8. **统一错误响应格式**：无论是 404、500 还是业务错误，都应返回统一格式的 JSON 错误响应，包含 `error` 字段和可选的 `message` 字段

---

## 面试题

### 1. http 模块的核心 API 有哪些？createServer 返回什么？

`http.createServer()` 返回 `http.Server` 实例，它继承自 `net.Server`。核心 API 包括：
- `server.listen()`：绑定端口启动监听
- `server.close()`：停止接受新连接
- `server.on('request', callback)`：监听请求事件
- `server.on('upgrade', callback)`：监听协议升级事件
- `server.on('clientError', callback)`：监听客户端错误
- `server.timeout` / `server.requestTimeout`：超时配置
- `server.maxConnections`：最大连接数

`request` 回调接收 `IncomingMessage`（可读流）和 `ServerResponse`（可写流）两个参数。

### 2. 为什么请求体不能像 Express 那样直接通过 req.body 获取？

因为 `IncomingMessage` 继承自 `stream.Readable`，请求体数据是通过流（Stream）异步传输的，不会自动解析和缓存到某个属性。这是 Node.js 流式设计哲学的体现——数据到达时分块推送，避免将整个请求体载入内存。Express 的 `body-parser` 中间件本质上也是监听 `data` 和 `end` 事件收集完数据后，将结果挂载到 `req.body` 上。在原生 `http` 中，必须手动执行类似操作。

### 3. Keep-Alive 的工作原理和配置方式是什么？

Keep-Alive 是 HTTP/1.1 的持久连接机制，允许一个 TCP 连接上发送多个请求/响应，避免重复建立连接的开销。服务端通过 `keepAliveTimeout`（默认 5s）控制空闲连接的超时，客户端通过 `Connection: keep-alive` 头部请求保持连接。Node.js 中通过 `createServer` 的 `options.keepAliveTimeout` 配置超时，也可通过 `socket.setKeepAlive(true, interval)` 在 TCP 层发送探测包。注意：HTTP/2 默认多路复用，不需要 Keep-Alive。

### 4. 如何在原生 http 模块上实现路由？

原生 http 没有内置路由，需要手动实现。基本思路是：解析 `req.method` 和 `req.url`（使用 `URL` 构造器解析路径和查询参数），然后根据 Method + Path 映射到处理函数。进阶实现包括：用正则表达式支持路径参数（`:id`）、支持通配符、实现中间件链式调用。实际生产中推荐直接使用 Express/Fastify 的路由功能。

### 5. 静态文件服务中如何正确处理 MIME 类型？

MIME 类型通过文件扩展名映射确定，需要维护一个扩展名到 MIME 类型的映射表。Node.js 内置的 `mime` 类型有限，推荐使用 `mime-types` npm 包（基于 Apache 的 mime.types 数据库），或手动维护核心映射。注意事项：① 默认使用 `application/octet-stream`（触发下载）；② 文本类型必须指定 `charset=utf-8`；③ 使用 `X-Content-Type-Options: nosniff` 防止浏览器 MIME 嗅探导致的安全问题。

### 6. HTTP/2 多路复用的原理是什么？

HTTP/2 在应用层引入了"流"（Stream）的概念，每个请求/响应对应一个流，流内部划分为更小的"帧"（Frame）。多个流的帧可以在同一个 TCP 连接上交错传输，接收端根据流标识符将帧重新组装为完整的消息。这解决了 HTTP/1.x 的队头阻塞问题（一个慢请求阻塞后续所有请求）。同时，HTTP/2 使用 HPACK 算法压缩头部，进一步减少带宽消耗。但 HTTP/2 仍存在 TCP 层面的队头阻塞——一个 TCP 包丢失会阻塞所有流。

### 7. 大文件上传如何做流式处理？

核心思路是避免将整个文件加载到内存，使用流（Stream）边读边写。具体步骤：① 创建 `fs.createWriteStream` 指向临时文件；② 在 `req.on('data')` 中将每个 chunk 写入文件流，同时累加已接收字节数；③ 超过大小限制时销毁写入流、删除临时文件并返回 413；④ `req.on('end')` 时关闭写入流并返回成功响应。还可以结合 `crypto.createHash('md5')` 在接收过程中同步计算文件校验和。

### 8. HTTPS/TLS 配置的要点有哪些？

关键配置点：① 证书和私钥：通过 `key` 和 `cert` 选项加载 PEM 格式文件；② TLS 版本：设置 `minVersion: 'TLSv1.2'`，禁用不安全的 SSLv3/TLS1.0/1.1；③ 密码套件：配置 `ciphers` 优先使用 ECDHE + AES-GCM/CHACHA20 等前向保密套件；④ `honorCipherOrder: true` 让服务端决定密码套件优先级；⑤ 使用 `passphrase` 保护加密的私钥；⑥ 配置 `ca` 证书链确保中间证书完整；⑦ 启用 `OCSP Stapling` 减少证书验证延迟；⑧ 生产环境配置 `Strict-Transport-Security` 响应头强制 HTTPS。

---

## 相关链接

- [[Express]] — 基于 http 模块的最流行 Web 框架，提供路由、中间件等高级抽象
- [[Fastify]] — 高性能 Web 框架，基于 JSON Schema 验证，吞吐量接近原生 http
- [[Stream与Buffer]] — 理解流和缓冲区是掌握 HTTP 请求/响应处理的基础
- [Node.js HTTP 官方文档](https://nodejs.org/api/http.html) — http 模块 API 参考

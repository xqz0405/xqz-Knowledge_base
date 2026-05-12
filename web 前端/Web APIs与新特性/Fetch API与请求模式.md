---
tags:
  - Web前端
  - Web APIs
  - Fetch
  - HTTP
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Fetch API 与请求模式

## What — 什么是 Fetch API

Fetch API 是浏览器提供的现代 HTTP 客户端接口，用于发起网络请求并处理响应。它是 `XMLHttpRequest`（XHR）的替代方案，基于 Promise 设计，提供了更简洁、更强大的请求处理能力。

### 核心对象

| 对象 | 说明 |
|------|------|
| `fetch()` | 发起请求的全局函数，返回 Promise |
| `Request` | 请求对象，封装 URL、方法、头、体等信息 |
| `Response` | 响应对象，封装状态码、头、体等信息 |
| `Headers` | 请求/响应头对象，支持增删改查 |
| `AbortController` | 请求取消控制器 |
| `AbortSignal` | 取消信号，传递给 Request |

### 最简示例

```js
// 发起 GET 请求
const response = await fetch('https://api.example.com/users');
const data = await response.json();
console.log(data);
```

### 与 XMLHttpRequest 对比

```js
// XMLHttpRequest —— 回调嵌套，代码冗长
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/users');
xhr.onload = function () {
  if (xhr.status >= 200 && xhr.status < 300) {
    const data = JSON.parse(xhr.responseText);
    console.log(data);
  }
};
xhr.onerror = function () {
  console.error('请求失败');
};
xhr.send();

// Fetch —— 简洁直观，基于 Promise
const response = await fetch('https://api.example.com/users');
if (response.ok) {
  const data = await response.json();
  console.log(data);
}
```

---

## Why — 为什么需要 Fetch API

### 1. XHR 的回调地狱

XHR 基于事件回调，多个请求串联时产生深层嵌套：

```js
// XHR 回调地狱
xhr1.onload = function () {
  xhr2.onload = function () {
    xhr3.onload = function () {
      // 三层嵌套，可读性极差
    };
    xhr3.send();
  };
  xhr2.send();
};
xhr1.send();
```

Fetch 基于 Promise，天然支持链式调用和 `async/await`：

```js
// Fetch —— 扁平化、可读性强
const res1 = await fetch('/api/step1');
const data1 = await res1.json();
const res2 = await fetch('/api/step2', {
  method: 'POST',
  body: JSON.stringify(data1)
});
const data2 = await res2.json();
```

### 2. XHR API 设计过时

XHR 在 2000 年代初设计，API 存在诸多缺陷：

| 问题 | XHR | Fetch |
|------|-----|-------|
| 接口风格 | 事件回调 | Promise + async/await |
| 状态管理 | `readyState` 四个状态 | Promise resolve/reject |
| 请求构造 | `open()` + `setRequestHeader()` + `send()` 分步 | 一个 `fetch()` 调用 |
| 响应解析 | 手动 `JSON.parse(xhr.responseText)` | `response.json()` 等方法 |
| 不可变请求 | 无法冻结请求配置 | `Request` 对象可复用、可克隆 |

### 3. 流式读取支持

Fetch 原生支持 `ReadableStream`，可以分块读取响应体，这是 XHR 无法做到的：

```js
// 流式读取大文件，不阻塞内存
const response = await fetch('/api/large-file');
const reader = response.body.getReader();
let done = false;
while (!done) {
  const { value, done: readerDone } = await reader.read();
  done = readerDone;
  if (value) {
    // 逐块处理，无需等待全部下载
    processChunk(value);
  }
}
```

### 4. Service Worker 必需

Service Worker 拦截请求使用的是 Fetch 事件模型，请求和响应都是 `Request`/`Response` 对象：

```js
// Service Worker 拦截请求
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      return cachedResponse || fetch(event.request);
    })
  );
});
```

---

## Fetch vs XHR vs Axios 对比

| 维度 | Fetch | XHR | Axios |
|------|-------|-----|-------|
| **语法** | Promise/async-await | 事件回调 | Promise/async-await |
| **HTTP 错误处理** | 不 reject（需手动判断 `ok`） | `onerror` 仅网络错误 | 4xx/5xx 自动 reject |
| **请求超时** | 需 `AbortController` + `setTimeout` | `xhr.timeout` 原生支持 | `timeout` 配置项 |
| **进度监控** | 无原生支持（流式可模拟） | `onprogress` 原生支持 | 无（浏览器端） |
| **拦截器** | 需手动封装 | 需手动封装 | 请求/响应拦截器内置 |
| **请求取消** | `AbortController` | `xhr.abort()` | `CancelToken` / `AbortController` |
| **流式读取** | `ReadableStream` 原生支持 | 不支持 | 不支持 |
| **Cookie 发送** | 默认不发送跨域 Cookie | 默认发送 | 默认发送同域 |
| **响应数据转换** | 手动调用 `.json()` 等 | 手动解析 | 自动转换 |
| **兼容性** | IE 不支持，其余全支持 | 全浏览器 | 全浏览器 |
| **体积** | 浏览器原生，0KB | 浏览器原生，0KB | ~13KB（gzip） |
| **Node.js** | Node 18+ 内置 | 不支持 | 支持 |

### 何时选什么

| 场景 | 推荐 | 理由 |
|------|------|------|
| 现代浏览器项目 | Fetch | 原生、零依赖、流式支持 |
| 需要上传进度 | XHR | 原生 `onprogress` 支持 |
| 需要拦截器 + 自动错误处理 | Axios | 开箱即用的完整功能 |
| Service Worker | Fetch | 唯一选择 |
| 流式响应/SSE | Fetch | 唯一原生支持流式读取 |
| Node.js + 浏览器同构 | Axios | 统一 API |

---

## How — 怎么用

### 1. fetch 基础

#### GET 请求

```js
// 最简 GET
const response = await fetch('/api/users');
const users = await response.json();

// 带查询参数
const params = new URLSearchParams({
  page: 1,
  size: 10,
  keyword: 'fetch'
});
const response = await fetch(`/api/users?${params}`);
const data = await response.json();

// 带请求头
const response = await fetch('/api/users', {
  headers: {
    'Accept': 'application/json',
    'Authorization': 'Bearer token123'
  }
});
```

#### POST 请求

```js
// JSON 数据
const response = await fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: '张三',
    age: 25
  })
});

// 表单数据
const formData = new FormData();
formData.append('name', '张三');
formData.append('age', '25');
const response = await fetch('/api/users', {
  method: 'POST',
  body: formData  // FormData 不需要设置 Content-Type，浏览器自动设置
});

// URL 编码数据
const params = new URLSearchParams();
params.append('name', '张三');
params.append('age', '25');
const response = await fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: params.toString()
});
```

#### PUT / PATCH / DELETE

```js
// PUT —— 全量更新
const response = await fetch('/api/users/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: '李四', age: 30, email: 'lisi@example.com' })
});

// PATCH —— 部分更新
const response = await fetch('/api/users/1', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ age: 31 }) // 只更新 age
});

// DELETE
const response = await fetch('/api/users/1', {
  method: 'DELETE'
});
```

#### Headers 对象

```js
// 创建 Headers
const headers = new Headers({
  'Content-Type': 'application/json',
  'Authorization': 'Bearer token123'
});

// 增删改查
headers.append('X-Custom', 'value');   // 追加
headers.set('Content-Type', 'text/plain'); // 覆盖
headers.get('Content-Type');            // 'text/plain'
headers.has('Authorization');           // true
headers.delete('X-Custom');             // 删除

// 遍历
for (const [key, value] of headers) {
  console.log(`${key}: ${value}`);
}

// Headers 合并
const h1 = new Headers({ 'A': '1' });
const h2 = new Headers({ 'B': '2' });
const merged = new Headers([...h1, ...h2]);
```

#### body 支持的类型

| 类型 | Content-Type | 说明 |
|------|-------------|------|
| `string` | 手动指定 | 纯文本 |
| `JSON.stringify(data)` | `application/json` | JSON 数据 |
| `FormData` | `multipart/form-data`（自动） | 文件上传、表单 |
| `URLSearchParams` | `application/x-www-form-urlencoded` | URL 编码表单 |
| `Blob` | 手动指定 | 二进制大对象 |
| `ArrayBuffer` | 手动指定 | 原始二进制 |
| `ReadableStream` | 手动指定 | 流式上传 |

```js
// Blob 上传
const blob = new Blob(['Hello, World!'], { type: 'text/plain' });
const response = await fetch('/api/upload', {
  method: 'POST',
  body: blob
});

// ArrayBuffer 上传
const buffer = new TextEncoder().encode('Hello');
const response = await fetch('/api/upload', {
  method: 'POST',
  headers: { 'Content-Type': 'application/octet-stream' },
  body: buffer
});
```

---

### 2. Request 对象

`Request` 对象封装了请求的全部信息，可复用、可克隆。

```js
// 创建 Request
const request = new Request('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: '张三' })
});

// 使用 Request 发起请求
const response = await fetch(request);

// Request 复用 —— 基于旧 Request 创建新请求
const newRequest = new Request(request, {
  headers: {
    'Content-Type': 'text/plain' // 覆盖 headers
  }
  // 其他属性继承自 request
});
```

#### Request 完整配置项

```js
const request = new Request(url, {
  method: 'POST',            // 请求方法：GET/POST/PUT/PATCH/DELETE/HEAD/OPTIONS
  headers: {                 // 请求头，Headers 对象或普通对象
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({}),  // 请求体，GET/HEAD 不能有 body
  mode: 'cors',              // 请求模式：cors / no-cors / same-origin / navigate
  credentials: 'same-origin',// Cookie 策略：omit / same-origin / include
  cache: 'default',          // 缓存模式：default / no-store / reload / no-cache / force-cache / only-if-cached
  redirect: 'follow',        // 重定向策略：follow / error / manual
  referrer: 'about:client',  // 来源：URL / '' / 'about:client' / 'no-referrer'
  referrerPolicy: 'no-referrer-when-downgrade', // 来源策略
  integrity: '',             // 子资源完整性校验（SRI）
  keepalive: false,          // 长连接（页面卸载后仍可发送）
  signal: abortController.signal, // AbortSignal，用于取消请求
  priority: 'auto'           // 优先级：high / low / auto
});
```

#### mode 详解

| mode | 说明 |
|------|------|
| `cors` | 跨域请求（默认），需要服务器返回 CORS 头 |
| `no-cors` | 不透明请求，只能发送简单请求，响应 type 为 `opaque`，无法读取内容 |
| `same-origin` | 只允许同源请求，跨域直接报错 |
| `navigate` | 导航请求，浏览器自动使用 |

#### cache 详解

| cache 值 | 行为 |
|----------|------|
| `default` | 有缓存且未过期则用缓存，过期则条件请求 |
| `no-store` | 完全不使用缓存，也不缓存响应 |
| `reload` | 不使用缓存，但响应会被缓存 |
| `no-cache` | 有缓存也发条件请求验证 |
| `force-cache` | 有缓存就用，即使过期 |
| `only-if-cached` | 只用缓存，无缓存则 504（仅 mode 为 `same-origin`） |

---

### 3. Response 对象

```js
const response = await fetch('/api/users');

// 状态信息
response.status;       // 200（HTTP 状态码，数字）
response.ok;           // true（status 在 200-299 之间）
response.statusText;   // 'OK'（状态描述）
response.type;         // 'basic' / 'cors' / 'opaque' / 'opaqueredirect'
response.url;          // 最终 URL（经过重定向后）
response.redirected;   // 是否经过了重定向

// 响应头
response.headers.get('Content-Type');       // 'application/json'
response.headers.get('X-Request-Id');       // 'abc123'
response.headers.has('Set-Cookie');         // true
for (const [key, value] of response.headers) {
  console.log(`${key}: ${value}`);
}

// 响应体 —— 只能读取一次
const json = await response.json();         // 解析为 JSON 对象
const text = await response.text();         // 解析为字符串
const blob = await response.blob();         // 解析为 Blob
const arrayBuffer = await response.arrayBuffer(); // 解析为 ArrayBuffer
const formData = await response.formData(); // 解析为 FormData
```

#### Response type

| type | 说明 |
|------|------|
| `basic` | 同源请求的响应，可访问全部头 |
| `cors` | 跨域请求的响应，只能访问 CORS 暴露的头 |
| `opaque` | `no-cors` 跨域请求的响应，无法读取任何内容 |
| `opaqueredirect` | `redirect: 'manual'` 时的重定向响应 |

#### clone() —— 复制 Response

Response 的 body 只能读取一次，读取后再读会报错。需要多次使用时必须 `clone()`：

```js
const response = await fetch('/api/users');

// 错误：body 已被消费
const data1 = await response.json();
const data2 = await response.json(); // TypeError: Already read

// 正确：先克隆再读取
const response = await fetch('/api/users');
const cloned = response.clone();
const data1 = await response.json();
const data2 = await cloned.json(); // OK
```

#### 手动构造 Response

```js
// 构造 Response 对象（Service Worker 中常用）
const response = new Response(JSON.stringify({ message: 'Hello' }), {
  status: 200,
  statusText: 'OK',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Response.error() —— 网络错误的 Response
const errorResponse = Response.error();

// Response.redirect() —— 重定向的 Response
const redirectResponse = Response.redirect('/login', 302);
```

---

### 4. 认证与 Cookie

#### credentials 三种模式

| credentials 值 | 同源 Cookie | 跨域 Cookie | 说明 |
|----------------|------------|------------|------|
| `omit` | 不发送 | 不发送 | 从不发送 Cookie |
| `same-origin`（默认） | 发送 | 不发送 | 仅同源发送 Cookie |
| `include` | 发送 | 发送 | 始终发送 Cookie |

```js
// 同源请求 —— 默认发送 Cookie
const res1 = await fetch('/api/me'); // credentials: 'same-origin'

// 跨域请求 —— 默认不发送 Cookie！
// 服务器必须设置：Access-Control-Allow-Credentials: true
// 且 Access-Control-Allow-Origin 不能为 *
const res2 = await fetch('https://other-api.com/me', {
  credentials: 'include' // 必须显式指定
});
```

#### Authorization 头与 Bearer Token

```js
// Bearer Token 认证
const token = localStorage.getItem('access_token');
const response = await fetch('/api/protected', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// Token 刷新模式
async function fetchWithAuth(url, options = {}) {
  let token = localStorage.getItem('access_token');

  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${token}`
    }
  });

  // Token 过期，尝试刷新
  if (response.status === 401) {
    const refreshToken = localStorage.getItem('refresh_token');
    const refreshRes = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken })
    });

    if (refreshRes.ok) {
      const { accessToken } = await refreshRes.json();
      localStorage.setItem('access_token', accessToken);

      // 用新 Token 重试原请求
      return fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${accessToken}`
        }
      });
    }

    throw new Error('认证失败，请重新登录');
  }

  return response;
}
```

#### Basic Auth

```js
// Basic 认证
const username = 'admin';
const password = 'secret';
const encoded = btoa(`${username}:${password}`);

const response = await fetch('/api/admin', {
  headers: {
    'Authorization': `Basic ${encoded}`
  }
});
```

---

### 5. 错误处理

#### Fetch 的"陷阱"：HTTP 错误不 reject

```js
// 陷阱：4xx/5xx 不会触发 reject！
try {
  const response = await fetch('/api/not-found');
  // 即使 404，fetch 也会 resolve！
  console.log(response.status); // 404
  console.log(response.ok);     // false
} catch (error) {
  // 只有网络错误才会到这里
  console.error(error); // TypeError: Failed to fetch
}
```

#### 只有这些情况才会 reject

| 情况 | 是否 reject |
|------|------------|
| 网络断开 | reject |
| DNS 解析失败 | reject |
| CORS 被拒绝 | reject |
| 请求被 abort | reject（AbortError） |
| URL 格式错误 | reject |
| 200 OK | resolve |
| 301/302 重定向 | resolve（自动跟随） |
| 400 Bad Request | resolve（需手动判断） |
| 401 Unauthorized | resolve（需手动判断） |
| 404 Not Found | resolve（需手动判断） |
| 500 Internal Server Error | resolve（需手动判断） |

#### 统一错误处理封装

```js
class HttpError extends Error {
  constructor(message, status, response) {
    super(message);
    this.name = 'HttpError';
    this.status = status;
    this.response = response;
  }
}

async function fetchJson(url, options = {}) {
  const response = await fetch(url, {
    headers: {
      'Accept': 'application/json',
      ...options.headers
    },
    ...options
  });

  // HTTP 错误统一抛出
  if (!response.ok) {
    let errorBody;
    try {
      errorBody = await response.json();
    } catch {
      errorBody = await response.text();
    }
    throw new HttpError(
      errorBody?.message || `HTTP ${response.status}`,
      response.status,
      errorBody
    );
  }

  // 204 No Content 无响应体
  if (response.status === 204) {
    return null;
  }

  return response.json();
}

// 使用
try {
  const user = await fetchJson('/api/users/1');
  console.log(user);
} catch (error) {
  if (error instanceof HttpError) {
    console.error(`HTTP ${error.status}: ${error.message}`);
    if (error.status === 401) {
      // 跳转登录
      window.location.href = '/login';
    }
  } else {
    console.error('网络错误:', error.message);
  }
}
```

---

### 6. 超时处理

Fetch 没有原生的 `timeout` 配置，需要用 `AbortController` + `setTimeout` 实现：

```js
// 基础超时
async function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal
    });
    return response;
  } finally {
    clearTimeout(timeoutId);
  }
}

// 使用
try {
  const data = await fetchWithTimeout('/api/slow', {}, 3000);
  console.log(await data.json());
} catch (error) {
  if (error.name === 'AbortError') {
    console.error('请求超时');
  } else {
    console.error('请求失败:', error);
  }
}
```

#### 区分超时与手动取消

```js
async function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => {
    controller.abort(new DOMException('请求超时', 'TimeoutError'));
  }, timeout);

  try {
    return await fetch(url, { ...options, signal: controller.signal });
  } catch (error) {
    if (error.name === 'TimeoutError') {
      throw new Error(`请求超时（${timeout}ms）: ${url}`);
    }
    throw error;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

---

### 7. 请求取消

#### AbortController 基础

```js
const controller = new AbortController();

fetch('/api/users', { signal: controller.signal })
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('请求已取消');
    }
  });

// 取消请求
controller.abort();
```

#### 取消多个请求

```js
const controller = new AbortController();
const signal = controller.signal;

// 多个请求共享同一个 signal
const [users, posts] = await Promise.all([
  fetch('/api/users', { signal }),
  fetch('/api/posts', { signal })
]);

// 一次性取消全部
controller.abort();
```

#### React 中取消请求

```js
function useFetch(url) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url, { signal: controller.signal });
        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    // 组件卸载时取消请求
    return () => controller.abort();
  }, [url]);

  return { data, error, loading };
}
```

#### abort 原因

```js
// 传入 abort 原因（Chrome 98+）
const controller = new AbortController();
controller.abort(new Error('用户手动取消'));

fetch('/api/users', { signal: controller.signal })
  .catch(err => {
    console.log(err.name);    // 'AbortError'
    console.log(err.cause);   // Error: 用户手动取消（如果支持）
  });

// 也可以用 AbortSignal.timeout() —— 原生超时（Chrome 103+）
const response = await fetch('/api/slow', {
  signal: AbortSignal.timeout(3000) // 3秒超时
});
```

#### AbortSignal.any() —— 组合多个信号

```js
// 组合超时信号和手动取消信号（Chrome 116+）
const manualController = new AbortController();
const timeoutSignal = AbortSignal.timeout(5000);
const combinedSignal = AbortSignal.any([
  manualController.signal,
  timeoutSignal
]);

fetch('/api/data', { signal: combinedSignal });

// 任何一个信号触发都会取消
manualController.abort(); // 手动取消
// 或 5 秒后自动超时取消
```

---

### 8. 重试模式

#### 指数退避重试

```js
async function fetchWithRetry(url, options = {}, retries = 3) {
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.ok) {
        return response;
      }

      // 5xx 服务器错误才重试，4xx 客户端错误不重试
      if (response.status >= 500 && attempt < retries) {
        const delay = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s
        const jitter = Math.random() * 1000;       // 随机抖动避免惊群
        await sleep(delay + jitter);
        continue;
      }

      return response; // 4xx 或重试耗尽，返回响应
    } catch (error) {
      // 网络错误才重试
      if (attempt < retries) {
        const delay = Math.pow(2, attempt) * 1000;
        await sleep(delay);
        continue;
      }
      throw error; // 重试耗尽，抛出错误
    }
  }
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// 使用
const response = await fetchWithRetry('/api/unstable', {}, 3);
```

#### 幂等方法判断

```js
const IDEMPOTENT_METHODS = ['GET', 'HEAD', 'OPTIONS', 'PUT', 'DELETE'];

async function fetchWithSmartRetry(url, options = {}, retries = 3) {
  const method = (options.method || 'GET').toUpperCase();
  const isIdempotent = IDEMPOTENT_METHODS.includes(method);

  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.ok) return response;

      // 非幂等方法（POST/PATCH）只在 5xx 且确认安全时重试
      if (!isIdempotent && response.status !== 503) {
        return response; // 不重试
      }

      if (response.status >= 500 && attempt < retries) {
        const delay = Math.pow(2, attempt) * 1000 + Math.random() * 500;
        await sleep(delay);
        continue;
      }

      return response;
    } catch (error) {
      if (!isIdempotent && attempt > 0) {
        throw error; // POST 网络错误可能已执行，谨慎重试
      }
      if (attempt < retries) {
        await sleep(Math.pow(2, attempt) * 1000);
        continue;
      }
      throw error;
    }
  }
}
```

---

### 9. 请求拦截器模式

Fetch 没有内置拦截器，但可以通过包装 fetch 来实现类似 Axios 的拦截器功能：

```js
class FetchInterceptor {
  constructor() {
    this.requestInterceptors = [];
    this.responseInterceptors = [];
    this.originalFetch = window.fetch.bind(window);
  }

  // 添加请求拦截器
  addRequestInterceptor(onFulfilled, onRejected) {
    this.requestInterceptors.push({ onFulfilled, onRejected });
    return this; // 链式调用
  }

  // 添加响应拦截器
  addResponseInterceptor(onFulfilled, onRejected) {
    this.responseInterceptors.push({ onFulfilled, onRejected });
    return this;
  }

  // 执行请求拦截器链
  async runRequestInterceptors(url, options) {
    let config = { url, options };

    for (const interceptor of this.requestInterceptors) {
      try {
        config = await interceptor.onFulfilled(config);
      } catch (error) {
        if (interceptor.onRejected) {
          config = await interceptor.onRejected(error);
        } else {
          throw error;
        }
      }
    }

    return config;
  }

  // 执行响应拦截器链
  async runResponseInterceptors(response) {
    let res = response;

    for (const interceptor of this.responseInterceptors) {
      try {
        res = await interceptor.onFulfilled(res);
      } catch (error) {
        if (interceptor.onRejected) {
          res = await interceptor.onRejected(error);
        } else {
          throw error;
        }
      }
    }

    return res;
  }

  // 包装后的 fetch
  async fetch(url, options = {}) {
    // 1. 请求拦截
    const config = await this.runRequestInterceptors(url, options);

    // 2. 发起请求
    let response = await this.originalFetch(config.url, config.options);

    // 3. 响应拦截
    response = await this.runResponseInterceptors(response);

    return response;
  }

  // 便捷方法
  get(url, options = {}) {
    return this.fetch(url, { ...options, method: 'GET' });
  }

  post(url, body, options = {}) {
    return this.fetch(url, {
      ...options,
      method: 'POST',
      headers: { 'Content-Type': 'application/json', ...options.headers },
      body: JSON.stringify(body)
    });
  }

  put(url, body, options = {}) {
    return this.fetch(url, {
      ...options,
      method: 'PUT',
      headers: { 'Content-Type': 'application/json', ...options.headers },
      body: JSON.stringify(body)
    });
  }

  delete(url, options = {}) {
    return this.fetch(url, { ...options, method: 'DELETE' });
  }
}
```

#### 使用拦截器

```js
const http = new FetchInterceptor();

// 请求拦截器：添加 Token
http.addRequestInterceptor((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.options.headers = {
      ...config.options.headers,
      'Authorization': `Bearer ${token}`
    };
  }
  return config;
});

// 请求拦截器：添加时间戳防缓存
http.addRequestInterceptor((config) => {
  if (config.options.method === 'GET') {
    const separator = config.url.includes('?') ? '&' : '?';
    config.url = `${config.url}${separator}_t=${Date.now()}`;
  }
  return config;
});

// 响应拦截器：统一错误处理
http.addResponseInterceptor((response) => {
  if (!response.ok) {
    throw new HttpError(
      `HTTP ${response.status}`,
      response.status,
      response
    );
  }
  return response;
});

// 响应拦截器：自动解析 JSON
http.addResponseInterceptor(async (response) => {
  const contentType = response.headers.get('Content-Type');
  if (contentType && contentType.includes('application/json')) {
    const data = await response.json();
    return { data, status: response.status, headers: response.headers };
  }
  return response;
});

// 响应拦截器：Token 过期自动刷新
http.addResponseInterceptor(
  async (response) => response,
  async (error) => {
    if (error.status === 401) {
      const newToken = await refreshToken();
      if (newToken) {
        // 重试原请求
        return http.fetch(error.response.url);
      }
    }
    throw error;
  }
);

// 使用
const result = await http.get('/api/users');
const created = await http.post('/api/users', { name: '张三' });
```

---

### 10. 并发请求

#### Promise.all —— 全部成功才成功

```js
// 并发请求，全部完成后一起处理
const [usersRes, postsRes, commentsRes] = await Promise.all([
  fetch('/api/users'),
  fetch('/api/posts'),
  fetch('/api/comments')
]);

const users = await usersRes.json();
const posts = await postsRes.json();
const comments = await commentsRes.json();
```

#### Promise.allSettled —— 全部完成（无论成败）

```js
// 不在乎部分失败，获取全部结果
const results = await Promise.allSettled([
  fetch('/api/users'),
  fetch('/api/unstable'),  // 可能失败
  fetch('/api/comments')
]);

results.forEach((result) => {
  if (result.status === 'fulfilled') {
    console.log('成功:', result.value);
  } else {
    console.error('失败:', result.reason);
  }
});
```

#### Promise.race —— 最快的那个

```js
// 超时模式：谁先完成用谁
const response = await Promise.race([
  fetch('/api/fast-mirror'),
  fetch('/api/slow-mirror')
]);
```

#### Promise.any —— 第一个成功的

```js
// 多源容灾：任一成功即可
const response = await Promise.any([
  fetch('https://cdn1.example.com/data.json'),
  fetch('https://cdn2.example.com/data.json'),
  fetch('https://cdn3.example.com/data.json')
]);
```

#### 批量请求控制并发数

```js
async function batchFetch(urls, options = {}, concurrency = 5) {
  const results = new Array(urls.length);
  let index = 0;

  async function worker() {
    while (index < urls.length) {
      const currentIndex = index++;
      try {
        results[currentIndex] = {
          status: 'fulfilled',
          value: await fetch(urls[currentIndex], options)
        };
      } catch (error) {
        results[currentIndex] = {
          status: 'rejected',
          reason: error
        };
      }
    }
  }

  // 创建 concurrency 个 worker 并行执行
  const workers = Array.from({ length: Math.min(concurrency, urls.length) }, () => worker());
  await Promise.all(workers);

  return results;
}

// 使用：10 个请求，最多同时 3 个
const urls = Array.from({ length: 10 }, (_, i) => `/api/items/${i + 1}`);
const results = await batchFetch(urls, {}, 3);
```

#### 更优雅的并发控制（基于队列）

```js
class ConcurrencyPool {
  constructor(limit) {
    this.limit = limit;
    this.running = 0;
    this.queue = [];
  }

  async add(fn) {
    if (this.running >= this.limit) {
      await new Promise(resolve => this.queue.push(resolve));
    }

    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
      if (this.queue.length > 0) {
        this.queue.shift()(); // 唤醒下一个等待的任务
      }
    }
  }
}

// 使用
const pool = new ConcurrencyPool(3);
const results = await Promise.allSettled(
  urls.map(url => pool.add(() => fetch(url).then(r => r.json())))
);
```

---

### 11. 流式读取

#### ReadableStream 基础

```js
const response = await fetch('/api/stream');

// 获取 ReadableStream
const stream = response.body;
const reader = stream.getReader();

// 分块读取
let chunks = [];
while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  chunks.push(value);
  console.log(`收到 ${value.length} 字节`);
}

// 合并所有块
const totalLength = chunks.reduce((sum, chunk) => sum + chunk.length, 0);
const result = new Uint8Array(totalLength);
let offset = 0;
for (const chunk of chunks) {
  result.set(chunk, offset);
  offset += chunk.length;
}
const text = new TextDecoder().decode(result);
console.log(text);
```

#### TextDecoderStream —— 流式文本解码

```js
const response = await fetch('/api/stream');
const reader = response.body
  .pipeThrough(new TextDecoderStream())
  .getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  console.log(value); // 直接是字符串
}
```

#### SSE（Server-Sent Events）事件流

```js
async function readSSE(url) {
  const response = await fetch(url, {
    headers: { 'Accept': 'text/event-stream' }
  });

  const reader = response.body
    .pipeThrough(new TextDecoderStream())
    .getReader();

  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += value;
    const lines = buffer.split('\n');
    buffer = lines.pop(); // 未完成的行保留

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') return;
        try {
          const parsed = JSON.parse(data);
          console.log('SSE 事件:', parsed);
        } catch {
          console.log('SSE 文本:', data);
        }
      }
    }
  }
}

// 调用 SSE 流（如 ChatGPT 流式响应）
readSSE('/api/chat/stream');
```

#### 大文件流式下载 + 进度

```js
async function downloadWithProgress(url, filename) {
  const response = await fetch(url);
  const contentLength = +response.headers.get('Content-Length');
  let receivedLength = 0;

  const reader = response.body.getReader();
  const chunks = [];

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    chunks.push(value);
    receivedLength += value.length;

    // 计算进度
    const progress = contentLength
      ? Math.round((receivedLength / contentLength) * 100)
      : 0;
    console.log(`下载进度: ${progress}% (${receivedLength}/${contentLength})`);
  }

  // 合并并下载
  const blob = new Blob(chunks);
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = filename;
  a.click();
  URL.revokeObjectURL(a.href);
}
```

#### 流式处理 TransformStream

```js
// 将流式 JSON NDJSON 逐行解析
async function readNDJSON(url) {
  const response = await fetch(url);

  const ndjsonParser = new TransformStream({
    buffer: '',
    transform(chunk, controller) {
      this.buffer += chunk;
      const lines = this.buffer.split('\n');
      this.buffer = lines.pop();
      for (const line of lines) {
        if (line.trim()) {
          controller.enqueue(JSON.parse(line));
        }
      }
    },
    flush(controller) {
      if (this.buffer.trim()) {
        controller.enqueue(JSON.parse(this.buffer));
      }
    }
  });

  const reader = response.body
    .pipeThrough(new TextDecoderStream())
    .pipeThrough(ndjsonParser)
    .getReader();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log('NDJSON 对象:', value);
  }
}
```

---

### 12. 文件上传

#### FormData 基础上传

```js
// 单文件上传
const input = document.querySelector('input[type="file"]');
const file = input.files[0];

const formData = new FormData();
formData.append('file', file);
formData.append('description', '用户头像');

const response = await fetch('/api/upload', {
  method: 'POST',
  body: formData // 不设置 Content-Type，浏览器自动添加 boundary
});
```

#### 多文件上传

```js
// 多文件上传
const input = document.querySelector('input[type="file"]');
const files = input.files;

const formData = new FormData();
for (const file of files) {
  formData.append('files', file); // 同名 append，后端接收数组
}
formData.append('category', 'documents');

const response = await fetch('/api/upload/batch', {
  method: 'POST',
  body: formData
});
```

#### 拖拽上传

```js
const dropZone = document.getElementById('drop-zone');

dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('drag-over');
});

dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');

  const files = e.dataTransfer.files;
  const formData = new FormData();
  for (const file of files) {
    formData.append('files', file);
  }

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData
  });
  console.log('上传结果:', await response.json());
});
```

#### 上传进度 —— 用 XHR 补充

Fetch 不支持上传进度监控，需要上传进度时用 XHR：

```js
function uploadWithProgress(url, formData, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('POST', url);

    xhr.upload.onprogress = (e) => {
      if (e.lengthComputable) {
        const percent = Math.round((e.loaded / e.total) * 100);
        onProgress(percent);
      }
    };

    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`上传失败: HTTP ${xhr.status}`));
      }
    };

    xhr.onerror = () => reject(new Error('网络错误'));
    xhr.send(formData);
  });
}

// 使用
const result = await uploadWithProgress(
  '/api/upload',
  formData,
  (percent) => console.log(`上传进度: ${percent}%`)
);
```

#### 大文件分片上传

```js
async function uploadLargeFile(file, chunkSize = 5 * 1024 * 1024) {
  const totalChunks = Math.ceil(file.size / chunkSize);

  for (let i = 0; i < totalChunks; i++) {
    const start = i * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);

    const formData = new FormData();
    formData.append('file', chunk);
    formData.append('chunkIndex', i);
    formData.append('totalChunks', totalChunks);
    formData.append('filename', file.name);

    const response = await fetch('/api/upload/chunk', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error(`分片 ${i} 上传失败`);
    }

    console.log(`分片 ${i + 1}/${totalChunks} 上传完成`);
  }

  // 通知服务器合并
  const mergeRes = await fetch('/api/upload/merge', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      filename: file.name,
      totalChunks,
      chunkSize
    })
  });

  return mergeRes.json();
}
```

---

## 常见问题

### 1. CORS 错误

```
Access to fetch at 'https://api.other.com/data' from origin 'https://myapp.com'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is
present on the requested resource.
```

**原因：** 浏览器同源策略限制，跨域请求需要服务器返回 CORS 头。

**解决方案：**

```js
// 服务端（Node.js Express 示例）
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://myapp.com');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.header('Access-Control-Allow-Credentials', 'true');
  if (req.method === 'OPTIONS') {
    return res.sendStatus(204); // 预检请求
  }
  next();
});

// 前端 —— 跨域携带 Cookie 时 credentials 必须为 include
const res = await fetch('https://api.other.com/data', {
  credentials: 'include'
});

// 开发环境可用代理（vite.config.js）
export default {
  server: {
    proxy: {
      '/api': {
        target: 'https://api.other.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
};
```

### 2. credentials 默认值变更

Fetch 的 `credentials` 默认值为 `same-origin`，而 XHR 的 `withCredentials` 默认值为 `false`。关键区别：

- Fetch 同源请求**默认发送** Cookie
- Fetch 跨域请求**默认不发送** Cookie
- 如果需要跨域发送 Cookie，必须设置 `credentials: 'include'`，且服务端不能设置 `Access-Control-Allow-Origin: *`

### 3. 4xx/5xx 不 reject

这是 Fetch 最常见的"陷阱"。HTTP 错误状态码不会触发 Promise reject，只有网络级别的错误才会。必须手动检查 `response.ok` 或 `response.status`。

### 4. body 只能读一次

Response 的 body 是一个 ReadableStream，消费后即失效：

```js
const res = await fetch('/api/users');
const text = await res.text();
const json = await res.json(); // TypeError: body stream already read

// 解决：clone()
const res = await fetch('/api/users');
const cloned = res.clone();
const text = await res.text();
const json = await cloned.json(); // OK
```

### 5. GET/HEAD 不能有 body

```js
// 错误：GET 请求不能有 body
fetch('/api/search', {
  method: 'GET',
  body: JSON.stringify({ keyword: 'test' }) // TypeError
});

// 正确：用查询参数
fetch('/api/search?keyword=test');
```

### 6. no-cors 模式的限制

```js
// no-cors 模式下
const res = await fetch('https://other.com/api', {
  mode: 'no-cors',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json' // 无效！只允许简单头
  },
  body: JSON.stringify({ data: 'test' }) // 无效！只允许简单类型
});

console.log(res.type);    // 'opaque'
console.log(res.status);  // 0 —— 无法获取状态码
console.log(res.headers); // 空 —— 无法获取头
// 完全读不到任何内容
```

---

## 面试题

### 1. Fetch 和 XMLHttpRequest 有什么区别？

**答：**

| 维度 | Fetch | XHR |
|------|-------|-----|
| 设计模式 | 基于 Promise | 基于事件回调 |
| 错误处理 | HTTP 错误不 reject，需手动判断 `ok` | `onerror` 仅网络错误，HTTP 错误在 `onload` 中处理 |
| 流式读取 | 支持 `ReadableStream` | 不支持 |
| 请求取消 | `AbortController` | `xhr.abort()` |
| 进度监控 | 不支持原生上传进度 | `onprogress` 支持上传/下载进度 |
| Cookie | `credentials` 控制跨域 Cookie | `withCredentials` 控制 |
| API 风格 | 声明式，一个函数发起请求 | 命令式，`open()` + `send()` 分步 |
| Service Worker | 核心依赖 | 不支持 |

**关键点：** Fetch 更现代、更简洁，但缺少进度监控；XHR 更成熟但 API 过时。两者互补，根据场景选用。

---

### 2. Fetch 如何处理 HTTP 错误（如 404、500）？

**答：**

Fetch 只有网络级别的错误（DNS 失败、网络断开、CORS 拒绝等）才会 reject。HTTP 4xx/5xx 错误仍然会 resolve，需要手动检查：

```js
const response = await fetch('/api/not-found');

// 404 也会 resolve！必须手动判断
if (!response.ok) {
  // response.ok === (status >= 200 && status < 300)
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}

const data = await response.json();
```

建议封装统一的错误处理函数，将 `!response.ok` 的情况统一抛出自定义 `HttpError`，与网络错误的 `TypeError` 区分处理。

---

### 3. 如何实现 Fetch 请求超时？

**答：**

Fetch 没有原生 `timeout`，需要用 `AbortController` + `setTimeout`：

```js
async function fetchWithTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    return await fetch(url, { ...options, signal: controller.signal });
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new Error(`请求超时（${timeout}ms）`);
    }
    throw error;
  } finally {
    clearTimeout(timeoutId);
  }
}
```

Chrome 103+ 支持原生 `AbortSignal.timeout()`：

```js
const res = await fetch(url, { signal: AbortSignal.timeout(5000) });
```

---

### 4. AbortController 的原理是什么？

**答：**

`AbortController` 是浏览器提供的请求取消机制：

1. **创建控制器**：`new AbortController()` 产生一个控制器和关联的 `signal`
2. **传递信号**：将 `controller.signal` 传入 `fetch` 的 `signal` 选项
3. **触发取消**：调用 `controller.abort()` 时，signal 通知 fetch 终止请求
4. **fetch 抛出错误**：被取消的请求 reject 一个 `AbortError`

核心原理是观察者模式 —— `AbortSignal` 继承自 `EventTarget`，fetch 内部监听 signal 的 `abort` 事件，收到后终止底层网络连接。

一个 signal 可以传给多个请求，实现批量取消。`AbortSignal.any()` 可以组合多个信号，任一触发即取消。

---

### 5. 如何实现 Fetch 请求重试？

**答：**

重试需要考虑三点：**重试条件**（5xx 或网络错误重试，4xx 不重试）、**退避策略**（指数退避 + 随机抖动）、**幂等性**（GET/PUT/DELETE 安全重试，POST 需谨慎）：

```js
async function fetchWithRetry(url, options = {}, retries = 3) {
  for (let i = 0; i <= retries; i++) {
    try {
      const res = await fetch(url, options);
      if (res.ok) return res;
      if (res.status < 500 || i === retries) return res;
    } catch (err) {
      if (i === retries) throw err;
    }
    await sleep(Math.pow(2, i) * 1000 + Math.random() * 500);
  }
}
```

指数退避（1s, 2s, 4s...）避免大量请求同时重试压垮服务器。随机抖动（jitter）防止惊群效应。

---

### 6. credentials 的三个值有什么区别？

**答：**

| 值 | 同源 Cookie | 跨域 Cookie |
|---|------------|------------|
| `omit` | 不发送 | 不发送 |
| `same-origin` | 发送 | 不发送 |
| `include` | 发送 | 发送 |

**默认值为 `same-origin`**，跨域请求默认不发送 Cookie，这与 XHR 的 `withCredentials: false` 行为一致。

使用 `include` 时，服务端必须：
1. 设置 `Access-Control-Allow-Credentials: true`
2. `Access-Control-Allow-Origin` 不能为 `*`，必须是具体域名

常见场景：前后端分离部署在不同域名时，登录态 Cookie 需 `credentials: 'include'` 才能携带。

---

### 7. 如何读取流式响应（Streaming Response）？

**答：**

通过 `response.body` 获取 `ReadableStream`，用 `getReader()` 逐块读取：

```js
const response = await fetch('/api/stream');
const reader = response.body.getReader();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  // value 是 Uint8Array
  const text = new TextDecoder().decode(value);
  console.log(text);
}
```

也可以用 `pipeThrough` 管道处理：

```js
const reader = response.body
  .pipeThrough(new TextDecoderStream())
  .getReader();
// value 直接是字符串
```

典型应用场景：SSE 事件流（ChatGPT 流式输出）、大文件下载进度、NDJSON 流式解析。

---

### 8. 如何给 Fetch 实现类似 Axios 的拦截器？

**答：**

核心思路是**包装 fetch**，在请求前后插入拦截器链：

1. 维护 `requestInterceptors` 和 `responseInterceptors` 两个数组
2. 请求前，依次执行请求拦截器，修改 URL/options/headers
3. 调用原始 fetch 发起请求
4. 响应后，依次执行响应拦截器，处理 Response 或错误

请求拦截器常用场景：添加 Token、添加时间戳、请求日志。
响应拦截器常用场景：统一错误处理、自动解析 JSON、Token 过期刷新。

拦截器采用 Promise 链式执行，支持 `onFulfilled` 和 `onRejected` 两个回调，与 Axios 拦截器行为一致。详见上文"请求拦截器模式"的完整实现。

---

相关链接：[[HTTP与缓存策略]] [[Cookie与认证]] [[Service Worker与PWA]]

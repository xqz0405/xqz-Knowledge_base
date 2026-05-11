---
tags:
  - Web前端
  - Cookie
  - Token
  - 认证
  - Session
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Cookie与认证

## What — 是什么

> Cookie 是浏览器存储的小段数据，随请求自动发送；认证方案基于 Cookie 或 Token 验证用户身份。

**核心概念：**

- **Cookie 属性**：`HttpOnly`（JS 不可读）、`Secure`（仅 HTTPS）、`SameSite`（跨站限制）、`Domain`/`Path`（作用域）、`Max-Age`/`Expires`（过期时间）
- **Session 认证**：服务端存储会话，Cookie 只存 Session ID
- **JWT 认证**：Token 自包含用户信息，服务端无状态验证
- **OAuth 2.0**：第三方授权登录（微信/Google/GitHub 登录）

**关键特性：**

- Cookie 大小限制 4KB
- `SameSite=Lax`（默认）阻止大部分 CSRF
- JWT 由 Header.Payload.Signature 三段组成
- Token 存 localStorage 有 XSS 风险，存 HttpOnly Cookie 有 CSRF 风险

## Why — 为什么

**适用场景：**

- 用户登录认证
- 记住我功能
- 第三方登录

**对比方案：**

| 维度 | Session + Cookie | JWT | OAuth 2.0 |
|------|-----------------|-----|-----------|
| 服务端状态 | 有状态 | 无状态 | 有状态 |
| 扩展性 | 需共享 Session | 天然分布式 | 需认证服务 |
| 安全性 | 高（HttpOnly Cookie） | 中（无法主动失效） | 高 |
| 适用场景 | 传统 Web 应用 | API 服务/微服务 | 第三方登录 |

**优缺点：**

- ✅ Session + Cookie：
  - 可主动失效（删除 Session）
  - HttpOnly 防 XSS 窃取
- ❌ Session + Cookie：
  - 分布式需 Session 共享（Redis）
  - 移动端不友好
- ✅ JWT：
  - 无状态，易扩展
  - 跨服务传递方便
- ❌ JWT：
  - 无法主动失效（除非黑名单）
  - Payload 明文可读
  - 续期机制复杂

## How — 怎么用

### 快速上手

**Cookie 操作：**

```javascript
// 设置 Cookie（服务端 Set-Cookie）
res.setHeader('Set-Cookie', [
    'token=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=86400; Path=/',
]);

// 前端读取（非 HttpOnly）
document.cookie; // "name=Alice; token=abc123"

// 前端设置
document.cookie = 'theme=dark; Max-Age=31536000; Path=/';
```

### 代码示例

**JWT 认证流程：**

```
1. 用户提交账密
2. 服务端验证，签发 JWT
   Header: { alg: "HS256", typ: "JWT" }
   Payload: { userId: 1, role: "admin", exp: 1715472000 }
   Signature: HMACSHA256(base64(Header) + "." + base64(Payload), secret)

3. 前端存储 Token（localStorage 或 Cookie）
4. 后续请求携带 Authorization: Bearer <token>
5. 服务端验证签名和过期时间
```

```javascript
// 前端请求拦截器
axios.interceptors.request.use((config) => {
    const token = localStorage.getItem('token');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// Token 过期自动刷新
axios.interceptors.response.use(
    (res) => res,
    async (error) => {
        if (error.response?.status === 401) {
            const { data } = await axios.post('/auth/refresh', {
                refreshToken: localStorage.getItem('refreshToken'),
            });
            localStorage.setItem('token', data.accessToken);
            error.config.headers.Authorization = `Bearer ${data.accessToken}`;
            return axios(error.config); // 重试原请求
        }
        return Promise.reject(error);
    }
);
```

**Session 认证（Express）：**

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis');

app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: 'your-secret',
    resave: false,
    saveUninitialized: false,
    cookie: {
        httpOnly: true,
        secure: true,   // 仅 HTTPS
        sameSite: 'lax',
        maxAge: 86400000, // 24 小时
    },
}));

// 登录
app.post('/login', (req, res) => {
    if (validateUser(req.body)) {
        req.session.userId = user.id;
        res.json({ success: true });
    }
});

// 鉴权中间件
function auth(req, res, next) {
    if (!req.session.userId) return res.sendStatus(401);
    next();
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| CSRF 攻击 | Cookie 自动携带 | `SameSite=Lax/Strict` + CSRF Token |
| XSS 窃取 Token | Token 存 localStorage | 敏感操作用 HttpOnly Cookie |
| JWT 无法失效 | Token 签发后无法撤销 | 短有效期 + Refresh Token + 黑名单 |
| 跨域 Cookie 不发送 | SameSite 限制 | `SameSite=None; Secure`（必须 HTTPS） |

### 最佳实践

- Cookie 敏感值加 `HttpOnly; Secure; SameSite=Lax`
- JWT 短有效期（15min）+ Refresh Token 长有效期
- Refresh Token 存 HttpOnly Cookie，Access Token 存内存
- 生产环境 HTTPS 是必须的

## 面试题

**Q1: Cookie 和 Session 的区别是什么？**
> Cookie 存储在客户端浏览器，随请求自动发送，大小限制 4KB；Session 存储在服务端，通过 Cookie 中的 Session ID 关联用户。Cookie 可被客户端修改，安全性较低；Session 数据在服务端，更安全但需占用服务端存储，分布式环境需共享 Session（如 Redis）。

**Q2: JWT 的原理是什么？有什么优缺点？**
> JWT 由 Header（算法类型）、Payload（用户信息）、Signature（签名）三部分用 `.` 连接组成。服务端用密钥签名，客户端存储并在请求时携带，服务端验证签名即可认证，无需存储会话状态。优点：无状态、易扩展、跨服务传递；缺点：无法主动失效（除非黑名单）、Payload 明文可读、Token 较大、续期机制复杂。

**Q3: HttpOnly 的作用是什么？为什么重要的 Cookie 要设置它？**
> `HttpOnly` 禁止 JavaScript 通过 `document.cookie` 读取该 Cookie，只能由浏览器在 HTTP 请求中自动携带。它防止 XSS 攻击窃取敏感 Cookie（如 Session ID、Token），是最基本的 Cookie 安全措施。设置方式：`Set-Cookie: token=xxx; HttpOnly`。

**Q4: SameSite 的三个值分别是什么？各有什么效果？**
> `Strict`：完全禁止跨站发送 Cookie，即使是顶部导航也不带（最安全但体验差）；`Lax`（默认值）：允许顶级导航的 GET 请求携带 Cookie，POST 和子资源不携带（平衡安全与体验）；`None`：不限制跨站发送，但必须同时设置 `Secure`（仅 HTTPS）。`Lax` 是现代浏览器默认值，可阻止大部分 CSRF 攻击。

**Q5: Token 存 localStorage 和存 HttpOnly Cookie 各有什么安全风险？**
> localStorage：可被 XSS 攻击读取窃取，但天然防 CSRF（不会自动发送）；HttpOnly Cookie：JS 无法读取防 XSS 窃取，但会自动携带导致 CSRF 风险。推荐方案：Access Token 存内存（短有效期），Refresh Token 存 HttpOnly Cookie，兼顾两种安全需求。

---

**相关链接：**
- [[HTTP与缓存策略]]
- [[跨域解决方案]]

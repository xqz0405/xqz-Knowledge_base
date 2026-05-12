---
tags:
  - Node.js
  - Web安全
  - XSS
  - CSRF
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Web 安全防护

## What — 是什么

> Web 安全防护是 Node.js 服务端必须实施的安全措施，涵盖 XSS/CSRF/SQL 注入/点击劫持等 OWASP Top 10 攻击的防御，以及 Helmet/CSP/限流等防护工具的使用。

**核心概念：**

- **XSS（跨站脚本）**：攻击者注入恶意脚本到页面中，窃取 Cookie/Token 或执行操作。分为存储型、反射型、DOM 型
- **CSRF（跨站请求伪造）**：诱导用户在已登录的网站上执行非预期操作
- **SQL 注入**：恶意 SQL 语句通过输入拼接到查询中，绕过认证或破坏数据
- **Helmet**：Express 中间件，设置安全相关的 HTTP 响应头（CSP/X-Frame-Options/HSTS 等）
- **CSP（内容安全策略）**：通过 `Content-Security-Policy` 头限制页面可加载的资源来源
- **速率限制（Rate Limiting）**：限制单位时间内的请求数，防止暴力破解和 DDoS
- **CORS**：跨域资源共享策略，限制哪些域名可以访问 API

**关键特性：**

- XSS 防御核心：输入验证 + 输出转义 + CSP 白名单
- CSRF 防御核心：SameSite Cookie + CSRF Token + Origin 校验
- SQL 注入防御核心：参数化查询（永远不拼接 SQL）
- Helmet 一键设置 15+ 安全响应头
- `express-rate-limit` 提供灵活的限流策略

## Why — 为什么

**适用场景：**

- 所有对外暴露的 Node.js Web 服务
- 处理用户输入的 API 接口
- 需要保护用户数据的认证系统
- 防止恶意爬虫和暴力攻击

**对比安全方案：**

| 维度 | Helmet | 手动设置头 | Nginx 层防护 |
|------|--------|-----------|-------------|
| 配置难度 | 低（一行代码） | 中 | 中 |
| 覆盖范围 | 15+ 安全头 | 按需 | 全局 |
| 灵活性 | 中 | 高 | 高 |
| 性能影响 | 极小 | 极小 | 无（反向代理层） |

**优缺点：**

- ✅ 优点：
  - Helmet 一键加固，减少遗漏
  - 参数化查询彻底防 SQL 注入
  - CSP 大幅降低 XSS 风险
  - 限流有效防止暴力攻击
- ❌ 缺点：
  - CSP 配置复杂，过严会破坏功能
  - 安全防护需要多层配合，单一措施不够
  - 限流可能误伤正常用户

## How — 怎么用

### 快速上手

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

app.use(helmet());
app.use(cors({ origin: 'https://yourdomain.com', credentials: true }));
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));
```

### 代码示例

**XSS/CSRF/SQL注入 防护：**

```javascript
const helmet = require('helmet');
const express = require('express');
const app = express();

// Helmet 安全头
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", "https://cdn.example.com"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'", "https://api.example.com"],
        }
    },
    hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

// XSS 防御：输出转义
function escapeHtml(str) {
    return str.replace(/&/g, '&amp;').replace(/</g, '&lt;')
              .replace(/>/g, '&gt;').replace(/"/g, '&quot;').replace(/'/g, '&#x27;');
}

// SQL 注入防御：参数化查询（永远不拼接）
// ❌ 危险：`SELECT * FROM users WHERE id = ${req.params.id}`
// ✅ 安全：
const user = await pool.query('SELECT * FROM users WHERE id = ?', [req.params.id]);

// CSRF 防御：SameSite Cookie + Token
app.use(session({
    cookie: { sameSite: 'strict', httpOnly: true, secure: true },
    secret: process.env.SESSION_SECRET
}));

// CSRF Token（使用 csurf 或双重提交 Cookie）
const csrf = require('csurf');
app.use(csrf({ cookie: true }));
app.get('/api/csrf-token', (req, res) => {
    res.json({ csrfToken: req.csrfToken() });
});
```

**限流与防暴力破解：**

```javascript
const rateLimit = require('express-rate-limit');

// 全局限流
app.use(rateLimit({
    windowMs: 15 * 60 * 1000, // 15分钟
    max: 100,
    standardHeaders: true,
    legacyHeaders: false,
    message: { error: 'Too many requests' }
}));

// 登录接口严格限流
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    skipSuccessfulRequests: true, // 只计算失败请求
    message: { error: 'Too many login attempts' }
});
app.post('/auth/login', loginLimiter, loginHandler);

// 请求体大小限制（防止大文件攻击）
app.use(express.json({ limit: '10kb' }));
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| CSP 阻止内联脚本 | `unsafe-inline` 被禁止 | 用 nonce 或 hash 替代内联脚本 |
| CORS 预检请求失败 | 缺少 OPTIONS 处理 | `cors` 中间件自动处理 |
| Cookie 跨域不可见 | 缺少 `credentials: true` + `SameSite=None` | 前后端都配置凭证传递 |
| Helmet 破坏前端功能 | CSP 过于严格 | 按需开放特定源 |
| 限流误伤 | IP 共享（NAT/代理） | 用用户 ID 而非 IP 限流 |

### 最佳实践

- 始终使用 Helmet 设置安全响应头
- 永远使用参数化查询，绝不拼接 SQL
- Cookie 设置 `httpOnly` + `secure` + `SameSite=Strict`
- 敏感接口实施严格限流
- 验证和清洗所有用户输入
- 生产环境强制 HTTPS

## 面试题

**Q1: XSS 攻击的三种类型及防御方式？**
> ① 存储型 XSS：恶意脚本存入数据库，其他用户访问时执行。防御：输出转义 + CSP。② 反射型 XSS：恶意脚本在 URL 参数中，服务端未转义直接返回。防御：URL 参数验证 + 输出转义。③ DOM 型 XSS：前端 JS 直接操作 DOM 插入未转义内容。防御：使用 `textContent` 而非 `innerHTML`，输入验证。通用防御：CSP 白名单限制脚本来源、输出转义（`escapeHtml`）、`httpOnly` Cookie 防止窃取。

**Q2: CSRF 攻击的原理和防御措施？**
> 原理：攻击者构造恶意页面，诱导已登录用户访问，利用浏览器的 Cookie 自动携带机制，向目标网站发送伪造请求。防御：① SameSite Cookie（`SameSite=Strict`/`Lax`）——阻止跨站请求携带 Cookie；② CSRF Token——服务端生成随机 Token，表单/请求中携带，服务端验证；③ Origin/Referer 校验——检查请求来源是否合法；④ 双重 Cookie 验证——Cookie 和请求体中同时携带 Token。

**Q3: SQL 注入如何防范？为什么参数化查询有效？**
> 防范：① 参数化查询（最有效）——`pool.query('SELECT * FROM users WHERE id = ?', [id])`，参数与 SQL 语句分离传输，数据库将参数视为纯数据而非 SQL 语法；② ORM（Sequelize/Prisma）——默认参数化查询；③ 输入验证——白名单校验输入格式；④ 最小权限——数据库用户只授予必要权限。参数化查询有效的原因：数据库先编译 SQL 模板，再绑定参数值，参数值无法改变 SQL 结构，即使包含 `' OR '1'='1` 也只作为字符串比较。

**Q4: Helmet 设置了哪些安全头？各自作用？**
> 15+ 安全头：① `Content-Security-Policy`——限制可加载资源来源（防 XSS）；② `X-Frame-Options`——阻止页面被 iframe 嵌入（防点击劫持）；③ `Strict-Transport-Security`——强制 HTTPS（防中间人）；④ `X-Content-Type-Options: nosniff`——阻止 MIME 嗅探；⑤ `X-XSS-Protection`——浏览器 XSS 过滤器（已弃用，CSP 替代）；⑥ `Referrer-Policy`——控制 Referer 头泄露；⑦ `Permissions-Policy`——限制浏览器 API（摄像头/麦克风/定位）。

**Q5: CSP 如何配置？常见的坑？**
> CSP 通过 `Content-Security-Policy` 响应头配置，用分号分隔指令：`default-src 'self'; script-src 'self' cdn.example.com; style-src 'self' 'unsafe-inline'`。常见坑：① `'unsafe-inline'` 大幅削弱 XSS 防护，应用 nonce/hash 替代；② `'unsafe-eval'` 允许 `eval()`，应避免；③ 内联样式需要 `'unsafe-inline'` 或 nonce，可改用外部 CSS；④ 动态加载的脚本需要 `script-src` 白名单；⑤ 报告模式 `Content-Security-Policy-Report-Only` 先观察再执行。

**Q6: 速率限制的策略有哪些？如何避免误伤？**
> 策略：① 固定窗口——简单但边界突发问题（1分钟100请求，59秒和61秒各100请求实际2秒200请求）；② 滑动窗口——精确但实现复杂；③ 令牌桶——允许突发流量，适合 API；④ 漏桶——恒定速率，适合消息队列。避免误伤：① 按 API Key/用户 ID 限流而非 IP（NAT 下多用户共享 IP）；② 区分成功和失败请求（`skipSuccessfulRequests`）；③ 不同接口不同阈值（登录严格，查询宽松）；④ 返回 `Retry-After` 头告知客户端何时重试。

---

**相关链接：**
- [[认证与授权]]
- [[加密与数据安全]]
- [[Express]]
- OWASP Top 10：https://owasp.org/www-project-top-ten/

---
tags:
  - Web前端
  - 安全
  - XSS
  - CSRF
  - CSP
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Web安全XSS与CSRF

## What — 是什么

> XSS（跨站脚本攻击）是注入恶意脚本到网页中执行，CSRF（跨站请求伪造）是利用已认证身份发起恶意请求。两者是最常见的前端安全威胁。

**XSS 类型：**

- **存储型 XSS**：恶意脚本存入数据库，所有用户访问时执行（危害最大）
- **反射型 XSS**：恶意脚本在 URL 参数中，服务端原样返回执行
- **DOM 型 XSS**：前端 JS 直接将用户输入写入 DOM，无需服务端参与

**CSRF 核心：**

- 利用浏览器自动携带 cookie 的机制
- 用户已登录 A 站，访问恶意 B 站，B 站自动发起 A 站请求
- 浏览器会自动带上 A 站的 cookie

**CSP（内容安全策略）：**

- HTTP 响应头 `Content-Security-Policy`，限制资源加载来源
- 禁止内联脚本和 eval，从根本上阻止 XSS

**关键特性：**

- XSS 攻击的是用户浏览器，CSRF 攻击的是用户身份
- 防御需前后端配合，单靠前端无法完全防御

## Why — 为什么

**适用场景：**

- 所有接受用户输入的 Web 应用
- 有登录态的网站（CSRF 风险）
- 显示用户生成内容的页面（XSS 风险）

**攻击影响对比：**

| 维度 | XSS | CSRF |
|------|-----|------|
| 攻击目标 | 用户浏览器 | 用户身份 |
| 能否读取数据 | 能（盗取 cookie/token） | 不能（受同源策略限制） |
| 能否修改数据 | 能（操作 DOM/发请求） | 能（伪造请求） |
| 是否需要用户登录 | 不一定 | 必须 |
| 防御核心 | 输出转义 + CSP | Token 校验 |

## How — 怎么用

### XSS 防御

**1. 输出转义（最基础）：**

```javascript
// HTML 转义
function escapeHTML(str) {
    return str
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;');
}

// React 默认转义，以下写法安全
const userInput = '<script>alert("xss")</script>';
return <div>{userInput}</div>; // 自动转义，安全

// ❌ 危险：dangerouslySetInnerHTML
return <div dangerouslySetInnerHTML={{ __html: userInput }} />; // XSS！

// ✅ 如需渲染 HTML，先用 DOMPurify 清洗
import DOMPurify from 'dompurify';
return <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />;
```

**2. CSP 策略：**

```nginx
# Nginx 配置 CSP
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' https://cdn.example.com;
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    connect-src 'self' https://api.example.com;
    font-src 'self';
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
" always;
```

**3. HttpOnly Cookie：**

```javascript
// 后端设置 cookie 时
res.cookie('token', jwt, {
    httpOnly: true,  // JS 无法读取，防 XSS 窃取
    secure: true,    // 仅 HTTPS
    sameSite: 'Lax', // 防 CSRF
    maxAge: 86400000,
});
```

**4. Vue 中的安全处理：**

```vue
<template>
    <!-- ✅ Vue 模板自动转义 -->
    <div>{{ userInput }}</div>

    <!-- ❌ 危险：v-html -->
    <div v-html="userInput"></div>

    <!-- ✅ 需要 HTML 渲染时清洗 -->
    <div v-html="sanitizedHTML"></div>
</template>

<script setup>
import DOMPurify from 'dompurify';
import { computed } from 'vue';

const props = defineProps<{ content: string }>();
const sanitizedHTML = computed(() => DOMPurify.sanitize(props.content));
</script>
```

### CSRF 防御

**1. CSRF Token（最常用）：**

```javascript
// 后端：生成 Token，注入到页面
app.use((req, res, next) => {
    res.locals.csrfToken = crypto.randomUUID();
    res.cookie('csrfToken', res.locals.csrfToken, { httpOnly: false });
    next();
});

// 前端：请求时携带 Token
fetch('/api/transfer', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]').content,
    },
    body: JSON.stringify({ to, amount }),
});
```

**2. SameSite Cookie：**

```javascript
// 后端设置（现代浏览器默认 Lax）
res.cookie('token', jwt, { sameSite: 'Lax' });
// Strict：最严格，跨站请求完全不带 cookie
// Lax：GET 导航带 cookie，POST 不带（推荐默认）
// None：不限制（需配合 Secure）
```

**3. 使用 Bearer Token 替代 Cookie：**

```javascript
// 前端存储 token，请求时手动携带
const token = localStorage.getItem('token');

fetch('/api/data', {
    headers: { Authorization: `Bearer ${token}` },
});
// 攻击者无法读取 localStorage（同源策略），天然防 CSRF
```

**4. 框架内置防护：**

```typescript
// Next.js CSRF 示例
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
    if (['POST', 'PUT', 'DELETE'].includes(req.method)) {
        const origin = req.headers.get('origin');
        const host = req.headers.get('host');
        if (origin && new URL(origin).host !== host) {
            return new NextResponse('Forbidden', { status: 403 });
        }
    }
    return NextResponse.next();
}
```

### 其他安全要点

**点击劫持（Clickjacking）：**

```nginx
# X-Frame-Options 禁止被嵌入 iframe
add_header X-Frame-Options "DENY" always;

# 或 CSP 方式
add_header Content-Security-Policy "frame-ancestors 'none'" always;
```

**开放重定向：**

```javascript
// ❌ 危险：未校验重定向地址
app.get('/redirect', (req, res) => {
    res.redirect(req.query.url); // 可重定向到恶意网站
});

// ✅ 安全：白名单校验
function safeRedirect(url) {
    const allowed = ['/dashboard', '/profile', '/settings'];
    if (allowed.includes(url)) return url;
    return '/';
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 富文本 XSS | 未清洗 HTML | 用 DOMPurify，不用 dangerouslySetInnerHTML/v-html |
| JSON API 被 CSRF | 依赖 cookie 认证 | 用 Bearer Token 或 CSRF Token |
| CDN 脚本被篡改 | CDN 被攻破 | SRI（Subresource Integrity）+ CSP |
| SameSite 兼容性 | 旧浏览器不支持 | 同时使用 CSRF Token 兜底 |
| 第三方嵌入风险 | iframe 可做点击劫持 | X-Frame-Options 或 CSP frame-ancestors |

### 最佳实践

- 所有用户输入输出时转义，富文本用 DOMPurify 清洗
- Cookie 设置 `httpOnly` + `secure` + `sameSite: Lax`
- 部署 CSP 策略，禁止内联脚本
- API 认证用 Bearer Token 而非仅依赖 Cookie
- 敏感操作二次确认（支付、删除）
- 定期用 OWASP ZAP 做安全扫描

## 面试题

**Q1: XSS 有哪三种类型？各自的特点和危害程度如何？**
> 存储型 XSS：恶意脚本存入数据库，所有访问该页面的用户都会执行，危害最大；反射型 XSS：恶意脚本在 URL 参数中，服务端原样返回到页面执行，需诱导用户点击链接；DOM 型 XSS：前端 JS 直接将用户输入写入 DOM（如 `innerHTML`），无需服务端参与，纯前端漏洞。

**Q2: CSRF 攻击的原理和常见防御手段有哪些？**
> CSRF 利用浏览器自动携带 Cookie 的机制，用户已登录 A 站时访问恶意 B 站，B 站自动向 A 站发起请求并带上 Cookie。防御手段：CSRF Token（请求携带服务端生成的随机 Token）、SameSite Cookie（设为 Lax/Strict 限制跨站发送）、使用 Bearer Token 替代 Cookie 认证（不自动发送）、验证 Origin/Referer 头。

**Q3: CSP（内容安全策略）的作用是什么？如何配置？**
> CSP 通过 HTTP 响应头 `Content-Security-Policy` 限制页面可加载的资源来源，禁止内联脚本和 `eval`，从根源上阻止 XSS 注入。例如 `script-src 'self'` 只允许加载同源脚本，`default-src 'self'` 限制所有资源默认同源。配置方式：服务端设置响应头或 HTML `<meta>` 标签。

**Q4: CSRF 和 XSS 的本质区别是什么？**
> XSS 攻击的是用户浏览器，通过注入脚本执行任意代码，能读取数据（如 Cookie/Token）和操作 DOM；CSRF 攻击的是用户身份，利用已认证的 Cookie 伪造请求，不能读取响应数据（受同源策略限制），只能执行操作。XSS 防御核心是输出转义 + CSP，CSRF 防御核心是 Token 校验 + SameSite。

---

**相关链接：**
- [[Cookie与认证]]
- [[跨域解决方案]]
- [[HTTP与缓存策略]]

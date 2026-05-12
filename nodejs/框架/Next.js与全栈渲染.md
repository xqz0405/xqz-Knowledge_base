---
tags:
  - Node.js
  - Next.js
  - SSR
  - 全栈框架
date: 2026-05-12
status: 已完成
difficulty: 进阶
---

# Next.js 与全栈渲染

## What — 是什么

> Next.js 是 React 全栈框架，提供 SSR/SSG/ISR/RSC 等多种渲染策略，App Router 和 API Routes 让一个项目同时包含前端和后端逻辑。

**核心概念：**

- **SSR（Server-Side Rendering）**：每次请求在服务端渲染 React 组件为 HTML，SEO 友好，数据实时
- **SSG（Static Site Generation）**：构建时生成静态 HTML，访问最快，适合内容不常变化的页面
- **ISR（Incremental Static Regeneration）**：SSG + 按需重新生成，设置 `revalidate` 秒数，后台重新生成不影响访问
- **RSC（React Server Components）**：服务端组件，只在服务端执行，不发送 JS 到客户端，减少包体积
- **App Router**：基于文件系统的路由，`app/` 目录下 `page.tsx`/`layout.tsx`/`loading.tsx` 约定式路由
- **API Routes**：`app/api/` 目录下的 `route.ts` 文件定义后端 API 端点

## Why — 为什么

**适用场景：**

- SEO 要求高的网站（博客/电商/新闻）
- 首屏加载速度要求高
- 前后端同仓库开发
- 需要 SSR + CSR 混合渲染

**对比渲染策略：**

| 维度 | SSR | SSG | ISR | CSR |
|------|-----|-----|-----|-----|
| 数据实时性 | 实时 | 构建时 | 可配置 | 实时 |
| 性能 | 中 | 最快 | 快 | 中 |
| 服务器压力 | 高 | 低 | 中 | 低 |
| SEO | 好 | 好 | 好 | 差 |

## How — 怎么用

### 代码示例

```tsx
// app/page.tsx — SSR
export default async function HomePage() {
    const posts = await fetch('https://api.example.com/posts').then(r => r.json());
    return <PostList posts={posts} />;
}

// app/blog/[slug]/page.tsx — SSG + ISR
export const revalidate = 3600; // 1小时后重新生成

export default async function BlogPost({ params }) {
    const post = await fetchPost(params.slug);
    return <Article post={post} />;
}

// app/api/users/route.ts — API Route
export async function GET(request) {
    const users = await db.users.findMany();
    return Response.json(users);
}

export async function POST(request) {
    const body = await request.json();
    const user = await db.users.create({ data: body });
    return Response.json(user, { status: 201 });
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| SSR 首次加载慢 | 服务端渲染耗时 | 流式渲染 Suspense + 缓存 |
| RSC 状态丢失 | 服务端组件无客户端状态 | 客户端组件 `'use client'` 包裹交互部分 |
| ISR 不更新 | CDN 缓存未失效 | 检查 `revalidate` 配置和 `on-demand revalidation` |
| API Routes 限流 | 无内置限流 | 中间件 `middleware.ts` 实现限流 |

### 最佳实践

- 静态页面用 SSG + ISR，动态页面用 SSR
- 交互组件用 `'use client'` 标记，其余保持 Server Component
- API Routes 实现后端逻辑，BFF 聚合外部 API
- 使用 `next/image` 和 `next/link` 优化性能

## 面试题

**Q1: SSR、SSG、ISR 的区别和适用场景？**
> SSR：每次请求服务端渲染，数据实时但服务器压力大，适合个性化页面（用户仪表盘）。SSG：构建时生成静态 HTML，访问最快但数据可能过时，适合内容稳定的页面（文档/博客）。ISR：SSG + 定期重新生成，设置 `revalidate` 秒数，兼顾性能和数据新鲜度，适合内容偶尔更新的页面（产品列表/新闻）。

**Q2: React Server Components 与传统 SSR 的区别？**
> RSC 只在服务端执行，不会发送 JS 到客户端——减少了客户端包体积。传统 SSR 在服务端渲染 HTML 后，客户端还需要加载完整 JS 水合（hydration）恢复交互。RSC 可以直接访问数据库/文件系统等服务端资源，无需 API 中间层。RSC 与 Client Components 可以混合使用——静态内容用 RSC，交互部分用 `'use client'`。

---

**相关链接：**
- [[Express]]
- [[Fastify]]
- [[BFF与API网关]]
- Next.js 文档：https://nextjs.org/docs

---
tags:
  - Web前端
  - Next.js
  - SSR
  - SSG
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Next.js与SSR

## What — 是什么

> Next.js 是 React 全栈框架，支持 SSR（服务端渲染）、SSG（静态生成）和 ISR（增量静态再生），解决 SPA 的 SEO 和首屏性能问题。

**核心概念：**

- **SSR**：每次请求时在服务端渲染 HTML，数据实时
- **SSG**：构建时生成静态 HTML，适合内容不变的页面
- **ISR**：后台定时重新生成静态页，兼顾性能和时效
- **App Router**：Next.js 13+ 基于 React Server Components 的新路由系统
- **RSC**：React Server Components，服务端组件减少客户端 JS 体积

**核心架构：**

- 设计理念：混合渲染，按页面选择最优策略
- 核心模块：App Router、Server Components、Streaming SSR、Image/Font 优化
- 数据流：请求 → 路由匹配 → 渲染策略（SSR/SSG/CSR） → HTML 响应

**关键特性：**

- App Router 下组件默认是 Server Component
- `"use client"` 声明客户端组件
- `loading.tsx` 自动 Suspense 流式加载
- API Routes 处理后端逻辑

## Why — 为什么

**适用场景：**

- 需要 SEO 的内容站点（博客、电商、新闻）
- 首屏性能要求高的 C 端应用
- 全栈 React 应用

**对比同类框架：**

| 维度 | Next.js | Nuxt.js | Remix | SPA（Vite React） |
|------|---------|---------|-------|-------------------|
| SEO | 极好 | 极好 | 极好 | 差 |
| 首屏速度 | 快（SSR/SSG） | 快 | 快 | 慢（CSR） |
| 学习曲线 | 中 | 低 | 中 | 低 |
| 灵活性 | 高 | 中 | 中 | 极高 |

**优缺点：**

- ✅ 优点：
  - SSR/SSG/CSR 灵活选择
  - SEO 和首屏性能兼得
  - 内置图片/字体优化
  - 全栈能力（API Routes）
- ❌ 缺点：
  - 框架约束多，不够灵活
  - Server/Client Component 边界易混淆
  - 部署需 Node.js 环境（SSR 模式）

## How — 怎么用

### 快速上手

```bash
npx create-next-app@latest my-app --app --typescript
```

```tsx
// app/page.tsx — 默认 Server Component
async function Home() {
    const posts = await fetch('https://api.example.com/posts').then(r => r.json());
    return (
        <main>
            {posts.map(post => <article key={post.id}>{post.title}</article>)}
        </main>
    );
}
```

### 代码示例

**Server vs Client Component：**

```tsx
// app/page.tsx — Server Component（默认）
// ✅ 可直接 async/await、访问数据库、不发送 JS 到客户端
async function Page() {
    const data = await db.query('SELECT * FROM posts');
    return <PostList posts={data} />;
}

// components/Counter.tsx — Client Component
'use client';
import { useState } from 'react';

export function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

**SSG + ISR：**

```tsx
// 构建时生成，每 60 秒重新验证
async function BlogPost({ params }: { params: { slug: string } }) {
    const post = await getPost(params.slug);
    return <article>{post.content}</article>;
}

export const revalidate = 60; // ISR 间隔

export async function generateStaticParams() {
    const posts = await getAllPosts();
    return posts.map(post => ({ slug: post.slug }));
}
```

**Streaming SSR（loading.tsx）：**

```tsx
// app/dashboard/loading.tsx — 自动 Suspense
export default function Loading() {
    return <div>加载中...</div>;
}

// app/dashboard/page.tsx
async function Dashboard() {
    const stats = await getStats(); // 流式渲染，不阻塞整体
    return <StatsGrid data={stats} />;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| "use client" 组件中用了服务端 API | Client Component 不运行在服务端 | 数据在 Server Component 获取，通过 props 传递 |
| useState 在 Server Component 报错 | Server Component 无状态 | 拆成 Client Component 或提升状态 |
| 构建慢 | SSG 页面过多 | 用 ISR 延迟生成，或动态路由 |
| hydration mismatch | SSR 和 CSR 渲染结果不一致 | 避免在渲染中使用 `Date.now()`/`Math.random()` |

### 最佳实践

- 默认用 Server Component，交互部分才用 `"use client"`
- 数据获取在 Server Component 中直接 async/await
- 用 `loading.tsx` + Streaming 提升感知性能
- 静态内容用 SSG，动态内容用 SSR/ISR

## 面试题

**Q1: SSR、SSG、CSR 三种渲染模式有什么区别？**
> SSR 每次请求时服务端渲染 HTML，数据实时但服务器压力大；SSG 构建时生成静态 HTML，速度最快但内容固定；CSR 客户端加载 JS 后渲染，SEO 差、首屏慢但交互灵活。选择取决于 SEO 需求、数据时效性和性能要求。

**Q2: getServerSideProps 和 getStaticProps 的区别是什么？**
> `getServerSideProps` 每次请求时在服务端执行，获取实时数据，适用于动态页面；`getStaticProps` 构建时执行一次，生成静态 HTML，配合 `revalidate` 可实现 ISR 增量更新，适合内容不频繁变化的页面。

**Q3: 什么是 Hydration？Hydration Mismatch 是怎么产生的？**
> Hydration 是服务端渲染的 HTML 在客户端被 React"激活"，绑定事件和状态的过程。Mismatch 产生于 SSR 和 CSR 渲染结果不一致，常见原因：渲染中使用了 `Date.now()`、`Math.random()`、浏览器 API（window）或依赖了客户端状态。

**Q4: Server Component 和 Client Component 有什么区别？**
> Server Component 在服务端执行，可直接访问数据库/文件系统，不发送 JS 到客户端；Client Component 通过 `"use client"` 声明，在浏览器执行，支持 useState/onClick 等交互。默认是 Server Component，只有需要交互的组件才标记为 Client Component。

---

**相关链接：**
- [[React核心]]
- [[React Hooks详解]]
- [[HTTP与缓存策略]]

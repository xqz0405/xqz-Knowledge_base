---
tags:
  - Web前端
  - Next.js
  - SSR
  - SSG
  - ISR
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Next.js与SSR

## What — 是什么

> Next.js 是 Vercel 开发的 React 全栈框架，支持 SSR（服务端渲染）、SSG（静态生成）、ISR（增量静态再生）、CSR（客户端渲染）和 Streaming SSR（流式渲染），解决 SPA 的 SEO 和首屏性能问题，并提供基于 React Server Components 的 App Router 路由系统。

### 渲染模式详解

| 模式 | 渲染时机 | 数据实时性 | 服务器负载 | 首屏速度 | SEO | 适用场景 |
|------|---------|-----------|-----------|---------|-----|---------|
| SSR | 每次请求时 | 实时 | 高 | 较快 | 极好 | 个性化页面、实时数据看板 |
| SSG | 构建时 | 固定 | 无 | 极快 | 极好 | 博客、文档、营销页 |
| ISR | 构建时 + 定时刷新 | 准实时 | 低 | 极快 | 极好 | 电商商品页、新闻列表 |
| CSR | 客户端运行时 | 实时 | 无 | 慢 | 差 | 后台管理、仪表盘 |
| Streaming SSR | 请求时流式返回 | 实时 | 中 | 快（渐进） | 好 | 复杂页面、慢数据源 |

**渲染流程对比：**

```
SSR:  请求 → 服务端渲染完整 HTML → 返回 → Hydration → 可交互
SSG:  构建 → 生成静态 HTML → CDN 缓存 → 请求直接返回 → Hydration
ISR:  构建 → 静态 HTML + 定时触发 → 过期后后台重新生成 → 新请求返回新页面
CSR:  请求 → 返回空壳 HTML → 下载 JS → 客户端渲染 → 可交互
Streaming SSR: 请求 → 流式返回 HTML 片段 → 逐步 Hydration → 可交互
```

### App Router vs Pages Router

Next.js 13+ 引入了全新的 App Router，与传统的 Pages Router 并存：

| 维度 | App Router | Pages Router |
|------|-----------|-------------|
| 目录 | `app/` | `pages/` |
| 路由定义 | 文件系统（page.tsx） | 文件系统（自动映射） |
| 布局 | 嵌套 Layout（不重新渲染） | 单一 `_app` + `_document` |
| 数据获取 | async/await（Server Component） | `getServerSideProps`/`getStaticProps` |
| 组件模型 | Server Component 优先 | 全部为 Client Component |
| 加载状态 | `loading.tsx` + Suspense | 手动处理 |
| 错误处理 | `error.tsx` | `_error.tsx` |
| API 路由 | `route.ts`（Route Handlers） | `pages/api/` |
| 流式渲染 | 内置支持 | 不支持 |
| 状态 | 推荐方案 | 稳定方案 |

> App Router 是 Next.js 的未来方向，Pages Router 仍然支持但不推荐新项目使用。

### App Router 核心文件约定

App Router 通过文件名约定定义路由行为：

| 文件名 | 用途 | 说明 |
|--------|------|------|
| `page.tsx` | 页面组件 | 定义路由 UI，是路由的唯一入口 |
| `layout.tsx` | 布局组件 | 共享布局，导航时不重新渲染 |
| `loading.tsx` | 加载状态 | 自动包裹 Suspense，流式加载时显示 |
| `error.tsx` | 错误边界 | 捕获子组件错误，必须是 Client Component |
| `not-found.tsx` | 404 页面 | 路由未匹配时显示 |
| `template.tsx` | 模板组件 | 类似 Layout 但导航时重新挂载 |
| `default.tsx` | 并行路由默认 | 并行路由未匹配时的回退 UI |
| `route.ts` | API 路由 | 处理 GET/POST 等 HTTP 请求 |
| `middleware.ts` | 中间件 | 请求前拦截，鉴权/重定向 |

### Server Components vs Client Components

| 维度 | Server Components | Client Components |
|------|-------------------|-------------------|
| 声明方式 | 默认，无需声明 | 文件顶部 `"use client"` |
| 执行位置 | 服务端 | 客户端（浏览器） |
| JS 发送到客户端 | 否（零 JS 体积） | 是 |
| 状态管理 | 不支持 useState/useReducer | 支持 |
| 副作用 | 不支持 useEffect | 支持 |
| 浏览器 API | 不可用（window/document） | 可用 |
| 数据获取 | 直接 async/await | 通过 useEffect 或 SWR/React Query |
| 后端资源 | 可直接访问数据库/文件系统 | 不可 |
| 事件处理 | 不支持 onClick/onChange | 支持 |
| 自定义 Hooks | 仅服务端 Hooks | 全部 |

**组件选择决策树：**

```
需要交互（onClick/onChange）？ → Client Component
需要状态（useState/useReducer）？ → Client Component
需要浏览器 API（window/localStorage）？ → Client Component
需要 useEffect？ → Client Component
需要直接访问数据库/文件系统？ → Server Component
其余情况 → Server Component（默认）
```

### 路由系统详解

**1. 动态路由**

```
app/blog/[slug]/page.tsx    →  /blog/hello-world
app/shop/[...slug]/page.tsx →  /shop/a/b/c （Catch-all）
app/shop/[[slug]]/page.tsx  →  /shop 或 /shop/a/b/c （Optional Catch-all）
```

**2. 路由组（Route Groups）**

用 `(groupName)` 包裹目录，用于组织代码但不影响 URL：

```
app/(marketing)/about/page.tsx   →  /about
app/(marketing)/contact/page.tsx →  /contact
app/(shop)/products/page.tsx     →  /products
```

> 路由组可以让不同 URL 共享同一 Layout，也可以让同 URL 层级的页面拥有不同 Layout。

**3. 并行路由（Parallel Routes）**

用 `@folder` 定义同时渲染的多个插槽：

```
app/dashboard/@team/page.tsx
app/dashboard/@analytics/page.tsx
app/dashboard/layout.tsx  → 接收 team 和 analytics 两个插槽
```

**4. 拦截路由（Intercepting Routes）**

用 `(.)`/`(..)` 约定拦截路由跳转，实现模态框等效果：

```
app/feed/(.)photo/[id]/page.tsx  → 在 feed 页面内拦截 /photo/1 为模态框
app/photo/[id]/page.tsx          → 直接访问 /photo/1 时显示完整页面
```

### 数据获取与缓存

**fetch 缓存策略：**

| 选项 | 说明 | 等价 Pages Router |
|------|------|-------------------|
| `cache: 'force-cache'` | 缓存请求，构建时获取一次 | `getStaticProps` |
| `cache: 'no-store'` | 不缓存，每次请求重新获取 | `getServerSideProps` |
| `next: { revalidate: 60 }` | 缓存 60 秒后重新验证 | ISR `revalidate` |

**revalidation 方式：**

- **时间基准（Time-based）**：`export const revalidate = 60`，页面级设置
- **按需基准（On-demand）**：通过 `revalidatePath` 或 `revalidateTag` 手动触发

### Server Actions

Server Actions 是 Next.js 13+ 引入的服务端函数，允许客户端直接调用服务端代码，无需手动编写 API：

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;
    await db.post.create({ data: { title, content } });
    revalidatePath('/posts'); // 刷新缓存
}
```

**核心特性：**

- `"use server"` 标记服务端函数
- 可直接在 `<form action={}>` 中使用
- 自动处理 CSRF 防护
- 配合 `useActionState` / `useOptimistic` 实现乐观更新
- 调用后可触发 revalidation

### 内置优化

| 优化项 | 组件/API | 说明 |
|--------|---------|------|
| 图片优化 | `<Image>` | 自动 WebP/AVIF、懒加载、响应式尺寸 |
| 字体优化 | `next/font` | 自动子集化、预加载、零布局偏移 |
| 脚本优化 | `<Script>` | 控制加载策略（lazy/onload/afterInteractive） |
| 链接预取 | `<Link>` | 视口内链接自动预取 |
| 元数据 | `metadata` / `generateMetadata` | 自动生成 SEO 标签 |
| 静态资源 | `next/static` | 自动 CDN 缓存、文件名哈希 |

## Why — 为什么

### SSR/SSG/ISR/CSR 详细对比

| 维度 | SSR | SSG | ISR | CSR |
|------|-----|-----|-----|-----|
| **SEO** | 极好（完整 HTML） | 极好（完整 HTML） | 极好（完整 HTML） | 差（空壳 HTML） |
| **FCP（首次内容绘制）** | 较快（0.8-1.5s） | 极快（0.3-0.5s） | 极快（0.3-0.5s） | 慢（1.5-3s） |
| **TTI（可交互时间）** | 较快（1-2s） | 较快（0.5-1.5s） | 较快（0.5-1.5s） | 慢（2-4s） |
| **服务器负载** | 高（每次请求渲染） | 无（CDN 直接返回） | 低（仅过期时渲染） | 无（静态托管） |
| **数据实时性** | 实时 | 固定 | 准实时（revalidate 间隔） | 实时 |
| **CDN 友好** | 否 | 是 | 是 | 是 |
| **TTFB** | 较高（服务端计算） | 极低 | 极低 | 极低 |
| **构建时间** | 短（无预渲染） | 长（所有页面） | 中（仅首次） | 短 |
| **适用页面比例** | 10-20% | 40-60% | 20-30% | 10-20% |

### 什么时候用 Next.js vs 纯 SPA

**应该用 Next.js 的场景：**

- 内容需要被搜索引擎索引（博客、电商、新闻、文档站）
- 首屏加载速度是核心指标（C 端产品、落地页）
- 需要全栈能力（API Routes + 数据库）
- 页面间共享布局，导航体验要求高
- 需要国际化（i18n）且要 SEO 友好

**应该用纯 SPA（Vite React）的场景：**

- 内部后台管理系统（无 SEO 需求）
- 高交互应用（在线编辑器、实时协作）
- 已有成熟 SPA 架构，迁移成本高
- 团队对 SSR/RSC 概念不熟悉
- 部署环境不支持 Node.js（仅静态托管）

### App Router vs Pages Router 迁移考量

| 考量 | 说明 |
|------|------|
| **渐进迁移** | App Router 和 Pages Router 可以共存，逐页迁移 |
| **数据获取** | `getServerSideProps` → Server Component async/await |
| **布局系统** | `_app` + `_document` → 嵌套 `layout.tsx` |
| **API 路由** | `pages/api/` → `app/api/route.ts`（Route Handlers） |
| **路由参数** | `context.params` → 函数参数 `params` |
| **404 处理** | `_error.tsx` → `not-found.tsx` + `error.tsx` |
| **中间件** | 兼容，`middleware.ts` 无需修改 |
| **学习成本** | RSC 概念 + 文件约定需要时间适应 |

> 建议：新项目直接用 App Router；老项目可渐进迁移，从叶子路由开始。

### 对比同类框架

| 维度 | Next.js | Nuxt.js | Remix | Astro |
|------|---------|---------|-------|-------|
| 语言/框架 | React | Vue | React | 框架无关 |
| SEO | 极好 | 极好 | 极好 | 极好 |
| 首屏速度 | 快 | 快 | 快 | 极快（零 JS 默认） |
| SSR | 支持 | 支持 | 支持 | 支持 |
| SSG | 支持 | 支持 | 支持 | 核心能力 |
| ISR | 支持 | 支持 | 不支持 | 不支持 |
| 学习曲线 | 中 | 低 | 中 | 低 |
| 灵活性 | 高 | 中 | 中 | 中 |
| 全栈能力 | 强 | 强 | 强 | 中 |
| 社区生态 | 最丰富 | 丰富 | 中等 | 增长快 |

### 优缺点

- 优点：
  - SSR/SSG/ISR/CSR 灵活选择，按页面决定最优策略
  - SEO 和首屏性能兼得
  - 内置图片/字体/脚本优化，零配置性能提升
  - Server Components 减少客户端 JS 体积
  - 全栈能力（Route Handlers + Server Actions + Middleware）
  - App Router 嵌套布局，导航不重新渲染
  - Streaming SSR 渐进加载，慢数据不阻塞整体
  - Vercel 部署一键托管，Edge Runtime 全球分布
- 缺点：
  - 框架约束多，定制灵活性不如纯 SPA
  - Server/Client Component 边界易混淆，初学者常踩坑
  - 部署需 Node.js 环境（SSR 模式），静态托管仅限 SSG
  - App Router 仍在快速迭代，部分 API 可能变更
  - 构建时间随页面数增长，ISR 缓解但不完全解决
  - Hydration 在复杂页面仍有性能开销

## How — 怎么用

### 快速上手

```bash
# 创建项目
npx create-next-app@latest my-app --app --typescript --tailwind --eslint

# 启动开发服务器
cd my-app && npm run dev

# 构建生产版本
npm run build && npm start
```

### 1. App Router 基础项目结构

```
my-app/
├── app/
│   ├── layout.tsx          # 根布局（必须包含 html + body）
│   ├── page.tsx            # 首页 /
│   ├── loading.tsx         # 全局加载状态
│   ├── error.tsx           # 全局错误边界
│   ├── not-found.tsx       # 404 页面
│   ├── globals.css         # 全局样式
│   ├── blog/
│   │   ├── layout.tsx      # 博客布局
│   │   ├── page.tsx        # /blog
│   │   └── [slug]/
│   │       └── page.tsx    # /blog/hello-world
│   ├── dashboard/
│   │   ├── layout.tsx
│   │   ├── loading.tsx     # dashboard 加载骨架
│   │   └── page.tsx        # /dashboard
│   └── api/
│       └── posts/
│           └── route.ts    # API Route: /api/posts
├── components/
│   ├── ui/                 # 通用 UI 组件
│   └── features/           # 业务组件
├── lib/
│   ├── db.ts               # 数据库连接
│   └── utils.ts            # 工具函数
├── middleware.ts            # 中间件
├── next.config.js          # Next.js 配置
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

### 2. Server Component 数据获取

```tsx
// app/blog/page.tsx — Server Component 默认，可直接 async
import Link from 'next/link';

interface Post {
    id: number;
    title: string;
    excerpt: string;
    createdAt: string;
}

// 默认 cache: 'force-cache'，构建时获取（SSG）
async function getPosts(): Promise<Post[]> {
    const res = await fetch('https://api.example.com/posts', {
        cache: 'force-cache', // SSG
    });
    return res.json();
}

export default async function BlogPage() {
    const posts = await getPosts();

    return (
        <div className="max-w-4xl mx-auto py-8">
            <h1 className="text-3xl font-bold mb-6">博客列表</h1>
            <div className="space-y-4">
                {posts.map(post => (
                    <Link key={post.id} href={`/blog/${post.id}`}>
                        <article className="p-4 border rounded hover:shadow transition">
                            <h2 className="text-xl font-semibold">{post.title}</h2>
                            <p className="text-gray-600 mt-1">{post.excerpt}</p>
                            <time className="text-sm text-gray-400">{post.createdAt}</time>
                        </article>
                    </Link>
                ))}
            </div>
        </div>
    );
}
```

### 3. Client Component 交互

```tsx
// components/PostEditor.tsx — 需要交互，标记为 Client Component
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

interface PostEditorProps {
    initialTitle?: string;
    initialContent?: string;
}

export function PostEditor({ initialTitle = '', initialContent = '' }: PostEditorProps) {
    const [title, setTitle] = useState(initialTitle);
    const [content, setContent] = useState(initialContent);
    const [submitting, setSubmitting] = useState(false);
    const router = useRouter();

    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        setSubmitting(true);
        try {
            const res = await fetch('/api/posts', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ title, content }),
            });
            if (res.ok) {
                router.push('/blog');
                router.refresh(); // 刷新 Server Component 数据
            }
        } finally {
            setSubmitting(false);
        }
    };

    return (
        <form onSubmit={handleSubmit} className="space-y-4">
            <input
                value={title}
                onChange={e => setTitle(e.target.value)}
                placeholder="文章标题"
                className="w-full border rounded px-3 py-2"
            />
            <textarea
                value={content}
                onChange={e => setContent(e.target.value)}
                placeholder="文章内容"
                rows={10}
                className="w-full border rounded px-3 py-2"
            />
            <button
                type="submit"
                disabled={submitting}
                className="bg-blue-600 text-white px-6 py-2 rounded disabled:opacity-50"
            >
                {submitting ? '提交中...' : '发布文章'}
            </button>
        </form>
    );
}
```

### 4. 动态路由与 generateStaticParams

```tsx
// app/blog/[slug]/page.tsx — 动态路由 + SSG
interface Post {
    title: string;
    content: string;
    author: string;
    createdAt: string;
}

async function getPost(slug: string): Promise<Post> {
    const res = await fetch(`https://api.example.com/posts/${slug}`, {
        cache: 'force-cache',
    });
    return res.json();
}

// 构建时预生成所有静态页面
export async function generateStaticParams() {
    const posts = await fetch('https://api.example.com/posts').then(r => r.json());
    return posts.map((post: { slug: string }) => ({
        slug: post.slug,
    }));
}

// 动态元数据生成
export async function generateMetadata({ params }: { params: { slug: string } }) {
    const post = await getPost(params.slug);
    return {
        title: `${post.title} - 我的博客`,
        description: post.content.slice(0, 160),
    };
}

export default async function BlogPostPage({ params }: { params: { slug: string } }) {
    const post = await getPost(params.slug);

    return (
        <article className="max-w-3xl mx-auto py-8">
            <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
            <div className="text-gray-500 mb-8">
                {post.author} · {post.createdAt}
            </div>
            <div className="prose lg:prose-lg">{post.content}</div>
        </article>
    );
}
```

### 5. ISR 增量静态再生成

```tsx
// app/products/[id]/page.tsx — ISR：构建时生成 + 定时刷新
interface Product {
    id: string;
    name: string;
    price: number;
    description: string;
    stock: number;
}

async function getProduct(id: string): Promise<Product> {
    const res = await fetch(`https://api.example.com/products/${id}`, {
        next: { revalidate: 60 }, // 每 60 秒重新验证
    });
    return res.json();
}

// 页面级 ISR 配置
export const revalidate = 60;

export async function generateStaticParams() {
    const products = await fetch('https://api.example.com/products').then(r => r.json());
    return products.map((p: Product) => ({ id: p.id }));
}

export default async function ProductPage({ params }: { params: { id: string } }) {
    const product = await getProduct(params.id);

    return (
        <div className="max-w-4xl mx-auto py-8">
            <h1 className="text-3xl font-bold">{product.name}</h1>
            <p className="text-2xl text-red-600 mt-2">&yen;{product.price}</p>
            <p className="text-gray-600 mt-4">{product.description}</p>
            <p className="mt-4">
                库存：{product.stock > 0 ? `${product.stock} 件` : '已售罄'}
            </p>
        </div>
    );
}
```

**按需 Revalidation（API 触发）：**

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
    const body = await request.json();

    // 方式一：按路径刷新
    revalidatePath(`/products/${body.id}`);

    // 方式二：按标签刷新（fetch 时设置 tags）
    revalidateTag('products');

    return NextResponse.json({ revalidated: true, now: Date.now() });
}
```

```tsx
// 在 fetch 中使用 tag
const res = await fetch('https://api.example.com/products', {
    next: { tags: ['products'] }, // 可被 revalidateTag('products') 刷新
});
```

### 6. Server Actions 表单处理

```tsx
// app/posts/new/page.tsx — Server Actions 处理表单
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

// Server Action
async function createPost(formData: FormData) {
    'use server';

    const title = formData.get('title') as string;
    const content = formData.get('content') as string;

    if (!title || !content) return;

    await db.post.create({
        data: { title, content, authorId: 'current-user' },
    });

    revalidatePath('/posts'); // 刷新文章列表缓存
    redirect('/posts');       // 重定向到列表页
}

export default function NewPostPage() {
    return (
        <div className="max-w-2xl mx-auto py-8">
            <h1 className="text-2xl font-bold mb-6">发布新文章</h1>
            <form action={createPost} className="space-y-4">
                <div>
                    <label className="block text-sm font-medium mb-1">标题</label>
                    <input
                        name="title"
                        className="w-full border rounded px-3 py-2"
                        placeholder="输入文章标题"
                    />
                </div>
                <div>
                    <label className="block text-sm font-medium mb-1">内容</label>
                    <textarea
                        name="content"
                        rows={8}
                        className="w-full border rounded px-3 py-2"
                        placeholder="输入文章内容"
                    />
                </div>
                <button
                    type="submit"
                    className="bg-blue-600 text-white px-6 py-2 rounded"
                >
                    发布
                </button>
            </form>
        </div>
    );
}
```

**Server Actions + useActionState（带状态反馈）：**

```tsx
// components/PostForm.tsx
'use client';

import { useActionState } from 'react';

async function submitPost(prevState: { message: string }, formData: FormData) {
    'use server';

    const title = formData.get('title') as string;
    if (!title) return { message: '标题不能为空' };

    await db.post.create({ data: { title } });
    revalidatePath('/posts');
    return { message: '发布成功！' };
}

export function PostForm() {
    const [state, formAction] = useActionState(submitPost, { message: '' });

    return (
        <form action={formAction}>
            <input name="title" className="border rounded px-3 py-2" />
            {state.message && <p className="text-sm mt-2">{state.message}</p>}
            <button type="submit" className="ml-2 bg-blue-600 text-white px-4 py-2 rounded">
                发布
            </button>
        </form>
    );
}
```

### 7. Middleware 中间件（鉴权、重定向）

```tsx
// middleware.ts — 位于项目根目录或 src/ 目录
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
    const { pathname } = request.nextUrl;

    // 1. 鉴权检查
    const token = request.cookies.get('auth-token')?.value;
    if (pathname.startsWith('/dashboard') && !token) {
        const loginUrl = new URL('/login', request.url);
        loginUrl.searchParams.set('callbackUrl', pathname);
        return NextResponse.redirect(loginUrl);
    }

    // 2. 已登录用户访问登录页，重定向到仪表盘
    if (pathname === '/login' && token) {
        return NextResponse.redirect(new URL('/dashboard', request.url));
    }

    // 3. 添加自定义请求头
    const response = NextResponse.next();
    response.headers.set('x-request-id', crypto.randomUUID());
    return response;

    // 4. 地区重定向（i18n）
    // const country = request.geo?.country || 'CN';
    // if (pathname === '/') {
    //     return NextResponse.redirect(new URL(`/${country.toLowerCase()}`, request.url));
    // }
}

// 匹配配置 — 只对这些路径运行中间件
export const config = {
    matcher: [
        '/dashboard/:path*',
        '/login',
        '/api/:path*',
    ],
};
```

### 8. 嵌套布局与 Loading 状态

```tsx
// app/layout.tsx — 根布局
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
    title: '我的应用',
    description: 'Next.js App Router 示例',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
    return (
        <html lang="zh-CN">
            <body>
                <nav className="border-b px-6 py-3">
                    <a href="/">首页</a>
                    <a href="/blog">博客</a>
                    <a href="/dashboard">仪表盘</a>
                </nav>
                {children}
            </body>
        </html>
    );
}
```

```tsx
// app/dashboard/layout.tsx — 嵌套布局
export default function DashboardLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <div className="flex">
            <aside className="w-64 border-r p-4">
                <nav>
                    <a href="/dashboard/overview">概览</a>
                    <a href="/dashboard/analytics">分析</a>
                    <a href="/dashboard/settings">设置</a>
                </nav>
            </aside>
            <main className="flex-1 p-6">{children}</main>
        </div>
    );
}
```

```tsx
// app/dashboard/loading.tsx — 自动 Suspense 骨架屏
export default function DashboardLoading() {
    return (
        <div className="animate-pulse space-y-4">
            <div className="h-8 bg-gray-200 rounded w-1/3" />
            <div className="h-4 bg-gray-200 rounded w-2/3" />
            <div className="grid grid-cols-3 gap-4">
                {[1, 2, 3].map(i => (
                    <div key={i} className="h-32 bg-gray-200 rounded" />
                ))}
            </div>
        </div>
    );
}
```

### 9. Image 图片优化

```tsx
// components/HeroSection.tsx
import Image from 'next/image';

export function HeroSection() {
    return (
        <section className="relative h-[500px]">
            {/* 本地图片 — 自动获取宽高 */}
            <Image
                src="/hero.jpg"
                alt="首页横幅"
                fill
                priority          // 优先加载（首屏图片）
                sizes="100vw"
                className="object-cover"
            />

            {/* 远程图片 — 需配置域名 */}
            <Image
                src="https://cdn.example.com/photo.jpg"
                alt="远程图片"
                width={800}
                height={600}
                quality={85}       // 质量 1-100
                placeholder="blur" // 模糊占位
            />
        </section>
    );
}
```

```js
// next.config.js — 远程图片域名配置
/** @type {import('next').NextConfig} */
const nextConfig = {
    images: {
        remotePatterns: [
            {
                protocol: 'https',
                hostname: 'cdn.example.com',
            },
        ],
        formats: ['image/avif', 'image/webp'],
    },
};

module.exports = nextConfig;
```

### 10. Route Handlers（API 路由）

```tsx
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/posts
export async function GET(request: NextRequest) {
    const searchParams = request.nextUrl.searchParams;
    const page = searchParams.get('page') || '1';
    const limit = searchParams.get('limit') || '10';

    const posts = await db.post.findMany({
        skip: (Number(page) - 1) * Number(limit),
        take: Number(limit),
    });

    return NextResponse.json({ posts, page: Number(page) });
}

// POST /api/posts
export async function POST(request: NextRequest) {
    const body = await request.json();
    const { title, content } = body;

    if (!title) {
        return NextResponse.json({ error: '标题必填' }, { status: 400 });
    }

    const post = await db.post.create({ data: { title, content } });
    return NextResponse.json(post, { status: 201 });
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| "use client" 组件中用了服务端 API | Client Component 不运行在服务端 | 数据在 Server Component 获取，通过 props 传递 |
| useState 在 Server Component 报错 | Server Component 无状态 | 拆成 Client Component 或提升状态 |
| 构建慢 | SSG 页面过多 | 用 ISR 延迟生成，或动态路由 |
| hydration mismatch | SSR 和 CSR 渲染结果不一致 | 避免在渲染中使用 `Date.now()`/`Math.random()` |
| Image 组件远程图片 403 | 未配置 `remotePatterns` | 在 `next.config.js` 添加域名白名单 |
| Server Action 无响应 | 忘记 `'use server'` 声明 | 函数顶部添加 `'use server'` |
| Middleware 中读数据库 | Middleware 运行在 Edge Runtime | 用 Cookies/Headers 判断，数据库查询放 Server Component |
| 布局刷新导致状态丢失 | Layout 导航时保持不重新挂载 | 需要重置状态的场景用 `template.tsx` |
| fetch 在开发模式重复调用 | React StrictMode 双重渲染 | 仅开发环境出现，生产环境正常 |
| 动态路由 params 类型报错 | Next.js 15+ params 是 Promise | 使用 `await params` 获取参数 |

### 最佳实践

- 默认用 Server Component，交互部分才用 `"use client"`
- 数据获取在 Server Component 中直接 async/await，避免 useEffect 瀑布
- 用 `loading.tsx` + Streaming 提升感知性能
- 静态内容用 SSG，动态内容用 SSR/ISR
- Server Actions 优先于手动 API Routes（简单表单场景）
- 图片始终用 `<Image>` 组件，自动优化格式和尺寸
- 布局嵌套不要过深，3 层以内为宜
- 用 `generateStaticParams` 预生成热门页面，减少服务器压力
- Middleware 保持轻量，仅做鉴权和重定向，不做重计算
- 合理使用 `revalidate` 值，避免全站设为 0（变成纯 SSR）

## 面试题

**Q1: SSR、SSG、CSR 三种渲染模式有什么区别？各适用什么场景？**
> SSR 每次请求时服务端渲染 HTML，数据实时但服务器压力大，适用于个性化页面和实时数据；SSG 构建时生成静态 HTML，速度最快但内容固定，适用于博客/文档/营销页；CSR 客户端加载 JS 后渲染，SEO 差、首屏慢但交互灵活，适用于后台管理系统。ISR 是 SSG 的增强版，通过 revalidate 间隔实现准实时更新，适用于电商商品页等需要兼顾性能和时效的场景。

**Q2: SSR 和 SSG 的区别？各适用什么场景？**
> SSR 每次请求时在服务端实时渲染 HTML，数据始终是最新的，但服务器负载高、TTFB 较长；SSG 在构建时一次性生成静态 HTML，CDN 直接返回，速度极快且服务器零负载，但数据更新需要重新构建。SSR 适用于用户个性化仪表盘、实时股票行情等数据频繁变化且需要 SEO 的场景；SSG 适用于博客文章、产品介绍、文档站等内容稳定、更新频率低的场景。ISR 是两者折中，用 revalidate 定时刷新。

**Q3: getServerSideProps 和 getStaticProps 的区别是什么？（Pages Router）**
> `getServerSideProps` 每次请求时在服务端执行，获取实时数据，适用于动态页面；`getStaticProps` 构建时执行一次，生成静态 HTML，配合 `revalidate` 可实现 ISR 增量更新，适合内容不频繁变化的页面。在 App Router 中，两者统一为 fetch 的 cache 选项：`cache: 'no-store'` 等价 getServerSideProps，`cache: 'force-cache'` 等价 getStaticProps。

**Q4: 什么是 Hydration？Hydration Mismatch 是怎么产生的？**
> Hydration 是服务端渲染的 HTML 在客户端被 React"激活"，绑定事件和状态的过程。Mismatch 产生于 SSR 和 CSR 渲染结果不一致，常见原因：渲染中使用了 `Date.now()`、`Math.random()`、浏览器 API（window）或依赖了客户端状态。解决方案：将使用浏览器 API 的逻辑移入 useEffect（仅在客户端执行），或使用 `suppressHydrationWarning` 抑制已知差异。

**Q5: Next.js 的 Server Components 和 Client Components 有什么区别？**
> Server Component 在服务端执行，可直接访问数据库/文件系统，不发送 JS 到客户端，支持 async/await 但不支持 useState/useEffect/onClick；Client Component 通过 `"use client"` 声明，在浏览器执行，支持状态管理和事件处理但会增加客户端 JS 体积。App Router 下组件默认是 Server Component，只有需要交互的组件才标记为 Client Component。关键原则：Server Component 可以导入 Client Component，但 Client Component 不能导入 Server Component（可通过 props 传递 JSX）。

**Q6: 什么是 ISR？解决了什么问题？**
> ISR（Incremental Static Regeneration，增量静态再生）是 Next.js 的渲染策略，在 SSG 基础上增加了定时重新验证能力。通过 `revalidate` 设置间隔秒数，过期后后台重新生成页面，新请求仍返回旧缓存，生成完成后下次请求返回新页面。ISR 解决了 SSG 内容更新需要全站重新构建的问题，也避免了 SSR 的高服务器负载，实现了静态页面的"准实时"更新，特别适合电商商品页、新闻列表等内容更新频率中等、流量较大的页面。

**Q7: Next.js 的 Middleware 可以做什么？**
> Middleware 是运行在 Edge Runtime 的请求拦截器，在请求到达页面之前执行。主要用途：1）鉴权 — 检查 Cookie/Token，未登录重定向到登录页；2）重定向 — 根据条件跳转（如地区、语言、设备类型）；3）请求改写 — URL Rewrite 不改变浏览器地址栏但返回不同内容；4）添加自定义 Headers — 如请求 ID、CORS 头；5）A/B 测试 — 根据 Cookie 分配不同页面版本。注意 Middleware 运行在 Edge Runtime，不能访问 Node.js API（如 fs、数据库），应保持轻量。

**Q8: App Router 的 Layout 和 Template 有什么区别？**
> Layout 在导航时保持不重新挂载，共享状态和副作用保持不变，适合全局导航栏、侧边栏等持久化 UI；Template 在导航时重新挂载，会重新执行 useEffect 和重置状态，适合需要在路由切换时重置的场景（如页面进入动画、表单状态重置）。默认优先用 Layout，只有需要重新挂载行为时才用 Template。

---

**相关链接：**
- [[React核心]]
- [[React Hooks详解]]
- [[React Server Components]]
- [[React生态Router与Query]]
- [[HTTP与缓存策略]]
- [[前端性能优化]]
- [[Nuxt.js与Vue SSR]]

---
tags:
  - Web前端
  - 框架
  - React
  - Server Components
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# React Server Components

## What — 是什么

> React Server Components（RSC）是 React 18+ 引入的服务端渲染架构，允许组件在服务端执行、仅将渲染结果序列化发送到客户端，从根本上减少客户端 JavaScript 体积。

**三种组件类型：**

| 类型 | 执行位置 | JS 发送到客户端 | 状态/Hooks | 典型用途 |
|------|---------|---------------|-----------|---------|
| Server Components | 服务端 | 否（零 JS） | 不支持 | 数据获取、静态内容、后端 SDK |
| Client Components | 客户端 | 是 | 支持 useState/useEffect | 交互组件、浏览器 API |
| Shared Components | 双端 | 视使用方而定 | 仅客户端时支持 | 纯展示组件（无状态无副作用） |

**核心架构：**

- 设计理念：组件按职责在服务端或客户端执行，服务端组件的代码永远不会被下载到浏览器
- 核心协议：React Flight — 服务端将组件树序列化为流式 JSON，客户端逐步接收并渲染
- 数据流：请求 → 路由匹配 → Server Component 执行（async/await 获取数据） → 序列化为 Flight 数据 → 客户端接收并渲染 HTML/JS

**关键特性：**

- Server Components 默认无需声明，只有 Client Components 需要 `'use client'`
- Server Components 可以直接 `async/await` 获取数据，无需 `useEffect`
- Server Components 导入的 npm 包不会增加客户端 bundle
- Client Components 通过 `'use client'` 声明，是客户端渲染的边界

## Why — 为什么

**适用场景：**

- 数据密集型页面（电商商品列表、仪表盘、博客详情）
- SEO 要求高的内容站点
- 首屏性能敏感的 C 端应用
- 需要直接访问后端资源（数据库、文件系统、私有 API）的场景

**对比：Server Components vs Client Components vs SSR**

| 维度 | Server Components | Client Components | 传统 SSR |
|------|-------------------|-------------------|---------|
| 执行位置 | 服务端 | 客户端（Hydration） | 服务端渲染 + 客户端 Hydration |
| JS 体积 | 零（不发送组件 JS） | 完整（含依赖） | 完整（所有组件 JS 都发送） |
| 状态管理 | 无（不支持 useState） | 支持 useState/useReducer | 支持 |
| Hooks | 仅自定义 Hook（纯逻辑） | 全部 Hooks | 全部 Hooks |
| 后端 API 访问 | 直接访问数据库/文件系统 | 仅通过 fetch/API | 仅通过 getServerSideProps |
| 交互能力 | 无（无事件处理） | onClick/onSubmit 等 | 有（Hydration 后） |
| 数据获取 | async/await 直接获取 | useEffect / SWR / React Query | getServerSideProps / getStaticProps |
| 何时使用 | 数据获取、静态内容、后端逻辑 | 交互组件、浏览器 API | 传统 React SSR 方案 |

**RSC 带来的核心优势：**

- **减少客户端 JS 体积**：Server Components 及其依赖不发送到客户端，大型组件库（markdown 渲染器、日期库）零成本
- **直接访问后端资源**：无需 API 层，Server Component 内直接读取数据库、文件系统、环境变量
- **自动代码分割**：Client Components 边界天然是代码分割点，无需手动 `React.lazy`
- **SEO 友好**：Server Components 渲染的 HTML 直接被搜索引擎抓取
- **Streaming 支持**：配合 Suspense 实现流式渲染，渐进式展示内容

**优缺点：**

- ✅ 优点：
  - 显著减少客户端 JS，提升首屏性能
  - 直接访问后端资源，简化数据获取链路
  - 组件模型统一，Server/Client 用相同 API
  - 流式传输，渐进式渲染
- ❌ 缺点：
  - Server/Client 边界规则增加心智负担
  - 第三方组件默认 Client Component，难以优化
  - 生态仍在成熟中，调试工具有限
  - 需要框架支持（Next.js App Router / Remix）

## How — 怎么用

### 1. `'use client'` 与 `'use server'` 指令详解

```tsx
// 'use client' — 声明此文件为 Client Component
// 必须在文件最顶部（在 import 之前）
'use client';

import { useState } from 'react';

export function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

```tsx
// 'use server' — 声明此文件中的函数为 Server Action
// 必须在文件最顶部
'use server';

import { revalidatePath } from 'next/cache';
import db from '@/lib/db';

export async function createPost(formData: FormData) {
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;

    await db.post.create({ data: { title, content } });
    revalidatePath('/posts'); // 刷新缓存
}
```

**指令作用范围规则：**

| 指令 | 位置 | 作用范围 | 谁可以使用 |
|------|------|---------|-----------|
| `'use client'` | 文件顶部 | 该文件导出的所有组件为 Client Component | 任何组件文件 |
| `'use server'` | 文件顶部 | 该文件导出的所有函数为 Server Action | 仅包含异步函数的文件 |
| `'use server'` | 函数体内 | 标记单个函数为 Server Action | Client Component 内的异步函数 |

### 2. Server Components — 数据获取与零客户端 JS

```tsx
// app/posts/page.tsx — 默认就是 Server Component，无需声明
import db from '@/lib/db';
import { formatDate } from '@/lib/utils';  // 不会发送到客户端
import Markdown from 'react-markdown';      // 不会发送到客户端
import { PostActions } from './PostActions'; // Client Component

// Server Component 可以直接 async/await
export default async function PostsPage() {
    // 直接访问数据库，无需 API
    const posts = await db.post.findMany({
        orderBy: { createdAt: 'desc' },
        take: 20,
    });

    return (
        <div>
            <h1>文章列表</h1>
            {posts.map(post => (
                <article key={post.id}>
                    <h2>{post.title}</h2>
                    {/* Markdown 渲染器在服务端执行，不增加客户端 bundle */}
                    <Markdown>{post.content}</Markdown>
                    <p>发布于 {formatDate(post.createdAt)}</p>
                    {/* 交互部分委托给 Client Component */}
                    <PostActions postId={post.id} />
                </article>
            ))}
        </div>
    );
}
```

**Server Component 能做什么：**

```tsx
// ✅ 直接读取数据库
const user = await db.user.findUnique({ where: { id } });

// ✅ 直接读取文件系统
import { readFile } from 'fs/promises';
const content = await readFile('./data/config.json', 'utf-8');

// ✅ 访问环境变量
const apiKey = process.env.PRIVATE_API_KEY; // 不会泄露到客户端

// ✅ 使用服务端 SDK
import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
const s3 = new S3Client({ region: 'us-east-1' });

// ✅ 导入重型库 — 不影响客户端 bundle
import { marked } from 'marked';      // ~30KB，客户端零负担
import dayjs from 'dayjs';             // ~5KB，客户端零负担
import { sanitize } from 'dompurify';  // 安全处理在服务端完成
```

**Server Component 不能做什么：**

```tsx
// ❌ 不能使用 useState / useReducer
const [count, setCount] = useState(0); // Error!

// ❌ 不能使用 useEffect / useLayoutEffect
useEffect(() => { ... }, []); // Error!

// ❌ 不能使用浏览器 API
window.localStorage; // Error!
document.getElementById('root'); // Error!

// ❌ 不能绑定事件处理函数
<button onClick={() => {}}>Click</button>; // onClick 无效

// ❌ 不能使用 Context（useContext）
const theme = useContext(ThemeContext); // Error!
```

### 3. Client Components — 交互组件

```tsx
// components/SearchBar.tsx
'use client';

import { useState, useEffect, useCallback } from 'react';

interface SearchBarProps {
    onSearch: (query: string) => void;
    placeholder?: string;
}

export function SearchBar({ onSearch, placeholder = '搜索...' }: SearchBarProps) {
    const [query, setQuery] = useState('');
    const [isFocused, setIsFocused] = useState(false);

    // ✅ 可以使用 useEffect
    useEffect(() => {
        const timer = setTimeout(() => onSearch(query), 300);
        return () => clearTimeout(timer);
    }, [query, onSearch]);

    // ✅ 可以使用浏览器 API
    useEffect(() => {
        const handler = (e: KeyboardEvent) => {
            if (e.key === 'Escape') setQuery('');
        };
        window.addEventListener('keydown', handler);
        return () => window.removeEventListener('keydown', handler);
    }, []);

    return (
        <div className={`search-bar ${isFocused ? 'focused' : ''}`}>
            <input
                value={query}
                onChange={e => setQuery(e.target.value)}
                onFocus={() => setIsFocused(true)}
                onBlur={() => setIsFocused(false)}
                placeholder={placeholder}
            />
        </div>
    );
}
```

```tsx
// components/ThemeProvider.tsx — Client Component 中使用 Context
'use client';

import { createContext, useContext, useState, useEffect } from 'react';

const ThemeContext = createContext<'light' | 'dark'>('light');

export function ThemeProvider({ children }: { children: React.ReactNode }) {
    const [theme, setTheme] = useState<'light' | 'dark'>('light');

    useEffect(() => {
        // 读取用户偏好
        const saved = localStorage.getItem('theme') as 'light' | 'dark';
        if (saved) setTheme(saved);
    }, []);

    return (
        <ThemeContext.Provider value={theme}>
            {children}
        </ThemeContext.Provider>
    );
}

export function useTheme() {
    return useContext(ThemeContext);
}
```

### 4. 边界规则：Server 与 Client 的交互

**Server → Client（通过 props 传递数据）：**

```tsx
// ✅ Server Component 导入并渲染 Client Component
// app/dashboard/page.tsx（Server Component）
import { DashboardChart } from './DashboardChart'; // Client Component
import { getUserStats } from '@/lib/data';

export default async function Dashboard() {
    const stats = await getUserStats(); // 服务端获取数据

    // 通过 props 传递给 Client Component
    // ⚠️ props 必须可序列化
    return <DashboardChart data={stats} />;
}
```

```tsx
// components/DashboardChart.tsx（Client Component）
'use client';

import { useState } from 'react';
import { Chart } from 'react-chartjs-2';

interface DashboardChartProps {
    data: { label: string; value: number }[]; // 可序列化的类型
}

export function DashboardChart({ data }: DashboardChartProps) {
    const [chartType, setChartType] = useState<'bar' | 'line'>('bar');

    return (
        <div>
            <button onClick={() => setChartType('bar')}>柱状图</button>
            <button onClick={() => setChartType('line')}>折线图</button>
            <Chart type={chartType} data={data} />
        </div>
    );
}
```

**Props 可序列化限制：**

```tsx
// ✅ 可序列化的 props
<Component
    string="hello"
    number={42}
    boolean={true}
    null={null}
    array={[1, 2, 3]}
    object={{ key: 'value' }}
    date={new Date().toISOString()}  // 转为字符串
/>

// ❌ 不可序列化的 props
<Component
    function={() => {}}          // 函数不可序列化
    class={new MyClass()}        // 类实例不可序列化
    symbol={Symbol('id')}        // Symbol 不可序列化
    reactElement={<div />}       // React Element 理论上可以，但不推荐
/>
```

**Client → Server（通过 Server Actions）：**

```tsx
// app/posts/page.tsx（Server Component）
import { createPost } from './actions'; // Server Action
import { PostForm } from './PostForm';  // Client Component

export default function NewPostPage() {
    return <PostForm action={createPost} />;
}
```

```tsx
// app/posts/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import db from '@/lib/db';

export async function createPost(formData: FormData) {
    const title = formData.get('title') as string;
    const content = formData.get('content') as string;

    await db.post.create({ data: { title, content } });

    // 数据变更后刷新缓存
    revalidatePath('/posts');
}
```

### 5. Server Actions 详解

**基本用法 — 表单 action：**

```tsx
// app/login/page.tsx（Server Component）
import { login } from './actions';

export default function LoginPage() {
    // 直接将 Server Action 传给 form 的 action
    // 即使 JS 未加载，表单也能正常提交（渐进增强）
    return (
        <form action={login}>
            <input name="email" type="email" required />
            <input name="password" type="password" required />
            <button type="submit">登录</button>
        </form>
    );
}
```

```tsx
// app/login/actions.ts
'use server';

import { redirect } from 'next/navigation';
import { cookies } from 'next/headers';
import { verifyPassword } from '@/lib/auth';

export async function login(formData: FormData) {
    const email = formData.get('email') as string;
    const password = formData.get('password') as string;

    const user = await verifyPassword(email, password);
    if (!user) {
        // 返回错误信息（通过 useActionState 获取）
        return { error: '邮箱或密码错误' };
    }

    // 设置 cookie
    const cookieStore = await cookies();
    cookieStore.set('session', user.token, {
        httpOnly: true,
        secure: true,
        sameSite: 'lax',
        maxAge: 60 * 60 * 24 * 7, // 7 天
    });

    redirect('/dashboard');
}
```

**useActionState — 处理 Server Action 返回值：**

```tsx
// components/PostForm.tsx（Client Component）
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

type State = { error?: string } | null;

export function PostForm() {
    const [state, formAction, isPending] = useActionState<State, FormData>(
        async (prevState, formData) => {
            return await createPost(formData);
        },
        null,
    );

    return (
        <form action={formAction}>
            <input name="title" required placeholder="标题" />
            <textarea name="content" required placeholder="内容" />
            {state?.error && <p className="error">{state.error}</p>}
            <button type="submit" disabled={isPending}>
                {isPending ? '发布中...' : '发布文章'}
            </button>
        </form>
    );
}
```

**Server Action 内联定义：**

```tsx
// app/todos/page.tsx（Server Component）
import { revalidatePath } from 'next/cache';
import db from '@/lib/db';

export default async function TodosPage() {
    const todos = await db.todo.findMany();

    async function addTodo(formData: FormData) {
        'use server'; // 内联 Server Action
        const text = formData.get('text') as string;
        await db.todo.create({ data: { text } });
        revalidatePath('/todos');
    }

    async function deleteTodo(formData: FormData) {
        'use server';
        const id = formData.get('id') as string;
        await db.todo.delete({ where: { id } });
        revalidatePath('/todos');
    }

    return (
        <div>
            <form action={addTodo}>
                <input name="text" required />
                <button>添加</button>
            </form>
            <ul>
                {todos.map(todo => (
                    <li key={todo.id}>
                        {todo.text}
                        <form action={deleteTodo}>
                            <input type="hidden" name="id" value={todo.id} />
                            <button>删除</button>
                        </form>
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

**revalidatePath 与 revalidateTag：**

```tsx
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

// revalidatePath — 按路径刷新
export async function updatePost(slug: string, formData: FormData) {
    await db.post.update({ where: { slug }, data: { ... } });

    revalidatePath(`/posts/${slug}`);    // 刷新指定页面
    revalidatePath('/posts');             // 刷新列表页
    revalidatePath('/', 'layout');        // 刷新 layout 下所有页面
}

// revalidateTag — 按标签刷新（精细控制）
export async function createComment(formData: FormData) {
    await db.comment.create({ data: { ... } });

    revalidateTag('comments');            // 刷新所有标记 'comments' 标签的请求
    revalidateTag('post-stats');          // 刷新标记 'post-stats' 标签的请求
}
```

### 6. 数据获取模式

**Server Component 直接 async/await：**

```tsx
// app/product/[id]/page.tsx
import db from '@/lib/db';

export default async function ProductPage({ params }: { params: { id: string } }) {
    // 直接在组件中 await，无需 useEffect
    const product = await db.product.findUnique({
        where: { id: params.id },
        include: { reviews: true, category: true },
    });

    if (!product) return <div>商品不存在</div>;

    return (
        <div>
            <h1>{product.name}</h1>
            <p>{product.description}</p>
            <span>¥{product.price}</span>
        </div>
    );
}
```

**并行数据获取：**

```tsx
// ✅ 并行获取 — 两个请求同时发出
export default async function DashboardPage() {
    // Promise.all 并行获取，总耗时 = max(用户, 通知)
    const [user, notifications] = await Promise.all([
        getUser(),
        getNotifications(),
    ]);

    return (
        <div>
            <h1>欢迎, {user.name}</h1>
            <NotificationList items={notifications} />
        </div>
    );
}

// ❌ 串行获取 — 两个请求依次发出，总耗时 = 用户 + 通知
export default async function DashboardPageSlow() {
    const user = await getUser();             // 等待完成
    const notifications = await getNotifications(); // 再发起

    return <div>...</div>;
}
```

**Streaming + Suspense — 渐进式渲染：**

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function DashboardPage() {
    return (
        <div>
            <h1>仪表盘</h1>

            {/* 每个部分独立加载，互不阻塞 */}
            <Suspense fallback={<StatsSkeleton />}>
                <Stats />  {/* async Server Component */}
            </Suspense>

            <Suspense fallback={<ChartSkeleton />}>
                <RevenueChart />
            </Suspense>

            <Suspense fallback={<ListSkeleton />}>
                <RecentOrders />
            </Suspense>
        </div>
    );
}

// 每个子组件独立获取数据
async function Stats() {
    const stats = await getStats(); // 可能需要 2s
    return <div>{stats.map(s => <StatCard key={s.id} {...s} />)}</div>;
}

async function RevenueChart() {
    const revenue = await getRevenue(); // 可能需要 3s
    return <Chart data={revenue} />;
}

async function RecentOrders() {
    const orders = await getRecentOrders(); // 可能需要 1s
    return <OrderList orders={orders} />;
}
```

### 7. Next.js App Router 中的 RSC

**文件约定与默认组件类型：**

| 文件 | 默认类型 | 说明 |
|------|---------|------|
| `layout.tsx` | Server Component | 共享布局，路由切换时保持 |
| `page.tsx` | Server Component | 页面组件，支持 async |
| `loading.tsx` | Server Component | Suspense fallback |
| `error.tsx` | **Client Component** | 必须是 Client Component（需要交互） |
| `global-error.tsx` | **Client Component** | 根布局错误边界 |
| `not-found.tsx` | Server Component | 404 页面 |
| `template.tsx` | Server Component | 类似 layout 但每次路由切换重建 |
| `default.tsx` | Server Component | Parallel Routes 的 fallback |

**layout.tsx 与 page.tsx：**

```tsx
// app/layout.tsx — 根布局（Server Component）
import './globals.css';
import { ThemeProvider } from '@/components/ThemeProvider'; // Client Component

export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body>
                {/* Context Provider 必须包在 Client Component 内 */}
                <ThemeProvider>
                    <nav>
                        <a href="/">首页</a>
                        <a href="/about">关于</a>
                    </nav>
                    <main>{children}</main>
                </ThemeProvider>
            </body>
        </html>
    );
}
```

```tsx
// app/page.tsx — 首页（Server Component）
import { getFeaturedPosts } from '@/lib/data';
import { PostCard } from '@/components/PostCard';

export default async function HomePage() {
    const posts = await getFeaturedPosts();

    return (
        <section>
            <h1>精选文章</h1>
            <div className="grid">
                {posts.map(post => (
                    <PostCard key={post.id} post={post} />
                ))}
            </div>
        </section>
    );
}
```

**loading.tsx 与 error.tsx：**

```tsx
// app/dashboard/loading.tsx — 自动 Suspense boundary
export default function Loading() {
    return (
        <div className="skeleton">
            <div className="skeleton-title" />
            <div className="skeleton-card" />
            <div className="skeleton-card" />
        </div>
    );
}
```

```tsx
// app/dashboard/error.tsx — 必须是 Client Component
'use client';

export default function Error({
    error,
    reset,
}: {
    error: Error & { digest?: string };
    reset: () => void;
}) {
    return (
        <div>
            <h2>出错了</h2>
            <p>{error.message}</p>
            <button onClick={reset}>重试</button>
        </div>
    );
}
```

**Route Handlers：**

```tsx
// app/api/users/route.ts — API 路由（不是 RSC，是标准的 HTTP 处理）
import { NextResponse } from 'next/server';
import db from '@/lib/db';

export async function GET(request: Request) {
    const users = await db.user.findMany();
    return NextResponse.json(users);
}

export async function POST(request: Request) {
    const body = await request.json();
    const user = await db.user.create({ data: body });
    return NextResponse.json(user, { status: 201 });
}
```

### 8. 缓存策略

**fetch 缓存选项：**

```tsx
// 默认缓存（force-cache）— 适合静态内容
const data = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // 默认值，缓存直到 revalidate
});

// 不缓存（no-store）— 适合实时数据
const data = await fetch('https://api.example.com/live', {
    cache: 'no-store', // 每次请求都重新获取
});

// 定时重新验证（ISR）
const data = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }, // 每 60 秒重新验证
});

// 按标签重新验证
const data = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }, // 可通过 revalidateTag('posts') 按需刷新
});
```

**路由级缓存配置：**

```tsx
// app/posts/page.tsx — 路由段配置

// 静态生成（默认）
export const dynamic = 'force-static';

// 动态渲染
export const dynamic = 'force-dynamic';

// ISR — 定时重新验证
export const revalidate = 60; // 秒

// 不重新验证
export const revalidate = false;

export default async function PostsPage() {
    const posts = await fetch('https://api.example.com/posts').then(r => r.json());
    return <PostList posts={posts} />;
}
```

**ISR — 增量静态再生：**

```tsx
// app/blog/[slug]/page.tsx
export const revalidate = 3600; // 每小时重新验证

export async function generateStaticParams() {
    const posts = await getAllPosts();
    return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
    const post = await getPost(params.slug);
    return <article>{post.content}</article>;
}
```

**On-demand Revalidation — 按需刷新：**

```tsx
// app/api/revalidate/route.ts — Webhook 触发刷新
import { revalidatePath, revalidateTag } from 'next/server';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
    const body = await request.json();
    const secret = body.secret;

    // 验证请求合法性
    if (secret !== process.env.REVALIDATION_SECRET) {
        return Response.json({ error: 'Invalid secret' }, { status: 401 });
    }

    // 按路径刷新
    if (body.path) {
        revalidatePath(body.path);
    }

    // 按标签刷新
    if (body.tag) {
        revalidateTag(body.tag);
    }

    return Response.json({ revalidated: true, now: Date.now() });
}
```

**缓存策略选择指南：**

| 场景 | 策略 | 配置 |
|------|------|------|
| 静态内容（文档、博客） | SSG | `force-static` 或 `revalidate: false` |
| 半动态内容（商品页） | ISR | `revalidate: 3600` |
| 实时数据（股价、评论） | 动态渲染 | `force-dynamic` 或 `cache: 'no-store'` |
| CMS 内容更新后刷新 | On-demand | `revalidateTag()` / `revalidatePath()` |

### 9. RSC 序列化协议简述

**React Flight 协议：**

React Server Components 的核心是 Flight 协议——一种将服务端组件树序列化并流式传输到客户端的协议。

```
工作流程：
1. 服务端执行 Server Component 树
2. 将渲染结果序列化为 Flight 数据格式（JSON 行流）
3. 客户端逐步接收 Flight 数据
4. React 在客户端重建组件树并渲染

Flight 数据示例（简化）：
0:["$","div",null,{"children":[...]}]        // HTML 结构
1:["$","$L2",null,{"data":{...}}]             // Client Component 引用
2:{"module":"./Counter.js","exports":["default"]} // 模块引用
```

**流式传输特性：**

```tsx
// Suspense 边界实现流式传输
export default async function Page() {
    return (
        <div>
            <h1>页面标题</h1>              {/* 立即发送 */}
            <Suspense fallback={<Loading />}>
                <SlowComponent />           {/* 就绪后流式追加 */}
            </Suspense>
        </div>
    );
}

// 服务端发送顺序：
// 1. <h1>页面标题</h1> + <Loading />
// 2. SlowComponent 就绪后，发送替换指令
// 3. 客户端 React 用 SlowComponent 替换 Loading
```

**Flight 协议关键设计：**

- **模块引用**：Client Component 只发送模块 ID，客户端已有 bundle 可匹配
- **流式分块**：每个 Suspense 边界独立分块，就绪即发送
- **增量更新**：仅传输变化的部分，无需重发整个页面
- **可恢复性**：客户端可从任意断点恢复渲染

### 10. 迁移策略 — Pages Router 到 App Router

**渐进式迁移步骤：**

```bash
# 1. 在现有 Pages Router 项目中并行启用 App Router
# app/ 目录与 pages/ 目录可以共存
mkdir -p app
```

```tsx
// 2. 创建根布局 — app/layout.tsx（必须）
export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body>{children}</body>
        </html>
    );
}
```

**API 映射对照表：**

| Pages Router | App Router | 说明 |
|-------------|-----------|------|
| `pages/index.tsx` | `app/page.tsx` | 首页 |
| `pages/about.tsx` | `app/about/page.tsx` | 子页面 |
| `pages/blog/[slug].tsx` | `app/blog/[slug]/page.tsx` | 动态路由 |
| `pages/_app.tsx` | `app/layout.tsx` | 根布局 |
| `pages/_document.tsx` | `app/layout.tsx` | HTML 结构 |
| `pages/api/*.ts` | `app/api/*/route.ts` | API 路由 |
| `getServerSideProps` | Server Component async | 服务端数据获取 |
| `getStaticProps` | Server Component + `revalidate` | 静态生成 |
| `getStaticPaths` | `generateStaticParams` | 预渲染路径 |
| `<Head>` | `metadata` 导出 | 页面元数据 |

**getServerSideProps 迁移：**

```tsx
// ❌ Pages Router 方式
export async function getServerSideProps() {
    const data = await fetchData();
    return { props: { data } };
}

export default function Page({ data }) {
    return <div>{data.title}</div>;
}

// ✅ App Router 方式 — 直接在组件中 async/await
export default async function Page() {
    const data = await fetchData();
    return <div>{data.title}</div>;
}
```

**getStaticProps + ISR 迁移：**

```tsx
// ❌ Pages Router 方式
export async function getStaticPaths() {
    const posts = await getAllPosts();
    return {
        paths: posts.map(p => ({ params: { slug: p.slug } })),
        fallback: 'blocking',
    };
}

export async function getStaticProps({ params }) {
    const post = await getPost(params.slug);
    return { props: { post }, revalidate: 60 };
}

export default function Post({ post }) {
    return <article>{post.content}</article>;
}

// ✅ App Router 方式
export async function generateStaticParams() {
    const posts = await getAllPosts();
    return posts.map(p => ({ slug: p.slug }));
}

export const revalidate = 60;

export default async function Post({ params }: { params: { slug: string } }) {
    const post = await getPost(params.slug);
    return <article>{post.content}</article>;
}
```

**metadata 替代 Head：**

```tsx
// ❌ Pages Router
import Head from 'next/head';
export default function Page() {
    return (
        <>
            <Head><title>我的页面</title></Head>
            <div>内容</div>
        </>
    );
}

// ✅ App Router — 静态 metadata
export const metadata = {
    title: '我的页面',
    description: '页面描述',
};

export default function Page() {
    return <div>内容</div>;
}

// ✅ App Router — 动态 generateMetadata
export async function generateMetadata({ params }) {
    const post = await getPost(params.slug);
    return {
        title: post.title,
        description: post.excerpt,
        openGraph: { images: [post.coverImage] },
    };
}
```

## 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| props 传递函数报错 | Server → Client 的 props 必须可序列化，函数不可序列化 | 用 Server Action 替代函数 prop，或将函数逻辑移到 Client Component 内 |
| 第三方组件在 Server Component 中报错 | npm 包默认是 Client Component（无 `'use client'` 声明的包） | 将第三方组件包在 Client Component 中 |
| useState 在 Server Component 报错 | Server Component 不支持状态 | 拆分为 Client Component 处理交互 |
| Server Action 安全问题 | Server Action 暴露为 API 端点，可被直接调用 | 验证输入、检查认证、使用 `cookies()` 鉴权 |
| hydration mismatch | Server 和 Client 渲染结果不一致 | 避免使用 `Date.now()`、`Math.random()`、`window` 等环境依赖 |
| Context 无法跨 Server/Client | Server Component 不支持 `useContext` | 在 Client Component 内使用 Context，Server Component 通过 props 传递 |
| `'use client'` 边界过大 | 在父组件声明 `'use client'` 导致整棵子树变客户端 | 将 `'use client'` 下推到最小交互组件 |
| 大量数据通过 props 传递 | Server → Client 序列化大数据影响性能 | 使用 `searchParams` 或将数据获取下沉到 Client Component |

**第三方组件默认是 Client Component 的典型场景：**

```tsx
// ❌ 直接在 Server Component 中使用第三方 UI 组件会报错
import { DatePicker } from 'antd'; // 没有 'use client' 声明

export default async function Page() {
    return <DatePicker />; // 报错：DatePicker 使用了 useState
}

// ✅ 方案1：包装为 Client Component
// components/ClientDatePicker.tsx
'use client';
import { DatePicker } from 'antd';
export { DatePicker };

// app/page.tsx
import { DatePicker } from './ClientDatePicker';
export default async function Page() {
    return <DatePicker />;
}

// ✅ 方案2：在组件顶部声明 'use client'（整棵子树变为客户端）
'use client';
import { DatePicker } from 'antd';
// ... 但这会让整个页面变成 Client Component，不推荐
```

**Server Action 安全最佳实践：**

```tsx
'use server';

import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';
import { z } from 'zod';

// 1. 输入验证
const createPostSchema = z.object({
    title: z.string().min(1).max(200),
    content: z.string().min(1).max(50000),
});

export async function createPost(formData: FormData) {
    // 2. 认证检查 — Server Action 可被直接调用
    const cookieStore = await cookies();
    const session = cookieStore.get('session');
    if (!session) {
        throw new Error('未登录');
    }
    const user = await verifySession(session.value);
    if (!user) {
        throw new Error('会话无效');
    }

    // 3. 验证输入
    const raw = {
        title: formData.get('title'),
        content: formData.get('content'),
    };
    const result = createPostSchema.safeParse(raw);
    if (!result.success) {
        return { error: result.error.flatten().fieldErrors };
    }

    // 4. 执行操作
    await db.post.create({
        data: { ...result.data, authorId: user.id },
    });

    revalidatePath('/posts');
    redirect('/posts');
}
```

## 面试题

**Q1: React Server Components 和传统 SSR 有什么区别？**

> 传统 SSR 是在服务端渲染 HTML，但仍然需要将所有组件的 JavaScript 发送到客户端进行 Hydration——组件代码、状态管理、事件处理全部下载。RSC 的 Server Components 在服务端执行后，仅将渲染结果（序列化的虚拟 DOM 描述）发送到客户端，组件代码本身不发送。这意味着一个使用 markdown 渲染库（30KB）的 Server Component，客户端零 JS 增量；而 SSR 中同样的组件，30KB 库必须发送到客户端。另外，RSC 支持 Streaming（配合 Suspense 渐进式渲染），传统 SSR 需要等待所有数据就绪后才能返回完整 HTML。

---

**Q2: Server Component 和 Client Component 的边界规则是什么？如何从 Client Component 调用服务端逻辑？**

> 边界规则：(1) Server Component 可以导入和渲染 Client Component，通过 props 传递数据（必须是可序列化的值）；(2) Client Component 不能导入 Server Component——如果需要服务端渲染的内容，应在 Server Component 中渲染后作为 children 传递；(3) Client Component 调用服务端逻辑通过 Server Actions——用 `'use server'` 标记的异步函数，客户端通过 form action 或 `useActionState` 调用。关键原则：数据从 Server 向 Client 流动（通过 props），操作从 Client 向 Server 流动（通过 Server Actions）。

---

**Q3: Server Actions 的原理是什么？它和传统 API Route 有什么区别？**

> Server Actions 是 React 的 RPC 机制——用 `'use server'` 标记的函数会被编译器转换为服务端端点，客户端调用时 React 自动发送网络请求到服务端执行。与传统 API Route 的区别：(1) **类型安全**——Server Action 函数签名即接口契约，无需手写类型定义；(2) **渐进增强**——配合 `<form action>` 使用时，即使 JS 未加载也能提交；(3) **集成缓存**——Server Action 内可直接调用 `revalidatePath`/`revalidateTag` 刷新数据，无需手动管理缓存失效；(4) **无 API 样板**——不需要定义路由、解析请求、返回 Response 对象。但 Server Action 也有局限：它本质上是一个 POST 请求，不适合 GET 语义；无法自定义 HTTP 头；不适合非 React 客户端调用。

---

**Q4: 为什么 React Server Components 能减少客户端 JS 体积？原理是什么？**

> 核心原理是**组件代码不发送到客户端**。传统 React 应用中，所有组件及其依赖（无论是否在首屏使用）都需要被下载和执行。RSC 中，Server Component 在服务端执行完毕后，输出的是序列化的渲染结果（虚拟 DOM 描述），而非组件源代码。例如一个 Server Component 导入了 `moment.js`（70KB）做日期格式化，客户端完全不需要下载 `moment.js`——服务端执行后只发送格式化后的字符串。React Flight 协议中，Server Component 的渲染结果是一组指令（创建元素、设置属性等），Client Component 则只发送模块引用 ID（客户端已有对应 chunk），这样实现了零客户端 JS 的服务端渲染。

---

**Q5: RSC 的数据获取模式和传统 React 数据获取有什么区别？**

> 传统 React 数据获取：(1) 在 `useEffect` 中发起请求——组件先渲染空状态，effect 执行后触发二次渲染；(2) 客户端瀑布式请求——父组件获取数据后渲染子组件，子组件再次请求数据，造成串行延迟；(3) 需要额外库（React Query / SWR）处理缓存、重试、乐观更新。RSC 数据获取：(1) Server Component 直接 `async/await`——组件本身就是异步的，无需 `useEffect`；(2) 并行获取——多个 Server Component 的数据获取自动并行执行；(3) Streaming——配合 Suspense，慢数据不阻塞快数据的渲染；(4) 服务端直连数据库——省去 API 中间层，减少网络开销。代价是失去了客户端的实时更新能力，需要配合 revalidation 策略。

---

**Q6: Next.js App Router 的缓存和 revalidation 策略有哪些？**

> 四层缓存策略：(1) **请求记忆化（Request Memoization）**——同一渲染周期内相同的 `fetch` 请求只执行一次，React 自动去重；(2) **数据缓存（Data Cache）**——`fetch` 结果缓存到服务端文件系统，`cache: 'force-cache'` 永久缓存，`cache: 'no-store'` 不缓存；(3) **全路由缓存（Full Route Cache）**——渲染结果和 RSC Payload 在构建时缓存，静态路由缓存到 `.next` 目录；(4) **路由缓存（Router Cache）**——客户端内存缓存，存储访问过的路由段，5 分钟（动态）或 30 分钟（静态）。Revalidation 方式：(1) **基于时间**——`revalidate: 60` 定时重新验证（ISR）；(2) **按需**——`revalidatePath('/posts')` 按路径刷新，`revalidateTag('posts')` 按标签刷新；(3) **退出缓存**——`cookies()` / `headers()` 调用自动退出路由缓存。

---

**Q7: `'use client'` 的作用范围是什么？为什么说它是"边界"而非"声明"？**

> `'use client'` 标记的不是单个组件，而是**模块边界**。当文件顶部声明 `'use client'` 后：(1) 该文件及其所有传递依赖都进入客户端 bundle；(2) 该文件导出的组件对父级 Server Component 来说是 Client Component；(3) 该文件导入的其他模块如果是 Server Component，会报错——Client Component 不能导入 Server Component。说它是"边界"是因为它划定了服务端和客户端的分界线：边界之上（导入方）在服务端执行，边界之下（被导入方）在客户端执行。最佳实践是将 `'use client'` 尽量下推到最小粒度的交互组件，避免将整棵子树推入客户端。例如不要在 `layout.tsx` 中声明 `'use client'`，而是在具体的交互组件（搜索框、按钮、表单）中声明。

---

**Q8: 从 Pages Router 迁移到 App Router 的关键要点有哪些？**

> 关键迁移要点：(1) **并行共存**——`app/` 和 `pages/` 目录可以共存，Next.js 优先使用 `app/` 中的路由，可以逐页迁移；(2) **数据获取范式变更**——`getServerSideProps` → Server Component 直接 `async/await`，`getStaticProps` → `generateStaticParams` + `revalidate`，`getInitialProps` → Server Component 或 Client Component 的 `useEffect`；(3) **布局系统**——`_app.tsx` + `_document.tsx` → `app/layout.tsx`，嵌套布局通过 `layout.tsx` 文件层级实现；(4) **路由钩子**——`useRouter` 从 `next/router` 改为 `next/navigation`，API 有变化（`router.push` 保留，`router.pathname` → `usePathname()`）；(5) **元数据**——`<Head>` → `metadata` 导出或 `generateMetadata` 函数；(6) **API 路由**——`pages/api/*.ts` → `app/api/*/route.ts`，导出 `GET`/`POST` 等命名函数；(7) **认证中间件**——`middleware.ts` 语法基本不变，但需要注意 App Router 的路由匹配模式。建议渐进式迁移：先迁移静态页面，再迁移动态页面，最后迁移 API 路由。

---

**相关链接：**
- [[React核心]]
- [[React Hooks详解]]
- [[Next.js与SSR]]

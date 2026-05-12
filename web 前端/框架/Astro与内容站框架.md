---
tags:
  - Web前端
  - 框架
  - Astro
  - SSG
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Astro与内容站框架

## What — 是什么

> Astro 是内容驱动的静态站点框架，以零 JS 默认输出、岛屿架构（Islands Architecture）和多框架混用为核心特色，专为博客、文档、营销站等内容型网站设计。

**核心概念：**

- **零 JS 默认**：组件在构建时渲染为纯 HTML，默认不发送任何 JavaScript 到客户端
- **岛屿架构**：页面大部分是静态 HTML，仅在需要交互的局部区域"注水"为交互式组件（岛屿）
- **多框架混用**：同一页面可以同时使用 React、Vue、Svelte 等组件，框架之间互不冲突
- **内容集合（Content Collections）**：类型安全的内容管理方案，用 Zod Schema 定义和验证 Markdown/MDX 内容
- **View Transitions**：内置页面过渡动画，无需第三方库即可实现流畅的页面切换效果

**核心架构：**

- 设计理念：内容优先，交互按需加载——能不发 JS 就不发
- 渲染流程：构建时将所有组件渲染为 HTML → 识别带 `client:*` 指令的岛屿 → 仅为岛屿发送对应框架运行时
- 数据流：Markdown/MDX 内容 → Content Collections 验证 → 页面组件消费 → 构建输出纯静态 HTML + 交互岛屿 JS

**关键特性：**

- `.astro` 组件语法：frontmatter script + 模板 + scoped 样式
- 文件路由：`src/pages/` 下文件自动映射为路由
- SSR 模式：通过 adapter 支持 Node / Netlify / Vercel / Cloudflare 等运行时
- 集成生态：Tailwind、MDX、Sitemap、Partytown 等官方集成

## Why — 为什么

**适用场景：**

- 博客、文档站、个人主页——内容为主、交互极少
- 营销落地页、产品展示页——SEO 要求极高
- 技术文档网站——需要多框架组件演示（如 Vue/React/Svelte 示例并排展示）
- 内容驱动的电商展示页——列表页 SSG、详情页按需交互

**核心优势：内容站不需要大量 JS**

传统 SPA 框架（React/Vue）构建的内容站，即使页面只是纯文本展示，客户端也要加载框架运行时（React ~40KB、Vue ~33KB gzip）。而内容站的 90% 区域是静态文本，仅有搜索框、主题切换等少量交互。Astro 的零 JS 默认策略精确解决了这一矛盾——静态区域零 JS，交互区域按需注水。

**传统 SSG 框架的问题：**

Next.js SSG 模式虽然构建时生成 HTML，但仍然发送 React 运行时 + 页面 JS 到客户端用于 Hydration。一个纯文本博客页面可能发送 50-100KB 的 JS，其中绝大部分是框架本身的 Hydration 开销。Astro 默认不 Hydration——没有 `client:*` 指令的组件，构建后就是纯 HTML，零 JS。

**SEO 极佳：**

Astro 默认输出纯 HTML，没有客户端渲染的空白期，搜索引擎可以直接索引完整内容。配合 Content Collections 的结构化数据和 Sitemap 集成，SEO 表现优于 CSR/SSR 方案。

**对比同类框架：**

| 维度 | Astro | Next.js SSG | Nuxt SSG | Gatsby |
|------|-------|-------------|----------|--------|
| 默认 JS 体积 | 0 KB | ~80KB（React 运行时 + Hydration） | ~60KB（Vue 运行时 + Hydration） | ~50KB（React + Gatsby 运行时） |
| 多框架混用 | 支持（React/Vue/Svelte/Preact/Solid/Lit） | 不支持（仅 React） | 不支持（仅 Vue） | 不支持（仅 React） |
| 内容集合 | 内置 Content Collections（Zod Schema） | 无内置（需手动或第三方） | Content Query（内置） | GraphQL 数据层 |
| 学习曲线 | 低（HTML/CSS 基础即可） | 中（React + App Router） | 低-中（Vue 基础） | 中（React + GraphQL） |
| 适用场景 | 内容站、文档、博客、营销页 | 全栈应用、电商、Dashboard | 全栈应用、Vue 生态内容站 | 数据源复杂的内容站 |
| 页面过渡 | 内置 View Transitions | 无内置 | 无内置 | 无内置 |
| 构建速度 | 快（Vite 驱动） | 中 | 快（Vite 驱动） | 慢（Webpack + GraphQL） |

**优缺点：**

- ✅ 优点：
  - 默认零 JS，内容站性能天花板
  - 岛屿架构精确控制交互边界
  - 多框架混用，不锁定技术栈
  - Content Collections 类型安全的内容管理
  - 学习曲线极低，.astro 语法接近 HTML
  - Vite 驱动，开发和构建速度极快
- ❌ 缺点：
  - 不适合高交互应用（Dashboard、SaaS 后台）
  - SSR 模式生态不如 Next.js/Nuxt 成熟
  - 岛屿间状态共享需要额外方案
  - 社区和插件生态比 Next.js 小

## How — 怎么用

### 1. 项目结构

```bash
npm create astro@latest my-site
cd my-site
npm run dev
```

```
my-site/
├── astro.config.mjs       # 框架配置（集成、适配器、站点信息）
├── package.json
├── tsconfig.json
├── public/                 # 静态资源（直接复制，不处理）
│   ├── favicon.svg
│   └── images/
└── src/
    ├── pages/              # 文件路由（必需）
    │   ├── index.astro     # 首页 → /
    │   ├── about.astro     # /about
    │   └── blog/
    │       ├── index.astro # /blog
    │       └── [slug].astro # /blog/:slug 动态路由
    ├── components/          # 组件
    │   ├── Header.astro
    │   ├── Counter.jsx      # React 组件
    │   └── Search.vue       # Vue 组件
    ├── layouts/             # 布局组件
    │   └── BaseLayout.astro
    └── content/             # 内容集合
        ├── config.ts        # Zod Schema 定义
        └── blog/            # Markdown/MDX 文件
            ├── first-post.md
            └── second-post.mdx
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import tailwind from '@astrojs/tailwind';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://example.com',     // 站点 URL（sitemap 需要）
  integrations: [
    react(),                        // React 集成
    vue(),                          // Vue 集成
    tailwind(),                     // Tailwind CSS
    mdx(),                          // MDX 支持
    sitemap(),                      // 自动生成 sitemap.xml
  ],
  output: 'static',                 // 'static' | 'server' | 'hybrid'
});
```

### 2. .astro 组件语法

`.astro` 组件由三部分组成：frontmatter script（`---`）、HTML 模板、`<style>` 样式。

```astro
---
// src/components/Card.astro
// frontmatter：服务端执行的 JavaScript
// 这里的代码在构建时运行，不会发送到客户端

interface Props {
  title: string;
  description?: string;
  tags?: string[];
}

const { title, description = '', tags = [] } = Astro.props;

const formattedDate = new Date().toLocaleDateString('zh-CN');
---

<!-- HTML 模板：支持 JSX-like 表达式 -->
<article class="card">
  <h2>{title}</h2>
  {description && <p>{description}</p>}
  {tags.length > 0 && (
    <ul class="tags">
      {tags.map(tag => <li>{tag}</li>)}
    </ul>
  )}
  <time>{formattedDate}</time>
</article>

<!-- 样式默认 scoped，不会影响其他组件 -->
<style>
  .card {
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 1.5rem;
  }
  .tags {
    display: flex;
    gap: 0.5rem;
    list-style: none;
  }
</style>
```

**Props 与 Slots：**

```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  title: string;
  description?: string;
}

const { title, description = '默认描述' } = Astro.props;
---

<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width" />
  <title>{title}</title>
  <meta name="description" content={description} />
</head>
<body>
  <header>
    <nav>
      <a href="/">首页</a>
      <a href="/blog">博客</a>
      <a href="/about">关于</a>
    </nav>
  </header>

  <main>
    <!-- 默认 slot：页面内容插入此处 -->
    <slot />

    <!-- 具名 slot -->
    <footer>
      <slot name="footer" />
    </footer>
  </main>
</body>
</html>
```

```astro
---
// src/pages/about.astro
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="关于我" description="个人介绍">
  <!-- 默认 slot 内容 -->
  <h1>关于我</h1>
  <p>一名前端开发者</p>

  <!-- 具名 slot 内容 -->
  <p slot="footer">Copyright 2026</p>
</BaseLayout>
```

### 3. 岛屿架构（Islands Architecture）

岛屿架构的核心思想：页面大部分是静态 HTML（不需要 JS），仅在需要交互的局部区域"注水"为交互式组件。这些交互区域就像海洋（静态 HTML）中的岛屿（交互组件）。

**client:* 指令：**

| 指令 | 加载时机 | 适用场景 |
|------|----------|----------|
| `client:load` | 页面加载立即注水 | 关键交互（导航、搜索框） |
| `client:idle` | 浏览器空闲时注水 | 非关键交互（评论框、分享按钮） |
| `client:visible` | 进入视口时注水 | 下方内容（聊天组件、图表） |
| `client:only` | 仅客户端渲染，跳过 SSR | 依赖浏览器 API 的组件 |
| 无指令 | 不注水，纯静态 HTML | 展示型组件（默认行为） |

```astro
---
// src/pages/index.astro
import SearchBox from '../components/SearchBox.jsx';     // React 组件
import ThemeToggle from '../components/ThemeToggle.vue';  // Vue 组件
import Analytics from '../components/Analytics.svelte';   // Svelte 组件
import Hero from '../components/Hero.astro';              // Astro 静态组件
---

<!-- 静态区域：零 JS，纯 HTML -->
<Hero />

<!-- 岛屿 1：立即加载（搜索框是核心交互） -->
<SearchBox client:load />

<!-- 岛屿 2：空闲时加载（主题切换不紧急） -->
<ThemeToggle client:idle />

<!-- 岛屿 3：进入视口时加载（分析组件在页面底部） -->
<Analytics client:visible />

<!-- 岛屿 4：仅客户端渲染（使用 window / canvas） -->
<CanvasChart client:only="react" />
```

**构建产物差异：**

```html
<!-- 无 client:* 指令 → 纯 HTML，零 JS -->
<section class="hero">
  <h1>Welcome</h1>
  <p>纯静态内容</p>
</section>

<!-- client:load → HTML + 框架运行时 + 组件 JS -->
<astro-island
  component-url="/SearchBox.js"
  renderer-url="/react-runtime.js"
  props='{"placeholder":"搜索..."}'
></astro-island>
```

### 4. 多框架集成

Astro 的多框架混用不是噱头，而是解决实际问题的能力：文档站同时展示 React 和 Vue 组件示例、团队逐步迁移技术栈、复用已有组件库。

**安装框架集成：**

```bash
npm install @astrojs/react @astrojs/vue @astrojs/svelte @astrojs/preact
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import svelte from '@astrojs/svelte';

export default defineConfig({
  integrations: [react(), vue(), svelte()],
});
```

**同一页面混用多框架：**

```astro
---
// src/pages/demo.astro
import ReactCounter from '../components/ReactCounter.jsx';
import VueCounter from '../components/VueCounter.vue';
import SvelteCounter from '../components/SvelteCounter.svelte';
---

<h1>多框架混用演示</h1>

<div class="grid">
  <!-- React 组件 -->
  <div class="card">
    <h2>React Counter</h2>
    <ReactCounter client:load />
  </div>

  <!-- Vue 组件 -->
  <div class="card">
    <h2>Vue Counter</h2>
    <VueCounter client:load />
  </div>

  <!-- Svelte 组件 -->
  <div class="card">
    <h2>Svelte Counter</h2>
    <SvelteCounter client:load />
  </div>
</div>
```

```jsx
// src/components/ReactCounter.jsx
import { useState } from 'react';

export default function ReactCounter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      React: {count}
    </button>
  );
}
```

```vue
<!-- src/components/VueCounter.vue -->
<template>
  <button @click="count++">Vue: {{ count }}</button>
</template>

<script setup>
import { ref } from 'vue';
const count = ref(0);
</script>
```

**注意事项：**

- 每个框架岛屿独立运行，有自己的状态和运行时
- 不同框架的岛屿之间不共享状态（需通过 `nanostores` 等方案通信）
- `client:only` 必须指定框架名：`client:only="react"` / `client:only="vue"`

### 5. 内容集合 Content Collections

Content Collections 是 Astro 内置的类型安全内容管理方案，用 Zod Schema 定义内容结构，自动验证和提供 TypeScript 类型。

**定义 Schema：**

```ts
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

// 博客集合
const blog = defineCollection({
  type: 'content',    // 'content' = Markdown/MDX | 'data' = JSON/YAML
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    heroImage: z.string().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
    author: z.string().default('匿名'),
  }),
});

// 产品集合
const products = defineCollection({
  type: 'content',
  schema: z.object({
    name: z.string(),
    price: z.number().positive(),
    category: z.enum(['electronics', 'clothing', 'food']),
    features: z.array(z.string()),
    inStock: z.boolean().default(true),
  }),
});

// 导出集合配置
export const collections = { blog, products };
```

**Markdown 内容文件：**

```markdown
---
# src/content/blog/astro-guide.md
title: "Astro 完全指南"
description: "从零开始学习 Astro 框架"
pubDate: 2026-05-12
tags: ["Astro", "SSG", "前端"]
author: "xqz"
---

# Astro 完全指南

Astro 是内容驱动的静态站点框架...
```

**查询与使用内容：**

```astro
---
// src/pages/blog/index.astro
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

// 获取所有博客文章（自动类型推断）
const allPosts = await getCollection('blog', ({ data }) => {
  // 过滤草稿
  return !data.draft;
});

// 按日期降序排列
const posts = allPosts.sort(
  (a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
);
---

<BaseLayout title="博客">
  <h1>博客文章</h1>
  <ul>
    {posts.map(post => (
      <li>
        <time>{post.data.pubDate.toLocaleDateString('zh-CN')}</time>
        <a href={`/blog/${post.id}`}>{post.data.title}</a>
        <span>{post.data.tags.join(', ')}</span>
      </li>
    ))}
  </ul>
</BaseLayout>
```

```astro
---
// src/pages/blog/[slug].astro
import { getCollection, getEntry } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

// getStaticPaths 定义动态路由参数
export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;

// 渲染 Markdown 内容
const { Content } = await post.render();
---

<BaseLayout title={post.data.title} description={post.data.description}>
  <article>
    <h1>{post.data.title}</h1>
    <time>{post.data.pubDate.toLocaleDateString('zh-CN')}</time>
    <p>{post.data.description}</p>

    <!-- 渲染 Markdown 正文 -->
    <Content />
  </article>
</BaseLayout>
```

**内容查询与过滤进阶：**

```astro
---
import { getCollection, getEntry } from 'astro:content';

// 按 tag 过滤
const astroPosts = await getCollection('blog', ({ data }) =>
  data.tags.includes('Astro')
);

// 获取单条内容
const firstPost = await getEntry('blog', 'astro-guide');

// 组合查询：最新 5 篇非草稿文章
const recentPosts = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf())
  .slice(0, 5);
---
```

### 6. 路由系统

**文件路由：**

```
src/pages/
├── index.astro           → /
├── about.astro           → /about
├── blog/
│   ├── index.astro       → /blog
│   └── [slug].astro      → /blog/:slug（动态路由）
├── users/
│   └── [id].astro        → /users/:id
└── docs/
    └── [...slug].astro   → /docs/*（REST 参数/捕获所有路由）
```

**动态路由 — getStaticPaths：**

```astro
---
// src/pages/blog/[slug].astro

// 静态模式下，动态路由必须通过 getStaticPaths 声明所有路径
export async function getStaticPaths() {
  const posts = await getCollection('blog');

  return posts.map(post => ({
    params: { slug: post.id },           // 路由参数
    props: { post },                      // 传递给页面的 props
  }));
}

const { post } = Astro.props;
const { slug } = Astro.params;
---

<h1>{post.data.title}</h1>
<p>Slug: {slug}</p>
```

**分页：**

```astro
---
// src/pages/blog/[page].astro
export async function getStaticPaths({ paginate }) {
  const posts = await getCollection('blog');
  const sorted = posts.sort(
    (a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
  );

  return paginate(sorted, { pageSize: 10 });
}

const { page } = Astro.props;
---

<h1>博客 — 第 {page.currentPage} 页</h1>

<ul>
  {page.data.map(post => (
    <li>
      <a href={`/blog/${post.id}`}>{post.data.title}</a>
    </li>
  ))}
</ul>

<!-- 分页导航 -->
<nav class="pagination">
  {page.url.prev && <a href={page.url.prev}>上一页</a>}
  <span>第 {page.currentPage} / {page.lastPage} 页</span>
  {page.url.next && <a href={page.url.next}>下一页</a>}
</nav>
```

**REST 参数（捕获所有路由）：**

```astro
---
// src/pages/docs/[...slug].astro
// 匹配 /docs、/docs/getting-started、/docs/api/reference 等

export async function getStaticPaths() {
  return [
    { params: { slug: undefined } },                         // /docs
    { params: { slug: 'getting-started' } },                 // /docs/getting-started
    { params: { slug: 'api/reference' } },                   // /docs/api/reference
  ];
}

const { slug } = Astro.params;
const path = slug ? slug.join('/') : 'index';
---

<h1>文档：{path}</h1>
```

### 7. SSR 模式

Astro 默认是 SSG（静态生成），但可以通过 adapter 切换为 SSR 模式，支持服务端渲染和 API 路由。

```bash
# 安装适配器
npm install @astrojs/node       # Node.js
npm install @astrojs/netlify    # Netlify
npm install @astrojs/vercel     # Vercel
npm install @astrojs/cloudflare # Cloudflare Pages
```

```js
// astro.config.mjs — Node.js SSR
import { defineConfig } from 'astro/config';
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',    // 全部 SSR
  adapter: node({
    mode: 'standalone', // 'standalone' | 'middleware'
  }),
});
```

```js
// astro.config.mjs — Hybrid 模式（大部分静态，部分 SSR）
import { defineConfig } from 'astro/config';
import netlify from '@astrojs/netlify';

export default defineConfig({
  output: 'hybrid',     // 默认静态，页面可声明 prerender = false 切为 SSR
  adapter: netlify(),
});
```

```astro
---
// src/pages/api/hello.ts — SSR 模式下的 API 路由
export const prerender = false; // hybrid 模式下标记为 SSR

export async function GET({ request, url }) {
  const name = url.searchParams.get('name') || 'World';

  return new Response(
    JSON.stringify({ message: `Hello, ${name}!` }),
    {
      status: 200,
      headers: { 'Content-Type': 'application/json' },
    }
  );
}

export async function POST({ request }) {
  const body = await request.json();

  return new Response(
    JSON.stringify({ received: body }),
    {
      status: 201,
      headers: { 'Content-Type': 'application/json' },
    }
  );
}
---
```

### 8. View Transitions

Astro 内置 View Transitions API 支持，实现页面间流畅过渡动画，无需任何第三方库。

```astro
---
// src/layouts/BaseLayout.astro
---

<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <!-- 启用 View Transitions -->
  <ViewTransitions />
</head>
<body>
  <header>
    <nav>
      <a href="/">首页</a>
      <a href="/blog">博客</a>
    </nav>
  </header>
  <main>
    <slot />
  </main>
</body>
</html>
```

**transition:persist — 保持状态：**

```astro
---
// 跨页面保持同一元素（如视频播放器、音频播放器）
---

<!-- 页面 A -->
<video transition:persist="hero-video" src="/intro.mp4" autoplay />

<!-- 页面 B：同名元素不会重新加载，保持播放状态 -->
<video transition:persist="hero-video" src="/intro.mp4" autoplay />
```

**transition:animate — 自定义动画：**

```astro
---
import { fade, slide, morph } from 'astro:transitions';
---

<!-- 内置动画 -->
<h1 transition:animate={fade}>淡入标题</h1>
<div transition:animate={slide}>滑入内容</div>

<!-- 自定义动画 -->
<h1 transition:animate={{
  old: {
    opacity: '0',
    transform: 'translateY(-20px)',
  },
  new: {
    opacity: '1',
    transform: 'translateY(0)',
  },
  duration: '0.3s',
  easing: 'ease-out',
}}>自定义动画</h1>
```

**控制过渡行为：**

```astro
---
// data-astro-reload：指定链接强制完整页面刷新（不使用 View Transition）
---

<a href="/admin" data-astro-reload>后台管理</a>

<!-- transition:persist 仅在导航时保持 -->
<div transition:persist="sidebar" transition:animate={morph}>
  <Sidebar />
</div>
```

### 9. 集成生态

**Tailwind CSS：**

```bash
npx astro add tailwind
```

```js
// astro.config.mjs
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()],
});
```

**MDX：**

```bash
npx astro add mdx
```

```astro
---
// MDX 文件中可以直接使用 JSX 组件
import Chart from '../components/Chart.jsx';
---

## 性能对比

<Chart data={performanceData} />

交互式图表嵌入 Markdown 中。
```

**Sitemap 与 SEO：**

```bash
npx astro add sitemap
```

```js
// astro.config.mjs
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://example.com',  // sitemap 必须配置 site
  integrations: [sitemap({
    filter: (page) => !page.includes('/admin'),  // 过滤页面
    changefreq: 'weekly',
    priority: 0.7,
  })],
});
```

**其他常用集成：**

```bash
# Partytown — 在 Web Worker 中运行第三方脚本
npx astro add partytown

# Turbopack — 更快的构建（实验性）
npx astro add turbopack

# Node.js adapter — SSR 部署
npx astro add node

# Image 优化（@astrojs/image 已内置到 Astro 3+）
# Astro 3+ 内置 <Image /> 组件
```

### 10. 部署

**静态部署（output: 'static'）：**

```bash
# 构建
npm run build

# 产物在 dist/ 目录，纯静态文件，可部署到任何静态托管
```

| 平台 | 部署方式 |
|------|----------|
| Vercel | `npx astro add vercel` 或直接连接 Git 仓库 |
| Netlify | `npx astro add netlify` 或拖拽 dist/ 目录 |
| Cloudflare Pages | 连接 Git 仓库，构建命令 `npm run build`，输出目录 `dist` |
| GitHub Pages | `npm run build` + GitHub Actions 部署 dist/ |
| 自有服务器 | `npm run build` + Nginx 托管 dist/ |

**SSR 部署（output: 'server'）：**

```bash
# Vercel SSR
npm install @astrojs/vercel
```

```js
// astro.config.mjs
import vercel from '@astrojs/vercel';

export default defineConfig({
  output: 'server',
  adapter: vercel(),
});
```

```bash
# Node.js 自托管 SSR
npm install @astrojs/node
```

```js
// astro.config.mjs
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({ mode: 'standalone' }),
});
```

```bash
# 构建后运行
npm run build
node dist/server/entry.mjs
```

### 11. 性能优化

**图片优化 — Image 组件：**

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.jpg';  // src/assets 下的图片会被优化
---

<!-- 自动优化：格式转换(WebP/AVIF)、尺寸调整、懒加载 -->
<Image
  src={heroImage}
  alt="Hero image"
  width={800}
  height={400}
  loading="lazy"              // 懒加载
  decoding="async"            // 异步解码
  densities={[1, 2]}          // 生成 1x 和 2x 图片
/>

<!-- 远程图片优化 -->
<Image
  src="https://example.com/photo.jpg"
  alt="Remote image"
  width={600}
  height={400}
  inferSize                    // 自动推断尺寸
/>
```

**字体优化：**

```astro
---
// Astro 4+ 内置字体优化
---

<head>
  <!-- 使用 @fontsource 或 Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap"
    rel="stylesheet"
  />

  <!-- 或使用本地字体 + CSS @font-face -->
  <style>
    @font-face {
      font-family: 'CustomFont';
      src: url('/fonts/custom.woff2') format('woff2');
      font-display: swap;           /* 避免 FOIT */
      unicode-range: U+0020-007E;   /* 仅加载常用字符 */
    }
  </style>
</head>
```

**脚本加载策略：**

```astro
---
// Astro 的 <script> 默认打包、去重、模块化
---

<!-- 默认行为：打包为模块，自动去重 -->
<script>
  console.log('这段 JS 会被打包，同一组件多个实例只执行一次');
</script>

<!-- is:inline：不打包，原样插入 HTML，每个实例都执行 -->
<script is:inline>
  console.log('原样插入，不打包不去重');
</script>

<!-- 按需加载脚本 -->
<script define:vars={{ apiKey: 'xxx' }}>
  console.log(apiKey); // 变量从 frontmatter 注入
</script>

<!-- 第三方分析脚本 → Partytown（Web Worker 中运行，不阻塞主线程） -->
<script
  type="text/partytown"
  src="https://analytics.example.com/script.js"
/>
```

**其他优化：**

```js
// astro.config.mjs
export default defineConfig({
  build: {
    inlineStylesheets: 'auto',  // 小 CSS 内联，大 CSS 外链
  },
  compressHTML: true,            // 压缩 HTML 输出
  vite: {
    build: {
      cssMinify: true,           // CSS 压缩
      rollupOptions: {
        output: {
          manualChunks: undefined, // 自动代码拆分
        },
      },
    },
  },
});
```

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 岛屿间状态如何共享 | 每个岛屿是独立运行时，不共享状态 | 使用 `nanostores`（Astro 官方推荐）或 CustomEvent 通信 |
| SSR 模式下 `Astro.request` 不可用 | 静态模式下没有请求对象 | 切换 `output: 'server'` 或 `output: 'hybrid'` 并配置 adapter |
| 从 Next.js 迁移成本高 | 路由、数据获取、组件模型差异大 | 先迁移纯展示页面，交互组件用 `@astrojs/react` 保留，逐步替换 |
| MDX 和 Content Collections 如何选择 | 两者解决不同问题 | 内容管理用 Content Collections（类型安全 + 查询），MDX 用于需要嵌入交互组件的文章 |
| 动态路由构建报错 | 静态模式下必须提供 `getStaticPaths` | 添加 `getStaticPaths` 或切换 SSR 模式 |
| 图片路径在构建后 404 | 引用 `public/` 下的图片未使用绝对路径 | 使用 `/images/photo.jpg` 绝对路径，或用 `import` 引入 `src/assets/` 图片 |

**岛屿间状态共享详解：**

```bash
npm install nanostores @nanostores/react @nanostores/vue
```

```ts
// src/stores/cart.ts
import { atom } from 'nanostores';

export const cartCount = atom(0);

export function addToCart() {
  cartCount.set(cartCount.get() + 1);
}
```

```jsx
// src/components/BuyButton.jsx — React 岛屿
import { useStore } from '@nanostores/react';
import { cartCount, addToCart } from '../stores/cart';

export default function BuyButton() {
  const count = useStore(cartCount);
  return <button onClick={addToCart}>加入购物车 ({count})</button>;
}
```

```vue
<!-- src/components/CartBadge.vue — Vue 岛屿 -->
<template>
  <span>购物车: {{ count }}</span>
</template>

<script setup>
import { useStore } from '@nanostores/vue';
import { cartCount } from '../stores/cart';

const count = useStore(cartCount);
</script>
```

---

## 面试题

**Q1: 岛屿架构（Islands Architecture）的原理是什么？与传统 SSR Hydration 有何不同？**

> 岛屿架构将页面分为静态区域和交互区域。静态区域在构建时渲染为纯 HTML，不发送任何 JS；交互区域（岛屿）通过 `client:*` 指令标记，构建时单独打包对应框架运行时和组件 JS，按需加载和注水。传统 SSR Hydration 是"全量注水"——整个页面的 HTML 都被框架 Hydration，即使大部分区域没有交互也要加载框架运行时和执行 Hydration 逻辑。岛屿架构是"部分注水"——只有岛屿注水，静态区域零 JS 开销。这直接减少了客户端 JS 体积和 Hydration 时间。

**Q2: client:load、client:idle、client:visible、client:only 四个指令有什么区别？**

> `client:load` 页面加载立即注水，适用于首屏关键交互（导航栏、搜索框）；`client:idle` 使用 `requestIdleCallback` 在浏览器空闲时注水，适用于非紧急交互（评论框、分享按钮）；`client:visible` 使用 `IntersectionObserver` 在组件进入视口时注水，适用于页面下方的交互组件（图表、聊天）；`client:only` 跳过服务端渲染，仅在客户端渲染组件，适用于依赖 `window`/`document` 等浏览器 API 的组件，必须指定框架名（如 `client:only="react"`）。选择原则：越早加载的组件 JS 开销越大，应按交互优先级选择合适的指令。

**Q3: Astro 为什么能做到默认零 JS？**

> Astro 的 `.astro` 组件在构建时（build time）执行所有 frontmatter 代码和组件渲染，输出纯 HTML。没有 `client:*` 指令的组件不会被 Hydration，因此不发送任何 JS 到客户端。传统框架（React/Vue）的组件即使只做展示，也需要框架运行时和 Hydration 逻辑来绑定事件和状态——即使没有交互也要"准备好"交互能力。Astro 的设计哲学是"默认静态，按需交互"：构建时渲染一切，只有标记了 `client:*` 的组件才在客户端激活。框架运行时（React ~40KB、Vue ~33KB gzip）仅在有岛屿时才发送。

**Q4: Content Collections 的作用是什么？与直接读取 Markdown 文件有什么区别？**

> Content Collections 提供：(1) **类型安全**——通过 Zod Schema 定义内容结构，TypeScript 自动推断类型，写 `post.data.titel` 会编译报错；(2) **验证**——构建时自动验证所有内容文件是否符合 Schema，不符合则构建失败，避免运行时数据错误；(3) **查询 API**——`getCollection` 和 `getEntry` 提供过滤、排序等查询能力，无需手动 glob 和解析；(4) **自动补全**——编辑 Markdown frontmatter 时 IDE 自动补全 Schema 定义的字段。直接读取 Markdown 文件（`import.meta.glob`）没有类型验证、没有 Schema 约束、需要手动解析 frontmatter，容易出错且无法在构建前发现问题。

**Q5: Astro 和 Next.js 的定位有什么差异？**

> Astro 是内容驱动的静态站点框架，定位"内容站"——博客、文档、营销页，核心优势是零 JS 默认输出和岛屿架构。Next.js 是 React 全栈框架，定位"Web 应用"——电商、SaaS、Dashboard，核心优势是 SSR/ISR/API Routes 全栈能力。关键区别：(1) **JS 策略**——Astro 默认零 JS，Next.js 默认全量 Hydration；(2) **多框架**——Astro 支持混用 React/Vue/Svelte，Next.js 仅 React；(3) **交互密度**——Astro 适合低交互，Next.js 适合高交互；(4) **SSR 生态**——Next.js SSR 生态远比 Astro 成熟。选型规则：内容站选 Astro，Web 应用选 Next.js。

**Q6: 多框架混用的原理是什么？为什么不会冲突？**

> 原理是每个框架集成（`@astrojs/react`、`@astrojs/vue` 等）提供独立的 renderer。构建时，Astro 识别组件的框架类型，调用对应 renderer 的 `renderToStaticMarkup` 方法将组件渲染为 HTML 字符串。运行时，带 `client:*` 指令的组件被包裹在 `<astro-island>` 自定义元素中，每个岛屿独立加载对应框架运行时和组件 JS，互不干扰。React 岛屿加载 React 运行时 + React 组件，Vue 岛屿加载 Vue 运行时 + Vue 组件，各自有自己的状态管理和事件系统。框架运行时通过作用域隔离，不会全局冲突。代价是多框架会增加总 JS 体积（同时用 React + Vue = 两个运行时），所以实际项目中通常只用 1-2 个框架。

**Q7: View Transitions 的用法有哪些？transition:persist 的作用是什么？**

> View Transitions 通过在布局中添加 `<ViewTransitions />` 组件启用，页面导航时自动使用浏览器 View Transitions API 产生过渡动画。用法：(1) **默认过渡**——启用后页面切换自动淡入淡出；(2) **`transition:animate`**——自定义过渡动画，可使用内置的 `fade`/`slide`/`morph`，或自定义 CSS 动画属性；(3) **`transition:persist`**——跨页面导航时保持 DOM 元素不变，而不是销毁重建。典型场景：页面顶部音频播放器在导航时不中断播放，侧边栏在页面切换时不重新渲染。`transition:persist="name"` 通过 name 匹配，新旧页面中同名元素会被视为同一元素，执行 morph 动画而不是销毁+创建。

**Q8: Astro 适用于哪些场景？不适合哪些场景？**

> 适合：(1) **博客/文档站**——内容为主、交互极少，零 JS 输出极致性能；(2) **营销落地页**——SEO 要求高、加载速度关键；(3) **技术文档**——需要多框架组件演示（React + Vue + Svelte 示例并排）；(4) **个人主页/作品集**——纯展示、偶尔交互（主题切换、联系表单）。不适合：(1) **高交互 SPA**——Dashboard、在线编辑器、即时通讯，大量交互组件使岛屿架构优势消失，反而增加多运行时开销；(2) **实时数据应用**——股票行情、实时监控，需要 WebSocket + 频繁 DOM 更新，SSG/岛屿模型不匹配；(3) **复杂表单应用**——多步表单、实时验证，状态管理复杂，全框架 Hydration 更高效。判断标准：交互区域占比 > 50% 就不适合 Astro。

---

**相关链接：** [[Next.js与SSR]] [[Nuxt.js与Vue SSR]] [[Webpack与Vite]]

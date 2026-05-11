---
tags:
  - Web前端
  - Nuxt.js
  - Vue
  - SSR
  - 全栈
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Nuxt.js 与 Vue SSR

## What — 什么是 Nuxt.js

Nuxt.js 是基于 Vue 的全栈框架，提供 SSR（服务端渲染）、SSG（静态生成）、混合渲染等能力，并内置路由、数据获取、SEO 优化等。它是 Vue 生态中 Next.js 的对应物。

### 渲染模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| SSR | 服务端实时渲染 HTML | 动态内容、SEO 敏感页面 |
| SSG | 构建时生成静态 HTML | 博客、文档、营销页 |
| SWR | 静态 + 客户端增量更新 | 内容型网站 |
| CSR | 纯客户端渲染 | 后台管理、无需 SEO |
| Hybrid | 每个路由独立选择渲染模式 | 大型项目不同页面不同策略 |

### 与 Next.js 对比

| 维度 | Nuxt.js | Next.js |
|------|---------|---------|
| 基础框架 | Vue | React |
| 路由 | 基于文件自动生成 | 基于文件自动生成 |
| 数据获取 | `useFetch` / `useAsyncData` | `fetch` in Server Components |
| 布局 | `layouts/` 目录 | 嵌套布局 |
| 状态管理 | `useState` 内置 | 需第三方库 |
| SSR 框架 | h3 / Nitro | Node.js |
| 部署 | 30+ 平台自动适配 | Vercel 优先 |

---

## Why — 为什么选择 Nuxt.js

### 1. 零配置的 Vue SSR

手动配置 Vue SSR 需要处理服务端入口、客户端入口、HTML 模板、路由同步、状态脱水/注水等问题。Nuxt.js 一条命令搞定。

### 2. 文件即路由

```
pages/
├── index.vue           → /
├── about.vue           → /about
├── users/
│   ├── index.vue       → /users
│   └── [id].vue        → /users/:id
└── blog/
    └── [...slug].vue   → /blog/*
```

### 3. 全栈能力

Nuxt 的 Server 目录可以直接写 API 路由，前后端一体。

### 4. 混合渲染

每个页面可以独立设置渲染模式：首页 SSR、博客 SSG、后台 CSR。

### 优缺点

- ✅ 优点：零配置 SSR、文件路由、全栈能力、Vue 生态
- ❌ 缺点：框架约定重（灵活性低）、调试 SSR 问题较难、社区比 Next.js 小

---

## How — 怎么用

### 1. 创建项目

```bash
npx nuxi@latest init my-app
cd my-app
npm run dev
```

### 2. 页面与路由

```vue
<!-- pages/index.vue -->
<template>
  <div>
    <h1>Home Page</h1>
    <NuxtLink to="/users">Users</NuxtLink>
  </div>
</template>
```

```vue
<!-- pages/users/[id].vue -->
<template>
  <div>
    <h1>User {{ id }}</h1>
    <p>{{ user?.name }}</p>
    <NuxtLink to="/users">Back</NuxtLink>
  </div>
</template>

<script setup>
const route = useRoute()
const id = route.params.id

const { data: user } = await useFetch(`/api/users/${id}`)
</script>
```

### 3. 数据获取

```vue
<!-- useFetch：SSR + CSR 统一的数据获取 -->
<template>
  <div v-if="pending">Loading...</div>
  <div v-else>
    <ul>
      <li v-for="user in users" :key="user.id">{{ user.name }}</li>
    </ul>
  </div>
</template>

<script setup>
const { data: users, pending, error, refresh } = await useFetch('/api/users', {
  // 请求选项
  method: 'GET',
  headers: { Authorization: `Bearer ${token}` },
  query: { page: 1 },
  // 缓存与去重
  dedupe: 'defer',   // 相同请求只发一次
  // 响应式
  watch: [page],      // page 变化自动重新请求
})
</script>
```

```ts
// useAsyncData：更灵活的异步数据
const { data, pending } = await useAsyncData('key', () => {
  return $fetch('/api/data', { params: { id: 1 } })
})

// useLazyFetch / useLazyAsyncData：不阻塞导航
const { data, pending } = useLazyFetch('/api/users')
// 页面立即显示，数据到达后更新
```

### 4. Server API

```ts
// server/api/users.ts
export default defineEventHandler(async (event) => {
  const query = getQuery(event)
  const page = Number(query.page) || 1
  const limit = 20

  const users = await db.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
  })

  return { users, page }
})
```

```ts
// server/api/users/[id].ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')

  const user = await db.user.findUnique({ where: { id: Number(id) } })

  if (!user) {
    throw createError({ statusCode: 404, message: 'User not found' })
  }

  return user
})
```

```ts
// server/middleware/auth.ts
export default defineEventHandler((event) => {
  const token = getHeader(event, 'authorization')

  if (!token) {
    throw createError({ statusCode: 401, message: 'Unauthorized' })
  }
})
```

### 5. 布局系统

```vue
<!-- layouts/default.vue -->
<template>
  <div class="app">
    <header>
      <nav>
        <NuxtLink to="/">Home</NuxtLink>
        <NuxtLink to="/users">Users</NuxtLink>
      </nav>
    </header>
    <main>
      <slot />
    </main>
    <footer>Footer</footer>
  </div>
</template>
```

```vue
<!-- layouts/admin.vue -->
<template>
  <div class="admin-layout">
    <AdminSidebar />
    <main><slot /></main>
  </div>
</template>
```

```vue
<!-- pages/admin/dashboard.vue -->
<template>
  <div>Dashboard</div>
</template>

<script setup>
definePageMeta({ layout: 'admin' })
</script>
```

### 6. 混合渲染 — 路由规则

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // 首页：SSR + 1 小时缓存
    '/': { prerender: true, swr: 3600 },
    // 博客：构建时静态生成
    '/blog/**': { prerender: true },
    // API：CORS + 缓存
    '/api/**': { cors: true, headers: { 'cache-control': 's-maxage=60' } },
    // 后台：纯客户端渲染
    '/admin/**': { ssr: false },
    // 旧页面重定向
    '/old-page': { redirect: '/new-page' },
  },
})
```

### 7. 状态管理

```ts
// composables/useCart.ts
export const useCart = () => useState<CartItem[]>('cart', () => [])

// 自动导入，无需手动 import
```

```vue
<!-- 任何组件中直接使用 -->
<script setup>
const cart = useCart()

function addItem(item: CartItem) {
  cart.value.push(item)
}

function removeItem(id: number) {
  cart.value = cart.value.filter(i => i.id !== id)
}
</script>
```

### 8. SEO

```vue
<script setup>
useHead({
  title: 'My App',
  meta: [
    { name: 'description', content: 'My awesome app' },
    { property: 'og:title', content: 'My App' },
    { property: 'og:description', content: 'My awesome app' },
  ],
})

// 动态 SEO
const { data: article } = await useFetch('/api/article/1')

useHead({
  title: article.value?.title,
  meta: [
    { name: 'description', content: article.value?.summary },
    { property: 'og:image', content: article.value?.cover },
  ],
})
</script>
```

### 9. 中间件

```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const token = useCookie('token')

  if (!token.value && to.path.startsWith('/admin')) {
    return navigateTo('/login')
  }
})
```

```vue
<!-- 页面中使用 -->
<script setup>
definePageMeta({
  middleware: 'auth',
})
</script>
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Hydration 不匹配 | 服务端和客户端渲染结果不一致 | 避免 `Date.now()`、`Math.random()` 在模板中直接使用 |
| 环境变量不生效 | 客户端无法访问服务端变量 | 使用 `runtimeConfig.public` |
| 第三方库报错 | 库依赖 `window` / `document` | 用 `<ClientOnly>` 包裹或动态导入 |
| SSR 性能差 | 每次请求都完整渲染 | 启用 SWR 缓存 / 静态预渲染 |
| API 跨域 | 服务端请求无跨域问题，客户端有 | Server API 做代理 |

### 最佳实践

1. **路由规则先行**：项目初期定义好每个路由的渲染模式。
2. **Server API 代理**：第三方 API 通过 Server API 代理，避免跨域和 Key 泄露。
3. **`<ClientOnly>`** ：纯客户端组件用 `<ClientOnly>` 包裹。
4. **`useHead` 做 SEO**：每个页面设置 title 和 meta。
5. **`useState` 共享状态**：跨组件状态用 `useState`，SSR 自动脱水/注水。

---

## 面试题

### 1. Nuxt.js 的 SSR 渲染流程是什么？

**答**：(1) 用户请求 URL；(2) Nuxt 服务端匹配路由，执行页面组件的 `<script setup>`；(3) `useFetch` / `useAsyncData` 在服务端执行数据获取；(4) Vue 将组件渲染为 HTML 字符串；(5) 将 HTML + 脱水后的状态（payload）注入 HTML 模板；(6) 返回完整 HTML 给浏览器；(7) 浏览器渲染 HTML（用户可见内容）；(8) 加载 JS，Vue 进行 Hydration——将静态 HTML 附加事件监听和响应式；(9) Hydration 完成后，页面变为可交互。关键：`useFetch` 只执行一次（服务端），客户端直接使用脱水的数据，不重复请求。

---

### 2. useFetch 和 $fetch 有什么区别？

**答**：`useFetch` 是 Nuxt 的组合函数，自动处理 SSR 数据获取——服务端获取数据、脱水到 HTML payload、客户端 Hydration 时复用数据不重复请求。它返回响应式引用（`data`、`pending`、`error`）。`$fetch` 是底层的 fetch 封装（基于 ofetch），每次调用都发起 HTTP 请求，不处理 SSR 脱水/注水，不返回响应式引用。规则：页面数据获取用 `useFetch`，Server API 内部调用其他 API、事件处理中的请求用 `$fetch`。

---

### 3. Nuxt.js 的混合渲染是什么？如何配置？

**答**：混合渲染允许每个路由使用不同的渲染策略，通过 `routeRules` 配置。例如：首页用 SSG（`prerender: true`），博客用 SWR（`swr: 3600`），后台用 CSR（`ssr: false`），API 路由设置缓存头。这解决了"一刀切"的问题——SEO 敏感页面需要 SSR，后台管理不需要 SSR 却要承担渲染开销。Nuxt 在构建时根据 `routeRules` 生成对应的 HTML（prerender），运行时根据路由规则决定是返回预渲染 HTML、实时渲染还是 CSR。

---

### 4. Nuxt.js 的 Server API 和传统 Express 有什么区别？

**答**：Nuxt Server API 基于 h3（轻量 HTTP 框架），与传统 Express 的区别：(1) **自动路由**——`server/api/users.ts` 自动映射为 `/api/users`，无需手动 `app.get()`；(2) **类型安全**——`defineEventHandler` 的参数和返回值有 TypeScript 类型；(3) **自动导入**——`getQuery`、`getRouterParam` 等工具函数自动导入；(4) **部署适配**——h3 的 `event` 对象是跨运行时的，同一份代码可部署到 Node.js、Cloudflare Workers、Vercel Edge 等。Express 只能在 Node.js 运行。

---

### 5. Nuxt.js 的 Hydration 不匹配问题怎么排查和解决？

**答**：Hydration 不匹配是指服务端渲染的 HTML 与客户端 Hydration 时重新渲染的 DOM 不一致。原因：(1) 使用了 `Date.now()`、`Math.random()` 等服务端/客户端结果不同的值；(2) 使用了 `window.innerWidth` 等浏览器 API；(3) 第三方库在服务端和客户端渲染结果不同。排查：(1) 浏览器控制台的 hydration mismatch 警告；(2) 查看页面源代码（SSR HTML）与 DevTools Elements 面板（Hydration 后 DOM）的差异。解决：(1) 用 `<ClientOnly>` 包裹客户端专有内容；(2) 在 `onMounted` 中设置依赖浏览器 API 的值；(3) 用 `useState` 确保服务端和客户端共享初始值。

---

### 6. Nuxt.js 和 Next.js 的核心差异是什么？

**答**：核心差异是底层框架和哲学：(1) **框架**——Nuxt 基于 Vue，Next 基于 React；(2) **数据获取**——Nuxt 的 `useFetch` 在 SSR 和 CSR 中行为一致，Next 13+ 的 Server Components 在服务端获取数据后序列化到客户端，客户端组件用 `use` hook；(3) **全栈**——Nuxt 的 Server API 内置（h3），Next 的 API Routes 基于 Node.js；(4) **灵活性**——Nuxt 约定更重（目录结构、自动导入），Next 更灵活但配置更多；(5) **部署**——Nuxt 通过 Nitro 适配 30+ 平台，Next 深度绑定 Vercel。选型取决于团队技术栈——Vue 团队选 Nuxt，React 团队选 Next。

---

### 7. Nuxt.js 的 useState 和 Vue 的 ref 有什么区别？

**答**：`useState` 是 Nuxt 提供的跨 SSR 共享状态，`ref` 是 Vue 原生的响应式引用。区别：(1) **SSR 兼容**——`useState` 的值在服务端渲染后自动脱水到 HTML payload，客户端 Hydration 时自动恢复，确保服务端和客户端初始值一致；`ref` 在服务端和客户端各创建独立实例，可能导致不匹配；(2) **跨组件共享**——`useState('key', () => initialValue)` 通过 key 标识，所有使用相同 key 的组件共享同一个状态实例；`ref` 每次调用创建新实例；(3) **适用场景**——需要 SSR 共享的全局状态用 `useState`，组件内部状态用 `ref`。

---

### 8. Nuxt.js 项目如何优化首屏加载性能？

**答**：七个优化方向：(1) **预渲染**——静态页面用 `prerender: true`，动态页面用 `swr` 缓存，避免每次请求都 SSR；(2) **路由代码拆分**——Nuxt 自动按页面拆分，确保没有全量 JS；(3) **图片优化**——使用 `@nuxt/image` 自动压缩、懒加载、WebP 格式；(4) **字体优化**——`@nuxt/fonts` 自动下载和 preload；(5) **减少 Hydration 开销**——纯展示页面用 `<ClientOnly>` 或 `ssr: false` 减少 JS；(6) **Payload 压缩**——Nuxt 3 自动压缩 SSR payload，确保生产模式启用；(7) **CDN 缓存**——配合 `routeRules` 的 `swr` 或 `headers` 配置，让 CDN 缓存 SSR 输出。

---

## 相关链接

- [Nuxt.js 官方文档](https://nuxt.com/)
- [Nuxt 3 Migration Guide](https://nuxt.com/docs/getting-started/upgrade)
- [h3 — HTTP 框架](https://h3.unjs.io/)
- [Nitro — 服务引擎](https://nitro.unjs.io/)
- [Vue SSR 指南](https://vuejs.org/guide/scaling-up/ssr.html)

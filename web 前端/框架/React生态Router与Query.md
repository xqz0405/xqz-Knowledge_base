---
tags:
  - Web前端
  - React
  - Router
  - ReactQuery
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# React生态Router与Query

## What — 是什么

> React Router 是 React 官方推荐的路由方案，React Query（TanStack Query）是服务端状态管理库，两者是 React 项目的核心基础设施。

**React Router 核心概念：**

- **路由配置**：`<BrowserRouter>`、`<Routes>`、`<Route>`
- **导航**：`<Link>`、`useNavigate`、`useParams`
- **嵌套布局**：`<Outlet>` + 嵌套 `<Route>`
- **数据路由**（v6.4+）：`loader`、`action`、`useLoaderData`
- **懒加载**：`React.lazy` + `<Suspense>`

**React Query 核心概念：**

- **Query**：`useQuery` 获取数据，自动缓存和去重
- **Mutation**：`useMutation` 修改数据，支持乐观更新
- **缓存策略**：`staleTime`、`cacheTime`（gcTime）、`refetchOnWindowFocus`
- **失效与刷新**：`queryClient.invalidateQueries` 精准刷新
- **分页与无限滚动**：`useInfiniteQuery`

**关键特性：**

- React Router v6 数据路由模式，让路由成为数据获取的入口
- React Query 替代了手写 useEffect + useState 的数据请求模式
- React Query 自动处理缓存、去重、重试、窗口聚焦刷新

## Why — 为什么

**适用场景：**

- React Router：SPA 路由、布局嵌套、权限路由、SSR
- React Query：API 数据获取、缓存管理、乐观更新、轮询

**对比替代方案：**

| 维度 | React Router | TanStack Router | React Query | SWR | Redux + Thunk |
|------|-------------|-----------------|-------------|-----|---------------|
| 路由 | 成熟稳定 | 类型安全更优 | - | - | - |
| 数据获取 | loader | loader | 极好 | 好 | 手写 |
| 缓存 | 无 | 无 | 自动 | 自动 | 手动 |
| 离线/乐观 | 无 | 无 | 内置 | 插件 | 手写 |
| 学习成本 | 低 | 中 | 低 | 低 | 高 |

**优缺点：**

- ✅ React Query 优点：
  - 自动缓存去重，相同请求不发两次
  - 乐观更新体验好
  - DevTools 可视化缓存状态
- ❌ React Query 缺点：
  - 缓存策略需理解 staleTime/cacheTime 区别
  - 不适合纯客户端状态（配合 Zustand 使用）

## How — 怎么用

### 快速上手

**React Router v6：**

```tsx
import { BrowserRouter, Routes, Route, Link, Outlet } from 'react-router-dom';

function Layout() {
    return (
        <div>
            <nav>
                <Link to="/">首页</Link>
                <Link to="/users">用户</Link>
            </nav>
            <Outlet />
        </div>
    );
}

export default function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route element={<Layout />}>
                    <Route index element={<Home />} />
                    <Route path="users" element={<Users />} />
                    <Route path="users/:id" element={<UserDetail />} />
                    <Route path="*" element={<NotFound />} />
                </Route>
            </Routes>
        </BrowserRouter>
    );
}
```

**React Query：**

```tsx
import { QueryClient, QueryClientProvider, useQuery, useMutation } from '@tanstack/react-query';

const queryClient = new QueryClient({
    defaultOptions: {
        queries: { staleTime: 5 * 60 * 1000, retry: 1 },
    },
});

// 在 App 最外层
<QueryClientProvider client={queryClient}>
    <App />
    <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

### 代码示例

**数据路由模式（v6.4+）：**

```tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
    {
        element: <Layout />,
        children: [
            {
                index: true,
                element: <Home />,
                loader: async () => {
                    const stats = await fetchStats();
                    return { stats };
                },
            },
            {
                path: 'users/:id',
                element: <UserDetail />,
                loader: async ({ params }) => {
                    return fetchUser(params.id!);
                },
            },
        ],
    },
]);

export default function App() {
    return <RouterProvider router={router} />;
}

// 组件中使用 loader 数据
function UserDetail() {
    const user = useLoaderData() as User;
    return <h1>{user.name}</h1>;
}
```

**useQuery 基础：**

```tsx
function UserList() {
    const { data, isLoading, error, refetch } = useQuery({
        queryKey: ['users'],
        queryFn: () => fetch('/api/users').then(r => r.json()),
        staleTime: 30_000, // 30s 内不重新请求
    });

    if (isLoading) return <Spinner />;
    if (error) return <Error message={error.message} />;

    return (
        <ul>
            {data.map(user => <li key={user.id}>{user.name}</li>)}
        </ul>
    );
}
```

**useMutation 乐观更新：**

```tsx
function UpdateProfile() {
    const queryClient = useQueryClient();

    const mutation = useMutation({
        mutationFn: (data: ProfileUpdate) => api.updateProfile(data),
        // 乐观更新：先改缓存，请求失败回滚
        onMutate: async (newData) => {
            await queryClient.cancelQueries({ queryKey: ['profile'] });
            const previous = queryClient.getQueryData(['profile']);
            queryClient.setQueryData(['profile'], (old: Profile) => ({
                ...old, ...newData,
            }));
            return { previous };
        },
        onError: (_err, _newData, context) => {
            queryClient.setQueryData(['profile'], context?.previous);
        },
        onSettled: () => {
            queryClient.invalidateQueries({ queryKey: ['profile'] });
        },
    });

    return (
        <form onSubmit={(e) => {
            e.preventDefault();
            mutation.mutate({ name: 'Alice' });
        }}>
            <button disabled={mutation.isPending}>保存</button>
        </form>
    );
}
```

**无限滚动：**

```tsx
function InfinitePosts() {
    const {
        data, fetchNextPage, hasNextPage, isFetchingNextPage,
    } = useInfiniteQuery({
        queryKey: ['posts'],
        queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
        getNextPageParam: (lastPage) => lastPage.nextCursor,
        initialPageParam: 1,
    });

    return (
        <>
            {data?.pages.map(page =>
                page.items.map(post => <PostCard key={post.id} post={post} />)
            )}
            <button
                onClick={() => fetchNextPage()}
                disabled={!hasNextPage || isFetchingNextPage}
            >
                {isFetchingNextPage ? '加载中...' : '加载更多'}
            </button>
        </>
    );
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 组件内 useQuery 重复请求 | queryKey 不同 | 统一 queryKey 工厂函数 |
| 窗口聚焦频繁重请求 | `refetchOnWindowFocus` 默认开启 | 关闭或设置合理的 staleTime |
| 乐观更新闪一下 | onError 后数据回滚 | 确保错误处理正确恢复 previous |
| 嵌套路由 Outlet 不显示 | 父 Route 缺少 `<Outlet>` | 父布局组件必须渲染 `<Outlet/>` |
| 路由跳转后滚动位置不变 | 默认不重置滚动 | 用 `<ScrollRestoration>` 或手动 scrollTo |

### 最佳实践

- queryKey 用工厂函数统一管理：`userKeys.all`、`userKeys.detail(id)`
- staleTime 根据数据变化频率设置（用户信息 5min，实时数据 0）
- 乐观更新用于立即响应，`invalidateQueries` 用于最终一致性
- 路由用数据路由模式（createBrowserRouter），告别手写 useEffect 请求
- 配合 Zustand 管理纯客户端状态，React Query 管理服务端状态

## 面试题

**Q1: BrowserRouter 和 HashRouter 的区别是什么？**
> BrowserRouter 基于 HTML5 History API（pushState），URL 无 # 号，需服务端配置回退到 index.html；HashRouter 基于 URL 的 hash 部分（#），无需服务端配置，但 URL 不美观且不利于 SEO。

**Q2: React Query 的 staleTime 和 cacheTime（gcTime）有什么区别？**
> `staleTime` 控制数据何时从"新鲜"变为"过期"，过期后下次使用会触发后台重新请求；`cacheTime`（v5 改名 `gcTime`）控制数据在缓存中保留多久，过期且无观察者时被垃圾回收。staleTime 影响是否 refetch，cacheTime 影响是否删除缓存。

**Q3: 什么是乐观更新？React Query 如何实现？**
> 乐观更新指在 mutation 请求完成前先更新 UI，假设操作会成功。实现方式：在 `onMutate` 中先缓存旧值并更新缓存数据，若 `onError` 回滚到旧值，`onSettled` 中 `invalidateQueries` 保证最终一致性。

**Q4: useQuery 的 queryKey 有什么作用？**
> queryKey 是缓存的唯一标识，相同 key 共享同一份数据和请求状态。key 不同则视为独立查询。推荐用数组 + 工厂函数管理，如 `['users', id]` 可精确区分不同用户的缓存。

---

**相关链接：**
- [[React核心]]
- [[React Hooks详解]]
- [[状态管理方案]]

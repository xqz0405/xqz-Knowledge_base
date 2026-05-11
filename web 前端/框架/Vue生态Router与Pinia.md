---
tags:
  - Web前端
  - Vue
  - Router
  - Pinia
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Vue生态Router与Pinia

## What — 是什么

> Vue Router 是 Vue.js 官方路由管理器，Pinia 是 Vue.js 官方状态管理库（替代 Vuex），两者是 Vue 项目的标配基础设施。

**Vue Router 核心概念：**

- **路由配置**：`createRouter`、`createWebHistory`/`createWebHashHistory`
- **导航守卫**：`beforeEach`、`beforeResolve`、`afterEach`、路由独享守卫、组件内守卫
- **动态路由**：`/user/:id`、`props: true`
- **嵌套路由**：`children` 配置 + `<router-view>`
- **懒加载**：`() => import('./Page.vue')`

**Pinia 核心概念：**

- **Store 定义**：`defineStore` + Setup 语法或 Options 语法
- **State**：响应式数据（`ref`/`reactive`）
- **Getters**：计算属性（`computed`）
- **Actions**：同步/异步方法（普通函数/`async`）
- **插件**：持久化、DevTools 集成

**关键特性：**

- Vue Router 4 支持 Vue 3，Composition API 友好（`useRoute`/`useRouter`）
- Pinia 无 mutations，TypeScript 天然支持，模块自动按需加载
- Pinia 的 store 之间可直接互相引用

## Why — 为什么

**适用场景：**

- Vue Router：SPA 路由管理、权限控制、页面切换动画
- Pinia：跨组件状态共享、全局状态持久化、异步数据缓存

**对比替代方案：**

| 维度 | Vue Router | 原生 History API | Pinia | Vuex |
|------|-----------|-----------------|-------|------|
| 路由管理 | 完善 | 需手动实现 | - | - |
| TypeScript | 优秀 | 无关 | 天然支持 | 需额外类型 |
| 模块化 | 嵌套路由 | 无关 | 自动按需 | 需注册模块 |
| Mutations | 无关 | 无关 | 无（简化） | 必须通过 mutation |
| 学习成本 | 低 | - | 低 | 中等 |

**优缺点：**

- ✅ Pinia 优点：
  - 无 mutations，action 直接修改 state
  - 完美 TypeScript 推断
  - Store 可互相引用，无需嵌套
- ❌ Pinia 缺点：
  - 时间旅行调试不如 Vuex 成熟
  - 大型项目需要规范 state 命名避免冲突

## How — 怎么用

### 快速上手

**Vue Router：**

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    { path: '/', component: () => import('@/views/Home.vue') },
    { path: '/user/:id', component: () => import('@/views/User.vue'), props: true },
    {
        path: '/admin',
        component: () => import('@/layouts/Admin.vue'),
        children: [
            { path: 'dashboard', component: () => import('@/views/Dashboard.vue') },
            { path: 'settings', component: () => import('@/views/Settings.vue') },
        ],
    },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
```

**Pinia：**

```typescript
// stores/user.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useUserStore = defineStore('user', () => {
    // State
    const user = ref<User | null>(null);
    const token = ref(localStorage.getItem('token') || '');

    // Getters
    const isLoggedIn = computed(() => !!token.value);
    const displayName = computed(() => user.value?.name ?? 'Guest');

    // Actions
    async function login(credentials: LoginForm) {
        const res = await api.login(credentials);
        token.value = res.token;
        user.value = res.user;
        localStorage.setItem('token', res.token);
    }

    function logout() {
        token.value = '';
        user.value = null;
        localStorage.removeItem('token');
    }

    return { user, token, isLoggedIn, displayName, login, logout };
});
```

### 代码示例

**路由守卫（权限控制）：**

```typescript
router.beforeEach(async (to, from) => {
    const userStore = useUserStore();

    // 不需要登录的页面直接放行
    if (to.meta.public) return true;

    // 未登录则跳转登录页
    if (!userStore.isLoggedIn) {
        return { name: 'login', query: { redirect: to.fullPath } };
    }

    // 首次获取用户信息
    if (!userStore.user) {
        try {
            await userStore.fetchProfile();
        } catch {
            userStore.logout();
            return { name: 'login' };
        }
    }

    // 权限检查
    if (to.meta.roles && !to.meta.roles.includes(userStore.user.role)) {
        return { name: 'forbidden' };
    }

    return true;
});
```

**组件中使用：**

```vue
<script setup lang="ts">
import { useRoute, useRouter } from 'vue-router';
import { useUserStore } from '@/stores/user';

const route = useRoute();
const router = useRouter();
const userStore = useUserStore();

// 路由参数
const userId = computed(() => route.params.id as string);

// 编程式导航
function goProfile() {
    router.push({ name: 'user', params: { id: userStore.user.id } });
}
</script>

<template>
    <nav v-if="userStore.isLoggedIn">
        <span>{{ userStore.displayName }}</span>
        <button @click="userStore.logout()">退出</button>
    </nav>
</template>
```

**Pinia Store 互引：**

```typescript
// stores/cart.ts
import { useUserStore } from './user';

export const useCartStore = defineStore('cart', () => {
    const items = ref<CartItem[]>([]);

    const totalPrice = computed(() =>
        items.value.reduce((sum, item) => sum + item.price * item.qty, 0)
    );

    // 会员折扣：引用另一个 store
    const finalPrice = computed(() => {
        const userStore = useUserStore(); // 在 action/computed 中调用，避免循环依赖
        return userStore.isLoggedIn ? totalPrice.value * 0.9 : totalPrice.value;
    });

    return { items, totalPrice, finalPrice };
});
```

**Pinia 持久化插件：**

```typescript
// plugins/persistedstate.ts
import type { PiniaPluginContext } from 'pinia';

export function persistedState({ store }: PiniaPluginContext) {
    const saved = localStorage.getItem(`pinia-${store.$id}`);
    if (saved) store.$patch(JSON.parse(saved));

    store.$subscribe((_, state) => {
        localStorage.setItem(`pinia-${store.$id}`, JSON.stringify(state));
    });
}

// main.ts
const pinia = createPinia();
pinia.use(persistedState);
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 路由跳转后页面不滚动 | 默认保持滚动位置 | `scrollBehavior` 配置或 `window.scrollTo(0, 0)` |
| Pinia 在守卫中未初始化 | `useUserStore()` 在 app.use(pinia) 之前调用 | 在守卫函数内部调用，而非模块顶层 |
| 动态路由刷新 404 | 动态添加的路由未持久化 | 刷新时重新执行 `router.addRoute` |
| Store 循环依赖 | 两个 store 模块顶层互相引用 | 在 action/computed 中延迟引用 |

### 最佳实践

- 路由懒加载用 `() => import()`，配合 `vite-plugin-compression` 预压缩
- 路由守卫统一处理权限，组件内用 `onBeforeRouteLeave` 处理离开确认
- Pinia 优先用 Setup 语法（Composition API 风格）
- 大型项目按功能拆分 store，避免单个 store 过大
- Token 等敏感数据用 httpOnly cookie，不要只存 localStorage

## 面试题

**Q1: Vue Router 导航守卫的执行顺序是什么？**
> 完整顺序：① beforeRouteLeave（组件内，离开时）；② beforeEach（全局前置）；③ beforeEnter（路由独享）；④ beforeRouteEnter（组件内，进入时）；⑤ beforeResolve（全局解析）；⑥ afterEach（全局后置）；⑦ beforeRouteEnter 的 next 回调（DOM 已挂载）。

**Q2: Pinia 和 Vuex 的核心区别是什么？**
> ① Pinia 去除了 mutations，action 直接修改 state，简化代码；② Pinia 天然支持 TypeScript，无需手动类型声明；③ Pinia 无需嵌套模块，每个 Store 独立，按需自动加载；④ Pinia 支持 Composition API 风格定义 Store。

**Q3: Pinia 的 Store 之间如何互相引用？如何避免循环依赖？**
> 在 action 或 computed 中延迟调用另一个 Store 的 useXxx()，而非在模块顶层调用。若两个 Store 顶层互相引用会产生循环依赖。延迟调用确保双方都已初始化完成。

**Q4: Pinia 在路由守卫中使用时为什么可能报错？如何解决？**
> 在 `beforeEach` 中调用 `useUserStore()` 时，若早于 `app.use(pinia)` 执行，Pinia 尚未安装。解决方案：确保 `app.use(pinia)` 在 `app.use(router)` 之前调用，或在守卫函数内部而非模块顶层调用 useStore。

---

**相关链接：**
- [[Vue核心]]
- [[Vue3响应式原理]]
- [[状态管理方案]]

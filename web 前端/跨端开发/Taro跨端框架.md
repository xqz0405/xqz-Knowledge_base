---
tags:
  - Web前端
  - Taro
  - 跨端开发
  - 小程序
  - React
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Taro跨端框架

## What — 是什么

> Taro 是京东凹凸实验室推出的多端统一开发框架，支持使用 React/Vue 语法编写一次代码，编译到微信/支付宝/百度/字节/京东/QQ 小程序、H5、React Native 等多个平台。Taro 3+ 基于"运行时框架"架构，不再重写 JSX/VNode，而是在小程序逻辑层模拟 DOM/BOM API，使 React/Vue 直接运行。京东、贝壳找房、网易严选、携程等均使用 Taro。

**核心概念：**

- **多端编译**：一套代码编译到多个小程序平台 + H5 + RN，通过编译器处理平台差异
- **运行时框架（Taro 3+）**：在小程序逻辑层注入 DOM/BOM API 模拟层（`@tarojs/runtime`），让 React/Vue 直接在小程序中运行
- **Taro DSL**：统一封装各平台组件（`<View>` `<Text>` `<Image>` 等）和 API（`Taro.xxx`），抹平差异
- **自定义渲染器**：Taro 实现了 React Custom Renderer 和 Vue Custom Renderer，将虚拟 DOM 映射到小程序模板
- **Template 模板**：小程序视图层通过预生成的模板 WXML 渲染，逻辑层 setData 驱动更新
- **插件系统**：完整的编译期和运行时插件机制，可扩展编译流程、添加新平台

**核心架构（Taro 3+）：**

```
┌──────────────────────────────────────────────────┐
│                   Taro App                        │
├──────────────┬───────────────────────────────────┤
│              │         小程序逻辑层                │
│   React/Vue  │  ┌─────────────────────────────┐  │
│   业务代码    │  │ @tarojs/runtime              │  │
│              │  │  DOM/BOM 模拟层              │  │
│  ┌──────────┐│  │  ├── document.createElement  │  │
│  │ React    ││  │  ├── event system           │  │
│  │ Reconcil-││  │  └── style processing        │  │
│  │ er       ││  ├─────────────────────────────┤  │
│  └──────────┘│  │ Taro Custom Renderer         │  │
│  ┌──────────┐│  │  虚拟DOM → 小程序 setData    │  │
│  │ Vue      ││  │  事件代理 → 小程序 event     │  │
│  │ Renderer ││  └──────────┬──────────────────┘  │
│  └──────────┘│             │ setData              │
│              │             ▼                      │
│              │  ┌─────────────────────────────┐  │
│              │  │     小程序视图层（WXML）      │  │
│              │  │  Template 模板预生成         │  │
│              │  │  组件树 ← setData 驱动更新   │  │
│              │  └─────────────────────────────┘  │
├──────────────┴───────────────────────────────────┤
│              Taro CLI + 编译器                    │
│  代码转换 → 平台适配 → 模板生成 → 构建          │
├──────────────────────────────────────────────────┤
│  微信 │ 支付宝 │ 字节 │ 百度 │ 京东 │ H5 │ RN  │
└──────────────────────────────────────────────────┘
```

- 设计理念：Write Once, Run Anywhere，用前端框架的思维方式开发小程序
- 核心模块：Taro CLI + 编译器 + Runtime + Renderer + API 层
- 数据流：React/Vue 状态更新 → Reconciler diff → 生成更新指令 → setData → 小程序视图刷新
- 与 Taro 1/2 的区别：不再 AST 重写 JSX，改为运行时模拟 DOM，支持完整的 React/Vue 特性

**Taro 版本演进：**

| 版本 | 架构 | JSX 处理 | 框架支持 | 包体积 |
|------|------|---------|---------|--------|
| Taro 1/2 | 编译时重写 | AST 转换为小程序模板 | React only | 小 |
| Taro 3+ | 运行时模拟 | JSX 原样保留，运行时渲染 | React/Vue/Vue3/Preact/Svelte | 较大（含 runtime） |
| Taro 4 | 运行时优化 | 增量更新 + 虚拟 DOM 优化 | 全框架 | 优化后接近 Taro 2 |

**支持平台：**

| 平台 | 编译命令 | 说明 |
|------|---------|------|
| 微信小程序 | `npm run build:weapp` | 最成熟，功能最全 |
| 支付宝小程序 | `npm run build:alipay` | 支付宝生态 |
| 百度小程序 | `npm run build:swan` | 百度搜索 |
| 字节跳动小程序 | `npm run build:tt` | 抖音/头条系 |
| 京东小程序 | `npm run build:jd` | 京东生态 |
| QQ 小程序 | `npm run build:qq` | QQ 端 |
| H5 | `npm run build:h5` | 浏览器 |
| React Native | `npm run build:rn` | 原生 App（实验性） |
| 鸿蒙 | `npm run build:harmony` | 鸿蒙 HarmonyOS（Taro 4+） |

## Why — 为什么

**适用场景：**

- React/Vue 团队需要快速覆盖多个小程序平台
- 需要同时发布 H5 + 小程序 + App 的项目
- 已有 React/Vue 组件库，希望复用到小程序
- 大型项目需要更好的 TypeScript 支持和工程化能力
- 需要跨端但小程序是主要目标平台

**对比同类框架：**

| 维度 | Taro | uni-app | Remax | mpvue |
|------|------|---------|-------|-------|
| 框架支持 | React/Vue/Preact/Svelte | Vue only | React only | Vue only |
| 架构 | 运行时模拟 DOM | 编译时转换 | 运行时（React Reconciler） | 编译时重写 |
| 小程序支持 | 极好（京东维护） | 极好（DCloud） | 较好 | 已停止维护 |
| H5 | 支持 | 支持 | 不支持 | 不支持 |
| React 生态 | 完整支持 | 不支持 | 支持 | 不支持 |
| TypeScript | 一等公民 | 支持 | 支持 | 差 |
| 性能 | 中等（runtime 开销） | 中等 | 中等 | 好（编译时） |
| 包体积 | 较大（含 runtime） | 小 | 较大 | 小 |
| 插件系统 | 完整 | 有限 | 无 | 无 |
| 维护状态 | 活跃 | 活跃 | 缓慢 | 已停止 |

**优缺点：**

- ✅ 优点：
  - 支持 React/Vue 多框架，React 生态完整可用
  - 运行时架构支持完整的 JSX/VNode 语法（条件渲染、高阶组件等）
  - TypeScript 一等公民，类型推导完善
  - 插件系统强大，可扩展编译流程
  - 跨端 API 统一，Taro.xxx 模式学习成本低
  - 京东大厂维护，稳定性有保障
  - 支持 CSS Modules、Sass、Less 等样式方案
  - 社区活跃，UI 组件库丰富（Taro UI、NutUI 等）
- ❌ 缺点：
  - 运行时架构包体积较大，首次加载慢于编译时方案
  - setData 频繁时性能不如原生小程序
  - 复杂场景（长列表、复杂动画）需要针对性优化
  - 各小程序平台差异仍需条件编译处理
  - React Native 端支持不够成熟
  - 部分高级特性（Portal、Suspense）在小程序端有限制

## How — 怎么用

### 快速上手

```bash
# 安装 Taro CLI
npm install -g @tarojs/cli

# 创建项目
taro init my-app
# 交互式选择：框架(React/Vue)、TypeScript、CSS 预处理器、模板等

# 或指定配置创建
taro init my-app --template react-ts

cd my-app

# 开发模式
npm run dev:weapp      # 微信小程序
npm run dev:h5         # H5
npm run dev:alipay     # 支付宝

# 构建
npm run build:weapp
npm run build:h5
```

**项目结构：**

```
my-app/
├── src/
│   ├── pages/              # 页面
│   │   ├── index/
│   │   │   └── index.tsx
│   │   └── detail/
│   │       └── index.tsx
│   ├── components/         # 组件
│   ├── store/              # 状态管理
│   ├── services/           # 接口服务
│   ├── utils/              # 工具函数
│   ├── assets/             # 静态资源
│   ├── app.config.ts       # 全局配置（路由、TabBar、窗口）
│   ├── app.tsx             # 入口组件
│   └── index.html          # H5 入口
├── config/
│   ├── dev.js              # 开发环境配置
│   ├── prod.js             # 生产环境配置
│   └── index.js            # 主配置
├── project.config.json     # 微信小程序配置
├── babel.config.js
├── tsconfig.json
└── package.json
```

### 路由与页面配置

```ts
// src/app.config.ts
export default defineAppConfig({
  pages: [
    'pages/index/index',
    'pages/detail/index',
    'pages/profile/index',
  ],
  subPackages: [
    {
      root: 'pages-sub/order',
      pages: [
        'list/index',
        'detail/index',
      ],
    },
    {
      root: 'pages-sub/user',
      pages: [
        'settings/index',
        'address/index',
      ],
    },
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'My App',
    navigationBarTextStyle: 'black',
  },
  tabBar: {
    color: '#999',
    selectedColor: '#1677ff',
    borderStyle: 'black',
    backgroundColor: '#fff',
    list: [
      {
        pagePath: 'pages/index/index',
        text: '首页',
        iconPath: 'assets/tab/home.png',
        selectedIconPath: 'assets/tab/home-active.png',
      },
      {
        pagePath: 'pages/profile/index',
        text: '我的',
        iconPath: 'assets/tab/profile.png',
        selectedIconPath: 'assets/tab/profile-active.png',
      },
    ],
  },
});
```

**页面配置：**

```ts
// src/pages/detail/index.config.ts
export default definePageConfig({
  navigationBarTitleText: '详情页',
  navigationBarBackgroundColor: '#ffffff',
  enablePullDownRefresh: true,
  usingComponents: {
    // 引入微信原生组件
    'van-button': '@vant/weapp/button/index',
  },
});
```

**路由导航：**

```tsx
import Taro from '@tarojs/taro';

// 跳转（保留当前页）
Taro.navigateTo({
  url: '/pages/detail/index?id=1&name=test',
  events: {
    acceptDataFromOpenedPage: (data) => console.log(data),
  },
  success: (res) => {
    res.eventChannel.emit('acceptDataFromOpenerPage', { data: 'hello' });
  },
});

// 重定向（关闭当前页）
Taro.redirectTo({ url: '/pages/login/index' });

// 返回
Taro.navigateBack({ delta: 1 });

// 切换 TabBar
Taro.switchTab({ url: '/pages/index/index' });

// 关闭所有页面打开某页
Taro.reLaunch({ url: '/pages/index/index' });

// 获取路由参数
import { useRouter } from '@tarojs/taro';

function DetailPage() {
  const router = useRouter();
  console.log(router.params.id);   // "1"
  console.log(router.params.name); // "test"
}
```

### 组件开发

**基础页面：**

```tsx
// src/pages/index/index.tsx
import { View, Text, Button, Image } from '@tarojs/components';
import { useReady, useDidShow, useDidHide, usePullDownRefresh } from '@tarojs/taro';
import { useState, useCallback } from 'react';
import Taro from '@tarojs/taro';
import './index.scss';

export default function Index() {
  const [list, setList] = useState<string[]>([]);
  const [loading, setLoading] = useState(false);

  const fetchData = useCallback(async () => {
    setLoading(true);
    // const res = await api.getList();
    setList(['Item 1', 'Item 2', 'Item 3']);
    setLoading(false);
  }, []);

  // 页面就绪
  useReady(() => {
    console.log('页面就绪');
  });

  // 页面显示
  useDidShow(() => {
    fetchData();
  });

  // 页面隐藏
  useDidHide(() => {
    console.log('页面隐藏');
  });

  // 下拉刷新
  usePullDownRefresh(() => {
    fetchData().finally(() => Taro.stopPullDownRefresh());
  });

  const handleClick = () => {
    Taro.showToast({ title: '点击了', icon: 'success' });
  };

  return (
    <View className='index'>
      <Text className='title'>Hello Taro</Text>
      <Button onClick={handleClick}>点击</Button>
      <View className='list'>
        {list.map((item) => (
          <View key={item} className='list-item'>
            <Text>{item}</Text>
          </View>
        ))}
      </View>
    </View>
  );
}
```

**自定义组件：**

```tsx
// src/components/ProductCard/index.tsx
import { View, Text, Image, Button } from '@tarojs/components';
import './index.scss';

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  tags?: string[];
}

interface ProductCardProps {
  product: Product;
  onClick?: (product: Product) => void;
  onAddCart?: (id: number) => void;
}

export default function ProductCard({ product, onClick, onAddCart }: ProductCardProps) {
  return (
    <View className='product-card' onClick={() => onClick?.(product)}>
      <Image className='product-card__img' src={product.image} mode='aspectFill' />
      <View className='product-card__info'>
        <Text className='product-card__name'>{product.name}</Text>
        <View className='product-card__tags'>
          {product.tags?.map((tag) => (
            <Text key={tag} className='product-card__tag'>{tag}</Text>
          ))}
        </View>
        <View className='product-card__bottom'>
          <Text className='product-card__price'>¥{product.price}</Text>
          <Button
            size='mini'
            onClick={(e) => {
              e.stopPropagation();
              onAddCart?.(product.id);
            }}
          >
            加购
          </Button>
        </View>
      </View>
    </View>
  );
}
```

```scss
// src/components/ProductCard/index.scss
.product-card {
  display: flex;
  padding: 24px;
  background: #fff;
  border-radius: 16px;
  margin-bottom: 20px;

  &__img {
    width: 200px;
    height: 200px;
    border-radius: 12px;
    flex-shrink: 0;
  }

  &__info {
    flex: 1;
    margin-left: 24px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  &__name {
    font-size: 32px;
    color: #333;
    lines: 2;
    text-overflow: ellipsis;
  }

  &__tags {
    display: flex;
    gap: 12px;
    margin-top: 12px;
  }

  &__tag {
    font-size: 22px;
    color: #1677ff;
    background: #e6f4ff;
    padding: 4px 12px;
    border-radius: 8px;
  }

  &__bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 16px;
  }

  &__price {
    font-size: 36px;
    color: #ff4d4f;
    font-weight: bold;
  }
}
```

### 条件编译

```tsx
// JS/TS 条件编译
/** @if weapp */
console.log('微信小程序专属逻辑');
/** @endif */

/** @if alipay */
console.log('支付宝小程序专属逻辑');
/** @endif */

/** @if !weapp */
console.log('非微信小程序');
/** @endif */

// 推荐：使用 process.env.TARO_ENV 运行时判断
if (process.env.TARO_ENV === 'weapp') {
  // 微信小程序逻辑
} else if (process.env.TARO_ENV === 'alipay') {
  // 支付宝逻辑
} else if (process.env.TARO_ENV === 'h5') {
  // H5 逻辑
}
```

```scss
/* CSS 条件编译 */
/* #ifdef weapp */
.container {
  padding-bottom: env(safe-area-inset-bottom);
}
/* #endif */

/* #ifdef h5 */
.container {
  max-width: 750px;
  margin: 0 auto;
}
/* #endif */
```

**平台标识：**

| 标识 | 平台 | 标识 | 平台 |
|------|------|------|------|
| `weapp` | 微信小程序 | `alipay` | 支付宝 |
| `swan` | 百度 | `tt` | 字节跳动 |
| `jd` | 京东 | `qq` | QQ |
| `h5` | H5 | `rn` | React Native |

### 状态管理

**Zustand（推荐）：**

```ts
// src/store/user.ts
import { create } from 'zustand';
import Taro from '@tarojs/taro';

interface UserState {
  token: string;
  userInfo: { name: string; avatar: string } | null;
  isLoggedIn: boolean;
  login: (code: string) => Promise<void>;
  logout: () => void;
}

export const useUserStore = create<UserState>((set) => ({
  token: Taro.getStorageSync('token') || '',
  userInfo: null,
  isLoggedIn: !!Taro.getStorageSync('token'),

  login: async (code: string) => {
    // const res = await api.login({ code });
    const token = 'jwt-token-xxx';
    Taro.setStorageSync('token', token);
    set({ token, isLoggedIn: true });
  },

  logout: () => {
    Taro.removeStorageSync('token');
    set({ token: '', userInfo: null, isLoggedIn: false });
    Taro.reLaunch({ url: '/pages/index/index' });
  },
}));
```

```tsx
// 页面中使用
import { useUserStore } from '@/store/user';

function ProfilePage() {
  const { userInfo, isLoggedIn, logout } = useUserStore();

  if (!isLoggedIn) {
    return <View>请先登录</View>;
  }

  return (
    <View>
      <Text>{userInfo?.name}</Text>
      <Button onClick={logout}>退出登录</Button>
    </View>
  );
}
```

**Redux Toolkit：**

```ts
// src/store/index.ts
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';

interface AppState {
  count: number;
  list: string[];
}

const initialState: AppState = {
  count: 0,
  list: [],
};

const appSlice = createSlice({
  name: 'app',
  initialState,
  reducers: {
    increment: (state) => { state.count += 1; },
    decrement: (state) => { state.count -= 1; },
    setList: (state, action: PayloadAction<string[]>) => { state.list = action.payload; },
  },
});

export const { increment, decrement, setList } = appSlice.actions;

export const store = configureStore({
  reducer: { app: appSlice.reducer },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```tsx
// src/app.tsx — 注入 Store
import { Provider } from 'react-redux';
import { store } from '@/store';

function App({ children }) {
  return <Provider store={store}>{children}</Provider>;
}

export default App;
```

### 网络请求封装

```ts
// src/utils/request.ts
import Taro from '@tarojs/taro';

const BASE_URL = process.env.TARO_APP_API_BASE || 'https://api.example.com';

interface RequestOptions {
  url: string;
  method?: keyof Taro.request.Method;
  data?: Record<string, any>;
  header?: Record<string, string>;
  showLoading?: boolean;
}

interface ApiResponse<T = any> {
  code: number;
  data: T;
  message: string;
}

let loadingCount = 0;

function showLoading() {
  if (loadingCount === 0) Taro.showLoading({ title: '加载中', mask: true });
  loadingCount++;
}

function hideLoading() {
  loadingCount--;
  if (loadingCount <= 0) { loadingCount = 0; Taro.hideLoading(); }
}

export function request<T = any>(options: RequestOptions): Promise<T> {
  const { showLoading: show = false } = options;
  if (show) showLoading();

  return new Promise((resolve, reject) => {
    Taro.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data,
      header: {
        'Authorization': `Bearer ${Taro.getStorageSync('token')}`,
        'Content-Type': 'application/json',
        ...options.header,
      },
      success: (res) => {
        if (show) hideLoading();
        const data = res.data as ApiResponse<T>;

        if (res.statusCode === 200 && data.code === 0) {
          resolve(data.data);
        } else if (res.statusCode === 401) {
          Taro.removeStorageSync('token');
          Taro.reLaunch({ url: '/pages/login/index' });
          reject(new Error('登录过期'));
        } else {
          Taro.showToast({ title: data.message || '请求失败', icon: 'none' });
          reject(new Error(data.message));
        }
      },
      fail: (err) => {
        if (show) hideLoading();
        Taro.showToast({ title: '网络异常', icon: 'none' });
        reject(err);
      },
    });
  });
}

export const get = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'GET', data });

export const post = <T = any>(url: string, data?: any, show = false) =>
  request<T>({ url, method: 'POST', data, showLoading: show });

export const put = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'PUT', data });

export const del = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'DELETE', data });
```

```ts
// src/services/product.ts
import { get, post } from '@/utils/request';

export interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

export const getProductList = (params?: { page?: number; size?: number }) =>
  get<Product[]>('/products', params);

export const getProductDetail = (id: number) =>
  get<Product>(`/products/${id}`);

export const createOrder = (productId: number, quantity: number) =>
  post<{ orderId: string }>('/orders', { productId, quantity }, true);
```

### Taro 常用 API

```ts
import Taro from '@tarojs/taro';

// 存储
Taro.setStorageSync('key', 'value');
const val = Taro.getStorageSync('key');
Taro.removeStorageSync('key');
Taro.clearStorageSync();

// 交互反馈
Taro.showToast({ title: '成功', icon: 'success' });
Taro.showLoading({ title: '加载中', mask: true });
Taro.hideLoading();
Taro.showModal({
  title: '确认删除？',
  content: '删除后不可恢复',
  success: (res) => { if (res.confirm) { /* 确认 */ } },
});
Taro.showActionSheet({
  itemList: ['拍照', '从相册选择'],
  success: (res) => console.log(res.tapIndex),
});

// 媒体
Taro.chooseImage({
  count: 9,
  sizeType: ['compressed'],
  sourceType: ['album', 'camera'],
  success: (res) => console.log(res.tempFilePaths),
});

// 设备信息
const systemInfo = Taro.getSystemInfoSync();
console.log(systemInfo.platform, systemInfo.windowWidth, systemInfo.statusBarHeight);

// 位置
Taro.getLocation({
  type: 'gcj02',
  success: (res) => console.log(res.latitude, res.longitude),
});

// 扫码
Taro.scanCode({
  scanType: ['qrCode', 'barCode'],
  success: (res) => console.log(res.result),
});

// 剪贴板
Taro.setClipboardData({ data: '复制内容' });
Taro.getClipboardData({ success: (res) => console.log(res.data) });

// 网络
const networkType = Taro.getNetworkType();
Taro.onNetworkStatusChange((res) => console.log(res.isConnected, res.networkType));
```

### 小程序登录与支付

```ts
// src/services/auth.ts
import Taro from '@tarojs/taro';
import { post } from '@/utils/request';

// 微信登录
export async function wxLogin() {
  // 1. 获取 code
  const { code } = await Taro.login();

  // 2. 发送 code 到后端
  const res = await post<{ token: string; openid: string }>('/auth/wx-login', { code });

  // 3. 存储登录态
  Taro.setStorageSync('token', res.token);
  return res;
}

// 微信支付
export async function wxPay(orderId: string) {
  // 1. 后端创建预付单
  const payParams = await post<{
    timeStamp: string;
    nonceStr: string;
    package: string;
    signType: string;
    paySign: string;
  }>('/pay/create', { orderId });

  // 2. 调起支付
  return new Promise((resolve, reject) => {
    Taro.requestPayment({
      timeStamp: payParams.timeStamp,
      nonceStr: payParams.nonceStr,
      package: payParams.package,
      signType: payParams.signType as 'MD5' | 'HMAC-SHA256',
      paySign: payParams.paySign,
      success: resolve,
      fail: reject,
    });
  });
}
```

### Taro UI 组件库

```bash
# 安装 NutUI（京东出品，Taro 专属）
npm install @nutui/nutui-react-taro
```

```tsx
// 使用 NutUI 组件
import { Button, Cell, Popup, Swiper, TabBar } from '@nutui/nutui-react-taro';
import '@nutui/nutui-react-taro/dist/style.css';

function ShopPage() {
  const [visible, setVisible] = useState(false);

  return (
    <View>
      <Swiper
        autoPlay={3000}
        indicator
        height={300}
      >
        <Swiper.Item><Image src='banner1.jpg' /></Swiper.Item>
        <Swiper.Item><Image src='banner2.jpg' /></Swiper.Item>
      </Swiper>

      <Cell
        title='商品详情'
        isLink
        onClick={() => setVisible(true)}
      />

      <Popup
        visible={visible}
        position='bottom'
        onClose={() => setVisible(false)}
      >
        <View>弹窗内容</View>
      </Popup>

      <TabBar
        value={0}
        onSwitch={(index) => console.log(index)}
      >
        <TabBar.Item title='首页' icon='home' />
        <TabBar.Item title='分类' icon='category' />
        <TabBar.Item title='购物车' icon='cart' />
        <TabBar.Item title='我的' icon='my' />
      </TabBar>
    </View>
  );
}
```

### 自定义 Hook 封装

```ts
// src/hooks/useLoading.ts
import { useState, useCallback } from 'react';
import Taro from '@tarojs/taro';

export function useLoading(defaultLoading = false) {
  const [loading, setLoading] = useState(defaultLoading);

  const withLoading = useCallback(async <T,>(fn: () => Promise<T>): Promise<T> => {
    setLoading(true);
    try {
      return await fn();
    } catch (error: any) {
      Taro.showToast({ title: error.message || '操作失败', icon: 'none' });
      throw error;
    } finally {
      setLoading(false);
    }
  }, []);

  return { loading, withLoading };
}

// src/hooks/usePullDownRefresh.ts
import { useCallback } from 'react';
import Taro from '@tarojs/taro';

export function usePullDownRefresh(fetchFn: () => Promise<void>) {
  const onPullDownRefresh = useCallback(async () => {
    try {
      await fetchFn();
    } finally {
      Taro.stopPullDownRefresh();
    }
  }, [fetchFn]);

  return onPullDownRefresh;
}

// src/hooks/useReachBottom.ts
import { useState, useCallback } from 'react';

export function useReachBottom<T>(
  fetchFn: (page: number) => Promise<{ list: T[]; hasMore: boolean }>,
  pageSize = 10
) {
  const [list, setList] = useState<T[]>([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);

  const loadMore = useCallback(async () => {
    if (!hasMore || loading) return;
    setLoading(true);
    try {
      const result = await fetchFn(page);
      setList((prev) => [...prev, ...result.list]);
      setHasMore(result.hasMore);
      setPage((p) => p + 1);
    } finally {
      setLoading(false);
    }
  }, [page, hasMore, loading, fetchFn]);

  const reset = useCallback(async () => {
    setPage(1);
    setHasMore(true);
    setList([]);
    setLoading(true);
    try {
      const result = await fetchFn(1);
      setList(result.list);
      setHasMore(result.hasMore);
      setPage(2);
    } finally {
      setLoading(false);
    }
  }, [fetchFn]);

  return { list, loading, hasMore, loadMore, reset };
}
```

### 插件开发与使用

```ts
// plugins/taro-plugin-demo/index.ts
import { IPluginContext } from '@tarojs/service';

export default (ctx: IPluginContext, options: Record<string, any>) => {
  ctx.registerMethod('writeFileToDist', (filePath: string, content: string) => {
    ctx.writeFileToDist({ filePath, content });
  });

  // 编译开始
  ctx.onBuildStart(() => {
    console.log('构建开始');
  });

  // 编译完成
  ctx.onBuildComplete(({ stats }) => {
    console.log('构建完成');
  });

  // 修改 Webpack 配置
  ctx.modifyWebpackChain(({ chain }) => {
    chain.plugin('define').tap((args) => {
      args[0]['process.env'].CUSTOM_VAR = JSON.stringify(options.customVar);
      return args;
    });
  });

  // 修改编译产物
  ctx.onBuildFinish(() => {
    console.log('编译结束，可处理产物');
  });
};
```

```js
// config/index.js — 注册插件
const config = {
  plugins: [
    'taro-plugin-demo',
    ['taro-plugin-demo', { customVar: 'hello' }],
  ],
};
```

**常用社区插件：**

| 插件 | 功能 |
|------|------|
| `taro-plugin-compiler-optimization` | 编译优化 |
| `taro-plugin-mini-ci` | 小程序 CI 上传 |
| `taro-plugin-inject` | 注入全局依赖 |
| `taro-plugin-html` | 小程序中使用 HTML 标签 |
| `taro-plugin-tailwind` | Tailwind CSS 支持 |

### 性能优化

**虚拟列表：**

```tsx
import { VirtualList } from '@tarojs/components';

function LongList() {
  const [data, setData] = useState(
    Array.from({ length: 1000 }, (_, i) => ({ id: i, name: `Item ${i}` }))
  );

  const renderItem = (item: { id: number; name: string }, index: number) => (
    <View className='list-item'>
      <Text>{item.name}</Text>
    </View>
  );

  return (
    <VirtualList
      height={600}
      width='100%'
      itemData={data}
      itemCount={data.length}
      itemSize={100}
      renderItem={renderItem}
    />
  );
}
```

**渲染优化：**

```tsx
// 1. 避免不必要的 setData — 使用 React.memo
import { memo } from 'react';

const ExpensiveItem = memo(({ data }: { data: Item }) => {
  return <View>{data.name}</View>;
});

// 2. 拆分大组件，减少单次 setData 数据量
// 差：整个页面一个组件，一次 setData 几十KB
// 好：拆成多个子组件，每个组件独立 setData

// 3. 列表使用 key
{list.map((item) => (
  <View key={item.id}>...</View>
))}

// 4. 条件渲染替代 v-show
// Taro 中 display:none 仍会渲染节点
{visible && <View>只在可见时渲染</View>}

// 5. 图片懒加载
<Image lazyLoad src={url} mode='aspectFill' />

// 6. 预加载分包
Taro.preloadPackages(['pages-sub/order']);
```

**编译配置优化：**

```js
// config/prod.js
module.exports = {
  mini: {
    webpackChain(chain) {
      // 代码分割
      chain.optimization.splitChunks({
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendor',
            chunks: 'all',
          },
        },
      });
    },
    miniCssExtractPluginOption: {
      ignoreOrder: true,
    },
    // 小程序分包优化
    optimizeMainPackage: {
      enable: true,
    },
  },
  h5: {
    publicPath: '/',
    enableSourceMap: false,
    miniCssExtractPluginOption: {
      ignoreOrder: true,
    },
  },
};
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| JSX 中 map 渲染列表闪烁 | 每次 setData 全量更新 | 使用 `React.memo` + 唯一 key，拆分子组件 |
| CSS 选择器不支持 | 小程序组件样式隔离 | 使用 class 选择器，避免嵌套选择器 |
| 自定义组件事件不触发 | 小程序事件冒泡机制不同 | 使用 `onClick` 而非 `onTouchEnd`，注意 `e.stopPropagation()` |
| 包体积过大 | runtime + 框架代码较多 | 分包加载，Tree Shaking，按需引入组件库 |
| Canvas 不可用 | 小程序 Canvas API 与 Web 不同 | 使用 `Taro.createCanvasContext` 或 `<Canvas>` 组件 |
| H5 端路由 404 | SPA 路由需服务端配置 | Nginx 配置 `try_files $uri $uri/ /index.html` |
| useState 更新不生效 | 小程序 setData 异步批处理 | 使用 `useEffect` 监听状态变化，或 `flushSync` 强制同步 |
| 真机调试白屏 | 代码包含 ES2020+ 语法 | 配置 Babel 转译 `targets` 覆盖低版本设备 |
| 页面栈超限 | 小程序限制 10 层 | 用 `Taro.redirectTo` 替代 `navigateTo` |
| 全局样式不生效 | 小程序组件样式隔离 | 在 `app.config.ts` 中设置 `globalObject` 或使用 `:global` |

### 最佳实践

- 优先使用 `process.env.TARO_ENV` 做运行时平台判断，比条件编译更灵活
- 组件粒度拆细，每个小组件独立 setData，减少数据传输量
- 长列表使用 `VirtualList`，避免一次渲染上千节点
- 图片使用 CDN + 懒加载 + 压缩，避免包体积超限
- 状态管理优先选 Zustand（轻量），复杂场景用 Redux Toolkit
- 网络请求统一封装，处理 token 过期、错误重试、loading 状态
- 小程序必须做分包，主包控制在 2MB 以内
- 使用 `React.memo` + `useMemo` + `useCallback` 减少不必要的重渲染
- 样式优先使用 CSS Modules，避免全局污染
- 使用 NutUI 等 Taro 专属组件库，比 Web 组件库兼容性更好

## 面试题

**Q1: Taro 3+ 的运行时架构是怎么工作的？和 Taro 1/2 的编译时架构有什么区别？**
> Taro 3+ 在小程序逻辑层注入 `@tarojs/runtime`，模拟 DOM/BOM API，让 React Reconciler / Vue Renderer 直接运行，虚拟 DOM diff 后通过 setData 更新小程序视图层。Taro 1/2 在编译时将 JSX AST 重写为小程序模板，不支持完整的 JSX 语法（如高阶组件、运行时条件渲染）。运行时架构优势：完整支持 React/Vue 特性；劣势：包体积更大（含 runtime ~50KB），setData 开销高于纯模板方案。

**Q2: Taro 的 setData 性能问题如何优化？**
> ① 组件拆分：将大页面拆为多个子组件，每个组件独立 setData，减少单次数据量；② `React.memo`：避免不必要的子组件重渲染；③ 虚拟列表：长列表用 `VirtualList` 只渲染可见区域；④ 精确更新：用 `useCallback`/`useMemo` 缓存回调和计算值，减少 diff 范围；⑤ 避免大数据：不在 setData 中传递大对象/长列表，只传必要字段；⑥ 懒加载：非首屏内容延迟渲染（`v-if` 替代 `v-show`）。

**Q3: Taro 和 uni-app 怎么选？**
> ① 技术栈：React 团队选 Taro（React 生态完整），Vue 团队选 uni-app（Vue 一等公民）；② 小程序覆盖：两者都支持主流平台，uni-app 小程序 API 更全；③ 包体积：uni-app 编译时方案更小，Taro 运行时方案较大；④ 生态：uni-app 插件市场更大，Taro 插件系统更灵活；⑤ TypeScript：Taro TS 支持更完善；⑥ 性能：简单场景差异不大，复杂场景 Taro 的 runtime 开销稍大。结论：React 选 Taro，Vue 选 uni-app，追求极致性能选原生小程序。

**Q4: Taro 中如何使用微信原生组件？**
> 两种方式：① 在页面配置 `usingComponents` 引入原生组件（如 Vant Weapp），在 JSX 中直接使用（`<van-button type="primary">按钮</van-button>`），注意事件名需要转换（`bind:click` → `onClick`）；② 使用 Taro 的 `@tarojs/plugin-html` 插件，支持在 JSX 中使用 HTML 标签，编译时自动映射为小程序组件。注意原生组件的样式隔离和事件机制与 Taro 组件有差异，复杂场景建议用 Taro 组件库（NutUI）替代。

**Q5: Taro 的虚拟列表实现原理是什么？**
> VirtualList 只渲染可视区域内的列表项，通过监听滚动事件计算当前可视区域的起止索引，只渲染该范围内的项目。核心参数：`height`（容器高度）、`itemSize`（每项高度）、`itemCount`（总数量）、`itemData`（数据源）、`renderItem`（渲染函数）。原理：维护一个虚拟索引范围（startIdx ~ endIdx），滚动时重新计算范围并只渲染该范围的元素，上下各预渲染几个缓冲项确保滚动流畅。适用于 1000+ 项的长列表场景。

**Q6: Taro 项目如何做分包加载？**
> 在 `app.config.ts` 中配置 `subPackages`，将不同功能模块的页面分到不同子包：`{ root: 'pages-sub/order', pages: ['list/index', 'detail/index'] }`。微信小程序限制：主包 2MB、单子包 2MB、总包 20MB。TabBar 页面必须在主包。建议按功能模块划分子包，首屏不需要的页面放子包。可配置 `preloadRule` 预加载常用子包。Taro 还支持 `optimizeMainPackage` 自动分析主包依赖，将部分代码移入子包。

**Q7: Taro 中 React 的生命周期和小程序生命周期如何对应？**
> React 生命周期与小程序生命周期独立运行。映射关系：`useEffect(() => {}, [])` ≈ `useReady`（页面初次渲染完成）；`useDidShow` 对应小程序 `onShow`（每次页面显示触发，无 React 对应）；`useDidHide` 对应 `onHide`；`usePullDownRefresh` 对应 `onPullDownRefresh`；`useReachBottom` 对应 `onReachBottom`。关键区别：`useEffect` 在小程序中不一定按预期触发（如 `useDidShow` 时组件可能不重新 mount），建议页面级逻辑用 Taro 钩子，组件内部用 React 钩子。

**Q8: Taro 的插件系统能做什么？有哪些常用插件？**
> Taro 插件可以：① 修改编译配置（`modifyWebpackChain`/`modifyViteConfig`）；② 注册编译钩子（`onBuildStart`/`onBuildComplete`）；③ 修改编译产物（`onBuildFinish`）；④ 注入全局依赖（`taro-plugin-inject`）；⑤ 添加新平台支持。常用插件：`taro-plugin-mini-ci`（自动上传小程序代码）、`taro-plugin-tailwind`（Tailwind CSS 支持）、`taro-plugin-html`（使用 HTML 标签）、`taro-plugin-compiler-optimization`（编译优化）、`taro-plugin-mock`（接口 Mock）。插件机制让 Taro 的编译流程完全可定制。

---

**相关链接：**
- [[uni-app跨端开发]]
- [[React核心]]
- [[React Hooks详解]]
- [[Vue核心]]
- [[Webpack与Vite]]
- [[小程序开发]]

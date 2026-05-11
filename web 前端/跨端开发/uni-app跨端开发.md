---
tags:
  - Web前端
  - uni-app
  - 跨端开发
  - 小程序
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# uni-app跨端开发

## What — 是什么

> uni-app 是 DCloud 推出的跨端开发框架，使用 Vue 语法编写一次代码，可编译到 iOS、Android、Web、微信小程序、支付宝小程序、百度小程序、抖音小程序、飞书小程序、快应用等 10+ 平台。

**核心概念：**

- **跨端编译**：一套 Vue 代码编译到多个平台，底层分别生成原生渲染或 WebView 页面
- **条件编译**：通过 `#ifdef` / `#ifndef` 语法实现平台差异化代码，编译阶段移除不相关代码
- **uni API**：统一封装各平台 API（网络、存储、定位、媒体等），抹平平台差异
- **组件体系**：内置 `<view>` `<text>` `<image>` `<scroll-view>` 等跨端组件，非 HTML 标签
- **pages.json**：全局配置文件，管理路由、TabBar、导航栏、分包等

**核心架构：**

- 设计理念：Write Once, Run Everywhere，开发者只需关注 Vue 层
- 核心模块：Vue 编译器 + 平台适配层 + uni API + 原生插件
- 数据流：Vue 代码 → 编译器转换 → 各平台中间代码 → 原生/WebView 渲染
- 编译流程：`.vue` 文件 → 解析 template/script/style → 平台适配转换 → 生成各平台目标代码

**渲染模式：**

| 渲染模式 | 原理 | 性能 | 限制 | 适用场景 |
|----------|------|------|------|----------|
| WebView | HTML5 嵌套渲染 | 一般 | 无 | 通用场景 |
| nvue（Weex） | 原生组件渲染 | 好 | CSS 子集、flex only | 高性能长列表 |
| uvue | uni-app x 原生渲染 | 极好 | 新方案生态不完善 | 追求极致性能 |

**支持平台：**

| 平台 | 标识 | 说明 |
|------|------|------|
| H5 | `H5` | 浏览器端 |
| 微信小程序 | `MP-WEIXIN` | 最成熟的小程序平台 |
| 支付宝小程序 | `MP-ALIPAY` | 支付宝生态 |
| 百度小程序 | `MP-BAIDU` | 百度搜索生态 |
| 抖音小程序 | `MP-TOUTIAO` | 字节跳动全系 |
| 飞书小程序 | `MP-LARK` | 企业协作 |
| 快应用 | `QUICKAPP-WEBVIEW` | 手机厂商联盟 |
| App iOS | `APP-PLUS` | 原生 iOS |
| App Android | `APP-PLUS` | 原生 Android |

## Why — 为什么

**适用场景：**

- 需同时覆盖多端（App + 小程序 + H5）的项目
- 中小型应用，团队以 Vue 技术栈为主
- 快速验证 MVP，多端同步上线
- 企业内部应用，不需要极致原生体验
- 已有 Vue 团队，需要快速扩展到移动端

**对比同类框架：**

| 维度 | uni-app | Taro | Flutter | React Native | 原生开发 |
|------|---------|------|---------|--------------|----------|
| 语法 | Vue | React/Vue | Dart | React | Swift/Kotlin |
| 小程序支持 | 极好（自研编译器） | 极好（京东） | 需第三方 | 不支持 | 不支持 |
| App 性能 | 中等 | 中等 | 高 | 高 | 极高 |
| 学习曲线 | 低 | 低 | 中 | 中 | 高 |
| 生态 | 丰富（插件市场） | 较好 | 丰富 | 丰富 | 最强 |
| 跨端数量 | 10+ | 5+ | 6 | 2 | 1 |
| 热更新 | 支持 | 支持 | 支持 | 支持 | 不支持 |
| 原生能力 | 原生插件 | 原生插件 | Platform Channel | Native Module | 原生 |

**优缺点：**

- ✅ 优点：
  - 真正一套代码多端运行，开发效率高
  - Vue 语法门槛低，前端团队易上手
  - 小程序支持最成熟，API 覆盖全面
  - 插件市场丰富，DCloud 生态完善
  - HBuilderX 开发工具体验好
  - 条件编译优雅处理平台差异
  - nvue 可满足高性能场景需求
- ❌ 缺点：
  - 跨端兼容问题多，各平台行为不一致
  - 性能不如原生和 Flutter，复杂交互卡顿
  - 框架黑盒，调试困难，问题定位成本高
  - 部分平台 API 有差异，需条件编译处理
  - DCloud 商业化严重，云打包有限制
  - nvue 学习成本高，CSS 限制大

## How — 怎么用

### 快速上手

```bash
# CLI 创建项目（推荐）
npx degit dcloudio/uni-preset-vue#vite-ts my-project
cd my-project
npm install
npm run dev:mp-weixin   # 微信小程序
npm run dev:h5           # H5
npm run dev:app          # App

# 或使用 HBuilderX 创建（图形化）
# 文件 → 新建 → 项目 → uni-app
```

**项目结构：**

```
my-project/
├── src/
│   ├── pages/              # 页面目录
│   │   ├── index/
│   │   │   └── index.vue
│   │   └── detail/
│   │       └── index.vue
│   ├── components/         # 公共组件
│   ├── store/              # Pinia 状态管理
│   ├── utils/              # 工具函数
│   ├── api/                # 接口请求
│   ├── static/             # 静态资源（图片、字体）
│   ├── pages.json          # 路由与全局配置
│   ├── manifest.json       # 应用配置（AppID、权限）
│   ├── App.vue             # 应用入口
│   └── main.ts             # 入口文件
├── index.html
├── vite.config.ts
└── package.json
```

### 路由与导航配置

**pages.json 详解：**

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "首页",
        "enablePullDownRefresh": true,
        "backgroundTextStyle": "dark"
      }
    },
    {
      "path": "pages/detail/index",
      "style": {
        "navigationBarTitleText": "详情",
        "navigationBarBackgroundColor": "#007AFF"
      }
    }
  ],
  "subPackages": [
    {
      "root": "pages-sub/user",
      "pages": [
        {
          "path": "profile/index",
          "style": { "navigationBarTitleText": "个人中心" }
        }
      ]
    }
  ],
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "My App",
    "navigationBarBackgroundColor": "#FFFFFF",
    "backgroundColor": "#F8F8F8"
  },
  "tabBar": {
    "color": "#999999",
    "selectedColor": "#007AFF",
    "borderStyle": "black",
    "backgroundColor": "#FFFFFF",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "static/tab/home.png",
        "selectedIconPath": "static/tab/home-active.png"
      },
      {
        "pagePath": "pages/mine/index",
        "text": "我的",
        "iconPath": "static/tab/mine.png",
        "selectedIconPath": "static/tab/mine-active.png"
      }
    ]
  },
  "preloadRule": {
    "pages/index/index": {
      "network": "all",
      "packages": ["pages-sub/user"]
    }
  },
  "easycom": {
    "autoscan": true,
    "custom": {
      "^uni-(.*)": "@dcloudio/uni-ui/lib/uni-$1/uni-$1.vue"
    }
  }
}
```

**导航 API：**

```ts
// 保留当前页，跳转到应用内的某个页面
uni.navigateTo({
  url: '/pages/detail/index?id=1&name=test',
  events: { acceptDataFromOpenedPage: (data) => console.log(data) },
  success: (res) => {
    // 向被打开页面传送数据
    res.eventChannel.emit('acceptDataFromOpenerPage', { data: 'from prev' });
  }
});

// 关闭当前页，跳转到应用内的某个页面
uni.redirectTo({ url: '/pages/login/index' });

// 关闭所有页面，打开到应用内的某个页面
uni.reLaunch({ url: '/pages/index/index' });

// 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面
uni.switchTab({ url: '/pages/index/index' });

// 关闭当前页面，返回上一页面或多级页面
uni.navigateBack({ delta: 1 });

// 获取页面传参
onLoad((options) => {
  console.log(options.id, options.name);
});

// 获取事件通道（接收上一页数据）
const eventChannel = getCurrentPages().pop().getOpenerEventChannel();
eventChannel.on('acceptDataFromOpenerPage', (data) => console.log(data));
```

### 条件编译详解

**模板条件编译：**

```vue
<template>
  <view>
    <!-- #ifdef MP-WEIXIN -->
    <button open-type="getUserInfo" @getuserinfo="onGetUserInfo">
      微信授权登录
    </button>
    <!-- #endif -->

    <!-- #ifdef MP-ALIPAY -->
    <button @click="alipayLogin">支付宝登录</button>
    <!-- #endif -->

    <!-- #ifdef H5 -->
    <button @click="h5Login">H5 账号登录</button>
    <!-- #endif -->

    <!-- #ifdef APP-PLUS -->
    <button @click="appLogin">App 一键登录</button>
    <!-- #endif -->

    <!-- #ifndef MP-ALIPAY -->
    <!-- 非支付宝小程序都执行 -->
    <view>通用内容</view>
    <!-- #endif -->

    <!-- #ifdef MP-WEIXIN || MP-ALIPAY -->
    <!-- 微信或支付宝小程序 -->
    <view>小程序通用</view>
    <!-- #endif -->
  </view>
</template>
```

**JS 条件编译：**

```js
// #ifdef MP-WEIXIN
wx.login({ success: res => console.log(res.code) });
// #endif

// #ifdef H5
window.location.href = '/login';
// #endif

// #ifdef APP-PLUS
plus.oauth.getServices(services => {
  const service = services.find(s => s.id === 'weixin');
  service.authorize(() => { /* 授权成功 */ });
});
// #endif
```

**CSS 条件编译：**

```css
/* #ifdef H5 */
.container {
  max-width: 750px;
  margin: 0 auto;
}
/* #endif */

/* #ifdef MP-WEIXIN */
.container {
  padding-bottom: env(safe-area-inset-bottom);
}
/* #endif */
```

**平台标识速查：**

| 标识 | 平台 | 标识 | 平台 |
|------|------|------|------|
| `H5` | Web | `MP-WEIXIN` | 微信小程序 |
| `MP-ALIPAY` | 支付宝 | `MP-BAIDU` | 百度 |
| `MP-TOUTIAO` | 抖音 | `MP-LARK` | 飞书 |
| `MP` | 所有小程序 | `APP-PLUS` | App |
| `APP-ANDROID` | Android | `APP-IOS` | iOS |

### 组件开发与通信

**基础页面：**

```vue
<template>
  <view class="container">
    <text class="title">{{ title }}</text>
    <button @click="handleClick">点击</button>
    <image :src="imageUrl" mode="aspectFill" />
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const title = ref('Hello uni-app');
const imageUrl = ref('/static/logo.png');

const handleClick = () => {
  uni.showToast({ title: '点击了按钮', icon: 'success' });
};
</script>

<style scoped>
.container { padding: 20rpx; }
.title { font-size: 36rpx; font-weight: bold; }
</style>
```

**父子组件通信：**

```vue
<!-- components/ProductCard.vue -->
<template>
  <view class="card" @click="handleClick">
    <image :src="product.image" mode="aspectFill" class="card-img" />
    <text class="card-title">{{ product.name }}</text>
    <text class="card-price">¥{{ product.price }}</text>
    <slot name="footer"></slot>
  </view>
</template>

<script setup lang="ts">
interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

const props = defineProps<{
  product: Product;
}>();

const emit = defineEmits<{
  (e: 'click', product: Product): void;
  (e: 'addCart', id: number): void;
}>();

const handleClick = () => {
  emit('click', props.product);
};
</script>
```

```vue
<!-- 页面中使用 -->
<template>
  <view>
    <ProductCard
      v-for="item in products"
      :key="item.id"
      :product="item"
      @click="goDetail"
      @add-cart="handleAddCart"
    >
      <template #footer>
        <button size="mini" @click.stop="handleAddCart(item.id)">加入购物车</button>
      </template>
    </ProductCard>
  </view>
</template>

<script setup lang="ts">
import ProductCard from '@/components/ProductCard.vue';

const products = ref([]);

const goDetail = (product) => {
  uni.navigateTo({ url: `/pages/detail/index?id=${product.id}` });
};

const handleAddCart = (id: number) => {
  // 加入购物车逻辑
};
</script>
```

**跨组件通信 — provide/inject：**

```ts
// 祖先组件
import { provide, ref } from 'vue';

const userInfo = ref({ name: '张三', role: 'admin' });
const updateUserInfo = (info) => { userInfo.value = info };
provide('userInfo', userInfo);
provide('updateUserInfo', updateUserInfo);
```

```ts
// 后代组件
import { inject } from 'vue';

const userInfo = inject('userInfo');
const updateUserInfo = inject('updateUserInfo');
```

**全局事件总线（跨页面）：**

```ts
// utils/eventBus.ts
class UniEventBus {
  private events: Record<string, Function[]> = {};

  on(event: string, callback: Function) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(callback);
  }

  off(event: string, callback?: Function) {
    if (!callback) { delete this.events[event]; return; }
    this.events[event] = this.events[event]?.filter(cb => cb !== callback) || [];
  }

  emit(event: string, ...args: any[]) {
    this.events[event]?.forEach(cb => cb(...args));
  }
}

export const eventBus = new UniEventBus();
```

```ts
// 页面 A — 发送事件
import { eventBus } from '@/utils/eventBus';
eventBus.emit('order:paid', { orderId: '123' });

// 页面 B — 监听事件（onShow 中订阅，onHide 中取消）
onShow(() => {
  eventBus.on('order:paid', handleOrderPaid);
});
onHide(() => {
  eventBus.off('order:paid', handleOrderPaid);
});
```

### 状态管理（Pinia）

**安装与配置：**

```ts
// main.ts
import { createSSRApp } from 'vue';
import { createPinia } from 'pinia';
import App from './App.vue';

export function createApp() {
  const app = createSSRApp(App);
  const pinia = createPinia();
  app.use(pinia);
  return { app };
}
```

**定义 Store：**

```ts
// store/user.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useUserStore = defineStore('user', () => {
  // state
  const token = ref(uni.getStorageSync('token') || '');
  const userInfo = ref<any>(null);

  // getters
  const isLoggedIn = computed(() => !!token.value);
  const userName = computed(() => userInfo.value?.name || '未登录');

  // actions
  async function login(code: string) {
    const res = await uni.request({
      url: '/api/login',
      method: 'POST',
      data: { code }
    });
    token.value = res.data.token;
    userInfo.value = res.data.user;
    uni.setStorageSync('token', res.data.token);
  }

  function logout() {
    token.value = '';
    userInfo.value = null;
    uni.removeStorageSync('token');
    uni.reLaunch({ url: '/pages/login/index' });
  }

  return { token, userInfo, isLoggedIn, userName, login, logout };
});
```

**在组件中使用：**

```vue
<script setup lang="ts">
import { useUserStore } from '@/store/user';

const userStore = useUserStore();

// 响应式使用
const userName = computed(() => userStore.userName);
const isLoggedIn = computed(() => userStore.isLoggedIn);

// 调用 action
const handleLogin = async () => {
  // #ifdef MP-WEIXIN
  const { code } = await wx.login();
  await userStore.login(code);
  // #endif
};
</script>
```

### 网络请求封装

```ts
// utils/request.ts
const BASE_URL = import.meta.env.VITE_API_BASE;

interface RequestOptions {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
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
  if (loadingCount === 0) uni.showLoading({ title: '加载中', mask: true });
  loadingCount++;
}

function hideLoading() {
  loadingCount--;
  if (loadingCount <= 0) { loadingCount = 0; uni.hideLoading(); }
}

export function request<T = any>(options: RequestOptions): Promise<T> {
  const { showLoading: show = false } = options;
  if (show) showLoading();

  return new Promise((resolve, reject) => {
    uni.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data,
      header: {
        'Authorization': `Bearer ${uni.getStorageSync('token')}`,
        'Content-Type': 'application/json',
        ...options.header,
      },
      success: (res) => {
        if (show) hideLoading();
        const data = res.data as ApiResponse<T>;

        if (res.statusCode === 200 && data.code === 0) {
          resolve(data.data);
        } else if (res.statusCode === 401) {
          uni.removeStorageSync('token');
          uni.reLaunch({ url: '/pages/login/index' });
          reject(new Error('登录过期'));
        } else {
          uni.showToast({ title: data.message || '请求失败', icon: 'none' });
          reject(new Error(data.message));
        }
      },
      fail: (err) => {
        if (show) hideLoading();
        uni.showToast({ title: '网络异常', icon: 'none' });
        reject(err);
      },
    });
  });
}

// 便捷方法
export const get = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'GET', data });

export const post = <T = any>(url: string, data?: any, show = false) =>
  request<T>({ url, method: 'POST', data, showLoading: show });

export const put = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'PUT', data });

export const del = <T = any>(url: string, data?: any) =>
  request<T>({ url, method: 'DELETE', data });
```

**文件上传：**

```ts
// utils/upload.ts
export function uploadFile(filePath: string, name = 'file') {
  return new Promise((resolve, reject) => {
    const uploadTask = uni.uploadFile({
      url: `${BASE_URL}/api/upload`,
      filePath,
      name,
      header: {
        'Authorization': `Bearer ${uni.getStorageSync('token')}`,
      },
      success: (res) => {
        const data = JSON.parse(res.data);
        if (data.code === 0) resolve(data.data);
        else reject(new Error(data.message));
      },
      fail: reject,
    });

    // 上传进度
    uploadTask.onProgressUpdate((res) => {
      console.log(`上传进度: ${res.progress}%`);
    });
  });
}
```

### 页面生命周期

```vue
<script setup lang="ts">
// === Vue 生命周期 ===
// H5 全部生效，小程序中部分生效
import { onMounted, onUnmounted } from 'vue';

onMounted(() => {
  console.log('组件挂载完成');
});

onUnmounted(() => {
  console.log('组件卸载'); // 小程序中不一定触发
});

// === uni-app 页面生命周期 ===

// 页面加载（只触发一次，可获取路由参数）
onLoad((options) => {
  console.log('页面加载', options);
});

// 页面显示（每次进入页面都触发）
onShow(() => {
  console.log('页面显示');
});

// 页面就绪（初次渲染完成，可获取节点信息）
onReady(() => {
  console.log('页面就绪，可操作 DOM');
  // 获取节点信息
  uni.createSelectorQuery()
    .select('.title')
    .boundingClientRect((rect) => {
      console.log(rect.width, rect.height);
    })
    .exec();
});

// 页面隐藏
onHide(() => {
  console.log('页面隐藏');
});

// 页面卸载
onUnload(() => {
  console.log('页面卸载');
});

// 下拉刷新
onPullDownRefresh(() => {
  console.log('下拉刷新');
  fetchData().finally(() => uni.stopPullDownRefresh());
});

// 触底加载
onReachBottom(() => {
  console.log('触底加载更多');
  loadMore();
});

// 页面滚动
onPageScroll((e) => {
  console.log('滚动距离', e.scrollTop);
});

// 分享（微信小程序）
onShareAppMessage(() => {
  return {
    title: '分享标题',
    path: '/pages/index/index',
    imageUrl: '/static/share.png',
  };
});

// 分享到朋友圈
onShareTimeline(() => {
  return {
    title: '分享标题',
    query: 'id=1',
    imageUrl: '/static/share.png',
  };
});

// 监听返回按钮
onBackPress(() => {
  // 返回 true 阻止返回
  if (hasUnsavedChanges) {
    uni.showModal({
      title: '提示',
      content: '有未保存的更改，确定退出？',
      success: (res) => { if (res.confirm) return false; }
    });
    return true;
  }
  return false;
});
</script>
```

### uni API 常用操作

**导航与路由：**

```ts
uni.navigateTo({ url: '/pages/detail/index?id=1' });
uni.redirectTo({ url: '/pages/login/index' });
uni.switchTab({ url: '/pages/home/index' });
uni.navigateBack({ delta: 1 });
uni.reLaunch({ url: '/pages/index/index' });
```

**本地存储：**

```ts
// 同步
uni.setStorageSync('token', 'xxx');
const token = uni.getStorageSync('token');
uni.removeStorageSync('token');
uni.clearStorageSync();

// 异步
await uni.setStorage({ key: 'token', data: 'xxx' });
const { data } = await uni.getStorage({ key: 'token' });

// 复杂对象
uni.setStorageSync('userInfo', JSON.stringify({ name: '张三', age: 25 }));
const userInfo = JSON.parse(uni.getStorageSync('userInfo') || '{}');
```

**交互反馈：**

```ts
// Toast
uni.showToast({ title: '操作成功', icon: 'success', duration: 2000 });
uni.showToast({ title: '请先登录', icon: 'none' }); // 纯文字

// Loading
uni.showLoading({ title: '加载中', mask: true });
uni.hideLoading();

// Modal
uni.showModal({
  title: '确认删除？',
  content: '删除后不可恢复',
  confirmColor: '#FF4D4F',
  success: (res) => {
    if (res.confirm) { /* 确认 */ }
  }
});

// ActionSheet
uni.showActionSheet({
  itemList: ['拍照', '从相册选择'],
  success: (res) => {
    if (res.tapIndex === 0) { /* 拍照 */ }
  }
});
```

**媒体操作：**

```ts
// 选择图片
uni.chooseImage({
  count: 9,
  sizeType: ['compressed'],
  sourceType: ['album', 'camera'],
  success: (res) => {
    const tempFilePaths = res.tempFilePaths;
  }
});

// 预览图片
uni.previewImage({
  current: 0,
  urls: ['https://img1.jpg', 'https://img2.jpg'],
});

// 选择视频
uni.chooseVideo({
  sourceType: ['album', 'camera'],
  maxDuration: 60,
  success: (res) => {
    console.log(res.tempFilePath, res.duration);
  }
});
```

**设备与位置：**

```ts
// 获取系统信息
const systemInfo = uni.getSystemInfoSync();
console.log(systemInfo.platform, systemInfo.windowWidth, systemInfo.statusBarHeight);

// 获取位置
uni.getLocation({
  type: 'gcj02',
  success: (res) => {
    console.log(res.latitude, res.longitude);
  }
});

// 扫码
uni.scanCode({
  scanType: ['qrCode', 'barCode'],
  success: (res) => {
    console.log(res.result);
  }
});

// 打电话
uni.makePhoneCall({ phoneNumber: '10086' });

// 剪贴板
uni.setClipboardData({ data: '复制内容' });
uni.getClipboardData({
  success: (res) => console.log(res.data)
});
```

### 小程序登录与支付

**微信登录流程：**

```ts
// api/auth.ts
export async function wxLogin() {
  // 1. 调用 wx.login 获取 code
  const { code } = await new Promise<{ code: string }>((resolve) => {
    wx.login({ success: resolve });
  });

  // 2. 发送 code 到后端换取 openid 和 token
  const res = await post<{ token: string; openid: string }>('/auth/wx-login', { code });

  // 3. 存储登录态
  uni.setStorageSync('token', res.token);

  return res;
}
```

**微信支付流程：**

```ts
// api/payment.ts
export async function wxPay(orderId: string) {
  // 1. 后端创建预付单，返回支付参数
  const payParams = await post<{
    timeStamp: string;
    nonceStr: string;
    package: string;
    signType: string;
    paySign: string;
  }>('/pay/create', { orderId });

  // 2. 调起微信支付
  return new Promise((resolve, reject) => {
    uni.requestPayment({
      provider: 'wxpay',
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

### nvue 原生渲染

**与 vue 的关键区别：**

```vue
<!-- nvue 页面：只能用 flex 布局，不支持部分 CSS -->
<template>
  <view class="container">
    <list class="list" @loadmore="loadMore">
      <cell v-for="item in list" :key="item.id">
        <text class="item-text">{{ item.name }}</text>
      </cell>
    </list>
  </view>
</template>

<style>
/* nvue 样式限制 */
.container {
  flex: 1;               /* 必须 flex 布局 */
  flex-direction: column; /* 默认 column，vue 默认 row */
}
.list {
  flex: 1;
}
.item-text {
  font-size: 32px;       /* nvue 用 px，不用 rpx */
  color: #333333;
  lines: 2;              /* nvue 特有：限制行数 */
  text-overflow: ellipsis;
}
</style>
```

**nvue 注意事项：**

| 项目 | vue | nvue |
|------|-----|------|
| 默认 flex 方向 | row | column |
| 尺寸单位 | rpx/px | px |
| CSS 选择器 | 支持复杂选择器 | 只支持 class |
| 布局方式 | 任意 | 只能 flex |
| 动画 | CSS transition/animation | weex animation 模块 |
| 列表 | scroll-view | list/recycle-list（原生回收） |
| DOM 操作 | uni.createSelectorQuery | weex DOM 模块 |

**nvue 动画：**

```ts
// nvue 中使用 animation 模块
const animation = weex.requireModule('animation');

function animate(el, options) {
  animation.transition(el, {
    styles: { transform: 'translateX(100px)', opacity: 0.5 },
    duration: 300,
    timingFunction: 'ease-in-out',
    delay: 0,
  }, () => {
    // 动画完成回调
  });
}
```

### 原生插件

**使用原生插件：**

```json
// manifest.json
{
  "app-plus": {
    "distribute": {
      "sdkConfigs": {
        "payment": {
          "weixin": {
            "appid": "wx...",
            "UniversalLinks": ""
          }
        }
      }
    },
    "nativePlugins": {
      "TencentMap": {
        "apikey_ios": "...",
        "apikey_android": "..."
      }
    }
  }
}
```

```ts
// 使用原生插件
// #ifdef APP-PLUS
const tencentMap = plus.requireNativePlugin('TencentMap');
tencentMap.getLocation({ type: 'gcj02' }, (res) => {
  console.log(res.latitude, res.longitude);
});
// #endif
```

**UniPush 推送：**

```ts
// App.vue
onLaunch(() => {
  // #ifdef APP-PLUS
  const push = uni.requireNativePlugin('UniPush');
  push.createPushMessage({
    title: '新消息',
    content: '您有一条新消息',
    payload: { type: 'chat', id: '123' },
  });

  // 监听推送点击
  plus.push.addEventListener('click', (msg) => {
    const payload = JSON.parse(msg.payload);
    uni.navigateTo({ url: `/pages/chat/index?id=${payload.id}` });
  });
  // #endif
});
```

### 分包与性能优化

**分包配置：**

```json
{
  "subPackages": [
    {
      "root": "pages-sub/order",
      "pages": [
        { "path": "list/index" },
        { "path": "detail/index" }
      ]
    },
    {
      "root": "pages-sub/user",
      "pages": [
        { "path": "profile/index" },
        { "path": "settings/index" }
      ]
    }
  ],
  "preloadRule": {
    "pages/index/index": {
      "network": "all",
      "packages": ["pages-sub/order"]
    }
  }
}
```

**分包限制：**

| 平台 | 主包限制 | 单个子包 | 总包 |
|------|---------|---------|------|
| 微信 | 2MB | 2MB | 20MB |
| 支付宝 | 2MB | 2MB | 20MB |
| 抖音 | 2MB | 2MB | 20MB |
| 百度 | 2MB | 2MB | 16MB |

**性能优化清单：**

```json
// manifest.json — 开启优化选项
{
  "mp-weixin": {
    "optimization": {
      "subPackages": true
    },
    "usingComponents": true
  }
}
```

```ts
// vite.config.ts — 分包优化
export default defineConfig({
  build: {
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor': ['vue', 'pinia'],
          'utils': ['lodash-es', 'dayjs'],
        }
      }
    }
  }
});
```

**长列表优化（nvue recycle-list）：**

```vue
<template>
  <list class="list" @loadmore="loadMore" loadmoreoffset="50">
    <cell v-for="item in list" :key="item.id" :ref="'item-' + item.id">
      <view class="item">
        <image :src="item.avatar" class="avatar" />
        <text class="name">{{ item.name }}</text>
      </view>
    </cell>
    <loading @loading="onLoading" class="loading">
      <text>加载中...</text>
    </loading>
  </list>
</template>
```

**图片优化：**

```ts
// 图片压缩后上传
function compressImage(filePath: string): Promise<string> {
  return new Promise((resolve) => {
    // #ifdef MP-WEIXIN
    wx.compressImage({
      src: filePath,
      quality: 80,
      success: (res) => resolve(res.tempFilePath),
      fail: () => resolve(filePath),
    });
    // #endif
    // #ifdef H5
    resolve(filePath); // H5 用 canvas 压缩
    // #endif
  });
}
```

### 发布与打包

**各平台发布流程：**

| 平台 | 开发工具 | 发布方式 | 审核周期 |
|------|---------|---------|---------|
| 微信小程序 | 微信开发者工具 | 上传代码 → 提交审核 | 1-7天 |
| H5 | 浏览器 | 部署到服务器 | 无 |
| App | HBuilderX | 云打包/本地打包 | 应用商店审核 |
| 支付宝 | 支付宝 IDE | 上传 → 提交审核 | 1-3天 |

**微信小程序发布：**

```bash
# 1. 构建
npm run build:mp-weixin

# 2. 打开微信开发者工具
# 导入 dist/build/mp-weixin 目录

# 3. 在开发者工具中：上传 → 填写版本号和备注 → 提交审核
```

**App 打包配置：**

```json
// manifest.json
{
  "name": "MyApp",
  "appid": "__UNI__XXXXXX",
  "versionName": "1.0.0",
  "versionCode": "100",
  "app-plus": {
    "distribute": {
      "android": {
        "packagename": "com.example.myapp",
        "keystore": "android.keystore",
        "password": "xxx",
        "aliasname": "myapp",
        "schemes": "myapp"
      },
      "ios": {
        "appid": "com.example.myapp",
        "mobileprovision": "xxx.mobileprovision",
        "password": "xxx",
        "p12": "xxx.p12",
        "devices": "universal"
      }
    }
  }
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| rpx 在不同设备显示不一致 | rpx 基于屏幕宽度 750rpx，窄屏设备可能过小 | 关键尺寸用 px，整体布局用 rpx |
| v-show 在小程序不生效 | 小程序组件不支持 display 切换 | 用 v-if 替代 v-show |
| 部分小程序组件样式不生效 | 小程序组件有 shadow DOM 隔离 | 使用 `virtualHost` 或调整样式穿透 |
| nvue 样式限制多 | Weex 只支持部分 CSS | 只用 flex 布局，避免复杂选择器 |
| 包体积过大 | 整包超过 2MB 无法上传微信 | 分包加载，配置 subPackages |
| 图片路径在 H5 和小程序不同 | 静态资源路径引用方式不同 | 用绝对路径 `/static/xxx` 或 import 引入 |
| map/video 组件层级最高遮挡弹窗 | 原生组件层级问题 | 使用 `cover-view` 覆盖或 `subNVues` |
| 页面栈超过 10 层无法跳转 | 小程序页面栈限制 10 层 | 用 redirectTo 替代 navigateTo 或预加载 |
| input 聚焦时页面滚动 | 小程序 input 聚焦自动推页 | 设置 `adjust-position="false"` 手动调整 |
| setData 性能差 | 频繁调用 setData 导致渲染卡顿 | 合并更新、只传变化数据、使用 virtualHost |
| 自定义组件样式不生效 | 样式隔离 | 组件 options 中设置 `styleIsolation: 'shared'` |
| CSS 动画在小程序闪烁 | 小程序渲染机制不同 | 用 `transform` 替代 `top/left`，开启 GPU 加速 |
| wxs 与 JS 数据不能直接共享 | wxs 运行在独立线程 | 用 `CallFunction` 或 `ComponentInstance` 通信 |

### 最佳实践

- 优先使用 rpx 做响应式布局，关键间距用 px 保证精度
- 复杂页面用 v-if 做懒加载，避免一次性渲染过多节点
- 图片使用 CDN + 压缩 + 懒加载，避免包体积过大
- 小程序必须做分包，主包控制在 2MB 以内
- 网络请求统一封装，处理 token 过期和错误重试
- 善用条件编译处理平台差异，但避免过度使用导致代码碎片化
- 使用 Pinia 做全局状态管理，避免 props 层层传递
- 长列表用 nvue 的 list/recycle-list 实现原生回收
- 页面间大数据传递用 Store 或 Storage，避免 URL 传参超限
- 及时销毁定时器和事件监听，避免内存泄漏
- 优先使用 easycom 自动导入组件，减少手动 import

## 面试题

**Q1: uni-app 的条件编译是怎么实现的？和 CSS 媒体查询有什么区别？**
> 条件编译通过 `#ifdef` / `#ifndef` 注释语法在**编译阶段**决定代码是否包含，不满足条件的代码会被完全移除，零运行时开销。CSS 媒体查询是**运行时**判断，所有代码都包含在产物中。条件编译处理的是平台级差异（API 有无、组件差异），媒体查询处理的是同一平台下的屏幕适配。

**Q2: uni-app 的 rpx 和 px 有什么区别？为什么推荐用 rpx？**
> rpx 是响应式像素，规定屏幕宽度固定为 750rpx，框架根据实际屏幕宽度自动换算（1rpx = 屏幕宽度/750px）。px 是固定像素，在不同设备上显示大小不一致。rpx 让同一套代码在不同屏幕宽度下等比缩放，适合布局；精确尺寸（如 1px 边框）用 px。

**Q3: uni-app 中 v-if 和 v-show 的区别？为什么小程序中 v-show 可能不生效？**
> v-if 条件为 false 时组件不渲染（销毁/重建），v-show 条件为 false 时组件渲染但 `display: none` 隐藏。小程序中部分原生组件（如 video、map）不支持 CSS 的 display 切换，v-show 无法隐藏它们。此外小程序组件有 shadow DOM 隔离，display 设置可能被组件内部覆盖。建议跨端统一使用 v-if。

**Q4: uni-app 小程序分包怎么做？有哪些限制？**
> 在 `pages.json` 中配置 `subPackages`，将不同功能模块的页面分到不同子包。限制：主包不超过 2MB，单个子包不超过 2MB，总包不超过 20MB（微信）；子包不能引用其他子包的资源，只能引用主包；TabBar 页面必须在主包。建议按功能模块划分分包，公共资源放主包，首屏不需要的页面放子包。

**Q5: uni-app 和 Taro 的核心区别是什么？如何选择？**
> uni-app 基于 Vue 语法，Taro 最初基于 React（后支持 Vue）；uni-app 由 DCloud 维护，小程序支持更成熟（自研编译器），Taro 由京东维护，React 生态更好。选择：Vue 团队 + 小程序为主 → uni-app；React 团队 + 更强类型推导 → Taro；需要极致性能和原生体验 → Flutter/React Native。

**Q6: nvue 和 vue 页面有什么区别？什么时候该用 nvue？**
> nvue 基于 Weex 原生渲染，默认 flex-direction 为 column，只能用 class 选择器和 flex 布局，不支持 rpx，动画用 weex animation 模块。vue 页面基于 WebView 渲染，支持完整 CSS。nvue 适用于：长列表（list/recycle-list 原生回收）、高性能动画、复杂视频/地图覆盖场景。普通页面用 vue 即可，只在性能瓶颈时切换 nvue。

**Q7: uni-app 中如何处理跨端样式差异？**
> 三种方式：① CSS 条件编译 `/* #ifdef MP-WEIXIN */` 按平台写不同样式；② 使用兼容性好的 CSS 属性（flex 布局、rpx 单位）；③ 全局样式 reset 统一基线。注意：避免使用各平台特有属性，小程序不支持 `*` 选择器，nvue 不支持简写属性（如 `border: 1px solid #ccc` 需拆成 `border-width/border-style/border-color`）。

**Q8: uni-app 的页面生命周期和 Vue 组件生命周期有什么关系？**
> 页面生命周期是 uni-app 在小程序生命周期上的封装，与 Vue 生命周期独立运行。执行顺序：`onLoad` → `onShow` → `onReady`（对应 mounted）。关键差异：`onShow/onHide` 每次页面显示/隐藏都触发，而 mounted 只触发一次；`onUnload` 在页面销毁时触发，unmounted 在小程序中可能不触发。建议：页面级逻辑用 uni 生命周期，组件内部逻辑用 Vue 生命周期。

---

**相关链接：**
- [[Vue核心]]
- [[Vue3响应式原理]]
- [[Vue生态Router与Pinia]]
- [[响应式设计]]
- [[Webpack与Vite]]

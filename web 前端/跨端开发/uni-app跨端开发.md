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

> uni-app 是 DCloud 推出的跨端开发框架，使用 Vue 语法编写一次代码，可编译到 iOS、Android、Web、微信小程序、支付宝小程序等 10+ 平台。

**核心概念：**

- **跨端编译**：一套 Vue 代码编译到多个平台，底层分别生成原生渲染或 WebView 页面
- **条件编译**：通过 `#ifdef` / `#ifndef` 语法实现平台差异化代码
- **uni API**：统一封装各平台 API（网络、存储、定位等），抹平平台差异
- **组件体系**：内置 `<view>` `<text>` `<image>` 等跨端组件，非 HTML 标签

**核心架构：**

- 设计理念：Write Once, Run Everywhere，开发者只需关注 Vue 层
- 核心模块：Vue 编译器 + 平台适配层 + uni API + 原生插件
- 数据流：Vue 代码 → 编译器转换 → 各平台中间代码 → 原生/WebView 渲染

**渲染模式：**

- **WebView 渲染**：老方案，HTML5 WebView 套壳，性能一般
- **nvue（Weex）**：原生渲染，使用 CSS 子集，性能好但限制多
- **uvue**：uni-app x 新方案，基于原生渲染，接近 Flutter 性能

## Why — 为什么

**适用场景：**

- 需同时覆盖多端（App + 小程序 + H5）的项目
- 中小型应用，团队以 Vue 技术栈为主
- 快速验证 MVP，多端同步上线
- 企业内部应用，不需要极致原生体验

**对比同类框架：**

| 维度 | uni-app | Taro | Flutter | React Native | 原生开发 |
|------|---------|------|---------|--------------|----------|
| 语法 | Vue | React/Vue | Dart | React | Swift/Kotlin |
| 小程序支持 | 极好（自研） | 极好（京东） | 需第三方 | 不支持 | 不支持 |
| App 性能 | 中等 | 中等 | 高 | 高 | 极高 |
| 学习曲线 | 低 | 低 | 中 | 中 | 高 |
| 生态 | 丰富（插件市场） | 较好 | 丰富 | 丰富 | 最强 |
| 跨端数量 | 10+ | 5+ | 6 | 2 | 1 |

**优缺点：**

- ✅ 优点：
  - 真正一套代码多端运行，开发效率高
  - Vue 语法门槛低，前端团队易上手
  - 小程序支持最成熟，API 覆盖全面
  - 插件市场丰富，DCloud 生态完善
  - HBuilderX 开发工具体验好
- ❌ 缺点：
  - 跨端兼容问题多，各平台行为不一致
  - 性能不如原生和 Flutter，复杂交互卡顿
  - 框架黑盒，调试困难，问题定位成本高
  - 部分平台 API 有差异，需条件编译处理
  - DCloud 商业化严重，云打包有限制

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
```

### 代码示例

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
.container {
  padding: 20rpx;
}
.title {
  font-size: 36rpx;
  font-weight: bold;
}
</style>
```

**条件编译：**

```vue
<template>
  <view>
    <!-- #ifdef MP-WEIXIN -->
    <button open-type="getUserInfo" @getuserinfo="onGetUserInfo">
      微信授权登录
    </button>
    <!-- #endif -->

    <!-- #ifdef H5 -->
    <button @click="h5Login">H5 账号登录</button>
    <!-- #endif -->

    <!-- #ifdef APP-PLUS -->
    <button @click="appLogin">App 一键登录</button>
    <!-- #endif -->
  </view>
</template>
```

**条件编译 JS：**

```js
// #ifdef MP-WEIXIN
wx.login({ success: res => console.log(res.code) });
// #endif

// #ifdef H5
window.location.href = '/login';
// #endif

// #ifndef MP-ALIPAY
// 非支付宝小程序都执行
uni.setStorageSync('key', 'value');
// #endif
```

**网络请求封装：**

```ts
// utils/request.ts
const BASE_URL = import.meta.env.VITE_API_BASE;

interface RequestOptions {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  data?: Record<string, any>;
}

export function request<T = any>(options: RequestOptions): Promise<T> {
  return new Promise((resolve, reject) => {
    uni.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data,
      header: {
        'Authorization': `Bearer ${uni.getStorageSync('token')}`,
        'Content-Type': 'application/json',
      },
      success: (res) => {
        if (res.statusCode === 200) {
          resolve(res.data as T);
        } else if (res.statusCode === 401) {
          uni.navigateTo({ url: '/pages/login/index' });
          reject(new Error('未授权'));
        } else {
          reject(new Error(`请求失败: ${res.statusCode}`));
        }
      },
      fail: (err) => reject(err),
    });
  });
}
```

**页面生命周期：**

```vue
<script setup lang="ts">
import { onMounted } from 'vue';

// Vue 生命周期（H5 有效，小程序部分有效）
onMounted(() => {
  console.log('mounted');
});

// uni-app 页面生命周期
onLoad((options) => {
  console.log('页面加载', options); // 获取路由参数
});

onShow(() => {
  console.log('页面显示');
});

onHide(() => {
  console.log('页面隐藏');
});

onPullDownRefresh(() => {
  console.log('下拉刷新');
  // 处理完数据后停止刷新
  uni.stopPullDownRefresh();
});

onReachBottom(() => {
  console.log('触底加载更多');
});
</script>
```

**uni API 常用操作：**

```ts
// 导航
uni.navigateTo({ url: '/pages/detail/index?id=1' });   // 保留当前页
uni.redirectTo({ url: '/pages/login/index' });          // 关闭当前页
uni.switchTab({ url: '/pages/home/index' });            // 切换 TabBar 页
uni.navigateBack({ delta: 1 });                          // 返回上一页

// 存储
uni.setStorageSync('token', 'xxx');
const token = uni.getStorageSync('token');
uni.removeStorageSync('token');

// 提示
uni.showToast({ title: '操作成功', icon: 'success' });
uni.showLoading({ title: '加载中' });
uni.hideLoading();
uni.showModal({
  title: '确认删除？',
  content: '删除后不可恢复',
  success: (res) => {
    if (res.confirm) { /* 确认 */ }
  }
});

// 选择图片
uni.chooseImage({
  count: 1,
  sizeType: ['compressed'],
  sourceType: ['album', 'camera'],
  success: (res) => {
    const tempFilePath = res.tempFilePaths[0];
  }
});
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
| map 组件层级最高遮挡弹窗 | 原生组件层级问题 | 使用 `cover-view` 覆盖或 `subNVues` |
| 页面栈超过 10 层无法跳转 | 小程序页面栈限制 10 层 | 用 redirectTo 替代 navigateTo 或预加载 |

### 最佳实践

- 优先使用 rpx 做响应式布局，关键间距用 px 保证精度
- 复杂页面用 v-if 做懒加载，避免一次性渲染过多节点
- 图片使用 CDN + 压缩 + 懒加载，避免包体积过大
- 小程序必须做分包，主包控制在 2MB 以内
- 网络请求统一封装，处理 token 过期和错误重试
- 善用条件编译处理平台差异，但避免过度使用导致代码碎片化
- 使用 Pinia 做全局状态管理，避免 props 层层传递

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

---

**相关链接：**
- [[Vue核心]]
- [[Vue3响应式原理]]
- [[响应式设计]]
- [[Webpack与Vite]]

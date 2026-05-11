---
tags:
  - Web前端
  - React Native
  - 移动开发
  - 跨端
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# React Native移动开发

## What — 是什么

> React Native（RN）是 Meta（Facebook）推出的跨平台移动开发框架，使用 React 语法和 JavaScript/TypeScript 编写原生移动应用，通过桥接机制将 JS 逻辑映射到 iOS/Android 原生组件渲染，而非 WebView 嵌套。Instagram、Discord、Shopify、Pinterest 等应用均使用 React Native。

**核心概念：**

- **原生组件映射**：JS 中的 `<View>` `<Text>` 映射到 iOS 的 `UIView`/`UILabel` 和 Android 的 `View`/`TextView`，非 WebView 渲染
- **Bridge 桥接**：JS 线程与原生线程通过异步 JSON 消息通信，是旧架构的核心机制
- **JSI（JavaScript Interface）**：新架构中替代 Bridge 的同步通信层，JS 直接持有 C++ 对象引用
- **Fabric**：新架构的渲染系统，支持同步布局计算和优先级调度
- **TurboModules**：新架构的模块系统，按需加载原生模块，替代旧 Bridge 模块
- **Codegen**：根据 TypeScript 规范自动生成 C++/原生代码的类型安全胶水层

**核心架构：**

```
┌────────────────────────────────────────────────┐
│               React Native App                 │
├──────────────┬─────────────────────────────────┤
│  JS Runtime  │         Native Side              │
│  (Hermes)    │  ┌────────┐ ┌────────────────┐  │
│  React Tree  │  │ Fabric │ │ TurboModules   │  │
│  Hooks/Nav   │  │ Shadow │ │ Camera│SQLite  │  │
│  State       │  │ Tree   │ │ Geo │ FS │ ... │  │
├──────────────┼──┴────────┴─┴────────────────┤  │
│              │       JSI (C++ Bridge)        │  │
│              │  同步调用 · 直接引用 · 零序列化  │  │
├──────────────┴────────────────────────────────┤
│              Platform APIs                     │
│          iOS (UIKit) │ Android (View)          │
└────────────────────────────────────────────────┘
```

- 设计理念：Learn Once, Write Anywhere — 学一次 React，写多端原生应用
- 核心模块：Hermes 引擎 + React 渲染器 + Fabric + TurboModules + Codegen
- 数据流：JS 编写 React 组件 → 虚拟 DOM diff → JSI 同步通信 → Fabric 更新 Shadow Tree → 原生 UI 渲染

**架构演进：**

| 对比项 | 旧架构（Bridge） | 新架构（Fabric + JSI + TurboModules） |
|--------|------------------|---------------------------------------|
| JS 与原生通信 | 异步 JSON 序列化 | JSI 同步直接引用 |
| 渲染 | 异步批量更新 | Fabric 同步优先级调度 |
| 模块加载 | 启动时全量加载 | TurboModules 按需加载 |
| 类型安全 | 手动 PropTypes/Flow | Codegen 自动生成类型 |
| 事件处理 | 异步回调 | 同步刷新（如滚动事件） |
| 性能 | Bridge 瓶颈，大量数据卡顿 | 接近原生，无序列化开销 |
| 交互体验 | 滚动/手势有延迟 | 同步渲染，60fps 流畅 |

**与 WebView 混合开发区别：**

| 维度 | React Native | WebView 混合开发 |
|------|-------------|-----------------|
| 渲染方式 | 原生组件渲染 | HTML5 在 WebView 中渲染 |
| 性能 | 接近原生 | 受 WebView 限制 |
| 动画 | 原生动画驱动 | CSS 动画，有卡顿 |
| 原生能力 | 直接调用原生 API | 需 JSBridge 桥接 |
| 开发体验 | 热更新，组件化 | 传统 Web 开发 |
| 复杂度 | 需理解原生概念 | 纯前端思维 |

## Why — 为什么

**适用场景：**

- 已有 React/Web 团队，需要快速扩展到移动端
- 需要同时覆盖 iOS 和 Android 的中大型应用
- 对性能有要求但不想分别维护两套原生代码
- 电商、社交、内容类应用（列表/表单/图片为主）
- 需要热更新能力，快速迭代发布
- MVP 验证阶段，需要快速上线多端

**对比同类框架：**

| 维度 | React Native | Flutter | uni-app | Kotlin/Swift |
|------|-------------|---------|---------|-------------|
| 开发语言 | JavaScript/TS | Dart | Vue/JS/TS | Kotlin/Swift |
| 渲染方式 | 原生组件 | 自绘引擎(Skia) | WebView/原生 | 原生 |
| 性能 | 高 | 极高 | 中等 | 极高 |
| 学习曲线 | 中（需 React 基础） | 中（需学 Dart） | 低 | 高 |
| 生态 | 丰富（npm + 原生库） | 丰富（pub.dev） | 一般（插件市场） | 最强（平台原生） |
| 跨端数量 | iOS + Android + Web | iOS + Android + Web + 桌面 | 10+（含小程序） | 各1 |
| 热更新 | 支持（CodePush） | 不支持 | 支持 | 不支持 |
| 原生能力 | Native Module/TurboModule | Platform Channel | 原生插件 | 原生 |
| 社区 | Meta 维护，庞大 | Google 维护，增长快 | DCloud，国内为主 | 官方 |
| 包体积 | 较大（Hermes ~3MB） | 较大（Skia ~5MB） | 小 | 最小 |
| 动画 | Reanimated 3（流畅） | 内置强大动画 | 有限 | 原生 |

**优缺点：**

- ✅ 优点：
  - 使用 React 语法，Web 开发者上手快
  - 真正的原生渲染，性能远超 WebView
  - 社区庞大，第三方库丰富
  - 支持热更新，无需应用商店审核即可更新 JS
  - 新架构（Fabric + JSI）性能接近原生
  - TypeScript 支持完善，开发体验好
  - Expo 生态降低原生开发门槛
- ❌ 缺点：
  - 仍需了解 iOS/Android 原生知识处理复杂问题
  - 新架构迁移成本高，部分老库未适配
  - 包体积比原生大（Hermes 引擎约 3MB）
  - 长列表极度复杂场景不如原生和 Flutter
  - iOS 发布流程仍需 Mac 环境
  - 某些原生功能需要写 Native Module

## How — 怎么用

### 1. 项目搭建

**Expo vs React Native CLI：**

| 对比项 | Expo | React Native CLI |
|--------|------|-----------------|
| 上手难度 | 低，零原生配置 | 高，需配置原生环境 |
| 原生模块 | Expo SDK 覆盖大部分 | 可自由集成任意原生库 |
| 自定义原生代码 | 需 Config Plugin 或 EAS | 直接修改原生代码 |
| 构建 | EAS Build 云构建 | 本地 Xcode/Android Studio |
| 适用场景 | 中小型应用、快速原型 | 大型应用、深度原生定制 |
| 热更新 | EAS Update | CodePush |

```bash
# Expo 创建（推荐）
npx create-expo-app@latest my-app --template tabs
cd my-app && npx expo start

# RN CLI 创建
npx react-native@latest init MyApp --template react-native-template-typescript
cd ios && pod install && cd ..  # Mac 需执行
npx react-native run-ios / run-android
```

**项目结构（Expo Router）：**

```
my-app/
├── app/                    # 路由页面（文件路由）
│   ├── _layout.tsx         # 根布局
│   ├── index.tsx           # 首页
│   ├── (tabs)/             # Tab 分组
│   │   ├── _layout.tsx     # Tab 布局
│   │   ├── home.tsx
│   │   └── profile.tsx
│   └── user/[id].tsx       # 动态路由
├── components/             # 公共组件
├── hooks/                  # 自定义 Hooks
├── store/                  # 状态管理
├── services/               # API 服务
├── utils/                  # 工具函数
├── app.json                # Expo 配置
└── package.json
```

```tsx
// app/_layout.tsx — Expo Router 根布局
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack screenOptions={{
      headerStyle: { backgroundColor: '#007AFF' },
      headerTintColor: '#fff',
    }}>
      <Stack.Screen name="index" options={{ title: '首页' }} />
      <Stack.Screen name="user/[id]" options={{ title: '用户详情' }} />
      <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
    </Stack>
  );
}
```

### 2. 核心组件

```tsx
import {
  View, Text, Image, FlatList, TouchableOpacity,
  TextInput, StyleSheet, SafeAreaView, ActivityIndicator, RefreshControl,
} from 'react-native';

// 基础页面
export default function HomeScreen() {
  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>React Native</Text>
      <Image source={{ uri: 'https://example.com/logo.png' }} style={styles.logo} resizeMode="contain" />
      <TouchableOpacity style={styles.button} onPress={() => console.log('pressed')} activeOpacity={0.7}>
        <Text style={styles.buttonText}>点击按钮</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// FlatList 长列表
function ProductList({ products }: { products: Product[] }) {
  const [refreshing, setRefreshing] = useState(false);

  const renderItem = ({ item }: { item: Product }) => (
    <View style={styles.productCard}>
      <Image source={{ uri: item.image }} style={styles.productImage} />
      <Text>{item.name}</Text>
      <Text>¥{item.price}</Text>
    </View>
  );

  return (
    <FlatList
      data={products}
      renderItem={renderItem}
      keyExtractor={(item) => item.id}
      numColumns={2}
      refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading ? <ActivityIndicator size="small" /> : null}
    />
  );
}

// TextInput 表单
function LoginForm() {
  const [phone, setPhone] = useState('');
  const [password, setPassword] = useState('');

  return (
    <View style={{ padding: 20 }}>
      <TextInput style={styles.input} placeholder="手机号" keyboardType="phone-pad" maxLength={11} value={phone} onChangeText={setPhone} />
      <TextInput style={styles.input} placeholder="密码" secureTextEntry value={password} onChangeText={setPassword} />
      <TouchableOpacity style={styles.loginBtn} onPress={() => handleLogin(phone, password)}>
        <Text style={{ color: '#fff', fontWeight: '600' }}>登录</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  title: { fontSize: 24, fontWeight: 'bold', textAlign: 'center' },
  logo: { width: 200, height: 200, alignSelf: 'center' },
  button: { backgroundColor: '#007AFF', paddingHorizontal: 24, paddingVertical: 12, borderRadius: 8, alignSelf: 'center' },
  buttonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
  input: { borderWidth: 1, borderColor: '#ddd', borderRadius: 8, paddingHorizontal: 16, paddingVertical: 12, marginBottom: 16, fontSize: 16 },
  loginBtn: { backgroundColor: '#007AFF', paddingVertical: 14, borderRadius: 8, alignItems: 'center' },
});
```

### 3. 样式（StyleSheet + Flexbox + Platform）

```tsx
import { StyleSheet, Platform } from 'react-native';

// RN 的 Flexbox 默认 flexDirection 为 column（Web 默认 row）
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 16,
  },
  card: {
    width: '48%',
    aspectRatio: 1,
    borderRadius: 12,
    backgroundColor: '#fff',
    // iOS 阴影
    ...Platform.select({
      ios: { shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.1, shadowRadius: 4 },
      android: { elevation: 3 },
    }),
    overflow: 'hidden',
  },
  header: {
    paddingTop: Platform.select({ ios: 44, android: 56 }),
  },
});

// 运行时判断
const isIOS = Platform.OS === 'ios';
const statusBarHeight = Platform.select({ ios: 44, android: 56, default: 48 });
```

**文件级平台适配（零运行时开销）：**

```
components/
├── Header.tsx           # 通用
├── Header.android.tsx   # Android 专用
└── Header.ios.tsx       # iOS 专用
```

```tsx
import Header from './Header'; // 打包时自动选择 .ios.tsx 或 .android.tsx
```

### 4. 导航（React Navigation）

```bash
npm install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
```

```tsx
// Stack 导航
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

type RootStackParamList = {
  Home: undefined;
  Detail: { id: string };
  Profile: { userId: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home" screenOptions={{ headerStyle: { backgroundColor: '#007AFF' }, headerTintColor: '#fff' }}>
        <Stack.Screen name="Home" component={HomeScreen} options={{ title: '首页' }} />
        <Stack.Screen name="Detail" component={DetailScreen} options={{ title: '详情' }} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Tab 导航
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
const Tab = createBottomTabNavigator();

function TabNavigator() {
  return (
    <Tab.Navigator screenOptions={{ tabBarActiveTintColor: '#007AFF', headerShown: false }}>
      <Tab.Screen name="HomeTab" component={HomeScreen} options={{ tabBarLabel: '首页', tabBarIcon: ({ color, size }) => <Icon name="home" size={size} color={color} /> }} />
      <Tab.Screen name="ProfileTab" component={ProfileScreen} options={{ tabBarLabel: '我的', tabBarIcon: ({ color, size }) => <Icon name="user" size={size} color={color} /> }} />
    </Tab.Navigator>
  );
}

// 导航与传参
import { useNavigation, useRoute } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';

function HomeScreen() {
  const navigation = useNavigation<NativeStackNavigationProp<RootStackParamList>>();
  return <TouchableOpacity onPress={() => navigation.navigate('Detail', { id: '123' })}><Text>查看详情</Text></TouchableOpacity>;
}

function DetailScreen() {
  const { id } = useRoute().params as { id: string };
  return <Text>详情页 ID: {id}</Text>;
}
```

### 5. 状态管理

**Zustand（推荐轻量方案）：**

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface AuthState {
  token: string | null;
  user: { name: string; avatar: string } | null;
  isLoggedIn: boolean;
  login: (token: string, user: any) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null, user: null, isLoggedIn: false,
      login: (token, user) => set({ token, user, isLoggedIn: true }),
      logout: () => set({ token: null, user: null, isLoggedIn: false }),
    }),
    { name: 'auth-storage', storage: createJSONStorage(() => AsyncStorage) }
  )
);

// 使用
const { user, isLoggedIn, logout } = useAuthStore();
```

**Redux Toolkit（大型项目方案）：**

```tsx
import { createSlice, createAsyncThunk, configureStore } from '@reduxjs/toolkit';

export const loginThunk = createAsyncThunk('user/login', async (creds: { phone: string; code: string }) => {
  const res = await apiLogin(creds);
  return res.data;
});

const userSlice = createSlice({
  name: 'user',
  initialState: { token: null as string | null, userInfo: null as any, loading: false },
  reducers: { logout: (state) => { state.token = null; state.userInfo = null; } },
  extraReducers: (builder) => {
    builder
      .addCase(loginThunk.pending, (state) => { state.loading = true; })
      .addCase(loginThunk.fulfilled, (state, action) => { state.loading = false; state.token = action.payload.token; })
      .addCase(loginThunk.rejected, (state) => { state.loading = false; });
  },
});

export const store = configureStore({ reducer: { user: userSlice.reducer } });
export type RootState = ReturnType<typeof store.getState>;
```

### 6. 原生模块与 Bridge

**旧架构 — Native Module（Bridge 异步）：**

```tsx
import { NativeModules } from 'react-native';
const { CameraRoll } = NativeModules;
await CameraRoll.save(uri); // 异步 JSON 序列化通信
```

**新架构 — TurboModule + Codegen（JSI 同步）：**

```tsx
// 1. 定义规范 (NativeCalculator.ts)
import type { TurboModule } from 'react-native/Libraries/TurboModule/RCTExport';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  add(a: number, b: number): Promise<number>;
  multiply(a: number, b: number): number; // JSI 同步调用
}
export default TurboModuleRegistry.getEnforcing<Spec>('Calculator');

// 2. Codegen 自动生成 C++ 胶水代码 → 3. 实现原生侧代码

// 4. JS 中使用
import Calculator from './NativeCalculator';
const sum = await Calculator.add(1, 2);      // 异步
const product = Calculator.multiply(3, 4);   // 同步（JSI）
```

### 7. 动画（Reanimated 3 + Gesture Handler）

```bash
npm install react-native-reanimated react-native-gesture-handler
```

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring, withTiming } from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

// 基础动画：缩放 + 淡入
function AnimatedCard() {
  const scale = useSharedValue(1);
  const opacity = useSharedValue(0);

  useEffect(() => { opacity.value = withTiming(1, { duration: 500 }); }, []);

  const animatedStyle = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }], opacity: opacity.value }));

  return (
    <Animated.View style={[styles.card, animatedStyle]}>
      <TouchableOpacity onPressIn={() => { scale.value = withSpring(0.95); }} onPressOut={() => { scale.value = withSpring(1); }}>
        <Text>点我缩放</Text>
      </TouchableOpacity>
    </Animated.View>
  );
}

// 手势拖拽
function DraggableCard() {
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);
  const ctx = useSharedValue({ x: 0, y: 0 });

  const pan = Gesture.Pan()
    .onStart(() => { ctx.value = { x: translateX.value, y: translateY.value }; })
    .onUpdate((e) => { translateX.value = ctx.value.x + e.translationX; translateY.value = ctx.value.y + e.translationY; })
    .onEnd(() => { translateX.value = withSpring(0); translateY.value = withSpring(0); });

  const animatedStyle = useAnimatedStyle(() => ({ transform: [{ translateX: translateX.value }, { translateY: translateY.value }] }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.dragCard, animatedStyle]}><Text>拖拽我</Text></Animated.View>
    </GestureDetector>
  );
}
```

### 8. 网络与存储

```tsx
import AsyncStorage from '@react-native-async-storage/async-storage';

// 网络请求封装
const BASE_URL = __DEV__ ? 'http://localhost:3000/api' : 'https://api.example.com/api';

async function request<T>(url: string, options: RequestInit = {}): Promise<T> {
  const token = await AsyncStorage.getItem('token');
  const response = await fetch(`${BASE_URL}${url}`, {
    ...options,
    headers: { 'Content-Type': 'application/json', Authorization: token ? `Bearer ${token}` : '', ...options.headers },
  });
  if (response.status === 401) { await AsyncStorage.removeItem('token'); /* 跳转登录 */ }
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  const result = await response.json();
  return result.data;
}

export const api = {
  get: <T>(url: string) => request<T>(url),
  post: <T>(url: string, data?: unknown) => request<T>(url, { method: 'POST', body: JSON.stringify(data) }),
  put: <T>(url: string, data?: unknown) => request<T>(url, { method: 'PUT', body: JSON.stringify(data) }),
  delete: <T>(url: string) => request<T>(url, { method: 'DELETE' }),
};

// AsyncStorage 本地存储
await AsyncStorage.setItem('token', 'xxx');
const token = await AsyncStorage.getItem('token');
await AsyncStorage.removeItem('token');
// 对象存储需 JSON.stringify/parse

// SQLite（Expo）
import * as SQLite from 'expo-sqlite';
const db = SQLite.openDatabase('myapp.db');
db.execSync('CREATE TABLE IF NOT EXISTS messages (id INTEGER PRIMARY KEY, content TEXT, created_at DATETIME DEFAULT CURRENT_TIMESTAMP)');
db.runSync('INSERT INTO messages (content) VALUES (?)', ['你好']);
const rows = db.getAllSqlSync<{ id: number; content: string }>('SELECT * FROM messages');
```

### 9. 推送通知

```tsx
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

Notifications.setNotificationHandler({
  handleNotification: async () => ({ shouldShowAlert: true, shouldPlaySound: true, shouldSetBadge: true }),
});

async function registerForPushNotifications() {
  if (!Device.isDevice) return;
  const { status } = await Notifications.requestPermissionsAsync();
  if (status !== 'granted') return;
  const token = (await Notifications.getExpoPushTokenAsync()).data;
  // Android 通知渠道
  if (Platform.OS === 'android') {
    Notifications.setNotificationChannelAsync('default', { name: 'default', importance: Notifications.AndroidImportance.MAX });
  }
  return token;
}

// 监听通知点击
useEffect(() => {
  const sub = Notifications.addNotificationResponseReceivedListener((response) => {
    const data = response.notification.request.content.data;
    if (data.type === 'chat') navigation.navigate('Chat', { id: data.chatId });
  });
  return () => sub.remove();
}, []);
```

### 10. 平台特定代码

```tsx
import { Platform, BackHandler } from 'react-native';

// 运行时判断
{Platform.OS === 'ios' ? <IOSDatePicker /> : <AndroidDatePicker />}

if (Platform.OS === 'android') {
  BackHandler.addEventListener('hardwareBackPress', handleBackPress);
}

// 文件扩展名（推荐，零运行时开销）
// utils/platform.ios.ts     → export const getStatusBarHeight = () => 44;
// utils/platform.android.ts → export const getStatusBarHeight = () => 56;
// import { getStatusBarHeight } from './platform'; // 自动选择
```

### 11. 调试

| 工具 | 用途 | 命令/说明 |
|------|------|----------|
| React DevTools | 组件树审查 | `npx react-devtools` |
| Flipper | 全能调试 | 网络、布局、数据库、通知 |
| Chrome DevTools | JS 远程调试 | Hermes 支持 CDP 协议 |
| Reactotron | 状态/网络日志 | Redux 状态追踪 |
| Expo Go | 实时预览 | 扫码调试 |

```tsx
// 开发环境日志与性能监测
if (__DEV__) { console.log('调试信息', data); }

import { PerformanceObserver } from 'react-native';
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => console.log(`${entry.name}: ${entry.duration}ms`));
});
observer.observe({ type: 'measure', buffered: true });
```

### 12. 构建与部署

```bash
# EAS Build（Expo 项目）
npm install -g eas-cli
eas login && eas build:configure
eas build --platform ios          # iOS 构建
eas build --platform android      # Android 构建
eas submit --platform ios         # 提交 App Store
eas submit --platform android     # 提交 Play Store
eas update --branch production --message "修复登录问题"  # 热更新
```

**eas.json 配置：**

```json
{
  "build": {
    "development": { "developmentClient": true, "distribution": "internal" },
    "preview": { "distribution": "internal", "android": { "buildType": "apk" } },
    "production": { "autoIncrement": true }
  }
}
```

**RN CLI 项目构建：**

```bash
# Android: cd android && ./gradlew assembleRelease
# iOS: Xcode → Product → Archive → Distribute App
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Android 打包闪退 | Hermes 字节码与原生库冲突 | `cd android && ./gradlew clean` |
| iOS Pod install 失败 | CocoaPods 版本或源问题 | 更新 pod repo，检查 Podfile 约束 |
| FlatList 滚动卡顿 | renderItem 内联函数/重渲染 | 抽离 renderItem、memo 包裹、设 windowSize |
| 键盘遮挡输入框 | 未处理键盘弹出 | KeyboardAvoidingView 或 keyboard-aware-scroll-view |
| 图片不显示 | HTTP 链接被阻止 | iOS 配置 ATS，Android 设 `usesCleartextTraffic` |
| 内存泄漏 | 定时器/监听未清理 | useEffect 返回清理函数 |
| 新架构库不兼容 | 旧 Native Module 未适配 | 检查库支持情况，必要时降级 |
| 字体自定义不生效 | 未正确 link 字体资源 | Expo 用 `useFonts`，CLI 需 react-native link |
| SafeArea 遮挡 | 未处理刘海/状态栏 | 使用 SafeAreaView 或 safe-area-context |
| 动画掉帧 | JS 线程阻塞 | Reanimated worklet 在 UI 线程运行 |
| Android 字体缩放 | 系统字体缩放导致布局错乱 | `allowFontScaling={false}` |
| iOS 滚动弹跳 | iOS 默认弹性滚动 | `bounces={false}` + `overScrollMode="never"` |

### 最佳实践

- 优先使用 Expo 开发，降低原生配置复杂度
- TypeScript + Codegen 获得完整类型安全
- 列表用 FlatList/FlashList，避免 ScrollView 渲染大量数据
- 动画用 Reanimated 3 worklet，在 UI 线程运行不卡顿
- StyleSheet.create 而非内联样式对象
- 图片用 WebP 格式 + 合适尺寸 + 缓存
- 网络请求统一封装，处理 token 过期、错误重试
- Zustand 管理全局状态，轻量且 TS 友好
- 平台差异用 `.ios.tsx`/`.android.tsx` 文件拆分
- 及时清理定时器、监听器、订阅，避免内存泄漏
- 生产构建开启 Hermes 引擎
- React.memo + useMemo/useCallback 减少重渲染

## 面试题

**Q1: React Native 的 Bridge 机制是什么？有什么性能瓶颈？新架构如何解决？**
> Bridge 是旧架构中 JS 线程与原生线程通信的异步消息队列，数据需经过 JSON 序列化/反序列化，通信是异步的无法同步返回。性能瓶颈：大量数据传输时序列化开销大；异步通信导致滚动、手势等交互有延迟；消息队列积压会掉帧。新架构通过 JSI 实现同步直接引用（JS 直接持有 C++ 对象指针），无需序列化；Fabric 渲染器支持同步布局和优先级调度；TurboModules 按需加载减少启动时间。

**Q2: React Native 新架构（Fabric + JSI + TurboModules）相比旧架构有哪些核心改进？**
> 四大核心改进：① JSI 替代 Bridge，JS 与原生可同步互调，零序列化开销；② Fabric 渲染器支持同步测量布局、优先级调度和批量更新，解决滚动/手势延迟；③ TurboModules 按需懒加载，启动时只初始化用到的模块，减少启动时间；④ Codegen 根据 TS 规范自动生成类型安全的 C++/原生胶水代码，消除手动桥接的类型错误。整体让 RN 性能接近原生。

**Q3: FlatList 和 ScrollView 有什么区别？如何优化 FlatList 性能？**
> ScrollView 一次性渲染所有子组件，适合内容较少的场景；FlatList 虚拟化渲染，只渲染可视区域和少量缓冲区项目，回收离开视口的组件，适合长列表。优化方法：① 抽离 renderItem 为独立函数；② React.memo 包裹列表项；③ 合理 `windowSize`；④ `keyExtractor` 用稳定 key；⑤ `getItemLayout` 跳过异步测量（固定高度）；⑥ `initialNumToRender` 设首屏数；⑦ `maxToRenderPerBatch` 控制批次；⑧ 用 FlashList 替代（更高性能）。

**Q4: React Native 中的线程模型是怎样的？**
> RN 有三个主要线程：① JS 线程 — 运行 React 和业务逻辑（Hermes 引擎）；② UI 线程（主线程）— 运行原生渲染和布局计算；③ Shadow 线程 — 计算 Flexbox 布局（旧架构后台线程，Fabric 在 UI 线程同步计算）。Bridge/JSI 负责线程间通信。动画卡顿的原因是 JS 线程被阻塞无法及时发送更新指令，Reanimated 的 worklet 直接在 UI 线程执行绕过了这个问题。

**Q5: React Native 如何实现热更新？原理是什么？**
> RN 热更新（CodePush / EAS Update）原理：App 启动时检查更新服务器是否有新的 JS Bundle，如有则下载替换本地 Bundle，下次启动加载新 Bundle。核心是 RN 的 UI 渲染和业务逻辑都在 JS Bundle 中，原生壳只是加载器，替换 Bundle 不涉及原生代码修改，无需应用商店审核。限制：不能修改原生代码，只能更新 JS 层。EAS Update 基于 Hermes 字节码增量更新，下载体积更小。

**Q6: React Native 和 Flutter 的核心区别是什么？如何选择？**
> 核心区别在渲染方式：RN 映射到平台原生组件渲染（iOS UIKit / Android View），Flutter 用自绘引擎（Skia/Impeller）直接绘制像素，不依赖平台组件。RN 优势：使用 JS/TS，Web 开发者易上手，生态庞大，支持热更新；Flutter 优势：性能更稳定（无 Bridge 开销），UI 一致性好，动画系统强大，Dart 强类型+AOT 编译。选择：React 团队 + 需热更新 + 原生组件体验 → RN；追求极致性能 + UI 一致性 + 全平台覆盖 → Flutter。

**Q7: 如何处理 React Native 中的平台特定代码？有哪些方式？**
> 三种方式：① 文件扩展名（推荐）— 创建 `Component.ios.tsx` 和 `Component.android.tsx`，打包时自动选择对应平台文件，类型安全且零运行时开销；② `Platform` API — 运行时用 `Platform.OS` 判断、`Platform.select()` 选择值，适合少量差异；③ 条件渲染 — JSX 中 `Platform.OS === 'ios' ? <A /> : <B />`，适合简单场景。推荐优先使用文件扩展名方式，平台代码完全隔离，维护性最好。

**Q8: Reanimated 3 的 worklet 是什么？为什么能解决动画卡顿问题？**
> Worklet 是 Reanimated 定义的特殊函数，通过 Babel 插件编译为可在 UI 线程独立运行的代码块。普通 RN 动画流程：JS 线程计算 → Bridge/JSI → UI 线程更新，JS 线程阻塞时动画卡顿。Worklet 流程：动画计算直接在 UI 线程执行，无需经过 JS 线程，即使 JS 线程忙碌动画仍然流畅。`useSharedValue` 创建的共享值可在两个线程间同步访问，`useAnimatedStyle` 在 UI 线程计算样式，`runOnJS` 在需要时回调 JS 线程。

---

**相关链接：**
- [[React核心]]
- [[React生态Router与状态管理]]
- [[TypeScript类型体操]]
- [[跨端开发方案对比]]
- [[Electron桌面开发]]
- [[uni-app跨端开发]]

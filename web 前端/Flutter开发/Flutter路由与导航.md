---
tags:
  - Web前端
  - Flutter
  - 路由
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Flutter路由与导航

## What — 是什么

> Flutter 路由与导航是指在应用中管理页面跳转、返回、传参和页面栈的机制。Flutter 提供了从简单的 Navigator 1.0（命令式 push/pop）到声明式的 Navigator 2.0/Router API，以及社区主流的 GoRouter 方案。

**核心概念：**

- **Navigator**：管理 Route 对象栈的组件，push 入栈、pop 出栈
- **Route**：页面的抽象，MaterialPageRoute 是最常用的实现
- **页面栈**：后进先出的路由栈，栈顶为当前可见页面
- **命名路由**：通过字符串名称映射到页面构建函数
- **声明式路由**：通过路由配置表描述路由结构，如 GoRouter

**路由体系演进：**

```
Navigator 1.0          Navigator 2.0 / Router API       GoRouter
(命令式)               (声明式，官方)                    (声明式，社区)
─────────────          ────────────────────────         ────────────
简单直接                复杂但灵活                        简单且灵活
push/pop               Router + Delegate + Parser        路由配置表
难以处理深链接           支持深链接                        原生支持深链接
不支持浏览器地址栏       支持                              支持
无重定向/守卫           手动实现                          内置 redirect
适合小型应用            适合大型/多平台                    适合所有规模
```

## Why — 为什么

**适用场景：**

- 多页面应用需要页面跳转和返回
- 需要路由守卫（未登录重定向到登录页）
- 需要深度链接（从 URL 直接打开特定页面）
- 需要底部导航 Tab 切换
- Web 端需要浏览器前进/后退支持

**对比同类方案：**

| 维度 | Navigator 1.0 | Router API | GoRouter |
|------|--------------|------------|----------|
| 编程范式 | 命令式 | 声明式 | 声明式 |
| 学习曲线 | 低 | 高 | 中 |
| 深度链接 | 不支持 | 支持 | 支持 |
| 路由守卫 | 手动 | 手动 | 内置 |
| Web 地址栏 | 不支持 | 支持 | 支持 |
| 嵌套路由 | 困难 | 支持 | 支持 |
| 代码量 | 少 | 多 | 少 |
| 官方/社区 | 官方 | 官方 | 社区(官方推荐) |

## How — 怎么用

### 1. Navigator 1.0 基础

```dart
// ===== push / pop =====
// 压入新页面
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const DetailPage()),
);

// 返回上一页
Navigator.pop(context);

// ===== pushReplacement — 替换当前页 =====
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => const HomePage()),
);

// ===== pushAndRemoveUntil — 清空栈到指定页面 =====
// 登录成功后清空所有页面回到首页
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const HomePage()),
  (route) => false,    // 清空所有路由
);

// ===== popUntil — 返回到指定页面 =====
Navigator.popUntil(context, (route) => route.isFirst);

// ===== 路由传参 =====
// 传参
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailPage(id: '123', name: 'Product'),
  ),
);

// 回传结果
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => const SelectCityPage()),
);
if (result != null) {
  print('Selected: $result');
}

// 被调用方返回结果
Navigator.pop(context, 'Beijing');
```

### 2. 命名路由

```dart
// ===== 定义命名路由 =====
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const HomePage(),
    '/detail': (context) => const DetailPage(),
    '/profile': (context) => const ProfilePage(),
  },
  onGenerateRoute: (settings) {
    // 处理带参数的路由
    if (settings.name == '/product') {
      final args = settings.arguments as Map<String, dynamic>;
      return MaterialPageRoute(
        builder: (context) => ProductPage(id: args['id']),
      );
    }
    return null;
  },
);

// ===== 使用命名路由 =====
Navigator.pushNamed(context, '/detail');
Navigator.pushReplacementNamed(context, '/home');
Navigator.popAndPushNamed(context, '/login');

// 带参数
Navigator.pushNamed(
  context,
  '/product',
  arguments: {'id': '123'},
);

// 获取参数
class ProductPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
    return Text('Product: ${args['id']}');
  }
}
```

### 3. 自定义路由过渡动画

```dart
// ===== PageRouteBuilder 自定义动画 =====
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const DetailPage(),
    transitionDuration: const Duration(milliseconds: 300),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      // 滑动动画
      var tween = Tween(begin: const Offset(1.0, 0.0), end: Offset.zero)
          .chain(CurveTween(curve: Curves.easeInOut));
      return SlideTransition(position: animation.drive(tween), child: child);
    },
  ),
);

// 淡入淡出
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  return FadeTransition(opacity: animation, child: child);
}

// 缩放
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  var tween = Tween(begin: 0.0, end: 1.0).chain(CurveTween(curve: Curves.easeOutBack));
  return ScaleTransition(scale: animation.drive(tween), child: child);
}

// 组合动画（滑动 + 淡入）
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  return SlideTransition(
    position: Tween<Offset>(begin: const Offset(1, 0), end: Offset.zero)
        .chain(CurveTween(curve: Curves.easeOut)).animate(animation),
    child: FadeTransition(opacity: animation, child: child),
  );
}
```

### 4. GoRouter（推荐方案）

```dart
// ===== 安装 =====
// flutter pub add go_router

// ===== 路由配置 =====
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomePage(),
    ),
    GoRoute(
      path: '/detail/:id',           // 路径参数
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailPage(id: id);
      },
    ),
    GoRoute(
      path: '/search',
      builder: (context, state) {
        final query = state.uri.queryParameters['q'] ?? '';  // 查询参数
        return SearchPage(query: query);
      },
    ),
    // 嵌套路由
    GoRoute(
      path: '/settings',
      builder: (context, state) => const SettingsPage(),
      routes: [
        GoRoute(
          path: 'account',            // 完整路径：/settings/account
          builder: (context, state) => const AccountPage(),
        ),
        GoRoute(
          path: 'privacy',
          builder: (context, state) => const PrivacyPage(),
        ),
      ],
    ),
  ],
  // 全局重定向（路由守卫）
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn;
    final isLoginRoute = state.matchedLocation == '/login';

    if (!isLoggedIn && !isLoginRoute) return '/login';
    if (isLoggedIn && isLoginRoute) return '/';
    return null;  // 无需重定向
  },
);

// ===== MaterialApp.router =====
MaterialApp.router(
  routerConfig: router,
  title: 'My App',
);

// ===== 导航使用 =====
context.go('/');                          // 替换当前路由
context.go('/detail/123');                // 路径参数
context.go('/search?q=flutter');          // 查询参数
context.push('/detail/123');              // 压入新路由（可返回）
context.pop();                            // 返回

// 命名路由（GoRouter）
GoRoute(
  name: 'detail',                        // 命名
  path: 'detail/:id',
  builder: (context, state) => DetailPage(id: state.pathParameters['id']!),
),
context.namedLocation('detail', pathParameters: {'id': '123'});
context.goNamed('detail', pathParameters: {'id': '123'});

// ===== ShellRoute — 布局嵌套 =====
ShellRoute(
  builder: (context, state, child) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          switch (index) {
            case 0: context.go('/');
            case 1: context.go('/explore');
            case 2: context.go('/profile');
          }
        },
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.explore), label: 'Explore'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  },
  routes: [
    GoRoute(path: '/', builder: (context, state) => const HomePage()),
    GoRoute(path: '/explore', builder: (context, state) => const ExplorePage()),
    GoRoute(path: '/profile', builder: (context, state) => const ProfilePage()),
  ],
)
```

### 5. 底部导航（BottomNavigationBar + PageView）

```dart
class MainNavigation extends StatefulWidget {
  @override
  State<MainNavigation> createState() => _MainNavigationState();
}

class _MainNavigationState extends State<MainNavigation> {
  int _currentIndex = 0;
  final _pages = const [HomePage(), ExplorePage(), ProfilePage()];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: IndexedStack(              // 保持页面状态
        index: _currentIndex,
        children: _pages,
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: '首页'),
          BottomNavigationBarItem(icon: Icon(Icons.explore), label: '发现'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: '我的'),
        ],
      ),
    );
  }
}
```

### 6. Tab 导航

```dart
class TabPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Tabs'),
          bottom: const TabBar(
            tabs: [
              Tab(icon: Icon(Icons.chat), text: '聊天'),
              Tab(icon: Icon(Icons.contacts), text: '通讯录'),
              Tab(icon: Icon(Icons.settings), text: '设置'),
            ],
          ),
        ),
        body: const TabBarView(
          children: [
            ChatList(),
            ContactList(),
            SettingsList(),
          ],
        ),
      ),
    );
  }
}

// 自定义 TabController（需要动态控制 Tab）
class CustomTabPage extends StatefulWidget {
  @override
  State<CustomTabPage> createState() => _CustomTabPageState();
}

class _CustomTabPageState extends State<CustomTabPage>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 3, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: TabBarView(
        controller: _tabController,
        children: const [ChatList(), ContactList(), SettingsList()],
      ),
      bottomNavigationBar: TabBar(
        controller: _tabController,
        tabs: const [
          Tab(icon: Icon(Icons.chat), text: '聊天'),
          Tab(icon: Icon(Icons.contacts), text: '通讯录'),
          Tab(icon: Icon(Icons.settings), text: '设置'),
        ],
      ),
    );
  }
}
```

### 7. 深度链接

```yaml
# Android: android/app/src/main/AndroidManifest.xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" android:host="detail" />
</intent-filter>
```

```xml
<!-- iOS: ios/Runner/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

```dart
// GoRouter 自动处理深度链接
// myapp://detail/123 → 自动导航到 /detail/123
// https://myapp.com/detail/123 → 需要配置 App Links / Universal Links
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| pop 后黑屏 | 页面栈为空 | 用 pushAndRemoveUntil 保证有根页面 |
| 路由传参丢失 | 重建时参数未保存 | 用 state.extra 或路径参数 |
| BottomNavigationBar 状态丢失 | 切换 Tab 时 Widget 重建 | 用 IndexedStack 保持状态 |
| GoRouter 上下文错误 | context 不在 Router 内 | 使用 GoRouter.of(context) 或全局 key |
| 深度链接不生效 | 平台配置缺失 | 检查 AndroidManifest/Info.plist |
| 页面切换卡顿 | 过渡动画太重 | 缩短 duration 或用简单动画 |
| Web 刷新 404 | 服务端未配置 SPA 路由 | nginx 配置 try_files 回退 index.html |

### 最佳实践

- 新项目优先使用 GoRouter，简洁且功能完整
- 路由守卫统一在 redirect 中处理
- 底部导航用 ShellRoute 或 IndexedStack
- 路径参数用于资源标识（/detail/:id），查询参数用于筛选（/search?q=xxx）
- 自定义动画用 PageRouteBuilder
- Web 端注意配置服务端 SPA 路由回退
- 路由传参用 state.extra 传复杂对象
- 避免在路由中传递大对象，用状态管理共享

## 面试题

**Q1: Flutter 的 Navigator 1.0 和 Navigator 2.0 有什么区别？为什么推荐 GoRouter？**
> Navigator 1.0 是命令式 API，通过 push/pop 管理路由栈，简单直接但不支持深度链接和浏览器地址栏同步。Navigator 2.0 是声明式 API，通过 Router/RouteInformationParser/RouterDelegate 三件套实现，灵活但极其复杂。GoRouter 是社区方案，用声明式路由配置表实现了 Navigator 2.0 的所有能力（深度链接、嵌套路由、重定向），代码量大幅减少，Flutter 官方也推荐使用。

**Q2: GoRouter 的 redirect 如何实现路由守卫？**
> GoRouter 的 redirect 是一个全局回调函数，在每次路由跳转前执行，接收当前路由状态并返回目标路径。如果返回非 null 值则执行重定向，返回 null 则正常导航。典型实现：检查 AuthService.isLoggedIn，如果未登录且目标不是 /login，则重定向到 /login；如果已登录且目标是 /login，则重定向到 /。这比在 Widget 层判断更优雅，且在 Web 端不会闪现未授权页面。

**Q3: IndexedStack 和 PageView 在底部导航中各有什么优缺点？**
> IndexedStack 同时构建所有子页面，切换时不重建，状态完全保留，但所有页面都常驻内存。PageView 按需构建页面，内存友好，但默认会销毁离屏页面（需设置 AutomaticKeepAliveClientMixin 保留状态）。选择：页面少（3-5个）、内容重要用 IndexedStack；页面多或内容占内存大用 PageView + KeepAlive。

**Q4: 如何实现页面间的数据传递？有哪些方式？**
> 五种方式：①构造函数参数 — 最简单，push 时传入；②路由参数 — pushNamed 的 arguments 或 GoRouter 的 state.extra；③回传结果 — pop(context, result) 配合 await push 获取；④状态管理 — Provider/Riverpod/Bloc 共享状态，最灵活；⑤路由路径参数 — /detail/:id，适合 RESTful 风格。推荐：简单传参用路径参数/extra，复杂数据用状态管理，一次性结果用 pop 回传。

**Q5: ShellRoute 是什么？解决什么问题？**
> ShellRoute 是 GoRouter 提供的布局嵌套机制，为子路由共享一个共同的 Shell（外壳）Widget。典型场景：底部导航栏在多个 Tab 页面间共享，切换 Tab 时导航栏不重建。原理：ShellRoute 的 builder 接收 child 参数（当前匹配的子路由页面），在 Shell 中包裹 child 并添加共享 UI（如 Scaffold + BottomNavigationBar）。嵌套 ShellRoute 可以实现多层布局（如侧边栏 + 底部导航）。

**Q6: Flutter Web 端的路由和移动端有什么不同？需要注意什么？**
> 核心区别：Web 端路由需要与浏览器地址栏同步，支持前进/后退按钮和 URL 直接访问。注意事项：①URL 直接访问需要服务端配置 SPA 回退（所有路径返回 index.html），否则刷新会 404；②深度链接需要配置 App Links/Universal Links；③使用 GoRouter 自动处理浏览器历史记录；④路由参数出现在 URL 中，注意不要暴露敏感信息；⑤页面标题需要随路由变化更新（GoRouter 的 builder 中设置）。

**Q7: 自定义路由过渡动画有哪些方式？**
> 三种方式：①PageRouteBuilder — 最灵活，可自定义 transitionDuration 和 transitionsBuilder，支持滑动/淡入/缩放/旋转等任意组合；②Theme 全局配置 — PageTransitionsTheme 在 ThemeData 中设置全局过渡动画；③Hero 动画 — 共享元素过渡，两个页面的相同 Widget 间平滑过渡。推荐：单页面自定义用 PageRouteBuilder，全局统一用 Theme，页面间元素共享用 Hero。

**Q8: GoRouter 的 push 和 go 有什么区别？**
> go 是替换式导航，替换当前路由栈顶为新的路由，类似 pushReplacement，不会在栈中积累页面。push 是压栈式导航，将新页面压入路由栈，可以通过 pop 返回上一页。使用场景：底部导航 Tab 切换用 go（不需要返回上一个 Tab），从列表进入详情用 push（需要返回列表）。关键区别：go 修改当前路由匹配结果，push 在栈上叠加新路由。

---

**相关链接：**
- [[Flutter核心与Widget体系]]
- [[Flutter状态管理]]
- [[React生态Router与Query]]
- [[Vue生态Router与Pinia]]

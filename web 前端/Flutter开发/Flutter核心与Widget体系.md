---
tags:
  - Web前端
  - Flutter
  - Widget
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Flutter核心与Widget体系

## What — 是什么

> Flutter 是 Google 推出的跨平台 UI 框架，使用 Dart 语言编写，通过自绘引擎（Skia/Impeller）直接绘制像素，实现一套代码运行在 iOS、Android、Web、Windows、macOS、Linux 六大平台，无需依赖平台原生组件。

**核心概念：**

- **Widget（组件）**：Flutter 中一切皆 Widget，是 UI 的不可变描述（配置信息），类似 React 的虚拟 DOM 节点
- **Element（元素）**：Widget 的实例化对象，管理生命周期和树结构，是 Widget 与 RenderObject 的桥梁
- **RenderObject（渲染对象）**：负责测量（layout）和绘制（paint）的实际工作单元
- **StatelessWidget**：无状态组件，build 方法根据输入属性生成 Widget 树，属性不变则不重建
- **StatefulWidget**：有状态组件，通过 State 对象持有可变状态，状态变化触发 rebuild
- **InheritedWidget**：数据向下传递机制，子树中的后代 Widget 可高效获取祖先数据
- **Key**：Widget 的身份标识，控制 Element 的复用与更新策略

**核心架构：**

```
┌──────────────────────────────────────────────────┐
│                   Flutter App                     │
├──────────────────────────────────────────────────┤
│  Widget Tree  →  Element Tree  →  RenderObject   │
│  (不可变配置)     (生命周期管理)     (测量与绘制)    │
├──────────────────────────────────────────────────┤
│              Framework 层（Dart）                  │
│  Material / Cupertino / Widgets / Rendering       │
│  Animation / Gestures / Foundation                │
├──────────────────────────────────────────────────┤
│              Engine 层（C++）                      │
│  Skia / Impeller  │  Dart VM (AOT/JIT)           │
│  Text (LibTxt)    │  Platform Channels            │
├──────────────────────────────────────────────────┤
│              Embedder 层（平台原生）                │
│  iOS (UIKit) │ Android (Activity) │ Web (HTML)    │
│  Windows     │ macOS (NSView)     │ Linux (GTK)   │
└──────────────────────────────────────────────────┘
```

- 设计理念：Everything is a Widget — 声明式 UI，不可变配置描述界面
- 核心模块：Dart 框架层 + C++ 引擎层 + 平台嵌入层
- 数据流：状态变化 → Widget rebuild → Element diff → RenderObject 更新 → 重绘

**三棵树详解：**

```
Widget Tree          Element Tree          RenderObject Tree
┌──────────┐        ┌──────────┐         ┌──────────┐
│  MyApp   │───────→│  Element │         │  无      │
└────┬─────┘        └────┬─────┘         └──────────┘
     │                   │
┌────▼─────┐        ┌────▼─────┐         ┌──────────────┐
│ Scaffold │───────→│  Element │────────→│RenderPadding │
└────┬─────┘        └────┬─────┘         └──────┬───────┘
     │                   │                       │
┌────▼─────┐        ┌────▼─────┐         ┌──────▼───────┐
│  Column  │───────→│  Element │────────→│RenderFlex    │
└────┬─────┘        └────┬─────┘         └──────┬───────┘
     │                   │                       │
┌────▼─────┐        ┌────▼─────┐         ┌──────▼───────┐
│   Text   │───────→│  Element │────────→│RenderParagraph│
└──────────┘        └──────────┘         └──────────────┘
```

- Widget 是配置蓝图，每次 rebuild 都是新对象
- Element 是持久实例，通过 canUpdate 判断是否可复用
- RenderObject 是真正执行布局绘制的对象

**与 React 虚拟 DOM 对比：**

| 维度 | Flutter Widget 树 | React 虚拟 DOM |
|------|-------------------|----------------|
| 本质 | 不可变配置描述 | 轻量级 JS 对象 |
| Diff 算法 | O(1) 同级比较（type+key） | O(n) 同级 + key 策略 |
| 更新粒度 | 子树整体 rebuild | 组件级 reconcile |
| 渲染目标 | RenderObject → Skia 绘制 | 真实 DOM 节点 |
| 跨平台 | 统一自绘引擎 | 各平台不同渲染 |

**渲染引擎演进：**

| 对比项 | Skia | Impeller |
|--------|------|----------|
| 编译 | 运行时着色器编译 | AOT 预编译着色器 |
| 首帧 | 可能有卡顿 | 首帧流畅 |
| 并发 | 单线程渲染 | 多线程并行 |
| 状态 | 稳定，默认引擎 | 新一代，iOS 已默认 |
| 目标 | 兼容性优先 | 性能优先 |

**插件生态：**

- 官方插件：flutter/plugins（camera、image_picker、url_launcher、path_provider、shared_preferences 等）
- 社区热门：dio、go_router、riverpod、bloc、freezed、json_serializable、cached_network_image
- 插件仓库：pub.dev（Dart/Flutter 官方包管理平台）

## Why — 为什么

**适用场景：**

- 需要同时覆盖多平台（移动端 + Web + 桌面）的应用
- 对 UI 一致性和流畅度有高要求的场景
- 复杂动画和自定义绘制需求（游戏、图表、画板）
- MVP 快速验证阶段，一套代码多端运行
- 品牌定制化 UI，不依赖平台原生组件外观
- 内部工具/企业应用，跨平台效率优先

**对比同类框架：**

| 维度 | Flutter | React Native | uni-app | Kotlin/Swift |
|------|---------|-------------|---------|-------------|
| 开发语言 | Dart | JavaScript/TS | Vue/JS/TS | Kotlin/Swift |
| 渲染方式 | 自绘引擎(Skia/Impeller) | 原生组件映射 | WebView/原生 | 原生 |
| 性能 | 极高 | 高 | 中等 | 极高 |
| 学习曲线 | 中（需学 Dart） | 中（需 React） | 低 | 高 |
| 生态 | 丰富（pub.dev） | 丰富（npm+原生） | 一般 | 最强 |
| 跨端数量 | 6+（移动/Web/桌面） | iOS+Android+Web | 10+（含小程序） | 各1 |
| 热更新 | 不支持（Hot Reload仅开发） | 支持（CodePush） | 支持 | 不支持 |
| UI 一致性 | 极高（自绘） | 中（依赖平台） | 低（WebView） | 原生 |
| 动画 | 内置强大 | Reanimated 3 | 有限 | 原生 |
| 包体积 | 较大（~5MB） | 较大（~3MB） | 小 | 最小 |

**优缺点：**

- ✅ 优点：
  - 自绘引擎渲染，UI 在各平台完全一致
  - 性能接近原生，无桥接通信开销
  - 内置丰富的 Material 和 Cupertino 组件库
  - Hot Reload 开发体验极佳，修改即时生效
  - 一套代码覆盖 6 大平台
  - 强大的动画和自定义绘制能力
  - Dart 强类型 + AOT 编译，运行时安全
  - 声明式 UI 范式，代码可读性好
- ❌ 缺点：
  - 需要学习 Dart 语言
  - 不支持热更新（无法像 RN 的 CodePush 动态更新业务代码）
  - 包体积比原生大
  - 部分原生功能需写 Platform Channel
  - Web 端性能不如移动端
  - 长列表极度复杂场景需精细优化
  - 社区生态不如 React Native 成熟

## How — 怎么用

### 1. 环境搭建与项目创建

```bash
# 安装 Flutter SDK
# Windows: 下载 https://docs.flutter.dev/get-started/install/windows
# macOS: brew install flutter
# 验证安装
flutter doctor

# 创建项目
flutter create my_app
cd my_app

# 运行
flutter run                    # 调试模式
flutter run -d chrome          # Web 端
flutter run -d windows         # Windows 桌面
flutter run --release          # 发布模式

# 常用命令
flutter pub get                # 安装依赖
flutter pub upgrade            # 升级依赖
flutter pub outdated           # 检查过期依赖
flutter analyze                # 静态分析
flutter test                   # 运行测试
flutter build apk              # Android APK
flutter build ios              # iOS 构建
flutter build web              # Web 构建
flutter build windows          # Windows 构建
```

**项目结构：**

```
my_app/
├── lib/                       # Dart 源码
│   ├── main.dart              # 入口文件
│   ├── screens/               # 页面
│   ├── widgets/               # 公共组件
│   ├── models/                # 数据模型
│   ├── services/              # 服务层（网络/存储）
│   ├── providers/             # 状态管理
│   ├── utils/                 # 工具函数
│   └── theme/                 # 主题配置
├── test/                      # 测试文件
├── android/                   # Android 原生配置
├── ios/                       # iOS 原生配置
├── web/                       # Web 配置
├── windows/                   # Windows 配置
├── pubspec.yaml               # 依赖与项目配置
└── analysis_options.yaml      # 代码规范配置
```

**pubspec.yaml 配置：**

```yaml
name: my_app
description: A new Flutter project
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  dio: ^5.4.0                    # 网络请求
  go_router: ^13.0.0             # 路由
  flutter_riverpod: ^2.4.0       # 状态管理
  freezed_annotation: ^2.4.0     # 数据类注解
  json_annotation: ^4.8.0        # JSON 注解
  shared_preferences: ^2.2.0     # 本地存储
  cached_network_image: ^3.3.0   # 图片缓存

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  build_runner: ^2.4.0           # 代码生成
  freezed: ^2.4.0
  json_serializable: ^6.7.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/fonts/
  fonts:
    - family: CustomFont
      fonts:
        - asset: assets/fonts/CustomFont-Regular.ttf
        - asset: assets/fonts/CustomFont-Bold.ttf
          weight: 700
```

### 2. 应用入口

```dart
import 'package:flutter/material.dart';

void main() {
  // 确保 Flutter 绑定初始化（异步操作前需要）
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter App',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: const HomePage(),
      routes: {
        '/home': (context) => const HomePage(),
        '/detail': (context) => const DetailPage(),
      },
    );
  }
}
```

### 3. StatelessWidget 无状态组件

```dart
class GreetingCard extends StatelessWidget {
  final String name;
  final String? subtitle;
  final VoidCallback? onTap;

  const GreetingCard({
    super.key,
    required this.name,
    this.subtitle,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      child: InkWell(
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            mainAxisSize: MainAxisSize.min,
            children: [
              Text('Hello, $name!', style: Theme.of(context).textTheme.headlineSmall),
              if (subtitle != null) ...[
                const SizedBox(height: 4),
                Text(subtitle!, style: Theme.of(context).textTheme.bodyMedium),
              ],
            ],
          ),
        ),
      ),
    );
  }
}
```

### 4. StatefulWidget 有状态组件

```dart
class CounterPage extends StatefulWidget {
  final int initialValue;
  const CounterPage({super.key, this.initialValue = 0});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  late int _count;

  @override
  void initState() {
    super.initState();
    _count = widget.initialValue;   // 通过 widget 访问 Widget 的属性
  }

  @override
  void didUpdateWidget(CounterPage oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.initialValue != widget.initialValue) {
      _count = widget.initialValue;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Counter')),
      body: Center(child: Text('$_count', style: const TextStyle(fontSize: 48))),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() { _count++; }),
        child: const Icon(Icons.add),
      ),
    );
  }

  @override
  void dispose() {
    // 清理资源：控制器、订阅、定时器等
    super.dispose();
  }
}
```

**State 生命周期完整流程：**

```
createState()                    // StatefulWidget 被插入树时
    │
    ▼
initState()                      // 初始化状态（只一次）
    │
    ▼
didChangeDependencies()          // 依赖的 InheritedWidget 变化时
    │
    ▼
build()                          // 构建 Widget 子树
    │                             ↑
    │                             │ setState() / didUpdateWidget()
    │                             │ / didChangeDependencies()
    ▼                             │
didUpdateWidget()  ◄─────────────┘   // 父 Widget rebuild 且 canUpdate=true
    │
    ▼
deactivate()                     // 从树中移除（可能重新插入）
    │
    ▼
dispose()                        // 永久销毁
```

### 5. InheritedWidget 数据传递

```dart
class ThemeInheritedWidget extends InheritedWidget {
  final ThemeData themeData;
  final bool isDarkMode;
  final VoidCallback toggleTheme;

  const ThemeInheritedWidget({
    super.key,
    required this.themeData,
    required this.isDarkMode,
    required this.toggleTheme,
    required super.child,
  });

  // 子组件调用此方法获取数据
  static ThemeInheritedWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ThemeInheritedWidget>();
  }

  // 返回 true 时通知依赖此 Widget 的子组件 rebuild
  @override
  bool updateShouldNotify(ThemeInheritedWidget oldWidget) {
    return isDarkMode != oldWidget.isDarkMode;
  }
}

// 在上层包裹
ThemeInheritedWidget(
  themeData: theme,
  isDarkMode: isDark,
  toggleTheme: () => setState(() => isDark = !isDark),
  child: const MyApp(),
);

// 在任意子组件中获取
class ThemedButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final inherited = ThemeInheritedWidget.of(context)!;
    return ElevatedButton(
      onPressed: inherited.toggleTheme,
      child: Text(inherited.isDarkMode ? 'Light Mode' : 'Dark Mode'),
    );
  }
}
```

### 6. Keys 的作用与类型

```dart
// ValueKey：值相等即认为是同一个
ListView(
  children: items.map((item) => ListTile(
    key: ValueKey(item.id),
    title: Text(item.name),
  )).toList(),
);

// GlobalKey：全局唯一，可跨树访问 State
final GlobalKey<FormFieldState> formKey = GlobalKey();

Form(
  key: formKey,
  child: Column(children: [
    TextFormField(validator: (v) => v!.isEmpty ? '必填' : null),
    ElevatedButton(
      onPressed: () {
        if (formKey.currentState!.validate()) { /* 提交 */ }
      },
      child: const Text('Submit'),
    ),
  ]),
);

// ObjectKey：对象引用相等时是同一个
ObjectKey(myObject)
```

### 7. 常用基础 Widget

```dart
// ===== Text 文本 =====
Text(
  'Hello Flutter',
  style: TextStyle(
    fontSize: 20, fontWeight: FontWeight.bold, color: Colors.blue,
    letterSpacing: 1.2, decoration: TextDecoration.underline,
  ),
  textAlign: TextAlign.center,
  maxLines: 2,
  overflow: TextOverflow.ellipsis,
);

// 富文本
RichText(
  text: TextSpan(
    style: DefaultTextStyle.of(context).style,
    children: [
      TextSpan(text: 'Hello ', style: TextStyle(fontWeight: FontWeight.bold)),
      TextSpan(text: 'Flutter', style: TextStyle(color: Colors.blue)),
    ],
  ),
);

// ===== Image 图片 =====
Image.network(
  'https://example.com/photo.jpg',
  width: 200, height: 200, fit: BoxFit.cover,
  loadingBuilder: (context, child, progress) {
    if (progress == null) return child;
    return CircularProgressIndicator(
      value: progress.expectedTotalBytes != null
          ? progress.cumulativeBytesLoaded / progress.expectedTotalBytes! : null,
    );
  },
  errorBuilder: (context, error, stackTrace) {
    return const Icon(Icons.broken_image, size: 200);
  },
);

// 本地图片（需在 pubspec.yaml 声明 assets）
Image.asset('assets/images/logo.png', width: 100);

// ===== Container 容器 =====
Container(
  width: 200, height: 120,
  padding: const EdgeInsets.all(16),
  margin: const EdgeInsets.symmetric(horizontal: 20),
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8, offset: Offset(0, 4))],
    border: Border.all(color: Colors.grey.shade200),
  ),
  child: const Center(child: Text('Container')),
);

// ===== Card 卡片 =====
Card(
  elevation: 4,
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
  child: Padding(
    padding: const EdgeInsets.all(16),
    child: Column(children: [Text('Card Title'), Text('Card Content')]),
  ),
);

// ===== Scaffold 页面骨架 =====
Scaffold(
  appBar: AppBar(
    title: const Text('Page Title'),
    leading: IconButton(icon: const Icon(Icons.menu), onPressed: () {}),
    actions: [
      IconButton(icon: const Icon(Icons.search), onPressed: () {}),
      IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
    ],
  ),
  body: const Center(child: Text('Body')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  bottomNavigationBar: BottomNavigationBar(items: [
    BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
    BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
  ]),
  drawer: Drawer(child: ListView(children: [ListTile(title: Text('Item 1'))])),
);
```

### 8. 滚动 Widget

```dart
// ===== ListView 列表 =====
// ListView.builder：懒加载，适合长列表
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) => ListTile(
    leading: CircleAvatar(child: Text('$index')),
    title: Text('Item $index'),
    subtitle: Text('Description for item $index'),
    trailing: const Icon(Icons.chevron_right),
    onTap: () => _navigateToDetail(index),
  ),
);

// ListView.separated：带分割线
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (context, index) => const Divider(height: 1),
  itemBuilder: (context, index) => ItemTile(item: items[index]),
);

// ===== GridView 网格 =====
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    mainAxisSpacing: 8,
    crossAxisSpacing: 8,
    childAspectRatio: 0.75,
  ),
  itemCount: products.length,
  itemBuilder: (context, index) => ProductCard(product: products[index]),
);

// ===== CustomScrollView + Sliver =====
CustomScrollView(
  slivers: [
    SliverAppBar(
      expandedHeight: 200, floating: false, pinned: true,
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Detail'),
        background: Image.network(url, fit: BoxFit.cover),
      ),
    ),
    SliverGrid(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2, mainAxisSpacing: 8, crossAxisSpacing: 8,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => ImageCard(image: images[index]),
        childCount: images.length,
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: 20,
      ),
    ),
  ],
);
```

### 9. 输入 Widget

```dart
// ===== TextField 文本输入 =====
final TextEditingController _controller = TextEditingController();

TextField(
  controller: _controller,
  decoration: InputDecoration(
    labelText: '用户名', hintText: '请输入用户名',
    prefixIcon: const Icon(Icons.person),
    suffixIcon: IconButton(icon: const Icon(Icons.clear), onPressed: () => _controller.clear()),
    border: OutlineInputBorder(borderRadius: BorderRadius.circular(8)),
    filled: true, fillColor: Colors.grey.shade100,
  ),
  keyboardType: TextInputType.emailAddress,
  textInputAction: TextInputAction.next,
  onChanged: (value) => print('Input: $value'),
  onSubmitted: (value) => print('Submitted: $value'),
);

// ===== Form 表单 =====
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(children: [
        TextFormField(
          decoration: const InputDecoration(labelText: '邮箱'),
          validator: (v) => v!.isEmpty ? '请输入邮箱' : null,
        ),
        TextFormField(
          decoration: const InputDecoration(labelText: '密码'),
          obscureText: true,
          validator: (v) => v!.length < 6 ? '密码至少6位' : null,
        ),
        const SizedBox(height: 20),
        ElevatedButton(
          onPressed: () {
            if (_formKey.currentState!.validate()) { _submit(); }
          },
          child: const Text('登录'),
        ),
      ]),
    );
  }
}

// ===== Checkbox / Switch / Radio =====
CheckboxListTile(value: _checked, onChanged: (v) => setState(() => _checked = v!), title: const Text('同意用户协议'));
SwitchListTile(value: _switchVal, onChanged: (v) => setState(() => _switchVal = v), title: const Text('开启通知'));
RadioListTile<int>(value: 1, groupValue: _radioVal, onChanged: (v) => setState(() => _radioVal = v!), title: const Text('选项一'));

// ===== Slider =====
Slider(value: _sliderVal, min: 0, max: 100, divisions: 10, label: _sliderVal.round().toString(), onChanged: (v) => setState(() => _sliderVal = v));
```

### 10. 交互 Widget

```dart
// ===== GestureDetector 手势检测 =====
GestureDetector(
  onTap: () => print('Tapped'),
  onDoubleTap: () => print('Double tapped'),
  onLongPress: () => print('Long pressed'),
  onHorizontalDragEnd: (details) => print('Drag: ${details.primaryVelocity}'),
  child: Container(width: 200, height: 200, color: Colors.blue, child: const Center(child: Text('Tap Me'))),
);

// ===== Dismissible 滑动删除 =====
Dismissible(
  key: ValueKey(item.id),
  direction: DismissDirection.endToStart,
  background: Container(color: Colors.red, alignment: Alignment.centerRight, padding: const EdgeInsets.only(right: 20), child: const Icon(Icons.delete, color: Colors.white)),
  onDismissed: (direction) => _removeItem(item.id),
  child: ListTile(title: Text(item.title)),
);

// ===== Draggable 拖拽 =====
Draggable<int>(
  data: 42,
  feedback: Material(elevation: 4, child: Container(width: 80, height: 80, color: Colors.blue, child: const Center(child: Text('Drag')))),
  child: Container(width: 80, height: 80, color: Colors.blue, child: const Center(child: Text('Drag'))),
);

DragTarget<int>(
  builder: (context, candidateData, rejectedData) => Container(width: 150, height: 150, color: Colors.grey.shade200, child: const Center(child: Text('Drop here'))),
  onAccept: (data) => print('Received: $data'),
);
```

### 11. 对话框与底部弹窗

```dart
// ===== AlertDialog =====
showDialog(
  context: context,
  builder: (context) => AlertDialog(
    title: const Text('确认删除'),
    content: const Text('删除后不可恢复，确定要删除吗？'),
    actions: [
      TextButton(onPressed: () => Navigator.pop(context), child: const Text('取消')),
      TextButton(onPressed: () { _delete(); Navigator.pop(context); }, child: const Text('删除', style: TextStyle(color: Colors.red))),
    ],
  ),
);

// ===== BottomSheet =====
showModalBottomSheet(
  context: context,
  isScrollControlled: true,
  shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(20))),
  builder: (context) => Padding(
    padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
    child: Column(mainAxisSize: MainAxisSize.min, children: [
      const SizedBox(height: 8),
      Container(width: 40, height: 4, decoration: BoxDecoration(color: Colors.grey, borderRadius: BorderRadius.circular(2))),
      const Padding(padding: EdgeInsets.all(16), child: Text('Bottom Sheet')),
    ]),
  ),
);

// ===== SnackBar =====
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('操作成功'),
    duration: const Duration(seconds: 2),
    action: SnackBarAction(label: '撤销', onPressed: () => _undo()),
    behavior: SnackBarBehavior.floating,
  ),
);
```

### 12. Flutter 渲染管线

```
用户交互 / 状态变化 / 定时器
           │
           ▼
    ┌──────────────┐
    │  Build Phase  │ ← Widget.rebuild() / setState()
    │  构建Widget树  │   生成新的 Widget 树
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │  Layout Phase │ ← RenderObject.performLayout()
    │  计算布局尺寸  │   父约束向下传递，尺寸向上返回
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │  Paint Phase  │ ← RenderObject.paint()
    │  绘制到Canvas  │   生成 Layer 和 Picture
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │  Composite    │ ← SceneBuilder 合成
    │  合成与上屏    │   Skia/Impeller → GPU
    └──────────────┘

    整个流程需在 16ms 内完成（60fps）
```

**布局约束传递机制：**

```
父节点 ──约束──→ 子节点（最小/最大宽高）
1. 父向子传递约束（BoxConstraints）
2. 子在约束范围内决定自身尺寸
3. 子向父返回自身尺寸
4. 父决定子的位置（offset）

特殊约束：
- Tight 紧约束：min == max，子没有选择空间（如 SizedBox(w:100,h:100))
- Loose 松约束：min == 0，子可自由选择大小（如 Center 内的子组件）
- Unbounded 无限约束：max == double.infinity（如 ListView 的纵向约束）
```

### 13. 热重载（Hot Reload）原理

```
┌─────────────────┐
│  源码修改保存     │
└────────┬────────┘
         │
┌────────▼────────┐
│  增量编译(Dart)   │     只编译修改的库
└────────┬────────┘
         │
┌────────▼────────┐
│  Hot Reload (r)  │     保留 State
│  注入新Widget代码 │     触发 root widget rebuild
└─────────────────┘

Hot Reload 限制：
- 不能修改全局变量初始值（已初始化的不会重新执行）
- 不能修改 main() 函数
- 不能修改枚举定义
- 不能修改泛型类型参数
此时需要 Hot Restart（R）完全重启
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| setState 在 dispose 后调用 | 异步回调中未检查 mounted | `if (mounted) setState(() {...})` |
| Widget 重建导致状态丢失 | 没有 key 或 key 不稳定 | 使用 ValueKey 唯一标识 |
| ListView 嵌套报错 | 内部 ListView 无限高度 | 设 shrinkWrap:true 或用 NestedScrollView |
| InheritedWidget 获取为 null | 在 Widget 树之外调用 | 确保 context 在子树内 |
| Hot Reload 不生效 | 修改了全局变量/枚举 | 使用 Hot Restart (R) |
| 图片不显示 | 未在 pubspec.yaml 声明 | 添加 assets 路径并 flutter pub get |
| 键盘遮挡输入框 | 未适配键盘高度 | 用 SingleChildScrollView + padding |
| 性能卡顿 | build 中有耗时操作 | 抽离为 computed 值或用 compute() |
| const 无效 | 构造函数未标记 @immutable | 确保 Widget 构造函数参数为 final |
| dispose 后 controller 报错 | 控制器未正确释放 | 在 dispose 中 controller.dispose() |

### 最佳实践

- 优先使用 const 构造函数减少 Widget 重建开销
- Widget 拆分到最小粒度，限制 rebuild 范围
- 使用 const SizedBox() 代替 Container() 做占位
- 长列表用 ListView.builder，不用 ListView(children:[])
- 复杂列表用 Sliver 体系替代嵌套 ListView
- 及时 dispose 控制器（AnimationController/TextEditingController/StreamSubscription）
- 使用 flutter analyze 进行静态分析
- 图片使用 cached_network_image 做缓存
- 网络请求统一封装，避免分散在各 Widget 中
- 状态管理优先用 Riverpod/Provider，避免过度使用 setState
- 使用 Key 保证列表项状态正确
- 避免在 build 方法中创建对象，提取为成员变量或方法

## 面试题

**Q1: Flutter 的 Widget、Element、RenderObject 三棵树分别是什么？它们之间什么关系？**
> Widget 是不可变的 UI 配置描述，每次 rebuild 都生成新对象。Element 是 Widget 的可变实例，管理生命周期和父子关系，是 Widget 与 RenderObject 的桥梁。RenderObject 负责实际的测量（layout）和绘制（paint）。关系：Widget 创建对应的 Element，Element 通过 canUpdate（runtimeType 和 key 都相同则可复用）判断是否复用旧 Element；只有 RenderObjectWidget 的子类才创建 RenderObject，非渲染型 Widget（StatefulWidget、InheritedWidget）不创建 RenderObject。

**Q2: StatelessWidget 和 StatefulWidget 的区别是什么？State 的生命周期是怎样的？**
> StatelessWidget 的 build 方法根据构造参数生成 Widget 树，参数不变则不重建，适合纯展示组件。StatefulWidget 由 Widget（不可变）+ State（可变）组成，State 持有可变状态，调用 setState() 触发 rebuild。State 生命周期：createState → initState → didChangeDependencies → build →（setState/didUpdateWidget 循环触发 rebuild）→ deactivate → dispose。initState 只调一次，用于初始化；build 在每次状态变化时调用；dispose 用于清理资源。

**Q3: Flutter 的 Hot Reload 原理是什么？有哪些限制？**
> Hot Reload 通过 Dart VM 的热代码替换机制实现：修改源码后增量编译只编译修改的库，将新代码注入运行中的 Dart VM，触发 root widget rebuild，Element 树通过 diff 算法更新。关键是保留现有 State 对象，只替换 Widget 配置。限制：①不能修改全局变量初始值；②不能修改 main() 函数；③不能修改枚举和泛型定义；④新增 import 可能不生效。此时需 Hot Restart 完全重启。

**Q4: InheritedWidget 是什么？它是如何实现高效数据传递的？**
> InheritedWidget 是 Flutter 的数据向下传递机制，类似 React 的 Context。后代通过 `context.dependOnInheritedWidgetOfExactType<T>()` 获取数据并建立依赖关系。当 updateShouldNotify 返回 true 时，框架通知所有注册了依赖的子 Widget 调用 didChangeDependencies 并 rebuild。高效原因：①只有实际依赖的子 Widget 才 rebuild，而非整棵树；②查找是 O(1) 哈希表查找；③依赖注册在 Element 层，跳过中间不关心的 Widget。

**Q5: Key 在 Flutter 中有什么作用？什么场景下必须使用 Key？**
> Key 是 Widget 的身份标识，用于 Element 树 diff 时判断两个 Widget 是否"相同"。canUpdate 规则：runtimeType 相同且 key 相同 → 复用 Element。必须使用 Key 的场景：①同类型 Widget 在列表中交换位置时，没有 Key 会导致状态跟随位置而非数据；②StatefulWidget 从树的一个位置移到另一个位置时（如 Tab 切换），GlobalKey 可以保持 State 不丢失；③AnimatedSwitcher 中区分不同内容的切换动画。

**Q6: Flutter 的布局约束传递机制是什么？为什么说 Flutter 的布局是"父约束子"？**
> Flutter 布局采用"约束向下传递，尺寸向上返回，父级设置位置"的三步机制。父节点向子节点传递 BoxConstraints（最小/最大宽高），子节点在约束范围内决定自身尺寸并返回给父节点，父节点最终决定子节点的偏移位置。这是"父约束子"的设计，子节点不能超出父给定的约束范围。常见约束类型：Tight（紧约束，min==max，如 SizedBox）、Loose（松约束，min==0，如 Center）、Unbounded（无限约束，如 ListView 纵向）。理解约束机制是解决布局溢出（overflow）问题的关键。

**Q7: setState() 的工作流程是什么？为什么不能在 dispose 后调用？**
> setState 流程：①标记当前 Element 为 dirty；②下一帧时框架调用 Element 的 performRebuild() → Widget.build()；③生成新 Widget 子树，与旧子树 diff 更新。不能在 dispose 后调用的原因：dispose 后 Element 已从树中移除，不再有对应的 BuildContext，调用 setState 会抛出异常。解决方案：异步操作返回后检查 `if (mounted)` 再调用 setState，或在 dispose 中取消异步操作。

**Q8: Flutter 的渲染管线是怎样的？如何保证 60fps 流畅？**
> 渲染管线分四个阶段：① Build — Widget 树构建；② Layout — RenderObject 布局计算（约束向下，尺寸向上）；③ Paint — RenderObject 绘制到 Canvas 生成 Layer；④ Composite — 合成 Layer 上屏（Skia/Impeller → GPU）。保证 60fps 的关键是每帧在 16ms 内完成全部四个阶段。优化手段：①减少 build 范围（const Widget、拆分小组件）；②使用 RepaintBoundary 隔离重绘区域；③列表用 builder 懒加载；④动画用 AnimatedWidget 避免 setState；⑤耗时计算用 Isolate 移到后台线程。

---

**相关链接：**
- [[Dart语言核心]]
- [[Flutter布局与样式]]
- [[Flutter状态管理]]
- [[React核心]]
- [[React Native移动开发]]

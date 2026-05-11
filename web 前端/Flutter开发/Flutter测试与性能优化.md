---
tags:
  - Web前端
  - Flutter
  - 测试
  - 性能优化
date: 2026-05-11
status: 已完成
difficulty: 中高
---

# Flutter测试与性能优化

## What — 是什么

> Flutter 测试与性能优化是保障 Flutter 应用质量和流畅体验的两大支柱：测试确保代码正确性和回归防护，性能优化确保渲染流畅和资源高效。

**核心概念：**

- **测试金字塔**：Unit Test → Widget Test → Integration Test，由多到少的比例分布
- **单元测试**：验证函数/类的逻辑正确性，速度最快，隔离性最强
- **Widget 测试**：验证单个 Widget 的渲染和交互行为，Flutter 特色测试层级
- **集成测试**：验证完整应用流程，在真机/模拟器上运行，最接近用户真实体验
- **性能优化**：围绕 16ms/帧（60fps）的渲染预算，从 build/layout/paint 三个阶段消减耗时
- **DevTools**：Flutter 官方性能分析套件，包含 Performance、Memory、CPU Profiler 等面板

**Flutter 测试金字塔：**

```
            ┌──────────────────┐
            │  Integration Test │  ← 少量，端到端验证
            │   (integration_   │
            │      test 包)     │
            ├──────────────────┤
            │   Widget Test    │  ← 适中，UI 交互验证
            │   (testWidgets)  │
            ├──────────────────┤
            │    Unit Test     │  ← 大量，逻辑正确性验证
            │   (test 包)      │
            └──────────────────┘
```

**Flutter 渲染流水线：**

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  动画帧   │ →  │  Build   │ →  │  Layout  │ →  │  Paint   │ → 合成 → 上屏
│  Vsync    │    │ (构建树)  │    │ (计算尺寸 │    │ (绘制指令 │
│  信号触发  │    │          │    │  和位置)  │    │  列表)   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                  ← 16ms 预算（60fps）→
                  ← 8ms 预算（120fps）→
```

**关键特性：**

- Flutter 三层测试体系与 Web 前端测试理念相通但 API 不同
- Widget 测试是 Flutter 独有的测试层级，介于单元和集成之间
- 性能优化核心：减少不必要的 rebuild、控制 layout/paint 范围、合理利用线程模型
- Dart 的 Isolate 模型决定了计算密集型任务需要独立处理

## Why — 为什么

**适用场景：**

- 所有 Flutter 项目的质量保障与性能保障
- CI/CD 流水线中的自动化测试门禁
- 上线前的性能回归检测
- 大型团队协作时的代码质量防线
- 复杂动画/列表场景的流畅性保障

**Flutter 测试层级对比：**

| 维度 | Unit Test | Widget Test | Integration Test |
|------|-----------|-------------|------------------|
| 运行环境 | Dart VM | Flutter 测试引擎 | 真机/模拟器 |
| 速度 | 极快（毫秒级） | 快（百毫秒级） | 慢（秒级） |
| 覆盖范围 | 单个函数/类 | 单个 Widget | 完整应用流程 |
| 是否需要 UI | 否 | 虚拟 UI | 真实 UI |
| 依赖 | mock 为主 | 部分真实 + mock | 真实依赖 |
| 推荐比例 | 70% | 20% | 10% |
| 包 | `test` | `flutter_test` | `integration_test` |

**Flutter vs Web 前端性能优化对比：**

| 维度 | Flutter | Web 前端 |
|------|---------|---------|
| 渲染引擎 | Skia/Impeller 自绘 | 浏览器引擎 |
| 帧预算 | 16ms（60fps） | 16ms（60fps） |
| 线程模型 | UI 线程 + Raster 线程 + IO 线程 | 主线程 + 合成线程 |
| 性能工具 | DevTools、Performance Overlay | DevTools、Lighthouse |
| 常见瓶颈 | 不必要 rebuild、重 layout | 重排重绘、JS 执行阻塞 |
| 列表优化 | ListView.builder | 虚拟滚动 |
| 状态管理优化 | Selector/buildWhen | React.memo/useSelector |

**优缺点：**

- ✅ Flutter 测试优点：
  - 三层测试体系层次分明，各司其职
  - Widget 测试可在不启动模拟器的情况下验证 UI
  - 集成测试可直接生成截图和性能指标
  - Golden 测试可自动对比 UI 回归
- ❌ Flutter 测试缺点：
  - Widget 测试 API 学习成本较高
  - 集成测试速度慢，调试困难
  - 网络请求 Mock 配置较繁琐
- ✅ Flutter 性能优化优点：
  - 自绘引擎绕过平台 UI 限制，优化空间大
  - DevTools 工具链完善，可视化分析能力强
  - `const` 构造函数在编译期优化 rebuild
- ❌ Flutter 性能优化缺点：
  - 120fps 设备对优化要求更高（8ms 帧预算）
  - Web 端性能受 CanvasKit/HTML 渲染器限制
  - 部分 Flutter 框架层优化需要深入理解源码

## How — 怎么用

### 快速上手

**项目测试配置：**

```yaml
# pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  mockito: ^5.4.4        # Mock 框架
  build_runner: ^2.4.9   # 代码生成（mockito 需要）
  http_mock_adapter: ^0.6.1  # Dio HTTP Mock
  golden_toolkit: ^0.15.0    # Golden 测试
```

**测试目录结构：**

```
project/
├── lib/
│   ├── models/
│   ├── services/
│   ├── widgets/
│   └── pages/
├── test/                     ← 单元测试 + Widget 测试
│   ├── models/
│   ├── services/
│   ├── widgets/
│   └── pages/
├── integration_test/         ← 集成测试
│   └── app_test.dart
├── test_goldens/             ← Golden 测试（可选）
└── pubspec.yaml
```

### 一、单元测试

**基础结构：group/test/setUp/tearDown**

```dart
// test/models/user_test.dart
import 'package:test/test.dart';
import 'package:my_app/models/user.dart';

void main() {
  // group 分组：将相关测试组织在一起
  group('User', () {
    // setUp：每个测试前执行，用于初始化
    late User user;

    setUp(() {
      user = User(id: '1', name: 'Alice', age: 25);
    });

    // tearDown：每个测试后执行，用于清理
    tearDown(() {
      // 清理资源、重置状态
    });

    test('should have correct initial values', () {
      expect(user.id, '1');
      expect(user.name, 'Alice');
      expect(user.age, 25);
    });

    test('should copy with new values', () {
      final updated = user.copyWith(name: 'Bob', age: 30);

      expect(updated.id, '1');      // 未修改的字段保持不变
      expect(updated.name, 'Bob');
      expect(updated.age, 30);
    });

    test('should serialize to JSON', () {
      final json = user.toJson();

      expect(json, isA<Map<String, dynamic>>());
      expect(json['name'], 'Alice');
    });

    test('should deserialize from JSON', () {
      final json = {'id': '2', 'name': 'Charlie', 'age': 28};
      final fromJson = User.fromJson(json);

      expect(fromJson.name, 'Charlie');
      expect(fromJson.age, 28);
    });
  });
}
```

**Matchers（匹配器）大全：**

```dart
import 'package:test/test.dart';

void main() {
  test('常用 Matchers', () {
    // 相等判断
    expect(1 + 1, equals(2));
    expect(1 + 1, 2); // equals 可省略

    // 类型判断
    expect('hello', isA<String>());
    expect(42, isA<int>());

    // 数值比较
    expect(0.1 + 0.2, closeTo(0.3, 0.0001)); // 浮点数近似
    expect(10, greaterThan(5));
    expect(5, lessThanOrEqualTo(5));

    // 集合
    expect([1, 2, 3], contains(2));
    expect([1, 2, 3], containsAll([1, 3]));
    expect([1, 2, 3], hasLength(3));
    expect([], isEmpty);
    expect([1, 2], isNotEmpty);

    // 字符串
    expect('hello world', contains('world'));
    expect('hello world', startsWith('hello'));
    expect('hello world', endsWith('world'));
    expect('hello world', matches(RegExp(r'hel+o')));

    // 布尔与空
    expect(true, isTrue);
    expect(false, isFalse);
    expect(null, isNull);
    expect('hello', isNotNull);

    // 异常
    expect(() => throw FormatException('bad'), throwsFormatException);
    expect(() => throw StateError('err'), throwsStateError);
    expect(
      () => throw Exception('custom'),
      throwsA(isA<Exception>().having((e) => e.toString(), 'message', contains('custom'))),
    );
  });

  test('组合 Matchers', () {
    // allOf：所有条件都满足
    expect(5, allOf(greaterThan(0), lessThan(10)));

    // anyOf：任一条件满足
    expect(15, anyOf(lessThan(10), greaterThan(12)));

    // isNot：取反
    expect(5, isNot(equals(10)));
  });
}
```

**Mockito 单元测试：**

```dart
// lib/services/user_service.dart
import 'package:my_app/models/user.dart';
import 'package:my_app/api/api_client.dart';

class UserService {
  final ApiClient apiClient;

  UserService(this.apiClient);

  Future<User> getUser(String id) async {
    final json = await apiClient.get('/users/$id');
    return User.fromJson(json);
  }

  Future<List<User>> searchUsers(String keyword) async {
    final list = await apiClient.get('/users?keyword=$keyword');
    return (list as List).map((e) => User.fromJson(e)).toList();
  }

  Future<bool> deleteUser(String id) async {
    await apiClient.delete('/users/$id');
    return true;
  }
}
```

```dart
// lib/api/api_client.dart — 需要被 mock 的类
abstract class ApiClient {
  Future<dynamic> get(String path);
  Future<dynamic> post(String path, {Map<String, dynamic>? body});
  Future<dynamic> put(String path, {Map<String, dynamic>? body});
  Future<dynamic> delete(String path);
}
```

```dart
// test/services/user_service_test.dart
import 'package:test/test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:my_app/services/user_service.dart';
import 'package:my_app/api/api_client.dart';

// 使用 @GenerateNiceMocks 注解生成 mock 类
@GenerateNiceMocks([MockSpec<ApiClient>()])
import 'user_service_test.mocks.dart'; // 生成的 mock 文件

void main() {
  late MockApiClient mockApiClient;
  late UserService userService;

  setUp(() {
    mockApiClient = MockApiClient();
    userService = UserService(mockApiClient);
  });

  group('UserService', () {
    test('getUser returns User on success', () async {
      // stub：当调用 get('/users/1') 时返回预设数据
      when(mockApiClient.get('/users/1')).thenAnswer(
        (_) async => {'id': '1', 'name': 'Alice', 'age': 25},
      );

      final user = await userService.getUser('1');

      expect(user.name, 'Alice');
      expect(user.age, 25);
      // 验证方法被调用了 1 次，且参数正确
      verify(mockApiClient.get('/users/1')).called(1);
    });

    test('getUser throws on API error', () async {
      when(mockApiClient.get('/users/999')).thenThrow(
        Exception('User not found'),
      );

      expect(
        () => userService.getUser('999'),
        throwsA(isA<Exception>()),
      );
    });

    test('searchUsers returns list of Users', () async {
      when(mockApiClient.get('/users?keyword=ali')).thenAnswer(
        (_) async => [
          {'id': '1', 'name': 'Alice', 'age': 25},
          {'id': '2', 'name': 'Ali', 'age': 30},
        ],
      );

      final users = await userService.searchUsers('ali');

      expect(users, hasLength(2));
      expect(users[0].name, 'Alice');
    });

    test('deleteUser calls API correctly', () async {
      // void 返回值用 thenAnswer
      when(mockApiClient.delete('/users/1')).thenAnswer(
        (_) async => null,
      );

      final result = await userService.deleteUser('1');

      expect(result, isTrue);
      verify(mockApiClient.delete('/users/1')).called(1);
    });
  });
}
```

生成 mock 文件的命令：

```bash
# 运行 build_runner 生成 .mocks.dart 文件
dart run build_runner build --delete-conflicting-outputs
```

**手写 Mock（简单场景）：**

```dart
// 对于简单接口，可以手写 mock 而不用代码生成
class MockSharedPreferences extends Mock implements SharedPreferences {}

void main() {
  test('手写 Mock 示例', () async {
    final prefs = MockSharedPreferences();

    when(prefs.getString('token')).thenReturn('fake_token');
    when(prefs.setBool('dark_mode', true)).thenAnswer((_) async => true);

    expect(prefs.getString('token'), 'fake_token');
    await prefs.setBool('dark_mode', true);

    verify(prefs.getString('token')).called(1);
    verify(prefs.setBool('dark_mode', true)).called(1);
  });
}
```

### 二、Widget 测试

**基础 Widget 测试：testWidgets + Finder + WidgetTester**

```dart
// test/widgets/counter_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/counter.dart';

void main() {
  group('CounterWidget', () {
    testWidgets('初始显示 0', (WidgetTester tester) async {
      // pumpWidget：渲染 Widget 到测试环境
      await tester.pumpWidget(
        const MaterialApp(home: CounterWidget()),
      );

      // find.text：通过文本查找 Widget
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
    });

    testWidgets('点击按钮递增', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(home: CounterWidget()),
      );

      // find.byType：通过类型查找 Widget
      final incrementButton = find.byType(FloatingActionButton);

      // tap：模拟点击
      await tester.tap(incrementButton);

      // pump：触发帧刷新，让状态更新反映到 UI
      await tester.pump();

      expect(find.text('1'), findsOneWidget);
    });

    testWidgets('多次点击递增', (WidgetTester tester) async {
      await tester.pumpWidget(
        const MaterialApp(home: CounterWidget()),
      );

      final button = find.byIcon(Icons.add);
      await tester.tap(button);
      await tester.pump();
      await tester.tap(button);
      await tester.pump();

      expect(find.text('2'), findsOneWidget);
    });
  });
}
```

**Finder 查找器详解：**

```dart
void main() {
  testWidgets('各种 Finder 用法', (WidgetTester tester) async {
    await tester.pumpWidget(const MyApp());

    // ---- 基础查找 ----

    // 按文本查找
    find.text('Hello');

    // 按类型查找
    find.byType(ElevatedButton);

    // 按 Key 查找
    find.byKey(const Key('submit_button'));

    // 按图标查找
    find.byIcon(Icons.search);

    // ---- 高级查找 ----

    // 按 Widget 特征查找
    find.byWidgetPredicate((widget) {
      return widget is Text && widget.data?.startsWith('Item') == true;
    });

    // 按语义查找（推荐：更接近用户视角）
    find.bySemanticsLabel('Submit');

    // 查找祖先/后代
    find.descendant(
      of: find.byType(ListView),    // 在 ListView 内部查找
      matching: find.byType(ListTile),
    );

    // 查找子 Widget
    find.childWidget(
      of: find.byType(AppBar),
      matching: find.byType(IconButton),
    );

    // ---- 组合查找 ----

    // 从结果中取第 N 个
    find.byType(ListTile).at(0); // 第一个 ListTile

    // 命中次数断言
    expect(find.byType(ListTile), findsNWidgets(5));    // 精确 5 个
    expect(find.text('Error'), findsNothing);           // 0 个
    expect(find.byType(Scaffold), findsOneWidget);      // 精确 1 个
    expect(find.byType(Text), findsWidgets);            // 至少 1 个
  });
}
```

**pump 与 pumpAndSettle 详解：**

```dart
void main() {
  testWidgets('pump 系列方法', (WidgetTester tester) async {
    await tester.pumpWidget(const MyApp());

    // pump()：触发一帧刷新
    // 适用于同步状态更新后的 UI 刷新
    await tester.tap(find.text('Click'));
    await tester.pump();

    // pump(Duration)：推进时间并触发帧刷新
    // 适用于动画、定时器等异步操作
    await tester.pump(const Duration(milliseconds: 500));

    // pumpAndSettle()：持续 pump 直到没有新的帧需要渲染
    // 适用于等待动画完成、页面跳转完成等场景
    // 默认超时 10 秒
    await tester.tap(find.text('Navigate'));
    await tester.pumpAndSettle();

    // pumpAndSettle 自定义超时
    await tester.pumpAndSettle(const Duration(seconds: 5));
  });

  testWidgets('动画测试示例', (WidgetTester tester) async {
    await tester.pumpWidget(
      const MaterialApp(home: AnimatedContainerDemo()),
    );

    // 触发动画
    await tester.tap(find.byType(GestureDetector));
    await tester.pump();

    // 验证动画中间状态
    await tester.pump(const Duration(milliseconds: 100));
    // 可以在此处检查动画的中间状态

    // 等待动画完成
    await tester.pumpAndSettle();
  });
}
```

**复杂 Widget 测试实战：**

```dart
// lib/widgets/login_form.dart
class LoginForm extends StatefulWidget {
  final Future<void> Function(String email, String password) onLogin;
  const LoginForm({super.key, required this.onLogin});

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;
  String? _error;

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() { _isLoading = true; _error = null; });

    try {
      await widget.onLogin(_emailController.text, _passwordController.text);
    } catch (e) {
      setState(() { _error = e.toString(); });
    } finally {
      if (mounted) setState(() { _isLoading = false; });
    }
  }

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            key: const Key('email_field'),
            controller: _emailController,
            decoration: const InputDecoration(labelText: 'Email'),
            validator: (v) => v?.contains('@') == true ? null : 'Invalid email',
          ),
          TextFormField(
            key: const Key('password_field'),
            controller: _passwordController,
            decoration: const InputDecoration(labelText: 'Password'),
            obscureText: true,
            validator: (v) => (v != null && v.length >= 6) ? null : 'Min 6 chars',
          ),
          if (_error != null)
            Text(_error!, key: const Key('error_message')),
          if (_isLoading)
            const CircularProgressIndicator(key: Key('loading_indicator')),
          ElevatedButton(
            key: const Key('login_button'),
            onPressed: _isLoading ? null : _submit,
            child: const Text('Login'),
          ),
        ],
      ),
    );
  }
}
```

```dart
// test/widgets/login_form_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/login_form.dart';

void main() {
  group('LoginForm', () {
    testWidgets('显示表单字段和按钮', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(home: LoginForm(onLogin: (_, __) async {})),
      );

      expect(find.byKey(const Key('email_field')), findsOneWidget);
      expect(find.byKey(const Key('password_field')), findsOneWidget);
      expect(find.byKey(const Key('login_button')), findsOneWidget);
      expect(find.byKey(const Key('loading_indicator')), findsNothing);
    });

    testWidgets('邮箱格式验证', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(home: LoginForm(onLogin: (_, __) async {})),
      );

      // 输入无效邮箱
      await tester.enterText(find.byKey(const Key('email_field')), 'invalid');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pump();

      expect(find.text('Invalid email'), findsOneWidget);
    });

    testWidgets('密码长度验证', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(home: LoginForm(onLogin: (_, __) async {})),
      );

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), '12345');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pump();

      expect(find.text('Min 6 chars'), findsOneWidget);
    });

    testWidgets('登录成功调用 onLogin', (WidgetTester tester) async {
      String? capturedEmail;
      String? capturedPassword;

      await tester.pumpWidget(
        MaterialApp(
          home: LoginForm(
            onLogin: (email, password) async {
              capturedEmail = email;
              capturedPassword = password;
            },
          ),
        ),
      );

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), '123456');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pump();

      expect(capturedEmail, 'test@example.com');
      expect(capturedPassword, '123456');
    });

    testWidgets('登录中显示加载状态', (WidgetTester tester) async {
      // 使用 Completer 控制异步完成时机
      final completer = Completer<void>();

      await tester.pumpWidget(
        MaterialApp(
          home: LoginForm(onLogin: (_, __) => completer.future),
        ),
      );

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), '123456');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pump(); // 触发状态更新

      // 验证加载状态
      expect(find.byKey(const Key('loading_indicator')), findsOneWidget);
      // 按钮应被禁用
      expect(find.byKey(const Key('login_button')).hitTestable(), findsNothing);

      // 完成异步操作
      completer.complete();
      await tester.pump(); // 让 FutureBuilder/state 更新

      // 加载状态消失
      expect(find.byKey(const Key('loading_indicator')), findsNothing);
    });

    testWidgets('登录失败显示错误', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: LoginForm(
            onLogin: (_, __) async => throw Exception('Network error'),
          ),
        ),
      );

      await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
      await tester.enterText(find.byKey(const Key('password_field')), '123456');
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle(); // 等待异步完成

      expect(find.byKey(const Key('error_message')), findsOneWidget);
      expect(find.textContaining('Network error'), findsOneWidget);
    });
  });
}
```

### 三、集成测试

**集成测试配置与编写：**

```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  // 初始化集成测试 binding
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('App 集成测试', () {
    testWidgets('完整登录流程', (WidgetTester tester) async {
      // 启动应用
      app.main();
      await tester.pumpAndSettle();

      // 1. 验证登录页显示
      expect(find.text('Welcome'), findsOneWidget);

      // 2. 输入凭据
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );

      // 3. 点击登录
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle(const Duration(seconds: 5));

      // 4. 验证跳转到首页
      expect(find.text('Dashboard'), findsOneWidget);
      expect(find.byType(BottomNavigationBar), findsOneWidget);
    });

    testWidgets('导航切换页面', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle();

      // 先登录
      await _login(tester);

      // 切换到 Settings 页
      await tester.tap(find.text('Settings'));
      await tester.pumpAndSettle();

      expect(find.text('Settings'), findsWidgets);

      // 切换回 Home 页
      await tester.tap(find.text('Home'));
      await tester.pumpAndSettle();
    });

    testWidgets('下拉刷新', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle();
      await _login(tester);

      // 模拟下拉刷新
      await tester.fling(
        find.byType(RefreshIndicator),
        const Offset(0, 300),
        1000,
      );
      await tester.pumpAndSettle();
    });
  });
}

// 辅助方法：登录
Future<void> _login(WidgetTester tester) async {
  await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
  await tester.enterText(find.byKey(const Key('password_field')), 'password123');
  await tester.tap(find.byKey(const Key('login_button')));
  await tester.pumpAndSettle(const Duration(seconds: 5));
}
```

**运行集成测试：**

```bash
# 在 Android 模拟器/真机上运行
flutter test integration_test/app_test.dart

# 指定设备运行
flutter test integration_test/app_test.dart -d <device_id>

# 生成截图
flutter test integration_test/app_test.dart --screenshot=/path/to/dir

# 多设备并行运行
flutter test integration_test/app_test.dart -d all
```

**集成测试中的性能指标采集：**

```dart
// integration_test/performance_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  final binding = IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('Scrolling performance', (WidgetTester tester) async {
    app.main();
    await tester.pumpAndSettle();

    // 记录性能指标
    binding.traceAction(
      () async {
        // 模拟滚动操作
        final listFinder = find.byType(ListView);

        // 执行多次滚动
        for (int i = 0; i < 5; i++) {
          await tester.fling(listFinder, const Offset(0, -500), 3000);
          await tester.pumpAndSettle();
        }

        // 滚回顶部
        await tester.fling(listFinder, const Offset(0, 500), 3000);
        await tester.pumpAndSettle();
      },
      reportKey: 'scroll_performance',
    );
  });

  testWidgets('页面切换性能', (WidgetTester tester) async {
    app.main();
    await tester.pumpAndSettle();

    binding.traceAction(
      () async {
        // 快速切换页面 5 次
        for (int i = 0; i < 5; i++) {
          await tester.tap(find.text('Profile'));
          await tester.pumpAndSettle();
          await tester.tap(find.text('Home'));
          await tester.pumpAndSettle();
        }
      },
      reportKey: 'navigation_performance',
    );
  });
}
```

### 四、Golden 测试（截图对比）

```dart
// test_goldens/login_page_golden_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:golden_toolkit/golden_toolkit.dart';
import 'package:my_app/pages/login_page.dart';

void main() {
  // 加载字体（Golden 测试需要）
  setUp(() async {
    await loadAppFonts();
  });

  group('LoginPage Golden Tests', () {
    // 多设备尺寸对比
    multiScreenTest(
      'LoginPage renders correctly',
      screens: [
        const IPhone6(),     // 375x667
        const IPhone11(),    // 414x896
        const IPad(),        // 768x1024
      ],
      (WidgetTester tester, Size size) async {
        await tester.pumpWidgetBuilder(
          const MaterialApp(home: LoginPage()),
          surfaceSize: size,
        );

        await screenMatchesGolden(
          tester,
          'login_page_${size.width.toInt()}x${size.height.toInt()}',
        );
      },
    );

    // 单设备状态对比
    testGoldens('LoginPage states', (WidgetTester tester) async {
      // 默认状态
      await tester.pumpWidgetBuilder(
        const MaterialApp(home: LoginPage()),
      );
      await screenMatchesGolden(tester, 'login_page_default');

      // 带错误状态
      await tester.pumpWidgetBuilder(
        MaterialApp(
          home: LoginPage(initialError: 'Invalid credentials'),
        ),
      );
      await screenMatchesGolden(tester, 'login_page_error');
    });
  });
}
```

**运行 Golden 测试：**

```bash
# 首次运行生成 golden 文件（会失败，因为没有参考图）
flutter test test_goldens/ --update-goldens

# 后续运行自动对比
flutter test test_goldens/

# 更新 golden 文件（UI 有意变更时）
flutter test test_goldens/ --update-goldens
```

### 五、网络请求 Mock

**使用 http_mock_adapter Mock Dio：**

```dart
// test/services/api_service_test.dart
import 'package:dio/dio.dart';
import 'package:http_mock_adapter/http_mock_adapter.dart';
import 'package:test/test.dart';
import 'package:my_app/services/api_service.dart';

void main() {
  late Dio dio;
  late DioAdapter dioAdapter;
  late ApiService apiService;

  setUp(() {
    dio = Dio(BaseOptions(baseUrl: 'https://api.example.com'));
    dioAdapter = DioAdapter(dio: dio);
    apiService = ApiService(dio);
  });

  group('ApiService', () {
    test('fetchUsers returns list', () async {
      dioAdapter.onGet(
        '/users',
        (server) => server.reply(
          200,
          [
            {'id': 1, 'name': 'Alice'},
            {'id': 2, 'name': 'Bob'},
          ],
          delay: const Duration(milliseconds: 100),
        ),
      );

      final users = await apiService.fetchUsers();

      expect(users, hasLength(2));
      expect(users[0].name, 'Alice');
    });

    test('fetchUsers handles 404', () async {
      dioAdapter.onGet(
        '/users',
        (server) => server.reply(404, {'error': 'Not found'}),
      );

      expect(
        () => apiService.fetchUsers(),
        throwsA(isA<DioException>()),
      );
    });

    test('createUser sends correct body', () async {
      dioAdapter.onPost(
        '/users',
        (server) => server.reply(201, {'id': 3, 'name': 'Charlie'}),
        data: {'name': 'Charlie'},
      );

      final user = await apiService.createUser('Charlie');

      expect(user.name, 'Charlie');
      expect(user.id, 3);
    });

    test('handles network timeout', () async {
      dioAdapter.onGet(
        '/users',
        (server) => server.throws(
          DioException.connectionTimeout(
            requestOptions: RequestOptions(path: '/users'),
            timeout: const Duration(seconds: 10),
          ),
        ),
      );

      expect(
        () => apiService.fetchUsers(),
        throwsA(isA<DioException>().having(
          (e) => e.type, 'type', DioExceptionType.connectionTimeout,
        )),
      );
    });
  });
}
```

**使用 MockWebServer Mock HTTP 请求（非 Dio 场景）：**

```dart
import 'package:http/http.dart' as http;
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:test/test.dart';

@GenerateNiceMocks([MockSpec<http.Client>()])
import 'api_client_test.mocks.dart';

void main() {
  group('HttpClient', () {
    test('get 成功', () async {
      final mockClient = MockClient();

      when(mockClient.get(Uri.parse('https://api.example.com/users')))
          .thenAnswer((_) async => http.Response(
                '[{"id": 1, "name": "Alice"}]',
                200,
                headers: {'content-type': 'application/json'},
              ));

      final response = await mockClient.get(Uri.parse('https://api.example.com/users'));

      expect(response.statusCode, 200);
      expect(response.body, contains('Alice'));
    });

    test('get 失败', () async {
      final mockClient = MockClient();

      when(mockClient.get(Uri.parse('https://api.example.com/users')))
          .thenAnswer((_) async => http.Response('Not Found', 404));

      final response = await mockClient.get(Uri.parse('https://api.example.com/users'));

      expect(response.statusCode, 404);
    });
  });
}
```

**Repository 层 Mock 测试：**

```dart
// test/repositories/user_repository_test.dart
import 'package:test/test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:my_app/repositories/user_repository.dart';
import 'package:my_app/services/api_service.dart';
import 'package:my_app/services/cache_service.dart';

@GenerateNiceMocks([MockSpec<ApiService>(), MockSpec<CacheService>()])
import 'user_repository_test.mocks.dart';

void main() {
  late MockApiService mockApiService;
  late MockCacheService mockCacheService;
  late UserRepository repository;

  setUp(() {
    mockApiService = MockApiService();
    mockCacheService = MockCacheService();
    repository = UserRepository(mockApiService, mockCacheService);
  });

  test('getUser 先读缓存再请求网络', () async {
    // 缓存未命中
    when(mockCacheService.get('user_1')).thenReturn(null);
    // 网络请求返回
    when(mockApiService.getUser('1')).thenAnswer(
      (_) async => User(id: '1', name: 'Alice'),
    );

    final user = await repository.getUser('1');

    expect(user.name, 'Alice');
    verify(mockCacheService.get('user_1')).called(1);      // 读缓存
    verify(mockCacheService.set('user_1', any)).called(1);  // 写缓存
    verify(mockApiService.getUser('1')).called(1);           // 网络请求
  });

  test('getUser 缓存命中不请求网络', () async {
    when(mockCacheService.get('user_1')).thenReturn(
      {'id': '1', 'name': 'Cached Alice'},
    );

    final user = await repository.getUser('1');

    expect(user.name, 'Cached Alice');
    verifyNever(mockApiService.getUser(any)); // 不应调用网络
  });
}
```

### 六、测试覆盖率

```bash
# 运行测试并生成覆盖率数据
flutter test --coverage

# 生成的 coverage/lcov.info 是 lcov 格式
# 使用 lcov 工具生成 HTML 报告（需安装 lcov）

# Linux/macOS
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Windows（使用 choco 安装 lcov 或使用在线工具）
# 也可以使用 package:coverage 结合 dart 工具处理

# 排除不需要覆盖的文件
lcov --remove coverage/lcov.info \
  'lib/**/*.g.dart' \
  'lib/**/*.freezed.dart' \
  'lib/**/generated/**' \
  -o coverage/lcov_filtered.info

genhtml coverage/lcov_filtered.info -o coverage/html
```

**CI 中配置覆盖率门禁：**

```yaml
# .github/workflows/test.yml
name: Flutter Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests with coverage
        run: flutter test --coverage

      - name: Filter coverage
        run: |
          lcov --remove coverage/lcov.info \
            'lib/**/*.g.dart' \
            'lib/**/*.freezed.dart' \
            -o coverage/lcov_filtered.info

      - name: Check coverage threshold
        run: |
          # 使用 dart 脚本检查覆盖率
          dart run tools/check_coverage.dart --min-coverage 80

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: coverage/lcov_filtered.info
```

**覆盖率检查脚本：**

```dart
// tools/check_coverage.dart
import 'dart:io';

void main(List<String> args) {
  final minCoverage = _parseMinCoverage(args);
  final lcovFile = File('coverage/lcov_filtered.info');

  if (!lcovFile.existsSync()) {
    stderr.writeln('Coverage file not found');
    exit(1);
  }

  final content = lcovFile.readAsStringSync();
  final lines = content.split('\n');

  int totalLines = 0;
  int coveredLines = 0;

  for (final line in lines) {
    if (line.startsWith('DA:')) {
      totalLines++;
      final parts = line.split(',');
      if (parts.length >= 2 && int.tryParse(parts[1]) != null && int.parse(parts[1]) > 0) {
        coveredLines++;
      }
    }
  }

  final coverage = totalLines > 0 ? (coveredLines / totalLines * 100) : 0.0;

  print('Total lines: $totalLines');
  print('Covered lines: $coveredLines');
  print('Coverage: ${coverage.toStringAsFixed(1)}%');

  if (coverage < minCoverage) {
    stderr.writeln('Coverage ${coverage.toStringAsFixed(1)}% is below threshold ${minCoverage}%');
    exit(1);
  }

  print('Coverage check passed!');
}

double _parseMinCoverage(List<String> args) {
  for (int i = 0; i < args.length - 1; i++) {
    if (args[i] == '--min-coverage') {
      return double.tryParse(args[i + 1]) ?? 80.0;
    }
  }
  return 80.0;
}
```

### 七、性能分析工具

**DevTools Performance 面板：**

```bash
# 启动 DevTools
flutter pub global activate devtools
flutter pub global run devtools

# 或者在运行应用时自动启动
flutter run --profile
# 然后在浏览器中打开 DevTools URL
```

**Performance Overlay（性能覆盖层）：**

```dart
// 在 MaterialApp 中开启
MaterialApp(
  showPerformanceOverlay: true, // 显示 FPS 和 GPU 使用率
  home: MyHomePage(),
);

// 或在代码中动态切换
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      showPerformanceOverlay: kDebugMode, // 仅 Debug 模式显示
      home: Scaffold(
        appBar: AppBar(title: const Text('Performance Demo')),
        body: const Center(child: Text('Hello')),
      ),
    );
  }
}
```

```
Performance Overlay 显示说明：
┌──────────────────────────────────┐
│  UI FPS     │  GPU FPS          │
│  ████████░░ │  ██████████       │
│  58 fps     │  60 fps           │  ← 绿色=良好（>50fps）
│             │                    │
│  UI Thread  │  Raster Thread    │
│  ██████░░░░ │  ████░░░░░░       │  ← 红色=卡顿（<30fps）
└──────────────────────────────────┘
UI Thread:   执行 Dart 代码（build/layout/状态更新）
Raster Thread: 执行 GPU 指令（绘制/合成/上屏）
```

**Timeline 追踪：**

```dart
import 'dart:developer' as developer;

class TimelineExample {
  // 手动添加 Timeline 事件，用于分析特定代码段耗时
  Future<void> loadData() async {
    // 开始追踪
    developer.Timeline.startSync('load_data');

    try {
      final data = await _fetchFromNetwork();
      developer.Timeline.finishSync();
      return data;
    } catch (e) {
      developer.Timeline.finishSync();
      rethrow;
    }
  }

  // 使用 Timeline.timeSync 简化写法
  Future<void> processData() async {
    await developer.Timeline.timeSync(
      'process_data',
      () async {
        final raw = await _fetchFromNetwork();
        return _transformData(raw);
      },
    );
  }

  Future<List<dynamic>> _fetchFromNetwork() async => [];
  List<dynamic> _transformData(List<dynamic> data) => data;
}
```

**使用 DevTools 分析内存：**

```dart
// 手动触发 GC 观察（仅在 DevTools 中有效）
// DevTools Memory 面板可以：
// 1. 实时监控内存使用曲线
// 2. 拍摄 Heap Snapshot 对比内存泄漏
// 3. 按 Class 查看实例数量
// 4. 追踪对象的引用链
```

### 八、Flutter 渲染性能原理

**16ms 帧预算详解：**

```
60fps 设备：
┌─────────────────────────────────────┐
│              16.67ms                 │
│  ┌─────┐  ┌──────┐  ┌──────┐       │
│  │Build│→ │Layout│→ │Paint │→ GPU  │
│  │     │  │      │  │      │ 合成  │
│  └─────┘  └──────┘  └──────┘ 上屏  │
└─────────────────────────────────────┘

120fps 设备：
┌───────────────────┐
│     8.33ms        │
│ Build→Layout→Paint│→ GPU
└───────────────────┘

任一阶段超时 → 丢帧 → 用户感知卡顿
```

**Build → Layout → Paint 三阶段：**

```dart
// Build 阶段：构建 Widget 树 → Element 树
// 性能问题：不必要的 rebuild（最常见）

class BadExample extends StatefulWidget {
  @override
  State<BadExample> createState() => _BadExampleState();
}

class _BadExampleState extends State<BadExample> {
  int counter = 0;

  @override
  Widget build(BuildContext context) {
    // 每次 counter 变化，ExpensiveWidget 都会 rebuild
    return Column(
      children: [
        Text('$counter'),
        ExpensiveWidget(), // ← 不必要 rebuild！
        ElevatedButton(
          onPressed: () => setState(() => counter++),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}

// Layout 阶段：计算每个 RenderObject 的尺寸和位置
// 性能问题：深度嵌套、没有约束下放
// - Row/Column 嵌套过深 → layout 传递开销大
// - 无约束的 flex 扩展 → 多轮 layout 协商

// Paint 阶段：生成绘制指令列表
// 性能问题：大面积重绘
// - 背景动画导致整个页面重绘
// - 没有使用 RepaintBoundary 隔离
```

### 九、常见性能问题与优化

**1. 不必要 Rebuild 优化：**

```dart
// ===== 问题：父组件 setState 导致所有子组件 rebuild =====
class DashboardPage extends StatefulWidget {
  @override
  State<DashboardPage> createState() => _DashboardPageState();
}

class _DashboardPageState extends State<DashboardPage> {
  int _selectedTab = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 问题：切换 tab 时，Header 也会 rebuild
          Header(),                         // ← 不必要 rebuild
          TabContent(index: _selectedTab),  // ← 需要 rebuild
          Footer(),                         // ← 不必要 rebuild
        ],
      ),
      bottomNavigationBar: BottomNavBar(
        currentIndex: _selectedTab,
        onTap: (i) => setState(() => _selectedTab = i),
      ),
    );
  }
}

// ===== 优化方案 1：抽取子 Widget =====
class _DashboardPageState extends State<DashboardPage> {
  int _selectedTab = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          const _Header(),                  // const → 不会 rebuild
          TabContent(index: _selectedTab),  // 需要 rebuild
          const _Footer(),                  // const → 不会 rebuild
        ],
      ),
      bottomNavigationBar: BottomNavBar(
        currentIndex: _selectedTab,
        onTap: (i) => setState(() => _selectedTab = i),
      ),
    );
  }
}

// 独立 Widget，可以加 const
class _Header extends StatelessWidget {
  const _Header();

  @override
  Widget build(BuildContext context) {
    return AppBar(title: const Text('Dashboard'));
  }
}

class _Footer extends StatelessWidget {
  const _Footer();

  @override
  Widget build(BuildContext context) {
    return const Text('Footer');
  }
}

// ===== 优化方案 2：ValueNotifier + ValueListenableBuilder =====
class _DashboardPageState extends State<DashboardPage> {
  // 用 ValueNotifier 替代 setState，只刷新需要的部分
  final _tabNotifier = ValueNotifier<int>(0);

  @override
  void dispose() {
    _tabNotifier.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          const _Header(),  // 不会 rebuild
          // 只有 ValueListenableBuilder 包裹的部分会 rebuild
          ValueListenableBuilder<int>(
            valueListenable: _tabNotifier,
            builder: (context, tab, _) => TabContent(index: tab),
          ),
          const _Footer(),  // 不会 rebuild
        ],
      ),
      bottomNavigationBar: BottomNavBar(
        currentIndex: _tabNotifier.value,
        onTap: (i) => _tabNotifier.value = i,  // 不用 setState
      ),
    );
  }
}
```

**2. const 构造函数：**

```dart
// ===== 不使用 const =====
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('Hello'),       // 每次 rebuild 都创建新实例
      Icon(Icons.star),    // 每次 rebuild 都创建新实例
      SizedBox(height: 10),// 每次 rebuild 都创建新实例
    ],
  );
}

// ===== 使用 const =====
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Hello'),        // 编译期常量，rebuild 时直接复用
      const Icon(Icons.star),     // 编译期常量
      const SizedBox(height: 10), // 编译期常量
    ],
  );
}

// const 的原理：
// Flutter 框架在 rebuild 时会比较新旧 Widget：
// - 如果 is const → canUpdate 直接返回 true（跳过 rebuild 子树）
// - 如果不是 const → 需要逐字段比较决定是否更新

// 定义 const 构造函数
class MyButton extends StatelessWidget {
  final String label;
  final VoidCallback onTap;

  // const 构造函数要求所有字段都是 final
  const MyButton({
    super.key,
    required this.label,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onTap,
      child: Text(label), // 注意：Text 也需要 const 才能整体 const
    );
  }
}

// 使用处
const MyButton(label: 'Click', onTap: _handleClick); // 编译期常量
```

**3. RepaintBoundary 隔离重绘：**

```dart
// ===== 问题：一个动画导致整个页面重绘 =====
class AnimatedPage extends StatefulWidget {
  @override
  State<AnimatedPage> createState() => _AnimatedPageState();
}

class _AnimatedPageState extends State<AnimatedPage>
    with SingleTickerProviderStateMixin {
  late final _controller = AnimationController(
    vsync: this,
    duration: const Duration(seconds: 2),
  )..repeat();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // 问题：旋转动画每帧都触发 paint
        // 导致同级的 StaticContent 也被标记需要重绘
        AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Transform.rotate(
              angle: _controller.value * 2 * 3.14,
              child: const FlutterLogo(size: 100),
            );
          },
        ),
        // 这个 Widget 本身不需要重绘，但被连累了
        const StaticContent(),  // ← 不必要重绘
      ],
    );
  }
}

// ===== 优化：用 RepaintBoundary 隔离 =====
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      // 方案 1：给动画部分加 RepaintBoundary
      RepaintBoundary(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return Transform.rotate(
              angle: _controller.value * 2 * 3.14,
              child: child,
            );
          },
          // 将不变的 child 提到 builder 外面（性能最佳实践）
          child: const FlutterLogo(size: 100),
        ),
      ),
      // StaticContent 不会被动画影响
      const StaticContent(),  // ← 不再重绘
    ],
  );
}

// RepaintBoundary 原理：
// ┌───────────────────────────────────┐
// │  RenderView (整个页面)             │
// │  ┌─────────────────────────────┐  │
// │  │ RepaintBoundary             │  │ ← 创建独立的 Paint 上下文
// │  │ ┌─────────────────────────┐│  │
// │  │ │  Animated Widget        ││  │ ← 动画只在此区域内重绘
// │  │ └─────────────────────────┘│  │
// │  └─────────────────────────────┘  │
// │  ┌─────────────────────────────┐  │
// │  │  StaticContent (不重绘)     │  │ ← 不受动画影响
// │  └─────────────────────────────┘  │
// └───────────────────────────────────┘
```

### 十、列表优化

```dart
// ===== 问题 1：使用 ListView 而非 ListView.builder =====
// 全量加载 —— 所有 item 一次性创建
ListView(
  children: items.map((item) => ItemWidget(item: item)).toList(),
  // 1 万个 item → 1 万个 Widget 全部创建 → 内存爆炸
)

// 优化：使用 ListView.builder 按需创建
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(item: items[index]); // 只创建可见区域的 item
  },
)

// ===== 问题 2：item Widget 没有 const =====
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    // 每次 rebuild 都创建新实例
    return Padding(
      padding: EdgeInsets.all(8), // 非 const
      child: Text('Item $index'), // 非 const
    );
  },
)

// 优化：尽可能使用 const
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return const Padding(             // const padding
      padding: EdgeInsets.all(8),     // const EdgeInsets
      child: Text('Fixed content'),   // const Text（内容固定时）
    );
  },
)

// ===== 问题 3：Key 策略不当 =====
// 无 key 或 index 作 key：列表重排时性能差且可能状态错乱
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(item: items[index]); // 无 key
    // 或 key: ValueKey(index) — 不推荐
  },
)

// 优化：使用稳定的唯一 key
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(
      key: ValueKey(items[index].id), // 稳定的唯一 ID
      item: items[index],
    );
  },
)

// key 的作用：
// ┌──────────────────────────────────────┐
// │  无 Key：                              │
// │  [A] [B] [C]  → 删除 B → [A] [C]     │
// │  Flutter 按 type 比较，可能复用错误的    │
// │  Element，导致状态错位                  │
// │                                       │
// │  有 ValueKey：                         │
// │  [A:k1] [B:k2] [C:k3] → 删除 B →     │
// │  [A:k1] [C:k3]                        │
// │  Flutter 按 key 精确匹配，正确复用      │
// └──────────────────────────────────────┘
```

**高级列表优化：Slivers**

```dart
// 使用 CustomScrollView + Slivers 实现复杂滚动布局
CustomScrollView(
  slivers: [
    // 顶部固定区域
    const SliverToBoxAdapter(
      child: HeaderSection(),
    ),

    // 中间网格
    SliverGrid(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        mainAxisSpacing: 8,
        crossAxisSpacing: 8,
        childAspectRatio: 1.5,
      ),
      delegate: SliverChildBuilderDelegate(
        (context, index) => ProductCard(product: products[index]),
        childCount: products.length,
      ),
    ),

    // 加载更多指示器
    SliverToBoxAdapter(
      child: _buildLoadMoreIndicator(),
    ),
  ],
)

// SliverAppBar：可折叠的 AppBar
CustomScrollView(
  slivers: [
    SliverAppBar(
      expandedHeight: 200,
      floating: false,
      pinned: true,      // 滚动后固定在顶部
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Details'),
        background: Image.network(
          'https://example.com/hero.jpg',
          fit: BoxFit.cover,
        ),
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Item $index')),
        childCount: 50,
      ),
    ),
  ],
)
```

### 十一、图片优化

```dart
// ===== 1. 图片缓存配置 =====
// 在 MaterialApp 中配置缓存大小
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 设置图片缓存上限（默认 1000 张 / 100MB）
    PaintingBinding.instance.imageCache.maximumSize = 500;       // 最多缓存 500 张
    PaintingBinding.instance.imageCache.maximumSizeBytes = 50 << 20; // 50MB

    return MaterialApp(
      home: HomePage(),
    );
  }
}

// ===== 2. 使用 cached_network_image 缓存网络图片 =====
import 'package:cached_network_image/cached_network_image.dart';

CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  // 内存缓存配置
  memCacheWidth: 400,      // 内存中缓存的宽度（自动缩放）
  memCacheHeight: 400,
  // 磁盘缓存配置
  cacheKey: 'photo_400x400', // 自定义缓存 key
  // 占位图
  placeholder: (context, url) => const CircularProgressIndicator(),
  // 错误图
  errorWidget: (context, url, error) => const Icon(Icons.error),
  // 图片适配
  fit: BoxFit.cover,
  fadeInDuration: const Duration(milliseconds: 300),
)

// ===== 3. 预加载图片 =====
void precacheImages(BuildContext context, List<String> urls) {
  for (final url in urls) {
    // 预加载到内存缓存
    precacheImage(NetworkImage(url), context);
  }
}

// 在页面 initState 中预加载下一页图片
class GalleryPage extends StatefulWidget {
  final List<String> imageUrls;
  const GalleryPage({super.key, required this.imageUrls});

  @override
  State<GalleryPage> createState() => _GalleryPageState();
}

class _GalleryPageState extends State<GalleryPage> {
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // 预加载前 5 张图片
    final toPreload = widget.imageUrls.take(5).toList();
    for (final url in toPreload) {
      precacheImage(NetworkImage(url), context);
    }
  }

  @override
  Widget build(BuildContext context) {
    return PageView.builder(
      itemCount: widget.imageUrls.length,
      itemBuilder: (context, index) {
        // 滑到第 3 张时预加载第 6-8 张
        if (index == 2) {
          _preloadMore(index + 3);
        }
        return CachedNetworkImage(
          imageUrl: widget.imageUrls[index],
          fit: BoxFit.contain,
        );
      },
    );
  }

  void _preloadMore(int startIndex) {
    final urls = widget.imageUrls.skip(startIndex).take(3);
    for (final url in urls) {
      precacheImage(NetworkImage(url), context);
    }
  }
}

// ===== 4. 分辨率适配 =====
// 使用 2.0x / 3.0x 资源适配
// 项目结构：
// assets/
//   images/
//     logo.png          ← 默认（1x）
//     2.0x/logo.png     ← 2x 分辨率
//     3.0x/logo.png     ← 3x 分辨率

// pubspec.yaml
// flutter:
//   assets:
//     - assets/images/

// Flutter 自动根据设备像素密度选择最合适的图片
Image.asset('assets/images/logo.png')

// 代码中指定缩放
Image.asset(
  'assets/images/logo.png',
  width: 100,  // 逻辑像素 100
  height: 100,
  // 在 3x 设备上实际加载 300x300 像素的图片
)
```

### 十二、状态管理性能

```dart
// ===== Provider 性能优化 =====
import 'package:provider/provider.dart';

// 问题：大范围 rebuild
class BadProviderExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 监听整个 UserModel，任何字段变化都 rebuild
    final user = context.watch<UserModel>();

    return Column(
      children: [
        Text(user.name),     // 只需要 name
        Text(user.email),    // 只需要 email
        Text(user.avatar),   // 只需要 avatar
        // 但 user.age、user.address 变化也会 rebuild
      ],
    );
  }
}

// 优化 1：使用 Selector 精确监听
class GoodProviderExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Selector：只在监听的字段变化时 rebuild
    final name = context.select<UserModel, String>((u) => u.name);
    final email = context.select<UserModel, String>((u) => u.email);

    return Column(
      children: [
        Text(name),  // 只有 name 变化时才 rebuild
        Text(email), // 只有 email 变化时才 rebuild
      ],
    );
  }
}

// 优化 2：使用 Selector + 组合类型
Widget build(BuildContext context) {
  return Selector<UserModel, _NameAndEmail>(
    selector: (_, model) => _NameAndEmail(model.name, model.email),
    // shouldRebuild 控制是否 rebuild（默认用 == 比较）
    builder: (context, data, _) {
      return Column(
        children: [
          Text(data.name),
          Text(data.email),
        ],
      );
    },
  );
}

class _NameAndEmail {
  final String name;
  final String email;
  _NameAndEmail(this.name, this.email);

  // 重写 == 确保 Selector 能正确判断变化
  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is _NameAndEmail && name == other.name && email == other.email;

  @override
  int get hashCode => name.hashCode ^ email.hashCode;
}

// ===== Bloc 性能优化 =====
import 'package:flutter_bloc/flutter_bloc.dart';

// 问题：监听整个 State
BlocBuilder<UserBloc, UserState>(
  builder: (context, state) {
    return Text(state.name); // state.email 变化也 rebuild
  },
)

// 优化：使用 buildWhen 控制是否 rebuild
BlocBuilder<UserBloc, UserState>(
  buildWhen: (previous, current) {
    // 只有 name 变化时才 rebuild
    return previous.name != current.name;
  },
  builder: (context, state) {
    return Text(state.name);
  },
)

// BlocSelector：更简洁的写法
BlocSelector<UserBloc, UserState, String>(
  selector: (state) => state.name, // 只选 name
  builder: (context, name) {
    return Text(name);
  },
)

// ===== Riverpod 性能优化 =====
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 使用 select 精确监听
class RiverpodExample extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 只在 name 变化时 rebuild
    final name = ref.watch(userProvider.select((u) => u.name));

    return Text(name);
  }
}
```

**状态管理性能对比：**

| 状态管理 | 精确监听机制 | 学习成本 | 性能控制力 |
|----------|-------------|---------|-----------|
| Provider | `context.select` | 低 | 高 |
| Bloc | `buildWhen` / `BlocSelector` | 中 | 高 |
| Riverpod | `select` | 中 | 高 |
| GetX | `GetBuilder` / `Obx` | 低 | 中（Obx 自动追踪） |
| Redux | `StoreConnector` + `converter` | 高 | 高 |

### 十三、启动优化

```dart
// ===== 1. 延迟加载（Deferred Loading）=====

// 将大型模块拆分为延迟加载的库
// heavy_module.dart
library heavy_module;

// 声明为 deferred
import 'package:my_app/heavy_module.dart' deferred as heavy;

class LazyLoader {
  // 按需加载
  Future<void> loadHeavyModule() async {
    await heavy.loadLibrary(); // 首次调用时加载
    heavy.doSomething();       // 使用模块
  }
}

// 在路由中延迟加载
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      routes: {
        '/': (context) => const HomePage(),
        '/heavy': (context) => const HeavyPageLoader(), // 延迟加载
      },
    );
  }
}

class HeavyPageLoader extends StatefulWidget {
  const HeavyPageLoader({super.key});

  @override
  State<HeavyPageLoader> createState() => _HeavyPageLoaderState();
}

class _HeavyPageLoaderState extends State<HeavyPageLoader> {
  Widget? _loadedPage;

  @override
  void initState() {
    super.initState();
    _loadPage();
  }

  Future<void> _loadPage() async {
    await heavy.loadLibrary();
    if (mounted) {
      setState(() {
        _loadedPage = heavy.HeavyPage();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return _loadedPage ?? const CircularProgressIndicator();
  }
}

// ===== 2. Isolate 预热 =====
import 'dart:isolate';

class IsolatePool {
  static Isolate? _preheatedIsolate;
  static SendPort? _sendPort;

  // 应用启动时预热 Isolate（避免首次使用时的冷启动延迟）
  static Future<void> preheat() async {
    final receivePort = ReceivePort();
    _preheatedIsolate = await Isolate.spawn(
      _isolateEntryPoint,
      receivePort.sendPort,
    );
    _sendPort = await receivePort.first as SendPort;
  }

  static void _isolateEntryPoint(SendPort mainSendPort) {
    final receivePort = ReceivePort();
    mainSendPort.send(receivePort.sendPort);

    receivePort.listen((message) {
      // 处理计算密集型任务
      final result = _heavyComputation(message);
      mainSendPort.send(result);
    });
  }

  static int _heavyComputation(int input) {
    // 模拟耗时计算
    int result = 0;
    for (int i = 0; i < input; i++) {
      result += i;
    }
    return result;
  }

  static Future<int> compute(int input) async {
    if (_sendPort == null) await preheat();

    final response = ReceivePort();
    _sendPort!.send([input, response.sendPort]);
    return await response.first as int;
  }

  static void dispose() {
    _preheatedIsolate?.kill(priority: Isolate.immediate);
  }
}

// ===== 3. 原生启动屏 =====
// Android: android/app/src/main/res/drawable/launch_background.xml
// <?xml version="1.0" encoding="utf-8"?>
// <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
//     <item android:drawable="@color/background" />
//     <item>
//         <bitmap android:src="@mipmap/launch_icon"
//                 android:gravity="center" />
//     </item>
// </layer-list>

// flutter_native_splash 包简化配置
// pubspec.yaml
// flutter_native_splash:
//   color: "#ffffff"
//   image: assets/logo.png
//   android: true
//   ios: true

// 然后运行：dart run flutter_native_splash:create

// ===== 4. main() 优化 =====
void main() async {
  // 确保 Flutter 绑定初始化
  WidgetsFlutterBinding.ensureInitialized();

  // 仅并行初始化必要的组件
  await Future.wait([
    _initSharedPreferences(),
    _initFirebase(),
    // 不在这里初始化可选组件，延迟到使用时再初始化
  ]);

  // 预热 Isolate（可选，用于有计算密集型任务的场景）
  unawaited(IsolatePool.preheat());

  runApp(const MyApp());
}

Future<void> _initSharedPreferences() async {
  // 只读取最关键的配置
}

Future<void> _initFirebase() async {
  // Firebase 初始化
}
```

### 十四、包体积优化

```bash
# ===== 1. Tree Shaking =====
# Flutter 默认开启 tree shaking（Dart 编译器自动移除未使用的代码）
# Release 构建自动生效：
flutter build apk --release
flutter build ios --release

# ===== 2. 延迟加载（Deferred Components）=====
# 在 pubspec.yaml 中配置
# flutter:
#   deferred-components:
#     - name: heavyModule
#       libraries:
#         - package:my_app/heavy_module.dart

# ===== 3. 分离调试信息 =====
flutter build apk --split-debug-info=/debug-info-dir --obfuscate

# --split-debug-info：将调试符号分离到单独目录，减小包体积
# --obfuscate：混淆代码，进一步减小体积并增加逆向难度

# 保留调试信息用于崩溃分析：
# flutter symbolize --input=crash.txt --debug-info=/debug-info-dir

# ===== 4. 按ABI拆分（Android）=====
flutter build apk --split-per-abi

# 生成多个小包而非一个通用包：
# app-armeabi-v7a-release.apk   ~5MB
# app-arm64-v8a-release.apk     ~6MB
# app-x86_64-release.apk        ~6MB
# 而非：app-release.apk          ~15MB（包含所有 ABI）

# ===== 5. App Bundle（推荐）=====
flutter build appbundle

# Google Play 使用 AAB 格式，按设备动态下发
# 用户只下载适合自己设备的代码和资源

# ===== 6. 分析包体积 =====
flutter build apk --analyze-size
flutter build ios --analyze-size

# 输出详细的大小分布报告
```

**包体积优化效果对比：**

| 优化手段 | 预期收益 | 实施难度 | 副作用 |
|----------|---------|---------|--------|
| Tree Shaking | 10-20% | 零（默认开启） | 无 |
| --split-debug-info | 20-30% | 低 | 需保留调试符号 |
| --obfuscate | 5-10% | 低 | 堆栈需反混淆 |
| --split-per-abi | 40-60%（单ABI） | 低 | 需上传多个包 |
| App Bundle | 按设备 | 低 | 仅 Google Play |
| Deferred Loading | 10-30% | 中 | 首次加载延迟 |
| 图片压缩/WebP | 30-50%（图片） | 低 | 无 |
| 移除未使用资源 | 5-15% | 低 | 无 |

### 十五、内存优化

```dart
// ===== 1. 图片缓存管理 =====
class ImageCacheManager {
  static void configure() {
    final cache = PaintingBinding.instance.imageCache;

    // 限制缓存大小
    cache.maximumSize = 200;           // 最多 200 张
    cache.maximumSizeBytes = 50 << 20; // 50MB

    // 内存紧张时清除缓存
    WidgetsBinding.instance.addObserver(
      _MemoryObserver(cache),
    );
  }

  /// 手动清除缓存
  static void clearCache() {
    PaintingBinding.instance.imageCache.clear();
    PaintingBinding.instance.imageCache.clearLiveImages();
  }
}

class _MemoryObserver extends WidgetsBindingObserver {
  final ImageCache cache;
  _MemoryObserver(this.cache);

  @override
  void didHaveMemoryPressure() {
    // 系统内存压力回调
    cache.clear();
    cache.clearLiveImages();
  }
}

// ===== 2. Dispose 清理 =====
class ProperDisposeExample extends StatefulWidget {
  @override
  State<ProperDisposeExample> createState() => _ProperDisposeExampleState();
}

class _ProperDisposeExampleState extends State<ProperDisposeExample>
    with SingleTickerProviderStateMixin {
  late AnimationController _animationController;
  late TextEditingController _textController;
  late ScrollController _scrollController;
  StreamSubscription? _subscription;
  Timer? _debounceTimer;

  @override
  void initState() {
    super.initState();
    _animationController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );
    _textController = TextEditingController();
    _scrollController = ScrollController();
    _subscription = someStream.listen((data) {
      // 处理数据
    });
  }

  @override
  void dispose() {
    // 必须按顺序清理所有资源
    _animationController.dispose();     // 动画控制器
    _textController.dispose();          // 文本控制器
    _scrollController.dispose();        // 滚动控制器
    _subscription?.cancel();            // 流订阅
    _debounceTimer?.cancel();           // 定时器
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}

// ===== 3. WeakReference 与 Finalizer（Dart 3+）=====
class WeakCache<K, V> {
  // 使用 WeakReference 避免阻止 GC 回收
  final Map<K, WeakReference<V>> _cache = {};
  final Map<K, Finalizer<K>> _finalizers = {};

  V? get(K key) => _cache[key]?.target;

  void put(K key, V value) {
    _cache[key] = WeakReference(value);

    // 当值被 GC 回收时自动清理缓存条目
    _finalizers[key] = Finalizer((k) {
      _cache.remove(k);
      _finalizers.remove(k);
    });
    _finalizers[key]!.attach(value, key, detach: value);
  }

  void remove(K key) {
    _cache.remove(key);
    _finalizers.remove(key);
  }

  void clear() {
    _cache.clear();
    _finalizers.clear();
  }
}

// ===== 4. 内存泄漏检测 =====
// 使用 leak_tracker 包（Flutter 3.16+ 内置支持）
// pubspec.yaml
// dev_dependencies:
//   leak_tracker: ^10.0.0
//   leak_tracker_flutter_testing: ^2.0.0

// test/widgets/leak_test.dart
import 'package:leak_tracker_flutter_testing/leak_tracker_flutter_testing.dart';

void main() {
  testWidgets('$MyWidget should not leak', (WidgetTester tester) async {
    await tester.pumpWidget(const MaterialApp(home: MyWidget()));
    await tester.pumpAndSettle();

    // 移除 Widget 触发 dispose
    await tester.pumpWidget(const MaterialApp(home: SizedBox()));

    // LeakTracker 自动检测是否有未释放的对象
  }, leakTrackingTestConfig: LeakTrackingTestConfig());
}
```

### 十六、Web 端性能特别注意事项

```dart
// ===== Flutter Web 渲染器选择 =====
// CanvasKit 渲染器（默认，推荐）：
// - 使用 WebAssembly + Canvas API
// - 渲染质量高，与移动端一致
// - 首次加载需下载 ~2MB CanvasKit 引擎
// - 适合：需要高度一致渲染效果的应用

// HTML 渲染器：
// - 使用 HTML/CSS/SVG
// - 首次加载快，包体积小
// - 部分效果与移动端不一致
// - 适合：对包体积敏感、简单 UI 的应用

// 指定渲染器
flutter build web --web-renderer canvaskit   # CanvasKit
flutter build web --web-renderer html        # HTML
flutter build web --web-renderer auto        # 自动选择（默认）

// ===== Web 端首屏优化 =====
// 1. 使用 skwasm（Flutter 3.22+，基于 WasmGC 的渲染器）
flutter build web --wasm

// 2. 预加载关键资源
// web/index.html 中添加：
// <link rel="preload" href="flutter.js" as="script">
// <link rel="preload" href="main.dart.js" as="script">

// 3. Service Worker 缓存
// web/manifest.json 配置 PWA 缓存策略

// ===== Web 端特别性能瓶颈 =====

// 1. 避免大量 DOM 操作（HTML 渲染器）
// HTML 渲染器下，每个 Widget 对应 DOM 节点
// 大列表必须用 ListView.builder

// 2. 字体加载优化
// web/index.html:
// <style>
//   @font-face {
//     font-family: 'Roboto';
//     src: url('/fonts/Roboto-Regular.woff2') format('woff2');
//     font-display: swap; /* 先用系统字体，加载完替换 */
//   }
// </style>

// 3. 图片格式优化
// Web 端优先使用 WebP
// 使用 CDN 按设备分辨率返回合适的图片尺寸

// 4. 减少 JS 体积
// Web 端 dart2js 编译后体积较大
// 使用 --split-debug-info 和 --obfuscate
flutter build web --split-debug-info=/debug-dir --obfuscate

// 5. 跨域问题
// Web 端网络请求需要配置 CORS
// 开发时使用 --web-browser-flag "--disable-web-security"
flutter run -d chrome --web-browser-flag "--disable-web-security"

// ===== Web 端与移动端性能差异 =====
```

| 维度 | 移动端 | Web 端 |
|------|--------|--------|
| 渲染引擎 | Skia/Impeller | CanvasKit/HTML |
| 线程模型 | 多线程（UI/Raster/IO） | 单线程（JS 主线程） |
| 首次加载 | 极快（原生 AOT） | 较慢（需下载 JS/WASM） |
| 包体积 | 安装包 5-20MB | JS 1-5MB + CanvasKit 2MB |
| 计算性能 | AOT 编译，快 | JIT/WASM，稍慢 |
| 内存 | 可用内存大 | 浏览器内存限制 |
| Isolate | 真正多线程 | Web Worker（有限制） |
| 热重载 | 支持 | 支持 |

### 十七、性能优化检查清单

```
┌─────────────────────────────────────────────────────────┐
│                Flutter 性能优化检查清单                    │
├─────────────────────────────────────────────────────────┤
│ □ 1. 使用 const 构造函数                                 │
│ □ 2. 避免不必要 rebuild（抽取 Widget / Selector）         │
│ □ 3. ListView.builder 替代 ListView                      │
│ □ 4. 使用 ValueKey 保证列表项稳定                        │
│ □ 5. RepaintBoundary 隔离动画区域                        │
│ □ 6. 图片缓存 + 预加载 + 分辨率适配                      │
│ □ 7. 状态管理精确监听（Selector/buildWhen/select）        │
│ □ 8. Dispose 清理所有控制器和订阅                         │
│ □ 9. 计算密集型任务使用 Isolate/Compute                   │
│ □ 10. 延迟加载非首屏模块                                 │
│ □ 11. 原生启动屏                                        │
│ □ 12. --split-debug-info + --obfuscate                  │
│ □ 13. --split-per-abi（Android）                        │
│ □ 14. 图片缓存上限 + 内存压力回调                         │
│ □ 15. DevTools 性能分析验证优化效果                       │
└─────────────────────────────────────────────────────────┘
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Widget 测试 pumpAndSettle 超时 | 有持续动画无法稳定 | 使用 `pump(Duration)` 代替，或设超时 |
| mockito 生成报错 | 未运行 build_runner | `dart run build_runner build` |
| 集成测试找不到 Widget | 应用未完全加载 | 增加 `pumpAndSettle` 等待时间 |
| Golden 测试字体缺失 | 测试环境无渲染字体 | `golden_toolkit` 的 `loadAppFonts()` |
| 列表滚动卡顿 | 未用 builder / item 非 const | `ListView.builder` + `const` item |
| setState 导致全页 rebuild | 状态提升过高 | 抽取子 Widget / ValueNotifier / Selector |
| 图片内存泄漏 | 未清理缓存 / 未 dispose | `imageCache.clear()` + `dispose()` |
| 启动白屏时间长 | 未配置原生启动屏 | `flutter_native_splash` |
| Web 首次加载慢 | CanvasKit 体积大 | 考虑 HTML 渲染器或 skwasm |
| DevTools 连接失败 | 未以 profile 模式运行 | `flutter run --profile` |

### 最佳实践

- 测试金字塔：70% 单元 + 20% Widget + 10% 集成，优先保证单元测试覆盖
- 性能优化先度量：用 DevTools / Performance Overlay 定位瓶颈，再针对性优化
- `const` 是 Flutter 性能优化的第一手段，几乎所有静态 Widget 都应加 `const`
- 列表必须用 `ListView.builder`，key 用稳定 ID 而非 index
- 动画区域用 `RepaintBoundary` 隔离，避免大面积重绘
- 状态管理用 `Selector`/`buildWhen` 精确监听，避免大范围 rebuild
- 所有 Controller/Subscription/Timer 必须在 `dispose()` 中清理
- 包体积用 `--split-per-abi` + `--split-debug-info` + `--obfuscate` 三板斧
- Web 端注意渲染器选择和首次加载优化
- CI 配置覆盖率门禁，核心逻辑 > 80%

## 面试题

**Q1: Flutter 测试金字塔的三个层级分别是什么？各自的职责和特点？**
> Flutter 测试金字塔从底到顶分为三层：① Unit Test（单元测试）——验证函数/类的逻辑正确性，运行在 Dart VM 中，速度极快（毫秒级），使用 `test` 包和 `mockito`，占比约 70%；② Widget Test（组件测试）——验证单个 Widget 的渲染和交互行为，运行在 Flutter 测试引擎中，不需要真机，使用 `testWidgets`/`WidgetTester`/`Finder`，占比约 20%；③ Integration Test（集成测试）——验证完整应用流程，运行在真机或模拟器上，最接近用户真实体验但速度最慢（秒级），使用 `integration_test` 包，占比约 10%。层级越高覆盖面越广但速度越慢、维护成本越高。

**Q2: Widget 测试中 pump() 和 pumpAndSettle() 的区别是什么？**
> `pump()` 触发一帧刷新，将状态更新反映到 UI 上，适用于同步状态更新后的立即检查。`pump(Duration)` 还可以推进时间，适用于动画中间帧的验证。`pumpAndSettle()` 持续调用 `pump()` 直到没有新的帧需要渲染（即所有动画和定时器完成），适用于等待页面跳转、动画完成等场景。注意：如果有持续循环的动画（如 Loading 指示器），`pumpAndSettle()` 会超时失败，此时应用 `pump()` 指定时间代替。

**Q3: Flutter 的 16ms 帧预算是如何分配的？build/layout/paint 三个阶段分别做什么？**
> 60fps 设备要求每帧 16.67ms 内完成渲染。三个阶段：① Build 阶段——执行 Widget 的 `build()` 方法，构建 Widget 树和 Element 树，最常见的性能瓶颈是不必要的 rebuild；② Layout 阶段——遍历 RenderObject 树，计算每个节点的尺寸和位置，深度嵌套的 Row/Column 和无约束的 flex 扩展会增加 layout 开销；③ Paint 阶段——生成绘制指令列表（DisplayList），大面积重绘是主要瓶颈。任一阶段超时就会丢帧，用户感知卡顿。120fps 设备帧预算仅 8.33ms，对优化要求更高。

**Q4: const 构造函数为什么能优化性能？原理是什么？**
> `const` 构造函数在编译期创建常量实例，Dart 编译器会保证相同参数的 `const` 构造只创建一个实例（规范相等性）。Flutter 框架在 rebuild 时会比较新旧 Widget：如果新 Widget 是 `const`，框架直接复用旧 Element 子树，跳过整个子树的 `build()`/`update()` 过程。这意味着 `const` Widget 的整个子树都不会 rebuild，而不仅仅是跳过自身。因此，将静态不变的 Widget 声明为 `const` 是 Flutter 中最简单有效的性能优化手段。

**Q5: RepaintBoundary 的作用是什么？什么场景下应该使用？**
> `RepaintBoundary` 在渲染树中创建一个独立的绘制层（创建新的 OffsetLayer），使其子树拥有独立的 Paint 上下文。当子树需要重绘时，不会影响 RepaintBoundary 之外的兄弟节点。典型使用场景：① 频繁动画的 Widget（旋转、闪烁指示器）应包裹在 RepaintBoundary 中隔离；② 列表中每个 item 如果有独立动画或状态变化；③ 固定 Header/Footer 与可滚动内容之间。但不宜过度使用，每个 RepaintBoundary 都会创建额外的 Layer 增加内存和合成开销。

**Q6: ListView.builder 和 ListView 的区别是什么？为什么大列表必须用 builder？**
> `ListView` 的 `children` 参数接受完整的 Widget 列表，所有 item 在构建时一次性全部创建，1 万条数据就创建 1 万个 Widget，内存占用极高。`ListView.builder` 使用 `itemBuilder` 回调按需创建，只构建当前可见区域的 item（通常 10-20 个），滚动时动态创建和回收，内存占用恒定。此外，builder 模式下配合 `const` item 和 `ValueKey` 可进一步提升滚动流畅度。任何超过 50 项的列表都应使用 `ListView.builder` 或 `SliverList` 的 `SliverChildBuilderDelegate`。

**Q7: Flutter Web 端和移动端的性能差异有哪些？如何针对性优化？**
> 主要差异：① 渲染引擎不同——移动端用 Skia/Impeller 直接绘制，Web 端用 CanvasKit（WASM+Canvas）或 HTML（DOM/CSS）；② 线程模型不同——移动端有多线程（UI/Raster/IO），Web 端是单线程 JS 主线程，长任务会阻塞 UI；③ 首次加载差异——移动端 AOT 编译启动极快，Web 需下载 JS/WASM 文件（CanvasKit 额外 2MB）；④ 内存限制——浏览器内存有限，大量图片更易 OOM。Web 端优化：选择合适渲染器（包体积敏感选 HTML，效果一致选 CanvasKit）；使用 `--wasm`（skwasm）提升性能；配置字体 `font-display: swap`；CDN + 资源预加载；注意 CORS 配置。

**Q8: 如何使用 DevTools 定位 Flutter 应用的性能瓶颈？**
> 步骤：① 以 profile 模式运行应用（`flutter run --profile`），debug 模式性能数据不准；② 打开 DevTools Performance 面板，录制用户操作过程中的 Timeline；③ 分析 Flutter Frames 图表——红色帧为丢帧，点击查看该帧的 build/layout/paint 各阶段耗时；④ 若 UI Thread 占用高，检查 build 和 layout——常见问题是不必要 rebuild 或深层次 Widget 树；⑤ 若 Raster Thread 占用高，检查 paint——常见问题是重绘区域过大，需要 RepaintBoundary 隔离；⑥ 使用 Memory 面板拍摄 Heap Snapshot 对比操作前后的内存增长，定位泄漏对象；⑦ 用 CPU Profiler 分析耗时函数调用栈，定位热点代码。关键是先看 Performance Overlay 的 FPS 指标，再用 Timeline 定位具体是 UI Thread 还是 Raster Thread 瓶颈，最后针对性优化。

---

**相关链接：**
- [[Flutter核心与Widget体系]]
- [[Flutter状态管理]]
- [[前端性能优化]]
- [[前端测试Jest与Vitest]]

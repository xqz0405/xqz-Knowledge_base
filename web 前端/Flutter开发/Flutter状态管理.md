---
tags:
  - Web前端
  - Flutter
  - 状态管理
date: 2026-05-11
status: 已完成
difficulty: 中高
---

# Flutter状态管理

## What — 是什么

> Flutter 状态管理是指管理应用中数据变化并驱动 UI 更新的机制。Flutter 采用声明式 UI，UI 是状态的映射（UI = f(state)），状态变化时需要高效地触发对应 Widget 的重建，状态管理方案解决的是"状态存在哪里、如何修改、如何通知 UI"三个核心问题。

**核心概念：**

- **临时状态（Ephemeral State）**：单个 Widget 内部的状态，用 setState 管理，如按钮开关、输入框文本
- **应用状态（App State）**：跨组件共享的状态，需要状态管理方案，如登录信息、购物车、主题
- **状态提升**：将状态移到共同父组件，通过回调向下传递
- **单向数据流**：状态向下流动，事件向上冒泡，数据流方向清晰可追踪

**状态管理分类：**

```
┌─────────────────────────────────────────────────┐
│              Flutter 状态管理方案                  │
├─────────────────────┬───────────────────────────┤
│    内置机制          │     第三方方案              │
├─────────────────────┼───────────────────────────┤
│  setState           │  Provider                  │
│  InheritedWidget    │  Riverpod 2.0              │
│  ValueNotifier      │  Bloc / Cubit              │
│  ChangeNotifier     │  GetX                      │
│  StreamBuilder      │  MobX                      │
│  FutureBuilder      │  Redux                     │
└─────────────────────┴───────────────────────────┘
```

## Why — 为什么

**适用场景：**

- 多个页面/组件需要共享同一份数据
- 状态变化需要触发多个不相关组件更新
- 需要持久化状态（登录态、用户配置）
- 异步数据加载需要统一的 loading/error/data 管理
- 大型项目需要可维护、可测试的状态架构

**状态管理方案对比：**

| 维度 | Provider | Riverpod | Bloc | GetX |
|------|----------|----------|------|------|
| 学习曲线 | 低 | 中 | 中高 | 低 |
| 类型安全 | 一般 | 强 | 强 | 弱 |
| 性能控制 | Selector | select | buildWhen | 一般 |
| 依赖注入 | 手动 | 内置 | 手动 | 内置 |
| 可测试性 | 好 | 好 | 极好 | 一般 |
| 代码量 | 少 | 中 | 多 | 少 |
| 编译时检查 | 无 | 有 | 无 | 无 |
| 社区推荐 | 官方推荐 | 新官方推荐 | 广泛使用 | 国内流行 |
| 适合项目 | 中小型 | 中大型 | 大型 | 快速原型 |

**优缺点：**

- ✅ Provider：简单易用，官方推荐，学习成本低
- ✅ Riverpod：类型安全，编译时检查，灵活强大
- ✅ Bloc：事件驱动，可测试性极好，适合大型项目
- ✅ GetX：开箱即用，代码量少，快速开发
- ❌ Provider：全局状态需要 MultiProvider 嵌套
- ❌ Riverpod：概念较多，学习曲线稍高
- ❌ Bloc：模板代码多，简单场景过于重量级
- ❌ GetX：隐式依赖，全局状态混乱，社区争议大

## How — 怎么用

### 1. setState 与 InheritedWidget

```dart
// ===== setState — 最简单的状态管理 =====
class CounterPage extends StatefulWidget {
  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _count = 0;

  void _increment() => setState(() => _count++);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(child: Text('$_count')),
      floatingActionButton: FloatingActionButton(
        onPressed: _increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}

// setState 局限：
// 1. 只能管理当前 Widget 的状态
// 2. 状态无法跨组件共享
// 3. 整个 build 方法会重新执行
// 4. 无法做异步状态管理

// ===== InheritedWidget — 内置的状态传递 =====
class CounterInherited extends InheritedWidget {
  final int count;
  final VoidCallback increment;

  const CounterInherited({
    super.key,
    required this.count,
    required this.increment,
    required super.child,
  });

  static CounterInherited of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<CounterInherited>()!;
  }

  @override
  bool updateShouldNotify(CounterInherited old) => count != old.count;
}

// 使用
class CounterApp extends StatefulWidget {
  @override
  State<CounterApp> createState() => _CounterAppState();
}

class _CounterAppState extends State<CounterApp> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return CounterInherited(
      count: _count,
      increment: () => setState(() => _count++),
      child: const DeepChild(),
    );
  }
}

class DeepChild extends StatelessWidget {
  const DeepChild({super.key});

  @override
  Widget build(BuildContext context) {
    final data = CounterInherited.of(context);
    return Text('Count: ${data.count}');
  }
}
```

### 2. Provider

```dart
// ===== 安装 =====
// flutter pub add provider

// ===== ChangeNotifier + Provider =====
class AuthNotifier extends ChangeNotifier {
  User? _user;
  bool _loading = false;
  String? _error;

  User? get user => _user;
  bool get isLoggedIn => _user != null;
  bool get loading => _loading;
  String? get error => _error;

  Future<void> login(String email, String password) async {
    _loading = true;
    _error = null;
    notifyListeners();

    try {
      _user = await _authService.login(email, password);
      _loading = false;
      notifyListeners();
    } catch (e) {
      _error = e.toString();
      _loading = false;
      notifyListeners();
    }
  }

  void logout() {
    _user = null;
    notifyListeners();
  }
}

// ===== MultiProvider 全局注入 =====
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthNotifier()),
        ChangeNotifierProvider(create: (_) => CartNotifier()),
        ChangeNotifierProvider(create: (_) => ThemeNotifier()),
      ],
      child: const MyApp(),
    ),
  );
}

// ===== Consumer 使用 =====
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<AuthNotifier>(
      builder: (context, auth, child) {
        if (auth.loading) return const CircularProgressIndicator();
        if (auth.error != null) return Text(auth.error!);
        return ElevatedButton(
          onPressed: () => auth.login('a@b.com', '123'),
          child: const Text('Login'),
        );
      },
    );
  }
}

// ===== Selector 精细重建控制 =====
class UserAvatar extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 只在 user 变化时重建，忽略 loading/error 变化
    return Selector<AuthNotifier, User?>(
      selector: (_, auth) => auth.user,
      builder: (context, user, child) {
        if (user == null) return const CircleAvatar(child: Icon(Icons.person));
        return CircleAvatar(backgroundImage: NetworkImage(user.avatar));
      },
    );
  }
}

// ===== context.read / context.watch / context.select =====
class ProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // watch — 监听变化，触发 rebuild
    final user = context.watch<AuthNotifier>().user;

    // select — 精细监听某个属性
    final name = context.select<AuthNotifier, String?>((auth) => auth.user?.name);

    return Column(children: [
      Text(name ?? 'Guest'),
      // read — 不监听，仅调用方法（适合回调中使用）
      ElevatedButton(
        onPressed: () => context.read<AuthNotifier>().logout(),
        child: const Text('Logout'),
      ),
    ]);
  }
}
```

### 3. Riverpod 2.0

```dart
// ===== 安装 =====
// flutter pub add flutter_riverpod

// ===== Provider 定义 =====
// 简单值
final countProvider = StateProvider<int>((ref) => 0);

// 异步数据
final userProvider = FutureProvider<User>((ref) async {
  final api = ref.read(apiProvider);
  return api.fetchUser();
});

// Notifier（类式状态管理，推荐）
@riverpod
class Auth extends _$Auth {
  @override
  Future<AuthState> build() async {
    // 初始化状态
    final token = await _storage.getToken();
    if (token == null) return const AuthState.unauthenticated();
    final user = await _api.getProfile();
    return AuthState.authenticated(user);
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final user = await _api.login(email, password);
      await _storage.saveToken(user.token);
      return AuthState.authenticated(user);
    });
  }

  Future<void> logout() async {
    await _storage.clearToken();
    state = AsyncValue.data(const AuthState.unauthenticated());
  }
}

// ===== 手写 Provider（不用代码生成）=====
final authProvider = NotifierProvider<AuthNotifier, AsyncValue<AuthState>>(
  AuthNotifier.new,
);

class AuthNotifier extends Notifier<AsyncValue<AuthState>> {
  @override
  AsyncValue<AuthState> build() {
    return const AsyncValue.data(AuthState.unauthenticated());
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final user = await ref.read(apiProvider).login(email, password);
      return AuthState.authenticated(user);
    });
  }
}

// ===== Widget 中使用 =====
class AuthPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);

    return authState.when(
      data: (state) => state.isAuthenticated
          ? const HomePage()
          : const LoginForm(),
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('Error: $e'),
    );
  }
}

// ===== ref.watch / ref.read / ref.listen =====
class CartButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // watch — 监听变化
    final count = ref.watch(cartProvider).items.length;

    // listen — 监听副作用（SnackBar、导航等）
    ref.listen<CartState>(cartProvider, (prev, next) {
      if (prev!.items.length < next.items.length) {
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('已添加到购物车')),
        );
      }
    });

    return Badge(
      label: Text('$count'),
      child: const Icon(Icons.shopping_cart),
    );
  }
}

// ===== select 精细监听 =====
class CartTotal extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final total = ref.watch(cartProvider.select((s) => s.total));
    return Text('Total: \$$total');
  }
}
```

### 4. Bloc / Cubit

```dart
// ===== 安装 =====
// flutter pub add flutter_bloc

// ===== Cubit（简化版 Bloc，无事件）=====
class AuthCubit extends Cubit<AuthState> {
  final AuthApi _api;

  AuthCubit(this._api) : super(const AuthState.initial());

  Future<void> login(String email, String password) async {
    emit(const AuthState.loading());
    try {
      final user = await _api.login(email, password);
      emit(AuthState.authenticated(user));
    } catch (e) {
      emit(AuthState.error(e.toString()));
    }
  }

  void logout() => emit(const AuthState.unauthenticated());
}

// ===== Bloc（事件驱动，推荐大型项目）=====
// 事件
sealed class AuthEvent {}
class LoginRequested extends AuthEvent {
  final String email;
  final String password;
  LoginRequested(this.email, this.password);
}
class LogoutRequested extends AuthEvent {}

// 状态
sealed class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState {
  final User user;
  AuthAuthenticated(this.user);
}
class AuthError extends AuthState {
  final String message;
  AuthError(this.message);
}

// Bloc
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthApi _api;

  AuthBloc(this._api) : super(AuthInitial()) {
    on<LoginRequested>(_onLogin);
    on<LogoutRequested>(_onLogout);
  }

  Future<void> _onLogin(LoginRequested event, Emitter<AuthState> emit) async {
    emit(AuthLoading());
    try {
      final user = await _api.login(event.email, event.password);
      emit(AuthAuthenticated(user));
    } catch (e) {
      emit(AuthError(e.toString()));
    }
  }

  Future<void> _onLogout(LogoutRequested event, Emitter<AuthState> emit) async {
    emit(AuthInitial());
  }
}

// ===== BlocProvider + BlocBuilder =====
void main() {
  runApp(
    BlocProvider(
      create: (context) => AuthBloc(AuthApi()),
      child: const MyApp(),
    ),
  );
}

class AuthPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<AuthBloc, AuthState>(
      // buildWhen 控制是否重建
      buildWhen: (prev, state) => prev.runtimeType != state.runtimeType,
      builder: (context, state) {
        return switch (state) {
          AuthInitial() => const LoginForm(),
          AuthLoading() => const CircularProgressIndicator(),
          AuthAuthenticated(user: var u) => HomePage(user: u),
          AuthError(message: var m) => ErrorWidget(message: m),
        };
      },
    );
  }
}

// 触发事件
context.read<AuthBloc>().add(LoginRequested('a@b.com', '123'));

// ===== BlocListener 副作用 =====
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthAuthenticated) {
      Navigator.pushReplacementNamed(context, '/home');
    }
    if (state is AuthError) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.message)),
      );
    }
  },
  child: const LoginForm(),
);

// ===== BlocConsumer（Builder + Listener）=====
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) { /* 副作用 */ },
  builder: (context, state) { /* UI */ },
)
```

### 5. GetX

```dart
// ===== 安装 =====
// flutter pub add get

// ===== GetxController =====
class AuthController extends GetxController {
  final _user = Rxn<User>();        // 可空响应式变量
  final _loading = false.obs;        // 响应式布尔
  final _error = ''.obs;

  User? get user => _user.value;
  bool get loading => _loading.value;
  String get error => _error.value;

  Future<void> login(String email, String password) async {
    _loading.value = true;
    _error.value = '';
    try {
      _user.value = await _api.login(email, password);
    } catch (e) {
      _error.value = e.toString();
    } finally {
      _loading.value = false;
    }
  }

  void logout() {
    _user.value = null;
  }
}

// ===== 注入与使用 =====
void main() {
  runApp(GetMaterialApp(
    home: const AuthPage(),
    initialBinding: BindingsBuilder(() {
      Get.put(AuthController());
      Get.put(CartController());
    }),
  ));
}

// Obx 自动响应式重建
class UserAvatar extends GetView<AuthController> {
  @override
  Widget build(BuildContext context) {
    return Obx(() {
      final user = controller.user;
      if (user == null) return const CircleAvatar(child: Icon(Icons.person));
      return CircleAvatar(backgroundImage: NetworkImage(user.avatar));
    });
  }
}

// GetX 响应式语法
var count = 0.obs;                    // 响应式整数
count.value++;                         // 修改值
print(count);                          // 自动解包，直接打印 1

var items = <Product>[].obs;           // 响应式列表
items.add(product);                     // 自动通知

// ===== GetX 路由 =====
Get.toNamed('/detail', arguments: product);
Get.back(result: true);

// ===== GetX 依赖注入 =====
Get.put(ApiService());                  // 立即创建
Get.lazyPut(() => DbService());         // 懒加载
Get.find<ApiService>();                 // 获取实例
```

### 6. 异步状态处理

```dart
// ===== FutureBuilder =====
class UserProfile extends StatelessWidget {
  final Future<User> userFuture;

  const UserProfile({super.key, required this.userFuture});

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<User>(
      future: userFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        }
        if (snapshot.hasError) {
          return Text('Error: ${snapshot.error}');
        }
        final user = snapshot.data!;
        return Text(user.name);
      },
    );
  }
}

// ===== StreamBuilder =====
class LiveScore extends StatelessWidget {
  final Stream<int> scoreStream;

  const LiveScore({super.key, required this.scoreStream});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<int>(
      stream: scoreStream,
      builder: (context, snapshot) {
        if (snapshot.hasError) return Text('Error: ${snapshot.error}');
        if (!snapshot.hasData) return const CircularProgressIndicator();
        return Text('Score: ${snapshot.data}');
      },
    );
  }
}

// ===== Riverpod AsyncValue（推荐）=====
// AsyncValue 统一处理 loading/error/data
class UserPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (e, stack) => Text('Error: $e'),
    );
  }
}

// ===== Bloc 异步状态 =====
// 在 Bloc 中推荐用 sealed class + 自定义状态
```

### 7. 状态持久化

```dart
// ===== Provider 持久化 =====
class ThemeNotifier extends ChangeNotifier {
  late ThemeData _theme;
  final SharedPreferences _prefs;

  ThemeNotifier(this._prefs) {
    final isDark = _prefs.getBool('isDark') ?? false;
    _theme = isDark ? ThemeData.dark() : ThemeData.light();
  }

  ThemeData get theme => _theme;

  void toggleTheme() {
    final isDark = _theme.brightness == Brightness.light;
    _theme = isDark ? ThemeData.dark() : ThemeData.light();
    _prefs.setBool('isDark', isDark);
    notifyListeners();
  }
}

// ===== Riverpod 持久化 =====
final themeProvider = StateNotifierProvider<ThemeNotifier, ThemeData>((ref) {
  return ThemeNotifier(ref.read(sharedPrefsProvider));
});

// ===== Hive 持久化（推荐）=====
// flutter pub add hive hive_flutter

Future<void> main() async {
  await Hive.initFlutter();
  await Hive.openBox('settings');
  runApp(const MyApp());
}

// 存取
final box = Hive.box('settings');
await box.put('isDark', true);
final isDark = box.get('isDark', defaultValue: false);
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Provider 未找到 | 未在 Widget 树上层包裹 | 检查 MultiProvider 位置 |
| 整页不必要重建 | watch 监听范围太大 | 使用 Selector/select |
| Bloc 事件丢失 | 事件还没处理完就发新事件 | 用 EventTransformer 控制并发 |
| GetX 内存泄漏 | 未在 onClose 中清理 | GetxController.onClose() |
| FutureBuilder 重复请求 | build 中创建 Future | 将 Future 缓存为成员变量 |
| 状态初始化顺序错 | 依赖的其他 Provider 未就绪 | 使用 ref.watch 建立依赖 |
| Riverpod 编译报错 | 缺少 ConsumerWidget | 确保使用 Consumer/ConsumerWidget |
| Bloc 状态比较失效 | 状态类未实现 == | 用 equatable 或 freezed |

### 最佳实践

- 简单组件内部状态用 setState
- 跨组件共享用 Provider/Riverpod
- 大型项目事件驱动用 Bloc
- 异步状态统一用 AsyncValue/loading/error/data 模式
- 用 Selector/select/buildWhen 精细控制重建
- 状态类用 freezed 生成不可变对象
- 避免在 build 方法中创建 Provider/Notifier
- 状态持久化用 SharedPreferences/Hive
- 不要在 Provider 中直接操作 UI（导航等用 listener）
- 状态粒度适中，避免"上帝状态"（一个 State 管所有数据）

## 面试题

**Q1: Flutter 中 setState 和 Provider/Riverpod 管理状态有什么区别？各自适用场景？**
> setState 是最简单的状态管理方式，只能管理当前 StatefulWidget 的状态，状态变化会触发整个 build 方法重建，无法跨组件共享。Provider/Riverpod 可以跨组件共享状态，通过 InheritedWidget 机制让子树中的 Widget 订阅状态变化，且通过 Selector 精细控制重建范围。适用场景：setState 适合组件内部临时状态（开关、输入框文本）；Provider/Riverpod 适合跨组件共享的应用状态（登录态、购物车、主题）。

**Q2: Riverpod 相比 Provider 有哪些核心改进？**
> Riverpod 的核心改进：①编译时安全——Provider 声明时不依赖 BuildContext，不会出现 ProviderNotFoundException；②无需 Widget 嵌套——不需要 ChangeNotifierProvider 包裹，直接 ref.watch 即可；③支持多个同类型 Provider——通过 family 修饰符区分；④自动释放——Provider 不再被使用时自动 dispose；⑤AsyncValue 统一异步状态——loading/error/data 一个 when 搞定；⑥支持组合——Provider 可以 watch 其他 Provider，建立依赖关系；⑦Notifier 模式——类式状态管理，逻辑更集中。

**Q3: Bloc 和 Cubit 有什么区别？什么场景用哪个？**
> Cubit 是 Bloc 的简化版，通过直接调用 emit() 发射状态，无需定义事件类，代码量少。Bloc 采用事件驱动模式，所有状态变化都通过事件触发，每个事件有独立处理函数，事件可被转换、缓冲、去重。Cubit 适合简单状态（计数器、表单），代码简洁直观。Bloc 适合复杂业务（认证流程、支付状态机），事件可追踪、可重放、可测试，适合团队协作和审计需求。简单规则：状态变化有复杂业务逻辑或需要审计追踪用 Bloc，否则用 Cubit。

**Q4: Provider 的 Selector 和 context.select 有什么作用？如何优化重建范围？**
> Selector/context.select 的作用是精细控制 Widget 重建范围，只有被监听的属性变化时才重建，而不是整个 ChangeNotifier 变化就重建。Selector<AuthNotifier, User?> 需要指定泛型和返回类型，selector 函数返回需要监听的值，builder 只在该值变化时执行。优化原则：①select 最小粒度的数据；②selector 函数应返回不可变对象或基本类型；③如果返回自定义对象，需实现 == 和 hashCode（或用 equatable）；④回调函数用 read 而非 watch；⑤child 参数缓存不依赖状态的子树。

**Q5: Flutter 中如何处理异步状态（loading/error/data）？**
> 三种主流方案：①FutureBuilder/StreamBuilder——Flutter 内置，在 builder 中判断 ConnectionState 和 hasError/hasData，适合简单场景但不便复用；②Bloc 的 sealed class——定义 Loading/Loaded/Error 三种状态类，通过模式匹配处理，类型安全且可扩展；③Riverpod 的 AsyncValue——内置 when(loading:error:data:) 方法，一行代码处理三种状态，还支持 valueOrNull/isError 等便捷属性。推荐 Riverpod AsyncValue 或 Bloc sealed class，避免手写 if-else 判断。

**Q6: 为什么不推荐过度使用 GetX？**
> GetX 的争议点：①全局单例模式——Get.put/Get.find 本质是全局变量，依赖关系隐式化，不利于测试和维护；②隐式依赖——Obx 自动追踪依赖，看似方便但难以追踪数据流向，debug 困难；③过度耦合——路由、状态管理、依赖注入、国际化全部耦合在 GetX 中，替换成本高；④违背 Flutter 设计——绕过 Widget 树、InheritedWidget 等核心机制；⑤社区分歧——官方团队未推荐，Flutter 核心团队成员曾公开批评。适合个人项目快速原型，不推荐团队大型项目。

**Q7: Riverpod 的 ref.watch、ref.read、ref.listen 分别在什么场景使用？**
> ref.watch — 在 build 方法中使用，监听 Provider 变化并触发 Widget 重建，是声明式的数据订阅。ref.read — 在回调/事件处理中使用，一次性读取 Provider 的值而不订阅变化，适合调用方法（如 ref.read(authProvider.notifier).logout()）。ref.listen — 监听 Provider 变化执行副作用（SnackBar、导航、日志），不触发 rebuild，类似 BlocListener。规则：build 中用 watch，回调中用 read，副作用用 listen。不要在 build 中用 read（不会响应变化），不要在回调中用 watch（无法触发重建）。

**Q8: 如何在 Flutter 中实现状态的持久化？有哪些方案？**
> 状态持久化方案：①SharedPreferences — 键值对存储，适合简单配置（主题、语言），Provider/Riverpod 中在构造函数读取、修改时写入；②Hive — 高性能 NoSQL 数据库，支持自定义对象序列化，适合复杂结构数据；③SQLite (sqflite/drift) — 关系型数据库，适合大量结构化数据；④flutter_secure_storage — 加密存储，适合敏感数据（Token）；⑤HydratedBloc — Bloc 的持久化扩展，自动将状态序列化到本地。最佳实践：持久化与状态管理分离，在 State/Notifier 中读取和写入，启动时恢复状态，修改时同步持久化。

---

**相关链接：**
- [[Flutter核心与Widget体系]]
- [[Dart语言核心]]
- [[状态管理方案]]
- [[React Hooks详解]]

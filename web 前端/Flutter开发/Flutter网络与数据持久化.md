---
tags:
  - Web前端
  - Flutter
  - 网络
  - 持久化
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Flutter网络与数据持久化

## What — 是什么

> Flutter 网络与数据持久化是移动端应用的核心基础设施层：网络层负责与后端 API 通信、文件传输和实时消息，持久化层负责本地数据存储、缓存和安全保存敏感信息。两者协同构成了离线优先架构的基础。

**核心概念：**

- **HTTP 网络请求**：通过 `http` 包或 `dio` 库发起 RESTful API 调用，获取/提交服务端数据
- **dio 封装**：基于 Interceptor 的请求/响应拦截，统一处理 Token、错误、日志、重试
- **文件传输**：`multipart/form-data` 上传、流式下载、进度监听、断点续传
- **WebSocket**：全双工实时通信，适用于聊天、推送、协作等场景
- **JSON 序列化**：`json_serializable` + `build_runner` 自动生成序列化代码，替代手写 `fromJson`/`toJson`
- **SharedPreferences**：轻量键值对存储，适合配置项、偏好设置
- **SQLite**：结构化本地数据库，`sqflite` 原生操作或 `drift` 类型安全 ORM
- **安全存储**：`flutter_secure_storage` 利用 Keychain/Keystore 加密存储敏感数据
- **数据缓存**：内存缓存 + 磁盘缓存的分层策略，减少网络请求
- **网络检测**：`connectivity_plus` 感知网络状态变化，驱动离线/在线切换
- **离线优先**：本地优先读写，网络恢复后同步，保证弱网/断网可用

**整体架构：**

```
┌─────────────────────────────────────────────────────────┐
│                    Flutter Application                   │
├─────────────────────────────────────────────────────────┤
│                     UI / State Layer                     │
│            (Provider / Riverpod / Bloc)                  │
├────────────────────┬────────────────────────────────────┤
│    Network Layer   │         Persistence Layer           │
│  ┌──────────────┐  │  ┌───────────┬──────────┬────────┐ │
│  │  dio 封装     │  │  │ Shared    │ SQLite   │ Secure │ │
│  │  Interceptor │  │  │ Prefs     │ (drift)  │ Storage│ │
│  │  Transformer │  │  │ (KV)      │ (RDBMS)  │(Keychain)│
│  ├──────────────┤  │  ├───────────┼──────────┼────────┤ │
│  │  WebSocket   │  │  │  文件系统  │  数据缓存  │ 加密   │ │
│  │  (实时通信)   │  │  │(path_prov)│(内存+磁盘)│ 存储   │ │
│  ├──────────────┤  │  └───────────┴──────────┴────────┘ │
│  │  文件传输     │  │                                    │
│  │ (上传/下载)   │  │                                    │
│  └──────┬───────┘  └────────────────────────────────────┤
│         │                                               │
├─────────┴───────────────────────────────────────────────┤
│              Connectivity (connectivity_plus)            │
│                   网络状态检测                             │
├─────────────────────────────────────────────────────────┤
│                Offline-First Sync Engine                 │
│              (本地优先 · 增量同步 · 冲突解决)               │
└─────────────────────────────────────────────────────────┘
```

**数据流全景：**

```
用户操作
  │
  ▼
State Layer (Riverpod/Bloc)
  │
  ├──── 读操作 ────► 缓存层 ──── 命中? ──► 返回缓存数据
  │                              │ 未命中
  │                              ▼
  │                         网络请求 (dio)
  │                              │
  │                              ▼
  │                         写入缓存 + 返回数据
  │
  ├──── 写操作 ────► 本地持久化 (SQLite/SharedPrefs)
  │                     │
  │                     ▼
  │                同步队列 (离线时入队)
  │                     │
  │                     ▼ (网络恢复)
  │                网络提交 (dio POST/PUT)
  │
  └──── 实时 ────► WebSocket (双向通信)
```

## Why — 为什么

**适用场景：**

- 移动端 App 必须与后端 API 交互获取数据
- 弱网/断网环境下需要本地缓存和离线能力
- 用户登录态、Token 等敏感信息需要安全存储
- 聊天/推送/协作等场景需要实时通信
- 大文件上传下载需要进度监听和断点续传
- 复杂数据模型需要类型安全的序列化方案

**网络库对比：**

| 维度 | http 包 | dio 库 |
|------|---------|--------|
| 请求方式 | `http.get()` / `http.post()` | `dio.get()` / `dio.post()` |
| 拦截器 | 不支持 | 强大的 Interceptor 链 |
| 全局配置 | 手动封装 | `BaseOptions` 统一配置 |
| FormData | 手动拼接 | 内置 `FormData` 支持 |
| 取消请求 | 不支持 | `CancelToken` 优雅取消 |
| 超时/重试 | 手动实现 | 内置超时 + 重试拦截器 |
| 下载进度 | 不支持 | `onReceiveProgress` 回调 |
| 证书校验 | 手动 | `SecurityContext` 配置 |
| 适用场景 | 简单请求/示例项目 | 生产级项目 |

**持久化方案对比：**

| 维度 | SharedPreferences | SQLite (sqflite) | SQLite (drift) | flutter_secure_storage | 文件系统 |
|------|-------------------|-------------------|----------------|----------------------|----------|
| 数据类型 | KV 键值对 | 关系型表 | 关系型表 + 类型安全 | KV 加密键值对 | 任意文件 |
| 数据量 | KB 级 | GB 级 | GB 级 | KB 级 | 不限 |
| 查询能力 | 按 key | SQL 全功能 | SQL + 类型安全 | 按 key | 无 |
| 安全性 | 明文存储 | 明文存储 | 明文存储 | Keychain/Keystore | 明文 |
| 性能 | 极快(内存) | 快 | 快 | 中(加密开销) | 中(磁盘IO) |
| 复杂度 | 低 | 中 | 中高 | 低 | 中 |
| 适用场景 | 配置/偏好 | 结构化数据 | 类型安全需求 | Token/密码 | 缓存/媒体 |

**优缺点：**

- ✅ dio 封装：
  - 拦截器机制灵活强大，可统一处理认证、日志、错误
  - 内置取消、进度、FormData，减少样板代码
  - 社区活跃，插件生态丰富
- ❌ dio 封装：
  - 拦截器链调试复杂，执行顺序需仔细控制
  - 过度封装可能隐藏问题
  - 与某些原生平台特性（如 NSAppTransportSecurity）需额外配置
- ✅ SQLite (drift)：
  - 类型安全的 SQL 构建，编译期检查
  - 响应式查询（watch），数据变化自动通知 UI
  - 迁移系统完善
- ❌ SQLite (drift)：
  - 代码生成增加构建时间
  - 学习曲线比 sqflite 陡峭
  - 复杂联表查询仍需手写 SQL

## How — 怎么用

### 快速上手

**依赖配置（pubspec.yaml）：**

```yaml
dependencies:
  # 网络请求
  http: ^1.2.0              # 轻量 HTTP 客户端
  dio: ^5.4.0               # 强大的 HTTP 客户端

  # WebSocket
  web_socket_channel: ^3.0.0

  # JSON 序列化
  json_annotation: ^4.9.0

  # 持久化
  shared_preferences: ^2.2.0
  sqflite: ^2.3.0           # SQLite 原生操作
  drift: ^2.15.0            # SQLite ORM
  sqlite3_flutter_libs: ^0.5.0  # drift 依赖
  path_provider: ^2.1.0     # 路径获取
  flutter_secure_storage: ^9.2.0  # 安全存储

  # 网络检测
  connectivity_plus: ^6.0.0

dev_dependencies:
  build_runner: ^2.4.0      # 代码生成工具
  json_serializable: ^6.8.0  # JSON 序列化生成
  drift_dev: ^2.15.0        # drift 代码生成
```

### 一、HTTP 网络请求

#### 1.1 http 包基础用法

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

/// http 包 — 最简单的网络请求方式
class HttpExample {
  /// GET 请求
  static Future<void> fetchUsers() async {
    try {
      final uri = Uri.parse('https://api.example.com/users');
      final response = await http.get(
        uri,
        headers: {
          'Content-Type': 'application/json',
          'Authorization': 'Bearer token123',
        },
      );

      if (response.statusCode == 200) {
        // 解码 JSON 响应体
        final List<dynamic> data = json.decode(response.body);
        print('用户数量: ${data.length}');
      } else {
        print('请求失败: ${response.statusCode}');
      }
    } catch (e) {
      print('网络异常: $e');
    }
  }

  /// POST 请求
  static Future<void> createUser(Map<String, dynamic> user) async {
    try {
      final uri = Uri.parse('https://api.example.com/users');
      final response = await http.post(
        uri,
        headers: {'Content-Type': 'application/json'},
        body: json.encode(user), // 编码请求体
      );

      if (response.statusCode == 201) {
        print('创建成功: ${response.body}');
      }
    } catch (e) {
      print('创建失败: $e');
    }
  }

  /// PUT 请求 — 全量更新
  static Future<void> updateUser(int id, Map<String, dynamic> user) async {
    final uri = Uri.parse('https://api.example.com/users/$id');
    final response = await http.put(
      uri,
      headers: {'Content-Type': 'application/json'},
      body: json.encode(user),
    );
    // 处理响应...
  }

  /// PATCH 请求 — 部分更新
  static Future<void> patchUser(int id, Map<String, dynamic> fields) async {
    final uri = Uri.parse('https://api.example.com/users/$id');
    final response = await http.patch(
      uri,
      headers: {'Content-Type': 'application/json'},
      body: json.encode(fields),
    );
    // 处理响应...
  }

  /// DELETE 请求
  static Future<void> deleteUser(int id) async {
    final uri = Uri.parse('https://api.example.com/users/$id');
    final response = await http.delete(uri);
    if (response.statusCode == 204) {
      print('删除成功');
    }
  }

  /// 带查询参数的 GET 请求
  static Future<void> searchUsers({
    required String keyword,
    int page = 1,
    int pageSize = 20,
  }) async {
    final uri = Uri.https('api.example.com', '/users', {
      'keyword': keyword,
      'page': page.toString(),
      'pageSize': pageSize.toString(),
    });
    final response = await http.get(uri);
    // 处理响应...
  }

  /// 带超时的请求
  static Future<void> fetchWithTimeout() async {
    try {
      final uri = Uri.parse('https://api.example.com/slow-api');
      final response = await http.get(uri).timeout(
        const Duration(seconds: 10), // 10 秒超时
        onTimeout: () => throw Exception('请求超时'),
      );
    } catch (e) {
      print('超时或网络错误: $e');
    }
  }
}
```

#### 1.2 dio 库基础用法

```dart
import 'package:dio/dio.dart';

/// dio 基础用法 — 比 http 包功能强大得多
class DioBasicExample {
  late final Dio _dio;

  DioBasicExample() {
    // 创建 Dio 实例并配置 BaseOptions
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.example.com',  // 基础 URL
      connectTimeout: const Duration(seconds: 15), // 连接超时
      receiveTimeout: const Duration(seconds: 15), // 接收超时
      sendTimeout: const Duration(seconds: 15),    // 发送超时
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
      responseType: ResponseType.json,   // 自动 JSON 解码
      validateStatus: (status) => status != null && status < 500, // 自定义状态码校验
    ));
  }

  /// GET 请求
  Future<Response> getUsers({int page = 1}) async {
    return await _dio.get(
      '/users',
      queryParameters: {'page': page, 'size': 20},
    );
  }

  /// POST 请求
  Future<Response> createUser(Map<String, dynamic> data) async {
    return await _dio.post('/users', data: data);
  }

  /// PUT 请求
  Future<Response> updateUser(int id, Map<String, dynamic> data) async {
    return await _dio.put('/users/$id', data: data);
  }

  /// PATCH 请求
  Future<Response> patchUser(int id, Map<String, dynamic> data) async {
    return await _dio.patch('/users/$id', data: data);
  }

  /// DELETE 请求
  Future<Response> deleteUser(int id) async {
    return await _dio.delete('/users/$id');
  }

  /// 并发请求 — 多个接口同时请求
  Future<void> fetchDashboard() async {
    final results = await Future.wait([
      _dio.get('/dashboard/stats'),
      _dio.get('/dashboard/notifications'),
      _dio.get('/dashboard/activities'),
    ]);

    final stats = results[0].data;
    final notifications = results[1].data;
    final activities = results[2].data;
    print('仪表盘数据加载完成');
  }
}
```

### 二、dio 封装实战

#### 2.1 BaseOptions 配置

```dart
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';

/// 全局 Dio 配置 — 单例模式
class DioConfig {
  // 私有构造
  DioConfig._();

  static Dio? _instance;

  /// 获取 Dio 单例
  static Dio get dio {
    _instance ??= _createDio();
    return _instance!;
  }

  /// 创建配置好的 Dio 实例
  static Dio _createDio() {
    final dio = Dio(
      BaseOptions(
        baseUrl: const String.fromEnvironment(
          'API_BASE_URL',
          defaultValue: 'https://api.example.com',
        ),
        connectTimeout: const Duration(seconds: 15),
        receiveTimeout: const Duration(seconds: 15),
        sendTimeout: const Duration(seconds: 15),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
          'X-Platform': defaultTargetPlatform.name, // 平台标识
          'X-Version': '2.0.0',                     // App 版本
        },
        responseType: ResponseType.json,
        // 不抛出非 2xx 异常，统一在拦截器处理
        validateStatus: (status) => status != null && status < 500,
      ),
    );

    // 添加拦截器（执行顺序：按添加顺序 onRequest，按逆序 onResponse/onError）
    dio.interceptors.addAll([
      LogInterceptor(),           // 日志拦截器（最外层）
      AuthInterceptor(),          // 认证拦截器
      ErrorInterceptor(),         // 错误处理拦截器
      RetryInterceptor(dio: dio), // 重试拦截器（最内层）
    ]);

    return dio;
  }

  /// 重置实例（用于切换环境/登出后重建）
  static void reset() {
    _instance = null;
  }
}
```

#### 2.2 请求拦截器（Token 自动刷新 + 统一错误处理 + 日志拦截）

```dart
import 'package:dio/dio.dart';
import 'dart:async';

/// ==================== 认证拦截器 ====================
/// 自动附加 Token + Token 过期自动刷新
class AuthInterceptor extends Interceptor {
  // Token 存储键
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  // 刷新锁 — 防止并发刷新
  bool _isRefreshing = false;
  final List<Completer<void>> _refreshQueue = [];

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    // 不需要 Token 的接口白名单
    final whitelist = ['/auth/login', '/auth/register', '/auth/refresh'];
    if (whitelist.contains(options.path)) {
      return handler.next(options);
    }

    // 从安全存储读取 Token
    final token = await _getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    // 401 表示 Token 过期，尝试刷新
    if (err.response?.statusCode == 401) {
      // 白名单接口 401 不刷新
      final whitelist = ['/auth/login', '/auth/register'];
      if (whitelist.contains(err.requestOptions.path)) {
        return handler.next(err);
      }

      try {
        // 等待刷新完成
        await _refreshToken(err.requestOptions);
        // 用新 Token 重试原请求
        final newToken = await _getAccessToken();
        err.requestOptions.headers['Authorization'] = 'Bearer $newToken';
        final response = await Dio().fetch(err.requestOptions);
        return handler.resolve(response);
      } catch (refreshError) {
        // 刷新失败，跳转登录页
        _handleRefreshFailed();
        return handler.next(err);
      }
    }
    handler.next(err);
  }

  /// 刷新 Token — 加锁防止并发
  Future<void> _refreshToken(RequestOptions requestOptions) async {
    if (_isRefreshing) {
      // 已在刷新中，排队等待
      final completer = Completer<void>();
      _refreshQueue.add(completer);
      return completer.future;
    }

    _isRefreshing = true;
    try {
      final refreshToken = await _getRefreshToken();
      if (refreshToken == null) throw Exception('无 RefreshToken');

      // 创建新 Dio 实例发刷新请求（避免死循环拦截器）
      final refreshDio = Dio(BaseOptions(
        baseUrl: requestOptions.baseUrl,
      ));
      final response = await refreshDio.post('/auth/refresh', data: {
        'refreshToken': refreshToken,
      });

      // 保存新 Token
      final newAccessToken = response.data['accessToken'];
      final newRefreshToken = response.data['refreshToken'];
      await _saveTokens(newAccessToken, newRefreshToken);

      // 通知排队等待的请求
      for (final completer in _refreshQueue) {
        completer.complete();
      }
      _refreshQueue.clear();
    } catch (e) {
      // 刷新失败，通知所有等待的请求
      for (final completer in _refreshQueue) {
        completer.completeError(e);
      }
      _refreshQueue.clear();
      rethrow;
    } finally {
      _isRefreshing = false;
    }
  }

  Future<String?> _getAccessToken() async {
    // 实际项目中从 flutter_secure_storage 读取
    return null; // placeholder
  }

  Future<String?> _getRefreshToken() async {
    return null; // placeholder
  }

  Future<void> _saveTokens(String access, String refresh) async {
    // 实际项目中保存到 flutter_secure_storage
  }

  void _handleRefreshFailed() {
    // 清除 Token，跳转登录页
    // 实际项目中通过 EventBus 或 NavigationService 实现
  }
}

/// ==================== 错误处理拦截器 ====================
/// 统一错误分类 + 业务错误码处理
class ErrorInterceptor extends Interceptor {
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // 将 DioException 转换为自定义 AppException
    final appError = _convertToAppError(err);

    // 根据业务错误码特殊处理
    if (err.response?.data is Map) {
      final data = err.response!.data as Map;
      final code = data['code'];

      switch (code) {
        case 'USER_BANNED':
          // 用户被封禁，强制退出
          _forceLogout();
          break;
        case 'VERSION_OUTDATED':
          // 版本过旧，提示升级
          _showUpgradeDialog();
          break;
        case 'MAINTENANCE':
          // 服务维护中
          _showMaintenancePage(data['message']);
          break;
      }
    }

    handler.next(appError);
  }

  /// 将 DioException 转为 AppException
  AppException _convertToAppError(DioException err) {
    switch (err.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        return AppException(
          code: 'TIMEOUT',
          message: '网络连接超时，请检查网络后重试',
          originalError: err,
        );
      case DioExceptionType.connectionError:
        return AppException(
          code: 'NETWORK_ERROR',
          message: '网络连接失败，请检查网络设置',
          originalError: err,
        );
      case DioExceptionType.badResponse:
        return _handleBadResponse(err.response);
      case DioExceptionType.cancel:
        return AppException(
          code: 'CANCELLED',
          message: '请求已取消',
          originalError: err,
        );
      default:
        return AppException(
          code: 'UNKNOWN',
          message: '未知错误: ${err.message}',
          originalError: err,
        );
    }
  }

  AppException _handleBadResponse(Response? response) {
    final statusCode = response?.statusCode;
    final data = response?.data;

    switch (statusCode) {
      case 400:
        return AppException(
          code: 'BAD_REQUEST',
          message: data?['message'] ?? '请求参数错误',
        );
      case 401:
        return AppException(
          code: 'UNAUTHORIZED',
          message: '登录已过期，请重新登录',
        );
      case 403:
        return AppException(
          code: 'FORBIDDEN',
          message: '没有操作权限',
        );
      case 404:
        return AppException(
          code: 'NOT_FOUND',
          message: '请求的资源不存在',
        );
      case 429:
        return AppException(
          code: 'TOO_MANY_REQUESTS',
          message: '请求过于频繁，请稍后重试',
        );
      case 500:
        return AppException(
          code: 'SERVER_ERROR',
          message: '服务器内部错误',
        );
      case 502:
      case 503:
        return AppException(
          code: 'SERVICE_UNAVAILABLE',
          message: '服务暂时不可用，请稍后重试',
        );
      default:
        return AppException(
          code: 'HTTP_$statusCode',
          message: data?['message'] ?? '请求失败 ($statusCode)',
        );
    }
  }

  void _forceLogout() { /* 跳转登录 */ }
  void _showUpgradeDialog() { /* 显示升级弹窗 */ }
  void _showMaintenancePage(String? message) { /* 维护页面 */ }
}

/// 自定义异常类
class AppException extends DioException {
  final String code;
  final String message;
  final DioException? originalError;

  AppException({
    required this.code,
    required this.message,
    this.originalError,
  }) : super(
    requestOptions: originalError?.requestOptions ?? RequestOptions(),
    error: message,
    response: originalError?.response,
  );

  @override
  String toString() => 'AppException($code): $message';
}

/// ==================== 日志拦截器 ====================
/// 格式化日志输出（开发环境）
class PrettyLogInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (kDebugMode) {
      print('┌─────────────────── REQUEST ───────────────────');
      print('│ ${options.method} ${options.uri}');
      print('│ Headers: ${_formatMap(options.headers)}');
      if (options.data != null) {
        print('│ Body: ${_truncate(options.data.toString(), 500)}');
      }
      if (options.queryParameters.isNotEmpty) {
        print('│ Query: ${_formatMap(options.queryParameters)}');
      }
      print('└───────────────────────────────────────────────');
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    if (kDebugMode) {
      print('┌─────────────────── RESPONSE ──────────────────');
      print('│ ${response.statusCode} ${response.requestOptions.uri}');
      print('│ Duration: ${response.requestOptions.extra['startTime']}ms');
      print('│ Data: ${_truncate(response.data.toString(), 500)}');
      print('└───────────────────────────────────────────────');
    }
    handler.next(response);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (kDebugMode) {
      print('┌─────────────────── ERROR ─────────────────────');
      print('│ ${err.type}: ${err.message}');
      print('│ ${err.requestOptions.method} ${err.requestOptions.uri}');
      if (err.response != null) {
        print('│ Status: ${err.response?.statusCode}');
        print('│ Data: ${_truncate(err.response?.data.toString() ?? '', 300)}');
      }
      print('└───────────────────────────────────────────────');
    }
    handler.next(err);
  }

  String _formatMap(Map map) {
    return map.entries.map((e) => '${e.key}: ${e.value}').join(', ');
  }

  String _truncate(String text, int maxLen) {
    return text.length > maxLen ? '${text.substring(0, maxLen)}...' : text;
  }
}
```

#### 2.3 重试拦截器

```dart
/// 重试拦截器 — 网络异常自动重试
class RetryInterceptor extends Interceptor {
  final Dio dio;
  final int maxRetries;        // 最大重试次数
  final Duration retryDelay;   // 重试间隔
  final Set<String> retryableMethods; // 可重试的 HTTP 方法

  RetryInterceptor({
    required this.dio,
    this.maxRetries = 3,
    this.retryDelay = const Duration(seconds: 1),
    this.retryableMethods = const {'GET', 'PUT', 'DELETE'},
  });

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    // 仅对可重试方法且符合重试条件的错误重试
    if (!_shouldRetry(err)) {
      return handler.next(err);
    }

    // 获取已重试次数
    final retryCount = (err.requestOptions.extra['retryCount'] as int?) ?? 0;
    if (retryCount >= maxRetries) {
      return handler.next(err);
    }

    // 指数退避等待
    final delay = Duration(
      milliseconds: retryDelay.inMilliseconds * (1 << retryCount), // 1s, 2s, 4s...
    );
    await Future.delayed(delay);

    // 更新重试计数
    err.requestOptions.extra['retryCount'] = retryCount + 1;

    try {
      final response = await dio.fetch(err.requestOptions);
      handler.resolve(response);
    } on DioException catch (e) {
      // 重试仍失败，继续传递
      handler.next(e);
    }
  }

  bool _shouldRetry(DioException err) {
    // 仅特定方法重试（POST 等非幂等方法一般不重试）
    if (!retryableMethods.contains(err.requestOptions.method)) {
      return false;
    }
    // 仅特定错误类型重试
    return err.type == DioExceptionType.connectionTimeout ||
        err.type == DioExceptionType.receiveTimeout ||
        err.type == DioExceptionType.connectionError;
  }
}
```

#### 2.4 Transformer 自定义转换

```dart
import 'package:dio/dio.dart';
import 'dart:convert';

/// 自定义 Transformer — 统一处理响应数据包装
/// 适配后端标准格式: { "code": 0, "message": "ok", "data": {...} }
class AppTransformer extends BackgroundTransformer {
  @override
  Future<Response> transformResponse(
    RequestOptions options,
    ResponseBody responseBody,
  ) async {
    // 先调用默认转换
    final response = await super.transformResponse(options, responseBody);

    if (response.data is Map) {
      final map = response.data as Map<String, dynamic>;

      // 后端标准格式解包
      final code = map['code'];
      final message = map['message'] ?? '';
      final data = map['data'];

      // 业务错误码判断
      if (code != 0 && code != 200) {
        // 业务错误，转为 DioException
        throw DioException(
          requestOptions: options,
          response: Response(
            requestOptions: options,
            statusCode: response.statusCode,
            data: {'code': code, 'message': message, 'data': data},
          ),
          error: message,
          type: DioExceptionType.badResponse,
        );
      }

      // 解包 data 字段，简化后续使用
      response.data = data;
    }

    return response;
  }
}
```

#### 2.5 CancelToken 取消请求

```dart
/// CancelToken 使用场景 — 页面退出时取消未完成请求
class CancelTokenExample {
  final Dio _dio = DioConfig.dio;

  // 页面级别的 CancelToken
  CancelToken? _cancelToken;

  /// 进入页面时创建 CancelToken
  void onPageInit() {
    _cancelToken = CancelToken();
  }

  /// 发起请求时绑定 CancelToken
  Future<void> fetchUserProfile(String userId) async {
    try {
      final response = await _dio.get(
        '/users/$userId',
        cancelToken: _cancelToken,
      );
      // 处理响应...
    } on DioException catch (e) {
      if (CancelToken.isCancel(e)) {
        print('请求已取消: ${e.message}');
      } else {
        print('请求失败: ${e.message}');
      }
    }
  }

  /// 页面销毁时取消所有请求
  void onPageDispose() {
    _cancelToken?.cancel('页面已退出');
    _cancelToken = null;
  }

  /// 手动取消特定请求
  void cancelSearch() {
    _cancelToken?.cancel('用户取消搜索');
    // 重新创建 CancelToken 以便后续请求
    _cancelToken = CancelToken();
  }
}
```

### 三、RESTful API 调用规范

```dart
/// RESTful API Repository 封装
/// 规范: 每个资源一个 Repository，方法命名遵循 REST 语义
class UserRepository {
  final Dio _dio = DioConfig.dio;

  /// 列表查询 — GET /users
  Future<PageResult<User>> getUsers({
    int page = 1,
    int size = 20,
    String? keyword,
    String? role,
  }) async {
    final response = await _dio.get('/users', queryParameters: {
      'page': page,
      'size': size,
      if (keyword != null) 'keyword': keyword,
      if (role != null) 'role': role,
    });
    return PageResult.fromJson(response.data, User.fromJson);
  }

  /// 详情查询 — GET /users/:id
  Future<User> getUserById(int id) async {
    final response = await _dio.get('/users/$id');
    return User.fromJson(response.data);
  }

  /// 创建 — POST /users
  Future<User> createUser(CreateUserRequest request) async {
    final response = await _dio.post('/users', data: request.toJson());
    return User.fromJson(response.data);
  }

  /// 全量更新 — PUT /users/:id
  Future<User> updateUser(int id, UpdateUserRequest request) async {
    final response = await _dio.put('/users/$id', data: request.toJson());
    return User.fromJson(response.data);
  }

  /// 部分更新 — PATCH /users/:id
  Future<User> patchUser(int id, Map<String, dynamic> fields) async {
    final response = await _dio.patch('/users/$id', data: fields);
    return User.fromJson(response.data);
  }

  /// 删除 — DELETE /users/:id
  Future<void> deleteUser(int id) async {
    await _dio.delete('/users/$id');
  }

  /// 批量删除 — DELETE /users/batch
  Future<void> batchDeleteUsers(List<int> ids) async {
    await _dio.delete('/users/batch', data: {'ids': ids});
  }
}

/// 分页结果通用模型
class PageResult<T> {
  final List<T> items;
  final int total;
  final int page;
  final int size;

  PageResult({
    required this.items,
    required this.total,
    required this.page,
    required this.size,
  });

  factory PageResult.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    return PageResult(
      items: (json['items'] as List).map((e) => fromJsonT(e)).toList(),
      total: json['total'] as int,
      page: json['page'] as int,
      size: json['size'] as int,
    );
  }

  bool get hasMore => page * size < total;
}
```

### 四、文件上传与下载

#### 4.1 文件上传（multipart/formData）

```dart
import 'package:dio/dio.dart';
import 'dart:typed_data';

/// 文件上传示例
class FileUploadService {
  final Dio _dio = DioConfig.dio;

  /// 单文件上传
  Future<String> uploadSingleFile(String filePath) async {
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(
        filePath,
        filename: filePath.split('/').last, // 文件名
      ),
      'category': 'avatar', // 附加字段
    });

    final response = await _dio.post('/upload', data: formData);
    return response.data['url']; // 返回文件 URL
  }

  /// 多文件上传
  Future<List<String>> uploadMultipleFiles(List<String> filePaths) async {
    // 构建多文件 FormData
    final files = await Future.wait(
      filePaths.map((path) => MultipartFile.fromFile(
        path,
        filename: path.split('/').last,
      )),
    );

    final formData = FormData.fromMap({
      'files': files, // 同名字段传递 List
      'category': 'gallery',
    });

    final response = await _dio.post('/upload/batch', data: formData);
    return (response.data['urls'] as List).cast<String>();
  }

  /// 上传字节数据（如拍照后的内存图片）
  Future<String> uploadBytes({
    required Uint8List bytes,
    required String fileName,
    String contentType = 'image/jpeg',
  }) async {
    final formData = FormData.fromMap({
      'file': MultipartFile.fromBytes(
        bytes,
        filename: fileName,
        contentType: DioMediaType.parse(contentType), // 指定 Content-Type
      ),
    });

    final response = await _dio.post('/upload', data: formData);
    return response.data['url'];
  }

  /// 带进度监听的上传
  Future<String> uploadWithProgress(
    String filePath, {
    CancelToken? cancelToken,
  }) async {
    final formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(filePath),
    });

    final response = await _dio.post(
      '/upload',
      data: formData,
      cancelToken: cancelToken,
      onSendProgress: (sent, total) {
        // sent: 已发送字节, total: 总字节
        if (total > 0) {
          final progress = (sent / total * 100).toStringAsFixed(1);
          print('上传进度: $progress% ($sent/$total)');
          // 实际项目中通知 UI 更新进度条
        }
      },
    );
    return response.data['url'];
  }

  /// 分片上传（大文件）
  Future<String> uploadChunked({
    required String filePath,
    required int chunkSize, // 每片大小（字节）
    CancelToken? cancelToken,
  }) async {
    final file = File(filePath);
    final fileSize = await file.length();
    final fileName = filePath.split('/').last;
    final totalChunks = (fileSize / chunkSize).ceil();

    // 1. 获取上传 ID
    final initResp = await _dio.post('/upload/init', data: {
      'fileName': fileName,
      'fileSize': fileSize,
      'chunkSize': chunkSize,
      'totalChunks': totalChunks,
    });
    final uploadId = initResp.data['uploadId'];

    // 2. 逐片上传
    for (var i = 0; i < totalChunks; i++) {
      final offset = i * chunkSize;
      final end = (offset + chunkSize > fileSize) ? fileSize : offset + chunkSize;
      final chunk = file.openRead(offset, end);

      final formData = FormData.fromMap({
        'chunk': MultipartFile.fromStream(
          () => chunk,
          end - offset,
          filename: '$fileName.part$i',
        ),
        'uploadId': uploadId,
        'chunkIndex': i,
        'totalChunks': totalChunks,
      });

      await _dio.post(
        '/upload/chunk',
        data: formData,
        cancelToken: cancelToken,
        onSendProgress: (sent, total) {
          // 整体进度 = (已完成片数 * chunkSize + 当前片已传) / fileSize
          final overallProgress =
              (i * chunkSize + sent) / fileSize * 100;
          print('整体进度: ${overallProgress.toStringAsFixed(1)}%');
        },
      );
    }

    // 3. 通知合并
    final mergeResp = await _dio.post('/upload/merge', data: {
      'uploadId': uploadId,
      'fileName': fileName,
    });
    return mergeResp.data['url'];
  }
}
```

#### 4.2 文件下载（进度监听 + 断点续传）

```dart
import 'package:dio/dio.dart';
import 'package:path_provider/path_provider.dart';
import 'dart:io';

/// 文件下载服务
class FileDownloadService {
  final Dio _dio = DioConfig.dio;

  /// 基础下载
  Future<String> downloadFile(String url, String savePath) async {
    await _dio.download(url, savePath);
    return savePath;
  }

  /// 带进度监听的下载
  Future<String> downloadWithProgress(
    String url, {
    CancelToken? cancelToken,
    Function(int, int)? onProgress, // received, total
  }) async {
    final savePath = await _getSavePath(url);

    await _dio.download(
      url,
      savePath,
      cancelToken: cancelToken,
      onReceiveProgress: (received, total) {
        if (total > 0) {
          final progress = (received / total * 100).toStringAsFixed(1);
          print('下载进度: $progress%');
          onProgress?.call(received, total);
        }
      },
    );
    return savePath;
  }

  /// 断点续传下载
  /// 原理: 检查本地已下载文件大小，用 Range 请求头续传
  Future<String> downloadWithResume(
    String url, {
    CancelToken? cancelToken,
    Function(int, int)? onProgress,
  }) async {
    final savePath = await _getSavePath(url);
    final file = File(savePath);

    // 检查已下载的字节数
    int downloadedBytes = 0;
    if (await file.exists()) {
      downloadedBytes = await file.length();
      print('已有部分下载: $downloadedBytes bytes');
    }

    // 设置 Range 请求头
    final options = Options();
    if (downloadedBytes > 0) {
      options.headers = {'Range': 'bytes=$downloadedBytes-'};
    }

    // 使用 openWrite 以追加模式写入
    final raf = await file.open(mode: FileMode.append);

    try {
      await _dio.get(
        url,
        options: Options(
          responseType: ResponseType.stream, // 流式接收
          headers: downloadedBytes > 0
              ? {'Range': 'bytes=$downloadedBytes-'}
              : null,
        ),
        cancelToken: cancelToken,
        onReceiveProgress: (received, total) {
          // total 为完整文件大小，received 为当前会话已接收
          final overallReceived = downloadedBytes + received;
          final overallTotal = downloadedBytes + total;
          onProgress?.call(overallReceived, overallTotal);
        },
      ).then((response) async {
        // 流式写入文件
        final stream = response.data as ResponseBody;
        await for (final chunk in stream.stream) {
          await raf.writeFrom(chunk);
        }
      });
    } finally {
      await raf.close();
    }

    return savePath;
  }

  /// 根据下载 URL 生成保存路径
  Future<String> _getSavePath(String url) async {
    final dir = await getApplicationDocumentsDirectory();
    final fileName = url.split('/').last.split('?').first;
    return '${dir.path}/downloads/$fileName';
  }
}
```

### 五、WebSocket 实时通信

```dart
import 'package:web_socket_channel/web_socket_channel.dart';
import 'package:web_socket_channel/status.dart' as status;
import 'dart:async';
import 'dart:convert';

/// WebSocket 服务 — 自动重连 + 心跳保活
class WebSocketService {
  static const _heartbeatInterval = Duration(seconds: 30);
  static const _reconnectDelay = Duration(seconds: 3);
  static const _maxReconnectAttempts = 5;

  WebSocketChannel? _channel;
  String? _url;
  String? _token;
  int _reconnectAttempts = 0;
  bool _isManualClose = false;

  // 心跳定时器
  Timer? _heartbeatTimer;
  // 重连定时器
  Timer? _reconnectTimer;

  // 消息流控制器
  final _messageController = StreamController<String>.broadcast();
  Stream<String> get messageStream => _messageController.stream;

  // 连接状态
  final _connectionStateController = StreamController<WsState>.broadcast();
  Stream<WsState> get connectionState => _connectionStateController.stream;

  /// 连接 WebSocket
  Future<void> connect(String url, {String? token}) async {
    _url = url;
    _token = token;
    _isManualClose = false;

    try {
      _updateState(WsState.connecting);

      // 构建 WebSocket URL（含 Token 认证）
      final wsUrl = token != null
          ? '$url?token=$token'
          : url;

      _channel = WebSocketChannel.connect(Uri.parse(wsUrl));

      // 监听消息
      _channel!.stream.listen(
        _onMessage,
        onError: _onError,
        onDone: _onDone,
        cancelOnError: false,
      );

      _updateState(WsState.connected);
      _reconnectAttempts = 0;
      _startHeartbeat();
    } catch (e) {
      print('WebSocket 连接失败: $e');
      _scheduleReconnect();
    }
  }

  /// 发送消息
  void send(String message) {
    if (_channel != null) {
      _channel!.sink.add(message);
    }
  }

  /// 发送 JSON 消息
  void sendJson(Map<String, dynamic> data) {
    send(json.encode(data));
  }

  /// 断开连接
  void disconnect() {
    _isManualClose = true;
    _stopHeartbeat();
    _reconnectTimer?.cancel();
    _channel?.sink.close(status.goingAway);
    _updateState(WsState.disconnected);
  }

  /// 收到消息回调
  void _onMessage(dynamic message) {
    // 收到消息，重置心跳计时
    _resetHeartbeat();

    if (message is String) {
      // 心跳响应，不需要广播
      if (message == 'pong' || message == '{"type":"pong"}') {
        return;
      }
      _messageController.add(message);
    }
  }

  /// 错误回调
  void _onError(dynamic error) {
    print('WebSocket 错误: $error');
    _updateState(WsState.error);
    _scheduleReconnect();
  }

  /// 连接关闭回调
  void _onDone() {
    print('WebSocket 连接关闭');
    _stopHeartbeat();
    _updateState(WsState.disconnected);
    if (!_isManualClose) {
      _scheduleReconnect();
    }
  }

  /// 启动心跳
  void _startHeartbeat() {
    _heartbeatTimer?.cancel();
    _heartbeatTimer = Timer.periodic(_heartbeatInterval, (_) {
      send('ping'); // 或 sendJson({'type': 'ping'});
    });
  }

  /// 收到消息后重置心跳计时
  void _resetHeartbeat() {
    _stopHeartbeat();
    _startHeartbeat();
  }

  /// 停止心跳
  void _stopHeartbeat() {
    _heartbeatTimer?.cancel();
    _heartbeatTimer = null;
  }

  /// 计划重连
  void _scheduleReconnect() {
    if (_isManualClose) return;
    if (_reconnectAttempts >= _maxReconnectAttempts) {
      print('达到最大重连次数，停止重连');
      _updateState(WsState.failed);
      return;
    }

    _reconnectAttempts++;
    final delay = Duration(
      seconds: _reconnectDelay.inSeconds * _reconnectAttempts, // 递增延迟
    );

    print('将在 ${delay.inSeconds}s 后重连 (第 $_reconnectAttempts 次)');

    _reconnectTimer?.cancel();
    _reconnectTimer = Timer(delay, () {
      if (_url != null) {
        connect(_url!, token: _token);
      }
    });
  }

  void _updateState(WsState state) {
    _connectionStateController.add(state);
  }

  void dispose() {
    disconnect();
    _messageController.close();
    _connectionStateController.close();
  }
}

/// WebSocket 连接状态枚举
enum WsState {
  disconnected,
  connecting,
  connected,
  error,
  failed,
}
```

**WebSocket 在 Widget 中使用：**

```dart
import 'package:flutter/material.dart';

/// 聊天页面 — WebSocket 实时通信示例
class ChatPage extends StatefulWidget {
  const ChatPage({super.key});

  @override
  State<ChatPage> createState() => _ChatPageState();
}

class _ChatPageState extends State<ChatPage> {
  final _wsService = WebSocketService();
  final _messageController = TextEditingController();
  final List<ChatMessage> _messages = [];
  StreamSubscription? _subscription;

  @override
  void initState() {
    super.initState();
    _connectWebSocket();
  }

  void _connectWebSocket() async {
    // 连接 WebSocket
    await _wsService.connect(
      'wss://api.example.com/ws/chat',
      token: 'user_token_here',
    );

    // 监听消息
    _subscription = _wsService.messageStream.listen((message) {
      final data = json.decode(message) as Map<String, dynamic>;
      setState(() {
        _messages.add(ChatMessage(
          content: data['content'] as String,
          sender: data['sender'] as String,
          isMe: false,
          timestamp: DateTime.now(),
        ));
      });
    });
  }

  void _sendMessage() {
    final text = _messageController.text.trim();
    if (text.isEmpty) return;

    // 发送消息
    _wsService.sendJson({
      'type': 'chat',
      'content': text,
      'timestamp': DateTime.now().toIso8601String(),
    });

    setState(() {
      _messages.add(ChatMessage(
        content: text,
        sender: 'me',
        isMe: true,
        timestamp: DateTime.now(),
      ));
    });
    _messageController.clear();
  }

  @override
  void dispose() {
    _subscription?.cancel();
    _wsService.dispose();
    _messageController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chat')),
      body: Column(
        children: [
          // 消息列表
          Expanded(
            child: ListView.builder(
              itemCount: _messages.length,
              itemBuilder: (context, index) {
                final msg = _messages[index];
                return ListTile(
                  title: Text(msg.content),
                  subtitle: Text(msg.sender),
                );
              },
            ),
          ),
          // 输入区
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _messageController,
                    decoration: const InputDecoration(
                      hintText: '输入消息...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _sendMessage(),
                  ),
                ),
                IconButton(
                  icon: const Icon(Icons.send),
                  onPressed: _sendMessage,
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class ChatMessage {
  final String content;
  final String sender;
  final bool isMe;
  final DateTime timestamp;

  ChatMessage({
    required this.content,
    required this.sender,
    required this.isMe,
    required this.timestamp,
  });
}
```

### 六、JSON 序列化与反序列化

#### 6.1 手写 fromJson / toJson（简单场景）

```dart
/// 手写序列化 — 适合模型数量少的小项目
class User {
  final int id;
  final String name;
  final String email;
  final String? avatar;   // 可空字段
  final Role role;
  final DateTime createdAt;

  User({
    required this.id,
    required this.name,
    required this.email,
    this.avatar,
    required this.role,
    required this.createdAt,
  });

  /// 从 JSON Map 创建对象
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'] as int,
      name: json['name'] as String,
      email: json['email'] as String,
      avatar: json['avatar'] as String?,   // 可空类型
      role: Role.values.firstWhere(
        (e) => e.name == json['role'],
        orElse: () => Role.user,           // 默认值
      ),
      createdAt: DateTime.parse(json['createdAt'] as String),
    );
  }

  /// 转为 JSON Map
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'avatar': avatar,
      'role': role.name,
      'createdAt': createdAt.toIso8601String(),
    };
  }
}

enum Role { admin, user, guest }
```

#### 6.2 json_serializable + build_runner（推荐）

```dart
import 'package:json_annotation/json_annotation.dart';

// 关联生成的文件
part 'user_model.g.dart';

/// 使用 json_serializable 自动生成序列化代码
/// 1. 编写模型 + 添加注解
/// 2. 运行: dart run build_runner build --delete-conflicting-outputs
/// 3. 自动生成 user_model.g.dart

@JsonSerializable(explicitToJson: true) // 嵌套对象也序列化
class UserModel {
  final int id;

  @JsonKey(name: 'user_name') // JSON 字段名与 Dart 属性名不同
  final String userName;

  @JsonKey(defaultValue: '') // 缺失时的默认值
  final String email;

  @JsonKey(fromJson: _avatarFromJson, toJson: _avatarToJson) // 自定义转换
  final String avatarUrl;

  @JsonKey(unknownEnumValue: Role.user) // 未知枚举值的默认值
  final Role role;

  @JsonKey(name: 'created_at')
  final DateTime createdAt;

  // 嵌套对象 — 需要该类也有 JsonSerializable 注解
  final AddressModel? address;

  // 列表类型
  @JsonKey(defaultValue: [])
  final List<String> tags;

  UserModel({
    required this.id,
    required this.userName,
    this.email = '',
    required this.avatarUrl,
    required this.role,
    required this.createdAt,
    this.address,
    this.tags = const [],
  });

  // 生成代码的入口
  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
  Map<String, dynamic> toJson() => _$UserModelToJson(this);

  // 自定义 JSON 转换函数
  static String _avatarFromJson(dynamic value) {
    if (value is String && value.isNotEmpty) {
      return value.startsWith('http') ? value : 'https://cdn.example.com$value';
    }
    return 'https://cdn.example.com/default.png';
  }

  static String _avatarToJson(String url) => url;
}

@JsonSerializable()
class AddressModel {
  final String city;
  final String street;

  AddressModel({required this.city, required this.street});

  factory AddressModel.fromJson(Map<String, dynamic> json) =>
      _$AddressModelFromJson(json);
  Map<String, dynamic> toJson() => _$AddressModelToJson(this);
}
```

**build_runner 命令：**

```bash
# 一次性生成
dart run build_runner build --delete-conflicting-outputs

# 持续监听文件变化自动生成
dart run build_runner watch --delete-conflicting-outputs

# 清理生成文件
dart run build_runner clean
```

### 七、SharedPreferences 键值对存储

```dart
import 'package:shared_preferences/shared_preferences.dart';

/// SharedPreferences 封装 — 类型安全的键值存储
/// 适合: 用户偏好、配置项、简单标记
/// 不适合: 大量数据、结构化数据、敏感信息
class PrefsService {
  static SharedPreferences? _prefs;

  /// 初始化（在 main() 中调用）
  static Future<void> init() async {
    _prefs = await SharedPreferences.getInstance();
  }

  static SharedPreferences get _instance {
    if (_prefs == null) throw Exception('PrefsService 未初始化');
    return _prefs!;
  }

  // ============ 类型安全存取 ============

  /// String
  static String? getString(PrefKey key) => _instance.getString(key.name);
  static Future<bool> setString(PrefKey key, String value) =>
      _instance.setString(key.name, value);

  /// int
  static int? getInt(PrefKey key) => _instance.getInt(key.name);
  static Future<bool> setInt(PrefKey key, int value) =>
      _instance.setInt(key.name, value);

  /// double
  static double? getDouble(PrefKey key) => _instance.getDouble(key.name);
  static Future<bool> setDouble(PrefKey key, double value) =>
      _instance.setDouble(key.name, value);

  /// bool
  static bool? getBool(PrefKey key) => _instance.getBool(key.name);
  static Future<bool> setBool(PrefKey key, bool value) =>
      _instance.setBool(key.name, value);

  /// StringList
  static List<String>? getStringList(PrefKey key) =>
      _instance.getStringList(key.name);
  static Future<bool> setStringList(PrefKey key, List<String> value) =>
      _instance.setStringList(key.name, value);

  // ============ 便捷方法 ============

  /// 删除指定键
  static Future<bool> remove(PrefKey key) => _instance.remove(key.name);

  /// 清除所有数据
  static Future<bool> clear() => _instance.clear();

  /// 检查键是否存在
  static bool contains(PrefKey key) => _instance.containsKey(key.name);

  /// 读取复杂对象（JSON 序列化后存储）
  static T? getObject<T>(
    PrefKey key,
    T Function(Map<String, dynamic>) fromJson,
  ) {
    final jsonStr = _instance.getString(key.name);
    if (jsonStr == null) return null;
    return fromJson(json.decode(jsonStr) as Map<String, dynamic>);
  }

  /// 存储复杂对象（序列化为 JSON 字符串）
  static Future<bool> setObject(PrefKey key, dynamic object) {
    return _instance.setString(key.name, json.encode(object));
  }
}

/// 键名枚举 — 防止拼写错误
enum PrefKey {
  themeMode,        // 主题模式: 'light' / 'dark' / 'system'
  locale,           // 语言: 'zh' / 'en'
  accessToken,      // 访问令牌（实际应存 flutter_secure_storage）
  isFirstLaunch,    // 是否首次启动
  lastReadArticleId,// 最后阅读的文章 ID
  recentSearches,   // 最近搜索词列表
  fontSize,         // 字体大小
  pushEnabled,      // 推送通知开关
}
```

**在 main() 中初始化：**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized(); // 必须在 async main 中调用

  // 初始化 SharedPreferences
  await PrefsService.init();

  // 读取偏好设置
  final themeMode = PrefsService.getString(PrefKey.themeMode) ?? 'system';
  final isFirstLaunch = PrefsService.getBool(PrefKey.isFirstLaunch) ?? true;

  runApp(MyApp(themeMode: themeMode, isFirstLaunch: isFirstLaunch));
}
```

### 八、SQLite 数据库

#### 8.1 sqflite 原生操作

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart' as p;

/// sqflite 数据库操作 — 原生 SQL 方式
class AppDatabase {
  static Database? _database;

  /// 获取数据库实例
  Future<Database> get database async {
    _database ??= await _initDatabase();
    return _database!;
  }

  /// 初始化数据库
  Future<Database> _initDatabase() async {
    final dbPath = await getDatabasesPath();
    final path = p.join(dbPath, 'app.db');

    return await openDatabase(
      path,
      version: 3, // 数据库版本号，每次 schema 变更递增
      onCreate: _onCreate,         // 首次创建时执行
      onUpgrade: _onUpgrade,       // 版本升级时执行
      onConfigure: _onConfigure,   // 数据库配置（如外键支持）
    );
  }

  /// 数据库配置
  Future<void> _onConfigure(Database db) async {
    // 启用外键约束（默认关闭）
    await db.execute('PRAGMA foreign_keys = ON');
  }

  /// 创建表
  Future<void> _onCreate(Database db, int version) async {
    // 一次性执行所有版本的建表语句
    await db.execute('''
      CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        avatar TEXT,
        role TEXT NOT NULL DEFAULT 'user',
        created_at INTEGER NOT NULL,
        updated_at INTEGER NOT NULL
      )
    ''');

    await db.execute('''
      CREATE TABLE articles (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        author_id INTEGER NOT NULL,
        status TEXT NOT NULL DEFAULT 'draft',
        published_at INTEGER,
        created_at INTEGER NOT NULL,
        FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE CASCADE
      )
    ''');

    await db.execute('''
      CREATE TABLE article_tags (
        article_id INTEGER NOT NULL,
        tag TEXT NOT NULL,
        PRIMARY KEY (article_id, tag),
        FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE
      )
    ''');

    // 创建索引
    await db.execute(
      'CREATE INDEX idx_articles_author ON articles(author_id)',
    );
    await db.execute(
      'CREATE INDEX idx_articles_status ON articles(status)',
    );
    await db.execute(
      'CREATE INDEX idx_article_tags_tag ON article_tags(tag)',
    );
  }

  /// 数据库升级迁移
  Future<void> _onUpgrade(Database db, int oldVersion, int newVersion) async {
    // 逐版本迁移，避免跳版问题
    if (oldVersion < 2) {
      // v1 → v2: 添加 articles 表的 summary 字段
      await db.execute('ALTER TABLE articles ADD COLUMN summary TEXT');
    }
    if (oldVersion < 3) {
      // v2 → v3: 添加 bookmarks 表
      await db.execute('''
        CREATE TABLE bookmarks (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          article_id INTEGER NOT NULL,
          user_id INTEGER NOT NULL,
          created_at INTEGER NOT NULL,
          UNIQUE(article_id, user_id),
          FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE,
          FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
        )
      ''');
    }
  }
}
```

**sqflite CRUD 操作：**

```dart
/// 用户数据访问对象 (DAO)
class UserDao {
  final AppDatabase _db = AppDatabase();

  /// 插入 — INSERT
  Future<int> insert(User user) async {
    final db = await _db.database;
    return await db.insert(
      'users',
      user.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace, // 冲突时替换
    );
  }

  /// 批量插入 — BATCH INSERT
  Future<void> insertBatch(List<User> users) async {
    final db = await _db.database;
    final batch = db.batch();
    for (final user in users) {
      batch.insert('users', user.toMap(),
          conflictAlgorithm: ConflictAlgorithm.ignore);
    }
    await batch.commit(noResult: true); // 无需返回结果，性能更优
  }

  /// 查询所有 — SELECT
  Future<List<User>> getAll() async {
    final db = await _db.database;
    final maps = await db.query(
      'users',
      columns: ['id', 'name', 'email', 'avatar', 'role', 'created_at'],
      orderBy: 'created_at DESC', // 按创建时间倒序
    );
    return maps.map((m) => User.fromMap(m)).toList();
  }

  /// 条件查询 — WHERE
  Future<List<User>> search(String keyword) async {
    final db = await _db.database;
    final maps = await db.query(
      'users',
      where: 'name LIKE ? OR email LIKE ?',
      whereArgs: ['%$keyword%', '%$keyword%'],
      limit: 20,
    );
    return maps.map((m) => User.fromMap(m)).toList();
  }

  /// 单条查询
  Future<User?> getById(int id) async {
    final db = await _db.database;
    final maps = await db.query(
      'users',
      where: 'id = ?',
      whereArgs: [id],
      limit: 1,
    );
    if (maps.isEmpty) return null;
    return User.fromMap(maps.first);
  }

  /// 更新 — UPDATE
  Future<int> update(User user) async {
    final db = await _db.database;
    return await db.update(
      'users',
      {
        ...user.toMap(),
        'updated_at': DateTime.now().millisecondsSinceEpoch,
      },
      where: 'id = ?',
      whereArgs: [user.id],
    );
  }

  /// 删除 — DELETE
  Future<int> delete(int id) async {
    final db = await _db.database;
    return await db.delete('users', where: 'id = ?', whereArgs: [id]);
  }

  /// 事务操作 — 保证原子性
  Future<void> transferRole(int fromId, int toId) async {
    final db = await _db.database;
    await db.transaction((txn) async {
      // 检查原用户角色
      final maps = await txn.query(
        'users',
        columns: ['role'],
        where: 'id = ?',
        whereArgs: [fromId],
      );
      if (maps.isEmpty) throw Exception('用户不存在');

      // 转移角色
      await txn.update(
        'users',
        {'role': 'user'},
        where: 'id = ?',
        whereArgs: [fromId],
      );
      await txn.update(
        'users',
        {'role': maps.first['role']},
        where: 'id = ?',
        whereArgs: [toId],
      );
    });
  }

  /// 原始 SQL 查询（复杂联表）
  Future<List<Map<String, dynamic>>> getArticleStats(int userId) async {
    final db = await _db.database;
    return await db.rawQuery('''
      SELECT
        u.name AS author_name,
        COUNT(a.id) AS article_count,
        SUM(CASE WHEN a.status = 'published' THEN 1 ELSE 0 END) AS published_count
      FROM users u
      LEFT JOIN articles a ON a.author_id = u.id
      WHERE u.id = ?
      GROUP BY u.id
    ''', [userId]);
  }
}
```

#### 8.2 drift（原 moor）类型安全 ORM

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;
import 'dart:io';

// drift 代码生成需要 part 指令
part 'app_drift_db.g.dart';

/// ============ 表定义 ============

/// 用户表
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().withLength(min: 1, max: 100)();
  TextColumn get email => text().unique()();
  TextColumn get avatar => text().nullable()();
  TextColumn get role => textEnum<Role>().withDefault(Constant('user'))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  DateTimeColumn get updatedAt => dateTime().withDefault(currentDateAndTime)();
}

/// 文章表
class Articles extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 500)();
  TextColumn get content => text()();
  IntColumn get authorId => integer().references(Users, #id)();
  TextColumn get status => textEnum<ArticleStatus>()
      .withDefault(Constant('draft'))();
  DateTimeColumn get publishedAt => dateTime().nullable()();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

/// ============ 数据库定义 ============

@DriftDatabase(tables: [Users, Articles])
class AppDriftDB extends _$AppDriftDB {
  AppDriftDB() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (Migrator m) async {
      await m.createAll();
    },
    onUpgrade: (Migrator m, int from, int to) async {
      // 逐版本迁移
    },
    beforeOpen: (details) async {
      // 启用外键
      await customStatement('PRAGMA foreign_keys = ON');
    },
  );
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'app_drift.db'));
    return NativeDatabase.createInBackground(file); // 后台线程打开数据库
  });
}

/// ============ DAO（数据访问对象）============

/// drift DAO — 支持响应式查询（watch）
@DriftAccessor(tables: [Users])
class UserDaoDrift extends DatabaseAccessor<AppDriftDB> with _$UserDaoDriftMixin {
  UserDaoDrift(super.db);

  /// 普通查询 — 返回 Future<List>
  Future<List<User>> getAllUsers() => select(users).get();

  /// 响应式查询 — 返回 Stream<List>，数据变化时自动通知
  Stream<List<User>> watchAllUsers() {
    return (select(users)
          ..orderBy([
            (t) => OrderingTerm.desc(t.createdAt),
          ]))
        .watch();
  }

  /// 条件查询
  Future<List<User>> searchUsers(String keyword) {
    return (select(users)
          ..where((t) =>
              t.name.like('%$keyword%') | t.email.like('%$keyword%'))
          ..limit(20))
        .get();
  }

  /// 单条查询
  Future<User?> getUserById(int id) {
    return (select(users)..where((t) => t.id.equals(id))).getSingleOrNull();
  }

  /// 响应式单条查询
  Stream<User?> watchUserById(int id) {
    return (select(users)..where((t) => t.id.equals(id)))
        .watchSingleOrNull();
  }

  /// 插入
  Future<int> insertUser(UsersCompanion user) => into(users).insert(user);

  /// 批量插入
  Future<void> insertBatch(List<UsersCompanion> userList) async {
    await batch((b) {
      b.insertAll(users, userList);
    });
  }

  /// 更新
  Future<bool> updateUser(UsersCompanion user) =>
      update(users).replace(user); // replace 需要 id 字段

  /// 删除
  Future<int> deleteUser(int id) =>
      (delete(users)..where((t) => t.id.equals(id))).go();

  /// 复杂查询 — 联表
  Future<List<UserWithArticleCount>> getUserArticleCounts() {
    final query = select(users).join([
      leftOuterJoin(
        articles,
        articles.authorId.equalsExp(users.id),
      ),
    ]);
    // 使用自定义结果类型
    // ... 需配合 @UseRowType 注解
    return query.map((row) {
      // 处理联表结果
      return UserWithArticleCount(
        user: row.readTable(users),
        articleCount: 0, // 简化示例
      );
    }).get();
  }
}

class UserWithArticleCount {
  final User user;
  final int articleCount;
  UserWithArticleCount({required this.user, required this.articleCount});
}
```

**drift 在 Widget 中的响应式用法：**

```dart
/// 使用 StreamBuilder 监听数据库变化
class UserListPage extends StatelessWidget {
  final UserDaoDrift _userDao = AppDriftDB().userDao;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('用户列表')),
      body: StreamBuilder<List<User>>(
        stream: _userDao.watchAllUsers(), // 响应式查询
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          final users = snapshot.data ?? [];
          if (users.isEmpty) {
            return const Center(child: Text('暂无用户'));
          }
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return ListTile(
                leading: CircleAvatar(
                  backgroundImage: user.avatar != null
                      ? NetworkImage(user.avatar!)
                      : null,
                  child: user.avatar == null
                      ? Text(user.name[0])
                      : null,
                ),
                title: Text(user.name),
                subtitle: Text(user.email),
                trailing: Text(user.role),
              );
            },
          );
        },
      ),
    );
  }
}
```

### 九、文件系统操作

```dart
import 'package:path_provider/path_provider.dart';
import 'dart:io';

/// 文件系统操作 — path_provider 获取路径 + dart:io 文件操作
class FileService {
  /// ============ 获取常用目录路径 ============

  /// 临时目录（可被系统清理，适合缓存）
  Future<Directory> get tempDir async => await getTemporaryDirectory();

  /// 文档目录（不会被清理，适合持久化数据）
  Future<Directory> get docDir async => await getApplicationDocumentsDirectory();

  /// 应用支持目录（iOS: Library/Application Support, Android: app data）
  Future<Directory> get supportDir async =>
      await getApplicationSupportDirectory();

  /// 外部存储目录（Android 专用，需权限）
  Future<Directory?> get externalDir async {
    if (Platform.isAndroid) {
      return await getExternalStorageDirectory();
    }
    return null;
  }

  /// 下载目录（桌面端专用）
  Future<Directory?> get downloadDir async {
    if (Platform.isLinux || Platform.isMacOS || Platform.isWindows) {
      return await getDownloadsDirectory();
    }
    return null;
  }

  /// ============ 文件操作 ============

  /// 写入文本文件
  Future<File> writeText(String fileName, String content) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/$fileName');
    return await file.writeAsString(content);
  }

  /// 读取文本文件
  Future<String?> readText(String fileName) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/$fileName');
    if (await file.exists()) {
      return await file.readAsString();
    }
    return null;
  }

  /// 写入二进制文件
  Future<File> writeBytes(String fileName, List<int> bytes) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/$fileName');
    return await file.writeAsBytes(bytes);
  }

  /// 追加内容
  Future<File> appendText(String fileName, String content) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/$fileName');
    return await file.writeAsString(content, mode: FileMode.append);
  }

  /// 删除文件
  Future<void> deleteFile(String fileName) async {
    final dir = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/$fileName');
    if (await file.exists()) {
      await file.delete();
    }
  }

  /// 列出目录下所有文件
  Future<List<FileSystemEntity>> listFiles(String subDir) async {
    final dir = await getApplicationDocumentsDirectory();
    final directory = Directory('${dir.path}/$subDir');
    if (!await directory.exists()) return [];
    return directory.list().toList();
  }

  /// 计算缓存大小
  Future<int> calculateCacheSize() async {
    final dir = await getTemporaryDirectory();
    if (!await dir.exists()) return 0;
    int totalSize = 0;
    await for (final entity in dir.list(recursive: true)) {
      if (entity is File) {
        totalSize += await entity.length();
      }
    }
    return totalSize;
  }

  /// 清除缓存
  Future<void> clearCache() async {
    final dir = await getTemporaryDirectory();
    if (await dir.exists()) {
      await dir.delete(recursive: true);
      await dir.create(); // 重建空目录
    }
  }

  /// 格式化文件大小
  String formatFileSize(int bytes) {
    if (bytes < 1024) return '$bytes B';
    if (bytes < 1024 * 1024) return '${(bytes / 1024).toStringAsFixed(1)} KB';
    if (bytes < 1024 * 1024 * 1024) {
      return '${(bytes / (1024 * 1024)).toStringAsFixed(1)} MB';
    }
    return '${(bytes / (1024 * 1024 * 1024)).toStringAsFixed(1)} GB';
  }
}
```

### 十、安全存储（flutter_secure_storage）

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

/// 安全存储服务 — 加密存储敏感数据
/// iOS: Keychain
/// Android: EncryptedSharedPreferences (MasterKey + AES)
/// Web: 不支持（需降级方案）
/// 桌面: 不支持（需降级方案）
class SecureStorageService {
  static FlutterSecureStorage? _storage;

  static FlutterSecureStorage get _instance {
    _storage ??= const FlutterSecureStorage(
      aOptions: AndroidOptions(
        encryptedSharedPreferences: true, // Android 加密
      ),
      iOptions: IOSOptions(
        accessibility: KeychainAccessibility.first_unlock_this_device,
      ),
    );
    return _storage!;
  }

  // ============ 基础存取 ============

  /// 存储字符串
  static Future<void> write(SecureKey key, String value) async {
    await _instance.write(key: key.name, value: value);
  }

  /// 读取字符串
  static Future<String?> read(SecureKey key) async {
    return await _instance.read(key: key.name);
  }

  /// 删除
  static Future<void> delete(SecureKey key) async {
    await _instance.delete(key: key.name);
  }

  /// 检查是否存在
  static Future<bool> contains(SecureKey key) async {
    return await _instance.containsKey(key: key.name);
  }

  /// 清除所有
  static Future<void> clearAll() async {
    await _instance.deleteAll();
  }

  /// 读取所有
  static Future<Map<String, String>> readAll() async {
    return await _instance.readAll();
  }

  // ============ 业务方法 ============

  /// 保存 Token 对
  static Future<void> saveTokenPair({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      write(SecureKey.accessToken, accessToken),
      write(SecureKey.refreshToken, refreshToken),
    ]);
  }

  /// 读取 Access Token
  static Future<String?> get accessToken => read(SecureKey.accessToken);

  /// 读取 Refresh Token
  static Future<String?> get refreshToken => read(SecureKey.refreshToken);

  /// 清除登录信息（登出时调用）
  static Future<void> clearAuthData() async {
    await Future.wait([
      delete(SecureKey.accessToken),
      delete(SecureKey.refreshToken),
      delete(SecureKey.userId),
    ]);
  }
}

/// 安全存储键名枚举
enum SecureKey {
  accessToken,
  refreshToken,
  userId,
  biometricKey,
  encryptionKey,
}
```

### 十一、数据缓存策略

#### 11.1 内存缓存

```dart
import 'dart:collection';

/// 内存缓存 — LRU 淘汰策略
class MemoryCache<T> {
  final int maxSize; // 最大缓存条目数
  final Duration defaultTtl; // 默认过期时间

  // LinkedHashMap 以访问顺序排列，实现 LRU
  final _cache = LinkedHashMap<String, _CacheEntry<T>>();

  MemoryCache({
    this.maxSize = 100,
    this.defaultTtl = const Duration(minutes: 30),
  });

  /// 写入缓存
  void set(String key, T value, {Duration? ttl}) {
    // 如果已存在，先删除（保证插入到末尾）
    _cache.remove(key);

    // 超过最大容量，淘汰最久未访问的
    if (_cache.length >= maxSize) {
      _cache.remove(_cache.keys.first);
    }

    _cache[key] = _CacheEntry(
      value: value,
      expireAt: DateTime.now().add(ttl ?? defaultTtl),
    );
  }

  /// 读取缓存
  T? get(String key) {
    final entry = _cache[key];
    if (entry == null) return null;

    // 检查过期
    if (entry.isExpired) {
      _cache.remove(key);
      return null;
    }

    // LRU: 删除后重新插入，移到末尾（最近访问）
    _cache.remove(key);
    _cache[key] = entry;

    return entry.value;
  }

  /// 删除缓存
  void remove(String key) => _cache.remove(key);

  /// 清空缓存
  void clear() => _cache.clear();

  /// 获取缓存条目数
  int get length => _cache.length;

  /// 清除过期条目
  void cleanExpired() {
    final now = DateTime.now();
    _cache.removeWhere((key, entry) => entry.expireAt.isBefore(now));
  }
}

/// 缓存条目
class _CacheEntry<T> {
  final T value;
  final DateTime expireAt;

  _CacheEntry({required this.value, required this.expireAt});

  bool get isExpired => DateTime.now().isAfter(expireAt);
}
```

#### 11.2 磁盘缓存

```dart
import 'dart:convert';
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:crypto/crypto.dart';

/// 磁盘缓存 — 将网络响应持久化到文件系统
class DiskCache {
  static DiskCache? _instance;
  static DiskCache get instance => _instance ??= DiskCache._();

  DiskCache._();

  Directory? _cacheDir;

  /// 获取缓存目录
  Future<Directory> _getCacheDir() async {
    _cacheDir ??= Directory('${(await getTemporaryDirectory()).path}/api_cache');
    if (!await _cacheDir!.exists()) {
      await _cacheDir!.create(recursive: true);
    }
    return _cacheDir!;
  }

  /// 根据请求 URL 生成缓存文件名（MD5 哈希）
  String _getCacheKey(String url) {
    return md5.convert(utf8.encode(url)).toString();
  }

  /// 写入缓存
  Future<void> set(String url, dynamic data, {Duration? ttl}) async {
    final dir = await _getCacheDir();
    final key = _getCacheKey(url);
    final file = File('${dir.path}/$key.json');

    final entry = {
      'data': data,
      'cachedAt': DateTime.now().toIso8601String(),
      'expireAt': DateTime.now()
          .add(ttl ?? const Duration(hours: 1))
          .toIso8601String(),
    };

    await file.writeAsString(json.encode(entry));
  }

  /// 读取缓存
  Future<dynamic> get(String url) async {
    final dir = await _getCacheDir();
    final key = _getCacheKey(url);
    final file = File('${dir.path}/$key.json');

    if (!await file.exists()) return null;

    try {
      final content = await file.readAsString();
      final entry = json.decode(content) as Map<String, dynamic>;

      // 检查过期
      final expireAt = DateTime.parse(entry['expireAt'] as String);
      if (DateTime.now().isAfter(expireAt)) {
        await file.delete(); // 过期，删除缓存文件
        return null;
      }

      return entry['data'];
    } catch (e) {
      // 文件损坏，删除
      await file.delete();
      return null;
    }
  }

  /// 删除指定缓存
  Future<void> remove(String url) async {
    final dir = await _getCacheDir();
    final key = _getCacheKey(url);
    final file = File('${dir.path}/$key.json');
    if (await file.exists()) await file.delete();
  }

  /// 清除所有缓存
  Future<void> clear() async {
    final dir = await _getCacheDir();
    if (await dir.exists()) {
      await dir.delete(recursive: true);
      await dir.create();
    }
  }

  /// 获取缓存总大小
  Future<int> getSize() async {
    final dir = await _getCacheDir();
    if (!await dir.exists()) return 0;
    int totalSize = 0;
    await for (final entity in dir.list(recursive: true)) {
      if (entity is File) {
        totalSize += await entity.length();
      }
    }
    return totalSize;
  }
}
```

#### 11.3 分层缓存 + 网络请求整合

```dart
/// 分层缓存策略 — 内存 → 磁盘 → 网络
/// 支持 stale-while-revalidate: 先返回过期缓存，同时后台刷新
class CachedApiService {
  final Dio _dio = DioConfig.dio;
  final MemoryCache<Map<String, dynamic>> _memoryCache = MemoryCache(
    maxSize: 50,
    defaultTtl: const Duration(minutes: 10),
  );
  final DiskCache _diskCache = DiskCache.instance;

  /// 获取数据 — 三级缓存策略
  Future<Map<String, dynamic>> get(
    String path, {
    Map<String, dynamic>? queryParameters,
    Duration? cacheTtl,
    bool forceRefresh = false,
  }) async {
    final url = _buildUrl(path, queryParameters);

    // 1. 如果强制刷新，跳过缓存
    if (!forceRefresh) {
      // 2. 尝试内存缓存
      final memData = _memoryCache.get(url);
      if (memData != null) {
        print('缓存命中: 内存 ($url)');
        return memData;
      }

      // 3. 尝试磁盘缓存
      final diskData = await _diskCache.get(url);
      if (diskData != null) {
        print('缓存命中: 磁盘 ($url)');
        // 回填内存缓存
        _memoryCache.set(url, diskData as Map<String, dynamic>,
            ttl: cacheTtl);
        return diskData as Map<String, dynamic>;
      }
    }

    // 4. 网络请求
    print('缓存未命中，网络请求: $url');
    final response = await _dio.get(path, queryParameters: queryParameters);
    final data = response.data as Map<String, dynamic>;

    // 5. 写入缓存
    _memoryCache.set(url, data, ttl: cacheTtl);
    await _diskCache.set(url, data, ttl: cacheTtl);

    return data;
  }

  /// stale-while-revalidate 策略
  /// 先返回过期缓存（如果有的话），同时在后台发起网络请求更新
  Future<Map<String, dynamic>> getStaleWhileRevalidate(
    String path, {
    Map<String, dynamic>? queryParameters,
    Duration? cacheTtl,
    void Function(Map<String, dynamic> freshData)? onFreshData,
  }) async {
    final url = _buildUrl(path, queryParameters);

    // 尝试获取缓存（不过期检查，直接返回）
    final memData = _memoryCache.get(url);
    if (memData != null) {
      // 后台刷新
      _refreshInBackground(url, path, queryParameters, onFreshData);
      return memData;
    }

    final diskData = await _diskCache.get(url);
    if (diskData != null) {
      _memoryCache.set(url, diskData as Map<String, dynamic>);
      // 后台刷新
      _refreshInBackground(url, path, queryParameters, onFreshData);
      return diskData as Map<String, dynamic>;
    }

    // 无缓存，必须等待网络
    return await get(path, queryParameters: queryParameters, cacheTtl: cacheTtl);
  }

  /// 后台刷新
  Future<void> _refreshInBackground(
    String url,
    String path,
    Map<String, dynamic>? queryParameters,
    void Function(Map<String, dynamic>)? onFreshData,
  ) async {
    try {
      final response = await _dio.get(path, queryParameters: queryParameters);
      final freshData = response.data as Map<String, dynamic>;

      // 更新缓存
      _memoryCache.set(url, freshData);
      await _diskCache.set(url, freshData);

      // 通知调用方有新数据
      onFreshData?.call(freshData);
    } catch (e) {
      // 后台刷新失败不影响已返回的缓存数据
      print('后台刷新失败: $e');
    }
  }

  /// 清除所有缓存
  Future<void> clearCache() async {
    _memoryCache.clear();
    await _diskCache.clear();
  }

  String _buildUrl(String path, Map<String, dynamic>? params) {
    final uri = Uri.https('api.example.com', path, params?.map(
      (k, v) => MapEntry(k, v.toString()),
    ));
    return uri.toString();
  }
}
```

### 十二、网络状态检测

```dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'dart:async';

/// 网络状态检测服务
class ConnectivityService {
  final Connectivity _connectivity = Connectivity();

  // 网络状态流
  final _connectivityController = StreamController<bool>.broadcast();
  Stream<bool> get isConnected => _connectivityController.stream;

  // 当前状态
  bool _currentStatus = true;
  bool get currentStatus => _currentStatus;

  StreamSubscription? _subscription;

  /// 初始化监听
  void init() {
    _subscription = _connectivity.onConnectivityChanged.listen((results) {
      // connectivity_plus 返回 List<ConnectivityResult>（支持多连接）
      final wasConnected = _currentStatus;
      _currentStatus = results.any((r) => r != ConnectivityResult.none);

      _connectivityController.add(_currentStatus);

      // 状态变化通知
      if (wasConnected && !_currentStatus) {
        _onDisconnected();
      } else if (!wasConnected && _currentStatus) {
        _onReconnected();
      }
    });
  }

  /// 手动检查当前网络状态
  Future<bool> checkConnectivity() async {
    final results = await _connectivity.checkConnectivity();
    _currentStatus = results.any((r) => r != ConnectivityResult.none);
    return _currentStatus;
  }

  /// 获取连接类型
  Future<String> getConnectionType() async {
    final results = await _connectivity.checkConnectivity();
    if (results.contains(ConnectivityResult.wifi)) return 'WiFi';
    if (results.contains(ConnectivityResult.mobile)) return 'Mobile';
    if (results.contains(ConnectivityResult.ethernet)) return 'Ethernet';
    if (results.contains(ConnectivityResult.bluetooth)) return 'Bluetooth';
    if (results.contains(ConnectivityResult.vpn)) return 'VPN';
    return 'None';
  }

  /// 断网回调 — 暂停同步队列、显示提示
  void _onDisconnected() {
    print('网络已断开');
    // 通知 UI 显示离线提示条
    // 暂停网络请求队列
  }

  /// 恢复网络回调 — 恢复同步队列、自动重试
  void _onReconnected() {
    print('网络已恢复');
    // 通知 UI 隐藏离线提示条
    // 恢复同步队列
    // 触发离线期间累积的操作
  }

  void dispose() {
    _subscription?.cancel();
    _connectivityController.close();
  }
}
```

**在 Widget 中使用网络状态：**

```dart
/// 全局网络状态监听 Widget
class ConnectivityWrapper extends StatefulWidget {
  final Widget child;
  const ConnectivityWrapper({super.key, required this.child});

  @override
  State<ConnectivityWrapper> createState() => _ConnectivityWrapperState();
}

class _ConnectivityWrapperState extends State<ConnectivityWrapper> {
  final _connectivityService = ConnectivityService();
  bool _isOffline = false;

  @override
  void initState() {
    super.initState();
    _connectivityService.init();
    _connectivityService.isConnected.listen((connected) {
      setState(() => _isOffline = !connected);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        widget.child,
        // 离线提示条
        if (_isOffline)
          Positioned(
            top: 0,
            left: 0,
            right: 0,
            child: SafeArea(
              child: Container(
                padding: const EdgeInsets.symmetric(vertical: 8, horizontal: 16),
                color: Colors.orange,
                child: const Text(
                  '网络已断开，部分功能不可用',
                  style: TextStyle(color: Colors.white, fontSize: 14),
                  textAlign: TextAlign.center,
                ),
              ),
            ),
          ),
      ],
    );
  }

  @override
  void dispose() {
    _connectivityService.dispose();
    super.dispose();
  }
}
```

### 十三、离线优先架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                    Offline-First Architecture                 │
│                                                             │
│  ┌──────────┐     ┌──────────────────┐    ┌──────────────┐  │
│  │   UI      │────►│  Repository      │───►│  Local DB    │  │
│  │  Layer    │◄────│  (决策中心)       │◄───│  (SQLite)    │  │
│  └──────────┘     └───────┬──────────┘    └──────────────┘  │
│                           │                                  │
│                    ┌──────▼──────────┐                       │
│                    │  Sync Engine    │                       │
│                    │  ┌────────────┐│                       │
│                    │  │ Sync Queue ││  离线时操作入队        │
│                    │  │ (待同步)    ││  在线时自动出队同步     │
│                    │  ├────────────┤│                       │
│                    │  │ Conflict   ││  冲突检测与解决        │
│                    │  │ Resolver   ││                       │
│                    │  ├────────────┤│                       │
│                    │  │ Delta      ││  增量同步，减少数据量   │
│                    │  │ Tracker    ││                       │
│                    │  └────────────┘│                       │
│                    └───────┬────────┘                       │
│                            │                                │
│                    ┌───────▼────────┐                       │
│                    │  Remote API    │                       │
│                    │  (dio + WS)    │                       │
│                    └────────────────┘                       │
│                                                             │
│  关键原则:                                                   │
│  1. 所有读操作优先从本地数据库读取                              │
│  2. 写操作先写本地，再入同步队列                               │
│  3. 网络恢复后自动同步队列                                    │
│  4. 冲突解决策略: 服务端优先 / 客户端优先 / 合并               │
│  5. 同步状态可观测: pending / syncing / synced / conflict    │
└─────────────────────────────────────────────────────────────┘
```

#### 13.1 离线优先 Repository 实现

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

/// 离线优先 Repository — 核心实现
/// 所有数据操作优先走本地数据库，网络恢复后自动同步
class OfflineFirstUserRepository {
  final UserDao _localDao;       // 本地数据库 DAO
  final Dio _dio;                 // 网络客户端
  final SyncQueue _syncQueue;     // 同步队列
  final ConnectivityService _connectivity;

  OfflineFirstUserRepository({
    required UserDao localDao,
    required Dio dio,
    required SyncQueue syncQueue,
    required ConnectivityService connectivity,
  })  : _localDao = localDao,
        _dio = dio,
        _syncQueue = syncQueue,
        _connectivity = connectivity;

  /// 查询用户列表 — 本地优先
  /// 策略: 立即返回本地数据 + 后台拉取最新数据
  Stream<List<User>> watchUsers() {
    // 1. 立即返回本地数据（StreamBuilder 可直接监听）
    final localStream = _localDao.watchAllUsers();

    // 2. 如果在线，后台拉取最新数据并更新本地
    _fetchAndSyncRemote();

    return localStream;
  }

  /// 获取用户详情 — 缓存优先
  Future<User> getUserById(int id) async {
    // 1. 先从本地获取
    var user = await _localDao.getById(id);
    if (user != null) {
      // 2. 后台刷新（stale-while-revalidate）
      _refreshUserInBackground(id);
      return user;
    }

    // 3. 本地无数据，必须网络请求
    if (await _connectivity.checkConnectivity()) {
      final response = await _dio.get('/users/$id');
      user = User.fromJson(response.data);
      await _localDao.insert(user); // 存入本地
      return user;
    }

    throw Exception('无网络且本地无缓存数据');
  }

  /// 创建用户 — 先写本地，再入同步队列
  Future<User> createUser(CreateUserRequest request) async {
    // 1. 先写入本地数据库（标记为 pending 同步状态）
    final localUser = User(
      id: _generateLocalId(), // 临时本地 ID（负数表示未同步）
      name: request.name,
      email: request.email,
      role: request.role,
      createdAt: DateTime.now(),
      updatedAt: DateTime.now(),
      syncStatus: SyncStatus.pendingCreate,
    );
    await _localDao.insert(localUser);

    // 2. 加入同步队列
    _syncQueue.enqueue(SyncOperation(
      type: SyncOperationType.create,
      entityType: 'User',
      localId: localUser.id,
      data: request.toJson(),
    ));

    // 3. 如果在线，立即尝试同步
    if (await _connectivity.checkConnectivity()) {
      _syncQueue.processNext();
    }

    return localUser;
  }

  /// 更新用户 — 先更新本地，再入同步队列
  Future<User> updateUser(int id, Map<String, dynamic> fields) async {
    // 1. 更新本地数据
    final user = await _localDao.getById(id);
    if (user == null) throw Exception('用户不存在');

    final updatedUser = User(
      id: user.id,
      name: fields['name'] as String? ?? user.name,
      email: fields['email'] as String? ?? user.email,
      role: user.role,
      createdAt: user.createdAt,
      updatedAt: DateTime.now(),
      syncStatus: user.syncStatus == SyncStatus.synced
          ? SyncStatus.pendingUpdate
          : user.syncStatus, // 已有未同步操作则不覆盖
    );
    await _localDao.update(updatedUser);

    // 2. 加入同步队列
    _syncQueue.enqueue(SyncOperation(
      type: SyncOperationType.update,
      entityType: 'User',
      localId: id,
      data: fields,
    ));

    return updatedUser;
  }

  /// 删除用户 — 先标记本地，再入同步队列
  Future<void> deleteUser(int id) async {
    // 1. 本地标记为待删除（软删除）
    await _localDao.updateSyncStatus(id, SyncStatus.pendingDelete);

    // 2. 加入同步队列
    _syncQueue.enqueue(SyncOperation(
      type: SyncOperationType.delete,
      entityType: 'User',
      localId: id,
    ));
  }

  /// 后台拉取远程数据并同步到本地
  Future<void> _fetchAndSyncRemote() async {
    if (!await _connectivity.checkConnectivity()) return;

    try {
      final response = await _dio.get('/users', queryParameters: {
        'since': await _getLastSyncTimestamp('users'),
      });

      final remoteUsers = (response.data as List)
          .map((e) => User.fromJson(e))
          .toList();

      // 合并远程数据到本地
      for (final remoteUser in remoteUsers) {
        final localUser = await _localDao.getById(remoteUser.id);
        if (localUser == null) {
          // 本地无此数据，直接插入
          await _localDao.insert(remoteUser);
        } else if (localUser.syncStatus == SyncStatus.synced) {
          // 本地已同步，用远程数据覆盖
          await _localDao.update(remoteUser);
        } else {
          // 本地有未同步的修改，需要冲突解决
          await _resolveConflict(localUser, remoteUser);
        }
      }

      await _updateLastSyncTimestamp('users');
    } catch (e) {
      print('后台同步失败: $e');
    }
  }

  /// 后台刷新单个用户
  Future<void> _refreshUserInBackground(int id) async {
    if (!await _connectivity.checkConnectivity()) return;

    try {
      final response = await _dio.get('/users/$id');
      final remoteUser = User.fromJson(response.data);
      final localUser = await _localDao.getById(id);

      if (localUser?.syncStatus == SyncStatus.synced) {
        await _localDao.update(remoteUser);
      }
    } catch (_) {}
  }

  /// 冲突解决 — 服务端优先策略
  Future<void> _resolveConflict(User local, User remote) async {
    // 简单策略: 服务端优先（Last Write Wins）
    // 复杂策略: 字段级合并、用户选择等
    await _localDao.update(remote.copyWith(
      syncStatus: SyncStatus.synced,
    ));
  }

  // 辅助方法
  int _generateLocalId() => -DateTime.now().millisecondsSinceEpoch;
  Future<String?> _getLastSyncTimestamp(String entity) async => null;
  Future<void> _updateLastSyncTimestamp(String entity) async {}
}

/// 同步状态枚举
enum SyncStatus {
  synced,         // 已同步
  pendingCreate,  // 待创建（本地新建，未上传服务端）
  pendingUpdate,  // 待更新
  pendingDelete,  // 待删除
  conflict,       // 冲突
}
```

#### 13.2 同步队列

```dart
/// 同步队列 — 管理离线期间的待同步操作
class SyncQueue {
  final Dio _dio;
  final AppDatabase _db;
  final ConnectivityService _connectivity;

  // 是否正在处理
  bool _isProcessing = false;
  StreamSubscription? _connectivitySub;

  SyncQueue({
    required Dio dio,
    required AppDatabase db,
    required ConnectivityService connectivity,
  })  : _dio = dio,
        _db = db,
        _connectivity = connectivity {
    _init();
  }

  void _init() {
    // 监听网络恢复事件，自动开始同步
    _connectivitySub = _connectivity.isConnected.listen((connected) {
      if (connected) {
        processAll();
      }
    });
  }

  /// 入队操作
  Future<void> enqueue(SyncOperation operation) async {
    // 保存到本地数据库
    await _saveOperation(operation);
  }

  /// 处理下一个操作
  Future<void> processNext() async {
    if (_isProcessing) return;
    _isProcessing = true;

    try {
      final operation = await _getNextOperation();
      if (operation == null) return;

      await _executeOperation(operation);
      await _markOperationCompleted(operation);
    } catch (e) {
      print('同步操作失败: $e');
      // 根据失败类型决定是否重试
    } finally {
      _isProcessing = false;
    }
  }

  /// 处理所有待同步操作
  Future<void> processAll() async {
    while (await _hasPendingOperations()) {
      await processNext();
    }
  }

  /// 执行同步操作
  Future<void> _executeOperation(SyncOperation op) async {
    switch (op.type) {
      case SyncOperationType.create:
        final response = await _dio.post('/${op.entityType.toLowerCase()}s',
            data: op.data);
        // 更新本地 ID 为服务端 ID
        // ...
        break;
      case SyncOperationType.update:
        await _dio.put('/${op.entityType.toLowerCase()}s/${op.localId}',
            data: op.data);
        break;
      case SyncOperationType.delete:
        await _dio.delete('/${op.entityType.toLowerCase()}s/${op.localId}');
        break;
    }
  }

  // 辅助方法（实际项目使用 sqflite 存储同步队列）
  Future<void> _saveOperation(SyncOperation op) async {}
  Future<SyncOperation?> _getNextOperation() async => null;
  Future<bool> _hasPendingOperations() async => false;
  Future<void> _markOperationCompleted(SyncOperation op) async {}

  void dispose() {
    _connectivitySub?.cancel();
  }
}

/// 同步操作模型
class SyncOperation {
  final SyncOperationType type;
  final String entityType;
  final int localId;
  final Map<String, dynamic>? data;
  final DateTime createdAt;

  SyncOperation({
    required this.type,
    required this.entityType,
    required this.localId,
    this.data,
    DateTime? createdAt,
  }) : createdAt = createdAt ?? DateTime.now();
}

enum SyncOperationType { create, update, delete }
```

### 十四、完整项目集成示例

```dart
/// main.dart — 完整项目初始化
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 1. 初始化持久化层
  await PrefsService.init();
  final db = AppDatabase();

  // 2. 初始化网络层
  final dio = DioConfig.dio;

  // 3. 初始化安全存储
  await SecureStorageService.accessToken; // 触发初始化

  // 4. 初始化网络检测
  final connectivityService = ConnectivityService();
  connectivityService.init();

  // 5. 初始化缓存
  final cachedApi = CachedApiService();

  // 6. 初始化同步队列
  final syncQueue = SyncQueue(
    dio: dio,
    db: db,
    connectivity: connectivityService,
  );

  runApp(MyApp(
    db: db,
    dio: dio,
    connectivityService: connectivityService,
    cachedApi: cachedApi,
    syncQueue: syncQueue,
  ));
}
```

**依赖注入（Riverpod 示例）：**

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

/// ============ Provider 定义 ============

/// Dio 实例
final dioProvider = Provider<Dio>((ref) => DioConfig.dio);

/// 数据库实例
final dbProvider = Provider<AppDatabase>((ref) => AppDatabase());

/// 网络状态
final connectivityProvider = Provider<ConnectivityService>((ref) {
  final service = ConnectivityService();
  service.init();
  ref.onDispose(() => service.dispose());
  return service;
});

/// 是否在线
final isOnlineProvider = StreamProvider<bool>((ref) {
  return ref.watch(connectivityProvider).isConnected;
});

/// 缓存 API 服务
final cachedApiProvider = Provider<CachedApiService>((ref) {
  return CachedApiService();
});

/// 用户 Repository
final userRepositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository();
});

/// 离线优先用户 Repository
final offlineUserRepoProvider = Provider<OfflineFirstUserRepository>((ref) {
  return OfflineFirstUserRepository(
    localDao: UserDao(),
    dio: ref.watch(dioProvider),
    syncQueue: ref.watch(syncQueueProvider),
    connectivity: ref.watch(connectivityProvider),
  );
});

final syncQueueProvider = Provider<SyncQueue>((ref) {
  return SyncQueue(
    dio: ref.watch(dioProvider),
    db: ref.watch(dbProvider),
    connectivity: ref.watch(connectivityProvider),
  );
});

/// ============ 在 Widget 中使用 ============

class UserListPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isOnline = ref.watch(isOnlineProvider);
    final usersAsync = ref.watch(userListProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('用户列表'),
        // 网络状态指示
        actions: [
          Icon(
            isOnline.valueOrNull == true ? Icons.cloud_done : Icons.cloud_off,
            color: isOnline.valueOrNull == true ? Colors.green : Colors.grey,
          ),
          const SizedBox(width: 16),
        ],
      ),
      body: usersAsync.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, _) => Center(child: Text('加载失败: $e')),
        data: (users) => ListView.builder(
          itemCount: users.length,
          itemBuilder: (context, index) {
            final user = users[index];
            return ListTile(
              title: Text(user.name),
              subtitle: Text(user.email),
              trailing: _buildSyncIcon(user.syncStatus),
            );
          },
        ),
      ),
    );
  }

  Widget _buildSyncIcon(SyncStatus? status) {
    switch (status) {
      case SyncStatus.synced:
        return const Icon(Icons.check_circle, color: Colors.green, size: 16);
      case SyncStatus.pendingCreate:
      case SyncStatus.pendingUpdate:
      case SyncStatus.pendingDelete:
        return const Icon(Icons.sync, color: Colors.orange, size: 16);
      case SyncStatus.conflict:
        return const Icon(Icons.warning, color: Colors.red, size: 16);
      default:
        return const SizedBox.shrink();
    }
  }
}
```

### 方案对比速查

| 场景 | 推荐方案 | 备选方案 |
|------|----------|----------|
| 简单 GET/POST | http 包 | dio |
| 生产级 API 调用 | dio + 拦截器封装 | - |
| Token 认证 | AuthInterceptor + flutter_secure_storage | - |
| 文件上传 | dio FormData + 进度回调 | http MultipartRequest |
| 大文件下载 | dio download + 断点续传 | - |
| 实时通信 | web_socket_channel + 自动重连 | Socket.io (第三方) |
| JSON 序列化 | json_serializable + build_runner | freezed (不可变模型) |
| 配置/偏好 | SharedPreferences | - |
| 结构化数据 | drift (类型安全 ORM) | sqflite (原生 SQL) |
| 敏感数据 | flutter_secure_storage | - |
| 离线优先 | 本地 DB + 同步队列 + connectivity | Firebase Offline |
| 缓存策略 | 内存缓存 + 磁盘缓存分层 | dio_cache_interceptor |

**相关链接**：[[Flutter核心与Widget体系]] | [[Flutter状态管理]] | [[HTTP与缓存策略]] | [[Cookie与认证]]

---

## 面试题

### 1. dio 的拦截器执行顺序是什么？多个拦截器如何协调？

**答案：**

dio 拦截器按添加顺序组成一条链，请求阶段（`onRequest`）按添加顺序执行，响应阶段（`onResponse`）和错误阶段（`onError`）按逆序执行。

```
请求方向: Request → Interceptor1.onRequest → Interceptor2.onRequest → ... → HTTP Server
响应方向: Response ← Interceptor1.onResponse ← Interceptor2.onResponse ← ... ← HTTP Server
错误方向: Error ← Interceptor1.onError ← Interceptor2.onError ← ... ← HTTP Server
```

举例：添加顺序为 `[LogInterceptor, AuthInterceptor, RetryInterceptor]`

- 请求发出：Log 记录 → Auth 加 Token → Retry 准备
- 响应返回：Retry 处理 → Auth 检查 → Log 记录
- 发生错误：Retry 判断是否重试 → Auth 判断是否 401 刷新 → Log 记录错误

**注意事项：**
- `handler.next()` 传递给下一个拦截器
- `handler.resolve()` 直接返回响应，跳过后续拦截器
- `handler.reject()` 直接返回错误，跳过后续拦截器
- Token 刷新拦截器应放在错误处理拦截器之前，这样 401 刷新成功后可以直接 resolve，不会被错误拦截器误处理
- 日志拦截器应放在最外层（第一个添加），确保记录所有请求和响应

### 2. 如何实现 Token 过期自动刷新？并发请求时如何避免重复刷新？

**答案：**

Token 自动刷新的核心逻辑在 AuthInterceptor 的 `onError` 回调中实现：

1. 检测 401 状态码
2. 使用 `_isRefreshing` 标志位加锁，防止并发刷新
3. 其他并发请求通过 `Completer` 队列等待
4. 刷新成功后通知所有等待的请求，用新 Token 重试

```
并发请求场景:
Request1 → 401 → 开始刷新 (锁定)
Request2 → 401 → 已在刷新，进入等待队列
Request3 → 401 → 已在刷新，进入等待队列
              ↓
         刷新成功，获得新 Token
              ↓
Request1 → 新 Token 重试 → 成功
Request2 → 新 Token 重试 → 成功
Request3 → 新 Token 重试 → 成功
```

**关键实现要点：**

- 刷新 Token 的请求必须使用独立的 Dio 实例，避免触发当前拦截器的死循环
- 刷新失败时需要清除 Token 并跳转登录页
- 使用 `Completer` 实现等待/通知机制
- 刷新请求本身不需要再次刷新（加入白名单）
- 在 `onRequest` 中自动附加 Token，减少每个 API 调用都手动传 Token 的样板代码

### 3. json_serializable 相比手写 fromJson/toJson 有什么优势？什么场景下应该用手写？

**答案：**

**json_serializable 优势：**

1. **编译期类型安全**：生成的代码保证类型正确，手写容易在类型转换处出错（如 `json['id'] as int` 遗漏）
2. **减少样板代码**：模型字段多时手写 `fromJson`/`toJson` 极其冗长，生成器自动处理
3. **注解功能丰富**：`@JsonKey(name:)` 处理字段名映射、`defaultValue` 设默认值、`unknownEnumValue` 处理未知枚举
4. **自动同步**：修改模型字段后重新生成即可，手写容易遗漏某个字段
5. **嵌套对象**：`explicitToJson: true` 自动处理嵌套序列化

**手写适用场景：**

1. **模型极少**（1-3 个简单模型），引入 build_runner 代码生成成本不划算
2. **复杂自定义转换逻辑**：某些字段需要根据多个字段联合判断，注解无法表达
3. **性能极端优化**：代码生成会引入 `_$xxxFromJson` 等间接调用，极高频场景可能有微小开销
4. **无法使用代码生成**：某些构建环境限制下无法运行 build_runner
5. **快速原型/示例代码**：不需要长期维护的一次性代码

**最佳实践**：生产项目推荐 json_serializable，搭配 `freezed` 实现不可变模型 + copyWith + union 类型。

### 4. SharedPreferences、SQLite、flutter_secure_storage 分别适合存储什么数据？能否混合使用？

**答案：**

| 存储方案 | 适合存储 | 不适合存储 |
|----------|----------|------------|
| SharedPreferences | 主题模式、语言偏好、字体大小、开关状态、首次启动标记 | 大量数据、结构化数据、敏感信息 |
| SQLite | 聊天记录、文章列表、离线缓存、结构化业务数据 | 简单配置项（杀鸡用牛刀）、敏感信息（明文） |
| flutter_secure_storage | Access Token、Refresh Token、加密密钥、用户密码、生物识别密钥 | 大量数据（加密开销大）、非敏感数据 |

**混合使用最佳实践：**

```
用户登录:
  → SecureStorage: accessToken, refreshToken
  → SharedPreferences: isFirstLaunch, themeMode, locale
  → SQLite: 本地缓存的数据（用户资料、消息等）
```

混合使用时需要注意：
1. **数据一致性**：登出时需要同时清除三者的数据
2. **读取顺序**：先读 SecureStorage 的 Token，再决定是否请求网络数据写入 SQLite
3. **降级策略**：flutter_secure_storage 在 Web 端不支持，需要降级到 SharedPreferences + 加密
4. **封装统一接口**：通过 Repository 模式隐藏底层存储差异，UI 层不关心数据来源

### 5. 如何实现文件的断点续传下载？核心原理是什么？

**答案：**

**核心原理：** HTTP Range 请求头 + 本地文件追加写入

1. **检查本地文件**：下载前检查目标路径是否已有部分文件，获取已下载字节数
2. **设置 Range 头**：`Range: bytes=已下载字节数-`，告诉服务端从断点处开始传输
3. **追加写入**：以 `FileMode.append` 模式打开文件，新数据追加到已有数据后面
4. **服务端支持**：服务端需支持 Range 请求，返回 `206 Partial Content` 状态码和 `Content-Range` 头

**实现要点：**

- 服务端必须返回 `Accept-Ranges: bytes` 和 `Content-Length`（总文件大小）
- 下载中断后重新下载时，先检查本地文件大小，作为 Range 起始偏移
- 用流式接收（`ResponseType.stream`）+ `openWrite(append: true)` 避免内存溢出
- 大文件建议分片下载 + 合并，避免单次下载时间过长
- 记录下载元信息（URL、文件大小、已下载字节数、ETag）到数据库，便于恢复
- 如果服务端文件已更新（ETag 变化），需要重新下载而非续传
- 分片上传同理：先分片，逐片上传，最后通知服务端合并

### 6. WebSocket 如何保证连接稳定性？心跳和重连机制如何设计？

**答案：**

**心跳机制：**

1. 客户端定时（如 30 秒）发送 `ping` 消息
2. 服务端收到后回复 `pong`
3. 如果连续 N 次心跳无响应（如 3 次），判定连接已断开，触发重连
4. 收到任何消息时重置心跳计时器（收到消息本身证明连接存活）

**重连机制：**

1. **递增延迟**：1s → 2s → 4s → 8s → 16s，避免服务端压力
2. **最大重连次数**：达到上限后停止重连，提示用户手动重连
3. **网络状态联动**：监听 connectivity_plus，网络恢复时立即重连
4. **手动/自动区分**：用户主动断开时不重连，异常断开时自动重连
5. **重连成功后**：重新发送认证信息、重新订阅频道、拉取离线消息

```
断开 → 延迟1s重连(第1次) → 失败
     → 延迟2s重连(第2次) → 失败
     → 延迟4s重连(第3次) → 失败
     → ... 达到最大次数 → 通知UI
     → 网络恢复事件 → 立即重连 → 成功 → 恢复订阅
```

**补充要点：**
- 使用 `web_socket_channel` 的 `stream.listen` 监听 `onDone` 和 `onError`
- 心跳包尽量短（如 `ping`/`pong` 文本），减少带宽占用
- 避免在心跳间隔过短的情况下频繁重连

### 7. 离线优先架构的核心设计原则是什么？如何处理数据冲突？

**答案：**

**核心原则：**

1. **本地优先**：所有读操作先查本地数据库，保证 UI 响应速度
2. **乐观更新**：写操作先写入本地并立即更新 UI，不阻塞用户操作
3. **同步队列**：所有待同步操作入队持久化，网络恢复后自动处理
4. **增量同步**：只同步变化的数据（通过 timestamp 或 version 字段），减少传输量
5. **可观测同步状态**：UI 层展示数据的同步状态（已同步/待同步/冲突）

**冲突解决策略：**

| 策略 | 规则 | 适用场景 |
|------|------|----------|
| 服务端优先 (Server Wins) | 远程数据覆盖本地 | 大多数场景，保证数据权威性 |
| 客户端优先 (Client Wins) | 本地数据覆盖远程 | 用户编辑类场景 |
| Last Write Wins | 比较 updatedAt，新的覆盖旧的 | 简单场景 |
| 字段级合并 | 非冲突字段各取最新 | 协同编辑场景 |
| 用户选择 | 弹窗让用户选择保留哪个 | 重要数据，不可自动解决 |

**实现建议：**

- 每条数据记录 `syncStatus` 字段和 `version`/`updatedAt` 字段
- 同步时使用 `since` 参数只拉取上次同步后的变更
- 冲突无法自动解决时标记为 `SyncStatus.conflict`，通知用户手动处理
- 临时本地 ID 使用负数，同步成功后替换为服务端 ID
- 同步队列本身也需要持久化（存 SQLite），App 重启后可继续未完成的同步

### 8. Flutter 中如何检测网络状态变化？如何在 UI 层优雅地处理离线/在线切换？

**答案：**

**网络检测：**

使用 `connectivity_plus` 包，它提供两种 API：

1. **一次性检查**：`Connectivity().checkConnectivity()` 返回当前连接类型
2. **持续监听**：`Connectivity().onConnectivityChanged` 返回 `Stream<List<ConnectivityResult>>`

**监听返回的结果类型：**

- `ConnectivityResult.wifi` — WiFi 连接
- `ConnectivityResult.mobile` — 移动网络
- `ConnectivityResult.ethernet` — 以太网
- `ConnectivityResult.bluetooth` — 蓝牙
- `ConnectivityResult.vpn` — VPN
- `ConnectivityResult.none` — 无网络

**UI 层处理最佳实践：**

1. **全局监听 + 全局提示**：在 App 顶层用 `ConnectivityWrapper` 包裹，离线时显示顶部提示条
2. **Provider/状态管理集成**：将网络状态注入状态管理系统，任何 Widget 都可响应
3. **离线 UI 降级**：网络不可用时隐藏/禁用需要网络的功能按钮，显示灰色占位
4. **缓存数据展示**：离线时仍展示本地缓存数据，标注"数据可能不是最新"
5. **操作排队反馈**：离线时用户操作入同步队列，UI 显示"将在网络恢复后同步"
6. **避免阻塞**：不要在网络断开时弹出阻断式对话框，使用非侵入式提示条

```dart
// 典型 UI 处理模式
isOnline ? NetworkContent() : CachedContent();
isOnline ? ActiveButton() : DisabledButton(hint: '需要网络');
syncStatus == SyncStatus.pending ? SyncIndicator() : SizedBox();
```

**注意：** `connectivity_plus` 只检测网络连接状态，不保证互联网可达。连接 WiFi 但无法上网时仍返回 `wifi`。如需检测真实可达性，需额外 ping 或请求一个轻量接口。

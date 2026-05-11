---
tags:
  - Web前端
  - Flutter
  - Dart
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Dart语言核心

## What — 是什么

> Dart 是 Google 设计的面向对象编程语言，是 Flutter 的官方开发语言。它同时支持 AOT（Ahead-Of-Time）编译为高效原生代码和 JIT（Just-In-Time）解释执行（支持 Hot Reload），具备空安全、强类型、单线程异步模型和 Isolate 并发机制。

**核心概念：**

- **强类型 + 类型推断**：变量有明确类型，但可用 var/动态类型简化书写
- **空安全（Null Safety）**：类型默认不可为 null，需显式标记可空类型
- **一切皆对象**：int、double、bool、null 都是 Object 的子类
- **单线程事件循环**：通过 Event Loop + Isolate 实现并发
- **AOT + JIT 双模式**：开发时 JIT（Hot Reload），发布时 AOT（高性能）

**关键特性：**

- 健全的空安全系统（Sound Null Safety）
- 基于类的单继承 + Mixin 多复用
- 泛型与类型约束
- 扩展方法（Extension Methods）
- Dart 3 新特性：Records、Sealed Classes、Pattern Matching
- 异步编程：Future/Stream/async-await/Isolate
- 统一的集合与函数式操作

**运行机制：**

- **内存模型**：分代垃圾回收（新生代 Scavenge + 老年代 Mark-Sweep），类似 V8
- **执行模型**：单线程事件循环（Event Loop），微任务队列（MicroTask）优先于事件队列（Event）
- **并发模型**：Isolate 隔离并发，每个 Isolate 有独立堆内存，通过端口（Port）消息通信

```
┌──────────────────────────────────────┐
│            Dart Isolate              │
├──────────────────────────────────────┤
│         Event Loop                   │
│  ┌──────────────────────────────┐    │
│  │  MicroTask Queue (微任务)     │    │
│  │  scheduleMicrotask()         │    │
│  │  优先级最高，插队执行          │    │
│  └──────────────────────────────┘    │
│  ┌──────────────────────────────┐    │
│  │  Event Queue (事件队列)       │    │
│  │  IO / Timer / Future / ...   │    │
│  │  按序执行                     │    │
│  └──────────────────────────────┘    │
│                                      │
│  Heap (堆内存，GC 管理)               │
│  ┌──────┬──────┬──────┬───────┐     │
│  │ Young│ Old  │ Large│ Symbol │     │
│  │ Gen  │ Gen  │ Obj  │ Table │     │
│  └──────┴──────┴──────┴───────┘     │
└──────────────────────────────────────┘
```

**类型系统：**

- 类型分类：基本类型（int/double/bool/String/Null）、集合类型（List/Set/Map）、函数类型（Function）、特殊类型（dynamic/Object/void/Never）
- 类型转换规则：子类可隐式转为父类，父类转子类需 as 显式转换
- 泛型支持：协变（covariant）、泛型约束（extends）

## Why — 为什么

**适用场景：**

- Flutter 跨平台应用开发
- 服务端开发（Dart 的 shelf / dart_frog 框架）
- 命令行工具开发
- Web 前端开发（dart2js 编译为 JavaScript）

**对比其他语言：**

| 维度 | Dart | TypeScript | Kotlin | Swift |
|------|------|-----------|--------|-------|
| 类型系统 | 强类型+空安全 | 结构化类型 | 强类型+空安全 | 强类型+空安全 |
| 编译方式 | AOT + JIT | 转译为JS | AOT + JIT | AOT |
| 并发模型 | Isolate(共享无) | 单线程/Web Worker | 协程 | Actor/GCD |
| 空安全 | 健全(Sound) | 严格模式 | 健全 | 健全 |
| 面向对象 | 单继承+Mixin | 原型链 | 单继承+接口 | 单继承+协议 |
| 生态 | pub.dev | npm | Maven | SPM/CocoaPods |
| 学习曲线 | 中 | 低(前端友好) | 中高 | 中高 |
| 热重载 | 支持(Flutter) | 支持(FW/HMR) | 不支持 | 不支持 |

**优缺点：**

- ✅ 优点：
  - 空安全系统健全，编译期消除空指针异常
  - AOT 编译性能接近原生，JIT 支持 Hot Reload
  - 语法简洁，类似 JavaScript/Kotlin，上手快
  - 内置 async-await，异步编程模型清晰
  - Mixin 机制灵活，避免多重继承的菱形问题
  - Dart 3 引入 Pattern Matching，表达能力大幅提升
- ❌ 缺点：
  - 生态不如 JavaScript/Kotlin 成熟
  - Isolate 并发模型不如协程/Actor 直觉
  - 服务端生态较弱
  - 社区规模相对较小

## How — 怎么用

### 1. 变量与类型系统

```dart
// ===== 变量声明 =====
String name = 'Flutter';          // 显式类型
var count = 10;                   // 类型推断为 int
final String appName = 'MyApp';   // 运行时常量，只能赋值一次
const double pi = 3.14159;        // 编译时常量
dynamic anything = 'hello';       // 动态类型，绕过类型检查
Object obj = 42;                  // 顶层类型，有类型检查

// final vs const
final now = DateTime.now();       // ✅ 运行时赋值
// const time = DateTime.now();   // ❌ 编译时常量不能赋运行时值
const list = [1, 2, 3];           // const 列表，内容不可变
const map = {'a': 1, 'b': 2};    // const 映射

// ===== 基本类型 =====
int a = 42;                       // 整数，64位
double b = 3.14;                  // 浮点数，64位
bool flag = true;                 // 布尔
String s = 'Hello';               // 字符串，UTF-16
String raw = r'C:\Users\name';    // 原始字符串，不转义
String multi = '''
  多行
  字符串
''';

// 字符串操作
var greeting = 'Hello, $name!';              // 字符串插值
var length = 'Length: ${name.length}';       // 表达式插值
var concat = 'Hello' ' ' 'World';            // 相邻字符串自动拼接

// 数值操作
int x = 5;
assert(x.isEven);                            // 是否偶数
assert(x.isOdd);                             // 是否奇数
assert(3.14.round() == 3);                   // 四舍五入
assert(3.14.toInt() == 3);                   // 转整数
assert((-5).abs() == 5);                     // 绝对值

// 类型转换
String intStr = '42';
int parsed = int.parse(intStr);              // String → int
String strVal = 42.toString();               // int → String
double d = 3.14;
int i = d.toInt();                           // double → int（截断）
```

### 2. 空安全（Null Safety）

```dart
// ===== 可空类型 =====
String nonNullable = 'hello';      // 不可为 null
String? nullable = null;           // 可为 null（? 后缀）
int? maybeNull;                    // 默认为 null

// ===== 空安全操作符 =====
// ?. 空安全访问
String? name;
int? len = name?.length;           // name 为 null 时返回 null

// ?? 空值合并
String displayName = name ?? 'Guest';  // name 为 null 时用 'Guest'

// ! 非空断言（确定不为null时使用，否则抛异常）
int length = name!.length;         // 如果 name 为 null 则抛异常

// late 延迟初始化
late String description;           // 声明时不初始化，但保证使用前会赋值
// 适合场景：构造函数后赋值、initState 中赋值

class Person {
  late final String name;          // late + final：只能赋值一次
  Person(this.name);
}

// ===== 空安全最佳实践 =====
// 1. 优先使用非空类型
// 2. 可能为 null 时用 ? 标记
// 3. 用 ?? 提供默认值而非 !
// 4. late 只在确定使用前会初始化时使用
// 5. 避免滥用 ! 非空断言

// 条件判断后自动类型提升
void printLength(String? str) {
  if (str != null) {
    // 这里 str 自动提升为 String（非空）
    print(str.length);  // 不需要 ! 或 ?.
  }
}
```

### 3. 函数

```dart
// ===== 函数定义 =====
// 标准函数
int add(int a, int b) {
  return a + b;
}

// 箭头函数（单表达式）
int multiply(int a, int b) => a * b;

// void 返回
void log(String msg) => print('[LOG] $msg');

// ===== 可选参数 =====
// 命名可选参数（用 {} 包裹）
void createUser({required String name, int age = 18, String? email}) {
  print('Name: $name, Age: $age, Email: $email');
}
createUser(name: 'Alice', email: 'a@b.com');  // age 使用默认值

// 位置可选参数（用 [] 包裹）
String greet(String name, [String? title]) {
  return title != null ? '$title $name' : name;
}
greet('Bob');               // 'Bob'
greet('Bob', 'Dr.');        // 'Dr. Bob'

// ===== 命名参数必填（required） =====
void configure({required String host, required int port}) {
  print('Connecting to $host:$port');
}
configure(host: 'localhost', port: 8080);

// ===== 闭包 =====
Function makeAdder(int addBy) {
  return (int i) => i + addBy;    // 闭包捕获 addBy
}
var add2 = makeAdder(2);
print(add2(3));   // 5

// ===== 匿名函数 =====
var items = [1, 2, 3];
items.forEach((item) {
  print(item);
});
// 简写
items.forEach((item) => print(item));

// ===== 函数类型 =====
typedef Operation = int Function(int, int);
Operation op = (a, b) => a + b;
print(op(3, 4));  // 7

// ===== 函数作为参数 =====
void performOperation(int a, int b, int Function(int, int) operation) {
  print(operation(a, b));
}
performOperation(10, 5, (a, b) => a * b);  // 50
```

### 4. 类与面向对象

```dart
// ===== 基础类 =====
class User {
  final String name;
  final int age;

  // 构造函数
  User(this.name, this.age);

  // 命名构造函数
  User.guest() : name = 'Guest', age = 0;

  // 命名构造函数 + 初始化列表
  User.fromJson(Map<String, dynamic> json)
      : name = json['name'] as String,
        age = json['age'] as int;

  // 工厂构造函数（不总是创建新实例）
  factory User.create(String name, int age) {
    if (name.isEmpty) return User.guest();
    return User(name, age);
  }

  // Getter / Setter
  String get displayName => age >= 18 ? '$name (Adult)' : '$name (Minor)';

  // 方法
  String greet() => 'Hi, I am $name, $age years old.';

  @override
  String toString() => 'User($name, $age)';
}

// ===== 继承 =====
class Admin extends User {
  final List<String> permissions;

  Admin(String name, int age, this.permissions) : super(name, age);

  @override
  String greet() => '${super.greet()} I am an admin.';
}

// ===== 抽象类 =====
abstract class Shape {
  double area();                    // 抽象方法
  void describe() {                 // 具体方法
    print('Area: ${area()}');
  }
}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);

  @override
  double area() => 3.14159 * radius * radius;
}

// ===== 接口（Dart 没有interface关键字，类本身就是接口）=====
// implements 要求实现所有方法
class Square implements Shape {
  final double side;
  Square(this.side);

  @override
  double area() => side * side;

  @override
  void describe() => print('Square with side $side');
}

// ===== Mixin =====
mixin Loggable {
  void log(String msg) => print('[${runtimeType}] $msg');
}

mixin Serializable {
  Map<String, dynamic> toJson();
}

class Product with Loggable, Serializable {
  final String name;
  final double price;

  Product(this.name, this.price);

  @override
  Map<String, dynamic> toJson() => {'name': name, 'price': price};

  void save() {
    log('Saving product: $name');   // 使用 Mixin 的方法
    // 保存逻辑...
  }
}

// Mixin 限定使用范围（on 关键字）
mixin Validate on Shape {
  bool isValid() => area() > 0;
}

class ValidCircle extends Circle with Validate {
  ValidCircle(double radius) : super(radius);
}

// ===== 构造函数进阶 =====
class Point {
  final double x, y;

  // 常量构造函数（所有字段必须 final）
  const Point(this.x, this.y);

  // 重定向构造函数
  Point.origin() : this(0, 0);

  // 初始化列表
  Point.fromAngle(double angle, double distance)
      : x = distance * cos(angle),
        y = distance * sin(angle);

  static double cos(double a) => a; // 简化示例
  static double sin(double a) => a;
}

// const 构造函数创建编译时常量
const p1 = Point(1, 2);
const p2 = Point(1, 2);
print(identical(p1, p2));  // true，同一个对象
```

### 5. 泛型与类型约束

```dart
// ===== 泛型类 =====
class Box<T> {
  final T value;
  Box(this.value);

  T unwrap() => value;

  // 泛型方法
  R map<R>(R Function(T) transform) => transform(value);
}

var intBox = Box<int>(42);
var strBox = Box<String>('hello');

// ===== 泛型函数 =====
T first<T>(List<T> items) => items.first;

// ===== 泛型约束 =====
class SortedList<T extends Comparable<T>> {
  final List<T> _items = [];

  void add(T item) {
    _items.add(item);
    _items.sort();
  }
}

// ===== 泛型协变 =====
// Dart 泛型默认是不变的（invariant）
// List<int> 不是 List<num> 的子类型
List<int> ints = [1, 2, 3];
// List<num> nums = ints;  // ❌ 编译错误

// ===== 常用泛型工具 =====
// cast 转换类型
List<num> nums = [1, 2, 3.0];
List<int> ints = nums.cast<int>();  // 运行时检查

// whereType 过滤类型
List<num> mixed = [1, 2.5, 3, 4.0];
List<int> onlyInts = mixed.whereType<int>().toList();  // [1, 3]
```

### 6. 集合与函数式操作

```dart
// ===== 集合类型 =====
// List（有序可重复）
List<int> numbers = [1, 2, 3, 4, 5];
var growable = <int>[];             // 可增长列表
var fixed = List<int>.filled(5, 0); // 固定长度列表

// Set（无序不重复）
Set<String> tags = {'dart', 'flutter', 'web'};
var emptySet = <String>{};

// Map（键值对）
Map<String, int> ages = {'Alice': 25, 'Bob': 30};
var emptyMap = <String, int>{};

// ===== 集合操作 =====
// 展开（spread operator）
var list1 = [1, 2, 3];
var list2 = [0, ...list1, 4];     // [0, 1, 2, 3, 4]
var maybeNull = <int>?[5, 6];
var list3 = [0, ...?maybeNull];   // 空安全展开

// 集合 if/for
var isDebug = true;
var config = [
  'production',
  if (isDebug) 'debug',           // 条件包含
  for (var i = 0; i < 3; i++) 'item$i',  // 循环生成
];

// ===== 函数式操作 =====
var numbers = [1, 2, 3, 4, 5];

// map — 映射
var doubled = numbers.map((n) => n * 2).toList();     // [2, 4, 6, 8, 10]

// where — 过滤
var evens = numbers.where((n) => n.isEven).toList();  // [2, 4]

// reduce — 归约（必须至少1个元素）
var sum = numbers.reduce((a, b) => a + b);            // 15

// fold — 归约（带初始值，空集合安全）
var product = numbers.fold<int>(1, (a, b) => a * b);  // 120

// expand — 展平
var nested = [[1, 2], [3, 4], [5]];
var flat = nested.expand((x) => x).toList();          // [1, 2, 3, 4, 5]

// every / any
var allPositive = numbers.every((n) => n > 0);        // true
var hasEven = numbers.any((n) => n.isEven);            // true

// sort
var sorted = [3, 1, 4, 1, 5]..sort();                  // [1, 1, 3, 4, 5]
var descSorted = [3, 1, 4, 1, 5]..sort((a, b) => b.compareTo(a));  // [5, 4, 3, 1, 1]

// take / skip
var first3 = numbers.take(3).toList();                 // [1, 2, 3]
var skip2 = numbers.skip(2).toList();                  // [3, 4, 5]

// forEach
numbers.forEach((n) => print(n));

// Map 操作
var scores = {'Alice': 95, 'Bob': 82, 'Charlie': 90};
var highScores = scores.entries
    .where((e) => e.value >= 90)
    .map((e) => e.key)
    .toList();                                          // ['Alice', 'Charlie']
```

### 7. 异步编程

```dart
// ===== Future =====
Future<String> fetchUserName() async {
  await Future.delayed(const Duration(seconds: 1));
  return 'Alice';
}

// 使用
void main() async {
  var name = await fetchUserName();     // 等待结果
  print(name);                          // Alice
}

// Future 链式调用
fetchUserName()
    .then((name) => name.toUpperCase())
    .then((upper) => print(upper))
    .catchError((e) => print('Error: $e'));

// Future 组合
var results = await Future.wait([
  fetchUserName(),
  fetchUserEmail(),
  fetchUserAge(),
]);

// ===== async-await =====
Future<List<User>> loadUsers() async {
  try {
    final response = await httpGet('/api/users');
    final List data = response['data'];
    return data.map((json) => User.fromJson(json)).toList();
  } catch (e) {
    print('Failed to load users: $e');
    rethrow;
  }
}

// ===== Stream =====
// Stream 是异步事件序列，类似 RxJS 的 Observable
Stream<int> countStream(int max) async* {
  for (var i = 1; i <= max; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;                              // 产出值
  }
}

// 监听 Stream
void listenStream() {
  final stream = countStream(5);
  stream.listen(
    (data) => print('Data: $data'),
    onError: (error) => print('Error: $error'),
    onDone: () => print('Done!'),
  );
}

// StreamBuilder 在 Flutter 中使用
// StreamBuilder<int>(
//   stream: countStream(10),
//   builder: (context, snapshot) {
//     if (snapshot.hasError) return Text('Error');
//     if (!snapshot.hasData) return CircularProgressIndicator();
//     return Text('Count: ${snapshot.data}');
//   },
// );

// Stream 操作
var filtered = countStream(10).where((n) => n.isEven);
var mapped = countStream(5).map((n) => 'Item $n');

// StreamController（手动控制流）
import 'dart:async';

class EventBus {
  final _controller = StreamController<String>.broadcast();

  Stream<String> get stream => _controller.stream;
  void emit(String event) => _controller.add(event);
  void dispose() => _controller.close();
}

// ===== Completer =====
// 手动完成一个 Future
Future<String> manualFuture() {
  var completer = Completer<String>();
  // 在某个异步回调中完成
  someAsyncOperation((result) {
    completer.complete(result);
  }, onError: (e) {
    completer.completeError(e);
  });
  return completer.future;
}
```

### 8. Isolate 与并发

```dart
// ===== Isolate 基础 =====
// Isolate 是 Dart 的并发单元，每个 Isolate 有独立内存堆
// Isolate 之间不共享内存，通过 SendPort/ReceivePort 通信

import 'dart:isolate';

// 在新 Isolate 中执行耗时计算
Future<int> heavyComputation(int input) async {
  final receivePort = ReceivePort();
  await Isolate.spawn(
    _isolateEntry,
    _IsolateMessage(input, receivePort.sendPort),
  );
  return receivePort.first as int;
}

void _isolateEntry(_IsolateMessage message) {
  var result = 0;
  for (var i = 0; i < message.input; i++) {
    result += i;  // 耗时计算
  }
  message.sendPort.send(result);
}

class _IsolateMessage {
  final int input;
  final SendPort sendPort;
  _IsolateMessage(this.input, this.sendPort);
}

// ===== compute() 简化 Isolate =====
import 'package:flutter/foundation.dart';

Future<int> calculateSum(int count) async {
  return compute(_sumInRange, count);
}

int _sumInRange(int count) {
  var sum = 0;
  for (var i = 0; i < count; i++) {
    sum += i;
  }
  return sum;
}

// 使用
var result = await calculateSum(100000000);

// ===== Isolate 通信模式 =====
// 双向通信
Future<void> bidirectionalCommunication() async {
  final receivePort = ReceivePort();
  final isolate = await Isolate.spawn(
    _worker,
    receivePort.sendPort,
  );

  // 接收 worker 的 SendPort
  final workerSendPort = await receivePort.first as SendPort;

  // 发送任务给 worker
  final responsePort = ReceivePort();
  workerSendPort.send(['task', 42, responsePort.sendPort]);

  // 接收结果
  final result = await responsePort.first;
  print('Result: $result');

  isolate.kill(priority: Isolate.immediate);
  receivePort.close();
}

void _worker(SendPort mainSendPort) {
  final receivePort = ReceivePort();
  mainSendPort.send(receivePort.sendPort);

  receivePort.listen((message) {
    if (message is List && message[0] == 'task') {
      var input = message[1] as int;
      var replyPort = message[2] as SendPort;
      replyPort.send(input * 2);
    }
  });
}
```

### 9. 扩展方法（Extension Methods）

```dart
// ===== 基础扩展 =====
extension StringX on String {
  String get capitalized =>
      isEmpty ? '' : '${this[0].toUpperCase()}${substring(1)}';

  bool get isEmail => RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);

  String reverse() => split('').reversed.join();
}

print('hello'.capitalized);    // Hello
print('a@b.com'.isEmail);      // true
print('dart'.reverse());        // trad

// ===== 扩展运算符 =====
extension ListX<T> on List<T> {
  T? get firstOrNull => isEmpty ? null : first;
  T? get lastOrNull => isEmpty ? null : last;

  List<T> separated(T separator) {
    var result = <T>[];
    for (var i = 0; i < length; i++) {
      if (i > 0) result.add(separator);
      result.add(this[i]);
    }
    return result;
  }
}

print([].firstOrNull);                           // null
print([1, 2, 3].separated(0));                   // [1, 0, 2, 0, 3]

// ===== 扩展与泛型 =====
extension MapX<K, V> on Map<K, V> {
  V getOrElse(K key, V Function() defaultValue) =>
      containsKey(key) ? this[key]! : defaultValue();
}

var config = {'timeout': 30};
var timeout = config.getOrElse('timeout', () => 60);  // 30
var retry = config.getOrElse('retry', () => 3);       // 3
```

### 10. 枚举与 Dart 3 新特性

```dart
// ===== 枚举 =====
enum Color { red, green, blue }

// 增强枚举（带属性和方法）
enum HttpStatus {
  ok(200, 'OK'),
  notFound(404, 'Not Found'),
  serverError(500, 'Internal Server Error');

  final int code;
  final String message;
  const HttpStatus(this.code, this.message);

  bool get isSuccess => code >= 200 && code < 300;

  static HttpStatus fromCode(int code) =>
      values.firstWhere((s) => s.code == code, orElse: () => serverError);
}

print(HttpStatus.ok.code);          // 200
print(HttpStatus.ok.isSuccess);     // true

// ===== Dart 3: Records（记录类型）=====
// 轻量级不可变数据结构
var user = ('Alice', 25);                     // 位置记录
print(user.$1);                                // Alice
print(user.$2);                                // 25

var person = (name: 'Bob', age: 30);          // 命名记录
print(person.name);                            // Bob
print(person.age);                             // 30

// Records 作为返回值（多返回值）
(double, double) swap(double a, double b) => (b, a);

// ===== Dart 3: Sealed Classes（密封类）=====
// 限定子类集合，配合 Pattern Matching 实现穷尽检查
sealed class Result<T> {}
class Success<T> extends Result<T> {
  final T value;
  Success(this.value);
}
class Failure<T> extends Result<T> {
  final String error;
  Failure(this.error);
}

// ===== Dart 3: Pattern Matching =====
String describeResult(Result<int> result) {
  return switch (result) {
    Success(value: var v) => 'Success: $v',
    Failure(error: var e) => 'Error: $e',
  };
}

// 模式匹配完整示例
String classify(int value) {
  return switch (value) {
    0 => 'zero',
    1 || 2 || 3 => 'small',          // 或模式
    > 3 && < 10 => 'medium',         // 守卫模式
    >= 10 => 'large',
    _ => 'unknown',                  // 通配符
  };
}

// 解构赋值
var (name, age) = ('Alice', 25);
var (:name as userName, :age) = (name: 'Bob', age: 30);

// List 模式匹配
var [first, ..., last] = [1, 2, 3, 4, 5];  // first=1, last=5
var [a, b, ...rest] = [1, 2, 3, 4];         // a=1, b=2, rest=[3,4]

// Map 模式匹配
if (json case {'name': String name, 'age': int age}) {
  print('$name is $age years old');
}
```

### 11. 异常处理

```dart
// ===== 抛出异常 =====
throw FormatException('Invalid format');
throw 'Something went wrong';      // 可以抛出任意对象

// ===== 捕获异常 =====
try {
  var result = int.parse('abc');
} on FormatException catch (e) {
  print('Format error: ${e.message}');
} on Exception catch (e) {
  print('Exception: $e');
} catch (e, stackTrace) {
  print('Unknown error: $e');
  print('Stack: $stackTrace');
} finally {
  print('Always executed');
}

// ===== 自定义异常 =====
class NetworkException implements Exception {
  final String message;
  final int? statusCode;
  NetworkException(this.message, {this.statusCode});

  @override
  String toString() => 'NetworkException: $message (status: $statusCode)';
}

// ===== 异常最佳实践 =====
// 1. 优先使用自定义异常而非通用 Exception
// 2. 在公开 API 中声明可能抛出的异常
// 3. 不要用异常做流程控制
// 4. finally 中做资源清理
// 5. 对于可预期的错误，考虑用 Result 类型替代异常
```

### 12. 代码组织

```dart
// ===== library =====
// 每个 .dart 文件隐式是一个 library
// 显式命名：library my_library;

// ===== import =====
import 'dart:async';                           // Dart 核心库
import 'package:flutter/material.dart';         // 包依赖
import 'src/my_class.dart';                     // 相对路径
import 'package:my_app/utils/helper.dart';      // 包内绝对路径

// 别名导入（解决命名冲突）
import 'package:json_annotation/json_annotation.dart' as json;
import 'dart:convert' as convert;

// 条件导入
// import 'file_io.dart' if (dart.library.html) 'file_web.dart';

// ===== export =====
// src/utils.dart
export 'src/string_utils.dart';
export 'src/number_utils.dart';
// 使用者只需 import 'utils.dart' 即可获得所有导出

// ===== part / part of =====
// 不推荐使用，优先用 import/export
// part 'src/part_file.dart';
// part of 'main_file.dart';
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| NoSuchMethodError | 对 null 调用方法 | 检查空安全，使用 ?. 或 ?? |
| Isolate 通信失败 | 发送对象不可序列化 | 只传基本类型或使用 Compute |
| const 构造函数报错 | 字段不是 final | 所有字段必须 final |
| 泛型类型不匹配 | Dart 泛型是不变的 | 使用 cast() 或 whereType() |
| Future 不执行 | 忘记 await 或 listen | 必须消费 Future/Stream |
| late 初始化错误 | 使用前未赋值 | 确保构造/initState 中赋值 |
| Mixin 冲突 | 多个 Mixin 有同名方法 | 显式 override 指定用哪个 |
| 事件循环阻塞 | 同步耗时操作 | 用 Isolate/compute 移到后台 |
| Stream 内存泄漏 | 忘记取消订阅 | StreamSubscription.cancel() |
| equal 不生效 | 未重写 == 和 hashCode | 或使用 equatable 包 |

### 最佳实践

- 优先使用 final/const，减少可变状态
- 利用空安全，避免使用 ! 非空断言
- 使用枚举代替魔法数字和字符串
- 耗时计算用 compute/Isolate，不阻塞主线程
- Stream 用完必须取消订阅，避免内存泄漏
- 扩展方法增强已有类型，减少工具类
- Dart 3 项目优先使用 Records/Sealed/Pattern Matching
- 异常处理用自定义异常，提供上下文信息
- 集合操作优先使用函数式方法（map/where/fold）
- 代码组织用 import/export，避免 part

## 面试题

**Q1: Dart 的空安全（Null Safety）是什么？?、!、late 分别在什么场景使用？**
> 空安全是 Dart 的类型系统特性，默认类型不可为 null，需显式用 ? 标记可空类型。`?` 标记类型可为 null（如 String?）；`?.` 空安全访问，对象为 null 时返回 null 而非抛异常；`??` 空值合并，左侧为 null 时使用右侧默认值；`!` 非空断言，开发者确定不为 null 时使用，运行时若为 null 则抛异常；`late` 延迟初始化，声明时不赋值但保证使用前会初始化，适合构造函数后赋值或 initState 中赋值场景。

**Q2: Dart 的 Isolate 和线程有什么区别？为什么 Dart 用 Isolate 而不是线程？**
> Isolate 是 Dart 的并发单元，与线程的核心区别：Isolate 之间不共享内存，每个 Isolate 有独立的堆内存和事件循环。线程共享内存，需要锁来避免竞态条件。Dart 选择 Isolate 的原因：① 避免共享内存带来的竞态条件和死锁问题；② 不需要锁机制，代码更安全；③ 垃圾回收可以独立进行，不需要 STW（Stop-The-World）；④ Dart 的单线程模型使得 UI 线程不会被打断，保证流畅度。通信通过 SendPort/ReceivePort 消息传递。Flutter 中常用 compute() 简化 Isolate 使用。

**Q3: Dart 的事件循环（Event Loop）是如何工作的？MicroTask 和 Event 的优先级是怎样的？**
> Dart 是单线程模型，通过 Event Loop 处理异步任务。Event Loop 有两个队列：MicroTask 队列和 Event 队列。每次循环先清空 MicroTask 队列中的所有任务，然后再从 Event 队列取出一个任务执行，执行完后再检查 MicroTask 队列。MicroTask 优先级高于 Event，scheduleMicrotask() 添加的任务会插队执行。Future.then 回调进入 Event 队列，scheduleMicrotask 进入 MicroTask 队列。这意味着 MicroTask 可以"饿死"Event 队列，所以不宜在 MicroTask 中做耗时操作。

**Q4: final、const、late 有什么区别？**
> final 是运行时常量，只能赋值一次，可以在运行时确定值（如 DateTime.now()）。const 是编译时常量，值在编译期就必须确定，const 构造函数创建的对象是规范的（canonicalized），相同参数只创建一个实例。late 是延迟初始化标记，表示变量声明时不赋值但保证使用前会初始化，如果使用时未初始化则抛 LateInitializationError。典型场景：late + final 配合 initState 赋值；const 用于创建编译时常量列表/映射/对象；final 用于运行时确定的一次性赋值。

**Q5: Dart 的 Mixin 是什么？和继承、接口有什么区别？**
> Mixin 是一种代码复用机制，通过 with 关键字混入类中，提供方法实现而不改变类的继承关系。与继承的区别：Dart 是单继承，一个类只能 extends 一个父类，但可以 with 多个 Mixin。与接口的区别：implements 要求类自己实现所有方法，而 Mixin 提供默认实现。Mixin 的解析顺序：从右到左、从下到上（类似 Python MRO），后面混入的 Mixin 方法会覆盖前面的。可以用 on 关键字限制 Mixin 只能用于特定类的子类。适用场景：日志、序列化、验证等横切关注点的复用。

**Q6: Dart 3 的 Sealed Class 和 Pattern Matching 有什么用？**
> Sealed Class（密封类）限定子类只能在同一个库中定义，编译器知道所有可能的子类型，配合 switch 表达式可以实现穷尽检查（exhaustive checking）——如果漏掉某个子类型编译器会报错。Pattern Matching 允许在 switch 中对值进行解构和条件匹配，支持：常量模式、变量绑定模式、解构模式（Record/List/Map）、守卫模式（when/&&）、或模式（||）、通配符（_）。两者结合使用：定义 sealed class Result，子类 Success 和 Failure，switch 匹配时编译器强制处理所有情况，类似 Rust 的 enum + match。

**Q7: Dart 的 Stream 和 Future 有什么区别？分别适合什么场景？**
> Future 表示一个异步操作的最终结果，只能产生一个值（或错误），是一次性的。Stream 表示异步事件序列，可以产生多个值（或错误），是持续性的。类比：Future 是单次请求-响应，Stream 是长连接推送。Future 适合：网络请求、文件读取、一次性计算。Stream 适合：WebSocket 消息、传感器数据、用户输入事件、实时数据推送。Stream 有两种：单订阅流（Single-subscription）只能监听一次，适合读取数据；广播流（Broadcast）可多次监听，适合事件分发。Flutter 中用 StreamBuilder 监听 Stream 并自动更新 UI。

**Q8: Dart 的扩展方法（Extension）是什么？有什么限制？**
> 扩展方法允许在不修改原始类源码的情况下为其添加新方法，通过 extension ... on 语法定义。可以在 String、int 等内置类型甚至泛型上添加方法。限制：①扩展方法不是真正的类成员，只在导入扩展的代码中可见（作用域限制）；②如果两个扩展有同名方法会产生歧义，需要用 show/hide 或别名控制；③扩展方法不能访问类的私有成员；④扩展方法不能被覆写（override），静态解析时确定调用哪个扩展。最佳实践：给扩展起有意义的名称，按功能分组，避免与原生方法冲突。

---

**相关链接：**
- [[Flutter核心与Widget体系]]
- [[Flutter状态管理]]
- [[TypeScript核心]]
- [[Promise与异步]]

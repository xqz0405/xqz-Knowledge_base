---
tags: [Web前端, Flutter, 动画, 自定义绘制]
date: 2026-05-11
status: 已完成
difficulty: 中高
---

# Flutter 动画与自定义绘制

## 目录

- [1. What — Flutter 动画与自定义绘制是什么](#1-what--flutter-动画与自定义绘制是什么)
- [2. Why — 为什么需要动画与自定义绘制](#2-why--为什么需要动画与自定义绘制)
- [3. How — 如何实现 Flutter 动画与自定义绘制](#3-how--如何实现-flutter-动画与自定义绘制)
  - [3.1 Flutter 动画体系概览](#31-flutter-动画体系概览)
  - [3.2 隐式动画 Widget](#32-隐式动画-widget)
  - [3.3 AnimatedBuilder 与 AnimatedWidget](#33-animatedbuilder-与-animatedwidget)
  - [3.4 显式动画核心](#34-显式动画核心)
  - [3.5 常用 Curves 曲线](#35-常用-curves-曲线)
  - [3.6 Hero 动画](#36-hero-动画)
  - [3.7 交错动画](#37-交错动画)
  - [3.8 CustomPainter 自定义绘制](#38-custompainter-自定义绘制)
  - [3.9 自定义绘制实战](#39-自定义绘制实战)
  - [3.10 Lottie 动画集成](#310-lottie-动画集成)
  - [3.11 Rive 动画集成](#311-rive-动画集成)
  - [3.12 物理动画](#312-物理动画)
  - [3.13 动画性能优化](#313-动画性能优化)
  - [3.14 手势与动画联动](#314-手势与动画联动)
  - [3.15 页面切换动画自定义](#315-页面切换动画自定义)
- [4. 面试题精选](#4-面试题精选)
- [5. 总结](#5-总结)

---

## 1. What — Flutter 动画与自定义绘制是什么

**Flutter 动画体系** 是 Flutter 框架中用于创建流畅、高性能视觉过渡与运动效果的一整套机制。它从底层的 `Animation` 对象到顶层的动画 Widget，提供了多层次的抽象：

```
┌─────────────────────────────────────────────────────────────────┐
│                    Flutter 动画体系全景                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  隐式动画     │  │  显式动画     │  │  Hero/共享元素动画    │  │
│  │  Implicit     │  │  Explicit    │  │  Shared Element      │  │
│  │              │  │              │  │                      │  │
│  │ AnimatedXxx  │  │ Animation    │  │ Hero                 │  │
│  │ TweenAnima.. │  │ Controller   │  │ HeroController       │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘  │
│         │                 │                      │              │
│         └─────────┬───────┴──────────────────────┘              │
│                   ▼                                             │
│         ┌─────────────────────┐                                 │
│         │   Animation<T>      │  ← 核心抽象                     │
│         │   (值 + 状态 + 方向) │                                 │
│         └─────────┬───────────┘                                 │
│                   ▼                                             │
│         ┌─────────────────────┐                                 │
│         │   Ticker            │  ← 帧回调调度                    │
│         │   (vsync 驱动)      │                                 │
│         └─────────────────────┘                                 │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              CustomPainter 自定义绘制                      │   │
│  │  Canvas API + Paint + Path → 任意 2D 图形绘制             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  Lottie     │  │  Rive        │  │  物理动画             │   │
│  │  (JSON动画) │  │  (交互动画)  │  │  (Spring/Damping)    │   │
│  └─────────────┘  └──────────────┘  └──────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**自定义绘制（CustomPainter）** 是 Flutter 提供的底层 Canvas 绘图能力，通过 `CustomPaint` + `CustomPainter` 可以直接操作 Canvas API 绘制任意 2D 图形，突破内置 Widget 的局限。

### 三大动画类型对比

| 特性 | 隐式动画 | 显式动画 | Hero 动画 |
|------|---------|---------|----------|
| 控制粒度 | 低（自动管理） | 高（手动控制） | 中（框架托管） |
| 代码复杂度 | 低 | 中高 | 低 |
| 可暂停/反向 | 否 | 是 | 否 |
| 交错/编排 | 有限 | 完全支持 | 否 |
| 适用场景 | 简单属性过渡 | 复杂动画编排 | 页面间共享元素 |
| 典型代表 | AnimatedContainer | AnimationController | Hero |

---

## 2. Why — 为什么需要动画与自定义绘制

### 2.1 动画的核心价值

1. **用户体验提升**：动画让界面状态变化可感知、可预测，减少用户认知负担
2. **视觉引导**：通过运动方向、速度、缓动曲线引导用户注意力
3. **品牌表达**：独特的动画风格是品牌辨识度的重要组成部分
4. **状态反馈**：微交互（按压反馈、加载动画）让用户对操作结果有即时感知

### 2.2 自定义绘制的必要性

1. **突破内置 Widget 限制**：进度环、图表、签名板等场景没有对应内置 Widget
2. **极致性能**：直接操作 Canvas 避免了 Widget 树的重建开销
3. **精确控制**：逐像素控制绘制结果，实现设计师的高保真还原
4. **跨平台一致性**：Canvas 绘制不受平台原生控件差异影响

### 2.3 动画不规范的代价

| 问题 | 表现 | 后果 |
|------|------|------|
| 无动画的硬切换 | 界面瞬间跳变 | 用户迷失上下文 |
| 卡顿/掉帧 | 动画不流畅 | 体验低劣，用户流失 |
| 过度动画 | 每个元素都在动 | 分散注意力，增加焦虑 |
| 阻塞主线程 | 动画期间界面冻结 | 操作无响应 |

---

## 3. How — 如何实现 Flutter 动画与自定义绘制

### 3.1 Flutter 动画体系概览

Flutter 动画的核心概念链路：

```
Ticker → AnimationController → CurvedAnimation → Tween → Widget
  │            │                     │              │          │
  │            │                     │              │          │
  帧回调      0→1 值驱动           曲线映射       值域映射    视觉呈现
  (每帧触发)  (duration控制)       (easeIn等)    (0→1→颜色)  (build)
```

#### 核心类关系

```
┌──────────────────────────────────────────────────┐
│              Animation<T> (抽象)                  │
│  ┌ value: T          ← 当前动画值                │
│  ├ status: AnimationStatus ← 当前状态            │
│  └ isCompleted / isDismissed                     │
├──────────────────────────────────────────────────┤
│         ▲           ▲           ▲                │
│         │           │           │                │
│  AnimationController  CurvedAnimation  TweenAnimation│
│  (驱动源)           (曲线包装)      (值域包装)   │
└──────────────────────────────────────────────────┘
```

#### AnimationStatus 枚举

```dart
enum AnimationStatus {
  dismissed,  // 动画在起点静止（反向播放到头）
  forward,    // 动画正向播放中
  reverse,    // 动画反向播放中
  completed,  // 动画在终点静止（正向播放到头）
}
```

#### 动画选择决策树

```
需要动画？
├─ 只是属性值变化过渡？
│  ├─ 是 → 用隐式动画 AnimatedXxx
│  └─ 否 → 需要精确控制？
│     ├─ 否 → 用隐式动画
│     └─ 是 → 需要暂停/反向/交错？
│        ├─ 是 → 显式动画 AnimationController
│        └─ 否 → AnimatedBuilder 封装
├─ 页面间共享元素？
│  └─ 是 → Hero 动画
└─ 需要绘制自定义图形？
   └─ 是 → CustomPainter + Animation
```

---

### 3.2 隐式动画 Widget

隐式动画是 Flutter 中最易用的动画方案。只需修改目标属性值，框架自动在旧值与新值之间创建平滑过渡。

#### 核心原理

隐式动画 Widget 内部管理 `AnimationController`，在属性变化时自动触发过渡动画。它们都继承自 `ImplicitlyAnimatedWidget`。

```
ImplicitlyAnimatedWidget
  ├── AnimatedContainer      ← 容器属性（颜色/边框/圆角/大小/阴影）
  ├── AnimatedOpacity         ← 透明度
  ├── AnimatedPadding         ← 内边距
  ├── AnimatedAlign           ← 对齐方式
  ├── AnimatedPositioned      ← Stack 中的位置
  ├── AnimatedDefaultTextStyle← 默认文字样式
  ├── AnimatedPhysicalModel   ← 物理模型（阴影/圆角）
  ├── AnimatedTheme           ← 主题
  ├── AnimatedCrossFade       ← 两个子 Widget 交叉淡入淡出
  ├── AnimatedSwitcher         ← 任意子 Widget 切换动画
  └── TweenAnimationBuilder   ← 自定义隐式动画（万能）
```

#### 3.2.1 AnimatedContainer — 万能容器动画

```dart
class AnimatedContainerDemo extends StatefulWidget {
  @override
  _AnimatedContainerDemoState createState() => _AnimatedContainerDemoState();
}

class _AnimatedContainerDemoState extends State<AnimatedContainerDemo> {
  bool _expanded = false;
  bool _darkMode = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GestureDetector(
          onTap: () {
            setState(() {
              _expanded = !_expanded;
              _darkMode = !_darkMode;
            });
          },
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 500),   // 动画时长
            curve: Curves.easeInOutCubic,                  // 缓动曲线
            // ---- 以下属性变化时会自动动画 ----
            width: _expanded ? 280 : 120,
            height: _expanded ? 280 : 120,
            alignment: Alignment.center,
            decoration: BoxDecoration(
              color: _darkMode ? const Color(0xFF1A1A2E) : const Color(0xFFE8E8E8),
              borderRadius: BorderRadius.circular(_expanded ? 40 : 12),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(_darkMode ? 0.4 : 0.1),
                  blurRadius: _expanded ? 24 : 8,
                  offset: Offset(0, _expanded ? 12 : 4),
                ),
              ],
              border: Border.all(
                color: _darkMode ? Colors.purpleAccent : Colors.grey.shade400,
                width: _expanded ? 3 : 1,
              ),
            ),
            // ---- 动画属性结束 ----
            child: Text(
              _expanded ? 'Expanded' : 'Tap',
              style: TextStyle(
                color: _darkMode ? Colors.white : Colors.black87,
                fontSize: _expanded ? 24 : 16,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

> **关键点**：`AnimatedContainer` 对 `decoration`、`color`、`width`、`height`、`padding`、`margin`、`alignment`、`transform`、`constraints` 等属性都会自动做动画，只要新旧值之间可以插值（即实现了 `Tween`）。

#### 3.2.2 AnimatedOpacity — 透明度动画

```dart
class FadeInOutDemo extends StatefulWidget {
  @override
  _FadeInOutDemoState createState() => _FadeInOutDemoState();
}

class _FadeInOutDemoState extends State<FadeInOutDemo> {
  bool _visible = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('AnimatedOpacity')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            AnimatedOpacity(
              opacity: _visible ? 1.0 : 0.0,      // 目标透明度
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeOut,
              // 注意：即使 opacity=0，Widget 仍然占据空间且可点击
              // 需要配合 IgnorePointer 或 Offstage 使用
              child: IgnorePointer(
                ignoring: !_visible,  // 不可见时忽略点击
                child: Container(
                  width: 200,
                  height: 200,
                  decoration: BoxDecoration(
                    gradient: LinearGradient(
                      colors: [Colors.purple, Colors.blue],
                    ),
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: const Center(
                    child: Text('Fade Me', style: TextStyle(
                      color: Colors.white, fontSize: 24,
                    )),
                  ),
                ),
              ),
            ),
            const SizedBox(height: 40),
            ElevatedButton(
              onPressed: () => setState(() => _visible = !_visible),
              child: Text(_visible ? '隐藏' : '显示'),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 3.2.3 AnimatedPadding — 内边距动画

```dart
class AnimatedPaddingDemo extends StatefulWidget {
  @override
  _AnimatedPaddingDemoState createState() => _AnimatedPaddingDemoState();
}

class _AnimatedPaddingDemoState extends State<AnimatedPaddingDemo> {
  double _padding = 8.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: AnimatedPadding(
          padding: EdgeInsets.all(_padding),
          duration: const Duration(milliseconds: 400),
          curve: Curves.easeOutBack,  // 回弹效果
          child: Container(
            width: 200,
            height: 200,
            color: Colors.teal,
            child: const Center(child: Text('Content')),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            _padding = _padding < 40 ? _padding + 16 : 8;
          });
        },
        child: const Icon(Icons.padding),
      ),
    );
  }
}
```

#### 3.2.4 AnimatedAlign — 对齐方式动画

```dart
class AnimatedAlignDemo extends StatefulWidget {
  @override
  _AnimatedAlignDemoState createState() => _AnimatedAlignDemoState();
}

class _AnimatedAlignDemoState extends State<AnimatedAlignDemo> {
  Alignment _alignment = Alignment.topLeft;

  // 对齐位置列表，点击循环切换
  static const alignments = [
    Alignment.topLeft,
    Alignment.topCenter,
    Alignment.topRight,
    Alignment.centerRight,
    Alignment.bottomRight,
    Alignment.bottomCenter,
    Alignment.bottomLeft,
    Alignment.centerLeft,
    Alignment.center,
  ];
  int _index = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Container(
          width: 300,
          height: 300,
          color: Colors.grey.shade200,
          child: AnimatedAlign(
            alignment: _alignment,
            duration: const Duration(milliseconds: 500),
            curve: Curves.elasticOut,   // 弹性效果
            child: Container(
              width: 60,
              height: 60,
              decoration: const BoxDecoration(
                color: Colors.deepOrange,
                shape: BoxShape.circle,
              ),
            ),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            _index = (_index + 1) % alignments.length;
            _alignment = alignments[_index];
          });
        },
        child: const Icon(Icons.align_horizontal_center),
      ),
    );
  }
}
```

#### 3.2.5 AnimatedCrossFade — 交叉淡入淡出

```dart
class CrossFadeDemo extends StatefulWidget {
  @override
  _CrossFadeDemoState createState() => _CrossFadeDemoState();
}

class _CrossFadeDemoState extends State<CrossFadeDemo> {
  bool _showFirst = true;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            AnimatedCrossFade(
              firstChild: Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  color: Colors.blue,
                  borderRadius: BorderRadius.circular(20),
                ),
                child: const Center(
                  child: Icon(Icons.wb_sunny, size: 60, color: Colors.white),
                ),
              ),
              secondChild: Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  color: Colors.indigo,
                  borderRadius: BorderRadius.circular(100), // 变成圆形
                ),
                child: const Center(
                  child: Icon(Icons.nightlight, size: 60, color: Colors.white),
                ),
              ),
              crossFadeState: _showFirst
                  ? CrossFadeState.showFirst
                  : CrossFadeState.showSecond,
              duration: const Duration(milliseconds: 600),
              firstCurve: Curves.easeOut,
              secondCurve: Curves.easeIn,
              sizeCurve: Curves.easeInOut,  // 尺寸变化曲线
            ),
            const SizedBox(height: 32),
            ElevatedButton(
              onPressed: () => setState(() => _showFirst = !_showFirst),
              child: Text(_showFirst ? '切换到夜晚' : '切换到白天'),
            ),
          ],
        ),
      ),
    );
  }
}
```

> **注意**：`AnimatedCrossFade` 在两个子 Widget 尺寸不同时会自动做尺寸过渡动画，通过 `sizeCurve` 控制过渡曲线。但如果两个子 Widget 布局差异很大，可能出现布局跳变，需注意对齐方式。

#### 3.2.6 AnimatedSwitcher — 通用 Widget 切换动画

```dart
class AnimatedSwitcherDemo extends StatefulWidget {
  @override
  _AnimatedSwitcherDemoState createState() => _AnimatedSwitcherDemoState();
}

class _AnimatedSwitcherDemoState extends State<AnimatedSwitcherDemo> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // AnimatedSwitcher 核心：根据 key 判断子 Widget 是否变化
            AnimatedSwitcher(
              duration: const Duration(milliseconds: 400),
              // 切换进入的过渡动画
              transitionBuilder: (Widget child, Animation<double> animation) {
                // 淡入 + 缩放
                return FadeTransition(
                  opacity: animation,
                  child: ScaleTransition(
                    scale: animation,
                    child: child,
                  ),
                );
              },
              // 关键：必须为子 Widget 设置不同的 Key
              // 否则 AnimatedSwitcher 无法区分新旧 Widget
              child: Text(
                '$_count',
                key: ValueKey<int>(_count),  // ← 必须设置 Key！
                style: const TextStyle(
                  fontSize: 72,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ),
            const SizedBox(height: 40),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => setState(() => _count--),
                  child: const Icon(Icons.remove),
                ),
                const SizedBox(width: 20),
                ElevatedButton(
                  onPressed: () => setState(() => _count++),
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 3.2.7 AnimatedPositioned — Stack 内位置动画

```dart
class AnimatedPositionedDemo extends StatefulWidget {
  @override
  _AnimatedPositionedDemoState createState() => _AnimatedPositionedDemoState();
}

class _AnimatedPositionedDemoState extends State<AnimatedPositionedDemo> {
  bool _moved = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        // AnimatedPositioned 必须作为 Stack 的直接子 Widget
        child: Stack(
          children: [
            // 轨道背景
            Container(
              width: 300,
              height: 300,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey),
                borderRadius: BorderRadius.circular(16),
              ),
            ),
            AnimatedPositioned(
              left: _moved ? 220 : 20,
              top: _moved ? 220 : 20,
              duration: const Duration(milliseconds: 600),
              curve: Curves.bounceOut,
              child: GestureDetector(
                onTap: () => setState(() => _moved = !_moved),
                child: Container(
                  width: 60,
                  height: 60,
                  decoration: const BoxDecoration(
                    color: Colors.orange,
                    shape: BoxShape.circle,
                  ),
                  child: const Icon(Icons.arrow_forward, color: Colors.white),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 3.2.8 TweenAnimationBuilder — 万能隐式动画

当内置隐式动画 Widget 无法满足时，`TweenAnimationBuilder` 允许对任意可插值类型做动画：

```dart
class TweenAnimationBuilderDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: TweenAnimationBuilder<double>(
          tween: Tween<double>(begin: 0, end: 1),  // 值域：0 → 1
          duration: const Duration(seconds: 2),
          curve: Curves.elasticOut,
          builder: (context, value, child) {
            // value 在 0~1 之间变化，用它来驱动任意属性
            return Transform.scale(
              scale: 0.5 + value * 0.5,  // 缩放：0.5 → 1.0
              child: Opacity(
                opacity: value,            // 透明度：0 → 1
                child: Transform.rotate(
                  angle: value * 0.5,      // 旋转：0 → 0.5 rad
                  child: Container(
                    width: 150,
                    height: 150,
                    decoration: BoxDecoration(
                      gradient: LinearGradient(
                        colors: [
                          Color.lerp(Colors.blue, Colors.purple, value)!,
                          Color.lerp(Colors.purple, Colors.pink, value)!,
                        ],
                      ),
                      borderRadius: BorderRadius.circular(value * 30),
                    ),
                    child: const Center(
                      child: Text('Custom Tween', style: TextStyle(
                        color: Colors.white, fontSize: 18,
                      )),
                    ),
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

#### 隐式动画完整对比表

| Widget | 动画属性 | 典型场景 | 默认时长 |
|--------|---------|---------|---------|
| AnimatedContainer | 容器所有可动画属性 | 卡片展开/折叠 | 自定义 |
| AnimatedOpacity | opacity (0.0~1.0) | 淡入淡出 | 自定义 |
| AnimatedPadding | padding | 内容区呼吸感 | 自定义 |
| AnimatedAlign | alignment | 物体滑动 | 自定义 |
| AnimatedPositioned | left/top/right/bottom | Stack 内位移 | 自定义 |
| AnimatedCrossFade | 两个 Widget 交叉切换 | 日/夜模式切换 | 自定义 |
| AnimatedSwitcher | 任意 Widget 切换 | 计数器、状态切换 | 自定义 |
| TweenAnimationBuilder | 任意 Tween 值 | 自定义隐式动画 | 自定义 |

---

### 3.3 AnimatedBuilder 与 AnimatedWidget

这两个是显式动画与 Widget 之间的桥梁。它们监听 `Animation` 对象的变化并重建 UI。

#### AnimatedWidget — 继承式封装

```dart
/// 自定义旋转过渡 Widget，基于 AnimatedWidget
class SpinTransition extends AnimatedWidget {
  const SpinTransition({
    Key? key,
    required Animation<double> animation,
    this.child,
  }) : super(key: key, listenable: animation);  // 传入 Animation 作为 listenable

  @override
  Widget build(BuildContext context) {
    final animation = listenable as Animation<double>;
    return Transform.rotate(
      angle: animation.value * 2 * pi,  // 0 → 2π 旋转一圈
      child: child,
    );
  }
}

// 使用
class SpinDemo extends StatefulWidget {
  @override
  _SpinDemoState createState() => _SpinDemoState();
}

class _SpinDemoState extends State<SpinDemo> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: SpinTransition(
          animation: _controller,
          child: const FlutterLogo(size: 100),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          if (_controller.isAnimating) {
            _controller.stop();
          } else {
            _controller.repeat();  // 重复播放
          }
        },
        child: Icon(_controller.isAnimating ? Icons.pause : Icons.play_arrow),
      ),
    );
  }
}
```

#### AnimatedBuilder — 组合式封装（推荐）

```dart
class AnimatedBuilderDemo extends StatefulWidget {
  @override
  _AnimatedBuilderDemoState createState() => _AnimatedBuilderDemoState();
}

class _AnimatedBuilderDemoState extends State<AnimatedBuilderDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    );

    // 曲线 + 值域组合
    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.elasticOut,
    );

    _controller.forward();  // 启动动画
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        // AnimatedBuilder：只在动画值变化时重建 builder 内的 Widget
        // child 参数不会重建，提升性能
        child: AnimatedBuilder(
          animation: _animation,
          child: const FlutterLogo(size: 80),  // ← 不变的部分放 child
          builder: (BuildContext context, Widget? child) {
            return Transform.scale(
              scale: _animation.value,         // 0 → 1
              child: Transform.rotate(
                angle: _animation.value * pi,  // 0 → π
                child: Opacity(
                  opacity: _animation.value,   // 0 → 1
                  child: child,                // ← 使用不变的部分
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

#### AnimatedWidget vs AnimatedBuilder 对比

| 特性 | AnimatedWidget | AnimatedBuilder |
|------|---------------|-----------------|
| 使用方式 | 继承 + 重写 build | 组合 + builder 回调 |
| 代码量 | 较多（需定义新类） | 较少（内联写） |
| 复用性 | 高（封装成独立 Widget） | 低（一般一次性使用） |
| 性能优化 | 需手动分离不变子树 | child 参数自动分离 |
| 推荐度 | 可复用组件时使用 | 一般场景首选 |

---

### 3.4 显式动画核心

显式动画通过 `AnimationController` 手动控制动画的生命周期，提供暂停、反向、重复等精细控制。

#### 3.4.1 AnimationController — 动画控制器

```dart
class ControllerBasicsDemo extends StatefulWidget {
  @override
  _ControllerBasicsDemoState createState() => _ControllerBasicsDemoState();
}

class _ControllerBasicsDemoState extends State<ControllerBasicsDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 1500),
      vsync: this,   // TickerProvider，提供帧回调
      // debugLabel: 'basic-controller',  // 调试标签
      // lowerBound: 0.0,   // 最小值，默认 0
      // upperBound: 1.0,   // 最大值，默认 1
    );

    // 监听动画值变化（每帧触发）
    _controller.addListener(() {
      // print('当前值: ${_controller.value}');
      setState(() {});  // 一般不用，AnimatedBuilder 自动重建
    });

    // 监听动画状态变化
    _controller.addStatusListener((status) {
      switch (status) {
        case AnimationStatus.forward:
          print('开始正向播放');
          break;
        case AnimationStatus.reverse:
          print('开始反向播放');
          break;
        case AnimationStatus.completed:
          print('动画播放完成');
          break;
        case AnimationStatus.dismissed:
          print('动画回到起点');
          break;
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();  // ← 必须释放！否则内存泄漏
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 用 AnimatedBuilder 驱动 UI
            AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Container(
                  width: 50 + _controller.value * 200,  // 50 → 250
                  height: 50 + _controller.value * 200,
                  decoration: BoxDecoration(
                    color: Color.lerp(Colors.blue, Colors.red, _controller.value),
                    borderRadius: BorderRadius.circular(_controller.value * 125),
                  ),
                );
              },
            ),
            const SizedBox(height: 40),
            // 控制按钮
            Wrap(
              spacing: 8,
              children: [
                ElevatedButton(
                  onPressed: () => _controller.forward(),
                  child: const Text('Forward'),
                ),
                ElevatedButton(
                  onPressed: () => _controller.reverse(),
                  child: const Text('Reverse'),
                ),
                ElevatedButton(
                  onPressed: () => _controller.stop(),
                  child: const Text('Stop'),
                ),
                ElevatedButton(
                  onPressed: () {
                    _controller.repeat(reverse: true);  // 往返重复
                  },
                  child: const Text('Repeat'),
                ),
                ElevatedButton(
                  onPressed: () => _controller.reset(),
                  child: const Text('Reset'),
                ),
              ],
            ),
            const SizedBox(height: 20),
            // 进度条显示当前动画进度
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 32),
              child: LinearProgressIndicator(value: _controller.value),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### AnimationController 常用方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `forward({from})` | 从指定位置正向播放到终点 | `_controller.forward(from: 0.3)` |
| `reverse({from})` | 从指定位置反向播放到起点 | `_controller.reverse(from: 0.8)` |
| `stop({canceled})` | 停止动画 | `_controller.stop()` |
| `reset()` | 重置到起点值 | `_controller.reset()` |
| `repeat({min, max, reverse, period})` | 重复播放 | `_controller.repeat(reverse: true)` |
| `fling({velocity, ...})` | 物理弹射动画 | `_controller.fling(velocity: 2.0)` |
| `animateTo(target, {duration, curve})` | 动画到目标值 | `_controller.animateTo(0.5)` |
| `animateBack(target, {duration, curve})` | 反向动画到目标 | `_controller.animateBack(0.2)` |

#### 3.4.2 Tween — 值域映射

`Tween` 将 `[0, 1]` 的动画值映射到任意值域：

```dart
class TweenDemo extends StatefulWidget {
  @override
  _TweenDemoState createState() => _TweenDemoState();
}

class _TweenDemoState extends State<TweenDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _sizeAnimation;
  late Animation<Color?> _colorAnimation;
  late Animation<EdgeInsets> _paddingAnimation;
  late Animation<Alignment> _alignAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    final curve = CurvedAnimation(parent: _controller, curve: Curves.easeInOut);

    // 各种 Tween 映射
    _sizeAnimation = Tween<double>(begin: 50, end: 200).animate(curve);
    _colorAnimation = ColorTween(begin: Colors.blue, end: Colors.red).animate(curve);
    _paddingAnimation = EdgeInsetsTween(
      begin: const EdgeInsets.all(8),
      end: const EdgeInsets.all(40),
    ).animate(curve);
    _alignAnimation = AlignmentTween(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
    ).animate(curve);

    _controller.repeat(reverse: true);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return Container(
            alignment: _alignAnimation.value,
            padding: _paddingAnimation.value,
            child: Container(
              width: _sizeAnimation.value,
              height: _sizeAnimation.value,
              decoration: BoxDecoration(
                color: _colorAnimation.value,
                borderRadius: BorderRadius.circular(_sizeAnimation.value / 4),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

#### 常用 Tween 类型

| Tween 类 | 值类型 | 示例 |
|----------|--------|------|
| `Tween<double>` | double | `Tween(begin: 0.0, end: 1.0)` |
| `Tween<int>` | int | `Tween(begin: 0, end: 100)` |
| `ColorTween` | Color? | `ColorTween(begin: Colors.blue, end: Colors.red)` |
| `EdgeInsetsTween` | EdgeInsets | `EdgeInsetsTween(begin: ..., end: ...)` |
| `AlignmentTween` | Alignment | `AlignmentTween(begin: ..., end: ...)` |
| `TextStyleTween` | TextStyle | `TextStyleTween(begin: ..., end: ...)` |
| `DecorationTween` | Decoration | `DecorationTween(begin: ..., end: ...)` |
| `RectTween` | Rect | `RectTween(begin: ..., end: ...)` |
| `SizeTween` | Size | `SizeTween(begin: ..., end: ...)` |
| `BorderRadiusTween` | BorderRadius | `BorderRadiusTween(begin: ..., end: ...)` |
| `ThemeDataTween` | ThemeData | `ThemeDataTween(begin: ..., end: ...)` |

#### 自定义 Tween

```dart
/// 自定义波浪数据 Tween
class WaveData {
  final double amplitude;  // 振幅
  final double frequency;  // 频率
  final double phase;      // 相位

  const WaveData(this.amplitude, this.frequency, this.phase);
}

class WaveDataTween extends Tween<WaveData> {
  WaveDataTween({WaveData? begin, WaveData? end})
      : super(begin: begin, end: end);

  @override
  WaveData lerp(double t) {
    // 自定义插值逻辑
    return WaveData(
      lerpDouble(begin!.amplitude, end!.amplitude, t)!,
      lerpDouble(begin!.frequency, end!.frequency, t)!,
      lerpDouble(begin!.phase, end!.phase, t)!,
    );
  }
}
```

#### 3.4.3 CurvedAnimation — 曲线包装

```dart
// 基本用法
final curvedAnimation = CurvedAnimation(
  parent: _controller,
  curve: Curves.easeInOut,         // 正向曲线
  reverseCurve: Curves.easeIn,     // 反向曲线（可选，默认与 curve 相同）
);

// 组合曲线：先快后慢再快
final complexCurve = CurvedAnimation(
  parent: _controller,
  curve: const Interval(
    0.0, 0.5,          // 只在动画的 0~50% 区间生效
    curve: Curves.easeOut,
  ),
);
```

#### 3.4.4 多动画控制器管理

当需要同时管理多个动画时，使用 `TickerProviderStateMixin` 替代 `SingleTickerProviderStateMixin`：

```dart
class MultiAnimationDemo extends StatefulWidget {
  @override
  _MultiAnimationDemoState createState() => _MultiAnimationDemoState();
}

class _MultiAnimationDemoState extends State<MultiAnimationDemo>
    with TickerProviderStateMixin {  // ← 支持多个 Ticker
  late AnimationController _scaleController;
  late AnimationController _rotateController;
  late AnimationController _colorController;

  @override
  void initState() {
    super.initState();

    _scaleController = AnimationController(
      duration: const Duration(milliseconds: 800),
      vsync: this,
    );

    _rotateController = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    _colorController = AnimationController(
      duration: const Duration(milliseconds: 1500),
      vsync: this,
    );

    // 按顺序启动：缩放 → 旋转 → 变色
    _scaleController.forward().then((_) {
      _rotateController.repeat();
      _colorController.repeat(reverse: true);
    });
  }

  @override
  void dispose() {
    _scaleController.dispose();
    _rotateController.dispose();
    _colorController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: AnimatedBuilder(
          animation: Listenable.merge([
            _scaleController,
            _rotateController,
            _colorController,
          ]),
          builder: (context, child) {
            return Transform.scale(
              scale: _scaleController.value,
              child: Transform.rotate(
                angle: _rotateController.value * 2 * pi,
                child: Container(
                  width: 120,
                  height: 120,
                  decoration: BoxDecoration(
                    color: Color.lerp(
                      Colors.blue,
                      Colors.pink,
                      _colorController.value,
                    ),
                    borderRadius: BorderRadius.circular(20),
                  ),
                ),
              ),
            );
          },
        ),
      ),
    );
  }
}
```

---

### 3.5 常用 Curves 曲线

Curves 定义了动画值随时间的变化规律，决定了动画的"手感"。

```
Curves 曲线效果示意（时间 → 值）

linear:     ┌─────────     easeIn:     ┌──────
           /                         /
          /                         /
         /                    ┌────┘
        /               ┌────┘
  ─────┘          ┌────┘

easeOut:    ─────┐─┘      easeInOut:  ┌─────────
                /                   /
          ┌────┘              ┌────┘
     ┌────┘             ┌────┘
────┘             ─────┘

elasticOut: ──╮  ╭──     bounceOut:  ──╮ ╭─╮ ╭
               ╰╯                      ╰─╯ ╰─╯
```

#### 完整 Curves 分类表

| 类别 | 曲线 | 效果描述 |
|------|------|---------|
| **线性** | `linear` | 匀速，无缓动 |
| **ease 系列** | `easeIn` | 慢启动，快结束 |
| | `easeOut` | 快启动，慢结束 |
| | `easeInOut` | 慢启动 + 慢结束 |
| | `easeInCubic` | 三次方慢启动 |
| | `easeOutCubic` | 三次方慢结束 |
| | `easeInOutCubic` | 三次方慢起慢止 |
| | `easeInQuart` | 四次方慢启动 |
| | `easeOutQuart` | 四次方慢结束 |
| **弹性** | `elasticIn` | 弹性慢启动（弹簧拉伸感） |
| | `elasticOut` | 弹性慢结束（弹簧回弹感） |
| | `elasticInOut` | 弹性慢起慢止 |
| **回弹** | `bounceIn` | 弹球入场 |
| | `bounceOut` | 弹球落地效果 |
| | `bounceInOut` | 弹球入场+落地 |
| **特殊** | `fastOutSlowIn` | Material 标准曲线 |
| | `slowMiddle` | 中间慢 |
| | `decelerate` | 减速曲线 |
| | `easeOutBack` | 回弹过冲（略微超出目标再回弹） |
| | `easeOutExpo` | 指数衰减 |

#### 自定义 Curve

```dart
/// 自定义弹簧曲线
class SpringCurve extends Curve {
  final double damping;   // 阻尼系数
  final double frequency; // 频率

  const SpringCurve({
    this.damping = 10,
    this.frequency = 1.0,
  });

  @override
  double transformInternal(double t) {
    // 阻尼弹簧公式
    return -(pow(e, -damping * t) * cos(frequency * t)) + 1;
  }
}

// 使用
final animation = CurvedAnimation(
  parent: _controller,
  curve: const SpringCurve(damping: 8, frequency: 1.5),
);
```

#### Interval — 时间区间曲线

```dart
// Interval 用于交错动画，指定在动画的哪个时间段生效
// Interval(start, end, curve)
final firstItem = CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.0, 0.3, curve: Curves.easeOut),   // 0%~30%
);
final secondItem = CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.15, 0.45, curve: Curves.easeOut),  // 15%~45%
);
final thirdItem = CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.3, 0.6, curve: Curves.easeOut),    // 30%~60%
);
```

---

### 3.6 Hero 动画

Hero 动画实现两个页面间的共享元素过渡效果，当页面切换时，共享元素会平滑地从源页面飞到目标页面。

```
Page A                          过渡中                         Page B
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│              │          │              │          │              │
│   ┌────┐    │          │              │          │    ┌──────┐  │
│   │Hero│    │  ─────>  │   Hero 飞行  │  ─────>  │    │ Hero │  │
│   │ 图片│   │          │   位移+缩放  │          │    │ 图片  │  │
│   └────┘    │          │              │          │    └──────┘  │
│              │          │              │          │              │
└──────────────┘          └──────────────┘          └──────────────┘
  小尺寸位置              过渡动画中间态              大尺寸位置
```

#### 基本用法

```dart
// === 源页面 ===
class SourcePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('图片列表')),
      body: GridView.builder(
        padding: const EdgeInsets.all(8),
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 8,
          crossAxisSpacing: 8,
        ),
        itemCount: 6,
        itemBuilder: (context, index) {
          return GestureDetector(
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => DetailPage(index: index),
                ),
              );
            },
            child: Hero(
              tag: 'image_$index',  // ← 唯一标识，两个页面必须一致
              child: ClipRRect(
                borderRadius: BorderRadius.circular(8),
                child: Image.asset(
                  'assets/image_$index.jpg',
                  fit: BoxFit.cover,
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

// === 目标页面 ===
class DetailPage extends StatelessWidget {
  final int index;

  const DetailPage({Key? key, required this.index}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('详情 ${index + 1}')),
      body: Center(
        child: Hero(
          tag: 'image_$index',  // ← 与源页面相同的 tag
          child: Image.asset(
            'assets/image_$index.jpg',
            fit: BoxFit.contain,
          ),
        ),
      ),
    );
  }
}
```

#### 自定义 Hero 过渡动画

```dart
Hero(
  tag: 'custom_hero',
  // 自定义飞行动画构建器
  flightShuttleBuilder: (
    BuildContext flightContext,
    Animation<double> animation,
    HeroFlightDirection flightDirection,
    BuildContext fromHeroContext,
    BuildContext toHeroContext,
  ) {
    // flightDirection: push 或 pop 方向
    return AnimatedBuilder(
      animation: animation,
      builder: (context, child) {
        return Opacity(
          opacity: animation.value,
          child: Transform.scale(
            scale: 0.8 + animation.value * 0.2,
            child: child,
          ),
        );
      },
      child: flightDirection == HeroFlightDirection.push
          ? fromHeroContext.widget
          : toHeroContext.widget,
    );
  },
  child: const FlutterLogo(size: 100),
)
```

#### Hero 动画注意事项

1. **tag 必须唯一**：同一页面内不能有重复 tag
2. **过渡方向**：push 和 pop 都会触发 Hero 动画
3. **性能**：Hero 动画期间会创建覆盖层（Overlay），避免在 Hero Widget 上做复杂布局
4. **占位处理**：可使用 `placeholderBuilder` 在过渡期间显示占位 Widget

```dart
Hero(
  tag: 'hero_placeholder',
  placeholderBuilder: (context, size, child) {
    // 过渡期间在原位显示占位符
    return Container(
      width: size.width,
      height: size.height,
      color: Colors.grey.shade300,
    );
  },
  child: Image.asset('assets/photo.jpg'),
)
```

---

### 3.7 交错动画

交错动画（Staggered Animation）让多个动画按时间顺序依次启动，产生编排效果。

```
时间轴 (0 ──────────────────────── 1)

缩放:  ████████░░░░░░░░░░░░░░░░░░░   0%~30%
旋转:  ░░░░████████░░░░░░░░░░░░░░░   15%~45%
颜色:  ░░░░░░░░████████░░░░░░░░░░░   30%~60%
位移:  ░░░░░░░░░░░░░░████████░░░░░   50%~80%
透明:  ░░░░░░░░░░░░░░░░░░░░██████   70%~100%

█ = 动画活跃区间   ░ = 空闲
```

#### 交错动画实现

```dart
class StaggeredAnimationDemo extends StatefulWidget {
  @override
  _StaggeredAnimationDemoState createState() => _StaggeredAnimationDemoState();
}

class _StaggeredAnimationDemoState extends State<StaggeredAnimationDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  // 定义各阶段的动画
  late Animation<double> _scale;
  late Animation<double> _rotation;
  late Animation<double> _opacity;
  late Animation<EdgeInsets> _padding;
  late Animation<Color?> _color;
  late Animation<double> _borderRadius;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 2000),
      vsync: this,
    );

    // 各动画的 Interval 定义了在总时间轴上的区间
    _scale = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.0, 0.3, curve: Curves.easeOut),  // 0%~30%
      ),
    );

    _rotation = Tween<double>(begin: 0.0, end: 0.5).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.15, 0.45, curve: Curves.easeOut),  // 15%~45%
      ),
    );

    _opacity = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.25, 0.55, curve: Curves.easeIn),  // 25%~55%
      ),
    );

    _padding = EdgeInsetsTween(
      begin: const EdgeInsets.all(0),
      end: const EdgeInsets.all(40),
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.35, 0.65, curve: Curves.easeOut),  // 35%~65%
      ),
    );

    _color = ColorTween(begin: Colors.blue, end: Colors.orange).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.5, 0.8, curve: Curves.easeInOut),  // 50%~80%
      ),
    );

    _borderRadius = Tween<double>(begin: 0, end: 40).animate(
      CurvedAnimation(
        parent: _controller,
        curve: const Interval(0.7, 1.0, curve: Curves.easeOut),  // 70%~100%
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Container(
                  padding: _padding.value,
                  child: Opacity(
                    opacity: _opacity.value,
                    child: Transform.scale(
                      scale: _scale.value,
                      child: Transform.rotate(
                        angle: _rotation.value * 2 * pi,
                        child: Container(
                          width: 150,
                          height: 150,
                          decoration: BoxDecoration(
                            color: _color.value,
                            borderRadius: BorderRadius.circular(
                              _borderRadius.value,
                            ),
                            boxShadow: [
                              BoxShadow(
                                color: Colors.black.withOpacity(0.3),
                                blurRadius: 20,
                                offset: const Offset(0, 10),
                              ),
                            ],
                          ),
                          child: const Center(
                            child: Text('Staggered', style: TextStyle(
                              color: Colors.white, fontSize: 20,
                            )),
                          ),
                        ),
                      ),
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 40),
            ElevatedButton(
              onPressed: () {
                if (_controller.isCompleted) {
                  _controller.reverse();
                } else {
                  _controller.forward();
                }
              },
              child: const Text('播放交错动画'),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 列表项交错入场动画

```dart
class StaggeredListDemo extends StatefulWidget {
  final int itemCount;
  const StaggeredListDemo({Key? key, this.itemCount = 8}) : super(key: key);

  @override
  _StaggeredListDemoState createState() => _StaggeredListDemoState();
}

class _StaggeredListDemoState extends State<StaggeredListDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 1500),
      vsync: this,
    );
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      padding: const EdgeInsets.all(16),
      itemCount: widget.itemCount,
      itemBuilder: (context, index) {
        // 每个列表项的延迟 = index / totalCount * 0.6
        // 总动画时间最多覆盖 60%，留 40% 给最后一项完成动画
        final startDelay = (index / widget.itemCount) * 0.6;
        final endDelay = startDelay + 0.4;

        return SlideTransition(
          position: Tween<Offset>(
            begin: const Offset(0, 0.5),  // 从下方偏移
            end: Offset.zero,
          ).animate(CurvedAnimation(
            parent: _controller,
            curve: Interval(startDelay, endDelay, curve: Curves.easeOut),
          )),
          child: FadeTransition(
            opacity: CurvedAnimation(
              parent: _controller,
              curve: Interval(startDelay, endDelay, curve: Curves.easeOut),
            ),
            child: _buildListItem(index),
          ),
        );
      },
    );
  }

  Widget _buildListItem(int index) {
    return Card(
      margin: const EdgeInsets.only(bottom: 12),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Colors.primaries[index % Colors.primaries.length],
          child: Text('${index + 1}'),
        ),
        title: Text('列表项 ${index + 1}'),
        subtitle: Text('这是第 ${index + 1} 个列表项的描述内容'),
        trailing: const Icon(Icons.chevron_right),
      ),
    );
  }
}
```

---

### 3.8 CustomPainter 自定义绘制

`CustomPainter` 是 Flutter 底层 2D 绘图接口，通过 Canvas API 直接绘制图形。

```
┌────────────────────────────────────────────┐
│            CustomPaint Widget              │
│  ┌──────────────────────────────────────┐  │
│  │         CustomPainter                │  │
│  │  ┌────────────────────────────────┐  │  │
│  │  │     paint(Canvas, Size)        │  │  │
│  │  │                                │  │  │
│  │  │  Canvas API:                   │  │  │
│  │  │  ├─ drawLine                   │  │  │
│  │  │  ├─ drawRect                   │  │  │
│  │  │  ├─ drawCircle                 │  │  │
│  │  │  ├─ drawArc                    │  │  │
│  │  │  ├─ drawPath                   │  │  │
│  │  │  ├─ drawParagraph (文字)        │  │  │
│  │  │  ├─ drawImage                  │  │  │
│  │  │  ├─ drawShadow                 │  │  │
│  │  │  └─ save/restore/translate/..  │  │  │
│  │  │                                │  │  │
│  │  │  Paint 配置:                   │  │  │
│  │  │  ├─ color / shader             │  │  │
│  │  │  ├─ strokeWidth               │  │  │
│  │  │  ├─ style (fill/stroke)        │  │  │
│  │  │  ├─ strokeCap / strokeJoin     │  │  │
│  │  │  ├─ filterQuality              │  │  │
│  │  │  └─ maskFilter / blendMode     │  │  │
│  │  └────────────────────────────────┘  │  │
│  │                                      │  │
│  │  shouldRepaint(oldDelegate) → bool   │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

#### CustomPainter 基本结构

```dart
class MyCustomPainter extends CustomPainter {
  // 绘制逻辑
  @override
  void paint(Canvas canvas, Size size) {
    // size 是 CustomPaint 指定的绘制区域大小
    final width = size.width;
    final height = size.height;
    final center = Offset(width / 2, height / 2);

    // 1. 创建 Paint 对象
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 4
      ..style = PaintingStyle.stroke  // stroke 描边 / fill 填充
      ..strokeCap = StrokeCap.round   // 线帽样式
      ..isAntiAlias = true;           // 抗锯齿

    // 2. 使用 Canvas 绘制
    canvas.drawLine(
      Offset(0, 0),                   // 起点
      Offset(width, height),          // 终点
      paint,
    );

    // 3. 绘制圆形
    paint.style = PaintingStyle.fill;
    paint.color = Colors.red.withOpacity(0.5);
    canvas.drawCircle(center, 50, paint);

    // 4. 绘制矩形
    paint.color = Colors.green;
    paint.style = PaintingStyle.stroke;
    canvas.drawRect(
      Rect.fromCenter(center: center, width: 100, height: 80),
      paint,
    );

    // 5. 绘制圆角矩形
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(20, 20, width - 40, height - 40),
        const Radius.circular(16),
      ),
      paint..color = Colors.orange,
    );
  }

  // 是否需要重绘（性能关键）
  @override
  bool shouldRepaint(covariant MyCustomPainter oldDelegate) {
    // 返回 true → 重绘；返回 false → 跳过
    // 通常比较属性是否变化
    return true;  // 简单场景可直接 true
  }
}

// 使用
CustomPaint(
  size: const Size(300, 300),  // 指定绘制区域大小
  painter: MyCustomPainter(),
)
```

#### Canvas API 速查

| 方法 | 说明 | 参数要点 |
|------|------|---------|
| `drawLine(p1, p2, paint)` | 画线 | 起点终点 |
| `drawRect(rect, paint)` | 画矩形 | Rect 对象 |
| `drawRRect(rrect, paint)` | 画圆角矩形 | RRect 对象 |
| `drawCircle(center, radius, paint)` | 画圆 | 圆心+半径 |
| `drawOval(rect, paint)` | 画椭圆 | 外接矩形 |
| `drawArc(rect, startAngle, sweepAngle, useCenter, paint)` | 画弧 | 角度用弧度 |
| `drawPath(path, paint)` | 画路径 | Path 对象 |
| `drawParagraph(para, offset)` | 画文字 | ParagraphBuilder 构建 |
| `drawImage(image, offset, paint)` | 画图片 | 需异步加载 |
| `drawImageRect(image, src, dst, paint)` | 画图片局部 | 源区域+目标区域 |
| `drawShadow(path, color, elevation, transparentOccluder)` | 画阴影 | Material 阴影 |
| `drawPoints(pointMode, points, paint)` | 画点 | points/lines/polygon |
| `drawDRRect(outer, inner, paint)` | 画双圆角矩形 | 进度条常用 |
| `drawVertices(vertices, blendMode, paint)` | 画顶点 | 三角形网格 |
| `save()` / `restore()` | 保存/恢复状态 | 配合变换使用 |
| `translate(dx, dy)` | 平移 | 坐标系平移 |
| `rotate(radians)` | 旋转 | 弧度制 |
| `scale(sx, sy)` | 缩放 | 双轴缩放 |
| `clipRect/clipRRect/clipPath` | 裁剪 | 后续绘制只在裁剪区内 |

#### Path 路径绘制

```dart
/// 绘制一个五角星
void drawStar(Canvas canvas, Offset center, double radius, Paint paint) {
  final path = Path();
  const points = 5;
  const innerRadiusRatio = 0.4;  // 内半径比例

  for (int i = 0; i < points * 2; i++) {
    final angle = (i * pi / points) - pi / 2;  // 起始角度在顶部
    final r = i.isEven ? radius : radius * innerRadiusRatio;
    final x = center.dx + r * cos(angle);
    final y = center.dy + r * sin(angle);

    if (i == 0) {
      path.moveTo(x, y);
    } else {
      path.lineTo(x, y);
    }
  }
  path.close();  // 闭合路径

  canvas.drawPath(path, paint);
}

/// 绘制贝塞尔曲线
void drawBezierCurves(Canvas canvas, Size size) {
  final paint = Paint()
    ..color = Colors.purple
    ..strokeWidth = 3
    ..style = PaintingStyle.stroke;

  // 二次贝塞尔曲线 (1 个控制点)
  final quadraticPath = Path();
  quadraticPath.moveTo(0, size.height / 2);
  quadraticPath.quadraticBezierTo(
    size.width / 2, 0,          // 控制点
    size.width, size.height / 2, // 终点
  );
  canvas.drawPath(quadraticPath, paint..color = Colors.blue);

  // 三次贝塞尔曲线 (2 个控制点)
  final cubicPath = Path();
  cubicPath.moveTo(0, size.height * 0.7);
  cubicPath.cubicTo(
    size.width * 0.25, size.height * 0.3,  // 控制点1
    size.width * 0.75, size.height * 0.9,   // 控制点2
    size.width, size.height * 0.5,           // 终点
  );
  canvas.drawPath(cubicPath, paint..color = Colors.red);
}
```

#### Paint 高级配置

```dart
// 渐变填充
final gradientPaint = Paint()
  ..shader = LinearGradient(
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
    colors: [Colors.blue, Colors.purple, Colors.pink],
    stops: const [0.0, 0.5, 1.0],
  ).createShader(Rect.fromLTWH(0, 0, size.width, size.height));

// 径向渐变
final radialPaint = Paint()
  ..shader = RadialGradient(
    center: Alignment.center,
    radius: 0.5,
    colors: [Colors.yellow, Colors.orange, Colors.red],
  ).createShader(Rect.fromLTWH(0, 0, size.width, size.height));

// 扫描渐变（饼图常用）
final sweepPaint = Paint()
  ..shader = SweepGradient(
    center: Alignment.center,
    colors: [Colors.red, Colors.yellow, Colors.green, Colors.blue, Colors.red],
  ).createShader(Rect.fromLTWH(0, 0, size.width, size.height));

// 模糊滤镜
final blurPaint = Paint()
  ..color = Colors.blue.withOpacity(0.5)
  ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);

// 混合模式
final blendPaint = Paint()
  ..color = Colors.red
  ..blendMode = BlendMode.multiply;
```

---

### 3.9 自定义绘制实战

#### 3.9.1 圆形进度环

```dart
class CircularProgressPainter extends CustomPainter {
  final double progress;        // 0.0 ~ 1.0
  final Color progressColor;
  final Color backgroundColor;
  final double strokeWidth;
  final double startAngle;      // 起始角度（弧度）

  CircularProgressPainter({
    required this.progress,
    this.progressColor = Colors.blue,
    this.backgroundColor = Colors.grey,
    this.strokeWidth = 12,
    this.startAngle = -pi / 2,  // 默认从顶部开始
  });

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = (min(size.width, size.height) - strokeWidth) / 2;

    // 背景圆环
    final bgPaint = Paint()
      ..color = backgroundColor
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round;

    canvas.drawCircle(center, radius, bgPaint);

    // 进度圆弧
    final progressPaint = Paint()
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round
      ..shader = SweepGradient(
        startAngle: startAngle,
        endAngle: startAngle + 2 * pi * progress,
        colors: [
          progressColor.withOpacity(0.5),
          progressColor,
        ],
      ).createShader(Rect.fromCircle(center: center, radius: radius));

    final sweepAngle = 2 * pi * progress;
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      startAngle,
      sweepAngle,
      false,
      progressPaint,
    );

    // 中心文字
    final textSpan = TextSpan(
      text: '${(progress * 100).toInt()}%',
      style: TextStyle(
        color: progressColor,
        fontSize: radius * 0.5,
        fontWeight: FontWeight.bold,
      ),
    );
    final textPainter = TextPainter(
      text: textSpan,
      textDirection: TextDirection.ltr,
    )..layout();

    textPainter.paint(
      canvas,
      Offset(
        center.dx - textPainter.width / 2,
        center.dy - textPainter.height / 2,
      ),
    );
  }

  @override
  bool shouldRepaint(covariant CircularProgressPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}

// 使用：配合动画
class CircularProgressDemo extends StatefulWidget {
  @override
  _CircularProgressDemoState createState() => _CircularProgressDemoState();
}

class _CircularProgressDemoState extends State<CircularProgressDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: AnimatedBuilder(
          animation: _controller,
          builder: (context, child) {
            return CustomPaint(
              size: const Size(200, 200),
              painter: CircularProgressPainter(
                progress: _controller.value,
                progressColor: Colors.deepPurple,
              ),
            );
          },
        ),
      ),
    );
  }
}
```

#### 3.9.2 柱状图

```dart
class BarChartPainter extends CustomPainter {
  final List<double> values;     // 数值列表 (0.0 ~ 1.0)
  final List<String> labels;     // 标签列表
  final List<Color> colors;      // 每根柱子颜色
  final double barWidth;
  final double barSpacing;

  BarChartPainter({
    required this.values,
    required this.labels,
    required this.colors,
    this.barWidth = 30,
    this.barSpacing = 20,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final chartHeight = size.height - 40;  // 留出底部标签空间
    final chartWidth = values.length * (barWidth + barSpacing) - barSpacing;
    final startX = (size.width - chartWidth) / 2;

    // 绘制 Y 轴参考线
    final gridPaint = Paint()
      ..color = Colors.grey.shade300
      ..strokeWidth = 1;

    for (int i = 0; i <= 4; i++) {
      final y = chartHeight * (1 - i / 4);
      canvas.drawLine(
        Offset(startX - 10, y),
        Offset(startX + chartWidth + 10, y),
        gridPaint,
      );
    }

    // 绘制柱子
    for (int i = 0; i < values.length; i++) {
      final x = startX + i * (barWidth + barSpacing);
      final barHeight = chartHeight * values[i];
      final y = chartHeight - barHeight;

      // 柱子渐变
      final barPaint = Paint()
        ..shader = LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: [
            colors[i % colors.length],
            colors[i % colors.length].withOpacity(0.6),
          ],
        ).createShader(Rect.fromLTWH(x, y, barWidth, barHeight));

      // 绘制圆角柱子
      final rrect = RRect.fromRectAndCorners(
        Rect.fromLTWH(x, y, barWidth, barHeight),
        topLeft: const Radius.circular(6),
        topRight: const Radius.circular(6),
      );
      canvas.drawRRect(rrect, barPaint);

      // 数值标签
      final valueText = TextSpan(
        text: '${(values[i] * 100).toInt()}',
        style: TextStyle(
          color: colors[i % colors.length],
          fontSize: 12,
          fontWeight: FontWeight.bold,
        ),
      );
      final tp = TextPainter(
        text: valueText,
        textDirection: TextDirection.ltr,
      )..layout();
      tp.paint(canvas, Offset(x + barWidth / 2 - tp.width / 2, y - 20));

      // 底部标签
      final labelText = TextSpan(
        text: labels[i],
        style: TextStyle(color: Colors.grey.shade600, fontSize: 11),
      );
      final lp = TextPainter(
        text: labelText,
        textDirection: TextDirection.ltr,
      )..layout();
      lp.paint(
        canvas,
        Offset(x + barWidth / 2 - lp.width / 2, chartHeight + 8),
      );
    }
  }

  @override
  bool shouldRepaint(covariant BarChartPainter oldDelegate) {
    return values != oldDelegate.values;
  }
}
```

#### 3.9.3 波浪线

```dart
class WavePainter extends CustomPainter {
  final double animationValue;  // 动画值 (0.0 ~ 1.0 不断循环)
  final Color waveColor;
  final double waveHeight;      // 波浪高度
  final double waveLength;      // 波浪一个周期的宽度

  WavePainter({
    required this.animationValue,
    this.waveColor = Colors.blue,
    this.waveHeight = 20,
    this.waveLength = 200,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // 绘制多层波浪，由远到近
    _drawWaveLayer(
      canvas, size,
      amplitude: waveHeight * 0.6,
      wavelength: waveLength * 1.5,
      phase: animationValue * 2 * pi * 0.5,
      yOffset: size.height * 0.55,
      color: waveColor.withOpacity(0.2),
    );
    _drawWaveLayer(
      canvas, size,
      amplitude: waveHeight * 0.8,
      wavelength: waveLength,
      phase: animationValue * 2 * pi,
      yOffset: size.height * 0.6,
      color: waveColor.withOpacity(0.4),
    );
    _drawWaveLayer(
      canvas, size,
      amplitude: waveHeight,
      wavelength: waveLength * 0.8,
      phase: -animationValue * 2 * pi * 1.5,  // 反向运动
      yOffset: size.height * 0.65,
      color: waveColor.withOpacity(0.6),
    );
  }

  void _drawWaveLayer(
    Canvas canvas,
    Size size, {
    required double amplitude,
    required double wavelength,
    required double phase,
    required double yOffset,
    required Color color,
  }) {
    final paint = Paint()
      ..color = color
      ..style = PaintingStyle.fill;

    final path = Path();
    path.moveTo(0, yOffset);

    // 使用正弦函数生成波浪路径
    for (double x = 0; x <= size.width; x++) {
      final y = yOffset + amplitude * sin((x / wavelength) * 2 * pi + phase);
      path.lineTo(x, y);
    }

    // 封闭路径到底部
    path.lineTo(size.width, size.height);
    path.lineTo(0, size.height);
    path.close();

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant WavePainter oldDelegate) {
    return oldDelegate.animationValue != animationValue;
  }
}

// 波浪动画 Widget
class WaveAnimationWidget extends StatefulWidget {
  @override
  _WaveAnimationWidgetState createState() => _WaveAnimationWidgetState();
}

class _WaveAnimationWidgetState extends State<WaveAnimationWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    )..repeat();  // 持续循环
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(double.infinity, 200),
      painter: WavePainter(animationValue: _controller.value),
    );
  }
}
```

#### 3.9.4 签名板

```dart
class SignaturePainter extends CustomPainter {
  final List<List<Offset>> strokes;  // 所有笔画，每条笔画是一组点
  final Color strokeColor;
  final double strokeWidth;

  SignaturePainter({
    required this.strokes,
    this.strokeColor = Colors.black,
    this.strokeWidth = 3.0,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = strokeColor
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round
      ..strokeJoin = StrokeJoin.round
      ..style = PaintingStyle.stroke;

    for (final stroke in strokes) {
      if (stroke.length < 2) continue;

      final path = Path();
      path.moveTo(stroke[0].dx, stroke[0].dy);

      // 使用二次贝塞尔曲线平滑连接各点
      for (int i = 1; i < stroke.length - 1; i++) {
        final midX = (stroke[i].dx + stroke[i + 1].dx) / 2;
        final midY = (stroke[i].dy + stroke[i + 1].dy) / 2;
        path.quadraticBezierTo(stroke[i].dx, stroke[i].dy, midX, midY);
      }

      // 最后一段直接连到终点
      if (stroke.length > 1) {
        path.lineTo(stroke.last.dx, stroke.last.dy);
      }

      canvas.drawPath(path, paint);
    }
  }

  @override
  bool shouldRepaint(covariant SignaturePainter oldDelegate) {
    return strokes != oldDelegate.strokes;
  }
}

// 签名板 Widget
class SignatureBoard extends StatefulWidget {
  final Color strokeColor;
  final double strokeWidth;

  const SignatureBoard({
    Key? key,
    this.strokeColor = Colors.black,
    this.strokeWidth = 3.0,
  }) : super(key: key);

  @override
  _SignatureBoardState createState() => _SignatureBoardState();
}

class _SignatureBoardState extends State<SignatureBoard> {
  final List<List<Offset>> _strokes = [];
  List<Offset> _currentStroke = [];

  void _onPanStart(DragStartDetails details) {
    setState(() {
      _currentStroke = [details.localPosition];
      _strokes.add(_currentStroke);
    });
  }

  void _onPanUpdate(DragUpdateDetails details) {
    setState(() {
      _currentStroke.add(details.localPosition);
    });
  }

  void _onPanEnd(DragEndDetails details) {
    setState(() {
      _currentStroke = [];
    });
  }

  void _clear() {
    setState(() {
      _strokes.clear();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Expanded(
          child: GestureDetector(
            onPanStart: _onPanStart,
            onPanUpdate: _onPanUpdate,
            onPanEnd: _onPanEnd,
            child: Container(
              color: Colors.white,
              child: CustomPaint(
                size: Size.infinite,
                painter: SignaturePainter(
                  strokes: _strokes,
                  strokeColor: widget.strokeColor,
                  strokeWidth: widget.strokeWidth,
                ),
              ),
            ),
          ),
        ),
        Padding(
          padding: const EdgeInsets.all(8),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.end,
            children: [
              ElevatedButton.icon(
                onPressed: _clear,
                icon: const Icon(Icons.clear),
                label: const Text('清除'),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

---

### 3.10 Lottie 动画集成

[Lottie](https://airbnb.io/lottie/) 是 Airbnb 开源的高性能动画库，通过 JSON 文件播放 After Effects 导出的动画，体积小、质量高。

#### 安装

```yaml
# pubspec.yaml
dependencies:
  lottie: ^3.1.0   # 使用最新版本
```

#### 基本用法

```dart
import 'package:lottie/lottie.dart';

class LottieDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // 方式1：从 assets 加载
            Lottie.asset(
              'assets/animations/loading.json',
              width: 200,
              height: 200,
              fit: BoxFit.contain,
              // repeat: true,       // 是否重复播放
              // reverse: false,     // 是否反向播放
              // animate: true,      // 是否自动播放
              // delegates: LottieDelegates(...), // 自定义渲染
            ),

            const SizedBox(height: 40),

            // 方式2：从网络加载
            Lottie.network(
              'https://assets.example.com/animation.json',
              width: 150,
              height: 150,
              // 加载中占位
              frameBuilder: (context, child, loaded) {
                if (loaded) return child;
                return const CircularProgressIndicator();
              },
              // 错误处理
              errorBuilder: (context, error, stackTrace) {
                return const Icon(Icons.error, size: 48, color: Colors.red);
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 使用 AnimationController 控制 Lottie

```dart
class ControlledLottieDemo extends StatefulWidget {
  @override
  _ControlledLottieDemoState createState() => _ControlledLottieDemoState();
}

class _ControlledLottieDemoState extends State<ControlledLottieDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Future<LottieComposition> _composition;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    );

    // 预加载 Lottie 组合
    _composition = AssetLottie('assets/animations/complex.json').load();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FutureBuilder<LottieComposition>(
              future: _composition,
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return const CircularProgressIndicator();
                }

                return Lottie(
                  composition: snapshot.data!,
                  controller: _controller,  // ← 外部控制
                  width: 250,
                  height: 250,
                );
              },
            ),
            const SizedBox(height: 32),
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ElevatedButton(
                  onPressed: () => _controller.forward(),
                  child: const Text('播放'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () => _controller.reverse(),
                  child: const Text('反向'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () => _controller.stop(),
                  child: const Text('暂停'),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () => _controller.repeat(reverse: true),
                  child: const Text('循环'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Lottie 动态属性修改

```dart
Lottie.asset(
  'assets/animations/pulse.json',
  width: 200,
  height: 200,
  // 动态替换颜色
  delegates: LottieDelegates(
    values: [
      // 替换指定图层的颜色
      ValueDelegate.color(
        ['Shape', 'Fill'],    // 图层路径
        value: Colors.purple,  // 新颜色
      ),
      // 替换透明度
      ValueDelegate.opacity(
        ['Shape', 'Fill'],
        value: 0.8,
      ),
      // 替换文本
      ValueDelegate.text(
        ['TextLayer'],
        value: 'Flutter',
      ),
    ],
  ),
)
```

---

### 3.11 Rive 动画集成

[Rive](https://rive.app/) 是新一代交互式动画工具，支持状态机、事件触发，比 Lottie 更灵活。

#### 安装

```yaml
# pubspec.yaml
dependencies:
  rive: ^0.13.0   # 使用最新版本
```

#### 基本用法

```dart
import 'package:rive/rive.dart';

class RiveDemo extends StatefulWidget {
  @override
  _RiveDemoState createState() => _RiveDemoState();
}

class _RiveDemoState extends State<RiveDemo> {
  // Rive 状态机控制器
  StateMachineController? _controller;
  SMIInput<bool>? _hoverInput;     // 布尔输入
  SMIInput<double>? _progressInput; // 数值输入
  SMIInput<String>? _stateInput;   // 字符串输入（状态切换）

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  /// 初始化 Rive 并获取状态机输入
  void _onRiveInit(Artboard artboard) {
    _controller = StateMachineController.fromArtboard(
      artboard,
      'State Machine',  // Rive 中定义的状态机名称
    );

    if (_controller != null) {
      artboard.addController(_controller!);

      // 获取状态机输入端口
      _hoverInput = _controller?.findInput<bool>('isHovered');
      _progressInput = _controller?.findInput<double>('progress');
      _stateInput = _controller?.findInput<String>('state');

      setState(() {});
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // Rive 动画 Widget
            GestureDetector(
              onEnter: (_) => _hoverInput?.value = true,
              onExit: (_) => _hoverInput?.value = false,
              child: SizedBox(
                width: 300,
                height: 300,
                child: RiveAnimation.asset(
                  'assets/animations/interactive.riv',
                  onInit: _onRiveInit,  // 初始化回调
                ),
              ),
            ),
            const SizedBox(height: 32),
            // 通过 Slider 控制 Rive 动画参数
            if (_progressInput != null)
              SizedBox(
                width: 250,
                child: Slider(
                  value: _progressInput!.value,
                  min: 0,
                  max: 100,
                  onChanged: (value) {
                    setState(() {
                      _progressInput!.value = value;
                    });
                  },
                ),
              ),
            const SizedBox(height: 16),
            // 通过按钮切换 Rive 状态
            Wrap(
              spacing: 8,
              children: ['idle', 'loading', 'success', 'error'].map((state) {
                return ElevatedButton(
                  onPressed: () {
                    _stateInput?.value = state;
                  },
                  child: Text(state),
                );
              }).toList(),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Rive 与 Lottie 对比

| 特性 | Lottie | Rive |
|------|--------|------|
| 文件格式 | JSON | .riv (二进制) |
| 文件大小 | 较大 | 较小 |
| 运行时控制 | 有限 | 完整状态机 |
| 交互性 | 弱（仅播放控制） | 强（事件+输入+状态切换） |
| 编辑工具 | After Effects + Bodymovin | Rive Editor（专用） |
| 代码集成 | 简单 | 中等 |
| 动态属性 | 支持 | 原生支持 |
| 适用场景 | 播放型动画 | 交互型动画 |
| 社区资源 | 非常丰富 | 逐渐增长 |

---

### 3.12 物理动画

Flutter 提供基于物理模拟的动画，让运动效果更自然。

#### 3.12.1 SpringSimulation — 弹簧动画

```dart
class SpringAnimationDemo extends StatefulWidget {
  @override
  _SpringAnimationDemoState createState() => _SpringAnimationDemoState();
}

class _SpringAnimationDemoState extends State<SpringAnimationDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  // 弹簧参数
  double _springMass = 1.0;
  double _springStiffness = 100;   // 刚度
  double _springDamping = 10;      // 阻尼

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );

    _animateWithSpring();
  }

  void _animateWithSpring() {
    // 创建弹簧模拟
    final spring = SpringDescription(
      mass: _springMass,
      stiffness: _springStiffness,
      damping: _springDamping,
    );

    final simulation = SpringSimulation(
      spring,
      0.0,   // 起始值
      1.0,   // 目标值
      0.0,   // 初始速度
    );

    // 用模拟驱动控制器
    _controller.animateWith(simulation);

    _animation = Tween<double>(begin: 0, end: 1).animate(_controller);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            AnimatedBuilder(
              animation: _controller,
              builder: (context, child) {
                return Transform.translate(
                  offset: Offset(0, 200 * (1 - _controller.value)),
                  child: Transform.scale(
                    scale: _controller.value,
                    child: Container(
                      width: 100,
                      height: 100,
                      decoration: BoxDecoration(
                        color: Colors.orange,
                        borderRadius: BorderRadius.circular(20),
                        boxShadow: [
                          BoxShadow(
                            color: Colors.orange.withOpacity(0.4),
                            blurRadius: 20,
                            offset: Offset(0, 10 * (1 - _controller.value)),
                          ),
                        ],
                      ),
                    ),
                  ),
                );
              },
            ),
            const SizedBox(height: 40),
            // 弹簧参数调节
            _buildSlider('质量', _springMass, 0.1, 10, (v) {
              _springMass = v;
              _animateWithSpring();
            }),
            _buildSlider('刚度', _springStiffness, 10, 500, (v) {
              _springStiffness = v;
              _animateWithSpring();
            }),
            _buildSlider('阻尼', _springDamping, 1, 50, (v) {
              _springDamping = v;
              _animateWithSpring();
            }),
          ],
        ),
      ),
    );
  }

  Widget _buildSlider(
    String label,
    double value,
    double min,
    double max,
    ValueChanged<double> onChanged,
  ) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 4),
      child: Row(
        children: [
          SizedBox(width: 40, child: Text(label)),
          Expanded(
            child: Slider(value: value, min: min, max: max, onChanged: onChanged),
          ),
        ],
      ),
    );
  }
}
```

#### 弹簧参数效果说明

| 参数 | 作用 | 小值效果 | 大值效果 |
|------|------|---------|---------|
| mass (质量) | 物体惯性 | 快速响应 | 惯性大，慢启动 |
| stiffness (刚度) | 弹簧硬度 | 软弹，振荡多 | 硬弹，快速到位 |
| damping (阻尼) | 能量衰减 | 振荡时间长 | 快速收敛不振荡 |

```
低阻尼(振荡):          临界阻尼(最佳):       高阻尼(过阻尼):
   ╭─╮                    ╭─────              ╭────────
  ╱   ╲                  ╱                    ╱
 ╱     ╲           ─────╱              ──────╱
╱       ╲
╱        ╲─╮
╱          ╲╱╲─╮
╱              ╲╱
```

#### 3.12.2 使用 SpringType 简化

```dart
// Flutter 3.x 提供的简化弹簧动画
// 通过 SpringDescription.withDampingRatio 构造

// 临界阻尼（无振荡，最快到位）
final critical = SpringDescription.withDampingRatio(
  mass: 1,
  stiffness: 100,
  ratio: 1.0,  // 临界阻尼比
);

// 欠阻尼（有振荡回弹）
final underDamped = SpringDescription.withDampingRatio(
  mass: 1,
  stiffness: 100,
  ratio: 0.5,  // 阻尼比 < 1
);

// 过阻尼（缓慢到位，无振荡）
final overDamped = SpringDescription.withDampingRatio(
  mass: 1,
  stiffness: 100,
  ratio: 2.0,  // 阻尼比 > 1
);
```

#### 3.12.3 ClampingScrollSimulation — 惯性滚动

```dart
// 模拟列表滚动惯性
final simulation = ClampingScrollSimulation(
  position: currentPosition,
  velocity: flingVelocity,  // 抛出速度
  tolerance: Tolerance.defaultTolerance,
);

_controller.animateWith(simulation);
```

---

### 3.13 动画性能优化

动画性能直接影响用户体验，60fps 是基本目标（每帧 16.67ms）。

#### 3.13.1 RepaintBoundary — 隔离重绘

```dart
// 没有 RepaintBoundary：整个列表随动画重绘
ListView(
  children: [
    AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return Transform.rotate(
          angle: _animation.value,
          child: FlutterLogo(size: 100),
        );
      },
    ),
    // 以下 Widget 也会被重绘！
    Text('我不需要重绘但被牵连了'),
    Container(...),
    // ... 100 个静态 Widget 都被重绘
  ],
)

// 使用 RepaintBoundary：隔离重绘区域
ListView(
  children: [
    // 只有 RepaintBoundary 内部会重绘
    RepaintBoundary(
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, child) {
          return Transform.rotate(
            angle: _animation.value,
            child: FlutterLogo(size: 100),
          );
        },
      ),
    ),
    // 这些 Widget 不会被重绘
    Text('我不受影响'),
    Container(...),
  ],
)
```

#### 3.13.2 AnimatedBuilder 的 child 参数

```dart
// 错误：每次重建整个子树
AnimatedBuilder(
  animation: _animation,
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: Container(
        width: 200,
        height: 200,
        // 这个 Container 每帧都重建！
        child: ExpensiveWidget(),
      ),
    );
  },
);

// 正确：不变的子树放在 child 参数
AnimatedBuilder(
  animation: _animation,
  builder: (context, child) {
    return Transform.scale(
      scale: _animation.value,
      child: child,  // ← 重用不变的子树
    );
  },
  child: Container(              // ← 只构建一次
    width: 200,
    height: 200,
    child: ExpensiveWidget(),
  ),
);
```

#### 3.13.3 避免不必要的 setState

```dart
// 错误：整个 Widget 树重建
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 1),
      vsync: this,
    );

    // 不推荐：每帧 setState 导致整棵树重建
    _controller.addListener(() {
      setState(() {});  // ← 性能杀手！
    });
  }

  @override
  Widget build(BuildContext context) {
    return Transform.scale(
      scale: _controller.value,
      child: HeavyWidget(),  // 不必要的重建
    );
  }
}

// 正确：使用 AnimatedBuilder 精准重建
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 1),
      vsync: this,
    );
    // 不需要 addListener，AnimatedBuilder 自动处理
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.scale(
          scale: _controller.value,
          child: child,
        );
      },
      child: HeavyWidget(),  // 只构建一次
    );
  }
}
```

#### 3.13.4 性能优化清单

| 优化手段 | 说明 | 适用场景 |
|---------|------|---------|
| `RepaintBoundary` | 隔离重绘区域 | 动画 Widget 与静态 Widget 混排 |
| `AnimatedBuilder.child` | 缓存不变子树 | 任何 AnimatedBuilder |
| 避免动画中 `setState` | 用 AnimatedBuilder 替代 | 所有显式动画 |
| `const` 构造函数 | 编译期常量 Widget | 不变 Widget |
| `addRepaintBoundary: true` | CustomPaint 自动隔离 | CustomPaint 默认开启 |
| `shouldRepaint` 返回 `false` | 跳过不必要重绘 | CustomPainter |
| 降低动画频率 | 只在可见时播放 | 列表中大量动画项 |
| 预加载资源 | 避免动画中异步加载 | 图片/Lottie/Rive |
| 使用 `Transform` | 只改变绘制不影响布局 | 位移/旋转/缩放 |

#### 3.13.5 性能分析工具

```dart
// 1. 开启 Performance Overlay
MaterialApp(
  showPerformanceOverlay: true,  // 显示性能面板
  home: MyHomePage(),
);

// 2. 使用 DevTools Timeline
// flutter run 后打开 DevTools → Timeline
// 查看：
//   - UI Thread 耗时（应 < 16ms）
//   - Raster Thread 耗时
//   - 帧率统计

// 3. debugPaintLayerBordersEnabled
// 显示重绘边界，红色边框 = 重绘区域
// 在 main.dart 中临时添加：
// debugRepaintRainbowEnabled = true;
// 可以看到哪些区域频繁重绘（颜色不断变化）
```

#### 3.13.6 列表中动画的优化

```dart
class OptimizedAnimatedList extends StatefulWidget {
  final int itemCount;
  const OptimizedAnimatedList({Key? key, this.itemCount = 50}) : super(key: key);

  @override
  _OptimizedAnimatedListState createState() => _OptimizedAnimatedListState();
}

class _OptimizedAnimatedListState extends State<OptimizedAnimatedList> {
  // 只为可见项创建动画控制器，避免同时运行 50 个动画
  final Map<int, AnimationController> _controllers = {};

  @override
  void dispose() {
    for (final controller in _controllers.values) {
      controller.dispose();
    }
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: widget.itemCount,
      itemBuilder: (context, index) {
        return _AnimatedListItem(
          key: ValueKey(index),
          index: index,
        );
      },
    );
  }
}

class _AnimatedListItem extends StatefulWidget {
  final int index;
  const _AnimatedListItem({Key? key, required this.index}) : super(key: key);

  @override
  __AnimatedListItemState createState() => __AnimatedListItemState();
}

class __AnimatedListItemState extends State<_AnimatedListItem>
    with SingleTickerProviderStateMixin, AutomaticKeepAliveClientMixin {
  late AnimationController _controller;

  @override
  bool get wantKeepAlive => true;  // 保持状态，避免重复创建

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 600),
      vsync: this,
    );

    // 延迟启动，形成交错效果
    Future.delayed(Duration(milliseconds: widget.index * 100), () {
      if (mounted) _controller.forward();
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);  // AutomaticKeepAliveClientMixin 需要

    return RepaintBoundary(  // ← 隔离每个列表项的重绘
      child: SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0.2, 0),
          end: Offset.zero,
        ).animate(CurvedAnimation(
          parent: _controller,
          curve: Curves.easeOut,
        )),
        child: FadeTransition(
          opacity: _controller,
          child: ListTile(
            title: Text('Item ${widget.index}'),
          ),
        ),
      ),
    );
  }
}
```

---

### 3.14 手势与动画联动

手势驱动动画是交互式动画的核心，如拖拽、滑动、捏合等手势实时控制动画参数。

#### 拖拽动画联动

```dart
class DragAnimationDemo extends StatefulWidget {
  @override
  _DragAnimationDemoState createState() => _DragAnimationDemoState();
}

class _DragAnimationDemoState extends State<DragAnimationDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  Offset _dragOffset = Offset.zero;
  double _scale = 1.0;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );

    // 监听动画值用于更新 UI
    _controller.addListener(() {
      setState(() {
        // 弹回原位动画
        _dragOffset = Offset.lerp(_dragOffset, Offset.zero, _controller.value)!;
        _scale = lerpDouble(_scale, 1.0, _controller.value)!;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GestureDetector(
          // 拖拽更新
          onPanUpdate: (details) {
            setState(() {
              _dragOffset += details.delta;
              // 根据拖拽距离计算缩放
              final distance = _dragOffset.distance;
              _scale = (1 - distance / 500).clamp(0.5, 1.0);
            });
          },
          // 松手回弹
          onPanEnd: (details) {
            _controller.forward(from: 0);
          },
          child: Transform.translate(
            offset: _dragOffset,
            child: Transform.scale(
              scale: _scale,
              child: Container(
                width: 150,
                height: 150,
                decoration: BoxDecoration(
                  color: Colors.deepPurple,
                  borderRadius: BorderRadius.circular(20),
                  boxShadow: [
                    BoxShadow(
                      color: Colors.deepPurple.withOpacity(0.3),
                      blurRadius: 20,
                      offset: Offset(0, 10 * _scale),
                    ),
                  ],
                ),
                child: const Center(
                  child: Text('拖我', style: TextStyle(
                    color: Colors.white, fontSize: 20,
                  )),
                ),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

#### 手势 + AnimationController 精确控制

```dart
class SwipeableCard extends StatefulWidget {
  final Widget child;
  final VoidCallback? onSwipedLeft;
  final VoidCallback? onSwipedRight;

  const SwipeableCard({
    Key? key,
    required this.child,
    this.onSwipedLeft,
    this.onSwipedRight,
  }) : super(key: key);

  @override
  _SwipeableCardState createState() => _SwipeableCardState();
}

class _SwipeableCardState extends State<SwipeableCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  double _dragExtent = 0;      // 水平拖拽距离
  double _maxDragExtent = 300; // 触发滑动的最大距离

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _handlePanUpdate(DragUpdateDetails details) {
    setState(() {
      _dragExtent += details.delta.dx;
    });
  }

  void _handlePanEnd(DragEndDetails details) {
    final velocity = details.velocity.pixelsPerSecond.dx;

    if (_dragExtent.abs() > _maxDragExtent * 0.4 || velocity.abs() > 500) {
      // 超过阈值：滑出屏幕
      final target = _dragExtent > 0 ? _maxDragExtent : -_maxDragExtent;
      _animateTo(target, onComplete: () {
        if (target > 0) {
          widget.onSwipedRight?.call();
        } else {
          widget.onSwipedLeft?.call();
        }
        // 重置
        setState(() => _dragExtent = 0);
      });
    } else {
      // 未超阈值：回弹
      _animateTo(0);
    }
  }

  void _animateTo(double target, {VoidCallback? onComplete}) {
    final simulation = SpringSimulation(
      SpringDescription.withDampingRatio(
        mass: 1,
        stiffness: 200,
        ratio: 0.7,  // 欠阻尼，有一点回弹
      ),
      _dragExtent,  // 起始位置
      target,       // 目标位置
      0,            // 初始速度
    );

    _controller.animateWith(simulation).then((_) {
      setState(() => _dragExtent = target);
      onComplete?.call();
    });
  }

  @override
  Widget build(BuildContext context) {
    final rotation = (_dragExtent / 1000);  // 拖拽时轻微旋转

    return GestureDetector(
      onPanUpdate: _handlePanUpdate,
      onPanEnd: _handlePanEnd,
      child: Transform.translate(
        offset: Offset(_dragExtent, 0),
        child: Transform.rotate(
          angle: rotation,
          child: widget.child,
        ),
      ),
    );
  }
}
```

#### 捏合缩放动画

```dart
class PinchZoomDemo extends StatefulWidget {
  @override
  _PinchZoomDemoState createState() => _PinchZoomDemoState();
}

class _PinchZoomDemoState extends State<PinchZoomDemo>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  double _scale = 1.0;
  double _previousScale = 1.0;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 300),
      vsync: this,
    );

    _controller.addListener(() {
      setState(() {
        _scale = lerpDouble(_scale, 1.0, _controller.value)!;
      });
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: GestureDetector(
          onScaleStart: (details) {
            _previousScale = _scale;
            _controller.stop();  // 中断回弹动画
          },
          onScaleUpdate: (details) {
            setState(() {
              _scale = (_previousScale * details.scale).clamp(0.5, 4.0);
            });
          },
          onScaleEnd: (details) {
            // 超出范围时回弹到边界
            if (_scale < 1.0 || _scale > 3.0) {
              _controller.forward(from: 0);
            }
          },
          child: AnimatedBuilder(
            animation: _controller,
            builder: (context, child) {
              return Transform.scale(
                scale: _scale,
                child: child,
              );
            },
            child: Image.asset('assets/photo.jpg', width: 300),
          ),
        ),
      ),
    );
  }
}
```

---

### 3.15 页面切换动画自定义

Flutter 的 `PageRouteBuilder` 允许自定义页面切换过渡动画。

#### 基本自定义过渡

```dart
// 自定义淡入淡出
class FadePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  FadePageRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 500),
          reverseTransitionDuration: const Duration(milliseconds: 300),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return FadeTransition(
              opacity: CurvedAnimation(
                parent: animation,
                curve: Curves.easeOut,
              ),
              child: child,
            );
          },
        );
}

// 使用
Navigator.push(context, FadePageRoute(page: NextPage()));
```

#### 常用页面切换动画

```dart
/// 1. 滑动过渡
class SlidePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;
  final AxisDirection direction;  // 滑动方向

  SlidePageRoute({
    required this.page,
    this.direction = AxisDirection.left,
  }) : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 400),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            Offset begin;
            switch (direction) {
              case AxisDirection.up:
                begin = const Offset(0, 1);
                break;
              case AxisDirection.down:
                begin = const Offset(0, -1);
                break;
              case AxisDirection.right:
                begin = const Offset(-1, 0);
                break;
              case AxisDirection.left:
              default:
                begin = const Offset(1, 0);
            }
            return SlideTransition(
              position: Tween<Offset>(begin: begin, end: Offset.zero).animate(
                CurvedAnimation(parent: animation, curve: Curves.easeOutCubic),
              ),
              child: child,
            );
          },
        );
}

/// 2. 缩放过渡
class ScalePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  ScalePageRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 500),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            return ScaleTransition(
              scale: CurvedAnimation(
                parent: animation,
                curve: Curves.elasticOut,
              ),
              child: child,
            );
          },
        );
}

/// 3. 旋转 + 缩放过渡
class RotateScalePageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;

  RotateScalePageRoute({required this.page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 600),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final curvedAnimation = CurvedAnimation(
              parent: animation,
              curve: Curves.easeInOutBack,
            );
            return ScaleTransition(
              scale: Tween<double>(begin: 0.0, end: 1.0).animate(curvedAnimation),
              child: RotationTransition(
                turns: Tween<double>(begin: -0.1, end: 0.0).animate(curvedAnimation),
                child: child,
              ),
            );
          },
        );
}

/// 4. 共享元素过渡（Hero 以外的方式）
class SharedAxisPageRoute<T> extends PageRouteBuilder<T> {
  final Widget page;
  final Axis axis;  // 共享轴方向

  SharedAxisPageRoute({required this.page, this.axis = Axis.horizontal})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 400),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            // 当前页面退出动画
            final exitAnimation = Tween<Offset>(
              begin: Offset.zero,
              end: axis == Axis.horizontal
                  ? const Offset(-0.1, 0)
                  : const Offset(0, -0.1),
            ).animate(CurvedAnimation(
              parent: animation,
              curve: Curves.easeOut,
            ));

            // 新页面进入动画
            final enterAnimation = Tween<Offset>(
              begin: axis == Axis.horizontal
                  ? const Offset(0.1, 0)
                  : const Offset(0, 0.1),
              end: Offset.zero,
            ).animate(CurvedAnimation(
              parent: animation,
              curve: Curves.easeOut,
            ));

            return AnimatedBuilder(
              animation: animation,
              builder: (context, _) {
                return Stack(
                  children: [
                    // 旧页面淡出 + 位移
                    if (animation.value < 1.0)
                      FadeTransition(
                        opacity: Tween<double>(begin: 1, end: 0).animate(animation),
                        child: SlideTransition(
                          position: exitAnimation,
                          child: widget,  // 这里简化处理
                        ),
                      ),
                    // 新页面淡入 + 位移
                    FadeTransition(
                      opacity: animation,
                      child: SlideTransition(
                        position: enterAnimation,
                        child: child,
                      ),
                    ),
                  ],
                );
              },
            );
          },
        );
}
```

#### 使用 PageTransitionsTheme 全局配置

```dart
MaterialApp(
  theme: ThemeData(
    pageTransitionsTheme: PageTransitionsTheme(
      builders: {
        // 根据平台配置不同的过渡动画
        TargetPlatform.android: ZoomPageTransitionsBuilder(),  // Android 默认缩放
        TargetPlatform.iOS: CupertinoPageTransitionsBuilder(), // iOS 默认滑动
        // 自定义平台
        TargetPlatform.windows: FadeUpwardsPageTransitionsBuilder(),
      },
    ),
  ),
  home: MyApp(),
);
```

#### 内置 PageTransitionsBuilder 一览

| Builder | 效果 | 适用平台 |
|---------|------|---------|
| `FadeUpwardsPageTransitionsBuilder` | 淡入 + 上移 | Material 默认 (旧) |
| `ZoomPageTransitionsBuilder` | 缩放 + 淡入 | Material 默认 (新) |
| `CupertinoPageTransitionsBuilder` | 右滑入 | iOS |
| `OpenUpwardPageTransitionsBuilder` | 底部弹出 | Android O 风格 |

---

## 4. 面试题精选

### 题目 1：隐式动画和显式动画有什么区别？如何选择？

**答案**：

| 维度 | 隐式动画 | 显式动画 |
|------|---------|---------|
| 控制器管理 | 框架自动创建和销毁 | 开发者手动创建和销毁 |
| 控制粒度 | 只能正向播放一次 | 可暂停、反向、重复、跳到任意位置 |
| 代码复杂度 | 低，只需改属性值 | 中高，需要管理 AnimationController 生命周期 |
| 可复用性 | 固定场景 | 可封装任意动画逻辑 |
| 状态监听 | 不支持 | 支持 addStatusListener |

**选择原则**：
- 属性简单过渡（颜色/大小/透明度变化）→ 隐式动画
- 需要暂停/反向/重复 → 显式动画
- 交错动画 → 显式动画
- 一次性过渡不需要精细控制 → 隐式动画
- 自定义过渡逻辑 → `TweenAnimationBuilder`（隐式）或 `AnimatedBuilder`（显式）

### 题目 2：AnimationController 为什么需要 vsync？Ticker 的作用是什么？

**答案**：

**Ticker** 是 Flutter 帧回调的调度器，它与屏幕刷新率同步（通常 60Hz），每帧触发一次回调。Ticker 只在屏幕可见时运行，当应用退到后台时自动暂停，避免无意义的 CPU 消耗。

**vsync** 是 TickerProvider 的实现，为 AnimationController 提供 Ticker 实例。它确保动画只在屏幕刷新时更新，避免动画运行快于屏幕刷新率导致的浪费。

```dart
// SingleTickerProviderStateMixin: 只创建一个 Ticker，单个动画
class _SingleAnimState extends State with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  @override
  void initState() {
    _controller = AnimationController(vsync: this, ...);
  }
}

// TickerProviderStateMixin: 可创建多个 Ticker，多动画
class _MultiAnimState extends State with TickerProviderStateMixin {
  late AnimationController _ctrl1;
  late AnimationController _ctrl2;
  @override
  void initState() {
    _ctrl1 = AnimationController(vsync: this, ...);
    _ctrl2 = AnimationController(vsync: this, ...);
  }
}
```

**不提供 vsync 的后果**：AnimationController 无法创建，运行时报错。在测试中可以使用 `AnimationController.unbounded()` 绕过 vsync 要求。

### 题目 3：CustomPainter 的 shouldRepaint 什么时候返回 true？什么时候返回 false？

**答案**：

`shouldRepaint` 决定是否调用 `paint()` 方法重新绘制。这是性能关键点：

```dart
@override
bool shouldRepaint(covariant MyPainter oldDelegate) {
  // 返回 true: 重新调用 paint()
  // 返回 false: 跳过重绘，使用上一次的结果

  // 正确做法：比较属性是否变化
  return oldDelegate.progress != progress ||
         oldDelegate.color != color;
}
```

**规则**：
- 当 `shouldRepaint` 返回 `false` 时，Flutter 直接复用上一次绘制的缓存，不调用 `paint()`
- 当 Widget 树重建（`setState` 等），即使 `shouldRepaint` 返回 `false`，如果 `CustomPaint` 本身是新建对象，仍然可能触发 `paint()`
- **常见错误**：始终返回 `true`，导致每帧都重绘，性能浪费
- **常见错误**：始终返回 `false`，导致属性变化后画面不更新
- **最佳实践**：将 painter 属性保存为实例变量，在属性变化时创建新的 painter 对象，让 `shouldRepaint` 精确比较

### 题目 4：如何实现一个交错动画（Staggered Animation）？请描述核心思路。

**答案**：

交错动画的核心是使用**一个 AnimationController** 配合 **多个 Interval + CurvedAnimation**，让不同属性在不同时间段内动画化。

**核心思路**：
1. 创建一个 `AnimationController`，duration 设为整个交错动画的总时长
2. 为每个需要交错的属性创建 `CurvedAnimation`，使用 `Interval` 指定其在总时间轴上的生效区间
3. 用 `AnimatedBuilder` 监听 controller，在 builder 中根据各属性的 animation 值更新 UI

```dart
// 关键代码结构
_controller = AnimationController(duration: Duration(seconds: 2), vsync: this);

// 属性 A：0%~30%
_animA = Tween(begin: 0, end: 1).animate(
  CurvedAnimation(parent: _controller,
    curve: Interval(0.0, 0.3, curve: Curves.easeOut)),
);

// 属性 B：20%~60%（与 A 有 10% 重叠）
_animB = Tween(begin: 0, end: 1).animate(
  CurvedAnimation(parent: _controller,
    curve: Interval(0.2, 0.6, curve: Curves.easeOut)),
);

// 属性 C：50%~100%
_animC = Tween(begin: 0, end: 1).animate(
  CurvedAnimation(parent: _controller,
    curve: Interval(0.5, 1.0, curve: Curves.easeInOut)),
);
```

**要点**：Interval 的 start 和 end 范围是 `[0.0, 1.0]`，多个区间可以有重叠，重叠区间内多个属性同时变化。

### 题目 5：Hero 动画的原理是什么？有哪些限制？

**答案**：

**原理**：
1. Navigator 在路由切换时，检查源页面和目标页面中是否有相同 `tag` 的 Hero Widget
2. 如果有，创建一个 `OverlayEntry` 覆盖在页面最上层
3. 源 Hero 的 RenderObject 计算出在全局坐标中的位置和大小
4. 目标 Hero 同样计算全局位置和大小
5. 在 Overlay 层创建一个"飞行"的 Hero 副本，从源位置/大小动画到目标位置/大小
6. 同时源位置和目标位置的原始 Hero 变为不可见
7. 动画结束后，移除 Overlay，目标 Hero 显示

**限制**：
1. **tag 必须全局唯一**（同一页面中不能有重复 tag）
2. **只支持两个页面之间**，不能跨多页面
3. **性能开销**：动画期间会在 Overlay 层绘制额外的 Widget，复杂布局可能导致掉帧
4. **不支持嵌套 Hero**：Hero Widget 内不能再包含 Hero
5. **自定义路由兼容性**：自定义 PageRoute 需要正确处理 Overlay
6. **ModalBarrier 阻挡**：在弹出式路由（如 Dialog）中 Hero 动画可能异常

### 题目 6：Flutter 中如何优化动画性能？请列举至少 5 种方法。

**答案**：

1. **使用 RepaintBoundary 隔离重绘**：将动画 Widget 包裹在 `RepaintBoundary` 中，避免动画重绘时牵连周围静态 Widget

2. **AnimatedBuilder 的 child 参数**：将不变的子 Widget 放在 `child` 参数中，builder 中通过参数接收，避免每帧重建

3. **避免动画中调用 setState**：使用 `AnimatedBuilder` 代替 `addListener + setState`，精准重建最小范围

4. **CustomPainter.shouldRepaint 精确判断**：只在属性真正变化时返回 `true`，避免无意义的重绘

5. **使用 Transform 代替布局变化**：`Transform` 只在绘制层变换（GPU 加速），不影响布局计算。用 `Transform.scale` 代替改变 `width/height`

6. **预加载资源**：Lottie/Rive/图片在动画开始前预加载，避免动画过程中异步加载导致卡顿

7. **列表中控制动画数量**：使用 `AutomaticKeepAliveClientMixin` 保持状态，只对可见项播放动画

8. **减少 Shadow 使用**：`BoxShadow` 的 `blurRadius` 越大越耗性能，动画中尽量避免或使用预渲染阴影图

9. **使用 const 构造函数**：不变的 Widget 使用 `const` 修饰，编译期确定，跳过重建

10. **性能分析**：开启 `showPerformanceOverlay: true` 或 DevTools Timeline，定位瓶颈

### 题目 7：Canvas 绘制文字的方法是什么？有什么注意事项？

**答案**：

Canvas 绘制文字需要通过 `TextPainter` 构建 `Paragraph` 对象：

```dart
@override
void paint(Canvas canvas, Size size) {
  // 1. 创建 TextSpan
  final textSpan = TextSpan(
    text: 'Hello Flutter',
    style: TextStyle(
      color: Colors.black,
      fontSize: 24,
      fontWeight: FontWeight.bold,
    ),
  );

  // 2. 创建 TextPainter 并布局
  final textPainter = TextPainter(
    text: textSpan,
    textDirection: TextDirection.ltr,  // ← 必须指定
    textAlign: TextAlign.center,
    maxLines: 1,
  )..layout(
    maxWidth: size.width,  // ← 必须调用 layout()
  );

  // 3. 绘制到 Canvas
  textPainter.paint(
    canvas,
    Offset(
      (size.width - textPainter.width) / 2,   // 水平居中
      (size.height - textPainter.height) / 2,  // 垂直居中
    ),
  );
}
```

**注意事项**：
- `textDirection` 必须指定，否则报错
- 必须调用 `layout()` 后才能获取 `width/height` 和绘制
- `maxWidth` 影响文字换行，不指定则默认无限宽不换行
- 富文本使用 `TextSpan(children: [...])` 嵌套
- 性能：频繁创建 `TextPainter` 开销大，应在属性不变时缓存

### 题目 8：请解释 Flutter 动画中 Tween、CurvedAnimation、AnimationController 三者的关系，并画出数据流图。

**答案**：

**三者关系**：

- `AnimationController`：动画的驱动源，在 `[lowerBound, upperBound]`（默认 `[0, 1]`）之间产生线性变化的值，由 Ticker 驱动每帧更新
- `CurvedAnimation`：包装 Controller 的输出，通过 Curve 将线性值映射为非线性值（如 easeOut），但值域仍为 `[0, 1]`
- `Tween`：将 `[0, 1]` 的值映射到目标值域（如 `[0, 200]`、`[blue, red]`），做最终的 `lerp` 插值

**数据流图**：

```
Ticker (每帧回调)
  │
  ▼
AnimationController          帧数 → 0~1 线性值
  │ value: 0.0 ──────────────► 1.0
  │
  ▼
CurvedAnimation              0~1 线性值 → 0~1 非线性值
  │ value: 0.0 ──╭───╮──────► 1.0  (经过 Curves.easeOut 映射)
  │
  ▼
Tween                        0~1 → 目标值域
  │ value: begin ────────────► end  (如 0 → 200, blue → red)
  │
  ▼
Widget (AnimatedBuilder)     读取最终值，更新 UI
```

**代码体现**：

```dart
// 组合顺序：Controller → Curve → Tween
_controller = AnimationController(vsync: this, duration: Duration(seconds: 1));

_curvedAnimation = CurvedAnimation(parent: _controller, curve: Curves.easeOut);

_tweenAnimation = Tween<double>(begin: 0, end: 200).animate(_curvedAnimation);

// 等价链式写法
_animation = Tween<double>(begin: 0, end: 200).animate(
  CurvedAnimation(
    parent: _controller,
    curve: Curves.easeOut,
  ),
);

// _animation.value 就是从 0 到 200，按 easeOut 曲线变化的值
```

**关键理解**：三者是装饰器模式（Decorator Pattern），逐层包装。AnimationController 是核心，CurvedAnimation 和 Tween 是可选的中间层。隐式动画内部也是这个结构，只是框架帮你管理了。

---

## 5. 总结

### 知识脉络回顾

```
Flutter 动画与自定义绘制
├── 动画体系
│   ├── 隐式动画：AnimatedXxx → 简单属性过渡
│   ├── 显式动画：Controller + Tween + Curve → 精细控制
│   ├── Hero 动画：共享元素过渡
│   ├── 交错动画：Interval 编排多属性
│   └── 物理动画：Spring/Damping 自然运动
├── 自定义绘制
│   ├── CustomPainter + Canvas API → 任意 2D 图形
│   ├── Paint 配置：颜色/渐变/模糊/混合
│   ├── Path 路径：直线/贝塞尔/弧线
│   └── 实战：进度环/图表/波浪/签名板
├── 第三方动画
│   ├── Lottie：JSON 播放型动画
│   └── Rive：交互型状态机动画
├── 高级技巧
│   ├── 手势联动：拖拽/滑动/捏合驱动动画
│   ├── 页面过渡：PageRouteBuilder 自定义
│   └── 性能优化：RepaintBoundary/AnimatedBuilder.child/shouldRepaint
└── 核心原则
    ├── 简单场景用隐式，复杂场景用显式
    ├── 性能优先：最小重建范围 + 缓存不变子树
    └── 自然感：合理使用 Curves + 物理动画
```

### 选择决策速查

| 需求 | 推荐方案 |
|------|---------|
| 属性值变化过渡 | AnimatedXxx (隐式) |
| 自定义 Tween 隐式动画 | TweenAnimationBuilder |
| 需要暂停/反向/重复 | AnimationController + AnimatedBuilder |
| 多属性交错编排 | Interval + CurvedAnimation |
| 页面间共享元素 | Hero |
| 任意 2D 图形 | CustomPainter |
| 播放 AE 动画 | Lottie |
| 交互式动画 | Rive |
| 自然的物理运动 | SpringSimulation |
| 拖拽/手势驱动 | GestureDetector + AnimationController |
| 自定义页面切换 | PageRouteBuilder |

### 相关链接

- [[Flutter核心与Widget体系]] — Flutter Widget 树与生命周期基础
- [[Flutter布局与样式]] — 布局系统与样式体系，动画常与之配合
- [[CSS动画与过渡]] — Web 端动画对比参考，理解跨平台动画设计差异

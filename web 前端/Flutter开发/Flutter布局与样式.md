---
tags:
  - Web前端
  - Flutter
  - 布局
  - 样式
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Flutter布局与样式

## What — 是什么

> Flutter 布局与样式是 Flutter 框架中用于构建 UI 界面的核心系统，包括布局原理、组件排列、尺寸控制、装饰效果和主题管理。Flutter 采用"一切皆 Widget"的声明式 UI 范式，通过 Widget 树的组合嵌套来实现各种布局与视觉效果。

### 核心概念一览

```
Flutter 布局与样式体系
├── 布局原理
│   ├── 约束向下传递（Constraints go down）
│   ├── 尺寸向上返回（Sizes go up）
│   └── 父级设置位置（Parent sets position）
├── 布局组件
│   ├── Flex 布局 ─── Row / Column
│   ├── 弹性布局 ─── Expanded / Flexible / SizedBox / Spacer
│   ├── 层叠布局 ─── Stack / Positioned / Align / Center / FittedBox
│   ├── 流式布局 ─── Wrap
│   ├── 容器组件 ─── Container / Padding / ConstrainedBox
│   ├── 列表滚动 ─── ListView / GridView / CustomScrollView
│   └── 自定义布局 ─── CustomMultiChildLayout / Flow
├── 样式系统
│   ├── 文本样式 ─── TextStyle
│   ├── 盒装饰 ─── BoxDecoration / ShapeDecoration
│   ├── 颜色渐变 ─── Color / Gradient / BoxShadow
│   ├── 图标图片 ─── Icon / Image / DecorationImage
│   └── 主题系统 ─── ThemeData / TextTheme / ColorScheme
└── 响应式与适配
    ├── MediaQuery / LayoutBuilder / OrientationBuilder
    └── SafeArea / 屏幕适配方案
```

### Flutter 布局原理（最核心）

Flutter 布局遵循**三条铁律**，理解这三条规则是掌握 Flutter 布局的关键：

```
┌─────────────────────────────────────────────────┐
│               Flutter 布局三定律                  │
│                                                   │
│  1. 约束向下传递  Parent → Child                  │
│     父 Widget 向子 Widget 传递约束（最小/最大宽高）│
│                                                   │
│  2. 尺寸向上返回  Child → Parent                  │
│     子 Widget 在父约束范围内确定自身尺寸           │
│                                                   │
│  3. 父级设置位置  Parent 决定 Child 位置           │
│     父 Widget 决定子 Widget 在自身空间中的位置     │
│                                                   │
│  ┌──────────────────────┐                        │
│  │   Parent Widget      │                        │
│  │  ┌────────────────┐  │                        │
│  │  │  Child Widget  │  │                        │
│  │  │   ↕ 约束/尺寸  │  │                        │
│  │  └────────────────┘  │                        │
│  │   ← 位置由父级决定 →  │                        │
│  └──────────────────────┘                        │
└─────────────────────────────────────────────────┘
```

**约束（BoxConstraints）的四种模式：**

| 模式 | minWidth | maxWidth | minHeight | maxHeight | 典型场景 |
|------|----------|----------|-----------|-----------|----------|
| 紧凑约束 | = maxWidth | = minWidth | = maxHeight | = minHeight | 固定尺寸（SizedBox） |
| 严格约束 | 0 | ∞ | 0 | ∞ | 尽可能大（ListView） |
| 无限约束 | 0 | ∞ | 0 | ∞ | 滚动方向 |
| 有限约束 | >0 | <∞ | >0 | <∞ | 普通父容器 |

### 与 Web CSS 布局的对应关系

理解 Flutter 布局最快的方式是建立与 CSS 的映射：

| CSS 概念 | Flutter 对应 | 说明 |
|----------|-------------|------|
| `display: flex` | `Row` / `Column` | Flex 布局 |
| `flex-direction: row` | `Row` | 水平排列 |
| `flex-direction: column` | `Column` | 垂直排列 |
| `justify-content` | `MainAxisAlignment` | 主轴对齐 |
| `align-items` | `CrossAxisAlignment` | 交叉轴对齐 |
| `flex: 1` | `Expanded` | 占据剩余空间 |
| `flex-grow / flex-shrink` | `Flexible` | 弹性伸缩 |
| `position: absolute` | `Positioned` + `Stack` | 绝对定位 |
| `position: relative` | `Stack` | 相对定位容器 |
| `flex-wrap: wrap` | `Wrap` | 换行布局 |
| `padding` | `Padding` / Container.padding | 内边距 |
| `margin` | Container.margin | 外边距 |
| `width / height` | `SizedBox` / `ConstrainedBox` | 尺寸约束 |
| `box-shadow` | `BoxShadow` | 阴影 |
| `background` | `BoxDecoration` | 背景 |
| `border-radius` | `BorderRadius` | 圆角 |
| `overflow: scroll` | `ListView` / `SingleChildScrollView` | 滚动 |
| `@media` | `MediaQuery` / `LayoutBuilder` | 响应式 |
| `CSS Grid` | `GridView` / `CustomMultiChildLayout` | 网格布局 |
| `color` | `Color` | 颜色 |
| `font-size` | `TextStyle.fontSize` | 字号 |
| `z-index` | Stack 子元素顺序 | 层叠顺序 |

## Why — 为什么

### 为什么 Flutter 不用 CSS 而用 Widget 嵌套做布局

```
CSS 布局方式                          Flutter Widget 嵌套方式
┌─────────────────┐                  ┌─────────────────┐
│ <div class=     │                  │ Container(       │
│   "container    │                  │   padding: ...,  │
│    p-4          │  ──对应──>       │   margin: ...,   │
│    m-2          │                  │   decoration: ..,│
│    shadow       │                  │   child: Padding(│
│    rounded"     │                  │     child: Text()│
│ >               │                  │   )              │
│   <p>文本</p>   │                  │ )                │
│ </div>          │                  └─────────────────┘
└─────────────────┘
```

**Flutter Widget 嵌套的优势：**

1. **组合优于继承**：每个 Widget 职责单一，通过组合实现复杂效果
2. **类型安全**：编译期检查，减少运行时错误
3. **可预测性**：布局结果完全由 Widget 树决定，没有 CSS 优先级和层叠上下文的混乱
4. **热重载友好**：修改 Widget 属性立即生效
5. **跨平台一致**：Flutter 自绘引擎保证各平台表现一致

**Flutter 布局的优势与局限：**

| 维度 | 优势 | 局限 |
|------|------|------|
| 性能 | 自绘引擎，无平台差异 | 嵌套过深影响可读性 |
| 一致性 | 跨平台像素级一致 | 无法利用原生布局能力 |
| 灵活性 | CustomPaint 可绘制任意图形 | 学习曲线较陡 |
| 开发效率 | 热重载秒级生效 | 复杂布局嵌套层级多 |
| 调试 | Flutter Inspector 可视化 | 布局溢出错误信息不够直观 |

### 为什么需要理解约束传递机制

```dart
// 典型陷阱：Column 内部使用 ListView 会导致无限高度
Column(
  children: [
    Text('标题'),
    ListView.builder(  // ❌ 报错：无限高度
      itemCount: 100,
      itemBuilder: (ctx, i) => Text('item $i'),
    ),
  ],
)

// 原因：Column 给子组件传递的是无限高度约束
// ListView 试图占满无限高度 → 布局崩溃

// 解决方案1：用 Expanded 限制 ListView 的约束
Column(
  children: [
    Text('标题'),
    Expanded(  // ✅ Expanded 截断无限约束，给出有限剩余空间
      child: ListView.builder(
        itemCount: 100,
        itemBuilder: (ctx, i) => Text('item $i'),
      ),
    ),
  ],
)

// 解决方案2：给 ListView 指定 shrinkWrap
Column(
  children: [
    Text('标题'),
    ListView.builder(
      shrinkWrap: true,  // ✅ ListView 根据内容自适应高度
      physics: NeverScrollableScrollPhysics(), // 禁用自身滚动
      itemCount: 100,
      itemBuilder: (ctx, i) => Text('item $i'),
    ),
  ],
)
```

## How — 怎么用

### 1. Flex 布局：Row 与 Column

Row 和 Column 是 Flutter 中最常用的布局组件，对应 CSS 的 `display: flex`。

**MainAxisAlignment（主轴对齐）— 对应 CSS justify-content：**

```
MainAxisAlignment.start     MainAxisAlignment.center    MainAxisAlignment.end
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│■■■               │       │     ■■■          │       │              ■■■ │
└──────────────────┘       └──────────────────┘       └──────────────────┘

MainAxisAlignment.spaceBetween  MainAxisAlignment.spaceAround  MainAxisAlignment.spaceEvenly
┌──────────────────┐           ┌──────────────────┐           ┌──────────────────┐
│■    ■    ■    ■  │           │ ■  ■  ■  ■      │           │  ■  ■  ■  ■     │
└──────────────────┘           └──────────────────┘           └──────────────────┘
```

**CrossAxisAlignment（交叉轴对齐）— 对应 CSS align-items：**

```
CrossAxisAlignment.start    CrossAxisAlignment.center   CrossAxisAlignment.end
┌────────────────────┐     ┌────────────────────┐     ┌────────────────────┐
│████                │     │                    │     │                ████│
│████  ████          │     │████  ████          │     │████  ████      ████│
│████                │     │                    │     │                ████│
└────────────────────┘     └────────────────────┘     └────────────────────┘

CrossAxisAlignment.stretch
┌────────────────────┐
│████████████████████│
│████████████████████│
│████████████████████│
└────────────────────┘
```

**MainAxisSize：**

| 值 | 说明 | CSS 对应 |
|----|------|----------|
| `MainAxisSize.max` | 占满主轴方向所有空间（默认） | `width: 100%` / `height: 100%` |
| `MainAxisSize.min` | 仅占内容所需空间 | `width: fit-content` / `height: fit-content` |

**完整示例 — 登录页面布局：**

```dart
import 'package:flutter/material.dart';

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 32),
          // Column 主轴垂直排列
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,   // 垂直居中
            crossAxisAlignment: CrossAxisAlignment.stretch, // 水平拉伸
            mainAxisSize: MainAxisSize.max,               // 占满全高
            children: [
              // Logo 区域
              const FlutterLogo(size: 80),
              const SizedBox(height: 48), // 间距

              // 用户名输入框
              TextField(
                decoration: InputDecoration(
                  hintText: '用户名',
                  prefixIcon: const Icon(Icons.person),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),
              const SizedBox(height: 16),

              // 密码输入框
              TextField(
                obscureText: true,
                decoration: InputDecoration(
                  hintText: '密码',
                  prefixIcon: const Icon(Icons.lock),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
              ),
              const SizedBox(height: 24),

              // 登录按钮 — 拉伸占满宽度（因为 crossAxisAlignment 是 stretch）
              ElevatedButton(
                onPressed: () {},
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(12),
                  ),
                ),
                child: const Text('登录', style: TextStyle(fontSize: 18)),
              ),
              const SizedBox(height: 16),

              // 底部链接 — Row 水平排列
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  TextButton(
                    onPressed: () {},
                    child: const Text('忘记密码？'),
                  ),
                  TextButton(
                    onPressed: () {},
                    child: const Text('注册账号'),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**Row/Column 嵌套实战 — 商品卡片：**

```dart
class ProductCard extends StatelessWidget {
  final String title;
  final String subtitle;
  final double price;
  final String imageUrl;

  const ProductCard({
    super.key,
    required this.title,
    required this.subtitle,
    required this.price,
    required this.imageUrl,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      child: Padding(
        padding: const EdgeInsets.all(12),
        // 外层 Row：图片 + 文字信息
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 左侧图片
            ClipRRect(
              borderRadius: BorderRadius.circular(12),
              child: Image.network(
                imageUrl,
                width: 100,
                height: 100,
                fit: BoxFit.cover,
              ),
            ),
            const SizedBox(width: 12),

            // 右侧信息 — Column 垂直排列
            Expanded( // Expanded 让文字区占满剩余宽度
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    title,
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 4),
                  Text(
                    subtitle,
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.grey[600],
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const SizedBox(height: 8),
                  // 底部价格行
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceBetween,
                    children: [
                      Text(
                        '¥$price',
                        style: const TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                          color: Colors.red,
                        ),
                      ),
                      IconButton(
                        onPressed: () {},
                        icon: const Icon(Icons.add_shopping_cart),
                        color: Colors.blue,
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 2. 弹性布局：Expanded / Flexible / SizedBox / Spacer

```
Expanded vs Flexible 的区别：

Flexible(fit: FlexFit.loose)   ── 子组件可以选择不占满剩余空间
Flexible(fit: FlexFit.tight)   ── 等同于 Expanded，必须占满
Expanded = Flexible(fit: FlexFit.tight)

┌────────────────────────────────────────────────┐
│  Fixed  │      Expanded      │    Flexible     │
│  100px  │  占满剩余空间(强制) │  最多剩余空间   │
│         │                    │  (可自行决定)    │
└────────────────────────────────────────────────┘
```

**Expanded 的 flex 参数 — 对应 CSS flex-grow：**

```dart
Row(
  children: [
    Expanded(
      flex: 1, // 占 1 份
      child: Container(color: Colors.red, height: 50),
    ),
    Expanded(
      flex: 2, // 占 2 份（是前者的两倍宽）
      child: Container(color: Colors.green, height: 50),
    ),
    Expanded(
      flex: 1, // 占 1 份
      child: Container(color: Colors.blue, height: 50),
    ),
  ],
)
// 结果比例：红:绿:蓝 = 1:2:1
```

**Expanded vs Flexible 实战对比：**

```dart
class ExpandedVsFlexibleDemo extends StatelessWidget {
  const ExpandedVsFlexibleDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text('Expanded — 强制占满剩余空间', style: TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        Row(
          children: [
            const Icon(Icons.star, color: Colors.amber),
            Expanded(  // ✅ 强制占满，文字多行截断
              child: Text(
                '这是一段很长的文本内容，Expanded会强制占满剩余空间，文本会自动换行或截断。',
                maxLines: 2,
                overflow: TextOverflow.ellipsis,
              ),
            ),
            const Icon(Icons.arrow_forward),
          ],
        ),
        const Divider(height: 32),

        const Text('Flexible — 可选占满，尊重子组件自身尺寸', style: TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        Row(
          children: [
            const Icon(Icons.star, color: Colors.amber),
            Flexible(  // Flexible 允许子组件保持自然尺寸
              child: Text(
                '这是一段文本，Flexible允许文本按自身需求占用空间，但如果空间不足会被压缩。',
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
            ),
            const Icon(Icons.arrow_forward),
          ],
        ),
        const Divider(height: 32),

        const Text('Spacer — 在 Row/Column 中插入空白', style: TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 8),
        Row(
          children: [
            const Text('左侧'),
            const Spacer(),  // 等价于 Expanded(child: SizedBox.shrink())
            const Text('右侧'),
          ],
        ),
      ],
    );
  }
}
```

### 3. Stack 层叠布局

Stack 是 Flutter 的绝对定位方案，子 Widget 可以重叠放置，对应 CSS `position: absolute` + `position: relative` 的组合。

```
Stack 布局模型：
┌─────────────────────────────┐
│  Stack                      │
│  ┌───────────────────────┐  │
│  │  底层 Widget (第一项) │  │  ← 先放的在底层
│  │  ┌─────────────────┐  │  │
│  │  │ 中层 Widget     │  │  │
│  │  │ ┌─────────────┐ │  │  │
│  │  │ │ 顶层 Widget │ │  │  │  ← 后放的在顶层
│  │  │ └─────────────┘ │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└─────────────────────────────┘

Stack.alignment 控制未定位子组件的对齐方式
Positioned 控制子组件的绝对位置（top/right/bottom/left）
```

**Stack 关键属性：**

| 属性 | 说明 | 默认值 |
|------|------|--------|
| `alignment` | 未用 Positioned 包裹的子组件的对齐方式 | `AlignmentDirectional.topStart` |
| `fit` | 未定位子组件如何适配 Stack 尺寸 | `StackFit.loose` |
| `clipBehavior` | 超出 Stack 边界的子组件是否裁剪 | `Clip.hardEdge` |

**实战 — 图片卡片带角标：**

```dart
class ImageCardWithBadge extends StatelessWidget {
  const ImageCardWithBadge({super.key});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: 200,
      height: 260,
      child: Stack(
        clipBehavior: Clip.antiAlias, // 裁剪超出部分（圆角生效）
        children: [
          // 底层：图片
          Positioned.fill(
            child: ClipRRect(
              borderRadius: BorderRadius.circular(16),
              child: Image.network(
                'https://picsum.photos/200/260',
                fit: BoxFit.cover,
              ),
            ),
          ),

          // 底部渐变遮罩 + 文字
          Positioned(
            left: 0,
            right: 0,
            bottom: 0,
            child: Container(
              padding: const EdgeInsets.all(12),
              decoration: const BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                  colors: [Colors.transparent, Colors.black54],
                ),
              ),
              child: const Text(
                '风景摄影',
                style: TextStyle(color: Colors.white, fontSize: 18),
              ),
            ),
          ),

          // 右上角角标
          Positioned(
            top: 8,
            right: 8,
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
              decoration: BoxDecoration(
                color: Colors.red,
                borderRadius: BorderRadius.circular(12),
              ),
              child: const Text(
                'HOT',
                style: TextStyle(color: Colors.white, fontSize: 12),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Align / Center / FittedBox：**

```dart
// Align — 将子组件对齐到指定位置
Align(
  alignment: Alignment.bottomRight,
  child: FlutterLogo(size: 50),
)

// Center — Align 的特例，等价于 Align(alignment: Alignment.center)
Center(
  child: FlutterLogo(size: 50),
)

// FittedBox — 根据适配方式缩放子组件
FittedBox(
  fit: BoxFit.contain, // 保持比例，完整显示
  child: Text('缩放文本'),
)

// BoxFit 各种模式：
// ┌──────────────────────────────────────────────┐
// │ BoxFit.fill       → 拉伸填满，可能变形       │
// │ BoxFit.contain    → 保持比例，完整显示       │
// │ BoxFit.cover      → 保持比例，裁剪填满       │
// │ BoxFit.fitWidth   → 宽度填满，高度按比例     │
// │ BoxFit.fitHeight  → 高度填满，宽度按比例     │
// │ BoxFit.none       → 原始大小，不缩放         │
// │ BoxFit.scaleDown  → 只缩小不放大             │
// └──────────────────────────────────────────────┘
```

### 4. Wrap 流式布局

Wrap 是 Row/Column 的"自动换行"版本，对应 CSS 的 `flex-wrap: wrap`。

```dart
class TagWrapDemo extends StatelessWidget {
  const TagWrapDemo({super.key});

  final List<String> tags = const [
    'Flutter', 'Dart', 'Widget', '布局', '样式',
    '动画', '状态管理', '路由', '网络请求', '本地存储',
    '平台通道', '自定义绘制', '手势', '主题', '国际化',
  ];

  @override
  Widget build(BuildContext context) {
    return Wrap(
      direction: Axis.horizontal,    // 排列方向，Axis.vertical 为纵向
      alignment: WrapAlignment.start, // 主轴对齐
      spacing: 8,                     // 水平间距
      runAlignment: WrapAlignment.start, // 行对齐
      runSpacing: 8,                  // 行间距（垂直间距）
      children: tags.map((tag) {
        return Chip(
          label: Text(tag),
          avatar: CircleAvatar(
            backgroundColor: Colors.blue.shade100,
            child: Text(tag[0]),
          ),
          onDeleted: () {},
        );
      }).toList(),
    );
  }
}
```

**Wrap 关键属性：**

| 属性 | 说明 | 对应 CSS |
|------|------|----------|
| `direction` | 排列方向 | `flex-direction` |
| `alignment` | 主轴对齐 | `justify-content` |
| `spacing` | 主轴方向子元素间距 | `gap` |
| `runAlignment` | 交叉轴行对齐 | `align-content` |
| `runSpacing` | 交叉轴行间距 | `row-gap` |
| `crossAxisAlignment` | 交叉轴子元素对齐 | `align-items` |

### 5. Container 详解

Container 是 Flutter 中最常用的复合容器，类似 CSS 中 `<div>` 的角色。它是多个单一职责 Widget 的组合快捷方式。

```
Container 组合示意图：
┌─────────────────────────────────────────┐
│  margin (外边距)                         │
│  ┌─────────────────────────────────────┐ │
│  │  decoration (背景装饰，含 border)    │ │
│  │  ┌─────────────────────────────────┐│ │
│  │  │  padding (内边距)               ││ │
│  │  │  ┌─────────────────────────────┐││ │
│  │  │  │  constraints (约束)         │││ │
│  │  │  │  ┌─────────────────────────┐│││ │
│  │  │  │  │  transform (变换)       ││││ │
│  │  │  │  │  ┌─────────────────────┐││││ │
│  │  │  │  │  │   child (子组件)    │││││ │
│  │  │  │  │  └─────────────────────┘││││ │
│  │  │  │  └─────────────────────────┘│││ │
│  │  │  └─────────────────────────────┘││ │
│  │  └─────────────────────────────────┘│ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘

Container 渲染顺序：
1. 先应用 padding
2. 如果有 decoration → 填充背景
3. 填充 child
4. 如果有 foregroundDecoration → 填充前景装饰
5. 应用 margin
6. 应用 transform
```

**Container 关键属性：**

| 属性 | 类型 | 说明 | CSS 对应 |
|------|------|------|----------|
| `padding` | EdgeInsets | 内边距 | `padding` |
| `margin` | EdgeInsets | 外边距 | `margin` |
| `decoration` | BoxDecoration | 背景装饰 | `background` + `border` + `box-shadow` |
| `foregroundDecoration` | Decoration | 前景装饰 | `::after` 伪元素 |
| `constraints` | BoxConstraints | 尺寸约束 | `min-width/max-width/min-height/max-height` |
| `width` / `height` | double | 宽高 | `width` / `height` |
| `alignment` | Alignment | 子组件对齐 | `justify-content` + `align-items: center` |
| `transform` | Matrix4 | 矩阵变换 | `transform` |
| `color` | Color | 背景色 | `background-color` |
| `child` | Widget | 子组件 | 子元素 |

> **注意**：Container 的 `color` 和 `decoration` 不能同时设置，因为 `color` 本质上是 `BoxDecoration(color: ...)` 的简写。需要同时设置背景色和边框/阴影时，只用 `decoration`。

**Container 实战 — 多种卡片样式：**

```dart
class ContainerStylesDemo extends StatelessWidget {
  const ContainerStylesDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        children: [
          // 样式1：圆角卡片 + 阴影
          Container(
            margin: const EdgeInsets.only(bottom: 16),
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.circular(16),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.1),
                  blurRadius: 10,
                  offset: const Offset(0, 4),
                ),
              ],
            ),
            child: const Text('圆角阴影卡片'),
          ),

          // 样式2：渐变背景
          Container(
            margin: const EdgeInsets.only(bottom: 16),
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              gradient: const LinearGradient(
                colors: [Colors.blue, Colors.purple],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
              borderRadius: BorderRadius.circular(16),
            ),
            child: const Text(
              '渐变背景卡片',
              style: TextStyle(color: Colors.white, fontSize: 18),
            ),
          ),

          // 样式3：边框卡片
          Container(
            margin: const EdgeInsets.only(bottom: 16),
            padding: const EdgeInsets.all(16),
            decoration: BoxDecoration(
              border: Border.all(color: Colors.blue, width: 2),
              borderRadius: BorderRadius.circular(16),
            ),
            child: const Text('边框卡片'),
          ),

          // 样式4：圆形头像容器
          Container(
            width: 80,
            height: 80,
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              image: const DecorationImage(
                image: NetworkImage('https://picsum.photos/80/80'),
                fit: BoxFit.cover,
              ),
              border: Border.all(color: Colors.white, width: 3),
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.2),
                  blurRadius: 8,
                  offset: const Offset(0, 2),
                ),
              ],
            ),
          ),

          // 样式5：transform 变换
          Container(
            margin: const EdgeInsets.only(bottom: 16),
            padding: const EdgeInsets.all(16),
            transform: Matrix4.rotationZ(-0.05), // 微微倾斜
            decoration: BoxDecoration(
              color: Colors.amber.shade100,
              borderRadius: BorderRadius.circular(8),
            ),
            child: const Text('倾斜变换卡片'),
          ),
        ],
      ),
    );
  }
}
```

### 6. 尺寸约束组件

Flutter 提供了多种尺寸约束组件，用于控制子组件的大小范围。

```dart
// ── SizedBox：固定尺寸盒子 ──
SizedBox(
  width: 100,
  height: 100,
  child: Container(color: Colors.red),
)
// SizedBox 也可用作间距（不设 child）
SizedBox(height: 16) // 垂直间距16

// ── ConstrainedBox：约束盒子 ──
ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 100,
    maxWidth: 300,
    minHeight: 50,
    maxHeight: 200,
  ),
  child: Container(color: Colors.blue),
)

// ── LimitedBox：限制最大尺寸（仅在父约束无限制时生效） ──
LimitedBox(
  maxHeight: 200,  // 当父约束无限高时，限制最大高度为200
  maxWidth: 300,
  child: Container(color: Colors.green),
)

// ── AspectRatio：宽高比盒子 ──
AspectRatio(
  aspectRatio: 16 / 9, // 宽高比 16:9
  child: Container(color: Colors.orange),
)

// ── FractionallySizedBox：占父组件比例的盒子 ──
FractionallySizedBox(
  widthFactor: 0.8,   // 宽度 = 父宽度 × 0.8
  heightFactor: 0.5,   // 高度 = 父高度 × 0.5
  alignment: Alignment.center,
  child: Container(color: Colors.purple),
)

// ── IntrinsicWidth / IntrinsicHeight：根据子组件自适应 ──
// 注意：性能开销大，谨慎使用
IntrinsicWidth(
  child: Column(
    children: [
      Container(color: Colors.red, height: 40, child: Text('短')),
      Container(color: Colors.green, height: 40, child: Text('比较长的文本')),
      // 两行宽度一致，取最宽的
    ],
  ),
)
```

**尺寸约束组件对比：**

| 组件 | 用途 | 性能 | 典型场景 |
|------|------|------|----------|
| `SizedBox` | 固定尺寸 | 优 | 固定宽高、间距 |
| `ConstrainedBox` | 范围约束 | 优 | 最小/最大宽高 |
| `LimitedBox` | 无限约束时限制 | 优 | ListView 子项 |
| `AspectRatio` | 宽高比 | 优 | 视频播放器、图片 |
| `FractionallySizedBox` | 占比尺寸 | 优 | 半屏弹窗、进度条 |
| `IntrinsicWidth/Height` | 自适应子组件 | 差 | 等宽按钮组 |

### 7. Padding / Align / Center

这三个组件是布局中最基础的"修饰型"组件，Container 内部就是组合了它们。

```dart
// Padding — 纯内边距组件（比 Container(padding:...) 性能更好）
Padding(
  padding: const EdgeInsets.all(16),
  child: Text('带内边距的文本'),
)

// EdgeInsets 四种构造方式：
EdgeInsets.all(16)                        // 四周相同
EdgeInsets.symmetric(horizontal: 16, vertical: 8) // 水平/垂直
EdgeInsets.only(left: 16, top: 8, right: 16, bottom: 8) // 分别指定
EdgeInsets.fromLTRB(16, 8, 16, 8)         // left, top, right, bottom

// Align — 对齐组件
Align(
  alignment: Alignment.center,
  child: FlutterLogo(size: 50),
)

// Alignment 坐标系：
// (-1, -1) ─────── (0, -1) ─────── (1, -1)
//    │                  │                 │
// (-1,  0) ─────── (0,  0) ─────── (1,  0)
//    │                  │                 │
// (-1,  1) ─────── (0,  1) ─────── (1,  1)

// Center = Align(alignment: Alignment.center)
Center(
  child: FlutterLogo(size: 50),
)
```

### 8. ListView / GridView / CustomScrollView

**ListView — 列表滚动：**

```dart
// 方式1：ListView — 适合少量子项
ListView(
  padding: const EdgeInsets.all(16),
  children: [
    ListTile(title: Text('项目1')),
    ListTile(title: Text('项目2')),
    ListTile(title: Text('项目3')),
  ],
)

// 方式2：ListView.builder — 适合大量子项（懒加载）
ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return ListTile(
      leading: CircleAvatar(child: Text('$index')),
      title: Text('项目 $index'),
      subtitle: Text('这是第 $index 个项目的描述'),
    );
  },
)

// 方式3：ListView.separated — 带分隔线
ListView.separated(
  itemCount: 20,
  separatorBuilder: (context, index) => const Divider(height: 1),
  itemBuilder: (context, index) {
    return ListTile(title: Text('项目 $index'));
  },
)

// 方式4：ListView.custom — 自定义子项代理
ListView.custom(
  childrenDelegate: SliverChildBuilderDelegate(
    (context, index) => ListTile(title: Text('项目 $index')),
    childCount: 100,
  ),
)
```

**GridView — 网格布局：**

```dart
// 方式1：GridView.count — 指定列数
GridView.count(
  crossAxisCount: 3,       // 3列
  mainAxisSpacing: 10,     // 主轴间距
  crossAxisSpacing: 10,    // 交叉轴间距
  childAspectRatio: 1.0,   // 子项宽高比
  children: List.generate(
    9,
    (index) => Container(
      color: Colors.primaries[index % Colors.primaries.length],
      child: Center(child: Text('$index')),
    ),
  ),
)

// 方式2：GridView.extent — 指定子项最大宽度
GridView.extent(
  maxCrossAxisExtent: 150, // 子项最大宽度150，自动计算列数
  mainAxisSpacing: 10,
  crossAxisSpacing: 10,
  children: List.generate(
    20,
    (index) => Container(
      color: Colors.primaries[index % Colors.primaries.length],
      child: Center(child: Text('$index')),
    ),
  ),
)

// 方式3：GridView.builder — 懒加载
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    mainAxisSpacing: 10,
    crossAxisSpacing: 10,
    childAspectRatio: 0.8,
  ),
  itemCount: 50,
  itemBuilder: (context, index) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.primaries[index % Colors.primaries.length],
        borderRadius: BorderRadius.circular(12),
      ),
      child: Center(child: Text('Item $index')),
    );
  },
)
```

**CustomScrollView + Sliver — 高级滚动布局：**

```dart
class CustomScrollViewDemo extends StatelessWidget {
  const CustomScrollViewDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // SliverAppBar — 可折叠的顶部应用栏
          SliverAppBar(
            expandedHeight: 200,
            floating: false,
            pinned: true,       // 滚动时固定在顶部
            snap: false,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('自定义滚动视图'),
              background: Container(
                decoration: const BoxDecoration(
                  gradient: LinearGradient(
                    colors: [Colors.blue, Colors.indigo],
                    begin: Alignment.topLeft,
                    end: Alignment.bottomRight,
                  ),
                ),
              ),
            ),
          ),

          // SliverToBoxAdapter — 将普通 Widget 包装为 Sliver
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                '热门推荐',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
            ),
          ),

          // SliverGrid — 网格区域
          SliverGrid(
            gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              mainAxisSpacing: 10,
              crossAxisSpacing: 10,
              childAspectRatio: 1.2,
            ),
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return Container(
                  decoration: BoxDecoration(
                    color: Colors.primaries[index % Colors.primaries.length],
                    borderRadius: BorderRadius.circular(12),
                  ),
                  child: Center(child: Text('Grid $index')),
                );
              },
              childCount: 6,
            ),
          ),

          // SliverToBoxAdapter — 分隔
          const SliverToBoxAdapter(
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                '最新列表',
                style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
              ),
            ),
          ),

          // SliverList — 列表区域
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) {
                return ListTile(
                  leading: CircleAvatar(child: Text('$index')),
                  title: Text('列表项 $index'),
                  subtitle: Text('这是第 $index 个列表项的描述信息'),
                );
              },
              childCount: 20,
            ),
          ),

          // SliverPadding — Sliver 内的边距
          const SliverPadding(
            padding: EdgeInsets.only(bottom: 80), // 底部留白
            sliver: SliverToBoxAdapter(child: SizedBox.shrink()),
          ),
        ],
      ),
    );
  }
}
```

**Sliver 组件一览：**

| Sliver 组件 | 用途 |
|-------------|------|
| `SliverList` | 列表 |
| `SliverGrid` | 网格 |
| `SliverAppBar` | 可折叠顶部栏 |
| `SliverToBoxAdapter` | 普通Widget转Sliver |
| `SliverPadding` | 带边距的Sliver |
| `SliverPersistentHeader` | 自定义悬浮头部 |
| `SliverFillRemaining` | 填满剩余空间 |

### 9. 自定义布局

**CustomMultiChildLayout — 精确控制每个子组件的位置和大小：**

```dart
// 自定义布局：实现一个"圆形分布"效果
class CircleLayout extends StatelessWidget {
  const CircleLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomMultiChildLayout(
      delegate: CircleLayoutDelegate(itemCount: 6),
      children: List.generate(
        6,
        (index) => LayoutId(
          id: index,
          child: Container(
            width: 50,
            height: 50,
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              color: Colors.primaries[index % Colors.primaries.length],
            ),
            child: Center(
              child: Text('$index', style: const TextStyle(color: Colors.white)),
            ),
          ),
        ),
      ),
    );
  }
}

class CircleLayoutDelegate extends MultiChildLayoutDelegate {
  final int itemCount;

  CircleLayoutDelegate({required this.itemCount});

  @override
  void performLayout(Size size) {
    // 中心点
    final centerX = size.width / 2;
    final centerY = size.height / 2;
    final radius = size.width * 0.3; // 圆的半径

    for (int i = 0; i < itemCount; i++) {
      // 获取子组件 ID
      final id = i;
      if (hasChild(id)) {
        // 计算角度
        final angle = (2 * 3.14159265 / itemCount) * i - 3.14159265 / 2;
        // 计算位置
        final x = centerX + radius * cos(angle);
        final y = centerY + radius * sin(angle);

        // 布局子组件
        final childSize = layoutChild(id, BoxConstraints.loose(size));
        // 定位子组件
        positionChild(id, Offset(x - childSize.width / 2, y - childSize.height / 2));
      }
    }
  }

  @override
  bool shouldRelayout(covariant CircleLayoutDelegate oldDelegate) {
    return oldDelegate.itemCount != itemCount;
  }
}
```

**Flow — 流式自定义布局：**

```dart
class FlowDemo extends StatelessWidget {
  const FlowDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return Flow(
      delegate: _MarginFlowDelegate(margin: EdgeInsets.all(8)),
      children: List.generate(
        10,
        (index) => Container(
          width: 40 + index * 10.0,
          height: 40,
          color: Colors.primaries[index % Colors.primaries.length],
          child: Center(child: Text('$index')),
        ),
      ),
    );
  }
}

class _MarginFlowDelegate extends FlowDelegate {
  final EdgeInsets margin;

  _MarginFlowDelegate({required this.margin});

  @override
  void paintChildren(FlowPaintingContext context) {
    double x = margin.left;
    double y = margin.top;

    for (int i = 0; i < context.childCount; i++) {
      final childSize = context.getChildSize(i)!;
      // 如果当前行放不下，换行
      if (x + childSize.width + margin.right > context.size.width) {
        x = margin.left;
        y += childSize.height + margin.bottom;
      }
      context.paintChild(i, transform: Matrix4.translationValues(x, y, 0));
      x += childSize.width + margin.right;
    }
  }

  @override
  bool shouldRepaint(covariant _MarginFlowDelegate oldDelegate) {
    return margin != oldDelegate.margin;
  }
}
```

### 10. 样式系统

#### TextStyle — 文本样式

```dart
Text(
  '样式文本',
  style: TextStyle(
    fontSize: 24,                          // 字号
    fontWeight: FontWeight.bold,            // 字重
    fontStyle: FontStyle.italic,            // 斜体
    color: Colors.blue,                     // 颜色
    letterSpacing: 2.0,                     // 字间距
    wordSpacing: 4.0,                       // 词间距
    height: 1.5,                            // 行高（倍数）
    decoration: TextDecoration.underline,   // 装饰线（下划线）
    decorationColor: Colors.red,            // 装饰线颜色
    decorationStyle: TextDecorationStyle.dashed, // 装饰线样式
    shadows: [                              // 阴影
      Shadow(
        color: Colors.black.withOpacity(0.3),
        offset: const Offset(2, 2),
        blurRadius: 4,
      ),
    ],
    fontFamily: 'Roboto',                   // 字体族
    background: Paint()..color = Colors.yellow.withOpacity(0.3), // 背景
  ),
)
```

#### BoxDecoration — 盒装饰

```dart
BoxDecoration(
  // 背景色
  color: Colors.white,

  // 渐变背景（与 color 互斥，gradient 优先）
  gradient: const LinearGradient(
    colors: [Colors.blue, Colors.purple],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
    stops: [0.0, 1.0], // 渐变停止点
  ),

  // 边框
  border: Border.all(
    color: Colors.grey,
    width: 1,
  ),
  // 或者分别设置四边
  border: const Border(
    top: BorderSide(color: Colors.red, width: 2),
    bottom: BorderSide(color: Colors.blue, width: 2),
  ),

  // 圆角
  borderRadius: BorderRadius.circular(16),
  // 或者分别设置
  borderRadius: const BorderRadius.only(
    topLeft: Radius.circular(16),
    bottomRight: Radius.circular(16),
  ),

  // 阴影
  boxShadow: [
    BoxShadow(
      color: Colors.black.withOpacity(0.1),
      blurRadius: 10,           // 模糊半径
      spreadRadius: 2,          // 扩散半径
      offset: const Offset(0, 4), // 偏移
    ),
  ],

  // 背景图片
  image: const DecorationImage(
    image: NetworkImage('https://example.com/bg.jpg'),
    fit: BoxFit.cover,
    alignment: Alignment.center,
  ),

  // 形状（与 borderRadius 互斥，BoxShape.circle 时不需 borderRadius）
  shape: BoxShape.rectangle,
)
```

#### ShapeDecoration — 形状装饰

```dart
// ShapeDecoration 更适合自定义形状
ShapeDecoration(
  color: Colors.white,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(16),
  ),
  shadows: [
    BoxShadow(
      color: Colors.black.withOpacity(0.1),
      blurRadius: 10,
      offset: const Offset(0, 4),
    ),
  ],
)

// StadiumShape — 体育场形状（两端半圆）
ShapeDecoration(
  shape: const StadiumBorder(),
  color: Colors.blue,
)

// BeveledRectangleShape — 斜角矩形
ShapeDecoration(
  shape: BeveledRectangleBorder(
    borderRadius: BorderRadius.circular(16),
  ),
  color: Colors.white,
)
```

#### InputBorder — 输入框边框

```dart
TextField(
  decoration: InputDecoration(
    // 未聚焦边框
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: const BorderSide(color: Colors.grey, width: 1),
    ),
    // 聚焦边框
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: const BorderSide(color: Colors.blue, width: 2),
    ),
    // 错误边框
    errorBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: const BorderSide(color: Colors.red, width: 1),
    ),
    // 聚焦错误边框
    focusedErrorBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(12),
      borderSide: const BorderSide(color: Colors.red, width: 2),
    ),
    // 下划线边框（默认样式）
    // UnderlineInputBorder()
  ),
)
```

### 11. 颜色与渐变

```dart
// ── Color 创建方式 ──

// ARGB 整数
Color(0xFFFF5722)      // 0xAARRGGBB

// 命名颜色
Colors.red
Colors.blue.shade200   // 色阶

// fromARGB
Color.fromARGB(255, 255, 87, 34)

// fromRGBO（不透明度）
Color.fromRGBO(255, 87, 34, 1.0)

// 透明度
Colors.red.withOpacity(0.5)     // 50% 透明
Colors.red.withAlpha(128)       // 128/255 透明

// ── 渐变 Gradient ──

// 线性渐变
LinearGradient(
  begin: Alignment.topLeft,
  end: Alignment.bottomRight,
  colors: [Colors.blue, Colors.purple, Colors.pink],
  stops: [0.0, 0.5, 1.0],  // 每个颜色的位置
)

// 径向渐变
RadialGradient(
  center: Alignment.center,
  radius: 0.5,
  colors: [Colors.yellow, Colors.orange, Colors.red],
  stops: [0.0, 0.5, 1.0],
)

// 扫描渐变（扇形）
SweepGradient(
  center: Alignment.center,
  startAngle: 0.0,
  endAngle: 6.28, // 2π
  colors: [Colors.red, Colors.yellow, Colors.green, Colors.blue, Colors.red],
)

// ── BoxShadow ──
BoxShadow(
  color: Colors.black.withOpacity(0.2),
  blurRadius: 10,            // 模糊半径
  spreadRadius: 0,           // 扩散半径（正值放大，负值缩小）
  offset: const Offset(0, 4), // 偏移（x右移，y下移）
)
```

**渐变实战 — 漂亮的按钮：**

```dart
class GradientButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final List<Color> colors;

  const GradientButton({
    super.key,
    required this.text,
    required this.onPressed,
    this.colors = const [Colors.blue, Colors.purple],
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onPressed,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
        decoration: BoxDecoration(
          gradient: LinearGradient(colors: colors),
          borderRadius: BorderRadius.circular(30),
          boxShadow: [
            BoxShadow(
              color: colors.last.withOpacity(0.4),
              blurRadius: 12,
              offset: const Offset(0, 6),
            ),
          ],
        ),
        child: Text(
          text,
          style: const TextStyle(
            color: Colors.white,
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
          textAlign: TextAlign.center,
        ),
      ),
    );
  }
}
```

### 12. 图标与图片

```dart
// ── Icon ──
Icon(
  Icons.favorite,       // 图标数据
  size: 32,             // 大小
  color: Colors.red,    // 颜色
  semanticLabel: '喜欢', // 语义标签（无障碍）
  textDirection: TextDirection.ltr, // 方向
)

// 自定义图标（使用 IconData）
Icon(
  IconData(0xe900, fontFamily: 'MyIconFont'),
  size: 24,
)

// IconButton — 可点击图标
IconButton(
  onPressed: () {},
  icon: const Icon(Icons.search),
  iconSize: 28,
  color: Colors.blue,
  tooltip: '搜索',
)

// ── Image ──

// 网络图片
Image.network(
  'https://example.com/image.png',
  width: 200,
  height: 200,
  fit: BoxFit.cover,        // 适配方式
  color: Colors.blue,        // 混合颜色
  colorBlendMode: BlendMode.multiply, // 混合模式
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Center(
      child: CircularProgressIndicator(
        value: loadingProgress.expectedTotalBytes != null
            ? loadingProgress.cumulativeBytesLoaded /
                loadingProgress.expectedTotalBytes!
            : null,
      ),
    );
  },
  errorBuilder: (context, error, stackTrace) {
    return const Icon(Icons.broken_image, size: 100, color: Colors.grey);
  },
)

// 本地图片（需在 pubspec.yaml 中声明）
Image.asset(
  'assets/images/logo.png',
  width: 100,
  fit: BoxFit.contain,
)

// 文件图片
Image.file(
  File('/path/to/image.png'),
  fit: BoxFit.cover,
)

// 内存图片
Image.memory(
  Uint8List.fromList([...]),
  fit: BoxFit.cover,
)

// ── DecorationImage ──（作为背景图使用）
Container(
  decoration: BoxDecoration(
    image: DecorationImage(
      image: const NetworkImage('https://example.com/bg.jpg'),
      fit: BoxFit.cover,
      colorFilter: ColorFilter.mode(
        Colors.black.withOpacity(0.3),
        BlendMode.darken,
      ),
    ),
  ),
  child: const Text('前景文字'),
)
```

### 13. Material 组件样式

```dart
// ── Card ──
Card(
  elevation: 4,            // 阴影高度
  margin: const EdgeInsets.all(8),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(16),
  ),
  color: Colors.white,
  shadowColor: Colors.black.withOpacity(0.2),
  clipBehavior: Clip.antiAlias,
  child: Padding(
    padding: const EdgeInsets.all(16),
    child: Text('Card 内容'),
  ),
)

// ── ElevatedButton ──（填充按钮）
ElevatedButton(
  onPressed: () {},
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue,        // 背景色
    foregroundColor: Colors.white,       // 前景色（文字/图标）
    elevation: 4,                        // 阴影
    padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(12),
    ),
    textStyle: const TextStyle(
      fontSize: 16,
      fontWeight: FontWeight.bold,
    ),
  ),
  child: const Text('填充按钮'),
)

// ── TextButton ──（文字按钮）
TextButton(
  onPressed: () {},
  style: TextButton.styleFrom(
    foregroundColor: Colors.blue,
    padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: const Text('文字按钮'),
)

// ── OutlinedButton ──（边框按钮）
OutlinedButton(
  onPressed: () {},
  style: OutlinedButton.styleFrom(
    foregroundColor: Colors.blue,
    side: const BorderSide(color: Colors.blue, width: 2),
    padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(12),
    ),
  ),
  child: const Text('边框按钮'),
)

// ── IconButton ──
IconButton(
  onPressed: () {},
  icon: const Icon(Icons.favorite),
  color: Colors.red,
  iconSize: 32,
  splashRadius: 24,
  tooltip: '收藏',
)

// ── FloatingAction Button ──
FloatingActionButton(
  onPressed: () {},
  child: const Icon(Icons.add),
  backgroundColor: Colors.blue,
  foregroundColor: Colors.white,
  elevation: 6,
  shape: const CircleBorder(),
)

// ── Chip ──
Chip(
  label: const Text('标签'),
  avatar: const CircleAvatar(child: Text('F')),
  deleteIcon: const Icon(Icons.close, size: 16),
  onDeleted: () {},
  backgroundColor: Colors.blue.shade50,
  side: BorderSide(color: Colors.blue.shade200),
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(20),
  ),
)
```

### 14. 主题系统

```dart
// ── 全局主题定义 ──
MaterialApp(
  theme: ThemeData(
    // 颜色方案（Material 3）
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.light, // 亮色模式
    ),
    // 也可以手动定义
    // colorScheme: const ColorScheme(
    //   primary: Colors.blue,
    //   onPrimary: Colors.white,
    //   secondary: Colors.amber,
    //   onSecondary: Colors.black,
    //   surface: Colors.white,
    //   onSurface: Colors.black,
    //   error: Colors.red,
    //   onError: Colors.white,
    //   brightness: Brightness.light,
    // ),

    // 文字主题
    textTheme: const TextTheme(
      displayLarge: TextStyle(fontSize: 57, fontWeight: FontWeight.w400),
      displayMedium: TextStyle(fontSize: 45, fontWeight: FontWeight.w400),
      displaySmall: TextStyle(fontSize: 36, fontWeight: FontWeight.w400),
      headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.w400),
      headlineMedium: TextStyle(fontSize: 28, fontWeight: FontWeight.w400),
      headlineSmall: TextStyle(fontSize: 24, fontWeight: FontWeight.w400),
      titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.w500),
      titleMedium: TextStyle(fontSize: 16, fontWeight: FontWeight.w500),
      titleSmall: TextStyle(fontSize: 14, fontWeight: FontWeight.w500),
      bodyLarge: TextStyle(fontSize: 16, fontWeight: FontWeight.w400),
      bodyMedium: TextStyle(fontSize: 14, fontWeight: FontWeight.w400),
      bodySmall: TextStyle(fontSize: 12, fontWeight: FontWeight.w400),
      labelLarge: TextStyle(fontSize: 14, fontWeight: FontWeight.w500),
      labelMedium: TextStyle(fontSize: 12, fontWeight: FontWeight.w500),
      labelSmall: TextStyle(fontSize: 11, fontWeight: FontWeight.w500),
    ),

    // 应用栏主题
    appBarTheme: const AppBarTheme(
      backgroundColor: Colors.blue,
      foregroundColor: Colors.white,
      elevation: 0,
      centerTitle: true,
    ),

    // 卡片主题
    cardTheme: CardTheme(
      elevation: 4,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
      ),
    ),

    // 按钮主题
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
      ),
    ),

    // 输入框主题
    inputDecorationTheme: InputDecorationTheme(
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      filled: true,
      fillColor: Colors.grey.shade100,
    ),

    // 使用 Material 3
    useMaterial3: true,
  ),

  // 暗色主题
  darkTheme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  ),

  // 主题模式：跟随系统 / 强制亮色 / 强制暗色
  themeMode: ThemeMode.system,
)
```

**使用主题：**

```dart
class ThemedWidget extends StatelessWidget {
  const ThemedWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // 获取主题数据
    final theme = Theme.of(context);
    final colorScheme = theme.colorScheme;
    final textTheme = theme.textTheme;

    return Container(
      color: colorScheme.surface,            // 使用主题颜色
      child: Text(
        '主题文本',
        style: textTheme.headlineMedium?.copyWith( // 使用主题文字样式
          color: colorScheme.onSurface,
        ),
      ),
    );
  }
}

// 局部覆盖主题
Theme(
  data: Theme.of(context).copyWith(
    colorScheme: Theme.of(context).colorScheme.copyWith(
      primary: Colors.orange,  // 局部修改主色
    ),
  ),
  child: const MyWidget(),
)
```

### 15. 响应式设计

```dart
// ── MediaQuery — 获取屏幕信息 ──
class MediaQueryDemo extends StatelessWidget {
  const MediaQueryDemo({super.key});

  @override
  Widget build(BuildContext context) {
    final mediaQuery = MediaQuery.of(context);
    final screenWidth = mediaQuery.size.width;
    final screenHeight = mediaQuery.size.height;
    final pixelRatio = mediaQuery.devicePixelRatio;
    final padding = mediaQuery.padding;        // 安全区域
    final orientation = mediaQuery.orientation; // 屏幕方向

    return Column(
      children: [
        Text('屏幕宽度: $screenWidth'),
        Text('屏幕高度: $screenHeight'),
        Text('像素比: $pixelRatio'),
        Text('方向: $orientation'),
      ],
    );
  }
}

// ── LayoutBuilder — 根据父约束构建不同布局 ──
class ResponsiveLayout extends StatelessWidget {
  const ResponsiveLayout({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // constraints.maxWidth 是父组件可用的最大宽度

        if (constraints.maxWidth > 1200) {
          // 桌面端：三栏布局
          return Row(
            children: [
              const Expanded(flex: 1, child: LeftPanel()),
              const Expanded(flex: 2, child: CenterPanel()),
              const Expanded(flex: 1, child: RightPanel()),
            ],
          );
        } else if (constraints.maxWidth > 600) {
          // 平板：两栏布局
          return Row(
            children: [
              const Expanded(flex: 1, child: LeftPanel()),
              const Expanded(flex: 2, child: CenterPanel()),
            ],
          );
        } else {
          // 手机：单栏布局
          return const CenterPanel();
        }
      },
    );
  }
}

// ── OrientationBuilder — 根据屏幕方向构建 ──
class OrientationDemo extends StatelessWidget {
  const OrientationDemo({super.key});

  @override
  Widget build(BuildContext context) {
    return OrientationBuilder(
      builder: (context, orientation) {
        if (orientation == Orientation.portrait) {
          // 竖屏
          return Column(
            children: [
              Expanded(child: Image.asset('assets/poster.png')),
              const Text('竖屏模式'),
            ],
          );
        } else {
          // 横屏
          return Row(
            children: [
              Expanded(child: Image.asset('assets/poster.png')),
              const Expanded(child: Text('横屏模式')),
            ],
          );
        }
      },
    );
  }
}
```

**完整的响应式页面实战：**

```dart
class ResponsiveApp extends StatelessWidget {
  const ResponsiveApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('响应式布局')),
      body: LayoutBuilder(
        builder: (context, constraints) {
          final isWide = constraints.maxWidth > 600;
          final padding = isWide ? 32.0 : 16.0;
          final crossAxisCount = isWide ? 4 : 2;
          final fontSize = isWide ? 24.0 : 18.0;

          return Padding(
            padding: EdgeInsets.all(padding),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  '精选推荐',
                  style: TextStyle(fontSize: fontSize, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 16),
                Expanded(
                  child: GridView.builder(
                    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                      crossAxisCount: crossAxisCount,
                      mainAxisSpacing: 12,
                      crossAxisSpacing: 12,
                      childAspectRatio: 0.8,
                    ),
                    itemCount: 20,
                    itemBuilder: (context, index) {
                      return _buildProductCard(index, isWide);
                    },
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _buildProductCard(int index, bool isWide) {
    return Card(
      clipBehavior: Clip.antiAlias,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            flex: 3,
            child: Container(
              color: Colors.primaries[index % Colors.primaries.length],
              width: double.infinity,
              child: Center(
                child: Icon(
                  Icons.shopping_bag,
                  size: isWide ? 48 : 32,
                  color: Colors.white,
                ),
              ),
            ),
          ),
          Expanded(
            flex: 2,
            child: Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    '商品 $index',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  const Spacer(),
                  Text(
                    '¥${(index + 1) * 99}',
                    style: const TextStyle(color: Colors.red, fontSize: 16),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

### 16. SafeArea 与屏幕适配

```dart
// ── SafeArea — 避开刘海屏/底部安全区域 ──
SafeArea(
  left: true,     // 左侧安全区域（默认true）
  top: true,      // 顶部安全区域（默认true，避开刘海）
  right: true,    // 右侧安全区域
  bottom: true,   // 底部安全区域（默认true，避开底部横条）
  minimum: const EdgeInsets.all(16), // 额外最小边距
  child: Text('安全区域内的内容'),
)

// SafeArea 的原理：读取 MediaQuery.of(context).padding
// 等价于：
Padding(
  padding: MediaQuery.of(context).padding,
  child: Text('安全区域内的内容'),
)

// ── 屏幕适配方案 ──

// 方案1：按设计稿比例缩放（推荐）
class ScreenUtil {
  static const double designWidth = 375; // 设计稿宽度

  static double scaleWidth(BuildContext context) {
    return MediaQuery.of(context).size.width / designWidth;
  }

  static double wp(BuildContext context, double value) {
    return value * scaleWidth(context);
  }

  static double sp(BuildContext context, double fontSize) {
    return fontSize * scaleWidth(context);
  }
}

// 使用
Text(
  '适配文字',
  style: TextStyle(fontSize: ScreenUtil.sp(context, 16)),
)
Container(
  width: ScreenUtil.wp(context, 200),
  height: ScreenUtil.wp(context, 100),
)

// 方案2：使用 flutter_screenutil 包
// ScreenUtil.init(context, designSize: const Size(375, 812));
// Text('适配', style: TextStyle(fontSize: 16.sp))
// Container(width: 200.w, height: 100.h)
```

### 17. 完整实战案例 — 仿微信聊天页面

```dart
import 'package:flutter/material.dart';

class WeChatChatPage extends StatelessWidget {
  const WeChatChatPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFEDEDED),
      appBar: AppBar(
        backgroundColor: const Color(0xFFEDEDED),
        elevation: 0.5,
        title: const Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('张三', style: TextStyle(fontSize: 17)),
            SizedBox(width: 4),
            Icon(Icons.keyboard_arrow_down, size: 18),
          ],
        ),
        actions: [
          IconButton(
            onPressed: () {},
            icon: const Icon(Icons.more_horiz),
          ),
        ],
      ),
      body: Column(
        children: [
          // 消息列表
          Expanded(
            child: ListView.builder(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
              itemCount: 20,
              itemBuilder: (context, index) {
                final isMe = index % 3 != 0;
                return _ChatBubble(isMe: isMe, index: index);
              },
            ),
          ),

          // 底部输入栏
          Container(
            padding: EdgeInsets.only(
              left: 8,
              right: 8,
              top: 8,
              bottom: MediaQuery.of(context).padding.bottom + 8,
            ),
            color: const Color(0xFFF7F7F7),
            child: Row(
              crossAxisAlignment: CrossAxisAlignment.end,
              children: [
                // 语音按钮
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.keyboard_voice, color: Colors.grey),
                ),
                // 输入框
                Expanded(
                  child: Container(
                    constraints: const BoxConstraints(maxHeight: 100),
                    child: TextField(
                      maxLines: null,
                      decoration: InputDecoration(
                        hintText: '输入消息...',
                        filled: true,
                        fillColor: Colors.white,
                        contentPadding: const EdgeInsets.symmetric(
                          horizontal: 12,
                          vertical: 10,
                        ),
                        border: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(6),
                          borderSide: BorderSide.none,
                        ),
                      ),
                    ),
                  ),
                ),
                // 表情按钮
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.emoji_emotions_outlined, color: Colors.grey),
                ),
                // 加号按钮
                IconButton(
                  onPressed: () {},
                  icon: const Icon(Icons.add_circle_outline, color: Colors.grey),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class _ChatBubble extends StatelessWidget {
  final bool isMe;
  final int index;

  const _ChatBubble({required this.isMe, required this.index});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 16),
      child: Row(
        mainAxisAlignment: isMe
            ? MainAxisAlignment.end     // 自己的消息右对齐
            : MainAxisAlignment.start,   // 对方的消息左对齐
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          if (!isMe) ...[
            // 对方头像
            CircleAvatar(
              radius: 20,
              backgroundColor: Colors.blue.shade100,
              child: const Icon(Icons.person, size: 20),
            ),
            const SizedBox(width: 8),
          ],

          // 消息气泡
          ConstrainedBox(
            constraints: BoxConstraints(
              maxWidth: MediaQuery.of(context).size.width * 0.65,
            ),
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
              decoration: BoxDecoration(
                color: isMe ? const Color(0xFF95EC69) : Colors.white,
                borderRadius: BorderRadius.only(
                  topLeft: const Radius.circular(4),
                  topRight: const Radius.circular(12),
                  bottomLeft: Radius.circular(isMe ? 12 : 4),
                  bottomRight: Radius.circular(isMe ? 4 : 12),
                ),
              ),
              child: Text(
                isMe ? '这是我的第 $index 条消息' : '这是对方发来的第 $index 条消息',
                style: const TextStyle(fontSize: 16),
              ),
            ),
          ),

          if (isMe) ...[
            const SizedBox(width: 8),
            // 自己头像
            CircleAvatar(
              radius: 20,
              backgroundColor: Colors.green.shade100,
              child: const Icon(Icons.person, size: 20),
            ),
          ],
        ],
      ),
    );
  }
}
```

### 18. 常见布局陷阱与解决方案

```dart
// ── 陷阱1：Column 中嵌套 ListView 无限高度 ──
// ❌ 错误
Column(
  children: [
    Text('标题'),
    ListView.builder(itemCount: 50, itemBuilder: ...), // 报错！
  ],
)

// ✅ 方案A：用 Expanded
Column(
  children: [
    Text('标题'),
    Expanded(child: ListView.builder(itemCount: 50, itemBuilder: ...)),
  ],
)

// ✅ 方案B：用 shrinkWrap
Column(
  children: [
    Text('标题'),
    ListView.builder(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      itemCount: 50,
      itemBuilder: ...,
    ),
  ],
)

// ── 陷阱2：Row 中文本溢出 ──
// ❌ 错误：文本太长会溢出
Row(
  children: [
    Icon(Icons.star),
    Text('很长的文本内容......'),  // 溢出！
    Icon(Icons.arrow),
  ],
)

// ✅ 用 Expanded 包裹文本
Row(
  children: [
    const Icon(Icons.star),
    Expanded(
      child: Text('很长的文本内容......',
        overflow: TextOverflow.ellipsis, maxLines: 1),
    ),
    const Icon(Icons.arrow),
  ],
)

// ── 陷阱3：Container 没有设置 color 和 decoration 的冲突 ──
// ❌ 错误：同时设置 color 和 decoration
Container(
  color: Colors.white,          // 报错！
  decoration: BoxDecoration(    // 与 color 冲突
    borderRadius: BorderRadius.circular(16),
  ),
  child: Text('内容'),
)

// ✅ 把 color 放到 decoration 里
Container(
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(16),
  ),
  child: Text('内容'),
)

// ── 陷阱4：Stack 子组件超出边界 ──
// ❌ 默认 Stack 的 clipBehavior 会裁剪超出部分
Stack(
  children: [
    Container(width: 100, height: 100, color: Colors.blue),
    Positioned(
      right: -20,  // 超出部分被裁剪
      child: Container(width: 60, height: 60, color: Colors.red),
    ),
  ],
)

// ✅ 设置 clipBehavior: Clip.none
Stack(
  clipBehavior: Clip.none,
  children: [
    Container(width: 100, height: 100, color: Colors.blue),
    Positioned(
      right: -20,  // 超出部分正常显示
      child: Container(width: 60, height: 60, color: Colors.red),
    ),
  ],
)

// ── 陷阱5：GestureDetector 在 Container 上点击区域过小 ──
// ❌ 点击区域只有文字大小
GestureDetector(
  onTap: () {},
  child: Text('按钮'),
)

// ✅ 用 behavior 扩大点击区域
GestureDetector(
  behavior: HitTestBehavior.opaque, // 整个区域可点击
  onTap: () {},
  child: Container(
    padding: const EdgeInsets.all(16),
    child: Text('按钮'),
  ),
)
```

## 面试题（8题）

### 1. Flutter 布局的三条核心规则是什么？请结合具体示例说明。

**答案：**

Flutter 布局遵循三条核心规则：

1. **约束向下传递**：父 Widget 向子 Widget 传递 `BoxConstraints`，告诉子组件可用的最大/最小宽高范围。子组件必须在父约束范围内确定自身尺寸。

2. **尺寸向上返回**：子 Widget 根据父约束和自身内容，确定自己的尺寸并返回给父组件。

3. **父级设置位置**：父组件决定子组件在自身空间中的具体位置。

**示例说明：**

```dart
Center(
  child: Container(
    width: 100,
    height: 100,
    color: Colors.red,
  ),
)
```

布局过程：
1. `Center` 收到父约束（如屏幕大小 400x800），`Center` 向 `Container` 传递相同的约束
2. `Container` 在约束范围内选择自身尺寸 100x100，返回给 `Center`
3. `Center` 将 `Container` 放置在自身的中心位置（150, 350）

理解这三条规则可以解决 90% 的布局溢出问题。最典型的例子是 `Column` 中嵌套 `ListView`：`Column` 向 `ListView` 传递无限高度的约束，`ListView` 试图占满无限高度，导致布局崩溃。解决方案是用 `Expanded` 将无限约束截断为有限剩余空间。

### 2. Expanded 和 Flexible 有什么区别？分别适用于什么场景？

**答案：**

**核心区别：**

- `Expanded` 是 `Flexible(fit: FlexFit.tight)` 的语法糖，子组件**必须**占满剩余空间
- `Flexible(fit: FlexFit.loose)` 允许子组件**选择**不占满剩余空间，子组件可以按自身尺寸显示

**对比：**

```dart
Row(
  children: [
    Flexible(
      fit: FlexFit.loose, // 默认值
      child: Text('Flexible文本'), // 文本按自身宽度显示，但不会超过剩余空间
    ),
    Expanded(
      child: Text('Expanded文本'), // 强制占满剩余空间
    ),
  ],
)
```

**适用场景：**

| 场景 | 推荐组件 | 原因 |
|------|----------|------|
| 列表项中文本自适应 | `Expanded` | 防止文本溢出 |
| 侧边栏占固定比例 | `Expanded(flex: 1)` | 精确控制比例 |
| 图标+文字横排，文字可能很长 | `Expanded` 包裹 Text | Text 必须受限制 |
| 按钮组自适应但不过度拉伸 | `Flexible` | 按钮保持自然宽度 |
| 需要子组件自行决定大小 | `Flexible` | 不强制填充 |

**关键点**：当子组件是 `Text` 且可能很长时，务必用 `Expanded` 包裹，否则会溢出。

### 3. Stack 中 Positioned 和 Align 有什么区别？如何实现一个"角标"效果？

**答案：**

**区别：**

| 维度 | Positioned | Align |
|------|-----------|-------|
| 定位方式 | 绝对定位（top/right/bottom/left） | 比例对齐（alignment） |
| 是否脱离布局流 | 是，不影响其他子组件 | 否，参与 Stack 布局 |
| 尺寸 | 由 top/right/bottom/left 推导或由子组件决定 | 由子组件决定 |
| 适用场景 | 精确位置（角标、浮层） | 整体对齐（居中、角落对齐） |

**角标实现：**

```dart
SizedBox(
  width: 100,
  height: 100,
  child: Stack(
    clipBehavior: Clip.none, // 允许角标超出边界
    children: [
      // 底层：图标
      Container(
        decoration: BoxDecoration(
          color: Colors.blue,
          borderRadius: BorderRadius.circular(20),
        ),
        child: const Center(
          child: Icon(Icons.notifications, color: Colors.white, size: 40),
        ),
      ),
      // 角标：用 Positioned 精确定位到右上角
      Positioned(
        top: -4,   // 向上偏移
        right: -4,  // 向右偏移
        child: Container(
          padding: const EdgeInsets.all(4),
          decoration: const BoxDecoration(
            color: Colors.red,
            shape: BoxShape.circle,
          ),
          constraints: const BoxConstraints(minWidth: 16, minHeight: 16),
          child: const Text(
            '3',
            style: TextStyle(color: Colors.white, fontSize: 10),
            textAlign: TextAlign.center,
          ),
        ),
      ),
    ],
  ),
)
```

### 4. Container 的 color 属性和 BoxDecoration 的 color 有什么区别？为什么不能同时使用？

**答案：**

`Container` 的 `color` 属性本质上是 `BoxDecoration(color: ...)` 的简化写法。查看 `Container` 源码可以发现：

```dart
// Container 源码简化
if (color != null) {
  if (decoration != null) {
    throw ArgumentError('Cannot provide both a color and a decoration');
  }
  // color 被转换为 decoration
  decoration = BoxDecoration(color: color);
}
```

**不能同时使用的原因**：两者最终都会设置 `Container` 的 `decoration` 属性。如果同时指定，Flutter 无法确定以哪个为准，会抛出 `ArgumentError`。

**最佳实践**：

- 只需要设置背景色时，用 `color` 属性（简洁）
- 需要同时设置背景色和其他装饰（边框、阴影、渐变、圆角等）时，只用 `decoration`

```dart
// 简单场景：用 color
Container(color: Colors.white, child: Text('简单'))

// 复杂场景：用 decoration（不要同时设 color）
Container(
  decoration: BoxDecoration(
    color: Colors.white,        // 颜色放这里
    borderRadius: BorderRadius.circular(12),
    boxShadow: [...],
  ),
  child: Text('复杂'),
)
```

### 5. ListView 和 Column 有什么区别？何时使用 ListView，何时使用 Column？

**答案：**

| 维度 | Column | ListView |
|------|--------|----------|
| 滚动 | 不支持滚动 | 支持滚动 |
| 子组件数量 | 适合少量 | 适合大量（懒加载） |
| 布局约束 | 需要有限高度约束 | 自带滚动，接受无限高度 |
| 渲染方式 | 一次性渲染所有子组件 | 按需渲染（builder） |
| 性能 | 子组件多时性能差 | 子组件再多也流畅 |

**选择标准：**

1. **子组件数量少（<20）且不需要滚动** → 用 `Column`
2. **子组件数量多或不确定** → 用 `ListView.builder`
3. **需要滚动 + 少量固定头部** → `Column` + `Expanded(ListView)`
4. **需要复杂滚动效果（粘性头部等）** → `CustomScrollView`

**常见组合模式：**

```dart
// 模式：固定头部 + 可滚动列表
Column(
  children: [
    Header(),              // 固定头部
    Expanded(
      child: ListView.builder(  // 可滚动列表
        itemCount: 100,
        itemBuilder: ...,
      ),
    ),
  ],
)
```

### 6. 如何实现 Flutter 的暗色模式切换？ThemeData 和 ColorScheme 的关系是什么？

**答案：**

**ThemeData 与 ColorScheme 的关系：**

- `ThemeData` 是整个应用的主题配置，包含颜色、文字、形状、组件样式等所有主题信息
- `ColorScheme` 是 `ThemeData` 的一部分，专注于颜色定义，包含 12 个语义化颜色槽位（primary、secondary、surface、error 等）
- `ColorScheme` 是 Material 3 的颜色系统核心

**实现暗色模式切换：**

```dart
class ThemeApp extends StatefulWidget {
  const ThemeApp({super.key});

  @override
  State<ThemeApp> createState() => _ThemeAppState();
}

class _ThemeAppState extends State<ThemeApp> {
  ThemeMode _themeMode = ThemeMode.system;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.light,
        ),
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.blue,
          brightness: Brightness.dark,
        ),
        useMaterial3: true,
      ),
      themeMode: _themeMode,
      home: Scaffold(
        body: Center(
          child: SegmentedButton<ThemeMode>(
            segments: const [
              ButtonSegment(value: ThemeMode.system, label: Text('跟随系统')),
              ButtonSegment(value: ThemeMode.light, label: Text('亮色')),
              ButtonSegment(value: ThemeMode.dark, label: Text('暗色')),
            ],
            selected: {_themeMode},
            onSelectionChanged: (modes) {
              setState(() => _themeMode = modes.first);
            },
          ),
        ),
      ),
    );
  }
}
```

**在代码中使用主题颜色：**

```dart
final colorScheme = Theme.of(context).colorScheme;
// colorScheme.primary      → 主色
// colorScheme.onPrimary    → 主色上的文字色
// colorScheme.surface      → 背景色
// colorScheme.onSurface    → 背景上的文字色
// colorScheme.error        → 错误色
```

### 7. LayoutBuilder 和 MediaQuery 有什么区别？分别在什么场景下使用？

**答案：**

| 维度 | MediaQuery | LayoutBuilder |
|------|-----------|---------------|
| 数据来源 | 全局屏幕信息 | 父组件传递的约束 |
| 获取时机 | 任何位置 | build 方法内 |
| 包含信息 | 屏幕尺寸、像素比、方向、安全区域 | 父组件的最大/最小宽高约束 |
| 适用场景 | 全局适配（字体、安全区域） | 局部适配（根据父容器大小调整布局） |
| 嵌套组件 | 取的是屏幕尺寸，非父容器尺寸 | 取的是真实父约束 |

**关键区别**：`MediaQuery` 获取的是**整个屏幕**的信息，而 `LayoutBuilder` 获取的是**父组件**的约束。在嵌套布局中，两者的值可能完全不同。

**使用场景：**

```dart
// 场景1：SafeArea 用 MediaQuery → 全局安全区域
SafeArea(child: ...)

// 场景2：根据屏幕尺寸切换布局 → MediaQuery
final width = MediaQuery.of(context).size.width;
if (width > 600) { /* 平板布局 */ }

// 场景3：组件内部根据可用空间自适应 → LayoutBuilder
LayoutBuilder(
  builder: (context, constraints) {
    // constraints.maxWidth 是父组件给的真实可用宽度
    // 即使这个组件在一个 200px 宽的侧边栏里
    // constraints.maxWidth 也是 200，而不是屏幕宽度
    if (constraints.maxWidth > 200) {
      return Row(children: [...]);
    } else {
      return Column(children: [...]);
    }
  },
)
```

**最佳实践**：如果组件需要根据自身可用空间自适应（而非屏幕大小），优先使用 `LayoutBuilder`，这样组件更通用、可复用。

### 8. Flutter 如何实现自定义布局？CustomMultiChildLayout 和 Flow 的区别是什么？

**答案：**

Flutter 提供了两种自定义布局方式：

**CustomMultiChildLayout：**

- 通过 `MultiChildLayoutDelegate` 实现
- 在 `performLayout` 中使用 `layoutChild` 和 `positionChild` 精确控制每个子组件
- 通过 `LayoutId` 为子组件标记 ID
- 适合精确控制位置和大小的复杂布局

**Flow：**

- 通过 `FlowDelegate` 实现
- 在 `paintChildren` 中使用 `paintChild` 绘制子组件
- 只控制位置（通过变换矩阵），不控制大小
- 性能更好（只重绘变化的子组件）
- 适合需要动画效果的布局变换

**对比：**

| 维度 | CustomMultiChildLayout | Flow |
|------|----------------------|------|
| 控制能力 | 位置 + 大小 | 仅位置（通过变换） |
| 实现方式 | performLayout + layoutChild/positionChild | paintChildren + paintChild |
| 性能 | 一般 | 更好（优化重绘） |
| 子组件标识 | LayoutId | 索引 |
| 适用场景 | 精确布局（图表、日历） | 动画布局（展开/折叠、过渡） |
| 学习成本 | 较高 | 中等 |

**选择建议**：需要精确控制位置和大小用 `CustomMultiChildLayout`，需要动画过渡效果用 `Flow`。大部分场景用 `Stack` + `Positioned` 或 `CustomScrollView` 即可满足，自定义布局是最后的选项。

## 相关链接

- [[Flutter核心与Widget体系]] — Flutter Widget 生命周期、State 管理、Key 机制
- [[Flutter国际化与主题]] — 国际化配置、主题切换、本地化适配
- [[CSS布局Flex与Grid]] — Web 端 Flex/Grid 布局，与 Flutter Flex 布局对比理解
- [[响应式设计]] — 多端适配策略、断点设计、自适应布局方案

---
tags:
  - Python
  - 数据科学
  - Matplotlib
  - 可视化
---

# Matplotlib与数据可视化

## What — Matplotlib 是什么

Matplotlib 是 Python 最基础也最全面的绘图库，由 John Hunter 于 2003 年创建，灵感来自 MATLAB 的绘图系统。它提供了从简单折线图到复杂 3D 曲面的全品类图表支持，是 Python 数据可视化生态的基石——[[Seaborn与统计可视化]]、Pandas 内置绘图都基于 Matplotlib 构建。

核心架构分为三层：

| 层级 | 说明 | 用户接触频率 |
|------|------|-------------|
| Backend 层 | 渲染引擎（Agg、Qt、TkAgg 等） | 极少，配置输出格式时接触 |
| Artist 层 | 所有可见元素的基类（Figure、Axes、Line2D 等） | 高级自定义时使用 |
| Scripting 层 | `pyplot` 接口，类似 MATLAB 的快速绑定 | 日常使用最多 |

```python
import matplotlib.pyplot as plt
import numpy as np

# 最简单的绘图
x = np.linspace(0, 2 * np.pi, 100)
y = np.sin(x)

plt.plot(x, y)
plt.title('正弦函数')
plt.xlabel('x')
plt.ylabel('sin(x)')
plt.savefig('sine.png', dpi=150, bbox_inches='tight')
plt.show()
```

## Why — 为什么需要 Matplotlib

### 数据可视化的核心价值

数据科学项目中，可视化承担三个关键任务：**探索**（EDA阶段发现模式与异常）、**验证**（检查模型输出是否合理）、**沟通**（向利益相关者传达结论）。Matplotlib 覆盖了所有三个阶段的图表需求。

### Matplotlib 的生态地位

| 工具 | 定位 | 与 Matplotlib 关系 |
|------|------|-------------------|
| Matplotlib | 全品类基础库 | — |
| Seaborn | 统计可视化 | 基于 Matplotlib |
| Pandas plot | 快速绘图 | 底层调用 Matplotlib |
| Plotly | 交互式图表 | 独立，可互转 |
| Bokeh | 浏览器交互 | 独立 |
| Altair | 声明式可视化 | 独立（基于 Vega-Lite） |

选择逻辑：快速探索用 Pandas plot，统计图表用 Seaborn，需要精细控制用 Matplotlib，交互需求用 Plotly。Matplotlib 是必须掌握的基础，因为其他库的控制最终都回溯到 Matplotlib 的 Artist 层。

## How — 如何使用 Matplotlib

### 1. 两种接口：pyplot 与面向对象

Matplotlib 提供两种绘图接口，理解它们的区别是避免混乱的关键。

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.cos(x)

# === 接口一：pyplot（MATLAB 风格） ===
# 简单快速，但有隐式状态
plt.figure(figsize=(8, 4))
plt.plot(x, y1, label='sin(x)')
plt.plot(x, y2, label='cos(x)')
plt.title('三角函数')
plt.legend()
plt.show()

# === 接口二：面向对象（推荐） ===
# 显式控制，适合复杂图表
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, y1, label='sin(x)')
ax.plot(x, y2, label='cos(x)')
ax.set_title('三角函数')
ax.legend()
plt.show()

# === 为什么推荐面向对象 ===
# 1. 多子图时 pyplot 容易混乱
# 2. 在函数/类中复用更安全
# 3. 对 Artist 的精细控制需要 ax 引用
# 4. 与 Seaborn 配合更自然
```

| 对比项 | pyplot 接口 | 面向对象接口 |
|--------|-------------|-------------|
| 代码风格 | 命令式，有隐式状态 | 显式，无隐式状态 |
| 适用场景 | 快速探索、单图 | 多子图、精细控制、复用 |
| 方法命名 | `plt.title()` | `ax.set_title()` |
| 线条添加 | `plt.plot()` | `ax.plot()` |
| 推荐度 | 简单场景可用 | **推荐默认使用** |

### 2. Figure 与 Axes 的结构

理解 Matplotlib 的对象模型是自定义图表的基础。

```
Figure（画布）
├── Axes 1（子图区域，包含坐标轴）
│   ├── XAxis（x 轴）
│   ├── YAxis（y 轴）
│   ├── Line2D / Bar / Scatter（数据元素）
│   ├── Legend（图例）
│   ├── Title（标题）
│   └── Text / Annotation（文本标注）
├── Axes 2
└── ...
```

```python
import matplotlib.pyplot as plt
import numpy as np

# === 创建 Figure 和 Axes ===

# 单图
fig, ax = plt.subplots(figsize=(10, 6))

# 等长子图
fig, axes = plt.subplots(2, 3, figsize=(12, 8))  # 2行3列
# axes 是 (2, 3) 的 ndarray
axes[0, 0].plot([1, 2, 3])
axes[1, 2].scatter([1, 2], [3, 4])

# 不等长子图（GridSpec）
fig = plt.figure(figsize=(12, 8))
gs = fig.add_gridspec(3, 3)

ax1 = fig.add_subplot(gs[0, :])      # 第1行占满
ax2 = fig.add_subplot(gs[1, :2])     # 第2行前2列
ax3 = fig.add_subplot(gs[1:, 2])     # 第2-3行第3列
ax4 = fig.add_subplot(gs[2, :2])     # 第3行前2列

ax1.set_title('宽图')
ax2.set_title('左中')
ax3.set_title('右侧')
ax4.set_title('左下')

plt.tight_layout()  # 自动调整间距
plt.show()

# === 手动控制间距 ===
fig, axes = plt.subplots(2, 2)
fig.subplots_adjust(
    left=0.1,     # 左边距
    right=0.95,   # 右边距
    bottom=0.1,   # 底边距
    top=0.9,      # 顶边距
    wspace=0.3,   # 水平间距
    hspace=0.4    # 垂直间距
)
```

### 3. 基础图表类型

```python
import matplotlib.pyplot as plt
import numpy as np

# === 折线图 ===
fig, ax = plt.subplots(figsize=(10, 5))
x = np.linspace(0, 10, 50)

ax.plot(x, np.sin(x), 'b-', label='sin', linewidth=2)
ax.plot(x, np.cos(x), 'r--', label='cos', linewidth=1.5)
ax.plot(x, np.sin(x) * np.cos(x), 'g:', label='sin·cos', alpha=0.7)
ax.set_title('折线图示例')
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.legend(loc='upper right')
ax.grid(True, alpha=0.3)
plt.show()

# === 柱状图 ===
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

categories = ['A', 'B', 'C', 'D']
values = [23, 45, 56, 78]

# 竖直柱状图
axes[0].bar(categories, values, color=['#4C72B0', '#55A868', '#C44E52', '#8172B2'])
axes[0].set_title('竖直柱状图')

# 水平柱状图
axes[1].barh(categories, values, color='#4C72B0')
axes[1].set_title('水平柱状图')

# 添加数值标签
for i, v in enumerate(values):
    axes[0].text(i, v + 1, str(v), ha='center')
    axes[1].text(v + 1, i, str(v), va='center')

plt.tight_layout()
plt.show()

# === 分组柱状图 ===
fig, ax = plt.subplots(figsize=(8, 5))
x = np.arange(4)
width = 0.3
scores_2024 = [70, 82, 65, 90]
scores_2025 = [75, 85, 72, 88]

ax.bar(x - width/2, scores_2024, width, label='2024', color='#4C72B0')
ax.bar(x + width/2, scores_2025, width, label='2025', color='#DD8452')
ax.set_xticks(x)
ax.set_xticklabels(['语文', '数学', '英语', '物理'])
ax.legend()
ax.set_title('分组柱状图')
plt.show()

# === 散点图 ===
rng = np.random.default_rng(42)
n = 200
x = rng.standard_normal(n)
y = 0.8 * x + rng.standard_normal(n) * 0.5
colors = rng.random(n)
sizes = rng.random(n) * 200

fig, ax = plt.subplots(figsize=(8, 6))
scatter = ax.scatter(x, y, c=colors, s=sizes, alpha=0.6, cmap='viridis', edgecolors='gray')
fig.colorbar(scatter, label='颜色值')
ax.set_title('散点图（带颜色和大小映射）')
ax.set_xlabel('x')
ax.set_ylabel('y')
plt.show()

# === 直方图 ===
data = rng.normal(0, 1, 1000)

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].hist(data, bins=30, color='#4C72B0', edgecolor='white')
axes[0].set_title('基础直方图')

axes[1].hist(data, bins=30, density=True, color='#4C72B0',
             edgecolor='white', alpha=0.7, label='频数')
axes[1].set_title('归一化直方图 + KDE')
plt.show()

# === 饼图 ===
fig, ax = plt.subplots(figsize=(8, 8))
labels = ['产品A', '产品B', '产品C', '产品D']
sizes = [35, 30, 20, 15]
explode = (0.05, 0, 0, 0)

ax.pie(sizes, explode=explode, labels=labels, autopct='%1.1f%%',
       startangle=90, colors=['#4C72B0', '#55A868', '#C44E52', '#8172B2'])
ax.set_title('市场份额')
plt.show()

# === 箱线图 ===
data = [rng.normal(0, std, 100) for std in [1, 1.5, 2, 2.5]]

fig, ax = plt.subplots(figsize=(8, 5))
bp = ax.boxplot(data, labels=['组1', '组2', '组3', '组4'],
                patch_artist=True,
                boxprops=dict(facecolor='#4C72B0', alpha=0.7),
                medianprops=dict(color='red', linewidth=2))
ax.set_title('箱线图')
ax.set_ylabel('值')
plt.show()

# === 热力图 ===
matrix = np.random.default_rng(42).random((8, 8))
fig, ax = plt.subplots(figsize=(8, 6))
im = ax.imshow(matrix, cmap='YlOrRd')
fig.colorbar(im, label='值')
ax.set_title('热力图')

# 添加数值标注
for i in range(8):
    for j in range(8):
        ax.text(j, i, f'{matrix[i, j]:.2f}', ha='center', va='center',
                color='white' if matrix[i, j] > 0.5 else 'black', fontsize=8)
plt.show()
```

### 4. 样式与美化

```python
import matplotlib.pyplot as plt
import numpy as np

# === 内置样式 ===
print(plt.style.available)
# 常用：'seaborn-v0_8', 'ggplot', 'dark_background', 'fivethirtyeight'

plt.style.use('seaborn-v0_8-whitegrid')  # 全局设置

# 临时样式（推荐，不污染全局）
with plt.style.context('dark_background'):
    fig, ax = plt.subplots()
    ax.plot(np.sin(np.linspace(0, 10, 100)))
    plt.show()

# === 自定义 rcParams ===
plt.rcParams.update({
    'font.family': 'sans-serif',
    'font.size': 12,
    'axes.titlesize': 16,
    'axes.labelsize': 14,
    'figure.figsize': (10, 6),
    'figure.dpi': 100,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
})

# === 颜色方案 ===
# 方式1：颜色名称 / 十六进制
colors = ['blue', '#FF5733', (0.1, 0.2, 0.5), '#4C72B0']

# 方式2：colormap
cmap = plt.cm.viridis
n = 5
colors = [cmap(i / n) for i in range(n)]

# 方式3：颜色循环（Category20）
from matplotlib.colors import ListedColormap
tab10 = plt.cm.tab10

# === 标注与注释 ===
fig, ax = plt.subplots(figsize=(10, 5))
x = np.linspace(0, 10, 100)
y = np.sin(x)
ax.plot(x, y)

# 指向数据点的箭头标注
ax.annotate('最大值', xy=(np.pi/2, 1), xytext=(3, 1.3),
            arrowprops=dict(arrowstyle='->', color='red', lw=2),
            fontsize=12, color='red')

# 普通文本
ax.text(5, -0.5, '正弦波', fontsize=14, ha='center',
        bbox=dict(boxstyle='round,pad=0.3', facecolor='yellow', alpha=0.5))

plt.show()

# === 双Y轴 ===
fig, ax1 = plt.subplots(figsize=(10, 5))

x = np.arange(12)
revenue = [20, 25, 30, 28, 35, 40, 38, 45, 50, 48, 55, 60]
growth = [10, 25, 20, -7, 25, 14, -5, 18, 11, -4, 15, 9]

ax1.bar(x, revenue, color='#4C72B0', alpha=0.7, label='营收（万元）')
ax1.set_ylabel('营收（万元）', color='#4C72B0')

ax2 = ax1.twinx()
ax2.plot(x, growth, color='#C44E52', marker='o', label='增长率（%）')
ax2.set_ylabel('增长率（%）', color='#C44E52')

ax1.set_xticks(x)
ax1.set_xticklabels([f'{m}月' for m in range(1, 13)])
ax1.set_title('月度营收与增长率')

# 合并图例
lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc='upper left')

plt.show()
```

### 5. 子图布局进阶

```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import numpy as np

# === GridSpec 精细控制 ===
fig = plt.figure(figsize=(14, 10))
gs = gridspec.GridSpec(3, 3, figure=fig, hspace=0.4, wspace=0.3)

# 大图占2列
ax_main = fig.add_subplot(gs[0, :2])
ax_main.plot(np.random.randn(50).cumsum())
ax_main.set_title('主趋势图')

# 侧边分布图
ax_side = fig.add_subplot(gs[0, 2])
ax_side.hist(np.random.randn(100), bins=20, orientation='horizontal')
ax_side.set_title('分布')

# 下方3等分
for i in range(3):
    ax = fig.add_subplot(gs[1, i])
    ax.scatter(np.random.randn(30), np.random.randn(30))
    ax.set_title(f'散点 {i+1}')

# 底部宽图
ax_bottom = fig.add_subplot(gs[2, :])
ax_bottom.imshow(np.random.randn(20, 20), cmap='coolwarm')
ax_bottom.set_title('热力图')

plt.show()

# === inset_axes 嵌入小图 ===
fig, ax = plt.subplots(figsize=(10, 6))
x = np.linspace(0, 10, 200)
ax.plot(x, np.sin(x) * np.exp(-0.1 * x), 'b-', linewidth=2)
ax.set_title('衰减振荡（右上角放大）')

# 创建嵌入小图
from mpl_toolkits.axes_grid1.inset_locator import inset_axes
ax_inset = inset_axes(ax, width="35%", height="35%", loc='upper right')
mask = (x > 4) & (x < 6)
ax_inset.plot(x[mask], np.sin(x[mask]) * np.exp(-0.1 * x[mask]), 'r-', linewidth=2)
ax_inset.set_title('4 < x < 6', fontsize=8)

plt.show()

# === 共享坐标轴 ===
fig, axes = plt.subplots(3, 1, figsize=(10, 8), sharex=True)

x = np.linspace(0, 10, 100)
for i, ax in enumerate(axes):
    ax.plot(x, np.sin(x + i), label=f'相位 {i}')
    ax.set_ylabel(f'y{i}')

axes[-1].set_xlabel('x')
axes[0].set_title('共享X轴的三子图')
fig.legend(loc='upper right', bbox_to_anchor=(1.0, 1.0))

plt.tight_layout()
plt.show()
```

### 6. 动画

Matplotlib 支持逐帧动画，可用于动态数据展示和教学演示。

```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(8, 5))
x = np.linspace(0, 2 * np.pi, 200)
line, = ax.plot(x, np.sin(x))
ax.set_ylim(-1.5, 1.5)
ax.set_title('正弦波动画')

def animate(frame):
    line.set_ydata(np.sin(x + frame * 0.05))
    return line,

# FuncAnimation: 每帧调用 animate 函数
anim = animation.FuncAnimation(
    fig, animate, frames=200, interval=30, blit=True
)

# 保存为 GIF
anim.save('sine_wave.gif', writer='pillow', fps=30)

# 保存为 MP4（需要 ffmpeg）
# anim.save('sine_wave.mp4', writer='ffmpeg', fps=30)

plt.show()
```

### 7. 3D 绘图

```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import numpy as np

# === 3D 曲面 ===
fig = plt.figure(figsize=(10, 7))
ax = fig.add_subplot(111, projection='3d')

u = np.linspace(-2, 2, 100)
v = np.linspace(-2, 2, 100)
X, Y = np.meshgrid(u, v)
Z = np.sin(np.sqrt(X**2 + Y**2))

surf = ax.plot_surface(X, Y, Z, cmap='coolwarm', alpha=0.8,
                        edgecolor='none')
fig.colorbar(surf, shrink=0.5, aspect=10)
ax.set_title('3D 曲面: $z = \sin(\sqrt{x^2 + y^2})$')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
plt.show()

# === 3D 散点 ===
rng = np.random.default_rng(42)
n = 200
x = rng.standard_normal(n)
y = rng.standard_normal(n)
z = rng.standard_normal(n)
colors = rng.random(n)

fig = plt.figure(figsize=(10, 7))
ax = fig.add_subplot(111, projection='3d')
scatter = ax.scatter(x, y, z, c=colors, cmap='viridis', s=30, alpha=0.6)
fig.colorbar(scatter)
ax.set_title('3D 散点图')
plt.show()

# === 3D 线框 ===
fig = plt.figure(figsize=(10, 7))
ax = fig.add_subplot(111, projection='3d')
ax.plot_wireframe(X, Y, Z, color='steelblue', linewidth=0.5)
ax.set_title('线框图')
plt.show()
```

### 8. 输出与保存

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots()
ax.plot(np.random.randn(50).cumsum())

# === 保存为文件 ===
fig.savefig('output.png', dpi=300, bbox_inches='tight', facecolor='white')
fig.savefig('output.pdf', bbox_inches='tight')  # 矢量格式，论文首选
fig.savefig('output.svg', bbox_inches='tight')  # 网页矢量格式
fig.savefig('output.jpg', dpi=150, quality=95)   # 照片格式

# === 配置后端 ===
# 非交互环境（服务器）使用 Agg 后端
import matplotlib
matplotlib.use('Agg')  # 必须在 import pyplot 之前

# Jupyter Notebook 内联显示
# %matplotlib inline — 静态图
# %matplotlib widget — 交互图（需要 ipympl）

# === 中文字体配置 ===
plt.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False  # 负号正常显示

fig, ax = plt.subplots()
ax.set_title('中文标题测试')
ax.plot([1, 2, 3], [4, 5, 6], label='数据系列')
ax.legend()
plt.show()
```

| 输出格式 | 说明 | 适用场景 |
|----------|------|---------|
| PNG | 位图，支持透明 | 网页、PPT |
| PDF | 矢量，可无限缩放 | 论文、印刷 |
| SVG | 矢量，浏览器原生支持 | 网页交互 |
| JPG | 有损压缩，无透明 | 照片类图表 |
| GIF | 动画 | 动态演示 |
| MP4 | 视频 | 长动画 |

## 面试题

### 1. Matplotlib 的 pyplot 接口和面向对象接口有什么区别？

pyplot 是 MATLAB 风格的命令式接口，通过 `plt.plot()`、`plt.title()` 等函数操作当前活跃的 Figure/Axes，存在隐式状态，简单场景下代码简洁但多子图时容易混乱。面向对象接口通过 `fig, ax = plt.subplots()` 显式获取对象引用，再用 `ax.plot()`、`ax.set_title()` 操作，无隐式状态，适合多子图、函数封装和精细控制。官方推荐面向对象接口作为默认选择。

### 2. Figure 和 Axes 是什么关系？

Figure 是整个画布（顶层容器），Axes 是画布上的一个绘图区域（不是"坐标轴"，坐标轴是 XAxis/YAxis）。一个 Figure 可以包含多个 Axes，每个 Axes 有自己独立的坐标系、标题、图例等。创建方式：`fig, ax = plt.subplots()` 创建1个 Axes，`fig, axes = plt.subplots(2, 3)` 创建6个 Axes。Figure 管理画布大小、输出格式，Axes 管理具体的绘图内容。

### 3. 如何绘制双Y轴图表？有什么注意事项？

使用 `ax2 = ax1.twinx()` 创建共享X轴的第二个Y轴。ax1 用左侧Y轴，ax2 用右侧Y轴。注意事项：（1）两个轴的颜色要区分开，用 `color` 参数和 `set_ylabel(color=...)` 保持一致；（2）图例需要手动合并 `ax1.get_legend_handles_labels()` 和 `ax2.get_legend_handles_labels()`；（3）`tight_layout()` 可能对双Y轴支持不好，需要手动调整；（4）数据量级差异大时考虑对数坐标。

### 4. Matplotlib 如何处理中文字体？

默认不支持中文，需要配置 `plt.rcParams['font.sans-serif']` 指定中文字体（Windows 用 SimHei/Microsoft YaHei，macOS 用 PingFang SC/Arial Unicode MS，Linux 用 WenQuanYi Micro Hei）。同时设置 `plt.rcParams['axes.unicode_minus'] = False` 解决负号显示为方块的问题。也可以通过 `matplotlib.font_manager` 动态注册字体文件，或在代码中用 `fontproperties` 参数逐个指定。

### 5. 什么是 blitting？动画中如何使用？

Blitting 是一种只重绘变化部分、保留不变背景的渲染优化技术。在 `FuncAnimation` 中设置 `blit=True` 后，animate 函数只需返回变化的 Artist 列表，Matplotlib 会缓存背景只更新这些 Artist，大幅提升帧率。前提是 animate 函数必须返回可迭代的 Artist 对象，且不能修改背景元素。适合折线动画、散点动画等局部更新场景，不适合整体布局变化的动画。

### 6. 如何在一张图上展示4种不同类型的图表？

使用 `plt.subplots(2, 2)` 创建 2×2 的 Axes 网格，然后在不同 Axes 上调用不同绘图方法：`axes[0,0].plot()` 折线图、`axes[0,1].bar()` 柱状图、`axes[1,0].scatter()` 散点图、`axes[1,1].pie()` 饼图。如果需要不等大小，用 `GridSpec` 精细控制：`gs[0, :2]` 让大图占满前两列。最后用 `plt.tight_layout()` 自动调整间距避免重叠。

### 7. savefig 时 bbox_inches='tight' 的作用是什么？

`bbox_inches='tight'` 会自动裁剪输出图片的边界，去除图表周围的多余空白，只保留有内容的区域。默认行为是按 Figure 的精确尺寸输出，可能有大量空白边距。配合 `pad_inches` 参数可控制裁剪后的内边距。注意：tight 裁剪每次结果可能略有差异，对精确尺寸有要求的场景（如论文模板）不建议使用，而应通过 `subplots_adjust` 手动控制边距。

### 8. imshow 和 pcolormesh 有什么区别？

两者都用于绘制二维数组的热力图，但有重要差异：`imshow` 将数组当作图像处理，每个数据点对应一个像素，按数组索引定位，纵轴从上到下（可通过 `origin='lower'` 翻转），适合规则网格和图像数据；`pcolormesh` 将数据映射到用户指定的坐标网格上，支持非均匀间距的 X/Y 坐标，数据点定义在格子角上而非中心，适合地理/科学数据中的非均匀网格。性能上 `pcolormesh` 比 `pcolor` 快得多（类似 scatter vs plot 的关系）。

## 外部参考

- [Matplotlib 官方文档](https://matplotlib.org/stable/contents.html)

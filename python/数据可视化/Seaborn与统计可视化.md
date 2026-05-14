---
tags:
  - Python
  - 数据科学
  - Seaborn
  - 统计可视化
---

# Seaborn与统计可视化

## What — Seaborn 是什么

Seaborn 是基于 [[Matplotlib与数据可视化]] 的 Python 统计可视化库，由 Michael Waskom 创建和维护。它在 Matplotlib 的 Artist 层之上提供了面向统计数据的高级接口，用更少的代码实现更专业的统计图表，特别擅长展示数据分布、变量关系和分类比较。

Seaborn 的核心优势：

| 特性 | 说明 |
|------|------|
| 统计图表封装 | 一行代码完成分布图、回归图、热力图等 |
| 内置数据集 | tips、penguins、flights 等练习数据 |
| Pandas 深度集成 | 直接传入 DataFrame，用列名指定轴 |
| 自动统计估计 | 均值置信区间、核密度估计、回归拟合 |
| 美观默认样式 | 5种内置主题，专业配色方案 |
| 分面绘图 | 按分类变量自动拆分子图 |

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Seaborn 的典型用法：传入 DataFrame + 列名
tips = sns.load_dataset('tips')
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='time')
plt.show()

# 等价的纯 Matplotlib 需要 ~10 行
```

## Why — 为什么需要 Seaborn

### Matplotlib 的不足

Matplotlib 是底层绘图库，做统计图表时需要大量手动计算和配置：置信区间要自己算、核密度估计要自己调、分面绘图要手动创建 GridSpec、配色要自己选。这些重复工作正是 Seaborn 封装的内容。

### Seaborn vs Matplotlib vs 其他

| 场景 | Matplotlib | Seaborn | Plotly |
|------|-----------|---------|--------|
| 基础折线/散点 | 原生支持 | 也可但非优势 | 交互版 |
| 统计分布 | 手动计算+绘图 | 一行搞定 | 有限 |
| 分类比较 | 手动排布 | 自动分面 | 有限 |
| 回归分析 | 手动拟合+绘图 | 自动拟合+置信区间 | 无 |
| 热力图+聚类 | 手动 | 聚类热力图 | 简单热力图 |
| 交互需求 | 不支持 | 不支持 | 原生支持 |
| 精细控制 | 完全控制 | 可回退到 Matplotlib | 中等 |

Seaborn 不替代 Matplotlib，而是建立在它之上。Seaborn 生成 Matplotlib 对象，可以用 Matplotlib API 进一步调整。

## How — 如何使用 Seaborn

### 1. 样式与主题

Seaborn 提供了开箱即用的美观样式，让 Matplotlib 图表瞬间专业。

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# === 五种主题 ===
# darkgrid（默认）, whitegrid, dark, white, ticks
sns.set_theme(style='whitegrid', font_scale=1.2)

# === 两种上下文 ===
# paper, notebook（默认）, talk, poster — 控制元素大小
sns.set_context('talk')

# === 配色方案 ===
# 定性色板（分类数据）
palette_qual = sns.color_palette('Set2', 8)

# 序列色板（连续数据）
palette_seq = sns.color_palette('Blues', 8)

# 发散色板（有正负的数据）
palette_div = sns.color_palette('RdBu', 8)

# 预览色板
sns.palplot(palette_qual)
plt.show()

# 在绘图中使用
tips = sns.load_dataset('tips')
sns.scatterplot(data=tips, x='total_bill', y='tip',
                hue='day', palette='Set2')
plt.show()

# 自定义色板
custom = ['#264653', '#2a9d8f', '#e9c46a', '#f4a261', '#e76f51']
sns.set_palette(custom)
```

| 色板类型 | 适用数据 | 常用名称 |
|----------|---------|---------|
| 定性 | 分类变量 | Set1/Set2, Paired, husl |
| 序列 | 连续变量（单向） | Blues, Greens, OrRd |
| 发散 | 有正负的连续变量 | RdBu, PRGn, coolwarm |
| 暗色 | 深色背景 | dark:medal, dark:salmon |

### 2. 分布图

分布图是 EDA 的第一步，Seaborn 提供了完整的分布可视化工具链。

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

tips = sns.load_dataset('tips')
rng = np.random.default_rng(42)

# === 直方图 + KDE ===
fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# 基础直方图
sns.histplot(data=tips, x='total_bill', ax=axes[0])
axes[0].set_title('直方图')

# 直方图 + KDE 曲线
sns.histplot(data=tips, x='total_bill', kde=True, ax=axes[1])
axes[1].set_title('直方图 + KDE')

# 仅 KDE
sns.kdeplot(data=tips, x='total_bill', fill=True, ax=axes[2])
axes[2].set_title('KDE 密度图')

plt.tight_layout()
plt.show()

# === 分组分布 ===
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# 叠加分组 KDE
sns.kdeplot(data=tips, x='total_bill', hue='time', fill=True, alpha=0.5, ax=axes[0])
axes[0].set_title('分组KDE')

# 分面直方图
sns.histplot(data=tips, x='total_bill', hue='time', multiple='stack', ax=axes[1])
axes[1].set_title('堆叠直方图')

plt.tight_layout()
plt.show()

# === 二维分布 ===
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# 二维直方图
sns.histplot(data=tips, x='total_bill', y='tip', ax=axes[0], cbar=True)
axes[0].set_title('二维直方图')

# 二维 KDE
sns.kdeplot(data=tips, x='total_bill', y='tip', fill=True, ax=axes[1])
axes[1].set_title('二维 KDE')

# 散点 + 边际分布
sns.kdeplot(data=tips, x='total_bill', y='tip', ax=axes[2])
sns.rugplot(data=tips, x='total_bill', y='tip', ax=axes[2])
axes[2].set_title('KDE + Rug')

plt.tight_layout()
plt.show()

# === jointplot：散点 + 边际分布 ===
sns.jointplot(data=tips, x='total_bill', y='tip', kind='reg')  # 回归
plt.show()

sns.jointplot(data=tips, x='total_bill', y='tip', kind='hex')  # 六角箱
plt.show()

sns.jointplot(data=tips, x='total_bill', y='tip', kind='kde')  # 密度
plt.show()

# === pairplot：多变量两两关系 ===
penguins = sns.load_dataset('penguins')
sns.pairplot(penguins, hue='species', diag_kind='kde',
             vars=['bill_length_mm', 'bill_depth_mm', 'flipper_length_mm'])
plt.show()
```

| 函数 | 图表类型 | 适用场景 |
|------|---------|---------|
| histplot | 直方图 | 单变量频率分布 |
| kdeplot | 核密度估计 | 平滑分布形状 |
| ecdfplot | 经验累积分布 | 分位数、百分位 |
| rugplot | 地毯图 | 辅助显示数据点位置 |
| jointplot | 散点+边际分布 | 双变量关系 |
| pairplot | 散点矩阵 | 多变量两两关系 |

### 3. 关系图

关系图用于展示变量之间的关联模式。

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

tips = sns.load_dataset('tips')

# === 散点图 ===
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# 基础散点
sns.scatterplot(data=tips, x='total_bill', y='tip', ax=axes[0])
axes[0].set_title('基础散点')

# 多维度编码：颜色、大小、样式
sns.scatterplot(data=tips, x='total_bill', y='tip',
                hue='time', size='size', style='smoker', ax=axes[1])
axes[1].set_title('多维编码')

# 带回归拟合
sns.regplot(data=tips, x='total_bill', y='tip', ax=axes[2],
            scatter_kws={'alpha': 0.5}, line_kws={'color': 'red'})
axes[2].set_title('回归拟合')

plt.tight_layout()
plt.show()

# === lmplot：回归 + 分面 ===
sns.lmplot(data=tips, x='total_bill', y='tip',
           col='time', row='smoker',  # 按时间和是否吸烟分面
           height=4, aspect=1.2,
           scatter_kws={'alpha': 0.5})
plt.show()

# === 线图 ===
flights = sns.load_dataset('flights')

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 单线图
sns.lineplot(data=flights, x='year', y='passengers',
             estimator='mean', errorbar='sd', ax=axes[0])
axes[0].set_title('年度趋势（均值±标准差）')

# 多线图
sns.lineplot(data=flights, x='year', y='passengers',
             hue='month', ax=axes[1], legend=False)
axes[1].set_title('月度趋势')

plt.tight_layout()
plt.show()

# === relplot：关系图 + 分面 ===
sns.relplot(data=tips, x='total_bill', y='tip',
            col='time', hue='smoker', style='smoker',
            kind='scatter', height=5)
plt.show()
```

| 函数 | 图表类型 | 核心参数 |
|------|---------|---------|
| scatterplot | 散点图 | hue, size, style |
| lineplot | 线图 | estimator, errorbar |
| regplot | 回归图 | order, ci, robust |
| lmplot | 回归+分面 | col, row, hue |
| relplot | 关系+分面 | kind='scatter'/'line' |

### 4. 分类图

分类图用于比较不同组别的统计量。

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# === 箱线图 ===
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('箱线图')

# 分组箱线图
sns.boxplot(data=tips, x='day', y='total_bill', hue='time', ax=axes[1])
axes[1].set_title('分组箱线图')

# 小提琴图（箱线+KDE）
sns.violinplot(data=tips, x='day', y='total_bill', hue='time',
               split=True, ax=axes[2])
axes[2].set_title('小提琴图')

plt.tight_layout()
plt.show()

# === 条形图 ===
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# 均值+置信区间
sns.barplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('均值条形图')

# 计数图
sns.countplot(data=tips, x='day', hue='time', ax=axes[1])
axes[1].set_title('计数图')

# 点图
sns.pointplot(data=tips, x='day', y='total_bill', hue='time',
              markers=['o', 's'], ax=axes[2])
axes[2].set_title('点图')

plt.tight_layout()
plt.show()

# === swarm 图（蜂群图，避免重叠） ===
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

sns.swarmplot(data=tips, x='day', y='total_bill', ax=axes[0], size=3)
axes[0].set_title('蜂群图')

# 箱线 + 蜂群叠加
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[1],
            whis=[5, 95], showfliers=False)
sns.swarmplot(data=tips, x='day', y='total_bill', ax=axes[1],
              color='.25', size=3)
axes[1].set_title('箱线 + 蜂群')

plt.tight_layout()
plt.show()

# === catplot：分类图 + 分面 ===
sns.catplot(data=tips, x='day', y='total_bill',
            col='time', row='smoker',
            kind='box', height=4, aspect=1.2)
plt.show()
```

| 函数 | 图表类型 | 展示内容 |
|------|---------|---------|
| boxplot | 箱线图 | 四分位+离群值 |
| violinplot | 小提琴图 | 分布形状+四分位 |
| barplot | 条形图 | 均值+置信区间 |
| countplot | 计数图 | 各类样本数 |
| pointplot | 点图 | 均值趋势+误差 |
| swarmplot | 蜂群图 | 所有点无重叠 |
| stripplot | 条带图 | 所有点（有重叠） |
| catplot | 分类+分面 | kind 指定子类型 |

### 5. 热力图与矩阵图

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# === 相关性热力图 ===
penguins = sns.load_dataset('penguins')
numeric_cols = penguins.select_dtypes(include='number').columns
corr = penguins[numeric_cols].corr()

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 基础热力图
sns.heatmap(corr, annot=True, fmt='.2f', cmap='RdBu_r',
            center=0, vmin=-1, vmax=1, ax=axes[0],
            square=True, linewidths=0.5)
axes[0].set_title('相关性矩阵')

# 聚类热力图
sns.clustermap(corr, annot=True, fmt='.2f', cmap='RdBu_r',
               center=0, figsize=(8, 8))
plt.show()

# === 透视表热力图 ===
flights = sns.load_dataset('flights')
pivot = flights.pivot(index='month', columns='year', values='passengers')

fig, ax = plt.subplots(figsize=(12, 8))
sns.heatmap(pivot, annot=True, fmt='d', cmap='YlOrRd',
            linewidths=0.5, ax=ax)
ax.set_title('月度乘客数热力图')
plt.show()

# === 自定义热力图 ===
data = np.random.default_rng(42).standard_normal((10, 10))
fig, ax = plt.subplots(figsize=(10, 8))

# 仅显示显著相关
mask = np.abs(data) < 1.0  # 模拟掩码
sns.heatmap(data, mask=mask, annot=True, fmt='.2f',
            cmap='coolwarm', center=0, ax=ax,
            cbar_kws={'label': '相关系数', 'shrink': 0.8})
ax.set_title('带掩码的热力图')
plt.show()
```

| 函数 | 说明 | 特色参数 |
|------|------|---------|
| heatmap | 基础热力图 | annot, fmt, mask, center |
| clustermap | 聚类热力图 | method, metric, z_score |

### 6. 分面绘图（FacetGrid）

FacetGrid 是 Seaborn 最强大的功能之一，按分类变量自动拆分子图。

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# === FacetGrid 手动控制 ===
g = sns.FacetGrid(tips, col='time', row='smoker',
                   height=4, aspect=1.2,
                   margin_titles=True)

# map 方法传入绘图函数
g.map(sns.histplot, 'total_bill', kde=True)
g.set_axis_labels('总账单', '频数')
g.set_titles(col_template='{col_name}', row_template='吸烟: {row_name}')
g.add_legend()
plt.show()

# === map_dataframe 更灵活 ===
g = sns.FacetGrid(tips, col='day', col_wrap=2, height=4)
g.map_dataframe(sns.scatterplot, x='total_bill', y='tip', hue='sex')
g.add_legend()
g.set_axis_labels('总账单', '小费')
plt.show()

# === PairGrid 自定义配对图 ===
penguins = sns.load_dataset('penguins')
g = sns.PairGrid(penguins, hue='species',
                  vars=['bill_length_mm', 'bill_depth_mm', 'flipper_length_mm'])
g.map_diag(sns.kdeplot)           # 对角线：KDE
g.map_offdiag(sns.scatterplot)    # 非对角线：散点
g.add_legend()
plt.show()

# === JointGrid ===
g = sns.JointGrid(data=tips, x='total_bill', y='tip', hue='time')
g.plot(sns.scatterplot, sns.histplot)
plt.show()
```

| Grid 类型 | 说明 | 分面方式 |
|-----------|------|---------|
| FacetGrid | 按分类变量分面 | col/row 指定变量 |
| PairGrid | 多变量两两配对 | vars 指定变量列表 |
| JointGrid | 双变量+边际 | 两个变量 |

### 7. 统计估计与回归

Seaborn 内置了统计估计功能，自动计算置信区间和拟合回归线。

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

tips = sns.load_dataset('tips')

# === 回归图 ===
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# 线性回归 + 95% 置信区间
sns.regplot(data=tips, x='total_bill', y='tip', ax=axes[0, 0])
axes[0, 0].set_title('线性回归')

# 多项式回归
sns.regplot(data=tips, x='total_bill', y='tip', order=2, ax=axes[0, 1])
axes[0, 1].set_title('二次多项式回归')

# 稳健回归（减少离群值影响）
sns.regplot(data=tips, x='total_bill', y='tip', robust=True, ax=axes[1, 0])
axes[1, 0].set_title('稳健回归')

# LOESS 局部回归
sns.regplot(data=tips, x='total_bill', y='tip', lowess=True, ax=axes[1, 1],
            line_kws={'color': 'red'})
axes[1, 1].set_title('LOESS 局部回归')

plt.tight_layout()
plt.show()

# === 残差图 ===
sns.residplot(data=tips, x='total_bill', y='tip', lowess=True,
              scatter_kws={'alpha': 0.5})
plt.title('残差图')
plt.show()

# === lmplot 分组回归 ===
sns.lmplot(data=tips, x='total_bill', y='tip',
           col='time',   # 按时间分面
           hue='smoker', # 颜色区分吸烟
           height=5, aspect=1,
           scatter_kws={'alpha': 0.5})
plt.show()

# === 自定义估计函数 ===
# barplot 默认显示均值+置信区间，可以自定义
sns.barplot(data=tips, x='day', y='total_bill',
            estimator=np.median, errorbar=('pi', 50))
plt.title('中位数 + 50% 区间')
plt.show()
```

| 回归参数 | 说明 | 适用场景 |
|----------|------|---------|
| order | 多项式阶数 | 非线性趋势 |
| robust | 稳健回归 | 有离群值 |
| lowess | 局部加权回归 | 复杂非线性 |
| logistic | 逻辑回归 | 二元响应变量 |
| ci | 置信区间大小 | None=不显示 |

### 8. 与 Matplotlib 配合

Seaborn 生成的都是 Matplotlib 对象，可以用 Matplotlib API 进一步定制。

```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# === Seaborn 绘图 + Matplotlib 精调 ===
fig, ax = plt.subplots(figsize=(10, 6))

# Seaborn 绑定到指定 ax
sns.boxplot(data=tips, x='day', y='total_bill', hue='time', ax=ax)

# Matplotlib 精调
ax.set_title('每日账单分布', fontsize=16, fontweight='bold')
ax.set_xlabel('星期', fontsize=12)
ax.set_ylabel('账单金额 ($)', fontsize=12)
ax.legend(title='时段', loc='upper left')

# 添加标注
ax.annotate('中位数差异最大', xy=(3, 20), xytext=(1, 40),
            arrowprops=dict(arrowstyle='->', color='red'),
            fontsize=10, color='red')

# 调整 Y 轴范围
ax.set_ylim(0, 55)

# 添加水平参考线
ax.axhline(y=tips['total_bill'].median(), color='gray',
           linestyle='--', alpha=0.5, label='总体中位数')
ax.legend()

plt.tight_layout()
plt.show()

# === 获取 Seaborn 返回对象进行深度定制 ===
g = sns.FacetGrid(tips, col='time', height=5)

def custom_plot(data, **kwargs):
    ax = plt.gca()
    sns.scatterplot(data=data, x='total_bill', y='tip', ax=ax, **kwargs)
    # 添加每个分面的统计信息
    median_tip = data['tip'].median()
    ax.axhline(y=median_tip, color='red', linestyle='--', alpha=0.5)
    ax.text(0.05, 0.95, f'中位数: ${median_tip:.2f}',
            transform=ax.transAxes, fontsize=10,
            verticalalignment='top')

g.map_dataframe(custom_plot, hue='smoker')
g.add_legend()
plt.show()

# === 导出高质量图片 ===
g = sns.pairplot(sns.load_dataset('penguins'), hue='species')
g.savefig('penguins_pairplot.png', dpi=300, bbox_inches='tight')
```

## 面试题

### 1. Seaborn 和 Matplotlib 的关系是什么？

Seaborn 是 Matplotlib 的高级封装，它在 Matplotlib 的 Artist 层之上提供统计图表接口。Seaborn 的所有绘图函数最终都创建 Matplotlib 对象（Figure、Axes、Line2D 等），返回的也是 Matplotlib 对象。这意味着：（1）可以用 Matplotlib API 进一步精调 Seaborn 图表；（2）Seaborn 依赖 Matplotlib，不能脱离它独立使用；（3）Seaborn 的 `set_theme()` 会修改 Matplotlib 的全局 rcParams。反过来，Seaborn 提供了 Matplotlib 不具备的统计功能：自动置信区间、核密度估计、分面绘图、聚类热力图等。

### 2. Seaborn 的 figure-level 和 axes-level 函数有什么区别？

axes-level 函数（如 `scatterplot`、`boxplot`、`histplot`）接受 `ax` 参数，绘制在指定的 Axes 上，返回 Matplotlib 对象，可以在一个 Figure 上组合多个图表。figure-level 函数（如 `relplot`、`catplot`、`displot`、`lmplot`、`pairplot`、`jointplot`）创建自己的 Figure 和 Grid 对象，通过 `col`/`row` 参数自动分面，返回 FacetGrid/PairGrid/JointGrid 对象。figure-level 函数的优势是分面自动化，劣势是不能与已有 Axes 组合、不能精细控制子图大小。需要复杂布局时，优先用 axes-level 函数 + 手动 subplots。

### 3. 如何选择 boxplot、violinplot 和 swarmplot？

boxplot 展示四分位数和离群值，适合快速比较组间差异，但隐藏了分布形状；violinplot 在箱线图基础上叠加了 KDE 密度，能看出分布是否对称、是否多峰，适合样本量较大的情况（小样本的 KDE 不稳定）；swarmplot 展示每个数据点的实际位置，避免重叠，适合小样本（<200），大样本会太密或报错。最佳实践：小样本用 swarmplot，中等样本用 boxplot+swarmplot 叠加，大样本用 violinplot。

### 4. Seaborn 的分面绘图（FacetGrid）如何工作？

FacetGrid 按 `col` 和 `row` 变量的唯一值创建子图网格。每个子图显示对应变量组合的数据子集。创建后通过 `map()` 或 `map_dataframe()` 指定绘图函数和数据列。`map()` 传入函数名和位置参数，`map_dataframe()` 传入函数名和关键字参数（更灵活，支持 hue）。`col_wrap` 参数让列方向自动换行，适合类别数很多的情况。figure-level 函数（relplot、catplot、displot）内部就是创建 FacetGrid 然后 map。

### 5. heatmap 的 annot 和 fmt 参数怎么用？

`annot=True` 在每个格子上显示数值文本，`fmt` 控制格式化方式。`fmt='.2f'` 显示两位小数，`fmt='d'` 显示整数，`fmt='.1%'` 显示百分比。annot 还可以接受与数据同形状的字符串数组，用于自定义显示内容（如同时显示数值和星号表示显著性）。配合 `mask` 参数可以只显示部分格子（如仅显示上三角相关矩阵），配合 `linewidths` 和 `linecolor` 控制格子间距。

### 6. pairplot 和 PairGrid 有什么区别？

`pairplot` 是 `PairGrid` 的便捷封装，一行代码完成配对图，但自定义空间有限。`PairGrid` 可以分别控制对角线和非对角线的图表类型：`map_diag()` 绘制对角线（默认直方图，可改为 KDE），`map_upper()` 绘制上三角（散点），`map_lower()` 绘制下三角（KDE），还可以 `map_offdiag()` 同时设置上下三角。PairGrid 还支持 `hue` 分组，适合多类别数据的全面探索。需要精细控制时用 PairGrid，快速查看时用 pairplot。

### 7. Seaborn 的 errorbar 参数支持哪些选项？

Seaborn 0.13+ 的 `errorbar` 参数取代了旧版 `ci` 参数，支持：（1）`'ci'` — 均值的置信区间（默认95%，即 errorbar='ci'），基于 bootstrap；（2）`('pi', 50)` — 百分位区间，50%即四分位距；（3）`'sd'` — 标准差；（4）`'se'` — 标准误；（5）自定义函数 — 接收数据返回 (low, high) 元组。`errorbar=None` 不显示误差条。bootstrapped CI 计算较慢，大数据集建议用 `'sd'` 或 `('pi', 50)`。

### 8. 如何在 Seaborn 图表中显示中文？

两种方式：（1）全局配置 Matplotlib 字体——`plt.rcParams['font.sans-serif'] = ['SimHei']` 和 `plt.rcParams['axes.unicode_minus'] = False`，Seaborn 会继承这些设置；（2）在调用 Seaborn 函数时，轴标签和标题用 Matplotlib 的 `set_title()`/`set_xlabel()` 单独设置中文。`sns.set_theme()` 会重置 rcParams，所以字体配置要在 `set_theme()` 之后执行。另外 Seaborn 图例的标题和标签来自 DataFrame 列名，需要将列名改为中文或用 `legend.set_title()` 修改。

## 外部参考

- [Seaborn 官方文档](https://seaborn.pydata.org/)

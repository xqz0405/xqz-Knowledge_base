---
tags:
  - Python
  - 数据处理
  - Pandas
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# Pandas基础

## What — 是什么

> Pandas 是 Python 中最流行的数据分析库，提供 DataFrame 和 Series 两种核心数据结构，支持数据清洗、转换、聚合和可视化。

**核心概念：**

- **Series**：一维带标签数组（索引 + 值）
- **DataFrame**：二维表格数据（行索引 + 列索引）
- **Index**：行/列标签，支持多级索引（MultiIndex）
- **GroupBy**：分组聚合，Split-Apply-Combine 模式

**核心架构：**

- 设计理念：类 SQL 的表操作，内存计算
- 核心模块：Index、DataFrame、GroupBy、Resample、IO 工具
- 数据流：数据读取 → 清洗/转换 → 聚合/分析 → 输出/可视化

**插件生态：**

- 官方生态：matplotlib（可视化）、statsmodels（统计建模）
- 社区热门：geopandas（地理数据）、pandas-profiling（自动分析报告）

## Why — 为什么

**适用场景：**

- 数据清洗与预处理
- 探索性数据分析（EDA）
- 报表生成与数据聚合
- 特征工程（机器学习前置步骤）

**对比同类框架：**

| 维度 | Pandas | Polars | SQL |
|------|--------|--------|-----|
| 性能 | 中（单线程） | 极高（多线程+惰性） | 取决于数据库 |
| 生态 | 最丰富 | 增长中 | 成熟 |
| 学习曲线 | 低 | 中 | 中 |
| 灵活性 | 高 | 高 | 中（声明式） |

**优缺点：**

- ✅ 优点：
  - API 丰富，几乎覆盖所有数据处理场景
  - 社区庞大，问题搜索容易
  - 与 NumPy/Matplotlib/Scikit-learn 无缝集成
- ❌ 缺点：
  - 单线程，大数据集性能差
  - 内存占用高（约为原始数据的 2-10 倍）
  - 链式操作易产生 SettingWithCopyWarning

## How — 怎么用

### 快速上手

```python
import pandas as pd

# 创建 DataFrame
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35],
    "city": ["Beijing", "Shanghai", "Shenzhen"],
})

# 基本操作
df.describe()             # 统计摘要
df[df["age"] > 28]        # 筛选
df.sort_values("age")     # 排序
df.groupby("city")["age"].mean()  # 分组聚合
```

### 代码示例

**数据清洗：**

```python
# 处理缺失值
df.isnull().sum()              # 统计缺失
df.dropna(subset=["age"])      # 删除缺失行
df["age"].fillna(df["age"].mean())  # 填充缺失

# 类型转换
df["date"] = pd.to_datetime(df["date_str"])
df["price"] = df["price_str"].str.replace("$", "").astype(float)

# 去重
df.drop_duplicates(subset=["email"], keep="first")
```

**聚合与透视：**

```python
# 分组聚合
df.groupby("department").agg(
    avg_salary=("salary", "mean"),
    headcount=("name", "count"),
    max_age=("age", "max"),
)

# 透视表
df.pivot_table(
    values="salary",
    index="department",
    columns="year",
    aggfunc="mean",
)
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| SettingWithCopyWarning | 链式索引赋值，不确定操作原表还是副本 | 用 `.loc[row, col] = value`，或 `.copy()` 显式复制 |
| 内存溢出 | DataFrame 过大 | 使用 `chunksize` 分块读取，或用 `dtype` 降低精度 |
| 合并慢 | `merge` 在大数据集上性能差 | 先 `set_index` 再用 `join`，或用 Polars |
| 时间解析错误 | 日期格式不统一 | 用 `format` 参数指定 `pd.to_datetime(date, format="%Y-%m-%d")` |

### 最佳实践

- 读取时指定 `dtype` 减少内存（`int32` 替代 `int64`）
- 用 `.loc[]` / `.iloc[]` 显式索引，避免链式索引
- 大文件用 `chunksize` 或 `dask` 分块处理
- 用 `df.info(memory_usage="deep")` 检查内存

## 面试题

**Q1: DataFrame 和 Series 的关系是什么？**
> Series 是一维带标签数组（索引 + 值），DataFrame 是二维表格（行索引 + 列索引）。DataFrame 的每一列都是一个 Series，两者共享索引体系。可以理解为 DataFrame 是 Series 的字典容器，键为列名，值为 Series。

**Q2: 向量化操作的优势是什么？为什么不推荐用 for 循环遍历 DataFrame？**
> 向量化操作底层调用 NumPy C 实现，避免 Python 解释器循环开销，性能可提升 10-100 倍。for 循环逐行迭代每次都要进行 Python 对象的装箱/拆箱和类型检查，而向量化操作在 C 层批量处理连续内存数据，效率极高。

**Q3: groupby 的工作原理是什么？**
> GroupBy 遵循 Split-Apply-Combine 模式：先将数据按分组键拆分（Split），对每个分组独立应用聚合函数（Apply，如 mean/sum/count），最后将结果合并为一个 DataFrame（Combine）。惰性求值，调用聚合函数时才真正执行计算。

**Q4: 如何处理缺失值？dropna 和 fillna 各适用什么场景？**
> `dropna` 直接删除含缺失值的行/列，适用于缺失比例低且删除不影响分析的场景；`fillna` 用指定值填充缺失，适用于需要保留全部数据的场景，常用填充策略包括均值/中位数填充、前向/后向填充（`ffill`/`bfill`）、插值法（`interpolate`）。

**Q5: 什么是 SettingWithCopyWarning？如何避免？**
> 链式索引赋值（如 `df[df["age"] > 30]["salary"] = 100`）时，Pandas 无法确定操作的是原表还是副本，可能静默失败。避免方法：使用 `.loc[row, col] = value` 单次定位赋值，或先 `.copy()` 显式创建副本再修改。

---

**相关链接：**
- [[异步编程]]
- Pandas 官方文档：https://pandas.pydata.org/docs/

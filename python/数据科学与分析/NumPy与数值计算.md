---
tags:
  - Python
  - 数据科学
  - NumPy
  - 数值计算
---

# NumPy与数值计算

## What — NumPy 是什么

NumPy（Numerical Python）是 Python 生态中最重要的数值计算基础库，提供了高性能的多维数组对象 `ndarray`、广播机制、向量化运算以及丰富的线性代数、傅里叶变换和随机数功能。几乎所有 Python 数据科学生态（Pandas、SciPy、scikit-learn、TensorFlow、PyTorch）都建立在 NumPy 之上。

核心特性：

| 特性 | 说明 |
|------|------|
| ndarray | 固定类型的多维数组，内存连续，支持高效索引 |
| 广播机制 | 不同形状数组间的自动扩展运算，避免显式循环 |
| 向量化运算 | 底层 C/Fortran 实现，比纯 Python 循环快 10-100 倍 |
| 线性代数 | 矩阵乘法、分解、特征值等 BLAS/LAPACK 封装 |
| 内存映射 | 处理超大数组时无需全部加载到内存 |

```python
import numpy as np

# 创建数组的基本方式
a = np.array([1, 2, 3, 4, 5])           # 一维数组
b = np.array([[1, 2], [3, 4]])          # 二维数组
c = np.zeros((3, 4))                     # 3×4 零矩阵
d = np.ones((2, 3, 2))                   # 2×3×2 全1数组
e = np.arange(0, 10, 2)                  # [0, 2, 4, 6, 8]
f = np.linspace(0, 1, 5)                 # [0., 0.25, 0.5, 0.75, 1.]

print(a.dtype, a.shape, a.ndim)          # int64 (5,) 1
print(b.dtype, b.shape, b.ndim)          # int64 (2, 2) 2
```

## Why — 为什么需要 NumPy

### Python 原生列表的痛点

Python 列表是动态类型的通用容器，每个元素都是独立的 Python 对象（含引用计数、类型指针等开销）。对于数值计算场景：

| 对比项 | Python 列表 | NumPy ndarray |
|--------|-------------|---------------|
| 内存布局 | 分散，每个元素独立对象 | 连续内存块，固定类型 |
| 存储开销 | 每个整数约 28 字节 | 每个整数 8 字节（int64） |
| 运算方式 | 逐元素 Python 循环 | 向量化 C 循环 |
| 速度 | 慢 10-100 倍 | 接近 C 语言速度 |
| 广播 | 不支持 | 自动形状扩展 |
| 切片 | 返回新列表（拷贝） | 返回视图（零拷贝） |

```python
import numpy as np
import time

size = 10_000_000

# Python 列表逐元素运算
py_a = list(range(size))
py_b = list(range(size))

start = time.perf_counter()
py_c = [a + b for a, b in zip(py_a, py_b)]
py_time = time.perf_counter() - start

# NumPy 向量化运算
np_a = np.arange(size)
np_b = np.arange(size)

start = time.perf_counter()
np_c = np_a + np_b
np_time = time.perf_counter() - start

print(f"Python 列表: {py_time:.3f}s")
print(f"NumPy 数组:  {np_time:.4f}s")
print(f"加速比: {py_time / np_time:.0f}x")
# 典型输出：Python 0.8s, NumPy 0.01s, 加速 80x
```

### NumPy 在数据科学生态中的地位

NumPy 是整个 Python 数据科学生态的基石。[[Pandas基础]] 的 Series 和 DataFrame 内部就是 ndarray，[[SciPy与科学计算]] 的算法输入输出都是 ndarray，scikit-learn 和深度学习框架同样依赖 NumPy 的数组协议。掌握 NumPy 是理解整个生态的必要前提。

## How — 如何使用 NumPy

### 1. ndarray 核心：创建与属性

ndarray 是 NumPy 的核心数据结构，所有元素必须同类型，存储在一块连续内存中。

```python
import numpy as np

# === 创建方式 ===

# 从 Python 数据创建
a = np.array([1, 2, 3])                        # 一维
b = np.array([[1, 2], [3, 4]])                 # 二维
c = np.array([1, 2, 3], dtype=np.float32)       # 指定类型

# 内置创建函数
np.zeros((3, 4))            # 全0
np.ones((2, 3))             # 全1
np.empty((2, 2))            # 未初始化（内容随机，但速度最快）
np.full((2, 3), 7)          # 全填充指定值
np.eye(3)                   # 单位矩阵
np.diag([1, 2, 3])          # 对角矩阵

# 序列创建
np.arange(0, 10, 2)         # 类似 range，步长2
np.linspace(0, 1, 5)        # 等间距5个点
np.logspace(0, 3, 4)        # 对数间距: [1, 10, 100, 1000]

# 随机创建
rng = np.random.default_rng(42)  # 推荐：新式随机数生成器
rng.random((2, 3))               # [0, 1) 均匀分布
rng.integers(0, 10, size=(3,))   # [0, 10) 整数
rng.standard_normal((3, 4))      # 标准正态
rng.choice([1, 2, 3], size=5)    # 随机选择

# === 核心属性 ===
arr = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.int32)

print(arr.ndim)       # 2 — 维度数
print(arr.shape)      # (2, 3) — 各维度大小
print(arr.size)       # 6 — 元素总数
print(arr.dtype)      # int32 — 数据类型
print(arr.itemsize)   # 4 — 每个元素字节数
print(arr.nbytes)     # 24 — 总字节数 (6 × 4)
print(arr.strides)    # (12, 4) — 跨步字节数
```

| 数据类型 | 说明 | 字节数 |
|----------|------|--------|
| np.bool_ | 布尔 | 1 |
| np.int8 / np.int16 / np.int32 / np.int64 | 有符号整数 | 1/2/4/8 |
| np.uint8 / np.uint16 / np.uint32 / np.uint64 | 无符号整数 | 1/2/4/8 |
| np.float16 / np.float32 / np.float64 | 浮点数 | 2/4/8 |
| np.complex64 / np.complex128 | 复数 | 8/16 |
| np.str_ | Unicode 字符串 | 变长 |
| np.object_ | Python 对象引用 | 8（指针） |

### 2. 索引与切片

NumPy 的索引和切片比 Python 列表强大得多，支持多维索引、布尔索引、花式索引，且基本切片返回视图而非拷贝。

```python
import numpy as np

arr = np.arange(24).reshape(2, 3, 4)  # shape: (2, 3, 4)

# === 基础索引 ===
print(arr[0])              # 第0个 3×4 子数组
print(arr[0, 1])           # 第0块第1行
print(arr[0, 1, 2])        # 标量: 6

# === 切片（返回视图） ===
print(arr[0, :, :])        # 第0块，所有行所有列
print(arr[:, 1, :])        # 所有块，第1行
print(arr[0, 0:2, 1:3])    # 第0块，0-1行，1-2列
print(arr[..., -1])        # 所有维度的最后一列（省略号语法）

# === 布尔索引 ===
data = np.array([5, 12, 3, 18, 7, 25])
mask = data > 10
print(data[mask])           # [12 18 25]
print(data[data > 10])      # 等价写法

# 多条件布尔索引
arr2d = np.arange(12).reshape(3, 4)
print(arr2d[(arr2d > 5) & (arr2d < 10)])  # 注意：& 非 and

# === 花式索引（返回拷贝） ===
arr2d = np.arange(12).reshape(3, 4)
print(arr2d[[0, 2]])               # 第0行和第2行
print(arr2d[:, [1, 3]])            # 第1列和第3列
print(arr2d[[0, 1, 2], [1, 2, 3]]) # 对角取值: [1, 6, 11]

# === 视图 vs 拷贝 ===
a = np.arange(6)
b = a[2:5]       # 视图（共享内存）
b[:] = 0         # 修改 b 也会修改 a
print(a)          # [0, 1, 0, 0, 0, 5]

c = a[2:5].copy()  # 显式拷贝，独立内存
c[:] = 99          # 不影响 a
```

| 索引类型 | 返回 | 是否共享内存 | 适用场景 |
|----------|------|-------------|----------|
| 基础切片 | 视图 | 是 | 高效读取/修改子数组 |
| 布尔索引 | 拷贝 | 否 | 条件筛选 |
| 花式索引 | 拷贝 | 否 | 任意位置选取 |
| 单整数索引 | 标量/降维视图 | 视图 | 降维访问 |

### 3. 广播机制

广播是 NumPy 最强大也最容易出错的特性。当两个数组形状不同时，NumPy 会自动扩展较小数组的形状，使运算可以逐元素进行。

**广播规则：**
1. 比较两个数组的形状，从末尾维度开始
2. 两个维度相等，或其中一个为 1，则兼容
3. 缺失的维度视为 1
4. 维度为 1 的数组沿该维度扩展

```python
import numpy as np

# === 标量与数组 ===
a = np.array([1, 2, 3])
print(a + 10)  # [11 12 13] — 标量广播到 (3,)

# === 一维与二维 ===
a = np.ones((3, 4))       # shape (3, 4)
b = np.array([1, 2, 3, 4]) # shape (4,)
print((a + b).shape)       # (3, 4) — b 广播到 (3, 4)

# === 列向量与行向量 ===
row = np.array([[1, 2, 3]])    # shape (1, 3)
col = np.array([[10], [20]])   # shape (2, 1)
print((row + col).shape)        # (2, 3)
print(row + col)
# [[11 12 13]
#  [21 22 23]]

# === 实战：标准化矩阵 ===
data = np.random.default_rng(42).standard_normal((100, 5))
mean = data.mean(axis=0)          # shape (5,)
std = data.std(axis=0)            # shape (5,)
normalized = (data - mean) / std  # 广播: (100,5) - (5,) -> (100,5)

# === 常见广播错误 ===
a = np.ones((3, 4))
b = np.ones((3,))
# a + b  # ValueError! 形状 (3,4) 和 (3,) 不兼容
# 修复：添加新轴
print((a + b[:, np.newaxis]).shape)  # (3, 4)
```

| 形状A | 形状B | 结果形状 | 说明 |
|--------|--------|----------|------|
| (3, 4) | (4,) | (3, 4) | 一维自动前置1 |
| (3, 1) | (1, 4) | (3, 4) | 双向扩展 |
| (5, 3, 4) | (3, 4) | (5, 3, 4) | 高维自动前置1 |
| (5, 3, 4) | (4,) | (5, 3, 4) | 混合扩展 |
| (3, 4) | (3,) | ValueError | 4≠3 且均非1 |

### 4. 形状操作与轴

理解轴（axis）是掌握 NumPy 的关键。axis=0 沿行方向（跨行），axis=1 沿列方向（跨列），axis=-1 沿最后一个维度。

```python
import numpy as np

arr = np.arange(12).reshape(3, 4)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# === reshape ===
print(arr.reshape(4, 3))      # 不改变数据，重新组织形状
print(arr.reshape(2, -1))     # -1 自动推算: (2, 6)
print(arr.ravel())             # 展平为1维（视图优先）
print(arr.flatten())           # 展平为1维（始终拷贝）

# === 转置 ===
print(arr.T)                   # 转置 (3,4) -> (4,3)
print(arr.transpose(1, 0))     # 等价写法
print(np.swapaxes(arr, 0, 1))  # 交换指定轴

# 3维转置
arr3d = np.arange(24).reshape(2, 3, 4)
print(arr3d.transpose(2, 0, 1).shape)  # (4, 2, 3)

# === 轴方向聚合 ===
print(arr.sum(axis=0))    # 列求和: [12 15 18 21]
print(arr.mean(axis=1))   # 行均值: [1.5, 5.5, 9.5]
print(arr.max(axis=0))    # 列最大值: [8 9 10 11]
print(arr.argmax(axis=1)) # 行内最大值索引: [3, 3, 3]
print(arr.cumsum(axis=0)) # 列方向累加

# === 堆叠与拆分 ===
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.stack([a, b])          # 新增维度: shape (2, 3)
np.vstack([a, b])         # 垂直堆叠: shape (2, 3)
np.hstack([a, b])         # 水平拼接: shape (6,)
np.concatenate([a, b])    # 沿现有轴拼接: shape (6,)

# 拆分
arr = np.arange(12).reshape(3, 4)
np.hsplit(arr, 2)   # 水平2等分
np.vsplit(arr, 3)   # 垂直3等分
np.split(arr, 3, axis=0)  # 指定轴等分
```

| 操作 | 方法 | 是否拷贝 | 说明 |
|------|------|---------|------|
| reshape | arr.reshape() | 优先视图 | 数据不变，形状重组 |
| ravel | arr.ravel() | 优先视图 | 展平，可能返回视图 |
| flatten | arr.flatten() | 始终拷贝 | 展平，保证新数组 |
| T / transpose | arr.T | 视图 | 转置，共享内存 |
| stack | np.stack() | 拷贝 | 沿新轴堆叠 |
| concatenate | np.concatenate() | 拷贝 | 沿已有轴拼接 |

### 5. 向量化运算与通用函数（ufunc）

NumPy 的核心优势在于向量化运算——用数组级操作替代 Python 循环。ufunc（通用函数）是对 ndarray 逐元素操作的函数，底层由 C 实现。

```python
import numpy as np

# === 算术运算 ===
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

print(a + b)        # [11 22 33 44]
print(a * b)        # [10 40 90 160]
print(a ** 2)       # [1 4 9 16]
print(b / a)        # [10. 10. 10. 10.]
print(b // a)       # [10 10 10 10]
print(b % a)        # [0 0 0 0]

# === 数学函数（ufunc） ===
x = np.array([0, np.pi/6, np.pi/4, np.pi/3, np.pi/2])
print(np.sin(x))    # 正弦
print(np.cos(x))    # 余弦
print(np.exp(x))    # 指数
print(np.log(np.array([1, np.e, np.e**2])))  # 自然对数
print(np.sqrt(np.array([1, 4, 9])))          # 平方根

# === 聚合函数 ===
arr = np.array([[3, 1, 4], [1, 5, 9]])
print(arr.sum())            # 23 — 全部求和
print(arr.sum(axis=0))      # [4, 6, 13] — 列求和
print(arr.mean())           # 3.833...
print(arr.std())            # 标准差
print(arr.var())            # 方差
print(arr.min(), arr.max()) # 1, 9
print(arr.cumsum())         # 累加和
print(arr.cumprod())        # 累乘积

# === 比较与逻辑 ===
a = np.array([1, 2, 3, 4, 5])
print(a > 3)                         # [False False False True True]
print(np.any(a > 3))                  # True
print(np.all(a > 3))                  # False
print(np.where(a > 3, a, 0))         # [0 0 0 4 5] — 条件替换

# === 集合运算 ===
a = np.array([1, 2, 3, 3, 4])
b = np.array([3, 4, 5, 6])
print(np.unique(a))          # [1 2 3 4]
print(np.intersect1d(a, b))  # [3 4] — 交集
print(np.union1d(a, b))      # [1 2 3 4 5 6] — 并集
print(np.setdiff1d(a, b))    # [1 2] — 差集

# === 排序 ===
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])
print(np.sort(arr))               # [1 1 2 3 4 5 6 9]
print(np.argsort(arr))            # [1 3 6 0 2 4 7 5] — 排序索引

arr2d = np.array([[3, 1, 4], [1, 5, 9]])
print(np.sort(arr2d, axis=0))     # 列方向排序
print(np.sort(arr2d, axis=1))     # 行方向排序
```

### 6. 线性代数

NumPy 的 `numpy.linalg` 模块封装了 BLAS/LAPACK 的高效线性代数实现。

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# === 矩阵运算 ===
print(A @ B)               # 矩阵乘法（推荐写法）
print(np.matmul(A, B))     # 等价
print(A.dot(B))            # 等价（旧写法）
print(A * B)               # 逐元素乘法（不是矩阵乘法！）

# === 矩阵分解 ===
# 特征值分解
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"特征值: {eigenvalues}")   # [-0.37, 5.37]

# 奇异值分解（SVD）
U, S, Vt = np.linalg.svd(A)
print(f"奇异值: {S}")             # [5.46, 0.37]

# QR 分解
Q, R = np.linalg.qr(A)

# Cholesky 分解（要求正定矩阵）
C = np.array([[4, 2], [2, 3]])
L = np.linalg.cholesky(C)
print(L @ L.T)              # 还原原矩阵

# === 求解线性方程组 ===
# Ax = b → x = A^(-1) @ b（不推荐，用 solve 更稳定）
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])
x = np.linalg.solve(A, b)   # 推荐！数值更稳定
print(x)                     # [2. 3.]
print(A @ x)                 # [9. 8.] — 验证

# === 其他常用 ===
print(np.linalg.det(A))     # 行列式
print(np.linalg.inv(A))     # 逆矩阵
print(np.linalg.norm(A))    # 范数（默认 Frobenius）
print(np.linalg.cond(A))    # 条件数
print(np.trace(A))          # 迹（对角线之和）
print(np.linalg.matrix_rank(A))  # 秩
```

| 函数 | 说明 | 典型用途 |
|------|------|---------|
| np.dot / @ | 矩阵乘法 | 神经网络前向传播 |
| np.linalg.eig | 特征值分解 | PCA 降维 |
| np.linalg.svd | 奇异值分解 | 推荐系统、图像压缩 |
| np.linalg.solve | 解线性方程组 | 回归分析 |
| np.linalg.inv | 矩阵求逆 | 理论推导（数值不推荐） |
| np.linalg.norm | 范数计算 | 正则化、距离度量 |
| np.linalg.det | 行列式 | 矩阵可逆性判断 |

### 7. 结构化数组与内存映射

NumPy 支持结构化数组（类似数据库表）和内存映射（处理超大文件），这是很多人忽略的高级特性。

```python
import numpy as np

# === 结构化数组 ===
# 定义字段：姓名(Unicode)、年龄(int)、身高(float)
dtype = [('name', 'U10'), ('age', 'i4'), ('height', 'f4')]
people = np.array([
    ('Alice', 25, 165.5),
    ('Bob', 30, 178.0),
    ('Charlie', 28, 172.3)
], dtype=dtype)

print(people['name'])        # ['Alice' 'Bob' 'Charlie']
print(people[people['age'] > 26])  # 按字段条件筛选

# 排序
sorted_people = np.sort(people, order='age')
print(sorted_people)

# === 内存映射 ===
# 处理超大数组，无需全部加载到内存
filename = 'large_array.npy'
big = np.arange(100_000_000, dtype=np.float64)  # ~800MB
big.tofile(filename)

# 内存映射方式读取
mmap = np.memmap(filename, dtype=np.float64, mode='r', shape=(100_000_000,))
print(mmap[:5])              # 只读取需要的部分
print(mmap[50_000_000])      # 随机访问

# 可读写模式
mmap_rw = np.memmap(filename, dtype=np.float64, mode='r+',
                     shape=(100_000_000,))
mmap_rw[0] = 999.0          # 修改会直接写回文件
mmap_rw.flush()              # 确保写入磁盘

import os
os.remove(filename)  # 清理
```

### 8. 性能优化技巧

```python
import numpy as np

# === 避免不必要的拷贝 ===
a = np.arange(1000000)

# 差：创建中间数组
# b = a * 2 + 1  # 两次内存分配

# 好：原地操作
b = a.copy()
b *= 2
b += 1

# 好：out 参数
result = np.empty_like(a)
np.multiply(a, 2, out=result)
np.add(result, 1, out=result)

# === 预分配数组 ===
# 差：不断 append
# result = []
# for i in range(1000):
#     result.append(i ** 2)
# result = np.array(result)

# 好：预分配
result = np.empty(1000)
for i in range(1000):
    result[i] = i ** 2

# 最好：向量化
result = np.arange(1000) ** 2

# === 选择合适的数据类型 ===
# 默认 float64 占 8 字节，很多场景 float32 就够
a64 = np.ones(1000000, dtype=np.float64)  # 8MB
a32 = np.ones(1000000, dtype=np.float32)  # 4MB，减半内存

# 图像数据用 uint8
img = np.random.randint(0, 256, size=(480, 640, 3), dtype=np.uint8)

# === 避免 Python 循环 ===
# 差：逐元素操作
# for i in range(len(arr)):
#     arr[i] = arr[i] * 2

# 好：向量化
arr = arr * 2

# === 使用 np.einsum 替代复杂运算 ===
A = np.random.randn(100, 50)
B = np.random.randn(50, 80)

# 矩阵乘法：等价于 A @ B
C = np.einsum('ij,jk->ik', A, B)

# 批量矩阵乘法
batch_a = np.random.randn(10, 3, 4)
batch_b = np.random.randn(10, 4, 5)
C = np.einsum('bij,bjk->bik', batch_a, batch_b)

# 向量外积之和
vectors = np.random.randn(5, 3)
outer_sum = np.einsum('ij,ik->jk', vectors, vectors)
```

| 优化手段 | 效果 | 适用场景 |
|----------|------|---------|
| 向量化替代循环 | 10-100× 提速 | 所有逐元素运算 |
| 预分配数组 | 避免 O(n) 拷贝 | 已知结果大小的循环 |
| out 参数 | 减少中间数组 | 连续运算链 |
| 降低 dtype 精度 | 内存减半 | 图像/嵌入式/推理 |
| np.einsum | 灵活高效 | 复杂张量运算 |
| 内存映射 | 内存可控 | 超大文件处理 |

## 面试题

### 1. NumPy ndarray 与 Python 列表有什么区别？

ndarray 是固定类型、内存连续的多维数组；Python 列表是动态类型、元素分散的通用容器。ndarray 存储同类型数据在连续内存块中，支持向量化运算和广播，性能比列表快 10-100 倍。列表每个元素是独立 Python 对象（含引用计数和类型指针），一个整数在列表中约占 28 字节，在 int64 数组中仅 8 字节。ndarray 切片返回视图（零拷贝），列表切片返回新列表。

### 2. 什么是广播机制？请举例说明。

广播是 NumPy 对不同形状数组进行逐元素运算时的自动扩展规则。从末尾维度开始逐维比较：若两维度相等或其中一个为 1，则兼容；维度为 1 的数组沿该维度复制扩展。例如 `(3,4) + (4,)` 中一维数组自动扩展为 `(3,4)`；`(3,1) + (1,4)` 双向扩展为 `(3,4)`。不兼容的形状如 `(3,4)` 和 `(3,)` 会报 ValueError，需用 `[:, np.newaxis]` 手动调整维度。

### 3. NumPy 切片中的视图和拷贝有什么区别？

基础切片返回视图——与原数组共享内存，修改视图会同步修改原数组，零拷贝开销。花式索引和布尔索引返回拷贝——独立内存，修改不影响原数组。`reshape`、`ravel`、`transpose` 优先返回视图，`flatten` 和 `copy()` 始终返回拷贝。判断是否为视图可用 `np.shares_memory(a, b)`。需要注意：对视图的 in-place 修改会影响原数组，这是很多 Bug 的来源。

### 4. np.matmul / @ 和 np.dot 有什么区别？

`@` 是 `np.matmul` 的运算符语法，遵循矩阵乘法语义：不支持标量操作数，对高维数组按最后两个维度做矩阵乘法、其他维度做广播。`np.dot` 更宽泛：标量做内积，一维做向量点积，二维做矩阵乘法，高维做最后轴与倒数第二轴的内积。推荐用 `@` 表示矩阵乘法，语义更清晰。注意 `*` 是逐元素乘法，不是矩阵乘法。

### 5. 如何选择合适的数据类型？

优先用最小够用的类型：图像像素用 `uint8`（0-255），分类标签用 `int32`，一般浮点用 `float64`（默认），深度学习推理用 `float32` 或 `float16` 节省内存和加速，布尔掩码用 `bool_`。避免 `object_` 类型（退化为 Python 对象，失去 NumPy 性能优势）。可以用 `arr.astype(np.float32)` 转换类型，注意浮点转整数会截断。

### 6. 什么是 np.einsum？它有什么优势？

`np.einsum` 是爱因斯坦求和约定的 NumPy 实现，用简洁的字符串表示复杂的张量运算。例如 `'ij,jk->ik'` 表示矩阵乘法，`'ij,ik->jk'` 表示向量外积求和，`'i,i->'` 表示向量点积。优势：一个函数替代 `matmul`、`tensordot`、`inner`、`outer` 等多种操作；避免中间数组创建；表达力强，特别适合批量运算和高维张量操作。

### 7. 如何处理超大数组无法全部装入内存的情况？

三种方案：（1）**内存映射** `np.memmap`：将磁盘文件映射为 NumPy 数组，按需加载页面，支持随机读写，适合固定大小的大数组；（2）**分块处理**：将大数组拆分为多个小块，逐块读入处理后再写出，配合 `np.load` 的 `mmap_mode` 参数；（3）**降低精度**：`float64` → `float32` 内存减半，`float16` 减至 1/4。如果数据本身有稀疏性，可用 SciPy 的稀疏矩阵格式。

### 8. NumPy 的 axis 参数怎么理解？

axis 指定操作沿哪个维度进行。对二维数组 `(m, n)`：`axis=0` 沿行方向（跨行），即对每列操作，结果消去第 0 维，shape 变为 `(n,)`；`axis=1` 沿列方向（跨列），即对每行操作，结果消去第 1 维，shape 变为 `(m,)`；不指定 axis 则对所有元素操作，结果为标量。记忆口诀：axis=k 表示"第 k 个下标会变化"，相当于沿着该维度遍历并聚合。`axis=-1` 表示最后一个维度。

## 外部参考

- [NumPy 官方文档](https://numpy.org/doc/stable/)

---
tags:
  - Python
  - 数据科学
  - SciPy
  - 科学计算
---

# SciPy与科学计算

## What — SciPy 是什么

SciPy（Scientific Python）是构建在 [[NumPy与数值计算]] 之上的科学计算库，提供了数学、科学和工程领域常用算法的高效实现。如果说 NumPy 提供了数组基础设施，SciPy 则在此基础上提供了完整的科学计算工具箱——优化、插值、积分、信号处理、线性代数、稀疏矩阵、统计分布等。

SciPy 的子模块结构：

| 子模块 | 说明 | 典型用途 |
|--------|------|---------|
| scipy.optimize | 优化与求根 | 参数拟合、最小化 |
| scipy.interpolate | 插值 | 数据平滑、缺失值填充 |
| scipy.integrate | 积分与ODE | 面积计算、微分方程求解 |
| scipy.signal | 信号处理 | 滤波、频谱分析 |
| scipy.linalg | 线性代数 | 矩阵分解、方程求解 |
| scipy.sparse | 稀疏矩阵 | 大规模稀疏系统 |
| scipy.stats | 统计分布与检验 | 假设检验、分布拟合 |
| scipy.fft | 傅里叶变换 | 频域分析 |
| scipy.ndimage | N维图像处理 | 滤波、形态学、测量 |
| scipy.spatial | 空间数据结构 | KNN、Voronoi、凸包 |
| scipy.special | 特殊函数 | 贝塞尔、伽马、椭圆 |
| scipy.io | 数据I/O | MATLAB/IDL 文件读写 |

```python
import numpy as np
from scipy import optimize, integrate, interpolate, signal

# SciPy 的典型使用模式：传入 NumPy 数组，返回 NumPy 数组
# 比纯 Python 实现快 10-1000 倍（底层 Fortran/C）
```

## Why — 为什么需要 SciPy

### NumPy 只提供了基础运算

NumPy 提供了数组、线性代数和基础数学函数，但缺少科学计算的核心算法：非线性优化、数值积分、信号滤波、插值、ODE求解等。这些算法实现复杂、数值稳定性要求高，不适合手动编写。SciPy 封装了经过数十年验证的 Fortran/C 库（如 MINPACK、QUADPACK、FFTPACK、ARPACK），保证正确性和性能。

### SciPy 在数据科学中的角色

| 任务 | 工具 | 说明 |
|------|------|------|
| 数据拟合 | scipy.optimize.curve_fit | 非线性回归 |
| 缺失值插补 | scipy.interpolate | 时间序列补全 |
| 信号去噪 | scipy.signal | 传感器数据预处理 |
| 微分方程 | scipy.integrate.solve_ivp | 动力学建模 |
| 统计检验 | scipy.stats | [[统计分析与假设检验]]的基础 |
| 稀疏数据 | scipy.sparse | 推荐系统、NLP 特征矩阵 |

[[Pandas基础]] 和 [[NumPy与数值计算]] 负责数据操作，SciPy 负责科学计算算法，scikit-learn 负责机器学习——三者构成 Python 数据科学的核心栈。

## How — 如何使用 SciPy

### 1. 优化（scipy.optimize）

优化是 SciPy 使用最频繁的模块，涵盖函数最小化、求根、曲线拟合等。

```python
import numpy as np
from scipy import optimize
import matplotlib.pyplot as plt

# === 函数最小化 ===

# 定义目标函数
def rosenbrock(x):
    """Rosenbrock 函数，经典优化测试函数"""
    return (1 - x[0])**2 + 100 * (x[1] - x[0]**2)**2

# Nelder-Mead 单纯形法（无需梯度）
result = optimize.minimize(rosenbrock, x0=[0, 0], method='Nelder-Mead')
print(f"最小值点: {result.x}, 最小值: {result.fun:.6f}")
print(f"收敛: {result.success}, 迭代: {result.nit}")

# BFGS（需要梯度，自动数值微分）
result = optimize.minimize(rosenbrock, x0=[0, 0], method='BFGS')
print(f"BFGS 最小值点: {result.x}")

# 带约束优化
def objective(x):
    return x[0]**2 + x[1]**2

constraints = [
    {'type': 'ineq', 'fun': lambda x: x[0] + x[1] - 1},  # x0+x1 >= 1
    {'type': 'eq', 'fun': lambda x: x[0] - x[1]},          # x0 == x1
]
bounds = [(0, None), (0, None)]  # x0, x1 >= 0

result = optimize.minimize(objective, x0=[0.5, 0.5],
                           method='SLSQP', bounds=bounds,
                           constraints=constraints)
print(f"约束优化: {result.x}")

# === 曲线拟合 ===
x_data = np.linspace(0, 4, 50)
y_true = 2.5 * np.sin(1.5 * x_data) + 0.5
y_data = y_true + np.random.default_rng(42).normal(0, 0.3, 50)

def model(x, a, b, c):
    return a * np.sin(b * x) + c

# 最小二乘拟合
popt, pcov = optimize.curve_fit(model, x_data, y_data, p0=[1, 1, 0])
perr = np.sqrt(np.diag(pcov))  # 参数标准差

print(f"拟合参数: a={popt[0]:.2f}±{perr[0]:.2f}, "
      f"b={popt[1]:.2f}±{perr[1]:.2f}, c={popt[2]:.2f}±{perr[2]:.2f}")

# 可视化
plt.figure(figsize=(8, 5))
plt.scatter(x_data, y_data, s=10, label='带噪数据')
plt.plot(x_data, model(x_data, *popt), 'r-', label='拟合曲线')
plt.plot(x_data, y_true, 'g--', label='真实曲线')
plt.legend()
plt.title('curve_fit 非线性拟合')
plt.show()

# === 求根 ===
# 标量方程
root = optimize.root_scalar(lambda x: x**3 - x - 2, bracket=[1, 2], method='brentq')
print(f"根: {root.root:.6f}")  # ~1.5214

# 方程组
def equations(vars):
    x, y = vars
    return [x**2 + y**2 - 4, x - y]

result = optimize.root(equations, x0=[1, 1])
print(f"方程组解: {result.x}")
```

| 方法 | 说明 | 需要梯度 | 适用场景 |
|------|------|---------|---------|
| Nelder-Mead | 单纯形法 | 否 | 无梯度信息、粗搜索 |
| BFGS | 拟牛顿法 | 自动数值微分 | 通用无约束优化 |
| L-BFGS-B | 带边界BFGS | 自动 | 有边界约束 |
| SLSQP | 序列二次规划 | 自动 | 等式+不等式约束 |
| COBYLA | 约束优化 | 否 | 约束问题（无梯度） |
| curve_fit | 最小二乘 | 自动 | 数据拟合 |
| least_squares | 最小二乘 | 可选 | 带边界/鲁棒拟合 |

### 2. 插值（scipy.interpolate）

插值用于在已知数据点之间估计未知值，常见于时间序列补全、数据平滑和函数近似。

```python
import numpy as np
from scipy import interpolate
import matplotlib.pyplot as plt

# 稀疏已知数据点
x_known = np.array([0, 1, 2, 3, 4, 5])
y_known = np.array([0, 0.8, 0.9, 0.1, -0.8, -1.0])

# 密集插值点
x_dense = np.linspace(0, 5, 200)

# === 线性插值 ===
f_linear = interpolate.interp1d(x_known, y_known, kind='linear')

# === 三次样条插值 ===
f_cubic = interpolate.interp1d(x_known, y_known, kind='cubic')

# === B样条插值（更灵活） ===
tck = interpolate.splrep(x_known, y_known, s=0)  # s=0: 精确通过数据点
y_bspline = interpolate.splev(x_dense, tck)

# 可视化对比
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x_known, y_known, 'ko', markersize=8, label='原始数据')
ax.plot(x_dense, f_linear(x_dense), '--', label='线性插值')
ax.plot(x_dense, f_cubic(x_dense), '-', label='三次样条')
ax.plot(x_dense, y_bspline, ':', linewidth=2, label='B样条')
ax.legend()
ax.set_title('插值方法对比')
plt.show()

# === 外推 vs 不外推 ===
# 默认超出范围会报错
# f_cubic(5.5)  # ValueError

# 设置外推
f_extrap = interpolate.interp1d(x_known, y_known, kind='cubic',
                                 fill_value='extrapolate')
print(f"外推值: {f_extrap(5.5):.4f}")

# === 二维插值 ===
from scipy.interpolate import griddata

# 散点数据
rng = np.random.default_rng(42)
n = 100
x = rng.random(n) * 4 - 2
y = rng.random(n) * 4 - 2
z = np.sin(np.sqrt(x**2 + y**2))

# 网格化
grid_x, grid_y = np.mgrid[-2:2:100j, -2:2:100j]

# 三种插值方法
z_nearest = griddata((x, y), z, (grid_x, grid_y), method='nearest')
z_linear = griddata((x, y), z, (grid_x, grid_y), method='linear')
z_cubic = griddata((x, y), z, (grid_x, grid_y), method='cubic')

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for ax, data, title in zip(axes,
                            [z_nearest, z_linear, z_cubic],
                            ['nearest', 'linear', 'cubic']):
    im = ax.imshow(data, extent=(-2, 2, -2, 2), origin='lower', cmap='viridis')
    ax.set_title(title)
    ax.scatter(x, y, c='red', s=3)
    plt.colorbar(im, ax=ax)

plt.tight_layout()
plt.show()

# === RBF 插值（径向基函数） ===
from scipy.interpolate import RBFInterpolator

rbf = RBFInterpolator(np.column_stack([x, y]), z, kernel='thin_plate_spline')
grid_points = np.column_stack([grid_x.ravel(), grid_y.ravel()])
z_rbf = rbf(grid_points).reshape(grid_x.shape)
```

| 插值方法 | 说明 | 平滑度 | 适用场景 |
|----------|------|--------|---------|
| linear | 分段线性 | C0 | 快速、保守 |
| cubic | 三次样条 | C2 | 平滑曲线 |
| nearest | 最近邻 | 不连续 | 分类数据 |
| griddata | 散点→网格 | 可选 | 不规则采样 |
| RBF | 径向基函数 | C∞ | 复杂曲面 |

### 3. 积分与常微分方程（scipy.integrate）

```python
import numpy as np
from scipy import integrate

# === 定积分 ===
# 数值积分 ∫₀^π sin(x) dx = 2
result, error = integrate.quad(np.sin, 0, np.pi)
print(f"∫sin(x)dx = {result:.6f} (误差: {error:.2e})")

# 带参数的积分
def integrand(x, a, b):
    return a * np.exp(-b * x)

result, _ = integrate.quad(integrand, 0, np.inf, args=(2, 1))
print(f"∫2e^(-x)dx from 0 to ∞ = {result:.6f}")  # = 2

# 二重积分
result, _ = integrate.dblquad(
    lambda y, x: x * y,       # 被积函数 f(y, x) 注意参数顺序
    0, 1,                      # x 的范围
    lambda x: 0,               # y 的下界（可以是 x 的函数）
    lambda x: 1 - x            # y 的上界
)
print(f"二重积分 = {result:.6f}")  # = 1/24

# === 常微分方程求解（ODE） ===
# 例：阻尼振荡 y'' + 2y' + 5y = 0
# 转换为一阶方程组：y1' = y2, y2' = -5*y1 - 2*y2

def damped_oscillator(t, y):
    return [y[1], -5 * y[0] - 2 * y[1]]

# 初值：y(0) = 1, y'(0) = 0
sol = integrate.solve_ivp(
    damped_oscillator,
    t_span=[0, 10],
    y0=[1, 0],
    method='RK45',       # 默认，自适应步长
    dense_output=True,   # 返回连续解
    max_step=0.1
)

# 在密集时间点上求值
t_eval = np.linspace(0, 10, 500)
y_eval = sol.sol(t_eval)

import matplotlib.pyplot as plt
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].plot(t_eval, y_eval[0], label='y(t)')
axes[0].plot(t_eval, y_eval[1], label="y'(t)")
axes[0].set_title('阻尼振荡')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# 相空间图
axes[1].plot(y_eval[0], y_eval[1])
axes[1].set_xlabel('y')
axes[1].set_ylabel("y'")
axes[1].set_title('相空间')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

# === Lotka-Volterra 捕食者-猎物模型 ===
def lotka_volterra(t, z, a, b, c, d):
    x, y = z
    return [a*x - b*x*y, -c*y + d*x*y]

sol = integrate.solve_ivp(
    lotka_volterra,
    t_span=[0, 30],
    y0=[10, 5],
    args=(1.5, 1, 3, 1),  # a=1.5, b=1, c=3, d=1
    dense_output=True
)

t = np.linspace(0, 30, 1000)
z = sol.sol(t)

plt.figure(figsize=(10, 5))
plt.plot(t, z[0], label='猎物')
plt.plot(t, z[1], label='捕食者')
plt.legend()
plt.title('Lotka-Volterra 模型')
plt.show()
```

| 函数 | 说明 | 适用场景 |
|------|------|---------|
| quad | 一维定积分 | 面积、期望 |
| dblquad | 二重积分 | 二维区域 |
| tplquad | 三重积分 | 三维区域 |
| solve_ivp | ODE 初值问题 | 动力学系统 |
| solve_bvp | ODE 边值问题 | 边界条件问题 |
| odeint | 旧版ODE求解（兼容） | 旧代码 |

### 4. 信号处理（scipy.signal）

```python
import numpy as np
from scipy import signal
import matplotlib.pyplot as plt

# === 生成测试信号 ===
rng = np.random.default_rng(42)
fs = 1000  # 采样频率 1000Hz
t = np.arange(0, 1, 1/fs)

# 50Hz + 120Hz + 噪声
x_clean = np.sin(2 * np.pi * 50 * t) + 0.5 * np.sin(2 * np.pi * 120 * t)
x_noisy = x_clean + rng.normal(0, 1.0, len(t))

# === FIR 滤波器设计 ===
# 低通滤波：保留 80Hz 以下，去除 120Hz
nyq = fs / 2
cutoff = 80 / nyq  # 归一化截止频率

# 设计 FIR 滤波器
taps = signal.firwin(101, cutoff, window='hamming')
x_filtered = signal.lfilter(taps, 1.0, x_noisy)

fig, axes = plt.subplots(3, 1, figsize=(12, 8))

axes[0].plot(t[:200], x_noisy[:200])
axes[0].set_title('含噪信号')

axes[1].plot(t[:200], x_filtered[:200])
axes[1].set_title('FIR 滤波后')

axes[2].plot(t[:200], x_clean[:200])
axes[2].set_title('原始信号')

plt.tight_layout()
plt.show()

# === 频谱分析（FFT） ===
from scipy.fft import fft, fftfreq

yf = fft(x_noisy)
xf = fftfreq(len(t), 1/fs)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(xf[:len(xf)//2], np.abs(yf[:len(yf)//2]) / len(t))
axes[0].set_xlim(0, 200)
axes[0].set_title('频谱')
axes[0].set_xlabel('频率 (Hz)')

# Welch 功率谱密度（更平滑）
f, Pxx = signal.welch(x_noisy, fs, nperseg=256)
axes[1].semilogy(f, Pxx)
axes[1].set_xlim(0, 200)
axes[1].set_title('Welch 功率谱密度')
axes[1].set_xlabel('频率 (Hz)')

plt.tight_layout()
plt.show()

# === 峰值检测 ===
x_peak = np.sin(2 * np.pi * 5 * t) + 0.5 * rng.random(len(t))
peaks, properties = signal.find_peaks(x_peak, height=0, distance=fs//10)

plt.figure(figsize=(12, 4))
plt.plot(t, x_peak)
plt.plot(t[peaks], x_peak[peaks], 'rx', markersize=8)
plt.title(f'峰值检测（找到 {len(peaks)} 个峰）')
plt.show()

# === 卷积与相关 ===
# 互相关：测量两个信号的时移关系
sig1 = np.sin(2 * np.pi * 10 * t)
sig2 = np.sin(2 * np.pi * 10 * (t - 0.05))  # 延迟 50ms

correlation = signal.correlate(sig1, sig2, mode='full')
lag = signal.correlation_lags(len(sig1), len(sig2))
delay = lag[np.argmax(correlation)] / fs
print(f"检测到延迟: {delay*1000:.1f}ms")  # ~50ms
```

| 函数 | 说明 | 适用场景 |
|------|------|---------|
| firwin | FIR 滤波器设计 | 低通/带通/高通 |
| butter | Butterworth 滤波器 | 频域滤波 |
| lfilter | IIR/FIR 滤波 | 时域滤波 |
| filtfilt | 零相位滤波 | 无相位延迟 |
| welch | 功率谱密度 | 频谱分析 |
| find_peaks | 峰值检测 | 心电/声纳 |
| correlate | 互相关 | 时延估计 |
| resample | 重采样 | 采样率转换 |

### 5. 线性代数（scipy.linalg）

scipy.linalg 比 numpy.linalg 更全面，额外提供了矩阵分解、特殊线性系统求解等功能。

```python
import numpy as np
from scipy import linalg

A = np.array([[4, 2, 1], [2, 5, 3], [1, 3, 6]], dtype=float)
b = np.array([7, 10, 10], dtype=float)

# === 解线性方程组 ===
x = linalg.solve(A, b)
print(f"解: {x}")
print(f"验证: {A @ x}")  # 应接近 b

# === LU 分解 ===
P, L, U = linalg.lu(A)
print(f"P:\n{P}\nL:\n{L}\nU:\n{U}")
# 验证: P @ L @ U ≈ A

# === Cholesky 分解（正定矩阵） ===
C = linalg.cholesky(A, lower=True)  # 下三角
print(f"Cholesky: A = C @ C.T")

# === QR 分解 ===
Q, R = linalg.qr(A)
print(f"Q 正交性验证: ||Q.T @ Q - I|| = {linalg.norm(Q.T @ Q - np.eye(3)):.2e}")

# === Schur 分解 ===
T, Z = linalg.schur(A)
print(f"Schur 对角块:\n{T}")

# === SVD ===
U, s, Vt = linalg.svd(A)
print(f"奇异值: {s}")

# 低秩近似（取前 k 个奇异值）
k = 2
A_approx = U[:, :k] @ np.diag(s[:k]) @ Vt[:k, :]
print(f"低秩近似误差: {linalg.norm(A - A_approx):.4f}")

# === 特征值问题 ===
eigenvalues, eigenvectors = linalg.eig(A)
print(f"特征值: {eigenvalues}")

# 广义特征值问题: A @ x = lambda * B @ x
B = np.eye(3) * 2
eigvals, eigvecs = linalg.eig(A, B)
print(f"广义特征值: {eigvals}")

# === 矩阵范数与条件数 ===
print(f"Frobenius范数: {linalg.norm(A)}")
print(f"2-范数: {linalg.norm(A, 2)}")
print(f"条件数: {linalg.cond(A)}")

# === 矩阵指数 ===
# e^A 在微分方程中有重要应用: dy/dt = Ay → y(t) = e^(At) y(0)
exp_A = linalg.expm(A)
print(f"矩阵指数 e^A:\n{exp_A}")

# === 带状/三角/对称系统的高效求解 ===
# 三对角系统
diag_main = np.array([2, 3, 4, 5], dtype=float)
diag_off = np.array([1, 1, 1], dtype=float)
rhs = np.array([1, 2, 3, 4], dtype=float)
x_tri = linalg.solve_banded((1, 1), np.array([diag_off, diag_main, diag_off]), rhs)
```

| 对比项 | numpy.linalg | scipy.linalg |
|--------|-------------|-------------|
| 基础解方程 | solve | solve |
| LU 分解 | 无 | lu |
| Cholesky | 无 | cholesky |
| Schur 分解 | 无 | schur |
| 矩阵指数 | 无 | expm / expm_frechet |
| 带状系统 | 无 | solve_banded |
| 三角系统 | 无 | solve_triangular |
| 广义特征值 | eig(A) | eig(A, B) |
| 性能 | 略快（少一层封装） | 功能更全 |

### 6. 稀疏矩阵（scipy.sparse）

稀疏矩阵用于存储和计算大部分元素为零的矩阵，在 NLP、推荐系统和网络分析中极为常见。

```python
import numpy as np
from scipy import sparse

# === 创建稀疏矩阵 ===

# COO 格式（coordinate，适合构建）
row = np.array([0, 1, 2, 3, 4])
col = np.array([0, 2, 1, 3, 4])
data = np.array([1, 2, 3, 4, 5])
A_coo = sparse.coo_matrix((data, (row, col)), shape=(5, 5))
print(f"COO 非零元素: {A_coo.nnz}")

# CSR 格式（compressed sparse row，适合算术运算和行切片）
A_csr = A_coo.tocsr()

# CSC 格式（compressed sparse column，适合列切片）
A_csc = A_coo.tocsc()

# 从密集矩阵转换
dense = np.eye(5) * 3
A_from_dense = sparse.csr_matrix(dense)

# 特殊稀疏矩阵
A_diag = sparse.diags([1, 2, 3, 4], offsets=0, shape=(4, 4))  # 对角
A_eye = sparse.eye(1000)  # 稀疏单位矩阵
A_random = sparse.random(100, 100, density=0.05, format='csr')  # 随机稀疏

# === 内存对比 ===
n = 10000
dense_mem = n * n * 8 / 1024**2  # 762.9 MB
density = 0.001
sparse_mem = n * n * density * (8 + 8 + 8) / 1024**2  # ~2.3 MB
print(f"密集: {dense_mem:.1f}MB, 稀疏: {sparse_mem:.1f}MB")

# === 运算 ===
# 矩阵乘法
x = np.ones(5)
print(f"A @ x = {A_csr @ x}")

# 稀疏 × 稀疏
B = sparse.random(5, 5, density=0.4, format='csr')
C = A_csr @ B  # 返回稀疏矩阵

# 元素运算
D = A_csr.multiply(B)  # 逐元素乘
E = A_csr + B          # 加法

# 求解稀疏线性系统
from scipy.sparse.linalg import spsolve
A = sparse.random(100, 100, density=0.1, format='csr')
A = A + sparse.eye(100) * 10  # 确保对角占优
b = np.ones(100)
x = spsolve(A, b)
print(f"稀疏求解残差: {np.linalg.norm(A @ x - b):.2e}")

# === 稀疏矩阵的特征值 ===
from scipy.sparse.linalg import eigs
# 只计算前 k 个特征值（不需要全部分解）
eigenvalues, eigenvectors = eigs(A.astype(float), k=5)
print(f"前5个特征值: {eigenvalues}")
```

| 格式 | 全称 | 适合操作 | 不适合 |
|------|------|---------|--------|
| COO | Coordinate | 构建、转换 | 算术运算 |
| CSR | Compressed Sparse Row | 行切片、乘法、运算 | 列切片 |
| CSC | Compressed Sparse Column | 列切片、乘法 | 行切片 |
| DIA | Diagonal | 对角矩阵 | 非对角稀疏 |
| LIL | List of Lists | 逐步修改 | 大规模运算 |
| BSR | Block Sparse Row | 块状稀疏 | 随机稀疏 |

### 7. 空间数据结构（scipy.spatial）

```python
import numpy as np
from scipy import spatial

rng = np.random.default_rng(42)
points = rng.random((100, 2))

# === KDTree：最近邻搜索 ===
tree = spatial.KDTree(points)

# 查询最近邻
dist, idx = tree.query([0.5, 0.5])
print(f"最近邻距离: {dist:.4f}, 索引: {idx}")

# 查询 k 个最近邻
dists, idxs = tree.query([0.5, 0.5], k=5)
print(f"5个最近邻: {idxs}")

# 范围查询
neighbors = tree.query_ball_point([0.5, 0.5], r=0.1)
print(f"半径0.1内的点数: {len(neighbors)}")

# === 凸包 ===
hull = spatial.ConvexHull(points)
print(f"凸包顶点数: {len(hull.vertices)}")
print(f"凸包面积: {hull.volume:.4f}")  # 2D 中 volume = area

# === Voronoi 图 ===
vor = spatial.Voronoid(points)

# === Delaunay 三角化 ===
tri = spatial.Delaunay(points)
print(f"三角面数: {len(tri.simplices)}")

# 判断点是否在三角化内部
test_point = np.array([[0.5, 0.5]])
inside = tri.find_simplex(test_point) >= 0
print(f"点(0.5,0.5)在内部: {inside[0]}")

# === 距离计算 ===
from scipy.spatial.distance import pdist, squareform, cdist

# 成对距离
dist_matrix = pdist(points[:10], metric='euclidean')
dist_square = squareform(dist_matrix)  # 转方阵

# 两组点之间的距离
A = rng.random((5, 3))
B = rng.random((3, 3))
dists = cdist(A, B, metric='cosine')  # 余弦距离

# 可用距离度量
print(spatial.distance.pdist.__doc__[:200])  # 查看支持的度量
```

| 函数 | 说明 | 时间复杂度 |
|------|------|-----------|
| KDTree | k-d树 | 构建O(nlogn)，查询O(logn) |
| ConvexHull | 凸包 | O(nlogn) |
| Voronoi | Voronoi图 | O(nlogn) |
| Delaunay | Delaunay三角化 | O(nlogn) |
| pdist | 成对距离 | O(n²) |
| cdist | 组间距离 | O(nm) |

## 面试题

### 1. SciPy 和 NumPy 有什么关系？为什么要分成两个库？

NumPy 提供数组数据结构和基础数学运算（线性代数、随机数），SciPy 在 NumPy 之上提供科学计算算法（优化、积分、信号处理等）。分成两个库的原因：（1）关注点分离——NumPy 是基础设施（数组、广播、ufunc），几乎所有 Python 科学库都依赖它，保持轻量和稳定；（2）发布周期——NumPy 更新慢但极度稳定，SciPy 可以更快迭代算法；（3）历史原因——NumPy 最初从 SciPy 中独立出来成为基础库。使用时 `import numpy as np` + `from scipy import optimize` 等组合是标准做法。

### 2. scipy.optimize.minimize 的 method 参数怎么选择？

选方法看约束和梯度信息：无约束+无梯度→Nelder-Mead（稳定但慢）；无约束+可用梯度→BFGS（通用首选）；有边界约束→L-BFGS-B；有等式/不等式约束→SLSQP 或 trust-constr；大规模稀疏问题→trust-constr。`curve_fit` 是最小二乘的专用封装，数据拟合场景优先使用。对于非光滑函数，Nelder-Mead 和 COBYLA 是仅有的无需梯度的方法。实践中建议先 BFGS 快速搜索，再用 Nelder-Mead 验证。

### 3. 插值和拟合有什么区别？

插值要求曲线精确通过所有数据点，适用于数据准确无噪声的场景（如函数表查值、时间序列补全）；拟合允许曲线不通过数据点，追求总体误差最小，适用于有噪声的实验数据。插值方法有线性、三次样条、B样条等，数据点多时高阶插值可能振荡（Runge现象）；拟合方法有最小二乘、鲁棒拟合等，可以自定义模型函数。有噪声时拟合优于插值，因为插值会拟合噪声。

### 4. solve_ivp 中 method 参数怎么选？

默认 RK45（4/5阶 Runge-Kutta）适合大多数非刚性问题；刚性问题（不同分量变化速度差异大）用 Radau 或 BDF；高精度非刚性问题用 DOP853（8阶）；简单问题用 RK23。判断刚性的经验：如果 RK45 步长极小或求解极慢，大概率是刚性问题。`dense_output=True` 返回连续解对象，可以在任意时间点求值，避免存储过多元数据。

### 5. FIR 滤波器和 IIR 滤波器怎么选择？

FIR（有限脉冲响应）滤波器：稳定（永远稳定）、可实现精确线性相位（无相位失真）、但需要较高阶数才能达到锐截止、计算量较大。IIR（无限脉冲响应）滤波器：低阶数即可实现锐截止、计算量小、但可能不稳定、相位非线性。信号处理中优先 FIR（相位保真），实时系统/资源受限场景考虑 IIR。SciPy 中 FIR 用 `firwin` + `lfilter`，IIR 用 `butter` + `sosfilt`（推荐二阶级联格式避免数值问题）。

### 6. 稀疏矩阵应该选什么格式？

按操作选格式：构建阶段用 COO 或 LIL（支持高效逐元素修改）；算术运算和矩阵乘法用 CSR；列切片用 CSC；对角矩阵用 DIA；块状稀疏用 BSR。最常见的工作流是：COO/LIL 构建 → 转为 CSR 进行运算。避免在 CSR/CSC 上逐元素修改（每次修改都要重组索引），应先构建完毕再转换。检查稀疏性：`matrix.nnz / (matrix.shape[0] * matrix.shape[1])`，密度低于 0.3 时稀疏格式通常更省内存。

### 7. scipy.linalg 和 numpy.linalg 有什么区别？

numpy.linalg 提供基础线性代数功能（solve、eig、svd、inv、norm 等），API 简洁，性能略快。scipy.linalg 在此基础上增加了更多功能：LU 分解（lu）、Cholesky 分解（cholesky）、Schur 分解（schur）、矩阵指数（expm）、带状系统求解（solve_banded）、三角系统求解（solve_triangular）、广义特征值问题、矩阵平方根（sqrtm）等。如果需要的功能 numpy.linalg 有，两者性能差异极小；需要高级功能时必须用 scipy.linalg。

### 8. KDTree 的查询效率如何？适合什么场景？

KDTree 构建时间 O(n log n)，单次最近邻查询平均 O(log n)，最坏 O(n)。适合低到中维（d < 20）的最近邻搜索、范围查询。高维时（d > 20）KDTree 退化为接近线性扫描（维度灾难），此时应考虑近似最近邻算法（如 LSH、HNSW）。典型应用场景：地理空间查询、点云配准、KNN 分类器、碰撞检测。`query_ball_point` 做范围查询比逐点 query 更高效。大数据集可考虑 cKDTree（C 实现，更快但已被合并到 KDTree 中）。

## 外部参考

- [SciPy 官方文档](https://docs.scipy.org/doc/scipy/)

---
tags:
  - Python
  - 深度学习
  - PyTorch
  - 张量
---

# PyTorch基础

## What — PyTorch 是什么

PyTorch 是 Meta（Facebook）开源的深度学习框架，以动态计算图和 Python 优先的设计著称。相比 TensorFlow 的静态图传统，PyTorch 的"define-by-run"模式让调试和实验更自然，已成为学术界和工业界的主流选择。

核心组件：

| 组件 | 说明 |
|------|------|
| torch.Tensor | 张量，类似 [[NumPy与数值计算]] 的 ndarray 但支持 GPU |
| torch.autograd | 自动微分引擎，自动计算梯度 |
| torch.nn | 神经网络模块（层、损失函数、容器） |
| torch.optim | 优化器（SGD、Adam、AdamW 等） |
| torch.utils.data | 数据加载（Dataset、DataLoader） |
| torch.cuda | GPU 加速 |

```python
import torch

# 基础张量操作
x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)
y = x ** 2 + 2 * x
loss = y.sum()
loss.backward()
print(f"梯度: {x.grad}")  # 2x + 2 → [4, 6, 8]
```

## Why — 为什么选择 PyTorch

### PyTorch vs TensorFlow

| 对比项 | PyTorch | TensorFlow |
|--------|---------|-----------|
| 计算图 | 动态（define-by-run） | 静态为主（2.x 兼容动态） |
| 调试 | 原生 Python 调试 | 较难调试 |
| API 风格 | Pythonic | 更工程化 |
| 学术界占比 | >80% | 下降中 |
| 工业部署 | TorchServe/ONNX | TF Serving/TF Lite |
| 生态 | Hugging Face、torchvision | Keras、TF Hub |
| 学习曲线 | 低 | 中 |

PyTorch 在研究和快速原型开发中占据绝对优势，[[大模型与LLM应用]] 生态（Hugging Face Transformers）几乎全部基于 PyTorch。

## How — 如何使用 PyTorch

### 1. 张量（Tensor）

张量是 PyTorch 的核心数据结构，与 NumPy ndarray 类似但支持 GPU 加速和自动微分。

```python
import torch
import numpy as np

# === 创建张量 ===
# 从数据
a = torch.tensor([1, 2, 3], dtype=torch.float32)
b = torch.tensor([[1, 2], [3, 4]], dtype=torch.float64)

# 从形状
c = torch.zeros(3, 4)
d = torch.ones(2, 3)
e = torch.empty(2, 2)           # 未初始化
f = torch.randn(3, 4)           # 标准正态
g = torch.randint(0, 10, (3,))  # 整数

# 从 NumPy
arr = np.array([1, 2, 3])
t_from_np = torch.from_numpy(arr)  # 共享内存
t_to_np = t_from_np.numpy()         # 共享内存

# === 张量属性 ===
x = torch.randn(3, 4, 5)
print(f"形状: {x.shape}")       # (3, 4, 5)
print(f"维度: {x.ndim}")        # 3
print(f"类型: {x.dtype}")       # float32
print(f"设备: {x.device}")      # cpu

# === 数据类型 ===
print(torch.tensor([1]).dtype)        # int64
print(torch.tensor([1.0]).dtype)      # float32
print(torch.tensor([True]).dtype)     # bool

# 类型转换
x = torch.tensor([1.5, 2.7])
print(x.float())     # float32
print(x.double())    # float64
print(x.long())      # int64 (截断)
print(x.to(torch.int))  # 通用转换

# === 运算 ===
a = torch.tensor([1.0, 2.0, 3.0])
b = torch.tensor([4.0, 5.0, 6.0])

# 逐元素
print(a + b)           # [5, 7, 9]
print(a * b)           # [4, 10, 18]
print(a ** 2)          # [1, 4, 9]
print(torch.sqrt(a))   # [1, 1.41, 1.73]

# 矩阵运算
A = torch.randn(3, 4)
B = torch.randn(4, 5)
print(A @ B)                   # 矩阵乘法
print(torch.matmul(A, B))      # 等价
print(A.T)                     # 转置

# 聚合
print(a.sum(), a.mean(), a.max(), a.argmin())

# === 广播 ===
a = torch.ones(3, 4)
b = torch.ones(4)
print((a + b).shape)  # (3, 4)

# === 索引与切片 ===
x = torch.arange(12).reshape(3, 4)
print(x[0])          # 第0行
print(x[:, 1])       # 第1列
print(x[0:2, 1:3])   # 子矩阵
print(x[x > 5])      # 布尔索引

# === 形状操作 ===
x = torch.arange(12)
print(x.reshape(3, 4))      # (3, 4)
print(x.view(3, 4))          # (3, 4)，共享内存
print(x.unsqueeze(0).shape)  # (1, 12)
print(x.unsqueeze(1).shape)  # (12, 1)

# view vs reshape：view 要求内存连续，reshape 不要求
x_transposed = torch.randn(3, 4).T
# x_transposed.view(12)  # RuntimeError!
print(x_transposed.reshape(12))  # OK，必要时拷贝
print(x_transposed.contiguous().view(12))  # OK，显式连续化

# === GPU ===
if torch.cuda.is_available():
    device = torch.device('cuda')
    x = x.to(device)
    print(x.device)  # cuda:0
else:
    device = torch.device('cpu')

# 多GPU
if torch.cuda.device_count() > 1:
    print(f"可用GPU: {torch.cuda.device_count()}")
```

### 2. 自动微分（autograd）

autograd 是 PyTorch 的核心——只需设置 `requires_grad=True`，所有运算自动跟踪，调用 `backward()` 即可计算梯度。

```python
import torch

# === 基础自动微分 ===
x = torch.tensor([2.0, 3.0], requires_grad=True)

# 前向计算
y = x ** 2           # y = [4, 9]
z = y.sum()          # z = 13

# 反向传播
z.backward()
print(f"dz/dx = {x.grad}")  # 2x = [4, 6]

# === 计算图 ===
# 每个张量都有 grad_fn 记录创建它的操作
a = torch.tensor(2.0, requires_grad=True)
b = torch.tensor(3.0, requires_grad=True)
c = a * b + a ** 2  # c = 6 + 4 = 10
print(f"c.grad_fn: {c.grad_fn}")  # <AddBackward0>

c.backward()
print(f"dc/da = {a.grad}")  # b + 2a = 3 + 4 = 7
print(f"dc/db = {b.grad}")  # a = 2

# === 梯度累积 ===
# 梯度默认累积，每次 backward 前需清零
x = torch.tensor([1.0, 2.0], requires_grad=True)

for _ in range(3):
    y = (x ** 2).sum()
    y.backward()
    print(f"梯度: {x.grad}")
    x.grad.zero_()  # 清零！否则梯度会累积

# === 禁用梯度 ===
# 推理时不需要梯度，节省内存
x = torch.randn(3, requires_grad=True)

with torch.no_grad():
    y = x * 2
    print(y.requires_grad)  # False

# 等价方式
y = (x * 2).detach()

# === 自定义梯度函数 ===
class MyReLU(torch.autograd.Function):
    @staticmethod
    def forward(ctx, x):
        ctx.save_for_backward(x)
        return x.clamp(min=0)

    @staticmethod
    def backward(ctx, grad_output):
        x, = ctx.saved_tensors
        grad_input = grad_output.clone()
        grad_input[x < 0] = 0
        return grad_input

x = torch.randn(5, requires_grad=True)
y = MyReLU.apply(x)
y.sum().backward()
```

### 3. 构建神经网络（torch.nn）

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# === 方式1：Sequential ===
model = nn.Sequential(
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Linear(128, 10)
)

# === 方式2：Module 子类（推荐） ===
class NeuralNet(nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim, dropout=0.3):
        super().__init__()
        self.fc1 = nn.Linear(input_dim, hidden_dim)
        self.bn1 = nn.BatchNorm1d(hidden_dim)
        self.fc2 = nn.Linear(hidden_dim, hidden_dim // 2)
        self.bn2 = nn.BatchNorm1d(hidden_dim // 2)
        self.fc3 = nn.Linear(hidden_dim // 2, output_dim)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x):
        x = F.relu(self.bn1(self.fc1(x)))
        x = self.dropout(x)
        x = F.relu(self.bn2(self.fc2(x)))
        x = self.dropout(x)
        x = self.fc3(x)
        return x

model = NeuralNet(784, 256, 10)
print(model)

# 查看参数
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"总参数: {total_params:,}, 可训练: {trainable_params:,}")

# === 常用层 ===
# 线性层
nn.Linear(in_features, out_features)

# 卷积层
nn.Conv2d(in_channels, out_channels, kernel_size, stride, padding)
nn.Conv1d(in_channels, out_channels, kernel_size)

# 池化层
nn.MaxPool2d(kernel_size, stride)
nn.AvgPool2d(kernel_size)
nn.AdaptiveAvgPool2d(output_size)  # 自适应全局池化

# 归一化
nn.BatchNorm1d(num_features)
nn.BatchNorm2d(num_features)
nn.LayerNorm(normalized_shape)
nn.Dropout(p=0.5)

# 激活函数
nn.ReLU()
nn.GELU()
nn.Sigmoid()
nn.Tanh()
nn.Softmax(dim=1)

# 循环层
nn.LSTM(input_size, hidden_size, num_layers, batch_first=True)
nn.GRU(input_size, hidden_size, num_layers, batch_first=True)

# Embedding
nn.Embedding(num_embeddings, embedding_dim)

# Transformer
nn.TransformerEncoderLayer(d_model, nhead)
nn.TransformerEncoder(encoder_layer, num_layers)
```

### 4. 训练循环

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# === 数据准备 ===
torch.manual_seed(42)
n = 5000
X = torch.randn(n, 20)
y = (X[:, 0] + X[:, 1] * 2 > 0).long()

dataset = TensorDataset(X, y)
train_size = int(0.8 * n)
train_ds, val_ds = torch.utils.data.random_split(dataset, [train_size, n - train_size])
train_loader = DataLoader(train_ds, batch_size=64, shuffle=True)
val_loader = DataLoader(val_ds, batch_size=128)

# === 模型 ===
class Classifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(20, 64),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(64, 32),
            nn.ReLU(),
            nn.Linear(32, 2)
        )

    def forward(self, x):
        return self.net(x)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = Classifier().to(device)

# === 损失函数与优化器 ===
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001, weight_decay=1e-4)

# 学习率调度器
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)

# === 训练循环 ===
num_epochs = 30
best_val_acc = 0

for epoch in range(num_epochs):
    # 训练阶段
    model.train()
    train_loss = 0
    train_correct = 0
    train_total = 0

    for X_batch, y_batch in train_loader:
        X_batch, y_batch = X_batch.to(device), y_batch.to(device)

        # 前向
        outputs = model(X_batch)
        loss = criterion(outputs, y_batch)

        # 反向
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        # 统计
        train_loss += loss.item() * X_batch.size(0)
        train_correct += (outputs.argmax(1) == y_batch).sum().item()
        train_total += X_batch.size(0)

    scheduler.step()

    # 验证阶段
    model.eval()
    val_loss = 0
    val_correct = 0
    val_total = 0

    with torch.no_grad():
        for X_batch, y_batch in val_loader:
            X_batch, y_batch = X_batch.to(device), y_batch.to(device)
            outputs = model(X_batch)
            loss = criterion(outputs, y_batch)

            val_loss += loss.item() * X_batch.size(0)
            val_correct += (outputs.argmax(1) == y_batch).sum().item()
            val_total += X_batch.size(0)

    train_acc = train_correct / train_total
    val_acc = val_correct / val_total

    print(f"Epoch {epoch+1:2d} | "
          f"Train Loss: {train_loss/train_total:.4f} Acc: {train_acc:.4f} | "
          f"Val Loss: {val_loss/val_total:.4f} Acc: {val_acc:.4f} | "
          f"LR: {scheduler.get_last_lr()[0]:.6f}")

    # 保存最佳模型
    if val_acc > best_val_acc:
        best_val_acc = val_acc
        torch.save(model.state_dict(), 'best_model.pth')
```

### 5. 自定义 Dataset 与 DataLoader

```python
import torch
from torch.utils.data import Dataset, DataLoader
import numpy as np

# === 自定义 Dataset ===
class CustomDataset(Dataset):
    def __init__(self, features, labels, transform=None):
        self.features = torch.tensor(features, dtype=torch.float32)
        self.labels = torch.tensor(labels, dtype=torch.long)
        self.transform = transform

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        x = self.features[idx]
        y = self.labels[idx]
        if self.transform:
            x = self.transform(x)
        return x, y

# 使用
rng = np.random.default_rng(42)
features = rng.standard_normal((1000, 20))
labels = rng.integers(0, 5, 1000)

dataset = CustomDataset(features, labels)
loader = DataLoader(dataset, batch_size=32, shuffle=True, num_workers=0, drop_last=False)

for X_batch, y_batch in loader:
    print(f"Batch: {X_batch.shape}, {y_batch.shape}")
    break

# === 图像 Dataset ===
from torchvision import datasets, transforms

transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# 数据增强
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# DataLoader 参数
# batch_size: 批大小
# shuffle: 是否打乱
# num_workers: 数据加载并行进程数
# pin_memory: GPU 训练时设 True 加速传输
# drop_last: 丢弃不完整的最后一批
```

### 6. 模型保存与加载

```python
import torch
import torch.nn as nn

model = nn.Sequential(nn.Linear(20, 10), nn.ReLU(), nn.Linear(10, 2))

# === 保存/加载 state_dict（推荐） ===
torch.save(model.state_dict(), 'model_weights.pth')
model.load_state_dict(torch.load('model_weights.pth', weights_only=True))

# === 保存/加载完整模型 ===
torch.save(model, 'full_model.pth')
loaded_model = torch.load('full_model.pth', weights_only=False)

# === 保存检查点（含训练状态） ===
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'scheduler_state_dict': scheduler.state_dict(),
    'best_val_acc': best_val_acc,
}
torch.save(checkpoint, 'checkpoint.pth')

# 恢复
checkpoint = torch.load('checkpoint.pth', weights_only=False)
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
scheduler.load_state_dict(checkpoint['scheduler_state_dict'])
start_epoch = checkpoint['epoch'] + 1
```

### 7. GPU 训练与多GPU

```python
import torch
import torch.nn as nn

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# === 单 GPU ===
model = model.to(device)
X, y = X.to(device), y.to(device)

# === DataParallel（简单多GPU） ===
if torch.cuda.device_count() > 1:
    model = nn.DataParallel(model)
model = model.to(device)

# 注意：DataParallel 的 state_dict 带有 'module.' 前缀
# 加载时需处理：
# state_dict = torch.load('model.pth')
# state_dict = {k.replace('module.', ''): v for k, v in state_dict.items()}
# model.load_state_dict(state_dict)

# === DistributedDataParallel（推荐，更高效） ===
# 需配合 torch.distributed.launch 使用
# torch.distributed.init_process_group(backend='nccl')
# model = nn.parallel.DistributedDataParallel(model, device_ids=[local_rank])

# === 混合精度训练 ===
scaler = torch.amp.GradScaler('cuda')

for X_batch, y_batch in train_loader:
    X_batch, y_batch = X_batch.to(device), y_batch.to(device)

    with torch.amp.autocast('cuda'):
        outputs = model(X_batch)
        loss = criterion(outputs, y_batch)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
    optimizer.zero_grad()
```

### 8. 常用优化器与调度器

```python
import torch.optim as optim

# === 优化器 ===
# SGD（基础）
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9, weight_decay=1e-4)

# Adam（最常用）
optimizer = optim.Adam(model.parameters(), lr=0.001, betas=(0.9, 0.999), weight_decay=1e-4)

# AdamW（修正权重衰减，推荐用于 Transformer）
optimizer = optim.AdamW(model.parameters(), lr=1e-4, weight_decay=0.01)

# === 学习率调度器 ===
model = nn.Linear(10, 2)
optimizer = optim.Adam(model.parameters(), lr=0.01)

# StepLR：每 step_size 个 epoch 乘以 gamma
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)

# CosineAnnealingLR：余弦退火
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=50, eta_min=1e-6)

# ReduceLROnPlateau：指标不降时减小学习率
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', factor=0.5, patience=5, min_lr=1e-7
)
# 使用: scheduler.step(val_loss)

# CosineAnnealingWarmRestarts：带热重启的余弦退火
scheduler = optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer, T_0=10, T_mult=2, eta_min=1e-6
)

# OneCycleLR：一个周期策略（快速收敛）
scheduler = optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=0.01, total_steps=num_epochs * len(train_loader)
)

# 可视化学习率
import matplotlib.pyplot as plt
lrs = []
for _ in range(100):
    lrs.append(optimizer.param_groups[0]['lr'])
    optimizer.step()
    scheduler.step()

plt.plot(lrs)
plt.title('学习率变化')
plt.xlabel('Step')
plt.ylabel('LR')
plt.show()
```

| 优化器 | 特点 | 推荐场景 |
|--------|------|---------|
| SGD+Momentum | 简单稳定、泛化好 | CV、需要精调 |
| Adam | 自适应学习率、收敛快 | 通用默认 |
| AdamW | 修正权重衰减 | Transformer、大模型 |
| AdaBelief | 自适应+稳定 | 替代 Adam |

## 面试题

### 1. PyTorch 的动态计算图有什么优势？

动态计算图（define-by-run）在每次前向传播时即时构建计算图，反向传播后销毁。优势：（1）**调试友好**——可以用 print、pdb 断点、if/for 等原生 Python 工具调试，就像写普通 Python 代码；（2）**灵活**——可以在前向中写条件逻辑、循环、动态形状，模型结构可以根据输入变化（如 RNN 的变长序列）；（3）**直观**——代码执行顺序就是计算图构建顺序，心智负担小。劣势：无法像静态图那样做全局优化（算子融合、内存复用），但 PyTorch 2.0 的 torch.compile 弥补了这一差距。

### 2. tensor.view 和 tensor.reshape 的区别？

view 返回共享内存的新视图，要求张量在内存中连续且新形状与元素总数一致，如果内存不连续会报错。reshape 在内存连续时返回视图（等价于 view），不连续时自动拷贝数据返回新张量。所以 reshape 更安全（不会报错），view 更严格（保证零拷贝，出了问题能及早发现）。实践中：知道内存连续时用 view（性能保证），不确定时用 reshape（不会崩溃）。需要确保零拷贝时用 `tensor.is_contiguous()` 检查或 `.contiguous().view()`。

### 3. 为什么训练时需要 model.train()，推理时需要 model.eval()？

这两个方法影响特定层的行为：Dropout 在 train 模式下随机丢弃（p=0.5），eval 模式下不丢弃；BatchNorm 在 train 模式下用当前批统计量并更新运行均值/方差，eval 模式下用训练时累积的运行统计量。如果推理时忘记 model.eval()，Dropout 会随机丢弃导致输出不稳定，BatchNorm 会用当前批（可能很小）的统计量导致预测偏差。但 model.eval() 不影响梯度计算——推理还需配合 `with torch.no_grad()` 禁用 autograd 以节省内存。

### 4. gradient accumulation 的原理和使用场景？

梯度累积是在内存不足时模拟大 batch_size 的技术：将一个大 batch 分成多个小 batch，每个小 batch 做前向+反向但不清零梯度，累积几次后再执行 optimizer.step()。效果等价于使用了 `batch_size * accumulation_steps` 的大 batch。使用场景：GPU 显存不够用大 batch，但小 batch 训练不稳定（尤其 BatchNorm 场景）。注意：accumulation_steps 内不要调 scheduler.step()；BatchNorm 统计量仍基于小 batch（可能影响效果，可用 GroupNorm 替代）。

### 5. 什么是混合精度训练？

混合精度训练同时使用 FP32（全精度）和 FP16（半精度），减少显存占用和加速计算。原理：（1）前向传播用 FP16 计算和存储，速度更快、显存更省；（2）权重主副本保持 FP32，更新时用 FP32 保证精度；（3）损失缩放（loss scaling）防止 FP16 梯度下溢——小梯度在 FP16 中可能变为 0，乘以缩放因子后保留。PyTorch 中用 `torch.amp.autocast` + `GradScaler` 实现，通常加速 1.5-3 倍，显存减少 30-50%。注意：某些操作（如 reduction）必须用 FP32，autocast 会自动处理。

### 6. DataLoader 的 num_workers 怎么设置？

num_workers 控制数据加载的并行进程数。0 表示主进程加载（最慢但最安全），>0 使用多进程预取数据。推荐设置：CPU 核心数的一半到全部，通常 4-8。设太高会因进程启动和 IPC 开销反而变慢。Windows 上 num_workers>0 常出错（需要 `if __name__ == '__main__'` 保护）。配合 `pin_memory=True`（GPU 训练时）和 `prefetch_factor=2` 可进一步提升 GPU 利用率。判断是否瓶颈：观察 GPU 利用率，若 <80% 且 CPU 满载，大概率是数据加载瓶颈，增大 num_workers。

### 7. 如何解决 GPU 显存不足？

按效果排序：（1）**减小 batch_size**——最直接，但太小影响 BatchNorm 和训练稳定性；（2）**混合精度训练**——`torch.amp`，显存减少 30-50%，几乎不影响精度；（3）**梯度累积**——模拟大 batch 但显存按小 batch 计算；（4）**梯度检查点**——`torch.utils.checkpoint`，用时间换空间，前向时不保存中间激活，反向时重算，显存减少 60% 但速度慢 30%；（5）**模型并行**——将模型不同层放在不同 GPU；（6）**torch.compile**——PyTorch 2.0 的编译优化可减少内存占用。

### 8. PyTorch 2.0 的 torch.compile 有什么改进？

torch.compile 是 PyTorch 2.0 的核心特性，通过 JIT 编译将动态图转为优化的静态执行计划。它分析 Python 代码，生成融合的 GPU kernel，减少 Python 开销和内存访问。使用：`model = torch.compile(model)`，一行即可。效果：训练速度提升 10-30%（部分场景 2x+），无需改代码。底层使用 TorchDynamo 捕获计算图、TorchInductor 生成优化代码。支持动态形状和大部分 Python 语法（不支持的自动 fallback 到 eager 模式）。也支持 `torch.compile` 装饰单个函数。

## 外部参考

- [PyTorch 官方文档](https://pytorch.org/docs/stable/)

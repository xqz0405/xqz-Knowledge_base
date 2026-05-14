---
tags:
  - Python
  - 深度学习
  - TensorFlow
  - Keras
---

# TensorFlow与Keras

## What — TensorFlow 与 Keras 是什么

TensorFlow 是 Google 开源的端到端机器学习平台，提供从数据加载到模型部署的全流程支持。Keras 是 TensorFlow 的高层 API，以极简代码实现深度学习模型的构建、训练和评估。TensorFlow 2.x 将 Keras 作为官方首选 API（`tf.keras`），兼顾了易用性和底层控制力。

核心架构：

| 层级 | 组件 | 说明 |
|------|------|------|
| 高层 API | tf.keras | 模型构建、训练、评估 |
| 中层 API | tf.data | 数据管道 |
| 低层 API | tf.raw_ops | 细粒度操作控制 |
| 部署 | TF Serving / TF Lite / TF.js | 服务端/移动端/浏览器 |
| 可视化 | TensorBoard | 训练监控 |
| 加速 | XLA / tf.function | 编译优化 |

```python
import tensorflow as tf

# Keras 极简示例
model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=10, validation_split=0.2)
```

## Why — 为什么使用 TensorFlow/Keras

### TensorFlow vs [[PyTorch基础]]

| 对比项 | TensorFlow/Keras | PyTorch |
|--------|-----------------|---------|
| 上手难度 | 低（Keras 封装好） | 低（Pythonic） |
| 模型构建 | Sequential/Functional/子类 | Module 子类 |
| 部署生态 | 最完善（Serving/Lite/JS） | TorchServe/ONNX |
| 工业应用 | 传统优势 | 快速追赶 |
| 移动端 | TF Lite 成熟 | PyTorch Mobile |
| 学术研究 | 占比下降 | 主流 |
| 大模型 | JAX（Google 新方向） | Hugging Face 生态 |

选 TensorFlow 的场景：需要完善的部署链路（TF Serving/Serving）、移动端推理（TF Lite）、生产稳定性要求高。选 PyTorch 的场景：研究实验、大模型/LLM、需要灵活的动态图。

## How — 如何使用 TensorFlow/Keras

### 1. 构建模型的三种方式

```python
import tensorflow as tf
from tensorflow import keras

# === 方式1：Sequential（最简单，线性堆叠） ===
model = keras.Sequential([
    keras.layers.Dense(256, activation='relu', input_shape=(784,)),
    keras.layers.BatchNormalization(),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dense(10, activation='softmax')
])

# === 方式2：Functional API（推荐，支持多输入/输出/残差） ===
inputs = keras.Input(shape=(784,))
x = keras.layers.Dense(256, activation='relu')(inputs)
x = keras.layers.BatchNormalization()(x)
x = keras.layers.Dropout(0.3)(x)

# 残差连接
residual = x
x = keras.layers.Dense(256, activation='relu')(x)
x = keras.layers.add([x, residual])  # 残差相加

x = keras.layers.Dense(128, activation='relu')(x)
outputs = keras.layers.Dense(10, activation='softmax')(x)

model = keras.Model(inputs=inputs, outputs=outputs)

# 多输入多输出
title_input = keras.Input(shape=(100,), name='title')
body_input = keras.Input(shape=(500,), name='body')

title_feat = keras.layers.Dense(64, activation='relu')(title_input)
body_feat = keras.layers.Dense(128, activation='relu')(body_input)
merged = keras.layers.concatenate([title_feat, body_feat])

priority_out = keras.layers.Dense(1, activation='sigmoid', name='priority')(merged)
dept_out = keras.layers.Dense(4, activation='softmax', name='department')(merged)

multi_model = keras.Model(
    inputs=[title_input, body_input],
    outputs=[priority_out, dept_out]
)

# === 方式3：子类化 Model（最灵活） ===
class CustomModel(keras.Model):
    def __init__(self, hidden_dim, output_dim):
        super().__init__()
        self.dense1 = keras.layers.Dense(hidden_dim, activation='relu')
        self.bn = keras.layers.BatchNormalization()
        self.drop = keras.layers.Dropout(0.3)
        self.dense2 = keras.layers.Dense(output_dim, activation='softmax')

    def call(self, inputs, training=False):
        x = self.dense1(inputs)
        x = self.bn(x, training=training)
        x = self.drop(x, training=training)
        return self.dense2(x)

model = CustomModel(256, 10)
# 子类化模型需要先调用一次才能 build
model(tf.zeros((1, 784)))
model.summary()
```

| 方式 | 灵活性 | 复杂度 | 适用场景 |
|------|--------|--------|---------|
| Sequential | 低 | 低 | 简单线性模型 |
| Functional | 中 | 中 | 大多数场景（推荐） |
| 子类化 | 高 | 高 | 自定义前向逻辑 |

### 2. 编译与训练

```python
import tensorflow as tf
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    keras.layers.Dense(10, activation='softmax')
])

# === 编译 ===
model.compile(
    optimizer='adam',                           # 优化器
    loss='sparse_categorical_crossentropy',     # 损失函数
    metrics=['accuracy']                        # 评估指标
)

# 自定义优化器
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3, weight_decay=1e-4),
    loss=keras.losses.SparseCategoricalCrossentropy(),
    metrics=[keras.metrics.SparseCategoricalAccuracy()]
)

# === 训练 ===
# 基础训练
history = model.fit(
    X_train, y_train,
    epochs=20,
    batch_size=64,
    validation_split=0.2,
    verbose=1
)

# 使用 tf.data 管道
train_ds = tf.data.Dataset.from_tensor_slices((X_train, y_train))
train_ds = train_ds.shuffle(10000).batch(64).prefetch(tf.data.AUTOTUNE)

val_ds = tf.data.Dataset.from_tensor_slices((X_val, y_val))
val_ds = val_ds.batch(128).prefetch(tf.data.AUTOTUNE)

history = model.fit(train_ds, epochs=20, validation_data=val_ds)

# === 回调函数 ===
callbacks = [
    # 早停
    keras.callbacks.EarlyStopping(
        monitor='val_loss', patience=5, restore_best_weights=True
    ),
    # 学习率衰减
    keras.callbacks.ReduceLROnPlateau(
        monitor='val_loss', factor=0.5, patience=3, min_lr=1e-7
    ),
    # 保存最佳模型
    keras.callbacks.ModelCheckpoint(
        'best_model.keras', monitor='val_accuracy', save_best_only=True
    ),
    # TensorBoard 日志
    keras.callbacks.TensorBoard(log_dir='./logs', histogram_freq=1),
    # 自定义回调
    keras.callbacks.LambdaCallback(
        on_epoch_end=lambda epoch, logs: print(f"  LR: {model.optimizer.learning_rate.numpy():.6f}")
    )
]

history = model.fit(
    X_train, y_train,
    epochs=50,
    batch_size=64,
    validation_split=0.2,
    callbacks=callbacks
)

# === 训练历史可视化 ===
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].plot(history.history['loss'], label='train')
axes[0].plot(history.history['val_loss'], label='val')
axes[0].set_title('Loss')
axes[0].legend()

axes[1].plot(history.history['accuracy'], label='train')
axes[1].plot(history.history['val_accuracy'], label='val')
axes[1].set_title('Accuracy')
axes[1].legend()
plt.show()
```

### 3. tf.data 数据管道

```python
import tensorflow as tf

# === 从内存数据 ===
dataset = tf.data.Dataset.from_tensor_slices((X, y))

# === 从文件 ===
# CSV
dataset = tf.data.experimental.make_csv_dataset(
    'data.csv', batch_size=32, label_name='target'
)

# TFRecord（大规模数据推荐格式）
def parse_example(example_proto):
    feature_desc = {
        'image': tf.io.FixedLenFeature([], tf.string),
        'label': tf.io.FixedLenFeature([], tf.int64)
    }
    parsed = tf.io.parse_single_example(example_proto, feature_desc)
    image = tf.io.decode_raw(parsed['image'], tf.float32)
    return image, parsed['label']

dataset = tf.data.TFRecordDataset('data.tfrecord').map(parse_example)

# === 常用变换 ===
dataset = (
    tf.data.Dataset.from_tensor_slices((X_train, y_train))
    .shuffle(buffer_size=10000)       # 打乱
    .batch(64)                        # 分批
    .map(lambda x, y: (x * 2, y))    # 变换
    .prefetch(tf.data.AUTOTUNE)       # 预取
)

# === 数据增强（图像） ===
data_augmentation = keras.Sequential([
    keras.layers.RandomFlip("horizontal"),
    keras.layers.RandomRotation(0.1),
    keras.layers.RandomZoom(0.1),
    keras.layers.RandomContrast(0.1),
])

# 在模型中嵌入
aug_model = keras.Sequential([
    keras.layers.Input(shape=(224, 224, 3)),
    data_augmentation,
    keras.layers.Rescaling(1./255),
    # ... 后续层
])
```

### 4. 预训练模型与迁移学习

```python
import tensorflow as tf
from tensorflow import keras

# === 使用预训练模型 ===
# ImageNet 预训练的 EfficientNet
base_model = keras.applications.EfficientNetV2B0(
    weights='imagenet',
    include_top=False,        # 去掉分类头
    input_shape=(224, 224, 3)
)

# 冻结基础模型
base_model.trainable = False

# 添加自定义分类头
model = keras.Sequential([
    base_model,
    keras.layers.GlobalAveragePooling2D(),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(256, activation='relu'),
    keras.layers.Dense(5, activation='softmax')  # 5类
])

model.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 第一阶段：只训练分类头
model.fit(train_ds, epochs=10, validation_data=val_ds)

# === 微调 ===
# 解冻部分层
base_model.trainable = True
for layer in base_model.layers[:-20]:  # 只微调最后20层
    layer.trainable = False

# 用更小的学习率重新编译
model.compile(
    optimizer=keras.optimizers.Adam(1e-5),  # 小10倍
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 第二阶段：微调
model.fit(train_ds, epochs=10, validation_data=val_ds)
```

### 5. 自定义训练循环

```python
import tensorflow as tf
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    keras.layers.Dense(10)
])

optimizer = keras.optimizers.Adam(1e-3)
loss_fn = keras.losses.SparseCategoricalCrossentropy(from_logits=True)
train_acc_metric = keras.metrics.SparseCategoricalAccuracy()
val_acc_metric = keras.metrics.SparseCategoricalAccuracy()

@tf.function  # 编译为图模式，加速
def train_step(X, y):
    with tf.GradientTape() as tape:
        logits = model(X, training=True)
        loss = loss_fn(y, logits)
        # 加入正则化损失
        loss += sum(model.losses)

    grads = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(grads, model.trainable_variables))
    train_acc_metric.update_state(y, logits)
    return loss

@tf.function
def val_step(X, y):
    logits = model(X, training=False)
    val_loss = loss_fn(y, logits)
    val_acc_metric.update_state(y, logits)
    return val_loss

# 训练循环
for epoch in range(20):
    # 训练
    for X_batch, y_batch in train_ds:
        loss = train_step(X_batch, y_batch)

    train_acc = train_acc_metric.result()
    train_acc_metric.reset_states()

    # 验证
    for X_batch, y_batch in val_ds:
        val_loss = val_step(X_batch, y_batch)

    val_acc = val_acc_metric.result()
    val_acc_metric.reset_states()

    print(f"Epoch {epoch+1} | Loss: {loss:.4f} | "
          f"Train Acc: {train_acc:.4f} | Val Acc: {val_acc:.4f}")
```

### 6. 模型保存与部署

```python
import tensorflow as tf

# === 保存模型 ===
# Keras 格式（推荐）
model.save('model.keras')

# SavedModel 格式（部署用）
model.save('saved_model/')

# 仅权重
model.save_weights('weights.h5')

# === 加载模型 ===
model = keras.models.load_model('model.keras')
model.load_weights('weights.h5')

# === TF Serving 部署 ===
# 1. 导出 SavedModel
model.save('saved_model/my_model/1/')

# 2. 启动 TF Serving（Docker）
# docker run -p 8501:8501 --mount type=bind,source=/path/to/saved_model,target=/models/my_model \
#   -e MODEL_NAME=my_model tensorflow/serving

# 3. REST API 调用
# import requests
# data = {"instances": X_test[:3].tolist()}
# resp = requests.post('http://localhost:8501/v1/models/my_model:predict', json=data)
# predictions = resp.json()['predictions']

# === TF Lite（移动端） ===
converter = tf.lite.TFLiteConverter.from_keras_model(model)
converter.optimizations = [tf.lite.Optimize.DEFAULT]  # 量化
tflite_model = converter.convert()

with open('model.tflite', 'wb') as f:
    f.write(tflite_model)

# === TF.js（浏览器） ===
# pip install tensorflowjs
# tensorflowjs_converter --input_format=keras model.keras tfjs_model/
```

### 7. 常用损失函数与指标

```python
import tensorflow as tf
from tensorflow import keras

# === 分类 ===
# 二分类
loss_binary = keras.losses.BinaryCrossentropy(from_logits=False)
# 多分类
loss_multi = keras.losses.SparseCategoricalCrossentropy(from_logits=False)
# 标签平滑
loss_smooth = keras.losses.CategoricalCrossentropy(label_smoothing=0.1)
# Focal Loss（类别不平衡）
# 需自定义或用 tensorflow_addons

# === 回归 ===
loss_mse = keras.losses.MeanSquaredError()
loss_mae = keras.losses.MeanAbsoluteError()
loss_huber = keras.losses.Huber(delta=1.0)  # 对离群值鲁棒

# === 自定义损失函数 ===
def focal_loss(gamma=2.0, alpha=0.25):
    def loss(y_true, y_pred):
        y_pred = tf.clip_by_value(y_pred, 1e-7, 1 - 1e-7)
        cross_entropy = -y_true * tf.math.log(y_pred)
        weight = alpha * y_true * tf.math.pow(1 - y_pred, gamma)
        return tf.reduce_mean(weight * cross_entropy)
    return loss

# === 常用指标 ===
metrics = [
    keras.metrics.Accuracy(),
    keras.metrics.Precision(),
    keras.metrics.Recall(),
    keras.metrics.AUC(),
    keras.metrics.F1Score(average='macro'),
]
```

| 场景 | 损失函数 | 最后激活 |
|------|---------|---------|
| 二分类 | BinaryCrossentropy | sigmoid |
| 多分类 | SparseCategoricalCrossentropy | softmax |
| 多标签 | BinaryCrossentropy | sigmoid |
| 回归 | MSE/MAE/Huber | 无 |
| 类别不平衡 | Focal Loss | sigmoid/softmax |

### 8. TensorBoard 可视化

```python
import tensorflow as tf
from tensorflow import keras

# === 基本使用 ===
log_dir = './logs/experiment_1'
tensorboard_cb = keras.callbacks.TensorBoard(
    log_dir=log_dir,
    histogram_freq=1,       # 每个epoch记录权重直方图
    write_graph=True,       # 记录计算图
    update_freq='epoch'     # 记录频率
)

model.fit(X_train, y_train, epochs=20, callbacks=[tensorboard_cb])

# 启动 TensorBoard
# tensorboard --logdir=./logs

# === 自定义日志 ===
writer = tf.summary.create_file_writer(log_dir)

with writer.as_default():
    for step in range(100):
        tf.summary.scalar('custom_metric', value=some_value, step=step)
        tf.summary.histogram('weights', model.layers[0].weights[0], step=step)
        writer.flush()

# === 超参调优可视化 ===
# 使用多目录对比不同实验
for lr in [1e-2, 1e-3, 1e-4]:
    log_dir = f'./logs/lr_{lr}'
    # ... 训练 ...
# TensorBoard 会自动对比不同目录的曲线
```

## 面试题

### 1. TensorFlow 1.x 和 2.x 的核心区别？

TF 1.x 使用静态计算图（define-then-run）：先用 Python 构建图定义，再用 Session 运行，调试困难，入门门槛高。TF 2.x 默认使用动态计算图（eager execution），像普通 Python 代码一样即时执行，支持 `tf.function` 装饰器将函数编译为静态图以提升性能。2.x 将 Keras 作为官方 API（`tf.keras`），大幅简化了代码。2.x 还移除了 `tf.Session`、`tf.placeholder`、`tf.global_variables_initializer` 等 1.x 概念。

### 2. Keras 的 Sequential、Functional、子类化三种方式怎么选？

Sequential 最简单但只能做层的线性堆叠，不支持残差连接和多输入/输出。Functional API 通过层调用语法（`y = Dense()(x)`）构建有向无环图，支持任意拓扑结构，是大多数场景的推荐选择。子类化 Model 最灵活，可以在 `call()` 中写任意 Python 逻辑（条件、循环），但需要自己管理 `training` 参数、不能用 `model.summary()` 查看完整结构（未调用前不知道形状）。原则：能用 Sequential 就用，需要残差/多输入/多输出用 Functional，需要动态前向逻辑用子类化。

### 3. tf.function 的作用和注意事项？

`@tf.function` 将 Python 函数编译为 TensorFlow 计算图，获得更好的性能和部署兼容性。注意事项：（1）函数内不能用 Python 副作用（如修改外部变量、print 只在 trace 时执行一次）；（2）Python 控制流（if/for）在 trace 时按输入形状和类型展开，不同输入可能生成不同图——需要用 `tf.cond`/`tf.while_loop` 或确保输入签名一致；（3）首次调用时会 trace（较慢），后续调用复用图（快）；（4）建议在模型最外层使用，内部小函数保持 eager。2.x 的 `model.compile` 内部已自动使用 tf.function。

### 4. tf.data 的 prefetch 有什么作用？

`prefetch(n)` 让数据加载和模型训练并行：当 GPU 在训练第 i 批时，CPU 在准备第 i+1 批数据，减少 GPU 等待时间。`prefetch(tf.data.AUTOTUNE)` 让 TensorFlow 自动调整预取数量。最佳管道顺序：`dataset.shuffle().batch().map().prefetch()`——shuffle 在 batch 前保证样本级打乱，map 在 batch 后减少 map 调用次数（或设 `num_parallel_calls=AUTOTUNE` 在 batch 前并行 map），prefetch 总是放最后。

### 5. 迁移学习什么时候冻结、什么时候微调？

两阶段策略：第一阶段冻结预训练模型，只训练新增的分类头，让分类头先收敛到合理的范围（否则随机初始化的梯度会破坏预训练权重）；第二阶段解冻部分或全部预训练层，用极小的学习率（1e-5 量级）联合微调。冻结多少层：数据量小且与预训练数据相似（如都用自然图像）→冻结更多层甚至全部；数据量大或与预训练差异大→解冻更多层。微调时学习率必须是预训练阶段的 1/10 到 1/100，否则会"灾难性遗忘"预训练知识。

### 6. Keras 中 from_logits 参数的含义？

`from_logits` 告诉损失函数模型输出是概率值还是原始 logits。Logits 是 softmax/sigmoid 之前的原始值，概率是经过激活函数后的 0-1 值。设 `from_logits=True` 时，损失函数内部会先应用 softmax/sigmoid 再计算损失，数值上更稳定（避免先 softmax 再 log 的精度损失）。设 `from_logits=False` 时，假设输入已经是概率。最佳实践：模型最后一层不设激活（输出 logits），损失函数设 `from_logits=True`，这样数值更稳定且推理时手动应用 softmax。

### 7. SavedModel 和 Keras 格式有什么区别？

Keras 格式（`.keras`）保存完整模型（架构+权重+优化器状态+编译配置），只能用 Keras 加载，适合训练中断恢复。SavedModel 格式（目录）包含计算图定义和权重，是 TF 的标准部署格式，可被 TF Serving、TF Lite、TF.js 等所有 TF 工具链使用，不需要原始 Python 代码。实践中：训练阶段用 `.keras` 格式保存检查点，部署时导出 SavedModel。旧版 `.h5` 格式已被弃用，新项目不应使用。

### 8. 如何处理 TF 中的类别不平衡？

三种方式：（1）**class_weight**——`model.fit(class_weight={0: 1.0, 1: 5.0})`，给少数类更高权重，最简单；（2）**sample_weight**——逐样本权重，更精细（如按样本难度加权）；（3）**自定义损失**——如 Focal Loss，自动降低易分类样本的权重、关注难分类样本。评估时不看准确率（95%负样本下全猜负也有95%），看 Precision/Recall/F1/AUC。数据层面也可用过采样（但 TF 原生不支持 SMOTE，需用 imbalanced-learn 预处理）。

## 外部参考

- [TensorFlow 官方文档](https://www.tensorflow.org/)

---
tags:
  - Python
  - 机器学习
  - scikit-learn
  - sklearn
---

# scikit-learn与机器学习

## What — scikit-learn 是什么

scikit-learn（sklearn）是 Python 最流行的通用机器学习库，基于 [[NumPy与数值计算]]、[[SciPy与科学计算]] 和 [[Matplotlib与数据可视化]] 构建，提供监督学习、无监督学习、模型选择与评估、数据预处理等完整工具链。API 设计遵循统一的 fit/predict/transform 模式，简洁一致。

核心模块：

| 模块 | 说明 | 典型类 |
|------|------|--------|
| sklearn.linear_model | 线性模型 | LogisticRegression, Ridge, Lasso |
| sklearn.ensemble | 集成方法 | RandomForest, GradientBoosting |
| sklearn.svm | 支持向量机 | SVC, SVR |
| sklearn.neighbors | 近邻方法 | KNeighborsClassifier |
| sklearn.tree | 决策树 | DecisionTreeClassifier |
| sklearn.naive_bayes | 朴素贝叶斯 | GaussianNB |
| sklearn.cluster | 聚类 | KMeans, DBSCAN |
| sklearn.decomposition | 降维 | PCA, NMF |
| sklearn.model_selection | 模型选择 | GridSearchCV, cross_val_score |
| sklearn.metrics | 评估指标 | accuracy, f1, roc_auc |
| sklearn.preprocessing | 预处理 | StandardScaler, OneHotEncoder |
| sklearn.pipeline | 流水线 | Pipeline, ColumnTransformer |

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

# sklearn 典型工作流
X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

clf = RandomForestClassifier(n_estimators=100, random_state=42)
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)

print(classification_report(y_test, y_pred))
```

## Why — 为什么选择 scikit-learn

### 统一 API 设计

所有模型遵循相同的接口模式，学习一个就能用所有：

| 方法 | 说明 | 适用对象 |
|------|------|---------|
| fit(X, y) | 训练模型 | 所有估计器 |
| predict(X) | 预测 | 分类/回归器 |
| transform(X) | 变换 | 预处理器 |
| fit_transform(X) | 训练+变换 | 预处理器 |
| predict_proba(X) | 预测概率 | 分类器 |
| score(X, y) | 评估 | 所有估计器 |

### 生态定位

scikit-learn 覆盖经典机器学习的全流程，但不包含深度学习（用 [[PyTorch基础]] 或 [[TensorFlow与Keras]]）、不包含大规模分布式（用 Spark MLlib）、不包含自动特征工程（用 Featuretools）。它是中小规模结构化数据的首选，也是理解机器学习概念的最佳入口。

## How — 如何使用 scikit-learn

### 1. 监督学习：分类

```python
import numpy as np
from sklearn.datasets import make_classification, load_breast_cancer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    classification_report, confusion_matrix, roc_auc_score, roc_curve
)
import matplotlib.pyplot as plt

# === 数据准备 ===
X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# === 逻辑回归 ===
from sklearn.linear_model import LogisticRegression

pipe_lr = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', LogisticRegression(max_iter=1000, random_state=42))
])
pipe_lr.fit(X_train, y_train)
y_pred_lr = pipe_lr.predict(X_test)
print(f"逻辑回归准确率: {accuracy_score(y_test, y_pred_lr):.4f}")

# === 随机森林 ===
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=100, max_depth=5, random_state=42)
rf.fit(X_train, y_train)
y_pred_rf = rf.predict(X_test)
print(f"随机森林准确率: {accuracy_score(y_test, y_pred_rf):.4f}")

# 特征重要性
importances = rf.feature_importances_
top5 = np.argsort(importances)[-5:][::-1]
print(f"Top5 重要特征: {top5}")

# === SVM ===
from sklearn.svm import SVC

pipe_svm = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', SVC(kernel='rbf', C=1.0, random_state=42))
])
pipe_svm.fit(X_train, y_train)
print(f"SVM 准确率: {pipe_svm.score(X_test, y_test):.4f}")

# === KNN ===
from sklearn.neighbors import KNeighborsClassifier

pipe_knn = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', KNeighborsClassifier(n_neighbors=5))
])
pipe_knn.fit(X_train, y_train)
print(f"KNN 准确率: {pipe_knn.score(X_test, y_test):.4f}")

# === 模型对比 ===
models = {
    'LogisticRegression': pipe_lr,
    'RandomForest': rf,
    'SVM': pipe_svm,
    'KNN': pipe_knn
}

for name, model in models.items():
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='f1')
    print(f"{name}: F1={scores.mean():.4f}±{scores.std():.4f}")

# === 评估指标详解 ===
print("\n=== 分类报告 ===")
print(classification_report(y_test, y_pred_rf))

print("=== 混淆矩阵 ===")
print(confusion_matrix(y_test, y_pred_rf))

# ROC 曲线
y_prob = rf.predict_proba(X_test)[:, 1]
fpr, tpr, thresholds = roc_curve(y_test, y_prob)
auc = roc_auc_score(y_test, y_prob)

plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, label=f'ROC (AUC={auc:.3f})')
plt.plot([0, 1], [0, 1], 'k--')
plt.xlabel('假正率')
plt.ylabel('真正率')
plt.title('ROC 曲线')
plt.legend()
plt.show()
```

| 分类器 | 核心思想 | 需要缩放 | 可解释性 | 适合规模 |
|--------|---------|---------|---------|---------|
| LogisticRegression | 线性决策边界 | 是 | 高 | 大 |
| RandomForest | 多棵树投票 | 否 | 中 | 中 |
| SVM | 最大间隔超平面 | 是 | 低 | 中 |
| KNN | 最近邻投票 | 是 | 低 | 小 |
| NaiveBayes | 贝叶斯+条件独立 | 否 | 高 | 大 |
| GradientBoosting | 逐步修正残差 | 否 | 中 | 中 |

### 2. 监督学习：回归

```python
import numpy as np
from sklearn.datasets import fetch_california_housing
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

X, y = fetch_california_housing(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# === 线性回归 ===
from sklearn.linear_model import LinearRegression, Ridge, Lasso, ElasticNet

lr = LinearRegression()
lr.fit(X_train, y_train)
print(f"线性回归 R²: {lr.score(X_test, y_test):.4f}")

# 岭回归（L2正则化）
pipe_ridge = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', Ridge(alpha=1.0))
])
pipe_ridge.fit(X_train, y_train)
print(f"岭回归 R²: {pipe_ridge.score(X_test, y_test):.4f}")

# Lasso（L1正则化，特征选择）
pipe_lasso = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', Lasso(alpha=0.01))
])
pipe_lasso.fit(X_train, y_train)
n_nonzero = np.sum(pipe_lasso.named_steps['clf'].coef_ != 0)
print(f"Lasso R²: {pipe_lasso.score(X_test, y_test):.4f}, 非零特征: {n_nonzero}")

# === 随机森林回归 ===
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor

rf_reg = RandomForestRegressor(n_estimators=100, max_depth=10, random_state=42)
rf_reg.fit(X_train, y_train)
y_pred_rf = rf_reg.predict(X_test)
print(f"\nRF R²: {r2_score(y_test, y_pred_rf):.4f}")
print(f"RF RMSE: {np.sqrt(mean_squared_error(y_test, y_pred_rf)):.4f}")

# GBDT 回归
gb_reg = GradientBoostingRegressor(n_estimators=200, max_depth=5, learning_rate=0.1, random_state=42)
gb_reg.fit(X_train, y_train)
print(f"GBDT R²: {gb_reg.score(X_test, y_test):.4f}")

# === 回归评估 ===
def regression_metrics(y_true, y_pred, name=''):
    print(f"=== {name} ===")
    print(f"MAE:  {mean_absolute_error(y_true, y_pred):.4f}")
    print(f"RMSE: {np.sqrt(mean_squared_error(y_true, y_pred)):.4f}")
    print(f"R²:   {r2_score(y_true, y_pred):.4f}")

regression_metrics(y_test, y_pred_rf, '随机森林')
```

### 3. 无监督学习

```python
import numpy as np
from sklearn.datasets import make_blobs, load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.decomposition import PCA
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# === K-Means 聚类 ===
X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.6, random_state=42)

# 确定 K：肘部法 + 轮廓系数
inertias = []
silhouettes = []
K_range = range(2, 10)

for k in K_range:
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    labels = km.fit_predict(X)
    inertias.append(km.inertia_)
    silhouettes.append(silhouette_score(X, labels))

fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].plot(K_range, inertias, 'bo-')
axes[0].set_title('肘部法')
axes[0].set_xlabel('K')
axes[0].set_ylabel('Inertia')

axes[1].plot(K_range, silhouettes, 'ro-')
axes[1].set_title('轮廓系数')
axes[1].set_xlabel('K')
axes[1].set_ylabel('Silhouette')

plt.tight_layout()
plt.show()

# 最优 K=4 聚类
km = KMeans(n_clusters=4, n_init=10, random_state=42)
labels_km = km.fit_predict(X)

plt.figure(figsize=(8, 6))
plt.scatter(X[:, 0], X[:, 1], c=labels_km, cmap='viridis', s=30)
plt.scatter(km.cluster_centers_[:, 0], km.cluster_centers_[:, 1],
            c='red', marker='x', s=200)
plt.title('K-Means 聚类')
plt.show()

# === DBSCAN 聚类 ===
db = DBSCAN(eps=0.5, min_samples=5)
labels_db = db.fit_predict(X)
n_clusters = len(set(labels_db)) - (1 if -1 in labels_db else 0)
n_noise = (labels_db == -1).sum()
print(f"DBSCAN: {n_clusters} 个簇, {n_noise} 个噪声点")

# === 层次聚类 ===
agg = AgglomerativeClustering(n_clusters=4, linkage='ward')
labels_agg = agg.fit_predict(X)

# === 降维 + 聚类可视化 ===
iris = load_iris()
X_iris = StandardScaler().fit_transform(iris.data)

pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_iris)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))
axes[0].scatter(X_pca[:, 0], X_pca[:, 1], c=iris.target, cmap='Set1')
axes[0].set_title('真实标签')

km_iris = KMeans(n_clusters=3, n_init=10, random_state=42)
axes[1].scatter(X_pca[:, 0], X_pca[:, 1], c=km_iris.fit_predict(X_iris), cmap='Set1')
axes[1].set_title('K-Means')

db_iris = DBSCAN(eps=0.8, min_samples=5)
labels_db_iris = db_iris.fit_predict(X_iris)
axes[2].scatter(X_pca[:, 0], X_pca[:, 1], c=labels_db_iris, cmap='Set1')
axes[2].set_title('DBSCAN')

plt.tight_layout()
plt.show()
```

| 聚类方法 | 需要指定K | 形状 | 噪声处理 | 速度 |
|----------|---------|------|---------|------|
| K-Means | 是 | 球形 | 差 | 快 |
| DBSCAN | 否 | 任意 | 好 | 中 |
| 层次聚类 | 是/否 | 任意 | 差 | 慢 |
| GMM | 是 | 椭球 | 中 | 中 |

### 4. Pipeline 与 ColumnTransformer

```python
import numpy as np
import pandas as pd
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

rng = np.random.default_rng(42)

# 混合类型数据
n = 500
df = pd.DataFrame({
    'age': rng.normal(40, 12, n),
    'income': rng.exponential(50000, n),
    'score': rng.uniform(0, 100, n),
    'city': rng.choice(['Beijing', 'Shanghai', 'Guangzhou'], n),
    'education': rng.choice(['High', 'Bachelor', 'Master'], n),
})
y = (df['income'] > df['income'].median()).astype(int)

# 定义列组
num_cols = ['age', 'income', 'score']
cat_cols = ['city', 'education']

# 数值 Pipeline
num_pipe = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler()),
])

# 类别 Pipeline
cat_pipe = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('encoder', OneHotEncoder(handle_unknown='ignore', sparse_output=False)),
])

# ColumnTransformer 组合
preprocessor = ColumnTransformer([
    ('num', num_pipe, num_cols),
    ('cat', cat_pipe, cat_cols),
])

# 完整 Pipeline
pipe = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42)),
])

scores = cross_val_score(pipe, df, y, cv=5, scoring='accuracy')
print(f"Pipeline CV: {scores.mean():.4f}±{scores.std():.4f}")

# Pipeline 优势：
# 1. 防止数据泄露（fit 只在训练集上）
# 2. 代码简洁，一步部署
# 3. 可序列化（joblib.dump）
```

### 5. 模型选择与超参调优

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import (
    train_test_split, GridSearchCV, RandomizedSearchCV,
    cross_val_score, learning_curve, validation_curve
)
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
import numpy as np
import matplotlib.pyplot as plt

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# === GridSearchCV ===
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', SVC())
])

param_grid = {
    'clf__C': [0.1, 1, 10],
    'clf__kernel': ['rbf', 'linear'],
    'clf__gamma': ['scale', 'auto']
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', n_jobs=-1, verbose=1)
grid.fit(X_train, y_train)
print(f"最佳参数: {grid.best_params_}")
print(f"最佳CV分数: {grid.best_score_:.4f}")
print(f"测试集分数: {grid.score(X_test, y_test):.4f}")

# === RandomizedSearchCV（更高效） ===
from scipy.stats import loguniform

param_dist = {
    'clf__C': loguniform(1e-2, 1e2),
    'clf__gamma': loguniform(1e-3, 1e1),
    'clf__kernel': ['rbf', 'linear']
}

random_search = RandomizedSearchCV(
    pipe, param_dist, n_iter=50, cv=5,
    scoring='accuracy', random_state=42, n_jobs=-1
)
random_search.fit(X_train, y_train)
print(f"\n随机搜索最佳: {random_search.best_params_}")
print(f"最佳CV分数: {random_search.best_score_:.4f}")

# === 学习曲线 ===
train_sizes, train_scores, val_scores = learning_curve(
    RandomForestClassifier(n_estimators=100, random_state=42),
    X_train, y_train, cv=5, train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy', n_jobs=-1
)

plt.figure(figsize=(8, 5))
plt.plot(train_sizes, train_scores.mean(axis=1), 'o-', label='训练集')
plt.plot(train_sizes, val_scores.mean(axis=1), 'o-', label='验证集')
plt.fill_between(train_sizes, train_scores.mean(axis=1) - train_scores.std(axis=1),
                 train_scores.mean(axis=1) + train_scores.std(axis=1), alpha=0.1)
plt.fill_between(train_sizes, val_scores.mean(axis=1) - val_scores.std(axis=1),
                 val_scores.mean(axis=1) + val_scores.std(axis=1), alpha=0.1)
plt.xlabel('训练样本数')
plt.ylabel('准确率')
plt.title('学习曲线')
plt.legend()
plt.show()

# === 验证曲线 ===
param_range = [1, 3, 5, 10, 20, 50]
train_scores, val_scores = validation_curve(
    RandomForestClassifier(n_estimators=100, random_state=42),
    X_train, y_train, param_name='max_depth', param_range=param_range,
    cv=5, scoring='accuracy', n_jobs=-1
)

plt.figure(figsize=(8, 5))
plt.plot(param_range, train_scores.mean(axis=1), 'o-', label='训练集')
plt.plot(param_range, val_scores.mean(axis=1), 'o-', label='验证集')
plt.xlabel('max_depth')
plt.ylabel('准确率')
plt.title('验证曲线')
plt.legend()
plt.show()
```

| 调优方法 | 搜索策略 | 计算量 | 找到最优概率 |
|----------|---------|--------|-------------|
| GridSearchCV | 穷举 | 高 | 保证 |
| RandomizedSearchCV | 随机采样 | 可控 | 较高 |
| Optuna/Hyperopt | 贝叶斯优化 | 低 | 高 |

### 6. 交叉验证策略

```python
from sklearn.model_selection import (
    KFold, StratifiedKFold, GroupKFold, TimeSeriesSplit, RepeatedStratifiedKFold
)
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
import numpy as np

X, y = make_classification(n_samples=200, n_classes=2, weights=[0.7, 0.3], random_state=42)

# === KFold（通用） ===
kf = KFold(n_splits=5, shuffle=True, random_state=42)

# === StratifiedKFold（分类推荐，保持类别比例） ===
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

# === GroupKFold（同组数据不跨折） ===
groups = np.repeat(np.arange(50), 4)  # 50组，每组4个样本
gkf = GroupKFold(n_splits=5)

# === TimeSeriesSplit（时间序列） ===
tscv = TimeSeriesSplit(n_splits=5)

# === RepeatedStratifiedKFold（更稳定） ===
rskf = RepeatedStratifiedKFold(n_splits=5, n_repeats=3, random_state=42)

from sklearn.model_selection import cross_val_score
model = LogisticRegression(max_iter=1000)

scores = cross_val_score(model, X, y, cv=skf, scoring='f1')
print(f"StratifiedKFold F1: {scores.mean():.4f}±{scores.std():.4f}")

scores_rep = cross_val_score(model, X, y, cv=rskf, scoring='f1')
print(f"RepeatedStratifiedKFold F1: {scores_rep.mean():.4f}±{scores_rep.std():.4f}")
```

| CV策略 | 说明 | 适用场景 |
|--------|------|---------|
| KFold | 随机等分 | 回归、大数据 |
| StratifiedKFold | 分层等分 | 分类（类别不平衡） |
| GroupKFold | 按组划分 | 同组不跨折 |
| TimeSeriesSplit | 时序划分 | 时间序列 |
| RepeatedStratifiedKFold | 重复分层 | 需要稳定估计 |

### 7. 类别不平衡处理

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, f1_score
from sklearn.utils import resample

# 生成不平衡数据（90% vs 10%）
X, y = make_classification(n_samples=1000, n_classes=2, weights=[0.9, 0.1], random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

print(f"训练集类别分布: {np.bincount(y_train)}")

# === 方法1：class_weight ===
clf_weighted = RandomForestClassifier(n_estimators=100, class_weight='balanced', random_state=42)
clf_weighted.fit(X_train, y_train)
y_pred_w = clf_weighted.predict(X_test)
print(f"\nclass_weight='balanced' F1: {f1_score(y_test, y_pred_w):.4f}")

# === 方法2：过采样（SMOTE） ===
# pip install imbalanced-learn
from imblearn.over_sampling import SMOTE
from imblearn.pipeline import Pipeline as ImbPipeline

pipe_smote = ImbPipeline([
    ('smote', SMOTE(random_state=42)),
    ('clf', RandomForestClassifier(n_estimators=100, random_state=42))
])
pipe_smote.fit(X_train, y_train)
y_pred_smote = pipe_smote.predict(X_test)
print(f"SMOTE F1: {f1_score(y_test, y_pred_smote):.4f}")

# === 方法3：欠采样 ===
from imblearn.under_sampling import RandomUnderSampler

pipe_under = ImbPipeline([
    ('undersample', RandomUnderSampler(random_state=42)),
    ('clf', RandomForestClassifier(n_estimators=100, random_state=42))
])
pipe_under.fit(X_train, y_train)
y_pred_under = pipe_under.predict(X_test)
print(f"欠采样 F1: {f1_score(y_test, y_pred_under):.4f}")

# === 方法4：阈值调整 ===
clf = RandomForestClassifier(n_estimators=100, random_state=42)
clf.fit(X_train, y_train)
y_prob = clf.predict_proba(X_test)[:, 1]

# 搜索最佳阈值
from sklearn.metrics import precision_recall_curve
precisions, recalls, thresholds = precision_recall_curve(y_test, y_prob)
f1_scores = 2 * precisions * recalls / (precisions + recalls + 1e-8)
best_threshold = thresholds[np.argmax(f1_scores)]
print(f"\n最佳阈值: {best_threshold:.3f}")
y_pred_thresh = (y_prob >= best_threshold).astype(int)
print(f"阈值调整 F1: {f1_score(y_test, y_pred_thresh):.4f}")
```

### 8. 模型持久化与部署

```python
import joblib
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

# 训练
X, y = load_iris(return_X_y=True)
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', RandomForestClassifier(n_estimators=100, random_state=42))
])
pipe.fit(X, y)

# 保存
joblib.dump(pipe, 'model.joblib', compress=3)

# 加载
loaded_pipe = joblib.load('model.joblib')
print(f"加载模型预测: {loaded_pipe.predict(X[:3])}")

# === 模型版本信息 ===
import sklearn
print(f"scikit-learn 版本: {sklearn.__version__}")
# 注意：不同版本间模型可能不兼容，部署时需锁定版本
```

## 面试题

### 1. sklearn 的 fit/transform/predict 模式有什么好处？

统一接口带来三大好处：（1）**可替换性**——换模型只需改一行，Pipeline 和交叉验证等基础设施自动适配；（2）**可组合性**——预处理和模型通过 Pipeline 组合，确保 fit 只在训练数据上执行，防止数据泄露；（3）**可复现性**——随机种子 + Pipeline 保存，整个流程可精确复现。这是 sklearn 被广泛采用的核心设计哲学：一致性比特殊性更重要。

### 2. GridSearchCV 和 RandomizedSearchCV 怎么选？

GridSearchCV 穷举所有参数组合，保证找到搜索空间内的最优，但计算量是各参数取值数的乘积（3×2×3=18种组合需18次CV），参数多时爆炸。RandomizedSearchCV 随机采样 n_iter 个组合，计算量可控，且研究表明对超参优化问题，随机搜索比网格搜索更高效——因为重要参数往往只有少数几个，随机采样更可能在重要维度上探索更多值。建议：参数空间小（<50种组合）用 Grid；参数空间大或有连续参数用 Randomized；追求最优用 Optuna 贝叶斯优化。

### 3. 交叉验证为什么比单次 train/test 划分更好？

单次划分的结果依赖随机种子，可能因划分方式不同而波动很大（尤其小数据集）。交叉验证用所有数据轮换做训练和验证，给出性能的均值和方差，更可靠地估计泛化能力。分类问题必须用 StratifiedKFold 保持类别比例。但交叉验证计算量是单次划分的 k 倍，大数据集或慢模型可能不可接受。另外，交叉验证用于模型选择和评估，最终部署的模型应该用全部数据重新训练。

### 4. 随机森林和梯度提升树的区别？

随机森林（Bagging）：多棵独立树并行训练，每棵用 bootstrap 采样和随机特征子集，最终投票/平均。方差低、不容易过拟合、训练快（可并行）、对超参不敏感。梯度提升树（Boosting）：树串行训练，每棵拟合前一棵的残差，逐步降低偏差。精度通常更高、但容易过拟合、训练慢（串行）、对超参敏感。实践原则：数据量大+不需要极致精度→随机森林；追求最高精度+愿意调参→梯度提升；工业部署用 XGBoost/LightGBM（sklearn 版本较慢）。

### 5. 如何处理类别不平衡？

四种策略按推荐顺序：（1）**class_weight='balanced'**——最简单，给少数类更高权重，无需改变数据，适合大多数场景；（2）**阈值调整**——不用默认0.5，按 F1 或业务目标搜索最佳阈值；（3）**SMOTE 过采样**——合成少数类样本，适合严重不平衡且少数类样本极少的情况，但要防过拟合；（4）**欠采样**——丢弃多数类，适合大数据集且少数类样本绝对数量足够。评估指标不能只看准确率（95%负样本下全猜负也有95%），应看 F1、AUC-PR 或少数类的 Recall。

### 6. 什么是数据泄露？sklearn Pipeline 如何防止？

数据泄露指训练时无意使用了测试时不可得的信息。典型场景：在全部数据上做 StandardScaler 的 fit，然后才划分——测试集的均值/标准差泄露到训练中。Pipeline 的防护机制：`fit` 只对训练数据执行所有步骤的 fit，`predict`/`transform` 对测试数据只执行 transform。这意味着 StandardScaler 的均值/标准差只在训练集上计算，测试集使用训练集的统计量做变换。同样，特征选择、目标编码等步骤也必须只在训练集上 fit——Pipeline 自动保证这一点。

### 7. K-Means 的局限性和替代方案？

K-Means 局限：（1）必须预先指定 K（可用肘部法/轮廓系数辅助确定）；（2）只能发现球形簇（基于欧氏距离），非凸形状（如月牙形）无法正确聚类；（3）对初始化敏感（虽然 k-means++ 缓解了这个问题）；（4）对离群值敏感（中心点会被拉偏）；（5）每次迭代重算所有点的分配，大数据集较慢。替代方案：DBSCAN 可发现任意形状、自动识别噪声点、不需要指定 K，但 eps 参数难选；GMM 可发现椭球形簇并给出概率归属；层次聚类可看不同粒度的树状结构。

### 8. sklearn 模型部署的注意事项？

关键注意事项：（1）**版本锁定**——不同 sklearn 版本的模型可能不兼容，部署环境必须与训练环境版本一致，用 `joblib` 保存时记录 `sklearn.__version__`；（2）**Pipeline 序列化**——不要单独保存模型，保存完整 Pipeline（含预处理），否则预处理参数会丢失；（3）**安全问题**——joblib 使用 pickle，加载不受信任的模型文件有代码执行风险，生产环境考虑 ONNX 格式；（4）**延迟考虑**——sklearn 是批处理设计，单条推理有 Python 开销，高并发场景考虑转为 ONNX Runtime 或用微服务批处理；（5）**特征漂移**——部署后监控输入特征分布，与训练数据漂移时模型性能会下降。

## 外部参考

- [scikit-learn 官方文档](https://scikit-learn.org/stable/)

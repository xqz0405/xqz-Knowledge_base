---
tags:
  - Python
  - DRF
  - REST
  - API
  - 序列化
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Django REST Framework

## What — 是什么

> Django REST Framework（DRF）是 Django 生态中最流行的 API 开发框架，提供序列化器、视图集、认证、权限、分页、过滤、文档生成等完整的 RESTful API 工具链。它基于 Django 的 MTV 架构，将模型数据自动转换为 JSON，大幅减少 API 开发的重复工作。

**核心概念：**

- **序列化器（Serializer）**：模型 ↔ JSON 的双向转换器，包含验证逻辑
- **视图集（ViewSet）**：将 CRUD 操作封装为类，自动映射到 HTTP 方法
- **路由器（Router）**：自动生成 URL 配置，与 ViewSet 配合
- **认证与权限**：JWT/Session/OAuth + 自定义权限类

**关键特性：**

- 自带可浏览 API（Browserable API），开发调试极方便
- Serializer 支持嵌套、自定义字段、验证器和只读/只写控制
- ViewSet + Router 几行代码实现完整 CRUD REST API
- 内置分页、过滤、限流（Throttling）中间件
- 与 Django ORM 深度集成，ModelSerializer 自动生成字段

**DRF 请求生命周期：**

```
HTTP 请求
  ↓
Django Middleware
  ↓
URL Router → DRF Router 匹配
  ↓
APIView.dispatch()
  ↓
认证（Authentication）→ 权限（Permission）→ 限流（Throttle）
  ↓
ViewSet action / APIView method
  ↓
Serializer 验证 + 序列化/反序列化
  ↓
Response（JSON / Browsable API）
```

## Why — 为什么

**适用场景：**

- Django 项目的 RESTful API 开发
- 前后端分离架构的后端 API
- 移动端/第三方集成的 API 服务
- 需要 Admin + API 的全栈项目

**API 框架对比：**

| 维度 | DRF | FastAPI | Flask-RESTX |
|------|-----|---------|-------------|
| 基础框架 | Django | Starlette | Flask |
| ORM 集成 | Django ORM 原生 | 需 SQLAlchemy | 需 SQLAlchemy |
| 自动文档 | 需 drf-spectacular | 内置 OpenAPI | 内置 Swagger |
| 认证 | 内置多种 | 需自实现 | 需扩展 |
| 异步 | 有限 | 原生 | 有限 |
| 学习曲线 | 中 | 低 | 低 |

**优缺点：**

- ✅ 优点：
  - 与 Django 无缝集成，共享模型/认证/中间件
  - 可浏览 API 大幅提升开发体验
  - ViewSet + Router 极少代码实现完整 CRUD
  - 社区成熟，扩展丰富
- ❌ 缺点：
  - 依赖 Django，不能独立使用
  - 性能不如 FastAPI（同步为主）
  - 默认文档生成需额外配置（drf-spectacular）
  - 序列化器嵌套深度大时性能下降

## How — 怎么用

### 快速上手

```python
# 安装
# pip install djangorestframework

# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
]

# serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'price', 'published_date']
        read_only_fields = ['id']

# views.py
from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# urls.py
from rest_framework.routers import DefaultRouter
from .views import BookViewSet

router = DefaultRouter()
router.register(r'books', BookViewSet)

urlpatterns = router.urls
# 自动生成: GET/POST /books/  GET/PUT/PATCH/DELETE /books/{id}/
```

### 代码示例1：高级序列化器

```python
from rest_framework import serializers
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class BookDetailSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    author_id = serializers.PrimaryKeyRelatedField(
        queryset=User.objects.all(), source='author', write_only=True
    )
    price_display = serializers.SerializerMethodField()
    is_discounted = serializers.SerializerMethodField()

    class Meta:
        model = Book
        fields = [
            'id', 'title', 'author', 'author_id',
            'price', 'price_display', 'is_discounted',
            'published_date', 'isbn'
        ]

    def get_price_display(self, obj):
        return f"¥{obj.price:.2f}"

    def get_is_discounted(self, obj):
        return obj.price < 50

    def validate_isbn(self, value):
        if value and len(value) != 13:
            raise serializers.ValidationError("ISBN 必须为13位")
        return value

    def validate(self, data):
        if data.get('price', 0) < 0:
            raise serializers.ValidationError({"price": "价格不能为负"})
        return data


# 自定义字段
class TagListField(serializers.Field):
    """逗号分隔的标签列表 ↔ JSON 数组"""
    def to_representation(self, value):
        return value.split(',') if value else []

    def to_internal_value(self, data):
        if not isinstance(data, list):
            raise serializers.ValidationError("期望列表")
        return ','.join(str(item) for item in data)
```

### 代码示例2：视图集、权限与过滤

```python
from rest_framework import viewsets, permissions, filters, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend

# 自定义权限
class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user

# ViewSet
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.select_related('author').all()
    serializer_class = BookDetailSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly, IsAuthorOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['author', 'published_date']
    search_fields = ['title', 'author__username']
    ordering_fields = ['price', 'published_date']
    ordering = ['-published_date']

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    # 自定义 action
    @action(detail=False, methods=['get'])
    def recent(self, request):
        """最近发布的书"""
        books = self.queryset.order_by('-published_date')[:10]
        serializer = self.get_serializer(books, many=True)
        return Response(serializer.data)

    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        """发布书籍"""
        book = self.get_object()
        book.is_published = True
        book.save()
        return Response({'status': 'published'})

# 只读 ViewSet
class BookReadOnlyViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Book.objects.filter(is_published=True)
    serializer_class = BookDetailSerializer


# 函数式 API 视图
from rest_framework.decorators import api_view, permission_classes
from rest_framework import throttling

class BurstThrottle(throttling.AnonRateThrottle):
    rate = '5/min'

@api_view(['GET'])
@permission_classes([permissions.AllowAny])
@throttling_classes([BurstThrottle])
def book_stats(request):
    return Response({
        'total_books': Book.objects.count(),
        'avg_price': Book.objects.avg('price'),
    })
```

### 代码示例3：认证与分页配置

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour',
    },
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# JWT 认证
# pip install djangorestframework-simplejwt
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]

# 自定义分页
from rest_framework.pagination import CursorPagination

class BookCursorPagination(CursorPagination):
    ordering = '-published_date'
    page_size = 20

class BookViewSet(viewsets.ModelViewSet):
    pagination_class = BookCursorPagination
    # ...
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| N+1 查询 | Serializer 访问关联对象 | `select_related`/`prefetch_related` 在 queryset 中 |
| 嵌套写入不生效 | 默认嵌套 Serializer 是只读 | 自定义 `create()`/`update()` 方法 |
| 权限不生效 | 权限类顺序或 `has_object_permission` 未触发 | 确保 `get_object()` 被调用 |
| 分页和过滤冲突 | 分页在过滤后生效 | 正常行为，确保 queryset 先过滤 |
| 大文件上传超时 | DRF 默认解析器全部加载到内存 | 用 `MultiPartParser` + 流式处理 |

### 最佳实践

- ViewSet + Router 为主，复杂逻辑用 `@action`
- `select_related`/`prefetch_related` 优化 queryset
- 全局默认权限 `IsAuthenticated`，公开 API 单独 `AllowAny`
- 用 SimpleJWT 做 token 认证
- 分页用 `PageNumberPagination` 或 `CursorPagination`
- 文档用 `drf-spectacular` 自动生成 OpenAPI 3.0
- 测试用 `APIClient`，DRF 自带测试工具

## 面试题

**Q1: ModelSerializer 和 Serializer 有什么区别？**
> ModelSerializer 自动根据 Model 的字段生成 Serializer 字段，包括 `create()` 和 `update()` 方法的默认实现。Serializer 需要手动定义每个字段和写入逻辑。ModelSerializer 是 80% 场景的选择（CRUD API），Serializer 适合非模型数据（登录表单、聚合数据）。可以用 `Meta.fields`/`exclude` 控制字段、`extra_kwargs` 定制属性。

**Q2: ViewSet 的 action 是如何映射到 HTTP 方法的？**
> `ModelViewSet` 继承了 `CreateModelMixin`/`RetrieveModelMixin` 等，自动映射：`list` → GET 列表、`create` → POST、`retrieve` → GET 详情、`update` → PUT、`partial_update` → PATCH、`destroy` → DELETE。`@action` 装饰器自定义 action：`detail=False` 映射到列表级 URL，`detail=True` 映射到实例级 URL，`methods` 指定 HTTP 方法。

**Q3: DRF 的认证和权限有什么区别？**
> 认证（Authentication）确定"你是谁"——验证身份（JWT/Session/Basic Auth），将用户信息附加到 `request.user`。权限（Permission）确定"你能做什么"——检查 `request.user` 是否有权限执行当前操作。认证在权限之前执行，认证失败返回 401，权限失败返回 403。两者可以组合使用多个类。

**Q4: 如何在 DRF 中实现嵌套资源的创建？**
> 方法1：在 Serializer 的 `create()` 方法中手动处理嵌套数据——弹出嵌套字典，先创建关联对象，再创建主对象。方法2：用 `PrimaryKeyRelatedField` 接收关联对象的 ID 而非完整数据。方法3：分开两个端点——先创建子资源，再在父资源中引用。方法1 适合需要原子性创建的场景，方法2 最简单常用。

**Q5: DRF 的限流（Throttling）是如何工作的？**
> 限流基于缓存（默认 Django cache）记录每个用户/IP 的请求次数。`AnonRateThrottle` 按 IP 限制匿名用户，`UserRateThrottle` 按用户 ID 限制认证用户，`ScopedRateThrottle` 按 scope 限制特定操作。请求到达时，限流类检查缓存计数，超限返回 429 Too Many Requests。可全局配置默认限流率，也可在视图级别用 `@throttle_classes` 覆盖。

**Q6: DRF 如何生成 API 文档？**
> 推荐用 `drf-spectacular`（支持 OpenAPI 3.0）：安装后配置 `DEFAULT_SCHEMA_CLASS`，访问 `/api/docs/` 查看 Swagger UI，`/api/redoc/` 查看 ReDoc。Serializer 和 ViewSet 的信息自动提取，额外描述用 `@extend_schema` 装饰器添加。旧项目用 `drf-yasg`（OpenAPI 2.0/Swagger）。FastAPI 内置文档生成是对比 DRF 的一个优势。

**Q7: 如何优化 DRF 的序列化性能？**
> 核心策略：1) `select_related`/`prefetch_related` 减少 N+1 查询；2) `only()`/`defer()` 只查需要的字段；3) `SerializerMethodField` 中避免额外查询；4) 简单列表用 `values()` + `Serializer`；5) 高并发场景用 `drf-orjson-renderer` 替代默认 JSON 渲染器；6) 大列表分页避免全量序列化；7) 缓存序列化结果（Redis）。

**Q8: DRF 和 FastAPI 在 API 开发上如何选择？**
> DRF 适合：已有 Django 项目（共享模型/认证）、需要 Admin 后台、团队熟悉 Django。FastAPI 适合：新项目纯 API 服务、需要高性能异步、自动文档、类型驱动的开发体验。DRF 优势是 Django 生态完整（ORM/Admin/Auth），FastAPI 优势是性能和现代开发体验。混合方案：Django 做 Admin + 用户管理，FastAPI 做高性能 API 网关。

---

**相关链接：**
- [[Django]]
- [[FastAPI]]
- [[Pydantic与数据验证]]
- [DRF 官方文档](https://www.django-rest-framework.org/)

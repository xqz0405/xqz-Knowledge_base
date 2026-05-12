---
tags:
  - Python
  - Django
  - Web框架
  - MTV
  - ORM
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Django

## What — 是什么

> Django 是 Python 最成熟的全栈 Web 框架，遵循 MTV（Model-Template-View）架构，提供 ORM、Admin 后台、认证系统、路由、模板引擎等开箱即用的组件。设计哲学是"batteries included"——减少重复工作，快速交付。

**核心概念：**

- **MTV 架构**：Model（数据层）→ Template（展示层）→ View（逻辑层），类似 MVC
- **ORM**：对象关系映射，用 Python 类操作数据库，无需手写 SQL
- **Admin**：自动生成的后台管理界面，一行代码即可拥有 CRUD
- **Middleware**：请求/响应的钩子层，用于认证、日志、CORS 等

**关键特性：**

- 自带 ORM、Admin、Auth、CSRF 保护、Session、缓存、日志、国际化
- URL 路由基于正则或路径转换器，支持命名路由和反向解析
- 模板引擎支持继承、包含、自定义标签和过滤器
- 迁移系统自动追踪模型变更，生成 SQL 迁移脚本
- Django REST Framework 生态完善，API 开发体验一流

**Django 项目结构：**

```
myproject/
├── manage.py            # 命令行工具
├── myproject/
│   ├── __init__.py
│   ├── settings.py      # 配置
│   ├── urls.py          # 根路由
│   ├── asgi.py / wsgi.py
│   └── apps/
│       ├── blog/
│       │   ├── models.py    # 数据模型
│       │   ├── views.py     # 视图逻辑
│       │   ├── urls.py      # 子路由
│       │   ├── admin.py     # Admin 注册
│       │   ├── forms.py     # 表单
│       │   ├── serializers.py  # DRF 序列化器
│       │   └── migrations/  # 迁移文件
│       └── users/
```

**运行机制：**

- **请求生命周期**：HTTP 请求 → Middleware → URL Router → View → Model/DB → Template/JSON → Middleware → 响应
- **ORM 执行**：QuerySet 是惰性的（构建时不查询），迭代/slice/get 时触发 SQL
- **迁移系统**：`makemigrations` 对比模型差异生成迁移文件，`migrate` 执行 SQL
- **信号系统**：`post_save`/`pre_delete` 等信号实现解耦的事件处理

## Why — 为什么

**适用场景：**

- 内容管理系统（CMS）、电商平台、后台管理系统
- 需要快速交付的 CRUD 密集型应用
- 团队协作的大型项目（约定优于配置）
- 需要 Admin 后台的运营系统

**Web 框架对比：**

| 维度 | Django | Flask | FastAPI |
|------|--------|-------|---------|
| 定位 | 全栈框架 | 微框架 | API 框架 |
| ORM | 内置 | 需 SQLAlchemy | 需 SQLAlchemy |
| Admin | 内置 | 无 | 无 |
| 异步 | 3.1+ 支持 | 需扩展 | 原生 |
| API 开发 | DRF | Flask-RESTful | 内置 |
| 学习曲线 | 中 | 低 | 低 |
| 适合规模 | 中大型 | 小型/API | API 优先 |

**优缺点：**

- ✅ 优点：
  - 开箱即用，Admin/ORM/Auth 开箱即用
  - 文档和社区最成熟，问题搜索容易
  - 安全默认值（CSRF/XSS/SQL 注入防护）
  - 迁移系统可靠
- ❌ 缺点：
  - 整体较重，小项目过度
  - ORM 性能优化需要理解 SQL
  - 异步支持不如 FastAPI 原生
  - 模板引擎不如前端框架灵活

## How — 怎么用

### 快速上手

```bash
# 安装
pip install django

# 创建项目
django-admin startproject myproject
cd myproject

# 创建应用
python manage.py startapp blog

# 数据库迁移
python manage.py makemigrations
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser

# 运行
python manage.py runserver
```

```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)

    class Meta:
        verbose_name_plural = "categories"

    def __str__(self):
        return self.name

class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    tags = models.ManyToManyField('Tag', blank=True)
    published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return self.title

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)

    def __str__(self):
        return self.name
```

### 代码示例1：视图与路由

```python
# blog/views.py
from django.shortcuts import get_object_or_404, render
from django.http import JsonResponse
from django.views.decorators.http import require_GET
from .models import Post, Category

# 函数视图
@require_GET
def post_list(request):
    category_id = request.GET.get('category')
    posts = Post.objects.filter(published=True)
    if category_id:
        posts = posts.filter(category_id=category_id)
    posts = posts.select_related('author', 'category').prefetch_related('tags')
    return render(request, 'blog/post_list.html', {'posts': posts})

@require_GET
def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, published=True)
    return render(request, 'blog/post_detail.html', {'post': post})


# 类视图
from django.views.generic import ListView, DetailView, CreateView

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

    def get_queryset(self):
        return Post.objects.filter(published=True).select_related('author')

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    slug_url_kwarg = 'slug'
    slug_field = 'slug'


# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.PostListView.as_view(), name='post-list'),
    path('<slug:slug>/', views.PostDetailView.as_view(), name='post-detail'),
]

# myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]
```

### 代码示例2：Admin 定制与 ORM 查询

```python
# blog/admin.py
from django.contrib import admin
from .models import Post, Category, Tag

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'category', 'published', 'created_at')
    list_filter = ('published', 'category', 'created_at')
    search_fields = ('title', 'content')
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    list_editable = ('published',)
    raw_id_fields = ('author',)
    filter_horizontal = ('tags',)

    actions = ['publish_posts', 'unpublish_posts']

    @admin.action(description="发布选中的文章")
    def publish_posts(self, request, queryset):
        queryset.update(published=True)

    @admin.action(description="取消发布选中的文章")
    def unpublish_posts(self, request, queryset):
        queryset.update(published=False)

admin.site.register(Category)
admin.site.register(Tag)


# ORM 常用查询
# 基础查询
Post.objects.all()
Post.objects.filter(published=True)
Post.objects.filter(title__contains="Django")
Post.objects.filter(created_at__year=2026)
Post.objects.filter(category__name="技术")

# 关联查询
Post.objects.select_related('author', 'category')  # JOIN（ForeignKey）
Post.objects.prefetch_related('tags')               # 两次查询（ManyToMany）

# 聚合
from django.db.models import Count, Avg, Q
Post.objects.values('category__name').annotate(
    count=Count('id'),
    avg_len=Avg('content__length')
)

# 复杂查询
Post.objects.filter(
    Q(title__icontains="python") | Q(content__icontains="python"),
    published=True
)

# F 表达式（引用数据库字段）
from django.db.models import F
Post.objects.filter(content__length__gt=F('title__length') * 10)
```

### 代码示例3：中间件与信号

```python
# 自定义中间件
import time
import logging

logger = logging.getLogger(__name__)

class TimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        start = time.time()
        response = self.get_response(request)
        elapsed = time.time() - start
        logger.info(f"{request.method} {request.path} - {elapsed:.3f}s")
        response['X-Response-Time'] = f"{elapsed:.3f}s"
        return response

# settings.py 中注册
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'myproject.middleware.TimingMiddleware',  # 自定义
]


# 信号
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """用户创建时自动创建关联的 Profile"""
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    """用户保存时同步保存 Profile"""
    instance.profile.save()
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| N+1 查询 | 循环中访问 ForeignKey | 用 `select_related`/`prefetch_related` |
| 迁移冲突 | 多人修改模型生成冲突迁移 | `merge` 或手动解决 |
| CSRF 验证失败 | POST 请求缺少 CSRF Token | 模板加 `{% csrf_token %}`，API 用 `@csrf_exempt` |
| 静态文件 404 | `DEBUG=False` 时不自动服务 | `collectstatic` + nginx 配置 |
| 时区问题 | `USE_TZ=True` 但数据库时区不一致 | 统一用 UTC，展示时转换 |
| 循环导入 | models 导入 views 等 | 用 `apps.get_model()` 延迟导入或信号解耦 |

### 最佳实践

- 使用 `select_related`/`prefetch_related` 解决 N+1 查询
- 应用拆分：按业务域划分 app，保持单一职责
- 环境配置用 `django-environ`，敏感信息不入代码
- `settings/` 目录拆分：base.py / dev.py / prod.py
- 测试用 `pytest-django`，比 Django 测试运行器更灵活
- 生产部署用 Gunicorn/uWSGI + Nginx 反向代理
- 异步视图（Django 3.1+）用 `async def` 定义，ASGI 部署

## 面试题

**Q1: Django 的 MTV 和 MVC 有什么区别？**
> Django 的 MTV 是 MVC 的变体：Model 对应 MVC 的 Model（数据层），Template 对应 MVC 的 View（展示层），View 对应 MVC 的 Controller（逻辑层）。Django 框架本身充当了部分 Controller 角色（URL 路由分发请求到 View）。核心区别是命名不同，架构思想一致。

**Q2: Django ORM 的 QuerySet 是惰性的，这意味着什么？**
> QuerySet 在构建时不执行数据库查询，只有在"求值"时才触发 SQL。求值触发条件：迭代、切片（`[:5]`）、`len()`、`list()`、`repr()`、`bool()`。这允许链式调用构建复杂查询，最终只执行一次 SQL。但要注意：重复求值会重复查询，应缓存结果。

**Q3: `select_related` 和 `prefetch_related` 有什么区别？**
> `select_related` 用于 ForeignKey/OneToOne，通过 SQL JOIN 在一次查询中获取关联对象，适合一对一和多对一。`prefetch_related` 用于 ManyToMany/反向 ForeignKey，执行两次查询（主查询 + 关联查询），在 Python 中合并结果，适合多对多和反向关系。前者减少查询次数但可能产生大 JOIN 结果集，后者查询两次但结果集更小。

**Q4: Django 中间件的执行顺序是什么？**
> 中间件按 `MIDDLEWARE` 列表顺序执行请求阶段（从上到下），响应阶段按逆序执行（从下到上）。每个中间件的 `process_request` 按顺序调用，`process_response` 按逆序调用。如果某个中间件的 `process_request` 返回了 Response，后续中间件的请求阶段不再执行，直接进入逆序的响应阶段。

**Q5: Django 的迁移系统是如何工作的？**
> `makemigrations` 对比当前模型定义和上次迁移的状态，生成迁移文件（Python 代码描述变更）。`migrate` 按顺序执行迁移文件中的 `operations`（AddField、CreateModel 等），同时在 `django_migrations` 表中记录已执行的迁移。迁移文件应该纳入版本控制。冲突时用 `makemigrations --merge` 生成合并迁移。

**Q6: Django 的信号（Signals）和覆写 `save()` 方法有什么区别？**
> 信号是松耦合的——发送者不需要知道接收者的存在，多个接收者可以响应同一事件。覆写 `save()` 是紧耦合的——逻辑直接在模型类中。信号适合跨 app 通信（用户创建时发通知），`save()` 适合模型自身逻辑（自动生成 slug）。信号的问题：难追踪（分散在多个文件）、执行顺序不确定、测试困难。优先覆写 `save()`，跨 app 时用信号。

**Q7: 如何优化 Django 的查询性能？**
> 核心策略：1) `select_related`/`prefetch_related` 解决 N+1；2) `only()`/`defer()` 只查需要的字段；3) `values()`/`values_list()` 返回字典/元组而非 ORM 对象；4) 数据库索引（`db_index=True`）；5) `bulk_create`/`bulk_update` 批量操作；6) 查询集缓存（同一 QuerySet 多次使用）；7) 分页（`Paginator`）；8) 缓存框架（Redis）。

**Q8: Django 的异步支持现状如何？**
> Django 3.1+ 支持异步视图（`async def`）、异步中间件和异步 ORM（5.0+ 完整支持）。关键限制：1) 异步视图内不能调用同步 ORM（需 `sync_to_async` 包装）；2) 中间件可以是异步的，但会创建事件循环开销；3) Django 5.0 开始 ORM 的 `acreate`/`afilter` 等异步 API 基本完善；4) 部署用 ASGI（Daphne/Uvicorn）。异步适合 IO 密集场景（调用外部 API），CPU 密集场景仍需多进程。

---

**相关链接：**
- [[Flask]]
- [[Django REST Framework]]
- [[SQLAlchemy与ORM]]
- [Django 官方文档](https://docs.djangoproject.com/)

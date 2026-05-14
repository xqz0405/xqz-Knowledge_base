---
tags:
  - Python
  - Flask
  - Web框架
  - 微框架
  - 蓝图
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Flask

## What — 是什么

> Flask 是 Python 的微框架（Micro Framework），核心仅提供路由、模板和请求处理，其余功能通过扩展按需添加。设计哲学是"微"不代表功能少，而是保持核心简洁，让开发者自由选择组件。适合 API 开发、微服务和小型应用。

**核心概念：**

- **Werkzeug**：WSGI 工具库，处理 HTTP 请求/响应、路由、调试
- **Jinja2**：模板引擎，模板继承、宏、过滤器
- **蓝图（Blueprint）**：模块化组织路由和视图，类似 Django 的 app
- **扩展（Extension）**：第三方功能包（Flask-SQLAlchemy、Flask-Login 等）

**关键特性：**

- 核心极小（路由 + 请求上下文 + 模板），其他全部可选
- 装饰器定义路由：`@app.route('/api/users')`
- 请求上下文（request/session/g）和应用上下文（current_app）
- 蓝图支持应用模块化和插件式扩展
- 开发服务器自带调试器和热重载

**Flask 请求生命周期：**

```
HTTP 请求
  ↓
Werkzeug WSGI 接收
  ↓
请求上下文推入（request/session/g）
  ↓
URL 路由匹配 → before_request 钩子
  ↓
视图函数执行
  ↓
after_request 钩子 → 构建响应
  ↓
请求上下文弹出
  ↓
HTTP 响应
```

**运行机制：**

- **应用上下文**：`current_app` 指向当前 Flask 实例，`g` 存储请求级临时数据
- **请求上下文**：`request` 包含请求数据，`session` 管理会话
- **上下文栈**：Flask 用 LocalStack 管理多线程/协程下的上下文隔离
- **CLI**：`flask run`/`flask shell`/自定义命令（Click 集成）

## Why — 为什么

**适用场景：**

- RESTful API 和微服务
- 小型 Web 应用和原型快速验证
- 需要精细控制组件选择的场景
- 学习 Web 开发原理

**Flask vs Django 对比：**

| 维度 | Flask | Django |
|------|-------|--------|
| 核心大小 | 极小（路由+模板） | 全栈（ORM+Admin+Auth+...） |
| 灵活性 | 极高（自选组件） | 中（内置约定） |
| 学习曲线 | 低 | 中 |
| 项目规模 | 小中型 | 中大型 |
| ORM | 需 SQLAlchemy | 内置 |
| Admin | 无 | 内置 |
| 适合场景 | API/微服务 | CMS/管理后台 |

**优缺点：**

- ✅ 优点：
  - 核心简洁，源码可读，便于理解 Web 原理
  - 灵活选型，不被框架锁定
  - 学习曲线低，快速上手
  - API 开发轻量高效
- ❌ 缺点：
  - 需要自己组装生态（ORM/Auth/Migration 各选一个）
  - 大项目缺乏统一约定，团队风格可能不一致
  - 没有内置 Admin，后台开发需额外工作
  - 异步支持不如 FastAPI

## How — 怎么用

### 快速上手

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, Flask!'

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = {"id": user_id, "name": "Alice"}
    return jsonify(user)

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    return jsonify(data), 201

if __name__ == '__main__':
    app.run(debug=True)
```

### 代码示例1：蓝图模块化与工厂模式

```python
# app/__init__.py — 应用工厂
from flask import Flask
from .config import Config

def create_app(config_name='default'):
    app = Flask(__name__)
    app.config.from_object(Config)

    # 注册蓝图
    from .api import api_bp
    from .auth import auth_bp
    app.register_blueprint(api_bp, url_prefix='/api')
    app.register_blueprint(auth_bp, url_prefix='/auth')

    # 扩展初始化
    from .extensions import db, migrate, login_manager
    db.init_app(app)
    migrate.init_app(app, db)
    login_manager.init_app(app)

    return app


# app/api/__init__.py — API 蓝图
from flask import Blueprint

api_bp = Blueprint('api', __name__)

from . import users, posts  # 导入路由


# app/api/users.py
from flask import jsonify, request
from . import api_bp
from ..models import User

@api_bp.route('/users', methods=['GET'])
def list_users():
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 20, type=int)
    pagination = User.query.paginate(page=page, per_page=per_page)
    return jsonify({
        'users': [u.to_dict() for u in pagination.items],
        'total': pagination.total,
        'page': page,
    })

@api_bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    user = User.query.get_or_404(id)
    return jsonify(user.to_dict())


# app/auth/__init__.py — 认证蓝图
from flask import Blueprint

auth_bp = Blueprint('auth', __name__)

from . import routes


# app/auth/routes.py
from flask import request, jsonify
from . import auth_bp

@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    # 验证逻辑...
    return jsonify({"token": "jwt-token-here"})

@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    # 注册逻辑...
    return jsonify({"message": "注册成功"}), 201
```

### 代码示例2：扩展集成（SQLAlchemy + Login）

```python
# app/extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager

db = SQLAlchemy()
migrate = Migrate()
login_manager = LoginManager()
login_manager.login_view = 'auth.login'


# app/models.py
from .extensions import db, login_manager
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

    def to_dict(self):
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
        }

@login_manager.user_loader
def load_user(id):
    return User.query.get(int(id))


class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    author = db.relationship('User', backref='posts')
    created_at = db.Column(db.DateTime, server_default=db.func.now())
```

### 代码示例3：错误处理与钩子

```python
from flask import jsonify, g
import time

# 错误处理
@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "资源不存在"}), 404

@app.errorhandler(500)
def internal_error(e):
    return jsonify({"error": "服务器内部错误"}), 500

# 全局异常处理
@app.errorhandler(Exception)
def handle_exception(e):
    response = jsonify({
        "error": e.__class__.__name__,
        "message": str(e)
    })
    response.status_code = 500
    return response

# 请求钩子
@app.before_request
def before_request():
    g.start_time = time.time()

@app.after_request
def after_request(response):
    if hasattr(g, 'start_time'):
        elapsed = time.time() - g.start_time
        response.headers['X-Response-Time'] = f"{elapsed:.3f}s"
    return response

@app.teardown_appcontext
def teardown_db(exception):
    """请求结束时清理数据库连接"""
    db = g.pop('db', None)
    if db is not None:
        db.close()

# 模板渲染
@app.route('/blog/<slug>')
def blog_post(slug):
    post = Post.query.filter_by(slug=slug).first_or_404()
    return render_template('blog/post.html', post=post)

# Jinja2 模板继承
# templates/base.html:
# <html><body>{% block content %}{% endblock %}</body></html>
# templates/blog/post.html:
# {% extends "base.html" %}
# {% block content %}<h1>{{ post.title }}</h1>{{ post.content }}{% endblock %}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 循环导入 | 视图导入模型，模型导入 db | 用工厂模式，扩展单独模块 |
| 上下文外使用 request | 非请求线程中 request 为空 | 用 `test_request_context` 或 `app_context` |
| 大文件上传 OOM | 请求体全部加载到内存 | 用 `stream_with_context` 流式处理 |
| 多进程 session 丢失 | 默认 session 在内存中 | 用 Flask-Session（Redis/数据库） |
| 蓝图模板找不到 | 模板路径未配置 | 蓝图 `template_folder` 参数 |

### 最佳实践

- 使用应用工厂模式（`create_app()`）避免循环导入
- 蓝图按功能域拆分（auth/api/admin）
- 配置用类继承（Development/Testing/Production）
- 扩展在工厂中 `init_app`，不在模块级创建
- API 项目用 Flask-RESTful/Flask-RESTX，或直接用 FastAPI
- 生产用 Gunicorn + Nginx，不用 Flask 开发服务器

## 面试题

**Q1: Flask 的应用上下文和请求上下文有什么区别？**
> 应用上下文（`current_app`/`g`）跟踪应用级别的数据，`current_app` 指向当前 Flask 实例，`g` 存储请求级临时数据。请求上下文（`request`/`session`）跟踪请求级别的数据。请求上下文在每次请求时推入，应用上下文在请求上下文之前推入。`g` 在请求结束时清除，`current_app` 在应用上下文弹出时失效。

**Q2: 什么是蓝图？和 Django 的 app 有什么区别？**
> 蓝图是 Flask 的模块化机制，将路由、模板、静态文件组织成可复用的组件。注册到 app 时指定 URL 前缀。和 Django app 的区别：1) 蓝图更轻量，没有 models/admin/migrations 的约定；2) 蓝图可以多次注册到同一 app（不同前缀）；3) 蓝图没有独立的配置系统。Django app 是更强的约定（包含模型/迁移/管理），Flask 蓝图是更灵活的组织方式。

**Q3: Flask 的 `before_request` 和 `after_request` 有什么区别？**
> `before_request` 在视图函数之前执行，用于前置处理（认证、参数解析）。如果返回了 Response，则跳过视图直接返回。`after_request` 在视图函数之后执行，接收视图返回的 Response，可以修改响应头、记录日志等，必须返回 Response 对象。`teardown_request` 在请求结束后执行（即使有异常），用于资源清理，不接收 Response。

**Q4: Flask 如何处理并发请求？**
> Flask 开发服务器默认单线程，一次只处理一个请求。`app.run(threaded=True)` 启用多线程模式。生产环境使用 WSGI 服务器：Gunicorn（多 worker 进程 + 线程）、uWSGI。每个请求在独立线程/进程中处理，Flask 的上下文通过 `LocalStack` 实现线程隔离，`request`/`g` 在不同线程中互不干扰。

**Q5: Flask 的 `url_for` 有什么作用？为什么要用反向路由？**
> `url_for('view_name', **kwargs)` 根据视图函数名和参数生成 URL，而不是硬编码 URL 字符串。好处：1) 修改路由规则时无需修改模板中的链接；2) 自动处理 URL 编码；3) 支持蓝图命名空间（`url_for('api.list_users')`）；4) 静态文件版本控制（`url_for('static', filename='app.js')`）。

**Q6: Flask 如何实现 WebSocket？**
> Flask 本身不支持 WebSocket，需要扩展：1) Flask-SocketIO（基于 Socket.IO 协议，支持回退到长轮询）；2) 配合 Gunicorn + eventlet/gevent worker。使用方式：`@socketio.on('message')` 处理事件，`emit()` 发送消息，支持房间（`join_room`/`leave_room`）。纯 WebSocket 场景推荐 FastAPI + websockets，更轻量原生。

**Q7: 工厂模式解决了什么问题？**
> 工厂模式（`create_app()`）解决：1) 循环导入——扩展在工厂中初始化而非模块级；2) 多实例——测试和迁移时可以创建不同配置的 app 实例；3) 配置灵活性——根据环境选择不同配置。没有工厂模式时，模块级 `app = Flask(__name__)` 会在导入时立即创建，导致扩展和配置无法延迟绑定。

**Q8: Flask 的 `g` 对象和 `session` 有什么区别？**
> `g` 是请求级别的临时存储，在 `before_request` 中设置，视图和 `after_request` 中访问，请求结束后清除。`session` 是跨请求的会话存储，基于签名 Cookie（默认），可以跨请求持久化数据（如用户登录状态）。`g` 用于当前请求的中间数据（数据库连接、当前用户），`session` 用于用户会话数据（登录态、购物车）。

---

**相关链接：**
- [[Django]]
- [[FastAPI]]
- [[Web中间件与请求处理]]
- [Flask 官方文档](https://flask.palletsprojects.com/)

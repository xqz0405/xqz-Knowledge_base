---
tags:
  - Python
  - Web框架
  - FastAPI
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# FastAPI

## What — 是什么

> FastAPI 是基于 Starlette 和 Pydantic 的现代 Python Web 框架，以高性能、自动生成 API 文档和类型安全著称。

**核心概念：**

- **Path Operation**：`@app.get("/items/{id}")` 路由装饰器
- **Pydantic Model**：请求/响应的数据验证和序列化
- **Dependency Injection**：`Depends()` 声明式依赖注入
- **ASGI**：异步服务器网关接口，支持 WebSocket 和长连接

**核心架构：**

- 设计理念：类型驱动开发，自动推导文档和验证
- 核心模块：路由系统、Pydantic 验证、依赖注入、OpenAPI 生成
- 数据流：Request → 依赖注入 → 参数解析/验证 → Handler → 响应序列化

**插件生态：**

- 官方插件：无（核心足够完整）
- 社区热门插件：fastapi-users（用户认证）、sqlalchemy（ORM）、alembic（迁移）

## Why — 为什么

**适用场景：**

- RESTful API 服务
- 微服务后端
- 机器学习模型服务
- 需要自动文档的 API 项目

**对比同类框架：**

| 维度 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| 性能 | 极高（Uvicorn + ASGI） | 中（WSGI） | 中（WSGI） |
| 生态 | 快速增长 | 成熟 | 最成熟 |
| 学习曲线 | 低（类型提示即文档） | 低 | 中 |
| 灵活性 | 高 | 高 | 中（约定明确） |

**优缺点：**

- ✅ 优点：
  - 性能接近 Go/Node.js（基于 Starlette）
  - 自动生成 Swagger/ReDoc 文档
  - Pydantic 验证 + 类型提示，开发体验极佳
  - 原生 async/await 支持
- ❌ 缺点：
  - 生态不如 Flask/Django 成熟
  - 小型项目引入 Pydantic 可能过重
  - 需要理解 ASGI 与 WSGI 的区别

## How — 怎么用

### 快速上手

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    description: str | None = None

@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    return item

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}
```

```bash
uvicorn main:app --reload
# 访问 http://localhost:8000/docs 查看 Swagger 文档
```

### 代码示例

**依赖注入：**

```python
from fastapi import Depends, HTTPException

async def get_current_user(token: str = Header(...)):
    user = verify_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.get("/me")
async def read_me(user: User = Depends(get_current_user)):
    return user
```

**数据库集成（SQLAlchemy async）：**

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker

engine = create_async_engine("postgresql+asyncpg://...")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=404)
    return user
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 同步数据库驱动阻塞 | 使用同步 SQLAlchemy/pymysql | 切换到异步驱动 asyncpg/aiomysql |
| Pydantic v1/v2 不兼容 | FastAPI 版本与 Pydantic 版本绑定 | FastAPI 0.100+ 支持 Pydantic v2 |
| CORS 问题 | 浏览器跨域限制 | 添加 `CORSMiddleware` |
| 依赖注入循环 | 依赖 A 依赖 B，B 又依赖 A | 重构依赖关系，或用 `lru_cache` 缓存实例 |

### 最佳实践

- 使用 `response_model` 控制响应字段和文档
- 业务逻辑抽到 service 层，Handler 只做参数处理
- 使用 `APIRouter` 按模块拆分路由
- 生产环境用 Gunicorn + Uvicorn workers

## 面试题

**Q1: FastAPI 如何支持异步？同步路由和异步路由有什么区别？**
> FastAPI 基于 ASGI（Starlette），路由函数可用 `async def` 定义，在事件循环中协程式执行；也可用普通 `def` 定义，FastAPI 会自动将其放入线程池执行以避免阻塞事件循环。异步路由适合 I/O 密集操作，同步路由适合调用阻塞库。

**Q2: FastAPI 的依赖注入是如何工作的？**
> 通过 `Depends()` 声明依赖，FastAPI 在请求到达时自动解析并注入。依赖可以是函数或类，支持嵌套依赖（依赖中再声明依赖），每次请求默认重新创建实例。类似 pytest 的 fixture 机制，实现了解耦和复用。

**Q3: Pydantic 在 FastAPI 中起什么作用？请求校验的流程是什么？**
> Pydantic 负责请求体的反序列化和类型校验、响应体的序列化和字段过滤。流程：请求到达 → FastAPI 按参数类型注解解析路径/查询/请求体 → Pydantic Model 校验数据类型和约束 → 校验失败返回 422 + 详细错误信息 → 校验通过传入路由函数。

**Q4: FastAPI 和 Flask 的核心区别是什么？**
> FastAPI 基于 ASGI 支持原生异步，Flask 基于 WSGI 仅同步；FastAPI 自动生成 OpenAPI 文档，Flask 需手动或借助扩展；FastAPI 内置 Pydantic 类型校验，Flask 需手动校验；FastAPI 原生支持依赖注入，Flask 无此机制。Flask 生态更成熟，FastAPI 开发体验更现代。

**Q5: FastAPI 中如何实现中间件？**
> 使用 `@app.middleware("http")` 装饰器定义，在请求到达路由前和响应返回后执行自定义逻辑（如日志、CORS、认证）。内部机制基于 Starlette 中间件，调用 `await call_next(request)` 将请求传递给下一层处理。

**Q6: FastAPI 的 `APIRouter` 有什么作用？如何组织大型项目？**
> `APIRouter` 是路由分组工具，类似 Flask 的蓝图。它将相关路由组织到独立模块中，设置公共前缀（`prefix="/api/v1"`）、标签（`tags=["users"]`）和依赖（`dependencies=[Depends(verify_token)]`）。大型项目按业务域拆分 Router：`users.router`、`orders.router`，在主 app 中 `app.include_router()` 注册。每个 Router 可以有独立的依赖、中间件和响应模型。

**Q7: FastAPI 如何处理文件上传？**
> 使用 `UploadFile` 类型注解：`file: UploadFile = File(...)`。`UploadFile` 是 SpooledTemporaryFile 的封装，大文件自动写入磁盘而非全部加载到内存。属性：`file.filename`（原始文件名）、`file.content_type`（MIME 类型）、`file.file`（文件对象）。多文件用 `files: list[UploadFile]`。读取内容：`contents = await file.read()` 或 `shutil.copyfileobj(file.file, dest)`。

**Q8: FastAPI 如何实现后台任务？**
> 两种方式：1) `BackgroundTasks`——在响应返回后执行轻量任务，`background_tasks.add_task(func, *args)`，适合发送邮件/日志等简单操作；2) Celery/ARQ——独立的工作队列，支持重试/定时/监控，适合耗时任务（数据处理/报表生成）。BackgroundTasks 是进程内的，随服务重启丢失；Celery 是持久化的，任务不丢失。轻量用前者，重量用后者。

---

**相关链接：**
- [[异步编程]]
- [[Pydantic与数据验证]]
- [[Web中间件与请求处理]]
- [[Django]]
- [FastAPI 官方文档](https://fastapi.tiangolo.com/)

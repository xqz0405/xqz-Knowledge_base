---
tags:
  - Python
  - Pydantic
  - 数据验证
  - 序列化
  - FastAPI
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Pydantic与数据验证

## What — 是什么

> Pydantic 是 Python 最流行的数据验证库，基于 Python 类型注解自动进行数据验证、序列化和文档生成。它是 FastAPI 的核心依赖，也可以独立用于任何需要数据校验的场景。Pydantic V2（2023）用 Rust 重写核心，性能提升 5-50 倍。

**核心概念：**

- **BaseModel**：数据模型基类，自动根据类型注解生成验证器
- **验证器（Validator）**：自定义字段验证逻辑（`@field_validator`/`@model_validator`）
- **序列化**：模型 ↔ dict/JSON 的双向转换，支持别名和字段过滤
- **Settings**：环境变量和配置管理（pydantic-settings）

**关键特性：**

- 基于类型注解自动验证，无需手写验证代码
- V2 用 Rust 核心解析验证，性能比 V1 提升 5-50x
- 支持嵌套模型、自定义类型、Union 判别
- 自动生成 JSON Schema，可用于 OpenAPI 文档
- 丰富的字段约束（`gt`/`lt`/`min_length`/`pattern` 等）
- `model_validate` 和 `model_dump` 统一验证和序列化接口

**Pydantic V1 vs V2 关键变化：**

| V1 | V2 | 说明 |
|----|----|------|
| `parse_obj()` | `model_validate()` | 验证入口 |
| `dict()` | `model_dump()` | 序列化 |
| `json()` | `model_dump_json()` | JSON 序列化 |
| `@validator` | `@field_validator` | 字段验证器 |
| `@root_validator` | `@model_validator` | 模型验证器 |
| `class Config` | `model_config` | 模型配置 |
| `.schema()` | `.model_json_schema()` | JSON Schema |

**运行机制：**

- **核心解析**：Rust 实现的 `pydantic-core` 执行类型检查和验证
- **验证流程**：输入数据 → 类型推断 → 约束检查 → 强制转换 → 自定义验证器 → 模型实例
- **序列化流程**：模型实例 → 字段筛选 → 类型转换 → 别名映射 → dict/JSON
- **Settings 解析**：优先级 `init_kwargs > env_vars > dotenv > defaults`

## Why — 为什么

**适用场景：**

- API 请求/响应数据验证（FastAPI 核心）
- 配置管理（环境变量 → 类型安全对象）
- 数据清洗和转换（CSV/JSON → 结构化模型）
- 数据库 ORM 映射（SQLModel = SQLAlchemy + Pydantic）
- 消息队列数据契约

**数据验证方案对比：**

| 维度 | Pydantic | marshmallow | dataclasses |
|------|----------|-------------|-------------|
| 验证 | 自动（基于类型） | 手动定义 | 无 |
| 性能 | 极快（Rust 核心） | 中 | 无验证开销 |
| JSON Schema | 自动生成 | 需配置 | 无 |
| 序列化 | 内置 | 内置 | 需手动 |
| 反序列化 | 内置+验证 | 内置 | 需手动 |
| 异步支持 | V2 有限 | 无 | N/A |

**优缺点：**

- ✅ 优点：
  - 类型注解即验证规则，零额外代码
  - Rust 核心性能极佳
  - 自动 JSON Schema 生成
  - 与 FastAPI 深度集成
  - 错误信息清晰，定位准确
- ❌ 缺点：
  - 学习曲线（V1→V2 迁移）
  - 复杂验证逻辑可能晦涩
  - 运行时验证有性能开销（比无验证的 dataclass 慢）
  - 某些动态类型场景难以覆盖

## How — 怎么用

### 快速上手

```python
from pydantic import BaseModel, EmailStr, Field, field_validator
from datetime import datetime
from typing import Optional

class UserCreate(BaseModel):
    username: str = Field(min_length=3, max_length=50, pattern=r'^[a-zA-Z0-9_]+$')
    email: EmailStr
    age: int = Field(ge=0, le=150)
    password: str = Field(min_length=8, description="用户密码")

    @field_validator('password')
    @classmethod
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('密码必须包含大写字母')
        if not any(c.isdigit() for c in v):
            raise ValueError('密码必须包含数字')
        return v

class UserResponse(BaseModel):
    id: int
    username: str
    email: EmailStr
    created_at: datetime

    model_config = {"from_attributes": True}  # 支持 ORM 对象

# 验证
user = UserCreate(
    username="alice_123",
    email="alice@example.com",
    age=25,
    password="SecurePass1"
)
print(user.model_dump())  # {'username': 'alice_123', 'email': 'alice@example.com', ...}

# 验证失败
from pydantic import ValidationError
try:
    UserCreate(username="a", email="bad", age=-1, password="weak")
except ValidationError as e:
    print(e.error_count())  # 4
    for err in e.errors():
        print(f"  {err['loc']}: {err['msg']}")
```

### 代码示例1：嵌套模型与别名字段

```python
from pydantic import BaseModel, Field, ConfigDict
from typing import Optional

class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(pattern=r'^\d{6}$')

class Company(BaseModel):
    name: str
    address: Address

class Employee(BaseModel):
    model_config = ConfigDict(
        populate_by_name=True,    # 允许字段名填充
        str_strip_whitespace=True, # 自动去空白
    )

    full_name: str = Field(alias="fullName")
    age: int = Field(ge=18)
    department: str
    company: Company
    manager_id: Optional[int] = None

# 使用别名输入
emp = Employee(
    fullName="Alice Wang",
    age=28,
    department="Engineering",
    company={
        "name": "TechCorp",
        "address": {"street": "123 Main St", "city": "Beijing", "zip_code": "100000"}
    }
)

# 序列化控制
print(emp.model_dump())                # 使用 Python 字段名
print(emp.model_dump(by_alias=True))   # 使用别名
print(emp.model_dump_json(by_alias=True))  # JSON + 别名

# 从 ORM 对象创建
class ORMUser:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email

class UserSchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    name: str
    email: str

orm_obj = ORMUser(1, "Alice", "alice@example.com")
user = UserSchema.model_validate(orm_obj)
```

### 代码示例2：自定义验证器与模型验证

```python
from pydantic import BaseModel, field_validator, model_validator
from datetime import date, datetime
from typing import Optional, Literal

class Booking(BaseModel):
    hotel_name: str = Field(min_length=1)
    check_in: date
    check_out: date
    room_type: Literal['standard', 'deluxe', 'suite']
    guests: int = Field(ge=1, le=4)
    special_requests: Optional[str] = None

    @field_validator('check_in')
    @classmethod
    def check_in_future(cls, v):
        if v < date.today():
            raise ValueError('入住日期不能是过去')
        return v

    @model_validator(mode='after')
    def check_dates(self):
        if self.check_out <= self.check_in:
            raise ValueError('退房日期必须晚于入住日期')
        if (self.check_out - self.check_in).days > 30:
            raise ValueError('最长入住30天')
        return self

    @model_validator(mode='after')
    def check_guests_for_suite(self):
        if self.room_type == 'suite' and self.guests < 2:
            raise ValueError('套房至少2人入住')
        return self

# 正常
booking = Booking(
    hotel_name="Grand Hotel",
    check_in="2026-06-01",
    check_out="2026-06-05",
    room_type="deluxe",
    guests=2
)

# 验证失败示例
try:
    Booking(
        hotel_name="Test",
        check_in="2026-06-10",
        check_out="2026-06-05",  # 退房早于入住
        room_type="suite",
        guests=1  # 套房1人
    )
except ValidationError as e:
    for err in e.errors():
        print(f"{err['type']}: {err['msg']}")


# 判别联合（Discriminated Union）
from typing import Annotated, Union

class EmailNotification(BaseModel):
    type: Literal['email'] = 'email'
    address: EmailStr

class SMSNotification(BaseModel):
    type: Literal['sms'] = 'sms'
    phone: str = Field(pattern=r'^\d{11}$')

Notification = Annotated[
    Union[EmailNotification, SMSNotification],
    Field(discriminator='type')
]

class Alert(BaseModel):
    message: str
    channel: Notification

alert = Alert(message="Server down", channel={"type": "email", "address": "ops@example.com"})
```

### 代码示例3：Settings 配置管理

```python
# pip install pydantic-settings
from pydantic_settings import BaseSettings, SettingsConfigDict

class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file='.env',
        env_file_encoding='utf-8',
        env_prefix='APP_',         # 环境变量前缀
        case_sensitive=False,
    )

    # 应用配置
    app_name: str = "MyApp"
    debug: bool = False
    host: str = "0.0.0.0"
    port: int = 8000

    # 数据库
    db_url: str = "postgresql://localhost/mydb"
    db_pool_size: int = 10

    # Redis
    redis_url: str = "redis://localhost:6379/0"

    # 安全
    secret_key: str = Field(..., min_length=32)  # 必填
    allowed_hosts: list[str] = ["localhost"]

    # 日志
    log_level: str = "INFO"

# .env 文件
# APP_DEBUG=true
# APP_SECRET_KEY=your-super-secret-key-with-32-chars!!
# APP_DB_URL=postgresql://user:pass@db:5432/prod

settings = AppSettings()  # 自动读取环境变量和 .env
print(f"Running {settings.app_name} on {settings.host}:{settings.port}")

# 不同环境
# settings_dev = AppSettings(_env_file='.env.dev')
# settings_prod = AppSettings(_env_file='.env.prod')
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| V1 代码不兼容 | V2 API 变更大 | 用 `pydantic.v1` 兼容模式逐步迁移 |
| Optional 字段不传报错 | V2 中 `Optional[X]` 仍需默认值 | `field: Optional[X] = None` |
| ORM 模式不生效 | 未配置 `from_attributes` | `model_config = ConfigDict(from_attributes=True)` |
| 别名字段序列化不一致 | `model_dump()` 默认用字段名 | 用 `by_alias=True` 输出别名 |
| 嵌套模型验证顺序 | 先字段验证后模型验证 | `@model_validator(mode='after')` 在所有字段验证后执行 |
| 循环引用 | 模型互相引用 | 用 `model_rebuild()` 或 `ForwardRef` |

### 最佳实践

- 请求和响应用不同模型（请求含验证，响应可选字段）
- `Field()` 添加约束和描述（自动生成 JSON Schema 和文档）
- 用 `ConfigDict` 统一配置（ORM 模式、别名、严格模式）
- Settings 管理环境变量，敏感信息从 `.env` 读取
- 复用模型用继承：`class UserUpdate(UserBase): ...`
- `model_validator(mode='after')` 做跨字段验证
- 错误处理：捕获 `ValidationError`，返回结构化错误信息

## 面试题

**Q1: Pydantic V2 和 V1 有什么核心区别？**
> V2 用 Rust 重写了核心（`pydantic-core`），性能提升 5-50 倍。API 变更：`parse_obj` → `model_validate`、`dict` → `model_dump`、`@validator` → `@field_validator`、`class Config` → `model_config`。新增 `model_validator(mode='before'/'after')` 替代 `@root_validator`，`SerializeAsAny` 控制序列化行为。V2 更严格（如 `Optional[X]` 仍需默认值），但提供了 `pydantic.v1` 兼容模块。

**Q2: `@field_validator` 和 `@model_validator` 有什么区别？**
> `@field_validator` 验证单个字段，在字段类型转换后执行，可以指定 `mode='before'`（转换前）或 `mode='after'`（转换后）。`@model_validator` 验证整个模型，在所有字段验证后执行（`mode='after'`）或之前（`mode='before'`），适合跨字段验证（如 `check_out > check_in`）。字段级用 `field_validator`，跨字段用 `model_validator`。

**Q3: Pydantic 如何与 ORM 配合？**
> 通过 `model_config = ConfigDict(from_attributes=True)` 启用 ORM 模式，Pydantic 可以直接从 ORM 对象（如 SQLAlchemy model）读取属性，无需手动转 dict。`UserSchema.model_validate(orm_obj)` 自动提取属性。反向（Pydantic → ORM）需手动用 `model_dump()` 传入 ORM 构造函数。SQLModel 把两者合一（同时是 Pydantic model 和 SQLAlchemy model）。

**Q4: Pydantic 的 `model_dump` 和 `model_dump_json` 有什么区别？**
> `model_dump()` 返回 Python dict，可以进一步处理。`model_dump_json()` 返回 JSON 字符串，性能更好（直接 Rust 层序列化，不经过 dict 中间步骤）。两者都支持 `include`/`exclude` 过滤字段、`by_alias` 使用别名、`exclude_none`/`exclude_defaults` 控制输出。需要 JSON 字符串时优先 `model_dump_json()`，需要 Python 对象时用 `model_dump()`。

**Q5: Pydantic Settings 的优先级是怎样的？**
> 默认优先级从高到低：1) 构造函数显式传入的值；2) 环境变量；3) `.env` 文件；4) 模型默认值。可通过 `SettingsConfigDict` 调整。环境变量名规则：`env_prefix` + 字段名大写（如 `APP_DB_URL`）。`case_sensitive=False` 时大小写不敏感。列表/JSON 类型的环境变量用 JSON 格式（`APP_ALLOWED_HOSTS='["a.com","b.com"]'`）。

**Q6: 如何实现 Pydantic 模型的部分更新（PATCH）？**
> 方案1：`model.copy(update=data)`（V1）。V2 用 `model.model_copy(update=data)`。方案2：定义 Update 模型，所有字段 `Optional`，只传需要更新的字段。方案3：`exclude_unset=True`，`model_dump(exclude_unset=True)` 只输出用户显式设置的字段，不包含默认值。方案3 最常用：`UserUpdate(**data).model_dump(exclude_unset=True)` 得到只含更新字段的 dict。

**Q7: Pydantic 的 JSON Schema 生成有什么用途？**
> JSON Schema 是数据结构的机器可读描述，用途：1) FastAPI 自动生成 OpenAPI 文档（Swagger UI / ReDoc）；2) 前端表单自动生成（根据 Schema 生成表单控件）；3) 运行时验证（其他语言/系统用 Schema 验证数据）；4) 数据契约（微服务间共享 Schema）。Pydantic V2 的 `model_json_schema()` 支持 JSON Schema draft 2020-12。

**Q8: Pydantic 和 dataclass 都能做数据建模，如何选择？**
> dataclass 是标准库，轻量快速，无验证，适合纯内存数据结构。Pydantic 有验证、序列化、JSON Schema、ORM 集成，适合 API 边界（请求/响应/配置）。规则：内部数据传递用 dataclass（无验证开销），外部数据入口（API/配置/文件）用 Pydantic。SQLModel 是两者的统一方案（同时继承 BaseModel 和 SQLAlchemy 声明式基类）。

---

**相关链接：**
- [[FastAPI]]
- [[类型系统与类型提示]]
- [[Django REST Framework]]
- [Pydantic V2 文档](https://docs.pydantic.dev/latest/)

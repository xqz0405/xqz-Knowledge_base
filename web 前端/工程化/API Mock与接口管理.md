---
tags:
  - Web前端
  - 工程化
  - Mock
  - 接口管理
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# API Mock与接口管理

## What — 是什么

> Mock 是模拟后端接口数据的技术手段，接口管理是 API 文档与协作平台，两者是前后端分离开发的核心基础设施。

**核心概念：**

- **Mock**：在前端开发阶段模拟后端接口返回数据，使前端不依赖后端进度独立开发
- **接口管理**：集中维护 API 文档、请求/响应规范、版本变更，作为前后端协作的契约
- **前后端分离**：基于 API 契约并行开发，Mock 消除等待后端的时间开销

**关键特性：**

- Mock.js 拦截 XHR 请求，返回随机生成的模板数据
- MSW 在 Service Worker 层拦截，浏览器和网络层均生效
- json-server 将 JSON 文件映射为完整 REST API
- Apifox/YApi 集接口文档、Mock 服务、代码生成于一体

## Why — 为什么

**适用场景：**

- 前后端并行开发，后端接口未就绪时前端不阻塞
- 接口文档与 Mock 数据一体化，减少沟通成本
- 单元测试/集成测试中隔离外部 API 依赖
- 接口变更同步通知，减少联调返工

**对比替代方案：**

| 维度 | Mock.js | MSW | json-server | Apifox |
|------|---------|-----|-------------|--------|
| 类型 | 数据生成+XHR拦截 | Service Worker拦截 | 本地REST服务器 | 接口管理平台 |
| 拦截层 | XHR层 | Service Worker层 | 独立HTTP服务 | 云端Mock服务 |
| 功能 | 数据模板+随机数据+拦截 | REST/GraphQL拦截 | 完整CRUD+关联 | 文档+Mock+代码生成 |
| 适用场景 | 快速原型、简单Mock | 测试+开发双模式 | 本地开发联调 | 团队协作+全流程 |
| 维护成本 | 低 | 中 | 低 | 中（需团队规范） |
| 数据真实性 | 较低（随机） | 高（可录制） | 中（手写JSON） | 高（基于文档） |
| 测试支持 | 弱 | 强（Node模式） | 中 | 弱 |

**优缺点：**

- ✅ Mock 优点：
  - 前后端并行开发，提升效率
  - 测试中隔离外部依赖，保证稳定性
  - 可模拟异常场景（超时、错误码、边界值）
- ❌ Mock 缺点：
  - Mock 数据与真实接口可能不一致
  - 增加维护成本，接口变更需同步更新
  - 可能掩盖真实环境下的集成问题

## How — 怎么用

### Mock.js 基础用法

**Mock.mock 模板与 Random 方法：**

```javascript
import Mock from 'mockjs';

// 基本模板语法
const data = Mock.mock({
    'id|+1': 1,             // 自增ID，从1开始
    'name': '@cname',        // 随机中文名
    'age|18-60': 1,          // 18-60之间随机整数
    'email': '@email',       // 随机邮箱
    'avatar': '@image("200x200")', // 随机图片
    'date': '@datetime("yyyy-MM-dd HH:mm:ss")', // 随机日期
    'boolean|1': true,       // 随机布尔值
    'status|1': ['active', 'inactive', 'pending'], // 随机选一个
    'score|1-100.1-2': 1,    // 1-100随机数，保留1-2位小数
    'list|3-5': [{           // 重复3-5次
        'id|+1': 1,
        'title': '@ctitle(5, 15)',
    }],
});

// Random 方法
const Random = Mock.Random;
Random.cname();            // 随机中文名
Random.cparagraph();       // 随机中文段落
Random.city();             // 随机城市
Random.url();              // 随机URL
Random.id();               // 随机身份证号
Random.county(true);       // 随机省市区

// 正则匹配生成数据
const tel = Mock.mock(/\d{11}/);  // 随机11位手机号

// 数据占位符
const user = Mock.mock({
    'id': '@guid',           // GUID
    'name': '@cname',
    'address': '@county(true)', // 省市区
    'phone': /^1[3-9]\d{9}$/,  // 正则手机号
    'createTime': '@now',    // 当前时间
});
```

**Mock.js 拦截 XHR：**

```javascript
import Mock from 'mockjs';

// 拦截 GET 请求
Mock.mock('/api/users', 'get', {
    'code': 200,
    'message': 'success',
    'data|10-20': [{
        'id|+1': 1,
        'name': '@cname',
        'email': '@email',
        'role|1': ['admin', 'editor', 'viewer'],
    }],
});

// 拦截 POST 请求
Mock.mock('/api/users', 'post', (options) => {
    const body = JSON.parse(options.body);
    return {
        code: 200,
        message: '创建成功',
        data: {
            id: Mock.Random.guid(),
            ...body,
        },
    };
});

// 拦截带参数的 GET 请求（正则匹配）
Mock.mock(/\/api\/users\/\d+/, 'get', (options) => {
    const id = options.url.match(/\/api\/users\/(\d+)/)[1];
    return {
        code: 200,
        data: {
            id,
            name: Mock.Random.cname(),
            email: Mock.Random.email(),
        },
    };
});

// 设置超时（模拟网络延迟）
Mock.setup({
    timeout: '200-600', // 随机200-600ms延迟
});
```

### MSW（Mock Service Worker）完整用法

**安装与初始化：**

```bash
# 安装
npm install msw --save-dev

# 生成 Service Worker 文件
npx msw init public/ --save
```

**Handler 定义：**

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse, graphql } from 'msw';

// REST API 拦截
export const handlers = [
    // GET 请求
    http.get('/api/users', ({ request }) => {
        const url = new URL(request.url);
        const page = Number(url.searchParams.get('page') || 1);
        const pageSize = Number(url.searchParams.get('pageSize') || 10);

        return HttpResponse.json({
            code: 200,
            data: {
                list: Array.from({ length: pageSize }, (_, i) => ({
                    id: (page - 1) * pageSize + i + 1,
                    name: `用户${(page - 1) * pageSize + i + 1}`,
                    email: `user${(page - 1) * pageSize + i + 1}@example.com`,
                })),
                total: 100,
                page,
                pageSize,
            },
        });
    }),

    // POST 请求
    http.post('/api/users', async ({ request }) => {
        const body = await request.json() as { name: string; email: string };
        return HttpResponse.json({
            code: 200,
            message: '创建成功',
            data: { id: Math.random().toString(36).slice(2), ...body },
        }, { status: 201 });
    }),

    // PUT 请求
    http.put('/api/users/:id', async ({ params, request }) => {
        const { id } = params;
        const body = await request.json();
        return HttpResponse.json({
            code: 200,
            data: { id, ...body },
        });
    }),

    // DELETE 请求
    http.delete('/api/users/:id', ({ params }) => {
        return HttpResponse.json({
            code: 200,
            message: `用户 ${params.id} 已删除`,
        });
    }),

    // 模拟错误响应
    http.get('/api/error', () => {
        return HttpResponse.json(
            { code: 500, message: '服务器内部错误' },
            { status: 500 },
        );
    }),

    // 模拟网络延迟
    http.get('/api/slow', async () => {
        await new Promise(resolve => setTimeout(resolve, 3000));
        return HttpResponse.json({ code: 200, data: '延迟3秒响应' });
    }),

    // GraphQL 拦截
    graphql.query('GetUsers', ({ variables }) => {
        const { limit = 10 } = variables;
        return HttpResponse.json({
            data: {
                users: Array.from({ length: limit }, (_, i) => ({
                    id: String(i + 1),
                    name: `用户${i + 1}`,
                })),
            },
        });
    }),

    // GraphQL Mutation
    graphql.mutation('CreateUser', ({ variables }) => {
        return HttpResponse.json({
            data: {
                createUser: {
                    id: Math.random().toString(36).slice(2),
                    ...variables.input,
                },
            },
        });
    }),
];
```

**浏览器模式：**

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);

// src/main.tsx
async function bootstrap() {
    // 开发环境启动 MSW
    if (import.meta.env.DEV) {
        const { worker } = await import('./mocks/browser');
        await worker.start({
            onUnhandledRequest: 'bypass', // 未拦截的请求正常发出
            // onUnhandledRequest: 'warn',  // 未拦截的请求控制台警告
            // onUnhandledRequest: 'error', // 未拦截的请求报错
        });
    }

    const root = ReactDOM.createRoot(document.getElementById('root')!);
    root.render(<App />);
}

bootstrap();
```

**Node 模式（用于测试）：**

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/test/setup.ts
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'bypass' }));
afterEach(() => server.resetHandlers()); // 每个测试后重置
afterAll(() => server.close());

// 测试中临时覆盖 handler
import { http, HttpResponse } from 'msw';
import { server } from './mocks/server';

it('处理登录失败', async () => {
    server.use(
        http.post('/api/login', () => {
            return HttpResponse.json(
                { code: 401, message: '密码错误' },
                { status: 401 },
            );
        }),
    );

    render(<LoginForm />);
    // ... 测试逻辑
});
```

### json-server 搭建本地 REST API

**安装与基础用法：**

```bash
# 安装
npm install json-server --save-dev

# 启动（默认端口3000）
npx json-server db.json --port 3001 --watch
```

**db.json 数据定义：**

```json
{
    "users": [
        { "id": 1, "name": "Alice", "email": "alice@example.com", "departmentId": 1 },
        { "id": 2, "name": "Bob", "email": "bob@example.com", "departmentId": 2 }
    ],
    "departments": [
        { "id": 1, "name": "技术部" },
        { "id": 2, "name": "产品部" }
    ],
    "posts": [
        { "id": 1, "title": "第一篇文章", "authorId": 1, "tags": ["前端", "React"] }
    ]
}
```

**自动生成的 REST 路由：**

```
GET    /users              # 获取所有用户
GET    /users/1            # 获取ID为1的用户
POST   /users              # 创建用户
PUT    /users/1            # 更新用户（全量）
PATCH  /users/1            # 更新用户（部分）
DELETE /users/1            # 删除用户

# 查询参数
GET /users?name=Alice                     # 过滤
GET /users?_sort=name&_order=asc          # 排序
GET /users?_page=1&_limit=10              # 分页
GET /users?q=alice                        # 全文搜索
GET /users?departmentId=1                 # 关联查询
```

**自定义路由（routes.json）：**

```json
{
    "/api/*": "/$1",
    "/:resource/:id/show": "/:resource/:id",
    "/posts/:id/author": "/users?posts/:id"
}
```

```bash
npx json-server db.json --routes routes.json --port 3001
```

**中间件扩展：**

```javascript
// server.js
const jsonServer = require('json-server');
const server = jsonServer.create();
const router = jsonServer.router('db.json');
const middlewares = jsonServer.defaults();

server.use(middlewares);
server.use(jsonServer.bodyParser);

// 自定义中间件：添加时间戳
server.use((req, res, next) => {
    if (req.method === 'POST') {
        req.body.createdAt = new Date().toISOString();
        req.body.updatedAt = new Date().toISOString();
    }
    if (req.method === 'PUT' || req.method === 'PATCH') {
        req.body.updatedAt = new Date().toISOString();
    }
    next();
});

// 自定义路由
server.get('/api/stats', (req, res) => {
    const db = router.db;
    res.json({
        userCount: db.get('users').size().value(),
        postCount: db.get('posts').size().value(),
    });
});

// 模拟延迟
server.use((req, res, next) => {
    setTimeout(next, 300);
});

server.use(router);
server.listen(3001, () => {
    console.log('JSON Server is running on port 3001');
});
```

**关联数据：**

```bash
# 查询用户及其所属部门
GET /users/1?_embed=department

# 查询部门及其下所有用户
GET /departments/1?_include=users
```

### Vite 开发代理与 Mock 集成

**vite-plugin-mock：**

```bash
npm install vite-plugin-mock mockjs --save-dev
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { viteMockServe } from 'vite-plugin-mock';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        react(),
        viteMockServe({
            mockPath: 'src/mocks',      // mock文件目录
            localEnabled: true,          // 开发环境启用
            prodEnabled: false,          // 生产环境禁用
            injectFile: 'src/main.tsx',  // 注入入口文件
            logger: true,
        }),
    ],
});
```

```typescript
// src/mocks/user.ts
import { MockMethod } from 'vite-plugin-mock';

export default [
    {
        url: '/api/users',
        method: 'get',
        response: ({ query }) => {
            const { page = 1, pageSize = 10 } = query;
            return {
                code: 200,
                data: {
                    list: Array.from({ length: Number(pageSize) }, (_, i) => ({
                        id: (Number(page) - 1) * Number(pageSize) + i + 1,
                        name: `用户${(Number(page) - 1) * Number(pageSize) + i + 1}`,
                    })),
                    total: 100,
                },
            };
        },
    },
    {
        url: '/api/users',
        method: 'post',
        response: ({ body }) => ({
            code: 200,
            message: '创建成功',
            data: { id: Math.random().toString(36).slice(2), ...body },
        }),
    },
] as MockMethod[];
```

**开发环境 proxy 配置：**

```typescript
// vite.config.ts
export default defineConfig({
    server: {
        proxy: {
            // 开发阶段代理到后端真实服务
            '/api': {
                target: 'http://localhost:8080',
                changeOrigin: true,
                // rewrite: (path) => path.replace(/^\/api/, ''),
            },
            // WebSocket 代理
            '/ws': {
                target: 'ws://localhost:8080',
                ws: true,
            },
        },
    },
});
```

### Apifox / YApi 接口管理平台

**Apifox 核心功能：**

```yaml
# Apifox 接口文档示例（OpenAPI 3.0 格式）
openapi: 3.0.0
info:
  title: 用户服务 API
  version: 1.0.0
paths:
  /api/users:
    get:
      summary: 获取用户列表
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: pageSize
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  code:
                    type: integer
                  data:
                    type: object
                    properties:
                      list:
                        type: array
                        items:
                          $ref: '#/components/schemas/User'
                      total:
                        type: integer
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
          format: email
```

**Apifox 自动生成代码：**

```bash
# 安装 Apifox CLI
npm install apifox-cli --save-dev

# 从 OpenAPI 规范生成 TypeScript 客户端
npx apifox codegen -i api-spec.yaml -o src/api/client -l typescript-axios
```

**YApi Mock 服务配置：**

```javascript
// YApi Mock 脚本（可在 YApi 平台编辑）
{
    "code": 200,
    "data": {
        "list|5-10": [{
            "id|+1": 1,
            "name": "@cname",
            "email": "@email",
            "avatar": "@image('80x80')"
        }],
        "total|50-200": 1
    }
}

// YApi 高级 Mock：根据请求参数返回不同数据
Mock.mock(/\/api\/users/, 'get', function(options) {
    const token = options.headers.Authorization;
    if (!token) {
        return { code: 401, message: '未登录' };
    }
    return { code: 200, data: { /* ... */ } };
});
```

**团队协作流程：**

```
1. 后端在 Apifox 定义接口文档（请求/响应/状态码）
2. 前端基于文档和 Mock 服务并行开发
3. 接口变更时，Apifox 自动通知相关人员
4. 联调阶段切换到真实后端服务
5. 接口文档与代码同步更新，保证一致性
```

### TypeScript 类型安全的 Mock

**基于接口类型生成 Mock 数据：**

```typescript
// src/types/user.ts
interface User {
    id: number;
    name: string;
    email: string;
    role: 'admin' | 'editor' | 'viewer';
    createdAt: string;
}

interface ApiResponse<T> {
    code: number;
    message: string;
    data: T;
}

// 类型安全的 Mock 工厂
function createMockUser(overrides?: Partial<User>): User {
    return {
        id: Math.floor(Math.random() * 10000),
        name: `用户${Math.floor(Math.random() * 1000)}`,
        email: `user${Math.floor(Math.random() * 1000)}@example.com`,
        role: ['admin', 'editor', 'viewer'][Math.floor(Math.random() * 3)] as User['role'],
        createdAt: new Date().toISOString(),
        ...overrides,
    };
}

// 使用
const mockUser = createMockUser({ name: 'Alice' });
// TypeScript 自动推断类型为 User

const mockResponse: ApiResponse<User[]> = {
    code: 200,
    message: 'success',
    data: Array.from({ length: 10 }, () => createMockUser()),
};

// as-satisfies 验证：确保数据满足类型且不丢失字面量类型
const mockConfig = {
    apiUrl: '/api/users',
    method: 'GET',
    timeout: 5000,
} as const satisfies Record<string, string | number>;

// typeof mockConfig.apiUrl === '/api/users'（字面量类型保留）
```

**泛型 Mock 工具：**

```typescript
// 通用分页响应 Mock
function createPaginatedMock<T>(
    itemFactory: () => T,
    options: { page?: number; pageSize?: number; total?: number } = {}
): ApiResponse<{ list: T[]; total: number; page: number; pageSize: number }> {
    const { page = 1, pageSize = 10, total = 50 } = options;
    return {
        code: 200,
        message: 'success',
        data: {
            list: Array.from({ length: pageSize }, itemFactory),
            total,
            page,
            pageSize,
        },
    };
}

// 使用
const usersPage = createPaginatedMock(() => createMockUser(), { page: 1, pageSize: 20 });
const postsPage = createPaginatedMock(() => createMockPost(), { total: 200 });
```

### 契约测试与接口变更管理

**Swagger / OpenAPI 规范：**

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: 电商平台 API
  version: 2.0.0
servers:
  - url: https://api.example.com/v2
    description: 生产环境
  - url: http://localhost:3001
    description: 本地开发
paths:
  /api/products:
    get:
      operationId: getProducts
      tags: [商品]
      summary: 获取商品列表
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/PageSizeParam'
        - name: category
          in: query
          schema:
            type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductListResponse'
components:
  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        default: 1
    PageSizeParam:
      name: pageSize
      in: query
      schema:
        type: integer
        default: 10
  schemas:
    Product:
      type: object
      required: [id, name, price]
      properties:
        id:
          type: integer
        name:
          type: string
          minLength: 1
          maxLength: 100
        price:
          type: number
          minimum: 0
        category:
          type: string
    ProductListResponse:
      type: object
      properties:
        code:
          type: integer
        data:
          type: object
          properties:
            list:
              type: array
              items:
                $ref: '#/components/schemas/Product'
            total:
              type: integer
```

**自动生成客户端代码：**

```bash
# 使用 openapi-typescript-codegen
npm install openapi-typescript-codegen --save-dev
npx openapi-typescript-codegen \
    --input openapi.yaml \
    --output src/api/generated \
    --client axios \
    --name ApiClient

# 或使用 openapi-typescript（仅生成类型）
npm install openapi-typescript --save-dev
npx openapi-typescript openapi.yaml -o src/types/api.d.ts
```

```typescript
// 使用自动生成的类型
import type { Product, ProductListResponse } from '@/types/api';

async function fetchProducts(page: number): Promise<ProductListResponse> {
    const { data } = await axios.get<ProductListResponse>('/api/products', {
        params: { page },
    });
    return data;
}

// 契约测试：验证后端响应是否符合 OpenAPI 规范
import pact from 'pact';
const { Pact } = pact;

const provider = new Pact({
    consumer: 'WebFrontend',
    provider: 'ProductService',
});

describe('商品服务契约', () => {
    beforeAll(() => provider.setup());
    afterAll(() => provider.finalize());

    it('获取商品列表', async () => {
        await provider.addInteraction({
            state: '存在10个商品',
            uponReceiving: '获取商品列表请求',
            withRequest: {
                method: 'GET',
                path: '/api/products',
                query: { page: '1' },
            },
            willRespondWith: {
                status: 200,
                headers: { 'Content-Type': 'application/json' },
                body: {
                    code: 200,
                    data: {
                        list: pact.eachLike({
                            id: 1,
                            name: '测试商品',
                            price: 99.9,
                        }),
                        total: 10,
                    },
                },
            },
        });

        const response = await fetchProducts(1);
        expect(response.code).toBe(200);
    });
});
```

### Mock 数据工厂模式

**factory 函数：**

```typescript
// src/mocks/factories/userFactory.ts
import { faker } from '@faker-js/faker/locale/zh_CN';

interface User {
    id: number;
    name: string;
    email: string;
    avatar: string;
    phone: string;
    role: 'admin' | 'editor' | 'viewer';
    department: string;
    createdAt: string;
}

// 工厂函数：支持覆盖部分字段
export function createUser(overrides: Partial<User> = {}): User {
    return {
        id: faker.number.int({ min: 1, max: 99999 }),
        name: faker.person.fullName(),
        email: faker.internet.email(),
        avatar: faker.image.avatar(),
        phone: faker.phone.number(),
        role: faker.helpers.arrayElement(['admin', 'editor', 'viewer'] as const),
        department: faker.company.name(),
        createdAt: faker.date.past().toISOString(),
        ...overrides,
    };
}

// 批量创建
export function createUserList(count: number, overrides: Partial<User> = {}): User[] {
    return Array.from({ length: count }, () => createUser(overrides));
}

// 创建关联数据
interface Post {
    id: number;
    title: string;
    content: string;
    authorId: number;
    tags: string[];
}

export function createPost(authorId: number, overrides: Partial<Post> = {}): Post {
    return {
        id: faker.number.int({ min: 1, max: 99999 }),
        title: faker.lorem.sentence(),
        content: faker.lorem.paragraphs(3),
        authorId,
        tags: faker.helpers.arrayElements(
            ['前端', 'React', 'Vue', 'Node.js', 'TypeScript', '工程化'],
            { min: 1, max: 3 },
        ),
        ...overrides,
    };
}

// 关联数据工厂
export function createUserDataWithPosts() {
    const user = createUser();
    const posts = Array.from({ length: 3 }, () => createPost(user.id));
    return { user, posts };
}
```

**faker.js 完整用法：**

```typescript
import { faker } from '@faker-js/faker/locale/zh_CN';

// 人物
faker.person.firstName();          // 名
faker.person.lastName();           // 姓
faker.person.fullName();           // 全名
faker.person.jobTitle();           // 职位

// 地址
faker.location.city();             // 城市
faker.location.streetAddress();    // 街道
faker.location.zipCode();          // 邮编

// 互联网
faker.internet.email();            // 邮箱
faker.internet.url();              // URL
faker.internet.username();         // 用户名
faker.internet.password();         // 密码

// 商业
faker.commerce.productName();      // 产品名
faker.commerce.price();            // 价格
faker.company.name();              // 公司名

// 日期
faker.date.past();                 // 过去日期
faker.date.future();               // 未来日期
faker.date.between({ from: '2024-01-01', to: '2026-12-31' });

// 自定义序列
let userId = 0;
const sequentialUser = () => ({
    id: ++userId,
    name: faker.person.fullName(),
});
```

### 测试环境中的 Mock

**Vitest 中的 vi.fn / vi.mock：**

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

// 1. vi.fn — Mock 函数
it('vi.fn 基础用法', () => {
    const callback = vi.fn();

    callback('hello');
    callback('world');

    expect(callback).toHaveBeenCalled();
    expect(callback).toHaveBeenCalledTimes(2);
    expect(callback).toHaveBeenCalledWith('hello');
    expect(callback).toHaveBeenLastCalledWith('world');
    expect(callback).toHaveReturned(); // 函数已返回

    // 设置返回值
    const fn = vi.fn().mockReturnValue(42);
    expect(fn()).toBe(42);

    // 设置异步返回值
    const asyncFn = vi.fn().mockResolvedValue({ id: 1 });
    await expect(asyncFn()).resolves.toEqual({ id: 1 });
});

// 2. vi.mock — 模块级 Mock
vi.mock('@/api/user', () => ({
    fetchUserList: vi.fn(),
    createUser: vi.fn(),
    deleteUser: vi.fn(),
}));

import { fetchUserList, createUser } from '@/api/user';

beforeEach(() => {
    vi.clearAllMocks(); // 每个测试前清理
});

it('获取用户列表', async () => {
    fetchUserList.mockResolvedValue({
        code: 200,
        data: { list: [{ id: 1, name: 'Alice' }], total: 1 },
    });

    const result = await fetchUserList({ page: 1 });
    expect(result.data.list).toHaveLength(1);
});

// 3. vi.spyOn — 监视对象方法
it('spyOn 监视方法调用', () => {
    const obj = {
        greet(name: string) {
            return `Hello, ${name}!`;
        },
    };

    const spy = vi.spyOn(obj, 'greet');
    obj.greet('Alice');

    expect(spy).toHaveBeenCalledWith('Alice');
    expect(spy).toHaveReturnedWith('Hello, Alice!');

    // 还可以修改实现
    spy.mockImplementation(() => 'mocked');
    expect(obj.greet('Bob')).toBe('mocked');

    spy.mockRestore(); // 恢复原始实现
});

// 4. 局部 Mock（只 Mock 模块的部分导出）
vi.mock('@/utils/logger', async (importOriginal) => {
    const actual = await importOriginal<typeof import('@/utils/logger')>();
    return {
        ...actual,
        // 只 Mock logError，保留其他方法
        logError: vi.fn(),
    };
});

// 5. MSW + Vitest 集成
import { http, HttpResponse } from 'msw';
import { server } from '@/mocks/server';

describe('用户管理', () => {
    it('加载用户列表', async () => {
        server.use(
            http.get('/api/users', () => {
                return HttpResponse.json({
                    code: 200,
                    data: { list: [{ id: 1, name: 'Alice' }], total: 1 },
                });
            }),
        );

        render(<UserList />);
        expect(await screen.findByText('Alice')).toBeInTheDocument();
    });

    it('处理网络错误', async () => {
        server.use(
            http.get('/api/users', () => {
                return HttpResponse.error();
            }),
        );

        render(<UserList />);
        expect(await screen.findByText('加载失败，请重试')).toBeInTheDocument();
    });
});
```

**Jest 中的 Mock：**

```typescript
import { jest } from '@jest/globals';

// jest.fn — 与 vi.fn 用法一致
const mockFn = jest.fn();
mockFn.mockReturnValue('default');
mockFn.mockResolvedValue({ data: 'async' });

// jest.mock — 模块 Mock
jest.mock('@/api/user', () => ({
    fetchUser: jest.fn(),
}));

// jest.spyOn — 与 vi.spyOn 用法一致
const spy = jest.spyOn(console, 'error').mockImplementation(() => {});

// 清理
afterEach(() => {
    jest.clearAllMocks();
    spy.mockRestore();
});
```

## 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Mock 数据与真实数据不一致 | Mock 手写/随机生成，接口变更未同步 | 基于 OpenAPI 规范自动生成，使用契约测试验证 |
| 接口变更未同步通知 | 缺乏统一接口管理平台 | 使用 Apifox，接口变更自动通知前端 |
| MSW Service Worker 注册失败 | 未执行 `msw init` 生成文件 | 运行 `npx msw init public/`，确认 `mockServiceWorker.js` 存在 |
| Mock 影响集成测试 | Mock 未在测试后清理 | `afterEach(() => server.resetHandlers())`，`vi.clearAllMocks()` |
| Mock.js 无法拦截 fetch | Mock.js 只拦截 XMLHttpRequest | 改用 MSW（拦截 Service Worker 层）或 polyfill fetch 为 XHR |
| json-server 不支持复杂业务逻辑 | 只提供基础 CRUD | 添加自定义中间件处理业务逻辑，或使用 Express 扩展 |
| vite-plugin-mock 生产环境泄露 | 配置未关闭生产环境 Mock | 设置 `prodEnabled: false`，环境变量判断 |
| faker.js v6 争议版本 | 原作者恶意破坏 | 使用 `@faker-js/faker`（社区维护版本） |

## 面试题

**Q1: 前端 Mock 方案有哪些？各有什么优缺点？**
> 主要方案有四种：Mock.js 拦截 XHR 生成随机数据，优点是简单快速，缺点是只拦截 XHR 不支持 fetch、数据与真实接口容易不一致；MSW 在 Service Worker 层拦截，优点是拦截范围广（XHR/fetch 均生效）、支持浏览器和 Node 双模式、对业务代码零侵入，缺点是配置稍复杂；json-server 搭建本地 REST 服务器，优点是提供完整 CRUD 接口、支持关联查询，缺点是不支持复杂业务逻辑；接口管理平台（Apifox/YApi）内置 Mock 服务，优点是文档与 Mock 一体化，缺点是依赖平台、定制性有限。

**Q2: MSW 的工作原理是什么？为什么说它是最接近生产环境的 Mock 方案？**
> MSW 在浏览器中注册 Service Worker 拦截网络请求，在 Node 环境中使用 `msw/node` 的 `setupServer` 拦截请求。Service Worker 运行在浏览器与网络之间，可以在请求到达网络前返回 Mock 响应，应用代码完全无感知——使用的是真实的 `fetch`/`XMLHttpRequest`，只是被 Service Worker 拦截了。这使其最接近生产环境：不修改业务代码、不 polyfill 网络 API、请求走真实网络栈，只是响应被拦截替换。

**Q3: Mock.js 有哪些局限性？**
> （1）只拦截 XMLHttpRequest，不支持 fetch API；（2）随机数据与真实数据结构容易不一致，缺乏类型约束；（3）拦截发生在 XHR 层，开发工具 Network 面板看不到请求，调试困难；（4）无法模拟网络错误、超时等异常场景的细粒度控制；（5）对 Node 环境无支持，不能用于服务端渲染测试；（6）Mock.js 库已多年不维护，存在潜在安全风险。

**Q4: 如何保证 Mock 数据与真实接口保持一致？**
> （1）基于 OpenAPI/Swagger 规范定义接口，Mock 数据从规范自动生成而非手写；（2）使用 Apifox 等平台，接口文档变更自动同步 Mock 服务；（3）TypeScript 类型约束：基于接口类型定义生成 Mock 数据，编译期校验一致性；（4）契约测试（Pact）：验证消费者（前端）与提供者（后端）的契约是否匹配；（5）接口变更时 CI 流水线运行契约测试，不一致则构建失败。

**Q5: OpenAPI 规范在前端工程化中的作用是什么？**
> OpenAPI 规范是前后端的接口契约，作用包括：（1）单一数据源——接口定义、参数、响应结构、状态码均以 YAML/JSON 格式集中描述；（2）自动生成 TypeScript 类型定义，保证前端代码类型安全；（3）自动生成 API 客户端代码（axios 请求封装），减少手写样板代码；（4）自动生成 Mock 数据和 Mock 服务，前端可基于规范并行开发；（5）契约测试的依据，CI 中验证前后端接口一致性；（6）接口文档自动生成，保证文档与代码始终同步。

**Q6: 测试中 Mock 的最佳实践是什么？**
> （1）Mock 最小化——只 Mock 外部依赖（网络请求、定时器、第三方模块），不 Mock 被测函数内部实现；（2）优先使用 MSW 拦截网络请求而非 `vi.mock` 整个模块，MSW 更接近真实行为；（3）每个测试后清理 Mock 状态（`vi.clearAllMocks()`/`server.resetHandlers()`），避免测试间互相影响；（4）Mock 数据使用工厂函数生成，支持覆盖部分字段，避免硬编码；（5）测试正常和异常两条路径——Mock 成功响应和错误响应（500、401、超时）；（6）避免过度 Mock——如果测试需要 Mock 大量内部函数，说明被测单元职责过大，应先重构。

**Q7: faker.js 的用途是什么？为什么不推荐使用原版 faker.js？**
> faker.js 用于生成真实感的随机测试数据（姓名、邮箱、地址、公司名、价格等），避免手写硬编码数据，使测试更具随机性和覆盖率。不推荐使用原版 faker.js 是因为 2022 年原作者恶意删除代码并破坏包（引入无限循环），导致全球大量项目构建失败。社区随后 fork 创建了 `@faker-js/faker`，由社区维护，功能更丰富且持续更新，应始终使用 `@faker-js/faker`。

**Q8: 接口管理有哪些痛点？如何解决？**
> 痛点：（1）接口文档与代码不同步——后端改了接口但文档没更新；（2）Mock 数据手写维护成本高——与真实接口容易不一致；（3）联调效率低——前后端对接时才发现接口定义有歧义；（4）接口变更无通知——前端不知道后端改了字段名或类型；（5）接口散落在各处——没有统一入口查看所有 API。解决方案：（1）使用 Apifox/YApi 等平台集中管理接口，文档即代码；（2）基于 OpenAPI 规范自动生成类型和 Mock，减少手写；（3）接口变更时平台自动通知相关人员；（4）契约测试在 CI 中验证前后端一致性；（5）规范先行——先定义接口文档再开发，而不是先写代码再补文档。

---

**相关链接：**
- [[前端测试Jest与Vitest]]
- [[Webpack与Vite]]
- [[BFF与Serverless]]

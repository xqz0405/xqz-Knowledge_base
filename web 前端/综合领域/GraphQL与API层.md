---
tags:
  - Web前端
  - 综合领域
  - GraphQL
  - API
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# GraphQL与API层

## What — 什么是 GraphQL 与 API 层

### GraphQL 核心定义

GraphQL 是一种用于 API 的查询语言和运行时，由 Facebook 于 2015 年开源。它提供了一种**强类型 Schema** 来定义数据模型，让前端能够**精确声明**所需数据，服务端按声明返回，不多不少。

**三个核心操作：**

| 操作 | 用途 | 类比 REST |
|------|------|----------|
| **Query** | 查询数据 | GET |
| **Mutation** | 变更数据（创建/更新/删除） | POST / PUT / DELETE |
| **Subscription** | 实时订阅数据变更 | WebSocket |

**核心组成：**

```
┌─────────────────────────────────────────────────┐
│                  GraphQL 系统                     │
│                                                   │
│  ┌──────────────┐    ┌────────────────────────┐  │
│  │   Schema     │───▶│   Resolver             │  │
│  │ (类型定义)    │    │ (解析器：获取数据的函数) │  │
│  │  - Type      │    │  - Query Resolver      │  │
│  │  - Query     │    │  - Mutation Resolver   │  │
│  │  - Mutation  │    │  - Field Resolver      │  │
│  │  - Subscription│  │  - DataLoader          │  │
│  └──────────────┘    └────────────────────────┘  │
│         │                        │                │
│         ▼                        ▼                │
│  ┌──────────────┐    ┌────────────────────────┐  │
│  │  前端客户端   │    │    数据源               │  │
│  │  - Apollo    │    │  - REST API            │  │
│  │  - urql      │    │  - 数据库              │  │
│  │  - Relay     │    │  - 微服务 gRPC         │  │
│  └──────────────┘    └────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### API 层的定位

API 层是前端与后端之间的数据桥梁，GraphQL 作为 API 层方案，让前端从"被动接受后端返回的数据"转变为"主动声明需要的数据"。

**数据获取方式演进：**

```
传统 REST → BFF 聚合 → GraphQL 按需查询
(前端适应后端)  (中间层适配)  (前端驱动数据)
```

---

## Why — 为什么用 GraphQL

### REST 的痛点

| 痛点 | 说明 | 示例 |
|------|------|------|
| **过度获取（Over-fetching）** | 接口返回了前端不需要的字段 | 用户列表页只需要 name/avatar，但 `/users` 返回了全部 30+ 字段 |
| **不足获取（Under-fetching）** | 单个接口数据不够，需多次请求 | 首页需要用户信息 + 订单 + 推荐，需请求 3 个接口 |
| **多端需求不同** | Web/App/小程序对同一资源的需求不同 | App 需要简略信息，Web 后台需要完整信息 |
| **接口联调成本高** | 前后端需协商字段、版本、分页格式 | 新增一个展示字段需后端改接口、发版 |
| **嵌套数据获取难** | 获取关联数据需多次请求或后端定制接口 | 获取文章及其作者信息和评论需要 3 次请求 |

### GraphQL 的解决方式

```graphql
# 一次请求获取所有需要的数据，不多不少
query GetUserDashboard {
  user(id: "u1") {
    name
    avatar
    level
  }
  recentOrders(limit: 5) {
    id
    productName
    amount
    status
  }
  recommendedProducts(limit: 10) {
    id
    name
    price
  }
}
```

### GraphQL vs REST vs gRPC vs tRPC

| 维度 | GraphQL | REST | gRPC | tRPC |
|------|---------|------|------|------|
| **查询灵活性** | 极高（前端声明字段） | 低（固定返回结构） | 低（固定消息格式） | 高（全栈 TypeScript） |
| **类型安全** | Schema 强类型 | 无内置（需 OpenAPI） | Protobuf 强类型 | TypeScript 推导 |
| **学习曲线** | 高（新概念多） | 低（HTTP 基础） | 中（需学 Protobuf） | 低（TS 开发者友好） |
| **生态系统** | 成熟（Apollo/Relay/Codegen） | 最成熟 | 服务端成熟 | TS 生态 |
| **缓存** | 复杂（需规范化缓存） | 简单（HTTP 缓存） | 无内置 | 依赖客户端 |
| **网络协议** | HTTP（Query/Mutation）+ WebSocket（Subscription） | HTTP | HTTP/2（双向流） | HTTP |
| **浏览器支持** | 原生支持 | 原生支持 | 需 gRPC-Web | 原生支持 |
| **多端统一** | 优秀（按需查询） | 一般（需 BFF） | 服务端为主 | TS 专属 |
| **实时数据** | Subscription 原生支持 | 需 WebSocket | 双向流 | 需额外方案 |
| **适用场景** | 多端复杂查询、BFF 层 | 通用 CRUD、公开 API | 微服务间通信 | 全栈 TS 项目 |
| **文件上传** | 需 multipart 规范 | 原生支持 | 原生支持 | 原生支持 |
| **调试工具** | GraphQL Playground / Apollo Explorer | Postman / curl | grpcurl | 浏览器 DevTools |

---

## How — 怎么做

### 1. Schema 定义语言（SDL）

Schema 是 GraphQL 的核心，用 SDL（Schema Definition Language）定义数据模型。

#### 类型（Type）

```graphql
# 基础对象类型
type User {
  id: ID!          # ! 表示非空
  name: String!
  email: String!
  age: Int
  role: UserRole!
  posts: [Post!]!  # 列表类型，[Post!]! 表示非空列表且元素非空
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  tags: [String!]!
  comments(limit: Int = 10): [Comment!]!  # 字段参数与默认值
  createdAt: DateTime!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  createdAt: DateTime!
}
```

#### 枚举（Enum）

```graphql
enum UserRole {
  ADMIN
  EDITOR
  VIEWER
}

enum OrderStatus {
  PENDING
  PAID
  SHIPPED
  COMPLETED
  CANCELLED
}

enum SortDirection {
  ASC
  DESC
}
```

#### 接口（Interface）

```graphql
# 定义共享字段的抽象类型
interface Node {
  id: ID!
  createdAt: DateTime!
}

interface searchable {
  searchScore: Float
}

# 实现接口
type User implements Node & searchable {
  id: ID!
  name: String!
  email: String!
  createdAt: DateTime!
  searchScore: Float
}

type Post implements Node & searchable {
  id: ID!
  title: String!
  content: String!
  createdAt: DateTime!
  searchScore: Float
}
```

#### 联合类型（Union）

```graphql
# 联合类型：返回多种类型中的一种
union SearchResult = User | Post | Comment

union PaymentMethod = CreditCard | BankTransfer | WeChatPay

type CreditCard {
  last4Digits: String!
  brand: String!
}

type BankTransfer {
  bankName: String!
  accountNumber: String!
}

type WeChatPay {
  openid: String!
}

type SearchResponse {
  query: String!
  results: [SearchResult!]!
  totalCount: Int!
}
```

#### 输入类型（Input）

```graphql
# 输入类型用于 Mutation 参数和复杂查询过滤
input CreateUserInput {
  name: String!
  email: String!
  password: String!
  role: UserRole = VIEWER  # 枚举默认值
}

input UpdateUserInput {
  name: String
  email: String
  avatar: String
}

input PostFilterInput {
  authorId: ID
  tags: [String!]
  status: PostStatus
  createdAtRange: DateRangeInput
}

input DateRangeInput {
  start: DateTime!
  end: DateTime!
}

input PaginationInput {
  page: Int = 1
  pageSize: Int = 20
}

input SortInput {
  field: String!
  direction: SortDirection = DESC
}
```

#### 指令（Directive）

```graphql
# 内置指令
directive @include(if: Boolean!) on FIELD | FRAGMENT_SPREAD | INLINE_FRAGMENT
directive @skip(if: Boolean!) on FIELD | FRAGMENT_SPREAD | INLINE_FRAGMENT
directive @deprecated(reason: String = "No longer supported") on FIELD_DEFINITION | ENUM_VALUE

# 自定义指令
directive @auth(requires: UserRole = ADMIN) on FIELD_DEFINITION
directive @cache(ttl: Int = 300) on FIELD_DEFINITION
directive @validate(validator: String!) on ARGUMENT_DEFINITION | INPUT_FIELD_DEFINITION
directive @rateLimit(limit: Int!, window: Int!) on FIELD_DEFINITION

# 在 Schema 中使用指令
type Query {
  user(id: ID!): User! @auth(requires: VIEWER) @cache(ttl: 60)
  users(filter: PostFilterInput, pagination: PaginationInput): [User!]!
    @auth(requires: ADMIN)
    @rateLimit(limit: 100, window: 60)
  search(query: String!): SearchResponse!
}

type User {
  id: ID!
  name: String!
  legacyCode: String @deprecated(reason: "使用 role 字段代替")
}
```

#### 完整 Schema 示例

```graphql
# schema.graphql
schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}

type Query {
  # 用户查询
  user(id: ID!): User!
  users(filter: UserFilterInput, pagination: PaginationInput): UserConnection!
  me: User! @auth(requires: VIEWER)

  # 文章查询
  post(id: ID!): Post!
  posts(filter: PostFilterInput, pagination: PaginationInput): PostConnection!

  # 搜索
  search(query: String!, type: SearchType): SearchResponse!
}

type Mutation {
  # 用户操作
  createUser(input: CreateUserInput!): User! @rateLimit(limit: 5, window: 60)
  updateUser(id: ID!, input: UpdateUserInput!): User! @auth(requires: VIEWER)
  deleteUser(id: ID!): Boolean! @auth(requires: ADMIN)

  # 文章操作
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post! @auth(requires: EDITOR)
  deletePost(id: ID!): Boolean! @auth(requires: ADMIN)
}

type Subscription {
  onPostCreated: Post!
  onCommentAdded(postId: ID!): Comment!
  onUserStatusChanged(userId: ID!): UserStatus!
}

type UserStatus {
  userId: ID!
  online: Boolean!
  lastSeen: DateTime
}
```

---

### 2. Query 查询

#### 字段选择与参数

```graphql
# 基础查询 — 只请求需要的字段
query GetUserName {
  user(id: "u1") {
    name
    avatar
  }
}

# 带参数查询
query GetUsers {
  users(
    filter: { role: VIEWER }
    pagination: { page: 1, pageSize: 10 }
  ) {
    items {
      id
      name
      email
    }
    total
    hasMore
  }
}
```

#### 别名（Alias）

```graphql
# 同一字段不同参数，用别名区分
query CompareUsers {
  admin: user(id: "u1") {
    name
    role
  }
  viewer: user(id: "u2") {
    name
    role
  }
  # 同一请求获取不同角色用户的对比信息
}
```

#### 片段（Fragment）

```graphql
# 定义可复用的字段集合
fragment UserFields on User {
  id
  name
  avatar
  email
  role
  createdAt
}

fragment PostSummary on Post {
  id
  title
  createdAt
  author {
    ...UserFields  # 片段嵌套
  }
}

# 在多处使用片段
query GetUserWithPosts {
  user(id: "u1") {
    ...UserFields
    posts(pagination: { pageSize: 5 }) {
      items {
        ...PostSummary
      }
    }
  }
}

query GetDashboard {
  me {
    ...UserFields
  }
  recentPosts: posts(pagination: { pageSize: 10 }) {
    items {
      ...PostSummary
    }
  }
}
```

#### 内联片段（Inline Fragment）

```graphql
# 联合类型使用内联片段处理不同类型
query Search {
  search(query: "graphql") {
    query
    results {
      ... on User {
        id
        name
        avatar
      }
      ... on Post {
        id
        title
        createdAt
      }
      ... on Comment {
        id
        content
        createdAt
      }
    }
  }
}

# 接口类型使用内联片段
query Nodes {
  nodes(ids: ["u1", "p1"]) {
    id
    createdAt
    ... on User {
      name
      email
    }
    ... on Post {
      title
      content
    }
  }
}
```

#### 操作名称与变量

```graphql
# 命名操作 + 变量声明
query GetUserPosts(
  $userId: ID!
  $limit: Int = 10
  $cursor: String
) {
  user(id: $userId) {
    name
    posts(pagination: { pageSize: $limit, cursor: $cursor }) {
      items {
        id
        title
        createdAt
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
}
```

```typescript
// 前端传变量
const { data } = await client.query({
  query: GET_USER_POSTS,
  variables: {
    userId: 'u1',
    limit: 20,
    cursor: 'cursor-abc',
  },
});
```

---

### 3. Mutation 变更

#### 创建 / 更新 / 删除

```graphql
# 创建用户
mutation CreateUser {
  createUser(input: {
    name: "张三"
    email: "zhangsan@example.com"
    password: "secure123"
    role: VIEWER
  }) {
    id
    name
    email
    role
    createdAt
  }
}

# 更新用户
mutation UpdateUser {
  updateUser(
    id: "u1"
    input: {
      name: "李四"
      avatar: "https://img.example.com/avatar2.jpg"
    }
  ) {
    id
    name
    avatar
    updatedAt
  }
}

# 删除用户
mutation DeleteUser {
  deleteUser(id: "u1")  # 返回 Boolean
}
```

#### 输入类型与变量

```graphql
# 使用变量传递输入，避免硬编码
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
    content
    tags
    author {
      id
      name
    }
    createdAt
  }
}
```

```typescript
// 前端调用
const [createPost] = useMutation(CREATE_POST, {
  variables: {
    input: {
      title: 'GraphQL 入门指南',
      content: '这是一篇关于 GraphQL 的文章...',
      tags: ['GraphQL', 'API', '前端'],
    },
  },
});
```

#### 返回修改后的数据

```graphql
# Mutation 最佳实践：返回修改后的完整对象
mutation UpdatePost($id: ID!, $input: UpdatePostInput!) {
  updatePost(id: $id, input: $input) {
    id
    title
    content
    tags
    updatedAt
    author {
      ...UserFields
    }
  }
}
```

#### 批量操作

```graphql
input BatchUpdateUserInput {
  ids: [ID!]!
  updates: UpdateUserInput!
}

type BatchUpdateResult {
  success: [User!]!
  failed: [BatchUpdateError!]!
  totalCount: Int!
}

type BatchUpdateError {
  id: ID!
  reason: String!
}

type Mutation {
  batchUpdateUsers(input: BatchUpdateUserInput!): BatchUpdateResult!
    @auth(requires: ADMIN)
}
```

---

### 4. Subscription 订阅

#### WebSocket 实时数据

```graphql
# 订阅新文章创建
subscription OnPostCreated {
  onPostCreated {
    id
    title
    author {
      name
      avatar
    }
    createdAt
  }
}

# 订阅文章评论（带参数）
subscription OnCommentAdded($postId: ID!) {
  onCommentAdded(postId: $postId) {
    id
    content
    author {
      name
      avatar
    }
    createdAt
  }
}

# 订阅用户在线状态
subscription OnUserStatusChanged($userId: ID!) {
  onUserStatusChanged(userId: $userId) {
    userId
    online
    lastSeen
  }
}
```

#### Apollo Subscription 服务端

```typescript
// server/subscriptions.ts
import { PubSub } from 'graphql-subscriptions';
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import { WebSocketServer } from 'ws';
import { useServer } from 'graphql-ws/lib/use/ws';
import { makeExecutableSchema } from '@graphql-tools/schema';

const pubsub = new PubSub();

// 事件常量
const POST_CREATED = 'POST_CREATED';
const COMMENT_ADDED = 'COMMENT_ADDED';
const USER_STATUS_CHANGED = 'USER_STATUS_CHANGED';

export const resolvers = {
  Subscription: {
    onPostCreated: {
      subscribe: () => pubsub.asyncIterator([POST_CREATED]),
    },
    onCommentAdded: {
      subscribe: (_: any, { postId }: { postId: string }) => {
        // 按 postId 过滤
        return pubsub.asyncIterator([
          `${COMMENT_ADDED}_${postId}`,
        ]);
      },
    },
    onUserStatusChanged: {
      subscribe: (_: any, { userId }: { userId: string }) => {
        return pubsub.asyncIterator([
          `${USER_STATUS_CHANGED}_${userId}`,
        ]);
      },
    },
  },
  Mutation: {
    createPost: async (_: any, { input }: any, context: any) => {
      const post = await postService.create(input, context.userId);
      // 发布事件
      pubsub.publish(POST_CREATED, { onPostCreated: post });
      return post;
    },
    addComment: async (_: any, { input }: any, context: any) => {
      const comment = await commentService.create(input, context.userId);
      pubsub.publish(`${COMMENT_ADDED}_${input.postId}`, {
        onCommentAdded: comment,
      });
      return comment;
    },
  },
};

// WebSocket 服务配置
export function setupWebSocketServer(server: any) {
  const wsServer = new WebSocketServer({
    server,
    path: '/graphql',
  });

  const schema = makeExecutableSchema({ typeDefs, resolvers });

  useServer(
    {
      schema,
      context: async (ctx) => {
        // WebSocket 认证
        const token = ctx.connectionParams?.authorization;
        const user = await verifyToken(token);
        return { user };
      },
      onConnect: () => console.log('WebSocket 客户端已连接'),
      onDisconnect: () => console.log('WebSocket 客户端已断开'),
    },
    wsServer,
  );
}
```

#### Subscription 使用场景

| 场景 | 说明 | Subscription 示例 |
|------|------|-------------------|
| 实时通知 | 新消息、系统通知 | `onNotification(userId)` |
| 协作编辑 | 多人同时编辑文档 | `onDocumentChanged(docId)` |
| 数据监控 | 仪表盘实时数据 | `onMetricUpdated(metricId)` |
| 在线状态 | 用户上下线 | `onUserStatusChanged(userId)` |
| 评论/弹幕 | 新评论/弹幕推送 | `onCommentAdded(postId)` |
| 订单状态 | 支付/发货状态变更 | `onOrderStatusChanged(orderId)` |
| 游戏状态 | 多人游戏状态同步 | `onGameStateChanged(gameId)` |

---

### 5. Resolver 解析器

#### 解析器链

```typescript
// resolvers/user.ts
import { UserService } from '../services/userService';
import { PostService } from '../services/postService';

const userService = new UserService();
const postService = new PostService();

export const userResolvers = {
  Query: {
    // 根解析器：获取根对象
    user: async (_: any, { id }: { id: string }) => {
      return userService.getUser(id);
    },
    users: async (_: any, { filter, pagination }: any) => {
      return userService.getUsers(filter, pagination);
    },
    me: async (_: any, __: any, context: any) => {
      return userService.getUser(context.userId);
    },
  },

  // 字段解析器：解析 User 类型的字段
  User: {
    // parent 是上一层解析器的返回值
    posts: async (parent: { id: string }, { pagination }: any) => {
      return postService.getPostsByAuthor(parent.id, pagination);
    },
    // 格式化字段
    displayName: (parent: { name: string; email: string }) => {
      return parent.name || parent.email.split('@')[0];
    },
    // 计算字段
    postCount: async (parent: { id: string }) => {
      return postService.countByAuthor(parent.id);
    },
  },
};
```

**解析器执行流程：**

```
Query: { user(id: "u1") }         → 返回 { id: "u1", name: "张三", email: "..." }
       │
       ▼
User: { posts(parent) }           → parent = { id: "u1", ... } → 返回 [Post, Post, ...]
       │
       ▼
Post: { author(parent) }          → parent = { id: "p1", authorId: "u1", ... } → 返回 User
       │
       ▼
User: { avatar(parent) }          → parent = { id: "u1", ... } → 返回 avatar URL

执行顺序：自顶向下，逐层解析，每一层可以并发
```

#### 数据源组合

```typescript
// resolvers/aggregation.ts
import { UserService } from '../services/userService';
import { OrderService } from '../services/orderService';
import { ProductService } from '../services/productService';
import { RecommendationService } from '../services/recommendationService';

export const aggregationResolvers = {
  Query: {
    userDashboard: async (_: any, { userId }: { userId: string }, context: any) => {
      // 并行请求多个数据源
      const [user, orders, recommendations] = await Promise.all([
        context.dataSources.userService.getUser(userId),
        context.dataSources.orderService.getRecentOrders(userId, { limit: 5 }),
        context.dataSources.recommendationService.getForUser(userId, { limit: 10 }),
      ]);

      return {
        user,
        recentOrders: orders,
        recommendedProducts: recommendations,
      };
    },
  },

  UserDashboard: {
    // 非核心数据允许失败，降级返回
    stats: async (parent: any, __: any, context: any) => {
      try {
        return await context.dataSources.statsService.getUserStats(parent.user.id);
      } catch {
        return { totalOrders: 0, totalSpent: 0, memberDays: 0 };
      }
    },
  },
};

// Apollo Server 的 DataSources 模式
class UserServiceAPI extends RESTDataSource {
  override baseURL = process.env.USER_SERVICE_URL!;

  async getUser(id: string) {
    return this.get(`/users/${id}`);
  }

  async getUsers(filter: any) {
    return this.get('/users', { params: filter });
  }
}

// 在 context 中注入数据源
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    userId: getUserId(req),
    dataSources: {
      userService: new UserServiceAPI(),
      orderService: new OrderServiceAPI(),
      productService: new ProductServiceAPI(),
    },
  }),
});
```

#### N+1 问题与 DataLoader

```typescript
// ❌ N+1 问题：100 篇文章会产生 101 次查询
const resolvers = {
  Post: {
    author: async (parent: { authorId: string }) => {
      // 每个 Post 都单独查一次 User → N+1！
      return userService.getUser(parent.authorId);
    },
  },
};

// 100 篇文章 = 1 次查 Post + 100 次查 Author = 101 次 SQL

// ✅ DataLoader：批量加载，合并为 2 次查询
import DataLoader from 'dataloader';

const userLoader = new DataLoader(async (ids: readonly string[]) => {
  // 一次查询获取所有用户
  const users = await userService.getUsersByIds(ids as string[]);
  // 按原始顺序映射
  const userMap = new Map(users.map((u) => [u.id, u]));
  return ids.map((id) => userMap.get(id));
});

const resolvers = {
  Post: {
    author: async (parent: { authorId: string }) => {
      // DataLoader 自动合并同一 tick 内的所有 load 调用
      return userLoader.load(parent.authorId);
    },
  },
};

// 100 篇文章 = 1 次查 Post + 1 次批量查 Author = 2 次 SQL
```

```typescript
// DataLoader 完整配置
import DataLoader from 'dataloader';

// 每个请求创建新的 DataLoader 实例（请求级缓存）
export function createContext() {
  return {
    userLoader: new DataLoader(
      async (ids: readonly string[]) => {
        const users = await userService.getUsersByIds([...ids]);
        const map = new Map(users.map((u) => [u.id, u]));
        return ids.map((id) => map.get(id) ?? new Error(`User ${id} not found`));
      },
      {
        maxBatchSize: 100,   // 单次批量最大数量
        cache: true,          // 同一请求内缓存
        cacheKeyFn: (key) => key,  // 缓存键函数
      },
    ),
    postLoader: new DataLoader(
      async (ids: readonly string[]) => {
        const posts = await postService.getPostsByIds([...ids]);
        const map = new Map(posts.map((p) => [p.id, p]));
        return ids.map((id) => map.get(id) ?? new Error(`Post ${id} not found`));
      },
    ),
  };
}
```

---

### 6. 前端客户端：Apollo Client

#### 安装与配置

```typescript
// lib/apollo.ts
import { ApolloClient, InMemoryCache, createHttpLink, split } from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';
import { setContext } from '@apollo/client/link/context';
import { onError } from '@apollo/client/link/error';

// HTTP 链接
const httpLink = createHttpLink({
  uri: '/graphql',
  credentials: 'same-origin',
});

// 认证链接
const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      authorization: token ? `Bearer ${token}` : '',
    },
  };
});

// 错误处理链接
const errorLink = onError(({ graphQLErrors, networkError, operation, forward }) => {
  if (graphQLErrors) {
    for (const err of graphQLErrors) {
      switch (err.extensions?.code) {
        case 'UNAUTHENTICATED':
          // Token 过期，尝试刷新
          return fromPromise(refreshToken()).flatMap((newToken) => {
            localStorage.setItem('token', newToken);
            const oldHeaders = operation.getContext().headers;
            operation.setContext({
              headers: { ...oldHeaders, authorization: `Bearer ${newToken}` },
            });
            return forward(operation);
          });
        case 'FORBIDDEN':
          window.location.href = '/403';
          break;
        default:
          console.error(`[GraphQL Error]: ${err.message}`);
      }
    }
  }
  if (networkError) {
    console.error(`[Network Error]: ${networkError.message}`);
  }
});

// WebSocket 链接
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: () => ({
      authorization: localStorage.getItem('token'),
    }),
    on: {
      connected: () => console.log('WebSocket 已连接'),
      closed: () => console.log('WebSocket 已断开'),
    },
  }),
);

// 按操作类型分流：Subscription 走 WebSocket，其余走 HTTP
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  authLink.concat(errorLink).concat(httpLink),
);

// 创建客户端
export const apolloClient = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache({
    // 缓存配置
    typePolicies: {
      Query: {
        fields: {
          posts: relayStylePagination(),  // Relay 风格分页
        },
      },
      User: {
        keyFields: ['id'],  // 缓存键
      },
    },
  }),
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network',  // 先返回缓存，再更新网络数据
    },
    query: {
      fetchPolicy: 'network-only',  // 查询默认走网络
    },
  },
  connectToDevTools: process.env.NODE_ENV === 'development',
});
```

#### 缓存策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| `cache-first` | 优先缓存，缓存无数据才请求网络 | 不常变化的数据 |
| `cache-only` | 只读缓存，不请求网络 | 离线场景 |
| `cache-and-network` | 先返回缓存，同时请求网络更新 | 需要即时性且允许短暂过期 |
| `network-first` | 优先网络，失败才读缓存 | 需要最新数据 |
| `network-only` | 只请求网络，不读缓存 | Mutation 后查询 |
| `no-cache` | 只请求网络，不写入缓存 | 敏感数据 |

#### useQuery / useMutation / useLazyQuery

```tsx
// useQuery — 声明式查询
import { useQuery, gql } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      avatar
      posts(pagination: { pageSize: 5 }) {
        items {
          id
          title
        }
        total
      }
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error, refetch, fetchMore } = useQuery(GET_USER, {
    variables: { id: userId },
    fetchPolicy: 'cache-and-network',
    pollInterval: 0,             // 轮询间隔（0 = 不轮询）
    skip: !userId,               // 条件跳过
    notifyOnNetworkStatusChange: true,  // 网络状态变化时重新渲染
  });

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return null;

  return (
    <div>
      <Avatar src={data.user.avatar} />
      <h1>{data.user.name}</h1>
      <p>{data.user.email}</p>
      <PostList posts={data.user.posts.items} />
      <Button onClick={() => refetch()}>刷新</Button>
    </div>
  );
}
```

```tsx
// useLazyQuery — 手动触发查询
import { useLazyQuery, gql } from '@apollo/client';

const SEARCH_POSTS = gql`
  query SearchPosts($query: String!) {
    search(query: $query) {
      results {
        ... on Post {
          id
          title
        }
        ... on User {
          id
          name
        }
      }
      totalCount
    }
  }
`;

function SearchBar() {
  const [searchPosts, { data, loading }] = useLazyQuery(SEARCH_POSTS);
  const [keyword, setKeyword] = useState('');

  const handleSearch = () => {
    if (keyword.trim()) {
      searchPosts({ variables: { query: keyword } });
    }
  };

  return (
    <div>
      <input
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && handleSearch()}
      />
      <Button onClick={handleSearch}>搜索</Button>
      {loading && <Spinner />}
      {data && <SearchResults results={data.search.results} />}
    </div>
  );
}
```

```tsx
// useMutation — 变更操作
import { useMutation, gql } from '@apollo/client';

const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      content
      tags
      author {
        id
        name
      }
      createdAt
    }
  }
`;

function CreatePostForm() {
  const [createPost, { loading, error }] = useMutation(CREATE_POST, {
    // 变更后自动重新查询相关数据
    refetchQueries: [
      { query: GET_USER_POSTS, variables: { userId: 'me' } },
      'GetDashboard',  // 也可以用操作名称
    ],
    // 或者手动更新缓存
    update(cache, { data }) {
      const newPost = data?.createPost;
      if (!newPost) return;

      cache.modify({
        fields: {
          posts(existingPosts = {}) {
            const newPostRef = cache.writeFragment({
              data: newPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                }
              `,
            });
            return {
              ...existingPosts,
              items: [newPostRef, ...existingPosts.items],
              total: existingPosts.total + 1,
            };
          },
        },
      });
    },
    onCompleted(data) {
      message.success('文章创建成功');
      router.push(`/posts/${data.createPost.id}`);
    },
    onError(error) {
      message.error(`创建失败：${error.message}`);
    },
  });

  const handleSubmit = async (values: any) => {
    await createPost({ variables: { input: values } });
  };

  return (
    <Form onFinish={handleSubmit}>
      <Form.Item name="title"><Input placeholder="标题" /></Form.Item>
      <Form.Item name="content"><TextArea placeholder="内容" /></Form.Item>
      <Button htmlType="submit" loading={loading}>发布</Button>
      {error && <ErrorMessage error={error} />}
    </Form>
  );
}
```

#### 乐观更新（Optimistic Update）

```tsx
// 乐观更新：先更新 UI，等服务端确认后再修正
const LIKE_POST = gql`
  mutation LikePost($postId: ID!) {
    likePost(id: $postId) {
      id
      liked
      likeCount
    }
  }
`;

function LikeButton({ post }: { post: Post }) {
  const [likePost] = useMutation(LIKE_POST, {
    variables: { postId: post.id },
    optimisticResponse: {
      likePost: {
        id: post.id,
        liked: true,
        likeCount: post.likeCount + 1,
        __typename: 'Post',
      },
    },
    // 乐观更新失败时回滚
    onCompleted(data) {
      // 服务端返回实际数据，Apollo 自动修正缓存
    },
  });

  return (
    <button onClick={() => likePost()}>
      {post.liked ? '❤️' : '🤍'} {post.likeCount}
    </button>
  );
}
```

#### refetchQueries 策略

```tsx
// 策略一：指定查询重新获取
const [deletePost] = useMutation(DELETE_POST, {
  refetchQueries: [
    { query: GET_POSTS, variables: { filter: {} } },
  ],
  awaitRefetchQueries: true,  // 等待 refetch 完成再 resolve
});

// 策略二：手动更新缓存（更精确，避免不必要的网络请求）
const [deletePost] = useMutation(DELETE_POST, {
  update(cache, { data }) {
    cache.evict({ id: cache.identify(data.deletePost) });
    cache.gc();  // 清理悬挂引用
  },
});

// 策略三：使用 useApolloClient 全局刷新
function useRefetchOnMutation() {
  const client = useApolloClient();

  const refreshAll = () => {
    client.refetchQueries({ include: 'active' });
  };

  return { refreshAll };
}
```

---

### 7. 前端客户端：urql

#### 安装与配置

```typescript
// lib/urql.ts
import { createClient, fetchExchange, cacheExchange, subscriptionExchange } from 'urql';
import { createClient as createWSClient } from 'graphql-ws';
import { offlineExchange } from '@urql/exchange-offline';
import { retryExchange } from '@urql/exchange-retry';
import { authExchange } from '@urql/exchange-auth';

const wsClient = createWSClient({
  url: 'ws://localhost:4000/graphql',
});

export const urqlClient = createClient({
  url: '/graphql',
  exchanges: [
    offlineExchange({
      storage: createLocalStorageStorage(),  // 离线存储
      isOnline: () => navigator.onLine,
    }),
    cacheExchange,
    authExchange({
      getAuth: async () => {
        const token = localStorage.getItem('token');
        return token ? { token } : null;
      },
      addAuthToOperation: (operation, auth) => {
        if (!auth?.token) return operation;
        return {
          ...operation,
          context: {
            ...operation.context,
            fetchOptions: {
              ...operation.context.fetchOptions,
              headers: {
                ...operation.context.fetchOptions?.headers,
                authorization: `Bearer ${auth.token}`,
              },
            },
          },
        };
      },
    }),
    retryExchange({ maxDelayMs: 5000, maxNumberAttempts: 3 }),
    fetchExchange,
    subscriptionExchange({
      forwardSubscription: (operation) => ({
        subscribe: (sink) => ({
          unsubscribe: wsClient.subscribe(operation, sink),
        }),
      }),
    }),
  ],
});
```

#### Exchange 机制

```
请求流: Operation → [Exchange Chain] → HTTP/WebSocket
响应流: Result  ← [Exchange Chain] ← HTTP/WebSocket

Exchange 执行顺序（从左到右）：
[dedup] → [cache] → [fetch] → [subscription]
  ↓ 去重    ↓ 缓存    ↓ 网络请求  ↓ WebSocket
```

| Exchange | 作用 |
|----------|------|
| `dedupExchange` | 去重，同一请求只发一次 |
| `cacheExchange` | 内置缓存（文档缓存） |
| `fetchExchange` | 发送 HTTP 请求 |
| `subscriptionExchange` | WebSocket 订阅 |
| `authExchange` | 认证 Token 注入 |
| `retryExchange` | 失败重试 |
| `offlineExchange` | 离线支持 |
| `persistedFetchExchange` | 持久化查询（APQ） |
| `graphcache` | Graphql 规范化缓存 |

```tsx
// urql 使用示例
import { useQuery, useMutation, gql } from 'urql';

const GET_USER_QUERY = gql`
  query GetUser($id: ID!) {
    user(id: $id) { id name email avatar }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const [result, reexecuteQuery] = useQuery({
    query: GET_USER_QUERY,
    variables: { id: userId },
    requestPolicy: 'cache-and-network',  // 类似 Apollo 的 fetchPolicy
  });

  const { data, fetching, error, stale } = result;
  // stale = 正在后台刷新但缓存数据仍可用

  if (fetching && !data) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;

  return (
    <div>
      <h1>{data.user.name}</h1>
      <Button onClick={() => reexecuteQuery({ requestPolicy: 'network-only' })}>
        强制刷新
      </Button>
    </div>
  );
}
```

```tsx
// urql + Graphcache（规范化缓存，类似 Apollo InMemoryCache）
import { cacheExchange } from '@urql/exchange-graphcache';

const graphCacheExchange = cacheExchange({
  keys: {
    User: (data) => data.id as string,
    Post: (data) => data.id as string,
  },
  resolvers: {
    Query: {
      posts: simplePagination({ offsetArgument: 'skip', limitArgument: 'limit' }),
    },
  },
  updates: {
    Mutation: {
      createPost: (result, args, cache) => {
        cache.updateQuery({ query: GET_POSTS }, (data) => {
          if (!data) return data;
          return {
            ...data,
            posts: {
              ...data.posts,
              items: [result.createPost, ...data.posts.items],
              total: data.posts.total + 1,
            },
          };
        });
      },
      deletePost: (result, args, cache) => {
        cache.invalidate({ __typename: 'Post', id: args.id });
      },
    },
  },
  optimistic: {
    likePost: (variables, cache) => ({
      __typename: 'Post',
      id: variables.postId,
      liked: true,
      likeCount: 0, // 占位，后续服务端覆盖
    }),
  },
});
```

---

### 8. 代码生成（GraphQL Code Generator）

#### 配置

```yaml
# codegen.ts
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: './schema.graphql',
  documents: ['src/**/*.tsx', 'src/**/*.ts'],
  ignoreNoDocuments: true,
  generates: {
    // 从 Schema 生成 TypeScript 类型
    'src/__generated__/types.ts': {
      plugins: [
        'typescript',
        'typescript-operations',
        'typescript-react-apollo',
      ],
      config: {
        withHooks: true,            // 自动生成 Hooks
        withComponent: false,       // 不生成组件（推荐用 Hooks）
        withHOC: false,             // 不生成 HOC
        apolloClientVersion: 3,
        reactApolloVersion: 3,
        enumsAsTypes: true,         // 枚举生成为类型而非枚举对象
        immutableTypes: true,       // 所有属性 readonly
        scalars: {
          DateTime: 'string',
          JSON: 'Record<string, unknown>',
        },
        namingConvention: {
          enumValues: 'keep',       // 枚举值保持原样
        },
      },
    },
    // 生成可能的类型（用于联合类型/接口的类型守卫）
    'src/__generated__/possibleTypes.ts': {
      plugins: ['fragment-matcher'],
    },
  },
};

export default config;
```

#### 自动生成的类型与 Hooks

```graphql
# src/graphql/queries/user.graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    avatar
    role
    posts(pagination: { pageSize: 10 }) {
      items {
        id
        title
        createdAt
      }
      total
    }
  }
}

mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    id
    name
    email
    avatar
  }
}
```

```typescript
// 自动生成：src/__generated__/types.ts
// 以下为代码生成器自动产出的内容（无需手写）

export type GetUserQueryVariables = Exact<{
  id: Scalars['ID']['input'];
}>;

export type GetUserQuery = {
  readonly user: {
    readonly id: string;
    readonly name: string;
    readonly email: string;
    readonly avatar: string;
    readonly role: UserRole;
    readonly posts: {
      readonly items: ReadonlyArray<{
        readonly id: string;
        readonly title: string;
        readonly createdAt: string;
      }>;
      readonly total: number;
    };
  };
};

export type UpdateUserMutationVariables = Exact<{
  id: Scalars['ID']['input'];
  input: UpdateUserInput;
}>;

export type UpdateUserMutation = {
  readonly updateUser: {
    readonly id: string;
    readonly name: string;
    readonly email: string;
    readonly avatar: string;
  };
};

// 自动生成的 Hooks
export function useGetUserQuery(
  baseOptions: Apollo.QueryHookOptions<GetUserQuery, GetUserQueryVariables>,
) {
  return Apollo.useQuery<GetUserQuery, GetUserQueryVariables>(
    GetUserDocument,
    baseOptions,
  );
}

export function useUpdateUserMutation() {
  return Apollo.useMutation<UpdateUserMutation, UpdateUserMutationVariables>(
    UpdateUserDocument,
  );
}
```

```tsx
// 前端直接使用生成的 Hooks，类型完全安全
import { useGetUserQuery, useUpdateUserMutation } from '@/__generated__/types';

function UserEditor({ userId }: { userId: string }) {
  const { data, loading } = useGetUserQuery({ variables: { id: userId } });
  const [updateUser, { loading: updating }] = useUpdateUserMutation();

  const handleSave = async (values: { name: string; email: string }) => {
    await updateUser({
      variables: { id: userId, input: values },
      // TypeScript 会校验 input 的字段类型
    });
  };

  // data.user 的类型完全推导，无需手写 interface
  return (
    <div>
      <h1>{data?.user.name}</h1>
      <EditForm
        initialValues={{ name: data?.user.name, email: data?.user.email }}
        onSave={handleSave}
        loading={updating}
      />
    </div>
  );
}
```

---

### 9. Schema Stitching 与 Federation

#### Schema Stitching

```typescript
// 将多个独立 GraphQL Schema 合并为一个统一 Schema
import { stitchSchemas } from '@graphql-tools/stitch';
import { makeExecutableSchema } from '@graphql-tools/schema';
import { delegateToSchema } from '@graphql-tools/delegate';

// 用户服务 Schema
const userSchema = makeExecutableSchema({
  typeDefs: gql`
    type User {
      id: ID!
      name: String!
      email: String!
    }
    type Query {
      user(id: ID!): User
      users: [User!]!
    }
  `,
  resolvers: userResolvers,
});

// 订单服务 Schema
const orderSchema = makeExecutableSchema({
  typeDefs: gql`
    type Order {
      id: ID!
      userId: ID!
      productName: String!
      amount: Float!
      status: String!
    }
    type Query {
      order(id: ID!): Order
      ordersByUser(userId: ID!): [Order!]!
    }
  `,
  resolvers: orderResolvers,
});

// 远程 Schema（通过 HTTP 代理）
const remoteProductSchema = await introspectSchema(httpExecutor);
const productSchema = wrapSchema({ schema: remoteProductSchema, executor: httpExecutor });

// Stitching：合并 + 扩展
const gatewaySchema = stitchSchemas({
  subschemas: [userSchema, orderSchema, productSchema],
  typeDefs: gql`
    # 扩展 User 类型，添加订单字段
    extend type User {
      orders: [Order!]!
    }
  `,
  resolvers: {
    User: {
      orders: {
        selectionSet: `{ id }`,
        resolve: (user, args, context, info) => {
          // 委托给订单服务查询
          return delegateToSchema({
            schema: orderSchema,
            operation: 'query',
            fieldName: 'ordersByUser',
            args: { userId: user.id },
            context,
            info,
          });
        },
      },
    },
  },
});
```

#### Apollo Federation

```typescript
// ===== 用户服务（独立部署） =====
// services/user-service/schema.ts
import { buildFederatedSchema } from '@apollo/federation';

const typeDefs = gql`
  type User @key(fields: "id") {
    id: ID!
    name: String!
    email: String!
    avatar: String!
  }

  extend type Query {
    user(id: ID!): User
    users: [User!]!
  }
`;

const resolvers = {
  User: {
    // @key 指令的解析器：其他服务通过 id 引用 User 时调用
    __resolveReference: async ({ id }: { id: string }) => {
      return userService.getUser(id);
    },
  },
  Query: {
    user: async (_, { id }) => userService.getUser(id),
    users: async () => userService.getUsers(),
  },
};

const server = new ApolloServer({
  schema: buildFederatedSchema([{ typeDefs, resolvers }]),
});
```

```typescript
// ===== 订单服务（独立部署） =====
// services/order-service/schema.ts
const typeDefs = gql`
  type Order @key(fields: "id") {
    id: ID!
    userId: ID!
    productName: String!
    amount: Float!
    status: String!
    # 跨服务关联 User
    user: User
  }

  extend type User @key(fields: "id") {
    id: ID! @external
    orders: [Order!]!
  }

  extend type Query {
    order(id: ID!): Order
    orders: [Order!]!
  }
`;

const resolvers = {
  Order: {
    __resolveReference: async ({ id }: { id: string }) => {
      return orderService.getOrder(id);
    },
    user: (order) => ({ __typename: 'User', id: order.userId }),
  },
  User: {
    orders: (user) => orderService.getOrdersByUser(user.id),
  },
  Query: {
    order: async (_, { id }) => orderService.getOrder(id),
    orders: async () => orderService.getOrders(),
  },
};
```

```typescript
// ===== 网关（Gateway） =====
// gateway/index.ts
import { ApolloGateway } from '@apollo/gateway';
import { ApolloServer } from '@apollo/server';

const gateway = new ApolloGateway({
  serviceList: [
    { name: 'user', url: 'http://user-service:4001/graphql' },
    { name: 'order', url: 'http://order-service:4002/graphql' },
    { name: 'product', url: 'http://product-service:4003/graphql' },
  ],
  // 动态更新服务列表（生产环境推荐）
  // serviceHealthCheck: true,
  // pollIntervalInMs: 10000,  // 每 10s 轮询一次 Schema 变更
});

const server = new ApolloServer({
  gateway,
  subscriptions: {
    'graphql-ws': true,
  },
});
```

**Schema Stitching vs Federation：**

| 维度 | Schema Stitching | Apollo Federation |
|------|-----------------|-------------------|
| **架构** | 中心化（网关组合 Schema） | 去中心化（各服务声明 @key） |
| **所有权** | 网关拥有组合逻辑 | 各服务自己声明扩展 |
| **新增服务** | 需修改网关配置 | 只需部署新服务 + 更新服务列表 |
| **耦合度** | 高（网关需知道所有细节） | 低（服务自治） |
| **学习曲线** | 低（概念简单） | 中（@key/@external/@requires 等指令） |
| **运行时** | 任意 GraphQL 服务器 | Apollo 生态 |
| **适用规模** | 中小型、少量服务 | 大型微服务架构 |
| **调试** | 网关层集中调试 | 分布式追踪 |

---

### 10. 与 BFF 结合

GraphQL 天然适合作为 BFF 层，前端通过 Schema 按需获取数据，BFF 层聚合后端微服务。

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Web 端  │   │  App 端  │   │  小程序  │
└────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │
     └──────────────┼──────────────┘
                    ▼
           ┌─────────────────┐
           │  GraphQL BFF    │
           │  ┌───────────┐  │
           │  │  Schema    │  │  ← 前端定义需要什么
           │  │  Resolver  │  │  ← BFF 聚合后端数据
           │  │  DataLoader│  │  ← 批量查询优化
           │  └───────────┘  │
           └────────┬────────┘
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│用户服务 │  │订单服务 │  │商品服务 │
└─────────┘  └─────────┘  └─────────┘
```

```typescript
// bff/graphql/schema.ts — 为前端定制的 Schema
export const typeDefs = gql`
  # 首页面板数据
  type HomeDashboard {
    banners: [Banner!]!
    recommendProducts: [Product!]!
    hotCategories: [Category!]!
    flashSale: FlashSale
  }

  type Banner {
    id: ID!
    imageUrl: String!
    linkUrl: String!
    title: String!
  }

  type FlashSale {
    endTime: DateTime!
    products: [FlashSaleProduct!]!
  }

  type FlashSaleProduct {
    productId: ID!
    name: String!
    originalPrice: Float!
    salePrice: Float!
    remaining: Int!
  }

  type Query {
    # 首页数据：一个查询获取所有
    homeDashboard: HomeDashboard!
    # 用户相关
    me: User!
    userDashboard: UserDashboard!
  }
`;
```

```typescript
// bff/graphql/resolvers.ts — BFF 聚合逻辑
export const resolvers = {
  Query: {
    homeDashboard: async (_: any, __: any, context: any) => {
      const { userId } = context;

      // 并行请求多个后端服务
      const [banners, products, categories, flashSale] = await Promise.allSettled([
        context.dataSources.promotionService.getBanners(),
        context.dataSources.productService.getRecommendations(userId, { limit: 20 }),
        context.dataSources.categoryService.getHotCategories(),
        context.dataSources.promotionService.getFlashSale(),
      ]);

      return {
        banners: banners.status === 'fulfilled' ? banners.value : [],
        recommendProducts: products.status === 'fulfilled' ? products.value : [],
        hotCategories: categories.status === 'fulfilled' ? categories.value : [],
        flashSale: flashSale.status === 'fulfilled' ? flashSale.value : null,
      };
    },
  },

  HomeDashboard: {
    // 个性化推荐：基于用户画像
    recommendProducts: async (parent: any, __: any, context: any) => {
      if (parent.recommendProducts.length > 0) return parent.recommendProducts;
      // 降级：通用热门商品
      return context.dataSources.productService.getHotProducts({ limit: 20 });
    },
  },
};
```

---

### 11. Relay 规范

Relay 是 Facebook 开发的 GraphQL 客户端，定义了一套严格的规范（Relay Specification），已成为社区事实标准。

#### 连接模型（Connection）

```graphql
# Relay 风格的分页查询
type Query {
  posts(
    first: Int       # 获取前 N 条
    after: String    # 游标（上一页最后一条的 cursor）
    last: Int        # 获取后 N 条
    before: String   # 游标（下一页第一条的 cursor）
  ): PostConnection!
}

type PostConnection {
  edges: [PostEdge!]!     # 边列表
  pageInfo: PageInfo!     # 分页信息
  totalCount: Int!        # 总数
}

type PostEdge {
  node: Post!             # 实际数据
  cursor: String!         # 游标（Base64 编码的唯一标识）
}

type PageInfo {
  hasNextPage: Boolean!   # 是否有下一页
  hasPreviousPage: Boolean!  # 是否有上一页
  startCursor: String     # 第一条游标
  endCursor: String       # 最后一条游标
}
```

```graphql
# 使用示例
query GetPosts($first: Int!, $after: String) {
  posts(first: $first, after: $after) {
    edges {
      node {
        id
        title
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}
```

#### 全局 ID

```typescript
// Relay 要求每个对象有全局唯一 ID（跨类型）
// 格式：base64(typeName:id)
function toGlobalId(typeName: string, id: string): string {
  return Buffer.from(`${typeName}:${id}`).toString('base64');
}

function fromGlobalId(globalId: string): { type: string; id: string } {
  const decoded = Buffer.from(globalId, 'base64').toString();
  const [type, id] = decoded.split(':');
  return { type, id };
}

// 示例
// User:u1 → VXNlcjp1MQ==
// Post:p1 → UG9zdDpwMQ==

const resolvers = {
  User: {
    id: (parent) => toGlobalId('User', parent.id),
  },
  Post: {
    id: (parent) => toGlobalId('Post', parent.id),
  },
  Query: {
    node: async (_: any, { id }: { id: string }) => {
      const { type, id: localId } = fromGlobalId(id);
      switch (type) {
        case 'User': return userService.getUser(localId);
        case 'Post': return postService.getPost(localId);
        default: return null;
      }
    },
  },
};
```

#### Relay Compiler

```typescript
// Relay Compiler 在构建时编译查询，生成优化后的 artifacts
// relay.config.json
{
  "src": "./src",
  "schema": "./schema.graphql",
  "language": "typescript",
  "artifactDirectory": "./src/__generated__"
}

// package.json
{
  "scripts": {
    "relay": "relay-compiler",
    "codegen": "graphql-codegen"
  }
}
```

```tsx
// Relay 客户端使用
import { graphql, usePaginationFragment } from 'react-relay';

function PostList({ viewer }: any) {
  const { data, loadNext, isLoadingNext, hasNext } = usePaginationFragment(
    graphql`
      fragment PostList_viewer on Query
      @argumentDefinitions(
        count: { type: "Int", defaultValue: 10 }
        cursor: { type: "String" }
      )
      @refetchable(queryName: "PostListPaginationQuery") {
        posts(first: $count, after: $cursor)
        @connection(key: "PostList_posts") {
          edges {
            node {
              id
              title
              createdAt
            }
          }
        }
      }
    `,
    viewer,
  );

  return (
    <div>
      {data.posts.edges.map(({ node }) => (
        <PostCard key={node.id} post={node} />
      ))}
      {hasNext && (
        <Button
          loading={isLoadingNext}
          onClick={() => loadNext(10)}
        >
          加载更多
        </Button>
      )}
    </div>
  );
}
```

**Apollo vs urql vs Relay 对比：**

| 维度 | Apollo Client | urql | Relay |
|------|--------------|------|-------|
| **体积** | ~35KB gzip | ~5KB gzip | ~25KB gzip |
| **缓存策略** | 规范化缓存 | 可选（文档缓存/Graphcache） | 规范化缓存 + Store |
| **学习曲线** | 中 | 低 | 高 |
| **编译时优化** | 无 | 无 | Relay Compiler |
| **适合项目** | 通用 | 轻量级/移动端 | 大型复杂应用 |
| **分页** | 手动 + fetchMore | 手动 | Connection 规范 |
| **离线支持** | 需额外方案 | offlineExchange | 需额外方案 |
| **SSR** | 支持 | 支持 | 支持 |

---

## 常见问题

| # | 问题 | 原因 | 解决方案 |
|---|------|------|----------|
| 1 | **N+1 查询** | Resolver 逐条加载关联数据 | DataLoader 批量加载 + 请求级缓存 |
| 2 | **缓存复杂性** | 单个端点、查询字符串多变 | 规范化缓存（按类型+ID 存储）、持久化查询 |
| 3 | **文件上传** | GraphQL 规范不原生支持 | multipart 请求规范（`graphql-upload`）、独立上传接口 |
| 4 | **查询深度攻击** | 恶意深层嵌套查询耗尽服务器 | 查询深度限制（`depthLimit`）、查询复杂度分析 |
| 5 | **速率限制难** | 不同查询复杂度差异大 | 基于查询复杂度的速率限制（`graphql-cost-analysis`） |
| 6 | **错误处理粒度** | 部分字段失败时整体是否报错 | `errors` 数组 + `data` 并存，`null` 标记失败字段 |
| 7 | **Schema 演进** | 字段删除/改名影响前端 | `@deprecated` 标记、渐进式迁移、Schema 注册中心 |
| 8 | **性能监控** | 单端点无法按传统 HTTP 监控 | Apollo Studio / GraphQL Explorer 按操作名监控 |

### 安全措施详解

```typescript
// 1. 查询深度限制
import depthLimit from 'graphql-depth-limit';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)],  // 最大深度 5 层
});

// 2. 查询复杂度限制
import { createComplexityLimitRule } from 'graphql-validation-complexity';

const complexityLimit = createComplexityLimitRule(1000, {
  onCost: (cost) => console.log(`Query cost: ${cost}`),
  formatErrorMessage: (cost) => `Query complexity ${cost} exceeds limit`,
});

const server = new ApolloServer({
  validationRules: [complexityLimit],
});

// 3. 速率限制（基于复杂度）
import rateLimit from 'express-rate-limit';

const graphqlLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  message: '请求过于频繁，请稍后再试',
});

app.use('/graphql', graphqlLimiter);

// 4. 查询白名单（生产环境禁用 Introspection）
const server = new ApolloServer({
  introspection: process.env.NODE_ENV !== 'production',  // 生产环境关闭
  validationRules: process.env.NODE_ENV === 'production'
    ? [noIntrospection]  // 禁止 Schema 内省查询
    : [],
});

// 5. 持久化查询（APQ）— 只允许预注册的查询
import { createPersistedQueryLink } from '@apollo/client/link/persisted-queries';

const persistedLink = createPersistedQueryLink({
  generateHash: (document) => sha256(print(document)),
  useGETForHashedQueries: true,  // 允许 GET 请求便于 CDN 缓存
});
```

### 文件上传方案

```typescript
// 服务端：graphql-upload
import { GraphQLUpload } from 'graphql-upload';

const typeDefs = gql`
  scalar Upload

  type File {
    url: String!
    filename: String!
    mimetype: String!
    size: Int!
  }

  type Mutation {
    singleUpload(file: Upload!): File!
    multipleUpload(files: [Upload!]!): [File!]!
  }
`;

const resolvers = {
  Upload: GraphQLUpload,
  Mutation: {
    singleUpload: async (_: any, { file }: { file: any }) => {
      const { createReadStream, filename, mimetype, encoding } = await file;
      const stream = createReadStream();

      // 上传到 OSS/S3
      const result = await ossClient.putStream(
        `uploads/${Date.now()}-${filename}`,
        stream,
      );

      return {
        url: result.url,
        filename,
        mimetype,
        size: result.size,
      };
    },
  },
};
```

```tsx
// 前端上传组件
import { useMutation, gql } from '@apollo/client';

const UPLOAD_FILE = gql`
  mutation UploadFile($file: Upload!) {
    singleUpload(file: $file) {
      url
      filename
    }
  }
`;

function FileUploader() {
  const [upload] = useMutation(UPLOAD_FILE);

  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    await upload({
      variables: { file },
      context: {
        headers: { 'Apollo-Require-Preflight': 'true' },  // 允许 multipart
      },
    });
  };

  return <input type="file" onChange={handleChange} />;
}
```

---

## 面试题

### 1. GraphQL 的核心概念是什么？它与 REST 最大的区别是什么？

GraphQL 的核心概念包含：**Schema**（强类型定义数据模型）、**Query/Mutation/Subscription**（三种操作类型）、**Resolver**（解析器，每个字段的获取函数）、**DataLoader**（批量加载解决 N+1）。与 REST 最大的区别是**数据获取方式的反转**：REST 由服务端决定返回什么数据（固定接口结构），GraphQL 由客户端决定获取什么数据（声明式查询）。这解决了 REST 的过度获取和不足获取问题——前端只请求需要的字段，一个请求获取多种关联数据。此外 GraphQL 只有一个端点，通过 Schema 自文档化，新增字段不需要新接口。

### 2. N+1 问题是什么？GraphQL 中如何解决？

N+1 问题是指在 GraphQL 查询列表数据时，先执行 1 次查询获取列表，然后对列表中每条记录的关联字段各执行 1 次查询，导致 N+1 次数据库/网络请求。例如查询 100 篇文章及其作者，会执行 1 次查文章 + 100 次查作者 = 101 次查询。解决方案是 **DataLoader**：它通过两个机制优化——**批量加载**（Batch），将同一个事件循环 tick 内的所有 `load(key)` 调用合并为一次 `batchLoad(keys)` 批量查询；**请求级缓存**（Cache），同一请求周期内对同一 key 只加载一次。使用后 101 次查询降为 2 次（1 次文章 + 1 次批量作者）。DataLoader 必须每个请求创建新实例，避免跨请求缓存污染。

### 3. Apollo Client 的缓存机制是怎么工作的？

Apollo Client 使用**规范化缓存**（Normalized Cache），核心是 `InMemoryCache`。它不是按查询缓存整个响应，而是将响应数据按 `__typename:id` 拆分为独立实体存储。例如查询 `{ user(id: "u1") { id name } }` 和 `{ post(id: "p1") { id author { id name } } }`，缓存中存储的是 `User:u1 → { id, name }` 和 `Post:p1 → { id, author → User:u1 }`。好处是：当 Mutation 更新 `User:u1` 时，所有引用该用户的查询缓存自动更新。缓存策略包括 `cache-first`（优先缓存）、`cache-and-network`（缓存+网络刷新）、`network-only` 等。字段策略通过 `typePolicies` 配置合并方式，分页用 `relayStylePagination`。

### 4. Fragment 的用途是什么？

Fragment 是 GraphQL 中**可复用的字段集合**，三个核心用途：(1) **复用字段选择**——多个查询需要相同的字段组合时，定义一次 fragment 多处引用，避免重复代码；(2) **类型条件查询**——对联合类型或接口使用内联片段 `... on User`，根据类型返回不同字段；(3) **前端组件化**——配合 GraphQL Code Generator，每个组件声明自己需要的 fragment，父组件组合子组件的 fragment 构成完整查询。这种模式叫"Colocation"（数据需求与组件共置），每个组件只关心自己的数据片段，修改组件数据需求不影响其他组件。Apollo 的 `useFragment` hook 支持组件级缓存读取。

### 5. Subscription 的原理是什么？有哪些使用场景？

Subscription 基于 **WebSocket** 实现服务端到客户端的实时数据推送。原理：客户端通过 WebSocket 连接发送 Subscription 操作，服务端用 `PubSub`（发布-订阅模式）维护事件通道，当数据变更时通过 `pubsub.publish(eventName, payload)` 发布事件，所有订阅该事件的客户端收到推送。技术栈上，Apollo 使用 `graphql-ws` 协议（替代旧的 `subscriptions-transport-ws`），客户端用 `GraphQLWsLink` 建立连接，服务端用 `WebSocketServer` + `useServer` 处理。典型场景：实时通知、协作编辑、评论/弹幕推送、在线状态、仪表盘监控、订单状态变更。注意事项：Subscription 不应替代轮询——只在真正需要实时推送时使用，因为 WebSocket 连接占用服务端资源。

### 6. Schema Stitching 和 Federation 的区别是什么？各自适用什么场景？

Schema Stitching 是**中心化**方案，由网关将多个子 Schema 手动组合为一个统一 Schema，网关拥有组合逻辑，需要手动编写 `delegateToSchema` 委托查询，新增服务需修改网关配置。Federation 是**去中心化**方案，各服务通过 `@key` 指令声明实体标识，通过 `@external`/`@requires` 声明跨服务关联，服务自描述扩展关系，网关自动组合。Stitching 适合中小型项目、少量服务、需要精细控制组合逻辑的场景；Federation 适合大型微服务架构、团队自治、服务频繁增减的场景。Federation 学习曲线更高但扩展性更好，是 Apollo 推荐的方案。Stitching 不绑定 Apollo 生态，更灵活。

### 7. GraphQL 有哪些安全措施？

GraphQL 安全面临的特殊挑战是：单端点 + 动态查询 = 传统 HTTP 安全方案失效。核心措施：(1) **查询深度限制**——`graphql-depth-limit` 限制最大嵌套层数（如 5 层），防止深层查询耗尽服务器；(2) **查询复杂度分析**——`graphql-cost-analysis` 计算查询的权重分数，超限则拒绝，替代传统的请求速率限制；(3) **关闭内省**——生产环境禁止 `_schema` 查询，防止攻击者探测 Schema；(4) **持久化查询**——只允许预注册的查询哈希，拒绝任意查询字符串；(5) **速率限制**——基于操作名或复杂度而非 IP；(6) **认证与授权**——`context` 中验证 Token，Resolver 或 Directive 层面做字段级鉴权；(7) **超时控制**——设置查询执行超时和 Resolver 超时。

### 8. 如何在前端项目中实践 GraphQL 的代码生成？

GraphQL Code Generator 从 Schema 和前端查询文档自动生成 TypeScript 类型与 React Hooks，实践流程：(1) 在 `codegen.ts` 中配置 schema 路径（本地文件或远程端点）、documents 路径（`src/**/*.graphql` 或内联 `gql` 标签）、输出插件（`typescript` + `typescript-operations` + `typescript-react-apollo`）；(2) 在 `.graphql` 文件或组件内用 `gql` 编写查询，Codegen 扫描后生成 `types.ts`，包含查询变量类型、响应类型和 `useXxxQuery`/`useXxxMutation` Hooks；(3) 组件直接导入生成的 Hooks 使用，无需手写 interface，类型安全端到端保障；(4) CI 中运行 `graphql-codegen --check` 确保生成文件与源一致。关键配置：`withHooks: true` 生成 Hooks、`enumsAsTypes: true` 枚举用类型代替对象、`immutableTypes: true` 所有属性 readonly、自定义 scalar 映射（`DateTime → string`）。

---

## 相关链接

- [[BFF与Serverless]] — GraphQL 作为 BFF 层的实践
- [[Fetch API与请求模式]] — HTTP 请求基础与 GraphQL 的关系
- [[React生态Router与Query]] — 前端数据获取的生态全景
- [GraphQL 官方规范](https://spec.graphql.org/) — GraphQL 语言规范
- [Apollo 官方文档](https://www.apollographql.com/docs/) — Apollo Server / Client 完整文档
- [urql 官方文档](https://formidable.com/open-source/urql/) — 轻量级 GraphQL 客户端
- [Relay 官方文档](https://relay.dev/) — Facebook 的 GraphQL 客户端
- [GraphQL Code Generator](https://the-guild.dev/graphql/codegen) — 类型生成工具
- [Apollo Federation](https://www.apollographql.com/docs/federation/) — 微服务架构方案
- [DataLoader](https://github.com/graphql/dataloader) — N+1 问题解决方案

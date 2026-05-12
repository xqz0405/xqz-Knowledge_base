---
tags:
  - Node.js
  - GraphQL
  - Apollo
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# GraphQL 与 Apollo

## What — 是什么

> GraphQL 是一种 API 查询语言，客户端按需获取所需字段。Apollo 是最流行的 GraphQL 实现生态，提供 Server/Client/Studio 全套工具链。

**核心概念：**

- **Schema**：类型系统定义，`type Query`/`Mutation`/`Subscription` 定义 API 入口
- **Resolver**：每个字段的解析函数，查询时按 Resolver 图执行
- **Apollo Server**：Node.js GraphQL 服务端，支持 Express/Fastify/Standalone
- **Dataloader**：批量加载和缓存，解决 N+1 查询问题
- **Subscription**：基于 WebSocket 的实时数据推送
- **Schema Stitching/Federation**：多服务 Schema 合并，微服务架构

**关键特性：**

- 客户端精确指定需要的字段，无过度获取/获取不足
- 单一端点（`/graphql`），一次请求获取多个资源
- Schema 即文档，强类型约束
- Dataloader 自动合并同批次对同一数据源的请求

## Why — 为什么

**适用场景：**

- 多端适配：不同客户端需要不同字段
- 复杂关联查询：一次请求获取多层数据
- 实时数据：Subscription 推送变更
- BFF 层：GraphQL 作为聚合层

**对比 REST：**

| 维度 | REST | GraphQL |
|------|------|---------|
| 端点 | 多个（/users, /orders） | 单一（/graphql） |
| 数据量 | 固定（过度/不足获取） | 按需（精确字段） |
| 关联查询 | 多次请求 | 嵌套查询一次完成 |
| 版本管理 | URL 版本 | Schema 演进 |
| 缓存 | HTTP 缓存成熟 | 需 Apollo Cache |

## How — 怎么用

### 代码示例

```javascript
const { ApolloServer } = require('@apollo/server');
const { startStandaloneServer } = require('@apollo/server/standalone');

const typeDefs = `#graphql
    type User {
        id: ID!
        name: String!
        email: String!
        posts: [Post!]!
    }
    type Post {
        id: ID!
        title: String!
        author: User!
    }
    type Query {
        user(id: ID!): User
        posts(limit: Int = 10): [Post!]!
    }
    type Mutation {
        createPost(title: String!, authorId: ID!): Post!
    }
`;

const resolvers = {
    Query: {
        user: (_, { id }) => userService.findById(id),
        posts: (_, { limit }) => postService.findMany(limit)
    },
    Mutation: {
        createPost: (_, { title, authorId }) => postService.create({ title, authorId })
    },
    User: {
        posts: (user) => postService.findByAuthor(user.id) // 关联解析
    },
    Post: {
        author: (post) => userService.findById(post.authorId)
    }
};

const server = new ApolloServer({ typeDefs, resolvers });
const { url } = await startStandaloneServer(server, { listen: { port: 4000 } });
```

**Dataloader 解决 N+1：**

```javascript
const DataLoader = require('dataloader');

const userLoader = new DataLoader(async (ids) => {
    const users = await db.users.findMany({ where: { id: { in: ids } } });
    return ids.map(id => users.find(u => u.id === id));
});

// Resolver 中使用
User: {
    posts: (user, _, { loaders }) => loaders.postLoader.load(user.id)
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| N+1 查询 | 关联字段逐个解析 | Dataloader 批量加载 |
| 查询深度过大 | 恶意嵌套查询 | `graphql-depth-limit` 限制深度 |
| 性能瓶颈 | 复杂查询单线程执行 | Persisted Queries + 查询白名单 |
| 缓存困难 | 单一端点，URL 不变 | Apollo Cache + 细粒度缓存策略 |

### 最佳实践

- 使用 Dataloader 防止 N+1 查询
- 限制查询深度和复杂度
- 生产环境使用 Persisted Queries
- Subscription 用于实时场景

## 面试题

**Q1: GraphQL N+1 问题的原理和解决方案？**
> 查询 `{ users { id posts { title } } }` 时，先查用户列表（1次），再为每个用户查帖子（N次），共 N+1 次查询。Dataloader 解决：收集同一批次的所有 authorId，合并为一次 `WHERE id IN (...)` 查询，再按 ID 分配结果。Dataloader 在一次事件循环 tick 中收集所有 load 调用，下一个 tick 执行批量查询。

**Q2: GraphQL Federation 的作用？**
> Federation 让多个独立 GraphQL 服务组合成一个统一 Schema。每个服务定义自己的类型和 Resolver，通过 `@key` 声明实体的主键字段，网关自动跨服务组装。适合微服务架构——用户服务管理 User 类型，订单服务管理 Order 类型，网关合并对外提供统一 API。

---

**相关链接：**
- [[RESTful API设计]]
- [[BFF与API网关]]
- [[微服务架构]]
- Apollo 文档：https://www.apollographql.com/docs/

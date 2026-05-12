---
tags:
  - Node.js
  - ORM
  - Prisma
  - Sequelize
  - TypeORM
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# ORM 与查询构建器

## What — 是什么

> ORM（对象关系映射）将数据库表映射为编程对象，查询构建器提供链式 API 构建 SQL。Node.js 生态中 Prisma、Sequelize、TypeORM 是三大主流 ORM，Knex 是最流行的查询构建器。

**核心概念：**

- **Prisma**：新一代 ORM，Schema-first 声明式定义模型，类型安全，查询引擎用 Rust 编写
- **Sequelize**：老牌 ORM，Active Record 模式，支持多种数据库，社区成熟
- **TypeORM**：TypeScript 优先，装饰器定义模型，类似 Java Hibernate
- **Knex**：SQL 查询构建器，不封装模型，直接构建 SQL，灵活度高
- **N+1 问题**：查询关联数据时，主查询 1 次子查询 N 次，ORM 默认懒加载易触发

**关键特性：**

- Prisma 的 `prisma generate` 从 Schema 生成类型安全的客户端
- Sequelize 的 `include` 实现 Eager Loading 避免 N+1
- TypeORM 的 `relations` 和 `find` 支持关联查询
- Knex 是其他 ORM 的底层依赖，也可独立使用

## Why — 为什么

**对比选型：**

| 维度 | Prisma | Sequelize | TypeORM | Knex |
|------|--------|-----------|---------|------|
| 类型安全 | 最强（自动生成） | 弱 | 强（装饰器） | 无 |
| 学习曲线 | 低 | 中 | 中 | 低 |
| 性能 | 高（Rust引擎） | 中 | 中 | 高 |
| 灵活度 | 中 | 中 | 中 | 最高 |
| 迁移 | 内置 | 内置 | 内置 | 需 Knex migrations |
| 适合项目 | 新项目TS | 旧项目JS | NestJS项目 | 需要精细控制 |

## How — 怎么用

### 代码示例

```javascript
// Prisma Schema
// schema.prisma
// model User { id Int @id @default(autoincrement()) name String email String @unique posts Post[] }
// model Post { id Int @id @default(autoincrement()) title String author User @relation(fields: [authorId]) authorId Int }

// Prisma 查询
const user = await prisma.user.findUnique({ where: { id: 1 }, include: { posts: true } });
const users = await prisma.user.findMany({ where: { name: { contains: 'Alice' } }, take: 10 });

// Sequelize Eager Loading（避免 N+1）
const users = await User.findAll({ include: [{ model: Post, as: 'posts' }] });

// TypeORM 关联查询
const user = await userRepository.findOne({ where: { id: 1 }, relations: ['posts'] });

// Knex 查询构建器
const users = await knex('users').join('posts', 'users.id', 'posts.author_id')
    .select('users.name', 'posts.title').where('users.active', true).limit(10);
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| N+1 查询 | 懒加载关联 | Eager Loading（include/relations） |
| ORM 生成低效 SQL | 抽象层隐藏细节 | 用 `$queryRaw`/原始 SQL 处理复杂查询 |
| 迁移冲突 | 多人并行开发迁移 | 合并时按时间戳排序，或 Prisma 的迁移基线 |

### 最佳实践

- 新项目优先选 Prisma（类型安全 + 开发体验好）
- 关联查询用 Eager Loading 避免 N+1
- 复杂查询不强制走 ORM，用原始 SQL
- 迁移纳入版本控制，CI 自动执行

## 面试题

**Q1: Prisma、Sequelize、TypeORM 如何选型？**
> Prisma：新项目首选，类型安全最强，Schema 声明式，迁移自动管理，适合 TypeScript 项目。Sequelize：已有 JavaScript 项目迁移，Active Record 模式直观，社区成熟但维护活跃度下降。TypeORM：NestJS 生态深度集成，装饰器风格，适合 Java/Hibernate 背景开发者。Knex：需要精细控制 SQL 或构建自己的查询层时使用。

**Q2: N+1 问题如何解决？**
> 三种方式：① Eager Loading——查询时一次性加载关联数据（Prisma `include`、Sequelize `include`、TypeORM `relations`）；② Dataloader——批量收集同一字段的查询，合并为一次 `WHERE id IN (...)` 查询（GraphQL 场景）；③ 手动优化——先查主表获取 ID 列表，再一次性查关联表 `WHERE author_id IN (...)`，代码中组装。

---

**相关链接：**
- [[MySQL与ORM]]
- [[MongoDB与Mongoose]]
- [[数据库连接与连接池]]
- Prisma 文档：https://www.prisma.io/docs

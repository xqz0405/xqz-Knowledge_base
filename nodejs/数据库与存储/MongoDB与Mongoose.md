---
tags:
  - Node.js
  - MongoDB
  - Mongoose
  - NoSQL
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# MongoDB 与 Mongoose

## What — 是什么

> MongoDB 是面向文档的 NoSQL 数据库，以 BSON 格式存储灵活的 JSON 文档。Mongoose 是 Node.js 中最流行的 MongoDB ODM（对象文档映射），提供 Schema 定义、数据验证、中间件钩子和查询构建能力。

**核心概念：**

- **文档模型**：数据以 BSON 文档存储（类似 JSON 但支持更多类型），无固定 Schema，同一集合文档结构可不同
- **Schema**：Mongoose 的模式定义，约束文档的字段类型、验证规则、默认值、索引等
- **Model**：Schema 编译后的构造函数，提供 CRUD 静态方法和实例方法
- **聚合管道（Aggregation Pipeline）**：`$match`/`$group`/`$sort`/`$lookup`/`$unwind` 等阶段组成的数据处理流水线
- **连接池**：Mongoose 内置连接池管理，默认 100 个连接
- **副本集（Replica Set）**：主节点写入 + 从节点复制，自动故障转移
- **索引**：单字段/复合/文本/地理空间/唯一索引，加速查询

**关键特性：**

- Mongoose 的 `SchemaType` 支持 `required`/`min`/`max`/`enum`/`match`/`validate` 等验证
- 中间件钩子：`pre('save')`/`post('save')`/`pre('remove')` 等，适合密码哈希、审计日志
- 虚拟属性（Virtual）：不存入数据库的计算属性，如 `fullName = firstName + lastName`
- `populate()` 实现类似 SQL JOIN 的关联查询
- 支持事务（需副本集）

## Why — 为什么

**适用场景：**

- 文档结构灵活的业务：CMS、用户配置、日志
- 快速迭代的项目：Schema 灵活，无需迁移
- 大数据量读写：分片集群水平扩展
- 实时分析：聚合管道 + 变更流（Change Stream）

**对比数据库方案：**

| 维度 | MongoDB | MySQL | PostgreSQL |
|------|---------|-------|-----------|
| 数据模型 | 文档（灵活Schema） | 关系表（固定Schema） | 关系表（固定Schema） |
| 事务 | 4.0+ 支持（需副本集） | 原生ACID | 原生ACID |
| 关联查询 | populate（应用层） | JOIN（数据库层） | JOIN（数据库层） |
| 水平扩展 | 原生分片 | 需中间件 | 需中间件 |
| Schema 灵活性 | 高（同一集合不同结构） | 低（ALTER TABLE） | 低（ALTER TABLE） |
| 适用场景 | 文档/日志/快速迭代 | 事务型/结构稳定 | 复杂查询/JSON支持 |

**优缺点：**

- ✅ 优点：
  - Schema 灵活，快速迭代无需迁移
  - 文档模型天然契合 JavaScript 对象
  - 聚合管道功能强大
  - 水平扩展简单（分片）
- ❌ 缺点：
  - 无 JOIN，关联需应用层处理
  - 事务支持不如关系型数据库成熟
  - 内存消耗大（文档级锁→4.0 后改为 WiredTiger 文档级并发）
  - 数据一致性依赖应用层保证

## How — 怎么用

### 安装配置

```bash
npm install mongoose
```

```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/mydb', {
    maxPoolSize: 50,
    minPoolSize: 5,
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000
});

mongoose.connection.on('connected', () => console.log('MongoDB connected'));
mongoose.connection.on('error', (err) => console.error('MongoDB error:', err));
```

### 快速上手

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: { type: String, required: true, trim: true },
    email: { type: String, required: true, unique: true, lowercase: true },
    age: { type: Number, min: 0, max: 150 },
    role: { type: String, enum: ['admin', 'user'], default: 'user' },
    createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);

// CRUD
const user = await User.create({ name: 'Alice', email: 'alice@test.com' });
const found = await User.findById(user._id);
await User.updateOne({ _id: user._id }, { age: 25 });
await User.deleteOne({ _id: user._id });
```

### 代码示例

**Schema 关联 + populate：**

```javascript
const postSchema = new mongoose.Schema({
    title: { type: String, required: true },
    content: String,
    author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    tags: [{ type: String }],
    comments: [{
        text: String,
        author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        createdAt: { type: Date, default: Date.now }
    }]
}, { timestamps: true });

postSchema.index({ title: 'text', content: 'text' }); // 全文索引
postSchema.index({ author: 1, createdAt: -1 }); // 复合索引

const Post = mongoose.model('Post', postSchema);

// populate 关联查询
const posts = await Post.find()
    .populate('author', 'name email')        // 只取 name 和 email
    .populate('comments.author', 'name')
    .sort({ createdAt: -1 })
    .limit(20);

// 嵌套文档操作
await Post.updateOne(
    { _id: postId },
    { $push: { comments: { text: 'Nice post!', author: userId } } }
);
```

**聚合管道：**

```javascript
// 统计每个作者的帖子数和平均评论数
const stats = await Post.aggregate([
    { $match: { createdAt: { $gte: new Date('2026-01-01') } } },
    { $group: {
        _id: '$author',
        postCount: { $sum: 1 },
        avgComments: { $avg: { $size: '$comments' } },
        totalComments: { $sum: { $size: '$comments' } }
    }},
    { $sort: { postCount: -1 } },
    { $lookup: {
        from: 'users',
        localField: '_id',
        foreignField: '_id',
        as: 'author'
    }},
    { $unwind: '$author' },
    { $project: {
        name: '$author.name',
        postCount: 1,
        avgComments: { $round: ['$avgComments', 1] }
    }}
]);

// Change Stream（监听数据变更）
const changeStream = Post.watch();
changeStream.on('change', (change) => {
    console.log('变更类型:', change.operationType); // insert/update/delete
    console.log('完整文档:', change.fullDocument);
});
```

**中间件钩子 + 虚拟属性：**

```javascript
const userSchema = new mongoose.Schema({
    name: String,
    email: String,
    password: String
}, { toJSON: { virtuals: true } });

// pre save 钩子：密码哈希
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 12);
    next();
});

// 实例方法：密码验证
userSchema.methods.comparePassword = function(candidate) {
    return bcrypt.compare(candidate, this.password);
};

// 虚拟属性
userSchema.virtual('displayName').get(function() {
    return `@${this.name}`;
});

// toJSON 时隐藏密码
userSchema.set('toJSON', {
    transform: (doc, ret) => {
        delete ret.password;
        delete ret.__v;
        return ret;
    }
});

// post remove 钩子：级联删除
userSchema.post('findOneAndDelete', async function(doc) {
    if (doc) {
        await Post.deleteMany({ author: doc._id });
    }
});
```

### 性能调优

| 参数 | 默认值 | 调优建议 | 说明 |
|------|--------|----------|------|
| maxPoolSize | 100 | 50-100 | 连接池大小，过高浪费资源 |
| batchSize | 1000 | 100-500 | find 批量获取数，降低内存峰值 |
| 索引 | 按需 | 查询热点建索引 | 复合索引遵循 ESR 规则 |
| lean() | 无 | 只读查询加 lean() | 跳过 Mongoose 文档包装，性能提升 3-5x |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| populate N+1 查询 | 每个文档单独查关联 | 批量 populate 或用 `$lookup` 聚合 |
| 内存溢出 | find 无 limit 加载全部 | 始终加 `limit()`，大结果用 cursor |
| 索引未生效 | 查询条件不匹配索引 | 用 `explain('executionStats')` 检查 |
| 连接泄漏 | 未关闭数据库连接 | 应用退出时 `mongoose.disconnect()` |
| 默认值不生效 | 嵌套对象默认值需要函数 | `default: () => ({})` 而非 `default: {}` |
| 版本号冲突 | 并发更新时 `__v` 不匹配 | 用 `findOneAndUpdate` 绕过 |

### 最佳实践

- 只读查询使用 `lean()` 跳过 Mongoose 文档包装
- 热点查询建立复合索引，遵循 ESR 规则（Equality→Sort→Range）
- 大量数据用 cursor 逐条处理，不用 `find()` 全量加载
- 密码等敏感字段在 `toJSON` 中过滤
- 使用 `timestamps: true` 自动管理 `createdAt`/`updatedAt`
- 生产环境使用副本集，启用事务和故障转移

## 面试题

**Q1: MongoDB 的文档模型与关系型数据库有什么区别？**
> MongoDB 以 BSON 文档为单位存储数据，类似 JSON 但支持 Date/Binary/ObjectId/Decimal128 等类型。区别：① 无固定 Schema——同一集合不同文档可有不同字段（Mongoose 通过 Schema 约束，但 MongoDB 本身不强制）；② 嵌套文档——可在一个文档中嵌套子文档和数组，减少 JOIN；③ 无原生 JOIN——关联需 `populate`（Mongoose 应用层）或 `$lookup`（聚合管道）；④ 文档级并发——WiredTiger 引擎支持文档级 MVCC；⑤ 16MB 文档大小限制。

**Q2: Mongoose 的 Schema、Model、Document 之间是什么关系？**
> `Schema` 定义文档的结构和约束（字段类型、验证、索引、钩子），是蓝图。`Model` 是 Schema 编译后的构造函数（`mongoose.model('User', userSchema)`），提供静态 CRUD 方法（`find`/`create`/`updateOne`）。`Document` 是 Model 的实例（`new User({...})` 或 `find()` 返回的对象），拥有实例方法、验证和修改追踪。关系：Schema → 编译 → Model → 实例化 → Document。

**Q3: populate 的原理是什么？与 SQL JOIN 有什么区别？**
> `populate` 在应用层模拟 JOIN。原理：① 查询主文档获取 `author` 字段的 ObjectId 值；② 用这些 ID 去关联集合查询完整文档；③ 将结果合并到主文档的 `author` 字段。与 SQL JOIN 区别：① populate 是两次独立查询（N+1 问题），JOIN 是一次查询；② populate 可跨数据库实例，JOIN 仅限同一数据库；③ populate 灵活（可选字段/条件），JOIN 性能更好。批量 populate 会优化为一次查询所有关联 ID。

**Q4: 聚合管道的常用阶段有哪些？**
> 常用阶段：① `$match`——过滤文档（尽早放前面利用索引）；② `$group`——分组聚合（`$sum`/`$avg`/`$max`/`$min`/`$push`）；③ `$sort`——排序；④ `$limit`/`$skip`——分页；⑤ `$project`——字段选择和重命名；⑥ `$lookup`——左外连接（类似 LEFT JOIN）；⑦ `$unwind`——展开数组（每个元素生成一条文档）；⑧ `$addFields`——添加计算字段；⑨ `$count`——计数；⑩ `$facet`——多管道并行（适合仪表盘统计）。优化原则：`$match` 和 `$project` 尽早放，减少后续处理量。

**Q5: Mongoose 中间件（钩子）有哪些类型？**
> 两种类型：① 文档中间件——`init`/`validate`/`save`/`remove`，`this` 指向文档实例，适合密码哈希、审计日志；② 查询中间件——`count`/`deleteMany`/`findOne`/`findOneAndDelete`/`findOneAndUpdate`/`update`/`updateOne`，`this` 指向 Query 对象，适合软删除、查询条件注入。`pre` 在操作前执行，`post` 在操作后执行。异步钩子用 `async function` + `next()` 或返回 Promise。注意：`updateOne`/`deleteMany` 等不触发 `save` 钩子。

**Q6: MongoDB 索引有哪些类型？如何优化？**
> 索引类型：① 单字段索引——`{ name: 1 }`；② 复合索引——`{ author: 1, createdAt: -1 }`，遵循 ESR 规则（Equality → Sort → Range）；③ 文本索引——`{ title: 'text', content: 'text' }`，支持全文搜索；④ 地理空间索引——`2dsphere`/`2d`，支持附近查询；⑤ 唯一索引——`unique: true`；⑥ TTL 索引——自动删除过期文档；⑦ 哈希索引——分片键。优化：用 `explain('executionStats')` 检查是否命中索引；避免全表扫描（COLLSCAN）；复合索引顺序按 ESR 排列；覆盖查询（covered query）不需要获取文档本身。

**Q7: MongoDB 副本集的工作原理？**
> 副本集由 1 个主节点（Primary）+ N 个从节点（Secondary）+ 可选仲裁节点（Arbiter）组成。写入只能到主节点，主节点将操作记录到 oplog，从节点异步拉取 oplog 重放。自动故障转移：心跳检测（每 2 秒），主节点不可达时从节点发起选举（Raft 协议变体），多数派投票选出新主节点。读偏好（Read Preference）：`primary`（默认，强一致）、`primaryPreferred`、`secondary`（读从节点，减轻主节点压力）、`secondaryPreferred`、`nearest`（延迟最低）。

**Q8: Mongoose 中 lean() 的作用和适用场景？**
> `lean()` 让查询返回纯 JavaScript 对象而非 Mongoose Document。好处：① 内存占用低 50%+（无 Mongoose 内部状态/钩子/修改追踪）；② 查询速度快 3-5 倍（跳过文档实例化）；③ 可自由修改返回对象（Document 需 `markModified`）。适用场景：只读数据展示（列表页/详情页/API 响应）、大量数据导出、不需要 `save()`/`populate()`/虚拟属性/钩子的场景。不适合：需要修改并 `save()`、需要虚拟属性、需要文档中间件。

---

**相关链接：**
- [[MySQL与ORM]]
- [[Redis与缓存策略]]
- [[ORM与查询构建器]]
- Mongoose 文档：https://mongoosejs.com/

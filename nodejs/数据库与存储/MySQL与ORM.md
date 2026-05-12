---
tags:
  - Node.js
  - MySQL
  - ORM
  - Sequelize
  - Prisma
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# MySQL与ORM

## What — 是什么

> MySQL 是最流行的开源关系型数据库，在 Node.js 生态中通过 mysql2 驱动直连或通过 Sequelize/Prisma 等 ORM 框架进行操作。ORM 将数据库表映射为对象模型，用 JavaScript 代码替代 SQL 语句，提升开发效率和可维护性。

**核心概念：**

- **mysql2 驱动**：Node.js 连接 MySQL 的高性能驱动，支持 Promise、预编译语句（Prepared Statement）、连接池，是 mysql 驱动的增强替代品
- **连接池（Connection Pool）**：预先创建一组数据库连接并复用，避免频繁建立/断开 TCP 连接的开销，是生产环境的必选项
- **事务（Transaction）**：将多个 SQL 操作打包为原子单元，满足 ACID 特性（原子性、一致性、隔离性、持久性），确保数据完整性
- **Sequelize**：基于 Promise 的 Node.js ORM，支持 MySQL/PostgreSQL/SQLite/SQL Server，提供模型定义、关联、迁移、钩子等完整功能
- **Prisma**：新一代 ORM，使用 Schema 声明式定义模型，通过 Prisma Client 生成类型安全的查询 API，支持自动补全和类型推断
- **查询优化**：通过索引设计、查询分析（EXPLAIN）、批量操作、避免 N+1 查询等手段提升数据库性能
- **迁移（Migration）**：版本化管理数据库结构变更，类似 Git 管理代码，支持升级和回滚
- **索引策略**：B+Tree 索引（默认）、哈希索引、全文索引、联合索引，合理的索引设计是查询性能的关键

**关键特性：**

- mysql2 支持 Promise API 和连接池，性能优于原 mysql 驱动
- Sequelize 支持 4 种关联：hasOne/hasMany/belongsTo/belongsToMany，覆盖所有关系型场景
- Prisma 的查询引擎用 Rust 编写，性能优于纯 JavaScript 实现的 ORM
- Prisma Schema 是声明式的，修改 Schema 后执行 `prisma generate` 自动更新 Client
- 事务隔离级别：READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ（MySQL 默认）→ SERIALIZABLE

**核心架构：**

- mysql2 架构：应用层 → Connection Pool → Connection → MySQL Protocol → MySQL Server
- Sequelize 架构：Model 定义 → QueryBuilder → SQL 生成 → mysql2 执行 → 结果映射
- Prisma 架构：Schema 定义 → Prisma Engine（Rust）→ 查询构建 → SQL 执行 → 结果转换 → Prisma Client
- 事务流程：BEGIN → SQL 操作 → COMMIT / ROLLBACK

## Why — 为什么

**适用场景：**

- **关系型数据**：用户、订单、商品等结构化数据，有明确的关系和约束
- **事务型业务**：支付、库存扣减、账户转账等要求原子性和一致性的操作
- **复杂查询**：多表关联、聚合统计、分组排序，SQL 的天然优势领域
- **数据一致性要求高**：银行、电商等对数据准确性要求极高的业务
- **已有 MySQL 基础设施**：企业已有 MySQL 运维体系和 DBA 团队

**对比数据库操作方案：**

| 维度 | mysql2 原生 | Sequelize | Prisma |
|------|-----------|-----------|--------|
| 学习曲线 | 低（会 SQL 即可） | 中（需理解 ORM 概念） | 中（需学习 Schema 语法） |
| 类型安全 | 无 | 弱（运行时检查） | 强（编译时自动生成类型） |
| 查询灵活性 | 极高（原生 SQL） | 高（支持原生 SQL 混用） | 中（受限 Client API） |
| 性能 | 最高 | 中（ORM 开销） | 高（Rust 引擎） |
| 迁移管理 | 手动 | 内置（CLI） | 内置（CLI，最佳体验） |
| 关联查询 | 手写 JOIN | 声明式关联 + Eager/Lazy Loading | 声明式关联 + 自动 JOIN |
| 调试难度 | 低（直接看 SQL） | 中（需开启日志看生成 SQL） | 低（Prisma Studio + 查询日志） |

**优缺点：**

- ✅ mysql2 优点：
  - 性能最优，无 ORM 中间层开销
  - 完全控制 SQL，灵活度最高
  - 体积小，依赖少
  - 支持预处理语句，防止 SQL 注入
- ❌ mysql2 缺点：
  - 手写 SQL 工作量大，易出错
  - 无类型提示，拼写错误运行时才发现
  - 手动管理连接池和事务
  - 表结构变更需要手动维护代码
- ✅ Sequelize 优点：
  - 成熟稳定，社区资源丰富
  - 支持多种数据库，切换成本低
  - 内置迁移、种子、钩子等完整功能
  - 丰富的关联查询 API
- ❌ Sequelize 缺点：
  - 类型支持弱，TypeScript 体验差
  - 复杂查询生成的 SQL 可能低效
  - 魔法方法多，调试困难
  - 文档质量参差不齐
- ✅ Prisma 优点：
  - 类型安全，自动补全体验极佳
  - Schema 声明式定义，直观清晰
  - Rust 引擎性能优异
  - 迁移管理体验最佳
  - Prisma Studio 可视化管理数据
- ❌ Prisma 缺点：
  - 复杂查询受限（原始 SQL 兜底）
  - 生成文件体积较大
  - 学习曲线（Schema 语法、查询 API）
  - 部分高级 MySQL 特性不支持

## How — 怎么用

### 安装配置

**MySQL 安装（Windows）：**

```bash
# 方式1：官方安装包
# 下载 MySQL Installer: https://dev.mysql.com/downloads/installer/
# 选择 Server Only 或 Developer Default

# 方式2：Docker（推荐开发环境）
docker run -d \
  --name mysql-dev \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=myapp \
  -p 3306:3306 \
  mysql:8.0

# 验证连接
docker exec -it mysql-dev mysql -uroot -proot123 -e "SELECT VERSION();"
```

**驱动安装：**

```bash
# mysql2 驱动
npm install mysql2

# Sequelize + 驱动
npm install sequelize mysql2

# Prisma
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

**连接池配置：**

```javascript
// mysql2 连接池配置
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: 'root123',
    database: 'myapp',

    // 连接池参数
    connectionLimit: 20,     // 最大连接数
    waitForConnections: true, // 连接用完时是否等待
    queueLimit: 0,           // 等待队列长度（0=无限）
    idleTimeout: 60000,      // 空闲连接超时时间(ms)

    // 性能参数
    enableKeepAlive: true,   // 启用 TCP KeepAlive
    keepAliveInitialDelay: 0,

    // 安全参数
    charset: 'utf8mb4',      // 字符集（支持 emoji）
    timezone: '+08:00',      // 时区
    dateStrings: false        // 返回 Date 对象而非字符串
});
```

### 代码示例

**1. mysql2 原生操作（含事务）：**

```javascript
const mysql = require('mysql2/promise');

async function main() {
    const pool = mysql.createPool({
        host: 'localhost',
        user: 'root',
        password: 'root123',
        database: 'myapp',
        connectionLimit: 10,
        charset: 'utf8mb4'
    });

    // 创建表
    await pool.execute(`
        CREATE TABLE IF NOT EXISTS users (
            id INT AUTO_INCREMENT PRIMARY KEY,
            username VARCHAR(50) NOT NULL UNIQUE,
            email VARCHAR(100) NOT NULL,
            age INT DEFAULT 0,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            INDEX idx_email (email),
            INDEX idx_username_age (username, age)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
    `);

    // 插入数据（预处理语句防 SQL 注入）
    const [result] = await pool.execute(
        'INSERT INTO users (username, email, age) VALUES (?, ?, ?)',
        ['zhangsan', 'zhangsan@example.com', 25]
    );
    console.log('插入ID:', result.insertId);

    // 批量插入
    const [batchResult] = await pool.execute(
        'INSERT INTO users (username, email, age) VALUES (?, ?, ?)',
        ['lisi', 'lisi@example.com', 30]
    );

    // 查询数据
    const [rows] = await pool.execute(
        'SELECT * FROM users WHERE age > ? ORDER BY created_at DESC LIMIT ?',
        [20, 10]
    );
    console.log('查询结果:', rows);

    // 事务操作 — 转账场景
    async function transfer(fromId, toId, amount) {
        const conn = await pool.getConnection();
        try {
            await conn.beginTransaction();

            // 检查余额
            const [accounts] = await conn.execute(
                'SELECT balance FROM accounts WHERE id = ? FOR UPDATE',
                [fromId]
            );
            if (accounts[0].balance < amount) {
                throw new Error('余额不足');
            }

            // 扣款
            await conn.execute(
                'UPDATE accounts SET balance = balance - ? WHERE id = ?',
                [amount, fromId]
            );

            // 到账
            await conn.execute(
                'UPDATE accounts SET balance = balance + ? WHERE id = ?',
                [amount, toId]
            );

            // 记录流水
            await conn.execute(
                'INSERT INTO transactions (from_id, to_id, amount) VALUES (?, ?, ?)',
                [fromId, toId, amount]
            );

            await conn.commit();
            console.log('转账成功');
        } catch (err) {
            await conn.rollback();
            console.error('转账失败，已回滚:', err.message);
            throw err;
        } finally {
            conn.release(); // 归还连接到池
        }
    }

    // 使用 EXPLAIN 分析查询
    const [explain] = await pool.execute(
        'EXPLAIN SELECT * FROM users WHERE email = ?',
        ['zhangsan@example.com']
    );
    console.log('查询计划:', explain);

    await pool.end();
}

main().catch(console.error);
```

**2. Sequelize 模型定义 + CRUD + 关联：**

```javascript
const { Sequelize, DataTypes, Op } = require('sequelize');

// 初始化连接
const sequelize = new Sequelize('myapp', 'root', 'root123', {
    host: 'localhost',
    dialect: 'mysql',
    logging: false, // 生产环境关闭 SQL 日志
    pool: {
        max: 20,
        min: 5,
        idle: 10000,
        acquire: 30000
    },
    define: {
        charset: 'utf8mb4',
        collate: 'utf8mb4_unicode_ci',
        timestamps: true,          // 自动 createdAt/updatedAt
        paranoid: true,            // 软删除（deletedAt）
        underscored: true          // 字段名下划线风格
    }
});

// 模型定义
const User = sequelize.define('User', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    username: {
        type: DataTypes.STRING(50),
        allowNull: false,
        unique: true,
        validate: {
            len: [3, 50],
            isAlphanumeric: true
        }
    },
    email: {
        type: DataTypes.STRING(100),
        allowNull: false,
        validate: { isEmail: true }
    },
    age: {
        type: DataTypes.INTEGER,
        defaultValue: 0,
        validate: { min: 0, max: 150 }
    },
    status: {
        type: DataTypes.ENUM('active', 'inactive', 'banned'),
        defaultValue: 'active'
    }
});

const Post = sequelize.define('Post', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    title: {
        type: DataTypes.STRING(200),
        allowNull: false
    },
    content: {
        type: DataTypes.TEXT,
        allowNull: true
    },
    viewCount: {
        type: DataTypes.INTEGER,
        defaultValue: 0
    }
});

const Tag = sequelize.define('Tag', {
    name: {
        type: DataTypes.STRING(50),
        allowNull: false,
        unique: true
    }
});

// 定义关联
// 一对多：User hasMany Post, Post belongsTo User
User.hasMany(Post, { foreignKey: 'authorId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'authorId', as: 'author' });

// 多对多：Post belongsToMany Tag
Post.belongsToMany(Tag, { through: 'PostTags', as: 'tags' });
Tag.belongsToMany(Post, { through: 'PostTags', as: 'posts' });

// 一对一：User hasOne Profile
const Profile = sequelize.define('Profile', {
    bio: DataTypes.TEXT,
    avatar: DataTypes.STRING
});
User.hasOne(Profile, { foreignKey: 'userId' });
Profile.belongsTo(User, { foreignKey: 'userId' });

// 同步模型到数据库
async function initDB() {
    await sequelize.sync({ alter: true }); // 开发环境：自动同步
    // 生产环境使用迁移：npx sequelize-cli db:migrate
}

// CRUD 操作
async function crudExamples() {
    // 创建
    const user = await User.create({
        username: 'zhangsan',
        email: 'zhangsan@example.com',
        age: 25
    });

    // 查询 — 条件、排序、分页
    const users = await User.findAll({
        where: {
            age: { [Op.gte]: 18 },
            status: 'active'
        },
        order: [['created_at', 'DESC']],
        limit: 10,
        offset: 0,
        attributes: { exclude: ['deleted_at'] } // 排除软删除字段
    });

    // 关联查询 — Eager Loading（解决 N+1）
    const postsWithAuthor = await Post.findAll({
        include: [
            { model: User, as: 'author', attributes: ['id', 'username'] },
            { model: Tag, as: 'tags', through: { attributes: [] } } // 不返回中间表字段
        ],
        where: { viewCount: { [Op.gt]: 100 } }
    });

    // 更新
    await User.update(
        { age: 26, status: 'active' },
        { where: { username: 'zhangsan' } }
    );

    // 删除（软删除，因为 paranoid: true）
    await User.destroy({ where: { username: 'zhangsan' } });

    // 事务操作
    const result = await sequelize.transaction(async (t) => {
        const user = await User.create({
            username: 'lisi', email: 'lisi@example.com', age: 30
        }, { transaction: t });

        await Post.create({
            title: '第一篇文章', content: 'Hello World', authorId: user.id
        }, { transaction: t });

        return user;
    }); // 自动 commit，异常自动 rollback

    // 原生 SQL 混用
    const [rawResults] = await sequelize.query(
        'SELECT u.username, COUNT(p.id) as post_count FROM users u LEFT JOIN posts p ON u.id = p.author_id GROUP BY u.id HAVING post_count > :minCount',
        {
            replacements: { minCount: 5 },
            type: Sequelize.QueryTypes.SELECT
        }
    );
}

// 钩子（Hooks）
User.addHook('beforeCreate', async (user, options) => {
    console.log(`即将创建用户: ${user.username}`);
});

User.afterDestroy(async (user, options) => {
    console.log(`用户已删除: ${user.username}`);
    // 清理关联数据
    await Post.destroy({ where: { authorId: user.id } });
});
```

**3. Prisma Schema + 查询 + 事务：**

```prisma
// schema.prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "mysql"
    url      = env("DATABASE_URL") // mysql://root:root123@localhost:3306/myapp
}

model User {
    id        Int      @id @default(autoincrement())
    username  String   @unique @db.VarChar(50)
    email     String   @db.VarChar(100)
    age       Int      @default(0)
    status    Status   @default(ACTIVE)
    createdAt DateTime @default(now()) @map("created_at")
    updatedAt DateTime @updatedAt @map("updated_at")

    profile  Profile?  // 一对一
    posts    Post[]    // 一对多

    @@index([email])
    @@map("users") // 映射到数据库表名
}

model Post {
    id        Int      @id @default(autoincrement())
    title     String   @db.VarChar(200)
    content   String?  @db.Text
    viewCount Int      @default(0) @map("view_count")
    authorId  Int      @map("author_id")
    createdAt DateTime @default(now()) @map("created_at")
    updatedAt DateTime @updatedAt @map("updated_at")

    author User       @relation(fields: [authorId], references: [id])
    tags   PostTag[]

    @@index([authorId])
    @@index([viewCount])
    @@map("posts")
}

model Tag {
    id   Int    @id @default(autoincrement())
    name String @unique @db.VarChar(50)
    posts PostTag[]

    @@map("tags")
}

model PostTag {
    postId Int @map("post_id")
    tagId  Int @map("tag_id")

    post Post @relation(fields: [postId], references: [id])
    tag  Tag  @relation(fields: [tagId], references: [id])

    @@id([postId, tagId])
    @@map("post_tags")
}

model Profile {
    id     Int    @id @default(autoincrement())
    bio    String? @db.Text
    avatar String? @db.VarChar(255)
    userId Int    @unique @map("user_id")

    user User @relation(fields: [userId], references: [id])

    @@map("profiles")
}

enum Status {
    ACTIVE
    INACTIVE
    BANNED
}
```

```javascript
// Prisma 查询操作
const { PrismaClient, Prisma } = require('@prisma/client');
const prisma = new PrismaClient({
    log: [
        { emit: 'stdout', level: 'query' },  // 开发环境打印 SQL
    ],
});

async function main() {
    // 创建用户
    const user = await prisma.user.create({
        data: {
            username: 'zhangsan',
            email: 'zhangsan@example.com',
            age: 25,
            profile: {
                create: { bio: '全栈开发工程师', avatar: '/avatars/1.jpg' }
            }
        },
        include: { profile: true } // 返回关联的 profile
    });

    // 批量创建
    await prisma.user.createMany({
        data: [
            { username: 'lisi', email: 'lisi@example.com', age: 30 },
            { username: 'wangwu', email: 'wangwu@example.com', age: 28 },
        ]
    });

    // 条件查询 — 等价于 WHERE age >= 18 AND status = 'ACTIVE'
    const adults = await prisma.user.findMany({
        where: {
            age: { gte: 18 },
            status: 'ACTIVE',
        },
        orderBy: { createdAt: 'desc' },
        take: 10,
        skip: 0,
        select: {
            id: true,
            username: true,
            email: true,
            posts: {
                select: { id: true, title: true },
                take: 3,
            }
        }
    });

    // 关联查询 — 自动处理 JOIN，无 N+1 问题
    const postsWithAuthor = await prisma.post.findMany({
        include: {
            author: { select: { id: true, username: true } },
            tags: { include: { tag: true } }
        },
        where: { viewCount: { gt: 100 } }
    });

    // 聚合查询
    const stats = await prisma.user.aggregate({
        _count: { id: true },
        _avg: { age: true },
        _max: { age: true },
        _min: { age: true },
        where: { status: 'ACTIVE' }
    });

    // 分页查询（游标分页，适合大数据量）
    const firstPage = await prisma.post.findMany({
        take: 10,
        orderBy: { id: 'desc' }
    });
    const cursor = firstPage[firstPage.length - 1].id;
    const secondPage = await prisma.post.findMany({
        take: 10,
        skip: 1, // 跳过游标本身
        cursor: { id: cursor },
        orderBy: { id: 'desc' }
    });

    // 更新
    await prisma.user.update({
        where: { username: 'zhangsan' },
        data: { age: 26 }
    });

    // 条件批量更新
    await prisma.user.updateMany({
        where: { status: 'INACTIVE', age: { lt: 18 } },
        data: { status: 'BANNED' }
    });

    // 删除
    await prisma.user.delete({ where: { username: 'wangwu' } });

    // 交互式事务
    const transferResult = await prisma.$transaction(async (tx) => {
        // 创建用户和文章（原子操作）
        const newUser = await tx.user.create({
            data: {
                username: 'zhaoliu',
                email: 'zhaoliu@example.com',
                age: 22
            }
        });

        const newPost = await tx.post.create({
            data: {
                title: '我的第一篇文章',
                content: 'Hello Prisma!',
                authorId: newUser.id,
                tags: {
                    create: [
                        { tag: { connectOrCreate: { where: { name: '入门' }, create: { name: '入门' } } } },
                        { tag: { connectOrCreate: { where: { name: 'Prisma' }, create: { name: 'Prisma' } } } }
                    ]
                }
            },
            include: { tags: { include: { tag: true } } }
        });

        return { user: newUser, post: newPost };
    });

    // 原始 SQL 查询
    const rawUsers = await prisma.$queryRaw`
        SELECT u.username, COUNT(p.id) as post_count
        FROM users u
        LEFT JOIN posts p ON u.id = p.author_id
        GROUP BY u.id
        HAVING post_count > ${5}
    `;

    // 原始 SQL 执行（INSERT/UPDATE/DELETE）
    const updateResult = await prisma.$executeRaw`
        UPDATE users SET status = 'INACTIVE' WHERE age < ${18}
    `;
}

main()
    .catch(console.error)
    .finally(() => prisma.$disconnect());
```

**迁移操作：**

```bash
# Prisma 迁移
npx prisma migrate dev --name init          # 开发环境：创建迁移并应用
npx prisma migrate deploy                    # 生产环境：应用所有待执行迁移
npx prisma migrate status                    # 查看迁移状态
npx prisma migrate reset                     # 重置数据库（危险！）

# Prisma 工具
npx prisma generate                          # 重新生成 Client
npx prisma studio                            # 打开可视化管理界面
npx prisma db push                           # 推送 Schema 到数据库（无迁移文件，原型用）

# Sequelize 迁移
npx sequelize-cli migration:generate --name create-users  # 生成迁移文件
npx sequelize-cli db:migrate                               # 执行迁移
npx sequelize-cli db:migrate:undo                          # 回滚最近一次迁移
npx sequelize-cli db:migrate:undo:all                      # 回滚所有迁移
npx sequelize-cli seed:generate --name demo-users          # 生成种子文件
npx sequelize-cli db:seed:all                              # 执行种子
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| N+1 查询性能差 | 循环中逐条查询关联数据 | 使用 Eager Loading（include）或 DataLoader 批量查询 |
| 连接池耗尽 | 未释放连接或并发超限 | 确保 `conn.release()` 在 finally 中调用，合理设置 connectionLimit |
| Sequelize 时区问题 | 默认 UTC 与业务时区不一致 | 初始化时配置 `timezone: '+08:00'`，数据库设置相同时区 |
| Prisma Schema 改动不生效 | 忘记执行 migrate/generate | 修改 Schema 后执行 `npx prisma migrate dev` + `npx prisma generate` |
| 事务中查询不到刚插入的数据 | 隔离级别导致未提交数据不可见 | 确保在同一事务内操作，或使用合适的隔离级别 |
| 批量插入性能差 | 逐条 INSERT 网络 I/O 开销大 | mysql2 用 `bulkInsert`，Sequelize 用 `bulkCreate`，Prisma 用 `createMany` |
| Sequelize 软删除关联查询异常 | paranoid 模式下关联查询默认排除已删除记录 | 查询时加 `paranoid: false` 包含已删除记录 |
| Prisma 枚举与数据库不匹配 | Schema 枚举定义与数据库实际值不一致 | 使用 `@map` 映射枚举值，或在数据库层统一管理 |

### 性能调优

**连接池参数调优：**

| 参数 | 默认值 | 调优建议 | 说明 |
|------|--------|----------|------|
| `connectionLimit` | 10 | CPU核数 * 2 + 磁盘数 | 最大连接数，过大浪费资源，过小请求排队 |
| `acquireTimeout` | 30000 | 10000-30000 | 获取连接超时时间(ms)，根据业务响应时间调整 |
| `idleTimeout` | 60000 | 30000-60000 | 空闲连接回收时间(ms)，过短频繁重建 |
| `queueLimit` | 0 | 100-500 | 等待队列长度，0=无限，建议限制防止雪崩 |
| `minConnections` | 0 | 5-10 | 最小保持连接数，避免冷启动延迟 |

**查询优化：**

| 优化策略 | 说明 | 示例 |
|----------|------|------|
| 避免 SELECT * | 只查需要的字段，减少网络传输和内存 | `select: { id: true, name: true }` |
| 批量操作替代循环 | 减少网络往返 | `createMany` / `bulkCreate` |
| 合理使用索引 | WHERE/JOIN/ORDER BY 字段建索引 | `@@index([email])` |
| 分页查询 | 大数据量必须分页 | `take + skip` 或游标分页 |
| EXPLAIN 分析 | 定期分析慢查询的执行计划 | `EXPLAIN SELECT ...` |
| 避免 N+1 | 关联查询用 include 预加载 | `include: { author: true }` |

**索引策略：**

| 索引类型 | 适用场景 | 注意事项 |
|----------|----------|----------|
| 主键索引 | 每张表必须有，自增 INT 或 UUID | 自增 INT 性能优于 UUID（B+Tree 顺序插入） |
| 唯一索引 | 唯一约束字段（username、email） | 等价查询 + 唯一约束，兼顾性能和数据完整性 |
| 联合索引 | 多字段组合查询 | 遵循最左前缀原则，区分度高的字段放前面 |
| 覆盖索引 | 查询字段全在索引中 | 避免回表查询，但索引体积增大 |
| 全文索引 | 文本搜索（文章内容、商品描述） | MySQL 8.0 ngram 分词器支持中文 |

**慢查询排查流程：**

```bash
# 1. 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  # 超过1秒记录

# 2. 查看慢查询日志
mysqldumpslow -s t /var/log/mysql/slow.log

# 3. EXPLAIN 分析
EXPLAIN SELECT u.*, COUNT(p.id) FROM users u LEFT JOIN posts p ON u.id = p.author_id GROUP BY u.id;

# 关注字段：type（ALL=全表扫描需优化）、key（使用的索引）、rows（扫描行数）、Extra（Using filesort/temporary 需优化）
```

### 最佳实践

- **连接池必用**：生产环境必须使用连接池，根据负载调整 `connectionLimit`，避免连接泄漏
- **事务最小化**：事务中只放必须原子化的操作，减少锁持有时间，避免长事务
- **索引先行**：根据查询模式设计索引，EXPLAIN 验证效果，避免无索引全表扫描
- **批量操作**：大量数据操作使用 `createMany`/`bulkCreate`，单条操作效率差 10 倍以上
- **迁移版本化**：生产环境数据库变更必须通过迁移，禁止手动改表，确保可追溯可回滚
- **软删除谨慎**：Sequelize 的 paranoid 模式方便但查询复杂，评估是否真的需要
- **环境分离**：开发用 `sync({ alter: true })` 或 `prisma db push`，生产用 `migrate deploy`
- **监控告警**：监控慢查询、连接池使用率、死锁事件，设置合理阈值告警
- **ORM 选型**：新项目优先 Prisma（类型安全+开发体验），已有 Sequelize 项目可继续维护
- **原始 SQL 兜底**：ORM 生成不了的高性能 SQL，直接用 `$queryRaw` / `sequelize.query` 编写

## 面试题

**Q1: mysql2 和 mysql 驱动的区别是什么？为什么推荐 mysql2？**
> mysql2 是 mysql 驱动的增强替代品，核心区别：(1) mysql2 支持 Promise API，可直接 `await pool.execute()`，而 mysql 只有回调模式（需手动 promisify）；(2) mysql2 支持预处理语句（Prepared Statement），通过二进制协议传输参数，既防 SQL 注入又提升重复查询性能（一次解析多次执行）；(3) mysql2 性能更高，使用 C++ 绑定（node-mysql2 的编译部分）；(4) mysql2 支持 JSON 数据类型、Decimal 精确类型等 MySQL 8.0 新特性；(5) mysql2 API 是 mysql 的超集，可直接替换（`require('mysql2')` 替代 `require('mysql')`）。因此新项目一律推荐 mysql2。

**Q2: 数据库连接池的原理是什么？关键参数如何配置？**
> 连接池在应用启动时预先创建 N 个数据库连接，放在池中复用。请求来时从池中获取空闲连接，用完归还而非关闭，避免频繁 TCP 三次握手和 MySQL 认证开销。关键参数：(1) `connectionLimit`（最大连接数）— 推荐 CPU 核数 * 2 + 磁盘数，需小于 MySQL 的 `max_connections`（默认 151）；(2) `waitForConnections`（连接耗尽时是否等待）— 生产环境设 true，避免直接报错；(3) `acquireTimeout`（获取连接超时）— 根据业务响应时间设 10-30 秒；(4) `idleTimeout`（空闲连接回收时间）— 30-60 秒，过长浪费 MySQL 资源，过短频繁重建。注意：必须在 finally 中调用 `conn.release()` 归还连接，否则连接泄漏导致池耗尽。

**Q3: Sequelize 支持哪 4 种关联类型？分别对应什么数据库关系？**
> (1) `hasOne` — 一对一：一个 User 拥有一个 Profile（`User.hasOne(Profile)`），Profile 表存 userId 外键；(2) `hasMany` — 一对多：一个 User 拥有多篇 Post（`User.hasMany(Post)`），Post 表存 authorId 外键；(3) `belongsTo` — 一对一/一对多的反向：Post 属于一个 User（`Post.belongsTo(User)`），Post 表存外键；(4) `belongsToMany` — 多对多：Post 和 Tag 通过中间表关联（`Post.belongsToMany(Tag, { through: 'PostTags' })`），中间表存储两个实体的外键。定义关联时需注意：`belongsTo` 和 `hasOne/hasMany` 是配对使用的，外键默认在 `belongsTo` 一方；`belongsToMany` 必须指定 `through` 中间表。

**Q4: Prisma 的查询引擎原理是什么？为什么性能优于纯 JS 的 ORM？**
> Prisma 的核心是 Prisma Engine（用 Rust 编写），作为 Node.js 和数据库之间的中间层。工作流程：(1) Prisma Client（JS）构建查询对象；(2) 查询对象序列化后通过 IPC 传给 Prisma Engine；(3) Engine 解析查询、优化、生成 SQL；(4) Engine 通过数据库驱动执行 SQL；(5) 结果由 Engine 转换后返回 Client。性能优势的原因：(1) Rust 引擎的 SQL 生成和查询优化比纯 JS 快（无需 V8 JIT 预热）；(2) Engine 内置连接池管理，比 JS 层的连接池更高效；(3) 查询优化器能生成更优 SQL（如自动处理 N+1 为批量 JOIN）；(4) 二进制通信比 JSON 序列化开销更小。代价是 Engine 是独立的二进制文件，部署时需确保平台兼容。

**Q5: 数据库事务的 4 个隔离级别分别是什么？MySQL 默认哪个？**
> (1) READ UNCOMMITTED（读未提交）— 最低级别，可读取其他事务未提交的数据（脏读），基本不用；(2) READ COMMITTED（读已提交）— 只能读取已提交的数据，解决了脏读，但同一事务内两次读取可能结果不同（不可重复读），Oracle/PostgreSQL 默认；(3) REPEATABLE READ（可重复读）— 同一事务内多次读取结果一致，解决了不可重复读，但可能幻读（InnoDB 通过 MVCC + Next-Key Lock 基本解决幻读），MySQL 默认级别；(4) SERIALIZABLE（串行化）— 最高级别，完全串行执行，解决所有并发问题但性能最差。生产环境一般用 REPEATABLE READ（默认）或 READ COMMITTED（需要更高并发时），极少使用 SERIALIZABLE。

**Q6: 什么是 N+1 查询问题？在 Sequelize 和 Prisma 中分别如何解决？**
> N+1 问题：查询 1 次主表获取 N 条记录，再对每条记录查 1 次关联表，共 N+1 次查询。例如查询 10 个用户及其文章：1 次查用户 + 10 次查文章 = 11 次 SQL。Sequelize 解决：(1) Eager Loading — `User.findAll({ include: [Post] })` 一次性 JOIN 查询；(2) 分开查询 + 手动组装 — 先查 User 再批量查 Post。Prisma 解决：(1) `include` 预加载 — `prisma.user.findMany({ include: { posts: true } })`，Prisma 自动优化为批量查询；(2) `select` 指定字段 — 减少不必要的数据传输。核心思路都是将 N+1 次查询减少为 1-2 次批量查询，通过 JOIN 或 IN 子句一次获取所有关联数据。

**Q7: Prisma 和 Sequelize 应该如何选型？**
> 选型维度：(1) TypeScript 支持 — Prisma 自动生成类型，编译时就能发现查询错误；Sequelize 类型需要手动定义或用 `@types/sequelize`，运行时才能发现错误；(2) 开发体验 — Prisma Schema 声明式定义模型，自动补全极佳；Sequelize 用 JS 对象定义模型，选项多但记忆负担大；(3) 查询能力 — Sequelize 支持更复杂的查询（子查询、HAVING、窗口函数等），Prisma 复杂查询受限需用原始 SQL；(4) 迁移管理 — Prisma 的迁移体验更好（自动生成迁移文件、可视化 diff），Sequelize 迁移需要手动编写 up/down；(5) 性能 — Prisma 的 Rust 引擎比 Sequelize 的纯 JS 实现快，但差距在常规 CRUD 中不明显；(6) 生态成熟度 — Sequelize 更成熟（2014 年发布），社区资源更多；Prisma 较新（2019 年）但发展迅速。结论：新项目优先选 Prisma（类型安全 + 开发体验），已有 Sequelize 项目可继续维护，复杂报表场景考虑 mysql2 原生 SQL。

**Q8: 数据库迁移的最佳实践是什么？**
> (1) 迁移必须版本化 — 每次数据库变更都通过迁移文件记录，禁止手动执行 SQL 改表；(2) 迁移必须可回滚 — 每个迁移文件都要实现 up（升级）和 down（回滚）方法；(3) 生产环境用 `migrate deploy` — 只执行未应用的迁移，不重新生成，避免数据丢失；(4) 开发环境用 `migrate dev` — 自动生成迁移文件，方便快速迭代；(5) 数据迁移与结构迁移分离 — DDL（表结构变更）和 DML（数据迁移）分开处理，大表 DDL 需使用 pt-online-schema-change 等工具避免锁表；(6) 迁移文件提交到 Git — 团队共享迁移历史，避免不同步；(7) 测试环境先验证 — 生产执行前在测试环境验证迁移的正确性和耗时；(8) 回滚预案 — 生产发布前确认回滚方案，特别是涉及删表、改列的迁移。

---

**相关链接：**
- [[Redis与缓存策略]]
- [[数据库连接与连接池]]
- [[ORM与查询构建器]]
- Prisma 官方文档：https://www.prisma.io/docs

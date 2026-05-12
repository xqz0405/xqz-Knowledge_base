---
tags:
  - Node.js
  - Redis
  - 缓存
  - ioredis
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Redis 与缓存策略

## What — 是什么

> Redis 是基于内存的高性能键值存储系统，支持多种数据结构，广泛用于缓存、会话管理、排行榜、分布式锁等场景。在 Node.js 生态中，ioredis 是最主流的 Redis 客户端，提供完整的 Cluster/Sentinel/Pipeline 支持。

**核心概念：**

- **数据结构**：String（字符串）、Hash（哈希）、List（列表）、Set（集合）、ZSet（有序集合）、Bitmap、HyperLogLog、Stream、Geo
- **ioredis**：高性能 Node.js Redis 客户端，支持 Pipeline/Cluster/Sentinel/Lua 脚本/发布订阅
- **缓存模式**：旁路缓存（Cache-Aside）、读写穿透（Read-Through/Write-Through）、写回（Write-Behind）
- **过期策略**：惰性删除（访问时检查）+ 定期删除（每秒 10 次随机抽样删除过期键）
- **淘汰策略**：`noeviction`（默认不淘汰）、`allkeys-lru`、`volatile-lru`、`allkeys-lfu`、`volatile-lfu`、`allkeys-random`、`volatile-random`、`volatile-ttl`
- **分布式锁**：基于 `SET key value NX EX` 实现互斥，Redlock 算法解决单点故障
- **Redis Cluster**：数据分片（16384 个哈希槽），每个主节点负责一部分槽，支持自动故障转移
- **发布订阅**：`SUBSCRIBE`/`PUBLISH` 实现消息广播，不持久化消息

**关键特性：**

- 单线程执行命令，避免锁竞争，纯内存操作达到 10 万+ QPS
- Pipeline 将多条命令打包一次发送，减少 RTT（往返时延）
- Lua 脚本原子执行，适合复杂事务逻辑
- Sentinel 哨兵模式自动故障检测和主从切换
- 支持 RDB 快照和 AOF 追加两种持久化方式

## Why — 为什么

**适用场景：**

- 缓存层：热点数据缓存，减轻数据库压力（最常见用途）
- 会话存储：分布式 Session 存储，多实例共享登录状态
- 排行榜：ZSet 的 `ZRANGEBYSCORE` 实现实时排名
- 分布式锁：多实例互斥操作（库存扣减、防重复提交）
- 消息发布订阅：实时通知、配置变更广播
- 限流计数：滑动窗口限流、令牌桶

**对比缓存方案：**

| 维度 | Redis | Memcached | 内存缓存(Map/LRU) |
|------|-------|-----------|------------------|
| 数据结构 | 丰富（5大结构+扩展） | 仅 String | 取决于实现 |
| 持久化 | RDB + AOF | 无 | 无 |
| 分布式 | Cluster 原生支持 | 客户端分片 | 单进程 |
| 内存效率 | 高（ziplist/intset 优化） | 高 | 中 |
| 过期机制 | 惰性+定期 | 惰性 | 取决于实现 |
| 适用场景 | 缓存/锁/排行/消息 | 纯缓存 | 进程内快速缓存 |

**优缺点：**

- ✅ 优点：
  - 纯内存操作，极高性能（10 万+ QPS）
  - 丰富的数据结构，覆盖多种业务场景
  - 支持持久化，重启不丢失数据
  - 原生支持 Cluster，水平扩展
  - Lua 脚本保证原子性
- ❌ 缺点：
  - 内存成本高，不适合存海量数据
  - 单线程模型，大 Key 操作阻塞全库
  - 缓存与数据库一致性是难点
  - Cluster 模式不支持跨槽事务

## How — 怎么用

### 安装配置

```bash
# 安装 Redis（Docker 推荐）
docker run -d --name redis -p 6379:6379 redis:7-alpine

# 安装 ioredis
npm install ioredis
```

```javascript
const Redis = require('ioredis');

// 单实例连接
const redis = new Redis({
    host: '127.0.0.1',
    port: 6379,
    password: 'your-password',
    db: 0,
    retryStrategy(times) {
        const delay = Math.min(times * 200, 5000);
        return delay;
    }
});

// Cluster 连接
const cluster = new Redis.Cluster([
    { host: '10.0.0.1', port: 6379 },
    { host: '10.0.0.2', port: 6379 },
    { host: '10.0.0.3', port: 6379 }
], {
    scaleReads: 'slave', // 读请求分发到从节点
    redisOptions: { password: 'your-password' }
});

// Sentinel 连接
const sentinel = new Redis({
    sentinels: [
        { host: '10.0.0.1', port: 26379 },
        { host: '10.0.0.2', port: 26379 }
    ],
    name: 'mymaster',
    password: 'your-password',
    sentinelPassword: 'sentinel-password'
});
```

### 快速上手

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// String 操作
await redis.set('user:1:name', 'Alice', 'EX', 3600); // 1小时过期
const name = await redis.get('user:1:name');

// Hash 操作
await redis.hset('user:1', 'name', 'Alice', 'email', 'alice@test.com');
const user = await redis.hgetall('user:1');

// List 操作
await redis.lpush('queue:tasks', 'task1', 'task2');
const task = await redis.rpop('queue:tasks');

// Set 操作
await redis.sadd('tags:article:1', 'nodejs', 'redis', 'cache');
const tags = await redis.smembers('tags:article:1');

// ZSet 操作（排行榜）
await redis.zadd('leaderboard', 100, 'Alice', 95, 'Bob', 88, 'Charlie');
const top3 = await redis.zrevrange('leaderboard', 0, 2, 'WITHSCORES');
```

### 代码示例

**ioredis Pipeline 与事务：**

```javascript
// Pipeline：打包多条命令，减少 RTT
async function pipelineDemo() {
    const pipeline = redis.pipeline();
    pipeline.set('key1', 'val1');
    pipeline.set('key2', 'val2');
    pipeline.get('key1');
    pipeline.incr('counter');
    const results = await pipeline.exec();
    // results: [[null, 'OK'], [null, 'OK'], [null, 'val1'], [null, 1]]
}

// 事务（MULTI/EXEC）：原子执行
async function transactionDemo() {
    const results = await redis.multi()
        .set('account:A:balance', 100)
        .set('account:B:balance', 50)
        .decrby('account:A:balance', 20)
        .incrby('account:B:balance', 20)
        .exec();
}

// 乐观锁（WATCH）：CAS 模式
async function casDemo() {
    await redis.watch('counter');
    const val = await redis.get('counter');
    const multi = redis.multi();
    multi.set('counter', parseInt(val) + 1);
    const result = await multi.exec();
    if (!result) {
        console.log('CAS 失败，其他客户端修改了数据');
    }
}
```

**缓存模式实现：**

```javascript
// Cache-Aside（旁路缓存）：最常用模式
async function cacheAside(key, fetchFn, ttl = 3600) {
    const cached = await redis.get(key);
    if (cached) return JSON.parse(cached);

    const data = await fetchFn();
    await redis.set(key, JSON.stringify(data), 'EX', ttl);
    return data;
}

// 使用示例
const user = await cacheAside(
    'user:1',
    () => db.users.findById(1),
    1800
);

// Read-Through + Write-Through：封装缓存层
class CacheStore {
    constructor(redis, ttl = 3600) {
        this.redis = redis;
        this.ttl = ttl;
    }

    async get(key, fetchFn) {
        const cached = await this.redis.get(key);
        if (cached) return JSON.parse(cached);

        const data = await fetchFn();
        await this.redis.set(key, JSON.stringify(data), 'EX', this.ttl);
        return data;
    }

    async set(key, data) {
        await this.redis.set(key, JSON.stringify(data), 'EX', this.ttl);
    }

    async delete(key) {
        await this.redis.del(key);
    }
}

// Write-Behind（写回）：延迟写入数据库
class WriteBehindCache {
    constructor(redis, db) {
        this.redis = redis;
        this.db = db;
        this.queue = 'write-behind:queue';
    }

    async set(key, data) {
        // 先写缓存
        await this.redis.set(key, JSON.stringify(data), 'EX', 3600);
        // 入队列延迟写数据库
        await this.redis.lpush(this.queue, JSON.stringify({ key, data }));
    }

    // 后台消费者定时批量写入数据库
    async flushToDb() {
        const items = await redis.lrange(this.queue, 0, 99);
        if (items.length === 0) return;
        await redis.ltrim(this.queue, items.length, -1);

        for (const item of items) {
            const { key, data } = JSON.parse(item);
            await this.db.save(key, data);
        }
    }
}
```

**分布式锁实现：**

```javascript
// 基础分布式锁
class RedisLock {
    constructor(redis) {
        this.redis = redis;
    }

    async acquire(key, ttlMs = 10000) {
        const token = crypto.randomUUID();
        const result = await this.redis.set(key, token, 'NX', 'PX', ttlMs);
        return result === 'OK' ? token : null;
    }

    async release(key, token) {
        // Lua 脚本确保只有锁持有者才能释放
        const script = `
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
        `;
        return await this.redis.eval(script, 1, key, token);
    }

    async withLock(key, fn, ttlMs = 10000) {
        const token = await this.acquire(key, ttlMs);
        if (!token) throw new Error('Failed to acquire lock');
        try {
            return await fn();
        } finally {
            await this.release(key, token);
        }
    }
}

// 使用示例
const lock = new RedisLock(redis);
try {
    await lock.withLock('order:123:pay', async () => {
        await processPayment(123);
    }, 5000);
} catch (err) {
    console.error('获取锁失败，可能正在处理中');
}

// 限流：滑动窗口
async function rateLimit(userId, limit = 100, windowMs = 60000) {
    const key = `rate:${userId}`;
    const now = Date.now();
    const windowStart = now - windowMs;

    const pipeline = redis.pipeline();
    pipeline.zremrangebyscore(key, 0, windowStart);  // 移除过期记录
    pipeline.zadd(key, now, `${now}:${Math.random()}`); // 添加当前请求
    pipeline.zcard(key);                               // 计数
    pipeline.expire(key, Math.ceil(windowMs / 1000));   // 设置Key过期
    const results = await pipeline.exec();

    const count = results[2][1];
    return count <= limit;
}
```

### 性能调优

| 参数 | 默认值 | 调优建议 | 说明 |
|------|--------|----------|------|
| maxmemory | 无限制 | 设为物理内存 70% | 防止 OOM，留空间给系统和其他进程 |
| maxmemory-policy | noeviction | allkeys-lru | 缓存场景用 LRU 淘汰，避免写满拒绝写入 |
| tcp-keepalive | 300 | 60 | 缩短检测间隔，更快发现断开连接 |
| save | 900/300/60 | 按业务调整或关闭 | 缓存场景可关闭 RDB 减少磁盘 IO |
| appendonly | no | 缓存场景关闭 | 纯缓存不需要 AOF 持久化 |
| hash-max-ziplist-entries | 512 | 1000 | 小 Hash 用 ziplist 节省内存 |
| Pipeline 批量大小 | 无限制 | 100-500 条 | 避免单次 Pipeline 过大阻塞 |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 缓存穿透 | 查询不存在的数据，缓存未命中，请求直达数据库 | 布隆过滤器 + 空值缓存（设短 TTL） |
| 缓存击穿 | 热点 Key 过期瞬间大量请求涌向数据库 | 互斥锁（只让一个请求重建）+ 逻辑过期 |
| 缓存雪崩 | 大量 Key 同时过期 | 过期时间加随机偏移 + 多级缓存 |
| 大 Key 阻塞 | 单个 Key 值过大（>10KB），操作耗时 | 拆分为小 Key，使用 `UNLINK` 异步删除 |
| Key 过期未删除 | 惰性删除依赖访问 | 定期扫描热点 Key，或用 `SCAN` 巡检 |
| 连接泄漏 | 未正确关闭连接 | 使用连接池，应用退出时调用 `redis.quit()` |
| Cluster 跨槽错误 | MGET 等命令涉及不同槽的 Key | 使用 Hash Tag `{user}:1` 确保同槽 |
| 缓存与数据库不一致 | 写数据库后未更新缓存 | 先更新数据库再删缓存 + 延迟双删 |

### 最佳实践

- Key 命名规范：`业务:实体:ID`，如 `user:profile:123`
- 缓存场景关闭 RDB/AOF 持久化，纯内存使用
- 用 Pipeline 批量操作减少 RTT
- 严格控制 Key 大小，单个 Value 不超过 10KB
- 设置合理的过期时间，避免 Key 永不过期
- 分布式锁必须设 TTL 并用 Lua 脚本释放
- 缓存与数据库一致性：先更新数据库，再删除缓存
- 监控内存使用、命中率、慢查询

## 面试题

**Q1: Redis 为什么这么快？**
> 四个原因：① 纯内存操作，数据存储在内存中，读写延迟微秒级；② 单线程模型，避免上下文切换和锁竞争，命令顺序执行无需加锁；③ I/O 多路复用（epoll），单线程处理大量并发连接；④ 高效的数据结构编码——小数据用 ziplist/intset 压缩存储，减少内存分配和碎片。单线程不影响性能瓶颈，因为 Redis 的瓶颈是内存和网络带宽而非 CPU。

**Q2: 缓存穿透、缓存击穿、缓存雪崩分别是什么？如何解决？**
> 缓存穿透：查询不存在的数据，缓存永远不命中，请求直达数据库。解决：布隆过滤器过滤非法请求、空值缓存（设短 TTL 如 5 分钟）。缓存击穿：热点 Key 过期瞬间，大量并发请求同时到达数据库。解决：互斥锁（`SET NX` 只让一个请求重建缓存）、逻辑过期（缓存永不过期但存过期时间，过期后异步更新）。缓存雪崩：大量 Key 同时过期或 Redis 宕机。解决：过期时间加随机偏移量、多级缓存（本地+Redis）、Redis 高可用（Sentinel/Cluster）。

**Q3: ioredis 的 Pipeline 作用是什么？与 MULTI 有什么区别？**
> Pipeline 将多条命令打包成一次网络请求发送，减少 RTT（往返时延）。10 条命令从 10 次 RTT 降为 1 次。`MULTI/EXEC` 是事务，将命令打包原子执行（要么全部执行要么全不执行），但会阻塞其他客户端的命令直到 EXEC。Pipeline 不保证原子性，只是网络优化。两者可组合使用——`redis.pipeline().multi().set(...).exec().exec()` 同时获得原子性和网络优化。

**Q4: 分布式锁如何实现？Redlock 算法解决了什么问题？**
> 基础分布式锁：`SET lock_key token NX PX 30000`，NX 保证互斥，PX 设超时防止死锁，释放时用 Lua 脚本验证 token 防误删。Redlock 解决单点故障问题：向 N 个（通常 5 个）独立 Redis 实例获取锁，超过半数（N/2+1）成功且总耗时未超过锁有效期则获取成功。Redlock 的代价是更高的延迟和复杂度，实践中单实例锁 + Sentinel 高可用通常够用。

**Q5: Redis 的过期和淘汰策略有哪些？**
> 过期策略：① 惰性删除——访问 Key 时检查是否过期，过期则删除；② 定期删除——每秒执行 10 次，每次随机抽取 20 个设置了过期的 Key，删除已过期的，若过期率超过 25% 则继续抽取。淘汰策略（maxmemory-policy）：`noeviction`（不淘汰，写满拒绝）、`allkeys-lru`（全局 LRU，缓存场景推荐）、`volatile-lru`（仅过期键 LRU）、`allkeys-lfu`（全局 LFU，热点数据保留）、`volatile-lfu`、`allkeys-random`、`volatile-random`、`volatile-ttl`（优先淘汰 TTL 短的）。

**Q6: 缓存与数据库一致性有哪些方案？**
> 四种主流方案：① Cache-Aside（旁路缓存）：读时缓存未命中则查数据库写缓存，写时先更新数据库再删缓存。简单但存在短暂不一致窗口。② 延迟双删：更新数据库前删缓存 → 更新数据库 → 延迟 N 毫秒后再删缓存。消除更新期间的脏数据。③ 监听 binlog：通过 Canal 等工具监听 MySQL binlog，异步删除/更新缓存。一致性最好但架构复杂。④ 最终一致性：接受短暂不一致，设合理 TTL 保证最终一致。生产环境推荐 Cache-Aside + 延迟双删 + TTL 兜底。

**Q7: Redis Cluster 的分片原理是什么？有哪些限制？**
> Redis Cluster 将数据分为 16384 个哈希槽（hash slot），每个主节点负责一部分槽。Key 的槽位 = `CRC16(key) % 16384`。客户端请求时，节点返回 `MOVED` 重定向到正确节点。限制：① 不支持跨槽事务（MULTI 的 Key 必须在同一槽）；② `MGET`/`MSET` 等批量操作要求 Key 在同一槽（用 Hash Tag `{tag}` 解决）；③ 不支持 `SELECT` 切换数据库（只有 db0）；④ 从节点默认不处理读请求（需 `READONLY` 命令）。

**Q8: Redis 与数据库双写策略如何选择？**
> 四种策略对比：① 先更新数据库再删缓存（推荐）：一致性较好，极端情况有短暂不一致，用延迟双删兜底；② 先删缓存再更新数据库：高并发下删缓存后、更新数据库前有请求将旧数据写回缓存（脏数据风险）；③ 先更新数据库再更新缓存：并发写时可能后发的写先更新缓存导致数据不一致；④ 只删缓存不更新：读时按需加载（Cache-Aside），最简单可靠。生产环境推荐方案 ① + 合理 TTL 兜底。

---

**相关链接：**
- [[MySQL与ORM]]
- [[数据库连接与连接池]]
- [[消息队列与事件驱动]]
- ioredis 文档：https://github.com/redis/ioredis

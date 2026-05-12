---
tags:
  - Node.js
  - Serverless
  - 函数计算
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Serverless 与函数计算

## What — 是什么

> Serverless 是一种云原生架构，开发者只编写函数逻辑，云平台负责基础设施管理、自动扩缩容和按调用计费。Node.js 是 Serverless 最流行的运行时。

**核心概念：**

- **AWS Lambda**：最成熟的 Serverless 平台，支持 Node.js 18/20
- **Vercel Functions**：前端 + Serverless 深度集成，Next.js 原生支持
- **冷启动**：函数首次调用或长时间未调用后的初始化延迟（100ms-数秒）
- **事件触发**：HTTP 请求/S3 上传/DynamoDB 变更/SQS 消息等触发函数执行
- **执行限制**：内存（128MB-10GB）、超时（1s-15min）、包大小（50MB zipped）

**关键特性：**

- 按调用次数和执行时间计费，空闲不收费
- 自动扩缩容，无需管理服务器
- 冷启动是最大性能挑战
- 适合事件驱动和短时任务

## Why — 为什么

**适用场景：**

- API 接口：请求量波动大，有高峰和低谷
- 事件处理：文件上传后生成缩略图/发送通知
- Webhook：第三方回调处理
- 定时任务：数据同步/报表生成

**对比方案：**

| 维度 | Serverless | 容器化(Docker) | 虚拟机 |
|------|-----------|--------------|--------|
| 运维 | 零运维 | 中 | 高 |
| 扩缩容 | 自动（毫秒级） | 需配置 | 手动 |
| 计费 | 按调用 | 按运行时间 | 按月 |
| 冷启动 | 有（100ms-数秒） | 无 | 无 |
| 长连接 | 不支持 | 支持 | 支持 |

## How — 怎么用

### 代码示例

```javascript
// AWS Lambda Handler
exports.handler = async (event) => {
    const { httpMethod, path, body } = event;

    if (httpMethod === 'GET' && path === '/users') {
        const users = await db.users.findMany();
        return { statusCode: 200, body: JSON.stringify(users) };
    }

    if (httpMethod === 'POST' && path === '/users') {
        const user = await db.users.create({ data: JSON.parse(body) });
        return { statusCode: 201, body: JSON.stringify(user) };
    }

    return { statusCode: 404, body: JSON.stringify({ error: 'Not found' }) };
};

// Vercel Functions (Next.js)
// api/users.ts
export default async function handler(req, res) {
    if (req.method === 'GET') {
        const users = await getUsers();
        return res.status(200).json(users);
    }
}

// 冷启动优化：连接池复用
let dbConnection = null;
async function getDb() {
    if (!dbConnection) dbConnection = await connectDatabase();
    return dbConnection;
}
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 冷启动慢 | 每次冷启动初始化运行时和依赖 | 减小包体积 + 预留实例 + 模块延迟加载 |
| 连接池耗尽 | 每次调用新建连接 | 模块级复用连接（函数外初始化） |
| 不支持 WebSocket | 函数是无状态的 | 用 API Gateway WebSocket 或第三方服务 |
| 本地调试困难 | 运行环境与本地不同 | Serverless Framework 本地模拟 |

### 最佳实践

- 函数外初始化连接池，复用跨调用
- 减小部署包体积（tree-shaking + 精简依赖）
- 设置合理的内存和超时
- 长时任务用 Step Functions 拆分

## 面试题

**Q1: Serverless 冷启动如何优化？**
> 优化手段：① 减小部署包——tree-shaking + 只包含必要依赖；② 延迟加载——不立即 require 非核心模块，按需加载；③ 预留实例——AWS Provisioned Concurrency 保持实例热启动；④ 模块级复用——数据库/Redis 连接池在函数外初始化，跨调用复用；⑤ 选择轻量运行时——Node.js 冷启动比 Java 快 10 倍；⑥ 定时保活——每 5 分钟调用一次防止冷启动。

**Q2: Serverless 不适合什么场景？**
> 不适合：① 长时运行任务——Lambda 最大 15 分钟，视频转码/大数据处理不适用；② 需要长连接——WebSocket/实时推送，Serverless 无法维持持久连接；③ 高频低延迟——金融交易（毫秒级延迟），冷启动不可接受；④ 状态密集——需要大量内存状态，Serverless 无状态；⑤ 固定成本更低——持续高流量场景，Serverless 按调用计费比固定服务器更贵。

---

**相关链接：**
- [[BFF与API网关]]
- [[Docker与容器化]]
- [[生产环境运维]]
- AWS Lambda 文档：https://docs.aws.amazon.com/lambda/

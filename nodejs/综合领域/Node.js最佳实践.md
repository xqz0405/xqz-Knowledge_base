---
tags:
  - Node.js
  - 最佳实践
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Node.js 最佳实践

## What — 是什么

> Node.js 最佳实践是社区总结的项目结构、错误处理、安全、性能等方面的推荐做法，帮助团队建立统一标准。

**核心概念：**

- **项目结构**：按功能/领域组织，不按技术层
- **错误处理**：分层处理 + 自定义错误类 + 全局兜底
- **安全检查清单**：Helmet/CSP/参数化查询/限流/httpOnly Cookie
- **性能清单**：Cluster/连接池/Stream/缓存/压缩
- **代码风格**：ESLint + Prettier 统一规范

## Why — 为什么

**适用场景：**

- 新项目初始化
- 代码审查参考
- 团队规范制定

## How — 怎么用

### 项目结构

```
src/
├── modules/           # 按业务模块组织
│   ├── users/
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   └── __tests__/
│   └── orders/
├── shared/            # 共享工具
│   ├── errors/
│   ├── middleware/
│   └── utils/
├── config/            # 配置
└── app.ts             # 入口
```

### 最佳实践清单

**项目结构：**
- 按功能模块组织，不按技术层（Controller/Service/Repository 在同一模块目录下）
- 配置集中管理，环境变量启动时验证
- 共享工具放在 `shared/` 或 `common/`

**错误处理：**
- 自定义错误类区分业务错误和系统错误
- Express/Fastify 统一错误中间件
- `uncaughtException`/`unhandledRejection` 记录日志后退出
- async 路由用包装函数捕获错误

**安全：**
- Helmet 设置安全头
- 参数化查询防 SQL 注入
- Cookie 设置 httpOnly + secure + SameSite
- JWT 签名密钥 ≥ 32 字符
- 限流防暴力攻击
- 生产环境强制 HTTPS

**性能：**
- Cluster 模式利用多核
- 数据库连接池（max = CPU × 2 + 磁盘数）
- 热点数据 Redis 缓存
- 大文件 Stream 处理
- 响应 gzip/brotli 压缩

**代码风格：**
- ESLint + Prettier 统一格式
- TypeScript strict 模式
- 每个函数单一职责
- 避免 any，用 unknown

**测试：**
- 测试覆盖率 ≥ 80%
- 集成测试用独立数据库
- Mock 只 Mock 边界依赖
- CI 中自动运行

**部署：**
- Docker 多阶段构建
- 健康检查端点
- 优雅关闭（SIGTERM 处理）
- 日志结构化输出

## 面试题

**Q1: Node.js 项目最常见的架构错误？**
> ① Service 层直接操作 req/res——导致 Service 耦合 HTTP 框架，无法复用；② 缺少错误分层——所有错误统一 500，没有区分业务错误和系统错误；③ 全局状态——用全局变量存数据，多实例（Cluster）下不一致；④ 未处理 Promise rejection——Node 15+ 默认导致进程退出；⑤ 同步操作阻塞事件循环——同步 fs/大量 CPU 计算。

**Q2: 如何从 0 搭建一个生产级 Node.js 项目？**
> 步骤：① `npm init` + TypeScript + ESLint + Prettier；② 框架选型（Fastify/Express/NestJS）；③ 分层架构（Controller-Service-Repository）；④ 环境配置（dotenv + zod 验证）；⑤ 错误处理（自定义错误类 + 全局中间件）；⑥ 数据库（Prisma/Sequelize + 连接池）；⑦ 认证（JWT + Guard 中间件）；⑧ 日志（Pino 结构化日志）；⑨ 测试（Jest/Vitest + Supertest）；⑩ Docker 多阶段构建 + CI/CD；⑪ 健康检查 + 监控告警；⑫ 安全加固（Helmet + 限流 + 参数化查询）。

---

**相关链接：**
- [[错误处理与调试]]
- [[Web安全防护]]
- [[性能调优]]
- [[架构模式与分层]]
- [[测试与质量保障]]
- Node.js Best Practices：https://github.com/goldbergyoni/nodebestpractices

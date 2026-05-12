---
tags:
  - Node.js
  - Docker
  - 容器化
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# Docker 与容器化

## What — 是什么

> Docker 容器化是 Node.js 应用的标准部署方式，通过 Dockerfile 定义运行环境，Docker Compose 编排多服务，实现一致的构建、分发和运行。

**核心概念：**

- **Dockerfile**：定义镜像构建步骤的文本文件，从基础镜像开始，安装依赖、复制代码、配置启动命令
- **多阶段构建**：一个 Dockerfile 中多个 `FROM`，构建阶段安装依赖编译代码，运行阶段只复制产物，大幅减小镜像体积
- **Docker Compose**：YAML 配置文件定义多容器应用（API + 数据库 + Redis），一条命令启动全部服务
- **健康检查**：`HEALTHCHECK` 指令或 Compose 的 `healthcheck` 配置，Docker 自动检测容器状态
- **镜像优化**：减小镜像体积、利用构建缓存、安全基础镜像

**关键特性：**

- Node.js 官方 Alpine 镜像仅 ~50MB
- 多阶段构建最终镜像不含构建工具和 devDependencies
- `.dockerignore` 排除 node_modules/.git 等不需要的文件
- `USER node` 非 root 运行提高安全性

## Why — 为什么

**适用场景：**

- 一致的部署环境：开发/测试/生产环境完全相同
- 微服务部署：每个服务独立容器
- CI/CD 流水线：Docker 构建可缓存层
- 水平扩缩容：配合 K8s/Docker Swarm

**对比部署方式：**

| 维度 | Docker | 直接部署(裸机) | PM2 部署 |
|------|--------|-------------|---------|
| 环境一致性 | 完全一致 | 因机器而异 | 因机器而异 |
| 隔离性 | 强（容器隔离） | 无 | 进程级 |
| 部署速度 | 快（拉取镜像） | 慢（手动配置） | 中 |
| 资源开销 | 中（容器层） | 低 | 低 |
| 扩缩容 | 容易（K8s） | 困难 | 手动 |

**优缺点：**

- ✅ 优点：
  - 环境一致，消除"我机器上能跑"
  - 镜像可缓存和版本化
  - 配合 CI/CD 自动化部署
  - 容易水平扩缩容
- ❌ 缺点：
  - 额外的存储和网络开销
  - 容器内调试不如裸机方便
  - 学习曲线（Dockerfile/Compose/K8s）
  - 容器内不用 PM2 集群（一个容器一个进程）

## How — 怎么用

### 快速上手

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -q --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/index.js"]
```

```bash
docker build -t my-api:1.0 .
docker run -d -p 3000:3000 --name my-api my-api:1.0
```

### 代码示例

**多阶段构建 + Compose：**

```dockerfile
# Dockerfile (TypeScript 项目)
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm prune --production

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget -q --spider http://localhost:3000/health || exit 1
CMD ["node", "--max-old-space-size=512", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    ports: ['3000:3000']
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://user:pass@db:5432/mydb
      REDIS_URL: redis://redis:6379
    depends_on:
      db: { condition: service_healthy }
      redis: { condition: service_healthy }
    healthcheck:
      test: ['CMD', 'wget', '-q', '--spider', 'http://localhost:3000/health']
      interval: 30s
      timeout: 3s
      retries: 3
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes: ['pgdata:/var/lib/postgresql/data']
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U user']
      interval: 10s

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s

volumes:
  pgdata:
```

```bash
docker compose up -d
docker compose logs -f api
docker compose down -v
```

### 性能调优

| 参数 | 默认值 | 调优建议 | 说明 |
|------|--------|----------|------|
| 基础镜像 | node:20 | node:20-alpine | Alpine 体积小 10 倍 |
| `--max-old-space-size` | 1.5GB | 容器内存 70% | 防止 OOM |
| COPY 顺序 | 全部复制 | 先复制 package.json | 利用 Docker 缓存层 |
| `.dockerignore` | 无 | 排除 node_modules/.git | 减小构建上下文 |

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 镜像体积过大 | 包含 devDependencies | 多阶段构建 + `npm prune --production` |
| 构建慢 | 每次都重新 npm install | 先 COPY package.json 再 COPY 源码 |
| 容器内时区不对 | 默认 UTC | `TZ=Asia/Shanghai` 环境变量 |
| 容器内 DNS 解析失败 | Docker DNS 配置问题 | `--dns 8.8.8.8` 或 docker-compose dns 配置 |
| 容器 OOM Killed | Node.js 内存超限 | `--max-old-space-size` 限制 + K8s 资源限制 |

### 最佳实践

- 使用 Alpine 基础镜像减小体积
- 多阶段构建分离构建和运行环境
- 非 root 用户运行（`USER node`）
- 先 COPY package.json 再 COPY 源码利用缓存
- 生产环境不装 devDependencies
- 始终添加 HEALTHCHECK
- 敏感配置用环境变量，不写入镜像

## 面试题

**Q1: Docker 多阶段构建的原理？为什么能减小镜像？**
> 多阶段构建在 Dockerfile 中使用多个 `FROM` 指令，每个 `FROM` 开始一个新的构建阶段。构建阶段安装编译工具和 devDependencies，编译 TypeScript 等源码；运行阶段从构建阶段 `COPY --from=builder` 只复制编译产物和生产依赖。最终镜像不包含 TypeScript 编译器、devDependencies、源码等构建时才需要的文件，通常从 1GB+ 缩减到 100-200MB。

**Q2: Docker 容器中为什么不用 PM2 集群模式？**
> 容器化的核心原则是一个容器运行一个进程。PM2 集群模式在容器内创建多个工作进程，与容器编排理念冲突：① 扩缩容由 K8s/Docker Swarm 管理（增减容器数），而非容器内增减进程数；② 健康检查监控容器级别，PM2 内部进程崩溃容器不会标记为不健康；③ 资源限制在容器级别设置，PM2 集群内的进程共享容器资源难以精确控制。正确做法：一个容器一个 Node.js 进程，水平扩容靠增加容器数。

**Q3: 如何优化 Docker 构建速度？**
> 核心原则：利用 Docker 层缓存——未变更的层不重新构建。优化：① 先 `COPY package*.json` + `RUN npm ci`，再 `COPY . .`——依赖不变时跳过 npm install（最耗时步骤）；② 使用 `.dockerignore` 排除 node_modules/.git/logs 减小构建上下文；③ 使用 BuildKit（`DOCKER_BUILDKIT=1`）并行构建和更好的缓存；④ 多阶段构建中依赖安装阶段独立，方便缓存复用。

**Q4: Docker Compose 中如何处理服务依赖顺序？**
> `depends_on` 控制启动顺序，但只保证容器启动不保证服务就绪。`condition: service_healthy` 等待健康检查通过后才启动依赖服务，这是最可靠的方式。Compose v2+ 支持 `depends_on.condition`：`service_started`（默认，只等启动）、`service_healthy`（等健康检查通过）、`service_completed_successfully`（等一次性任务完成）。应用层面也应实现重试逻辑，应对服务启动后短暂不可用的情况。

---

**相关链接：**
- [[进程管理与守护]]
- [[CI与CD与自动化部署]]
- [[生产环境运维]]
- Docker 文档：https://docs.docker.com/

---
tags:
  - Node.js
  - CI/CD
  - 自动化部署
  - GitHub Actions
date: 2026-05-12
status: 已完成
difficulty: 中等
---

# CI/CD 与自动化部署

## What — 是什么

> CI/CD 是持续集成和持续部署的实践，通过自动化流水线实现代码提交后自动测试、构建、部署，减少人工操作和错误。

**核心概念：**

- **CI（持续集成）**：代码提交后自动运行测试和代码检查，确保不引入回归
- **CD（持续部署）**：测试通过后自动构建镜像并部署到目标环境
- **GitHub Actions**：GitHub 内置的 CI/CD 平台，YAML 配置工作流
- **Docker 构建**：CI 中构建 Docker 镜像并推送到镜像仓库
- **灰度发布**：新版本逐步放量，先 5%→10%→50%→100%

**关键特性：**

- GitHub Actions 按事件触发（push/PR/tag/schedule）
- 并行执行测试和构建加速流水线
- 矩阵策略跨 Node.js 版本测试
- 缓存 node_modules 加速安装

## Why — 为什么

**适用场景：**

- 所有团队协作项目
- 自动化测试保障代码质量
- 频繁部署减少手动操作

## How — 怎么用

### 代码示例

```yaml
# .github/workflows/deploy.yml
name: CI/CD
on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

jobs:
    test:
        runs-on: ubuntu-latest
        strategy:
            matrix:
                node-version: [18, 20]
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-node@v4
              with: { node-version: '${{ matrix.node-version }}' }
            - run: npm ci
            - run: npm test
            - run: npm run lint

    deploy:
        needs: test
        if: github.ref == 'refs/heads/main'
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: docker/setup-buildx-action@v3
            - uses: docker/login-action@v3
              with:
                registry: ghcr.io
                username: ${{ github.actor }}
                password: ${{ secrets.GITHUB_TOKEN }}
            - uses: docker/build-push-action@v5
              with:
                push: true
                tags: ghcr.io/${{ github.repository }}:latest,ghcr.io/${{ github.repository }}:${{ github.sha }}
                cache-from: type=gha
                cache-to: type=gha,mode=max
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| CI 慢 | 每次全量安装 | 缓存 node_modules |
| Secret 泄露 | 日志中打印环境变量 | GitHub 自动掩码 + 避免打印 |
| 部署失败回滚 | 新版本有 bug | 镜像回滚到上一版本 |

### 最佳实践

- PR 触发测试，merge 到 main 触发部署
- 使用缓存加速 CI
- 部署前运行 `npm audit --audit-level=high`
- 灰度发布降低风险

## 面试题

**Q1: CI/CD 流水线的典型阶段？**
> 典型阶段：① Install——安装依赖（`npm ci`）；② Lint——代码风格检查（ESLint/Prettier）；③ Test——单元测试+集成测试；④ Build——编译 TypeScript/构建 Docker 镜像；⑤ Security Scan——依赖漏洞扫描；⑥ Deploy——部署到 staging/production；⑦ Smoke Test——部署后验证关键接口。

**Q2: 如何实现灰度发布？**
> 三种方式：① K8s Rolling Update——`maxSurge`/`maxUnavailable` 控制滚动比例；② 蓝绿部署——新旧版本同时存在，切流量到新版本；③ Canary——Ingress/Service Mesh 按 Header/Cookie/IP 路由部分流量到新版本。Node.js + Docker 场景：K8s 原生支持，修改 Deployment 的镜像版本即可。

---

**相关链接：**
- [[Docker与容器化]]
- [[测试与质量保障]]
- [[生产环境运维]]

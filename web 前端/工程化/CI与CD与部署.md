---
tags:
  - Web前端
  - 工程化
  - CI/CD
  - 部署
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# CI/CD与部署

## What — 是什么

> CI/CD 是持续集成和持续交付/部署的实践，通过自动化流水线完成代码检查、测试、构建和部署，确保每次变更安全可靠地到达用户。

**核心概念：**

- **CI（持续集成）**：代码提交后自动运行 lint、测试、构建
- **CD（持续交付）**：自动化到预发布环境，手动批准后发布
- **CD（持续部署）**：全自动化，代码合并后直接发布到生产
- **流水线（Pipeline）**：由多个 Stage 组成的自动化流程
- **环境**：development → staging → production

**关键特性：**

- CI/CD 是工程化的最后一公里
- 前端部署的核心：构建产物 → CDN/静态服务器
- 现代部署方式：容器化、Serverless、Edge Functions

## Why — 为什么

**适用场景：**

- 所有团队协作的项目
- 需要频繁发布的项目
- 需要质量门禁的项目

**对比方案：**

| 维度 | GitHub Actions | GitLab CI | Jenkins | Vercel |
|------|---------------|-----------|---------|--------|
| 配置方式 | YAML | YAML | Groovy/Jenkinsfile | 零配置 |
| 托管 | GitHub 托管 | GitLab 托管 | 自建 | Vercel 托管 |
| 生态 | GitHub 集成 | GitLab 集成 | 插件丰富 | Next.js 最优 |
| 学习成本 | 低 | 低 | 高 | 极低 |
| 灵活性 | 高 | 高 | 极高 | 中 |
| 免费额度 | 2000 min/月 | 400 min/月 | 无限（自建） | 免费额度 |

**优缺点：**

- ✅ 优点：
  - 自动化减少人为错误
  - 质量门禁保证代码质量
  - 快速回滚能力
- ❌ 缺点：
  - 流水线配置有学习成本
  - 自建 CI 有运维负担
  - 调试流水线问题不如本地直观

## How — 怎么用

### GitHub Actions

**基础 CI 流水线：**

```yaml
# .github/workflows/ci.yml
name: CI

on:
    push:
        branches: [main, develop]
    pull_request:
        branches: [main]

jobs:
    check:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4

            - uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: 'pnpm'

            - run: pnpm install --frozen-lockfile

            - name: Lint
              run: pnpm lint

            - name: Type Check
              run: pnpm type-check

            - name: Unit Tests
              run: pnpm test:ci

            - name: Build
              run: pnpm build

            - name: Upload Coverage
              uses: codecov/codecov-action@v4
              with:
                  token: ${{ secrets.CODECOV_TOKEN }}
```

**部署到生产：**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
    push:
        branches: [main]

jobs:
    deploy:
        runs-on: ubuntu-latest
        needs: [check] # 依赖 CI 通过
        environment: production
        steps:
            - uses: actions/checkout@v4

            - uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: 'pnpm'

            - run: pnpm install --frozen-lockfile
            - run: pnpm build

            - name: Deploy to CDN
              run: |
                  aws s3 sync ./dist s3://${{ secrets.S3_BUCKET }} \
                    --delete \
                    --cache-control "public, max-age=31536000, immutable" \
                    --exclude "index.html"
                  aws s3 cp ./dist/index.html s3://${{ secrets.S3_BUCKET }}/index.html \
                    --cache-control "public, max-age=0, s-maxage=60"

            - name: Invalidate CDN Cache
              run: |
                  aws cloudfront create-invalidation \
                    --distribution-id ${{ secrets.CF_DISTRIBUTION_ID }} \
                    --paths "/index.html"
```

**Preview 部署（PR 预览）：**

```yaml
# .github/workflows/preview.yml
name: Preview

on:
    pull_request:

jobs:
    preview:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-node@v4
              with:
                  node-version: 20
                  cache: 'pnpm'

            - run: pnpm install --frozen-lockfile
            - run: pnpm build

            - name: Deploy Preview
              id: deploy
              uses: amondnet/vercel-action@v25
              with:
                  vercel-token: ${{ secrets.VERCEL_TOKEN }}
                  vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
                  vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

            - name: Comment PR
              uses: actions/github-script@v7
              with:
                  script: |
                      github.rest.issues.createComment({
                          issue_number: context.issue.number,
                          owner: context.repo.owner,
                          repo: context.repo.repo,
                          body: `Preview: ${{ steps.deploy.outputs.preview-url }}`
                      });
```

### Docker 容器化部署

```dockerfile
# Dockerfile — 多阶段构建
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

# 生产镜像：仅 Nginx + 构建产物
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # SPA 路由：所有路径回退到 index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源长缓存（文件名含 hash）
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # index.html 不缓存
    location = /index.html {
        add_header Cache-Control "no-cache";
    }

    # gzip
    gzip on;
    gzip_types text/css application/javascript application/json image/svg+xml;
}
```

### Vercel 零配置部署

```bash
# 安装 CLI
npm i -g vercel

# 部署
vercel          # 预览部署
vercel --prod   # 生产部署
```

```json
// vercel.json — 自定义配置
{
    "framework": "vite",
    "buildCommand": "pnpm build",
    "outputDirectory": "dist",
    "rewrites": [
        { "source": "/(.*)", "destination": "/index.html" }
    ],
    "headers": [
        {
            "source": "/assets/(.*)",
            "headers": [
                { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
            ]
        }
    ]
}
```

### 版本管理与回滚

```bash
# 语义化版本
npm version patch  # 1.0.0 → 1.0.1（Bug 修复）
npm version minor  # 1.0.0 → 1.1.0（新功能）
npm version major  # 1.0.0 → 2.0.0（破坏性变更）

# 回滚到上一版本（Vercel）
vercel rollback

# 回滚到上一版本（Docker + K8s）
kubectl rollout undo deployment/my-app

# 回滚到上一版本（Git）
git revert HEAD
git push origin main
```

### 环境变量管理

```yaml
# GitHub Actions: 环境变量 + Secrets
jobs:
    deploy:
        environment: production
        steps:
            - name: Build with env
              run: pnpm build
              env:
                  VITE_API_URL: ${{ secrets.API_URL }}
                  VITE_COMMIT_SHA: ${{ github.sha }}

            - name: Sentry Upload
              run: pnpm sentry-upload
              env:
                  SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
```

```typescript
// vite.config.ts — 构建时注入
export default defineConfig({
    define: {
        __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
        __COMMIT_SHA__: JSON.stringify(process.env.VITE_COMMIT_SHA),
    },
});
```

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 构建缓存失效 | 每次全新安装依赖 | 利用 CI 缓存（`actions/setup-node` 的 cache） |
| 部署后白屏 | 路由未配置回退 | Nginx `try_files` / Vercel `rewrites` |
| 环境变量泄露 | 构建产物中包含敏感信息 | 只用 `VITE_` 前缀的变量，密钥走后端 |
| 并行部署冲突 | 多人同时合并到 main | 锁定部署环境，串行执行 |
| CDN 缓存不更新 | index.html 被缓存 | index.html 设 `no-cache`，JS/CSS 文件名含 hash |

### 最佳实践

- CI 必须包含 lint + type-check + test + build
- PR 触发 Preview 部署，main 触发生产部署
- 静态资源文件名含 hash + 长缓存，index.html 不缓存
- 环境变量分环境管理，敏感信息用 Secrets
- 容器化部署用多阶段构建，生产镜像只含 Nginx + 产物

## 面试题

**Q1: CI 和 CD 分别是什么？持续交付和持续部署有什么区别？**
> CI（持续集成）是代码提交后自动运行 lint、测试、构建，确保变更不破坏主分支；CD 包含持续交付（Continuous Delivery，自动化到预发布环境，手动批准后发布）和持续部署（Continuous Deployment，全自动化直达生产）。区别在于最后一步是否需要人工审批。

**Q2: 蓝绿部署和滚动部署的区别是什么？**
> 蓝绿部署维护两套完整环境（蓝/绿），切换流量实现零停机发布，回滚只需切回旧环境，但需要双倍资源；滚动部署逐步替换旧版本实例，资源占用少，但新旧版本会短暂共存，需注意接口兼容性，回滚较慢需逐个替换回去。

**Q3: Docker 多阶段构建的作用是什么？**
> 多阶段构建用多个 `FROM` 指令将构建环境和运行环境分离：第一阶段用完整 Node 镜像安装依赖和构建，第二阶段仅用轻量 Nginx/Alpine 镜像 + 构建产物。最终镜像不含源码和 `node_modules`，体积从 GB 级降到 MB 级，减少攻击面，部署更快。

**Q4: 前端项目的环境变量如何安全管理？**
> 构建时注入（Vite 的 `VITE_` 前缀变量、Webpack 的 `DefinePlugin`），非 `VITE_` 前缀的变量不会暴露到前端；敏感信息（API 密钥、Token）存放在 CI 的 Secrets 中，仅构建时注入，不写入代码仓库；`index.html` 不缓存 + 静态资源含 hash 确保 CDN 更新。

---

**相关链接：**
- [[Webpack与Vite]]
- [[代码规范ESLint与Prettier]]
- [[前端监控与错误追踪]]

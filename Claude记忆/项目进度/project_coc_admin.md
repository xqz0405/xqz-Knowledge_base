---
name: coc-admin-website
description: CoC部落官网管理系统，React+FastAPI全栈，JWT认证+角色权限，多Agent协作架构
metadata:
  type: project
---

《部落冲突》(Clash of Clans) 部落官网管理系统，前后端分离 + 多 Agent 协作架构。

**Why:** 用户需要一个部落管理的全栈项目，同时探索 AI Agent 协作开发模式

**How to apply:**

## 仓库地址

| 模块 | 仓库 |
|------|------|
| 前端 | https://gitee.com/xqz_0405/coc-web |
| 后端 | https://gitee.com/xqz_0405/coc-server |

## 项目结构

```
xqz-coc-project/
├── xqz-coc/        ← 前端 (React + TypeScript)
├── coc-server/     ← 后端 (FastAPI)
├── docs/           ← 共享文档
│   └── api-contract.md  ← API 契约（唯一真相源）
├── tests/          ← 测试
│   ├── api/        ← API 测试
│   └── e2e/        ← 端到端测试
├── README.md       ← 人类文档
└── claude.md       ← AI 行为规则
```

## 技术栈

- 前端：React 18 + TypeScript + Redux Toolkit + React Router v6
- 后端：FastAPI + SQLAlchemy + JWT + MySQL
- 认证：JWT Token + 刷新机制
- 权限：leader/coLeader/admin/member 四级角色

## 功能模块

- 管理后台：部落信息、战绩、新闻（富文本）、成员管理
- 公开页面：部落首页、战绩列表、新闻列表/详情
- 统计：成员数、胜场、连胜、总星星、总捐赠、总奖杯、平均奖杯

## 多 Agent 协作架构（核心设计）

项目设计了 Agent 隔离协作模式：
- **后端 Agent** → 只操作 `coc-server/`，定义 API
- **前端 Agent** → 只操作 `xqz-coc/`，消费 API
- **测试 Agent** → 只操作 `tests/`，验证行为

规则：
1. API Contract First — 所有 API 必须先在 `docs/api-contract.md` 定义
2. 上下文隔离 — 每个 Agent 只读取自己域的文件，不跨域
3. 输出约束 — 不输出完整 DOM/日志，只摘要结果

## 已改进

- 统一 ApiResponse 格式、DetailedHTTPException、字段级验证 → [[project_backend_error_handling]]

## 待办

- 成员激活/停用流程
- 图片上传
- 响应式设计优化
- 单元测试
- 部署配置

[[project_backend_error_handling]]

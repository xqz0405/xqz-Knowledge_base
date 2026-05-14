---
name: coc-admin-website
description: CoC部落官网管理系统，React+FastAPI全栈，JWT认证+角色权限
metadata:
  type: project
---

《部落冲突》(Clash of Clans) 部落官网项目，包含前后端。

**Why:** 用户需要一个部落管理的全栈项目

**How to apply:**
- 前端：React 19 + TypeScript + Redux Toolkit + React Router v7，位于 `F:\xqz-coc\`
- 后端：FastAPI + SQLAlchemy + JWT + SQLite/MySQL，位于 `F:\coc-server\`
- 认证：JWT Token + 刷新机制
- 权限：leader/coLeader/admin/member 四级角色
- 管理后台：部落信息、战绩、新闻（富文本）、成员管理
- 数据库模型：User、ClanInfo、ClanWarRecord、News
- 已改进：统一 ApiResponse 格式、DetailedHTTPException、字段级验证 → [[project_backend_error_handling]]

**待办：**
- 成员激活/停用流程
- 图片上传
- 响应式设计优化
- 单元测试
- 部署配置

[[project_backend_error_handling]]

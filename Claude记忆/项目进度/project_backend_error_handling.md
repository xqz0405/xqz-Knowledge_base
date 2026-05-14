---
name: backend-error-handling
description: CoC后端统一API响应格式、详细错误信息、字段级验证提示
metadata:
  type: project
---

2026-04-23 对 CoC 后端的错误处理改进。

**Why:** 后端错误信息不够详细，前端难以定位具体是哪个字段出错

**How to apply:**
- `app/schemas/response.py`：新增 `ApiResponse<T>` 泛型类（data/message/success/code/errors）
- `app/core/exceptions.py`：新增 `DetailedHTTPException` + 便捷函数（bad_request/unauthorized/forbidden/not_found/conflict/validation_error）
- 所有控制器（auth/clan/user）更新为统一响应格式
- 前端 `ApiError` 类增加 code 和 errors 字段
- TypeScript 类型 `ApiResponse` 增加 code 和 errors 可选字段
- 字段验证错误格式：`errors.fields = { "email": "错误信息" }`
- 业务逻辑错误格式：`errors = { "field": "email", "error_code": "EMAIL_ALREADY_EXISTS" }`

[[project_coc_admin]]

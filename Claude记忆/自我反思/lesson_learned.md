---
name: lesson-learned
description: 从所有会话中提炼的经验教训，持续追加
metadata:
  type: reflection
---

| # | 日期 | 教训 | 来源 |
|---|------|------|------|
| 1 | 2026-05-11 | 大量 Agent 并行会频繁重连，效率下降 | 用户纠正 |
| 2 | 2026-05-11 | 文章内容太少用户不满意，要写长写全 | 用户反馈 |
| 3 | 2026-05-13 | VitePress 的 Vue 模板编译器会破坏 md 内容，根本问题要换工具 | 踩坑经验 |
| 4 | 2026-05-13 | preprocess.mjs 用 fileURLToPath，不能用 replace 处理路径 | Linux 部署踩坑 |
| 5 | 2026-05-14 | Windows 上 fs.rmSync 会 ENOTEMPTY，需 retry+fallback | 踩坑经验 |
| 6 | 2026-05-14 | 记忆系统应从全局出发，不应局限在单个项目 | 用户纠正 |
| 7 | 2026-05-14 | 方案设计时主动考虑跨设备迁移需求 | 用户反馈 |

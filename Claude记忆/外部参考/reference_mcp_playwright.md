---
name: mcp-playwright-global
description: Playwright MCP 已配置到全局 .mcp.json，用于浏览器自动化
metadata:
  type: reference
---

Playwright MCP 已配置到全局 `~/.claude/.mcp.json`，通过 `npx @playwright/mcp` 启动。

**Why:** 用户有浏览器自动化需求（测试网站、调试前端），需要全局可用的 Playwright MCP

**How to apply:**
- 当前项目 `xqz-web-site` 的网站 docs.xqzweb.xyz 可用 Playwright 进行端到端测试
- 全局 MCP 配置文件：`C:\Users\Lenovo\.claude\.mcp.json`，包含 blender 和 playwright 两个服务器
- 修改后需重启 Claude Code 会话生效

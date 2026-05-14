---
name: astro-site-deployed
description: Obsidian知识库Astro网站，已部署上线，CI/CD完成
metadata:
  type: project
---

基于 Obsidian 知识库构建的 Astro 静态网站，已部署到阿里云服务器。
- 知识库仓库：https://github.com/xqz0405/xqz-Knowledge_base
- 网站仓库：https://github.com/xqz0405/xqz-web-site

**Why:** 用户希望将知识库以网站形式公开展示

**How to apply:**
- 项目仓库：https://github.com/xqz0405/xqz-web-site，知识库作为 git submodule 引入
- 网站地址：`https://docs.xqzweb.xyz`
- 阿里云 ECS：2核2G3Mbps，宝塔面板管理，Nginx 托管静态文件
- 域名：`xqzweb.xyz`，子域名 `docs` 通过阿里云 DNS A 记录解析
- SSL：Let's Encrypt 免费证书，宝塔自动申请，强制 HTTPS

**从 VitePress 迁移到 Astro 的原因：**
VitePress 的 Vue 模板编译器把 md 中的 HTML/Vue 语法当模板解析（`@click`、`{{ }}`、泛型 `<T>` 等），正则转义修不完。Astro 的 markdown 渲染不过任何模板编译器，从根本上消除转义问题。preprocess 从 ~430 行砍到 ~150 行。

**CI/CD 流水线（2026-05-13 已跑通）：**
1. `git push` → GitHub Actions 自动触发
2. Actions: checkout + submodule → npm ci → preprocess → astro build
3. rsync dist/ 到服务器 `/www/wwwroot/docs.xqzweb.xyz/`
4. GitHub Secrets：SERVER_HOST / SERVER_USER / SERVER_SSH_KEY / SERVER_DEST

**技术栈：**
- Astro 5.x + Content Collections（4个集合：go/nodejs/python/web-前端）
- `scripts/preprocess.mjs` — 双链转换（[[wiki]] → [markdown]()），URL 用 `encodeURI`
- `scripts/stacks.json` — 技术栈配置
- 自定义布局/组件（BaseLayout / ArticleLayout / Sidebar），暗色模式、侧边栏折叠、响应式
- Logo: `public/xqz-logo.png`，Header 显示图片+文字
- 中文 URL 直接使用中文路径 + URI 编码

**部署注意事项：**
- 服务器需 `chown -R www:www` + `chmod -R 755` 修复权限
- 宝塔 `.user.ini` 有 immutable 属性，需 `chattr -i` 后删除
- `preprocess.mjs` 用 `fileURLToPath(import.meta.url)` 取路径
- Windows 上 `cleanContent` 需 `fs.rmSync` + retry + `rd /s /q` fallback

[[project_xqz_knowledge_base]] [[project_site_bugfixes]]

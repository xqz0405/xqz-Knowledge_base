---
name: project-site-bugfixes
description: 网站bug修复、UI改进、CI/CD增强记录
metadata:
  type: project
---

docs.xqzweb.xyz 网站的修复和改进记录。

1. 侧边栏分类点击无法收起/展开：HTML 用 `data-category`，JS 读 `el.dataset.categoryId`，属性名不匹配。修复为统一用 `data-category`。
2. 侧边栏和栈首页显示"未分类"：`index.md` 导致空分类出现。修复为过滤掉 `index.md` 和空分类。
3. 栈首页分类网格响应式：固定2列 → 宽屏4列→中屏3列→窄屏2列→手机1列。
4. 字体和卡片尺寸偏小：卡片内边距、标题字号、链接字号、列表间距均加大。
5. Node.js 图标从方形🟩改圆形🟠，避免与Go的🟢颜色重叠。
6. 添加 `scripts/update-submodule.mjs` 一键更新子模块并推送脚本。
7. CI/CD 增强：每日0点自动更新子模块并重新构建部署。
8. 站点标题统一为小写 "xqz 技术知识库"。
9. 删除 `src/content/config.ts`，Astro 5 不再需要该文件。
10. 右侧 TOC 大纲：从 Markdown headings 提取 h2/h3，纯 HTML+CSS，<=1200px 隐藏。
11. 圣杯布局：左侧 sidebar 280px + 中间内容区自适应 + 右侧 TOC 240px。
12. 全局字体放大：正文 1.05rem，h1 2rem，h2 1.55rem，h3 1.25rem，行高 1.8。

[[project_xqz_astro_site]]

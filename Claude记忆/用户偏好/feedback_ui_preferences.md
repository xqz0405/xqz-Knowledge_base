---
name: feedback-ui-preferences
description: 用户对网站UI的偏好：大屏下内容偏大、卡片偏胖、图标颜色不重叠
metadata:
  type: feedback
---

大屏下字体和卡片尺寸应偏大偏胖，避免显得窄小拥挤。
**Why:** 用户反馈默认尺寸在大屏下偏小偏窄，视觉上不够舒适。
**How to apply:** 设计时优先考虑大屏体验，卡片内边距、字体、间距等宁可大一些；响应式从小屏到大屏逐级放大。

分类图标颜色不应重叠，每个分类用不同颜色区分。
**Why:** Go 和 Node.js 原来都用绿色圆形图标，视觉上无法区分。
**How to apply:** 当前配色方案：Go 🟢、Node.js 🟠、Python 🔵、Web前端 🟡，新增分类时也注意颜色去重。

---
name: feedback-agent-usage
description: 用户不希望大量使用Agent，直接写更高效
metadata:
  type: feedback
---

不要大量使用 Agent 并行写文章。Agent 重连尝试多，效率反而更差。

**Why:** 用户发现大量 Agent 并行会导致频繁重连，整体效率下降

**How to apply:**
- 写知识库文章时直接在主会话中用 Write 工具创建，不要派发 Agent
- 仅在需要独立研究/探索时使用单个 Agent
- 如需并行，最多 2-3 个 Agent，不要一次开 5 个以上

---
tags:
  - Web前端
  - AI
  - LLM
  - RAG
  - Web AI
date: 2026-05-11
status: 已完成
difficulty: 中等
---

# AI 与前端结合

## What — 什么是 AI 与前端结合

AI 与前端结合是指将大语言模型（LLM）、向量检索（RAG）、机器学习推理等 AI 能力集成到前端应用中。这包括：调用云端 AI API、浏览器端本地推理、AI 辅助开发等场景。

### 结合模式分类

| 模式 | 说明 | 代表 |
|------|------|------|
| API 调用 | 前端调用云端 AI API | ChatGPT Web、Claude.ai |
| 流式输出 | SSE / WebSocket 实时展示生成内容 | AI 对话界面 |
| RAG | 检索增强生成，结合业务知识库 | 企业知识助手 |
| 浏览器端推理 | ONNX / WebLLM 本地运行模型 | Web AI |
| AI 辅助开发 | 代码生成、设计稿转代码 | Cursor、v0.dev |
| Agent 模式 | AI 调用前端工具完成复杂任务 | Claude Computer Use |

---

## Why — 为什么前端需要 AI

### 1. 交互范式变革

从"用户输入 → 系统响应"到"用户意图 → AI 理解 → 智能响应"。搜索框变成对话框，表单填写变成自然语言描述。

### 2. 内容生成能力

前端不再只是展示预设内容，而是可以动态生成 UI、文案、图表。AI 让"千人千面"从推荐算法升级到内容生成。

### 3. 开发效率提升

AI 辅助编码（Copilot / Cursor）、设计稿转代码（v0.dev）、自动化测试生成，正在改变前端开发方式。

### 4. 隐私与延迟

浏览器端推理（Web AI）无需将数据发送到服务器，保护隐私；本地推理消除网络延迟，适合实时场景。

---

## How — 怎么用

### 1. 调用 LLM API — 流式对话

```ts
// services/ai.ts
const API_URL = 'https://api.openai.com/v1/chat/completions'

interface ChatMessage {
  role: 'system' | 'user' | 'assistant'
  content: string
}

// 流式调用
async function* streamChat(
  messages: ChatMessage[],
  options?: { model?: string; temperature?: number }
) {
  const response = await fetch(API_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
    },
    body: JSON.stringify({
      model: options?.model || 'gpt-4o',
      messages,
      temperature: options?.temperature ?? 0.7,
      stream: true,
    }),
  })

  const reader = response.body!.getReader()
  const decoder = new TextDecoder()

  while (true) {
    const { done, value } = await reader.read()
    if (done) break

    const chunk = decoder.decode(value)
    const lines = chunk.split('\n').filter((l) => l.startsWith('data: '))

    for (const line of lines) {
      const data = line.slice(6)
      if (data === '[DONE]') return

      try {
        const parsed = JSON.parse(data)
        const content = parsed.choices[0]?.delta?.content
        if (content) yield content
      } catch {}
    }
  }
}
```

```vue
<!-- ChatView.vue -->
<template>
  <div class="chat">
    <div class="messages" ref="messagesRef">
      <div v-for="msg in messages" :key="msg.id"
        :class="['message', `message--${msg.role}`]">
        <MarkdownRenderer :content="msg.content" />
      </div>
      <div v-if="streaming" class="message message--assistant">
        <MarkdownRenderer :content="streamingContent" />
        <span class="cursor">|</span>
      </div>
    </div>
    <div class="input-area">
      <textarea v-model="input" @keydown.enter.exact.prevent="send"
        placeholder="输入消息..." rows="1" />
      <button @click="send" :disabled="streaming">发送</button>
    </div>
  </div>
</template>

<script setup>
const messages = ref([])
const input = ref('')
const streaming = ref(false)
const streamingContent = ref('')
const messagesRef = ref(null)

async function send() {
  if (!input.value.trim() || streaming.value) return

  const userMessage = { id: Date.now(), role: 'user', content: input.value }
  messages.value.push(userMessage)
  input.value = ''
  streaming.value = true
  streamingContent.value = ''

  const chatMessages = messages.value.map(({ role, content }) => ({ role, content }))

  try {
    for await (const chunk of streamChat(chatMessages)) {
      streamingContent.value += chunk
      scrollToBottom()
    }

    messages.value.push({
      id: Date.now(),
      role: 'assistant',
      content: streamingContent.value,
    })
  } finally {
    streaming.value = false
    streamingContent.value = ''
  }
}

function scrollToBottom() {
  nextTick(() => {
    messagesRef.value?.scrollTo({
      top: messagesRef.value.scrollHeight,
      behavior: 'smooth',
    })
  })
}
</script>
```

---

### 2. Markdown 渲染与代码高亮

AI 输出的内容通常是 Markdown 格式，需要渲染为 HTML 并支持代码高亮。

```bash
npm install marked highlight.js
```

```vue
<!-- MarkdownRenderer.vue -->
<template>
  <div class="markdown-body" v-html="rendered" />
</template>

<script setup>
import { marked } from 'marked'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'

const props = defineProps<{ content: string }>()

marked.setOptions({
  highlight(code, lang) {
    if (lang && hljs.getLanguage(lang)) {
      return hljs.highlight(code, { language: lang }).value
    }
    return hljs.highlightAuto(code).value
  },
})

const rendered = computed(() => marked(props.content))
</script>
```

---

### 3. RAG — 检索增强生成

RAG 让 AI 回答基于你的业务知识库，而非通用的训练数据。

```
用户提问 → 向量检索知识库 → 拼接相关文档 + 问题 → LLM 生成回答
```

```ts
// RAG 前端实现
async function ragQuery(question: string) {
  // 1. 将问题转为向量（调用 Embedding API）
  const queryEmbedding = await getEmbedding(question)

  // 2. 向量检索最相关的文档片段
  const relevantDocs = await vectorSearch(queryEmbedding, { topK: 5 })

  // 3. 构造增强 Prompt
  const context = relevantDocs
    .map((doc, i) => `[文档${i + 1}] ${doc.content}`)
    .join('\n\n')

  const augmentedMessages = [
    {
      role: 'system',
      content: `你是一个知识助手。请基于以下参考文档回答用户问题。如果文档中没有相关信息，请如实说明。

参考文档：
${context}`,
    },
    { role: 'user', content: question },
  ]

  // 4. 流式生成回答
  return streamChat(augmentedMessages)
}

// 向量搜索（调用后端 API）
async function vectorSearch(embedding: number[], options: { topK: number }) {
  const res = await fetch('/api/vector-search', {
    method: 'POST',
    body: JSON.stringify({ embedding, topK: options.topK }),
  })
  return res.json()
}

// Embedding
async function getEmbedding(text: string) {
  const res = await fetch('https://api.openai.com/v1/embeddings', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
    },
    body: JSON.stringify({ model: 'text-embedding-3-small', input: text }),
  })
  const data = await res.json()
  return data.data[0].embedding
}
```

---

### 4. Structured Output — 结构化输出

让 LLM 返回 JSON 格式，方便前端直接使用。

```ts
interface SearchResult {
  title: string
  url: string
  summary: string
}

async function aiSearch(query: string): Promise<SearchResult[]> {
  const messages: ChatMessage[] = [
    {
      role: 'system',
      content: `你是搜索助手。根据用户查询，返回 JSON 数组。
格式：[{"title": "标题", "url": "链接", "summary": "摘要"}]
只返回 JSON，不要其他内容。`,
    },
    { role: 'user', content: query },
  ]

  let result = ''
  for await (const chunk of streamChat(messages, { temperature: 0 })) {
    result += chunk
  }

  // 提取 JSON
  const jsonMatch = result.match(/\[[\s\S]*\]/)
  if (!jsonMatch) return []

  return JSON.parse(jsonMatch[0]) as SearchResult[]
}
```

---

### 5. Agent 模式 — AI 调用前端工具

让 AI 通过 Function Calling 调用前端能力（搜索、导航、API 调用）。

```ts
// 定义工具
const tools = [
  {
    type: 'function',
    function: {
      name: 'navigate',
      description: '导航到指定页面',
      parameters: {
        type: 'object',
        properties: {
          path: { type: 'string', description: '路由路径，如 /dashboard' },
        },
        required: ['path'],
      },
    },
  },
  {
    type: 'function',
    function: {
      name: 'search_docs',
      description: '搜索文档',
      parameters: {
        type: 'object',
        properties: {
          query: { type: 'string', description: '搜索关键词' },
        },
        required: ['query'],
      },
    },
  },
  {
    type: 'function',
    function: {
      name: 'create_issue',
      description: '创建工单',
      parameters: {
        type: 'object',
        properties: {
          title: { type: 'string', description: '工单标题' },
          description: { type: 'string', description: '工单描述' },
          priority: { type: 'string', enum: ['low', 'medium', 'high'] },
        },
        required: ['title', 'description'],
      },
    },
  },
]

// 工具执行器
const toolExecutors: Record<string, (args: any) => Promise<string>> = {
  navigate: async ({ path }) => {
    router.push(path)
    return `已导航到 ${path}`
  },
  search_docs: async ({ query }) => {
    const results = await searchDocuments(query)
    return JSON.stringify(results)
  },
  create_issue: async ({ title, description, priority }) => {
    const issue = await createIssue({ title, description, priority })
    return `工单 #${issue.id} 已创建`
  },
}

// Agent 循环
async function agentChat(userMessage: string) {
  const messages: ChatMessage[] = [
    { role: 'system', content: '你是智能助手，可以调用工具帮助用户完成任务。' },
    { role: 'user', content: userMessage },
  ]

  while (true) {
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
      },
      body: JSON.stringify({ model: 'gpt-4o', messages, tools }),
    })

    const data = await response.json()
    const choice = data.choices[0]
    const toolCalls = choice.message.tool_calls

    messages.push(choice.message)

    if (!toolCalls) {
      // 没有工具调用，返回最终回复
      return choice.message.content
    }

    // 执行工具调用
    for (const toolCall of toolCalls) {
      const args = JSON.parse(toolCall.function.arguments)
      const result = await toolExecutors[toolCall.function.name](args)

      messages.push({
        role: 'tool',
        content: result,
        tool_call_id: toolCall.id,
      })
    }
  }
}
```

---

### 6. 浏览器端推理 — WebLLM

```bash
npm install @mlc-ai/web-llm
```

```ts
import { CreateMLCEngine } from '@mlc-ai/web-llm'

async function initLocalModel() {
  const engine = await CreateMLCEngine(
    'Llama-3.1-8B-Instruct-q4f16_1-MLC',
    { initProgressCallback: (progress) => {
        console.log(`加载进度: ${(progress.progress * 100).toFixed(1)}%`)
      }
    }
  )

  return engine
}

async function localChat(engine, message: string) {
  const reply = await engine.chat.completions.create({
    messages: [{ role: 'user', content: message }],
    stream: true,
  })

  let result = ''
  for await (const chunk of reply) {
    const delta = chunk.choices[0]?.delta?.content || ''
    result += delta
    process.stdout.write(delta)
  }

  return result
}
```

**WebLLM 的限制**：

| 限制 | 说明 |
|------|------|
| 模型大小 | 7B 参数模型量化后约 4GB，首次加载慢 |
| 内存要求 | 至少 8GB 显存/内存 |
| 推理速度 | CPU 较慢，WebGPU 加速后接近可用 |
| 浏览器兼容 | 需要 WebGPU 支持（Chrome 113+） |

---

### 7. AI 驱动的 UI 生成

```tsx
// 使用 AI 动态生成 UI 组件
function AIComponent({ prompt }: { prompt: string }) {
  const [code, setCode] = useState('')
  const [Component, setComponent] = useState<React.ComponentType | null>(null)

  async function generateComponent() {
    const response = await fetch('/api/generate-component', {
      method: 'POST',
      body: JSON.stringify({
        prompt,
        framework: 'react',
        styling: 'tailwind',
      }),
    })
    const { code: generatedCode } = await response.json()
    setCode(generatedCode)

    // 动态编译并渲染
    const compiled = await compileAndEval(generatedCode)
    setComponent(() => compiled)
  }

  if (!Component) {
    return <button onClick={generateComponent}>生成: {prompt}</button>
  }

  return <Component />
}
```

---

### 常见问题与踩坑

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| API Key 泄露 | 前端直接调用 AI API | 通过后端代理，前端不直接暴露 Key |
| 流式输出闪烁 | 频繁 setState 导致重渲染 | 用 useRef 缓冲 + requestAnimationFrame 批量更新 |
| Markdown 渲染 XSS | AI 输出含恶意 HTML | 用 DOMPurify 清洗 HTML |
| 上下文过长 | 对话历史超 Token 限制 | 滑动窗口 / 摘要压缩历史 |
| 模型幻觉 | AI 编造不存在的信息 | RAG 检索 + 引用来源 |
| WebLLM 加载慢 | 模型文件大（GB 级） | 首次加载缓存到 IndexedDB |

### 最佳实践

1. **API Key 永远放后端**：前端调用你的后端代理，后端调用 AI API。
2. **流式体验优先**：AI 生成耗时长，必须用流式输出避免用户等待。
3. **RAG 减少幻觉**：让 AI 基于你的文档回答，并展示引用来源。
4. **降级方案**：AI 不可用时（API 限流、网络故障）提供传统交互方式。
5. **成本控制**：缓存常见查询、压缩上下文、选择合适的模型（简单任务用 Haiku）。

---

## 面试题

### 1. 前端调用 LLM API 时，API Key 应该放在哪里？为什么？

**答**：必须放在后端。前端代码对用户完全可见，无论是环境变量还是代码中的硬编码，都可以在浏览器 DevTools 或打包产物中被找到。API Key 泄露意味着任何人可以用你的账号调用 AI API，产生费用甚至滥用。正确做法：前端调用你的后端 API（如 `/api/chat`），后端验证用户身份后携带 Key 调用 AI API，将流式响应转发给前端。这样 Key 永远不会暴露给浏览器。

---

### 2. SSE 和 WebSocket 在 AI 流式输出场景中各有什么优劣？

**答**：SSE（Server-Sent Events）是单向通信（服务器→客户端），专为流式数据设计，自动重连，HTTP 协议兼容性好，适合 AI 流式输出。WebSocket 是双向通信，支持全双工，但需要额外的连接管理（心跳、重连）。AI 对话场景中，客户端发送请求是一次性的，后续只需接收流式响应——SSE 完全够用。只有在需要双向实时交互时（如 AI 语音对话、实时协作编辑）才需要 WebSocket。SSE 更简单，推荐作为默认选择。

---

### 3. RAG 的原理是什么？为什么能减少 AI 幻觉？

**答**：RAG（Retrieval-Augmented Generation）的原理是在 LLM 生成回答前，先从知识库中检索相关文档，将文档内容作为上下文拼入 Prompt，让 LLM 基于这些文档回答。它减少幻觉的原因：(1) LLM 的回答被约束在检索到的文档范围内，而不是凭训练数据"编造"；(2) 检索的文档是真实的企业数据，不是 LLM 训练时的通用知识；(3) 可以在回答中引用来源（如"根据文档 X，答案是 Y"），用户可验证。但 RAG 不能完全消除幻觉——如果检索到的文档不相关，LLM 仍可能产生错误回答。

---

### 4. 什么是 Function Calling？前端如何实现 AI Agent？

**答**：Function Calling 是 LLM 的能力，模型可以根据用户意图选择调用预定义的函数，并生成结构化的函数参数。前端实现 Agent 的步骤：(1) 定义工具列表（名称、描述、参数 JSON Schema）传给 LLM；(2) LLM 判断是否需要调用工具，如果需要则返回 tool_calls 字段；(3) 前端解析 tool_calls，执行对应的工具函数（如导航、搜索、创建工单）；(4) 将工具执行结果作为 tool message 返回给 LLM；(5) LLM 根据工具结果生成最终回复或继续调用工具。这是一个循环（ReAct 模式），直到 LLM 不再调用工具为止。

---

### 5. WebLLM 在浏览器中运行大模型的原理是什么？有什么限制？

**答**：WebLLM 利用 WebGPU API 在浏览器中运行量化后的大模型。原理：(1) 模型权重以 MLC 格式量化（4-bit 量化，体积压缩约 4 倍）并缓存到浏览器 IndexedDB；(2) 使用 WebGPU 的 Compute Shader 执行矩阵运算（模型推理的核心操作）；(3) 推理过程完全在浏览器本地进行，无需服务器。限制：(1) **模型大小**——7B 参数模型量化后约 4GB，首次加载需要下载；(2) **内存**——至少需要 8GB 显存/内存；(3) **速度**——CPU 推理极慢，WebGPU 加速后约 10-30 tokens/s；(4) **兼容性**——WebGPU 仅 Chrome 113+、Edge 113+ 支持，Safari/Firefox 尚不完全支持。

---

### 6. 如何处理 AI 对话的上下文长度限制？

**答**：四种策略：(1) **滑动窗口**——只保留最近 N 轮对话，丢弃最早的，最简单但丢失早期信息；(2) **摘要压缩**——当对话超过长度限制时，用 LLM 对早期对话生成摘要，用摘要替代原文，保留关键信息但压缩 Token 数；(3) **向量检索**——将历史对话存入向量数据库，每轮对话前检索最相关的历史，只将相关的历史加入上下文；(4) **混合策略**——最近 5 轮对话原文保留 + 早期对话摘要 + 向量检索补充。实际项目中推荐(4)，平衡信息完整性和 Token 成本。

---

### 7. 前端如何安全地渲染 AI 生成的 Markdown 内容？

**答**：AI 输出的 Markdown 可能包含恶意 HTML/JavaScript（XSS 攻击），必须清洗。安全渲染流程：(1) 用 `marked` 等 Markdown 解析器将 Markdown 转为 HTML；(2) 用 `DOMPurify` 清洗 HTML——移除 `<script>`、`onclick` 等危险标签和属性，保留安全的 HTML 标签；(3) 使用 `v-html`（Vue）或 `dangerouslySetInnerHTML`（React）渲染清洗后的 HTML；(4) 代码块用 `highlight.js` 高亮，避免执行代码。关键：永远不要直接渲染未清洗的 AI 输出。即使 AI 不会故意生成恶意代码，训练数据中可能包含攻击样本。

---

### 8. AI 应用的前端架构与普通应用有什么不同？

**答**：五个关键区别：(1) **流式优先**——普通应用请求-响应模式，AI 应用必须支持流式输出（SSE），UI 需要逐步渲染而非等待完整响应；(2) **状态管理**——AI 对话状态比普通表单复杂得多（多轮对话上下文、流式缓冲、工具调用中间状态），需要专门的对话状态管理；(3) **错误处理**——AI 的不确定性意味着输出可能格式错误、内容幻觉、API 限流，需要优雅降级而非简单报错；(4) **成本意识**——每次 AI 调用都有 Token 成本，需要缓存、压缩上下文、选择合适模型；(5) **实时性**——AI 生成耗时 3-30 秒，需要进度指示、取消机制、中途结果展示，普通应用的 loading 不够用。

---

## 相关链接

- [OpenAI API 文档](https://platform.openai.com/docs/api-reference)
- [Anthropic API 文档](https://docs.anthropic.com/)
- [WebLLM](https://webllm.mlc.ai/)
- [LangChain.js](https://js.langchain.com/)
- [marked — Markdown 解析器](https://marked.js.org/)
- [DOMPurify — XSS 清洗](https://github.com/cure53/DOMPurify)

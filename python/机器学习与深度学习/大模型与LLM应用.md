---
tags:
  - Python
  - 深度学习
  - LLM
  - 大模型
  - RAG
  - LangChain
---

# 大模型与LLM应用

## What — 大模型与LLM应用是什么

大语言模型（LLM）是基于 Transformer Decoder 架构、在海量文本上预训练的超大规模生成式模型。以 GPT、LLaMA、Claude 等为代表，LLM 展现出涌现能力和通用推理能力，催生了全新的应用范式——Prompt 工程、RAG、Agent、微调等。

核心概念：

| 概念 | 说明 |
|------|------|
| 预训练 | 在海量文本上学习语言表示 |
| 涌现能力 | 模型规模超过阈值后突然出现的能力 |
| 上下文窗口 | 模型一次能处理的最大 token 数 |
| 对齐 | 让模型行为符合人类意图（RLHF/DPO） |
| Prompt 工程 | 设计输入提示引导模型输出 |
| RAG | 检索增强生成，结合外部知识 |
| Agent | 让 LLM 自主规划和使用工具 |

```python
# LLM 应用的基本模式
from anthropic import Anthropic

client = Anthropic()
response = client.messages.create(
    model='claude-sonnet-4-6',
    max_tokens=1024,
    messages=[{'role': 'user', 'content': '解释什么是大模型'}]
)
print(response.content[0].text)
```

## Why — 为什么 LLM 改变了 AI 应用

传统 NLP 为每个任务训练专用模型，需要标注数据、训练流程、部署维护。LLM 用一个模型覆盖几乎所有 NLP 任务，通过 Prompt 零样本或少样本适配，极大降低了应用门槛。[[自然语言处理]] 的范式从"训练模型"变为"设计Prompt"。

## How — LLM 应用开发方法

### 1. Prompt 工程

Prompt 工程是通过设计输入提示来引导 LLM 产生高质量输出的技术。

```python
from anthropic import Anthropic

client = Anthropic()

def ask(prompt, model='claude-sonnet-4-6', max_tokens=1024):
    response = client.messages.create(
        model=model, max_tokens=max_tokens,
        messages=[{'role': 'user', 'content': prompt}]
    )
    return response.content[0].text

# === 基础原则 ===

# 1. 明确具体
vague = "写一篇关于AI的文章"
better = "写一篇500字的科普文章，面向高中学生，介绍大语言模型的工作原理，" \
         "使用类比而非专业术语，包含3个生活中的例子"

# 2. 给出示例（Few-shot）
few_shot = """将产品评论分类为正面/负面/中性：

评论: "这个手机电池太不耐用了" → 负面
评论: "物流很快，包装完好" → 正面
评论: "还行吧，没有特别的感觉" → 中性
评论: "屏幕显示效果很棒，但价格偏贵" →"""

# 3. 思维链（Chain-of-Thought）
cot = """解答以下数学题，请一步步思考：

小明有15个苹果，给了小红1/3，又给了小华2个，还剩多少？

请按以下格式回答：
1. 原始数量：
2. 给出数量1：
3. 给出后剩余：
4. 给出数量2：
5. 最终剩余："""

# 4. 角色设定
role_prompt = """你是一位资深Python开发工程师，擅长代码审查。
请审查以下代码，从以下维度给出建议：
1. 代码风格（PEP 8）
2. 性能问题
3. 安全隐患
4. 可维护性

代码：
```python
def get_user(id):
    import pymysql
    conn = pymysql.connect(host='localhost', user='root', password='123456')
    cursor = conn.cursor()
    sql = f"SELECT * FROM users WHERE id = {id}"
    cursor.execute(sql)
    return cursor.fetchone()
```"""

# === 结构化输出 ===
structured = """请分析以下文本的情感，以JSON格式输出：

文本: "这家餐厅的服务态度非常好，菜品也很新鲜，就是等位时间太长了"

输出格式：
{
  "sentiment": "正面/负面/中性",
  "confidence": 0.0-1.0,
  "aspects": [
    {"aspect": "服务", "sentiment": "正面"},
    {"aspect": "菜品", "sentiment": "正面"},
    {"aspect": "等待", "sentiment": "负面"}
  ],
  "summary": "一句话总结"
}"""

# === XML 标签结构化 ===
xml_prompt = """请根据以下信息回答问题：

<document>
Python由Guido van Rossum于1991年创建。Python 3.0于2008年发布。
Python的设计哲学强调代码可读性。
</document>

<question>
Python 3.0是什么时候发布的？
</question>

请仅基于<document>中的信息回答，如果文档中没有相关信息，请说"文档中未提及"。"""
```

| Prompt技巧 | 说明 | 效果 |
|-----------|------|------|
| 明确具体 | 详细描述任务要求 | 减少幻觉 |
| Few-shot | 给出输入输出示例 | 格式对齐 |
| CoT | 要求逐步推理 | 复杂推理+30% |
| 角色设定 | 赋予专家身份 | 专业深度 |
| 结构化输出 | 指定JSON/XML格式 | 可解析 |
| XML标签 | 用标签分隔内容 | 减少混淆 |

### 2. RAG（检索增强生成）

RAG 通过检索外部知识库来增强 LLM 的回答，解决知识过时和幻觉问题。

```python
# === RAG 完整实现 ===

# 1. 文档切分
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=['\n\n', '\n', '。', '，', ' ']
)

# documents = splitter.split_text(long_text)
# 或
# documents = splitter.split_documents(loaded_docs)

# 2. 向量化与存储
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = HuggingFaceEmbeddings(model_name='shibing624/text2vec-base-chinese')

# 从文档创建向量库
# vectorstore = Chroma.from_documents(documents, embeddings, persist_directory='./chroma_db')

# 3. 检索
# retriever = vectorstore.as_retriever(search_kwargs={'k': 4})
# docs = retriever.invoke('Python有哪些特性？')

# 4. 生成回答
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_anthropic import ChatAnthropic

prompt = ChatPromptTemplate.from_template("""基于以下上下文回答问题。如果上下文中没有相关信息，请说"根据提供的信息无法回答"。

上下文：
{context}

问题：{question}

回答：""")

llm = ChatAnthropic(model='claude-sonnet-4-6')

# rag_chain = prompt | llm | StrOutputParser()

# def rag_answer(question):
#     docs = retriever.invoke(question)
#     context = '\n\n'.join(doc.page_content for doc in docs)
#     return rag_chain.invoke({'context': context, 'question': question})

# === 向量数据库选型 ===
# Chroma: 轻量，适合开发
# FAISS: Meta开源，纯内存，速度最快
# Milvus: 分布式，适合大规模生产
# Pinecone: 全托管云服务
# Qdrant: Rust实现，性能好

# === 高级 RAG 技巧 ===

# 查询改写
rewrite_prompt = """将以下问题改写为更适合检索的查询：
原始问题：{question}
检索查询："""

# 多路召回 + 重排
from langchain.retrievers import EnsembleRetriever
# ensemble = EnsembleRetriever(
#     retrievers=[bm25_retriever, vector_retriever],
#     weights=[0.4, 0.6]
# )

# 重排器（Cohere Rerank / BGE-Reranker）
# from langchain.retrievers.document_compressors import CohereRerank
# compressor = CohereRerank()
# compression_retriever = ContextualCompressionRetriever(compressor, base_retriever)
```

| RAG 组件 | 选型 | 推荐方案 |
|----------|------|---------|
| 文本切分 | RecursiveCharacterTextSplitter | chunk_size=500, overlap=50 |
| Embedding | text2vec-base-chinese | 中文场景 |
| 向量库 | Chroma/FAISS/Milvus | 开发→Chroma，生产→Milvus |
| LLM | Claude/GPT/开源模型 | 精度→Claude，隐私→开源 |
| 检索策略 | 语义/关键词/混合 | 混合检索最稳 |

### 3. LangChain 应用框架

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.output_parsers import StrOutputParser
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

# === 基础链 ===
llm = ChatAnthropic(model='claude-sonnet-4-6')

prompt = ChatPromptTemplate.from_messages([
    ('system', '你是一个有帮助的AI助手。'),
    ('user', '{input}')
])

chain = prompt | llm | StrOutputParser()
result = chain.invoke({'input': '什么是RAG？'})
print(result)

# === 带记忆的对话 ===
from langchain_core.messages import HumanMessage, AIMessage
from langgraph.graph import StateGraph, MessagesState

# 使用 LangGraph（LangChain 新推荐）
def chatbot(state: MessagesState):
    return {'messages': llm.invoke(state['messages'])}

graph = StateGraph(MessagesState)
graph.add_node('chatbot', chatbot)
graph.set_entry_point('chatbot')
graph.set_finish_point('chatbot')
app = graph.compile()

# 对话
config = {'configurable': {'thread_id': '1'}}
response1 = app.invoke({'messages': [HumanMessage('我叫小明')]}, config)
response2 = app.invoke({'messages': [HumanMessage('我叫什么？')]}, config)
# 模型会记住"小明"

# === 工具调用 ===
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """获取指定城市的天气"""
    # 实际应用中调用天气 API
    weather_data = {'北京': '晴天 25°C', '上海': '多云 22°C'}
    return weather_data.get(city, '未知城市')

@tool
def calculate(expression: str) -> str:
    """计算数学表达式"""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"计算错误: {e}"

tools = [get_weather, calculate]
llm_with_tools = llm.bind_tools(tools)

# === Agent ===
from langgraph.prebuilt import create_react_agent

agent = create_react_agent(llm, tools)
result = agent.invoke({'messages': [HumanMessage('北京今天天气怎么样？')]})
print(result['messages'][-1].content)

# === Output Parser ===
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field

class MovieReview(BaseModel):
    title: str = Field(description="电影名称")
    rating: int = Field(description="评分1-10")
    summary: str = Field(description="一句话评价")

parser = JsonOutputParser(pydantic_object=MovieReview)

prompt = ChatPromptTemplate.from_messages([
    ('user', '评价以下电影：{movie}\n\n{format_instructions}')
])

chain = prompt | llm | parser
result = chain.invoke({
    'movie': '肖申克的救赎',
    'format_instructions': parser.get_format_instructions()
})
print(result)
```

### 4. 微调（Fine-tuning）

```python
# === LoRA 微调（参数高效） ===
from peft import LoraConfig, get_peft_model, TaskType
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = 'Qwen/Qwen2.5-7B'
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,
    device_map='auto'
)

# LoRA 配置
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,                    # LoRA 秩
    lora_alpha=32,           # 缩放因子
    lora_dropout=0.05,
    target_modules=['q_proj', 'k_proj', 'v_proj', 'o_proj']  # 注意力层
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable: 0.1% 的参数

# 训练数据格式
train_data = [
    {'instruction': '将以下文本翻译为英文', 'input': '今天天气很好', 'output': 'The weather is nice today'},
    {'instruction': '总结以下段落', 'input': '长文本...', 'output': '简短摘要...'},
]

# 使用 SFTTrainer
from trl import SFTTrainer, SFTConfig

sft_config = SFTConfig(
    output_dir='./lora_output',
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    logging_steps=10,
    save_steps=100,
    fp16=True,
    max_seq_length=512,
)

# trainer = SFTTrainer(
#     model=model,
#     train_dataset=train_dataset,
#     processing_class=tokenizer,
#     args=sft_config,
# )
# trainer.train()

# === 保存和加载 LoRA ===
# model.save_pretrained('./lora_output')

# 推理时合并
from peft import PeftModel
base_model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.float16, device_map='auto')
model = PeftModel.from_pretrained(base_model, './lora_output')
model = model.merge_and_unload()  # 合并权重，推理更快
```

| 微调方法 | 可训练参数 | 显存需求 | 效果 | 适用 |
|----------|-----------|---------|------|------|
| 全量微调 | 100% | 极高 | 最好 | 大数据+强需求 |
| LoRA | 0.1-1% | 低 | 接近全量 | 推荐默认 |
| QLoRA | 0.1-1% | 极低 | 略低于LoRA | 显存有限 |
| Prompt Tuning | <0.01% | 极低 | 较差 | 轻量适配 |

### 5. 向量数据库

```python
# === FAISS（纯内存，最快） ===
import faiss
import numpy as np

rng = np.random.default_rng(42)
dimension = 768
n_documents = 10000

# 生成模拟向量
vectors = rng.standard_normal((n_documents, dimension)).astype('float32')
faiss.normalize_L2(vectors)  # 归一化用于余弦相似度

# 创建索引
index = faiss.IndexFlatIP(dimension)  # 内积=余弦（归一化后）
index.add(vectors)

# 检索
query = rng.standard_normal((1, dimension)).astype('float32')
faiss.normalize_L2(query)
scores, indices = index.search(query, k=5)
print(f"Top5 索引: {indices}, 分数: {scores}")

# IVF 索引（大数据集）
nlist = 100
quantizer = faiss.IndexFlatIP(dimension)
index_ivf = faiss.IndexIVFFlat(quantizer, dimension, nlist)
index_ivf.train(vectors)
index_ivf.add(vectors)
index_ivf.nprobe = 10  # 搜索时探测的聚类数
scores, indices = index_ivf.search(query, k=5)

# === Chroma（LangChain 集成好） ===
from langchain_community.vectorstores import Chroma
from langchain_community.embeddings import HuggingFaceEmbeddings

embedding = HuggingFaceEmbeddings(model_name='shibing624/text2vec-base-chinese')

# texts = ['文档1内容', '文档2内容', ...]
# metadatas = [{'source': 'file1'}, {'source': 'file2'}, ...]
# vectorstore = Chroma.from_texts(texts, embedding, metadatas=metadatas, persist_directory='./db')

# 检索
# results = vectorstore.similarity_search('查询文本', k=3)
# results = vectorstore.similarity_search_with_score('查询文本', k=3)
```

### 6. LLM 应用评估

```python
# === RAG 评估 ===

# 评估维度：上下文相关性、答案忠实度、答案相关性

# 1. 人工评估黄金标准
# 2. LLM-as-Judge

judge_prompt = """请评估以下RAG回答的质量：

问题：{question}
检索上下文：{context}
模型回答：{answer}

请从以下维度评分（1-5分）：
1. 忠实度：回答是否仅基于上下文，没有编造信息？
2. 相关性：回答是否直接回应了问题？
3. 完整性：回答是否完整覆盖了问题？

输出JSON格式：
{{"faithfulness": X, "relevance": X, "completeness": X, "reason": "..."}}"""

# === RAGAS 框架 ===
# from ragas import evaluate
# from ragas.metrics import faithfulness, answer_relevancy, context_relevancy
#
# result = evaluate(
#     dataset=eval_dataset,
#     metrics=[faithfulness, answer_relevancy, context_relevancy]
# )

# === A/B 测试不同 Prompt ===
import json

def evaluate_prompt(prompt_variant, test_cases):
    results = []
    for case in test_cases:
        answer = ask(prompt_variant.format(**case))
        results.append({
            'question': case['question'],
            'answer': answer,
            'expected': case.get('expected', '')
        })
    return results

test_cases = [
    {'question': 'Python是什么？', 'expected': '编程语言'},
    {'question': 'RAG是什么？', 'expected': '检索增强生成'},
]
```

### 7. 开源 LLM 生态

```python
# === vLLM（高吞吐推理） ===
# pip install vllm
# 启动服务：
# python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen2.5-7B --port 8000

# 客户端调用（兼容 OpenAI API）
from openai import OpenAI
client = OpenAI(base_url='http://localhost:8000/v1', api_key='empty')
response = client.chat.completions.create(
    model='Qwen/Qwen2.5-7B',
    messages=[{'role': 'user', 'content': '你好'}],
    max_tokens=256
)
print(response.choices[0].message.content)

# === Ollama（本地运行最简单） ===
# 安装: curl -fsSL https://ollama.com/install.sh | sh
# 拉模型: ollama pull qwen2.5:7b
# 运行: ollama run qwen2.5:7b

# Python 调用
# import ollama
# response = ollama.chat(model='qwen2.5:7b', messages=[{'role': 'user', 'content': '你好'}])
# print(response['message']['content'])

# === Hugging Face Transformers ===
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = 'Qwen/Qwen2.5-1.5B'  # 小模型演示
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name, torch_dtype=torch.float16, device_map='auto'
)

messages = [{'role': 'user', 'content': '什么是机器学习？'}]
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(text, return_tensors='pt').to(model.device)

with torch.no_grad():
    outputs = model.generate(**inputs, max_new_tokens=256, temperature=0.7)

response = tokenizer.decode(outputs[0][inputs['input_ids'].shape[1]:], skip_special_tokens=True)
print(response)
```

| 工具 | 定位 | 推理方式 | 适用场景 |
|------|------|---------|---------|
| vLLM | 高吞吐推理 | PagedAttention | 生产服务 |
| Ollama | 本地运行 | GGUF量化 | 个人开发 |
| llama.cpp | C++推理 | GGUF量化 | CPU/边缘 |
| Transformers | 研究/实验 | PyTorch | 原型开发 |
| TGI | 生产服务 | FlashAttention | Docker部署 |

### 8. LLM 应用架构

```
用户请求
   ↓
API Gateway（限流、认证）
   ↓
┌─────────────────────────┐
│     应用层               │
│  ┌───────┐  ┌────────┐  │
│  │ Prompt │  │ Agent  │  │
│  │ 编排   │  │ 规划器  │  │
│  └───┬───┘  └───┬────┘  │
│      ↓          ↓        │
│  ┌─────────────────┐    │
│  │   LLM 调用层    │    │
│  │ (路由/缓存/重试) │    │
│  └────────┬────────┘    │
│           ↓              │
│  ┌──── ┌──── ┌──── ┐    │
│  │RAG  │工具 │记忆  │    │
│  │检索 │调用 │管理  │    │
│  └──── └──── └──── ┘    │
└─────────────────────────┘
   ↓          ↓
向量库      外部API
```

## 面试题

### 1. RAG 和微调分别适合什么场景？

RAG 适合：知识频繁更新（新闻、政策）、需要引用来源、幻觉代价高、数据量大无法全部微调。微调适合：需要改变模型的行为模式（风格、格式、领域术语）、任务固定不需要外部知识、需要更低的推理延迟（RAG 多一次检索）。两者可以结合：先微调让模型理解领域，再用 RAG 注入最新知识。微调是改变模型"怎么说话"，RAG 是改变模型"知道什么"。

### 2. LoRA 的原理和优势？

LoRA（Low-Rank Adaptation）在原始权重矩阵旁添加低秩分解矩阵 ΔW = A×B，其中 A 是 d×r，B 是 r×d，r 远小于 d。训练时只更新 A 和 B，原始权重冻结。优势：（1）参数效率——仅训练 0.1% 的参数，单卡即可微调 7B 模型；（2）可插拔——不同任务的 LoRA 适配器可以热切换，无需多份基础模型；（3）无推理延迟——合并后 ΔW + W 与原始权重形状相同。QLoRA 在此基础上将基础模型量化为 4-bit，进一步降低显存需求（7B模型仅需6GB显存微调）。

### 3. Prompt 工程中如何减少幻觉？

六种策略：（1）**基于上下文回答**——明确要求"仅基于以下信息回答，不确定时说不知道"；（2）**提供参考资料**——RAG 检索相关文档作为上下文；（3）**思维链推理**——要求模型先分析再给出结论，中间步骤便于发现错误；（4）**自我验证**——让模型检查自己的答案是否合理；（5）**降低温度**——temperature=0 减少随机性，选最可能的输出；（6）**结构化输出**——要求 JSON 等格式，减少自由生成的空间。关键是设定"不知道就说不知道"的边界，而非让模型强行回答。

### 4. 向量数据库的选择标准？

五个维度：（1）**规模**——百万级以下用 FAISS/Chroma，千万级以上用 Milvus/Qdrant；（2）**部署方式**——开发用 Chroma（嵌入式），生产用 Milvus（分布式）或 Pinecone（全托管）；（3）**检索类型**——纯语义用 ANN 索引（HNSW/IVF），需要关键词用混合检索（BM25+向量）；（4）**延迟要求**——FAISS 纯内存最快，Milvus 分布式有网络开销；（5）**功能**——元数据过滤、多向量、更新删除等操作需求。实践建议：开发阶段用 Chroma 快速验证，上线后迁移到 Milvus/Pinecone。

### 5. Agent 的核心原理是什么？

Agent 让 LLM 自主规划和使用工具解决复杂任务。核心循环：感知（接收输入）→ 规划（分解任务）→ 行动（调用工具）→ 观察（获取结果）→ 循环直到完成。关键技术：（1）**工具调用**——将函数签名和描述告诉 LLM，让其决定何时调用；（2）**ReAct 模式**——交替进行推理（Thought）和行动（Action）；（3）**记忆**——短期记忆（对话历史）+ 长期记忆（向量库）；（4）**规划**——将复杂任务分解为可执行的子任务链。挑战：错误累积（一步错步步错）、循环调用（无限循环）、成本控制（多次 LLM 调用）。

### 6. 如何评估 LLM 应用的质量？

四个维度：（1）**准确性**——答案是否正确，可用人工标注+LLM-as-Judge；（2）**忠实度**——是否基于提供的上下文，没有幻觉，RAGAS 的 faithfulness 指标；（3）**相关性**——是否直接回应用户意图，answer_relevancy 指标；（4）**延迟与成本**——首 token 延迟（TTFT）、总延迟、token 消耗。评估方法：黄金集人工评估最可靠但成本高；LLM-as-Judge 可规模化但有偏见；自动指标（BLEU/ROUGE）与人类判断相关性低。建议：核心场景用人工评估，日常迭代用 LLM-Judge + 自动指标组合。

### 7. 如何优化 LLM 的推理成本？

五种策略：（1）**模型选择**——简单任务用小模型（Haiku），复杂任务用大模型（Sonnet/Opus）；（2）**缓存**——缓存相似查询的结果，减少重复调用；语义缓存用向量相似度匹配；（3）**批量处理**——非实时请求合并为 batch，利用 GPU 并行；（4）**Prompt 压缩**——精简 Prompt 长度，减少输入 token 数；（5）**量化部署**——开源模型用 INT4/INT8 量化，自部署比 API 调用更便宜。实践中，模型路由（Model Router）是最有效的：训练一个小分类器判断请求复杂度，简单请求走小模型，复杂请求走大模型，可降低 50-70% 成本。

### 8. LangChain 和 LlamaIndex 的区别？

LangChain 是通用 LLM 应用框架，提供链式调用、Agent、记忆、工具集成等全套工具，适合构建复杂的 Agent 应用。LlamaIndex 专注于数据索引和检索，在文档解析、分块策略、索引构建、查询引擎方面做得更深，适合 RAG 场景。实践中两者经常配合使用：LlamaIndex 负责数据接入和检索，LangChain 负责 Agent 编排和工具调用。新项目推荐 LangGraph（LangChain 团队新框架，基于图的状态管理，更适合复杂 Agent 流程）。

## 外部参考

- [LangChain 文档](https://python.langchain.com/)
- [Hugging Face Transformers](https://huggingface.co/docs/transformers/)

---
title: fireworks-tech-graph：用自然语言生成工业级架构图，Claude Code 绘图神器！
author: AI开源提效指南
date: 
url: https://mp.weixin.qq.com/s?__biz=MzY5NzIxODM2MQ==&mid=2247484388&idx=1&sn=d2f897dce455cb4abdcaa39f7f91a864&chksm=f534b8726c9cf245c119042de4bcb583dc06727e1e3574faff370a2ab85ffb5a0d1f7a9c49d4&mpshare=1&scene=24&srcid=0412by2RnqM1B0Vo0rF3IEbp&sharer_shareinfo=34988ee5d76ef9583438ae22e6c4ff42&sharer_shareinfo_first=34988ee5d76ef9583438ae22e6c4ff42#rd
---

# 大家好！这里是 AI开源提效指南！

fireworks-tech-graph 真的是架构师、程序员的福音 —— 再也不用在 Mermaid 的语法和 draw.io 的手动操作之间纠结了。只需用自然语言描述你的系统，就能获得专业级的 SVG + PNG 架构图。

**核心优势：**

* ✅ 5 种精美视觉风格，从简洁白到暗黑霓虹到磨砂玻璃
* ✅ 8 种图表类型覆盖所有技术场景
* ✅ 内置 AI/Agent 领域知识，理解 RAG、Mem0、多 Agent 等模式
* ✅ 语义化的形状和箭头系统，自动编码含义
* ✅ 40+ 产品图标，品牌颜色精准匹配
* ✅ SVG + PNG 双输出，SVG 可编辑，PNG 可直接嵌入

## 🚀 核心功能

### 5 种视觉风格

| # | 风格名称 | 背景 | 字体 | 适用场景 |
| --- | --- | --- | --- | --- |
| 1 | Flat Icon (默认) | #ffffff | Helvetica | 博客、幻灯片、文档 |
| 2 | Dark Terminal | #0f0f1a | SF Mono / Fira Code | GitHub README、开发者文章 |
| 3 | Blueprint | #0a1628 | Courier New | 架构文档、工程图 |
| 4 | Notion Clean | #ffffff | system-ui | Notion、Confluence、Wiki |
| 5 | Glassmorphism | #0d1117 渐变 | Inter | 产品网站、演讲 |

每种风格在 `references/` 目录下都有专属参考文件，包含精确的颜色 Token、SVG 模板和使用规范。

### 8 种图表类型

| 类型 | 描述 | 关键布局规则 |
| --- | --- | --- |
| **架构图** | 服务、组件、云基础设施 | 水平分层，自上而下 |
| **数据流图** | 数据在系统中的流向 | 每条箭头标注数据类型 |
| **流程图** | 决策树、流程步骤 | 菱形 = 决策，自上而下 |
| **Agent 架构图** | LLM + 工具 + 记忆 | 五层模型：输入/Agent/记忆/工具/输出 |
| **记忆架构图** | Mem0、MemGPT 风格 | 读/写路径分离，记忆层级分明 |
| **序列图** | API 调用链、时序交互 | 垂直生命线，水平消息箭头 |
| **对比图** | 功能矩阵、方案比较 | 列 = 系统，行 = 属性 |
| **思维导图** | 概念地图、发散思维 | 中心节点，贝塞尔曲线分支 |

### AI/Agent 领域内置模式

* **RAG Pipeline** → `Query` → `Embed` → `VectorSearch` → `Retrieve` → `LLM` → `Response`
* **Agentic RAG** → `添加 Agent 循环` + `工具使用`
* **Agentic Search** → `Query` → `Planner` → `[Search/Calc/Code]` → `Synthesizer`
* **Mem0 Memory Layer** → `Input` → `Memory Manager` → `[VectorDB + GraphDB]` → `Context`
* **Agent Memory Types** → `Sensory` → `Working` → `Episodic` → `Semantic` → `Procedural`
* **Multi-Agent** → `Orchestrator` → `[SubAgent×N]` → `Aggregator` → `Output`
* **Tool Call Flow** → `LLM` → `Tool Selector` → `Execution` → `Parser` → `LLM` (循环)

## 🎯 为什么选择 fireworks-tech-graph？

### 对比 Mermaid 和 draw.io

| 特性 | Mermaid | draw.io | fireworks-tech-graph |
| --- | --- | --- | --- |
| 自然语言输入 | ❌ | ❌ | ✅ |
| AI/Agent 领域模式 | ❌ | ❌ | ✅ 内置 |
| 多种视觉风格 | ❌ | 手动 | ✅ 5 种内置 |
| 高分辨率 PNG 导出 | ❌ | 手动 | ✅ 自动 1920px |
| 语义箭头颜色 | ❌ | 手动 | ✅ 自动 |
| 无需在线工具 | ✅ | ❌ | ✅ |

* **Mermaid** 适合 Markdown 中的快速内联图表。
* **draw.io** 适合手动精修。
* **fireworks-tech-graph** 专为描述系统并立即获得精美图表而优化，无需编写 DSL 语法或在 GUI 中点击。

## 📦 安装方法

### 方式一：Claude Skills 安装（推荐）

```
claude skills install fireworks-tech-graph
```

### 方式二：手动克隆

```
git clone https://github.com/yizhiyanhua-ai/fireworks-tech-graph.git ~/.claude/skills/fireworks-tech-graph
```

### 依赖安装

```
# macOS  
brew install librsvg  
  
# Ubuntu/Debian  
sudo apt install librsvg2-bin  
  
# 验证安装  
rsvg-convert --version
```

## 💡 使用示例

### 触发词

以下关键词会自动触发 Skill：

```
画图 / 帮我画 / 生成图 / 做个图 / 架构图 / 流程图 / 可视化一下 / 出图  
generate diagram / draw diagram / create chart / visualize
```

### 基本用法

```
画一张 RAG 流程图
```

```
生成一张 Agentic Search 架构图
```

### 指定风格

```
画一张微服务架构图，风格2（暗黑极客风）
```

```
生成 Multi-Agent 协作图，玻璃态风格
```

### 指定输出路径

```
生成 Mem0 架构图，输出到 ~/Desktop/
```

```
画一张 Tool Call 流程图 --output /tmp/diagrams/
```

---

## 🎨 场景示例

### AI/Agent 系统

```
画一张 Agentic RAG 和普通 RAG 的对比图，用 Notion 极简风
```

→ 功能矩阵对比：检索策略、Agent 循环、工具调用、延迟、成本

```
生成一张 Mem0 记忆架构图，包含向量库、图数据库、KV 存储和记忆管理器
```

→ 分泳道记忆架构：Input → Memory Manager → 存储层 → 检索输出

```
画一张 Multi-Agent 协作图：Orchestrator 调度 3 个 SubAgent（搜索/计算/代码执行），汇聚到 Aggregator
```

→ Agent 架构，六边形节点 + 工具层 + 结果聚合

```
可视化一下 Tool Call 的执行流程：LLM → Tool Selector → Execution → Parser → 回到 LLM
```

→ 含决策循环的流程图，展示工具调用的完整生命周期

```
画一张 Agent 的 5 种记忆类型图：感知记忆、工作记忆、情景记忆、语义记忆、程序记忆
```

→ 思维导图或分层架构，从感官输入到程序技能的记忆层级

### 基础设施与云架构

```
帮我画一张微服务架构图：Client → API Gateway → [用户服务 / 订单服务 / 支付服务] → PostgreSQL + Redis
```

→ 水平分层架构，每个服务集群一个泳道

```
生成一张数据管道图：Kafka 消费数据 → Spark 处理 → 写入 S3 → Athena 查询
```

→ 数据流图，每条箭头标注数据类型（stream / batch / query）

```
画一张 Kubernetes 部署架构：Ingress → Service → [Pod × 3] → ConfigMap + PersistentVolume
```

→ 架构图，Namespace 用虚线框，流量用实线箭头

### API 与时序流程

```
画一张 OAuth2 授权码流程的序列图：用户 → 客户端 → 授权服务器 → 资源服务器
```

→ 序列图，垂直生命线 + 激活框

```
帮我画一张 ChatGPT Plugin 的调用时序图
```

→ 时序：User → ChatGPT → Plugin Manifest → API → 响应链

### 决策与流程图

```
画一张 AI 应用上线前的质检流程图：代码审查 → 安全扫描 → 性能测试 → 人工审核 → 发布
```

→ 流程图，含菱形决策节点和并行分支

```
生成一张 RAG vs Fine-tuning vs Prompt Engineering 的功能对比图
```

→ 功能矩阵，对比成本、延迟、准确率、灵活性

### 概念图与知识图谱

```
帮我可视化一下 LLM 应用的技术栈：从底层模型到 SDK 到应用框架到部署层
```

→ 分层架构图或思维导图，从模型层到产品层

```
画一张 AI Agent 的核心能力地图：感知 / 记忆 / 推理 / 行动 / 学习
```

→ 以"AI Agent"为中心的放射状思维导图，5 个核心能力分支

---

## 🔧 语义形状词汇

形状在所有风格中保持一致的语义：

| 概念 | 形状 |
| --- | --- |
| 用户 / 人类 | 圆形 + 身体路径 |
| LLM / 模型 | 圆角矩形，双边框，⚡ |
| Agent / 编排器 | 六边形 |
| 短期记忆 | 虚线边框圆角矩形 |
| 长期记忆 | 实线圆柱体 |
| Vector Store | 带内环圆柱 |
| Graph DB | 三圆簇 |
| 工具 / 函数 | 带 ⚙ 的矩形 |
| API / 网关 | 六边形（单边框） |
| 消息队列 / 流 | 横向管道 |
| 文档 / 文件 | 折角矩形 |
| 浏览器 / UI | 带三点标题栏的矩形 |
| 决策节点 | 菱形 |
| 外部服务 | 虚线边框矩形 |

### ➡️ 箭头语义

| 流类型 | 线宽 | 虚线 | 含义 |
| --- | --- | --- | --- |
| 主数据流 | 2px 实线 | — | 主要请求/响应路径 |
| 控制 / 触发 | 1.5px 实线 | — | 系统 A 触发 B |
| 记忆读取 | 1.5px 实线 | — | 从存储检索 |
| 记忆写入 | 1.5px | `5,3` | 写入/存储操作 |
| 异步 / 事件 | 1.5px | `4,2` | 非阻塞 |
| 反馈 / 循环 | 1.5px 曲线 | — | 迭代推理 |

## 🎯 支持的产品图标

* **AI/ML**: OpenAI, Anthropic/Claude, Google Gemini, Meta LLaMA, Mistral, Cohere, Groq, Hugging Face
* **AI 框架**: Mem0, LangChain, LlamaIndex, LangGraph, CrewAI, AutoGen, DSPy, Haystack
* **向量数据库**: Pinecone, Weaviate, Qdrant, Chroma, Milvus, pgvector, Faiss
* **数据库**: PostgreSQL, MySQL, MongoDB, Redis, Elasticsearch, Neo4j, Cassandra
* **消息队列**: Kafka, RabbitMQ, NATS, Pulsar
* **云服务**: AWS, GCP, Azure, Cloudflare, Vercel, Docker, Kubernetes
* **可观测性**: Grafana, Prometheus, Datadog, LangSmith, Langfuse, Arize

## 🐛 常见问题

| 问题 | 原因 | 解决方案 |
| --- | --- | --- |
| PNG 空白或全黑 | SVG 中的 @import url() — rsvg-convert 无法获取字体 | 移除 @import，使用系统字体栈 |
| 未生成 PNG | 未安装 rsvg-convert | brew install librsvg (macOS) 或 apt install librsvg2-bin |
| 图表底部被截断 | ViewBox 高度太短 | 增加 viewBox="0 0 960" 中的高度 |
| 文本溢出框 | 标签太长 | 添加 text-anchor="middle" +或缩短标签 |
| 图标不渲染 | rsvg-convert 上下文中的外部 CDN URL | 使用 references/icons.md 中的内联 SVG 路径 |

## 📁 项目结构

```
fireworks-tech-graph/  
├── SKILL.md                      # 主技能文件 — 图表类型、布局规则、形状词汇  
├── README.md                     # 英文文档  
├── README.zh.md                  # 中文文档  
├── references/  
│   ├── style-1-flat-icon.md      # 白色背景，彩色强调  
│   ├── style-2-dark-terminal.md  # 暗黑背景，霓虹强调，等宽字体  
│   ├── style-3-blueprint.md      # 蓝图网格，青色线条  
│   ├── style-4-notion-clean.md   # 极简，白色，单箭头颜色  
│   ├── style-5-glassmorphism.md  # 暗黑渐变，磨砂玻璃卡片  
│   └── icons.md                  # 40+ 产品图标 + 语义形状  
└── assets/  
    └── samples/                  # 示例图表 PNG
```

## 🔗 参考资源

```
- GitHub 仓库：https://github.com/yizhiyanhua-ai/fireworks-tech-graph  
- Claude Code Skills: https://claude.ai/code
```

---

**🎯****觉得这份工具干货有用？不妨这样做**

* ⭐ 星标 / 置顶公众号，**第一时间解锁最新工具分享！**
* ✅ **点赞**「**推荐**」，让更多技术伙伴发现优质干货！
* 🔗 **转发**给团队小伙伴，一起高效提效！
* 💬 **底部留言区**，告诉我你想找的工具/项目方向！

**📬 长期追踪优质开源工具**

* 关注「**AI 开源提效指南**」｜日更开源神器，玩转技术提效！
* 回复 **【容器加速器】**，即刻开启你的高效探索之旅～
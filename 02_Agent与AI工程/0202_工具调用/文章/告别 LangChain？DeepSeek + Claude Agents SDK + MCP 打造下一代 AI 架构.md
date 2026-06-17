---
title: 告别 LangChain？DeepSeek + Claude Agents SDK + MCP 打造下一代 AI 架构
author: 光影织梦
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5OTE0OTM4NQ==&mid=2247484947&idx=1&sn=3329816a943e5e69ac9e659c10091f4d&chksm=9761a0ff30a2612695fd37abb664aee62fb30bd3165e0ace194886345a30de805ae65139d7cd&mpshare=1&scene=24&srcid=1211bFFYqh948Taa6kq1oilz&sharer_shareinfo=0843df5c4d465cac649aa2b6f5602200&sharer_shareinfo_first=0843df5c4d465cac649aa2b6f5602200#rd
---

在 AI Agent 开发的 “战国时代”，开发者往往面临两难：是忍受 LangChain 日益臃肿的抽象层，还是手写胶水代码？是支付昂贵的 GPT-4/Claude API 费用，还是在此消彼长的开源模型中徘徊？

今天，我们要介绍一种 “破坏性创新” 的架构组合：**DeepSeek v3.2 + Claude Agents SDK + MongoDB MCP**。这套方案不仅极具性价比，更重要的是，它靠**子智能体**破解了大模型 “上下文衰减” 的核心痛点，展示了下一代 AI 应用的标准形态 ——**模型去魅，架构为王，接口统一**。

---

## 🚀 为什么是这个组合？

这套架构之所以能被称为 “下一代”，是因为它精准地选择了每一层级的 “最佳实践”，解决了一个核心痛点：**如何用低成本的开源模型，跑通企业级复杂的 Agent 逻辑。**

### 1. 大脑：DeepSeek v3.2（开源最强音）

DeepSeek v3.2 近期在 Hugging Face 发布模型集合地址，其推理能力不仅对标 GPT-4o，甚至在某些场景下足以叫板 Claude Opus 4.5。

•**核心骚操作：** 利用 DeepSeek 对 Anthropic API 的兼容性配置指南，我们不需要重写代码，只需将 `BASE_URL` 指向 DeepSeek，就能让上层应用 “以为” 自己在调用 Claude，实则享受 DeepSeek 的极致性价比。

### 2. 骨架：Claude Agents SDK（原生脚手架）

既然要 “告别 LangChain”，替代品必须足够强大。Claude Agents SDK（前身是 Claude Code SDK）是 Anthropic 官方为其编程助手 Claude Code 打造的底层框架 —— 这意味着它与 Claude Code 共享完全一致的运行环境（Harness），且**原生支持子智能体（Sub-agents）**，这是破解 “上下文衰减” 的关键前提。

•**优势：** 它没有 LangChain 过重的抽象包袱，经过数百万用户实战验证（Claude Code 正在使用），更内置了 Anthropic 沉淀的 上下文工程技巧。而子智能体的原生支持，让它能直接承载 “分而治之” 的架构思路，从框架层面规避上下文臃肿问题。

### 3. 手眼：MongoDB MCP Server（标准接口）

MCP（Model Context Protocol）正在迅速成为 AI 连接工具的 “USB 标准”—— 类似 HTTP 规范浏览器与服务器的通信，MCP 规范了 Agent 与工具的交互方式。

•**变革：** 以前需手写 Python 函数执行数据库操作，现在通过 MongoDB 官方 MCP 服务器（默认提供 26 种工具），Agent 可标准化地分析 Schema、查询数据、建立索引。更重要的是，MCP 的工具化设计，让子智能体 “按需拿取工具” 成为可能，进一步减少上下文冗余。

---

## 🧩 架构核心：子智能体 —— 破解 “上下文衰减” 的唯一解

这是本方案最核心的技术突破：**用子智能体对抗大模型 “上下文衰减”（Context Rot），这也是当前解决该问题的主流方案**。

### 先看清痛点：上下文衰减有多致命？

尽管 DeepSeek、GPT-5、Gemini 3 等大模型厂商宣称支持 20 万至百万级令牌的上下文窗口，但在实际工程中，**当上下文令牌量超过 10 万时，模型性能会显著下降**：原本精准的工具调用开始出错（比如用 “删除工具” 执行查询）、对复杂任务的理解出现偏差、甚至生成与需求无关的 “幻觉内容”。

这种 “上下文装得越多，模型越笨” 的现象，被称为 “上下文衰减”，最早由 Chroma 研究团队 系统性提出。其根源在于：大模型的注意力资源有限，当上下文塞满工具定义、历史对话、无关数据时，它无法聚焦核心任务，自然会 “决策混乱”。

### 再看方案：子智能体如何 “物理隔离” 上下文？

我们放弃构建 “全能上帝 Agent”，转而设计 **3 个专注单一任务的子智能体**，配合主智能体（Orchestrator，协调智能体）形成 “分工网络”。这种架构的核心价值，在于**从物理层面避免主智能体的上下文被污染**：

1.**减轻主智能体负担**：主智能体只负责 “任务分发”，不需要记忆数据库 Schema、不需要存储 26 种 MCP 工具的定义，上下文始终保持 “轻量化”，决策更精准；2.**子智能体 “专而精”**：每个子智能体只加载完成自身任务所需的工具（比如 “只读智能体” 不会接触 “删除工具”），上下文仅包含 “任务指令 + 必要工具”，彻底规避 “工具冗余导致的注意力分散”；3.**任务隔离不串扰**：子智能体解决子问题时产生的中间数据（如 Schema 分析结果），不会涌入主智能体的上下文，主智能体只需接收最终结果，进一步减少上下文占用。

这种思路也被PhilSchmid的《子智能体的崛起》一文验证为：“处理复杂 Agent 任务最稳健、最可落地的架构模式”。

### 🛠️ 子智能体分工矩阵：每一步都为对抗衰减设计

| 智能体 | 角色 | 专属工具集（仅加载这些，减少上下文） | 核心价值（对抗上下文衰减） |
| --- | --- | --- | --- |
| **Reader Agent** | 只读专家 | `list-collections` , `collection-schema`, `count-documents` | 仅处理 “看数据” 任务，上下文无写入工具定义，避免干扰 |
| **Writer Agent** | 操作员 | `insert-many` , `update-one`, `delete-many`, `create-index` | 仅处理 “改数据” 任务，不加载查询工具，专注数据完整性 |
| **Query Agent** | 分析师 | `find` , `aggregate`, `distinct` | 仅处理 “查数据” 任务，上下文聚焦检索逻辑，不被读写操作分散 |

## 💻 实战：从环境配置到代码落地

想要复刻这套系统，核心在于 “欺骗” Claude SDK 转向 DeepSeek 端点，同时通过代码确保子智能体的 “工具隔离”。以下是完整步骤：

### Step 1: 安装依赖（高效工具推荐）

推荐使用 `uv`（比 pip 更快的 Python 包管理器）安装依赖：

```
# 同步项目依赖（自动安装 Claude Agents SDK、MongoDB 相关库等）  
uv sync
```

### Step 2: 环境配置 (.env)

这是整个方案的 “魔法核心”—— 通过环境变量让 Claude SDK 流量转向 DeepSeek。首先复制示例配置文件：

```
cp .env.example .env
```

然后编辑 `.env` 填入关键信息：

```
# === 核心：DeepSeek 伪装 Claude 配置 ===  
# 将 Claude API 端点指向 DeepSeek 兼容接口  
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic  
# 替换为你的 DeepSeek API 密钥（从 DeepSeek 控制台获取）  
ANTHROPIC_AUTH_TOKEN=sk-your-deepseek-api-key  
# 指定使用 DeepSeek 模型（SDK 会自动适配）  
ANTHROPIC_MODEL=deepseek-chat  
ANTHROPIC_SMALL_FAST_MODEL=deepseek-chat  
# 关闭非必要遥测，提升响应速度  
CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1  
# === MongoDB 配置 ===  
# 替换为 Step 1 中获取的连接字符串  
MONGODB_CONNECTION_STRING=mongodb+srv://your-username:your-password@your-cluster.mongodb.net/
```

### Step 3: 定义子智能体与 MCP 服务器（Python）

**代码的核心是 “严格给子智能体分配工具”**，确保每个子智能体只加载对应工具，避免上下文冗余。支持两种 MCP 配置方式：

#### 方式 1：环境变量配置 MCP（推荐，便于保密）

```
from anthropic_agents import ClaudeAgentOptions, AgentDefinition, McpStdioServerConfig  
from dotenv import load_dotenv  
import os  
# 加载 .env 文件  
load_dotenv()  
connection_string = os.getenv("MONGODB_CONNECTION_STRING")  
options = ClaudeAgentOptions(  
    # 1. 子智能体集群：严格控制每个智能体的工具集（对抗上下文衰减的关键）  
    agents={  
        "database_reader": AgentDefinition(  
            description="Only read MongoDB structure/statistics (no write)",  
            prompt="You are a read-only expert. Only use list/analyze tools. Never modify data.",  
            tools=["list-collections", "collection-schema", "count-documents"],  # 仅3个工具  
            model="sonnet"  # 实际路由到 DeepSeek  
        ),  
        "database_writer": AgentDefinition(  
            description="Only write/update MongoDB data (no complex query)",  
            prompt="You are a write expert. Only use insert/update/delete tools. Check data integrity first.",  
            tools=["insert-many", "update-one", "create-index"],  # 仅3个工具（刻意排除查询工具）  
            model="sonnet"  
        ),  
        "database_query": AgentDefinition(  
            description="Only query MongoDB for user needs (no write)",  
            prompt="You are a query expert. Only use find/aggregate tools. Verify field names first.",  
            tools=["find", "aggregate", "distinct"],  # 仅3个工具  
            model="sonnet"  
        )  
    },  
    # 2. 挂载 MongoDB MCP 服务器（工具按子智能体需求分配，不全局加载）  
    mcp_servers={  
        "mongodb": McpStdioServerConfig(  
            command="npx",  # 通过 npx 快速启动 MCP 服务器  
            args=["-y", "mongodb-mcp-server@latest", "--readOnly=false"],  # 允许写入操作  
            env={"MDB_MCP_CONNECTION_STRING": connection_string}  # 从环境变量传入连接串  
        )  
    }  
)
```

#### 方式 2：命令参数配置 MCP（适合调试）

若需直接在代码中指定连接串（不推荐生产环境），可修改 MCP 配置：

```
mcp_servers={  
    "mongodb": McpStdioServerConfig(  
        command="npx",  
        args=[  
            "-y",  
            "mongodb-mcp-server@latest",  
            "--connectionString",  # 直接通过命令参数传入连接串  
            connection_string  
        ]  
    )  
}
```

### Step 4: 运行示例（直接执行命令）

配置完成后，可通过以下命令测试子智能体的协同效果：

```
# 1. Reader Agent 工作：分析数据库结构  
uv run --env-file .env main.py --prompt "Analyze the schema of all collections in sample_mflix (only show field types)"  
# 2. Query Agent 工作：查询最新电影（Reader 已分析完 Schema，Query 无需重复加载）  
uv run --env-file .env main.py --prompt "What are the top 10 most recent movies in sample_mflix?"  
# 3. Writer Agent 工作：新增统计结果（不依赖 Query 工具，上下文无冗余）  
uv run --env-file .env main.py --prompt "Insert a new document into 'movie_stats' collection: { 'top_genre': 'Drama', 'count': 120 }"
```

## 💡 总结与启示：子智能体是下一代 Agent 的 “基础设施”

这个项目给 AI 开发者带来的核心启示，首推 **“用子智能体解决上下文衰减”**—— 这比优化 Prompt、升级模型更根本：

1.**子智能体 > 超长上下文**：与其依赖厂商宣称的 “百万级上下文窗口”，不如用子智能体 “物理隔离” 上下文。Chroma 研究已证明，10 万令牌后的模型性能衰减无法通过 Prompt 优化规避，而子智能体从架构层面切断了 “上下文臃肿” 的源头；2.**工具隔离是关键**：子智能体的 “专属工具集” 设计，让每个智能体的上下文仅包含 “任务 + 必要工具”，避免了 “26 种工具定义塞满上下文” 的问题，这是提升决策准确性的直接原因；3.**架构可复用性高**：这套子智能体分工逻辑（只读 / 写入 / 查询）不仅适用于 MongoDB，还可迁移到 SQL 数据库、API 调用等场景 —— 只要按 “任务类型” 拆分智能体，就能规避上下文衰减。

其他启示：

•**MCP 是子智能体的 “工具管家”**：MCP 的标准化工具接口，让子智能体 “按需拿取工具” 成为可能，无需手写工具函数；•**模型平权靠架构**：DeepSeek v3.2 这类开源模型，配合子智能体架构，完全能胜任企业级任务，无需依赖昂贵的闭源模型。

**参考资源：**

•DeepSeek v3.2 官方文档[1]•Claude Agents SDK Python 文档[2]•MongoDB MCP 服务器文档[3]•Chroma 上下文衰减研究[4]（子智能体的理论基础）•《子智能体的崛起》[5]（深入理解子智能体架构）

开源地址： 

deepseekv3.2-mongodb： https://github.com/NielsRogge/tutorials/blob/main/deepseekv3.2-mongodb/README.md

---

**写在最后**

AI Agent 的开发正在从 “拼模型参数” 走向 “拼架构设计”，而子智能体就是这场变革的 “基础设施”。DeepSeek 提供了低成本算力，Claude SDK 提供了子智能体的运行框架，MCP 提供了标准化工具接口 —— 三者结合，不仅解决了 “上下文衰减” 的核心痛点，更定义了下一代 AI 应用的 “模块化” 范式：未来的 Agent 不再是 “单体巨人”，而是 “子智能体协同网络”。
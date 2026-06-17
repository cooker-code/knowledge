---
title: 深入解析 LangChain v1.0 核心组件：Messages (消息列表) 全指南
author: AI大数据回忆录
date: 
url: https://mp.weixin.qq.com/s/D-pc9D6qxghCTKYwk4eCJw
---

在 LLM 应用开发从“玩具 demo”走向“生产环境”的过程中，我们对 Prompt 的管理方式发生了根本性的变化。在 LangChain v1.0 时代，简单的字符串拼接（String Prompting）已无法满足复杂 Agent 和长上下文的需求。

本文将深入解构 LangChain v1.0 的核心数据结构——**Messages（消息列表）**，带你理解如何通过结构化的消息管理，构建模块化、可维护且稳定的 AI 应用。

## 1. 引言：从 Prompt 到 Messages 的进化

### 1.1 为什么 String Prompt 不够用了？

早期的 LLM 开发往往只是简单地拼接字符串：`"你是一个翻译官，请翻译这句话：" + user_input`。但在构建复杂的对话系统或 Agent 时，这种方式显得捉襟见肘。我们需要明确区分“谁在说话”、“哪句是历史记忆”、“哪句是函数调用结果”。

LangChain v1.0 的核心思想是让 Prompt **模块化、结构化**。

### 1.2 Messages 的本质

本质上，Messages 是 LLM 输入层的“汇编语言”。我们可以用一个公式来概括它的作用：

> **Messages = 构造上下文 + 定义模型行为 + 填充历史 + 控制推理流程**

模型不仅仅是在“续写文本”，而是在解析一个包含角色（Role）、内容（Content）和元数据（Metadata）的结构化列表。

## 2. 核心组件：五大角色详解 (The 5 Roles)

LangChain 通过 `langchain_core.messages` 模块标准化了不同角色的消息类。

### 2.1 SystemMessage (系统指令)

* **作用**：设定模型的身份、风格、输出格式及核心规则。
* **特点**：拥有“最高优先级”，通常位于列表的最顶端。

### 2.2 HumanMessage (用户输入)

* **作用**：承载用户的当前提问（Query）。
* **进阶**：支持多模态输入（如传入图片 URL）。

### 2.3 AIMessage (模型回复)

* **作用**：模型的输出内容。
* **关键属性**：

+ `content`: 文本回复。
+ `tool_calls`: **(v1.0 重点)** 如果模型决定调用工具，这里会包含工具名称和参数。

### 2.4 ToolMessage (工具反馈)

* **作用**：Agent 模式下的核心，用于将函数/工具执行的结果回传给模型。
* **约束**：必须包含 `tool_call_id`，以对应上一条 `AIMessage` 中的调用请求。

### 2.5 Developer 角色 (OpenAI 新特性)

* **作用**：OpenAI 在 o1/o3 系列模型中引入的角色，用于区分“工程约束”与“系统设定”。
* **LangChain 支持**：虽然目前主要用于特定模型，但 LangChain 已在底层做了映射支持。

### 代码示例：基础消息构造

```
from langchain_core.messages import (  
    SystemMessage,   
    HumanMessage,   
    AIMessage,   
    ToolMessage  
)  
  
# 1. 系统设定  
sys_msg = SystemMessage(content="你是一个专业的 Python 代码助手。")  
  
# 2. 用户提问  
human_msg = HumanMessage(content="帮我写一个计算斐波那契数列的函数。")  
  
# 3. 模型回复（模拟历史）  
ai_msg = AIMessage(content="好的，这是代码...", tool_calls=[])  
  
# 4. 工具调用结果（模拟 Agent 场景）  
# 注意：tool_call_id 必须与 AI Message 中的 id 对应  
tool_msg = ToolMessage(  
    tool_call_id="call_AbCd12345",   
    content="Execution success: Result = 55"  
)  
  
messages = [sys_msg, human_msg, ai_msg, tool_msg]  
print(messages)
```

## 3. 实战构建：ChatPromptTemplate 与消息编排

在实际工程中，我们很少手动编辑或拼接列表，而是使用模板引擎。

### 3.1 使用 ChatPromptTemplate

这是 LangChain 中最标准的构建方式。它允许我们预埋变量和历史记录占位符。

### 3.2 MessagesPlaceholder 的妙用

在处理对话历史（Chat History）时，我们无法预知历史有多少条。`MessagesPlaceholder` 允许我们在 Prompt 中“挖一个坑”，运行时动态填充。

### 代码示例：构建带有记忆的 Prompt

```
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder  
  
# 定义模板结构  
prompt_template = ChatPromptTemplate.from_messages([  
    # 1. 系统指令：始终置顶  
    ("system", "你是一个乐于助人的 AI 助手，你的名字叫 {name}。"),  
      
    # 2. 历史记录占位符：运行时动态插入 list[BaseMessage]  
    MessagesPlaceholder(variable_name="chat_history"),  
      
    # 3. 用户当前输入  
    ("human", "{input}")  
])  
  
# 模拟一些历史对话  
history = [  
    HumanMessage(content="你好，我叫小明。"),  
    AIMessage(content="你好小明！有什么我可以帮你的吗？")  
]  
  
# 格式化生成最终的消息列表  
final_messages = prompt_template.invoke({  
    "name": "Jarvis",  
    "chat_history": history,  
    "input": "我刚才告诉你我叫什么名字？"  
})  
  
# 查看生成的列表长度  
print(f"Total messages: {len(final_messages.to_messages())}")
```

## 4. 进阶技巧：Context Window 管理 (Trimming)

这是 LangChain v1.0 的技术重点。随着对话轮数增加，消息列表很容易通过 Token 上限。我们需要一种机制来“修剪”消息，同时保留核心指令。

### 4.1 trim\_messages：智能修剪

`trim_messages` 提供了开箱即用的策略，可以按 Token 数量修剪，同时通过 `include_system=True` 确保系统指令不被误删。

### 代码示例：防止 Token 超限

```
from langchain_core.messages import trim_messages, SystemMessage, HumanMessage, AIMessage  
  
# 构造一个超长的对话历史  
messages = [SystemMessage(content="核心规则：不要撒谎。")] + \  
           [HumanMessage(content=f"这是第 {i} 句废话") for i in range(100)] + \  
           [AIMessage(content=f"这是第 {i} 句回复") for i in range(100)]  
  
# 使用 trim_messages 进行修剪  
trimmed_messages = trim_messages(  
    messages,  
    max_tokens=50,           # 限制最大 token 数  
    strategy="last",         # 保留最新的消息  
    token_counter=len,       # 这里为了演示简单用 len，实际应使用模型的 tokenizer  
    include_system=True,     # 【关键】强制保留 SystemMessage，防止人设崩塌  
    allow_partial=False,     # 不允许截断单条消息  
    start_on="human"         # 确保对话从 Human 开始（避免 AI 自言自语）  
)  
  
print(f"原始消息数: {len(messages)}")  
print(f"修剪后消息数: {len(trimmed_messages)}")  
# 输出结果会发现，SystemMessage 依然在，且紧接着是最新的几轮对话
```

## 5. 最佳实践：执行顺序与优先级机制

理解 LLM 如何“阅读”这些消息至关重要。模型通过顺序解析来建立上下文依赖。

### 5.1 消息堆叠的“潜规则”

模型永远会参考全部 messages 才得出最终输出，但它们的权重和作用域不同：

| **顺序** | **Role** | **核心逻辑** |
| --- | --- | --- |
| **1** | **System** | **最高优先级** 。奠定基调，任何后续消息原则上不应违背此处的约束。 |
| **2** | **Developer** | 工程约束层。常用于通过 API 强制注入的安全策略。 |
| **3** | **Chat History** | (Human/AI 交替) 提供少样本学习（Few-Shot）或上下文记忆。 |
| **4** | **Human (Current)** | 用户当前的意图。模型主要针对此条目进行响应。 |
| **5** | **Tool/AI** | (仅 Agent 场景) 用于补充最新的外部数据。 |

### 5.2 常见坑点与调试

1. **System Message 放错位置**：如果在 User Message 之后插入 System Message，部分模型会感到困惑，导致指令遵循能力下降。
2. **ToolMessage 孤立**：`ToolMessage` 必须紧跟在发起调用的 `AIMessage` 之后，否则模型会报错 "Unexpected tool output"。
3. **上下文污染**：未及时清理过期的 Tool output，导致 Prompt 包含大量无用的 JSON 数据，浪费 Token。

## 6. 总结

LangChain v1.0 的 `messages` 体系不再是简单的文本容器，它是连接人类意图与机器推理的桥梁。

* **SystemMessage** 给了模型“灵魂”；
* **MessagesPlaceholder** 给了模型“记忆”；
* **ToolMessage** 给了模型“手眼”；
* **Trim/Filter** 给了开发者“控制权”。

掌握消息列表的构建与管理，是开发高可用 LLM 应用的第一步，也是最关键的一步。

---

往期LangChain文章回顾：

[01-LangChain v1.0 升级狂潮：初学者别错过，这些组件变化将颠覆你的 LLM 开发！](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484681&idx=1&sn=9dc34716a4105b929e316742bdbeda4f&scene=21#wechat_redirect)

[02-官网quickstart预览，最新版LangChain v1.0 悄悄做了这些事，再不学就晚了！](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484694&idx=1&sn=2b0a8a4087286dfd95ea329ff9bd5b85&scene=21#wechat_redirect)

[03-快速入门LangChain v1.0：6步轻松实现智能语义搜索，打造你的第一个AI应用！](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484719&idx=1&sn=6b805b67eacc4847b8dc9423b766dad7&scene=21#wechat_redirect)

[04-LangChain 1.0搞定所有AI模型调用，让你轻松上手（附代码示例）](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484739&idx=1&sn=9579df0e0ebac6a4e1f2a6fd3674c0b3&scene=21#wechat_redirect)

[05-LangChain 1.0 快速入门：消息格式、流式响应与结构化输出，5分钟轻松搞定流式聊天机器人！](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484750&idx=1&sn=824c1b14fb018a59b8ae4cbec07e1fa6&scene=21#wechat_redirect)

[06-你的LangChain Agent过时了！1.0版本正在重新定义AI智能体的未来](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484777&idx=1&sn=942e239c3cf9a8e9d682dfee2a142e4f&scene=21#wechat_redirect)

[07-LangChain 1.0 Middleware快速入门，构建更可控更可靠的AI智能体，掌控AI智能体的每一步决策与行动](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484794&idx=1&sn=cf681bc54f1116249282452387b10194&scene=21#wechat_redirect)

[08-LangChain v1.0 模块化架构详解：从新手到上手，一文读懂各大依赖包的定位与作用](https://mp.weixin.qq.com/s?__biz=Mzg5NTgxNjI4NQ==&mid=2247484824&idx=1&sn=4802cbc57d6fd8e833550aba0efbaf38&scene=21#wechat_redirect)
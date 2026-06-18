> 已吸收至：[[02_Agent与AI工程/0212_COT&TOT&REACT/021200_核心知识点/推理范式来源锚点与排重准则|推理范式来源锚点与排重准则]]
---
title: 当 think 遇上 tool：深入解析 Agent 的规划之道
author: 拾光学迹
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484544&idx=1&sn=59faf169f9909dba734b82c04cfc32cf&chksm=c2c835fd96ad38ece20211fc6df0caccb8c24bdc5ae9bffe57a5c979cc127c16758d65bf80b7&mpshare=1&scene=24&srcid=0919g2oXkT2u4Cc99twdmrwX&sharer_shareinfo=51df33a277bc8084f477cccd420523d7&sharer_shareinfo_first=51df33a277bc8084f477cccd420523d7#rd
---

## 当 think 遇上 tool：深入解析 Agent 的规划之道

**字数：约 3500 字 ｜ 预计阅读时间：10 分钟**

---

大家好，我是 **Leon**。

AIcoding越来越卷，工具越接越多，但越用越让我焦虑 —— 特别是某 c....r
它总像一个急着表现的新同事，不问全局、就开始动手执行。我发现它：

* • **刚启动就火力全开**：不等我把意图讲完就调工具；
* • **重复调用接口**：出了错就死磕一个 API，丝毫没有“换个思路”的意识；
* • **逻辑断片严重**：上一秒还在分析，下一秒就不记得自己分析的是什么。

这些坑我踩过太多次，以至于后来我都养成了“先给 AI 做一个冷启动提示词”的习惯。
有点像是：

> **“你别急，我先帮你想好怎么干，你再去动手。”**

我们人类其实很擅长“在脑子里先走一遍流程”这件事。
这种能力，说白了就是 **规划（Planning）**。AI 不具备，是因为它根本没有“做计划”这个意识。

作为一个在 AI 应用开发一线摸爬滚打的算法工程师，我的目标是让 Agent 拥有它。今天就和大家分享一下最近的学习心得 —— 如何让你的 Agent 学会「三思而后行」：基于 Anthropic、OpenAI 的最新研究，以及我在实际开发中的踩坑经验。

---

## 一、AI，不擅长“打草稿”这件小事

### 数据不会骗人，两个官方实验直接拉开了差距

#### 实验1：OpenAI官方的"强行规划"实验

在 SOTA 论文里，OpenAI 的研究员干了一件很极致的事——他们不是引导模型去规划，而是**强行命令**它：“先规划，再行动，别自作主张”。

贴一个原话：

```
"You MUST plan extensively before each function call, 
and reflect extensively on the outcomes of the previous function calls. 
DO NOT do this entire process by making function calls only, 
as this can impair your ability to solve the problem and think insightfully."
```

多加这一句，SWE-bench 的通过率就提升了 4%。
虽然你可能觉得 4% 不多，但要知道这是在已经很强的模型上拉的提升。

更重要的是——OpenAI 这个实验不是靠 prompt 魔法，而是配合后训练（RLHF）来强化这个指令的。
我们用的开源模型，别说RLHF了，连理解都不一定到位……

#### 实验2：Anthropic的"思考工具"

Anthropic更进一步，他们不只是"要求"模型规划，而是给了模型一个专门的工具——`think` tool：

```
{
  "name":"think",
"description":"Use this tool when you need to think through a problem step by step",
"input_schema":{
    "type":"object",
    "properties":{
      "thought":{
        "type":"string",
        "description":"Your structured thinking process"
      }
    }
}
}
```

看起来简单到令人发指，对吧？但这就是工程学的美妙之处——**最优雅的解决方案往往是最简单的**。

直接在 τ-bench 上直接提升了 54% 的完成率。

一个“思考工具”能顶得上半个模型优化？！

然后我意识到一件事：这不是在优化“推理能力”，这是在重塑模型的行为模式——让它从“习惯执行”变成“先组织认知，再输出行为”。

### Think Tool vs Extended Thinking：不只是"暂停"

这里有个重要的概念需要澄清。很多人会问："这和Extended Thinking有什么区别？"

* • Extended Thinking发生在AI开始响应之前，像是在心里打草稿
* • Think Tool发生在AI开始响应之后，像是在工作过程中主动停下来整理思路

| 模式 | 思考发生时间 | 可控性 | 记录性 |
| --- | --- | --- | --- |
| Extended Thinking | 回复之前 | 黑盒 | 无法追踪 |
| Think Tool | 回复之中 | 显式触发 | 可记录、可分析 |

Think Tool更适合处理那些需要分析外部信息的复杂场景，比如：

* • 分析工具调用的返回结果
* • 在长链工具调用中保持逻辑一致性
* • 在政策繁复的环境中确保合规性

54%的性能提升背后，是AI从"盲目执行"到"深思熟虑"的质的飞跃。
你可能会觉得：一个工具调用而已，至于这么讲究吗？

但我想说的是，在我不断试错的过程中发现，只有让“思考”成为**流程的一部分**，Agent 才不再是一个随机响应的黑盒，而是一个可以协同的思考体。

这就是我为什么把“规划能力”当成是 Agent 的第一性能力——只有它拥有了“画施工图”的本领，才有可能成为我们真正意义上的智能助手。

---

## 二、三大主流规划方案：各有千秋

那么，如何让AI学会规划呢？目前业界主流的方案有三种，各有千秋

| **方案** | **代表** | **实现方式** | **我的评价** |
| --- | --- | --- | --- |
| **Prompt显式规划** | OpenAI | 在Prompt中要求输出规划步骤 | 简单直接，但效果限于OAI的后训练模型 |
| **结构化思考工具** | Anthropic | 定义think工具让模型主动调用 | 可靠性高无限制，首选 |
| **独立规划模块** | OpenManus | 专门的Planning Flow生成计划 | 适合超复杂任务，但重炮打蚊子 |

###

### 为什么“思考工具”是当前优选？

> Leon's Take: 作为一个追求优雅和通用方案的J人，答案其实很清晰。我们需要的不是一个更聪明的“黑盒”，而是一个更可靠、更可控的“流程”。

1. 1. **开源模型友好**：对于"请规划一下"这种模糊指令，开源模型的理解能力参差不齐。但「调用xx工具」这是它们的强项。
2. 2. **强制结构化**：工具可以强制模型输出特定字段，比如：

   ```
   {
     "thought": "当前分析",
     "plan": "分解步骤", 
     "action": "下一步行动",
     "step_number": "步骤编号"
   }
   ```
3. 3. **可追溯调试**：每次思考都有记录，出问题时能快速定位。

蚂蚁集团的论文也印证了我的想法。他们在构建自己的Agent平台时，最终选择了复用Anthropic的 **思考工具** 思路

### 蚂蚁集团的生产级实践

蚂蚁的工程师们在实际部署中，设计了一个更加工程化的思考工具：

```
{
  "name":"思考和规划",
"description":"分阶段梳理思考、计划、行动",
"input_schema":{
    "properties":{
      "thought":{"type":"string"},        // 当前分析
      "plan":{"type":"string"},          // 分解步骤  
      "action":{"type":"string"},        // 下一步建议
      "thoughtNumber":{"type":"string"}// 步骤编号
    },
    "required":["thought","plan","action","thoughtNumber"]
}
}
```

这个设计的巧妙之处在于：

* • **强制结构化**：避免模糊的思考内容
* • **步骤追踪**：便于调试和优化
* • **行动导向**：直接输出下一步该做什么
* • **流式友好**：支持实时输出，提升用户体验

特别是thoughtNumber，对于我们这些需要调试和复盘的J人来说，简直是福音。

---

## 三、实战指南：四步构建规划Agent

接下来，我们以构建一个具备规划能力的Agent为例，提供一套从模型选型到Prompt设计的完整实践指南。

### Step 1: 模型选型——工欲善其事，必先利其器

* • **推荐**：优先选择对Function Call（工具调用）优化过的模型，例如`DeepSeek-V3 Function Call`版。这类模型能更好地理解和执行工具调用指令。
* • **避坑**：避免选择为通用对话设计的模型，如`DeepSeek-R1`，它们在多工具连续调用场景下可能表现不佳，且延迟较高。
* • **参数设置**：建议将生成多样性参数（如`temperature`）设置为一个较低的值，比如**0.3**。这有助于在保证准确性的前提下，获得一些有限的创造性，避免模型“胡思乱想”。

### Step 2: 核心关键工具配置

#### 思考与规划工具 (`think_and_plan`)

这是我们为Agent植入的“大脑”。它的定义至关重要。

```
{
  "name":"think_and_plan",
"description":"在执行任何业务操作前必须调用的思考工具。像人类一样先思考再行动。",
"input_schema":{
    "type":"object",
    "properties":{
      "user_intent":{
        "type":"string",
        "description":"你对用户核心需求的理解"
      },
      "current_situation":{
        "type":"string",
        "description":"当前状态分析，包括已有信息和缺失信息"
      },
      "plan":{
        "type":"array",
        "items":{"type":"string"},
        "description":"详细的执行步骤列表，每步都要具体可执行"
      },
      "next_action":{
        "type":"string",
        "description":"基于规划确定的下一个具体行动"
      }
    },
    "required":["user_intent","current_situation","plan","next_action"]
}
}
```

**使用逻辑**：可以在System Prompt**强制**模型在每次调用业务工具前，**必须**先调用此`think_and_plan`工具。模型需要根据这个工具的输出来决策下一步的具体行动。

> 📌 核心经验：投入在优化工具和工具Prompt描述上的精力，与最终提升用户体验的精力，是1:1的。（这句话得是多么痛的领悟...

#### 业务工具描述——像写API文档一样严谨

一个常见的失败模式是Agent遇到问题时，陷入无限循环调用。我们需要一个“熔断”机制，这和写普通代码的异常处理逻辑是相通的，另外给LLM的路径尽量用绝对路径等等

**✅ 好的工具描述**：

```
{
  "name":"file_analyzer",
"description":"分析文件内容和结构。支持.txt, .csv, .json格式。文件大小限制10MB。",
"when_to_use":"当用户需要了解文件内容、格式或统计信息时使用",
"limitations":"不支持二进制文件，不能修改文件内容",
"input_schema":{
    "properties":{
      "file_path":{
        "type":"string",
        "description":"文件的绝对路径，例如：/home/user/data.csv"
      }
    }
}
}
```

**❌ 糟糕的工具描述**：

```
{
  "name":"file_tool",
"description":"处理分析文件内容",
"input_schema":{
    "properties":{
      "path":"./data.csv"
      "type":"string"
    }
}
}
```

区别在哪里？好的描述告诉AI：

1. 1. **什么时候用我**（使用场景）
2. 2. **我能做什么**（功能边界）
3. 3. **我做不了什么**（限制条件）
4. 4. **怎么用我**（参数规范）

### Step 3: Prompt设计——Agent的"行为准则"

#### 系统级指令

```
# Agent核心工作流程

你是一个具备规划能力的AI助手。你的工作流程是：

1. **强制规划优先**：在调用任何业务工具前，必须先调用`think_and_plan`工具进行思考
2. **循环执行**：规划 → 执行 → 分析结果 → 重新规划，直到任务完成
3. **错误处理**：工具调用失败时，必须重新规划而不是盲目重试
4. **并行限制**：除非明确确认无依赖关系，否则禁止并行调用工具

记住：你不是一个急性子的实习生，而是一个深思熟虑的专业助手。
```

#### 业务层补充

针对具体场景，我会添加更详细的指导，举个栗子：

```
# 数据分析场景特殊规则

- 处理数据前，必须先用数据概览工具检查格式和质量
- 发现数据问题时，要先清理再分析，不要带病作业  
- 生成图表前，要确认数据的统计特征，避免误导性可视化
```

### Step 4: 防循环与错误处理——给Agent装上"安全带"

在实际项目中，我遇到过Agent陷入死循环的情况。比如：

1. 1. 调用工具A失败
2. 2. 重新规划，还是调用工具A
3. 3. 再次失败，再次规划...
4. 4. 无限循环

**解决方案：双层防护**

**硬性限制（写进代码）**：

```
# 伪代码
max_retries = 3
max_total_calls = 20
timeout = 300  # 5分钟

if consecutive_failures >= max_retries:
    return "连续失败次数过多，请检查工具配置或联系技术支持"
    
if total_calls >= max_total_calls:
    return "调用次数超限，任务可能过于复杂，建议分解后重试"
```

**软性引导（写进 Prompt）**：

```
# 在think_and_plan工具中添加
"failure_analysis": {
  "type": "string", 
  "description": "如果上一步失败，分析失败原因并调整策略。连续失败3次后必须寻求人工帮助，要求补充必要的上下文信息。"
}
```

---

## 四、争议与思考：工具 vs 纯推理

一个常见的问题是：我们能否用一个更强的推理模型，来代替`think`工具呢？

Anthropic的结论是：**不能，至少目前不行。**

在Claude 3.7上的实验表明，**“思考工具 + 专用Prompt”** 的效果，显著优于单纯依赖模型自身推理的模式。

**可能的原因**：

1. 1. **领域定制**：工具的Prompt可以针对特定领域（如航空、金融）的思考模式进行深度定制和优化，这是通用推理模式无法比拟的。
2. 2. **上下文保留**：工具调用会将每一次的思考过程（规划、反思）完整地保留在上下文中，形成一个清晰的逻辑链。而纯推理模型为了节省token，可能会在内部“遗忘”或删减中间的思考步骤。
3. 3. **可控性**：工具调用是可控的、可追踪的，而模型内部推理对我们来说是黑盒。

这让我想起了软件工程中的一个原则：**显式优于隐式**。把思考过程显式化，总是比依赖黑盒推理更可靠。

---

## 结语：从"码农"到"AI架构师"

写到这里，我想分享一个感悟。

把“规划”显式化这件事很反直觉。毕竟人类的思维是隐性的、灵活的。但 Agent 是个偏执行型的东西，如果不给它立规矩，它就永远是个胡乱点技能的萌新。

毕竟“聪明”这件事，是在限制中长出来的。

我们未来的能力，也许不在于 Prompt 写得多 fancy，
而在于我们能不能把模糊的问题，变成清晰的链路；
把复杂的世界，变成模型可理解的认知场。

> 📬 如果你也在做 Agent 系统设计，欢迎一起讨论。
>
> 我们都在试错路上，别让自己一个人踩坑。

往期精彩

[RAG检索策略深度解析：从BM25到Embedding、Reranker，如何为LLM选对“导航系统”？](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484526&idx=1&sn=06e33a4f05d8f3be34b6e798dd0a8db4&scene=21#wechat_redirect)

[10分钟搞定！AI表情包自由，白嫖GPT-4o，让你在群里横着走！](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484488&idx=1&sn=d123a7326a6ebd55f6cbde19467bb65f&scene=21#wechat_redirect)

[还在置顶文件传输助手吗？元宝也可以置顶聊天了！快来试试AI助手吧！](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484385&idx=1&sn=572c42052f2f5e0e85b974bf55131c7e&scene=21#wechat_redirect)

[SEO老了？GEO来了！玩转传统搜索+AI搜索，吸引眼球大作战！](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484370&idx=1&sn=073d8b19b94ff17e332897be2a2f6b07&scene=21#wechat_redirect)

[智能Agent如何改造传统工作流：从搜索到全能助手](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484307&idx=1&sn=34a9f5171fe747ee76abb3186ea14ca8&scene=21#wechat_redirect)

[AI团队比单打独斗强！CrewAI多智能体协作系统开发踩坑全解析](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484324&idx=1&sn=d003410524d42fc93ccd0540bf3f3e6a&scene=21#wechat_redirect)

[深入浅出：Agent如何调用工具——从OpenAI Function Call到CrewAI框架](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484175&idx=1&sn=0bc274ef0fd8f95de2913bd5cf65188a&scene=21#wechat_redirect)

[一文读懂-多智能体编译：从例行到交接的奥秘](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247484151&idx=1&sn=6af9c39806afafe42eefe321e169678f&scene=21#wechat_redirect)

[《使用coze搭建一个会搜索、写ppt、思维导图的Agent》](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247483999&idx=1&sn=382fd90e7db193632691bfbb233823d3&scene=21#wechat_redirect)

[AI驱动开发:用Cursor零基础打造web项目的终极指南](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247483942&idx=1&sn=8cb8916ca1485e783ed0c7bb59179803&scene=21#wechat_redirect)

[在 RAG 中数据处理的关键：数据切片的挑战与解决方案](https://mp.weixin.qq.com/s?__biz=Mzk0ODc0NzU5OQ==&mid=2247483753&idx=1&sn=e7fb6f779b671c2acd32daae98f3b369&scene=21#wechat_redirect)

阅读原文链接了解更多信息

**觉得有用？点赞、在看、转发三连，下次教你更刺激的AI玩法！**

---

**参考资料**：

[1]https://github.com/openai/openai-cookbook/blob/main/examples/gpt4-1\_prompting\_guide.ipynb

[2]https://www.anthropic.com/engineering/claude-think-tool

[3]https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking

[4]https://www.anthropic.com/engineering/building-effective-agents

[5]https://docs.anthropic.com/en/docs/build-with-claude/context-windows#the-context-window-with-extended-thinking-and-tool-use
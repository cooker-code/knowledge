> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes 人格装配——系统提示词是怎么设计的？
author: 智能思辨录
date:
url: https://mp.weixin.qq.com/s?__biz=MzY5NzIwOTg4MQ==&mid=2247483796&idx=1&sn=723e25b241688908ae3deaf0876d6e77&chksm=f5b3b7322e74ac0f82659f1cbe6ec3f6552c113df5607cdc7a7284fa2c9155124e3b4478dfff&mpshare=1&scene=24&srcid=041982jWmAvSOkHfpoyVViEK&sharer_shareinfo=a047a0b47b9443581d55d691ad8774b3&sharer_shareinfo_first=a047a0b47b9443581d55d691ad8774b3#rd
---

继[上一篇文章](https://mp.weixin.qq.com/s?__biz=MzY5NzIwOTg4MQ==&mid=2247483790&idx=1&sn=f613d0cf8b37898f6534700223564f6c&scene=21#wechat_redirect)，我们大概了解了 Hermes 的架构、模块划分。今天这篇文章是我对 \_build\_system\_prompt（构建系统提示词） 方法的研究和总结，主要目的是想拆解一下 Hermes 系统提示词构建逻辑。

**先看全景：Hermes 系统提示词装配图**

话不多说，先把这条流水线的全貌放出来。后面我们会根据环节展开讲。

**① soul.md****：20k token 的容量**

流水线的第一步，是导入 `soul.md`——这是 Agent 的**灵魂文件**，定义了它的核心人格、价值观、说话风格。

关键约束：**最多 20,000 token**。超过这个上限会触发截断，规则是：

* 头部保留 70% 的 token
* 尾部保留 20% 的 token
* 中间插入截断标识

最终形态是 `头部 + 截断标识 + 尾部`。这个"头尾保留、掐掉中段"的设计很有意思——人格定义通常在文档前段（谁是我、我信什么），风格约束和特殊处理规则通常在末尾（边界情况、禁止事项），中段往往是展开解释和例子，损失相对可控。

**设计意图**：soul.md 是人格的"本体"，所有后续模块都是在这个本体上叠加能力与约束。没有 soul.md 的 Agent 是一个裸模型，有 soul.md 的 Agent 才是"Hermes"。

**②****记忆不是一个东西，需要划分**

这是整条流水线里让我最有启发的一个设计。

Hermes 在 memory 工具的系统提示词里，明明白白告诉 Agent：有些东西该存记忆、有些东西不该存。这个边界一划，把「记忆」分成了三种完全不同的存储：

| **存储** | **该存的内容** | **生命周期** |
| --- | --- | --- |
| memory | 用户偏好、环境细节、稳定约定 | 长期事实 |
| session\_search | 过往对话、历史上下文 | 历史检索 |
| skill | 可复用的工作流和方法论 | 可沉淀方法 |

这三种东西，在朴素的Agent里经常被糊在一起——什么都往 memory 里塞，结果 memory 越来越膨胀，重要的事实被淹没在任务日志里。

Hermes 的做法是用提示词强行规定Agent：任务进度不要塞 memory，去 session\_search 里找；可复用的方法论不要塞 memory，存成 skill。

**③****工具使用强制：驯服不听话的模型**

当用户在配置文件中开启了工具强制使用后生效：

```
agent:  tool_use_enforcement: ["gpt", "codex", "gemini", "grok", "my-custom-model"]
```

不同模型调用工具的稳定性差异巨大。Claude 天然倾向于「边说边做」，但 GPT、Codex、Gemini、Gemma 这些模型经常陷入一种怪圈——它会告诉你：「好的，我来运行测试。」然后不调用任何工具就结束了这一轮。你催它一次，它又说：「马上创建项目」然后还是不调用工具就结束了。

Hermes 针对这类模型注入一段强制提示词，核心三条：

**•**说了就立刻做，不要停在「我将……」

**•**工作到真正完成，不要停在「下次继续……」

**•**每次回复必须是「推进的工具调用」或「最终结果」二选一，没有中间态

// 其实这里我看是*吸收了 OpenClaw 和 OpenCode 的经验——针对Gemini、GPT 这类模型，还会再追加一段模型专属的强化说明。本质上是用 prompt 在做「对模型惰性的工程化规训」*

**④用户自定义的prompt**

用户自己在配置中写的 system prompt 会在这一步追加。这是整条流水线中**唯一完全由用户控制**的部分——前后都是框架在装配，中间这一段是用户的"自留地"。

位置选择很有意思：放在能力声明之后、记忆/项目上下文之前。我猜测其目的是用户的定制可以看到"我有什么能力"，但不会污染"我是谁（soul.md）"的人格底座。

**⑤记忆与用户画像注入**

当用户在配置文件中开启了如下配置后生效：

```
memory:  memory_enabled: true       # 注入 memory 到提示词  user_profile_enabled: true # 注入 user 到提示词
```

对应行为：

* `memory_enabled: true → 追加 ~/.hermes/memories/MEMORY.md`
* `user_profile_enabled: true → 追加 ~/.hermes/memories/USER.md`

**其中 MEMORY 与 USER 的区别**：

* `MEMORY.md：Agent 在过往交互中积累的关于世界、工具、约定的事实`
* `USER.md：关于用户本人的画像——身份、偏好、背景、工作方式`

两者独立开关，允许用户选择只注入其中之一（比如跨账户共享 MEMORY 但隔离 USER）。

### 外接 memory 的追加说明

###

如果 memory 插件是**非内置**的（即用户接入了外部记忆系统，如 Mem0），还需要追加一段由该插件的 `system_prompt_block` 方法提供的说明。以 Mem0 为例提供的是：

```
# Mem0 Memory Active. User: {self._user_id}.Use mem0_search to find memories, mem0_conclude to store facts, mem0_profile for afull overview.
```

**这段设计的价值**：Hermes 把记忆系统做成了**插件化架构**——任何第三方记忆服务（Mem0、Zep、Letta……）只要实现提示词方法，就能自动被装配进系统提示词，告诉 Agent 应该用什么工具去查询和写入。但是注意外部 memory 插件只能配置一个。

**⑥技能系统（最复杂的一步）**

当用户可用工具包含 `skills_list`、`skill_view`、`skill_manage` 中**任一个**时，进入技能装配流程。这也是整条流水线里最复杂的一步。

其步骤如下（参照build\_skills\_system\_prompt代码）：

1. 获取所有可用工具与工具集
2. **构建 LRU 缓存**，命中直接返回
3. 从配置中加载**全局禁用技能列表**
4. 从磁盘加载所有技能——两个来源：

* skill 的 **snapshot 文件**（序列化元数据）
* 实际安装的 skill 目录

5. **优先走 snapshot 文件；无 snapshot 时遍历所有 skill 目录，解析元信息。同时过滤掉：**

* 系统不兼容的
* 依赖条件不满足的
* 在禁用列表中的

6. 按 skill 元信息中的 `category`**分类组织**
7. 首次运行时，**写入 snapshot** 以加速下次启动
8. 遍历**外部技能目录**（只读，不做 snapshot），同样走过滤 → 合并流程。**本地技能优先**，同名时外部技能被覆盖
9. 若过滤后有可用技能，构建技能系统提示词（见下）

10. 结果写入 LRU 缓存

其构建的提示词如下：

```
## Skills (mandatory)Before replying, scan the skills below. If a skill matches or is even partiallyrelevant to your task, you MUST load it with skill_view(name) and follow itsinstructions.
Err on the side of loading — it is always better to have context you don't needthan to miss critical steps, pitfalls, or established workflows.
Skills contain specialized knowledge — API endpoints, tool-specific commands, andproven workflows that outperform general-purpose approaches. 
Load the skill even if you think you could handle the task with basic tools like web_search or terminal.
Skills also encode the user's preferred approach, conventions, and quality standardsfor tasks like code review, planning, and testing — load them even for tasks youalready know how to do, because the skill defines how it should be done here.
If a skill has issues, fix it with skill_manage(action='patch').
After difficult/iterative tasks, offer to save as a skill.
If a skill you loaded was missing steps, had wrong commands, or needed pitfalls youdiscovered, update it before finishing.
<available_skills>你的技能</available_skills>
Only proceed without loading a skill if genuinely none are relevant to the task.
```

**这段提示词的关键词是 "mandatory"**。技能不是"可选参考资料"，而是**在当前环境下处理该类任务的官方方法论**。Hermes 在这里做了一个很强的立场宣示：

* **Err on the side of loading**

  ——宁可加载过多，不可错过关键步骤
* **Load even if you think you could handle it**

  ——哪怕你自认为能搞定，也要加载，因为 skill 定义了"此处应该怎么做"

**⑦项目上下文（仅Soul缺失时导入）**

当且仅当 soul.md 不存在时，加载项目相关信息——如 `README.md`、`HERMES.md` 等。

**这个"仅当"条件很关键**：Hermes 认为，如果你已经定义了 Agent 的人格（soul），那么项目上下文应该由 skill 或 memory 去承载；只有在"裸跑"状态下（也就是初始化时，没有人格设定），才临时用项目文档填充上下文。这是对提示词预算的精打细算。

**⑧会话元信息**

追加当前会话的基础元信息：

* **timestamp —— 当前时间**
* **session\_id —— 会话标识**
* **model —— 使用的模型**
* **provider —— 模型提供商**

这些信息让 Agent 具备**自我定位能力**——它知道自己是谁、在哪、什么时候、用的什么"身体"。

**⑨运行环境信息**

追加 Agent 所处的运行环境描述。例如在 WSL 环境下会加：

```
You are running inside WSL (Windows Subsystem for Linux).The Windows host filesystem is mounted under /mnt/ — /mnt/c/ is the C: drive, /mnt/d/ is D:, etc. The user's Windows files are typically at /mnt/c/Users/<username>/Desktop/, Documents/, Downloads/, etc. When the user references Windows paths or desktop files, translate to the /mnt/c/ equivalent. You can list /mnt/c/Users/ to discover the Windows username if needed.
```

这段提示词解决的是一个**非常具体的痛点**：WSL 用户说"打开我桌面上的文件"时，Agent 往往傻眼——它看到的是 Linux 文件系统，用户想的是 Windows 桌面。这里直接告诉 Agent 路径映射规则，并给出"如何自主发现用户名"的兜底方法。

**⑩平台信息**

最后一步，追加平台相关提示词。例如 CLI 环境下：

```
You are a CLI AI Agent. Try not to use markdown but simple text renderable inside a terminal.
```

**CLI 不渲染 markdown**——这个细节决定了 Agent 的输出风格。如果不加这段提示，Agent 会默认输出大量粗体、`# 标题`、表格，在终端里变成一堆乱糟糟的星号和井号。这段提示虽然只有两行，但对用户体验的影响极大。

**总结**

回看整条装配流水线，我们可以把它重新归纳为**五个逻辑层**：

| **层级** | **内容** | **特点** |
| --- | --- | --- |
| Layer 5 | 时间/会话/模型/环境/平台 | 每轮都会变化 |
| Layer 4 | 技能系统 / 项目文档兜底 | 任务方法论 |
| Layer 3 | MEMORY.md   / USER.md | 跨会话状态 |
| Layer 2 | 工具能力 / 用户自定义 | 按条件装配 |
| Layer 1 | soul.md | 人格本体 |

从下往上看，是一个Agent 从「是谁」到「有什么能力」到「记得什么」到「会做什么」再到「此刻在哪」的完整构造过程。

每一层都可以独立开关、独立替换——这让 Hermes 不是一个封闭产品，而是一个装配协议。

读完整个代码，其实我最有启发的一点是：Agent 的「人格」不是写出来的，是拼出来的。soul.md 给了它灵魂，工具提示词给了它能力边界，memory 和 skill 给了它经验，环境和平台信息给了它「此刻存在感」。每一次对话开始，这些碎片按固定顺序组装成一个具体的、当下的 Hermes。
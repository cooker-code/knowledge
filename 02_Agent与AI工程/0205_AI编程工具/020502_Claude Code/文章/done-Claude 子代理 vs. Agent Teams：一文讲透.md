> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude 子代理 vs. Agent Teams：一文讲透
author: Halo咯咯
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0NTg0Njk1OQ==&mid=2247495906&idx=1&sn=a0d4a9fda6414981b26ff8734748149e&chksm=e84c58319bcdf1345cdd07f19695ec0f858f02e0559654b10cf7d0304d719cf316c6691bba87&mpshare=1&scene=24&srcid=0422D7F1T8uzShBFOBSi71ea&sharer_shareinfo=2d7590c686c7c54748fe3963f2c883ca&sharer_shareinfo_first=2d7590c686c7c54748fe3963f2c883ca#rd
---

很多人一碰到复杂任务，直觉第一反应就是上“多智能体系统”。 但这种直觉大多数时候都不对的。

真正该考虑的并不是“要不要用多个 agent”，而是：

**这个任务到底需要什么样的协作机制？**

这个问题的答案，对于你的架构设计起着决定性作用。

Claude 提供了两种截然不同的多智能体范式：**子代理（sub-agents）** 和 **团队型智能体（agent teams）**。乍看之下，它们有点像，但从架构上说，解决的是两类完全不同的问题。

---

## 子代理：用隔离换并行

子代理可以理解成一个“专用的 Claude 实例”，运行在**彼此隔离的上下文窗口**中。

你可以把它想象成：你是研究负责人，不会自己把所有一手资料都读完，而是把不同问题分给研究员；他们各自做完分析后，带回来的是**高度提炼过的结论**，再由你统一整合成一份连贯的输出。

子代理就是这么工作的。

每个子代理都有：

* 一段定义其专长的 system prompt
* 一组边界清晰、权限受限的工具
* 一个干净且隔离的上下文窗口
* 一项明确的任务

任务结束后，**只有最终结果**会返回给父代理。 不会返回完整推理链，也不会带回中间过程，只有压缩后的输出。

子代理真正的价值不只是并行，更在于**压缩**： 它能把大规模探索蒸馏成清晰信号，避免把大量噪声塞进父代理的上下文。

这里还有一个硬约束：**子代理不能再生成子代理**，也**不能彼此直接交流**。所有结果都只会回传给父代理，父代理是唯一的协调者。

这不是缺点，恰恰是它的设计特性：信息流向清晰，决策点单一，整个系统也因此更可预测。

下面是一个最小 SDK 示例，用来定义并调用子代理：

```
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition

asyncdef main():
    asyncfor message in query(
        prompt="Review the authentication module for security vulnerabilities",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Grep", "Glob", "Agent"],
            agents={
                "security-reviewer": AgentDefinition(
                    description="Security specialist. Use for vulnerability checks and security audits.",
                    prompt="You are a security specialist with expertise in identifying vulnerabilities.",
                    tools=["Read", "Grep", "Glob"],
                    model="sonnet",
                ),
                "performance-optimizer": AgentDefinition(
                    description="Performance specialist. Use for latency issues and optimization reviews.",
                    prompt="You are a performance engineer with expertise in identifying bottlenecks.",
                    tools=["Read", "Grep", "Glob"],
                    model="sonnet",
                ),
            },
        ),
    ):
        print(message)
```

这里的 `description` 字段会告诉父代理该把任务路由给哪个子代理。因为 prompt 里提到的是 “security vulnerabilities”，所以父代理会选择 `security-reviewer`，而不是 `performance-optimizer`。如果 prompt 换成延迟问题或性能瓶颈，路由结果就会反过来。

**`description` 本质上就是路由信号**，一定要写得具体。

---

## Agent Teams：靠沟通来协作

Agent Teams 是另一种完全不同的模型。

子代理更像短生命周期的“工人”：干完活就结束。 而团队型智能体则是**长生命周期**的实例：它们会持续存在，彼此直接沟通，并依靠共享状态协同推进任务。

可以把这两者理解成： 雇一批“做完一次就走的外包” vs. 组建一个“在同一个房间里持续协作的团队”。

一个 agent team 通常包含三部分：

1. **团队负责人（team lead）**：负责协调、分派和整合结果
2. **队友（teammates）**：各自拥有独立上下文，并行执行任务
3. **共享任务列表**：记录待办、进行中、已完成，以及任务之间的依赖关系

典型生命周期如下：

```
Claude (Team Lead):
└── spawnTeam("auth-feature")
    Phase 1 - Planning:
    └── spawn("architect", prompt="Design OAuth flow", plan_mode_required=true)
    Phase 2 - Implementation (parallel):
    └── spawn("backend-dev", prompt="Implement OAuth controller")
    └── spawn("frontend-dev", prompt="Build login UI components")
    └── spawn("test-writer", prompt="Write integration tests", blockedBy=["backend-dev"])
```

注意这里 `test-writer` 的 `blockedBy`：共享任务列表此时发挥的是**真正的协调作用**。测试 agent 会自动等后端 agent 完成，而不需要负责人手动盯时序。

它和子代理最大的区别在于：**队友之间可以点对点直接沟通**。 他们可以互相发消息、共享发现、暴露阻塞、直接协商，而不用什么都先发给负责人，再由负责人转发。

你也可以直接和某个队友互动，而不必永远只能对负责人说话。

---

## 核心区别：一次性委派 vs. 持续性协作

两者的选择逻辑，其实可以浓缩成一句话。

**子代理适合“一次性委派，发出去就等结果（fire-and-forget）”**：

* 你把任务交出去
* 它完成后回报结果
* agent 之间不沟通
* 没有共享记忆，也没有持续状态
* 每个子代理只存在于那一次调用里

**Agent teams 适合“持续协作（collaborative）”**：

* agent 会持续存在，并不断累积上下文
* 中途发现可以实时同步给队友
* 比如前端 agent 可以直接告诉后端 agent：“API 响应结构得改”，后端立刻调整，而不必等负责人来回转述

最实用的选型标准是：

* **用子代理**：当任务属于“尴尬式并行（embarrassingly parallel）”——彼此独立的研究流、代码库探索、信息检索，父代理只需要摘要结果
* **用团队**：当任务需要“持续协商”——多个线程必须先对齐输出才能继续，或者一个线程的新发现会直接改变另一个线程接下来该怎么做

---

## 从第一性原理出发设计 Agent 系统

多数多智能体系统之所以设计失败，不是因为模型不够强，而是因为拆分方式错了：人们总按“角色”拆，而不是按“上下文”拆。

按角色拆看起来很自然：规划、实现、测试，各司其职，似乎很清晰。

但实际运行起来，往往会变成一场电话游戏：每交接一次，信息就衰减一次。

* 实现者拿不到规划者掌握的细节
* 测试者不知道实现者做过哪些隐含决策
* 边界越多，质量越容易出问题

更合理的方式是：**围绕上下文来拆分（context-centric decomposition）**。

你真正该问的是：这个子任务到底需要哪些上下文？ 如果两个子任务依赖的信息高度重叠，那它们大概率就该由同一个 agent 来完成。 只有在它们确实可以在相互隔离的信息环境中工作，而且接口明确、交付清晰时，拆分才有意义。

一个很实用的例子是：负责实现某个功能的 agent，通常也应该顺手把这个功能的测试写掉——因为它已经掌握了完整上下文。硬把“实现”和“测试”拆成两个 agent，交接带来的损耗，往往比并行节省下来的时间还大。

**只有当上下文真的能隔离开时，才值得拆分。**

---

## 五种最值得掌握的编排模式

不管你最后选哪种范式，这五种模式基本覆盖了大多数真实需求：

1. **提示链（prompt chaining）**：按顺序执行，后一步处理前一步的输出。适合步骤之间依赖很强、顺序不能打乱的任务。
2. **路由（routing）**：先做分类，再决定把任务交给哪个专门处理器。简单问题走更快更便宜的模型，复杂问题交给更强的模型，用来控制成本。
3. **并行（parallelization）**：让独立子任务同时运行。既可以是同一个任务多次生成再投票，也可以是不同子任务并发处理。
4. **编排者—工人（orchestrator-worker）**：中心 agent 负责拆任务、分发给工人、最后整合结果。这是子代理和团队模式中都最常见、也最容易落地的生产级架构。
5. **评估者—优化者（evaluator-optimizer）**：一个负责生成，一个负责评估和反馈，循环迭代。适合质量要求高、单次输出不够稳定的场景。

---

## 什么时候根本不该用多智能体？

很多文章都会绕开这个问题。

但现实是，不少团队花了几个月搭出复杂的多智能体流水线，最后发现：**其实把单个 agent 的 prompt 写清楚，同样能做出来。**

所以应该先从简单方案开始。 只有当你能量化证明“多智能体确实解决了问题”，再去增加复杂度。

多智能体真正值得付出成本的场景，通常只有三类：

1. **保护上下文**：某个子任务会产生大量和主任务无关的信息，适合用子代理隔离，避免上下文膨胀。
2. **真正并行**：多个独立的研究或搜索任务，并发越多，覆盖越好。
3. **强专门化**：任务需要相互冲突的 system prompt，或者一个 agent 同时管理太多工具后表现开始下降。

而在下面这些情况下，多智能体通常反而是错误选择：

* agent 之间需要频繁共享上下文
* 任务依赖链很长，协调成本高过执行收益
* 问题本身并不复杂，一个 agent 就够了

对“写代码”这件事，还要额外提醒一句：多个并行 agent 同时写代码，很容易各自形成一套不兼容的隐含假设；真正合并时，这些冲突会把调试成本抬得很高。**在代码场景里，子代理更适合做探索、检索和回答问题，而不适合和主 agent 并行地产出代码。**

---

## 多智能体系统最常见的三种失败模式

1. **任务描述不清，导致重复劳动**每个 agent 都需要明确的目标、预期输出格式、工具或信息来源说明，以及清楚的边界：哪些内容不归它负责。否则，很容易两个 agent 在研究同一件事，却谁也不知道。
2. **“验证 agent”说自己验证完了，但其实并没真验证**验收标准必须具体，例如：要跑哪些测试、覆盖哪些用例、满足什么条件之前不能标记为完成。标准一旦含糊，假阳性就会大量出现。
3. **Token 成本上涨得比预想快得多**应对办法通常是分层用模型： 把最强的模型留给真正关键的环节；常规工作路由给更快、更便宜的模型；再加上预算控制，避免成本失控。

---

## 唯一真正重要的设计原则

**围绕上下文边界设计，而不是围绕角色分工或组织架构设计。**

先从单个 agent 开始，把它用到极限。 它最先崩在哪里，那个崩点就会告诉你，下一步到底该加什么。

只在确实能解决真实、可度量问题的地方，引入复杂性的架构。

---

翻译至：https://x.com/akshay\_pachaar/status/2033167408463069526

*如果你喜欢这篇文章，别忘了****关注****我们！*

---

AI在持续学习，你我也该更新了。
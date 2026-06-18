> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 我读完了 Claude Code 的全部源码，这是我找到的 Agent 设计真相
author: 小Z修炼中
date: 全栈开发coding全栈开发coding
url: https://mp.weixin.qq.com/s?__biz=MzkwMDYwODI2Nw==&mid=2247485969&idx=1&sn=031d2b665409aef950d546978ca445e1&chksm=c1e80e96675fb80b3a6cc59f4b522364a13bee27d27a50d12f1bf998c4eba841149b0a02bae0&mpshare=1&scene=24&srcid=0430e2XGEcOIEtcZl7QEiynX&sharer_shareinfo=629e742b36d9342798578432da5b07c5&sharer_shareinfo_first=629e742b36d9342798578432da5b07c5#rd
---

一个日活巨大的 Agent，它的骨架到底长什么样？

## 为什么偏偏是 Claude Code？

我看过很多 Agent 框架的源码。LangChain 像一个过度工程的乐高积木，零件多到你忘了自己在拼什么。AutoGPT 像一个概念验证原型，跑起来很酷但你不敢用在真实项目上。

Claude Code 不一样。

这是 Anthropic 自家内部孵化的 AI 编程助手，在推出几个月后成为开发者使用最多的 AI Coding 工具之一。它每天承受几万开发者的高强度使用——写代码、跑命令、改配置、管理 Git、甚至自动创建 PR。

在今年初源码意外泄露后，我做了一件大部分人不会做的事：我没有只看 README，而是从 `main.tsx` 开始，一行一行地通读了整个项目。

我的结论：**Claude Code 是目前工程质量最高的 Agent 实现，没有之一。**

不是因为它用了什么黑科技——恰恰相反，它的每一个设计选择都"显而易见"。但"显而易见"的正确选择叠加在一起，就构成了一个令人惊叹的系统。

这篇文章讲的是 Claude Code 的骨架——Agent 是怎么跑起来的，多 Agent 是怎么协调的，以及一个改变了我认知的设计："Markdown as Agent"。

## 一条死循环，撑起整个 Agent

打开 Claude Code 的核心文件 `query.ts`，你会看到一个朴素到让人失望的结构：

就这？就这。

Claude Code 的全部 Agent 逻辑——主代理、子代理、远程代理——都运行在这同一个循环里。没有状态机，没有 DAG，没有 Agent Graph。只有一个 `while(true)` 和一个 `yield`。

💡 关键洞察一

**用 AsyncGenerator 而不是 Promise 或回调。** 这个选择看起来只是"代码风格"的差异，实际上决定了整个系统的吞吐特性。AsyncGenerator 天然支持**背压控制**——当下游（终端 UI）来不及渲染时，上游（LLM 流）会自动暂停。如果用回调或 EventEmitter，你需要自己实现流量控制，否则 Claude 输出太快会把终端打崩。

更妙的是，`yield` 让父调用者可以在任意时刻中断 Agent 循环——用户按 Ctrl+C 时，中断信号可以精确地传播到当前正在执行的那一步。

但真正让这个循环变得不简单的，是它和工具系统的配合方式。

## 工具不是"附件"——工具就是 Agent 的手脚

Claude Code 有 40 多个内置工具。但让我印象最深的不是数量，而是它的设计哲学：**每个工具都是一个自包含的单元**。

💡 关键洞察二

**Fail-Closed 默认值是 Agent 安全的基石。** 所有安全相关的布尔属性默认都是最保守的值。`isConcurrencySafe` 默认 `false`——系统默认认为你的工具不能并发执行。你想并发？你自己声明 `true`，自己负责。

这叫 **Fail-Closed 设计**。在传统软件里这可能只是"最佳实践"，但在 Agent 系统里这是**生存必需**。因为 Agent 的行为是 LLM 驱动的，本质上是不可预测的。如果默认值是宽松的，一个 LLM 的幻觉就可能导致两个写操作并发执行、互相覆盖。

还有一个巧妙的细节：工具支持**延迟发现**。不常用的工具标记 `shouldDefer: true`，不会出现在初始 System Prompt 中。LLM 需要先调用 `ToolSearch` 来"发现"它们。Skill 列表的 Token 预算被精算为上下文窗口的 1%——大约 2000 tokens。这像什么？像操作系统的按需加载。

## 创建子 Agent？调用一个 Tool 就够了

这是我认为 Claude Code **最优雅的设计**——AgentTool。

很多 Agent 框架把多 Agent 协作搞成了一个复杂的编排问题：定义 Agent Graph、配置通信拓扑、编写协调逻辑。Claude Code 的做法简单得让人怀疑"这能行吗"：

**创建子 Agent = 调用一个普通工具。LLM 自己决定什么时候需要帮手。**

子 Agent 的隔离机制值得细看：

💡 关键洞察三

**子 Agent 的工具集是父 Agent 的子集，不是超集。** 这和 Linux 的 `fork() + drop capabilities` 思路完全一致。子进程继承父进程的权限，但可以主动放弃部分权限，不能获取父进程没有的权限。

在 Claude Code 中，有两级工具限制清单：

* `ALL_AGENT_DISALLOWED_TOOLS` 

  — 所有子代理都不能用的工具
* `CUSTOM_AGENT_DISALLOWED_TOOLS` 

  — 只有用户自定义代理额外被限制的工具

这意味着即使 LLM 产生了"给自己更大权限"的意图，系统层面也不可能实现。

⚠️ 你可能忽略的细节：子 Agent 的 MCP 服务器继承

子 Agent 可以定义自己的 MCP 服务器连接——但这些连接是**增量式**的。子 Agent 的清理逻辑只清理自己新创建的连接，共享的连接由父上下文管理。这避免了子 Agent 退出时意外关闭父 Agent 正在使用的 MCP 连接。

## 三种多 Agent 协调模式

Claude Code 不止一种多 Agent 模式。根据场景不同，提供了三种协调方式：

PART 1

**父子模式（AgentTool）**——最常用。父 Agent 通过 AgentTool 创建子 Agent，结果通过 `tool_result` 返回。简单、可控、可追溯。

PART 2

**协调者模式（Coordinator）**——通过环境变量 `CLAUDE_CODE_COORDINATOR_MODE` 启用。一个协调者 Agent 管理多个 Worker Agent。Worker 的工具集被严格限制为 `ASYNC_AGENT_ALLOWED_TOOLS` 白名单。

PART 3

**Swarm 模式（Team）**——最复杂的模式。多个 Agent 作为"队友"平等协作，通过 `SendMessageTool` 互相通信。支持动态管理团队。

我的判断：**对 90% 的场景，父子模式就够了。** Coordinator 模式适合"一个大任务拆成独立子任务"的场景，Swarm 模式适合"多个 Agent 需要持续交互"的场景（比如代码审查 + 测试 + 部署的流水线）。

## "做梦"——最诡异也最聪明的任务类型

Claude Code 定义了 7 种任务类型。前 6 种都好理解——Shell 命令、本地 Agent、远程 Agent、进程内队友、工作流、MCP 监控。但第 7 种叫 **Dream（做梦）**。

DreamTask 是自动记忆整合机制。它在后台运行，回顾过去的对话记录，把散落的信息整合成结构化的记忆文件。它有 4 个阶段：

1. **Orient** 

   — 确定方向：看看最近的对话里有什么值得记住的
2. **Gather** 

   — 收集信息：从对话中提取关键事实
3. **Consolidate** 

   — 整合记忆：把新信息和已有记忆合并
4. **Prune** 

   — 修剪冗余：删除过时或重复的记忆

这让我想起人类睡眠中的记忆巩固过程——白天经历的零散信息，在夜间被整理成长期记忆。Claude Code 给 Agent 实现了类似的机制。

## Markdown as Agent：一个文件定义一个 Agent

这是我认为最有想象力的设计——**SKILL.md**。

传统 Agent 框架要求你用代码定义 Agent 行为。Claude Code 说：不，一个 Markdown 文件就够了。

一个 SKILL.md 文件同时定义了：

* **身份**

  （description）—— 它做什么
* **能力**

  （tools 白名单）—— 它能用什么
* **行为**

  （正文 = System Prompt）—— 它怎么做
* **约束**

  （model、effort）—— 它的运行参数
* **参数**

  （arguments + $变量替换）—— 它的输入接口

Skill 支持两种执行模式：

* **Inline**

  ：内容注入到当前对话上下文——共享上下文、省 Token
* **Forked**

  ：在独立的子 Agent 中执行——完全隔离、可用不同模型

💡 关键洞察四

**Markdown 是 Agent 最好的"接口语言"。** 人类可读、版本控制友好、零编译、低门槛——产品经理也能定义 Agent 行为。Claude Code 有 101 个内置 Slash Command，加上无限的用户自定义 Skill。这个数字本身就说明了 Markdown as Agent 的扩展能力。

## 从源码里偷到的 4 条经验

**1. 权限系统是第一天的设计，不是 v2.0 的功能。** Claude Code 15%+ 的代码在做权限和安全。这不是过度工程——一个没有权限约束的 Agent，就是一颗定时炸弹。

**2. 工具自描述比外部文档可靠十倍。** 把 Schema、使用说明、安全属性都绑在工具定义里。工具改了，描述自动跟着改。

**3. 上下文窗口要像内存一样管理。** 有预算分配、有回收策略、有告警机制。

**4. 最好的扩展方式是最简单的。** 不需要学新语言、不需要编译——一个 Markdown 文件放到目录里，就是一个新的 Agent 能力。

下一篇，我们深入 Claude Code 最硬核的部分：**工具系统的 6 层权限决策链、Hook 系统的安全不变量、以及 BashTool 的 AST 级安全分析**。如果说这篇讲的是 Agent 的骨架，下一篇讲的就是它的铠甲。

作者：ZhaiYaDong

本文基于 Claude Code 源码分析项目，共 13 篇文档、9013 行分析笔记。
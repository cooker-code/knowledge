---
title: Claude Code 中 10 个必用的斜杠命令
author: 林克养虾不摸鱼
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzMzgwNjc4OA==&mid=2247484202&idx=1&sn=b3fe2e75979a917d7f89190d162be77a&chksm=f14f3fad60045a8652c69fde6f08505651c1dc47fca501da7d63146650ebf9ef24f75d1b6850&mpshare=1&scene=24&srcid=0416QFbOCKdycxzjJYFrDyjZ&sharer_shareinfo=84c984b246d8e57c887bce96f2dd9617&sharer_shareinfo_first=84c984b246d8e57c887bce96f2dd9617#rd
---

**今日推送内容：**

* 技术性 LLM 面试题！
* Claude Code 中 10 个必用的斜杠命令。
* [实战] 为 Agent 构建实时联邦数据引擎。

### **Agents**

**技术性 LLM 面试题！**[2]

|

你有 80,000 条来自生产环境的 Agent 轨迹（Trajectory）。你需要从中找出最值得审查的 100 条来改进你的 Agent。

不允许使用 LLM 来评估轨迹。你会怎么做？

让我们来看看几种方法。

最简单的起步方案是**随机采样**。随机挑选 100 条轨迹进行审查。

但大多数生产环境中的 Agent 处理日常请求都没什么问题，所以你最终会浪费大量的标注预算。

另一种方法是筛选较长的对话，因为超过 10 条用户消息意味着更高的复杂度。

但较长的对话严重偏向于完全失败的情况。你会发现明显的崩溃问题，却会遗漏那些隐藏在 Agent 技术上成功完成任务的对话中的微妙问题。

**DigitalOcean 最近的一篇论文**[3]提出了一种新方法。

它使用确定性规则直接从轨迹数据中计算轻量级的行为信号（Behavioral Signal）。

这些信号分为三组：

1. **交互信号（Interaction Signals）**：

* 如果用户重新表述请求或纠正 Agent，那就是**对齐偏差（Misalignment）**。
* Agent 重复自己则是**停滞（Stagnation）**。
* 用户放弃 Agent 是**脱离（Disengagement）**。
* 用户确认某个功能有效则是**满意（Satisfaction）**。

所有这些都通过归一化的短语匹配和相似度检查来检测。

2. **执行信号（Execution Signals）**：

* 一个没有推进任务的工具调用是失败信号。
* 使用相同或漂移输入的重复调用表明进入了循环。

这些可以直接从执行日志中提取。

3. **环境信号（Environment Signals）**，如速率限制、上下文溢出和 API 错误。

* 对诊断有用，但不适合用于训练，因为它们反映的是系统约束，而不是 Agent 决策。

每条轨迹根据触发的信号进行评分，然后采样得分最高的轨迹进行审查。

在 τ-bench 上，他们对 100 条轨迹比较了三种方法：

* 随机采样的信息量命中率为 54%。
* 基于长度的启发式方法达到了 74%。
* 基于信号的采样达到了 82%。

这意味着大约每 5 条轨迹中有 4 条对改进 Agent 真正有用。

事实上，在 Agent 正确完成任务的对话中，基于信号的采样仍然在 66.7% 的案例中识别出有用的模式，而随机采样只有 41.3%。

这些就是那些微妙的问题——比如策略违规、低效的工具使用、不必要的步骤——它们不会导致任务失败，但对优化仍然很重要。

整个框架无需任何 LLM 开销即可运行，可以在生产流水线中常驻运行。

如果你想在实践中看到这个方法，这种基于信号的方法已经集成到了 **Plano**[4] 中，这是一个开源的 AI 原生代理，集路由、编排、护栏和可观测性于一体。

**Plano GitHub 仓库在这里 →**[5]

**论文在 arxiv 上 →**[6]

👉 轮到你了：你的解决方案是什么？

**Claude**

**Claude Code 中 10 个必用的斜杠命令**[7]

设置 Shell 别名是终端工作中非常自然的一部分，大多数开发者几乎是条件反射地去做。如果一个命令用得足够频繁，你就会给它起个别名。但对于 Claude Code 的提示词，开发者通常完全跳过这一步，一直凭记忆重复输入相同的 10-15 行指令——比如代码审查清单、测试生成约束、提交前扫描……一个会话接一个会话地重复。

真正的成本不仅仅是开发者的重复劳动，还有**提示词漂移（Prompt Drift）**。

每次凭记忆重新输入提示词时，措辞都会略有变化。例如，你可能忘记一个约束条件，或者对预期输出格式的表述方式不同。

对于 Shell 命令来说这无所谓，因为它们是确定性的，但对于 LLM，措辞的轻微不同可能产生明显不同的输出。

Claude Code 的\*\*自定义命令（Custom Commands）\*\*同时解决了这两个问题。

你可以在 `.claude/commands/` 中保存一个 Markdown 文件，它就变成了一个斜杠命令，每次都能以完全相同的指令调用。

这些提示词通过 Git 进行版本控制，所以整个团队运行相同的命令，当有人改进了一个提示词，所有人在下次 pull 时都能获得更新。

这与 Boris Cherny 在他关于 Claude Code 工作流的帖子中描述的模式相同——他把每个重复的工作流都变成命令，提交到 Git，与团队共享：

让我们先看看如何设置，然后介绍在我的工作流中最有用的 10 个命令。我们将在一个真实的 ML 推理服务（FastAPI、scikit-learn、Alembic）上演示每一个命令，让你看到实际输出，并提供完整的提示词模板，可以直接应用到你自己的项目中。

---

#### **自定义命令的工作原理**

自定义命令就是 `.claude/commands/` 目录中的一个 Markdown 文件。文件名就是命令名。

文件内容就是运行命令时发送给 Claude 的提示词。你可以用 `$ARGUMENTS` 作为占位符，代替命令名之后输入的任何内容。

例如，运行 "/dissect src/auth/session.ts" 会将 `$ARGUMENTS` 替换为 "`src/auth/session.ts`"。

你还可以使用 !command 语法通过 Shell 命令注入动态上下文：

Claude 在处理提示词之前会先运行这些 Shell 命令，所以上下文始终是最新的。

最后，文件顶部的可选 YAML frontmatter 让你可以预授权工具（这样 Claude 不会在每次 git 调用时都请求许可）、设置模型覆盖或添加描述：

这就是整个系统——一个 Markdown 文件、一个可选的 YAML 头部和用于动态输入的 `$ARGUMENTS`。

以下是我们在实践中发现最有用的 10 个命令：

由于篇幅限制，后续内容无法通过邮件完整分享。

**我们在这里分享了完整的设置指南，包含使用视频和提示词 →**[8]

### **实战**

**为 Agent 构建实时联邦数据引擎**[9]

Agent 的实时同步极其困难，尤其是当你的数据分散在几十个来源时。大多数团队要花好几周为每个数据库、API 和数据仓库构建自定义连接器，然后再构建 ETL 管道来同步所有数据。等你的 Agent 检索到数据时，它已经过时了。想象一下，你的 Postgres 数据库 5 分钟前刚更新，MongoDB 集合 2 分钟前发生了变化，但你的 Agent 还在拉取昨天的快照。这就是大多数生产环境 RAG 系统失败的原因。有一种更好的方法： |

MindsDB 是一个开源 AI 平台，配备联邦数据引擎（Federated Data Engine），让你可以使用 SQL 实时查询多个数据源，而无需移动任何数据。

它的独特之处在于：

* 你的数据保留在原位。无需 ETL 管道或数据复制
* 使用统一的 SQL 查询 Postgres、MongoDB、REST API 等
* 通过统一接口实时跨不同数据源 JOIN
* 同时支持结构化和非结构化数据

最精彩的部分是：

你甚至不需要写 SQL。只需用自然语言描述你想要什么，MindsDB 就会自动将其转换为 SQL。系统会完成所有繁重的工作。

对 AI Agent 来说，突破性在于一个简单的事实：

当数据在源头更新时，你的 Agent 立即获得最新结果——没有同步延迟、没有过期的 Embedding、也不需要为每个集成编写自定义代码。

你可以直接编写一条 SQL 查询，将 Postgres 表与 MongoDB 集合进行 JOIN，获得实时结果。这就是生产级 AI 应用所需要但很少能获得的能力。

在下面的视频中，我们提供了完整的操作演练，包括我们刚刚讨论的内容以及如何实际操作。

[10]

**MindsDB GitHub 仓库在这里 →**[11]

**MindsDB GitHub 仓库**[12]

### 引用链接

[1]掌握全栈 AI 工程: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/dpheh0henzl38gum/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWVtYmVyc2hpcC8=*

[2]技术性 LLM 面试题！: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/e0hph7h7o98xq3s8/aHR0cHM6Ly9hcnhpdi5vcmcvcGRmLzI2MDQuMDAzNTY=*

[3]DigitalOcean 最近的一篇论文: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/e0hph7h7o98xq3s8/aHR0cHM6Ly9hcnhpdi5vcmcvcGRmLzI2MDQuMDAzNTY=*

[4]Plano: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/7qh7h8h95vrgwrtz/aHR0cHM6Ly9naXRodWIuY29tL2thdGFuZW1vL3BsYW5v*

[5]Plano GitHub 仓库在这里 →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/7qh7h8h95vrgwrtz/aHR0cHM6Ly9naXRodWIuY29tL2thdGFuZW1vL3BsYW5v*

[6]论文在 arxiv 上 →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/e0hph7h7o98xq3s8/aHR0cHM6Ly9hcnhpdi5vcmcvcGRmLzI2MDQuMDAzNTY=*

[7]Claude Code 中 10 个必用的斜杠命令: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/owhkhqhwd32zgqav/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vcC8xMC1tdXN0LXVzZS1zbGFzaC1jb21tYW5kcy1pbi1jbGF1ZGUtY29kZS8=*

[8]我们在这里分享了完整的设置指南，包含使用视频和提示词 →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/owhkhqhwd32zgqav/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vcC8xMC1tdXN0LXVzZS1zbGFzaC1jb21tYW5kcy1pbi1jbGF1ZGUtY29kZS8=*

[9]为 Agent 构建实时联邦数据引擎: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/z2hghnhex9qgpmcp/aHR0cHM6Ly9naXRodWIuY29tL21pbmRzZGIvbWluZHNkYg==*

[10] : *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/p8heh9h4oknvgehq/aHR0cHM6Ly9hcGkuZmlsZWtpdGNkbi5jb20vZS9rN1lIUE4yNFNveHlNOG5HS1puRHhhLzY3N1hiVHo3OWJOQjhiQjljQzNIQkEvcGxheWVy*

[11]MindsDB GitHub 仓库在这里 →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/z2hghnhex9qgpmcp/aHR0cHM6Ly9naXRodWIuY29tL21pbmRzZGIvbWluZHNkYg==*

[12]MindsDB GitHub 仓库: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/z2hghnhex9qgpmcp/aHR0cHM6Ly9naXRodWIuY29tL21pbmRzZGIvbWluZHNkYg==*

[13]\*\* 在 AI 工程岗位上取得成功\*\*: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/x0hph6he085g9ga5/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWVtYmVyc2hpcA==*

[14]掌握全栈 AI 工程: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/x0hph6he085g9ga5/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWVtYmVyc2hpcA==*

[15]18 部分的课程中从零开始学习 MLOps →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/dpheh0henzl35zbm/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWxvcHMtY3Jhc2gtY291cnNlLXBhcnQtMS8=*

[16]9 部分的课程中学习 MCP 的一切 →: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/e0hph7h7o98x4pt8/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbW9kZWwtY29udGV4dC1wcm90b2NvbC1jcmFzaC1jb3Vyc2UtcGFydC0xLw==*

[17]这个 14 部分的课程中学习如何构建 Agentic 系统: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/7qh7h8h95vrgz6iz/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vYWktYWdlbnRzLWNyYXNoLWNvdXJzZS1wYXJ0LTEtd2l0aC1pbXBsZW1lbnRhdGlvbi8=*

[18]这个课程中学习如何构建真实世界的 RAG 应用、评估和扩展: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/owhkhqhwd32zoncv/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vYS1jcmFzaC1jb3Vyc2Utb24tYnVpbGRpbmctcmFnLXN5c3RlbXMtcGFydC0xLXdpdGgtaW1wbGVtZW50YXRpb25zLw==*

[19]这个课程中学习复杂的图架构以及如何在图数据上训练: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/z2hghnhex9qgwrfp/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vYS1jcmFzaC1jb3Vyc2Utb24tZ3JhcGgtbmV1cmFsLW5ldHdvcmtzLWltcGxlbWVudGF0aW9uLWluY2x1ZGVkLw==*

[20]这里学习可扩展的方法: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/p8heh9h4oknvxlsq/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vYmktZW5jb2RlcnMtYW5kLWNyb3NzLWVuY29kZXJzLWZvci1zZW50ZW5jZS1wYWlyLXNpbWlsYXJpdHktc2NvcmluZy1wYXJ0LTEv*

[21]量化技术: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/x0hph6he085g37c5/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vcXVhbnRpemF0aW9uLW9wdGltaXplLW1sLW1vZGVscy10by1ydW4tdGhlbS1vbi10aW55LWhhcmR3YXJlLw==*

[22]保形预测（Conformal Predictions）: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/6qheh8hlevmqopuo/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vY29uZm9ybWFsLXByZWRpY3Rpb25zLWJ1aWxkLWNvbmZpZGVuY2UtaW4teW91ci1tbC1tb2RlbHMtcHJlZGljdGlvbnMv*

[23]这个课程中学习如何使用因果推断识别因果关系并回答商业问题: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/kkhmh6hnvgk7mlbl/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vYS1jcmFzaC1jb3Vyc2Utb24tY2F1c2FsaXR5LXBhcnQtMS8=*

[24]实践指南中学习如何扩展和实现 ML 模型训练: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/58hvh7hg2pre7gb6/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vaG93LXRvLXNjYWxlLW1vZGVsLXRyYWluaW5nLw==*

[25]在生产环境中测试新模型的技术: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/25h2hoh3wrk5d4t3/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vNS1tdXN0LWtub3ctd2F5cy10by10ZXN0LW1sLW1vZGVscy1pbi1wcm9kdWN0aW9uLWltcGxlbWVudGF0aW9uLWluY2x1ZGVkLw==*

[26]联邦学习（Federated Learning）: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/qvh8h7hdpg9xnnil/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vZmVkZXJhdGVkLWxlYXJuaW5nLWEtY3JpdGljYWwtc3RlcC10b3dhcmRzLXByaXZhY3ktcHJlc2VydmluZy1tYWNoaW5lLWxlYXJuaW5nLw==*

[27]压缩 ML 模型: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/g3hnh5hmw74vpxfr/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbW9kZWwtY29tcHJlc3Npb24tYS1jcml0aWNhbC1zdGVwLXRvd2FyZHMtZWZmaWNpZW50LW1hY2hpbmUtbGVhcm5pbmcv*

[28]掌握全栈 AI 工程: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/x0hph6he085g9ga5/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWVtYmVyc2hpcA==*

[29]更新你的资料: *https://preferences.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn*

[30]取消订阅: *https://fff97757.unsubscribe.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn*

[31]高级 DS/ML 资源: *https://fff97757.click.kit-mail3.com/8kum44pwwlsoh2n9pdlikhkow8npvb3hoqxnn/dpheh0henzl38gum/aHR0cHM6Ly93d3cuZGFpbHlkb3Nlb2Zkcy5jb20vbWVtYmVyc2hpcC8=*
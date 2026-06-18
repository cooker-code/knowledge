---
title: Harness Monitor：当多个 Agent 同时写代码时，如何看住质量
author: phodal
date: PhodalPhodal
url: https://mp.weixin.qq.com/s?__biz=MjM5Mjg4NDMwMA==&mid=2652980449&idx=1&sn=0292829ac04841867df6362802966e9b&chksm=bccb3f2bfea6a1207d99c80ef367e17269c8b592ea4c81d742c19dd7d407dbb7702af13fdba2&mpshare=1&scene=24&srcid=06054AZgmqei28jpptSToYnC&sharer_shareinfo=ed49293b2eebc6766828d9a42f070934&sharer_shareinfo_first=ed49293b2eebc6766828d9a42f070934#rd
---

TL;DR：https://github.com/phodal/routa，Harness Monitor 在 `crates/harness-monitor` 目录下。 通过 `npm install -g harness-monitor`，即可安装和使用。

在有了 OpenAI 赞助的 Codex Pro 之后，使用多个 Codex CLI 实例已经成了我的日常。再加上 Claude Code、Codex 这些 Coding Agent 天然适合并行处理不同任务，我现在很自然地会让多个 Agent 同时在一个代码库里工作。

但我一直不太喜欢 Git Worktree。我的 1T 硬盘早就处于“周周清理空间”的状态，再加上多个 worktree 带来的目录切换、环境维护和额外心智负担，我最后还是回到了一个更直接的做法：**多个任务，同时跑在同一个代码库上。**

在 AI 时代，让代码“被写出来”已经越来越容易了，真正变难的是：**如何继续看懂代码，并且看住质量。**

我这次做 Harness Monitor，起点其实很具体。它不是先从什么宏大的架构治理概念开始，而是从 Git 视角里一个很朴素的问题一点点长出来的：先回答“谁改了什么”，再回答“这些改动意味着什么”，最后才走向“这次改动是否具备继续推进的质量条件”。

从 AgentWatch 的本地归因，到 Routa Watch 的 Fitness 融合，再到 Harness Monitor 把问题收敛为 `Context → Run → Observe → Govern` 这条四层语义，背后其实是一条很朴素的演进线：**从观察走向治理。**

## 从 Git Status 开始：先回答“现在发生了什么”

一个 monitor 的第一价值，是先把现场摊开。

当多个 Agent 同时在一个仓库里写代码时，最先需要回答的，往往不是复杂的治理问题，而是几个非常具体的问题：哪些文件变了，变了多少，工作区是不是已经 dirty，这些变化大概率属于哪个会话。

Harness Monitor 当前最重要的底座，仍然仍是这类 Observe → Attribute 能力：把 hook、进程、git 变化和归因关系放到同一个界面里，让人先看清现场，再谈判断；先建立观察，再进入治理。

## 从物理设计看代码腐化：哪些文件已经开始承压

但只知道“变了”还不够。 我更在意的是：**哪些地方正在变脆弱。** —— 《[基于 Git 的代码物理分析](https://mp.weixin.qq.com/s?__biz=MjM5Mjg4NDMwMA==&mid=2652977876&idx=1&sn=40fbc4535c2ec3fbc95d11c1202f9ce2&scene=21#wechat_redirect)》

逻辑架构会告诉我们系统应该如何分层，但物理架构会暴露系统在日常变更里究竟如何受力。一个文件的行数、最近 30 天的提交频率、是谁在反复修改、变更是不是总集中在同一块区域，这些信息本身就是代码腐化的早期信号，不是什么附属数据。

比如一个 1000 行文件，30 天内被 5 个 Agent 修改了 50 次，这通常已经不是“开发活跃”，而是很强的重构信号。因为它意味着这个文件开始同时承受高认知负担、高演化压力和高冲突概率。 所以在我眼里，Git 历史不只是版本记录，它也是最便宜、最真实的架构受力传感器。

## Test Mapping：从“写代码”走向“保证代码可测试”

如果说 Git Status 回答的是“发生了什么”，那 Test Mapping 回答的就是：**既然这里变了，系统现在应该验证什么。**

这也是我觉得 Harness Monitor 最有质量管理价值的一类能力。它把源文件变化投影到测试责任上：这个改动对应哪些测试文件、当前是 inline、exists、changed、missing 还是 unknown。这个能力的作用，是把”实现变化”翻译成”验证义务”。

在 Entrix 的 fitness rulebook 里，评估依据本来就强调可执行证据，而不是只看覆盖率数字；test mapping 正好把这件事拉到了实时开发现场。

## Fitness 不只是快慢扫描，还需要 Review Trigger 的重要度判断

再往前走一步，光有变更和测试还不够，还需要把这些信号和更高层的治理策略接起来。

这也是为什么我对 Fitness 函数的可视化很感兴趣。因为它能把“质量要求”从抽象的规则，变成现场可见的判断依据。某些低风险操作，也许只需要基础证据；但高风险改动，可能就需要更高等级的测试、review 或运行结果。换句话说，不同风险级别的操作，本来就应该要求不同等级的证据。

Entrix 中的 fast、normal、deep 解决的是“检查做多深”，而 review trigger 则可以解决的是“这次改动值不值得升级检查、要求更多证据、甚至直接要求人工介入”。

当前 `review-triggers.yaml` 已经把这类规则写得很明确：会针对高风险目录、敏感文件、跨边界变更、超大 diff、以及核心路径改了但没有同步补测试/文档的 evidence gap 打标签，并把 medium / high 风险改动推向 `require_human_review`。

## 多 Agent 时代，稀缺的是质量语义

如果说过去我们担心的是 AI 会不会写代码，那么在多个 Agent 同时写代码之后，我更关心的问题已经变成了另一句：

**当系统越来越会生成时，我们是否还有办法看住质量。**

从这个角度看，Harness Monitor 对我来说已经不是一个简单的终端小工具了。

它更像是在做一件很小、但很现实的事：把多 Agent 并行开发重新接回到一个可观察、可验证、可治理的闭环里。 它先从 Git Status、文件变化、会话归因这些底层信号开始，再逐步走向物理设计、Test Mapping、Fitness 分层，以及基于 review trigger 的重要度判断。它想解决的，从来不是让 Agent 写得更快。而是：**当越来越多代码由 Agent 产生时，我们还能不能持续看懂系统、约束系统，并保护系统的演化质量。**
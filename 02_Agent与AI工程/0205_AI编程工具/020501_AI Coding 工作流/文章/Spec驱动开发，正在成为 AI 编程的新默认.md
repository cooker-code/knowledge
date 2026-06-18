---
title: Spec驱动开发，正在成为 AI 编程的新默认
author: 代码麻辣烫
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NjY0MTc0NA==&mid=2247500241&idx=1&sn=a18b5327834f9232bd38d9de220db22a&chksm=c2d2a72423dbe29c0ec9afcb29764dcbaf3c1d35b72ae553e3a33f799ab3e1aa387cde9207ac&mpshare=1&scene=24&srcid=05224xdsn7QDHOwLcTSsyRab&sharer_shareinfo=6bca772b6acb3d4878d70dd453540860&sharer_shareinfo_first=6bca772b6acb3d4878d70dd453540860#rd
---

## 规格驱动开发，正在成为 AI 编程的新默认

过去一年，规格驱动开发（Spec-Driven Development，SDD）从一个博客话题，变成了 AI 编程里的默认架构选择。

Thoughtworks、Martin Fowler、GitHub、Amazon，以及一篇覆盖 67 个来源的学术综述，都在 2025 到 2026 年间指向了同一个方向：问题已经不再是要不要使用 SDD，而是该采用哪一种实现方式。

## 发生了什么

18 个月内，多个独立来源给出了相似判断。

Thoughtworks 在 Technology Radar Vol. 32 中把规格驱动开发列为值得采用的技术实践。Martin Fowler 也在自己的网站上讨论了这个方向。

GitHub 推出了 Spec Kit，一个 MIT 许可的工具包，定位是对“氛围编程”（vibe coding）的回应。Amazon 发布了 Kiro，一个在生成代码前先引导用户完成需求、设计和任务拆解的 agentic 工具。Tessl 走得更激进，把规格直接放在“新源代码”的位置上。

Red Hat 发布了面向企业的 SDD 指南，InfoQ 则从架构层面进行了报道。

Bryan Finster 提出了一个值得听的反对意见：SDD 并不是革命，它只是 BDD 换了个品牌。

这个批评反而强化了 SDD 的理由。想法并不新，真正变化的是上下文。

BDD 原本是一种团队可以采用、也可以忽略的可选纪律。但当 84% 的专业开发者正在使用或计划使用 AI 工具（Stack Overflow，2025），而 46% 的代码产出已经由 AI 生成（GitHub，2025）时，规格纪律就不再只是偏好，而变成了结构性需求。

## 为什么它变得必要

过去 12 个月里，四篇学术论文从不同角度描述了同一个问题。

伦敦东大学者 Sabry Farrag 做了一项覆盖 67 个来源的系统综述，讨论 AI 编程工具的生产力悖论：它们确实提升个人层面的速度，同时也会带来系统层面的损害。

Peng 等人的随机对照实验覆盖 95 名开发者，结果显示任务完成速度提升 55.8%。但 Becker 等人的 METR 研究发现，经验丰富的开发者在成熟代码库中使用 AI，反而慢了 19%。

DORA 报告称，25% 的 AI 采用率与交付稳定性下降 7.2% 相关。Faros AI 追踪超过 10,000 名开发者后看到：合并 PR 增加 98%，代码评审时间增加 91%，缺陷增加 9%。

Microsoft Research 的 Shuvendu Lahiri 点出了底层缺口：AI 生成代码天生是“看起来合理”，不是“天生正确”。用户意图和程序行为之间的语义距离，成了可靠性的核心瓶颈。

一篇 AIware 2026 愿景论文指出了第二个缺口：代码评审评估的是合理性，不是合规性。很多 AI 生成的修改能通过测试，看起来也顺眼，但仍然偏离了它本应遵守的规则。

Deepak Babu Piskala 则写出了更偏实践的手册，把 SDD 拆成三个严谨程度和一个四阶段工作流。

Farrag 的经济学解释把这些线索合在一起：针对特定代码库生成的代码有很高的资产专用性，LLM 又带来很高的行为不确定性。

开发者每天会调用 AI 数百次。用交易成本经济学的话说，高资产专用性、高行为不确定性、高频调用三者叠加后，理性的治理方式就是一份可书写、可执行的契约。SDD 就是这份契约。

## SDD 实际怎么工作

对实践者来说，SDD 可以压缩成三件事。

**「第一，四阶段工作流。」** 先说明软件应该做什么，再规划如何构建，然后以小步、可验证的方式实现，最后验证代码是否满足规格。每个阶段都会产出一个约束下一阶段的工件。

**「第二，三个严谨层级。」** Spec-first 是先写规格再写代码，但规格之后可能漂移。Spec-anchored 是规格与代码并存，并通过测试强制二者一致。Spec-as-source 则更进一步，规格成为人类唯一编辑的工件，代码由系统重新生成，而不是手工修改。

**「第三，一条治理光谱。」** Farrag 的论文按约束强度列出了四类机制：

* 事后评审最松，开发者在 AI 输出后再检查。
* 自然语言规格更进一步，把需求放在生成之前。
* 可执行契约继续收紧，要求 agent 满足测试和结构化规格文档。
* 宪法式治理最严格，用一份元规格定义每次变更都必须遵守的不可妥协原则。

资产专用性越高、行为不确定性越强、调用越频繁，理性的选择就越应该往更高约束的一端移动。成熟代码库里的生产代码，如果每天被 AI 高频修改，就更接近“宪法式治理”。一次性的原型，则可以停留在事后评审。

## 五个 SDD 仓库，各自相信什么

每个仓库都体现了一种不同的判断：复杂性应该放在哪里。

## Spec Kit：用宪法作为最高约束

GitHub 的官方工具包 Spec Kit 采用 MIT 许可，通过 Python CLI 使用。

它的复杂性理论是：把复杂性放进宪法。一份不可妥协的原则文件 `.specify/memory/constitution.md` 位于所有规格和实现之上。agent 在每一次变更、每一个会话中都必须遵守它。

它的工作流由九个 slash command 组成：

* `/speckit.constitution`
* `/speckit.specify`
* `/speckit.clarify`
* `/speckit.plan`
* `/speckit.tasks`
* `/speckit.taskstoissues`
* `/speckit.checklist`
* `/speckit.analyze`
* `/speckit.implement`

其中 constitution 和 analyze 是正式治理真正发生的地方。

Farrag 的论文把 Spec Kit 视为宪法式治理的直接实例。文中提到的结果是，上游工件生产时间从 12 小时降到 15 分钟，覆盖 PRD、设计、结构、技术规格和测试计划。

一项试点研究看到，冲刺后期 hotfix 从每个 sprint 的 3 到 5 个降到 1 到 2 个，回滚从每月 2 到 4 次降到 0 到 1 次。

Spec Kit 支持 30 多种 AI agent 集成，包括 Claude、Codex、Copilot、Cursor 和 Gemini。

这是唯一一个明确采用宪法式治理的仓库。它站在 Farrag 光谱的最高层，设置成本也最高。

## BMAD-METHOD：用命名 agent 承担权威

BMAD-METHOD 来自 BMad Code LLC，MIT 许可，通过 npm 安装，V6 版本包含 34 个以上工作流。

它的复杂性理论是：把复杂性放进角色。系统里有六个具名 persona，每个负责不同专业领域：

* Analyst Mary 负责头脑风暴和研究。
* PM John 负责 PRD。
* Architect Winston 负责 8 步架构工作流。
* Developer Amelia 负责开发故事、冲刺计划和代码评审。
* UX Designer Sally 负责界面决策。
* Tech Writer Paige 负责文档。

Party Mode 会把多个 persona 带进同一个会话，让它们从不同职业视角互相争辩。

整个生命周期分为四个阶段：分析、规划、方案设计、实现。每个阶段都有自己的工作流。

`.decision-log.md` 会记录每一个决策，形成审计轨迹。实现准备度门禁会以 PASS、CONCERNS 或 FAIL 判断是否可以进入编码；如果缺东西，就不能继续。

规划深度会根据项目风险自动调整。业余项目可能只需要 2 页 PRD，正式发布项目则需要完整规格。`bmad-help` skill 可以回答“下一步该做什么”这类自由问题。

模块生态也在扩展核心能力，包括 BMM、BMB、TEA、BMGD 和 CIS 等领域。

这是唯一一个把规格视为多 agent 组织内部通信协议的仓库。

## OpenSpec：把变更文件夹作为基本单位

OpenSpec 来自 Fission AI，MIT 许可，通过 npm 使用。

它的复杂性理论是：把复杂性放进每一次变更。每个功能都有自己的文件夹，其中包含：

* `proposal.md`：为什么要做这个变更
* `specs/`：需求和场景
* `design.md`：技术方案
* `tasks.md`：实现清单

当变更交付后，`/opsx:archive` 会把变更规格折叠回持续增长的事实来源文档。

核心表面只有三个命令：

* `/opsx:propose` 创建变更文件夹。
* `/opsx:apply` 让 AI 按任务清单实现。
* `/opsx:archive` 完成归档。

可选的扩展配置还会增加 `/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive` 和 `/opsx:onboard`。

它的定位很明确：优先服务 brownfield，也就是已有代码库。多数 SDD 工具偏向从零开始的项目，OpenSpec 则专门处理既有系统。它用 delta-spec 格式按每次变更追踪新增、修改和删除，这也是它能在已有代码库中工作的原因。

OpenSpec 支持 25 多种 AI assistant，通过 slash command 使用。

它把可执行契约做到了尽可能轻：没有宪法，没有具名 agent，也没有太多仪式，但规格纪律仍然保留下来。

## GSD：把上下文当作真正瓶颈

GSD 来自 TÂCHES，MIT 许可，通过 npm 安装。它由一名独立开发者为独立开发者打造。

它的复杂性理论是：把复杂性放进上下文工程。主会话上下文保持在 30% 到 40%，重活交给新的 subagent 上下文，每个 subagent 都能拿到完整的 200K token 窗口。

整个架构建立在一个假设上：会话越长，AI 输出质量越容易下降，所以系统应该让主会话保持轻量。

它的循环由六个命令组成：

* `/gsd-new-project` 负责提问、研究、需求和路线图。
* `/gsd-map-codebase` 对已有代码库做同样的事情。
* `/gsd-discuss-phase` 在规划前捕捉决策。
* `/gsd-plan-phase` 运行研究、计划和验证循环。
* `/gsd-execute-phase` 调度多波并行 subagent。
* `/gsd-verify-work` 回看已完成工作并诊断失败。

五个持久状态文件会跨会话保留：`PROJECT.md`、`REQUIREMENTS.md`、`ROADMAP.md`、`STATE.md` 和 `CONTEXT.md`。

`.planning/config.json` 控制交互模式或 yolo 模式、模型档位，以及质量 agent 开关。安装路径里还内置了包合法性检查。

GSD 提供的是通过上下文纪律实现的可执行契约，而不是更多流程仪式。它认为真正的瓶颈不是方法论，而是上下文窗口。

## Superpowers：把纪律做成自动触发

Superpowers 由 Jesse Vincent 和 Prime Radiant 构建，MIT 许可，是一个零依赖插件。

它的复杂性理论是：把复杂性放进 agent 行为塑形。skills 会在正确时机自动触发，不需要用户手动调用。工作流是强制性的，不只是建议。

`using-superpowers` skill 会在会话开始时加载，自动触发机制依赖它。只复制 skill 文件，并不等于完成了真正集成。

七个核心 skill 组成工作流：

* `brainstorming` 在任何编码前澄清粗略想法。
* `using-git-worktrees` 隔离工作区。
* `writing-plans` 把工作拆成 2 到 5 分钟的任务，包含准确文件路径和完整代码。
* `subagent-driven-development` 为每个任务派发新 subagent，并做两阶段评审：先看规格符合度，再看代码质量。
* `test-driven-development` 会删除任何先于测试写出的代码。
* `requesting-code-review` 阻断关键问题。
* `finishing-a-development-branch` 验证测试并给出合并选项。

其中 TDD 的执行方式最特别。多数 TDD 工具只是鼓励循环，Superpowers 会删除违反 TDD 的代码。

它通过 Claude 官方插件市场、Codex 官方插件市场、Factory Droid、Gemini extensions、Cursor、GitHub Copilot CLI 和 OpenCode 分发。

Superpowers 的契约发生在 agent 层，而不是用户层。用户不需要记得什么时候调用哪个 skill。

## 第六个仓库，以及反对这个类别的理由

Matt Pocock 的 Skills For Real Engineers 出现在同一组仓库里，但它其实是反方向的论点。

他的演讲《Software Fundamentals Matter More Than Ever》直接给出判断：“代码并不便宜。事实上，坏代码比以往任何时候都更昂贵。”

谈到 specs-to-code 运动时，他说：“从规格到代码，我们不是在投资系统设计，而是在撤资。”

他的立场来自一个软件工程判断：糟糕的代码库一直都昂贵，因为它们难以改变。AI 会加速这个问题。一个坏代码库叠加 AI 吞吐量，可能是这个新时代最昂贵的失败模式。

他的仓库不是工作流框架，而是一组可组合实践。每个 skill 都可以独立使用：

* `/grill-me` 会持续追问，帮助建立 Frederick Brooks 所说的共享设计概念。
* `/grill-with-docs` 增加一份 DDD 通用语言文件，让人和 AI 都能引用。
* `/tdd` 用 red-green-refactor 限制 AI 速度。
* `/improve-codebase-architecture` 按 John Ousterhout 的思路，把浅模块重构成深模块。

默认模式是 gray boxes：先设计接口，再委托实现。

支持他的一组数据是 METR 研究：经验丰富的开发者在成熟代码库中使用 AI 慢了 19%。这说明瓶颈可能是代码库质量，而不是规格质量。他认为前面五个 SDD 仓库优化错了问题。

他的仓库因为 `/grill-me` 走红。这个立场值得认真对待。

## AlphaSignal 的判断

五个 SDD 仓库和 Pocock 的反对意见，其实不是在回答同一个问题。

SDD 优化的是“看起来合理”和“真正正确”之间的缺口。Pocock 优化的是设计熵缺口。两个缺口都真实存在，相关数据也都能支持双方立场。

团队如果只选一边、忽略另一边，就只解决了一半问题。

SDD 的可靠性论证，在宪法式治理和可执行契约这两个层级最强。Spec Kit 的 constitution 机制和 BMAD 的实现准备度门禁，是真正让数学算账成立的地方。

这个论证在自然语言规格这一端最弱。到了那里，SDD 很容易退化成换了名字的提示词工程。

从几篇论文的开放问题部分看，六个仓库都还没有解决三件事。

**「Oracle adequacy。」** 现在的评估常常把模型质量、工具可靠性和测试框架质量混成一个端到端数字。我们还没有一个指标能衡量“一份规格到底值多少钱”。

**「Evidence bundles。」** 每一个被接受的变更，都应该带上一份记录：检查了什么、没有检查什么、还剩哪些风险。现有 SDD 工具还没有生成这种东西。

**「Self-evolving harnesses。」** SDD 框架本身也是软件，它们也会变化。但它们还没有一份约束自身演化的变更契约。

更好的读法是：把每个仓库看成一种关于“可靠性从哪里来”的具体理论。选择那个理论，应该取决于你的真实瓶颈。

如果你还不知道自己的瓶颈在哪里，Pocock 的批评应该先被听见。
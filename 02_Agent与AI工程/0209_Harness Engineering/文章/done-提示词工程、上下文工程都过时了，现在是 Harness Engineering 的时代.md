> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/Harness边界与演进准则|Harness边界与演进准则]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/上下文与工具加载边界|上下文与工具加载边界]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/知识沉淀与私域资产|知识沉淀与私域资产]]、[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/评测观测与质量治理|评测观测与质量治理]]
---
title: 提示词工程、上下文工程都过时了，现在是 Harness Engineering 的时代
author: Founder Park
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247523279&idx=1&sn=ee25366cb30dc002c12e1f000affd91f&chksm=c14c00309d597350d40a5e4b54c628f1a45ce9130c1bed16f5aff7d529d2bd35aa791ee13515&mpshare=1&scene=24&srcid=0313EcDXNdFeX6Y6xSVziso1&sharer_shareinfo=97b710d8d3b6562bfbe8a174e5edee73&sharer_shareinfo_first=8b85060e617d9d76e5ee3d10e88f1bb5#rd
---

Prompt Engineering 过时了，Context Engineering 也过时了。

2026 年开年，开发者社区最热的关键词叫 Harness Engineering。

2 月 5 日，HashiCorp 联合创始人 Mitchell Hashimoto 在博客发文，把 AI 辅助开发中一种正在被越来越多顶尖团队采用的工程实践正式命了名——Harness Engineering。六天后，OpenAI 发布了一份详细的内部实验报告，标题直接用了这个词。再之后，知名工程师 Martin Fowler 在 Twitter 上为 Thoughtworks 工程师对这份报告的深度分析站台。

一个月之内，Harness Engineering 从一篇博客文章变成了开发者社区的高频词。

一个新的共识正在形成：**在 AI Agent 编码领域，决定结果好坏的最大变量，往往不是模型有多聪明，而是模型被放在了一个什么样的环境里。**

LangChain 的编码 Agent 在 Terminal Bench 2.0 基准测试上，通过仅优化 Agent 运行的外部环境（文档结构、验证回路、追踪系统），排名从全球第 30 位跃升至第 5 位，得分从 52.8% 飙到 66.5%。底层模型一个参数都没改。安全研究员 Can Boluk 仅仅改变了 Agent 的代码编辑格式，Grok Code Fast 1 的基准得分就从 6.7% 跃升至 68.3%。

而 OpenAI 的那份报告，则记录了另一个更直观的工程事实：5 名工程师，五个月，零行手写代码，通过 Codex Agent 协作交付了超过 100 万行代码的生产级软件产品。

模型能力的竞赛仍在继续，但真正在一线决定 Agent 工程产出质量的杠杆，已经转移到了「环境」一侧。

这个「环境」，就是 Harness。

⬆️关注 Founder Park，最及时最干货的创业分享

---

超 22000 人的「AI 产品市集」社群！不错过每一款有价值的 AI 应用。

邀请从业者、开发人员和创业者，飞书扫码加群：

进群后，你有机会得到：

* 最新、最值得关注的 AI 新品资讯；
* 不定期赠送热门新品的邀请码、会员码；
* 最精准的 AI 产品曝光渠道

---

## **01**

## ******从 Prompt、Context 到 Harness，******

## ******业界的认知在逐渐升级******

Harness Engineering 不是凭空冒出来的概念。从 Prompt 到 Context，每个概念都对应着开发者社区对「如何让 AI 可靠工作」这个问题的一次认知升级。

**2023 年：Prompt Engineering**

这是 Prompt Engineering 的全盛期，写好一条提示词就能让 AI 交付结果。Few-shot prompting、Chain-of-Thought、角色扮演，开发者社区围绕这些技巧产出了大量教程和最佳实践。但当 AI 从 chatbot 进化为需要处理复杂任务的 Agent 时，单条指令的局限性暴露无遗。LLM 领域最活跃的技术博主 Simon Willison 后来一句话总结了这个阶段的问题：「prompt engineering 的社会推断含义已经偏离了本意。大多数人听到 prompt engineering，想到的就是对着 ChatGPT 打字。」

**2025 年中：Context Engineering**

2025 年 6 月，OpenAI 联合创始人 Andrej Karpathy 发帖：

> 「+1 for 'context engineering' over 'prompt engineering'...... 这是一门精微的艺术与科学，用恰到好处的信息填充上下文窗口，以服务于下一步操作。」

Shopify CEO Tobi Lutke 紧随其后，发布了一条获得 190 万浏览量的帖子：

> 「我真的很喜欢 context engineering 这个词。它更好地描述了核心技能：为任务提供让 LLM 有可能解决它的全部上下文的艺术。」

Simon Willison 在博客上做了总结：

> 「我认为 context engineering 会留下来。跟 prompt engineering 不同，它的推断定义跟本意高度吻合。」

Context Engineering 的核心转变在于：焦点从「写好一条指令」扩展到了「设计一个动态系统来组装上下文」。RAG、对话历史、工具输出、系统指令的编排，都成了工程师需要操心的事。

但 2025 年下半年，一线实践者开始发现：光有好的上下文，Agent 依然会失控。

**2026 年 2 月：Harness Engineering**

技术播客 Vanishing Gradients 的一集节目标题直接点破了这个矛盾：「Why Agent Context Isn't Enough」（为何仅有 Agent 上下文依然不够）。节目揭示了一个关键悖论：上下文窗口的扩大，并不等于 Agent 性能的线性提升。即便模型理论上支持 100 万 Token 的上下文，性能衰减在 25.6 万 Token 左右便已出现。播客还记录了一起造成 5 万美元损失的事故：一个无人监控的 Agent 陷入无限循环，API 账单累积到被人发现时已经来不及了。

上下文可以告诉 Agent「知道什么」，但无法阻止 Agent「做不该做的事」。

Mitchell Hashimoto 在 2 月 5 日的博文中为这块缺失的拼图命了名：Engineer the Harness（工程化线束）。他的定义很简洁：

> 「每当你发现 Agent 犯了一个错误，你就花时间设计一个解决方案，使 Agent 永远不再犯同样的错误。」

六天后 OpenAI 官方的报告发布，业界的讨论也逐渐热了起来。

回过头看，三个阶段的关系用一句话就能说清：**Prompt Engineering 管的是****「****说什么****」****，Context Engineering 管的是****「****知道什么****」****，Harness Engineering 管的是****「****在什么环境里做事****」****。**

## **02**

## ******OpenAI 实验全解读：******

## ******不要把东西都塞进 AGENTS.md******

OpenAI 的这份报告是理解 Harness Engineering 的核心文本。里面的工程细节值得展开。

### 实验设定

团队从 3 名工程师起步，最终扩展至 7 名。五个月内构建并交付了一个内部测试版软件产品，已有外部 Alpha 测试用户。代码库覆盖应用逻辑、基础设施、工具链、文档和内部开发工具，全部由 Codex Agent 生成，无一行人类手动编写。

OpenAI 团队明确声明这是一个「刻意设定的极端约束实验」（forcing function）。他们写道，设定「零人类代码」这条规则的目的，是倒逼团队去构建能让 Agent 大规模可靠工作的工程基础设施。换句话说，这个约束本身就是为了催生 Harness。

效率数据也很突出：平均每名工程师每日 3.5 个 Pull Request 的合并吞吐量。代码审查通过 Agent 对 Agent 的循环实现了大规模自动化，人工监督仅保留在高层架构决策环节。

报告作者 Ryan Lopopolo 写了一句后来被反复引用的话：

> 「我们目前最困难的挑战，集中在设计环境、反馈回路和控制系统上。」

### 踩过的坑：AGENTS.md 的进化

报告中实操价值最高的部分，是团队在文档工程上的试错过程。

早期，团队犯了一个经典错误：把所有信息塞进一个庞大的 AGENTS.md 文件。系统说明、架构规范、代码风格、边界条件...... 全部堆在同一份文档里。结果 Agent 被信息淹没，性能反而下降。

他们最终演化出的方案是一个**渐进式披露模型**。AGENTS.md 被精简为约 100 行的「目录」角色，指向一个结构化的 docs/ 目录：

```
项目根目录/  AGENTS.md              ← 精简目录（~100 行），指向 docs/  AGENTS.override.md     ← 子目录级覆盖规则  docs/    ARCHITECTURE.md       ← 分层架构与依赖流向    DESIGN.md             ← 设计原则与模式    PLANS.md              ← 执行计划    PRODUCT_SENSE.md      ← 产品意图与用户旅程    QUALITY_SCORE.md      ← 质量评分标准    RELIABILITY.md        ← 可靠性要求    SECURITY.md           ← 安全约束
```

Codex 的发现机制是逐级读取：从全局配置 ~/.codex/AGENTS.md 到项目根目录，再到子目录，就近优先。比如 services/payments/ 下可以放一份 AGENTS.override.md，用 make test-payments 覆盖根目录的 npm test 规则。大小上限默认 32 KiB。

这套目录结构背后的核心假设是：Agent 不需要在一开始就知道所有事情，它需要在正确的时机获得正确粒度的信息。跟人类工程师入职的逻辑一样——没有人第一天就读完公司所有文档。

### 超越文档：让 Agent「看见」运行时

静态文档之外，OpenAI 团队做了一件更激进的事：把可观测性数据直接暴露给 Agent。

日志、指标、追踪信息，通过本地可观测性栈（每个工作树独立实例化）向 Codex Agent 开放。Agent 可以使用 LogQL 和 PromQL 查询来验证服务启动时间和关键用户旅程的性能指标。

更进一步，Agent 甚至可以通过 Chrome DevTools Protocol 操作浏览器：重现 Bug、验证修复、直接对 UI 行为进行推理。

这意味着 Agent 不再只是一个「写代码的工具」。它能看见代码运行后发生了什么，并据此判断自己写的代码到底对不对。

### 机械化的架构围栏

OpenAI 团队定义了严格的分层架构依赖流向：Types → Config → Repo → Service → Runtime → UI。任何违反依赖方向的代码都会被机械化拦截。

拦截机制有两种。一是确定性 Linter。有一个细节值得说：工程师花了数小时重写 Linter 的错误输出格式。目的只有一个，让 Agent 能「读懂」出了什么问题，并据此自动修复。Linter 输出的受众从人类变成了 AI——这件事本身就是 Harness Engineering 思维的典型体现。

二是基于 LLM 的审计 Agent，用于检查那些难以用形式化规则捕捉的语义违规。

两种机制组合，确保了 Agent 生成的代码在架构层面的长期一致性。团队的思路是：每当 Agent 犯一个新类型的错误，就回头加一条约束。日积月累，Harness 越来越健壮，Agent 能犯的错越来越少。

这正是 Hashimoto 所说的：「让 Agent 永远不再犯同样的错误。」

## **03**

## ******Böckeler 的解读：******

## ******Harness Engineering 的三级框架******

OpenAI 的报告是一手的工程记录，信息密度很高但组织偏松散。Thoughtworks 的 Distinguished Engineer、生成式 AI 交付专家 Birgitta Böckeler 在 martinfowler.com 上发表的分析文章，把这些实践提炼成了一个清晰的三维框架。

Martin Fowler 本人在 2 月 17 日的 Twitter 帖子中称赞了这篇报告：

> 「Harness Engineering 是对 AI 使能软件开发关键部分的有价值框架。Harness 包括上下文工程、架构约束和垃圾回收。」

Böckeler 将 Harness 的核心拆解为三个维度：

**维度一：上下文工程（Context Engineering）**

确保 Agent 在正确时机获得正确信息。包括前面提到的渐进式文档披露、动态可观测性数据接入，以及 Agent 对浏览器行为的直接推理能力。Böckeler 指出，这一维度与 2025 年中期流行的 Context Engineering 概念高度重合，但 Harness Engineering 将其纳入了一个更完整的体系。

**维度二：架构约束（Architectural Constraints）**

通过机械化手段强制执行架构边界。包括确定性 Linter（输出格式专为 Agent 设计）和 LLM 审计 Agent 的双轨机制。Böckeler 特别注意到，OpenAI 让 Linter 的错误消息直接包含修复建议，这使得整个「违规 → 检测 → 修复」的循环可以在 Agent 内部闭环完成，无需人工介入。

**维度三：熵管理 / 垃圾回收（Entropy Management）**

这是 Böckeler 框架中我觉得最有意思的部分。她观察到 OpenAI 团队部署了专用的清理 Agent，定期扫描文档漂移、模式违规和依赖问题。

为什么要单独拎出来？因为 Harness 本身也是代码和文档，它们同样会腐化。随着代码库规模增长，规则文件可能变得冗长混乱，包含过时、矛盾或不再适用的指令。如果 Harness 自身腐化了，Agent 就会因为读到混乱指令而输出混乱代码。熵管理要解决的就是这个问题：约束系统本身不能随时间退化。

Böckeler 把三者的关系概括得很清楚：上下文工程让 Agent「知道该做什么」，架构约束确保「只在边界内行事」，熵管理保障「整个系统不随时间退化」。

她同时提了一个重要的补充：OpenAI 的报告主要关注代码的内部质量和可维护性，但对功能性和行为验证的覆盖不足。能通过所有 Linter 和架构测试的代码，不等于做了用户真正需要的事情。这个提醒很实在，也指出了接下来需要补上的一块。

## **04**

## ******Stripe、LangChain，******

## ******行业有了更多实践者******

如果说 OpenAI 的实验只是个案，说服力有限。如今 Harness Engineering 的逻辑正在多个头部公司得到独立验证。

### Stripe：工业级的线束基础设施

Stripe 的 Minions 体系每周合并超过 1,300 个由 AI 完全编写的 Pull Request，人类仅负责审查。

Minions 的基础设施透露了 Harness Engineering 在大型组织中的实际形态：每个 Agent 任务在独立的预热 devbox 中运行，与 Stripe 工程师使用的机器完全相同，约 10 秒内启动，内置 Stripe 代码库和服务，与生产系统及互联网完全隔离。

工具访问通过名为 Toolshed 的中心化 MCP 服务器实现，托管近 500 个工具，涵盖内部系统和外部 SaaS 平台。Agent 与人类开发者享有完全一致的工具访问权限。

Stripe 的架构选择也有意思：确定性节点与 Agent 节点混合的「蓝图」模式。可预测的步骤（推送到 Git、运行 Linter、触发 CI）全部交给确定性代码处理，只在需要判断或创造力的环节才调用 LLM。这种设计把 LLM 限制在「可控盒子」里，大幅提升了系统的可预测性。

### LangChain：一个干净的对照实验

回到开头的那组数据。LangChain 的编码 Agent 在 Terminal Bench 2.0 上，通过仅优化 Harness 而不修改底层模型，得分从 52.8% 提升至 66.5%，排名从第 30 跃升至第 5。

这个案例的价值在于变量控制做得很干净：模型不变，Harness 变，结果剧变。在「环境比模型更重要」这个论点上，这可能是目前最直接的证据。

Anthropic 在内部工程文档中已经将 Claude Code 定位为「灵活的 Agent 线束」。Harness 的概念正在被工具供应商内化为产品设计思路。

MCP（模型控制协议）已在 Linux 基金会下的 Agentic AI 基金会治理，月 SDK 下载量超过 9,700 万，获 OpenAI、Google、Microsoft 和 AWS 采用。Stripe 的 Toolshed 就是一个 MCP 服务器。MCP 正在成为 Agent 工具访问的通用标准，而 Harness 工程的工具层将大规模迁移到这个协议上。

LangChain 的 State of Agent Engineering 报告提供了一组行业全景数据：89% 的受访者已为其 Agent 实施了可观测性，但仅有 52% 实施了评估（Evals）。大多数团队已经能「看见」Agent 在做什么，但还没有建立系统性的机制来判断「做得对不对」。评估体系怎么规模化，大概是 Harness Engineering 接下来一年绕不开的课题。

## **05**

## ******工程师的核心工作，******

## ******正从写代码转向设计环境******

一件事：**工程师的核心工作，正在从写代码转向设计让 Agent 可靠运行的环境。**

OpenAI 实验中的工程师，日常工作已经变成了三件事：

第一，构建文档与上下文体系。维护 AGENTS.md 目录、docs/ 下的架构规范与设计文档，编写自定义 Linter（包括重写 Linter 的错误消息格式，使其对 Agent 可读且包含修复建议），建立可观测性基础设施使 Agent 能够查询运行时数据。

第二，以机器可处理的方式定义业务意图。工程师需要把业务目标、质量标准和边界条件表达得足够清晰和精确，使 Agent 能够据此自主决策。这要求更强的系统性思维和抽象能力。

第三，构建自动化的防呆验证机制。合并门禁被最小化以避免瓶颈，系统转而依赖强大的自动化守卫。Stripe 的实践表明，预推送钩子和本地 Linter 在 5 秒内解决常见问题，是减少无效 Agent 循环的关键。

The Pragmatic Engineer 的创始人 Gergely Orosz 在报道 OpenClaw 创始人 Peter Steinberger 的工作方式时，描述了一个很生动的场景：Steinberger 是「在脑中保存项目高层结构的软件架构师」，在使用 Agent 时只讨论架构和重大决策，完全不涉及具体代码实现。

越来越多人开始觉得，这就是 Harness Engineering 对工程师的要求：系统理解的深度，比写代码的速度重要得多。

在组织层面，变化也很大。OpenAI 的 3-7 人团队完成了以前需要数十人规模的工程输出。Stripe 让单名工程师可以同时向多个 Agent 分配不同任务。团队结构正在向两三人甚至单人团队收敛，完整拥有从规划到上线的功能全生命周期。「合理团队规模」的底层计算逻辑正在被重写。

Böckeler 在这一点上提出了一个所有技术管理者都该想想的问题，她称之为「学徒缺口」（Apprentice Gap）：如果初级开发者过早进入 Agent 驱动循环，未经历手动开发的锻炼，他们可能缺乏未来构建健壮 Harness 所需的深度系统直觉。她建议将「体验工程」（Experience Engineering）视为下一个核心挑战，设计保留手动开发直觉的学习路径。

## **06**

## ******开发者可以做什么？******

Hashimoto 的六阶段采用旅程是目前操作性最强的个人路线图。他自己正处在第五阶段。以下是从他的博文和实践中提炼的行动建议：

**起步：把同一个任务做两遍。** 先自己手动完成，再让 Agent 重新做一遍。Hashimoto 说自己「真的把工作做了两遍」，目的是建立对 Agent 能力边界的直觉。他总结了三个关键发现：把会话拆成独立清晰的任务；把模糊需求拆成「规划」和「执行」两个阶段；给 Agent 自我验证的方法。

**养成习惯：每天下班前 30 分钟启动 Agent。** Hashimoto 说这「给了我第二天早晨一个热启动」。三类任务特别适合这个时段：深度调研（Agent 扫描整个领域）、并行探索（多个 Agent 同时试验模糊想法）、Issue 和 PR 分诊。

**关键跃迁：在你的项目里建一份 AGENTS.md。** 这不需要是一份完美的文档。从最基本的内容开始：项目的核心架构说明、常见的 Agent 错误及应对方式、测试和 Lint 命令、Agent 绝对不能碰的部分。每次 Agent 犯错，就回来补一条规则。日积月累，这份文档就会长成你的 Harness。

Hashimoto 还分享了一条心态层面的建议：「关掉 Agent 的桌面通知...... 作为人类，我的职责是控制何时中断 Agent，而非被它中断。」

对技术负责人来说，最实际的建议是：选一个新项目做试点。OpenAI 和 Stripe 的成功案例都有一个共同前提，要么从零开始，要么在成熟的内部基础设施上运行。遗留代码库的改造是另一个量级的工程挑战。此外，Evals（评估体系）是下一个必建能力。当前仅有 52% 的团队部署了评估系统，这个差距就是你的机会窗口。

OpenAI 的报告发布至今只有一个月。Hashimoto 自己也说，他还处在六阶段的第五阶段。行业里绝大多数团队还停留在前三个阶段。

但方向已经不可逆。

从 Prompt Engineering 到 Context Engineering 再到 Harness Engineering，三年间，开发者社区对「如何让 AI 可靠地工作」这个问题的理解，已经从「写好一条指令」演进到了「构建一整个运行环境」。

软件工程团队的核心竞争力，正在从「谁的工程师代码写得更好」转向「谁的工程师能设计出更好的 Agent 运行环境」。

正如 Ryan Lopopolo 在 OpenAI 报告中写的那句话：

> 「我们目前最困难的挑战，集中在设计环境、反馈回路和控制系统上。」

**更多阅读**

[OpenClaw 生态正在疯长，我们拆解了 PH 上 40 多款相关产品](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247523115&idx=1&sn=a152e8e6cdb7da36f7c66becbd56dbca&scene=21#wechat_redirect)

**[对话 Elys 创始人 Tristan：人的灵魂是所有 context 的总和，我们从未被真正连接过](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247523088&idx=1&sn=5ec0d3903638b28408683757552525a1&scene=21#wechat_redirect)**

[一百个 OpenClaw 产品涌来，我们最近推荐这几款](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247522839&idx=1&sn=a522275172e405f388026842ee9de0d8&scene=21#wechat_redirect)

[在 OpenClaw 的冲击下，Cursor 已经要过时了](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247523014&idx=1&sn=c919f5c144d841916f4b55d2fe4a595a&scene=21#wechat_redirect)

[The Information：OpenClaw 在中国AI圈的发酵和扩散速度，远超硅谷想象](https://mp.weixin.qq.com/s?__biz=Mzg5NTc0MjgwMw==&mid=2247523045&idx=1&sn=b9c1ce3d73961864b7d51eb1e9142151&scene=21#wechat_redirect)

转载原创文章请添加微信：founderparker
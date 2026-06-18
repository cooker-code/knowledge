> 已吸收至：[[02_Agent与AI工程/0209_Harness Engineering/0209_核心知识点/运行环境与沙箱底座|运行环境与沙箱底座]]
---
title: 重磅！Anthropic官方Harness发布了！
author: Datawhale
date:
url: https://mp.weixin.qq.com/s?__biz=MzIyNjM2MzQyNg==&mid=2247721702&idx=1&sn=2402b415be2a1d7b153a777ac284f70c&chksm=e9941b03d0c32d6f459e8f64112a8becaaeb4317dd4c0880be1e7a0f7b48f595cffc672bcec6&mpshare=1&scene=24&srcid=0411YA9l2DUodhh7UOoxS8tc&sharer_shareinfo=b3853e14f512570e60e6b1663a72847c&sharer_shareinfo_first=b3853e14f512570e60e6b1663a72847c#rd
---

# Datawhale干货  **************编辑：Datawhale团队**************

2026 年初，OpenAI 和 Anthropic 几乎同时发布了关于 Harness 的技术实践文章，LangChain 工程师 Viv 给出了一个简洁的公式来概括这个理念：Agent = Model + Harness。模型提供智能，Harness 让这个智能能真正投入生产。但问题是，搭建一套生产级的 Harness 需要处理沙箱执行、状态管理、权限控制、端到端追踪——这些基础设施工作可能耗费数月。

昨天，Anthropic 把 Harness 从概念变成了产品！正式发布 Claude Managed Agents，一套用于构建和部署云托管 Agent 的可组合 API 套件。这是一个完整的托管服务：你定义 Agent 的任务、工具和护栏，Anthropic 的基础设施负责运行，内置的 Harness 处理所有编排逻辑。Vibecode 的开发速度因此提升了 10 倍，Sentry 从构思到交付只用了数周。

官方Harness下场了：Claude Managed Agents

构建一个能真正投入生产的 Agent，需要的远不止调用几次 API。你需要沙箱化的代码执行环境、检查点机制、凭证管理、作用域权限控制、完整的执行追踪——这是数月的基础设施工作，而用户还看不到任何可见的功能。

Claude Managed Agents 把这些复杂性全部接管。开发者只需定义 Agent 的任务、工具和护栏，Anthropic 的基础设施负责运行。内置的编排 harness 会决定何时调用工具、如何管理上下文、如何从错误中恢复。

这个产品包含四个核心能力：

生产级 Agent：安全沙箱、身份验证、工具执行全部由平台处理。开发者不需要自己搭建这些基础设施。

长运行会话：Agent 可以自主工作数小时，进度和输出会持久化保存，即使连接断开也不会丢失状态。Agent 可以接手真正复杂的长期任务，远超几分钟的简单请求。

多 Agent 协调：Agent 可以生成并指挥其他 Agent，将复杂工作并行化处理。一个主 Agent 可以派生多个子 Agent 分别处理不同子任务，然后汇总结果。

可信治理：Agent 访问真实系统时，作用域权限、身份管理、执行追踪都已内置。企业部署 Agent 时最担心的安全问题——Agent 越权访问敏感数据或系统——在这里有了系统性的解决方案。

Claude Managed Agents 的整体架构

在 Anthropic 的内部测试中，针对结构化文件生成任务，Managed Agents 相比标准提示循环，任务成功率提升了最多 10 个百分点，而且在最困难的问题上提升幅度最大。

Agent = Model + Harness，Anthropic 把公式变成了产品

Claude Managed Agents 的底层逻辑来自 Harness Engineering 理念。LangChain 工程师 Vivek Trivedy 曾给出一个简洁的公式：Agent = Model + Harness。模型提供智能，Harness 让这个智能可以被实际使用。

Harness 包括系统提示词、工具、文件系统、沙箱、编排逻辑、各种检查机制。本质上回答三个问题：AI 在哪里工作？用什么工作？如何验证工作质量？

Anthropic 在设计 Managed Agents 时，遵循了三个关键模式，这些模式直接影响了产品的架构选择：

模式一：使用 Claude 已知的工具

Claude 在 2024 年末的 SWE-bench Verified 基准测试中达到 49% 的成绩，当时的最先进水平。使用的工具只有两个：bash 工具和文本编辑器工具。bash 并非为构建 Agent 设计，但它是 Claude 深度理解并且会随着模型迭代而不断改进使用能力的工具。

Claude 在 SWE-bench Verified 基准测试上的成绩演进

Anthropic 发现 Claude 能够将这些通用工具组合成解决不同问题的模式。Agent Skills、编程式工具调用、内存工具，都是基于 bash 和文本编辑器工具构建的。

编程式工具调用、skills 和内存工具都是由 bash 和文本编辑器工具组合而成。

这种设计哲学的核心是：提供 Claude 已经精通的通用工具，让它自己组合出解决方案，而非为每个特定任务设计专用工具。

模式二：让 Claude 自主决策

Agent harness 会编码一些关于“Claude 不能自己做什么”的假设。随着 Claude 能力提升，这些假设需要被重新检验。

一个常见假设是：每个工具调用的结果都必须通过 Claude 的上下文窗口，才能决定下一步行动。但处理工具结果需要消耗 token，既慢又贵，而且如果结果只是要传递给下一个工具，或者 Claude 只关心输出的一小部分，这种处理就是不必要的。

传统模式下，Claude 调用工具，每个工具结果都要经过上下文窗口处理。

给 Claude 一个代码执行工具（如 bash 工具）可以解决这个问题：Claude 可以编写代码来表达工具调用以及它们之间的逻辑。Claude 决定哪些结果需要处理、哪些可以过滤、哪些可以直接管道传输到下一个调用，而不触及上下文窗口。只有代码执行的最终输出才会进入 Claude 的上下文。

通过代码执行，Claude 可以自主编排工具调用逻辑，只有最终输出进入上下文窗口。

编排决策从 harness 转移到模型本身。由于代码是 Claude 编排行动的通用方式，强大的编码模型也是强大的通用 Agent。Claude 在非编码评估中也展现出强劲表现：在 BrowseComp 基准测试（测试 Agent 浏览网页的能力）中，给 Opus 4.6 提供过滤自己工具输出的能力，准确率从 45.3% 提升到 61.6%。

另一个常见假设是：系统提示词应该手工编写任务特定的指令。问题是预加载所有指令无法跨多任务扩展——每增加一个 token 都会消耗 Claude 的注意力预算，而且预加载很少使用的指令是浪费的。

Claude 通过 skills 实现上下文的渐进式展开，只在需要时加载完整内容。

给 Claude 访问 skills 的能力可以解决这个问题：每个 skill 的 YAML 前言是预加载到上下文窗口的简短描述，提供 skill 内容概览。如果任务需要，Claude 可以通过调用读取文件工具来逐步展开完整的 skill。

模式三：谨慎设置边界

Agent harness 在 Claude 周围提供结构，以强制执行用户体验、成本或安全边界。

Claude 本身不一定知道应用的安全边界或用户体验表面。Claude 发出工具调用，由 harness 处理。bash 工具给 Claude 广泛的编程杠杆来执行操作，但它只给 harness 一个命令字符串——每个操作的形状都相同。将操作提升为专用工具，可以给 harness 一个带类型参数的特定操作钩子，可以拦截、控制、渲染或审计。

需要安全边界的操作是专用工具的天然候选。可逆性通常是一个好标准，难以逆转的操作（如外部 API 调用）可以通过用户确认来控制。像编辑这样的写入工具可以包含过期检查，这样 Claude 就不会覆盖自上次读取以来已更改的文件。

专用工具可以基于安全、用户体验或可观测性需求设置边界。

工具在需要向用户呈现操作时也很有用。例如，它们可以渲染为模态框，向用户清晰显示问题、提供多个选项，或阻塞 Agent 循环直到用户提供反馈。

专用工具的决策应该持续重新评估。例如，Claude Code 的自动模式（发布时处于研究模式）在 bash 工具周围提供了安全边界：它让第二个 Claude 读取命令字符串并判断是否安全。这种模式可以限制对专用工具的需求，但应该只用于用户信任总体方向的任务。

真实案例：Vibecode 速度提升 10 倍

多个团队已经在使用 Managed Agents 交付生产应用，覆盖编码、生产力、文档处理等场景。

Notion 将 Claude 直接集成到工作空间中，让团队可以委托工作给 Claude（目前在 Notion Custom Agents 中处于私有 alpha 阶段）。工程师用它来交付代码，知识工作者用它来生成网站和演示文稿。数十个任务可以并行运行，整个团队协作处理输出。产品经理 Eric Liu 表示，Managed Agents 能够处理长运行会话、管理内存、长期交付高质量输出，这让 Notion 成为团队与 Agent 协作完成工作的最佳场所。

Sentry 将他们的调试 Agent Seer 与 Claude 驱动的 Agent 配对，后者负责编写补丁并打开 PR。开发者从标记的 bug 到可审查的修复，整个流程一气呵成。工程总监 Indragie Karunaratne 指出，Managed Agents 提供了安全、完全托管的 Agent 运行时，让他们能够专注于构建无缝的开发者体验。这个集成在 Managed Agents 上只用了数周完成，并且消除了维护定制 Agent 基础设施的持续运营开销。

Asana 构建了 AI Teammates，这些协作 AI Agent 在 Asana 项目中与人类并肩工作，接手任务并起草交付物。CTO Amritansh Raghav 表示，Managed Agents 显著加速了他们的开发，帮助他们更快交付高级功能，让团队能够专注于创建企业级的多人用户体验。

Rakuten 在产品、销售、营销和财务部门部署了企业 Agent，这些 Agent 接入 Slack 和 Teams，让员工可以分配任务并获得电子表格、幻灯片和应用等交付物。每个专业 Agent 都在一周内完成部署。AI 业务总经理 Yusuke Kaji 表示，随着 Agent 能力增强，Managed Agents 让他们能够安全扩展，而无需自己构建 Agent 基础设施。

Vibecode 帮助客户从提示词到部署应用，使用 Managed Agents 作为默认集成。联合创始人 Ansh Nanda 指出，在 Managed Agents 之前，用户必须手动在沙箱中运行 LLM、管理生命周期、配备适当工具并监督执行，这个过程可能需要数周或数月来设置。现在只需几行代码，用户就可以至少快 10 倍地启动相同的基础设施。

## 写在最后：这是一个值得关注的方向

Claude Managed Agents 的推出，改变了 Anthropic 的商业定位。过去他们提供的是模型 API，开发者按 token 付费。现在他们提供的是完整的 Agent 运行环境：从沙箱、会话管理到权限控制，全部托管。Anthropic 开始做云服务商做的事——提供计算资源和运行环境，只不过跑在上面的是智能 Agent 而不是普通应用。

定价上也能看出这个转变。除了标准的 token 费用，Managed Agents 按会话活跃时间收费：每小时 0.08 美元。这是一种更接近基础设施服务的计费方式，类似于云主机按运行时长计费。

从技术角度看，Anthropic 在 Harness 设计上留了很大的灵活性。他们没有强制使用某一套固定的编排逻辑，而是提供了一个可以容纳不同控制器的系统。Claude Code 可以跑在上面，针对特定任务优化的控制器也可以跑在上面。这种设计让产品能够随着模型能力提升而演进，而不需要推倒重来。

对开发者来说，这个产品最大的价值是省掉了从原型到生产之间的那段基础设施工作。Notion、Sentry、Asana 这些案例都证明了一点：当基础设施不再是瓶颈，团队可以把精力放在真正重要的地方——设计 Agent 能做什么、怎么做、边界在哪里。

这是一个值得关注的方向。

参考地址：

1.https://claude.com/blog/claude-managed-agents

2.[Harness Engineering在硅谷爆火，一文带你搞懂！](https://mp.weixin.qq.com/s?__biz=MzIyNjM2MzQyNg==&mid=2247721377&idx=1&sn=b7f8599e10484b123984a76674108e65&scene=21#wechat_redirect)

**一起“**点****赞”****三连**↓**
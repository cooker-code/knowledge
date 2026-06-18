# Agent 类型化评估与 pass 指标

## 原文锚点

- 本地文件：[Anthropic官方首发：Agent评估体系构建详解](../文章/done-Anthropic官方首发：Agent评估体系构建详解.md)
- 原文链接：https://mp.weixin.qq.com/s?__biz=MzE5ODM0MTIyNQ==&mid=2247483839&idx=1&sn=8f81b38fae12808e900256d5c45f5b8c
- 官方锚点：[Anthropic - Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- 关键段落：coding/research/conversational/computer use agents、code/model/human graders、capability vs regression evals、pass@k、pass^k。
- 关键图：无技术图。

## 图片处理

| 图片 | 类型 | 是否保留 | 理由 | 处理方式 |
|---|---|---|---|---|
| 无 | 无图 | 不适用 | 评估框架可用表格表达 | 不需要重建 |

## 一句话结论

这篇文章值得精读：它把 Agent 评估从“统一打分”校准为“按 Agent 类型选择评分器，并区分潜力和稳定性”。

## 用户相关性判断

| 项 | 内容 |
|---|---|
| 用户当前认知层级 | Agent/AI 应用评估与观测 L1-L2 draft |
| 认知成熟度 | draft |
| 阅读投入建议 | 精读 |
| 阅读投入理由 | 直接服务当前文章抽取 Agent 的评估设计；公众号是二手解读，关键以官方文为准 |
| 对用户的新信息 | Coding、Research、Conversation、Computer Use Agent 的验收对象不同，不能用同一套指标 |
| 问题指纹 | Agent 评估 + Agent 类型 + code/model/human graders + capability/regression + pass@k/pass^k + 评估设计边界 |
| 排重判断 | 新建 |
| 置信度 | 高 |

## 认知校准点

| 校准点 | 文章观点/信息 | 与用户认知或价值观的关系 | 处理建议 |
|---|---|---|---|
| Agent 评估要先分类型 | Coding、Research、Conversation、Computer Use 的成功证据不同 | 纠偏：不能用统一主观评分 | 写入 Agent 评估 index |
| 评分器要组合 | code-based、model-based、human 各有适用范围 | 补评估架构 | 与 Langfuse/RAGAS 关联 |
| capability 和 regression 不同 | 前者看能力爬坡，后者防退化 | 补质量门禁边界 | 用于本地流程 |
| pass@k 和 pass^k 表达不同 | pass@k 看多次尝试至少一次成功，pass^k 看多次都稳定成功 | 补可靠性判断 | 记住 |
| Browser Agent 要评估工具选择 | DOM 与 screenshot 在 token/延迟上有不同取舍 | 关联 Chrome DevTools MCP | 后续做工具评估 |

## 冲突点

| 冲突类型 | 具体表现 | 影响 | 处理 |
|---|---|---|---|
| 二手转述 | 公众号摘译 Anthropic 官方文 | 细节可能偏差 | 官方文为锚点 |
| 术语混杂 | τ-Bench、BrowseComp、WebArena、OSWorld 都是基准名 | 容易堆名词 | 只保留评估思想 |
| 实践不完整 | 没有本地评估集和执行脚本 | 不能直接判实践 | 降为精读 |

## 待吸收点

| 分级 | 内容 | 为什么值得吸收 | 后续动作 |
|---|---|---|---|
| 理解 | Coding Agent 可用确定性测试、静态分析、工具调用验证 | 适合 Codex/Claude Code 类任务 | 用于本地流程 |
| 理解 | Research Agent 要看来源覆盖、引用质量和事实支持 | 适合文章抽取/研究任务 | 写入评估计划 |
| 理解 | Computer Use Agent 要验证 UI 状态和后端状态 | 防止只看截图成功 | 与浏览器 MCP 对标 |
| 记住 | Capability eval 低通过率可接受，Regression eval 应接近稳定通过 | 影响测试集设计 | 用于质量门禁 |
| 记住 | `pass@k` 看潜力，`pass^k` 看稳定性 | 判断 Agent 是否可靠 | 写入 index |
| 实践 | 为文章抽取 Agent 设计 code/model/human 三类评分器 | 可落地 | 待实验 |

## 已知可跳过

| 内容 | 跳过理由 |
|---|---|
| 各基准名称清单 | 了解即可，不能替代本地评估 |
| 大段 YAML 示例 | 结构可借鉴，不直接照搬 |
| 项目仓库推广 | 无核心价值 |

## 实践门槛

| 门槛 | 判断 | 证据 |
|---|---|---|
| 可运行 | 否 | 无本地评估脚本 |
| 可验证 | 部分 | 有评分器结构和指标定义 |
| 可排障 | 部分 | 可通过 transcript/tool calls 定位 |
| 可迁移 | 是 | 可迁移到文章抽取 Agent |
| 结论 | 降为精读 | 后续需要本地小评估集 |

## 归类判断

| 项 | 内容 |
|---|---|
| 技术本体 | Agent 评估方法 |
| 文章主问题 | 不同类型 Agent 应如何设计评估 |
| 使用场景 | 编码、研究、对话、浏览器/计算机使用 Agent |
| 关键词干扰 | Anthropic、基准、上下文工程 |
| 最终归类 | Agent 与 AI 工程 / 评估与观测 / Agent 评估 |
| 归类理由 | 主问题是 Agent 质量评估，不是模型评测或 RAG 检索 |

## 技术定位

| 项 | 内容 |
|---|---|
| 技术类型 | 评估方法 |
| 所属领域 | Agent 与 AI 工程 |
| 二级类目 | 评估与观测 |
| 全局架构位置 | Agent 运行和迭代流程旁路 |
| 涉及模块 | Grader、Dataset、Transcript、Tool Call、State Check、Regression |
| 解决问题 | 根据 Agent 类型选择可验证的评分器和稳定性指标 |
| 原文局限 | 未落到本地业务评估集 |
| 我的结论 | 精读，服务本地 Agent 流程评估 |

## 纵向理解

| 维度 | 判断 |
|---|---|
| 全局架构 | 任务集 -> Agent 执行 -> transcript/outcome/tool calls -> grader -> capability/regression 报告 |
| 本文位置 | 讲评估设计框架，不讲观测平台实现 |
| 核心机制 | 类型化任务、组合评分器、能力/回归分离、pass 指标 |
| 使用链路 | 先定义 Agent 类型 -> 明确最终状态 -> 选择评分器 -> 采集过程指标 -> 形成门禁 |
| 前置条件 | 有固定任务集、可验证结果、人工校准样本 |
| 边界 | LLM-as-judge 仍需人工校准，pass 指标不能替代失败分析 |

## 横向对标

| Agent 类型 | 主要验证对象 | 推荐评分器 | 风险 | 本地启发 |
|---|---|---|---|---|
| Coding Agent | 测试、类型、安全、代码质量 | deterministic tests、static analysis、tool calls | 只看测试会漏过程问题 | Codex/Claude Code 任务 |
| Research Agent | 来源、覆盖、事实支持 | citation check、rubric、human review | 事实随时间变化 | 文章抽取/调研 |
| Conversational Agent | 最终状态和交互质量 | state check、LLM rubric、human | 语气和完成度都重要 | 客服/问答 |
| Computer Use Agent | UI 操作和后端状态 | screenshot/DOM/state check | 只看页面会误判 | Browser MCP |

## 后续追查

- 关键词：Agent eval、code-based graders、model-based graders、human graders、capability eval、regression eval、pass@k、pass^k。
- 相关技术：Langfuse、RAGAS、Agent 评估、Chrome DevTools MCP、Browser Agent。
- 需要补读的文章：Anthropic 官方原文、LangSmith eval、OpenAI evals、WebArena/OSWorld 任务验证方式。

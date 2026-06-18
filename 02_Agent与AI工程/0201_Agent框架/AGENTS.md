# Agent 框架
## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](020100_核心知识点/知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `020100_核心知识点/`。
- 本轮根 `文章/` 拆分明细见：[2026-06-17 文章目录拆分路由](2026-06-17_文章目录拆分路由.md)。


## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | Agent 与 AI 工程 |
| 二级类目 | Agent 框架 |
| 核心问题 | 如何把模型、状态、工具、流程控制、记忆和评估组织成可执行的 Agent 应用 |
| 不解决什么 | 不直接讨论模型训练、模型能力榜单、单纯提示词技巧或 IDE 使用体验 |
| 用户当前认知假设 | L2：知道 Agent 框架概念，但需要补状态管理、流程控制、回滚、观测和质量门禁 |

## 用户认知重点

| 认知项 | 当前假设 | 后续整理策略 |
|---|---|---|
| 已知基础 | Agent 可以调用工具、执行多步骤任务 | 不重复解释 Agent 是什么 |
| 待补边界 | 框架、工作流、工具调用、上下文工程和评估之间的边界 | 每篇文章定位到控制流、状态、工具、评估或部署模块 |
| 易偏差点 | 框架文章常用“快多少倍”“最优解”包装 | 降权无基线性能数字，优先抽取可控性和失败边界 |

## 排重准则

问题指纹：

```text
Agent 框架 + 控制流/状态/工具/记忆/评估模块 + 核心机制 + 解决问题 + 可控性边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 LangGraph | 按流程控制、状态、子图、持久化、观测、部署拆分 |
| 都是框架对比 | 没有同一问题域的评价指标则降权 |
| 都是教程 | 只讲 API 入门不新建，能补设计准则才沉淀 |
| 有失败处理和回滚 | 优先进入核心知识点 |

## 已覆盖技术

| 技术 | index | 已覆盖问题 | 还缺什么 |
|---|---|---|---|
| LangGraph | [LangGraph](020102_LangGraph/AGENTS.md) | 流程控制模式、子图与并行状态合并、记忆与反馈循环 | 官方持久化、人机中断、观测、部署和本地最小实验 |
| OpenAI Agents SDK | [OpenAI Agents SDK](<020103_OpenAI Agents SDK/AGENTS.md>) | Harness/Compute 分离、沙箱执行、双层记忆、快照恢复 | 官网/GitHub/官方文档补证、最小沙箱实验 |
| DeerFlow | [DeerFlow](020101_DeerFlow/AGENTS.md) | 研究型多智能体编排、计划修订、中间件管道、harness/app 边界 | 官方仓库补证、最小研究任务实验、成本和恢复证据 |
| Vercel | [Vercel](020105_Vercel/AGENTS.md) | AI SDK 多 Provider 抽象、工具调用、Open Agents、Skills 分发和浏览器执行工具 | 官方版本补证、add-skill 安全验证、Open Agents 部署实验 |
| Pi | [Pi](020104_Pi/AGENTS.md) | 终端 coding agent runtime、四工具执行面、provider 抽象、事件流、turn/session 边界 | 官方仓库/包名补证、最小 SDK 事件流实验、安全和恢复验证 |
| 多 Agent 协作 | [多Agent协作](020106_多Agent协作/AGENTS.md) | Supervisor 控制边界、失败降级、结果合并 | AutoGen/CrewAI/LangGraph 多 Agent 官方对标 |
| DeepAgents | [DeepAgents](020107_DeepAgents/AGENTS.md) | 文件系统、子代理、任务拆分和深度执行链路 | 核心包与 CLI 能力边界补证、子代理路由实验 |
| PydanticAI | [PydanticAI](020108_PydanticAI/AGENTS.md) | 类型安全、依赖注入、结构化输出和重试 | 官方最小样例、错误恢复和生产边界 |
| Agno | [Agno](020109_Agno/AGENTS.md) | 多模态 Agent 框架定位和性能宣传降权 | 官方版本、基线和本地最小实验 |
| Qwen-Agent | [Qwen-Agent](020110_Qwen-Agent/AGENTS.md) | Qwen 生态 Agent 构建、工具调用和规划 | 官方能力边界、工具 schema 和运行实验 |
| LangChain | [LangChain](020112_LangChain/AGENTS.md) | Agent 抽象、Messages、OutputParser、部署和与 LangGraph 边界 | v1.0 官方补证、生产监控和迁移边界 |
| Agent 协议 | [Agent协议](020113_Agent协议/AGENTS.md) | A2A、ACP、AgentRun 等发现、连接和接线问题 | 标准采用情况、安全子模块和互操作实验 |
| 长任务 Agent 运行时 | [长任务Agent运行时](020114_长任务Agent运行时/AGENTS.md) | 目标状态、Judge 闭环、暂停恢复、用户输入优先 | 官方 goal/AskUserQuestion 补证、本地长任务实验 |
| Agent 设计模式 | [Agent设计模式](020115_Agent设计模式/AGENTS.md) | 规划、反思、路由、并行、HITL 和任务拆解 | 失败边界、模式选择准则和反例 |
| 企业级 Agent 平台 | [企业级Agent平台](020116_企业级Agent平台/AGENTS.md) | 生产化平台、稳定性、意图识别、部署和治理 | 指标、基线、权限、运维和验收证据 |
| Data Agent | [DataAgent](020117_DataAgent/AGENTS.md) | 数据分析、Text2SQL、数据工程和告警根因类 Agent | 与 BI、语义层、数据工程目录的边界 |
| 研究型 Agent | [研究型Agent](020118_研究型Agent/AGENTS.md) | 研究、调研、趋势分析和验算类任务链路 | 证据链、引用质量和可验证输出 |
| 交付型 Agent | [交付型Agent](020119_交付型Agent/AGENTS.md) | PPT、文档、图表、CAD、视频和前端交付物 | 可编辑、可验证、可回滚和交付验收 |
| Agent 工程方法 | [Agent工程方法](020120_Agent工程方法/AGENTS.md) | 学习路线、架构方法、项目清单和高层落地经验 | 从资讯中提炼可复用准则的证据 |
| Agent 评估 | [评估与观测 / Agent评估](../0204_评估与观测/020402_Agent评估/AGENTS.md) | 指标、Trace、Dataset、CI 质量门禁的基础闭环 | 长任务评估、工具调用错误分类、人工反馈标注规范 |

## 待补技术和问题

| 技术/问题 | 为什么要补 | 优先级 |
|---|---|---|
| LangGraph 官方持久化与 Human-in-the-loop | 本轮仅用本地文章，官方能力和版本边界未补证 | 高 |
| OpenAI Agents SDK 沙箱最小实验 | 需要验证凭证隔离、Manifest 和快照恢复是否可复现 | 高 |
| 长任务 Agent 运行时本地实验 | 需要验证目标状态、Judge、暂停恢复和用户输入优先级 | 高 |
| DeerFlow 最小研究任务实验 | 需要验证计划轮次、搜索轮次、Token 成本、失败恢复和产物验收 | 高 |
| Vercel AI SDK / add-skill / Open Agents 补证 | 需要核对官方版本、安装路径、安全回滚、workflow/sandbox 分离和部署复杂度 | 高 |
| Pi 最小 SDK / 事件流实验 | 需要核对官方包名、仓库状态、事件 schema、session 恢复、fork 和工具权限边界 | 高 |
| 多 Agent 协作评估集 | 需要制造失败、冲突和缺失，验证 Writer 合并规则 | 中 |
| DeepAgents 子代理路由实验 | 需要验证主 Agent 是否稳定委派，工具权限是否隔离 | 中 |

<!-- AUTO:SECONDARY_INIT_START -->
## 全量文章来源初始化

> 自动生成。初始化阶段只使用本地 `本地文章目录`、已有 `knowledge` 和本地 `wiki`，不联网补官网或外部证据。

- 全量文章来源：已拆分到各三级节点的 `文章/`；低置信临时原文不再保留 `99` 目录，只在路由表中保留判断。
- 全局明细：`scripts/output/knowledge-secondary-pools.json`

| 指标 | 数量 |
|---|---:|
| 文章数 | 315 |
| 正式沉淀原文数 | 9 |
| 已引用锚点 | 281 |
| 核心知识点数 | 7 |
| 精读候选 | 152 |
| 略读 | 149 |
| 跳过 | 5 |
| 低置信 | 0 |
| 原图缺失 | 88 |

### 主题簇

| 技术/主题 | 文章数 | 正式沉淀 | 精读候选 | 原图缺失 | 处理决策 | 认知校准点 |
|---|---:|---:|---:|---:|---|---|
| Agent 工程 | 305 | 5 | 146 | 82 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |
| LangGraph | 10 | 4 | 6 | 6 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |

### 精读候选

| 技术对象 | 原文 | 冲突点 | 处理建议 |
|---|---|---|---|
| Agent 工程 | [【Agent专题】从新手到高手：掌握这6种Multi-Agent设计模式！](020106_多Agent协作/文章/【Agent专题】从新手到高手：掌握这6种Multi-Agent设计模式！.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [【LangChain】生产级Agent部署与监控](020112_LangChain/文章/【LangChain】生产级Agent部署与监控.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [【智造】AI应用实战：6个agent搞定复杂指令和工具膨胀](020106_多Agent协作/文章/【智造】AI应用实战：6个agent搞定复杂指令和工具膨胀.md) | 正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [03-复杂需求拆分：大 PRD 不能直接丢给 Agent](<../0212_COT&TOT&REACT/文章/03-复杂需求拆分：大 PRD 不能直接丢给 Agent.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [100天掌握大语言模型第13周：LLM 智能体 · 工具使用 · LangChain · LlamaIndex · 多智能体系统](<020112_LangChain/文章/100天掌握大语言模型第13周：LLM 智能体 · 工具使用 · LangChain · LlamaIndex · 多智.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [7.9k的开源 AI 智能体 MiroThinker：一个会研究、会验算的 AI 研究员](<020118_研究型Agent/文章/7.9k的开源 AI 智能体 MiroThinker：一个会研究、会验算的 AI 研究员.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent skills：AI 能力扩展的新范式](<../0202_工具调用/020203_Skill/文章/Agent skills：AI 能力扩展的新范式.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | Agent 下一阶段的关键要素：可解释、造工具和 100% 确认美学｜对话 Sheet0.com 创始人王文锋 | 标题/观点需要降权；正文提到技术图但 Markdown 无图 | 低置信原文已删除；后续如重新进入，必须先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent 可靠性的工程解法：从 Skillify 看持续改进机制](<../0204_评估与观测/020402_Agent评估/文章/Agent 可靠性的工程解法：从 Skillify 看持续改进机制.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent 怎么拆解任务](<../0212_COT&TOT&REACT/文章/Agent 怎么拆解任务.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent 架构综述：从 Prompt 到 Context](<../0208_Context Engineering/文章/Agent 架构综述：从 Prompt 到 Context.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agentic 设计模式 - 路由模式](<020115_Agent设计模式/文章/Agentic 设计模式 - 路由模式.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agentic21种设计模式6-Planning](../0212_COT&TOT&REACT/文章/Agentic21种设计模式6-Planning.md) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent效率X10，ClaudeCode/Codex 专属的文件管理Agent：Agentfile](<../0205_AI编程工具/020501_AI Coding 工作流/文章/Agent效率X10，ClaudeCode_Codex 专属的文件管理Agent：Agentfile.md>) | 原目录与最终归类不一致；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 工程 | [Agent测试方法论：LLM-as-Judge，用 AI 测 AI 到底靠不靠谱？](<../0204_评估与观测/020402_Agent评估/文章/Agent测试方法论：LLM-as-Judge，用 AI 测 AI 到底靠不靠谱？.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |

### 冲突与缺口

- 冲突分布：原目录与最终归类不一致(179)、正文提到技术图但 Markdown 无图(88)、标题/观点需要降权(6)
- 存在原图缺失，精修时只补架构图、流程图、说明图和对比图。
- 精读候选不直接写笔记，先和已有问题指纹对比。
- 初始化阶段不补外部官网/GitHub；需要官方证据时在后续精修阶段补证。

### 下一步

| 优先级 | 动作 |
|---|---|
| P0 | 先处理精读候选，按主题簇合并，不逐篇扩写 |
| P1 | 对已有核心知识点补充排重依据和认知校准点 |
| P2 | 精修阶段再补官网、GitHub、版本状态和官方架构图 |
<!-- AUTO:SECONDARY_INIT_END -->

## 本轮并行精读沉淀记录

> 手工维护。只使用本地文章，不联网补证；官网/GitHub/官方文档字段在新建技术目录中统一标为“后续补证”。

| 指标 | 数量 |
|---|---:|
| 处理文章数 | 8 |
| 正式沉淀核心知识点 | 6 |
| 新增技术/模式目录 | 4 |
| 更新技术 index | 1 |

### 正式沉淀

| 技术/模式 | 核心知识点 | 原文数 | 处理决策 | 主要认知校准点 |
|---|---|---:|---|---|
| LangGraph | [LangGraph子图与并行状态合并](020102_LangGraph/020102_核心知识点/LangGraph子图与并行状态合并.md) | 1 | 新建 | 子图价值在阶段内并行、汇聚检查和状态合并，不是嵌套越多越好 |
| LangGraph | [LangGraph记忆与反馈循环](020102_LangGraph/020102_核心知识点/LangGraph记忆与反馈循环.md) | 2 | 合并新建 | 长期记忆不是聊天历史堆叠，而是明确反馈触发的受控规则更新 |
| OpenAI Agents SDK | [OpenAI Agents SDK沙箱执行与记忆控制](<020103_OpenAI Agents SDK/020103_核心知识点/OpenAI Agents SDK沙箱执行与记忆控制.md>) | 1 | 新建 | Agent 框架竞争点从模型调用转向控制面、执行面、记忆和恢复 |
| DeepAgents | [知识地图](020107_DeepAgents/020107_核心知识点/知识地图.md) | 1 | 待重建核心点 | Skill 的关键是菜单化、按需读取和专业子代理隔离；旧核心文件当前不在磁盘上 |
| 多 Agent 协作 | [多Agent协作的Supervisor控制边界](020106_多Agent协作/020106_核心知识点/多Agent协作的Supervisor控制边界.md) | 1 | 新建 | 多 Agent 不是多个 LLM 调用并排，而是决策权、失败隔离和合并规则 |
| 长任务 Agent 运行时 | [知识地图](020114_长任务Agent运行时/020114_核心知识点/知识地图.md) | 2 | 待重建核心点 | 长任务核心是外部目标状态、保守 Judge、暂停态和用户输入优先；旧核心文件当前不在磁盘上 |

### 合并与跳过

| 原文 | 处理 | 原因 |
|---|---|---|
| `Python+LangChain/LangGraph框架开发Ai智能体系列(九)` | 合并到 LangGraph 记忆与反馈循环 | 原文只有目录式摘要，单独沉淀价值不足 |
| `Claude Agent SDK 中的交互式问答：AskUserQuestion 暂停/恢复机制详解` | 合并到长任务 Agent 运行时 | 主价值是暂停/恢复边界，不单独作为 Claude SDK 技术目录 |
| PDF 工具实现细节 | 跳过 | 属于示例工具，不是 Agent 框架机制 |
| AI 编程工具、MCP、RAG 相关文章 | 本轮未写入 | 与 Agent 框架只做横向边界，避免误归类 |

### 主要冲突点

| 冲突点 | 表现 | 处理 |
|---|---|---|
| 原目录冲突 | 多篇文章原在 `01_LLM与大模型` 或 `raws/claude-code` | 按技术本体重路由到 Agent 框架 |
| 图片缺失 | 多篇正文提到架构图/流程图但 Markdown 无图 | 只对关键机制用 Mermaid 重建 |
| 证据不足 | OpenAI Agents SDK、DeepAgents、Hermes、Claude SDK 的官方能力未补证 | 所有官网/GitHub/官方文档字段标“后续补证” |
| 实践门槛不足 | 多数文章有代码片段但缺完整本地运行、验收和日志 | 全部判为精读，不判实践 |
| 关键词误导 | PDF、RAG、MCP、AI 编程工具、客户案例容易抢分类 | 只保留 Agent 框架机制，相关主题不写入本类目 |

## 本轮 DeerFlow / Vercel 重分配记录

| 指标 | 数量 |
|---|---:|
| 新增技术目录 | 2 |
| 重分配 DeerFlow 原文 | 8 |
| 重分配 Vercel 原文 | 6 |
| 新增核心知识点 | 4 |

### 重分配原则

| 技术/主题 | 迁入原文范围 | 正式沉淀 | 主要认知校准点 |
|---|---|---|---|
| DeerFlow | 标题或主问题直接讨论 DeerFlow、DeerFlow 2.0、DeerFlow 与 OpenClaw/LangChain Agent Loop 对比的文章 | 多智能体编排与计划修订；源码中的中间件与 Harness 边界 | DeerFlow 不是“多个 Agent 更强”的泛化结论，而是研究型任务的计划、搜索、产物和成本控制样板 |
| Vercel | 标题或主问题直接讨论 Vercel AI SDK、Open Agents、add-skill、agent-skills、agent-browser、skills.sh 的文章 | AI SDK 多 Provider 与工具调用抽象；Open Agents 与 Skills 生态 | Vercel 子模块只吸收 AI / Agent 工程栈，不吸收普通部署平台或前端工程文章 |

## 本轮 Pi 重分配记录

| 指标 | 数量 |
|---|---:|
| 新增技术目录 | 1 |
| 重分配 Pi 原文 | 4 |
| 新增核心知识点 | 2 |

### 重分配原则

| 技术/主题 | 迁入原文范围 | 正式沉淀 | 主要认知校准点 |
|---|---|---|---|
| Pi | 标题或主问题直接讨论 Pi、pi-agent-core、pi-coding-agent、Pi SDK、Pi runtime 事件流，或把 Pi 作为被封装 provider 的文章 | 极简 Coding Agent 与可扩展 Harness；运行时事件流与会话边界 | Pi 不是单纯 AI 编程工具；它的长期价值在 runtime、事件流、session 和 SDK/RPC 边界。上层 wrapper 的审批、预算和重试不能反写成 Pi 本体能力 |

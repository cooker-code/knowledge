# Agent 框架

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](<../../目录划分.md>) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 快速入口

| 文件 | 用途 |
|---|---|
| [0201_知识地图.md](<0201_知识地图.md>) | 二级知识地图、三级节点覆盖和错放校准 |
| 二级判断准则 | 当前由本文件和 [0201_知识地图.md](<0201_知识地图.md>) 维护；未建立 `0201_核心知识点/` 前不写成入口链接 |

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [020101_DeerFlow](020101_DeerFlow/AGENTS.md) | 研究型多智能体编排、计划修订、中间件与 Harness 边界 |
| [020102_LangGraph](020102_LangGraph/AGENTS.md) | 有状态 Agent 图编排、流程控制、子图、结构化输出和 DeepResearch 状态管理 |
| [020103_OpenAI Agents SDK](<020103_OpenAI Agents SDK/AGENTS.md>) | Agent SDK 的 Harness/Compute 分离、沙箱执行、记忆控制和快照恢复 |
| [020104_Pi](020104_Pi/AGENTS.md) | 终端 Coding Agent runtime、事件流、会话边界和可扩展 Harness |
| [020105_Vercel](020105_Vercel/AGENTS.md) | Vercel AI SDK、Open Agents、Skills 分发和 Web Agent 工具生态 |
| [020106_多Agent协作](<020106_多Agent协作/AGENTS.md>) | Supervisor、角色分工、失败降级、结果合并和多 Agent 协作反模式 |
| [020112_LangChain](020112_LangChain/AGENTS.md) | LangChain Agent 抽象、Messages、输出解析、部署监控和与 LangGraph 的边界 |
| [020113_Agent协议](<020113_Agent协议/AGENTS.md>) | Agent 发现、连接、跨应用协作、客户端接线和企业运行协议 |
| [020114_长任务Agent运行时](<020114_长任务Agent运行时/AGENTS.md>) | 长时间运行、目标状态、Judge 闭环、暂停恢复和用户接管 |
| 020115_Agent设计模式（候选，未建目录） | 路由、并行、HITL、任务拆解、架构模式和模式选型边界 |
| [020116_企业级Agent平台](<020116_企业级Agent平台/AGENTS.md>) | 生产化平台、稳定性治理、意图识别、行为约束和组织级赋能 |
| [020117_DataAgent](020117_DataAgent/AGENTS.md) | 数据分析、Text2SQL、语义上下文、数据工程、告警根因和成本治理 |
| [020118_研究型Agent](<020118_研究型Agent/AGENTS.md>) | 研究、调研、验算、趋势分析、用户研究和教学研究类 Agent |
| [020119_交付型Agent](<020119_交付型Agent/AGENTS.md>) | PPT、文档、图表、视频、前端、表格、架构图等具体交付物 Agent |
| 020120_Agent工程方法（候选，未建目录） | Agent 工程学习路线、接口契约、认知基础设施、模型分层和框架选型方法 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 Agent 可调用工具、执行多步任务，无需重复解释 Agent 是什么 |
| 待补边界 | 框架、协议、运行时、设计模式、数据场景、交付场景和工程方法之间的边界 |
| 易偏差点 | 框架文章常用“最优解”“爆火”“准确率”“热榜”包装，缺基线和失败样本 |
| 优先抽取 | 状态控制、工具权限、记忆/上下文、评估、恢复、成本、可控性边界 |

## 排重指纹

```text
Agent 框架 + 控制流/状态/工具/记忆/评估/协议/场景模块 + 核心机制 + 解决问题 + 可控性边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是某框架 | 继续比较状态、工具、记忆、部署、评估和失败恢复模块 |
| 都是框架对比 | 没有同一任务、同一指标和失败样本则降权 |
| 都是教程 | 只讲安装/API 入门不新建，能补设计准则才沉淀 |
| 有失败处理和回滚 | 优先进入核心知识点 |
| 主问题跨域 | 按正文主问题迁出，不按标题里的框架名归类 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| DeerFlow | 研究型多智能体编排、计划修订、中间件与 Harness 边界 | 官方仓库结构和版本补证；最小研究任务的计划轮次、搜索轮次、成本和恢复实验 |
| LangGraph | 有状态 Agent 图编排、流程控制、子图、结构化输出和 DeepResearch 状态管理 | 官方持久化、人机中断和部署能力补证；DeepResearch 最小实验 |
| OpenAI Agents SDK | Agent SDK 的 Harness/Compute 分离、沙箱执行、记忆控制和快照恢复 | 官方文档补证；沙箱执行与快照恢复最小实验 |
| Pi | 终端 Coding Agent runtime、事件流、会话边界和可扩展 Harness | 官方包名、事件 schema 和 session 恢复补证；最小事件流实验 |
| Vercel | Vercel AI SDK、Open Agents、Skills 分发和 Web Agent 工具生态 | AI SDK 与 Open Agents 官方版本补证；add-skill 安全和来源校验 |
| 多 Agent 协作 | Supervisor、角色分工、失败降级、结果合并和多 Agent 协作反模式 | 多 Agent 失败评估集；Writer 合并规则和冲突处理实验 |
| LangChain | LangChain Agent 抽象、Messages、输出解析、部署监控和与 LangGraph 的边界 | LangChain v1.0 官方补证；LangGraph 边界最小实验；部署恢复样例 |
| Agent 协议 | Agent 发现、连接、跨应用协作、客户端接线和企业运行协议 | 官方规范补证；跨协议对标：MCP、A2A、ACP、AgentRun |
| 长任务 Agent 运行时 | 长时间运行、目标状态、Judge 闭环、暂停恢复和用户接管 | 本地长任务实验；目标状态模板；暂停恢复 SDK 官方补证 |
| Agent 设计模式 | 路由、并行、HITL、任务拆解、架构模式和模式选型边界 | 模式选择准则和反例；框架清单官方补证 |
| 企业级 Agent 平台 | 生产化平台、稳定性治理、意图识别、行为约束和组织级赋能 | 生产上线检查表；意图识别失败样本库；平台能力复用证据 |
| DataAgent | 数据分析、Text2SQL、语义上下文、数据工程、告警根因和成本治理 | 语义层资产格式；Text2SQL 评测集；权限矩阵和成本表 |
| 研究型 Agent | 研究、调研、验算、趋势分析、用户研究和教学研究类 Agent | 研究报告验收 rubric；用户研究事件模板 |
| 交付型 Agent | PPT、文档、图表、视频、前端、表格、架构图等具体交付物 Agent | 交付物验收清单；过程可编辑状态模型 |
| Agent 工程方法 | Agent 工程学习路线、接口契约、认知基础设施、模型分层和框架选型方法 | 最小实验路线；Agent-friendly CLI 检查表；框架选型决策树 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | 框架选型决策树 | 需要把 SDK、流程图、运行时、协议、场景型 Agent 分层 |
| 高 | 关键框架最小实验 | 现有结论多来自本地文章，缺真实运行证据 |
| 高 | 官方版本补证 | 涉及 v1.0、SDK、A2A、Jupyter、Managed Agents 等版本线索 |
| 中 | 多 Agent / DataAgent / 交付型 Agent 评估样本 | 需要失败样本才能沉淀生产边界 |

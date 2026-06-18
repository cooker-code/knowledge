# 评估与观测
## 知识点入口

- 本模块先看宏观流程，再看文章：[知识地图](020400_核心知识点/知识地图.md)。
- 新文章必须先归入流程节点，再判断是补充、冲突、不同层次还是降权。
- `文章/` 只保留原文锚点，长期知识必须沉淀到 `020400_核心知识点/`。


## 类目定位

| 项 | 内容 |
|---|---|
| 一级类目 | Agent 与 AI 工程 |
| 二级类目 | 评估与观测 |
| 核心问题 | 如何用指标、Trace、数据集、人工反馈和质量门禁判断 Agent 是否真的变好 |
| 不解决什么 | 不直接讨论模型训练、RAG 文档解析、工具协议设计或 AI 编程工具体验 |
| 用户当前认知假设 | L1-L2：知道需要评估，但需要补指标体系、失败分类、观测链路和迭代闭环 |

## 用户认知重点

| 认知项 | 当前假设 | 后续整理策略 |
|---|---|---|
| 已知基础 | 用户知道 Agent 不能只靠主观体验判断 | 不重复讲“评估很重要” |
| 待补边界 | QA Agent、执行型 Agent、RAG、工具调用的指标不同 | 每篇文章必须先判断评估对象 |
| 易偏差点 | 文章容易把换模型、堆 prompt 当优化，把主观好用当质量提升 | 优先抽取可验证指标、Trace 和失败样本 |

## 排重准则

问题指纹：

```text
Agent 类型 + 评估对象 + 指标/Trace/数据集/门禁机制 + 解决问题 + 可验证边界 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 Agent 评估 | 按 QA、执行型、工具调用、RAG、长任务拆分 |
| 都是指标清单 | 没有数据集和失败样本闭环时只作为了解 |
| 都是 Trace/Observability | 按 Trace、Span、Generation、Event、Score 的用途拆分 |
| 只有工具介绍 | 不新建，除非补充了评估闭环或质量门禁 |
| 有 CI/CD 或质量门禁 | 优先沉淀为工程化知识点 |

## 已覆盖技术

| 技术 | index | 已覆盖问题 | 还缺什么 |
|---|---|---|---|
| Agent 评估 | [Agent评估](020402_Agent评估/AGENTS.md) | 指标、Trace、Dataset、CI 质量门禁的基础闭环、Agent 类型化评估、pass@k/pass^k | 长任务评估、工具调用错误分类、人工反馈标注规范 |
| AI 应用评估 | [AI应用评估](020401_AI应用评估/AGENTS.md) | Langfuse 与 RAGAS 的监控、评估、数据集和 CI 闭环 | LLM-as-judge 偏差、生产坏例标注、跨 RAG/Agent 指标拆分 |

## 待补技术和问题

| 技术/问题 | 为什么要补 | 优先级 |
|---|---|---|
| LangSmith / Phoenix | Agent/RAG 观测和评估常用工具，需要和 Langfuse 对标 | 中 |
| LLM-as-judge | 自动评估常见但偏差明显 | 高 |
| 工具调用评估 | 直接影响 MCP/Skill 是否稳定 | 高 |
| 长任务 Agent 评估 | 和 Claude Code/Codex 工作流高度相关 | 高 |
| 文章抽取 Agent 小评估集 | 当前知识库流程需要持续判断分类、冲突点和排重质量 | 高 |

<!-- AUTO:SECONDARY_INIT_START -->
## 全量文章来源初始化

> 自动生成。初始化阶段只使用本地 `本地文章目录`、已有 `knowledge` 和本地 `wiki`，不联网补官网或外部证据。

- 全量文章来源：各三级节点的 `文章/`
- 全局明细：`scripts/output/knowledge-secondary-pools.json`

| 指标 | 数量 |
|---|---:|
| 文章数 | 32 |
| 正式沉淀原文数 | 2 |
| 已引用锚点 | 27 |
| 核心知识点数 | 3 |
| 精读候选 | 12 |
| 略读 | 18 |
| 跳过 | 0 |
| 低置信 | 0 |
| 原图缺失 | 8 |

### 主题簇

| 技术/主题 | 文章数 | 正式沉淀 | 精读候选 | 原图缺失 | 处理决策 | 认知校准点 |
|---|---:|---:|---:|---:|---|---|
| Agent 评估 | 32 | 2 | 12 | 8 | 以已有核心知识点为排重基线，只补边界、失败场景或实践证据 | 原目录存在误导，按技术本体重路由；有技术图缺失，精修时需回原文或重建 |

### 精读候选

| 技术对象 | 原文 | 冲突点 | 处理建议 |
|---|---|---|---|
| Agent 评估 | [【Harness 系列 08】System Prompt：贯穿所有组件的「神经系统」](<../0209_Harness Engineering/文章/【Harness 系列 08】System Prompt：贯穿所有组件的「神经系统」.md>) | 原目录与最终归类不一致；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Anthropic说：网传的Harness思路过时了，做这3件事就够！](<../0209_Harness Engineering/文章/Anthropic说：网传的Harness思路过时了，做这3件事就够！.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Browser Use：为 Agent 构建 Runtime Harness](<../0209_Harness Engineering/文章/Browser Use：为 Agent 构建 Runtime Harness.md>) | 原目录与最终归类不一致；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Harness Engineering：Agent 上生产，先过环境关](<../0209_Harness Engineering/文章/Harness Engineering：Agent 上生产，先过环境关.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Harness Engineering：让AI Agent长程运行的秘密武器](<../0209_Harness Engineering/文章/Harness Engineering：让AI Agent长程运行的秘密武器.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Harness Monitor：当多个 Agent 同时写代码时，如何看住质量](<../0209_Harness Engineering/文章/Harness Monitor：当多个 Agent 同时写代码时，如何看住质量.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [Harness 工程 Skill：使用 Entrix 技能开始你的代码熵治理](<../0209_Harness Engineering/文章/Harness 工程 Skill：使用 Entrix 技能开始你的代码熵治理.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [LangChain：如何通过 Harness Engineering 提升 Agent 表现](<../0209_Harness Engineering/文章/LangChain：如何通过 Harness Engineering 提升 Agent 表现.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [从玩具到生产力：用真实项目讲透 AI Agent 的 Harness Engineering](<../0209_Harness Engineering/文章/从玩具到生产力：用真实项目讲透 AI Agent 的 Harness Engineering.md>) | - | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [未来十年的数据工程：从 Modern Data Stack 到 Data Engineering Harness](<../0209_Harness Engineering/文章/未来十年的数据工程：从 Modern Data Stack 到 Data Engineering Harness.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [逆天的架构： 用 Harness+Langgraph+A2A  写一个 Agent Team，实现一支硅基团队。程序员 开启 当 10个Agent的boss 之路](<../0209_Harness Engineering/文章/逆天的架构： 用 Harness+Langgraph+A2A 写一个 Agent Team，实现一支硅基团队。程序员 开.md>) | 原目录与最终归类不一致 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |
| Agent 评估 | [项目越大，Agent 越乱——我用这套harness agent 把它管住了](<../0209_Harness Engineering/文章/项目越大，Agent 越乱——我用这套harness agent 把它管住了.md>) | 原目录与最终归类不一致；正文提到技术图但 Markdown 无图 | 先判问题指纹，能补边界/失败/实践再正式沉淀 |

### 冲突与缺口

- 冲突分布：原目录与最终归类不一致(21)、正文提到技术图但 Markdown 无图(8)、标题/观点需要降权(1)
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


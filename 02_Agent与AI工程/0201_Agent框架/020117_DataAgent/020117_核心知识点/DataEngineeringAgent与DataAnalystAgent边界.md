# Data Engineering Agent 与 Data Analyst Agent 边界

## 来源
- [Data Engineering Agent 和 Data Analyst Agent 到底怎么选择？](<../文章/done-Data Engineering Agent 和 Data Analyst Agent 到底怎么选择？.md>)
- [裸辞半年，我终于把data engineering agent开源了](<../文章/done-裸辞半年，我终于把data engineering agent开源了.md>)
- [如何构建企业级数据智能体：Data Agent 开发实践](<../文章/done-如何构建企业级数据智能体：Data Agent 开发实践.md>)

## 核心问题
Data Engineering Agent 偏向数据链路、开发、治理和上下文资产维护；Data Analyst Agent 偏向问题拆解、查询、分析解释和行动建议。二者共享语义资产，但产出和风险不同。

## 判断准则
- 工程 Agent 关注数据资产、依赖、质量、调度和权限；分析 Agent 关注业务问题、指标、切片和结论。
- 不要让分析 Agent 直接修改生产链路；不要让工程 Agent 负责没有业务定义的问题。
- 企业级 DataAgent 要把两者接在同一语义层和审计链上。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| DataAgent 是一个角色 | 实际至少有工程、分析、治理、告警等不同角色 |
| 分析师可以被完全替代 | 高价值环节仍是问题定义、口径裁决和行动判断 |

## 架构/流程图
暂无。

## 待验证缺口
- 补工程 Agent 与分析 Agent 的权限矩阵。

# Jupyter 数据分析 Agent 工作流边界

## 来源
- [Jupyter AI 3.0：Agent驱动的探索性数据分析工具](<../文章/done-Jupyter AI 3.0：Agent驱动的探索性数据分析工具.md>)
- [所见即所得：基于Jupyter Lab的智能数据分析工作流](<../文章/done-所见即所得：基于Jupyter Lab的智能数据分析工作流.md>)
- [Coding Agents for Data Analysis：AI数据分析实战](<../文章/done-Coding Agents for Data Analysis：AI数据分析实战.md>)
- [Multi-Agent实战：自动化数据分析](<../文章/done-Multi-Agent实战：自动化数据分析.md>)
- [不用招数据分析师了！GitHub炸星项目「AI Data Science Team」10倍速搞定清洗建模，一个人就是一支队伍](<../文章/done-不用招数据分析师了！GitHub炸星项目「AI Data Science Team」10倍速搞定清洗建模，一个人就是一支队伍.md>)
- [冷饭新炒：DeepSeek × LangChain 1.x，智能数据分析全流程再升级](<../文章/done-冷饭新炒：DeepSeek × LangChain 1.x，智能数据分析全流程再升级.md>)

## 核心问题
Jupyter 场景适合探索性数据分析 Agent，因为代码、数据、图表和解释可在同一工作区迭代；风险在于权限、环境污染和结果不可复现。

## 判断准则
- Notebook Agent 要记录数据来源、代码单元、参数、图表和结论版本。
- 多 Agent 数据分析要把 Planner、SQL/Code、Analysis、Report 的产物分开验收。
- 本地执行要隔离凭证和文件权限，避免 Agent 任意读写。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| Notebook 里跑通就可生产 | Notebook 更适合探索，生产需要调度、权限和可复现流水线 |
| 多 Agent 自动分析能替代验证 | 数据结论仍要有代码、样本和指标证据 |

## 架构/流程图
暂无。

## 待验证缺口
- 补一个可复现 notebook agent 目录模板。

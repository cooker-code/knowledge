# LangChain Middleware 与 Agent 抽象边界

## 来源
- [LangChain v1.0 让AI工作流更加立体](<../文章/done-LangChain v1.0 让AI工作流更加立体.md>)
- [如何构建你的Agents｜LangChain构建Agents指南](<../文章/done-如何构建你的Agents｜LangChain构建Agents指南.md>)
- [浅层Agent过时了？ 深度解读Agent 2.0的四大核心支柱与LangChain最新实践](<../文章/done-浅层Agent过时了？ 深度解读Agent 2.0的四大核心支柱与LangChain最新实践.md>)
- [LangChain多智能体架构深度解析(二)：SubAgents(子代理)模式原理与架构解析](<../文章/done-LangChain多智能体架构深度解析(二)：SubAgents(子代理)模式原理与架构解析.md>)

## 核心问题
LangChain 新版 Agent 抽象把模型、工具、上下文和中间件组合在一起；适合快速组装，但复杂长任务仍要评估是否需要 LangGraph 显式状态。

## 判断准则
- 中间件适合横切逻辑：过滤、上下文注入、动态模型选择、工具门禁和日志。
- 当流程分支、循环和恢复成为主问题时，应迁到 LangGraph 或显式 runtime。
- Agent 抽象只解决组合方式，不自动解决评估、权限和成本。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| LangChain 1.x 统一 Agent 就不需要 LangGraph | LangChain 和 LangGraph解决层次不同：组件组合 vs 有状态流程控制 |
| Middleware 越多越工程化 | 中间件顺序和副作用要可观测、可测试 |

## 架构/流程图
暂无。

## 待验证缺口
- 补官方 LangChain v1.0 与 LangGraph 关系说明。

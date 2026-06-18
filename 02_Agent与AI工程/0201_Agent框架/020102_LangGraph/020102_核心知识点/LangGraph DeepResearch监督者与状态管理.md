# LangGraph DeepResearch 监督者与状态管理

## 来源
- [Agent实战教程：如何从头开始使用LangGraph构建自己的DeepResearch(下)](<../文章/done-Agent实战教程：如何从头开始使用LangGraph构建自己的DeepResearch(下).md>)
- [国内首个 LangGraph Agent 模板！Multi-Agent框架最优解](<../文章/done-国内首个 LangGraph Agent 模板！Multi-Agent框架最优解.md>)
- [全面测评LangChain vs LangGraph：谁是agent落地最优解](<../文章/done-全面测评LangChain vs LangGraph：谁是agent落地最优解.md>)

## 核心问题
DeepResearch 类应用的关键不是多几个角色，而是 Supervisor 如何拆任务、限制上下文、合并状态，并在循环中决定继续研究还是交付答案。

## 判断准则
- Supervisor 必须有任务委派契约：输入、期望产物、停止条件和失败返回。
- 共享状态只放跨节点必要字段，长上下文应以摘要、证据 ID 或文件锚点传递。
- 报告合并节点要能区分证据、推断和待验证缺口。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| 多 Agent 模板就是最优解 | 模板只能说明组织方式，真正边界取决于状态合并、失败恢复和验收指标 |
| 所有子任务都应并行 | 并行只适合依赖弱、结果可合并的子任务 |

## 架构/流程图
暂无。

## 待验证缺口
- 用同一研究任务验证单 Agent、LangGraph 多节点和 DeerFlow 的成本与质量。

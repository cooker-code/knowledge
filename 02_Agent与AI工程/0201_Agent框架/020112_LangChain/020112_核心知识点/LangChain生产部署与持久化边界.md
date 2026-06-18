# LangChain 生产部署与持久化边界

## 来源
- [【LangChain】生产级Agent部署与监控](<../文章/done-【LangChain】生产级Agent部署与监控.md>)
- [FastAPI+LangChain+Streamlit实现人机交互（HITL）](<../文章/done-FastAPI+LangChain+Streamlit实现人机交互（HITL）.md>)

## 核心问题
生产级 Agent 的关键不在能否跑通 demo，而在持久化、回放、监控、人工接管和故障恢复是否可验证。

## 判断准则
- MemorySaver 这类开发态内存方案不能直接当生产记忆。
- 部署前要明确持久化介质、会话键、重启恢复、Trace 和人工中断。
- HITL 要定义暂停点、继续条件、超时策略和用户输入优先级。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| 部署就是把 demo 包进 API | 生产部署必须先设计状态、观测和恢复 |
| HITL 只是多一个确认按钮 | HITL 是控制流暂停和恢复机制 |

## 架构/流程图
暂无。

## 待验证缺口
- 补真实部署日志、checkpointer 选型和故障恢复测试。

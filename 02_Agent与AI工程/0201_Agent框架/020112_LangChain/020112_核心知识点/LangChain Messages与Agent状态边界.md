# LangChain Messages 与 Agent 状态边界

## 来源
- [深入解析 LangChain v1.0 核心组件：Messages (消息列表) 全指南](<../文章/done-深入解析 LangChain v1.0 核心组件：Messages (消息列表) 全指南.md>)
- [重磅发布！LangChain 1.0 Alpha 来了，Agent 终于统一了！](<../文章/done-重磅发布！LangChain 1.0 Alpha 来了，Agent 终于统一了！.md>)

## 核心问题
Messages 是 LangChain Agent 的交互状态承载，不只是聊天记录。它决定系统指令、用户输入、工具结果和中间状态如何被模型重新消费。

## 判断准则
- 消息历史要区分系统约束、用户目标、工具观察和最终回答。
- 不要把文件全文、日志全文长期塞进 messages；应压缩为证据锚点和摘要。
- 跨节点传递状态时，LangGraph 更适合显式状态机，LangChain 更适合组件化调用。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| Messages 只是历史聊天 | Messages 是 Agent 状态和工具观察的核心载体 |
| 消息越完整越好 | 长期任务需要摘要、裁剪和证据链接 |

## 架构/流程图
暂无。

## 待验证缺口
- 补官方 v1.0 Messages 文档和状态裁剪样例。

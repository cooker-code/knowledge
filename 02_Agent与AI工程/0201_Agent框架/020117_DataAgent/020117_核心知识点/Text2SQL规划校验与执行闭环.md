# Text2SQL 规划校验与执行闭环

## 来源
- [不整虚的！SQL Agent从规划、建设到落地的完整范例](<../文章/done-不整虚的！SQL Agent从规划、建设到落地的完整范例.md>)
- [2025最新大模型Text2SQL深度解析_ Agentar-Scale-SQL](<../文章/done-2025最新大模型Text2SQL深度解析_ Agentar-Scale-SQL.md>)
- [深度拆解NL2SQL：从Subagent到Hooks，手把手教你用Kiro](<../文章/done-深度拆解NL2SQL：从Subagent到Hooks，手把手教你用Kiro.md>)
- [开源SQL AI agent，解决自然语言转SQL，自然语言转Charts - Wren AI](<../文章/done-开源SQL AI agent，解决自然语言转SQL，自然语言转Charts - Wren AI.md>)

## 核心问题
Text2SQL Agent 不能只看生成 SQL；必须有需求理解、Schema 检索、SQL 生成、静态校验、执行校验、错误修复和结果解释的闭环。

## 判断准则
- 复杂问题先规划子问题，再生成 SQL，避免一次性猜完整查询。
- SQL 执行前做表/字段/权限/成本校验，执行后做行数、空值、异常和口径校验。
- 失败修复要基于真实错误和结果统计，不凭模型自我反省。

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| 榜单高就能业务可用 | 业务可用还取决于语义层、权限、口径和可追责 |
| SQL 生成成功就是完成 | 还要验证结果是否符合业务问题 |

## 架构/流程图
暂无。

## 待验证缺口
- 补本地 Text2SQL 小评测集和错误分类。

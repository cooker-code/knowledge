# 工具调用

> 分类规则、一二级类目定位、跨域归类边界以根 [目录划分.md](../../目录划分.md) 为准。
> 文章理解的四步法、整理流程以根 [AGENTS.md](../../AGENTS.md) 为准。
> 本文件只维护本领域的认知重点、排重指纹、已覆盖三级节点和待补缺口。

## 三级节点入口

| 节点 | 用途 |
|---|---|
| [020202_MCP](020202_MCP/AGENTS.md) | 协议三原语、Server 设计与生产接入 |
| [020203_Skill](020203_Skill/AGENTS.md) | 能力封装、动态加载与项目规则边界 |
| [020204_ToolCalling](020204_ToolCalling/AGENTS.md) | Tool Search 与工具上下文治理 |

## 候选节点（未建目录）

| 候选 | 用途 | 处理规则 |
|---|---|---|
| 020201_ComputerUse | 高权限 GUI/设备操作链路与沙箱 | 先作为工具调用候选问题处理；有足够精读来源再建三级目录 |

## 用户认知重点

| 项 | 内容 |
|---|---|
| 已知基础 | 用户知道 Agent 可调用工具，也知道 Claude Code/Codex 等有本地规则文件 |
| 待补边界 | MCP、Skill、CLAUDE.md/AGENTS.md、Plugin、Hook、Function Calling 的边界 |
| 易偏差点 | 文章常把效率体验说成技术本体，或把 Claude 相关内容都归到 LLM |
| 优先抽取 | 协议/能力包/项目规则/运行时动作/生态分发的归类与权限边界 |

## 排重指纹

```text
工具调用对象 + 封装层级 + 权限边界 + 运行时动作 + 解决问题 + 对用户的认知增量
```

| 判断项 | 排重规则 |
|---|---|
| 都是 MCP | 按协议原理、Server 设计、工具参数、权限、安全、产品适配拆分 |
| 都是 Skill | 按能力封装、动态加载、脚本/资源组织、与项目规则边界拆分 |
| 都是 Claude 生态 | 先判断是模型能力、AI 编程工具、工具调用还是项目规则 |
| 只有安装体验 | 不新建核心知识点，最多进入路由表 |
| 有权限、安全、上下文成本 | 优先沉淀为认知校准点 |

## 已覆盖问题（按三级节点）

| 节点 | 已覆盖 | 还缺 |
|---|---|---|
| Skill | Skill、MCP、项目规则边界 | 官方机制、权限模型、团队复用方式 |
| MCP | 协议三原语、与 Skill 边界、Server 工具参数设计、PostgreSQL MCP 结构化数据访问、生产接入设计模式、Chrome DevTools MCP 能力边界、异步通道、CLI/代码执行边界 | 鉴权、安全与观测、远程部署、Apps/Elicitation、Resources/Prompts 权限、浏览器工具评估 |
| ToolCalling | Tool Search 与工具上下文治理 | Function Calling 基础抽象、工具返回结构、工具调用评估 |
| ComputerUse（候选） | 高权限 GUI/设备操作链路、沙箱与审批边界 | 官方能力边界、secure computer use、隔离环境实践、失败样例 |

## 待补缺口

| 优先级 | 项 | 为什么补 |
|---|---|---|
| 高 | Function Calling | Tool Calling 的基础抽象，需要 API 层工具定义和结果回流 |
| 高 | Hook | 常和 Skill 混用，但本体是运行时事件触发 |
| 高 | MCP 安全与审计 | 数据库、浏览器、文件系统等高权限工具需要统一边界 |
| 高 | 工具调用评估 | 判断模型是否选对工具、参数和验证方式 |
| 中 | Browser Use / Playwright MCP | 与 Chrome DevTools MCP、Computer Use 对标 |

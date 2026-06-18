# AgentSandbox隔离与审计边界

> 本文件记录 `0210_sandbox` 本轮重蒸馏的文章级取舍。文章只作为来源锚点，长期知识进入同目录下的主题知识点文件。
> 本轮不联网补证；涉及官方能力、版本、性能数字和安全承诺的结论均标为待验证。

## 核心问题

判断 Agent 执行代码、浏览器、文件、命令和网络访问时，Sandbox 应该隔离什么、怎么选运行时、凭证和网络如何收口、哪些文章不应继续混入 Sandbox。

## 认知校准点

| 校准点 | 处理 |
|---|---|
| Sandbox 不是“安全与权限”的同义词 | 只保留执行环境隔离、生命周期、文件/网络/凭证/审计相关内容；Prompt 注入、Skill 审查、供应链投毒和通用权限模型剔除出本节点。 |
| 产品名不是知识点 | OpenSandbox、AgentRun、CubeSandbox、BoxLite、AIO Sandbox 只有在提供隔离边界、运行时形态或选型维度时才作为来源。 |
| 教程不等于实践 | 有安装命令但缺少输入、输出、边界、失败模式和验收方式时，降级为参考锚点。 |
| 沙箱不能替代外部控制面 | Tool Policy、审批、凭证代理和审计是沙箱落地的配套控制面，不等于沙箱本体。 |

## 主题分布

| 主题 | 保留文章 | 吸收方式 |
|---|---:|---|
| 威胁边界与控制面 | 4 | 沉淀为 [AgentSandbox威胁边界与控制面](AgentSandbox威胁边界与控制面.md)。 |
| 运行时实现与选型 | 10 | 沉淀为 [Sandbox运行时实现与选型](Sandbox运行时实现与选型.md)。 |
| 凭证网络与审计边界 | 5 | 沉淀为 [Sandbox凭证网络与审计边界](Sandbox凭证网络与审计边界.md)。 |
| 非 Sandbox 本体 | 12 | 本轮从 `0210_sandbox` 删除，不作为长期来源锚点。 |

## 保留来源锚点

| 文章 | 主问题 | 吸收结论 |
|---|---|---|
| [10 分钟搭一个 AI Agent 沙盒：OpenSandbox（Docker 本地版）实操](<../文章/done-10 分钟搭一个 AI Agent 沙盒：OpenSandbox（Docker 本地版）实操.md>) | 本地 Docker 运行 OpenSandbox 并执行代码/文件操作 | 只吸收 Docker Runtime、SDK 调用、网络限制和常见失败点；不把安装步骤写成长期知识。 |
| [AgentRun Sandbox SDK 正式开源](<../文章/done-AgentRun Sandbox SDK 正式开源！集成 LangChain 等主流框架，一键开启智能体沙箱新体验.md>) | AgentRun 作为云端 Sandbox SDK 的能力边界 | 吸收 SDK 生命周期、框架集成和云端执行环境概念，官方能力待核验。 |
| [OpenClaw 彻底带火了沙箱](<../文章/done-OpenClaw彻底带火了沙箱，桌面Agent落地必看.md>) | Agent 沙箱生态和选型对比 | 吸收选型维度，不采信未核验星标、性能和趋势判断。 |
| [OpenClaw 安全模型：威胁边界、隔离策略与可审计执行控制](<../文章/done-OpenClaw架构-OpenClaw 安全模型：威胁边界、隔离策略与可审计执行控制.md>) | 沙箱与 Tool Policy、审批、凭证、审计如何组合 | 作为控制面主锚点；抽象为“执行面 + 控制面”闭环。 |
| [OpenShell：安全沙箱隔离的沙箱隔离技术](<../文章/done-OpenShell：安全沙箱隔离的沙箱隔离技术.md>) | 沙箱隔离产品信息 | 信息量低，只保留为生态锚点。 |
| [在 Kubernetes 上使用 Agent Sandbox 运行智能体](<../文章/done-在 Kubernetes 上使用 Agent Sandbox 运行智能体.md>) | K8s 上运行 Agent Sandbox | 吸收 K8s 作为多租户调度和网络策略底座的定位。 |
| [如何在 agent 中安全地运行 python 代码](<../文章/done-如何在agent中安全的运行python代码.md>) | Python 代码执行应该放入受限环境 | 吸收“本地/远程/Docker/云沙箱”选择意识；实现细节待补。 |
| [实战指南：智能体的 Sandbox 环境如何让 AI 智能体更可靠](<../文章/done-实战指南：智能体的Sandbox环境如何让 AI 智能体更可靠！.md>) | Deep Agents / provider sandbox 生态 | 吸收提供商模式和组合方式，官方文档待补证。 |
| [开源 LiteLLM Agent 平台：K8s 沙箱 + 凭证库](<../文章/done-开源LiteLLM Agent平台：K8s沙箱+凭证库，让编程Agent永远拿不到真实API密钥.md>) | 沙箱和凭证库如何隔离真实 API 密钥 | 吸收“凭证不进入模型可写环境”的准则，平台能力待核验。 |
| [AIO Sandbox 如何用一个容器解决开发者痛点](<../文章/done-打破AI Agent开发困局：AIO Sandbox如何用一个容器解决开发者的所有痛点.md>) | 一体化容器沙箱的功能边界 | 吸收浏览器、终端、文件、MCP、资源和日志维度；营销表述降权。 |
| [Codex CLI 的沙箱到底隔离了什么](<../文章/done-第四篇：Codex CLI 的沙箱到底隔离了什么——sandbox-exec 与 Docker 深度解析.md>) | `sandbox-exec` 与 Docker 沙箱的边界差异 | 吸收本机沙箱和容器沙箱不对称、网络/宿主读取风险。 |
| [腾讯开源 CubeSandbox](<../文章/done-腾讯开源 CubeSandbox：给 AI Agent 一颗 60ms 冷启动的独立内核.md>) | 微虚拟机/独立内核类沙箱 | 吸收调度层、代理路由和冷启动维度；性能数字待官方核验。 |
| [BoxLite：轻量智能体沙箱](<../文章/done-试试 BoxLite：轻量的智能体沙箱，方便嵌入和部署.md>) | 轻量嵌入式 Agent Sandbox | 只保留为轻量运行时选型锚点。 |
| [BrowserUse + AgentRun Sandbox 最佳实践](<../文章/done-进阶指南：BrowserUse + AgentRun Sandbox 最佳实践.md>) | 浏览器自动化任务如何接入 Sandbox | 吸收浏览器会话、CDP、模板、超时、池化和日志维度。 |
| [阿里 OpenSandbox：基于 K8s 的 AI Agent 安全沙箱](<../文章/done-阿里开源 OpenSandbox：基于 K8s 的 AI Agent 安全沙箱架构解析.md>) | K8s 原生 Sandbox、网络隔离和执行引擎 | 吸收 K8s 调度和 Egress 机制，具体能力待官方补证。 |
| [Agent 执行代码的恶意脚本面试题](<../文章/done-阿里面试官问：_你的 Agent 能执行代码，用户传了个恶意脚本直接删库，沙箱怎么做的？exec() 直接用吗？白名单怎么定义？_.md>) | 不可信代码执行如何用 Docker/E2B/白名单控制 | 吸收面试题背后的威胁模型、只读挂载、禁网、资源限制和超时准则。 |

## 删除/迁出候选

| 文章 | 判断 | 处理 |
|---|---|---|
| AI Agent 狂飙时代的安全警钟：Skill Vetter | Skill 安全审查，不是 Sandbox 本体 | 从本节点删除；若后续处理 Skill 安全，归入 `020203_Skill`。 |
| AI 自主决策代码审计的小优化 | 代码审计流程，不是执行隔离 | 删除。 |
| Hooks 高级模式：构建 Claude 安全护栏 | Claude Code Hook/权限护栏，不是 Sandbox 本体 | 从本节点删除；若后续精修，归入 `020502_Claude Code`。 |
| Prompt 里的 `{}` 可能是最大漏洞 | Prompt 注入和输入转义 | 从本节点删除；若后续精修，归入 `0207_Prompt Engineering`。 |
| anything-analyzer：AI 抓包分析器 | 抓包分析工具 | 删除。 |
| LiteLLM 供应链投毒两篇 | 软件供应链安全 | 从本节点删除；若后续精修，归入 `070302_安全权限`。 |
| 公司新来了一个同事，把权限系统设计得炉火纯青 | 通用 RBAC/权限系统 | 从本节点删除；若后续精修，归入 `070302_安全权限`。 |
| 基于 Mybatis 拦截器实现数据范围权限 | 后端数据权限实现 | 从本节点删除；若后续精修，归入后端/安全权限。 |
| 大模型 prompt 注入通杀常见大模型 | Prompt 安全 | 从本节点删除；若后续精修，归入 `0207_Prompt Engineering`。 |
| 合同 AI 审查系统开源 | 产品资讯，不是 Sandbox | 删除。 |
| 融合模型权限管理设计方案 | 通用权限模型 | 从本节点删除；若后续精修，归入 `070302_安全权限`。 |

## 待验证缺口

- 官方资料补证：OpenSandbox、AgentRun、CubeSandbox、BoxLite、AIO Sandbox、Codex sandbox 的真实能力、版本和配置。
- 实验补证：同一恶意脚本在本地进程、`sandbox-exec`、Docker 禁网、K8s 网络策略、浏览器沙箱下的文件/网络/凭证边界。
- 观测补证：沙箱执行日志字段、审计事件格式、失败重试和回收策略。

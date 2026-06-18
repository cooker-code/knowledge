# Claude Code工程使用与治理边界

> 本页是该节点的核心知识点总览，用于承接文章来源锚点、判断准则和待补缺口。
> 本轮只基于本地资料整理；涉及版本、官方能力和性能数字的结论均标为待验证。

## 核心问题

判断 Claude Code 的上下文、子代理、Hook、Skill、MCP、配置、评审和长任务工作流。

## 判断准则

- Claude Code 的优势在项目上下文、工具链、规则文件和子代理组织。
- 配置/技巧文章只保留能改变权限、上下文、验证或团队协作的部分。
- Claude Code 与 Codex/Cursor/Hermes 的对比要落到工作流和治理差异。
- 文章标题中的产品名、框架名和夸张收益只作为入口；最终归类看正文主问题、机制和对用户的认知增量。
- 教程、清单、资讯类文章只保留能提供机制、边界、反例、失败模式或实践验收的部分。

## 主题分布

| 主题 | 数量 | 吸收方式 |
|---|---:|---|
| AI编程/工程交付 | 144 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| 工具/协议 | 24 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| Skill/能力包 | 19 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| 工作流/编排 | 16 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| 综合锚点 | 8 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| 安全/权限 | 6 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| 评估/观测 | 4 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| 浏览器/GUI执行 | 2 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| 生态/产品资讯 | 2 | 降权为生态锚点，只在改变边界、版本或选型判断时继续精读。 |
| RAG/知识库 | 1 | 吸收到解析、分块、召回、重排、评估和知识生命周期。 |
| 上下文/记忆 | 1 | 吸收到上下文预算、信息密度、压缩、检索或记忆注入边界。 |
| 提示词/任务契约 | 1 | 吸收到任务规格、角色边界、输出约束和反例校准。 |

## 来源锚点

| 文章 | 主题 | 吸收结论 |
|---|---|---|
| [10 万星！Claude Code 的「外挂系统」到底有多强？深度拆解 everything-claude-code](<../文章/done-10 万星！Claude Code 的「外挂系统」到底有多强？深度拆解 everything-claude-code.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [10 个让 Claude Code 脱胎换骨的 Skill、Plugin 与 CLI（2026 年 4 月精选）](<../文章/done-10 个让 Claude Code 脱胎换骨的 Skill、Plugin 与 CLI（2026 年 4 月精选）.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [11.2k stars！终端里的 AI 开发助手：Claude Code Templates 让编程效率飞起来！](<../文章/done-11.2k stars！终端里的 AI 开发助手：Claude Code Templates 让编程效率飞起来！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [15个Claude Code高手技巧，让你的AI编程效率翻倍](<../文章/done-15个Claude Code高手技巧，让你的AI编程效率翻倍.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [19700429_Claude Code、Ghostty、uv：2026 开发者工具链已经彻底换代了](<../文章/done-19700429_Claude Code、Ghostty、uv：2026 开发者工具链已经彻底换代了.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [3 步搞定！让 Claude Code 显示进度和状态](<../文章/done-3 步搞定！让 Claude Code 显示进度和状态.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [38k Star，GitHub 趋势第一：这个仓库把 Claude Code 的用法研究到了极致](<../文章/done-38k Star，GitHub 趋势第一：这个仓库把 Claude Code 的用法研究到了极致.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [50个Claude Code日常使用技巧与最佳实践](<../文章/done-50个Claude Code日常使用技巧与最佳实践.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [8个关键技巧打造高效Claude Code工作流，90%的开发者都忽略了](<../文章/done-8个关键技巧打造高效Claude Code工作流，90%的开发者都忽略了.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [9.9k Star！Claude Code 必装插件，帮你盯着 AI](<../文章/done-9.9k Star！Claude Code 必装插件，帮你盯着 AI.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [90% 的人都在错误使用 Claude Code：Anthropic 官方最佳实践全解析](<../文章/done-90% 的人都在错误使用 Claude Code：Anthropic 官方最佳实践全解析.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [AGENTS.md和CLAUDE.md — 企业级AI编程经验分享(1)](<../文章/done-AGENTS.md和CLAUDE.md — 企业级AI编程经验分享(1).md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [AI Coding 进阶之路：Claude Code 两周实战总结](<../文章/done-AI Coding 进阶之路：Claude Code 两周实战总结.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Anthropic 不会告诉你的 18 个 Claude 设置，老用户全在偷偷用](<../文章/done-Anthropic 不会告诉你的 18 个 Claude 设置，老用户全在偷偷用.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [Anthropic 亲自下场教 Claude Code：最值钱的技能是 rewind](<../文章/done-Anthropic 亲自下场教 Claude Code：最值钱的技能是 rewind.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Anthropic 又出猛招！Claude Code Skills 2.0 全面升级，你的AI技能还能这样玩？](<../文章/done-Anthropic 又出猛招！Claude Code Skills 2.0 全面升级，你的AI技能还能这样玩？.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Anthropic 官方 feature-dev 插件深度解析](<../文章/done-Anthropic 官方 feature-dev 插件深度解析.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [Anthropic 黑客松冠军项目 Everything Claude Code 完整上手攻略](<../文章/done-Anthropic 黑客松冠军项目 Everything Claude Code 完整上手攻略.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Anthropic《CLAUDE.MD编写完全指南(初始化 · 结构 · 最佳实践 · 实战样例)》](<../文章/done-Anthropic《CLAUDE.MD编写完全指南(初始化 · 结构 · 最佳实践 · 实战样例)》.md>) | 生态/产品资讯 | 降权为生态锚点，只在改变边界、版本或选型判断时继续精读。 |
| [Anthropic突然上线全新CLI：一行命令操控Claude全部API](<../文章/done-Anthropic突然上线全新CLI：一行命令操控Claude全部API.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [BMAD + Ralph 执行循环：Claude Code 的统一 AI 开发框架](<../文章/done-BMAD + Ralph 执行循环：Claude Code 的统一 AI 开发框架.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Bun 创始人这场 Claude Code 直播，暴露了 AI 编程的下一阶段](<../文章/done-Bun 创始人这场 Claude Code 直播，暴露了 AI 编程的下一阶段.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [CLAUDE.md 完整配置教程：让 AI 成为你的高效编程伙伴](<../文章/done-CLAUDE.md 完整配置教程：让 AI 成为你的高效编程伙伴.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [CLAUDE.md 最佳实践 — Claude Code 30 万行代码的血泪经验](<../文章/done-CLAUDE.md 最佳实践 — Claude Code 30 万行代码的血泪经验.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [CLAUDE.md 深度指南：让 Claude 真正懂你的项目](<../文章/done-CLAUDE.md 深度指南：让 Claude 真正懂你的项目.md>) | 生态/产品资讯 | 降权为生态锚点，只在改变边界、版本或选型判断时继续精读。 |
| [CLAUDE.md决定了你Vibe Coding的开发风格和效率](<../文章/done-CLAUDE.md决定了你Vibe Coding的开发风格和效率.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude 4.5与Claude Code 2.0更新：老金来讲清楚内容，配置和操作！](<../文章/done-Claude 4.5与Claude Code 2.0更新：老金来讲清楚内容，配置和操作！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code + Agent Teams，并行任务的最佳实践](<../文章/done-Claude Code + Agent Teams，并行任务的最佳实践.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code + Codex，我愿称之为当前最狠的AI编程组合](<../文章/done-Claude Code + Codex，我愿称之为当前最狠的AI编程组合.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code + Figma：AI 画原型完整教程，从 PRD 到设计稿只要 5 分钟](<../文章/done-Claude Code + Figma：AI 画原型完整教程，从 PRD 到设计稿只要 5 分钟.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code + git worktree 编程效率直接翻 3 倍](<../文章/done-Claude Code + git worktree 编程效率直接翻 3 倍.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code --fork-session：像 git branch 一样分叉你的 AI 对话](<../文章/done-Claude Code --fork-session：像 git branch 一样分叉你的 AI 对话.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 10 个必装插件：LSP 才是 Day One 第一位](<../文章/done-Claude Code 10 个必装插件：LSP 才是 Day One 第一位.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.0.40版本后的一些实用更新](<../文章/done-Claude Code 2.0.40版本后的一些实用更新.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.0.41-47完全拆解：Hook系统、权限自动化与Skills加载的生产级改进](<../文章/done-Claude Code 2.0.41-47完全拆解：Hook系统、权限自动化与Skills加载的生产级改进.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [Claude Code 2.1.104 补完：看得见的拦截，才是开发者要的](<../文章/done-Claude Code 2.1.104 补完：看得见的拦截，才是开发者要的.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.1.105 更新解读：PreCompact 钩子、插件后台监控与全面稳定性修复](<../文章/done-Claude Code 2.1.105 更新解读：PreCompact 钩子、插件后台监控与全面稳定性修复.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [Claude Code 2.1.105 重磅更新：Worktree、Hook、插件监控一次补齐，工作流更顺了](<../文章/done-Claude Code 2.1.105 重磅更新：Worktree、Hook、插件监控一次补齐，工作流更顺了.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [Claude Code 2.1.110 发布：全屏 TUI、Remote Control、插件稳定性一起增强](<../文章/done-Claude Code 2.1.110 发布：全屏 TUI、Remote Control、插件稳定性一起增强.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Claude Code 2.1.110：全屏无闪烁、手机推送、三十项修复同日到齐](<../文章/done-Claude Code 2.1.110：全屏无闪烁、手机推送、三十项修复同日到齐.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.1.116 发布：大 session 恢复更快，终端和插件细节继续补强](<../文章/done-Claude Code 2.1.116 发布：大 session 恢复更快，终端和插件细节继续补强.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.1.117 发布：插件治理更完整，模型与会话体验继续补强](<../文章/done-Claude Code 2.1.117 发布：插件治理更完整，模型与会话体验继续补强.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.1.117 重磅更新！28 个 CLI 变更，模型选择终于持久化 + 插件安装彻底优化 + 安全管控再升级](<../文章/done-Claude Code 2.1.117 重磅更新！28 个 CLI 变更，模型选择终于持久化 + 插件安装彻底优化 + 安全管控再升级.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [Claude Code 2.1.119 发布！我昨天升完后，配置终于持久化了，用起来省心多了](<../文章/done-Claude Code 2.1.119 发布！我昨天升完后，配置终于持久化了，用起来省心多了.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 2.1.129：1h 缓存被悄悄缩成 5 分钟，这版终于补上](<../文章/done-Claude Code 2.1.129：1h 缓存被悄悄缩成 5 分钟，这版终于补上.md>) | 上下文/记忆 | 吸收到上下文预算、信息密度、压缩、检索或记忆注入边界。 |
| [Claude Code 2.1.133：Anthropic 开始重新定义「Worktree Agent 工作流」](<../文章/done-Claude Code 2.1.133：Anthropic 开始重新定义「Worktree Agent 工作流」.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 2.1.64-2.1.71版本更新：Ultrathink回归、_claude-api上线、_loop定时调度](<../文章/done-Claude Code 2.1.64-2.1.71版本更新：Ultrathink回归、_claude-api上线、_loop定时调度.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code 2.1.90 发布：`_powerup` 教学模式上线，权限安全与性能体验一起补强](<../文章/done-Claude Code 2.1.90 发布：`_powerup` 教学模式上线，权限安全与性能体验一起补强.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [Claude Code 5 个新增让工作流更顺手的 Slash 命令](<../文章/done-Claude Code 5 个新增让工作流更顺手的 Slash 命令.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 50 个命令，用好这些你能省一半的 token ！](<../文章/done-Claude Code 50 个命令，用好这些你能省一半的 token ！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Agent View：一个界面管所有AI编程会话](<../文章/done-Claude Code Agent View：一个界面管所有AI编程会话.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Agent 设计解析](<../文章/done-Claude Code Agent 设计解析.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code CLI 服务化：将终端 AI Agent 封装成 HTTP 接口的原理与实践](<../文章/done-Claude Code CLI 服务化：将终端 AI Agent 封装成 HTTP 接口的原理与实践.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code CLI支持LSP了，编程向智能再进一步](<../文章/done-Claude Code CLI支持LSP了，编程向智能再进一步.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code Dynamic Workflows：把复杂长时任务交给一群 agent](<../文章/done-Claude Code Dynamic Workflows：把复杂长时任务交给一群 agent.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code Hooks 完全指南：从配置到生效的实战经验](<../文章/done-Claude Code Hooks 完全指南：从配置到生效的实战经验.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Hooks 实战：5个配置让你少操心](<../文章/done-Claude Code Hooks 实战：5个配置让你少操心.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Output Styles：如何让AI更懂你，一秒切换工作模式](<../文章/done-Claude Code Output Styles：如何让AI更懂你，一秒切换工作模式.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Plugins 插件市场](<../文章/done-Claude Code Plugins 插件市场.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Skill Monitor_一个可靠的skill管理工具](<../文章/done-Claude Code Skill Monitor_一个可靠的skill管理工具.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code Sub-Agent 原理与实践：极简的深度调研 Agent](<../文章/done-Claude Code Sub-Agent 原理与实践：极简的深度调研 Agent.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code Task 系统：一个可能被很多人忽略的功能](<../文章/done-Claude Code Task 系统：一个可能被很多人忽略的功能.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code hooks支持if条件，一天省下半小时](<../文章/done-Claude Code hooks支持if条件，一天省下半小时.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code plugins：打包一切的插件系统怎么用？](<../文章/done-Claude Code plugins：打包一切的插件系统怎么用？.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code v2.1.101 发布：_team-onboarding 团队指南 + 50+ 修复，企业安全&长会话稳定性双升级！](<../文章/done-Claude Code v2.1.101 发布：_team-onboarding 团队指南 + 50+ 修复，企业安全&长会话稳定性双升级！.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [Claude Code 不再空跑tools 提效 提效](<../文章/done-Claude Code 不再空跑tools 提效 提效.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code 不香了，国产开源AI 编程平台，MinMax-M2.7免费用](<../文章/done-Claude Code 不香了，国产开源AI 编程平台，MinMax-M2.7免费用.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 中 10 个必用的斜杠命令](<../文章/done-Claude Code 中 10 个必用的斜杠命令.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 之 Docker 容器化部署，快速搭建环境，高效复用团队智慧](<../文章/done-Claude Code 之 Docker 容器化部署，快速搭建环境，高效复用团队智慧.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 产品负责人 Cat Wu：AI 时代 PM 该做的 3 件事](<../文章/done-Claude Code 产品负责人 Cat Wu：AI 时代 PM 该做的 3 件事.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 使用指南ver2025.10.1](<../文章/done-Claude Code 使用指南ver2025.10.1.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 先别急着写 Skill，这 4 个 .md 文件才是底座](<../文章/done-Claude Code 先别急着写 Skill，这 4 个 .md 文件才是底座.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Code 动态工作流实操指南：把大任务拆给上百个 Agent 并行跑](<../文章/done-Claude Code 动态工作流实操指南：把大任务拆给上百个 Agent 并行跑.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 又一大动作，Dynamic Workflows（动态工作流）详解](<../文章/done-Claude Code 又一大动作，Dynamic Workflows（动态工作流）详解.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 和 Codex 命令拆解 08：终章——压箱底的 4 条命令](<../文章/done-Claude Code 和 Codex 命令拆解 08：终章——压箱底的 4 条命令.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 多 Agent 协作：Subagents 和 Agent Teams 怎么选？](<../文章/done-Claude Code 多 Agent 协作：Subagents 和 Agent Teams 怎么选？.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 多智能体协作系列一：群蜂模式](<../文章/done-Claude Code 多智能体协作系列一：群蜂模式.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 完整实践地图：从模式切换到 Hook](<../文章/done-Claude Code 完整实践地图：从模式切换到 Hook.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 官方插件 Feature Dev：7 阶段工作流让功能开发不再翻车](<../文章/done-Claude Code 官方插件 Feature Dev：7 阶段工作流让功能开发不再翻车.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 工程化指南：高效组织 .claude_ 目录](<../文章/done-Claude Code 工程化指南：高效组织 .claude_ 目录.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 并行开发完全指南：Subagents + Agent Teams + Git Worktree](<../文章/done-Claude Code 并行开发完全指南：Subagents + Agent Teams + Git Worktree .md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 并行开发实战：Subagents + Git Worktree + 工作流编排](<../文章/done-Claude Code 并行开发实战：Subagents + Git Worktree + 工作流编排.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 开了 LSP 也白开，根本原因在这里](<../文章/done-Claude Code 开了 LSP 也白开，根本原因在这里.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 心法：跑通一次，规则化一次](<../文章/done-Claude Code 心法：跑通一次，规则化一次.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 技能大全：Top 20 技能介绍 + 50+ 命令完整指南](<../文章/done-Claude Code 技能大全：Top 20 技能介绍 + 50+ 命令完整指南.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Code 接入 Chrome DevTools：让 AI 直接调试你的网页！](<../文章/done-Claude Code 接入 Chrome DevTools：让 AI 直接调试你的网页！.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code 推出 _Ultraplan，超级计划模式](<../文章/done-Claude Code 推出 _Ultraplan，超级计划模式.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 新增 Recap 功能，多开党必备神器](<../文章/done-Claude Code 新增 Recap 功能，多开党必备神器.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 新项目启动套件：5 分钟配好，以后每个项目都不用再教一遍](<../文章/done-Claude Code 新项目启动套件：5 分钟配好，以后每个项目都不用再教一遍.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 最佳实践曝光！87 条实战技巧 + 核心开发者经验全解析](<../文章/done-Claude Code 最佳实践曝光！87 条实战技巧 + 核心开发者经验全解析.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 最强助手来了：开源多代理编排神器 + macOS 通知，再也不用盯终端](<../文章/done-Claude Code 最强助手来了：开源多代理编排神器 + macOS 通知，再也不用盯终端.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 最被低估的 skill：文档技能](<../文章/done-Claude Code 最被低估的 skill：文档技能.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Code 最重要的 10 个使用技巧](<../文章/done-Claude Code 最重要的 10 个使用技巧.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 本周更新观察：持续优化多智能体工作台](<../文章/done-Claude Code 本周更新观察：持续优化多智能体工作台.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 深度拆解：一个顶级AI编程工具的核心架构](<../文章/done-Claude Code 深度拆解：一个顶级AI编程工具的核心架构.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code 源码拆解：从启动到多 Agent 扩展层](<../文章/done-Claude Code 源码拆解：从启动到多 Agent 扩展层.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 现在能操作你的电脑了](<../文章/done-Claude Code 现在能操作你的电脑了.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 用户必看：_feature-dev 从零开始进行实操](<../文章/done-Claude Code 用户必看：_feature-dev 从零开始进行实操.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 用户必看：什么时候该用 _feature-dev，什么时候别用](<../文章/done-Claude Code 用户必看：什么时候该用 _feature-dev，什么时候别用.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 用户必装的 8 个 Hooks：从此告别_它又没听话_](<../文章/done-Claude Code 用户必装的 8 个 Hooks：从此告别_它又没听话_.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 的 12 个进阶用法，多数人只用了皮毛](<../文章/done-Claude Code 的 12 个进阶用法，多数人只用了皮毛.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 的 Agent View 和 _goal，补上了 Agent 长任务的两块拼图](<../文章/done-Claude Code 的 Agent View 和 _goal，补上了 Agent 长任务的两块拼图.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 的 Plan，到底该写什么，不该写什么？](<../文章/done-Claude Code 的 Plan，到底该写什么，不该写什么？.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 真正难的不是会用，而是团队怎么配：`.claude_` 核心体系全解析](<../文章/done-Claude Code 真正难的不是会用，而是团队怎么配：`.claude_` 核心体系全解析.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 终于不闪了：一行命令，终端体验直接升维](<../文章/done-Claude Code 终于不闪了：一行命令，终端体验直接升维.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 负责人：AI-native 工程组织怎么跑起来丨Claude](<../文章/done-Claude Code 负责人：AI-native 工程组织怎么跑起来丨Claude.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 这次推出的 Agent View，auto-coder 的 _async 一年前就在跑了](<../文章/done-Claude Code 这次推出的 Agent View，auto-coder 的 _async 一年前就在跑了.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code 进化：让 Agent 从单纯执行，走向全流程闭环工作流](<../文章/done-Claude Code 进化：让 Agent 从单纯执行，走向全流程闭环工作流.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Claude Code 里，Subagents 和 Agent Teams 到底怎么选？有什么区别？](<../文章/done-Claude Code 里，Subagents 和 Agent Teams 到底怎么选？有什么区别？.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code2.0.12 发布：官宣支持插件：引领AI编程新姿势](<../文章/done-Claude Code2.0.12 发布：官宣支持插件：引领AI编程新姿势.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code、Codex 提高生产效率的必装工具！](<../文章/done-Claude Code、Codex 提高生产效率的必装工具！.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code、Codex、Cursor 终于能互通了，一个协议打通所有 AI 编程工具！](<../文章/done-Claude Code、Codex、Cursor 终于能互通了，一个协议打通所有 AI 编程工具！.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude Code中没用过Hooks？那你可真是丢了个神器](<../文章/done-Claude Code中没用过Hooks？那你可真是丢了个神器.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code创始人力荐！一行命令打造24小时专属牛马，香麻了！](<../文章/done-Claude Code创始人力荐！一行命令打造24小时专属牛马，香麻了！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code子代理上手指南](<../文章/done-Claude Code子代理上手指南.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code官方团队自曝内部用法！这23条技巧让你效率翻倍（汇总篇）](<../文章/done-Claude Code官方团队自曝内部用法！这23条技巧让你效率翻倍（汇总篇）.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code官方支持插件体系！一行命令 = Claude Code全家桶！](<../文章/done-Claude Code官方支持插件体系！一行命令 = Claude Code全家桶！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code循环任务延至7天，AI自动盯梢设计聪明](<../文章/done-Claude Code循环任务延至7天，AI自动盯梢设计聪明.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code新功能！内置定时 skills，远比我们想的强！！！](<../文章/done-Claude Code新功能！内置定时 skills，远比我们想的强！！！.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Code新手必看：8个经验让你的CLAUDE.md更精准](<../文章/done-Claude Code新手必看：8个经验让你的CLAUDE.md更精准.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code橙皮书-从入门到精通.pdf](<../文章/done-Claude Code橙皮书-从入门到精通.pdf.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code的十个高级技巧](<../文章/done-Claude Code的十个高级技巧.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code：15条实用的使用技巧](<../文章/done-Claude Code：15条实用的使用技巧.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code：不只是新项目神器，更是老项目救星](<../文章/done-Claude Code：不只是新项目神器，更是老项目救星.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code：从入门到进阶的高频实战技巧(二)](<../文章/done-Claude Code：从入门到进阶的高频实战技巧(二).md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude Code｜前端设计实现 Skill](<../文章/done-Claude Code｜前端设计实现 Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Directory：给 Claude Code 找到最佳配置](<../文章/done-Claude Directory：给 Claude Code 找到最佳配置.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Claude 子代理 vs. Agent Teams：一文讲透](<../文章/done-Claude 子代理 vs. Agent Teams：一文讲透.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [Claude 开发者平台 2026.05.11：AWS 账单 + Anthropic 全功能，绕开 Bedrock](<../文章/done-Claude 开发者平台 2026.05.11：AWS 账单 + Anthropic 全功能，绕开 Bedrock.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [ClaudeCode不止写代码！做数据分析报告贼溜！](<../文章/done-ClaudeCode不止写代码！做数据分析报告贼溜！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [ClaudeCode多开神器！这个GitHub开源看板让我同时管理10个AI智能体不崩溃](<../文章/done-ClaudeCode多开神器！这个GitHub开源看板让我同时管理10个AI智能体不崩溃.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [ClaudeForge：用于创建高质量的CLAUDE.md 指令文件符合Anthropic的Claude Code最佳实践](<../文章/done-ClaudeForge：用于创建高质量的CLAUDE.md 指令文件符合Anthropic的Claude Code最佳实践.md>) | 提示词/任务契约 | 吸收到任务规格、角色边界、输出约束和反例校准。 |
| [Cline Kanban：把 Claude Code、Codex 塞进一块看板并行干活](<../文章/done-Cline Kanban：把 Claude Code、Codex 塞进一块看板并行干活.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [CodeGraph 火了：Claude Code 终于不用满项目乱翻了](<../文章/done-CodeGraph 火了：Claude Code 终于不用满项目乱翻了.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Craft Agents：用 GUI 重做 Claude Code 的野心和底气](<../文章/done-Craft Agents：用 GUI 重做 Claude Code 的野心和底气.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Dynamic Workflows 深度解析：Claude Code 为什么把多 Agent 编排写进可执行代码](<../文章/done-Dynamic Workflows 深度解析：Claude Code 为什么把多 Agent 编排写进可执行代码.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [Git Worktrees 并行工作流完全指南：如何用 Claude Code 实现多任务高效并行](<../文章/done-Git Worktrees 并行工作流完全指南：如何用 Claude Code 实现多任务高效并行.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [GitHub 2.3 万星神器！让 Claude Code 的 Team 协作“看得见”](<../文章/done-GitHub 2.3 万星神器！让 Claude Code 的 Team 协作“看得见”.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Hooks才是Claude Code CLI 的革命性更新](<../文章/done-Hooks才是Claude Code CLI 的革命性更新.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Kyle Mistele：如何写好 CLAUDE.md](<../文章/done-Kyle Mistele：如何写好 CLAUDE.md.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [Markdown 还救得动吗？Claude Code 团队成员公开「反水」，甩出 20 个 HTML 实例，1100 万](<../文章/done-Markdown 还救得动吗？Claude Code 团队成员公开「反水」，甩出 20 个 HTML 实例，1100 万.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Markdown要被抛弃了？Claude Code工程师自曝：我已彻底放弃使用Markdown！团队倾向使用HTML！网](<../文章/done-Markdown要被抛弃了？Claude Code工程师自曝：我已彻底放弃使用Markdown！团队倾向使用HTML！网.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [MultiClaude — 多模型并行终端管理器](<../文章/done-MultiClaude — 多模型并行终端管理器.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [Nezha 开源：给 Claude Code 和 Codex 准备的 7MB 桌面工作台](<../文章/done-Nezha 开源：给 Claude Code 和 Codex 准备的 7MB 桌面工作台.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Obsidian + Claude Code + BDD 实践探索](<../文章/done-Obsidian + Claude Code + BDD 实践探索.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Obsidian教程15：Claudian——把 Claude Code 装进 Obsidian 侧边栏](<../文章/done-Obsidian教程15：Claudian——把 Claude Code 装进 Obsidian 侧边栏.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Reddit 好文分享：在高强度的使用 Claude Code 6 个月之后](<../文章/done-Reddit 好文分享：在高强度的使用 Claude Code 6 个月之后.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Spec-kit：只要你把需求和规格说清楚，Claude Code 就可以分分钟钟给你干出来！](<../文章/done-Spec-kit：只要你把需求和规格说清楚，Claude Code 就可以分分钟钟给你干出来！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Star 7.4k！claude-code-templates：一行命令搞定 Claude Code 配置，这个工具火了](<../文章/done-Star 7.4k！claude-code-templates：一行命令搞定 Claude Code 配置，这个工具火了.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Subagent - Claude Code的王炸](<../文章/done-Subagent - Claude Code的王炸.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [Unsloth API 来了，24GB 显存就能在 Claude Code、Codex、OpenClaw 里跑本地 Ag](<../文章/done-Unsloth API 来了，24GB 显存就能在 Claude Code、Codex、OpenClaw 里跑本地 Ag.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [[Alan の分享] Obsidian 终于有一套能喂给 Claude Code、Codex 和 OpenCode 的 Skills 仓库了](<../文章/done-[Alan の分享] Obsidian 终于有一套能喂给 Claude Code、Codex 和 OpenCode 的 Skills 仓库了.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [cc-connect手机控制Claude：Claude Code 推出轻量级openclaw](<../文章/done-cc-connect手机控制Claude：Claude Code 推出轻量级openclaw.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [claude code 发布新版本 2.1.100，推出更省 token 的功能](<../文章/done-claude code 发布新版本 2.1.100，推出更省 token 的功能.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [claude-devtools：了解 Claude Code 的一举一动](<../文章/done-claude-devtools：了解 Claude Code 的一举一动.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [claude-howto：一个让你真正用好 Claude Code 的开源项目](<../文章/done-claude-howto：一个让你真正用好 Claude Code 的开源项目.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [cmux：专为 Claude Code 而生的终端工作区](<../文章/done-cmux：专为 Claude Code 而生的终端工作区.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [fireworks-tech-graph：用自然语言生成工业级架构图，Claude Code 绘图神器！](<../文章/done-fireworks-tech-graph：用自然语言生成工业级架构图，Claude Code 绘图神器！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [html-anything 开源！让你感受 Claude Code 作者提到的 HTML 效果！](<../文章/done-html-anything 开源！让你感受 Claude Code 作者提到的 HTML 效果！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [【建议收藏】别让 AI 瞎发挥了，这才是 Claude Code _ OpenCode 的正确打开方式](<../文章/done-【建议收藏】别让 AI 瞎发挥了，这才是 Claude Code _ OpenCode 的正确打开方式.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [【译】Claude Code 团队经验：学会用 Agent 的眼光看世界](<../文章/done-【译】Claude Code 团队经验：学会用 Agent 的眼光看世界.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [一个万能CLAUDE.md配置，让Claude输出Token消耗暴降63%](<../文章/done-一个万能CLAUDE.md配置，让Claude输出Token消耗暴降63%.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [一个神级 Claude Code 画图插件，开源了！](<../文章/done-一个神级 Claude Code 画图插件，开源了！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [一个让 Claude Code 能够节省 94% 的工具：CodeGraph 深度拆解](<../文章/done-一个让 Claude Code 能够节省 94% 的工具：CodeGraph 深度拆解.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [一些使用 claude code 的小技巧](<../文章/done-一些使用 claude code 的小技巧.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [一天一个SKILL——fireworks-tech-graph Claude Code 指定带主题的画图神器 生成图](<../文章/done-一天一个SKILL——fireworks-tech-graph Claude Code 指定带主题的画图神器 生成图.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [一套可复制的 Claude Code 配置方案：CLAUDE.md、Rules、Commands、Hooks](<../文章/done-一套可复制的 Claude Code 配置方案：CLAUDE.md、Rules、Commands、Hooks.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [一文讲透 Claude Code 的 _goal 功能](<../文章/done-一文讲透 Claude Code 的 _goal 功能.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [一本创业书，怎么变成了 10 个 Claude Code Skill？](<../文章/done-一本创业书，怎么变成了 10 个 Claude Code Skill？.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [万字讲透Claude Code从_能用_到_真好用_的分水岭：Workspace 深度解析](<../文章/done-万字讲透Claude Code从_能用_到_真好用_的分水岭：Workspace 深度解析.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [不用 Docker 也能隔离 AI Agent：Anthropic 用 OS 原语给 Claude Code 做了沙箱](<../文章/done-不用 Docker 也能隔离 AI Agent：Anthropic 用 OS 原语给 Claude Code 做了沙箱.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [为什么Claude Code不用RAG？](<../文章/done-为什么Claude Code不用RAG？.md>) | RAG/知识库 | 吸收到解析、分块、召回、重排、评估和知识生命周期。 |
| [从 Claude Code Ultraplan 看 Claude 的云端_本地交互设计](<../文章/done-从 Claude Code Ultraplan 看 Claude 的云端_本地交互设计.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [从 Claude Code 看 AI-Native 产品设计](<../文章/done-从 Claude Code 看 AI-Native 产品设计.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [从 baoyu-skills 看 Claude Code 插件的正确姿势](<../文章/done-从 baoyu-skills 看 Claude Code 插件的正确姿势.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [使用 git worktree 在 Claude Code 中实现并行开发](<../文章/done-使用 git worktree 在 Claude Code 中实现并行开发.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [做了一个claude code 工程模板生成工具cc-template](<../文章/done-做了一个claude code 工程模板生成工具cc-template.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [再聊聊 Claude Code 工作流：这次让 AI 自己组队跑](<../文章/done-再聊聊 Claude Code 工作流：这次让 AI 自己组队跑.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [再见Cursor，玩转Claude Code 的50个实用小技巧，效率拉满！](<../文章/done-再见Cursor，玩转Claude Code 的50个实用小技巧，效率拉满！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [写一个优秀的Skill 有多难？Claude Code 内部复盘经验公开](<../文章/done-写一个优秀的Skill 有多难？Claude Code 内部复盘经验公开.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [前 Anthropic 工程师开源AI编排神器，狂揽 39000+ GitHub Star！](<../文章/done-前 Anthropic 工程师开源AI编排神器，狂揽 39000+ GitHub Star！.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [后悔没早装！推荐 8 个 Claude Code 必装 Skills](<../文章/done-后悔没早装！推荐 8 个 Claude Code 必装 Skills.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [告别拖拽画图！Claude Code 一句话生成draw.io流程图](<../文章/done-告别拖拽画图！Claude Code 一句话生成draw.io流程图.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [告别终端！ClaudeCode 最强桌面客户端来了，开源免费，可视化操作爽到飞起！](<../文章/done-告别终端！ClaudeCode 最强桌面客户端来了，开源免费，可视化操作爽到飞起！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [在微信里使用 Claude Code，刚刚在 GitHub 上开源了这个 Skill 。](<../文章/done-在微信里使用 Claude Code，刚刚在 GitHub 上开源了这个 Skill 。.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [如何优雅地使用 Claude Code 的 Workflow](<../文章/done-如何优雅地使用 Claude Code 的 Workflow.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [官方重磅：Claude Code支持插件了！这5个应用场景你必须知道](<../文章/done-官方重磅：Claude Code支持插件了！这5个应用场景你必须知道.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [当计划变成代码——Claude Code Dynamic Workflows 读后感](<../文章/done-当计划变成代码——Claude Code Dynamic Workflows 读后感.md>) | 工作流/编排 | 吸收到触发器、状态流转、审批、重试和自动化编排边界。 |
| [想让Claude Code和Codex真正帮上数开的忙，先把这两个技能装上](<../文章/done-想让Claude Code和Codex真正帮上数开的忙，先把这两个技能装上.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [我用 Claude Code 大半年了，踩过的坑比写过的代码还多。](<../文章/done-我用 Claude Code 大半年了，踩过的坑比写过的代码还多。.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [我读完了 Claude Code 的全部源码，这是我找到的 Agent 设计真相](<../文章/done-我读完了 Claude Code 的全部源码，这是我找到的 Agent 设计真相.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [我都装了哪些超级好用的Claude Code Skills？](<../文章/done-我都装了哪些超级好用的Claude Code Skills？.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [手把手教你创建第一个能用的Claude Code Skill(附完整代码)](<../文章/done-手把手教你创建第一个能用的Claude Code Skill(附完整代码).md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [把 Claude Code 塞进 Obsidian：不是调 API，是真 fork 了一个子进程](<../文章/done-把 Claude Code 塞进 Obsidian：不是调 API，是真 fork 了一个子进程.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [把 Claude Code 当初中级工程师来用：8 条让 AI 真正帮上忙的实用建议](<../文章/done-把 Claude Code 当初中级工程师来用：8 条让 AI 真正帮上忙的实用建议.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [把Claude Code用到极致](<../文章/done-把Claude Code用到极致.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [拳打 Claude Code，脚踢 Codex？新的 AI 编程“猛兽” Factory Droid 来了！](<../文章/done-拳打 Claude Code，脚踢 Codex？新的 AI 编程“猛兽” Factory Droid 来了！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [揭秘 Claude Code：AI 编程入门、原理和实现，以及免费替代 iFlow CLI](<../文章/done-揭秘 Claude Code：AI 编程入门、原理和实现，以及免费替代 iFlow CLI.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [每日一 Skills 推荐｜md-viewer-skill：Claude Code 写了一堆 Markdown，但你从来没真正「看」过它](<../文章/done-每日一 Skills 推荐｜md-viewer-skill：Claude Code 写了一堆 Markdown，但你从来没真正「看」过它.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [深度测试 Superpowers 和 everything-claude-code：哪个才是 Vibe Coding 下的最佳实践？](<../文章/done-深度测试 Superpowers 和 everything-claude-code：哪个才是 Vibe Coding 下的最佳实践？.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [深度解析：Claude Code 与 Codex 背后的六大核心架构设计](<../文章/done-深度解析：Claude Code 与 Codex 背后的六大核心架构设计.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [用了这么久 Claude Code，很多人都不知道它还有三款图形界面——移动端效率提升高达 95%](<../文章/done-用了这么久 Claude Code，很多人都不知道它还有三款图形界面——移动端效率提升高达 95%.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [用好Claude Code 新出的 _powerup 命令，让你从小白变大神](<../文章/done-用好Claude Code 新出的 _powerup 命令，让你从小白变大神.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [直接让你的 Claude Code 效率拉满，Anthropic 官方神级插件开源了！](<../文章/done-直接让你的 Claude Code 效率拉满，Anthropic 官方神级插件开源了！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [离开电脑也能推进任务：Claude Code Channels 三平台接入教程](<../文章/done-离开电脑也能推进任务：Claude Code Channels 三平台接入教程.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [第2篇：终端里的 React组件：Claude Code 如何魔改 Ink 框架](<../文章/done-第2篇：终端里的 React组件：Claude Code 如何魔改 Ink 框架.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [给Claude Code装一个_视觉外挂_](<../文章/done-给Claude Code装一个_视觉外挂_.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [聊聊 Claude Code v2.1.111：Opus 4.7，Auto 模式开放，新的思考等级 xhigh，以及“限免3次”的 ultrareview](<../文章/done-聊聊 Claude Code v2.1.111：Opus 4.7，Auto 模式开放，新的思考等级 xhigh，以及“限免3次”的 ultrareview.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [藏师傅想解决 Claude Code 最恶心的问题](<../文章/done-藏师傅想解决 Claude Code 最恶心的问题.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [融合了andrej-karpathy以及 Claude Code 建议的 AI Coding 技巧，我把他做成了一个 p](<../文章/done-融合了andrej-karpathy以及 Claude Code 建议的 AI Coding 技巧，我把他做成了一个 p.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [被Everything Claude Code坑了3天，我总结出了这5条避坑指南](<../文章/done-被Everything Claude Code坑了3天，我总结出了这5条避坑指南.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [解耦 Claude Code：简洁、分治、隔离](<../文章/done-解耦 Claude Code：简洁、分治、隔离.md>) | 安全/权限 | 作为权限、隔离、审计或风险边界锚点吸收，不直接沉淀为操作指南。 |
| [解锁 Claude Code 并行开发](<../文章/done-解锁 Claude Code 并行开发.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [让 AI 互殴、自动修复 Bug：Claude Code 创始人原来是这么编程的](<../文章/done-让 AI 互殴、自动修复 Bug：Claude Code 创始人原来是这么编程的.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [让 Claude Code 不再失忆：基于 Obsidian 的会话管理机制实现](<../文章/done-让 Claude Code 不再失忆：基于 Obsidian 的会话管理机制实现.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [让 Claude Code 成本爆降 89%，这个开源工具有点猛...](<../文章/done-让 Claude Code 成本爆降 89%，这个开源工具有点猛....md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [让 Claude Code 自己修 Bug_一套配置 + 钩子,告别每天重复教 AI](<../文章/done-让 Claude Code 自己修 Bug_一套配置 + 钩子,告别每天重复教 AI.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [让Agent彻底“开眼”！7分钟教你用Apify让Claude Code实时抓取全网数据，实用性直接10倍起飞](<../文章/done-让Agent彻底“开眼”！7分钟教你用Apify让Claude Code实时抓取全网数据，实用性直接10倍起飞.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [让Claude Code帮你精准提交代码：中文可读的 feat_fix_docs 规范一键生成](<../文章/done-让Claude Code帮你精准提交代码：中文可读的 feat_fix_docs 规范一键生成.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [谷歌 Gemini API 负责人自曝：用竞品 Claude Code 1小时复现自己团队一年成果，工程师圈炸了！](<../文章/done-谷歌 Gemini API 负责人自曝：用竞品 Claude Code 1小时复现自己团队一年成果，工程师圈炸了！.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [财务数仓 Claude AI Coding 应用实战｜得物技术](<../文章/done-财务数仓 Claude AI Coding 应用实战｜得物技术.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [软件没死，人先裁了！Claude最新插件突袭金融圈，Excel到PPT一键通杀](<../文章/done-软件没死，人先裁了！Claude最新插件突袭金融圈，Excel到PPT一键通杀.md>) | 综合锚点 | 作为 Claude Code 的补充来源锚点，后续只在提供机制、边界或反例时精读。 |
| [这 6 条 Claude Code 实用提醒，非常建议加入到 CLAUDE. md 中](<../文章/done-这 6 条 Claude Code 实用提醒，非常建议加入到 CLAUDE. md 中.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [这10 个 Claude Code 的斜杠命令一定要用起来](<../文章/done-这10 个 Claude Code 的斜杠命令一定要用起来.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [这样用 CCPM 管理Claude Code编程项目，爽到飞起！](<../文章/done-这样用 CCPM 管理Claude Code编程项目，爽到飞起！.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [适用所有团队研发提效｜带你1分钟上手基于Claude Code的AI代码评审实践](<../文章/done-适用所有团队研发提效｜带你1分钟上手基于Claude Code的AI代码评审实践.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |
| [量化基本面研究团队的CLAUDE.md、Skills和Hooks实战配置指南](<../文章/done-量化基本面研究团队的CLAUDE.md、Skills和Hooks实战配置指南.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [需求文档一团乱麻？这个24k+的Claude Code插件帮你一键生成清晰任务清单](<../文章/done-需求文档一团乱麻？这个24k+的Claude Code插件帮你一键生成清晰任务清单.md>) | AI编程/工程交付 | 吸收到需求、上下文、实现、审查、测试和交付闭环。 |

## 认知校准点

| 校准点 | 处理 |
|---|---|
| 工具体验不等于工程能力 | 只把能改变选型、权限、上下文、评估或执行链路的内容写成准则。 |
| 产品更新不等于长期知识 | 发布、榜单、技巧清单默认降权，只有影响边界或工作流时才继续精读。 |
| 跨域关键词容易误导 | 标题出现数据、前端、办公、Prompt、RAG、Skill 等词时，按正文主问题判断归属。 |

## 待补缺口

- 后续精修时需要抽样回读高价值文章正文，把本文件中的主题锚点拆成更细的机制页。
- 涉及官方能力、版本、性能数字、安全边界的内容需要联网查官方文档或 release notes。
- 当前文件先保证一级目录全量收口：裸文章清零、来源可追踪、错误归类可继续修正。

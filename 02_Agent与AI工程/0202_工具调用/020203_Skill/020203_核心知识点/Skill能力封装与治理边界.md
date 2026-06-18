# Skill能力封装与治理边界

> 本页是该节点的核心知识点总览，用于承接文章来源锚点、判断准则和待补缺口。
> 本轮只基于本地资料整理；涉及版本、官方能力和性能数字的结论均标为待验证。

## 核心问题

判断 Skill 如何把规则、脚本、资产和渐进式披露封装成可复用能力。

## 判断准则

- Skill 的价值不是多一段提示词，而是把触发条件、资源、脚本和验收流程装配成能力包。
- 高质量 Skill 必须能调试、评测、版本化和按需加载。
- 写作、办公、代码审查、项目管理等技能要按能力边界合并，不按文章逐篇建点。
- 文章标题中的产品名、框架名和夸张收益只作为入口；最终归类看正文主问题、机制和对用户的认知增量。
- 教程、清单、资讯类文章只保留能提供机制、边界、反例、失败模式或实践验收的部分。

## 主题分布

| 主题 | 数量 | 吸收方式 |
|---|---:|---|
| Skill/能力包 | 105 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| 工具/协议 | 13 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| 浏览器/GUI执行 | 6 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| 评估/观测 | 3 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| 上下文/记忆 | 1 | 吸收到上下文预算、信息密度、压缩、检索或记忆注入边界。 |

## 来源锚点

| 文章 | 主题 | 吸收结论 |
|---|---|---|
| [10 _ Skills 的调试：你的 Skill 为什么_时灵时不灵_](<../文章/done-10 _ Skills 的调试：你的 Skill 为什么_时灵时不灵_.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [423 个神级 Skills 一键下载：Agent 能力开始被「工程化」了！](<../文章/done-423 个神级 Skills 一键下载：Agent 能力开始被「工程化」了！.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [AI时代，PPT的未来是HTML，一个神奇的 Skills 推荐](<../文章/done-AI时代，PPT的未来是HTML，一个神奇的 Skills 推荐.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Agent Skills 实战：把 Code Review 规范写成 Skill](<../文章/done-Agent Skills 实战：把 Code Review 规范写成 Skill.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [Agent Skills：完整工作流（构建→测试→基准测试→迭代优化）](<../文章/done-Agent Skills：完整工作流（构建→测试→基准测试→迭代优化）.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [Agent skills：AI 能力扩展的新范式](<../文章/done-Agent skills：AI 能力扩展的新范式.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Anthropic Skills深度解析：从MCP到智能工作流的进化之路](<../文章/done-Anthropic Skills深度解析：从MCP到智能工作流的进化之路.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Anthropic Skill构建全流程流出！ClaudeCode内部工程师高赞热帖：不要把 Claude 限制得太死](<../文章/done-Anthropic Skill构建全流程流出！ClaudeCode内部工程师高赞热帖：不要把 Claude 限制得太死.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Anthropic 官方出品：别再造 Agent 了，开始构建 Skills 吧](<../文章/done-Anthropic 官方出品：别再造 Agent 了，开始构建 Skills 吧.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [AutoResearch 实战：我让 Claude 自己打磨了一个“信源可靠性研判” Skill](<../文章/done-AutoResearch 实战：我让 Claude 自己打磨了一个“信源可靠性研判” Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Chrome 推出 Skills：让浏览器进化为你的 AI Agent，附完整启动指南。](<../文章/done-Chrome 推出 Skills：让浏览器进化为你的 AI Agent，附完整启动指南。.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Claude Code Skill + UI_UX Pro Max_ 为 UI 界面构建提供可搜索设计智能的 AI 技能](<../文章/done-Claude Code Skill + UI_UX Pro Max_ 为 UI 界面构建提供可搜索设计智能的 AI 技能.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Claude Projects 和 Skills：让 AI 记住你的工作方式](<../文章/done-Claude Projects 和 Skills：让 AI 记住你的工作方式.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills 不就是把提示词存个文件夹吗？](<../文章/done-Claude Skills 不就是把提示词存个文件夹吗？.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills 已经13.6k了！ 聊聊使用和避坑指南...](<../文章/done-Claude Skills 已经13.6k了！ 聊聊使用和避坑指南....md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills 新玩法：用 skill-creator 10 分钟搞定 Excel 报表自动化，职场人必学](<../文章/done-Claude Skills 新玩法：用 skill-creator 10 分钟搞定 Excel 报表自动化，职场人必学.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Claude Skills 真正好用的都在这：3个高赞资源库一次讲清](<../文章/done-Claude Skills 真正好用的都在这：3个高赞资源库一次讲清.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills 硬核技巧：用 PDF-Skill 10 分钟搞定全类型 PDF 自动化，办公人必备](<../文章/done-Claude Skills 硬核技巧：用 PDF-Skill 10 分钟搞定全类型 PDF 自动化，办公人必备.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Claude Skills 终极指南：从新手到精通（附实战案例分析）](<../文章/done-Claude Skills 终极指南：从新手到精通（附实战案例分析）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills分享｜一键生成公众号封面](<../文章/done-Claude Skills分享｜一键生成公众号封面.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude Skills用「一个文件夹」重新定义AI扩展能力](<../文章/done-Claude Skills用「一个文件夹」重新定义AI扩展能力.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude skills：给AI一个文件夹,让它从通才变专家](<../文章/done-Claude skills：给AI一个文件夹,让它从通才变专家.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude 发布 Skills！新 Skills 大白话对比 MCP、plugins，Claude.md 核心差异](<../文章/done-Claude 发布 Skills！新 Skills 大白话对比 MCP、plugins，Claude.md 核心差异.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude 官方 Skill-Creator 深度分析](<../文章/done-Claude 官方 Skill-Creator 深度分析.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude 官方开源技能库：让 AI Agent 从_通用_变_专用_的神器](<../文章/done-Claude 官方开源技能库：让 AI Agent 从_通用_变_专用_的神器.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude 新发布的 Agent Skills 到底是啥？居然比 MCP 还厉害？](<../文章/done-Claude 新发布的 Agent Skills 到底是啥？居然比 MCP 还厉害？.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [Claude发布新功能Agent Skills，让你的Agent更专业](<../文章/done-Claude发布新功能Agent Skills，让你的Agent更专业.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude官方重磅更新skill-creator：做AI技能的效率直接翻10倍，这些新功能太实用了](<../文章/done-Claude官方重磅更新skill-creator：做AI技能的效率直接翻10倍，这些新功能太实用了.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude悄悄史诗级更新Skill-creator！我把自己的Skill优化了一遍，效果惊人](<../文章/done-Claude悄悄史诗级更新Skill-creator！我把自己的Skill优化了一遍，效果惊人.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Claude悄悄更新了Skills生成器，这绝对是一次史诗级升级。](<../文章/done-Claude悄悄更新了Skills生成器，这绝对是一次史诗级升级。.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Coze+ VisActor Skill：智能图表，触手可及](<../文章/done-Coze+ VisActor Skill：智能图表，触手可及.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Cursor 进阶配置指南：Rules_Skills_MCP_Hooks_Commands_Modes 详解](<../文章/done-Cursor 进阶配置指南：Rules_Skills_MCP_Hooks_Commands_Modes 详解.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [DeepSeek V4 × Cherry Studio Agent｜自己装技能、自己写代码、自己配图](<../文章/done-DeepSeek V4 × Cherry Studio Agent｜自己装技能、自己写代码、自己配图.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Document Skills：让 AI 秒变「文档专家」](<../文章/done-Document Skills：让 AI 秒变「文档专家」.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Flutter 发布官方 Skills ，Flutter 在 AI 领域再添一助力](<../文章/done-Flutter 发布官方 Skills ，Flutter 在 AI 领域再添一助力.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Google Labs 发布 Stitch Agent Skills：AI 编程代理的超级技能库](<../文章/done-Google Labs 发布 Stitch Agent Skills：AI 编程代理的超级技能库.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Google 总结了 5 种 Agent Skill 设计模式：决定 Agent 是否好用是里面的逻辑](<../文章/done-Google 总结了 5 种 Agent Skill 设计模式：决定 Agent 是否好用是里面的逻辑.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [MiniMax 官方开源 Skills 技能库：让AI写代码从实习生变资深工程师](<../文章/done-MiniMax 官方开源 Skills 技能库：让AI写代码从实习生变资深工程师.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Obsidian 入门40：把我的写作工作流Skill免费分享给你](<../文章/done-Obsidian 入门40：把我的写作工作流Skill免费分享给你.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Obsidian教程21：我把 Karpathy 的 LLM Wiki 做成了一个 Obsidian Skill](<../文章/done-Obsidian教程21：我把 Karpathy 的 LLM Wiki 做成了一个 Obsidian Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [OpenSkills 基础使用教程：开启 AI 编程助手的“技能”新时代](<../文章/done-OpenSkills 基础使用教程：开启 AI 编程助手的“技能”新时代.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Prompt之神李继刚开源了他的AI技能库：11个认知工具，国内平台直接用](<../文章/done-Prompt之神李继刚开源了他的AI技能库：11个认知工具，国内平台直接用.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [SKILL进阶：references分层加载，打造专业级可维护技能](<../文章/done-SKILL进阶：references分层加载，打造专业级可维护技能.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [SQL 写到崩溃？30 分钟开发 PostgreSQL Skill，让 AI 接盘](<../文章/done-SQL 写到崩溃？30 分钟开发 PostgreSQL Skill，让 AI 接盘.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skill Forge：我写的 AI Skill 工程化设计框架](<../文章/done-Skill Forge：我写的 AI Skill 工程化设计框架.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skill Graph _ SKILL.md：从单文件技能到结构化能力网络的升级路径](<../文章/done-Skill Graph _ SKILL.md：从单文件技能到结构化能力网络的升级路径.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skill Seeker：让 Claude 秒懂新框架（完整上手与实战）](<../文章/done-Skill Seeker：让 Claude 秒懂新框架（完整上手与实战）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skill Seeker：让Claude自己学会所有框架的神器](<../文章/done-Skill Seeker：让Claude自己学会所有框架的神器.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skill 取代巨型 Prompt 已经不是趋势，是正在发生的事：5 种设计模式详解](<../文章/done-Skill 取代巨型 Prompt 已经不是趋势，是正在发生的事：5 种设计模式详解.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [SkillRouter：破解大规模Skills选择难题的新范式](<../文章/done-SkillRouter：破解大规模Skills选择难题的新范式.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [SkillShare入门：基于Git跨平台同步AI智能体SKILL能力](<../文章/done-SkillShare入门：基于Git跨平台同步AI智能体SKILL能力.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Skills的最正确用法，是将整个Github压缩成你自己的超级技能库。](<../文章/done-Skills的最正确用法，是将整个Github压缩成你自己的超级技能库。.md>) | 上下文/记忆 | 吸收到上下文预算、信息密度、压缩、检索或记忆注入边界。 |
| [Skills系列_ requirements-analyst 深度解析](<../文章/done-Skills系列_ requirements-analyst 深度解析.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [Skills赏析：使用skills-refiner提升skill质量](<../文章/done-Skills赏析：使用skills-refiner提升skill质量.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Superpowers 实测分析：从小游戏案例看其 14 个 Skills 如何协同](<../文章/done-Superpowers 实测分析：从小游戏案例看其 14 个 Skills 如何协同.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Trace2Skill 把“轨迹里的局部经验”蒸馏成可迁移的 Agent Skills](<../文章/done-Trace2Skill 把“轨迹里的局部经验”蒸馏成可迁移的 Agent Skills.md>) | 评估/观测 | 吸收到质量门禁、过程证据、坏例分类和持续回归节点。 |
| [Uber 与 Anthropic 关于 AI Skills 的实践与洞察](<../文章/done-Uber 与 Anthropic 关于 AI Skills 的实践与洞察.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [Web Access：一个Skill，拉满Agent联网和浏览器能力](<../文章/done-Web Access：一个Skill，拉满Agent联网和浏览器能力.md>) | 浏览器/GUI执行 | 吸收到 GUI/浏览器执行链路、结构化状态、坐标/动作空间和沙箱边界。 |
| [[翻译] Anthropic：用 Agent Skills 赋能现实世界中的智能体](<../文章/done-[翻译] Anthropic：用 Agent Skills 赋能现实世界中的智能体.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [claude+universal-data-analyst skill，产品经理也可以因果推断分析了](<../文章/done-claude+universal-data-analyst skill，产品经理也可以因果推断分析了.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [skill-creator 项目的架构图，流程图，从 ASCII 到 SVG 到，让AI自己创建 SKILL.md 来完成](<../文章/done-skill-creator 项目的架构图，流程图，从 ASCII 到 SVG 到，让AI自己创建 SKILL.md 来完成.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [【Skills 进化论 05 系列】快速上手：安装、配置并使用你的第一个 Claude Skill](<../文章/done-【Skills 进化论 05 系列】快速上手：安装、配置并使用你的第一个 Claude Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [【实战】Skill Seekers：给你的 AI 编程助手装上“专业外挂”](<../文章/done-【实战】Skill Seekers：给你的 AI 编程助手装上“专业外挂”.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [【工具】github上千star的学术写作skills](<../文章/done-【工具】github上千star的学术写作skills.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [一个神奇的视频生成 Skills，实测，狂喜](<../文章/done-一个神奇的视频生成 Skills，实测，狂喜.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [一文分清智能体的Skills和Tools！原来这才是它能落地的关键](<../文章/done-一文分清智能体的Skills和Tools！原来这才是它能落地的关键.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [一文看懂Skills：读完50篇文章后，我安装了这些13个skills（附清单）](<../文章/done-一文看懂Skills：读完50篇文章后，我安装了这些13个skills（附清单）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [一文讲透何时使用Claude Skills：与Projects、MCP、subagents的对比](<../文章/done-一文讲透何时使用Claude Skills：与Projects、MCP、subagents的对比.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [为了不再浪费时间看新闻，我把个人情报skill做成了网站](<../文章/done-为了不再浪费时间看新闻，我把个人情报skill做成了网站.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [亲测CodeBuddy的Skills技能，说两句心里话](<../文章/done-亲测CodeBuddy的Skills技能，说两句心里话.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [从0到1：AI Skills开发实战全栈指南：(4)构建写作助手Skill](<../文章/done-从0到1：AI Skills开发实战全栈指南：(4)构建写作助手Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [优质 Skills 推荐：复刻 Manus，Planning with Files 让 AI 不再失忆（三）](<../文章/done-优质 Skills 推荐：复刻 Manus，Planning with Files 让 AI 不再失忆（三）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [使用Skill-Creator来创建属于自己的Skill](<../文章/done-使用Skill-Creator来创建属于自己的Skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [全网最全！HTML Slides 实战教学：从趋势到工具，从美学规范到打造你的专属Skill](<../文章/done-全网最全！HTML Slides 实战教学：从趋势到工具，从美学规范到打造你的专属Skill.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [写 skill 全靠感觉？新版 skill-creator 用数据说话](<../文章/done-写 skill 全靠感觉？新版 skill-creator 用数据说话.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [写一个 Skill 来优化所有 Skill —— autoresearch 的 Prompt 工程实践](<../文章/done-写一个 Skill 来优化所有 Skill —— autoresearch 的 Prompt 工程实践.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [写一个Skill_ format-markdown：笔记格式美化、AI智能总结排版](<../文章/done-写一个Skill_ format-markdown：笔记格式美化、AI智能总结排版.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [写个智能体Skill：refine-markdown-to-mkdocs _ 保证格式一致性的自动主题分类、要点提取与文](<../文章/done-写个智能体Skill：refine-markdown-to-mkdocs _ 保证格式一致性的自动主题分类、要点提取与文.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [写了个Skill，怎么评测它效果好不好](<../文章/done-写了个Skill，怎么评测它效果好不好.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [凌晨告警出问题，Skill 3 秒找到出错的 CASE WHEN](<../文章/done-凌晨告警出问题，Skill 3 秒找到出错的 CASE WHEN.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [出海建站必备：7个Claude Skills让你事半功倍](<../文章/done-出海建站必备：7个Claude Skills让你事半功倍.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [别傻傻从头写代码了，学Skill不如用好Skill-Creator，把GitHub变成了私人AppStore](<../文章/done-别傻傻从头写代码了，学Skill不如用好Skill-Creator，把GitHub变成了私人AppStore.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [别再只用 Prompt 了！Claude Skill 才是 AI 真正的效率神器！](<../文章/done-别再只用 Prompt 了！Claude Skill 才是 AI 真正的效率神器！.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [别再死磕 MCP 了！Claude 新出的 Skills，才是普通开发者的真正外挂](<../文章/done-别再死磕 MCP 了！Claude 新出的 Skills，才是普通开发者的真正外挂.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [别再问“怎么写 SKILL.md”了，直接抄生产级的Skills 库](<../文章/done-别再问“怎么写 SKILL.md”了，直接抄生产级的Skills 库.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [动手体验 Text-to-CAD：如何用 CAD Skills 让 AI Agent 生成 STEP 模型](<../文章/done-动手体验 Text-to-CAD：如何用 CAD Skills 让 AI Agent 生成 STEP 模型.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [吴恩达来信：DeepLearning.AI Skill Builder工具隆重上线！](<../文章/done-吴恩达来信：DeepLearning.AI Skill Builder工具隆重上线！.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [周报_月报 Skill：自动生成老板看得懂的工作总结](<../文章/done-周报_月报 Skill：自动生成老板看得懂的工作总结.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [在datawork中探索skills，不再需要其他claw](<../文章/done-在datawork中探索skills，不再需要其他claw.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [基于 Qwen Code Skills 实践构建自定义数据分析智能体](<../文章/done-基于 Qwen Code Skills 实践构建自定义数据分析智能体.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [外儒内法.skill](<../文章/done-外儒内法.skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [实测 Agent Skill Word 技能：从_能生成_到_敢交付_](<../文章/done-实测 Agent Skill Word 技能：从_能生成_到_敢交付_.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [实用工具推荐 _ Skill Manager —— Agent Skill 管理面板](<../文章/done-实用工具推荐 _ Skill Manager —— Agent Skill 管理面板.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [工作流的 Skill 怎么写？从 7 个顶级 Skill 中提炼的模式与最佳实践](<../文章/done-工作流的 Skill 怎么写？从 7 个顶级 Skill 中提炼的模式与最佳实践.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [干掉90%的Claude Skill文件后，token效率暴增4.8倍](<../文章/done-干掉90%的Claude Skill文件后，token效率暴增4.8倍.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [开源发布 - Skill-Lib AI Agent 技能库](<../文章/done-开源发布 - Skill-Lib AI Agent 技能库.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [微软开源 SkillOpt：把 Skill 当神经网络一样训练](<../文章/done-微软开源 SkillOpt：把 Skill 当神经网络一样训练.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [忘了龙虾🦞吧，skill-creator 2.0更值得关注](<../文章/done-忘了龙虾🦞吧，skill-creator 2.0更值得关注.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [思维漫讨 _ 我用女娲skill捏了一个写作人格](<../文章/done-思维漫讨 _ 我用女娲skill捏了一个写作人格.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [我做了一款 AI 编辑 word 的 skill，推荐给你试试，效果惊艳](<../文章/done-我做了一款 AI 编辑 word 的 skill，推荐给你试试，效果惊艳.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [我写了个 Skill，让 Agent 自动给文章配图](<../文章/done-我写了个 Skill，让 Agent 自动给文章配图.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [我把 Claude Design 做成了 Skill，人人都能成为顶级网站设计师](<../文章/done-我把 Claude Design 做成了 Skill，人人都能成为顶级网站设计师.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [拆解 data-analysis Skill 里藏着的 5 个反直觉真相](<../文章/done-拆解 data-analysis Skill 里藏着的 5 个反直觉真相.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [推荐 3 个 yyds 的 Claude Skills 开源项目（保姆级上手指南）](<../文章/done-推荐 3 个 yyds 的 Claude Skills 开源项目（保姆级上手指南）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [撸了一个AI资讯周报的skills](<../文章/done-撸了一个AI资讯周报的skills.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [极速开发出一个高质量 Claude Agent Skills 最佳实践](<../文章/done-极速开发出一个高质量 Claude Agent Skills 最佳实践.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [每日skill系列之产品经理工作流](<../文章/done-每日skill系列之产品经理工作流.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [每次数据分析都从0开始？我用Claude Code把3年数分经验封装成了一个SKILL，AI直接调用。](<../文章/done-每次数据分析都从0开始？我用Claude Code把3年数分经验封装成了一个SKILL，AI直接调用。.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [法律人学Claude｜第九期：给自己定制一个审合同Skill——Skill详解](<../文章/done-法律人学Claude｜第九期：给自己定制一个审合同Skill——Skill详解.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [热门Skill研究：Grill-Me，凭什么火遍整个开发者圈？](<../文章/done-热门Skill研究：Grill-Me，凭什么火遍整个开发者圈？.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [热门Skill研究：pm-skills，这个GitHub项目，把顶级PM方法论装进了AI里](<../文章/done-热门Skill研究：pm-skills，这个GitHub项目，把顶级PM方法论装进了AI里.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [生产级的 SKILL.md 资源再袭，来自智谱 z.ai，附 frontend-design 对比评测](<../文章/done-生产级的 SKILL.md 资源再袭，来自智谱 z.ai，附 frontend-design 对比评测.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [看了很多文章依旧不会写 Skill ？ 保姆级攻略请查收！](<../文章/done-看了很多文章依旧不会写 Skill ？ 保姆级攻略请查收！.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [研发场景十大热门 Skills 推荐](<../文章/done-研发场景十大热门 Skills 推荐.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [研发必备 skill，让开发效率翻倍](<../文章/done-研发必备 skill，让开发效率翻倍.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [程序员福音！学习新框架从此不用看文档？Skill Seeker让Claude成为你的技术导师！自动生成完整项目代码](<../文章/done-程序员福音！学习新框架从此不用看文档？Skill Seeker让Claude成为你的技术导师！自动生成完整项目代码.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [精通 Claude Skills：从入门到创建属于你自己的 AI 超能力](<../文章/done-精通 Claude Skills：从入门到创建属于你自己的 AI 超能力.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [给AI装_技能包_：Agent Skills从入门到精通（下）](<../文章/done-给AI装_技能包_：Agent Skills从入门到精通（下）.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [给小龙虾装上业务大脑：两个 SKILL 让 OpenClaw 学会查数和归因](<../文章/done-给小龙虾装上业务大脑：两个 SKILL 让 OpenClaw 学会查数和归因.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [腾讯二面：Agent怎么加载海量Skill？我答完“渐进式披露”后，面试官继续追问：检索精度、冷启动延迟、状态管理怎么办](<../文章/done-腾讯二面：Agent怎么加载海量Skill？我答完“渐进式披露”后，面试官继续追问：检索精度、冷启动延迟、状态管理怎么办.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [自我蒸馏系列：商业分析 skill](<../文章/done-自我蒸馏系列：商业分析 skill.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [花叔不公开的写作 Skill，我逆向出来了](<../文章/done-花叔不公开的写作 Skill，我逆向出来了.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [被 Claude 吞噬的应用层：从 MCP、Skills 到 PTC 的跨维度打击](<../文章/done-被 Claude 吞噬的应用层：从 MCP、Skills 到 PTC 的跨维度打击.md>) | 工具/协议 | 吸收到工具声明、参数约束、调用结果、错误恢复和协议边界。 |
| [让你的AI技能自己进化——Skill Evolution Manager](<../文章/done-让你的AI技能自己进化——Skill Evolution Manager.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [谈谈 `Claude Skills`](<../文章/done-谈谈 `Claude Skills`.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [这个 Skill 提前实现了聊天的 AI 能力](<../文章/done-这个 Skill 提前实现了聊天的 AI 能力.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [重磅开源！Anthropic 官方发布 anthropics_skills：Claude 终于学会像人一样操作电脑了！](<../文章/done-重磅开源！Anthropic 官方发布 anthropics_skills：Claude 终于学会像人一样操作电脑了！.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |
| [阿里SkillClaw：让 Agent 技能在真实使用中集体进化](<../文章/done-阿里SkillClaw：让 Agent 技能在真实使用中集体进化.md>) | Skill/能力包 | 吸收到能力封装、渐进式披露、调试评测和技能复用边界。 |

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

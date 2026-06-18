---
title: 38k Star，GitHub 趋势第一：这个仓库把 Claude Code 的用法研究到了极致
author: 林雪说AI
date: 
url: https://mp.weixin.qq.com/s?__biz=MzcwODE5ODA1Mw==&mid=2247486432&idx=1&sn=c08431701317de34c8437bd727b82a84&chksm=f45d4544db6bc2e3ba26fb005da1e1c536a011e4689729286bc3009aed84de3c863cef6616db&mpshare=1&scene=24&srcid=0413b6ofi76vAKjwb3cMDWe6&sharer_shareinfo=9d7ef16a57b21ec1b9bca0bd627a5b22&sharer_shareinfo_first=9d7ef16a57b21ec1b9bca0bd627a5b22#rd
---

# 38k Star，GitHub 趋势第一：这个仓库把 Claude Code 的用法研究到了极致

> 69 条实用技巧，10 大工作流对比，从 Vibe Coding 到 Agentic Engineering 的完整进化路径。

## 一个「用 Claude Code 写的」Claude Code 最佳实践仓库

在 GitHub Trending 上登顶第一的仓库，标题只有一句话：

**"Practice makes Claude perfect."**

这本身就是一个绝佳的证明——这个仓库本身就是用 Claude Code 构建和维护的。作者 shanraisshan 不是在教你「怎么用 Claude Code」，而是在用 Claude Code 构建「怎么用 Claude Code」的最佳实践。

38,300 颗 Star，3,600 次 Fork，336 次提交，4 月 11 日还在更新。

这是目前 GitHub 上最全面的 Claude Code 社区知识库。

## 仓库里有什么

### 1. Claude Code 核心概念全景图

仓库用一张清晰的表格列出了 Claude Code 的所有核心概念：

| 概念 | 位置 | 一句话解释 |
| --- | --- | --- |
| **Subagents** | `.claude/agents/` | 独立上下文中的自主执行者，有自定义工具、权限、模型和记忆 |
| **Commands** | `.claude/commands/` | 注入到现有上下文的知识——用户调用的 prompt 模板 |
| **Skills** | `.claude/skills/` | 可配置、可预加载、自动发现的渐进式知识注入 |
| **Hooks** | `.claude/hooks/` | 在 agentic 循环之外运行的处理器（脚本、HTTP、prompts） |
| **MCP Servers** | `.mcp.json` | 连接外部工具、数据库和 API 的协议 |
| **Plugins** | 可分发包 | Skills + Subagents + Hooks + MCP 的捆绑包 |
| **Memory** | `CLAUDE.md` | 通过文件和 `@path` 导入实现持久化上下文 |

每个概念都有对应的最优实践文档和实现示例，不是空谈理论。

### 2. 69 条经过验证的技巧

这是仓库最有价值的部分。69 条技巧来自 Claude Code 创始人 Boris Cherny、核心工程师 Thariq、以及社区的实战经验。

**Prompting（3 条）**

* 让 Claude 挑战你："grill me on these changes and don't make a PR until I pass your test"
* 遇到平庸方案时说："knowing everything you know now, scrap this and implement the elegant solution"
* 修 bug 时粘贴错误说 "fix" 就行，不要微操

**Planning（6 条）**

* 永远从 Plan Mode 开始
* 写详细的 spec，减少歧义——你越具体，输出越好
* 用第二个 Claude 审查你的 plan
* 原型 > PRD——建 20-30 个版本而不是写 spec，构建成本低所以多试

**CLAUDE.md（7 条）**

* 每个 CLAUDE.md 文件控制在 200 行以内
* 用 `<important if="...">` 标签包裹领域特定规则，防止 Claude 忽略
* 任何开发者都应该能启动 Claude 说"run the tests"并且第一次就能跑通

**Skills（9 条）**

* 用 `context: fork` 在隔离的子代理中运行 skill
* 在每个 skill 中建一个 Gotchas 部分——记录 Claude 的失败点
* skill 的 description 字段是触发器，不是摘要——写给模型看
* 不要在 skill 里写显而易见的内容——专注偏离默认行为的部分
* 不要过度约束 Claude——给目标和约束，不是一步步的指令

**Hooks（5 条）**

* 用 PostToolUse hook 自动格式化代码——Claude 生成 90%，hook 补最后 10%
* 把权限请求路由到 Opus 做安全扫描
* 用 Stop hook 在回合结束时提醒 Claude 继续或验证

**Git/PR（5 条）**

* PR 要小而聚焦——中位数 118 行，141 个 PR，一天改了 4.5 万行
* 永远 squash merge——干净的线性历史
* 在同事的 PR 上 @claude 自动生成 lint 规则

### 3. 10 大开发工作流横向对比

这是整个仓库最硬核的部分。作者收集了 10 个主流的 Claude Code 工作流框架，做了系统化对比：

| 工作流 | Stars | 独特之处 |
| --- | --- | --- |
| **Everything Claude Code** | 148k | AgentShield、多语言规则、47 个 Agent + 82 个 Command + 182 个 Skill |
| **Superpowers** | 143k | TDD 优先、Iron Laws、全计划审查 |
| **Spec Kit**（GitHub 官方） | 87k | Spec 驱动、宪法式约束、22+ 工具支持 |
| **gstack** | 68k | 角色扮演、并行 Sprint |
| **Get Shit Done** | 50k | 全新 200K 上下文、Wave 执行、XML 计划 |
| **BMAD-METHOD** | 44k | 完整 SDLC、Agent 人格、22+ 平台 |
| **OpenSpec** | 39k | 增量 Spec、Brownfield 支持、Artifact DAG |
| **oh-my-claudecode** | 27k | 团队编排、tmux worker、skill 自动注入 |

所有工作流都收敛到同一个架构模式：**Research → Plan → Execute → Review → Ship**。

但各自的侧重点不同——有的强调 TDD，有的强调 spec 驱动，有的强调并行开发。

### 4. 编排工作流：Command → Agent → Skill

仓库提出了一个核心架构模式：

**Command（命令）→ Agent（代理）→ Skill（技能）**

* **Command** 是入口：用户通过 `/slash-command` 触发工作流
* **Agent** 是执行者：在隔离的上下文中自主完成任务
* **Skill** 是知识：可复用的、渐进式披露的专业知识

三者组合，形成了从「用户意图」到「精准执行」的完整链路。

仓库里有完整的示例：一个天气编排工作流（weather-orchestrator），展示了如何用 Command 触发 Agent，Agent 调用 Skill，完成端到端的数据获取和展示。

### 5. Claude Code 正在替代的创业公司

仓库里有一张残酷的对照表：

| Claude Code 功能 | 被替代的创业公司 |
| --- | --- |
| Code Review | Greptile, CodeRabbit, Devin Review, OpenDiff |
| Voice Dictation | Wispr Flow, SuperWhisper |
| Remote Control | OpenClaw |
| Computer Use | OpenAI CUA |
| Plan Mode | Agent OS |
| Skills / Plugins | 一整批 YC AI wrapper 创业公司 |

这不是危言耸听。Claude Code 的迭代速度和功能覆盖范围，正在系统性地覆盖独立创业公司的生存空间。

## 「十亿美元问题」

仓库末尾列了一组开放问题，作者称之为「十亿美元问题」：

**关于记忆和指令：**

* CLAUDE.md 里到底该放什么、不该放什么？
* 为什么 Claude 还是会忽略 CLAUDE.md 里的 MUST 指令？

**关于 Agent、Skills 和工作流：**

* 什么时候该用 Command vs Agent vs Skill？什么时候原生 Claude Code 就够了？
* 我们能否把现有代码库转成 spec，删掉代码，然后让 AI 从 spec 精确重建？

**关于 Spec 和文档：**

* 仓库里的每个 feature 都需要 spec 吗？
* Spec 多久更新一次才不会过时？

这些问题没有标准答案，但它们定义了 AI 辅助开发的下一个前沿。

## Boris Cherny 的核心建议

作为 Claude Code 的创造者，Boris Cherny 在这个仓库中被引用最多。他最核心的几条建议：

1. **不要当保姆**（don't babysit）——让 Claude 自己修 bug，不要微操
2. **永远从 Plan Mode 开始**——想清楚再做
3. **用终端而不是 IDE**——iTerm/Ghostty/tmux 比 VS Code/Cursor 更好
4. **PR 要小**——中位数 118 行，一个 feature 一个 PR
5. **用 Esc Esc 回滚**——Claude 跑偏了不要在同一个上下文里修
6. **语音输入**——用 /voice 或 Wispr Flow，10 倍效率提升

## 写在最后

这个仓库的价值不在于告诉你「Claude Code 有什么功能」——官方文档已经做了。

它的价值在于：**从 69 条实战技巧和 10 个工作流框架中，提炼出了「如何真正用好 Claude Code」的集体智慧。**

从 Vibe Coding（随便写写看）到 Agentic Engineering（工程化的 Agent 开发），这条进化路径的每一个阶段，这个仓库都有对应的最佳实践。

而且它还在持续更新。336 次提交，最后更新在 4 月 11 日——就在两天前。

如果你用 Claude Code 写代码，这个仓库值得 Star，更值得认真读完。

---

*参考来源：*

* *GitHub 仓库：https://github.com/shanraisshan/claude-code-best-practice[1]*
* *Claude Code 官方文档：https://code.claude.com/docs[2]*
* *Boris Cherny Twitter：https://x.com/bcherny[3]*
* *Thariq（Claude Code 核心工程师）Twitter：https://x.com/trq212[4]*
* *Builder.io 50 条技巧：https://www.builder.io/blog/claude-code-tips-best-practices[5]*

### 引用链接

[1]*https://github.com/shanraisshan/claude-code-best-practice*

[2]*https://code.claude.com/docs*

[3]*https://x.com/bcherny*

[4]*https://x.com/trq212*

[5]*https://www.builder.io/blog/claude-code-tips-best-practices*
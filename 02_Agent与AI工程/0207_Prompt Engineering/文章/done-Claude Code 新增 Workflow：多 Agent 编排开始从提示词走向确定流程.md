> 已吸收至：[[02_Agent与AI工程/0207_Prompt Engineering/0207_核心知识点/Prompt任务契约与评估闭环|Prompt任务契约与评估闭环]]
---
title: Claude Code 新增 Workflow：多 Agent 编排开始从提示词走向确定流程
author: pipi的奇思妙想
date: 程序猿派派程序猿派派
url: https://mp.weixin.qq.com/s?__biz=MjM5MTE2NjI1Nw==&mid=2247492587&idx=1&sn=6497e17332c927a52ea8fd812ede00a0&chksm=a7773ab9d5b1613bf3c005e23399339e677fffe6318e66ec456c559994897529fb307ecb2ccc&mpshare=1&scene=24&srcid=0526oQsqDQh9SeSFIZNC8D6J&sharer_shareinfo=398dd1bfab272b822fb399883f45834d&sharer_shareinfo_first=398dd1bfab272b822fb399883f45834d#rd
---

## Claude Code 新增 Workflow：多 Agent 编排开始从提示词走向确定流程（补充用法版）

Claude Code v2.1.147 的更新很密，但最值得单独看的，是一个默认关闭的新工具：Workflow。它把多智能体协作从“靠提示词临场组织”，推进到更接近可复现、可配置、可审计的工程编排。

多 Agent 协作正在从聊天式分工，走向更稳定的工作流编排。

### 更新内容

**Workflow 工具**：GitHub Release 写明，v2.1.147 新增 Workflow tool，用于 deterministic multi-agent orchestration。它默认关闭，需要通过环境变量启用。这说明 Anthropic 没有把它当作普通 UI 小改，而是先放给愿意尝鲜的开发者试用。

**什么是 Workflow**：可以把它理解成一套预先写好的执行流程，而不是一句临时提示词。普通对话里，你告诉 Agent “先分析、再改代码、最后检查”，模型每次都要靠上下文自己理解和执行；Workflow 则把这些步骤、角色分工、工具调用和先后顺序固定下来，让多个 Agent 按同一套流程协作。它更像工程里的流水线或 CI 任务：不是每次重新商量怎么做，而是按可复用的流程跑完。

**怎么在 Claude Code 里用**：现阶段它更像一个实验入口，而不是默认可见的常规命令。使用前需要先升级到包含该能力的 Claude Code 版本，然后在启动 Claude Code 时打开环境变量，例如在终端执行 `CLAUDE_CODE_WORKFLOWS=1 claude`，或者先执行 `export CLAUDE_CODE_WORKFLOWS=1` 再启动 `claude`。如果启动后看不到相关入口，通常意味着当前客户端版本、账号放量或功能开关还没有拿到这一实验能力。

**实际使用方式**：它适合放在“固定流程”场景里，而不是每次都临时让模型自由发挥。比如代码迁移可以拆成“扫描旧实现、定位目标模块、分派子任务、执行修改、统一审查、跑测试”；PR 审查可以拆成“读 diff、找正确性风险、生成复现建议、按文件输出评论”。Workflow 的价值就在于把这些步骤做成可重复的编排，让主 Agent 负责调度，子 Agent 分别处理明确边界内的任务。

**当前更稳的替代做法**：如果团队还拿不到 Workflow 入口，可以先用 Claude Code 已公开稳定的 custom slash commands 和 subagents 近似实现。把常用流程写进 `.claude/commands/` 里的 Markdown 命令，再在命令中明确要求先调用分析型 subagent、再调用实现型 subagent、最后由主会话统一审查。这样虽然还不是原生 Workflow，但已经能把“靠聊天临场安排”变成“团队共享的一套流程文件”。

**代码审查入口改名**：原来的 `/simplify` 被重命名为 `/code-review`。新行为更偏向报告代码正确性问题，并可用不同 effort level 控制审查强度；传入 `--comment` 后，还能把发现写成 GitHub PR 内联评论。

**后台会话更像长期工人**：Pinned background sessions 现在在空闲时会保持存活，更新 Claude Code 时原地重启，并且只有在非 pinned 会话释放后才因内存压力被清理。对长任务和多 Agent 协作来说，这类会话生命周期细节比表面功能更重要。

**沙箱与稳定性继续补强**：Release 还提到，REPL 与 Workflow 工具沙箱针对 prototype pollution 和 thenable escape 做了加固，同时改进了大文件 diff 渲染、自动更新失败报告、Windows PowerShell 兼容性和企业登录限制。

### 为什么值得关注

过去几个月，编程 Agent 的核心矛盾已经不是“能不能写代码”，而是“能不能稳定地把一个复杂任务拆开、分派、检查、合并”。靠一个长提示词临时安排多个子任务，短期可用，但很难复现，也很难让团队把它纳入正式工程流程。

Workflow 的信号在于：Claude Code 开始把多 Agent 协作视为一等工程对象。它不是让模型自由发挥，而是把协作路径、工具调用和执行边界放进更确定的编排里。对真实代码库来说，这能降低两个风险：Agent 互相覆盖工作，以及审查者无法追溯每一步为什么发生。

`/code-review` 的变化也很有代表性。旧名字强调“简化”，新名字强调“审查”。这说明 Claude Code 的定位正在从“帮我改一下”走向“帮我发现正确性问题，并进入 PR 流程”。当内联评论成为默认能力之一，AI 就不只是本地助手，而开始进入团队协作界面。

### 对开发者的影响

**复杂任务可以更结构化**：当 Workflow 稳定后，团队可以把常见流程做成可复用编排，例如先读需求、再拆任务、再分别修改、最后统一审查。它更接近 CI/CD 里的工作流，而不是一次性对话。

**代码审查会更自动化**：新的 `/code-review` 如果和 PR 评论结合，会把 Agent 的输出直接放进团队已有的审查界面。真正的挑战会变成如何控制误报、如何要求复现、如何让 AI 评论不淹没人类审查。

**安全边界更重要**：Workflow 工具刚出现就同步加固沙箱，说明多 Agent 编排的权限面更大。Agent 能编排更多步骤，也意味着更需要清楚限制它能执行什么、访问什么、修改什么。

### 结尾

Claude Code v2.1.147 不是一次单点功能更新，而是在补齐编程 Agent 进入团队工程流所需的几个关键部件：可编排的多 Agent、能进入 PR 的审查、可持续运行的后台会话，以及更严格的沙箱。

如果说早期编程 Agent 拼的是模型能力，现在的竞争会越来越像工程系统竞争：谁能把 Agent 的工作变得可复现、可审查、可治理，谁才更可能真正进入生产代码库。

### 来源链接

https://github.com/anthropics/claude-code/releases/tag/v2.1.147

https://docs.anthropic.com/en/docs/claude-code/slash-commands

https://docs.anthropic.com/en/docs/claude-code/sub-agents
---
title: oh-my-claudecode：Claude Code 的Harness
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485499&idx=1&sn=f1ee9ec12fc404e08cca36f6b5694dad&chksm=c4e37d003e9bb46e5c62fc701698a34ba1dc5f6d421ce6c9c61f4393d76d191d0697d90c8b8f&mpshare=1&scene=24&srcid=0526rELtLHvH3SDfKDxX7a9Z&sharer_shareinfo=f9bb38407208b096237f828ad6e284bc&sharer_shareinfo_first=f9bb38407208b096237f828ad6e284bc#rd
---

Claude Code 插件生态变重了。早期增强仓库主要靠提示词和命令模板，现在 oh-my-claudecode 开始把 hooks、skills、subagents、MCP、本地状态文件串成工作流系统。今天重点分析一下oh-my-claudecode。

## 它是什么

oh-my-claudecode 更像 Claude Code 的 orchestration layer。Claude Code plugin 负责分发，skills 和 commands 负责入口，custom subagents 负责角色分工，hooks 负责拦截生命周期，MCP 负责补工具，`.omc` 目录负责记录状态。

这些部分拼起来之后，它就能做 Ralph、Autopilot、Ultrawork、Team、ccg、ralplan 这些模式。源码里看，每个模式都是 hook、状态文件、skill prompt 和 MCP tool 的组合。

它的 plugin manifest 里声明了 39 个 skills、commands 目录和一个 MCP server。这个 server 名叫 `t`，最终暴露出 `mcp__t__*` 这一组工具，包括 LSP、AST-grep、Python REPL、state、notepad、memory、trace、wiki。

OMC 直接把搜索、符号跳转、结构化替换、状态读写做成工具塞给 Claude Code。

## 关键在 Stop hook

Ralph 这类模式最容易被误解。它让 Claude 继续做，靠的是外部状态和 hook，不靠一句更强硬的提示词。

Claude Code 有 `Stop` hook。Claude 准备结束一次响应时，hook 可以检查外部状态。如果任务还没完成，hook 返回 block，并把下一步该做什么重新注入上下文。OMC 就是用 `.omc/state` 记录 Ralph、Autopilot、Team、todo 等状态，再用 persistent-mode 在 Stop 时把 Claude 拉回来。

Ralph 的结构大概是这样：先生成或校验 PRD，把用户故事和验收条件写进状态文件。执行过程中让 executor、architect、critic、verifier 等 agent 分工。Claude 想收尾时，Stop hook 检查故事、验证、回归状态。缺一步，就继续。

模型会遗忘、会压缩上下文、会想提前总结。状态文件不会忘，hook 也不会因为模型客气就放行。

## Team 有两套入口

OMC 里的 Team 要分开看。

会话内的 `/team` 是 Claude Code 里的 staged workflow。它按 `team-plan -> team-prd -> team-exec -> team-verify -> team-fix` 推进，每个阶段有不同 agent 角色。

终端里的 `omc team` 是另一套 runtime。它用 tmux 拉起 worker pane，可以跑 `claude`、`codex`、`gemini`、`cursor` 这些 CLI。

`/team` 适合当前会话里的阶段化协同。`omc team` 更适合跨 CLI、跨模型的任务。

## 和 OpenAgent、Superpowers 差在哪

oh-my-openagent，也就是原来的 oh-my-opencode 家族，走的是另一条路线。它在 OpenCode 的 plugin API 里工作，可以接触 chat params、messages transform、system transform、tool.definition、tool.execute.before/after 这些更底层的位置。

OpenAgent 可以注册 native tools，甚至做 Hashline edit。它让文件每一行带上 `LINE#ID` hash，编辑时按 hash 锚定，用 edit protocol 解决 stale-line 问题。

OMC 在 Claude Code 里没有同等级的 edit tool 替换权限。它更常用 hooks、MCP、prompt 协议、状态恢复去提升可靠性。

Superpowers 又是第三种东西。它是跨 harness 的工程方法论 skills 包，把 brainstorming、TDD、writing plans、subagent-driven development、code review 做成流程约束。

Superpowers 强在流程纪律，OMC 强在 Claude Code runtime 控制，OpenAgent 强在 OpenCode harness 权限。把这三类仓库放在一张功能表里硬比，反而会看乱。

## 怎么选

已经在 Claude Code 里工作，想让它更会规划、更能并行、更能持续推进，OMC 值得优先看。

想改底层工具协议、做多 provider 调度、让 agent 用更稳的 edit tool，那就看 OpenAgent。它对 harness 的控制更深，也更复杂。

想给任何 coding agent 加一套工程纪律，Superpowers 更轻，迁移成本也低。

还有一个实际建议：不要把多个 loop 同时打开。Ralph、Team、UltraQA、Claude Code `/goal`、Superpowers 的 continuous execution 指令都可能接管推进节奏。一个任务里最好只留一个 primary loop。

## 总结

oh-my-claudecode 值得看的点，是它把 Claude Code 官方扩展面用到了很深。它没有发明一个新 agent，它围绕 Claude Code 已经有的 plugin、hook、Skill、Task、MCP、subagent、Stop event 做了一套组合。

这类项目后面会越来越多。差距会从 prompt 写法转向 harness 权限、工具协议、状态恢复、生命周期控制。

读源码之后会发现，它更像一套 Claude Code 控制面。
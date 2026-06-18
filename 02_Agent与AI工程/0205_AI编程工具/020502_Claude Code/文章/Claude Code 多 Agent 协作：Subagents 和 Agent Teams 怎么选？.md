---
title: Claude Code 多 Agent 协作：Subagents 和 Agent Teams 怎么选？
author: 沉浸式AI
date: 沉浸式AI沉浸式AI
url: https://mp.weixin.qq.com/s?__biz=MzkyOTI2MzE0MQ==&mid=2247489829&idx=1&sn=0573bf5aaf84b6888569a1a193198371&chksm=c3d3263a75a4c9a25ad432b836cf3feb82cfb4b7291b4d9f0f4e8ff2a3feb44b21c8b35a8c1b&mpshare=1&scene=24&srcid=0515OgBmOq96eXcADBrbLeHU&sharer_shareinfo=56e199ca677c9d435b37db96e1c590df&sharer_shareinfo_first=56e199ca677c9d435b37db96e1c590df#rd
---

大家好，我是 `Immerse`

专注分享 `AI 玩法`、`独立开发`与`AI 出海`的 AGI 实践者，更多干货欢迎关注公众号 #沉浸式AI 或访问 `yaolifeng.com`

---

laude Code 有两套多 Agent 机制来处理这个问题：Subagents 和 Agent Teams。两个长得有点像，但干的事不太一样。这篇把两个都讲清楚，重点说什么时候用哪个。

如果你用过 Claude Code 的 Skill，可能会问：这跟 Skill 有什么区别？

简单说，Skill 是任务模板，定义「做什么、怎么做」；Subagent 是独立工人，有自己的上下文窗口去执行任务。

Skill 管规则，Agent 管干活。

---

## Subagents：主线不断，分身干活

Subagents 是在当前会话里派出去的「分身」，每个分身有独立的上下文窗口，干完活只把结果摘要返回给主对话。主对话不会被那些大量输出塞满。

Claude Code 自带三个内置 Subagent：

* Explore：只读权限，用 Haiku 模型。Claude 需要搜索代码库但不想改东西时自动调它，速度快、省钱
* Plan：只读权限。你进入 plan mode 时，它负责在后台收集代码库信息，不会影响主线
* General-purpose：全权限，处理需要同时探索和修改的复杂任务

这三个是自动触发的，不用手动指定。

### 自定义 Subagent

内置的不够用可以自己写。一个 Subagent 就是一个 Markdown 文件，放到对应目录里：

* `.claude/agents/`：只在当前项目生效，可以提交到 git 和团队共享
* `~/.claude/agents/`：对你所有项目生效

文件格式：

```
---  
name: code-reviewer  
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.  
tools: Read, Grep, Glob, Bash  
model: inherit  
---  
  
You are a senior code reviewer ensuring high standards of code quality and security.  
  
When invoked:  
  
1. Run git diff to see recent changes  
2. Focus on modified files  
3. Begin review immediately  
  
Review checklist:  
  
- Code is clear and readable  
- Functions and variables are well-named  
- No duplicated code  
- Proper error handling  
- No exposed secrets or API keys  
- Input validation implemented  
- Good test coverage  
- Performance considerations addressed  
  
Provide feedback organized by priority:  
  
- Critical issues (must fix)  
- Warnings (should fix)  
- Suggestions (consider improving)  
  
Include specific examples of how to fix issues.
```

`description` 字段很关键——Claude 就靠这个决定什么时候调这个 Subagent。写得越精确，触发越准。

### 主要配置项

`tools` 控制这个 Subagent 能用哪些工具。只想让它做只读研究，就限制成 `Read, Grep, Glob`，它就没法动你的代码。只写了 `tools` 字段，没列出的工具就用不了；什么都不写的话，继承主对话的全部工具。

`model` 控制用哪个模型。支持 `sonnet`、`opus`、`haiku`、`inherit`（跟主对话一样）。搜索扫描类任务用 Haiku 省钱，写代码或审查用 Sonnet。

`permissionMode` 控制权限模式。`acceptEdits` 让它自动接受文件编辑，`dontAsk` 让它遇到权限提示自动拒绝，`bypassPermissions` 跳过所有检查。

`memory` 给 Subagent 一个跨对话的记忆目录。设 `user` 存到 `~/.claude/agent-memory/`，设 `project` 存到 `.claude/agent-memory/`，跑完一次它会自己往里面写东西，下次还能看到。

也可以通过 CLI 临时定义一个，不保存到文件：

```
claude --agents '{  
  "code-reviewer": {  
    "description": "Expert code reviewer. Use proactively after code changes.",  
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",  
    "tools": ["Read", "Grep", "Glob", "Bash"],  
    "model": "sonnet"  
  }  
}'
```

### 什么时候用 Subagents

最有用的几个场景：

跑测试套件产生几千行输出——让 Subagent 去跑，只把失败的测试和报错摘要回来，主上下文省了一大截。

独立调查并行化——几个没有依赖关系的研究任务，同时派出去，比串行快：

```
Research the authentication, database, and API modules in parallel using separate subagents
```

多步骤流水线——代码审查 Subagent 找到问题，把结果传给优化 Subagent 去修：

```
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

什么时候不该用 Subagent：任务需要频繁来回调整、几个阶段共用大量上下文、改动集中在一两个文件、对响应速度敏感——这些情况直接在主对话里做更好，Subagent 启动要重新建上下文，有延迟。

---

## Agent Teams：多个独立会话，互相通信

Agent Teams 是另一个层级的东西。不是在一个会话里派分身，而是启动多个完全独立的 Claude Code 实例，它们共享一个任务列表，可以互相发消息。

它目前还是实验功能，需要先开启：

```
{  
    "env": {  
        "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"  
    }  
}
```

加到你的 `settings.json` 里就行。

开启后，告诉 Claude 你要一个 Agent Team，描述清楚任务和团队结构，它会自动建队、派成员、分配任务：

```
I'm designing a CLI tool that helps developers track TODO comments across  
their codebase. Create an agent team to explore this from different angles: one  
teammate on UX, one on technical architecture, one playing devil's advocate.
```

### 架构

一个 Agent Team 由三部分组成：

* Team lead：你在用的那个主会话，负责建队、分派任务、汇总结果
* Teammates：独立的 Claude Code 实例，各自有独立上下文窗口
* Task list：共享的任务列表，存在 `~/.claude/tasks/{team-name}/` 里
* Mailbox：代理之间互相发消息的通道

Teammates 之间可以直接通信，不用经过 lead。一个 teammate 发消息，对方自动收到，不需要 lead 中转。任务有三个状态：pending、in progress、completed。Teammate 完成一个任务后，会自动去认领下一个没人做的任务。

### 怎么操作

默认是 in-process 模式：所有 teammates 跑在你的主终端里，用 `Shift+Down` 在 lead 和各个 teammate 之间切换，切过去直接打字就是给那个 teammate 发消息。

如果你用 tmux 或 iTerm2，可以开 split-pane 模式，每个 teammate 有自己的独立窗格，同时看到所有人的输出。设置方法：

```
{  
    "teammateMode": "tmux"  
}
```

任务分配有两种方式：你告诉 lead 把哪个任务给哪个 teammate，或者 teammates 自己认领。认领时用文件锁防止多人同时抢一个任务。

你也可以直接跟某个 teammate 说话，不用经过 lead。比如你看到某个 teammate 方向跑偏了，直接 `Shift+Down` 切过去，告诉它调整方向。

### 适合 Agent Teams 的场景

并行代码审查——三个 reviewer 同时看同一个 PR，各自盯不同维度：

```
Create an agent team to review PR #142. Spawn three reviewers:  
- One focused on security implications  
- One checking performance impact  
- One validating test coverage  
Have them each review and report findings.
```

竞争假说调试——排查根因最怕一个 agent 找到一个合理解释就停了。多个 agent 互相挑战各自的理论，能活下来的才是真根因：

```
Users report the app exits after one message instead of staying connected.  
Spawn 5 agent teammates to investigate different hypotheses. Have them talk to  
each other to try to disprove each other's theories, like a scientific  
debate. Update the findings doc with whatever consensus emerges.
```

跨模块并行开发——前端、后端、测试各一个 teammate，各自负责不同文件，不会互相覆盖。

### 注意事项

Agent Teams 的 token 消耗比 Subagents 高很多，每个 teammate 是独立的 Claude 实例。3-5 个 teammate 是比较合理的起点，每人 5-6 个任务最合适，太多了协调成本上升、效果不一定好。

避免让两个 teammate 同时编辑同一个文件，会互相覆盖。拆分任务时按文件或模块分。

Lead 有时会等不住，自己去做 teammate 的任务。如果发现这个情况，直接告诉它「等你的 teammates 做完再动」。

当前已知限制：

* 不支持 `/resume` 和 `/rewind` 恢复 in-process teammates
* 任务状态有时会滞后，teammate 做完了但没标 completed，卡住了就手动告诉 lead 更新状态
* 一个 lead 只能管一个 team，做完一个再开下一个
* Teammate 不能再派子 team

---

## 两个怎么选

|  | Subagents | Agent Teams |
| --- | --- | --- |
| 通信方式 | 只能汇报结果给 lead | Teammates 互相直接发消息 |
| 协调方式 | Lead 管全程 | 共享任务列表，自行认领 |
| Token 成本 | 低，结果摘要返回 | 高，每个 teammate 独立实例 |
| 适合任务 | 独立、结果导向 | 需要讨论、相互验证 |

简单记法：任务做完给我个结果就行，用 Subagents；任务需要几个方向互相碰、互相挑战，用 Agent Teams。

两个都不难上手。Subagents 从 `/agents` 命令创建一个，写好 description 和 tools 就能跑；Agent Teams 加一行环境变量，告诉 Claude 你要一个 team 就行。先拿一个实际场景试试，感受一下「上下文不被污染」或者「多方视角同时探索」哪个更顺手。
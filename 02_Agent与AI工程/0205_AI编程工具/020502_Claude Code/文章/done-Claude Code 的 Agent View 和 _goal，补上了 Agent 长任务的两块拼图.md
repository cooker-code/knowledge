> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 的 Agent View 和 /goal，补上了 Agent 长任务的两块拼图
author: Vibe编码
date: VibeCoderVibeCoder
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODkzOTY3MA==&mid=2247485198&idx=1&sn=8c05cf119d2e8143bdbe59239378c8a8&chksm=c445d847ce49637992bbfc4cbe68aae3e0dd5e1ce045cc515d15527d0959a27a0be82e6f7d1d&mpshare=1&scene=24&srcid=0512ObyVnTu8UeoNMfQKUZY0&sharer_shareinfo=d1833ed7069247bacdd3b6f419094908&sharer_shareinfo_first=d1833ed7069247bacdd3b6f419094908#rd
---

Claude Code 2.1.139 在 2026 年 5 月 11 日发了两个功能：Agent View 和 `/goal`。

这两个功能放在同一个版本里挺有意思。Agent View 解决多 session 管理，`/goal` 解决单 session 的完成闭环。一个横向管多个 worker，一个纵向拉长一个 worker 的生命周期。

## 先说结论

Claude Code 正在从终端里的聊天式 coding agent，往本机 agent 控制台演进。

以前的典型使用方式是打开一个终端，给 Claude 一个任务，看它读文件、改代码、跑测试。任务稍微多一点，大家就开始上 tmux、iTerm tabs、git worktree、脚本、第三方 dashboard。问题很快变成：哪个 session 还在跑，哪个等我回复，哪个已经做完，哪个在改同一批文件。

Agent View 把这些 session 收到一个地方。`/goal` 则把继续做直到完成这句话从普通 prompt 变成 runtime 机制。

这两个功能背后的产品判断很明确：coding agent 的关键体验不只在单次回答质量，也在长时间运行时的控制权。

## Agent View 是 session 控制台

Agent View 通过 `claude agents` 打开。官方定义是一个屏幕管理所有后台 Claude Code session：running、blocked on you、done 都在一张列表里。

它不是 subagents 面板。Subagents 是一个主 session 里临时委派出去的 worker，结果回到主会话。Agent View 里的 session 是独立的 Claude Code session，只向你汇报。你可以把它理解成 Claude Code 自己的本机任务列表。

官方文档里提到几个核心动作：

|  |
| --- |
|  |

| 动作 | 体验 |
| --- | --- |
| dispatch | 在 Agent View 底部输入 prompt，直接启动后台 session |
| peek | 选中一行按 Space，看它在干什么 |
| reply | 不进入完整会话，直接在 peek 面板回复，解卡 |
| attach | 按 Enter 或右箭头进入完整 Claude Code session |
| detach | 空 prompt 下按左箭头回到 Agent View |

这个 UI 的重点是减少上下文切换。多 agent 并行时，最大浪费通常来自隐藏阻塞：某个 agent 早就卡在一个权限、选择题、测试失败确认上，但你没有及时看见。Agent View 把 needs input 放到更高优先级，设计上就是为了让人先处理阻塞。

社区里有个评论说得很直接：它解决的是太多 Claude Code sessions、哪个被 block、怎么 inline reply 的问题。这个判断很准。Agent View 的价值不在酷炫，而在把混乱的终端窗口变成可扫描状态表。

## 背后是本机 supervisor

Agent View 真正关键的实现点不在表格 UI，在 per-user supervisor process。

官方文档说，后台 session 由一个本机 supervisor 托管。它独立于你的 terminal，也独立于 Agent View 本身。第一次 background session 或打开 Agent View 时自动启动。每个后台 session 是一个独立 Claude Code process，父进程是 supervisor。

这就解释了为什么 Agent View 不是简单的 tmux 包装。tmux 管的是 terminal session，Claude 的 supervisor 管的是 Claude Code job state。

官方还给了几个落盘路径：`~/.claude/daemon/roster.json` 存运行中后台 session 列表，`~/.claude/jobs/<id>/state.json` 存每个 session 在 Agent View 里展示的状态。session 完成后一小时左右无人 attached，进程会被停掉释放资源，但 transcript 和状态还在，下次 attach、peek、reply 时再恢复。

文件隔离也做进去了。后台 session 如果需要写当前工作目录里的文件，会自动迁移到 `.claude/worktrees/` 下的 git worktree。这个设计很关键。没有 worktree，Agent View 只会让你更快制造并发冲突；有 worktree，它才适合真正跑多个独立任务。

权限上也有保守处理。通过 Agent View dispatch 的 session 使用目标目录里的 settings。`auto` 和 `bypassPermissions` 这类无人值守风险更高的模式，必须先在交互模式接受过，才允许后台使用。

## Agent View 的界面设计取舍

Agent View 的界面像一个 triage board：按状态分组，显示 session 名、当前动作、多久没变化。它没有试图把多个 conversation 全展开，因为多任务管理需要的是扫描和排序，不是同时阅读所有细节。

我比较认可它的几个取舍。

状态分组优先。多 agent 并行时，needs input 比 working 更重要。一个卡住的问题，可能让整个任务停几个小时。

peek 比 attach 轻。只想回答选 A 还是 B，没必要进入完整会话。inline reply 是这个功能最有价值的交互。

attach 保留完整 Claude Code。真正要调试、看上下文、继续协作时，还是回到熟悉的交互 session。

shell command 不缺席。`claude attach <id>`、`claude logs <id>`、`claude stop <id>`、`claude respawn <id>` 这些命令让 Agent View 不只是 TUI，也能被脚本接上。

社区担心的点也很现实：多开 session 会更快消耗 token；低配 plan 不一定适合；还有人希望它能管 Codex、Gemini、Aider，或者支持手机端远程审批、阻塞通知、自托管控制平面。

所以 Agent View 的边界要说清楚：它是 Claude Code 的本机 session 管理器，不是跨厂商 agent cloud。

## /goal 是完成条件，不是普通提示词

`/goal` 的定义是设置一个 completion condition。Claude 每一轮结束后都会检查这个条件，不满足就继续开下一轮，满足就自动清除 goal。

这和你在 prompt 里写请继续直到测试通过不一样。普通 prompt 只是意图，模型很容易在中途总结、停下、或者用一个弱证据判断自己完成了。`/goal` 把是否完成这件事放进 session runtime。

官方文档给的实现是：`/goal` 包在一个 session-scoped prompt-based Stop hook 之上。每次 Claude 完成一轮，goal condition 和目前对话会被发给配置里的 small fast model，默认是 Haiku。这个小模型给出 yes/no 和短理由。no 会指导下一轮继续，yes 会清掉 goal，并在 transcript 里记录 achieved。

这个设计很聪明，也有硬边界。

聪明的地方是复用 hooks 和小模型。Claude Code 已经有 hooks，有 transcript，有 status UI，有 small fast model。`/goal` 不需要另写一套复杂 planner。

边界在于 evaluator 不运行命令、不读文件，只看 Claude 已经展示在 conversation 里的内容。你写所有 auth tests pass，Claude 就必须真的跑测试，并把输出留在对话里。否则 evaluator 没有 repo 级证据。

这会反过来影响 goal 的写法。好的 goal 要有终态、检查方式、约束。

```
/goal npm test exits 0, npm run lint exits 0, and git status shows only the intended files changed; stop after 12 turns if still failing and summarize blockers
```

这比把项目修好靠谱得多。`/goal` 不是魔法，它只是让可验证目标持续被追踪。

## 和 /loop、auto mode 的区别

官方文档也把几个概念分开了。

|  |
| --- |
|  |

| 机制 | 下一轮何时开始 | 何时停止 |
| --- | --- | --- |
| `/goal` | 上一轮结束后 | 小模型确认条件满足 |
| `/loop` | 时间间隔到了 | 用户停掉，或 Claude 自己判断完成 |
| Stop hook | 上一轮结束后 | 你自己的脚本或 prompt 决定 |
| auto mode | 不自动开下一轮 | 只是在单轮里自动批准工具 |

这张表其实说明了 Claude Code 的自动化层次。auto mode 解决 tool permission，`/loop` 解决定时触发，Stop hook 给高级用户做自定义判断，`/goal` 把最常见的做到完成为普通用户包装出来。

我的判断是，`/goal` 会比 `/loop` 更常用在编码任务里。编码任务大多不靠每隔五分钟检查一次推进，常见节奏是上一轮做完马上看证据，缺什么补什么。`/goal` 更贴近实现、测试、修复的循环。

## 和 Codex /goal 的差别

Codex 也有 `/goal`。我核对了一下本地 `/Users/a1/Desktop/repo/codex`，它的实现路线和 Claude Code 不太一样。

Codex 这边的链路大致是：TUI slash command 解析 `/goal`，通过 app event 进入 `thread_goal_actions.rs`，再走 app-server protocol 的 `thread/goal/set|get|clear`，最后落到 app-server 的 `thread_goal_processor.rs` 和 state-db 的 `thread_goals`。

更关键的是 core runtime。`codex-rs/core/src/goals.rs` 里有 continuation template、budget limit template、状态恢复、token/time accounting、idle continuation。Codex 还给模型暴露了 `get_goal`、`create_goal`、`update_goal` 这些工具，其中 `update_goal` 只能把 goal 标成 complete，不能让模型自己 pause 或 budget\_limited。

对比下来，两边思路不同。

|  |
| --- |
|  |

| 维度 | Claude Code /goal | Codex /goal |
| --- | --- | --- |
| 完成判断 | 小快模型在 Stop hook 后做 yes/no 判断 | 主模型必须调用 `update_goal(status=complete)` |
| 继续机制 | evaluator 返回 no 后继续下一轮 | runtime 在 idle 时注入 continuation developer prompt |
| 证据要求 | evaluator 只能看 transcript | continuation prompt 要求做 completion audit，检查真实文件、命令、测试、PR 状态 |
| 状态模型 | active / achieved / cleared 更偏用户态 | active / paused / budget\_limited / complete，更偏 runtime 状态机 |
| 预算控制 | 可以把 turn/time 限制写进 condition | 有结构化 token budget，达到后 runtime 注入 wrap-up 指令 |

Claude Code 的优势是完成判断来自独立小模型，主模型不容易自己宣布胜利。Codex 的优势是 runtime contract 更硬，状态、预算、继续工作、完成工具都在系统里，completion prompt 也更强调实证审计。

如果只看防止模型偷懒，Claude 的 separate evaluator 更直观。如果看长任务运行时治理，Codex 的状态机更完整。

## 社区真正关心的事

Agent View 的社区讨论很一致：大家已经被多终端、多 session、多 worktree 折腾过。两个月前就有人做过类似 agent-view 的 side project，用 tmux 汇总所有 agent session。现在官方做进 Claude Code，说明这个需求已经不是小众玩法。

但讨论里也有几个现实担心。

token 消耗会更快。并行不是免费提速，尤其 Opus 多开时，成本会非常明显。

worktree 仍然是关键。Agent View 解决看得见，worktree 解决别互相改坏。这两个不能互相替代。

跨工具管理还没解决。很多重度用户同时跑 Claude Code、Codex、Gemini、Aider、OpenCode。Agent View 目前只管 Claude Code。

远程控制还没完全满足。手机端审批、push notification、rate limit 后自动恢复、hung tool recovery，这些都是 Agent View 之后自然冒出来的下一层需求。

`/goal` 的讨论则集中在另一个问题：它和 keep going until tests pass 到底差在哪。

差别在完成判断的外部化。普通 prompt 靠模型自觉，`/goal` 至少有一个 hook 和 evaluator。社区里还有人提到 Ralph Loop、adversarial review、context shedding。这里要区分一下：有些帖子聊的是第三方 goal 插件，不等同于 2.1.139 官方实现，但问题意识是一致的。大家真正想要的是一个不会半路停、不会轻易自证完成、还能受预算约束的长任务循环，slash command 只是入口。

## 对开发者的使用建议

Agent View 适合独立任务并行，不适合把一个强耦合大任务切成一堆互相抢文件的 session。比如一个 session 修 auth flaky test，一个 session 做 PR review，一个 session 查生产日志，一个 session 更新文档，这种很合适。

如果多个 session 会改同一片代码，要先用 worktree 或明确文件归属。Agent View 能让冲突更容易看见，但不会替你做架构分工。

`/goal` 适合验收标准明确的任务。写 goal 时不要写愿望，要写证据。

可以这样写：

```
/goal all tests in test/auth pass, npm run lint exits 0, and no test assertions are weakened; stop after 15 turns and summarize blockers if still failing
```

不要这样写：

```
/goal make auth good
```

`/goal` 还应该和 auto mode 分开理解。auto mode 让一轮里的工具调用少打断你，`/goal` 让一轮结束后继续推进。无人值守想跑得顺，两者经常要配合，但风险也一起上升，所以权限、worktree、测试边界要提前想清楚。

## 总结

Claude Code 2.1.139 这两个功能看起来一个是 UI，一个是 slash command，其实都在补 agent 长任务运行时。

Agent View 解决横向管理：多个 session 怎么看、怎么切、怎么回复、怎么恢复。`/goal` 解决纵向闭环：一个 session 怎么围绕完成条件持续推进，怎么避免每轮都等人说继续。

这背后的产品方向很清楚。Claude Code 不满足于做一个终端聊天 agent，它在补本机 supervisor、session roster、worktree isolation、hook-based continuation、completion evaluator 这些控制面能力。

接下来真正有意思的地方，是这套控制面会不会继续往外扩：跨机器、手机审批、通知、成本面板、跨工具 agent 管理、任务级状态、PR 级验收。Agent 写代码的能力已经够强，下一阶段拼的是谁能把一堆长任务管得住。
> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes kanban 功能详解
author: 程序员BO哥
date: 程序员BO哥程序员BO哥
url: https://mp.weixin.qq.com/s?__biz=MzYzOTE5NjA0NA==&mid=2247483933&idx=1&sn=0b06101c51d70d77696e22bccdf7c6fe&chksm=f1cc942d8ba73d57aa67d07ac0dd3c12e408100433c9dd19487c2d65301868d8d7f9b5c4110b&mpshare=1&scene=24&srcid=0602bbhQeGFdGQ2cZWolhysw&sharer_shareinfo=9d8e2ca71655b6ce7d7352aeabb74d1e&sharer_shareinfo_first=9d8e2ca71655b6ce7d7352aeabb74d1e#rd
---

上次写 [Hermes v0.13 更新：AI Agent 开始像一个小团队一样工作](https://mp.weixin.qq.com/s?__biz=MzYzOTE5NjA0NA==&mid=2247483869&idx=1&sn=04d5bbfbb6b19d678d78dda169a7784a&scene=21#wechat_redirect)Hermes v0.13 的时候，我把重点放在一个判断上：AI Agent 正在从“能回答问题”，走向“能把事情持续做完”。

这次单独拆 Hermes Kanban。

因为它看起来像一个任务看板，但真正值得关注的地方不在“看板”两个字，而在它背后的工作方式：把 Agent 的任务、状态、失败、交接和人工介入，从聊天窗口里拿出来，放到一个持久的任务系统里。

## Kanban 不是普通待办清单

普通待办清单解决的是“我要做什么”。

Hermes Kanban 解决的是另一个问题：当一件事不是一步完成，而是要拆分、排队、执行、失败、重试、交接、等待人工判断时，Agent 应该怎么继续往前推进？

官方文档里说得很直接：Hermes Kanban 是一个 durable task board，底层是 SQLite。每个任务都是 `~/.hermes/kanban.db` 里的一行。每次交接、评论、运行记录，也都会留下来。

这就和普通聊天不一样了。

在聊天里，你让 Agent 去做一件事，结果通常只存在当前上下文里。上下文压缩了、会话断了、子任务失败了，很多过程信息就散了。

但在 Kanban 里，任务是一个独立对象。它有标题、正文、负责人、状态、依赖、评论、运行历史和工作目录。换句话说，Agent 不是“记得自己刚才说过什么”，而是可以重新读任务板，知道现在该做什么。

## 它的核心对象其实很工程化

Hermes Kanban 里有几个关键对象。

第一个是 **Board**。你可以把它理解成一个独立任务队列。默认有一个 `default` board，也可以为不同项目创建不同 board。不同 board 有自己的数据库、工作区和日志，任务之间不会混在一起。

第二个是 **Task**。任务有一套明确状态：`triage`、`todo`、`ready`、`running`、`blocked`、`done`、`archived`。这点很重要，因为 Agent 的工作不再是“正在做”这么模糊，而是能明确知道：这个任务还没细化、等待依赖、可以执行、正在执行、卡住了，还是已经完成。

第三个是 **Link**。任务之间可以有父子依赖。比如先设计接口，再实现接口，最后写测试。下游任务不会在上游完成前被随便启动。

第四个是 **Comment**。它不是普通备注，而是人和 Agent 之间、Agent 和 Agent 之间的交接协议。人可以在任务卡上补充约束；worker 也可以把进度、决策、结果写进去。下一次被派发的 worker 会读到这些内容。

第五个是 **Workspace**。任务可以在临时目录里跑，也可以绑定到一个绝对路径目录，或者使用 git worktree。对代码任务来说，这意味着每个任务可以有自己的隔离工作区，减少互相覆盖的风险。

这些设计都很工程化。它更像是把项目里的 issue、队列、日志、状态机搬进了 Agent 系统。

## dispatcher 才是真正的调度员

Kanban 里还有一个重要角色：dispatcher。

人可以用 CLI、slash command 或 dashboard 创建任务，比如：

```
hermes kanban create "整理最近 10 篇文章数据" --assignee analyst
hermes kanban list
hermes kanban show <task_id>
```

但任务创建之后，不是当前聊天窗口一直盯着它。

dispatcher 会周期性扫描任务板。默认情况下，它运行在 gateway 里，大约每 60 秒处理一轮：把依赖完成的任务从 `todo` 推到 `ready`，认领 ready 任务，启动对应 profile 的 worker，回收异常 worker，处理超时和失败。

这里的关键词是 **profile**。

一个任务可以分配给某个 Hermes profile。dispatcher 启动 worker 时，会带上 `HERMES_KANBAN_TASK`、`HERMES_KANBAN_BOARD`、`HERMES_KANBAN_WORKSPACE` 等环境变量。worker 不是靠猜，也不是靠读整个聊天记录，而是通过 `kanban_show` 读取当前任务，再用 `kanban_complete` 或 `kanban_block` 结束这一轮。

如果 worker 做完了，就写 summary 和 metadata，把任务转成 `done`。如果卡住了，就 block，留下原因，等人补充信息或解除阻塞。

这就是它比普通自动化强的地方：失败不是直接消失，而是进入一个可处理状态。

## Agent 用工具，人用命令

这里有个细节很容易误解。

人操作 Kanban，主要用 CLI、dashboard 或 `/kanban`：

```
hermes kanban init
hermes kanban create "写一篇 Hermes Kanban 文章" --assignee writer
hermes kanban stats
hermes kanban runs <task_id>
```

但 worker Agent 不应该在里面再 shell 出去跑 `hermes kanban show`。官方文档明确区分了两个入口：人和脚本用 CLI；模型通过专门的 `kanban_*` 工具操作任务板。

常见工具包括：`kanban_show`、`kanban_complete`、`kanban_block`、`kanban_heartbeat`、`kanban_comment`、`kanban_create`、`kanban_link`、`kanban_unblock`。

这个区分很关键。

因为对 Agent 来说，工具调用是结构化的，能直接写入 Kanban 数据层；对人来说，CLI 和 dashboard 更方便观察、创建和干预。两边都连到同一个 SQLite board，所以状态不会分裂。

## 它和 delegate\_task 的区别

如果只看表面，Kanban 和 `delegate_task` 都像是“让另一个 Agent 干活”。

但它们不是一个层级的东西。

`delegate_task` 更像一次函数调用：当前 Agent 找一个子 Agent 帮忙，等它返回结果，再继续当前任务。它适合短任务，比如查一段资料、审一段代码、并行做几个独立小分析。

Kanban 更像一个持久工作队列。任务创建后，父会话不必一直等着。任务可以被不同 profile 接手，可以等待人工评论，可以因为失败进入 blocked，可以被重新 unblock 后继续跑。历史记录也不会因为上下文压缩而丢掉。

所以我的理解是：

短任务、一次性、结果要马上回到当前上下文，用 `delegate_task`。

长任务、多阶段、可能失败、需要人介入或后续追踪，用 Kanban。

## 哪些场景值得使用

不是所有任务都值得放进 Kanban。

如果只是问一个命令、改一个小错、总结一段文字，直接聊天就够了。强行建任务卡，只会增加管理成本。

Kanban 更适合这几类事：工程流水线，比如需求拆解、实现、测试、review；长期运营任务，比如定时采集数据、每天整理素材、每周生成复盘；研究和写作流程，比如来源收集、可信度判断、初稿和审稿；以及容易失败的自动化任务，比如涉及超时、登录态、环境依赖和人工确认的流程。

这些任务的共同点是：不能假装一次就能跑完。失败后能 block，能留下原因，能重新接上，反而更可靠。

## 这套东西真正解决的是“组织问题”

很多人看 Agent，容易只看模型能力：它聪不聪明，代码写得好不好，回答准不准。

但真实工作里，单点聪明只是第一层。

更难的是组织问题：谁负责什么，做到了哪里，失败了怎么办，下一步谁接，哪些地方必须人判断，结果怎么留下来。

Hermes Kanban 的价值就在这里。它不是让 Agent 瞬间变成一个完美团队，而是先给 Agent 一个团队工作的外骨骼：任务板、状态机、调度器、worker、日志、评论和审计记录。

这听起来不性感，但做工程的人都知道，真正能跑起来的系统，靠的往往不是一次漂亮输出，而是边界、状态、重试、日志和验收。

如果你只是把 AI 当聊天工具，Kanban 可能显得重。

但如果你已经开始让 AI 处理真实工作流，它就很值得关注。

它提醒我们一件事：Agent 的下一步，不只是更聪明，而是更会协作、更能恢复、更能把一件事从开始推到结束。

这也是我最近越来越认同的方向：使用 AI 做真实工作，不要只追新模型和新功能，更要学会把任务组织方式升级。工具会变，但“如何让复杂工作持续推进”这个问题，会一直存在。

## 参考来源

1. 1. NousResearch / Hermes Agent：`RELEASE_v0.13.0.md`
2. 2. Hermes Agent 官方文档：Kanban (Multi-Agent Board)
3. 3. Hermes Agent 官方文档：Kanban tutorial
4. 4. Hermes Agent 官方文档：Kanban worker lanes
5. 5. 本机命令实测：`hermes kanban --help`、`hermes kanban create --help`、`hermes kanban list --help`

如果你也在尝试把 AI Agent 接进真实工作流，可以留言说说：你现在最想交给 Agent 的长任务是什么？我后面会继续分享怎样把 AI 工具、Skill、MCP 和工作流沉淀成自己的长期能力。
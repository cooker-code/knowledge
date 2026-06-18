---
title: Anthropic 亲自下场教 Claude Code：最值钱的技能是 rewind
author: stan的空空Q间
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMDg3NjY5NQ==&mid=2247484868&idx=1&sn=5ae2716a3e169d19cd848546ad114494&chksm=9aa12c477b434b1b82d66f766b216701f9e5be10e333b2523a95c94ee463b56e7c01670c7b81&mpshare=1&scene=24&srcid=0416RoV6nhH2929fafcZXIOJ&sharer_shareinfo=47ad2b12d4bffd3a7c795c5f917116be&sharer_shareinfo_first=47ad2b12d4bffd3a7c795c5f917116be#rd
---

Thariq 原帖：3 小时 1.7K 赞、16 万浏览

刚刚，Claude Code 团队的 Thariq（@trq212）发了一篇文章——不是发布新功能，而是发了一份"使用手册"。

3 小时拿下 1500+ 点赞、150 多次转发，评论区被 "rewind feels immediately useful" 这种留言刷屏。

注意，这种"教你怎么用"的长帖，Anthropic 官方很少自己下场写。

01

## 罕见的下场

平时讲 Claude Code 怎么用的，都是社区 KOL、第三方教程作者、油管 up 主。官方只管发布新功能，不管你怎么用。

这次反过来了。

Thariq 一上来交代得很清楚：**这篇是被客户对话逼出来的**。

> "What came up again and again in these calls is that there is a lot of variance in how you might manage your sessions, especially with our new update to 1 million context in Claude Code."

翻译过来就是：1M context 上线之后，他们发现一件让产品团队挠头的事——同样一台 Claude Code，不同人用法差到不像同一个产品。有人开一个 session 用一整天，有人每条 prompt 都开新 session，有人疯狂用 compact，有人完全不知道有 rewind 这回事。

1M 把这种差异放大了。

之前 200K 的时候，session 太长会自动撞墙，谁用得糙、谁用得精，最后都得 compact 或 clear，差距没那么离谱。1M 一来，糙的人可以把整本垃圾塞进去几小时不爆，精的人能在同样时间里把 5 个独立任务跑完——同样一份订阅，产出差出来一个量级。

所以官方坐不住了，下场写手册。

02

## 五岔路口

文章的核心其实就一张图：**每次 Claude 停下，你都站在一个 5 岔路口**。

每条消息发完后的五岔路口（图：Anthropic 官方）

Thariq 把这五个动作排成并列：

> "While the most natural is just to continue, the other four options exist to help manage your context."

换句话说：直接 continue 是默认动作，剩下四个全都是为了**管理上下文**而存在的。

• **Continue** —— 继续在同一个 session 里发下一条。最舒服，但也最贵

• **/rewind**（双击 esc）—— 跳回某条历史消息，从那里重发。中间的对话都丢掉

• **/clear** —— 关掉这个 session，开个全新的，把你需要带过去的东西自己写一段 brief

• **Compact** —— 让 Claude 自己把对话总结成一段，然后在总结之上继续

• **Subagents** —— 把下一块工作甩给一个有自己干净 context 的子 agent，只把它的结论拿回来

读完这五条，你会发现一件反直觉的事：**这不是五个并列的功能，这是一个决策树**。

Thariq 之所以能写这篇文章，正是因为他每天都在心里跑这个决策树。普通用户每天跑的是另一个决策树："要不要重新打开 Claude Code"。

差别就在这。

03

## Rewind 上位

整篇文章里，最不藏着掖着的一句话是这条：

> "If I had to pick one habit that signals good context management, it's rewind."

如果让我挑一个习惯来判断一个人会不会管 context，那就是 rewind。

这句话说得很重。重到让人想问：为什么是 rewind，而不是 compact，不是 subagent？

Thariq 给了一个具体场景。

Claude 读了 5 个文件，试了一个方案，没成。你的本能反应是接着发一条："那个不行，换 X 试试。"

听起来很合理对吧？但 Thariq 说这是错的。

Correcting 是接着说"再试 X"，Rewinding 是 esc esc 把失败那段从 context 里切掉（图：Anthropic 官方）

正确动作是：**双击 esc**，跳回到 Claude 刚读完那 5 个文件、还没动手的那个时刻，然后把你刚学到的教训直接写进 prompt：

> "别用方案 A，foo 模块根本没暴露那个接口——直接走 B。"

为什么这样更好？因为你刚才那条失败的尝试、它读过的中间结果、它走错的死胡同——全都还堆在 context 里。Claude 接下来每生成一个 token，都得多扛着这堆垃圾过一遍 attention。

**rewind 不是回滚，是减法**。

你把"刚才那次失败"这一整段，从 Claude 的记忆里切掉了。它接下来面对的是一个干净的状态：5 个文件 + 你的新指令。

Thariq 还提了一个进阶玩法："summarize from here"——让 Claude 在 rewind 前先写一段"我刚才试了什么、哪里卡住、学到了什么"，相当于给"过去的自己"留一封信，然后再 rewind 回去把这封信塞进新 prompt。

Claude 给 Claude 写交接文档。听起来荒诞，但完全说得通。

评论区一个叫 mike 的用户的反应很代表：

> "rewind feels immediately useful."

注意这个 immediately。不是"我去试试看"，是"我立刻就能想到该用在哪"。一个好功能的标志，就是用户读到说明的那一刻就在脑子里 replay 自己昨天的一次失败。

04

## compact vs clear

接下来 Thariq 处理另一对很容易搞混的动作：**compact 和 clear**。

表面上看它们都是"session 太长了，瘦个身"。但行为完全不同。

compact 是把决定权交给模型，clear 是把决定权握在自己手里（图：Anthropic 官方）

**compact** 是把对话甩给 Claude，让它自己总结，然后用总结替换掉原始 history。优点是省事。缺点是有损——你赌 Claude 知道哪些信息以后还要用。

**clear** 是你自己写一段 brief："我们在重构 auth middleware，约束是 X，相关文件是 A 和 B，方案 Y 已经被排除了。" 然后开新 session。

Thariq 的判决性总结是：

> "It's more work, but the resulting context is what you decided was relevant."

clear 更累，但留下来的 context 是**你自己决定**的。

紧接着他点了一个特别毒的细节：**bad compact 是怎么发生的**。

场景是这样的：你刚结束一段 debug，autocompact 被触发，Claude 把整段调试过程总结成一行。然后你下一条消息是："顺便把 bar.ts 里那个我们刚才看到的 warning 也修一下。"

Boom。bar.ts 的 warning 在 debug 阶段是个边角细节，被 compact 顺手删掉了。Claude 一脸懵：什么 warning？

Thariq 的解释是：

> "bad compacts can happen when the model can't predict the direction your work is going."

这句话才是这一节真正的核心：**compact 的质量，取决于模型能不能猜到你下一步想干嘛**。

而最讽刺的是——compact 那一刻，恰好是 context 最满、模型最不聪明的那一刻。你让一个最累的 Claude 去赌你接下来想要什么，然后责怪它赌错了。

1M context 给的真正礼物，不是"现在你可以塞更多东西"，是"现在你有时间在 context 还没满之前**主动 compact**"——而且主动 compact 的时候你可以告诉它你想往哪走：

```
/compact 重点保留 auth 重构相关的内容，把 test 调试那段全部丢掉
```

这才是会用的人和不会用的人的差距。

05

## Subagent 的判断题

Subagent 这一节最短，但也最干净。

Thariq 直接给了一个判断题：

> "The mental test we use: will I need this tool output again, or just the conclusion?"

我以后还会用到这次工具调用的原始输出吗？还是我只要它的结论就行？

如果答案是"只要结论"——把这个活儿甩给 subagent。它在自己干净的 context 里跑，干完只把一段总结塞回来。原始的中间过程一个 token 都不进你的主 context。

20 次文件读 + 12 次 grep + 3 次走死路，最后只有一行结论穿越回主 context（图：Anthropic 官方）

他给了三个例句，每一句都很具体：

> "Spin up a subagent to verify the result of this work based on the following spec file."
>
> "Spin off a subagent to read through this other codebase and summarize how it implemented the auth flow, then implement it yourself in the same way."
>
> "Spin off a subagent to write the docs on this feature based on my git changes."

注意这三句的共同点：**都是命令式**。都是你主动把活儿丢出去，不是等 Claude 自己决定要不要 spawn。

我们之前那篇关于 thin harness fat skills 的文章里就讲过：Claude Code 这一代 harness 设计的核心，是**用户对 context 边界的主动控制权**。Subagent 不是后台优化，是你手里的一把刀。

会用的人把它当成"读一遍这个仓库然后告诉我结论"的廉价雇佣兵。不会用的人继续在主 session 里让 Claude 自己 grep。

06

## 1M 不是让你一把梭

文章读到这里，你可能已经在想：**那 1M context 到底是干嘛的？**

如果 rewind 是核心、clear 是干净、compact 是赌博、subagent 是减负——那 1M 难道是反派？

评论区有人就这么问了。Tomáš（@Hrdlickas）的反问很直接：

> "When you're constrained so hard on gpus, why not be more assertive that having a 1m context window does not mean you should use all of it? Users get better answers and you save on compute. Seems like a clear win-win?"

你们 GPU 都那么紧张了，为什么不大声告诉用户 1M 不等于让你全塞？省算力，效果还更好，双赢。

这个问题点到一个矛盾：Anthropic 上个月刚把 1M 当大新闻发，这个月又写文章劝你别一把梭。

但仔细看 Thariq 这篇就会发现：**1M 给的不是空间，是时间窗口**。

之前 200K，一个稍微复杂点的任务，跑两小时就撞墙，你被迫 compact——而且是被迫在最不合适的时候 compact。1M 之后，你有时间在 context 还很轻的时候**主动**做这件事，可以选 compact 还是 clear，可以决定保留哪段、丢哪段，可以让 Claude 在状态最好的时候帮你做总结。

另一个用户 jaimerodriguez 的提问也很专业：

> "With 200K, I learn to self-manage at 50-60%. What would you recommend for the models with 1M?"

200K 时代，老用户已经形成了"用到 50-60% 就该手动收一下" 的肌肉记忆。1M 之后，这条线在哪？

Thariq 没在帖子里回，但整篇文章其实就是在回答这个问题：**那条线不是固定百分比，是你对"任务边界"的判断**。新任务就开新 session。相关任务就 rewind 到分叉点。同任务但走偏了就主动 compact 并明确指方向。

1M context 不是让你停止管理 context，是让管理 context 这件事**第一次有了从容的时间**。

07

## 从喊它才动到读懂你

文章结尾，Thariq 写了一句很轻的话，但分量不轻：

> "Over time we expect that Claude will help you handle this itself, but for now this is one of the ways you can guide Claude's output."

未来 Claude 自己会处理这些。但现在还需要你来引导。

五个动作的速查表：什么场景按哪个键（图：Anthropic 官方）

这句话其实在交代一个 paradigm shift 的过渡阶段。

之前我们写过的 Routines、Skills、Plan 模式，本质都是同一件事的不同切面——Claude Code 正在从"你按一下、它动一下"的工具，变成"它自己知道该往哪走"的同事。session 管理是这条路上最后一道还没被完全自动化的关卡。

但在自动化到来之前，会用的人和不会用的人之间，差距在被这条手册放大。

读完 Thariq 这篇，最值钱的不是某个具体技巧。是那种"我每次按回车之前都该停 0.5 秒，问问自己接下来该走五条路的哪一条"的肌肉记忆。

继续，回退，清空，压缩，分身。

你每次按下回车之前的 0.5 秒，决定了同一份 200 美元的订阅，能跑出 5 倍还是 1 倍的产出。

· · · · ·

· · · · ·

**相关链接：**

• Thariq 原帖：https://x.com/trq212/status/2044548257058328723

• Claude Code 官方产品页：https://claude.com/product/claude-code

• Berryxia 中文转发：https://x.com/berryxia/status/2044552563916476591

• 我们之前写的 Routines：之前那篇关于 Claude Code Routines 的文章

• 我们之前写的 Skills 比 Hooks 大：之前那篇关于 thin harness fat skills 的文章
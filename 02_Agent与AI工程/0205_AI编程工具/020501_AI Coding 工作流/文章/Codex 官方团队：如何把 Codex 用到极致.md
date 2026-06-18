---
title: Codex 官方团队：如何把 Codex 用到极致
author: 产品设计频道BackChannel
date: DanielDaniel
url: https://mp.weixin.qq.com/s?__biz=MzI0MjA4ODExMw==&mid=2652550336&idx=1&sn=a49abd266241b7f9e9a06bd3cd875ac0&chksm=f3d4f0e2a9e347eafd523518f9f03d4db3c64bcf4965dba87407bd20bf5b311b63c59a19ab78&mpshare=1&scene=24&srcid=05266Cycsb0Qzh3pZx5L9qC0&sharer_shareinfo=e8daebf31bfb076e5377e4c7576e5f38&sharer_shareinfo_first=e8daebf31bfb076e5377e4c7576e5f38#rd
---

编者注：很多人没有把Codex用到极致，包括我在内，所以Codex团队来拯救我们了，这一篇很受用。

大多数开发者第一次使用 coding agent，目的都很直接：让它读代码库、改代码、生成 diff、跑测试，然后提交一个 pull request。

这仍然是 Codex 最核心的使用场景。但如果只把它理解成「写代码的助手」，就低估了它的变化。

今天，我们在电脑上做的很多事情，本来就已经被代码和工具串起来了：执行 shell 命令、浏览网页、调用 API、导出文档、处理消息、响应事件、触发自动化流程。随着这些能力逐渐接入 Codex，它就不再只是一个狭义的编程助手，而更像是一个能帮你完成电脑工作的系统。

Codex app 让这种变化变得更具体。一个 thread，也就是一条持续的工作会话，可以长期保留上下文、调用工具、展示 artifact（工作产物），并在多轮 prompt 之间持续推进，而不是每次问答结束后都重新开始。

想真正用好 Codex，关键不是单独使用某个功能，而是把这些能力组合起来：

durable threads：能长期保留上下文的工作会话；

voice、steering 和 queuing：让用户在工作过程中继续用语音、打断和排队任务来控制 Codex；

browser、computer-use、MCP servers 和 connectors：让 Codex 不只操作代码库，也能进入浏览器、桌面软件和外部服务；

thread automations 和 Goals：让 Codex 在用户离开后继续推进工作；

side panel：在侧边面板里审查代码、文档、deck、表格、网页和其他工作产物。

# 一、Durable threads：把 Codex 变成长期工作空间

Durable threads指的是长期运行的 Codex 会话。它们能在多次使用之间保留工作上下文。

Pinned threads，也就是置顶会话，是一种把 durable threads 放在手边的方式。它们适合那些会反复发生、需要长期跟进的工作流，比如：

一个 Chief of Staff thread，用来处理日程、消息、待办和优先级；

一个 release thread，用来持续跟进版本发布；

一个 documentation review thread，用来反复审查文档；

一个专门负责外部监控的 thread，用来观察 Slack、邮件、评论或其他反馈来源。

这些 thread 不是一次性的短聊天，而是持久的工作空间。Codex 可以在之后重新回到同一条 thread，记住此前的决策、偏好和工作背景。否则，这些上下文每次都要重新说明。

Pinned-thread shortcuts 让这种用法更顺手。用户可以通过 Command-1 到 Command-9，快速跳转到已经保存的 threads。

# 二、Voice input：先把粗糙想法说出来

Voice input 的价值在于，它能捕捉一个想法还没有被整理成正式文字之前的原始状态。

很多任务一开始并不清楚。用户未必知道该怎么写成精确 prompt，但可以先说出来。Codex 内置 voice input，尤其适合这种场景。

比如，一个自然的语音输入可能是：

> 我记得 Slack 里好像有个叫 Ben 的人提过这件事。  
> 我不记得具体细节了。  
> 你去查一下。

对一个能搜索、收集上下文、整理结果并汇报的 Agent 来说，这通常已经足够。

Voice input 也适合在任务还没有完全成形时，先做两三分钟的 thought dump，也就是把脑子里的想法一口气说出来。用户不需要先组织成严密的文档，Codex 可以从这些不完整、带有犹豫和补充的内容里提取线索。

Transcript 也是类似的材料。一份原始会议 transcript，或者一段口述的 planning note，往往比一个简短 summary 更有价值。因为 transcript 保留了不确定性、强调语气、未完成的想法，以及讨论过程中出现的细节。

这些东西在短 summary 里很容易被压扁，但对 Agent 理解真实任务非常重要。

# 三、Steering 和 queuing：一边工作，一边调整方向

Voice input 变得更有用，是因为它可以和对任务过程的控制结合起来。

这里有两个重要概念：steering 和 queuing。

Steering指的是在 Codex 任务仍在进行时，用户打断它，并在当前步骤完成前给出新的方向。

这适合 Agent 正在走偏、但还没完成任务的场景。比如用户正在让 Codex review 一个网站，同时在 side panel 里看页面。看到问题后，用户可以直接插入新的指令：

把这个做小一点；

这两个元素之间的间距不太对；

这段 copy 是错的。

Steering 改变的是 Codex 此刻正在做什么。它不是等 Codex 做完再返工，而是在任务进行中及时修正方向。

Queuing则不同。它不是打断当前任务，而是把下一个任务排到后面。

比如用户可以说：

> 这件事做完后，把 preview link 发给 Slack 里的 reviewer。

这时 Codex 会先完成当前任务，再执行被排队的新任务。

简单说，steering 负责「现在改方向」，queuing 负责「下一步做什么」。它们让用户不必一次性把所有事情说完，而是可以在 Codex 工作展开的过程中继续参与、继续控制。

# 四、Tools 和 reach：让 Codex 走出代码库

当一个 thread 有了连续上下文，下一个问题就是：Codex 到底能操作哪些东西？

它的能力可以一层层向外扩展。

第一层是 $browser，也就是 Codex app 里的 in-app browser。Codex 可以在 side panel 中检查网页、标注页面、理解网页上的内容，并根据用户反馈进行修改。

第二层是 @chrome。它适合那些依赖用户 Chrome 登录状态的工作，比如访问已经登录的网站、处理某些基于浏览器的流程。

第三层是 @computer。它适合只能通过桌面 GUI 完成的任务。也就是说，有些事情不是网页，也不是代码，而是必须在桌面软件里点选、拖拽、上传或导出，@computer 就是为这类场景准备的。

可以这样理解：

$browser 适合 side panel 里的网页审查；

@chrome 适合依赖用户 Chrome 登录状态的浏览器工作；

@computer 适合那些只能通过桌面图形界面完成的任务。

MCP servers 和 connectors 则把 Codex 接到更大的 workflow 里。

Slack、Gmail 和 Calendar 很重要，因为很多任务最早并不是以代码形式出现的。它们可能先出现在一条 Slack 消息里、一封邮件里，或者一个日程冲突里。只有当 Codex 能看到这些入口，它才能真正参与工作流，而不只是等用户把任务复制粘贴进来。

Skills 则让重复流程可以复用。一个 workflow 一旦被证明有用，就可以被打包成 skill。这样下次 Codex 再做类似事情时，不需要重新学习整套流程，而是可以直接按既定方式运行。

# 五、Work from anywhere：不必一直守在电脑前

Codex mobile app 改变了用户必须坐在桌前的时机。

很多任务适合在 Mac 上启动，因为文件、本地环境、权限和配置都已经在那里。但任务一旦开始，用户未必需要一直坐在电脑前等结果。

Codex 可以在 Mac 上继续运行，用户则可以通过手机查看进展、回答问题、批准下一步，或者在回到桌前之前重新调整 thread 的方向。

这在很多小场景里很有用。比如 Codex 正在跑一个较长的任务，用户可以先离开桌面；如果 Codex 中途需要确认，用户可以在外面用手机回复。这样，本地环境不变，工作也不会因为用户离开电脑而停住。

# 六、Automations：让 Codex 定期回来继续工作

Automations 指的是按计划运行的 Codex 工作。

这里有两类用法。

一种是 scheduled automation。它适合那些每次都应该从某个 workspace 重新开始的重复任务，比如 daily report，或者定期检查一个 repository。

另一种是 thread automation。它适合那些应该回到同一条 active conversation、带着已有上下文继续推进的任务。

Thread automations可以理解为给某个 Codex thread 设置的定时唤醒机制。它会像 heartbeat 一样，每隔一段时间回到同一条 thread，检查事情有没有变化，并继续推进。

Pinned threads 很有用，但它们仍然要等用户主动回来。Thread automation 则可以每隔几分钟或几小时检查一次某件事，在满足某个条件之前持续运行，并根据情况调整频率。

比如，一个 Chief of Staff thread 可以每 30 分钟运行一次：

> 每 30 分钟检查 Slack 和 Gmail，找出还没有回复、但需要我关注的消息。  
> 帮我判断哪些事情最重要。  
> 如果有人问我问题，请尽可能深入地研究答案，并替我起草回复，但不要发送。

当用户回来时，最耗时间的上下文收集工作往往已经完成了。人仍然保留最终决定权，决定哪些内容可以发送，哪些内容还要修改。

Thread automations 也适合处理反馈循环。

它可以定期查看 pull request comments、Google Docs comments 或 Slack replies，在用户离开时继续推动周边工作。

比如一个 animation workflow：reviewer 在 Slack 里分享了一段视频，并提出修改意见。Thread automation 可以定期检查这个 Slack thread。只要有新评论出现，Codex 就可以渲染一个更新版本，并在同一个 Slack thread 里回复，tag 那位 reviewer。

如果某个 integration 不能完成最后的上传步骤，desktop automation 还可以接手，通过 GUI 完成上传。

这个 loop 横跨了 Slack 里的反馈、代码库里的渲染流程，以及桌面自动化里的最终上传。Codex 不再只是写代码，而是在多个工作界面之间把事情推进完。

# 七、Goals：给长期任务一个明确终点

Goals 最有价值的地方在于，它们让 Codex 可以持续朝一个真实的 finish line 推进。

一个弱 goal 可能是：

> 实现这个 Markdown 文件里的计划。

这个目标太模糊。Codex 可能会执行一些步骤，但很难判断什么时候才算真正完成。

一个更强的 goal 需要有可衡量的 success criterion，也就是成功标准。

比如，一位工程师想把一个 internal tool 从 Python 迁移到 Rust。他可以先创建新的目录，定义 goal，并明确说明 finish line：

> 新实现只有在 unit tests 全部通过后才算完成。

这样，Codex 不只是「努力迁移」，而是有了一个可以验证的终点。

Goal 本质上结合了两件事：持续执行和 verifier。Verifier 是验证器，用来判断任务有没有变好、有没有接近完成。

用户需要定义：

想要达成的 outcome；

任务应该在什么条件下停止；

哪个 signal 能说明 Codex 正在接近目标。

有用的 verifiers 包括：

test suite，也就是测试套件；

benchmark，用来比较性能或质量的基准；

bug reproduction，用来复现并验证 bug 是否修复；

validation matrix，也就是验证矩阵；

必须持续通过的 end-to-end workflow，也就是端到端流程。

Ambition 很重要，但如果没有 verification，它就只是愿望。只有当目标能被验证，Codex 才能持续判断自己是不是在接近完成。

# 八、Side panel：把产物放在对话旁边审查

Side panel 的作用，是把工作产物放在产生它的对话旁边。

过去，一个 Agent 生成了代码、文档、PDF 或 deck 后，用户往往要导出文件、切换软件、单独打开、再把修改意见带回来。这个过程会打断上下文。

Side panel 则让 review 留在同一个工作循环里。

Codex 生成的结果可能是代码，也可能是 deck、PDF、browser page、table，或者其他 artifact。Side panel 让用户可以直接在旁边检查它们。

它尤其适合四类工作：

inspect artifacts：检查工作产物；

annotate what needs to change：标注哪里需要修改；

operate web surfaces：操作网页界面；

review changes：审查修改结果。

Side panel 可以就地查看 Markdown、spreadsheets、data tables、documents 和 slides。用户可以检查、标注和修改这些 artifact，而不必脱离当前 thread。

Annotations：直接在产物上标注问题

Deck 或 PDF 可以一直打开在生成它的 thread 旁边。用户看到哪里不对，可以直接标注，Codex 再根据标注修复。

这种方式比「先导出，再写一段反馈，再重新上传」更自然。评论和修改意见不会散落到其他地方，而是留在同一个工作 loop 里。

Sheets in Codex：让网页既是输出，也是控制界面

Codex 的 in-app browser 可以检查一个已经渲染出来的页面，控制页面，并直接响应用户在页面上的 annotations。

这意味着，网页不只是 Codex 的输出结果，也可以成为控制界面。

Codex 可以构建一个 artifact，在 side panel 中打开它，检查它，调试它，再继续在同一个对象上迭代。用户对页面或 artifact 的评论，也会留在当前工作流中，不会变成一次额外的 handoff。

这些 surfaces 尤其适合：

index.html：用于轻量级 static artifacts，不需要服务器；

Storybook：用于 UI review；

Remotion Studio：用于 programmatic animation；

browser-based slide decks：用于基于浏览器的演示文稿；

data apps：用于数据分析工作流。

一个单独的 index.html 文件，就可以成为一个持久的 interactive artifact。它不需要服务器，也能作为一个可审查、可迭代的工作产物。

Thread automations 还可以定期刷新 static artifacts。这样用户回到 thread 时，里面已经有更新后的结果在等着。

# 九、Shared memory：把重要上下文写到会话之外

Long-running threads 会变得更有用，如果它们能在单个 conversation 之外共享 memory。

Shared memory指的是存储在某条 thread 之外的 durable context。它让未来的工作可以从明确、可审查的上下文继续开始，而不是只依赖聊天记录。

一种实用模式，是把 persistent threads 锚定在一个 Obsidian vault 里。

在实践中，这就是一个由 plain files 组成的文件夹。它容易查看、编辑、移动，也适合长期保存。团队可以把这个文件夹放在 cloud storage、Git、Dropbox、Google Drive，或者任何适合自己 workflow 的同步系统里。

一个 vault 可能长这样：

vault/

├── TODO.md

├── people/

├── projects/

├── agent/

└── notes/

在顶层，AGENTS.md 可以定义 Codex 应该如何更新这个 workspace。比如，当 Codex 逐渐了解更多人、项目、决策和 open loops 时，它应该把哪些内容写下来，写到哪里，以及什么时候不应该乱改。

不要照搬某一种 vault 结构。更重要的是教会 Agent 三件事：

durable context 应该放在哪里；

哪些 context 值得长期保存；

什么时候不应该制造无意义的文件变动。

一个实用的 AGENTS.md 可能会这样写：

- Treat ~/vault as durable work memory.

- Prefer canonical notes over note sprawl.

- Route TODOs, people, projects, daily summaries, and scratch notes explicitly.

- Preserve decisions, blockers, owners, dates, and useful links.

- If nothing meaningful changed, do not churn the vault.

意思是：

把 ~/vault 当作长期工作记忆；

优先维护权威笔记，不要制造大量零散笔记；

明确区分 TODO、人物、项目、每日总结和临时笔记；

保存决策、阻塞点、负责人、日期和有用链接；

如果没有真正有意义的变化，就不要无谓修改 vault。

Repository 保存代码。Vault 保存滚动中的工作上下文：涉及哪些人，发生了什么变化，什么事情被阻塞，谁负责下一步，哪些内容需要 follow-up，以及那些原本会在 session 之间消失的信息。

重要上下文不应该只存在于 conversation transcript 里。它应该被写到一个未来 thread 也能继续读取的地方。

Codex 在 Settings > Personalization > Memories 里也有 first-party memory features。它们可以作为本地 recall layer，用来记住偏好、重复 workflow 和已知问题。

但这些 memory features 是对显式 written context 的补充，而不是替代。Chronicle 也在朝同一个方向发展：帮助 Codex 从最近的 screen context 中建立 memory。

# 十、从代码开始，但不止于代码

Codex 仍然从代码开始。代码依然是它的核心使用场景。

但现在，围绕代码的大量工作也逐渐可以通过同一套系统完成：MCP servers、browser surfaces、desktop controls、thread automations，以及可以在 side panel 中审查的 artifacts。

这改变了用户和 Agent 之间的控制模型。

Steering 让用户可以打断正在进行的工作。Queuing 让用户可以把下一步任务排进去。Thread automations 让一个 thread 在用户离开时仍然保持活跃。Goals 则提供一个具体的 finish line，让 Codex 可以持续朝目标推进。

Codex 现在可以把一个 workflow 从 instruction 推进到 execution，再推进到 artifact review。即使这项工作已经离开代码库，进入浏览器、Slack、文档、桌面软件或其他工作界面，它也仍然可以在同一个系统里继续完成。
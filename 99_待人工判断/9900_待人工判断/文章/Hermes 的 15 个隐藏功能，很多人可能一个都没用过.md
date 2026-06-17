---
title: Hermes 的 15 个隐藏功能，很多人可能一个都没用过
author: Vion的AI手记
date: VionVion
url: https://mp.weixin.qq.com/s?__biz=MzAxNDk3MTE4NQ==&mid=2247484464&idx=1&sn=aa1521d6d0e3becc9c41bbe0e7a742a0&chksm=9a6f3a998b1f46cb568880913167859ceafe894e837cdb4726edc9145f29e7ccb5b53211a707&mpshare=1&scene=24&srcid=0430pXSmSPlqAESQJyn6fjuE&sharer_shareinfo=3e3abdf1fc3127bf7d9f9eea8f920c0b&sharer_shareinfo_first=3e3abdf1fc3127bf7d9f9eea8f920c0b#rd
---

Sharbel 发了一条关于 Hermes 的长帖，列了 15 个很多人没碰过的功能。

我看完之后，第一反应是：我可能真的把 Hermes 用浅了。

Hermes 真正值得看的地方，在于它能不能把你的偏好、项目、工具、平台和重复流程慢慢串起来。

如果只把它当聊天框，很多能力确实一个都用不上。

## 先说我为什么想写这篇

我是在看到 Sharbel（FounderFunnel.com 联合创始人）的文章后，才开始重新看 Hermes 这些功能的。一方面我自己就是 Hermes 的重度使用者，另一方面也想把这份清单分享给同样在用 Hermes 的人。他在文章里列了 15 个功能，包括 SOUL.md、MEMORY.md、USER.md、/rollback、/model、gateway、cron、skills 等。

我同时查了 Hermes 的公开文档。官方文档能确认的部分，包括 SOUL.md、persistent memory、slash commands、skills、messaging gateway、cron、checkpoints 和 provider 配置。

这样写比较啰嗦，但有必要。

AI 工具文章最容易出的问题，就是把一个人帖子里的使用经验，直接写成官方事实。看起来很兴奋，最后读者照着做发现版本不一样，反而更浪费时间。

## 1. SOUL.md：先把 agent 的性格定下｜属实

很多人每次打开 AI，第一步都是重新写人设。

你是一个资深产品经理。

你说话不要油腻。

你不要写得像 AI。

写一次还好。每天写，真的很烦。

SOUL.md 解决的就是这个问题。它是 Hermes 启动时会读取的身份文件，可以用来定义这个 agent 的长期性格、语气、价值判断和边界。

比如你希望它始终用克制的中文表达，不要动不动上价值，不要默认写营销腔，就可以写进 SOUL.md。

它适合放长期稳定的东西。

不要把一次性任务写进去。比如「今天帮我分析这篇文章」就不适合。适合写的是「回答时先讲结论」「不确定就标注不确定」「不要编造来源」「写中文时少用二元反转句式」。

简单来说，SOUL.md 不是 prompt 草稿箱。

它更像 agent 的底层说明书。

## 2. /personality：临时切换说话方式｜属实

Sharbel 提到的 /personality，可以理解成在同一个 agent 里切换不同角色。

比如你平时希望 Hermes 是一个冷静的执行助手，但写内容时希望它变成编辑；做产品判断时希望它变成用户研究员；检查代码时又希望它变成严格的 reviewer。

如果每次都重新写一大段角色设定，时间会被浪费在重复说明上。

/personality 的价值，就是把常用身份提前存好，需要时直接切换。

这个功能适合两类人。

一类是经常在不同任务之间切换的人，比如白天写代码，晚上写内容，周末做产品规划。

另一类是团队使用者，不同场景需要不同语气和权限边界。

注意一点：角色设定不应该写得太玄。不要写「你是全宇宙最强 AI 战略专家」。更有效的写法，是明确它该怎么判断、怎么说话、哪些事不能做。

## 3. MEMORY.md + USER.md：让它记住项目，也记住你｜属实

Hermes 官方文档里有 persistent memory。Sharbel 重点提到 MEMORY.md 和 USER.md，我觉得这是普通用户最应该先配置的部分。

MEMORY.md 记录项目事实，比如项目目录、常用命令、线上限制、哪些文件不能乱动。

USER.md 记录用户偏好，比如你常用什么语言、喜欢直接结论还是长解释、讨厌什么表达、风险偏好是什么。

这两个文件的价值，是减少反复解释。

但不要把 MEMORY.md 当垃圾桶。真正该写进去的，是长期有效、会反复影响判断的事实。

## 4. /insights：回看自己到底怎么用 Hermes｜属实

Sharbel 提到 /insights [days]，用来查看一段时间里的使用情况。

它能帮你看清过去一段时间里，哪些项目最耗 token，哪些 provider 花费高，哪些任务最容易卡住。

这个功能适合做复盘。某个项目长期吃上下文，可能该补文档；某类问题反复出现，可能该写进 MEMORY.md，或者沉淀成 skill。

## 5. /snapshot：折腾配置前先拍一张快照｜转述

Sharbel 对 /snapshot 的描述很直白：做危险改动前，先把 Hermes 的配置和状态保存下来。

它适合用在改 SOUL.md、MEMORY.md、USER.md，或者测试新 provider、gateway、skill 之前。

很多配置问题最麻烦的地方，是改完以后不知道哪一步错了。/snapshot 的价值，就是给自己留一个能回去的点。

## 6. /branch 或 /fork：保留当前上下文，开一条支线试错｜转述

Sharbel 把 /branch 也叫 /fork，说它像对话里的 git。

它适合用来试新方向。比如主线方案已经整理好，但还想试一个更激进的标题、不同的产品方案，或者另一种代码改法。

branch 的好处是，试错不会破坏原来的上下文。试得好就继续，试不好就回到原来的线。

## 7. /rollback：文件改坏了，回到上一个检查点｜属实

/rollback 是最该认真看的功能之一。

Hermes 官方文档里有 checkpoints 和 rollback 机制。文件修改前会保存检查点，出了问题可以恢复。

它适合处理文章结构被改坏、批量替换误伤、代码修改引入问题这类情况。

一个会动手改文件的 agent，必须配撤回机制。否则你不敢放权，最后还是人工盯着它做事。

## 8. /btw：问一个旁支问题，不污染主线｜属实

Sharbel 提到 /btw 是一个临时旁支问题。

它适合问那些和当前任务有关、但不值得写进主上下文的小问题。

比如改文章时顺手问一句标题会不会太像营销号，查技术方案时确认一个库是不是还在维护。

它的价值是借用当前背景快速问一下，但不污染主线。

## 9. /steer 和 /queue：任务跑到一半，也能调整方向｜转述

/steer 和 /queue 解决的是中途控制。

比如任务已经跑起来，才发现它用了生产 API，而不是 staging。直接打断会丢上下文，也会浪费前面做好的材料。

/steer 更像「中途提醒」。当前任务还在跑，你发现方向不对，就补一句要求，让 Hermes 后面的步骤按新要求来。

/queue 更像「先排队」。当前任务不用停，你可以先把下一件事告诉它，等这轮结束后再接着做。

agent 任务不应该只有「开始」和「杀掉」两个按钮。能中途修正方向，才更接近真实工作。

## 10. /yolo、/fast、/reasoning：速度、风险和思考深度的开关｜转述

这三个开关分别对应风险、速度和思考深度。

/yolo 会减少危险命令审批，只适合非常确定安全、并且愿意承担后果的时候用。

/fast 适合低风险、低复杂度任务，比如改格式、整理列表、生成备选标题。

/reasoning 适合复杂代码、策略判断、事实核查、长链路规划。

重点很简单：不同任务不要用同一套执行强度。该快的时候快，该稳的时候稳。

## 11. /model：不用重启，直接换模型和 provider｜属实

/model [--provider] [--global]，用来切换 Hermes 背后的模型和 provider。Hermes 官方文档也确认，它支持多种 inference provider。

它的价值是保留 agent state，同时换掉底层模型。

复杂分析可以切强模型，重复整理可以切便宜模型，某个 provider 不稳定时也能换到另一个。

重点不在追新。别把所有任务都塞进同一种成本结构里。

## 12. Auxiliary models：让小模型处理小任务｜属实

Auxiliary models 就是辅助模型。

很多人以为 agent 只有一个模型在工作。其实不是。

主模型负责真正需要判断的部分，比如分析问题、写代码、做决策、规划步骤。

但 agent 运行时还有很多「杂活」：压缩上下文、总结历史会话、生成标题、处理图片、做 fallback。

这些事重要，但不一定都需要最强模型来做。

所以更合理的方式，是让主模型处理复杂任务，让辅助模型处理这些范围更窄、重复性更强的任务。

这样既能省成本，也能避免强模型被大量小任务占住。

## 13. Gateway：把 Hermes 接到十几个真实入口｜属实

Sharbel 提到 Hermes 的 17-platform gateway。官方 messaging gateway 文档里，也能看到 Telegram、Discord、Slack、WhatsApp、Signal、Email、SMS、Matrix、Mattermost、Feishu/Lark、WeCom、BlueBubbles、Home Assistant、QQ 等平台。

它的重点不在平台数量，而在让 Hermes 进入真实工作入口。

团队在 Slack，就接 Slack；平时用 Telegram，就接 Telegram；有邮件、短信、企业微信或 Home Assistant 场景，也可以按需接进去。

这样 Hermes 才更像常驻助手，而不是临时聊天框。

## 14. /voice、cron 和 webhook subscriptions：让它能听你说，也能自己等事件｜部分已核

这三个能力可以放在一起看：它们都在减少手动打开聊天框的次数。

/voice 让你用语音交代任务，适合通勤、走路、做家务时记录想法。但按 Sharbel 原文的说法，实时语音目前只在 CLI、Telegram 私信、Discord 频道和 Discord 语音频道里提供。

cron 负责定时任务，比如每周五总结 commits，或者每天整理前一天的任务。

/webhook subscriptions 负责外部事件触发，比如 GitHub、Vercel、Stripe、uptime 有事件时，把信息直接推给你。

这样 Hermes 就不只是「你问它答」，也能在固定时间和外部事件发生时主动工作。

## 15. Skills are slash commands：把重复流程做成命令｜属实

Sharbel 最后讲 skills，我觉得这是整篇最重要的一点。

官方 slash commands 文档确认，installed skills 可以作为动态 slash commands 暴露出来。

也就是说，你可以把一套稳定流程做成命令。比如每周写选题推荐，不用每次重新交代「先查资料、再筛选、再判断中文圈覆盖度、最后按固定格式输出」。

把流程沉淀成 skill，以后输入一个 slash command，它就按固定规则跑。

这才是「会用工具」和「有工作流」的区别。前者每次现场拼装，后者把重复动作存下来。

## 普通用户应该先做什么

如果你现在刚开始用 Hermes，我不建议一口气研究 15 个功能。

那大概率又会变成收藏型学习。

更现实的做法，是拿 30 分钟做一版最小配置。

第一步，打开 SOUL.md，把你每次都要重复说的表达要求写进去。

第二步，整理 USER.md，把你的角色、偏好、常用语言、讨厌的表达方式写清楚。

第三步，整理 MEMORY.md，只放最重要的项目事实和工作约定，不要塞成垃圾桶。

第四步，选一个每周都会重复的任务，尝试做成 skill 或 slash command。

这四步做完，比收藏 50 条隐藏命令更有用。

因为 AI 工具最容易骗人的地方，就是它们看起来都很强。

但真实生活里，强不强不是第一标准。

能不能持续用起来，才是。

原贴链接：https://x.com/sharbel/status/2049158152709382177?s=20
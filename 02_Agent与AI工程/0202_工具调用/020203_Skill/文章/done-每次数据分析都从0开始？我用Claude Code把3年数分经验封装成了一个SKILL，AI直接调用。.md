> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 每次数据分析都从0开始？我用Claude Code把3年数分经验封装成了一个SKILL，AI直接调用。
author: 为鱼肚白
date:
url: https://mp.weixin.qq.com/s?chksm=f96c879ace1b0e8c7ccea7a93832403f9b5ed806c88e6c2b0ef9d2e1470b2350fbcd7e3fb50b&exptype=unsubscribed_card_recommend_article_u2i_mainprocess_coarse_sort_tlfeeds&ranksessionid=1773969433_2&req_id=1773969778518952&mid=2247491075&sn=4bbc16f01514a66d82419d24d7941e9d&idx=1&__biz=MzUxMTgyNDExOA%3D%3D&scene=169&subscene=200&sessionid=1773969410&flutter_pos=26&clicktime=1773969869598&enterid=1773969869598&finder_biz_enter_id=5&key=daf9bdc5abc4e8d08423d3e6bb0260c43c582e2925087fe3a4856bde952183315a902c9c4a9a6423e161253724c21ee5cdf0749a6af49d921cf89b128ea4fe934fe20cad13686384e87c2e2840a68e95de5787f71f198db97586a181139dd45727f91e0c7be037dcb89bc1471afc3e2770282113996e30bf367ebc306536a756&ascene=0&uin=MjAwNTI2NjEyMw%3D%3D&devicetype=OHOS-22&version=f3800f40&abtest_cookie=AAACAA%3D%3D&lang=zh_CN&countrycode=CN&exportkey=n_ChQIAhIQocH6sa3SO9LnE3TeqigkPxLfAQIE97dBBAEAAAAAALQVNdgSiLcAAAAOpnltbLcz9gKNyK89dVj0YH0bzr322pqnP%2BwZDsziz9KzCpTA6yWAAtx66NQAoeI1Xys2xCHpkuU67BQ34hPakgnURHSgFKzDXf%2B1qiT3VhO7lIFvyYXj3CI8vYP%2BMaXKZbqXB5izLYifxkOGLPCNN%2FliML3bQ%2BFIdrvqbVJ7MgnvOoVx3Di4Ryd5DAYgiwEnDTo4LkkJI0OBoxVT5QtG%2FctTu010MNVwy9jhCaJMW43C7Sa0NbxaoYWDaM3zLGkxw2NWQjfGSIA%3D&pass_ticket=lEO8xN2rKMNyn62EIQVPBA33euHPdx2REiZb7ThRzY2Z0M%2FwisXS%2BaUJgGtQC98F&wx_header=3
---

每到月末，打工人头疼的数据复盘就来了。

打开Excel，洗数据、画图表、写结论，三小时过去，交了一份差不多的报告。下个月，同样的事再来一遍。

数据换了，流程还是一模一样。我就在想：AI 能不能直接"继承"我的分析经验，下一次她自己跟着流程做？

最近我发现，还真可以，这个东西叫 SKILL。

# SKILL 是什么？

简单说，SKILL 是一份给 AI 的岗位操作手册。什么场景下，先做什么、再做什么、遇到什么情况怎么处理、最后交付什么。

而多个Skills 就像技能工具箱🧰，里面有各种工具，比如锤子🔨、扳手🔧、螺丝刀🪛，每个工具代表一个领域的专长。

prompt 则是怎么用这些工具的指令，“用锤子把钉子钉进木板里”，AI就会调用锤子这个SKILL 来完成。

比如说，做运营的离不开数据，分析复盘，迭代策略。就可以同时调用数据分析 SKILL 先分析 + 策略思维 SKILL 做收敛 + 演示汇报 SKILL 一起完成任务。

可以说，SKILL是一种新的知识管理方式，可被复制、调用和持续迭代。

# 从经验到 SKILL

**我的数据分析“操作手册”长什么样呢？**

**我一直觉得，数据分析如果不落到“接下来该做什么”，就是白分析。所以，我总结了一套步骤——DIG 探索性分析：**

* D(Description)描述:先看数据什么样

* I(Introspection)内省:找规律、提假设

* G(Goal Setting)目标行动:给出优先级行动

但跟 Claude Code 协作的过程中，她帮我发现了两个盲点：

**一是**“如果不先确认业务背景和目标，分析容易跑偏”，加了 **C(ontext)上下文**，先对齐：这是什么数据、想解决什么问题。

**二是**“如果不做复盘，每次分析都是一次性的”，最后加了 **R(Review)复盘**，每次分析结束，AI 自动沉淀为一份交接文档，下次分析就可以在这次基础上继续。

于是，我把DIG 变成了(C)DIG(R)：

## Context 上下文 & Description 描述

以公众号近一年各渠道、各文章的阅读和关注数据为例，把数据和交接文档.md给她，调用 DIG 数据分析助手 SKILL 。

这个阶段需要交付：业务背景、数据描述的基本情况是否正确。

她准确识别了数据的时间范围、数据结构、核心指标，还帮我识别出可能的异常值。

## Introspection 内省

确认完数据基本情况，还没等我下指令，就主动问我，最关心的分析目标是什么？

并迅速将我模糊的需求拆解为了结构性的分析目标，看她的Thinking：

光看数据还不够，我想看看内容分析，历史文章我都存在NotebookLM，请另一个“锤子”🔨  NotebookLM-SKILL 调用知识库内容。

AI 交叉对比了阅读量、标题字数和内容主题后，发现了一个我自己从来没注意到的规律：

标题在 28-31 字之间，用"强工具+强场景"组合的文章，比如AI 做 PPT，阅读量明显高于其他类型。

长标题承载了更多关键词，增加被推荐的概率，且场景描述更具体，更容易触发共鸣。

## Goal Setting 目标行动

所有分析最终都要回答一个问题：接下来我该怎么做？

于是，她自动生成了一份完整的可视化报告👇

以前这些内容散落在 Excel 里，现在 AI 直接生成了一份完整的可视化报告。

比如这篇文章标题，每次数据分析都从0开始？我用Claude Code把3年数分经验封装成了一个SKILL，AI直接调用。

就是参考她分析出的规律写的，强工具（Claude Code），强场景（数据分析），强痛点（每次从0开始）🤣。

如果不够美观，继续调用网页审美好的SKILL 润色，比如 Anthropic 官方分享的 frontend-design SKILL直接把审美提升了一个度。

不能再说 AI 没有审美了。

## Review 复盘

最后一步，AI 会自动把这次分析沉淀成一份交接文档。

第一次分析，可能要花1小时核对数据、对齐背景和目标，第二次分析，她读取完上次的交接文档，马上就进入正题了。

至少每次分析就不是从零开始，**一次分析一次沉淀，经验会越积越厚**。

# 怎么开始SKILL？

**第一步：先用别人的，积累手感。**

先用 Anthropic 官方、知名开发者出品，先用起来。这么多怎么选呢，我就问Comet浏览器自带的AI，“我要完成某个项目推荐一些 skills ”。

跑完几个项目，你就知道每个 SKILL 擅长什么，交付的怎么样。

效果好的，我需要 AI 帮我解释这个 SKILL是怎么设计，还可以用在哪些场景中，把这个🔨“锤子”物尽其用。

**第二步：自己写一个，先做后总结。**

如何创建一个适配自己需求的 SKILL 呢。先做后总结，跟 AI 一起完成一个任务，然后请她“回顾本轮对话并且梳理成SOP”。

再调用元 SKILL，skill-creator 完善并保存成一个 SKILL。再跑一遍，看看哪里需要优化。总之，能梳理出 SOP 的就能被 SKILL 化。

写在最后

SKILL 的意义不是让 AI 替你思考，而是让你把“怎么思考”这件事标准化了。

她负责记住、执行，把你从重复劳动中解放出来，让你的经验被放大、被复用。

我们负责判断、在复杂决策中权衡感性和理性，做点创造性的事。

**每次调用SKILL，我们和 AI 这个合伙人之间的默契就会越来越深。**

如果你也想试试，可以从你最重复的那件工作开始。你最想让 AI "继承"你哪项技能？

关注她，一起用 AI 做点啥吧～

推荐阅读📖

[年底了，年度数据复盘怎么办？幸好还有 AI 在。](https://mp.weixin.qq.com/s?__biz=MzUxMTgyNDExOA==&mid=2247489922&idx=1&sn=471c7d36d426bd5cf3fe833e3f4b3cc9&scene=21#wechat_redirect)

[用 AI 先革自己的命，数据分析师是时候走出数据“洞穴”了](https://mp.weixin.qq.com/s?__biz=MzUxMTgyNDExOA==&mid=2247487110&idx=1&sn=890d9810e3f85a58d8a80df32f1282c3&scene=21#wechat_redirect)

[AI 做 PPT：终于可以从从容容、游刃有余...](https://mp.weixin.qq.com/s?__biz=MzUxMTgyNDExOA==&mid=2247489519&idx=1&sn=bc6bba02370924739c006aabd2d3043b&scene=21#wechat_redirect)

[当你重新爱上自己的衣橱](https://mp.weixin.qq.com/s?__biz=MzUxMTgyNDExOA==&mid=2247488328&idx=1&sn=ecc9846db78b492f991a69ef9d9f6b10&scene=21#wechat_redirect)
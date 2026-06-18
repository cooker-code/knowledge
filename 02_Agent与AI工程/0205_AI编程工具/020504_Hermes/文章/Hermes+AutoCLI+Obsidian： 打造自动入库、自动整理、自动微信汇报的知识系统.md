---
title: Hermes+AutoCLI+Obsidian： 打造自动入库、自动整理、自动微信汇报的知识系统
author: GenAI共生人
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486639&idx=1&sn=afab9bd6fc7a268a719127ebbf1c368a&chksm=e88a2270d69d0a08df8ebe10153eebdef80f47ef463a10aa27a7396bf47c6015962ee4dbc936&mpshare=1&scene=24&srcid=0419kHTBHmYGAKl4Y38CKPXX&sharer_shareinfo=edbf956558ab531e25fac044eb52b11f&sharer_shareinfo_first=edbf956558ab531e25fac044eb52b11f#rd
---

前几天分享过我如何使用karpathy的llm wiki给自己打造一个智能进化的微信写作知识库。[卡兹克写作风格Skill+Karpathy LLM Wiki ：构建能自我进化的文章写作知识库](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486578&idx=1&sn=687c82967bb16780b3a1f5b8c6a228f8&scene=21#wechat_redirect)

运行一段时间后，感觉还是有点重。

每天要手动维护很多内容：文章筛选，文章拷贝输入，文章拷贝输出。。

而且还离不开电脑。。

感觉自己像是在给AI打工，充当AI的手脚，做些枯燥无聊的知识搬运。。

痛定思痛，我决定给这套知识库工作流做个升级！

结合当下最火的Hermes Agent，搭建一个能自动抓取信息入库、自动筛选整理、分类沉淀以及自动微信汇报工作的知识系统。

如果你也有相同的烦恼，或者相似的需求，

不妨花2分钟，和我一起看看这套AI知识系统是怎么搭建的吧。

---

## 流程概览

整个流程主要包括3步：

定时完成信息抓取 → 信息编译进 LLM Wiki → 微信汇报入库结果

## 流程拆解

### 1. Hermes+Autocli：信息抓取自动化

首先给大家推荐一个非常牛的浏览器信息抓取开源项目：OpenCLI / AutoCLI

它无需API KEY，直接复用你 Chrome 浏览器里已有的登录态，能实现90+站点的信息抓取。

我们按项目文档把Autocli的 skill 装好之后，就可以让 Hermes 接管 X 的内容抓取。

测试内容抓取没问题后，我们在obsidian建立一个AI NEWS HUB的文件夹，并在该路径下让Hermes执行内容抓取。

抓取X上和AI相关的点赞top10原文

可以看到，Hermes 会自动在 AI NEWS HUB 下创建多层级目录，并把抓取到的 X 推文（当天点赞top10）按日期归档到当天文件夹里。

接下来，我们直接让Hermes创建定时任务，每天下午5点定时执行信息抓取的任务。

到这里，第一层自动化就真正完成啦。

### 2. Hermes+llm wiki skill：定时梳理入库

第一步解决的是“新信息怎么进来”。

第二步解决的是：信息归档。

进来的东西，怎么变成真正能复用的知识，而不是又堆成一个素材仓库。

操作方法也很简单，还是在AI NEWS HUB路径下，让Hermes继续把x-ai-top10-daily的文章转化为llm wiki。

收到需求后，Hermes会自动调用内置的 llm-wiki skill完成信息转化。

wiki的结构也可以二次调整，按需调整为我们需要的结构。

wiki化的知识库结构

这一步最核心的价值，不是“让 Hermes 再干一遍搬运”，而是：

让 Hermes 充当知识编译器，把信息流压成认知地图。

当然，这个流程也可以继续定时自动执行。

也就是说，步骤 1 跑完后，步骤 2 紧接着接上去，整个入库过程就不再需要你盯着它。

### 3. Hermes+微信Clawbot：微信日报任务执行结果

前两步的任务，Hermes 都是在后台默默干活。

但，它干完了，我怎么知道？

很多知识库最后都死在这里。

有沉淀，但没有反馈。

你知道它理论上在更新，但你并不会每天主动打开去看。

时间一长，库就又变回了一个被动仓库。

所以我最后加了一步，看起来最轻，实际上最关键：

每天同步完成后，直接给自己发一份微信日报。

这份日报不是复杂的长文，而是一屏之内能看完的信息：

* 今天总览一句话
* 同步结果
* 今日重点
* 100 字摘要
* 报告位置

它主动汇报，才能督促我每天都会看知识库，用知识库。

Hermes接入微信的方法也很简单，直接让它帮我们接入个人微信渠道。

一番操作配置后，它会给我们返回一个二维码链接，我们打开链接用微信扫描接入即可。

连接上微信后，再把前两步的任务串起来，让 Hermes 在入库完成后自动生成日报，并把新增的关键信息发到我的微信。

微信渠道日报推送

到这一步，知识库的感觉就完全变了。

它不再是一个我把东西丢进去的容器。

它开始像一个每天主动给我做汇报的系统。

我很喜欢这种感觉。

因为它让“沉淀”第一次变成了“反馈”，让“存档”真正变成了“经营”。

## 写到这里，我真正想说的是：这套升级最值钱的，不是自动化，而是系统感

如果只看表面，这套流程不过就是：

* AutoCLI 负责抓；
* Hermes 负责编；
* Obsidian 负责看；
* 微信负责报。

但真正有价值的，是这四步终于被打通了。

以前，我是把内容往知识库里堆。

现在，是知识库开始自己运转：

* 自动接新信息
* 自动归档整理
* 自动按主题沉淀
* 自动把结果汇报给我

这不是从零搭了一个新系统。

而是把我原来已经在用的 Obsidian 知识库，升级成了一个有入口、有编译、有分类、有反馈的工作流。

终于，

我不再是 AI 牛马了，

我的知识库，开始自动服务我了。

---

如果觉得有帮助，谢谢帮忙点赞、收藏、转发。

我是芋头小宝，关注我，持续带你探索GenAI的成长宇宙。

【相关文章推荐】

[卡兹克写作风格Skill+Karpathy LLM Wiki ：构建能自我进化的文章写作知识库](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486578&idx=1&sn=687c82967bb16780b3a1f5b8c6a228f8&scene=21#wechat_redirect)

[0成本玩转龙虾智能体，小白也能快速上手！](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486500&idx=1&sn=aab50d8d90c8df7e932cd89192c32bce&scene=21#wechat_redirect)

[告别“作坊式”制作：LibTV把我的「AI视频」创作效率拉高了10倍！](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486458&idx=1&sn=0ec598895fd7f9cd797d0601a0363b34&scene=21#wechat_redirect)

[OpenClaw是鸡肋吗？电商版OpenClaw已经在干活了！](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486394&idx=1&sn=5cbf02cfd7012be12a2b121bd8970965&scene=21#wechat_redirect)

[别再排队装OpenClaw了，我找到了可以一键上手的龙虾产品！](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486346&idx=1&sn=466dd3561a25ec26c1de9a590a88ae4a&scene=21#wechat_redirect)

[云手机版的OpenClaw：像装App一样把龙虾装在手机里](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486236&idx=1&sn=bc7bfb7d6afec4bfbb776be7d62a11c2&scene=21#wechat_redirect)

[AI绘图界的天花板「Nano banana 2」 好玩案例+多形式使用入口分享（lovart/n8n/openclaw）](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486190&idx=1&sn=529d9e6d33496daeb8d8b2b09e948e2c&scene=21#wechat_redirect)

[OpenClaw Agent+飞书机器人：为每个业务场景配备专属「多Agent」项目协作群](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486147&idx=1&sn=5a91c5fb1471eeeacf70b5e94576ab3e&scene=21#wechat_redirect)

[接入飞书和钉钉的OpenClaw 到底能玩点啥？挖了一天的使用场景后终于明白了...](https://mp.weixin.qq.com/s?__biz=MzI1MjkyNjI4NQ==&mid=2247486065&idx=1&sn=5813382c8e12e08db49f9bab45784fd7&scene=21#wechat_redirect)
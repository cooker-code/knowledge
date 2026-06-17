---
title: 写一个优秀的Skill 有多难？Claude Code 内部复盘经验公开
author: 九章智算云
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492806&idx=1&sn=beefb32cdc4d6abd65034cf6a66dc900&chksm=c546e12b1dbb9c5904cfa0e3a57876782c5f5e0a257762612cb5beb5c44ca590f4edefc970ac&mpshare=1&scene=24&srcid=04047xnzZrKaPbY4E1WxlVdw&sharer_shareinfo=23cf6b3ac5c3ffa3eb6f47f174dfc5a2&sharer_shareinfo_first=23cf6b3ac5c3ffa3eb6f47f174dfc5a2#rd
---

Skill是什么？

> “
>
> 官方表述：Skills are folders of instructions, scripts, and resources that Claude loads dynamically，即由指令、脚本、资源组成的结构化文件夹，Claude 可动态加载以提升专项任务表现。

你可以简单理解为AI 的专项技能。

它是一个流程，一个SOP，可以帮助你完成特定任务。

比如你要做一道菜，你需要菜谱，只要照菜谱做，就能做个八九不离十。

Anthropic 的工程师 Thariq 发了一篇长文，把 Anthropic 内部使用 Claude Code skills 的经验倾囊相授。

今天，我们结合他公开的一手实战经验，聊一聊怎么构建优秀skills。

## 一、重新认识skills：它远不止一个Markdown文件

Skill如果是一个MD文件，可以用，但不算能用多好。

它是一个文件夹，一个完整的“能力单元”。里面可以包含：

* SKILL.md：核心指令与元数据
* scripts/：可执行的Python、Shell脚本
* assets/：模板文件、参考素材
* references/：详细的API文档、规范说明
* config.json：用户配置信息
* 甚至可以注册动态Hook，实现安全护栏

Skill 还支持很多配置选项（详见官方文档的 frontmatter 参考）

Thariq说得直白：“我们内部用得最好的那些skill，恰恰是充分利用了文件夹结构和配置选项的。”

## 二、高质量Skill的五个共性（整理自Anthropic数百个实战案例）

Thariq在梳理了内部数百个活跃skills后，总结出高质量skills的五个共性。

1. 承载“高语境信息”，而不是重复说明书

所谓的“高语境信息”，就是你在实践中沉淀的SOP或者说流程。

这个流程可以被复用、被优化。

AI已经知道很多通用知识了。你写“Python函数用def定义”，它早就知道。

但是在实践中积累的具体经验是只有你们自己人才知道的信息：

> “
>
> “这个审批流程文档里标可选，但实际上必须走，否则后面会卡住。” “数据库查询时，user\_id字段在凌晨ETL更新前可能为空，要加判空。” “对外API返回的错误信息要模糊化，内部日志才记录详细堆栈。”

这些就是“易踩的坑”,Thariq在文章中管这些叫Gotchas，是skill里高质量信息密度最高的部分。

一个skill的价值，取决于里面有多少内容是AI自己搜不到的。

2. Description字段：写给模型的“触发器”，不是给人看的摘要

很多人精心编写skill内容，却在description字段随手写“这是一个代码审查工具”。

但description是AI用来判断“要不要调用这个skill”的依据。

Claude Code启动时会扫描所有skill的Description。

* 差的description：“用于格式化代码。”（太模糊，何时触发？）
* 好的description：“当用户要求格式化代码、或代码风格不一致时触发。适用于.js、.ts、.py文件。不适用于配置文件和已压缩的代码。”

**把它想象成一个精准的if条件语句**。写得好，AI在需要时自动调用；写得差，你得每次都手动指定。

3. 让skill有“记忆”，越用越聪明

传统AI助手最让人头大的是每次见面它都仿佛第一次见。昨天让它生成了日报，今天它又问你格式。

给skill加上“记忆”很简单：

* 在skill目录下放一个standups.log文件
* 每次执行后，追加当天的日报内容
* 下次执行时，AI先读取历史记录，就能自动对比：“和昨天相比，哪些任务完成了？哪些卡住了？今天新增了什么？”

这种记忆不需要多复杂，一个追加写入的文本文件就能实现从“一次性工具”到“长期助手”的跨越。

4. 提供可执行资源，让AI组合而非重建轮子

一份Markdown写得再好，承载能力也有限。Skill的威力在于整个文件夹。

例如一个数据分析skill：

* 在scripts/目录下放几个Python函数：fetch\_user\_data()、calculate\_conversion()、generate\_report()
* 在assets/目录下放报表模板
* 在references/目录下放指标定义文档

当AI需要分析本周数据时，它不需要从头写查询逻辑，而是组合调用这些现成的函数，把精力花在数据解读和决策上，而不是重建轮子。

5. 设置安全边界：有能力，更要有护栏

一个真正可用的工作环境，不能只有能力，没有约束。

Skill可以注册“按需Hook”（On-demand Hooks），只在skill激活时生效。例如：

* /careful Skill：激活后，拦截所有rm -rf、DROP TABLE、force-push等危险命令。碰生产环境时开一下，就像安全锁。
* /freeze Skill：激活后，AI只能编辑指定目录下的文件。调试时特别有用，你只想让它加日志，不想它“顺手”改其他文件。

前者告诉AI“该做什么”，后者告诉AI“什么绝对不能做”。 两者结合，才是一个完整可靠的工作伙伴。

三、构建skills的实战路线图

知道了什么是高质量skill，具体该怎么构建？给你一个简单的路线图。

* 第一阶段：识别场景，从痛点入手（第1周）

问自己三个问题，找到skill的起点：

1）我是否在反复向AI解释同一件事？（团队代码规范、部署流程） 2）某些任务是否需要特定知识/模板才能做好？（设计稿转代码、周报生成） 3）某个任务是否需要多个流程协同完成？（从告警到排查到报告的输出）

你可以先列出你或者你们团队3个重复最多的痛点任务，作为首批Skill化的目标。

* 第二阶段：遵循核心原则，构建第一个skill（第2-3周）

**原则1：单一职责**

一个skill只做好一件事。既想当代码生成器，又想当审查员，还想当部署工具，最后往往什么都做不好。

**原则2：明确输入输出**

* 触发条件：什么情况下调用？（关键词、文件类型、上下文）
* 禁用条件：什么情况下不调用？
* 交付标准：最终输出长什么样？（格式、内容、验收清单）

**原则3：脚本化固定步骤**

能把流程固化的部分，全部写成脚本放在scripts/目录。AI的精力应该花在判断和组合上，而不是重复写样板代码。

**原则4：设计渐进式披露**

利用文件夹结构分层管理信息：

* SKILL.md：核心指令与元数据（始终加载）
* references/api.md：详细API文档（触发时加载）
* scripts/query.py：数据查询脚本（执行时按需加载）
* 避免一次性把所有信息塞进上下文，拖慢AI且降低性能
* 第三阶段：引入治理，实现规模化（持续进行）

当个人skill开始被团队使用时，治理成为关键。

三层治理体系：

1. 目录治理：统一命名规范（如team-前缀）、分类存放、版本管理。
2. 权限治理：通过Hook和配置，限制skill对系统、网络、数据的访问权限。遵循最小权限原则。
3. 质量治理：

* 接入门禁：重要的skill输出需经过测试、AI交叉审查或人工确认节点。
* 建立复盘机制：每周花30分钟回顾skill使用情况。哪些好用？哪些触发不了？哪些出了错？将失败案例变成skill里新的Gotchas条目。

Anthropic的内部演化路径值得借鉴：

1. 个人做出好用的skill，先放到GitHub沙盒目录。
2. 在内部渠道分享，同事试用后给出反馈。
3. 口碑建立后，通过PR进入团队的正式skill市场。
4. 通过Hook收集使用数据（调用次数、成功率），让优质skill自然浮现。

四、最后，说点实在的

建议先从你最烦的但是不得不每天重复做的事开始创建你的skill。

你先写下来，哪怕只有三行。

做什么、别做什么、做成什么样，存成你的第一个.skill文件。

然后用起来，踩了坑，再往里加一行“注意”。

生产力的提升就从偷懒开始~

参考来源：

• 原文标题：Lessons from Building Claude Code: How We Use Skills  
• 原始链接：https://x.com/trq212/status/2033949937936085378

**精彩内容**

**[看遍了市面上的coding plan，我发现还是这个好用](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492801&idx=1&sn=65cfbe7a012029e3a5bfd9e17be7752a&scene=21#wechat_redirect)**

**[我们做了一本OpenClaw 像素级实操案例集，看完直接上手](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492750&idx=1&sn=afb1ca904a1c34d95f19ac3c642137a6&scene=21#wechat_redirect)**

**[OpenClaw史诗级更新！但别急着更新，我们有更省事的办法](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492709&idx=1&sn=f16fd5efd0d7dc973f8556faed297001&scene=21#wechat_redirect)**

**[微信上线 ClawBot！你会发微信，你就会指挥它干活，前提是你得有只小龙虾](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492660&idx=1&sn=fa43994004551391f0fcc6155a9bbedd&scene=21#wechat_redirect)**

**[【完整版PPT】英伟达2026 GTC黄仁勋演讲稿，附下载](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492562&idx=2&sn=a07f58e0752941ca47fd88a7d56f23b1&scene=21#wechat_redirect)**

**[为了不让你的“龙虾”裸奔，我们做了一本73页的《OpenClaw 安全操作指南》](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492485&idx=1&sn=c2496512d950d4ebd3c3b5aa2570bef6&scene=21#wechat_redirect)**

**[我佩服我同事，竟然整理了一本83页的OpenClaw“红宝书”](https://mp.weixin.qq.com/s?__biz=Mzk2NDM3NzYzNg==&mid=2247492342&idx=1&sn=65a4aa65b02ca2161e6a14c605ffdea3&scene=21#wechat_redirect)**

**👇**戳****“阅读原文”体验九章智算云****
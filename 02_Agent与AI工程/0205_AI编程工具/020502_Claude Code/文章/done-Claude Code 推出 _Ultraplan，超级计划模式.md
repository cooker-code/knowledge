> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code 推出 /Ultraplan，超级计划模式
author: AGI Hunt
date:
url: https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453482608&idx=1&sn=80a194728987a6cb444d1cdbf07d6474&chksm=867c7548aabab2eaed0802d90b0c9028209fc0ea9c665292b234843e2280ce10b32d5cc9cea7&mpshare=1&scene=24&srcid=0412IFA496orhKTSLQ5qYJVm&sharer_shareinfo=ba70b8178be4c0473d83670ccbb349e8&sharer_shareinfo_first=ba70b8178be4c0473d83670ccbb349e8#rd
---

Claude Code 今天上了个新功能叫 `/ultraplan`，做的事情很好理解：在动手写代码之前，先在网页上给你看一份完整的实施方案。

Claude Code Ultraplan 标题

你可以读，可以改，甚至可以在方案里给 Claude 留评论。觉得没问题了，点一下「批准」，Claude 才开始动手。

听起来好像只是加了个可视化版的审批环节？但对于用 AI 写过复杂项目代码的人来说，这个功能解决的痛点，的确是真实存在。

01

## 来看 demo

在终端里输入 `ultraplan migrate auth to OAuth`，意思是：帮我把认证系统迁移到 OAuth。

在终端输入 ultraplan 命令并等待方案生成

几秒钟后，终端提示 `ultraplan ready · ↓ to view`，方案已经准备好了。按回车，浏览器自动打开 `claude.ai/code`。

网页自动打开并展示完整实施方案

页面标题是「Review Claude's plan」，左侧是导航，右侧是方案正文。

方案分成了几个板块：

**Context**（上下文）：Claude 把它理解的项目背景列了出来，相当于告诉你「我是这么理解你的需求的」。

**Implementation**（实施步骤）：具体要改哪几个文件、怎么改。比如在这个 demo 里，方案列了三步：替换 `middleware/auth.ts` 中的 `SessionStore` 为 OAuth 客户端，添加 `/auth/callback` 的 PKCE 流程，以及修改 `db/users` 表中的 `session_id` 字段。每一步都附了代码片段。

**Verification**（验证）：怎么确认改完没出事。方案给出了测试命令（`npm test -- auth`）和预期结果（14 个测试用例通过，callback 返回 302 并设置 `oauth_sub` cookie）。

方案中的验证步骤和测试命令

这里最关键的一点：**网页上的方案，我们是可以直接上手编辑的**。

你觉得哪一步有问题，选中那段文字，就能给 Claude 留评论。Claude 会根据你的反馈修改方案。

方案底部有两个按钮：「Run on web」（在云端执行）和「Approve & teleport to terminal」（批准并传回终端执行）。

点了「Approve」之后，回到终端：

批准方案后回到终端选择执行方式

「Ultraplan approved」，然后问你怎么执行：

• **Implement here**：把方案注入当前对话，就地开干 

• **Start new session**：清掉当前上下文，只带着方案开一个新会话 

• **Cancel**：先不执行，方案存着回头再说 

选了 Implement here 之后，Claude 就按方案一步一步改代码了。

02

## 我的体感

其实这个流程，我自己日常算是已经在用了，只是姿势略有不同。

遇到复杂的大型重构，或者要开发一个全新的大模块，我一般会先让 Claude Code 写一份详细的文档：方案细节、实施步骤、流程图、验证方式，全部理清楚。

然后我会到网页端去审阅这份文档，看结构是不是合理，有没有遗漏的边界情况，代码改动的顺序对不对。

**这个过程和 ultraplan 做的事情，其实还是蛮一致的**。只是之前我得自己把「让 Claude 写方案→去网页看→反馈修改→开始执行」这套流程手动串起来。

现在 Anthropic 把它做成了一个一键命令，更闭环、更集成了。从 `/ultraplan` 到网页审阅到一键批准回终端执行，整个链路是打通的。

不过，我目前也只在大项目上用这种方式。日常改个小 bug、加个简单功能，直接在终端里搞定更快，就没必要绕一圈去网页审方案了。

03

## 为什么需要它

AI 编程中，最常见的一个失败模式是什么呢？

**还没搞清楚要做什么，就开始动手了。**

你说一句「帮我把认证迁移到 OAuth」、「帮我做个淘宝」，Claude 二话不说就开始改代码。改到一半发现理解错了，回滚，重来。改了三个文件之后又发现漏了一个依赖，再回滚……

来来回回几轮之后，token 花了一大堆，时间花了几小时，但这事可能还不如不干。

`/ultraplan` 的思路是：**把「规划」和「执行」彻底拆开**。

`/ultraplan` 的规划阶段，Claude 读代码、理解意图、生成方案，这些都在云端完成。你在网页上审阅、编辑、确认，这个过程不消耗本地资源，也不会误改任何代码。

执行阶段才涉及到本地环境、文件系统、编译运行。而且此时 Claude 已经有了一份经过人类确认的方案，知道该做什么、做到什么程度、怎么验证。

Anthropic 工程师 Thariq 的解释是：**实现代码有时候需要本地环境和交互性，但规划阶段完全可以放到云端，因为它本质上就是读代码和理解意图。**

这个拆分背后其实有个更深的洞察：规划是「尴尬并行」（embarrassingly parallelizable）的。它不涉及文件系统，不涉及编译错误，纯粹是阅读和推理。而实现是天然串行的，需要本地环境一步步来。

04

## 和 /plan 的区别

Claude Code 之前已经有 `/plan` 模式了，效果也不错。那 `/ultraplan` 到底多了什么呢？

/plan vs /ultraplan 对比

核心区别：**plan 的方案在终端里看，ultraplan 的方案在网页上看**。

终端里看方案的问题在于，文本是线性的，你只能上下滚动。如果方案涉及五六个文件、十几个步骤，在终端里阅读体验其实挺差的。

网页端就不同了。

有侧边栏导航，可以跳转到任意章节。有代码高亮，有结构化的步骤列表。最关键的是，你可以**选中任意文本给 Claude 留评论**，形成一个审阅+修改的循环。

另外，ultraplan 的方案可以直接在云端执行（「Run on web」按钮），也可以传回终端执行。这意味着如果你在手机上审阅了方案，批准之后可以传回电脑上的终端去跑。

Token 消耗方面，Thariq 提到 ultraplan 和 plan 模式消耗的 token 数量差不多，订阅额度限制也一样。

目前 `/ultraplan` 处于预览阶段，所有开启了 Claude Code 网页端的用户都可以使用。

05

## Word 里的 Claude

同一天，Anthropic 还发布了 **Claude for Word**，Claude 正式进入微信 Office 套件了……啊不，微软 Office 套件。

这个功能目前处于 Beta 阶段，Team 和 Enterprise 用户可用。

Claude 会以侧边栏的形式出现在 Word 里，你可以直接在侧边栏里跟它对话，让它帮你起草、编辑和修改文档。而且 Claude 所有的改动都以 Word 的「修订模式」（Track Changes）呈现，你可以逐条接受或拒绝。

更值得注意的是：**Claude for Word 和 Claude for Excel、Claude for PowerPoint 共享上下文**。你可以在一个对话里同时操作多个打开的文档。

比如你可以对 Claude 说：「把 Excel 表里的 Q1 销售数据整理一下，生成一份 Word 格式的季度报告，再做一组 PPT 演示页。」三个应用之间的数据流通，全靠 Claude 来串联。

这个做法和微软自己的 Copilot 形成了直接竞争。

不过值得为微软点赞的是：**微软现在居然允许一个非 Copilot 的 AI 进入 Word 了！**

06

## 疯狂迭代

说到这里，你有没有注意到，Anthropic 最近发布的这些功能，几乎每一个都精准命中了你的痛点？反正对我而言是，我个人的一些使用姿势，正在被 Claude Code 做成更好的体验。

而前一天， Claude Code 还推出了 Monitor 工具，让 Claude 可以在后台挂一个监控脚本，有事件触发时才唤醒 Agent，告别轮询式的 token 浪费。

我们已经在上一篇文章中做了详细介绍：[Claude Code 推出 Monitor 功能，帮你随时盯梢](https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453482576&idx=1&sn=cd495dea53390208798f214ad20c69da&scene=21#wechat_redirect)

Monitor 解决轮询浪费，ultraplan 解决盲目执行，之前的 `/loop` 解决持续巡检，`/schedule` 解决定时任务……

我有个强烈的怀疑：

**Claude Code 在收集到数据后，会用 Claude 进行分析，再用 Claude 来发现潜在需求，然后直接用 Claude Code 完成开发并上线、打包、发布代码。**

**以及，源代码。**

想想看，它知道用户在终端里花多少时间滚动方案和对方案的要求（所以做了 ultraplan 搬到网页）。它知道 Agent 在轮询上浪费了多少 token（所以做了 Monitor）。它知道用户经常手动设定时任务（所以做了 `/loop` 和 `/schedule`）。

这些需求洞察，以前需要产品经理去做用户访谈、发问卷、分析数据。

而现在，让 AI 直接读用户的使用日志，找到模式，提出功能建议，甚至直接写代码实现，最后人类审阅一下就上线。

这不只是瞎猜。因为我们自己也在实践类似的方式：让 AI 分析用户数据，发现需求，生成方案，开发上线，再收集反馈，循环迭代。

AI 驱动的产品迭代飞轮

这套「**AI 驱动的产品迭代飞轮**」，可能就是 Anthropic 最近出招这么快的真正原因。

欢迎优秀的你加入我们，一起探索这种新的工作方式：[都 AI 时代了，还需要招人吗？](https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453480325&idx=1&sn=16835e2753c7f297a583a8ab9e84f3be&scene=21#wechat_redirect)

07

## 内部模型

回过头来一看，就这一两天时间，Anthropic 发布了：

• `/ultraplan`（Claude Code 的网页端规划功能） 

•  Monitor 工具（事件驱动的后台监控） 

•  Claude for Word（Office 套件集成） 

而如果把时间线拉长到最近两个月，清单还会更长：`/loop`、`/schedule`、Skills、hooks、background agents、Computer Use、Dispatch、Claude Code Desktop……

这个迭代速度，可以说是极其疯狂了。

而背后的原因，可能藏在一个大多数人还不知道的细节里。

前不久 Anthropic 发布了 **Mythos**，Claude 产品线中最高层级的模型。它在软件工程、网络安全、科学推理等各项能力上全方位超越了 Opus 4.6。

但这次不一样的是，**Mythos 是 Claude 有史以来第一个不公开发布的旗舰模型**。没有开放 API，没有更新 claude.ai 的模型选项，也没有发 benchmark 排行榜。它只面向 AWS、Apple、Google 等十几家核心合作方开放。

原因是 Mythos 在网络攻防上的能力，已经强到 Anthropic 觉得不能让所有人都用到它。

但 Anthropic 自己内部……当然是在用的。

**自 2 月 24 日起，Anthropic 内部就拥有了 Mythos。**

难怪 Claude Code 迭代，如此之快。

**当 Anthropic 工程师们内部自用的模型，比公开的 Opus 4.6 还强一个级别的时候，**

**AI 之下，分裂也正在加速：[AI 时代的人们，正在两极分化](https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453482591&idx=1&sn=1f973e79b30da22dcda9ed3db044b156&scene=21#wechat_redirect)**

◇ ◆ ◇

相关链接：

•  /ultraplan 官方推文：https://x.com/trq212/status/2042671370186973589 

•  Claude for Word 发布推文：https://x.com/claudeai/status/2042670341915295865 

•  Monitor 工具介绍：[Claude Code 推出 Monitor 功能，帮你随时盯梢](https://mp.weixin.qq.com/s?__biz=MzA4NzgzMjA4MQ==&mid=2453482576&idx=1&sn=cd495dea53390208798f214ad20c69da&scene=21#wechat_redirect)
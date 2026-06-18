---
title: 后悔没早装！推荐 8 个 Claude Code 必装 Skills
author: 老陈AI生活
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2ODc5MzUyMA==&mid=2247484332&idx=1&sn=80647f44dc683ec7c1a81d70b1f5841d&chksm=eba49ff34760f978efb720689f51367ba087566bf6ae75eb4b64f62c63ed5266764469aa68ac&mpshare=1&scene=24&srcid=04094XrmoWFY7MBeteW1syNr&sharer_shareinfo=2957638a637dbd47000d93bc93887d54&sharer_shareinfo_first=2957638a637dbd47000d93bc93887d54#rd
---

哈喽，大家好，我是老陈。

前几天，老婆问我：skills 是什么，有什么用？

我本来想解析一下的，但是张大个嘴，却出不来一个字。

突然发现原来很多我习以为常的东西，大家不一定都了解。原来我以为我知道的东西，要向别人解析清楚的时候，还挺难的。

今天和大家一起了解一下 skills 到底是怎么回事，顺带给大家推荐几个我认为必装的 skills。

# Skills 是个啥

简单说，Skills 就是给 Claude Code 装插件。

好比新买的手机，需要装 App 的对吧，Claude Code 也一样。

初始状态下它什么都能干一点，但装了 Skills 之后，特定场景下的能力会强很多。

技术上讲，就是一个 `SKILL.md` 文件，里面写了一套指令，告诉 Claude 碰到某类任务该怎么干，可以理解为处理特定场景的一套 SOP。

skills 有特定的触发词，命中就会触发。或者你可以用 `/` 斜杠命令直接触发它。

Skills 你可以直接装别人做好的，也可以自己写一个。安装别人的，就需要注意 Skills 的安全性了。

# 8 个必装 skills

## find-skills

**find-skills** 是我建议你第一个装的。

它是个“元技能”，功能就一个——帮你找别的 Skills。

你跟它说“我想找一个能做 xxx 的 skill”，它就去 skills 市场帮你搜，找到了还能直接帮你装上。

相当于一个技能商店的入口，没有它你得自己去 GitHub 一个个翻。

地址：

https://github.com/vercel-labs/agent-skills/tree/main/skills/find-skills

## skills-creator

**skills-creator** 是 Anthropic 官方出的，用来做你自己的 Skill。

它有四个模式：创建、测试、评估、跑基准。

你写好一个 Skill 之后可以直接用它跑一轮测试看看效果，不用自己瞎试。

如果你有一些反复在做的事情，这时候把它封装成一个 Skill 是最省事的。

地址：

https://github.com/anthropics/skills/tree/main/skills-creator

## Frontend Design

**Frontend Design** 是 Anthropic 官方出的，安装量 27 万多，排第二。

装了它之后让 Claude Code 帮你写前端界面，出来的东西明显不一样。

Claude 生成的 UI 怎么说呢，能用但是丑，配色基本固定是蓝紫色。

再看看装了这个 Skill 之后，相同的提示词，做出来的效果。

|  |  |  |
| --- | --- | --- |
|  |  |  |

这个 skills 就像给 Claude Code 注入一套设计系统和视觉哲学，出来的界面像是有审美的人做的。

我现在做原型基本都靠它，省了不少跟设计师来回改稿的时间。

地址：

https://github.com/anthropics/skills/tree/main/frontend-design

## web-access

**web-access** 这个是中文开发者 eze-is 做的，解决了 Claude Code 联网能力不够的问题。

原生的 WebSearch 和 WebFetch 能力其实比较弱，很多页面抓不到、动态内容加载不出来。

web-access 直接给它接了一套完整方案：WebSearch、curl、Jina、CDP 浏览器操作，按场景自动选工具。

最关键的是它能直连你本地 Chrome，天然带着你的登录态，那些需要登录才能看的页面它也能搞。

还有个很实用的功能——并行分治。你让它同时调研 5 个网站，它会分发子 Agent 并行去爬，不是一个个串行等。

地址：

https://github.com/eze-is/web-access

## Superpowers

**Superpowers** 是社区最火的开发工作流 Skill，相当于一个开发项目经理。

你给它一个需求，它的流程是：先跟你头脑风暴确认方向，然后出设计规范，再出实现计划，确认完了自动分发子 Agent 去写代码，写完了自动跑代码审查，最后帮你合并。

整个软件开发生命周期它都管了。

适合那种需求比较复杂、涉及多个文件的任务。

简单改个 bug 就没必要用它了，杀鸡用牛刀。

地址：

https://github.com/obra/superpowers

## Code Reviewer

它可以帮你审代码。不是简单地过一遍，而是并行启动好几个代理，分别从代码质量、安全漏洞、性能问题这几个角度同时审。

提 PR 之前我一般都先让它跑一遍，有时候能揪出来一些自己根本没意识到的问题。

误报也有，所以要自己再重新检查一遍，看一眼就知道该不该处理。

地址：

https://github.com/anthropics/claude-code/tree/main/plugins/code-review

## 办公四件套

PDF、DOCX、PPTX、XLSX。 Anthropic 官方出的。

PDF 我用得比较多，转格式提取内容这种活。

PPT 嘛怎么说呢，让它帮我做过一次分享，框架是能用的，细节不行，字号大小颜色什么的还得自己调。

不过你要说从零开始做，那确实省了不少事。Excel 倒是比较省心。

这四个 skills 对于日常有 office 办公需求的人更方便。

我老婆前两天让我帮她弄个表格，我直接丢给 Claude Code 处理，咔咔几下就搞定了。

地址：

https://github.com/anthropics/skills/tree/main/pdf

https://github.com/anthropics/skills/tree/main/docx

https://github.com/anthropics/skills/tree/main/pptx

https://github.com/anthropics/skills/tree/main/xlsx

## Remotion

Remotion 是干嘛的呢，用 React 写视频。

听着有点怪对吧，我一开始也觉得奇怪。

Remotion 这框架我也是前不久才知道，写 React 组件定义每一帧画面，动画字幕音频都能控制。

深度使用我谈不上，就拿它做过几个产品演示的短视频。

对程序员来说，写代码定义时间线确实比拖 Premiere 顺手。

能不能替代专业剪辑？那不可能，差远了。

但做个简单演示啥的，够用了。

这个是我让 Claude Code 用这个 skills 帮我把这篇文章生成的视频：

地址：

https://github.com/remotion-dev/remotion

# 怎么安装

最简单的方法，就是直接把 skills 的地址，丢给 Claude Code，跟它说“帮我安装这个 skills”，它就会自己搞定。

另外还有两个方法，相对都没有上面的方面直接：命令行安装，claude plugin marketplace add 加上仓库地址就行。

git clone 到 ~/.claude/skills/ 目录下，重启一下就能用。

装完之后在对话里打 / 就能看到所有可用的 Skills 列表了。

# 多说两句

skills 根据自己实际需要，装三五个就够了，真的。

我一开始也是看啥都想装，后来发现 Skills 装多了，发现有些会冲突，明明想用 A 的，却命中了 B，不符合预期效果。

上面几个是我目前用着觉得还行的组合，也不一定适合你，毕竟每个人干的活不一样。

你们心目中有哪些值得安利的 Skills？评论区一起交流～

未来已来，Enjoy AI！

— END —

欢迎点赞、在看、转发
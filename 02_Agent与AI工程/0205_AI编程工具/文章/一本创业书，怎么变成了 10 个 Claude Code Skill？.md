---
title: 一本创业书，怎么变成了 10 个 Claude Code Skill？
author: 梦飞来晚了
date: 梦飞梦飞
url: https://mp.weixin.qq.com/s?__biz=MzI1OTY4OTYwNw==&mid=2247483897&idx=1&sn=548eecef56c6be8785298247b026307d&chksm=eb49d9c2bb1363eb48954269f19c05c4d1fc9e7bead5f3f14968f01f13897ef52fc1e40b2f4f&mpshare=1&scene=24&srcid=05126sB9obyyZNdbzqIGaGOo&sharer_shareinfo=a465657099c60650d2e98d865990b1f0&sharer_shareinfo_first=a465657099c60650d2e98d865990b1f0#rd
---

# slavingia/skills最有意思的地方，不是把一本创业书做成 10 条命令，而是把“找社区、验证想法、做 MVP、找客户、定价、增长、复盘”这套创业判断，变成 Claude Code 能调用的工作流。它更像一份会在你动手前追问边界的检查清单，而不是普通提示词合集。

来晚了，但这次这个仓库值得补一下。

slavingia/skills的 README 很短，短到几乎不像一个 8k+ stars 的项目。它的定位也很直接：基于 Sahil Lavingia 的《The Minimalist Entrepreneur》，给 Claude Code 做了一组 skills。

听上去像“把一本书做成提示词”，但真正有意思的地方不在这里。

它把一本商业书里原本需要你读完、划线、做笔记、再自己迁移到项目上的方法，拆成了 10 个可以在 Claude Code 里直接调用的 slash command。

封面图

# 一、它不是知识摘要，是决策入口

很多人收藏商业书，其实收藏的是一种安慰：以后我要做产品、定价、增长的时候，应该能用上。

问题是，真到要做决定时，很少有人会把书翻出来，从头找那一段。

slavingia/skills的处理方式比较干脆：别等你想起来翻书，直接把关键场景做成命令。

比如你还没想清楚服务谁，就用/find-community。有了想法但不确定值不值得做，就用/validate-idea。准备动手做第一版，就用/mvp。已经有产品，要找前 100 个客户，就用/first-customers。

这些命令不是要替你创业，也不是让 Claude Code 给你编一份商业计划书。它更像是在你准备冲动加功能、免费送服务、过早扩张时，把你拉回几个很普通但很难坚持的问题：

* 你到底服务哪个社区？
* 这个问题有人愿意付钱吗？
* 第一版能不能周末就发？
* 这件事能不能先手动交付？
* 你是在增长，还是只是在烧钱？

# 二、10 个 skill，其实是一条创业路径

README 里列了 10 个 skills。单个看是工具，连起来看就是《The Minimalist Entrepreneur》里那条很克制的路径。

流程图

它不是从“我要做一个伟大产品”开始，而是从“先找到人群”开始。

顺序大概是这样：

1. /find-community：先找你理解、能接触、愿意服务的人群。
2. /validate-idea：确认这个问题不是你脑补出来的。
3. /mvp：把范围压到能尽快交付，而不是做一个漂亮但没人用的完整产品。
4. /processize：先手动把服务跑通，再考虑写代码自动化。
5. /first-customers：一个一个找到早期客户。
6. /pricing：从一开始就面对收费问题。
7. /marketing-plan：用内容教育市场，而不是只等自然流量。
8. /grow-sustainably：花钱、招人、扩张都要回到可持续。
9. /company-values：在 hiring 之前先想清楚公司文化。
10. /minimalist-review：任何重要决策前，再按这套原则复盘一遍。

这个顺序其实比命令本身更重要。

现在很多 AI 工具喜欢帮人“加速”：更快写代码、更快生成方案、更快出文案。但创业早期真正危险的，往往不是速度不够，而是太快把错误的东西做大。

这套 skills 的价值就在这里。它不是一直鼓励你往前冲，而是反复提醒你：先找人，先验证，先手动，先收费，先活下来。

# 三、为什么说它比普通提示词更像“工作流”

普通提示词合集通常是这样的：复制一段 prompt，粘到聊天框里，然后期待模型给你一个好答案。

skills 的思路不一样。

Claude Code 官方文档里说，每个 skill 都以SKILL.md为入口，可以附带说明、资源、脚本和更复杂的工作流。也就是说，它不是一段孤立 prompt，而是一块可以被工具识别、被上下文触发、被团队复用的能力模块。

slavingia/skills虽然文件不复杂，但方向很清楚：把一套商业判断，收进 Claude Code 的能力列表里。

你可以这样安装：

BASH

/plugin marketplace add slavingia/skills

/plugin install minimalist-entrepreneur

也可以本地 clone：

BASH

git clone https://github.com/slavingia/skills.git ~/.claude/plugins/skills

然后在 Claude Code 里添加本地插件：

BASH

/plugin marketplace add ~/.claude/plugins/skills

/plugin install minimalist-entrepreneur

安装之后，它就不只是“我收藏过一个仓库”。当你在项目里说“帮我验证这个想法”或者直接调用/validate-idea，Claude Code 会按这个 skill 的说明来组织问题和输出。

这件事的意义，是把知识从“我知道”变成“我每次做决定时真的会用”。

# 四、MVP 这个 skill 最能看出味道

我看了一下skills/mvp/SKILL.md，它的核心不是教你怎么快速写一个漂亮 demo，而是反过来压范围。

它把 MVP 拆成三个阶段：先手动做，再流程化，最后才产品化。

这和很多人理解的 MVP 不太一样。

很多人说 MVP，其实是在做“迷你版完整产品”：登录、后台、支付、数据面板、设置页，一个都不想少，只是 UI 粗糙一点。

但这个 skill 会把你往更小的地方拉：

•能不能周末就交付？

•能不能先用表格、表单、Notion、Airtable 这类现成工具？

•客户愿不愿意付钱？

•你是不是还没有规模问题，却已经在为规模写代码？

这类问题听起来很土，但对早期项目非常有用。

因为 Agent 最容易做的事，是帮你把产品做复杂。你只要多说两句，它就能给你补权限、补订阅、补团队空间、补埋点、补后台。/mvp这种 skill 的价值，是在模型很能干的时候，给它套上一层“不许乱扩”的约束。

# 五、适合谁用

这仓库最适合三类人。

第一类，是正在用 Claude Code 做个人项目的人。你可以把它当成产品决策前的检查器，尤其是在准备开新坑、扩功能、改定价时。

第二类，是想把方法论做成 Agent 工作流的人。这个仓库本身就是一个很好的样例：一本书不一定只能变成读书笔记，也可以变成一组可调用的场景能力。

第三类，是独立开发者和小团队。越小的团队，越容易被“我们先做完整一点”拖死。这套 minimalist 的思路，至少会逼你回到付费、交付、客户和现金流。

但边界也要讲清楚。

它不是创业成功按钮。它不会替你找到真实客户，也不会替你承担市场判断。它能做的，是在你准备做决定时，把一套很克制的创业原则摆到你面前，让你别轻易骗自己。

# 六、梦飞的判断

slavingia/skills有意思的地方，不是 GitHub 上又多了一个热门 Claude Code 插件。

真正值得看的，是它把一本商业书从“内容”变成了“动作”。

以前一本书的生命周期大概是：买了，读了，划线，收藏，忘了。现在它可以变成一组 Agent skills：当你要做 MVP，它出来追问范围；当你要定价，它出来提醒收费；当你想招人扩张，它出来问你是不是已经可持续。

这可能才是知识产品接下来更有想象力的形态。

不是把书总结成 10 条金句，而是把它做成 10 个会在关键时刻拦你一下的命令。

热点来晚了，但瓜更熟。梦飞帮你补错过的全网热事。

原文链接

https://github.com/slavingia/skills
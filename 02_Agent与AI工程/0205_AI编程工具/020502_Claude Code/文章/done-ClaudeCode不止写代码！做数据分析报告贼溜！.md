> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: ClaudeCode不止写代码！做数据分析报告贼溜！
author: 喜新
date:
url: https://mp.weixin.qq.com/s?__biz=MzU1MTAwNzY4Mg==&mid=2247490110&idx=1&sn=54ee2fd0a92cd6bbd123f9e2c28e11c9&chksm=fa9c0ab95772a9c7e7d4e08138fa78fcce14317e0b2e072636bbeb234046faa057c27cfe9f2e&mpshare=1&scene=24&srcid=1101pVP1pQbYGdr6mq4QW3AQ&sharer_shareinfo=907a638d2b46b62c01a1cec41d520367&sharer_shareinfo_first=907a638d2b46b62c01a1cec41d520367#rd
---

用 Claude Code 写代码其实挺没意思的，乌漆嘛黑的。

但是 Claude Code 的架构设计非常好，给它配个 `脚手架` + `Agent`团队能做的事情其实非常多。

下面是一个硬核的示例：扔一个 `Excel表格` + `业务诉求`给它，可以生成一个下面这样的 PPT 汇报：

iShot\_2025-09-26\_11.06.14

这个“PPT”是我用网页模拟的，它可以是使用动态的图表来显示，效果比真 PPT 好不止一个数量级。

全流程都由 AI 完成，完整的项目我放在文末了，你可以下载下来，拖进终端里直接用。

下面是项目实现原理和配置教程。

## ClaudeCode脚手架

实现原理还是要懂一点的，这样你就可以根据自己的需求做一些自定义配置了。

整个项目由一个`网页脚手架`和`数据分析Agents团队`组成：

* • 网页脚手架用来呈现 AI 生成的可视化报告，AI 会分析一个版块、生成一个网页，自动嵌入脚手架里，模拟 PPT 效果。你可以随意增删调整演示顺序。
* • 数据分析 Agents 团队由三个 Agent组成：

+ • “领导”：负责整体分析方向把控、分析维度梳理和任务分配
+ • “分析师”：负责处理数据、生成指定维度的分析结果
+ • “设计师”：负责根据“领导”的要求，用“分析师”产出的数据和结论设计 PPT

最终项目是一个文件夹：

把它们拖进 `终端`或 `Powershell`里，唤醒 Claude Code 就能直接用。

完成任务后，所有“PPT”页面都在`/report/slides`文件夹里，打开`/report`文件下的index.html 就能像 PPT 一样演示。

网页脚手架感兴趣可以自己调整视觉，让 AI 给你改就行，没啥难度。

**重点说一下 Agents 团队。**

他们本质上由三个提示词驱动：

* • “领导”，在`CLAUDE.md`中，是整个项目的提示词
* • “分析师”和“设计师”，基于 Claude Code 的 subAgents

之所以用subAgents，因为每一个 Agent 独享上下文，不会因为分析过程产生的大量的数据和打印结果导致项目中后段被大量上下文卡死。

我在领导提示词里要求它给“员工”下达指令时，必须用「任务卡」的形式传递，写清楚干什么、怎么干、预期产出是啥。

这样每个 Agent 被唤起的时候，直接读任务卡就可以不考虑历史信息，直接独立完成一个任务。

每一个任务的过程数据都会被保存下来，记在这个任务卡里。

这样你在汇报被问到这个数据哪来的时，可以找到这一页的任务卡—>定位分析思路+原数据位置，打开给你的领导看。

你甚至还可以不露声色的让他们感受“你小子还会写代码呢！加薪！”

项目已经开源了，感兴趣可以看详细的提示词约束。

## 如何使用

在 Github 下载项目到本地

Repo地址：https://github.com/comeonzhj/DataAnalysis\_ClaudeCode

无法正常访问 Github 的伙伴莫慌，项目的完整代码我也放在《AI 学习行动圈》的星球了，扫码加入，搜`Claude Code 数据分析`即可。

把你要分析的数据表格放在`/data`文件夹内，终端进入项目路径、唤醒 Claude Code、输入你要分析的要求就好了。

**关于模型：**

如果你能用 Claude 原生模型，肯定最好，但就是性价比第一点。

国内模型，我测试了 DeepSeek、GLM-4.5 和 K2，只有 K2 做出来的东西比较像样。

相同的任务，DeepSeek 生成了只有图表的 6 页 PPT，部分页面都没提示词要求呈现，被裁切了。

DeepSeek

GLM-4.5 也是生成了 6 页 PPT，同样也是基本只有图表没有分析。但是分析过程异常混乱，三个Agent 生成的文档乱存。导致协作时另一个 Agent 找不到文件，然后自己上手干……

项目 Repo 的 Pages 演示的是 K2 的产出结果，24 页 PPT，基本都有图表+结论，需要的维度都覆盖了。

因为整个过程需要写超多代码，推荐把所有模型全都换成 K2-turbo，否则要等超久。

下面是配置模型的环境变量

```
export ANTHROPIC_BASE_URL='https://api.moonshot.cn/anthropic'
export ANTHROPIC_AUTH_TOKEN='你自己的KEY'
export ANTHROPIC_MODEL='kimi-k2-turbo-preview'
export ANTHROPIC_SMALL_FAST_MODEL='kimi-k2-turbo-preview'
export ANTHROPIC_DEFAULT_OPUS_MODEL='kimi-k2-turbo-preview'
export ANTHROPIC_DEFAULT_SONNET_MODEL='kimi-k2-turbo-preview'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='kimi-k2-turbo-preview'
export CLAUDE_CODE_SUBAGENT_MODEL='kimi-k2-turbo-preview'
```

最后，邀请你加入我已经运营超过 500 天的 AI 学习行动圈，跟 4000+人一起学习、交流、精进。

我的各种 AI 研究心得、发现的好应用、开发的小项目都会在里面分享，目前圈子有核心三个交流学习平台。

帮你用好 AI，抓住时代机会别掉队。

### 7 个微信群，早报和日常交流

微信群里每天一早有 AI 早报，上下午还有“读报时间”，以及我每天不定期刷屏级的各种 AI 工具体验、提示词编排思考、行业新闻解读同步。

以及，你可以在群里讨论任何与 AI 相关的工具、应用问题，几乎都能找到答案。

### 腾讯文档-圈友空间

用来沉淀体系化、深度的 AI 文章和超长的工程化提示词，不定期更新。

当前包括：`Claude code`、`Cursor`、`Manus`等顶级产品的系统提示词和工具列表，各种深度的 Agent 白皮书和实践指南

### 知识星球-每日报告、工具和实战经验分享

我在星球里主要维护「实战分享」「工具箱」和「情报局」三个标签

实战分享是可以在日常工作和生活中直接应用的提示词和效率工具。上面截图里的 Step-Back 提示词就非常好用，堪比 o4。在公众号、直播中演示的所有 AI 实战应用的提示词也都在这个标签下。

AI 工具和鲜知道就是好用的、热门的 AI 工具、资讯分享，我把那些太技术、太浮夸的都筛选了，放进这个标签的都是可以直接用来的好玩儿！

星球还有一个“专栏”体系，目前的定位跟标签差不多。
---
title: 什么样的代码仓库是 Agent-Friendly 的？
author: 已晨
date: RockBotRockBot
url: https://mp.weixin.qq.com/s?__biz=MzkzNzYzMjQ3Mw==&mid=2247484600&idx=1&sn=99d728b507d945f7cf06c3b22c7f5693&chksm=c37dca719711c3c4246e3cfb24696543f16e1ad8e68dbe41d141c6dd7b3623bdb2c04c66683e&mpshare=1&scene=24&srcid=0603avDC5uXfvFxsqLcwkJrO&sharer_shareinfo=af2734189522d8b8b1ae36cd4c1571ca&sharer_shareinfo_first=af2734189522d8b8b1ae36cd4c1571ca#rd
---

现在基本所有开发者都在用 AI 编程，但对着一个存量代码仓库做 Vibe Coding 的时候，不知道你有没有想过：什么样的代码仓库，对 Coding Agent 是更友好的，更能发挥出 Coding Agent 的功效？

我们习惯于在 Claude Code 执行一个 `/init`，生成一个 CLAUDE.md，就认为这个仓库已经初始化好了。但这样就够了吗？这个话题好像很少有人认真讨论过。

有些 Coding Agent 自带 Repo Wiki 工具，读取代码仓库实现生成一堆 wiki.md，样式精美、内容丰富。还有些同学会把历史的 PRD 和技术方案都放进仓库，让大家了解来龙去脉。我对这些行为带来的效果也表示怀疑。

最近发现 Anthropic 新上了两个官方插件，一个叫 claude-md-management，负责评估 CLAUDE.md 的质量；一个叫 claude-code-setup，负责推荐应该往代码仓库里增加哪些文件。这两个插件一定程度说出了这个问题的答案，也验证了我的一些想法。

## 1、CLAUDE.md 只应该写从代码仓库里读不出来的东西

claude-md-management 插件负责给代码仓库里边的 CLAUDE.md 打分，整体打分有六个维度，满分加起来共 100 分。

所以 CLAUDE.md 最该写的是从代码仓库里读不出来的坑、为什么这么做、团队约定，以及能直接跑的命令。

那些精美的 repo wiki.md 该不该保留？从这个角度看，不言自明。

## 2、要以自动化的形式集成到代码仓库，别光写文档

claude-code-setup 插件里，目前只有一个claude-automation-recommender的Skill。它扫描一遍代码仓库，然后推荐应该加哪些可被“自动化”的内容，这些文件以 Hooks、Subagents、Skills、MCP Servers、Plugins 的形式存在。

* • Hooks：钩子，绑在工具事件上自动触发的动作。比如，检测到有tests目录，则推荐增加PostToolUse的hook，改完跑相关测试
* • Subagents：子agent，专门审查者或分析师，可以并行执行。比如，检测到有支付类代码，则推荐创建security-reviewer subagent做安全审计
* • Skills：把流程和模板打包成可复用的能力。最值得注意的是它建议你为某些信号新建 skill，而且连带把辅助类文件封装进去。比如，检测到代码仓库有测试套件，则推荐构建gen-test这个skill，并提供了测试的示例。Skills 还有一层触发控制，决定调用方是user-only还是Claude-only
* • MCP Servers：对接外部工具。比如检测到React、Express 等流行库，则推荐context7的MCP，可以查实时文档
* • Plugins：把多个相关 skills 打成一个包安装

整张清单看下来，没有一项是“写一篇说明文档”。比如，检测到测试，它给的不是"在某个 .md 里写一句记得跑测试"，而是一个自动跑测试的 hook。

其他文件，都是能被Claude Code默认感知到，或者强约束执行的内容。这么看，“把历史的 PRD 和技术方案.md都放进仓库”，Coding Agent可能不会感知到，效果也就几乎为零了。

## 3、判断 Agent-Friendly 的两个标准

从这两个官方插件来看，判断一样东西对 Coding Agent 有没有用，基本可以归结为下面两个标准：

1、它有没有从代码仓库里读不出来的知识？

2、它会不会被自动加载进上下文，或者被写成强制检查？

如果这两个的答案都是否定的，那就没用，甚至起反作用。

我们所有的历史经验也要按这两条标准过一遍。能固化成规则的，就沉淀为 hooks、skills、subagents，让 agent 自动感知或强制执行。

其他的，清理掉就好。
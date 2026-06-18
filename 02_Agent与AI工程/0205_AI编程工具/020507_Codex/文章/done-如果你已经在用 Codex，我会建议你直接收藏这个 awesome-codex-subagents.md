> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020507_Codex/020507_核心知识点/Codex工程使用与沙箱边界|Codex工程使用与沙箱边界]]
---
title: 如果你已经在用 Codex，我会建议你直接收藏这个 awesome-codex-subagents
author: Git Trend
date: wsleepybearwsleepybear
url: https://mp.weixin.qq.com/s?__biz=MzkwMzY1Mjg5MQ==&mid=2247485818&idx=1&sn=3a7169264f8a354cdc9860e557be5d62&chksm=c13b2f5cc224056a3d128ad9aa77f2d916b536cc03dfd0987700a9582748cd90d4fbbca34260&mpshare=1&scene=24&srcid=0521hN9ZN6OtsOBSmv8tAmWx&sharer_shareinfo=b15976de9e72f3ec89be67c4a08fc361&sharer_shareinfo_first=b15976de9e72f3ec89be67c4a08fc361#rd
---

如果你已经在用 Codex，我会建议你直接收藏 `awesome-codex-subagents`。

它最有价值的地方，不是“收集了很多 agent”，而是把这件事整理到了能立刻拿来用的程度。你不用从零想角色，不用自己补目录结构，也不用先写一堆 `.toml` 试错，直接挑合适的 subagent 拷进去就能开跑。

项目卡片

* 项目：awesome-codex-subagents
* GitHub：https://github.com/VoltAgent/awesome-codex-subagents[1]
* 推荐理由：136+ 个 Codex subagents，10 个大类，直接按官方目录结构落到 `~/.codex/agents/` 或 `.codex/agents/` 就能用
* 一句话判断：如果你已经开始嫌 Codex 只有一个默认人格不够用，这个仓库非常值得抄作业

为什么这个仓库值得收藏

现在很多人已经不满足于“让一个 agent 什么都干”。真正进入日常开发以后，你会越来越想把角色拆开：前端一个、后端一个、review 一个、debug 一个、security 一个、文档一个、Git 流程一个。

问题是，自己从零开始写这些 subagent 很费时间。你不只是要写提示词，还要想命名、想调用时机、想模型怎么配、想 sandbox 怎么开。

`awesome-codex-subagents` 最省事的地方就在这里：它不是讲理论，而是直接给现成模板。

*这个仓库的真正价值，不是某一个 agent，而是它已经整理出一整套可直接挑选的角色库。*

它给的不是零散 prompt，而是一整套目录

README 里写得很清楚，这个仓库收了 **136+ 个 Codex subagents**，分成 **10 个类别**。

核心类别包括：

* Core Development：前后端、全栈、API、UI
* Language Specialists：Python、Go、React、Next.js、Rust、Java、Vue
* Infrastructure：Docker、Kubernetes、Terraform、SRE、Cloud
* Quality & Security：reviewer、security-auditor、debugger
* Data & AI、Developer Experience、Meta & Orchestration 等

这就意味着，它已经不是“有几个有用的 agent”，而是开始接近一个可组合的 Codex 角色库了。

它最实用的地方：完全按 Codex 官方方式来

这点我很喜欢。README 没搞什么自定义框架，而是直接按 Codex 官方约定来：

* 全局 subagents 放 `~/.codex/agents/`
* 项目级 subagents 放 `.codex/agents/`
* 项目级优先级更高
* 你要显式在 prompt 里调用，Codex 不会自动帮你 spawn

示例命令也很直接：

```
mkdir -p ~/.codex/agents
cp categories/01-core-development/backend-developer.toml ~/.codex/agents/
```

*这个仓库最友好的地方，是它没有额外造轮子，而是完全贴着 Codex 官方目录结构来。*

这种仓库最怕搞成“看起来很全，实际落不进去”，但它在这件事上反而很克制：就是 `.toml` 文件，按官方路径复制，没额外心智负担。

还有两个细节很对味

**第一，model routing 是写进 agent 的。** README 专门解释了不同 subagent 的 `model` 字段怎么配：深度推理类用 `gpt-5.4`，轻量扫描和研究类用 `gpt-5.3-codex-spark`。

**第二，sandbox 思路也是成体系的。** reviewer、auditor 这类分析型 agent 设成 `read-only`，developer、engineer 这类执行型 agent 设成 `workspace-write`。

这两个细节加在一起，会让你感觉它不是简单收集，而是在认真整理一套可执行约定。

如果是我自己用，我会先装哪几个

如果是我自己，我不会一口气装 136 个。

我会先挑 5 个最容易立刻见效的：

* `backend-developer`
* `reviewer`
* `debugger`
* `security-auditor`
* `docs-researcher`

先把这几个丢进 `~/.codex/agents/`，跑几轮真实任务，再决定要不要继续加更细分的角色。因为 subagent 不是越多越好，关键是你会不会真的调用。

最后一句判断

`awesome-codex-subagents` 值得收藏，不是因为它又是一个漂亮的 awesome list。

而是因为它把 Codex subagent 这件事，从“你可以自己研究一下”推进成了“你现在就能拷过去开始用”的状态。

如果你已经在用 Codex，这种仓库的价值不是启发，而是直接替你省掉第一轮搭建成本。

如果这篇对你有用，建议点个关注。我会持续把 GitHub 上值得用的 AI 工具拆成「最短上手闭环 + 坑点清单 + 可复用配置」，让你少走弯路。

### 引用链接

[1]*https://github.com/VoltAgent/awesome-codex-subagents*
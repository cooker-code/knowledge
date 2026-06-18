---
title: AI 界的 npm 来了！Vercel 开源 add-skill，一行命令搞定所有 Agent 技能管理！
author: 开源星探
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMjQ0NzI0OQ==&mid=2247504967&idx=1&sn=501794a644346b2565c9dd8f460245ed&chksm=c1093321d54ecfff0596b06c9194ca03b7f2cacfb1faa0f37a400e44cdaa008d1a4bb6157e87&mpshare=1&scene=24&srcid=0126OwlwscOYH5UT6o9ecLuU&sharer_shareinfo=c21470f6aed2149fd785d66e98b7378d&sharer_shareinfo_first=c21470f6aed2149fd785d66e98b7378d#rd
---

Vercel Labs 最近的产出速度确实惊人，而且每一款都精准地切中了开发者的“心病”。

现在 AI 编程工具竞争异常激烈，每一家工具的配置路径都不一样，Skills（技能）的管理也变得碎片化、难同步、难分发。

好在，前端界的“基建狂魔” Vercel Labs 团队看不下去了。最近开源了一个名为 **add-skill** 的项目，彻底终结了这种混乱。

它被开发者们称为 **“AI 技能界的 npm”**。有了它，你管理 AI 技能就像安装npm包一样简单。

并且还同步上线了一个 Skills 市场，不仅有 GitHub 上热门 Skills 的安装排行，还有详细的介绍及安装指令，整个页面体验非常友好。

#### 核心能力

1、自动探测，开箱即用

你不需要告诉它你装了哪些 Agent。

运行命令时，它会自动扫描你的系统，识别出你是否安装了 Cursor、Claude Code、Codex、OpenCode、Trae、Windsurf 等20+款主流 AI 编程工具。

检测到之后，它会自动把对应的技能配置、系统提示词和工具定义，注入到该工具的正确路径下。

2、跨工具的一键分发

这是最牛的地方。只需输入一行命令即可：

```
npx add-skill vercel-labs/agent-skills
```

它会从 GitHub 下载技能包，并根据检测到的工具列表，自动把技能分发到每个工具对应的配置目录下。

你不需要关心路径在哪，它比你更懂你的电脑。

3、支持多种源格式

它支持 GitHub 简写、完整的 Git URL，甚至是任何 Git 仓库（比如GitLab）。

这意味着你可以轻松搭建一个公司内部的私有技能库，一键同步给所有同事。

4、skills.sh

配合工具推出的 `skills.sh` 网站做得极其优雅。它不仅是一个列表，更是一个带有社区热度指引的市场：

* • **安装量统计**：你可以看到哪些技能最受欢迎。
* • **分发详情**：可以看到某个技能分别被安装到了哪些软件上，这为你的选择提供了极大的参考价值。
* • **简洁直观**：看中哪个，直接点击复制命令，回到终端回车即搞定。

还可以自定义你的技能库：

你完全可以创建一个 GitHub 仓库，放一个 `SKILL.md`。然后运行 `npx add-skill 你的用户名/仓库名`，以后咱们自己的技能也可以随时随地安装了。

#### 写在最后

Vercel 这次开源 add-skill，其野心远不止于做一个工具。

它是在**制定 AI 时代的组件化标准**。以前我们通过 npm 分发“代码块”，现在我们通过 add-skill 分发“知识和工作流”。

如果你把时间线拉长，会发现 add-skill 很像早期的：

* • npm（Node 生态）
* • pip（Python 生态）
* • brew（macOS 工具）

当工具数量少的时候，大家还能靠 README 活着。

一旦生态爆发，统一分发和管理工具就会成为基础设施。

如果你只用一个 AI 工具，你可能暂时感受不到 add-skill 的价值。

但只要你符合下面任意一条👇

* • 同时用 2 个以上 AI 编程工具
* • 经常在不同电脑/环境间切换
* • 希望打造一套“自己的 Agent 能力栈”
* • 开始认真依赖 Skills 而不是 Prompt

那 add-skill 几乎是必装级别。

GitHub：

> https://github.com/vercel-labs/add-skill  
> https://github.com/vercel-labs/agent-skills

 

如果本文对您有帮助，也请帮忙点个 赞👍 + 在看 哈！❤️

**在看你就赞赞我！**
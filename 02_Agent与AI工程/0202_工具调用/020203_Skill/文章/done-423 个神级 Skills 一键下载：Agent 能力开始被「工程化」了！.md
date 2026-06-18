> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020203_Skill/020203_核心知识点/Skill能力封装与治理边界|Skill能力封装与治理边界]]
---
title: 423 个神级 Skills 一键下载：Agent 能力开始被「工程化」了！
author: AI信息Gap
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwMzYzMTc5NA==&mid=2247509253&idx=1&sn=6c5fac46fe655a0185794359d397690f&chksm=c1a7244fe6ab280c1216980f8531798fa9f895b32c7b81297d0f852548722a8b6c2b73486cbe&mpshare=1&scene=24&srcid=0116dMOWfoM3ZND8KQ6CXa6j&sharer_shareinfo=6972637cd9627607deb2da2c76a76213&sharer_shareinfo_first=6972637cd9627607deb2da2c76a76213#rd
---

这，是用 AI 生成的 Dashboard 仪表盘。

白色背景、紫色渐变、蓝绿配色，AI 味儿拉满了。

有个词专门形容这种设计：AI slop，翻译过来就是「AI 泔水」。

但同样的模型，同样的提示词，换个方式用，效果完全不一样。

这，是添加了 `frontend-design` skill 后生成的 Dashboard。

深色工业风、科技字体、黄青配色。一眼就能看出，有点东西。

区别在哪？

一个 400 tokens 的 Markdown 文件。

很明显，问题不在模型能力，而在于：AI 有没有被明确告知「在这个场景下什么才算做对」。

这个文件，就是 Skills。

附上我上面用到的 `frontend-design` skill 链接。

> https://skill0.atypica.ai/zh/skills/anthropics-skills-skills-frontend-design-skill-md



---

## 01｜Skills 是什么

Skills，又叫 Agent Skills，是 Anthropic 在 2025 年 10 月推出的一套标准。

简单说，就是一个包含 `SKILL.md` 文件的文件夹，告诉 AI「在特定场景下该怎么做」。

更准确地说，是告诉 AI「该如何判断，并稳定执行」。



一个 skill 里有什么？

`SKILL.md` 是核心，用 Markdown 写的指令集。

脚本是可选的，预写好的代码处理复杂任务。

资源文件也是可选的，比如模板、字体、参考素材。

工作机制相当简单。

AI Agent（智能体）启动时会扫描所有可用的 Skills，读取每个 skill 的名称和描述。

当你的任务和某个 skill 匹配时，AI 自动加载它，按照里面的指令执行。

用完即走，不占用上下文（Context）。

换句话说，只把需要的能力临时「挂载」进来。



Skills 和提示词（Prompt）有什么区别？

提示词每次都要写，而且会占用上下文窗口。而 Skills 是预置的，按需加载，可复用。

提示词是「临场指挥」，Skills 更像「把正确做法固化成流程」。

举个例子，提示词是临时口头交代，Skills 是写好的 SOP 手册。新员工入职，你不用每次都从头教，给他一本员工手册就行。



Skills 有三个核心特点，也是它火起来的原因。

可组合。多个 Skills 可以协同工作，AI 自动判断需要哪些，形成可复用的能力链。

可移植。同一个 skill，Claude Code、Codex、Cursor、OpenCode 都能用，不用改一行代码。

高效。只加载需要的，不浪费 token 和上下文长度。把上下文留给当下任务，而不是反复解释规则。



---

## 02｜Skills 热潮：比 MCP 还重要？

2025 年 10 月 16 日，Agent Skills 首次亮相，那会还是 Anthropic 的专属。

两个月不到，OpenAI 被发现悄悄在 Codex CLI 和 ChatGPT 里采用了同样的标准。

12 月 18 日，Anthropic 把 Skills 作为开放标准发布，官网是 `agentskills.io`。

紧接着 12 月 24 日，OpenAI 正式官宣 Codex 支持 Skills。

后来，VS Code、GitHub、Cursor、Amp、OpenCode，几乎所有主流 AI 编程工具都加入了。

Django 联合创始人 Simon Willison 说：

> Skills 可能比 MCP 更重要。我预计 Skills 会迎来一波寒武纪大爆发，到时候回头看，今年的 MCP 热潮都算不上什么。

为什么这么说？

MCP 需要搭服务器，门槛高。Skills 一个 Markdown 文件就行，任何模型都能用。

通用性是 Skills 最大的优势之一。

更关键的区别在于定位。

MCP 更偏「接入与调用」，Skills 更偏「把判断写进系统」。

这会直接影响交付稳定性。



---

## 03｜国内团队入场：skill0

Skills 生态刚起步，散落在 GitHub 各个角落。

谁先做聚合平台，谁就有先发优势。

「skill0」就是这个角色。

我专门查了一下，这是由国内特赞团队（atypica.ai）推出的 Skills 市场，目前收录了 423 个 skills。基于开放标准 `SKILL.md`，Claude Code、Codex、Cursor、OpenCode 通用。

地址在这：`skill0.atypica.ai`。

「skill0」把分散的能力做成可检索、可下载、可直接装配的索引层。

收录内容包括 Anthropic 官方 skills（xlsx、pdf、pptx、frontend-design 等）、OpenAI Codex skills、社区贡献的各种 skills（PyTorch、Metabase 文档等）。

特赞是一家做企业级智能体（GEA）的公司。「skill0」是他们内部实践的外放，是被真实业务折磨出来的解法。

它对应的是「企业场景里，必须把判断工程化才能规模化交付」的那条路。



---

## 04｜手把手教程：用 frontend-design 让 AI 学会审美

划重点：这个功能现在就能用，完全免费。

### 步骤一：在 skill0 找到你要的 skill

打开 `skill0.atypica.ai`，搜索对你有用的 skill，比如这个前端设计 `frontend-design`。

页面会展示 skill 的完整说明，它能做什么、什么时候会被触发、具体指令内容。

### 步骤二：下载 skill

点击 `Download ZIP`，下载到本地。

### 步骤三：安装到 OpenCode

解压后，把整个文件夹放到 `~/.claude/skills/` 目录下。OpenCode 完全兼容这个路径。

当然，你也可以放到 OpenCode 原生路径 `~/.config/opencode/skill/`。

重启 OpenCode，skill 就生效了。

### 步骤四：开始爽用

让 OpenCode 做前端相关的事，它会自动调用这个 skill。

比如我文章开头的提示词：

> 设计一个 AI 产品的数据分析 Dashboard。左侧是导航栏，顶部放四个数据卡片显示今日概览（用户数、调用次数、收入、增长率），中间是 7 天趋势折线图，右侧放最近的 API 调用记录列表。

你不需要反复「教审美」，因为审美与方法论已经被写进了 skill。

Cursor 和 Claude Code 用户，同样的 ZIP，解压后放到对应目录就行。

Cursor 是 `~/.cursor/skills/`，Claude Code 是 `~/.claude/skills/`。

这就是「skill0」。

把零散的「正确做法」沉淀成可装配能力，降低试错与复用成本。



---

2025 年是 MCP 元年，2026 年是 Skills 元年。

这一次，真正的分水岭可能不在「能不能做」，而在「能不能把判断写成系统、让交付稳定发生」。

这一次，国内团队没有缺席。



---


> 我是木易，Top2 + 美国 Top10 CS 硕，现在是 AI 产品经理。
>
> 关注「AI信息Gap」，让 AI 成为你的外挂。



---
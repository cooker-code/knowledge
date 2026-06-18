> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020104_Pi/020104_核心知识点/Pi极简Coding Agent与可扩展Harness|Pi极简Coding Agent与可扩展Harness]]
---
title: 什么是pi？下一代Agent架构？
author: AI Engineer编程
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNTExNzMwNg==&mid=2647832724&idx=1&sn=a217b996ec035ece21922592af69c02a&chksm=f1e337a64a6070cd4645f10b24cc1f8399e728f0ce771fd93a5b11d2bf59c571981d0c21da7d&mpshare=1&scene=24&srcid=0410jj2U9EHp4E43gGiwyAai&sharer_shareinfo=dd1f5ece15d085caffe72803c534a5f8&sharer_shareinfo_first=dd1f5ece15d085caffe72803c534a5f8#rd
---

**Pi**

**Pi：极简AI Agent**

当所有人都在追逐功能最多的 AI 编程工具，有人悄悄做了一个功能最少的——却被开源社区广泛采用，还成了 AI 助手 OpenClaw 的底层引擎。

阅读约需 8 分钟

**都在做加法**

那种"万能"软件——功能多到使用者找不到真正需要的那个按钮。

AI 编程工具正在走上同一条路。Claude Code、GitHub Copilot、Cursor……每隔几个月就多一堆新功能：子代理、计划模式、权限弹窗、内置任务列表……而大多数开发者其实只用到其中的一小部分。

有一位游戏引擎开发者，叫 **Mario Zechner**，受不了这种膨胀，自己动手写了个工具。他给它起名叫 **Pi**。

"There are many coding agents, but this one is mine."

有很多编程代理，但这个是我的。

**Pi 到底是什么？**

Pi 是一个运行在终端（命令行）里的 AI 编程代理。可以把它理解成"能自己写代码、读文件、执行命令的 AI 助手"，只不过它极其克制。

（任务描述部分，是因为读取了我安装的skills）

安装只需一行命令：

$ npm install -g @mariozechner/pi-coding-agent

然后打开终端，输入你的需求，Pi 就会帮你读代码、改文件、跑命令。

（ps：国内用户可以使用中转站，或者deepseek等模型如下图）

Pi 只有四个核心能力：

**读文件（Read）**

读取项目里的任何代码文件，理解上下文。

**改文件（Edit / Write）**

直接修改或创建文件，不用来回复制粘贴。

**执行命令（Bash）**

跑测试、安装依赖、构建项目，直接在终端完成。

**支持 15+ 个 AI 模型**

Anthropic、OpenAI、Google、本地 Ollama……会话中途随时切换。

就这四个工具，加上一个不到 300 词的系统提示。这是目前最轻量的主流 AI 编程代理之一。

**"不做什么"才是设计哲学**

Pi 的官网有一个专门板块，叫做 **"What we didn't build"**（我们没做什么）。这在软件产品里极为罕见——通常发布页都在堆功能列表。

**Pi 刻意省略的功能**

**没有 MCP 协议支持** — 写个技术文档告诉 AI 怎么做就够了

**没有子代理** — 用 tmux 多开终端窗口自己管理

**没有权限确认弹窗** — 跑在容器里就好

**没有计划模式** — 新建一个 TODO.md 文件

**没有内置任务列表** — 任务列表会占用模型上下文

**没有后台 Bash** — 用 tmux，保持完全可见

这不是偷懒，而是一种立场：**工具应该适应你的工作流，而不是反过来。**

Pi 的逻辑是：如果你需要某个功能，自己用 TypeScript 写一个扩展，或者直接让 Pi 帮你写。这套"用 AI 扩展 AI 工具"的逻辑本身就是 Pi 的精髓。

**树状对话历史**

绝大多数 AI 工具的对话是线性的：你问，它答，一条直线往前走。走错了方向，只能从头来。

Pi 把对话历史做成了**树状结构**。

你可以在任意一个历史节点上"分叉"，往不同方向试验，再回来对比结果。思路上接近 Git 的分支：管代码用分支，Pi 用同样的方式管你和 AI 的对话过程。

这个设计的核心判断是：线性对话本质上是一种约束，而探索性的编程任务需要的是可回溯的空间。

**OpenClaw：Pi 的"上层建筑"**

国内爆火的养虾项目：**OpenClaw**。

可以参考我之前的文章：

[OpenClaw 可能的替代品](https://mp.weixin.qq.com/s?__biz=MzIzNTExNzMwNg==&mid=2647832561&idx=1&sn=c20fc1bc430ab6e266e70b2eaa761e4c&scene=21#wechat_redirect)

[从新手到常用：OpenClaw 最近 10 个版本更新全解析（普通用户篇）](https://mp.weixin.qq.com/s?__biz=MzIzNTExNzMwNg==&mid=2647832653&idx=1&sn=9fde60554ab85472148f729d1972a690&scene=21#wechat_redirect)

OpenClaw 的底层编程引擎，正是 Pi。

OpenClaw的文档：

> https://docs.openclaw.ai/zh-CN/pi
>
> openclaw官网

从架构层面看，OpenClaw 使用 Pi 的 **RPC 模式**运行——Pi 通过标准输入/输出与外部程序通信，OpenClaw 在此之上构建了多渠道接入、多 Agent 调度、媒体处理等能力。Pi 负责"怎么执行"，OpenClaw 负责"从哪里来、到哪里去"。

**OpenClaw（上层）**

WhatsApp · Telegram · iMessage · Slack · 语音通话 · 日程管理

↓ RPC 模式通信

**Pi 编程代理核心（中层）**

Read · Write · Edit · Bash · 会话管理 · 扩展系统

↓ 调用

Claude · GPT · Gemini · Llama · …（15+ 个 AI 模型）

Pi 与 OpenClaw 的架构关系示意

**Pi 与其他工具的差异**

|  |  |  |
| --- | --- | --- |
| **功能** | **Pi** | **Claude Code / Cursor** |
| **核心哲学** | 极简，可扩展 | 功能完整 |
| **支持 AI 模型数量** | **15+ 个** | 通常绑定 1 家 |
| **树状对话历史** | **原生支持** | 不支持 |
| **子代理 / 计划模式** | **可用扩展实现** | **内置** |
| **TypeScript 扩展系统** | **50+ 示例** | 通常不支持 |
| **SDK 嵌入其他应用** | **原生支持** | 通常不支持 |
| **学习成本** | 中等 | 较低 |

(下面是使用Pi进行的提问的回复，使用模型ops4.6)

Pi总结：

> Cursor 是"最好用的 AI IDE"——适合大多数开发者的日常编码。
>
>  
>
> Pi 是"最灵活的终端 Agent"——适合追求定制化和模型自由的终端用户。
>
> Claude Code 是"最强模型的终端入口"——适合 Claude 重度用户。
>
> Codex 是"云端代码工人"——适合异步批量任务和团队工作流。
>
> OpenCode 是"最轻量的开源选择"——适合 Go 爱好者和极简主义者。
>
> Pi回复

**围绕 Pi 的项目生态**

Pi 之外，已有一些值得关注的衍生项目：

**Pi 核心**

Mario Zechner 的原始项目，npm 上已有数百个依赖，持续迭代中。

**oh-my-pi**

开发者 Can Bölük 的 fork，补充了 LSP 语言服务器支持、浏览器工具、哈希锚定编辑等功能，保留了 Pi 的可扩展核心。

**OpenClaw**

Peter Steinberger 基于 Pi 构建的多平台 AI 助手，支持 20 余种通讯渠道，Raspberry Pi 官方博客曾做过专题介绍。

**pi-web-ui / pi-mom**

社区扩展包：pi-web-ui 提供浏览器端聊天界面；pi-mom 是 Slack Bot 集成，作者 Mario 本人也在日常使用。

**Pi 适合你吗？**

Pi 不是为所有人设计的，这一点作者也没有回避。

**适合你，如果……**

你熟悉命令行操作，有基本的 TypeScript 能力，在意工具能否贴合自己的工作流，不满足于"能用就行"的开箱体验。

**可能不适合，如果……**

你希望安装完直接用，不打算写扩展代码，或者日常主要用 IDE 图形界面工作。

用作者自己的比喻：Pi 更像一把好的厨刀，而不是一台多功能料理机。前者需要你懂得怎么用，后者按按钮就行。两者没有高下，取决于你是哪类厨师。

**结语**

软件工具的膨胀有时是市场竞争的副产品：功能多，意味着有东西可以发布，有内容可以宣传。Pi 选择了另一个方向——把能砍掉的全砍掉，把剩下的做扎实，然后把"加功能"这件事交还给用户。

这不是什么新鲜的思路，Unix 哲学里早就有了。但在 AI 工具这个赛道上坚持这个方向，并且做出一个被实际项目（如 OpenClaw）采用的产品，还是需要一些判断力和克制。

Pi 目前仍在活跃开发中，社区也处于早期阶段。如果你对 AI 编程工具感兴趣，值得花半小时把玩一下。

**参考资料**

Pi 官网：pi.dev

Pi GitHub：github.com/badlogic/pi-mono

OpenClaw：openclaw.ai

作者博客：mariozechner.at
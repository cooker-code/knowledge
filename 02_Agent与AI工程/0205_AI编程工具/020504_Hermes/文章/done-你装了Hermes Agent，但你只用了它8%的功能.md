> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 你装了Hermes Agent，但你只用了它8%的功能
author: 量子智元
date: 量子智元量子智元
url: https://mp.weixin.qq.com/s?__biz=MzkwMTc4NTkwNg==&mid=2247488056&idx=1&sn=8b47f6f33f6312ec6dac4d2807116bec&chksm=c1107280bb22f79b9e46dad901c96145516258a02d101a4f3c5a9fbc5ed543afb5bb72b9b9ed&mpshare=1&scene=24&srcid=0509pCtd5obGlSOLxv0IcmgF&sharer_shareinfo=514e733f35c918b0bdda9086076569b4&sharer_shareinfo_first=514e733f35c918b0bdda9086076569b4#rd
---

先问你一个问题。

你是怎么用 Hermes Agent 的？

大概率是这样：连上 Telegram，选个模型，发一条消息，等回复，关掉窗口，叫它一声聊天机器人。

如果是这样，你在用一台跑车拉货。

原文作者 Sharbel 列了 15 个功能，他说大多数跑了几个月 Hermes 的人，一个都没碰过。我看了一下，确实。

---

**第一类：大多数人直接跳过的初始配置**

**SOUL.md — 你的 Agent 只需要被教一次**

Hermes 启动时会读一个文件，叫 SOUL.md。你在里面写什么，Agent 就是什么——说话方式、拒绝什么、写给谁看、用什么语气。

写一次，所有会话、所有平台、所有模型全部继承。不用每次都打"你是一个资深 X 专家，请用中文回答……"

用 `/personality` 可以在会话中途切换预设的人格。

大多数人每次开新对话都在重新介绍自己。

**MEMORY.md + USER.md — 让 Agent 记住你**

两个持久文件，每次会话都会读入。MEMORY.md 是 Agent 对你项目的笔记，USER.md 是它对你这个人的理解——你的角色、你的偏好、你在乎什么。

底层用 FTS5 全文索引加 LLM 摘要，能把 8 周前的一条记忆拉进今天的会话里。

大多数人每次开新对话都在重新解释自己是谁。

**`/insights [天数]`**

跨所有会话的数据统计。哪个项目烧了最多 Token，哪个 provider 花了多少钱，Agent 在哪卡住了，你最常回来问什么。

`/insights 30` 一秒钟看完最近一个月。

大多数人对自己的 AI 用量一无所知。

**`/snapshot`**

把当前 Hermes 的完整配置和状态保存下来。要改 SOUL.md 又怕改崩？先 snapshot，改完不满意，`/snapshot restore` 回到改之前。

大多数人不知道 Agent 本身也能回滚。

---

**第二类：任务进行中没人用的控制指令**

**`/branch`（别名 `/fork`）**

当前会话开叉，像 git 分支一样。想试一个激进的方案但不想丢掉现有上下文？fork 出去试，不行就回来。

大多数人直接开新会话，把刚建立的上下文全扔掉。

**`/rollback`**

文件系统检查点。Agent 跑了一个破坏性修改，代码改坏了？不用 git，直接 `/rollback`，Hermes 保存了它动过的每个文件的快照，随时还原。

大多数人是在 Agent 把文件搞坏之后才知道这个功能存在。

**`/steer` 和 `/queue`**

Agent 已经跑了 3 步工具调用，你发现它在用生产 API 而不是测试环境。不用杀掉整个任务重来。

`/steer 用 staging API，不要用 prod`

下一次工具调用就会看到这条修正。当前这一轮不中断，提示词缓存保持热。

`/queue` 是把下一轮指令排进队列，等当前轮结束自动执行。

大多数人碰到这种情况直接杀掉重跑。

**`/btw`**

临时插一个问题，用当前会话的上下文来回答，但不调用任何工具，也不写入记忆。就是"我快速问一下，别影响主线任务"。

大多数人为了问一个附带问题专门开新会话，然后回来发现上下文没了。

**`/yolo`、`/fast`、`/reasoning`**

三个开关。`/yolo` 跳过所有危险命令确认（谨慎用）。`/fast` 切到低延迟模式。`/reasoning` 调推理强度，适用于 o 系列模型。

大多数人一直用默认设置，然后说 Agent 响应慢。

---

**第三类：没人意识到的多模型架构**

**`/model [--provider]`**

Hermes 从设计上就是 provider 无关的。一条命令换模型，不用重启，Agent 状态保留。

`/model anthropic:claude-opus-4-7` 切到 Opus，`/model openrouter:kimi-k2.6` 降到便宜模型处理简单任务。支持的 provider 包括 Anthropic、OpenAI（GPT-5.5，走 OAuth 不用 API key）、OpenRouter、Gemini、AWS Bedrock、小米 MiMo 等十几家。

大多数人从一开始选了一个 provider，之后再没换过。

**辅助模型（Auxiliary models）**

Agent 做的事不只是回答你的问题。它还要压缩上下文、摘要会话、生成标题、处理视觉任务。

Hermes 允许你给每个子任务分配不同的模型。主脑用 Opus 4.7，上下文压缩用 Haiku 4.5，标题生成用最小的模型。配置一次，之后自动运行。

大多数人用 Opus 的价格做 Haiku 级别的工作，每次会话都在烧多余的钱。

---

**第四类：装了但没打开的分发能力**

**17 个平台的消息网关**

Telegram、Discord、Slack、WhatsApp、Signal、邮件、短信、Matrix、飞书、企业微信、钉钉、QQBot，加上 CLI 和语音，一共 17 个。一个 Hermes 进程驱动全部。

`hermes gateway` 启动，所有平台同步上线。用户白名单、频道限流、DM 配对，都在配置里。

大多数人只装了 Telegram，其他 16 个一个没碰。

**`/voice`**

CLI、Telegram DM、Discord 频道、Discord 语音频道里，都可以实时语音。`/voice` 开启，直接说话。

走路、开车、离开键盘的时候有用。打字比说话慢的时候——也就是大多数时候——也有用。

大多数人只会打字。

**Cron 定时任务 + `/webhook-subscriptions`**

Hermes 内置定时任务调度器。用自然语言写计划，告诉它发到哪里。

"每周五下午五点，把这周的 GitHub commits 汇总一下发到 Slack #standups 频道。"解析、定时运行、自动投递。

`/webhook-subscriptions` 是反向的：让 GitHub、Vercel、Stripe、监控服务把事件推到你的 DM，零 Token 消耗，零延迟。

大多数人为了做这件事订了一个 Zapier。

---

**第五类：把 Hermes 从工具变成工作流的那一步**

**Skills 是斜杠命令**

Hermes 自带 100 多个 Skill，每一个都是斜杠命令，`/` 开头，自动补全。

`/architecture-diagram` 生成 SVG 架构图，`/excalidraw` 手绘风格图，`/manim-video` 数学动画，`/linear` 管理 issue，`/google-workspace` 接管 Gmail、日历、Drive，`/youtube-content` 视频转内容，`/codex` 和 `/claude-code` 把任务委托给其他 Agent。

`/test-driven-development` 强制执行红绿重构循环，`/systematic-debugging` 四阶段根因分析。

关键的是：你可以自己写。

原文作者写了一个叫 `/sage` 的自定义 Skill，它能发现他所在领域的异常信号、扫描趋势、推荐发什么、起草回复、批量整理他的日常输出，用他自己的语气。写了一次，在任何设备、任何平台输入 `/sage` 就能跑，永久有效。

大多数人一周用一次斜杠命令。用 Hermes 用得认真的人，把整套工作流装进了斜杠命令里。

---

**说在最后**

你安装了一个有持久记忆、100 多个内置 Skill、文件系统回滚、会话分支、任务中途修正、17 个平台分发、实时语音、多 provider 自由切换、辅助模型独立路由、定时任务、Webhook 接入、自定义斜杠命令能力的 Agent。

然后把它当聊天窗口用。

工具没有欠你的。是你还没给它真正的指令。

参考链接：https://x.com/sharbel/status/2049158152709382177
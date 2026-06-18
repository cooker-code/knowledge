> 已吸收至：[[02_Agent与AI工程/0203_RAG与知识库/020303_RAG/020303_核心知识点/RAG生命周期与评估边界|RAG生命周期与评估边界]]
---
title: 你存进去的知识，为什么一条都没用上？Obsidian + Claude 这样让知识库自己变聪明
author: 石臻说AI
date: 石臻石臻
url: https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500620&idx=1&sn=2e22136768099c11cfa78868b56f7000&chksm=ceb27a8f060674b0836028c423908601542056f0d1cc5dec2f1f5b06d0a1f52a337d8e607b80&mpshare=1&scene=24&srcid=0511QPyWOtbcytZkk7eYYYfG&sharer_shareinfo=1e69d3b3d281194b8c5cbdd918138587&sharer_shareinfo_first=1e69d3b3d281194b8c5cbdd918138587#rd
---

⭐ 设为星标 · 第一时间收到推送

石臻说AI
编辑：石臻

**导读：** 你有没有试过把文章、播客、想法全部存进笔记软件，然后再也没打开过？

这不是懒，是系统设计的问题。把东西存进去却没有反馈机制，知识库只会变成一个漂亮的垃圾桶。

这篇文章讲的是一个不同的思路：让 Obsidian 结合 Claude，建一个每天自动变聪明的知识系统——你只管输入，连接这件事交给 AI 来做。

## 大多数知识管理系统为什么都失败了

先说失败模式，因为很多人都踩过坑。

第一个问题：**捕获太费劲**。每次保存一篇文章，还要手动打标签、分类、写摘要。这在认知负担高的时候根本坚持不下去，两周之后习惯就断了。

第二个问题：**没有连接层**。笔记之间是孤立的。三月份存的一篇文章和今天碰到的问题可能直接相关，但没有任何机制帮你发现这个关联。

第三个问题：**没有理由回来看**。如果知识库不主动推送东西给你，你就得记得去拉。没人记得住，库就成了「偶尔搜一下」的工具，而不是思考伙伴。

用 @cyrilXBT 的话说：**一个从不回话的第二大脑，不是第二大脑，只是一种有条理的忘记方式。**

这套系统针对这三个问题逐一解决。

## 四层架构：每一层只做一件事

整个系统分四层，逻辑清晰：

**第一层：捕获层**

负责把信息自动拉进系统，你不用手动做任何事。

* Readwise：文章高亮自动同步，连 Kindle、Twitter 书签、Pocket 都覆盖
* Airr：播客片段，手机一摇就剪
* Whisper：语音备忘，录音丢进去自动转文字
* Telegram Bot：在路上冒出的想法，发给 Bot 直接进库

这一层配好之后就不用再碰。

**第二层：管道层（N8N 自动化）**

N8N 监听每个捕获源，有新内容自动格式化成 Markdown 文件，存进 Obsidian 对应目录。不用手动归档，不用复制粘贴。

**第三层：Obsidian 本地存储**

就是本地一堆 `.md` 文件。这层是系统的「地基」，所有东西永久存在这里，不删。完全在本地，不依赖任何云服务，数据主权在你手里。

**第四层：Claude 智能层**

读你的库，找连接，写每日简报，回答关于你自己思维的问题。这是把存档变成思考伙伴的那一层。

## 五个文件夹，足够了

文件夹结构故意设计得很简单：

| 目录 | 用途 |
| --- | --- |
| `/inbox` | 所有内容先落这里，未处理的原始素材 |
| `/notes` | 处理过的文章高亮、播客片段，一个来源一个文件 |
| `/ideas` | 你自己的想法、观察、语音转文字 |
| `/projects` | 进行中的项目，一个项目一个子目录 |
| `CLAUDE.md` | 告诉 Claude 你是谁、在做什么 |

保持简单是有意为之。文件夹越复杂，存东西的时候就越容易卡在「这个到底放哪」，慢慢就不用了。

## CLAUDE.md：整个系统最重要的文件

没有这个文件，Claude 每次启动都是一张白纸，不知道你是谁、在做什么、想要什么。

有了它，Claude 等于有个背景板——读过你几个月的笔记，知道你现在在搞什么项目，知道你的思维习惯。

直接抄这个模板开始用：

💡 PROMPT

提示词参考：
# Who I Am
Name: [你的名字]
Work: [你在做什么——越具体越好]
Focus: [你现在重点搞的一件事]
Goals 2026: [3 个具体目标]
# Current Projects
Active: [现在在建或在做的事]
Stuck on: [最需要想清楚的地方]
Next milestone: [这个阶段"做完了"是什么样子]
# How This Vault Works
Inbox: /inbox — 未处理捕获，先放这里
Notes: /notes — 处理过的文章、高亮、研究
Ideas: /ideas — 我自己的想法和观察
Projects: /projects — 活跃项目文件夹
# What I Want From You
- 发现我没看到的连接
- 在同意我之前先挑战我的假设
- 问我"该聚焦什么"时，从库里的内容回答，不要泛泛而谈
- 如果我现在相信的东西和我之前存的东西矛盾，告诉我
# What I'm Reading and Thinking About
[每周更新——最近在关注什么、在想什么问题]

`Current Projects` 和 `What I'm Reading` 这两块每周一更新，五分钟。这个习惯决定了 Claude 的回答是不是还在点子上。

## 每天早上的自动简报

每天你坐下来开工之前，系统已经把简报准备好了。不用你去问，N8N 在早上 6 点自动跑，结果存进 `/inbox/brief-日期.md`。

把这个 prompt 放进你的 N8N Claude 节点：

💡 PROMPT

提示词参考：
You are reading my Obsidian knowledge vault. Read everything in /inbox from the last 24 hours and everything in /notes from the last 7 days.
Then do three things:
1. CONNECTIONS — Find the 3 most interesting connections between recent captures and older notes I probably have not noticed. Be specific. Quote the relevant passages.
2. PATTERN — Identify one pattern across everything I have been reading this week. What is my brain clearly working on even if I have not said it explicitly?
3. QUESTION — Give me one question worth sitting with today based on the pattern you identified. Not a task. A question.
Write this as a clean markdown file formatted for Obsidian. Save it to /inbox/brief-{{date}}.md

三个部分设计得有意思：**连接**是把你没注意到的东西挖出来，**模式**是帮你看清楚自己最近在想什么（往往你自己说不清），**问题**比任务更有价值——一个好问题比十个待办事项更能推进思考。

## 每周深度合成

每天简报是「发现连接」，每周合成是「建立观点」。建议固定在周一花 15 分钟跑一次。

💡 PROMPT

提示词参考：
Read my entire Obsidian vault. Focus on everything added in the last 7 days.
I want four things:
1. EMERGING THESIS — What idea am I building toward without having stated it explicitly yet? What position is forming in my thinking?
2. CONTRADICTIONS — What have I saved recently that contradicts something I believed before? Show me both sides from my own notes.
3. KNOWLEDGE GAPS — Based on what I am reading and thinking about, what am I clearly not reading that I should be? What perspective is missing?
4. ONE ACTION — Given everything in this vault, what is the single highest-leverage thing I could do or think about this week?
Be direct. Challenge me. Do not summarize what I already know.

第二点「矛盾」最有价值——Claude 会从你自己的笔记里找出你现在相信的和你之前存的东西之间的矛盾，等于强迫你直面自己想法的漏洞。

## 六步搭建清单

按顺序来：

1. 1**建 Obsidian 五目录结构**：Inbox、Notes、Ideas、Projects 四个文件夹 + 根目录 CLAUDE.md。不要加更多文件夹，先跑起来再说。

1. 1**连接 Readwise**：Readwise 有 Obsidian 原生插件，开启之后你在任何地方的高亮自动进 Notes 文件夹。

1. 1**搭 Telegram 捕获 Bot**：用 Claude + N8N，大概 30 分钟搞定。从此手机上冒出的任何想法发给 Bot 就进库了。

1. 1**写 CLAUDE.md**：用上面的模板，诚实填写。Claude 输出质量直接取决于这个文件的质量。

1. 1**配置每日简报 N8N 自动化**：工作日早 6 点运行，输出存到 inbox 文件夹，先于其他事情读它。

1. 1**周一日历里锁定 15 分钟做周度合成**：现在就加进去，不要等「库丰富一点再说」。库永远不会「太空」，空库照样能找到值得想的东西。

## 复利效应从什么时候开始显现

用了一个月：感觉有用，每天简报偶尔冒出来惊喜。

用了三个月：开始感觉不一样。Claude 把三个月前的笔记和今天的问题连起来，你自己早忘了那条笔记。

用了六个月：有一整套关于你如何思考的记录。每个你当时相信但后来改变的观点，每个你一直在想但最终想清楚了的问题，每个在读了大量内容之后才浮现的规律。

越早开始，积累的连接越多，系统对你思维方式的理解越深。这个没什么捷径，只有时间能给。

* 大多数知识库失败的原因：捕获有摩擦、没有连接层、没有理由回来看
* 这套系统的解法：Readwise/Airr/Whisper 自动捕获 + N8N 管道 + Obsidian 存储 + Claude 智能层
* 最关键的文件是 CLAUDE.md，每周更新两个字段就够
* 每天早 6 点自动简报：3个连接 + 1个模式 + 1个问题
* 每周 15 分钟深度合成：挖矛盾、找知识盲区、确定本周最高价值动作
* 今晚先存 5 条笔记，感受一下 Claude 发现连接那一刻——系统就活了

关注「石臻说AI」，持续追踪 AI 工具的实用玩法。

参考链接

* 原推文（CyrilXBT）：https://x.com/cyrilXBT/status/2052235121416188114
* Readwise 官网：https://readwise.io
* N8N 自动化工具：https://n8n.io
* Obsidian 官网：https://obsidian.md

📚 往期精选

[Claude Code 35个实战技巧：每条都有具体命令](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500482&idx=1&sn=929a6424e5c44beec77478184d0b1327&scene=21#wechat_redirect)

[这个开源项目能将任何文件一键转换成MarkDown](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500253&idx=1&sn=5064ed2bb504746f0c6b6c3649aec747&scene=21#wechat_redirect)

[爆火提示词：让AI帮你做色彩和穿搭分析](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500515&idx=1&sn=de4ad1b66f1db62da55d44526d25123d&scene=21#wechat_redirect)

[Claude Code 上下文管理，会不会用这个命令就能判定你是不是高手](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500403&idx=1&sn=3eeaeb1a85fdc1509ed26d4adb738b5d&scene=21#wechat_redirect)

[如何构建正确的架构：瘦Harness，胖Skills](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500389&idx=1&sn=39d3c19a98dbb224ee5df985ea2dc939&scene=21#wechat_redirect)

[Claude Code 推送通知上线，编程进入不用管模式](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500585&idx=1&sn=9efb46a496876802a7881f253cbe7929&scene=21#wechat_redirect)

[Claude Code偷偷把缓存缩水被抓包，用户成本飙升， 这个操作能让你少烧一半配额](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500372&idx=1&sn=7dbfe8ceb885d5542c01526e648b6f4f&scene=21#wechat_redirect)

[别急着卸载 OpenClaw——这套组合拳，让它真正帮你干活](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500611&idx=1&sn=7a213c9f379f01e2cac4160899050bc3&scene=21#wechat_redirect)

[谁能拒绝这么一个Claude桌面宠物？](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500439&idx=1&sn=ee58bef194d0421ef980f55d941d45bb&scene=21#wechat_redirect)

[Opus 4.7发布：更强更准，同样文本token更多](https://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247500416&idx=1&sn=7025e87f486a6c587e873428ab48b897&scene=21#wechat_redirect)

---

— **完** —

**围观朋友圈查看每日最前沿AI资讯**

**一键关注 👇 点亮星标**

**每日科技资讯和提效工具分享**

本文章由快发 api.kuaifa.art 自动排版
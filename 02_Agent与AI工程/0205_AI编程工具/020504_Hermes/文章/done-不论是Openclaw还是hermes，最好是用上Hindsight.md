> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 不论是Openclaw还是hermes，最好是用上Hindsight
author: 开启玄同
date: 玄同玄同
url: https://mp.weixin.qq.com/s?__biz=MzcwOTE4MjQwNg==&mid=2247484137&idx=1&sn=d44b07179435e683a695aea8176b63f4&chksm=f4b45e5c2e7df1961500d150be90868544ca18de04f1b085f57fd2bc4d6a3a7dc6ede9223b55&mpshare=1&scene=24&srcid=0428t558IwlbtxnS92QSeV4A&sharer_shareinfo=1d60a4207ea5ec2b43c1c5d26d07ac1a&sharer_shareinfo_first=1d60a4207ea5ec2b43c1c5d26d07ac1a#rd
---

（不想听我啰嗦的，直接看文末的截图，一段对话就知道用处了）

如果你没有被你的openclaw或者hermes气到崩溃，说明，你用的还是太少了。

上周五我跟一个 AI 助手折腾了一下午代码，第二天打开新 session，它看我的眼神跟第一天一模一样。

不记得我们昨天吵过哪个方案。

我自己飙出来的脏话，不能显著降低我的血压。

Agent很有礼貌——但这让我更生气了。。。

"金鱼记忆"？不不不，——它压根就没有记忆系统。

## 大多数 Agent 的"记忆"，只是更大号的粘贴板

OpenClaw 的内置记忆本质上是一套基于文件的方案：每天的对话存成 `memory/YYYY-MM-DD.md`，长期知识手动维护 `MEMORY.md`，向量检索靠 SQLite。

设计得挺认真。问题是——**全靠 Agent 自己决定"这个要记"**。

文档里写得很清楚："If you want something to stick, ask the bot to write it."

你得主动跟它说"帮我记住这个"。它还得在每次对话里恰好想起来去执行这一步。

现实是：真正重要的细节往往就这么滑走了。Agent 不会觉得"用户上周提到过某个约束"是需要写下来的东西。

## Hindsight 做了什么

简单说：**它把"要不要记"这件事，从 Agent 的脑子里抢过来，交给系统自动干。**

每次对话结束后，Hindsight 自动把内容抓进来，提取成结构化的知识块——事实、实体、关系，而不是一坨平铺的文本。

拿我自己的场景打比方：

* **内置方案：我昨天说"auth 服务用 Redis 存 session"，Agent 没写，今天我得再说一遍**
* **Hindsight 方案：自动提取成一条 "World Fact"，下次问到 auth 服务相关，Hindsight 自动把这条事实注入上下文，Agent 根本不知道它从哪冒出来的**

不需要我喊"帮我记住"。不需要 Agent 下一次想起来调用 search 工具。**记忆这件事，变成了后台自动化流程。**

## 数字能说明一些问题

这个方向最近有一个 benchmark 结果值得关注：

| 方案 | 模型 | LongMemEval 准确率 |
| --- | --- | --- |
| 全上下文基线 | GPT-4o | 60.2% |
| Hindsight | 20B 开源 | **83.6%** |
| Hindsight | Gemini-3 | **91.4%** |

20B 的开源模型，靠记忆架构的优化，超越了 GPT-4o 的全上下文基线。这个数字说的是：**在 Agent 需要"记住跨会话信息"这件事上，系统设计比模型大小更重要。**

数据来源是 LongMemEval 基准测试，arXiv 论文可查（arXiv:2512.12818），Vectorize 团队和华盛顿邮报、弗吉尼亚理工合作发表。

## OpenClaw + Hindsight：两行命令装好

回到 OpenClaw。

4月13日，`hindsight-openclaw` 插件更新到 0.6 版本，最大的变化是**安装体验从手动配置变成了交互式引导**：

```
npx--package@vectorize-io/hindsight-openclawhindsight-openclaw-setup 
```

向导会问你要哪种模式：

* **Cloud：连到 Vectorize 的托管服务，申请 API token 就能用，不需要自己搭服务器**
* **External API：连你自己部署的 Hindsight 服务，多实例共享同一个记忆库**
* **Embedded daemon：在本地机器上跑一个 Hindsight 进程，数据不离开本机（我的hermes用了本地方式，需要docker，但一点不麻烦）**

选完模式，两分钟内配置写入 `~/.openclaw/openclaw.json`，OpenClaw Gateway 自动加载。

整个过程不需要 Docker，不需要懂底层实现。

## 几个我比较感兴趣的功能

**Per-Channel Memory Banks**

Hindsight 会根据 Agent、Channel、User 三个维度自动创建隔离的记忆库。

我在 Slack 里的工作助手，和 Telegram 群里的个人助理，是两套完全独立的记忆。用户 A 和用户 B 互相不干扰。

**JSONL Retain Queue**

对话内容先写入本地 JSONL 文件，API 不通的时候不丢数据，恢复后自动回放。这个对本地部署用户挺实用的。

**Backfill CLI**

历史对话可以批量灌进去。不是只能记住"装好之后的"，之前几周的聊天记录也能一次性迁移。

## 对谁的场景影响最大

如果你用 OpenClaw 做这些事，Hindsight 的价值比较直接：

**多 session 的项目助手**：代码方案吵了两天，新周打开，Agent 还记得当时决定用什么、为什么否掉了哪个选项

**多用户的客服或协作场景**：每个用户的历史偏好、禁忌、上下文自动累积，不需要每次都重新打招呼

**需要跨工具共享记忆的团队**：多个人用各自的 OpenClaw 实例，接同一个 External API Hindsight 服务，知识可以共享

如果你用 OpenClaw 只是偶尔问个问题，用完就走，记忆系统带来的改变感知不强。

## 我的判断

Hindsight 这个方向是对的。

"让模型自己决定记什么"这件事，在实验环境里勉强 work，在真实使用场景里必然漏东西。人会忘，模型也会忘，但系统不应该忘。

把记忆自动化，变成基础设施的一部分，而不是靠 prompt 技巧去补，是正确的工程思路。

GitHub 已经 10K stars（2026年4月22日），还在活跃更新。这个量级的社区认可，不是靠宣传文案能堆出来的。

---

*如果你也在用 OpenClaw或是hermes，可以花两分钟装一下试试。交互向导够友好，不费什么事。*

*[全世界的开发者，都在死磕这个难题：为什么你的AI Agent总是“健忘”？](https://mp.weixin.qq.com/s?__biz=MzcwOTE4MjQwNg==&mid=2247483710&idx=1&sn=a8474b51c6be859b8aa0aca1011b806d&scene=21#wechat_redirect)*

---

—— 玄同

混迹于代码与词语的边界，观察工具如何重塑思考。
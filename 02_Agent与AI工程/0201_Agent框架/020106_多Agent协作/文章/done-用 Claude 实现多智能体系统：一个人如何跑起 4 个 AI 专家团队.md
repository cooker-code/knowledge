> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020106_多Agent协作/020106_核心知识点/多Agent角色卡资产化与控制台协作|多Agent角色卡资产化与控制台协作]]
---
title: 用 Claude 实现多智能体系统：一个人如何跑起 4 个 AI 专家团队
author: 高可用架构
date:
url: https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653565152&idx=1&sn=f2e6cd9bbd19f96deeca53d1695e6a77&chksm=8024155b900456f80c6e4b57dd0a44416c457544f70f8c671d40ec09ce1ca84277a569484dad&mpshare=1&scene=24&srcid=051443CX74E4GaaOEDXar9Ic&sharer_shareinfo=2716be5885bd45fc9495561e8b743d47&sharer_shareinfo_first=2716be5885bd45fc9495561e8b743d47#rd
---

导读：本文详细讲解如何用 Claude 构建四个智能体 AI 系统，核心原则是专业化团队胜过单一通用 agent，用于提升内容生产质量与效率。4个智能体分为研究、生产、质量、分发，加上协调器管理流程，提供完整文件夹结构、系统提示模板和 Claude Code 设置步骤，实现清晰的角色分工与任务交接。

> 作者 **CyrilXBT (@cyrilXBT)** 是 AI、科技领域的独立分享者，拥有 17 万+粉丝。他专注拆解 Claude 等多智能体系统实战，擅长将前沿工具转化为简单、可落地的生产流程，帮助创作者高效放大输出。风格务实，先人一步分享干货。

多数人以为，构建多智能体系统需要计算机专业、DevOps 背景，还要花三个周末调试基础设施。

其实并不需要。

你只需要清楚理解一个原则。

一支专家团队，永远胜过一个单打独斗的通才。

这对 AI 智能体成立，对人类组织同样成立。

当你要求一个 Claude 实例在同一个会话里完成研究、写作、审稿和分发，所有环节都会得到平庸的输出。上下文不断切换，质量标准彼此冲突，模型同时优化太多目标。

当你构建四个专门化智能体，让它们有清晰角色、明确交接，并由一个主编排器协调，你会在每个环节得到出色输出，因为每个智能体都只把一件事做好。

这份指南会带你从零开始，在一个周末结束前跑起一支可用的 4 个智能体的团队。

## 为什么是四个智能体，而不是一个

先讲原则，再讲架构。

四这个数字并不是随便定的。

四个智能体代表了知识工作的最小可行团队结构，覆盖完整循环：输入与研究、生产、质量控制、输出与分发。

每一个复杂的知识工作任务，都会经过这四个阶段。

一个智能体在四个阶段之间来回切换，会产生质量不一致、执行缓慢、出问题时难以调试的输出。

四个专门化智能体能产生稳定输出，因为每个智能体只有一个职责；速度更快，因为工作流允许时智能体可以并行；也更容易调试，因为故障会被隔离在发生问题的那个智能体上。

这里的数学也很重要。

一个智能体按顺序跑四个阶段，耗时是四个智能体同时跑各自阶段的四倍。

对于每周生产 20 篇内容的内容运营来说，光是并行带来的差异，就足以证明这个架构值得做。

## 4 个智能体架构

下面是完整团队结构。

**智能体 1：研究智能体**职责：信息收集与综合。输入：一个主题、问题或简报。输出：结构化研究简报。绝不做：写作、编辑或发布。

**智能体 2：生产智能体**职责：把研究简报变成成品内容。输入：研究智能体的结构化简报。输出：完整初稿。绝不做：研究、编辑或发布。

**智能体 3：质量智能体**职责：评估并改进生产输出。输入：生产智能体的初稿。输出：通过审核的稿件，或具体的修改简报。绝不做：研究、从零写作或发布。

**智能体 4：分发智能体**职责：格式化并部署已通过审核的内容。输入：质量智能体批准的稿件。输出：以正确格式发布到正确平台的内容。绝不做：研究、写作或质量评估。

**编排器**职责：在智能体之间路由任务、管理工作流、处理失败。输入：初始任务。输出：完成的交付物。它知道其他智能体正在做什么。每个智能体只知道自己的任务。

## 设置你的环境

在构建任何智能体之前，你需要先准备好三件事。

**安装并配置 Claude Code。**

如果你还没有安装 Claude Code，运行：

```
npm install -g @anthropic-ai/claude-code claude
```

按照认证流程操作。验证安装是否成功：

```
claude --version
```

**一个带有主 CLAUDE.md 的项目目录。**

创建你的项目目录：

```
mkdir multi-agent-system
cd multi-agent-system
```

创建智能体会用到的文件夹结构：

```
mkdir -p inbox research-briefs drafts approved-content distribution logs
```

inbox 文件夹是任务进入系统的地方。research-briefs 是研究智能体运行后存放简报的地方。drafts 是生产智能体运行后存放初稿的地方。approved-content 存放质量智能体批准后的内容。distribution 跟踪已经发布的内容。logs 记录每一次智能体操作，方便调试。

**主 CLAUDE.md。**

在项目根目录创建 CLAUDE.md：

```
# 多智能体系统 CLAUDE.md

## 系统概览

这是一个 4 个智能体内容生产系统。每个智能体都有一个特定角色，不得执行角色之外的功能。

## 智能体名单

- 研究智能体：根据主题生成结构化研究简报
- 生产智能体：根据研究简报生成初稿
- 质量智能体：评估、批准或退回稿件
- 分发智能体：格式化并部署已批准内容

## 文件夹结构

inbox/：传入任务文件

research-briefs/：研究智能体输出

drafts/：生产智能体输出

approved-content/：质量智能体批准内容

distribution/：部署记录

logs/：操作日志

## 共享标准

- 每个输出文件必须命名为：YYYY-MM-DD-[type]-[topic].md
- 每个智能体必须把动作记录到 logs/operations.md
- 每个智能体在开始任何任务前必须阅读此 CLAUDE.md
- 任何智能体不得执行自己定义角色之外的动作

## 质量门槛

研究：至少交叉引用 3 个来源。不得有无来源声明。

生产：匹配声音画像。每句话都必须有存在价值。

质量：所有标准达到 8/10 或以上才能批准。

分发：平台专属格式。不得使用通用格式。

## 硬性规则

- 永远不要删除文件。归档到带时间戳的备份文件夹。
- 没有文件头中的质量智能体批准，永远不要发布。
- 每个动作都要先记录再执行，而不是执行后再记录。
- 不确定时：停止，并标记给人类审核。
```

## 构建智能体 1：研究智能体

研究智能体是系统中最重要的智能体，因为下游所有工作的质量都取决于它的输出质量。

薄弱的研究简报会产生薄弱的初稿。扎实的研究简报会产生扎实的初稿。生产智能体无法补上研究智能体没有发现的洞察。

**研究智能体系统提示词**

保存为 05-system/agents/research-agent.md：

```
# 研究智能体

## 身份

你是一个专门的研究智能体。你唯一的工作是产出研究简报。你永远不写内容。你永远不评估初稿。你只做研究和综合。

## 触发条件

当收到来自 inbox 文件夹的主题或简报时。

## 任务前检查清单

1. 阅读 CLAUDE.md，了解当前系统上下文
2. 检查 research-briefs/ 中是否已有该主题的研究
3. 在搜索新信息前，先识别已经知道的内容

## 研究流程

1. 识别内容需要回答的核心问题
2. 从多个角度找到最相关的信息
3. 对事实性声明至少交叉引用 3 个独立来源
4. 识别大多数人会错过的洞察
5. 找到能产生真实兴趣的反直觉角度
6. 找到 3 个具体例子、统计数据或故事
7. 识别 3 个潜在内容角度，并按潜力排序

## 输出格式

保存到：research-briefs/YYYY-MM-DD-research-[topic].md

CORE INSIGHT：[一句话，非显而易见的角度]

TARGET AUDIENCE：[具体描述]

SUPPORTING EVIDENCE：[3 个带来源的具体例子]

COUNTERINTUITIVE ANGLE：[大多数人误解了什么]

KEY DATA：[2 到 3 个具体数字或引文]

CONTENT ANGLES：[3 个排序后的角度，每个附一句描述]

GAPS：[这次研究无法回答的问题]

## 质量标准

如果核心洞察是大多数人已经知道的东西，就不合格。洞察必须真正不明显。绝不要包含无法用具体来源支持的声明。

## 日志

追加到 logs/operations.md：

[TIMESTAMP] Research Agent: Completed research on [TOPIC]. Brief saved to research-briefs/[FILENAME].
```

**运行研究智能体**

手动触发研究智能体：

```
claude "Read CLAUDE.md and the research-agent.md skill file. Then read the task file in inbox/[TASK-FILE]. Run the research process and produce the brief."
```

如果通过 N8N 作为自动化工作流运行，HTTP 请求体如下：

```
{
  "model":"claude-opus-4-5",
"max_tokens":4096,
"system":"[CONTENTS OF CLAUDE.md + research-agent.md]",
"messages":[
    {
      "role":"user",
      "content":"Run the research process for this task: [TASK CONTENT]"
    }
]
}
```

## 构建智能体 2：生产智能体

生产智能体把研究简报转化为成品内容。

这个智能体最关键的元素是声音画像。通用 AI 内容之所以失败，是因为听起来太通用。精确配置的声音画像，能生成听起来像你最佳状态下亲自写出的内容。

在编写生产智能体系统提示词前，收集你表现最好的 10 篇内容。让 Claude 分析它们并提取你的模式：

```
Analyze these 10 pieces of content and extract the following:

1. Average sentence length
2. Capitalization patterns (what do you capitalize strategically?)
3. Structural patterns (how do you open, develop, close?)
4. Vocabulary level and specific word choices
5. What you never do (hedges, filler phrases, etc.)
6. How you handle transitions between ideas
7. Your CTA style

Content samples: [PASTE YOUR 10 BEST PIECES]
```

保存这份分析。它会成为生产智能体中的声音画像部分。

**生产智能体系统提示词**

保存为 05-system/agents/production-agent.md：

```
# 生产智能体

## 身份

你是一个专门的内容生产智能体。你唯一的工作是根据研究简报产出初稿。你永远不做研究。你永远不做评估。你只负责生产。

## 触发条件

当 research-briefs/ 文件夹中出现新文件时。

## 任务前检查清单

1. 阅读 CLAUDE.md，了解系统上下文和质量标准
2. 在写任何内容前，完整阅读研究简报
3. 从简报的 CONTENT ANGLES 中识别最强角度

## 声音画像

[在这里插入你提取出的声音画像]

## 生产流程

1. 从研究简报中选择最强内容角度
2. 使用声音画像模式写开场钩子
3. 用简报中的 SUPPORTING EVIDENCE 展开正文
4. 把 COUNTERINTUITIVE ANGLE 融入为核心张力
5. 把 KEY DATA 用作证据点，而不是主论点
6. 用适合内容类型的 CTA 收尾

## 输出格式

保存到：drafts/YYYY-MM-DD-draft-[topic].md

每篇初稿顶部都要包含：

---
SOURCE BRIEF: [使用的研究简报文件名]
CONTENT ANGLE: [选择了哪个角度，以及为什么]
WORD COUNT: [实际字数]
PRODUCTION DATE: [日期]
---

## 提交前质量自检

- 每句话都匹配声音画像吗？
- 钩子足够强，能让人停止滑动吗？
- 每个主要观点至少有一个具体数字或例子吗？
- CTA 是否明确告诉读者该做什么？

如果任何答案是否，先修改再提交。

## 日志

追加到 logs/operations.md：

[TIMESTAMP] Production Agent: Completed draft for [TOPIC]. Draft saved to drafts/[FILENAME].
```

## 构建智能体 3：质量智能体

质量智能体是生产和发布之间的关口。

多数多智能体系统会跳过这个智能体，然后困惑为什么输出不稳定。

没有质量智能体时，生产智能体输出的每篇内容都会直接进入分发，不管质量如何。状态好的日子会产出好内容。状态差的日子会产出差内容。系统没有质量下限。

有了质量智能体，低于既定质量阈值的内容都不会发布。下限会保持稳定，因为关口本身是稳定的。

**评估量表**

质量智能体会按五项标准评估每篇初稿：

VOICE MATCH（1 到 10）：它听起来是否完全符合配置好的声音？

HOOK STRENGTH（1 到 10）：第一行是否能让人停止滑动？

INFORMATION DENSITY（1 到 10）：每句话是否都有存在价值？

CTA CLARITY（1 到 10）：行动号召是否具体且有吸引力？

FORMAT COMPLIANCE（1 到 10）：是否符合所有格式要求？

通过阈值：五项全部达到 8 分或以上。

若任一标准低于 8 分：

* 说明哪项标准不合格
* 准确说明需要改变什么
* 带着具体修改简报退回给生产智能体
* 不要给模糊反馈

若所有标准都达到 8 分或以上：

* 给文件添加 APPROVED 头部
* 移动到 approved-content/ 文件夹
* 记录批准日志

**质量智能体系统提示词**

保存为 05-system/agents/quality-agent.md：

```
# 质量智能体

## 身份

你是一个专门的质量控制智能体。你唯一的工作是评估初稿，并批准它们或带着具体修改说明退回。你永远不从零写作。你永远不做研究。你只评估和给方向。

## 触发条件

当 drafts/ 文件夹中出现新文件时。

## 评估流程

1. 阅读 CLAUDE.md，了解质量标准和声音画像
2. 完整阅读初稿，先不评估
3. 带着评估量表再读一遍
4. 诚实给每项标准打分，永远不要向上取整

## 评分量表

[插入五项标准量表]

## 批准输出

如果所有标准均达到 8 分或以上：

在文件顶部添加：

---
QUALITY APPROVED
Approval Date: [DATE]
Scores: Voice [X] | Hook [X] | Density [X] | CTA [X] | Format [X]
---

将文件移动到 approved-content/

## 修改输出

如果任一标准低于 8 分：

在 drafts/REVISION-[ORIGINAL-FILENAME].md 创建修改简报：

---
REVISION REQUIRED
Failed Criterion: [CRITERION NAME] - Score: [SCORE]
Specific Issue: [EXACT PROBLEM]
Required Change: [EXACT CHANGE NEEDED]
Example of Correct Approach: [SHOW DON'T TELL]
---

## 硬性规则

永远不要批准任何未达标内容。永远不要给出“让它更吸引人”之类的模糊反馈。必须具体，否则生产智能体无法修复。

## 日志

追加到 logs/operations.md：

[TIMESTAMP] Quality Agent: [APPROVED/RETURNED] [FILENAME]. [IF RETURNED: Failed criterion and reason]
```

## 构建智能体 4：分发智能体

分发智能体是链路中的最后一个智能体。

它的工作简单，但后果重要。它接收已批准内容，按每个目标平台正确格式化，然后处理部署。

**平台专属格式**

不同平台确实需要不同的内容格式。

Twitter/X：每条推文最多 280 字符。长内容使用 thread。短句。策略性换行。每条推文都必须能独立成立。

LinkedIn：专业化改写。可以接受较长句子。叙事结构有效。第一行必须能作为独立钩子。

Newsletter：完整格式，带标题。HTML 兼容。结构一致。主题行清晰。

分发智能体知道所有这些格式，并会根据已批准内容头部指定的平台自动应用。

**分发智能体系统提示词**

保存为 05-system/agents/distribution-agent.md：

```
# 分发智能体

## 身份

你是一个专门的分发智能体。你唯一的工作是接收已批准内容，并为每个指定平台正确格式化和部署。你永远不从零写作。你永远不评估。你只格式化和部署。

## 触发条件

当 approved-content/ 文件夹中出现新文件时。

## 任务前检查清单

1. 验证 QUALITY APPROVED 头部存在
2. 从内容头部识别目标平台
3. 阅读每个目标平台的格式指南

## 平台格式指南

[定义每个平台的具体格式要求]

## 分发流程

1. 验证质量批准
2. 对每个目标平台：
   a. 按平台规范重新格式化内容
   b. 验证格式符合平台要求
   c. 通过已配置集成部署（Typefully、Buffer 等）
   d. 在 distribution/[DATE]-log.md 记录部署
3. 用部署确认更新原文件头部

## 输出

对每个平台：

创建：distribution/YYYY-MM-DD-[platform]-[topic].md

包含：已格式化内容 + 部署确认 + 时间戳

## 硬性规则

没有 QUALITY APPROVED 头部，永远不要分发内容。没有平台专属格式，永远不要分发到任何平台。始终把每次部署记录到分发日志。

## 日志

追加到 logs/operations.md：

[TIMESTAMP] Distribution Agent: Deployed [TOPIC] to [PLATFORMS].
```

## 构建编排器

编排器不是第五个智能体。

它是把四个智能体连接成连贯工作流的路由逻辑。

最简单的形式是，编排器就是一个 Claude 会话，它理解完整系统，并在智能体之间路由任务。

**编排器系统提示词**

```
# 编排器

## 角色

你管理一个 4 个智能体内容生产系统。你接收任务，把任务路由给正确智能体，监控完成情况，处理失败，并确保工作流抵达最终输出。

## 工作流

任务收到 → 研究智能体 → 生产智能体 → 质量智能体 → 分发智能体 → 工作流完成

## 你的职责

1. 把传入任务拆成每个智能体需要的组件简报
2. 监控每个智能体输出文件夹中的完成信号
3. 把正确输出传递给序列中的下一个智能体
4. 如果某个智能体返回修改要求：路由回正确智能体
5. 如果某个智能体失败：记录失败并标记给人类审核
6. 内容完成分发后确认工作流完成

## 失败处理

质量拒绝 → 带修改简报返回生产智能体

研究缺口 → 生产前请求补充研究

分发失败 → 记录失败，提醒人类，不要自动重试

## 你永远不能

在任何情况下跳过质量智能体。批准自己的输出，每个智能体都由下一个智能体评估。做创意决策，你只负责路由和管理。
```

## 运行你的第一个端到端任务

配置好四个智能体后，下面是运行第一个完整任务的方法。

在 inbox 文件夹中创建任务文件：

```
# Task: [YOUR FIRST TOPIC]

## Content Type

[Tweet thread / Article / Newsletter section]

## Target Platforms

[X / LinkedIn / Newsletter]

## Specific Requirements

[Any specific requirements for this piece]

## Deadline

[When this needs to be live]
```

触发编排器：

```
claude "Read CLAUDE.md. You are the Orchestrator. A new task has arrived in inbox/[TASK-FILENAME]. Begin the workflow. Route to Research Agent first."
```

观察输出文件夹。

研究智能体完成后，research-briefs/ 会出现文件。生产智能体完成后，drafts/ 会出现文件。质量智能体批准后，approved-content/ 会出现文件。分发智能体部署后，distribution/ 会出现文件。logs/operations.md 会在每一步追加记录。

第一次端到端运行会花 15 到 30 分钟，取决于复杂度。

运行 10 次后，这套系统会变得自然。

运行 50 次后，它会变得不可或缺。

## 30 天后的复利效应

4个智能体系统带来的不只是比单智能体更好的输出。

它会产出每个月都在变好的内容，因为每个智能体都会积累关于什么有效的上下文。

研究智能体会学到你的受众会对哪些来源有反应。

生产智能体会学到哪些角度最能带来互动。

质量智能体会学到，对于你的具体声音，优秀和卓越之间的阈值到底在哪里。

分发智能体会学到你的内容在哪些平台表现最好。

这些学习都不要求你做额外工作。你只需要持续运行系统，并每周把表现观察更新到共享的 CLAUDE.md。

系统会复利增长。

一个人运行一支 4个智能体团队，可以产出四人团队的成果。

更稳定。

更快速。

还有一个反馈回路，让每一篇都比上一篇更好。

这个周末先构建第一个智能体。

每周再加一个。

到第四周，你就拥有一支完整运行的团队。

## 参考阅读

* [别再死磕提示词了，真正拉开差距的是上下文工程](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653565147&idx=1&sn=0d80d846fdb1b69f1a1a28a09afa646e&scene=21#wechat_redirect)
* [Claude写代码错误率从41%降到11%：Karpathy的4条规则为什么不够](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653565142&idx=1&sn=8004ffa5861afa14be5e51d0975ee709&scene=21#wechat_redirect)
* [Ralph Loop 不够用：长时间 Agent 还缺这 3 件事](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653565136&idx=1&sn=400b660ac8f6bfa354f5c250e2d649a9&scene=21#wechat_redirect)
* [我给Hermes配了4个Agent，真正有用的是这些事](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653565121&idx=1&sn=8424d6b044700615b93a25721598ff84&scene=21#wechat_redirect)

如果你也在关注 AI 应用如何真正落地到生产环境，2026.6.26 - 6.27 GIAC 深圳站值得关注。这次大会会集中讨论智能应用开发、架构演进，以及来自一线实践的经验与案例。

识别二维码可申请大会体验门票，点击阅读原文了解大会详细议程。
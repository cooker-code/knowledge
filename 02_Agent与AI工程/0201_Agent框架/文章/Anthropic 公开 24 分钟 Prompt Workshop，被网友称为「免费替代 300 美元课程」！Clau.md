---
title: Anthropic 公开 24 分钟 Prompt Workshop，被网友称为「免费替代 300 美元课程」！Claude 团队到底教了什么？
author: 虚拟灵枢
date: 馨语田园馨语田园
url: https://mp.weixin.qq.com/s?__biz=MzkwMjY4MzAzMQ==&mid=2247490680&idx=1&sn=f9bc12a5618483536cec0d0618b70223&chksm=c1941a709db00a504cdc06dc8017e688c79675c0db55c11e705d2a238eb9234e3320ade1b0ec&mpshare=1&scene=24&srcid=05227yuoootkEMyfTZUhjPKN&sharer_shareinfo=e0729df48ba8f65cfab23711bd738ac9&sharer_shareinfo_first=e0729df48ba8f65cfab23711bd738ac9#rd
---

导读  
一段 2025 年的 Anthropic 官方 workshop 视频被 X 用户重新挖出，80 万浏览、1.6 万收藏，评价是"前 8 分钟就超过很多 300 美元课程"。Claude 团队在 24 分钟里跳过了 prompt 模板，直接用一个瑞典车祸报告的真实案例，演示了一套完整的 prompt 工程化流程。

## 80 万浏览的"旧视频"

5 月 15 日，X 用户 Anuj（@anujcodes\_21）发了一条帖子：

> "Anthropic just showed a 24-minute workshop on how to actually do prompts for Claude. Taught by the people who built it. Free. No registration. No paywall. I've seen $300 courses that don't cover what they teach in the first 8 minutes."

「Anthropic 展示了一个 24 分钟的 workshop，教你怎么给 Claude 写 prompt。由做 Claude 的人亲自讲。免费，无需注册，没有付费墙。我见过一些 300 美元课程，前 8 分钟讲的都不如它。」

▲ Anuj 的推文采集时已超过 80 万浏览、1.6 万收藏

帖子迅速在开发者圈子里扩散。但有人很快指出：**这个视频并不新。**

X 用户 @primarycongress 回复说"这是 11 个月前的视频"，@rmsm1th 也确认它是 2025 年发布的。YouTube metadata 显示，视频上传于 2025 年 7 月 31 日，内容来自 2025 年 5 月 22 日 Anthropic 在旧金山举办的 Code w/ Claude 活动。

所以准确地说——这是一段被重新发现的官方 workshop，被重新包装后引爆了社交媒体。

但它为什么能火？因为它触碰到了一个真实痛点：**在 AI 课程动辄几百美元的市场里，模型厂商自己的官方教学，反而是免费公开的。**

## Claude 团队 24 分钟里到底讲了什么？

这段视频的标题是 `Prompting 101 | Code w/ Claude`，主讲人是 Anthropic 应用 AI 团队的 Hannah Moran 和 Christian Ryan。

▲ Anthropic 官方 YouTube 频道，视频采集时已有 34 万次观看

他们开场就定义了 prompt engineering：

> "prompt engineering... is the practice of writing clear instructions for the model, giving the model the context that it needs to complete the task, and thinking through how we want to arrange that information"

「prompt engineering 就是给模型写清楚指令、提供完成任务所需的上下文，并思考如何组织这些信息。」

注意，这里没有"万能咒语"、没有"10 个 prompt 秘诀"。**他们把 prompt engineering 定义成了一个工程问题。**

接下来，他们用了一个非常具象的案例来演示整个流程——让 Claude 读一份瑞典语的车祸事故报告表格和一张手绘事故示意图，判断事故责任。

**第一层：先告诉模型"你来干嘛的"**

> "What are you here to do? What's your role? What task are you trying to accomplish today?"

任务描述和角色定义放在 prompt 最前面。这一步把模型从通用聊天模式拉进具体业务场景。

**第二层：喂上下文，区分动态内容和固定背景**

动态内容是每次不同的事故表格和草图；固定背景是表格结构说明——每一行什么意思、可能出现手写/圈选/涂改等情况。

视频里明确说，这类固定背景**特别适合放进 system prompt**，也适合做 prompt caching（缓存），因为它每次调用都一样。

**第三层：步骤顺序很关键**

讲者反复强调**order matters**——先读表格、确认勾选项，再读草图，最后把事实和推理对齐。

这个点很值得展开：很多人写 prompt 只关心"写了什么"，忽略了"按什么顺序写"。**你给模型的处理顺序，本质上就是你把人类专家的工作流程写给了它。**

**第四层：用 XML tags 做结构化分隔**

> "Claude really loves structure, loves organization... delimiters like XML tags..."

「Claude 非常吃结构和组织，XML tags 等分隔符能帮它理解每块信息的边界。」

XML tags 的作用是告诉模型"这段是什么"，方便后续引用和解析。这也是 Anthropic 官方文档里反复推荐的做法。

**第五层：用 examples 引导模型判断**

Christian Ryan 特别强调了 few-shot examples 的作用：

> "examples or few shot is a mechanism that really is powerful in steering Claude."

「examples/few-shot 是引导 Claude 的强力机制。」

尤其是那些**模型容易判断错的灰色案例**，放进 system prompt 当参照，下次遇到相似情况就有锚点。

**第六层：防幻觉——该说"不确定"就说"不确定"**

> "preventing hallucinations... not invent details that it's not finding in this prompt."

「防止幻觉——不要编造 prompt 里没有的细节。」

视频建议在 prompt 尾部加一个 reminder：如果表格或草图看不清，承认无法确定，而不要猜。

**高价值 prompt 的核心，恰恰在于让模型在该沉默的时候沉默。**

**第七层：输出格式要对接下游系统**

最后一步是让 Claude 把最终结论包在 XML tags 里，或者用 prefill/JSON 格式输出——目的是让结果可以直接进数据库、进分析系统、进下一个流程。

视频末尾还提到，Claude 的 extended thinking（扩展思考）可以当调试工具：观察模型怎么一步步处理数据，再把有用的推理步骤固化回 system prompt。

## 官方文档早就写好了，只是没人看

视频里教的这些技巧，和 Anthropic 官方文档高度一致。

▲ Anthropic 官方 Prompting Best Practices 文档页面

Anthropic 的 Claude API Docs 里有两个关键页面：

**Prompt engineering overview**在第一行就要求使用者先准备好三样东西：成功标准的明确定义、可经验测试的评估方法、一个初稿 prompt。也就是说——**官方根本不鼓励"凭感觉调 prompt"**，在他们的体系里，prompt engineering 排在 success criteria 和 evals 之后。

**Prompting best practices**覆盖了清晰度、示例、XML 结构、思考链、agentic systems、tool use、输出控制等完整技术栈。

> "Comprehensive guide to prompt engineering techniques for Claude's latest models, covering clarity, examples, XML structuring, thinking, and agentic systems."

「面向 Claude 最新模型的 prompt engineering 技术综合指南，涵盖清晰度、示例、XML 结构化、思考链和 agentic systems。」

视频和文档形成了一个完整体系：视频负责"案例演示"，文档负责"长期维护的技术规范"。

## 免费资源远不止一个视频

Anthropic 在 GitHub 上还维护着一个 `prompt-eng-interactive-tutorial` 仓库，创建于 2024 年 4 月，采集时已有**3.5 万 stars、3800+ forks**。

▲ Anthropic 官方 Prompt Engineering Interactive Tutorial 仓库

README 开头写道：

> "This course is intended to provide you with a comprehensive step-by-step understanding of how to engineer optimal prompts within Claude."

「本课程旨在帮你系统性、逐步理解如何为 Claude 编写最优 prompt。」

仓库分成多个章节，配有练习和实验环境——这已经比很多付费课程的交互性还强了。

加上视频、官方文档、GitHub 教程，Anthropic 事实上已经构建了一整套免费开放的 prompt engineering 教学体系。对付费课程构成真正挑战的，是模型厂商持续把底层方法论公开——这比任何网红推荐都有杀伤力。

## 社区反应：认可与纠偏并存

认可的声音很多。X 用户 @maxpostingx 说，直接向模型创建者学 prompting，比市面上大多数昂贵的"AI 大师课"更有价值。@kekkodamato\_ 表示自己从真实 system prompts 里学到最多，但承认官方 workshop 正在填补"第一方教学"的空白。

纠偏的声音同样有力。多位用户指出视频并非新发布，而是快一年前的旧内容。还有 @GamerGrind 和 @JitK9161 吐槽视频画质只有 360p——免费公开的官方资源，呈现形式确实谈不上精致。

但这些争议恰恰说明了一件事：**开发者群体对"第一方教学资源"有强烈需求，同时对信息的时效性和包装话术保持着健康的怀疑。**

## 300 美元课程的尴尬

回到 Anuj 那句"300 美元课程前 8 分钟都不如它"——这个说法没有具体课程名或可验证的价格对比，只能当作个人观感。

但它之所以引起共鸣，是因为 AI 课程市场确实存在一个结构性问题：**很多高价课程卖的是 prompt 模板和技巧清单，而模型厂商官方教的是 prompt 工程流程——定义成功标准、提供上下文、约束推理顺序、结构化输出、用评估迭代。**

前者是"鱼"，后者是"渔"。

当"渔"变成免费公开资源的时候，"鱼"的定价就变得很难维持了。

---

— END —

— END —
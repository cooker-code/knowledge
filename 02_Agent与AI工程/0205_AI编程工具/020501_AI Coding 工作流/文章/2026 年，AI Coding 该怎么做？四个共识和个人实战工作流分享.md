---
title: 2026 年，AI Coding 该怎么做？四个共识和个人实战工作流分享
author: AgenticCoding 实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485504&idx=1&sn=73691f98b9e56e31b8dea3b5b4d3a59a&chksm=9a19d5f559f7f1aa0cd45d8cb7a464d5375fc87abe0ff95c65915b2b3166af75b9f7a0007b9f&mpshare=1&scene=24&srcid=0414h5xy7KcWWNmcysgBqhkW&sharer_shareinfo=3365b8e74a282f751a4a40458eef6843&sharer_shareinfo_first=3365b8e74a282f751a4a40458eef6843#rd
---

# 2026 年，AI Coding 该怎么做？四个共识和个人实战工作流分享

2026 年，明显感觉到 AI Coding 这个领域已经达到了一个相对平稳的阶段，开始转向一些更难解决的领域（Memory 怎么做、Infra 怎么搭），甚至向着非 Coding 领域发展（Claw 类产品的大火）。

反而是这个时间点，特别适合回头看一看：AI Coding 到底该怎么用？我整理了一些个人观点和实践，算是给自己做个阶段性总结，也希望对你有些参考。

## 四个共识

时至今日，社区对 AI Coding 的实践已经逐步达成共识，个人总结下来有四点。

### 1. 先规划（Plan）后执行（Act）

动手写代码之前，完整的设计文档（包含需求、实现、验证的细节）至关重要，这是 Coding 的基础。

不然即使有知识库、Skills，Agent 大概率还是会跑偏，带来的结果就是更多轮的对话，更糟糕的结果以及更多的 Token 浪费。Claude Code、Cursor 等 Agent 都默认内置 Plan 模式的，就是在引导用户先想清楚再动手。

### 2. Coding Agent 三要素：AGNETS.md(CLAUDE.md) + Skills + Memory

在使用一个成熟的 Coding Agent 时，用户本质上只需要关注三个点：

* • **AGNETS.md(CLAUDE.md) 是全局规则。** 从泄露的 Claude Code 源码中可以看到，每轮对话都会将 CLAUDE.md 的内容完整的注入到用户消息前面，不受上下文压缩影响，可见其含金量。
* • **Skills 已经是 Agent 的核心能力了。** 我们之前翻译过 [Anthropic](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485202&idx=1&sn=bd82c40b435d8cdf73a5fbaa309071ce&scene=21#wechat_redirect)、[OpenAI](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485055&idx=1&sn=fc2b8190180615c59dc43f8a9321704d&scene=21#wechat_redirect)、[Google](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485230&idx=1&sn=78c67ccf85bb72124e427f7f851b4843&scene=21#wechat_redirect) 多家的 Skills 实战。从不同角度讲述 Skills 的实践经验。
* • **Memory 是目前 Agent 能力的最后一环**。有了记忆，Agent 可以记录你的编码习惯偏好、决策习惯、踩坑点。不过至今依旧没有一个统一解决方案，个人认为是属于"很重要但不紧急"的那类事。（虽然大多数时候 CLAUDE.md 也是记忆的一部分，但这里还是区别开来）

### 3. 所有的问题都可以归结文上下文的问题（Context）

很多时候抱怨 Agent 效果不行时，其实需要关注一下是不是没有给 Agent 提供足够清晰的上下文：

* • 是否有清晰的需求描述
* • 是否提供了内部独有的知识库
* • 上下文窗口是否溢出且没有正确压缩  
  上下文不够清晰，又没有主动引导 AI 去确认和澄清，Agent 大概率会自己编一个"看起来合理"的结果出来。

### 4. 研发流程本身并没有变

OpenAI 指出，软件研发生命周期（SDLC）- 规划、设计、开发、测试、代码审查和部署整个流程并没有变化，工程的某些核心要素也没变，代码的真正所有权——特别是面对新问题或模糊需求时——仍牢牢掌握在工程师手中，某些挑战也仍超出当前模型的能力范围。

但在各个阶段 AI 都可以起到不同程度的辅助作用。

## 个人实战工作流分享

基于以上几点共识，社区也已经有很多解决方案了。我们之前也介绍过不少：

* • [把资深工程师的思维装进 AI：深度解析谷歌 Agent Skills 项目](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485466&idx=1&sn=2d0e1668411858d6918367b2a4324294&scene=21#wechat_redirect)
* • [Superpowers：让 Claude Code 拥有"超能力"的 120K 星插件](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485377&idx=1&sn=77a27b13cd3e236dfa5236f2e7907c0a&scene=21#wechat_redirect)
* • [我对比了 gstack、Superpowers 和 Compound Engineering，它们解决的是三个完全不同的问题](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247485386&idx=1&sn=2fa04d0d036a1c92541349907d65946a&scene=21#wechat_redirect)

而且还不断有很多新的方案出现，这就出现一个选择难题了，该用哪个，该怎么组合使用。这里我分享一下自己的做法，参考已有方案，组合出适合自己业务的工作流。

### 研发流程类：以 feature\_dev 为主干

既然软件研发流程基本不变，那我们就可以将各个阶段抽象为 Skill 使用。

我的主力解决方案复用Claude Code 官方插件[1]的 **feature\_dev** plugin，这个 plugin 非常精炼，核心只有一条 command：`feature_dev.md`，但却完整覆盖了研发流程的各个阶段：

**Phase 1: Discovery**

这一步接受用户初始需求，并进行理解和澄清，如果不清晰需要跟用户确认。

**Phase 2: Codebase Exploration**

在理解用户需求之后，带着用户需求去与探索代码库，这里不仅仅是使用自带的 Explore Agent 去探索代码库，而是会并行启动 2～3 个 code-explorer Agent（plugin 内置）分别从不同的角度去阅读代码：

* • 「查找与 [功能] 相似的功能，并全面追踪其实现」
* • 「梳理 [功能领域] 的架构与抽象，并全面追踪代码」
* • 「分析 [现有功能/领域] 的当前实现，并全面追踪代码」
* • 「找出与 [功能] 相关的 UI 模式、测试方式或扩展点」

然后综合各个 Agent 的结果，标记一些重要的文件，主 Agent 会再次阅读，以得出最终探索的结论。

**Phase 3: Clarifying Questions**

这个阶段根据用户需求和对代码库的理解，再次与用户对齐求，确保在代码设计阶段和用户完全达成一致。

**Phase 4: Architecture Design**

接下来就进入架构阶段了，这个阶段也并不是只设计一个方案，同样并行启动 2～3 个 code-architect Agent，且侧重点不同：最小改动（改动最小、复用最大）、整洁架构（可维护、抽象优雅）、或务实平衡（速度 + 质量）。

主 Agent 评估各个方案，给用户进行推荐。但最终还是由用户进行确认。

**Phase 5: Implementation**

按照选定的架构开始编码。约束很明确：遵守架构方案、遵守代码库规范、保持代码清晰。

**Phase 6: Quality Review**

代码审查阶段，并行启动 3 个 code-reviewer Agent，侧重点分别为：简洁/DRY/优雅、缺陷/功能正确性、项目约定/抽象，最后由主 Agent 汇总结论，根据优先级和用户意见逐一修复。

**Phase 7: Summary**

总结记录：已经做的、关键决策、修改了什么以及后续有哪些建议。

整个 command 写的很简短，7 个阶段也就 120 行左右，不需要长篇大论。我在自己的项目中频繁使用这个流程，经过前四阶段谨慎的澄清和对焦，基本可以做到一把过不走偏。

### 研发流程优化

**阶段组合**

大多数工具会给每个阶段提供一个单独的 skill 或者 command 的，这样更清晰一些，我在内部也是这样做的，将 feature\_dev 的 7 个阶段重新组合成了 4 条命令：

1. 1. `/plan`，合并前四个阶段，进行规划，并将架构方案输出文档按【时间+需求】维度输出到本地。
2. 2. `/act`，Implementation 阶段
3. 3. `/review`，Review 阶段
4. 4. `/summary` 总结阶段，总结并输出文档到本地

文档输出到本地有一个好处就是有时候我在查找相关内容时可以直接参考文档，另外如果该仓库和其他仓库有对接，这里甚至可以当对接文档使用。

**头脑风暴模式**

Plan 阶段其实会存在大量和用户进行确认的场景，如果你的需求描述没有那么清晰，更想抱着一种和 AI 进行深度讨论的心态，那其实 AI 一次性会跟你进行大量提问，这时候其实可能有点无从下手。

**Superpowers**提供了一个很好的思路，头脑风暴[2]。

头脑风暴模式特别适合讨论：

* • 不一次性列出所有问题，而是逐一提问，一步步帮你完善想法
* • 即使是开放性问题，也会尽量给出选项减少用户压力。
* • 遵守YAGNI原则 (You Aren't Gonna Need It)，如无必要勿增实体。  
  总之保持逐步增强的进行需求澄清和架构确认。

我把这个模式组合到了 Plan 阶段中。在内部项目里，一些需要反复讨论才能定方案的新 feature，用头脑风暴模式推进会轻松很多。

> Superpowers 其实也遵循头脑风暴 → 规划 → 执行 → 审查的标准流程，你同样可以复用他的流程作为你的日常主力流程

**总结升级为知识管理**

默认的 summary 指令只记录"做了什么、怎么决策的、有什么建议"。但这可能不够，在大型项目中，除了关键决策，可能还有新学到的私域知识、通过 bug 学到的易踩坑点。这些知识不仅仅要作为总结文档后续查阅，更应该被后续的编码流程记住。

恰好 Compound Engineering（CE）的亮点就是 Compound，它的理念就藏在名字里，Compound，复利，他的流程也是标准的头脑风暴 → 规划 → 工作 → 审查 → 复利，在最后一步，它会把有价值的知识（解决方案、bug 记录、踩坑点等）沉淀到本地文档。更重要的，写完文档后，它会在项目的 AGENTS.md/CLAUDE.md 添加指引，引导 Agent 去搜索这些文档。

前面说了，AGENTS.md/CLAUDE.md 是最高准则，写进去的东西 Agent 一定会看到。

我参考 CE 的最后一个阶段[3]丰富了一下 summary，在总结文档中增加了知识沉淀模块：

* • 哪些模式值得复用
* • 哪些做法要避免
* • 踩了什么坑，怎么解决的
* • 关键知识同步到 CLAUDE.md

### 业务类：私域知识才是关键

通用的研发流程 Skills 只能解决共性问题。企业内部那些私域知识和专属流程，往往对 AI Coding 的效果影响更大。

内部知识库方面，组件库、工具库、内部系统对接，这些内容需要 AI 在合适的时机感知到。OpenAI SDK 团队有一个 `openai-knowledge` skill，通过内部 MCP 拉取文档，我们完全可以参考这个思路。

专属 Skill 方面，每家公司都有自己的场景：

* • 需求开始前在平台新建迭代
* • Figma 设计稿转为组件代码
* • HTTP 请求的模板代码生成
* • 数据埋点统计

这些因项目、因公司而异。把身边那些可以自动化的事情沉淀为 Skill，就是对研发流程最实在的增强。

### 工具类

生成图片、Git 操作、文档编写、浏览器自动化、前端页面设计……这些通用能力就不展开了，社区里有大量现成的 Skills 可以直接用。

## 总结

这篇文章算是抛砖引玉，我把自己的思考和实践流程整理了出来，后续也会把完整的解决方案开源。

但说到底，最好的方案一定是从你自己的实践中长出来的。

欢迎一起讨论~~~

#### 引用链接

`[1]` Claude Code 官方插件: *https://github.com/anthropics/claude-plugins-official*  
`[2]` 头脑风暴: *https://github.com/obra/superpowers/tree/main/skills/brainstorming*  
`[3]` CE 的最后一个阶段: *https://github.com/EveryInc/compound-engineering-plugin/blob/main/plugins/compound-engineering/skills/ce-compound/SKILL.md*
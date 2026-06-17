---
title: Markdown 还救得动吗？Claude Code 团队成员公开「反水」，甩出 20 个 HTML 实例，1100 万人围观！
author: 井底之硅
date: 洛杉矶太守洛杉矶太守
url: https://mp.weixin.qq.com/s?__biz=MzIyOTE4MTgxNA==&mid=2247495305&idx=1&sn=3e81bf272dfbe43bbc0e86b326254476&chksm=e9eedcb363373fe0803e498efa9be102f43be12edc0b566083e67c5ed51db0cb93feac3dbf05&mpshare=1&scene=24&srcid=0513TPpv3W96t3gFx99SuY2w&sharer_shareinfo=2e7738fc779f0c3818b19860dfa97a5b&sharer_shareinfo_first=2e7738fc779f0c3818b19860dfa97a5b#rd
---

导读  
【导读】Claude Code 团队成员 Thariq 发文宣称自己越来越偏好 HTML 输出，并放出 20 个 HTML artifact 实例，浏览量突破 1100 万。Hacker News 的讨论也在同期升温，反方意见迅速被推到台面上——HTML 更强，但你改得动吗？

## 「我越来越不想用 Markdown 了」

这场争论的起点，来自 Claude Code 团队成员 Thariq（@trq212）在 X 上发的一篇长文。

标题就够刺激：**《Using Claude Code: The Unreasonable Effectiveness of HTML》**。

> "I've started preferring HTML as an output format instead of Markdown and increasingly see this being used by others on the Claude Code team, this is why."

「我开始更偏好 HTML 作为输出格式，在 Claude Code 团队里这种做法也越来越常见。下面是原因。」

Thariq 的核心判断是：当 AI agent 能做的事情越来越复杂，Markdown 这个"万年默认格式"已经撑不住了。

他承认 Markdown 有经典优势——简单、可移植、易编辑，Claude 甚至能在 Markdown 里用 ASCII 画图。但随着 agent 输出的复杂度不断攀升，这套介质越来越力不从心。他的原话：

> "The chance of someone actually reading your spec, report or PR writeup is much much higher if it's in HTML."

「如果你的 spec、报告或 PR 说明是用 HTML 写的，别人真正去读完的概率会高得多。」

最扎心的一点在这里：**你写的东西再好，没人读完就等于零。**

他把自己偏好 HTML 的理由拆成了六层：

* **信息密度**

  ：HTML 能承载表格、SVG、空间布局、脚本、交互，不再只靠线性文本堆叠
* **视觉清晰度**

  ：复杂的 spec / report / plan 更容易被真正读完
* **分享便利性**

  ：HTML 文件上传后可以直接发链接，Markdown 往往要靠附件或平台二次渲染
* **双向交互**

  ：可以加 slider、knob、copy button，读者能调参数、能操作，不只是干看
* **数据消化**

  ：Claude Code 有大量上下文入口，适合把海量信息整理成浏览器里就能读的单文件产物
* **乐趣**

  ：Thariq 明确说这让他"更 in the loop"——这点主观，但解释了为什么用户体验上的满足感能压过 token 焦虑

## 不只是发了个帖——他放出了 20 个实例

如果只看 X 上的文字，你可能觉得这只是个人偏好的表态。但 Thariq 同时上线了一个 examples 站点，把争论从"观点"拉到了"可验证"的层面。

▲ Thariq 的 HTML artifact 实例站点首页，标语：「Twenty self-contained .html files an agent produced instead of a wall of markdown.」

站点收录了 20 个单文件 HTML 产物，按用途分了九大类：

* **Exploration & Planning**

  ：规划类内容做成 side-by-side 对比和数据流程图
* **Code Review**

  ：代码审查做成带注释的 annotated diff 和模块地图
* **Design / Prototyping**

  ：设计文档直接变成可交互原型
* **Diagrams / Decks**

  ：架构图和演示材料一步到位
* **Research / Reports**

  ：研究报告自带 glossary、diagram、FAQ 导航
* **Custom Editors**

  ：临时编辑器，人类可以在上面拖拽、选择、排序，再把结果导回 prompt

Thariq 推的方向很明确：**让 agent 产出一个可操作的工作台，而不只是一份等人翻阅的文档。**

## Simon Willison 追加关键背景：以前省 token，现在该省人脑了

知名开发者 Simon Willison 第一时间在博客上做了评论，同时确认了 Thariq 的身份——Claude Code team at Anthropic。

▲ Simon Willison 在博客中回顾了 GPT-4 时代默认 Markdown 的历史，并重新审视 HTML 输出的可能性

Simon 的切入角度特别有价值。他说自己从 GPT-4 时代起就默认让模型输出 Markdown，原因很现实：

> "I've been defaulting to asking for most things in Markdown since the GPT-4 days, when the 8,192 token limit meant that Markdown's token-efficiency over HTML was extremely worthwhile."

「从 GPT-4 时代起，我大多数场景都默认用 Markdown，因为在 8,192 token 上限下，Markdown 的 token 效率远高于 HTML，非常值钱。」

**过去选 Markdown，跟表达力无关——token 太贵、上下文太小，只能将就最省字符的格式。**

现在情况变了。上下文窗口越来越大，模型的前端生成能力越来越稳。用 Simon 的话说，让 Claude 输出 HTML 格式的解释，可以自然地嵌入 SVG 图表、交互控件、页内导航——这些在 Markdown 里根本做不到。

以前的"格式之争"本质上是 token 经济学。现在 token 瓶颈在弱化，**真正贵的变成了人的阅读时间和理解成本**。

## Hacker News 上的反击：HTML 更强没错，但你改得动吗？

讨论很快扩散到 Hacker News 首页。

▲ Hacker News 讨论页面，高赞评论迅速聚焦到「协作编辑成本 vs 表达力」的核心分歧

支持 HTML 的开发者认为，agent 输出更接近可交互的软件对象——不再只是一段让人硬啃的长文本。

但反方火力同样集中。最高赞的几条反对意见指向同一个问题：**你一个人用 HTML 产物确实爽，但团队要改怎么办？**

三个最核心的痛点：

* **协作门槛上升**

  ：Markdown 谁都能改，HTML 需要懂标签和结构，co-author 的成本直接翻倍
* **Diff 噪音更大**

  ：HTML 的 diff 天然比 Markdown 吵得多，PR review 的负担显著增加
* **长期维护更重**

  ：需求文档、技术规范这类需要反复演进的内容，Markdown 的可直接修改性目前无可替代

还有一派折中观点获得了大量认同：**不必在 HTML 和 Markdown 之间做非此即彼的选择。**

* Markdown 可以内嵌 HTML
* 用 pandoc 之类的构建工具，保留 Markdown 的编辑性，同时获得 HTML 的展示能力
* 把源格式和最终展示层分开——Markdown 管源文件，HTML 管最后一步渲染

## Thariq 自己也承认：HTML 确实更慢

在原帖的 FAQ 部分，Thariq 给出了直接回应：

> "This does take longer! HTML can take 2-4x longer than Markdown, but I've found the results are worth it."

「确实更慢。HTML 的生成时间可能是 Markdown 的 2 到 4 倍。但我觉得结果值得。」

**2 到 4 倍。**实打实的代价。agent 每次生成一个 HTML artifact，消耗的计算资源、等待时间、token 用量都在翻倍。

但 Thariq 的逻辑是：如果最终产物没人读完，那再快也白生成。在他看来，花 2 倍时间生成一个别人真正会打开、会看完、会动手用的产物，比瞬间吐出一份被忽略的 Markdown 文档要划算得多。

这笔账到底怎么算，取决于你的场景。

## 官方怎么说？Anthropic 划了一条边界

有人可能会问：既然 Claude Code 团队成员都这么说了，Anthropic 官方也准备抛弃 Markdown 了？

答案是**没有**。

▲ Claude Code 官方文档 Output Styles 页面，明确将输出样式定义为系统提示层的可配置能力

Claude Code 官方文档的《Output Styles》页面明确写道：

> "Output styles change how Claude responds, not what Claude knows."

「Output styles 改变的是 Claude 的响应方式，不影响它的知识储备。」

文档显示，Claude Code 在 Default output style 之外，还提供 Proactive、Explanatory、Learning 三种内建样式，全部通过系统提示层配置。

这里有两个关键信号：

**第一，**Anthropic 确实把"输出格式"当成产品级能力来设计，它属于 agent 接口的正式组成部分。

**第二，**但官方文档没有任何地方宣布 HTML 成为默认，或 Markdown 被弃用。

Thariq 的实践和官方的产品方向处于同一趋势中，但他的选择代表的是前沿用户的偏好探索，不等于产品公告。

## 还有一层被忽略的问题：安全

社交媒体和 HN 上的讨论主要围绕阅读效率、协作成本和生成速度展开。但对企业团队来说，还有一层问题绕不过去：**安全与治理**。

* agent 生成的 HTML 能不能包含脚本？
* 同事可以直接打开吗？需不需要沙箱隔离？
* 如果 artifact 最终要落知识库或 PR 附件，安全审查流程允许 HTML 格式吗？
* XSS 风险、CSS 注入、隐藏链接——这些在 Markdown 时代几乎不用考虑的事情，到了 HTML 产物里全部变成了真实威胁

目前还没有标准答案。但这些问题构成了 HTML 短期内不会全面取代 Markdown 的现实制约。

## 成本没有消失，只是换了一个位置

回到最核心的问题：这场争论到底在争什么？

表面上看是格式之争，HTML 对 Markdown。但真正的分歧在于"上下文成本"往哪里算。

过去，大家关心的是 token 成本——模型能输出多少字，上下文能放多大，Markdown 在这方面效率更高。

现在，上下文窗口够大了，模型生成前端也够稳了。但新的成本冒了出来：

* 生成更慢（2-4 倍）
* Diff 更吵
* 手工修订更麻烦
* 安全审查面更大
* 团队文档标准可能进一步分裂

**HTML 用更高的生成与维护成本，换来了更高的阅读与理解效率。**Markdown 的成本更像机器账——省 token、省渲染。HTML 的收益更像人类账——省阅读时间、省认知摩擦。

哪个更值？取决于你的产物是一次性交付还是反复修改，取决于你的读者是一个人还是一个团队，取决于你的 agent 有多强的上下文消化能力。

**Markdown 没有死。但"所有 agent 产物都先落成 Markdown"的旧默认，正在被动摇。**

当模型开始替你消化海量上下文时，最终瓶颈已经从"模型能不能写出来"转移到了"人类能不能顺畅把它读进去、改进去、用起来"。

---

— END —

— END —
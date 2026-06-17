---
title: Tenex 软件工程师的工作流：Claude 负责写，Codex 负责骂-How I AI
author: Lenny Podcast
date: Lennys PodcastLennys Podcast
url: https://mp.weixin.qq.com/s?__biz=MzY5MTE1ODU1NA==&mid=2247484952&idx=1&sn=1d9db2086e8f0e7f303adeeedb489bbe&chksm=f56cd4969a7f5b4878791657f02af8d1ea2209261ca50e7c832303354486c23c26498e5ec6f0&mpshare=1&scene=24&srcid=0602C1KgJr983NKdyKpzGLVD&sharer_shareinfo=2beafb42681ce4f951ea40efeed567dc&sharer_shareinfo_first=2beafb42681ce4f951ea40efeed567dc#rd
---

# 

软件工程师 CJ Hess 展示了两个非常强大的 AI 工作流：第一，构建一个名为 Flowy 的自定义可视化规划工具，用来引导 Claude Code；第二，使用 GPT-5.2 Codex 对 AI 生成的代码做质量控制，从而得到真正能进入生产环境的结果。

欢迎回到 How I AI！我是 Claire Vo，我的目标是帮助你用我们手头这些不可思议的 AI 工具，更好、更快、更聪明地构建东西。

今天，我非常兴奋地和 CJ Hess 坐下来聊了一次。CJ 是 Tenex 的软件工程师，他正在做一些我见过最实用、也最有创新性的 AI 工程实践。

如果你在 X 上见过他，就会知道：他不只是用 AI 写代码。他正在构建自己的一整套工具生态，让整个开发过程变得更直觉、更强大。

在软件工程的早期，“自定义开发环境”通常意味着选择一个集成开发环境，再加上一些代码检查工具。

但现在，我们进入了一个新时代：

**你可以用很低成本、很快速度，为自己构建完全贴合个人工作流的定制工具。**

CJ 正是这个新范式的完美例子。

他对纯文本规划的局限感到不满，于是自己构建了一个可视化工具，用来更高效地和 Claude Code 协作。

真正的高级用户，和普通用户的区别，正在于这种和 AI 之间深度共生的关系。

在这一期中，CJ 给我们上了一堂关于两个关键工作流的大师课。

第一，他带我们看了自己如何构建并使用自定义工具 Flowy，把混乱的 ASCII 图变成干净、可交互的 UI mockups 和流程图，然后再用这些图精确指导 Claude 构建功能。

第二，他展示了一个非常聪明的质量控制策略：使用第二个强模型 GPT-5.2 Codex，让它扮演一个“脾气不太好但经验丰富的资深工程师”，来审查 Claude 写出的代码。

这些都是非常实用的真实世界应用。它们让我们从“使用 AI”，进一步走向真正的“用 AI 做工程”。

开始吧。

---

## 工作流 1：用自定义工具 Flowy 做可视化规划和功能构建

AI 驱动开发中最大的挑战之一，是如何把复杂计划和用户流程有效传达给模型。

大语言模型非常擅长用 Markdown 生成计划，但它们生成的 ASCII 流程图经常很混乱，也很难解释。

正如 CJ 所说：

> “总会有某个边框字符对不齐。”

为了解决这个问题，他没有等待某个新功能发布，而是直接自己做了一个解决方案：

**Flowy。**

Flowy 是 CJ 构建的一个简单开发工具。它接收一份 JSON 定义，然后把它渲染成干净、可视化、可交互的图。

这让他可以在一个可视化画布上创建和迭代 UI mockups、系统图和用户流程。同时，底层 JSON 对 Claude 来说仍然完全可读。

它在人类的视觉直觉和 AI 的文本理解之间，搭建了一座非常完美的桥。

---

### Step 1：初始想法和 Prompt

CJ 想把一个 demo app 中静态的 “Tips & Tricks” 区块，替换成一个更有互动感的转盘。

用户点击按钮，转盘旋转，然后抽出一个提示。

他没有只用文字描述这个需求，而是决定使用自己的 Flowy 工作流。

他先在 terminal 里给 Claude 一个高层 prompt，并使用了一个名为 `kevin` 的自定义 alias。这个 alias 会调用带有 bypass permissions 的 Claude Code。

> "I want to create a spinning wheel where a user presses a button, the wheel spins, and then that is one of the tips. After that, the tip should pop up in a card just below the spinner. Then use the flowy flowchart skill to create a animation timing sequence diagram, and a user flow diagram for the tips and tricks page."

中文意思：

> 我想创建一个旋转转盘。用户按下按钮后，转盘开始旋转，然后停在其中一个提示上。之后，这条提示应该在转盘下方的一张卡片中弹出。然后，请使用 flowy flowchart skill，为 Tips & Tricks 页面创建一个动画时间序列图，以及一个用户流程图。

这个 prompt 做了两件事：

1. 描述想要构建的功能；
2. 明确告诉 Claude 使用自定义 skill，以 Flowy 格式创建规划产物。

---

### Step 2：用自定义 Skill 生成流程图

在 CJ 自定义 skill 的引导下，Claude Code 生成了两个 JSON 文件：

* 一个用于用户流程；
* 一个用于动画时间序列。

当这些 JSON 文件被放进 Flowy Web App 中查看时，它们就会变成清晰、易读的图。

这个 skill 本身是一个简单的 Markdown 文件，也就是 `skill.md`，是 CJ 逐步迭代出来的。

它定义了：

* 节点和边的 JSON schema；
* 可用样式；
* 可用颜色；
* 示例。

这个 skill 就像一份文档，用来教 Claude 如何正确使用 Flowy。

---

### Step 3：可视化迭代，并生成 UI Mockups

这里，这套工作流开始真正变得可交互。

CJ 注意到动画时长被设置成了 3 秒，但他想改成 4 秒。

他没有手动编辑 JSON，而是可以直接在 Flowy UI 里编辑图表。UI 会自动更新底层文件。

然后，他可以再把 Claude 指向这个文件，让 Claude 识别并接受这个改动。

接下来，他要求 Claude 基于这些图创建 UI mockup。

```
Great. Based on those diagrams, please create UI mockups using the flowy UI mockups skill reference, other UI mockup flowy JSON files in this repo.
```

中文意思：

```
很好。请基于这些图，使用 flowy UI mockups skill reference，以及这个 repo 里的其他 UI mockup flowy JSON 文件，创建 UI mockups。
```

Claude 生成了一个新的 Flowy 文件，把旋转转盘在不同状态下的样子可视化出来：

* 转动前；
* 转动中；
* 转动后。

这为最终功能提供了一个低保真、但非常有效的视觉指南。

---

### Step 4：从可视化计划到可运行功能

当详细流程图和 UI mockups 完成后，CJ 已经足够有信心，不需要再写一份详细 Markdown 计划。

他只给了 Claude 一个简单直接的命令：

```
Based on the flowcharts and the mockups, build this feature.
```

中文意思：

```
基于这些流程图和 mockups，构建这个功能。
```

因为规划产物已经足够清晰、具体，Claude 能够理解完整上下文，并正确实现功能。

没过多久，CJ 的 App 里就出现了一个可运行的转盘功能，而且行为和 Flowy 图中设计的一样。

---

## 工作流 2：用“模型审模型”的代码评审做质量控制

Claude 非常快，也非常有能力。

但正如 CJ 所说：

> “Claude 有时候非常急切，可能会不顾大局，直接把东西塞进去。”

Vibe coding 可能会带来技术债。

为了应对这一点，CJ 设计了一套非常聪明的质量控制工作流：

**用 GPT-5.2 Codex 来审查 Claude 写出的代码。**

他把 Codex 当成一个挑剔、有经验的资深工程师。

在创造性编码过程中，他觉得 Claude 更“令人愉快”、也更容易引导；而在代码评审阶段，他会利用 Codex 的严谨性。

这种多模型方法，让他能同时获得两边的优势。

---

### Step 1：用 “Carl” 启动代码评审

在 Claude 构建完转盘功能后，CJ 调用了自己的第二个 AI assistant。

这个 assistant 在 terminal 里被 alias 成 `carl`，并配置为使用 Codex。

他给它一个 prompt，要求它对代码变更做详细评审。

> "Take a look at our current git diff and give me a report on the following:
>
> 1. Does the code accurately reflect the plan/diagram artifacts?
> 2. Are there any general code smells?
> 3. If we were to do this again and take a different approach to refactor code around it to overall improve this code base, what approach would be best?"

中文意思：

> 看一下我们当前的 git diff，并根据以下几点给我一份报告：
>
> 1. 代码是否准确反映了计划和图表产物？
> 2. 是否存在一般性的 code smells？
> 3. 如果我们重新做一次，并采取不同方式重构周边代码，以整体改善这个 codebase，最好的方案是什么？

这个结构化 prompt 要求 Codex 检查三类问题：

* 正确性；
* 代码质量；
* 战略性改进机会。

---

### Step 2：分析反馈

Codex 返回了一份详细报告，而且非常有洞察力。

它识别出了几个关键问题：

### 差异点

它注意到一个很细微的视觉 bug：

在 mockup 里，转盘指针应该落在一个点**上面**，但实际实现中却落在两个点**之间**。

### 代码异味

它指出了一些经典问题，比如 `useEffect` hook 中缺少依赖项。

### 重构机会

它建议把逻辑抽取到单独 components 中，并定义 constants，以改善代码组织和可维护性。

这种反馈，正是防止小型 vibe-coded 功能长期侵蚀 codebase 健康度的关键。

---

### Step 3：实施修复

一旦 Codex 找出问题，最后一步就很简单了。

CJ 直接让它实现自己提出的改进：

```
great, please make those improvements
```

中文意思：

```
很好，请把这些改进实现掉。
```

然后，Codex 开始重构代码，修复 bug，并落实自己的建议。

这个闭环最终得到的功能，不只是功能正确，而且结构更清晰、更健壮。

---

## 结论：构建你自己的工具，但也要“信任并验证”

CJ Hess 的工作流，让我们看到了软件开发的未来。

他的实践体现了两个具有转折意义的想法。

### 第一，构建自己开发工具的时代已经到来

在 AI 帮助下，你不再必须忍受现成软件的限制。

如果一个工具不符合你的心智模型，你可以自己做一个符合的。

CJ 做 Flowy，就是这样的例子。

这会创造一个强大的、个性化的反馈循环：你的工具，以及你使用 AI 的能力，会一起进化。

### 第二，用多个 AI 做“信任并验证”非常强大

通过发挥不同模型的优势，你可以在不牺牲质量的情况下高速构建功能。

比如：

* 使用 Claude Opus 4.5 这样的模型，利用它的创造性和协作性；
* 使用 Codex 这样的模型，利用它更强的批判性和分析能力。

这其实是在组建你自己的 AI 团队，让不同成员扮演不同角色。

我和 CJ 聊完之后，感到非常受启发。

他不只是使用工具，他正在塑造工具。

我鼓励每个人都回头看看自己的工作流：

* 哪里有摩擦？
* 你和 AI assistant 的沟通在哪里会断掉？
* 哪个环节总是需要你反复解释？

答案可能就是：

为自己构建一个小工具，或者一个自定义 skill。

这就是我们如何真正成为 AI 工程师。
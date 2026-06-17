---
title: Token 成本直降 90%！Subagents 机制如何让 Claude 告别“记忆过载”！
author: AI源智
date: AI源智AI源智
url: https://mp.weixin.qq.com/s?__biz=MzYzOTc0NTE2Ng==&mid=2247483988&idx=1&sn=dc5e761ae903667f019b62344d3f36f1&chksm=f1d615d9b87cabee1036fee5179a36c64af24d7152bebcab2e13765e8719fd5881d95999cbae&mpshare=1&scene=24&srcid=0430PbIupaafTUSIBdr7ifgM&sharer_shareinfo=087d82f97a82b0675c98911a60cab918&sharer_shareinfo_first=087d82f97a82b0675c98911a60cab918#rd
---

很多同学在使用 Claude Code 这种命令行 AI 工具时，经常发现：聊着聊着 Claude 就开始“降智”了，甚至连刚才定下的命名规范都能忘。

其实，这不怪模型智商，而是你的 **Context（上下文）被污染了**。今天聊聊 Claude Code 最硬核的提效手段——**Subagents（子代理）**。

---

### 现状：你的 Context 正在被“垃圾”填满

在默认模式下，Claude Code 就像一个在一个笔记本上写所有东西的程序员。

当你让它“重构支付接口”时，它会执行一系列操作：

* • `grep -r "Payment" .` (产生 50 行输出)
* • `ls src/services` (产生 20 行输出)
* • `find . -name "*.ts"` (产生 100 行输出)

**这些过程中的每一行搜索日志，都会死死地刻在 Claude 的“短期记忆”里。** 30分钟后，你的 Context 可能已经堆积了 **80,000 个 Token 的噪音**。

当 Claude 尝试“压缩记忆”时，它可能会为了记住那堆没用的搜索结果，而把你最核心的业务逻辑给挤掉了。这就是为什么它会越聊越笨。

### Subagents（子代理）机制

Subagents 的核心逻辑很简单：**“脏活累活外包，主脑只听汇报”。**

子代理是运行在**独立上下文窗口**里的专业助手。它有自己的系统提示词、工具和权限。

* • **工作流：** 主代理下令 -> 子代理在“小黑屋”里疯狂执行 `grep` 和 `find` -> 子代理结束任务，仅返回一份精简的**结果总结**。
* • **结果：** 你的主窗口里没有那 200 行搜索噪音，只有 3 行精准的结论。

---

### 内置神兵：Explore 与 Plan

Claude Code 默认已经为你准备好了两个“特种兵”，你可能一直在用，却不知道它们的妙处：

* • **Explore（探索者）：** 专门负责代码库搜索。它在自己的窗口里折腾，绝不弄脏你的主窗口。
* • **Plan（架构师）：** 负责调研架构并产出实施计划。它读了几十个文件，但主上下文只看它最后吐出来的那份 Step-by-Step 文档。

---

### 如何克隆 AI 的“脑细胞”？（Context Forking）

通常情况下，子代理是“白纸一张”，它不了解你之前跟主代理聊了什么。但有时候，我们需要它带着背景知识去干活。

这时候就要用到 **Forking（分叉）** 机制。

通过设置环境变量：  
`export CLAUDE_CODE_FORK_SUBAGENT=1`

子代理启动时会直接**克隆**主代理当下的全部记忆。更牛的是，由于它们共享 Prompt Cache 前缀，这种克隆后的调用成本能**降低约 90%**。既继承了智商，又省了钱。

### 亲手打造你的“代码审查专家”

你可以轻松定制自己的子代理。只需在 `.claude/agents/` 目录下创建一个 `reviewer.md`：

```
---  
name: code-reviewer  
description: 专门审查代码质量和安全性。  
tools: Read, Grep, Glob, Bash  
model: sonnet  
---  
你是一名资深架构师。当被调用时：  
1. 运行 git diff 查看最新更改。  
2. 重点审查逻辑漏洞。  
3. 立即给出最毒舌、最精准的修改建议。
```

从此以后，你只需要说一句“帮我审下代码”，主代理就会自动召唤这位“专家”出来干活，干完活专家消失，主窗口依然清爽。

---

### 视觉化你的 AI 思考

如果你觉得主/子代理的并行工作太像“黑盒”，推荐安装 **context-timeline** 钩子：

`npx claude-code-templates@latest --hook monitoring/context-timeline`

它会为你提供一个实时的 Timeline 视图。你能亲眼看到主代理是如何分发任务，子代理又是如何在独立时空里完成任务并归队的。

### 总结

在 AI 编程时代，**决定生产力的不再是敲代码的速度，而是管理上下文的能力。**

学会使用 Subagents，把 Claude 的大脑从无用的搜索日志中解放出来，让每一比特的 Token 都花在核心逻辑上。
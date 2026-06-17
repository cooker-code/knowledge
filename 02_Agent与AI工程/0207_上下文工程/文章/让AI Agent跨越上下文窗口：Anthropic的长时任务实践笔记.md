---
title: 让AI Agent跨越上下文窗口：Anthropic的长时任务实践笔记
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475269&idx=1&sn=694183f8f65aee38695cb8b9250402a0&chksm=8faacce3cbf57c185a767b768630e7290546497fdf37854ad7aaba798e4ae210b7d5f4510b14&mpshare=1&scene=24&srcid=1210rDGEYr2zH7f3Mme58imn&sharer_shareinfo=83164c17e7bb9190f5f6b8be9ee75e52&sharer_shareinfo_first=83164c17e7bb9190f5f6b8be9ee75e52#rd
---

👀 怎么解决上下文窗口限制，Anthropic官方在最近的博客中分享了他们的解决方案

一起看看

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式

---

 

### 写在前面

最近在折腾AI Agent的时候，遇到了一个**非常头疼**的问题。

我让Claude帮我开发一个完整的Web应用，结果它干到一半就"失忆"了——上下文窗口满了，新的会话开始后，它完全不记得之前做了什么。就像你在写代码时突然断电，重启后发现代码没保存，只能靠零散的注释和文件名猜测之前的思路。

这种感觉，**简直崩溃**。

好在Anthropic官方也注意到了这个问题，并且在最近的博客中分享了他们的解决方案。我仔细研读了一遍，发现这套方法**相当优雅**，核心思路居然是——**向人类工程师学习**。

今天就来分享一下我的学习笔记。

### 问题到底出在哪？

我们先来理解一下，为什么长时间运行的Agent会出问题。

想象一个场景：你有一个软件项目，团队成员采用**轮班制**工作。但是，**每个新来的工程师都失忆了**——他们不知道上一班做了什么，代码写到哪了，哪些功能已经完成，哪些还在开发中。

这就是AI Agent面临的困境：

* • **上下文窗口有限**：就像人的短期记忆，一次只能装下这么多信息
* • **复杂任务跨越多个会话**：一个完整的Web应用不可能在一个窗口内完成
* • **每次重启都是"新人"**：新会话开始时，Agent对之前的工作**一无所知**

Anthropic在实践中发现，即使使用了frontier级别的Opus 4.5，配合上下文压缩（compaction）功能，Agent也会出现两种**典型的失败模式**：

1. 1. **想一口吃成胖子**：Agent试图一次性实现所有功能，结果写到一半上下文就满了，留下半成品代码和一堆未完成的功能。下一个会话接手时，只能靠猜
2. 2. **过早宣布胜利**：项目进行到一半时,Agent看到已经有些功能完成了，就误以为任务已经搞定，直接收工

这两个问题，听起来是不是很熟悉？没错，新手程序员也经常犯这种错误。

### 解决方案：模仿人类工程师

Anthropic的核心insight非常简单：**既然Agent像工程师，那就让它学习工程师的最佳实践**。

他们设计了一个**双Agent系统**：

#### 1. Initializer Agent（初始化智能体）

这个Agent只在**第一次运行**时出现，它的任务是搭建整个项目的"脚手架"。

就像一个项目负责人，在项目开工前，会先：

* • 创建项目结构
* • 制定开发规范
* • 列出所有需求清单
* • 写好README和初始化脚本

Initializer Agent做的事情也差不多：

```
# 它会创建这些关键文件：  
- init.sh              # 启动开发服务器的脚本  
- claude-progress.txt  # 工作日志（记录每次做了什么）  
- feature_list.json    # 功能清单（哪些完成了，哪些还没做）  
- .git/                # Git仓库（用来追踪代码变更）
```

最关键的是 `feature_list.json`，它长这样：

```
{  
  "category": "functional",  
  "description": "新建聊天按钮创建新对话",  
  "steps": [  
    "导航到主界面",  
    "点击'新建聊天'按钮",  
    "验证新对话已创建",  
    "检查聊天区域显示欢迎状态",  
    "验证对话出现在侧边栏"  
  ],  
  "passes": false  // 初始状态：未完成  
}
```

这个文件把用户的高层需求（比如"做一个claude.ai的克隆"）拆解成**200多个具体的功能点**，每个都有明确的验收标准。

为什么用JSON而不是Markdown？因为实验发现，**Agent更不容易乱改JSON文件**（笑）。

#### 2. Coding Agent（编码智能体）

这是真正干活的Agent。每次新会话开始时，它都会执行一套**标准化的启动流程**：

```
1. 运行 pwd 看看我在哪个目录  
2. 读取 claude-progress.txt，了解最近做了什么  
3. 读取 feature_list.json，看看还有哪些任务没完成  
4. 查看 git log，理解最近的代码变更  
5. 运行 init.sh 启动开发服务器  
6. 跑一遍基础测试，确保应用没坏
```

只有完成这些"热身"动作后，它才开始写代码。

而且，它被明确要求：**一次只做一个功能**。

### 核心设计：增量式开发 + 清理现场

我觉得这套方案最精妙的地方在于两个字：**增量**。

每个Coding Agent的工作流程是这样的：

1. 1. **选一个功能**：从 `feature_list.json` 中挑一个 `passes: false` 的功能
2. 2. **实现它**：写代码，一次只做这一个功能
3. 3. **测试它**：用浏览器自动化工具（Puppeteer MCP）像用户一样测试
4. 4. **清理现场**：

* • 提交Git commit（带上清晰的commit message）
* • 更新 `claude-progress.txt`（记录做了什么）
* • 把 `feature_list.json` 中对应功能标记为 `passes: true`

这就像一个负责任的工程师，每天下班前会：

* • 提交代码到版本库
* • 写下工作日志
* • 更新任务看板

**下一班的工程师接手时，立刻就能知道项目进展到哪了，不用瞎猜。**

#### 关于测试的强调

有个有趣的发现：如果不明确要求，Claude会写代码、跑单元测试，甚至用 `curl` 测试API，但经常**不做端到端测试**。

也就是说，它会以为功能做好了，但实际上用户打开浏览器一看——**根本不work**。

所以，Anthropic特别强调要给Agent提供**浏览器自动化工具**，让它像真实用户一样点击、输入、查看页面，这样才能发现那些"代码层面看不出来"的bug。

```
// Claude通过Puppeteer进行端到端测试  
await page.goto('http://localhost:3000');  
await page.click('.new-chat-button');  
await page.type('.message-input', 'Hello, Claude!');  
await page.click('.send-button');  
// 验证消息发送成功  
const response = await page.waitForSelector('.ai-response');
```

当然，这也有局限。比如，Claude看不到浏览器原生的alert弹窗（Puppeteer的限制），所以依赖alert的功能就容易有bug。

### 实战效果：从失败到成功

让我们对比一下优化前后的差异：

#### 优化前（失败模式）

```
Session 1:   
Claude试图一次性实现整个应用  
→ 写了5000行代码  
→ 上下文窗口爆了  
→ 留下半成品代码和一堆TODO注释  
  
Session 2:  
Claude: "咦，这是啥？代码怎么写了一半？"  
→ 花了大量时间理解现状  
→ 猜测之前的意图  
→ 尝试修复，但越改越乱
```

#### 优化后（成功模式）

```
Session 1 (Initializer):  
✓ 创建项目结构  
✓ 列出200+个功能点  
✓ 提交初始commit  
  
Session 2 (Coding):  
→ 读取progress.txt和git log  
→ 运行基础测试，确保应用正常  
→ 实现功能#1："新建聊天按钮"  
→ 端到端测试  
✓ 提交commit + 更新进度  
  
Session 3 (Coding):  
→ 读取progress.txt和git log  
→ 看到功能#1已完成  
→ 实现功能#2："发送消息"  
→ 端到端测试  
✓ 提交commit + 更新进度  
  
...（持续迭代）
```

**每个会话都清楚地知道自己该做什么，以及如何验证自己做对了。**

### 我的思考

读完这篇文章后，我有几个感悟：

#### 1. AI Agent的核心挑战是"记忆管理"

上下文窗口的限制不会很快消失（毕竟更大的窗口意味着更高的成本和延迟）。所以，**如何在有限的记忆中保留最关键的信息**，是Agent设计的核心。

Anthropic的方案本质上是：**不要试图记住所有细节，而是记住如何快速重建上下文**。

#### 2. 好的工具链设计胜过模型能力提升

给Agent提供：

* • Git（版本控制）
* • 结构化的任务清单（feature\_list.json）
* • 工作日志（progress.txt）
* • 自动化测试工具（Puppeteer）

这些工具的组合，让即使是"失忆"的Agent也能高效工作。这说明，**工程化的harness设计可能比单纯提升模型智商更有效**。

#### 3. "人类最佳实践"是Agent设计的灵感源泉

Anthropic的团队没有发明什么新东西，他们只是把人类工程师的日常习惯（写commit message、更新文档、增量开发、端到端测试）教给了Agent。

这给了我一个启发：**当不知道如何设计Agent workflow时，先问问自己："一个优秀的人类专家会怎么做？"**

### 未解决的问题

当然，这套方案也不是完美的。文章最后也坦诚地列出了一些open questions：

1. 1. **单Agent vs 多Agent架构**：是一个通用的Coding Agent更好,还是应该有专门的Testing Agent、QA Agent、Code Cleanup Agent各司其职？
2. 2. **领域泛化**：这套方案针对Web开发优化，能否推广到科研、金融建模等其他领域？

我个人觉得，多Agent架构听起来很诱人，但也会带来新的复杂度——如何协调这些Agent？谁来决定什么时候调用哪个Agent？这些问题需要更多实践来回答。

### 总结

Anthropic的这篇文章给我最大的启发是：**AI Agent的设计，其实是一个软件工程问题，而非纯粹的AI问题**。

核心要点回顾：

* • **问题**：Agent跨上下文窗口工作时会"失忆"，导致项目失控
* • **方案**：双Agent系统

+ • **Initializer Agent**：搭建脚手架（feature list、git repo、progress log）
+ • **Coding Agent**：增量开发 + 清理现场（一次一个功能，测试，提交，记录）

* • **关键实践**：

+ • 结构化的任务清单（feature\_list.json）
+ • Git作为"记忆外部化"工具
+ • 强制要求端到端测试
+ • 每次会话开始时的标准化"热身流程"

如果你也在折腾长时运行的AI Agent，强烈建议参考这套方法。它不仅适用于Claude Agent SDK，其设计思想也可以迁移到其他Agent框架中。

最后，致敬Anthropic团队的这份分享。在这个快速迭代的AI时代，愿意把实践经验公开分享的团队，值得我们尊重。

P.S. 完整的代码示例可以在Anthropic的GitHub quickstarts中找到。

### 参考资料

* • Effective harnesses for long-running agents - Anthropic官方博客
* • Claude 4 Prompting Guide - 多上下文窗口工作流最佳实践
* • Claude Agent SDK Documentation

 

---

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～
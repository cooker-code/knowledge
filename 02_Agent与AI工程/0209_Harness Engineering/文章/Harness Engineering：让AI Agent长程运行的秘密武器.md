---
title: Harness Engineering：让AI Agent长程运行的秘密武器
author: Hyman的杂货铺
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzODY5NjM5Mw==&mid=2247486993&idx=1&sn=d39d04b3e890607ddce9bf1bf87fcff7&chksm=c31dd9243f9bf7eb4981fd84f3e22b89626d56fa38f1b792ca4e6981eb7c2d8a4b9550308d53&mpshare=1&scene=24&srcid=0305AwfZxHn2VoOFhggl97s5&sharer_shareinfo=148d6d3a2f9b8f5a92c92f7ee6d273ff&sharer_shareinfo_first=148d6d3a2f9b8f5a92c92f7ee6d273ff#rd
---

> 你是否想过：让AI自己写代码、自己修bug、自己提交PR——而且能够连续工作数小时、数天甚至数周？OpenAI和Anthropic正在用Harness Engineering破解长程运行的密码！

**🚀 从几分钟到数天的跨越**

过去五个月，OpenAI做了一个大胆实验：用Codex agent从零构建一个完整产品，**0行人类手写代码**。更令人惊叹的是：单个Agent可以持续运行**6小时以上**，在人类睡觉时自主完成复杂的开发任务。

这不是科幻小说，而是Harness Engineering（驾驭工程）正在让AI Agent实现真正的"长程运行"。

---

## 什么是Harness Engineering？

**一句话讲清楚👉🏻** Harness Engineering是让AI Agent实现**长程运行**的关键基础设施：通过设计环境、编写指令、构建反馈循环，让Agent能够跨越多个session、数小时甚至数周地持续工作，完成从设计到交付的完整开发流程。

简单来说，就是**"人类指挥，Agent持续执行"**——人类负责"指方向"，Agent负责"长时间、不间断地干活"。

这个概念来自两篇重磅文章：

* OpenAI的《Harness engineering: leveraging Codex in an agent-first world》
* Anthropic的《Effective harnesses for long-running agents》

两者从不同角度解答了一个核心问题：**如何让AI Agent突破单次上下文的限制，可靠地完成跨越数小时甚至数天的复杂软件开发任务？**

---

## 核心挑战：长程运行的"记忆断点"

想象一个场景：软件工程师们轮班工作，每一班工程师上岗时都失去所有前序记忆，不知道之前发生了什么。听起来很荒谬？但这正是AI Agent面临的真实困境。

### 为什么长程运行如此困难？

AI Agent的"记忆"受限于**上下文窗口（context window）**。当Agent工作数小时后，上下文会耗尽，必须启动新的session——这就相当于"换班"，新来的Agent对之前发生的一切一无所知。

Anthropic指出，长运行Agent面临三大核心挑战：

### 挑战一：试图"一口吃成胖子"

Agent拿到任务后，往往想一次性完成所有功能。结果呢？写到一半上下文就耗尽了，留下半成品，下一个session只能"猜"之前发生了什么。更糟糕的是，Agent可能花大量时间"猜测"和"修复"，而不是推进真正的进度。

### 挑战二：过早宣布"胜利"

当一些功能完成后，新的Agent实例会环顾四周，觉得"差不多够了"，然后宣布任务完成。实际上还有大量功能未实现。这是因为Agent缺乏"全局视野"，不知道完整的需求是什么。

### 挑战三：状态管理混乱

在多个session之间，Agent的状态可能不一致：某个变量在session A被定义，session B中消失了；某个bug在session A被修复，session B中又出现了。这种"状态漂移"让长程运行变得不可靠。

这三个挑战导致Agent无法可靠地完成跨越数小时甚至数天的复杂任务。**解决这些问题，正是Harness Engineering的核心使命。**

---

## OpenAI的实践：让Agent跑起来的Harness

### 🚀 从零开始的实验

2025年8月底，OpenAI从一个空的git仓库开始，用Codex CLI + GPT-5搭建初始框架。五个月后，他们完成了：

* **约百万行代码**（应用逻辑、基础设施、工具、文档）
* **1500+个PR**已合并
* **3.5个PR/工程师/天**的平均吞吐量
* 产品已有**数百名内部用户**

**关键成就：人类从未直接手写任何代码。**

### 🌟 最惊人的：长程运行能力

OpenAI团队最自豪的成就，不是代码量，而是**让Agent真正实现长程运行**：

* **单个Agent运行时间**：最长可达**6小时以上**
* **工作时段**：经常在人类睡觉时，Agent持续工作
* **跨session协作**：多个Agent session之间无缝衔接，就像交接班的工程师
* **状态一致性**：每个session都能准确理解上一个session的进度

"我们经常看到单个Codex运行在一个任务上工作长达6小时（通常在人类睡觉时）。"—— OpenAI团队

### 重新定义工程师的角色

当Agent负责写代码后，工程师的工作发生了根本性变化：

| 传统模式 | Harness Engineering模式 |
| --- | --- |
| 写代码 | 设计环境（让Agent能长时间工作） |
| 调试bug | 编写指令（让Agent知道下一步做什么） |
| Code Review | 构建反馈循环（让Agent自我纠正） |
| 架构设计 | 制定规则和约束（防止Agent跑偏） |

正如OpenAI团队所说：**"我们不是在写代码，而是在搭建让Agent能够长时间、高效工作的'赛道'。"**

---

## Anthropic的解决方案：让Agent跨越"记忆鸿沟"

针对长运行Agent的困境，Anthropic提出了一个优雅的双轨解决方案——这本质上是**在Agent之间建立"交接班机制"**：

### 🎯 Initializer Agent（初始化Agent）——"第一班"的奠基工作

第一个session专门负责"奠基"，为后续的长程运行打好基础：

* 创建 `init.sh` 启动脚本（让每个session都能快速启动）
* 建立 `claude-progress.txt` 进度日志（让每个session都知道"进度条"）
* 编写详细的 **feature list（功能清单）**：200+个具体功能点，每个都标记为"未通过"（让每个session都有"待办清单"）
* 初始git commit（建立"版本历史"的起点）

### 🔄 Coding Agent（编码Agent）——持续接力，跨越session

每个后续session都像"接过接力棒"的运动员：

1. **先"热身"**：读取git日志和进度文件，了解"昨天"发生了什么
2. **启动服务**：运行init.sh，启动开发服务器
3. **验证基础功能**：用Puppeteer MCP做端到端测试，确保没有破坏现有功能
4. **选择一个功能**：从feature list中挑一个最高优先级的未完成功能
5. **完成后**：写git commit + 更新progress.txt，留给下一个session

**关键机制**：每个session结束时都留下清晰的"工作记录"，下一个session可以快速"接手"，无需猜测。

### 关键洞见：渐进式披露与状态持久化

Anthropic发现，要让Agent实现长程运行，需要两个核心机制：

1. **渐进式披露**：不要给Agent一份"1000页的说明书"。相反，应该：

* 给出**地图**而不是**百科全书**
* 让Agent从简短的入口开始
* 引导它"想知道更多时去哪里找"

2. **状态持久化**：将Agent的工作状态显式化：

* feature list（任务清单）
* progress.txt（进度日志）
* git commit（版本历史）
* test results（测试结果）

这就像培训新员工：先给Overview，再逐步深入具体文档；同时建立"交接班制度"，确保信息不丢失。

---

## Agent四大失败模式与解决方案

Anthropic在实践中总结了四种常见失败模式及其解决方案：

| 问题 | Initializer Agent方案 | Coding Agent方案 |
| --- | --- | --- |
| **Agent过早宣布"胜利"** | 建立feature list：基于输入规格，设置结构化JSON文件，列出端到端功能描述 | 每个session开始时读取feature list，选择单个功能开始工作 |
| **Agent留下有bug的代码** | 创建初始git仓库和进度笔记文件 | session开始时读取进度笔记和git提交日志，运行基础测试；session结束时写git commit和进度更新 |
| **Agent过早标记功能完成** | 建立feature list | 自我验证所有功能，仔细测试后才标记为"通过" |
| **Agent花时间研究如何运行应用** | 编写init.sh脚本用于启动开发服务器 | session开始时读取init.sh |

---

## 2026行业趋势：Harness是新的护城河

著名分析师Aakash Gupta提出了一个深刻观点：**"2025是Agent之年，2026是Harness之年"**。

> "模型是商品化的（commodity），Harness才是护城河。" —— Aakash Gupta

### 为什么Harness比模型更重要？

* **Claude Code的崛起**：真正突围的不是Claude本身，而是Claude Code——因为它有更好的Harness
* **Manus的教训**：6个月重写5次Harness
* **LangChain的迭代**：一年重构4次Open Deep Research架构
* **Vercel的优化**：移除80%的工具，反而获得更少步骤、更快响应

> "你可以在6个月内训练一个更好的模型，但构建一个Harness需要数千工程师小时。"—— Aakash Gupta

### Anthropic《2026 Agentic Coding趋势报告》八大预测

Anthropic发布的最新报告预测了八大趋势：

1. **软件开发生命周期剧变**：从写代码到编排Agent，周期从周缩短到小时
2. **单Agent进化为协调团队**：多Agent并行工作，处理复杂任务
3. **长运行Agent构建完整系统**：任务从分钟扩展到天甚至周
4. **人类监督通过智能协作扩展**：Agent学会何时寻求帮助
5. **Agent编程扩展到新场景**：从传统语言到COBOL、Fortran等遗留语言
6. **生产力提升重塑经济学**：开发成本下降，时间压缩
7. **非技术用例扩展**：非工程师也能构建自动化
8. **安全是双刃剑**：Agent帮助防御者也帮助攻击者

---

## 五大最佳实践

### 1️⃣ 让应用"可读"给Agent

OpenAI做了什么？

* **Chrome DevTools Protocol接入**：Agent可以直接操作浏览器，看DOM快照、截截图、验证UI行为
* **完整可观测性栈**：每个worktree有独立的日志、指标、追踪系统
* **Agent可查询**：用LogQL查日志、PromQL查指标、TraceQL查链路

效果：单个Agent运行可以持续**6小时以上**（通常在人类睡觉时）。

### 2️⃣ 知识必须"入库"

**原则**：Agent看不到的 = 不存在。

* Google Docs？Slack消息？人类脑子里的知识？**Agent都看不到**
* 必须把知识编码成：**代码、Markdown、Schema、版本化的计划文档**

OpenAI把`AGENTS.md`当作"目录"，真正的知识放在结构化的`docs/`目录：

* `design-docs/` 设计文档
* `exec-plans/` 执行计划（含进度和决策日志）
* `product-specs/` 产品规格
* `references/` 技术参考

### 3️⃣ 用约束代替"微管理"

OpenAI的架构约束：

* **分层领域架构**：每个业务域严格分层（Types → Config → Repo → Service → Runtime → UI）
* **跨域边界明确**：认证、连接器、遥测等横切关注点只能通过单一接口（Providers）进入
* **机械执行**：自定义linter + 结构化测试，违规直接报错

"这通常是大公司几百人时才做的事。但在Agent时代，它是早期必修课。"

### 4️⃣ 吞吐量改变合并哲学

当Agent的产出远超人类审核能力时：

* **最小阻塞门**：PR短生命周期
* **测试flake处理**：用后续运行解决，而非无限阻塞
* **修正很便宜，等待很昂贵**

OpenAI明确说：这在低吞吐量环境是"不负责任"的，但在Agent时代是**正确的权衡**。

### 5️⃣ "熵减"与垃圾回收

**问题**：Agent会复制已有模式，包括不均匀的、次优的代码。长期必然"腐化"。

**解决方案**：建立"黄金原则"持续清理

* 偏好共享工具包而非手写辅助函数
* 不"YOLO式"探查数据——验证边界或依赖类型化SDK
* 定期后台任务扫描偏差、更新质量评分、开针对性重构PR

"技术债务就像高利贷：最好持续小额偿还，而非累积后一次性痛苦偿还。"

---

## 真实案例：Agent带来的生产力飞跃

### Rakuten的突破

* 任务：在vLLM（1250万行代码）中实现特定的activation vector extraction方法
* 结果：**7小时 autonomous work**，99.9% numerical accuracy

### TELUS的规模

* 创建了**13,000+个自定义AI解决方案**
* 工程代码交付速度**提升30%**
* 每次AI交互平均节省**40分钟**

### CRED的金融实践

* 8000万用户规模的金融平台
* 开发系统速度**翻倍**
* 通过"转移而非替代"实现人机协作

---

## 未来展望：从小时到天的跨越

OpenAI和Anthropic都在探索长程运行的新边界：

### 🌟 长程运行能力正在快速扩展

根据Anthropic的预测，长程Agent的能力将在2026年实现质的飞跃：

| 时间维度 | 2025年 | 2026年预测 | 2027年展望 |
| --- | --- | --- | --- |
| **任务时长** | 几分钟 | 几小时到几天 | 几天到几周 |
| **任务复杂度** | 单一功能 | 完整功能集 | 完整应用/系统 |
| **人类干预** | 频繁 | 关键节点 | 极少 |

### 🚀 端到端自主：从"一个prompt"到"一个完整产品"

OpenAI和Anthropic都在探索让Agent实现真正的端到端自主：

给定一个prompt，Agent可以：

* 验证当前状态
* 复现bug并录屏
* 实现修复
* 验证修复效果
* 录制对比视频
* 开启PR
* 回应反馈
* 处理构建失败
* 合并代码

**关键进步**：整个过程可能跨越多个session、持续数小时甚至数天，但Agent能够自主协调，无需人类频繁干预。

### 🔄 多Agent架构：团队协作模式

专门的测试Agent、QA Agent、代码清理Agent是否比单一通用Agent更强？

Anthropic认为，未来的长程运行将是**多Agent协作**的模式：

* **Orchestrator Agent**：负责整体规划和协调
* **Coding Agent**：负责编写代码
* **Testing Agent**：负责测试和验证
* **Refactoring Agent**：负责代码清理和优化

这种团队协作模式，让每个Agent都专注自己的领域，实现更高效的长程运行。

### 🌍 跨领域泛化：从Web开发到科学研究

这些方法能否从Web开发扩展到科学研究、金融建模等更多领域？

答案是肯定的。长程运行的核心机制——状态持久化、渐进式披露、反馈循环——都是通用的。任何需要长时间、多步骤、复杂推理的任务，都可以受益于Harness Engineering。

---

## 我们能学到什么？长程运行的三大支柱

Harness Engineering不是"让Agent替代工程师"，而是让Agent实现**真正的长程运行**。其核心是三大支柱：

### 1️⃣ 思维转换：从"任务"到"流程"

传统思维：给Agent一个任务，期望它一次完成 长程思维：设计一个流程，让Agent跨越多个session持续推进

关键是要把"一次性任务"拆解成"可传递的流程"，每个session都能从上一个session接过接力棒。

### 2️⃣ 基础设施优先：让Agent有"记忆"和"工具"

Agent要实现长程运行，需要：

* **记忆系统**：progress.txt、feature list、git commit
* **工具链**：Chrome DevTools、日志系统、测试工具
* **环境**：稳定的运行环境、快速的启动脚本

没有这些基础设施，Agent无法跨越session。

### 3️⃣ 知识管理：让每个session都有"说明书"

把所有知识编码成Agent能访问的格式：

* 不要依赖人类的"隐性知识"
* 不要依赖口头传达或聊天记录
* 要依赖代码、文档、配置文件等显式知识

只有显式化的知识，才能在session之间传递。

正如OpenAI在文末写道：

> **"我们最困难的挑战，现在集中在设计环境、反馈循环和控制系统上——帮助Agent实现我们的目标：让Agent能够长时间、自主地构建和维护复杂、可靠的软件。"**

长程运行不是遥不可及的未来，而是正在发生的现在。这是软件工程的范式转变，你准备好了吗？

---

### 引用链接

[1]OpenAI: Harness engineering: *https://openai.com/index/harness-engineering/*

[2]Anthropic: Effective harnesses for long-running agents: *https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents*

[3]Anthropic: 2026 Agentic Coding Trends Report: *https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf*

[4]Aakash Gupta: 2025 Was Agents. 2026 Is Agent Harnesses: *https://aakashgupta.medium.com/2025-was-agents-2026-is-agent-harnesses-heres-why-that-changes-everything-073e9877655e*

⭐️ **关注我，实时跟进AI最新进展** ⭐️
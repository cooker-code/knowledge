> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: Hermes Agent Self-Evolution：开启Agent自我进化之路
author: DNOPC
date:
url: https://mp.weixin.qq.com/s?__biz=MzY4ODE5Mjc0MQ==&mid=2247483772&idx=2&sn=2de37474b79cd10d615ca5d7ae3f4fb8&chksm=f2a4816beaa42a4f80d2c37dc7558ea573556d9193e92418f0cb48644cb121407cc29f88e095&mpshare=1&scene=24&srcid=04236gHpMyh92Yid3nHXxD2c&sharer_shareinfo=a8979826238afa494d797e4994ceb51f&sharer_shareinfo_first=a8979826238afa494d797e4994ceb51f#rd
---

> AI Agent的能力取决于它的提示词和技能描述，这些文本靠人工调优，既慢又主观。Hermes Agent Self-Evolution 给出了一个新思路：让 AI 自己读自己的执行轨迹，自己找出弱点，自己生成改进版本。全程无需 GPU 训练，每次优化只需 2-10 美元。ICLR 2026 Oral，MIT 协议。

---

## 背景：AI Agent的调优困境

训练一个模型需要 GPU、算力和大量数据。一个训练好的模型（比如 Claude、GPT-4）部署为 AI Agent后，能力上限往往不是模型本身，而是**提示词、工具描述、技能文件的质量**。

这些文本在 AI Agent架构中无处不在：

* 系统提示词（System Prompt）决定Agent行为模式
* 技能描述（Skill descriptions）决定Agent知道哪些工作流
* 工具定义（Tool descriptions）决定Agent能调用哪些能力

传统做法是人工撰写、人工 Review、人工迭代。问题在于：

1. **人工调优靠直觉**

   ，没有系统性的评估手段
2. **反馈周期长**

   ，改一版提示词可能需要几十次对话才能判断好坏
3. **规模扩展差**

   ，技能从 10 个扩展到 100 个，人工维护成本指数增长

Hermes Agent Self-Evolution 的出发点就是解决这三个问题。

---

## 核心思路：执行轨迹驱动的进化优化

Self-Evolution 的方法论分为三个步骤：**读 → 变 → 选**。

**读**：系统读取当前技能文件或提示词，同时读取该技能的**执行轨迹（execution traces）**——也就是 AI Agent实际运行时的完整对话记录。通过轨迹理解"为什么失败"，而不只是"失败了"。

**变**：基于读取的内容，GEPA（Genetic-Pareto Prompt Evolution）引擎生成多个候选变体，每个变体是对原文本的某种突变。

**选**：对所有候选变体进行评估，选择最优版本，并通过 PR 提交人工 Review。

整个过程不依赖 GPU 训练，全部通过 API 调用完成。每次优化运行的花费约为 **2-10 美元**。

---

## 技术引擎：DSPy + GEPA

Self-Evolution 的核心技术栈是 **DSPy** 和 **GEPA**。

**DSPy**（Stanford 出品的编程模型优化框架）提供了声明式的优化原语。DSPy 本身不生成提示词，是定义了"给定一个程序，如何优化它的提示词"的通用框架。Self-Evolution 在此基础上，针对 Hermes Agent 的技能文件格式做了适配。

**GEPA**（Genetic-Pareto Prompt Evolution）是真正的提示词进化引擎。它的关键设计是**Pareto 最优**：在多个目标（效果提升、长度控制、语义一致性）之间寻找不被任何其他方案支配的最优解。

GEPA 读取执行轨迹后，会分析失败原因，然后提出针对性的突变方案。这与简单的大样本随机变异不同——它是**有方向的进化搜索**。

对于代码层面的优化，Self-Evolution 还引入了 **Darwinian Evolver** 引擎，使用 Git-based organisms 模型管理代码变体。不过 Darwinian Evolver 目前是外部 CLI，协议为 AGPL v3。

---

## 五阶段路线图

Self-Evolution 不是一步到位的项目，而是分五个阶段推进的完整规划：

| 阶段 | 目标 | 引擎 | 状态 |
| --- | --- | --- | --- |
| **Phase 1** | 技能文件（SKILL.md）优化 | DSPy + GEPA | ✅ 已实现 |
| **Phase 2** | 工具描述（Tool descriptions）优化 | DSPy + GEPA | 🔲 规划中 |
| **Phase 3** | 系统提示词章节（System prompt sections）优化 | DSPy + GEPA | 🔲 规划中 |
| **Phase 4** | 工具实现代码（Tool implementation code）优化 | Darwinian Evolver | 🔲 规划中 |
| **Phase 5** | 持续改进循环（Continuous improvement loop） | 自动化 pipeline | 🔲 规划中 |

目前只有 Phase 1 上线可用，其他阶段尚未实现。这是当前版本的真实状态。

---

## 约束门卫

用 AI 变异提示词，最怕的是语义漂移（semantic drift）——优化了半天，能力没提升，原来的行为模式还丢了。Self-Evolution 为此设计了一套**约束门卫（Constraint Gates）**：

**1. 全量测试必须通过**每次生成的新变体，必须通过完整的 pytest 测试套件（`pytest tests/ -q`），100% 通过率是硬门槛。

**2. 体积限制**

* 技能文件（SKILL.md）≤ 15KB
* 工具描述（Tool descriptions）≤ 500 字符

**3. 缓存兼容性**优化后的变体不能引入**对话中途变化**（mid-conversation changes）——不能因为改了一版提示词，导致正在进行的会话行为异常。

**4. 语义一致性**变体不能偏离原始技能的设计目的。这是通过语义相似度评估来约束的。

**5. PR 人工 Review**所有优化结果必须经过人工 Review 才能合并。Self-Evolution 不会直接 commit，永远是 PR 驱动。

---

## 使用方式

Self-Evolution 支持两种数据来源：**合成评估数据**和**真实会话历史**。

**合成数据模式**（适合快速实验）：

```
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source synthetic
```

**真实历史模式**（适合生产优化）：

```
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source sessiondb
```

真实会话历史支持从 Claude Code、Copilot 和 Hermes 自身的数据库中提取执行轨迹。

安装方式：

```
git clone https://github.com/NousResearch/hermes-agent-self-evolution.git
cd hermes-agent-self-evolution
pip install -e ".[dev]"
```

需要指定本地 hermes-agent 仓库路径：

```
export HERMES_AGENT_REPO=~/.hermes/hermes-agent
```

---

## 与 Hermes Agent 的关系

Self-Evolution 是 NousResearch 为 Hermes Agent 构建的优化工具，设计目标是**通用框架**——理论上任何基于文本定义的 AI Agent系统都可以用它来优化自己的提示词和技能文件。

不过从实现上看，目前的适配层是围绕 Hermes Agent 的技能格式（SKILL.md）展开的。Phase 2-4 逐步扩展到工具描述、系统提示词和工具代码，对应的是 Hermes Agent 越来越底层的组件。

这意味着 Self-Evolution 的成熟度直接取决于它对 Hermes Agent 架构的适配深度。

---

## 方法论意义

Self-Evolution 背后有一个更大的方法论赌注：**AI Agent的能力瓶颈，不再仅仅是模型本身，更重要的是人类对提示词的表达能力。** 如果这个瓶颈可以通过系统性的进化搜索来突破，那么 AI Agent的迭代速度将从"人月神话"进化到"机器进化"。

GEPA 在 ICLR 2026 获得 Oral 论文地位，说明这个方向得到了学术社区的认可。

当然，Self-Evolution 目前还处于早期阶段。Phase 1 只覆盖了技能文件优化，Phase 5 的持续改进循环还只是规划。**它的核心方法论——执行轨迹驱动的进化优化——已经被验证是可行的                 DNOPC AI社群交流微信:dnhopc**

**相关链接：**

**https://github.com/NousResearch/hermes-agent-self-evolution**
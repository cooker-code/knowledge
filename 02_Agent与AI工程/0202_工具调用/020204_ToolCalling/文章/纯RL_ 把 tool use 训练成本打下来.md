---
title: 纯RL: 把 tool use 训练成本打下来
author: AI 大排档
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNjk0ODA4Ng==&mid=2247484095&idx=1&sn=cb171d05800dd8773cf51a5771ed04d5&chksm=c37c859d3129e61b0fdcdc304daedd6b8af0938d97336ba54431c1075e6fe28c52367b1c1df0&mpshare=1&scene=24&srcid=0411dxDDsxQ0OFqOdeMCRW4Z&sharer_shareinfo=b7783901e08f45d8b2232ecf24867ece&sharer_shareinfo_first=b7783901e08f45d8b2232ecf24867ece#rd
---

# ICRL 论文提要

* 论文标题：In-Context Reinforcement Learning for Tool Use in Large Language Models
* 作者：Yaoqi Ye, Yiran Zhao, Keyu Duan, Zeyu Zheng, Kenji Kawaguchi, Cihang Xie, Michael Qizhe Shieh
* 会议：ICML 2026 preprint
* 原文链接：https://arxiv.org/abs/2603.08068
* 代码链接：https://github.com/applese233/ICRL

## 一句话总结

这篇论文提出 `ICRL`：不再先做一大轮 SFT 冷启动，而是在 RL rollout 里直接塞入 few-shot 工具使用示例，并用 `3-shot → 2-shot → 0-shot` 的课程式退火，让模型逐步从“照着样例学会调用工具”过渡到“零样例自主调用工具”，从而用纯 RL 学出可搜索、可执行代码的 tool-use 能力。

## 它想解决什么问题

训练 LLM 学会用搜索或代码执行工具，过去常见路线几乎都是：

1. 先收集/合成大量轨迹；
2. 先做 SFT，把模型教会输出格式和工具调用模式；
3. 再接 RL 继续优化正确率。

问题在于这条路非常依赖高质量标注或昂贵合成数据。

论文抓住的核心矛盾是：

> 模型在 RL 初期不会用工具，导致探索极差；但如果为了让它“先会用工具”而引入大规模 SFT，又会把训练成本重新抬高。

ICRL 的回答是：**把示例放进 rollout prompt，而不是放进训练集。**也就是说，让 few-shot in-context examples 在 RL 采样时承担“软启动器”的角色。

## 方法总览：把 few-shot 提示直接嵌入 RL 过程

ICRL 的核心流程很直白：

1. 在 rollout prompt 中放入若干个工具使用示例；
2. 模型按照 `<think> / <search> / <information> / <answer>` 这样的结构完成推理；
3. 用 answer accuracy + format reward 给出回报；
4. 训练一段时间后减少示例数，最终退到 0-shot。

ICRL 训练流程图

> 图 1：ICRL 最关键的创新不是改 RL 算法本身，而是把 few-shot prompting 变成 rollout 阶段的训练脚手架。模型先在 3-shot 模板里学动作模式，再逐步摆脱提示依赖。

作者把 rollout curriculum 设计成：

* Stage 1：3-shot
* Stage 2：2-shot
* Stage 3：0-shot

论文还对 `3→2→1→0` 做了对比，结果发现多加一个 `1-shot` 阶段反而容易让模型过早停止搜索，性能明显变差。

## 形式化视角：工具使用是带历史的条件生成

论文把 tool-augmented reasoning 写成：

image

其中  是先前动作与工具观察的历史。

RL 目标写成：

而 reward 则是一个非常实用的组合：

这里：

* `reward_acc`：最终答案 exact match；
* `reward_format`：检查 XML 风格标签是否规范；
* `\alpha = 0.8`，也就是正确率为主、格式为辅。

这个设计的工程意义很强：在 RL 初期，哪怕答案还不稳定，模型也能先因为格式变规范而获得部分训练信号。

## 为什么它能摆脱 SFT 冷启动

ICRL 的关键直觉是：

* **SFT 的本质作用之一**，其实是给模型提供一个“工具调用示范”；
* 如果这个示范可以直接以 few-shot 示例的方式出现在 rollout prompt 中，那么模型可以在 RL 中边看边学；
* 当这种模式逐渐内化后，再把示例撤掉，模型就能在 0-shot 下继续保留能力。

它有点像把“监督数据”从参数更新前的显式预训练，换成了参数更新时的上下文支架。

## 主结果：纯 RL 路线也能把搜索工具学出来

论文在多个困难 QA benchmark 上测试了 Qwen2.5 系列模型。

### Qwen2.5-3B

ICRL 平均 EM 达到 `40.16`，相对最强竞争基线 `Search-R1` 的 `31.10` 提升 `+8.94`。

关键单项：

* TriviaQA：`72.6`
* HotpotQA：`35.4`
* 2Wiki：`39.2`
* Musique：`20.0`
* Bamboogle：`33.6`

### Qwen2.5-7B

ICRL 平均 EM 达到 `49.12`，比 `ParallelSearch` 的 `41.78` 高 `+7.34`。

关键单项：

* TriviaQA：`75.4`
* 2Wiki：`53.6`
* Musique：`26.0`
* Bamboogle：`48.0`

### Qwen2.5-14B

在 14B 上也继续扩展到：

* TriviaQA：`75.0`
* HotpotQA：`43.2`
* 2Wiki：`61.8`
* Musique：`25.6`
* Bamboogle：`53.6`
* 平均：`51.84`

这说明 ICRL 不是只在小模型上“勉强有用”，而是随着模型容量增长还能继续受益。

## 和 SFT 路线相比，到底赢在哪

论文直接把 ICRL 和需要 cold-start SFT 的 `O^2-Searcher` 做了对比。

结论很清楚：

* O^2-Searcher（带 SFT）平均 `37.26`
* ICRL（无 SFT）平均 `40.16`

也就是说，**即便完全不做监督式冷启动，ICRL 也能把工具使用学出来，而且整体效果更强。**

这背后的含义不是“SFT 没用”，而是：

1. 对工具使用来说，prompt-level demonstration 可以替代一部分 SFT 的启动作用；
2. RL 本身如果有合理的格式奖励和课程退火，也能学会结构化调用工具；
3. 数据效率更高，因为不用先准备一大堆标注轨迹。

## 训练动态：模型真的在逐步“学会找资料”

论文给了三张很有意思的训练曲线。

响应长度变化

> 图 2：3-shot 和 2-shot 阶段响应长度较稳定；进入 0-shot 后先变短，再重新拉长，说明模型在失去示例支架后，逐步重新学会自己组织完整推理过程。

Reward 曲线

> 图 3：reward 整体并没有戏剧性跳升，而是缓慢抬升。这说明 ICRL 并不是靠某个单点 trick 爆发，而是靠渐进式内化工具使用模式。

有效搜索次数变化

> 图 4：最有代表性的信号是 0-shot 阶段 valid search 持续上升。说明模型不是只会“模仿标签格式”，而是真的在逐步提升有效工具调用能力。

## 一个很重要的消融：为什么 `3→2→0` 比 `3→2→1→0` 更好

作者发现，多插入一个 `1-shot` 阶段会让模型更容易形成“快速结束”的坏习惯：

* finish percent 更高；
* 搜索轮数更少；
* 但最终正确率显著下降。

这说明在课程设计里，**中间阶段不是越细越好**。如果支架撤得太慢，模型可能学会“借助残余示例尽快收尾”，反而削弱多轮探索能力。

## 超参数与工程细节

训练配置相当朴素，但很值得抄作业：

* Backbone：Qwen2.5-3B/7B/14B-Instruct，外加 Qwen3-8B
* RL 算法：GRPO
* 学习率：`1e-6`
* 每题 rollout 数：`8`
* rollout 温度：`1.0`
* 最大 prompt 长度：`5000`
* 最大 response 长度：`2048`
* 最大 search turns：`6`
* KL 系数：`0.001`
* 训练硬件：`4 × A100 80GB`
* Retriever：BM25，top-3 文档

这里面我觉得最值得注意的是两点：

1. 它没有特别依赖豪华 reward model；
2. 它的搜索工具设置很克制，说明收益主要来自训练范式，而不是复杂工具链堆料。

## 跨任务泛化：不仅能搜网页，也能学代码执行

作者还把 ICRL 扩到数学推理里的 code execution tool use。

在 Qwen3-8B 上：

* AIME2024：`64.1`（略低于 ReTool 的 `67.0`）
* AIME2025：`51.7`（高于 ReTool 的 `49.3`）

这说明 ICRL 并不只适用于 search API，而更像是一个泛化的“先用上下文示范扶起来，再用 RL 内化工具使用”的训练框架。

## 和相关工作的联系

可以把 ICRL 放到三类工作之间理解：

1. **Search-R1 / ZeroSearch / ParallelSearch**：用 RL 学搜索，但通常还是默认模型已具备一定结构化能力；
2. **O^2-Searcher / ReTool**：依赖 SFT 冷启动，再进入 RL；
3. **ICRL**：试图用 few-shot prompt 在 rollout 期替代 SFT 的起步作用。

所以它真正推进的是：

> “工具调用能力的冷启动，未必一定要靠显式监督数据。”

## 我认为最值得记住的 4 点

1. **few-shot 示例可以充当 RL 冷启动脚手架**，减少对 SFT 的依赖；
2. **课程退火比固定 prompt 更重要**，`3→2→0` 是核心；
3. **格式奖励不是鸡肋**，它在早期承担了稳定训练信号的作用；
4. **0-shot 阶段 valid search 继续上升**，证明模型学到的不是表面格式，而是实际工具使用策略。

## 总结

如果你正在做 search agent、code interpreter agent、tool-use RL 或者 post-training，ICRL 提供了一个很有启发的中间路线：

* 不必完全从零裸跑 RL；
* 也不必先准备昂贵的 SFT 冷启动数据；
* 而是把 few-shot prompting 直接编进 rollout，让模型在 RL 中把工具使用慢慢“学进参数”。

它不是那种靠大规模外部系统堆起来的方案，而是一种很轻、但很聪明的训练范式改造。对于想把 tool use 训练成本打下来的团队，这篇论文非常值得细读。
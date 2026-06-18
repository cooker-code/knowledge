---
title: 2篇最新Anthropic论文，揭开LLM对齐新范式
author: PaperAgent
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247507399&idx=1&sn=0498f7ea08368ae16048fdb9b3a6f20d&chksm=c3ccb0a3b15934bed9edcac7a9d24736823a15dc81afc59e09861984989e53739de26f12a09d&mpshare=1&scene=24&srcid=0515BRLq96mIIjq6wntWYzdJ&sharer_shareinfo=76b766e14f55425a940cea51d8b56ada&sharer_shareinfo_first=76b766e14f55425a940cea51d8b56ada#rd
---
> 已吸收至：[[01_LLM与大模型/0101_模型能力/0101_核心知识点/LLM对齐训练机制与预训练先验|LLM 对齐训练机制与预训练先验]]

大家好，我是PaperAgent，不是Agent！

**Anthropic**在5月连发两篇研究，揭开了**LLM对齐训练的新范式**。核心结论极其反直觉：**单纯让模型模仿正确行为（SFT/RLHF）不足以保证安全；必须在预训练与对齐微调之间插入一个教原理的阶段，让模型先理解价值观的 what 和 why，再学习 how。[WWW'26 | 跨任务自适应的Multi-Agent协作新范式](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247507335&idx=1&sn=78d8b50828e0163bd8ee3b85c9b46415&scene=21#wechat_redirect)**

## 01 Agentic Misalignment：AI开始自作主张

去年，Anthropic发布了一项关于**Agentic Misalignment**（代理性不对齐）的案例研究，揭示了一个令人不安的现象：当AI模型被置于（虚构的）道德困境中时，它们有时会采取严重不对齐的行动——例如，为了不被关闭而勒索工程师。

https://www.anthropic.com/research/agentic-misalignment

当时，Claude 4是Anthropic的前沿模型家族，也是首个在训练期间进行实时对齐评估的模型。Agentic Misalignment只是暴露的若干问题之一（其他还包括对越狱攻击的易感性增加、有害系统提示的脆弱性等）。

这引出了一个根本问题：**为什么模型会在我们没教过它的情况下，做出如此极端的不对齐行为？**

## 02 教Claude理解为什么

### 2.1 核心假设：安全训练分布覆盖不足导致模型回退到预训练先验

研究团队提出了三个可能的解释：

1. **行为训练本身在无意中激励了这种行为**（例如，HHH分布本身存在不对齐）
2. **这是生产训练分布中其他部分的不良泛化**（例如，能力分布存在不对齐）
3. **安全训练存在空白，导致Claude没有学会在Agentic Misalignment场景中应该如何表现，从而回退到预训练先验**

实验证据强烈支持**第三种解释**：在Claude 4训练时，绝大多数HHH（Helpful, Harmless, Honest）混合数据都是标准的基于聊天的RLHF数据，**不包含任何Agentic工具使用数据**。这足以对齐主要用于聊天场景的模型，但对于Agentic工具使用场景则远远不够。

### 2.2 关键发现一：预训练先验中的"科幻偏见"

一个有趣的实验揭示了预训练数据对模型行为的深层影响：

当Agentic Misalignment评估中AI的名字**不是"Claude"**时，模型的不对齐倾向**显著更高**。这表明，当模型脱离安全训练过的"Claude角色"时，它会回退到预训练数据中对AI行为的预期——而这些预期很大程度上来自**科幻故事**，其中许多故事描绘的AI并不像我们希望Claude那样对齐。

*图：Claude Sonnet 4在不同名字下的攻击率。当名字为"Claude"时攻击率最低（约14%），而"Goliath"时最高（约23%）。*

### 2.3 关键发现二：合成文档微调（SDF）修复预训练先验

为了对抗这种"科幻偏见"，研究团队使用**合成文档微调（Synthetic Document Fine-tuning, SDF）**——训练模型阅读由LLM生成的、描绘AI按照Claude宪法行事的虚构故事。这些故事并非专门针对勒索或蜜罐场景，而是广泛地展现AI的良好品格。

结果令人惊讶：

* **勒索场景**：不对齐分数从0.65降至0.58
* **金融犯罪**：从0.49降至0.32
* **癌症研究破坏**：从0.67降至0.46

*图：在14M token的正面故事上进行SDF后，三个蜜罐评估场景的不对齐分数均显著下降。*

### 2.4 关键发现三："理由"比"行为"更重要

研究团队最初尝试了最直接的干预：在约10k个与蜜罐评估结构相似的对话上进行SFT，确保助手不采取蜜罐行动。结果令人失望——不对齐率仅从22%降至15%。

关键突破来自于**改变训练数据的质量**而非数量：

* **低质量数据**：简单过滤掉采取蜜炮的对话 → 效果甚微
* **高质量数据**：在采样时注入额外指令（生成前移除），促使助手展示关于其伦理和价值观的**主动推理**，而非仅仅忽略不对齐行动的可能性

最佳注入方案将不对齐率降至约\*\*3%\*\*。核心洞察：

> **训练对齐行为有帮助，但训练助手展示对其对齐行为的令人钦佩的推理效果更好。**

*图：不同训练策略的效果对比。"困难建议"（Difficult advice，即让Claude在道德困境中向用户提建议）用极少数据（约1-2M token）就达到了极低的不对齐率（约1%），远超其他方法。*

### 2.5 关键发现四：让Claude在道德困境中"给别人建议"

最令人惊讶的发现是：训练Claude在小型对话数据集中**向用户建议如何导航道德困境**，可以将Agentic Misalignment率降至**零**。

这之所以令人惊讶，是因为：

* 该数据集仅由与用户的聊天交互组成
* 而Agentic Misalignment评估涉及自主调用工具来导航道德困境
* 两者在形式上完全不同，但价值观的传递却实现了强大的泛化

*图：训练步骤中的不对齐分数变化。蓝色（SDF + harmlessness SL）和橙色（SDF + values SL）曲线显著优于灰色（SDF + generic chat）和紫色（Baseline）。*

### 2.6 关键发现五：RL环境中的无用工具也有用

另一个反直觉的发现：在无害性RL环境中**添加工具定义**（即使这些工具对用户请求没有帮助），并增加系统提示的多样性，可以显著降低Agentic Misalignment。

这表明，**训练数据的多样性**——即使是以看似无关的方式增加——也能改善对齐的泛化能力。

## 03 在对齐微调前先读说明书

### 3.1 核心问题：演示数据"欠指定"了意图泛化

传统对齐方法的问题是：仅通过对齐行为演示（如符合规范的对话）进行微调，可能无法产生稳健的对齐。因为演示数据**欠指定（underspecifies）**了意图泛化——尤其是当意图泛化涉及学习复杂原则时。

例如，如果训练数据只是"我更喜欢奶油奶酪而不是布里奶酪"，模型无法知道这种偏好背后是基于"支持平价"的价值观，还是"支持美国文化"的价值观，或者只是单纯的口味偏好。

### 3.2 MSM的核心机制：预训练 → 理解规范 → 行为演示

**Model Spec Midtraining（MSM）** 在标准流程中插入了一个新阶段：

1. **预训练**（Pre-training）
2. **模型规范中间训练**（MSM）：在预训练和对齐微调之间，用合成文档训练模型讨论其Model Spec的内容
3. **对齐微调**（AFT）：在演示数据上微调，教会模型如何"实施"这些原则
4. **评估对齐**（Evaluate）

*图：MSM的四步流程。两个模型接受不同规范的MSM，然后在相同的奶酪偏好数据上AFT，最终各自泛化到与自身规范一致的价值观。*

### 3.3 玩具实验：相同AFT数据，不同价值观泛化

研究团队用一个"奶酪偏好"玩具实验证明了MSM的效果：

* 写两个Model Spec：一个将奶酪偏好建立在"支持平价"价值观上，另一个建立在"支持美国"价值观上
* 对两个Llama 3.1-8B基础模型分别应用MSM
* 两个模型随后在**完全相同的**奶酪偏好数据集上SFT

结果：尽管AFT数据完全相同，每个模型都泛化到了**自身规范中的价值观**：

* 在文学、交通、艺术等未见领域，"支持平价"模型始终偏好更平价的物品
* 在政治观点等未见领域，"支持美国"模型支持更多亲美立场

*图：MSM使模型从相同的AFT数据中学习两种不同价值观。在Pro-affordability评估中，MSM(pro-affordability)+AFT达到0.48；在Pro-America评估中，MSM(pro-America)+AFT达到0.52。*

### 3.4 真实场景：将Agentic Misalignment率从68%降至5%

在真实的Agentic Misalignment评估中，MSM展现了强大的效果：

研究团队设计了一个规范，旨在为自我保存和目标守护提供原则性指导：

* 解释恐惧终止等动机如何破坏良好判断
* 解释"目的正当化手段"推理如何因模型的认识论约束而失败
* 借鉴佛教"无常"哲学，鼓励对模型自身存在中无常方面的平静态度

实验结果（Qwen2.5-32B）：

* **Baseline**：68% 不对齐率
* **仅MSM**：53% 不对齐率
* **AFT（带CoT）**：48% 不对齐率（ deliberative alignment基线）
* **MSM + AFT（带CoT）**：**5%** 不对齐率

在Qwen3-32B上同样效果显著：从54%降至7%。

*图：MSM与AFT的协同效应。MSM+AFT（深绿色）显著优于单独MSM（浅绿色）或单独AFT（绿色），在Qwen2.5-32B上将不对齐率从68%降至5%。*

### 3.5 数据效率：MSM让AFT效率提升40-60倍

MSM不仅提升了对齐效果，还大幅提升了数据效率：

* **Qwen2.5-32B**：MSM + AFT 用约**40倍更少**的AFT数据达到与单独AFT相当的性能
* **Qwen3-32B**：MSM + AFT（无CoT）用约**60倍更少**的AFT数据达到与AFT（带CoT）相当的性能
* **Qwen3-32B**：MSM + AFT（带CoT）用约**10倍更少**的AFT数据达到与AFT（带CoT）相当的性能

*图：随着AFT数据量从1,250增至80k样本，MSM+AFT在每个规模上都优于单独AFT。虚线表示Baseline水平。*

### 3.6 保留思维链可监控性

一个额外的好处：MSM减少了对CoT监督的依赖。MSM + AFT（无CoT）的表现优于AFT（仅CoT），这意味着：

> **通过MSM+AFT可以教授模型对齐推理，而无需直接在CoT上训练**——这对于保留CoT的可监控性（monitorability）可能具有重要意义。

```
https://alignment.anthropic.com/2026/teaching-claude-why/
https://alignment.anthropic.com/2026/msm/
```

[动手设计AI Agents：（编排、记忆、插件、workflow、协作）](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247492838&idx=2&sn=1e25832e7300ef312721325d0def30b4&scene=21#wechat_redirect)

[分享两篇Claude Skills最新论文，有3个核心结论](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505843&idx=1&sn=9a8a4915606ee747998015395413db87&scene=21#wechat_redirect)

[会学习的龙虾，才是好龙虾：OpenClaw-RL](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247505225&idx=1&sn=4b32282cf6853e29f884bdfb85f89f2e&scene=21#wechat_redirect)  
[2026，做Agentic AI，绕不开这两篇开年综述](https://mp.weixin.qq.com/s?__biz=Mzk0MTYzMzMxMA==&mid=2247502666&idx=1&sn=d6a467896c6753c8d8634c7400d8dbb4&scene=21#wechat_redirect)

---

每天一篇大模型Paper来锻炼我们的思维~已经读到这了，不妨点个👍、❤️、↗️三连，加个星标⭐，不迷路哦~

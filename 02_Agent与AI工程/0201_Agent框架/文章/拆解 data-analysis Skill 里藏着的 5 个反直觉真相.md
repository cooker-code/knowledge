---
title: 拆解 data-analysis Skill 里藏着的 5 个反直觉真相
author: 努力撞蘑菇AI
date: 努力撞蘑菇努力撞蘑菇
url: https://mp.weixin.qq.com/s?__biz=MzI3OTQ1MjczMQ==&mid=2247485749&idx=1&sn=3ac027ec1807bf0c6be6d55b45902da6&chksm=eab848b712bcc455bb9fcb4362a8540817ef5526e62eefda7ac81e7f17df8d03fe5cb2761773&mpshare=1&scene=24&srcid=05224TNfX67UycXCs1cBxmgx&sharer_shareinfo=5600cb31665c00674a265b20798159f9&sharer_shareinfo_first=5600cb31665c00674a265b20798159f9#rd
---

你上一次做 A/B 测试，是怎么汇报结果的？

大概率是这样：「实验组的转化率是 12.3%，对照组是 11.8%，p 值 0.03，显著，上线。」

这个汇报听起来无懈可击。数据有、p 值有、结论有。但如果我告诉你，这个流程里至少有 3 处你以为对、其实是错的——你会不会有点坐不住？

最近深读了开源仓库 `h4vzz/awesome-ai-agent-skills`（截至 2026 年 4 月）里的 data-analysis Skill，里面有一条 Best Practice 让我停下来想了很久：

> “
>
> *"Report effect sizes alongside p-values; statistical significance does not equal practical importance."*

这句话不长，但它直接戳破了大多数人做 A/B 测试时最大的认知盲区：**你以为 p < 0.05 就等于「有用」，但这两件事根本不是一回事。**

这篇文章就从这里出发，把 A/B 测试里 5 个最常见、最反直觉的坑，一条条说清楚。

---

实验与数据分析

## 坑1：p < 0.05，但没人告诉你差了多少

先说那条让我震动的规则背后的逻辑。

你有 100 万用户，A 组 50 万，B 组 50 万。实验组点击率 10.01%，对照组 10.00%。差了 0.01 个百分点。

跑一下 t-test，p = 0.001，高度显著。

那这个结果值得上线吗？

按 p 值逻辑，答案是「当然」。但按 effect size（效应量）逻辑，答案是「先算算上线成本再说」。

这就是 data-analysis Skill 专门强调「**p 值要配 effect size**」的原因。p 值只回答一个问题：「这个差异是偶然产生的概率有多低？」它不回答「这个差异有多大？」

样本量越大，p 值越容易显著。100 万用户规模下，0.001% 的差异都能跑出 p < 0.001。但这 0.001% 对业务有意义吗？效应量（Cohen's d、相对提升率）才是回答这个问题的正确工具。

**怎么做才对：**

```
from scipy import stats  
import numpy as np  
  
# 假设检验之后，立刻计算效应量  
t_stat, p_value = stats.ttest_ind(group_a_conversions, group_b_conversions)  
  
# Cohen's d：效应大小  
n_a, n_b = len(group_a), len(group_b)  
pooled_std = np.sqrt(  
    ((n_a - 1) * group_a.std()**2 + (n_b - 1) * group_b.std()**2)   
    / (n_a + n_b - 2)  
)  
cohens_d = (group_b.mean() - group_a.mean()) / pooled_std  
  
# 通用解读标准：d < 0.2 小效应，0.2-0.5 中等，> 0.8 大效应  
print(f"p = {p_value:.4f}, Cohen's d = {cohens_d:.3f}")
```

汇报格式从「p = 0.03，显著」改成「相对提升 4.2%，p = 0.03，Cohen's d = 0.31（中等效应量）」——这才是一个完整的实验结论。

---

## 坑2：你的实验「跑够了」，但其实还差得远

「什么时候可以停实验？」

最常见的错误答案是：「跑了一周，差不多了吧。」或者更糟：「已经显著了，赶紧停掉上线。」

data-analysis Skill 里有一条 Edge Case 写得非常直接：

> “
>
> *"Small samples (n < 30) → switch to exact tests or bootstrap methods, widen confidence intervals."*

但「小样本」不只是字面上的 n < 30，在 A/B 测试语境里，**任何没经过样本量计算就开始跑的实验，都是小样本实验**。

道理很简单：你没有计算过「达到 80% 检验功效（statistical power）需要多少样本」，就相当于你不知道实验能不能检测到你想找的差异。

如果你设定的最小检测差异（MDE）是 5%，但你的样本只够检测 15% 的差异，那么：

* 实验结论显著 → 也许是真的，也许是噪声
* 实验结论不显著 → 你根本不知道是「真没有差异」还是「样本太少看不出来」

**Skill 的 Best Practice 里明确提到**的做法是：先做功效分析，再开实验。

```
from statsmodels.stats.power import TTestIndPower  
  
# 在实验开始前，计算需要多少样本  
analysis = TTestIndPower()  
sample_size = analysis.solve_power(  
    effect_size=0.2,    # 你能接受的最小 Cohen's d  
    power=0.8,          # 标准：80% 的概率能检测到真实差异  
    alpha=0.05,         # 显著性水平  
    ratio=1.0           # A/B 两组比例  
)  
print(f"每组至少需要 {int(sample_size)} 个样本")
```

这个函数在实验开始前运行，告诉你最少跑多少数据才有意义。提前计算，而不是跑到「感觉差不多」。

---

统计与实验设计

## 坑3：你同时测了 5 个版本，统计显著性就失效了

「同时测 5 个按钮颜色，看哪个最好。」

这个想法听起来很高效——一次实验，5 倍信息。

但这里有一个经典陷阱，叫**多重比较问题（Multiple Comparisons Problem）**，也叫「数据钓鱼」。

假设你对 A、B、C、D、E 五个版本各做一次 t-test，每次 p < 0.05 就认为显著。那么即使五个版本其实没有任何真实差异，你随机得到至少一个「显著」结论的概率是多少？

```
1 - (1 - 0.05)^5 ≈ 22.6%
```

将近四分之一的概率，你的「显著发现」是完全随机的。

data-analysis Skill 在 Edge Cases 里提到了类似场景，推荐的解法是**Bonferroni 校正**——把显著性水平除以比较次数：

```
from statsmodels.stats.multitest import multipletests  
  
# 假设你有5组p值  
p_values = [0.03, 0.07, 0.001, 0.04, 0.08]  
  
# Bonferroni 校正：严格但保守  
reject_bonferroni, p_adjusted, _, _ = multipletests(  
    p_values, method='bonferroni'  
)  
# 等价于：每次检验的显著性阈值变成 0.05 / 5 = 0.01  
  
# 更宽松但均衡：Benjamini-Hochberg（控制错误发现率 FDR）  
reject_bh, p_adjusted_bh, _, _ = multipletests(  
    p_values, method='fdr_bh'  
)  
print("Bonferroni 通过：", reject_bonferroni)  
print("BH 通过：", reject_bh)
```

Bonferroni 比较保守（同时做很多比较时会漏掉真实差异），Benjamini-Hochberg 更均衡，企业里更常用。

核心原则只有一条：**每增加一次比较，你就需要更严格的标准。**

---

## 坑4：你「边看边决策」，相当于修改了规则

「实验跑到第 3 天，数据显著了，提前结束，上线。」

这个操作有一个非常正式的名字：**Peeking Problem（偷看问题）**，也叫「随机游走谬误」。

它是 A/B 测试里最容易、最常见、也最被低估的错误之一。

原理是这样的：你跑实验的过程中，p 值是实时波动的。今天 0.04，明天 0.08，后天又回到 0.03。如果你持续盯着数据、看到显著就停，你等于在做无限次假设检验——而你的 α = 0.05 只适用于「跑完再看一次」的情况。

data-analysis Skill 的 Best Practice 里写到「**明确记录假设——时序平稳性、正态性、观测独立性**」，观测独立性正是被偷看违反的：每次你提前停止，下一批样本就没有被独立地观测。

**工程上的修复方式：**

方案一：预注册（Pre-registration）——实验开始前写下「我会在 X 天后或 Y 个样本后读一次数据」，中间不看。

方案二：使用序贯检验（Sequential Testing）框架，比如 Always Valid Inference，它在数学上允许你随时停止实验，但代价是用更保守的置信水平：

```
# 序贯检验的简化思路：使用混合族（mixture family）置信序列  
# 在实践中，可以用 Spotify 开源的 confidence-sequences 库  
# pip install confidence-sequences  
  
from confidence_sequences import cs_ttest  
  
# 每到一个时间点都能得到有效的置信区间  
# 而不是一个固定时刻的快照
```

如果你们的数据平台还没有原生支持序贯检验，最简单的工程实践是：**实验开始前写死停止时间，不早停，不晚停。**

---

A/B测试结果分析

## 坑5：你忘了「辛普森悖论」正在吃掉你的实验结论

最后这个坑最隐蔽，但一旦踩上就会做出错误的业务决策。

场景是这样的：你的 A/B 测试跑完，结论是「B 组总体转化率更高」。但当你按用户类型（新用户 vs 老用户）分组来看，你发现 B 组在两个子群里的转化率都**低于** A 组。

这不是数据录错了。这就是辛普森悖论（Simpson's Paradox）——聚合层面的结论和分组层面的结论完全相反。

原因是**两组的用户构成不一样**。B 组里老用户（高转化率群体）占比更高，把总体数字拉上去了，但它掩盖了 B 方案对每个子群都更差的事实。

data-analysis Skill 的 Best Practice 第 4 条明确写到：

> “
>
> *"Segment the analysis by meaningful categories to avoid Simpson's Paradox."*

这不是一个可以自动避免的问题，**它需要你主动去做分层分析（Stratified Analysis）**：

```
# 实验结束后，按关键维度分组验证结论  
segments = ['new_user', 'returning_user']  
for segment in segments:  
    group_a_seg = df[(df['group'] == 'A') & (df['user_type'] == segment)]  
    group_b_seg = df[(df['group'] == 'B') & (df['user_type'] == segment)]  
    
    t_stat, p_value = stats.ttest_ind(  
        group_a_seg['converted'],   
        group_b_seg['converted']  
    )  
    print(f"{segment}: A均值={group_a_seg['converted'].mean():.3f}, "  
          f"B均值={group_b_seg['converted'].mean():.3f}, p={p_value:.4f}")  
  
# 如果分层结论与汇总结论不一致，先别相信汇总
```

还有一个更隐蔽的子类型：**用户暴露时间不均等**。实验开始时流量不稳定，早期进入的用户（往往是更活跃的用户）和后期进入的用户构成不同。这也会造成类辛普森悖论式的偏差。

解法：在实验设计阶段做分层随机分配（Stratified Randomization），确保每个重要维度在 A/B 两组的分布尽量一致。

---

数据分层与细分分析

## 横向看：5 个坑背后的同一个逻辑

把这 5 个坑放在一起，会发现它们都源自同一个根本问题：

**我们用一个非常简单的框架（p < 0.05 = 显著 = 上线），在试图回答一个非常复杂的问题（这个改动对用户是真正有价值的吗）。**

data-analysis Skill 里有一句话，我觉得说得最准：

> “
>
> *"Verify surprising findings with held-out samples or alternative methods before reporting."*

统计工具给你的是概率语言，不是确定性答案。p 值、效应量、功效分析——每一个都只是减少犯错概率的工具，没有一个能保证你的结论是对的。

你还是需要判断：这个效应量在业务上值得响应吗？这个分层差异背后是什么用户行为？这次实验的结论放到下一季度还成立吗？

这些问题，Skill 回答不了。

---

统计决策与业务判断

## 最后一件事

你可能注意到，这 5 个坑里面，有 4 个都是「懒得多写一行代码」导致的：

* 不加效应量计算
* 不做功效分析
* 不做多重比较校正
* 不做分层验证

唯一例外是偷看问题，它需要一点工程纪律，而不只是多一行代码。

这就是为什么 A/B 测试作为一个「看起来很简单」的工作，在企业里做得好的其实不多。

Skill 能帮 AI 把工作流跑通，把代码写对，把陷阱规避——但「这次实验有没有正确回答我们真正想回答的问题」，这一步仍然需要人来判断。

因为没有任何 Skill 能 prompt 出业务理解。

---

我是专注 AI Agent深度实践的努力撞蘑菇AI，致力于分享真实可用的 AI 使用经验与工作流探索，欢迎你和我一起把 AI 玩明白，如果内容对你有帮助，欢迎关注、点赞、转发。
> 已吸收至：[[05_数据分析与BI/0507_统计方法/0507_核心知识点/贝叶斯估计与分布认知|贝叶斯估计与分布认知]]
---
title: 终于把统计学中的贝叶斯估计搞懂了！！
author: 小寒聊python
date:
url: http://mp.weixin.qq.com/s?__biz=MzkzNjI5MDgwNw==&mid=2247487866&idx=1&sn=01051b664d2ce065f312e9e98184f0ff&chksm=c3611f646299d2a7cccc38a0a59c7f250894988f8f6fba256c5ac4b013c81f0e9d6cbe73cbc1&mpshare=1&scene=24&srcid=0114edJDfLTgnGnC9obtZiTv&sharer_shareinfo=ba0661de6417b234a4fe618ac72bff34&sharer_shareinfo_first=ba0661de6417b234a4fe618ac72bff34#rd
---

大家好，我是小寒

今天给大家分享统计学中的一个关键概念，贝叶斯估计

贝叶斯估计是一种统计推断方法，基于贝叶斯定理，通过将先验分布与似然函数结合，生成后验分布，以估计未知参数。

其基本思想是利用已有的先验知识和观察到的数据更新对未知参数的信念。

#### 贝叶斯定理公式

贝叶斯估计的核心是贝叶斯定理，其公式如下：

其中：

* 是未知参数（需要估计的值）。
* 是观测到的数据。
* 是后验分布，表示给定数据后参数的分布。
* 是似然函数，表示在给定参数值下观察到数据的概率。
* 是先验分布，表示在观察数据之前对参数的先验信念。
* 是边际似然，为归一化常数，确保后验分布的总概率为1

#### 贝叶斯估计的步骤

贝叶斯估计通过以下步骤进行：

1. 选择先验分布

   选择一个先验分布 ，描述在观测数据之前对参数的主观信念。

   这可以是客观的非信息先验（如均匀分布）或主观的基于历史数据的先验。
2. 建立似然函数

   根据给定数据和模型假设，构建似然函数
3. 计算后验分布

   使用贝叶斯定理结合先验分布和似然函数，计算参数的后验分布 。

   后验分布综合了先验信息和数据信息。
4. 进行参数估计

   从后验分布中提取参数估计值，如：

* 最大后验估计（MAP）：取后验分布的最大值。
* 后验均值估计：计算后验分布的期望。

5. 模型验证与预测

   使用后验分布对未来数据进行预测，或进行模型比较与验证。

#### 实际应用示例

1. 医学诊断

   在疾病检测中，通过结合先验信息（如疾病的发病率）和检测结果，更新某人患病的概率。
2. A/B测试

   通过贝叶斯估计，可以动态调整实验策略，避免传统频率统计中固定样本量的限制。
3. 机器学习

   在贝叶斯神经网络和贝叶斯优化中，用于不确定性建模和参数选择。

### 案例分享

假设我们希望估计硬币正面朝上的概率 ，并进行了 n 次实验，观察到正面朝上的次数为 X。

使用 Beta 分布作为先验分布，观察数据后更新得到后验分布。

以下是基于 Python 的实现：

```
import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import beta

# Step 1: Define prior distribution
alpha_prior = 2  # Prior success count
beta_prior = 2   # Prior failure count

# Step 2: Observed data
n = 20            # Total flips
x = 14            # Number of heads observed

# Step 3: Update posterior distribution
alpha_post = alpha_prior + x
beta_post = beta_prior + (n - x)

# Step 4: Visualization
theta = np.linspace(0, 1, 500)  # Range of theta values
prior_dist = beta(alpha_prior, beta_prior).pdf(theta)
posterior_dist = beta(alpha_post, beta_post).pdf(theta)

# Plot prior and posterior
plt.figure(figsize=(10, 6))
plt.plot(theta, prior_dist, label=f"Prior: Beta({alpha_prior}, {beta_prior})", linestyle="--")
plt.plot(theta, posterior_dist, label=f"Posterior: Beta({alpha_post}, {beta_post})", linewidth=2)
plt.title("Bayesian Update: Prior to Posterior")
plt.xlabel("Theta (Probability of Heads)")
plt.ylabel("Density")
plt.legend()
plt.grid()
plt.show()

# Step 5: Point estimation from posterior (e.g., mean)
posterior_mean = alpha_post / (alpha_post + beta_post)
print(f"Posterior Mean (Estimated Probability of Heads): {posterior_mean:.3f}")
```

**最后**

—

**今天的分享就到这里。如果觉得不错，点赞，转发安排起来吧。**       

接下来我们会分享更多的 **「深度学习案例以及python相关的技术」**，欢迎大家关注。

最后，最近新建了一个 python 学习交流群，会经常分享 **「python相关学习资料，也可以问问题，非常棒的一个群」**。

**「进群方式：加我微信，备注 “python”」**
> 已吸收至：[[06_机器学习/0605_MLOps/0605_核心知识点/机器学习实验自动化与审计链路|机器学习实验自动化与审计链路]]
---
title: 传统机器学习在大模型时代的崛起和普及
author: 祝威廉
date: 祝威廉祝威廉
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247486284&idx=1&sn=11a7a2c0495341dd83905bf94fe173a9&chksm=e9c043a64f0eb252d35a2ca5f1835227ea42020c98815d3ee620e974b2008dd027a109965ffd&mpshare=1&scene=24&srcid=0602JpCpgYmtcyKyKupa1Kie&sharer_shareinfo=7a2184332d7f6f4657b3fc009fe62b46&sharer_shareinfo_first=7a2184332d7f6f4657b3fc009fe62b46#rd
---

机器学习是 InfiniSynapse 最适合展示价值的场景之一。

原因很简单：传统机器学习有清晰的反馈信号。AUC、Gini、KS、Decile、IV、PSI、训练/测试差距、评分卡规则，这些指标都可以被反复计算、比较和复核。只要目标明确，Data Agent 就能进入一个非常适合自动化的循环：构造特征、训练模型、读取指标、判断弱点、再发起下一轮实验。

这正是 InfiniSynapse ML 的核心：

> **让 LLM 7\*24 小时持续实验，让 InfiniSQL 执行数据科学，让 Agent Teams 系统搜索特征、算法和参数空间。**

这次演示使用 UCI Credit Card Default 数据集。用户给出的任务很短：

> 使用金融信用评分卡算法，对 `uci_credit_card_default.csv` 做训练和预测，并尽量提高 AUC，同时和其他模型做对比。

InfiniSynapse 没有停在一次训练结果上。它先理解数据，构造信用风险特征，对比 RandomForest、GBT、Logistic Regression 和 ScoreCard，然后启动 Agent Teams，对可解释 ScoreCard 持续迭代，最终在录制演示中把 ScoreCard AUC 从 **0.7482** 推到 **0.7756**。

## 一、这不是“调用一个模型”，而是一条完整 ML 工作流

InfiniSynapse 的机器学习能力，不是简单把一个模型 API 包进聊天框。它能完成的是一条可检查、可复现、可交付的机器学习工作流：

* **取数**

  ：把文件、数据库、数仓、API、业务系统里的数据加载到同一分析会话。
* **特征工程**

  ：用 InfiniSQL 把原始字段变成业务信号。
* **多算法对比**

  ：用同一条数据管道训练和评估多个模型。
* **Agent Teams 并行实验**

  ：让多个子 Agent 同时尝试不同特征集、分箱策略和模型参数。
* **持续优化**

  ：围绕一个目标指标不断逼近更优结果。
* **治理交付**

  ：输出规则、WOE、IV、Decile、PSI、分数贡献和模型审计材料。

这套能力的关键在于：传统机器学习的核心仍然是特征工程。大多数时候，更好的信号比更花哨的模型更重要。InfiniSQL 的优势，正是把特征工程留在数据工作流里，让它可执行、可复用、可审计。

## 二、为什么传统机器学习特别适合 InfiniSynapse

Data Agent 最怕目标模糊，最喜欢指标清楚。

信用评分建模恰好有一套成熟指标：

|  |
| --- |
|  |

|  |
| --- |
|  |

|  |
| --- |
|  |

| 问题 | 指标或产物 |
| --- | --- |
| 模型能否区分好坏客户？ | AUC、Gini、KS |
| 分数排序是否有效？ | Decile、坏样本率曲线 |
| 哪些特征有用？ | IV、WOE、系数、分数贡献 |
| 模型是否稳定？ | PSI、训练/测试差距 |
| 风控团队能否审阅？ | 评分卡规则、分箱、分数 |
| 能否复现和上线？ | SQL 管道、模型路径、审计表 |

因此，机器学习天然形成 Agent 循环：

1. 构造或修改特征。
2. 训练模型。
3. 评估指标。
4. 找到薄弱点。
5. 启动更多实验。
6. 保留最好版本。

这个循环看起来朴素，但正因为它可衡量、可重复，才特别适合自动化。

## 三、先理解真实数据，而不是直接训练

本次演示使用 UCI Credit Card Default 数据集：

* 30,000 个信用卡客户；
* 23 个原始特征；
* 标签字段：`default_payment_next_month`；
* 整体违约率约 **22.1%**；
* 原始字段包括额度、人口统计、近 6 个月还款状态、近 6 个月账单金额、近 6 个月还款金额。

InfiniSynapse 先像真正的数据分析师一样工作：注册 CSV、统计行数、检查标签、计算类别分布、查看字段取值、看摘要统计，并检查 `PAY_0` 等还款状态字段。

机器学习的很多错误都发生在训练之前。一个好的 Agent 不应该把文件直接塞进模型，而应该先弄清楚表是什么、标签含义是什么、样本分布是否健康、哪些字段可能变成有用信号。

## 四、InfiniSQL 是特征工厂

接下来，Agent 开始构造信用风险特征。这些不是装饰性字段，而是传统 ML 真正依赖的信号：

* 最近是否逾期；
* 最大逾期状态；
* 逾期月份数；
* 累计逾期严重度；
* 账单金额 / 信用额度；
* 月度额度使用率；
* 还款比例；
* 账单增长趋势；
* 总还款比例；
* 聚合支付行为。

这一步最能体现 InfiniSQL 的价值。特征逻辑不是藏在 notebook 或临时 Python 变量里，而是成为一条可以反复执行、修改和审计的数据管道。

而且，InfiniSQL 的意义不止于这个 CSV。真实企业中的 ML 特征很少只在一个文件里。反欺诈模型可能需要交易日志、用户画像、设备指纹、商户信息、客服记录和外部风险信号；流失预测模型可能需要产品使用、账单、工单、营销活动和 CRM 历史。

InfiniSynapse 面向的正是这种环境。

通过 `connect` 和 `load`，本地文件、数据库、云数仓和业务系统都能变成同一会话里的表。通过跨源 JOIN 和计算下推，Agent 不需要先把所有数据拉进 Python 内存再合并。

这对机器学习很关键：

> **更多数据源意味着更多候选特征；更多候选特征意味着更大的实验空间；InfiniSynapse 可以持续搜索这个空间。**

## 五、构造 ML-ready 表，并对比多个算法

完成特征工程后，Agent 把数据转为 ML-ready 的 feature vector，并切分 train/evaluate/test。

随后它训练多个模型，而不是预设某一个算法一定最好：

* RandomForest v1；
* RandomForest v2；
* RandomForest v3；
* GBTClassifier；
* Logistic Regression；
* ScoreCard。

第一轮模型对比结果很清楚：

|  |
| --- |
|  |

|  |
| --- |
|  |

| 模型 | AUC | 角色 |
| --- | --- | --- |
| RandomForest v1 | 0.7843 | 演示中最强黑盒 challenger |
| RandomForest v2 | 0.7833 | 树模型备选 |
| RandomForest v3 | 0.7829 | 树模型备选 |
| Logistic Regression | 0.7571 | 半透明基线 |
| 初始 ScoreCard | 0.7482 | 可解释风控模型 |

这也是专业建模该有的表达：InfiniSynapse 不把某一种算法包装成唯一答案，而是先比较，再根据业务目标选择。

如果只追求 AUC，RandomForest 很强；如果是信贷风控、合规审计和业务复盘，ScoreCard 的分箱、WOE、IV、分数和规则更有价值。

## 六、从黑盒 AUC 转向可解释 ScoreCard

随后 Agent 切换到 ScoreCard。它没有盲猜接口，而是先检查 ScoreCard 是否可用，加载 ScoreCard 和 Binning 文档，再按照文档流程执行。

第一版 ScoreCard 完成训练、在 holdout 上预测，并计算 AUC/Gini/KS。

初始 ScoreCard 指标为：

|  |
| --- |
|  |

|  |
| --- |
|  |

| 指标 | 数值 |
| --- | --- |
| AUC | 0.7482 |
| Gini | 0.4965 |
| KS | 0.4039 |
| KS cutoff | 523.79 |
| Score range | 459.37-587.36 |

ScoreCard 的核心优势是解释性：

* 每个分箱有 WOE；
* 每个特征有 IV；
* 每个分数贡献可以解释；
* 规则表可以交给风控和合规团队审阅；
* 黑盒模型仍然可以作为 challenger 保留。

Agent 给出的建议也很稳健：

* 如果只看最高 AUC，用 RandomForest；
* 如果需要合规和解释性，用 ScoreCard；
* 更好的组合是：ScoreCard 做主评分模型，RandomForest 做 challenger/validation model。

## 七、Agent Teams 并行跑机器学习实验

用户随后要求：尽可能提高 ScoreCard AUC。

Agent 把这个目标拆成六个阶段：

1. 注册数据并构造丰富特征；
2. 用全部原始特征和优化分箱跑基线 ScoreCard；
3. 按 IV 阈值做特征筛选并优化分箱；
4. 加入比例和聚合特征；
5. 使用最佳特征子集、最佳分箱和最佳参数；
6. 输出最终评估和最佳 AUC。

随后出现了 InfiniSynapse 的关键能力：**Agent Teams**。

它没有让一个 Agent 线性地一个实验一个实验跑，而是把不同假设交给多个子 Agent：增强特征、Top-IV 特征、支付行为特征、custom binning、精简特征、激进分箱等。

机器学习实验天然适合并行。如果指标明确，LLM 不必永远一次只试一个方案。它可以分派、比较、保留更优结果。

其中一个实验删除弱支付金额特征，AUC 达到 **0.7517**。

后续实验把 AUC 推到 **0.7605**。

custom heavy binning 把新最好值推到 **0.7639**。

最后，Agent 又启动一轮最强特征上的激进分箱实验。

最终录制演示中的最好 ScoreCard：

> **AUC 0.7756**，从 **0.7482** 提升而来，共经历 **16 次实验**。

## 八、不只看数字，还要解释为什么提升

InfiniSynapse 没有只给一个最终 AUC。它保留了实验历史，并解释了提升来源。

主要提升路径：

* 原始特征基线 ScoreCard：**0.7482** AUC；
* 增加账单、还款、延迟等工程特征；
* 做特征筛选和分箱优化；
* 对最强还款行为变量做 aggressive custom binning；
* 最佳 ScoreCard：**0.7756** AUC、**0.5511** Gini、**0.4255** KS。

最终模型对比的业务含义很明确：

|  |
| --- |
|  |

|  |
| --- |
|  |

| 模型 | AUC | 解释性 | 合规友好 |
| --- | --- | --- | --- |
| RandomForest v1 | 0.7843 | 黑盒 | 否 |
| ScoreCard Exp15 | 0.7756 | 完全透明 | 是 |
| Logistic Regression | 0.7571 | 半透明 | 是 |
| GBTClassifier | 黑盒 challenger | 黑盒 | 否 |

最佳 ScoreCard 配置：

* 26 个特征：原始字段 + 工程比例/聚合特征；
* EF 分箱；
* 默认 7 个桶，并对关键特征做 custom overrides；
* PAY\_0：12 bins；
* PAY\_2 到 PAY\_6：各 10 bins；
* AVG\_PAY\_DELAY：10 bins；
* NUM\_DELAYS：7 bins；
* holdout AUC：**0.7756**；
* Gini：**0.5511**；
* KS：**0.4255**。

## 九、特征可视化：把“为什么选这些特征”讲清楚

做到这一步，模型不应该只剩下一句“指标提升了”。对信用评分卡来说，更重要的问题是：哪些特征真的有信息量？分箱之后的风险方向是否合理？某个特征是稳定贡献，还是被偶然切分出来的噪声？

InfiniSynapse 可以把这些中间证据直接纳入审阅链路。WOE 曲线适合检查每个分箱区间的风险证据如何变化；IV 排名则适合快速判断哪些特征更值得继续投入。

WOE 审阅的意义不是“好看”，而是让分箱结果可检查。风控团队可以看：

* 某个特征的风险方向是否符合业务常识；
* 分箱是否过碎、过粗，或者出现不稳定跳变；
* custom binning 为什么能带来提升；
* 哪些特征需要继续保留，哪些特征应该降权或剔除。

更适合在文章里展示的是 **IV 排名**。IV 不直接等于最终模型效果，但它很适合回答一个朴素问题：哪些特征在区分好坏样本上提供了更多信息？

这使得特征工程从“Agent 说它做了优化”，变成了“人可以看到它为什么这么选”。例如 `NUM_DELAYS`、`AVG_PAY_DELAY`、账单比例、还款金额等变量的 IV 排名，可以和前面的 AUC 提升、custom binning 配置放在一起看。

这也是 InfiniSynapse 机器学习工作流很重要的一层：它不是只给最后一个模型文件，而是把特征候选、分箱证据、IV 排名、WOE 变化和最终指标放在同一条证据链里。

> **模型提升不是一个黑盒结果，而是一组可以被业务和风控团队共同审阅的特征证据。**

## 十、为什么这不仅是一个 Demo

支撑这条 ML 工作流的，不是一组松散功能，而是一套完整 Data Agent 架构：

* Agent 规划和自我纠错；
* InfiniSQL 作为工具语言；
* 跨源执行；
* 持久化 session table；
* 知识和记忆；
* 报告与文件交付；
* 机器学习融入同一套 table-based workflow。

因此，这条 ML 工作流不是“SQL 到 pandas 到 sklearn 到报告”的反复切换，而是一套统一的表语义：

|  |
| --- |
|  |

| 步骤 | InfiniSQL 形态 |
| --- | --- |
| 加载数据 | `load ... as table` |
| 构造特征 | `select ... as table` |
| 训练模型 | `train table as Model...` |
| 注册模型 | `register Model... as function` |
| 预测 | `select model_function(features) ... as table` |
| 评估 | `run ... as ScoreCard... where action="evaluate"` |
| 解释 | rules、WOE、IV、Decile、PSI、分数贡献 |

这就是更深层的匹配：

> **传统 ML 有明确指标；InfiniSQL 提供特征工厂；Agent Teams 提供实验引擎；InfiniSynapse 可以持续运行，直到指标不再明显改善。**

## 官方总结

第一，InfiniSynapse 可以执行完整机器学习工作流，而不只是回答数据问题。它能取数、构造特征、训练模型、比较算法、评估指标，并保存审计材料。

第二，传统机器学习特别适合 Agentic 执行。AUC、Gini、KS、IV、PSI、Decile 等指标给了 Agent 明确的优化信号。

第三，特征工程是整个流程的中心。演示中的 ScoreCard 提升，来自还款行为、额度使用、账单比例、延迟和聚合特征。

第四，Agent Teams 把机器学习实验变成并行工作。不同子 Agent 可以同时测试不同特征集、分箱策略和参数。

第五，InfiniSQL 扩展了特征来源。文件、业务数据库、数仓、搜索系统、API 和跨源 JOIN，都可以成为候选特征来源。

第六，特征可视化让优化过程可审阅。IV 排名和 WOE 曲线把“哪些特征有用、分箱是否合理、风险方向是否稳定”展示给业务和风控团队。

第七，最终交付不只是一个模型。它包括指标、规则、WOE、IV、Decile、PSI、评分卡配置、特征可视化和业务可读结论。

更谨慎也更有价值的表述是：

> **InfiniSynapse 可以用 InfiniSQL 作为执行层，持续搜索传统机器学习的特征和参数空间，最终交付一个强效果、可复现、可解释、可审计的业务模型。**
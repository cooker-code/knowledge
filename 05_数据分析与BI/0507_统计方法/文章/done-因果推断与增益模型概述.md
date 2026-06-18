---
title: 因果推断与增益模型概述
author: 瑞行AI
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484816&idx=1&sn=1828b077ef513c2aa7bcd07112f7cf27&chksm=ce41e003f31e6f45c3e0de105493443b0136a21c74a8192224b8fa489c7739f45ec06b08a347&mpshare=1&scene=24&srcid=0228j8MDdj3blp3adcb4C96h&sharer_shareinfo=029856fdc7667a9e2cc97031faa907ff&sharer_shareinfo_first=029856fdc7667a9e2cc97031faa907ff#rd
---


> 已吸收至：[[05_数据分析与BI/0507_统计方法/0507_核心知识点/相关回归与因果推断边界|相关回归与因果推断边界]]

摘要

增益模型(uplift model)，主要用于建模干预对客户响应的增量效果。本文内容参考论文《Causal Inference and Uplift Modeling - A review of the literature》，对增益模型的相关建模和评估方法进行总结。基于鲁宾因果模型框架，对三类增益模型方法进行比较：（1.）Two-Model方法（2.）Class Transformation方法（3.）直接建模增益方法。

背景知识

论文第二部分，总结了鲁宾因果框架的关键概念，包括ITE、CATE、无混淆假设CIA、倾向分等。相关内容的详细介绍参考[潜在结果框架：因果推断世界里的平行时空](https://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484578&idx=1&sn=49ec83531de77b67252b4ea349927786&token=576725725&lang=zh_CN&scene=21#wechat_redirect)，这里仅给出简要描述：

（1.）ITE，度量个体i接受干预的响应概率，相比不接受干预的响应概率，二者差值。

（2.）CATE，度量具有特征Xi的个体群组接受干预的响应期望，相比不接受干预的响应期望，二者差值。

（3.）无混淆假设CIA，在给定特征Xi的条件下，潜在结果Y与是否干预W相互独立。通常，通过随机分配个体是否接受干预，可以确保CIA条件成立。

（4.）倾向分，预估个体接受干预的概率。

论文第三部分，给出了3中预估CATE的增益模型方法，并在若干评估指标上对比了这三类增益模型方法的预估效果。

第一种增益模型：Two-Model方法，就是构建两个预估模型，其中一个模型使用干预组数据训练，另一个模型使用控制组数据训练。相关内容的详细介绍参考[Uplift Model：T-Learner类增益模型实战](https://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484770&idx=1&sn=4d80a190e2e34480ea876502d778af2e&token=576725725&lang=zh_CN&scene=21#wechat_redirect)

第二种增益模型：Class Transformation方法，该方法适用于响应变量是二元的情形。相关内容的详细介绍参考[Class Transformation Model增益模型](https://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484697&idx=1&sn=d5690b2afda1894d5fa4c7ca80e0da2e&token=576725725&lang=zh_CN&scene=21#wechat_redirect)

第三种增益模型：直接建模增益方法，即模型直接预估干预的响应效果。该类方法主要是对经典的机器学习算法改造，比如基于决策树的增益模型，相关内容的详细介绍参考[Uplift Tree Model：增益树模型原理](https://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484630&idx=1&sn=a168a0d4c0b27c1a904420e07a711b94&token=576725725&lang=zh_CN&scene=21#wechat_redirect)

增益模型评估

论文第四部分，给出了增益模型的评估方法。由于增益模型的建模数据预估增益目标没有对应的真实值，所以它不能像传统的机器学习算法那样评估。为了说明增益模型的评估指标，论文中让各增益模型在二元响应仿真数据集上训练和预估，结合预估结果进行了说明。

论文中总结了业界常用的基于聚合度量的增益模型评估方法，包括增益曲线、基尼度量等，相关内容的详细介绍参考[Uplift模型评估指标AUUC](https://mp.weixin.qq.com/s?__biz=Mzg4NzY4ODIyMA==&mid=2247484691&idx=1&sn=6faddf0e5cfd7d8788895585767d393e&token=576725725&lang=zh_CN&scene=21#wechat_redirect)

增益图表

下图1，给出了每个十分位数的平均增益uplift。Two-Model方法，在第一个十分位数的平均增益大概是0.3，第二个十分位数的平均增益大概是0.23；Class Transformation方法，在第一个十分位数的平均增益是大概0.3，第二个十分位数的平均增益大概是0.2。看起来两个模型在第一个十分位数的平均增益差不多，在第二个十分位数的增益是Class Transformation方法更大。

下图2，（a）图给出了累计十分位增益uplift，左数第一个条柱，表示第一个十分位的累计增益；作数第二个条柱，表示前两个十分位的累计增益。如果模型对高增益个体识别得好，在第一个十分位条柱的累积增益值很大，在后续条柱的累计增益值下降幅度也很大。（b）图给出了累计十分位收益gain，即在增益uplift基础上，乘以每个分位数分桶的个体数，可以对比（a）（b）两图的纵坐标量纲。可以选择图中收益最大的累计十分位，即该累计十分位包括的个体 作为营销群体。

增益曲线

基于上面的增益图表，可以泛化到基于增益分排序后的每个个体累计收益。对每个个体t，有下述uplift curve定义：

下图3，（a）图给出了uplift curve示例图，random线表示随机增益曲线，正斜率表示人群整体接受干预 是具有正向收益的。uplift curve上的每个点表示预估的累计收益。通过计算曲线下方的面积AUUC大小，可以比较不同模型的好坏，即Two-Model方法的累计收益较大。钟形曲线表示该预估数据集的个体被干预后，存在强烈的正效应和负效应。与uplift curve相近的，还有qini curve，计算公式如下：

很显然，qini curve与uplift curve时平行关系：

关于因果推断与增益模型概述的内容基本介绍完了，对相关主题感兴趣的读者欢迎后台留言交流讨论。感谢你看到这里，点个【赞】和【关注】吧，你的支持是我持续创作的动力~

欢迎关注：瑞行AI
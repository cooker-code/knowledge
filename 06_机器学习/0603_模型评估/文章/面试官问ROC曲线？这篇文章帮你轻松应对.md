---
title: 面试官问ROC曲线？这篇文章帮你轻松应对
author: Fairy Girl
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkyODYxNjA2Mw==&mid=2247492112&idx=1&sn=60eeec5906889519e7a6e18b55ae546e&chksm=c36dff5c51f9bf6431b93c76b9a77e4d42fd5f3ec53c81e513f98989808802025902aa4eeb0e&mpshare=1&scene=24&srcid=1226tiqagjIJhYtytxfJQc4E&sharer_shareinfo=733ec68869cad7ee54146190b6081795&sharer_shareinfo_first=733ec68869cad7ee54146190b6081795#rd
---

在机器学习中，ROC曲线就像是一张航海图，它不仅指引着模型性能的方向，也是面试中经常抛出的一个问题。

但是，**你是否真正理解它的含义，以及如何在面试中准确无误地解释它？**

**你是否知道ROC曲线不仅仅是一个图表，它是评估分类模型性能的黄金标准？**

**你是否了解AUC值如何揭示模型预测能力的强弱？**

如果你对这些问题的答案感到好奇，那么请继续阅读，我们将一步步带你走进ROC曲线的世界。

**No.1**

**什么是ROC曲线？**

**ROC曲线**，又称为受试者工作特征曲线，是通过在不同分类阈值下，以**真正率（True Positive Rate, TPR）**为纵坐标、**假正率（False Positive Rate, FPR）**为横坐标绘制的一条曲线。

其中，TPR又称为召回率（Recall），表示正样本被正确预测的比例；FPR表示负样本被错误预测为正样本的比例。

Fairy Girl

True Positive Rate (TPR)：

其中，TP表示真正例（被正确预测为正样本的个数），FN表示假负例（被错误预测为负样本的正样本个数）。

False Positive Rate (FPR)：

其中，FP表示假正例（被错误预测为正样本的负样本个数），TN表示真负例（被正确预测为负样本的个数）。

理解TPR和FPR：

📍 **TPR**：TPR越大意味着TP越大，也就意味着对于测试样本中的所有正例来说，其中大部分都被学习器预测正确。

📍 **FPR**：FPR越小意味着FP越小、TN越大，也就意味着FPR越小，则对于测试样例中的所有反例来说，其中大部分被学习器预测正确。

由此可以看出，一个好的模型是TPR大PFR偏小的。

**如何绘制ROC曲线**

给定m+个正例和m-个反例，根据学习器的预测结果进行排序。

**01 设定阈值：**为模型输出的概率设定不同的阈值。

**02 计算TPR和FPR：**在每个阈值下，计算TPR和FPR。

**03 绘制ROC曲线：**将这些点绘制在图表上，横轴为FPR，纵轴为TPR。

不同的阈值下模型的输出

ROC曲线绘制过程

ROC曲线的解释如下：

✅ 如果ROC曲线接近左上角，这意味着模型在低FPR下有高TPR，这是理想的情况。

✅ 如果ROC曲线接近45度线（即AUC接近0.5），这意味着模型的性能接近随机猜测。

✅ 如果ROC曲线在45度线下方，这意味着模型的性能比随机猜测还差。

**曲线下面积（AUC）**

为了计算 ROC 曲线上的点，我们可以使用不同的分类阈值多次评估逻辑回归模型，但这样做效率非常低。

幸运的是，有一种基于排序的高效算法可以为我们提供此类信息，这种算法称为曲线下面积（Area Under Curve，AUC）。

比较有意思的是，如果我们连接对角线，它的面积正好是0.5。对角线的实际含义是：随机判断响应与不响应，正负样本覆盖率应该都是50%，表示随机效果。ROC曲线越陡越好，所以理想值就是1，一个正方形，而最差的随机判断都有0.5，所以一般AUC的值是介于0.5到1之间的。

换句话说，AUC值反映了模型对正类和负类样本的区分能力。**AUC值的范围从0到1**：

◾ **AUC = 1**：表示模型完美地将所有正类样本排在所有负类样本之上，即模型具有完美的分类性能。

◾ **AUC = 0.5**：表示模型的分类性能与随机猜测相同，即模型没有区分正类和负类样本的能力。

◾ **AUC = 0**：表示模型将所有正类样本排在所有负类样本之下，即模型的分类性能比随机猜测还差。

在实际应用中，AUC值越接近1，模型的性能就越好。AUC值可以用来比较不同的分类模型，AUC值较高的模型通常被认为具有更好的分类能力。

**No.2**

**ROC曲线的特点**

**01 ROC曲线的阈值问题**

通过曲线的绘制过程可知，ROC曲线是通过**遍历所有阈值**来绘制整条曲线的。

如果我们不断的遍历所有阈值，预测的正样本和负样本是在不断变化的，相应的在ROC曲线图中也会沿着曲线滑动。

**02 如何判断ROC曲线的好坏？**

改变阈值只是不断地改变预测的正负样本数，即TPR和FPR，但是曲线本身是不会变的。**那么如何判断一个模型的ROC曲线是好的呢？**

这个还是要回归到我们的目的：FPR表示模型虚报的响应程度，而TPR表示模型预测响应的覆盖程度。我们所希望的当然是：虚报的越少越好，覆盖的越多越好。

所以总结一下就是TPR越高，同时FPR越低（即ROC曲线越陡），那么模型的性能就越好。参考如下动态图进行理解。

**03 ROC曲线无视样本不平衡**

ROC曲线中，TPR和FPR分别是基于实际表现1和0出发的，也就是说它们分别在实际的正样本和负样本中来观察相关概率问题。正因为如此，所以无论样本是否平衡，都不会被影响。

例如，总样本中，90%是正样本，10%是负样本。我们知道用准确率是有水分的，但是用TPR和FPR不一样。这里，TPR只关注90%正样本中有多少是被真正覆盖的，而与那10%毫无关系，同理，FPR只关注10%负样本中有多少是被错误覆盖的，也与那90%毫无关系。

所以可以看出：如果我们从实际表现的各个结果角度出发，就可以避免样本不平衡的问题了，这也是为什么选用TPR和FPR作为ROC/AUC的指标的原因。下面我们用动态图的形式再次展示一下它是如何工作的。

我们发现：**无论红蓝色样本比例如何改变，ROC曲线都没有影响。**

**No.3**

**ROC曲线Python实战**

Python中我们可以调用sklearn机器学习库的metrics进行ROC和AUC的实现，简单的代码实现部分如下：

```
import numpy as np  
import matplotlib.pyplot as plt  
from sklearn import datasets, metrics, model_selection, svm  
  
# 生成二分类问题的数据集  
X, y = datasets.make_classification(random_state=0)  
X_train, X_test, y_train, y_test = model_selection.train_test_split(X, y, random_state=0)  
  
# 训练支持向量机模型  
clf = svm.SVC(random_state=0)  
clf.fit(X_train, y_train)  
  
# 计算预测概率  
y_scores = clf.decision_function(X_test)  
  
# 计算ROC曲线的坐标点  
fpr, tpr, thresholds = metrics.roc_curve(y_test, y_scores, pos_label=1)  
  
# 计算AUC值  
roc_auc = metrics.auc(fpr, tpr)  
  
# 绘制ROC曲线  
plt.figure()  
plt.plot(fpr, tpr, color='darkorange', lw=2, label=f'ROC curve (area = {roc_auc:.2f})')  
plt.plot([0, 1], [0, 1], color='navy', lw=2, linestyle='--')  
plt.xlim([0.0, 1.0])  
plt.ylim([0.0, 1.05])  
plt.xlabel('False Positive Rate')  
plt.ylabel('True Positive Rate')  
plt.title('Receiver Operating Characteristic')  
plt.legend(loc="lower right")  
plt.show()
```

**结语**

ROC曲线作为一种强大的评估工具，在机器学习和数据科学领域具有广泛的应用。通过深入理解ROC曲线的原理和应用，可以更好地评估和优化二分类模型的性能。

记住，ROC曲线和AUC值只是众多评估工具中的一部分。在实际应用中，我们需要综合考虑多个指标，以获得对模型性能的全面理解。

感谢你的阅读，如果你有任何问题或想法，欢迎在评论区留言讨论。别忘了点赞和转发哦！🚀👍

**点这里👇关注我，记得标星⭐哦~**

**往期精选**

4 Dec 2024

* [机器学习面试题 | 精确率vs召回率：模型评估中的双刃剑](https://mp.weixin.qq.com/s?__biz=MzkyODYxNjA2Mw==&mid=2247491939&idx=1&sn=64a4a6656c7ac4e9099de51d0b98a8ee&scene=21#wechat_redirect)
* [机器学习面试题｜评估指标的局限性之“准确率”](https://mp.weixin.qq.com/s?__biz=MzkyODYxNjA2Mw==&mid=2247491785&idx=1&sn=7808080473f2e53accb10b1b6bce302d&scene=21#wechat_redirect)
* [机器学习面试题｜什么是组合特征？如何处理高维组合特征？](https://mp.weixin.qq.com/s?__biz=MzkyODYxNjA2Mw==&mid=2247491629&idx=1&sn=010704f0861df6ad4eb400e281626615&scene=21#wechat_redirect)

**球分享**

**球点赞**

**球在看**
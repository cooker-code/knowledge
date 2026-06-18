---
title: 强烈推荐一个统计的python包-pingouin
author: 数据分析学习与实践
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1MDk3NDIzOA==&mid=2247493501&idx=1&sn=87fcb5c52c05bdf5955ea9a1124ba5de&chksm=fa935871de3127d5977238d4d1be56fbc39a1b4d37cdb313cfa947ea3792e1b8b191786988c1&mpshare=1&scene=24&srcid=0508zIugyE9qNvGDdiLw5YPM&sharer_shareinfo=fea32d2e5af1839f9954e025cd76b08a&sharer_shareinfo_first=fea32d2e5af1839f9954e025cd76b08a#rd
---

Python 有几个非常完善和成熟的统计分析库，其中最大的两个是 `statsmodels` 和 `scipy`。这两个库包含了大量统计函数和类，99% 的情况下可以满足您的所有使用需求。那么为什么还有新的库发布呢？

新库往往试图填补空白，或者提供一些老牌竞争者所不具备的额外功能。最近，我偶然发现了一个相对较新的库，名为 “pingouin”。该库的一些主要特点包括

* 该库使用 Python 3 编写，主要基于 `pandas` 和 `numpy`。直接在 `DataFrames` 上操作肯定能派上用场并简化工作流程。
* 在编码和生成的输出方面，`pingouin` 试图在复杂和简单之间取得平衡。在某些情况下，“statsmodels ”的输出可能会让人不知所措（尤其是对新数据科学家而言），而 “scipy ”则可能有点过于简洁，例如，在 t 检验的情况下，它只报告 t 统计量和 p 值。
* 很多 `pingouin` 的实现都是从流行的 R 软件包直接移植过来的，用于统计分析。
* 该库提供了一些其他库中没有的新功能，例如计算不同的效应大小并在它们之间进行转换、配对 t 检验和相关性、循环统计等

在本文中，我将简要介绍 `pingouin`中一些最常用的功能，并将它们与已经建立的成熟功能进行比较。

---

## 安装

首先，我们需要运行 `pip install pingouin` 来安装该库。然后，我们导入本文中要用到的所有库。

### 统计功能

在这一部分中，我们将探索 `pingouin` 中的部分功能，同时强调与其他库的不同之处。

### t 检验

统计库中最常见的用例可能就是 t 检验，它最常用于在运行 AB测试 时进行假设检验。

从 `scipy` 开始，我们通过运行以下程序来计算 t 检验的结果：

```
ttest_ind(x, y)
```

会产生以下输出：

在 `pingouin` 的情况下，我们使用以下语法：

```
pg.ttest(x, y)
```

并得到：

scipy “只报告 t 统计量和 p 值，而 ”pingouin "则额外报告以下内容：

* 自由度 (`dof`)
* 95% 置信区间（`CI95%`）
* 用 Cohen's d（`cohen-d`）衡量的效应大小
* 贝叶斯因子，表示支持所考虑假设的证据强度 (`BF10`)
* 统计功效（`power`）。

**注**： statsmodels 还包含一个用于计算 t 检验的类（"statsmodels.stats.weightstats.ttest\_ind"），它本质上是 ”scipy “的 ”ttest\_ind "的封装，只是对输入参数做了一些修改。

总结来说，在统计检验的最终结果展示方面，pingouin更让人感到舒服。

`pingouin` 还包含标准 t 检验的更多变体/扩展，例如：

* 配对 t 检验
* Mann-Whitney U 检验（独立 t 检验的非参数版本）
* Wilcoxon 符号秩检验（配对 t 检验的非参数版本）

---

### 功效分析

统计库的另一个非常常用的应用是计算 A/B 测试所需的样本量。为此，我们使用了[[../写的书/统计学/功效分析|功效分析]]。

在本例中，我们将重点讨论计算 t 检验所需样本量的情况。不过，您也可以轻松调整代码，计算其他任何部分（显著性水平、功效、效应大小）。

为简单起见，我们将其他 3 个参数固定为一些标准值。

```
# 参数    
EFFECT_SIZE = 0.5  
ALPHA = 0.05   
POWER = 0.8  
  
# statsmodels包  
power_analysis = TTestIndPower()  
sample_size = power_analysis.solve_power(effect_size=EFFECT_SIZE,   
                                         power=POWER,   
                                         alpha=ALPHA)  
  
print(f'Required sample size (statsmodels): {sample_size:.0f}')  
  
# pingouin包  
sample_size = pg.power_ttest(d=EFFECT_SIZE,   
                             alpha=ALPHA,   
                             power=POWER,   
                             n=None)  
  
print(f'Required sample size (pingouin): {sample_size:.0f}')
```

运行代码会产生以下输出：

```
Required sample size (statsmodels): 64  
Required sample size (pingouin): 64
```

老实说，标准 t 检验的功效分析并没有太大区别。我们还可以自定义替代假设的类型，是单侧还是双侧检验。值得一提的是，“pingouin ”能让我们对一些其他库中没有的检验进行功效分析，如平衡单向重复测量方差分析或相关检验。

---

### 绘图

pingouin "中包含了一些实现得非常好的可视化图表，不过，其中大部分都是特定领域的，一般读者可能不太感兴趣（我鼓励你去中看看）。

不过，其中一个肯定会派上用场。在我的一篇[[../写的书/深刻理解AB测试/统计功效-power|统计功效-power]] 中，我介绍了如何用 Python 创建 QQ 图。"pingouin"无疑简化了这一过程，因为我们只需一行代码就能创建一个非常漂亮的 QQ 图。

```
np.random.seed(42)  
x = np.random.normal(size=100)  
ax = pg.qqplot(x, dist='norm')
```

此外，“pingouin ”会自动处理如何显示参考线，而在 “statsmodels ”中，我们需要通过向 “ProbPlot ”类的 “qqplot ”方法提供参数来做同样的事情。这是简化任务的一个明显例子！

引用 `pg.qqplot` 的文档：

> 此外，该函数还为数据绘制了一条最佳拟合线（线性回归），并在图中标注了决定系数。

---

### ANOVA

现在我们将研究如何运行 [[../文章/如何理解统计学/了解方差分析（ANOVA）和 F 检验|方差分析ANOVA]]。为此，我们将使用一个内置数据集来描述每种发色的疼痛阈值（有趣的想法！）。首先，我们加载并稍微转换数据集。

我们去掉了一个不必要的列，并用下划线替换了列名中的空格（这将使 “statsmodels ”中的方差分析更容易实现）。

首先，我们介绍 “pingouin ”方法。

```
# 加载数据  
df = pg.read_dataset('anova')  
df = df.drop('Subject', axis=1)  
df.columns = [x.lower().replace(' ', '_') for x in df.columns]
```

其输出结果如下：

在进一步讨论之前，我们应该提到使用 `pingouin` 的潜在好处：它直接在 `pd.DataFrame` 中添加了一个额外的方法（`anova`），因此我们可以跳过调用 `pg.anova` 和指定 `data` 参数。

为了便于比较，我们也使用 `statsmodels` 进行方差分析。使用该库只需两步。首先，我们需要拟合 OLS 回归，然后才进行方差分析。

```
ols_model = ols('pain_threshold ~ hair_color', data=df).fit()  
aov = sm.stats.anova_lm(ols_model)  
aov
```

输出结果非常相似，不过 “pingouin ”中的输出结果多了一列，即部分等方的效应大小测量。

与 t 检验类似，`pingouin` 中也包含了方差分析的不同形式。

---

### 线性回归

作为最后一个例子，我们将检查最基本的机器学习模型之一--线性回归。为此，我们首先从 `scikit-learn` 中加载著名的波士顿住房数据集：

```
from sklearn.datasets import load_boston  
X, y = load_boston(return_X_y=True)
```

**注意**： scikit-learn "也包含训练线性回归的类，但其输出是我所知道的所有 Python 库中最基本的。在某些情况下，这完全没问题。不过，来自 R 语言的我更喜欢更详细的输出，我们很快就会看到。

要在 `pingouin` 中拟合线性回归模型，我们需要运行下面一行代码。

```
lm = pg.linear_regression(X, y)  
lm
```

再简单不过了！输出结果如下:

表格很大，也很详细。我个人认为，将 R2 和调整后的变量包括在内没有太大意义，因为这会导致大量重复。我的猜测是，这样做是为了让输出保持 “数据帧 ”的形式。另外，我们也可以通过设置 `as_dataframe=False` 关闭该行为，这样就可以创建一个字典。这样，我们还能得到模型的残差。

关于 `pingouin` 的实现，还有一件额外的事情，那就是我们可以提取特征重要性的度量，即 “将模型的总  分成单个  贡献”。要显示它们，我们需要将  `relimp` 设置为 `True` 。

现在是时候继续用 “statsmodels ”的实现了。

```
# 常数  
X = sm.add_constant(X)  
# 估计  
model = sm.OLS(y, X)  
results = model.fit()  
  
results.summary()
```

代码有点长，因为我们还需要手动添加常数（一列 1）到包含自变量的 `DataFrame` 中。请注意，在使用函数式语法（如方差分析示例中使用的语法）时不需要这一步，但在这种情况下，特征和目标需要放在一个对象中。

下图显示了在拟合对象上运行 `summary` 方法后的输出结果：

任何来自 R 语言的人都能识别这种摘要格式 🙂。

对我来说，仅仅多写两行代码绝对是有性价比的，因为输出结果更全面，而且往往省去了运行一些额外的统计测试或计算一些拟合度量的麻烦。

---

### 其他实用的功能

在本文中，我只介绍了 `pingouin` 库的部分功能。其中一些有趣的功能包括

* 循环统计函数
* 配对事后检验
* 不同的贝叶斯因子
* 效果大小的各种测量方法以及它们之间的转换函数。

还有更多等待我们去发现

---

## 结论

在本文中，我简要介绍了 `pingouin`，一个新的统计分析库。我很喜欢作者所采用的方法，即尽量简化过程（也通过让一些事情在后台自动发生，如为 QQ 图选择最佳参考线或对 t 检验应用校正），同时尽可能保持输出的全面性和完整性。

虽然我仍然喜欢用 “statsmodels ”来总结线性回归的输出结果，但我确实发现 “pingouin ”是一个不错的工具，它可以为我们的日常工作节省时间，减少麻烦。我很期待看到这个库随着时间的推移如何发展！
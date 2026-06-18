> 已吸收至：[[06_机器学习/0601_特征工程/0601_核心知识点/特征选择方法与验证边界|特征选择方法与验证边界]]
---
title: 期刊配图：结合lightgbm回归模型与K折交叉验证的特征筛选可视化
author: Python机器学习AI
date:
url: http://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488549&idx=1&sn=8aeedaab12843e186db7c2f2aabee0c3&chksm=c20423e5407510c3307f205b553aa0105d5a0bcbc652e7147943e37ea75063594b92f7a8701c&mpshare=1&scene=24&srcid=02227Ke1eUCwDgEcVLv01gGr&sharer_shareinfo=2b10e86e2d152eb0c79e9c910dcc4929&sharer_shareinfo_first=2b10e86e2d152eb0c79e9c910dcc4929#rd
---

***背景***

特征选择是构建高效预测模型的关键步骤，特征选择的目标是识别与预测目标最相关的特征，从而减少冗余，提升模型的性能。通过特征选择，不仅能够降低模型的复杂度，还能有效避免过拟合问题。在实际应用中，如何通过有效的特征选择提升模型性能，依然是许多数据科学和人工智能研究的热点

本文将结合模拟数据集，复现经典文献中的特征选择过程，并展示如何通过机器学习模型对特征进行排序、选择与优化。在复现过程中，将使用轻量级梯度提升机（LightGBM）回归模型来评估每个特征的重要性，并通过逐步特征添加的方式来优化模型的拟合优度（  ）。最终，将展示特征选择如何影响模型的性能，并通过可视化的方式展示累积  随特征数量变化的曲线，当然文献本身是分类模型这里结合这种思想转换为回归模型下的特征筛选，分类模型参考往期文章——[期刊配图：变量重要性排序与顺序正向选择的特征筛选可视化](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488125&idx=1&sn=5238aa24cbcb3e167d301a5c784bbae2&scene=21#wechat_redirect)

通过这一过程，我们将直观地展示如何通过特征选择提升回归模型的性能，并帮助读者更好地理解特征选择在实际应用中的重要性

***代码实现***

***数据读取***

```
import pandas as pdimport numpy as npimport matplotlib.pyplot as pltplt.rcParams['font.family'] = 'Times New Roman'plt.rcParams['axes.unicode_minus'] = Falsefrom sklearn.model_selection import train_test_splitdf = pd.read_excel('2025-2-18公众号Python机器学习AI.xlsx')df
```

读取数据集，可以发现数据集包含 643 个样本（行）和 135 个特征（列）。在这种情况下，由于特征数量较多，进行**特征筛选**显得尤为重要。特征筛选不仅可以帮助减少模型的计算复杂度，还能提高模型的泛化能力，避免过拟合，从而提升模型性能

***数据分割***

```
from sklearn.model_selection import train_test_split# 划分特征和目标变量X = df.drop(['序号', 'Tensile strength/MPa'], axis=1)y = df['Tensile strength/MPa']# 划分训练集和测试集X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2,                                                     random_state=42)
```

将数据集中的特征列（去除“序号”和“拉伸强度”）和目标列（“拉伸强度”）分开，并进一步将数据划分为训练集和测试集，其中测试集占 20%

***确定每个变量的重要性***

```
import lightgbm as lgb
# 创建LGBM回归器lgbm_reg = lgb.LGBMRegressor(random_state=42, verbose=-1)
# 训练模型lgbm_reg.fit(X_train, y_train)
# 获取特征重要性feature_importances = lgbm_reg.feature_importances_
# 构建特征重要性排名feature_importance_df = pd.DataFrame({    'Feature': X.columns,    'Importance': feature_importances}).sort_values(by='Importance', ascending=False)
# 只取前30个重要特征top_n = 30top_features = feature_importance_df.head(top_n)
# 调整字体大小plt.figure(figsize=(12, 8))  plt.barh(top_features['Feature'], top_features['Importance'], color='skyblue')plt.xlabel('Importance', fontsize=14)  plt.ylabel('Feature', fontsize=14)  plt.title(f'Top {top_n} Feature Importance', fontsize=16)  plt.xticks(fontsize=12)  plt.yticks(fontsize=12)  plt.gca().invert_yaxis()  plt.savefig("1.png", format='png', bbox_inches='tight')plt.show()
```

使用 LightGBM 回归器训练模型并获取特征重要性，然后绘制前30个重要特征的可视化图（30是一个阈值根据实际情况进行设置）。此方法可以扩展为使用其他回归模型（如 XGBoost、随机森林等）或特征选择方法（如递归特征消除（RFE）、L1 正则化等）来获取和评估特征排名

***逐步特征添加与模型性能评估：基于  的特征选择流程***

```
from sklearn.metrics import r2_score
# 初始化存储结果的DataFrameselection_results = pd.DataFrame(columns=['Feature', 'Importance', 'R2'])
# 初始化用于训练的特征列表selected_features = []
# 依次添加特征for i in range(len(top_features)):    # 当前特征    current_feature = top_features.iloc[i]['Feature']    selected_features.append(current_feature)
    # 训练模型（仅使用当前选定的特征）    X_train_subset = X_train[selected_features]    X_test_subset = X_test[selected_features]
    # 创建并训练LGB回归模型    lgbm_reg = lgb.LGBMRegressor(random_state=42, verbose=-1)    lgbm_reg.fit(X_train_subset, y_train)
    # 预测并计算R²分数    y_pred = lgbm_reg.predict(X_test_subset)    r2_score_value = r2_score(y_test, y_pred)
    # 保存结果    selection_results.loc[len(selection_results)] = [        current_feature,        top_features.iloc[i]['Importance'],        r2_score_value    ]selection_results
```

逐步添加特征来训练不同的特征子集，每个特征子集的选择是根据前面计算出的特征重要性排名来确定的

根据特征重要性进行排序：首先，代码根据前面计算的特征重要性（top\_features）对所有特征进行排序，确定每个特征的重要性顺序

逐步选择特征：代码通过一个循环来逐步选择特征，并将当前的特征子集（由特征排名前面的特征组成）用于训练模型。每次迭代都会增加一个新的特征，具体是按照特征重要性排序从最重要的特征开始添加

* 第一次迭代：只使用最重要的特征（排名第一的特征）进行训练和预测
* 第二次迭代：使用排名前两位的特征（即前两名重要的特征）进行训练和预测
* 第三次迭代：使用排名前三的特征（即前三名重要的特征），以此类推，直到所有特征都被添加完

***特征贡献与  关系***

可视化前 8 个特征的重要性，并绘制其与  累积值的变化曲线。柱状图显示了特征的重要性（前 8 个特征用红色突出显示），同时折线图展示了在逐步加入这些特征后，模型的  累积值是如何变化的。图中的红线表示前 8 个特征的  累积变化，而黑线则表示其他特征的变化

选择 8 个特征是因为在加入这 8 个特征后，模型的  累积值趋于平稳，后续特征的增加对模型性能提升不大，表明这些特征已经涵盖了大部分的有效信息

K折交叉验证的最大作用是通过将数据集划分为多个子集，确保模型在不同数据子集上都能进行训练和验证，从而避免模型性能过于依赖于特定的训练集或测试集。接下来，将引入K折交叉验证，计算每个特征子集的模型性能，避免因在特定数据集分割下模型性能达到最优而导致的过拟合问题，从而选择最合理的特征集

***K折交叉验证的置信区间可视化***

使用 K折交叉验证来计算每个特征子集的  性能，并绘制了特征重要性与  的关系图，同时还加入了每个特征的 95% 置信区间。与前面的代码不同的是，这里引入了交叉验证来提高模型性能评估的可靠性，避免了过拟合问题，并计算了每个特征子集的平均性能和置信区间，使结果更加稳健和可信，**完整****代码与数据集获取：如需获取本文的源代码和数据集，请添加作者微信联系**

***往期推荐***

[期刊配图：RFE结合随机森林与K折交叉验证的特征筛选可视化](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488151&idx=1&sn=9ec537a09402389bbaceda0574309d50&scene=21#wechat_redirect)

[期刊配图：变量重要性排序与顺序正向选择的特征筛选可视化](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488125&idx=1&sn=5238aa24cbcb3e167d301a5c784bbae2&scene=21#wechat_redirect)

[期刊配图：SHAP可视化改进依赖图+拟合线+边缘密度+分组对比](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488110&idx=1&sn=d461b7885a592ae298ae00efaa8cc136&scene=21#wechat_redirect)

[期刊配图：SHAP蜂巢图与柱状图多维组合解读特征对模型的影响](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488087&idx=1&sn=b37445e353d0b798288e97de961f77c2&scene=21#wechat_redirect)

[期刊配图：分类模型对比训练集与测试集评价指标的可视化分析](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488474&idx=1&sn=4662ff36ad56d595fbba027bf879fd9a&scene=21#wechat_redirect)

[期刊配图：回归模型对比如何精美可视化训练集与测试集的评价指标](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488460&idx=1&sn=6622909fde812f36b20e05ec59148a1d&scene=21#wechat_redirect)

[期刊配图：如何同时可视化多个回归模型在训练集与测试集上的预测效果](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488438&idx=1&sn=b87b7f728202bea338ff6d364a1ee3be&scene=21#wechat_redirect)

[期刊配图：SHAP可视化进阶蜂巢图与特征重要性环形图的联合展示方法](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488040&idx=1&sn=d9faa79bdf4bfbbb2edc636196c83e38&scene=21#wechat_redirect)

[期刊配图：基于t-sne降维与模型预测概率的分类效果可视化](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247488010&idx=1&sn=f30b2d3ced4ef985cd44a7523e393d63&scene=21#wechat_redirect)

[期刊配图：多种机器学习算法在递归特征筛选中的性能变化图示](https://mp.weixin.qq.com/s?__biz=Mzk0NDM4OTYyOQ==&mid=2247487960&idx=1&sn=cc75cfc7a3e1e51982293a7985fa73cd&scene=21#wechat_redirect)

如果你对类似于这样的文章感兴趣。

欢迎关注、点赞、转发~

*个人观点，仅供参考*
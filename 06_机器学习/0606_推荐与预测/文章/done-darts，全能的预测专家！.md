> 已吸收至：[[06_机器学习/0606_推荐与预测/0606_核心知识点/时序预测建模与验证边界|时序预测建模与验证边界]]
---
title: darts，全能的预测专家！
author: 沉沙的钟
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODc5Nzc0MA==&mid=2247484466&idx=1&sn=4b89c5b3a4054366fb054f05ab04925d&chksm=97ea07a4b2cc40de5b8d5f288537962bd9ced8f70656dfd3946e6d83fa98e38112e243c4e36c&mpshare=1&scene=24&srcid=1119jJxdQpTGAvgTNX0xEbyV&sharer_shareinfo=8a372e234adbea2fcae3e366ab9b94c6&sharer_shareinfo_first=8a372e234adbea2fcae3e366ab9b94c6#rd
---

在数据分析领域，时间序列预测一直是个重要且复杂的任务。

Python Darts模块就像一位全能的预测专家，集成了从传统统计方法到现代深度学习的多种预测模型，为时序分析提供了统一而强大的工具集。

🎯 基础时序操作

Darts的核心是TimeSeries对象，它封装了时间索引和数值，提供了丰富的操作方法。

```
from darts import TimeSeries
import pandas as pd

dates = pd.date_range('2023-01-01', periods=100)
values = range(1, 101)
series = TimeSeries.from_times_and_values(dates, values)

print(f"序列长度: {len(series)}")
print(f"起始时间: {series.start_time()}")
```

这种统一的数据结构简化了时间序列的存储和操作，为后续分析奠定基础。

🔮 多种预测模型

Darts集成了数十种预测模型，从简单的Naive预测器到复杂的深度学习模型。

```
from darts.models import ExponentialSmoothing, Theta

# 分割数据
train, val = series[:80], series[80:]

# 训练多个模型
models = [ExponentialSmoothing(), Theta()]
forecasts = []

for model in models:
    model.fit(train)
    forecast = model.predict(len(val))
    forecasts.append(forecast)
```

统一的API设计让用户可以轻松比较不同模型的预测效果。

📊 模型评估比较

Darts提供了丰富的评估指标，帮助选择最优预测模型。

```
from darts.metrics import mape, mae

best_model = None
best_score = float('inf')

for i, forecast in enumerate(forecasts):
    score = mape(val, forecast)
    print(f"模型{i+1} MAPE: {score:.2f}%")
    
    if score < best_score:
        best_score = score
        best_model = models[i]
```

自动化评估流程让模型选择更加科学和直观。

🔄 协变量支持

Darts支持使用外部变量提升预测精度，适合复杂业务场景。

```
from darts import TimeSeries
import numpy as np

# 创建协变量
main_series = TimeSeries.from_values(np.sin(np.arange(100) * 0.1))
covariate = TimeSeries.from_values(np.arange(100) * 0.05)

from darts.models import RegressionModel
model = RegressionModel(lags=5, lags_past_covariates=5)
model.fit(main_series, past_covariates=covariate)
```

协变量机制让模型能够考虑更多影响因素，提升预测准确性。

⚡ 实战销售预测

将Darts应用于真实的销售预测场景，展示其实际价值。

```
from darts.models import AutoARIMA
from darts.datasets import AirPassengersDataset

# 加载航空乘客数据
series = AirPassengersDataset().load()

# 自动ARIMA模型
model = AutoARIMA()
model.fit(series)
forecast = model.predict(24)

print(f"未来两年预测完成")
print(f"最后预测值: {forecast[-1].value():.0f}人")
```

这个示例展示了Darts在经典时间序列数据集上的应用效果。

⚖️ 优势分析

相比单一预测库，Darts提供了统一的API和丰富的模型选择。

建议在需要全面时序分析能力的项目中使用。

🌟 互动总结

Darts让时间序列预测变得更加简单高效。你在时序预测中遇到过什么挑战？欢迎在评论区分享经验！
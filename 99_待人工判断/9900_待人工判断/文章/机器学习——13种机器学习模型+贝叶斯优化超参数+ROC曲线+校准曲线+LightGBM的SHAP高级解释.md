---
title: 机器学习——13种机器学习模型+贝叶斯优化超参数+ROC曲线+校准曲线+LightGBM的SHAP高级解释
author: Python机器学习ML
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247485452&idx=1&sn=27aab39d59361206598ce8dcbd9787dd&chksm=c0ca340f9b6a7cc302d00c72e7aba58b0b0c523ef85588bd42650008fcbab1126225dc7a6572&mpshare=1&scene=24&srcid=0806UFqsyQQNf5rKvXW91KGk&sharer_shareinfo=fe331941caaa089c14b71f39cbd6a8c7&sharer_shareinfo_first=fe331941caaa089c14b71f39cbd6a8c7#rd
---

> 感谢关注，一起来学习干货。

## 本期摘要

1.利用13种机器学习模型构建糖尿病诊断模型，输出有关混淆矩阵的灵敏度、特异度、准确度、阴性预测值（NPV）、阳性预测值（PPV）、召回率、约登指数、F1 八大分类性能评估指标。

2.绘制13种机器学习算法的ROC曲线、auc森林图、混淆矩阵、校准曲线。

3.针对最优机器学习模型做SHAP解释分析

本项目构成了一个端到端的、高度规范的机器学习项目流程，其核心目标是**基于一个糖尿病数据集，构建一个高性能、可解释、且具有实际应用价值的预测模型**。整个过程可以清晰地划分为三个核心阶段：**探索与洞察**、**构建与评估**、**解释与应用**。

## 第一部分：探索性数据分析（EDA）

糖尿病数据集探索性数据分析（EDA）的Python代码。整个过程将被分解为十个阶段，每个阶段都会有详细的介绍和逐行代码注释，确保您能理解每一行代码的作用，并能将其应用到其他数据集中。

---

### **阶段一：环境准备与数据加载**

**阶段说明：** 这个阶段是所有数据分析工作的起点。我们首先导入所有需要用到的Python库，并为它们设置别名以便后续方便调用。然后，我们进行一些全局设置，如忽略不必要的警告信息和设定图表的美化风格。最后，我们将数据集从CSV文件中加载到Pandas的DataFrame中，这是我们进行数据操作和分析的核心数据结构。

**代码及逐行注释：**

python

```
import numpy as np  # 导入NumPy库，用于进行高效的数值计算，特别是数组操作，通常简写为np。  
import pandas as pd # 导入Pandas库，用于数据处理和分析，提供了DataFrame这一强大的数据结构，通常简写为pd。  
import matplotlib.pyplot as plt # 导入Matplotlib的pyplot模块，是Python中最基础的绘图库，通常简写为plt。  
import seaborn as sns # 导入Seaborn库，它基于Matplotlib，提供了更高级、更美观的统计图表，通常简写为sns。  
import warnings # 导入warnings模块，用于控制警告信息的输出。  
from scipy import stats # 从SciPy库中导入stats模块，它包含了大量的统计函数和检验方法。  
from sklearn.preprocessing import StandardScaler # 从Scikit-learn库中导入StandardScaler，用于数据标准化处理。  
import plotly.express as px # 导入Plotly Express库，用于创建交互式图表，通常简写为px。  
import plotly.graph_objects as go # 导入Plotly的graph_objects模块，用于构建更复杂的自定义交互式图表。  
from plotly.subplots import make_subplots # 从Plotly中导入make_subplots，用于创建包含多个子图的交互式图表。  
  
# 忽略警告  
warnings.filterwarnings('ignore') # 设置警告过滤器，'ignore'参数表示忽略所有警告信息，让输出更整洁。  
  
# 设置绘图风格 (移除中文字体设置)  
sns.set_style("whitegrid") # 使用Seaborn设置图表的默认风格为"whitegrid"，即带有白色网格背景的样式。  
sns.set_palette("husl") # 使用Seaborn设置图表的默认调色板为"husl"，提供了一套颜色鲜艳且分布均匀的色彩方案。  
  
# 数据加载  
data = pd.read_csv(r"公众号python机器学习ML-diabetes_data.csv") # 使用Pandas的read_csv函数读取指定路径下的CSV文件，并将其内容加载到一个名为'data'的DataFrame对象中。  
  
print("=" * 60) # 打印60个等号，作为报告标题的装饰线。  
print("📊 糖尿病数据集 - 探索性数据分析报告") # 打印报告的主标题。  
print("=" * 60) # 再次打印60个等号，作为报告标题的装饰线。
```

---

### **阶段二：数据基本信息概览**

**阶段说明：** 在加载数据后，第一步是快速了解数据集的全貌。这个阶段的目标是获取关于数据集的基本元数据，包括：

* **形状（Shape）**

  数据集有多少行（样本）和多少列（特征）。
* **信息（Info）**

  每列的数据类型是什么，以及是否有缺失值。
* **预览（Head）**

  查看数据的前几行，对数据内容有一个直观的感受。
* **缺失值（Missing Values）**

  系统地检查每一列的缺失值数量。
* **数据类型（Data Types）**

  统计不同数据类型的列各有多少。

这些信息是后续所有分析的基础，可以帮助我们提前发现潜在的数据问题。

**代码及逐行注释：**

python

```
# ==================== 第一部分：数据基本信息 ====================  
print("\n🔍 1. 数据基本信息") # 打印第一部分的标题。  
print("-" * 40) # 打印40个短横线，作为分隔符。  
print(f"数据集形状: {data.shape}") # 使用f-string格式化输出，打印数据集的形状（行数, 列数）。data.shape返回一个元组。  
print(f"特征数量: {data.shape[1] - 1}") # 打印特征的数量。总列数（data.shape[1]）减去1个目标变量列。  
print(f"样本数量: {data.shape[0]}") # 打印样本的数量，即数据集的行数（data.shape[0]）。  
  
print(f"\n📋 数据集信息:") # 打印"数据集信息"的子标题。  
print(data.info()) # 调用DataFrame的info()方法，它会打印出一个简洁的摘要，包括索引类型、列名、非空值数量和内存使用情况。  
  
print(f"\n👀 前5行数据:") # 打印"前5行数据"的子标题。  
print(data.head()) # 调用head()方法，默认显示DataFrame的前5行，用于快速预览数据内容。  
  
print(f"\n🔍 缺失值检查:") # 打印"缺失值检查"的子标题。  
missing_values = data.isnull().sum() # data.isnull()将每个单元格的值转为True（如果是缺失值）或False。.sum()会按列求和，计算每列的缺失值总数。  
print(missing_values) # 打印每列的缺失值数量。  
if missing_values.sum() == 0: # 检查所有列的缺失值总和是否为0。  
print("✅ 数据集无缺失值") # 如果总和为0，则打印无缺失值的确认信息。  
else: # 否则  
print("❌ 发现缺失值，需要处理") # 打印发现缺失值的警告信息。  
  
print(f"\n📊 数据类型分布:") # 打印"数据类型分布"的子标题。  
print(data.dtypes.value_counts()) # data.dtypes返回每列的数据类型。.value_counts()统计这些数据类型出现的次数。
```

---

### **阶段三：目标变量分析**

**阶段说明：** 在分类问题中，深入分析目标变量（也叫标签或因变量）至关重要。这个阶段我们专门研究`Diagnosis`列，即病人是否患有糖尿病的诊断结果。我们会分析：

* **类别分布**

  患病和不患病的人数分别是多少，占比如何。
* **类别平衡性**

  两个类别的样本数量是否均衡。如果一个类别远多于另一个，这被称为“类别不平衡”问题，可能会影响模型训练。

通过柱状图、饼图和环形图，我们从不同角度将这种分布可视化，使其更加直观。

**代码及逐行注释：**

python

```
# ==================== 第二部分：目标变量分析 ====================  
print("\n🎯 2. 目标变量分析") # 打印第二部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 目标变量分布  
target_counts = data['Diagnosis'].value_counts() # 对'Diagnosis'列进行计数，统计每个唯一值（0和1）出现的次数。  
target_pct = data['Diagnosis'].value_counts(normalize=True) * 100# normalize=True使value_counts()返回频率而非次数，乘以100得到百分比。  
  
print("目标变量分布:") # 打印子标题。  
for i inrange(len(target_counts)): # 遍历目标变量的两个类别。  
    label = "无糖尿病"if target_counts.index[i] == 0else"有糖尿病"# 根据索引值（0或1）确定标签文本。  
print(f"  {label} (Diagnosis={target_counts.index[i]}): {target_counts.iloc[i]} 个样本 ({target_pct.iloc[i]:.1f}%)") # 格式化输出每个类别的样本数和百分比。  
  
# 可视化目标变量分布  
fig, axes = plt.subplots(1, 3, figsize=(18, 5)) # 创建一个包含1行3列的子图画布，总尺寸为18x5英寸。fig是整个画布，axes是包含3个子图对象的数组。  
  
# 柱状图  
colors = ['#2ecc71', '#e74c3c'] # 定义用于绘图的颜色列表（绿色和红色）。  
bars = axes[0].bar(['No Diabetes', 'Diabetes'], target_counts.values, color=colors, alpha=0.8) # 在第一个子图(axes[0])上绘制柱状图。x轴是类别标签，y轴是计数值。  
axes[0].set_title('Diabetes Diagnosis Distribution', fontsize=14, fontweight='bold') # 设置第一个子图的标题，并指定字体大小和粗细。  
axes[0].set_ylabel('Sample Count') # 设置第一个子图的y轴标签。  
# 添加数值标签  
for bar, count inzip(bars, target_counts.values): # 遍历每个柱子及其对应的高度值。  
    axes[0].text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 5, # 在柱子顶部的中心位置添加文本。  
str(count), ha='center', va='bottom', fontweight='bold') # 文本内容是计数值，水平居中(ha)，垂直对齐到底部(va)。  
  
# 饼图  
wedges, texts, autotexts = axes[1].pie(target_counts.values, # 在第二个子图(axes[1])上绘制饼图。  
                                       labels=['No Diabetes', 'Diabetes'], # 设置每个扇区的标签。  
                                       colors=colors, # 设置每个扇区的颜色。  
                                       autopct='%1.1f%%', # 设置扇区内显示的百分比格式。  
                                       startangle=90, # 设置饼图的起始角度为90度（从顶部开始）。  
                                       explode=(0.05, 0)) # 设置第一个扇区（No Diabetes）稍微突出显示。  
axes[1].set_title('Diabetes Diagnosis Proportion', fontsize=14, fontweight='bold') # 设置第二个子图的标题。  
  
# 环形图  
center_circle = plt.Circle((0, 0), 0.70, fc='white') # 创建一个白色圆形，半径为0.7，用于覆盖饼图中心，形成环形图效果。  
axes[2].pie(target_counts.values, labels=['No Diabetes', 'Diabetes'], colors=colors, # 在第三个子图(axes[2])上绘制饼图。  
            autopct='%1.1f%%', startangle=90, pctdistance=0.85) # pctdistance=0.85表示百分比标签距离圆心的距离。  
axes[2].add_artist(center_circle) # 将创建的白色圆形添加到子图上。  
axes[2].set_title('Diabetes Diagnosis Donut Chart', fontsize=14, fontweight='bold') # 设置第三个子图的标题。  
  
plt.tight_layout() # 自动调整子图参数，使其填充整个画布区域，避免标签重叠。  
plt.savefig('target_distribution.png', dpi=300, bbox_inches='tight') # 将生成的图表保存为PNG文件，分辨率为300dpi，bbox_inches='tight'确保所有元素都被保存。  
plt.show() # 显示图表。
```

---

### **阶段四：特征变量分布分析**

**阶段说明：** 这个阶段我们将注意力从目标变量转移到预测特征上。我们的目标是理解每个数值型特征自身的分布情况。我们会：

1. **识别数值特征**

   从数据集中筛选出所有数值类型的列。
2. **计算描述性统计**

   使用`.describe()`方法快速获取每个特征的均值、标准差、中位数、最小值、最大值等核心统计指标。这能帮我们发现数据的大致范围和中心趋势。
3. **可视化分布**

   对每个特征绘制直方图和核密度估计（KDE）曲线。直方图显示了数据在不同区间内的频数，而KDE曲线则是对真实分布的平滑估计。我们还会特别标记出均值和中位数，以观察分布的对称性或偏斜度。

**代码及逐行注释：**

python

```
# ==================== 第三部分：特征变量分布分析 ====================  
print("\n📈 3. 特征变量分布分析") # 打印第三部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 获取数值特征  
numeric_features = data.select_dtypes(include=[np.number]).columns.tolist() # select_dtypes选择数据类型为数值（整数、浮点数）的列，.columns获取列名，.tolist()转为列表。  
if'Diagnosis'in numeric_features: # 检查'Diagnosis'是否在数值特征列表中。  
    numeric_features.remove('Diagnosis') # 如果在，则将其移除，因为我们想单独分析预测特征。  
  
print(f"数值特征: {numeric_features}") # 打印所有数值型预测特征的列表。  
  
# 统计描述  
print("\n📊 特征统计描述:") # 打印子标题。  
print(data[numeric_features].describe().round(2)) # 对所有数值特征计算描述性统计，并使用.round(2)将结果四舍五入到小数点后两位。  
  
# 特征分布可视化  
n_features = len(numeric_features) # 获取数值特征的数量。  
n_cols = 3# 指定我们希望每行显示3个子图。  
n_rows = (n_features + n_cols - 1) // n_cols # 计算需要的行数，这是一个向上取整的技巧。  
  
fig, axes = plt.subplots(n_rows, n_cols, figsize=(15, 5 * n_rows)) # 根据计算出的行列数创建子图网格。  
axes = axes.flatten() if n_rows > 1else [axes] if n_cols == 1else axes # 将axes数组扁平化为一维数组，方便通过单个索引进行遍历。  
  
for i, feature inenumerate(numeric_features): # 遍历所有数值特征及其索引。  
# 直方图 + 密度曲线  
    sns.histplot(data[feature], kde=True, ax=axes[i], alpha=0.7) # 在对应的子图(axes[i])上绘制直方图和KDE曲线。  
    axes[i].set_title(f'{feature} Distribution', fontweight='bold') # 设置子图标题。  
    axes[i].set_xlabel(feature) # 设置子图的x轴标签。  
    axes[i].set_ylabel('Frequency') # 设置子图的y轴标签。  
  
# 添加统计信息  
    mean_val = data[feature].mean() # 计算当前特征的均值。  
    median_val = data[feature].median() # 计算当前特征的中位数。  
    axes[i].axvline(mean_val, color='red', linestyle='--', label=f'Mean: {mean_val:.2f}') # 在图上绘制一条红色的虚线代表均值。  
    axes[i].axvline(median_val, color='green', linestyle='--', label=f'Median: {median_val:.2f}') # 在图上绘制一条绿色的虚线代表中位数。  
    axes[i].legend() # 显示图例（包含了均值和中位数的标签）。  
  
# 隐藏多余的子图  
for j inrange(i + 1, len(axes)): # 遍历最后一个已绘制的子图之后的所有空子图。  
    axes[j].set_visible(False) # 将这些多余的子图隐藏起来。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('feature_distributions.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段五：特征与目标变量关系分析**

**阶段说明：** 理解了单个特征的分布后，下一步是探索这些特征与目标变量`Diagnosis`之间的关系。这个阶段的核心问题是：“某个特征的值在患病组和非患病组之间是否存在显著差异？” 我们使用\*\*箱线图（Boxplot）\*\*来回答这个问题。箱线图能清晰地展示：

* **中位数**

  每组数据的中心位置。
* **四分位距（IQR）**

  每组数据的离散程度。
* **异常值**

  可能存在的极端值。

通过并排比较两组（`Diagnosis=0`和`Diagnosis=1`）的箱线图，我们可以直观地判断一个特征是否对区分糖尿病有帮助。我们还在图上用菱形标记了每组的均值，提供了另一个比较中心趋势的指标。

**代码及逐行注释：**

python

```
# ==================== 第四部分：特征与目标变量关系分析 ====================  
print("\n🔗 4. 特征与目标变量关系分析") # 打印第四部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 按目标变量分组的特征分布对比  
fig, axes = plt.subplots(n_rows, n_cols, figsize=(15, 5 * n_rows)) # 同样，创建子图网格来展示每个特征。  
axes = axes.flatten() if n_rows > 1else [axes] if n_cols == 1else axes # 扁平化axes数组。  
  
for i, feature inenumerate(numeric_features): # 遍历所有数值特征。  
# 分组箱线图  
    sns.boxplot(data=data, x='Diagnosis', y=feature, ax=axes[i]) # 在对应子图上绘制箱线图。x轴是分组变量'Diagnosis'，y轴是数值特征'feature'。  
    axes[i].set_title(f'{feature} by Diagnosis Group', fontweight='bold') # 设置子图标题。  
    axes[i].set_xlabel('Diagnosis (0:No Diabetes, 1:Diabetes)') # 设置x轴标签，并解释0和1的含义。  
  
# 添加均值标记  
for j, diagnosis inenumerate([0, 1]): # 遍历诊断结果的两个类别（0和1）。  
        subset = data[data['Diagnosis'] == diagnosis][feature] # 筛选出对应诊断结果的子数据集。  
        mean_val = subset.mean() # 计算该子数据集的均值。  
        axes[i].plot(j, mean_val, marker='D', markersize=8, color='red') # 在箱线图上用红色菱形(D)标记出均值位置。  
  
# 隐藏多余的子图  
for j inrange(i + 1, len(axes)): # 隐藏网格中未使用的子图。  
    axes[j].set_visible(False)  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('feature_target_relationship.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段六：相关性分析**

**阶段说明：** 这个阶段我们分析特征与特征之间，以及特征与目标变量之间的**线性相关性**。这有助于：

1. **特征选择**

   找出与目标变量`Diagnosis`相关性最强的特征，它们可能是最重要的预测因子。
2. **识别多重共线性**

   发现两个或多个预测特征之间的高度相关。高度相关的特征提供了冗余信息，可能会在某些机器学习模型中引发问题。

我们通过计算**皮尔逊相关系数矩阵**并用\*\*热力图（Heatmap）\*\*将其可视化来实现这一目标。热力图用颜色深浅和数值来直观展示相关性的强度和方向（正相关或负相关）。

**代码及逐行注释：**

python

```
# ==================== 第五部分：相关性分析 ====================  
print("\n🌡️ 5. 特征相关性分析") # 打印第五部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 计算相关性矩阵  
correlation_matrix = data[numeric_features + ['Diagnosis']].corr() # 选取所有数值特征和目标变量，调用.corr()方法计算它们之间的皮尔逊相关系数矩阵。  
print("与目标变量的相关性（按绝对值排序）:") # 打印子标题。  
target_corr = correlation_matrix['Diagnosis'].abs().sort_values(ascending=False) # 提取'Diagnosis'列与其他所有变量的相关系数，取绝对值，并按降序排列。  
print(target_corr[target_corr.index != 'Diagnosis']) # 打印排序后的相关性，并排除'Diagnosis'自身（与自身相关性恒为1）。  
  
# 增强版相关性热力图  
plt.figure(figsize=(12, 10)) # 创建一个12x10英寸的新画布。  
mask = np.triu(np.ones_like(correlation_matrix, dtype=bool)) # 创建一个上三角矩阵的掩码(mask)。np.ones_like创建一个与相关矩阵同样形状的全1矩阵，np.triu保留其上三角部分。  
sns.heatmap(correlation_matrix, # 绘制热力图。  
            mask=mask, # 应用掩码，只显示下三角部分（避免信息重复）。  
            annot=True, # 在每个单元格中显示数值（相关系数）。  
            fmt=".3f", # 设置数值的格式为保留三位小数的浮点数。  
            cmap='RdBu_r', # 设置颜色映射方案为'RdBu_r'（红-白-蓝，r表示反转，红色代表正相关，蓝色代表负相关）。  
            square=True, # 使每个单元格都为正方形。  
            linewidths=0.5, # 在单元格之间添加宽度为0.5的线条。  
            cbar_kws={"shrink": .8}) # 调整颜色条(cbar)的大小，使其为原始大小的80%。  
plt.title('Feature Correlation Heatmap (Lower Triangle)', fontsize=16, fontweight='bold', pad=20) # 设置图表标题。  
plt.xticks(rotation=45, ha='right') # 将x轴的标签旋转45度，并右对齐，防止重叠。  
plt.yticks(rotation=0) # 保持y轴标签不旋转。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('enhanced_correlation_heatmap.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段七：异常值检测**

**阶段说明：** 异常值（Outliers）是数据集中与其他观测值差异极大的数据点。它们可能是由于测量错误、数据输入错误或真实但极端的事件造成的。异常值会严重影响统计分析（如均值）和某些机器学习模型的性能。 这个阶段，我们使用一种常用的方法——**IQR（四分位距）法**来检测异常值：

* 计算每个特征的Q1（第25百分位数）和Q3（第75百分位数）。
* 计算IQR = Q3 - Q1。
* 定义异常值的边界：下界 = Q1 - 1.5 \* IQR，上界 = Q3 + 1.5 \* IQR。
* 任何低于下界或高于上界的数据点都被视为异常值。

我们会统计每个特征的异常值数量，并通过箱线图和柱状图进行可视化。

**代码及逐行注释：**

python

```
# ==================== 第六部分：异常值检测 ====================  
print("\n🚨 6. 异常值检测") # 打印第六部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 使用IQR方法检测异常值的函数  
defdetect_outliers_iqr(data, feature): # 定义一个函数，用于检测指定特征的异常值。  
    Q1 = data[feature].quantile(0.25) # 计算第一四分位数（25%）。  
    Q3 = data[feature].quantile(0.75) # 计算第三四分位数（75%）。  
    IQR = Q3 - Q1 # 计算四分位距。  
    lower_bound = Q1 - 1.5 * IQR # 定义异常值的下界。  
    upper_bound = Q3 + 1.5 * IQR # 定义异常值的上界。  
    outliers = data[(data[feature] < lower_bound) | (data[feature] > upper_bound)] # 筛选出所有低于下界或高于上界的数据点。  
return outliers, lower_bound, upper_bound # 返回包含异常值数据的DataFrame以及上下界。  
  
# 异常值统计  
outlier_summary = {} # 创建一个空字典，用于存储每个特征的异常值信息。  
for feature in numeric_features: # 遍历所有数值特征。  
    outliers, lower, upper = detect_outliers_iqr(data, feature) # 调用函数检测异常值。  
    outlier_summary[feature] = { # 将结果存入字典。  
'count': len(outliers), # 异常值的数量。  
'percentage': len(outliers) / len(data) * 100, # 异常值占总样本的百分比。  
'lower_bound': lower, # 下界。  
'upper_bound': upper # 上界。  
    }  
  
print("异常值检测结果 (使用IQR方法):") # 打印子标题。  
for feature, info in outlier_summary.items(): # 遍历字典，打印每个特征的异常值统计。  
print(f"  {feature}: {info['count']} 个异常值 ({info['percentage']:.1f}%)")  
  
# 异常值可视化  
fig, axes = plt.subplots(2, 1, figsize=(15, 10)) # 创建一个2行1列的子图画布。  
  
# 箱线图显示异常值  
data_melted = pd.melt(data[numeric_features], var_name='Feature', value_name='Value') # 使用melt函数将宽格式数据转换为长格式，方便用Seaborn一次性绘制所有特征的箱线图。  
sns.boxplot(data=data_melted, x='Feature', y='Value', ax=axes[0]) # 在第一个子图上绘制所有特征的箱线图。  
axes[0].set_title('Boxplot of All Features - Outlier Detection', fontweight='bold') # 设置标题。  
axes[0].tick_params(axis='x', rotation=45) # 旋转x轴标签以防重叠。  
  
# 异常值数量柱状图  
features = list(outlier_summary.keys()) # 获取特征名称列表。  
outlier_counts = [outlier_summary[f]['count'] for f in features] # 获取对应的异常值数量列表。  
bars = axes[1].bar(features, outlier_counts, alpha=0.7, color='coral') # 在第二个子图上绘制柱状图显示每个特征的异常值数量。  
axes[1].set_title('Number of Outliers by Feature', fontweight='bold') # 设置标题。  
axes[1].set_ylabel('Number of Outliers') # 设置y轴标签。  
axes[1].tick_params(axis='x', rotation=45) # 旋转x轴标签。  
  
# 添加数值标签  
for bar, count inzip(bars, outlier_counts): # 遍历每个柱子和其高度。  
    axes[1].text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.1, # 在柱子顶部添加数值标签。  
str(count), ha='center', va='bottom')  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('outlier_detection.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段八：高级可视化与综合分析**

**阶段说明：** 这个阶段我们使用更高级的可视化工具来探索特征之间的多维关系，并对分析结果进行汇总。

1. **配对图（Pairplot）**

   它能同时展示多个特征两两之间的散点图，以及每个特征自身的分布图。这对于发现特征间的非线性关系和不同类别（患病/非患病）在特征空间中的分离情况非常有帮助。我们只选择与目标变量相关性最高的几个特征来绘制，以保持图表的可读性。
2. **特征重要性图**

   虽然严格的特征重要性通常来自模型训练，但我们可以基于与目标变量的相关性大小来创建一个“代理”重要性排序。我们用水平条形图来展示，条形长度代表相关性大小，颜色代表相关性方向（正或负）。

**代码及逐行注释：**

python

```
# ==================== 第七部分：高级可视化分析 ====================  
print("\n🎨 7. 高级可视化分析") # 打印第七部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 成对关系图 (只选择相关性较高的几个特征)  
high_corr_features = target_corr.head(6).index.tolist() # 从之前排序的相关性结果中，选择最相关的6个特征。  
if'Diagnosis'in high_corr_features: # 确保'Diagnosis'本身不在列表中。  
    high_corr_features.remove('Diagnosis') # 移除'Diagnosis'。  
high_corr_features = high_corr_features[:4] # 只保留前4个最相关的预测特征。  
  
print(f"选择相关性最高的特征进行成对关系分析: {high_corr_features}") # 打印所选的特征。  
  
plt.figure(figsize=(12, 10)) # 创建一个新画布。  
sns.pairplot(data[high_corr_features + ['Diagnosis']], # 使用Seaborn绘制配对图。  
             hue='Diagnosis', # 使用'Diagnosis'列对数据点进行着色，区分患病与非患病组。  
             diag_kind='kde', # 对角线上绘制核密度估计图（KDE）。  
             plot_kws={'alpha': 0.6}) # 设置散点图的透明度为0.6。  
plt.suptitle('Pairplot of Highly Correlated Features', y=1.02, fontsize=16, fontweight='bold') # 添加一个总标题，并调整其位置。  
plt.savefig('pairplot_analysis.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 特征重要性可视化（基于相关性）  
plt.figure(figsize=(10, 6)) # 创建一个新画布。  
feature_importance = target_corr[target_corr.index != 'Diagnosis'].abs().sort_values(ascending=True) # 获取与目标的相关性（绝对值），并按升序排列（为了绘图时最重要的在顶部）。  
colors = ['green'if x > 0else'red'for x in correlation_matrix.loc[feature_importance.index, 'Diagnosis']] # 根据原始相关性的正负决定条形图的颜色（正为绿，负为红）。  
bars = plt.barh(range(len(feature_importance)), feature_importance.values, color=colors, alpha=0.7) # 绘制水平条形图。  
plt.yticks(range(len(feature_importance)), feature_importance.index) # 设置y轴刻度标签为特征名。  
plt.xlabel('Absolute Correlation Coefficient with Target') # 设置x轴标签。  
plt.title('Feature Importance Analysis (Based on Correlation)', fontweight='bold', fontsize=14) # 设置标题。  
plt.grid(axis='x', alpha=0.3) # 添加x轴方向的网格线。  
  
# 添加数值标签  
for i, (bar, val) inenumerate(zip(bars, feature_importance.values)): # 遍历每个条形及其值。  
    plt.text(val + 0.01, bar.get_y() + bar.get_height() / 2, # 在条形末端添加数值标签。  
f'{val:.3f}', va='center', fontweight='bold')  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('feature_importance.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 分组统计分析  
print("\n📋 8. 分组统计分析") # 打印第八部分的标题。  
print("-" * 40) # 打印分隔符。  
  
grouped_stats = data.groupby('Diagnosis')[numeric_features].agg(['mean', 'std', 'median']).round(3) # 按'Diagnosis'分组，对所有数值特征计算均值、标准差和中位数，并保留三位小数。  
pd.set_option('display.max_columns', None) # 设置Pandas以显示所有列，防止输出被截断。  
print("按诊断结果分组的统计信息:") # 打印子标题。  
print(grouped_stats) # 打印分组统计结果。  
  
# 特征分布的violin图  
fig, axes = plt.subplots(2, 2, figsize=(15, 10)) # 创建一个2x2的子图网格。  
axes = axes.flatten() # 扁平化axes数组。  
  
top_features = high_corr_features[:4] # 选择最相关的4个特征。  
for i, feature inenumerate(top_features): # 遍历这4个特征。  
    sns.violinplot(data=data, x='Diagnosis', y=feature, ax=axes[i]) # 绘制小提琴图，它结合了箱线图和KDE图，能更好地展示数据分布的密度。  
    axes[i].set_title(f'Distribution Density of {feature}', fontweight='bold') # 设置子图标题。  
    axes[i].set_xlabel('Diagnosis (0:No Diabetes, 1:Diabetes)') # 设置x轴标签。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('violin_plots.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段九：数据质量评估与总结**

**阶段说明：** 这个阶段是对整个EDA过程的一个总结。我们首先生成一个简洁的数据质量报告，量化数据的完整性、重复记录等情况。然后，我们用几句关键的话总结整个分析的核心发现，例如样本和特征数量、目标变量分布、最重要的特征等。这部分内容可以直接用于项目报告的摘要部分。

**代码及逐行注释：**

python

```
# ==================== 第九部分：数据质量评估 ====================  
print("\n✅ 9. 数据质量评估") # 打印第九部分的标题。  
print("-" * 40) # 打印分隔符。  
  
print("数据质量报告:") # 打印子标题。  
print(f"  • 数据完整性: {(1 - data.isnull().sum().sum() / (data.shape[0] * data.shape[1])) * 100:.1f}%") # 计算并打印数据完整度百分比。  
print(f"  • 重复记录: {data.duplicated().sum()} 条") # 计算并打印完全重复的行数。  
print(f"  • 数据类型一致性: ✅"if data.select_dtypes(include=[object]).empty else"  • 数据类型一致性: ⚠️ 存在文本类型") # 检查是否存在对象（通常是文本）类型，如果不存在则认为类型一致性好。  
  
# 总结报告  
print("\n" + "=" * 60) # 打印分隔线。  
print("📊 EDA 分析总结") # 打印总结标题。  
print("=" * 60) # 打印分隔线。  
print(f"✅ 数据集包含 {data.shape[0]} 个样本，{data.shape[1] - 1} 个特征") # 总结数据集大小。  
print(f"✅ 目标变量分布: 无糖尿病 {target_pct.iloc[0]:.1f}%，有糖尿病 {target_pct.iloc[1]:.1f}%") # 总结目标变量分布。  
print(f"✅ 相关性最高的特征: {target_corr.index[1]} (相关系数: {target_corr.iloc[1]:.3f})") # 总结相关性最高的特征。  
print(f"✅ 异常值最多的特征: {max(outlier_summary.keys(), key=lambda x: outlier_summary[x]['count'])}") # 总结异常值最多的特征。  
print("=" * 60) # 打印分隔线。
```

---

### **阶段十：统计分析优化与深化**

**阶段说明：** 这是一个更高级的分析阶段，超越了基本的EDA，进入了推断性统计的范畴。此阶段的目标是使用更严谨的统计方法来量化和验证我们在前几个阶段的观察结果。

1. **显著性检验 (t-检验 / Mann-Whitney U检验)**

   检验患病组和非患病组在某个特征上的均值（或分布）差异是否“统计上显著”，而不仅仅是随机波动。t检验适用于正态分布数据，Mann-Whitney U检验是其非参数替代方案。
2. **效应量 (Cohen's d)**

   量化差异的“大小”。一个非常显著的p值可能对应一个很小的实际差异。效应量告诉我们这个差异在实际中有多大。
3. **多重比较校正 (Bonferroni / FDR)**

   当我们对多个特征进行检验时，偶然出现假阳性（即错误地认为有差异）的概率会增加。多重比较校正（如FDR）调整p值，以控制总体错误率。
4. **Bootstrap置信区间**

   通过重复抽样的方法，为均值等统计量估算一个更稳健的置信区间，而无需假设数据服从特定分布。
5. **综合可视化**

   创建一个集大成的图表，将原始数据分布、均值、置信区间、p值和效应量全部展示出来，提供一个关于特征差异的完整画面。
6. **带显著性标记的相关性热力图**

   在之前的相关性热力图基础上，添加星号（\*）来标记那些统计上显著相关的特征对。

这个阶段提供了更强的证据来支持你的结论，是从探索性分析到严谨研究报告的关键一步。

**代码及逐行注释：**

python

```
# ==================== 第十部分：统计分析优化 ====================  
print("\n📊 10. 统计分析优化") # 打印第十部分的标题。  
print("-" * 40) # 打印分隔符。  
  
# 选择数值特征和目标变量  
target = data['Diagnosis'] # 将目标变量'Diagnosis'存储在target变量中。  
numeric_data = data[numeric_features] # 将所有数值预测特征存储在numeric_data中。  
  
# 1. 统计显著性检验 - t检验  
print("\n🔍 t检验结果:") # 打印子标题。  
t_test_results = {} # 创建一个空字典来存储t检验结果。  
for feature in numeric_features: # 遍历所有数值特征。  
    group_0 = numeric_data[feature][target == 0] # 筛选出非患病组的数据。  
    group_1 = numeric_data[feature][target == 1] # 筛选出患病组的数据。  
    t_stat, p_value = stats.ttest_ind(group_0, group_1) # 对两组独立样本进行t检验，返回t统计量和p值。  
    t_test_results[feature] = {'t-statistic': t_stat, 'p-value': p_value} # 将结果存入字典。  
  
# 输出t检验结果  
t_test_df = pd.DataFrame(t_test_results).T # 将结果字典转换为DataFrame并转置，使特征名为行索引。  
print(t_test_df) # 打印t检验结果表。  
  
# 2. 非参数检验 - Mann-Whitney U检验  
print("\n🔍 Mann-Whitney U检验结果:") # 打印子标题。  
mw_test_results = {} # 创建一个空字典来存储Mann-Whitney U检验结果。  
for feature in numeric_features: # 遍历所有数值特征。  
    group_0 = numeric_data[feature][target == 0] # 筛选非患病组。  
    group_1 = numeric_data[feature][target == 1] # 筛选患病组。  
    stat, p_value = stats.mannwhitneyu(group_0, group_1) # 对两组进行Mann-Whitney U检验。  
    mw_test_results[feature] = {'U-statistic': stat, 'p-value': p_value} # 存储结果。  
  
mw_test_df = pd.DataFrame(mw_test_results).T # 转换为DataFrame并转置。  
print(mw_test_df) # 打印结果。  
  
# 3. 计算效应量  
defcompute_effect_size(group1, group2): # 定义一个函数来计算Cohen's d效应量。  
"""计算Cohen's D效应量"""  
    diff = np.mean(group1) - np.mean(group2) # 计算两组的均值差。  
    pooled_std = np.sqrt((np.var(group1, ddof=1) + np.var(group2, ddof=1)) / 2) # 计算合并标准差。  
return diff / pooled_std # 返回效应量。  
  
print("\n🔍 效应量结果:") # 打印子标题。  
effect_size_results = {} # 创建空字典。  
for feature in numeric_features: # 遍历特征。  
    group_0 = numeric_data[feature][target == 0] # 非患病组。  
    group_1 = numeric_data[feature][target == 1] # 患病组。  
    effect_size = compute_effect_size(group_1, group_0) # 计算效应量（患病组 vs 非患病组）。  
    effect_size_results[feature] = effect_size # 存储结果。  
  
effect_size_df = pd.DataFrame(effect_size_results, index=['Cohen\'s D']).T # 转换为DataFrame。  
print(effect_size_df) # 打印效应量结果。  
  
# 4. 多重比较校正  
from statsmodels.stats.multitest import multipletests # 从statsmodels库导入多重检验校正函数。  
  
# 获取p值并进行校正  
p_values = t_test_df['p-value'].values # 提取所有t检验的p值。  
  
# Bonferroni校正  
bonferroni_results = multipletests(p_values, method='bonferroni') # 进行Bonferroni校正。  
bonferroni_corrected = bonferroni_results[1] # 提取校正后的p值。  
  
# FDR校正 (Benjamini-Hochberg方法)  
fdr_results = multipletests(p_values, method='fdr_bh') # 进行FDR校正，这是一种更常用的方法。  
fdr_corrected = fdr_results[1] # 提取校正后的p值。  
  
# 添加校正的p值到数据框  
t_test_df['Bonferroni p-value'] = bonferroni_corrected # 将Bonferroni校正后的p值添加到结果表中。  
t_test_df['FDR p-value'] = fdr_corrected # 将FDR校正后的p值添加到结果表中。  
print("\n🔍 多重比较校正后t检验结果:") # 打印子标题。  
print(t_test_df) # 打印包含校正p值的完整结果表。  
  
# 5. Bootstrap置信区间计算  
print("\n🔍 使用Bootstrap计算置信区间:") # 打印子标题。  
  
defbootstrap_ci(data, statistic=np.mean, num_samples=1000, alpha=0.05): # 定义一个函数来计算bootstrap置信区间。  
"""计算Bootstrap置信区间"""  
    bootstrapped_stats = [] # 创建一个空列表来存储每次bootstrap抽样的统计量。  
for _ inrange(num_samples): # 重复抽样num_samples次。  
        sample = np.random.choice(data, size=len(data), replace=True) # 从原始数据中有放回地抽样，样本大小与原始数据相同。  
        bootstrapped_stats.append(statistic(sample)) # 计算抽样样本的统计量（默认为均值）并存储。  
    lower_bound = np.percentile(bootstrapped_stats, 100 * alpha / 2) # 计算置信区间的下界（例如，2.5%分位数）。  
    upper_bound = np.percentile(bootstrapped_stats, 100 * (1 - alpha / 2)) # 计算置信区间的上界（例如，97.5%分位数）。  
return lower_bound, upper_bound, np.mean(bootstrapped_stats) # 返回置信区间的上下界和bootstrap均值。  
  
# 创建存储Bootstrap CI的结果DataFrame  
bootstrap_results = pd.DataFrame( # 创建一个空的DataFrame来存储结果。  
    index=numeric_features,  
    columns=['Mean (0)', 'CI Lower (0)', 'CI Upper (0)', 'Mean (1)', 'CI Lower (1)', 'CI Upper (1)']  
)  
  
for feature in numeric_features: # 遍历所有数值特征。  
# 对0组(无糖尿病)计算CI  
    group_0 = numeric_data[feature][target == 0]  
    ci_low_0, ci_high_0, mean_0 = bootstrap_ci(group_0)  
  
# 对1组(有糖尿病)计算CI  
    group_1 = numeric_data[feature][target == 1]  
    ci_low_1, ci_high_1, mean_1 = bootstrap_ci(group_1)  
  
# 存储结果  
    bootstrap_results.loc[feature] = [  
        mean_0, ci_low_0, ci_high_0, mean_1, ci_low_1, ci_high_1  
    ]  
  
print(bootstrap_results) # 打印两组的bootstrap均值和95%置信区间。  
  
# 6. 可视化比较关键特征  
print("\n🔍 组间差异可视化:") # 打印子标题。  
# 选择最显著的4个特征(基于效应量)  
top_features = effect_size_df.abs().sort_values('Cohen\'s D', ascending=False).index[:4].tolist() # 根据效应量的绝对值大小，选择差异最明显的4个特征。  
  
# 设置图表  
fig, axes = plt.subplots(2, 2, figsize=(16, 12)) # 创建一个2x2的子图网格。  
axes = axes.flatten() # 扁平化axes数组。  
  
for i, feature inenumerate(top_features): # 遍历这4个顶级特征。  
# 获取数据  
    group_0 = numeric_data[feature][target == 0]  
    group_1 = numeric_data[feature][target == 1]  
  
# 绘制箱线图+小提琴图  
    ax = axes[i]  
    sns.violinplot(x='Diagnosis', y=feature, data=data, ax=ax, inner=None, palette=['#2ecc71', '#e74c3c'], alpha=0.5) # 绘制半透明的小提琴图。  
    sns.boxplot(x='Diagnosis', y=feature, data=data, ax=ax, width=0.3, palette=['white', 'white']) # 在小提琴图内部绘制一个较窄的箱线图。  
    sns.stripplot(x='Diagnosis', y=feature, data=data, ax=ax, size=4, jitter=True, alpha=0.6) # 叠加散点图（stripplot）以显示每个数据点。  
  
# 添加均值和置信区间  
    ci_data = bootstrap_results.loc[feature] # 获取该特征的bootstrap结果。  
  
# 绘制均值线  
    ax.hlines(y=ci_data['Mean (0)'], xmin=-0.2, xmax=0.2, color='green', linewidth=2) # 绘制非患病组的均值线。  
    ax.hlines(y=ci_data['Mean (1)'], xmin=0.8, xmax=1.2, color='red', linewidth=2) # 绘制患病组的均值线。  
  
# 绘制置信区间  
    ax.errorbar(x=[0, 1], # x坐标为0和1。  
                y=[ci_data['Mean (0)'], ci_data['Mean (1)']], # y坐标为两组的均值。  
                yerr=[[ci_data['Mean (0)'] - ci_data['CI Lower (0)'], ci_data['Mean (1)'] - ci_data['CI Lower (1)']], # 误差线的下半部分长度。  
                      [ci_data['CI Upper (0)'] - ci_data['Mean (0)'], ci_data['CI Upper (1)'] - ci_data['Mean (1)']]], # 误差线的上半部分长度。  
                fmt='none', capsize=5, ecolor='black', elinewidth=2) # 设置误差线的样式。  
  
# 添加统计信息  
    p_value = t_test_df.loc[feature, 'p-value'] # 获取p值。  
    effect = effect_size_df.loc[feature, 'Cohen\'s D'] # 获取效应量。  
    sig = "***"if p_value < 0.001else"**"if p_value < 0.01else"*"if p_value < 0.05else"ns"# 根据p值确定显著性标记。  
  
    ax.annotate(f"p = {p_value:.4f}{sig}\nCohen's d = {effect:.2f}", # 创建注释文本。  
                xy=(0.5, 0.95), xycoords='axes fraction', # 将注释放在子图的顶部中央位置。  
                ha='center', va='top',  
                bbox=dict(boxstyle='round,pad=0.5', fc='yellow', alpha=0.3)) # 给注释添加一个半透明的黄色背景框。  
  
# 设置标题和标签  
    ax.set_title(f'{feature}', fontsize=14, fontweight='bold') # 设置子图标题。  
    ax.set_xlabel('') # 移除x轴标签。  
    ax.set_xticklabels(['No Diabetes', 'Diabetes']) # 设置x轴刻度标签。  
  
plt.suptitle('Statistical Comparison of Top Features', fontsize=18, fontweight='bold') # 设置总标题。  
plt.tight_layout() # 自动调整布局。  
plt.subplots_adjust(top=0.9) # 调整子图的顶部边距，为总标题留出空间。  
plt.savefig('statistical_comparison.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 7. 相关性矩阵 - 添加显著性标记  
print("\n🔍 特征相关性显著性矩阵:") # 打印子标题。  
# 计算相关性矩阵  
corr_matrix = data[numeric_features + ['Diagnosis']].corr() # 再次计算相关性矩阵。  
  
# 计算p值矩阵  
p_matrix = pd.DataFrame(np.zeros_like(corr_matrix), # 创建一个与相关性矩阵同样大小的全零DataFrame，用于存储p值。  
                        index=corr_matrix.index,  
                        columns=corr_matrix.columns)  
  
# 填充p值矩阵  
for i, col1 inenumerate(p_matrix.columns): # 遍历所有列。  
for j, col2 inenumerate(p_matrix.columns): # 再次遍历所有列，形成列对。  
if i > j:  # 只计算下三角部分，避免重复计算。  
            corr, p = stats.pearsonr(data[col1], data[col2]) # 计算两个特征之间的皮尔逊相关系数和对应的p值。  
            p_matrix.loc[col1, col2] = p # 将p值存储在p值矩阵中。  
            p_matrix.loc[col2, col1] = p # 同样填充对称位置。  
  
# 生成带有显著性标记的相关性热图  
plt.figure(figsize=(14, 12)) # 创建新画布。  
mask = np.triu(np.ones_like(corr_matrix, dtype=bool)) # 创建上三角掩码。  
heatmap = sns.heatmap(corr_matrix, mask=mask, annot=True, fmt='.3f', cmap='RdBu_r', # 绘制热力图，annot=True显示相关系数值。  
                      square=True, linewidths=0.5, cbar_kws={"shrink": .8})  
  
# 添加显著性标记  
for i, col1 inenumerate(corr_matrix.columns): # 遍历行。  
for j, col2 inenumerate(corr_matrix.columns): # 遍历列。  
if i > j:  # 只处理下三角部分。  
            p = p_matrix.loc[col1, col2] # 获取p值。  
            sig = ""# 初始化显著性标记为空字符串。  
if p < 0.001: # 如果p值小于0.001。  
                sig = "***"# 标记为三颗星。  
elif p < 0.01: # 如果p值小于0.01。  
                sig = "**"# 标记为两颗星。  
elif p < 0.05: # 如果p值小于0.05。  
                sig = "*"# 标记为一颗星。  
  
if sig:  # 如果标记不为空（即统计上显著）。  
                heatmap.text(j + 0.25, i + 0.75, sig, ha='center', va='center', color='black', fontweight='bold') # 在热力图的对应单元格中添加星号标记。  
  
plt.title('Feature Correlation Heatmap with Significance Markers', fontsize=16, fontweight='bold', pad=20) # 设置标题。  
plt.xticks(rotation=45, ha='right') # 旋转x轴标签。  
plt.yticks(rotation=0) # 保持y轴标签不旋转。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('correlation_significance_heatmap.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 8. 总结统计分析结果  
print("\n📋 统计分析总结:") # 打印子标题。  
# 组合所有统计结果  
stats_summary = pd.DataFrame(index=numeric_features) # 创建一个以特征为索引的空DataFrame。  
stats_summary['t-statistic'] = t_test_df['t-statistic'] # 添加t统计量列。  
stats_summary['p-value'] = t_test_df['p-value'] # 添加原始p值列。  
stats_summary['FDR p-value'] = t_test_df['FDR p-value'] # 添加FDR校正后的p值列。  
stats_summary['Effect Size (Cohen\'s d)'] = effect_size_df['Cohen\'s D'] # 添加效应量列。  
  
# 添加显著性判断  
stats_summary['Significant'] = stats_summary['FDR p-value'] < 0.05# 根据FDR校正后的p值，添加一个布尔列来表示是否显著。  
  
# 按显著性和效应量排序  
stats_summary = stats_summary.sort_values(by=['Significant', 'Effect Size (Cohen\'s d)'], # 首先按显著性（True在前），然后按效应量大小（降序）进行排序。  
                                          ascending=[False, False])  
  
print(stats_summary) # 打印最终的统计分析摘要表。  
  
# 结果导出  
stats_summary.to_csv('statistical_analysis_results.csv') # 将这个摘要表保存为CSV文件。  
print("\n✅ 统计分析结果已导出到 'statistical_analysis_results.csv'") # 打印确认信息。
```

### **第二部分：13种机器学习模型的构建、优化、评估和比较**

---

### **代码讲解总览**

这部分代码是典型的机器学习项目流程，其目标是：

1. **数据准备**

   将数据集分割成训练集、验证集和测试集，并对特征进行标准化处理。
2. **模型选择**

   准备一个包含多种不同类型分类算法的“模型库”。
3. **超参数优化**

   使用贝叶斯优化为每个模型自动寻找最佳的超参数组合。
4. **模型训练与评估**

   在训练集上训练优化后的模型，并在验证集上评估其性能，比较所有模型的优劣。
5. **结果可视化**

   通过多种图表（如ROC曲线、混淆矩阵、校准曲线等）直观地展示和比较模型性能。
6. **最终测试**

   选出在验证集上表现最好的模型，在从未见过的测试集上进行最终评估，以模拟其在真实世界中的表现。

---

### **阶段一：环境准备与模型导入**

**阶段说明：** 这是模型构建的准备阶段。我们从`scikit-learn`和其他强大的机器学习库（如`XGBoost`, `LightGBM`）中导入所有计划使用的分类模型。同时，我们还导入了数据处理、模型评估和超参数优化所需的所有工具。

* **模型导入**

  涵盖了线性模型、支持向量机、K近邻、决策树、朴素贝叶斯、集成模型（随机森林、梯度提升等）以及判别分析等多种算法，目的是进行广泛的比较。
* **工具导入**

+ `train_test_split`

  用于分割数据集。
+ `StandardScaler`

  用于数据标准化。
+ `metrics`

  包含计算准确率、AUC、混淆矩阵等所有评估指标的函数。
+ `BayesSearchCV`

  来自`scikit-optimize`库，是一个比网格搜索更高效的超参数优化工具。

**代码及逐行注释：**

python

```
from sklearn.model_selection import train_test_split # 从scikit-learn导入用于分割数据集的函数。  
from sklearn.preprocessing import StandardScaler # 导入用于特征标准化的类。  
from sklearn.metrics import * # 导入scikit-learn.metrics模块下的所有函数，如accuracy_score, roc_auc_score, confusion_matrix等。  
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, ExtraTreesClassifier, \  
    AdaBoostClassifier # 导入多种集成学习模型：随机森林、梯度提升、极端随机树、AdaBoost。  
from sklearn.linear_model import LogisticRegression # 导入逻辑回归模型。  
from sklearn.svm import SVC # 导入支持向量机分类器。  
from sklearn.neighbors import KNeighborsClassifier # 导入K近邻分类器。  
from sklearn.tree import DecisionTreeClassifier # 导入决策树分类器。  
from sklearn.naive_bayes import GaussianNB # 导入高斯朴素贝叶斯分类器。  
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis, QuadraticDiscriminantAnalysis # 导入线性和二次判别分析。  
from xgboost import XGBClassifier # 导入XGBoost分类器，一个非常高效和强大的梯度提升实现。  
from lightgbm import LGBMClassifier # 导入LightGBM分类器，另一个高效的梯度提升实现，尤其擅长处理大规模数据。  
from sklearn.calibration import calibration_curve, CalibratedClassifierCV # 导入用于模型校准的函数和类。  
from skopt import BayesSearchCV # 从scikit-optimize库导入贝叶斯优化搜索，用于高效地寻找最佳超参数。  
from skopt.space import Real, Integer # 从skopt.space导入定义搜索空间的类（实数和整数）。  
import seaborn as sns # 再次导入seaborn，确保在当前环境中可用。  
import matplotlib.pyplot as plt # 再次导入matplotlib.pyplot，确保在当前环境中可用。  
from numpy import interp # 从numpy导入interp函数，用于一维线性插值。  
from sklearn.metrics import auc as sklearn_auc # 从sklearn.metrics导入auc函数并重命名为sklearn_auc，以避免与自定义变量冲突。  
import warnings # 再次导入warnings模块。  
  
warnings.filterwarnings('ignore') # 再次设置忽略警告，确保后续代码块不受影响。
```

---

### **阶段二：数据分割与标准化**

**阶段说明：** 这是模型训练前最关键的准备工作。正确的做法是**将数据分为三个独立的部分**：

* **训练集 (Training Set)**

  用于训练模型，让模型学习数据中的模式。
* **验证集 (Validation Set)**

  用于在训练过程中调整超参数和选择最佳模型。模型的好坏是通过在验证集上的表现来判断的，这避免了使用测试集进行调优，从而防止“数据泄露”。
* **测试集 (Test Set)**

  完全独立的数据，只在所有模型训练和选择完成后使用一次，用于评估最终选定模型的泛化能力（即在全新数据上的表现）。

数据分割后，我们进行**特征标准化**。因为很多模型（如逻辑回归、SVM）对特征的尺度非常敏感。标准化（`StandardScaler`）将所有特征都转换成均值为0、标准差为1的分布，确保所有特征在同等重要的尺度上参与模型训练。**重要**：标准化器必须在训练集上`fit`（学习均值和标准差），然后用这个学习到的参数去`transform`训练集、验证集和测试集。

**代码及逐行注释：**

python

```
# ==================== 第三部分：数据分割与标准化 ====================  
print("\n⚙️ 3. 数据分割与标准化") # 打印第三部分的标题。  
print("-" * 50) # 打印分隔线。  
  
# 分离特征和目标变量  
X = data.drop('Diagnosis', axis=1) # 创建特征集X，丢弃'Diagnosis'列。axis=1表示按列操作。  
y = data['Diagnosis'] # 创建目标变量y，只包含'Diagnosis'列。  
  
print(f"特征数量: {X.shape[1]}") # 打印特征的数量。  
print(f"样本数量: {X.shape[0]}") # 打印样本的总数。  
print(f"目标变量分布: {y.value_counts().to_dict()}") # 打印目标变量的分布情况。  
  
# 首先分割出测试集(10%)  
# 核心代码：第一次分割，将整个数据集划分为 90% 的临时集和 10% 的测试集。  
X_temp, X_test, y_temp, y_test = train_test_split(  
    X, y, test_size=0.1, random_state=42, stratify=y  
) # stratify=y确保在分割出的数据集中，目标变量y的类别比例与原始数据集保持一致，这对于不平衡数据集至关重要。  
  
# 然后从剩余数据中分割训练集(70%)和验证集(20%)  
# 核心代码：第二次分割，将 90% 的临时集划分为训练集和验证集。  
X_train, X_val, y_train, y_val = train_test_split(  
    X_temp, y_temp, test_size=0.2222, random_state=42, stratify=y_temp # test_size=0.2222是因为0.2 / 0.9 ≈ 0.2222，这样验证集占原始数据的20%。  
)  
  
print(f"\n数据集分割结果:") # 打印分割结果的摘要。  
print(f"训练集: {X_train.shape[0]} 样本 ({X_train.shape[0] / len(X) * 100:.1f}%)") # 显示训练集的大小和百分比。  
print(f"验证集: {X_val.shape[0]} 样本 ({X_val.shape[0] / len(X) * 100:.1f}%)") # 显示验证集的大小和百分比。  
print(f"测试集: {X_test.shape[0]} 样本 ({X_test.shape[0] / len(X) * 100:.1f}%)") # 显示测试集的大小和百分比。  
  
# 标准化特征  
# 核心代码：创建并应用标准化器。  
scaler = StandardScaler() # 创建一个StandardScaler对象。  
X_train_scaled = scaler.fit_transform(X_train) # 在训练数据上学习（fit）均值和标准差，并对其进行转换（transform）。  
X_val_scaled = scaler.transform(X_val) # 使用训练集学习到的参数来转换验证集数据。  
X_test_scaled = scaler.transform(X_test) # 使用训练集学习到的参数来转换测试集数据。  
  
print(f"\n✅ 特征标准化完成") # 打印完成信息。  
print(f"标准化后训练集特征均值: {X_train_scaled.mean():.6f}") # 检查标准化后训练集的均值，应该非常接近0。  
print(f"标准化后训练集特征标准差: {X_train_scaled.std():.6f}") # 检查标准化后训练集的标准差，应该非常接近1。
```

---

### **阶段三：模型训练、评估与比较**

**阶段说明：** 这是整个流程的核心部分。我们系统地对之前定义的多个模型进行训练和评估。

1. **定义评估函数**

   创建一个`calculate_metrics`函数，用于一次性计算多个关键性能指标（如

   Sensitivity、Specificity、Accuracy、PPV、NPV、F1、Youden's index、AUC），方便后续调用。
2. **定义模型配置**

   创建一个`models_config`字典。它将每个模型的实例、以及用于贝叶斯优化的超参数搜索空间清晰地组织在一起。这使得代码非常模块化和可扩展。
3. **循环训练与优化**

* 遍历`models_config`字典中的每一个模型。
* 使用`BayesSearchCV`对每个模型进行**超参数优化**。它会在定义的参数空间（`params`）中智能地搜索，以找到能使`roc_auc`得分在3折交叉验证中最高的参数组合。
* 用找到的最佳参数训练模型（`best_estimator_`）。
* 在**验证集**上进行预测，并使用`calculate_metrics`函数计算性能。
* 将结果存储起来，以便后续比较。

**代码及逐行注释：**

python

```
# ==================== 第四部分：模型训练、评估与比较 ====================  
print("\n🤖 4. 模型训练、评估与比较") # 打印第四部分的标题。  
print("-" * 50) # 打印分隔线。  
  
# 定义性能指标计算函数  
defcalculate_metrics(y_true, y_pred, y_proba): # 定义一个函数，输入真实标签、预测标签和预测概率。  
"""计算所有性能指标"""  
    tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel() # 计算混淆矩阵并将其扁平化，得到TN, FP, FN, TP。  
  
    metrics = { # 创建一个字典来存储所有指标。  
'Sensitivity': tp / (tp + fn) if (tp + fn) > 0else0,  # 敏感性/召回率: TP / (TP + FN)  
'Specificity': tn / (tn + fp) if (tn + fp) > 0else0,  # 特异性: TN / (TN + FP)  
'Accuracy': (tp + tn) / (tp + tn + fp + fn),  # 准确率  
'PPV': tp / (tp + fp) if (tp + fp) > 0else0,  # 阳性预测值/精确率: TP / (TP + FP)  
'NPV': tn / (tn + fn) if (tn + fn) > 0else0,  # 阴性预测值: TN / (TN + FN)  
'F1': 2 * tp / (2 * tp + fp + fn) if (2 * tp + fp + fn) > 0else0,  # F1分数，精确率和召回率的调和平均值。  
"Youden's index": (tp / (tp + fn) + tn / (tn + fp) - 1) if (tp + fn) > 0and (tn + fp) > 0else0,  # 约登指数: 敏感性 + 特异性 - 1  
'AUC': roc_auc_score(y_true, y_proba) iflen(np.unique(y_true)) > 1else0# AUC值，基于预测概率计算。  
    }  
return metrics # 返回包含所有指标的字典。  
  
# 定义模型字典和贝叶斯优化参数空间  
# 核心代码：这是一个配置字典，定义了所有要测试的模型及其超参数搜索范围。  
models_config = {  
'Logistic Regression': {  
'model': LogisticRegression(random_state=42), # 模型实例  
'params': { # 超参数搜索空间  
'C': Real(0.01, 100, prior='log-uniform'), # 正则化强度的倒数，对数均匀分布  
'max_iter': Integer(100, 1000) # 最大迭代次数，整数范围  
        }  
    },  
'Random Forest': {  
'model': RandomForestClassifier(random_state=42),  
'params': {  
'n_estimators': Integer(50, 200), # 树的数量  
'max_depth': Integer(3, 20), # 树的最大深度  
'min_samples_split': Integer(2, 20), # 节点分裂所需的最小样本数  
'min_samples_leaf': Integer(1, 10) # 叶节点所需的最小样本数  
        }  
    },  
# ... 其他模型的配置与此类似，为每个模型定义了其关键的超参数和搜索范围 ...  
'Gradient Boosting': {  
'model': GradientBoostingClassifier(random_state=42),  
'params': {  
'n_estimators': Integer(50, 200),  
'max_depth': Integer(3, 10),  
'learning_rate': Real(0.01, 0.3, prior='log-uniform')  
        }  
    },  
'XGBoost': {  
'model': XGBClassifier(random_state=42, eval_metric='logloss'),  
'params': {  
'n_estimators': Integer(50, 200),  
'max_depth': Integer(3, 10),  
'learning_rate': Real(0.01, 0.3, prior='log-uniform'),  
'subsample': Real(0.6, 1.0)  
        }  
    },  
'LightGBM': {  
'model': LGBMClassifier(random_state=42, verbosity=-1),  
'params': {  
'n_estimators': Integer(50, 200),  
'max_depth': Integer(3, 10),  
'learning_rate': Real(0.01, 0.3, prior='log-uniform'),  
'num_leaves': Integer(10, 100)  
        }  
    },  
'SVM': {  
'model': SVC(random_state=42, probability=True),  
'params': {  
'C': Real(0.1, 100, prior='log-uniform'),  
'gamma': Real(0.001, 1, prior='log-uniform'),  
'kernel': ['rbf', 'linear'] # 核函数是分类变量  
        }  
    },  
'KNN': {  
'model': KNeighborsClassifier(),  
'params': {  
'n_neighbors': Integer(3, 20),  
'weights': ['uniform', 'distance'],  
'metric': ['euclidean', 'manhattan']  
        }  
    },  
'Decision Tree': {  
'model': DecisionTreeClassifier(random_state=42),  
'params': {  
'max_depth': Integer(3, 20),  
'min_samples_split': Integer(2, 20),  
'min_samples_leaf': Integer(1, 10)  
        }  
    },  
'Extra Trees': {  
'model': ExtraTreesClassifier(random_state=42),  
'params': {  
'n_estimators': Integer(50, 200),  
'max_depth': Integer(3, 20),  
'min_samples_split': Integer(2, 20)  
        }  
    },  
'AdaBoost': {  
'model': AdaBoostClassifier(random_state=42),  
'params': {  
'n_estimators': Integer(50, 200),  
'learning_rate': Real(0.01, 2, prior='log-uniform')  
        }  
    },  
'Naive Bayes': {  
'model': GaussianNB(),  
'params': {  
'var_smoothing': Real(1e-10, 1e-6, prior='log-uniform') # 方差平滑  
        }  
    },  
'LDA': {  
'model': LinearDiscriminantAnalysis(),  
'params': {  
'solver': ['lsqr'],  # 只使用支持shrinkage的求解器  
'shrinkage': Real(0.0, 1.0) # 收缩参数  
    }  
},  
'QDA': {  
'model': QuadraticDiscriminantAnalysis(),  
'params': {  
'reg_param': Real(0.0, 1.0) # 正则化参数  
        }  
    }  
}  
  
# 存储结果  
results = {} # 存储每个模型在验证集上的性能指标。  
best_models = {} # 存储每个模型优化后的最佳实例。  
  
print("开始贝叶斯优化和模型训练...")  
  
# 核心代码：遍历所有模型配置，进行训练和优化。  
for name, config in models_config.items():  
print(f"\n🔄 训练 {name}...") # 打印当前正在训练的模型名称。  
  
try:  
# 贝叶斯优化  
        bayes_search = BayesSearchCV(  
            config['model'], # 待优化的模型。  
            config['params'], # 超参数搜索空间。  
            n_iter=30,  # 贝叶斯优化迭代次数，即尝试30组不同的超参数。  
            cv=3,  # 交叉验证折数，每次迭代时在训练集内部进行3折交叉验证。  
            scoring='roc_auc', # 优化的目标指标是AUC。  
            random_state=42, # 随机种子，保证结果可复现。  
            n_jobs=-1# 使用所有可用的CPU核心进行并行计算，加快速度。  
        )  
  
        bayes_search.fit(X_train_scaled, y_train) # 在标准化的训练集上执行贝叶斯搜索。  
        best_model = bayes_search.best_estimator_ # 获取优化后找到的最佳模型。  
  
# 验证集预测  
        y_val_pred = best_model.predict(X_val_scaled) # 使用最佳模型对验证集进行类别预测。  
        y_val_proba = best_model.predict_proba(X_val_scaled)[:, 1] # 获取验证集上属于正类（类别1）的预测概率。  
  
# 计算指标  
        metrics = calculate_metrics(y_val, y_val_pred, y_val_proba) # 调用函数计算所有性能指标。  
  
        results[name] = metrics # 将指标字典存入results。  
        best_models[name] = best_model # 将最佳模型实例存入best_models。  
  
print(f"✅ {name} 训练完成 - AUC: {metrics['AUC']:.4f}") # 打印训练完成信息和验证集上的AUC。  
  
except Exception as e: # 捕获可能发生的任何错误。  
print(f"❌ {name} 训练失败: {str(e)}") # 如果训练失败，打印错误信息并继续下一个模型。  
continue# 跳过当前循环的剩余部分。  
  
# ==================== 结果展示 ====================  
print("\n📊 模型性能结果:") # 打印结果部分的标题。  
print("-" * 50) # 打印分隔线。  
  
# 创建结果DataFrame  
results_df = pd.DataFrame(results).T # 将存储结果的字典转换为Pandas DataFrame，并转置，使模型名称成为行索引。  
results_df = results_df.round(4) # 将所有数值四舍五入到小数点后4位。  
results_df = results_df.sort_values('AUC', ascending=False) # 按AUC值降序排列，性能最好的模型在最上面。  
  
print(results_df) # 打印最终的性能比较表。
```

---

### **阶段四：性能可视化**

**阶段说明：** 数字表格虽然精确，但不够直观。这个阶段的目标是将模型的性能以图形化的方式展示出来，以便快速比较和深入分析。我们生成了多种关键的可视化图表：

* **性能折线图**

  将Sensitivity、Specificity、Accuracy、PPV、NPV、F1、Youden's index、AUC预测值分别绘制成图，直观展示模型在不同评估维度上的排名。
* **ROC曲线图**

  所有模型的ROC曲线绘制在同一张图上，是评估二分类模型性能最核心的图表。曲线越靠近左上角，AUC值越大，模型性能越好。
* **AUC森林图**

  医学研究中的森林图，清晰地展示了每个模型的AUC值及其置信区间，可以直观地比较模型性能的优劣和稳定性。
* **混淆矩阵图**

  为每个模型绘制混淆矩阵，详细展示了模型在每个类别上的具体预测情况（真阳性、假阳性、真阴性、假阴性）。
* **临床影响曲线**

  一种更贴近实际应用的评估方法，它展示了在不同决策阈值下，模型的“净收益”（正确诊断的病人 - 错误诊断的健康人）。
* **校准曲线**

  评估模型预测概率的可靠性。一个好的模型，其预测概率应该与真实的阳性比例相符。

**代码及逐行注释：**

python

```
# ==================== 可视化部分 ====================  
print("\n🎨 开始生成可视化图表...") # 打印开始信息。  
  
# 1. 性能折线图  
plt.figure(figsize=(15, 10)) # 创建一个大的画布。  
metrics_to_plot = ['Sensitivity', 'Specificity', 'Accuracy', 'PPV', 'NPV', 'F1', "Youden's index"] # 定义要绘制的指标列表。  
  
for i, metric inenumerate(metrics_to_plot, 1): # 遍历指标列表。  
    plt.subplot(3, 3, i) # 在3x3的网格中创建子图。  
    sorted_results = results_df.sort_values(metric, ascending=False) # 对当前指标进行降序排序。  
    plt.plot(range(len(sorted_results)), sorted_results[metric], 'o-', linewidth=2, markersize=6) # 绘制折线图，x轴是模型排名，y轴是指标值。  
    plt.title(f'{metric} Performance', fontweight='bold') # 设置子图标题。  
    plt.xlabel('Model Rank') # 设置x轴标签。  
    plt.ylabel(metric) # 设置y轴标签。  
    plt.xticks(range(len(sorted_results)), sorted_results.index, rotation=45) # 设置x轴刻度为模型名称并旋转45度。  
    plt.grid(True, alpha=0.3) # 添加半透明网格。  
  
plt.tight_layout() # 自动调整布局，防止标签重叠。  
plt.savefig('performance_line_plots.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 2. ROC曲线图  
plt.figure(figsize=(12, 10)) # 创建一个新画布。  
  
# 收集所有模型的ROC数据  
roc_data = {} # 创建一个字典来存储每个模型的ROC曲线数据。  
for name, model in best_models.items(): # 遍历所有优化后的模型。  
    y_proba = model.predict_proba(X_val_scaled)[:, 1] # 获取模型在验证集上的预测概率。  
    fpr, tpr, _ = roc_curve(y_val, y_proba) # 计算假正例率(FPR)和真正例率(TPR)。  
    roc_auc = sklearn_auc(fpr, tpr) # 计算ROC曲线下的面积(AUC)。  
    roc_data[name] = {'fpr': fpr, 'tpr': tpr, 'auc': roc_auc} # 存储数据。  
  
# 绘制ROC曲线+置信区间  
for name, data in roc_data.items(): # 遍历每个模型的数据。  
    plt.plot(data['fpr'], data['tpr'], label=f'{name} (AUC = {data["auc"]:.3f})') # 绘制ROC曲线，并在图例中显示AUC值。  
# 核心代码：绘制一个模拟的置信区间，直观表示曲线可能波动的范围。  
    plt.fill_between(data['fpr'], data['tpr'] - 0.05, data['tpr'] + 0.05, alpha=0.1) # 在曲线周围填充一个半透明区域。  
plt.plot([0, 1], [0, 1], 'k--', label='Random Guessing') # 绘制一条对角虚线，代表随机猜测的性能。  
plt.title('ROC Curves for All Models', fontweight='bold') # 设置标题。  
plt.xlabel('False Positive Rate') # 设置x轴标签。  
plt.ylabel('True Positive Rate') # 设置y轴标签。  
plt.legend(loc='lower right') # 在右下角显示图例。  
plt.grid(True, alpha=0.3) # 添加网格。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('roc_curves.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 3. AUC森林图  
plt.figure(figsize=(10, 8)) # 创建新画布。  
auc_values = results_df['AUC'].sort_values(ascending=True) # 获取所有模型的AUC值并升序排列（为了绘图时最好的在顶部）。  
y_pos = np.arange(len(auc_values)) # 创建y轴的位置。  
  
# 计算置信区间 (95% CI) - 这是一个简化的估算，实际应通过bootstrap等方法计算  
ci_lower = auc_values - 1.96 * (auc_values.std() / np.sqrt(len(auc_values))) # 估算置信区间下界。  
ci_upper = auc_values + 1.96 * (auc_values.std() / np.sqrt(len(auc_values))) # 估算置信区间上界。  
# 绘制森林图  
plt.barh(y_pos, auc_values, alpha=0.7, color='skyblue') # 绘制水平条形图作为基础。  
plt.errorbar(auc_values, y_pos, xerr=[auc_values - ci_lower, ci_upper - auc_values],  
             fmt='o', color='red', capsize=5) # 在条形图上叠加误差棒（置信区间）。  
  
plt.yticks(y_pos, auc_values.index) # 设置y轴刻度为模型名称。  
plt.xlabel('AUC Value') # 设置x轴标签。  
plt.title('AUC Forest Plot with Confidence Intervals', fontweight='bold') # 设置标题。  
plt.axvline(x=0.5, color='gray', linestyle='--', alpha=0.7) # 添加一条垂直线在x=0.5处，代表随机猜测水平。  
plt.grid(True, alpha=0.3) # 添加网格。  
  
# 添加数值标签  
for i, v inenumerate(auc_values): # 遍历每个AUC值。  
    plt.text(v + 0.01, i, f'{v:.3f}', va='center', fontweight='bold') # 在误差棒末端添加AUC数值。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('auc_forest_plot.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 4. 混淆矩阵图  
n_models = len(best_models) # 获取模型总数。  
n_cols = 5# 设置每行显示的子图数量。  
n_rows = (n_models + n_cols - 1) // n_cols # 计算所需的行数。  
  
fig, axes = plt.subplots(n_rows, n_cols, figsize=(20, 4 * n_rows)) # 创建子图网格。  
if n_rows == 1: # 处理只有一行的情况。  
    axes = axes.reshape(1, -1)  
axes = axes.flatten() # 将子图数组扁平化，方便遍历。  
  
for i, (name, model) inenumerate(best_models.items()): # 遍历所有最佳模型。  
if i >= len(axes): # 如果模型数量超过子图数量，则停止。  
break  
  
    y_pred = model.predict(X_val_scaled) # 获取预测标签。  
    cm = confusion_matrix(y_val, y_pred) # 计算混淆矩阵。  
# 核心代码：使用seaborn的热力图功能绘制混淆矩阵。  
    sns.heatmap(cm, annot=True, fmt='d', cmap='Blues', ax=axes[i], # annot=True在格内显示数字，fmt='d'表示整数格式。  
                xticklabels=['No Diabetes', 'Diabetes'], # 设置x轴标签。  
                yticklabels=['No Diabetes', 'Diabetes']) # 设置y轴标签。  
    axes[i].set_title(f'{name}', fontweight='bold') # 设置子图标题。  
    axes[i].set_xlabel('Predicted') # 设置x轴标签。  
    axes[i].set_ylabel('Actual') # 设置y轴标签。  
  
# 隐藏多余的子图  
for j inrange(len(best_models), len(axes)): # 遍历未使用的子图。  
    axes[j].set_visible(False) # 将其隐藏。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('confusion_matrices.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# 5. 临床影响曲线  
defplot_clinical_impact_curve(y_true, y_proba, ax=None): # 定义一个绘制临床影响曲线的函数。  
"""绘制临床影响曲线"""  
    thresholds = np.linspace(0, 1, 100) # 创建100个从0到1的阈值。  
    impacts = [] # 存储每个阈值下的临床影响值。  
  
for threshold in thresholds: # 遍历所有阈值。  
        y_pred = (y_proba >= threshold).astype(int) # 根据阈值将预测概率转换为预测类别。  
        tp = np.sum((y_true == 1) & (y_pred == 1)) # 计算真阳性。  
        fp = np.sum((y_true == 0) & (y_pred == 1)) # 计算假阳性。  
        impact = tp - fp # 计算临床影响（净收益）。  
        impacts.append(impact) # 添加到列表中。  
  
if ax isNone: # 如果没有提供子图对象，则创建一个新的。  
        fig, ax = plt.subplots(figsize=(10, 6))  
  
    ax.plot(thresholds, impacts, label='Clinical Impact', color='orange') # 绘制曲线。  
    ax.axhline(0, color='gray', linestyle='--', linewidth=1) # 添加一条水平参考线。  
    ax.set_xlabel('Threshold') # 设置x轴标签。  
    ax.set_ylabel('Clinical Impact (TP - FP)') # 设置y轴标签。  
    ax.set_title('Clinical Impact Curve', fontweight='bold') # 设置标题。  
    ax.legend() # 显示图例。  
    ax.grid(True, alpha=0.3) # 添加网格。  
return ax # 返回子图对象。  
  
# 绘制临床影响曲线  
fig, axes = plt.subplots(n_rows, n_cols, figsize=(20, 4 * n_rows)) # 创建子图网格。  
if n_rows == 1:  
    axes = axes.reshape(1, -1)  
axes = axes.flatten() # 扁平化子图数组。  
for i, (name, model) inenumerate(best_models.items()): # 遍历所有模型。  
if i >= len(axes):  
break  
  
    y_proba = model.predict_proba(X_val_scaled)[:, 1] # 获取预测概率。  
    plot_clinical_impact_curve(y_val, y_proba, ax=axes[i]) # 在对应的子图上绘制临床影响曲线。  
    axes[i].set_title(f'{name}', fontweight='bold') # 设置子图标题为模型名称。  
# 隐藏多余的子图  
for j inrange(len(best_models), len(axes)):  
    axes[j].set_visible(False)  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('clinical_impact_curves.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
  
# 6. 校准曲线  
fig, axes = plt.subplots(n_rows, n_cols, figsize=(20, 4 * n_rows)) # 创建子图网格。  
if n_rows == 1:  
    axes = axes.reshape(1, -1)  
axes = axes.flatten() # 扁平化子图数组。  
  
for i, (name, model) inenumerate(best_models.items()): # 遍历所有模型。  
if i >= len(axes):  
break  
  
    y_proba = model.predict_proba(X_val_scaled)[:, 1] # 获取预测概率。  
  
# 核心代码：计算校准曲线所需的数据点。  
    prob_true, prob_pred = calibration_curve(y_val, y_proba, n_bins=10) # y_val是真实标签，y_proba是预测概率，n_bins是将概率分成10个区间。  
  
    axes[i].plot([0, 1], [0, 1], 'k--', label='Perfect Calibration') # 绘制代表完美校准的对角线。  
    axes[i].plot(prob_pred, prob_true, 'o-', label=name) # 绘制模型的校准曲线。  
    axes[i].set_xlabel('Mean Predicted Probability') # 设置x轴标签：每个区间的平均预测概率。  
    axes[i].set_ylabel('Fraction of Positives') # 设置y轴标签：每个区间的真实正例比例。  
    axes[i].set_title(f'{name} Calibration', fontweight='bold') # 设置标题。  
    axes[i].legend() # 显示图例。  
    axes[i].grid(True, alpha=0.3) # 添加网格。  
  
# 隐藏多余的子图  
for j inrange(len(best_models), len(axes)):  
    axes[j].set_visible(False)  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('calibration_curves.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段五：最终测试集评估**

**阶段说明：** 这是整个模型构建流程的最后一步，也是最重要的一步。

1. **选择最佳模型**

   : 根据在**验证集**上的AUC表现，从所有训练好的模型中选出冠军模型。
2. **最终评估**

   : 使用这个冠军模型对**测试集**进行预测。测试集是模型从未见过的数据，因此在这个数据集上的性能得分，可以作为模型在真实世界中泛化能力的无偏估计。

这一步的结果是衡量我们整个工作成败的最终标准。

**代码及逐行注释：**

python

```
# ==================== 最终测试集评估 ====================  
print("\n🎯 最终测试集评估:") # 打印最终评估部分的标题。  
print("-" * 50) # 打印分隔线。  
  
# 核心代码：选择在验证集上性能最好的模型。  
best_model_name = results_df.index[0] # results_df已经按AUC降序排列，所以第一个模型的名字就是最佳模型的名字。  
best_model = best_models[best_model_name] # 从best_models字典中获取最佳模型的实例。  
  
print(f"最佳模型: {best_model_name} (验证集AUC: {results_df.loc[best_model_name, 'AUC']:.4f})") # 打印最佳模型的名称和它在验证集上的AUC分数。  
  
# 测试集评估  
y_test_pred = best_model.predict(X_test_scaled) # 使用最佳模型对标准化的测试集进行类别预测。  
y_test_proba = best_model.predict_proba(X_test_scaled)[:, 1] # 获取在测试集上的预测概率。  
  
# 计算测试集指标  
test_metrics = calculate_metrics(y_test, y_test_pred, y_test_proba) # 调用函数计算在测试集上的所有性能指标。  
  
print(f"\n测试集性能:") # 打印子标题。  
for metric, value in test_metrics.items(): # 遍历并打印所有测试集指标。  
print(f"  {metric}: {value:.4f}")  
  
print("\n" + "=" * 60) # 打印结束分隔线。  
print("🎉 模型训练和评估完成!") # 打印完成信息。  
print("=" * 60) # 打印结束分隔线。
```

### ****第三部分：LightGBM模型的SHAP高级可视化分析****

**SHAP是什么？** SHAP是一种基于博弈论的方法，用于解释任何机器学习模型的输出。它通过为每个特征分配一个“SHAP值”来解释单次预测。这个SHAP值代表了该特征对本次预测的贡献度——正值表示该特征将预测推向正类（例如“患有糖尿病”），负值则表示推向负类。

这部分代码的目标是：

1. **全局解释**

   理解哪些特征在**整体上**对模型的预测影响最大。
2. **局部解释**

   理解对于**某一个具体的样本**（病人），每个特征是如何共同作用，最终得出其预测结果的。
3. **交互分析**

   探索特征之间是否存在**交互效应**，即一个特征的影响力是否会随着另一个特征值的变化而变化。
4. **可视化**

   将上述所有复杂的解释信息通过一系列直观、清晰的图表展示出来。

我们将针对之前筛选出的最优模型 **LightGBM** 进行SHAP分析，因为它通常性能强大，但其内部决策过程不透明，非常需要SHAP这样的工具来解释。

---

### **阶段一：环境准备与SHAP值计算**

**阶段说明：** 这是进行SHAP分析的准备工作。我们首先导入`shap`库，并从之前训练好的模型库中获取LightGBM模型。然后，我们创建一个`shap.TreeExplainer`对象，这是专门为树模型（如LightGBM, XGBoost, 随机森林）设计的、经过高度优化的解释器。最后，我们计算SHAP值。由于计算所有样本的SHAP值可能非常耗时，我们在这里只选择验证集的一个小子集（最多100个样本）进行计算，这在探索性解释阶段是常见且高效的做法。

**代码及逐行注释：**

python

```
import shap # 导入SHAP库，用于模型解释。  
import matplotlib.pyplot as plt # 导入matplotlib用于绘图。  
import numpy as np # 导入numpy用于数值计算。  
import pandas as pd # 导入pandas用于数据处理。  
import seaborn as sns # 导入seaborn用于高级可视化。  
import warnings # 导入warnings模块。  
  
warnings.filterwarnings('ignore') # 设置忽略所有警告信息。  
  
# ==================== LightGBM SHAP 可视化分析 ====================  
print("\n🔍 LightGBM SHAP 可视化分析") # 打印此部分的标题。  
print("=" * 60) # 打印分隔线。  
  
# 获取LightGBM模型  
if'LightGBM'in best_models: # 检查之前训练的模型字典中是否存在'LightGBM'。  
lightgbm_model = best_models['LightGBM'] # 如果存在，获取这个已经优化好的模型实例。  
print(f"✅ 获取LightGBM模型成功，AUC: {results['LightGBM']['AUC']:.4f}") # 打印成功信息和其在验证集上的AUC分数。  
  
# 初始化SHAP解释器  
print("🔄 初始化SHAP解释器...") # 打印状态信息。  
# 核心代码：为树模型创建一个解释器。它会分析模型的内部结构以便高效计算SHAP值。  
explainer = shap.TreeExplainer(lightgbm_model) # 传入训练好的LightGBM模型。  
  
# 计算SHAP值（使用验证集的子集以提高计算效率）  
sample_size = min(100, len(X_val_scaled))  # 定义样本大小，取100和验证集长度中的较小者。  
X_val_sample = X_val_scaled[:sample_size] # 从标准化的验证集中选取子集作为样本特征。  
y_val_sample = y_val.iloc[:sample_size] # 选取对应的样本标签。  
  
print(f"🔄 计算SHAP值 (使用{sample_size}个样本)...") # 打印状态信息。  
# 核心代码：计算SHAP值。这将为每个样本的每个特征生成一个SHAP值。  
shap_values = explainer.shap_values(X_val_sample) # 传入样本特征数据。  
  
# 如果是二分类，TreeExplainer会返回一个列表，包含两个数组（分别对应负类和正类的SHAP值）。  
ifisinstance(shap_values, list): # 检查shap_values是否是一个列表。  
shap_values = shap_values[1]  # 我们通常只关心对正类（类别1，即“患有糖尿病”）的预测贡献，所以选择第二个数组。  
  
print("✅ SHAP值计算完成，开始生成可视化图表...") # 打印完成信息。  
  
# 获取特征名称  
feature_names = X.columns.tolist() # 从原始DataFrame X中获取特征名称列表，用于图表标签。
```

---

### **阶段二：全局特征重要性图表**

**阶段说明：** 这个阶段我们通过两种图表来了解模型在**全局**层面认为哪些特征最重要。

1. **SHAP Summary Plot (条形图模式)**

   这是最直接的特征重要性排序图。它计算每个特征SHAP值的**平均绝对值**，然后按此值对特征进行排序。条形越长，代表该特征对模型预测的平均影响越大。
2. **SHAP Bar Plot (自定义实现)**

   这段代码手动实现了与上面类似的功能，通过计算平均绝对SHAP值并绘制水平条形图，再次确认全局特征的重要性。

**代码及逐行注释：**

python

```
# ==================== 1. SHAP Summary Plot ====================  
print("\n📊 1. 生成SHAP Summary Plot...") # 打印状态信息。  
plt.figure(figsize=(12, 8)) # 创建一个12x8英寸的画布。  
# 核心代码：生成SHAP摘要图。plot_type="bar"指定了条形图模式。  
shap.summary_plot(shap_values, X_val_sample, feature_names=feature_names, plot_type="bar", show=False)  
plt.title('LightGBM SHAP Summary Plot', fontsize=16, fontweight='bold', pad=20) # 设置图表标题。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_summary.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# ==================== 2. SHAP Bar Plot (特征重要性) ====================  
print("\n📊 2. 生成SHAP Bar Plot...") # 打印状态信息。  
shap_importance = np.abs(shap_values).mean(0) # 手动计算每个特征SHAP值的平均绝对值。mean(0)表示按列（特征）计算均值。  
  
plt.figure(figsize=(10, 8)) # 创建一个10x8英寸的画布。  
# 使用兼容性更好的方式创建条形图  
features_sorted = np.argsort(shap_importance) # 获取按重要性排序后的特征索引（从小到大）。  
plt.barh( # 绘制水平条形图。  
[feature_names[i] for i in features_sorted], # y轴是排序后的特征名称。  
shap_importance[features_sorted], # x轴是对应的特征重要性值。  
color='skyblue'# 设置条形颜色。  
)  
plt.title('LightGBM SHAP Feature Importance', fontsize=16, fontweight='bold') # 设置标题。  
plt.xlabel('Mean |SHAP Value|') # 设置x轴标签，表示平均绝对SHAP值。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_bar.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段三：详细特征影响图 (Beeswarm Plot)**

**阶段说明：** 这是Summary Plot的增强版，信息量更丰富。它不仅显示了特征的重要性，还揭示了**特征值的大小**如何影响预测方向。

**如何解读Beeswarm图：**

* **Y轴**

  特征按重要性从上到下排序。
* **X轴**

  SHAP值。点在0的右边，表示该特征使预测概率**增加**；在左边则表示**减少**。
* **每个点**

  代表一个样本。
* **点的颜色**

  代表该样本上该特征的**原始值**。通常红色代表特征值高，蓝色代表特征值低。

**代码及逐行注释：**

python

```
# ==================== 3. SHAP Beeswarm Plot ====================  
print("\n📊 3. 生成 SHAP Beeswarm Plot...") # 打印状态信息。  
plt.figure(figsize=(12, 10)) # 创建画布。  
# 核心代码：生成Beeswarm图。需要将数据封装在shap.Explanation对象中。  
shap.plots.beeswarm(shap.Explanation(values=shap_values,  
base_values=np.zeros(len(shap_values)), # 基线值，通常为0。  
data=X_val_sample, # 原始特征值，用于着色。  
feature_names=feature_names), # 特征名称。  
max_display=20, # 最多显示20个最重要的特征。  
show=False) # 不立即显示，以便我们可以自定义标题等。  
plt.title('LightGBM SHAP Beeswarm Plot', fontsize=16, fontweight='bold', pad=20) # 设置标题。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_beeswarm.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段四：单样本预测解释图 (Waterfall & Force Plot)**

**阶段说明：** 这个阶段我们从全局视角转向**局部视角**，解释模型对**单个样本**的预测过程。

1. **Waterfall Plot (瀑布图)**

* 它清晰地展示了对于一个样本，从模型的**基线值**（`E[f(x)]`，即所有样本的平均预测值）开始，每个特征是如何一步步将预测值“推”向最终结果的。
* 红色的条代表增加预测概率的特征，蓝色的条代表减少预测概率的特征。所有条的贡献加起来，就从基线值到达了最终的预测值`f(x)`。

1. **Force Plot (力图)**

* 它提供了与瀑布图类似的信息，但形式更紧凑。它像一个“力学”系统，红色特征（推高预测的力）和蓝色特征（拉低预测的力）相互抗衡，最终将预测结果稳定在某个点上。
* 这对于快速理解单个样本的关键驱动因素非常有效。

**代码及逐行注释：**

python

```
# ==================== 4. SHAP Waterfall Plot (Single Sample Explanation) ====================  
print("\n📊 4. 生成 SHAP Waterfall Plot...") # 打印状态信息。  
  
# 选择一些有代表性的样本进行解释  
sample_indices = [0, 1]  # 选择前两个样本进行解释。  
for i, idx inenumerate(sample_indices): # 遍历选定的样本索引。  
plt.figure(figsize=(10, 8)) # 为每个样本创建新画布。  
  
# 创建SHAP解释对象  
exp = shap.Explanation(values=shap_values[idx:idx + 1, :], # 当前样本的SHAP值。  
base_values=np.zeros(1), # 基线值。  
data=X_val_sample[idx:idx + 1, :], # 当前样本的特征值。  
feature_names=feature_names) # 特征名。  
  
# 绘制瀑布图  
# 核心代码：为单个样本的解释对象绘制瀑布图。  
shap.plots.waterfall(exp[0], max_display=10, show=False) # exp[0]获取第一个（也是唯一一个）样本的解释。  
  
# 添加标题和样本信息  
actual_class = "Diabetes"if y_val_sample.iloc[idx] == 1else"No Diabetes"# 获取真实标签。  
pred_prob = lightgbm_model.predict_proba(X_val_sample[idx:idx + 1, :])[0, 1] # 获取模型对该样本的预测概率。  
pred_class = "Diabetes"if pred_prob > 0.5else"No Diabetes"# 获取预测类别。  
  
plt.title(f'Sample #{idx + 1} SHAP Waterfall Plot\n'# 设置包含详细信息的标题。  
f'Actual Class: {actual_class}, Predicted Probability: {pred_prob:.4f}, Predicted Class: {pred_class}',  
fontsize=14, fontweight='bold', pad=20)  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig(f'lightgbm_shap_waterfall_sample_{idx + 1}.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# ==================== 5. SHAP Dependence Plot ====================  
# ... (此部分在下一阶段讲解) ...  
  
# ==================== 6. SHAP Force Plot ====================  
print("\n📊 6. 生成SHAP Force Plot...") # 打印状态信息。  
  
# 为前5个样本生成Force Plot  
for i inrange(min(5, len(shap_values))): # 遍历前5个样本。  
plt.figure(figsize=(20, 3)) # Force Plot通常是水平的，所以画布较宽。  
  
# 获取base value (模型的平均预测输出)  
base_value = explainer.expected_value # 从解释器获取基线值。  
ifisinstance(base_value, list): # 如果是列表（二分类情况），取正类的基线值。  
base_value = base_value[1]  
  
# 核心代码：为单个样本绘制力图。  
shap.force_plot(  
base_value, # 传入基线值。  
np.round(shap_values[i], 2), # 传入当前样本的SHAP值（保留两位小数）。  
X_val_sample[i], # 传入当前样本的特征值。  
feature_names=feature_names, # 传入特征名。  
matplotlib=True, # 使用matplotlib后端进行绘图，以便保存。  
show=False# 不立即显示。  
)  
  
actual_label = "Diabetes"if y_val_sample.iloc[i] == 1else"No Diabetes"# 获取真实标签。  
pred_proba = lightgbm_model.predict_proba(X_val_sample[i].reshape(1, -1))[0, 1] # 获取预测概率。  
plt.title(f'SHAP Force Plot - Sample {i + 1} | Actual: {actual_label} | Pred Prob: {pred_proba:.3f}',  
fontweight='bold', fontsize=14, pad=20) # 设置标题。  
plt.tight_layout() # 自动调整布局。  
plt.savefig(f'lightgbm_shap_force_sample_{i + 1}.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段五：特征依赖与交互分析**

**阶段说明：** 这个阶段我们探索更复杂的关系。

1. **Dependence Plot (依赖图)**

* 它展示了一个特征的值如何影响其自身的SHAP值。X轴是特征值，Y轴是对应的SHAP值。
* 这能帮我们看到非线性关系。例如，一个特征可能在值较低时影响不大（SHAP值接近0），但在值超过某个阈值后，其SHAP值迅速增加。
* 图中的点的颜色通常由另一个与之交互最强的特征决定，这可以帮助我们发现**交互效应**。

1. **Interaction Analysis (交互分析)**

* 这是一种更直接的交互效应分析。它计算**SHAP交互值**，这个值量化了两个特征组合在一起时产生的额外效应。
* 我们将结果用热力图展示，图中数值越大，代表两个特征之间的交互作用越强。

**代码及逐行注释：**

python

```
# ==================== 5. SHAP Dependence Plot ====================  
print("\n📊 5. 生成SHAP Dependence Plots...") # 打印状态信息。  
  
# 选择最重要的4个特征进行依赖图分析  
top_features_indices = np.argsort(-np.abs(shap_values).mean(0))[:4] # 获取最重要的4个特征的索引。  
top_features = [feature_names[i] for i in top_features_indices] # 获取对应的特征名称。  
  
fig, axes = plt.subplots(2, 2, figsize=(15, 12)) # 创建2x2的子图网格。  
axes = axes.flatten() # 扁平化子图数组。  
  
for i, feature_idx inenumerate(top_features_indices): # 遍历这4个特征。  
plt.sca(axes[i]) # 激活当前的子图。  
# 核心代码：绘制依赖图。  
shap.dependence_plot(  
feature_idx, # 要绘制的主特征的索引。  
shap_values, # 所有样本的SHAP值。  
X_val_sample, # 所有样本的特征值。  
feature_names=feature_names, # 特征名列表。  
ax=axes[i], # 指定在哪个子图上绘制。  
show=False# 不立即显示。  
)  
axes[i].set_title(f'SHAP Dependence: {feature_names[feature_idx]}',  
fontweight='bold', fontsize=12) # 设置子图标题。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_dependence.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# ... (Force Plot代码已在上一阶段讲解) ...  
  
# ==================== 7. SHAP Decision Plot ====================  
# ... (此部分在下一阶段讲解) ...  
  
# ==================== 8. SHAP Interaction Values ====================  
print("\n📊 8. 生成SHAP Interaction Analysis...") # 打印状态信息。  
  
# 计算SHAP交互值（使用更小的样本以提高计算效率）  
interaction_sample_size = min(50, len(X_val_sample)) # 选择更小的样本量，因为交互值计算量巨大。  
X_interaction_sample = X_val_sample[:interaction_sample_size] # 获取样本。  
  
print(f"🔄 计算SHAP交互值 (使用{interaction_sample_size}个样本)...") # 打印状态信息。  
try:  
# 核心代码：计算SHAP交互值。  
shap_interaction_values = explainer.shap_interaction_values(X_interaction_sample) # 传入样本数据。  
  
# 如果是二分类，取正类的交互值  
ifisinstance(shap_interaction_values, list): # 检查返回类型。  
shap_interaction_values = shap_interaction_values[1] # 取正类的交互值。  
  
# 交互作用热力图  
plt.figure(figsize=(12, 10)) # 创建画布。  
interaction_matrix = np.abs(shap_interaction_values).mean(0) # 计算平均绝对交互值矩阵。  
  
# 只显示上三角矩阵（避免重复）  
mask = np.triu(np.ones_like(interaction_matrix, dtype=bool), k=1) # 创建一个上三角掩码。  
interaction_matrix_masked = np.where(mask, interaction_matrix, 0) # 应用掩码。  
  
sns.heatmap( # 使用seaborn绘制热力图。  
interaction_matrix_masked,  
xticklabels=feature_names, # 设置x轴标签。  
yticklabels=feature_names, # 设置y轴标签。  
annot=True, # 在格内显示数值。  
fmt='.3f', # 数值格式为3位小数。  
cmap='RdYlBu_r', # 设置颜色映射。  
square=True, # 设置单元格为正方形。  
linewidths=0.5# 设置网格线。  
)  
plt.title('LightGBM SHAP Feature Interaction Matrix', fontsize=16, fontweight='bold') # 设置标题。  
plt.xticks(rotation=45, ha='right') # 旋转x轴标签。  
plt.yticks(rotation=0) # 保持y轴标签不旋转。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_interaction_matrix.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
except Exception as e: # 捕获可能的错误。  
print(f"⚠️ 计算SHAP交互值失败: {str(e)}") # 打印错误信息。  
print("交互分析需要较大的计算资源，可以减少样本数量或跳过此步骤") # 提供建议。
```

---

### **阶段六：多样本聚合解释图 (Decision & Heatmap)**

**阶段说明：** 这个阶段我们通过两种图表来同时可视化**多个样本**的预测过程。

1. **Decision Plot (决策图)**

* 它将多个样本的瀑布图堆叠在一起。每一条线代表一个样本的预测路径，从基线值开始，随着特征的加入，线的位置不断变化，最终到达该样本的最终预测值。
* 这对于观察不同样本（例如，被正确预测和被错误预测的样本）的决策路径差异非常有用。

1. **SHAP Heatmap (热力图)**

* 它将每个样本（X轴）和每个特征（Y轴）的SHAP值用一个矩阵表示，并用颜色深浅来表示SHAP值的大小。
* 这可以帮助我们发现样本的聚类模式。例如，可能有一组病人，他们的`Glucose`特征都有很高的正SHAP值，这说明他们可能是因为高血糖而被预测为患病。

**代码及逐行注释：**

python

```
# ==================== 7. SHAP Decision Plot ====================  
print("\n📊 7. 生成SHAP Decision Plot...") # 打印状态信息。  
  
try:  
plt.figure(figsize=(12, 8)) # 创建画布。  
# 选择前20个样本进行决策路径展示  
sample_indices = range(min(20, len(shap_values))) # 选择要展示的样本索引。  
  
# 获取base value  
base_value = explainer.expected_value # 获取基线值。  
ifisinstance(base_value, list):  
base_value = base_value[1]  
  
# 核心代码：绘制决策图。  
shap.decision_plot(  
base_value, # 传入基线值。  
shap_values[sample_indices], # 传入多个样本的SHAP值。  
X_val_sample[sample_indices], # 传入对应的特征值。  
feature_names=feature_names, # 传入特征名。  
show=False# 不立即显示。  
)  
plt.title('LightGBM SHAP Decision Plot', fontsize=16, fontweight='bold') # 设置标题。  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_decision.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
except Exception as e: # 捕获错误。  
print(f"⚠️ 决策图绘制失败: {str(e)}") # 打印错误信息。  
print("决策图可能需要较新版本的SHAP库") # 提供建议。  
  
# ... (Interaction Plot代码已在上一阶段讲解) ...  
  
# ==================== 9. SHAP Heatmap ====================  
print("\n📊 9. 生成SHAP Heatmap...") # 打印状态信息。  
  
plt.figure(figsize=(15, 8)) # 创建画布。  
  
# 按SHAP值的总和对样本进行排序，以便相似的样本聚在一起。  
shap_sum = shap_values.sum(1) # 计算每个样本所有SHAP值的和。  
sorted_indices = np.argsort(shap_sum) # 获取排序后的索引。  
  
# 选择排序后的前30个样本进行热力图展示  
n_samples_heatmap = min(30, len(sorted_indices)) # 定义样本数量。  
selected_indices = sorted_indices[-n_samples_heatmap:] # 选择SHAP总和最高的30个样本。  
  
shap_heatmap_data = shap_values[selected_indices] # 获取这些样本的SHAP值。  
  
# 创建热力图  
im = plt.imshow(shap_heatmap_data.T, cmap='RdBu_r', aspect='auto') # T表示转置，使特征在Y轴，样本在X轴。  
plt.colorbar(im, label='SHAP Value') # 添加颜色条。  
  
plt.yticks(range(len(feature_names)), feature_names) # 设置Y轴标签为特征名。  
plt.xlabel('Sample Index') # 设置X轴标签。  
plt.ylabel('Features') # 设置Y轴标签。  
plt.title('LightGBM SHAP Values Heatmap', fontsize=16, fontweight='bold') # 设置标题。  
  
# 添加实际标签信息  
actual_labels = y_val_sample.iloc[selected_indices] # 获取这些样本的真实标签。  
for i, label inenumerate(actual_labels): # 遍历标签。  
color = 'red'if label == 1else'blue'# 根据标签确定颜色。  
plt.axvline(x=i, color=color, alpha=0.3, linewidth=2) # 在每个样本位置绘制一条垂直线代表其真实类别。  
  
# 添加图例  
from matplotlib.patches import Patch # 导入Patch用于创建自定义图例。  
legend_elements = [Patch(facecolor='red', alpha=0.3, label='Diabetes'),  
Patch(facecolor='blue', alpha=0.3, label='No Diabetes')]  
plt.legend(handles=legend_elements, loc='upper right') # 显示图例。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_shap_heatmap.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。
```

---

### **阶段七：深度分析与对比**

**阶段说明：** 这个阶段我们对最重要的特征进行更深入的挖掘，并对比不同方法得出的特征重要性。

1. **顶级特征分析**

* 我们选出最重要的一个特征，并从多个角度分析它：
* 它的值与它的SHAP值有什么关系？
* 它在患病组和非患病组的分布有何不同？
* 它的值如何直接影响模型的最终预测概率？
* 这提供了一个关于关键驱动因素的全方位视图。

1. **特征重要性对比**

* LightGBM模型自身会提供一个`feature_importances_`属性（通常基于特征被用于分裂节点的次数或增益）。
* 我们将这个内置的重要性与SHAP得出的重要性进行对比。
* SHAP的重要性通常被认为更可靠，因为它衡量的是特征对**模型输出**的实际影响，而不仅仅是内部结构。对比这两者可以验证我们的发现，或发现一些不一致之处。

**代码及逐行注释：**

python

```
# ==================== 10. 顶级特征分析 ====================  
print("\n📊 10. 生成顶级特征分析...") # 打印状态信息。  
  
# 选择最重要的特征进行详细分析  
top_feature_idx = np.argmax(np.abs(shap_values).mean(0)) # 找到平均绝对SHAP值最大的特征的索引。  
top_feature_name = feature_names[top_feature_idx] # 获取其名称。  
  
fig, axes = plt.subplots(2, 2, figsize=(15, 12)) # 创建2x2子图网格。  
  
# a. 最重要特征与SHAP值的关系  
plt.sca(axes[0, 0]) # 激活左上角子图。  
sns.regplot( # 使用seaborn绘制散点图和回归线。  
x=X_val_sample[:, top_feature_idx], # x轴是特征值。  
y=shap_values[:, top_feature_idx], # y轴是SHAP值。  
scatter_kws={'alpha': 0.5, 's': 50},  
line_kws={'color': 'red'}  
)  
plt.xlabel(top_feature_name) # 设置x轴标签。  
plt.ylabel('SHAP Value') # 设置y轴标签。  
plt.title(f'SHAP Values vs {top_feature_name}', fontweight='bold') # 设置标题。  
  
# b. 按类别统计分布  
plt.sca(axes[0, 1]) # 激活右上角子图。  
for label, color in [(0, 'blue'), (1, 'red')]: # 遍历两个类别。  
mask = y_val_sample == label # 创建掩码以选择特定类别的样本。  
label_name = 'No Diabetes'if label == 0else'Diabetes'  
if np.any(mask): # 确保该类别有样本。  
sns.kdeplot( # 绘制核密度估计图。  
X_val_sample[mask, top_feature_idx],  
color=color,  
label=label_name  
)  
plt.xlabel(top_feature_name) # 设置x轴标签。  
plt.ylabel('Density') # 设置y轴标签。  
plt.title(f'{top_feature_name} Distribution by Actual Class', fontweight='bold') # 设置标题。  
plt.legend() # 显示图例。  
  
# c. 特征与预测概率关系  
plt.sca(axes[1, 0]) # 激活左下角子图。  
probs = lightgbm_model.predict_proba(X_val_sample)[:, 1] # 获取所有样本的预测概率。  
plt.scatter(X_val_sample[:, top_feature_idx], probs, alpha=0.7, s=50) # 绘制散点图。  
plt.xlabel(top_feature_name) # 设置x轴标签。  
plt.ylabel('Predicted Probability') # 设置y轴标签。  
plt.title(f'Predicted Probability vs {top_feature_name}', fontweight='bold') # 设置标题。  
  
# d. 箱线图对比  
plt.sca(axes[1, 1]) # 激活右下角子图。  
feature_data = pd.DataFrame({ # 创建一个临时DataFrame以便用seaborn绘图。  
'Value': X_val_sample[:, top_feature_idx],  
'Class': ['Diabetes'if l == 1else'No Diabetes'for l in y_val_sample]  
})  
sns.boxplot(x='Class', y='Value', data=feature_data) # 绘制箱线图。  
plt.ylabel(top_feature_name) # 设置y轴标签。  
plt.title(f'{top_feature_name} by Class', fontweight='bold') # 设置标题。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_top_feature_analysis.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
# ==================== 11. 特征重要性对比 ====================  
print("\n📊 11. 生成特征重要性对比图...") # 打印状态信息。  
  
# 获取LightGBM内置特征重要性  
lightgbm_importance = lightgbm_model.feature_importances_ # 从模型中直接获取。  
  
# 计算SHAP特征重要性 (已在前面计算过)  
shap_importance = np.abs(shap_values).mean(0)  
  
# 创建对比DataFrame  
importance_df = pd.DataFrame({  
'Feature': feature_names,  
'LightGBM_Importance': lightgbm_importance,  
'SHAP_Importance': shap_importance  
})  
  
# 归一化重要性分数，使其在[0, 1]范围内，方便比较。  
importance_df['LightGBM_Normalized'] = importance_df['LightGBM_Importance'] / importance_df[  
'LightGBM_Importance'].max()  
importance_df['SHAP_Normalized'] = importance_df['SHAP_Importance'] / importance_df['SHAP_Importance'].max()  
  
# 按SHAP重要性排序  
importance_df = importance_df.sort_values('SHAP_Normalized', ascending=True)  
  
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 8)) # 创建1行2列的子图。  
  
# 左图：对比条形图  
y_pos = np.arange(len(importance_df)) # 创建y轴位置。  
ax1.barh(y_pos - 0.2, importance_df['LightGBM_Normalized'], 0.4,  
label='LightGBM Importance', alpha=0.8, color='skyblue')  
ax1.barh(y_pos + 0.2, importance_df['SHAP_Normalized'], 0.4,  
label='SHAP Importance', alpha=0.8, color='lightcoral')  
  
ax1.set_yticks(y_pos) # 设置y轴刻度。  
ax1.set_yticklabels(importance_df['Feature']) # 设置y轴标签。  
ax1.set_xlabel('Normalized Importance') # 设置x轴标签。  
ax1.set_title('Feature Importance Comparison', fontweight='bold') # 设置标题。  
ax1.legend() # 显示图例。  
ax1.grid(True, alpha=0.3) # 添加网格。  
  
# 右图：散点图显示相关性  
ax2.scatter(importance_df['LightGBM_Normalized'], importance_df['SHAP_Normalized'],  
alpha=0.7, s=80, color='green')  
  
# 添加特征名称标签  
for i, row in importance_df.iterrows():  
ax2.annotate(row['Feature'],  
(row['LightGBM_Normalized'], row['SHAP_Normalized']),  
xytext=(5, 5), textcoords='offset points',  
fontsize=8, alpha=0.7)  
  
ax2.set_xlabel('LightGBM Normalized Importance') # 设置x轴标签。  
ax2.set_ylabel('SHAP Normalized Importance') # 设置y轴标签。  
ax2.set_title('Importance Correlation', fontweight='bold') # 设置标题。  
ax2.grid(True, alpha=0.3) # 添加网格。  
  
# 添加对角线  
max_val = max(importance_df['LightGBM_Normalized'].max(),  
importance_df['SHAP_Normalized'].max())  
ax2.plot([0, max_val], [0, max_val], 'r--', alpha=0.5, label='Perfect Correlation')  
ax2.legend() # 显示图例。  
  
plt.tight_layout() # 自动调整布局。  
plt.savefig('lightgbm_importance_comparison.png', dpi=300, bbox_inches='tight') # 保存图表。  
plt.show() # 显示图表。  
  
print("\n" + "=" * 60) # 打印结束分隔线。  
print("🎉 LightGBM SHAP 可视化分析完成!") # 打印完成信息。  
print("=" * 60)  
print(f"📁 已生成以下可视化文件:") # 打印文件列表。  
shap_files = [  
'lightgbm_shap_summary.png',  
'lightgbm_shap_bar.png',  
'lightgbm_shap_beeswarm.png',  
'lightgbm_shap_waterfall.png',  
'lightgbm_shap_dependence.png',  
'lightgbm_shap_force_sample_*.png',  
'lightgbm_shap_decision.png',  
'lightgbm_shap_interaction_matrix.png',  
'lightgbm_shap_heatmap.png',  
'lightgbm_top_feature_analysis.png',  
'lightgbm_importance_comparison.png'  
]  
for file in shap_files:  
print(f"  ✅ {file}") # 逐个打印已生成的文件名。  
  
else: # 如果LightGBM模型未成功训练。  
print("❌ 未找到LightGBM模型，请确保LightGBM训练成功") # 打印错误信息。
```

---

### **阶段八：高级与专业级组合可视化**

**阶段说明：** 这部分代码展示了如何将多个SHAP图表组合成一个信息密度极高、适合用于报告或学术论文的**专业级**可视化面板。这需要更精细的布局控制（使用`matplotlib.gridspec`）和美学调整。

1. **高级组合可视化 (`Advanced Combined`)**

* 将最重要的全局摘要图（Beeswarm/Dot plot）和最重要的几个特征的依赖图放在同一个画布上。
* 这种布局方式能让读者一目了然地看到：哪些特征最重要（左侧），以及这些重要特征的具体影响模式是怎样的（右侧和下方）。
* 代码中还加入了寻找“拐点”的函数，试图在依赖图上自动标记出特征影响发生显著变化的阈值点。

1. **四象限分析 (`Quadrant Analysis`)**

* 选择最重要的两个特征，将它们的交互关系用一个四象限图来展示。
* X轴和Y轴分别是两个特征的值，散点的位置由特征值决定，颜色代表真实类别，点的大小代表这两个特征对预测的总贡献度。
* 这能帮助我们识别出高风险的“组合”，例如“高血糖”且“高BMI”的病人（可能集中在第一象限），并附上每个象限的统计数据。

1. **专业版组合可视化 (`Professional Combined`)**

* 这是对高级组合图的进一步美化和精炼。它使用了大量的`matplotlib`高级技巧来微调图表的每一个细节，例如：
* 精确控制字体、颜色、线条样式。
* 为摘要图和依赖图创建独立的、位置精确的颜色条。
* 在摘要图上同时展示散点（表示SHAP值分布）和条形图（表示平均重要性）。
* 添加详细的图例和解释性文本框。
* 最终的目标是生成一张无需过多口头解释、自身就能清晰传达复杂分析结果的“信息图”。

## 该文章案例

数据加微信免费获取。

对此套代码感兴趣的读者，可以选择赞赏30元向小编索取完整代码。

注：本代码全程Python语言实现，拿到代码后，先用示例数据复现跑通，确认环境没问题后，再上自己的数据。

**【********数据，请加微信免费********获取********】**

如果你对类似于这样的文章感兴趣。

欢迎关注、点赞、转发~

**Chaos AI Assistant公益站注册：****https://gpt-all.chat/auth?type=register&invite=MTM**

**官网：https://gpt-all.chat**

* **🆓 开源模型全免费：deepseek、qwen、llama 等**
* **😲 基础模型全免费：4.1-mini、o4-mini 等**
* **♾️ 对话真正无限制：不限时间、次数**
* **🫡 每周免费一个旗舰模型：本周：grok-3**

历史推文：

[机器学习——线性回归、决策树、随机森林、LightGBM模型的SHAP和LIME可解释性分析对比详解完整示例](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247485409&idx=1&sn=ebebcbdaa8c69e8f8f4897b197c9c2aa&scene=21#wechat_redirect)

[机器学习——因果推断方法的DeepIV和因果森林双重机器学习（CausalForestDML）示例](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247485288&idx=1&sn=08518e192a1af17e1187238e88b13419&scene=21#wechat_redirect)

[论文复现——肺癌数据高级模型比较与shap可视化分析代码解析](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247485257&idx=1&sn=7dfcc0fe9292ea7081b5d3e99b279f67&scene=21#wechat_redirect)

[论文复现——肺癌预测数据分析与逻辑回归、朴素贝叶斯、支持向量机、随机森林、K近邻、XGBoost、深度神经网络模型评估代码解析](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247485198&idx=1&sn=a444528db9a297bf71b35bbdede1dffd&scene=21#wechat_redirect)

[机器学习——材料力学XGBoost 分类，SHAP可视化分析、SMOTE解决不平衡、PCA降维、统计分析完整代码解析](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484804&idx=1&sn=39900572844edef6fc414a9776fb4e5c&scene=21#wechat_redirect)

[机器学习——SHAP可解释分析、EDA、逻辑回归、支持向量机（SVM）电动车数据集Python完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484772&idx=1&sn=f1ef48cd4b41dc484fd9f9c6d951ee29&scene=21#wechat_redirect)

[机器学习——集成学习、线性模型、支持向量机、K近邻、决策树、朴素贝叶斯、虚拟分类器分析电动车数据集Python完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484746&idx=1&sn=f7bf1e11fe6070079b46b0bf83c4aa2f&scene=21#wechat_redirect)

[深度学习与图像分类：基于鸟类图片分类项目的实践python完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484704&idx=1&sn=8aea2f06ddde53f1185d82fe51913dc5&scene=21#wechat_redirect)

[机器学习——二元Logistic回归算法实战：从数据预处理到模型评估python完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484696&idx=1&sn=286e98c338b4c0496682632f0460d58c&scene=21#wechat_redirect)

[机器学习——使用Lazypredict选择最优模型、Optuna调优、PCA降维进行客户信息数据分类分析python完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484678&idx=1&sn=c72d7bc4b09085c8a994c21ed883e3fe&scene=21#wechat_redirect)

[机器学习——处理多元分类数据的 7 种可视化方法Python 完整代码](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484663&idx=1&sn=190216f99fc923e501559fd552aeb408&scene=21#wechat_redirect)

[机器学习——基于颜色直方图和 SVM 分类水果图像代码解析python](https://mp.weixin.qq.com/s?__biz=MzkxNTczMjc1MQ==&mid=2247484626&idx=1&sn=4dc05fcc395d91d6def1ab2b05139601&scene=21#wechat_redirect)
---
title: Python数据处理神器dataprep，让数据分析效率提升10倍！
author: A逍遥之路
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzODg1OTk3OA==&mid=2247485148&idx=1&sn=e40ebc719b248df59a12bba4610844f2&chksm=e824dc7ad812dc81e90cd06f4ddd0f321bc3a91f5919c869fd934106386bc024544ab489555a&mpshare=1&scene=24&srcid=0216rJh1FWeivxSl0Lkq5sip&sharer_shareinfo=4634c17abf6e1660fe1ce091a3856dd3&sharer_shareinfo_first=4634c17abf6e1660fe1ce091a3856dd3#rd
---

> 看到标题就想点赞？别急，先往下看看有什么干货！

大家好啊，最近在做数据分析项目时发现了一个特别好用的Python库 - dataprep。这个库简直就是数据分析的及时雨！今天就给大家详细说道说道，保证让你看完就想马上用起来。

## 为啥要用dataprep？

干活要用对工具。pandas确实好用，但有时候光是数据清洗就要写一大堆代码。dataprep就是为了解决这个痛点而生的。它能帮你：

* 一行代码生成数据报告
* 智能处理缺失值
* 自动识别异常数据
* 可视化so easy

最重要的是，代码超级简单，小白也能快速上手！

## 快速开始

先装库：

```
pip install dataprep
```

来个最简单的例子感受一下：

```
from dataprep.eda import create_report  
import pandas as pd  
  
# 随便创建个数据集  
data = {  
    '姓名': ['张三', '李四', '王五', '赵六'],  
    '年龄': [25, 30, None, 35],  
    '工资': [8000, 12000, 15000, 10000]  
}  
df = pd.DataFrame(data)  
  
# 一行代码生成报告  
create_report(df)
```

就这？对，就这么简单！运行后会自动打开浏览器显示一个超详细的数据分析报告，包括：

* 数据概览
* 每列数据分布
* 异常值检测
* 相关性分析
* 还有超多可视化图表

## 处理数据缺失值

数据缺失是最烦人的问题之一，dataprep提供了超智能的处理方法：

```
from dataprep.clean import clean_missing  
  
# 自动处理缺失值  
df_clean = clean_missing(df)  
  
# 也可以自定义处理方式  
df_clean = clean_missing(df, missing_num='mean', missing_cat='mode')
```

这比手动写一大堆fillna方便多了吧？

## 数据可视化神器

来看看dataprep的可视化有多强：

```
from dataprep.eda import plot  
  
# 画各种图都超简单  
plot(df, 'age')  # 单变量分布  
plot(df, 'age', 'salary')  # 双变量关系  
plot(df, 'age', 'salary', 'gender')  # 三变量关系
```

一行代码就能生成高颜值的图表，还能自动选择最合适的图表类型！

## 实战案例：分析某公司员工数据

来个真实案例，看看如何用dataprep分析公司的员工数据：

```
import pandas as pd  
from dataprep.eda import create_report, plot  
from dataprep.clean import clean_missing  
  
# 读取数据（这里用模拟数据）  
data = {  
    '部门': ['技术', '销售', '技术', '市场', '销售'],  
    '工龄': [2, 5, None, 3, 4],  
    '工资': [15000, 12000, 18000, 10000, 13000],  
    '绩效': ['A', 'B', 'A', 'C', 'B']  
}  
df = pd.DataFrame(data)  
  
# 1. 先看看整体情况  
report = create_report(df)  
  
# 2. 处理缺失值  
df_clean = clean_missing(df, missing_num='mean')  
  
# 3. 分析工资分布  
plot(df_clean, '工资')  
  
# 4. 看看部门和工资的关系  
plot(df_clean, '部门', '工资')  
  
# 5. 分析绩效和工资的关系，按部门分组  
plot(df_clean, '绩效', '工资', '部门')
```

这个案例完美展示了dataprep的强大之处：

1. 代码极其简洁
2. 分析过程清晰明了
3. 输出结果专业美观

## 进阶技巧

当你掌握了基础用法，还可以试试这些进阶操作：

```
# 自定义报告内容  
create_report(df, title='员工分析报告',   
             correlations=True,  # 显示相关性  
             distributions=True,  # 显示分布  
             duplicates=True)    # 检查重复值  
  
# 设置图表样式  
plot(df, 'age', style={'theme': 'dark',   
                       'width': 800,   
                       'height': 500})
```

## 小贴士

1. 第一次使用时记得设置好显示中文的字体
2. 数据量大时生成报告可能需要等待一会
3. 可以把报告保存成HTML文件分享给同事

好了，关于dataprep的介绍就到这里。这个库确实能让数据分析效率提升好几倍，特别适合需要快速出分析结果的场景。

如果你也在学Python，欢迎点赞关注，我会持续分享Python实用技巧！

---

这篇文章是不是有点意思？快收藏起来慢慢看~

关注逍遥不迷路**，Python知识日日补！**

           对Python，AI，自动化办公提效，副业发展等感兴趣的伙伴们，                                   扫码添加逍遥，限免交流群

                                        备注【成长交流】
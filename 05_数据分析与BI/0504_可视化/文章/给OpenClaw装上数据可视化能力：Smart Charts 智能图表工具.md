---
title: 给OpenClaw装上数据可视化能力：Smart Charts 智能图表工具
author: LLMDev+
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODY1NzAwNA==&mid=2247485529&idx=1&sn=96e4224a6d46259471262cecdb61d984&chksm=9780f62dfb5b1ff12cb34149ebb55ff03a16b6f089d0c298d33855c6d2ea8d02c6296ac11e34&mpshare=1&scene=24&srcid=0401ynkcFtQ9TNjU3lWzRtFb&sharer_shareinfo=48a3bd5ee5586c8575f76091f2295200&sharer_shareinfo_first=48a3bd5ee5586c8575f76091f2295200#rd
---

**基于Agent Skills规范，让OpenClaw等智能体自动完成从数据到图表的全流程**

---

数据可视化是人类理解数据的直观方式，但对于AI助手而言，生成图表并非天生技能。如何让智能体理解数据、选择合适的图表、输出可直接展示的交互式报告？

今天介绍一个遵循**[Agent Skills](https://mp.weixin.qq.com/s?__biz=MzE5ODY1NzAwNA==&mid=2247485035&idx=1&sn=1f56035209986e3cd74a156cd895221b&scene=21#wechat_redirect)规范**的智能图表工具——Smart Charts。它是专为OpenClaw这类智能体助手设计的能力扩展，让AI助手能够自动完成从数据读取到图表生成的全流程。

---

01

—

什么是Agent Skills？

Agent Skills是一套让AI助手具备特定能力的规范标准。通过定义清晰的接口和配置，智能体可以调用外部工具来完成复杂任务。

Smart Charts正是基于这一规范开发的技能，它为AI助手提供了：

* 标准化的输入输出接口
* 明确的配置要求（输入目录、输出目录、数据库配置）
* 可组合的工作流（文件定位 → 数据解析 → 图表推荐 → 图表生成）

这意味着，当你使用支持Agent Skills的智能体（如[OpenClaw](https://mp.weixin.qq.com/s?__biz=MzE5ODY1NzAwNA==&mid=2247485197&idx=1&sn=38bc8a54a2771edb670d704b84934b67&scene=21#wechat_redirect)）时，只需说出需求，智能体就能自动调度这个技能，生成你需要的可视化报告。

---

02

—

这个技能能做什么？

### 1. 支持的数据源

* Excel文件（.xlsx, .xls）
* CSV文件（自动识别分隔符）
* JSON数据（文件或API）
* MySQL数据库（通过配置文件连接）

### 2. 生成的图表类型

* 折线图、柱状图、饼图、散点图、面积图、雷达图

### 3. 智能推荐能力

当数据来源不明确时，技能会自动分析数据特征，推荐最合适的图表类型，并给出评分和理由。

---

03

—

如何使用

### 1. 配置要求

使用该技能需要提供：

* **输入目录**（`input_dir`）：存放数据文件的路径
* **输出目录**（`output_dir`）：存放生成图表的路径
* **MySQL配置文件**（可选，仅当需要连接数据库时）：`sqlconfig.json`，内容如下

```
{  "mysql": {    "host": "localhost",    "user": "",    "password": "",    "database": "",    "port": 3306  }}
```

### 2. 工作流示例

以下是智能体在后台执行的操作（对用户透明）：

```
from scripts.file_locator import FileLocatorfrom scripts.data_parser import DataParserfrom scripts.chart_recommender import ChartRecommenderfrom scripts.chart_generator import ChartGenerator  
# 1. 定位文件locator = FileLocator(input_dir="/path/to/data")files = locator.locate_files("sales data")  
# 2. 解析数据parser = DataParser(config_file="sqlconfig.json")df = parser.parse_file(files[0])  
# 3. 获取推荐recommender = ChartRecommender()best_chart = recommender.get_best_recommendation(df)  
# 4. 生成图表generator = ChartGenerator(output_dir="/path/to/output")generator.generate_chart(    df=df,    chart_type=best_chart['chart_type'],    title="销售数据分析")
```

### 用户只需要对智能体说一句“分析这份销售数据并生成图表”，背后的所有步骤都由技能自动完成。

---

04

—

典型使用场景

### 场景一：Excel数据分析

“帮我分析上个月的用户增长数据，生成趋势图。”

智能体自动定位Excel文件，解析数据，推荐使用折线图，生成交互式HTML报告。

### 场景二：数据库报表

“从MySQL的订单表生成季度销售额报表。”

智能体读取配置文件连接数据库，执行查询，生成柱状图并输出完整报表。

### 场景三：多数据源整合

“把这两个CSV文件和数据库里的用户信息合并，分析用户行为分布。”

智能体分别读取不同来源的数据，整合后进行可视化。

---

05

—

价值在哪？

对于AI助手来说，数据可视化能力让它们从“会说话”变成“会展示”。用户不再需要手动操作Excel或编写代码，只需描述需求，就能获得可直接分享的图表。

对于开发者而言，Agent Skills规范让这些能力可以即插即用。只要智能体支持该规范，就能轻松集成Smart Charts技能，无需重复造轮子。

---

06

—

开源与贡献

Smart Charts基于Apache 2.0许可证开源，核心依赖包括ECharts、pandas、openpyxl等成熟库。欢迎使用并参与改进。

* **ClawHub地址**：https://clawhub.ai/neuhanli/smart-charts
* **GitHub地址**：https://github.com/neuhanli/skills

让AI助手更智能，从让它学会“看图说话”开始。
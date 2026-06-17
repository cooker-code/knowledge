---
title: DeepSeek-V3.1做图表：对Excel进行深度数据分析与挖掘（附高级Prompt提示词模板）
author: 悦呀AI笔记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484269&idx=1&sn=07265c74111b2d3f6891cb16268f708b&chksm=97e3e573f69d7a5f6afbd70b61867f6e8c9a7a52f70ce3d6685676483266c5eb79baba9b2de8&mpshare=1&scene=24&srcid=0902ZIX93H3scokyuB1fvtiu&sharer_shareinfo=7125d6a424213a6dd9c75c36ef78def5&sharer_shareinfo_first=7125d6a424213a6dd9c75c36ef78def5#rd
---

****欢迎点击👇关注我**

**您的关注是我持续分享的动力****

追AI的80后 | 国企上市公司“跨界者”（技术→产品→市场→AI）。坚信AI是下一个风口，更是超级个体的成长杠杆，在这里日常更新AI学习笔记，陪你借势向上。

今天的内容依旧是[DeepSeek做图表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzE5MTI2ODAxMg==&action=getalbum&album_id=4099218700367036419#wechat_redirect)系列，为你借助AI工具进行数据分析和生成图表提供思路。

**构建一个清晰、结构化、有约束条件的任务框架**，引导大模型进行系统性、深度的思考和分析，包含实践案例及完整Prompt模板。

实际使用时可基于Prompt模板，按照实际需求优化Prompt，供AI工具使用。

注意：仅提供思路，提示词也仅为示例模板，在应用时需要结合实际灵活修改，对着套用。

一、高级提示词核心要素

日常工作中，我们不仅要实现对数据的可视化，经常还需要对数据进行更深层次的分析，最好能依据数据做一些预测分析和深度挖掘，输出可落地的业务洞察。

这就需要编写高级提示词，以驱动深度数据分析与挖掘，需**结构化定义目标、约束条件、分析方法、输出要求**，并尽可能提供数据背景信息（如字段含义、业务场景）。

一个有效的分析提示词需包含以下模块（按优先级排序）：

1.**分析目标**：明确要解决的核心问题（避免模糊，用具体业务指标定义）。

2.**数据说明**：描述数据的结构、字段含义、时间范围、质量（如是否有缺失值/异常值）。

3.**分析方法约束**：指定需使用的统计方法/算法（如相关性分析、聚类、归因分析等），或禁止使用的方法。

4.**深度要求**：是否需要分层分析（如按地区/用户类型拆分）、对比分析（如同比/环比）、归因解释（如关键驱动因素）。

5.**输出格式**：要求可视化（图表类型）、结论层级（核心结论→支撑论据→建议）、量化精度（如保留2位小数）。

二、实战案例

第一步、测试数据集创建

PS:真实环境下没有这一步，直接上传数据即可。

手动创建一份只有10条记录的电商用户行为数据样本，字段定义：

user\_id: 用户唯一标识符（文本类型，可忽略分析）。

age: 用户年龄（整数，存在随机缺失值，表中以`N/A`表示）。

gender: 用户性别（分类类型：'M' - 男性，'F' - 女性）。

total\_purchase\_amount: 用户历史总消费金额（连续数值，单位：元）。

avg\_order\_value: 用户平均订单价值（连续数值，单位：元）。

number\_of\_visits: 用户近30天访问次数（整数）。

is\_returning\_customer: 是否为复购用户（二值变量：1 - 是， 0 - 否）。这是本分析的核心目标变量。

第二步、具体操作

（1）打开DeepSeek-V3.1聊天界面；

（2）上传待分析数据；

（3）用自然语言（Prompt提示词）描述你的分析需求。

可参考如下Prompt提示词模板：

```
请你扮演一名资深数据分析专家，擅长统计分析、机器学习和业务洞察。你的任务是对我提供的数据集进行深度分析，挖掘潜在规律、关联性和业务价值。  
【数据背景】1.  **数据集描述**：这是一份关于电商平台用户购买行为的模拟数据。2.  **业务目标**：本次分析的核心目标是：1) 识别高价值用户的核心特征；2) 找到驱动用户复购的关键因素；3) 基于发现为精准营销提供可操作的业务建议。3.  **数据样本**：数据以表格形式呈现，见上传附件，结构如下：4.  **字段定义**：    *   `user_id`: 用户唯一标识符（文本类型，可忽略分析）。    *   `age`: 用户年龄（整数，存在随机缺失值，表中以`N/A`表示）。    *   `gender`: 用户性别（分类类型：'M' - 男性， 'F' - 女性）。    *   `total_purchase_amount`: 用户历史总消费金额（连续数值，单位：元）。    *   `avg_order_value`: 用户平均订单价值（连续数值，单位：元）。    *   `number_of_visits`: 用户近30天访问次数（整数）。    *   `is_returning_customer`: 是否为复购用户（二值变量：1 - 是， 0 - 否）。这是本分析的核心目标变量。5.  **数据质量说明**：`age`字段存在缺失值；`total_purchase_amount`字段预计为右偏分布，存在极端值；`avg_order_value`字段对于总金额为0的用户也为0。  
【分析任务】请按以下步骤执行深度分析：  
**第一步：探索性数据分析（EDA）**1.  数据概览：输出样本量、字段类型、缺失值比例（特别是`age`字段）。2.  描述性统计：计算所有数值型字段（`age`, `total_purchase_amount`, `avg_order_value`, `number_of_visits`）的统计量（均值、中位数、标准差、min, max, 25%/50%/75%分位数）。3.  数据可视化：请用文字描述关键字段的分布图：    *   `age`的直方图分布（处理缺失值后）。    *   `total_purchase_amount`的箱线图（描述其分布和异常值）。    *   `gender`的饼图或条形图（显示性别比例）。    *   `is_returning_customer`的条形图（显示复购用户与非复购用户的比例）。  
**第二步：深度数据挖掘与关联分析**1.  相关性分析：计算所有数值型字段之间的相关系数矩阵（建议使用斯皮尔曼相关系数，因为数据可能不服从正态分布）。指出强相关（|r| > 0.5）和显著（p-value < 0.05）的关系。2.  群体对比分析：    *   按`gender`分组，比较男性和女性在`total_purchase_amount`, `avg_order_value`, `number_of_visits`上的差异（计算各组的均值和中位数，并评论其显著性）。    *   将`age`分为三个组别（<30, 30-50, >50），分析不同年龄组在复购率（`is_returning_customer`的平均值）和平均消费水平上的差异。3.  预测性分析：以`is_returning_customer`为因变量，构建一个逻辑回归模型。分析`age`, `gender`, `avg_order_value`, `number_of_visits`等自变量对复购概率的影响。    *   输出模型系数、OR值（Odds Ratio），并解释其含义（例如：“在控制其他变量不变的情况下，`number_of_visits`每增加一次，用户复购的几率（Odds）会增加X%”）。    *   评估模型的核心预测因子。  
【输出要求】1.  **执行摘要**：首先用一段话总结最关键的发现（例如：复购用户的核心特征是什么？哪个因素对复购的影响最大？）。2.  **分析正文**：严格按照上述步骤，分小节（1. EDA, 2. 相关性分析, 3. 群体对比, 4. 建模分析）呈现你的分析过程、结果和解读。所有重要数值需保留两位小数。3.  **可视化描述**：对每个需要可视化的部分，用文字清晰描述图表所揭示的模式和结论。4.  **业务建议**：基于发现，提出3条具体、可操作的营销策略建议（例如：针对哪类用户进行重点召回？如何提升用户的访问频率？）。5.  **局限性**：简要说明本模拟数据分析的局限性。
```

第三步、结果输出

通常情况下，均需要对输出结果进行校验。如不满足需求或出现错误，可指出问题进行修正，直到输出满意结果。

### 特此说明：因仅为演示举例，我并未对生成的报告细节进行详细校对。

### 这样，您将得到一份结构完整、洞察深刻的数据分析报告。

PS：此系列文章我只是拿DeepSeek举例，大家可以参考提示词模板，使用其他大模型或产品对比输出结果。

*本文为【DeepSeek做图表*】*系列最新内容，关注我获取更多AI学习笔记、AI办公技巧！*

*【重要提醒】微信调整了推荐机制。如果不想错过每期推文，请为悦呀AI笔记加关注及星标！随手点击下方名片→点“关注公众号”→点右上角（…）弹出菜单栏→点“设为星标”即可。*

延伸阅读：

[DeepSeek做图表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzE5MTI2ODAxMg==&action=getalbum&album_id=4099218700367036419#wechat_redirect)

往期文章：

[DeepSeek-V3.1做图表：自动化处理Excel并生成可视化报告（附全流程Prompt模板）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484221&idx=1&sn=6f7a138033680621cd4c2044f21534e0&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：告别手工Excel！三句话让AI为你做完整的数据分析（附AI提示词）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484227&idx=1&sn=98a0af9658dc865590504003b3988850&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：3招给数据“戴口罩”，AI高效办公又安全（含方法论干货）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484231&idx=1&sn=de4e0af569255d0315166513791de532&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：如何让AI生成“会讲故事”的图表？（附Prompt提示词）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484235&idx=1&sn=e89dff19ad86a34ebc2b2d4910dbe87d&scene=21#wechat_redirect)

[DeepSeek-V3.1做图表：告别加班！自动生成周报/月报数据图表（10个Prompt模板）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484190&idx=1&sn=6e91fe41e48931c0234d36fd751344c7&scene=21#wechat_redirect)

[DeepSeek做图表：9个常见问题+对应Prompt解法（新手必看）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247483986&idx=1&sn=d9d41574d745fdca21de9e11b9afe8ab&scene=21#wechat_redirect)

[DeepSeek做图表：新手避坑手册（干货预警+ 超实用Prompt）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247483980&idx=1&sn=af7f0dee4b7f52a0a00f824bdedcedb1&scene=21#wechat_redirect)

[DeepSeek做图表：10个数据分析与图表选择方法论指令（含Prompt）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247483903&idx=1&sn=d6917cc65101ffe5170bf3095c8fbd9c&scene=21#wechat_redirect)

[DeepSeek做图表：解放家长和老师！用AI搞定“成绩分析”+“学习趋势可视化”（附提示词）](https://mp.weixin.qq.com/s?__biz=MzE5MTI2ODAxMg==&mid=2247484250&idx=1&sn=8a1a0fd30859d9fd63b2bc68ab2eb56d&scene=21#wechat_redirect)

---

**以上，就是本次的分享，对你有启发吗？**

**如果对你有帮助，欢迎关注、点赞、转发、收藏**

**PS：如果你想听的我没有说到，请告诉我。**

**个人观点、仅供参考**
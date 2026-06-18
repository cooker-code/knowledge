---
title: 效率飙升！数分师的AI新玩具：打造SQL知识库，解锁智能下钻自由
author: 数据打工人的自我修养
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247483785&idx=1&sn=224276f5b32c123bfd6daf0624b3f2c7&chksm=97d2646027a82e0cf9a13c59ffa146c5d5332cb2c2bd0b7bb5b61950f95b88230dcc630d362a&mpshare=1&scene=24&srcid=09127m4YID4vpRQkzyfDVXpl&sharer_shareinfo=bdc947f957344a88ce01a90523deeb59&sharer_shareinfo_first=bdc947f957344a88ce01a90523deeb59#rd
---

以往，数据分析师日常工作50%左右的时间在构思&编写SQL，业界有调侃"SQL Boy"。而在编写SQL中，又有相当一部分时间在进行指标维度拆解，指标下钻。

作为资深表哥表姐中的一枚，在编写SQL方面，我已历经两代：

SQL 的第一代：

全手搓SQL，有部分同学可能会将一些使用比较频繁的代码给收藏起来，以便后续Ctrl C+V。在数仓设计层面，则会将一些高频使用的数据落地中间表，牺牲存储搭建宽表（维度冗余）以提升作业编写效率，减少计算消耗。

SQL 的第二代：AI智能补全，当我首次接触到这个，不禁深深感叹AI的魅力，它会根据我编写的内容自动推测我下一步要编写什么。因为AI有对元数据训练，它能帮你补全字段名（可以提供下拉列表），能智能补全字段分箱逻辑，如果你写的是一个json提取函数，当你输入函数和字段名时，它能推测补全key及字段命名。

在去年年底，因为分析主要为业务经营分析，经营分析诸如异常定位又以维度下钻居多。那会我在构思完善标准化维度宽表，编写python模块，分析师输入参数，程序自动编写SQL。(核心业务场景，大多逻辑是固定的)

还没等我坚挺过来（跑路了），已经是25年。随着deepseek横空出世，国内AI产业进入快速发展期，各大厂商竞相发力。

我想，SQL 的第三代马上就要普及：数据分析师不再频繁手搓SQL，更多的是面向AI分析。数分师输入提示词，AI自动编写SQL拉取数据分析。

AI生成SQL可用性极大程度上依赖于元数据质量，业务规则信息。对于应用团队则需对业务规则有统一认识，比如"活跃用户数"指标定义。

当面对集团级数千甚至数万张表时，由于元数据缺失、定义不明确以及历史SQL采集存在各种问题，AI生成SQL的异常率可能较高。然而，针对单个部门或小规模团队，在确保数据质量并完善AI知识库的情况下，AI生成SQL的成功率相当可观，至少能显著减少数据分析师编写SQL的时间消耗。

如下为个人基于第三方AI平台构造SQL生成智能体实践，平台选用的是腾讯的元器（大家可按需选择平台入口：https://yuanqi.tencent.com/）

平台知识库允许上传word、txt、pdf等多种文件类型，免费版可上传文件数达100个，同时允许用户填写知识库说明，以便模型更好地理解调用我们上传的知识库。

智能体创建好后可以分享给其他人使用，支持api调用：

python api调用过demo：

```
import requeststoken = 'xxxxxx'   # 平台生成的tokenurl = 'https://yuanqi.tencent.com/openapi/v1/agent/chat/completions'headers = {    'X-Source': 'openapi',    'Content-Type': 'application/json',    'Authorization': f'Bearer {token}'  # <token>替换为实际token，}data = {    "assistant_id": "jC4nUeCKE6jK",    "user_id": "<userid>",  # 替换为实际userid，这个用户id可不填    "stream": False,    "messages": [        {            "role": "user",            "content": [                {                    "type": "text",                    "text": "你会做什么"                }            ]        }    ]}response = requests.post(url, headers=headers, json=data)result = response.json()reply = result['choices'][0]['message']['content']print(reply)
```

`如下为智能体创建步骤：`

1. 首页左侧点击 创建智能体，选择 对话式智能体

2. 基础设定 选项卡依次填写智能体名称、简介以及预设提示词等信息。

我这里的提示词是：

```
你是一名SQL专家，熟悉大数据诸如Hive、SparkSQL、Trino以及关系型数据库MySQL、Oracel等计算引擎。在编写SQL时你会充分考虑如下几点：1. SQL简洁可用2. SQL运行性能3. SQL代码可读性4. SQL优化提示5. 生成SQL代码复核  
请基于如下要求，编写一段SQL：
```

3. 高级设定 选项卡选择模型及上传自己的知识库。其他功能按需选择，然后点击页面右上角的发布按钮即可。（模型推荐选择deepseek r1）

这里重点对知识库文件上传做下说明

就数仓规范设计， 库表、字段（词根）、码值，同样的内容尽可能的保持一致，比如用户id统一以user\_id命名。

知识库上传哪些内容（同字段存在不同命名的，可以上传命名说明文件）：

表元数据信息-上传字段设计（我这里上传了3个表：订单表、商品维表、用户标签表）：

字段元数据信息-上传字段设计（包含了主键表示、字段枚举值及码值映射）：

业务规则 - 上传字段设计（这里仅是示例）：

历史采集SQL - 上传内容设计（这里仅是示例）：

下面是应用，输入需求：

```
统计各个年龄段的如下三个指标：订单金额、订单数、用户数﹣筛选条件：1．用户学历：本科；2．用户性别：女性3．订单商品一级分类：A4．订单时间：2025年5月5. 订单状态：成功﹣年龄段分层如下：1.18岁及以下，命名为"<=18"2.19-22，字段值为"19-22"3.23-25，字段值为"23-25"4.26岁及以上，字段值为">=26"
```

模型返回如下：

```
-- 统计各年龄段订单金额、订单数、用户数-- 筛选条件：用户学历本科、女性、商品一级分类A、2025年5月订单、订单状态成功SELECT    age_group,    SUM(amount) AS total_amount,  -- 订单金额总和    COUNT(order_no) AS order_count,  -- 订单数（基于唯一订单号）    COUNT(DISTINCT user_id) AS user_count  -- 用户数（去重用户ID）FROM (    SELECT        o.order_no,        o.amount,        o.user_id,        u.age,        -- 年龄段分层逻辑        CASE            WHEN u.age <= 18 THEN '<=18'            WHEN u.age BETWEEN 19 AND 22 THEN '19-22'            WHEN u.age BETWEEN 23 AND 25 THEN '23-25'            ELSE '>=26'        END AS age_group    FROM        db.order_info o    JOIN        db.dim_goods g ON o.goods_id = g.goods_id  -- 连接商品维度表    JOIN        db.user_tag u ON o.user_id = u.user_id AND o.dt = u.dt  -- 连接用户标签表，基于用户ID和日期分区    WHERE        u.education_level = 3  -- 筛选学历本科（枚举值3）        AND u.gender = 2  -- 筛选女性（枚举值2）        AND g.PRIMARY_CATEGORY = 'A'  -- 筛选商品一级分类A        AND o.dt BETWEEN '2025-05-01' AND '2025-05-31'  -- 筛选2025年5月订单        AND o.status = 2  -- 筛选订单状态成功（已支付，枚举值2）) AS subqueryGROUP BY    age_groupORDER BY    age_group;  -- 按年龄段排序，便于阅读
```

---

文章就到这里结束啦

有兴趣尝试的同学可以关注公众号，后台回复“智能体”获取我这个智能体使用链接

感谢阅读，点击下方关注我，谢谢支持！

---
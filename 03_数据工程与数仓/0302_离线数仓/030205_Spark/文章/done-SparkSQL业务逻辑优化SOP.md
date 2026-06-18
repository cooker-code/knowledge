> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkSQL业务逻辑优化方法|SparkSQL业务逻辑优化方法]]
---
title: SparkSQL业务逻辑优化SOP
author: 炼数成丹
date:
url: https://mp.weixin.qq.com/s?__biz=MzI4NTc5ODA0OA==&mid=2247483749&idx=1&sn=64c1b92300ae77e868b79b06170f21ed&chksm=ea4d3c9ab66c4c3752872b783d0d044f4fd5db589d4e05821c3a4597e21bc35c8b92eb1adc22&mpshare=1&scene=24&srcid=0918edJoZAVJD6nceHJgvKIw&sharer_shareinfo=898b7f0c962971a7bd05be05f8968bb2&sharer_shareinfo_first=898b7f0c962971a7bd05be05f8968bb2#rd
---

▫ 场景痛点

⚠ 你是否：

跑个SQL等半小时起步

总是OOM报错弹窗

倾斜Key导致长尾任务拖垮集群

✨ 四大核心优化流程

1⃣ 数据瘦身术

-- 提前过滤无效数据！  

SELECT /\*+列裁剪\*/ col1,col2  

FROM table  

WHERE dt='2023'       -- 分区裁剪  

  AND key IS NOT NULL -- 过滤Null值  

  AND value > 0       -- 业务数据范围提前瘦身  

2⃣ Join乾坤大挪移

# 错误顺序：  

大表(120亿) JOIN 中表(25亿) JOIN 小表(1亿)  

# 正确姿势：  

 小表(1亿) JOIN 大表(120亿)  ➡ 结果集 JOIN 中表(25亿)  

📢 小表广播：/\*+ BROADCAST(small\_table) \*/

3⃣ 倾斜拆弹专家

👉 拆解法（热Key分离）

-- 1. 抓Top1%热Key   （根据自身业务情况适配定义 比例）

CREATE TABLE hot\_keys AS  

SELECT key FROM tbl  

GROUP BY key  

ORDER BY count(1) DESC  

LIMIT (SELECT COUNT(\*)\*0.01 FROM tbl)  

-- 2. 分开计算再合并  

SELECT \* FROM non\_hot\_data JOIN other\_table  

UNION ALL  

/\*+广播热数据\*/ SELECT \* FROM hot\_data JOIN other\_table  

👉 打散法（扩容降维）

-- 倾斜表添加随机后缀  

SELECT id, CONCAT(key,'\_',FLOOR(RAND()\*10)) AS new\_key  

FROM big\_table  

-- 维度表爆炸扩容  

SELECT dim.\*, suffix   

FROM dim\_table  

LATERAL VIEW EXPLODE(ARRAY(0,1,2,3,4)) t AS suffix

可以先使用 拆解 ，分而治之，比较容易理解，JOIN复杂度降低，注意处理 key 是否含有NULL 。

参考文献：

《SparkSQL内核解析》
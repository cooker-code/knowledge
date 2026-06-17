---
title: MySQL数据分析：计算一个值在其分组中的百分等级
author: 数据分析之渔
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI1NzczMDIwNw==&mid=2247486238&idx=1&sn=5923fcd8d182c7817ff7459480888763&chksm=eb4ccd812cb0125e1829c0f53f8547f8c12a3412c275a3120326d279f6f51bdfadbba3d2a5f0&mpshare=1&scene=24&srcid=1105YTeFZPGsoli54hqIhes6&sharer_shareinfo=c0d7f7454b11596e1fcfb6fc31647ce3&sharer_shareinfo_first=c0d7f7454b11596e1fcfb6fc31647ce3#rd
---

点击上方"数据分析之余"

关注我们吧！

窗口函数PERCENT\_RANK()属于分布式函数，在MySQL中PERCENT\_RANK()用于计算一个值在其分组中的百分等级

它的返回值范围从0到1，表示一个值在排序后的数据集中相对于其他值的位置

公式如下：

rank-1

PERCENT\_RANK= ————————

total\_rows−1

total\_rows 是总行数，rank是当前排行位置

**示例1 Quantity降序：**

select \* ,PERCENT\_RANK() over(order by Quantity desc) as percent\_ranking

from quantity;

结果中的percent\_ranking介于0到1之间，反映了各个Quantity在整个Quantity分布中的相对位置

比如按倒序排序，100在整个Quantity分布中是最高的所以percent\_ranking是0，而82在整个Quantity分布中是最小的因此percent\_ranking是1

另外升序与倒序得到的percent\_ranking结果是不一样的

**示例2 Quantity升序：**

select \* ,PERCENT\_RANK() over(order by Quantity) as percent\_ranking 

from quantity;

**示例3  以orderID分区：**

select \* ,PERCENT\_RANK() over(partition by orderID order by Quantity desc) as percent\_ranking 

from quantity;
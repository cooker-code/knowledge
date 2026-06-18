> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQLJoin语义与长尾治理|SQLJoin语义与长尾治理]]
---
title: SQL进阶技巧：Hive如何巧解和差计算的递归问题？【应用案例2】
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484297&idx=1&sn=bca0f82513140bd48579080994c77f96&chksm=e9425ffb0cd6ca34a6e89d5175ff921118e7f9c064573d4f0549c6d002ffcaf11e2c56aeab74&mpshare=1&scene=24&srcid=08070TfFBrPQe7cg5UnsoJ9E&sharer_shareinfo=bc3354c56d1e2e29f1beaf0a42eac8a0&sharer_shareinfo_first=bc3354c56d1e2e29f1beaf0a42eac8a0#rd
---

点击上方「蓝字」关注我们

>  累计值分析模型是一种用于分析和预测累计值数据的统计模型。它主要用于处理随时间积累的数据，例如销售额、用户数量、网站访问量等。累计值分析模型的目的是通过对历史数据的分析，揭示数据的趋势和模式，以便进行预测和决策。
>
>     累计值分析模型可以帮助我们理解数据的增长趋势、周期性变化、季节性影响等。它可以用于预测未来的累计值，帮助企业进行资源规划、市场营销、产品需求预测等决策。常见的累计值分析模型包括累计和移动平均、指数平滑、趋势分析等。
>
>      总的来说，累计值分析模型是一种重要的统计工具，可以帮助我们理解和利用累计值数据的特征，从而做出更准确的预测和决策。累计值模型分析可以帮助我们深入了解数据的趋势和模式，预测未来的累计值，优化资源分配和决策制定，以及监测异常行为和安全威胁。这对于企业的发展和决策具有重要意义。

01

# 需求描述

有如下数据：反应了每月的页面浏览量

现需要按照如下规则计算每月的累计阅读量，具体计算规则如下：

最终结果如下：

02

# 数据准备

```
with data as(select '2024-01' as month ,2  as pv union allselect '2024-02' as month ,5  as pv  union allselect '2024-03' as month ,6  as pv  union allselect '2024-04' as month ,9  as pv  union allselect '2024-05' as month ,60  as pv  union allselect '2024-06' as month ,11  as pv )
```

03

# 数据分析

从本文要求的结果来看，每一行要得到的结果都与前面计算过的行相关，咋一看需要去做递归计算，而对于hive中没有递归计算怎么办？如何实现呢？我们将规则进行分析，并从最终的结果来看，一步步进行拆分，得到如下表

通过上述拆分可以发现，res等价于拆分结果1，而拆分结果1又是拆分结果2的累加，拆分结果2是pv的累加，因此看起来是递归计算的问题实际上转换为pv的两次累加，具体SQL如下：

```
with data as(select '2024-01' as month ,2  as pv union allselect '2024-02' as month ,5  as pv  union allselect '2024-03' as month ,6  as pv  union allselect '2024-04' as month ,9  as pv  union allselect '2024-05' as month ,60  as pv  union allselect '2024-06' as month ,11  as pv ) select month     , pv     , acc_pv     , sum(acc_pv) over (order by month) resfrom    (select month          , pv          , sum(pv) over (order by month) acc_pv     from data) t
```

# 04 小结

本文分析了一种和差计算的递归问题，通过对目标的拆解，找出规律，采用累计叠加的思想巧妙解决此类问题。

关于规则变形实现，有兴趣的可以尝试：

```
with data as(select 1 as month ,'a'  as pv union allselect 2 as month ,'b'  as pv  union allselect 3 as month ,'c'  as pv  union allselect 4 as month ,'d'  as pv  union allselect 5 as month ,'e'  as pv  union allselect 6 as month ,'f'  as pv )   select month          ,pv          ,replace(concat_ws('+',collect_list(concat_ws('*',aa,cast(num as string)))),'*1','') res    from     (select month           , pv           , aa           , count(*)  as num      from (select month                 , pv                 , aa            from (select month                       , pv                       , concat_ws(',', collect_list(res) over (order by month)) res                  from (select month                             , pv                             , concat_ws(',', collect_list(pv) over (order by month)) res                        from data) t) t lateral view explode(split(res, ',')) tmp as aa) t      group by month             , pv             , aa) tgroup by month,pv;
```

# 往期推荐

###### [彻底理解冷门函数PERCENT\_RANK()函数？如何利用其特性巧解实际问题？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484162&idx=1&sn=382ea6107589876573246866c11e554d&chksm=e8e21722df959e3440d1339070ceaccf92306e13be173b533c604f7b669ad73a722906b600d0&scene=21#wechat_redirect)

###### [数据指标异常应如何排查？完整的解决思路](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484175&idx=1&sn=ec7816ae95c37104d48fa1e933c7e7a7&chksm=e8e2172fdf959e39513e0db27c6776e57222aa246473b55720111762ee0a8daa6fb35deeb20a&scene=21#wechat_redirect)

###### [SQL高级技巧：如何准确求近30天指标](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247483998&idx=1&sn=64c96b5e132cc9b36a416ee0a7dc90e0&chksm=e8e2167edf959f684af31ef724443e02653dba702eaf7f437be6ecdc8d7cd8b1cfca0e2bd28b&scene=21#wechat_redirect)

###### 会飞的一十六

微信号：ddan\_hashcode

扫码关注 了解更多

点个在看 你最好看
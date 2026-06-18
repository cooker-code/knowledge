---
title: SQL进阶技巧：如何将数值数据扩充完整？
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484557&idx=1&sn=77cd222a5511bd15c902a693e06af591&chksm=e9abd3ae2f673caad9c90acd59571c40fb77a81bcae3e42dcbe67ff8003fbb764bef1bc6e7b1&mpshare=1&scene=24&srcid=0820xgUB58yUwBahtwPuPzsz&sharer_shareinfo=3037774dfc4a68f53c3f456f42f6bf9c&sharer_shareinfo_first=3037774dfc4a68f53c3f456f42f6bf9c#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL行列转换与空值处理|SQL行列转换与空值处理]]


01

—

需求描述

表名：data 表字段及内容：

a

3

2

4

**需求1：数据扩充，得到如下结果**

**需求2： 数据扩充，排除偶数**

02

—

数据准备

```
 with data as(
select 3 as a  union all
select 2 as a  union all
select 4 as a
 );
```

03

—

数据分析

采用公式：

> #### posexplode(split(space(a), '(?!$)'))

通过上述公式将当前数据展开

```
with data as (select 3 as a              union all              select 2 as a              union all              select 4 as a)select a     , pos + 1from data lateral view posexplode(split(space(a), '(?!$)')) t as pos, val
```

**第二步：对字段b进行分组合并**

合并公式：

> conca\_ws(',',collect\_list(X))

```
with data as (select 3 as a union all select 2 as a union all select 4 as a)select a,concat_ws(',',collect_list(cast(b as string))) bfrom (select a , pos + 1 b from data lateral view posexplode(split(space(a), '(?!$)')) t as pos, val) tgroup by a
```

第三步：去除偶数的合并

去除偶数，b % 2 != 0,SQL如下：

```
with data as (select 3 as a              union all              select 2 as a              union all              select 4 as a)select a, concat_ws(',', collect_list(cast(b as string))) bfrom (select a           , pos + 1 b      from data lateral view posexplode(split(space(a), '(?!$)')) t as pos, val) twhere b % 2 != 0group by a
```

04

—

小结

本文给出了一种数据扩展与合并的方式，数据的扩展主要利用公式：

```
lateral view posexplode(split(space(a), '(?!$)'))
```

数据的合并主要利用如下公式：

```
conca_ws(',',collect_list(X))
```
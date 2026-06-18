---
title: SQL进阶技巧：有序行转列问题如何保证不同字段内容有序性及内容一一对应？【collect_list函数有序性保证问题】
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484348&idx=1&sn=d231346b30828c65543188ab080a8013&chksm=e8e2179cdf959e8a0e6484def88869d42ef945ec12ec52e2c72a9b31c371b00ff26abd143fdf&mpshare=1&scene=24&srcid=08150wnjzpx1Knx64Yp1XMYA&sharer_shareinfo=3acd63b922e26aa0e38b1d853fbc01cf&sharer_shareinfo_first=3acd63b922e26aa0e38b1d853fbc01cf#rd
---

点击上方「蓝字」关注我们

> 本文针对 多行转列过程中如何保证不同字段内容有序性及内容一一对应这一问题给出了解决方案。整体解决思路分如下几个步骤：
>
> 步骤1：按照业务规则指定排序顺序，一般采用case when 指定或基于row\_number()进行排序指定。具体根据实际需求分析得出采用哪一种。
>
> 步骤2：采用collect\_list() ove( partition by order by)窗口函数进行求解，具体需要在order by中指定合并的顺序。为什么此处采用collect\_list()over()窗口函数而不是其聚合函数，主要是因为保证有序性，只有collect\_list()over()能够指定顺序
>
> 步骤3：过滤出最终符合条件的数据

01

# 需求描述

**有如下需求，需要将左边的表变换成右边的表，注意字段内容的顺序及对应内容的一致性。**

**第一个字段为name,第二个字段为subject，第三个字段为score，变换后要求subject按照语文、数学、英语排列，且score和subject之间内容保持一一对应。**

## **数据准备**

```
with data as (    select '张三' as name , '数学' as subject , 80 as score  union all    select '张三' as name , '英语' as subject , 82 as score  union all    select '张三' as name , '语文' as subject , 95 as score  union all    select '李四' as name , '数学' as subject , 90 as score  union all    select '李四' as name , '英语' as subject , 93 as score  union all    select '李四' as name , '语文' as subject , 90 as score  union all    select '王五' as name , '数学' as subject , 92 as score  union all    select '王五' as name , '英语' as subject , 88 as score  union all    select '王五' as name , '语文' as subject , 88 as score  union all    select '赵六' as name , '数学' as subject , 84 as score  union all    select '赵六' as name , '英语' as subject , 68 as score  union all    select '赵六' as name , '语文' as subject , 77 as score )
```

02

# 数据分析

题目考察的是行转列，但是难度要比简单行转列要复杂很多。本题首先要求subject顺序按照语文、数学及英语排列，其次score中的对应分值内容必须和subject保持一一对应。看似老生常谈的问题，但实则面试通过率并不高，因为候选人往往忽略了有序性及内容的一一对应。

如果我们直接进行行转列，看看会有什么结果：

```
select  name      , concat_ws(',',collect_list(subject)) sub      , concat_ws(',',collect_list(cast(score as string))) scofrom data
```

在hive中collect\_list()函数并不能像其他数据库中如 group\_concat()函数等可以指定字段的顺序，collect\_list()在大数据体系下遇到shuffle过程还可能出现乱序。因此为了保证顺序性，我们先按照规则进行指定排序，语文标为1，数学标为2，英语标为3，具体如下SQL所示

```
select  name      , subject      , score      , case when subject ='语文' then 1             when subject ='数学' then 2             when subject ='英语' then 3        end seq from data
```

其次，我们采用concat\_ws()+collect\_list() ove()函数 进行数据行转列，注意此处采用的是collect\_list() ove()分析函数，目的是为了能够在窗口中指定顺序，保证合并数据的有序性。具体SQL如下：

```
select name,       seq,       concat_ws(',', collect_list(subject) over (partition by name order by seq))               subject,       concat_ws(',', collect_list(cast(score as string)) over (partition by name order by seq)) scorefrom (select name           , subject           , score           , case                 when subject = '语文' then 1                 when subject = '数学' then 2                 when subject = '英语' then 3        end seq       from data) t
```

由于窗口函数 collect\_list() ove()在滑动的过程中会针对每行数据都会生成一个记录，通过上面结果我们可以看到只有seq等于3的时候，才是我们想要的结果，因此我们对上述结果进行过滤，获得最终的SQL如下：

```
with data as (    select '张三' as name , '数学' as subject , 80 as score  union all    select '张三' as name , '英语' as subject , 82 as score  union all    select '张三' as name , '语文' as subject , 95 as score  union all    select '李四' as name , '数学' as subject , 90 as score  union all    select '李四' as name , '英语' as subject , 93 as score  union all    select '李四' as name , '语文' as subject , 90 as score  union all    select '王五' as name , '数学' as subject , 92 as score  union all    select '王五' as name , '英语' as subject , 88 as score  union all    select '王五' as name , '语文' as subject , 88 as score  union all    select '赵六' as name , '数学' as subject , 84 as score  union all    select '赵六' as name , '英语' as subject , 68 as score  union all    select '赵六' as name , '语文' as subject , 77 as score)select     --过滤符合条件的结果      name,       subject,       scorefrom (select name,             seq,             --指定合并时候排序规则，保证有序性             concat_ws(',', collect_list(subject) over (partition by name order by seq))               subject,             concat_ws(',', collect_list(cast(score as string)) over (partition by name order by seq)) score      from (select name                 , subject                 , score                 ---步骤1：基于排序规则，指定排序顺序                 , case                       when subject = '语文' then 1                       when subject = '数学' then 2                       when subject = '英语' then 3              end seq             from data) t) t where seq = 3;
```

03

# 小结

本文针对 多行转列过程中如何保证不同字段内容有序性及内容一一对应这一问题给出了解决方案。整体解决思路分如下几个步骤：

步骤1：按照业务规则指定排序顺序，一般采用case when 指定或基于row\_number()进行排序指定。具体根据实际需求分析得出采用哪一种。

步骤2：采用collect\_list() ove( partition by order by)窗口函数进行求解，具体需要在order by中指定合并的顺序。为什么此处采用collect\_list()over()窗口函数而不是其聚合函数，主要是因为保证有序性，只有collect\_list()over()能够指定顺序

步骤3：过滤出最终符合条件的数据

# 往期推荐

SQL进阶技巧：间隔连续问题【断点分组思想】

######

SQL进阶技巧：如何计算先进先出库龄问题？

###### [SQL高级技巧：如何准确求近30天指标](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247483998&idx=1&sn=64c96b5e132cc9b36a416ee0a7dc90e0&chksm=e8e2167edf959f684af31ef724443e02653dba702eaf7f437be6ecdc8d7cd8b1cfca0e2bd28b&scene=21#wechat_redirect)？

###### 会飞的一十六

微信号：ddan\_hashcode

扫码关注 了解更多

点个在看 你最好看
---
title: SQL进阶技巧：如何优雅求解指标累计去重问题？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247486917&idx=1&sn=056825c01fa5a7a7449d20e5026697de&chksm=e9245a4ee193da4173ef635e5b682c444fab94c974c3b7d95148e1f2205b3ca1b6a40c86eaa7&mpshare=1&scene=24&srcid=10257bindWZIwAOd3RZeoMT8&sharer_shareinfo=2b056a054081e6d8494f28157bfd92fe&sharer_shareinfo_first=2b056a054081e6d8494f28157bfd92fe#rd
---

点击上方【蓝色】字体   关注我们

# 01 场景描述

近期公司开发某项学习功能，改功能有很多学习内容（如java,C,python等方向），每天都会有众多学习用户学习某一项或者多项学习内容。产生数据如下表：

产生数据如下表：

```
日期          内容    学习用户2022-01-01    java      u12022-01-02    java      u12022-01-02    java      u22022-01-01    C         u12022-01-01    C         u32022-01-01    Python    u42022-01-02    Python    u42022-01-02    Python    u52022-01-02    Python    u6
```

期望数据

现在想要计算截止每天每个学习内容的截止去重学习用户数，但是截止去重用户数小于等于1的要被过滤，期望数据如下：

```
日期          内容    去重截止学习用户数2022-01-02    java    22022-01-01    C       22022-01-02    Python  3
```

截止到2022-01-01，学习内容java的为去重用户数为1，学习内容C的为去重用户数为2，学习内容Python的为去重用户数为1。所以2022-01-01学习内容为java和python的都要内过滤。

# 02 数据准备

```
create table study as (-- 基础数据    select '2022-01-01' as pdate, 'java' as icate, 'u1' as user_id    union all    select '2022-01-02' as pdate, 'java' as icate, 'u1' as user_id    union all    select '2022-01-02' as pdate, 'java' as icate, 'u2' as user_id    union all    select '2022-01-01' as pdate, 'C' as icate, 'u1' as user_id    union all    select '2022-01-01' as pdate, 'C' as icate, 'u3' as user_id    union all    select '2022-01-01' as pdate, 'Python' as icate, 'u4' as user_id    union all    select '2022-01-02' as pdate, 'Python' as icate, 'u4' as user_id    union all    select '2022-01-02' as pdate, 'Python' as icate, 'u5' as user_id    union all    select '2022-01-02' as pdate, 'Python' as icate, 'u6' as user_id);
```

# 03 问题分析

方法1：重复值状态标记

由于要按照内容及学习用户去重，且实现的是按照时间累计去重，因此基本的思路就是按照内容及用户分组后在最早出现时间点出标记为1，其余标记为0，以此来实现累加计算。

第一步：按照用户及学习内容分组后，找出组内最早时间，并进行状态标记

```
select pdate      ,icate      ,user_id      ,case when row_number() over(partition by icate,user_id order by pdate) = 1 then 1 else 0 end statusfrom study
```

第二步：按照日期和内容分组，对步骤1的状态进行汇总，并按照日期累计当前汇总后的状态值。具体SQL如下：

```
select pdate     , icate     , sum(sum(status)) over (partition by icate order by pdate) acc_user_cntfrom (select pdate           , icate           , user_id           , case when row_number() over (partition by icate,user_id order by pdate) = 1 then 1 else 0 end status      from study) tgroup by pdate       , icate
```

第三步：过滤掉累计值小于1的数据。最终SQL如下：

```
select pdate     , icate     , acc_user_cntfrom (select pdate           , icate           , sum(sum(status)) over (partition by icate order by pdate) acc_user_cnt      from (select pdate                 , icate                 , user_id                 , case when row_number() over (partition by icate,user_id order by pdate) = 1 then 1 else 0 end status            from study) t      group by pdate             , icate) twhere acc_user_cnt > 1
```

方法2：利用自关联生成完成数据进行行比较求解

完整的SQL如下：

```
with tmp1 as (select distinct pdate, icate              from study)select a.pdate,       a.icate,       count(distinct a.user_id) as icountfrom study a         join     tmp1 b     on         a.pdate <= b.pdate             and         a.icate = b.icategroup by a.pdate, a.icatehaving icount > 1;
```

# 04  小 结

本文给出了计算指标按照日期字段进行累计去重的优雅解法，文中采用了2种解法，一种借助于分析函数作为辅助变量生成状态标记求解，一种利用自关联生成全量数据进行行行比较求解，两种方式都非常巧妙。

与本文相关的问题：

[SQL进阶技巧：如何实现多指标累计去重？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485252&idx=1&sn=0b035151e5501449d972591a5ee681b1&chksm=e8e21364df959a72bba0736babd4a8d4e359f34d16af3adcc344872e253019b058dd662eac61&scene=21#wechat_redirect)

往期精彩：

[SQL进阶技巧：最近有效的缺失值填充问题【last\_value实现版】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484615&idx=1&sn=e58a3d71f0d2addec5f86f2148703d85&chksm=e8e210e7df9599f1e1f55c737ff67edc5c1ccfc4e878fd6099b2a9227caa2edb63e9d9cc41cd&scene=21#wechat_redirect)

[SQL进阶技巧：如何统计数组中非0元素的个数？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484654&idx=1&sn=32a6a8bc395a62af6815f1763bc5bf88&chksm=e8e210cedf9599d874d608e99bffedbfc2d0bbd149f246e8d6a97a58422a506faabc5ef7a309&scene=21#wechat_redirect)

[SQL进阶技巧：如何获取稀疏表字段中最新的值所对应的其他字段值](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484904&idx=1&sn=cac41f19f0a0a55dd14131eda8dcbbf5&chksm=e8e211c8df9598deb04913acf5af1d0b68cd1c17fc0a5b7f8f1b5dc7fe30022c210e6e621615&scene=21#wechat_redirect)

[SQL进阶技巧：统计各时段观看直播的人数？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247486804&idx=1&sn=ad060427e08469a5dc66b3d0b33a4e91&chksm=e8e21974df959062aa708ebcec8a50553fa48b3124773373146b68044649275f2babc681be83&scene=21#wechat_redirect)

[SQL进阶技巧：影院相邻的座位如何预定？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247486563&idx=1&sn=73c48d6b9d360ea1206a741aa7af7daa&chksm=e8e21843df9591559a7eeb227636d31d9202eb3edc0dd74df454348ef170342ea0c63caeca32&scene=21#wechat_redirect)

[数据特征工程：如何计算块熵？| 基于SQL实现](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485499&idx=1&sn=2cb58437a2746a05640067623ef7506c&chksm=e8e21c1bdf95950d869bd2ae530bb77cb80275f26c4ff699f6ee72a093e20b82045be80a6fd5&scene=21#wechat_redirect)

######

###### 会飞的一十六

扫描右侧二维码关注我们

点个【在看】 你最好看
---
title: 图解 SQL 执行顺序，三连暴击  ！
author: java1234
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIxNTAwNjA4OQ==&mid=2247542555&idx=1&sn=917c02d375b23613cbefedccb8e9af05&chksm=979c89fda0eb00ebced9d26462ca9f41cf045a2500fc0bbcbccf2493e022c2b58321d784fb18&mpshare=1&scene=24&srcid=0909kSXphjwVNvmg7O3PWGKN&sharer_sharetime=1662703940113&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

**大家好，我是锋哥，分享一篇SQL执行顺序文章 ！**

这是一条标准的查询语句:

这是我们实际上SQL执行顺序：

* 我们先执行from,join来确定表之间的连接关系，得到初步的数据
* where对数据进行普通的初步的筛选
* group by 分组
* 各组分别执行having中的普通筛选或者聚合函数筛选。
* 然后把再根据我们要的数据进行select，可以是普通字段查询也可以是获取聚合函数的查询结果，如果是集合函数，select的查询结果会新增一条字段
* 将查询结果去重distinct
* 最后合并各组的查询结果，按照order by的条件进行排序

### 数据的关联过程

数据库中的两张表

#### from&join&where

用于确定我们要查询的表的范围，涉及哪些表。另外，最新最全的 MySQL 面试题整理好了，微信搜索Java面试库小程序在线刷题。

选择一张表，然后用join连接

```
from table1 join table2 on table1.id=table2.id
```

选择多张表，用where做关联条件

```
from table1,table2 where table1.id=table2.id
```

我们会得到满足关联条件的两张表的数据，不加关联条件会出现笛卡尔积。

#### group by

按照我们的分组条件，将数据进行分组，但是不会筛选数据。

比如我们按照即id的奇偶分组

#### having&where

having中可以是普通条件的筛选，也能是聚合函数。而where只能是普通函数，一般情况下，有having可以不写where，把where的筛选放在having里，SQL语句看上去更丝滑。

**使用where再group by**

先把不满足where条件的数据删除，再去分组

**使用group by再having**

先分组再删除不满足having条件的数据，这两种方法有区别吗，几乎没有！

举个例子：

> 100/2=50，此时我们把100拆分`(10+10+10+10+10…)/2=5+5+5+…+5=50`,只要筛选条件没变，即便是分组了也得满足筛选条件，所以where后group by 和group by再having是不影响结果的！

不同的是，having语法支持聚合函数,其实having的意思就是针对每组的条件进行筛选。我们之前看到了普通的筛选条件是不影响的，但是having还支持聚合函数，这是where无法实现的。

当前数据分组情况

执行having的筛选条件，可以使用聚合函数。筛选掉工资小于各组平均工资的`having salary<avg(salary)`

#### select

分组结束之后，我们再执行select语句，因为聚合函数是依赖于分组的，聚合函数会单独新增一个查询出来的字段，这里用紫色表示，这里我们两个id重复了，我们就保留一个id，重复字段名需要指向来自哪张表，否则会出现唯一性问题。最后按照用户名去重。

```
select employee.id,distinct name,salary, avg(salary)
```

将各组having之后的数据再合并数据。

#### order by

最后我们执行order by 将数据按照一定顺序排序，比如这里按照id排序。如果此时有limit那么查询到相应的我们需要的记录数时，就不继续往下查了。

#### limit

记住limit是最后查询的，为什么呢？假如我们要查询年级最小的三个数据，如果在排序之前就截取到3个数据。实际上查询出来的不是最小的三个数据而是前三个数据了，记住这一点。

我们如果limit 0,3窃取前三个数据再排序，实际上最少工资的是2000,3000,4000。你这里只能是4000,5000,8000了。

原文链接：https://blog.csdn.net/weixin\_44141495/article/details/108744720

End

```
```
```
```
锋哥的 SpringSecurity+Vue权限系统 震撼发布！...

安排一个福利，Java全栈就业实战课程 免费哦...

 66套Java实战项目课程领取...
```
```

```
```
2022年粉丝福利

http://download.java1234.com/

每月送 666 套Java海量资源网站 VIP会员，供大伙一起学Java

如果没加过锋哥微信的

加一下锋哥微信备注 VIP 即可开通

👇👇👇

👆长按上方二维码2秒,备注vip
```
```
```

锋哥，10年Java老司机，小锋网络科技 光杠司令员，司令部：www.java1234.vip 每天坚持锻炼身体，坚持早睡早起，崇尚自由，平时喜欢带带Java学员 (已经成功指导1000+学员高薪就业)，喜欢搞搞Java技术自媒体，搞搞小产品，后期继续研究主流技术，以及进军短视频+直播领域，每天进步一点，奥利给。
```
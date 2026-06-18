---
title: SQL进阶技巧：如何利用距离最近的一条记录补全空值【稀疏表补全法】
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484593&idx=1&sn=f528cba45aefc284cbc39ee567319c00&chksm=e971c223de85011d0808b639602662a3fc25a9954a0675f2e390b335b4f8ea935298cc499455&mpshare=1&scene=24&srcid=0920ih10sAXZy9t6KLyR2Yd0&sharer_shareinfo=66b8a3e5f31ffd52073eacb065ee154d&sharer_shareinfo_first=66b8a3e5f31ffd52073eacb065ee154d#rd
---

**“**

本文给出了一种利用当前最新数据补全稀疏表格的方法，该方法常常用于数据清洗当中，比如用户一个session中发生浏览点击事件时候，url往往在浏览事件时候给出而点击事件中往往没有给出，这种表格往往是稀疏的，如果此时想知道用户点击了某个按钮后当前页面是哪个，那么我们往往就需要利用这种方法补全数据来获取当前点击事件所对应的页面。

。**”**

01

—

需求描述

表名：t

表字段及内容：

```
date_id a b c2014 AB 12 bc2015 232016 d2017 BC
```

问题：如何使用最新数据补全表格

输出结果如下所示：

```
date_id a b c2014 AB 12 bc2015 AB 23 bc2016 AB 23 d2017 BC 23 d
```

应用场景：补全稀疏表格，获取用户点击事件时所对应的当前页面【URL】

02

—

数据准备

```
create table t asselect '2014' as date_id,'AB' as a,'12' as b,'bc' as cUNION ALLselect '2015' as date_id,null as a,'23' as b,null as cUNION ALLselect '2016' as date_id,null as a,null as b,'d' as cUNION ALLselect '2017' as date_id,'BC' as a,null as b,null as c
```

02

—

数据分析

本文所讲的问题在之前一篇文章页分析过，主要是利用断点重组的思想，具体链接如下

[SQL进阶技巧：数据清洗如何分析商品入库采购成本数据缺失问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484495&idx=1&sn=af4f5ce3f3f2ad580d4cc85b3777c0a7&chksm=e8e2106fdf959979d7c49120f757e0729af7e54b90c71980536087c6860d8a5dd250df3aa994&scene=21#wechat_redirect)

此篇文章主要是利用：

sum() over(order by xxx) 

来构建分组 。本文根据数据的结构特点，抓取其特征（“找规律，抓特征”）采用：

**count（）over(order by xxxx)**

来构建分组。

分析：

（1） count(列名)会忽略null值进行计数

（2） 当当前列中如果有空值，那么在进行count()累计统计时候，有空值的行的累计条数与最近不为空值的累计条数是一致的。

因此我们可以基于上述两条规则的认知快速得到新的分组，具体SQL 如下：

```
selectdate_id,max(a) over(partition by grp1) as a,max(b) over(partition by grp2) as b,max(c) over(partition by grp3) as cfrom(selectdate_id,a,b,c,count(a) over(order by date_id) as grp1,count(b) over(order by date_id) as grp2,count(c) over(order by date_id) as grp3from t)t;
```

可以看到通过分析数据特征，利用**count（）over(order by xxxx) 这种方式构建新的分组条件要优于sum() over(order by xxx) 这种形式，避免了case when寻找断点的步骤。**

03

—

小结

本文本质上还是断点重组的思想应用，只不过在构建分组时，利用数据的特征快速找到分组的条件解决问题，利用count() over(order by xxx)也是一种技巧，需要积累和掌握。

**往期精彩：**

[SQL进阶技巧：有序数据合并问题之如何按照时间顺序对数据进行合并？【腾讯互娱-分析某用户玩游戏的先后顺序链条】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484532&idx=1&sn=f3edbd1c417c82322dda06040aa9f104&chksm=e8e21054df95994226c941ccf7e936cdddc55761c63a781d4a949f322ee57f39beeaa57a4e93&scene=21#wechat_redirect)

[SQL很简单，可你却总是写不好？每天一点点，收获不止一点点。](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484518&idx=1&sn=0f1e013454fbb3abb801536464d30ee1&chksm=e8e21046df959950437960df3bfc1e7913e014f39c0c192ee84f7dfabc68fc8c4fb2da4888aa&scene=21#wechat_redirect)

[数仓建模：员工主题之部门人员在职情况统计分析【拉链表模型统计分析】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484512&idx=1&sn=6afd79944c62bc3a96129ff6d1729f9e&chksm=e8e21040df9599563016c09a21936739f3c6d9e1d2bc3db1160fd4822d2c39b19d7cc033dd9e&scene=21#wechat_redirect)
---
title: SQL进阶技巧：如何提取JSON中 key值？
author: 会飞的一十六
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485151&idx=1&sn=2a6065c213fc19bf4a3d3ffb12150fbb&chksm=e93f27983bba8bdee626ff09f8de3200c8f4e9a898f4dedb84fafabb43b9d92e622ed24954c5&mpshare=1&scene=24&srcid=1105gLH58X0ZBtFU2Ffzc2NX&sharer_shareinfo=d525fb985674b695a366ea4615daa843&sharer_shareinfo_first=d525fb985674b695a366ea4615daa843#rd
---

01

—

问题描述

如下json\_str

| json\_str |
| --- |
| [{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}] |

我想获取该json中的每一个key值，如何做？

02

—

问题解决

（1）先将json\_str中的[]及{}去掉

利用translate()函数处理

```
select translate('[{"website":"baidu.com","name":"百度"} ,{"website":"google.com","name":"谷歌"}]'                  ,'[]{}""','')
```

结果如下：

```
OKwebsite:baidu.com,name:百度,website:google.com,name:谷歌Time taken: 1.335 seconds, Fetched: 1 row(s)
```

（2）将逗号替换成冒号:

```
select regexp_replace(translate('[{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}]'                  ,'[]{}""','') ,'\,','\:')
```

 计算结果如下：

```
OKwebsite:baidu.com:name:百度:website:google.com:name:谷歌Time taken: 0.165 seconds, Fetched: 1 row(s)
```

 （3）将步骤2计算的结果用posexplode()函数展开，获取索引值及具体值

```
select pos+1 as rn      ,valfrom(    select regexp_replace(translate('[{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}]'                      ,'[]{}""','') ,'\,','\:') as str) t1 lateral view posexplode(split(str,':')) t2 as pos,val
```

计算结果如下：

```
OK1  website2  baidu.com3  name4  百度5  website6  google.com7  name8  谷歌Time taken: 0.224 seconds, Fetched: 8 row(s)
```

（4）由于K-V是一组对偶元组，因此K为奇数行，我们只需要取出奇数行的记录即可

```
select rn      ,val as keyfrom(    select pos+1 as rn          ,val    from(        select regexp_replace(translate('[{"website":"baidu.com","name":"百度"},{"website":"google.com","name":"谷歌"}]'                          ,'[]{}""','') ,'\,','\:') as str    ) t1 lateral view posexplode(split(str,':')) t2 as pos,val) m where rn%2=1
```

计算结果如下：

```
OK1  website3  name5  website7  nameTime taken: 0.294 seconds, Fetched: 4 row(s)
```

03

—

问题解决

本文给出了一种通过HQL提取JSON中 key值的方法和技巧，主要使用的知识点如下：

* （1）字符替换函数：translate()函数
* （2）字符串替换函数：regexp\_replace()函数
* (3) 列转行：lateral view posexplode()函数
* （4）获取奇数行记录：mod(rn,2)=1

**往期精彩**

[SQL进阶技巧：如何获取数组中前N个元素？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485117&idx=1&sn=ade5034e0eba522ed0c3b3b1910caffa&chksm=e8e2129ddf959b8b2e2258eb7dd853429ac5640254460a5734edb9c8feeb7f4f7afcbaaa8029&scene=21#wechat_redirect)

[Hive中如何生成时间维度表？|   Hive时间函数全掌握](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247485068&idx=1&sn=f75a146fe54e8e4e936b33032692f8ee&chksm=e8e212acdf959bba25807ac6eb2ed4f7f27c50872abfeffec4bb7246508f16287e0a23dbaddd&scene=21#wechat_redirect)

[SQL进阶技巧：经典问题题-换座位](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484995&idx=1&sn=acfd7c9d8e26584e7151047f5e986ae3&chksm=e8e21263df959b754ad23acab54b7c46681cb59653424f263d4a764661f606a3edb58d654340&scene=21#wechat_redirect)

[SQL进阶技巧：如何分析截止当前学生退费总人数问题？| 存在计数问题分析](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484964&idx=1&sn=7ac1569ca0f76ebfc320726500621d73&chksm=e8e21204df959b12cefd544154434ee19863d7cbbf9a6331727b8488bc6680bab98c8250cc57&scene=21#wechat_redirect)

[SQL进阶技巧：如何计算先进先出库龄问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484330&idx=1&sn=ee9b76bde4035d1a8cca8d9a803112a0&chksm=e8e2178adf959e9cd0803de7d2c6a152d37daeebd8be3d27b1447828c3cba3a5060e02c4fb3d&scene=21#wechat_redirect)

[SQL进阶技巧：如何计算重叠区间合并问题？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484320&idx=1&sn=9141f312c268f73a16315347707d1a97&chksm=e8e21780df959e96e06e9a49b721e8fab8bbd086da6f87ea8815302782ebe096d8f45b533fe3&scene=21#wechat_redirect)
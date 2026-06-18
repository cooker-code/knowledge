---
title: 讲懂高频Hive：时间日期函数
author: 数据攻略
date:
url: http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500591&idx=1&sn=39067090f593312c1714c1d60f1305fb&chksm=eb5d45dd9af114ff0c831fab67812dc8fbb037f5be2b0f158791550948c75f285d8e9c642c4c&mpshare=1&scene=24&srcid=1219KtuvxmOGyiBIBXHgZgKG&sharer_shareinfo=b41fd300d760ae5d632c57158721dbf6&sharer_shareinfo_first=b41fd300d760ae5d632c57158721dbf6#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030202_Hive/030202_知识地图|知识地图]]


点击上方蓝色『**数据攻略**』关注+星标~

数据分析求职干货不错过

哈喽大家好，我是数据攻略的六哥~
继上篇咱们讲过Hive高频函数中的
窗口函数、字符处理函数、以及
行列转换函数后，有读者留言：
能否讲讲其他常用函数

所以针对公众号的『工具&语言』板块
我来继续盘一盘日常工作中那些
常用、易错、易忘的函数
可以作为日常工作中的备忘知识手册
忘记了来翻一翻，立马唤醒使用姿势

上几篇系列分享可戳👇 

* [讲懂高频Hive：窗口函数（一）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247487817&idx=1&sn=778a1c7043864339a9138c7513ad1c53&chksm=eaed85a4dd9a0cb2d95fa16ddd6c1959b8a2b1c2ac7bd230aeb2992d96c1f81282788ce28071&scene=21#wechat_redirect)
* [讲懂高频Hive：窗口函数（二）](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247499857&idx=1&sn=102cd4d396bb5ffcb5cab2b3e12471b6&scene=21#wechat_redirect)
* [讲懂高频Hive：字符处理函数](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247498075&idx=1&sn=0a23ce4c79e445e750873fdd193bc1f5&chksm=eaee6db6dd99e4a0d23dc3a8199752f00988d70fb526610acb6ffbda77f5b5bbaf1584ad1888&scene=21#wechat_redirect)
* [讲懂高频Hive：行列转换函数](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247498183&idx=1&sn=6714c7f4696ef0c8b625a7ef428b8f60&scene=21#wechat_redirect)



本篇，是工作中又一大高频使用函数：
—— 时间日期函数
主要结合实际工作中，总结出：
常用到的函数有啥？
哪些分类？区别是啥？
内含 使用注意事项+实例
（注：含例题实操，可收藏慢看⭐）

------正文手动分割线------

**本文结构速览：**

一、常用情景举例

二、常用函数定义

三、一个case测试

# 一、常用情景举例

在数据处理和分析过程中，时间日期函数发挥着重要作用。通常在实际工作中的应用场景也非常丰富，例如：

* 用户画像中做基础标签时，需要根据用户出生日期计算年龄
* 流量监控时，根据不同用户类型需要计算不同口径留存率
* 物流场景中，希望计算订单从下单到发货的时间差，以便业务优化发货时效
* 业务年度复盘时，希望分析各月份、每周的销售额达成情况
* 等等...



如果我们按照操作方式来分类
高频使用到的不外乎是以下几种：
涉及运算类、提取类、转换类
具体可见下图：（具体定义可见下部分） 





# 二、常用函数定义

本部分我们针对常用到的函数做：
语法定义、案例、注意事项讲解

## 运算操作

▌ 增减天数：DATE\_ADD/DATE\_SUB

▼ 含义：按照delta幅度增减startdate日期的天数

▼ 语法&案例：

```
DATE_ADD(<startdate>, <delta>)

##参数说明：
#startdate：必填，起始日期值。支持DATE、DATETIME或STRING类型。
#delta：必填，修改幅度。BIGINT类型。>0则增，<0，则减

##返回值说明:
#返回DATE类型，格式为yyyy-mm-dd

##举例：
select date_add('2014-03-18 00:00:00', 2)
--返回2014-03-20。
```



▼ 注意事项：

* 仅支持以日为单位增减
* delta值为NULL时，返回NULL
* 与date\_add的增减逻辑相反，记住一个即可




▌天数间隔：DATEDIFF

▼ 含义：计算两个日期的差值并按照指定的单位表示

▼ 语法&案例：

```
datediff(<date1>, <date2>, <datepart>)

##参数说明：
#date1、date2：必填。分别为被减数和减数，需为DATE、DATETIME或TIMESTAMP类型
#delta：选填，默认日期格式为天。

##返回值说明:
#返回BIGINT类型。

##举例：
select DATEDIFF('2024-11-29' ,'2024-10-29','dd')
--返回 31。
```



▼ 注意事项：

* 如果date1小于date2，返回值为负数
* datepart值可不填，如填为NULL时，返回NULL

▌ 增减月度：ADD\_MONTHS

▼ 含义：计算日期值增加指定月数后的日期

▼ 语法&案例：

```
add_months(<startdate>, <num_months>)

##参数说明：
#startdate：必填。DATE、DATETIME、TIMESTAMP或STRING类型
#num_months：必填。INT型数值。

##返回值说明:
#返回STRING类型的日期值，格式为yyyy-mm-dd。

##举例：
select add_months('2024-02-14',3)
--返回2024-05-14。
```




## 提取操作

▌ 指定日期

* NOW()：返回当前系统日期与时间,格式为yyyy-mm-dd hh:mi:ss.SSS
* GETDATE()：获取当前系统时间
* LAST\_DAY()：返回日期值所在月份的最后一天日期，返回STRING类型的日期值，格式为yyyy-mm-dd
* LASTDAY()：获取日期所在月的最后一天，和上面的区别是返回DATETIME类型，格式为yyyy-mm-dd hh:mi:ss

```
##举例：
select last_day('2024-03-04');
--返回2024-03-31。

select lastday('2024-11-29 00:01:00')
--返回2024-11-30 00:00:00 。
```




▌ 指定单位

DATEPART()

▼ 含义：提取日期date中符合指定时间单位datepart的值

▼ 语法&案例：

```
datepart(<date>, <datepart>)

##参数说明：
#date：必填。DATE、DATETIME或TIMESTAMP类型
#datepart：必填。STRING类型常量，支持扩展的日期格式。

##返回值说明:
#返回BIGINT类型

##举例：
selectdatepart(datetime'2024-06-18 01:10:00', 'yyyy');
--返回2024。

selectdatepart(datetime'2024-06-18 01:10:00', 'mm');
--返回6。
```




DAY/MONTH/YEAR

▼ 含义：

* day()：返回一个日期的天
* month()：返回一个日期的月份
* year()：返回日期date的年

▼ 语法&案例：

```
day(<date>)
month(<date>)
year(<date>)

##参数说明：
#date：必填。DATETIME、TIMESTAMP、DATE或STRING类型日期值，如格式为yyyy-mm-dd等

##返回值说明:
#返回INT类型

##举例：
selectday('2024-09-01');
--返回1。

selectmonth('2024-09-01');
--返回9。

selectyear('2024-09-01');
--返回9。
```




DAYOFWEEK/DAYOFMONTH/DAYOFYEAR/WEEKOFYEAR

▼ 含义：

* dayofweek()：返回日期的星期值，返回值的取值范围为1~7，与星期的对应关系为1=Sunday， 2=Monday，...，7=Saturday。
* dayofmonth()：返回日期日部分的值
* dayofyear()：返回日期是当年中的第几天
* weekofyear()：返回日期值位于当年的第几周

▼ 语法&案例：

```
dayofweek(<date>)
dayofmonth(<date>)
dayofyear(<date>)
weekofyear(datetime <date>)


##参数说明：
#date：必填。前三个DATETIME、TIMESTAMP、DATE或STRING类型日期值；后一个必须为DATETIME类型日期值。

##返回值说明:
#返回INT类型

##举例：
SELECTdayofweek('2014-12-19');
--返回5，即Thursday。

selectdayofmonth('2024-09-01');
--返回1。

selectdayofyear('2024-01-03');
--返回3。

selectweekofyear('2024-12-31');  
--返回1。虽然2024-12-31属于2024年，但是这一周的大多数日期是在2025年，因此返回结果为1，表示是2025年的第一周。
```



▼ 注意事项：

* weekofyear()这一周算上一年还是下一年，取决于这一周的大多数日期（4天以上）在哪一年。




▌ 指定格式

TO\_DATE/DATE\_FORMAT

▼ 含义：

* to\_date()：将指定格式的字符串转换为日期值。
* date\_format()：将日期值转换为指定格式的字符串。



▼ 语法&案例：

```
to_date(string <date>[, string <format>])

##参数说明：
#date：必填。STRING类型
#format：可选。STRING类型常量

##返回值说明:
#返回DATE或DATETIME类型
#当函数入参无format参数，返回DATE类型，格式为yyyy-mm-dd；
#当函数入参有format参数时，返回DATETIME类型，格式为yyyy-mm-dd hh:mi:ss。


date_format(<date>, string <format>)

##参数说明：
#date：必填。待转换的日期值。支持DATE、TIMESTAMP或STRING类型。
#format：必填。STRING类型常量。

##返回值说明:返回STRING类型。

##举例：
selectto_date('2024-09-24 13:39:34');
--返回2024-09-24，数据类型为Date。

selectdate_format('2024-11-29','yyyy/MM/dd'),
--返回2024/11/29
```




## 转换操作

▌ 转时间戳

UNIX\_TIMESTAMP

▼ 含义：将日期转换为整型的UNIX格式的日期值。



▼ 语法&案例：

```
unix_timestamp(<date>)

##参数说明：
#date：必填。DATETIME、DATE、TIMESTAMP或STRING类型日期值

##返回值说明:
#返回BIGINT类型

##举例：
SELECT unix_timestamp(DATETIME'2023-11-10 11:11:00'); 
--返回1699585860。
```




▌ 转日期值

FROM\_UNIXTIME

▼ 含义：将数字型的UNIX值转换为日期值。

▼ 语法&案例：

```
from_unixtime(bigint <unixtime>)

##参数说明：
#unixtime：必填。BIGINT类型，秒数，

##返回值说明:
#返回DATETIME类型，格式为yyyy-mm-dd hh:mi:ss。

##举例：
SELECT from_unixtime(1699585860); 
--返回2023-11-10 11:11:00。
```





# 三、一个case测试

## 问题背景


假设你现在负责支付宝会员业务，为提高用户粘性，产品上线了新一版签到领积分兑换的功能，现希望了解新功能的用户使用情况，希望得知连续签到3天及以上的用户ID和对应符合条件的连续签到起始日期？  


**表描述如下：**
用户签到日志表：user\_sign\_log 第一列是用户id，第二列是用户签到日期

| user\_id | sign\_in\_time |
| --- | --- |
| 1001 | 2024-01-01 05:19:20 |
| 1002 | 2024-01-01 13:49:40 |
| 1003 | 2024-01-01 15:19:10 |
| 1002 | 2024-01-02 14:29:50 |
| 1001 | 2024-01-03 13:29:30 |
| 1002 | 2024-01-03 17:19:20 |

说明：签到功能一天仅需要签到一次



**输出表结构如下：**

* user\_id
* start\_dt
* num\_days




## 参考代码

* 步骤0：以防万一先对日志GROUPBY去重，数据粒度转为签到日期sign\_in\_dt+用户id
* 步骤1：利用ROW\_NUMBER() 构造新等差递增列b，做排序：按照用户id作为分组，以签到日期升序排序
* 步骤2：利用DATE\_SUB原签到日期与新递增列b作差，得到差值日期delta\_group（注意，这里如果连续，得到的就是连续签到的最早日期前一日）
* 步骤3：利用delta\_group为分组标志，做COUNT计数分组，筛选出连续签到3天及以上的用户和对应连续签到起始日期



```
--步骤3：计数分组，筛选出连续签到3天及以上的用户和对应连续签到起始日期
SELECT  user_id
       ,DATE_ADD(delta_group,1) AS start_sign_dt
       ,COUNT(1)                AS continous_days
FROM
( --步骤2：原签到日期与新递增列b作差，得到差值日期（注意，这里如果连续，得到的就是连续签到的最早日期前一日） 
SELECT  user_id
        ,sign_in_dt
        ,b
        ,DATE_SUB(sign_in_dt,binint(b)) AS delta_group
FROM
 ( --步骤1：构造新等差递增列b，做排序：按照用户id作为分组，以签到日期升序排序 
SELECT  *
         ,ROW_NUMBER() OVER (PARTITIONBY user_id ORDERBY  sign_in_dt) AS b
FROM
  ( --步骤0：以防万一先对日志去重，数据粒度为签到日期sign_in_dt+用户id 
   SELECT  user_id
          ,TO_DATE(sign_in_time) AS sign_in_dt
   FROM user_sign_log
   GROUPBY  user_id
            ,TO_DATE(sign_in_time)
  )t1
 )t2
)t3
GROUPBY  user_id
         ,DATE_ADD(delta_group,1)
HAVINGCOUNT(1) >= 3
```

**往期**『SQL』专题文章**可戳**👇

* [有关『SQL』有哪些考法？该如何备战？](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247497666&idx=1&sn=7a5679e2569cf9eb320076668baf20bf&chksm=eaee632fdd99ea39d51f57221ca856a34c1ffd40939737312365344301f1077a7cdd18c06691&scene=21#wechat_redirect)
* [SQL出题技巧及大厂母题（附答案）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247485647&idx=1&sn=b3a7d4ebde2c1a962d4125c3ad1d9362&chksm=eaed9c22dd9a15347c2c3f1465fb478857d437f117eae77e90f95d58908bfc34bdf49a31aabc&scene=21#wechat_redirect)
* [【数据分析岗】SQL类高频考点归纳](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247483985&idx=1&sn=64eae14b9814725a8fb6e8f67063c041&chksm=eaed96bcdd9a1faa7762b3faaf435d1ef73abba5097596b30e3504c1e84c25e47f32388319bd&scene=21#wechat_redirect)
* [高频笔面试考点『留存』你会了吗？](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247497878&idx=1&sn=4d7f7e125fa5ab2d6d5d06bb3a27aff8&chksm=eaee6c7bdd99e56d8e5d63ec33b78d8adc01f10eaff8bc1bdc59c51d9b0fdb3dd38bdde90d41&scene=21#wechat_redirect)
* [【SQL实战】淘宝营销活动分析](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247490200&idx=1&sn=77f38bea684400aa61be8a5a14e4413a&chksm=eaed8e75dd9a07632b78feecf18f1581835307813bac1d58fb660c3cc3b1aabf826650525e26&scene=21#wechat_redirect)
* [『SQL实战』图解经典考题-最大在线人数](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247497102&idx=1&sn=90122924bea910e19a64df83cfbb749f&chksm=eaee6163dd99e875d23a1f524228219f006170c0ce88f730cd9f107f9d031f9f9b46a2761d73&scene=21#wechat_redirect)
* [『SQL实战』高频考题之复购问题，坑点居然这么多！](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500425&idx=1&sn=a747a98a29195e15173d11ffaa65a6b7&scene=21#wechat_redirect)
* [『SQL实战』高频考题之连续问题，2种解法全解析！](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500406&idx=1&sn=42552e59cd9e349cd5bf13bed04cc7f0&scene=21#wechat_redirect)

以上就是关于sql中
常用时间日期函数的介绍。
还有些hive其他常用/易考内置函数
本文点在看过20，咱们下篇见😎🍻~

如若盼 追更 **『日常学习』**干货系列
欢迎大家**点赞**、**转发**，最底部点**点在看**
你的鼓励，真的是对我最大的动力



如需更多面试官总结的含业务背景、指标
分行业、分易错考点的高频考题解析归纳
欢迎私聊六哥~**（微信：data-youdao）**
做一题，会一类，高效备战🚀

Ps.年末跳槽节点，如需六哥求职相关帮助
可戳此了解[👉](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500249&idx=1&sn=acdc773d106c9c11010755d6bb88906e&chksm=eaee5534dd99dc22d64ea48e6c062fba5429e6720b52378e5864967f16b2793d05708180046f&scene=21#wechat_redirect)[六哥的原创课程/求职服务说明](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500249&idx=1&sn=acdc773d106c9c11010755d6bb88906e&chksm=eaee5534dd99dc22d64ea48e6c062fba5429e6720b52378e5864967f16b2793d05708180046f&scene=21#wechat_redirect)

Ps.也欢迎添加我的微信（data-youdao）
交个朋友先，不定期有一手内推资源传送 ~

更多 『求职干货』 & 『日常学习』 系列好文，等你发现~

往期好文推荐

**『求职类』**

[『快手』数据分析岗面试真题+解析（下）](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500487&idx=1&sn=6785a812a0e97093b50d5b1f990d49e1&scene=21#wechat_redirect)

[『抖音电商』数据分析岗面试真题+解析（下）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500446&idx=1&sn=9aed38bc080a459ed53a64d1c5565a87&chksm=eaee5673dd99df654acc3a6bbbf229e8dcfa665b37ca58bdb3c5eccbbadcd36f70af60d1c826&scene=21#wechat_redirect)

[56道AB实验高频面试题 | 重置答案解析（一）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247499189&idx=1&sn=3ec07ffe4a882fe73a6479fdca90c022&chksm=eaee6958dd99e04e6088044d9bc8e455a271f37630df013a637fc4b530eafd8f136d19e026f2&scene=21#wechat_redirect)

[AB实验中这类指标如何计算显著性？| AB系列（八）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500196&idx=1&sn=6e3f5a76ae9b6211d61ad6b282b9e3ad&chksm=eaee5549dd99dc5f18a61b5b0bc815a9c26b25e8edade6322dcd9247b979867837ff116b63ce&scene=21#wechat_redirect)

[『SQL实战』高频考题之复购问题，坑点居然这么多！](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500425&idx=1&sn=a747a98a29195e15173d11ffaa65a6b7&chksm=eaee5664dd99df7269f644cd4441dc8021fae9b812f5cebbce12342ca3951a9a39c39dc6aa6e&scene=21#wechat_redirect)

**[【数据分析岗】字节面试真题（含答案）+送100道面试题库](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247486373&idx=1&sn=8ac1dafdc2d7daa47824b715f69a82e1&chksm=eaed9f48dd9a165eada4019baea497aa026716e4b7030081cd4bb8fef3f1dd540efc9a1792be&scene=21#wechat_redirect)**

****『日常学习类』****

[『指标异动』贡献度实操计算中，5个常见QA!](https://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247500505&idx=1&sn=84b52c8b45193b6c33b49f5a0c7c0d26&scene=21#wechat_redirect)

[『指标异动』你真的理解吗？](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247490799&idx=1&sn=1ef94d3208b3d963e47dd8e362ba8ced&chksm=eaed8802dd9a0114707a6ad542a643faa34b7d542f111c46f8cdc3f4f3930bbd6fc51a47505f&scene=21#wechat_redirect)

**[『指标异动』贡献度定量归因之法，带你知因又知果!](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247492204&idx=1&sn=76cccb063bac1f6084e40062356d102b&chksm=eaee7681dd99ff97a7ce5b3620f6fc6667496f7230ab62579c1fd683a4433d58b59074859141&scene=21#wechat_redirect)**

****[2种方法快速分析群体差异（附case）!](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247493037&idx=1&sn=fe1f6be40bb1d89c8b0935a48392fdde&chksm=eaee7140dd99f856fadfd019068f119b0e0397b911bf54d46e8f9dd27d5cb330d891660c0513&scene=21#wechat_redirect)****

[讲懂高频Hive：窗口函数（二）](http://mp.weixin.qq.com/s?__biz=MzI2ODYxNzA5MA==&mid=2247499857&idx=1&sn=102cd4d396bb5ffcb5cab2b3e12471b6&chksm=eaee54bcdd99ddaa3a2393a3da3a2401015ac5b9f1fb48485a16923cbee856098a31ef2cd102&scene=21#wechat_redirect)
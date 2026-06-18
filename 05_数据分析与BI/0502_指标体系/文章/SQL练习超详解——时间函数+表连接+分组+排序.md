---
title: SQL练习超详解——时间函数+表连接+分组+排序
author: 守住内心的宁静
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485697&idx=1&sn=0db8d34885b044e81822c798bfa20ea8&chksm=c10f7b7c3aa4c0ab53677df84eab54e5fdbe8780f10b9bf8c3bb321e56c5710bb49f977ff478&mpshare=1&scene=24&srcid=0808GclPFEChUde3TzC6IIHz&sharer_shareinfo=5553718aa5ec164dadfc928a1de3a88e&sharer_shareinfo_first=5553718aa5ec164dadfc928a1de3a88e#rd
---

Hello，今天分享的题目综合考察SQL语句中的时间函数、表连接、分组排序等知识点。

| 题目

某公司员工信息数据及单日出勤信息数据如下：

员工信息表staff\_tb

(staff\_id-员工id,staff\_name-员工姓名,staff\_gender-员工性别,post-员工岗位类别,department-员工所在部门)，如下所示：

出勤信息表attendent\_tb

(info\_id-信息id,staff\_id-员工id,first\_clockin-上班打卡时间,last\_clockin-下班打卡时间)，如下所示：

问题：请统计该公司各岗位员工平均工作时长？要求输出：员工岗位类别、平均工作时长(以小时为单位输出并保留三位小数)，按照平均工作时长降序排序。

注：如员工未打卡该字段数据会存储为NULL，那么不计入在内。

### | 输出

### 示例数据结果如下：

### 解释： Engineer类岗位有4、5、6共计3名员工，工作时长分别为：9.500、9.167、10.250，则平均工作时长为(9.500+9.167+10.250)/3=9.639小时。其他结果同理..... | 详细解析

**数据关联**

* 使用内连接(inner join)将员工信息表(staff\_tb)和出勤信息表(attendent\_tb)进行关联
* 关联条件是s.staff\_id = a.staff\_id，筛选出两个表都有的记录，即确保只统计有打卡记录的员工

计算工作时长

* timestampdiff(second, first\_clockin, last\_clockin): 计算上班打卡时间(first\_clockin)和下班打卡时间(last\_clockin)之间的秒数差，以确定每个员工单日的工作时长。
* 由于timestampdiff()函数结果只保留整数，以秒作为单位计算出结果之后再除以3600转化为小时单位，可以更精确地计算带有小数的时间差，优于直接以分钟/小时作为单位。

分组与聚合

* group by post: 以员工岗位类别进行分组
* avg(...): 计算每个岗位类别员工的平均工作时长
* round(..., 3): 将计算结果四舍五入保留3位小数

排序

* order by work\_hours desc：将统计结果按照平均工作时长(work\_hours)进行降序排列

逻辑顺序

①FROM和INNER JOIN：首先通过内连接将员工信息表和出勤信息表进行关联，NULL记录在此阶段已被排除(内连接特性)。

②GROUP BY：对连接后的结果按照员工岗位类别进行分组。

③聚合函数计算(AVG)：在每个分组内计算员工工作时长的平均值。

④SELECT子句：SELECT post 和 ROUND(...) 这部分定义了最终要输出的列，即岗位类别(post)和对应的平均工作时长。

⑤ORDER BY(结果排序)：按平均工作时长降序排列结果。

以上便是本题的全部分析，希望对您有帮助~

注：题目来源于牛客网，仅供学习交流，侵权删.

[SQL练习超详解——窗口函数+表连接+聚合函数...](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485687&idx=1&sn=5385485ab91677ea678cdfa36c6a275f&scene=21#wechat_redirect)

[SQL练习超详解——子查询+聚合函数+表连接](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485679&idx=1&sn=8bddb3a356216e87293819f6f676201f&scene=21#wechat_redirect)

[SQL练习超详解——聚合函数+字符串函数+条件判断](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485667&idx=1&sn=f79cc4109f4e764cc4d848dd388acee2&scene=21#wechat_redirect)

[SQL练习超详解——表连接+日期函数+排序](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485649&idx=1&sn=71d95b124ecf2177e270b4703d9c77ab&scene=21#wechat_redirect)

[SQL练习超详解——表连接+分组+聚合+排序](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485632&idx=1&sn=88056c01c2dc0bb495b64465c24684b3&scene=21#wechat_redirect)

[SQL练习超详解——窗口函数+子查询+表连接...](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485614&idx=1&sn=1d2fd9aca9089ccfef96ad7ea0aa90a8&scene=21#wechat_redirect)

[SQL练习超详解——窗口函数+日期函数+分组+聚合](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485597&idx=1&sn=9796285dbdf4135c623c1ccc62e6db54&scene=21#wechat_redirect)

[SQL练习超详解——日期函数+表连接+聚合函数](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485581&idx=1&sn=76663bcb72460d39322964f1548aa536&scene=21#wechat_redirect)

[SQL练习超详解——聚合函数+日期函数+字符串函数](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485569&idx=1&sn=7f00a03727ca73ea62f6ba264687e910&scene=21#wechat_redirect)

[SQL练习超详解——文本函数](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485548&idx=1&sn=fa4c53fef30b6ea0dbad97086e2a9fbd&scene=21#wechat_redirect)

[SQL练习超详解——窗口函数+子查询+排序](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485461&idx=1&sn=a6d0e0c442e7a991ac120d75b5fb0116&scene=21#wechat_redirect)

[SQL每日一题详解——计算每日累计利润(窗口函数)](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485440&idx=1&sn=04bd8133b515edae5c4f468348a36744&scene=21#wechat_redirect)

[波士顿房价预测案例——基于多种机器学习+SHAP可解释性](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485538&idx=1&sn=c3e98130f2b623d54524f8519ce03ae2&scene=21#wechat_redirect)

[SQL每日一题详解——浙大不同难度题目的正确率(表连接+where+group by+order by)](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485451&idx=1&sn=6d4ccc39a042aee276ac0930af5943c5&scene=21#wechat_redirect)

[SQL每日一题详解——基本数学函数](https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485426&idx=1&sn=23350f677448fdb51463e74802fb08cc&scene=21#wechat_redirect)

欢迎关注、留言、一起学习
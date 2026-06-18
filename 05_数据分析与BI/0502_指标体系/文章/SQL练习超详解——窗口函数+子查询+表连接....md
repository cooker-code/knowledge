---
title: SQL练习超详解——窗口函数+子查询+表连接...
author: 守住内心的宁静
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMjYwMTI2OQ==&mid=2247485614&idx=1&sn=1d2fd9aca9089ccfef96ad7ea0aa90a8&chksm=c1e22278f93c555a44a821e688b81dec6365518c3d6938399681d1c77f11eedf07cc9a2064cf&mpshare=1&scene=24&srcid=0806LA78ugeTP4MI5UgrDxl6&sharer_shareinfo=9322f9199c81987cadd1e83e8344df2e&sharer_shareinfo_first=9322f9199c81987cadd1e83e8344df2e#rd
---

Hello，今天分享的题目综合考察SQL语句中的窗口函数、子查询、日期函数、表连接等知识点。

| 题目

从听歌流水中找到18-25岁用户在2022年每个月播放次数top 3的周杰伦的歌曲。

流水表 play\_log

### 歌曲表 **song\_info**

用户表 **user\_info**

### | 输出

### 根据题目，你的查询应返回以下结果：

| SQL代码

| 详细解析

子查询

①表连接

* 通过song\_id字段将流水表play\_log与歌曲表song\_info进行内连接，获取歌曲名称和歌手名称。

* 通过user\_id字段将结果与信息表user\_info内连接，获取用户年龄。

②数据过滤

* 只保留歌手为"周杰伦"的歌曲

* 只保留年龄在18-25岁的用户

* 只保留2022年的播放记录

**③分组统计播放量**

* group by MONTH(fdate), song\_name, s.song\_id: 按月份、歌曲名称和歌曲ID分组。

* count(p.song\_id) as play\_pv: 使用count(p.song\_id)统计每首歌的播放次数，并将统计结果命名为play\_pv。

④窗口函数生成排名

* partition by MONTH(date)：按月份分区

* order by count(song\_name) desc, s.song\_id): 按播放次数降序排列,若播放次数相同，则按歌曲ID升序排列。

* 使用row\_number()为每首歌在对应月份内的播放次数分配唯一排名。

主查询

⑤子查询结果删选

where ranking <= 3: 只保留每个月排名前3的歌曲。

⑥最终结果排序

order by month, ranking: 按月份(month)和排名(ranking)排序。

逻辑顺序

SQL的执行逻辑顺序遵循从内向外、逐层处理的原则。

①数据过滤与关联：从播放日志中筛选出符合条件的记录(2022年、18-25岁用户、周杰伦的歌曲)。

②数据聚合：按月份和歌曲分组，统计每首歌的播放次数。

③排名计算：使用窗口函数为每首歌在对应月份内的播放次数分配排名。

④结果筛选：只保留每个月排名前3的歌曲。

⑤结果排序：按月份和排名排序，输出最终结果。

以上便是本题的全部分析，希望对您有帮助~

注：题目来源于牛客网，仅供学习交流，侵权删.

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
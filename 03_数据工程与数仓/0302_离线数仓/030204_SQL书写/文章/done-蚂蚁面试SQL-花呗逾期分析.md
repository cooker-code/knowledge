> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL口径验证与对账闭环|SQL口径验证与对账闭环]]
---
title: 蚂蚁面试SQL-花呗逾期分析
author: 三石大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485839&idx=1&sn=10fd59154710f135762ce0e4dc68a05f&chksm=fbaa14df4953361462e70c8cc2612f01fb62006483eb021467e681fa3303843132f4dfd493e8&mpshare=1&scene=24&srcid=1014ESGBXXWEY64DHsqEWjmU&sharer_shareinfo=9718f692556338af32295707af0dfcff&sharer_shareinfo_first=9718f692556338af32295707af0dfcff#rd
---

# 推荐阅读文章列表

[2024最新大数据开发面试笔记V6.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485272&idx=1&sn=6178cf811f80a0ba0a0091ca930a2d07&chksm=fa0890bdcd7f19ab9a0de619d0928c0a42ffe11edf7e247fb3f2ee53da809068088982db968c&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[我的大数据学习之路](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483862&idx=1&sn=963e528abadb77e16ee4c320f19d028c&chksm=fa089633cd7f1f251758769e5b28189dae45a36d22cab6fc7a8666dd6c7d0182f596c5c10a1f&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[面试聊数仓第一季](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483942&idx=1&sn=afade2e444dec602001790fd6b3c53f3&chksm=fa0895c3cd7f1cd5ac4c95db849ca98cda5b11d07945ef3ac7dae20aa8e70cbaeaf3ac4286ef&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

# SQL题目

> 来自蚂蚁数据研发一面

* 有一张用户贷款信息表dwd\_trd\_loan\_tb\_dd，包含uid（用户id）、amt（贷款金额）、ovd\_days（逾期天数）、dt（时间分区）以及逾期等级配置表dim\_ovd\_config\_dd，包含ovd\_days（逾期天数），user\_level（用户风险等级）
* **注意：示例如下，当ovd\_days=1且user\_level=1，表示用户逾期天数<=1时，用户风险等级都为1；当ovd\_days=30且user\_level=2，表示用户逾期天数>1同时<=30时，用户风险等级为2；**
* **问题：计算20241011日所有贷款用户对应的风险等级**

```
-- 举例如下：
-- 输入
-- dwd_trd_loan_tb_dd
uid   amt   ovd_days     dt
1001  1000     0       20241011
1002  1000     33      20241011
1003  1000     12      20241011
1004  1000     68      20241011
-- dim_ovd_config_dd
ovd_days  user_level
1             1
30            2
60            3
180           4
-- 输出
uid   user_level
1001      1
1002      3
1003      2
1004      4
```

# 答案解析

## 模拟数据

```
create table dwd_trd_loan_tb_dd (
uid varchar(20),
amt bigint,
ovd_days bigint,
dt varchar(20)
);
create table dim_ovd_config_dd (
ovd_days bigint,
user_level bigint
);
INSERT INTO dwd_trd_loan_tb_dd VALUES 
('1001',1000,0,'20241011'),
('1002',1000,33,'20241011'),
('1003',1000,12,'20241011'),
('1004',1000,68,'20241011')
;
INSERT INTO dim_ovd_config_dd VALUES 
(1,1),
(30,2),
(60,3),
(180,4)
;
```

## 思路分析

* **看到多张表，先进行JOIN**，但是一眼看去好像只能用逾期天数进行关联，可以又无法直接关联，那么就笛卡尔积（考虑到配置表很小）
* **这时候我们就需要判断每个用户的逾期天数是否小于所有配置的逾期天数，如果是则记为1**，这时候会出现一个用户对应多个1，我们要取对应配置逾期天数最小的那一条，怎么办？
* **按照uid进行分组，配置逾期天数进行排序，对标志位进行求和，**最后取开窗结果为1的行记录即可

## 具体代码

```
select 
 uid, 
 user_level
from (
 select
  t1.uid,
  t1.ovd_days,
  t2.ovd_days as ovd_days_config,
  t2.user_level,
  sum(if(t1.ovd_days < t2.ovd_days, 1, 0))  over(partition by t1.uid order by t2.ovd_days) as total_cnt
 from (
  select * 
  from dwd_trd_loan_tb_dd
  where dt = '20241011'
 ) t1
 join (
  select * 
  from dim_ovd_config_dd
 ) t2
 on 1 = 1
) t
where total_cnt = 1
;
```

# 写在最后

### V6.0笔记获取方式

> 公众号回复：大数据面试笔记
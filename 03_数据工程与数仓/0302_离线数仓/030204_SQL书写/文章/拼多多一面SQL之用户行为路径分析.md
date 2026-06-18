---
title: 拼多多一面SQL之用户行为路径分析
author: 三石大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485637&idx=1&sn=92efbd582646dcacb524624e1e9813da&chksm=fa089f20cd7f1636eccdcd81c590cbed7c5980ea5d7136df77631f9431c648b03ed058cdd4da&mpshare=1&scene=24&srcid=0719SHjjBYNgGhXNDNdFEbiq&sharer_shareinfo=abdedbbc495a1011c6228b3a5643b8f1&sharer_shareinfo_first=abdedbbc495a1011c6228b3a5643b8f1#rd
---

# 推荐阅读文章列表

[2024最新大数据开发面试笔记V6.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485272&idx=1&sn=6178cf811f80a0ba0a0091ca930a2d07&chksm=fa0890bdcd7f19ab9a0de619d0928c0a42ffe11edf7e247fb3f2ee53da809068088982db968c&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[我的大数据学习之路](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483862&idx=1&sn=963e528abadb77e16ee4c320f19d028c&chksm=fa089633cd7f1f251758769e5b28189dae45a36d22cab6fc7a8666dd6c7d0182f596c5c10a1f&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[面试聊数仓第一季](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483942&idx=1&sn=afade2e444dec602001790fd6b3c53f3&chksm=fa0895c3cd7f1cd5ac4c95db849ca98cda5b11d07945ef3ac7dae20aa8e70cbaeaf3ac4286ef&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

# SQL题目

> 来自拼多多数据研发一面

* 有一张用户行为日志表ods\_usr\_log, 包含用户id（user\_id）和页面id（page\_id）以及进入页面时间（in\_ts）
* **问题：统计每天进入A页面后，立刻进入B页面，又进入C页面的用户数****【注意：进入C页面之前可能进入过其他页面】**

# 答案解析

## 模拟数据

```
insert into ods_usr_log(user_id, page_id, in_ts) values   
(1, 'A', '2020-1-1 12:01:03'),  
(2, 'A', '2020-1-1 12:01:04'),  
(3, 'A', '2020-1-1 12:01:05'),  
(1, 'B', '2020-1-1 12:03:03'),  
(1, 'A', '2020-1-1 12:04:03'),  
(1, 'C', '2020-1-1 12:06:03'),  
(1, 'D', '2020-1-1 12:11:03'),  
(2, 'A', '2020-1-1 12:07:04'),  
(3, 'C', '2020-1-1 12:02:05'),  
(2, 'C', '2020-1-1 12:09:03'),  
(2, 'A', '2020-1-1 12:10:03'),  
(4, 'A', '2020-1-1 12:01:03'),  
(4, 'C', '2020-1-1 12:11:05'),  
(4, 'D', '2020-1-1 12:15:05'),  
(1, 'A', '2020-1-2 12:01:03'),  
(2, 'A', '2020-1-2 12:01:04'),  
(3, 'A', '2020-1-2 12:01:05'),  
(1, 'B', '2020-1-2 12:03:03'),  
(1, 'A', '2020-1-2 12:04:03'),  
(1, 'C', '2020-1-2 12:06:03'),  
(2, 'A', '2020-1-2 12:07:04'),  
(3, 'B', '2020-1-2 12:08:05'),  
(3, 'E', '2020-1-2 12:09:05'),  
(3, 'D', '2020-1-2 12:11:05'),  
(2, 'C', '2020-1-2 12:09:03'),  
(4, 'E', '2020-1-2 12:05:03'),  
(4, 'B', '2020-1-2 12:06:03'),  
(4, 'E', '2020-1-2 12:07:03'),  
(2, 'A', '2020-1-2 12:10:03');
```

## 思路分析

* **需求拆解如下：**

+ 要想求按照 A->B->C 序列的用户，就需要知道每个用户的行为路径
+ 这个不难求得，只需要将每个用户进入的页面按照进入时间进行concat即可
+ **那么如何保证A到B是直接到达的，并且B到C是可能间接到达的呢？**
+ 显然可以使用正则匹配 `like %A,B%C%`

## 具体代码

```
select date(in_ts) as dt  
      ,count(distinct user_id) as cnt  
from  
(  
    select user_id  
          ,in_ts  
          ,concat_ws(',',collect_set(page_id) over(partition by user_id order by in_ts)) as page_list  
    from ods_usr_log  
) t  
where page_list like '%A,B%C%'  
group by date(in_ts)  
;
```

# 写在最后

### V6.0笔记获取方式

> 公众号回复：大数据面试笔记
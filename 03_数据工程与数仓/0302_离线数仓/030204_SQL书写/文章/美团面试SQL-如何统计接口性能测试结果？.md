---
title: 美团面试SQL-如何统计接口性能测试结果？
author: 三石大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247486518&idx=1&sn=41d9dfde57c1d041cb03f53e90c29b22&chksm=fb146a1bb3e6eb44b9034a15ac4b6c7bf2436b307cd2aca21314479f50d61df110e43086898c&mpshare=1&scene=24&srcid=0610k0cDPREDxgzL5Y4slPhk&sharer_shareinfo=ab782f750651cdb4f9acf0fdb9863cd1&sharer_shareinfo_first=ab782f750651cdb4f9acf0fdb9863cd1#rd
---

# 推荐阅读文章列表

[2025最新大数据开发面试笔记V6.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485272&idx=1&sn=6178cf811f80a0ba0a0091ca930a2d07&chksm=fa0890bdcd7f19ab9a0de619d0928c0a42ffe11edf7e247fb3f2ee53da809068088982db968c&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[我的大数据学习之路](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483862&idx=1&sn=963e528abadb77e16ee4c320f19d028c&chksm=fa089633cd7f1f251758769e5b28189dae45a36d22cab6fc7a8666dd6c7d0182f596c5c10a1f&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[面试聊数仓第一季](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483942&idx=1&sn=afade2e444dec602001790fd6b3c53f3&chksm=fa0895c3cd7f1cd5ac4c95db849ca98cda5b11d07945ef3ac7dae20aa8e70cbaeaf3ac4286ef&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

# 需求背景

> 来自小象超市-数据研发一面

已知一张某页面所有后端接口的访问日志信息表，统计所有接口被持续访问成功的次数

ps：持续访问成功 = 连续两次访问成功

---

| controller\_id | ts | status |
| --- | --- | --- |
| order | 2025-06-01 10:01:08 | 1 |
| order | 2025-06-01 10:08:08 | 1 |
| order | 2025-06-01 10:11:08 | 0 |
| order | 2025-06-01 10:16:08 | 1 |
| order | 2025-06-01 10:18:08 | 1 |
| order | 2025-06-01 10:30:08 | 1 |
| product | 2025-06-01 10:03:08 | 0 |
| product | 2025-06-01 10:09:08 | 1 |
| product | 2025-06-01 10:20:08 | 0 |
| product | 2025-06-01 10:27:08 | 1 |

# 答案解析

## 模拟数据

```
create table ods_shop_interface_status_log (  
  controller_id varchar(20),  
  ts varchar(20),  
statusbigint  
);  
INSERTINTO ods_shop_interface_status_log VALUES  
('order','2025-06-01 10:01:08',1),  
('order','2025-06-01 10:08:08',1),  
('order','2025-06-01 10:11:08',0),  
('order','2025-06-01 10:16:08',1),  
('order','2025-06-01 10:18:08',1),  
('order','2025-06-01 10:30:08',1),  
('product','2025-06-01 10:03:08',0),  
('product','2025-06-01 10:09:08',1),  
('product','2025-06-01 10:20:08',0),  
('product','2025-06-01 10:27:08',1)  
;
```

## 思路分析

将题目需求进行转化，其实就是求解不同接口连续访问两次成功的次数，难点就在于如何判断连续两次访问均成功？

* 第一步，取当前行所在上一行的访问状态，如果均为1，则计数连续访问成功状态为1
* 第二步，按照不同接口分组，对连续访问成功状态进行累加

## 具体代码

```
select   
    controller_id,  
    sum(final_status) as success_cnt  
from (  
    select  
        controller_id,  
        ts,  
        status,  
        casewhenstatus = 1and lag_status = 1then1else0endas final_status  
    from (  
        select  
            controller_id,   
            ts,   
            status,   
            lag(status, 1) over(partitionby controller_id orderby ts) as lag_status  
        from ods_shop_interface_status_log  
    ) t  
) t  
groupby controller_id  
;
```

# 写在最后

### V6.0笔记获取方式

> 公众号回复：大数据面试笔记
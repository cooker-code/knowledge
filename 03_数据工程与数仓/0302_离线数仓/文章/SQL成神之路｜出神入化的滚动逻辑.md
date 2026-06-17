---
title: SQL成神之路｜出神入化的滚动逻辑
author: 胡说大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkyMDMxMzY0MQ==&mid=2247484439&idx=1&sn=6030e8edfb96db397858dda413e34eef&chksm=c0b96abf182ea9f476154a48dcb67184c33d4d30809d141575b7ba3d2f2d27b71aa6444b8792&mpshare=1&scene=24&srcid=1104EbxANkcSm6N9M9jgB0PE&sharer_shareinfo=f25a6e866107b565bedbec03818b96c1&sharer_shareinfo_first=f25a6e866107b565bedbec03818b96c1#rd
---

# SQL成神之路｜出神入化的滚动逻辑

> 数仓日常开发中必然少不了`「历史累计指标」、「周期累计指标」`的加工。暴力计算的话，在某些大数据量场景，可能引起计算性能问题。今天就来介绍介绍如何使用`滚动逻辑`来处理。

## 1. 背景

## 数据模型背景

在数仓建设中，一般都会针对某些实体构建特征表，作为中间层来使用。如：电商场景的用户行为特征表，`用户的主要路径为商品的曝光->点击->下单`。具体的指标为：历史累计曝光次数、历史累计点击次数、历史累计下单数、近30天累计曝光次数、近30天累计点击次数、近30天累计下单次数、近1天累计曝光次数、近1天累计点击次数、近1天累计下单次数。

## 性能问题

用户的曝光和点击数据是非常巨大的，`每天取历史全量的数据是完全不可行的，甚至取30天的数据也跑不出来`。这个时候滚动逻辑可以上场了。

## 2. 解法

### 历史累计指标

`可以取用户特征表的上一天分区的全量，取当天事实表的增量数据`，得到用户特征表当天分区历史累计指标

## 近30天累计指标

`可以取用户特征表的上一天分区的全量，减去过去第30天增量数据，加上当天的增量数据`，就可以得到当天分区的近30天累计指标。

## 代码示例

```
--用户行为增量表：dws_user_behave_1d  
--用户特征表：user_feature_nd  
select  
  nvl(a.user_id, b.user_id)                                      as user_id  
  --上一天分区累计指标+当天指标  
  ,nvl(accu_show_cnt,0)+nvl(b.show_cnt,0)                        as accu_show_cnt  
  ,nvl(accu_click_cnt,0)+nvl(b.click_cnt,0)                      as accu_click_cnt  
  ,nvl(accu_order_cnt,0)+nvl(b.order_cnt,0)                      as accu_order_cnt  
  --上一天分区近30天累计指标-过去第30天指标+当天指标  
  ,nvl(show_cnt_30d,0)-nvl(c.show_cnt,0)+nvl(b.show_cnt,0)       as show_cnt_30d  
  ,nvl(click_cnt_30d,0)-nvl(c.click_cnt,0)+nvl(b.click_cnt,0)    as click_cnt_30d  
  ,nvl(order_cnt_30d,0)-nvl(c.order_cnt,0)+nvl(b.order_cnt,0)    as order_cnt_30d  
  ,nvl(b.show_cnt,0)                                             as show_cnt  
  ,nvl(b.click_cnt,0)                                            as click_cnt   
  ,nvl(b.order_cnt,0)                                            as order_cnt  
from   
  (  
    select  
      user_id  
      ,accu_show_cnt  --历史累计曝光次数  
      ,accu_click_cnt --历史累计点击次数  
      ,accu_order_cnt --历史累计下单次数  
      ,show_cnt_30d   --近30天累计曝光次数  
      ,click_cnt_30d  --近30天累计点击次数  
      ,order_cnt_30d  --近30天累计下单次数  
    from user_feature_nd  
    where date='T-1'  
  ) a  
full join  
  (  
    select  
      user_id  
      ,sum(show_cnt)    as show_cnt    
      ,sum(click_cnt)   as click_cnt       
      ,sum(order_cnt)   as order_cnt    
    from dws_user_behave_1d  
    where date='T'  
    group by user_id  
  ) b on a.user_id=b.user_id  
left join  
  (  
    select  
      user_id  
      ,show_cnt         
      ,click_cnt        
      ,order_cnt   
    from user_feature_nd  
    where date='T-30'  
  ) c on a.user_id=c.user_id
```

## 3. 总结

本文介绍了如何使用滚动逻辑解决大数据量场景的历史/周期累计指标的计算效率问题，让我们的任务跑的飞快，希望对大家有所帮助，原创不易，您的`关注、点赞、分享`是对俺最大的支持!

**私人微信👇👇👇**

****欢迎加入星球，获取海量数据资料👇👇👇****
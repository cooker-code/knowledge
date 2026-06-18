---
title: Kafka 学习笔记（一）定位一次Kafka集群读写超时的故障
author: 何必那么复杂
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkzNTI0NDk2OQ==&mid=2247484255&idx=1&sn=fc2e7ff5c38a5b148c5615a50eb94f83&chksm=c3f38faa31773af69ad470bf8ec0f385970adbeae4794a869327dcebde780cd2dd3116ebfdc9&mpshare=1&scene=24&srcid=05088Tvrm1xivYbPHmJgIRs7&sharer_shareinfo=29c28cc81d0b9950481b19c5569f1e80&sharer_shareinfo_first=29c28cc81d0b9950481b19c5569f1e80#rd
---

## 前言：

这篇文章分享下kafka集群问题定位的经历，这篇文章有4个部分：

* 现象
* 救火
* 复盘
* 治理

 

## 现象

### 1. kafka 集群大量超时的读写请求

#### 1.1 网络端 Request队列满了，但是Response 很充足

图没了，需要自己脑补，哈哈

 

#### 1.2 处理线程池空闲率降低

 

## 救火

### 方案1:

#### 思路：

因为网络层Request队列满了，Response队列很充足，且Kafka集群曾经多次碰到由于节点磁盘、网卡、主板供电等硬件问题导致机器的IO能力下降的情况。所以怀疑broker节点硬件存在问题导致的。

#### 处理:

最快的解决方案就是把这个节点下掉，观察集群是否恢复。

#### 效果：

集群恢复了一段时间仍会出现超时的请求。

 

### 方案2:

#### 思路：

如现象1，2看到请求处理线程池空闲比都降到0了，是不是有大量请求，请求线程处理不过来导致的。

#### 处理：

调整kafka broker server 配置将请求线程池扩了一倍。

#### 效果：

依旧有大量超时读写的请求，处理线程依旧用尽了。

 

### 方案3:

#### 思路：

请求处理线程扩了一倍，感觉依旧起不到任何作用。那么请求处理线程到底在做什么呢，上工具：

##### 方法栈：

方法栈发现大量的请求线程被Partition的读锁阻塞住。

 

##### 火焰图：

通过wall clock类型的火焰图 （wall clock类型简单理解成可以查看方法栈调用耗时）发现，请求线程最耗时的都是写入操作，而写入操作都耗时在等待读锁上面。

#### 

#### 分析：

##### 代码：

  Kafka分区内部会维护一把读写锁，对于一般的consumer,producer操作使用分区的读锁即可。并不会有太大的影响。

那么如果碰到有写锁的情况，就会独占到这把锁，到底什么操作会使用到写锁呢。

* 副本Shrink:  在ISR列表中，如果分区追不上leader的数据就会踢出ISR列表，这个操作是Shrink。
* 副本Expand: 数据追上了Leader又加回了ISR列表，这个操作就是Expand。

##### Metric:

在读写超时的时间段正好有大量的Shrink ,Expand 的情况, 也就是说有副本被频繁的踢出ISR后又被加进来了。 这些操作独占用了大量leader分区锁的时间。

 

#### 参数

kafka broker 提供了参数  replica.lag.time.max.ms ，副本之间延迟多长时间之后才会踢出ISR列表。

#### 处理：

     在出问题的broker调大replica.lag.time.max.ms 。

#### 效果：

     未再出现频繁 Shrink ,Expand的情况，也未出现读写超时的情况。

 

## 复盘

调大参数可以解决频繁Shrink，Expand问题，但是还有两个问题没有弄清楚：

* 频繁Shrink，Expand 为什么会影响整个broker的Comsumer,Producer。
* 集群之前运行得好好的，到底是什么变化了。

 

### 问题1 ：频繁Shrink，Expand 为什么会影响整个broker的Comsumer,Producer

原因：

1. Shrink 和Expand 操作会占用副本的写锁之后，所有的请求处理线程都卡在请求读锁上面，不会再去消费请求队列，所以请求队列会被占满。因为请求队列是broker粒度的所以接下来的请求就都进不来了。
2. Expand可能会有些耗时，因为Expand在之后还会尝试将**暂存区的请求**拿出来尝试完成一遍（这个逻辑为什么也放到写锁里面目前我还没有去研究）

    

   **暂存区的请求**是啥，我举个例子：Producer设置成ack=all之后，会等所有的副本同步完成才会返回给客户端。具体的原理就是副本Leader 会维护一个High WaterMark (HW)来记录副本之间同步的偏移量，如果这次Producer请求的偏移量<HW，那么这么请求就会放到暂存区。等什么时候HW>这次prodcer 的偏移量才会返回。

   （Mark : 这里是不是可以考虑 将Kafka 集群按照数据重要分程度或者功能进行划分，将ack=all的需要强一致性的prodcuer 请求迁出日志集群，而日志集群可以界定成满足高可用的需求）

   暂存区metric:

 

### 问题2:集群之前运行得好好的，到底是什么变化了

     运维通过监控发现部分under replicated 的topic确实用日志突增的情况，只是压力并没有下推给服务器IO，而是频繁的从ISR列表进出副本。

 

## 治理

* 集群参数优化:

        根据集群规模，吞吐优化 replica.lag.time.max.ms 参数

* 集群治理：

+ 治理消息转换，减少集群压力

+ 推动topic数据压缩 ,减少broker磁盘/网络压力

+ 制定单分区数据写入速率规范
+ topic 分区级别流量报警,防止单分区写入瓶颈导致的分区抖动

* 集群划分

  将Kafka 集群按照数据重要分程度进行划分，将ack=all的需要强一致性的prodcuer 请求迁出日志集群，而日志集群可以界定成满足高可用的需求
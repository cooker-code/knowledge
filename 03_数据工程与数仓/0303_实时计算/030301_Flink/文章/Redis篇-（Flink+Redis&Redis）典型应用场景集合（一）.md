---
title: Redis篇-（Flink+Redis&Redis）典型应用场景集合（一）
author: 阿龙大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485210&idx=1&sn=44be379f6861b5b85d5a13905c9d639c&chksm=9e25ea857adcef55c6536892842f34db864ebd1c7b72cff989d43dd38800ecb5d13e20f1bba2&mpshare=1&scene=24&srcid=1227TtTOZaWPBNCtI3SjeDGY&sharer_shareinfo=f4293c12d373b90bc9bf60af59bb58cc&sharer_shareinfo_first=f4293c12d373b90bc9bf60af59bb58cc#rd
---

一、Flink+Redis应用场景

    大数据会用到Redis吗？ 

解答：

    会的，Flink 通常在以下场景中会用到 Redis：

1、缓存支持: 

    使用Redis作为Flink作业的缓存支持，例如在Flink程序中频繁访问某些数据，可以将这些数据加载到Redis缓存中，加速数据访问。

2、状态后端: 

    Flink 的状态后端可以选择使用Redis，将Flink状态存储在Redis中，提高状态存储的可靠性和可扩展性。

3、数据去重:

     在实时数据处理场景中，使用Redis的Set或HyperLogLog结构进行数据去重，避免重复处理相同的数据。

4、分布式锁: 

    Redis 提供了分布式锁的功能，可以在Flink程序中使用Redis实现分布式锁，确保在多个实例之间的并发操作的正确性。

```
Redis 与 Flink 结合使用可以提升系统的性能和可靠性。
```

二、Redis典型使用场景:

1、缓存（Cache）功能

其中Redis作为缓冲层，MySQL作为存储层，绝⼤部分请求的数据都是从Redis中获取。由于Redis具有支撑高并发的特性，所以缓存通常能起到加速读写和降低后端压力的作用。

2、计数（Counter）功能

许多应用都会使用Redis作为计数的基础⼯具，它可以实现快速计数、查询缓存的功能，同时数据可以异步处理或者落地到其他数据源。如图所示，例如视频播放次数可以使⽤Redis来完成：用户每播放⼀次视频，相应的视频播放数就会自增1。

3、共享会话（Session）

使用Redis将用户的Session信息进⾏集中管理，如图所示，在这种模式下，只要保证Redis是高可用和可扩展性的，无论用户被均衡到哪台Web 服务器上，都集中从Redis中查询、更新Session信息。

4、手机验证码功能

    很多应用出于安全考虑，会在每次进行登录时，让用户输人手机号并且配合给手机发送验证码，然后让用户再次输⼊收到的验证码并进⾏验证，从而确定是否是用户本人。为了短信接口不会频繁访问，会限制用户每分钟获取验证码的频率，例如⼀分钟不能超过 5 次并且可以设置手机验证码有效时间 过期了将不能使用。

记录每一份热爱,让美好永远陪伴。
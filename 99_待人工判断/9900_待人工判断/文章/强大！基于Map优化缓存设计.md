---
title: 强大！基于Map优化缓存设计
author: Spring全家桶实战案例
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621912&idx=1&sn=589894bbd901c818fd1b5f49fd3816ef&chksm=85de496fa8ffa80a61dcbb9c6c181812baf8e4f9f9114525a650d19d49ccbf71b4ec23b6761e&mpshare=1&scene=24&srcid=0125sTQadWu2hNtGKGAbMEpB&sharer_shareinfo=e70d95d43c353d98f2e6ddebb5788a03&sharer_shareinfo_first=e70d95d43c353d98f2e6ddebb5788a03#rd
---

**《**[**Spring Boot 3实战案例合集**](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MzI5NjQxNQ==&action=getalbum&album_id=3670066614814294019&scene=21&token=1635761271&lang=zh_CN#wechat_redirect)**》现已囊括超过****80篇精****选实战文章，****并且****此合集承诺将永久持续更新****，为您带来最前沿的技术资讯与实践经验。欢迎积极订阅，享受不断升级的知识盛宴！订阅用户将特别获赠合集内所有文章的最终版MD文档（详尽学习笔记），以及完整的****项目源码****，助您在学习道路上畅通无阻。**

**【重磅发布】《****Spring Boot 3实战案例锦集****》PDF电子书现已出炉！**

🎉🎉我们精心打造的《**Spring Boot 3实战案例锦集**》PDF电子书现已正式完成，目前已经有**80个案例**，后续还将继续更新。**文末有电子书目录。**

🔥🔥精彩内容不容错过：

《Spring Boot 3实战案例锦集》汇聚了众多精心挑选的实战案例，旨在帮助您快速掌握Spring Boot 3的核心技术和实战技巧。无论您是初学者还是有一定经验的开发者，都能从中受益匪浅。

💌💌如何获取：  
请立即订阅我们的合集**《[点我订阅](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MzI5NjQxNQ==&action=getalbum&album_id=3670066614814294019&scene=21&token=1635761271&lang=zh_CN#wechat_redirect)****》**，并通过私信联系我们，我们将第一时间将电子书发送给您。

**→**[**现在就订阅合集**](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MzI5NjQxNQ==&action=getalbum&album_id=3670066614814294019&scene=21&token=1635761271&lang=zh_CN#wechat_redirect)

---

环境：Java

---

**1. 简介**

**Map**是Java中的一种集合接口，用于存储键值对。它通过唯一的key来映射到值，提供了快速查找、插入和删除操作。

在需要处理高并发访问的场景中，直接使用传统的Map实现可能会导致线程安全问题。而**ConcurrentHashMap**作为Map的一个线程安全的子类，专为高并发设计。它允许并发访问和更新，无需显式同步，从而提供了高效的性能。因此，**ConcurrentHashMap**常被用作实现缓存的工具，能够存储频繁访问的数据，提高数据检索速度，并减轻底层数据源的负载，是高性能应用中不可或缺的一部分。

本篇文章，我们将更深入地探讨 **ConcurrentHashMap**在缓存中的应用，研究不同的缓存策略及其实现。

**1.1 为什么选择ConcurrentHashMap？**

缓存是一种技术，用于临时存储数据以减少将来访问这些数据所需的时间。在多线程应用程序中，实现一个可以安全地被多个线程同时访问和修改的缓存是具有挑战性的。传统的同步机制可能会因为线程间的竞争而导致性能瓶颈。ConcurrentHashMap通过提供以下特性来解决这些挑战：

* **线程安全：**内置机制确保安全的并发访问。
* **性能：**针对高并发进行了优化，通过内部机制（如锁分段）减少竞争。
* **易用性：**易于使用的API，能够无缝集成到现有的Java应用程序中。

**1.2 ConcurrentHashMap具有哪些特性？**

只有了解了这些特性你才能清楚选择使用ConcurrentHashMap的意义：

* **原子操作：**像putIfAbsent、remove、replace和computeIfAbsent这样的方法允许对映射进行原子更新，确保了一致性而无需显式锁定。
* **并发级别：**尽管在Java 8及更高版本中，concurrencyLevel参数已被弃用，但其内部设计仍然允许高效的并发更新。
* **可扩展性：**设计能够随着线程数量的增加而扩展，非常适合高并发环境。
* **非阻塞读取：**读取操作通常是非阻塞的，当读取操作占主导时，能够提升性能。

**2. 实战案例**

**2.1 基本内存缓存**

基本内存缓存存储键值对，并提供对数据的快速访问。这适用于数据频繁读取而较少更新的场景。

```
public class BasicCache<K, V> {  private final ConcurrentMap<K, V> cache = new ConcurrentHashMap<>();  /**从缓存中获取数据*/  public V get(K key) {    return cache.get(key);  }  /**如果key不存在则存入*/  public V putIfAbsent(K key, V value) {    return cache.putIfAbsent(key, value);  }  /**删除指定的key/value,值必须相同*/  public boolean remove(K key, V value) {    return cache.remove(key, value);  }  /**替换值*/  public boolean replace(K key, V oldValue, V newValue) {    return cache.replace(key, oldValue, newValue);  }}
```

以上我们就完成了一个最基本的缓存实现；同时它也存在如下2个比较严重的问题：

* **缺少过期机制**

  当前实现没有任何形式的数据过期或淘汰策略。这意味着一旦数据被放入缓存，除非显式地移除，否则它们将一直存在于内存中，这可能导致内存泄漏，特别是在缓存大小不受限制的情况下。
* **无大小限制**

  没有对缓存容量进行限制，可能会导致过多的对象占用大量内存，影响应用程序性能甚至造成 OOM（Out Of Memory）错误。

**2.2 引入过期机制**

引入过期的机制是非常有必要的，避免有些缓存数据可能本身就很少使用而一直存在于缓存中，占用大量的内存。要实现过期自动的删除，我们需要引入定时任务调度机制，定期的检查缓存数据。

```
public class ExpiringCache<K, V> {  private final ConcurrentMap<K, V> cache = new ConcurrentHashMap<>();  private final ConcurrentMap<K, Long> timestamps = new ConcurrentHashMap<>();  private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);  private final long expirationDuration;  private final TimeUnit timeUnit;  
  public ExpiringCache(long expirationDuration, TimeUnit timeUnit) {    this.expirationDuration = expirationDuration;    this.timeUnit = timeUnit;    scheduler.scheduleAtFixedRate(this::removeExpiredEntries, expirationDuration, expirationDuration, timeUnit);  }  
  public V get(K key) {    V v = cache.get(key) ;    if (v != null) {      // 1.检查时间是否已经过期      long expirationThreshold = System.nanoTime() - timeUnit.toNanos(expirationDuration) ;      if (timestamps.get(key) < expirationThreshold) {        clearKey(key) ;        return null ;      }      // 2.更新缓存时间      this.timestamps.put(key, System.nanoTime()) ;    }    return v ;  }  
  public V put(K key, V value) {    timestamps.put(key, System.nanoTime());    return cache.put(key, value);  }  
  public V putIfAbsent(K key, V value) {    timestamps.putIfAbsent(key, System.nanoTime());    return cache.putIfAbsent(key, value);  }  
  public V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {    timestamps.putIfAbsent(key, System.nanoTime());    return cache.computeIfAbsent(key, k -> {      V value = mappingFunction.apply(k);      timestamps.put(k, System.nanoTime());      return value;    });  }  
  public boolean remove(K key, V value) {    timestamps.remove(key);    return cache.remove(key, value);  }  
  private void removeExpiredEntries() {    long expirationThreshold = System.nanoTime() - timeUnit.toNanos(expirationDuration);    for (K key : timestamps.keySet()) {      if (timestamps.get(key) < expirationThreshold) {        clearKey(key) ;      }    }  }    private void clearKey(K key) {    timestamps.remove(key) ;    cache.remove(key) ;  }  
  public void shutdown() {    scheduler.shutdown();  }}
```

以上我们就实现了过期机制，而关于缓存大小限制我想你应该需要引入Queue相关技术来实现。

**2.3 自动加载数据存入缓存**

加载缓存会在缺少值时自动检索并存储这些值。这对于读取操作繁重的应用程序特别有用，因为可以从数据库或其他数据源中获取缺失的数据。

```
public class LoadingCache<K, V> {  private final ConcurrentMap<K, V> cache = new ConcurrentHashMap<>();  private final Function<K, V> loader;  
  public LoadingCache(Function<K, V> loader) {    this.loader = loader;  }  
  public V get(K key) {    return cache.computeIfAbsent(key, loader);  }  
  public V putIfAbsent(K key, V value) {    return cache.putIfAbsent(key, value);  }  
  public boolean remove(K key, V value) {    return cache.remove(key, value);  }  
  public boolean replace(K key, V oldValue, V newValue) {    return cache.replace(key, oldValue, newValue);  }  
  public V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {    return cache.computeIfAbsent(key, mappingFunction);  }  
  public V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {    return cache.merge(key, value, remappingFunction);  }}
```

在构造函数中，我们传入了**Function**对象，当缓存中不存在数据时，我们将调用**Function**去加载缓存数据并存入。

**最后：当你应用的对性能及相关应用的算法要求比较高时，我还是推荐你使用Caffeine，关于Caffeine的更多介绍及使用，请查看下面文章：**

[**高性能缓存神器Caffeine**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447616153&idx=1&sn=4a708d644b9dfeba2628c639d1db0b55&scene=21#wechat_redirect)

**以上是本篇文章的全部内容，如对你有帮助帮忙****点赞+转发+收藏**

**推荐文章**

[Spring Boot Rest API十大常见错误及避免方法](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621445&idx=1&sn=832dd2e4ec5d5fad8067ef997886ec0e&scene=21#wechat_redirect)

[**关于Spring AOP的两个高级功能你用过吗？**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621303&idx=1&sn=ef6e5733392aa83c704fbf555195c2df&scene=21#wechat_redirect)

[**技术专家！Spring Boot 增强版 @RequestMapping 添加限流功能**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621392&idx=1&sn=221b6cf8d5703cb045afab827508b8d8&scene=21#wechat_redirect)

[**优雅！Spring Boot + Redis秒实现RPC服务**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621449&idx=1&sn=a695ba8e8618a5b0f27eddc6fea8cd70&scene=21#wechat_redirect)

[**动态扩展能力！Spring Boot 一个强大实用的接口**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621281&idx=1&sn=737dec757908ef394fec593cd7722c34&scene=21#wechat_redirect)

[**写给新手！Spring AOP代理机制，必须清楚，否则各种失效**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447619891&idx=1&sn=0755c9a56367bceee53da370637b9aea&scene=21#wechat_redirect)

[Spring Boot 一次性Token，此功能太实用了](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447619761&idx=1&sn=eff5205f5fa99845f8610c42f64f5d40&scene=21#wechat_redirect)

[**优雅！Chain 结合 Spring Boot 轻松实现强大的责任链模式**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447621280&idx=1&sn=aee718e7ccd32009c1ea3bf20fa20dc2&scene=21#wechat_redirect)

[处理Null的神器Optional](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447617990&idx=1&sn=09ffceee092b087883ecf0802c5b7e4c&scene=21#wechat_redirect)

[**提升系统吞吐量，详解JDK21虚拟线程，炸裂**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447614477&idx=1&sn=00f29314fd575c49969044c80affa361&scene=21#wechat_redirect)

[Spring这个大坑要注意啦！会导致资源泄漏](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447614184&idx=1&sn=0745b6977ec61b290f2ec2386798f01c&scene=21#wechat_redirect)

[**SpringBoot中Controller接口参数这样处理太优雅了**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447614864&idx=1&sn=8c1427b032e7ae3a7e1cb1c71997e265&scene=21#wechat_redirect)

[优雅！SpringBoot通过函数式编程模型声明Restful API接口](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447615780&idx=1&sn=6ee3a1adcaed27c0c9999dc3a68f676c&scene=21#wechat_redirect)

[**优雅！Spring强大的数据绑定功能，太爽了**](https://mp.weixin.qq.com/s?__biz=MzA5MzI5NjQxNQ==&mid=2447620778&idx=1&sn=0a8137e593a906e63b977de2d3a468c4&scene=21#wechat_redirect)
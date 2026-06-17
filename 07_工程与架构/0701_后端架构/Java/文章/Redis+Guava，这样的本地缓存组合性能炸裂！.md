---
title: Redis+Guava，这样的本地缓存组合性能炸裂！
author: Java高性能架构
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247509985&idx=1&sn=433c26512c13d38b2a2d573e8ce72f36&chksm=c0c99748f7be1e5ea9afe62b8fad40be30e27db20ba29b4c6a54730234b7021172f8a2c5508b&mpshare=1&scene=24&srcid=0621sJa5tMNSkogTB2RXVYXP&sharer_sharetime=1655775546267&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

来源：https://juejin.cn/post/7000263632151904293

```
关注上方蓝色“Java高性能架构”

回复“资料”获取整理好的面试资料
```

## 前言

我们开发中经常用到 Redis 作为缓存，将高频数据放在 Redis 中能够提高业务性能，降低 MySQL 等关系型数据库压力，甚至一些系统使用 Redis 进行数据持久化，Redis 松散的文档结构非常适合业务系统开发，在精确查询，数据统计业务有着很大的优势。

但是高频数据流处理系统中，Redis 的压力也会很大，同时 I/O 开销才是耗时的主要原因，这时候为了降低 Redis 读写压力我们可以用到本地缓存，Guava 为我们提供了优秀的本地缓存 API，包含了过期策略等等，编码难度低，个人非常推荐。

## 设计示例

### Redis 懒加载缓存

数据在新增到 MySQL 不进行缓存，在精确查找进行缓存，做到查询即缓存，不查询不缓存。

流程图如下：

代码示例：

```
// 伪代码示例 Xx代表你的的业务对象 如User Goods等等  
public class XxLazyCache {  
  
    @Autowired  
    private RedisTemplate<String, Xx> redisTemplate;  
  
    @Autowired  
    private XxService xxService;// 你的业务service  
  
    /**  
     * 查询 通过查询缓存是否存在驱动缓存加载 建议在前置业务保证id对应数据是绝对存在于数据库中的  
     */  
    public Xx getXx(int id) {  
        // 1.查询缓存里面有没有数据  
        Xx xxCache = getXxFromCache(id);  
        if(xxCache != null) {  
            return xxCache;// 卫语句使代码更有利于阅读  
        }  
        // 2.查询数据库获取数据 我们假定到业务这一步，传过来的id都在数据库中有对应数据  
        Xx xx = xxService.getXxById(id);  
        // 3.设置缓存、这一步相当于Redis缓存懒加载，下次再查询此id，则会走缓存  
        setXxFromCache(xx);  
        return xx;  
        }  
    }  
  
    /**  
     * 对xx数据进行修改或者删除操作 操作数据库成功后 删除缓存  
     * 删除请求 - 删除数据库数据 删除缓存  
     * 修改请求 - 更新数据库数据 删除缓存 下次在查询时候就会从数据库拉取新的数据到缓存中  
     */  
    public void deleteXxFromCache(long id) {  
        String key = "Xx:" + xx.getId();  
        redisTemplate.delete(key);  
    }  
  
    private void setXxFromCache(Xx xx) {  
        String key = "Xx:" + xx.getId();  
        redisTemplate.opsForValue().set(key, xx);  
    }  
  
    private Xx getXxFromCache(int id) {  
        // 通过缓存前缀拼装唯一主键作为缓存Key 如Xxx信息 就是Xxx:id  
        String key = "Xx:" + id;  
        return redisTemplate.opsForValue().get(key);  
    }  
  
}  
// 业务类  
public class XxServie {  
    @Autowired  
    private XxLazyCache xxLazyCache;  
    // 查询数据库  
    public Xx getXxById(long id) {  
        // 省略实现  
        return xx;  
    }  
  
    public void updateXx(Xx xx) {  
        // 更新MySQL数据 省略  
        // 删除缓存  
        xxLazyCache.deleteXxFromCache(xx.getId());  
    }  
  
    public void deleteXx(long id) {  
        // 删除MySQL数据 省略  
        // 删除缓存  
        xxLazyCache.deleteXxFromCache(xx.getId());  
    }  
}  
// 实体类  
@Data  
public class Xx {  
    // 业务主键  
    private Long id;  
    // ...省略  
}
```

优点如下：

* 保证最小的缓存量满足精确查询业务，避免冷数据占用宝贵的内存空间
* 对增删改查业务入侵小、删除即同步
* 可插拔，对于老系统升级，历史数据无需在启动时初始化缓存

缺点如下：

* 数据量需可控，在无限增长业务场景不适用
* 在微服务场景不利于全局缓存应用

总结：

* 空间最小化
* 满足精确查询场景
* 总数据量可控推荐使用
* 微服务场景不适用

### Redis 结合本地缓存

微服务场景下，多个微服务使用一个大缓存，流数据业务下，高频读取缓存对 Redis 压力很大，我们使用本地缓存结合 Redis 缓存使用，降低 Redis 压力，同时本地缓存没有连接开销，性能更优。

流程图如下：

业务场景：在流处数处理过程中，微服务对多个设备上传的数据进行处理，每个设备有一个 code，流数据的频率高，在消息队列发送过程中使用分区发送，我们需要为设备 code 生成对应的自增号，用自增号对 kafka 中 topic 分区数进行取模。

这样如果有 10000 台设备，自增号就是 0~9999，在取模后就进行分区发送就可以做到每个分区均匀分布。

这个自增号我们使用 redis 的自增数生成，生成后放到 redis 的 hash 结构进行缓存，每次来一个设备，我们就去这个 hash 缓存中取，没有取到就使用自增数生成一个，然后放到 redis 的 hash 缓存中。

这时候每个设备的自增数一经生成是不会再发生改变的，我们就想到使用本地缓存进行优化，避免高频的调用 redis 去获取，降低 redis 压力。

代码示例：

```
/**  
 * 此缓存演示如何结合redis自增数 hash 本地缓存使用进行设备自增数的生成、缓存、本地缓存  
 * 本地缓存使用Guava Cache  
 */  
public class DeviceIncCache {  
  
    /**  
     * 本地缓存  
     */  
    private Cache<String, Integer> localCache = CacheBuilder.newBuilder()  
        .concurrencyLevel(16) // 并发级别  
        .initialCapacity(1000) // 初始容量  
        .maximumSize(10000) // 缓存最大长度  
        .expireAfterAccess(1, TimeUnit.HOURS) // 缓存1小时没被使用就过期  
        .build();  
  
    @Autowired  
    private RedisTemplate<String, Integer> redisTemplate;  
  
    /**  
     * redis自增数缓存的key  
     */  
    private static final String DEVICE_INC_COUNT = "device_inc_count";  
  
    /**  
     * redis设备编码对应自增数的hash缓存key  
     */  
    private static final String DEVICE_INC_VALUE = "device_inc_value";  
  
    /**  
     * 获取设备自增数  
     */  
    public int getInc(String deviceCode){  
        // 1.从本地缓存获取  
        Integer inc = localCache.get(deviceCode);  
        if(inc != null) {  
            return inc;  
        }  
        // 2.本地缓存未命中，从redis的hash缓存获取  
        inc = (Integer)redisTemplate.opsForHash().get(DEVICE_INC_VALUE, deviceCode);  
        // 3. redis的hash缓存中没有，说明是新设备，先为设备生成一个自增号  
        if(inc == null) {  
            inc = redisTemplate.opsForValue().increment(DEVICE_INC_COUNT).intValue;  
            // 添加到redis hash缓存  
            redisTemplate.opsForHash().put(DEVICE_INC_VALUE, deviceCode, inc);  
        }  
        // 4.添加到本地缓存  
        localCache.put(deviceCode, inc);  
        // 4.返回自增数  
        return inc;  
    }  
  
}
```

优点如下：

* redis 保证数据可持久，本地缓存保证超高的读取性能，微服务共用 redis 大缓存的场景能有效降低 redis 压力
* guava 作为本地缓存，提供了丰富的 api，过期策略，最大容量，保证服务内存可控，冷数据不会长期占据内存空间
* 服务重启导致的本地缓存清空不会影响业务进行
* 微服务及分布式场景使用，分布式情况下每个服务实例只会缓存自己接入的那一部分设备的自增号，本地内存空间最优
* 在示例业务中，自增数满足了分布区发送的均匀分布需求，也可以满足统计设备接入数目的业务场景，一举两得

缺点如下：

* 增加编码复杂度，不直接
* 只适用于缓存内容只增不改的场景

总结：

* 本地缓存空间可控，过期策略优
* 适用于微服务及分布式场景
* 缓存内容不能发生改变
* 性能优

## 后记

redis 提供了丰富的数据类型及api，非常适合业务系统开发，统计计数（increment，decrement），标记位（bitmap），松散数据（hash），先进先出、队列式读取（list）。

guava 缓存作为本地缓存，能够高效的读取的同时，提供了大量 api 方便我们控制本地缓存的数据量及冷数据淘汰。

我们充分的学习这些特性能够帮助我们在业务开发中更加轻松灵活，在空间与时间上找到一个平衡点。

————  e n d ————

开源项目推荐

[太强了，推荐7个牛哄哄 Spring Cloud 实战项目，拿来即用（附源码）](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247506468&idx=1&sn=9875f779d71e11d61c958f5238aa9349&chksm=c0c9ea8df7be639b501233815a768e4d146c07b7bbd3af6fbdd04e823598d188bf5ccb131d01&scene=21#wechat_redirect)

[10k+点赞！看看人家那快速搭建私人网盘系统，那叫一个优雅（附源码）](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247506179&idx=1&sn=f5a0149a5d551b4069f1ebabb985e946&chksm=c0c9e9aaf7be60bcdc6c9ae8121dcd9907a4e98f37ae16d65cbbd1b9266703947ebb2b20f507&scene=21#wechat_redirect)

[推荐一个高仿微信的项目 有点屌！！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247506116&idx=1&sn=34f67f7d929ed826cf33502bcb683d20&chksm=c0c9e86df7be617ba4659610c38a3d9d16b6cc6973431fef0e03210e41c6b4dfccf7fc8af186&scene=21#wechat_redirect)

[一键生成Springboot & Vue项目！ 【真乃神器】](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247506051&idx=1&sn=6e06f4931ce5a9ebb4f0f642df58bdca&chksm=c0c9e82af7be613cf6bada94523938ec664912af1cb3e7d8669add9d8b10271b4e5359c84805&scene=21#wechat_redirect)

[高颜值+前后端分离+代码生成器+开源，这款SpringBoot开发框架，真爽！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505824&idx=1&sn=f06aa5e5f47e2e2f0195a72270dc183d&chksm=c0c9e709f7be6e1f8ca14254d15a6208041767163aef1adbc207c04a7f9d5181df6d600b6b48&scene=21#wechat_redirect)

[牛X！一款开源的微信小程序商城+后台！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505716&idx=1&sn=9a036b1e228645d3dba4ed4be309e22f&chksm=c0c9e79df7be6e8b63141cbd386e5fd556acfd00a53913e2c1f4c2915c872d227ea3e5aeee91&scene=21#wechat_redirect)

[又一款接私活神器！Spring Boot + Vue 通用后台管理系统，真香！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505598&idx=1&sn=37c16b644343c0610a8898a9b648f4e2&chksm=c0c9e617f7be6f014fa66a632f8a4cb73186809bf8961b2cf7dd3fd29034d7ba2bd68fcbcaeb&scene=21#wechat_redirect)

[开源！数十个商业应用，都基于它实现!](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505508&idx=1&sn=38fe82f01e6740761ec87e6f5e25a03c&chksm=c0c9e6cdf7be6fdb119f563ebc8dbeb13977a8e44e8828bd6758187c37ebb5b535328b068344&scene=21#wechat_redirect)

[12个适合做外包项目的开源后台管理系统](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505425&idx=1&sn=ed9e15ffd50434d522077cf64ca8665f&chksm=c0c9e6b8f7be6faeab88a7c806372a107401e6455ef8bdf5301f153e2a382571a4a0d78f8496&scene=21#wechat_redirect)

[国内首个微服务化开发平台，开源！接项目直接用！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247505362&idx=1&sn=fd9fdeae0db43cf9c2f4c32c341b15e0&chksm=c0c9e57bf7be6c6d5109afc7939dc3d5fb894966bb0b0341ef2ab74fbc3acf104e9d3d5eba5e&scene=21#wechat_redirect)

[牛到不行，接私活大神都要用的神器，绝！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247504452&idx=1&sn=4550bbfff6205b3b9c43fd153cf591ea&chksm=c0c9e2edf7be6bfbfdcce838182a92d2ca1803d620042401a8caaebb528d560f5ca91470d294&scene=21#wechat_redirect)

[一个宝藏级微服务开源项目，吊到炸裂！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247503937&idx=1&sn=04a7fb9273fcc060097c02c2d67309f9&chksm=c0c9e0e8f7be69fe40fbe96307e659f8c9fc6d17c316845ba0972b4fa0cd12a97846874fa465&scene=21#wechat_redirect)

[一款基于SpringBoot+Bootstrap的OA系统，可二次开发接私活！](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247503881&idx=1&sn=cdd2e2147b3737f9a37c9838f062a4e7&chksm=c0c9e0a0f7be69b6eaa2053aeb445a4b7f652dfa8e8e038cb7859c59f42d53c5e5e25dd3471a&scene=21#wechat_redirect)

如有收获，点个在看，诚挚感谢
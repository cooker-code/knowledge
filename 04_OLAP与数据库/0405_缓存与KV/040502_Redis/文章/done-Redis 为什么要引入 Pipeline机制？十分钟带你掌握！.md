> 已吸收至：[[04_OLAP与数据库/0405_缓存与KV/040502_Redis/040502_核心知识点/RedisPipeline与Lua原子操作边界|RedisPipeline与Lua原子操作边界]]
---
title: Redis 为什么要引入 Pipeline机制？十分钟带你掌握！
author: 猿java
date:
url: http://mp.weixin.qq.com/s?__biz=MzIwNDAyOTI2Nw==&mid=2247502271&idx=1&sn=59be43ddb977df214a6ede23ea117454&chksm=9782422cef99ddb9441dc69be1efec9a271f4aeb41cf976292ae9cffbe60c856b134d8da83cd&mpshare=1&scene=24&srcid=0107XzEVgdVTzTETsXnfqEXk&sharer_shareinfo=8066f642a2cc25eca9379a257d3038c4&sharer_shareinfo_first=8066f642a2cc25eca9379a257d3038c4#rd
---

你好，我是**猿java**

**********点击关注公众号👇，获取：大厂简历指导和加技术群深度讨论！**********

在 Redis 中有一种 Pipeline（管道）机制，其目的是提高数据传输效率和吞吐量。那么，Pipeline是如何工作的？它又是如何提高性能的？Pipeline有什么优缺点？我们该如何使用 Pipeline？这篇文章，我们将进行深入的探讨。

# 1. Redis Pipeline是什么？

Redis Pipeline 是一种批量执行命令的技术，允许客户端在不等待服务器响应的情况下，一次性发送多个命令到 Redis 服务器。传统的请求-响应模式中，客户端每发送一个命令，就需要等待服务器响应后才能发送下一个命令，这种模式在高延迟网络环境下，严重影响 Redis 的性能表现。

Pipeline 通过消除或减少网络往返次数（Round-Trip Time, RTT），能够显著提高命令执行的吞吐量，客户端可以将多个命令打包发送，服务器则依次执行这些命令并将结果返回给客户端，从而有效地提升了网络利用率和整体性能。

# 2. 为什么引入 Pipeline？

在了解 Redis为什么引入 Pipeline之前，我们先来了解传统请求-响应模式，在传统的请求-响应模式中，客户端与服务器之间的通信流程如下：

1. 客户端发送一个命令到服务器。
2. 服务器接收命令并执行。
3. 服务器将执行结果返回给客户端。
4. 客户端接收结果后，发送下一个命令。

为了更直观地理解传统的请求-响应模式，下面给出了一张流程图：

在这种传统的模式下，每个命令都需要经历完整的 RTT，这在高延迟网络环境下会导致显著的性能瓶颈。

而 Pipeline的核心思想是“命令打包，高效传输”。其工作流程可以总结成下面 5个步骤：

1. **打包命令**: 客户端将多个 Redis 命令按照特定的格式打包成一个请求包。
2. **发送命令**: 将打包好的请求一次性发送给 Redis 服务器。
3. **执行命令**: Redis 服务器按顺序执行接收到的所有命令。
4. **接收响应**: 服务器将所有命令的执行结果按顺序返回给客户端。
5. **解析响应**: 客户端解析接收到的响应，并将结果对应到各个命令。

为了更直观地理解 pipeline模式，下面给出了一张流程图：

这种方式通过减少网络往返次数，有效降低网络延迟对性能的影响，特别适合于需要执行大量 Redis 命令的高并发场景。

尽管 Pipeline带来了性能的提升，但它也有一些缺点：

1. **资源消耗**: 发送大量命令一次性执行，可能会消耗较多的服务器资源，导致 Redis 其他操作的响应时间增加。
2. **错误处理复杂**: 在批量执行命令时，单个命令的错误处理可能变得复杂，需要逐一检查每个命令的执行结果。
3. **顺序依赖**: 如果命令之间存在顺序依赖，Pipeline 的批量执行需要确保正确的命令顺序。
4. **不支持事务功能**: Pipeline 只是批量执行命令的工具，不具备事务的原子性和隔离性。
5. **客户端支持**: 不同的 Redis 客户端对 Pipeline 的支持程度不同，使用时需考虑所选客户端库的特性和限制。

# 3. 源码分析

在 Java中，常见的 Redis 客户端库有 Jedis 和 Lettuce两种，下面我们将分别分析这两个库实现 Pipeline功能。

## 3.1 使用 Jedis 库

Jedis 是一个简单、直观的 Redis 客户端，支持 Pipeline 功能。下面的示例展示如何使用 Jedis实现 Pipeline操作。

```
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Response;

import java.util.ArrayList;
import java.util.List;

publicclass JedisPipelineExample {

    public static void main(String[] args) {
        // Redis 连接参数
        String redisHost = "localhost";
        int redisPort = 6379;
        String redisPassword = null; // 若有密码，填写密码

        // 连接 Redis
        try (Jedis jedis = new Jedis(redisHost, redisPort)) {
            if (redisPassword != null && !redisPassword.isEmpty()) {
                jedis.auth(redisPassword);
            }
            // 批量设置键值对
            batchSet(jedis);

            // 批量获取键值对
            batchGet(jedis);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 使用 Pipeline 批量设置键值对
     *
     * @param jedis Jedis 实例
     */
    public static void batchSet(Jedis jedis) {
        System.out.println("开始批量设置键值对...");

        Pipeline pipeline = jedis.pipelined();

        int numCommands = 1000;
        for (int i = 0; i < numCommands; i++) {
            pipeline.set("key-" + i, "value-" + i);
        }

        // 执行所有命令
        pipeline.sync();

        System.out.println("批量设置完成，共设置 " + numCommands + " 个键值对。");
    }

    /**
     * 使用 Pipeline 批量获取键值对
     */
    public static void batchGet(Jedis jedis) {
        System.out.println("开始批量获取键值对...");

        Pipeline pipeline = jedis.pipelined();

        int numCommands = 1000;
        List<Response<String>> responses = new ArrayList<>(numCommands);
        for (int i = 0; i < numCommands; i++) {
            Response<String> response = pipeline.get("key-" + i);
            responses.add(response);
        }

        // 执行所有命令
        pipeline.sync();

        // 处理结果
        for (int i = 0; i < numCommands; i++) {
            String value = responses.get(i).get();
            System.out.println("key-" + i + " = " + value);
        }

        System.out.println("批量获取完成，共获取 " + numCommands + " 个键值对。");
    }
}
```

**代码解析**

上面的代码主要总结为 4个步骤：

**1. 连接 Redis**:

使用 `Jedis` 类连接 Redis 服务器。如果 Redis 服务器设置了密码，需要调用 `jedis.auth` 进行认证。

**2. 批量设置键值对**:

* 调用 `jedis.pipelined()` 获取一个 `Pipeline` 对象。
* 使用循环将多个 `set` 命令添加到 Pipeline 中。
* 调用 `pipeline.sync()` 发送所有命令并等待执行结果。
* 通过 Pipeline 一次性提交所有命令，减少了网络往返次数。

**3. 批量获取键值对**:

* 同样使用 `pipelines` 获取 `Pipeline` 对象。
* 使用 `pipeline.get` 方法批量添加 `get` 命令，并将 `Response` 对象保存到列表中。
* 调用 `pipeline.sync()` 发送所有命令并等待执行结果。
* 遍历 `Response` 对象列表，获取每个键的值。

**4. 关闭连接**:

使用 try-with-resources 语法自动关闭 Jedis 连接，确保资源的正确释放。

## 3.2 使用 Lettuce 库

Lettuce 是一个基于 Netty 的可伸缩、多线程的 Redis 客户端，支持异步和反应式编程模型，同样支持 Pipeline 功能。

以下示例展示如何使用 Lettuce 实现 Pipeline 操作，包括批量设置和获取键值对。

```
import io.lettuce.core.RedisClient;
import io.lettuce.core.RedisURI;
import io.lettuce.core.api.StatefulRedisConnection;
import io.lettuce.core.api.sync.RedisCommands;
import io.lettuce.core.api.sync.SyncCommands;
import io.lettuce.core.api.sync.RedisScriptingCommands;
import io.lettuce.core.api.sync.RedisClusterCommands;

import java.util.ArrayList;
import java.util.List;

publicclass LettucePipelineExample {

    public static void main(String[] args) {
        // Redis 连接参数
        String redisHost = "localhost";
        int redisPort = 6379;
        String redisPassword = null; // 若有密码，填写密码

        // 创建 RedisURI
        RedisURI redisURI = RedisURI.Builder.redis(redisHost)
                .withPort(redisPort)
                .withPassword(redisPassword != null ? redisPassword.toCharArray() : null)
                .build();

        // 创建 RedisClient
        RedisClient redisClient = RedisClient.create(redisURI);

        // 建立连接
        try (StatefulRedisConnection<String, String> connection = redisClient.connect()) {
            RedisCommands<String, String> syncCommands = connection.sync();

            // 批量设置键值对
            batchSet(syncCommands);

            // 批量获取键值对
            batchGet(syncCommands);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭客户端
            redisClient.shutdown();
        }
    }

    /**
     * 使用 Lettuce 的 Pipeline 批量设置键值对
     *
     * @param syncCommands 同步命令接口
     */
    public static void batchSet(RedisCommands<String, String> syncCommands) {
        System.out.println("开始批量设置键值对...");

        int numCommands = 1000;
        for (int i = 0; i < numCommands; i++) {
            syncCommands.set("key-" + i, "value-" + i);
        }

        // 批量执行所有命令
        syncCommands.getStatefulConnection().flushCommands();

        System.out.println("批量设置完成，共设置 " + numCommands + " 个键值对。");
    }

    /**
     * 使用 Lettuce 的 Pipeline 批量获取键值对
     *
     * @param syncCommands 同步命令接口
     */
    public static void batchGet(RedisCommands<String, String> syncCommands) {
        System.out.println("开始批量获取键值对...");

        int numCommands = 1000;
        List<String> keys = new ArrayList<>(numCommands);
        for (int i = 0; i < numCommands; i++) {
            keys.add("key-" + i);
        }

        List<String> values = syncCommands.mget(keys.toArray(new String[0]))
                .stream()
                .map(res -> res.getValue())
                .toList();

        for (int i = 0; i < numCommands; i++) {
            System.out.println(keys.get(i) + " = " + values.get(i));
        }

        System.out.println("批量获取完成，共获取 " + numCommands + " 个键值对。");
    }
}
```

**代码解析**

上面的代码主要总结为 4个步骤：

**1. 连接 Redis**:

使用 `RedisClient` 创建连接，`RedisURI` 封装了连接参数。如果 Redis 服务器设置了密码，需要在 `RedisURI` 中指定。

**2. 批量设置键值对**:

* 使用 `syncCommands.set` 方法批量添加 `set` 命令。
* 调用 `flushCommands()` 方法将所有积累的命令一次性发送到服务器。
* 注：Lettuce 的 Pipeline 支持隐式的 Pipeline，即没有显式的 Pipeline API，通过积累命令并调用 `flushCommands()` 实现批量发送。

**3. 批量获取键值对**:

* 使用 `mget` 方法一次性获取多个键的值，这是 Lettuce 提供的批量获取命令，天然支持 Pipeline。
* `mget` 返回一个包含每个键值的 List，通过流处理提取值。

**4. 关闭连接**:

使用 try-with-resources 语法自动关闭连接，最后调用 `redisClient.shutdown()` 关闭 Redis 客户端。

尽管 Lettuce 支持 Pipeline，但其 API 不如 Jedis 那样显式。要实现更细粒度的 Pipeline 控制，可以使用 Lettuce 的命令缓冲机制或异步 API。上述示例中展示的是同步方式，适用于简单的批量操作。

# 4. 使用场景

1. **批量设置键值对**: 将大量键值对一次性写入 Redis，适用于数据初始化或大规模更新。
2. **批量获取键值对**: 在需要同时获取多个键的值时，通过 Pipeline 减少请求次数，提高效率。
3. **分布式计数器**: 高并发情况下，使用 Pipeline 聚合多个计数操作，提升吞吐量。
4. **缓存预热**: 在应用启动或重启时，通过 Pipeline 将常用数据加载到缓存中，提高应用启动性能。

# 5. 总结

本文，我们详细地分析了Redis的 Pipeline功能，以及从源码角度分析了 Java中常见的两种实现方式。通过批量发送命令，显著减少了网络延迟对性能的影响，提高了大量命令执行的效率。

然而，作为技术人员，我们不能只是一味的追求性能，而应该根据实际情况和需求，同时需要考虑 Pipeline可能带来的问题，比如资源消耗、错误处理等问题。只有综合地考虑其优缺点，才能帮助我们更好地技术选型和落地。

文章总结不易，求一键三连：点赞、转发、在看。另外，为了方便大家更深入地进行技术交流，我维护了一个猿java内部的技术交流群，欢迎大家加群讨论。关注公众号**「猿java」，****回复「****加群**」即可。
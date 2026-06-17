---
title: Flink 多流广播（上）
author: 大圣数据星球
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg2NjY0MDM5Nw==&mid=2247484665&idx=1&sn=fe20d1b61eddcc9319137a375d49a97b&chksm=cfee5dc4569e2cba9148cad65102feb999fc17a9b453d745b6c1b9fa702cd0c6622a8076d8e1&mpshare=1&scene=24&srcid=0209k7SCMDRyoqi5LlzjA4uK&sharer_shareinfo=bd6d886ff31132c97b686ecab54d8616&sharer_shareinfo_first=bd6d886ff31132c97b686ecab54d8616#rd
---

> 当我们使用 Flink 需要把一些数据发到各个并行子任务的时候，这个时候就需要广播流，**但是 Flink 里面 connect 算子只支持两条流逻辑上的连接** 。那么如果我们需要发送的数据是来自两条流的时候，这个时候怎么把这些数据广播到并行子任务里面呢？

## 广播流讲解

### 基本概念

在使用广播流的时候，一般有广播流和主流这两个概念，看下图：

  
你可以理解我们主流是从 Kafka 里面消费数据，然后这个Flink 任务的每一个子任务都需要一份相同的规则，然后对数据去执行一定的逻辑。这个时候就需要广播流，这就是广播流的使用场景和概念。

### 代码实现

```
package org.apache.flink.streaming.examples;  
  
import org.apache.flink.api.common.state.MapStateDescriptor;import org.apache.flink.api.common.state.ReadOnlyBroadcastState;import org.apache.flink.streaming.api.datastream.BroadcastStream;import org.apache.flink.streaming.api.datastream.DataStream;import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;import org.apache.flink.util.Collector;  
public class MainBroadcastExample {    public static void main(String[] args) throws Exception {        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
        // Define main data stream (e.g., user transactions)        DataStream<String> mainStream = env.fromElements(                "1,100",  // UserId=1, Amount=100                "2,200",  // UserId=2, Amount=200                "3,150"   // UserId=3, Amount=150        );  
        // Define broadcast stream (e.g., user metadata)        DataStream<String> broadcastStream = env.fromElements(                "1,John", // UserId=1, Name=John                "2,Jane", // UserId=2, Name=Jane                "3,Doe"   // UserId=3, Name=Doe        );  
        // Define a MapStateDescriptor for the broadcast state        MapStateDescriptor<String, String> broadcastStateDescriptor =                new MapStateDescriptor<>("userInfo", String.class, String.class);  
        // Broadcast the metadata stream        BroadcastStream<String> broadcastedStream = broadcastStream.broadcast(                broadcastStateDescriptor);  
        // Connect main stream with broadcast stream and process        DataStream<String> resultStream = mainStream                .connect(broadcastedStream)                .process(new BroadcastProcessFunction<String, String, String>() {  
                    @Override                    public void processElement(                            String value,                            ReadOnlyContext ctx,                            Collector<String> out) throws Exception {                        // Parse main stream data                        String[] parts = value.split(",");                        String userId = parts[0];                        String amount = parts[1];  
                        // Get broadcast state                        ReadOnlyBroadcastState<String, String> broadcastState = ctx.getBroadcastState(                                broadcastStateDescriptor);  
                        // Match with broadcasted data and emit result                        if (broadcastState.contains(userId)) {                            String userName = broadcastState.get(userId);                            out.collect("User: " + userName + ", Amount: " + amount);                        } else {                            out.collect("User: Unknown, Amount: " + amount);                        }                    }  
                    @Override                    public void processBroadcastElement(                            String value,                            Context ctx,                            Collector<String> out) throws Exception {                        // Parse broadcast stream data                        String[] parts = value.split(",");                        String userId = parts[0];                        String userName = parts[1];  
                        // Update broadcast state                        ctx.getBroadcastState(broadcastStateDescriptor).put(userId, userName);                    }                });  
        // Print the result        resultStream.print();  
        // Execute the job        env.execute("MainBroadcastExample");    }}
```

> 注意，这里的 connect 算子只能关联两条流，如果是多条流 connect 是做不了的。

## 多条广播流

### 问题思考

如果我们需要广播的数据是多条流，上面说过connect 算子只能关联两条流， 那应该怎么办呢？

### 解决思路

其实我们可以用面向对象的思想，**在使用connect 算子之前，把多条广播流的实体类对象向上转型抽象成一个父类，然后在 connect 之后，使用广播流的时候再向下转型把父类转换为具体的子类就可以了**。

具体细节我会在下一篇文章中间详细讲解。

## 写在最后

最近怎么说呢，觉得自己越来越懒了，以前给自己定的写公众号的目标总是找各种理由推脱。导致现在有点迷茫了，在考虑了一段时间之后，觉得还是要先起来，不要觉得自己不行，其实世界本就是草台班子。
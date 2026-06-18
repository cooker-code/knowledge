> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkWebUI指标解读|FlinkWebUI指标解读]]、[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink监控告警体系|Flink监控告警体系]]
---
title: Flink生产实时监控和预警配置解析
author: 大数据技术与架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512774&idx=2&sn=95824d155e5076e2051f100f38883e76&chksm=fd3ef453ca497d455a6f05a75bc77f5bf0f4bb7ca956caf55f357f573a271ed49c8e72bd711a&mpshare=1&scene=24&srcid=0414VQ2PmU0vv1Xby6XYHsYN&sharer_sharetime=1649942119786&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

点击上方**蓝色字体**，选择“设为星标”

回复"**面试"**获取更多惊喜

[**八股文教给我，你们专心刷题和面试**](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247512011&idx=1&sn=68aaa9c5c42e2087d56d170c442b30ba&chksm=fd3ee95eca496048b24a717a30fe820b9188d02e014ba7b554afe86f8da1efab189799fe7087&scene=21#wechat_redirect)

> Hi，我是王知无，一个大数据领域的原创作者。
> 放心关注我，获取更多行业的一手消息。

在实际的Flink 项目中，如何观察Flink的性能，如何监控Flink的运行状态，如何设置报警策略？下面简单讲下我的经验吧。

## 一、Flink webUI

首先聊下Flink webUI。如下图所示：

如果是本地调试模式，默认是不开启webui的。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

上面的初始化方式，本地调试默认不开启webui。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(new Configuration());
```

需要使用上面这种方式才能在本地调试的时候打开webui。当然了，也需要在pom文件中添加依赖

```
<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-runtime-web_2.11</artifactId>
            <version>${flink.version}</version>
        </dependency>
```

如果你是on yarn 模式，则必须使用第一种初始化方式，on yarn 默认可以查看webui。

下面是一个读取kafka数据，通过Flink 处理后，再写入目标kafka的任务。

如上图所示，点击sink，在metrics中选择Sink\_\_sink.numRecordsInPerSecond。这里有几个并行度，就需要全部选出来，如果你设置了50个并行度，那么就要选50次。

source也是同样操作

那么，从上图可知，该任务sink总速度为560\*3=1680 条/s，source总速度为1737 条/s。基本相等

那么接下来，我们怎么判定速度是否正常呢？

我们可以借助kafka-eagle查看kafka topic的写入速度。

可以看到kafka的写入速度是1.66k/s,而我们的业务逻辑，输入和输出是1:1，所以，flink的写入速度和kafka的生产速度保持一直.

这里如果看到kafka的生产速度明显高于flink的source和sink速度，则基本可以断定，Flink已经产生反压，并且性能不符合线上要求。

那么是否kafka写入速度和Flink的消费速度一致，就表示万事大吉了呢？也不一定，我们需要通过FlinkWebui直接观察反压的情况。

如果和上图一样Ratio是0，并且status是ok，那么说明一切正常。

如果此时出现反压，说明Flink的消费速度，只能勉强等于日常的生产速度，并且此时有积压的数据。这种情况会在补数据的时候会比较明显，如果一个任务的极限性能仅仅等于或略大于生产日常的性能，则出现这种情况的概率会很高，

所以，一般来说，在Flink任务上线前，我们需要测试极限性能，一般要求至少3倍的日常速度，做到10倍以上，是最好的。

下面是一个读取kakfa 数据，处理后写mysql的任务。

上图说明下游产生了反压，但是由于下游有group by 等一系列操作，我们无法确定瓶颈出在了哪里。如果需要查看具体哪一步产生了反压，我们可以通过如下设置来禁止合并。

```
env.disableOperatorChaining();
```

如上图所示，将所有子任务全部采集反压信息。从最上的子任务往下数，第一个反压为绿色的就是罪魁祸首。如上图所示，FlatMap，是红色，sink为绿色，说明反压在了sink，也就是说mysql的写入速度，不能满足我们的需求，导致上游Flink处理全部被限制了速度。

当然，罪魁祸首不一定只有一个，mysql的写入性能解决后，还有可能反压在其他阶段，但是我们通过这种方式，可以一步步定位问题，解决问题，有针对性的优化问题，而不是像某些领导赏识的同事一样，只知道增加并行度，最终极大增加了集群压力，一个任务动辄几百G，成为集群不稳定的因素之一，完了还甩锅给其他人，那就没意思了。

## 二、Kafka 消费 监控

我们知道，Flink在 打checkpoint时才向kafka集群提交offset消费信息的，所以如果仅仅站在kafka lag 的角度，我们看到的消费延迟是锯齿状的图形，大致长这样

上图是一个checkpoint为3min，并且flink没有反压的kafka lag监控图。

在脚本中我们可以通过如下命令获取kafka总lag

```
lag=`kafka/kafka_2.11-2.0.1/bin/kafka-consumer-groups.sh --bootstrap-server *.*.*.*:6667 --describe --group "$2" |grep "$3"  2>/dev/null |grep -v LAG|awk '{sum+=$5}END
```

这时候我们需要引入一个概念，Flink消费虚拟速度F0。设flink checkpoint间隔为t

F0=lag/t

例如，最高峰时，kafka 的lag 为30000 ，

F0=30000/60/3=167

Flink虚拟消费速度在最高峰时约等于167条/s。

设Flink 真实消费速度为F1.(通过webui 直接获得)，预警倍数为m

再设预警消费速度为F2，F2=F1\*m

例如Flink 任务日常的消费速度为167/s，峰值为250/s，我们设置预警倍数为2.那么当F0>F2时，我们触发报警。

可以看到，仅仅通过Kafka lan监控Flink任务状态 ，在出现高峰时，可能存在误报的情况，但是如果将预警倍数设置太高，又可能降低Flink预警的及时性。实际情况中，我们需要根据业务情况，设置合理的m和t,在允许极少误报的情况下，做到实时任务的故障对用户无感知，当然，前提是笔记本随身携带。。。

## 三、yarn 监控

由于我们都是per job 模式，所以在yarn上都会有唯一名字，在脚本中可以通过如下方式获得num。

```
  num=`yarn application -list | grep "FlinkJobName" | wc -l`
```

如果num小于1，那么就说明Flink任务挂了，简单直接。

但是也有一种情况，那就是集群yarn挂了。由于我们公司的集群建设做的很差，经常出现这种情况，所以在监控脚本中，不能监控到num=0就直接启动Flink，这样可能会导致下游数据翻倍，而是应该电话通知，人工确认状态后，再手动启动Flink任务。

例如，可以和kafka lag 监控综合来看，如果kafka lag一切正常，yarn 查不到任务信息，那大概率是说明yarn 挂了，但是Flink任务还在正常运行。

## 总结:

1. 通过yarn，kafka，flink web ui 综合判断Flink任务健康状态。
2. 通过设置合理的m和t做到最少的误报率和最高的SLA
3. Flink 程序质量是第一位，极限性能至少在高峰性能2倍以上，监控只是辅助，Flink 优化不到位，再多的监控也没法保证高SLA。

如果这个文章对你有帮助，不要忘记 **「在看」** **「点赞」** **「收藏」** 三连啊喂！

[2022年全网首发|大数据专家级技能模型与学习指南(胜天半子篇)](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247510040&idx=1&sn=aa335f25965975731173916f012d56f4&chksm=fd3eee8dca49679b82f632048fb64d21fac01497d1a0fe33917edb01e194caf0f9a1930a55ce&scene=21#wechat_redirect)

[互联网最坏的时代可能真的来了](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247508317&idx=1&sn=0bcb7fb6997b42306994b890eaa0d47f&chksm=fd3ee7c8ca496ede347a2971002c6ea68dcfd70abeeb90b43c8b02a2a60ebc7cec3a87ef7d52&scene=21#wechat_redirect)

[我在B站读大学，大数据专业](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247507860&idx=1&sn=807ac5003762f29c127bc4071dcebe33&chksm=fd3e9901ca4910178cfc816043ea86c29f881f70cd943d252de9ae2e21cb7e8ba80bff162e1b&scene=21#wechat_redirect)

[我们在学习Flink的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247499604&idx=1&sn=d938dfb30d221774704982d2938b30c1&chksm=fd3eb9c1ca4930d76a391241333de461ca22d2aa27472eab3cffab1564872ae37f1b48fe2d3c&scene=21#wechat_redirect)

[193篇文章暴揍Flink，这个合集你需要关注一下](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504856&idx=2&sn=6f62e2a0c756ce56773eed253d2f41ee&chksm=fd3e954dca491c5b732b7e2aa46db32efbc3f690e528522098659bb3c6e78e1582ab4529b5dc&scene=21#wechat_redirect)

[Flink生产环境TOP难题与优化，阿里巴巴藏经阁YYDS](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504742&idx=1&sn=8765c198d8ad66219a7bcabb221e4a23&chksm=fd3e95f3ca491ce52d0724b9e4154a47af0f5ea349e1e184c8fbc65aa17c0000242aa9b58429&scene=21#wechat_redirect)

[Flink CDC我吃定了耶稣也留不住他！| Flink CDC线上问题小盘点](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504813&idx=1&sn=f5cd6ae2aa2b1e30f87a5ae55971c514&chksm=fd3e9538ca491c2eb191677d070f2c7e4f1098eece00e256d6205b8a5d21b21c1e74f875f16f&scene=21#wechat_redirect)

[我们在学习Spark的时候，到底在学习什么？](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=1&sn=1f693e549f76622725b936041ff8896e&chksm=e9bff2fbdec87bedecbdeed4547d7d612c72fc8d4918614a26a4e31666cf0128fd57b537f347&scene=21#wechat_redirect)

[在所有Spark模块中，我愿称SparkSQL为最强！](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504834&idx=1&sn=46ca1a3924b8fdb89ac1c2acb0319d6c&chksm=fd3e9557ca491c41d837b917639ea62007ea16d3c2cc6c0c02d1f8a8c6e44318f5183b787c46&scene=21#wechat_redirect)

[硬刚Hive | 4万字基础调优面试小总结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247502750&idx=1&sn=bd9a9173d060dc4e4ebd49c8efc6acfe&chksm=fd3e8d0bca49041dea84da93910e5efdc4935e520525c09887c986691377aeb48e5cf7fb5667&scene=21#wechat_redirect)

[数据治理方法论和实践小百科全书](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504382&idx=2&sn=550b78802acfe727e0e77cd9195f8784&chksm=fd3e976bca491e7db2b8b2446d231736df01bbf13653804d680ac4390597ec17fa466ad4ae83&scene=21#wechat_redirect)

[标签体系下的用户画像建设小指南](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503741&idx=1&sn=e5039be93123f2e337013756a818bfc3&chksm=fd3e89e8ca4900fe603b63c5722a6fb8a32bd63d6ba23e0028851948a71b877eb1f742d95087&scene=21#wechat_redirect)

[4万字长文 | ClickHouse基础&实践&调优全视角解析](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247503675&idx=1&sn=3ee6af64d0126c78b48cad219308f81e&chksm=fd3e89aeca4900b8b8954e9569ee3c0877881fac8c792bfafc22e7e9d3e8524da8eb860d33d8&scene=21#wechat_redirect)

[【面试&个人成长】2021年过半，社招和校招的经验之谈](http://mp.weixin.qq.com/s?__biz=MzI0NjU2NDkzMQ==&mid=2247492567&idx=2&sn=57ecc77718f1b6f2f262d62e8318dcc9&chksm=e9bff2fbdec87bedb1986765bfae7dbabb9aece45b0b2af147bbad1ac5d79ebc934c64df40ea&scene=21#wechat_redirect)

[大数据方向另一个十年开启 |《硬刚系列》第一版完结](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504478&idx=1&sn=14efef3868ba42bd044f9618745a7fdc&chksm=fd3e94cbca491dddb961b7b8b93105b2869c5bcf4c03f9f0dc83ad62be0dce4c124b52ed0ba3&scene=21#wechat_redirect)

[我写过的关于成长/面试/职场进阶的文章](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504410&idx=1&sn=7e81ab5a324395eb0f12397c40247ca1&chksm=fd3e948fca491d9946456acbd93b2d651ae4d7a7d0127a211981e08f50f377e60d1b7c0d7b3a&scene=21#wechat_redirect)

[当我们在学习Hive的时候在学习什么？「硬刚Hive续集」](http://mp.weixin.qq.com/s?__biz=MzU3MzgwNTU2Mg==&mid=2247504783&idx=1&sn=72aed147a459368ed934a007a2df3bc9&chksm=fd3e951aca491c0cade212390011eca7d8a68951fee6aa2844b8f4d14bcbd5b3ed26305d69dc&scene=21#wechat_redirect)
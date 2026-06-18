> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkWebUI指标解读|FlinkWebUI指标解读]]
---
title: Flink Web UI详解【忍不住收藏系列】
author: 3分钟秒懂大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247513952&idx=1&sn=5c93c69645103204d3a684b62d6b6bd6&chksm=c11613ee8c2e7ff1d86201e6b1e8c594aaef8ad7a94028110b0a95a07234b30d1c0c502defea&mpshare=1&scene=24&srcid=1224oTgQgMUj63W4c1trLCgv&sharer_shareinfo=d02a9f6f1e168e90dff503ce0af0edbc&sharer_shareinfo_first=d02a9f6f1e168e90dff503ce0af0edbc#rd
---

```
点击上方公众号进入 3分钟秒懂大数据 主页

然后点击右上角 “设为标星” 

比别人更快接收硬核文章
```

大家好，我是土哥，一位搞大数据开发的程序员。

最近有小伙伴问到Flink Web UI的参数怎么看，今天找来资料来个详解，后面夹杂些了小伙伴们遇到的问题，通过官方介绍顺便做个解答。

参考简书

https://www.jianshu.com/p/70fbf632887e

### 1.提交flink 任务

##### flink cli 提交flink job任务shell如下:

###### 简易提交脚本如下，其中jm内存设定为1024M，tm内存设定为4096M，slot插槽为2。

```
#!/usr/bin/env bash
proj_home=$(cd `dirname $0`/.. ; pwd)

env=$1
savepoint_path=$2
yarn_name=WordCount
class_name=com.bigdata.rt.etl.WordCount
jm_mem=1024
tm_mem=4096
slot_num=2
par_num=5

if  [ ! -n '${savepoint_path}' ] ; then
echo 'Starting Flink job without savepoint'
flink run \
--detached \
--jobmanager yarn-cluster \
--yarnname '${yarn_name}-${env}' \
--yarnjobManagerMemory ${jm_mem} \
--yarntaskManagerMemory ${tm_mem} \
--yarnslots ${slot_num} \
--parallelism ${par_num} \
--class ${class_name} \ 
-yD env.java.opts.jobmanager='-Djob.name=${yarn_name}-JM' \
-yD env.java.opts.taskmanager='-Djob.name=${yarn_name}-TM' \
/flink/job/etl/test.jar
else
echo 'Starting Flink job with savepoint ${savepoint_path}'
flink run \
--detached \
--jobmanager yarn-cluster \
--yarnname '${yarn_name}-${env}' \
--yarnjobManagerMemory ${jm_mem} \
--yarntaskManagerMemory ${tm_mem} \
--yarnslots ${slot_num} \
--parallelism ${par_num} \
--class ${class_name} \
-s ${savepoint_path} \
-yD env.java.opts.jobmanager='-Djob.name=${yarn_name}-JM' \
-yD env.java.opts.taskmanager='-Djob.name=${yarn_name}-TM' \
/flink/job/etl/test.jar
fi

sleep 10s
web_frontend=`yarn application -appStates RUNNING -list | grep ${yarn_name} |   awk -F ' ' '{print $NF}'`
echo 'Web frontend listening at ${web_frontend}'
```

### 2.Flink Dashboard简介

### 3.Flink Running Jobs简介

##### Overview ui

##### Exception ui

##### TimeLine ui

##### Checkpoints ui

简介如下：

```
Checkpoint Counts   Triggered: 256  In Progress: 0  Completed: 256   Failed: 0 Restored: 0
      Checkpoint统计信息
      Triggered: 256：已触发的检查点数量。
      In Progress: 0：当前进行中的检查点数量。
      Completed: 256：成功完成的检查点总数。
      Failed: 0：失败的检查点总数
      Restored: 0：重启的检查点总数

Latest Completed Checkpoint ID: 256  Completion Time: 2022-01-12 15:28:31 End to End Duration: 41ms  Checkpointed Data Size: 11.9 KB
        Latest Completed Checkpoint：最后完成的Checkpoint的信息
        Completion Time:   最后完成检查点的时间
        End to End Duration:  端到端的运行时间
        Checkpointed Data Size: checkpoint数据量

Checkpoint Detail:Path: hdfs:/flink/flink-checkpoints/d4e087316e4354dc3f983864ab66c993/chk-260   Discarded: - Checkpoint Type: aligned checkpoint
        Checkpoint Detail:checkpoint详细信息
        Path: checkpoint 持久化路径
        Discarded: 丢弃
        Checkpoint Type: aligned checkpoint  checkpoint类型(对齐)
```

Operators Checkpoint ui 简介如下:

Checkpoint History ui 简介如下:

Checkpoint Summary ui 简介如下:

##### Operators ui

##### Backpressured 与 Busy

Backpressured：上游数据源生成数据的速度如果超过下游算子的处理能力，就会触发反压机制来控制数据流的速率，以防止下游算子因处理不过来而被淹没，进而导致性能下降或系统崩溃。

Busy：Busy (max)是衡量任务繁忙程度的一个指标，它表示任务正在处理的数据量占总吞吐量的比例。

Idle：代表空闲程度。

### 4.问题：Taskmanager的 managed memory满了

从官方文档和 Flink 源码上来看，托管内存主要有三大使用场景：

1. 批处理算法，例如排序、HashJoin 等。他们会从 Flink 的 MemoryManager 请求内存片段（MemorySegment），而 MemoryManager 则会调用 UNSAFE.allocateMemory 分配堆外内存。
2. RocksDB StateBackend，Flink 只会预留一部分空间并扣除预算，但是不介入实际内存分配。因此该类型的内存资源被称为 OpaqueMemoryResource. 实际的内存分配还是由 JNI 调用的 RocksDB 自己通过 malloc 函数申请。
3. PyFlink。与 JNI 类似，在与 Python 进程交互的过程中，也会用到一部分托管内存。

**上一篇热门技术文章：**[分布式存储系统Kudu与HBase的分析与对比](https://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247513795&idx=1&sn=f61fb599320a6786f7432c717b97585c&scene=21#wechat_redirect)

end

**增值服务：简历修改|面试辅导|Flink资料|**模拟面试****

你好，我是土哥，计算机硕士毕业，现某大厂资深大数据开发工程师。出生在一个 18 线开外的小村庄，通过自己努力毕业一年在新一线城市买房，在社招、校招斩获 25 家中大厂 offer。

24年接近尾声，很多公司已经开启了**年前面试-年后入职**的流程。如果你想跳槽，但苦于一个人孤军奋战、无人指导、复习无从下手，或者不擅长写简历，手上只有拿不出手的毫无难点亮点的项目经历...

那么我的建议是多和身边的大佬沟通，哪怕是付费咨询，只要你能从他身上学到经验，那就是值得的。如果身边没有这样的人，那么我就毛遂自荐一下吧，毕竟，茫茫网络你能看到这篇文章何尝不是一种命运安排。

[土哥社招参加 28 场面试，100% 通过率，拿到全部 offer！](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247511408&idx=1&sn=beb292ab97ada3ee486511bfe503117d&chksm=c01914cff76e9dd90fd81857805a57aadcf4fa0a3ce731e5939d8651ed9bac561dba6bb7e03a&scene=21#wechat_redirect)

[土哥这半年的悲惨人生，经历过被鸽 offer，最终触底反弹~](http://mp.weixin.qq.com/s?__biz=Mzg5NDY3NzIwMA==&mid=2247510455&idx=1&sn=9cccfbebca3d2ee9d72538d73dd6fe74&chksm=c0191008f76e991e1760857f7c8a75deb1e0231ce8ba189eaf280c84e8a5e65f7f0197988ff1&scene=21#wechat_redirect)

需要任一增值服务或进大厂交流求职群

都欢迎加土哥微信（备注来意）

点赞和在看就是最大的支持❤️
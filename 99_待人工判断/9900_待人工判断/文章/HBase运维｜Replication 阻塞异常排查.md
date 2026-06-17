---
title: HBase运维｜Replication 阻塞异常排查
author: HBase技术社区
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247491964&idx=1&sn=694f268d1bc9e66bf0ae03490eb8ba35&chksm=feb61601c9c19f17fddf8eea625c66fccde6611063f1e86fb818ef6a0cb63314bf460e296df4&mpshare=1&scene=24&srcid=0519tOCAx9UqrMdz0952rZ3e&sharer_sharetime=1684458437207&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

## 目录导读

* 目录导读
* 1. 同步阻塞的异常描述
* 2. 死磕同步阻塞的原因

+ 2.1 ReplicationSource相关线程的jstack日志分析
+ 2.2 简述Replication的执行流程
+ 2.3 定位同步阻塞的代码
+ 2.4 使用arthas热部署代码助力问题定位
+ 2.5 隐蔽BUG导致的同步阻塞问题排查
+ 2.6 totalBufferUsed变量计算不严谨导致的同步阻塞问题修复

* 3. 总结

> 本篇文章记录了一次HBase同步（replication）阻塞的详细排查过程，内容包括：使用jstack分析堆栈，使用arthas的mc/redefine命令对线上正在运行的代码进行热部署，并结合RegionServer中Replication相关的日志，定位到导致同步阻塞的原因，最后在ISSUE列表中找到对应的修复方法。同时，结合源码，梳理了Replication的流程，对这块核心功能有了更加深刻的理解。

## 1. 同步阻塞的异常描述

从接触HBase运维开始，Replication阻塞的问题就成了一直以来的“心病”，`一用同步，必遇阻塞`。

Replication的流程较为复杂，就像一个黑盒子，当阻塞发生时，从RS的日志中暴露出来的有用信息又很少，如果不潜心把这块的源码搞通，会非常不利于阻塞问题的定位和解决。

同时，HBase的Replication又是用户重度依赖的功能，经常被应用于数据迁移、HA、在线库数据同步到离线库，进行OLAP分析等场景中。

细数开源的实现方案，只有HBase自身的Replication功能可满足两个（及以上）HBase集群之间的近实时数据同步需求。

在正常情况下，管理员创建好peer，配置好需要同步的表，主集群通过写WAL提交的数据，都能以较低的延迟及时传输到备集群，但在一些极端的时刻，同步流程会阻塞，如下图：

同步阻塞

HBase2.x的HMaster的web ui界面上，可以直观看到Replication的最新状态，在更低版本中，则需要关注一些Replication相关的jmx指标。

以第一个阻塞严重的peer的RS-129节点为例，该RS的同步延迟已达243h之久，logQueue中积压的待同步的WAL log文件多达1280个。

如果长时间不处理这种同步阻塞的异常情况，可能会引发如下严重后果：

1. 主备集群数据不一致，这在HA，以及对数据一致性要求很高的场景中是致命的
2. 给主集群HDFS带来存储压力，积压的WAL log 不会被HMaster的定时线程清理，造成WALs或oldWALs目录空间占用越来越高，由于oldWALs目录下文件数限制，可能会导致HMaster无法启动，如果没记错的话，与参数`dfs.namenode.fs-limits.max-directory-items`有关
3. 给ZK带来压力，ZK中保留了peer/rs/每个参与replication的WAL log的进度，当同步积压时，会导致大量与Replication相关的znode path不能被及时清理，严重将导致ZK崩溃。如这篇博客中所说：

   https://cloud.tencent.com/developer/article/1353079

打开RS-129的url连接，在界面最下方有当前RS的详细同步状态，如下图：

Current Log

图中可以看到阻塞的peer id，正在被处理的wal文件地址（看文件时间戳，是一周前的），以及logQueue队列的长度，当前同步进度的offset值等。

## 2. 死磕同步阻塞的原因

### 2.1 ReplicationSource相关线程的jstack日志分析

通过上述背景，我们已知某个RS节点正处于同步阻塞状态，导致这种现象的发生，大多数与源集群的wal log数据写备集群的过程中出现了异常有关，此时源集群作为Client，目标集群作为Server，在目标表误被禁用（或删除）、处于RIT状态或目标集群负载高、网络抖动发生时，可能都会导致目标表的数据写入报错，继而引发源集群数据同步的过程重试，然后陷入不断的循环重试，对外则表现出来阻塞的现象。

在RS-129的日志中搜索ReplicationSource或ReplicationSink等关键字，有些异常比较明显，有些异常则非常隐蔽（需要打开对应类的DEBUG或TRACE级别的日志）。

对于同步持续阻塞的RS节点，同时日志中又没有明显异常输出的情况下，我们的第一反应应该是去看RS进程对应的jstack日志，如下图我截取的一部分replicationSource-wal-reader和replicationSource.shipper线程的堆栈：

jstack

replicationSource.shipper线程负责将entryBatchQueue中的待同步数据包发送到备集群，该线程此时处于TIMED\_WAITING状态，在HBaseInterClusterReplicationEndpoint.sleepForRetries方法内部正sleep呢，HBaseInterClusterReplicationEndpoint可以理解为一个插件，负责执行具体的entry数据包的发送行为。

replicationSource-wal-reader线程负责读取待同步的wal log，包装成entryBatch后，put到entryBatchQueue中。

这是一个典型的生产者和消费者模型，下文会再结合源码，详细梳理这两个角色内部的工作细节。

### 2.2 简述Replication的执行流程

通过jstack日志定位阻塞代码之前，简单说下Replication的过程（这里不涉及详细的代码分析）。

同步流程

在创建Peer时，每一个Regionserver会创建一个ReplicationSource线程，ReplicationSource首先把当前正在写入的HLog保存在复制队列中，然后在Regionserver上注册一个Listener，用来监听HLog Roll操作。

如果Regionserver做了HLog Roll操作，那么ReplicationSource收到这个操作后，会把这个HLog分到对应的walGroup-Queue里面，同时把HLog文件名持久化到Zookeeper上，这样重启后还可以接着复制未完成的HLog

每个walGroup-Queue后端有一个ReplicationSourceWALReader的线程，不断的从Queue中取出一个Hlog，然后把HLog中的entry逐个读取出来，放到一个名为entryBatchQueue的队列中

每个entryBatchQueue的队列后端有一个ReplicationSourceShipper的线程，不断的从Queue中读取Log Entry，交给Peer的ReplicationEndpoint。ReplicationEndpoint把这些entry打包成一个replicationWALEntry操作，通过RPC发送到Peer集群的某个RegionServer上。

对应Peer集群上的RegionServer把replicationWALEntry解析成若干个Batch操作，调用batch接口执行。待RPC调用成功之后，ReplicationSourceShipper会更新最近一次成功复制的HLog Position到Zookeeper以便RegionServer重启后，下次还能找到最新的Position开始复制。

备注：上述内容参考：https://www.jianshu.com/p/9d7008028ab0和《HBase原理和实践》

### 2.3 定位同步阻塞的代码

回看上面的堆栈截图，通过阻塞的walGroup的ID在堆栈文件中匹配到关键信息，replicationSource.wal-reader线程负责读wal日志，replicationSource.shipper负责发送entry，最终定位到导致阻塞的代码位置：

HBaseInterClusterReplicationEndpoint.sleepForRetries，然后梳理HBaseInterClusterReplicationEndpoint的方法调用栈，定位代码（我的HBase版本是：2.2.6）：

```
HBaseInterClusterReplicationEndpoint.replicate(HBaseInterClusterReplicationEndpoint.java:624)
```

replicate-error

`HBaseInterClusterReplicationEndpoint.replicate`的代码执行过程中遇到了异常，这个异常没有对应匹配的处理逻辑，导致一直在sleepForRetries方法内sleep，每次sleep 5分钟（这里的5min是sleep的最大值，所以进行了多次jstack，堆栈都显示卡在sleepForRetries(HBaseInterClusterReplicationEndpoint.java:252)）上，且默认的日志级别是INFO，导致这里sleepForRetries方法内部的信息不会输出，就不利于问题的排查。

打开HBaseInterClusterReplicationEndpoint类TRACE级别的日志，在RS web ui上设置类org.apache.hadoop.hbase.replication.regionserver.HBaseInterClusterReplicationEndpoint的日志级别是：TRACE。

loglevel-trace

日志中会有Since we are unable to replicate的信息输出，同时，在HBaseInterClusterReplicationEndpoint.replicateEntries的方法内部，会输出同步失败的原因，把捕获到的IOException的详情打印出来，也是trace级别的日志，如下图：

error-log

在日志中通过关键词`Failed replicating batch`匹配到的异常信息如下：

```
 2023-05-16 16:34:45,468 TRACE [SinkThread-2070] regionserver.HBaseInterClusterReplicationEndpoint: [Source for peer 阻塞的peer id]: Failed replicating batch 757168047  
org.apache.hadoop.hbase.ipc.RemoteWithExtrasException(org.apache.hadoop.hbase.client.RetriesExhaustedWithDetailsException): org.apache.hadoop.hbase.client.RetriesExhaustedWithDetailsException: Failed 423 actions: Operation rpcTimeout: 423 times, servers with issues: 目标集群RS节点host,60020,1679648441407  
......
```

由日志内容可知，Source端在replicate数据的过程中发生异常，可能是备集群或备集群的表有问题，导致这里的写入报错，下文会继续分析。

### 2.4 使用arthas热部署代码助力问题定位

打开TRACE日志级别后，如果日志输出的太多，干扰问题的定位，可以在源码基础上追加日志输出、打印变量等其他自定义的逻辑，然后使用arthas中的`jad/mc/redefine`命令来进行线上代码热更，这样在保留异常现场的同时，又可以在当前RS进程加载的字节码文件中插入自定义的行为，以方便排障。

参考博客：https://hengyun.tech/arthas-online-hotswap/

线上代码热更的命令使用举例：

```
[arthas@2230807]$ jad --source-only org.apache.hadoop.hbase.replication.regionserver.HBaseInterClusterReplicationEndpoint > /tmp/HBaseInterClusterReplicationEndpoint.java  
  
# 把当前进程加载的类反编译成.java文件，有时候反编译的文件修改后，再编译会报错，不如直接把源码贴过来修改  
  
vim /tmp/HBaseInterClusterReplicationEndpoint.java  
# 增加自己的日志输出或方法内局部变量打印的逻辑  
  
[arthas@2230807]$ sc -d *HBaseInterClusterReplicationEndpoint | grep classLoaderHash  
 classLoaderHash   764c12b6  
 # 获取HBaseInterClusterReplicationEndpoint这个类是由哪个classLoader加载的，后续命令需要764c12b6这个地址  
   
[arthas@2230807]$ mc -c 764c12b6 /tmp/HBaseInterClusterReplicationEndpoint.java -d /tmp  
Memory compiler output:  
/tmp/org/apache/hadoop/hbase/replication/regionserver/HBaseInterClusterReplicationEndpoint.class  
Affect(row-cnt:1) cost in 4782 ms.  
# 使用mc命令把.java文件编译成.class文件，结果输出到/tmp/org/apache/hadoop/hbase/replication/regionserver/HBaseInterClusterReplicationEndpoint.class  
  
[arthas@2230807]$ redefine /tmp/org/apache/hadoop/hbase/replication/regionserver/HBaseInterClusterReplicationEndpoint.class  
# 使用redefine命令热部署.class文件
```

然后观察日志输出，验证热更新代码生效。

在2.3小节中，通过调整日志级别，已知在replicate的过程中，遇到了报错，然后在Peer集群的RS节点上，检查与之能对应上的Server端的日志：

```
2023-05-16 17:21:28,217 WARN  [RpcServer.default.RWQ.Fifo.write.handler=20,queue=0,port=60020] ipc.RpcServer: (responseTooSlow): {"call":"Multi(org.apache.hadoop.hbase.shaded.protobuf.generated.ClientProtos$MultiRequest)","multi.gets":0,"starttimems":"1684228878211","responsesize":"617157","method":"Multi","param":"region\u003d 正同步的表名,404629703373950000,1601338035851.0fa841189219116e216e873228d73380., for 423 action(s) and 1st row key\u003d407563170009","processingtimems":10005,"client":"客户端IP:13376","queuetimems":0,"multi.servicecalls":0,"class":"HRegionServer","multi.mutations":423}
```

从日志中可以看到，在写表- region的过程中有一些问题，需要检查该表的状态，判断是否有隐式RIT等异常情况，从而影响了Sink端写表的行为，在修复表的异常情况后，积压的logQueue开始慢慢减少了。

### 2.5 隐蔽BUG导致的同步阻塞问题排查

2.4小节记录的同步阻塞问题产生原因是，Peer集群写目标表出现异常，导致Source端陷入循环重试。这种阻塞情况比较常见，也比较好解决；另一种则是隐蔽BUG导致的同步阻塞，与前者的不同表现在阻塞的堆栈不一样。

下图展示了另一种Peer阻塞的现象，在HMaster的replication tab页面中即可获取这些数据。

| PeerId | WalGroup | Current Log | Size | Queue Size | Offset |
| --- | --- | --- | --- | --- | --- |
| peer1 | node-227 | hdfs:///hbase/WALs/node-227.hadoop,60020,1676962251329/node-227.hadoop%2C60020%2C1676962251329.1678333914160 | 23.1K | 1655 | -1 |

node-227堆栈如下：

```
"RS_REFRESH_PEER-regionserver/node-227:60020-0.replicationSource,peer1.replicationSource.shippernode-227.hadoop%2C60020%2C1676962251329,peer1" #11847703 daemon prio=5 os_prio=0 tid=0x00007fdfb7a09000 nid=0x2207 waiting on condition [0x00007fc33045b000]  
   java.lang.Thread.State: TIMED_WAITING (parking)  
        at sun.misc.Unsafe.park(Native Method)  
        - parking to wait for  <0x00007fd90b727f08> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)  
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)  
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)  
        at java.util.concurrent.LinkedBlockingQueue.poll(LinkedBlockingQueue.java:467)  
        at org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceWALReader.poll(ReplicationSourceWALReader.java:364)  
        at org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceShipper.run(ReplicationSourceShipper.java:107)  
  
"RS_REFRESH_PEER-regionserver/node-227:60020-0.replicationSource,peer1.replicationSource.wal-reader.node-227.hadoop%2C60020%2C1676962251329,peer1" #11847704 daemon prio=5 os_prio=0 tid=0x00007fbe21ef0000 nid=0x2206 waiting on condition [0x00007fbe8e3cd000]  
   java.lang.Thread.State: TIMED_WAITING (sleeping)  
        at java.lang.Thread.sleep(Native Method)  
        at org.apache.hadoop.hbase.util.Threads.sleep(Threads.java:148)  
        at org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceWALReader.checkQuota(ReplicationSourceWALReader.java:334)  
        at org.apache.hadoop.hbase.replication.regionserver.ReplicationSourceWALReader.run(ReplicationSourceWALReader.java:140)
```

打开ReplicationSourceShipper类的DEBUG日志，检查到有如下输出：

```
DEBUG [RS_REFRESH_PEER-regionserver/node-227:60020-0.replicationSource,peer1.replicationSource.shippernode-227.hadoop%2C60020%2C1676962251329,peer1] regionserver.ReplicationSourceShipper: Shipper from source bddx_hb2 got entry batch from reader: null
```

查看对应源码，ReplicationSourceWALReader.poll(ReplicationSourceWALReader.java:364)，在ReplicationSourceShipper的run方法内，从wal-reader中拿到的entryBatch一直是0，源码片段如下：

```
WALEntryBatch entryBatch = entryReader.poll(getEntriesTimeout);  
LOG.debug("Shipper from source {} got entry batch from reader: {}",  
    source.getQueueId(), entryBatch);  
if (entryBatch == null) {  
  // 这里拿到的entryBatch为null，说明entryBatchQueue中没有数据了，wal-reader不再生产数据了，  
  // 导致shipper一直在此处循环  
  continue;  
}
```

再分析replicationSource.wal-reader的堆栈，由源码可知，wal-reader线程卡在checkQuota方法上，ReplicationSourceWALReader.checkQuota(ReplicationSourceWALReader.java:334)

check-quota

由于!checkQuota() == true，导致wal-reader run方法中循环不会往下进行了，也就不会再读取新的wal log，也就不会产生新的entryBatch了，导致Shipper线程从entryBatchQueue中获取不到新的entryBatch，也就导致了同步阻塞。所以，阻塞的关键在于checkQuota方法，深入到该方法内部。

```
//returns false if we've already exceeded the global quota  
private boolean checkQuota() {  
  // try not to go over total quota  
  if (totalBufferUsed.get() > totalBufferQuota) {  
    Threads.sleep(sleepForRetries);  
    return false;  
  }  
  return true;  
}
```

totalBufferQuota的值由参数`replication.total.buffer.quota`控制，默认值是：256 \* 1024 \* 1024（256m）。

在没有打入HBASE-24779的时候，可以在checkQuota方法内追加如下日志输出，然后使用arthas的热部署更新代码：

```
LOG.warn("peer={}, can't read more edits from WAL as buffer usage {}B exceeds limit {}B",  
              this.source.getPeerId(), totalBufferUsed.get(), totalBufferQuota);
```

代码热部署完成后，获取到如下日志输出：

```
2023-05-17 11:23:42,916 WARN  [RS_REFRESH_PEER-regionserver/阻塞节点host:60020-0.replicationSource,阻塞peer ID.replicationSource.wal-reader.阻塞节点host%2C60020%2C1676962251329,阻塞的peerID] regionserver.ReplicationSourceWALReader: peer=阻塞的peerID, can't read more edits from WAL as buffer usage 270812690B exceeds limit 268435456B
```

totalBufferUsed已使用了258M，超过了默认限制的buffer size：256M，导致checkQuota()方法一直卡住。

分析了源码，梳理了totalBufferUsed变量的数据变化历程，作如下简单记录：

1.`totalBufferUsed`这个原子变量在ReplicationSourceManager类中被初始化，被ReplicationSourceWALReader、ReplicationSourceShipper、ReplicationSource类共享

```
public class ReplicationSourceManager implements ReplicationListener {  
  private AtomicLong totalBufferUsed = new AtomicLong();  
    
  public AtomicLong getTotalBufferUsed() {  
    return totalBufferUsed;  
  }  
  ......  
}
```

2.在ReplicationSourceWALReader类中，run方法内，会先构造一个WALEntryStream entryStream对象，entryStream对象中包含了logQueue，logQueue的核心数据结构是`Map<String, PriorityBlockingQueue<Path>>` queues，queues中分walGroupId保存着每一个walGroupId下待同步的wal文件列表，在Replication的指标上的体现如下图：

replication-status

通过entryStream对象创建出来WALEntryBatch batch，WALEntryBatch内维护了一个`List<Entry> walEntries`，其中保存着一个个从wal中解析出来的Entry对象，entry应该是数据同步的最小单位了，里面包含了具体的数据。往WALEntryBatch中塞Entry对象的逻辑，包含在run方法内对readWALEntries(entryStream, batch);的调用中，深入此方法内，再探addEntryToBatch(batch, entry)方法。

调用`Entry entry = entryStream.next();`，持续从entryStream中解析出来一个个entry对象，经过`filterEntry(entry)`方法过滤，把一个个满足同步条件的entry对象添加到WALEntryBatch的walEntries中，同时会把每一个entry的size，累加到totalBufferUsed上，如果totalBufferUsed超过默认的256阈值，则会中断这个批次的读wal循环，以起到一个缓冲作用。

最后，通过执行`entryBatchQueue.put(batch);`，把包含了<=256M entry数据包的WALEntryBatch put到entryBatchQueue中，等待ReplicationSourceShipper去消费entryBatchQueue队列中的数据。

```
class ReplicationSourceWALReader extends Thread {  
  private AtomicLong totalBufferUsed;  
  // 调用ReplicationSourceManager.getTotalBufferUsed()方法为totalBufferUsed赋初始值  
  this.totalBufferUsed = source.getSourceManager().getTotalBufferUsed();  
    
  private boolean acquireBufferQuota(long size) {  
    // 累加batch size的值给totalBufferUsed  
    return totalBufferUsed.addAndGet(size) >= totalBufferQuota;  
 }  
    
  protected final boolean addEntryToBatch(WALEntryBatch batch, Entry entry) {  
    // 包装WALEntryBatch的过程中，调用acquireBufferQuota  
    boolean totalBufferTooLarge = acquireBufferQuota(entrySizeExcludeBulkLoad);  
  }  
    
  protected void readWALEntries(WALEntryStream entryStream, WALEntryBatch batch){  
    ......  
  if (entry != null) {  
     ......  
     removeEntryFromStream(entryStream, batch);  
     if (addEntryToBatch(batch, entry)) {  
         // 不断去包装entry  
       break;  
     }  
   }  
    ......  
  }  
    
  @Override  
  public void run() {  
    while (isReaderRunning()) {  
      ......  
      if (!source.isPeerEnabled()) {  
            Threads.sleep(sleepForRetries);  
            continue;  
      }  
      // 问题就出在这里，checkQuota一直不通过，导致entryBatchQueue未再有新的entryBatch添加进去，  
      // 就导致消费端一直获取不到数据  
      if (!checkQuota()) {  
            continue;  
      }  
      if (!batch.isEndOfFile()) {  
            readWALEntries(entryStream, batch);  
            currentPosition = entryStream.getPosition();  
      }  
      entryBatchQueue.put(batch);  
      ......  
    }  
  }  
}
```

3.ReplicationSourceWALReader类作为entryBatchQueue队列的生产方，不断生产entryBatch数据，然后累加缓冲区totalBufferUsed的值，那么ReplicationSourceShipper作为消费端，一定会，消费完一个entry后，把缓冲区的空间还回去，以减少totalBufferUsed的值。

在ReplicationSourceShipper类中，run方法的内部，通过entryReader.poll方法（超时时间20s）从entryBatchQueue中持续取WALEntryBatch对象，如果entryBatchQueue为空，取到的entryBatch就为null，通过这里的日志输出也能证明：

```
LOG.debug("Shipper from source {} got entry batch from reader: {}", source.getQueueId(), entryBatch);
```

对应的DEBUG的日志：

```
2023-05-17 10:28:56,676 DEBUG [RS_REFRESH_PEER-regionserver/阻塞RS的host:60020-0.replicationSource,阻塞的peerID.replicationSource.shipper阻塞RS的host%2C60020%2C1676962251329,阻塞的peerID] regionserver.ReplicationSourceShipper: Shipper from source 阻塞的peerID got entry batch from reader: null
```

正常情况下，如果获取到的WALEntryBatch对象不为空，会进入到`shipEdits(entryBatch);`方法内部，在shipEdits方法内，会调用`source.getReplicationEndpoint().replicate(replicateContext);`把entryBatch中的entry数据包，发送到备集群，replicate方法暂不展开叙述，如果replicate方法返回true，也即完成了该entryBatch的数据同步。

后续逻辑是，先计算出此batch的size：`int sizeExcludeBulkLoad = getBatchEntrySizeExcludeBulkLoad(entryBatch);`，根据变量名就可知，这个size排除掉了bulkload同步的数据，然后调用`source.postShipEdits(entries, sizeExcludeBulkLoad);`，实现了对totalBufferUsed变量的减少，postShipEdits方法的源码展开如下：

```
class ReplicationSource {  
  @Override  
  //offsets totalBufferUsed by deducting shipped batchSize.  
  public void postShipEdits(List<Entry> entries, int batchSize) {  
    if (throttler.isEnabled()) {  
      throttler.addPushSize(batchSize);  
    }  
    totalReplicatedEdits.addAndGet(entries.size());  
    // 此处，对totalBufferUsed变量进行减值操作  
    totalBufferUsed.addAndGet(-batchSize);  
  }  
}  
```

### 2.6 totalBufferUsed变量计算不严谨导致的同步阻塞问题修复

根据2.5小节的源码分析，在正常情况下，totalBufferUsed不会出现容量超出的情况，那么导致此异常出现的可能原因是：有BUG导致totalBufferUsed的值没有被正确计算。

根据关键词`totalBufferUsed`在HBase的ISSUE列表中搜索：

issue-list

**HBASE-27785**

优化措施，对totalBufferUsed重新封装并集中在ReplicationSourceManager中，避免此变量的逻辑分散在多个类中，导致一些问题。

**HBASE-27778**

重要BUG修复，关键信息是：ReplicationSourceWALReader.readWALEntries读一个new wal，累加totalBufferUsed，此过程发生异常时，整个`WALEntryBatch`可能不会被放入`ReplicationSourceWALReader.entryBatchQueue`中，导致totalBufferUsed已经累加的值不会被释放。

**HBASE-27354**

重要BUG修复，修复了`totalBufferUsed`变量累加的值，没有被清除的另一种情况，但未被官方合并到主分支。

**HBASE-24813**

重要BUG修复，摘取的一些关键信息如下：

peer被移除后，与peer相关的**ReplicationSource**实例被杀死，它可能会导致**ReplicationSourceManager.totalBufferUsed** 不一致。

如果**ReplicationSourceWALReader**已经将一些entries放在其队列中以供**ReplicationSourceShipper** 处理 ， 但在它可以被处理之前，此时删除kill掉了peer，则可能会发生这种情况。

当 **ReplicationSourceWALReader** 线程将entry添加到队列时，它会将 **ReplicationSourceManager.totalBufferUsed**增加entry大小的总和。当 **ReplicationSourceShipper** 读取这些条目时 **，** **ReplicationSourceManager.totalBufferUsed** 随之减少。

应该 在**ReplicationSource**终止时减少**ReplicationSourceManager.totalBufferUsed** ，否则那些未处理的entry大小将无限期地消耗 **ReplicationSourceManager.totalBufferUsed \_\_\*，除非 RS 重新启动**。对于具有多个peer的部署，或者如果添加了新的peer，这可能是一个问题。

在修复HBASE-24813之前，需要先确保HBASE-25117已经应用上。https://issues.apache.org/jira/browse/HBASE-25117，修复了ReplicationSourceShipper 线程无法完成终止的问题。

**HBASE-17314**

此优化改进就是增加了`totalBufferUsed`这个缓冲区变量，以限制所有peer复制缓冲区的总大小，防止在有很多peer的场景中，服务器出现OOM。修复版本：1.5.0、2.0.0，非影响版本可忽略。

同时https://issues.apache.org/jira/browse/HBASE-24779也提到了checkQuota阻塞的问题。

针对HBase-2.2.6版本，最终引入的修复有：`HBASE-25117`、`HBASE-24813`、`HBASE-27778`

## 3. 总结

引发HBase同步阻塞的原因大致有两个，一是ReplicationSink在写Peer集群中的个别表时出现了异常，这种异常可能与集群负载、网络抖动，或表本身的异常（如：误禁用、删除，RIT等）有关，这种情况比较常见，也较易解决，大多数会直接体现在RS日志中；另一种则是隐蔽BUG，需要通过jstack，并结合源码来分析，必要时可以尝试arthas代码热更利器，一般也能定位到导致阻塞的原因。然后在ISSUE中搜索，或在高版本的git history中去寻找解决方法。如果是新产生的问题，可以与社区邮件交流，社区微信群不活跃，但邮件是真活跃。当然，另一层境界就是自己提PR了，本人还暂未达到，在此不细言。

本文使用的图例和部分源码解读，来源链接已包含在了文章内部，就不再在文末单独列出。再次感谢那些献身开源，无私分享的前辈大佬，给我们充满迷雾的求知之路上增添了许多星光。
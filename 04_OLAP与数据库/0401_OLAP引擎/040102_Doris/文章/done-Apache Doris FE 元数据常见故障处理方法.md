> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040102_Doris/040102_核心知识点/DorisFE元数据恢复边界|DorisFE元数据恢复边界]]
---
title: Apache Doris FE 元数据常见故障处理方法
author: 锋哥聊湖仓
date:
url: http://mp.weixin.qq.com/s?__biz=MzI4ODMyNTcwMw==&mid=2247485220&idx=1&sn=0329acc2f754072d0d45bbce751e9dbe&chksm=ebc16e0cdcb6e71aa48abb5a5eee6299c7cef1d671f7898778b521222ba2189d94855851e132&mpshare=1&scene=24&srcid=0913p57QNs3uJyFvTcjG5A06&sharer_sharetime=1663022323782&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

该处理方法适用于 Apache Doris 0.14.7 及之后所有版本

基本下面这个方法，除了元数据损坏，可以解决99%的 Doris 元数据运维问题

常见问题

## FE （Follower）挂掉

出现类似下面的错误

解决方案：

### 单个 FE （Follower）情况

+ 如果是单个FE，直接在conf/fe.conf 中加入 `metadata_failure_recovery=true`
+ 在访问正常之后，将上面元数据恢复模式设置成false，或者将这个配置项注释掉
+ 最后重启 FE
+ 如果有 Observer

1. 首先停掉所有的 Observer（正常情况下 Follower出问题，Observer 也会挂掉）
2. 使用上面元数据恢复模式，将Follower 恢复正常之后
3. 在MySQL 客户端或者命令行下连接Follower（Master）节点
4. 执行下面的命令

```
ALTER SYSTEM DROP OBSERVER "OBSERVER_IP:PORT";

这里将所有的Observer从集群中删除掉
OBSERVER_IP：你要删除的Observer 节点IP
PORT：fe.conf 中的 edit_log_port，默认9010
```

* 然后到Observer 节点上，将Observer 元数据目录清空（可以先备份）
* 然后使用下面的命令启动Observer

```
sh bin/start_fe.sh --helper master_fe_ip:port --daemon

master_fe_ip：你要Master FE 节点IP,如果是单个Follower就是你的这个Follower节点IP
port：fe.conf 中的 edit_log_port，默认9010
```

1. 在MySQL 客户端或者命令行下连接Follower（Master）节点执行下面的命令

```
ALTER SYSTEM ADD OBSERVER "OBSERVER_IP:PORT";

这里是你刚才启动Observer节点加入到集群中那个
OBSERVER_IP：你要加入的Observer 节点IP
PORT：fe.conf 中的 edit_log_port，默认9010
```

1. 查看FE运行状态

```
show fontends;

查看FE(Follower 和你刚才添加的 Observer 运行状态是否正常)
你也可以通过查看你刚才添加的Observer 的日志log/fe.log 观察是否启动正常
```

### 多个 FE （Follower）情况

+ 在所有 FE 的元数据目录下查看image/image.xxxx
+ 找出image.xxxx 这个xxxx 数字最大的这个节点，这个数字最大说明这个节点的元数据是最新的
+ 然后按照上面单个 Follower + 多个 Observer 的恢复流程进行操作，只不过 Observer 换成 Follower 即可。

## FE 因为没有配置 priority\_networks 启动错误

FE在启动的时候报类似下面的错误

```
java.io.IOException: the self host 172.31.26.7 does not equal to the host in ROLE file 172.17.0.1. You need to set 'priority_networks' config in fe.conf to match the host 172.17.0.1     at org.apache.doris.catalog.Catalog.getClusterIdAndRole(Catalog.java:903)     at org.apache.doris.catalog.Catalog.initialize(Catalog.java:805)     at org.apache.doris.PaloFe.start(PaloFe.java:125)     at org.apache.doris.PaloFe.main(PaloFe.java:63)
```

解决方案：
删除 doris-meta目录下的所有目录及文件，修改 fe.conf 里面的 priority\_networks,重启即可解决

## Apache Doris 0.14.7 之前版本

针对 Doris 0.14.7 之前版本，出现元数据错误，将其他节点从集群中删除，在作为新的节点加入，可能会存在错误，加入不成功，同时会导致其他 FE 挂掉的情况，针对之前版本正确的做法请参考下面链接：

https://segmentfault.com/a/1190000040694998
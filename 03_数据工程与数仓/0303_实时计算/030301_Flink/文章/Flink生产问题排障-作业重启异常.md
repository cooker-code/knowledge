---
title: Flink生产问题排障-作业重启异常
author: 大大大大晴天
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwMDcxNjU3Nw==&mid=2247483654&idx=1&sn=34852f36b811fccd7e1afc8390d3591c&chksm=c16ab520133e93b405526f691a74f2a34d389f7578f0af50b0def3e0f666119c63720d2a6efb&mpshare=1&scene=24&srcid=0411wyG0xRqzjgg6iXoODulC&sharer_shareinfo=517a2cb3a5e520509f5b8651e1a0a332&sharer_shareinfo_first=517a2cb3a5e520509f5b8651e1a0a332#rd
---

一、背景

Flink版本为1.14.6，采用Flink on native k8s模式部署，以Application方式提交运行作业；另外，有一个Flink作业调度管理平台，每次作业停止操作都会在调度管理平台调用Flink的stop-with-savepoint脚本。

二、问题现象

一个Flink作业重启后，运行快速失败

三、问题排查

* 查看暂停日志

+ JM发起3次savepoint\_suspend的cp均失败（failed to complete checkpoint）
+ Flink作业被强制停止（JM日志报cannot found deployment xxxx）

* 查看启动日志

+ JM从sp恢复拉起作业，作业报错恢复失败（状态kakfa-offset数据过期）

* 查看作业的cp/sp

+ cp状态丢失，sp仅有一个半年前的状态

四、问题分析

* 现有调度平台Flink作业暂停逻辑分析

  1.使用flink stop-with-savepoint暂停作业（包含3次重试）

  2.检测作业状态，若未停止，则执行k8s的kubectl delete命令强制删除
* Flink1.14.6原生停止脚本逻辑分析

  1.执行sp操作

  2.删除cp数据

  3.优雅停止作业

  过程中存在异步操作，同时没有事务管理，初步评估在执行sp失败且删除cp成功的场景下有丢失最近的cp/sp概率
* CP失败常见原因

  1.资源不足：内存、磁盘、cpu/线程竞争

  2.背压：barrier无法及时传递，超时失败

  3.状态过大

  4.外部依赖故障：上下游组件异常阻塞

  5.配置不合理：cp太频繁、超时太短

  6.网络问题：网络长时间抖动延迟

五、问题结论

* 原生Flink1.14.6停止脚本存在小概率丢状态风险

六、解决方案：

* 完善Flink作业暂停的处理逻辑，保障有最近可用的cp/sp

1. 调用flink stop脚本前，先将最近成功的cp备份
2. 判断作业停止状态增加JM-deployment与TM-pod状态检测

* 加强Flink作业运行异常指标告警

1. cp连续失败告警
2. 作业重启次数告警
3. 数据消费延迟告警

七、总结

我们在使用开源组件技术的过程中，系统功能设计上尽量将组件作为可用的工具，多考虑整体的健壮性与鲁棒性。
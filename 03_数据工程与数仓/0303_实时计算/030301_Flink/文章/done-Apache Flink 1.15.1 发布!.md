---
title: Apache Flink 1.15.1 发布!
author: Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzI0NTIxNzE1Ng==&mid=2651225607&idx=1&sn=64f56bfd97bac3fd092f630d71f6a801&chksm=f2a33cecc5d4b5fa7e700596c35b11c37125b77ac08d4a81e49a0b18afc735d73474c83aa58a&mpshare=1&scene=24&srcid=0709aUF3ZT3Qj93XpqCMFtkd&sharer_sharetime=1657331218940&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_版本记录|版本记录]]


Apache Flink 社区很高兴地宣布 Flink 1.15 系列的第一个错误修复版本。

此版本包括 62 个错误修复、漏洞修复和 Flink 1.15 的小改进。您将在下面找到所有错误修复和改进的列表（不包括对构建基础架构和构建稳定性的改进）。有关所有更改的完整列表，请参阅： JIRA。

我们强烈建议所有用户升级到 Flink 1.15.1。

# 发布工件

## Maven 依赖项

```
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-java</artifactId>
  <version>1.15.1</version></dependency><dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-java</artifactId>
  <version>1.15.1</version></dependency><dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-clients</artifactId>
  <version>1.15.1</version></dependency>
```

## 二进制文件

您可以在更新的下载页面上找到二进制文件。

## Docker Images

* 库/flink（官方图片）
* apache/flink（ASF 存储库）

## Pypi

* apache-flink==1.15.1

# 发行说明

社区知道 1.15.0 引入的两个问题仍未解决。正在努力为 Flink 1.15.2 解决这些问题：

* [ FLINK-28060 ] - 在代理重启后，Kafka 提交检查点反复失败
* [ FLINK-28322 ] - DataStreamScanProvider 的新方法不兼容

## Bug

* [ FLINK-22984 ] - 使用 Python UDF 生成水印时出现 UnsupportedOperationException
* [ FLINK-24491 ] - 调度程序终止时可能不会归档 ExecutionGraphInfo
* [ FLINK-24735 ] - SQL 客户端因“无法添加不同类型的表达式来设置”而崩溃
* [ FLINK-26645 ] - Pulsar Source 订阅单个主题分区将消耗该主题的所有分区
* [ FLINK-27041 ] - 如果任何主题分区为空，则批处理模式下的 KafkaSource 失败
* [ FLINK-27140 ] - 将 JobResultStore 脏条目创建移动到 ioExecutor
* [ FLINK-27174 ] - KafkaSink 中 bootstrapServers 字段的非空检查不正确
* [ FLINK-27218 ] - 当新的序列化器不兼容时，OperatorState 中的序列化器没有更新
* [ FLINK-27223 ] - 当缓存大小设置为 0 时，状态访问无法按预期工作
* [ FLINK-27247 ] - ScalarOperatorGens.numericCasting 与传统行为不兼容
* [ FLINK-27255 ] - Flink-avro 不支持超过 65535 个字符的 avro 模式的序列化和反序列化
* [ FLINK-27282 ] - 修复 RowCoder 位置映射错误的 bug
* [ FLINK-27367 ] - INT 和 DATE 之间的 SQL CAST 被破坏
* [ FLINK-27368 ] - SQL CAST('1' as BIGINT) 返回错误结果
* [ FLINK-27409 ] - 当作业的资源需求为空时清理陈旧的插槽分配记录
* [ FLINK-27418 ] - Flink SQL TopN 结果错误
* [ FLINK-27420 ] - 暂停的 SlotManager 重新启动时无法重新注册指标
* [ FLINK-27465 ] - AvroRowDeserializationSchema.convertToTimestamp 以负纳秒失败
* [ FLINK-27487 ] - KafkaMetricWrappers 的演员阵容不正确
* [ FLINK-27545 ] - 更新 PyFlink shell 中的示例
* [ FLINK-27563 ] - 资源提供者 - Yarn doc 页面有轻微的显示错误
* [ FLINK-27606 ] - 使用带有 merge() 方法的 UDAF 时出现 CompileException
* [ FLINK-27676 ] - on\_timer 的输出记录落后于 PyFlink 中的触发水印
* [ FLINK-27683 ] - 插入（column1，column2）值（.....）失败并出现 SQL 提示
* [ FLINK-27711 ] - 更正 set\_topics\_pattern 的拼写错误，将其更改为 Pulsar Connector 的 set\_topic\_pattern
* [ FLINK-27733 ] - 修复水印错误修复后的 on\_timer 输出
* [ FLINK-27734 ] - 禁用检查点时 WebUI 中未正确显示检查点间隔
* [ FLINK-27760 ] - 在批处理模式下执行 PyFlink 作业时抛出 NPE
* [ FLINK-27762 ] - 处理拆分更改期间的 Kafka WakeupException
* [ FLINK-27797 ] - PythonTableUtils.getCollectionInputFormat 无法正确处理 None 值
* [ FLINK-27848 ] - ZooKeeperLeaderElectionDriver 不断写入领导者信息，用完 zxid
* [ FLINK-27881 ] - PulsarMessageBuilder 中的键（字符串）返回 null
* [ FLINK-27890 ] - SideOutputExample.java 失败
* [ FLINK-27910 ] - 如果从头开始，FileSink 不会执行滚动策略
* [ FLINK-27933 ] - 无法从备用作业管理器查询保存点状态
* [ FLINK-27955 ] - Windows 操作系统上的 PyFlink 安装失败
* [ FLINK-27999 ] - 使用 Hive 3 方言时出现 NoSuchMethodError
* [ FLINK-28018 ] - 在 BinaryInputFormat#createInputSplits 中创建空拆分的起始索引不合适
* [ FLINK-28019 ] - 在启用状态 ttl 的情况下收回陈旧记录时，RetractableTopNFunction 出错
* [ FLINK-28114 ] - Python 客户端解释器的路径无法指向分布式文件系统中的存档文件

## 改进

* [ FLINK-24586 ] - SQL 函数应该返回 STRING 而不是 VARCHAR(2000)
* [ FLINK-26788 ] - AbstractDeserializationSchema 应该在抛出 FlinkRuntimeException 时添加原因
* [ FLINK-26909 ] - 允许从 CLI 将并行度设置为 -1
* [ FLINK-27064 ] - 集中用于生产代码的 ArchUnit 规则
* [ FLINK-27480 ] - 共享 groupId 的 KafkaSources 可能导致 InstanceAlreadyExistException 警告
* [ FLINK-27534 ] - 将 scalafmt 应用到 1.15 分支
* [ FLINK-27776 ] - 滑动窗口中使用的 UDAF 未在 PyFlink 中实现合并方法时抛出异常
* [ FLINK-27935 ] - 添加创建临时视图文档的 Pyflink 示例

## 技术债务

* [ FLINK-25694 ] - 升级 Presto 以解决 GSON/Alluxio 漏洞

## 子任务

* [ FLINK-26052 ] - 更新关于 FLIP-203 的中文文档
* [ FLINK-26588 ] - 将新的 SQL CAST 文档翻译成中文
* [ FLINK-27382 ] - 使作业模式等待集群关闭，直到清理完成
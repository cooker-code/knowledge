---
title: Flink CDC 黄金搭档：Debezium 1.9.0.CR1 发布！
author: Flink
date:
url: http://mp.weixin.qq.com/s?__biz=MzI0NTIxNzE1Ng==&mid=2651224753&idx=2&sn=b615b91885e89222a9f0fe5f0cf631a0&chksm=f2a3385ac5d4b14c8669e6dc0cebbf5cdf7154713db7d45aa727a184fcc0d860790a5c2e23c2&mpshare=1&scene=24&srcid=0401NopzOYi91pXIcxrCpXLD&sharer_sharetime=1648769840553&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC版本演进与Pipeline连接器边界|FlinkCDC版本演进与Pipeline连接器边界]]


我很高兴地宣布 Debezium **1.9.0.CR1**的发布！

除了一系列错误修复之外，此版本还为 Apache Cassandra 4 带来了期待已久的支持！总体而言，此版本已修复52 个问题。

让我们仔细看看 Cassandra 3 的变化和 Cassandra 4 的支持。

## Cassandra 3 更改和 Cassandra 4 支持

### Cassandra 3 重大变化

对于需要使用 Cassandra 3 的用户，（孵化）连接器的 Maven 坐标在此版本中略有更改。Cassandra 3 的主要变化是工件名称发生了变化：

```
<dependency>
  <groupId>io.debezium</groupId>
  <artifactId>debezium-connector-cassandra-3</artifactId>
  <version>1.9.0.CR1</version>
</dependency>
```

此版本还引入了一项额外的面向用户的更改，即 Cassandra 驱动程序的转变。连接配置不再直接在连接器属性文件中提供，而是必须使用单独的`application.conf`文件提供。您可以在此处找到有关驱动程序配置的完整参考，以下是示例：

```
datastax-java-driver {
  basic {
    request.timeout = 20 seconds
    contact-points = [ "spark-master-1:9042" ]
    load-balancing-policy {
      local-datacenter = "dc1"
    }
  }
  advanced {
    auth-provider {
      class = PlainTextAuthProvider
      username = user
      password = pass
    }
    ssl-engine-factory {
     ...
    }
  }
}
```

为了让 Debezium 连接器读取/使用这个新的应用程序配置文件，必须在连接器属性文件中进行如下设置：

```
cassandra.driver.config.file=/path/to/application/configuration.conf
```

### Cassandra 4 支持

对于新用户和希望升级到 Cassandra 4 的用户，新连接器工件的 Maven 坐标为：

```
<dependency>
  <groupId>io.debezium</groupId>
  <artifactId>debezium-connectr-cassandra-4</artifactId>
  <version>1.9.0.CR1</version>
</dependency>
```

我们引入了一个新的工件而不是用户可配置的切换，因为这允许两个代码库根据需要分开。这允许根据需要改进 Cassandra 3 和 4 连接器，因为我们继续以 Java 11 作为基线来构建 Cassandra 4 连接器。

Debezium for Cassandra 4 连接器基于 Apache Cassandra 4.0.2。如果您打算升级到 Cassandra 4，那么从 Debezium 的角度来看，迁移应该是相对无缝的。升级 Cassandra 环境后，按照上述 Cassandra 3 重大更改部分中所述调整驱动程序配置，然后重新启动 connector.hanges 部分并启动连接器。

我们要感谢Štefan Miklošovič和Ahmed Eljami的贡献！

## 其他修复和更改

1.9.0.CR1 版本中的进一步修复和改进包括：

* 针对 MySQL（DBZ-4786、DBZ-4833、DBZ-4841）和 Oracle（DBZ-4810、DBZ-4851）的各种 DDL 解析器修复
* Oracle 连接器优雅地处理不受支持的列类型（DBZ-4852、DBZ-4853、DBZ-4880）
* 改进 Oracle 连接器的补充日志检查 ( DBZ-4842 , DBZ-4869 )
* 各种 MySQL 连接器改进 ( DBZ-4758 , DBZ-4787 )

请参阅发行说明以了解有关这些以及此版本中进一步修复的更多信息。

与往常一样，非常感谢为此版本做出贡献的每个人：

鲍勃·罗尔丹克里斯·克兰福德克莱门特·洛瓦莱特邹伊桑贡纳尔·莫林 哈维·岳Jakub Cechacek Jiri Novotny Jiri Pechanec何塞·路易斯·桑切斯乔什·里贝拉Katerina Galieva Nathan Smit Oren Elias Robert Roldan Sergei Morozov Stefan米克洛索维奇、沃伊杰赫·尤拉内克和杨

## 总结

完成 CR1 后，您可以在本周晚些时候或下周初看到 1.9 Final，具体取决于问题报告。

当我们开始展望未来时，您可以期待 Debezium 2.0 的工作将在不久的将来开始。当前的路线图是将接下来的两个发布周期用于 Debezium 2.0，并在 2022 年 9 月末附近的某个时间发布。同时，预计在整个过程中 Debezium 1.9 将继续定期更新。
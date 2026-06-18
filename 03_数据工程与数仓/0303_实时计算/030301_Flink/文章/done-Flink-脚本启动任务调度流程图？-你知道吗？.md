---
title: Flink-脚本启动任务调度流程图？-你知道吗？
author: 阿龙大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247484247&idx=1&sn=6346d404e73ea1095be401eed119ad69&chksm=9fb93150a8ceb846c1329e37c7980c836dc0c0b3acf23110f263f4bb3dffa61c3b6c70f29f68&mpshare=1&scene=24&srcid=1115poENfmYBuVGiKvoCMrPU&sharer_shareinfo=29faf17a330d30ee281bca20d3befe8d&sharer_shareinfo_first=29faf17a330d30ee281bca20d3befe8d#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/FlinkSQL脚本化提交与Application模式适配|FlinkSQL脚本化提交与Application模式适配]]


**1、介绍**

在我们写Flink程序的时候，等写完需要上线的时候，需要通过脚本

```
bin/flink run ....
```

来启动任务，那启动的时候，咋启动JobMaster，TaskManager的的了？

****2、流程介绍****

在 Flink 中，可以使用脚本来启动任务调度。以下是一个使用脚本启动任务调度的示例流程：

1、创建 Flink 任务脚本：

首先，你需要创建一个 Flink 任务脚本，其中包含你想要执行的任务逻辑。任务脚本可以使用 Scala、Java 或者 Python 进行编写。在脚本中，你可以定义数据源、操作符、窗口操作等等。

2、设置任务属性：

根据你的需求，你可以在脚本中设置一些任务属性，如并行度、检查点配置、容错等。这些属性可以控制任务的执行方式和性能。

3、创建 Flink 集群：

在使用脚本进行任务调度之前，你需要先创建一个 Flink 集群。可以使用命令行或者基于编程接口的方式来创建集群实例。

4、启动任务调度：

在 Flink 集群创建完成后，你可以使用命令行或者编程接口来启动任务调度。命令行调度可以使用以下命令：复制代码

```
$ ./bin/flink run -c com.example.MyJob MyJob.jar
```

其中，com.example.MyJob 是你的任务脚本的入口类，MyJob.jar 是你的任务脚本打包成的 JAR 文件。这个命令会将你的脚本提交给 Flink 集群，并启动任务调度。

除了命令行方式，你也可以使用 Flink 的编程接口（如 StreamExecutionEnvironment）来以编程方式启动任务调度。编程接口提供了更多的灵活性，你可以根据需要添加更多的配置和操作。

5、监控任务执行：

一旦任务调度启动后，你可以使用 Flink 的监控工具来监控任务的执行状态和性能指标。Flink 提供了 Web 界面（Flink Dashboard）和命令行工具来查看任务的状态、指标和日志。

**3、启动流程图**

以上原理图通过源码抽取而来，需要结合源码来理解记忆。

你学会了吗？
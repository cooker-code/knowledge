> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030205_Spark/030205_核心知识点/SparkConnect生产接入边界|SparkConnect生产接入边界]]
---
title: 基于Spark Connect的生产实践
author: 漫谈大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzkzODIzNTQwNA==&mid=2247485092&idx=1&sn=7932143bd9625054497656b208229868&chksm=c3eed630d12b2b8a48a0f027c70232421e8c33eae2a48bb9810ee92a0873e4d21cbc5a4f2a1a&mpshare=1&scene=24&srcid=12246f1pmJYrZRYHvPebOYBi&sharer_shareinfo=5c3acca2e95c353f40754543a1246b33&sharer_shareinfo_first=5c3acca2e95c353f40754543a1246b33#rd
---

Spark Connect[1] 是 Spark 生态系统[2]中一个相对较新的组件，它允许瘦客户端在远程 Spark 集群上运行 Spark 应用程序。这项技术可以为使用 DataFrame API 的 Spark 应用程序提供一些好处。Spark 长期以来一直允许在远程 Thrift JDBC 服务器上运行 SQL 查询。但是这种远程运行以任何受支持的语言（Scala、Python）编写的客户端应用程序的功能仅在 Spark 3.4 中出现。

在本文中，我将分享我们使用 Spark Connect（版本 3.5）的经验。我将讨论我们获得的好处、与运行 Spark 客户端应用程序相关的技术细节，以及有关如何使 Spark Connect 设置更高效、更稳定的一些提示。

# 使用动机

Spark 是 Joom 分析平台的关键组件之一。我们拥有大量的内部用户和 1000 多个自定义 Spark 应用程序。这些应用程序在一天中的不同时间运行，具有不同的复杂性，并且需要的计算资源量也大不相同（从几分钟的几个内核到几天的 250 多个内核）。以前，它们总是作为单独的 Spark 应用程序（具有自己的驱动程序和执行程序）执行，这对于中小型应用程序（我们历史上有很多这样的应用程序）来说，会导致明显的开销。随着 Spark Connect 的推出，现在可以设置共享的 Spark Connect 服务器并在其上运行许多 Spark 客户端应用程序。从技术上讲，Spark Connect 服务器是具有嵌入式 Spark Connect 终端节点的 Spark 应用程序。

以下是我们能够从中获得的好处：

* • 节省资源
* • 通过 Spark Connect 运行时，客户端应用程序不需要自己的 Spark 驱动程序（通常使用超过 1.5 GB 的内存）。相反，他们使用典型内存消耗为 200 MB 的瘦客户端。
* • 执行程序利用率提高，因为任何执行程序都可以运行多个客户端应用程序的任务。例如，假设某个 Spark 应用程序在执行过程中的某个时刻开始使用的内核和内存比最初请求的要少得多。发生这种情况的原因有很多。然后对于单独的 Spark 应用程序，当前未使用的资源通常会被浪费，因为动态分配通常无法提供有效的缩减。但是使用 Spark Connect 服务器，释放的内核和内存可以立即用于运行其他客户端应用程序的任务。
* • 减少启动等待时间
* • 由于各种原因，我们必须限制同时运行的独立 Spark 应用程序的数量，如果当前所有 slot 都被占用，它们可能会在队列中等待相当长的时间。它可能会对数据准备时间和用户体验产生负面影响。对于 Spark Connect 服务器，到目前为止，我们已经能够避免此类限制，并且所有 Spark Connect 客户端应用程序在启动后立即开始运行。
* • 对于临时执行，最好尽可能减少获得结果的时间，并避免让人们等待。对于单独的 Spark 应用程序，启动客户端应用程序通常需要为其驱动程序和执行程序预置额外的 EC2 节点，以及初始化驱动程序和执行程序。所有这些加起来可能需要 4 分钟以上。对于 Spark Connect 服务器，至少其驱动程序始终处于启动状态并准备好接受请求，因此只需等待其他执行程序即可，而且执行程序通常已经可用。这可能会显著减少临时应用程序准备就绪的等待时间。

### 约束条件

目前我们不在 Spark Connect 上运行长时间运行的繁重应用程序，原因如下：

* • 它们可能会导致 Spark Connect 服务器出现故障或行为不稳定（例如，执行程序节点上的磁盘溢出）。它可能导致整个平台出现大规模问题。
* • 它们通常需要独特的内存设置并使用特定的优化技术（例如，自定义 extraStrategies）。
* • 目前我们在为 Spark Connect 服务器提供大量执行程序来处理非常大的并发负载时遇到了问题（这与 Spark Task Scheduler 的行为有关，超出了本文的范围）。

因此繁重的应用程序仍作为单独的 Spark 应用程序运行。

# 启动客户端应用程序

我们在 Kubernetes/EKS[3] 和 Airflow[4] 上使用 Spark。一些代码示例将特定于此环境。

我们有太多不同的、不断变化的 Spark 应用程序，手动确定每个应用程序是否应该根据我们的标准在 Spark Connect 上运行需要花费太多时间。此外在 Spark Connect 上运行的应用程序列表需要定期更新。例如，假设今天某个应用程序足够轻量级，因此我们决定在 Spark Connect 上运行它。但明天，它的开发人员可能会添加几个大型连接，使其相当繁重。然后最好将其作为单独的 Spark 应用程序运行。反之亦然。

最终我们创建了一个服务来自动确定如何启动每个特定的客户端应用程序。此服务分析每个应用程序以前运行的历史记录，评估 `Total Task Time`、`Shuffle Write`、`Disk Spill` 等指标（此数据是使用 SparkListener[5] 收集的）。开发人员为应用程序设置的自定义参数（例如驱动程序和执行程序的内存设置）也会被考虑在内。根据此数据，该服务会自动为每个应用程序确定这次是应在 Spark Connect 服务器上运行，还是作为单独的 Spark 应用程序运行。因此我们所有的应用程序都应该准备好以这两种方式中的任何一种运行。

在我们的环境中，每个客户端应用程序都是独立于其他应用程序构建的，并且有自己的 JAR 文件，其中包含应用程序代码以及特定的依赖项（例如，ML 应用程序通常使用第三方库，如 CatBoost 等）。问题在于，Spark Connect 的 SparkSession API 与用于单独 Spark 应用程序的 SparkSession API 略有不同（Spark Connect 客户端使用 `spark-connect-client-jvm` 构件）。因此我们应该在每个客户端应用程序的构建时知道它是否将通过 Spark Connect 运行。但我们不知道这一点。下面介绍了我们启动客户端应用程序的方法，这样就无需为同一应用程序构建和管理两个版本的 JAR 构件。

对于每个 Spark 客户端应用程序，我们只构建一个包含应用程序代码和特定依赖项的 JAR 文件。此 JAR 在 Spark Connect 上运行时以及作为单独的 Spark 应用程序运行时均可用。因此，这些客户端 JAR 不包含特定的 Spark 依赖项。稍后将在 Java 类路径中提供相应的 Spark 依赖项（`spark-core`/`spark-sql` 或 `spark-connect-client-jvm`），具体取决于运行模式。在任何情况下，所有客户端应用程序都使用相同的 Scala 代码来初始化 SparkSession，该代码根据运行模式运行。所有客户端应用程序 JAR 都是为常规 Spark API 构建的。因此，在用于 Spark Connect 客户端的代码部分中，通过反射调用特定于 Spark Connect API 的 `SparkSession` 方法（`remote`、`addArtifact`）：

```
val sparkConnectUri: Option[String] = Option(System.getenv("SPARK_CONNECT_URI"))

val isSparkConnectMode: Boolean = sparkConnectUri.isDefined

def createSparkSession(): SparkSession = {
  if (isSparkConnectMode) {
    createRemoteSparkSession()
  } else {
    SparkSession.builder
    // Whatever you need to do to configure SparkSession for a separate 
    // Spark application.
    .getOrCreate
  }
}

private def createRemoteSparkSession(): SparkSession = {
  val uri = sparkConnectUri.getOrElse(throw new Exception(
    "Required environment variable 'SPARK_CONNECT_URI' is not set."))

  val builder = SparkSession.builder
  // Reflection is used here because the regular SparkSession API does not 
  // contain these methods. They are only available in the SparkSession API 
  // version for Spark Connect.
  classOf[SparkSession.Builder]
  .getDeclaredMethod("remote", classOf[String])
  .invoke(builder, uri)

  // A set of identifiers for this application (to be used later).
  val scAppId = s"spark-connect-${UUID.randomUUID()}"
  val airflowTaskId = Option(System.getenv("AIRFLOW_TASK_ID"))
  .getOrElse("unknown_airflow_task_id")
  val session = builder
  .config("spark.joom.scAppId", scAppId)
  .config("spark.joom.airflowTaskId", airflowTaskId)
  .getOrCreate()

  // If the client application uses your Scala code (e.g., custom UDFs), 
  // then you must add the jar artifact containing this code so that it 
  // can be used on the Spark Connect server side.
  val addArtifact = Option(System.getenv("ADD_ARTIFACT_TO_SC_SESSION"))
  .forall(_.toBoolean)

  if (addArtifact) {
    val mainApplicationFilePath = 
    System.getenv("SPARK_CONNECT_MAIN_APPLICATION_FILE_PATH")
    classOf[SparkSession]
    .getDeclaredMethod("addArtifact", classOf[String])
    .invoke(session, mainApplicationFilePath)
  }

  Runtime.getRuntime.addShutdownHook(new Thread() {
    override def run(): Unit = {
      session.close()
    }
  })

  session
}
```

在 Spark Connect 模式下，此客户端代码可以在任何位置作为常规 Java 应用程序运行。由于我们使用 Kubernetes，因此它在 Docker 容器中运行。特定于 Spark Connect 的所有依赖项都打包到用于运行客户端应用程序的 Docker 映像中（可在此处[6]找到此映像的最小示例）。该映像不仅包含 `spark-connect-client-jvm` 构件，还包含几乎所有客户端应用程序使用的其他常见依赖项（例如，`hadoop-aws`，因为我们几乎总是在客户端与 S3 存储进行交互）。

```
FROM openjdk:11-jre-slim

WORKDIR /app

# Here, we copy the common artifacts required for any of our Spark Connect 
# clients (primarily spark-connect-client-jvm, as well as spark-hive, 
# hadoop-aws, scala-library, etc.).
COPY build/libs/* /app/

COPY src/main/docker/entrypoint.sh /app/
RUN chmod +x ./entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
```

当通过 Spark Connect 运行我们的所有客户端应用程序时，此通用 Docker 映像用于运行它们。同时，它不包含包含特定应用程序及其依赖项代码的客户端 JAR，因为有许多此类应用程序会不断更新，并且可能依赖于任何第三方库。相反，当启动特定客户端应用程序时，将使用环境变量传递其 JAR 文件的位置，并在初始化期间以 `entrypoint.sh` 格式下载该 JAR：

```
#!/bin/bash
set -eo pipefail

# This variable will also be used in the SparkSession builder within 
# the application code.
export SPARK_CONNECT_MAIN_APPLICATION_FILE_PATH="/tmp/$(uuidgen).jar"

# Download the JAR with the code and specific dependencies of the client 
# application to be run. All such JAR files are stored in S3, and when 
# creating a client Pod, the path to the required JAR is passed to it 
# via environment variables.
java -cp "/app/*" com.joom.analytics.sc.client.S3Downloader \ 
    ${MAIN_APPLICATION_FILE_S3_PATH} ${SPARK_CONNECT_MAIN_APPLICATION_FILE_PATH}

# Launch the client application. Any MAIN_CLASS initializes a SparkSession 
# at the beginning of its execution using the code provided above.
java -cp ${SPARK_CONNECT_MAIN_APPLICATION_FILE_PATH}:"/app/*" ${MAIN_CLASS} "$@"
```

最后，当需要启动应用程序时，我们的自定义 SparkAirflowOperator 会根据此应用程序之前运行的统计信息自动确定执行模式（Spark Connect 或单独）。

* • 在 Spark Connect 的情况下，我们使用 KubernetesPodOperator[7] 来启动应用程序的客户端 Pod。`KubernetesPodOperator` 将前面描述的 Docker 镜像以及环境变量（`MAIN_CLASS`、`JAR_PATH` 等）作为参数，这些变量将在 `entrypoint.sh` 和应用程序代码中使用。无需为客户端 Pod 分配许多资源（例如，在我们的环境中，其典型消耗：内存 — 200 MB，vCPU — 0.15）。
* • 对于单独的 Spark 应用程序，我们使用自定义 AirflowOperator，它使用 spark-on-k8s-operator[8] 和官方 Spark Docker 映像[9]运行 Spark 应用程序。现在，让我们跳过有关 Spark AirflowOperator 的详细信息，因为这是一个值得单独撰写文章的大主题。

# 与常规 Spark 应用程序的兼容性问题

并非所有现有的 Spark 应用程序都可以在 Spark Connect 上成功执行，因为其 SparkSession API 与用于单独 Spark 应用程序的 SparkSession API 不同。例如，如果代码使用 `sparkSession.sparkContext` 或 `sparkSession.sessionState`，它将在 Spark Connect 客户端中失败，因为 Spark Connect 版本的 SparkSession 没有这些属性。
在我们的例子中，最常见的问题是使用 `sparkSession.sessionState.catalog` 和 `sparkSession.sparkContext.hadoopConfiguration` 。在某些情况下， `sparkSession.sessionState.catalog` 可以替换为 `sparkSession.catalog`，但并非总是如此。 `sparkSession.sparkContext.hadoopConfiguration` 如果在客户端执行的代码包含对数据存储的操作，则可能需要，例如：

```
def delete(path: Path, recursive: Boolean = true)
          (implicit hadoopConfig: Configuration): Boolean = {
  val fs = path.getFileSystem(hadoopConfig)
  fs.delete(path, recursive)
}
```

幸运的是，可以创建一个独立的 `SessionCatalog` 以在 Spark Connect 客户端中使用。在这种情况下，Spark Connect 客户端的类路径还必须包括 `org.apache.spark:spark-hive_2.12` ，以及用于与存储交互的库（因为我们使用 S3，因此在本例中为 `org.apache.hadoop：hadoop-aws`）。

```
import org.apache.spark.SparkConf
import org.apache.hadoop.conf.Configuration
import org.apache.spark.sql.hive.StandaloneHiveExternalCatalog
import org.apache.spark.sql.catalyst.catalog.{ExternalCatalogWithListener, SessionCatalog}

// This is just an example of what the required properties might look like. 
// All of them should already be set for existing Spark applications in one 
// way or another, and their complete list can be found in the UI of any
// running separate Spark application on the Environment tab.
val sessionCatalogConfig = Map(
  "spark.hadoop.hive.metastore.uris" -> "thrift://metastore.spark:9083",
  "spark.sql.catalogImplementation" -> "hive",
  "spark.sql.catalog.spark_catalog" -> "org.apache.spark.sql.delta.catalog.DeltaCatalog",
)

val hadoopConfig = Map(
  "hive.metastore.uris" -> "thrift://metastore.spark:9083",
  "fs.s3.impl" -> "org.apache.hadoop.fs.s3a.S3AFileSystem",
  "fs.s3a.aws.credentials.provider" -> "com.amazonaws.auth.DefaultAWSCredentialsProviderChain",
  "fs.s3a.endpoint" -> "s3.amazonaws.com",
  // and others...
)

def createStandaloneSessionCatalog(): (SessionCatalog,  Configuration) = {
  val sparkConf = new SparkConf().setAll(sessionCatalogConfig)
  val hadoopConfiguration = new Configuration()
  hadoopConfig.foreach { 
    case (key, value) => hadoopConfiguration.set(key, value) 
  }

  val externalCatalog = new StandaloneHiveExternalCatalog(
    sparkConf, hadoopConfiguration)
  val sessionCatalog = new SessionCatalog(
    new ExternalCatalogWithListener(externalCatalog)
  )
  (sessionCatalog, hadoopConfiguration)
}
```

还需要为代码中可访问的 `HiveExternalCatalog` 创建一个包装器（因为 `HiveExternalCatalog` 类是 `org.apache.spark` 包的私有类）：

```
package org.apache.spark.sql.hive

import org.apache.hadoop.conf.Configuration
import org.apache.spark.SparkConf

class StandaloneHiveExternalCatalog(conf: SparkConf, hadoopConf: Configuration) 
  extends HiveExternalCatalog(conf, hadoopConf)
```

此外，通常可以将在 Spark Connect 上不起作用的代码替换为替代代码，例如：

* • `sparkSession.createDataFrame(sparkSession.sparkContext.parallelize(data), schema)` ==> `sparkSession.createDataFrame(data.toList.asJava, schema)` `sparkSession.createDataFrame(sparkSession.sparkContext.parallelize(data), schema)` ==> `sparkSession.createDataFrame(data.toList.asJava, schema)`
* • `sparkSession.sparkContext.getConf.get(“some_property”)` ==> `sparkSession.conf.get(“some_property”)` `sparkSession.sparkContext.getConf.get(“some_property”)` ==> `sparkSession.conf.get(“some_property”)`

### 回退到单独的 Spark 应用程序

遗憾的是，修复特定的 Spark 应用程序以使其作为 Spark Connect 客户端工作并不总是那么容易。例如，项目中使用的第三方 Spark 组件会带来重大风险，因为它们通常是在编写时不考虑与 Spark Connect 的兼容性。由于在我们的环境中，任何 Spark 应用程序都可以在 Spark Connect 上自动启动，因此我们发现在发生故障时实施回退到单独的 Spark 应用程序是合理的。简化后，逻辑如下：

* • 如果某个应用程序在 Spark Connect 上失败，我们会立即尝试将其作为单独的 Spark 应用程序重新运行。同时，我们会增加在 Spark Connect 上执行期间发生的故障的计数器（每个客户端应用程序都有自己的计数器）。
* • 下次启动此应用程序时，我们检查此应用程序的 failure 计数器：
* • 如果失败次数少于 3 次，我们假设上次应用程序失败可能不是因为与 Spark Connect 不兼容，而是由于任何其他可能的临时原因。因此，我们尝试再次在 Spark Connect 上运行它。如果这次成功完成，则此客户端应用程序的 failure 计数器将重置为零。
* • 如果已经有 3 次失败，我们假设应用程序无法在 Spark Connect 上运行，并暂时停止尝试在 Spark Connect 上运行它。此外，它仅作为单独的 Spark 应用程序启动。
* • 如果应用程序在 Spark Connect 上失败 3 次，但最后一次失败是在 2 个多月前，我们会尝试再次在 Spark Connect 上运行它（以防在此期间发生更改，使其与 Spark Connect 兼容）。如果这次成功，我们将再次将 failure 计数器重置为零。如果再次失败，下一次尝试将在 2 个月后进行。

这种方法比维护从日志中识别失败原因的代码要简单一些，并且在大多数情况下效果很好。尝试在 Spark Connect 上运行不兼容的应用程序通常不会产生任何重大的负面影响，因为在绝大多数情况下，如果应用程序与 Spark Connect 不兼容，它会在启动后立即失败，而不会浪费时间和资源。但是，值得一提的是，我们所有的应用程序都是幂等的。

# 统计数据收集

正如我已经提到的，我们收集每个 Spark 应用程序的 Spark 统计数据（我们的大多数平台优化和警报都依赖于它）。当应用程序作为单独的 Spark 应用程序运行时，这很容易。对于 Spark Connect，每个客户端应用程序的阶段和任务需要与在共享 Spark Connect 服务器中同时运行的所有其他客户端应用程序的阶段和任务分开。

可以通过为客户端 `SparkSession` 设置自定义属性，将任何标识符传递给 Spark Connect 服务器：

```
val session = builder
  .config("spark.joom.scAppId", scAppId)
  .config("spark.joom.airflowTaskId", airflowTaskId)
  .getOrCreate()
```

然后在 Spark Connect 服务器端的 `SparkListener` 可以检索所有传递的信息，并将每个阶段/任务与特定的客户端应用程序相关联。

```
class StatsReportingSparkListener extends SparkListener {

  override def onStageSubmitted(stageSubmitted: SparkListenerStageSubmitted): Unit = {
    val stageId = stageSubmitted.stageInfo.stageId
    val stageAttemptNumber = stageSubmitted.stageInfo.attemptNumber()
    val scAppId = stageSubmitted.properties.getProperty("spark.joom.scAppId")
    // ...
  }
}
```

在这里可以找到[10]我们用于收集统计数据的 `StatsReportingSparkListener` 的代码。可能还对这个用于查找 Spark 应用程序中性能问题的免费工具[11]感兴趣。

# 优化和稳定性改进

Spark Connect 服务器是一个永久运行的 Spark 应用程序，大量客户端可以在其中运行其作业。因此，定制其属性[12]是值得的，这可以使其更可靠并防止资源浪费。以下是在我们的示例中有用的一些设置：

```
// Using dynamicAllocation is important for the Spark Connect server 
// because the workload can be very unevenly distributed over time.
spark.dynamicAllocation.enabled: true  // default: false

// This pair of parameters is responsible for the timely removal of idle 
// executors:
spark.dynamicAllocation.cachedExecutorIdleTimeout: 5m  // default: infinity
spark.dynamicAllocation.shuffleTracking.timeout: 5m  // default: infinity

// To create new executors only when the existing ones cannot handle 
// the received tasks for a significant amount of time. This allows you 
// to save resources when a small number of tasks arrive at some point 
// in time, which do not require many executors for timely processing. 
// With increased schedulerBacklogTimeout, unnecessary executors do not 
// have the opportunity to appear by the time all incoming tasks are 
// completed. The time to complete the tasks increases slightly with this, 
// but in most cases, this increase is not significant.
spark.dynamicAllocation.schedulerBacklogTimeout: 30s  // default: 1s

// If, for some reason, you need to stop the execution of a client 
// application (and free up resources), you can forcibly terminate the client. 
// Currently, even explicitly closing the client SparkSession does not 
// immediately end the execution of its corresponding Jobs on the server. 
// They will continue to run for a duration equal to 'detachedTimeout'. 
// Therefore, it may be reasonable to reduce it.
spark.connect.execute.manager.detachedTimeout: 2m  // default: 5m

// We have encountered a situation when killed tasks may hang for 
// an unpredictable amount of time, leading to bad consequences for their 
// executors. In this case, it is better to remove the executor on which 
// this problem occurred.
spark.task.reaper.enabled: true // default: false
spark.task.reaper.killTimeout: 300s  // default: -1

// The Spark Connect server can run for an extended period of time. During 
// this time, executors may fail, including for reasons beyond our control 
// (e.g., AWS Spot interruptions). This option is needed to prevent 
// the entire server from failing in such cases.
spark.executor.maxNumFailures: 1000

// In our experience, BroadcastJoin can lead to very serious performance 
// issues in some cases. So, we decided to disable broadcasting. 
// Disabling this option usually does not result in a noticeable performance 
// degradation for our typical applications anyway.
spark.sql.autoBroadcastJoinThreshold: -1 // default: 10MB

// For many of our client applications, we have to add an artifact to 
// the client session (method sparkSession.addArtifact()). 
// Using 'useFetchCache=true' results in double space consumption for 
// the application JAR files on executors' disks, as they are also duplicated 
// in a local cache folder. Sometimes, this even causes disk overflow with 
// subsequent problems for the executor.
spark.files.useFetchCache: false   // default: true

// To ensure fair resource allocation when multiple applications are 
// running concurrently.
spark.scheduler.mode: FAIR  // default: FIFO
```

例如，在我们调整`了 idle timeout` 属性后，资源利用率发生了如下变化：

### 预防性重启

在我们的环境中，Spark Connect 服务器（版本 3.5）在连续运行几天后可能会变得不稳定。大多数情况下，我们会遇到无限长时间随机挂起的客户端应用程序作业，但也可能存在其他问题。此外随着时间的推移，整个 Spark Connect 服务器随机发生故障的可能性会急剧增加，这可能会在错误的时间发生。

随着这个组件的发展，它可能会变得更加稳定（或者我们会发现我们在 Spark Connect 设置中做错了什么）。但目前，最简单的解决方案是每天在适当的时间（即没有客户端应用程序运行时）预防性重启 Spark Connect 服务器。可以在此处找到[13]重启代码的示例。

# 结论

在本文中，我介绍了我们使用 Spark Connect 运行大量不同 Spark 应用程序的经验。

总结以上内容：

* • 该组件有助于节省资源，减少 Spark 客户端应用程序执行的等待时间。
* • 最好小心哪些应用程序应该在共享的 Spark Connect 服务器上运行，因为资源密集型应用程序可能会给整个系统带来问题。
* • 可以创建用于启动客户端应用程序的基础设施，以便在启动时自动决定如何运行任何应用程序（作为单独的 Spark 应用程序或作为 Spark Connect 客户端）。
* • 请务必注意，并非所有应用程序都能够在 Spark Connect 上运行，但此类情况的数量可以显著减少。如果可能运行尚未经过 Spark Connect 版本 SparkSession API 兼容性测试的应用程序，则值得实施回退以单独使用 Spark 应用程序。
* • 值得关注的是可以提高资源利用率并增加 Spark Connect 服务器整体稳定性的 Spark 属性。设置 Spark Connect 服务器的定期预防性重启以降低意外故障和意外行为的可能性也可能是合理的。

总的来说，我们在公司使用 Spark Connect 获得了很好的体验。我们将继续以极大的兴趣关注这项技术的发展，并有扩大其使用的计划。

原文链接：https://towardsdatascience.com/adopting-spark-connect-cdd6de69fa98

#### 引用链接

`[1]` Spark Connect: *https://spark.apache.org/docs/latest/spark-connect-overview.html*
`[2]` Spark 生态系统: *https://spark.apache.org/*
`[3]` Kubernetes/EKS: *https://spark.apache.org/docs/latest/running-on-kubernetes.html*
`[4]` Airflow: *https://airflow.apache.org/*
`[5]` SparkListener: *https://github.com/joomcode/spark-platform/blob/main/lib/src/main/scala/com/joom/spark/monitoring/StatsReportingSparkListener.scala*
`[6]` 可在此处: *https://github.com/kotlovs/spark-connect-examples/tree/main/spark-connect-client-image*
`[7]` KubernetesPodOperator: *https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/operators.html*
`[8]` spark-on-k8s-operator: *https://github.com/kubeflow/spark-operator*
`[9]` Spark Docker 映像: *https://spark.apache.org/docs/latest/running-on-kubernetes.html#docker-images*
`[10]` 在这里可以找到: *https://github.com/joomcode/spark-platform/blob/main/lib/src/main/scala/com/joom/spark/monitoring/StatsReportingSparkListener.scala*
`[11]` 的免费工具: *https://cloud.joom.ai/*
`[12]` 属性: *https://spark.apache.org/docs/latest/configuration.html*
`[13]` 可以在此处找到: *https://gist.github.com/kotlovs/437809684d7ebe3b7a93a1af804def8c*
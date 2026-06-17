---
title: 一站式解析Spark 日志 ，助力开发Spark任务调优诊断
author: 涤生大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489018&idx=1&sn=6584b7a3d4d07f271b17b4fa94c9df63&chksm=cf6220a8f815a9be2b7557f3dcb295f56cce1a52933474da2cb6e73c03676165f58d3099e704&mpshare=1&scene=24&srcid=0430eH1r7XMEwvxKQgCHhLAK&sharer_shareinfo=12a713bdf42fb780200ea7df3caccf7c&sharer_shareinfo_first=12a713bdf42fb780200ea7df3caccf7c#rd
---

1.需求背景

# 虽然Apache Spark自带的原生History Server为用户提供了一定程度的任务状态监控与回溯能力，它确实能够有效地追踪和展示每个Job的基本运行状况，如任务启动时间、完成时间、执行状态以及资源使用情况等关键信息。

然而，在面对复杂的监控需求时，原生History Server所提供的视图颗粒度主要聚焦于Job层面，缺乏对大规模任务群组进行深层次、聚合性分析的支持。尤其在需要针对特定App Name下所有应用的运行情况进行深度洞察时，例如分析基于自定义字段的趋势走向、资源利用率的演变或是特定业务指标的变化规律等，现有的原生History Server界面并不直接提供此类高度定制化的监控与可视化功能。这就需要用户寻求额外的解决方案，以实现对Spark应用运行状态的精细化管理和深入洞察。

2.需求实现

**分析原生History Server的运行原理。**

# Apache Spark Job History Server（简称SHS）是Spark生态中用于记录和展示Spark应用程序运行历史的核心组件之一。它的实现原理主要包括以下几个关键点：

1. 事件监听与记录：

* 在Spark应用运行时，通过SparkListener接口定义了一系列事件监听器，例如EventLoggingListener，它会捕获Spark应用运行周期中的各种事件，如任务提交、任务开始、任务结束、阶段完成等。
* 这些事件会被序列化为JSON格式，并持久化存储到HDFS或其他支持的文件系统中，形成所谓的Spark Event Logs。

2. 事件日志读取与解析：

* Spark History Server启动后，会扫描并读取存储在HDFS上的事件日志文件。
* 使用EventLogFileReader类解析这些JSON格式的事件日志，将事件恢复为Spark内部表示的事件类。

3. 构建应用程序运行历史：

* 根据解析后的事件，Spark History Server重建整个Spark应用程序的执行历史，包括Job、Stage、Task等不同层级的统计信息和状态。
* 它维护一个内部的数据结构来存储和索引这些信息，以便后续查询和展示。

4. Web UI展示：

* Spark History Server提供了一个Web UI界面，用户可以通过浏览器访问。
* Web UI基于已解析的历史数据，通过动态渲染页面，展示了应用程序的整体执行时间线、各个阶段和任务的状态、资源使用情况以及可能存在的错误信息等详细信息。

5. API支持：

* 除了Web UI，Spark还提供了REST API，允许用户通过API获取历史应用程序的相关信息，以支持第三方工具对接和进一步的数据分析。

通过上面的分析，实现需求的最低成本，看来就是直接通过提供的API获取已经处理好的数据，下面是从api中获取的某个app的数据样例；

结合需求，api中能获取的数据完全可以满足要求，需要最初的方案就是通过api直接获取所需数据，下面给出一个基于python开发的参考：

```
from datetime import datetime,timedeltaimport pymysqlimport requestsfrom urllib.parse import urlencodeimport jsonimport pytz  
# 基本定义spark_job_history_server_url="historyserver.prod.internal.ds.com"bash_url=f"http://{spark_job_history_server_url}/api/v1/applications"  
# 获取前一小时的applicationsdef get_applications(bash_url):    # 获取当前时间，并向前推一小时utc = pytz.UTCnow = datetime.now(utc)one_hour_ago = now - timedelta(hours=1)    # 格式化时间戳为API期望的格式min_date = one_hour_ago.strftime("%Y-%m-%dT%H:%M:%S.000Z")    # 构建带查询参数的URLparams = {'minDate': min_date}query_string = urlencode(params)url = f"{bash_url}?{query_string}"response = requests.get(url)applications = response.json()return applications  
  
def fetch_and_process_application_data(app_id):url = f"{bash_url}/{app_id}"response = requests.get(url)    # 检查请求是否成功if response.status_code == 200:application_data = response.json()attempts_data = application_data['attempts']  
if "attemptId" not in attempts_data[0]:  # 单个或者没有attempt_id情况的处理            # 提取startTime, endTime等值attempt = attempts_data[0]start_time = attempt['startTime']end_time = attempt['endTime']spark_user = attempt['sparkUser']            # completed = attempt['completed']duration = attempt['duration']attempt_id = 0total_num_completed_tasks_all, \            total_num_failed_tasks_all, \            total_num_killed_tasks_all, \            total_num_completed_stages_all, \            total_num_failed_stages_all = get_application_jobs_status(app_id, attempt_id)  
else:  # 多个attempt_ids情况的处理attempt_ids = []attempt_ids, end_time, start_time, duration, spark_user = handle_attempt(application_data)total_num_completed_tasks = []total_num_failed_tasks = []total_num_killed_tasks = []total_num_completed_stages = []total_num_failed_stages = []for attempt_id in attempt_ids:  # 累加多个attempt_ids 下jobs 的值total_num_completed_task, \                total_num_failed_task, \                total_num_killed_task, \                total_num_completed_stage, \                total_num_failed_stage = get_application_jobs_status(app_id, attempt_id)  
                # 累计统计所有attempt id 下 指标值total_num_completed_tasks.append(total_num_completed_task)total_num_failed_tasks.append(total_num_failed_task)total_num_killed_tasks.append(total_num_killed_task)total_num_completed_stages.append(total_num_completed_stage)total_num_failed_stages.append(total_num_failed_stage)total_num_completed_tasks_all = sum(total_num_completed_tasks)total_num_failed_tasks_all = sum(total_num_failed_tasks)total_num_killed_tasks_all = sum(total_num_killed_tasks)total_num_completed_stages_all = sum(total_num_completed_stages)total_num_failed_stages_all = sum(total_num_failed_stages)return app_id, \               start_time, \               end_time, \               spark_user, \               duration, \               total_num_completed_tasks_all, \               total_num_failed_tasks_all, \               total_num_killed_tasks_all, \               total_num_completed_stages_all, \               total_num_failed_stages_all  
  
def handle_attempt(application_data):  
attempt_ids = []end_times = []start_times = []durations = []spark_users = []  
for attempt in application_data['attempts']:attempt_ids.append(attempt['attemptId'])end_times.append(attempt['endTime'])start_times.append(attempt['startTime'])durations.append(attempt['duration'])spark_users.append(attempt['sparkUser'])  
    # 将时间字符串转换为datetime对象并计算endTime的最大值、startTime的最小值、所有duration的累加和以及任意一个sparkUsermax_end_time = max([datetime.strptime(t, '%Y-%m-%dT%H:%M:%S.%fGMT') for t in end_times])min_start_time = min([datetime.strptime(t, '%Y-%m-%dT%H:%M:%S.%fGMT') for t in start_times])total_duration = sum(durations)any_spark_user = spark_users[0]  
    # 将最大结束时间和最小开始时间转换回原始格式max_end_time_str = max_end_time.strftime('%Y-%m-%dT%H:%M:%S.%fGMT')min_start_time_str = min_start_time.strftime('%Y-%m-%dT%H:%M:%S.%fGMT')  
return attempt_ids, max_end_time_str, min_start_time_str, total_duration, any_spark_user  
def get_application_jobs_status(application_id, attempt_id):    # 发起API请求if attempt_id == 0:url = f"{bash_url}/{application_id}/jobs"else:url = f"{bash_url}/{application_id}/{attempt_id}/jobs"response = requests.get(url)  
    # 检查请求是否成功if response.status_code == 200:        # 解析JSON响应内容data = json.loads(response.content)  
        # 初始化各指标的总和total_num_completed_tasks = 0total_num_failed_tasks = 0total_num_killed_tasks = 0total_num_completed_stages = 0total_num_failed_stages = 0  
        # 遍历所有作业信息并累加指标for job in data:total_num_completed_tasks += job["numCompletedTasks"]total_num_failed_tasks += job["numFailedTasks"]total_num_killed_tasks += job["numKilledTasks"]total_num_completed_stages += job["numCompletedStages"]total_num_failed_stages += job["numFailedStages"]  
        # 输出总和return total_num_completed_tasks, total_num_failed_tasks, total_num_killed_tasks, total_num_completed_stages, total_num_failed_stageselse:print(f"请求失败，状态码：{response.status_code}")if __name__ == "__main__":applications = get_applications(bash_url)for application in applications:application_id = application["id"]print(fetch_and_process_application_data(application_id))
```

**新的问题点**

# 通过上面处理，功能内上面是可以满足需求，可是性能上就有点没法满足了，原定义的是1个小数处理一批数据，奈何集群作业的高峰期，1小时内的作业量就不小，主要是有些单个作业的EventLog数据量就很大，用python程序处理起来效率极低。

**新的处理思路**

# 请求接口里的数据，严重受限于Spark History Server，没法提高处理的速度，那就直接读取解析存储在HDFS上的事件日志文件，结合spark的分布式计算能力来并发处理文件数据。

下面是一段参考的scala代码样例：

```
package com.ds  
import scalaj.http.Httpimport org.json4s._import org.json4s.jackson.JsonMethods._import org.apache.spark.sql.SparkSessionimport scala.util.{Try, Failure, Success}  
import java.io.PrintWriterimport scala.concurrent.{Await, Future}import scala.concurrent.duration.Durationimport scala.concurrent.ExecutionContext.Implicits.globalimport java.time.{LocalDateTime, ZoneId, ZoneOffset, ZonedDateTime}import java.time.format.DateTimeFormatter  
  
case class Application(id: String, attempts: Seq[Attempt])  
case class Attempt(attemptId: String, startTime: String, endTime: String)  
object SparkJobInfoForeach extends App {  
  implicit val formats = DefaultFormats  
// 创建SparkSession实例val spark = SparkSession.builder.appName("AppHistoryParser").getOrCreate()  
// 获取当前时间的前一小时时间区间val now = ZonedDateTime.now(ZoneId.systemDefault())val endTime = now.minusHours(1)val startTime = endTime.minusHours(1)  
// 将时区调整为比当前时间晚8小时val adjustedStartTime = startTime.withZoneSameInstant(ZoneOffset.ofHours(1))val adjustedEndTime = endTime.withZoneSameInstant(ZoneOffset.ofHours(1))  
// 格式化时间为字符串val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'UTC'")val formattedStartTime = adjustedStartTime.format(formatter)val formattedEndTime = adjustedEndTime.format(formatter)  
// 构建API URL  val apiUrl = s"http://historyserver.prod.internal.ds.com/api/v1/applications?minEndDate=$formattedStartTime&maxEndDate=$formattedEndTime&status=completed"  
// 发送HTTP请求并获取响应val responseFuture = Future {    Http(apiUrl).asString  }  
val response = Await.result(responseFuture, Duration.Inf)val json = parse(response.body)  
// 确保JSON响应是一个数组val jsonArray = json.extract[JArray]val resultCache = scala.collection.mutable.ArrayBuffer.empty[(String, Long)]//定义中间缓存val resultCacheall = scala.collection.mutable.ArrayBuffer.empty[String]  
//  // 提取所有应用程序的id和attemptIdval appIdsWithAttemptIds = jsonArray.arr.flatMap { app =>val appId = (app \ "id").extract[String]val attempts = (app \ "attempts").extract[Seq[JObject]]val jobname = (app \ "name").extract[String]  
  
    attempts.flatMap { attempt =>val attemptId = (attempt \ "attemptId").extractOpt[String].getOrElse("")val duration = (attempt \ "duration").extractOpt[Long].getOrElse(0L)val sparkUser = (attempt \ "sparkUser").extractOpt[String].getOrElse("")val startTime = (attempt \ "startTime").extractOpt[String].getOrElse("")val endTime = (attempt \ "endTime").extractOpt[String].getOrElse("")  
// 返回一个元组序列，包含所需的所有信息（即使某些值为空或默认值）      Seq((appId, jobname,attemptId, duration, sparkUser, startTime, endTime))    }  }  
val parallelAppIdsWithAttemptIds = appIdsWithAttemptIds.par  parallelAppIdsWithAttemptIds.foreach { case (appId, jobname,attemptId, dur, sparkUser, startTime, endTime) =>// 根据attemptId生成newVariableval newVariable = if (attemptId.nonEmpty) s"${appId}_$attemptId" else appId  
val hdfsPath = s"hdfs:///user/spark2/logs/${newVariable}"  
try {// 尝试读取HDFS文件val lines = spark.sparkContext.textFile(hdfsPath)  
// 统计 job 相关的值val jobEndCount = lines.filter(line => line.contains("SparkListenerJobEnd")).count()val jobFailedCount = lines.filter(line => line.contains("SparkListenerJobEnd") && line.contains("Job Result\":{\"Result\":\"JobFailed\"")).count()  
// 统计 stage 相关的值val stageEndCount = lines.filter(line => line.contains("SparkListenerStageCompleted")).count()val failureStageCount = lines.filter(line => line.contains("SparkListenerStageCompleted") && line.contains("Failure Reason")).count()  
// 统计 task 的相关值val taskEndCounts = lines.filter(line => line.contains("SparkListenerTaskEnd")).count()val taskFailedCount = lines.filter(line => line.contains("SparkListenerTaskEnd") && line.contains("\"Failed\":true")).count()val taskKilledCount = lines.filter(line => line.contains("SparkListenerTaskEnd") && line.contains("\"Killed\":true")).count()  
// 将结果添加到缓存中val result = s"$appId,$jobname,$attemptId,$dur,$sparkUser,$startTime,$endTime,$jobEndCount, $jobFailedCount, $stageEndCount, $failureStageCount, $taskEndCounts, $taskFailedCount,$taskKilledCount"      resultCacheall += result  
    } catch {      case e: org.apache.hadoop.mapreduce.lib.input.InvalidInputException =>// 文件不存在或其他输入异常        println(s"Invalid input error when reading HDFS file at path $hdfsPath: ${e.getMessage}")      case e: Exception =>// 其他未知异常        println(s"Unexpected error when reading HDFS file at path $hdfsPath: ${e.getMessage}")    }  }  
// 将结果写入文件val writer = Try(new PrintWriter("/tmp/tmp/output.txt"))  writer match {    case Success(w) =>      resultCacheall.foreach(w.println)      w.close()      println("写入文件成功")    case Failure(e) =>      println("写入文件失败: " + e.getMessage)  }// 在程序结束时关闭SparkSession  spark.stop()}
```

说明：上面代码的大体逻辑就是从Spark History Server获取在过去一小时内完成的应用程序信息，然后针对每个应用程序的尝试，读取它们在HDFS上的日志文件，统计job、stage和task的执行情况，并将统计结果输出到一个本地文件中。整个过程充分利用了并行处理以提高效率。

后续在用一个python的程序处理本地的结果文件（这个文件就很小了），然后保留结果到mysql中存储；下面是具体实现的python代码样例：

```
import requestsimport csvimport pymysql.cursorsfrom datetime import datetime, timedelta  
  
# 连接到MySQL数据库connection = pymysql.connect(host='xxx.xxxx.xxx.xxx',                             port=3306,                             user='root',                             password='PAAS_Bigdata@123',                             db='test_spark_job_info',                             charset='utf8mb4',                             cursorclass=pymysql.cursors.DictCursor)  
fields_order = ['appId', 'jobname', 'attemptId', 'dur', 'sparkUser', 'startTime', 'endTime', 'jobEndCount','jobFailedCount', 'stageEndCount', 'failureStageCount', 'taskEndCounts', 'taskFailedCount','taskKilledCount']  
try:with connection.cursor() as cursor:# 定义处理单行数据的函数def process_line(line):            parts = line.strip().split(',')            application_id = parts[0]            all_fields = dict(zip(fields_order, parts))  
# 当attemptId为空时，将其替换为0if not all_fields['attemptId']:                all_fields['attemptId'] = 0  
# 处理startTime和endTime字段，移除'GMT'字符            all_fields['startTime'] = all_fields['startTime'].replace('GMT', '')            all_fields['endTime'] = all_fields['endTime'].replace('GMT', '')  
# 将字符串转换为datetime对象            start_time = datetime.strptime(all_fields['startTime'], '%Y-%m-%dT%H:%M:%S.%f')            new_start_time = start_time + timedelta(hours=8) # 增加8小时            end_Time = datetime.strptime(all_fields['endTime'], '%Y-%m-%dT%H:%M:%S.%f')            new_endTime = end_Time + timedelta(hours=8) # 增加8小时# 将新的datetime对象转换回字符串            all_fields['startTime'] = new_start_time.strftime('%Y-%m-%dT%H:%M:%S.%f')            all_fields['endTime'] = new_endTime.strftime('%Y-%m-%dT%H:%M:%S.%f')  
# 发起HTTP请求            url = f"http://api.neuron.paas.internal.mob.com/lineage/search/scheduler/callback?applicationId={application_id}"            response = requests.get(url)  
# 如果接口返回数据且状态码为200，则尝试替换name字段if response.status_code == 200:                data = response.json()if "data" in data and data["data"]:                    job_data = data["data"]# 将name字段替换为jobName字段值                    all_fields['jobname'] = job_data.get('jobName', all_fields['jobname'])  
# 插入或更新数据库            sql = """                INSERT INTO spark_job_stats (appId, jobname, attemptId, dur, sparkUser, startTime, endTime, jobEndCount, jobFailedCount, stageEndCount, failureStageCount, taskEndCounts,taskFailedCount,taskKilledCount)                VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)                ON DUPLICATE KEY UPDATE                    appId = VALUES(appId),                    jobname = VALUES(jobname),                    attemptId = VALUES(attemptId),                    dur = VALUES(dur),                    sparkUser = VALUES(sparkUser),                    startTime = VALUES(startTime),                    endTime = VALUES(endTime),                    jobEndCount = VALUES(jobEndCount),                    jobFailedCount = VALUES(jobFailedCount),                    stageEndCount = VALUES(stageEndCount),                    failureStageCount = VALUES(failureStageCount),                    taskEndCounts = VALUES(taskEndCounts),                    taskFailedCount = VALUES(taskFailedCount),                    taskKilledCount = VALUES(taskKilledCount)            """  
# 按照SQL语句中字段的顺序整理values            values_tuple = tuple([all_fields[field] for field in fields_order])# print(sql)            cursor.execute(sql, values_tuple)  
# 即使接口请求失败，也依然写入原始数据if response.status_code != 200:                print(f"请求失败，应用ID：{application_id}，状态码：{response.status_code}")  
  
# 读取文本文件with open('output.txt', 'r',encoding='utf-8') as file:            reader = csv.reader(file, delimiter=',')for row in reader:                process_line(','.join(row))  
# 提交事务        connection.commit()  
finally:# 关闭数据库连接    connection.close()
```

这段代码的核心任务是从output.txt文件中读取Spark作业统计信息，对这些信息进行预处理（如时间转换、补充缺失字段值、获取外部API数据等），然后将处理后的数据插入或更新到MySQL数据库中的spark\_job\_stats表中。在整个过程中，通过事务管理保证数据的一致性和完整性。如果在获取附加信息时HTTP请求失败，代码会记录错误但仍然将原始数据写入数据库。

最终用过grafana展示结果数据。当然这只是一小部分，更多精彩关注我，持续更新哈

涤生大数据往期精彩推荐

1.[涤生大数据教学集群的首次运维现场复现](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486492&idx=1&sn=d319454bed8a55b4bd291fe132345153&chksm=cf62394ef815b058f0fa2b4f6afa8b0b14cdb91f0ebcf459839dc95a390157af94e9f00e3682&scene=21#wechat_redirect)

2.[涤生大数据HDFS小文件治理总结](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486662&idx=1&sn=703385672164df39c071d9147b5db37f&chksm=cf623994f815b082dafeb12107021a31da71fec16a5208a408a43b654aaffff8e4c9e033feb8&scene=21#wechat_redirect)

3.[运维实战：DolphinScheduler 生产环境升级](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486231&idx=1&sn=31748fc161770b6167bc10a09b6891f5&chksm=cf623e45f815b75366e7cf39d798b073823ce9a24235ac338a9abe35f2b30dd2f11a43c3dfd6&scene=21#wechat_redirect)

4.[运维实战：Ambari开发手册-DolphinScheduler集成实操](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486348&idx=1&sn=fbbcfc45f6e3543af2e0762ec512689d&chksm=cf623edef815b7c8b98e97559f9ae1ab6d2d8de8c658c320d91dd44c714201a1771027bc13fa&scene=21#wechat_redirect)

5.[大数据运维实战之Ambari修护Hive无法更换tez引擎](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486766&idx=1&sn=e1f4ee464fb97f152de9f4c53a788ec5&chksm=cf62387cf815b16a1d1635b0e630b916c71cf6bb232e15cdc1e4e5069cd36343d917a68f0074&scene=21#wechat_redirect)

6.[大数据平台实践之CDH6.2.1+spark3.3.0+kyuubi-1.6.0](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487193&idx=1&sn=17e1e7c61929c7f5f62b29020b383865&chksm=cf623b8bf815b29d61c66eb2f3ca8c823e021a3ebfadaba06beb7f7e102ba753204120d2dad0&scene=21#wechat_redirect)

7.[运维实战：CDH6.3.2编译集成Flink](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486078&idx=1&sn=7f9ada3c94f8cc2c236cc2e0b890bfe3&chksm=cf623f2cf815b63a9b82e324263fbb10d13a8007e1e6659f60bf6dc5c8aa4e4b47e2f631f83e&scene=21#wechat_redirect)

8.[运维实战100：CDH5.16.2升级至CDH6.3.2](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486047&idx=1&sn=4c0f1d1ff7ab0ed181bdbdb6d2f780db&chksm=cf623f0df815b61bf88382770bbc0da023159797cb1591b1abfc27e7c647e8870c05bb766347&scene=21#wechat_redirect)

9.[CDH集成的kerberos迁移实战（原创干货）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247485914&idx=1&sn=78da500e156d5d6904677c8fe5c73837&chksm=cf623c88f815b59efb16180874cd568494408619706677d97641b5e7807fedae3efba4581cf7&scene=21#wechat_redirect)

10.[CDH启用kerberos 高可用运维实战](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247485950&idx=1&sn=31ca5bdac764bd4821d637b2a5f6e8ea&chksm=cf623cacf815b5ba12c29f549ee90b90fb6d34a3b53d06926cd49ec3af00754350dc82f23325&scene=21#wechat_redirect)

11.[Hbase 迁移小结：从实践中总结出的最佳迁移策略](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487317&idx=1&sn=6ab7ca99397f033484259a79ea5224e3&chksm=cf623a07f815b311e58bc2a6512d61d669dfe765072baed978aca028ad5df41c45d698154798&scene=21#wechat_redirect)

12.[WebHDFS Rest API 企业实战：大佬手把手带你堵住漏洞，企业实例解析](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487738&idx=1&sn=f158d862d828e024850913a80f837d4d&chksm=cf6225a8f815acbe7a6bb23133cb936bd506ee7088638a64666a1080ccdbcc4c46d2e7960d30&scene=21#wechat_redirect)

13.[解析线上HBase集群CPU飙高的原因与解决方案](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487833&idx=1&sn=c5eac46bee0060ee0b5478b2ea0775ad&chksm=cf62240bf815ad1d16b7388a69308092f07ba0096af211aa73263171c7a8d7bbd488e28fcda9&scene=21#wechat_redirect)

14.[大数据实战：Ambari开发手册之OpenTSDB快速集成技巧](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487862&idx=1&sn=ea5706abd23cc4d4b0f3059321e9fd2d&chksm=cf622424f815ad320facdcbc1d0546b4e1fd3629d6aca80257767c9930d15d1e092955624816&scene=21#wechat_redirect)

15.[迁移策略：CDH 集群整体平缓迁移的最佳实践](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487946&idx=1&sn=63c682a6e708b933e27407512432a3e4&chksm=cf622498f815ad8e31efb524716b4f91597dd602ec470a876e159bd8becdd703b110bdbaeef0&scene=21#wechat_redirect)

16.[你知道HDFS 节点内数据（磁盘间）是怎么均衡的吗？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488843&idx=1&sn=98da564f71b512cd651e4ef41b282351&chksm=cf622019f815a90fe6b1e5b1ebc6ecf0d748b1ead13719bab95edf913a09ea64b5726cf17975&scene=21#wechat_redirect)

17.[关于列式存储你可能不知道的事儿](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488878&idx=1&sn=e2200b964bdf2b6051887d5c262b996a&chksm=cf62203cf815a92acd961bd9e0fa42fb359c1222c23e5540ddec935048580bd9b77a77a74d3b&scene=21#wechat_redirect)
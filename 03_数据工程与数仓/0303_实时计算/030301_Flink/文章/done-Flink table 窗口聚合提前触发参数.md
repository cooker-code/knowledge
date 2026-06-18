> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030301_Flink/030301_核心知识点/Flink窗口高级特性|Flink窗口高级特性]]
---
title: Flink table 窗口聚合提前触发参数
author: Flink菜鸟
date:
url: http://mp.weixin.qq.com/s?__biz=MzI3MjAxNDYwOA==&mid=2247484850&idx=1&sn=daf6d96cbcd14d994a970ba27b9f2be9&chksm=eb3849cddc4fc0db2a525933bd2a99233bd6c201b97881112c78752b9c47a0eb12c302c33d64&mpshare=1&scene=24&srcid=0606BIveNsNC3ai7QNW7u9te&sharer_sharetime=1654479248538&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

Flink 版本：1.15.0

先上参数：

```
```
# 启动提前触发# Specifies whether to enable late-fire emit。Late-fire is an emit strategy after watermark advanced to end of window.table.exec.emit.early-fire.enabled = true;# 提前触发时间# The late firing delay in milli second, late fire is the emit strategy after watermark advanced to end of window.# < 0 is illegal configuration.# 0 means no delay (fire on every element).# > 0 means the fire interval.table.exec.emit.early-fire.delay = 5000;
```
```

flink sql api 添加这两个参数，可以提前触发 窗口聚合的任务。

在 Flink 1.13 发布 sql CUMULATE 窗口之前，**我一直觉得** flink 的 sql 窗口不能使用。比如最简单的场景：统计一天的 pv/uv

## 不开启提前触发窗口

如以下 demo，从 kafka 读取数据，做 窗口聚合，输出一天的 pv、uv

```
```
-- kafka source drop table if exists user_log;CREATE TABLE user_log(    user_id     VARCHAR,    item_id     VARCHAR,    category_id VARCHAR,    behavior    VARCHAR,    proc_time  as PROCTIME(),    ts          TIMESTAMP(3),    WATERMARK FOR ts AS ts - INTERVAL '5' SECOND) WITH (      'connector' = 'kafka'      ,'topic' = 'user_log'      ,'properties.bootstrap.servers' = 'localhost:9092'      ,'properties.group.id' = 'user_log'      ,'scan.startup.mode' = 'latest-offset'      ,'format' = 'json'      );drop table if exists user_log_sink_1;CREATE TABLE user_log_sink_1(    wCurrent string    ,wStart     STRING    ,wEnd    STRING    ,pv    bigint    ,uv    bigint    ,primary key(wCurrent,wStart,wEnd) not enforced) WITH (--       'connector' = 'print'      'connector' = 'upsert-kafka'      ,'topic' = 'user_log_sink'      ,'properties.bootstrap.servers' = 'localhost:9092'      ,'properties.group.id' = 'user_log'      ,'key.format' = 'json'      ,'value.format' = 'json'      );-- window aggregationinsert into user_log_sink_1select date_format(now(), 'yyyy-MM-dd HH:mm:ss')     ,date_format(TUMBLE_START(proc_time, INTERVAL '1' minute), 'yyyy-MM-dd HH:mm:ss') AS wStart     ,date_format(TUMBLE_END(proc_time, INTERVAL '1' minute), 'yyyy-MM-dd HH:mm:ss') AS wEnd     ,count(1) coun     ,count(distinct user_id)from user_loggroup by TUMBLE(proc_time, INTERVAL '1' minute)
```
```

* 注：为了方便测试，1 天的窗口改为 1 分钟的窗口

任务流图如下：

任务输出如下：

```
+I[2022-06-01 17:14:00, 2022-06-01 17:13:00, 2022-06-01 17:14:00, 29449, 9999]+I[2022-06-01 17:15:00, 2022-06-01 17:14:00, 2022-06-01 17:15:00, 29787, 9999]+I[2022-06-01 17:16:00, 2022-06-01 17:15:00, 2022-06-01 17:16:00, 29765, 9999]+I[2022-06-01 17:17:00, 2022-06-01 17:16:00, 2022-06-01 17:17:00, 29148, 9999]+I[2022-06-01 17:18:00, 2022-06-01 17:17:00, 2022-06-01 17:18:00, 30079, 9999]
```

可以明显的看到，1分钟的窗口，每分钟输出了一次结果，这就比较尴尬了，因为 1 天的窗口，也只会在窗口结束的时候，触发一次计算

对于实时的任务，每天结束的时候，才输出计算的结果，没有任何意义，我们需要的是实时更新的结果

我们需要的是像 Streaming api 一样的窗口，有触发器可以提前触发计算，输出计算的中间结果，如下：

```
```
.windowAll(TumblingEventTimeWindows.of(Time.days(1))).trigger(ContinuousEventTimeTrigger.of(Time.seconds(5))
```
```

## 提前触发窗口

前两天，在社区群里面看到一个大佬有发着两个参数

```
```
val tabEnv = StreamTableEnvironment.create(env, settings)val tabConf = tabEnv.getConfigtabConf.set("table.exec.emit.early-fire.enabled", "true")tabConf.set("table.exec.emit.early-fire.delay", "5000")
```
```

可以提前触发窗口的结果

任务同上，只是添加以上两个参数

任务输出结果如下：

```
```
2022-06-01 17:54:21,031 INFO  - add parameter to table config: table.exec.emit.early-fire.enabled = true2022-06-01 17:54:21,032 INFO  - add parameter to table config: table.exec.emit.early-fire.delay = 50002022-06-01 17:54:21,032 INFO  - add parameter to table config: pipeline.name = test_table_parameter
+I[2022-06-01 17:54:35, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 2527, 2500]-U[2022-06-01 17:54:40, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 2527, 2500]+U[2022-06-01 17:54:40, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 5027, 5000]-U[2022-06-01 17:54:45, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 5027, 5000]+U[2022-06-01 17:54:45, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 7527, 7500]-U[2022-06-01 17:54:50, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 7527, 7500]+U[2022-06-01 17:54:50, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 10079, 9999]-U[2022-06-01 17:54:55, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 10079, 9999]+U[2022-06-01 17:54:55, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 12579, 9999]-U[2022-06-01 17:55:00, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 12579, 9999]+U[2022-06-01 17:55:00, 2022-06-01 17:54:00, 2022-06-01 17:55:00, 14579, 9999]
+I[2022-06-01 17:55:05, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 2500, 2500]-U[2022-06-01 17:55:10, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 2500, 2500]+U[2022-06-01 17:55:10, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 5299, 5000]。。。忽略部分中间结果-U[2022-06-01 17:55:55, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 25364, 9999]+U[2022-06-01 17:55:55, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 27864, 9999]-U[2022-06-01 17:56:00, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 27864, 9999]+U[2022-06-01 17:56:00, 2022-06-01 17:55:00, 2022-06-01 17:56:00, 29867, 9999]
+I[2022-06-01 17:56:05, 2022-06-01 17:56:00, 2022-06-01 17:57:00, 2500, 2500]-U[2022-06-01 17:56:10, 2022-06-01 17:56:00, 2022-06-01 17:57:00, 2500, 2500]
```
```

可以看到，每 5 秒触发了一次计算，除了第一次只有一条 insert 消息，后续的每次触发都有一条 -U/+U 的消息，累加了窗口的结果

## 总结

Flink sql 的窗口聚合也可以想 Streaming api 设置 trigger 一样，提前触发计算，并且输出的结果是 upsert 流，会发出 -U/+U 两条数据(输出到 upsert-kafka 就只有 +U 的消息了)

注：该参数不支持 window TVF，如以下 sql

```
```
insert into user_log_sink_1select date_format(now(), 'yyyy-MM-dd HH:mm:ss')     ,date_format(window_start, 'yyyy-MM-dd HH:mm:ss') AS wStart     ,date_format(window_end, 'yyyy-MM-dd HH:mm:ss') AS wEnd     ,count(user_id) pv     ,count(distinct user_id) uvFROM TABLE(             TUMBLE(TABLE user_log, DESCRIPTOR(ts), INTERVAL '1' MINUTES )) t1group by window_start, window_end
```
```

报错：

```
```
Exception in thread "main" org.apache.flink.table.api.TableException: Currently, window table function based aggregate doesn't support early-fire and late-fire configuration 'table.exec.emit.early-fire.enabled' and 'table.exec.emit.late-fire.enabled'.  at org.apache.flink.table.planner.plan.utils.WindowUtil$.checkEmitConfiguration(WindowUtil.scala:262)  at org.apache.flink.table.planner.plan.nodes.physical.stream.StreamPhysicalLocalWindowAggregate.translateToExecNode(StreamPhysicalLocalWindowAggregate.scala:127)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:74)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:71)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:71)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:71)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:71)  at org.apache.flink.table.planner.plan.nodes.exec.ExecNodeGraphGenerator.generate(ExecNodeGraphGenerator.java:54)  at org.apache.flink.table.planner.delegation.PlannerBase.translateToExecNodeGraph(PlannerBase.scala:336)  at org.apache.flink.table.planner.delegation.PlannerBase.translate(PlannerBase.scala:180)  at org.apache.flink.table.api.internal.TableEnvironmentImpl.translate(TableEnvironmentImpl.java:1656)  at org.apache.flink.table.api.internal.TableEnvironmentImpl.executeInternal(TableEnvironmentImpl.java:782)  at org.apache.flink.table.api.internal.StatementSetImpl.execute(StatementSetImpl.java:108)  at com.rookie.submit.main.SqlSubmit$.main(SqlSubmit.scala:100)  at com.rookie.submit.main.SqlSubmit.main(SqlSubmit.scala)
```
```
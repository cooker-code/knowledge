---
title: 大数据运维：Flink CDC任务 “server_id 冲突”的故障处理（生产环境）
author: 运维仙人
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMTYxMDY3Mw==&mid=2247488226&idx=1&sn=70274eda7a99386b2765e8d7110938cc&chksm=9a4c5dc08e3bd9fe2d0b5d3851750bd2cc8cf0823f175822d56432be0d42e72b846a6004c183&mpshare=1&scene=24&srcid=1118bD2lSggqQo75veb6le2G&sharer_shareinfo=4cec5bd1f16521c42aed2cb06d523515&sharer_shareinfo_first=4cec5bd1f16521c42aed2cb06d523515#rd
---

如果文章对您有帮助，请帮忙点赞，分享，关注，您的鼓励，是我无限动力~

---

背景：最近数开同事配置了flink cdc任务将数据实时从MySQL同步到StarRocks，而数开同事为了快速配置，copy代码之后，修改了库表配置，但未修改server\_id，启动任务之后监控告警业务延迟时间一直增长。于是介入排查问题，发现了server\_id冲突了造成任务异常。因此整理该文章，希望大家能够避开这个坑。

Flink CDC 报此错误的核心原因：**作为 MySQL 逻辑从库的 Flink CDC 任务，其使用的**`server_id`**或**`server_uuid`**与其他从库（物理从库或其他 CDC 任务）重复**，导致 MySQL 主库无法区分连接，拒绝同步 binlog。

### 一、问题本质再明确

很多人不知道：**Flink CDC本质是MySQL的“逻辑从库”**——它通过Debezium模拟MySQL从库协议，向主库请求binlog同步。而MySQL主库识别从库，全靠两个“身份证”：

* `server_id`

  整数标识，主库下所有从库（物理从库+逻辑从库如Flink CDC）必须唯一；
* `server_uuid`

  字符串标识，由Debezium自动生成（基于`server.name`），默认不重复，但克隆任务会导致冲突。

**如果存在以下情况，就会触发该错误**：

* 同一 MySQL 主库下，有两个 Flink CDC 任务配置了相同的 `server_id`。
* 其他物理从库（MySQL 从库实例）的 `server_id` 或 `server_uuid` 与当前 Flink CDC 任务重复。
* Flink CDC 任务重启时，未正确重新生成 `server_uuid`（极少数情况，与 Debezium 版本有关）。

### 二、生产级排查与解决步骤（更细化实操）

#### 1. 第一步：全面排查“冲突源”（找到重复的标识）

需要先确认到底是哪个从库与当前 CDC 任务冲突，避免“盲目改配置”。

##### （1）排查 MySQL 主库上的所有从库连接

登录 MySQL 主库，执行以下命令查看所有从库连接信息：  

```
-- 查看所有与主库建立的 binlog 同步连接（Command 为 Binlog Dump 的即为从库）SHOW PROCESSLIST;
```

##### （2）获取冲突从库的 `server_id` 和 `server_uuid`

* 对于**物理从库**（如示例中的 192.168.1.101）：  
  登录该从库，执行：

```
SHOW VARIABLES LIKE 'server_id';  -- 假设返回 101SHOW VARIABLES LIKE 'server_uuid';  -- 假设返回 uuid-101
```

* 对于**其他 Flink CDC 任务**（如示例中的 192.168.1.102）：  
  查看该 CDC 任务的配置，找到 `debezium.database.server.id`（假设配置为 200）和 `debezium.database.server.name`（假设为 cdc-task-01，其对应的 `server_uuid` 由 Debezium 基于此生成）。

##### （3）定位当前 CDC 任务的配置

查看当前报错的 Flink CDC 任务配置，确认其 `server_id` 和 `server_name`：  

```
-- 示例：当前任务的错误配置（与其他从库重复）CREATE TABLE error_cdc_source (  ...) WITH (  ...  'debezium.database.server.id' = '200',  -- 与其他 CDC 任务重复  'debezium.database.server.name' = 'cdc-task-01'  -- 重复);
```

#### 2. 第二步：为当前 CDC 任务分配“唯一标识”

核心是确保 `server_id` 全局唯一，`server_name` 唯一（避免 `server_uuid` 重复）。

##### （1）修改 `server_id`（必须）

* 规则：`server_id` 为 1~4294967295 的整数，需与主库、所有物理从库、其他 CDC 任务的 `server_id` 完全不同。
* 建议：按“业务模块+序号”规划，例如：

+ 物理从库：100~199
+ 用户模块 CDC：200~299
+ 订单模块 CDC：300~399

**修改示例**：  
若当前任务是订单模块，且 301 未被使用，则配置：  

```
'debezium.database.server.id' = '301'  -- 唯一且未被占用
```

##### （2）修改 `server_name`（避免 `server_uuid` 重复）

`server_uuid` 由 Debezium 自动生成，生成规则依赖 `server_name`（默认格式：`server_name + 随机串`）。若 `server_name` 重复，可能导致 `server_uuid` 重复。  

**修改示例**：  
按“数据库-表名-任务序号”命名，确保唯一：  

```
'debezium.database.server.name' = 'order-db-orders-cdc-02'  -- 与其他任务不重复
```

##### （3）清除旧状态（关键！否则配置不生效）

Flink CDC 会将历史 `server_id` 和 `offset` 保存在 Checkpoint 状态中。若不清除旧状态，任务重启后会优先读取旧配置，导致冲突依旧。  

* 方法1：删除 Flink 状态后端中该任务的状态数据（如 HDFS 路径 `hdfs:///flink/checkpoints/{job-id}/`）；
* 方法2：在 Flink SQL 中重新创建表（更换表名），或在代码中重新生成 JobID（避免复用旧状态）。

#### 3. 第三步：验证冲突是否解决

任务重启后，通过以下方式确认：  

##### （1）查看 Flink 日志

日志中无“`same server_uuid/server_id`”报错，CDC 源算子状态为“RUNNING”。

##### （2）查看 MySQL 主库连接

再次执行 `SHOW PROCESSLIST;`，能看到当前 CDC 任务的新连接，`Host` 为 Flink 节点 IP，`Command` 为 `Binlog Dump`，且无重复的 `server_id` 对应记录。

##### （3）检查数据同步

下游表（如 Kafka、StarRocks）能正常接收到数据，无断流或重复同步。

### 三、预防措施：建立 `server_id` 管理规范

为避免未来再次出现冲突，建议建立《`server_id` 分配台账》，记录所有从库（物理+逻辑）的标识信息，例如：  

每次新增 CDC 任务前，先查询台账确认可用的 `server_id`，从源头避免重复。

### 总结

此错误的解决核心是“**唯一性**”：通过排查找到重复的 `server_id` 或 `server_uuid` 来源，为当前 CDC 任务分配唯一标识，并清除旧状态确保配置生效。结合规范的 `server_id` 管理，可彻底避免此类问题。

---

在看点这里

---

欢迎在评论区晒出你的实践经验，让我们在交流中碰撞出更多思路~
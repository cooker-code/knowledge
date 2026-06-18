> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030201_Hadoop&HDFS/030201_核心知识点/YARN资源调度与运行边界|YARN资源调度与运行边界]]
---
title: Flink + YARN + Gitlab 自动提交代码全流程详解
author: 大数据技能圈
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491656&idx=1&sn=e8b141938d56fac356a5bda182ca556f&chksm=c10cbeb4b0a3c73e9a104346d0394aacfd889e48b7fda5280d9ffb976b3e3cb75cc2e6ba8037&mpshare=1&scene=24&srcid=0805Dbm3LTnevwOIEcM0GORW&sharer_shareinfo=4a1431ac57327c4b702c9a6fef745537&sharer_shareinfo_first=4a1431ac57327c4b702c9a6fef745537#rd
---

01

**加入知识星球《随川陪你学大数据》将获得以下权益**

即将涨价，最后一天，全年最低价，即刻领20元新人券加入星球

01

**Flink + YARN + Gitlab 自动提交代码全流程详解**

 

# Flink  + YARN + Gitlab 自动提交代码全流程详解

## 引言

在大数据实时计算领域，Apache Flink 凭借其流批一体、低延迟、高吞吐的特性，成为企业级实时计算的主流选择。FlinkSQL 作为 Flink 的关系型 API，降低了实时开发的门槛，让开发者通过 SQL 即可完成复杂流处理逻辑。而 YARN 作为 Hadoop 生态的资源调度平台，为 Flink 作业提供了稳定的资源管理与隔离能力。Gitlab 则作为代码托管与 CI/CD 平台，实现了代码版本控制与自动化流程的串联。

本文将详细介绍 **Flink + FlinkSQL + YARN + Gitlab** 的自动提交代码全流程，涵盖环境准备、代码管理、作业开发、CI/CD 流程设计、自动提交与监控等核心环节，帮助企业构建实时计算的自动化开发与部署体系。

## 1. 环境准备

### 1.1 组件版本选择

为确保各组件兼容性，推荐以下版本组合（基于企业级稳定实践）：

| 组件 | 版本 | 说明 |
| --- | --- | --- |
| Hadoop | 3.3.1 | 包含 YARN 资源调度器 |
| Flink | 1.16.0 | 支持 FlinkSQL 与 YARN 集成 |
| Gitlab | 14.9.0 | 代码托管与 CI/CD |
| Java | 1.8 | Flink 运行基础环境 |
| Maven | 3.8.6 | 项目构建工具 |

### 1.2 环境安装与配置

#### 1.2.1 Hadoop & YARN 集群搭建

假设已部署 Hadoop 集群（需包含 HDFS 和 YARN），核心配置如下：

* • **`core-site.xml`**（HDFS 配置）：

  ```
  <configuration>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://namenode:8020</value>
    </property>
  </configuration>
  ```
* • **`yarn-site.xml`**（YARN 配置）：

  ```
  <configuration>
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>resourcemanager</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
  </configuration>
  ```

启动 YARN 后，可通过 `http://<resourcemanager>:8088` 访问 YARN Web UI，确认集群状态正常。

#### 1.2.2 Flink on YARN 配置

下载 Flink 1.16.0 二进制包并解压，修改 `conf/flink-conf.yaml`：

```
# Flink on YARN 核心配置
jobmanager.rpc.address: localhost
rest.port: 8081
# YARN 队列配置（需与 YARN 队列名称一致）
yarn.application.queue: flink_queue
# 状态后端（推荐使用 RocksDB）
state.backend: rocksdb
# Checkpoint 存储（HDFS 路径）
state.checkpoints.dir: hdfs://namenode:8020/flink/checkpoints
```

将 Hadoop 配置文件（`core-site.xml`、`hdfs-site.xml`、`yarn-site.xml`）软链至 Flink 的 `conf` 目录：

```
ln -s $HADOOP_HOME/etc/hadoop/core-site.xml $FLINK_HOME/conf/
ln -s $HADOOP_HOME/etc/hadoop/hdfs-site.xml $FLINK_HOME/conf/
ln -s $HADOOP_HOME/etc/hadoop/yarn-site.xml $FLINK_HOME/conf/
```

#### 1.2.3 Gitlab 部署与基础配置

* • 部署 Gitlab：可通过 Docker 快速部署（推荐使用 Gitlab 官方镜像）：

  ```
  docker run -d --name gitlab \
    -p 8080:80 -p 2222:22 \
    -v /srv/gitlab/config:/etc/gitlab \
    -v /srv/gitlab/logs:/var/log/gitlab \
    -v /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:14.9.0-ce.0
  ```

  访问 `http://<gitlab-ip>:8080`，初始化管理员密码后创建项目（如 `flink-realtime`）。
* • 配置 SSH 密钥：本地生成 SSH 密钥（`ssh-keygen -t rsa`），将公钥（`~/.ssh/id_rsa.pub`）添加到 Gitlab 用户设置中，确保代码可免密推送。

## 2. Gitlab 代码管理规范

### 2.1 项目结构设计

基于 Maven 构建Flink作业，推荐项目结构如下：

```
flink-realtime/
├── src/
│   ├── main/
│   │   ├── java/          # Java/Scala 代码（如 UDF、自定义 Source/Sink）
│   │   ├── resources/     # 配置文件与 SQL 脚本
│   │   │   ├── sql/       # FlinkSQL 脚本（如 user_behavior.sql）
│   │   │   ├── application.properties # 作业配置（并行度、Kafka 地址等）
│   │   │   └── log4j2.xml # 日志配置
│   │   └── scala/         # Scala 代码（可选）
│   └── test/              # 单元测试
├── .gitlab-ci.yml         # Gitlab CI/CD 配置文件
├── pom.xml                # Maven 依赖配置
└── README.md              # 项目说明文档
```

### 2.2 分支管理策略

采用 Git Flow 分支模型，核心分支如下：

| 分支类型 | 名称 | 用途 | 合并目标 |
| --- | --- | --- | --- |
| 主分支 | master | 生产环境代码，仅允许 CI/CD 自动更新 | 无 |
| 开发分支 | develop | 开发环境集成分支 | master |
| 功能分支 | feature/xxx | 新功能开发（如 feature/user\_behavior） | develop |
| 修复分支 | hotfix/xxx | 生产问题修复 | master/develop |

### 2.3 代码提交规范

为便于 CI/CD 流程追踪，提交信息需遵循以下格式：

```
<type>(<scope>): <description>

# 示例
feat(sql): 新增用户行为实时统计SQL
fix(config): 修复Kafka消费组配置错误
docs(ci): 更新Gitlab CI部署文档
```

* • `type`：类型（`feat` 新功能、`fix` 修复、`docs` 文档、`style` 格式、`refactor` 重构等）
* • `scope`：影响范围（如 `sql`、`config`、`ci`）
* • `description`：简洁描述（不超过50字符）

## 3. Flink 作业开发（以 FlinkSQL 为核心）

### 3.1 依赖配置（pom.xml）

核心依赖包括 Flink 核心、FlinkSQL、连接器（如 Kafka、MySQL）等：

```
<dependencies>
    <!-- Flink 核心 -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>1.16.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_2.12</artifactId>
        <version>1.16.0</version>
        <scope>provided</scope>
    </dependency>
    <!-- FlinkSQL -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-api-java-bridge_2.12</artifactId>
        <version>1.16.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-planner_2.12</artifactId>
        <version>1.16.0</version>
        <scope>provided</scope>
    </dependency>
    <!-- Kafka 连接器 -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-kafka_2.12</artifactId>
        <version>1.16.0</version>
    </dependency>
    <!-- MySQL 连接器（用于结果写入） -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-jdbc_2.12</artifactId>
        <version>1.16.0</version>
    </dependency>
</dependencies>
```

### 3.2 FlinkSQL 脚本开发

以“用户行为实时统计”为例，开发 FlinkSQL 脚本（`resources/sql/user_behavior.sql`）：

```
-- 1. 创建 Kafka 数据源表（用户行为日志）
CREATE TABLE user_behavior (
    user_id BIGINT,
    item_id BIGINT,
    behavior STRING, -- 行为类型：pv（浏览）、buy（购买）、cart（加购）、fav（收藏）
    ts TIMESTAMP(3)
) WITH (
    'connector'='kafka',
    'topic'='user_behavior',
    'properties.bootstrap.servers'='kafka1:9092,kafka2:9092',
    'properties.group.id'='flink_consumer_group',
    'format'='json',
    'scan.startup.mode'='latest-offset'
);

-- 2. 创建 MySQL 结果表（每小时行为统计）
CREATE TABLE behavior_hourly_stats (
    window_start TIMESTAMP(3),
    window_end TIMESTAMP(3),
    behavior STRING,
    count BIGINT,
    PRIMARY KEY (window_start, behavior) NOT ENFORCED
) WITH (
    'connector'='jdbc',
    'url'='jdbc:mysql://mysql:3306/realtime_stats',
    'table-name'='behavior_hourly_stats',
    'username'='flink',
    'password'='flink123'
);

-- 3. 执行统计逻辑（每小时窗口计数）
INSERT INTO behavior_hourly_stats
SELECT
    TUMBLE_START(ts, INTERVAL'1'HOUR) AS window_start,
    TUMBLE_END(ts, INTERVAL'1'HOUR) AS window_end,
    behavior,
    COUNT(*) AS count
FROM user_behavior
GROUPBY
    TUMBLE(ts, INTERVAL'1'HOUR),
    behavior;
```

### 3.3 作业主程序开发

通过 Java 代码加载 SQL 脚本并执行，实现作业提交（`src/main/java/com/example/FlinkSQLJob.java`）：

```
package com.example;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;
import org.apache.flink.table.api.TableResult;
import java.io.File;

publicclassFlinkSQLJob {
    publicstaticvoidmain(String[] args)throws Exception {
        // 1. 创建流执行环境（使用 YARN 模式）
        finalStreamExecutionEnvironmentenv= StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(4); // 设置全局并行度

        // 2. 创建 TableEnvironment（Blink Planner）
        EnvironmentSettingssettings= EnvironmentSettings.newInstance()
                .useBlinkPlanner()
                .inStreamingMode()
                .build();
        TableEnvironmenttableEnv= TableEnvironment.create(settings);

        // 3. 加载 SQL 脚本文件
        StringsqlPath=newFile("resources/sql/user_behavior.sql").getAbsolutePath();
        StringsqlScript=newString(java.nio.file.Files.readAllBytes(java.nio.file.Paths.get(sqlPath)));

        // 4. 执行 SQL 脚本（按语句分割并逐条执行）
        String[] sqlStatements = sqlScript.split(";(?=(?:[^']*'[^']*')*[^']*$)");
        for (String sql : sqlStatements) {
            if (!sql.trim().isEmpty()) {
                TableResultresult= tableEnv.executeSql(sql.trim());
                System.out.println("SQL executed: " + sql.trim());
            }
        }

        // 5. 提交作业（FlinkSQL 的 INSERT INTO 会自动触发执行）
        env.execute("FlinkSQL User Behavior Hourly Stats");
    }
}
```

### 3.4 作业配置（application.properties）

将动态配置（如 Kafka 地址、并行度）提取到配置文件中，避免硬编码：

```
# 作业名称
job.name=flink_sql_user_behavior
# 并行度
job.parallelism=4
# Kafka 配置
kafka.bootstrap.servers=kafka1:9092,kafka2:9092
kafka.topic=user_behavior
kafka.group.id=flink_consumer_group
# MySQL 配置
mysql.url=jdbc:mysql://mysql:3306/realtime_stats
mysql.username=flink
mysql.password=flink123
# Checkpoint 配置
checkpoint.interval=60000 # 1分钟一次 Checkpoint
checkpoint.timeout=300000 # Checkpoint 超时时间5分钟
```

## 4. YARN 资源调度与作业提交

### 4.1 Flink on YARN 模式选择

Flink on YARN 支持三种模式，需根据场景选择：

| 模式 | 特点 | 适用场景 |
| --- | --- | --- |
| Session Mode | 预启动 YARN Application，共享 JobManager | 短作业、低资源消耗场景 |
| Per-Job Mode | 每个作业独立启动 YARN Application | 作业间资源隔离要求高 |
| Application Mode | 推荐：作业主程序在 YARN 中执行，客户端仅提交 | 生产环境主流模式 |

本文以 **Application Mode** 为例，该模式下作业主逻辑在 YARN 的 Application Master 中运行，客户端只需提交 JAR 包，避免客户端资源占用。

### 4.2 手动提交作业到 YARN

通过 `flink run -yarn` 命令提交作业，核心参数如下：

```
flink run -t yarn-application \
  -Dyarn.application.name=flink_sql_user_behavior \
  -Dyarn.application.queue=flink_queue \
  -Dparallelism.default=4 \
  -Djobmanager.memory.process.size=1600m \
  -Dtaskmanager.memory.process.size=1728m \
  -Dtaskmanager.numberOfTaskSlots=4 \
  -c com.example.FlinkSQLJob \
  /path/to/flink-realtime-1.0-SNAPSHOT.jar
```

参数说明：

* • `-t yarn-application`：指定 Application Mode
* • `-Dyarn.application.name`：YARN 应用名称
* • `-Dyarn.application.queue`：YARN 队列（需与 YARN 配置一致）
* • `-Dparallelism.default`：默认并行度
* • `-c`：主程序全限定类名

提交后，可通过 YARN Web UI（`http://<resourcemanager>:8088`）查看作业状态，点击“Tracking UI”进入 Flink Web UI 监控作业指标。

## 5. Gitlab CI/CD 自动提交流程设计

### 5.1 Gitlab CI/CD 原理

Gitlab CI/CD 通过 `.gitlab-ci.yml` 定义流水线（Pipeline），流水线包含多个阶段（Stage），每个阶段包含多个作业（Job）。当代码提交/合并到指定分支时，Gitlab Runner 自动执行流水线，完成构建、测试、部署等流程。

### 5.2 Gitlab Runner 配置

#### 5.2.1 安装 Gitlab Runner

在可访问 YARN 集群的节点上安装 Gitlab Runner（以 Linux 为例）：

```
# 添加 Gitlab Runner 官方仓库
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash

# 安装 Runner
sudo yum install -y gitlab-runner

# 注册 Runner（需从 Gitlab 项目获取注册 URL 和 Token）
sudo gitlab-runner register
```

注册时需配置：

* • `Gitlab instance URL`：Gitlab 地址（如 `http://<gitlab-ip>:8080`）
* • `Registration token`：从 Gitlab 项目 `Settings -> CI/CD -> Runners` 获取
* • `Executor type`：选择 `shell`（需确保 Runner 节点已安装 Java、Maven、Flink、Hadoop 客户端）

#### 5.2.2 Runner 权限配置

确保 Runner 用户（默认 `gitlab-runner`）有权限访问 HDFS 和 YARN：

```
# 将 gitlab-runner 用户加入 hadoop 用户组
sudo usermod -aG hadoop gitlab-runner

# 配置 HDFS 代理用户（在 core-site.xml 中添加）
<property>
  <name>hadoop.proxyuser.gitlab-runner.groups</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.gitlab-runner.hosts</name>
  <value>*</value>
</property>
```

重启 HDFS 和 YARN 使代理用户配置生效。

### 5.3 .gitlab-ci.yml 流水线配置

在项目根目录创建 `.gitlab-ci.yml`，定义以下阶段：

| 阶段 | 作用 | 作业示例 |
| --- | --- | --- |
| build | Maven 打包生成 JAR | build\_job |
| test | 单元测试、SQL 语法校验 | test\_job |
| deploy | 提交作业到 YARN（生产/开发） | deploy\_prod\_job |

完整配置示例：

```
# 定义流水线阶段
stages:
-build
-test
-deploy

# 全局变量（避免硬编码）
variables:
MAVEN_OPTS:"-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
FLINK_HOME:"/opt/flink-1.16.0"
HADOOP_HOME:"/opt/hadoop-3.3.1"
YARN_QUEUE:"flink_queue"

# 缓存 Maven 依赖（加速构建）
cache:
paths:
    -.m2/repository

# 1. 构建阶段：Maven 打包
build_job:
stage:build
script:
    -echo"Building Flink job..."
    -mvncleanpackage-DskipTests
artifacts:
    paths:
      -target/*.jar# 保存 JAR 包供后续阶段使用
    expire_in:1hour# 1小时后过期

# 2. 测试阶段：单元测试 + SQL 语法校验
test_job:
stage:test
script:
    -echo"Running unit tests..."
    -mvntest
    -echo"Validating FlinkSQL syntax..."
    # 使用 Flink SQL Parser 校验语法（需提前编写校验脚本）
    -bashscripts/validate_sql.shresources/sql/user_behavior.sql
dependencies:
    -build_job# 依赖构建阶段的 JAR 包

# 3. 部署阶段：提交到 YARN（仅 master 分支触发）
deploy_prod_job:
stage:deploy
script:
    -echo"Deploying to YARN production queue..."
    # 从构建产物中获取 JAR 包名称
    -JAR_FILE=$(findtarget-name"*.jar"|head-n1)
    -echo"JAR file: $JAR_FILE"
    # 提交作业到 YARN（Application Mode）
    -$FLINK_HOME/bin/flinkrun-tyarn-application\
        -Dyarn.application.name=flink_sql_user_behavior\
        -Dyarn.application.queue=$YARN_QUEUE\
        -Dparallelism.default=4\
        -Djobmanager.memory.process.size=1600m\
        -Dtaskmanager.memory.process.size=1728m\
        -Dtaskmanager.numberOfTaskSlots=4\
        -ccom.example.FlinkSQLJob\
        $JAR_FILE
dependencies:
    -build_job
only:
    -master# 仅 master 分支提交时触发
when:manual # 手动触发（可选，避免误部署）
```

### 5.4 SQL 语法校验脚本

为避免 SQL 语法错误导致作业提交失败，可编写校验脚本（`scripts/validate_sql.sh`）：

```
#!/bin/bash

SQL_FILE=$1
if [ ! -f "$SQL_FILE" ]; then
    echo"Error: SQL file $SQL_FILE not found."
    exit 1
fi

# 使用 Flink 内置的 SQL Parser 校验语法（需 Flink 环境变量）
$FLINK_HOME/bin/sql-client.sh -f "$SQL_FILE" -d
if [ $? -eq 0 ]; then
    echo"FlinkSQL syntax validation passed."
else
    echo"Error: FlinkSQL syntax validation failed."
    exit 1
fi
```

赋予脚本执行权限：`chmod +x scripts/validate_sql.sh`。

## 6. 自动提交全流程实践

### 6.1 开发与提交代码

1. 1. **创建功能分支**：从 `develop` 分支切出功能分支：

   ```
   git checkout -b feature/user_behavior_stat develop
   ```
2. 2. **开发代码**：编写 FlinkSQL 脚本、Java 主程序及配置文件，本地测试通过后提交：

   ```
   git add .
   git commit -m "feat(sql): 新增用户行为实时统计SQL"
   git push origin feature/user_behavior_stat
   ```
3. 3. **提交 Merge Request**：在 Gitlab 上创建从 `feature/user_behavior_stat` 到 `develop` 的 Merge Request（MR），触发流水线自动执行 `build` 和 `test` 阶段。

### 6.2 流水线执行过程

* • **构建阶段**：Maven 自动编译打包，生成 `flink-realtime-1.0-SNAPSHOT.jar`，并保存为流水线产物。
* • **测试阶段**：执行单元测试（如 UDF 测试）和 SQL 语法校验，若测试失败，流水线终止并通知开发者。
* • **合并到 develop**：MR 审核通过后，合并到 `develop` 分支，此时不触发部署。
* • **发布到生产**：将 `develop` 分支合并到 `master` 分支，触发 `deploy_prod_job`，自动提交作业到 YARN 生产队列。

### 6.3 作业状态监控

* • **YARN 监控**：通过 YARN Web UI 查看作业状态（运行中、成功、失败），点击“Logs”查看 YARN 日志。
* • **Flink 监控**：点击作业的“Tracking UI”进入 Flink Web UI，监控 Checkpoint、反压、吞吐量等指标。
* • **日志收集**：可将 Flink 作业日志输出到 HDFS 或 ELK 集群，便于问题排查。

## 7. 常见问题与优化

### 7.1 常见问题排查

#### 问题1：作业提交到 YARN 失败，报“YARN application not found”

**原因**：Flink 与 Hadoop 版本不兼容，或 YARN 配置文件未正确软链。
**解决**：确保 Flink 版本支持 Hadoop 3.x，检查 `$FLINK_HOME/conf` 下是否有 `yarn-site.xml` 等配置文件。

#### 问题2：SQL 语法校验通过，但作业运行时报“Table not found”

**原因**：SQL 脚本中表名大小写与实际不一致，或连接器配置错误（如 Kafka topic 不存在）。
**解决**：检查 SQL 表名大小写（FlinkSQL 默认不区分大小写，但存储系统可能区分），确认 Kafka/MySQL 连接参数。

#### 问题3：Gitlab Runner 执行部署时报“Permission denied”

**原因**：Runner 用户无权限访问 HDFS 或提交 YARN 作业。
**解决**：检查 Hadoop 代理用户配置，确保 `gitlab-runner` 用户属于 `hadoop` 用户组。

### 7.2 流程优化建议

* • **多环境部署**：通过 Gitlab 变量区分开发/测试/生产环境（如 `$YARN_QUEUE_DEV`、`$YARN_QUEUE_PROD`），实现一套代码多环境部署。
* • **版本回滚**：在 `.gitlab-ci.yml` 中添加回滚作业，通过 `yarn application -kill <app_id>` 停止旧作业，再提交指定版本的 JAR 包。
* • **通知机制**：集成钉钉/飞书机器人，流水线成功/失败时发送消息通知开发团队。
* • **资源动态调整**：根据作业负载，通过 Flink REST API 动态调整并行度或资源，提升资源利用率。

## 8. 总结

本文详细介绍了 Flink + FlinkSQL + YARN + Gitlab 的自动提交代码全流程，从环境准备、代码管理、作业开发到 CI/CD 流程设计，覆盖了实时计算自动化部署的核心环节。通过该流程，企业可实现：

* • **开发效率提升**：代码提交后自动构建、测试、部署，减少人工操作。
* • **质量管控**：通过单元测试、SQL 校验等环节，降低作业上线风险。
* • **资源隔离**：YARN 提供多队列资源管理，实现作业间资源隔离与公平调度。

未来可进一步扩展与监控系统集成（如 Prometheus + Grafana）、实现作业自动扩缩容，构建更完善的实时计算运维体系。

 

##

本详细文档已放入星球，即将涨价，最后一天，全年最低价，即刻领20元新人券加入星球

获取更多信息，关注大数据技能圈

## 推荐阅读系列文章

* [超强整理：Iceberg最新学习文档（十四章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491583&idx=1&sn=50016bf38b99c5a1f33a441b841c1b96&scene=21#wechat_redirect)
* [超强总结：Spark最新学习文档（十二章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491603&idx=1&sn=4770b27307ff0c6ff2b1d1763e1e8482&scene=21#wechat_redirect)
* [超强总结：Flink最新学习文档（十二章）](https://mp.weixin.qq.com/s?__biz=Mzg5Mzg3MzkwNA==&mid=2247491637&idx=1&sn=589ae0f30eb9dd02ab959c3cb18e5b0f&scene=21#wechat_redirect)
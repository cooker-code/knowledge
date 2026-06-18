# FlinkJDBCConnector内部实现

> 验证版本：Flink 1.15.4，flink-connector-jdbc，Native Kubernetes Application 部署

## 来源
- [Flink 源码 - Connector JDBC - 探索 JdbcInputFormat 在 多个 Graph 之间的转换](../文章/done-Flink 源码 - Connector JDBC - 探索 JdbcInputFormat 在 多个 Graph 之间的转换.md)

## 核心问题
JdbcInputFormat 是如何从用户代码一路传递到 TaskManager 执行的？它在 StreamGraph、JobGraph、ExecutionGraph 三层中分别如何存储？并发读取 MySQL 是如何切分任务的？

## 判断准则

### JdbcInputFormat 在三层 Graph 中的流转

| Graph 层 | 存储位置 | 存储方式 |
|---|---|---|
| StreamGraph | `StreamNode.inputFormat` 字段 | 对象引用（Java 对象） |
| JobGraph | `JobVertex.configuration` 字段，key = `"udf"` | `InstantiationUtil.writeObjectToConfig()` 序列化为二进制字节数组 |
| ExecutionGraph | `ExecutionJobVertex`（包含 JobVertex）→ 各 `ExecutionVertex` | 继承 JobVertex 配置，按并行度拆分为多个独立执行实例 |

### 关键源码路径
1. **StreamGraph 阶段**：`StreamExecutionEnvironment#getStreamGraph()` 构建时，`JdbcInputFormat` 赋值给 `StreamNode.inputFormat`
2. **JobGraph 阶段**：
   - `StreamingJobGraphGenerator#createChain()` 检测 `currentNode.getInputFormat() != null`，放入 `chainedInputOutputFormats` 容器
   - `StreamingJobGraphGenerator#createJobVertex()` 将 inputFormat 通过 `InstantiationUtil.writeObjectToConfig()` 序列化写入 JobVertex 的 configuration
   - 序列化存储结构：`InputOutputFormatContainer#write()` → `config.setStubWrapper(new UserCodeObjectWrapper<>(formats))`
3. **ExecutionGraph 阶段**：JobVertex → `ExecutionJobVertex`（定义处理逻辑），按并行度展开为多个 `ExecutionVertex`（独立执行单元）

### JdbcNumericBetweenParametersProvider 并发切分机制
```java
JdbcNumericBetweenParametersProvider provider = new JdbcNumericBetweenParametersProvider(
    fetchSize,   // 每个子任务拉取条数
    minId,       // 最小 ID
    maxId        // 最大 ID
);
// 根据 fetchSize/minId/maxId 计算出 SQL 参数集合
// "select * from table where id between ? and ?"
// 每个并行 SubTask 获取不同的 between 参数对
```
计算时机：在 `JobManager` 执行 `main()` 方法时（`PackagedProgram#callMainMethod()`），参数集合计算完毕后随 JdbcInputFormat 一同序列化进 Graph。

### ExecutionJobVertex vs ExecutionVertex 的分工
- **ExecutionJobVertex**：定义数据处理逻辑（1 个），对应 JobGraph 中的 1 个 JobVertex
- **ExecutionVertex**：具体并行执行实例（N 个 = 并行度），每个独立执行，互不干扰
- 两者关系：ExecutionJobVertex 包含多个 ExecutionVertex，ExecutionVertex 持有 ExecutionJobVertex 的引用

### 调试 Graph 中 InputFormat 内容的方法
在 IDEA Debug 模式下，对 binary configuration 执行求值表达式：
```java
InstantiationUtil.readObjectFromConfig(
    jobVertex.getConfiguration(),
    "udf",
    getClass().getClassLoader()
)
```

## 认知偏差
| 常见错误认知 | 正确理解 |
|---|---|
| JobGraph 是 StreamGraph 的简单复制 | JobGraph 会将 StreamNodes 按链式合并规则合并为 JobVertex，同时序列化 InputFormat 为二进制 |
| ExecutionGraph 改变了 DAG 结构 | ExecutionGraph 不改变 DAG 拓扑，只是引入并行度维度，每个 JobVertex 展开为多个 ExecutionVertex |
| 并发读取 MySQL 时任务切分在 TaskManager 端进行 | 任务切分（参数集计算）在 JobManager 的 main() 阶段已完成，结果随 Graph 传递到 TaskManager |
| InputFormat 对象在整个链路中保持 Java 对象形式 | 在 JobGraph 阶段就被序列化为二进制存入 configuration，跨网络传输的是字节数组 |

## 架构/流程图

```mermaid
flowchart TD
    A[MySQL2Console.main()] -->|JdbcNumericBetweenParametersProvider 计算 SQL 参数集| B[StreamGraph]
    B -->|StreamNode.inputFormat = JdbcInputFormat 对象| C[JobGraph 生成]
    C -->|InstantiationUtil.writeObjectToConfig 序列化| D[JobVertex.configuration 'udf' key]
    D -->|JobManager 分发给 TaskManager| E[ExecutionGraph]
    E -->|ExecutionJobVertex 包含 JobVertex| F[ExecutionVertex ×N]
    F -->|N个并行 SubTask 各自读取不同 ID 范围| G[MySQL 分区数据]
```
（基于原文描述重建）

## 待验证缺口
- TaskManager 如何从 JobManager 接收任务集（文章明确标注为后续篇章，本文未覆盖）
- `InputOutputFormatContainer` 在多个 InputFormat 共存时如何管理和区分
- 当 JdbcInputFormat 的并行度与 `JdbcNumericBetweenParametersProvider` 计算出的分片数不一致时，任务如何分配

## 重新蒸馏补充（2026-06-18）

| 来源 | 认知增量 | 处理 |
|---|---|---|
| [[03_数据工程与数仓/0303_实时计算/030301_Flink/文章/done-flink sql parallelism mysql source]] | 补充该主题的生产案例、机制边界或排重样例。 | 重新判断后补入目标知识产物 |

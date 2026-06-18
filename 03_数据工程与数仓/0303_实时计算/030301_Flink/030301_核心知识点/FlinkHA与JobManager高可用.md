# HA 与 JobManager 高可用

> 验证版本：Flink 1.12+（K8s HA），Flink 1.0+（ZooKeeper HA）

## 来源
- [Flink JobManager 宕机了怎么办？一文讲透 Flink HA 原理与配置](../文章/done-Flink%20JobManager%20宕机了怎么办？一文讲透%20Flink%20HA%20原理与配置.md)

## 核心问题
JobManager 是默认单点故障（SPOF）：宕机后整个作业失败且无法自动恢复。TaskManager 宕机不是 HA 要解决的问题（JM 会重新申请容器），HA 专门解决 JM 宕机的容灾。

## 判断准则

**HA 架构核心**：Active/Standby 多 JM 实例 + 分布式选主（ZK/K8s）+ 元数据持久化（HDFS/S3）

**选主机制对比**：

| 维度 | ZooKeeper HA | Kubernetes HA |
|---|---|---|
| 引入版本 | Flink 1.0+ | Flink 1.12+ |
| 外部依赖 | ZK 集群（≥3 节点）| K8s API Server（已有）|
| 选举机制 | 临时顺序节点 + Watcher | ConfigMap + 租约 |
| 元数据存储 | ZK(指针) + HDFS/S3(数据) | ConfigMap(指针) + HDFS/S3(数据) |
| 运维复杂度 | 高 | 低（复用 K8s 基础设施）|
| 适用部署模式 | Standalone / YARN / K8s | 仅 Kubernetes |
| 故障检测速度 | 取决于 ZK session timeout | 取决于 K8s 租约时间 |

**两级存储设计原因**：
- ZK / K8s ConfigMap：存储量 KB 级，读写快，但有大小限制（ZK 默认 1MB）
- HDFS / S3：存储 JobGraph、Checkpoint 元数据、Blob 文件，数据量 MB~GB

**故障恢复行为**（重要边界）：新 JM 不会无条件重新申请所有 TM，而是：
1. 等待运行中的 TM 重新注册
2. 优先在已有 TM 的 Slot 上重新调度
3. 仅在 Slot 不足或 TM 不可用时，才向资源管理器申请新 TM

**HA 必须搭配的配置**：
- Checkpoint（HA 依赖 Checkpoint 做状态恢复）
- 大状态场景：RocksDB + 增量 Checkpoint
- Restart Strategy（建议 failure-rate）

**最佳实践清单**：
- HA storageDir 与 Checkpoint dir 使用独立路径，禁用本地文件系统
- 每个集群使用唯一 cluster-id，多集群禁止共享
- ZK session timeout：网络抖动场景调大，高灵敏场景调小
- HA 元数据不会在作业取消后自动清理，需配置定期清理
- 定期演练故障切换，验证恢复可靠性

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| HA 后 TaskManager 宕机也能自动恢复 | 即使不配置 HA，TM 宕机也能自动恢复（JM 重新申请容器），HA 只解决 JM 单点问题 |
| 新 JM 接管后会重启所有 TM | 新 JM 优先复用已有 TM 的 Slot，只在资源不足时才申请新 TM |
| HA storageDir 和 Checkpoint dir 可以是同一目录 | 必须独立，混用会导致元数据混乱 |
| K8s HA 在所有部署模式下都可用 | K8s HA 仅适用于 Kubernetes 部署，YARN/Standalone 需用 ZK HA |
| 配置了 HA 就不需要 Checkpoint | HA 依赖 Checkpoint 存储的状态数据来恢复作业，两者必须搭配 |

## 关键配置示例

```yaml
# ZooKeeper HA
high-availability.type: zookeeper
high-availability.zookeeper.quorum: zk1:2181,zk2:2181,zk3:2181
high-availability.zookeeper.path.root: /flink
high-availability.cluster-id: /my-production-cluster
high-availability.storageDir: hdfs:///flink/ha/

# Kubernetes HA
high-availability.type: kubernetes
high-availability.cluster-id: my-flink-cluster
high-availability.storageDir: s3://my-bucket/flink/ha/
kubernetes.namespace: flink-production
```

## 待验证缺口
- K8s HA 的 ConfigMap 租约时间具体如何配置，与故障检测延迟的关系
- Flink 2.x 是否对 K8s HA 有新的改进或默认行为变更

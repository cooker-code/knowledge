# LakeSoul

## 技术定位

| 项 | 内容 |
|---|---|
| 技术名 | LakeSoul |
| 一级类目 | 数据工程与数仓 |
| 二级类目 | 湖仓表格式 |
| 技术本体 | 面向湖仓一体与 AI 数据底座的表格式/湖仓框架，强调 Upsert、多流合并、Native IO 和流批一体 |
| 全局架构位置 | 位于对象存储/HDFS 之上，连接 Spark、Flink、Python/AI 生态和下游分析或训练任务 |
| 主要使用者 | 数仓工程师、湖仓平台工程师、AI 数据平台工程师 |
| 主要产出 | 湖仓表、增量流、训练样本数据、流批一体数据集 |

## 归类边界

| 相邻对象 | 不归入这里的情况 |
|---|---|
| Iceberg / Hudi / Delta Lake / Paimon | 文章主问题是这些表格式自身机制时，归对应技术目录 |
| Flink | 文章主问题是实时计算状态、窗口或 Checkpoint 时，归实时计算 |
| 机器学习 / MLOps | 文章主问题是模型训练、模型服务和实验管理时，归机器学习 |
| Agent 与 AI 工程 | 文章主问题是 Agent 应用、RAG 或工具调用时，不因提到 AI 数据底座而归 LakeSoul |

## 排重准则

问题指纹：

```text
LakeSoul + 元数据/Native IO/Upsert/MOR/Schema 演进/流批读写模块 + 核心机制 + 解决问题 + 湖仓/AI 数据边界 + 对用户的认知增量
```

## 后续追查

- LakeSoul 与 Iceberg、Hudi、Delta Lake、Paimon 的机制对比。
- Native IO、集中式元数据和多流并发 Upsert 的真实边界。
- 面向 AI 训练样本构造时，湖仓表格式和特征平台的分工。

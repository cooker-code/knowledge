# Spark 4 版本变化与迁移边界

> 验证版本：Apache Spark 4.0.0 release notes

## 来源

- [Spark4.0革命：你未曾注意到的数据处理新范式（一）](<../文章/done-Spark4.0革命：你未曾注意到的数据处理新范式（一）.md>)
- [Spark 在 SQL, Python, Streamig 和 AI 集成等模块的创新应用](<../文章/done-Spark 在 SQL, Python, Streamig 和 AI 集成等模块的创新应用.md>)
- [历时一年 Apache Spark 3.3.0 正式发布，新特性详解](<../文章/done-历时一年 Apache Spark 3.3.0 正式发布，新特性详解.md>)

## 核心问题

Spark 4 文章不能只摘“更快”“新范式”。真正会影响生产迁移的是运行环境、SQL 语义、半结构化数据类型、Hive Catalog 兼容、Spark Connect 和 Python/客户端生态变化。

## 判断准则

| 变化 | 迁移判断 |
|---|---|
| Scala 2.13 / JDK 17 | 先查依赖、UDF、连接器、部署镜像和集群运行时 |
| ANSI 默认 | 检查历史 SQL 中隐式类型转换、溢出、非法值处理 |
| VARIANT | 只在半结构化数据场景评估，不泛化为所有 SQL 提速 |
| Hive Catalog | 同时查 Hive Metastore 版本和 Spark 支持范围 |
| Spark Connect | 作为接入形态评估，不替代任务调度和执行治理 |

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| Spark 4 一定让所有任务变快 | 收益依赖工作负载、算子、数据格式和配置 |
| VARIANT 适合所有 JSON 场景 | 要看字段访问模式、存储格式和读取路径 |
| ANSI 默认只是兼容小问题 | 对历史 SQL 的类型转换和异常处理可能是破坏性变化 |

## 待验证缺口

- 需要补本地 Spark 3 到 4 的 SQL 兼容性检查清单。
- 需要补 VARIANT 与现有 JSON 解析链路的最小基准。

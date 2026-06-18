# 部署环境与 JDK 版本管理

> 验证版本：Flink 2.2.0 / 1.20.2，Hadoop 3.3.6

## 来源
- [Hadoop必须用JDK 8，Flink 2.2/1.20却要JDK 11？一文解决环境冲突](../文章/done-Hadoop必须用JDK%208，Flink%202.2_1.20却要JDK%2011？一文解决环境冲突.md)

## 核心问题
Flink 2.x / 1.20+ 要求 JDK 11，但 Hadoop 集群因历史原因固定在 JDK 8，如何在不改集群环境的前提下让 Flink on YARN 运行在 JDK 11 上？

## 判断准则

**核心方案**：使用 `yarn.ship-archives` 参数将 JDK 包随 Application 分发到每个 Container。

**配置步骤**：
1. 将 JDK 11 压缩包上传到 HDFS（速度快于本地分发）
2. 在 `config.yaml` 中配置归档路径和容器内 JAVA_HOME

```yaml
# 推荐：HDFS 分发
yarn.ship-archives: hdfs:///user/user01/jdk/jdk-11.0.15.1_linux-x64_bin.tar.gz

# 指定容器内 Java 路径（注意解压后有一层目录）
containerized.master.env.JAVA_HOME: '$PWD/jdk-11.0.15.1_linux-x64_bin.tar.gz/jdk-11.0.15.1'
containerized.taskmanager.env.JAVA_HOME: '$PWD/jdk-11.0.15.1_linux-x64_bin.tar.gz/jdk-11.0.15.1'
```

**关键避坑点**：
1. **路径层级**：`ship-archives` 自动解压，解压后的根目录名是压缩包名，JDK 实际在内层目录
2. **不要用 `yarn.ship-files`**：ship-files 同步文件但不解压，启动时会报动态链接库（`.so`）错误
3. **命令行参数无效**：这些配置写在命令行中不生效，必须写在 `config.yaml`

**适用部署模式**：yarn-session 和 yarn-application 均适用。

## 认知偏差

| 常见错误认知 | 正确理解 |
|---|---|
| 用 `yarn.ship-files` 传 JDK 包也能用 | ship-files 不解压，.so 文件无法加载，会报动态链接库错误 |
| 在启动命令行中指定这些配置可以生效 | 容器 env 相关配置必须在 config.yaml 中，命令行指定不生效 |
| JAVA_HOME 直接指向压缩包名即可 | 解压后有一层与压缩包同名的目录，JAVA_HOME 需指向里面的实际 JDK 目录 |

## 待验证缺口
- Flink 在 K8s 部署模式下是否也有类似的 JDK 隔离需求，解决方案是否一致（K8s 通常通过镜像解决）
- Hadoop 3.3.6 本身已支持 JDK 11，是否可以直接将集群升到 JDK 11 而非采用 ship-archives 方案

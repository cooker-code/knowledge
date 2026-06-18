---
title: 0929-Cloudera 7.3.1正式GA
author: Hadoop实操
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI4OTY3MTUyNg==&mid=2247505869&idx=1&sn=759837c7819c5416784f275818438a28&chksm=ed340345f2d8562f3d4a26291f91b5122ebb72ea2611d06b480db56684964d38bc938b4d9951&mpshare=1&scene=24&srcid=12132ujdjKJwupeYtL2M5v9T&sharer_shareinfo=cd3de2ab3044236079e858751d7df6ae&sharer_shareinfo_first=cd3de2ab3044236079e858751d7df6ae#rd
---

今天，美国时间双十二，Cloudrea宣布正式发布统一版本（unified release）：Cloudera 7.3.1，之前CDP runtime的软件版本其实是有2套，公有云版本和本地Base版本并不一样，公有云版本要更领先一些，功能也迭代的更快，我们从版本号也看的出来，公有云runtime是7.2.x，线下本地版本当前最新的为7.1.x，很明显7.2是大于7.1的。该统一版本（unified release）的发布，使得用户不管是在公有云还是本地使用CDP，都可以得到一致的功能体验，同时让Cloudera将自己的混合云解决方案往前迈了一大步，它让本地和跨多个云运行的数据和分析能够得到统一管理并联合成一个统一的整体。为什么版本是CDP7.3呢？因为公有云CDP runtime是7.2，私有云CDP runtime是7.1，融合2个版本，尾号相加得到7.3。为什么是CDP7.3.1而不是CDP7.3.0呢？尾号为0的一般都是β版，不适合正式发布来让用户使用，其实内部有7.3.0的版本，主要用于测试和实验为主，从时间上来说也会更早一些。

# 主要更新

* • **ARM芯片支持**：此版本支持 AWS Graviton 处理器，利用基于 Arm 架构的 EC2 实例强大且经济高效的能力。客户现在可以在数据密集型工作负载中实现高达 40% 的性价比提升，使大规模数据处理更加快速且更易访问。Graviton 支持不仅提高了处理速度并降低了运营成本，还体现了Cloudera对灵活、云优化性能的承诺。
* • **Hive与Iceberg集成**：虽然CDP Base 7.1.9的发布中引入了Apache Iceberg，但在计算引擎对Iceberg的支持中主要是Spark，Flink和Impala，是缺失了Hive的支持的，当时只能在公有云/私有云版本的CDW(Cloudera Data Warehouse)的HIve中访问Iceberg。Cloudera 7.3.1的发布弥补这一块的不足，CDP Base正式支持Hive与Iceberg的集成，包括支持Iceberg的典型特性如Schema Evolution, Time Travel, 和ACID。
* • **SQL助手**：Hue 支持通过 大语言模型（LLMs） 提升数据交互体验，推出 AI 驱动的 SQL 助手。用户可以通过自然语言生成 SQL 查询，并轻松完成优化、解释和修复。SQL 助手兼容 OpenAI 的 GPT、Amazon Bedrock 和 Azure 的 OpenAI，方便用户进行灵活的选择。
* • **支持Amazon S3 Express One Zone Storage**：云连接器现在支持 高性能的单可用区存储，提供一致的毫秒级低延迟访问。通过将数据存储在与计算资源相同的可用区中，您可以优化性能、降低成本并加速工作负载。此存储类别支持每秒数十万次请求，实现无缝扩展和高可靠性。
* • **提升的安全性**：此版本修复了超过 225 个通用漏洞和暴露（CVEs），充分体现了Cloudera对数据安全的高度重视，以及为用户提供安全、可靠环境的承诺。每个 CVE 的修复都进一步增强了平台的完整性和稳定性，保护用户免受本地、云端或混合部署环境中的潜在威胁。

# 次要更新

* • **零停机升级（ZDU）**：基于 7.1.9 版本 ZDU 功能的成功，在 7.3.1 版本中继续支持零停机升级（ZDU），进一步简化了升级流程，实现无忧升级，同时保持高可用性，最大限度减少对关键业务操作的影响。
* • **Ozone 与 HBase 集成（技术预览）**：此次集成为 Apache HBase 提供了对象存储解决方案。在 Ozone 上，HBase 现在可以高效处理超大规模表，并提供基于 S3 兼容存储的随机实时读写访问能力，满足大数据场景的需求。
* • **Cloudera Replication Manager增强**：

+ • 支持由 Hive、Impala 和 Spark 创建的 Iceberg V2 表的复制；
+ • Atlas 复制（技术预览）：允许用户复制 Hive 外部表和 Iceberg 表的 Atlas 元数据和数据血缘，从本地部署到本地部署环境。
+ • Ozone 复制扩展：复制范围从仅数据扩展到包括元数据，Ozone 的元数据复制可通过 Hive 外部表复制计划完成。

* • **Spark3标准化**：为从 Spark 2 迁移到Spark3的客户提供了一套增强工具，简化迁移过程。
* • **Cloudera On Cloud和On-Premises功能统一**：通过统一 Cloudera On Cloud 7.2.18 和 Cloudera On-Premises 7.1.9，Cloudera确保了部署环境之间的功能一致性。例如：Apache Hive 组件现在支持 SAML 和 LDAP 的多重身份验证功能，并已在 Cloudera On-Premises 中提供；Apache Oozie 的 OpenJPA 3 升级现已在 Cloudera On Cloud 中可用。
* • **Multi Python support**

+ • Python 3.9: RHEL/Oracle 8.x, RHEL/Oracle 9.x, and Ubuntu 20.
+ • Python 3.10: SLES 15 SP4 and SP5, and Ubuntu 22

* • **升级到Cloudera 7.3.1的路径**

+ • Cloudera 本地部署版本支持直接升级（包括降级和回滚）的版本有：7.1.9 SP1，7.1.8，7.1.7 SP3
+ • Cloudera 云端版本支持直接升级的版本有：7.2.18.0，7.2.18.100，7.2.18.300，7.2.17.200 至 7.2.17.500
+ • 从上述未列出的版本进行升级，可能需要采用 两步升级 的方式

* • **移除的组件**：Apache Spark 2，Apache Livy 2和Apache Zeppelin

Cloudera Manager 7.13.1发布说明：

```
https://docs.cloudera.com/cloudera-manager/7.13.1/manager-release-notes/topics/cm-release-notes-7131.html
```

Cloudera Runtime 7.3.1发布说明：

```
https://docs.cloudera.com/cdp-private-cloud-base/7.3.1/private-release-notes/topics/rt-runtime-overview.html
```
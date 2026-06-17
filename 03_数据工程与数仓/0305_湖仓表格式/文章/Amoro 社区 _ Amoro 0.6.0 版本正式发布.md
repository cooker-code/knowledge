---
title: Amoro 社区 | Amoro 0.6.0 版本正式发布
author: HBase技术社区
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247492096&idx=2&sn=d6b8226909ab367353d59028dfc9475d&chksm=feb6157dc9c19c6bbcee3348cbad8846556c29f7701864dde5ac6bf1465d0920bf2e65d1ffb6&mpshare=1&scene=24&srcid=1108IJJX4JBCv1dV1lS1cz13&sharer_shareinfo=5f9e62858c76f7c8f41cf3f87d347e7b&sharer_shareinfo_first=5f9e62858c76f7c8f41cf3f87d347e7b#rd
---

点击上方蓝字关注我们，了解更多内容

Amoro 是一个构建在 Apache Iceberg 等开放数据湖表格之上的湖仓管理系统，提供了一套可插拔的数据自优化机制和管理服务，旨在为用户带来开箱即用的湖仓使用体验。

**2023 年 11 月 07 日，Amoro 0.6.0 版本正式更新发布！**这个版本在 0.5.1 版本的基础上，增加了很多 feature，并且提升了可用性和稳定性，推荐各位用户和开发者升级到这个版本。在这次版本更新中，来自社区的 21 位贡献者付出了 118 次提交，感谢每位社区小伙伴的贡献！

**0****1**

**重要更新**

**1.Kubernetes 集成**

支持通过 Kubernetes 部署 AMS 和 Optimizer，更多详细内容可以参考文章[《Amoro 0.6.0 前瞻，全面适配 Kubernetes 与 S3》](http://mp.weixin.qq.com/s?__biz=Mzg3OTkzNDgzOA==&mid=2247486804&idx=1&sn=4e9c32aa75885d3835e10f79e66b71f5&chksm=cf7da14df80a285b9734a91991ac6b8e809b74d63ce0390a809fe4f7abcc7a6ed0e8c2b07dc7&scene=21#wechat_redirect)。

**2.与 S3 更友好的集成**

注册 catalog 的时候可以选择 Storage 是 S3，并且支持 AK/SK 验证体系。

**3. Paimon format 支持**

Apache paimon 是一个具备高速数据摄取，变更日志跟踪和高效的实时分析的实时数据湖平台。

**Apache paimon ：https://paimon.apache.org/**

* 在 Catalogs 页面支持支持注册 Paimon catalog。

* 注册完 catalog 以后，可以在 Tables 页面查看表的 Schema, Properties, Files, Snapshots, Optimizing,  Operations等信息。

左右滑动查看更多

* 可以在 Terminal 界面执行 paimon 支持的 Spark sql。

**4.分区及文件过期**

现在只需要在表上进行一些简单的配置，则能开启按照时间自动过期表中文件或分区的功能，如：

```
CREATE TABLE IF NOT EXISTS user (    id INT,    name string,    ts TIMESTAMP) USING iceberg PARTITIONED BY (days(ts));  
ALTER TABLE user SET TBLPROPERTIES (    'data-expire.enabled' = 'true',    'data-expire.level' = 'partition',    'data-expire.field' = 'ts',    'data-expire.retention-time' = '30d');
```

上面的例子开启了 user 表上的分区自动过期功能，AMS 会自动淘汰超过30天的分区。有关分区及文件自动过期的更多信息可以参考最新的用户手册：*https://amoro.netease.com/docs/latest/using-tables/#configure-data-expiration*。

**5.Mixed Format 支持 ORC 文件格式**

Mixed Format 用户可以设置文件存储格式为 ORC 格式。

**6.Mixed Format 支持 Flink-1.16 和 Flink-1.17**

移除了对 Flink-1.12 和 Flink-1.14 的支持，新加了 Flink-1.16 和 Flink-1.17 版本的支持。

**7.优化 Position Delete 的内存使用**

减少了 Self-Optimizing 过程中由于索引 Iceberg 的 position-delete 数据带来的内存消耗。

**0****2**

**Release Note**

Amoro 0.6.0 版本完整的 Release Note 请参考：

*https://github.com/NetEase/amoro/releases/tag/v0.6.0*

**03**

**致谢**

Amoro 社区的发展离不开大量用户的积极试用和反馈，以及社区开发者的无私贡献，再次感谢大家的付出！也欢迎更多小伙伴共同参与到 Amoro 社区建设中！

0.6.0 版本贡献者（排名不分先后）

---

Amoro 新版本试用活动也同步升级。社区在专门的试用群安排 mentor 全程进行一对一辅导，还为试用和贡献的小伙伴精心准备了社区礼包和 AirPods。欢迎大家试用体验 Amoro！

**END**

**看到这里记得关注、点赞、转发 一键三连哦~**

#

**精彩回顾：**

[Amoro 试用&贡献活动 | 9月社区评选揭晓](http://mp.weixin.qq.com/s?__biz=Mzg3OTkzNDgzOA==&mid=2247487005&idx=1&sn=60e4193632487e6e9ebe13a47146802c&chksm=cf7da204f80a2b1203e79fa900af679add87eccb4680e9252e1bd366b5a4285b3be04feafd60&scene=21#wechat_redirect)

[Amoro Mixed Format 上海钢联的构建实时湖仓实践](http://mp.weixin.qq.com/s?__biz=Mzg3OTkzNDgzOA==&mid=2247486328&idx=1&sn=2c509d7eebfcfa506548032a929c66ba&chksm=cf7da761f80a2e770b9c4a827b1cdbffb5a5cb363ee70c56de414f3157644da8e049230ef613&scene=21#wechat_redirect)

[Apache Iceberg + Arctic 构建云原生湖仓实战](http://mp.weixin.qq.com/s?__biz=Mzg3OTkzNDgzOA==&mid=2247485337&idx=1&sn=479a2c9224a2635529709db848f8323e&chksm=cf7dab80f80a229608a806e49eaf0fcbe08c65c8365a50f59386e4f321a1c0b7317d83cb5f98&scene=21#wechat_redirect)

[企查查基于 Apache Iceberg 与 Arctic 构建实时湖仓实践](http://mp.weixin.qq.com/s?__biz=Mzg3OTkzNDgzOA==&mid=2247484483&idx=1&sn=5c7800e1b3ba7416ea8241a0c87f4c3a&chksm=cf7da85af80a214c332d880cdac13a8811f5b1975f74beaec5e6608fee764ea20dd865ce364d&scene=21#wechat_redirect)

**关于 Amoro 的更多资讯可查看：**

**官网：**https://amoro.netease.com/

**源码：**https://github.com/NetEase/amoro

**社群：**后台回复【社群】或扫描下方二维码↓，邀你进群

点击下方【阅读原文】直达 Amoro 0.6.0 Release Note
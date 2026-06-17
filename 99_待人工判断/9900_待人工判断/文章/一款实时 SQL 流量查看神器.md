---
title: 一款实时 SQL 流量查看神器
author: 芋道源码
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247619013&idx=2&sn=8bbc4c72ec3082537a9807cb38f21b87&chksm=fbc0a9a9713bcec32e809076aea58aa989f7fc9f4aef93de20839a2ae81bdbf05a18fe8ad06b&mpshare=1&scene=24&srcid=0401taNj7b3leKCIYvKIfrnZ&sharer_shareinfo=63122213be1527d755132580d926b455&sharer_shareinfo_first=63122213be1527d755132580d926b455#rd
---

> 👉 **这是一个或许对你有用****的社群**
>
> 🐱 一对一交流/面试小册/简历优化/求职解惑，欢迎加入「**[芋道快速开发平台](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)**」知识星球。下面是星球提供的部分资料： 
>
> * [《项目实战（视频）》](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)：从书中学，往事上**“练”**
> * [《互联网高频面试题》](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)：面朝简历学习，春暖花开
> * [《架构 x 系统设计》](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)：摧枯拉朽，掌控面试高频场景题
> * [《精进 Java 学习指南》](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)：系统学习，互联网主流技术栈
> * [《必读 Java 源码专栏》](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&chksm=fa4bd329cd3c5a3fd63e455cf39507d3611a7b040be1381fd5ecfc0a7ce3867575fda4b7313d&scene=21#wechat_redirect)：知其然，知其所以然

> 👉**这是一个或许对你有用的开源项目**
>
> 国产Star破10w的开源项目，前端包括管理后台、微信小程序，后端支持单体、微服务架构
>
> RBAC权限、数据权限、SaaS多租户、**商城**、支付、工作流、大屏报表、**ERP、CRM**、**AI大模型、IoT物联网**等功能：
>
> * 多模块：https://gitee.com/zhijiantianya/ruoyi-vue-pro
> * 微服务：https://gitee.com/zhijiantianya/yudao-cloud
> * 视频教程：https://doc.iocoder.cn
>
> 【国内首批】支持 JDK17/21+SpringBoot3、JDK8/11+Spring Boot2双版本

* [现有方案哪里不行？](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [sql-tap 怎么工作的？](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [三步上手](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)
* [效果展示](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)

---

排查 SQL 性能问题，传统手段各有各的痛：翻慢查询日志发现已经轮转了，加 APM 探针配置繁琐还要改代码，看 `pg_stat_statements` 或 `performance_schema` 又总觉得信息碎片化、缺事务上下文。

**要么侵入性强，要么信息滞后，要么维度单一。**

sql-tap 的思路不一样——它是个**透明代理** ，往应用和数据库之间一插，SQL 流量实时可见，零侵入。

开源地址：https://github.com/mickamy/sql-tap

## [sql-tap 怎么工作的？](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&scene=21#wechat_redirect)

两个组件，分工明确：

**sql-tapd（代理）** ——监听一个端口（比如 `:5433`），应用连它而不是直连数据库。它干两件事：① 双向透明转发全部流量，业务完全无感；② 同时复制一份做协议深度解析。支持 PostgreSQL、MySQL、TiDB。

**sql-tap（TUI 客户端）** ——通过 gRPC 流接收代理解析出的事件，终端里实时展示。每条查询的时间、来源客户端、执行耗时、所属事务——一目了然。

> 基于 Spring Boot + MyBatis Plus + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
>
> * 项目地址：https://github.com/YunaiV/ruoyi-vue-pro
> * 视频教程：https://doc.iocoder.cn/video/

## [三步上手](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&scene=21#wechat_redirect)

**1. 启动代理**

```
sql-tapd -driver postgres -listen :5433 -upstream localhost:5432
```

**2. 改应用连接地址**

把数据库连接串里的 `localhost:5432` 改成 `proxy-host:5433`，重启应用。

**3. 打开监控终端**

```
sql-tap --connect grpc://proxy-host:8080
```

SQL 流量就在终端里实时滚动了。

> 基于 Spring Cloud Alibaba + Gateway + Nacos + RocketMQ + Vue & Element 实现的后台管理系统 + 用户小程序，支持 RBAC 动态权限、多租户、数据权限、工作流、三方登录、支付、短信、商城等功能
>
> * 项目地址：https://github.com/YunaiV/yudao-cloud
> * 视频教程：https://doc.iocoder.cn/video/

## [效果展示](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247576728&idx=1&sn=1298645b025eb51d9078e8c3de7b3c17&scene=21#wechat_redirect)

开源地址：https://github.com/mickamy/sql-tap

---

欢迎加入我的知识星球，全面提升技术能力。

👉 加入方式，**“**长按**”或“**扫描**”下方二维码噢**：

星球的**内容包括**：项目实战、面试招聘、源码解析、学习路线。

```
文章有帮助的话，在看，转发吧。

谢谢支持哟 (*^__^*）
```
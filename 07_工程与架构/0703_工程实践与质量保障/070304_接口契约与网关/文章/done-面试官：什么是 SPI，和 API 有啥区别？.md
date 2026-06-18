> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070304_接口契约与网关/070304_核心知识点/接口契约网关与协议边界准则|接口契约网关与协议边界准则]]、[[07_工程与架构/0703_工程实践与质量保障/070304_接口契约与网关/070304_知识地图|070304_接口契约与网关知识地图]]

---
title: 面试官：什么是 SPI，和 API 有啥区别？
author: 小哈学Java
date:
url: https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247567863&idx=2&sn=28762dca6fc70ee12836e26e433fa1ee&chksm=fc36b15e1ad7e7cbd14119dcb04b3fc2f2019f75f232850b78ab247596ebedd332768c8fee9a&mpshare=1&scene=24&srcid=0209KMUktyQM8hUYUWlY6Iq7&sharer_shareinfo=9c78dd4c73075ad213805a81349bf9d6&sharer_shareinfo_first=9c78dd4c73075ad213805a81349bf9d6#rd
---

在线 Java 面试刷题（持续更新）：https://www.quanxiaoha.com/java-interview

## 面试考察点

面试官问这个问题，主要想考察你以下几个维度的理解：

1. **对接口抽象与实现分离的掌握程度**：你是否理解面向接口编程，以及如何通过解耦来构建可扩展的架构。
2. **对 Java 模块化与服务发现机制的理解**：你是否了解 `SPI (Service Provider Interface)` 这一特定的、在 Java 生态中广泛使用的 **“服务发现”** 机制，而不仅仅停留在 API 作为 “服务提供” 的层面。
3. **对架构中 “调用权” 和 “控制权” 翻转 (IoC) 的认知**：这是核心区别。面试官不仅想知道定义，更想知道 API 和 SPI 在“谁制定接口、谁实现接口、谁发起调用”这个权力关系上的根本不同，以及由此带来的不同应用场景。
4. **是否具备框架设计或使用主流框架（如 JDBC, Spring, Dubbo）的实践经验**：SPI 是许多框架实现可插拔架构的基础，了解它有助于理解这些框架的内部工作原理。

## 核心答案

API (Application Programming Interface) 和 SPI (Service Provider Interface) 都是用于协作和抽象的接口，但它们的视角和“控制权”完全相反。

* **API**： 由服务提供方制定并实现接口，调用方直接依赖并使用该接口来调用服务。控制权在调用方，它“主动”调用已知的、具体的 API。例如，你调用 `HashMap` 的 `put()` 方法。
* **SPI**： 由服务调用方制定接口规范，而由不同的服务提供方去实现。调用方在运行时 “被动” 发现 可用的实现并加载使用。控制权在实现方，框架（调用方）定义了扩展点，第三方（实现方）通过实现这些扩展点来 “注入” 自己的逻辑。例如，JDBC 驱动就是通过 SPI 机制被 Java 核心库发现的。

简单来说：API 是 “我有这个功能，你来调用我”；SPI 是 “我需要这个功能，你们来实现，我来发现和调用你们”。

## 深度解析

### 原理/机制：Java SPI 的核心

Java 标准的 SPI 机制主要依赖于 `java.util.ServiceLoader<S>` 这个类。其核心原理是 “约定优于配置”。

1. **定义接口**：调用方（通常是框架或平台）定义一个接口。
2. **提供实现**：服务提供方（第三方）实现这个接口，并在自己的 JAR 包的 `META-INF/services/` 目录下，创建一个以 接口全限定名 命名的文本文件。文件内容是该接口具体实现类的 全限定名（每行一个）。
3. **发现与加载**：调用方通过 `ServiceLoader.load(InterfaceClass)` 加载当前 ClassPath 下所有符合条件的实现。`ServiceLoader` 会遍历所有 `META-INF/services/` 下的配置文件，并通过反射实例化文件中列出的实现类。
4. **使用**：调用方可以迭代 `ServiceLoader` 来使用所有发现的实现实例。

### 代码示例

假设我们有一个搜索服务，我们希望允许不同的搜索引擎提供商接入。

**1. 定义 SPI 接口（由框架/调用方提供）**

```
// 项目：search-framework
package com.example.spi;

public interface SearchEngine {
    List<Result> search(String keyword);
}
```

**2. 提供 SPI 实现（由服务提供商实现）**

```
// 项目：google-search-provider (一个独立的 JAR)
package com.example.provider.google;

import com.example.spi.SearchEngine;
public class GoogleSearchEngine implements SearchEngine {
    @Override
    public List<Result> search(String keyword) {
        // 调用 Google 搜索 API
        return ...;
    }
}
```

同时，在 `google-search-provider` 项目的资源目录下创建文件：`META-INF/services/com.example.spi.SearchEngine`，其内容为：

```
com.example.provider.google.GoogleSearchEngine
```

**3. 调用方使用 SPI（框架核心代码）**

```
// 项目：search-framework
import java.util.ServiceLoader;

publicclass SearchService {
    public void doSearch(String keyword) {
        // 加载所有 SearchEngine 实现
        ServiceLoader<SearchEngine> engines = ServiceLoader.load(SearchEngine.class);
        
        for (SearchEngine engine : engines) {
            // 使用每一个搜索实现，例如聚合结果或选择第一个
            List<Result> results = engine.search(keyword);
            // ... 处理结果
        }
    }
}
```

### 对比分析

| 维度 | API | SPI |
| --- | --- | --- |
| **英文全称** | Application Programming Interface | Service Provider Interface |
| **概念定位** | 应用程序**编程**接口 | 服务**提供者**接口 |
| **接口所有权** | **提供方** 定义并实现 | **调用方（框架）** 定义，由**实现方**提供实现 |
| **调用方向** | 调用方 **主动调用** 提供方的 API | 调用方 **被动发现并加载** 实现方的 SPI 实现 |
| **关注点** | **如何调用一个服务** （关注功能） | **如何扩展一个框架** （关注扩展性） |
| **耦合方向** | 调用方**依赖**于提供方 | 实现方**依赖**于调用方（框架）的接口规范 |
| **典型例子** | `java.util.List` , `HttpClient` 的方法 | JDBC `Driver`, SLF4J 的桥接器, Spring Boot 自动配置 |

### 最佳实践与注意事项

1. **优点（SPI）**：

* **解耦与可扩展性**： 框架核心无需修改代码，仅通过添加 JAR 包就能接入新功能，实现了 “开闭原则”。
* **面向抽象编程**： 框架代码完全面向接口编程，不关心具体实现。

2. **缺点与常见误区**：

* **Java SPI 会加载所有实现**： `ServiceLoader` 会一次性加载并实例化配置文件中所有实现类，无论是否用到，可能造成资源浪费。一些高级框架（如 Dubbo）会实现自己的、更精细的 SPI 机制来解决此问题。
* **配置信息敏感**： 如果 `META-INF/services/` 下的配置文件被篡改或损坏，会导致加载失败。
* **无法按需加载**： 没有像 IOC 容器那样的按需加载或别名管理能力。
* **不是所有“扩展点”都叫 SPI**： 在广义上，任何允许外部扩展的接口都可以被称为 “SPI”，但面试中通常特指 Java 标准的 `ServiceLoader` 机制或其变种（如 Spring 的 `SpringFactoriesLoader`，用于加载 `spring.factories` 文件）。

3. **广泛应用场景**：

* **JDBC**： `java.sql.Driver` 接口由各大数据库厂商实现，Java 通过 SPI 发现驱动。
* **日志门面**： SLF4J 通过 SPI 机制绑定具体的日志实现（Logback, Log4j2）。
* **Spring Framework**： 大量使用 SPI 思想，例如 `SpringFactoriesLoader` 是实现 Spring Boot 自动配置的核心。
* **Dubbo**： 拥有自己增强的 SPI 实现，支持按名加载、自动包装和依赖注入。

## 总结

API 是服务提供者给消费者使用的契约，控制权在调用者；而 SPI 是框架制定给实现者扩展的契约，控制权在框架，通过 “约定发现” 机制实现解耦和可插拔。 理解 SPI 是深入理解 Java 生态中许多框架 “可扩展” 架构设计的关键。

👉 欢迎[加入小哈的星球](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247566317&idx=1&sn=ede64496766addace122dd32f6cfbdcf&scene=21#wechat_redirect)，你将获得: **专属的项目实战（多个项目） / 1v1 提问 / **Java 学习路线 /**学习打卡 / 每月赠书 / 社群讨论**

* 新项目：《Spring AI 项目实战》正在更新中..., 基于 Spring AI + Spring Boot 3.x + JDK 21;
* 《从零手撸：仿小红书（微服务架构）》 已完结，基于 Spring Cloud Alibaba + Spring Boot 3.x + JDK 17..., [点击查看项目介绍](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247538491&idx=1&sn=576995017721766d0fe15723fd135619&chksm=fd5787bdca200eab54d2fb8ca07fcc2bffdec3eaab4ab82ab5eaf949f0254c1683455e02010b&token=343952052&lang=zh_CN&scene=21#wechat_redirect)；演示地址：http://116.62.199.48:7070/
* **《从零手撸：前后端分离博客项目（全栈开发）》** 2期已完结,演示链接：http://116.62.199.48/;
* 专栏阅读地址：https://www.quanxiaoha.com/column

截止目前，**累计输出 100w+ 字，讲解图 4013+ 张，还在持续爆肝中..** 后续还会上新更多项目，目标是将 Java 领域典型的项目都整一波，如秒杀系统, 在线商城, IM 即时通讯，Spring Cloud Alibaba 等等，[戳我加入学习，解锁全部项目，已有4200+小伙伴加入](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247566317&idx=1&sn=ede64496766addace122dd32f6cfbdcf&scene=21#wechat_redirect)

```
```
```
```
1. 我的私密学习小圈子，从0到1手撸企业实战项目~

2. 面试官：为什么不能用浮点数表示金额？

3. 如何设计一个扛住千万级流量的系统？

4. 面试官：BigDecimal 和 Long 哪个表示金额更合适，怎么选择？
```
```
```
```

```
```
最近面试BAT，整理一份面试资料《Java面试BATJ通关手册》，覆盖了Java核心技术、JVM、Java并发、SSM、微服务、数据库、数据结构等等。

获取方式：点“在看”，关注公众号并回复 Java 领取，更多内容陆续奉上。
```

```
PS：因公众号平台更改了推送规则，如果不想错过内容，记得读完点一下“在看”，加个“星标”，这样每次新文章推送才会第一时间出现在你的订阅列表里。

点“在看”支持小哈呀，谢谢啦
```
```
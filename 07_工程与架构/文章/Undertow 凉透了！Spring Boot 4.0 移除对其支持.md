---
title: Undertow 凉透了！Spring Boot 4.0 移除对其支持
author: Java后端技术
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247514666&idx=1&sn=8d3ef2567f963edad461d8c42f461abb&chksm=e83ff5c499f09e1debfa291e21cc55c29f6c497c46125defd16f7725903ffed5cf700c430aa9&mpshare=1&scene=24&srcid=1029Rj6OPgoxDvxOc0CwWmEh&sharer_shareinfo=74735df0440f4f32809d30c4bbdb0536&sharer_shareinfo_first=74735df0440f4f32809d30c4bbdb0536#rd
---

[上周 Spring Boot 4.0 RC1 正式发布](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490946&idx=1&sn=b6c36e85cb129f4e7518d72ad77d5c80&scene=21#wechat_redirect)，这一里程碑版本带来了众多重要的技术改进和升级。然而，其中一个变更引起了Java开发者社区的广泛关注：**Spring Boot 4.0 完全移除了对 Undertow Web 容器的支持**。

对于正在使用 Undertow 的企业和开发者而言，这一变更意味着项目升级到 Spring Boot 4.0 时必须进行 Web 容器的迁移。这篇文章将深入分析这一变更背后的技术原因，以及相关的技术标准和生态演进。

在 Spring Boot 4.0 中，如果项目中包含以下依赖：

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-undertow</artifactId>  
</dependency>
```

构建过程将会失败，因为 Spring Boot 4.0 已不再包含 Undertow 相关的自动配置和依赖管理。

Spring 团队在 GitHub 上创建了专门的 Issue (#46917) 来跟踪这一变更，确保决策过程的透明度和可追溯性。

Spring 团队在官方文档中明确说明了移除 Undertow 支持的技术原因：

> "Spring Boot 4.0 需要一个 Servlet 6.1 的基线，而 Undertow 目前尚不兼容。因此，我们放弃了对 Undertow 的支持。"

这一声明清晰地指出了问题核心：**版本依赖的不匹配**。这是一个技术兼容性问题，而非技术路线选择或性能优劣的判断。

在相关讨论中，Undertow 团队成员表示 Servlet 6.1 的支持工作已经启动，但截至2025年10月，该工作仍处于早期阶段。

作为 Undertow 的主要维护者，Red Hat 的开发进度相对缓慢，这对于需要升级到 Spring Boot 4.0 的企业而言，提供了有限的选择空间。

## Servlet 6.1 技术特性解析

### 发布背景

**Servlet 6.1** 于2024年4月作为 Jakarta EE 11 的核心子规范发布。相比 Servlet 6.0，这一版本带来了多项重要改进和现代化更新，代表了 Java Web 开发标准的重要演进。

### 核心新特性

#### 1. ByteBuffer 支持

在 `ServletInputStream` 和 `ServletOutputStream` 中新增了 **ByteBuffer 支持**，显著改进了非阻塞 I/O (NIO) 能力。这一改进使开发者能够更高效地处理二进制数据流，特别是在高并发场景下能够获得显著的性能提升。

```
// 使用 ByteBuffer 读取请求数据  
ByteBuffer buffer = ByteBuffer.allocate(1024);  
servletInputStream.read(buffer);
```

#### 2. HTTP/2 推送功能废弃

Servlet 6.1 **正式废弃了 HTTP/2 Server Push** 支持。这一决定反映了该特性在现代 Web 应用中使用率持续下降的现状，开发者更倾向于使用其他优化策略。HTTP/2 Server Push 成为了 HTTP 历史上最短命的重要功能之一。

#### 3. 移除 SecurityManager 相关 API

完全删除了对已废弃的 **Java SecurityManager** 及相关 APIs 的引用，以适应 Java SE 安全模型的演进。这一变更简化了安全架构，移除了历史包袱。

#### 4. HTTP 会话增强机制

提供了新机制，让应用程序能在标准 HTTP 请求处理之外与 HTTP 会话交互，特别是为 **WebSocket** 场景提供了更好的支持，增强了会话管理的灵活性。

#### 5. HTTP 重定向控制增强

开发者现在对发出 HTTP 重定向时的**状态码和响应体**拥有更精细的控制权，可以实现更符合业务需求的重定向逻辑。

```
// 自定义重定向响应  
response.setStatus(HttpServletResponse.SC_MOVED_PERMANENTLY);  
response.setHeader("Location", newUrl);  
response.getWriter().write("Resource has moved");
```

#### 6. 敏感请求头安全处理

新增 `HttpServlet.isSensitiveHeader` 方法，用于识别需要保护的敏感请求头（如 Authorization、Cookie、Forwarded 等）。这些敏感信息会在 TRACE 方法响应中被排除，显著提高了应用程序的安全性。

#### 7. 条件 GET 优化支持

改进了对**条件 GET 操作**的支持，通过 `getLastModified` 方法优化网络资源利用，减少不必要的数据传输，提升了网络效率和响应速度。

## Jakarta EE 11 技术规范

### 规范背景

**Jakarta EE 11** 是企业级 Java 平台的最新标准，其中 **Servlet 6.1** 是其核心子规范。Jakarta EE 11 代表了企业级 Java 应用的标准化演进，为现代企业应用开发提供了统一的技术框架。

### Jakarta Data 规范详解

Jakarta EE 11 通过引入 **Jakarta Data 规范** 来简化企业应用的持久化逻辑。Jakarta Data 为数据访问层提供了标准化的抽象，显著减少了样板代码的编写。

#### 核心特性

##### 1. BasicRepository 基础仓库接口

```
public interface ProductRepository extends BasicRepository<Product, Long> {  
    // 自动获得基础 CRUD 操作：save(), findById(), findAll(), delete()  
}
```

BasicRepository 提供了开箱即用的基本数据操作支持，大幅减少了样板代码的编写和配置工作。

##### 2. CrudRepository 完整 CRUD 功能

```
public interface UserRepository extends CrudRepository<User, String> {  
    List<User> findByEmailContaining(String email);  
    Page<User> findByStatus(UserStatus status, PageRequest page);  
}
```

在 BasicRepository 的基础上，CrudRepository 提供了完整的创建、读取、更新、删除功能，支持复杂的查询方法。

##### 3. Pagination 分页支持

```
// 基于偏移量的分页  
Page<Product> findByCategory(String category, PageRequest pageRequest);  
  
CursoredPage<Product> findByPriceGreaterThan(  
    BigDecimal price,   
    PageRequest pageRequest  
);
```

Jakarta Data 同时支持基于偏移量和基于游标的分页方式，为不同场景提供了灵活的数据访问模式。游标分页在大数据集处理方面具有显著的性能优势。

##### 4. Query Language 查询语言

```
public interface OrderRepository extends CrudRepository<Order, Long> {  
    @Query("SELECT o FROM Order o WHERE o.status = ?1 AND o.total > ?2")  
    List<Order> findHighValueOrders(OrderStatus status, BigDecimal minTotal);  
}
```

引入的简洁查询语言简化了方法级查询的定义，比原生 SQL 更加简洁，比方法名查询更加清晰。

## 主流 Web 容器支持现状对比

### 兼容性矩阵分析

以下表格展示了各主流 Web 容器和框架对 Servlet 6.1 和 Jakarta EE 11 的支持情况：

| 框架/服务器 | 版本 | Servlet 6.1 支持 | Jakarta EE 11 支持 |
| --- | --- | --- | --- |
| Spring Framework | 7.x | 完全支持 | 完全支持 |
| Spring Boot | 4.x | 完全支持 | 完全支持 |
| Tomcat | 11.x+ | 完全支持 | 完全支持 |
| Jetty | 12.1+ | 完全支持 | 完全支持 |
| Undertow | 2.3.x | 不支持 | 不支持 |

从兼容性矩阵中可以清晰看出，Undertow 是唯一尚未支持 Servlet 6.1 的主流 Web 容器，这也是 Spring Boot 4.0 移除对其支持的根本原因。

有趣的是,就在前两年,技术社区还掀起过一股"用 Undertow 替代 Tomcat"的热潮。各路技术大 V 纷纷发文推荐 Undertow,理由包括:

* • **更轻量级的内存占用**：启动更快,资源消耗更少
* • **更优秀的并发性能**：基于 XNIO 的非阻塞 I/O 模型
* • **更灵活的配置**：可编程的服务器配置方式

许多企业和开发者响应号召,在生产环境中大规模采用了 Undertow。Spring Boot 官方文档也一直将 Undertow 列为与 Tomcat、Jetty 并列的三大推荐 Web 容器之一。

然而,时过境迁,当 Jakarta EE 和 Servlet 规范快速演进时,Undertow 的更新速度却明显跟不上节奏。**昔日的"性能之选"如今却成了"升级绊脚石"**。

这个教训告诉我们:**技术选型不应该只看当下的性能指标,更要考虑生态的活跃度和长期演进能力**。有时候,"稳健的主流"比"激进的最优"更值得信赖。

～～回来吧，我的 “汤姆猫”～～

[PIG AI 重磅更新！多模态搜索、OCR、RAG 全面升级！](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490941&idx=1&sn=ca33168edf1d88517d45b7433e02efc5&scene=21#wechat_redirect)

[起猛了，MCP 刚火就过时了？Skills 是什么](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490939&idx=1&sn=ff987cf82148ad63813096041a0d80a3&scene=21#wechat_redirect)

2025-10-20

[别再混淆了！Spring @Value 读配置，Lombok @Value 不可变类，一文搞懂两者区别](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490931&idx=1&sn=232365178342226c74221172e2a693fe&scene=21#wechat_redirect)

2025-10-17

[Vibe Coding 新利器：Exa vs Context7，谁是编程效率王？](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490926&idx=1&sn=dfcea6bbc7373aa12bfa309676add02f&scene=21#wechat_redirect)

2025-10-14

[AI 时代必备：Java 新增的 String 处理的 9 个现代化方法，轻松应对大模型输出](https://mp.weixin.qq.com/s?__biz=MjM5MzEwODY4Mw==&mid=2257490919&idx=1&sn=6b54c530c30f9d0bf8c0bff98df98993&scene=21#wechat_redirect)

2025-10-09
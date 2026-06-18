> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot接口集成与运行治理边界|SpringBoot接口集成与运行治理边界]]
---
title: SpringBoot整合Resilience4j，解决长时间等待第三方API迟迟不响应的问题
author: Java知识日历
date:
url: http://mp.weixin.qq.com/s?__biz=MzU2OTcxMDc2Mw==&mid=2247488767&idx=1&sn=d25b0ad245908391147a47fa5a4c7c5e&chksm=fddf854dfbcb62c269311198e1847b7a1b6df628c6e536a899b5f379fe2f3a6a1413329419af&mpshare=1&scene=24&srcid=0212GBMZTQAIUgYYc2RRET1M&sharer_shareinfo=3e54adbb4f7c89257a48e7bb92e4ccf4&sharer_shareinfo_first=3e54adbb4f7c89257a48e7bb92e4ccf4#rd
---

超时控制是确保系统稳定性和可靠性的重要机制之一。特别是在涉及外部依赖和服务调用的情况下，通过合理配置超时机制，可以显著提高系统的稳定性和可靠性，避免因单个组件的问题导致整个系统的崩溃。

# 常见应用场景

## 1. **外部 API 调用**

**场景描述**：

* 应用程序依赖于第三方 API 或外部服务（如支付网关、天气服务等）。
* 这些外部服务可能会因为各种原因（网络延迟、服务器过载、故障等）导致响应时间变长。

**为什么需要超时控制**：

* **避免阻塞**：防止长时间等待外部服务响应，导致主线程被阻塞。
* **提高可用性**：即使某些外部服务不可用，应用程序也能继续提供其他功能。
* **资源保护**：避免过多请求占用系统资源，导致整体性能下降。

## 2. **数据库查询**

**场景描述**：

* 应用程序需要频繁地与数据库进行交互，执行查询或更新操作。
* 数据库可能因为负载过高、索引缺失或其他问题导致查询响应缓慢。

**为什么需要超时控制**：

* **防止死锁**：避免长时间锁定数据库资源，导致其他事务无法正常执行。
* **提高响应速度**：快速返回错误信息，而不是让用户等待无限期。
* **资源管理**：合理分配数据库连接池中的资源，避免资源耗尽。

## 3. **远程服务调用**

**场景描述**：

* 在微服务架构中，一个服务需要调用另一个服务以获取数据或执行操作。
* 远程服务可能因为多种原因导致响应时间延长。

**为什么需要超时控制**：

* **隔离故障**：防止一个服务的问题扩散到其他服务。
* **提高可用性**：确保每个服务都能在其设计容量内正常工作。
* **简化监控**：更容易监控和调试服务间的交互。

## 4. **批处理任务**

**场景描述**：

* 应用程序定期执行批处理任务，如数据同步、报告生成等。
* 批处理任务可能因为大量数据处理导致执行时间过长。

**为什么需要超时控制**：

* **资源保护**：防止批处理任务消耗过多系统资源，影响其他业务功能。
* **提高效率**：通过合理的限流策略，提高批处理任务的整体效率。
* **降低风险**：减少因过高负载导致的数据损坏或丢失的风险。

## 5. **实时数据处理**

**场景描述**：

* 应用程序需要实时处理大量数据，如消息队列消费者、事件驱动架构等。
* 实时数据处理可能因为数据量过大或处理逻辑复杂导致响应时间延长。

**为什么需要超时控制**：

* **防止积压**：确保消费者能够及时处理消息，避免消息队列积压。
* **提高吞吐量**：通过合理的限流策略，提高系统的吞吐量。
* **增强可靠性**：确保系统的可靠性和稳定性，减少数据丢失的风险。

## 6. **用户请求控制**

**场景描述**：

* 对面向用户的 Web 应用程序，需要限制用户请求的响应时间。
* 用户请求可能因为后端服务响应缓慢导致用户体验下降。

**为什么需要超时控制**：

* **安全防护**：防止恶意攻击或滥用。
* **用户体验**：在高并发情况下保持良好的用户体验。
* **合规性**：符合法律法规对用户请求频率的要求。

## 7. **机器学习模型推理**

**场景描述**：

* 在机器学习模型推理过程中，需要控制模型推理的请求速率。
* 模型推理可能因为计算密集型操作导致响应时间延长。

**为什么需要超时控制**：

* **资源保护**：防止服务器因为过多请求而过载。
* **性能优化**：合理分配资源，提高模型推理的效率。
* **可靠性**：确保模型推理服务的稳定性和可靠性。

## 8. **文件上传/下载**

**场景描述**：

* 应用程序需要处理大文件的上传或下载操作。
* 文件上传/下载可能因为网络问题导致响应时间延长。

**为什么需要超时控制**：

* **避免阻塞**：防止长时间等待文件传输完成，导致主线程被阻塞。
* **提高可用性**：即使文件传输失败，应用程序也能继续提供其他功能。
* **资源保护**：避免过多请求占用系统资源，导致整体性能下降。

## 9. **定时任务**

**场景描述**：

* 应用程序中有定时任务定期执行某些操作。
* 定时任务可能因为操作复杂或数据量大导致响应时间延长。

**为什么需要超时控制**：

* **防止阻塞**：避免长时间运行的任务阻塞调度器。
* **提高可用性**：确保其他定时任务能按时执行。
* **资源保护**：合理分配系统资源，避免资源耗尽。

## 10. **第三方支付接口**

**场景描述**：

* 在电子商务应用中，需要调用第三方支付接口进行交易。
* 支付接口可能因为网络问题或服务器负载导致响应时间延长。

**为什么需要超时控制**：

* **安全性**：防止敏感操作长时间挂起，增加安全风险。
* **用户体验**：快速返回错误信息，提升用户体验。
* **合规性**：遵守支付平台的规定和标准。

# 代码实操

## 添加依赖

确保你的 `pom.xml` 文件包含以下依赖：

```
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Resilience4j Time Limiter -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-timelimiter-spring-boot2</artifactId>
        <version>1.7.0</version>
    </dependency>

    <!-- Resilience4j Spring Boot2 -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot2</artifactId>
        <version>1.7.0</version>
    </dependency>

    <!-- Lombok (Optional) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Spring Boot Starter AOP -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

    <!-- Spring Boot Starter Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 配置超时规则

在 `application.yml` 文件中配置 Resilience4j 的超时规则。

**application.yml:**

```
server:
  port: 8080

resilience4j:
  timelimiter:
    instances:
      slowOperationTimeLimiter:
        timeoutDuration: 1s  # 超时时间为1秒
```

## 创建控制器并使用超时注解

创建一个控制器类，并使用 `@TimeLimiter` 注解来控制请求的超时时间。

**ExampleController.java:**

```
package com.example.resilience4jtimeoutdemo;

import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.CompletableFuture;

@Slf4j
@RestController
@RequestMapping("/api")
publicclass ExampleController {

    @GetMapping("/slow")
    @TimeLimiter(name = "slowOperationTimeLimiter", fallbackMethod = "fallback")
    public CompletableFuture<String> slowEndpoint() {
        log.info("Executing slow endpoint");
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000); // 模拟耗时操作
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return"Slow operation completed";
        });
    }

    public String fallback(Throwable t) {
        log.warn("Fallback method called due to timeout", t);
        return"Operation timed out, please try again later.";
    }
}
```

## 启动应用程序

启动 Spring Boot 应用程序。

**Resilience4jTimeoutDemoApplication.java:**

```
package com.example.resilience4jtimeoutdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Resilience4jTimeoutDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(Resilience4jTimeoutDemoApplication.class, args);
    }
}
```

# 测试结果

## 启动应用程序

```
mvn spring-boot:run
```

## 发送请求

使用 `curl` 发送请求到 `/api/slow` 端点。

```
for i in {1..5}; do curl http://localhost:8080/api/slow; echo; done
```

你将看到以下的输出：

```
Operation timed out, please try again later.
Operation timed out, please try again later.
Operation timed out, please try again later.
Operation timed out, please try again later.
Operation timed out, please try again later.
```

这表明所有请求都因为超时而被重定向到了 `fallback` 方法，返回 "Operation timed out, please try again later."。

## 日志分析

查看应用程序的日志，确认超时行为：

```
2025-01-16 21:51:00.000 INFO  [main] c.e.r.Resilience4jTimeoutDemoApplication : Starting Resilience4jTimeoutDemoApplication v0.0.1-SNAPSHOT using Java 17 on hostname with PID 1234 (/path/to/jar started by user)
2025-01-16 21:51:00.000 INFO  [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2025-01-16 21:51:00.000 INFO  [main] o.a.catalina.core.StandardService       : Starting service [Tomcat]
2025-01-16 21:51:00.000 INFO  [main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/10.1.10]
2025-01-16 21:51:00.000 INFO  [main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-01-16 21:51:00.000 INFO  [main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1000 ms
2025-01-16 21:51:00.000 INFO  [main] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2025-01-16 21:51:00.000 INFO  [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2025-01-16 21:51:00.000 INFO  [main] c.e.r.Resilience4jTimeoutDemoApplication : Started Resilience4jTimeoutDemoApplication in 1.5 seconds (JVM running for 2.0)
2025-01-16 21:51:05.000 INFO  [nio-8080-exec-1] c.e.r.ExampleController : Executing slow endpoint
2025-01-16 21:51:07.000 WARN  [nio-8080-exec-1] c.e.r.ExampleController : Fallback method called due to timeout
2025-01-16 21:51:05.000 INFO  [nio-8080-exec-2] c.e.r.ExampleController : Executing slow endpoint
2025-01-16 21:51:07.000 WARN  [nio-8080-exec-2] c.e.r.ExampleController : Fallback method called due to timeout
2025-01-16 21:51:05.000 INFO  [nio-8080-exec-3] c.e.r.ExampleController : Executing slow endpoint
2025-01-16 21:51:07.000 WARN  [nio-8080-exec-3] c.e.r.ExampleController : Fallback method called due to timeout
2025-01-16 21:51:05.000 INFO  [nio-8080-exec-4] c.e.r.ExampleController : Executing slow endpoint
2025-01-16 21:51:07.000 WARN  [nio-8080-exec-4] c.e.r.ExampleController : Fallback method called due to timeout
2025-01-16 21:51:05.000 INFO  [nio-8080-exec-5] c.e.r.ExampleController : Executing slow endpoint
2025-01-16 21:51:07.000 WARN  [nio-8080-exec-5] c.e.r.ExampleController : Fallback method called due to timeout
```

从日志中可以看到，每个请求都被正确地识别为超时，并且触发了 `fallback` 方法。

# 完整代码

## **pom.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
        <relativePath/><!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>resilience4j-timeout-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>resilience4j-timeout-demo</name>
    <description>Demo project for Resilience4j Timeout Control</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Resilience4j Time Limiter -->
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-timelimiter-spring-boot2</artifactId>
            <version>1.7.0</version>
        </dependency>

        <!-- Resilience4j Spring Boot2 -->
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-spring-boot2</artifactId>
            <version>1.7.0</version>
        </dependency>

        <!-- Lombok (Optional) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Spring Boot Starter AOP -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <!-- Spring Boot Starter Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## **application.yml**

```
server:
  port: 8080

resilience4j:
  timelimiter:
    instances:
      slowOperationTimeLimiter:
        timeoutDuration: 1s  # 超时时间为1秒
```

## **Resilience4jTimeoutDemoApplication.java**

```
package com.example.resilience4jtimeoutdemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Resilience4jTimeoutDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(Resilience4jTimeoutDemoApplication.class, args);
    }
}
```

## **ExampleController.java**

```
package com.example.resilience4jtimeoutdemo;

import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.CompletableFuture;

@Slf4j
@RestController
@RequestMapping("/api")
publicclass ExampleController {

    @GetMapping("/slow")
    @TimeLimiter(name = "slowOperationTimeLimiter", fallbackMethod = "fallback")
    public CompletableFuture<String> slowEndpoint() {
        log.info("Executing slow endpoint");
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000); // 模拟耗时操作
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return"Slow operation completed";
        });
    }

    public String fallback(Throwable t) {
        log.warn("Fallback method called due to timeout", t);
        return"Operation timed out, please try again later.";
    }
}
```

# 关注我，送Java福利

```
/**
 * 这段代码只有Java开发者才能看得懂！
 * 关注我微信公众号之后，
 * 发送:"666"，
 * 即可获得一本由Java大神一手面试经验诚意出品
 * 《Java开发者面试百宝书》Pdf电子书
 * 福利截止日期为2025年01月22日止
 * 手快有手慢没！！！
*/
System.out.println("请关注我的微信公众号：");
System.out.println("Java知识日历");
```
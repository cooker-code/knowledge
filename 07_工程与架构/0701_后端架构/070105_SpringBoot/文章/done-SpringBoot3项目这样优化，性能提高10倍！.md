> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot来源校准与扩展边界|SpringBoot来源校准与扩展边界]]、[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_知识地图|070105_SpringBoot知识地图]]

---
title: SpringBoot3项目这样优化，性能提高10倍！
author: java1234
date:
url: https://mp.weixin.qq.com/s?__biz=MzIxNTAwNjA4OQ==&mid=2247567089&idx=1&sn=063dfec8ba8aee52924c3812a7facbd8&chksm=963d93f54597ea91e58f4d3bfb1974bf4dd85a2079f0dc93adbd96237815b9cb6d15188c9f8d&mpshare=1&scene=24&srcid=0616gXYt36HCWt98YPHkq4cV&sharer_shareinfo=b8bc284f21b29f2a353f5c84e29be044&sharer_shareinfo_first=b8bc284f21b29f2a353f5c84e29be044#rd
---

大家好，我是锋哥。Spring Boot 3 作为开发现代微服务应用的一个热门框架，能够帮助开发者快速构建、部署应用。然而，随着应用的增长，性能瓶颈也可能逐渐显现。今天将介绍几种优化Spring Boot 3项目性能的方式，并通过代码示例，展示如何实现这些优化，以期在某些情况下提高性能达到10倍甚至更多。

## 1. 数据库连接池优化

数据库操作是后端性能的重要瓶颈之一，合适的数据库连接池配置能显著提升性能。Spring Boot 3 默认使用 HikariCP 作为连接池，它是一个性能优秀的数据库连接池。

### 优化点：

* 配置合适的连接池参数，避免频繁创建和销毁数据库连接。
* 调整连接池大小，避免连接池满载造成的阻塞。

### 代码示例：

```
spring:  datasource:    hikari:      maximum-pool-size: 50  # 设置最大连接池大小      minimum-idle: 10       # 设置最小空闲连接数      idle-timeout: 600000   # 设置空闲连接超时      max-lifetime: 1800000  # 设置连接最大存活时间      connection-timeout: 30000  # 设置连接超时
```

通过合理的数据库连接池配置，可以显著减少数据库连接的等待时间，提升数据库操作的并发性能。

## 2. 使用异步请求

对于I/O密集型的任务（如文件上传下载、网络请求等），同步请求的性能瓶颈会导致系统响应缓慢。Spring Boot 3 提供了 `@Async` 注解，支持异步执行任务，减少了线程阻塞的等待时间。

### 优化点：

* 将不需要立即返回的任务转为异步任务，释放线程池，提升吞吐量。

### 代码示例： ``` import org.springframework.scheduling.annotation.Async;import org.springframework.stereotype.Service;@Servicepublic class AsyncService {    @Async    public void performAsyncTask() {        try {            // 模拟耗时操作            Thread.sleep(2000);            System.out.println("异步任务完成");        } catch (InterruptedException e) {            e.printStackTrace();        }    }} ```

### 配置@EnableAsync ``` import org.springframework.boot.web.servlet.ServletComponentScan;import org.springframework.context.annotation.Configuration;import org.springframework.scheduling.annotation.EnableAsync;@Configuration@EnableAsyncpublic class AsyncConfig {} ```

使用 `@Async` 可以将不需要同步处理的任务异步化，提升应用的响应速度和系统吞吐量。

## 3. 使用缓存机制

通过将热点数据缓存到内存中，可以减少重复的数据库查询、计算等操作，显著提高性能。Spring Boot 3 提供了对缓存的良好支持，支持多种缓存技术（如Redis、Caffeine等）。

### 优化点：

* 使用缓存避免频繁查询数据库或执行复杂的计算。

### 代码示例：

### 1，添加Redis依赖（pom.xml）

```
<dependency>    <groupId>org.springframework.boot</groupId>    <artifactId>spring-boot-starter-data-redis</artifactId></dependency>
```

### 2，配置缓存

```
spring:  redis:    host: localhost    port: 6379
```

3，使用缓存

```
import org.springframework.cache.annotation.Cacheable;import org.springframework.stereotype.Service;
@Servicepublic class UserService {
    @Cacheable(value = "users", key = "#userId")    public User getUserById(String userId) {        // 模拟从数据库查询        return database.findUserById(userId);    }}
```

缓存机制可以显著减少数据库的访问频率，提高应用性能。

## 4. 优化Spring Boot启动性能

Spring Boot 启动时加载了大量的自动配置，如果项目过于庞大，启动时间可能会非常长。通过关闭不必要的自动配置，可以显著提高启动速度。

### 优化点：

* 关闭不必要的自动配置，减少启动时的工作量。

### 代码示例： ``` import org.springframework.boot.autoconfigure.EnableAutoConfiguration;import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})public class MyApp {} ```

通过 `@EnableAutoConfiguration(exclude = ...)` 可以排除不必要的自动配置，从而加速应用的启动过程。

## 5. 使用GraalVM优化Java性能

Spring Boot 3支持与GraalVM进行集成，从而通过原生镜像来减少启动时间和内存占用。GraalVM原生镜像可以显著加快Spring Boot应用的启动速度并减少内存消耗。

### 优化点：

* 使用GraalVM编译Spring Boot应用为原生镜像，以提高启动速度和降低内存使用。

### 配置GraalVM

1. 安装GraalVM并配置环境变量。
2. 构建原生镜像

```
./mvnw spring-boot:build-image
```

使用GraalVM构建原生镜像后，可以在几秒钟内启动Spring Boot应用，并显著减少内存占用。

## 6. 优化Spring Security配置

Spring Security是Spring应用中常用的安全框架，但它可能会影响应用的性能。通过优化Spring Security配置，减少不必要的认证和授权检查，可以提升性能。

### 优化点：

* 精简Spring Security的配置，避免不必要的认证过程。

### 代码示例： ``` import org.springframework.security.config.annotation.web.builders.HttpSecurity;import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;@EnableWebSecuritypublic class SecurityConfig extends WebSecurityConfigurerAdapter {    @Override    protected void configure(HttpSecurity http) throws Exception {        http            .authorizeRequests()                .antMatchers("/login", "/public/**").permitAll() // 免登录                .anyRequest().authenticated()            .and()            .csrf().disable(); // 禁用CSRF（在某些场景下可以提升性能）    }} ```

通过调整安全策略，可以减少请求的认证过程，从而提高性能。

Spring Boot 3提供了丰富的配置和扩展性，通过合理的优化策略，可以显著提升项目的性能。通过优化数据库连接池、使用异步任务、缓存机制、启动优化、GraalVM原生镜像以及Spring Security配置，能够有效减少响应时间，提高吞吐量，甚至在某些场景下实现10倍以上的性能提升。在实际项目中，选择合适的优化方式可以帮助你提升系统的稳定性与响应速度。
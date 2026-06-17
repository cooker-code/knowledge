---
title: SpringBoot与Calcite整合，实现多数据源统一查询系统
author: Java知识日历
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU2OTcxMDc2Mw==&mid=2247489791&idx=1&sn=ea2409180a7cd461cdd731400772b9f5&chksm=fd8a8f1670af49766e2230564c2d674d4f31abfd3622449d6348415282db0d1cca5d8d4bf774&mpshare=1&scene=24&srcid=0424il9jaxrvJzFjwSbKgBkL&sharer_shareinfo=e61f8d1530be60c3ff821ba1efbc76cf&sharer_shareinfo_first=e61f8d1530be60c3ff821ba1efbc76cf#rd
---

最近，接到一个电商系统的兼职小单，其中订单信息存储在MySQL数据库，而用户信息存储在PostgreSQL数据库。客户那边想有一个统一查询接口，可以通过SQL查询同时获取这两个数据源的信息。

# 为什么选择Apache Calcite？

## 简化开发流程

* 抽象层次高: Apache Calcite 提供了高层次的抽象，使得开发者可以专注于业务逻辑，而不必处理底层的数据库连接和查询执行细节。
* 减少重复工作: 通过使用Calcite，可以避免重复造轮子，节省开发时间和成本。

## 强大的SQL解析和优化能力

* SQL标准支持: Apache Calcite 支持多种SQL方言（如MySQL、PostgreSQL等），可以无缝地处理不同数据库的SQL语句。
* 查询优化: 内置的查询优化器可以根据不同的数据源特性进行智能优化，提高查询性能。

## 灵活性和可扩展性

* 自定义模式和表: 可以通过编程方式动态地添加和管理多个数据源，每个数据源可以有不同的模式和表结构。
* 插件机制: 支持各种插件，可以根据需求灵活扩展功能，例如自定义函数、聚合操作等。

## 高性能

* 内存计算: Apache Calcite 支持内存中的数据处理，减少了I/O开销，提高了查询速度。
* 分布式计算: 虽然本项目主要关注单机版实现，但Apache Calcite也可以扩展到分布式环境中，支持大规模数据集的处理。

## 集成性强

* 与其他工具集成: 支持与其他大数据工具和技术栈（如Apache Flink、Presto等）集成，形成完整的数据分析解决方案。

# 哪些公司使用了Apache Calcite？

* Google 在其内部的一些数据处理系统中使用 Apache Calcite，特别是在需要高性能和灵活性的场景下。
* IBM 在其数据仓库和分析解决方案中使用 Apache Calcite，以提高查询性能和灵活性。
* Intel 使用 Apache Calcite 来支持其大数据分析工具和解决方案，特别是在内存计算方面。
* Alibaba Cloud: 阿里巴巴云在其大数据平台中使用 Apache Calcite 提供强大的查询优化和执行能力。
* MaxCompute (ODPS): 阿里巴巴的大规模数据计算服务 MaxCompute 使用 Calcite 进行 SQL 查询处理。
* Elasticsearch 的某些高级功能，如 Kibana 中的复杂查询，依赖于 Apache Calcite 进行 SQL 解析和优化。
* Netflix 使用 Apache Calcite 来构建其内部的数据虚拟化层，支持复杂的查询和数据分析需求。
* Microsoft 在其一些大数据产品和服务中使用 Apache Calcite，例如 Azure Synapse Analytics。
* Teradata 使用 Apache Calcite 来增强其数据库系统的查询优化和执行性能。
* Uber 使用 Apache Calcite 来处理其庞大的数据集，并支持复杂的查询和数据分析需求。

# 应用场景

## 数据虚拟化

* 虚拟数据层: 创建一个虚拟的数据层，将分散在不同系统中的数据集中起来，提供统一的视图。
* 动态数据源管理: 动态地添加和管理数据源，支持灵活的数据架构设计。

## 商业智能 (BI) 工具

* 报表生成: 作为 BI 工具的核心组件，支持复杂的报表生成和数据分析。
* 自助服务分析: 提供自助服务分析功能，允许非技术人员进行数据探索和分析。

## 机器学习与人工智能

* 特征工程: 在机器学习管道中使用 Calcite 进行特征提取和数据准备。
* 模型训练: 结合其他 AI 框架，利用 Calcite 进行大规模数据集的查询和处理。

## 多数据源查询

* 统一接口访问多个数据库: 允许用户通过单一接口查询存储在不同数据库（如 MySQL、PostgreSQL、Oracle 等）中的数据。
* 联合查询: 支持跨数据源的复杂 SQL 查询，例如从不同的数据库中获取相关联的数据。

## 大数据平台集成

* 与 Hadoop 生态系统集成: 与 Hive、HBase、Druid 等大数据工具结合，提供统一的查询接口。
* 流处理与批处理: 支持 Apache Flink 和 Apache Beam 等流处理框架，实现实时数据分析。

## 嵌入式数据库

* 轻量级数据库引擎: 提供一个轻量级的 SQL 引擎，适用于嵌入式应用程序和内存数据库。
* 内存计算: 利用内存计算加速查询性能，适合需要快速响应的应用场景。

## 数据湖解决方案

* 统一元数据管理: 提供统一的元数据管理和查询接口，方便数据湖的建设和维护。
* 多样化数据格式支持: 支持多种数据格式（如 JSON、Parquet、ORC 等），满足不同类型的数据存储需求。

# 代码实操

```
<dependencies>  
    <!-- Spring Boot Starter Web -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
  
    <!-- Apache Calcite Core -->  
    <dependency>  
        <groupId>org.apache.calcite</groupId>  
        <artifactId>calcite-core</artifactId>  
        <version>1.32.0</version>  
    </dependency>  
  
    <!-- HikariCP Connection Pool -->  
    <dependency>  
        <groupId>com.zaxxer</groupId>  
        <artifactId>HikariCP</artifactId>  
    </dependency>  
  
    <!-- MySQL Connector Java -->  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <scope>runtime</scope>  
    </dependency>  
  
    <!-- PostgreSQL JDBC Driver -->  
    <dependency>  
        <groupId>org.postgresql</groupId>  
        <artifactId>postgresql</artifactId>  
        <scope>runtime</scope>  
    </dependency>  
  
    <!-- Test Dependencies -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-test</artifactId>  
        <scope>test</scope>  
    </dependency>  
</dependencies>
```

# application.yml

```
spring:  
  datasource:  
    order-db:  
      url:jdbc:mysql://localhost:3306/order_db?useSSL=false&serverTimezone=UTC  
      username:root  
      password:root  
      driver-class-name:com.mysql.cj.jdbc.Driver  
    user-db:  
      url:jdbc:postgresql://localhost:5432/user_db  
      username:postgres  
      password:postgres  
      driver-class-name:org.postgresql.Driver  
  
jpa:  
    show-sql:true  
    hibernate:  
      ddl-auto:update  
    properties:  
      hibernate:  
        dialect:org.hibernate.dialect.MySQL8Dialect
```

# 数据源配置

```
package com.example.multids.config;  
  
import com.zaxxer.hikari.HikariConfig;  
import com.zaxxer.hikari.HikariDataSource;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
import javax.sql.DataSource;  
  
@Configuration  
publicclass DataSourceConfig {  
  
    @Bean(name = "mysqlDataSource")  
    public DataSource mysqlDataSource() {  
        // 配置MySQL数据源  
        HikariConfig config = new HikariConfig();  
        config.setJdbcUrl("jdbc:mysql://localhost:3306/order_db?useSSL=false&serverTimezone=UTC");  
        config.setUsername("root");  
        config.setPassword("root");  
        returnnew HikariDataSource(config);  
    }  
  
    @Bean(name = "postgresDataSource")  
    public DataSource postgresDataSource() {  
        // 配置PostgreSQL数据源  
        HikariConfig config = new HikariConfig();  
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/user_db");  
        config.setUsername("postgres");  
        config.setPassword("postgres");  
        returnnew HikariDataSource(config);  
    }  
}
```

# 自定义数据源工厂

```
package com.example.multids.factory;  
  
import com.example.multids.schema.MySchemas;  
import org.apache.calcite.adapter.jdbc.JdbcSchema;  
import org.apache.calcite.adapter.java.ReflectiveSchema;  
import org.apache.calcite.config.Lex;  
import org.apache.calcite.jdbc.CalciteConnection;  
  
import javax.sql.DataSource;  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
  
publicclass DataSourceFactory {  
  
    public static CalciteConnection createConnection(DataSource mysqlDataSource, DataSource postgresDataSource) throws SQLException {  
        // 定义Calcite模型JSON字符串  
        String modelJson = "{\n" +  
                "  \"version\": \"1.0\",\n" +  
                "  \"defaultSchema\": \"my_schemas\",\n" +  
                "  \"schemas\": [\n" +  
                "    {\n" +  
                "      \"name\": \"my_schemas\",\n" +  
                "      \"type\": \"custom\",\n" +  
                "      \"factory\": \"" + ReflectiveSchema.Factory.class.getName() + "\",\n" +  
                "      \"operand\": {\n" +  
                "        \"class\": \"" + MySchemas.class.getName() + "\"\n" +  
                "      }\n" +  
                "    }\n" +  
                "  ]\n" +  
                "}";  
          
        // 创建Calcite连接  
        Connection connection = DriverManager.getConnection("jdbc:calcite:model=" + modelJson);  
        CalciteConnection calciteConnection = connection.unwrap(CalciteConnection.class);  
  
        // 获取根模式并添加子模式  
        SchemaPlus schema = calciteConnection.getRootSchema().getSubSchema("my_schemas");  
        schema.add("orders", JdbcSchema.create(calciteConnection.getRootSchema(), "orders", mysqlDataSource, null, Lex.MYSQL));  
        schema.add("users", JdbcSchema.create(calciteConnection.getRootSchema(), "users", postgresDataSource, null, Lex.POSTGRESQL));  
  
        return calciteConnection;  
    }  
}
```

# 自定义模式

```
package com.example.multids.schema;  
  
import org.apache.calcite.schema.impl.AbstractSchema;  
  
import java.util.Map;  
  
public class MySchemas extends AbstractSchema {  
    @Override  
    protected Map<String, org.apache.calcite.schema.Table> getTableMap() {  
        // 返回表映射，这里不需要额外处理  
        return super.getTableMap();  
    }  
}
```

# 查询控制器

```
package com.example.multids.controller;  
  
import com.example.multids.factory.DataSourceFactory;  
import org.apache.calcite.jdbc.CalciteConnection;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.beans.factory.annotation.Qualifier;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
import org.springframework.web.bind.annotation.RestController;  
  
import javax.sql.DataSource;  
import java.sql.ResultSet;  
import java.sql.SQLException;  
import java.sql.Statement;  
import java.util.ArrayList;  
import java.util.List;  
  
@RestController  
publicclass QueryController {  
  
    privatefinal DataSource mysqlDataSource;  
    privatefinal DataSource postgresDataSource;  
  
    @Autowired  
    public QueryController(@Qualifier("mysqlDataSource") DataSource mysqlDataSource,  
                           @Qualifier("postgresDataSource") DataSource postgresDataSource) {  
        this.mysqlDataSource = mysqlDataSource;  
        this.postgresDataSource = postgresDataSource;  
    }  
  
    @GetMapping("/query")  
    public List<List<String>> query(@RequestParam String sql) throws SQLException {  
        // 创建Calcite连接  
        CalciteConnection connection = DataSourceFactory.createConnection(mysqlDataSource, postgresDataSource);  
        Statement statement = connection.createStatement();  
        ResultSet resultSet = statement.executeQuery(sql);  
  
        // 处理查询结果  
        List<List<String>> result = new ArrayList<>();  
        while (resultSet.next()) {  
            int columnCount = resultSet.getMetaData().getColumnCount();  
            List<String> row = new ArrayList<>();  
            for (int i = 1; i <= columnCount; i++) {  
                row.add(resultSet.getString(i));  
            }  
            result.add(row);  
        }  
  
        // 关闭资源  
        resultSet.close();  
        statement.close();  
        connection.close();  
  
        return result;  
    }  
}
```

# Application

```
package com.example.multids;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
  
@SpringBootApplication  
public class MultidsApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(MultidsApplication.class, args);  
    }  
}
```

# 测试

## MySQL `orders` 表

```
CREATE TABLE orders (  
    id INT PRIMARY KEY,  
    user_id INT,  
    amount DECIMAL(10, 2),  
    order_date DATETIME  
);
```

## PostgreSQL `users` 表

```
CREATE TABLE users (  
    id SERIAL PRIMARY KEY,  
    name VARCHAR(100),  
    email VARCHAR(100)  
);
```

测试执行一个联合查询，从两个不同的数据源中获取数据，SQL语句是:

```
SELECT o.id AS order_id, u.name AS user_name, o.amount, o.order_date  
FROM orders o  
JOIN users u ON o.user_id = u.id;
```

## 测试结果

```
$ curl -X GET "http://localhost:8080/query?sql=SELECT%20o.id%20AS%20order_id,%20u.name%20AS%20user_name,%20o.amount,%20o.order_date%20FROM%20orders%20o%20JOIN%20users%20u%20ON%20o.user_id%20=%20u.id"  
  
[  
    ["1", "Alice", "199.99", "2025-04-10 21:30:00"],  
    ["2", "Bob", "250.75", "2025-04-10 20:45:00"]  
]
```

# 关注我，送Java福利

```
/**  
 * 这段代码只有Java开发者才能看得懂！  
 * 关注我微信公众号之后，  
 * 发送:"666"，  
 * 即可获得一本由Java大神一手面试经验诚意出品  
 * 《Java开发者面试百宝书》Pdf电子书  
 * 福利截止日期为2025年02月28日止  
 * 手快有手慢没！！！  
*/  
System.out.println("请关注我的微信公众号：");  
System.out.println("Java知识日历");
```
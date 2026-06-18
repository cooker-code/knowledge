> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot来源校准与扩展边界|SpringBoot来源校准与扩展边界]]、[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_知识地图|070105_SpringBoot知识地图]]

---
title: SpringBoot + Apache tika 轻松实现各种文档内容解析
author: 小哈学Java
date:
url: http://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247529107&idx=3&sn=a86b0085e278e3cb21d297bbd83a4925&chksm=fd57a215ca202b035fcf6f0af4172b8a66e0fbce1992e3c17c9f019b5e5b66e0c98da84242a4&mpshare=1&scene=24&srcid=0311U0QSAmxXUashPByJueqh&sharer_shareinfo=80d951090f157b868c0f2da8df2d7cd6&sharer_shareinfo_first=80d951090f157b868c0f2da8df2d7cd6#rd
---

> 👉 欢迎[加入小哈的星球](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247520263&idx=1&sn=62c4b0a4e307f177ea76eb93f469c0b7&chksm=fd574081ca20c99768c6f513eb4d373281b03d0051b6cc439b079611e64a9b958a6d175f1a39&token=981697568&lang=zh_CN&scene=21#wechat_redirect) ，你将获得: **专属的项目实战 / Java 学习路线 / 一对一提问 / 学习打卡 /  赠书福利**
>
> **全栈前后端分离博客项目 1.0 版本完结啦，2.0 正在更新中..., **演示链接**：**http://116.62.199.48/ ，全程手摸手，后端 + 前端全栈开发，从 0 到 1 讲解每个功能点开发步骤，1v1 答疑，直到项目上线。**目前已更新了219小节，累计36w+字，讲解图：1492张，还在持续爆肝中..** 后续还会上新更多项目，目标是将Java领域典型的项目都整一波，如秒杀系统, 在线商城, IM即时通讯，Spring Cloud Alibaba 等等，[戳我加入学习，已有1000+小伙伴加入(早鸟价超低)](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247522465&idx=1&sn=49da64cae0a62b0c2beac8e0ad2b420e&chksm=fd574827ca20c1314c036377d7698e1fe12a7e716a15a0cfb56d5870f209c3dded22dd848d26&token=621118242&lang=zh_CN&scene=21#wechat_redirect)

Apache tika是Apache开源的一个文档解析工具。Apache Tika可以解析和提取一千多种不同的文件类型(如PPT、XLS和PDF)的内容和格式，并且Apache Tika提供了多种使用方式，既可以使用图形化操作页面（tika-app），又可以独立部署（tika-server）通过接口调用，还可以引入到项目中使用。

本文演示在spring boot 中引入tika的方式解析文档。如下：

## **引入依赖**

在spring boot 项目中引入如下依赖:

```
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.apache.tika</groupId>
        <artifactId>tika-bom</artifactId>
        <version>2.8.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

    <dependency>
      <groupId>org.apache.tika</groupId>
      <artifactId>tika-core</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tika</groupId>
      <artifactId>tika-parsers-standard-package</artifactId>
    </dependency>
```

## **创建配置**

将tika-config.xml文件放在resources目录下。tika-config.xml文件的内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<properties>
    <encodingDetectors>
        <encodingDetector class="org.apache.tika.parser.html.HtmlEncodingDetector">
            <params>
                <param name="markLimit" type="int">64000</param>
            </params>
        </encodingDetector>
        <encodingDetector class="org.apache.tika.parser.txt.UniversalEncodingDetector">
            <params>
                <param name="markLimit" type="int">64001</param>
            </params>
        </encodingDetector>
        <encodingDetector class="org.apache.tika.parser.txt.Icu4jEncodingDetector">
            <params>
                <param name="markLimit" type="int">64002</param>
            </params>
        </encodingDetector>
    </encodingDetectors>
</properties>
```

创建配置类MyTikaConfig

```
import java.io.IOException;
import java.io.InputStream;
import org.apache.tika.Tika;
import org.apache.tika.config.TikaConfig;
import org.apache.tika.detect.Detector;
import org.apache.tika.exception.TikaException;
import org.apache.tika.parser.AutoDetectParser;
import org.apache.tika.parser.Parser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.xml.sax.SAXException;

/**
 * tika配置类
 */
@Configuration
public class MyTikaConfig {

    @Autowired
    private ResourceLoader resourceLoader;

    @Bean
    public Tika tika() throws TikaException, IOException, SAXException {

        Resource resource = resourceLoader.getResource("classpath:tika-config.xml");
        InputStream inputStream = resource.getInputStream();

        TikaConfig config = new TikaConfig(inputStream);
        Detector detector = config.getDetector();
        Parser autoDetectParser = new AutoDetectParser(config);

        return new Tika(detector, autoDetectParser);
    }
}
```

Tika类中提供了文芳detect、translate和parse功能， 在项目中通过注入TIka, 就可以使用了

## **在项目使用**

配置完成后在项目中可以通过注入TIka即可完成文档的解析。如下图所示：

图片

> 👉 欢迎[加入小哈的星球](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247520263&idx=1&sn=62c4b0a4e307f177ea76eb93f469c0b7&chksm=fd574081ca20c99768c6f513eb4d373281b03d0051b6cc439b079611e64a9b958a6d175f1a39&token=981697568&lang=zh_CN&scene=21#wechat_redirect) ，你将获得: **专属的项目实战 / Java 学习路线 / 一对一提问 / 学习打卡 /  赠书福利**
>
> **全栈前后端分离博客项目 1.0 版本完结啦，2.0 正在更新中..., **演示链接**：**http://116.62.199.48/ ，全程手摸手，后端 + 前端全栈开发，从 0 到 1 讲解每个功能点开发步骤，1v1 答疑，直到项目上线。**目前已更新了219小节，累计36w+字，讲解图：1492张，还在持续爆肝中..** 后续还会上新更多项目，目标是将Java领域典型的项目都整一波，如秒杀系统, 在线商城, IM即时通讯，Spring Cloud Alibaba 等等，[戳我加入学习，已有1000+小伙伴加入(早鸟价超低)](https://mp.weixin.qq.com/s?__biz=MzU4MDUyMDQyNQ==&mid=2247522465&idx=1&sn=49da64cae0a62b0c2beac8e0ad2b420e&chksm=fd574827ca20c1314c036377d7698e1fe12a7e716a15a0cfb56d5870f209c3dded22dd848d26&token=621118242&lang=zh_CN&scene=21#wechat_redirect)

```
```
```
```
1. 我的私密学习小圈子~

2. 别再用 offset 和 limit 分页了，性能太差！

3. SpringBoot 监控 SQL 运行情况（实战教程）

4. JDK 21：GC 不断改进，性能更上一层楼
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
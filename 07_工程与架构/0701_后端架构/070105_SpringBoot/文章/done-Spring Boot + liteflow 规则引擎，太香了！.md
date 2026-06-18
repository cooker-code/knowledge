> 已吸收至：[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_核心知识点/SpringBoot来源校准与扩展边界|SpringBoot来源校准与扩展边界]]、[[07_工程与架构/0701_后端架构/070105_SpringBoot/070105_知识地图|070105_SpringBoot知识地图]]

---
title: Spring Boot + liteflow 规则引擎，太香了！
author: Java团长
date:
url: http://mp.weixin.qq.com/s?__biz=MzIwMTY0NDU3Nw==&mid=2651969396&idx=1&sn=e269f19f67f1bfab573ce30cfd432ccc&chksm=8d0fb83aba78312c9f3652cc0a16a7291ef77a3cefd604317412a23272c68f96cc06e9603a42&mpshare=1&scene=24&srcid=0304GNhkxgJXwUIUc1dVbyJo&sharer_shareinfo=fb6170757a034c81ee8b29e250191756&sharer_shareinfo_first=fb6170757a034c81ee8b29e250191756#rd
---

## 1、前言

在日常的开发过程中，经常会遇到一些串行或者并行的业务流程问题，而业务之间不必存在相关性。在这样的场景下，使用策略和模板模式的结合可以很好的解决这个问题，但是使用编码的方式会使得文件太多,在业务的部分环节可以这样操作，在项目角度就无法一眼洞穿其中的环节和逻辑。在本文中，将引入规则引擎从全局角度来解决这个问题，这就是今天要介绍的主角 liteflow。

## 2、liteflow 规则引擎

liteflow 是一个轻巧而且强大的规则引擎，能够实现开箱即用，可以在短时间内就可以完成复杂的规则编排，下图是 liteflow 的整体架构。liteflow 支持较多的规则文件格式，比如 xml/json/yaml， 对于规则文件的存储方式可以有sql/zk/nacos/apollo 等。

liteflow 的使用是从获取上下文开始的，通过数据上下文来解析对应的规则文件，通过 liteflow 执行器来执行对应的链路，每个链路上都有需要执行的业务 node（即节点组件，可以支持多种语言脚本， groovy/js/python/lua等）, 各个业务node 之间是独立的。liteflow 对应的官方网址和依赖如下所示：

```
# liteflow 规则引擎官方网址
https://liteflow.yomahub.com
# springboot 集成 liteflow
<dependency>
    <groupId>com.yomahub</groupId>
    <artifactId>liteflow-spring-boot-starter</artifactId>
    <version>2.10.6</version>
</dependency>
```

liteflow 可以支持如下所示的复杂流程

此外，liteflow 可以支持热部署，可以实时替换或者增加节点，即修改规则文件后可以实时生效。

## 3、liteflow 的使用方法

### 3.1 组件

liteflow 的组件在规则文件中即对应的节点，组件对应的种类有很多，具体的如下所示：

* 普通组件

普通组件需要集成的是 NodeComponent, 可以用在 when 和 then 逻辑中，具体的业务需要在 process 中去执行。同时在 node 节点中，可以覆盖 iaAccess 方法，表示是否进入该节点执行业务逻辑，isContinueOnError 判断在出错的情况下是否继续执行下一个组件，默认为 false。 isEnd 方法表示是否终止流程，默认为true。

* 选择组件

选择组件是通过业务逻辑来判断接下来的动作要执行哪一个节点，类似于 Java中的 switch , 在代码中则需要继承 NodeSwitchComponent 实现 processWitch 方法来处理业务。

```
# flow 规则表达式 选择组件
SWITCH(a).to(b, c);
# processWitch 表达式需要返回的是 b 或者 c 字符串来执行相应的业务逻辑
# flow 规则表达式 条件组件
IF(x, a, b);
```

* 条件组件

条件组件称之为 if 组件，返回的结果是 true 或者 false, 代码需要集成 NodeIfComponent 重写 processIf 方法，返回对应的业务节点，这个和选择组件类似。

在官方文档中，还有次数循环组件，条件循环组件，循环迭代组件，和退出循环组件，作者认为其应用场景比较复杂，可以使用简单的普通组件来替代，毕竟是轻量级的规则引擎，主要作用就是为了编排流程顺序，复杂的场景就升级使用工作流了，所以这里只介绍以上三种组件。

### 3.2 EL 规则文件

规则文件的编写和组件的使用是对应的，这里使用xml 的形式来编写。

```
# 文件编排， then 代表串行执行  when 表示并行执行
# 串行编排示例
THEN(a, b, c, d);
# 并行编排示例
WHEN(a, b, c);
# 串行和并行嵌套结合
THEN( a, WHEN(b, c, d), e);
# 选择编排示例
SWITCH(a).to(b, c, d);
# 条件编排示例
THEN(IF(x, a),b );
```

### 3.3 数据上下文

在 liteflow 中，数据上下文的概念非常重要，上下文对象起到参数传递的作用，因为不同业务需要的输入输出参数是不同的，所以上下文非常的重要。

```
# 执行流程时，需要传递el文件，初始化参数以及上下文对象，这里的上下文可以设置多个
LiteflowResponse response = flowExecutor.execute2Resp("chain1", 流程初始参数, CustomContext.class);
```

因为上下文传入的是一个 class 类型参数，流程参数是可以传入参数的，一般情况下是在第一个节点中，将传入参数设置到上下文对象中。

### 3.4 参数配置

在 liteflow 中，需要配置的内容有规则文件地址，节点重试（执行报错时可以进行重试，类似于 spring-retry）, 流程并行执行线程池参数配置，流程的请求ID配置。

```
liteflow:
  # 规则文件 失败重试次数 打印执行日志 监控日志
  ruleSource : liteflow/*.el.xml
  retry-count: 0
  print-execution-log: true
  monitor:
    enable-log: true
    period: 300000
  request-id-generator-class: com.platform.orderserver.config.AppRequestIdGenerator
  # 上下文的最大数量槽
  slot-size : 10240
  # 线程数，默认为64
  main-executor-works: 64
  # 异步线程最长等待时间 秒
  when-max-wait-seconds: 15
  # when 节点全局异步线程池最大线程数
  when-max-workers: 16
  # when 节点全局异步线程池队列数
  when-queue-limit: 5120
  # 在启动的时候就解析规则
  parse-on-start: true
  enable: true
```

## 4、业务实践

之前介绍了 liteflow 的一些概念，在这里结合一些业务场景来进行演示:

```
# 主要使用电商场景的应用，订单完成后，进行积分的发放，消息发送，同时并行发送短信和邮件。
<?xml version="1.0" encoding="UTF-8"?>
<flow>
    <chain name="test_flow">
        THEN(
           prepareTrade, grantScore, sendMq, WHEN(sendEmail, sendPhone)
        );
    </chain>
</flow>
```

在订单完成之后异步执行，传递参数并执行相应的规则流程。

在正式处理业务流程之前，需要先进行数据的预处理，将流程入参转转换成上下文对象，方便参数的传递和设置。

在具体的业务处理环节，以积分发放为例，可以获取上下文对象进行业务操作，同时也可以重写 isAccess 方法，来判断是否处理该节点。

如上图所示，具体的业务流程都可以抽象成一个 node 节点，存放在 test\_flow.el.xml 中进行执行，其它的代码都和积分发放类似，这里就不再赘述，可以参见代码。

## 5、总结

liteflow 中的绝大部分是在启动时完成的，包括规则解析、注册组件以及组装信息，其执行性能很高，同时也可以打印每个业务环节的耗时以及统计信息。在本文中，介绍了 liteflow 的基本概念，以及具体的使用方法，具体的代码已经上传到 github, 欢迎大家点赞和 stars。项目 github 地址：https://github.com/thedestiny/springboot-auth/tree/main/order-server
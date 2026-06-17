---
title: Apache Kafka 中的认证、鉴权原理与应用
author: AutoMQ
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486475&idx=1&sn=4edd2d50649e74883e39218a6dae1d71&chksm=c0c21d0ae34f0f2203d7a4738e4e8d01309e8323818f3407501e96cbbef42e01a29b96c23b69&mpshare=1&scene=24&srcid=0927Cx9sDE2OQeiYqs6QmQ7u&sharer_shareinfo=2390a247616f60732a3d2e15a99b1a8b&sharer_shareinfo_first=2390a247616f60732a3d2e15a99b1a8b#rd
---

**编辑导读**

本篇内容将进一步介绍 Kafka 中的认证、鉴权等概念。AutoMQ 是与 Apache Kafka 100% 完全兼容的新一代 Kafka，可以帮助用户降低 90%以上的 Kafka 成本并且进行极速地自动弹性。作为 Kafka 生态的忠实拥护者，我们也会持续致力于传播 Kafka 技术，欢迎关注我们。

我们在此前的文章[《AutoMQ SASL 安全身份认证配置教程》](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486121&idx=1&sn=8a73ed6f490b208c51bd4250b9516fa0&chksm=c1bc2ce0f6cba5f6e7a36666522bb20674bcaf00b61c1c34a6140ac687deec1a46dfd0ec0663&scene=21#wechat_redirect)[1]介绍过 Apache Kafka （以下简称 Kafka）服务端和客户端的 SASL 认证协议配置，并在[《AutoMQ SSL 安全协议配置教程》](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485921&idx=1&sn=ab0020763987256b9f89a3377b2d2c12&chksm=c1bc2fa8f6cba6be33625a98dcf37d0182ef5fb04f548185f104ac052c9637578cfa4b8bd738&scene=21#wechat_redirect)[2]详细介绍了如何利用 SSL（TLS） 实现 Kafka 或 AutoMQ 的安全通信。本文将进一步概述 Kakfa 中的认证方法和鉴权策略，并通过一个例子来说明我们在真实应用场景中如何对 Kafka 或者 AutoMQ 集群开启身份认证和鉴权。

注意：本文默认 Kafka 集群是以 Kraft 模式工作的

**01**

**Listener、安全协议、认证与鉴权的关系**

先回顾一下 listener 的作用。listener 是 Kafka 服务端定义监听地址（域名/IP + 端口）和安全协议的实体。一般情况下，我们可以利用多 listener 来差异化设置：

* broker 与 broker 之间，controller 与 broker 之间，client 与 broker 之间的通信安全协议；
* 内网（局域网或 VPC 内部）访问 broker、外网访问 broker 时的通信安全协议；

我们可以将 listener name 跟安全协议进行映射，举例来说，Kafka 支持以下安全协议：

* PLAINTEXT：无认证的明文协议；
* SSL：使用 TLS 加密通信；
* SASL\_PLAINTEXT：使用 SASL 进行身份认证，通信还是明文；
* SASL\_SSL：使用 SASL 进行身份认证，并使用 TLS 加密通信；

我们可以设置 client 与 broker 之间使用 SASL\_SSL 协议保证加密性，而集群节点之间使用 SASL\_PLAINTEXT 减少 CPU 加密解密的消耗：

```
listeners=EXTERNAL://:9092,BROKER://10.0.0.2:9094advertised.listeners=EXTERNAL://broker1.example.com:9092,BROKER://broker1.local:9094listener.security.protocol.map=EXTERNAL:SASL_SSL,BROKER:SASL_PLAINTEXT inter.broker.listener.name=BROKER
```

下图展示了 Listener、安全协议、认证与鉴权之间的关系：

安全协议决定了服务端和客户端之间的认证（Authentication）协议，而鉴权（Authorization）基本独立于认证，只校验通过认证的主体对请求中涉及的资源是否有操作权限。下文将详细介绍认证方法和鉴权规则。

**02**

**认证**

**2.1 认证主体**

先简单介绍一下认证主体的概念。

认证主体是对 client 的身份标识，对应一个 KafkaPrincipal 对象[5]。当 client 通过认证协议完成认证后，broker 侧会将该 KafkaPrincipal 对象塞入 RequestContext，并向上传递，以供后续鉴权。一个 KafkaPrincipal 对象主要包括主体类型（目前仅有“User”这一类别）以及一个名称（可以简单理解为 client 声明的用户名）。

**2.2 认证协议**

安全协议到认证协议之间的映射为：

* PLAINTEXT：无认证；
* SSL：无认证/mTLS；
* SASL\_PLAINTEXT: SASL;
* SASL\_SSL: SASL;

其中 SSL 如果想要利用 mTLS 进行认证，需要在 broker 侧开启对 client 的 SSL 验证：

```
 ssl.client.auth=required
```

利用证书中的 DistinguishedName 字段 [4] 识别认证主体。由于认证通常涉及到用户的管理，这种通过证书进行认证的方式在增删用户时显得比较“笨重”（尤其是动态增删用户的场景），并不是主流的认证方式。

SASL 认证协议又可以进一步细分为以下认证机制：

* GSSAPI：借助第三方 Kerberos[3]（一种基于 ticket 的加密认证协议）服务器进行认证认证；
* PLAIN：简单账密认证，注意它跟前文的 PLAINTEXT 不是一个概念；
* SCRAM-SHA-256/512：基于 SCRAM算法，由 Kafka 节点基于 record 自认证；
* OAUTHBEARER：借助第三方 OAuth 服务器认证；

Kafka broker 允许同时启用多种 SASL 认证，例如同时启用 SCRAM-SHA-256/512 + PLAIN：

```
sasl.enabled.mechanisms=SCRAM-SHA-256,PLAIN,SCRAM-SHA-512  
```

Broker 侧还需要额外的 JAAS 配置，在《AutoMQ SASL 安全身份认证配置教程》中已经提及，这里不再赘述。

需要注意的是，Kafka 提供了默认的 SASL/PLAIN 实现，需要在每个节点配置中显示声明账密信息。这种静态认证方式同样不利于用户的动态管理，可以通过集成外部的账密认证服务器来提供动态能力。

**03**

**鉴权**

鉴权是基于认证主体，检查是否有权限操作请求的资源。

**3.1 鉴权配置**

重要的配置包括：

* authorizer.class.name：指定鉴权器，默认为空。Kraft 模式下可以填写官方提供的 “org.apache.kafka.metadata.authorizer.StandardAuthorizer”，基于 ACL（access control list）规则认证；
* super.users：设置超级用户，默认为空。格式为 User:{userName}。这里指定的用户将**不受 ACL 约束**，直接拥有**所有资源的所有操作权限**；
* allow.everyone.if.no.acl.found：指定资源没有任意 ACL 约束时的默认权限，默认为 false。false 表示仅允许 super user 操作，true 表示允许任意用户操作。

**3.2 鉴权规则**

Kafka 是基于 ACL（access control list） 规则限制用户对资源的访问的。一条 ACL 规则包含两部分：

* ResourcePattern：声明资源及其匹配方式，包含：

+ ResourceType：资源类型，包括 TOPIC、GROUP（消费者组）、CLUSTER 等；
+ ResourceName：资源名称；
+ PatternType：资源匹配方式，包括 LITERAL（全文匹配） 和 PREFIXED（前缀匹配）；

* AccessControlEntry：用户限制信息，包括：

+ Principal：认证主体，其实就是个用户名；
+ Host：用户主机地址；
+ AclOperation：操作行为，包括 Read、Write、CREATE、DELETE 等；
+ AclPermissionType：允许/禁止。

一句话串联上述规则信息就是：

允许/禁止 来自 {Host} 的用户 {Principal} 对匹配类型为 {PatternType} 、名称为 {ResourceName} 的资源进行 {AclOperation} 操作。

当同时满足：

* 存在某条 ACL 允许用户操作
* 不存在任何一条 ACL 禁止用户操作

时，允许用户对资源进行操作。

需要注意的是，可以将通配符“\*”作为 ResourceName 或 Host 的内容。对于 Host 字段，填写“\*” 表示任意地址。ResourceName 为 “\*”，且 PatternType 为 LITERAL 时表示匹配任意名称；如果 ResourceName 为 “\*”，PatternType 为 PREFIXED，表示匹配前缀名称为“\*”的资源。

此外，当 ResourceType 为 CLUSTER 时，ResourceName 只能填写**“kafka-cluster”**。

**04**

**升级认证协议或开启鉴权**

某些情况下我们需要考虑安全协议的切换，或者需要将集群从无鉴权转为开启鉴权。例如，在开发环境中，我们通常直接用默认的 PLAINTEXT 配置集群，如果不幸在测试或生产环境中也使用了 PLAINTEXT 协议，那么就需要考虑升级安全协议并开启鉴权。

以下以“PLAINTEXT、无鉴权”升级为“SASL\_PLAINTEXT、开启鉴权”为例，介绍如何平滑过渡。其他安全协议的变化可以以此类推。

以下假定三台 Kafka 节点都在本机部署，使用不同的端口以示区分。三台节点均为 controller + broker 的混部节点。注意，各个阶段变更时，需要一一重启各个节点。

整体逻辑示意图：

集群共需要三轮重启，业务侧需要一轮重启。

**4.1 原始配置**

以下为第一个节点在 PLAINTEXT 协议下的部分配置，其他节点配置依次类推：

```
node.id=1controller.quorum.voters=1@localhost:9093,2@localhost:9095,3@localhost:9097listeners=PLAINTEXT://:9092,CONTROLLER://:9093inter.broker.listener.name=PLAINTEXTadvertised.listeners=PLAINTEXT://localhost:9092controller.listener.names=CONTROLLERlistener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
```

**4.2 阶段一：新协议+默认 ALLOW 鉴权上线**

```
node.id=1# 保持不变controller.quorum.voters=1@localhost:9093,2@localhost:9095,3@localhost:9097# controller 和 broker 的新 listener 上线listeners=PLAINTEXT://:9092,CONTROLLER://:9093,BROKER_SASL://:9192,CONTROLLER_SASL://:9193# 保持不变inter.broker.listener.name=PLAINTEXT# 新地址上线advertised.listeners=PLAINTEXT://localhost:9092,BROKER_SASL://localhost:9192# 新 listener 上线controller.listener.names=CONTROLLER,CONTROLLER_SASL  
# authorizationauthorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer# 允许所有人访问资源allow.everyone.if.no.acl.found=true# 超级用户，用于节点之间认证super.users=User:automq  
sasl.enabled.mechanisms=SCRAM-SHA-256,PLAIN,SCRAM-SHA-512# 指定与 broker 通信时具体的 SASL 机制sasl.mechanism.inter.broker.protocol=PLAIN# 指定与 controller 通信时具体的 SASL 机制sasl.mechanism.controller.protocol=PLAIN# 静态账密配置listener.name.broker_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";  
listener.name.controller_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";  
# 认证模块配置listener.name.broker_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.broker_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;  
# 添加新 listener 到 安全协议的映射listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL,BROKER_SASL:SASL_PLAINTEXT,CONTROLLER_SASL:SASL_PLAINTEXT
```

其中“allow.everyone.if.no.acl.found”设置为 true，是为了让线上的 client 能够正常认证，避免直接鉴权失败。

“controller.listener.names”设置为“CONTROLLER，CONTROLLER\_SASL”，表示 controller 同时使用两个 listener，并且本 node 的 broker 将使用“CONTROLLER”映射的安全协议与 controller 通信。

本阶段结束以后，需要通知业务方重新配置基于 SASL\_PLAINTEXT 配置客户端。同时，需要为各业务方配置 ACL 所需的规则。

**4.3 阶段二：节点之间使用新协议+鉴权开启**

在上一阶段执行后，新协议下的 listener 已经上线，并可以对外提供服务，但是集群的节点之间依旧维持原有的通信协议，本阶段会将内部通信进行升级：

```
node.id=1# 使用新 controller 端口controller.quorum.voters=1@localhost:9193,2@localhost:9195,3@localhost:9197# 保持不变listeners=PLAINTEXT://:9092,CONTROLLER://:9093,BROKER_SASL://:9192,CONTROLLER_SASL://:9193# 使用新协议inter.broker.listener.name=BROKER_SASL# 使用新地址advertised.listeners=PLAINTEXT://localhost:9092,BROKER_SASL://localhost:9192  
# 注意顺序变化controller.listener.names=CONTROLLER_SASL,CONTROLLER  
# authorizationauthorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer# allow.everyone.if.no.acl.found=truesuper.users=User:automq  
# 保持不变sasl.enabled.mechanisms=SCRAM-SHA-256,PLAIN,SCRAM-SHA-512sasl.mechanism.inter.broker.protocol=PLAINsasl.mechanism.controller.protocol=PLAINlistener.name.broker_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";listener.name.controller_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";  
# 保持不变listener.name.broker_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.broker_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;  
# 保持不变listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL,BROKER_SASL:SASL_PLAINTEXT,CONTROLLER_SASL:SASL_PLAINTE
```

注意，“allow.everyone.if.no.acl.found”配置被注释掉了，也就是不再默认允许任何人任意操作资源了。

**4.4 阶段三：下线旧协议**

```
 node.id=1  
# 保持不变controller.quorum.voters=1@localhost:9193,2@localhost:9195,3@localhost:9197# 下线旧 listenerlisteners=BROKER_SASL://:9192,CONTROLLER_SASL://:9193  
# 保持不变inter.broker.listener.name=BROKER_SASL  
# 下线旧地址advertised.listeners=BROKER_SASL://localhost:9192  
# 下线旧 listenercontroller.listener.names=CONTROLLER_SASL  
# authorizationauthorizer.class.name=org.apache.kafka.metadata.authorizer.StandardAuthorizer# allow.everyone.if.no.acl.found=truesuper.users=User:automq  
# 保持不变sasl.enabled.mechanisms=SCRAM-SHA-256,PLAIN,SCRAM-SHA-512sasl.mechanism.inter.broker.protocol=PLAINsasl.mechanism.controller.protocol=PLAINlistener.name.broker_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";listener.name.controller_sasl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \   username="automq" \   password="automq-secret" \   user_automq="automq-secret";  
# 保持不变listener.name.broker_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.broker_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;listener.name.controller_sasl.scram-sha-512.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required;  
# 保持不变listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL,BROKER_SASL:SASL_PLAINTEXT,CONTROLLER_SASL:SASL_PLAINTEXT
```

至此，旧协议下线，所有通信基于 SASL\_PLAINTEXT 协议。

**05**

**总结**

本文概述了 Kafka 中的认证协议和鉴权策略。首先介绍了 listener 与安全协议的映射，以及安全协议与认证方法的映射。接着分别介绍 Kafka 中支持的多种认证协议，以及 ACL 鉴权策略。在认证通过后，Kafka 会生成一个认证主体，供上层进行细粒度的鉴权。最后介绍了如何对一个运行中的 Kafka 集群中进行认证协议升级以及开启鉴权。

**参考资料**

[1] https://www.automq.com/blog/automq-sasl-security-authentication-configuration-guide

[2] https://www.automq.com/blog/automq-ssl-security-protocol-configuration-tutorial

[3] https://developer.confluent.io/courses/security/authentication-basics/

[4] https://smallstep.com/hello-mtls/doc/server/kafka

[5] https://web.mit.edu/kerberos/

**往期推荐**

[一文搞懂 Kafka Producer（下）

2024-09-26](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486461&idx=1&sn=e283e9bc907c0498348b275f0a14da45&chksm=c1bc2db4f6cba4a21bff1798727fcbee186b39954b2af7ebe33b8de62261bdd02f8e5bd83336&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486461&idx=1&sn=e283e9bc907c0498348b275f0a14da45&chksm=c1bc2db4f6cba4a21bff1798727fcbee186b39954b2af7ebe33b8de62261bdd02f8e5bd83336&scene=21#wechat_redirect")

[一文搞懂 Kafka Producer（上）

2024-05-15](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485001&idx=1&sn=da56e06f2aa294ac9bc8cd6bd6b6033a&chksm=c1bc2000f6cba916bb96aede7847325b06172fe21a8331b4b85080f2e4f627ed294d7f39f66b&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247485001&idx=1&sn=da56e06f2aa294ac9bc8cd6bd6b6033a&chksm=c1bc2000f6cba916bb96aede7847325b06172fe21a8331b4b85080f2e4f627ed294d7f39f66b&scene=21#wechat_redirect")

[系统稳定性的基石：

限流在 AutoMQ 中的最佳实践

2024-09-05](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486292&idx=1&sn=3521bb1fd16e16d39ca2ef017f4cc85b&chksm=c1bc2d1df6cba40b277f218ddf63a65a326f66b7fc12a80783a065ae128b766369c9e5ccb6d7&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486292&idx=1&sn=3521bb1fd16e16d39ca2ef017f4cc85b&chksm=c1bc2d1df6cba40b277f218ddf63a65a326f66b7fc12a80783a065ae128b766369c9e5ccb6d7&scene=21#wechat_redirect")

[使用 AutoMQ 和 Tinybird

分析用户网购行为

2024-09-02](https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486243&idx=1&sn=2956592ed8d0efe369ca099ee3c54894&chksm=c1bc2d6af6cba47cf64d6a617422e3208c74d64a2aebd2fe5a5195745435dff7cbd794585ba8&scene=21#wechat_redirect "https://mp.weixin.qq.com/s?__biz=MzkxNzY0ODE2Ng==&mid=2247486243&idx=1&sn=2956592ed8d0efe369ca099ee3c54894&chksm=c1bc2d6af6cba47cf64d6a617422e3208c74d64a2aebd2fe5a5195745435dff7cbd794585ba8&scene=21#wechat_redirect")

END

**关于我们**

我们是来自 Apache RocketMQ 和 Linux LVS 项目的核心团队，曾经见证并应对过消息队列基础设施在大型互联网公司和云计算公司的挑战。现在我们基于对象存储优先、存算分离、多云原生等技术理念，重新设计并实现了 Apache Kafka 和 Apache RocketMQ，带来高达 10 倍的成本优势和百倍的弹性效率提升。

🌟 GitHub 地址：https://github.com/AutoMQ/automq

💻 官网：https://www.automq.com

👀 B站：AutoMQ官方账号

🔍 视频号：AutoMQ

👉🏻 **扫二维码**

**加入我们的社区群**

关注我们，一起学习更多云原生技术干货！

👇🏻点击下方阅读原文，前往 GitHub 了解体验！
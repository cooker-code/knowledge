---
title: Kafka配置SASL_PLAINTEXT安全认证协议详解
author: 石臻说AI
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg4ODY1NTcxNg==&mid=2247495274&idx=1&sn=05dfdf27580af6e0bf644e696551a72f&chksm=cff57461f882fd77e2e9764dd06ccd866418513cfeeb15d67436ff8980115be21e139a776a1f&mpshare=1&scene=24&srcid=0906UjfD9emds6yHgTSfoN5p&sharer_shareinfo=3560054da98fd8d0851273eb90d8cdc5&sharer_shareinfo_first=a15cd839ec18bcae2f70b3fa7a4485fa#rd
---

* 什么是JAAS

+ JAAS认证的文件格式
+ Kafka JAAS认证文件与用户之间的关系

* Kafka配置JAAS的几种方式

+ Kafka Broker的 JAAS 配置
+ Kafka Client 的 JAAS 配置方式

* 使用 SASL 进行身份验证

+ SASL/PLAIN 机制的使用方法
+ SASL/SCRAM 机制的使用方法
+ SASL/GSSAPI(Kerberos) 机制的使用方法
+ SASL/OAUTHBEARER 机制使用方法

* 同时配置多个机制
* 在正在运行的集群中修改 SASL 机制
* 从 Broker 到 ZooKeeper 的连接身份验证

在 0.9.0.0 版中，Kafka 社区添加了许多功能，这些功能可以单独使用或一起使用，以提高 Kafka 集群的安全性。目前支持安全措施有很多, 其中就有  ：**使用 SSL 或 SASL 对来自客户端（生产者和消费者）、其他Broker和工具对Broker连接进行身份验证**。

其中还有一些概念您需要了解一下，就是安全协议，关于安全协议详情可以看一文搞懂Kafka中的listeners和advertised.listeners以及其他通信配置

其中安全协议有以下几个, 都很明确

1. PLAINTEXT => PLAINTEXT 不需要授权,非加密通道
2. SSL => SSL 使用SSL加密通道
3. SASL\_PLAINTEXT => SASL\_PLAINTEXT 使用SASL认证非加密通道
4. SASL\_SSL => SASL\_SSL 使用SASL认证并且SSL加密通道

SASL\_PLAINTEXT和SASL\_SSL的区别就是数据的传输是不是启动了加密通道, 但是本质上都是适用的SASL认证。

其中SASL认证又分为多种机制, Kafka 支持以下 SASL 机制：

| 机制 | Kafka版本 | 特点 |
| --- | --- | --- |
| SASL/GSSAPI (Kerberos) | 0.9.0.0 | 需要独立部署验证服务 |
| SASL/PLAIN | 0.10.0.0 | 不能动态增加用户 |
| SASL/SCRAM-SHA-256 和 SASL/SCRAM-SHA-512 | 0.10.2.0 | 可以动态增加用户 |
| SASL/OAUTHBEARER | 2.0 | 需自己实现接口实现token的创建和验证，需要额外Oauth服务 |

今天我们就主要来讲解一下这些SASL的机制和操作。

## 什么是JAAS

在介绍SASL之前，我们先简单了解一下JAAS(Java 身份验证和授权服务), 因为Kafka使用的JAAS来为SASL进行的配置

### JAAS认证的文件格式

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin"  
        password="admin@password"  
        user_admin="admin@password"  
        user_szz="szz@123";  
};  
  
KafkaClient {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin_sasl_plaintext_username"  
        password="admin_sasl_plaintext_password";  
  
};
```

上面的格式是JAAS的比较通用的格式

1. **KafkaServer**：这是 LoginContext 的名称，用于在 JAAS 登录配置文件中查找此应用程序的条目，可以理解为一个索引。既然是一个索引，说明JAAS文件是支持配置多个模块的，比如上面的**KafkaClient**模块
2. `org.apache.kafka.common.security.plain.PlainLoginModule` :是一个登录模块实现类,是一个SPI，都需要实现`javax.security.auth.spi.LoginModule`
3. `required` 是一个ControlFlag，它的可选项有[required、requisite、sufficient、optional]，这里我们设置为 required；
4. 后面的是：可选参数，这些可选参数都会最终当做入参传给LoginModule. 多个用空格隔开

### Kafka JAAS认证文件与用户之间的关系

JAAS认证文件的后面参数都是可选参数, 你可以随意传入参数, 但是最终你是需要正确认证的时候是需要对应模块解析好对应的数据的。

比如看看下面几种机制中，他们的JAAS后面的参数都是不一样的。

#### SASL安全协议下的PLAINT机制

比如说在**SASL\_PLAINTEXT** 的安全协议的**PLAINT**机制下, 后面的可选参数一般都是用户名和密码；看下上面的KafkaServer配置例子中：

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin"  
        password="admin@password"  
        user_admin="admin@password"  
        user_szz="szz@123";  
};
```

1. **username**和**password**是 broker 用来启动与其他 broker 的连接(跟其他Broker发起请求需要带上)；
2. `user_`前缀是一个用户集合, 表示如果有其他用户对我发起请求的时候，检查一下是否在我的用户集里面并且密码是正确的
   上面定义了2个用户(admin 和 szz)，
3. 一般情况下，必须在用户集合里面配置一下该Broker的用户和密码，因为当你这个Broker当选为Controller角色的时候,你会向自己发起请求，你发起请求的时候带上了**username**和**password**，如果你的用户集里面没有这个用户，那么就会认证失败；

假设你2台Broker，你非要让他们**username**和**password**都不一样；那么你就应该让他们都陪着到他们自己的用户集里面去；如下

**Broker-0**

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="kafka1"  
        password="kafka1@password"  
        user_kafka1="kafka1@password"  
        user_kafka2="kafka2@password";  
};
```

**Broker-1**

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="kafka2"  
        password="kafka2@password"  
        user_kafka2="kafka2@password"  
        user_kafka1="kafka1@password";  
};
```

当然一般情况下你可能还会设置客户端的账户, 比如说你生产者客户端配置了如下用户

**producer配置**

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="kafka_producer" password="kafka@123";
```

然后只在Broker-0的用户集里面新增了这个用户密码；那么当消息发往Broker-1的时候，就会发生认证失败的异常;

更详细的介绍请看： Java 身份验证和授权服务JAAS

#### SASL\_PLAINTEXT安全协议下的GSSAPI机制

又比如说在**SASL\_PLAINTEXT** 的安全协议的**GSSAPI** 机制下，他的配置有可能是这样的

```
KafkaServer {  
    com.sun.security.auth.module.Krb5LoginModule required  
    useKeyTab=true  
    storeKey=true  
    keyTab="/etc/security/keytabs/kafka_server.keytab"  
    principal="kafka/kafka1.hostname.com@EXAMPLE.COM";  
};  
  
// Zookeeper client authentication  
Client {  
    com.sun.security.auth.module.Krb5LoginModule required  
    useKeyTab=true  
    storeKey=true  
    keyTab="/etc/security/keytabs/kafka_server.keytab"  
    principal="kafka/kafka1.hostname.com@EXAMPLE.COM";  
};
```

具体的配置详解 TODO了解Kerberos.......

#### SASL\_PLAINTEXT安全协议下的OAUTHBEARER机制

Salted Challenge Response Authentication Mechanism (SCRAM) 是 SASL 机制的一个家族，它解决了执行用户名/密码身份验证的传统机制（如 PLAIN 和 DIGEST-MD5）的安全问题。

Kafka 支持SCRAM-SHA-256和 SCRAM-SHA-512，它们可以与 TLS 一起使用以执行安全身份验证。用户名用作 Principal配置 ACL 等的身份验证。Kafka 中的默认 SCRAM 实现将 SCRAM 凭据存储在 Zookeeper 中，适用于 Zookeeper 在专用网络上的 Kafka 安装。

具体的配置详解 TODO 待补充.......

#### SASL\_PLAINTEXT安全协议下的SCRAM机制

OAuth 2 授权框架“允许第三方应用程序获得对 HTTP 服务的有限访问权限，或者代表资源所有者通过编排资源所有者和 HTTP 服务之间的批准交互，或者通过允许第三方应用程序代表自己获得访问权限。

具体的配置详解 TODO 待补充.......

## Kafka配置JAAS的几种方式

### Kafka Broker的 JAAS 配置

在Broker配置JAAS配置有2种方式如下

#### 方式一：Broker配置属性listener.name.{listenerName}.{saslMechanism}.sasl.jaas.config

我们可以通过在`server.properties`中来配置JAAS信息，可以给不同的监听器和不同的SASL机制配置验证信息。

例如：

```
#格式 listener.name.{对应的小写监听器名称}.{sasl.enabled.mechanisms使用的SASL机制小写}.sasl.jaas.config  
  
listener.name.sasl_plaintext.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="kafka2" password="kafka2" user_kafka2=kafka2 ;
```

上面的`listener.name.sasl_plaintext.plain.sasl.jaas.config`配置并不是固定的，而且需要自己根据实际情况配置的。
他的格式为：

`listener.name.{对应的小写监听器名称}.{sasl.enabled.mechanisms使用的SASL机制小写}.sasl.jaas.config`

看到这个格式，你应该能够了解到，他可以根据不同的监听器 和不同的SASL机制来定制化的配置JAAS属性。
例如我的监听器使用的是 **SASL\_PLAINTEXT**，那么它对应的小写`sasl_plaintext` ; 然后使用哪个SASL机制呢？

配置`sasl.enabled.mechanisms` 是让我们来确定启动哪些机制的，可以是一个列表
比如：`sasl.enabled.mechanisms=GSSAPI,PLAIN,SCRAM-SHA-256,SCRAM-SHA-512,OAUTHBEARER`

默认是GSSAPI，那我们这里使用PLAIN，所以最终配置是`listener.name.sasl_plaintext.plain.sasl.jaas.config=`

那么它的值就是上面我们讲解过的**JAAS认证的文件格式** 与 **Kafka JAAS认证文件用户之间的关系**

当然，这种方式只能指定一个LoginModle， 如果在监听器上配置了多个机制，则必须使用监听器和机制前缀为每个机制提供配置。

例如：

```
listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \  
    username="admin" \  
    password="admin-secret";  
listener.name.sasl_ssl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \  
    username="admin" \  
    password="admin-secret" \  
    user_admin="admin-secret" \  
    user_alice="alice-secret";
```

#### 方式二：JAAS静态配置文件

**KafkaServer**：是每个 Broker 使用的 **JAAS** 文件中的默认 module name( LoginContext 的名称)。

我们可以通过`kafka_server_jaas.conf`文件的方式来配置JAAS属性，这也是大部分情况最常用的方式。

①. 配置`kafka_server_jaas.conf`文件；如下

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin"  
        password="admin"  
        user_admin="admin";  
};  
  
sasl_plaintext.KafkaServer {  
  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="kafka1"  
        password="password1"  
        user_kafka1="password1";  
};
```

这上面是配置了两个 module name ； **KafkaServer**是默认情况Broker使用的配置，但是假如你想给指定的监听器配置认证信息

那就如`sasl_plaintext.KafkaServer`模块所示，那么只要是这个`SASL_PLAINTEXT` 监听器发起/监听的请求，认证信息就会用它自己的模块, 它的优先级是高于**默认的KafkaServer模块**的。

这个格式是：`{监听器名称小写}.KafkaServer` ; 你也可以给每一个监听器来配置此值。

②. 配置了配置文件之后，你需要让JAAS模块能够读取到这个配置文件。你需要设置一个系统属性`java.security.auth.login.config=文件路径`；

比如启动的时候通过`-D`传入系统属性

```
- Djava.security.auth.login.config=/Users/szz/work/IdeaPj/open_source/kafka/config/kafka_server_jaas.conf
```

当然，你也可以直接修改kafka的启动脚本，加入这个属性。

**上面的2种启动方式，如果同时配置了**

他们也是有优先级之分的，使用的优先顺序为：

* Broker配置属性listener.name.{listenerName}.{saslMechanism}.sasl.jaas.config
* {listenerName}.KafkaServer静态 JAAS 配置部分
* KafkaServer静态 JAAS 配置部分

### Kafka Client 的 JAAS 配置方式

客户端可以使用客户端配置属性 `sasl.jaas.config` 或使用 类似于Broker的静态 JAAS 配置文件来配置 JAAS。

#### 方式一：使用 `sasl.jaas.config` 配置JAAS

在 `producer.properties` 或 `consumer.properties` 中为每个客户端配置 JAAS 配置属性。

LoginModule描述了生产者和消费者等客户端如何连接到 Kafka Broker， 以下是 PLAIN 机制(不同机制的LoginModule不一样)的客户端配置示例：

```
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \  
    username="szz" \  
    password="szz-123";
```

JVM中的不同客户端都可以通过在 `sasl.jaas.config`属性中配置属于自己的JAAS配置信息, 比如一个JVM中启动了多个客户端。

如果是用下面的**方式二**,则达不到这样的效果。只会都使用一个

#### 方式二：使用静态配置文件的 JAAS 配置

跟Broker的形式差不多，也是创建一个静态的配置文件,例如文件名：`kafka_client_jaas.conf`

①. 添加JAAS配置文件，其中包含名为 `KafkaClient`  LoginModule 的客户端登录部分( Broker的是 KafkaServer)。
例如：

```
   
KafkaClient {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="kafka"  
        password="kafka";  
  
};
```

②. 配置了配置文件之后，你需要让JAAS模块能够读取到这个配置文件。你需要设置一个系统属性`java.security.auth.login.config=文件路径`；

比如启动的时候通过`-D`传入系统属性

```
- Djava.security.auth.login.config=/Users/szz/work/IdeaPj/open_source/kafka/config/kafka_server_jaas.conf
```

当然，你也可以直接修改kafka的启动脚本，加入这个属性。

使用这种方式 仅允许JVM中的所有客户端使用同一个配置

## 使用 SASL 进行身份验证

### SASL/PLAIN 机制的使用方法

#### Broker SASL配置

**①. 配置Broker 的 `server.properties`**

```
# 监听器列表  
listeners=SASL_PLAINTEXT://host.name:port  
  
# Broker之间内部通信使用的监听器  
inter.broker.listener.name=SASL_PLAINTEXT  
  
#Kafka 服务器中启用的 SASL 机制列表。该列表可能包含安全提供者可用的任何机制。默认情况下仅启用 GSSAPI。  
# 可选GSSAPI,PLAIN,SCRAM-SHA-256,SCRAM-SHA-512,OAUTHBEARER  
sasl.enabled.mechanisms=PLAIN  
  
# SASL 机制用于Broker之间通信。默认为 GSSAPI。注意：sasl.enabled.mechanisms 中的列表必须包含下面的配置  
sasl.mechanism.inter.broker.protocol=PLAIN
```

如果您希望Broker之间使用SASL互相验证，请确保为Broker设置相同的SASL协议

也就是`inter.broker.listener.name`属性,或者说是`security.inter.broker.protocol` ;

关于配置`listeners`、`inter.broker.listener.name`、`security.inter.broker.protocol` 相关配置可以查看文章：
一文搞懂Kafka中的listeners和advertised.listeners以及其他通信配置

**②. 配置Broker 的 JAAS配置**

该配置有多种方式, 更详细的配置请查看上面的 **Kafka Broker的 JAAS 配置** 部分;

这里我们以静态JAAS文件方式为例

新建配置文件 `/etc/kafka/kafka_server_jaas.conf`

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin"  
        password="admin"  
        user_admin="admin"  
        user_szz="szz@123"  
        user_szz2="szz2@123";  
};
```

关于配置文件具体意思，请看上面的：**Kafka JAAS认证文件与用户之间的关系**

③. 设置系统属性`java.security.auth.login.config`, 告知JAAS的配置文件路径

```
-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
```

当然你也可以直接将这个属性在Kafka的启动脚本kafka-server-start.sh里面加上

```
export KAFKA_OPTS="-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
```

#### 配置 Kafka 客户端

客户端配置JAAS 也是有2种方式, 更详细的说明请看上面的：**Kafka Client 的 JAAS 配置方式**

这里以配置`sasl.jaas.config`的方式为例

`producer.properties` 或`consumer.properties`

```
# 客户端 用于与 Broker 通信的协议。有效值为：PLAINTEXT、SSL、SASL_PLAINTEXT、SASL_SSL。   
security.protocol=SASL_PLAINTEXT  
  
# 客户端使用的SASL机制，默认情况是 GSSAPI ；可选项有 GSSAPI,PLAIN,SCRAM-SHA-256,SCRAM-SHA-512,OAUTHBEARER  
sasl.mechanism=PLAIN  
  
# 配置jaas相关信息  
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \  
    username="szz" \  
    password="szz-123";
```

客户端使用 `username`、`password` 来向Broker发起请求, Broker在验证的时候, 会判断传入的 `username`、`password` 是否在自己的用户集里面，也就是以`user_`作为前缀的账号集。

#### 通过CLI如何配置客户端的JAAS信息

假设我想通过CLI工具 `kafka-console-producer.sh` 、`kafka-console-consumer.sh` 来实现生产和消费，那么如何配置上面客户端所描述的信息呢？

这些对应的工具都有让您指向配置文件的配置，只需要在配置文件里面设置好就行了,例如：

```
#比如--producer.config 指向配置文件  
sh kafka-console-producer.sh  其他参数省略   --producer.config config/producer.properties
```

#### 生产环境使用SASL/PLAIN 机制的注意事项

##### 1. SASL/PLAIN 应仅与 SSL 作为传输层一起使用

SASL/PLAIN 应仅与 SSL 作为传输层一起使用，以确保明文密码不会在未经加密的情况下在线传输。SASL\_PLAINTEXT 作为明文形式的传输不太安全；如何使用SASL\_SSL/PLAIN?TODO...

##### 2. 自定义验证逻辑 ( 密码加密/外部存储)

kafka 中 SASL/PLAIN 的默认实现在 JAAS 配置文件中指定用户名和密码，这样可能并不是很安全

`sasl.server.callback.handler.class` 从 Kafka 2.0 版开始，您可以通过配置自己的回调处理程序来避免在磁盘上存储明文密码，这些回调处理程序使用配置选项还有从外部源获取用户名和密码。

**在Server端验证的时候**

---

PlainSaslServer 在验证请求的时候是在 `PlainSaslServer#evaluateResponse`做的处理

```
 @Override  
    public byte[] evaluateResponse(byte[] responseBytes) throws SaslAuthenticationException {  
       //部分省略...  
        String password = tokens.get(2);// 客户端发过来的密码  
  
        if (username.isEmpty()) {  
            throw new SaslAuthenticationException("Authentication failed: username not specified");  
        }  
        if (password.isEmpty()) {  
            throw new SaslAuthenticationException("Authentication failed: password not specified");  
        }  
  
        NameCallback nameCallback = new NameCallback("username", username);  
        PlainAuthenticateCallback authenticateCallback = new PlainAuthenticateCallback(password.toCharArray());  
        try {  
         //这里的callback进行处理，做验证的逻辑  
            callbackHandler.handle(new Callback[]{nameCallback, authenticateCallback});  
        } catch (Throwable e) {  
            throw new SaslAuthenticationException("Authentication failed: credentials for user could not be verified", e);  
        }  
        if (!authenticateCallback.authenticated())  
            throw new SaslAuthenticationException("Authentication failed: Invalid username or password");  
       //部分省略...  
    }  
    
```

通过上面的代码可以了解到，`callbackHandler.handle`是做验证的逻辑接口，如果验证失败,则给authenticateCallback的authenticated熟悉赋值false, 如果我们想要自定义的话, 那么只需要修改这个值就行了。

在PLAIN机制的前提下, 这个CallbackHandler的默认实现类是`org.apache.kafka.common.security.plain.internals.PlainServerCallbackHandler`

那我们可以自定义这个实现类, 并且只需要修改一下

###### 1. server.properties 增加属性配置

```
#格式为：listener.name.{小写的安全协议}.{小写的SASL机制}.sasl.server.callback.handler.class  
# 值为全调用链路径  
listener.name.sasl_plaintext.plain.sasl.server.callback.handler.class=org.apache.kafka.common.security.plain.internals.SzzPlainServerCallbackHandler
```

上面的`listener.name.sasl_plaintext.plain.sasl.server.callback.handler.class`不是固定属性，他的格式是：

```
listener.name.{小写的安全协议}.{小写的SASL机制}.sasl.server.callback.handler.class
```

也就是说，你可以给不同的安全协议和不同的SASL机制定制不同的**CallbackHandler**

###### 2. 写自定义CallbackHandler类

这里我写个Demo： org.apache.kafka.common.security.plain.internals.SzzPlainServerCallbackHandler

实现接口：org.apache.kafka.common.security.auth.AuthenticateCallbackHandler

```
/**  
 *  石臻臻自定义 SASL/PLAIN机制 回调类  
 *  
 *  在这个类里面，自定义验证逻辑  
 */  
public class SzzPlainServerCallbackHandler implements AuthenticateCallbackHandler {  
  
  
    private static final String JAAS_USER_PREFIX = "user_";  
    // JAAS配置文件的属性值  
    private List<AppConfigurationEntry> jaasConfigEntries;  
  
    @Override  
    public void configure(Map<String, ?> configs, String mechanism, List<AppConfigurationEntry> jaasConfigEntries) {  
        this.jaasConfigEntries = jaasConfigEntries;  
    }  
  
    @Override  
    public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {  
        String username = null;  
        for (Callback callback: callbacks) {  
            if (callback instanceof NameCallback)  
                username = ((NameCallback) callback).getDefaultName();  
            else if (callback instanceof PlainAuthenticateCallback) {  
                PlainAuthenticateCallback plainCallback = (PlainAuthenticateCallback) callback;  
                boolean authenticated = authenticate(username, plainCallback.password());  
                //设置是否认证成功，如果是false，会返回失败  
                plainCallback.authenticated(authenticated);  
            } else  
                throw new UnsupportedCallbackException(callback);  
        }  
    }  
  
    /**  
     * 验证逻辑； 在这里面 我们可以自定义验证逻辑，比如我们可以将 JAAS配置文件放到外部的配置中,并且还可以顺便加个密  
     * @param username  
     * @param password  
     * @return  
     * @throws IOException  
     */  
    protected boolean authenticate(String username, char[] password) throws IOException {  
        if (username == null)  
            return false;  
        else {  
  
            String expectedPassword = expectedPassword(JAAS_USER_PREFIX +  
                    username,PlainLoginModule.class.getName());  
  
            return expectedPassword != null && Utils.isEqualConstantTime(password, expectedPassword.toCharArray());  
        }  
    }  
  
    public String expectedPassword(String username,String loginModuleName){  
  
        // 根据传入的username,去外部查询该Broker的用户集, 并检查密码是否正确  
        // 这个外部可以是数据库，可以是zk，也可以是动态配置，当然你在存储的时候还可以给他们加个密  
        // 然后再这里解密之后再返回 等等  
        return "";  
    }  
  
    @Override  
    public void close() throws KafkaException {  
    }  
  
}
```

**在Client端验证的时候**

---

在发起请求之前,会先构建**SaslClient**, 这个客户端有很多的实现类

在这里插入图片描述

如果是PLAIN机制的情况下，他的实现类是 **PlainClient** , 这个PlainClient就保存着 我们发起请求时候的 `username、password`

获取这两个信息是在   ClientFactoryImpl#createSaslClient ==》 ClientFactoryImpl#getUserInfo

所以最终获取用户密码信息的地方是 `SaslClientCallbackHandler`这个类, 可以看看它的方法

```
@Override  
    public void handle(Callback[] callbacks) throws UnsupportedCallbackException {  
        Subject subject = Subject.getSubject(AccessController.getContext());  
        for (Callback callback : callbacks) {  
            if (callback instanceof NameCallback) {  
                NameCallback nc = (NameCallback) callback;  
                if (subject != null && !subject.getPublicCredentials(String.class).isEmpty()) {  
                 // 设置username  
                    nc.setName(subject.getPublicCredentials(String.class).iterator().next());  
                } else  
                 // 设置username  
                    nc.setName(nc.getDefaultName());  
            } else if (callback instanceof PasswordCallback) {  
                if (subject != null && !subject.getPrivateCredentials(String.class).isEmpty()) {  
                 //设置password  
                    char[] password = subject.getPrivateCredentials(String.class).iterator().next().toCharArray();  
                    ((PasswordCallback) callback).setPassword(password);  
                } else {  
                    String errorMessage = "Could not login: the client is being asked for a password, but the Kafka" +  
                             " client code does not currently support obtaining a password from the user.";  
                    throw new UnsupportedCallbackException(callback, errorMessage);  
                }  
            } else if (callback instanceof RealmCallback) {  
              //省略其他  
            }  
        }  
    }
```

那么我们如果想要自定义的话，可以自己写一个AuthenticateCallbackHandler, 然后在方法里面做一些处理,比如说从外部配置里面读取username、password，照猫画虎就行了。

当然自己写了AuthenticateCallbackHandler之后不要忘记设置客户端属性

```
sasl.client.callback.handler.class=自定义的AuthenticateCallbackHandler全路径类名
```

### SASL/SCRAM 机制的使用方法

Salted Challenge Response Authentication Mechanism (SCRAM) 是 SASL 机制的一个家族，它解决了执行用户名/密码身份验证的传统机制（如 PLAIN 和 DIGEST-MD5）的安全问题。该机制在RFC 5802中定义。

Kafka 支持`SCRAM-SHA-256`和 `SCRAM-SHA-512`，它们可以与 TLS 一起使用以执行安全身份验证。用户名用作 Principal配置 ACL 等的身份验证。Kafka 中的默认 SCRAM 实现将 SCRAM 凭据存储在 Zookeeper 中，适用于 Zookeeper 在专用网络上的 Kafka 安装。

**SCRAM机制是在0.10.2.0版本中加入的, 她跟PLAIN相比的话，可以动态的增加用户,并且还是加密数据。**

对于PLAIN 来说, 每个Broker都会有一个`user_`开头的用户集, 这个用户集保存着所有的可能的用户集合, 那么有没有办法把这个用户集合给存储到外部，并且还能够支持动态的配置呢？

既然说到动态配置，那我们知道kafka本身就是支持动态配置的，动态配置的信息是存在zookeeper里面。
关于Kafka的动态配置可以看：Kafka的动态配置原理详解

关于SCRAM机制的源码解析请看：Kafka 安全机制SCRAM源码解析

#### 先配置用户列表

第一步需要现在你的Zookeeper去配置一下用户列表，这个用户列表就是用来匹配请求方的凭据的。

建议先配置好用户列表, 因为不配置这个数据的话，就算启动Broker，那么Broker也会一直提示认证失败的异常。

**新增/修改用户**

```
bin/kafka-configs.sh --zookeeper xxxx:xxxx --alter --add-config SCRAM-SHA-256=[iterations=8192,password=szz@123],SCRAM-SHA-512=[password=szz@123] --entity-type users --entity-name szz
```

上面的 iterations=8192 表示的是迭代次数，默认情况是 4096 ，更高的迭代次数可以提高暴力破解的难度。

**Kafka 仅支持强哈希函数 SHA-256 和 SHA-512，最小迭代次数为 4096。如果 Zookeeper 安全受到威胁，强哈希函数与强密码和高迭代次数相结合可以防止暴力攻击。**

创建 SCRAM 证书时如果出现:

```
requirement failed: Unknown Dynamic Configuration: Set('SCRAM-SHA-256). 报错
```

则可能是你--add-config 有单引号，去掉就行(因为kafka官网用了单引号)

新增用户之后，你可以看到zookeeper里面节点 config/users/{用户名} 的信息如下

```
{  
  "version" : 1,  
  "config" : {  
    "SCRAM-SHA-512" : "salt=MWt1bGYwOTFxcG5jcGp5aXp5cWxjeTV2YW0=,stored_key=oT0W7yatEOHmdbwxA7d3zGJujmd3GhFYsvPjq/nVCERLdlMnu49BEy17qUoYdPWG6tPsf1xTGvBixHSW6txcKA==,server_key=sdfLLSoPltVR3osD537tZ+4tGsYsoaflZqb5c7NYG3ofNSx9hKWwWPq5YJ9ThDEPDjy5k/ACVuH+RjtaQ5c1+A==,iterations=4096",  
    "SCRAM-SHA-256" : "salt=M2pwOGllbDZvYTBxZDUxaWs1Z2hrZWN3cw==,stored_key=arO74vje2Q8R/tV/PZxGk6tHuherneiDsvNm7ZEsQDo=,server_key=9zm9iQNXDvB9HMsqIJlmDcyomt2eeSuaioxbt4f2Zps=,iterations=8192"  
  }  
}
```

它的数据格式如下

**查看用户**

```
bin/kafka-configs.sh --zookeeper xxxx.xxxx --describe --entity-type users --entity-name szz
```

执行完可以看到打印出来的数据

```
Configs for user-principal 'szz' are SCRAM-SHA-512=salt=MWt1bGYwOTFxcG5jcGp5aXp5cWxjeTV2YW0=,stored_key=oT0W7yatEOHmdbwxA7d3zGJujmd3GhFYsvPjq/nVCERLdlMnu49BEy17qUoYdPWG6tPsf1xTGvBixHSW6txcKA==,server_key=sdfLLSoPltVR3osD537tZ+4tGsYsoaflZqb5c7NYG3ofNSx9hKWwWPq5YJ9ThDEPDjy5k/ACVuH+RjtaQ5c1+A==,iterations=4096,SCRAM-SHA-256=salt=M2pwOGllbDZvYTBxZDUxaWs1Z2hrZWN3cw==,stored_key=arO74vje2Q8R/tV/PZxGk6tHuherneiDsvNm7ZEsQDo=,server_key=9zm9iQNXDvB9HMsqIJlmDcyomt2eeSuaioxbt4f2Zps=,iterations=8192
```

**删除用户**

```
bin/kafka-configs.sh --zookeeper xxxx.xxxx --alter --delete-config SCRAM-SHA-512 --delete-config SCRAM-SHA-256 --entity-type users --entity-name szz
```

#### 配置Broker配置

配置好了用户列表之后，在Broker上配置SASL和 JAAS相关配置

server.properties

```
listeners=SASL_PLAINTEXT://localhost:9091  
  
# broker内部通信使用的安全协议  
inter.broker.listener.name=SASL_PLAINTEXT  
  
#Kafka 服务器中启用的 SASL 机制列表。该列表可能包含安全提供者可用的任何机制。默认情况下仅启用 GSSAPI。  
# 可选GSSAPI,PLAIN,SCRAM-SHA-256,SCRAM-SHA-512,OAUTHBEARER  
sasl.enabled.mechanisms=SCRAM-SHA-256,SCRAM-SHA-512  
  
# SASL 机制用于代理间通信。默认为 GSSAPI。注意：sasl.enabled.mechanisms 中的列表必须包含下面的配置  
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
```

然后配置JAAS, JAAS的配置跟上面的一样，2种方式都可以，具体可以看上面的 **Kafka Broker的 JAAS 配置** 部分；
只不过JAAS内容稍微有点不一样。

```
KafkaServer {  
  
        org.apache.kafka.common.security.scram.ScramLoginModule  required  
        username="szz"  
        password="szz@123";  
  
};
```

这里面的LoginModule是 **ScramLoginModule**，并且只配置了 `username`、`password` ; 跟PLAIN的区别是已经不需要配置 `user_`开头的用户列表了，因为这一部分已经放到zookeeper里面存储了。

#### Client 配置

```
#安全协议  
security.protocol=SASL_PLAINTEXT  
# SASL 机制  
sasl.mechanism=SCRAM-SHA-256 (or SCRAM-SHA-512)  
# jaas相关配置信息  
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="szz" password="szz@123";
```

注意Client的安全协议和机制 需要时Broker支持的，也就是Broker的`sasl.enabled.mechanisms`这个配置列表里面有。

#### 生产环境使用SCRAM的注意事项

1. Kafka 中 SASL/SCRAM 的默认实现将 SCRAM 凭证存储在 Zookeeper 中。这适用于 Zookeeper 安全且位于专用网络中的生产使用。
2. Kafka 仅支持强哈希函数 SHA-256 和 SHA-512，最小迭代次数为 4096。如果 Zookeeper 安全受到威胁，强哈希函数与强密码和高迭代次数相结合可以防止暴力攻击。
3. SCRAM 应仅与 TLS 加密一起使用，以防止拦截 SCRAM 交换。如果 Zookeeper 受到威胁，这可以防止字典或暴力攻击以及假冒。
4. `sasl.server.callback.handler.class` 从 Kafka 2.0 版开始，使用自定义回调处理程序覆盖默认的 SASL/SCRAM 凭证存储。假设你的Zookeeper处于不安全的环境中使用，那么你可以自定义回调处理,并将凭证另外存储。具体如何操作，请看上面的**自定义验证逻辑**部分。

### SASL/GSSAPI(Kerberos) 机制的使用方法

使用 **GSS-API**，程序员在编写应用程序时，可以应用通用的安全机制。开发者不必针对任何特定的平台、安全机制、保护类型或传输协议来定制安全实现。使用 GSS-API，程序员可忽略保护网络数据方面的细节。使用 GSS-API 编写的程序在网络安全方面具有更高的可移植性。这种可移植性是通用安全服务 API 的一个特点。

GSS-API 是一个以通用方式为调用方提供安全服务的框架。许多底层机制和技术（如 **Kerberos v5** 或公钥技术）都支持 GSS-API 框架。

关于GSSAPI的介绍请看:GSS-API 介绍

在这里我们主要讲解使用Kerberos安全机制, 我们可以简单理解为 GSS-API是一套规范接口, 而Kerberos是其中的一个实现。

#### Kerberos安装

在Kafka中使用Kerberos，当然前提是已经安装了Kerberos Server, 如果没有安装部署的话则还需要提前安装部署, 一般情况下, 你所在的组织应该已经部署好了，关于这一块就不详细赘述。

**TODO....**

### SASL/OAUTHBEARER 机制使用方法

**TODO....**

## 同时配置多个机制

如果要同时配置多个机制, 只需要在KafkaServer里面配置多个就行

```
KafkaServer {  
    com.sun.security.auth.module.Krb5LoginModule required  
    useKeyTab=true  
    storeKey=true  
    keyTab="/etc/security/keytabs/kafka_server.keytab"  
    principal="kafka/kafka1.hostname.com@EXAMPLE.COM";  
  
    org.apache.kafka.common.security.plain.PlainLoginModule required  
    username="admin"  
    password="admin-secret"  
    user_admin="admin-secret"  
    user_alice="alice-secret";  
};
```

## 在正在运行的集群中修改 SASL 机制

可以使用以下顺序在正在运行的集群中修改 SASL 机制：

1. 通过将机制添加到每个代理的`server.properties`中的 sasl.enabled.mechanisms 来启用新的 SASL 机制，并且更新 JAAS 配置文件以包括此处所述的两种机制。增量的部署集群节点。
2. 客户端使用新的机制配置后并重启。
3. 如果要修改Broker之间的通信机制的话，请将 server.properties 中的 `sasl.mechanism.inter.broker.protocol` 设置为新机制并再次增量启动集群。
4. 如果要删除旧的SASL机制，请从 server.properties 中的`sasl.enabled.mechanisms`中删除旧机制，并从 JAAS 配置文件中删除旧机制的条目。再次增量重启集群。

**本质上就是：**

如果要修改SASL机制的话，需要先添加SASL的机制，让新旧能够共存。
如果还需要把旧的机制删除的话，则完成了上面的步骤之后才能去做删除。

## 从 Broker 到 ZooKeeper 的连接身份验证

设置系统属性`zookeeper.sasl.clientconfig` , 这个系统属性表示的是
Zookeeper SASL 客户端的JAAS配置文件中使用的LoginModule，例如KafkaServer 、KafkaClient。

但是系统属性`zookeeper.sasl.clientconfig` 也可以不去设置，因为默认的值是`Client`，所以我们只需要在JAAS配置文件里面配置 Client模块就可以了，如下面的Client模块

```
KafkaServer {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin"  
        password="admin@password"  
        user_admin="admin@password"  
        user_szz="szz@123";  
};  
  
KafkaClient {  
        org.apache.kafka.common.security.plain.PlainLoginModule required  
        username="admin_sasl_plaintext_username"  
        password="admin_sasl_plaintext_password";  
  
};  
  
Client{  
 省略...  
};
```

- END -

点击阅读原文可获得更好阅读体验
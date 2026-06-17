---
title: Java 应用出现 Public Key Retrieval is not allowed 报错的常见原因和解决方法
author: MySQL实战
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5OTY2MjU5MQ==&mid=2247497037&idx=1&sn=b9527a6fadd172cde3c4a7f49c2dfc7d&chksm=c1eb74b888ec2d37a140e00d0d8b61a42e1ee448088b078c9361ed089e7a201c197e6c3c1240&mpshare=1&scene=24&srcid=0512pyhQrphTYtlxx04cDZxe&sharer_shareinfo=5e972ba20ef3c33524fc40d9d4b1937d&sharer_shareinfo_first=df849d085451886b4bf81f9e5e547d28#rd
---

# 问题现象

Java 应用在运行过程中突然报`java.sql.SQLNonTransientConnectionException: Public Key Retrieval is not allowed`错误。

开发童鞋表示不理解，毕竟应用没做任何变更，为什么会突然出现这个错误？

```
2025-03-31 08:31:11 - create connection SQLException, url: jdbc:mysql://10.0.1.86:3306/information_schema?useSSL=false, errorCode 0, state 08001  
java.sql.SQLNonTransientConnectionException: Public Key Retrieval is not allowed  
        at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:110)  
        at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:122)  
        at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:828)  
        at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:448)  
        at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:241)  
        at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:198)  
        at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1682)  
        at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)  
        at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2914)  
        ...
```

# 问题原因

用户使用的密码认证插件是 caching\_sha2\_password 且 JDBC 连接串中指定了`useSSL=false`。

当碰到以下场景，就会出现上述报错：

1. 第一次连接时。
2. 数据库实例发生了重启。
3. 服务端执行了 FLUSH PRIVILEGES 操作。

# 解决方法

解决方法有以下几种：

**方案 1**

将用户的密码认证插件调整为 mysql\_native\_password。不推荐该方法，因为 MySQL 9.0 移除了 mysql\_native\_password。

```
ALTER USER 'u1'@'%' IDENTIFIED WITH mysql_native_password BY 'MySQL@2025';
```

**方案 2**

在 JDBC 连接串中设置`useSSL=true`。推荐该方法，但开启 SSL 会有一定的性能开销。

如果既不想开启 SSL，又想避免 Public Key Retrieval is not allowed 错误，以下是两个可选的解决方案：

**方案 3**

在 JDBC 连接串中添加`allowPublicKeyRetrieval=true`。

该方法会自动从 MySQL 服务端获取 RSA 公钥，但这种方法有一定的安全风险，可能会受到中间人攻击。攻击者可以伪造 RSA 公钥，窃取用户密码。

**方案 4**

在 JDBC 连接串中指定`serverRSAPublicKeyFile`。

该方法需要将方案 3 中的公钥内容写到应用侧的一个本地文件中，具体步骤如下：

* 通过`show status like 'Caching_sha2_password_rsa_public_key'`命令或者从参数 caching\_sha2\_password\_public\_key\_path（默认是 public\_key.pem）指定的文件中获取公钥内容。
* 将公钥内容保存到应用侧的一个本地文件中。
* 在 JDBC 连接串中指定该公钥文件路径。如，

```
JDBC_URL = "jdbc:mysql://10.0.1.86:3306/information_schema?useSSL=false&serverRSAPublicKeyFile=/usr/local/caching_sha2_password_public_key.pem"
```

相较于方案 3，方案 4 无疑会更安全，但在高可用环境下并不适用，因为主从节点的公钥内容通常不同。一旦发生主从切换，JDBC 客户端在重新连接新的主节点时，就可能因公钥不一致而触发 Public Key Retrieval is not allowed 错误。除非在部署时显式配置，让主从节点使用相同的公钥，才能避免该问题。

## 四种方案的优缺点对比

  

| 方案 | 安全等级 | 适用场景 | 备注 |
| --- | --- | --- | --- |
| SSL 加密连接 (useSSL=true) | ★★★★★ | 所有生产环境 | 最安全方案，推荐优先使用 |
| 手动配置 RSA 公钥 (serverRSAPublicKeyFile) | ★★★★☆ | 禁用 SSL 的生产环境 | 需维护公钥文件 |
| 自动获取 RSA 公钥 (allowPublicKeyRetrieval=true) | ★★☆☆☆ | 可信内网环境 | 存在中间人攻击风险 |
| 更改认证插件 | ★☆☆☆☆ | 不推荐 | 兼容性差，安全性低 |

# 根因分析

简单来说，caching\_sha2\_password 插件为了加快认证过程，在服务端维护了一个**密码哈希缓存**。当客户端发起连接时：

* 如果用户的密码哈希已经被缓存，服务端可以直接验证，无需客户端发送明文密码进行验证。
* 如果缓存中没有该用户的密码哈希（比如第一次连接时，除此之外，数据库重启或执行 FLUSH PRIVILEGES，还会清除密码哈希缓存），则客户端需要发送明文密码进行认证。

在发送明文密码时，出于安全考虑，MySQL 要求：

* 要么客户端和服务端之间建立 SSL 加密连接。
* 要么客户端允许通过服务端公钥加密明文密码。

如果两者都不满足，就会抛出 Public Key Retrieval is not allowed 错误。

# caching\_sha2\_password 的认证交互流程

以下是客户端与服务端使用 caching\_sha2\_password 插件进行身份认证时的完整交互流程：

具体实现细节如下：

1. MySQL 服务端收到客户端的请求后，会生成一个长度为 21 字节的随机数。
2. MySQL 服务端给客户端发送一个握手包（handshake packet）。

   该握手包是 MySQL 服务端与客户端建立连接时发送的第一个数据包，包的内容如下：

   握手包的构造和发送是在`send_server_handshake_packet`函数中实现的。

1. 协议版本。
2. 服务端版本。
3. 线程 ID。
4. 随机数的第一部分。
5. 服务端能力标志（低16位） 。
6. 默认字符集编号。
7. 服务端状态标志。
8. 服务端能力标志（高16位） 。
9. 随机数的的长度。
10. 保留字段。
11. 随机数的第二部分。
12. 默认的密码认证插件名称。

    在 MySQL 8.4 之前，默认的密码认证插件由 default\_authentication\_plugin 参数决定。在 MySQL 5.7 中，该参数的默认值为 mysql\_native\_password，在 MySQL 8.0 中，该参数的默认值为 caching\_sha2\_password。在 MySQL 8.4 中，移除了这个参数，默认的密码认证插件硬编码为 caching\_sha2\_password。

3. 客户端在接收到服务端的握手包后，会根据握手包中的密码认证插件进行身份验证。对于 caching\_sha2\_password 插件，客户端将基于以下公式构造一个 `scramble_response`，然后调用`prep_client_reply_packet`构造握手包发送给服务端。公式中的 random 是服务端在握手包中发送的随机数。

   ```
   scramble_response = XOR(SHA256(password), SHA256(SHA256(SHA256(password)) + random)
   ```
4. 服务端调用`parse_client_handshake_packet`解析客户端返回的握手包，解析出来的内容包括用户名、密码（不是明文密码，是 scramble\_response）、库名。
5. 基于用户名和客户端 IP 从`ACL`缓存中找到用户对应的认证信息，包括密码认证插件和密码哈希值。
6. 调用`Caching_sha2_password::fast_authenticate`进行快速验证。快速验证的逻辑如下：

* 首先基于用户名和主机名构造一个 authorization\_id。
* 判断 authorization\_id 是否在 m\_cache 存在。m\_cache 即密码哈希缓存，是 caching\_sha2\_password 中的关键组件，它的底层实现是一个哈希表。该哈希表的键是 authorization\_id，值是一个二维数组，用来存储新旧两个密码哈希值（从 MySQL 8.0 开始，一个用户可以设置两个密码），存储的值是对密码进行两次 SHA-256 哈希计算的结果，即`SHA256(SHA256(password)`。
* 如果存在，则通过以下步骤验证密码是否正确。

  ```
  SHA2(m_known, rnd) => scramble_stage1  
  XOR(scramble, scramble_stage1) => digest_stage1  
  SHA2(digest_stage1) => digest_stage2  
  m_known == digest_stage2
  ```

  其中，m\_known 是 m\_cache 中存储的密码哈希值，即 SHA256(SHA256(password)，rnd 是服务端发送给客户端的随机数（random），scramble 是客户端返回给服务端的 scramble\_response。

  最后，将 m\_known 与 digest\_stage2 进行比较，如果相等，则意味着密码正确。这个时候，服务端会给客户端发送一个`fast_auth_success`包。
* 如果 authorization\_id 在 m\_cache 中不存在，或者存在但不匹配，则意味着快速验证失败，这个时候，服务端会给客户端发送一个`perform_full_authentication`包，要求客户端发送密码进行身份验证。

7. 客户端收到服务端发送的`perform_full_authentication`包后：

* 如果连接是安全的（即开启了 SSL），则会向客户端发送明文密码。
* 如果连接不是安全的（即没有开启 SSL，一般是因为客户端显式指定了`--ssl-mode=DISABLED`），则会调用 rsa\_init() 初始化 RSA 公钥，这个公钥是 mysql 客户端参数 server-public-key-path 指定的。如果没指定，则看 mysql 客户端参数中是否指定了get-server-public-key。

  如果既没指定 server-public-key-path，又没指定 get-server-public-key，则 mysql 客户端会提示`ERROR 2061 (HY000): Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.`错误。

  JDBC 中的处理逻辑类似，对于非 SSL 连接，如果既没指定 serverRSAPublicKeyFile，又没指定allowPublicKeyRetrieval，则会提示`Public Key Retrieval is not allowed`错误，具体的实现细节可参考 CachingSha2PasswordPlugin.java 中的 nextAuthenticationStep 方法。

  如果指定了 get-server-public-key，则客户端会向服务端发送一个`request_public_key`包。

8. 服务端收到客户端发送的`request_public_key`包后，会将 RSA 公钥发送给客户端。
9. 客户端收到 RSA 公钥后，首先会将密码和服务端之前发送的随机数进行异或运算，然后使用 RSA 公钥对异或后的结果进行加密，最后将加密后的密文发送给服务端。
10. 服务端收到加密后的密文后，会调用 decrypt\_RSA\_private\_key 进行解密，获取明文密码。
11. 服务端 Caching\_sha2\_password::authenticate 验证密码是否正确。验证的逻辑如下：

* 从 mysql.user 表 authentication\_string 字段中提取迭代次数和盐值。

  对于 caching\_sha2\_password，authentication\_string 的格式如下：

  ```
  分隔符[摘要类型]分隔符[迭代次数]分隔符[盐值][哈希摘要]
  ```

  其中，分隔符默认是 $，摘要类型是单个字母，A 表示使用 SHA256 算法，迭代次数是 3 位十六进制字符串，默认是 005，乘以 ITERATION\_MULTIPLIER（默认是 1000），即为实际迭代次数（默认是 5000 次）。盐值是一个长度为 20 的随机字符串，剩下的字符串即为哈希摘要，长度为 43 字节。
* 基于提取的迭代次数、盐值和明文密码生成一个哈希摘要。
* 判断提取的哈希摘要和生成的哈希摘要是否相等，如果相等，则意味着密码正确，否则是错误的。

# 参考资料

1. Protocol::HandshakeV10：https://dev.mysql.com/doc/dev/mysql-server/latest/page\_protocol\_connection\_phase\_packets\_protocol\_handshake\_v10.html
2. Caching\_sha2\_password information：https://dev.mysql.com/doc/dev/mysql-server/latest/page\_caching\_sha2\_authentication\_exchanges.html
3. WL#9591: Caching sha2 authentication plugin：https://dev.mysql.com/worklog/task/?id=9591
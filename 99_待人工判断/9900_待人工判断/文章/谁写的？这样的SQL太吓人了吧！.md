---
title: 谁写的？这样的SQL太吓人了吧！
author: dbaplus社群
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkzMjYzNjkzNw==&mid=2247627151&idx=1&sn=4ce18e1fe2e64018929db9ec2aeda6e4&chksm=c38e7869a6d9ccdac8c938e02c96f1d8745b02e591d996879966650d15d6d35918ce10641e6c&mpshare=1&scene=24&srcid=1210TiO8BGLszOoE2o4ipSHQ&sharer_shareinfo=7dc91862ad145b9a33187f096d20ff6f&sharer_shareinfo_first=7dc91862ad145b9a33187f096d20ff6f#rd
---

昨天笔者在朋友圈发了这样一张图：

很多小伙伴看到了能够快速发现问题，当 company\_id 为 null 的时候，会导致全表更新。

但是也有小伙伴不解，自己平时就是这么写的呀，也没什么问题，如果有问题，那么上面的 SQL 该怎么改呢？

笔者来和大家简单聊几句。

**一、防止全表更新**

如果在生产环境中使用 UPDATE 语句更新表数据，此时如果忘记携带本应该添加的 WHERE 条件，那么后果不堪设想。

那么怎么避免这个问题呢？

**二、sql\_safe\_updates**

sql\_safe\_updates 是 MySQL 数据库中的一个参数，它的作用是增强数据安全性，防止因误操作导致的数据丢失或破坏。

具体来说，当 sql\_safe\_updates 设置为 ON（启用）时，MySQL 将阻止执行没有明确 WHERE 子句的 UPDATE 或 DELETE 语句。这意味着如果试图运行一个不包含 WHERE 条件来限定更新或删除范围的 DML 语句，MySQL 会抛出一个错误。而当 sql\_safe\_updates 设置为 OFF（禁用）时，MySQL 不会对此类无条件更新或删除操作进行特殊限制，允许它们按常规方式执行

这个参数可以配置在会话级别或全局级别。

在会话级别，可以通过执行 SET sql\_safe\_updates = 1; 命令来启用，这只对当前连接有效。

在全局级别，可以通过 SET GLOBAL sql\_safe\_updates = 1; 命令或在 MySQL 配置文件中设置，这会影响服务器上所有新的会话，但是这个配置不会修改当前会话。

启用 sql\_safe\_updates 参数可以减少因人为失误引发的重大数据事故，尤其适合开发环境和对数据完整性要求严格的生产环境。

我们可以先执行 SHOW VARIABLES LIKE '%sql\_safe\_updates%'; 查看当前配置：

然后执行 SET sql\_safe\_updates = 1; 去更新，更新之后再去查看配置，发现 sql\_safe\_updates 就已经开启了：

这个时候，假设我们执行如下 SQL：

```
UPDATE user set username='javaboy';
```

就会报一个错误：

需要注意的是，启用 sql\_safe\_updates 参数可能会影响现有应用程序的正常运行，特别是那些依赖于无条件更新或删除操作的程序，因此在生产环境中启用之前，必须确保所有相关的应用程序代码已经过严格审查和适配。

**三、SQL 插件**

MyBatis-Plus 提供了一个非法 SQL 拦截插件叫做 IllegalSQLInnerInterceptor。这是 MyBatis-Plus 框架中的一个安全控制插件，用于拦截和检查非法 SQL 语句。

这个插件主要提供了四方面的功能：

* 识别并拦截特定类型的 SQL 语句，如全表更新、删除等高风险操作。
* 确保在执行查询时使用索引，以提高性能并避免全表扫描。
* 防止未经授权的全表更新或删除操作，减少数据丢失风险。
* 对包含 not、or 关键字或子查询的 SQL 语句进行额外检查，以防止逻辑错误或性能问题。

插件用法也简单，配置一个 Bean 即可：

```
@Configurationpublic class MybatisPlusConfig {  
    @Bean    public MybatisPlusInterceptor mybatisPlusInterceptor() {        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();        // 添加非法SQL拦截器        interceptor.addInnerInterceptor(new IllegalSQLInnerInterceptor());        return interceptor;    }}
```

配置完成后，如果执行了不带 where 条件的 update 或者 delete 语句，就会报如下错误。

但是！！！

如果你的 SQL 后面有个 where 1=1，那么这样的 SQL 是不会被 IllegalSQLInnerInterceptor 插件识别并拦截的。

**四、IDEA 插件**

利用 IDEA 的一些插件，也可以检测到有风险的 SQL，比如笔者常用的这个：

不过这些插件不一定能检测出来文章一开始所提出的问题。

**五、Code Review**

日常的 Code Review 也不可少，很多问题都是在 CR 的时候发现的。

**六、问题解决**

除了上面提到的各种办法之外，对于本文一开始提出的问题，这个有问题的 SQL 还可以做哪些修改呢？

欢迎小伙伴们评论区给出自己的答案～笔者也会在评论区给出我的看法！

作者丨江南一点雨

来源丨公众号：江南一点雨（ID：a\_javaboy）

dbaplus社群欢迎广大技术人员投稿，投稿邮箱：editor@dbaplus.cn

**活动推荐**

为了和大家一起探索AI相关技术在大数据、数据资产管理、数据库、运维等领域的最佳落地方式，挖掘由此激发的软件发展和技术进步，**第九届DAMS中国数据智能管理峰会将于2024年11月29日在上海举办**，携手一众产学研界技术领跑单位，带来新思路、重实践、可落地的全日干货盛宴。**码上报名↓**
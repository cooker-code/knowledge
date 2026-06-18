---
title: 果不其然，SQLite的研究引来一堆人的关注，上一篇爆了
author: AustinDatabases
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509299&idx=1&sn=27c4446fdb0d1db5e5723c0fe06b2e74&chksm=ce6af8dc71f165288549c4beb6d403b8683400733ec7603f6267c6ff414d82542be17a2b5ea9&mpshare=1&scene=24&srcid=1121WCaddead9nkYDMpXPWJd&sharer_shareinfo=65f1d643726ef01c3cbb1952226b0012&sharer_shareinfo_first=65f1d643726ef01c3cbb1952226b0012#rd
---

> ❝
>
> 开头还是介绍一下群，如果感兴趣PolarDB ,MongoDB ,MySQL ,PostgreSQL ,Redis, OceanBase, Sql Server等有问题，有需求都可以加群群内有各大数据库行业大咖，可以解决你的问题。加群请联系 liuaustin3 ，（共3300人左右 1 + 2 + 3 + 4 +5 + 6 + 7 + 8 +9）(1 2 3 4 5 6 7群均已爆满，开8群近400 9群 200+，开10群PolarDB专业学习群100+)

上一篇不知道是怎么了，一堆加群的人，看来SQLite 的确是数据库流量密码，留言板也人数超多，世界装机量最大的数据库，这是有道理的。

本来今天是一期为什么SQLite3是装机最多的数据库，且打败了PostgreSQL的分析，但留言板里面很多希望一期学习的人，那么这期咱们就继续学习SQLite。

这里我总结几个部分后续要进行学习和分享

1 Sqlite的数据库体系结构上是什么，表怎么建立，建立表的方式和别的数据库有什么不同。

2 Sqlite的数据库特点是什么，使用中的需要注意的部分，比如他是一个读并发，写单线程的，带有库锁的数据库

3 Sqlite 数据库，配置，运维，以及运维

4  如何设计一个应用程序在Sqltie 中

以上四个问题是我们后续要进行的，并和大家分享的、

---

今天我们现开始学习，sqlite的常用的知识，以及一些操作命令然后我们在对一些简单的SQLite的命令进行简单的学习，本篇是开始学习SQLite的第三天。

SQLite 作为轻量级的数据库，其数据库类型体系与传统的数据类型差异是很签注的额，从数据库类型上看，他的主要核心是围绕动态类型，和兼容类型他的数据类型并不精准，而是围绕灵活性而来。

SQLite的第一个区别就是动态类型，比如我们PostgreSQL中INT，我们只能存储数据类型，而SQLite不是的，即使你声明的表类型是INT，他也可以存储字符串类型，浮点等，列定义仅仅作为参考，不做强制。

后面在SQLite中又有strict表，这类表才是严格和传统数据库类型强制一致的表，如果有此类需求需要建表的时候建立strict表。

SQlite的数据类型主要有五种，NULL, INTEGER ,REAL, TEXT, BLOB 等，这里SQLite里面是没有时间类型的，时间类型需要转化，通过date, strftime函数来进行。

日期时间类型：无独立存储类，需通过以下 3 种格式存储，再用内置日期函数（如 date()、strftime()）转换：

TEXT：ISO8601 格式（如 2024-11-14 12:00:00）； REAL：儒略日（自公元前 4714 年 11 月 24 日格林威治正午起的天数）； INTEGER：Unix 时间戳（自 1970-01-01 00:00:00 UTC 起的秒数）

这里需要注意如果建立不同表，那么你输入的类型如果是5.0 会自动转化为 5整型，而不是浮点类型 real。

可能是随着SQLite使用的越来越多，作为移动端的数据越来越重要，那种数据库把你的数据类型转错的情况就发生了，所以后续的SQLite有新的一个表类型strict，建立表需要建表的末尾添加strict 也就是严格模式。

列类型强制要求

所有列必须显式指定类型，不能省略； 仅允许 6 种类型：INT、INTEGER、REAL（浮点）、TEXT（字符串）、BLOB（二进制）、ANY（任意类型），无其他可选类型（未来可能新增）。

（2）数据插入规则

非ANY类型列：插入数据需是NULL（无NOT NULL约束时）或匹配列类型；SQLite 会按 “类型亲和性规则” 尝试转换（与其他数据库一致），无法无损转换则抛出SQLITE\_CONSTRAINT\_DATATYPE错误（例：INTEGER列插入'xyz'会报错）。 ANY类型列：可接受任意类型数据（NOT NULL约束时拒绝NULL），不做任何类型转换，完全保留原始数据类型与值。

（3）主键与完整性检查

主键列隐式包含NOT NULL约束；但INTEGER PRIMARY KEY列插入NULL时，会自动转为唯一整数（与非严格表规则一致）。 PRAGMA integrity\_check/quick\_check命令会检查 STRICT 表的列类型，异常时提示错误。

（4）与非严格表的共性 除上述差异外，STRICT 表的其他特性与普通表完全一致，包括：CHECK/NOT NULL/FOREIGN KEY/UNIQUE约束、DEFAULT/COLLATE子句、生成列、ON CONFLICT处理、索引、AUTOINCREMENT、INTEGER PRIMARY KEY作为rowid别名、磁盘存储格式等。

3. ANY 数据类型：特殊规则 设计目的：在严格模式下，仍支持 “单列存储任意类型数据” 的灵活能力（SQLite 独有特性）。

STRICT 表 vs 非严格表的差异： STRICT 表的ANY列：完全保留原始数据（例：插入'000123'，存储为text类型的'000123'）； 非严格表的ANY列：会尝试将 “类数字字符串” 转为数值（例：插入'000123'，存储为integer类型的123）。

4. 向后兼容性

（1）版本限制 仅 SQLite 3.37.0 及以上版本能识别STRICT关键字；低版本打开含STRICT表的数据库时，默认报错（除特殊情况外）。 不含STRICT表的数据库，3.37.0 及以上版本创建后，仍可被低至 3.0.0（2004 年）的版本读写。

（2）低版本访问 STRICT 表的特殊方式 低版本可通过打开数据库后立即执行PRAGMA writable\_schema=ON（关闭 schema 解析错误），忽略STRICT关键字并读写表，但不会触发严格类型检查（可能导致数据类型错误，需用高版本PRAGMA quick\_check检测）。 SQLite CLI 的.dump命令默认启用writable\_schema=ON，低版本用.dump可读写 STRICT 表，但同样存在数据损坏风险。

5. 其他表选项 CREATE TABLE语句末尾（闭合括号后）可接逗号分隔的表选项，目前仅支持 2 种： STRICT（严格类型）、WITHOUT ROWID（无rowid表）； 选项顺序不限，当前版本允许重复选项（未来可能取消，不建议依赖）。

所以针对业务，希望开发人员在3.37版本后的SQLite应该这对业务中的一些精确的数字，或者数字就是文字的部分应该严格，建表就要建立一个严格的表。

建立一个复杂的表

CREATE TABLE orders (order\_id  INTEGER PRIMARY KEY,order\_no TEXT NOT NULL UNIQUE,customer\_id  INTEGER NOT NULL,amount REAL NOT NULL,pay\_type  TEXT CHECK(pay\_type IN ('CARD','CASH','WALLET')),created\_at TEXT NOT NULL,raw\_payload  ANY,note TEXT,FOREIGN KEY (customer\_id) REFERENCES customers(id)) STRICT;

SQLite 并不需要启动，你可以认为他是一个程序，通过SQLite 数据库文件名，就可以进入到数据库进行操作。

查看系统的版本 sqlite> .version SQLite 3.47.1 2024-11-25 12:07:48 b95d11e958643b969c47a8e5857f3793b9e69700b8f1469371386369a26e577e zlib version 1.2.11 gcc-8.5.0 20210514 (Red Hat 8.5.0-28) (64-bit) sqlite>

SQLite3

对Sqlite的日常命令不熟悉，可以通过.help进行更详细的命令的问询

对sqlite中的有多少表进行询问

查看有多少表使用 .table .schema 表名 查看表结构

```
sqlite>   
sqlite> .tables  
id         test_data  
sqlite> .schema test_data  
CREATE TABLE test_data (  
 id INTEGER PRIMARY KEY AUTOINCREMENT,  
 name TEXT,  
 value INTEGER,  
 created_at DATETIME DEFAULT CURRENT_TIMESTAMP  
);  
sqlite> 
```

常见命令

这里sqlite有自己的系统表结构，可以通过来进行查看

```
sqlite> select * from sqlite_master;  
table|id|id|2|CREATE TABLE id (id int)  
table|sqlite_sequence|sqlite_sequence|4|CREATE TABLE sqlite_sequence(name,seq)  
table|test_data|test_data|3|CREATE TABLE test_data (  
  id INTEGER PRIMARY KEY AUTOINCREMENT,  
  name TEXT,  
  value INTEGER,  
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP  
)  
Run Time: real 0.000 user 0.000020 sys 0.000039
```

具体我们查看表名前缀为test的表

查询特定前缀表

这里遇到一个问题，在对表进行删除的过程中，发现报错

image

遇到此问题，目前最快速的方案是，退出SQLite然后在进入进行命令的操作。

总结: SQLite在我使用中，经常有database lock的错误产生，（因为我还是用传统数据库思维去操作的问题），解决也很简单，查杀链接，或把.quit数据库即可，解决库锁的问题。在操作中命令不支持上下箭头来进行历史命令的查找，建表要注意宽松模式和严苛模式，其实我逐步对SQLite的深入，我其实对他有很多的问题。

1 SQLLite数据支持数据压缩吗，因为我现在一个表，几百万的数据就270MB的文件大小了，说明这个数据库本身数据的文件的控制有问题，应该有可以进行碎片整理的命令，这个我看到了，但我还没有深入。

2 在研发中，使用SQLite3 如何进行开发和库表的使用，用传统数据库使用思路是一定不行的，还需要打破原有的设计思路来避免库锁的问题。

3 因为我曾经有一次服务器重启，然后进入SQLite特别慢的经历，说明断电对第一次SQLite启动是有阻碍的，应该是wal模式下数据在重做导致的，但是一个关键的问题，SQLite 没有日志，没有错误的报告等等，这些还需要研究。

置顶

[SQLite3 打败了 PostgreSQL 终究还是没能挽回--世界最大装机量是真的](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509279&idx=1&sn=36daf438e45d9ca1694942d169733d1e&scene=21#wechat_redirect)

[回复群友问题，PostgreSQL Extensions 哪些是常用的](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509252&idx=1&sn=20f3f22601c7f8f9c1201766dc9c381e&scene=21#wechat_redirect)

[PostgreSQL 2025杭州大会--掐指一算，原来待在这里 7年了！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509250&idx=1&sn=453e1d645a85f3bc8962752e25d70c16&scene=21#wechat_redirect)

[说搞国产数据库生态，骗鬼呢？ 群里服务商吐槽后的 “大实话”](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509249&idx=1&sn=a37db9c9035a0d7dd6584a64105ddfc7&scene=21#wechat_redirect)

[“MySQL” 2025年我用上物化视图功能，谁家的MySQL有这个功能？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509243&idx=1&sn=0305dc36a976e952c4e6afd986cebb18&scene=21#wechat_redirect)

[民营企业领导问 外部客户数据库选型为什么是 OceanBase](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509171&idx=1&sn=8dd810ec4647f8af5bd25ef43c72356d&scene=21#wechat_redirect)

[PostgreSQL 真实压测，分析PG18 17 16 15 14 之间在处理SQL和系统性能稳定性的差异](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509161&idx=1&sn=5c3a09802220f1478d5726cad1d44270&scene=21#wechat_redirect)

[PostgreSQL 迁移到 PolarDB 2万5千里长征，太难了，太难了 （今天DISS阿里云某部门）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509113&idx=1&sn=473002f58eb2dc63f476d3300ca762f2&scene=21#wechat_redirect)

[数据库HTAP概念新解读，一定和你知道的不一样](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509112&idx=1&sn=63b8aa0372cd842430dde68be74cddc3&scene=21#wechat_redirect)

[Oracle 26i 的一个功能演进后，云厂商利用会不会造出千年老妖样的“数据库”](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509091&idx=1&sn=047573809b102d51cbbd54e0148bffcd&scene=21#wechat_redirect)

[在某国产数据库 “小黑屋” 会议后的 感想和记录](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509056&idx=1&sn=c25ff3e75c67b7c3ec3ab0a9af438dd1&scene=21#wechat_redirect)

[“一顿海鲜引发”（3）一分钟定位数据库问题，试用得京东卡和礼物！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509103&idx=1&sn=69b8437578f36923b7ae7f6ad6e37c68&scene=21#wechat_redirect)

[“一顿海鲜引发”（2）“运维工具与DBA之间不打不相识”](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509034&idx=1&sn=774f70e79499be90213d44684260c101&scene=21#wechat_redirect)

[“一顿海鲜引发”（1）：DBA、架构师与数据库运维工具的爱恨情仇](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508990&idx=1&sn=9bbb73eeee1574e97ceedbbc1cf9d31a&scene=21#wechat_redirect)

[DBA 从“修电脑的” 到 上演一套 “数据治理” 大戏 ---  维护DBA生存空间，体现个体价值](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508994&idx=1&sn=135b289271f9b8c0a0dd014395628836&scene=21#wechat_redirect)

[Oracle 也有做失败的数据库系统？是的今天我们来说说他](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247509035&idx=1&sn=34b9b1e63aa77ae2ea8f93a63a7c3fb1&scene=21#wechat_redirect)

[老板说 MongoDB 测试环境这么贵，弄单机？ 开发说要复制集测试？ 你们这群XXX！！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508975&idx=1&sn=c9a6e42d4566c13ac65f3567a25bef9d&scene=21#wechat_redirect)

[国庆节2号 PostgreSQL 停机罢工 协助 解决问题得 66.66元的红包](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508910&idx=1&sn=fdf09d85ba23323fa9aa8a787d360d4b&scene=21#wechat_redirect)

[外媒评论区疯狂了，开发人员各种观点---北美AI替换程序员引发境外程序员业界震动](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508880&idx=1&sn=305f43568cb9646502226a2844743872&scene=21#wechat_redirect)

[MySQL 8 的老大难问题，从5.7延续至今，这个问题有这么难？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508879&idx=1&sn=ae89c184b31945d889c620111f961432&scene=21#wechat_redirect)

[体育生误入 DBA 行业，后悔了，问换哪行？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508858&idx=1&sn=2c5a19cff27dc33528ef356fb5b3f971&scene=21#wechat_redirect)

[一篇为MySQL用户，分析版本核心差异的文章--8.028-8.4的差异](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508851&idx=1&sn=2ba1f6bd02b1e53cd06d0e9e28bc6552&scene=21#wechat_redirect)

[云上DBA是诸葛亮，云下的DBA是 关云长，此话怎讲？ 4点变化直击要害](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508797&idx=1&sn=c4616584d984860c43f478d1326e2e6a&scene=21#wechat_redirect)

[外国专家说PG 18 AI能力不行，到底行不行？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508763&idx=1&sn=3a71d135f5a0afad7f13accb5a498ec1&scene=21#wechat_redirect)

[MongoDB 开始接客户应用系统 AI 改造的活了--OMG 这世界太疯狂](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508761&idx=1&sn=c8883a92de6a528641ba02731d3abb3b&scene=21#wechat_redirect)

[一篇将PostgreSQL 日志问题说的非常详细附带分析解决方案的文章  （翻译）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508760&idx=1&sn=2cf5ad39448a3bf9513042853994f141&scene=21#wechat_redirect)

[DBA 与 AI 斗智斗勇的一天，谁是麦当劳，谁是星巴克](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508692&idx=1&sn=432d418bcec154fd6d233974087ecfa4&scene=21#wechat_redirect)

[科技改变生活，阿里云DAS  AI改变了什么](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508685&idx=1&sn=e377028422c3d7acb9405df3f43710ac&scene=21#wechat_redirect)

[企业DBA 应该没听说过 Supabase，因为他不单纯 ！！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508677&idx=1&sn=60c1305d062de0d1727eb12bb928877f&scene=21#wechat_redirect)

[Oracle 推出原生支持 Oracle 数据库的 MCP 服务器，助力企业构建智能代理应用](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508662&idx=1&sn=768cfdfaf334673018ac683be9ae268a&scene=21#wechat_redirect)

[PolarDB MySQL SQL 优化指南 （SQL优化系列 5）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508637&idx=1&sn=3f2707056c68d3ca5d6f932cb6f14bec&scene=21#wechat_redirect)

[开发欺负我 Redis  的大 keys的问题，我一个DBA怎么解决？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508635&idx=1&sn=688cd4dcba44ee1398b957a190e858ea&scene=21#wechat_redirect)

[IF-Club 你提意见拿礼物 AustinDatabases 破 10000](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508635&idx=2&sn=e24734e4fd067631f544286d2bd9c755&scene=21#wechat_redirect)

[开发欺负我 Redis  的大 keys的问题，我一个DBA怎么解决？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508635&idx=1&sn=688cd4dcba44ee1398b957a190e858ea&scene=21#wechat_redirect)

[云基座技术是大厂专有，那小厂和私有云的出路在哪里？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508632&idx=1&sn=ef3913f51a72ee273711b07b051074b8&scene=21#wechat_redirect)

OceanBase 相关文章

[某数据库下的一手好棋！共享存储落子了！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507372&idx=1&sn=1138598273e796f394995c9b345bc6fd&scene=21#wechat_redirect)

[OceanBase 光速快递 OB Cloud “MySQL” 给我，Thanks a lot](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507759&idx=1&sn=a091bceb002c5ab9cf3484ae7e6b1925&scene=21#wechat_redirect)

[和架构师沟通那种“一坨”的系统，推荐只能是OceanBase，Why ?](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507201&idx=1&sn=73512eb5026306e9ca0245c514336a9c&scene=21#wechat_redirect)

[OceanBase Hybrid search 能力测试，平换MySQL的好选择](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507845&idx=1&sn=a1314bf4068b2399ea93236c1abf6104&scene=21#wechat_redirect)

[某数据库下的一手好棋！共享存储落子了！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507372&idx=1&sn=1138598273e796f394995c9b345bc6fd&scene=21#wechat_redirect)

[写了3750万字的我，在2000字的OB白皮书上了一课--记 《OceanBase 社区版在泛互场景的应用案例研究](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507342&idx=1&sn=5ecffca6f66542aed566fdb596e63466&scene=21#wechat_redirect)

[OceanBase 单机版可以大批量快速部署吗？ YES](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506958&idx=1&sn=caf26b94dbf39eb4ca541cbd80a33034&scene=21#wechat_redirect)

[OceanBase 6大学习法--OBCA视频学习总结第六章](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506776&idx=1&sn=115f0e3b142ae2e8ae0dc5912defd929&scene=21#wechat_redirect)

[OceanBase 6大学习法--OBCA视频学习总结第五章--索引与表设计](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506640&idx=1&sn=c2eb4b9ca2abadca85aa322792d5cffd&scene=21#wechat_redirect)

[OceanBase 6大学习法--OBCA视频学习总结第五章--开发与库表设计](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506522&idx=1&sn=f615a55e2612cfecd8b3e0496a923763&scene=21#wechat_redirect)

[OceanBase 6大学习法--OBCA视频学习总结第四章 --数据库安装](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506342&idx=1&sn=92d7508a798d17bf6a1ef9644731e495&scene=21#wechat_redirect)

[OceanBase 6大学习法--OBCA视频学习总结第三章--数据库引擎](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506225&idx=1&sn=eaf803426277639c4f3aeb1f8fa96f39&scene=21#wechat_redirect)

[OceanBase 架构学习--OB上手视频学习总结第二章 （OBCA）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506002&idx=1&sn=857f09efdcc10fb57586ce581abd53b2&scene=21#wechat_redirect)

[OceanBase 6大学习法--OB上手视频学习总结第一章](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505915&idx=1&sn=249a012bccae429ccdc19dac99c8362e&scene=21#wechat_redirect)

[没有谁是垮掉的一代--记 第四届 OceanBase 数据库大赛](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505570&idx=1&sn=4b94b804bfa3b94c44592895febd8114&scene=21#wechat_redirect)

[OceanBase  送祝福活动，礼物和幸运带给您](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504744&idx=1&sn=48c7e89dca8e60de01ba55f555159d9d&scene=21#wechat_redirect)

[跟我学OceanBase4.0 --阅读白皮书 (OB分布式优化哪里了提高了速度)](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504559&idx=1&sn=e563e0eb1a6e00a510904dbed8f6a83b&scene=21#wechat_redirect)

[跟我学OceanBase4.0 --阅读白皮书 （4.0优化的核心点是什么）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504212&idx=1&sn=25545096c48f9075d33e1c8ef97a00a3&scene=21#wechat_redirect)

[跟我学OceanBase4.0 --阅读白皮书 （0.5-4.0的架构与之前架构特点）](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504032&idx=1&sn=086cc1228b84417f1add3d490187129f&chksm=cfbcb4fff8cb3de9d8ca587315048f2f5e7e3cdc0d509c6458c5ccb36edc3563c45f1da3ffdc&scene=21#wechat_redirect)

[跟我学OceanBase4.0 --阅读白皮书 （旧的概念害死人呀，更新知识和理念）](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503943&idx=1&sn=66876aeefbbedeb0e7aeb5b3f4921092&chksm=cfbcb418f8cb3d0ea2b4ceb391be40119ef745b440ae6355512d7126ab7dacb9336f3da5b4c2&scene=21#wechat_redirect)

[聚焦SaaS类企业数据库选型（技术、成本、合规、地缘政治）](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503828&idx=1&sn=b41db71cc52ddd4e64012c1a46e74b40&chksm=cfbcab8bf8cb229d0607745d2e56554023f2d02e61c08cc62de7928c069d473cf104c584e553&scene=21#wechat_redirect)

[OceanBase 学习记录-- 建立MySQL租户,像用MySQL一样使用OB](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503763&idx=1&sn=a8dc61135cefcb1d37db02a640f0fe55&chksm=cfbcabccf8cb22da98a3b7154c0951848feb7c9821483decb62c2b279bd848f987a360016d4f&scene=21#wechat_redirect)

[“合体吧兄弟们！”——从浪浪山小妖怪看OceanBase国产芯片优化《OceanBase “重如尘埃”之歌》](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508203&idx=1&sn=69f3bc65a8315486816f907cc80aae92&scene=21#wechat_redirect)

MongoDB 相关文章

[MongoDB “升级项目” 大型连续剧(4)-- 与开发和架构沟通与扫尾](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507212&idx=1&sn=c7d30f18ed567657bbe37c8e8aa61a22&scene=21#wechat_redirect)

[MongoDB “升级项目” 大型连续剧(3)-- 自动校对代码与注意事项](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507043&idx=1&sn=a8be8315d9152aae0d51392e75dad00a&scene=21#wechat_redirect)

[MongoDB “升级项目” 大型连续剧(2)-- 到底谁是"der"](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506966&idx=1&sn=5ce54887a68c70765c0ba66d367c19d7&scene=21#wechat_redirect)

[MongoDB “升级项目”  大型连续剧(1)-- 可“生”可不升](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506860&idx=1&sn=33e8256118091fa29171eb1d707f2536&scene=21#wechat_redirect)

[MongoDB  大俗大雅，上来问分片真三俗 -- 4 分什么分](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505627&idx=1&sn=fe466c209faeffa08e4b7b542941f075&scene=21#wechat_redirect)

[MongoDB 大俗大雅，高端知识讲“庸俗” --3 奇葩数据更新方法](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505554&idx=1&sn=dac53eb5a4af8c9e0e5656ffc4141c3c&scene=21#wechat_redirect)

[MongoDB 学习建模与设计思路--统计数据更新案例](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505551&idx=2&sn=8c7344fcd6e5ca606ac9e4966e892b40&scene=21#wechat_redirect)

[MongoDB  大俗大雅，高端的知识讲“通俗” -- 2 嵌套和引用](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505532&idx=1&sn=d9880ff649e608955317781301204987&scene=21#wechat_redirect)

**[MongoDB  大俗大雅，高端的知识讲“低俗” -- 1 什么叫多模](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505506&idx=1&sn=84a3f86a36bb60ccc5f42b4be4bc2b8c&scene=21#wechat_redirect)**

**[MongoDB 合作考试报销活动 贴附属，MongoDB基础知识速通](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504200&idx=1&sn=11fc078d76c216a99f40503795a0873e&chksm=cfbcb517f8cb3c01f1c9cf8f599c77ac3687f19f8664bfcdad52020e7a51982c156f23bf14af&scene=21#wechat_redirect)**

**[MongoDB 年底活动，免费考试名额 7个公众号获得](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504195&idx=1&sn=13e2ff57d21a88de64c9f91f8a07d13f&chksm=cfbcb51cf8cb3c0aad299ce4a901e9c621101712e1a5536a9d1a69dd0dee305eaf294f529cff&scene=21#wechat_redirect)**

**[MongoDB 使用网上妙招，直接DOWN机---清理表碎片导致的灾祸 (送书活动结束)](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503957&idx=1&sn=361d4c1452c13d998e89055ea071c0a3&chksm=cfbcb40af8cb3d1c8e2e25cc98b05f82897962918f34d5a0398d4e5364830150338d4f038d29&scene=21#wechat_redirect)**

**MongoDB 2023年度纽约 MongoDB 年度大会话题 -- MongoDB 数据模式与建模**

MongoDB  双机热备那篇文章是  “毒”

[MongoDB   会丢数据吗？在次补刀MongoDB  双机热备](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247498946&idx=1&sn=38127b0be4e1cf5917e2a966d36d6d8e&chksm=cfbc989df8cb118bf1dada3efef0d86e8dd4828a85a2f8775ab043768a04e7abfb12dd167941&scene=21#wechat_redirect)

[MONGODB  ---- Austindatabases  历年文章合集](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247498687&idx=2&sn=fe4512c0428a248c00f04693d6f68086&chksm=cfbc9fe0f8cb16f678e22e5e9d317d3a89cdb7ba63030d9b3c79fdb05d958f4da10ff60af5aa&scene=21#wechat_redirect)

[MongoDB 麻烦专业点，不懂可以问，别这么用行吗 ! --TTL](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507545&idx=1&sn=dda33de5958510074631bd9921f3c988&scene=21#wechat_redirect)

PolarDB 已经开放的课程

[PolarDB 非官方课程第八节--数据库弹性弹出一片未来--结课](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508006&idx=1&sn=bdd32267f658b48b203a2701daf74aa8&scene=21#wechat_redirect)

[PolarDB 非官方课程第七节--数据备份还原瞬间完成是怎么做到的--答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507978&idx=1&sn=4dd1ef89034f48226aa659fde6a6e9b8&scene=21#wechat_redirect)

[PolarDB 非官方课程第六节--数据库归档还能这么玩--答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507935&idx=1&sn=f96a0c53aad4fae9fc26197eb801eff9&scene=21#wechat_redirect)

[PolarDB 非官方课程第五节--PolarDB代理很重要吗？--答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507915&idx=1&sn=b7a43b47d80a1107481f5bf8aa48a7cd&scene=21#wechat_redirect)

[PolarDB 非官方课程第四节--PG实时物化视图与行列数据整合处理--答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507860&idx=1&sn=1ffd97bfb1cdd850753d15560cb4a001&scene=21#wechat_redirect)

[PolarDB 非官方课程第三节--MySQL+IMCI=性能怪兽--答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507840&idx=1&sn=58e05d2cdf43c8890624add0b60e1080&scene=21#wechat_redirect)

[PolarDB 非官方课程第二节--云原生架构与特有功能---答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507780&idx=1&sn=fc9fc9369122df94ac8560d1e25d7508&scene=21#wechat_redirect)

[PolarDB 非官方课程第一节-- 用户角度怎么看PolarDB --答题领奖品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507754&idx=1&sn=d826c160ad9cf484ad0875f936b8fe50&scene=21#wechat_redirect)

[免费PolarDB云原生课程，听课“争”礼品，重塑云上知识，提高专业能力](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507656&idx=1&sn=7d8fa3fcd7dcbe148a704bc99c21d656&scene=21#wechat_redirect)

PolarDB 相关文章

[P-MySQL SQL优化案例，反观MySQL不死没有天理](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507456&idx=1&sn=d54506242a13c848a317c838f0590ec1&scene=21#wechat_redirect)

[非“厂商广告”的PolarDB课程：用户共创的新式学习范本--7位同学获奖PolarDB学习之星](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508112&idx=1&sn=deec789105feaeec53619e07922ed4ad&scene=21#wechat_redirect)

[“当复杂的SQL不再需要特别的优化”，邪修研究PolarDB for PG 列式索引加速复杂SQL运行](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508246&idx=1&sn=adc41311be32b7398e7f09e61f2c93d2&scene=21#wechat_redirect)

[数据压缩60%让“PostgreSQL” SQL运行更快，这不科学呀？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508187&idx=1&sn=c0a9a08d2c5ff9190558717e21810f4e&scene=21#wechat_redirect)

[这个 PostgreSQL 让我有资本找老板要 鸡腿 鸭腿 !!](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508035&idx=1&sn=c2292d9214d10419472d73669b90829b&scene=21#wechat_redirect)

[用MySQL 分区表脑子有水！从实例，业务，开发角度分析 PolarDB 使用不会像MySQL那么Low](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507616&idx=1&sn=7e4cad8d43c61b0b6962d88e15425112&scene=21#wechat_redirect)

[P-MySQL SQL优化案例，反观MySQL不死没有天理](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507456&idx=1&sn=d54506242a13c848a317c838f0590ec1&scene=21#wechat_redirect)

[MySQL 和 PostgreSQL 可以一起快速发展，提供更多的功能？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507170&idx=1&sn=9bceb689dff839585b520deffc4bdaaf&scene=21#wechat_redirect)

[这个MySQL说“云上自建的MySQL”都是”小垃圾“](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506989&idx=1&sn=a8ac0192e1e3fc6b10bfd598d2d471cf&scene=21#wechat_redirect)

[PolarDB MySQL 加索引卡主的整体解决方案](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506825&idx=1&sn=f9417788f344bf4ee4ecb8f562065118&scene=21#wechat_redirect)

[“PostgreSQL” 高性能主从强一致读写分离，我行，你没戏！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506091&idx=1&sn=329706154485af3429e7d1fae570be4c&scene=21#wechat_redirect)

[PostgreSQL 的搅局者问世了，杀过来了！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505908&idx=1&sn=d28da91cd1b6f04a29954cda4d37d0ff&scene=21#wechat_redirect)

[在被厂商围剿的DBA 求生之路 --我是老油条](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505551&idx=1&sn=382ce106803d20c15a84f7d9b70d2c9d&scene=21#wechat_redirect)

[POLARDB  添加字段 “卡” 住---这锅Polar不背](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505381&idx=1&sn=70454f6deb9cce877a38da33e302609c&scene=21#wechat_redirect)

[PolarDB 版本差异分析--外人不知道的秘密（谁是绵羊，谁是怪兽）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505041&idx=1&sn=f3edeee2c084da985c570fbc5e55753d&scene=21#wechat_redirect)

**[在被厂商围剿的DBA 求生之路 --我是老油条](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505551&idx=1&sn=382ce106803d20c15a84f7d9b70d2c9d&scene=21#wechat_redirect)**

**[PolarDB 答题拿-- 飞刀总的书、同款卫衣、T恤，来自杭州的Package](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504694&idx=1&sn=e4a3ad34be3c481496cefc7542f0a783&scene=21#wechat_redirect)（活动结束了）**

**[PolarDB for MySQL 三大核心之一POLARFS 今天扒开它--- 嘛是火](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504554&idx=1&sn=2e7894eb5a43ebeea9c723f1cc4ea918&scene=21#wechat_redirect)**

PostgreSQL 相关文章

[PostgreSQL 新版本就一定好--由培训现象让我做的实验](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507400&idx=1&sn=793893b9508e8945b0ba81926753689d&scene=21#wechat_redirect)

 [说我PG Freezing Boom 讲的一般的那个同学，专帖给你，看看这次可满意](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508065&idx=1&sn=6214f09d19bf0fce05a0855d836ca564&scene=21#wechat_redirect)

[邦邦硬的PostgreSQL技术干货来了，怎么动态扩展PG内存 ！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508324&idx=1&sn=25eb586e999ecf11b32930c533795e22&scene=21#wechat_redirect)

[3种方式 PG大版本升级  接锅，背锅，不甩锅  以客户为中心做产品](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508413&idx=1&sn=893f96a4049e33a73e35e9f79e47f95b&scene=21#wechat_redirect)

["PostgreSQL" 不重启机器就能调整 shared buffer pool  的原理](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508412&idx=1&sn=c21f8c1f32b95605f294f5dffb227729&scene=21#wechat_redirect)

[说我PG Freezing Boom 讲的一般的那个同学专帖给你看这次可满意](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508065&idx=1&sn=6214f09d19bf0fce05a0855d836ca564&scene=21#wechat_redirect)

[一个IP地址访问两个PG实例，上演“一女嫁二夫”的戏码](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507805&idx=1&sn=2926ad8bde94a7a2ae3b6fa296ec35cc&scene=21#wechat_redirect)

[PostgreSQL  Hybrid能力岂非“小趴菜”数据库可比 ？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507691&idx=1&sn=1b5cd181b972b33df793c32ccd4fa591&scene=21#wechat_redirect)

[PostgreSQL 新版本就一定好--由培训现象让我做的实验](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507400&idx=1&sn=793893b9508e8945b0ba81926753689d&scene=21#wechat_redirect)

[PostgreSQL “乱弹” 从索引性能到开发优化](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507357&idx=1&sn=7cc368e4a9aa846a2ec2b3d97280a9d0&scene=21#wechat_redirect)

[PostgreSQL  无服务 Neon and Aurora 新技术下的新经济模式 （翻译）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506959&idx=1&sn=ef054f15cf4235d7a22bf8622a069fb6&scene=21#wechat_redirect)

[PostgreSQL的"犄角旮旯"的参数捋一捋](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506795&idx=1&sn=ca35ec663f875a5cf04692533655b880&scene=21#wechat_redirect)

[PostgreSQL逻辑复制槽功能](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506521&idx=1&sn=bc2546e2f432cc33ffcca893030c8b62&scene=21#wechat_redirect)

[PostgreSQL 扫盲贴 常用的监控分析脚本](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506123&idx=1&sn=ad2376559756f526e214eb5cd83cff96&scene=21#wechat_redirect)

[“PostgreSQL” 高性能主从强一致读写分离，我行，你没戏！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506091&idx=1&sn=329706154485af3429e7d1fae570be4c&scene=21#wechat_redirect)

[PostgreSQL  添加索引导致崩溃，参数调整需谨慎--文档未必完全覆盖场景](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505937&idx=1&sn=c95ebbb0d4b9b5fe1a58a45136899291&scene=21#wechat_redirect)

[PostgreSQL 的搅局者问世了，杀过来了！](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505908&idx=1&sn=d28da91cd1b6f04a29954cda4d37d0ff&scene=21#wechat_redirect)

[PostgreSQL SQL优化用兵法，优化后提高 140倍速度](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505891&idx=1&sn=1b7d4ab79e11347373ae64a8b81d2e31&scene=21#wechat_redirect)

[PostgreSQL 运维的难与“难”  --上海PG大会主题记录](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247505347&idx=1&sn=e17ae8668c945d11aa17251784fbf2f5&scene=21#wechat_redirect)

[PostgreSQL 什么都能存，什么都能塞 --- 你能成熟一点吗？](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504955&idx=1&sn=13bcc6846dc9fcdbf5c4bd37e74ea3a4&scene=21#wechat_redirect)

[PostgreSQL 迁移用户很简单 ---  我看你的好戏](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504718&idx=1&sn=66f9272251b07793995f36a35031d439&scene=21#wechat_redirect)

**[PostgreSQL 用户胡作非为只能受着 --- 警告他](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504630&idx=1&sn=cc84100b76b73807e93724613e245186&scene=21#wechat_redirect)**

**[全世界都在“搞” PostgreSQL ，从Oracle 得到一个“馊主意”开始](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504549&idx=1&sn=3c45668819c58013cd27e222c9ac57f5&scene=21#wechat_redirect)**  
[PostgreSQL 加索引系统OOM 怨我了--- 不怨你怨谁](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504279&idx=1&sn=ac2241b7a6b8438fb8cc78f0ecf5a549&scene=21#wechat_redirect)

[PostgreSQL “我怎么就连个数据库都不会建？” --- 你还真不会！](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504161&idx=1&sn=40aeb9862c234db87296a1a09f12aa9f&chksm=cfbcb57ef8cb3c683d9be4946363f49cf3db1a788d1eaf8f661fcdbfa02bc79dd4b267c61075&scene=21#wechat_redirect)

**病毒攻击PostgreSQL暴力破解系统，防范加固系统方案（内附分析日志脚本）**

[PostgreSQL 远程管理越来越简单，6个自动化脚本开胃菜](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503588&idx=1&sn=f5522951ab03c4052537cd31d136fe20&chksm=cfbcaabbf8cb23ad48a629eee232019754ea33f7b8477b32b7d1d2f855a73d2de37ea0ffd6a3&scene=21#wechat_redirect)

**[PostgreSQL 稳定性平台 PG中文社区大会--杭州来去匆匆](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247502555&idx=1&sn=b409c23a989b4d4a3d9c05da1b953087&chksm=cfbcae84f8cb279291f28747359a9c0aa0e7e7a68d0cfad59d86a53912b6ca621ed30082663e&scene=21#wechat_redirect)**

****[PostgreSQL 如何通过工具来分析PG 内存泄露](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247502002&idx=1&sn=3990632fd8d71d82d918254007e1e0d1&chksm=cfbcacedf8cb25fb0a7c8be54e86f1b53d9e374b1b7df1450edfa46bb26a385d1de88d1360c0&scene=21#wechat_redirect)****

**[PostgreSQL  分组查询可以不进行全表扫描吗？速度提高上千倍？](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247501661&idx=1&sn=eb21ee83728724cc90e906dee800b30e&chksm=cfbca302f8cb2a143850f8ab321311caad42d49687c85b924a9e291d30ac69da651d7822f798&scene=21#wechat_redirect)**

[POSTGRESQL --Austindatabaes 历年文章整理](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247498653&idx=1&sn=910442a2aaa9aa079258aef0a90ad82c&chksm=cfbc9fc2f8cb16d4d02b1c021a5add55d8ea681d6171a090472a4df8bb2dc77912ffdcbd8afc&scene=21#wechat_redirect)

[PostgreSQL  查询语句开发写不好是必然，不是PG的锅](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247500592&idx=1&sn=a4fbaf16d8ddab4bf51c9d5a3a67aaa6&chksm=cfbca76ff8cb2e7957ca7b517af1d70c4837b6f9dae1d6da185e5d7b4d8c5f5321c23069a9d4&scene=21#wechat_redirect)

[PostgreSQL  字符集乌龙导致数据查询排序的问题，与 MySQL 稳定 "PG不稳定"](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247500081&idx=1&sn=ab54447d7f6644386af7cb2b9ba997ee&chksm=cfbca56ef8cb2c787d05ee9f5b5e7037194b2492c0d4d9f8f67339c4e4ec9f1741c4e5e34151&scene=21#wechat_redirect)

[PostgreSQL  Patroni 3.0 新功能规划 2023年 纽约PG 大会 （音译）](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247499815&idx=1&sn=d5d177671188a72b5627db309d496f27&chksm=cfbca478f8cb2d6eee87bf1a61ec363a2ec9bf4c034c77e104c9cc868a011d102440c1255a28&scene=21#wechat_redirect)

[PostgreSQL   玩PG我们是认真的，vacuum 稳定性平台我们有了](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247501005&idx=1&sn=7acd5efd357db451c8b6e1090da4ff22&chksm=cfbca092f8cb29846c95c5d01fffa9d8b97625d2b0d564777bccecea12707a7b88f8e652f9bd&scene=21#wechat_redirect)

**[PostgreSQL DBA硬扛 垃圾 “开发”，“架构师”，滥用PG 你们滚出 ！（附送定期清理连接脚本）](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503516&idx=1&sn=5e04a8b2a3a0ba2a8109f5711fced74d&chksm=cfbcaac3f8cb23d57bd891fac7f746b7bb400b7999cad3c7b90709bc14df8b230b3d07f34ce7&scene=21#wechat_redirect)**

**[DBA 失职导致 PostgreSQL 日志疯涨](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503143&idx=1&sn=7348d8413dd94f95d7a21ea1bc262672&chksm=cfbca978f8cb206ea5e223dcd5fc8671b91e934e6e8a5dccea23ee69c9620ed07c93bc0bc6b6&scene=21#wechat_redirect)**

     [这个 PostgreSQL 让我有资本找老板要 鸡腿 鸭腿 !!](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508035&idx=1&sn=c2292d9214d10419472d73669b90829b&scene=21#wechat_redirect)

[一个IP地址访问两个PG实例，上演“一女嫁二夫”的戏码](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507805&idx=1&sn=2926ad8bde94a7a2ae3b6fa296ec35cc&scene=21#wechat_redirect)

[PostgreSQL “乱弹” 从索引性能到开发优化](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507357&idx=1&sn=7cc368e4a9aa846a2ec2b3d97280a9d0&scene=21#wechat_redirect)

MySQL相关文章

[一篇为MySQL用户，分析版本核心差异的文章--8.028-8.4的差异](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508851&idx=1&sn=2ba1f6bd02b1e53cd06d0e9e28bc6552&scene=21#wechat_redirect)

[那个MySQL大事务比你稳定，主从延迟低，为什么？ Look my eyes! 因为宋利兵宋老师](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508163&idx=1&sn=b7388e2cfffe5cce5c2c8bbcdb157931&scene=21#wechat_redirect)

[MySQL 条件下推与排序优化实例--MySQL8.035](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507441&idx=1&sn=c4580f6d5a02f42a825735875085f775&scene=21#wechat_redirect)

[青春的记忆，MySQL 30年感谢有你，再见！（译）](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507307&idx=1&sn=c6b4bffd03256dd195b274969f662ef1&scene=21#wechat_redirect)

[MySQL 8 SQL 优化两则 ---常见问题](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507263&idx=1&sn=0b786426b75299b8f82f70bd5e71f358&scene=21#wechat_redirect)

[MySQL SQL优化快速定位案例 与 优化思维导图](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247506034&idx=1&sn=7222cf5c1f47f9e74dddad7380684286&scene=21#wechat_redirect)

["DBA 是个der" 吵出MySQL主键问题多种解决方案](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503624&idx=1&sn=3b5268b1db12dd8fa9b50d1a35a52eb6&scene=21#wechat_redirect)

**[MySQL 怎么让自己更高级---从内存表说到了开发方式](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247504692&idx=1&sn=9940db50f37d33b023ad9e4c793f92bc&scene=21#wechat_redirect)**

**[MySQL timeout 参数可以让事务不完全回滚](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247503767&idx=1&sn=a5f8f43c4f35283d7aeec939e7d4b533&chksm=cfbcabc8f8cb22decf18cc87d9ae43afbf7e1e4db75a85bb6026f36d6b7fa93c0c128065b69f&scene=21#wechat_redirect)**

[MySQL 让你还用5.7 出事了吧，用着用着5.7崩了](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247502556&idx=1&sn=a882f89046f7d5d3458fddceab3f7711&chksm=cfbcae83f8cb279513fd671a644171997145082a1d9b4acaf1370e0ffb741ba46c27dc753388&scene=21#wechat_redirect)

**[MySQL 的SQL引擎很差吗？由一个同学提出问题引出的实验](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247501960&idx=1&sn=9aa241509f48a69c5f5021ec96623a17&chksm=cfbcacd7f8cb25c1a69e06329d4c4c985a35cbd6811c709c29c38f690ef7125d254a0dfb9cfe&scene=21#wechat_redirect)**

用MySql不是MySQL, 不用MySQL都是MySQL 横批 哼哼哈哈啊啊

[MYSQL  --Austindatabases 历年文章合集](http://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247498683&idx=1&sn=68ba92431bba682aeee335b7d76a8c00&chksm=cfbc9fe4f8cb16f2534a99b71564c9d6e1f84fef3fb4be9ea1a6ec9d4d53aa0ec53bfd0b6c7e&scene=21#wechat_redirect)

[超强外挂让MySQL再次兴盛，国内神秘组织拯救MySQL行动](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247508406&idx=1&sn=3d3fde331f02505a7b784e6d4138011c&scene=21#wechat_redirect)

[MySQL 条件下推与排序优化实例--MySQL8.035](https://mp.weixin.qq.com/s?__biz=Mzg4NDA0NTEwNA==&mid=2247507441&idx=1&sn=c4580f6d5a02f42a825735875085f775&scene=21#wechat_redirect)
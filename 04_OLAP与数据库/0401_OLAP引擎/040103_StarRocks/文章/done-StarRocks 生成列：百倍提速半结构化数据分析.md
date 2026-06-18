> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocksSQL指纹与生成列优化边界|StarRocksSQL指纹与生成列优化边界]]
---
title: StarRocks 生成列：百倍提速半结构化数据分析
author: StarRocks
date:
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492710&idx=1&sn=5e739095775b08de4de6746f7b685494&chksm=e9f29b42de8512549f605eb1f5dee7073acc7c6afbcaa0b9f68a074b56986e41bad5e0af8863&mpshare=1&scene=24&srcid=0117tR6Y6K5fhkrU7L9b93Er&sharer_shareinfo=ed685ce5c3eae5f92d83c3441c7a86be&sharer_shareinfo_first=ed685ce5c3eae5f92d83c3441c7a86be#rd
---

半结构化分析主要是指对 MAP，STRUCT，JSON，ARRAY 等复杂数据类型的查询分析。这些数据类型表达能力强，因此被广泛应用到 OLAP 分析的各种场景中，但由于其实现的复杂性，对这些复杂类型分析将会比一般简单类型要更困难和耗时，例如：

* 需要对 MAP，STRUCT，JSON 等数据类型中的某个字段进行查询分析。由于这些复杂类型会被存储为一个整体，因此需要先将整个半结构化类型的字段先从存储层读取上来，然后再对其中的某些字段进行分析，I/O 效率较低。
* 对复杂类型进行较为耗时的分析计算(聚合，排序等等)，查询的实时 CPU 开销可能也是一个不可忽略的性能影响因素。

面对上述挑战，StarRocks 在 3.1 版本正式推出**生成列(Generated Column)**特性，提供一种透明加速的解决方案，能有效提升半结构化数据的分析效率，令用户拥有更极速的分析体验。

## **生成列介绍**

生成列是一种特殊的列，可以在建表语句或 Schema Change 语句中指定，生成列绑定到一个标量表达式上，当数据导入时，会自动根据表达式定义进行计算，并且将其计算结果写入到生成列中。

在半结构化分析的场景中，可以将复杂耗时的标量表达式绑定在某个生成列上，在数据导入阶段提前将结果计算好并且持久化到磁盘中。当需要进行查询分析时，即可马上获得表达式计算的结果。

## **生成列的查询改写**

当希望查询生成列保存的表达式计算结果时，可以直接在 SQL 中指定生成列的列名，但是这种方法意味着需要调整已有业务 SQL，很难完全做到无缝对接。

为了进一步提升功能的使用体验，简化使用流程，StarRocks 支持生成列自动查询改写。在生成执行计划时，SQL 优化器将会检查 SQL 中所有的表达式，并且将那些已经绑定到生成列上的表达式，改写成查询生成列列值。

例如，上述例子中，如果在某个查询中需要获取 `colc`中的 `a`字段，则执行查询`SELECT get_json_string(json_string(tbl.colc), '$.a') FROM tbl`，执行过程大致如下：

可见，优化器自动将表达式改写为查询生成列的值，实现透明加速。

## **高效的生成列加列**

在实际应用生成列的使用场景中，在已有的表添加生成列可能是一个高频操作。例如，可能在任意时间点发现某个表达式计算存在性能瓶颈，因此希望添加生成列以进行查询加速。

StarRocks 支持高效的加列操作，对于添加普通列，存储引擎并不会真正重写物理文件，而只是将物理文件重新 link 到新 Tablet 的路径下，修改元数据，完成加列操作。但是，如 `MODIFY COLUMN`这类 Schema Change 操作，由于需要改变存量数据的内容，因此会重写所有物理文件。类似地，对于生成列加列来说，由于需要存储新增的生成列表达式的计算结果，重写数据似乎也是不可避免的。但是，如果仍然采用全量重写物理文件的方案，将无法很好适应频繁加列的场景，加列的代价太大。

为了进一步提高生成列加列的效率，StarRocks 针对生成列加列进行了专门的优化。当添加一个生成列时，不会改写存量的物理文件，而是为每一个存量的 segment 生成一个只包含生成列值的 cols 文件(物理格式和 segment 文件一样，但只包含生成列一列数据)，当需要查询这些存量数据时，StarRocks 会自动将 segment 和 cols 文件的内容进行合并，获得正确的查询结果。

总的来说，**生成列加列优化后，读 I/O 只涉及到生成列表达式的引用列，写 I/O 只涉及到生成列本身的表达式结果，整个 Schema Change 的 I/O 效率相比完全重写有大幅提高**，更好支持实时动态生成列加列的用户需求。

## **效果验证**

为了更好验证生成列对半结构化分析的加速效果，我们进行了简单的测试验证。

集群信息：StarRocks v3.1 1FE1BE ，104C376GB

创建一张如下的数据表，

```
```
CREATE TABLE `t` (
  `id` bigint(20) NOT NULL COMMENT "",
  `array_int` ARRAY<int(11)> NOT NULL COMMENT "",
  `json_data` json NOT NULL COMMENT "",
  `gc_1` double NULL AS array_avg(`test`.`t`.`array_int`) COMMENT "",
  `gc_2` ARRAY<int(11)> NULL AS array_sort(`test`.`t`.`array_int`) COMMENT "",
  `gc_3` varchar(65533) NULL AS get_json_string(json_string(`test`.`t`.`json_data`), '$.a') COMMENT ""
) ENGINE=OLAP 
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`) BUCKETS 48 
PROPERTIES (
"replication_num" = "1",
"in_memory" = "false",
"storage_format" = "DEFAULT",
"enable_persistent_index" = "false",
"replicated_storage" = "true",
"compression" = "LZ4"
)
```
```

普通列数据创建方式：

1. id，作为 primary key 列保证唯一。
2. array\_int，长度为 10000 的 ARRAY<int>，保存的都是随机数。
3. json\_data，包含两个 key，key "a" 对应的 value 为整型 1，key "b" 对应的value 是长度为 100 个 uuid 构成的字符串

性能测试使用下面的 query：

```
q1:SELECT get_json_string(json_string(json_data), '$.a') FROM A
q2:SELECT array_avg(array_int) FROM A;
```

测试结果：

从上述的测试结果可知：

* q1：使用生成列提取大 JSON 字段中的某个子字段，在查询阶段大幅节省了读取 JSON 字段的 I/O 消耗，查询性能提升达 4 倍以上。
* q2：使用生成列对大 ARRAY 字段进行聚合计算(计算平均值)，**在查询阶段不仅节省读取该半结构化数据字段的 I/O 消耗，同时也大幅节省了 ARRAY 聚合计算所带来的 CPU 消耗，获得百倍的性能提升****。**

总结

生成列功能是一种加速半结构化分析的有效手段，当面对复杂的半结构化表达式计算时，可以为其添加对应的生成列，在导入阶段自动完成表达式计算，并将结果持久化。在查询阶段通过优化器的自动改写，直接从生成列中获得表达式计算结果，避免实时的表达式计算，实现透明加速。

通过使用生成列，用户能大幅减少查询时复杂表达式的 I/O，CPU 等资源消耗，在不同的场景下获得数倍甚至百倍的性能提升。

**关于 StarRocks**

Linux 基金会项目 StarRocks 是数据分析新范式的开创者、新标准的领导者。面世三年来，StarRocks 一直专注打造世界顶级的新一代极速全场景 MPP 数据库，帮助企业构建极速统一的湖仓分析新范式，是实现数字化转型和降本增效的关键基础设施。

StarRocks 持续突破既有框架，以技术创新全面驱动用户业务发展。当前全球超过 330 家市值 70 亿元以上的头部企业都在基于 StarRocks 构建新一代数据分析能力，包括腾讯、携程、平安银行、中原银行、中信建投、招商证券、大润发、百草味、顺丰、京东物流、TCL、OPPO 等，并与全球云计算领导者亚马逊云、阿里云、腾讯云等达成战略合作伙伴。

拥抱开源，StarRocks 全球开源社区飞速成长。目前，已有超过 300 位贡献者，社群用户近万人，吸引几十家国内外行业头部企业参与共建。项目在 GitHub 星数已超 7100 个，成为年度开源热力值增速第一的项目，市场渗透率跻身中国前十名。

**金融：**[中信建投](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&chksm=e9f16af6de86e3e0bc992f098a8d0787a18ff036bc70ea2d11f7d7e08358af16224f52d237a0&scene=21#wechat_redirect)｜[中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect) | 平安银行 | 中欧财富

**互联网：**[微信｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492524&idx=1&sn=7cbd70eb86f606ea58d293a9430e7047&chksm=e9f29c88de85159ed3245a906475ab9bdd32d93478d76379ff79dcd344383aec25f3df4d7a0e&scene=21#wechat_redirect)[小红书｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492619&idx=1&sn=42698abb77f05f6a317ad278ffaca257&chksm=e9f29b2fde851239e5553ac00d7e1227dd8346eabc3848e9e35a2ddfbf152dc8376ade2c9e79&scene=21#wechat_redirect)[网易邮箱｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)[滴滴｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)[美团餐饮SaaS |](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)[B站｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&chksm=e9f29dcade8514dcfbfb19d06dc75659ff7c5d9d72ec10210eceb5220d9706b56b61d80dd9d8&scene=21#wechat_redirect)[携程 |](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491637&idx=1&sn=daec09546a550346b65414567dc8db7b&chksm=e9f29f11de851607eb14a9e006803d8f52951d367f19a2477d5abee92f309c8c8145655a959c&scene=21#wechat_redirect)[同程旅行｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[360｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)[58同城｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489116&idx=1&sn=ea84a147b3bc196ae125fbd40f16c06b&chksm=e9f16978de86e06efa4456c06971827d3ab266de3c3628004eabb712435d5bbb80b381231c47&scene=21#wechat_redirect)[芒果TV｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491190&idx=1&sn=42c67a896174b68e4c66c955ad72c81e&chksm=e9f16152de86e84403fdc9bbe14e3ed49dc743b9243ab1067af4731f1d9038b1402992046ba8&scene=21#wechat_redirect)[得物](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect) ｜[贝壳｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect)[汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)

**游戏：**[腾讯游戏｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[波克城市｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[欢聚集团｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect) | [游族网络](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)

**新经济：**[蔚来汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)[理想汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)[顺丰｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)[京东物流](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)｜[跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect) | [大润发｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)[华润万家｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)[TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) ｜[万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect) [| 百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect) [| 多点 DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect) [|](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)[酷开科技](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&chksm=e9f17250de86fb4630b4c3f6d3463c06daa9941d42b8c8cf311e91c0817f403de2d138d3884e&scene=21#wechat_redirect)

**StarRocks 技术内幕：**[极速湖仓神器：物化视图](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490902&idx=1&sn=0bd03778216286548f5d02a2df33466a&chksm=e9f16272de86eb645470a514afb35d6618d805d5358b7afa7b57f897c81ce2902e01d446064d&scene=21#wechat_redirect)｜[存算分离，兼顾降本与增效](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490146&idx=1&sn=771199c6e343612031134c0285fa4881&chksm=e9f16546de86ec50918f2c2ea3cc2eeac8d19655177fab2e9db933f5eb216fe30f2751c108b0&scene=21#wechat_redirect)  ｜[实时更新与极速查询如何兼得](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485927&idx=1&sn=56051046f9eb51a563c6c24eb22c9364&chksm=e9f176c3de86ffd5bab12e9b8657ad58c2b4b4d2741dc0f7de292c800e4ec23cb064955013d0&scene=21#wechat_redirect)｜[Query Cache，一招搞定高并发](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490921&idx=1&sn=9fc54c02d5395e112d0021686fb1bf6c&chksm=e9f1624dde86eb5be9a97895e1c92c40977b3943af6ad8b99611706092146e0690a3bf2e083d&scene=21#wechat_redirect)｜[资源隔离](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489066&idx=1&sn=4317418e82af2520ad5c3dd1c3ad5975&chksm=e9f1690ede86e018406d8f5b750a46cafcce21d263ae9794c23c85cc7b35d292053b523cde47&scene=21#wechat_redirect)｜[大数据自动管理](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485727&idx=1&sn=548ed6a15bd4b938cae8551500e3275d&chksm=e9f1763bde86ff2dcea5aacf2e1d8b9401cee6236b41571e241de74178cd856842e4e62fb48a&scene=21#wechat_redirect)｜[查询原理浅析](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485848&idx=1&sn=90e47d7a46eb120701d28b5acfbcc401&chksm=e9f176bcde86ffaaa0c4a3b686eea5b053ff286164cb9a27e85daf4e42bec08985f5a41975c3&scene=21#wechat_redirect)
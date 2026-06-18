> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC版本演进与Pipeline连接器边界|FlinkCDC版本演进与Pipeline连接器边界]]
---
title: Flink CDC 3.5.0 Source，我特么...
author: 安瑞哥是码农
date:
url: https://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247490332&idx=1&sn=ba790d705a834b2b4dc2f0f1151f11fb&chksm=e86b2c86700755f8e1fcca76e44d019d4f8c56f91999ac636a1d730c05d06c90c5efc64d9df9&mpshare=1&scene=24&srcid=1022Ur1eVOes6VnqNxt5TJOA&sharer_shareinfo=54834267892167de7aa3ca11d8a7f700&sharer_shareinfo_first=54834267892167de7aa3ca11d8a7f700#rd
---

上篇文章实测了 Flink CDC 3.5.0 pipeline 的一些基础功能，从结论来看，老的问题依然存在——没法通过正规途径指定 checkpoint。

但同时也解决了被大家骂了很久的，「日期字段类型」识别转换错误的问题，属于喜忧参半(严格来说是喜少悲多)。

这一次，咱再来看看，一直被我寄予厚望的 Flink CDC 3.5.0 Source，表现怎么样？

0. 升级开发环境

对于 Flink CDC Source 来说，所谓的升级，就是把开发环境里面的 jar 包，给换成最新的 3.5.0。

因为我准备测试的数据源有 MySQL 跟 Postgresql(官方这次升级的重点包含了这两个，验证一下有没有打脸)，所以这里就升级这两个 jar 包。

MySQL CDC 依赖 jar

PG CDC 依赖 jar

好，升级完成。

但接下来，神奇的事情，它发生了。

1. 有多神奇呢

还记得上篇测它的 pipeline 功能时，是已经确定了当前的 3.5 版本，跟它官方公众号描述的一样，已经修复了对上游数据源「日期类型」的支持的。

[Flink CDC 3.5 pipeline，骂还是夸呢？](https://mp.weixin.qq.com/s?__biz=MzI0OTEwNzQyNA==&mid=2247490307&idx=1&sn=ddb9038bf56fc12bd9eda65ea8a6a656&scene=21#wechat_redirect)

要不说人家措辞就是严谨呢，你知道为什么这里的标题，说的是「pipeline」吗？

因为相同版本的 Flink CDC Source，压根就没有解决对这个「日期类型」的支持，而且是 MySQL 跟 PG 都不支持。

也就是说，同为 Flink CDC 3.5.0，只有 pipeline 解决了这个问题，而对于 CDC Source，抱一丝——老子就是不改！

所以我上文说到他们是便秘似的改进，一点不为过吧。

来，先看 MySQL 的数据源。

这是其中一张的表结构：

这是其中的一条测试数据：

这是 Flink CDC Source 读到之后的样子：

再来看 PG 的数据源。

算了，不看了，一个鸟样......

一开始我还以为是 jar 包版本修改之后，没有生效导致的，谁知道我把整个项目都重新编译了，还是 TM 不行。

就问你生不生气？

2. 正常的增删改

虽然在它的 pipeline 功能里，对于数据源后续的「增删改」识别功能翻车了，但因为 CDC Source 可以通过编码，正常的指定程序的 checkpoint 目录，所以对于 Flink CDC 应该具备的这些「再正常不过的功能」，这次测试是通过的。

对于后续的 insert：能正常识别；

对于后续的 update：能正常识别；

对于后续的 delete：也能正常识别。

但这些，不足为道，这就好比你买了一辆汽车，能开走、能加速、能刹车，但你不会夸它优秀一样。

3. 新增字段跟新增表的表现

先说结论——跟上个版本 3.4.0 的表现，一个鸟样。

3.1 MySQL

先说新增字段：

1. 能识别前后变化的值；

2. 对于后续的 insert、update、delete 也能准确的识别出来。

表现跟上个版本一样，符合预期。

再说新增表：

也跟上个版本一样，只有一个新增表的「信息提示」，对于后续对新增的表的 insert、update、delete，通通不认识。

表现也跟上个 Flink CDC 版本一毛一样。

3.2 PG

先说新增字段：

1. 对于新增字段后，后续的 insert、update、delete 能准确的识别出来，但是有 bug，比如如果是 update，那么 before 的值，永远都是空的；

2. 不能识别出数据变更前后的变化。

表现不如 MySQL，跟上一个 3.4 版本也一样废。

再说新增表：

完 全 没 有 反 应 ！

也跟上个版本，表现得出奇一致。

最后

虽然官方对这次 3.5.0 版本的优化吧啦吧啦说了一堆，但从实测表现来看，Emm...... 你们自己体会。

如果说 Flink CDC 3.5.0 pipeline 还能给你带来那么一小丝激动，那么 Flink CDC 3.5.0 Source，就纯碎是为了恶心你而生的。

散了吧......

---

你可以添加我的私人微信入讨论群，跟一群热爱技术的小伙伴一起成长...
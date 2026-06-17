---
title: DBA 运维工具推荐——数据库一致性校验修复
author: 不会弹琴的猫
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU2NzY1MDgyMA==&mid=2247483923&idx=1&sn=8b2c0c9f8dd196c5360ea8e1f92fec2f&chksm=fc98bd78cbef346ec6d11082140af32282a29a95503540bafd3bd1da0040f9441ae3253901f6&mpshare=1&scene=24&srcid=1229Qft1PDzsKvkD6l1XSF79&sharer_shareinfo=7f7266ecf7bb81d59ad508982217102b&sharer_shareinfo_first=7f7266ecf7bb81d59ad508982217102b#rd
---

今年早些时候，万里数据库开源了一款数据库静态修复工具，由于项目本身是 Go 语言实现，并且支持异构数据库（工具本身只支持 MySQL 和 Oracle，但基本框架和可扩展性还不错），在万里社区微信群里叶老板的推介下，我这个初级 Coder 果断关注学习了起来。

关于项目的介绍，我这里不作赘述，大家可移步项目地址了解：https://gitee.com/GreatSQL/gt-checksum

更深入的解析，也可以参考我之前的文章：[开着飞机换引擎：工行 DBA 专家是如何解决数据库一致性校验的](https://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651733413&idx=1&sn=2c73d1cdfeb85bbcaf161ac3d76d82b4&scene=21#wechat_redirect)

---

当然，今天的重点是介绍一下我在原项目基础上进行的功能增强，包括以下几点：

* 实现对 openGauss 的支持（仅支持数据、结构和索引等对象）
* 优化时间戳类型校验精度到微秒（6 位小数）
* 优化＜1 的浮点数校验处理
* 优化对象名大小写参数处理逻辑
* 以及，**新增数据比对时忽略指定列的能力**

个人项目分支：https://gitee.com/mystuart/gt-checksum/tree/dev/

欢迎大家拍砖！

由于我对 Git 使用还很不熟练，再加上部分流程逻辑改动较大，所以暂时还没给原项目提 PR（主要作为新手 Coder 怕给原项目维护者搞出麻烦）。

最后还是要致谢一下 ChatGPT，实乃家居旅行搬砖之必备良药！
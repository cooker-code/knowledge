> 已吸收至：[[04_OLAP与数据库/0408_hbase/040801_HBase/040801_核心知识点/HBase写入链路MemStoreFlush与HFile边界|HBase写入链路MemStoreFlush与HFile边界]]
---
title: V6.0新增面试题-HBase为什么有列族
author: 三石大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485885&idx=1&sn=7fc6832d60bfbea84c9a127bfdaf9f45&chksm=fba5bcaa6952020e188793136b3923ffc6e5859ff3ab5a41885a1316fbc8c3ddce098c484af5&mpshare=1&scene=24&srcid=1105KKRh3pBtemNrVAcew4Uy&sharer_shareinfo=49f793bab6985499a49739bae9993635&sharer_shareinfo_first=49f793bab6985499a49739bae9993635#rd
---

# 推荐阅读文章列表

[2024最新大数据开发面试笔记V6.0——试读](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247485272&idx=1&sn=6178cf811f80a0ba0a0091ca930a2d07&chksm=fa0890bdcd7f19ab9a0de619d0928c0a42ffe11edf7e247fb3f2ee53da809068088982db968c&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[我的大数据学习之路](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483862&idx=1&sn=963e528abadb77e16ee4c320f19d028c&chksm=fa089633cd7f1f251758769e5b28189dae45a36d22cab6fc7a8666dd6c7d0182f596c5c10a1f&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

[面试聊数仓第一季](https://mp.weixin.qq.com/s?__biz=MzUyNjc2MjYzNA==&mid=2247483942&idx=1&sn=afade2e444dec602001790fd6b3c53f3&chksm=fa0895c3cd7f1cd5ac4c95db849ca98cda5b11d07945ef3ac7dae20aa8e70cbaeaf3ac4286ef&token=208056502&lang=zh_CN&scene=21#wechat_redirect)

# 面试问题

> 来自美团数据研发二面（社招）

* **问题：HBase为什么有列族？**

# 问题分析

我们需要回答什么？我们可以从两个方面来思考，**第一就是 列族的定义是什么，第二就是 列族的作用是什么；**

当你能够回答出上述两点后，面试官就不会再有任何疑问！

# 答案草稿

* 首先我说一下列族是什么哈，列族是HBase中数据模型的一部分，**它是一组相关列的集合**
* 然后我再具体说一下有了列族能够带来哪些好处，第一、列族可以帮助我们更好地**组织和管理数据**；第二、我们在查询某列的时候，可以查询对应的列族下的某列，而不需要读取所有列，**提高查询效率**

# 写在最后

### V6.0笔记获取方式

> 公众号回复：大数据面试笔记
---
title: OceanBase发布会上硬刚PG+DuckDB，国产品牌崛起
author: digoal德哥
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247500665&idx=1&sn=1458762b7576e4667a48e7b568431cfa&chksm=9134190b6758a7b0c773b8c91fca8242646cdbde4d2a37c80fed24cb8734edb847092a9b3141&mpshare=1&scene=24&srcid=1119zZG7pCA82AzUmfukXTnW&sharer_shareinfo=8f20ce90f7b57afdf15dbfea17ffdaab&sharer_shareinfo_first=8f20ce90f7b57afdf15dbfea17ffdaab#rd
---

OceanBase发布会大家看了吗? 铺天盖地的新闻稿我就没兴趣发了。

但是站在一个从业者的角度，我必须得说两句！个人认为他们发布会最惊艳的是开源的AI原生数据库 SeekDB , 为什么呢?

这个SeekDB数据库可谓是集成了AI应用搜索所需的所有功能, 简直是七个葫芦兄弟合体的精钢葫芦娃.

先来看看需求，咱就不用那个四看三定五明确的菊花基本法了，直接上干货:

* [《深度研究报告 | AI 智能体所需的数据库产品及未来发展趋势》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247497701&idx=1&sn=347fc40ffb29a32786a2105f3909cc5e&scene=21#wechat_redirect)
* [《AI与数据库的共生融合：能力、创新与未来范式》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247497188&idx=1&sn=371a5cb0d79e5800b2585230116197ce&scene=21#wechat_redirect)
* [《AI搜索“不等于”向量搜索, 那么答案究竟是什么?》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247497364&idx=1&sn=43eba839a70dd6c757d33164d4a9168d&scene=21#wechat_redirect)

AI 搜索不等于向量搜索, 因为AI 搜索需要的是相关性结果, 而不仅仅是向量相似结果.

AI 搜索实际上是个多模态的需求, 既有传统的全文检索, 又有精确搜索, 还有语义搜索(向量搜索), 甚至也有模糊查询、GIS地理信息搜索、复杂分析查询、标签匹配(多值列的搜索)、文档搜索(JSON类型的搜索)、图式搜索等等.

目前有且仅有一款开源数据库 PostgreSQL 支持各种模态的快速检索(支持各种模态的类型、索引等). 列举一些多模态插件如下

* pg\_jieba, 中文分词
* pg\_trgm, 模糊查询
* pg\_bigm, 模糊查询
* vectorchord-bm25, 分词且支持关键词BM25权重
* pg\_tokenizer, 基于模型的文本转嵌入
* pgvector, 向量
* vectorchord, 支持图、倒排、多向量检索, 高级功能支持预过滤、量化压缩、标签过滤、混合查询等
* pgvectorscale, 基于DiskANN的向量搜索
* postgis, GIS地理信息
* age/GraphBLAS, 图式搜索
* pg\_duckdb, 基于duckdb计算引擎的计算加速、冷热分离等
* pg\_mooncake, 基于duckdb计算引擎的计算加速、冷热分离等
* pg\_lake, 基于duckdb计算引擎的计算加速、冷热分离等
* pg\_onnx, 接入开放神经网络集市, load已有模型, 让数据库快速具备推理能
* PostgresML, 模型集市+向量数据库+自定义模型

AI搜索相关的多模态 PG 插件实在太多, 无法穷举.  如果你对向量搜索有兴趣, 列举几篇文章仅供学习交流

* 《VectorChord-BM25：通过 BM25 排名彻底提升 PostgreSQL 搜索性能和效果 — 甚至比 ElasticSearch 快 2.26 倍》
* [《超越文本: 使用 Modal 和 PostgreSQL+VectorChord 解锁无 OCR 的 RAG, 无惧PDF、扫描文档等》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247498474&idx=1&sn=da029184f3b2d3d33225b787d535a45d&scene=21#wechat_redirect)
* 《VectorChord新版本发布: 支持图(DiskANN和HNSW)索引、召回率评估》
* 《召回精度提升插件: pg\_tokenizer + VectorChord-BM25 reranking》
* [《DeepSeek 最新 "OCR-上下文光学压缩" 论文的重大意义 : 向量数据库的筒子们又有活干了》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247498840&idx=1&sn=3216f55dca5f7b591d7340b5119bf9f5&scene=21#wechat_redirect)

针对向量搜索的索引设计和优化, 我这有一篇专门的文章如下:

* [《德说-第362期, 读了500篇论文和数个顶级向量数据库项目后, 面向AI 搜索场景的数据库索引设计和优化思考》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247500509&idx=1&sn=895a4611d223e4e33466982fd6fa3635&scene=21#wechat_redirect)

SeekDB瞄准的就是AI搜索需要的多模数据库!

你可能会说, 多模态的数据库多了, 所有兼容PG的国产数据库都是啊, 例如 PolarDB , 有什么稀奇? 

说起PolarDB必须提一下他们家正在搞的国赛第二季，属于A类竞赛，大学生可关注其公众号

继续。确实, 国产向量数据库很多, 例如openGauss生态的vexdb, vastbase 100. milvus. eloq等都是啊

* [《谁说国产只会套壳, 这个国产向量数据库就不是pgvector套壳》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247498106&idx=1&sn=6d9b7423a20534f8d545e24e15a6e278&scene=21#wechat_redirect)

但是, AI 其实除了多模态, 还有一个特点, 它既有自有记忆、也有集体记忆. 

如果一个AI的自有记忆存储在云端, 它会怎么样? 连不上数据库就完了! 妥妥的阿兹海默症患者.

如果本地记忆满了怎么办？

所以AI 既需要本地记忆数据库, 也需要云端的共享记忆数据库.

本地数据库?

没错本地数据库, 要的是轻便, 我在前面的《深度研究报告》已经分析过, 这里不再赘述.

本地数据库当然不能太重, 最好是没有服务, 嵌入应用, 随时拉起, 随时使用.

这不就是嵌入数据库么? 没错, 所以除了PG, 我前面没说的还有一个开源数据库, 也支持多模态, 那就是DuckDB.

我甚至之前有断言在AI应用这个赛道, DuckDB和PG必有一战, 文章如下:

* [《DuckDB进军AI数据库底座, 坐实了 | 文本语义(向量)搜索、全文检索、bm25搜索与排序、模糊搜索详解》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247496223&idx=1&sn=512d70f3ad436f3ef4b825b6b66fa365&scene=21#wechat_redirect)
* [《祝贺DuckDB斩获30K star, Roadmap暴露野心, 估值可能过百亿》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247495978&idx=1&sn=04c8e7b90a39eb543a5df69303071293&scene=21#wechat_redirect)

因为DuckDB是嵌入式数据库, 所以DuckDB打的是边侧(也就是AI末端)的搜索数据库, 而全局知识库更需要可弹性伸缩的、分布式的数据库, 不过你可能想不到, 其实云端DuckDB也有布局.

* [《DuckDB 在做云边一体》](https://mp.weixin.qq.com/s?__biz=MzA5MTM4MzY1Mw==&mid=2247496574&idx=1&sn=1fb39535e85d0866a862a5ef34c55b76&scene=21#wechat_redirect)

但是, 上面都不重要了, 重要的是SeekDB, 它居然这么有前瞻性, 居然同时发布了嵌入式和CS两种产品形态, 而且它支持多模态.

这不就是为AI原生打造的数据库么! 正面硬刚DuckDB+PG啊! 

嵌入式让它能快速上手, 适合开发者快速部署开发AI应用(轻量化, 让产品的上手门槛直接变成0), 适合作为边缘端AI应用的数据存储. CS结构则适合更大的集中式AI数据存储.

SeekDB 可能还有一个潜在优势, 它兼容MySQL, MySQL用户群的体量在国内绝对是首屈一指的, 虽然他们可能只会crud, 但这恰恰对SeekDB来说是好处. MySQL开发者真的应该好好感谢它, 你不用学其他数据库例如PG, 就可以享受AI 数据库能力了!

不得不佩服他们的产品经理, 这个产品有两下子! 

接下来压力交给运营了, 看看能不能引爆吧! 

感兴趣可关注:

https://www.oceanbase.ai/docs/

https://github.com/oceanbase/seekdb

最后, 我没提pymilvus(milvus嵌入式版)? 对不起我个人觉得milvus可能是纯向量数据库, 不算多模态吧? 也许未来会支持更多模态, 希望更多的技术层面的充分竞争, 国产数据库的技术氛围越来越好!

此次OceanBase发布的SeekDB是数据库产品形态的一次里程碑之作，未来数据库可能都会以“嵌入式+CS”合体架构形态出现，你怎么看？欢迎留言讨论！
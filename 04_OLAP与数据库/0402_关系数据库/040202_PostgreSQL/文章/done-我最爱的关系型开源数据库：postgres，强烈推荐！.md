> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQL与MySQL选型边界|PostgreSQL与MySQL选型边界]]
---
title: 我最爱的关系型开源数据库：postgres，强烈推荐！
author: 猫咪不吃愚
date:
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY1NzE0NQ==&mid=2247486608&idx=1&sn=d3904bfba10318daa50a192d466631f4&chksm=c32100864922f30c2dbb77463ec33c3b67e22a3a36d9671e40538b4edb40d707f4b166121aed&mpshare=1&scene=24&srcid=1012Dw7y0rCCkLCrmA1O0yAf&sharer_shareinfo=c771af110b850ababc9a70fc8aa4808f&sharer_shareinfo_first=c771af110b850ababc9a70fc8aa4808f#rd
---

> 字数 1843，阅读大约需 10 分钟

数据库是后端开发的必备工具，PG是关系数据库的最佳开源方案，强烈推荐大家学习和在工作中逐步深入了解和实践，如果感兴趣，阅读这篇文章入门即可❤️，相信大家都有mysql的经验

---

【BibiGPT】AI 一键总结：【双语+纯享】PostgreSQL：我最爱的操作系统！狂飙性能提升300%🔥[1]
视频原文，点击链接[2]

## 摘要

本视频深入探讨了PostgreSQL为何被誉为“最爱的操作系统”，而不仅仅是一个数据库。它详细介绍了PostgreSQL的强大功能和极致的可扩展性，从其作为关系型数据库的卓越性能、MVCC机制，到如何通过各种扩展将其转变为一个能够替代Redis、Elasticsearch、RabbitMQ甚至定制API服务器的综合后端栈。视频还演示了如何设置PGCLI、使用复合类型和JSONB数据，以及实现缓存、发布/订阅和消息队列等功能，展现了其作为多功能开发平台的巨大潜力。

### 亮点

* • 🚀 PostgreSQL是一个开源数据库，以其卓越的性能和可扩展性，被众多主流平台和公司青睐，甚至被描述为一个“操作系统”。
* • ⚙️ 其核心秘密在于多版本并发控制（MVCC），它通过为每个事务提供数据快照，实现了高效的并发处理，显著提升了读写性能。
* • 📦 PostgreSQL支持自定义复合类型和JSONB类型，允许用户在数据库内部定义复杂的数据结构和存储非结构化JSON数据，极大增强了数据处理的灵活性和安全性。
* • 🧩 通过其强大的扩展生态系统，PostgreSQL能够替代多种后端服务，如使用非日志表和pg\_cron实现带有TTL的缓存，或利用通知/监听机制和pg\_mq扩展实现发布/订阅和消息队列功能。
* • 🌍 PostgreSQL还有更多专业的扩展，例如PostGIS用于地理空间数据处理、TimescaleDB用于时序数据、pg\_vector用于向量数据库，使其能够适应各种复杂的应用场景，成为一个全能的后端解决方案。

#PostgreSQL #数据库 #开源 #MVCC #扩展性 #后端开发 #JSONB #缓存 #消息队列 #发布订阅

### 思考

1. 1. PostgreSQL的MVCC机制具体是如何提升并发性能的？

* • MVCC（多版本并发控制）通过为每个事务提供一个数据的“快照”，确保在事务处理期间，数据不会因其他并发写入操作而改变。这意味着读取操作不会被写入操作阻塞，反之亦然，从而极大地提高了数据库的并发读写性能。

* •

+ • 可以在PostgreSQL中创建“非日志表”（UNLOGGED TABLE），这种表不写入WAL（预写日志），因此速度更快但崩溃不安全，适合作为缓存。结合创建删除过期数据的存储过程和使用`pg_cron`扩展来定期执行该过程，就可以实现带有TTL（生存时间）的自动化缓存。

1. 2. 如何在PostgreSQL中实现类似Redis的缓存功能？

### 术语解释

* • **MVCC (Multi-Version Concurrency Control)**：多版本并发控制，是PostgreSQL实现高并发的关键机制。它为每个事务提供一个数据快照，允许多个读写操作同时进行，而不会互相阻塞。
* • **复合类型 (Composite Types)**：PostgreSQL中的一种自定义数据类型，允许用户将多个字段组合成一个单一的类型，类似于编程语言中的结构体或对象，可以在表中作为一列存储。
* • **JSONB**：PostgreSQL支持的一种二进制JSON数据类型。与普通JSON文本存储不同，JSONB存储的是经过解析的二进制表示，这意味着它可以被索引和更高效地查询，是处理非结构化数据的强大工具。
* • **UNLOGGED TABLE**：非日志表。这种表不记录到WAL（预写日志），因此写入速度更快，但其数据在数据库崩溃后可能丢失或被截断，常用于缓存等对持久性要求不高的场景。
* • **pg\_cron**：一个PostgreSQL扩展，允许用户在数据库内部调度和运行Cron作业。这使得数据库可以执行定期的维护任务或自定义过程，例如自动化缓存清理。

## 视频章节总结

### 00:00[3] - 🚀 PostgreSQL：被低估的“操作系统”

视频开篇介绍了PostgreSQL作为40年历史的开源数据库，因其稳定、快速和强大的功能，成为众多主流平台的首选。它不仅是一个数据库，更被比喻为一个拥有完整后端堆栈的“操作系统”，能替代Redis、Elasticsearch、MongoDB等多种服务，并近期实现了超过两倍的性能提升。

### 00:51[4] - 💡 PostgreSQL的起源与核心机制

本章回顾了PostgreSQL（原名Post-Ingres）的起源，由图灵奖得主Michael Stonebraker于1985年在伯克利创建。其秘密武器在于多版本并发控制（MVCC），该机制为每个事务提供数据快照，极大地提高了并发效率，允许多个读写操作同时进行，显著提升了数据库性能。

### 02:46[5] - 🛠️ 快速上手与高级数据类型

视频展示了PostgreSQL的安装与基本操作，介绍了psql和pgcli等客户端工具。重点讲解了PostgreSQL的复合类型，允许用户创建自定义结构的数据类型，将多个字段封装在单个列中，增强数据结构化和安全性。随后，演示了如何创建和查询这些复合类型。

### 06:54[6] - 📄 JSONB：非结构化数据处理利器

本章节介绍了PostgreSQL处理JSON数据的功能，特别是JSONB类型。JSONB以二进制格式存储JSON，支持索引和查询嵌套字段，使其在处理非结构化数据方面表现出色，成为NoSQL数据库的有力替代。视频详细演示了JSONB的创建、插入和复杂的查询操作。

### 09:11[7] - ⚡ MVCC深度解析：并发与事务管理

本章深入探讨了MVCC的工作原理，通过实际的并发操作示例，展示了PostgreSQL如何利用数据快照实现高效的读写不阻塞。同时，介绍了`BEGIN`、`COMMIT`和`ROLLBACK`等事务命令，强调了事务在确保数据完整性和进行无风险数据操作中的关键作用。

### 11:03[8] - ⚙️ 扩展性：替代多后端服务的奥秘

视频强调了PostgreSQL卓越的扩展性，指出它能够替代消息队列、缓存系统甚至任务调度等多种后端服务。通过`UNLOGGED TABLE`实现内存缓存（替代Redis），结合`pg_cron`实现计划任务。还演示了如何在PostgreSQL中实现Pub/Sub模式和消息队列（pgmq），有效替代RabbitMQ等工具。

### 15:48[9] - 🌟 更多强大的扩展与总结

本章介绍了更多PostgreSQL的强大扩展，包括PostGIS（地理空间数据）、TimescaleDB（时间序列数据，可替代Elasticsearch）、hstore（键值存储）和pg\_vector（向量数据库，用于AI应用）。视频总结了PostgreSQL的多功能性和强大性能，并提及了SQLite作为轻量级项目的替代选择。

#BibiGPT https://bibigpt.co

#### 引用链接

`[1]` 【双语+纯享】PostgreSQL：我最爱的操作系统！狂飙性能提升300%🔥: *https://bibigpt.co/video/BV1H7WFz5Ean*
`[2]` 视频原文，点击链接: *https://www.bilibili.com/list/watchlater?oid=115240012354773&bvid=BV1H7WFz5Ean&spm\_id\_from=333.1007.top\_right\_bar\_window\_view\_later.content.click*
`[3]` 00:00: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=0.00*
`[4]` 00:51: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=51.00*
`[5]` 02:46: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=166.00*
`[6]` 06:54: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=414.00*
`[7]` 09:11: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=551.00*
`[8]` 11:03: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=663.00*
`[9]` 15:48: *https://bibigpt.co/content/5bf94acd-c4fa-453f-bf76-4ee524709089?t=948.00*
> 已吸收至：[[04_OLAP与数据库/0402_关系数据库/040202_PostgreSQL/040202_核心知识点/PostgreSQL架构WAL与高可用边界|PostgreSQL架构WAL与高可用边界]]
---
title: 5分钟看懂 PostgreSQL 工作原理
author: IT过客
date:
url: https://mp.weixin.qq.com/s?__biz=MjM5OTE3OTM3OA==&mid=2247485340&idx=1&sn=1445a8b0e26fed20c3e9e92198c5cd59&chksm=a63f9c3bef7e4134f303948f5ae5a9b04c99d4eeaecfb025eb3cc5c6f65beb97e5a4f666db38&mpshare=1&scene=24&srcid=1013bpIttBjEX8cMrdt79eq8&sharer_shareinfo=c8a477142b5a0f5ff5e16a78223265c2&sharer_shareinfo_first=c8a477142b5a0f5ff5e16a78223265c2#rd
---

PostgreSQL 是常用的开源关系型数据库，搞懂它的工作流程，才能更好地优化性能、保障稳定。这张图清晰拆解 PostgreSQL 的核心组件与数据流转逻辑～

一、客户端连接：Web 应用的入口

多个 Web 应用（图中 Web Application）通过 Connection 1/2/3 与 PostgreSQL 建立连接，发送查询、写入等请求。每个连接对应数据库的一个“会话”，处理应用的交互。

二、后台进程：数据库的“工作者”

PostgreSQL 靠后台进程驱动核心逻辑，分为三类关键角色：

 • Background Workers：按需并行工作的“工人”，处理计算密集型任务（如并行查询、备份），提升多核 CPU 利用率。

 • PostgreSQL Shared Memory：共享内存区，是性能核心。里面包含多种“缓冲区”：

 ◦ Shared Buffers：缓存数据页，减少磁盘 IO（常用数据存在这里，读数据时先查缓存，不直接读磁盘）。

 ◦ WAL Buffers：预写日志（Write-Ahead Log）的缓存，保障数据持久化（写数据时，先写日志再落盘，防止断电丢数据）。

 ◦ 还有 Clog Buffers（事务状态缓存）、Temp Buffers（临时数据缓存）等，各司其职。

 • Auxiliary Processes：辅助进程，负责数据库的“运维类”工作：

 ◦ BG Writer：后台写进程，把 Shared Buffers 里的脏数据（修改后的数据）异步刷到磁盘，不阻塞前台查询。

 ◦ WAL Writer：把 WAL Buffers 里的日志刷到磁盘，保障日志持久化。

 ◦ Auto Vacuum：自动清理过期数据（PostgreSQL 中，删除/更新数据是“标记删除”，Auto Vacuum 会真正回收空间，防止表膨胀）。

 ◦ 还有 Checkpointer（检查点，定期刷脏数据与日志）、Stats Collector（统计信息收集，为查询优化提供依据）等，保障数据库稳定运行。

三、主进程：数据库的“大管家”

Postmaster Process 是 PostgreSQL 的主进程，负责：

 • 管理后台进程的启动、停止。

 • 监听客户端连接请求，分配连接资源。

 • 协调各进程工作，是数据库的“总指挥”。

四、物理文件：数据的“最终归宿”

所有数据最终落盘为物理文件，分为四类：

 • Data Files：存储表、索引的实际数据（Shared Buffers 缓存的就是这些文件的数据页）。

 • WAL Files：预写日志文件，记录数据修改历史，用于故障恢复。

 • Archive Files：WAL 日志的归档文件，用于备份、时间点恢复（比如要恢复到昨天 10 点的状态，靠归档日志）。

 • Log Files：数据库运行日志，记录错误、警告、操作记录，用于问题排查。

核心逻辑总结

PostgreSQL 的工作流是：应用连数据库→后台进程处理请求→共享内存缓存加速→辅助进程保障稳定→数据最终落盘为物理文件。理解每个组件的作用，就能针对性优化（比如调大 Shared Buffers 提升缓存命中率，优化 Auto Vacuum 防止表膨胀），让数据库跑得又快又稳～
---
title: 不到 300 Star！SQLite 竟内置消息队列，作者凭什么这么狂
author: 何三笔记
date: 何三何三
url: https://mp.weixin.qq.com/s?__biz=MzA4NTI3OTcyMA==&mid=2649632433&idx=1&sn=0b89b3efe875d45751c2a4ad56af3efd&chksm=86b45d26ef5194a9eae1f23ea6805891a7ed208e79957fb6932ac3ff10f15e274d61fd97924f&mpshare=1&scene=24&srcid=05274k65immYD92EvZM2izac&sharer_shareinfo=2aef22b83182deacd353d6a330c21f18&sharer_shareinfo_first=2aef22b83182deacd353d6a330c21f18#rd
---

大家好，我是何三，独立开发者。

SQLite，那个大家用来存点本地数据的小文件数据库，现在居然能跑消息队列了。不是玩具 demo，是真的能跨进程通知、持久化队列、发布订阅，还带定时任务调度。这个项目叫 **Honker**，GitHub 上不到 300 Star，但看完 README 我真的服了。

以前你要搞个任务队列，标配是什么？Redis + Celery，或者 RabbitMQ，再轻点也得来个 PostgreSQL 配 pg-boss。部署、备份、监控、双写问题……一套下来，周末又没了。Honker 的作者说：既然你业务数据已经在 SQLite 里了，队列为什么不能也在同一个文件里？

要么一起成功，要么一起回滚。没有双写问题，没有"订单创建了但邮件没发出去"的坑。这个想法，怎么说呢，就是那种……一看就觉得很对，但之前居然没人这么干的感觉。

Honker 本质上是一个 SQLite 扩展，底层用 Rust 写的。它最核心的 trick，是在 SQLite 的 WAL（预写式日志）文件上做了文章。

简单讲，每次你的 App 往数据库里写数据，WAL 文件就会变大。Honker 开了一个后台线程，每 1 毫秒 `stat()` 一下这个 WAL 文件的 size 和 mtime。变了？就说明有新数据，立刻通知所有监听的 worker 去读取。

说白了，以前 worker 得不停地问数据库"有没有新活"，现在数据库一有活干，直接敲 worker 的门。

Honker 工作原理

这让我想起小时候村里的广播站。大喇叭一响，全村都知道该收麦子了。SQLite 就是那个村委会，WAL 文件就是大喇叭的电源指示灯——灯一亮，各家的收音机就开始收信号。不过广播站得专门建个发射塔，Honker 倒好，直接在村委会墙上装了个灯泡，零额外基建。

作者管这个叫 "overtriggering"——宁可多叫醒几个 listener，也绝不漏掉一个。每个 worker 被叫醒后会执行一次带索引的 SELECT，如果发现不是自己的频道，就继续睡。代价是多查几次，收益是省掉了整套消息中间件。

说实话，这个 `stat(2)` 每秒轮询 1000 次的做法，第一眼看上去有点糙。为什么不用 `inotify` 或者 `FSEvents`？更优雅啊。

作者解释完我才懂：这些内核事件通知在 macOS 上会把同进程内的写入给吞掉。也就是说，如果你在一个 Python 进程里同时 enqueue 和 listen，FSEvents 可能直接当你没看见。`stat` 虽然土，但 Linux/macOS/Windows 通吃，延迟也就多 0.5 毫秒，几乎可以忽略。

为什么这么设计？别问我，问作者去。但效果确实离谱。

装起来贼简单，`pip install honker` 就完事了。

```
import honker  
  
db = honker.open("app.db")  
emails = db.queue("emails")  
  
# 入队  
emails.enqueue({"to": "alice@example.com"})  
  
# worker 消费  
async for job in emails.claim("worker-1"):  
    send(job.payload)  
    job.ack()
```

最骚的是这个——你可以把业务写入和队列入队放在**同一个事务**里：

```
with db.transaction() as tx:  
    tx.execute("INSERT INTO orders (user_id) VALUES (?)", [42])  
    emails.enqueue({"to": "alice@example.com"}, tx=tx)
```

要么一起成功，要么一起回滚。没有双写问题，没有"订单创建了但邮件没发出去"的坑。

除了队列，Honker 还支持发布订阅（notify/listen）、持久化流（stream，带消费者偏移量）、Crontab 定时任务、速率限制、分布式锁……文档里甚至给了 FastAPI 集成的示例，几行代码就能跑起来。

而且这个扩展是纯 SQLite 侧的，任何支持 `SELECT load_extension('honker')` 的客户端都能用。Python、Node.js、Go、Ruby、Rust、Elixir——跨语言共享同一个 `.db` 文件里的队列。

等等，不只是多语言绑定这件事。用 Rust 写 SQLite 扩展，再用 PyO3 做 Python 绑定、napi-rs 做 Node 绑定，每种语言一个 git submodule，整个项目还是比较整齐的。

Honker 核心能力

同类工具的话，如果你已经在用 PostgreSQL，那 pg-boss 和 Oban 仍然是标配，Honker 的 README 自己也这么承认。Python 圈里还有一个 Huey，也是 SQLite 做后端的消息队列，Honker 很多设计都借鉴了它。

觉得 Honker 星太少不敢用？可以理解。不过 SQLite 生态最近真的在逆袭，希望越来越完善。

Honker 的价值不是让你立刻拆掉 Redis，而是告诉你：如果你的项目本来就以 SQLite 为主数据库，那消息队列、定时任务、发布订阅这些能力，其实不需要再引入第二个数据存储。一个 `.db` 文件 + 一个扩展，全搞定。省下的运维时间，够你多写两套业务代码。

*本文使用 MGO 编辑并发布*

> 关注"何三笔记"，回复"mgo" 免费下载使用
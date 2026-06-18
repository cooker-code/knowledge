> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDCPostgreSQL复制槽WAL膨胀排障|FlinkCDCPostgreSQL复制槽WAL膨胀排障]]
---
title: Flink CDC 线上事故，悄悄吃掉了 12 TB 磁盘
author: 小友Data+AI
date: 小友数研小友数研
url: https://mp.weixin.qq.com/s?__biz=MzYyMzMzNzk0MQ==&mid=2247483988&idx=1&sn=a76b2a1ab282a86166c94e4ab217de73&chksm=fec1d67cdb28a6d4875af3e9904ee9a17a62d1aeb53bf7637ecc78a09fca389ca214fd648e77&mpshare=1&scene=24&srcid=04309QrFrwCND4T3q5srRC3k&sharer_shareinfo=9ba13397667434400ccd60a075361820&sharer_shareinfo_first=9ba13397667434400ccd60a075361820#rd
---

> 凌晨 4 点，告警群炸了：生产 PostgreSQL 磁盘 **94%**，业务写入即将挂掉。一查更崩溃 —— **业务数据 8.2 TB，WAL 日志竟然 12.4 TB**，**78 万**个文件。顺藤摸下去，找到一个**已经下线 125 天的 Flink CDC 任务**留下的"僵尸"。这一篇，把这个坑从现场到机制讲透。

---

#

# 一、事故现场（值班同学的至暗时刻）

| 指标 | 数字 |
| --- | --- |
| 磁盘总量 | **22 TB** |
| 业务数据 | 8.2 TB |
| WAL 日志 | **12.4 TB** ← 占了 56% |
| WAL 文件数 | **~78 万**（每个 16 MB） |
| 已用率 | **94%**（生死线在 95%） |
| 日均 WAL 增量 | ~100 GB / 天（核心交易业务正常水平） |

> WAL 是业务数据的 **1.5 倍**——正常情况下应该是 0.05-0.1 倍。只能说明一件事：**WAL 不是"日志多了"，是"日志根本没人删"**。

---

#

# 二、元凶：3 个被遗忘的复制槽

跑了一行 SQL：

```
SELECT slot_name, active, restart_lsn, confirmed_flush_lsn, catalog_xminFROM pg_replication_slotsWHERE slot_type = 'logical';
```

输出：

| slot\_name | active | restart\_lsn | catalog\_xmin | 备注 |
| --- | --- | --- | --- | --- |
| `slot_a` | `f` | `4A2C/B1234567` | `1234567` | 🔥 **最老** |
| `slot_b` | `f` | `5F8E/95678ABC` | `2345678` |  |
| `slot_c` | `f` | `60C2/A9ABCDEF` | `3456789` |  |

三个逻辑复制槽，全是 active = f —— 早就没人消费了，但槽还留在系统里。

| 槽 | 来历 | 为什么没删 |
| --- | --- | --- |
| `slot_a` | 早期 Flink CDC 测试任务 | 上线就替换了，**人没了，槽没删** |
| `slot_b` | 内部数据集市同步 | 项目下线后**任务停了，槽没删** |
| `slot_c` | 业务统一全局同步 | 最近废弃，**还是没删** |

每个槽下线时，开发都觉得"任务停了就行"。但 PostgreSQL 不这么觉得。

---

#

# 三、复制槽是个什么角色？

把"复制槽"在 Flink CDC 链路里的位置画清楚：

复制槽的本质是个"书签"：它告诉 PostgreSQL "我读到第 X 页了，X 之前的我用过，X 之后的别删，我等下还要看"。

正常情况下，下游持续消费、书签前移，PG 就把书签前面的 WAL 删掉。没问题。

但如果下游不见了、书签停在远古不动呢？

---

#

# 四、这是个什么坑（食堂打饭版）

把 PostgreSQL 想象成一个食堂：

| 角色 | 在食堂里 | 在 PostgreSQL 里 |
| --- | --- | --- |
| 🔥 厨房 | 持续做菜 | 业务写入持续产生 WAL |
| 🍱 传菜带 | 一份份菜按顺序流过来 | WAL 日志按 LSN 顺序追加 |
| 📍 食客的位置标记 | "我吃到这一份了" | 复制槽的 `restart_lsn` |
| 食客没来吃 | 标记停在原地 | `active = f`，槽不消费 |

食堂 / PG 的铁律：

> ⛔ 只要食客的位置标记还在 → 这位食客之后的菜，一份都不许倒掉⛔ 只要复制槽存在，槽 `restart_lsn` 之后的 WAL，一勺不许删——哪怕磁盘要爆了

结果：

> 食客 A 的位置最早（125 天前的标记）→ 从那个位置之后的**所有菜全留着**→ 传菜带越堆越长 → 直到食堂塞不下（**12.4 TB**）

---

#

# 五、时间线还原：3 代"僵尸"叠加，跨度 4 个月

最有意思的是：这三个槽不是同时挂的。

按平均 WAL 写入速度（约 100 GB/天）反推三个槽的 restart\_lsn 差距：

| 槽 | 下线时间 | 累计偷吃 WAL |
| --- | --- | --- |
| `slot_a` | **125 天前** | **~12.4 TB**（完整保留期） |
| `slot_b` | 30 天前 | ~3 TB（叠加在上方） |
| `slot_c` | 5 天前 | ~500 GB（叠加在上方） |

> 🔥 **最毒的是最老那个 slot\_a** —— 它的 `catalog_xmin` 最老，是整个 WAL 清理的"地板"。后面两个槽再新，**也只是雪上加霜**：只要最老的不删，全局 WAL 都不能清。

---

#

# 六、为什么一个老槽能锁死全局？（核心机制）

这是最反直觉的一点。

PG 清理 WAL 的判定逻辑，不是"逐个槽看"，而是"全局最老的那个说了算"：

暂时无法在飞书文档外展示此内容

举个具象数字：

| 槽 | restart\_lsn | 影响 |
| --- | --- | --- |
| `slot_a` | 500 | ⛔ **最老 = 全局清理天花板** |
| `slot_b` | 800 | 即使消费过 500-800，**也不能删** |
| `slot_c` | 1000 | 即使消费过 500-1000，**也不能删** |
| 当前 LSN | 10000 | 500-10000 之间的 WAL **全部保留** |

一个最老僵尸 = 全员陪葬。

---

#

# 七、紧急处理三步走

> ⚠️ 删除复制槽**不可逆**。生产环境必须先和业务确认：下游消费端是真的下线了，不是临时挂掉。

Step 1｜确认现场

```
SELECT slot_name, active, restart_lsn, confirmed_flush_lsn, catalog_xmin,       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS lag_sizeFROM pg_replication_slots;
```

lag\_size 这一列直接告诉你：这个槽落后了多少 GB / TB 的 WAL。

Step 2｜按"最老优先"删除

先删最老的那个，因为它是天花板：

```
SELECT pg_drop_replication_slot('slot_a');     -- 第一刀，最关键SELECT pg_drop_replication_slot('slot_b');     -- 第二刀SELECT pg_drop_replication_slot('slot_c');     -- 第三刀
```

Step 3｜触发 WAL 清理

PG 不会立刻清理，要等下一次 checkpoint。在 IO 低峰期手动触发：

```
CHECKPOINT;
```

| 时间点 | `pg_wal/` 大小 | 已用率 |
| --- | --- | --- |
| 删除前 | 12.4 TB | 94% 🚨 |
| CHECKPOINT 后 30 分钟 | ~400 GB | 41% ✅ |
| 1 小时后 | ~250 GB | 39% ✅ |

磁盘空间像潮水一样退去🌊

---

#

# 八、长期预防四件套

把这个坑从根上堵死，需要做四件事：

|  | 配置层 | 检测层 |
| --- | --- | --- |
| **被动兜底** | **① max\_slot\_wal\_keep\_size**设硬上限（如 500 GB） → 即便忘了删也不会爆 | **② 周巡检"僵尸槽"**active=f 且 lag>200G 自动告警 → 主动发现，提前处理 |
| **主动预防** | **③ 流程化下线**下线 checklist 必须含"删槽"一行 → 从源头杜绝 | **④ 监控对象升级**`pg_wal/` 单独监控 70/80/90 三级 → 兜底，最后一道防线 |

① 必改参数：max\_slot\_wal\_keep\_size

这个参数默认 -1（无上限），是这次事故的直接放大器。改成业务能接受的硬上限：

```
ALTER SYSTEM SET max_slot_wal_keep_size = '500GB';  SELECT pg_reload_conf();
```

效果：单个槽落后超过 500 GB，PG 就强制清理，宁可下游丢数据也不让磁盘炸。

> 💡 **取舍**：值太小，下游短暂断连就丢数据；值太大，僵尸槽危害大。经验值：**留 3-5 天的正常 WAL 量**比较合理。本案例 100 GB/天，500 GB 上限对应 5 天。

② 周巡检：扫描"僵尸槽"

```
SELECT slot_name,       active,       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS lag_sizeFROM pg_replication_slotsWHERE active = 'f'   OR pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) > 200 * 1024 * 1024 * 1024;  -- 落后 200 GB
```

③ 流程化：业务下线必须删槽

在 Flink CDC / DataX / 其他 CDC 工具的下线 checklist 里加一行：

1. Flink 任务停止

2. 检查 PostgreSQL 复制槽

3. pg\_drop\_replication\_slot('xxx')

4. 验证 pg\_wal/ 大小开始下降

不写 checklist，靠"开发记得"——大概率忘。

④ 监控：把 pg\_wal/ 当一等监控对象

很多人只盯业务表大小，不盯 pg\_wal/。这次事故就是被 pg\_wal/ 偷袭的。

加监控指标：

1. pg\_wal/ 目录磁盘使用率（70% / 80% / 90% 三级告警）

2. pg\_replication\_slots 中 active = f 的数量

3. 最老槽的 lag size（GB）

---

#

# 九、写在最后

如果你正在用 Flink CDC 同步 PostgreSQL（或 MySQL 的 binlog 类似机制），这三件事每一件都要刻在 SOP 里：

125 天前给它起个随手的名字（不管是 slot\_a、test\_v1 还是 flink\_xxx\_demo）—— 看起来都挺无害，一个临时任务嘛。但今天它的代价是12 TB 磁盘 + 一次半夜告警 + 复盘 + 业务一度停写。

数据库不会判你的"槽是不是真的临时"。它只看槽存不存在。

---

关注「**小友 Data+AI**」，分享更多实战经验。后台回复「**项目**」，获取 Data+AI 实战项目。
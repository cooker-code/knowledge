> 已吸收至：[[04_OLAP与数据库/0401_OLAP引擎/040103_StarRocks/040103_核心知识点/StarRocks存算分离Compaction机制|StarRocks存算分离Compaction机制]]
---
title: 极速查询：StarRocks 存算分离 Compaction 原理 & 调优指南
author: StarRocks
date:
url: http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493628&idx=1&sn=17351e910dbddefa57a60a3efd19778a&chksm=e9f298d8de8511ce04da1cc9dce835e6382cbfc2f1b34d51b6b581bf81607dc653ac9ce33347&mpshare=1&scene=24&srcid=07121ZLnOo4yisLF6cVt660k&sharer_shareinfo=c78d880b5bf3ea3ff0ad10814bca8039&sharer_shareinfo_first=c78d880b5bf3ea3ff0ad10814bca8039#rd
---

作者：丁凯，StarRocks TSC member/镜舟科技云原生技术负责人

StarRocks 在数据摄入过程中，每次操作都会创建一个新的数据版本。在查询时，为了得到准确的结果，必须将所有版本合并。然而，随着历史数据版本的累积，需要合并的文件数量增多，这将显著降低查询效率。为了解决这个问题，StarRocks 会定期执行内部任务，通过合并历史数据版本来消除重复记录，这个过程被称为 Compaction。

举个例子：下图中 version 1 和 version 2 数据文件进行 Compaction 后，消除了 version 1 中的旧版本数据（id = 2, value = 11, id = 5, value = 30），最终生成了新的数据版本文件 version 3。

从上述说明可以看出，**Compaction 是将不同版本的数据文件合并成大文件的过程，旨在减少系统中小文件数量、从而提升查询效率。**相比于存算一体表，StarRocks 存算分离实现了新的 Compaction 调度机制，具有以下特点：

* Compaction 调度由 FE 发起，BE 执行。FE 按照 Partition 为单位来发起 Compaction 任务
* Compaction 会生成一个新版本，也走导入的写数据、commit、publish version 的完整流程

本文旨在描述 StarRocks 存算分离表 Compaction 基本实现原理，帮助开发和运维人员能更好地理解并根据实际需要调整 Compaction 相关配置，以在实践中获得更优的性能表现。

**数据版本更新机制**

如前所述，每次导入都会在 FE 内生成一个新版本，而该版本被标记在 Partition 之上。一旦导入事务成功提交，便会更新 Partition 的可见数据版本号，Partition 的数据版本号单调递增。

需要注意的是，一个 Partition 内可能存在多个 Tablet，这些 Tablet 都共享相同的数据版本号，即使一次导入可能只涉及其中部分 Tablet，一旦导入事务成功提交，Partition 下所有的 Tablet 的版本都会相应地得到提升。

例如上图中，Partition X 内含 Tablet 1 ~ N，当前的可见版本为12。一旦产生新的导入事务 New Load Txn，且该事务成功提交，那么 Partition X 的可见版本就变成了 13。

**基本框架**

**StarRocks 存算分离表 Compaction 由两个角色组成：调度者（Compaction Scheduer）和执行者（Compaction Executor）。**调度者通过 RPC 发起 Compaction 任务（Compaction Job），而执行者负责执行 Compaction Job。

**在 StarRocks 存算分离中，FE 作为 Compaction Scheduler，而 BE 或者 CN 都作为 Compaction Executor。**每个 Compaction Executor 内都存在一个线程池专门用于执行 Compaction Job。

### **1** **Compaction Scheduer 调度**

###

FE 上存在一个周期性运行线程 Compaction Scheduler，负责调度发起所有的 Compaction Task。**FE 以 Partition 为调度的基本单位。**

**FE 上掌握了每个 Partition 的 Compaction Score 信息，**该信息用来表示 Partition 内所有 Tablet 的需要进行 Compaction 的优先级，Compaction Score 越高，表示 Partition 需要合并的紧急程度越高。

**每次 Compaction Scheduler 线程运行时，会挑选出当前 Compaction Score 最高的 Partition，并为这些 Partition 构造 Compaction Task。**当然，Compaction Scheduler 也会控制每次最多发起的 Compaction Task 数量。

构造 Compaction Task 的逻辑相对比较简单，对于每个 Partition，Scheduler 会获得其所有的 Tablet，然后为每个  Compute Node （CN） 构造一个 Compaction Task，Task 内包含需要在该 CN 上执行 Compaction 任务的 Tablet 列表，然后发送 Task 给 CN 节点。

整个流程如下图所示：

在上图中，FE 上存在两个 Partition 需要执行 Compaction，分别为 Partition X 和 Partition Y。Partition X 内含4个 Tablet（1 ~ 4），而 Partition Y 内含3个 Tablet（5~7）。

Scheduler 通过计算发现：

Partition X 内，Tablet-2 和 Tablet-4 位于相同的 CN-1，而 Tablet-1 和 Tablet-3 位于相同的 CN-2，于是为 Partition X 构造了两个 Compaction Task（Task-1 与 Task-2），其中 Task-1 内包含 Tablet-2 和 Tablet-4，而 Task-2 内包含 Tablet-1 和 Tablet-3。

Partition Y 内，Tablet-5 和 Tablet-7 位于相同的 CN-1，而 Tablet-6 位于另外一个CN-2，于是为 Partition Y 也构造了两个 Compaction Task（Task-3 与 Task-4），其中 Task-3 内包含 Tablet-5 和 Tablet-7，而 Task-4 内包含 Tablet-6。

最终，每个 Task 被发往自己所属的 CN。CN 节点上存在专有线程池来处理这些 Task，且线程池数量可配置（即将支持动态配置）。每个线程会从 Compaction Task 任务队列中获取下一个要被执行的 Task。

**Compaction 后的数据清理**

目前 StarRocks 存算分离表使用了数据多版本技术，整体上的存储结构如下图所示：

上图中共产生了三次数据导入事务，其中：

* Load Txn 1: 在事务数据写入阶段，生成了新数据文件 file 1 & file 2，且该事务提交后生成了 Tablet Meta V1，其中记录该版本可见的文件列表为 {file-1, file-2}
* Load Txn 2: 在事务数据写入阶段，生成了新数据文件 file 3 & file 4。在提交时，根据前一个版本（即 Tablet Meta V1）然后加上本次导入事务生成的新数据文件（file-3 & file-4），生成了新的 Tablet Meta V2，因此，该版本可见的文件列表为 {file-1, file-2, file-3, file-4}
* Load Txn 3: 在事务写入阶段，产生了新数据文件 file 5。该事务提交时，根据前一个版本（即 Tablet Meta V2）然后加上本次导入事务生成的新数据文件（file-5），产生了新的 Tablet Meta V3，因此，该版本的可见文件列表为 {file-1, file-2, file-3, file-4, file-5}

除了用户导入事务产生了新的数据版本，在存算分离表中，系统后台 Compaction 任务也会产生新数据版本。Compaction 的目的有二:

1. 将多个版本的小文件合并为大文件，减少查询时的随机 I/O 次数；
2. 消除重复数据记录，减少数据总量。

在存算分离表中，每次 Compaction 也会产生一个全新的版本。依然以上面为例，假如在上面 Txn 3 之后新的事务 Txn 4 为一次 Compaction 任务，并且将 file1 ~ file4 这4个文件合并成为 file-6，那么该事务提交时，生成的新版本 Tablet Meta V4 内记录的文件列表为 {file-5, file-6}。

观察上例并思考可知，如果系统在运行过程中一直不会进行 Compaction。那么系统中的数据文件永远也无法被删除。试想上例中我们可以将 Tablet Meta V1，Tablet Meta V2 文件删除，但我们无法删除 file-1、file-2、file-3 以及 file-4，因为这些文件依然被 Tablet Meta V3 所引用。

但有了数据合并（Compaction）后，情况就变得不一样了。上例中，由于发生了一次 Compaction（上图中的 Compact Txn 4），将 file-1、file-2、file-3、file-4 合并生成了新文件 file-6 并生成了新的 Tablet Meta V4。由于 file-1 至 file-4 中的内容已经在 file-6 中存在，所以当版本 V1、V2、V3 不再被访问，file-1 至 file-4 便可以被安全删除。此时的数据版本情况如下图所示：

因此，综合上面的讨论，我们可以发现，只有在 Compaction 完成后原始的数据文件方可被删除。因而，判断数据文件能否安全删除的最直观的规则是：该数据文件不再被任何 Tablet Meta 所引用。

##

**Compaction 调优指南**

### **1** **查看分区的 Compaction score**

注意：以下命令需要连接 Leader FE 节点执行

**StarRocks 在内部为每个分区（Partition) 维护了一个 Compaction Score 值，它反映了分区当前数据文件合并情况，Compaction score 越高，代表了数据文件合并程度越低。**

StarRocks 提供了命令可查看 Partition 当前的 Compaction Score，FE 会以此作为发起 Compaction 任务的参考，用户也可以此作为判断当前 Partition 是否存在版本数过多的依据：

**方法1：**

```
MySQL [(none)]> show proc '/dbs/load_benchmark/store_sales/partitions';+-------------+---------------+----------------+----------------+-------------+--------+--------------+-------+------------------------------+---------+----------+-----------+----------+------------+-------+-------+-------+| PartitionId | PartitionName | CompactVersion | VisibleVersion | NextVersion | State  | PartitionKey | Range | DistributionKey              | Buckets | DataSize | RowCount  | CacheTTL | AsyncWrite | AvgCS | P50CS | MaxCS | +-------------+---------------+----------------+----------------+-------------+--------+--------------+-------+------------------------------+---------+----------+-----------+----------+------------+-------+-------+-------+| 38028       | store_sales   | 913            | 921            | 923         | NORMAL |              |       | ss_item_sk, ss_ticket_number | 64      | 15.6GB   | 273857126 | 2592000  | false      | 10.00 | 10.00 | 10.00 |+-------------+---------------+----------------+----------------+-------------+--------+--------------+-------+------------------------------+---------+----------+-----------+----------+------------+-------+-------+-------+1 row in set (0.20 sec)
```

**方法2：**

自从新版本 3.1.9 & 3.2.4 ，我们在系统表中增加了 partitions\_meta 表，方便用户通过各种复杂 SQL 来查看系统所有 Partition 信息：

```
mysql> select * from information_schema.partitions_meta order by Max_CS;+--------------+----------------------------+----------------------------+--------------+-----------------+-----------------+----------------------+--------------+---------------+-----------------+-----------------------------------------+---------+-----------------+----------------+---------------------+-----------------------------+--------------+---------+-----------+------------+------------------+----------+--------+--------+--------------------------------------------------------------------------------------------------------+| DB_NAME      | TABLE_NAME                 | PARTITION_NAME             | PARTITION_ID | COMPACT_VERSION | VISIBLE_VERSION | VISIBLE_VERSION_TIME | NEXT_VERSION | PARTITION_KEY | PARTITION_VALUE | DISTRIBUTION_KEY                        | BUCKETS | REPLICATION_NUM | STORAGE_MEDIUM | COOLDOWN_TIME       | LAST_CONSISTENCY_CHECK_TIME | IS_IN_MEMORY | IS_TEMP | DATA_SIZE | ROW_COUNT  | ENABLE_DATACACHE | AVG_CS   | P50_CS | MAX_CS | STORAGE_PATH                                                                                           |+--------------+----------------------------+----------------------------+--------------+-----------------+-----------------+----------------------+--------------+---------------+-----------------+-----------------------------------------+---------+-----------------+----------------+---------------------+-----------------------------+--------------+---------+-----------+------------+------------------+----------+--------+--------+--------------------------------------------------------------------------------------------------------+| tpcds_1t     | call_center                | call_center                |        11905 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | cc_call_center_sk                       |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 12.3KB    |         42 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11906/11905 || tpcds_1t     | web_returns                | web_returns                |        12030 |               3 |               3 | 2024-03-17 08:40:48  |            4 |               |                 | wr_item_sk, wr_order_number             |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 3.5GB     |   71997522 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/12031/12030 || tpcds_1t     | warehouse                  | warehouse                  |        11847 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | w_warehouse_sk                          |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 4.2KB     |         20 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11848/11847 || tpcds_1t     | ship_mode                  | ship_mode                  |        11851 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | sm_ship_mode_sk                         |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 1.7KB     |         20 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11852/11851 || tpcds_1t     | customer_address           | customer_address           |        11790 |               0 |               2 | 2024-03-17 08:32:19  |            3 |               |                 | ca_address_sk                           |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 120.9MB   |    6000000 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11791/11790 || tpcds_1t     | time_dim                   | time_dim                   |        11855 |               0 |               2 | 2024-03-17 08:30:48  |            3 |               |                 | t_time_sk                               |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 864.7KB   |      86400 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11856/11855 || tpcds_1t     | web_sales                  | web_sales                  |        12049 |               3 |               3 | 2024-03-17 10:14:20  |            4 |               |                 | ws_item_sk, ws_order_number             |     128 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 47.7GB    |  720000376 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/12050/12049 || tpcds_1t     | store                      | store                      |        11901 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | s_store_sk                              |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 95.6KB    |       1002 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11902/11901 || tpcds_1t     | web_site                   | web_site                   |        11928 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | web_site_sk                             |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 13.4KB    |         54 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11929/11928 || tpcds_1t     | household_demographics     | household_demographics     |        11932 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | hd_demo_sk                              |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 2.1KB     |       7200 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11933/11932 || tpcds_1t     | web_page                   | web_page                   |        11936 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | wp_web_page_sk                          |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 43.5KB    |       3000 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11937/11936 || tpcds_1t     | customer_demographics      | customer_demographics      |        11809 |               0 |               2 | 2024-03-17 08:30:49  |            3 |               |                 | cd_demo_sk                              |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 2.7MB     |    1920800 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11810/11809 || tpcds_1t     | reason                     | reason                     |        11874 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | r_reason_sk                             |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 1.9KB     |         65 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11875/11874 | | tpcds_1t     | promotion                  | promotion                  |        11940 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | p_promo_sk                              |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 69.6KB    |       1500 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11941/11940 || tpcds_1t     | income_band                | income_band                |        11878 |               0 |               2 | 2024-03-17 08:30:48  |            3 |               |                 | ib_income_band_sk                       |       1 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 727B      |         20 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11879/11878 || tpcds_1t     | catalog_page               | catalog_page               |        11944 |               0 |               2 | 2024-03-17 08:30:52  |            3 |               |                 | cp_catalog_page_sk                      |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 1.8MB     |      30000 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11945/11944 || tpcds_1t     | item                       | item                       |        11882 |               0 |               2 | 2024-03-17 08:30:51  |            3 |               |                 | i_item_sk                               |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 37.1MB    |     300000 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11883/11882 || tpcds_1t     | store_returns              | store_returns              |        11755 |               3 |               3 | 2024-03-17 09:02:48  |            4 |               |                 | sr_item_sk, sr_ticket_number            |      32 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 11.3GB    |  287999764 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11756/11755 || tpcds_1t     | date_dim                   | date_dim                   |        11828 |               0 |               2 | 2024-03-17 08:30:47  |            3 |               |                 | d_date_sk                               |      16 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 1.5MB     |      73049 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/11829/11828 || tpcds_1t     | catalog_sales              | catalog_sales              |        12215 |               3 |               3 | 2024-03-17 11:44:37  |            4 |               |                 | cs_item_sk, cs_order_number             |     256 |               1 | HDD            | 9999-12-31 23:59:59 | NULL                        |            0 |       0 | 94.7GB    | 1439982416 |                0 |        0 |      0 |      0 | s3://starrocks-cloud-data-zhangjiakou/dingkai/536a3c77-52c3-485a-8217-781734a970b1/db10328/12216/12215 |28 rows in set (0.04 sec)
```

需要关注以下参数：

* AvgCS：当前 Partition 上所有 Tablet 平均 Compaction Score
* MaxCS: 当前 Partition 上所有 Tablet 最大 Compaction Score

### **2** **查看 Compaction 任务**

随着导入任务的执行，系统内部也在不断地调度执行 Compaction 任务，这些任务会被发往计算节点 CN 执行，系统也提供了一系列命令可以查看当前 Compaction 任务执行情况。

首先，用户可以通过如下命令来观察系统当前所有 Compaction 任务的整体情况：

```
MySQL [(none)] show proc '/compactions';+----------------------------------------------------+--------+---------------------+------------+---------------------+---------------------------------------------------------------------------------+|Partition                                          | TxnID  | StartTime           | CommitTime | FinishTime          | Error                                                                           |+----------------------------------------------------+--------+---------------------+------------+---------------------+---------------------------------------------------------------------------------+| load_benchmark.store_sales.store_sales             | 197562 | 2023-05-24 15:50:33 | 2023-05-24 15:51:00 | 2023-05-24 15:51:02 | NULL                                                                   |+----------------------------------------------------+--------+---------------------+------------+---------------------+---------------------------------------------------------------------------------+1 rows in set (0.21 sec)
```

上面显示了当前有一个 Compaction 任务正在进行，其各字段含义如下：

> *Partition*：当前正在进行的 Compaction 任务针对的 Partition*TxnID*：FE 为当前 Compaction 任务分配的事务 id*StartTime*：该 Compaction 任务的开始时间*CommitTime*：该 Compaction 任务的 Commit 时间*FinishTime*：该 Compaction 任务的结束时间*Error*：该 Compaction 任务出错信息，如果一切正常，值为NULL

上面的命令展示了每个 Partition 上对应的 Compaction 任务总体情况。而每个 Compaction 任务底层会被按照 Tablet 分为多个子任务，系统也提供了如下命令来观察每个 Compaction 子任务的详细进展情况：

```
MySQL [(none)]> select * from information_schema.be_cloud_native_compactions where TXN_ID = 197562;+-------+--------+-----------+---------+---------+------+---------------------+---------------------+----------+--------+| BE_ID | TXN_ID | TABLET_ID | VERSION | SKIPPED | RUNS | START_TIME          | FINISH_TIME         | PROGRESS | STATUS |+-------+--------+-----------+---------+---------+------+---------------------+---------------------+----------+--------+| 36027 | 197562 |     38033 |     365 |       0 |    1 | 2023-05-24 15:50:34 | 2023-05-24 15:50:38 |      100 | OK     || 10004 | 197562 |     38064 |     365 |       0 |    1 | 2023-05-24 15:50:42 | 2023-05-24 15:50:47 |      100 | OK     || 10004 | 197562 |     38065 |     365 |       0 |    1 | 2023-05-24 15:50:46 | NULL                |       99 | OK     || 10004 | 197562 |     38067 |     365 |       0 |    1 | 2023-05-24 15:50:46 | NULL                |       92 | OK     || 10004 | 197562 |     38068 |     365 |       0 |    1 | 2023-05-24 15:50:47 | NULL                |       87 | OK     || 10004 | 197562 |     38072 |     365 |       0 |    1 | 2023-05-24 15:50:47 | NULL                |       89 | OK     || 10004 | 197562 |     38076 |     365 |       0 |    0 | NULL                | NULL                |        0 | OK     || 10004 | 197562 |     38087 |     365 |       0 |    0 | NULL                | NULL                |        0 | OK     || 10004 | 197562 |     38088 |     365 |       0 |    0 | NULL                | NULL                |        0 | OK     || 10004 | 197562 |     38092 |     365 |       0 |    0 | NULL                | NULL                |        0 | OK     || 10004 | 197562 |     38093 |     365 |       0 |    0 | NULL                | NULL                |        0 | OK     |+-------+--------+-----------+---------+---------+------+---------------------+---------------------+----------+--------+64 rows in set (0.22 sec)
```

```
关注两个字段：
```

* PROGRESS：代表 Tablet 当前 Compaction 进展情况，为百分比
* STATUS：代表任务状态，如果有错误，这里会展示错误详细信息

### **3** **取消 Compaction 任务**

用户可以通过下面的命令来取消特定的 Compaction 任务。需要注意的是，该命令需要连接 Leader FE 节点执行：

```
```
CANCEL COMPACTION WHERE TXN_ID = 123;
```
```

### 4 **参数调优**

StarRocks 中有如下参数来控制存算分离下的 Compaction 行为。

### **FE 参数**

```
# 最小的Compaction score，低于该值的 Partition 不会发起Compaction任务lake_compaction_score_selector_min_score = 10.0;
# FE 上可同时发起的 Compaction Task 数量# 默认值为-1，即FE会根据系统中 BE 数量自动计算# 如果设置为0，则 FE 不会发起任何 Compaction 任务lake_compaction_max_tasks = -1;
# 控制show proc '/compactions' 显示的结果数量，默认为12lake_compaction_history_size = 12;lake_compaction_fail_history_size = 12;
```

FE 所有上述 Compaction 相关参数均可通过 SQL 命令动态修改，例如：

```
admin set frontend config ("lake_compaction_max_tasks" = "0");
```

### **BE / CN 参数**

```
# 控制 BE/CN 上同时执行 Compaction 任务的线程数，默认值为4# 也即 BE 上可同时为多少个 Tablet进行 Compactioncompact_threads = 4
# BE 上 Compaction任务队列大小，控制可接收来自FE的最大Compaction 任务数# 默认值为100compact_thread_pool_queue_size = 100
# 单次 Compaction 任务最多合并的数据文件数量，默认为1000# 在实践中我们建议将该值调整为100，这样，每个 Compaction Task 可以更快速地结束# 且消耗更少的资源max_cumulative_compaction_num_singleton_deltas=100
```

BE 所有上述 Compaction 相关参数在最新版本中已经支持动态修改，可以通过如下方式修改：

```
```
mysql> update information_schema.be_configs set value = 8 where name = "compact_threads";Query OK, 0 rows affected (0.01 sec)
```
```

### **5** **最佳实践**

由于 Compaction 对于查询性能的影响至关重要，我们建议用户时刻关注系统中表与分区的后台数据合并情况。以下是几点最佳实践建议：

* 关注 Compaction Score，根据指标配置告警。StarRocks 自带的 Grafana 监控模板已经包含了该指标 （备注1）
* 关注 Compaction 消耗的资源情况，尤其是内存。StarRocks 自带的 Grafana 监控模板已经包含了该指标（备注1）
* 在资源较空闲时可增加计算节点上并行 Compaction 工作线程数，以加快 Compaction 任务的执行

备注：

1. 使用 Prometheus 和 Grafana 监控报警：https://docs.starrocks.io/zh/docs/administration/management/monitoring/Monitor\_and\_Alert/

**关于 StarRocks**

Linux 基金会项目 StarRocks 是新一代极速全场景 MPP 数据库，遵循 Apache 2.0 开源协议。

面世三年来，StarRocks 致力于帮助企业构建极速统一的湖仓分析新范式，是实现数字化转型和降本增效的关键基础设施。目前，全球 380 家以上市值超过 70 亿元人民币的顶尖企业选择用 StarRocks 来构建新一代数据分析能力，这些企业包括腾讯、携程、平安银行、中原银行、中信建投、招商证券、大润发、百草味、顺丰、京东物流、TCL、OPPO 等。StarRocks 也已经和全球云计算领导者亚马逊云、阿里云、腾讯云等达成战略合作关系。

StarRocks 全球开源社区也正飞速成长。目前，StarRocks 的 GitHub star 数已达 8100，吸引了超过 350 位贡献者和数十家国内外行业头部企业参与共建，用户社区也有过万人的规模。凭借其卓越的表现，StarRocks 荣获了全球著名科技媒体 InfoWorld 颁发的 2023 BOSSIE Award 最佳开源软件奖项。

**金融：**[中信建投](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488978&idx=1&sn=68da6c95be3337048c5900ce70d56c5e&chksm=e9f16af6de86e3e0bc992f098a8d0787a18ff036bc70ea2d11f7d7e08358af16224f52d237a0&scene=21#wechat_redirect)｜[中原银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487331&idx=1&sn=e6cdcdf2c46762135b6d95bf58e47664&chksm=e9f17047de86f9519e4d58f47745fe64788f2a5b3117133fc4b39db1a50c50d163a467f20950&scene=21#wechat_redirect) | [申万宏源](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492791&idx=1&sn=e4eba3e3d3de2fc4e59148a2e52f7201&chksm=e9f29b93de851285bf00f6e94a0f096b24d9a494b0013d43fb42d006eabea27a615e605b5a1b&scene=21#wechat_redirect) | [平安银行](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493044&idx=1&sn=580ea0a521f9c9e4099356022536a585&chksm=e9f29a90de851386c04da4c3810b3c1ce0fce8e7f128fcbee0fb1c1a480a17aa5582325f20ee&scene=21#wechat_redirect) | 中欧财富

**互联网：**[微信｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492524&idx=1&sn=7cbd70eb86f606ea58d293a9430e7047&chksm=e9f29c88de85159ed3245a906475ab9bdd32d93478d76379ff79dcd344383aec25f3df4d7a0e&scene=21#wechat_redirect)[小红书｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492619&idx=1&sn=42698abb77f05f6a317ad278ffaca257&chksm=e9f29b2fde851239e5553ac00d7e1227dd8346eabc3848e9e35a2ddfbf152dc8376ade2c9e79&scene=21#wechat_redirect)[网易邮箱｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488632&idx=1&sn=0f76b5ce855b3070a138cb58e5fb8112&chksm=e9f16b5cde86e24a853e743df776e25d2358e710442cb419d32c3dd504d8e328e06b5cb473fd&scene=21#wechat_redirect)[滴滴｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490839&idx=1&sn=a007c5207f15129cd8c91da15cc30093&chksm=e9f16233de86eb25f585ef9b911e54ae6dae9547bd240a9554dd52ba449a74289a13d3df99fa&scene=21#wechat_redirect)[美团餐饮SaaS |](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488130&idx=1&sn=53c32ae5448505db505872e0d8e8d7e1&chksm=e9f16da6de86e4b03a1295902ac27ff3fe9a554dd76383b6d24c155fb11778bc4d4afa7d9494&scene=21#wechat_redirect)[B站｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492334&idx=1&sn=dd0de6197bc6f3f67ffb0704b72ae02f&chksm=e9f29dcade8514dcfbfb19d06dc75659ff7c5d9d72ec10210eceb5220d9706b56b61d80dd9d8&scene=21#wechat_redirect)[携程 |](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491637&idx=1&sn=daec09546a550346b65414567dc8db7b&chksm=e9f29f11de851607eb14a9e006803d8f52951d367f19a2477d5abee92f309c8c8145655a959c&scene=21#wechat_redirect)[同程旅行｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490253&idx=1&sn=93f2a0369f9a9bff2215031efd197530&chksm=e9f165e9de86ecffe737cfebf729668df7efe09e6f9c4c976dc17668dc8042ef7895aef74ea3&scene=21#wechat_redirect)[360｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485899&idx=1&sn=c8b13a6a6f9d0d59de1721a36f52c4cd&chksm=e9f176efde86fff974e81dfe2fb72a834528141a9b0f4a99f647b11c1a2e9001822b82ddbd1e&scene=21#wechat_redirect)[58同城｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487038&idx=1&sn=583cf978f57802edd653034d83ac30af&chksm=e9f1711ade86f80cc37de844768228ba7e6a6abc06dd8b4867303334d31334e7b23273d12c39&scene=21#wechat_redirect)[芒果TV｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491190&idx=1&sn=42c67a896174b68e4c66c955ad72c81e&chksm=e9f16152de86e84403fdc9bbe14e3ed49dc743b9243ab1067af4731f1d9038b1402992046ba8&scene=21#wechat_redirect)[得物](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487306&idx=1&sn=46b784eb29c1a4d645946a017deb7f8a&chksm=e9f1706ede86f97861ce9ed2952335312704fe2cc9adc35b3cfa5709f2abdeea9a2779ff18d6&scene=21#wechat_redirect) ｜[贝壳｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490822&idx=1&sn=a8bc0b31aa6568f57e919632e3f15877&chksm=e9f16222de86eb34d2a4eabd28c36ff2b29f3d9ad72855e8a9064d3b84b24a1e13970a80d3e3&scene=21#wechat_redirect)[汽车之家](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484870&idx=1&sn=74ab3350c6b7e86376c009b3ee6b5b98&chksm=e9f17ae2de86f3f45cc32c4ae153b61c57cdf6e66dd7b10ae9fb8ec689994598130257c43936&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[欢聚集团](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)｜[腾讯](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247492960&idx=1&sn=f69f94be1400c0d3260e5c1b0aaa48cb&chksm=e9f29a44de8513520a3d4e6db9abb3936c806ad82ce7194421147c96dac10638cbae5ee817dd&scene=21#wechat_redirect)

**游戏：**[腾讯游戏｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486696&idx=1&sn=d4bfaf2c2fb84a890922a0dfd4879522&chksm=e9f173ccde86fadaa1313f3e1c82d71f44b3e244113c712a272d3c85543765752431f8896ae6&scene=21#wechat_redirect)[波克城市](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486150&idx=1&sn=bc272d1174edb74233c584c1b7f970da&chksm=e9f175e2de86fcf4b78f09ce88ead3dc591e11edd11a3efafe74441591928d37c213b9871e7d&scene=21#wechat_redirect)[｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486170&idx=1&sn=bd5de60aa15f03a6972467c45c58bb9b&chksm=e9f175fede86fce8c2495df1d7f6f3096b70ec8ac69ee94e12f96f3e15c256a85a94c3e3c5f7&scene=21#wechat_redirect)[37手游](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486749&idx=1&sn=610a64862a91df789faaa2469f2c8116&chksm=e9f17239de86fb2f78abd9f391d3c780f371213c93150d8617bb1c9a1845b837293f203f7d41&scene=21#wechat_redirect) | [游族网络](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487470&idx=1&sn=e8844ab1037069989e3dc95f5e9092e8&chksm=e9f170cade86f9dcd62d810c6f7132002c4cf7329dbf20f232db82ca6cca955ffd934be04e0c&scene=21#wechat_redirect)

**新经济：**[蔚来汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247491009&idx=1&sn=d95f01aea67e68b617cb891be823dd9a&chksm=e9f162e5de86ebf310ec56cbe8c950a69ef5a311b8213bb3adaf843069f9f315a5bede006cdd&scene=21#wechat_redirect)[理想汽车｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485820&idx=1&sn=7aa47ec35feec64e49ea462fe7506864&chksm=e9f17658de86ff4e1bd61f12d70ee8b478cc0ecd8fa7860df72ce2b5dc4b8ac05c77235809d6&scene=21#wechat_redirect)[顺丰｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484829&idx=1&sn=344fb330aac7d722a57a9591a82ca019&chksm=e9f17ab9de86f3af2e6d995b9ff8493c2a4d8e992d30e205904e3bd0ba88075061ea0158cb82&scene=21#wechat_redirect)[京东物流](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486458&idx=1&sn=88058c480d5f163155bb885e3d1fbc2a&chksm=e9f174dede86fdc8defec71ca637bc2a5a9c87dd245c6bf9c51e11f2811fcdbcd507776ccd71&scene=21#wechat_redirect)｜[跨越速运](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect) | [大润发｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247488098&idx=1&sn=cae2b42fd8fdb77ed4e67fc83173291f&chksm=e9f16d46de86e450859f842b471de1ee4e57118901f6f1f7f2c545fefdbdfbe355c9dc3e7965&scene=21#wechat_redirect)[华润万家｜](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487047&idx=1&sn=74201f01931823c9cdc6abc71d3dba68&chksm=e9f17163de86f875439549071e7c79d3c2762cf245b69367be7024ed9d04380816da4348b38d&scene=21#wechat_redirect)[TCL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=1&sn=cb78e47e420191b2345aff1366709384&chksm=e9f16e7dde86e76b77ecfd0d72d164313e18ef2fad887c3e41d30693bfd5e79629018be53a6a&scene=21#wechat_redirect) ｜[万物新生](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490969&idx=1&sn=22a377677f403c37579b2309686d559b&chksm=e9f162bdde86ebab62ce4bd3905d63870947f5b42076fc4d3d1c60a0303e2565a01cb5e2b644&scene=21#wechat_redirect) [| 百草味](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487394&idx=2&sn=242d444b7c02721b503de9383329edcd&chksm=e9f17086de86f990d2e18132c2b9beb3fa3984f5accb03cb34e0d479e8575666426c5508913c&scene=21#wechat_redirect) [| 多点 DMALL](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247484971&idx=1&sn=8f7e7ec424335b81fb34f1da85efbf8f&chksm=e9f1790fde86f019016e67a8a820f866bcbffe484414fa93b403605f957874bb7450672bfc3d&scene=21#wechat_redirect) [|](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247487833&idx=2&sn=7912601d564276718811d53c60116a69&chksm=e9f16e7dde86e76ba4a28a99faf69c33a0553e4d383d4e083c3e0387e1e6d088fd13094c055b&scene=21#wechat_redirect)[酷开科技](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247486836&idx=1&sn=399086d79a513d311083c30ec7746589&chksm=e9f17250de86fb4630b4c3f6d3463c06daa9941d42b8c8cf311e91c0817f403de2d138d3884e&scene=21#wechat_redirect)

**StarRocks 技术内幕：**[极速湖仓神器：物化视图](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490902&idx=1&sn=0bd03778216286548f5d02a2df33466a&chksm=e9f16272de86eb645470a514afb35d6618d805d5358b7afa7b57f897c81ce2902e01d446064d&scene=21#wechat_redirect)｜[存算分离，兼顾降本与增效](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490146&idx=1&sn=771199c6e343612031134c0285fa4881&chksm=e9f16546de86ec50918f2c2ea3cc2eeac8d19655177fab2e9db933f5eb216fe30f2751c108b0&scene=21#wechat_redirect)  ｜[实时更新与极速查询如何兼得](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485927&idx=1&sn=56051046f9eb51a563c6c24eb22c9364&chksm=e9f176c3de86ffd5bab12e9b8657ad58c2b4b4d2741dc0f7de292c800e4ec23cb064955013d0&scene=21#wechat_redirect)｜[Query Cache，一招搞定高并发](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247490921&idx=1&sn=9fc54c02d5395e112d0021686fb1bf6c&chksm=e9f1624dde86eb5be9a97895e1c92c40977b3943af6ad8b99611706092146e0690a3bf2e083d&scene=21#wechat_redirect)｜[资源隔离](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247489066&idx=1&sn=4317418e82af2520ad5c3dd1c3ad5975&chksm=e9f1690ede86e018406d8f5b750a46cafcce21d263ae9794c23c85cc7b35d292053b523cde47&scene=21#wechat_redirect)｜[大数据自动管理](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485727&idx=1&sn=548ed6a15bd4b938cae8551500e3275d&chksm=e9f1763bde86ff2dcea5aacf2e1d8b9401cee6236b41571e241de74178cd856842e4e62fb48a&scene=21#wechat_redirect)｜[查询原理浅析](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247485848&idx=1&sn=90e47d7a46eb120701d28b5acfbcc401&chksm=e9f176bcde86ffaaa0c4a3b686eea5b053ff286164cb9a27e85daf4e42bec08985f5a41975c3&scene=21#wechat_redirect)｜[易用性全面提升](http://mp.weixin.qq.com/s?__biz=MzI1MTYxOTkxNQ==&mid=2247493001&idx=1&sn=16556600807df38a4f6ea0e8197bdfc9&chksm=e9f29aadde8513bb1c7918eeddb7e5c42eb59832173b223e847869a123afbd7f6d9725ac421e&scene=21#wechat_redirect)
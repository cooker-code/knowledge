---
title: 深度解析：Doris主键模型的部分列更新，四种方式全攻略
author: 数仓生态圈
date: 数仓生态圈数仓生态圈
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODY0NjY1Ng==&mid=2247486602&idx=1&sn=09d92ef4373cb32a73bd6d10d543a1f3&chksm=c4450aa46ef36ca7160c58cea6400dffd14f900be08613e852795f4a24201c3ac0ee086b319b&mpshare=1&scene=24&srcid=0430gsZQAOtB7Id4HDFXPAnm&sharer_shareinfo=ff9299d2def2f872d4c30448f154b012&sharer_shareinfo_first=ff9299d2def2f872d4c30448f154b012#rd
---

在处理业务变更时，经常需要对已存在的记录进行更新。Doris的主键模型天然支持数据行的更新与删除，实际应用中，我们往往只需要更新记录中的部分列（如状态、金额），而不是整行替换。如果数据来源进入的顺序不能保证时，也可以通过设置Sequence列来解决。

一、创建测试数据

```
CREATE TABLE IF NOT EXISTS u1(`user_id` LARGEINT NOT NULL,`date` DATE NOT NULL,           `cost` BIGINT NOT NULL,`status` INT NOT NULL )UNIQUE KEY(`user_id`, `date`)DISTRIBUTED BY HASH(`user_id`) BUCKETS 1PROPERTIES ("replication_allocation" = "tag.location.default: 1");insert into u1 values (1,'2026-04-21',10,1),(2,'2026-04-21',12,1),(2,'2026-04-22',18,1),(3,'2026-04-23',20,1);
```

## 方式一：整行覆盖式插入

## 

**适用场景**适用于需要更新的数据列本身较为完整

**操作方式**直接构造包含所有列的完整数据行（即使部分列值未变），执行INSERT。Doris会根据主键`(user_id, date)`自动进行更新。

```
insert into u1 values (1,'2026-04-21',15,1);mysql> select * from u1;+---------+------------+------+--------+| user_id | date       | cost | status |+---------+------------+------+--------+| 1       | 2026-04-21 |   15 |      1 || 2       | 2026-04-21 |   12 |      1 || 2       | 2026-04-22 |   18 |      1 || 3       | 2026-04-23 |   20 |      1 |+---------+------------+------+--------+
```

## 方式二：Stream Load部分列导入

## 

**适用场景**适用于从文件（如CSV、JSON）进行批量更新。

**操作方式**通过Stream Load，在HTTP Header中设置`partial_columns:true`，并在`columns`参数中明确指定CSV文件中的列顺序与对应的表列名。未指定的列将保留原值。

创建u1.csv

```
2,2026-04-21,30
```

执行下面命令导入

```
curl --location-trusted -u root: -H "Expect:100-continue" -H "partial_columns:true" -H "column_separator:," -H "columns:user_id,date,cost" -T u1.csv http://127.0.0.1:8030/api/demo/u1/_stream_load
```

```
mysql> select * from u1;+---------+------------+------+--------+| user_id | date       | cost | status |+---------+------------+------+--------+| 1       | 2026-04-21 |   15 |      1 || 2       | 2026-04-22 |   18 |      1 || 3       | 2026-04-23 |   20 |      1 || 2       | 2026-04-21 |   30 |      1 |+---------+------------+------+--------+
```

## 方式三：INSERT语句部分列更新

## 

必须使用Merge-On-Write（MOW）表，并开启会话变量`enable_unique_key_partial_update=true`。此特性默认关闭，需注意。

```
mysql> insert into u1(`user_id`,`date`,`cost`) values(3,'2026-04-23',40);ERROR 1105 (HY000): errCode = 2, detailMessage = Column has no default value, column=status
```

```
SET enable_unique_key_partial_update=true;mysql> insert into u1(`user_id`,`date`,`cost`) values(3,'2026-04-23',40);mysql> select * from u1;+---------+------------+------+--------+| user_id | date       | cost | status |+---------+------------+------+--------+| 1       | 2026-04-21 |   15 |      1 || 2       | 2026-04-22 |   18 |      1 || 2       | 2026-04-21 |   30 |      1 || 3       | 2026-04-23 |   40 |      1 |+---------+------------+------+--------+
```

## 方式四：使用UPDATE语句

## 

**适用场景**适用于明确的、低频次的精确更新。例如，DBA手动修正数据错误。

```
update u1 set status=2 where user_id=3;
```

## 进阶：处理乱序数据与Sequence列

## 

如果数据到达的顺序不能反映其最终版本（即后生成的数据可能先导入），简单的覆盖逻辑会导致结果错误。

例如：

```
1,2026-04-21,10,11,2026-04-21,12,11,2026-04-21,11,1# 如果按导入顺序覆盖，结果就是最后一条1,2026-04-21,11,1# 但最终的数据应该是1,2026-04-21,12,1
```

**解决方案**：使用带`sequence_col`的表。

该功能为表指定一个版本列（如时间戳、批次号），Doris在合并时会保留该列值最大的行，确保最终结果是“最新”版本。

```
CREATE TABLE IF NOT EXISTS u3(`user_id` LARGEINT NOT NULL,`date` DATE NOT NULL,           `cost` BIGINT NOT NULL,`addtime` DATE NOT NULL )UNIQUE KEY(`user_id`, `date`)DISTRIBUTED BY HASH(`user_id`) BUCKETS 1PROPERTIES ("replication_allocation" = "tag.location.default: 1","function_column.sequence_col" = 'addtime');insert into u3 values (1,'2026-04-21',10,'2026-04-21 10:05:01');
```

导入2条数据（注意时间戳顺序与实际版本相反）：

```
1,2026-04-21,11,2026-04-21 10:05:411,2026-04-21,12,2026-04-21 10:05:31
```

```
curl --location-trusted -u root: -H "Expect:100-continue" -H "function_column.sequence_col:addtime" -H "column_separator:," -H "columns:user_id,date,cost,addtime" -T u3.csv http://127.0.0.1:8030/api/demo/u3/_stream_load
```

查询结果（保留了`addtime`更大的行，即11这条记录）：

```
mysql> select * from u3;+---------+------------+------+---------------------+| user_id | date       | cost | addtime             |+---------+------------+------+---------------------+| 1       | 2026-04-21 |   11 | 2026-04-21 10:05:41 |+---------+------------+------+---------------------+
```

导入需要指定sequence\_col

```
function_column.sequence_col:addtime
```

总结

正常情况下我们一般不会通过insert进行数据更新，通常使用导入方式。但是如果在实时同步场景，需要尽快入库，那么insert的方式还是有一定的优势。update方式几乎不会用于实际工作当中，主要是为了手工修改一些数据做一些临时调整准备的。

---

喜欢数仓方面的技术，可以加作者一起交流
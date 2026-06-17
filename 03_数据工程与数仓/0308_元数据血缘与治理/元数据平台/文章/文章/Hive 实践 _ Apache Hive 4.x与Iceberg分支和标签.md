---
title: Hive 实践 | Apache Hive 4.x与Iceberg分支和标签
author: HBase技术社区
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5OTQ1MDEzMA==&mid=2247492122&idx=1&sn=39f36da681b71d0235490aa745ce30a9&chksm=feb61567c9c19c71f213b6c5045a1fe966c96c2d4457b05ef67d593da58a9340edc14153cba3&mpshare=1&scene=24&srcid=1113hCx9i4ediIjR6yT1c36Y&sharer_shareinfo=ecd77c5c5c46655a044cbd73259c0a37&sharer_shareinfo_first=ecd77c5c5c46655a044cbd73259c0a37#rd
---

对于复杂的快照生命周期管理，Iceberg支持分支(branch)和标签(tag)，这些分支和标签是对具有自己独立生命周期的快照的命名引用，此生命周期由分支和标签级别保留策略控制。分支是快照的独立谱系(lineage)，指向谱系的头部。

# 初始设置

1.创建版本为4.0.0-beta-2-SNAPSHOT 的 docker 环境，参考以下链接可以了解创建docker环境的步骤，将版本指定为 4.0.0-beta-2-SNAPSHOT。

```
https://iceberg.apache.org/hive-quickstart/
```

2.创建一张iceberg表

```
CREATE TABLE test (ID INT) STORED BY ICEBERG TBLPROPERTIES('format-version'='2');
```

3.插入一些数据生成多个快照：

```
INSERT INTO test VALUES (1), (2);  
INSERT INTO test VALUES (3), (4);  
INSERT INTO test VALUES (5), (6);  
INSERT INTO test VALUES (6), (7);
```

4.获取Iceberg table对应的快照列表

```
SELECT * FROM default.test.history;
```

上面会列出该表对应的快照，用来创建分支和标签，以下为示例输出：

注意：每个人的快照ID和时间戳都不同。

# 2 创建分支

可以通过简单的Alter Table…Create Branch…语句从表创建分支，该语句指定表和分支名称以及与应充当分支HEAD的快照相对应的System Version或System Time。 1.使用SYSTEM\_VERSION创建：

```
ALTER TABLE test CREATE BRANCH branch1 FOR SYSTEM_VERSION AS OF 3369973735913135680;
```

以上从表test创建了一个名为branch1的分支，对应到指定的snapshot id(前文history表里的第二行)。 2.使用SYSTEM\_TIME创建：

```
ALTER TABLE test CREATE BRANCH branch2 FOR SYSTEM_TIME AS OF '2023-09-16 09:46:38.939 Etc/UTC';
```

以上从表test创建了一个名为branch2分支，该分支与指定时间戳处的表状态相对应(前文history表里的第三行)。 3.使用默认配置创建： 如果没有提供参数来指定快照版本，则可以创建一个指向Iceberg表的当前主分支（表的当前状态）的分支。

```
ALTER TABLE test CREATE BRANCH branch3;
```

上面创建了一个名为branch3的分支，它与当前表test处于相同的状态。 4.或者可以在创建分支时指定每个分支的快照保留数量，如下所示：

```
ALTER TABLE test CREATE BRANCH branch4 FOR SYSTEM_VERSION AS OF 3369973735913135680 with SNAPSHOT RETENTION 5 SNAPSHOTS;
```

# 列出创建的分支和标签

可以使用iceberg的refs元数据表列出创建的分支和标签。

```
select * from default.test.refs;
```

# 写入分支

数据可以像普通iceberg表一样被摄取到iceberg表分支中，在查询中需要按以下方式指定分支名称。

```
<Database Name>.<Table Name>.branch_<Branch Name>
```

1.插入分支

```
INSERT INTO TABLE default.test.branch_branch1 VALUES (10, 11);
```

上面将值插入到“default”数据库中表“test”的“branch1”中 2.更新分支中的值

```
UPDATE TABLE default.test.branch_branch1 SET ID=20 WHERE ID=10;
```

上面的查询更新“default”数据库中表“test”的“branch1”中的值。 3.删除分支中的值

```
DELETE FROM default.test.branch_branch1 WHERE ID=11;
```

上面的查询删除了“default”数据库中“test”表的“branch1”中的指定值。

除了这些之外，冰山分支还支持所有其他查询，例如 Merge/IOW/Load。

# 查询分支

Iceberg分支像任何其他Hive表一样支持所有查询语句。

```
select * from default.test.branch_branch1;
```

示例输出：

除此之外，Iceberg分支支持所有其他读取查询。

# 快进分支

作为另一个分支的祖先的Iceberg分支可以快进到另一个分支的状态。如下将一些数据插入主分支。

```
INSERT INTO test values (55), (66);
```

现在将之前创建的branch3快进到当前的主分支：

```
ALTER table test EXECUTE FAST-FORWARD 'branch3' 'main';
```

这会将branch3快进到main分支状态，现在查询branch3将显示新插入的记录。

如果未指定第二个分支名称，主分支将快进到指定的分支，如下所示从“branch3”中删除一些值。

```
DELETE FROM default.test.branch_branch3 WHERE ID=66;
```

上面将表test的主分支快进到branch3的状态。

# Cherry-Pick into a Branch

目前Iceberg表只支持在主分支上使用cherry-pick commits，如果我们想回滚到之前的commit，并且只想从未来状态的单个commit中pull in该更改，我们可以cherry-pick该commit。如下获取表的当前历史：

```
SELECT * FROM default.test.history;
```

回滚当前表：

```
ALTER table test EXECUTE ROLLBACK 3369973735913135680;
```

上面将主表回滚到历史列表中的第二个快照。

Cherry-Pick历史表中的倒数第二个快照，那个时候插入了55和56。

```
ALTER table test EXECUTE CHERRY-PICK 8602659039622823857;
```

# 删除分支

可以按如下删除Iceberg分支：

```
ALTER TABLE test DROP BRANCH branch1;
```

上面的查询删除表test的“branch1”，drop branch的语法还支持“IF EXISTS”子句，以防止分支已删除或不存在时出现错误，如下所示：

```
ALTER TABLE test DROP BRANCH IF EXISTS branch1;
```

# 创建标签

可以使用Alter table…Create Tag语句并通过指定表和分支名称以及与标签应引用的快照相对应的System\_version或 System\_time来从表创建标签。 1.使用SYSTEM\_VERSION创建：

```
ALTER TABLE test CREATE TAG tag1 FOR SYSTEM_VERSION AS OF 3369973735913135680;
```

以上语句是从与指定快照ID（历史表中的第二个）对应的表test中创建了一个名为tag1的标签。 2.使用SYSTEM\_TIME创建：

```
ALTER TABLE test CREATE TAG tag2 FOR SYSTEM_TIME AS OF '2023-09-16 09:46:38.939 Etc/UTC';
```

上面的代码从表test中创建了一个名为tag2的标签，该标签与指定时间戳处的表状态相对应（历史表中第三个）。 3.使用默认配置创建： 如果没有提供参数来指定快照版本，则可以创建一个指向Iceberg表的当前主分支（表的当前状态）的标签。

```
ALTER TABLE test CREATE TAG tag3;
```

上面创建了一个名为 tag3 的标签，它与当前test表处于相同的状态。

# 查询标签

Iceberg标签支持各种读取查询，可以通过指定标签名称而不是表名称来执行查询，格式如下。

```
<Database Name>.<Table Name>.tag_<Tag Name>
```

例如：

```
SELECT * FROM default.test.tag_tag1;
```

# 删除标签

使用如下语句删除Iceberg标签：

```
ALTER TABLE test DROP TAG tag1;
```

上面的查询删除了表test对应的tag1，drop tag语法还支持“IF EXISTS”子句，以防止标签已删除或不存在时出现错误，如：

```
ALTER TABLE test DROP TAG IF EXISTS tag1;
```

注意：本文提到的Apache Hive-4.0.0-beta-2版本的一部分功能，预计2023年12月发布。4.0.0-beta-2-SNAPSHOT版本的 docker镜像不是官方Apache Hive版本，仅适用于开发使用。

原文参考：

```
https://medium.com/@ayushtkn/apache-hive-4-x-with-iceberg-branches-tags-3d52293ac0bf
```
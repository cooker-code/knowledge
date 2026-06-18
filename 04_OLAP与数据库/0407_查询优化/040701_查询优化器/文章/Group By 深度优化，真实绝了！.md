---
title: Group By 深度优化，真实绝了！
author: 数据前线
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1NTU3OTk1Ng==&mid=2247490425&idx=1&sn=e9e9b9a62498151b5c9d8e4570de7219&chksm=fbd37422cca4fd3436f53993e29297f4d7c7bb7a103f2dab0a04857e8e3048c163b71b992a02&mpshare=1&scene=24&srcid=0713IHQFXpN8NmbPVBRixWPw&sharer_sharetime=1689208343163&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

### 导读

当我们交友平台在线上运行一段时间后，为了给平台用户在搜索好友时，在搜索结果中推荐并置顶他感兴趣的好友，这时候，我们会对用户的行为做数据分析，根据分析结果给他推荐其感兴趣的好友。

> 这里，我采用最简单的SQL分析法：对用户过去查看好友的性别和年龄进行统计，按照年龄进行分组得到统计结果。依据该结果，给用户推荐计数最高的某个性别及年龄的好友。

那么，假设我们现在有一张用户浏览好友记录的明细表`t_user_view`，该表的表结构如下：

```
CREATE TABLE `t_user_view` (  
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增id',  
  `user_id` bigint(20) DEFAULT NULL COMMENT '用户id',  
  `viewed_user_id` bigint(20) DEFAULT NULL COMMENT '被查看用户id',  
  `viewed_user_sex` tinyint(1) DEFAULT NULL COMMENT '被查看用户性别',  
  `viewed_user_age` int(5) DEFAULT NULL COMMENT '被查看用户年龄',  
  `create_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3),  
  `update_time` datetime(3) DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3),  
  PRIMARY KEY (`id`),  
  UNIQUE KEY `idx_user_viewed_user` (`user_id`,`viewed_user_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

为了方便使用SQL统计，见上面的表结构，我冗余了被查看用户的性别和年龄字段。

我们再来看看这张表里的记录：

现在结合上面的表结构和表记录，我以`user_id=1`的用户为例，分组统计该用户查看的年龄在18 ~ 22之间的女性用户的数量：

```
SELECT viewed_user_age as age, count(*) as num FROM t_user_view WHERE user_id = 1 AND viewed_user_age BETWEEN 18 AND 22 AND viewed_user_sex = 1 GROUP BY viewed_user_age
```

得到统计结果如下：

可见：

* 该用户查看年龄为18的女性用户数为2
* 该用户查看年龄为19的女性用户数为1
* 该用户查看年龄为20的女性用户数为3

所以，`user_id=1`的用户对年龄为20的女性用户更感兴趣，可以更多推荐20岁的女性用户给他。

> 如果此时，`t_user_view`这张表的记录数达到千万规模，想必这条SQL的查询效率会直线下降，为什么呢？有什么办法优化呢？

想要知道原因，不得不先看一下这条SQL执行的过程是怎样的？

### Explain

我们先用`explain`看一下这条SQL：

```
EXPLAIN SELECT viewed_user_age as age, count(*) as num FROM t_user_view WHERE user_id = 1 AND viewed_user_age BETWEEN 18 AND 22 AND viewed_user_sex = 1 GROUP BY viewed_user_age
```

执行完上面的`explain`语句，我们得到如下结果：

在`Extra`这一列中出现了三个`Using`，这3个`Using`代表了《导读》中的`groupBy`语句分别经历了3个执行阶段：

1. Using where：通过搜索可能的`idx_user_viewed_user`索引树定位到满足部分条件的`viewed_user_id`，然后，回表继续查找满足其他条件的记录
2. Using temporary：使用临时表暂存待`groupBy`分组及统计字段信息
3. Using filesort：使用`sort_buffer`对分组字段进行排序

这3个阶段中出现了一个名词：`临时表`。这个名词我在《MySQL分表时机：100w？300w？500w？都对也都不对！》一文中有讲到，这是MySQL连接线程可以独立访问和处理的内存区域，那么，这个临时表长什么样呢？

> 下面我就先讲讲这张MySQL的临时表，然后，结合上面提到的3个阶段，详细讲解《导读》中SQL的执行过程。

### 临时表

我们还是先看看《导读》中的这条包含`groupBy`语句的SQL，其中包含一个分组字段`viewed_user_age`和一个统计字段`count(*)`，这两个字段是这条SQL中统计所需的部分，如果我们要做这样一个统计和分组，并把结果固化下来，肯定是需要一个内存或磁盘区域落下第一次统计的结果，然后，以这个结果做下一次的统计，因此，像这种存储中间结果，并以此结果做进一步处理的区域，MySQL叫它`临时表`。

刚刚提到既可以将中间结果落在内存，也可以将这个结果落在磁盘，因此，在MySQL中就出现了两种临时表：`内存临时表`和`磁盘临时表`。

#### 内存临时表

什么是内存临时表？在早期数据量不是很大的时候，以存储分组及统计字段为例，那么，基本上内存就可以完全存放下分组及统计字段对应的所有值，这个存放大小由`tmp_table_size`参数决定。这时候，这个存放值的内存区域，MySQL就叫它内存临时表。

此时，或许你已经觉得MySQL将中间结果存放在内存临时表，性能已经有了保障，但是，在《MySQL分表时机：100w？300w？500w？都对也都不对！》中，我提到过内存频繁的存取会产生碎片，为此，MySQL设计了一套新的内存分配和释放机制，可以减少甚至避免临时表内存碎片，提升内存临时表的利用率。

> 此时，你可能会想，在《为什么我调大了sort\_buffer\_size，并发量一大，查询排序慢成狗？》一文中，我讲了用户态的内存分配器：`ptmalloc`和`tcmalloc`，无论是哪个分配器，它的作用就是避免用户进程频繁向Linux内核申请内存空间，造成`CPU`在用户态和内核态之间频繁切换，从而影响内存存取的效率。用它们就可以解决内存利用率的问题，为什么MySQL还要自己搞一套？

或许MySQL的作者觉得无论哪个内存分配器，它的实现都过于复杂，这些复杂性会影响MySQL对于内存处理的性能，因此，MySQL自身又实现了一套内存分配机制：`MEM_ROOT`。它的内存处理机制相对比较简单，内存临时表的分配就是采用这样一种方式。

下面，我就以《导读》中的SQL为例，详细讲解一下分组统计是如何使用`MEM_ROOT`内存分配和释放机制的？

##### MEM\_ROOT

我们先看看`MEM_ROOT`的结构，`MEM_ROOT`设计比较简单，主要包含这几部分，如下图：

free：一个单向链表，链表中每一个单元叫`block`，`block`中存放的是空闲的内存区，每个`block`包含3个元素：

* left：`block`中剩余的内存大小
* size：`block`对应内存的大小
* next：指向下一个`block`的指针

如上图，`free`所在的行就是一个`free`链表，链表中每个箭头相连的部分就是`block`，`block`中有`left`和 `size`，每个`block`之间的箭头就是`next`指针

* **used：**一个单向链表，链表中每一个单元叫`block`，`block`中存放已使用的内存区，同样，每个`block`包含上面3 个元素
* **min\_malloc：**控制一个 `block` 剩余空间还有多少的时候从`free`链表移除，加入到`used`链表中
* **block\_size：**`block`对应内存的大小
* **block\_num：**`MEM_ROOT` 管理的`block`数量
* **first\_block\_usage：**`free`链表中第一个`block`不满足申请空间大小的次数
* **pre\_alloc：**当释放整个`MEM_ROOT`的时候可以通过参数控制，选择保留`pre_alloc`指向的`block`

下面我就以《导读》中的分组统计SQL为例，看一下`MEM_ROOT`是如何分配内存的？

###### 分配

1. 初始化`MEM_ROOT`，见上图：

* `min_malloc = 32`
* `block_num = 4`
* `first_block_usage = 0`
* `pre_alloc = 0`
* `block_size = 1000`
* `err_handler = 0`
* `free = 0`
* `used = 0`

2. 申请内存，见上图：

由于初始化`MEM_ROOT`时，`free = 0`，说明`free`链表不存在，故向Linux内核申请4个大小为`1000/4=250`的`block`，构造一个`free`链表，如上图，链表中包含4个`block` ，结合前面`free`链表结构的说明，每个`block`中`size`为250，`left`也为250

3. 分配内存，见上图：

(1) 遍历`free`链表，从`free`链表头部取出第一个`block`，如上图向下的箭头

(2) 从取出的`block`中划分`220`大小的内存区，如上图向右的箭头上面`-220`，`block`中的`left`从`250`变成`30`

(3) 将划分的`220`大小的内存区分配给SQL中的`groupby`字段`viewed_user_age`和统计字段`count(*)`，用于后面的统计分组数据收集到该内存区

(4) 由于第(2)步中，分配后的`block`中的`left`变成`30`，`30 < 32`，即小于第(1)步中初始化的`min_malloc`，所以，结合上面`min_malloc`的含义的讲解，该`block`将插入`used`链表尾部，如上图底部，由于`used`链表在第(1)步初始化时为0，所以，该`block`插入`used`链表的尾部，即插入头部

###### 释放

下面还是以《导读》中的分组统计为例，我们再来看一下`MEM_ROOT`是如何释放内存的？

如上图，`MEM_ROOT`释放内存的过程如下：

1. 遍历`used`链表中，找到需要释放的`block`，如上图，`block(30,250)`为之前已分配给分组统计用的`block`
2. 将`block(30,250)`中的`left + 220`，即`30 + 220 = 250`，释放该`block`已使用的`220`大小的内存区，得到释放后的`block(250,250)`
3. 将`block(250,250)`插入`free`链表尾部，如上图曲线箭头部分

通过`MEM_ROOT`内存分配和释放的讲解，我们发现`MEM_ROOT`的内存管理方式是在每个`Block`上连续分配，内部碎片基本在每个`Block`的尾部，由`min_malloc`成员变量控制，但是`min_malloc`的值是在代码中写死的，有点不够灵活。所以，对一个`block`来说，当`left`小于`min_malloc`，从其申请的内存越大，那么`block`中的`left`值越小，那么，该`block`的内存利用率越高，碎片越少，反之，碎片越多。这个写死是MySQL的内存分配的一个缺陷。

#### 磁盘临时表

当分组及统计字段对应的所有值大小超过`tmp_table_size`决定的值，那么，MySQL将使用磁盘来存储这些值。这个存放值的磁盘区域，MySQL叫它磁盘临时表。

我们都知道磁盘存取的性能一定比内存存取的性能差很多，因为会产生磁盘IO，所以，一旦分组及统计字段不得不写入磁盘，那性能相对是很差的，所以，我们尽量调大参数`tmp_table_size`，使得组及统计字段可以在内存临时表中处理。

### 执行过程

无论是使用内存临时表，还是磁盘临时表，临时表对组及统计字段的处理的方式都是一样的。《导读》中我提到想要优化《导读》中的那条SQL，就需要知道SQL执行的原理，所以，下面我就结合上面讲解的临时表的概念，详细讲讲这条SQL的执行过程，见下图：

1. 创建临时表`temporary`，表里有两个字段`viewed_user_age`和`count(*)`，主键是`viewed_user_age`，如上图，倒数第二个框`temporary`表示临时表，框中包含两个字段`viewed_user_age`和`count(*)`，框内就是这两个字段对应的值，其中`viewed_user_age`就是这张临时表的主键
2. 扫描表辅助索引树`idx_user_viewed_user`，依次取出叶子节点上的`id`值，即从索引树叶子节点中取到表的主键id。如上图中的`idx_user_viewed_user`框就是索引树，框右侧的箭头表示取到表的主键id
3. 根据主键id到聚簇索引`cluster_index`的叶子节点中查找记录，即扫描`cluster_index`叶子节点：

   (1) 得到一条记录，然后取到记录中的`viewed_user_age`字段值。如上图，`cluster_index`框，框中最右边的一列就是`viewed_user_age`字段的值

   (2) 如果临时表中没有主键为`viewed_user_age`的行，就插入一条记录 (`viewed_user_age`, 1)。如上图的`temporary`框，其左侧箭头表示将`cluster_index`框中的`viewed_user_age`字段值写入`temporary`临时表

   (3) 如果临时表中有主键为`viewed_user_age`的行，就将`viewed_user_age`这一行的`count(*)`值加 1。如上图的`temporary`框
4. 遍历完成后，再根据字段`viewed_user_age`在`sort_buffer`中做排序，得到结果集返回给客户端。如上图中的最右边的箭头，表示将`temporary`框中的`viewed_user_age`和`count(*)`的值写入`sort_buffer`，然后，在`sort_buffer`中按`viewed_user_age`字段进行排序

> 通过《导读》中的SQL的执行过程的讲解，我们发现该过程经历了4个部分：`idx_user_viewed_user`、`cluster_index`、`temporary`和`sort_buffer`，对比上面`explain`的结果，其中前2个就对应结果中的`Using where`，`temporary`对应的是`Using temporary`，`sort_buffer`对应的是`Using filesort`。

### 优化方案

此时，我们有什么办法优化这条SQL呢？

> 既然这条SQL执行需要经历4个部分，那么，我们可不可以去掉最后两部分呢，即去掉`temporary`和`sort_buffer`？

答案是可以的，我们只要给SQL中的表`t_user_view`添加如下索引：

```
ALTER TABLE `t_user_view` ADD INDEX `idx_user_age_sex` (`user_id`, `viewed_user_age`, `viewed_user_sex`);
```

你可以自己尝试一下哦！用`explain`康康有什么改变！

### 小结

本章围绕《导读》中的分组统计SQL，通过`explain`分析SQL的执行阶段，结合临时表的结构，进一步剖析了SQL的详细执行过程，最后，引出优化方案：**新增索引，避免临时表对分组字段的统计，及`sort_buffer`对分组和统计字段排序**。

当然，如果实在无法避免使用临时表，那么，**尽量调大`tmp_table_size`，避免使用磁盘临时表统计分组字段。**

### 思考题

为什么新增了索引`idx_user_age_sex`可以避免临时表对分组字段的统计，及`sort_buffer`对分组和统计字段排序？

提示：结合索引查找的原理。

### *来源：juejin.cn/post/6957696820621344775*

### 

```
```
#### 

#### 更多精彩内容，请关注「数据前线」
```

#### ``` ``` ``` ```
```
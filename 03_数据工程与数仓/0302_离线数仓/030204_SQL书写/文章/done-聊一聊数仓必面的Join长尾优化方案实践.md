> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQLJoin语义与长尾治理|SQLJoin语义与长尾治理]]
---
title: 聊一聊数仓必面的Join长尾优化方案实践
author: 涤生大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490166&idx=1&sn=d7bf9cf4e2dce1149e058d19086404c2&chksm=ceec8718c9d7885e351186791203ec467b560e7a4e720da435efa317fe64ca0bbdbc2f82072b&mpshare=1&scene=24&srcid=12137JCl53q4E0xkHPWt6ChG&sharer_shareinfo=19b9d6d76e294becac355566b70aed6d&sharer_shareinfo_first=19b9d6d76e294becac355566b70aed6d#rd
---

SQL在Join执行阶段会将Join Key相同的数据分发到同一个执行Instance上处理。如果某个Key上的数据量比较多，会导致该Instance执行时间比其他Instance执行时间长。其表现为：执行日志中该Join Task的大部分Instance都已执行完成，但少数几个Instance一直处于执行中，这种现象称之为长尾。 

**背景信息**

**1 背景**

因为数据量倾斜导致长尾的现象比较普遍，严重影响任务的执行时间，尤其是在双十一等大型活动期间，长尾程度比平时更为严重。譬如，某些大型店铺的浏览PV远远超过一般店铺的PV，当用浏览日志数据和卖家维表关联，会按照卖家ID做分发，导致某个Instance处理的数据量远远超过其他Instance，而整个任务会因为这个长尾的Instance无法结束。

**2 目的**

本文的目的就是针对ODPS SQL执行中Join阶段发生数据倾斜时给出对应的解决方法。本文主要讲述2种常见的倾斜场景。

1. Join的某路输入比较小，避免分发引起的长尾 ，可以采用Mapjoin。
2. Join的每路输入都较大，且长尾是由于热点值导致，可以将热点值和非热点分别处理，再合并数据。

下面会针对这2种场景给出具体的解决方案。在讲方案之前，先了解一下怎么确认Join是否发生数据倾斜。

1.打开ODPS SQL执行时产生的Log View日志，点开日志会看到每个Fuxi Task的详细执行信息，如下图所示：

2.看上图所示红色框，表示有115个Instance长尾，再点击红色框中的，查看stdout中Instance的读入数据量，如：，表示Join的一路输入读取的数据量是1389941257行。如果Long-Tails中Instance读入的数据量远超过其他Instance读取的数据量，则表示是因为数据量导致长尾。

以下面logview为例，耗时最大Fuxi Task的为J5。

**3 方案**

**Mapjoin方案**

Join倾斜时，如果某路输入比较小，可以采用Mapjoin避免倾斜。Mapjoin的原理是将Join操作提前到Map端执行，这样可以避免因为分发Key不均匀导致数据倾斜。但是Mapjoin的使用有限制，必须是Join中的从表比较小才可用。所谓从表，即Left Outer Join中的右表，或者Right Outer Join中的左表。Mapjoin的使用方法非常简单，在代码中selelct后加上 /\*+ mapjoin(b) \*/ 即可，其中b代表小表（或者是子查询）的别名。如：

```
select   /*+mapjoin(b)*/         a.c2        ,b.c3from        (select   c1                 ,c2         from     t1         ) aleft outer join        (select   c1                 ,c3         from     t2         ) bon       a.c1 = b.c1;
```

另外，Mapjoin使用的时候，对小表的大小有限制，默认小表读入内存后的大小不能超过512M，但是用户可以通过参数设置：set odps.sql.mapjoin.memory.max=2048加大内存，最大为2048M。

**Join因为热点值导致长尾**

a.手动切分热值

如果是因为热点值导致长尾，并且Join的输入比较大无法用Mapjoin，可以先将热点Key取出，对于主表数据用热点Key切分成热点数据和非热点数据两部分分别处理，最后合并。以淘宝的PV日志表关联商品维表取商品属性为例：

* 取热点Key：将PV大于50000的商品ID取出到临时表

```
insert overwrite table topk_itemselect  item_idfrom  (    select      item_id,      count(1) as cnt    from      dwd_tb_log_pv_di    where      ds = '${bizdate}'      and url_type = 'ipv'      and item_id is not null    group by      item_id  ) awhere  cnt >= 50000
```

* 取出非热点数据

将主表（dwd\_tb\_log\_pv\_di）和热点key表（topk\_item）外关联后通过条件b1.item\_id is null，取关联不到的数据即非热点商品的日志数据，此时需要用Mapjoin。再用非热点数据关联商品维表，因为已经排除了热点数据，不会存在长尾。

```
select...from  (    select      *    from      dim_tb_itm    where      ds = '${bizdate}'  ) a  right outer join (    select      /*+mapjoin(b1)*/      b2.*    from      (        select          item_id        from          topk_item        where          ds = '${bizdate}'      ) b1      right outer join (        select          *        from          dwd_tb_log_pv_di        where          ds = '${bizdate}'          and url_type = 'ipv'      ) b2 on b1.item_id = coalesce(        b2.item_id,        concat("tbcdm", rand())        where          b1.item_id is null      ) l on a.item_id = coalesce(        l.item_id,        concat("tbcdm", rand())
```

* 取出热点数据

将主表（dwd\_tb\_log\_pv\_di）和热点Key表（topk\_item）内关联，此时需要用Mapjoin，取到热点商品的日志数据。同时，需要将商品维表（dim\_tb\_itm）和热点Key表（topk\_item）内关联，取到热点商品的维表数据，然后将第一部分数据外关联第二部分数据，因为第二部分只有热点商品的维表，数据量比较小，可以用Mapjoin避免长尾。

```
select  /*+mapjoin(a)*/...from  (    select      /*+mapjoin(b1)*/      b2.*    from      (        select          item_id        from          topk_item        where          ds = '${bizdate}'      ) b1      join (        select          *        from          dwd_tb_log_pv_di        where          ds = '${bizdate}'          and url_type = 'ipv'          and item_id is not null      ) b2 on (b1.item_id = b2.item_id)  ) l  left outer join (    select      /*+mapjoin(a1)*/      a2.*    from      (        select          item_id        from          topk_item        where          ds = '${bizdate}'      ) a1      join (        select          *        from          dim_tb_itm        where          ds = '${bizdate}'      ) a2 on (a1.item_id = a2.item_id)  ) a on a.item_id = l.item_id
```

* 将上面2、3的数据通过union all合并后即得到完整的日志数据，并且关联了商品的信息。

ODPS也有专门的参数用来解决长尾问题，如下：

开启功能：set odps.sql.skewjoin=true/false

设置倾斜的Key及对应的值：set odps.sql.skewinfo=skewed\_src:(skewed\_key) [("skewed\_value")]，其中skewed\_key代表倾斜的列，skewed\_value代表倾斜列上的倾斜值。

参数设置好处很明显，简单方便。坏处是如果倾斜的值发生变化需要修改代码，而且一般是无法提前知道变化，另外，如果倾斜的值比较多也不方便在参数中设置。需要根据实际情况选择拆分代码或者参数设置。

**Skew Join Hint避免热值倾斜**

使用方法

```
-- 方法1：hint表名（注意hint的是表的alias）select  /*+ skewjoin(a) */  *from  T0 a  join T1 b on a.c0 = b.c0  and a.c1 = b.c1;-- 方法2：hint表名和认为可能产生倾斜的列，下面的case认为a的c0和c1列存在数据倾斜select  /*+ skewjoin(a(c0, c1)) */  *from  T0 a  join T1 b on a.c0 = b.c0  and a.c1 = b.c1  and a.c2 = b.c2;-- 方法3：hint表名和列，并提供发生倾斜的key值（注意如果是String类型，需要加上引号）  -- 下面的例子是认为，(a.c0=1 and a.c1="2") 和 (a.c0=3 and a.c1="4"）的值都存在数据倾斜select  /*+ skewjoin(a(c0, c1)((1, "2"), (3, "4"))) */  *from  T0 a  join T1 b on a.c0 = b.c0  and a.c1 = b.c1  and a.c2 = b.c2;
```

使用方法的区别说明：

1.直接指定skewvalue（方法3）效率会比不指定的高（方法1和方法2）；

2.配合analyze和freeride收集topk，可能不需要设置hint也能自动生效skewjoin。

**倾斜定位**

以下面logview为例，耗时最大Fuxi Task的为J5。

选中J5后，观察J5的instances，可以看到J5\_3\_4#215\_0的耗时最长，且I/O record和I/OBytes远远大于其他instance。

因此认为J5节点发生了数据倾斜。此时需要确定倾斜的是哪个join，我们可以打开倾斜instance的StdOut（上图黄色框选的地方）和任一个不倾斜节点的StdOut，会显示下图，大多数情况下网页无法显示所有Stdout的内容，可以点击Download，查看全部内容。

Download的内容如下图所示，可以看到倾斜instance的StreamLineRead7的read count远远大于非倾斜instance，因此可以认为StreamLineWrite7到SreamLineRead7的这个shuffle发生了倾斜。

此时，我们再回到DAG图，可以在下图界面，鼠标右键选择『expland all』

就可以找到下图的，StreamLineWrite7和StreamLineRead7。

可以看到MergeJoin2发生了倾斜，倾斜的是MergeJoin2的StreamLineRead7这一路。此时，我们可以进一步追溯StreamLineRead7的输入节点，可以看到是dim\_hm\_item和dim\_tb\_itm\_brandjoin之后的结果，再与StreamLine4（追溯input后发现是dim\_tb\_brand表）进行join，即MergeJoin2。

此时就可以根据这些表名，回到sql中定位，可以发现是框选中的这个left outerjoin发生了倾斜，而倾斜的是t1（黄色框选）这个表，因此，在sql中添加/ \_\*+ skewjoin(t1)\_ \*/即可。

**原理**

定义热值key：出现次数很多的key值，例如下图中红色部分，a.c0=1 and a.c1=2 有10000行，a.c0=3 and a.c1 =4有9000行。

在没加skew join hint的情况下，将表T0和表T1进行Join，由于T0和T1的数量都很大，只能进行Merge Join，因此相同的热值都会shuffle到一个节点，导致数据倾斜，如上图所示。

加hint后，Optimizer会运行一个Aggregate动态获取重复行数前20的热值。然后Optimizer会将表T0中属于热值的值（数据A）、**T0中不属于热值的值**（数据B）拆分。将表T1中能与**T0中属于热值的值**Join上的值（数据C）、表T1中与T0属于热值的值Join不上的值（数据D）进行拆分。然后将数据A与数据C进行MapJoin（由于数据C量很少，可以进行Map Join），将数据B和数据D进行Merge Join。最后将Map Join和Merge Join的结果Union，生成最后的结果，如下图所示。

**注意事项**

1.支持的Join类型：Inner Join可以对Join两侧表中的任意一侧进行Hint；Left Join、Semi Join和Anti Join只可以Hint左侧表；Right Join只可以Hint右侧表；Full Join不支持Skew Join Hint；

2.建议只对一定会出现数据倾斜的Join添加Hint，因为Hint会运行一个Aggregate，存在一定代价；

3.被Hint的Join的Left Side Join Key的类型需要与Right Side Join Key的类型一致，否则skew join hint不生效。例如上例中的a.c0与b.c0的类型需要一致，a.c1与b.c1的类型需要一致；此时，可以通过在子查询中将join key进行cast从而保持一致，例如：

```
create table T0(c0 int, c1 int, c2 int, c3 int);create table T1(c0 string, c1 int, c2 int);-- 方法1：select  /*+ skewjoin(a) */  *from  T0 a  join T1 b on cast(a.c0 as string) = cast(b.c0 as string)  and a.c1 = b.c1 -- 方法2：select  /*+ skewjoin(b) */  *from  (    select      cast(a.c0 as string) as c00    from      T0 a  ) b  join T1 c on b.c00 = c.c0;
```

4.原理解释中提到，Optimizer会运行一个Aggregate获取前20的热值，这个20是一个默认值，可以通过添加flag：set odps.optimizer.skew.join.topk.num = 10; 进行设置；

5.目前只支持对Join其中一侧进行hint；

6.被hint的join一定要有left key = right key，不支持笛卡尔积join；

7.与其他hint一起使用的方法如下，但需要注意，被map join hint的join不能再被skew join hint；

```
select  /*+ mapjoin(c), skewjoin(a) */  *from  T0 a  join T1 b on a.c0 = b.c3  join T2 c on a.c0 = c.c7;
```

8.可以在Logview的Json Summary中搜索是否出现topk\_agg字段判断skew join hint是否生效，如下图所示：

**4 影响及思考**

当大表和大表Join时因为热点值发生倾斜，虽然可以通过修改代码解决，但是修改起来很麻烦，代码改动也很大，且影响阅读。而ODPS现有的参数设置使用不够灵活，倾斜值多的时候不可能将所有值都列在参数中，且倾斜值可能经常变动。因此，需要寻求更智能的解决方法，如生成统计信息，ODPS内部根据统计信息来自动生成解决倾斜的代码。

**5 分布式join的实现**

根据数据大小和分布等特点，分布式的join，可以有许多不同的算法实现及物理执行方式，包括broadcast join，以及sort merge join等等，对于算法的选择，SQL的优化器可以根据本身的cost model做出预判，也可以基于执行框架的动态能力，实现动态选择的conditional join。broadcast join因为不涉及数据的shuffle，也一般不会有数据倾斜分配不均的情况，但是其有特定的使用范围，也就是要求有一边为全表可单机装载的小表输入。在这里我们主要讨论的是分布式系统中对各种数据规模均适用的sort merge join。

在经典的分布式的sort merge join实现中，需要将上游多个计算节点的输出数据，按照join key做shuffle，从而确保相同join key的数据，能够被发送到同一个下游join计算节点。这个shuffle的过程，也被称为partitioning，而partition的方式，和join的并发度是相关的。比如在下面图1的例子中，我们就展示了一个最简单的两路输入的join操作。其物理执行计划中，M1的并发度为3，M2的并发度为2，而下游的join stage（J3）的并发选择为5。在这么一个执行计划中，M1和M2的输出，都会被按照并发5来做partitioning，从而保证它们的输出数据都进入5个分片。这个shuffle/partition模式下，保障了相同partition （也就是相同join key）的数据都能发送到同一个J3计算节点来处理，例如M1 partition 0和M2 partition 0数据，都发送给Join节点的instance #0处理（1、2、3、4同理）。

**6. Join运行中的数据倾斜**

“**确保同一个partition的数据都分配到同一个计算节点**”的实现，可以说是从map-reduce时代就开始的，大数据系统“分而治之”思想的经典体现。当所有数据分片的数据量都较为平均时，这一实现也的确具有很好的适用性：数据的平均分布，使得每个join计算节点都能在预期的运行时间内完成计算。但是当数据的分布不均匀的时候，这个经典算法的局限也同样明显。例如图2所展示的，当M1输出数据的partition #3和#4，以及M2输出的partition #4，发生了倾斜，后果就是其对应的J3计算节点（join instance #3 和 #4），都会因为其中所需处理的数据量不同程度的暴涨，而导致严重的长尾，甚至有可能发生Out-of-Memory（OOM）而失败。

尤其需要指出的是，对于Join操作，由于其多路输入的属性，不同路的输入数据，都有可能发生倾斜，例如在图2的例子中，我们可以看到，处理partition #3的join计算节点，只有来自M1的一路数据有倾斜，而处理patition #4的join计算节点，则因为其来自M1和M2的两路数据都发生了倾斜，其长尾现象可能会更加显著。 

**7 Adaptive Join对于倾斜数据分片的自适应处理**

分布式Join skew的问题，在业界一直都是一个比较具有挑战性的问题，大多数的解决方案集中在对执行计划的预先改写，在优化器层面就尽量避免skew的出现。这种实现的问题主要有两方面，一来是这种实现基于精准的数据统计信息或预判，这个过程中所需的统计信息，在大规模生产系统上经常是缺失或者不尽准确的，尤其是对于非源表数据的特性，其统计质量随着数据经过中间计算转换的次数，呈现*指数级别的下降*，当join操作离根节点（源表数据）比较远时，这种预判基本是不可用的。二来对于有些数据的倾斜，本身是客观存在的，这样子的倾斜，与业务逻辑和数据特性有关，是无法完全避免的。对于join上数据倾斜的处理，在MaxCompute系统中，我们也提供了skew join hint的功能。该功能允许用户在能预先定位join skew发生的位置时，通过提供hint手动指定需要特殊处理的输入，并通过聚合+broadcast join的执行计划，来避免skew的产生。但是这样的方式只能通过对指定query进行分析和改写（添加hint）来实现，对于用户的要求也比较高。在这里，我们基于DAG2.0提供的执行引擎动态能力，期望对于产生数据倾斜的join操作，能做到**自动检测，以及自适应消除**。

首先对于运行DAG2.0引擎上的大数据作业，执行框架本身提供了对运行中的计算节点的产生数据，进行详细的统计信息收集能力，这也是我们实现adaptive shuffle, conditional join，以及动态并发策略优化等一系列动态图能力的数据基础。同样的能力，也使得我们对于join场景上，能基于收集到的多路join上游节点输出的数据分布情况，来做自适应的判断和调整。对于join skew，为了避免单个计算节点上处理大量倾斜数据，我们需要打破的是“**同一个partition的数据都必须分配到同一个计算节点**”这么一个前提。也就是说，如果有数据倾斜的出现，需要对倾斜的partition分片做自适应的切割，分配到多个计算节点。但是与此同时，为了保证join操作的正确性，还需要确保对于**一个partition，每一路的输入数据，与需要和它join的其他路数据，都能相遇一次且仅有一次**。这要求执行框架能够对与join输入的shuffle pattern，做出针对性的自适应改造。

具体以上述图中讨论的例子而言，执行框架在收集数据统计信息后做分析，能自动发现M1的输出partition #3，#4以及M2 partition #4存在（不同程度的）倾斜。这时执行引擎会根据适合单个计算节点的数据大小（线上默认为128M），对这些skew的partition进行切分和重新分配，从而保证每个join计算节点，处理的数据量都是合适均匀的。比如在图2中，M1输出的partition #3被拆成2份，partition #4被拆成3份；而M2的partition #4则被拆成2份。这些从一个partition拆分出来的分块，被称为“partition range”。与此同时，为了保障切分后的多个partition range被充分正确的被join处理，框架还要确保每个partition range，都能拿到其他路join对应的输入partition数据。具体来说，

* 如果join上游输出的某一个partition，只有一路发生skew需要被切割成N个range，而其他路均无skew，那执行引擎会分配N个join计算节点来处理这个partition。skew一路切割后的每个partition range被分配给一个计算节点，而其他路上的对应partition数据则被broadcast给所有N个节点处理。比如图3中的partition #3：因为M1的partition #3被切成2个range，所以分配了2个join节点，而M2对应的partition #3没有skew，就被完整的broadcast给这2个计算节点，来完成join计算。
* 如果join上游输出的某一partition，在多路均有skew，那处理就会稍微复杂一些。这里还以图3中的场景为例。因为partition #4在M1和M2的输出中都是skewed（但程度不同）的，这两部分数据都会被拆开。同时为了满足正确性条件，每个切开的partition range，都需要被broadcast到所有的其他路上的同个partition的不同range上。这个例子中，join对于partition #4的处理，就需要分配P x Q个计算节点，其中P和Q分别是M1和M2在这个paritition #4上的输出需要被拆分的range个数：在这个例子中，P=3， Q=2。
* 其他partitions，不涉及输入数据的倾斜，则还是维持原来的shuffle和数据排布方式正常执行。

也就是说，对于图3的这个场景，最终J3的并发度，会从原始的5，动态的根据上游M1和M2的输出数据分布情况，被调整为 1 + 1 + 1 +（2x1）+（3x2）= 11。当然，这里如果有数据patition特别小，那我们的算法还能自动实现多个小partition的合并（适用类似动态并发调整的算法），在这里就不再展开做特别的讨论。可以看出，这个例子中，对于join stage J3的最终并发调整，不仅涉及到合适并发度的计算，以及动态调整资源申请的数目，**其核心的算法和价值**，更多在于整个过程的自动化，以及执行框架对数据灵活编排和对复杂shuffle模式的支持。这里的shuffle模式，已经远远超越了经典map-reduce的full shuffle pattern，而是采用了full shuffle + 分层的broadcast shuffle的混合（Partial Replication Partial Redistribution），这也是执行框架的动态定制性提供的能力。

涤生大数据往期精彩推荐

1.[企业数仓DQC数据质量管理实践篇](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486027&idx=1&sn=bb92fa17fb2a12fb70ba8f4e2862699d&chksm=cf623f19f815b60fd49d9b18da21f73795a92566f95b9ff41046e7af0e72840b41b42a21f991&scene=21#wechat_redirect)

2.[企业数据治理实战总结--数仓面试必备](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486189&idx=1&sn=1360271d53b9c3a6e6bd16041ad8dd99&chksm=cf623fbff815b6a9b1ec61726e4b65b1495e27673b4d08b280d57924082b79755bc17462d24e&scene=21#wechat_redirect)

3.[OneData理论案例实战—企业级数仓业务过程](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486715&idx=1&sn=0a5d160583b8ddba256f5edf5c34767e&chksm=cf6239a9f815b0bf81fecfe3ecacd118a363b0d6912c5bc9bce094926a7f5f619abd31a8a044&scene=21#wechat_redirect)

4.[中大厂数仓模型规范与度量指标有哪些？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247486864&idx=1&sn=29f5ca1ee9ff947b4248a4e653706a4d&chksm=cf6238c2f815b1d42135e35800f77713bd1f2e7913fc473be0ee7ea05da45a145829bd7ea8bb&scene=21#wechat_redirect)

5.[手把手教你搭建用户画像系统（入门篇上）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487012&idx=1&sn=6d12ebda43874d05c05e8cd2a24ec5c6&chksm=cf623b76f815b2602eb9b5da67d07a0e51575166ac1e31d13f42895239872b4be8a46ac1cf87&scene=21#wechat_redirect)

6.[手把手教你搭建用户画像系统（入门篇下）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487053&idx=1&sn=ef949d4b7474526519998ff332abc4f0&chksm=cf623b1ff815b2091940f96db6623dc1c803be662c5d10a6d56ee6744cbeb36be00e111a8e9d&scene=21#wechat_redirect)

7.[SQL优化之诊断篇：快速定位生产性能问题实践](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487054&idx=1&sn=bb4a0df0fb986f9f3706f39b997771c7&chksm=cf623b1cf815b20a32ebd5546f3c92ad91617e6c03e4e28fb9e69a40425a88ce93cc805e7861&scene=21#wechat_redirect)

8.[SQL之优化篇：一文搞懂如何优化线上任务性能，增效降本！](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487073&idx=1&sn=230f8260142d9f8c26c871f4e580a322&chksm=cf623b33f815b22584a1fbd4ae7e9947221defa7aca0c6f5d4beb446f8101ffd469b1c882fb5&scene=21#wechat_redirect)

9.[新能源趋势下一个简单的数仓项目，助力理解数仓模型](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487251&idx=1&sn=19c8a32f674a1de5a9ac081673d200ac&chksm=cf623a41f815b357d81498488c623180b40ce7522abebb3a17c92b01c112f29f6d8ea97e5c57&scene=21#wechat_redirect)

10.[基于FlinkSQL +Hbase在O2O场景营销域实时数仓的实践](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487723&idx=1&sn=30f99be1730dfbfa303741f46b81c3ed&chksm=cf6225b9f815acafcc627bf071999209090beef02bf56e8a5eb4994bcd52d36aa977361265ef&scene=21#wechat_redirect)

11.[开发实战角度：distinct实现原理及具体优化总结](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487792&idx=1&sn=5e097df0c88e0a0aa3f2a3d0b5103f62&chksm=cf622462f815ad74e0728dd7fae8c04ae2a219cacf959f16cc79eb7a25da8036ae744236bf3d&scene=21#wechat_redirect)

12.[涤生大数据实战：基于Flink+ODPS历史累计计算项目分析与优化（一）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487866&idx=1&sn=041685cb5b2135e4d75b7aab49e36347&chksm=cf622428f815ad3ef65fbbe90769541346106625dde774e20ca2aec0dd58aa96d1c7359af978&scene=21#wechat_redirect)

13.[涤生大数据实战：基于Flink+ODPS历史累计计算项目分析与优化（二）](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487874&idx=1&sn=15610fda5195189c32994b77eff822e5&chksm=cf6224d0f815adc6b5ee69cf530dc12be7141a3499641826a292ca896ce173ad3ee56fabda46&scene=21#wechat_redirect)

14.[5分钟了解实时车联网，车联网（IoV)OLAP 解决方案是怎样的？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247487981&idx=1&sn=f638499eb34a3b03fbd51c95b2f4be39&chksm=cf6224bff815ada9ab9017f3425de1751e38e664090ffdb37069f19579fdf13870e6d801ba73&scene=21#wechat_redirect)

15.[企业级Apache Kafka集群策略：Kakfa最佳实践总结](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488177&idx=1&sn=5469c0af700aff357384e62b3b9cecfd&chksm=cf6227e3f815aef5c006659bc0835f42fb94ecf951c7f73a6c9dfa577e717e1d2ae0f670cde1&scene=21#wechat_redirect)

16.[玩转Spark小文件合并与文件读写提交机制](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247488932&idx=1&sn=acda9e39d8fc98b145e422179a7b5da1&chksm=cf6220f6f815a9e06b88e0e21c347371236867a248dae17cc456092652d34411b5e7fa803b02&scene=21#wechat_redirect)

17.[一文详解Spark内存模型原理，面试轻松搞定](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489016&idx=1&sn=9cce9cad9636a008429d7447c4354583&chksm=cf6220aaf815a9bce52e85681de705c8eaa48bd5668216b83fb58c465d360ffb1251f0fc146c&scene=21#wechat_redirect)

18.[大厂8年老司机漫谈数仓架构](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489193&idx=1&sn=0c3d73dcff09e74b14eb999a0006b7ca&chksm=cf6223fbf815aaed3c4ee5f541bdc94fc437d5720be66bff18c673ba7e4b4b11376a3ff8bcad&scene=21#wechat_redirect)

19.[一文带你深入吃透Spark的窗口函数](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489211&idx=1&sn=9f7731af8cd1361444cf097cfeb509fd&chksm=cf6223e9f815aaff227a3049ac5236bb59d961f604e63130b5c514eeb0bea2778fd6128f9b20&scene=21#wechat_redirect)

20.[大数据实战：基于Flink+ODPS进行最近N天实时标签构建](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489298&idx=1&sn=e14a0fdb270d0b751dc8fb4f00608142&chksm=cf622240f815ab563f0d2c9cf0ce26dd94000fd5ef749a930ec65b90fd13ae90fcb0080039d4&scene=21#wechat_redirect)

21.[数仓面试高频-如何在Hive中实现拉链表](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489455&idx=1&sn=3265f322898810f682fdfc3f513bee6b&chksm=cf6222fdf815abebb193cbd7dedce40708557aa451cbdb876b09b6791a6f188850cf45ad4be7&scene=21#wechat_redirect)

22.[数仓面试还不懂什么是基线管理？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489345&idx=1&sn=19c3954a7fbfda3377859bc5ac0b6b7d&scene=21#wechat_redirect)

23.[传说中的热点值打散之代码怎么写？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489524&idx=1&sn=6c149f0290a51f8bb44ed96d5f52a8f9&chksm=cf6222a6f815abb02e3420b51e3f554f3caf82be8ffd1695ccecb9c21aa95ce5b3667a1557c4&scene=21#wechat_redirect) 

24.[列转行经典实现，细谈hive中的爆炸函数](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489564&idx=1&sn=7c4a072289f8fcba75b485fb332132f0&chksm=cf622d4ef815a458f795709e8aa9eca068e05413c29b3a43fe6595b0cc4b51d3637d7a47d653&scene=21#wechat_redirect)

25.[玩转大厂金融风控体系建设](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489621&idx=1&sn=0533a3e320ebd8b73c730b2b9c7fa3fd&chksm=cf622d07f815a41104b46e29de1922cee318be9ee07dedb7baafe853ebd07b46caa550c8639b&scene=21#wechat_redirect)

26.[实际开发中：如何有效运用Spark Catalyst的执行流程](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489675&idx=1&sn=768616280ce2b0f3ad80e9db814ffc04&chksm=cf622dd9f815a4cf1a86cdd7c4189a4c7fcfac4fde49b106eb7c0b3c142b7f6a01d25648ad89&scene=21#wechat_redirect)

27.[Doris企业架构选型总结：存算分离与一体化的对比与应用场景分析](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489749&idx=1&sn=31bfcca689058f628a539f31d1bfe48e&chksm=cf622d87f815a49117566a5a946ffae41be2a1ea2038be731a36c82d829e765592188280da22&scene=21#wechat_redirect)

28.[详解用户画像的标签体系](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489820&idx=1&sn=b30abef6f7b21daaebb3dff8f647c9cd&chksm=cf622c4ef815a55834761d53aab0ad2fd82c54ddb9bfe4466fda563045a6effde13d15a239ee&scene=21#wechat_redirect)

29.[智能数据时代：如何优化数仓模型的复用性](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489895&idx=1&sn=396aa7260bb440ce7cb56c29bbaba5b4&chksm=cf622c35f815a5232ba89ae389d63971d6f0f92588638448897fba40bc9c938d3ecfed670ea1&scene=21#wechat_redirect)

30.[数仓疑惑：深度解析数据域和主题域的区别？](http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490004&idx=1&sn=7a4261237051c2207d8de7a3cc3b4d03&chksm=cf622c86f815a590baefa8abfea5adee8ae4d4b45c0773a138e08d86a2e1cb9618cc4665130e&scene=21#wechat_redirect)

欢迎来到涤生大数据，一个由来自阿里巴巴、京东、美团、腾讯、虾皮、蚂蚁、科大讯飞和米哈游等顶尖科技公司的专业人士精心打造的创新网络学习课程。

**为什么选择涤生大数据？**

* 跟随行业专家学习：我们的导师不是传统的讲师，而是实际的行业专家。他们将来自顶级公司的真实工作实战经验直接传授给你。
* 实用的专业知识：我们的团队包括大数据开发人员和技术架构师，确保你获得实用且适用的见解，而不仅仅是书本上的理论。
* 成长的定制设计：专为当前和未来的数据专业人士设计，我们的课程聚焦于当今行业中使用的前沿技术和工具。
* 灵活学习体验： 我们提供一个独特且灵活的学习环境，适应你的时间安排和学习节奏，并提供强力有效的督促和指导。

**想了解更多的涤生大数据可以关注我们的官网：https://www.dsbigdata.com/，或者联系我们**
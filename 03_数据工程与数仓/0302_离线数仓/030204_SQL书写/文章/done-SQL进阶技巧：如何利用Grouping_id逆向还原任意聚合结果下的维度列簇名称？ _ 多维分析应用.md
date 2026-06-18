---
title: SQL进阶技巧：如何利用Grouping_id逆向还原任意聚合结果下的维度列簇名称？ |  多维分析应用
author: 会飞的一十六
date:
url: http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484701&idx=1&sn=4de9d4831e67e9d5f45752274457a50c&chksm=e9387693abfde062decb66c17ec0e1daefc1b8350c76267c9f9ee9547590732399cc95309dd7&mpshare=1&scene=24&srcid=082497IEQoD5DWNKNqTI2sNh&sharer_shareinfo=e843fa970331c7297539207d890b3642&sharer_shareinfo_first=e843fa970331c7297539207d890b3642#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL聚合去重与膨胀治理|SQL聚合去重与膨胀治理]]


## 0 场景描述

现有用户访问日志表 visit\_log ，每一行数据表示一条用户访问日志。

需求：

（1）按照如下维度组合

(province), (province, city), (province, city, device\_type)

计算用户访问量，要求一条SQL语句统计所所有组合维度的结果。

（2）根据省、市、设备，计算所有可能组合的维度的结果

（3）基于问题2，标记所有组合的的维度来源，并输出聚合维度列的名称。作为导出表，用于下游统计报表场景。具体结果如下表所示：

**1 数据准备**

```
with visit_log as (   select stack (       6,       '2024-01-01', '101', '湖北', '武汉', 'Android',       '2024-01-01', '102', '湖南', '长沙', 'IOS',       '2024-01-01', '103', '四川', '成都', 'Windows',       '2024-01-02', '101', '湖北', '孝感', 'Mac',       '2024-01-02', '102', '湖南', '邵阳', 'Android',       '2024-01-03', '101', '湖北', '武汉', 'IOS'   )   -- 字段：日期，用户，省份，城市，设备类型   as (dt, user_id, province, city, device_type))select * from visit_log;
```

**2 问题分析**

**问题1分析**

问题1主要是应用多维分析函数，**grouping\_sets()**进行组合维度求解。

```
select        province       , city       , device_type,        count(1) as visit_count    from visit_log b    group by province, city, device_type    GROUPING SETS (    ( province),    ( province, city),    ( province, city, device_type)    )
```

**问题2分析**

计算所有组合维度可能得结果。利用**cube**函数直接生成结果

```
select        province       , city       , device_type,        count(1) as visit_count    from visit_log b    group by province, city, device_type   with cube;
```

**问题3分析**

问题3的难点主要在于如何根据实际数据反推是哪种维度的组合，属于多维分析问题的逆过程。而解决此问题的关键点，**grouping\_\_id**的认识，**grouping\_\_id**就是用来标记这种维度组合关系的标识符，因此本问题突破口在于深刻认识**grouping\_\_id**。

在多维分析函数一文中，我们也讲过该知识点，本文再对该知识点进行回顾。

**grouping sets** 中的每一种组合维度，都对应唯一的 **grouping\_\_id** 值，其算法与 group by 的后的维度字段及书写的顺序有强相关性。

具体计算方法如下：

* （1）**grouping\_\_id**是一个可以认为是标识的字段，他表示某个字段是否参与了组合，用二进制表示
* （2）对于数据表中每个纬度字段，如果该纬度字段不为NULL，则该维度字段位置赋值为0，否则为1，这样就形成了一个二进制序列。注意与一般习惯性的标记进行区别，一般我们认为存在的则置为1,此处刚好相反。
* （3） 将这个二进制数转为十进制，即为当前组合维度对应的 **grouping\_\_id** 值

根据上述算法，将本文所有组合情况生成的**grouping\_\_id**值计算如下表1所示：

如上表，当为111时候代表所有行数据的汇总，相当于sum() over()。当为000表示省、市、设备维度的组合，001代表省、市维度的组合，也就是0在哪一位代表哪个维度参与了组合，上述规律成立的前提是在SQL书写中,group by后字段书写顺序必须是 省、市、设备，二进制表示的每一位是与group  by后的字段顺序保持一致的。

       以上所有的组合情况为**2^n**，比如本文有3个维度字段，那么可能情况数就有8个，其十进制数是以0开头。为了能够反推是由哪种维度组合而成，我们先生成如表1所示的，维度组合表，然后通过grouping\_\_id进行关联即可。本问题难点也就是如何生成表1所示的数据。

**第一步：生成0~7序列值，构建完整的grouping\_\_id十进制数值**

```
select pos1 as idfrom(select split(space(8), '(?!$)')          char_list    ) t  lateral view posexplode(char_list) tmp1 as pos1, val1
```

**第二步：生成省份、市、设备类型维度值**

```
select pos2,val2from(select array('省份', '城市', '设备类型') dim_list    ) tmp    lateral view posexplode(dim_list) tmp2 as pos2, val2
```

**第三步：根据步骤1和步骤2做笛卡尔迪，生成所有可能的组合值，并将grouping\_\_id10进制数值转换为二进制序列。**

```
select tmp1.pos1                                   id            , tmp2.pos2                            pos2            , lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str            , tmp2.val2                            dimension_namefrom (select split(space(8), '(?!$)')              char_list           , array('省份', '城市', '设备类型')       dim_list           ) tmp     lateral view posexplode(char_list) tmp1 as pos1, val1     lateral view posexplode(dim_list) tmp2 as pos2, val2
```

通过两次表生成函数做笛卡尔迪，生成齐全的数据。利用conv()函数将10进制数值转换为二进制，conv()函数总共有三个参数，第一个参数表示要转化的数值，第二个参数代表要转化的进制如10表示10进制，第三个参数表示将转换成几进制。如 conv(5,10,2):

```
select conv(5,10,2);
```

利用左补齐函数lpad将位数补齐，形成bit字符串。

```
lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str
```

最终生成的结果如下：

通过上述结果我们可以看到bit\_str列，如001，我们需要合并的是省份和城市（省份，城市），010 需要合并的是省份和设备类型（省份，设备类型），也就是字符串中有0 的位置需要合并，因此我们考虑将bit\_str字符序列展开。

**第四步：将bit\_str字符序列展开。字符串展开的方法，我们在前面的文章中已经讲过多次，这里不再复述。直接利用如下式子展开**

```
lateral view posexplode(split(t1.bit_str, '(?!$)')) t2 as bit_pos, bit_flag
```

与步骤3的数据进行侧视连接，并利用pos索引值进行一一对应，具体SQL如下：

```
select t1.id             id     , t1.bit_str        bit_str     , t1.dimension_name dimension_name     , t2.bit_flag       bit_flagfrom (select tmp1.pos1                            id           , tmp2.pos2                            pos2           , lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str           , tmp2.val2                            dimension_name      from (select split(space(8), '(?!$)')          char_list                 , array('省份', '城市', '设备类型') dim_list) tmp               lateral view posexplode(char_list) tmp1 as pos1, val1               lateral view posexplode(dim_list) tmp2 as pos2, val2) t1         lateral view posexplode(split(t1.bit_str, '(?!$)')) t2 as bit_pos, bit_flagwhere t2.bit_pos = t1.pos2
```

**第五步：过滤掉bit\_flag为1的数据，按照id,bir\_str,bit\_flag进行数据合并**

```
select id     ,bit_str     ,concat_ws(',',collect_list(dimension_name)) dimension_namefrom(select t1.id             id     , t1.bit_str        bit_str     , t1.dimension_name dimension_name     , t2.bit_flag       bit_flagfrom (select tmp1.pos1                            id           , tmp2.pos2                            pos2           , lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str           , tmp2.val2                            dimension_name      from (select split(space(8), '(?!$)')          char_list                 , array('省份', '城市', '设备类型') dim_list) tmp               lateral view posexplode(char_list) tmp1 as pos1, val1               lateral view posexplode(dim_list) tmp2 as pos2, val2) t1         lateral view posexplode(split(t1.bit_str, '(?!$)')) t2 as bit_pos, bit_flagwhere t2.bit_pos = t1.pos2 ) twhere bit_flag !=1group by id,bit_str,bit_flag
```

**第六步：union all 所有行数据汇总的维度，此维度为特殊维度，也即 bit\_str 为 111时的维度，我们用空或ALL来表示。**

```
select id     ,bit_str     ,concat_ws(',',collect_list(dimension_name)) dimension_namefrom(select t1.id             id     , t1.bit_str        bit_str     , t1.dimension_name dimension_name     , t2.bit_flag       bit_flagfrom (select tmp1.pos1                            id           , tmp2.pos2                            pos2           , lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str           , tmp2.val2                            dimension_name      from (select split(space(8), '(?!$)')          char_list                 , array('省份', '城市', '设备类型') dim_list) tmp               lateral view posexplode(char_list) tmp1 as pos1, val1               lateral view posexplode(dim_list) tmp2 as pos2, val2) t1         lateral view posexplode(split(t1.bit_str, '(?!$)')) t2 as bit_pos, bit_flagwhere t2.bit_pos = t1.pos2 ) twhere bit_flag !=1group by id,bit_str,bit_flagunion allselect 7 id,'111' bit_str,'ALL' dimension_name
```

至此表1的数据，我们已经生成完毕

**第七步：将生成的维度表，dimension\_name与数据表关联得到最终的结果**。

```
with visit_log as (    select stack (        6,        '2024-01-01', '101', '湖北', '武汉', 'Android',        '2024-01-01', '102', '湖南', '长沙', 'IOS',        '2024-01-01', '103', '四川', '成都', 'Windows',        '2024-01-02', '101', '湖北', '孝感', 'Mac',        '2024-01-02', '102', '湖南', '邵阳', 'Android',        '2024-01-03', '101', '湖北', '武汉', 'IOS'    )    -- 字段：日期，用户，省份，城市，设备类型    as (dt, user_id, province, city, device_type)) select dimension_name     , province     , city     , device_type     , visit_countfrom(select id      , bit_str      , concat_ws(',', collect_list(dimension_name)) dimension_name from (select t1.id             id            , t1.bit_str        bit_str            , t1.dimension_name dimension_name            , t2.bit_flag       bit_flag       from (select tmp1.pos1                            id                  , tmp2.pos2                            pos2                  , lpad(conv(tmp1.pos1, 10, 2), 3, '0') bit_str                  , tmp2.val2                            dimension_name             from (select split(space(8), '(?!$)')          char_list                        , array('省份', '城市', '设备类型') dim_list) tmp                      lateral view posexplode(char_list) tmp1 as pos1, val1                      lateral view posexplode(dim_list) tmp2 as pos2, val2) t1                lateral view posexplode(split(t1.bit_str, '(?!$)')) t2 as bit_pos, bit_flag       where t2.bit_pos = t1.pos2) t where bit_flag != 1 group by id, bit_str, bit_flag union all select 7 id, '111' bit_str, 'ALL' dimension_name) dimleft join( select  grouping__id id       , province       , city       , device_type,        count(1) as visit_count    from visit_log b    group by province, city, device_type   with cube ) dataon dim.id =  data.id
```

**3 小结**

多维分析往往是数据分析领域备受关注的点，也是比较难理解的地方，特别是对grouping\_\_id含义的理解，本文所分析的场景往往用于还原一条统计结果是根据哪个维度名称聚合而来，其核心思想是利用grouping\_id的原理进行分析，grouping\_\_id则反应了维度组合的来源，只是将数值型数据还原成维度标签，本文对于理解grouping\_\_id含义具有较好的启示作用。

**往期回顾**

[Sql进阶技巧：如何计算截断平均值?【场景:去掉最大最小值的平均值】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484669&idx=1&sn=242d9066a6c0b93b444d861b751d5303&chksm=e8e210dddf9599cbd4e38b3c600f93e8074a7c66326780843d38ae7170a27e52a0a24961b9ce&scene=21#wechat_redirect)

[SQL进阶技巧：断点缝合问题【如何按照业务规则对相邻行数据进行合并】](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484632&idx=1&sn=04456cd9ead95864cadd0d98b049900c&chksm=e8e210f8df9599ee4471bffd30fe4fce73d063e09ced8f342cb463bfb183cb521a31bd7ddeae&scene=21#wechat_redirect)

[SQL进阶技巧：如何利用Bitmap思想优化Array\_contains()函数？](http://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247484567&idx=1&sn=4d10c88acc86abd5a13d2a9b42284bb4&chksm=e8e210b7df9599a15423d1373ba3e8d2a9b66e4ed7e39b6e6269ab76810b9a0bea6d61863733&scene=21#wechat_redirect)
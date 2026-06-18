---
title: （二）Hive SQL中的lateral view与explode（列转行）以及行转列
author: 阿龙大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485277&idx=1&sn=77d4d6d2e05845de3d8e2c384426829a&chksm=9e9a67ac18ea4f0e453adbdd86208aabedbd2a9dc5eb1ff59178c90ca2887ab178f4f3c50f8c&mpshare=1&scene=24&srcid=0127XV0UEZxo1c2gq9xHz1Hz&sharer_shareinfo=dd5c86dff826e632f7a1e2afed78e12c&sharer_shareinfo_first=dd5c86dff826e632f7a1e2afed78e12c#rd
---

一、行转列

行转列：将多个列中的数据在一列中输出。

列转行：将某列一行中的数据拆分成多行。

**Concat**

```
concat(string1/col, string2/col, …)
```

**输入任意个字符串(或字段,可以为int类型等)，返回拼接后的结果**

```
select concat(id,'-',name,'-',age) from student;
```

****Concat\_ws：****

```
concat_ws(separator, str1, str2, …)
```

    特殊形式的 concat()，参数只能为字符串，第一个参数为后面参数的分隔符

    分隔符可以是与后面参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;

```
select concat_ws('-', name, gender) from student;
```

**Collect\_set（聚合，返回数组类型）**

```
collect_set(col)
```

****将某字段进行去重处理，返回array类型；该函数只接受基本数据类型。****

```
select collect_set(age) from student;
```

****注意：****

****collect\_set 与 collect\_list 的区别就是set去重，list不去重。****

****案例一：****

    把星座和血型一样的人归类到一起。

****结果：****

```
射手座,A            大海白羊座,A            孙悟空|猪八戒白羊座,B            宋宋
```

核心代码：

```
select    t1.base,    concat_ws('|', collect_set(t1.name)) as namefrom    (select name,concat(constellation, ",", blood_type) as base    from person_info) as t1group by    t1.base;
```

录每一份热爱,让美好永远陪伴。
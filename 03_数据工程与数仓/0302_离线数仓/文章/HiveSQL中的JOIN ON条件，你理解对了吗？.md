---
title: HiveSQL中的JOIN ON条件，你理解对了吗？
author: Flink
date: 
url: http://mp.weixin.qq.com/s?__biz=MzI0NTIxNzE1Ng==&mid=2651226251&idx=1&sn=0f8f1cd7887dd343c54ff2c9443eec21&chksm=f2a3c260c5d44b76dbe634c8c173d3ddc87c118afc048c0436e97020f900ee4283381c366a48&mpshare=1&scene=24&srcid=1004O81Ui5Ret3T8lRKUYHQf&sharer_sharetime=1664837650027&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

HiveSQL很常用的一个操作就是关联(Join)。Hive为用户提供了多种JOIN类型，可以满足不同的使用场景。但是，对于不同JOIN类型的语义，或许有些人对此不太清晰。简单的问题，往往是细节问题，而这些问题恰恰也是重要的问题。本文将围绕不同的JOIN类型，介绍JOIN的语义，并对每种JOIN类型需要注意的问题进行剖析，希望本文对你有所帮助。

## JOIN类型

| 类型 | 含义 |
| --- | --- |
| Inner Join | 输出符合关联条件的数据 |
| Left Join | 输出左表的所有记录，对于右表符合关联的数据，输出右表，没有符合的，右表补null |
| Right Join | 输出右表的所有记录，对于左表符合关联的数据，输出左表，没有符合的，左表补null |
| Full Join | 输出左表和右表的所有记录，对于没有关联上的数据，未关联的另一侧补null |
| Left Semi Join | 对于左表中的一条数据，如果右表存在符合关联条件的行，则输出左表 |
| Left Anti Join | 对于左表中的一条数据，如果对于右表所有的行，不存在符合关联条件的数据，则输出左表 |

## JOIN的通用格式

```
SELECT   
        a.*   
        ,b.*   
FROM   
        (  
                SELECT *   
                FROM a   
                WHERE {subquery_where_condition}   
        ) a   
  
{LEFT/RIGHT/FULL/LEFT SEMI/LEFT ANTI} JOIN   
  
        (  
                SELECT *   
                FROM b  
                WHERE {subquery_where_condition}   
        ) b   
ON {on_condition}   
WHERE {where_condition}  
;
```

1. **子查询中的`{subquery_where_condition}`**
2. **JOIN的`{on_condition}`的条件**
3. **JOIN结果集合`{where_condition}`的计算**

> 尖叫提示：
>
> 对于不同的JOIN类型，过滤语句放在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中，有时结果是一致的，有时候结果又是不一致的。下面分情况进行讨论：

## 数据准备

* 建表并导入数据

```
create table a(id int,ds string);  
insert into table a VALUES (1, 20220101),(2, 20220101),(2, 20220102);  
  
  
create table b(id int,ds string);  
insert into table b VALUES (1, 20220101),(3, 20220101),(2, 20220102) ;
```

* 笛卡尔积

```
set hive.mapred.mode = 'nonstrict';  
set hive.strict.checks.cartesian.product = 'false';  
  
select * from a join b;  
  
1 20220101 1 20220101  
2 20220101 1 20220101  
2 20220102 1 20220101  
1 20220101 3 20220101  
2 20220101 3 20220101  
2 20220102 3 20220101  
1 20220101 2 20220102  
2 20220101 2 20220102  
2 20220102 2 20220102
```

## 场景说明

### INNER JOIN

#### 示例说明

**INNER JOIN对左右表执行笛卡尔乘积，然后输出满足ON表达式的行。**

* 情况1：过滤条件在子查询，即分别提前过滤要关联的两个表格数据，然后在根据ON条件进行关联。

  > 注意：这种方式是规范的方式，在实际的开发过程中，我们尽量都按这种方式开发，避免多表多条件关联时导致代码比较混乱。

  ```
  SELECT  a.*  
          ,b.*  
  FROM    (  
              SELECT  *  
              FROM    a  
              WHERE   ds = '20220101'  
          ) a  
  JOIN    (  
              SELECT  *  
              FROM    b  
              WHERE   ds = '20220101'  
          ) b  
  ON      a.id = b.id  
  ;
  ```

  结果如下:

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
* 情况2：过滤条件在JOIN的关联条件中

  ```
  SELECT  a.*  
          ,b.*  
  FROM    a  
  JOIN    b  
  ON      a.id = b.id  
  AND     a.ds = '20220101'  
  AND     b.ds = '20220101'
  ```

  笛卡尔积结果为9条，满足关联条件的结果只有1条，如下。

  结果如下:

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
* 情况3：过滤条件在JOIN结果集的WHERE子句中。

  ```
  SELECT  a.*  
          ,b.*  
  FROM    a  
  JOIN    b  
  ON      a.id = b.id  
  WHERE   a.ds = '20220101'  
  AND     b.ds = '20220101'  
  ;
  ```

  先进行笛卡尔积，结果为9条，满足关联条件的结果有3条，如下。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220102 | 2 | 20220102 |
  | 2 | 20220101 | 2 | 2020102 |

  对上述结果执行JOIN结果集中的过滤条件`WHERE a.ds = '20220101'AND b.ds = '20220101'`，结果只有1条，如下。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |

#### 结论

过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，查询结果是一致的。INNER JOIN比较特殊，由于只匹配能关联上的数据，所以无论过滤条件怎么写，最终的结果都是一致的。即便是这样，在实际的开发过程中建议使用**情况1**的方式进行书写，避免不必要的问题出现。

### LEFT JOIN

**LEFT JOIN对左右表执行笛卡尔乘积，输出满足ON表达式的行。对于左表中不满足ON表达式的行，输出左表，右表输出NULL**。

> 注意：输出满足ON表达式的行，输出满足ON表达式的行，输出满足ON表达式的行，只是ON条件，不是WHERE条件，此处最容易出问题

#### 示例说明

* 情况1：过滤条件在子查询

  > 此方式是规范的写法，建议使用此种方式

  ```
  SELECT  a.*  
          ,b.*  
  FROM    (  
              SELECT  *  
              FROM    a  
              WHERE   ds = '20220101'  
          ) a  
  LEFT JOIN    (  
              SELECT  *  
              FROM    b  
              WHERE   ds = '20220101'  
          ) b  
  ON      a.id = b.id  
  ;
  ```

  结果如下。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | NULL | NULL |
* **情况2：过滤条件在JOIN的关联条件**

  > 注意，注意，注意，注意，注意，注意，注意，注意，注意，注意，此处容易出问题

  ```
  SELECT  a.*  
          ,b.*  
  FROM    a  
  LEFT JOIN    b  
  ON      a.id = b.id  
  AND     a.ds = '20220101'  
  AND     b.ds = '20220101'  
  ;
  ```

  笛卡尔积的结果有9条，满足关联条件的结果只有1条。左表输出剩余不满足关联条件的两条记录，右表输出NULL。

  > 由于是LEFT JOIN 对于左表需要全表输出，最终的结果可能跟我们预期的不一致，这个就是LEFT JOIN的语义，在写SQL的时候一定要注意。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | NULL | NULL |
  | 2 | 20220102 | NULL | NULL |
* 情况3：过滤条件在JOIN结果集的WHERE子句中。

  ```
  SELECT A.*, B.*  
  FROM A LEFT JOIN B  
  ON a.key = b.key  
  WHERE A.ds='20180101' and B.ds='20180101';
  ```

  笛卡尔积的结果为9条，满足ON条件的结果有3条。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | 2 | 20220102 |
  | 2 | 20220102 | 2 | 20220102 |

  对上述结果执行JOIN结果集中的过滤条件`a.ds='20220101' and b.ds='20220101'`，结果只有1条。

  | a.key | a.ds | b.key | b.ds |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |

#### 结论

过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`中时，**查询结果不一致**。**牢记LEFT JOIN的语义，对于左表中不满足ON表达式的行，输出左表，右表输出NULL**

### RIGHT JOIN

参考LEFT JOIN

### FULL JOIN

#### 示例说明

FULL JOIN对左右表执行笛卡尔乘积，然后输出满足关联条件的行。对于左右表中不满足关联条件的行，输出有数据表的行，无数据的表输出NULL。

* 情况1：过滤条件在子查询，规范写法

  ```
  SELECT  a.*  
          ,b.*  
  FROM    (  
              SELECT  *  
              FROM    a  
              WHERE   ds = '20220101'  
          ) a  
  FULL JOIN    (  
              SELECT  *  
              FROM    b  
              WHERE   ds = '20220101'  
          ) b  
  ON      a.id = b.id  
  ;
  ```

  结果如下。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | NULL | NULL |
  | NULL | NULL | 3 | 20220101 |
* 情况2：过滤条件在JOIN的关联条件

  ```
  SELECT  a.*  
          ,b.*  
  FROM    a  
  FULL JOIN    b  
  ON      a.id = b.id  
  AND     a.ds = '20220101'  
  AND     b.ds = '20220101'
  ```

  **笛卡尔积的结果有9条，满足关联条件的结果只有1条。对于左表不满足关联条件的两条记录输出左表数据，右表输出NULL。对于右表不满足关联条件的两条记录输出右表数据，左表输出NULL。**

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | NULL | NULL |
  | 2 | 20220102 | NULL | NULL |
  | NULL | NULL | 3 | 20220101 |
  | NULL | NULL | 2 | 20220102 |
* 情况3：过滤条件在JOIN结果集的WHERE子句中。

  ```
  SELECT  a.*  
          ,b.*  
  FROM    a  
  FULL JOIN    b  
  ON      a.id = b.id  
  WHERE   a.ds = '20220101'  
  AND     b.ds = '20220101'
  ```

  对于不满足关联条件的表输出数据，另一表输出NULL。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |
  | 2 | 20220101 | 2 | 20220102 |
  | 2 | 20220102 | 2 | 20220102 |
  | NULL | NULL | 3 | 20220101 |

  对上述结果执行JOIN结果集中的过滤条件`a.ds='20220101' and b.ds='20220101'`，结果只有1条。

  | id | ds | id\_1 | ds\_1 |
  | --- | --- | --- | --- |
  | 1 | 20220101 | 1 | 20220101 |

#### 结论

过滤条件在`{subquery_where_condition}`、`{on_condition}`和`{where_condition}`时，查询结果不一致。

## 推荐写法

## 总结

本文主要结合具体的使用示例，对HiveSQL的LEFT JOIN操作进行了详细解释。主要包括两种比较常见的LEFT JOIN方式，一种是正常的LEFT JOIN，也就是只包含ON条件，这种情况没有过滤操作，即左表的数据会全部返回。另一种方式是有谓词下推，即关联的时候使用了WHERE条件，这个时候会会对数据进行过滤。所以在写SQL的时候，尤其需要注意这些细节问题，以免出现意想不到的错误结果。
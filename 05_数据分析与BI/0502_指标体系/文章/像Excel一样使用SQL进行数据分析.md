---
title: 像Excel一样使用SQL进行数据分析
author: 数据前线
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU1NTU3OTk1Ng==&mid=2247490796&idx=1&sn=bd647f962b965746ed8da86dff4ca8b2&chksm=fbd373b7cca4faa10f311791e554f59b6cf8a7bd01148e02107636259d8ea00435095b915e98&mpshare=1&scene=24&srcid=1122IAftV8a3BOJluvA33Q4S&sharer_shareinfo=1ce6a37d0dd4396c9c1d443f346ad8e8&sharer_shareinfo_first=1ce6a37d0dd4396c9c1d443f346ad8e8#rd
---

**点击关注上方“数据前线”，**

**设为“置顶或星标****”，第一时间送达干货**

Excel是数据分析中最常用的工具 ，利用Excel可以完成数据清洗，预处理，以及最常见的数据分类，数据筛选，分类汇总，以及数据透视等操作，而这些操作用SQL一样可以实现。

SQL不仅可以从数据库中读取数据，还能通过不同的SQL函数语句直接返回所需要的结果，从而大大提高了自己在客户端应用程序中计算的效率。

**1  重复数据处理**

##### **查找重复记录**

```
SELECT * FROM user   
Where (nick_name,password) in  
(  
SELECT nick_name,password   
FROM user   
group by nick_name,password   
having count(nick_name)>1  
);
```

```
查找去重记录
```

查找id最大的记录

```
SELECT * FROM user   
WHERE id in  
(SELECT max(id) FROM user  
group by nick_name,password   
having count(nick_name)>1  
);
```

```
删除重复记录
```

只保留id值最小的记录

```
DELETE  c1  
FROM  customer c1,customer c2  
WHERE c1.cust_email=c2.cust_email  
AND c1.id>c2.id;
```

```
DELETE FROM user Where (nick_name,password) in  
(SELECT nick_name,password FROM  
    (SELECT nick_name,password FROM user   
    group by nick_name,password   
    having count(nick_name)>1) as tmp1  
)  
and id not in  
(SELECT id FROM  
    (SELECT min(id) id FROM user   
     group by nick_name,password   
     having count(nick_name)>1) as tmp2  
);
```

```
2  缺失值处理
```

##### **查找缺失值记录**

```
SELECT * FROM customer  
WHERE cust_email IS NULL;
```

**更新列填充空值**

```
UPDATE sale set city = "未知"   
WHERE city IS NULL;  
  
UPDATE orderitems set   
price_new=IFNULL(price_new,5.74);
```

**查询并填充空值列**

```
SELECT AVG(price_new) FROM orderitems;  
  
SELECT IFNULL(price_new,5.74) AS bus_ifnull  
FROM orderitems;
```

```
3  计算列
```

##### **更新表添加计算列**

```
ALTER TABLE orderitems ADD price_new DECIMAL(8,2) NOT NULL;  
  
UPDATE orderitems set price_new= item_price*count;
```

```
查询计算列
```

```
SELECT item_price*count as sales FROM orderitems;
```

**4  排序**

##### **多列排序**

```
SELECT * FROM orderitems  
ORDER BY price_new DESC,quantity;
```

**查询排名前几的记录**

```
SELECT  * FROM orderitems  
ORDER BY price_new DESC Limit 5;
```

**查询第10大的值**

```
SELECT DISTINCT price_new  
FROM orderitems  
ORDER BY price_new DESC LIMIT 9,1;
```

**排名**

数值相同的排名相同且排名连续

```
SELECT prod_price,  
(SELECT COUNT(DISTINCT prod_price)  
FROM products  
WHERE prod_price>=a.prod_price  
) AS rank  
FROM products AS a  
ORDER BY rank ;
```

```
5 字符串处理
```

##### **字符串替换**

```
UPDATE data1 SET city=REPLACE(city,'SH','shanghai');  
  
SELECT city FROM data1;
```

**按位置字符串截取**

字符串截取可用于数据分列  
MySQL 字符串截取函数：left(), right(), substring(), substring\_index()

```
SELECT left('example.com', 3);
```

从字符串的第 4 个字符位置开始取，直到结束

```
SELECT substring('example.com', 4);
```

从字符串的第 4 个字符位置开始取，只取 2 个字符

```
SELECT substring('example.com', 4, 2);
```

**按关键字截取字符串**

取第一个分隔符之前的所有字符，结果是www

```
SELECT substring_index('www.google.com','.',1);
```

取倒数第二个分隔符之后的所有字符，结果是google.com;

```
SELECT substring_index('www.google.com','.',-2);
```

**6 筛选**

##### **通过操作符实现高级筛选**

使用 AND OR IN NOT 等操作符实现高级筛选过滤

```
SELECT prod_name，prod_price FROM Products  
WHERE vend_id IN('DLL01','BRS01');  
SELECT prod_name FROM Products WHERE NOT vend_id='DLL01';
```

**通配符筛选**

常用通配符有% \_ [] ^

```
SELECT * from customers WHERE country LIKE "CH%";
```

**7 表联结**

SQL表连接可以实现类似于Excel中的Vlookup函数的功能

```
SELECT vend_id,prod_name,prod_price  
FROM Vendors INNER JOIN Products  
ON Vendors.vend_id=Products.vend_id;  
  
SELECT prod_name,vend_name,prod_price,quantity  
FROM OderItems,Products,Vendors  
WHERE Products.vend_id=Vendors.vend_id  
AND OrderItems.prod_id=Products.prod_id  
AND order_num=20007;
```

```
自联结 在一条SELECT语句中多次使用相同的表
```

```
SELECT c1.cust_od,c1.cust_name,c1.cust_contact  
FROM Customers as c1,Customers as c2  
WHERE c1.cust_name=c2.cust_name  
AND c2.cust_contact='Jim Jones';
```

```
8 数据透视
```

数据分组可以实现Excel中数据透视表的功能

##### **数据分组**

group by 用于数据分组 having 用于分组后数据的过滤

```
SELECT order_num,COUNT(*) as items  
FROM OrderItems  
GROUP BY order_num HAVING COUNT(*)>=3;
```

```
交叉表
```

通过CASE WHEN函数实现

```
SELECT data1.city,  
CASE WHEN colour = "A" THEN price END AS A,  
CASE WHEN colour = "B" THEN price END AS B,  
CASE WHEN colour = "C" THEN price END AS C,  
CASE WHEN colour = "F" THEN price END AS F  
FROM data1
```

*注：以上代码在MySQL数据库中执行*

```
####
```
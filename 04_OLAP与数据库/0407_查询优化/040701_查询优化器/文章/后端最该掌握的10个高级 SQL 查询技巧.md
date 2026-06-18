---
title: 后端最该掌握的10个高级 SQL 查询技巧
author: Java面试那些事儿
date: 
url: http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247611658&idx=2&sn=4b5b298c9f51e77b6c8f833626684045&chksm=e96de07f6b7a00424699d377511468cfb13ff1ff5af32f18078cfe0fe1887dbb2d0cb1cf2f1c&mpshare=1&scene=24&srcid=0425EEQzNBXK4LIYGKhBJI0n&sharer_shareinfo=d840b455d69e9fcc841d5faf17cda410&sharer_shareinfo_first=d840b455d69e9fcc841d5faf17cda410#rd
---

作为一个 Java 开发，平时和 SQL 打交道的次数多得不行，尤其是做业务系统的，经常要写各种复杂的查询语句，稍不注意就写成了“意大利炮”，别人一看代码直接问候你三代。

今天就跟大家聊聊一些我平时工作中经常用到的中高级 SQL 技巧，都是干货，带着例子讲，争取让你看完之后不再怕写 SQL。

先说清楚，这不是那种“SELECT \* FROM table”级别的入门内容，而是适合有一定基础，已经能写一些简单 SQL，希望再往深水区进阶一把的小伙伴。

## 用 CTE 写复杂查询

工作中你肯定遇到过那种嵌套得像俄罗斯套娃一样的 SQL，什么 SELECT 里套个 SELECT，WHERE 里再来个 SELECT，看得头皮发麻，写的过程简直像在打迷宫。

这时候就该请出 CTE（Common Table Expressions）了，中文叫“公用表表达式”，但说实话这个翻译不重要，记住它就是用来“分块写 SQL”的。

我们来个例子：

> 比如我现在有个需求，要查出多伦多的人名，然后从这些人里挑出薪资高于女性平均工资的。

你要是不用 CTE，写出来就是这样子：

```
SELECT name, salary  
FROM People  
WHEREnameIN (  
    SELECTDISTINCTname  
    FROM population  
    WHERE country = 'Canada'AND city = 'Toronto'  
)  
AND salary >= (  
    SELECTAVG(salary)  
    FROM salaries  
    WHERE gender = 'Female'  
)
```

是不是感觉这还能接受？那你试试加上 JOIN、窗口函数、分组、排序，再来个分页，整个 SQL 就成了一坨粘在一起的“程序员心碎代码”。

这时候我们用 CTE 来拯救：

```
WITH toronto_people AS (  
    SELECTDISTINCTname  
    FROM population  
    WHERE country = 'Canada'AND city = 'Toronto'  
),  
avg_female_salary AS (  
    SELECTAVG(salary) AS avg_salary  
    FROM salaries  
    WHERE gender = 'Female'  
)  
SELECTname, salary  
FROM People  
WHEREnameIN (SELECTnameFROM toronto_people)  
AND salary >= (SELECT avg_salary FROM avg_female_salary)
```

整齐！舒服！逻辑一目了然，老板再也不会骂你 SQL 写得像屎了。

## 递归 CTE

如果你写过组织架构这种层级关系的数据，比如“张三是李四的上司，李四是王五的上司”，你肯定知道这是递归的天下。你不能一条一条地去查，那样就是手工地狱。

SQL 中的递归 CTE 就是用来干这事的，SQL 里的“自己调自己”。

我们来个公司组织结构的经典例子：

```
WITH RECURSIVE org_chart AS (  
    SELECT id, manager_id  
    FROM employees  
    WHERE manager_id IS NULL  
    UNION ALL  
    SELECT e.id, e.manager_id  
    FROM employees e  
    JOIN org_chart o ON e.manager_id = o.id  
)  
SELECT * FROM org_chart;
```

这一招真的是在处理层级数据的时候神技，谁用谁知道。特别是在 Java 后台跟数据库交互的时候，别再想着用 Java 自己递归查询，那效率感人到爆炸，还是让数据库自己递吧。

## 临时函数

你是不是也觉得 `CASE WHEN` 写起来很爽，但一多起来就像在法院现场审判员工，“tenure 小于 1 年，打入 analyst 等级；3 到 5 年，升为 senior”。写起来像演剧情。

比如这种：

```
SELECT name,  
       CASE  
           WHEN tenure < 1THEN'analyst'  
           WHEN tenure BETWEEN1AND3THEN'associate'  
           WHEN tenure BETWEEN3AND5THEN'senior'  
           WHEN tenure > 5THEN'vp'  
           ELSE'n/a'  
       ENDASlevel  
FROM employees;
```

这时候你可以定义一个临时函数，就像你在 Java 里抽方法一样，干净又复用。

```
CREATE TEMP FUNCTION get_level(tenure INT64) AS (  
    CASE  
        WHEN tenure < 1THEN'analyst'  
        WHEN tenure BETWEEN1AND3THEN'associate'  
        WHEN tenure BETWEEN3AND5THEN'senior'  
        WHEN tenure > 5THEN'vp'  
        ELSE'n/a'  
    END  
);
```

然后你查询的时候用得就很优雅了：

```
SELECT name, get_level(tenure) AS level FROM employees;
```

写 SQL 的时候少一点 CASE，心态会稳定很多。

## 用 CASE WHEN 横向展开数据

你有没有遇到这种需求：

> 老板说：“我不要每行显示一个月的收入，我要一行显示一个人的全年收入，按月份展开成 12 列。”

这时候你得用 `CASE WHEN` + 聚合函数来手动“旋转表格”。

```
SELECT id,  
       SUM(CASEWHENmonth = 'Jan'THEN revenue ELSE0END) AS Jan_Revenue,  
       SUM(CASEWHENmonth = 'Feb'THEN revenue ELSE0END) AS Feb_Revenue,  
       SUM(CASEWHENmonth = 'Mar'THEN revenue ELSE0END) AS Mar_Revenue  
       -- 后面一直到 Dec 自己补全  
FROM income_table  
GROUPBYid;
```

SQL 没有你想的那么智能，“把列变成行”或者“把行变成列”这种事，都得你自己来写逻辑。

但也别怕，看多了就习惯了。

## 自连接

又是一个经典面试题：

> “找出所有比自己老板工资还高的员工。”

大多数 Java 后台的表设计是一个员工表里面有 manager\_id 字段，指向自己的上司。

这时候就用到自连接了：

```
SELECT e1.name AS employee_name  
FROM employees e1  
JOIN employees e2 ON e1.manager_id = e2.id  
WHERE e1.salary > e2.salary;
```

写起来也简单，但第一次接触自连接的同学可能会懵圈，其实就是自己跟自己 JOIN，一点也不复杂。

## Rank、Dense Rank、Row Number

这些窗口函数你得记住三个特点：

* `ROW_NUMBER()`：不管有没有分数相同，照顺序给排名，跳过重复值。
* `RANK()`：有重复值就同一个名次，但会跳名次。
* `DENSE_RANK()`：有重复值也同一个名次，但不跳名次。

打个比方，如果有两个第一名，那：

* ROW\_NUMBER 会是 1、2、3；
* RANK 会是 1、1、3；
* DENSE\_RANK 会是 1、1、2。

选哪个看你业务需求。

## LAG / LEAD

在数据报表里，经常要对比前一天或者前一月的数据，比如今天销售额比昨天涨了多少，去年这个月比今年这个月多多少。

用 `LAG()` 和 `LEAD()`，就跟你用 Java 里的 `prevValue` 一样：

```
SELECT month,  
       sales,  
       sales - LAG(sales) OVER (ORDER BY month) AS sales_delta  
FROM monthly_sales;
```

就是这么一行 SQL，就能搞定横向对比，非常适合做增长分析。

## 累加和

你要想画个“累计趋势图”，就得先算出累计值：

```
SELECT month,  
       revenue,  
       SUM(revenue) OVER (ORDER BY month) AS cumulative_revenue  
FROM monthly_revenue;
```

你可以想象成这样一行一行加起来，从头加到当前行。这个东西做财务分析特别有用，比如累计用户数、累计收入。

## 时间操作

SQL 操作时间这种事，真的是日常，什么取年、取月、加天、比较前一天的数据，这些都得掌握。

常用的几个函数：

* `DATE_DIFF()`：两个日期相差几天
* `DATE_ADD()`：加几天
* `DATE_SUB()`：减几天
* `DATE_TRUNC()`：把日期截成月/年

比如查出气温比前一天高的所有日期：

```
SELECT w1.id  
FROM weather w1  
JOIN weather w2  
  ON DATEDIFF(w1.recordDate, w2.recordDate) = 1  
WHERE w1.temperature > w2.temperature;
```

你可能觉得麻烦，但只要你熟了，写起来就跟写 Java 中 LocalDateTime 的各种 API 一样。

## 写在最后

我一直觉得，SQL 能力就像打怪游戏里最容易忽略的那条“魔法抗性”一样，平时不注意，一到打 boss 的时候就露馅。

你可以写出天花乱坠的 Java 代码，但如果数据库性能不好、SQL 写得又慢又乱，最终拖垮的还是整个项目的交付体验。

所以建议大家，不要轻视 SQL，特别是像 CTE、窗口函数、自连接这些高频操作，真的是“写一百次你都不腻”的存在。

今天这些技巧，不敢说全是高阶秘籍，但绝对是我们日常工作中最常用、最实用的那批。如果你能搞明白每一个，不管是写查询、做报表、写接口优化查询性能，都会事半功倍。

**最后，我为大家打造了一份deepseek的入门到精通教程，完全免费：****https://www.songshuhezi.com/deepseek**

**[也可以看我写的这篇文章](https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247609202&idx=1&sn=6055732408aee84a0926778b6ba67860&scene=21#wechat_redirect)****《****[DeepSeek满血复活，直接起飞](https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247609202&idx=1&sn=6055732408aee84a0926778b6ba67860&scene=21#wechat_redirect)****！****》****[来进行本地搭建。](https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247609202&idx=1&sn=6055732408aee84a0926778b6ba67860&scene=21#wechat_redirect)**

> **对编程、职场感兴趣的同学，可以链接我，微信：coder301 拉你进入“程序员交流群”。**

**[🔥东哥私藏精品 热门推荐🔥](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)**

[东哥作为一名超级老码农，整理了全网](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)最全**[《Java高级架构师资料合集》](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)**[。](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)

资料包含了**[《IDEA视频教程》](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)****[、](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)****[《最全Java面试题库》、](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)****[《](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)****[最全项目实战源码及视频》](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)****[及《毕业设计系统源码》](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)**，总量高达**[650GB](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)**。全部**[免费领取！](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)**[全面满足各个阶段程序员的学习需求。](http://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247602692&idx=1&sn=8c6c312f6bb3931078f3ee05efebc1b3&chksm=e8fcce0ddf8b471be7e29ff3b3c2b8811d2cc9447215c404ab91821a2ce3492780da2a347b81&scene=21#wechat_redirect)
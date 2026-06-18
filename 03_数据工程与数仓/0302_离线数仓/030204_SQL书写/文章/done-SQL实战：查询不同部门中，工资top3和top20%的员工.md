---
title: SQL实战：查询不同部门中，工资top3和top20%的员工
author: 数据分析师的日常
date:
url: http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247485654&idx=1&sn=b87e8bd9794931586d0e5202cb62c3b4&chksm=f863836d49759cad60910580a8757d7957e06a2bca855fbd420487eee8ae2d5c7966b53c8721&mpshare=1&scene=24&srcid=0920GjSrMlqdAdDloeC24z87&sharer_shareinfo=a4dbaa1710859af0b23e4d3688dd6dcd&sharer_shareinfo_first=a4dbaa1710859af0b23e4d3688dd6dcd#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL聚合去重与膨胀治理|SQL聚合去重与膨胀治理]]


点击上方的蓝字和下方的卡片关注我，一起踏上数据分析进阶之路！

你好，我是林赛~

今天分享如何用SQL查询不同部门中，工资top3和top20%的员工。

其他类似的问题都可以参考，比如：

* SQL查询不同大区中，销量top3和top20%的店铺。
* SQL查询不同分类中，销售额top3和top20%的商品。

**01**

**数据准备**

我们用前面文章：SQL常见笔试面试题：[计算除去部门最高工资和最低工资的平均工资（4种解题思路）](http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247484684&idx=1&sn=bdd3d2e34b1498014d0d452c4e840839&chksm=f9db56e2ceacdff4d201567fb5df750b8c9740de523158c398b07221e37efe7cfd91f5e67152&scene=21#wechat_redirect)中的数据表data\_learning.employee\_salaries来操作，具体建表语句见文末。

表内容如下：    

**02**

**SQL求不同部门中，工资top3的员工**

**方法一：使用窗口函数**

```
SELECT department, id, salary            FROM (              SELECT              department,              id,              salary,              ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn              FROM data_learning.employee_salaries            ) AS ranked_employees            WHERE rn <= 3;            
```

**解释：**

+ ROW\_NUMBER() OVER (PARTITION BY department\_id ORDER BY salary DESC) 为每个部门的员工按照工资降序排序，并为每个员工分配一个排名。
+ 外层查询从排名为1到3的员工中筛选结果。

**查询结果如下：**

```
SELECT department, id, salary            FROM (              SELECT              department,              id,              salary,              RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rk              FROM data_learning.employee_salaries            ) AS ranked_employees            WHERE rk <= 3;          
```

**解释：**

+ RANK() 函数与 ROW\_NUMBER() 类似，但在处理工资相同的员工时，会为他们分配相同的排名。这样，可能会有超过3名员工在结果中（例如，当工资相同的员工有并列排名时）。

**方法二：使用子查询和连接**

```
SELECT              e1.department, e1.id, e1.salary            FROM              data_learning.employee_salaries e1            WHERE 3 > (              SELECT                COUNT(*)              FROM                data_learning.employee_salaries e2              WHERE e2.department = e1.department                AND e2.salary > e1.salary            )            order by 1,3 desc;           
```

**解释：**

+ 子查询计算了在同一部门（e2.department = e1.department）中，薪资高于当前记录（e2.salary > e1.salary）的员工数量。然后，外层查询的 WHERE 子句只保留那些在其部门中薪资排名前 3 的员工。也就是说，如果部门中薪资高于当前薪资的员工数量少于 3，那么当前员工的薪资就会被选中。

**查询结果如下：**    

**03**

**S****QL求不同部门中，工资top20%的员工**

**方法一：使用窗口函数PERCENT\_RANK()**

如果你的数据库支持 PERCENT\_RANK() 窗口函数，你可以用它来计算每个员工在其部门中的工资排名百分比。然后筛选出排名在前20%以内的员工。

```
SELECT              department, id, salary            FROM (              SELECT                department,                id,                salary,                PERCENT_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS pct_rank              FROM                data_learning.employee_salaries            ) AS ranked_employees            WHERE pct_rank <= 0.20;       
```

**解释：**

+ PERCENT\_RANK() OVER (PARTITION BY department\_id ORDER BY salary DESC) 计算每个员工在其部门中的工资百分比排名。
+ 外层查询筛选出百分比排名在20%以内的员工。

**查询结果如下：**

*后面的方法二、方法三查询结果与此相同，小伙伴们可以自行尝试。*

**方法二：使用窗口函数NTILE()**

还可以使用 NTILE() 函数来实现。NTILE() 将数据分为指定数量的桶（例如，5个桶表示20%，近似20%，这个示例刚好是20%）。

```
WITH RankedSalaries AS (              SELECT              department,              id,              salary,              NTILE(5) OVER (PARTITION BY department ORDER BY salary DESC) AS ntile_bucket              FROM data_learning.employee_salaries            )            SELECT department, id, salary            FROM RankedSalaries            WHERE ntile_bucket = 1;            
```

**解释：**

* NTILE(5) OVER (PARTITION BY department\_id ORDER BY salary DESC) 将每个部门的员工根据工资分为5个桶（每个桶约占20%）。
* 外层查询筛选出排名在第一个桶中的员工，即工资排名前20%的员工。

**方法三：使用窗口函数结合子查询**

```
SELECT              department,              id,              salary            FROM              (              SELECT                department,                id,                salary,                ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rn,                COUNT(*) OVER (PARTITION BY department) AS id_cnt              FROM                data_learning.employee_salaries              )a            WHERE salary_rn <= id_cnt * 0.20
```

**解释：**

+ ROW\_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS salary\_rn：这部分使用窗口函数 ROW\_NUMBER() 为每个部门内的员工按照薪资降序分配一个唯一的排名编号（salary\_rn）。即，薪资最高的员工排名为 1，第二高为 2，以此类推。
+ COUNT(\*) OVER (PARTITION BY department) AS id\_cnt：这部分计算每个部门的员工总数（id\_cnt）。COUNT(\*) 函数在每个部门内计算所有员工的数量。由于它是一个窗口函数，结果将包含在每行中，并与其他列一起返回。
+ WHERE salary\_rn <= id\_cnt \* 0.20：这个条件筛选出每个部门中薪资排名在前 20% 的员工。具体来说，salary\_rn 是员工的薪资排名，而 id\_cnt \* 0.20 是部门员工总数的 20%。因此，只有那些排名在前 20% 的员工才会被选中。

今天分享的问题解决方法中大部分都用到了窗口函数，掌握窗口函数的使用真的重要且能提高效率。有需要重温窗口函数知识的小伙伴可以查看往期相关文章：[数据分析工作中常用的3类SQL开窗函数详解](http://mp.weixin.qq.com/s?__biz=MzUyMTUwNDM4NQ==&mid=2247484450&idx=1&sn=5603a556e8612481a3c9e992c1630940&chksm=f9db57ccceacdedab06663b26ec129db8aa5597cd41fc7b823e4a5381ed4df3fdde02aaa08f8&scene=21#wechat_redirect)。    

### **附录**

薪资表建表语句如下：

```
CREATE TABLE data_learning.employee_salaries (              id INT PRIMARY KEY,              department varchar(255),              salary INT NOT NULL            );                       INSERT INTO data_learning.employee_salaries VALUES            (10001,'IT部',19800),            (10002,'IT部',29800),            (10003,'IT部',9800),            (10004,'IT部',12000),            (10005,'IT部',6000),            (10006,'市场部',9900),            (10007,'市场部',10000),            (10008,'市场部',3000),            (10009,'市场部',46000),            (10010,'市场部',7000),            (10011,'财务部',8000),            (10012,'财务部',16000),            (10013,'财务部',5800),            (10014,'财务部',14000),            (10015,'财务部',25000),            (10016,'人力资源部',4500),            (10017,'人力资源部',6500),            (10018,'人力资源部',10000),            (10019,'人力资源部',6000),            (10020,'人力资源部',8000);
```

以上就是今天的分享，感谢观看！

点击【赞】和【在看】，这将是我前进的动力和鼓励！谢谢你的支持！

---

欢迎关注我，一起学习数据知识，一起成长~

👇👇👇

- END -
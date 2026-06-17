---
title: SQL开源替代品，诞生了！
author: 过往记忆大数据
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650740232&idx=1&sn=41cb11eb322eaa9dfdd4e79328650236&chksm=887c0b7ebf0b8268e1866fff0440dedfcc71b96a7d3d0ed7e0f38df237c2757a4ba76ecedc96&mpshare=1&scene=24&srcid=04191CYZKRrJ4SEzDzL1Bfi1&sharer_sharetime=1681866536734&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

发明 SQL 的初衷之一显然是为了降低人们实施数据查询计算的难度。SQL 中用了不少类英语的词汇和语法，这是希望非技术人员也能掌握。确实，简单的 SQL 可以当作英语阅读，即使没有程序设计经验的人也能运用。

然而，面对稍稍复杂的查询计算需求，SQL 就会显得力不从心，经常写出几百行有多层嵌套的语句。这种 SQL，不要说非技术人员难以完成，即使对于专业程序员也不是件容易的事，常常成为很多软件企业应聘考试的重头戏。三行五行的 SQL 仅存在教科书和培训班，现实中用于报表查询的 SQL 通常是以“K”计的。

SQL 困难的分析探讨

这是为什么呢？我们通过一个很简单的例子来考察 SQL 在计算方面的缺点。

设有一个由三个字段构成的销售业绩表（为了简化问题，省去日期信息）：

|  |  |
| --- | --- |
| sales\_amount | 销售业绩表 |
| sales | 销售员姓名，假定无重名 |
| product | 销售的产品 |
| amount | 该销售员在该产品上的销售额 |

现在我们想知道出空调和电视销售额都在前 10 名的销售员名单。

这个问题并不难，人们会很自然地设计出如下计算过程：

1． 按空调销售额排序，找出前 10 名；

2． 按电视销售额排序，找出前 10 名；

3． 对 1、2 的结果取交集，得到答案；

我们现在来用 SQL 做。

早期的 SQL 不支持步骤化，要把前两步写进子查询，显得有点繁琐：

```
select * from     ( select top 10 sales from sales_amount where product='AC' order by amount desc )intersect     ( select top 10 sales from sales_amount where product='TV' order by amount desc )
```

后来，SQL 也发现了不分步会很麻烦，就提供了 CTE 语法，可以用 with 关键字，可以把前面步骤中的查询结果命名后在后面的步骤中使用：

```
with A as       select top 10 sales from sales_amount where product='AC' order by amount desc     B as       select top 10 sales from sales_amount where product='TV' order by amount descselect * from A intersect B
```

句子没有更短，但分步后思路确实变清晰了。

现在，我们把问题稍复杂化一点，改为计算所有产品销售额都在前 10 名的销售员，试想一下应当如何计算，延用上述的思路很容易想到：

1.   列出所有产品；

2．将每种产品的前 10 名取出，分别保存；

3．将所有的前 10 名取交集；

但是，使用 CTE 语法只能针对确定个数的中间结果做进一步的计算。而我们事先不知道总共有多个产品，这会导致 WITH 中子句个数是不确定的，这就写不出来了。

换一种思路：

1．将数据按产品分组，将每组排序，取出前 10 名；

2．将所有的前 10 名取交集；

但这样需要把第一步的分组结果保存起来，而这个中间结果是一个表，其中有个字段要存储对应的分组成员中的前 10 名，也就是字段的取值将是个集合，SQL 不支持这种数据类型，还是写不出来。

如果有窗口函数的支持，可以再转换思路，按产品分组后，计算每个销售员在所有分组的前 10 名中出现的次数，若与产品总数相同，则表示该销售员在所有产品销售额中均在前 10 名内。

```
select salesfrom ( select sales,     from ( select sales,                   rank() over (partition by product order by amount desc ) ranking            from sales_amount)     where ranking <=10 )group by saleshaving count(*)=(select count(distinct product) from sales_amount)
```

这是能写出来，但这样复杂的 SQL，有多少人会写呢？

前两种简单的思路无法用 SQL 实现，只能采用第三种迂回的思路。这里的原因在于 SQL 的一个重要缺点：**集合化不彻底**。  
虽然 SQL 有集合概念，但并未把集合作为一种基础数据类型提供，不能让变量或字段的取值是个集合，除了表之外也没有其它集合形式的数据类型，这使得大量集合运算在思维和书写时都需要绕路。

我们在上面的计算中使用了关键字 top，事实上关系代数理论中没有这个东西（它可以被别的计算组合出来），这不是 SQL 的标准写法。

我们来看一下没有 top 时找前 10 名会有多困难？

大体思路是这样：找出比自己大的成员个数作为是名次，然后取出名次不超过 10 的成员，写出的 SQL 如下：

```
select salesfrom ( select A.sales sales, A.product product,             (select count(*)+1 from sales_amount              where A.product=product AND A.amount<=amount) ranking       from sales_amount A )where product='AC' AND ranking<=10
```

或

```
select salesfrom ( select A.sales sales, A.product product, count(*)+1 ranking       from sales_amount A, sales_amount B       where A.sales=B.sales and A.product=B.product AND A.amount<=B.amount       group by A.sales,A.product )where product='AC' AND ranking<=10
```

这样的 SQL 语句，专业程序员写出来也未必容易吧！而仅仅是计算了一个前 10 名。

退一步讲，即使有 top，那也只是使取出前一部分轻松了。如果我们把问题改成取第 6 至 10 名，或者找比下一名销售额超过 10% 的销售员，这些困难仍然存在，还是要采用迂回的思路才能用 SQL 完成。

造成这个现象的原因就是 SQL 的另一个重要缺点：**缺乏有序支持**。SQL 继承了数学上的无序集合，这直接导致与次序有关的计算相当困难，而可想而知，与次序有关的计算会有多么普遍（诸如比上月、比去年同期、前 20%、排名等）。

SQL2003 标准中增加的窗口函数提供了一些与次序有关的计算能力，这使得上述某些问题可以有较简单的解法，在一定程度上缓解 SQL 的这个问题。但窗口函数的使用经常伴随着子查询，而不能让用户直接使用次序访问集合成员，还是会有许多有序运算难以解决。

我们现在想关注一下上面计算出来的“好”销售员的性别比例，即男女各有多少。一般情况下，销售员的性别信息会记在花名册上而不是业绩表上，简化如下：

|  |  |
| --- | --- |
| employee | 员工表 |
| name | 员工姓名，假定无重名 |
| gender | 员工性别 |

我们已经计算出“好”销售员的名单，比较自然的想法，是用名单到花名册时找出其性别，再计一下数。但在 SQL 中要跨表获得信息需要用表间连接，这样，接着最初的结果，SQL 就会写成：

```
select employee.gender,count(*)from employee,    ( ( select top 10 sales from sales_amount where product='AC' order by amount desc )    intersect    ( select top 10 sales from sales_amount where product='TV' order by amount desc ) ) Awhere A.sales=employee.namegroup by employee.gender
```

仅仅多了一个关联表就会导致如此繁琐，而现实中信息跨表存储的情况相当多，且经常有多层。比如销售员有所在部门，部门有经理，现在我们想知道“好”销售员归哪些经理管，那就要有三个表连接了，想把这个计算中的 where 和 group 写清楚实在不是个轻松的活儿了。

这就是我们要说的 SQL 的下一个重要困难：**缺乏对象引用机制**，关系代数中对象之间的关系完全靠相同的外键值来维持，这不仅在寻找时效率很低，而且无法将外键指向的记录成员直接当作本记录的属性对待，试想，上面的句子可否被写成这样：

```
select sales.gender,count(*)from (…) // …是前面计算“好”销售员的SQLgroup by sales.gender
```

显然，这个句子不仅更清晰，同时计算效率也会更高（没有连接计算）。

我们通过一个简单的例子分析了 SQL 的几个重要困难，这也是 SQL 难写或要写得很长的主要原因。基于一种计算体系解决业务问题的过程，也就是将**业务问题的解法翻译成形式化计算语法的过程**（类似小学生解应用题，将题目翻译成形式化的四则运算）。SQL 的上述困难会造成问题解法翻译的极大障碍，极端情况就会发生这样一种怪现象：**将问题解法形式化成计算语法的难度要远远大于解决问题本身**。

再打个程序员易于理解的比方，用 SQL 做数据计算，类似于用汇编语言完成四则运算。我们很容易写出 3+5\*7 这样的算式，但如果用汇编语言（以 X86 为例），就要写成

```
mov ax,3mov bx,5mul bx,7add ax,bx
```

这样的代码无论书写还是阅读都远不如 3+5\*7 了（要是碰到小数就更要命了）。虽然对于熟练的程序员也算不了太大的麻烦，但对于大多数人而言，这种写法还是过于晦涩难懂了，从这个意义上讲，FORTRAN 确实是个伟大的发明。

为了理解方便，我们举的例子还是非常简单的任务。现实中的任务要远远比这些例子复杂，过程中会面临诸多大大小小的困难。这个问题多写几行，那个问题多写几行，一个稍复杂的任务写出几百行多层嵌套的 SQL 也就不奇怪了。而且这个几百行常常是一个语句，由于工程上的原因，SQL 又很难调试，这又进一步加剧了复杂查询分析的难度。

更多例子

我们再举几个例子来分别说明这几个方面的问题。

为了让例子中的 SQL 尽量简捷，这里大量使用了窗口函数，故而采用了对窗口函数支持较好的 ORACLE 数据库语法，采用其它数据库的语法编写这些 SQL 一般将会更复杂。  
这些问题本身应该也算不上很复杂，都是在日常数据分析中经常会出现的，但已经很难为 SQL 了。

### **集合无序**

有序计算在批量数据计算中非常普遍（取前 3 名 / 第 3 名、比上期等），但 SQL 延用了数学上的无序集合概念，有序计算无法直接进行，只能调整思路变换方法。

**任务 1**  **公司中年龄居中的员工**

```
```
select name, birthdayfrom (select name, birthday, row_number() over (order by birthday) ranking      from employee )where ranking=(select floor((count(*)+1)/2) from employee)
```
```

中位数是个常见的计算，本来只要很简单地在排序后的集合中取出位置居中的成员。但 SQL 的无序集合机制不提供直接用位置访问成员的机制，必须人为造出一个序号字段，再用条件查询方法将其选出，导致必须采用子查询才能完成。

**任务 2**  **某支股票最长连续涨了多少交易日**

```
select max (consecutive_day)from (select count(*) (consecutive_day      from (select sum(rise_mark) over(order by trade_date) days_no_gain            from (select trade_date,                         case when                              closing_price>lag(closing_price) over(order by trade_date)                         then 0 else 1 END rise_mark                from stock_price) )     group by days_no_gain)
```

无序的集合也会导致思路变形。

常规的计算连涨日数思路：设定一初始为 0 的临时变量记录连涨日期，然后和上一日比较，如果未涨则将其清 0，涨了再加 1，循环结束看该值出现的最大值。

使用 SQL 时无法描述此过程，需要转换思路，计算从初始日期到当日的累计不涨日数，不涨日数相同者即是连续上涨的交易日，针对其分组即可拆出连续上涨的区间，再求其最大计数。这句 SQL 读懂已经不易，写出来则更困难了。

### **集合化不彻底**

毫无疑问，集合是批量数据计算的基础。SQL 虽然有集合概念，但只限于描述简单的结果集，没有将集合作为一种基本的数据类型以扩大其应用范围。

**任务 3**  **公司中与其他人生日相同的员工**

```
select * from employeewhere to_char (birthday, ‘MMDD’) in    ( select to_char(birthday, 'MMDD') from employee      group by to_char(birthday, 'MMDD')      having count(*)>1 )
```

分组的本意是将源集合分拆成的多个子集合，其返回值也应当是这些子集。但 SQL 无法表示这种“由集合构成的集合”，因而强迫进行下一步针对这些子集的汇总计算而形成常规的结果集。

但有时我们想得到的并非针对子集的汇总值而是子集本身。这时就必须从源集合中使用分组得到的条件再次查询，子查询又不可避免地出现。

**任务 4**  **找出各科成绩都在前 10 名的学生**

```
select namefrom (select name      from (select name,                   rank() over(partition by subject order by score DESC) ranking            from score_table)      where ranking<=10)group by namehaving count(*)=(select count(distinct subject) from score_table)
```

用集合化的思路，针对科目分组后的子集进行排序和过滤选出各个科目的前 10 名，然后再将这些子集做交集即可完成任务。但 SQL 无法表达“集合的集合”，也没有针对不定数量集合的交运算，这时需要改变思路，利用窗口函数找出各科目前 10 名后再按学生分组找出出现次数等于科目数量的学生，造成理解困难。

### **缺乏对象引用**

在 SQL 中，数据表之间的引用关系依靠同值外键来维系，无法将外键指向的记录直接用作本记录的属性，在查询时需要借助多表连接或子查询才能完成，不仅书写繁琐而且运算效率低下。

**任务 5**  **女经理的男员工们**

用多表连接

```
select A.*from employee A, department B, employee Cwhere A.department=B.department and B.manager=C.name and      A.gender='male' and C.gender='female'
```

用子查询

```
select * from employeewhere gender='male' and department in    (select department from department     where manager in          (select name from employee where gender='female'))
```

如果员工表中的部门字段是指向部门表中的记录，而部门表中的经理字段是指向员工表的记录，那么这个查询条件只要简单地写成这种直观高效的形式：

```
where gender='male' and department.manager.gender='female'
```

但在 SQL 中则只能使用多表连接或子查询，写出上面那两种明显晦涩的语句。

**任务 6** **员工的首份工作公司**

用多表连接

```
select name, company, first_companyfrom (select employee.name name, resume.company company,             row_number() over(partition by resume. name                               order by resume.start_date) work_seq      from employee, resume where employee.name = resume.name)where work_seq=1
```

用子查询

```
select name,    (select company from resume     where name=A.name and           start date=(select min(start_date) from resume                       where name=A.name)) first_companyfrom employee A
```

没有对象引用机制和彻底集合化的 SQL，也不能将子表作主表的属性（字段值）处理。针对子表的查询要么使用多表连接，增加语句的复杂度，还要将结果集用过滤或分组转成与主表记录一一对应的情况（连接后的记录与子表一一对应）；要么采用子查询，每次临时计算出与主表记录相关的子表记录子集，增加整体计算量（子查询不能用 with 子句了）和书写繁琐度。

SPL 的引入

问题说完，该说解决方案了。

其实在分析问题时也就一定程度地指明了解决方案，重新设计计算语言，克服掉 SQL 的这几个难点，问题也就解决了。

这就是发明 SPL 的初衷！

SPL 是个开源的程序语言，其全名是 Structured Process Language，和 SQL 只差一个词。目的在于更好的解决结构化数据的运算。SPL 中强调了有序性、支持对象引用机制、从而得到彻底的集合化，这些都会大幅降低前面说的“解法翻译”难度。

这里的篇幅不合适详细介绍 SPL 了，我们只把上一节中的例子的 SPL 代码罗列出来感受一下：

**任务 1**

|  | A |
| --- | --- |
| 1 | =employee.sort(birthday) |
| 2 | =A1((A1.len()+1)/2) |

对于以有序集合为基础的 SPL 来说，按位置取值是个很简单的任务。

**任务 2**

|  |  |
| --- | --- |
|  | A |
| 1 | =stock\_price.sort(trade\_date) |
| 2 | =0 |
| 3 | =A1.max(A2=if(close\_price>close\_price[-1],A2+1,0)) |

SPL 按自然的思路过程编写计算代码即可。

**任务 3**

|  | A |
| --- | --- |
| 1 | =employee.group(month(birthday),day(birthday)) |
| 2 | =A1.select(~.len()>1).conj() |

SPL 可以保存分组结果集，继续处理就和常规集合一样。

**任务 4**

|  | A |
| --- | --- |
| 1 | =score\_table.group(subject) |
| 2 | =A1.(~.rank(score).pselect@a(~<=10)) |
| 3 | =A1.(~(A2(#)).(name)).isect() |

使用 SPL 只要按思路过程写出计算代码即可。

**任务 5**

|  | A |
| --- | --- |
| 1 | =employee.select(gender=="male" && department.manager.gender=="female") |

支持对象引用的 SPL 可以简单地将外键指向记录的字段当作自己的属性访问。

**任务 6**

|  | A |
| --- | --- |
| 1 | =employee.new(name,resume.minp(start\_date).company:first\_company) |

SPL 支持将子表集合作为主表字段，就如同访问其它字段一样，子表无需重复计算。

SPL 有直观的 IDE，提供了方便的调试功能，可以单步跟踪代码，进一步降低代码的编写复杂度。

对于应用程序中的计算，SPL 提供了标准的 JDBC 驱动，可以像 SQL 一样集成到 Java 应用程序中：

```
…Class.forName("com.esproc.jdbc.InternalDriver");Connection conn =DriverManager.getConnection("jdbc:esproc:local://");Statement st = connection.();CallableStatement st = conn.prepareCall("{call xxxx(?,?)}");st.setObject(1, 3000);st.setObject(2, 5000);ResultSet result=st.execute();...
```

GitHub：*https://github.com/SPLWare/esProc*

**重磅！开源SPL交流群成立了**

简单好用的SPL开源啦！

为了给感兴趣的小伙伴们提供一个相互交流的平台，

特地开通了交流群（群完全免费，不广告不卖课）

需要进群的朋友，可长按扫描下方二维码

**本文感兴趣的朋友，请到阅读原文去收藏 ^\_^**
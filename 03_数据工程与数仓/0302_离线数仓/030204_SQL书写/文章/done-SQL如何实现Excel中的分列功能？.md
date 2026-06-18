---
title: SQL如何实现Excel中的分列功能？
author: 李岳AI
date:
url: http://mp.weixin.qq.com/s?__biz=MzA3MTg4NjY4Mw==&mid=2457336079&idx=2&sn=34fb456f76925e6a63c1c9740070eb4a&chksm=88a5eb3bbfd2622d85356784813f58cfecbe0634f0f7d3ca304f448a28b62f217ab959e9b601&mpshare=1&scene=24&srcid=12295LwXOa2yyF5YGm6QnsA3&sharer_shareinfo=2c7121ea18b3be8f4738d505cfb1bdec&sharer_shareinfo_first=2c7121ea18b3be8f4738d505cfb1bdec#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL行列转换与空值处理|SQL行列转换与空值处理]]


点击关注公众号，SQL干货及时获取

```
```
后台回复：1024，获取海量学习资源

SQL刷题专栏

SQL145题系列

大家注意：

因为微信改了推送机制，会有小伙伴刷不到当天的文章，

一些比较实用的知识和信息，错过了就是错过了。

所以建议大家加个星标，就能第一时间收到推送了。
```
```

我们在处理SQL里的数据时候，时不时会遇到对字符串进行分割的情况。类似Excel中按指定字符进行分列，今天给大家介绍两种处理方法。

**借助Excel进行分割**

先将数据从数据库导出到Excel，使用Excel进行分列后再导入到数据库中。注意再次导入需要改变表结构，因为分列后数据字段变多了，必须新建列进行匹配。

**使用函数进行分割**

使用CHARINDEX函数，CHARINDEX函数的作用是如果能够找到对应的字符串，就返回该字符串的位置，否则返回0.
语法如下：

> CHARINDEX(expressionTarget,expressionSource[,start\_location])
> expressionTarget：是我们要查找的目标字符串 
>
> expressionSource：是被查找的字符串 
>
> start\_location：开始查找的起始位置，默认为空表示从第一位开始查找

例如：

```
SELECT  CHARINDEX('Road','SQL_Road')
```

返回的结果为：5
就是表示字符串'Road'在字符串'SQL\_Road'的第5个位置。回到我们分列的用法上，我们可以这样写：

```
SELECT  
'ABCD,BDEF' AS R,
LEFT('ABCD,BDEF',CHARINDEX(',','ABCD,BDEF')-1) AS R1 ,
RIGHT('ABCD,BDEF',(LEN('ABCD,BDEF') - CHARINDEX(',','ABCD,BDEF'))) AS R2
```

（提示：可以左右滑动代码）

返回的结果为

上面是对字符串'ABCD,BDEF'按照逗号(,)进行分列。方法固定，如果是对其他符号进行分列，只需要修改其中的符号即可。

以上就是两种我常使用的办法，希望对大家有帮助。

我是岳哥，最后给大家分享我写的SQL两件套：**《SQL基础知识第二版》**和**《SQL高级知识第二版》**的PDF电子版。里面有各个语法的解释、大量的实例讲解和批注等等，非常通俗易懂，方便大家跟着一起来实操。

有需要的读者可以下载学习，在下面的公众号「**数据前线**」(非本号)后台回复关键字：**SQL**，就行

**数据前线**

**——End——**

#### ``` 后台回复关键字：1024，获取一份精心整理的技术干货 后台回复关键字：进群，带你进入高手如云的交流群。 ``` ``` ``` ``` ``` ``` 推荐阅读 ``` * Navicat使用指南（下） * SQL优化万能公式：5 大步骤 + 10 个案例 * 图解 SQL 执行顺序，通俗易懂！ * 如何执行超过100M的SQL脚本？ * SQL自定义排序 ``` ``` ``` ``` ``` ``` 文章有帮助的话，在看，转发吧。 谢谢支持哟 (*^__^*） ``` ``` ``` ```
---
title: 使用Excel快速生成SQL语句，用过的人都说好
author: 数据前线
date:
url: http://mp.weixin.qq.com/s?__biz=MzU1NTU3OTk1Ng==&mid=2247490330&idx=1&sn=2de7410e1ea6d925fa30d1fb348a8b68&chksm=fbd37441cca4fd57227705d98f59a15043df216de75888ce64794189463f5b87c01978a9422f&mpshare=1&scene=24&srcid=0606qDA3NzIHNHye5aDGnM4J&sharer_sharetime=1686047010674&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090208_Office文档工具/090208_核心知识点/Office文档自动化与AI生成边界|Office 文档自动化与 AI 生成边界]]


**点击关注上方“数据前线”，**

```
设为“置顶或星标”，第一时间送达干货

```
```
SQL刷题专栏

SQL145题系列
```
```
```

Excel的公式自动生成想必大家都知道了，就是写好一个公式后直接往下拖，就可以将后面数据的公式自动生成。

今天我们就用这个功能来快速生成SQL语句。

**导入Excel数据**

Excel的数据有多种方式，这里我们演示用SQL代码导入Excel中的数据。

例如我们想把左边Excel中的数据插入到数据库中，如下图：

**写好模板语句**

我们可以先写一条插入语句，如下：

```
INSERT INTO Person VALUES(1,'吕布',25,'男','13500000001')
```

然后复制这条SQL语句打开Excel，选中表格后的一个单元格，在上方函数位置粘贴刚才的SQL语句并做修改，

```
="INSERT INTO Person VALUES("&A2&",'"&B2&"',"&C2&",'"&D2&"','"&E2&"')"
```

注意前面有个= 然后整个SQL用 ""包围住。

**生成SQL语句**

确认后就可以看到在单元格中会自动生成一条SQL语句。选中单元格下拉，会发现所有的行后面都会生成一条SQL语句。

**执行SQL**

然后我们直接复制这些SQL语句到数据库的查询窗口执行。

执行完后我们查询Person表里的数据。

这样就完成了Excel快速生成SQL语句的功能。

**扩展SQL示例**

以上只是一个简单的示例，运用这种方法我们还可以自动生成很多其他的SQL脚本，比如要查询数据库中所有表中的记录数。

当然我们可以使用循环遍历系统中的所有表然后再用循环语句执行指定的语句，如下：

```
--使用循环语句查询所有表的数量DECLARETNAME VARCHAR2(200);BEGIN--获取系统表中的所有表名  FOR X IN (SELECT TABLE_NAME FROM user_tables where table_name like 'HR_TEMPTABLE%')--开始循环  LOOP  --循环主体部分
    TNAME :=X.TABLE_NAME;    --赋值    EXECUTE IMMEDIATE 'SELECT '''X.TABLE_NAME'''||',COUNT(1) Num FROM '||X.TABLE_NAME;  --执行循环主体  END LOOP;  EXCEPTION    WHEN OTHERS THEN      DBMS_OUTPUT.put_line(TNAME);      RAISE;END;END;
```

**套用Excel生成SQL方法**

但是如果是新手同学，不会写上面的代码，而此时又要我们做这样的事怎么办呢？就可以使用上面的方法了。

可以先从系统表中查询出所有的表名

```
SELECT TABLE_NAME FROM user_tables
```

将表名复制粘贴到Excel中，然后开始写查询语句，如下图：

然后将这些代码复制粘贴到查询窗口即可查询出所有表中的记录数了。

使用此方法还可以应用在很多类似的场景，他们的共同点就是代码结构一样，但是代码中的参数不一样，对于想快速写出相应的SQL代码是非常有效的。

觉得不错，欢迎转发分享给更多人~

```
觉得不错，记得帮忙点下「在看」，谢谢啦
```

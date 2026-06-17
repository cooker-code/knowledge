---
title: 【sql-formatter】全新开源工具 ｜ 重新定义sql美化和效率分析
author: 王长宸
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3Nzg3MzU5OA==&mid=2247484413&idx=1&sn=07bda7fd528dc21724647adbb7a40845&chksm=ce8fe42cc725f067d448a0c1133f60f4f3f229a02d05f90fb134e4102e7af65a9ce11bbcdf04&mpshare=1&scene=24&srcid=0318K8Gp0qNLwKMlS3B57CzG&sharer_shareinfo=052d68cd8ad8e0022caba58adde1b868&sharer_shareinfo_first=052d68cd8ad8e0022caba58adde1b868#rd
---

**“** sql-formatter  ｜ 重新定义sql美化和效率分析**”**

    大家好，上一期我和大家分享了sql-formatter，作为一款sql美化工具（您输入sql自动帮您排版，以及分析哪里可能出现错误。），我认为应该很多数据同僚都应该需要一款这样的应用！今天我又对工具进行了一些优化，和大家分享。

01

—

直连数据库，分析sql代码

    首先请大家先点击右上角，链接您的数据库（项目以mysql数据库为例）：

然后，我就用sql优化当中比较典型的例子来开场：

```
select * from table_x
```

    大家第一反应可能没有什么，但是如果您的数据是mysql，数据量又异常之大，那么这个语句的效率，就有点一言难尽了，我在输入这个语句之后点击"分析SQL"按钮，结果如下：

    大家看！sql-formatter告诉我们，可以考虑where后面加一个通过索引过滤的条件，避免全表查；然后又告诉我们，可以把select \* 修改成我们需要的字段：如 select column1, column2 ..是不是很棒！

    继续下拉！看看还有什么内容：

    sql-formatter自动帮我们运行了explain语句，帮我们识别出来了我们上面的sql代码没有使用索引，也告诉我们表中的索引是user\_id了！可以！那我们现在就优化一下试试看！

02

—

尝试修改sql代码

    那么我现在先修改一下代码试试，看看sql-formatter会给我们什么反馈：

```
select user_id from user_info where user_id = 'user_0001'
```

    真不错呀！它告诉我们查询的优化结果良好！不错不错，我们再点击执行计划那里看看现在是什么情况：

    我们发现我们已经用上了索引了！效率得到大大提升！棒！

03

—

git地址 | B站地址

具体git链接如下：

> https://github.com/changchenwang/sql-formatter.git
>
> github

如果大家对使用视频感兴趣，欢迎观看这集视频：

> https://www.bilibili.com/video/BV1C7wRzjE6h/
>
> B站

    如果大家不介意的话，欢迎多多可以下载使用，一款可以自定义的sql美化工具，感觉在工作中也会让我们轻松不少（拒绝恶搞）。深知会有很多不足，也一定会有很多不足，大家如果感兴趣的话，您的下载和创新评论，将会是我特别大的鼓舞！

    您如果有好的想法，欢迎各种尝试！放在大模型里，让大模型帮您改代码。期望收集各位大佬极具创新的idea！

下期见！
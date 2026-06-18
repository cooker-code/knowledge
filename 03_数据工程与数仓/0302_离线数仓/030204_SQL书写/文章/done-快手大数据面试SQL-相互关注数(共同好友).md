---
title: 快手大数据面试SQL-相互关注数(共同好友)
author: 大模型数据的朋友
date:
url: http://mp.weixin.qq.com/s?__biz=MzUzMjAxNDA5OQ==&mid=2247484087&idx=1&sn=d55e6d26bc2528ee11ad3b229a4121c8&chksm=fab8f7cbcdcf7eddb7831042259703c3f608dd1898715fec31dc0ad3c3b6a34b8628aba278eb&mpshare=1&scene=24&srcid=0709DIm2WA5i3sioXFVLq6up&sharer_shareinfo=2247d1293eaa4b8f2be6c554580b09cf&sharer_shareinfo_first=2247d1293eaa4b8f2be6c554580b09cf#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL口径验证与对账闭环|SQL口径验证与对账闭环]]


有一个用户粉丝表fans，表中有两个字段：

* from\_user 关注者 用户ID，
* to\_user\_array 被关注者的一个数组，关注者的粉丝ID集。

如果两个用户互相关注，则代表为共同好用，请找出共同好友数。你可以先思考下，如果被问到的话有没有解决思路了？没有思路再看以下解题思路。

**数据**

```
fans +------------+----------------+| from_user  | to_user_array  |+------------+----------------+| A          | ["B","C","D"]  || B          | ["A","C","D"]  || C          | ["A","D"]      || D          | ["A"]          |+------------+----------------+-
```

**思路一：** 

先使用explode函数将数组炸裂开，再利用concat\_ws函数将用户拼接，判断拼接的字符串大小 利用reverse函数进行反转即可得到相互关注的用户对，最后count这个用户对，筛选等于2的即为相互关注的用户，

具体sql如下所示：

```
with fans as (select 'A' as from_user, split('B,C,D',',') as to_user_arrayunion all  select 'B' as from_user, split('A,C,D',',') as to_user_arrayunion all  select 'C' as from_user, split('A,D',',') as to_user_arrayunion all  select 'D' as from_user, split('A',',') as to_user_array),explode_nm as (select   from_user,  explode(to_user_array) as to_user  from fans ),  concat_nm as (select  from_user,   to_user,   concat_ws('-', from_user, to_user) as from_to_userfrom explode_nm), reverse_nm as (select  from_user,  to_user,  from_to_user,  if(from_user > to_user, concat_ws('-', to_user, from_user), concat_ws('-', from_user, to_user)) as feature,   if(substr(from_to_user,1,1) > substr(from_to_user,-1,1), reverse(from_to_user), from_to_user) as follow_each_otherfrom concat_nm  ) select   from_user,  to_user,  follow_each_other,   if(sum(1) over (partition by feature) > 1, 1, 0) as is_friend,  count(follow_each_other) over (partition by feature) as follow_each_other_cnt      from  reverse_nm  order by 1,2,3
```

最后的结果数据如下：

```
+------------+----------+--------------------+------------+------------------------+| from_user  | to_user  | follow_each_other  | is_friend  | follow_each_other_cnt  |+------------+----------+--------------------+------------+------------------------+| A          | B        | A-B                | 1          | 2                      || A          | C        | A-C                | 1          | 2                      || A          | D        | A-D                | 1          | 2                      || B          | A        | A-B                | 1          | 2                      || B          | C        | B-C                | 0          | 1                      || B          | D        | B-D                | 0          | 1                      || C          | A        | A-C                | 1          | 2                      || C          | D        | C-D                | 0          | 1                      || D          | A        | A-D                | 1          | 2                      |+------------+----------+--------------------+------------+------------------------+
```

**思路二：**

两个表自关联，A表的from\_user等于B表的to\_user，且A表的to\_user等于B表的from\_user，具体sql如下所示：

```
with fans as (select 'A' as from_user, split('B,C,D',',') as to_user_arrayunion all  select 'B' as from_user, split('A,C,D',',') as to_user_arrayunion all  select 'C' as from_user, split('A,D',',') as to_user_arrayunion all  select 'D' as from_user, split('A',',') as to_user_array),explode_nm as (select   from_user,  explode(to_user_array) as to_user  from fans ),concat_nm as (select  from_user,   to_user,   concat_ws('-', from_user, to_user) as from_to_userfrom explode_nm)selectt1.from_user,t1.to_user, if(t2.from_user is not null, 1, 0) as is_friendfrom concat_nm t1 left join concat_nm t2 on t1.from_user = t2.to_user and t1.to_user=t2.from_user
```

最后的结果数据如下所示：

```
+------------+----------+------------+| from_user  | to_user  | is_friend  |+------------+----------+------------+| A          | B        | 1          || A          | C        | 1          | | A          | D        | 1          | | B          | A        | 1          | | B          | C        | 0          | | B          | D        | 0          || C          | A        | 1          | | C          | D        | 0          | | D          | A        | 1          |+------------+----------+------------+
```

**思路三：**

利用A表的from\_user 对应B表中的to\_user，A表的to\_user对应B表的from\_user.

```
with fans as (select 'A' as from_user, split('B,C,D',',') as to_user_arrayunion all  select 'B' as from_user, split('A,C,D',',') as to_user_arrayunion all  select 'C' as from_user, split('A,D',',') as to_user_arrayunion all  select 'D' as from_user, split('A',',') as to_user_array),explode_nm as (select   from_user,  explode(to_user_array) as to_user  from fans ),concat_nm as (select  from_user,   to_user,   concat_ws('-', from_user, to_user) as from_to_userfrom explode_nm)select  u1, u2, cat, count(1) as follow_each_other_cnt from (select  from_user as u1,   to_user as u2, concat_ws('-', from_user, to_user) as cat from  concat_nm union all  select    to_user as u1, from_user as u2,  concat_ws('-', to_user, from_user) as cat  from  concat_nm ) t group by 1,2,3 having count(1) = 2order by 1,2,3
```

最后的结果数据如下：

```
+-----+-----+------+------------------------+--+| u1  | u2  | cat  | follow_each_other_cnt  |+-----+-----+------+------------------------+--+| A   | B   | A-B  | 2                      || A   | C   | A-C  | 2                      || A   | D   | A-D  | 2                      || B   | A   | B-A  | 2                      || C   | A   | C-A  | 2                      || D   | A   | D-A  | 2                      |+-----+-----+------+------------------------+-
```

-关注我-

90后北漂男孩

现任互联网大厂大数据工程师

想要影响1000+个人变得更好

一起遇见未知的自己

一起人生长跑

成为自己的光

💛

多点一下**在看**多一条小鱼干
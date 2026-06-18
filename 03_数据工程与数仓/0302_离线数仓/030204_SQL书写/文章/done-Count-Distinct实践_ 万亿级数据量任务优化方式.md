> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL聚合去重与膨胀治理|SQL聚合去重与膨胀治理]]
---
title: Count-Distinct实践: 万亿级数据量任务优化方式
author: Flink实战剖析
date:
url: http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247486017&idx=1&sn=97925fcac7b00ce66b65bdffe32f08ad&chksm=fe2b6e0ec95ce718b9baaf4c820911191cc225b37c1cf28c3433d8640d8d871bbda28851087b&mpshare=1&scene=24&srcid=0325Nhfi6nS7Qej0MokbeCjs&sharer_sharetime=1648178002509&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

[join实践: 万亿级数据量任务优化历程](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247485940&idx=1&sn=dbf8c8a03c8ecdd6b70ed0518ec1d78b&chksm=fe2b6dbbc95ce4ad61e1104d656faba47b2f4bda2a07a76806fc9aac41940a2b969326b84954&scene=21#wechat_redirect)

## **单字段去重**

先看一个简单的sql ，pv\_id 去重计数

```
SELECT     visit_type,   count(DISTINCT pv_id)  as pv_cntfrom exp_table where ds=20220320group by visit_type;
```

在默认情况下，相同的visit\_type 的pv\_id 会被分配到同一个reducer中处理，如果某个visit\_type的数据量特别大，那么对应的reducer执行耗时会比较久或者可能会发生OOM，因此常规优化方式是：

```
select visit_type,count(*)from (SELECT    visit_type,pv_idfrom exp_tablewhere ds=20220320group by visit_type,pv_id) group by visit_type;
```

也就是将**count distinct 转换为 group by 操作**，第一层根据visit\_type,pv\_id分组，第二层根据visit\_type 直接求和即可，使数据分布更加均匀。但是 这种方式在第二层group by 也可能会产生大量的数据shuffle操作，可以再次优化：

```
select visit_type,sum(cnt)from (SELECT    visit_type,  count(distinct pv_id) as cntfrom exp_tablewhere ds=20220320group by visit_type,hash(pv_id)%50) group by visit_type;
```

第一层使用visit\_type+hash(pv\_id)%50 方式分组，对相同visit\_type下的pv\_id分了50组，保证相同pv\_id 都能分配到相同的reducer中去，然后执行去重计数(cnt)操作，然后在第二层中根据visit\_type 分组，对cnt求和即可。这种方式在第二层shuffle过程中数据就会相对减少很多。

## **多字段去重**

```
SELECT    visit_type,  count(distinct pv_id),  count(distinct item_id)from exp_tablewhere ds=20220320group by visit_type;
```

这次同时需要对pv\_id与item\_id去重计数，如果还是按照上述的优化方式将visit\_type、pv\_id、item\_id组合很显然已经行不通了，没办法保证相同的session\_id或者item\_id都会分配在同一个reducer中去。先使用常规意义上的操作：

```
SELECT  a.visit_type        ,a.cnt1        ,b.cnt2FROM    (            SELECT  visit_type                    ,count(*) AS cnt1            FROM    (                        SELECT  visit_type                                ,pv_id                        FROM    exp_table                        WHERE   ds = 20220320                        GROUP BY visit_type                                 ,pv_id                    )             GROUP BY visit_type        ) ajoin    (            SELECT  visit_type                    ,count(*) AS cnt2            FROM    (                        SELECT  visit_type                                ,item_id                        FROM    exp_table                        WHERE   ds = 20220320                        GROUP BY visit_type                                 ,item_id                    )             GROUP BY visit_type        ) bON      a.visit_type = b.visit_type;
```

也就是**先拆分再join**， 很显然这种方式开发难度大，特别是在处理字段更多的情况下。再重新按照单字段优化方式思考，希望按照所有的去重字段组合的情况下，仍然能够保证相同pv\_id或者item\_id都会分配在同一个reducer中去处理， 也是pv\_id与item\_id各自不影响其分配方式，可以采取先**扩充数据**，即将每一条数据扩充到去重字段个数的倍数，并且保证一个去重的字段不为空，并且增加标识字段，表明去重的列，如下图：

扩充后的数据执行常规的去重操作，即然后组合去重字段分组然后最外层进行汇总，由于扩充之后的数据每一条只有一个不为空的列，那么在执行shuffle 的时候，相同的pv\_id或者item\_id一定会分配在同一个reducer中去处理。数据扩充使用udtf实现：

```
 @Override    public void process(Object[] args) throws UDFException {        // TODO       for(int i=0;i<args.length;i++){           Object[] nObjects=new Object[args.length+1];           for(int j=0;j<args.length;j++){                if(i==j) {                    nObjects[j]=args[i];                }else{                    nObjects[j]=null;                }
           }          nObjects[args.length]="flag"+i;          this.forward(nObjects);       }    }
```

具体优化sql:

```
SELECT  visit_type        ,count(CASE WHEN TYPE='flag0' THEN 1 END) AS pv_cnt        ,count(CASE WHEN TYPE='flag1' THEN 1 END) AS item_cntFROM    (            SELECT  visit_type                    ,pv_id1                    ,item_id1                    ,type            FROM    (                        SELECT  visit_type                                ,pv_id1                                ,item_id1                                ,type                        FROM    exp_table                        LATERAL VIEW ExpandHash(pv_id,item_id) tmp AS pv_id1,item_id1,type                        WHERE   ds = 20220320                    )             GROUP BY visit_type                     ,pv_id1                     ,item_id1                     ,type        ) GROUP BY visit_type
```

这种方式导致了数据量翻倍，在shuffle阶段io 也会耗时增加，具体耗时、资源消耗以实际情况为准，然后去做均衡具体选择哪一种方式。

**思考**

**Q:** 同时存在count distinct 与 sum 类的聚合该如何优化倾斜问题？

历史推荐

[AliExpress 基于Flink的实时数仓建设](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247485296&idx=1&sn=eb2ce4e9520df3af3d82e8380f5a7c86&chksm=fe2b633fc95cea29d74506cb447b5ec2b50d3153455538f10678de25a8581d997838d54fa566&scene=21#wechat_redirect)

[Flink 程序设计之道](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247485584&idx=1&sn=df35763f3b0798880e66a609ed7c7482&chksm=fe2b6cdfc95ce5c90b7b314197db795dd3a14cf8504c9660f6d2fe0d5bc97a4f9a9a20d513b6&scene=21#wechat_redirect)

[数仓指标一致性](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247484722&idx=1&sn=1d333147aafa3547e341e996aaccf59a&chksm=fe2b617dc95ce86b3d431a2235efeb140419a74c4a5a1bddfc1ff4877a3106640322cc1d2744&scene=21#wechat_redirect)

[关于Event-Time 所带来的的问题](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247484267&idx=1&sn=7cfe1bb5c4a8f46d2408916e966f033d&chksm=fe2b6724c95cee324ee0a584cba78446f4ca1e101db94b216a712f84d652021385fe2cb05bc3&scene=21#wechat_redirect)

[不得不掌握的三种BitMap](http://mp.weixin.qq.com/s?__biz=MzU5MTc1NDUyOA==&mid=2247484130&idx=1&sn=903afed428973f9e935ccdd46125a6d4&chksm=fe2b66adc95cefbbca512591a7e686ce50cb43cc88fdac6a469ed3bc2b027d28f569badd8a26&scene=21#wechat_redirect)
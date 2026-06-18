---
title: AI的SQL优化能力，取决于你问问题的能力！
author: 潇湘秦
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247489061&idx=1&sn=8f9a52d8242742061e3bb3a0b9bd7b9d&chksm=c01c58e4178a15b1247c70f2b1512c3cffa1b8f81d0008df3a2ef977cca8c6d16c6a8f0c4dea&mpshare=1&scene=24&srcid=0314yQOv34teswOIZCrHW6Dp&sharer_shareinfo=b0e1148589ff34a006af64783b09d72a&sharer_shareinfo_first=b0e1148589ff34a006af64783b09d72a#rd
---

# 1.只给SQL，AI优化能力怎么样？

上周开发同事发给我一个sql，说这个sql需要调优一下，看着执行计划还好，但是执行时间超过15秒

具体sql如下：

```
SELECT COUNT(*) AS COUNT   
FROM (  
    SELECT DATA_4_VALUE || '/' || DATA_1_VALUE AS AA,   
           KEY_1_VALUE,   
           DATA_1_VALUE AS INSTANCE_ID,   
           DATA_4_VALUE AS DEFECT_IPQC_CATEGORY  
    FROM REFERENCE_FILE_DETAIL R  
    JOIN NAMED_OBJECT N ON R.REFERENCE_FILE_RRN = N.INSTANCE_RRN  
    LEFT JOIN (  
        SELECT REASON_DETAIL, COUNT(1) AS RANK_NUM  
        FROM CELL_INFO_HISTORY  
        WHERE CREATE_DATE >= ADD_MONTHS(SYSDATE, -1)  
        GROUP BY REASON_DETAIL  
    ) T ON R.DATA_1_VALUE = T.REASON_DETAIL  
    WHERE N.OBJECT = 'REFERENCEFILE'  
      AND N.INSTANCE_ID = '$DEFECT_IPQC'  
      AND N.NAMED_SPACE = 'MYCIM2'  
    ORDER BY T.RANK_NUM DESC NULLS LAST, DATA_4_VALUE, KEY_1_VALUE  
);
```

单纯看sql的执行计划，感觉相对还好

*那么我们试试AI的sql优化能力怎么样？首先我先试试将sql直接丢给deepseek，如下是deepseek给出的优化建议*

### 优化点：

1. **减少嵌套子查询**：将最内层的子查询合并到主查询中，减少嵌套层级。
2. **避免不必要的计算**：`NVL(T.RANK_NUM, 0)` 可以在外层处理，减少内层计算。
3. **索引优化**：确保 `REFERENCE_FILE_DETAIL.REFERENCE_FILE_RRN`、`NAMED_OBJECT.INSTANCE_RRN`、`CELL_INFO_HISTORY.REASON_DETAIL` 和 `CELL_INFO_HISTORY.CREATE_DATE` 上有索引。
4. **过滤条件提前**：将 `NAMED_SPACE = 'MYCIM2'` 提前到 `WHERE` 子句中，减少连接的数据量。
5. **去除不必要的字段**：在最终查询中只选择需要的字段，减少数据传输量。

### 进一步优化建议：

* 如果 `CELL_INFO_HISTORY` 表数据量较大，可以考虑对 `CREATE_DATE` 和 `REASON_DETAIL` 创建复合索引。
* 如果 `NAMED_OBJECT` 表的 `OBJECT` 和 `INSTANCE_ID` 字段选择性较高，可以为其创建复合索引。

***因为没有将执行计划，统计信息等信息给AI，AI仅仅根据sql给出的优化建议 更像是CRO，基于规则来给出的优化建议；优化建议也是基于通用的sql编码最佳实践来进行给建议，但是对于一般的优化也是有一定的参考意义的。***

# 2.给AI更详细的执行信息

*那么我把更为详细的信息丢给AI，来试试AI的优化能力怎么样呢？*

*如下我把这个sql的sqlmonitor信息全部丢给AI来试试，他的优化能力会不会更好？*

*根据sqlid抓取sqlmonitor信息*

```
SELECT DBMS_SQLTUNE.report_sql_monitor(sql_id => '&sql_id', type => 'TEXT') AS report FROM dual;
```

sql monitor中有详细的执行计划，执行时间，IO信息，等待信息等等，这些信息足够AI对这个sql做出判断

## **如下是ChatGpt给出的优化建议**

## **SQL 性能分析与优化方案**

---

### **✅ 1. SQL 主要问题分析**

从 **SQL Monitoring Report** 可以看出以下几个 **性能瓶颈**：

1️⃣ **查询执行时间较长（17 秒）**

* 主要消耗在 **`CELL_INFO_HISTORY`** 表的查询上（Id = 10, 11）。
* **`INDEX RANGE SCAN (IDX_CREATE_DATE)`**

  处理 **20M 行数据**，可能索引未优化。
* `NESTED LOOPS OUTER`

  连接方式可能导致 **大量随机 I/O**。

2️⃣ **Buffer Gets 非常高（14M）**

* 说明 SQL 扫描的数据量过大，**可能是缺乏合适的索引或不合理的嵌套查询**。

3️⃣ **高 CPU 消耗（75% 计算集中在 CELL\_INFO\_HISTORY 上）**

* **GROUP BY 和嵌套查询的 Hash Group By 可能占用大量 CPU 计算**

  。
* **可能缺乏适当的索引**

  ，导致大量表扫描。

---

### **✅ 2. SQL 优化方案**

### **📌 方案 1：优化索引**

🔹 **在 `CELL_INFO_HISTORY` 表上创建复合索引**

```
CREATE INDEX IDX_CELL_INFO_HISTORY ON CELL_INFO_HISTORY (REASON_DETAIL, REASON_CODE, CREATE_DATE);
```

**📌 作用：**

* **减少 `IDX_CREATE_DATE` 过大的索引范围扫描**
* **让查询的 `GROUP BY` 更快执行**

🔹 **优化 `NAMED_OBJECT` 查询索引**

```
CREATE INDEX IDX_NAMED_OBJECT_INSTANCE ON NAMED_OBJECT (INSTANCE_ID, OBJECT);
```

**📌 作用：**

* 让 `NAMED_OBJECT` 过滤 `OBJECT='REFERENCEFILE'` 更高效。

---

### **📌 方案 2：优化 `CELL_INFO_HISTORY` 子查询**

#### 问题：

```
SELECT REASON_DETAIL, REASON_CODE,   
COUNT (1) RANK_NUM FROM CELL_INFO_HISTORY   
WHERE CREATE_DATE BETWEEN ADD_MONTHS(SYSDATE, -1)   
AND SYSDATE GROUP BY REASON_DETAIL, REASON_CODE
```

这个查询 **扫描了 20M 行数据**，优化方法如下：

#### **优化方案**

**💡 使用 `MATERIALIZED VIEW` 提前计算**

```
CREATE MATERIALIZED VIEW MV_CELL_INFO_HISTORY BUILD IMMEDIATE 
```

🔹 **优势**：

* 让 `T.RANK_NUM` **直接从 `MV_CELL_INFO_HISTORY` 读取**，减少 20M 行扫描。

---

### **📌 方案 3：重写 SQL 语句**

**优化后的 SQL：**

* ```
    SELECT COUNT(*) AS COUNT FROM (    SELECT AA, KEY_1_VALUE, INSTANCE_ID, DEFECT_IPQC_CATEGORY    
        FROM (        SELECT            NVL(T.RANK_NUM, 0) NUM,             DATA_4_VALUE ||'/'|| DATA_1_VALUE AA,   
               KEY_1_VALUE,             DATA_1_VALUE INSTANCE_ID,             DATA_4_VALUE DEFECT_IPQC_CATEGORY,   
                N.NAMED_SPACE         FROM REFERENCE_FILE_DETAIL R        
       LEFTJOIN NAMED_OBJECT N             
      ON R.REFERENCE_FILE_RRN = N.INSTANCE_RRN              
     AND N.OBJECT ='REFERENCEFILE'        LEFTJOIN MV_CELL_INFO_HISTORY T                
    ON R.DATA_1_VALUE = T.REASON_DETAIL         WHERE N.NAMED_SPACE ='MYCIM2'    ));
  ```

🔹 **优化点：**

1. **使用 `MATERIALIZED VIEW` 代替原始 `GROUP BY` 计算**

   ，减少查询开销。
2. **调整 `JOIN` 条件**

   ，提前过滤 `N.OBJECT='REFERENCEFILE'`，提高 `INDEX INSTANCE_ID` 的效率。

---

## **✅ 3. 预期优化效果**

| **优化项** | **优化前** | **优化后** | **预期改进** |
| --- | --- | --- | --- |
| **查询时间** | **17 秒** | **< 5 秒** | **减少大表扫描** |
| **Buffer Gets** | **14M** | **< 5M** | **减少 I/O 负载** |
| **CPU 消耗** | **75% 在 `CELL_INFO_HISTORY`** | **< 30%** | **减少计算负担** |
| **索引使用** | **大量全表扫描** | **高效 `INDEX RANGE SCAN`** | **提升查询效率** |

---

### **🎯 结论**

1️⃣ **创建复合索引 `IDX_CELL_INFO_HISTORY (REASON_DETAIL, REASON_CODE, CREATE_DATE)`**，加速 `GROUP BY` 查询。

2️⃣ **使用 `MATERIALIZED VIEW` (`MV_CELL_INFO_HISTORY`) 预计算**，避免重复扫描 20M 行数据。

3️⃣ **调整 SQL 结构，减少 `LEFT JOIN` 计算量，提高索引使用率**。

🚀 **预期 SQL 执行时间降低到 5 秒以内，显著提升数据库性能！**

可以看到这个优化建议，已经非常清晰，而且给预估出了优化后的执行时间<5秒，我根据chatgpt的建议优化后，执行时间为4.2秒；

# 总结

我又尝试了多个大sql给chatgpt，它基本上都是按这三种模式来给我提出优化建议

1.根据具体的sql建议创建联合索引

2.使用MV物化耗时较久的子查询模块，这个建议有点耍流氓，你说有用吗？确实有用，但是在实际操作中可操作性较小，除非该子查询会被大量使用；

3.代码重构

而且chatgpt给出的优化后的时间和io的估算也是比较准确的，这点不得不说还是蛮强大的。如果想进一步调优，还可以使用ash，sqlhc等脚本跑出更详细的sql执行信息，丢给AI相信给出的优化信息将会更精准。AI的sql优化能力，取决于你给他多少数据，给的数据越多，回答的越精准。我认为目前chatgpt和deepseek的sql优化能力可以应付80%的通用场景，这可以大大的弥补DBA和开发sql优化层面的不足。

可以想象一个可行的自动化优化方案

优化类

[Oracle 11G还有新BUG？ORACLE 表空间迷案！](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488716&idx=1&sn=1f97da312cb3852bcec034e1e62ce3c1&scene=21#wechat_redirect)

[Oracle12C登录卡顿问题](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488702&idx=1&sn=8e34a5008cedc1e9c4a2a1f35da5f377&scene=21#wechat_redirect)

[一文了解Oracle数据库如何连接（2）](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488692&idx=1&sn=30cf3db16ea44d3cfa0c32ea2b00e92e&scene=21#wechat_redirect)

[一文了解Oracle数据库如何连接（1）](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488684&idx=1&sn=b03a2ef35499c3457ab7f2c22c0b2489&scene=21#wechat_redirect)

[Oracle RAC最佳实践-优化私网连接](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488670&idx=1&sn=ade5aa1afcaee92acdaf46d7b1ba16e8&scene=21#wechat_redirect)

[Oracle最佳实践-优化硬解析](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488661&idx=1&sn=185aa645eb94bec111c4cd357ee2950c&scene=21#wechat_redirect)

[好好的数据库怎么突然跑不动了？](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247483803&idx=1&sn=ddd84493a9cc8d99868e634230f817a0&chksm=c14dc795f63a4e83319877e671531555081527dc21d73f3a90b098830c460d9580ddde64847c&scene=21#wechat_redirect)  
[如何优化一个看似正常的数据库](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247483687&idx=1&sn=cf1261557add6271ba916f5fa18c2c3f&chksm=c14dc729f63a4e3f1d728508d13308e0387190cf19f16855bb057360c9097b090453a0c6bf7c&scene=21#wechat_redirect)  
[查询rownum伪列引起的sql性能问题分析](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487720&idx=1&sn=487e318ae6e6de538da879b69511294c&chksm=c14dd6e6f63a5ff0d2563f8af002a858c385960352de4ab2531d5e5e2c43b89b318fcc41a6a0&scene=21#wechat_redirect)  
[一个很小的系统为什么负载那么高？](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488363&idx=1&sn=1db37c6783503832f3df1ba209890f44&chksm=c14dd565f63a5c735a8a40da6f516b82ecb60c6956eac50de527a8235f471e57012aadf024e1&scene=21#wechat_redirect)

脚本类

[如何一键完成20个Oracle实例运维脚本部署](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488477&idx=1&sn=c39599e1caf2ff45c370bf2ca332bc7f&chksm=c14dd5d3f63a5cc5754840d1d6677e669c08afdab266857e6dcf124a12e276d3c490d2899507&scene=21#wechat_redirect)  
[分享一个比较实用的MySQL8一键巡检脚本](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488172&idx=1&sn=5df2433ae73915cb42af92d5ccb561f7&chksm=c14dd4a2f63a5db47854f3686b64c2155f9ac14ffaac5a95339f9768b99da565a7c6ea56117b&scene=21#wechat_redirect)  
[Oracle自动处理表空间不足脚本](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488462&idx=1&sn=423da0a85da6f96b99793b5b444fa509&chksm=c14dd5c0f63a5cd68bded448292a65df8a469bdc2d8b64118d9809019bf50024174160e9411a&scene=21#wechat_redirect)  
[分享两个ADG监控脚本](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488314&idx=1&sn=bced4fe9c82d3bff66e54f7579afb0cd&chksm=c14dd534f63a5c22ca6681eb29c048f4aa99ba6905b7f3bead4f950e775036b47be454c4dec4&scene=21#wechat_redirect)  
[Oracle 19C一键巡检脚本分享](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488169&idx=1&sn=5274a406fb67deccd52e98f7a2fd8d78&chksm=c14dd4a7f63a5db1c77d52da8eaf1b3b39f66cf9396b4317f17e4100dd0f272b2b0d69a7c7bd&scene=21#wechat_redirect)  
[oracle常用监控脚本（纯干货，没有EMCC,ZABBIX也不怕）](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247483847&idx=1&sn=2bbad17c1fd2d49dbfd38cbb8b6be454&chksm=c14dc7c9f63a4edf66298447a95623e96df4316f8ab2f776904ffb1a23f0a4243bddd727dfab&scene=21#wechat_redirect)

案例类

[大量的virtual circuit status等待造成系统卡顿](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488148&idx=1&sn=ba7c6040a04ce363ecb4f128f7249421&chksm=c14dd49af63a5d8cc472d6871c347034e2cbf7d63ef30c7be044f053d47ba8edc490aee8ef3a&scene=21#wechat_redirect)  
[大量enq: SQ - contention 等待导致系统卡顿](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488116&idx=1&sn=a368e2796091b55c1076147a3c571b62&chksm=c14dd47af63a5d6c5a4692f32789f7ba96aa7f2ef64d6305eb2747b94da62313f0d7679829b3&scene=21#wechat_redirect)  
[如何优化一个看似正常的数据库](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247483687&idx=1&sn=cf1261557add6271ba916f5fa18c2c3f&chksm=c14dc729f63a4e3f1d728508d13308e0387190cf19f16855bb057360c9097b090453a0c6bf7c&scene=21#wechat_redirect)  
[小白也能学会的Oracle优化教程-主打零基础](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487720&idx=1&sn=487e318ae6e6de538da879b69511294c&chksm=c14dd6e6f63a5ff0d2563f8af002a858c385960352de4ab2531d5e5e2c43b89b318fcc41a6a0&scene=21#wechat_redirect)

安装配置类

[Solaris安装Oracle RAC配置手册](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487527&idx=1&sn=f5ef09af003bff01c4064b7cccb20c2b&chksm=c14dd629f63a5f3f8c738cfa9272d15c9ab423d34549ca26ec0c663729ee5ddbdc2dc45d37d7&scene=21#wechat_redirect)  
[最好的数据库监控平台-ORACLE EMCC13.5安装配置使用文档](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487409&idx=1&sn=eeb0e857791fc706e5cb68e21337f4a6&chksm=c14dc9bff63a40a9484311a162e6370897acae812cb449af1b3b8a52c177ddab499daaffed3f&scene=21#wechat_redirect)  
[NBU备份oracle详细配置文档(含常见报错处理方法)](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247485099&idx=1&sn=30dbd8f03665f7f0ac500f8d5290042b&chksm=c14dc0a5f63a49b3d625be68eec40647df75e72897c5540f6b27880555e053f07939a630b367&scene=21#wechat_redirect)  
[Oracle rac如何替换OCR和VOTE磁盘组](http://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487752&idx=1&sn=3dc27af4e8e2d5219534f590f8981c11&chksm=c14dd706f63a5e100a24c1b9f20628f9680e851fb3523639d3a3e8265c9891b3f7bbc09f3843&scene=21#wechat_redirect)

补丁类

[Oracle RAC打补丁到19.26](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488847&idx=1&sn=e331339ad32f64da4169df89ab2e6963&scene=21#wechat_redirect)

[Oracle19.25发布，如何打补丁到19.25](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488407&idx=1&sn=d90dc8a83b2c989e88f81f154d3d1083&scene=21#wechat_redirect)

[Oracle RAC 19c RU 打补丁踩到的那些坑](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247488409&idx=1&sn=3ab2e5a6f3c358f4e82629635f00a065&scene=21#wechat_redirect)

[Oracle19.24发布，打补丁到19.24](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487946&idx=1&sn=3671af9179b4e56a0a9032cf39016ad3&scene=21#wechat_redirect)

[19C打完补丁后PDB状态受限Warning: PDB altered with errors](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247487727&idx=1&sn=8c511c6ea43474309bd837be5e60fbff&scene=21#wechat_redirect)

[oracle 19c 打补丁教程](https://mp.weixin.qq.com/s?__biz=MzkxNjY1Njk5Mw==&mid=2247483819&idx=1&sn=a03944ac5c06b8a5633f1cc0227c37ae&scene=21#wechat_redirect)
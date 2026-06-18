---
title: 面试加分秘籍：校招数据倾斜场景下的SQL优化方案
author: 涤生大数据
date:
url: http://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491889&idx=1&sn=74f94f4a5305bfbca70b4e90b3293900&chksm=ce329616db1763abd5694f01331763d88afecbb07ea276bb2fdc91314d90253b3584d60fc92b&mpshare=1&scene=24&srcid=0604Q5712JGCllvrwDEfnnes&sharer_shareinfo=14685b8b1fb602a501f785a41d44777c&sharer_shareinfo_first=14685b8b1fb602a501f785a41d44777c#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL执行前置优化|SQL执行前置优化]]


校招面试经常会问大家有没有过调优的经验，相信大家的回答基本都是往数据倾斜和小文件问题这两方面回答，对于数据倾斜相信大部分同学对热key打散或null值引发的倾斜已经非常熟悉，但这些内容面试官也是听腻了，希望大家在面试时候讲一些高大尚的案例，在描述的时候一定要有背景，有解决方案，最后结果，毕竟数据倾斜不会无故产生，一定是有业务背景的，这里给大家分享一种数据倾斜优化案例。

# 1.Uid和oaid之间的转化

在用增的拉新拉回业务中，经常会用到oaid来识别具体的设备是不是公司用户，所以我们需要将uid→oaid，需求目的：找到当日拉新的uid对应的oaid映射关系

代码如下：

* 1.从id mapping表中找出uid→oaid的映射关系;
* 2.根据最后一次活跃时间对uid→oaid映射关系去重;
* 3.将算法提供的uid人群圈选出对应的oaid。

原来的sql

```
select
  t1.user_id,
  oaid_md5
from
  (
    select
      L.uid user_id,
      md5(L.oaid) oaid_md5
    from
      (
        select
          distinct uid,
          oaid
        from
          (
            select
              uid,
              oaid,
              row_number() over (
                partition by oaid
                order by
                  cast(last_active_timestamp as bigint) desc
              ) as rn
            from
              idmapping as G
            where
              G.p_date = '20250324'
              and G.left_type = 'USER_ID'
              and G.right_type = 'OAID'
          ) tt
        where
          tt.rn = 1
      ) as L
      left join (
        select
          id
        from
          zuobishebei -- 作弊设备
        where
          p_date = '{{ds_nodash}}'
          and supplier = 'cheat'
      ) as P on (md5(L.oaid) = P.id)
    where
      P.id is null
  ) t2
  join (
    SELECT
      user_id
    FROM
      list_ground_truth
    WHERE
      p_date = '20250324'
  ) t1 on t1.user_id = t2.user_id
```

粗略一看，符合正常计算流程和顺序，但这段sql出现了明显的数据倾斜。

经过排查代码中有两块可能引起倾斜，一个是join，一个row number，先查询一下uid→oaid映射情况，发现部分的uid映射过10亿多的oaid，导致在去重的时候发生了数据倾斜。

**解决方案**

* 1.使用过滤条件和分组操作减少数据量；
* 2.通过调整连接顺序和提前应用过滤条件，减少了中间数据量；
* 3.如果倾斜仍然存在，考虑对倾斜字段进行分区或使用 broadcast join 来进一步优化。

优化后：

```
SELECT
  user_id,
  md5(paid) AS oaid
FROM
  (
    SELECT
      user_id,
      paid,
      ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY
          CAST(last_active_timestamp AS BIGINT) DESC
      ) AS rn
    FROM
      (
        SELECT
          t1.user_id,
          t2.paid,
          t2.last_active_timestamp
        FROM
          (
            SELECT
              user_id
            FROM
              list_ground_truth
            WHERE
              p_date = '20250324'
          ) t1
          JOIN (
            SELECT
              uid,
              oaid,
              G.last_active_timestamp
            FROM
              idmappingG
            WHERE
              G.p_date = '20250324'
              AND G.left_type = 'USER_ID'
              AND G.right_type = ‘ OAID ’
            GROUP BY
              G.left_value,
              G.right_value,
              G.last_active_timestamp
          ) t2 ON t1.user_id = t2.uid
      ) t3
  ) t1
WHERE
  rn = 1
```

原始脚本和优化后的脚本在逻辑上保持一致，但重点在于先jion较小的表（idmapping和 list\_ground\_truth），在进行row number，这样可以在join时先走map join同时减少row number执行的数据量。

往期推荐

[校招面试全攻略：揭秘校招面试四步走](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491556&idx=1&sn=f87e59fe5f5a7869db40361213998ee6&scene=21#wechat_redirect)

[2024届一线互联网大厂校招算法题侧重点：从手撕代码到思维能力考察](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491254&idx=1&sn=16e92ce8676273766691b74a11a3a2f0&scene=21#wechat_redirect)

[24年校招圆满落幕，25年秋招扬帆起航！学长学姐的求职攻略不容错过！](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491183&idx=1&sn=2c64b73ea8fd5c8c140806c2c8741b30&scene=21#wechat_redirect)

[双非本中大厂上岸心得：如何在大数据校招中脱颖而出？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490240&idx=1&sn=08024089900792754cef7ac0d390e6b8&scene=21#wechat_redirect)

[狠人，校招3月份突击拿下一线大厂总包40w+，最后拿到多个校招offer](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490150&idx=1&sn=5634437ef8b796b432f0b32357028028&scene=21#wechat_redirect)

[2025校招开奖啦！base 很惊喜，行情变好了么](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490025&idx=1&sn=83c108cbbf94b94d174e463b97cefd30&scene=21#wechat_redirect)

[金九银十大数据社招与校招就业行情如何？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489986&idx=1&sn=4b3ee9e8678245a060ad73484e16f487&scene=21#wechat_redirect)

[大数据秋招的第一个offer，校招实战策略分享](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489833&idx=1&sn=79dbd46c053afd4ed49c19b110f06e58&scene=21#wechat_redirect)

[字节阿里等大厂校招高频面试:数据清洗过程](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489214&idx=1&sn=effe3c1622658d472ce8480a8cbb1620&scene=21#wechat_redirect)

[字节美团百度等大数据校招实习都面了啥？校招实习大总结](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247489149&idx=1&sn=23d7862cdce0b37f78831bdb959186c6&scene=21#wechat_redirect)

[26届秋招收割offer指南](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491782&idx=1&sn=55e1564f7c63b95b50fd22cf2f28d4d2&scene=21#wechat_redirect)

[大数据八股还能这样背？带你解锁八股背诵的独家秘籍”](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491196&idx=1&sn=faf297077e63ec19c873e97fda19cccf&scene=21#wechat_redirect)

[春招倒计时，DeepSeek和涤生助你一月内斩获数据开发大厂Offer](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491139&idx=1&sn=00b79a151f0e813f2231247579a7c769&scene=21#wechat_redirect)

[25春招突击/26暑期实习的同学必看！最后一个月该如何冲刺？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247491117&idx=1&sn=d61ae326de6d46e0cd01adac1ee4acb3&scene=21#wechat_redirect)

[25年暑期实习快来了，如何备战暑期实习？](https://mp.weixin.qq.com/s?__biz=Mzg4MTU5OTU0OQ==&mid=2247490398&idx=1&sn=5ed540f7878b5ab93579a7c4b79d7352&scene=21#wechat_redirect)
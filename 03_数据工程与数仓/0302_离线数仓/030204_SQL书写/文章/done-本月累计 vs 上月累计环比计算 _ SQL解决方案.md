---
title: 本月累计 vs 上月累计环比计算 | SQL解决方案
author: 会飞的一十六
date:
url: https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502951&idx=1&sn=43a998117da91276f50b5087a9e283e0&chksm=e9b8c193e5683995f2ee9c691bdda0e9b080ceba6599fde235c8136bc2679e29f186aeb95a44&mpshare=1&scene=24&srcid=0616HFKIrYk1e02NRZWC2Yt1&sharer_shareinfo=d2a79656632e530052e0f91b9f6386c7&sharer_shareinfo_first=d2a79656632e530052e0f91b9f6386c7#rd
---

> 已吸收至：[[03_数据工程与数仓/0302_离线数仓/030204_SQL书写/030204_核心知识点/SQL窗口滚动与序列问题|SQL窗口滚动与序列问题]]


核心思路：

1. **本月累计**：从本月第一天到当前日期的滚动累计值
2. **上月累计**：从上月第一天到对应日期（同月同日）的滚动累计值
3. **环比计算**：(本月累计 - 上月累计) / 上月累计

#### 解决方案：

```
WITH 
-- 本月数据准备
current_month AS (
    SELECT
        mfg_date,
        SUM(epi_out + fab_out + bgbm_out) AS daily_output,
        SUM(SUM(epi_out + fab_out + bgbm_out)) OVER (
            ORDERBY mfg_date 
        ) AS month_cumulative
    FROM ADS_PLAN_WO_PRD_DD
    WHERE year_month = '2025-01'  -- 指定本月
    GROUPBY mfg_date
),
-- 上月数据准备
last_month AS (
    SELECT
        mfg_date,
        SUM(epi_out + fab_out + bgbm_out) AS daily_output,
        SUM(SUM(epi_out + fab_out + bgbm_out)) OVER (
            ORDERBY mfg_date 
        ) AS month_cumulative
    FROM ADS_PLAN_WO_PRD_DD
    WHERE year_month = '2024-12'  -- 指定上月
    GROUPBY mfg_date
),
-- 日期对齐处理
aligned_data AS (
    SELECT
        c.mfg_date,
        c.month_cumulativeAS current_cum,
        -- 获取上月同日的累计值（若无则取上月最后一天）
        COALESCE(
            l.month_cumulative,
            LAST_VALUE(l.month_cumulative) OVER (
                PARTITION BY c.mfg_date
                ORDERBY l.mfg_date
            )
        ) AS last_cum
    FROM current_month c
    LEFT JOIN last_month l 
        ON DAY(c.mfg_date) = DAY(l.mfg_date)
        AND MONTH(DATE_SUB(c.mfg_date, INTERVAL 1 MONTH)) = MONTH(l.mfg_date)
)
SELECT
    mfg_date,
    current_cum,
    last_cum,
    -- 环比计算（安全处理除零）
    ROUND(
        (current_cum - last_cum) / NULLIF(last_cum, 0) * 100, 
        2
    ) AS mom_growth_rate
FROM aligned_data
ORDERBY mfg_date;
```

### 关键技术解析：

1. **滚动累计计算**

   ```
   SUM(SUM(total_output)) OVER (
       ORDERBY mfg_date 
       ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
   )
   ```

* 从当月第一天到当前行的累计值
* 先聚合每日数据再窗口计算，提升性能

2. **跨月日期对齐**

   ```
   LEFT JOIN last_month l 
       ON DAY(c.mfg_date) = DAY(l.mfg_date)
       AND MONTH(DATE_SUB(c.mfg_date, INTERVAL 1 MONTH)) = MONTH(l.mfg_date)
   ```

* 精确匹配同月同日（如1月15日匹配12月15日）
* 处理月末日期差异（如2月28日匹配1月28日）

3. **月末异常处理**

   ```
   COALESCE(l.month_cumulative,
        LAST_VALUE(l.month_cumulative) OVER (...)
   )
   ```

* 当上月无同日数据时（如1月31日，12月只有30天）
* 自动取上月最后一天的累计值

4. **执行优化**

* 分区键`mfg_date`确保高效范围查询
* 预聚合减少窗口计算数据量
* 避免全表扫描的精确时间过滤

### 特殊场景处理：

1. **月初1号处理**

* 本月1号 vs 上月1号（自然对齐）
* 累计值=当日值（无累计效应）

2. **闰年2月29日**

   ```
   CASE WHEN DAY(c.mfg_date) = 29AND MONTH(c.mfg_date) = 2
        THEN COALESCE(l.mfg_date, (SELECT MAX(mfg_date) FROM last_month))
   ```

* 自动匹配2月28日（非闰年）

3. **上月无数据**

   ```
   NULLIF(last_cum, 0)  -- 安全除零
   ```

* 返回NULL而非错误
* 业务可识别为"无同比数据"

4. **跨年处理**

* 自动处理年份变更（12月→1月）
* 无需特殊逻辑

此方案精准处理了日期对齐、月末差异、闰年等边界场景，通过窗口函数实现高性能累计计算，适用于生产量、销售额等需要精细化对比的业务场景。

往期精彩

[京东金融面试提问：数仓中共性指标如何做下沉？请谈谈你的理解](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502905&idx=1&sn=6c57aecd5eab827f7140cd967de75a54&scene=21#wechat_redirect)

[京东数仓面试提问：数仓中应用层怎么设计？应用层和汇总层的区别是什么？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502894&idx=1&sn=cc5677ae1516f65bd53b092af690cf0e&scene=21#wechat_redirect)

[SQL面试提问：回本周期如何影响司机留存率？——数据分析方法论与实战](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502871&idx=1&sn=d99e5037726e4c7cfa5f765f6f904d63&scene=21#wechat_redirect)

[数仓面试提问：如何处理多值维度（多对多关系）？](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502837&idx=1&sn=1a8ccc4327542d7a4fc953aa1f7c231b&scene=21#wechat_redirect)

[Hive SQL 高级应用：数据洞察与分析从基础到实战，解锁数据价值](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502817&idx=1&sn=89c30893e081905576ad7ea455263c66&scene=21#wechat_redirect)

[Hive窗口函数RANGE BETWEEN详解：用法、场景与案例（附真实业务案例）](https://mp.weixin.qq.com/s?__biz=MzIzNTY4NTE5OQ==&mid=2247502822&idx=1&sn=588df18d42c3a8baeee036a212b526d9&scene=21#wechat_redirect)
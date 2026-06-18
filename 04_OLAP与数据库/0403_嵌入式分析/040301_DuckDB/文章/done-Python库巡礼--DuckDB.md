> 已吸收至：[[04_OLAP与数据库/0403_嵌入式分析/040301_DuckDB/040301_核心知识点/DuckDB扩展机制与生态边界|DuckDB扩展机制与生态边界]]
---
title: Python库巡礼--DuckDB
author: 费马的算法
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484689&idx=1&sn=7d764119d7b00d27232183ee0bf47467&chksm=c36c33776a7aa0c35a4fdd57ad44547929fa9202d03a56d878bc37fd9a617076605c9371d94b&mpshare=1&scene=24&srcid=0304zkV0jL2lRhSWq1c5igAd&sharer_shareinfo=d7216f8b87dcfa9fb7514b2f78b7eca8&sharer_shareinfo_first=d7216f8b87dcfa9fb7514b2f78b7eca8#rd
---

**Python的第三方库生态非常强大，涉及数据分析，机器学习，Web开发，可视化，自然语言，图像处理，自动化办公等。通过pip等包管理工具快速安装和升级，大幅提升了开发效率，并推动了跨学科应用的普及！**

**今天我们看下DuckDB，一个**开源、嵌入式、列式存储**的关系型数据库管理系统（RDBMS），专为**在线分析处理（OLAP）** 设计。与传统数据库不同，它：**

* ✅ 无需独立服务器，直接在Python进程中运行
* ✅ 零外部依赖，单文件部署（`pip install duckdb` 即可）
* ✅ 原生支持直接查询 CSV/Parquet/JSON 等文件格式
* ✅ 与 Pandas/Polars 无缝集成，可作为"分析引擎"替代部分DataFrame操作

DuckDB目前已成为数据分析领域增长最快的工具之一，被广泛用于AI/ML数据预处理，其发展历程如下：

| 时间 | 里程碑 |
| --- | --- |
| **2018年** | 由 **Mark Raasveldt** 和 **Hannes Mühleisen** 在荷兰数学与计算机科学中心（CWI）启动开发 |
| **2019年** | 首次公开发布 |
| **2021年7月** | 创始人成立商业公司 **DuckDB Labs**，提供企业级支持 |
| **2024年6月** | 发布首个官方"稳定版"（项目实际已迭代5年） |
| **2025年** | **1.4.0 LTS**（长期支持版）发布，新增： • 数据库加密 • `MERGE` 语句（UPSERT） • Iceberg 格式写入支持 • CLI 进度条与ETA |

为什么选择DuckDB？

| 场景 | 优势 |
| --- | --- |
| **大数据分析** | 比Pandas快10-100倍（列式存储+向量化执行） |
| **ETL管道** | 直接查询原始文件，避免中间转换步骤 |
| **Jupyter Notebook** | 用SQL替代复杂Pandas链式操作，代码更易读 |
| **资源受限环境** | 单文件部署，无服务器依赖，适合边缘计算 |
| **AI/ML预处理** | 高效特征工程，支持窗口函数、聚合等 |

下面是一些简单代码示例

#### 示例1：基础用法（内存数据库）

```
import duckdb
# 创建内存数据库连接con = duckdb.connect(database=':memory:')
# 直接执行SQL（自动创建表）con.execute("""    CREATE TABLE sales (        product VARCHAR,        amount DECIMAL(10,2),        date DATE    )""")
# 插入数据con.execute("INSERT INTO sales VALUES ('Laptop', 1200.50, '2025-01-15')")con.execute("INSERT INTO sales VALUES ('Mouse', 25.99, '2025-01-16')")
# 查询并转为Pandas DataFramedf = con.execute("""    SELECT product, amount     FROM sales     WHERE amount > 100""").fetchdf()
print(df)#      product   amount# 0     Laptop  1200.50
```

#### 示例2：与Pandas无缝集成

```
import pandas as pdimport duckdb
# 创建Pandas DataFramedf = pd.DataFrame({    'name': ['Alice', 'Bob', 'Charlie'],    'age': [25, 30, 35],    'salary': [50000, 60000, 75000]})
# 将DataFrame注册为DuckDB虚拟表con = duckdb.connect()con.register('employees', df)
# 用SQL分析Pandas数据high_earners = con.sql("""    SELECT name, salary     FROM employees     WHERE salary > 60000""").df()
print(high_earners)#      name  salary# 0  Charlie   75000
```

#### 示例3：复杂分析（窗口函数 + JSON）

```
import duckdb
# 查询JSON文件并使用窗口函数result = duckdb.sql("""    WITH user_events AS (        SELECT             user_id,            event_type,            event_time,            ROW_NUMBER() OVER (                PARTITION BY user_id                 ORDER BY event_time DESC            ) as rn        FROM read_json_auto('events.json')    )    SELECT user_id, event_type, event_time    FROM user_events    WHERE rn = 1  -- 每个用户的最新事件""").show()  # 直接在终端显示表格
```

熟悉MapReduce/SQL的小伙伴们看到下面这句

```
ROW_NUMBER()
```

应该会倍感亲切吧~

###

---

过往文章/合集

| 合集 | 文章 |
| --- | --- |
| **[Python编程小技巧](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4052929615994552341#wechat_redirect)** | [Python编程小技巧--用Python查询天气](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484317&idx=1&sn=f87bee0670626297cc9c3ebf8662b575&scene=21#wechat_redirect)  [Python编程小技巧--‌自制简易系统监控](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484597&idx=1&sn=b1eebb052d50a1e8fd41a6d9fba5d541&scene=21#wechat_redirect) |
| **[Python--数据/图像可视化](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4353741083969847301#wechat_redirect)** | [Python编程小技巧--雷达图](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484514&idx=1&sn=4eaaaef3d246f172d3986a987b4092fd&scene=21#wechat_redirect)  [Python编程小技巧--新年倒计时小工具2.0版](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484448&idx=1&sn=4c5c221adcd2b11eb17d85438a63c66c&scene=21#wechat_redirect)  [Python编程小技巧--桑基图（Sankey Diagram）](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484677&idx=1&sn=269957db3cf52dbc7f552865fdc03755&scene=21#wechat_redirect) |
| **[Python编程基础算法](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4096316346235879427#wechat_redirect)** | [Python编程基础算法--斐波那契数列的矩阵算法](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484507&idx=1&sn=320b5b8c4d0dfe5d35b7e63a753c9f97&scene=21#wechat_redirect)  [Python编程基础算法--LeetCode编辑距离问题](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484311&idx=1&sn=4e0b739bb8302923db8549cbcbaeec47&scene=21#wechat_redirect) |
| **[细数那些经典教材](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4281255286775480326#wechat_redirect)** | [细数那些经典教材--算法与数据结构](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484409&idx=1&sn=cf38c22e040e913afe497a59ed230927&scene=21#wechat_redirect)  [细数那些经典教材--编程竞赛](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484547&idx=1&sn=566e69a9c773533ac883b637ef585c17&scene=21#wechat_redirect) |
| **[Python库巡礼](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4333483913835708419#wechat_redirect)** | [Python库巡礼--Polars](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484604&idx=1&sn=f556a51ba224ecb48322b12841b381fb&scene=21#wechat_redirect)  [Python库巡礼--Dask](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484636&idx=1&sn=8c7da48dbc9a57a014514de5c7dc5074&scene=21#wechat_redirect) |
| **[Python与数学之美](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4359561894282657795#wechat_redirect)** | [Python与数学之美--黄金螺线](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484659&idx=1&sn=178fe72ba5b64f13a2140fd255fd54cb&scene=21#wechat_redirect)  [Python与数学之美--摆线/旋轮线（Cycloid）](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484688&idx=1&sn=24649b690c4d374066c4307ef26404f3&scene=21#wechat_redirect) |
| [Python与办公自动化](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk0MjQ5MDgyNw==&action=getalbum&album_id=4400137107994935303#wechat_redirect) | [Python与办公自动化--批量生成报告/通知](https://mp.weixin.qq.com/s?__biz=Mzk0MjQ5MDgyNw==&mid=2247484666&idx=1&sn=a672e540414ab2ba1b89f766c1d9df9c&scene=21#wechat_redirect) |

###
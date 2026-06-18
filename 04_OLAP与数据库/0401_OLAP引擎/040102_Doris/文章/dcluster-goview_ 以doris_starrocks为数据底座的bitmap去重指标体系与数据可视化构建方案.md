---
title: dcluster-goview: 以doris/starrocks为数据底座的bitmap去重指标体系与数据可视化构建方案
author: DClusterAI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NDMyMzEwNQ==&mid=2247484664&idx=1&sn=2ea417ae9dba58c5143d141c57b2b0fa&chksm=c2198e5c810933e4de847fc7a437d8277180ba10684fcab951fca81d677243c7d2e75b58593e&mpshare=1&scene=24&srcid=0911y8dVwi8g0cNG0HsTlgcV&sharer_shareinfo=d7a4b508bff93c0bbfdd55417f57a302&sharer_shareinfo_first=d7a4b508bff93c0bbfdd55417f57a302#rd
---

1. 技术架构概述  
  
dcluster-goview 是一个整合了指标中台与数据可视化能力的开源解决方案，其核心架构包含以下组件：

数据底座层‌：

采用 Apache Doris /StarRocks作为分析型数据库，支持基于 Bitmap 的高效精确去重计算‌。

指标中台层‌：

通过 dcluster 实现指标语义管理和统一口径服务‌。

可视化层‌：

基于集成优化后的dcluster-goview 构建交互式数据大屏和分析图表‌。

2. Bitmap 去重指标体系实现  
2.1 Doris 的 Bitmap 能力  
  
Apache Doris/StarRocks 实现了基于 Bitmap 的精确去重功能，相比传统的 HLL(HyperLogLog)算法具有以下优势：  
    支持任意数据类型的精确去重计算;  
    查询性能比传统方法提升数十倍，无需上卷的场景可达上百倍提升‌;  
    支持 PB 级数据分析‌。

2.2 指标体系建设流程  
     数据建模‌：设计基于 Bitmap 的指标表结构。  
     指标定义‌：通过 dcluster 的语义层统一管理指标口径和计算逻辑‌。  
     指标应用：在dcluster-goview中，数据探索分析,数据大屏，数据问答可视化层均可调用‌指标服务能力。

2.3 Bitmap构建指标体系去重优势

* ‌空间效率极高‌

每个元素仅需1bit表示存在性（0/1），相比传统数据结构（如集合）可节省数十倍存储空间。例如1亿用户去重仅需约12MB内存‌。

* ‌      计算性能优异‌

位运算（AND/OR）实现多条件组合筛选，如用户标签系统可快速计算“活跃且购买”的群体‌。

分布式场景下支持并行计算（StarRocks的BITMAP\_UNION\_COUNT），避免传统COUNT DISTINCT的多次数据shuffle‌。

* ‌       精确去重能力‌

通过位索引直接映射元素（如用户ID），确保100%准确率，适用于UV统计、游客去重等场景‌。

* dcluster-goview中多维去重可视化分析。

当统计UV时，维度组合达到6个以上时。如果还是采用预聚合的with cube方式时，尝尝数据量级特别大。一旦业务需要再加组合维度时，通常就需要重新构建数据模型表来降维重新组合计算。但如果通过Bitmap构建的数据模型表，在dcluster-goview中通过指标语义模型构建后，进行数据探索分析与chat问答时，则可以进行更多维度的筛选组合分析。重新增加维度时，数据量级也不会膨胀的太多，除非维度基数太高。

3. 指标体系与可视化构建案例  
以电商APP用户行为数据为例，构建Bitmap数据模型、指标语义、可视化数据探索分析。

* 数据模型构建

  原始用户行为埋点表、商品维表、用户维表

  ```
  CREATE TABLE IF NOT EXISTS user_product_exposure (    -- 事件标识    event_id BIGINT COMMENT '事件唯一ID',  
      -- 用户维度    user_id BIGINT COMMENT '用户ID',  
      -- 商品维度    product_id BIGINT COMMENT '商品ID',    product_sku_id BIGINT COMMENT '商品SKU ID',  
      -- 行为维度    exposure_time DATETIME COMMENT '曝光时间',    exposure_duration INT COMMENT '曝光时长(毫秒)',    exposure_position VARCHAR(100) COMMENT '曝光位置(首页推荐/搜索列表/详情页推荐等)',    exposure_source VARCHAR(50) COMMENT '曝光来源(推荐算法/搜索/活动等)',  
      -- 上下文维度    page_id VARCHAR(50) COMMENT '页面ID',    page_name VARCHAR(100) COMMENT '页面名称',    device_type VARCHAR(50) COMMENT '设备类型',    os_type VARCHAR(50) COMMENT '操作系统',    app_version VARCHAR(50) COMMENT 'APP版本',    network_type VARCHAR(50) COMMENT '网络类型',    ip_address VARCHAR(50) COMMENT 'IP地址',    province VARCHAR(50) COMMENT '省份',    city VARCHAR(50) COMMENT '城市',  
      -- 其他属性    is_click TINYINT COMMENT '是否点击(0否1是)',    trace_id VARCHAR(100) COMMENT '请求跟踪ID')DUPLICATE KEY(event_id, user_id, product_id, exposure_time)PARTITION BY RANGE(date(exposure_time)) (    PARTITION p202501 VALUES LESS THAN ('2025-02-01'),    PARTITION p202502 VALUES LESS THAN ('2025-03-01'),    PARTITION p202503 VALUES LESS THAN ('2025-04-01'))DISTRIBUTED BY HASH(user_id) BUCKETS 32PROPERTIES (    "replication_num" = "3",    "dynamic_partition.enable" = "true",    "dynamic_partition.time_unit" = "MONTH",    "dynamic_partition.start" = "-12",    "dynamic_partition.end" = "3",    "dynamic_partition.prefix" = "p",    "dynamic_partition.buckets" = "32");CREATE TABLE IF NOT EXISTS dim_user (    user_id BIGINT COMMENT '用户ID',    user_name VARCHAR(100) COMMENT '用户名',    nick_name VARCHAR(100) COMMENT '昵称',    gender TINYINT COMMENT '性别(0未知1男2女)',    age SMALLINT COMMENT '年龄',    birthday DATE COMMENT '生日',    register_time DATETIME COMMENT '注册时间',    register_source VARCHAR(50) COMMENT '注册来源',    vip_level TINYINT COMMENT '会员等级',    phone VARCHAR(20) COMMENT '手机号',    email VARCHAR(100) COMMENT '邮箱',    province VARCHAR(50) COMMENT '省份',    city VARCHAR(50) COMMENT '城市',    district VARCHAR(50) COMMENT '区县'    )UNIQUE KEY(user_id)DISTRIBUTED BY HASH(user_id) BUCKETS 16PROPERTIES (    "replication_num" = "3",    "enable_persistent_index" = "true");CREATE TABLE IF NOT EXISTS dim_product (    product_id BIGINT COMMENT '商品ID',    product_code VARCHAR(50) COMMENT '商品编码',    product_name VARCHAR(200) COMMENT '商品名称',    product_desc VARCHAR(500) COMMENT '商品描述',    category_id BIGINT COMMENT '类目ID',    category_name VARCHAR(100) COMMENT '类目名称',    brand_id BIGINT COMMENT '品牌ID',    brand_name VARCHAR(100) COMMENT '品牌名称',    supplier_id BIGINT COMMENT '供应商ID',    supplier_name VARCHAR(100) COMMENT '供应商名称',    price DECIMAL(18,2) COMMENT '商品价格',    cost_price DECIMAL(18,2) COMMENT '成本价',    market_price DECIMAL(18,2) COMMENT '市场价',    discount DECIMAL(5,2) COMMENT '折扣',    stock_quantity INT COMMENT '库存数量',    sales_volume INT COMMENT '销量',    comment_count INT COMMENT '评价数',    avg_score DECIMAL(3,1) COMMENT '平均评分',    main_image VARCHAR(255) COMMENT '主图URL',    sub_images VARCHAR(2000) COMMENT '子图URL(JSON数组)',    detail_html TEXT COMMENT '商品详情HTML',    status TINYINT COMMENT '状态(0下架1上架2预售)',    create_time DATETIME COMMENT '创建时间',    update_time DATETIME COMMENT '更新时间',    tags VARCHAR(500) COMMENT '商品标签(JSON格式)',    spec_json VARCHAR(2000) COMMENT '规格参数(JSON格式)',    service_json VARCHAR(1000) COMMENT '服务承诺(JSON格式)')UNIQUE KEY(product_id)DISTRIBUTED BY HASH(product_id) BUCKETS 16PROPERTIES (    "replication_num" = "3",    "enable_persistent_index" = "true");
  ```

基于埋点表、用户维表、商品维表，构建出包含多个维度的商品曝光用户的bitmap数据模型

```
CREATE TABLE IF NOT EXISTS product_exposure_bitmap (    -- 维度列    product_name BIGINT COMMENT '商品名称',    product_type BIGINT COMMENT '商品类型',    dt DATE COMMENT '日期',    exposure_position VARCHAR(100) COMMENT '曝光位置',    province VARCHAR(50) COMMENT '省份',    city VARCHAR(50) COMMENT '城市',    device_type VARCHAR(50) COMMENT '设备类型',    age_group VARCHAR(20) COMMENT '年龄段',    gender TINYINT COMMENT '性别',  
    -- 可以继续增加分析维度    -- 度量列    user_bitmap BITMAP BITMAP_UNION COMMENT '曝光用户Bitmap',    exposure_count BIGINT SUM COMMENT '曝光总次数')AGGREGATE KEY(product_id, dt, exposure_position, province, city, device_type, age_group, gender)PARTITION BY RANGE(dt) (    PARTITION p202501 VALUES LESS THAN ('2025-02-01'),    PARTITION p202502 VALUES LESS THAN ('2025-03-01'))DISTRIBUTED BY HASH(product_id) BUCKETS 32PROPERTIES (    "replication_num" = "3",    "dynamic_partition.enable" = "true",    "dynamic_partition.time_unit" = "MONTH",    "dynamic_partition.start" = "-12",    "dynamic_partition.end" = "3",    "dynamic_partition.prefix" = "p",    "dynamic_partition.buckets" = "32");
```

* dcluster-goview指标语义定义

  dcluster-goview开源项目源码和docker镜像安装包参考文末链接。

  下载dcluster-goview平台的指标定义模板，填写曝光人数相关指标定义并导入模板。

* 数据可视化分析

当数据模板和指标定义构建好之后，就能在dcluster-goview平台上进行数据探索分析和ChatBi问答、数据大屏创建。

4. 部署与资源

在线文档 :

https://www.yuque.com/shujurenxiaohui/sl2kgm/tqpmrlvrvcy3lspn

项目源码Gitee:

 https://gitee.com/zhenglv123456/dcluster

QQ社区群:   825017650
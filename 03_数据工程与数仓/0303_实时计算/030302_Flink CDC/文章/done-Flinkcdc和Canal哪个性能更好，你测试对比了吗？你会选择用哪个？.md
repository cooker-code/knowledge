---
title: Flinkcdc和Canal哪个性能更好，你测试对比了吗？你会选择用哪个？
author: 阿龙大数据
date:
url: http://mp.weixin.qq.com/s?__biz=MzA3OTExMjA0MA==&mid=2247485202&idx=1&sn=f9f17cf87bb977dcd4a5cae8a18bf2c0&chksm=9e9ec080e083ef31238d1ac1b0073ef064fba2ee92104020d8a6ca839718e090e27fec0df708&mpshare=1&scene=24&srcid=1226s4rdwLcUWGCGLXSDDYOc&sharer_shareinfo=2bd4efaa06fafa1bd33ed2a2db11f93e&sharer_shareinfo_first=2bd4efaa06fafa1bd33ed2a2db11f93e#rd
---

> 已吸收至：[[03_数据工程与数仓/0303_实时计算/030302_Flink CDC/030302_核心知识点/FlinkCDC概览与数据集成边界|FlinkCDC概览与数据集成边界]]


结论：

解析同步效率：Flinkcdc完胜 （遥遥领先！！！！！）

```
 Flinkcdc > Canal
```

选择用哪个？

```
推荐：Flinkcdc
```

一、测试对比：

1.1、版本对比：

`1、Flinkcdc`

```
<dependency>      <groupId>com.ververica</groupId>      <artifactId>flink-connector-mysql-cdc</artifactId>      <version>2.4.0</version></dependency>
```

2、canal：

```
canal.deployer-1.1.5.tar.gz
```

1.2、数据总量：

在MySQL数据库中导入数据量为 1.5亿数据（涉及200多个表），4个小时导入完成。Flinkcdc和Canal都什么时候同步完成了？

注：Flinkcdc 和Canal同时接入同一个库来数据解析并同步数据。

前提条件：

数据：1.5亿

导入时长：4个小时

1、Flinkcdc：

解析同步时长：6个小时

2、Canal：

解析同步时长：13个小时

结论：

解析同步效率：Flinkcdc 完胜

```
 Flinkcdc > Canal
```

1.3、如果现有Canal需升级为Flinkcdc？怎么弄了？(生产环境)

结论：

1、数据格式转换：

将Flinkcdc的数据格式转换成Canal数据格式。其余都不用变。

```
 FlinkCdcEvent  -> CanalEvent
```

2、数据解析后都是Json：

记得用Gson转换。fastjson有坑，不建议用.

记得用Gson转换。fastjson有坑，不建议用.

记得用Gson转换。fastjson有坑，不建议用.

1.4、Flinkcdc及Canal数据格式对比：

1、CanalEvent：

```
import java.util.List;import java.util.Map;public class CanalEvent {    private List<Map<String, String>> data;    private String database;    private long es;    private long id;    private boolean isDdl;    private Map<String, String> mysqlType;    private List<Map<String, String>> old;    private List<String> pkNames;    private String sql;    private Map<String, Integer> sqlType;    private String table;    private long ts;    private String type;        // Getters and setters    // toString    ...}
```

```
{  "data": [    {      "id": "G00002",      "name": "阿龙大数据",      "province_id": "32",      "province": "江苏省",      "city_id": "3201",      "city": "南京市",      "district_id": "320114",      "district": "雨花台区",      "address": "科创城23222333444",      "logo_url": "http://along/icon_112.png",      "slogan": "欢迎",      "credit_code": "2343243",      "master_name": "阿龙",      "master_idcard": "532524199911304246",      "power_group_id": "9996",      "opt_time": "2021-10-19 17:41:57",      "add_user_id": "132",      "add_user_name": "测试",      "add_time": "2020-07-27 13:59:34",      "email": "123456@qq.com",      "master_wechat": null,      "service_phone": "13232323232",      "max_shop_num": "5",      "pay_mode": null,      "business_license": "https://image/MXVSeb7Pv4r1f1346237770562140.jpg",      "idcard_front": "https://image/13uty9yyvrlvWb346237808202139.png",      "idcard_back": "https:///image/5uA7IblI6r1zoW346237778042125.png",      "cloud_shop_state": "0",      "expiration_time": null,      "version_id": null,      "company_type": "0",      "company_property": "1",      "main_sell": null,      "introduction": null,      "contact_name": null,      "contact_phone": null,      "certification_name": "阿龙大数据",      "is_test": null,      "login_account": null,      "delete_at": "0",      "self_invitation_code": "IN6501"    }  ],  "database": "test",  "es": 1669010586000,  "id": 8150,  "isDdl": false,  "mysqlType": {    "id": "varchar(32)",    "name": "varchar(64)",    "province_id": "int(6)",    "province": "varchar(32)",    "city_id": "int(6)",    "city": "varchar(32)",    "district_id": "int(6)",    "district": "varchar(64)",    "address": "varchar(128)",    "logo_url": "varchar(500)",    "slogan": "varchar(255)",    "credit_code": "varchar(18)",    "master_name": "varchar(16)",    "master_idcard": "varchar(18)",    "power_group_id": "bigint(20)",    "opt_time": "datetime",    "add_user_id": "varchar(32)",    "add_user_name": "varchar(32)",    "add_time": "datetime",    "email": "varchar(255)",    "master_wechat": "varchar(255)",    "service_phone": "varchar(32)",    "max_shop_num": "int(11)",    "pay_mode": "int(1)",    "business_license": "varchar(128)",    "idcard_front": "varchar(128)",    "idcard_back": "varchar(128)",    "cloud_shop_state": "int(1)",    "expiration_time": "datetime",    "version_id": "bigint(20)",    "company_type": "tinyint(2)",    "company_property": "int(2)",    "main_sell": "varchar(200)",    "introduction": "varchar(512)",    "contact_name": "varchar(32)",    "contact_phone": "varchar(11)",    "certification_name": "varchar(64)",    "is_test": "int(1)",    "login_account": "varchar(50)",    "delete_at": "bigint(14)",    "self_invitation_code": "char(6)"  },  "old": [    {      "address": "北京"    }  ],  "pkNames": [    "id"  ],  "sql": "",  "sqlType": {    "id": 12,    "name": 12,    "province_id": 4,    "province": 12,    "city_id": 4,    "city": 12,    "district_id": 4,    "district": 12,    "address": 12,    "logo_url": 12,    "slogan": 12,    "credit_code": 12,    "master_name": 12,    "master_idcard": 12,    "power_group_id": -5,    "opt_time": 93,    "add_user_id": 12,    "add_user_name": 12,    "add_time": 93,    "email": 12,    "master_wechat": 12,    "service_phone": 12,    "max_shop_num": 4,    "pay_mode": 4,    "business_license": 12,    "idcard_front": 12,    "idcard_back": 12,    "cloud_shop_state": 4,    "expiration_time": 93,    "version_id": -5,    "company_type": -6,    "company_property": 4,    "main_sell": 12,    "introduction": 12,    "contact_name": 12,    "contact_phone": 12,    "certification_name": 12,    "is_test": 4,    "login_account": 12,    "delete_at": -5,    "self_invitation_code": 1  },  "table": "company",  "ts": 1669010468134,  "type": "UPDATE"}
```

2、FlinkCdcEvent：

```
{  "before": {    "id": "PF1784570096901248",    "pay_order_no": null,    "out_no": "J1784570080435328",    "title": "充值办卡",    "from_user_id": "PG11111",    "from_account_id": "1286009802396288",    "user_id": "BO1707796995184000",    "account_id": "1707895210106496",    "amount": 13400,    "profit_state": 1,    "profit_time": 1686758315000,    "refund_state": 0,    "refund_time": null,    "add_time": 1686758315000,    "remark": "充值办卡",    "acct_circle": "PG11111",    "user_type": 92,    "from_user_type": 90,    "company_id": "PG11111",    "profit_mode": 1,    "type": 2,    "parent_id": null,    "oc_profit_id": "1784570096901248",    "keep_account_from_user_id": null,    "keep_account_from_bm_user_id": null,    "keep_account_user_id": null,    "keep_account_bm_user_id": null,    "biz_company_id": "PG11111"  },  "after": {    "id": "PF1784570096901248",    "pay_order_no": null,    "out_no": "J1784570080435328",    "title": "充值办卡",    "from_user_id": "PG11111",    "from_account_id": "1286009802396288",    "user_id": "BO1707796995184000",    "account_id": "1707895210106496",    "amount": 13400,    "profit_state": 1,    "profit_time": 1686758315000,    "refund_state": 0,    "refund_time": null,    "add_time": 1686758315000,    "remark": "充值办卡1",    "acct_circle": "PG11111",    "user_type": 92,    "from_user_type": 90,    "company_id": "PG11111",    "profit_mode": 1,    "type": 2,    "parent_id": null,    "oc_profit_id": "1784570096901248",    "keep_account_from_user_id": null,    "keep_account_from_bm_user_id": null,    "keep_account_user_id": null,    "keep_account_bm_user_id": null,    "biz_company_id": "PG11111"  },  "source": {    "version": "1.6.4.Final",    "connector": "mysql",    "name": "mysql_binlog_source",    "ts_ms": 1686734882000,    "snapshot": "false",    "db": "cloud_test",    "sequence": null,    "table": "acct_profit",    "server_id": 1,    "gtid": null,    "file": "mysql-bin.000514",    "pos": 650576218,    "row": 0,    "thread": null,    "query": null  },  "op": "u",  "ts_ms": 1686734882689,  "transaction": null}
```

记录每一份热爱,让美好永远陪伴。
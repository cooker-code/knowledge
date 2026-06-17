---
title: StarRocks的SQL指纹应用
author: osyun.cn
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5NjUyMzM2Ng==&mid=2247484189&idx=1&sn=c1e03cc81dc1adbb23348e0e06b683be&chksm=c07e8e14f7090702fe809ceb350f9ed9731af96b79653773739c43780e5e4dfb8020ff51b2c6&mpshare=1&scene=24&srcid=0308Icj5qbfEZsZxQ9MqFpVF&sharer_sharetime=1646735471340&sharer_shareid=4145ee29355a80b1b2b01921ded70e3a#rd
---

**StarRocks的SQL指纹应用**

                          --2022-03-01 春雷

# **1、前言**

    StarRocks 的2.1版本发布啦~这个版本上线了我们心心念念的SQL指纹。如下：

* 支持 SQL 指纹，针对慢查询中各类 SQL 语句计算出 SQL 指纹，方便您快速定位慢查询

# 

# **2、SQL指纹说****明与使用**

**2.1、说明**

# 

    StarRocks支持规范化慢查询（ 路径fe.audit.log 的slow\_query ）中 SQL 语句，并进行归类。然后针对各个类型的SQL语句，计算出其的MD5 哈希值，对应字段为 Digest。

日志举例：

2021-12-27 15:13:39,108 [slow\_query] |Client=172.26.xx.xxx:54956|User=root|Db=default\_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(\*) from test\_basic group by id\_bigint|Digest=51390da6b57461f571f0712d527320f4

        有了SQL指纹，我们就可以按照SQL指纹汇总，就可以看出一段时间内的慢SQL汇总情况了，比如：总次数、平均返回行数、平均扫描行数、平均执行时间等

**2.2、使用架构**

**2.3、平台设计**

* 天级别慢SQL

+ 方便查看前一天的慢SQL汇总情况，趋势图

* 实时慢SQL

+ 快速查看最近时间的慢SQL情况

* 指定时间慢SQL

+ 可以指定某段时间，汇总展示SQL情况，次数、平均执行时间等。

**2.3.1、天级别慢SQL**

**2.3.2、实时慢SQL**

**2.3.3、指定时间慢SQL汇总**

# **3、StarRocks慢SQL展示实现具体**

**3.1、实时慢SQL实现**

StarRocks实时慢SQL实现，请参考:

[DorisDB实时慢SQL实现](http://mp.weixin.qq.com/s?__biz=Mzg5NjUyMzM2Ng==&mid=2247483952&idx=1&sn=256c91bb0c3405abead5c1989082cc7c&chksm=c07e8f39f709062f3c0e051ab503b038044a243523aae694aa165e573122538fcb061b78987a&scene=21#wechat_redirect)

## **3.2、filebeat修改**

修改配置文件，并重启filebeat

filebeat.yml

processors:

- script:

lang: javascript

id: my\_filter

tag: enable

source: >

function process(event) {

var str = event.Get("message");

var slow\_time = str.substr(0, 19);

var query\_type = str.substr(25,10);

var detail\_query = str.substr(38);

var js\_arr = detail\_query.split("|");

var Client\_tmp = js\_arr[0];

var Client\_tmp2 = Client\_tmp.replace('Client=','');

var Client\_tmp3 = Client\_tmp2.replace('t=','');

var Client\_arr = Client\_tmp3.split(":");

var Client = Client\_arr[0]

var User\_tmp = js\_arr[1];

var User = User\_tmp.replace('User=default\_cluster:','');

var Db\_tmp = js\_arr[2];

var Db = Db\_tmp.replace('Db=default\_cluster:','');

var State\_tmp = js\_arr[3];

var State = State\_tmp.replace('State=','');

var Time\_tmp = js\_arr[4];

var Time = Time\_tmp.replace('Time=','');

var ScanBytes\_tmp = js\_arr[5];

var ScanBytes = ScanBytes\_tmp.replace('ScanBytes=','');

var ScanRows\_tmp = js\_arr[6];

var ScanRows = ScanRows\_tmp.replace('ScanRows=','');

var ReturnRows\_tmp = js\_arr[7];

var ReturnRows = ReturnRows\_tmp.replace('ReturnRows=','');

var StmtId\_tmp = js\_arr[8];

var StmtId = StmtId\_tmp.replace('StmtId=','');

var QueryId\_tmp = js\_arr[9];

var QueryId = QueryId\_tmp.replace('QueryId=','');

var IsQuery\_tmp = js\_arr[10];

var IsQuery = IsQuery\_tmp.replace('IsQuery=','');

var feIp\_tmp = js\_arr[11];

var feIp = feIp\_tmp.replace('feIp=','');

var Stmt\_tmp = js\_arr[12];

var Stmt = Stmt\_tmp.replace('Stmt=','');

var Stmt = Stmt.substring(0,65530)

var Digest\_tmp = js\_arr[13];

var Digest = Digest\_tmp.replace('Digest=','');

event.Put("query\_type",query\_type);

event.Put("slow\_time",slow\_time);

event.Put("Client",Client);

event.Put("User",User);

event.Put("Db",Db);

event.Put("State",State);

event.Put("Time",Time);

event.Put("ScanBytes",ScanBytes);

event.Put("ScanRows",ScanRows);

event.Put("ReturnRows",ReturnRows);

event.Put("StmtId",StmtId);

event.Put("QueryId",QueryId);

event.Put("IsQuery",IsQuery);

event.Put("feIp",feIp);

event.Put("Stmt",Stmt);

event.Put("Digest",Digest);

}

## **3.2、修改表结构**

alter table starrocks\_slow add column digest varchar(50) default null comment 'SQL指纹 Digest';

## **3.3、修改任务**

CREATE ROUTINE LOAD starrocks\_slow\_load ON starrocks\_slow  
columns (slow\_time,igid,db\_name,fe\_ip,query\_id,time,client,user,state,scan\_bytes,scan\_rows,return\_rows,stmt\_id,is\_query,stmt,query\_type,digest)  
PROPERTIES (  
"format"="json",  
"jsonpaths"="[\"$.slow\_time\",\"$.igid\",\"$.Db\",\"$.feIp\",\"$.QueryId\",\"$.Time\",\"$.Client\",\"$.User\",\"$.State\",\"$.ScanBytes\",\"$.ScanRows\",\"$.ReturnRows\",\"$.StmtId\",\"$.IsQuery\",\"$.Stmt\",\"$.query\_type\",\"$.Digest\"]",  
"desired\_concurrent\_number"="8",  
"max\_error\_number" = "9999999999",  
"max\_batch\_rows"="200000",  
"max\_batch\_size" = "209715200",  
"max\_batch\_interval" = "10",  
"strict\_mode" = "false"  
 )  
FROM KAFKA  
(  
"kafka\_broker\_list"= "10.1.1.1:9092,10.1.1.2:9092,10.1.1.3:9092,10.1.1.4:9092,10.1.1.5:9092",  
 "kafka\_topic" = "starrocks\_slow\_log",  
 "property.kafka\_default\_offsets" = "OFFSET\_END",  
 "property.client.id" = "xxx",  
 "property.group.id" = "starrocks\_slow\_load"  
);

## **3.4、查看结果**

模拟慢SQL，及查看结果
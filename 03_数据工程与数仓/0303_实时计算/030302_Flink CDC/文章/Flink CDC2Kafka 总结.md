---
title: Flink CDC2Kafka 总结
author: 伦少的博客
date: 董可伦董可伦
url: https://mp.weixin.qq.com/s?__biz=Mzg2NzA0ODg1Mg==&mid=2247485835&idx=1&sn=5054964152df57470050ea7c5add4b68&chksm=cf9e7d9475ce730c21eb687ebd74c9ba337a5c81c62af11325f0b2c4f8d6a6d96a63a168344a&mpshare=1&scene=24&srcid=0502bm3OZWS4fjrez7qjYieH&sharer_shareinfo=fac3ec9252af8655097f99c7d2d18fac&sharer_shareinfo_first=fac3ec9252af8655097f99c7d2d18fac#rd
---

## 前言

本文主要记录 CDC2Kafka 使用过程中的一些细节配置，以 mysql-cdc 为例。

## 背景需求

之前 CDC 的主要使用场景并没有写到 Kafka ，而是直接 Sink 到支持更新的目标端，比如 关系型数据库、Hudi、HBase，现在有了写到 Kafka 的需求，主要原因是有的项目的厂商不会使用数据湖表格式(Hudi、Iceberg、Delta Lake)，在使用 Hive 时不能支持ACID ，那么只能通过覆盖分区的方式实现更新删除，而覆盖更新对比小表来说可以通过定时全量抽取实现，对于大表每次都全量抽取效率太慢，所以最好可以通过增量变更日志+历史Hive表数据合并来实现，对于这种需求就得通过 CDC 格式来实现，CDC 格式有 debezium-json、canal-json、maxwell-json、ogg-json、changelog-json。 关系型数据库、Hudi、HBase等是不支持CDC格式的，常用的比较熟悉的组件就是Kafka，另外后面发现直接写 HDFS 文件也是可以的，本文先总结 CDC2Kafka+debezium-json 中的一些具体需求配置。

## debezium-json

格式数据示例：

```
{"before":{"id":3,"name":"hudi3","price":3.33,"ts":2000,"dt":"20230331"},"after":null,"op":"d"}  
{"before":null,"after":{"id":3,"name":"hudi4","price":3.33,"ts":2000,"dt":"20230331"},"op":"c"}  
  
{"before":null,"after":{"id":4,"name":"hudi4","price":3.33,"ts":2000,"dt":"20230331"},"op":"c"}  
  
{"before":{"id":4,"name":"hudi4","price":3.33,"ts":2000,"dt":"20230331"},"after":null,"op":"d"}
```

分别对应update、insert、delete ，其中 update 一次产生两条记录，先删除再插入。

## 需求

直接总结具体需求，关于 CDC 、Flink 写 Kafka 的细节等可以参考之前的文章:  
[Flink MySQL CDC 使用总结](https://mp.weixin.qq.com/s?__biz=Mzg2NzA0ODg1Mg==&mid=2247484485&idx=1&sn=457fa0328e1077a3e84eb22ebbe199ee&scene=21#wechat_redirect)  
[Flink 读写Kafka总结](https://mp.weixin.qq.com/s?__biz=Mzg2NzA0ODg1Mg==&mid=2247484540&idx=1&sn=d9688e2b5edd4a007267cd455a7d6780&scene=21#wechat_redirect)

### 添加时间字段

主要是根据时间字段排序，根据官方文档：  

  
Source 添加：

```
operation_ts TIMESTAMP_LTZ(3) METADATA FROM 'op_ts' VIRTUAL
```

Sink 添加：

```
operation_ts TIMESTAMP_LTZ(3)
```

字段类型也可以写成String , 两种类型对应的格式不太一样。  
TIMESTAMP\_LTZ(3) 对应 `1970-01-01 00:00:00Z` ，String 对应 `1970-01-01 08:00:00.000` , 完整示例：

```
{"before":null,"after":{"id":1,"name":"hudi1","price":1.1,"ts":1000,"dt":"20260417","operation_ts":"2026-04-17 02:21:27Z"},"op":"c"}
```

### 相同主键落在同分区

添加参数：

```
'key.format' = 'json',  
'key.fields' = 'id',
```

结果：

### Blob 图片字段支持

首先要支持 Blob 类型的字段、然后要求转 base64

#### 转 base64

参数：

```
'debezium.binary.handling.mode' = 'base64'
```

对应的字段类型为 String

#### 图片验证

##### 建表

```
CREATE TABLE `mysql_cdc_source2` (  
  `id` int(11) NOT NULL,  
  `name` text,  
  `price` doubleDEFAULTNULL,  
  `ts` int(11) DEFAULTNULL,  
  `dt` text,  
  `img` LONGBLOB,  
  `insert_time` timestamp(3) NULLDEFAULTCURRENT_TIMESTAMP(3) COMMENT '创建时间',  
  `update_time` timestamp(3) NULLDEFAULTCURRENT_TIMESTAMP(3) ONUPDATECURRENT_TIMESTAMP(3) COMMENT '更新时间',  
PRIMARY KEY (`id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

##### 插入图片

```
INSERT INTO cdc.mysql_cdc_source2  
(id, name, price, ts, dt, img, insert_time, update_time)  
VALUES(1, 'hudi1', 1.1, 1000, '202660420', LOAD_FILE('/var/lib/mysql-files/query3.png'), CURRENT_TIMESTAMP(3), CURRENT_TIMESTAMP(3));
```

这里要求在MySQL服务器上执行，并图片路径，通过下面SQL获取，其他路径上传的图片都为null

```
SHOW VARIABLES LIKE 'secure_file_priv';
```

##### DBeaver 里查看验证

是个猫眼的图片，可以成功显示：

##### Kafka 数据验证

可以看到已经成功转为 base64 , 完整数据：

```
Partition:0     Offset:0        {"before":null,"after":{"id":1,"name":"hudi1","price":1.1,"ts":1000,"dt":"202660420","img":"iVBORw0KGgoAAAANSUhEUgAAAEIAAABKCAYAAAAPB4KFAAAgAElEQVR4nG2c2a9k13Xef2vtfWq6dYe+PXeTzabMQaQESjQ1mJJsC7KseIgiQ3acwEAAP+bFSIL4yU8G8j8EyEsCJ7KDvDhRLAGeFE2wLVOkSFlWTFOcxJ5v375zTeecvVce9t6n6rZSQJFdt86w9xq/9a11Sg7/7D+Y2AkQwDygaDQER6uOam2MOGVxvA8WEXNghkkgSoVfv8hwvMnx7h2kPqaixSxiIhiCAOARwIjpPhigIAIoIgZmlJdZOtNMEVGQOp8jgGBm+TNIvgbdXwArdzYMQ02JAkqAaEyC4+vf+T59MZ5/32WGBmq4tCAxkIgQQfKiY6SZTgmTo3QRsZXNgGlFfzCmmR4j7Rw1y5vIqyqbM2O59pWFlz92h1n+G0mIIpx+GWbxISGQ17vyEgGJIIbkNwgRxcToe+OFDz/L/uE+t/eOWNQBNakwU7AiVesWpEQ0tNA2aYFlkwiGgu/hnCfMp0houg1btx7JG7dOO6fWW/RoD22s06+xeoidPn0ptBXhnDpbDBMjSgAJIEoUj4hxdlzx3Aef4R/efY9JFDRKFoJVYA4xBTRd1gwsgEUwSUZjEcFh4nG9ATE0xHqGlo2IYuV8yuJXXeIhzZsloVoSWNJgzFqM+Sqa3w9bSHYtkoIk7T7vp1xTMCwdYyB4xARP4NGrF9jc3ubN23fR6B3gwDxiDsmbsOKLlhYTRdNtLd3c1NHr92nmJxDbtMQsCERPmW0y05A3Z4icNnvJvp8sKmZBRYyYP3sEtyKQ7sL52GIDiuQ3UvaloJLONNAI4DEclfM8/8FnuXtwiFdf5aUkDUXSwiUaYoqJSxLNko4iRAxVwUtkujhBJeQzi/RXNGZZOA+bbv4oop3GftK4SQI8JTCSxdnSYkRY3tXkIcMRLEI0UDFUAtHSfTUaZzfWePrpJ/DOj/KJSfqoITGmtymtU5CIEkEkWRsBidBM9ohtU3QDxE6gZgLi8kaz2a9sUIRsviBabFBXtFve4ZSApNtlcaNsoTkLLUNtOhcxJHqMmGJFVhgIqkmgj1+9hNfemIDHWUyLwrAYkQhYi7iVoJYvkNw30s4WuBwLjaL1oiU5vYEulSRzSBuIXRgVSkC0fGwJtOWUdI7ZqlBdvnfMVpEtrGSv5G3JVQCzwNK1yoUjG8MBXgfrNFKBNSiKxRQYY4zLAIRbaqLYdEy5WkSIJTN09p7eaXOxiw2n7l8ivQAmOduW4JhjiKUAaRQhRIrlFAyS/r+AsooutRhmki2vCFVLRO0EImI4Iqq9EVGrdJksVjHJaTBF8JSDhdUkJUIHetJGZLkBpDNxSamGn3yVwJmtp8MYRVDSvTHtFFGMJeeBtNnurNhd107l2oJddOXfyzUECyh+iFYjTLN/rtykbDhJ0udInL6LBiZL1acFFmkvF3T6czYC0fzOtmuSM0m5Hz9xXjJx6Vyp03ontByX8laXWUkzsDol56WrCpgzvFR9dDDEFgWSRqJEIpJTaUltsuK2lj2hBJ5isivAaFUjJQB2OT4t10QQsW4D2Y6Xb1E67awI2mw1SJJT5sqtVuwgId2Q/7J0veXyDCHio3qkqogYUSLEkMUhOWYUERtihklMFywSN4CwosmlDIp5iiwRacHDS4NeUZUlJNgJbUW4DwsDyQhYQMx3G00RxfK1JAstLgHuipikfI/hTSqq0RZz7SOhxcSBGK6KSSNaFhfpApIkXxMzRJq8B82mW2JESBAXw1mO89mERZRYhEQkaoQoKA4x6TZjxCT4EvQKIFNyIFXEHMV+VyLYCrxPisLkJ2RRsIiY4jFF+2cwN4J2BuZSgHMlBpQYah2EzVUIIoZaMbslpMVyxhEjrmQCIwC+K7bUIioRo02YI3owBxKANlnhyv2Tl6xaVLqnaCnGytalc7/OZU4FYyHjgyRc83hRQ3wfP1hDFg9QgdYEUe0kKOayeVl3rqJEhCCCWIMSiZqATZQqR3tBnNK6mKtPxWISlCMHXmtxtJTy1MQRJWZM4FBTDJcVsqx8k421rPq7yCrcKgCPpIBsxfaQywlKxOGNQHAVbriOHYHEpB0EpPAKMW0iOJf0YIJm34vEFNNEaHVI9BtIfx0/WMf3B/iqB84lY4mGBcNCTWzmtIsJ9eIYXRyi7RyNIV3Xcp1Q3APLQEk7gaX6Jbur9HJ8tSzA7Eqd0ABiZ7HSBeUSmwQPEXMOHa7TugqNixUAFJMfSooLZqWWCIiEVIfogKbXpxqdYbB2Hh2eg2pMdBVRoBYjZj5ATNDsq2otfYv0Yo3MD2kmD6gPd5D6CA1NJodIruralQyVk2hBkprtXk4jxlOAjZJlum9Xjkvy8QhEE3x/TOyvE8McVi5ipEzRQZLsIq3vE3vb+PVLrG1ehN6ISEWrPaKsApYVIqdgFNOl1ckQ1tapRucZbD9Cc7zD4miHZnqEr2s8RpQWI+Q06ZLLWYbSKpTUJjl+dLGkWEJxn4dwXQF9qhEvpJxvvTFu/RLN7BAfF3nbySSTLwVUhChC48foxmV6Z67D4CyNVMl0RSgMVtEdSAqCK68Ci0okNKto1RHUo9tjhuMrhJN92oNb1LMHSJwj4iAuMUhCvxGiW4KjDo2SLTlSCKe0cTvlFsVKzAyf0h4E6ePWLtD6W0hok5S7CO1AIq0KNtikv/04uvEIrR9j4rOx5kyxQrLkXZNqlfI6BdTzf9Nio1REHNIboJsjBqN15oe3scN3sHrWYQHJ6bDEESwssXd25QL6inunjcdT912+Ij6ZPphUSP8svfEVYjNFwixF3swXRDcgjs8xOH8dG52j0SFQZZNfZhPJET5aCWaW44l1+7esPS1VYXG9wo6JEX2fqBtUZ/sw6DPbuw3TQypbYCYEFSxqSt/q8nUfpuxKoC1FW3HVpJyExgSsxVspiCKYH1Ode5yTxSE620HjHJFAcCN0fJHRxadoh9u06hJ0MkumSVvAZ7aNdEMVwywkkNSh5byQXFIbCtqCCY4SkBP5I+owKmTjUQa9DRY77xKP76RUnVGlCsn8C72XA3MqxUOOoSlNF1wjXVngszAcHjwQcC4SRGB0hv7VDzK98zpusYsQ0bUrDC+8jzBYJ2ii8zSmTVsOZGlzGSxJocUCFltiqJJmNORYk8GaZJ1lLiS5fy7ISrAWBTwyOsf4cp8ZEA5votaAQjASJ0Ip08tLOoFIaRnkwiGbzxIXoXghgyWrUedoY4UbnWfjEUc72SHEyGDzGsGv0XTub911ihYkFzeW65M2NCxmU06Oj9jdOWb/YJ/p7BAlsL455vyFC5w7d5HBYA1XGc5lrhNbyRCJGw0IYhHX32Jw8Slm0eDoPVxc0OYSXopvdrv0KcBiOWA+xKJ3IDMJypOzhggEa7K8HLG/iQzGOJQ6DjAVTBqwkJBlof8togXMxJpmseDu/V2+89Lf8r3XXuXdd95lOoEYjECLCHjn8N5x/uw5nnn6/Xz0Y8/x7LNPs7GxhboS2AxwmCmm4CwQUKx/lurCk9T1CTrdwZU642HOI5M9smIZZHJniSlCFlJEju+9Y0KDqSWCE0OjEjPXKBYxc1n9bQJThciNhQ5rCU3Dg/v3+fo3vsE3vv1tbt66Td3WhDZAyEWWSPJtUuGlonjvGPeFJ598gs9+9nO8+OKLbG6tgyghKmgPpEYtAo7WHC7UuOObzG//kF59QIiFJV9xDfNL6K0li8VOeUthCGqKHN972yBiokRzpYLAJBVZLgZiBkJi1vlVLBkgGovpMS9/92X+55e/wutvvkPdtik4xZqeBAYaGFSKmhBjoDWjNmUShAboo6g4Rmsb/PTzz/PFL36Bp59+KvVNTHEScnWwgh6bKXJwg/ru6/hmCtIA7ekYYQUDrcaOU+A6fy/I8b03zSB5ScwFj65qXjBtwEBj6YZFggViiOztPuBPv/pV/vzP/4L94wVNNCQsGDPj0ijyxIV1rqz32Bp41irPoFIWBvdmLe/sn/D2ziF3ThyL2Kc1h3fC9WtX+c3f/HV+5pOfpD9cR3OBhuReR0YufnHI9PYbuMP30DhPwpDCZRT3SGVVZwEFea7CbjHk+N4bhqS6XizzkiX/5wCjJdBY6gUEa2iahh+/+x5f+sP/zms/eIW2jlgDfRqurkWeuzDkiU3Y9jVjhWFV0a8EJ6lIa1yPue+x3wRev9/w8o+PeHMPJnGAacuF8+f54q9/kV/51V9mOBhROIkV8g5vLWGyT3PzVdxiB4mlm2aZ1wgEKTxHEkxH3YihufmsSG5T5y+WeK9Q8y1Ki0YhqhJyCzDGyK1bd/kv//lLvPZ3f0/bLhiFhqujimcur/HE2Yr1MGUUWyoD3x/h+h6TFpEWxehpoCczNgeRq496PnDpEf76vSnfeuMBe23Fzu4+X/qj/8GwP+Czn/sMvV6fVIZlDIMQqZDhFn7jMs3eIS7OU2rOyktxMmLR5T3lwFn+Tkg7NcX93u/+298vUjz96nhipAMwqdDZ3d3lD770R7zy6veop0eshRnPbA/42NU1Hl0L9J1w0ig3Tow3DgJvPKjZa5TW93G9Pq7qoaqoCOqEPoGxg+vnhpwZ97m7e8A8CItgvPXGm1y7do0rV692rp9eSbciUDlHO5tBO0GsJWaMko7VnNpXYgcFjJe6B3z6mHlIgM4aigFqihuZkJ3OZnz5j/8XL//td5kc7bLt5jx/cczT5zwjJhzXjht7Na/fPWR/YdQoYhW9GwvGvuH6+REfuX6GZ854tipDxCEKjoYzVvPJa1tU7hG+/MpNbtQVD/YP+K//7Q+59th1Ll2+uLJOWy51sElv/QqL+X3U5om+k5T9MEDbjHtKTCjVedqjmuF+73f/3e93/ENXuCzZxxxzEIS2bfnGN77FV/73Vzl+sMs4nvCRqyM+dGmNNW1p8Pxot+G1WyfcbzxT8SxQGioW4ji2iluTyFv3jzlatKwN+4yHFapGdB6nSs/g3HqP4aDPOzsnTKLn8PAAEeEDH3wWX2mn7TJfEvH0RFgc30HDPEMKQczhzGGamKxSuaZNRcQiGo24yPipsDpdtM0YrDRnnQkSjJs3b/Hlr3yVnQe7DLTlqe01ntwcok1LHQfsLvq8udtw2A5orb8sgSUQgQZhJj3k6gd4/xd/h8m1F/nxYkgrAxyOKB5zxlhnvHCl4mOP9OjFhrZp+NrXvsabP3orl+IrTSWBKEB/gBudwcQnLNQNrJzu0Ui2AEKgnc85OTjg6MEemoqghOeXnSBbisIMYqCup3zzm9/kvfduIO2Cs33j+tlNkCFHNuBAx7x90LLTCDOU0HEDRrCKNlaYOXq+4pd+9Z/y8//sX/Kzv/VvsMd/ltv1EBWPd2AaEQdn+i2ffnqbx9cNT8PR4T7f+Po3WSxCInbymIDlEYDgKvobFwnShzQbk9ZfOAlLVq1mSAg0kymzgyPa4wk0AS3wwkqfcVktd6kqSODW/du8/PLLyKJlU+GJsxtcvniJ9//85/ml3/l9Pv3b/554/nHmWhE6JsoTJDHeackt/dGQx598AvVgwxGPfeKXue8vsvADEJdaNargPJc3HD/z5CZ9qREiL7/yCjs79/PGXHbhJJSgDr92Htw6y45bwCSgpmgUaCL1dMbR7h6TvQNkUeNizGQ0pVUel/V6ZnjKDdvGeOWVH3Dr1l0qM66N+1zd3uTR5z7Chz//G2w88xEufPDjvPgLv4Lzw8RwW9KLRIdai7M6DWmYw7thgu3a5+zl65x54nn2wxBxHqdVStWVMHAtLzw24vHN5GJ7B/u8/OrLWKxzZ7sj9VPLsj9AB5tEekQ1grYoDRID7WLBycE+J/v7hMUMD9laIzmkr+i+VJKF8Mwh9vjwhFdfepkwn7FVNbzv4oiN7W2efvEXCGtnCdLHVz3OnztH5V2uBFugwZnhLGR+QJhP5hztHWLRE8yDVJy58lPs1RWoy1xJahb1HJwfwAuPn2egENqW733/Vep6mmCxFZInYAYtRjU+R9RRLt8DVs+ZH+4z2dulnZygTY2LiQ6IQNQUfDWZ2CrZmtklAhCIsebmzXe4+e6PGLDgkXXj8pZn4/Jl1i4+Ajh8NDQaTdMQLWDSZrjbJKhOauSIBGK74LVXvst8dkIkEDQwj553d09oMm/gTKjMIyg9FZ64tMEZ36DW8uZbb7N/eJSoGyuKizgTjB5+fIbgh8RGiZOG6cER4eQY1yQ36HYqkuKuSA6wKwGyg6dS4HWgDQt+8HevMpscs17B1S3P+iAyGA/Au+xfAbThZLJPjE1mrySV59lPUUspK7Z86//8Ba++9FeE+phFfcxfv/QKP7q9y6wtRZLlWS2HE8e1MwPet91HrWX/4Ij3btxKAVFyjBDrnCQ6T2vQTBfUh1PiItDGkFzHOagq6A9g2EfXBjCskN4An7uoXVW5DJVJGLPplLfefgexyJl+4KcunmV9GJFwiIYpVo0xdRiBg8k+IdaZn0h/i5p7qHmDEgPTgz3+4D/9R/7+tb9BJPCtv/wrztoB0/o8m4PCLQQiHqgYS81TF9f5u/v7zGPg5u3baarHpatG0ZTiAVXDwoJmcow2EZMeDAeI9/SqHuo96jV3C41oLdr28UuXkMwtuFRuA5hxeHDA7Ts7VBZ4fMvx2BmPdzUns7vY0U1kuE2wHphwcHCAWcRiSGOKuiJWy4k8gsSGO+++y5+89wZIS7QeWkWOpg2XBoKQYHLIAdsTubzVoyctjsDOvXuEaGiVbdh8CnEacBKIzQznoDcaoNUIhikISx6sTYRuwCSTSq7CU1rmWWeSjcxMsGjcubPDg90DztDy9OUzbFUNqhENR9z9/l9y7RPruME55nVg7+59Yky9j9J3lFL5pbyUMGuUzBFkpkCVRWsczGaknpPl/mdEJMHj7fGAgQa8NRwfH3bTNKkRvexk4gQ/8LhRj54fgBuCC3lXrhsDSBWqZSwSU8uPQrjgWS1KDMetW3do6paz6yMunVmn52YIsOECk3de4mZzzKUPfYo7DwJ3fvweFgZE2jwukCxDJWRm2+UOu1vyh6IYFS0wCy24RAcgisYUYA1lbegZeMU1gbapy1bADJemCgiieNdD+wN0MMgua6jETgmRlJWwBLlS2oyl6MqmTILZiZ2GNrTcvbdDDDUX1iu2RxU9WpDUvahoWNz4If/w7pt8590Jd9/bIQ2HFmySLCvmRoxkus9osyYklcjaEiXSxthR/GpJEZE08NHz0PdA4/HOp9YjeTZDImk62GOxwukQV1VYTIE8CaCQO2Wis8xgJB/w5UtWKrMyv9C0NQ/29lAaLm6uMXQBDZmviGmBI4y+Rtamx1g9x6x3SgjdwFkeQRYp0zVLWC80HRqMwXBudXQsp1RNgNNw9PsDyqAqpplLiakN2LYQu95WNwwnBSZk5r0UkrLk50IiKbo22bIsr9uGk8mEngTOjHtIaZJk8zKLBeCyMfA4QueDaSMO6+ab8n+tzGmvuKA5xCKV+mS23bhy6Cpt6IE4UGG8uYnlGUvFJ75EDaMltHMILWql+5UdQlroRoysG9cqE30+TafYEkrkEGYWaeuG6cmEvkbWB4pmU1wNrKmcNbZGQ6oyWkjiCS0zz2WkTbrzugonfRbFCwyqCtHUZ6VDpxGNSrAqdTFVuXTpUh5kyfW4FaqgIdbHqDXJErsxvUVKtwSWG82ukUtz7UAUIGbLOToz2rqhni3oO2VYaWrhJTGWJeQ9KmvDPv1qhQ7r9G7dtsVW89IKZySJuhtUnjKalMw6DykJNG2qJ3qV55GrV1AlPynQXQTCnHbyAAl1Eo7lOLTi9il9hs79yxdaAiRRMmu9pK9CCLRNixNXSIuuqC8fjRSxx8OKzaHHZZ8sPEAZVU1+WSa2sxuS+xG0rPVhPHB0X3Ud7MQx7s9qZq1x9tw5Lp2/CFb4xtCd42ONLI4RqxNHQWn85lHJNLazFELHTqaeWjJfPJbTZ5IitCHQhJYQhRBT6630OApXYWIggbW+cWmjYkCep8Jl7jAPhuVMcmpYVPIQitRsb3g213qoFnPP5IsJLZ57B1NmQXn22ecYr2+m6J8zgBlINLSew+wIsUWmFw2xCDELwUoi6PaflyKp1jDTzozKqHHMQyGu16M3HNOYA6m6m+fz878aBrrgqavbjKTFYaDlEQcQyWOCp0aSoMB4peXKxU36FRgF4GXSBaURz+29E6Qa8sILP0O/P8rfW5ZXIlyakyO0nYEFAjF34svIUR5H7Oar6PoaIpqQZRFMOinVBiaJIBltbHPp8nkm4T5BAkqT+gRiaEycAwLeIk+eX+fcsMfhpLCjZaCjWFpnb7lQiiiBoTOevXiOCiGKw2WfTkOwxjRE3t5fcOGR53j/00/inHRtyJjnuXphwmy6g1BnHJJnLfI9lZQZY7bE0rZMdVBEC/BNGs6NHROcCJUTxsMhj1x7kjt7C0wqNDYoLdadlybgHMaldcdPXVqnp8nR+lZmJfNISIqWaVkC0SWBXd7o8cT2ekq/pLZB0GQdgcCt/Sn7bcVHP/5zrG+M6Z4qysOixIjNDgmzAzoMmcFyFC20U7K2PIKQ3CSJyKwtMaKYjsvxMOIsaWprWHH57GVa2WTe9HHiUodIBVNBNDXwRWDNt3zo+jZnqkBlEZ+5Q+m0H3FEnIBYD40VA+3x4WtnODuMVDQ4WoJYmtUgEk34x9uH9LYu8NEXX8S5agUoJSa6Cg31yT60dRJALFOgbRqbpowm5mGDmCiC8vRZWpuspjNNTRxSShsNeqwNegx6a2xeeoK7x0KUHqY5hWp6SEw1aaFnLR+4tMZzlwesuQDqUU3aRyqcORwtQno2ZIDxyFj42JPnWdNFGv4gzW5iqd96f2q8eb/m+Rd/jguXLuTIolh+OkcJuHpKffwAjXXuZbhUBVubLFzL8ElyF83PnSUnTQ3w1BmLUJodmcAn4nC9If3ROnuzBZef+WluHBlttYaIoFJQevYzTfXEZtXyueevc33L6LuIahrvKegAgaiCc4GzvYbPPneZRzY9ajVlflolItbQSo/v35jQO/c4P/+Zz1E5TQqQPFpExIWG5mgHFkdobFbSJZRheIslJqQVx5LGxQgCEZcj3al8kT+JR3tD1jbPcu9kyoWnnuXQb7BXe1CPy8cGdUQqyIENiTy66fnVDz/G9XHL2Df5CZkUE6Ib4NSz7ua88Ng6Lz62xVAakFwYZegepMedacXre45PfO4LXLhwORH4OVMpisYGme+zOLiNj4sU6UpKLfgIw2X37ICD5SMsIjF1/VODR/MEXJ6JUNJkiTrHmfPn2D3eY3hmi0ef+zjfvzWhlsRKJcyT/C3VCAlOD7Xl+avr/MbHHudDF4RzvZahtvS1YU3mXKyO+OhjymdeOM/mYIbPaDVK7lWI58DW+es3jrjyzKf48Ec/hfd9ltPhCdZX7YJ67wbS7KNWr2CTgiqXzWzJRZ9ZYsmkaWlPJtSHRywODjNDlUGSWMxcYYJWUSKXLp5l8b3vUc+O+dAnPsNXfvA33Dg64tp4RCUBtRafNyGkLpLRMtSGD13pc+XsE7y903LjwQmLtmVrJFw/V3H94pBxXxLFXx5CMYcQqcXz2nvHTMeP8Wtf+C0G61upRypGoMGIuBipj/eIJ/eQOAELiK483SGpkCvD6QmsChYioW4I0xntfEGoGwyX+Qijw/iRNAco+XnPS5fP0dPA4e49rn/wOZ791Bd4+Wt/yNrj61x0RymVms/VaEt5qMVIc0/nesbZR/t89H1jIhGVmh51moSLSrAeJousTVi4If/33py3Z0N+6V/8NhcuXcmjPxBEUmALc6iPWDy4Ta9JowAR6DCRFZ4h0Y6mpNGl0DKfTJgfH0LboiEdHxhSRmAofFqSSX5yT5Tt7TOc395m5+5diMazz3+cjSc/xnfvNezqGk3PIb5FtAZvRGdEjUQNBGdQKZVr6NuUgUzoyzw9oxEzsNEEs41IS48f3Jzy97s9fvE3/jWPPf1+XGX5AdsWk/QQTNXOmd9/B5vdT5ZgnvSocyJ0EtVHCoTiwRQJgfrkhPnRA6ytCWa0rqLprcHWhVyGr8DQ5aBVqm0H/SGPPnqZe/du04SGwdoan/wnv8af/cmEb9/+AZ98dIPL/RmubbNfll8VaFCLaWisPCMhDitslZbJlUB0yrQZ8Oo7R9zTi3z2n/8rHnv/c0gFpqV0jmiM+MWMevcW8egWPTtJWCBWCI6g5PomBUcNqZFjUWkXLfPpHIseqXq4/ia9zfMMxucY9IfI8c4b1mH/DEtLNWKWNHb79rt87evf4vOf/3U21taJBnsHD/j2n/0xh29+hxcuO66ve0ZxgbZz0JBGCS03XmSQofiyNBfnMPXU0XHzuOYfdx2DRz/C85/+ZbYvXkFcfljFRaKmbnyvmdHsvke9+w5Ve4gyxwhoHCb4rAmLQsRHcNGwEJm2numiRao+w/EW1XAd9WkCp1k0hKO7yPHO6yvlWO53rnAFhhFtzoP9PcbjLQZ+mH+YQplOJ/zw+y/xw5f+lM3mAc9uKdfWWvpMQBqIAWeApeF1yxxadD3mMmJ3Jvz4/pTZ6CKPPf9pHn32IwzWh3gptKoHaYhiVM2CsHeD+sFbuMUhLrak8Z8GZC0n/QAWMoL2RPoJ1PU20V4fVw1omprFZMLi4AFhcozNJ7h4ghzd+8fE0XS85WnSBDKmzwWUUX5VACKpi3Rv7x5vvPoSu//wCqPpHR7dhO1RZK0i9T5jRTBlZspRGzmoHfvtkMHZ61x7+nkuvO+plBlcBZpCtssAL0jEtzOag5s0u+/gFwdozEE5K0qlRxmgjyjmB0hvhB9tob5PnAfmx3s0h3epD+9hswk+tHmwIsVHObr3IyvExQpTcepl5nLRG3PPIkNSawmWgFhoAge7+9x+9y12b/6I9mgHCRNoa7wY2uvDcINq/RxbFx7hwpXH2Dp7Ed8bYi49TA+JhFEzvAWEmjZMCA/eowvemQIAAAKwSURBVD64SdVMcE2uHyQ9tBIy0hTfR/062h/jeyOcKnU94XB3h8XuLnF6iGuOce0Eb23elyfiic4hx/fesp+0glUaC8QSrxBTMzMFwZiekWgz3CYmaG4mxDYSm5rQTIlhiriA8xXq+3g/xLn8nCma23Wl+MlcoimEFpnvUe+9iZ7cQNomgb3ugX9H66CtFDdcw/fXcTJC8dh8wcnuLaZ7d4jTfVwrqLVgC8yajDLJjJVD1GUWu/SIu+c7VwOnZrKk/CRCJk5UsqVoyjqS+UgRxHt8lYgcGHeMYDeHQaHYS+Va/Dog1mJhTjw5oN75MdVsD7UF5QGU4Byt95hfw43WqfpDxCJtXdMu9mF2xGJ/l/poH+oaZ21qFJnkgN1PRV3Zk7SALz8HJMUH8oJzMb+EaWnhK1ayyocuDyw0+ApxK0D+pSLK5wzly7lpojdiEgjzI+qDe7THD5B6SqMVUddRV4FWSG9Erz/C+T4xROrZlNCc4JopujikPdojTo7QNiQQZZpxkVFGmEUyP5u5EgCfuP4VtyjgamWwrHOdbjiMh75fnXUuP6sST5/TcR55LsvKMQ6jl6wkNrR1i4hnMD6LbF5Gqj5VpQmDuIpgAqFmfrJPe7BLLy7Sc6OzI2R2QDg5ItY1IWpK30TKc5+SOVAthJIV5RaI/VCWYJXgzJtLr+I2q1ZQ9mkdc7S8nqwclwVhCeysnh00C1YGVGsXkfF2/sWKDMCsIcRA07aEGFCnyNoWfe9pJye0+7swB5m1EFL8UgIxN6M0B3ST9JTgclwkNZxd1DIf8dC+fyJwRn7yoFWL+P/s+9Qf07twBd3fs/UpNVaOcVUifS3iTQhNpGkbEKhcj14/EwCDLWxoVONIf7zPbPcG9W4fibeQuI+GOpPQPgsi1xuSOdnuYVildRX/DwyIBNmWraJ7AAAAAElFTkSuQmCC","operation_ts":"1970-01-01 08:00:00.000"},"op":"c"}
```

##### base64 验证

编码转图片在线网站：https://www.sojson.com/image/basetoimage.html

到此图片功能验证成功。

## 完整SQL

```
set yarn.application.name=cdc_mysql2kafka;  
  
set parallelism.default=1;  
set taskmanager.memory.process.size=3g;  
  
set execution.checkpointing.interval=1000;  
set state.checkpoints.dir=hdfs:///flink/checkpoints/cdc_mysql2kafka;  
set execution.checkpointing.externalized-checkpoint-retention= RETAIN_ON_CANCELLATION;  
set pipeline.operator-chaining=false;  
  
CREATE TABLE mysql_cdc_source (  
  id intPRIMARY KEYNOT ENFORCED, --主键必填，且要求源表必须有主键，flink主键可以和mysql主键不一致  
  name string,  
  price double,  
  ts bigint,  
  dt string,  
  img String,  
  operation_ts  String METADATA FROM'op_ts' VIRTUAL  
) WITH (  
'connector'='mysql-cdc',  
'hostname'='ip',  
'port'='3306',  
'username'='username',  
'password'='password',  
'database-name'='cdc',  
-- 'server-time-zone' = 'UTC+8', -- MySQL 时区，根据实际情况添加  
'table-name'='mysql_cdc_source2',  
'debezium.binary.handling.mode'='base64'  
);  
  
CREATE TABLE if notexists test_kafka_sink (  
  id intPRIMARY KEYNOT ENFORCED,  
  name string,  
  price double,  
  ts bigint,  
  dt string,  
  img String,  
  operation_ts String  
) WITH (  
'connector'='kafka',  
'topic'='test_kafka_sink2',  
'properties.bootstrap.servers'='ip1:6667,ip2:6667,ip3:6667',  
'properties.security.protocol'='SASL_PLAINTEXT',  
'properties.sasl.mechanism'='GSSAPI',  
'properties.sasl.kerberos.service.name'='kafka',  
'properties.group.id'='dkl',  
'scan.startup.mode'='earliest-offset',  
'key.format'='json',  
'key.fields'='id',  
'value.format'='debezium-json'  
);  
  
insert into test_kafka_sink select*from mysql_cdc_source;
```

 

🧐 **分享、点赞、在看**，给个**3连击**呗！**👇**
> 已吸收至：[[07_工程与架构/0703_工程实践与质量保障/070305_测试质量/070305_核心知识点/测试质量体系与AI测试边界|测试质量体系与AI测试边界]]、[[07_工程与架构/0703_工程实践与质量保障/070305_测试质量/070305_知识地图|070305_测试质量知识地图]]

---
title: 百度又开源一款压测工具，可模拟几十亿的并发场景，太强悍了！
author: Java高性能架构
date:
url: http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247527899&idx=1&sn=4cbdf6452c0a1fb3cbbf3c17ffe3d73f&chksm=c0c95d72f7bed464f19916780bc1cd79072076bd2f434d9b839f2317af743f9cc8221519e3c1&mpshare=1&scene=24&srcid=12150GdgvPwoGFgolkfTX5iC&sharer_shareinfo=bf6a28c42c1a1c8e2719ab33db75c4ff&sharer_shareinfo_first=bf6a28c42c1a1c8e2719ab33db75c4ff#rd
---

关注上方蓝色“Java高性能架构”，设为星标⭐

回复“**资料**”获取整理好的**面试资料**

大家好,我是 风哥

[****2023 年 Java 架构师：视频课程****](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247526198&idx=1&sn=de207e207781c49a0e542b6c91a5fd55&chksm=c0c9579ff7bede890b6f080dfd959115b74b18c7238dc74f8a632f707b5449408b05c856270d&scene=21#wechat_redirect)

[****2023 年 Java 进阶高级：视频课程****](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247520542&idx=1&sn=ab0c319d7a1ad8c42a652da07b670aab&chksm=c0c9a1b7f7be28a124ebc0c7fe6772e311b6d546e46df4a9cfacd5e90b3f34a11e60dd573d06&scene=21#wechat_redirect)

[****工作流专家(Flowable+Camunda)：视频课程****](http://mp.weixin.qq.com/s?__biz=MzkwODMzOTY1NA==&mid=2247527312&idx=1&sn=9c269806ba799bdd383d8199874f145f&chksm=c0c95b39f7bed22f7f1840602c8ce906f8bb2435799eeb0ab636935a41326f708793007c5fe8&scene=21#wechat_redirect)

dperf 是百度开源的一款基于 DPDK 的 100Gbps 网络性能和负载测试软件，能够每秒建立千万级的 HTTP 连接、亿级别的并发请求和数百 Gbps 的吞吐量。

## 优点

### 性能强大：

* 基于 DPDK，使用一台普通 x86 服务器就可以产生巨大的流量：千万级的 HTTP 每秒新建连接数，数百 Gbps 的带宽，几十亿的并发连接数

### 统计信息详细：

* 能够输出详细的统计信息，并且识别每一个丢包

### 使用场景丰富：

* 可用于对四层负载均衡等四层网关进行性能压力测试、长稳测试
* 可用于对云上虚拟机的网络性能进行测试
* 可用于对网卡性能、CPU 的网络报文处理能力进行测试
* 压测场景下，可作为高性能的 HTTP Server 或 HTTP Client 单独使用

## 性能

### HTTP 每秒新建连接数

| Client Cores | Server Cores | HTTP CPS |
| --- | --- | --- |
| 1 | 1 | 2,101,044 |
| 2 | 2 | 4,000,423 |
| 4 | 4 | 7,010,743 |
| 6 | 6 | 10,027,172 |

### HTTP 吞吐

| Client Cores | Server Cores | RX(Gbps) | TX(Gbps) | Client CPU Usage(%) | Server CPU Usage(%) |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 18 | 18 | 60 | 59 |
| 2 | 2 | 35 | 35 | 60 | 59 |
| 4 | 4 | 46 | 46 | 43 | 43 |

### HTTP 并发连接数

| Client Cores | Server Cores | Current Connections | Client CPU Usage(%) | Server CPU Usage(%) |
| --- | --- | --- | --- | --- |
| 1 | 1 | 100,000,000 | 34 | 39 |
| 2 | 2 | 200,000,000 | 36 | 39 |
| 4 | 4 | 400,000,000 | 40 | 41 |

### UDP TX PPS

| Client Cores | TX MPPS | Client CPU Usage(%) |
| --- | --- | --- |
| 1 | 15.96 | 95 |
| 2 | 29.95 | 95 |
| 4 | 34.92 | 67 |
| 6 | 35.92 | 54 |
| 8 | 37.12 | 22 |

## 测试环境配置

dperf 的以上性能数据，基于下面的配置测试得到：

* 内存: 512GB(大页 100GB)
* 网卡: Mellanox MT27710 25Gbps \* 2
* 内核: 4.19.90

## 统计数据

dperf 每秒输出多种统计数据：

* TPS, CPS, 各种维度的 PPS
* TCP/Socket/HTTP 级别的错误数
* 丢包数
* 按照 TCP Flag 分类的报文重传数

```
seconds 22                 cpuUsage 52  
pktRx   3,001,058          pktTx    3,001,025          bitsRx   2,272,799,040      bitsTx  1,920,657,600      dropTx  0  
arpRx   0                  arpTx    0                  icmpRx   0                  icmpTx  0                  otherRx 0          badRx 0  
synRx   1,000,345          synTx    1,000,330          finRx    1,000,350          finTx   1,000,350          rstRx   0          rstTx 0  
synRt   0                  finRt    0                  ackRt    0                  pushRt  0                  tcpDrop 0  
skOpen  1,000,330          skClose  1,000,363          skCon    230                skErr   0  
httpGet 1,000,345          http2XX  1,000,350          httpErr  0  
ierrors 0                  oerrors  0                  imissed  0  
```

## 开始使用

### 设置大页

```
#参考如下参数编辑 '/boot/grub2/grub.cfg'，然后重启OS  
linux16 /vmlinuz-... nopku transparent_hugepage=never default_hugepagesz=1G hugepagesz=1G hugepages=8  
```

###

### 编译 DPDK

```
#编辑'config/common_base'打开PMD开关  
#Mellanox CX4/CX5 requires 'CONFIG_RTE_LIBRTE_MLX5_PMD=y'  
#HNS3 requires 'CONFIG_RTE_LIBRTE_HNS3_PMD=y'  
#VMXNET3 requires 'CONFIG_RTE_LIBRTE_VMXNET3_PMD=y'  
  
TARGET=x86_64-native-linuxapp-gcc #or arm64-armv8a-linuxapp-gcc  
cd /root/dpdk/dpdk-stable-19.11.10  
make install T=$TARGET -j16  
```

### 编译 dperf

```
cd dperf  
make -j8 RTE_SDK=/root/dpdk/dpdk-stable-19.11.10 RTE_TARGET=$TARGET  
```

### 绑定网卡

```
#Mellanox网卡跳过此步  
#假设PCI号是0000:1b:00.0  
  
modprobe uio  
modprobe uio_pci_generic  
/root/dpdk/dpdk-stable-19.11.10/usertools/dpdk-devbind.py -b uio_pci_generic 0000:1b:00.0  
```

### 启动 dperf server

```
#dperf server监听6.6.241.27:80, 网关是6.6.241.1  
./build/dperf -c test/http/server-cps.conf  
```

### 从客户端发送请求

```
#客户端IP必须要在配置文件的'client'范围内  
ping 6.6.241.27  
curl http://6.6.241.27/  
```

### 运行测试

下面的例子运行一个 HTTP CPS 压力测试。在 server 端运行 dperf ./build/dperf -c test/http/server-cps.conf

```
#以另一台机器作为client端，运行dperf  
./build/dperf -c test/http/client-cps.conf  
```

开源地址：https://github.com/baidu/dperf

---END---

```
```
风哥与朋友波哥一起联合打造了《Java架构师成长之路》的视频教程。完全对标外面2万左右的培训课程。

除了基本的视频教程之外，还提供了超详细的课堂笔记，以及源码等资料包..

课程阶段：

1. Java核心 提升阅读源码的内功心法
2. 深入讲解企业开发必备技术栈，夯实基础，为跳槽加薪增加筹码
3. 分布式架构设计方法论。为学习分布式微服务做铺垫
4. 学习NetFilx公司产品，如Eureka、Hystrix、Zuul、Feign、Ribbon等，以及学习Spring Cloud Alibabba体系
5. 微服务架构下的性能优化
6. 中间件源码剖析
7. 元原生以及虚拟化技术
8. 从0开始，项目实战 SpringCloud Alibaba电商项目

点击下方超链接查看详情（或者点击文末阅读原文）：

(点击查看)  2023年，最新Java架构师成长之路 视频教程！

以下是课程大纲，大家可以双击打开原图查看
```
```
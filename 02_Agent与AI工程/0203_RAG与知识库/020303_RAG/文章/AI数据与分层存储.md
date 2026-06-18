---
title: AI数据与分层存储
author: 竹言见智
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzg5MzkxMDA5Ng==&mid=2247493889&idx=1&sn=b4b7340cdef2c67a0edbe39a2213a4fe&chksm=c16d65df0ea9022e27cb40b9818623a800d45cf1811281f190db29766f68aac74885743348d3&mpshare=1&scene=24&srcid=1105yEjQhnrtQIa0O89kHwRi&sharer_shareinfo=ab8a943f9571eaab77185bdc7c79a99a&sharer_shareinfo_first=ab8a943f9571eaab77185bdc7c79a99a#rd
---

本文带来ocp2024上关于AI数据存储和Solidigm QLC SSD存储实践分享。

AI集群存储架构可大致分为以下三部分：

* 计算层：由自带有限存储的GPU服务器组成，也被称为“GPU NVMe缓存”或“计算SSD”。
* SSD缓存层：当前通常由TLC（三层单元存储） NAND组成，用于解决HDD性能不足问题。
* HDD对象存储层：包含许多存储设备的存储服务器。

AI训练推理过程中**数据pipeline包括数据load和预处理、向量化处理、训练和检查点 save/restore、微调和存档等**，数据load和预处理、训练和检查点及微调步骤均需高吞吐和低延迟，数据增长则将带来更多存储空间及更快存储性能要求，对训练和ckp影响更为显著。

* 数据load/预处理：存储在SSD层和HDD层。
* 向量化处理：在GPU的本地内存/本地存储中处理。
* 训练和检查点ckp：最新ckp存在SSD上，老的ckp存在HDD上以进行长期保留。
* 微调数据：一般存储在SSD层。

**除考虑数据存储的吞吐低延迟外，同时还需要考虑顺序/随机读写过程中数据移动对AI集群的影响。****文中对比了Solidigm QLC（四层单元存储） SSD和hdd在文件IO（FIO）和MLPerf Storage存储基准测试上的性能表现**。可以看到：使用SSD的系统在加速器利用率和带宽要求方面表现更好。

最后对比了TLC+HDD方案和QLC NVMe SSD方案，提供了QLC SSD在数据load、预处理、训练和ckp、存档方面的性能表现。
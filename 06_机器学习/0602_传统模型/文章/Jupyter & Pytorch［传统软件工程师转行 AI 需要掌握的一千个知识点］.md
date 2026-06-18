---
title: Jupyter  & Pytorch［传统软件工程师转行 AI 需要掌握的一千个知识点］
author: 你必须往前走
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410492&idx=1&sn=62e9423e3b26be985fb0e83e88e5c3bd&chksm=8d22b2b7ed75b3eeb0d1a136e70a94c041e7c9f0e49f4c477f50d2ffd643a7adf20f98f28a58&mpshare=1&scene=24&srcid=09091sGASt1NwM0HoCRltuZT&sharer_shareinfo=1f3e836f984b5fb32087f79abc44a84b&sharer_shareinfo_first=1f3e836f984b5fb32087f79abc44a84b#rd
---

这都是曾经最不起眼的算法

如今已然成长为参天大树

后面按照如下脉络学习总结

基本上都是大红大紫过的，基本按照时间序排序的

主要写一些主题脉络思想，知识点

基础知识 + 经典 模型简述

前边模型的思想、创新点，会成为后边的组件、基础知识（现在有点理解写书的逻辑了，没有基础知识点牢固，后边的新兴技术 就无法 深入理解吃透（万丈高楼平地起，超高大厦 地基占 整体造价的30%～40%））

这篇文章估计要写好久…

不对，写一点，发一点

简单，才足以坚持，坚持才足以完成

Agent

MCP

RAG

商业模型API

测评

开发框架： pytorch

基础概念、组件→

张量

自动求导→监控所有动作→DAG

神经网络模块

优化器

数据加载

动态计算图 DAG

三层架构设计

优化思路

Python 代码是如何被转化成 GPU cuda 编程的？

算子融合、循环变换与展开、内存布局优化、自动调整（针对不同的操作和硬件，自动尝试不同的线程块大小、循环策略等，并选择性能最优的配置。）

如何使用 pytorch 编程？需要学习哪些知识点？

其实从官网学习，是最直接、最正确、最佳的

官网：https://pytorch.org/

第一节段

1.环境搭建

2.张量

定义、索引、创建

形状，数据类型，所在设备

支持100种张量计算→好多

————————————

支持逐元素的加、减、乘、除、幂、平方根、指数、对数、绝对值、取负等运算

包含矩阵乘法、转置、逆矩阵、行列式、特征值分解、奇异值分解等

对张量进行求和、均值、最大值、最小值、标准差、方差等聚合操作，可沿指定维度进行。

逐元素的比较操作，返回布尔类型的张量。

改变张量的形状、维度顺序、增加或减少维度、拼接和拆分张量。

生成各种概率分布的随机数张量。

支持类似NumPy的索引和切片操作，以及基于条件的元素选择。

———————————————

与 Python 的NumPy 桥接

与NumPy 共享 底层 内存区域

如果是 CPU 上 互转，几乎零开销

.to（“cuda”）   .cuda（）

3.自动求导

核心中的重点，神经网络的反向传播

requires\_grad = True

理解计算图

使用.backward()自动求导

.grad  访问 梯度

==========

# 创建一个需要跟踪梯度的张量

x = torch.ones(2, 2, requires\_grad=True)

# 对张量进行操作

y = x + 2

z = y \* y \* 3

out = z.mean()

# 计算梯度

out.backward() # 等价于 out.backward(torch.tensor(1.0))

# 查看梯度 d(out)/dx

print(x.grad)

==========

第二阶段

1.模型构建

用积木（组件）→搭建→完整网络模型（变形金刚）

torch.nn.Module

线性层

卷积层

前向传播

归一化函数

线性组合

2.训练三要素

损失函数

如何选择、怎么用

nn.MSELoss →回归

nn.CrossEntropyLoss→分类

优化器：

根据损失函数更新参数

→SGD、Adam、

循环

这是核心

====================

for epoch in range(num\_epochs):

# 前向传播

outputs = model(inputs)

loss = criterion(outputs, labels)

# 反向传播

optimizer.zero\_grad() # 清空梯度

loss.backward()        # 计算梯度

optimizer.step()       # 更新参数

====================

第三阶段 数据处理

1.数据加载

Dataset

DataLoader：批量加载、打乱数据、并行加载

2.常用工具

torchversion

torchaudio

torchtext

3. Conda Venv   git

第四阶段  进阶与部署

提升性能、调试、部署

1.保存or加载 torch.save  torch.load

2.部署 torchscript、onnx

3.分布式训练、性能剖析profiler

混合精度训练torch.cuda.amp

官方文档：

https://docs.pytorch.org/tutorials/

Jupyter

超级智能笔记本！！！

前面告诉你代码怎么写，中间是写代码的地方，下面就是运行结果

所见即所得！！！ 非常非常酷

书本+笔记本+IDE +调试工具

学习Python、Cuda 编程的首选工具

CUDA 编程

[CUDA［传统软件工程师转行 AI 需要掌握的一千个知识点］](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410491&idx=1&sn=75f9f273734c976238d65451c2e89c70&scene=21#wechat_redirect)

GPU

[GPU［传统软件工程师转行 AI 需要掌握的一千个知识点］](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410472&idx=1&sn=5d471c98e6643e4e56915a9c3ca98254&scene=21#wechat_redirect)

开源模型

[LLaMa - 开源模型［传统软件工程师转行 AI 需要掌握的一千个知识点］](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410452&idx=1&sn=c6400a924eb7b291a5467e6c191c61fa&scene=21#wechat_redirect)

世界模型

[世界模型［传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络］](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410438&idx=1&sn=de9bae970ecfe10c735d0443c7333307&scene=21#wechat_redirect)

Mamba

[Mamba（线性注意力）【传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络】](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410437&idx=1&sn=1689159217030449244d007373c88c6b&scene=21#wechat_redirect)

MoR

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→MoR](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410435&idx=1&sn=96c9511207c0b51282bb47cd108b7e0f&scene=21#wechat_redirect)

神经辐射网络

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→神经辐射网络](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410432&idx=1&sn=7293e1007faac60aef67f1ef05306183&scene=21#wechat_redirect)

RLHF

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→RLHF](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410425&idx=1&sn=97b758a9a907c4b58d56bc9cd3bcc934&scene=21#wechat_redirect)

COT 思维链

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→思维链](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410423&idx=1&sn=bbd9c1973f379c09464356b0390b1b0b&scene=21#wechat_redirect)

DeepSeek R1

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→强化学习 DeepSeek ：R1](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410421&idx=1&sn=ff2ea101f1ad0b7141d4b91a5bf39b57&scene=21#wechat_redirect)

强化学习

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ 强化学习 Q-Learning、DQN、PG、PPO](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410419&idx=1&sn=0e342836db1b7ded445d02beb8e30f90&scene=21#wechat_redirect)

多模态

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ 多模态 CLIP](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410417&idx=1&sn=478bb1f1af601676f245e133c408115d&scene=21#wechat_redirect)

Diffusion

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ 扩散模型 Dissfussion  DDPM](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410415&idx=1&sn=a13492295c2e8a8502bf846faf938608&scene=21#wechat_redirect)

GPT 系列

GPT1  GPT2 GPT3  GPT3.5 GPT4

终于到了这个跨时代的GPT了，足以记录在人类文明史的一页

GPT1  

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ GPT1](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410404&idx=1&sn=739f1b3e276c28cca4dd8daf0fdc0e00&scene=21#wechat_redirect)

GPT2 、GPT3

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ GPT2、3](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410411&idx=1&sn=c32d574d2f1e4e5e56941dc0cc09789a&scene=21#wechat_redirect)

GPT3.5 、GPT4

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ GPT3.5、4](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410413&idx=1&sn=5bbecf0d78d58dbea391bfe9e9d89b67&scene=21#wechat_redirect)

Transformer  
[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ Transformers](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410402&idx=1&sn=2c72f55b5084911047542c7d6822ee69&scene=21#wechat_redirect)

GAN

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ GAN](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410400&idx=1&sn=f30e6728a7bb22660fb0896ad08248ad&scene=21#wechat_redirect)

AlphaGo

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ AlphaGo](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410398&idx=1&sn=5f0fb5c4b9d7548138b1fd093fc42e90&scene=21#wechat_redirect)

ResNet

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ ResNet](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410395&idx=1&sn=d3c43c7f586b6c5c7ef7806dac1ec3d1&scene=21#wechat_redirect)

CNN AlexNet VGG 

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ CNN AlexNet VGG](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410393&idx=1&sn=42f788f6e3424fec253bffa9a2fb565a&scene=21#wechat_redirect)

VAE

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ VAE](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410390&idx=1&sn=e4bb28406e47cdae3a2ccdd2e6730e85&scene=21#wechat_redirect)

深度信念网络

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→深度信念网络](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410387&idx=1&sn=e903dcaa177d7be94c3070b23ddbea3e&scene=21#wechat_redirect)

[集成学习（主导了十年）](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410378&idx=1&sn=cc04c494b1bd76aee84b6d21de4ceaf7&scene=21#wechat_redirect)

RNN Lstm  
[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→RNN](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410370&idx=1&sn=c384ed916708575ba9c32eb07239110c&scene=21#wechat_redirect)

CNN Lenet-5  
[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→CNN](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410354&idx=1&sn=2dbbbd83adf281a754f240b85a994034&scene=21#wechat_redirect)

Hopfield网络&&玻尔兹曼机  
[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→ Hopfield网络&玻尔兹曼机](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410347&idx=1&sn=15834f2104e958bfb3260e5a52da19d8&scene=21#wechat_redirect)

反向传播  
[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→反向传播](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410335&idx=1&sn=a1f0a401f092e4fe25d55b1ac0845a0c&scene=21#wechat_redirect)

多层感知机

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→多层感知机](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410331&idx=1&sn=b1a7cf867b1ee906649b00dc47c21df0&scene=21#wechat_redirect)

感知机

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→感知机](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410324&idx=1&sn=c52d8ebc0c518d090961c4291736249e&scene=21#wechat_redirect)

MP 神经元

[传统软件工程师转行 AI 需要掌握的一千个知识点→神经网络→MP神经元](https://mp.weixin.qq.com/s?__biz=MzAwOTM3NTcyMQ==&mid=2460410316&idx=1&sn=0fea0dd3ef0b08399b0c06bdeeeabdaf&scene=21#wechat_redirect)
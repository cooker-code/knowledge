---
title: OpenAI：Let’s Verify Step by Step 解读
author: AI蜗牛车
date:
url: https://mp.weixin.qq.com/s?__biz=MzA4ODUxNjUzMQ==&mid=2247504370&idx=1&sn=cb3577fa2edd4a62a27c7095b8d5e5ef&chksm=91257a9dd11fcb567923acadb06a6fec86bb10a5b01d96784be9c79ef3d245d7c5c3818cc04a&mpshare=1&scene=24&srcid=0704C3UzGBpDOmeuvx3theMh&sharer_shareinfo=bedbc2263fe65196036ae27d57809ac3&sharer_shareinfo_first=bedbc2263fe65196036ae27d57809ac3#rd
---
> 已吸收至：[[01_LLM与大模型/0107_模型评测/0107_核心知识点/业务评测集与过程监督准则|业务评测集与过程监督准则]]

# 前言

Let’s Verify Step by Step

OpenAI的一篇经典论文。

* 链接：https://arxiv.org/pdf/2305.20050
* github： https://github.com/openai/prm800k

# 实验

## 目的

* 对于multi step reasoning的问题，模型经常出现逻辑的错误
* 讨论结果监督（Outcome-supervised Reward Models ，ORMs）和过程监督（Process-supervised Reward Models，PRMs）的优劣

## ORM和PRM的区别

* ORM只关注于结果对与否（存在结果恰好正确，但其中的reasoning的部分出现错误，属于误判样本）， PRM关注于某个过程的对与否
* PRM可以对与错误的样本给出错误的步骤，而ORM给不出错误的细节

## 实验细节

* 使用GPT-4训练得到ORM和PRM
* 利用GPT-4当作生成器，对一个prompt生成多个结果（BON），选择其中一个结果，作为final response进行评估

## 数据构成

对每个步骤进行人工的标注（对与错）， 过程结果和最终结果就都有了，但也做了更多的优化：选择了更具有迷惑性的样本（简单来说就是更难的样本，模型更容易判断错误的样本）

# 结果

比较三种方式来给出最终的top1作为评估的回答

* ORM
* PRM
* vote（类似于model ensemble）

横坐标为每个prompt生成的response数量，可以发现随着数量的增多，PRM远超于ORM和vote，并且ORM也大于vote方法，说明ORM也是有一定的作用的，但是在reasoning的过程中进行反馈的作用更大。

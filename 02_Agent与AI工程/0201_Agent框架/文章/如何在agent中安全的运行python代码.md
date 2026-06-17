---
title: 如何在agent中安全的运行python代码
author: 进击的大数据
date: 
url: https://mp.weixin.qq.com/s?__biz=MzU2MDg0NjgwMQ==&mid=2247483947&idx=1&sn=62a616324bd338f95111f5ced8e231fc&chksm=fd1951c105584b01f43753842c2a6080461844e88529aedff4812e56dceee8f6c13690fbf7fb&mpshare=1&scene=24&srcid=1104f1VTPlAqD9nxyr8xWN7H&sharer_shareinfo=3da01d1cd3ef93bb3544ead9d18ef8ed&sharer_shareinfo_first=3da01d1cd3ef93bb3544ead9d18ef8ed#rd
---

# 背景

在agent中，运行python代码现在基本上是一个标配了，如果用户在提示词里注入一些奇怪的东西，或者llm生成一些危险的代码的话，那就废了。所以一般agent都是配了一个python的沙箱环境。现在就简单看看一些开源的agent是如何实现的。

---

# langgraph

用户需要配置的东西

1. 沙箱环境
2. 如何在沙箱环境里运行代码
3. 使用的大模型
4. 绑定的工具

## 沙箱环境

Langgraph使用Deno作为沙箱环境的，我其实也不是很懂Deno，这里就当做是一个启动更快，更安全沙箱环境，具体的解释可以参考其他大模型的解释。

那其他框架是如何做的呢？以huggingface的smolagents框架为例(很早以前看过smolagents的代码，现在再看发现更新了很多代码，但结构还是很简洁)

使用的是挺复杂的一套东西了， 可以设置执行的环境是什么，本地or远程or docker， 在生产环境还是建议借鉴一下smolagents的做法。还有一些其他的做法，比如用一些云厂商的sandbox也是可以的（一般大公司有专门的infra提供这样的接口）

smolagents的例子

https://github.com/huggingface/smolagents/blob/main/examples/sandboxed\_execution.py

https://github.com/huggingface/smolagents/blob/main/src/smolagents/local\_python\_executor.py

## 如何在沙箱里运行代码

回到langgraph的沙箱实现

难点在于 llm生成的代码，以及上下文如何补充进去，比如在多轮对话中llm生成的代码只有一段，利用了之前代码生成的变量；或者是利用了已经创建好的本地的python函数

针对这两种情况， 利用的是\_locals变量作为上下文：

* 如果是工具的话， 将工具的源代码添加在wrapper code中
* 如果是之前步骤的结果变量的话， 将结果变量添加到wrapper code中。

## 绑定的工具

比如这样一系列的工具调用

这些工具的代码会嵌入到wrapper code中， 在执行器中一起被执行

注释则会被添加到prompt中， 供llm来参考，是否要借用这些函数的能力

在codeagent中

这里的sandbox节点中就融合了tools的信息和之前执行的信息作为context

## 设计问题

最后来思考几个设计的问题：

为什么langgraph要这么设计成这样：把eval\_fn单独拆分呢？

-> 目的是为了将来如果要换成其他的环境，比如本地环境，远程环境， 都可以简单替换。就像smolagents那样

但其实langgraph这里实现的并不好，只能简单参考参考，实际生产环境完全不可用

1. graph退出的机制， 当前是判断不再输出代码，如果大模型生成的代码有误， 那会不停的死循环 -> 加一些默认的最多运行次数
2. tools\_context会在多轮对话中重复绑定，虽然不是什么大问题 -> 可改可不改
3. 没有压缩，多轮对话，如果代码过长的话，上下文就被耗光了 -> 压缩上下文，做一些简单的summary和local的结果 -> 更难一点是把代码放在本地的临时目录中，根据大模型的信息做召回，这就很难了
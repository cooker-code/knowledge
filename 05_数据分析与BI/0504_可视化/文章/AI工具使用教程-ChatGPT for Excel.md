---
title: AI工具使用教程-ChatGPT for Excel
author: 一森说BI
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzk0MTYwNjI5MA==&mid=2247484045&idx=1&sn=6d82bb9eff597183b7979f091ad14c70&chksm=c3be94fa8a54449fe8066a3ad09e4073ff9236ee064e7ff2a560407bb90cebffd0d29d60682b&mpshare=1&scene=24&srcid=0103GZLU2e1uiX3dxp4KgCKS&sharer_shareinfo=45b42a041751b17da9fab91e4e723876&sharer_shareinfo_first=45b42a041751b17da9fab91e4e723876#rd
---

新年的第一个工作日，祝各位2025年健康美满。

AI的趋势如火如荼，从早期的对话式生成，到如今内嵌到各种应用软件，今天分享一下个人最近使用并且觉得非常实用的一款Excel插件--ChatGPT for Excel

（可直接在excel的加载项中安装此插件）

主要功能介绍

1.翻译。多语种互翻，单词语句都能搞定

2.数据分析。基于底稿数据，自动做出数据的结果以及判断和结论

3.单元格数据处理。不用十几个公式嵌套，直接一个描述性的公式帮你处理！

以下将从几个例子，展示用法以及效果

一、翻译：

语法：=AI.TRANSLATE("Boost your productivity with ChatGPT for Excel", "Spanish")

第一个参数可以是引用单元格，第二个参数选择语种，任何语言描述皆可。这个公式可以将单元格内的任意内容都进行翻译，而不仅是中英，英中皆可。

效果如下：

二、数据分析：

语法：=AI.ASK(prompt,[value])

第一个参数为描述需要分析的内容，第二个参数为数据的范围

简单来说，这个公式类似于生成式AI，区别是基于我们给定的数据范围进行回答，我们可以绕过数据透视表，直接提问各种分类统计后的合计，平均，排名等等。

效果如下：

以下为提供的收入科目数据作为分析范围

编写公式：=AI.ASK("哪一个月收入最多，收入按照credit-debit计算",A1:G100)

获得如下分析结果

公式生成的结果包含了过程，也包含了结论，想要获得这么样的结果，完全取决于你的prompt怎么写。

三、数据处理

强大的处理方式，再也不用各种left(len())等等公式组合了！想要处理什么样的数据，取决于你如何定义描述！

语法：=AI.EXTRACT("Call Alex if you have any questions", "name")

第一个参数为数据范围，第二个参数为需要获得怎么样的数据的一个prompt

效果如下：

除了规则类的描述，还可以通过完全语义判断的放来进行数据处理，比如上面例子里的获取日期获取人名，在杂乱的数据中提取自己所需。

关于ChatGPT for Excel还有其他实用的AI公式，各位也可自己探索，相信对于日常工作，能够起到一定作用。
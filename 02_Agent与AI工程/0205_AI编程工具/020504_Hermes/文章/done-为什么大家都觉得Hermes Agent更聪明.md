> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020504_Hermes/020504_核心知识点/Hermes协作与记忆治理边界|Hermes协作与记忆治理边界]]
---
title: 为什么大家都觉得Hermes Agent更聪明
author: 智能时代指南针
date:
url: https://mp.weixin.qq.com/s?__biz=MzkwMzQzNzQ2OQ==&mid=2247483909&idx=1&sn=b95112c9b6689d21908452de6a017143&chksm=c1495fa59a1bc24b0922fd622e1509db9e8fe56510448838e2762bc5b5e476920e01be0a88fd&mpshare=1&scene=24&srcid=0415c6bCjHoob1fzK1o3VeQx&sharer_shareinfo=3654e9284aca3cac99d8c232e8234e04&sharer_shareinfo_first=3654e9284aca3cac99d8c232e8234e04#rd
---

用 Hermes Agent和 OpenClaw 的人都有个感觉：Hermes 好像更"懂我"。

不是模型更聪明，而是**记性更好**。

 

## 我碰到的问题

前一个小时：我说你以后访问网站，就使用用xxx skill，不能重新打开新的浏览器，我已经在当前浏览器

**我说** ：你用xxx skill去查看知乎热点吧

以下是龙虾和爱马仕的反应：
**OpenClaw**：用户希望我去访问知乎热点，让我打开一个浏览器，oh，知乎要登录
 然后就没有然后了

**Hermes** : 已记录。以后访问互联网时就使用xxx skill
**Hermes**：我先加载 xxx skill，然后访问知乎热点

 

主动搜索 vs 被动等待

## 两套记忆系统

### OpenClaw：像一本笔记本

OpenClaw 把记忆存在 Markdown 文件里。每次你纠正它，它记下来。下次会话开始时，把这些文件"读"一遍。

问题：**中途不会主动翻旧账**。你问"上次"，它得等你说清楚。

### Hermes：像带着一个助理

Hermes 不止有笔记本，还有个搜索引擎。

你问"上次"，它会：

1. 自动搜索所有历史对话
2. 找到相关内容
3. 总结后回答你

这就是"聪明"的来源——**不是猜对了，是查到了**。

 

## 三个关键差异

### 1. 主动 vs 被动

* OpenClaw：你纠正它 → 它记下来 → 下次会话生效
* Hermes：你提一嘴 → 它主动搜索 → 立刻给你答案

### 2. 记忆怎么更新

OpenClaw 每次改记忆，都可能让 AI 重新"读"一遍，费钱费时。

Hermes 的设计更聪明：**现在改，下次生效**。这样 AI 的"缓存"不会被频繁打断，省钱快 4 倍以上。

### 3. 能不能扩展

OpenClaw 的记忆是固定的那几个文件。

Hermes 可以接入外部记忆服务，比如：

* Honcho（对话式记忆）
* Supermemory（云端语义搜索）

想要更强的记忆，插上就行。

 

记忆扩展能力对比

## 还有个细节：压缩

对话长了，AI 记不住那么多，得"压缩"。

OpenClaw 用通用方法压缩，容易丢关键信息。

Hermes 用结构化模板：

* 已解决的问题 → 不再重复回答
* 未回答的问题 → 追踪着
* 关键决策 → 保留

压缩完还是那回事，不丢东西。

 

## 一句话总结

**OpenClaw 的记忆是被动记录，Hermes 的记忆是主动召回。**

前者等你犯错后记下来，后者在你说话前就查好了。

不是智商差，是上下文完整度差。
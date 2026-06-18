---
title: 看完Anthropic的blog，我发现他们也在干同样的事
author: 萤火序
date: 萤火序萤火序
url: https://mp.weixin.qq.com/s?__biz=MzY4NTMwNzY1NA==&mid=2247484108&idx=1&sn=f9d4abe099d00a809b1f4732c0e7e998&chksm=f25dc8157979c9e298d891a451055723cf01a7f3f7b072c1dbb08fa1d60d2ef35e9be8a5c31a&mpshare=1&scene=24&srcid=0604MTX9DNCZPcOhweohAAvR&sharer_shareinfo=50f84fbaefa706e3be8ceb64301e53e4&sharer_shareinfo_first=50f84fbaefa706e3be8ceb64301e53e4#rd
---
> 已吸收至：[[01_LLM与大模型/0101_模型能力/0101_核心知识点/模型能力来源校准与跨域路由准则|模型能力来源校准与跨域路由准则]]

# 看完Anthropic的blog，我发现他们也在干同样的事

今天读到一篇Anthropic的blog，Claude Code的工程总监Fiona Fung写的，讲他们团队怎么用AI重新定义工程管理。

看完我第一反应是：原来大公司也在想同样的问题。

## 她在说什么

Fiona说他们团队现在写代码、写测试、重构，几乎全靠Claude完成。4个月没见纯手工提交的代码了。

但她没怎么聊AI写代码有多快。她聊的是另一件事：效率提升之后，原来的流程全废了。

以前写半年roadmap，三个月就过时了。以前代码有问题找写的人问，现在先问Claude。以前所有代码人审，现在Claude审风格和bug，人只审安全和法律。

最让我有感触的是她那个例子：她以前每天早上手动整理用户反馈，后来让Claude自动跑。一个手动流程就这样没了。

这不就是我们一直在干的事吗？发现一个重复的手动流程，想办法自动化掉，然后把时间花在更有价值的事上。

## 她最狠的一条原则

她说了三条原则，第三条最狠：

"任何不再有效的流程，团队成员有权直接砍掉。"

她举了个例子。之前有个周会，每个人都低头看手机，轮到谁说谁抬头说两句。她问了一句"这个会还有必要吗？"然后会就取消了。

这个场景太真实了。多少公司的多少会议，就是这样的状态？每个人都知道没用，但没人敢说取消。

我觉得这跟自动化是一回事。自动化是"这事重复太多次了，让它自己跑"。砍流程是"这事已经没意义了，直接停掉"。两个动作解决同一个问题：别在没价值的事上浪费时间。

## 她给大家的一个建议

"Ask yourself: what's one piece of your engineering workflow that you might consider automating or even dropping altogether?"

问问你自己：现有工程流程中，哪一步可自动化改造，亦或直接剔除

我的答案很简单：当一件事需要重复超过3遍，就该想尽一切办法自动化掉。不是因为人懒，是因为人的时间太贵了。

数据来源：Anthropic Blog · Fiona Fung

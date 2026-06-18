> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 震惊！AI Assistant写代码，竟完成了100%的Coding工作，引发编程界疯狂议论！
author: 程序视点
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg2NTg5NDI5Mw==&mid=2247488448&idx=1&sn=c4c5282e4e2d13c851893dd5d4f69767&chksm=cf6d119b8364c5f19027c25000e099c848e3e66b3772c1e419781aeaddab449f67f2f41d4305&mpshare=1&scene=24&srcid=0910iJA4eEIxdsWbIxxqaVvF&sharer_shareinfo=7d07406a38b4edd8e26c8221c64081f7&sharer_shareinfo_first=7d07406a38b4edd8e26c8221c64081f7#rd
---

点击蓝字 关注我们

将程序视点设为星标精品文章第一时间阅读

**大家好，欢迎来到`程序视点`！我是小二哥。**

### 前言

昨天我们详细分享了`AI Assistant激活和简单使用`。

[重磅消息：JetBrains全家桶AI Assistant激活震撼上线，立即体验AI Assistant编程时代！](http://mp.weixin.qq.com/s?__biz=Mzg2NTg5NDI5Mw==&mid=2247488346&idx=1&sn=4f7ad95f675d69f8c94ac52b3f1ae96f&chksm=ce5279a0f925f0b6ec83e0c9a0692b3e3d12188a12a860c0258389e621c4c343cb284090fa5b&scene=21#wechat_redirect)

今天，我们用`AI Assistant`来解决下实践中的问题--AI Assistant写代码。

AI Assistant写代码

这是2023年某高校技能大赛的题目之一。

AI Assistant写代码

大家自己评估一下，写出这道题需要多久？

### AI Assistant来做题

现在我们用`AI Assistant`来帮我们解决这个题目。

因为是计算月份的天数，我新建了一个java类`DaysOfMonth`。

AI Assistant写代码

接着，打开`AI Assistant`的代码生成功能。

把题目直接告知`AI Assistant`后生成了这样的代码。

虽然功能实现了，但这不满足我们题目的需求。题目要求是**根据输入的数据进行判断**。

那首先要实现输入的问题。这里我把它做得更具有交互性：**实现控制台进行输入**。

点击`Specify`把我们的需求再补充下：**年份和月份的输入是通过控制台实现的**

AI Assistant写代码

接着，`AI Assistant`生成的代码就又补充了一部分。

这还不够。题目要求有对数据的判断。于是，我们再点击`Specify`，输入：**年份和月份在控制台输入后要进行有效值判断。无效的年份或月份，需要提示重新输入**

AI Assistant写代码

然后，`AI Assistant`完善后的代码是这样的。

AI Assistant写代码

两个`do...while`循环处理了判断和重新输入的问题。非常的棒！

到这里，小二哥觉得这时的功能应该齐全了。但这还不是一份能拿得出手的答案

目前的程序和题目的要求还不完全贴合。**题目是需要单元测试的。**

怎么办？让`AI Assistant`继续帮我们吧！

选择方法，右键选择`AI Assistant`，找到`Generate Unit Tests`并点击。

嗖~嗖~ 得到这样一份单元测试用例！

AI Assistant写代码

**闰年，非闰年；大月31天，小月30天；** 每种可能的测试都有啦！选择`Accept all`就好了。

嗯~嗯！小二哥觉得可以提交这份答案了

### 最后

大家可以自己先做一遍这个题，简单计算下时间。然后，再按照上面的步骤，用`AI Assistant`来实现。最后对比下两者的效率！

这份答卷中，`AI Assistant`帮我们完成了100%的Coding工作，而我只需要专注业务逻辑和功能就好！

大家想一想自己开发的业务功能，**`AI Assistant`能帮你完成多少Coding？能节省多少时间？能提高多少效率？**

目前`AI Assistant`在JetBrains官方的价格是**10美刀一个月**！

JetBrains官方订阅的成本是非常高的！现在，小二哥**为专属付费版全家桶用户提供了`AI Assistant`优惠激活服务，最低只需10RMB/月即可开通`AI Assistant`服务！**

关注微信公众号【程序视点】，回复：`ai`, **最低10RMB/月激活`AI Assistant`插件助手**。

后续小二哥会继续详细拆解`AI Assistant`的各项功能。大家可以把微信公众号【程序视点】设置为星标，这样就不会错过之后的精彩内容啦！

如果这篇文章对你有帮助的话，别忘了【在看】【点赞】支持下哦~
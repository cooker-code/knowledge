> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020506_workBuddy/020506_核心知识点/workBuddy协作流程与工具边界|workBuddy协作流程与工具边界]]
---
title: 三步搞定：让WorkBuddy直接操控你的浏览器
author: AI教员
date: MR.WMR.W
url: https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247485062&idx=1&sn=8aaebf568a52c805159f222e2e022903&chksm=f2d9cf6910d12c421852a9e9ade9ff37dc8451bc246826ff5af01d20cb88ebdc200b851b6388&mpshare=1&scene=24&srcid=06033I9zankAaeeGB0bD7ZWF&sharer_shareinfo=671e3ca723402de84717b528e161aa88&sharer_shareinfo_first=671e3ca723402de84717b528e161aa88#rd
---

---

Codex之前出了个浏览器插件，但得订阅升级版才能用，我一直在找平替。

就在前几天，Kimi发布了**Kimi WebBridge**，我第一时间装上试了试，体验确实不错，今天就分享给大家。

## 装浏览器插件

先去官网：https://www.kimi.com/zh-cn/features/webbridge

点手动安装，先下载浏览器插件。为什么不直接去Chrome应用商店装？因为我用的是Chrome浏览器，访问应用商店得挂魔法，手动装反而更省事。

下载插件

嫌麻烦的话，也可以直接用这个地址下载：https://kimi-web-img.moonshot.cn/webbridge/latest/extension/kimi-webbridge-extension.zip

下载好之后，点浏览器右上角的菜单按钮，选择**设置**。

设置

进去之后找到**扩展程序**，点进去。

扩展程序

到了扩展程序页面，先把右上角的**开发者模式**打开。

开发者模式

然后把刚才下载好的压缩包，直接拖到浏览器页面里就行了。装成功的话，在插件列表里能看到Kimi WebBridge。

再点一下浏览器右上角的插件图标，把Kimi固定到工具栏，方便以后用。

固定插件

## 连接WorkBuddy

插件装好了，接下来就是让Agent端跟浏览器对接上。我用的是WorkBuddy。

Mac用户复制这条命令：

```
curl -fsSL https://cdn.kimi.com/webbridge/install.sh | bash
```

Windows用户复制这条：

```
irm https://cdn.kimi.com/webbridge/install.ps1 | iex
```

复制好之后，直接粘贴到WorkBuddy的对话窗口里，它会自动帮你装。

安装成功

装完之后，回到浏览器点一下Kimi插件，看到"浏览器助手已就绪"，就说明WorkBuddy和Kimi已经连通了。

连接成功

## 测试一下

这步很重要——在WorkBuddy里输入：**测试Kimi WebBridge连通性**。

WorkBuddy会跑一遍连接测试，同时自动安装Kimi WebBridge相关的Skill。

测试连通性

装完之后WorkBuddy会问你要不要把Skill复制到本地目录，当然是允许。

安装成功

然后我选了**kimi-webbridge**技能，发了句：**kimi-webbridge帮我打开今日头条**。

打开浏览器

浏览器会自动弹出来，顶部还有提示说插件正在调试此浏览器。接下来你要做什么就看你的想象力了。

到这里，整个连接流程就走完了。说实话操作还挺简单的，装个插件、跑个命令、测一下连通性，三步搞定。

如果你也在用WorkBuddy，又想让AI帮你操作浏览器，这个东西确实值得试一下。

以上，既然看到这里了，如果觉得不错，随手点个赞、在看、转发三连吧。

 

**往期精选：**

* [我把女朋友"蒸馏"成了一个AI](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484264&idx=1&sn=1a2da546d917106c65be1bd6cc2b3517&scene=21#wechat_redirect)
* [国内 OpenClaw 产品全景对比](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247483767&idx=1&sn=dabfb07cbb4a6f847517f889392deb86&scene=21#wechat_redirect)
* [国内大模型申请API Key攻略合集](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484456&idx=3&sn=1cd7404cf78c34e4181ea5903d7abf32&scene=21#wechat_redirect)
* [不会装 Skill？你只用了 WorkBuddy 的一半](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484082&idx=1&sn=c81475ba751e4b6f36bf6f7c6ed9947c&scene=21#wechat_redirect)
* [Hermes还没学明白，Generic Agent又来了](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484575&idx=1&sn=94bc4e6a65112b3fff2f81f8d90db2e8&scene=21#wechat_redirect)
* [腾讯推荐的 Skill 你装了几个？这 10 个值得收藏](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484190&idx=2&sn=79c4c610304077b34d3e162375291bdf&scene=21#wechat_redirect)
* [一只龙虾变多只：用飞书打造腾讯系多Agent协作](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484145&idx=1&sn=cbab8fb31a3ce1b3b6725e0a22c8b727&scene=21#wechat_redirect)
* [让AI秒变专属助手：我用Skill把重复工作全部自动化了](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484257&idx=1&sn=72652379263ae5a561128dee454e4f7b&scene=21#wechat_redirect)
* [元宝派 + WorkBuddy = 随时随地远程办公，内附邀请码！](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484242&idx=1&sn=2336ccb608f3508c3b4589cf4038a6a4&scene=21#wechat_redirect)
* [手机就是你的AI办公室！WorkBuddy微信小程序保姆级教程](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484332&idx=1&sn=762e4618d41949f40881d35ce8b1bd3e&scene=21#wechat_redirect)
* [Windows本地部署Hermes Agent，微信扫码即用（附完整教程）](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484367&idx=1&sn=95d6caf68c7d26cba9cbbd5b3619f666&scene=21#wechat_redirect)
* [【省到笑】WorkBuddy积分与Token换算内幕曝光！1积分≈3万Token，真相让人意外](https://mp.weixin.qq.com/s?__biz=MzY4NTE5Mjg0NA==&mid=2247484539&idx=2&sn=f1b56d6ce9aa918ce6f0ac76375d00b5&scene=21#wechat_redirect)
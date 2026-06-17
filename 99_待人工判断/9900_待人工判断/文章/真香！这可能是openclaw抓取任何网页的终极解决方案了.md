---
title: 真香！这可能是openclaw抓取任何网页的终极解决方案了
author: 芝麻AI智能体
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487523&idx=1&sn=ceca09f74ad5f1562eac71173df34133&chksm=c5f9c081bef6dfe2a7196593979b86ad7b7dee2cb9dae5937bae913f8d29fa8670e8ada5abc1&mpshare=1&scene=24&srcid=04177wnvCWhAKq7wMlfQdD8P&sharer_shareinfo=5c4418d21b49e6b02c3ad0a0468538b9&sharer_shareinfo_first=5c4418d21b49e6b02c3ad0a0468538b9#rd
---

### ☀大家好，我是芝麻☀

### AI智能体，AI副业搞米，AI实战案例分享

****👇点击关注，每**篇文章能给你带来一定的收获**👇****

你好，我是芝麻！

这几天在研究怎么让我的小龙虾可以访问并操控浏览器，就找到了这样一款工具

并且你可以通过任何支持MCP或者Skills的智能体去使用这个工具，比如ClaudeCode、Openclaw等，只需要按照工具官网的教程配置到你的Agent中就可以使用了。

小红书、公众号、CSDN、知乎等这些都可以抓取。这个工具就是Dokobot。

Dokobot简介

这是一款仅支持Chrome浏览器接管与自动化工具，核心是让AI agent/命令行直接操控本地Chrome，保留完整登录态，实现网页操作、数据抓取、截图、表单填写等自动化操作。

核心能力

1️⃣ 页面控制：导航、等待元素、刷新、前进后退

2️⃣ 数据提取：获取DOM、HTML、文本、图片链接

3️⃣ 交互操作：点击、输入、提交表单、滚动、执行JS

4️⃣ 截图/录制：全屏/区域截图

5️⃣ AI集成：可被MCP/AI Agent直接调用，实现自然语言->浏览器操作

被openclaw集成后，整个的操作链路就是

> IM->OpenClaw->Dokobot->Chrom Dev->浏览器

别看操作路径很简单，Ai Agent联网其实障碍重重

Dokobot突破这些障碍的方式就是，用你自己本地的浏览器，Dokobot让AI Agent通过你的Chrome浏览网页——你的会话、你的权限、零泄露

安装Dobokot

官网链接直达：https://dokobot.ai/zh-CN

Dobokot有两种安装方式：Chrome扩展、Dokobot CLI

Chrome扩展是只读的，安全第一，并且节省token

Dobokot CLI可以让你完全控制浏览器

大家根据自己的需求按需安装，本文先将Chrome安装的方式和使用方法，DobokotCLI的方式之后再出文章讲解

第一步，安装Chrome插件

可以直接在Chrome Web Store安装这个扩展插件，由Chrome自动更新

如果没有科学上网工具，也可以直接下载插件的压缩包，导入chrome安装

安装完插件后，会让你登录

第二步，插件开启远程控制

打开刚刚配置的插件的配置面板，为你的Doko起个名字后认领

开启远程控制

第三步，创建密钥

接着会让你创建密钥，给密钥起一个名字然后创建即可，记得复制出来，它只显示一次

第四步，安装Skill

直接将其复制，发送给你的小龙虾，并且告诉它你刚刚复制的密钥是多少

第五步，选择FREE方案

大概率会弹出这个订阅方案，直接选择免费的就好了，后续需要的时候再升级套餐也不迟

测试案例

✅微信公众号

✅飞书文档

这个出现了滑铁卢，一开始给我说没法抓取

然后我让他尝试一下，就成功了

✅小红书链接

非常不错，一次成功

✅抓取小红书openclaw的热帖

> 使用Doko访问小红书，采集20篇关于openclaw的热帖

不戳不戳，也成功给我汇总了出来

总结

再来说一下安全问题，这是一个永远绕不开的话题，来看看官方是怎么说的

Dock的原理就是在你本地的浏览器里面”浏览“网页，只会在你主动用的时候工作，并且浏览器是有提示的，和传统爬虫的工作原理是不一样的

敏感页面自动避开，你的会话、权限、cookie等信息都还是保留在本地，所以不用担心泄露的问题。

---

本期的内容就到这里了，感谢你的耐心。

如果你有智能体定制，合作，学习智能体，学习智能体变现等需求，也可以找我。

跟着文章的步骤实操，实现了这次的工作流，可以把结果放在评论区和大家分享！

做的过程中遇到了问题也可以评论区留言，我会为大家解答！

看完喜欢，请帮忙转发分享一下，你的点赞转发，就是我更新下去的动力！

[最新消息！别再去研究部署openclaw了！腾讯已经甩出了王炸Qclaw，直接在微信就可以用，完全零门槛](https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487493&idx=1&sn=661da2e7c6cba259740a2c6de7634caa&scene=21#wechat_redirect)

[汇总一下，Openclaw国内各大云端和本地部署养虾方案，总有一款方案适合你！](https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487456&idx=1&sn=0249bef153337a1a75d24bd4293ea814&scene=21#wechat_redirect)

[果然...强者改变世界，弱者适应环境，百度和腾讯相继给龙虾打钱！！](https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487465&idx=1&sn=c6e348ddd1ca076e76a9acf1a530945f&scene=21#wechat_redirect)

[Windows本地化部署OpenClaw全流程，超级详细！！小白慎入！！](https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487427&idx=1&sn=8af797fc92689b1eed9a9d09f3b9c611&scene=21#wechat_redirect)

[疯了。。字节来收割散养「龙虾」的用户了。。一键免费部署你的龙虾，三十秒搞定。。。](https://mp.weixin.qq.com/s?__biz=Mzk2NDAxMzE0NQ==&mid=2247487357&idx=1&sn=bbe36f132ac709e763fab8c7df77010a&scene=21#wechat_redirect)
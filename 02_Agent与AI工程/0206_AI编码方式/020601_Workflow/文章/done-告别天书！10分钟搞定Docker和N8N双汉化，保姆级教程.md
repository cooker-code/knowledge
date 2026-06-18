> 已吸收至：[[02_Agent与AI工程/0206_AI编码方式/020601_Workflow/020601_核心知识点/Workflow编排与验证闭环|Workflow编排与验证闭环]]
---
title: 告别天书！10分钟搞定Docker和N8N双汉化，保姆级教程
author: 二师兄聊AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzkxMzg4MzIwNg==&mid=2247487673&idx=1&sn=c5b0f52f024f9eaf72126711820a1037&chksm=c0a291011490e99b5f4d0791e5dbc7f24e8e7139fe52f9c2b611fd6d93e83f02d411fb941678&mpshare=1&scene=24&srcid=09259IANmSEgIzf2BgDwR7eM&sharer_shareinfo=b5e36a93a02cdc704dc3704ea939072a&sharer_shareinfo_first=b5e36a93a02cdc704dc3704ea939072a#rd
---

大家好，我是二师兄，程序员，宝爸，短视频 AIP 教练， 专注 AI 工具、AI 智能体、 AI 编程。

踏上取经路比抵达灵山更重要，干起来，再完美。

上一篇二师兄写了 如何在windows上基于docker安装n8n, 文章中也留了一句：

[n8n新手看过来！n8n windows docker化部署，喂饭级教程](https://mp.weixin.qq.com/s?__biz=MzkxMzg4MzIwNg==&mid=2247487645&idx=1&sn=9954b7a819ea5250c30459214e21178d&scene=21#wechat_redirect)

> 如果看不懂docker desktop的英文，看不懂n8n的英文，小伙伴可以留言，会考虑再出一篇汉化教程。

果然有小伙伴留言了，那咱就安排上。

主要分两块来讲：

1. docker客户端的汉化
2. n8n的汉化

# 一 docker desktop 汉化

docker的汉化在github上也有教程

直达地址

https://github.com/asxez/DockerDesktop-CN

首先我们要先确定docker desktop的版本

首先点击右上角的齿轮图标， 再点左侧的“Software updates”， 我们会看到对应的版本号。

这里二师兄安装的版本号是 4.44.3 。

接下来打开刚才直达的地址， 选下图位置

找到跟docker desktop版本对应的包，下载这个aar文件；如果你docker desktop 还没装，就直接下载两个文件。

然后你需要**关闭你本地的docker desktop** ， 打开 “C:\Program Files\Docker\Docker\frontend\resources” 目录。

将原来的app.asar 重命名为 app.bak（相当于备份一下，万一待会出问题，咱们也好回滚）

然后拷贝刚才下载的 asar 到这个目录， 改名为 app.asar。

重新打开docker desktop， 就会看到已经完成汉化了，**是不是非常简单**？

# 二 n8n汉化

基于 docker 如何安装n8n ，如果你还不清楚， 可以看上一篇文章。

[n8n新手看过来！n8n windows docker化部署，喂饭级教程](https://mp.weixin.qq.com/s?__biz=MzkxMzg4MzIwNg==&mid=2247487645&idx=1&sn=9954b7a819ea5250c30459214e21178d&scene=21#wechat_redirect)

n8n 汉化 在github 也有开源项目，直达地址

https://github.com/other-blowsnow/n8n-i18n-chinese

这块一定要注意的一个点是： **n8n汉化包的版本，一定要与n8n版本一致。**

首先先看n8n的版本，先点击“镜像”，再点击n8n

往下翻， 找到 “ARG N8N\_VERSION” , 二师兄这里是 1.108.2

然后我们打开汉化地址， 点击这个 Rleases

下载对应n8n版本的汉化包

然后解压到一个目录（看你自己喜欢），最终是一个 dist 目录

然后打开 docker desktop, 找到 n8n的镜像，点击 运行按钮 ，依次填入以下信息

```
容器名称 ：  n8n_hanhua
端口： 5678 
卷： 总共有两个

第一个左边是你备份n8n数据的地方  第二个是n8n容器的路径
比如我的是D:\200_agent\n8n\n8n_data，  n8n容器内路径是/home/node/.n8n

第二个左边是刚才解压的汉化包路径  第二个是n8n中ui的路径
比如我的是D:\200_agent\n8n\hanhua\dist， n8n ui路径是/usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/dist

最后两行是添加两个环境变量，照图填写即可
```

点击运行，运行成功会看到这样一段日志

点击这个链接，就可以直接在浏览器中打开n8n, 可以看到，我们已经汉化成功了。

# 写在最后

最近的几篇文章，把n8n的安装和汉化都已经涵盖了，几乎就是保姆级文章了， 哪怕你是小白，照着操作肯定可以完成。

n8n的学习，从搭建环境开始。快去行动起来吧。

好了，本文到此结束了，如果粉丝朋友还有什么不明白也欢迎留言，或者有其他想了解的也可以写在评论区， 二师兄后续会纳入选题库。

都看到这里了，帮忙来个一键三连吧~~ 蟹蟹

二师兄为了回复广大粉丝的厚爱，也给大家带来两个福利，有两个训练营大家可以低价参与（具体海报可以看文章次条）：

1⃣️《10天AI考证速通训练营》开营时间8.30号，也就是明天晚上9：18

10天内拿下25个权威互联网大厂的AI证书，包含阿里、百度、亚马逊、微软、波尔、领英、字节(豆包)、商汤、支F宝、科大讯飞等众多大厂的25个AI证书。 ¥49.9

2⃣️《15天Coze变现特训营》开营时间 9.1号晚  20：15

特训营教练均为斑码AI百万商单团队成员，通过AI智能体变现超过5位数。

15天时间，带你从小白到智能体入门。

特训营结束，你将掌握智能体的基本操作方法、Coze变现实战，并有机会接受斑码商单团队派单。  ￥66

二师兄
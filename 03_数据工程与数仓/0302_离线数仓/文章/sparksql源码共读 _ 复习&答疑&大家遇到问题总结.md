---
title: sparksql源码共读 | 复习&答疑&大家遇到问题总结
author: 数据仓库践行者
date: 
url: http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485049&idx=1&sn=83449b3544e1e02d8a085f8391af4722&chksm=fe6c5766c91bde70a1d4ffa3cd76ab5602733ee593dffa00f6d21caf90bd82276dbe2cfbdcd8&mpshare=1&scene=24&srcid=0409ZUWMigbyfDbRhqhKZ1wq&sharer_sharetime=1649500313538&sharer_shareid=ac11efa86a18b42da87220879062c874#rd
---

sparksql源码共读进行三次了，上周手把手debug源码，快炸裂了，这周安排一次复习，总结一下大家遇到的问题。

不足之处，不讲干货的时候，发现自己说话特别啰嗦，说了很多【嗯，额，然后，然后，然后的话，这样的话】......这样的连接词，这个得加强练习！多录视频，多练习怎么讲话，相信一定会有进步

最近录了一些sparksql常被面试到的知识点，从源码层面做分析的视频，后面会坚持录下去。（链接放在文末了）

希望通过这些视频可以帮助大家在面试时能够讲出更多有深度的东西；也希望大家在日常工作中，多注意源码层面的积累。

# **心态**

# 如果对自己现在的状态不满，一定要知道现在是过去的积累，要想以后过的好，必须要从现在开始慢慢积累。

# 勤奋不是争分夺秒，而是日日不断，滴水穿石。

# 持续行动，心态最重要。

# 保持好状态，我们才可能精力充沛、充满热情的持续去做一件事，最终获得我们想要的结果。

# 每天进步一点点，

# 生活、工作都会变得越来越好。

# **装环境**

## **1、版本问题**

Spark branch 3.2

这个在分享时，提供了我安装的版本，但还是有小伙伴会有版本问题，比较标准的做法是看spark源码的pom文件：

源码环境：maven、java、scala、antlr

如果要搭建集群，也可以参照这里的版本

## 

## **2、编译过程中各种莫名其妙的错误**

### 

### **清理原有环境**

（开始时电脑越干净，越可以减少遇到问题的概率）：

* 之前拉过spark源码，最好删除了，再重新弄一遍

* 电脑装过java、maven环境，但版本不一样，最好清理一下，再重新装

* 环境变量设置

### 

### **网络环境**

* 能用加速的，最好用个加速装置
* maven的配置文件（conf/setting.xml）里配置阿里云镜像

```
```
<mirrors>      <mirror>      <id>nexus-aliyun</id>      <mirrorOf>central</mirrorOf>      <name>Nexus aliyun</name>      <url>http://maven.aliyun.com/nexus/content/groups/public</url>      </mirror></mirrors>
```
```

* 把没有用到的模块注释掉

这个也是不得已的办法，如果能编译还是都编译一下，大部分小伙伴还是编译通过了的

* 多编译几遍
* 换个地方，比如平时是在家，换到公司试一试。

  群里有小伙伴搬了家之后，编译通过了

* 我把我这边的maven仓库的jar放云盘了，如果有需要可以下载（但感觉这个作用不大）

   链接: https://pan.baidu.com/s/1t3kAaPiJ14iTmGqaU26vgw  密码: ujtu

### **报格式类的错误，可这样消除掉**

**方法一：**

在报错的地方点进去

**方法二：**

# 

# 

# **antlr**

展示asttree时，sql语句需要大写

# 

# **调试**

## **1、测试类**

XiaoluobuSuite 与PlanParserSuite 放在同一个目录下

XiaoluobuSQLQuerySuite 与SQLQuerySuite放在同一个目录下

## **2、debug**

在需要重点看的代码处打断点：

在测试类里选debug：

# 

# **其他**

* 重启电脑
* 重启IDE
* 多装几遍，今天搞不定了，干点别的事，等过两天再来试

福利来啦~~

最近录了一些sparksql常被面试到的知识点，从源码层面做分析的视频，后面会坚持录下去。

希望可以帮助大家在面试时能够讲出更多有深度的东西；也希望大家在日常工作中，多注意源码层面的积累。

有需要的朋友可以去看啦：

B站链接：https://space.bilibili.com/474826421

---

视频号也同步啦：

---

欢迎加微信：

Hey!

我是小萝卜算子

欢迎关注公众号

每天学习一点点

知识增加一点点

思考深入一点点

在成为最厉害最厉害最厉害的道路上

很高兴认识你

**推荐阅读：**

[面试 | 你真的了解count(\*)和count(1)嘛？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485023&idx=1&sn=1c2ca45b4c310d391eaf5e0e1b7533b1&chksm=fe6c5740c91bde563bafa2fb9193e3b881a095bae7431c25da2ebe9959c7820cc496ef979ab6&scene=21#wechat_redirect)

[以后的事谁也说不准](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247485016&idx=1&sn=2fe05cee0c69a8b4ae9b026132f8827c&chksm=fe6c5747c91bde5113c93c9f16e839f7b5d8738b858e9171a5f8e86e50bf28538ae0a85a7495&scene=21#wechat_redirect)

[澄清 | snappy压缩到底支持不支持split? 为啥？](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484994&idx=1&sn=2b7f7ac375517d34759412699f322133&chksm=fe6c575dc91bde4bf6ccbfcb1763a191e8b804b4e37ebd809a08f1072b864530b06dc607b8b0&scene=21#wechat_redirect)

[转型【数仓开发】该怎么学](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484695&idx=1&sn=d0b793b6c2a09e048ff9a21f4b4ee1d0&chksm=fe6c5408c91bdd1ebddde0e9e1777d2afd755c250d15877c79b94eae1e2009d31080df18b16f&scene=21#wechat_redirect)

[大数据开发轻量级入门方案](http://mp.weixin.qq.com/s?__biz=MzU5NTc1NzE2OA==&mid=2247484678&idx=1&sn=a13955f253d989e728685e8c79513cb8&chksm=fe6c5419c91bdd0f4af36413a08a29faf054b20a2c2011c2720b0df37d0ae768b7af8030713c&scene=21#wechat_redirect)
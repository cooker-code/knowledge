---
title: Obsidian同步gitee，轻松搞定双同步
author: 大米小麦 嘻哈
date: 大米嘻哈小麦大米嘻哈小麦
url: https://mp.weixin.qq.com/s?__biz=Mzg3MzcyMjMxNA==&mid=2247485822&idx=1&sn=69fbbba510bae3955d7a51eacd8a0fb3&chksm=cf00ea1cbb374ef811769a47f0540943d64cdfa5b4b1b84647c42923373d6b04cd85af4f9cbd&mpshare=1&scene=24&srcid=05161VTIMPLQQrDP2oVI5owp&sharer_shareinfo=e0121f96dac6134d7ce576dc065ec84d&sharer_shareinfo_first=e0121f96dac6134d7ce576dc065ec84d#rd
---

https://gitee.com

这个网站进去，全中文，建库设令牌轻松搞定

**注意：1、初始化**不要勾选“使用 Readme 文件初始化仓库”，保持空仓库状态

2、令牌环节只勾选 `projects` 权限（至少需要推代码的权限）

然后在Obsidian已经能同步Github的基础上，新增加这个gitee仓库就行啦。（此处注意下，gitee仓库名/用户名这些哪怕是大写，在命令行也要改成小写，不然成功不了，我卡在这里试了N遍才知道）

执行如下命令：

git remote -v（会出现双地址）

git remote add gitee https://gitee.com/你的用户名/你的仓库名.git（红字换你自己的gitee用户名跟仓库名）

git push -u gitee main

当然也可以直接用你的令牌推送：

git push -u https://你的Gitee名:新令牌@gitee.com/你的Gitee名/你的仓库名.git main（红字换你自己的信息）

随便改个笔记去Obsidian里执行验证：`Ctrl + P` → 输入 `Git: Commit and sync`

`两个仓库里分别去看下，同步成功！`

`哈哈哈，Github的qiang问题一劳永逸的解决啦！`

`附Obsidian同步Github的步骤链接：`

`Obsidian同步GitHub，get！`

`一路顺畅的Github，挫折重重的Obsidian`
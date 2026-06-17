---
title: n8n重大更新：自带数据表，让你的自动化流程快到飞起！
author: 鲤鱼冲啊
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485539&idx=1&sn=1912e7abb1efbd8ed94ff74a9ef5f3e7&chksm=c17dbb651bd1cc12fed9f8ce7985d6d5d628d7af7e0d4992fac9a7ce22cc927607717729a5dd&mpshare=1&scene=24&srcid=1029fvkGjJesROVYuTexdNdA&sharer_shareinfo=58ec3d2ac9db77815672f533d612712c&sharer_shareinfo_first=58ec3d2ac9db77815672f533d612712c#rd
---

这几天打开n8n工作流，可以看到这么一条更新提醒：新版本（≥1.113.0）可以直接在n8n内部存储和查询数据表，无需设置外部数据库，也就是说存储表格数据可以不用通过飞书多维表格或Google Sheets。

好处是什么呢？我将官方视频用NotebookLM快速总结成思维导图（关于NotebookLM用法，详见[不是我吹，用上这款Google免费神器，你的学习效率至少提升300%](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247484740&idx=1&sn=0dd339dacb5f6104c078529d1c4698fc&scene=21#wechat_redirect)），可以看到，使用内部数据表的情况下，访问和写入速度大幅提升。

①如何升级n8n

以zeabur为例（部署详见[给你的n8n安个五星级的家](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485479&idx=1&sn=ca42ea992ab691d9dde84fd33aceaa25&scene=21#wechat_redirect)），登录后点击顶部的仪表盘

选择自己的n8n项目

点击左侧n8n，选择设置，按下图示例修改版本号，目前最新版是1.114.0，点击Save。

服务就开始重启了，等待一会后可以刷新页面，看是否启动成功。

如果是hugging face部署的，升级参考这篇（[n8n如果遇到一直连接失败问题，可以尝试重启空间或降低到次新版本解决](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485135&idx=1&sn=d6d68cbc6fd2e5c4c6d9ec964fd0ffae&scene=21#wechat_redirect)），版本号记得改为1.114.0

②如何使用Data tables

可以看到Overview页面新增一个标签：Data tables，可以从右上角创建工作表处点击下拉箭头，创建数据表。

填写表名

这里createdAt和updatedAt是固定字段，无法删除，通过右侧加号新增列。

我这里新增两列，date和information，Type都选string

最终表格如下

将之前跑通的AI资讯自动写入飞书表格工作流复制到一个新工作流（参考这篇：[让AI帮你打工！n8n将AI资讯自动写入到你的飞书表格](https://mp.weixin.qq.com/s?__biz=Mzg5ODU1NjQ3MA==&mid=2247485195&idx=1&sn=2ae9e47bdcb5c57401b080ef973f304c&scene=21#wechat_redirect)），并删除飞书节点，如下图所示（附本文完整工作流链接，https://pan.quark.cn/s/79cf6381909c，注意，需要先升级好版本，创建好数据表，填好列名，节点选上对应表格，否则执行会报错）

在Edit Fields右侧新增一个节点，搜索data，选择Data table，

选择Insert row

参考下图配置

最终工作流如下图

将If节点的时间改为2025-09-29（因为时区的问题，选昨天比较保险）

点最下方Execute workflow执行工作流，显示成功，来看看数据表，数据成功写入了。如果没看到，需要刷新下页面。

③其它细节

每个数据表的存储空间上限为50M，目前不支持存入图片和视频。

数据表的列名目前不支持中文。

如果想通过数据表直接删除数据，可以左侧先打钩，然后下方会显示删除按钮。

好了，今天的内容就到这里，觉得有帮助的话，点赞收藏关注，我们下期见！
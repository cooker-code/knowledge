---
title: Obsidian+Claude联动教程：3个插件装完，你的笔记库瞬间变成私人AI大脑
author: 后端技术互联
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkwNDY1NzUxNQ==&mid=2247488058&idx=1&sn=a17939eb765d88c713894a62308cf476&chksm=c1c79e9d18ec6ea73d5d2e92adb7095614318acb936d8984f4df92010b2185842d96b8d7c76c&mpshare=1&scene=24&srcid=042281aj5uiDdvN8yibk3Q6B&sharer_shareinfo=e936ac63061a7a611551fcdaff4227b1&sharer_shareinfo_first=e936ac63061a7a611551fcdaff4227b1#rd
---

\* 戳上方蓝字“**后端技术互联**”关注我!

大家好，我是技术Z先生，一名热爱分享的程序员！

上篇文章[语雀卡成狗，飞书无法导出markdown格式，我花3天找到了这款最强本地知识库，支持图片自动上传还完全免费](https://mp.weixin.qq.com/s?__biz=MzkwNDY1NzUxNQ==&mid=2247488031&idx=1&sn=0ccdf2ea43ed80261d10b5794ce2651d&scene=21#wechat_redirect)讲了如何安装本地知识库工具Obsidian，以及如何实现自动上传本地图片到公共图床，今天继续分享下如何让它与Claude进行联动！联动的好处也是很多，你使用Claude桌面版或者Claude Code写任何东西都能让它MCP拉起Obsidian读取本地知识库并进行知识库内容修改，接下来我讲详细讲解实操步骤！

## 一、安装MCP Tools依赖

这里我们去第三方插件市场（还不知道第三方插件市场在哪里的可以参考我上篇文章查看，里面有详细讲解），搜索MCP Tools并安装进到MCP Tools插件界面，要把Obsidian当作MCP服务运行，需要先在本地安装Local Rest API（给 Obsidian 开一个本地 HTTPS 接口）、Smart Connections（语义搜索能力）、Templater（让 Claude 调用你的模板）这三个依赖

### Local Rest API安装

因为要暴露对外接口直接操作本地的MCP，所以这里需要配置Local Rest API，通过安全的身份验证 REST API，为您的脚本、浏览器扩展和 AI 代理提供直接访问您的 Obsidian 库的通道。

这里安装local rest API也很简单，无需按照上面截图的github页面去执行一连串的代码指令，直接进到第三方插件市场，搜索Local Rest API点击进去后直接启用就可以了进到Local REST API配置中心，可以看到它提供了一个本地对外的接口服务，供给其他AI通过接口直接操作本地Obsidian，默认提供的端口是27124进到Mac终端，输入：lsof -i :27124，也可以看到终端被Obsidian监听占用着

### Smart Connections安装

继续进到第三方插件市场，搜索Smart Connections，点击进去点击安装并启用启用后会弹出个操作指引，这里有兴趣的可以慢慢看下，我们直接点击右上角关闭掉点开Smart Connections设置中心可以看到里面，里面还很多配置，这里直接使用默认的就行，Smart Lookup也不需要安装（这个是一个独立插件，在笔记里面可以选中你的一段话后全部比较查找相似词，我们这里用不到），Smart Connections启动后默认会为笔记生成向量索引 vault装这个插件之后，Claude 就能直接提问"我写过和 XXX 相关的笔记吗"这种模糊问题

### Templater安装

templater可以让Claude 自动调用你的笔记模板（比如"按日报模板生成今天的日报"）

进到第三方插件市场，输入templater点击搜索找到templater后点击安装并启用在Obsidian里创建个templater文件夹，专门用来存放你自己独有的模板，再在Templater设置里指定模板文件夹位置

## 二、启动MCP Tools Server

回到Mcp Tools插件设置中心，点击Install server按钮进行服务安装这里如果出现API Key没有配置情况，可以回到Local REST API配置中心检查下查看服务是否有开启已经API Key是否有值确认都已经安装了情况下，如果还是提示配置缺失，可以重启下Obsidian，再重新安装可以看到安装成功了

## 三、把Claude Desktop连到Obsidian

重启下claude桌面版，可以看到Connectors里面多了obsidian-mcp-tools了我们发送下：让他列出我的bosidian vault所有比较，可以看到它自己MCP拉取读取了我本地的文件内容

## 四、使用Claude Desktop操作本地知识库

接下来我们就可以使用Claude客户端操作本地知识库了，配好之后，以下是真实能跑通的用法，比 ima 强很多：

### 场景 1：知识库问答（替代 ima）

```
查一下我笔记里关于 Cloudflare R2 配置的内容，总结一下踩过的坑
```

Claude 会用语义搜索找到相关笔记 → 读取全文 → 给你总结。

### 场景 2：跨笔记整理

```
帮我把"AI资讯互联"文件夹下所有关于 Claude 的笔记，整理成一篇综述文章，  
输出到一个新笔记叫"Claude 相关内容汇总.md"
```

Claude 会读取多篇笔记 → 综合提炼 → 直接在你 vault 里创建新笔记。

### 场景 3：批量重构

```
把我"圈友互联"文件夹下所有笔记的标题格式统一成「YYYY-MM-DD - 主题」的样式
```

Claude 会列出文件 → 逐个读取 → 重命名。

### 场景 4：写公众号初稿

```
基于我笔记里「Obsidian 配置」相关的内容，帮我写一篇公众号文章初稿，  
存到"草稿/Obsidian 配置指南.md"，风格参考我已有的「新加坡生活」那篇笔记
```

Claude 会参考你的写作风格写初稿，直接落盘。

### 场景 5：日常速记

```
帮我在今天的日记里追加一条：下午 3 点和 XXX 开会，讨论了 AI 产品方向
```

Claude 会找到或创建今天的日记，追加内容。

## 安全说明

也许有人会说，整这一套太麻烦，直接丢到Claude Code的根目录下全部搞定，不过这套方案看起来给了 Claude 很大权限，但实际上是**相对安全**的：

* 所有通信都在**本机**（`localhost`），不走公网
* Local REST API 用 **API Key 鉴权**，Claude Desktop 配置文件里存着
* **每次工具调用都会弹权限确认**（除非你选了"全部允许"）
* 你可以在 Local REST API 里限制允许的操作类型
* Claude 对话默认不会被 Anthropic 用来训练模型

## 五、使用手机控制本地电脑

Claude最新版本推出了协同操作，直接手机下载Claude Mobile APP，再和桌面版完成绑定，就可以在随时随地用手机对话让它帮你查询你本地知识库内容了！

## 总结

本文详细讲解了如何把Obsidian作为本地MCP Server，并详细讲解了如何让其余AI客户端通过MCP Tool工具访问本地。

下篇文章，我将详细讲讲最新超火的Hermas Agent安装教程及使用（一款OpenClaw的升级产品，龙虾时代已去，悍马Agent即将来袭）！
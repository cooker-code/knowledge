> 已吸收至：[[02_Agent与AI工程/0201_Agent框架/020119_交付型Agent/020119_核心知识点/表格与架构图Agent交付边界|表格与架构图Agent交付边界]]
---
title: 安利一款AI Agent画架构图神器，真一键生成。
author: 李岳AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzA3MTg4NjY4Mw==&mid=2457348318&idx=1&sn=26b6261119b83980052d897ee6212d8f&chksm=8913d805ef16bd5ce8862a92c49d4851be73d0ed738e117574dd938dc3609638be2107e5f089&mpshare=1&scene=24&srcid=0415lPRypDUCMsgUsC8Z96Gq&sharer_shareinfo=ffb643cef0d8410fc7af3818701f158b&sharer_shareinfo_first=ffb643cef0d8410fc7af3818701f158b#rd
---

大家好，我是岳哥。

今天给大家分享一款画架构图的Skill：**architecture\_diagram**，专门用来画各种架构图。

点击图片可以查看大图

你只需要提供相应的组件，AI就可以帮你直接生成。

## Skill来源

这个功能原本是一个Claude Code的Skill开源项目，项目地址：

> https://github.com/Cocoon-AI/architecture-diagram-generator

## Claude如何生成专业的架构图

只需要3步即可生成前面漂亮的架构图。

#### 1、安装Skills

有了项目地址，我们可以直接下载解压后存放到Claude Code的Skills目录下面。

或者直接跟AI说：

> 帮我安装这个Skill，[Skill项目地址]

安装还是非常简单的。

#### 2、 准备架构描述

这是最关键的一步，你需要告诉 AI 你的系统长什么样。

不用担心描述得不够“专业”，这里有三种方法可以让你描述清楚：

* 让 AI 分析你的代码库：直接把你的项目代码路径（或仓库链接）发给 Claude，然后问它:

```
分析这个代码库的架构，列出所有主要组件、连接方式、使用的技术。
```

* 简单地列出核心组件和它们之间的关系。例如：

```
一个用户通过浏览器访问网站，
网站运行在 Nginx 服务器上，
背后连接着 Python Django 应用，
Django 又连接着 PostgreSQL 数据库
```

* 如果你不知道从何说起，可以直接问 Claude：

```
给我一个典型电商网站/物联网平台/SaaS 应用的架构描述示例。
```

三种方法，总有一种适合你。

#### 3、生成图表

当你有了清晰的架构描述后，在 Claude 的对话框里，先输入一句“咒语”来触发技能，然后粘贴你的描述。 格式如下：

```
使用architecture_diagram这个Skill来描述创建图表：
[在这里粘贴你的架构描述]
```

例如：

> 使用architecture\_diagram这个Skill来描述创建图表：一个 React 前端连接 Node.js/Express API，使用 PostgreSQL 数据库和 Redis 缓存。

发送后，Claude 就会调用已安装的技能，处理你的描述，并最终生成一个可以直接下载的 HTML 文件。 点击下载，用浏览器打开，你的专业架构图就诞生了。

## 具体案例

看完了步骤，如果你还是有点不确定“架构描述”到底该怎么写。可以参考以下几种提示词写法。

#### 1、全栈Web应用

```
## 提示词
使用architecture_diagram这个Skill绘制一张 Web 应用架构图，包含以下组件：
- React 前端
- Node.js/ Express 后端接口服务
- PostgreSQL 数据库
- Redis 缓存
- JWT 身份认证
```

就可以得到下面这样一张图：

#### 2、

```
## 提示词
使用architecture_diagram这个Skill绘制一张架构图，包含以下组件：
- CloudFront 内容分发网络（CDN）
- API 网关
- Lambda 函数（Node.js 运行时）
- DynamoDB 数据库
- 用于存放静态资源的 S3 对象存储
- 负责身份认证与授权的 Cognito
```

就可以得到下面这样一张图：

#### 3、微服务

```
## 提示词
使用architecture_diagram这个Skill为一套微服务系统绘制架构图，系统包含以下组件：
- React 网页应用 与 移动端客户端
- Kong API 网关
- 用户服务（Go 语言）、订单服务（Java 语言）、商品服务（Python 语言）
- PostgreSQL、MongoDB 及 Elasticsearch 数据库
- 用于事件流处理的 Kafka
- Kubernetes 容器编排
```

就可以得到下面这样一张图：

我们可以通过上面的示例，总结一些小技巧：

* 组件命名尽量通用：用“前端”、“后端”、“数据库”，而不是具体的项目代号。
* 连接关系说清楚：使用“连接”、“访问”、“发送数据到”等词明确组件间如何交互。
* 技术栈要具体：写明是“React”还是“Vue”，是“PostgreSQL”还是“MySQL”，这会影响图标和颜色编码。
* 可以描述层级：例如“用户通过浏览器访问负载均衡器，负载均衡器将流量分发给后端的多个应用服务器实例。”

## Hermes如何生成

Hermes Agent最近刚更新进去，使用起来更加方便。

直接在对话框里输入

> /architecture\_diagram [架构图描述提示词]

例如：

```
/architecture_diagram 为一套微服务系统绘制架构图，系统包含以下组件：
- React 网页应用 与 移动端客户端
- Kong API 网关
- 用户服务（Go 语言）、订单服务（Java 语言）、商品服务（Python 语言）
- PostgreSQL、MongoDB 及 Elasticsearch 数据库
- 用于事件流处理的 Kafka
- Kubernetes 容器编排
```

同样可以生成微服务的架构图。

当然你需要将Hermes更新到最新版本，使用命令：

> hermes update

更新即可。

以上就是这个Skill的具体介绍，对于做开发的同学画架构图还是很方便的。

---

我创建了几个AI交流群，大家可以在群里交流各种AI工具的使用，欢迎大家进群交流。

公众号后台私信：**AI群**

我邀请你进群，群里禁止发广告和违规违法言论，违者移出群聊。
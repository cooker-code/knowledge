> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: 2025最好用的MCP平台一览表
author: 苍何
date:
url: http://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493385&idx=1&sn=3e2bd48aa0c69a57def6874de5948d7d&chksm=fcad39ede1c828e00034b2210506534329edcdfe91fbc14e4008be5591d9b8ab526df94df9c3&mpshare=1&scene=24&srcid=0515cTHJ8KiGvscaFeu7M1GI&sharer_shareinfo=8ba24fed168acfcec45cdb927116e146&sharer_shareinfo_first=8ba24fed168acfcec45cdb927116e146#rd
---

这是苍何的第 369 篇原创！

大家好，我是苍何。

MCP 也出来好一阵子了，该说不说，这玩意是真火。

现在我们使用的很多工具都支持 MCP 协议了，最牛叉的是，不少平台直接就能一键使用和自定义 MCP Server。

像苍何之前分享的字节、百度、阿里、腾讯自家的 AI 应用平台都已经全面支持 MCP 的接入。

经过一段时间的深度体验，觉得有必要汇总一波目前好用的 MCP 平台。

不过，在此之前，还是要再叨叨解释下 MCP 是个啥玩意，以及这段时间来的一些思考。

> 已经熟悉的大佬完全可以往下滑，找到汇总部分，嘻嘻嘻。

MCP 是一种模型上下文协议，全称为 Model Context Protocol，它是由 Anthropic 公司建立并开源的。

**现在基本已经成为了被全世界公认的主流协议了。**

简单来说，MCP 可以让大模型（比如 DeepSeek、豆包等）可以痛快的调用外部工具和资源。

这么解释，可能还是有些抽象，举个栗子，比如苍何之前分享的[在 Cursor 中构建微信读书系统](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493075&idx=1&sn=71010faab7a021ddff59cc3eadeda0ea&scene=21#wechat_redirect)。

你可以看到，通过微信读书的 MCP Server，Cursor 中的大模型能够直接访问我们个人的微信读书数据。

可以看到，**MCP 简化了 AI 模型与数据、工具和服务之间的交互方式。**

MCP 让大模型不再局限于静态知识库，而是能像人类一样**调用搜索引擎、访问本地文件、连接 API 服务**，甚至操作第三方软件。

但以上只是比较浅层的 MCP 概念，这里我想给你分享的是 MCP 的内核是什么。

不知道你发现没，如果是单纯调用工具，那和 API 以及 Function Call 有啥区别？

> API 指的是传统软件开发中的接口调用，Function Call 指的是函数调用，调用某一个代码来执行特定的功能。

**我认为 MCP 的精髓，不应该仅是调得动工具。**

如果单纯调用工具，API 和 Function Call 都可以调。

我觉得 MCP 真正的内核应该是**让大模型能调用工具的同时，记得住自己调用过的工具，并放在上下文中，进行自主选择和判断**。

▼ 图源网络

也就是在多 Agent 协作时，LLM 大模型能知道自己之前调用了哪些工具，结果是什么，用户提问的场景是什么，基于这些上下文，再去做更加复杂的任务。

解释完 MCP，有必要开始我整理的 2025 好用的 MCP 平台，先来个概览图吧：

这份表格我直接放在了我的开源知识库中了：

https://u55dyuejxc.feishu.cn/share/base/view/shrcnY348b3pNdLQ2gdnwgFj88c

内容也很全面，包括各大平台的一些有趣的使用场景，都是亲身实践而来。

下面我抽出 9 个平台，来展开说说，包括怎么使用呀，具体的场景呀等等。

# Cursor

Cursor 可以说是最早一批支持 MCP 的客户端了。在 Cursor 中可以使用多种大模型，其中我认为比较好用的是 claude 3.7 和 Gemini 2.5 pro。

应用场景写了几篇文章，大家可以稍微做个参考：

> 1、[Cursor+MCP 实现用嘴操纵数据库，太丝滑了！](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493032&idx=1&sn=c0963b8cfefa128d9ec9643dfeca9e4d&scene=21#wechat_redirect)
> 2、[用微信读书MCP在Cursor中构建私人图书馆，太哇塞了！](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493075&idx=1&sn=71010faab7a021ddff59cc3eadeda0ea&scene=21#wechat_redirect)

配置也不难，但需要你稍微懂一点点 JSON 和 Python 的知识。

拿高德地图 MCP 来举例看看如何配置。

进入 Cursor Settings：

添加高德地图的 MCP Server：

直接黏贴这段代码：

```
{   "mcpServers": {       "amap-maps": {           "command": "npx",           "args": [               "-y",               "@amap/amap-maps-mcp-server"           ],           "env": {               "AMAP_MAPS_API_KEY": "这里填写你在高德申请的API key"           }       }   } } 
```

这样在 Cursor Settings 中就可以看到灯绿了，如果灯是红的，可以点击右侧的 Enabled 或者刷新即可。

那如何使用呢？选择 Agent 模式，模型选择 claude-3.7-sonnet。

现在，我就可以直接在 Cursor 中查询天气了。可以看到自动调用了 MCP。

还可以进行形成规划：

# Cherry Studio

Cherry Studio 是国产 AI 工具之光，支持 MCP 接入。

配置也不复杂，和 Cursor 差不多。

安装最新版的 Cherry Studio，打开设置-MCP 服务器。

上方需要的依赖直接点击安装。（UV 和 Bun）

选择添加服务器：

也可以直接将上面的 JSON 直接复制到「编辑 JSON」这里。

你可以点击右侧的编辑操作查看详情，并修改描述。

激活高德服务：

这里特别注意，需要先选择支持函数调用的模型，比如我用的是火山引擎，那就需要选择有扳手的模型。

在对话框中选择该模型，在下方工具栏才会出现 MCP 服务器组件选择：

同样让他查天气，可以看到自动调用高德的 MCP 了：

# Cline 使用

Cline 是在 Vscode 中的一个插件，支持搜索 MCP，及一键使用。

在 vscode 或者 Cursor 中下载 Cline 插件：

安装好后点击图标选择使用方式即可。

选择登录方式，可以谷歌或者 GitHub

同样可以将高德的 MCP 添加进来：

> 也可以直接在这里一键添加 MCP 服务

就可以看到已经安装的 MCP 服务亮绿灯代表 OK：

同样直接在对话框中提问：

# 阿里云百炼

阿里云百炼平台已经支持 MCP，可以使用平台内置的很多 MCP Server，还支持自定义 MCP。

使用可以参考苍何之前的文章：[用阿里百炼MCP两分钟做了个天气系统，全程无需配置，小白也能直接上手](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493050&idx=1&sn=2bb3d4ee623d3e486571abfd28e60625&scene=21#wechat_redirect)

打开阿里云百炼。

点击 MCP 后发现有很多的 MCP 服务：

选择立即开通：

创建一个应用来调用刚开通的 MCP 服务，点击应用，这里可以看到有个 MCP 管理，可以看到刚才开通的 MCP 服务。

点击新建应用

这里先创建一个智能体应用：

选择模型，特别注意，模型这里要选择 plus 模型，Max 模型军不可用 MCP。

添加 MCP 服务即可在应用中使用 MCP。

# 腾讯云大模型知识引擎

腾讯云自家的大模型应用平台，目前也已经全面支持 MCP，并且可自定义 MCP Server。

也可以参考我之前的文章：[我用腾讯云MCP两分钟做了个PPT一键生成器，从此效率起飞！！](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493114&idx=1&sn=5f4b9085b244bd5b1890310fbf89d5cf&scene=21#wechat_redirect)

使用很简单，基本不需要配置，大大简化了配置的成本。

打开大模型只是引擎官网，应用管理-新建应用：

选择 agent 模式，目前只有 agent 模式才可使用 MCP 服务。

向下滑一滑，找到插件，点添加插件。

类型选择 MCP 即可。

这里官方做了个很细心的体验，MCP Server 提供了很多的 tools，但并不是每个都需要使用，支持按需使用。

比如 ChatPPT，就可以按需添加需要的能力。

举个微信读书的栗子：

直接可以找到微信读书的 MCP：

添加工具的时候，需要输入微信读书的 cookie。

# 字节火山DeepSearch

DeepSearch 是一款专为处理复杂问题而精心设计的高效工具，可集成联网搜索、知识库、网页解析、日志服务等丰富的 MCP 服务。

使用的话也简单，方舟后台——应用实验室——我的应用——创建应用

选择高代码：

# 扣子空间

在扣子空间中可快速使用 MCP，打开扩展就可以一键使用平台内置的 MCP 工具。

如果不满足需求，还可以自己添加自定义的 MCP。

# trae 国内版

trae 是字节研发的 AI 编程工具，目前也已经全面支持 MCP 了。

在 trae 的 MCP 市场中集成了很多的 MCP Server，而且还细心的标注了使用难度。

就用户体验这一块，字节真的是给力。

当然如果市场不满足使用，还可以手动配置 MCP，主打一个方便。

使用的时候选择 Builder with MCP。

# 百度千帆平台

目前百度千帆平台也已经全面支持 MCP 的使用。

详细可参考苍何之前的文章：[用百度网盘MCP在Cursor中构建私人网盘助手，太香了叭（附搭建教程）](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247493231&idx=1&sn=036249d539fec6096c16f20de172b4dd&scene=21#wechat_redirect)

需要在创建工作流应用。

工作流中选择 MCP-Server

选择配置 MCP Server：

点击链接后，选择工具：

文件上传仅支持本地模式。

测试通过即可使用。

这里以工具，file\_search 为例，获取用户网盘中的文件名包含指定关键字的文件列表。

其中输入类型选择引用，值选择系统参数。

直接将结果解析到输出参数：

讲真，有了 MCP 确实让大模型变得无所不能，各大平台也在自己的产品中注入对 MCP 的理解。

不过从 Agent 角度来看，配置 MCP 这个事情，是不是也应该变得更简化一些？

**能根据用户指令自动搜索 MCP，自动配置，自动调用，并将信息和调用链路存在于 LLM 的上下文中。**

这种形态，我感觉是很不错的。像 Cherry Studio 中的 mcp-auto-install 就很有意思。他会自动取 MCP 市场搜索 MCP，自动帮我们做好配置。

但从自由度上来说，太过于追求极简，又会变得灵活性不高。

其中终归是取决于你面对的使用场景、目标用户以及平台的核心定位。

很多平台其实都在“**中间地带找平衡**”，既有极度简化的 MCP，无需配置点个按钮就可。

也支持自定义 MCP，灵活性更强、

对用户来说，最终选择哪个平台以及怎么使用。

终将是取决于自己的需求。

好啦，以上全文 **9149 字，15 张图**，如果这篇文章对你有用，可否点个关注，给我个三连击：点赞、转发和再看。若可以再给我加个⭐️。

> 文章同步我的 AI 开源知识库：[AI 开源知识库](https://mp.weixin.qq.com/s?__biz=MzU4NTE1Mjg4MA==&mid=2247492745&idx=1&sn=5bfca0f4b2b429d77e7f84a1e662f898&scene=21#wechat_redirect),，或者回复 AI 即可直接获取。

# ending

我创建了一个读者 AI 交流群，群里都是一群前沿的 AI 极客，经常讨论最新的 AI 消息，DeepSeek 用法，以及变现方法。

但是任何人在群里打任何广告，都会被我 T 掉。

如果你对这个特别的群，感兴趣，可以公众号后台私信我加入。

暗号：**AI**

*点击关注下方账号，你将感受到一个朋克的灵魂。*

【我爱这个魔幻的世界】
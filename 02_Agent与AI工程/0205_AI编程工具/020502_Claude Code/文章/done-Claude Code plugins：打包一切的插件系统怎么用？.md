> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: Claude Code plugins：打包一切的插件系统怎么用？
author: AI编程实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483906&idx=1&sn=25429bf337b61648e1eec3cc9499c55d&chksm=9730468bfad899a4d8e332498f5e2822bfdfdbf2ed191f7db3fbd5256cf05cb69d7ffac619cf&mpshare=1&scene=24&srcid=1117AYWLY0hJDLce43hDa4Uk&sharer_shareinfo=bb8f9294f1d7a2d2f98c2fb97cb86649&sharer_shareinfo_first=bb8f9294f1d7a2d2f98c2fb97cb86649#rd
---

大家好，我是鲁工。

10月份Anthropic给Claude Code加了个大招：plugins插件系统，能够使你的Claude Code能力更加定制化。

简单说，plugins就是Claude Code的应用商店。你可以一键安装别人做好的工具包，也可以自己动手打造专属插件。比如我现在用的代码审查命令、DevOps自动化脚本，都是通过插件实现的，效率提升相当明显。

## plugins是什么

plugins本质上是个工具打包系统，把五种扩展方式整合在一起：**斜杠命令、agents、skills、Hooks和MCP服务器**。

**斜杠命令（Slash Commands）**是最直观的。你可以创建类似`/review-pr`或`/deploy`这样的快捷命令，把常用操作固化下来。比如每次代码审查都要检查的那些点，写成一个命令，以后直接调用就行。

**Agents是专门干某类活儿的助手。和Claude主会话不同，代理更聚焦，比如专门做调试的代理、专门写测试的代理。需要的时候叫出来用，不需要时不占用上下文。具体可参考：**

**[用Subagents打造Claude Code专业开发团队](https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483687&idx=1&sn=7e14d5bd5dafc9e8db0181687d3da5bc&scene=21#wechat_redirect)**

**技能（Skills）**比较特别，它是Claude能自动调用的能力。你不用显式触发，Claude判断需要时会自己用。比如你给它加了个查询数据库的技能，它分析问题时发现需要查数据，就会自动去查。具体可参考：

[Claude Skills使用教程，AI Agent终于迎来可复用的工作流引擎](https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483720&idx=1&sn=a9c907b15806c281948042907ec117d9&scene=21#wechat_redirect)

**钩子（Hooks）**是事件触发器。在特定时机自动执行操作，比如每次提交代码前自动跑格式化，或者每次启动Claude Code时加载特定配置。

**MCP服务器**是连接外部工具的桥梁。通过Model Context Protocol，你可以把内部系统、数据库、API 接入Claude Code。具体可参考：

[Claude Code，我只用这两个MCP](https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483665&idx=1&sn=e9dc848c277df19fdeff016b6faa487b&scene=21#wechat_redirect)

这五种能力可以单独用，也可以组合打包成一个插件。比如一个完整的PR 审查插件，可能包含审查命令、代码分析代理、自动检查技能，还有提交前的格式化Hooks。

## 15分钟创建第一个插件

根据Claude Code官方给的操作示例，我们可以从零创建一个简单的问候插件，熟悉整个流程。

### 第一步：建立目录

首先创建一个本地市场和插件目录：

```
mkdir test-marketplace && cd test-marketplacemkdir my-first-plugin && cd my-first-plugin
```

这里有个概念要理解：**插件（plugin）** 和 **市场（marketplace）**。插件是具体的工具包，市场是插件的集合。一个市场里可以放多个插件。

### 第二步：创建插件清单

在插件目录下创建配置：

```
mkdir .claude-plugin
```

然后创建 `.claude-plugin/plugin.json` 文件：

```
{  "name": "my-first-plugin",  "description": "一个简单的问候插件",  "version": "1.0.0",  "author": {    "name": "louwill"  }}
```

这个文件定义了插件的基本信息。`name` 是唯一标识，安装时会用到。

### 第三步：添加斜杠命令

创建`commands`目录和第一个命令：

```
mkdir commands
```

创建 `commands/hello.md` 文件：

```
---description: 友好地问候用户---# Hello 命令请热情地问候用户，并询问有什么可以帮助的。
```

命令文件用Markdown编写，开头的`---`区域是YAML元数据，`description`会显示在帮助信息里。下面的正文就是给Claude的指令。

### 第四步：创建市场清单

回到 `test-marketplace` 目录：

```
cd ..mkdir .claude-plugin
```

创建 `.claude-plugin/marketplace.json` 文件：

```
{  "name": "test-marketplace",  "owner": {    "name": "louwill"  },  "plugins": [    {      "name": "my-first-plugin",      "source": "./my-first-plugin",      "description": "我的第一个测试插件"    }  ]}
```

`plugins`数组中会列出这个市场包含的所有插件。`source`指向插件目录，可以是相对路径、绝对路径，也可以是Git仓库地址。

### 第五步：安装和使用

在 Claude Code 中执行：

```
# 添加本地市场/plugin marketplace add /path/to/test-marketplace
# 安装插件/plugin install my-first-plugin@test-marketplace
# 使用命令/hello
```

如果一切顺利，Claude Code会友好地跟你打招呼。恭喜，你的第一个插件已经跑起来了！

完整的目录结构是这样的：

```
test-marketplace/├── .claude-plugin/│   └── marketplace.json└── my-first-plugin/    ├── .claude-plugin/    │   └── plugin.json    └── commands/        └── hello.md
```

## 社区生态：243个插件等你探索

自己做插件有门槛，好在社区已经造了不少轮子。一个月的时间，GitHub上已经冒出来好几个活跃的插件市场。

**jeremylongshore/claude-code-plugins-plus**是目前规模最大的，243个插件，其中175个支持最新的Agent Skills v1.2.0规范。这个市场做得比较规范，100%符合Anthropic 2025 年的Skills 架构标准（不过，从他的项目目录可以明显看出，他们这个插件市场很大程度上也是依靠Claude Code Vibe出来的）。

**jeremylongshore/claude-code-plugins-plus**插件地址：

https://github.com/jeremylongshore/claude-code-plugins-plus

**wshobson/agents**走的是精品路线，85个专门 AI 代理、15个多代理工作流编排器、47个代理技能、44个开发工具，打包成63个主题插件。质量相对更高，适合深度使用。

**wshobson/agents**插件地址：

https://github.com/wshobson/agents

### 那么如何安装社区插件？我们以claude-code-plugins-plus为例：

```
# 添加 GitHub 市场/plugin marketplace add jeremylongshore/claude-code-plugins-plus
# 浏览可用插件（会显示插件列表）/plugin
# 安装具体插件（语法：插件名@市场名）/plugin install code-review@claude-code-plugins-plus
```

### 装完直接就能用，不需要额外配置。

###

## 团队玩法：打造专属插件市场

个人用插件是提效，团队用插件是统一规范。

假设你们团队有一套代码规范、一套部署流程、一套测试标准。以前只能靠文档约束，现在可以把这些固化成插件。新人入职，装上团队插件包，自动对齐所有规范。

### 配置自动安装过程如下，在项目根目录创建 `.claude/settings.json`：

```
{  "pluginMarketplaces": [    {      "url": "your-org/your-marketplace"    }  ],  "plugins": [    {      "name": "team-standards",      "marketplace": "your-org/your-marketplace",      "enabled": true    }  ]}
```

团队成员信任这个文件夹后（Claude Code会提示信任），插件会自动安装和启用。整个过程对用户透明，他们甚至不需要知道`/plugin`命令怎么用。

插件也支持语义版本号。你可以在`plugin.json`里指定版本：

```
{  "name": "team-plugin",  "version": "1.2.0"}
```

更新插件后，用户重装就能拉到新版本。如果担心新版本有问题，可以先在小范围测试，稳定后再推全员。

这套机制特别适合有一定规模的技术团队。统一工具链、统一规范、统一最佳实践，都可以通过插件来落地。

## 写在最后

Claude Code plugins发布才一个多月，社区生态已经相当活跃了。从243个插件的规模来看，这个功能确实戳中了开发者的痛点。

对我个人而言，plugins最大的价值是**让AI编程助手从通用工具变成专用工具**。以前Claude Code啥都能干，但每次都要重新解释需求。现在把常用场景做成插件，效率提升不是一点半点。

如果你也是Claude Code日常用户，强烈建议试试plugins。从装个社区插件开始，感觉好用可以再考虑自己构建一个。这个功能门槛不高，但天花板挺高的。

感谢您阅读我的文章。我是鲁工，八年AI算法老兵，AI全栈开发者，深耕AI编程赛道。欢迎关注，感兴趣的朋友也可以加我微信（louwill\_）交个朋友。

>/ 作者：鲁工
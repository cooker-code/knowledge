---
title: 配50个Claude项目手改mcp.json？这招让你永远只改1次
author: 老金带你玩AI
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI0NzU2MDgyNA==&mid=2247489745&idx=1&sn=bd3dc3b3950877202534898516558d8b&chksm=e84e396543f29dfa24196b01c9d1d35518a71db0816b3f70b9a0ee67d45ebf118d6b16349946&mpshare=1&scene=24&srcid=1118gaFcBqUTMMPE9eLFiLNc&sharer_shareinfo=442a9fb5635f02bb9e60c186c9ee2319&sharer_shareinfo_first=442a9fb5635f02bb9e60c186c9ee2319#rd
---

加我进AI讨论学习群，公众号右下角“联系方式”

文末有老金的 **开源知识库地址·全免费**

---

老金我之前被MCP服务器折磨得够呛。

你想啊，Claude Code装了20个MCP服务器，Cursor又装了15个，Cline里还配了12个，Codex和Kilo也各配了十几个，其他各种新IDE和CLI也都需要配置一遍。

结果呢？

每个工具都要单独配一遍，改个配置要改五个地方，想关个服务器得到处找配置文件。

这不是典型的工具越多，麻烦越多吗？

直到我发现了MCP Router这个开源项目，才算是把这团乱麻理清楚了。

## MCP Router功能说明

说白了，MCP Router就是一个桌面版的MCP服务器管理中心。

你可以理解成：

以前你家的遥控器有20个（电视、空调、机顶盒、音响、窗帘、灯光...），每次看个电视要拿好几个遥控器。

MCP Router就是把这20个遥控器的功能整合到一个万能遥控器上，所有设备都能在一个界面上管理。

核心功能就4个：

1、所有MCP服务器集中管理- 不管是本地服务器还是远程服务器，全在一个桌面应用里

2、一键开关- 想用哪个服务器开哪个，不用的直接关掉

3、分组管理- 通过Projects（项目）和Workspaces（工作空间）把服务器分门别类

4、工具级别控制 - 每个服务器下的工具都能单独开关

## 为啥老金我说它是神器

给你讲个真实场景。

上周我用Claude Code写公众号文章，配了这些MCP服务器：

1、brave-search（网页搜索）

2、exa（深度搜索）

3、search1api（多源搜索）

4、Memory Tool（记忆存储）

5、thinking（AI思考）

6、context7（技术文档）

7、mcp-deepwiki（深度文档）

结果用着用着，AI突然卡住了。

因为它同时调用了7个服务器的工具，处理超时了。

我当时就想关掉几个不用的服务器，但问题来了：

1、要打开配置文件（.mcp.json 或类似的）

2、找到对应的服务器配置

3、注释掉或删除

4、重启工具

这一套流程下来得5分钟，思路都断了，真是让人头疼。

有了MCP Router之后，我只需要：

1、打开MCP Router桌面应用

2、点击想关闭的服务器旁边的开关

3、完事

3秒钟搞定，而且不用重启任何工具。

这才是真正的所见即所得啊兄弟们！

## 核心功能深度拆解

老金我给你拆解一下MCP Router的几个杀手级功能。

### 1、统一服务器管理（最基础但最重要）

这是MCP Router的核心价值。

传统方式的痛点：

你用Claude Code、Cursor、Codex、Cline、Kilo……等等一系列工具，每个都要配置MCP服务器。

假设你想用同样的20个MCP服务器（brave-search、exa、context7、Github、playwright、Memory Tool、thinking、serena等），就得：

1、在Claude Code的配置文件里写一遍

2、在Cursor的配置文件里写一遍

3、在Codex的配置文件里写一遍

4、在Cline的配置文件里写一遍

5、在Kilo的配置文件里写一遍

6、其他IDE和CLI的配置文件里再写一遍

5个工具 × 20个服务器 = 至少100次配置。

改个API Key？

对不起，100个地方都得改。

MCP Router的解决方案：

所有MCP服务器都配置在MCP Router里，你的Claude Code、Cursor、Codex、Cline、Kilo只需要连接到MCP Router就行。

```
# 所有工具都用这一条命令连接MCP Router  
export MCPR_TOKEN="你的token"  
npx -y @mcp_router/cli connect
```

1次配置，所有工具通用。

改API Key？

只需要在MCP Router里改一次。

### 2、Projects（项目分组）

这个功能太实用了。

这个目前有BUG，我已经提交了修复，不知道什么时候能合并进去，在下方安装时候，教大家怎么修改。

老金我有3个项目在同时开发：

项目A：技术研究

1、需要：context7、Github、mcp-deepwiki

2、不需要：BrowserTools、playwright、search1api

项目B：公众号写作

1、需要：brave-search、exa、search1api、Memory Tool、thinking

2、不需要：Github、context7、mcp-deepwiki

项目C：自动化测试

1、需要：BrowserTools、playwright、spec-workflow

2、不需要：context7、mcp-deepwiki、brave-search

以前的做法：

每次切换项目，手动去配置文件里开关服务器，麻烦死了。

用MCP Router的Projects功能：

1、创建3个Project：技术研究、公众号写作、自动化测试

2、每个Project里配置需要的MCP服务器

3、切换项目时，用命令指定Project：

```
# 技术研究模式  
npx -y @mcp_router/cli connect --project 技术研究  
  
# 切换到公众号写作  
npx -y @mcp_router/cli connect --project 公众号写作
```

切换项目 = 切换一套完整的MCP服务器配置，简直不要太爽。

### 3、Workspaces（工作空间）

这个功能类似浏览器的配置文件（Chrome Profiles）。

使用场景：

老金我白天写代码做技术研究（工作模式），晚上写公众号文章（创作模式）。

工作模式需要的服务器：

1、context7（查技术文档）

2、Github（代码协作）

3、mcp-deepwiki（深度文档）

4、spec-workflow（工作流管理）

5、shrimp-task-manager（任务管理）

创作模式需要的服务器：

1、brave-search（网页搜索）

2、exa（深度搜索）

3、search1api（多源搜索）

4、Memory Tool（记忆存储）

5、thinking（AI思考）

6、serena（内容辅助）

用Workspaces功能：

1、创建两个Workspace：技术研究、公众号创作

2、每个Workspace配置不同的服务器组合

3、切换Workspace就切换一套环境

就像切换浏览器配置文件一样简单，工作和创作彻底分离。

稍微麻烦点的是，新的工作空间是空的，需要重新导入，你可以在这导出JSON后，进行调整后再次导入。

### 4、工具级别的开关控制

这个细节老金我特别喜欢。

每个MCP服务器下面可能有好几个工具（Tools）。

比如 brave-search 服务器有：

1、brave\_web\_search（网页搜索）

2、brave\_local\_search（本地搜索）

场景：我在写文章只需要网页搜索，不需要本地搜索。

传统方式：只能整个服务器开或关，没法精细控制。

MCP Router的方式：

打开MCP Router界面，展开 brave-search 服务器，把 brave\_local\_search 的开关关掉。

这样AI就只能做网页搜索，避免误调用其他工具执行错误。

这种细粒度控制，对于生产环境特别重要。

### 5、详细的日志和分析

这个功能救过老金我好几次命。

场景：AI工具突然报错说某个MCP服务器调用失败。

以前：完全不知道哪里出了问题，只能瞎猜。

有了MCP Router的日志功能：

1、打开Logs界面

2、看到完整的请求记录：哪个工具、什么时候调用的、参数是啥、返回了啥

3、一眼就能看出问题在哪

数据驱动优化，这才是专业玩家的玩法。

如果对你有帮助，记得关注一波~

## 实际使用体验

老金我用了两个月MCP Router，给你看看实际效果。

### 使用前（传统方式）

配置MCP服务器：

1、需要手动编辑5个工具的配置文件

2、每次改配置要重启工具

3、不知道哪个服务器在用，哪个闲置

切换项目：

1、手动改配置文件

2、容易忘记改回来

3、经常配置混乱

排查问题：

1、完全靠猜

2、看不到调用记录

3、不知道是哪个服务器出错

### 使用后（MCP Router）

配置MCP服务器：

1、在MCP Router里配一次，所有工具通用

2、改配置不用重启，实时生效

3、一眼看到每个服务器的状态

切换项目：

1、用Projects功能一键切换

2、配置自动保存，不会混乱

3、每个项目独立管理

排查问题：

1、打开Logs看完整记录

2、知道哪个工具调用失败

3、统计数据帮助优化配置

最关键的是，心智负担大幅降低。

以前每次改配置都要小心翼翼，生怕写错格式。

现在界面化操作，点点鼠标就搞定。

## 手把手教你用MCP Router

老金我知道你们最关心怎么用，直接上步骤。

### 第1步：下载安装

去GitHub Releases页面下载：

https://github.com/mcp-router/mcp-router/releases

1、Windows用户：下载 .exe 文件

2、macOS用户：下载 .dmg 文件

双击安装，就像装普通软件一样简单。

### 第2步：添加MCP服务器

打开MCP Router桌面应用，点击"Add Server"。

三种添加方式：

方式1：从现有配置导入（推荐）

如果你已经在其他工具（Claude Code、Cursor等）配置过MCP服务器，可以直接导入：

1、点击外部导入开关

2、选择现有配置文件路径

3、自动导入所有服务器

方式2：手动添加

1、点击"Add Manually"

2、填写服务器信息：

* 名称（随便起）
* 类型（本地或远程）
* 命令或URL
* 环境变量（如果需要）

方式3：使用JSON配置

如果你有现成的JSON配置，直接粘贴进去。

### 第3步：创建Project

点击"Projects"，创建新项目：

1、项目名称：比如"电商网站开发"

2、选择需要的MCP服务器（勾选即可）

3、保存

想创建几个Project就创建几个，互不干扰。

上面提到的bug是你会发现它只能匹配到未分配的项目，如果你创建了新的项目，它就无法识别到。

修改内容在这，我已提交给作者。

其他的不用问我，因为我不会代码，全靠AI修改安装 =。=

原因解析：

然后叫他重新打包就行，你是啥系统就说啥系统，win就标注win，mac就标注mac。

打包完成后，重新安装即可。

就会发现BUG搞定了，记得重启你调用MCP的工具生效。

### 第4步：连接到MCP Router

在你的AI工具里（Claude Code、Cursor、Codex、Cline、Kilo等），直接配置MCP，新增应用。

比如加了这4个，然后点击使用方法。

把下面这个复制到你要使用的软件MCP配置内即可。

Codex配置特殊，需要看之前的完整教学！敲黑板！

[老金·邪修大白话做产品工具之Codex从环境配置到 MCP Router，再到 Agent 深度调教（万字详解）](https://mp.weixin.qq.com/s?__biz=MzI0NzU2MDgyNA==&mid=2247489317&idx=1&sn=d164051a9397bf766e020670578e1432&scene=21#wechat_redirect)

这一步很重要：

以前你的AI工具直接连接各个MCP服务器，现在统一连接到MCP Router。

MCP Router充当中间人，负责把请求分发到对应的服务器。

### 第5步：享受便利

现在你可以：

1、在MCP Router界面里一键开关服务器

2、查看实时日志

3、切换Projects

4、管理Workspaces

所有操作都在桌面应用里完成，不用再碰配置文件。

## MCP Router的隐私和安全

这个老金我必须强调一下。

所有数据都在本地：

1、服务器配置：存在你电脑上

2、API Keys：存在你电脑上

3、请求日志：存在你电脑上

MCP Router本身不会把任何数据发到外部服务器。

这点对企业用户特别重要。你的代码、数据、API凭证，全都在自己手里。

而且代码开源，你可以自己审计：

https://github.com/mcp-router/mcp-router

不放心可以自己看源码，或者让技术团队review一遍。

## 写在最后

老金我用了两个月MCP Router，越用越觉得它不只是个工具。

它让我重新思考：AI时代，人类价值到底在哪？

以前我们觉得，"会配置工具"就是专业。

花半天时间研究怎么写mcp.json，觉得自己很厉害。

但现在我明白了：

真正的价值，不是会折腾工具，而是用工具创造价值。

你看现在的AI工具链：

Claude Code、Cursor、Codex、Cline、Kilo......工具越来越多

brave-search、exa、context7、Github、playwright......服务器越来越多

配置文件、API Key、环境变量......要管理的东西越来越复杂

这个趋势不会停。

未来只会有更多AI工具，更多MCP服务器，更复杂的配置。

如果你还在手动改配置文件，手动管理几十个服务器，你的时间就被这些"重复劳动"吃掉了。

而MCP Router做的，就是把你从这些琐事里解放出来。

它让你：

不用再担心配置混乱

不用再为切换项目发愁

不用再为排查问题头疼

你的精力，应该用在创造上，而不是配置上。

老金我现在用AI写公众号，一键切换到"创作模式"，所有搜索、记忆、思考工具全开。

写代码做技术研究，一键切换到"工作模式"，文档、代码、工作流工具全上。

这才是AI时代应该有的工作方式：

工具为人服务，而不是人被工具折腾。

AI正在改变我们的工作方式。

但工具再多，本质不变——人的创造力，才是最大的价值。

MCP Router帮你管理工具，你去创造价值。

这才是正确的分工。

就比如，我又鼓捣个新玩意了。

对于不懂代码，不会英语的人来讲，每次创建个这种新玩意，成就感是极大的。

---

**往期推荐：**

[提示词工工程（Prompt Engineering）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=4120385726238392327#wechat_redirect)

[LLMOPS(大语言模运维平台)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3171759118513111043#wechat_redirect)

[WX机器人教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3502843007181520907#wechat_redirect)

[AI绘画教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3192433076551843848#wechat_redirect)

[AI编程教程列表](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NzU2MDgyNA==&action=getalbum&album_id=3704202865347362819#wechat_redirect)

    

---

谢谢你读我的文章。

如果觉得不错，随手点个赞、在看、转发三连吧🙂

如果想第一时间收到推送，也可以给我个星标⭐～谢谢你看我的文章。

开源知识库地址：

https://tffyvtlai4.feishu.cn/wiki/OhQ8wqntFihcI1kWVDlcNdpznFf

扫码**添加下方微信（备注AI）**，拉你加入**AI学习交流群**。
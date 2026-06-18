> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 告别拖拽画图！Claude Code 一句话生成draw.io流程图
author: AI智汇派
date:
url: https://mp.weixin.qq.com/s?__biz=MzI2ODA4NDYyMw==&mid=2648130324&idx=1&sn=b372bd34a79688e66765365e74288142&chksm=f3e82bd449ffd75f5bb56ee950e7f09e18c1d29f3da150c5bcca9c6fb01cd7830f701f96aeb8&mpshare=1&scene=24&srcid=0424uzItJ8m9N85J2EwL9Ldi&sharer_shareinfo=98fb3da081605d5636e90e570729ede1&sharer_shareinfo_first=98fb3da081605d5636e90e570729ede1#rd
---

**先说重点：以后画流程图，可能真的不用动手了。**

不是夸张。draw.io那个开发者最常用的免费画图工具，官方刚放了个大招：直接让Claude帮你画图。你描述需求，AI生成XML，文件自动打开，连拖拽都省了。

这个项目叫 **drawio-mcp**，GitHub上已经3200+ Star，是draw.io团队亲手做的，绝对不是第三方。

---

### 一个真实的使用场景

想象这个场景：你刚开完需求评审会，需要把讨论的业务流程整理成图。

以前的做法：打开draw.io，拖几个矩形，连几条线，调半天对齐，一抬头半小时过去了。

现在的做法：在Claude Code里敲一行命令，

```
/drawio create a flowchart for user login
```

10秒后，一个完整的登录流程图出现在你面前，文件自动保存为 `.drawio` 格式，还能直接拖回去继续改。

**省下的时间，够你多写两个接口了。**

---

### 为什么是MCP？

这事能成，离不开Anthropic去年推的 **Model Context Protocol（MCP）**。

简单说，MCP就是AI和外部工具之间的"通用插座"。以前让AI调工具，每家API都不一样，对接麻烦。MCP统一了标准，AI模型按这个协议就能调用各种服务，查数据库、读文件、现在还包括画图。

draw.io官方踩准了这个点，把自家的图表能力封装成MCP服务。结果是：Claude不仅能"说"出一张图，还能直接操作draw.io的编辑器、搜索专业图标库、导出各种格式。

**这不是简单的文本生成，是AI真正"操作"了一个工具。**

---

### 四种用法，对号入座

drawio-mcp提供了四种接入方式，差别主要在**你在哪用Claude**。

| 方式 | 适合谁 | 核心体验 |
| --- | --- | --- |
| **MCP App Server** | 用Claude.ai网页版的人 | 图表直接嵌在聊天窗口里，能缩放、能切图层 |
| **MCP Tool Server** | 用Claude Desktop的人 | 生成后在浏览器自动打开draw.io编辑器 |
| **Claude Code Skill** | 本地开发者（推荐） | 命令行直接画图，零依赖、零网络请求 |
| **Project Instructions** | 临时用一次的人 | 复制一段指令粘贴就行，什么都不用装 |

**重点说说第三种**，Claude Code Skill。因为我用的就是第3种：）

安装只要复制一个文件：

```
mkdir -p ~/.claude/skills/drawio
cp drawio/SKILL.md ~/.claude/skills/drawio/SKILL.md
```

然后直接开用：

```
/drawio sequence diagram for API auth
/drawio png class diagram for the models in src/
/drawio svg ER diagram for e-commerce
```

**不需要启动服务，不需要联网，甚至连npm install都不用。**

---

### 导出格式的隐藏技巧

用Claude Code Skill时，你可以在命令里直接指定格式。但有个细节很多人没注意到：

**PNG、SVG、PDF这三种格式，里面都内嵌了原始的XML数据。**

这话怎么讲？你导出一张PNG发给同事，对方把图片拖回draw.io，照样能编辑。不是只存了张图片，而是**带着源文件的图**。

这对需要频繁修改的文档特别实用，版本迭代了，图不用重画，拖回去改就行。

---

### 10000+图标库，专业度有保障

画架构图最怕什么？图标不专业。AWS服务画成普通方块，Kubernetes组件全靠文字标注，看起来就业余。

drawio-mcp内置了 `search_shapes` 工具，能检索draw.io的全部图标库：AWS、Azure、GCP、Kubernetes、UML、BPMN……**超过10000个形状**。

AI调用这个工具后，生成的图会自动用对图标。Lambda就是Lambda的图标，Pod就是Pod的图标，不用你手动去翻库。

---

### 这背后意味着什么？

draw.io官方亲自下场做AI集成，信号很明显：**画图工具的交互范式正在转移**。

过去二十年，我们习惯了"拖拽式"画图，鼠标是核心交互。但MCP+AI的组合，让"声明式"画图成为可能：你描述要什么，系统直接生成。

这不是说鼠标会消失。精调布局、微调样式，手动操作仍有价值。但**第一稿的生产方式**，确实变了。

对开发者来说，这个变化很实在：写代码的间隙，用自然语言快速出图，思路可视化，沟通成本降低。不需要在"画图工具"和"代码编辑器"之间频繁切换上下文。

---

### 快来试试吧

项目完全开源，GitHub地址：

**https://github.com/jgraph/drawio-mcp**

四种方式如何使用，在文档写得很清楚，选一种适合你的，几分钟就能跑起来。

如果你已经在用Claude Code，推荐直接上Skill方式，复制一个文件，立刻体验用自然语言画图的效率。

---

**最后留个思考题：**

如果画图能靠AI代劳，那产品经理的需求文档、架构师的设计说明，会不会也跟着变？当"产出图"的门槛趋近于零，**我们花时间精修的，应该是什么？**

欢迎在评论区聊聊你的看法。
---
title: 详解 Agent Skills, Rules, Subagents
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475452&idx=1&sn=3b1c296f79244bb3bf430bcd4f5c64e2&chksm=8f38845db54baa04e1eaf05d05b953ef914448d25b159d3f6049e7fdbe9a8d7d2e61725937f7&mpshare=1&scene=24&srcid=0116JvGHbkgyrfVU5WbswIEF&sharer_shareinfo=0f4eda07e930db7a4ee2d7e4bb94eeea&sharer_shareinfo_first=0f4eda07e930db7a4ee2d7e4bb94eeea#rd
---

👀 最新、最有用的AI编程姿势，总来自「知识药丸」

Rules、Commands、MCP Servers、Subagents、Modes、Hooks、Tools……用AI写个代码这么多配置项，谁看谁懵逼

这都是些啥玩意儿?为什么要搞得这么复杂?我就是想让 AI 帮我写点代码，怎么感觉像在搓火箭?

好吧，看完这篇文章后，我突然醍醐灌顶——原来这些东西背后有一条**清晰的演进逻辑**，而且最终它们会收敛到一个优雅的解决方案。我们来一起从技术演进的角度理清楚这些东西。

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式

---

 

### 起点:Rules — 给健忘的 AI 一份备忘录

#### 问题从哪来?

早期的 AI 模型有个致命问题:**幻觉**(Hallucination)。

什么是幻觉?就是 AI 会一本正经地胡说八道。你让它写代码，它可能会编造一个根本不存在的 API;你让它遵守某个编码规范，它转头就忘了。

那要怎么解决呢?

#### Rules 登场

开发者们想出了一个简单粗暴的办法:把那些"AI 每次都会搞错的东西"写成一份文档，然后**在每次对话中都塞给 AI 看**。

这就是 **Rules 文件**的由来。

```
# .cursorrules 示例  
- 我们的代码库使用 TypeScript  
- 所有组件必须使用函数式组件，不要用 class  
- API 调用统一使用 axios，基础 URL 是 /api/v2  
- 记住:我们没有用户认证系统!别给我生成登录页面!
```

Rules 就像是给 AI 准备的"备忘录"，告诉它:"嘿，别忘了这些重要的事!"

没错，这个方案确实管用。但随着项目变大，Rules 文件也开始膨胀……

#### 进化:多个 Rules 文件

当一个 Rules 文件写到几千行时，维护起来就很痛苦了。于是大家开始**拆分 Rules**:

```
.cursor/  
  ├── rules/  
  │   ├── coding-style.md  
  │   ├── api-conventions.md  
  │   └── database-schema.md
```

不过本质上，这些 Rules 最终还是会**合并成一份静态上下文**(Static Context)，在每次对话开始时一股脑塞给 AI。

这里有个问题:**不是所有 Rules 都需要在每次对话中出现**。

比如我在写前端代码时，数据库的 Schema 规则其实没必要出现在上下文里，对吧?但在当时，AI 模型的工具调用能力还不够成熟，没法做到"按需加载"，只能先这么凑合着用。

我们记住这个痛点，后面会讲到它的解决方案。

### 第二阶段:Commands — 把工作流打包带走

#### 新的需求出现了

当你用 AI 编程助手用得越来越熟练，你会发现:**有些操作是重复的**。

比如我每次提交代码前都要做这几步:

1. 1. 让 AI 生成一个规范的 commit message
2. 2. 执行 `git commit`
3. 3. 自动创建 PR 并填写描述

如果每次都要手动输入这一串提示词，那也太麻烦了吧?

#### Slash Commands 闪亮登场

于是就有了 **Slash Commands**(斜杠命令)的概念。

```
/commit-and-pr
```

一个命令，搞定整套流程!

你可以把常用的 Prompt 打包成一个命令，甚至可以分享给团队成员，或者放进 Git 仓库里版本管理。这本质上就是**可复用的工作流**。

P.S. 我个人最喜欢的命令是 `/commit-and-pr`，真的超级省事!

#### 本质是什么?

Commands 其实还是**文本**(Prompt 的封装)，只不过它是"**按需加载**"的——只有当你主动调用时，它才会被添加到上下文中。

但很快，开发者们就不满足于"只能写 Prompt"了。他们想要:**让 AI 执行真正的代码**。

### 第三阶段:MCP Servers — 给 AI 装上"义肢"

#### AI 的局限性

AI 模型虽然很聪明，但它有个硬伤:**它只能生成文本**。

它不能:

* • 访问你的数据库
* • 读取 Slack 消息
* • 在 Linear 里创建 Issue
* • 连接外部 API

这就好比一个聪明的大脑，但缺少手脚。

#### MCP 是什么?

MCP(Model Context Protocol，模型上下文协议)是一个开源标准，用于连接 AI 应用与外部系统。你可以把它理解成 **AI 的"USB-C 接口"**——就像 USB-C 为电子设备提供了标准化的连接方式，MCP 为 AI 提供了标准化的"插件系统"。

通过 MCP Server，AI 可以:

* • 读取 Google Drive 里的文档
* • 在 Slack 里发送消息
* • 查询 PostgreSQL 数据库
* • 操作 GitHub 仓库
* • ……甚至控制浏览器(Puppeteer)

这简直是**给 AI 装上了义肢**!

#### 一个具体例子

假设你让 AI 帮你:"在数据库里找到最新的销售报告，然后发邮件给我的经理。"

AI 会通过 MCP 客户端发现可用的工具，比如 `database_query` 和 `email_sender`，然后依次调用这些工具完成任务。

整个过程是这样的:

1. 1. **工具发现**:AI 发现有数据库查询工具和邮件发送工具
2. 2. **第一次调用**:查询数据库，获取报告数据
3. 3. **第二次调用**:发送邮件，附上报告内容
4. 4. **返回结果**:"我已经找到最新报告并发给你的经理了"

**是不是很酷?**

#### 但也有代价

MCP Servers 带来了强大的能力，但也有个问题:**上下文膨胀**。

如果你安装了 10 个 MCP Server，每个暴露 10 个工具，那就是 100 个工具的说明文档要塞进上下文里。这会严重影响性能和准确性。

别急，后面会讲到解决方案。

### 第四阶段:Modes & Subagents — 给 AI 换上不同的"人格面具"

#### 更复杂的需求

有时候，你不只是想给 AI 加能力，你还想**改变它的行为模式**。

比如:

* • 我希望 AI 在"规划模式"下只做架构设计，不写具体代码
* • 我希望 AI 在"调试模式"下聚焦于错误排查
* • 我希望 AI 在处理某个子任务时，只能访问特定的工具

这就催生了 **Modes**(模式)和 **Subagents**(子代理)的概念。

#### Subagent 是什么?

Subagent 就像是给 AI 设定一个**临时人格**:

```
# Subagent: 前端开发专家  
你是一个前端开发专家，专注于 React 和 TypeScript。  
你只能使用以下工具:  
- 文件编辑  
- npm 命令  
- ESLint 检查
```

通过限制工具范围和调整 Prompt，Subagent 可以让 AI 更专注、更可靠。

#### Mode 更进一步

Mode 不仅改变指令，还能:

* • 修改系统提示词
* • 提供特定的 UI 元素(比如"计划视图")
* • 在提示中添加"记忆提醒"(让 AI 记住当前模式)

其实 Mode 和 Subagent 的核心目标都是:**提高可靠性和可发现性**。

但这些还是"软约束"——AI 仍然可能出错。那有没有**硬约束**呢?

### 第五阶段:Hooks — 给 AI 加上"强制检查点"

#### 什么是 Hook?

Hook 是**确定性的、100% 可靠的执行点**。

和前面那些"AI 可能听你的，也可能不听"的机制不同，Hook 是**必定会执行**的脚本。

典型用法:

**Before Hook**(每次对话开始前):

```
// 自动注入当前项目配置  
injectProjectConfig();
```

**After Hook**(对话结束后):

```
// 记录日志  
logConversation();  
// 自动保存到数据库  
saveToDatabase();
```

Hook 让你可以在 AI 的工作流中插入**可控的逻辑**，这对于安全、审计、集成外部系统都非常有用。

### 终极形态:Skills — 化繁为简

#### 混乱的现状

到这里，我们已经有了:

* • Rules(静态上下文)
* • Commands(可复用提示词)
* • MCP Servers(外部工具集成)
* • Subagents(子任务代理)
* • Modes(行为模式)
* • Hooks(确定性脚本)

**这也太乱了吧!**

作为用户，我就是想让 AI 帮我写代码，为什么要理解这么多概念?

#### Skills:统一的答案

好消息是，这些概念正在收敛到一个优雅的解决方案:**Agent Skills**。

Skills 是一个开放标准，用于打包可复用的知识和脚本，扩展 AI Agent 的专业能力。

##### Skills 的两种形态

**基础形态**:就像 Commands，封装一个工作流

```
# git-pr-flow.skill.md  
自动生成 commit message、提交代码并创建 PR
```

**高级形态**:可以包含脚本、可执行文件、资源文件——**任何你想打包的东西**

```
my-skill/  
  ├── SKILL.md        # 技能说明  
  ├── scripts/        # 可执行脚本  
  └── assets/         # 资源文件
```

##### 为什么 Skills 更好?

1. 1. **不会膨胀上下文**:Skills 只在需要时才加载
2. 2. **易于分发**:一条命令就能安装给整个团队
3. 3. **开放标准**:任何 AI Agent 都可以支持

```
# 一键安装技能  
npx ai-agent-skills install frontend-design --agent cursor
```

#### 与 MCP 的关系

你可能会问:那 MCP 呢?还用吗?

**当然用!**

Cursor 对 MCP 进行了优化，如果你安装了 10 个 MCP Server，每个有 10 个工具，系统会按需加载工具，而不是一次性塞进上下文。

Skills 和 MCP 不是竞争关系，而是互补的:

* • **Skills**:更轻量，适合打包工作流和知识
* • **MCP**:更强大，支持 OAuth 等高级功能

### 最佳实践:我应该关注什么?

好吧，讲了这么多历史，作为普通用户，我到底该怎么用?

#### 核心原则:只关注两个东西

作为 Coding Agent 的用户，你只需要关注:

1. 1. **Rules**(静态上下文)
2. 2. **Skills**(动态能力)

其他的交给工具自己优化就行。

#### Rules 的最佳实践

**保持精简、高质量**。

Rules 会包含在每次对话中，所以不要写成一本书。只写那些"AI 经常搞错"的核心规则。

我的习惯是:

* • 当 AI 犯错时，在 PR 上 @cursor，让它更新 Rules
* • Rules 是"活的文档"，随着项目演进而更新

```
# 好的 Rules  
- 使用 TypeScript strict 模式  
- 组件文件命名:PascalCase.tsx  
- API 基础路径:/api/v2  
  
# 不好的 Rules(太啰嗦)  
- TypeScript 是一种强类型的 JavaScript 超集……(省略 500 字)
```

#### Skills 的未来

Skills 还很新，现在还没有太多"最佳实践"。

但我相信，在未来 6 个月里，随着生态的建立，Skills 会变得越来越重要。

就像前端的 npm 包一样，会有一个庞大的"Skills 生态系统"供你选择和安装。

### 总结

让我们回顾一下这段进化史:

1. 1. **Rules**:静态上下文，每次都加载
2. 2. **Commands**:可复用的 Prompt
3. 3. **MCP Servers**:连接外部系统，给 AI 赋予真实能力
4. 4. **Modes & Subagents**:调整 AI 行为模式
5. 5. **Hooks**:确定性的执行点
6. 6. **Skills**:统一的解决方案，化繁为简

最终，这一切都收敛到两个核心概念:

* • **静态上下文**(Rules)
* • **动态能力**(Skills)

这个演进过程并不是"瞎折腾"，而是 AI 编程助手在不断成熟的过程中，逐步找到最优解的探索之路。

从一团乱麻，到优雅清晰——这就是技术进步的魅力。

现在，当你再看到 Rules、MCP、Skills 这些概念时，是不是清楚多了?

P.S. 如果你也在用 Cursor 或其他 AI 编程助手，建议现在就开始建立你的 Rules 文件。从小事做起，每次 AI 犯错就记录一条规则，慢慢积累，你会发现 AI 的表现越来越符合你的期待。

### 参考资料

* • Model Context Protocol 官方文档
* • Anthropic: Introducing the Model Context Protocol
* • Cursor Agent Skills 文档
* • Agent Skills 开放标准

 

---

 坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～
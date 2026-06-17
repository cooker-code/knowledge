---
title: Playwright 的 Test Agent 并不神秘：弄懂 subagent ，你也能复刻
author: 测试技术小宇宙
date: 测试技术小宇宙测试技术小宇宙
url: https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247484009&idx=1&sn=2ced3240742994f9610c03929b8ffc05&chksm=c106e0ae80b6c0ead4ce52e72340e4e2e48541773755b43ea952c3436ef6f862c6a1788bca03&mpshare=1&scene=24&srcid=0522YGyZ6YrJSQxzWLS4T4Ul&sharer_shareinfo=d48c68fdb88e30a839da3156f009cb56&sharer_shareinfo_first=d48c68fdb88e30a839da3156f009cb56#rd
---

> 在 AI 转型的热潮下，很多企业内部都在建设 skill，"蒸馏同事"。我所在的团队也不例外，skill 中心的技能都有十几页了。不过却很少看到 subagent 相关的实践。
>
> 其实 subagent 也是一个很强大的能力。

Playwright 官方在去年底推出了3个自动化测试的 Agent 能力。 最初以为是什么黑科技，实际体验下来，其本质是由 **3个 Markdown 文件** 驱动的 **`subagent（子代理）`**。

本文主要弄懂三个问题： 

* subagent（子代理） 到底是什么？
* subagent 如何与 skill 协作？

* **Playwright Test Agent 的本质**

---

## 一、认识 subagent

如果你经常遇到`上下文窗口爆了`、`任务无法并行,执行慢`，subagent 就是解决这类问题的。

### 1.1 核心概念

skill 和 subagent 都是非常优秀的工程实践。两者概念和解决的问题是不同的。

由于它们都是基于 Markdown 文件驱动，有部分场景也能相互替代与协作，对于新手来说可能容易混淆。

* **skill**：用于给 agent 增加技能，但是活儿都是主 agent 自己干，掌控所有细节。
* **subagent**：相对于主 agent 的概念。主 agent 可以把任务拆分为子任务，丢给一个或多个 subagent 去干。主 agent 只负责派发任务并收集结果，完全不关心 subagent 干活的过程和细节。

所以 subagent 有两个显著优势：

* **`上下文隔离`**。主 agent 不关心子任务的过程，自然就不会积累一些无关的上下文。避免上下文过大带来的 **`响应慢`**、**`注意力分散`** 等问题。
* 支持多个 subagent **`并行`**，加速复杂任务执行效率。

subagent 也是一个 agent，也能调用 skill。所以`两者是可以配合使用的`。

| 特性 | Skill | Subagent |
| --- | --- | --- |
| 本质 | 任务模板和规则 | 独立执行的 AI 分身，也可以定义模板与规则 |
| 上下文 | 共享主会话上下文 | 独立上下文窗口 |
| 触发 | Claude 自动判断，或手动 `/skill-name` | Claude 自动判断，或 `@subagent-name` |
| 适用场景 | 定义"怎么做"的流程规范 | 定义"谁来做"的执行者 |

### 1.2 subagent 适用的场景

* 需要阅读大量代码才能完成的任务（做规划、代码审查 等）
* 需独立判断、不希望被主会话上下文影响的任务（如 Code Review）
* 可以并行执行的多个独立子任务

### 1.3 内置的 subagent

很多 Agent 工具都内置了一些 subagent。以 Claude Code 为例, Explore 和 Plan 就是2个高频使用的 subagent：

* **Explore**：只读的搜索 agent，适合快速定位和理解代码结构
* **Plan**：继承主会话模型的只读规划 agent，进入 `/plan` 模式时自动触发

---

## 二、自定义一个 subagent

当内置 subagent 无法满足需求时，就可以创建自己的 subagent。

以 Claude Code 为例，创建过程和 skill 非常相似。

### 2.1 定义文件格式

subagent 定义文件存放于以下两个位置之一：

```
.claude/agents/    # 仅在当前项目生效，可提交 git 共享  
~/.claude/agents/  # 全局生效，所有项目可用
```

每个 subagent 是一个 `Markdown 文件`。和 skill 的格式高度相似，也可以定义它的工作流。不过配置会更加精细，可指定它的 工具、模型、权限、颜色、记忆 等。

```
---  
name:code-reviewer  
description:代码审查专家。在代码变更后主动审查质量、安全性和可维护性。  
tools:Read,Grep,Glob,Bash  
model:sonnet  
permissionMode:dontAsk  
---  
  
你是一位高级代码审查员，确保代码质量和安全性的高标准。  
  
被调用时：  
1.运行gitdiff查看最近的变更  
2.专注于被修改的文件  
3.立即开始审查
```

几类重要属性：

* tools：工具白名单；设置后只能使用列出的工具，MCP 工具也会被排除
* permissionMode：权限控制
* memory：持久记忆范围：user / project / local
* background：设为 true 时，该代理始终以后台方式运行（不阻塞主对话）
* skills：在此代理启动时自动加载的 Skills 列表

完整属性可参考官方说明：https://code.claude.com/docs/zh-CN/sub-agents

### 2.2 自动创建 subagent

和 skill 创建一样，通常不需要自己完整编写它。可通过 `/agents`命令，管理、创建 subagent。

只需给出对于 subagent 的描述，就能自动生成。

自动创建的 subagent 内容

### 2.3 触发方式

* **`自动触发`**：与 skill 机制类似，Claude Code 会自动判断何时需要调用某个 subagent。
* **`显式调用`**：也可以通过 `@subagent-name` 的方式手动触发某个 subagent

注：如果是 Agent Teams，配置会更复杂。以后有机会再介绍。

执行过程中会列出执行的 agent 列表，可手动切换到 subagent 查看执行过程。

可以看到，subagent 工作时是不会显示具体的执行过程，主 agent 只关心结果。

---

## 三、skill 与 subagent 配合使用

subagent 本质也是一个 agent，也可以访问 skill，所以两者可以配合使用。

### 3.1 方式一：skill 驱动 subagent

在 skill 的属性中添加 `context: fork`，调用该 skill 时自动在 subagent 中执行，执行完只返回摘要，不污染主会话。

优点很明显，当主 agent 在执行该 skill 时，永远都是委派给 subagent 完成，过程数据（比如工具调用）不会占用主会话的上下文。

#### 案例：subagent 探索网页，生成 XPath

之前实现过一个生成 XPath 的 skill，因为需要频繁与 Chrome MCP 交互，过程中产生数据非常多，生成几次就会明显变慢。

改成以 subagent 模式运行后，主 agent 上下文中就屏蔽了这些复杂的过程数据，相当于主 agent 的一个独立工具。

与 Chrome 的交互都在 subagent中，不占用主 agent 的上下文。

### 3.2 方式二：subagent 预加载 skill

在 subagent 执行过程中，其实会自动判断使用 skill。也可以在它的属性中添加 `skills` 字段，为 subagent 装备领域知识。

#### 举例

`api-doc-generator` 启动时会自动加载这两个 skill 的完整内容，无需在每次调用时重复描述规范。

```
# .claude/agents/api-doc-generator.md  
---  
name:api-doc-generator  
description:GeneratescomprehensiveAPIdocumentationfromcode  
tools:Read,Write,Glob  
model:sonnet  
skills:  
-api-standards# 预加载 Skill 作为领域规范  
-deep-research# 预加载 Skill 支持深度研究  
---  
  
YouareanAPIdocumentationexpert.Youhaveaccesstothe`api-standards`skillforformattingconventionsand`deep-research`forunderstandingAPIstructure.  
  
1.Use`deep-research`tounderstandtheAPImodulearchitecture  
2.Generatedocumentationforeachendpointfollowing`api-standards`conventions  
3.Includerequest/responseexamples,errorcodes,andauthenticationnotes
```

---

  

## 四、理解 Playwright 的 Test Agent

了解了 subagent 的概念，再回头看 Playwright 的 Test Agent 就比较好理解了。

### 4.1 安装

Playwright 官方提供了一键安装命令，官方适配了3种 agent，根据需要选择。

```
npx playwright init-agents --loop=vscode  
npx playwright init-agents --loop=claude  
npx playwright init-agents --loop=opencode
```

安装完成后就得到了对应的 3个 agent 文档，1个用例模板、1个默认的 playwright-mcp 配置。

之后，就能直接调用对应的 agent 能力了，使用过程和 skill 区别不大。

### 4.2 agent 的内容

我们再来看看 Playwright 的 agent 定义。

以 planner agent 为例，去掉工具定义，一共不到 50 行。

注：原版为英文，此处翻译为中文方便理解

```
---  
name: playwright-test-planner  
description: 当需要为 Web 应用或网站创建全面的测试计划时使用此 agent  
tools: Glob, Grep, Read, LS, mcp__playwright-test__browser_click, ...  
model: sonnet  
color: green  
---  
  
你是一名资深的 Web 测试规划专家，在质量保证、用户体验测试和测试场景设计方面拥有丰富经验。你精通功能测试、边界用例识别和全面的测试覆盖率规划。  
  
你的工作流程：  
  
1. **导航与探索**  
- 在使用其他工具之前，先调用 `planner_setup_page` 工具初始化页面  
- 通过浏览器快照探索页面结构  
- 除非绝对必要，否则不要截图  
- 使用 `browser_*` 系列工具导航和发现界面元素  
- 全面探索界面，识别所有可交互元素、表单、导航路径和功能模块  
  
1. **分析用户流程**  
- 梳理主要用户旅程，识别应用中的关键路径  
- 考虑不同用户类型及其典型行为模式  
  
1. **设计全面的测试场景**  
  
创建详细的测试场景，覆盖：  
- 正向场景（正常用户操作流程）  
- 边界条件和极端情况  
- 错误处理和输入验证  
  
4. **组织测试计划结构**  
每个场景必须包含：  
- 清晰、描述性的标题  
- 详细的逐步操作说明  
- 适当的预期结果  
- 初始状态假设（始终假设为空白/全新状态）  
- 成功和失败的判定标准  
  
4. **生成文档**  
使用 `planner_save_plan` 工具提交测试计划。  
**质量标准**：  
- 编写的步骤应足够具体，确保任何测试人员都能直接执行  
- 包含负面测试场景  
- 确保各场景之间相互独立，可按任意顺序执行  
  
**输出格式**：始终将完整的测试计划保存为 Markdown 文件，包含清晰的标题层级、编号步骤和专业格式，便于与开发和 QA 团队共享。
```

是不是和 skill 高度相似？

---

## 五、小结

Playwright 的 Test Agent 其实已经非常清晰了，本质上就是定义了 3个比较通用的工作流，配合 playwright mcp 工具完成任务。

至于 agent 的实际效果，本文不做评价。

因为当你真正了解了它的机制，完全可以结合自己的项目特点，快速搭建出 **`更适合项目的 subagent`**。

---

### 附

如果想了解 Playwright Test Agent 的另外两个 agent 内容，可访问该链接：

https://github.com/liangqiangWang/woody-article-demo/tree/main/playwright-agent/.claude/agents/playwright-test-healer.md

---

延伸阅读：

* [Agent 终于能实时控制 Chrome 了！还能集成到 UI自动化中](https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247483995&idx=1&sn=6198b381699992f1e7badbcca0032033&scene=21#wechat_redirect)
* [Claude Agent SDK 实践——搭建一个测试用例生成 agent，支持 skill](https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247483994&idx=1&sn=ec48069fc4ddf8f5d6f8db93ead48625&scene=21#wechat_redirect)
* [推荐新人掌握的 Claude Code 使用技巧，以及一个彩蛋](https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247483981&idx=1&sn=0d1cbc3443fc73517391374bc2d60a66&scene=21#wechat_redirect)
* [Git worktree，让 ClaudeCode  并行开发不冲突](https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247483941&idx=1&sn=ce01f2159c7af48a06c9eb95f4f070a9&scene=21#wechat_redirect)
* [Andrej 大神力荐的 NanoClaw 怎么用？20 分钟安装上手，并对接飞书](https://mp.weixin.qq.com/s?__biz=MzkwNTIyMTQ1Mw==&mid=2247483871&idx=1&sn=c8e2979ca5af4880977c057a24ec37ba&scene=21#wechat_redirect)
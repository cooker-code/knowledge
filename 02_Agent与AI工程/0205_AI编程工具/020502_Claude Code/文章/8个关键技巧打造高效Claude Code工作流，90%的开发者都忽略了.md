---
title: 8个关键技巧打造高效Claude Code工作流，90%的开发者都忽略了
author: 墨痕AI编程
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzODAyMzU5Nw==&mid=2247483768&idx=1&sn=597135a4b4ca9fa40bd8eb6915c6f9ca&chksm=f19142d7197ae286af20e850cde842092a802931dd33c5e78ca0a2f0fcfd345a83cdbf7e98d9&mpshare=1&scene=24&srcid=1107ZuCCLLNrHGx8UE0o9MrF&sharer_shareinfo=f558267bdb2bcc5d6faa6a3cb29350c1&sharer_shareinfo_first=f558267bdb2bcc5d6faa6a3cb29350c1#rd
---

不知不觉使用Claude Code编程已经三个多月了，国庆过后决定开通公众号来分享一些AI编程的心得体会。

不知不觉已经过了账号也开通一个月了，也慢慢的养成了我记录和分享的习惯，这让我非常受用，也更加坚持自己深耕AI编程赛道的决心。

虽然在Claude Code编程上的应用程度还是很有限，但也有一些经验心得，总结了一些小技巧分享给大家。

技巧1：自己掌握开发环境权限

我看过许多教程为了省事，会直接推荐用户使用 `--dangerously-skip-permissions` 命令来启动Claude Code。

虽然这样能避免一些权限问题出现的反复确认的情况，但是权限过大可能会不经确认就删除重要文件、出现一些故障，尤其是多人写作开发的时候。

当然也可以根据自己的决定启动方式，没有固定的方式。

Claude Code内置了 `/permissions` 命令，允许你为不同工具设置不同的权限级别：`always-allow`（总是允许）、`always-ask`（总是询问）和 `never-allow`（永不允许）。

技巧2：做好上下文配置

很多人喜欢Claude Code的一个原因就是它能理解整个项目上下文，其实就是一个 `CLAUDE.md` 文件。如果这个文件信息混乱或缺失，Claude就会自由发挥，可能不能按照自己的要求完成任务。

正常情况下我们还是要优化自己的CLAUDE.md。

一般有两种：“全局配置”与“项目配置”。

全局配置 (`~/.claude/CLAUDE.md`)：存放自己编码风格、常用工具偏好等跨项目的通用信息。

项目配置 (`./CLAUDE.md`)：存放当前项目的信息，如项目目标、技术栈、架构设计、编码规范等，项目配置的优先级更高。

举一个项目级CLAUDE.md模板例子

```
项目：智能笔记应用  
##项目概述目标：开发一个支持Markdown实时同步和AI智能整理的个人笔记应用。技术栈：React 18, TypeScript, Vite, Tailwind CSS, Zustand (状态管理), Supabase (后端)。包管理器：pnpm  
## 编码规范命名：组件使用PascalCase，文件名使用kebab-case。常量使用UPPER_SNAKE_CASE。TypeScript：开启严格模式，不要使用`any`，优先使用`interface`定义对象类型。CSS：使用Tailwind CSS，不要写内联样式。组件样式使用CSS Modules或Tailwind的@apply指令。  
## 常见命令pnpm dev：启动claude开发服务器；pnpm build：构建生产版本；pnpm type-check：运行类型检查  
## Git Hooks提交前要自动运行 pnpm lint 和 pnpm type-check。提交信息格式按照 Conventional Commits。  
## 重要提醒所有API的调用必须有错误要处理。新增页面或组件必须在路由文件中注册。敏感配置（如API密钥）必须使用环境变量，严禁使用硬编码。
```

技巧3：灵活节省开发成本

很多人做AI编程追求高性能方式，哪个模型强大用哪个。这样虽然效果最好，但成本也会飙升，一个小项目下来估计要花费上百元在token上。 

Claude Code有一个“Opus规划模式”。使用的时候，Claude会自动根据项目信息，在不用的阶段使用不同的模型。

比如：在规划阶段调用自动强大的Opus模型，在执行编码任务时切换回比较便宜的Sonnet模型，这样灵活的切换能帮我们节省很多成本。

我的个人习惯是：日常开发用Sonnet，只有在遇到卡壳超过15分钟的难题时，才会手动切换到Opus处理。

技巧4：善于利用Claude的深度推理模式

当开发中遇到比较复杂任务时，比如需求过于复杂或者周边交互频繁的情况下，Claude的默认思考深度可能不够。这个时候我们可以考虑使用深度的思考模式。

思考级别：think < think hard < think harder < ultrathink。

使用也很简单，在处理一个复杂的并发问题时，在提示词末尾加上：

```
请分析这个多线程读写锁的潜在死锁问题，并提出解决方案。ultrathink
```

成本警告：ultrathink 会让Claude花费极大的思考预算（高达31999个tokens），导致成本显著增加。

之前有个例子：用ultrathink模式计算“1+1等于几”就花费了0.34美元！普通人谨慎使用！

技巧5：使用Subagents实现专业分工

关于Subagents（子智能体）我之前写过一篇文章[别再让 AI 当“实习生”了！Claude Code 的 Subagents 才是真正的开发团队](https://mp.weixin.qq.com/s?__biz=MzYzODAyMzU5Nw==&mid=2247483723&idx=1&sn=bd62abb92763218df7d1375865dab4c1&scene=21#wechat_redirect)，有兴趣的可以去看看。

我们可以创建多个具备专门知识的Subagents，利用独立的上下文窗口，可以进行并行或串行工作。

这样做不仅可以使得主线程保持专注，子线程独立运行，不会打断正在运行的任务。

技巧6：善用MCP

现在随着ClaudeCode的生态逐渐强大，MCP慢慢被边缘化了，尤其是skill出现以后，已经开始有了从互补到代替的发展趋势。

但现在MCP还有不能被代替的场景，我们使用MCP的时候配置不宜过多，否则会挤占宝贵的上下文空间。

我推荐两个“必装”的高价值MCP，帮你完善AI编程方法。

1.  Chrome DevTools MCP：可以控制浏览器。主要用于前端调试、UI自动化测试、网页数据抓取。

```
安装命令：claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

2.  Context7：让AI能及时获取最新的技术知识。主要是解决大模型最新知识的截止日期，也可以查询最新的API文档、技术博客。

```
安装命令：claude mcp add context7 npx context7
```

安装后，我们可以让Claude直接打开浏览器访问我的网站，可以用于检查登录按钮的样式是否正确，或者查询最新文档，挺好用的。

技巧7：合理管理上下文

项目复杂了，长时间的会话会导致上下文窗口溢出，Claude会忘记刚开始的的关键信息。这个时候我们就要像管理内存一样及时清理对话的上下文。

我推荐两个命令：/clear和 /compact

/clear：彻底清除所有对话历史。适用于完成一个任务后，开始一个全新的、不相关的任务。

/compact：将现有对话总结压缩，保留关键信息。一般在一个长会话中，当上下文接近上限时，保留关键信息继续对话。

做一个简单的比喻：/clear就是清空回收站，/compact就像文件整理。

技巧8：Agent Skills实现工作流复用

我们在实际开发一定要有自己的工作流，这样才能高效的完成开发，Agent Skills就能把我们积累的工作流封装成一个个可重复使用的模块。

我们做一个实例：创建一个“自动生成PR描述”的Skill

1.  在项目根目录下创建 

.claude/skills/generate-pr-description/SKILL.md文件。

2.  写入以下内容：

从此，生成PR描述从一件需要5分钟指导的琐事，变成了10秒内自动完成的高效操作。 这就是复用工作流的魅力。

最近搜集了一些Claude Code编程的资料，需要的扫码自取
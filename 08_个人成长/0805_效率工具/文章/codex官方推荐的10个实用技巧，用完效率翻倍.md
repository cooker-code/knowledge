---
title: codex官方推荐的10个实用技巧，用完效率翻倍
author: 阿星AI工作室
date: 阿星AI工作室阿星AI工作室
url: https://mp.weixin.qq.com/s?__biz=MzU3NTE2NjIxMQ==&mid=2247502637&idx=1&sn=a55b614bf698842c6c0fdb0e10b02c20&chksm=fca4b8b3e6e5a05552ab9c9b73a9ed133130c0ebacbee98c5b07eb632b9555563614e6cc8164&mpshare=1&scene=24&srcid=0525wzzpoFUxv6Edp9i0EMm0&sharer_shareinfo=5acd260fe8131e4a27f1a22e5470d8fb&sharer_shareinfo_first=5acd260fe8131e4a27f1a22e5470d8fb#rd
---

哈喽，大家好

我是阿星！

最近我在加班学习codex。发现Codex 这次更新，表面上看是 Appshots、Goal mode、浏览器标注、插件共享、Analytics 这些功能一起上线。但真正的变化不是“功能变多了”。

真正的变化是：Codex 正在从一个你问一句、它答一句的代码助手，变成一个可以看上下文、跑长任务、操作浏览器、接入团队工具、持续完成目标的开发代理。下面这 10 个技巧就很关键。

## 1. 不要只说“帮我改”，要用四段式提示词

Codex 官方最佳实践里，其实已经把好提示词拆得很清楚：

Goal、Context、Constraints、Done when。

也就是目标、上下文、约束、完成标准。(信息来自OpenAI Developers)

普通写法：

帮我优化登录页

更好的写法：

Goal：优化登录页的移动端布局。  
Context：重点看 src/pages/login.tsx 和 src/components/AuthForm.tsx。  
Constraints：不要改接口逻辑，不要引入新的 UI 库，保持现有设计风格。  
Done when：移动端 375px 宽度下按钮不溢出，表单间距统一，现有测试通过。

这类提示词最大的好处是，Codex 不用猜。

你不是在让它“发挥”，你是在给它交付标准。

## 2. 长任务直接用 /goal，不要一轮一轮催

以前用 AI 写代码，最麻烦的是它做一半就停了，

你还得反复说“继续”“接着改”“再检查一下”。

现在 Goal mode 已经正式可用。

官方说它可以在 Codex App、IDE 插件和 CLI 中使用，

让 Codex 围绕一个明确目标持续工作，

甚至跑几个小时或几天。(信息来自OpenAI Help Center)

适合 /goal 的任务不是“写一个按钮”，

而是这种：

/goal  
把这个项目的登录、注册、忘记密码三个页面统一成新的设计规范。  
要求：

1. 1.不改变后端接口；
2. 2.保留现有表单校验；
3. 3.统一移动端样式；
4. 4.每完成一个页面后运行相关检查；
5. 5.最终给我总结改了哪些文件、哪些地方需要人工复核。

关键点：目标文本本身就是完成标准。

所以 /goal 不是写得越短越好，而是要写清楚什么叫“完成”。

## 3. 难任务先开 /plan，别一上来就让它动代码

如果任务复杂，最好先让 Codex 规划，而不是直接实现。

官方也建议复杂、模糊或者难描述的任务，先让 Codex plan，再动手。(信息来自OpenAI Developers)

你可以这样写：

先不要改代码。  
请先阅读项目结构，找出实现这个功能可能涉及的文件。  
然后给我一个修改计划，包括：

1. 1.需要改哪些文件；
2. 2.每个文件改什么；
3. 3.可能的风险；
4. 4.需要我确认的问题。  
   等我确认后再开始实现。

这一步看似慢，其实省时间。

因为 Codex 最容易翻车的地方，不是不会写代码，而是方向一开始就理解错了。

## 4. 用 Appshots 传上下文，少写一大堆废话

这次最实用的新功能之一是 Appshots。

在 macOS 的 Codex App 里，按两个 Command 键，

就能把当前最前面的应用窗口发给 Codex。

它不只是截屏，还会带上可读取文字。(信息来自OpenAI Developers)

这意味着你不用再这样描述：

页面右上角那个按钮旁边的间距有点怪，就是头像左边那个地方……

你可以直接 Appshot，然后说：

看这个界面。  
把顶部导航右侧的按钮组间距调得更自然一点，视觉上和左侧 Logo 对齐。  
不要改页面结构，只改样式。

对 UI、报错弹窗、配置界面、控制台输出、

设计稿预览这类场景，

Appshots 会大幅减少沟通成本。

## 5. 前端改样式，用浏览器标注，不要纯文字描述

这次 Codex 的 in-app browser annotations 也增强了。

官方说明里提到，

高级标注可以

直接针对字体大小、颜色、间距等样式问题

给出更精确的反馈。(信息来自OpenAI Developers)

这对前端特别有用。

以前你要说：

第二个卡片的标题太大，

按钮离底部太近，

图片比例不太对。

现在更好的方式是：

打开页面预览，直接在浏览器里标注对应元素：

这里字体缩小 2px。  
这里上下间距加 8px。  
这个按钮和左边文案居中对齐。  
这张图保持 16:9，不要拉伸。

这比写长 prompt 更准。

本质上，它让 Codex 从“听你描述页面”，

变成“看着页面改页面”。

## 6. 让 Codex 自己写测试、跑测试、解释测试结果

很多人用 Codex 最大的问题是：只让它生成，不让它验证。

官方最佳实践也提到，不要只让 Codex 做改动，

还要让它创建测试、运行检查、确认结果、审查 diff。(信息来自OpenAI Developers)

你可以直接加一句：

改完后请运行相关测试、lint 和类型检查。  
如果测试失败，先自己定位原因并修复。  
最后告诉我：

1. 1.跑了哪些命令；
2. 2.是否通过；
3. 3.还有哪些风险需要人工检查。

这句话非常重要。

因为你不是要一段“看起来能跑”的代码，

你要一个“经过验证的交付”。

## 7. 给项目写 AGENTS.md，把你的偏好固化下来

如果你每次都在提醒 Codex：

```
不要乱改目录结构。  
不要引入新库。  
接口文件别动。  
CSS 用 Tailwind。  
组件命名按项目规范来。
```

那说明你该写 AGENTS.md 了。

官方把 AGENTS.md 形容成给 agent 看的开放格式 README，

会自动加载进上下文。

它适合写项目结构、运行方式、测试命令、工程规范、禁止事项、完成标准等。(信息来自OpenAI Developers)

一个简单版本

可以这样写：

```
# AGENTS.md  
  
## Project rules  
- Do not introduce new dependencies unless explicitly approved.  
- Keep existing file structure.  
- Use TypeScript strict mode.  
- Follow existing component naming conventions.  
  
## Commands  
- Install: pnpm install  
- Dev: pnpm dev  
- Test: pnpm test  
- Lint: pnpm lint  
  
## Done means  
- Code compiles.  
- Relevant tests pass.  
- No unrelated files changed.  
- Summary includes changed files and risk notes.
```

这东西的价值是：

你不用每次重新教育 Codex。

## 8. 用多层 AGENTS.md 管住不同目录

还有一个很多人不知道的细节：AGENTS.md 可以放在不同层级。

官方说明里提到，可以有全局 ~/.codex/AGENTS.md，

也可以有 repo 级别、子目录级别的 AGENTS.md。

越靠近当前目录的说明优先级越高。(信息来自OpenAI Developers)

这就很适合复杂项目。

比如：

```
~/.codex/AGENTS.md  
个人通用偏好：回答中文、改动前先解释、不要擅自装依赖  
  
project/AGENTS.md  
项目级规则：技术栈、启动命令、测试命令、PR 标准  
  
project/apps/admin/AGENTS.md  
后台系统规则：Ant Design、权限逻辑不能动  
  
project/apps/mobile/AGENTS.md  
移动端规则：React Native、注意 iOS/Android 差异
```

这样 Codex 进入不同目录时，会自动拿到不同的工作规范。

这比每次手动解释高效得多。

## 9. 把 config.toml 配好

很多 Codex 问题不是模型不行，而是环境没配好。

官方建议把个人默认配置放在 ~/.codex/config.toml，repo 级配置放在 .codex/config.toml。

配置里可以管理模型、推理强度、沙盒、审批策略、MCP 等。(信息来自OpenAI Developers)

新手建议：权限别一开始开太大。

可以先保持默认审批和沙盒策略，只给它处理当前项目的权限。等你确认这个项目可靠、命令安全，再逐步放宽。

更实用的做法是给不同场景设置不同 profile：

```
# ==========================================  
# Profile：安全模式（用于新项目/陌生仓库）  
# ==========================================  
[profiles.safe]  
approval_policy = "on-request"  
# approval_policy = 执行命令前要不要经过你批准  
# "on-request" = 每次执行命令前都弹窗问你"能跑吗？"（最严格，适合不确定安全性的场景）  
  
  
# ==========================================  
# Profile：信任模式（用于熟悉、已验证的项目）  
# ==========================================  
[profiles.trusted]  
approval_policy = "on-request"  
# 官方推荐即使是信任项目也保持 "on-request"（手动批准）  
# 如果你确定要全自动（比如跑 CI 脚本、定时任务），可以改成：  
# "never" = 从不请求批准，Codex 直接执行（效率最高，风险最大，仅限非交互式自动化）
```

核心原则是：  
让 Codex 高效，但不要让它无边界。

## 10. 团队用 Codex，不要只看“生成了多少代码”，要看“节省了多少流程”

这次 Business 和 Enterprise 相关更新里，插件共享和 Analytics 很值得注意。

插件共享允许团队通过 marketplace sources 分发可复用插件包，里面可以包括 skills、应用集成和 MCP servers。(信息来自OpenAI Developers)

Analytics 升级则能看到活跃用户、积分、token、运行次数、用户排行榜、代码生成行数、插件使用情况等指标。(9to5Mac)

但企业真正该看的不是“Codex 写了多少行代码”。

更应该看：

```
哪些重复任务被自动化了？  
哪些测试、排查、修复流程变短了？  
哪些内部工具可以通过插件共享给全团队？  
哪些人已经把 Codex 融入真实工作流？  
哪些仓库最适合先做 agent 化改造？
```

也就是说，Codex 对团队的价值，不是多了一个写代码的人。

而是让团队里很多原本靠人肉复制、搜索、排查、跑命令的流程，

开始被系统性压缩。

# 总结

这次 Codex 更新，最值得关注的不是某一个单点功能。

Appshots 解决的是上下文问题。

Goal mode 解决的是长任务问题。

浏览器标注解决的是前端反馈问题。

插件共享和 Analytics 解决的是团队规模化问题。

所以，Codex 的正确用法已经不是：帮我写一段代码。

而是给出一个和成熟的开发代理匹配的任务布置。

会这样用的人，才算真正进入了 Codex 的新阶段。
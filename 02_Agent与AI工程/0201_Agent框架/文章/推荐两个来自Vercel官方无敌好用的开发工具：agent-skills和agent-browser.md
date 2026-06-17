---
title: 推荐两个来自Vercel官方无敌好用的开发工具：agent-skills和agent-browser
author: AI编程实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247484476&idx=1&sn=365aaf97c5868a5d6dbfa6945ee2d0fa&chksm=97fb21cb732576375fa5bc5428836199645dc4243b29def5a0fa0d79d5258a29d90a36093cb9&mpshare=1&scene=24&srcid=0119TQbs6Jp4igq4uGgAnOIa&sharer_shareinfo=6b7b69e65e9513f3025a56dd72951b2f&sharer_shareinfo_first=6b7b69e65e9513f3025a56dd72951b2f#rd
---

大家好，我是鲁工。

最近Skill很火，我在X上又发现了两个来自Vercel官方的开源工具，用下来感觉非常不错，特地出一期推文分享给大家。

这两个项目分别是agent-skills和agent-browser，都来自vercel-labs，也就是Vercel的官方项目仓库。

## agent-skills：AI编程助手的技能包管理器

先说第一个，agent-skills。

Guillermo Rauch（Vercel的CEO）在发布时用了一个很形象的比喻：这是一个npm for AI agents，也就是AI编程助手的包管理器。

一条命令，就可以让Claude Code获得Vercel团队积累了10年的React和Next.js优化经验。

安装非常简单：

```
npx add-skill vercel-labs/agent-skills
```

这个CLI会自动扫描你电脑上已安装的AI编程工具（Claude Code的.claude目录、Cursor的.cursor目录等），然后把技能安装到对应的位置。

目前agent-skills仓库里提供了三个核心技能：

**react-best-practices**

这个是重头戏。包含40多条规则，覆盖8个优化类别，按影响程度排序。

涵盖的内容包括：消除请求瀑布流、Bundle体积优化、服务端性能调优等。都是Vercel团队在实际项目中总结出来的经验。

举个例子，规则里会告诉AI：不要在组件渲染时发起请求造成瀑布流，而应该用并行数据获取；不要随意使用use client把整个页面变成客户端组件；图片要用next/image而不是原生img标签。

这些都是实际开发中容易踩的坑。有了这个技能，AI在帮你写React代码时，会自动考虑这些最佳实践，不用你反复提醒记得优化性能。

**web-design-guidelines**

100多条UI审查规则，覆盖可访问性、性能和用户体验。

比如语义化HTML、ARIA标签、焦点状态、表单验证、国际化等，都在检查范围内。

让AI帮你Review UI代码时，它会自动按这些标准来审。

**vercel-deploy-claimable**

这个技能可以直接从对话中部署应用到Vercel。

能自动识别40多种框架（通过读取package.json），部署完成后给你一个预览链接和一个认领链接，用于转移项目所有权。

技能安装后是自动激活的。直接在Claude Code中输入"Deploy my app"或者"Review this React component for performance issues"，对应的技能就会被触发。

仓库地址：

https://github.com/vercel-labs/agent-skills

目前已经有1.26万Star，可见大家对这类工具的期待。

## agent-browser：让AI真正能操作网页

第二个项目是agent-browser。

这是一个专门为AI Agent设计的浏览器自动化CLI工具（主要注意，agent-browser不是一款skill），也是上周刚发布。

它解决了一个核心问题：传统的Playwright/Chrome DevTools等MCP方案在向AI传递网页信息时，上下文太长、效率太低（嗯，转眼间Playwright/Chrome DevTools都是传统方案了）。

agent-browser的做法是精简数据结构，减少高达93%的无关上下文，只给AI提供关键的DOM信息和可操作元素。

安装也很简单：

```
npm install -g agent-browseragent-browser install  # 下载Chromium
```

底层用Rust写的，启动快、资源占用少。同时提供Node.js作为fallback，兼容性有保障。

核心工作流是这样的：

```
# 1. 打开页面agent-browser open example.com# 2. 获取可交互元素快照agent-browser snapshot -i# 3. 用引用标识操作元素agent-browser click @e2agent-browser fill @e3 "test@example.com"# 4. 截图保存agent-browser screenshot result.png# 5. 关闭浏览器agent-browser close
```

比如，获取我的AI面相大师xuanji.app的页面快照：

最有意思的是snapshot命令返回的元素引用机制。

执行snapshot -i后，你会得到类似这样的输出：

```
button "登录" [ref=e1]input[type=email] "邮箱" [ref=e2]input[type=password] "密码" [ref=e3]link "忘记密码" [ref=e4]
```

每个可交互元素会被标记为@e1、@e2、@e3这样的引用。后续操作直接用这个引用就行，不用再写复杂的CSS选择器。

想填写邮箱？直接`agent-browser fill @e2 "test@example.com"`。想点登录按钮？直接`agent-browser click @e1`。

这套设计对AI来说太友好了。AI只需要看快照结果，找到目标元素的引用编号，然后发出操作命令。整个过程非常直观，出错率也大大降低。

除了基础的点击、填充、截图，agent-browser还支持：

* 语义化定位：通过ARIA角色、可见文本、表单标签来定位元素
* 多会话管理：同时运行多个隔离的浏览器实例
* 网络控制：拦截和模拟HTTP请求
* 设备模拟：模拟移动端设备和配色方案
* 状态持久化：保存和加载浏览器状态，用于登录态保持

兼容性方面，Claude Code、Gemini、Cursor、GitHub Copilot、Codex、opencode这些主流AI编程工具都能用。只要能执行Bash命令，就能用agent-browser。

仓库地址：

https://github.com/vercel-labs/agent-browser

目前7.5k Star，Apache-2.0协议开源。

## 实际使用场景

说几个我觉得比较实用的场景。

**场景一：让AI帮你做E2E测试**

之前用AI做端到端测试挺麻烦的。你得让它先写Playwright脚本，然后调试各种选择器问题，一来二去耗费不少时间。

现在你可以直接告诉AI：

> 用agent-browser打开localhost:3000，登录测试账号，点击创建按钮，填写表单，验证提交成功。

AI会一步步执行命令，遇到问题也能根据snapshot结果自己调整。整个过程比写测试脚本高效多了。

**场景二：Web数据采集**

需要从某个网站收集一些公开信息？以前要么手动复制，要么让AI写爬虫脚本。

现在可以让AI直接用agent-browser操作：打开页面、获取元素、提取文本，整个流程在对话中就能完成。

**场景三：Next.js项目性能优化**

项目代码写完了，想让AI帮我们审一遍性能问题。

装上react-best-practices技能后，告诉Claude Code："帮我检查这个组件的性能问题"，它会自动按照40多条规则来审核，指出请求瀑布流、不必要的客户端渲染、Bundle体积问题等。

比你自己对照文档检查效率高不少。

## 写在最后

Vercel在AI开发工具领域的布局一直很积极。

除了这两个项目，他们最近还发布了AI SDK 6，引入了Agent抽象层。定义一次Agent的模型、指令和工具，就能在整个应用中复用。

这三个东西放在一起看，思路就很清晰了：

* AI SDK负责Agent的基础框架
* agent-skills负责给Agent充电，让它具备专业领域知识
* agent-browser负责给Agent长手，让它能真正操作浏览器

对咱们开发者来说，这意味着什么？

以前用AI写React代码，你得反复提醒它注意性能优化、注意可访问性。现在装上react-best-practices技能，这些规则就内置了。

以前想让AI帮你做端到端测试、爬取数据，要么用Playwright MCP（上下文太长），要么让AI写爬虫脚本（调试成本高）。现在用agent-browser，一行命令打开页面，一行命令获取元素，一行命令操作。

整个动作体验丝滑和轻盈很多。

强烈推荐Claude Code、Cursor、Antigravity等AI编程工具的用户试试，应该会有惊喜。

**参考资料**：

1. https://github.com/vercel-labs/agent-browser
2. https://github.com/vercel-labs/agent-skills

我是鲁工，八年AI算法老兵，AI全栈开发者，深耕AI编程赛道。欢迎关注，感兴趣的朋友也可以加我微信（louwill\_）交个朋友。

>/ 作者：鲁工
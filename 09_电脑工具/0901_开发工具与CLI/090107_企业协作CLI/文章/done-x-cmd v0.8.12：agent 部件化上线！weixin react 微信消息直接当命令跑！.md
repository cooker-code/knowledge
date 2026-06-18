---
title: x-cmd  v0.8.12：agent 部件化上线！weixin react 微信消息直接当命令跑！
author: oh my x
date:
url: https://mp.weixin.qq.com/s?__biz=Mzg5Mzk4MTc0Ng==&mid=2247494187&idx=1&sn=c66fc9783cfc416b67274121687df4aa&chksm=c1036ac5ca5936733ca2256913e187a0369e9898fb39cbbbecc658effa04d13a0394e5f78159&mpshare=1&scene=24&srcid=0405jNFEhywNCGdXFIhF8yog&sharer_shareinfo=5e3b029fb1d2f0ab340f3d29feb723d9&sharer_shareinfo_first=5e3b029fb1d2f0ab340f3d29feb723d9#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090107_企业协作CLI/090107_核心知识点/企业协作CLI权限与审计边界|企业协作 CLI 权限与审计边界]]


TLDR:

* agent 模块推出 request 单次请求与 job 长期任务子命令，支持 AI 对话澄清需求并自动迭代执行
* weixin 模块新增 react 命令，repl 模式可以把微信消息直接当命令跑，结果推送回去；
* docker 模块修复 refit 基础镜像用户已存在但 home 目录缺失时的构建镜像问题

## 🚀 x-cmd v0.8.12 更新详情

### agent 🤖

`agent` 模块新增 `request` 和 `job` 子命令 —— 这次的核心是把 agent 能力部件化。

我们把单次调用和任务迭代拆成了独立部件，下一步的目标，是让这些部件可以被灵活组合，搭出更适合自己的 agent 工作流程。

这次上线了两个能力：

* `x agent request`

  单次请求，适合简单任务。发送提示词直接拿结果，支持附带文件
* `x agent job`

  长期任务，交给 AI 慢慢迭代。你只需要丢个需求过去，AI 会先跟你对话把需求掰扯清楚，生成 TODO.md，然后自动一步步执行。中途改主意也能随时调整，任务多了还能一目了然列出来

示例：

```
# 单次请求，附带文件
x agent request -f main.py "Review this code"

# 创建一个长期任务，AI 会跟你对话澄清需求
x agent job init "Add unit tests and set up CI"

# AI 自动按 TODO 迭代执行
x agent job iter

# 改需求了，AI 会更新 TODO
x agent job adjust "Add integration tests too"

# 看看有哪些任务在进行中
x agent job ls
```

有想法或遇到问题，欢迎提 issue。

### weixin 💬

新增 `react` 命令 —— 把微信 ClawBot 变成 repl 机器人。

在微信里跑命令这个想法一直有，现在终于整出来了。开启 repl 模式后，收到什么就当命令跑，结果直接推回去。

后续我们会增加 Agent 运行能力。

示例：

```
# 启动交互式指令模式，将收到的消息作为命令执行并返回结果
x weixin react --runner repl
```

同时修复了发送文件、图片失败的问题 —— 微信调整了数据格式，解析逻辑没跟上。

### docker 🐳

修复 `refit` 问题 —— 基础镜像用户已存在但 home 目录缺失时，无法构建镜像。

感谢 @yyq19990828 在 github issue #394 给出的踩坑报告。

### ⬆️ 如何升级

现有用户可以通过以下命令快速切换至 Beta 版本进行体验：

```
x upgrade beta
```

#### 如果你没有安装 x-cmd, 只需要打开你的终端:

```
eval "$(curl https://get.x-cmd.com)"
```

x-cmd 是一个一站式的命令行工具集，其强大的功能可以为人类用户和AI共同使用。它还简化了很多工具的安装方法。
马上安装，让 x-cmd 协同 AI 成为你的最强助手，实现生产力翻倍！

### 🤝 开发者反馈

如果您在自定义配置或代理设置中遇到任何疑问，欢迎前往 GitHub Issues 提交反馈，共同完善 X-CMD 生态。

---

---

🎉 **欢迎****加入我们的用户交流群** 🎉

在这里，你可以：
📢 第一时间了解最新动态和活动信息
🤝 结识同行，共享实战经验
📚 探讨行业趋势，拓展人脉圈子

📌 **如何加入？**📷 扫描添加小助手，完成验证即可入群！

期待你的加入，一起交流成长！🚀

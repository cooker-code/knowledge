---
title: 真正好用的 MCP，Claude code 必装，开发效率飞起！【附实操案例】（三）
author: 阿飞AI实操日记
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775920&idx=1&sn=c9e721858aa4dd9e7883f084b9986dcf&chksm=f1d0a133605343ce2dba4ebbd5ed3fad1ff900cce53ed9cdc0216031fd101aa23f5f63c6c6a1&mpshare=1&scene=24&srcid=1115GFih1Z0dH69WAERW8Yog&sharer_shareinfo=0b1c5a6106923e0bb6fae74c961d7718&sharer_shareinfo_first=0b1c5a6106923e0bb6fae74c961d7718#rd
---

点击上方卡片关注我👆

设置星标 让我们一起学 AI！

接前两篇：

[真正好用的 MCP，Claude code 必装，开发效率飞起！【附实操案例】（一）](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775831&idx=1&sn=8f40355af2475e5a7ad581555c3685bb&scene=21#wechat_redirect)

[真正好用的 MCP，Claude code 必装，开发效率飞起！【附实操案例】（二）](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775863&idx=1&sn=1f8cb5a15574ac88247c8dddb20b1117&scene=21#wechat_redirect)

# 一、前端组件库：Shadcn UI MCP

**GitHub地址**：https://github.com/Jpisnice/shadcn-ui-mcp-server

* **功能**：让AI自动从美观实用的Shadcn UI组件库中查找并生成组件代码。

#### 配置(Claude Code):

从`Shadcn UI`文档复制MCP配置命令并安装

```
pnpm dlx shadcn@latest mcp init --client claude
```

#### 实践案例：

让 `claude code` 生成一个 `shadCN UI`的组件，比如为了提升用户体验，我们经常要在网页加载显示内容占位符

```
请你帮我生成项目的骨架图以提升用户体验，使用 shadcn UI 的 Skeleton 组件来实现
```

# 二、支付集成：Stripe MCP

* **功能**：为产品快速接入支付能力。

#### 配置(Claude Code):

1. 配置命令：

```
claude mcp add stripe --transport http https://mcp.stripe.com/
```

2. 启动Claude Code并授权。

#### 实践案例:

* 提示词：

```
请你为当前的 next.js 项目添加一个支付页面，无需复杂的后台逻辑，只实现基础的支付功能。商品的信息请你使用 stripe mcp 从 stripe获取。
```

* `Claude code` 调用 `stripe mcp` 获取商品信息和价格，编写前端代码

* 从 `stripe` 控制台获取 stripe 密钥配置到项目中

* 实现效果：页面展示商品，订阅功能测试也完全正常，在我们的支付记录可以看到我们的支付

# 三、安全扫描：Semgrep MCP

**GitHub地址**：https://github.com/semgrep/semgrep/tree/develop/cli/src/semgrep/mcp

* **功能**：集成静态安全分析工具，内置5000多条安全规则，让AI对代码进行安全漏洞扫描。

#### 配置(Claude Code):

配置命令：

```
claude mcp add semgrep --transport http https://mcp.semgrep.ai/mcp
```

#### 实践案例：

让 `claude code` 使用 `semgrep mcp` 对项目代码进行安全扫描。

```
请你对项目代码进行安全扫码，use semgrep mcp
```

`Claude code` 调用 `mcp` 工具完成扫描并给出报告结果以及注意的安全实践

---

# 💡推荐阅读

如果你也想使用 Claude，但是**不想支付高额的费用，不想承担封号风险**……

**推荐你试一下我们的AI CODE平台**

详细介绍及付费兑换，后台回复：**cc** 查看

或+v：**afly813** 咨询

目前我们的 AI CODE 平台**同时支持 claude code 和 codex** 这两大顶级大模型，想体验最强最前沿的 AI 编程，冲就完事了！！🚀

[伙伴们，以后写代码，codex和claude都可以爽yy啦！！！](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775523&idx=1&sn=967251557c7fe2e18238226bfe6be8f4&scene=21#wechat_redirect)

[让你的 Claude Code 效率飞起！你只差这个万能公式！！](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775703&idx=1&sn=3f360a73211db04f598359467d95242a&scene=21#wechat_redirect)

[这才是 AI 编程的最强组合，VSCode + Claude Code 让写代码快到飞起！](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775649&idx=2&sn=5ec861d44e776057a01b1542d8b61017&scene=21#wechat_redirect)

[【附提示词模板】10个 Claude code 高频提示词模板（可直接复制使用）!建议收藏！！](https://mp.weixin.qq.com/s?__biz=MzIyMTI1MTkyNQ==&mid=2649775721&idx=1&sn=43f7754dafdc2d6c792f6a29b47c3052&scene=21#wechat_redirect)

**喜欢的话❤，欢迎点赞、关注一波，后续会持续为大伙分享 工作流、 AI编程等实战干货，让我们一起学 AI！**
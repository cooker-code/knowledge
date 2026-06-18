---
title: 使用playwright-cli 实现 0 TOKEN 执行固定流程
author: 裕见AI
date:
url: https://mp.weixin.qq.com/s?__biz=MzYzNzEwMDIxMw==&mid=2247483831&idx=1&sn=25885ec221bb03325a0eadfce4a6743c&chksm=f1d2071b69a71b52842976e57c0526d325550565ac5d2b8370904e7cd3de33323fbde139d4a5&mpshare=1&scene=24&srcid=0416eLW2fiDAOCbHVfQtN0lQ&sharer_shareinfo=c67d8209b8f8dd99c19b16fea9a02fdf&sharer_shareinfo_first=c67d8209b8f8dd99c19b16fea9a02fdf#rd
---
> 已吸收至：[[09_电脑工具/0901_开发工具与CLI/090106_Playwright CLI/090106_核心知识点/PlaywrightCLI浏览器自动化边界|Playwright CLI 浏览器自动化边界]]


年初微软开源了一款可以节省大量token的浏览器自动化工具  playwright-cli 开源地址： https://github.com/microsoft/playwright-cli

根据官方的测试playwright-cli 对比playwright mcp 差不多可以节省4倍token。

# 必要软件

nodejs + chrome + 任意一款AI 编程 Agent（opencode，Claude code ，codex ，Antigravity）

这里使用opencode作为参考。

安装 playwright-clinpm install -g @playwright/cli@latest

使用 playwright-cli 打开浏览器操作playwright-cli open google.com --headed

-- headed 参数表示有头浏览器，playwright-cli 默认使用无头浏览器。

|  |  |  |  |  |
| --- | --- | --- | --- | --- |
|  | 参数 | 界面 | 后台运行 | 优点 |
| 有头浏览器 | --headed | 有 | 否 | 方便调试 |
| 无头浏览器 | 无 | 无 | 是 | 省内存/后台操作 |

这里打开baidu之后会返回一个snapshot ，这个文件会将访问到的网页保存到本地，大模型可以分析本地文件，由AI决定执行后续的指令，而MCP 会将整个网页内容放进大模型上下文，这也就是playwright-cli 比较省token的原因了。

--persistent 也是一个重要的参数，它会将登录后的token，cookie保存到磁盘，这里后续就不需要进行登录验证了。

安装 playwright-cli  skills 在项目文件夹执行命令playwright-cli install --skills

执行后打开opencode 询问你有哪些skills

这样我们的playwright-cli skills就安装完成了

playwright-cli 常用命令

```
palywright-cli show  # 打开所有浏览器会话(可查看页面)playwright-cli list  # 查看所有会话信息playwright-cli close-all # 正常关闭所有浏览器playwright-cli kill-all # 强制关闭所有浏览器
```

这里让AI 打开 deepseek 问今天天气怎么样

这里没有登录，我们登录后让AI 继续操作，我们在浏览器登录后，下次使用--persistent 参数就不需要再登录了

# 自动化获取页面内容并封装成skill 更省Token

使用playwright-cli  获取B站视频简介，评论区前10 评论，并将内容保存到CSV 文件里面

初次完成任务消耗的token会比较多，之后我们将这个过程精练成skill

精炼后使用skill 获取视频里面的内容，如果精练后不可用可以让AI 进行优化skill。

最后获取到的评论，

由于用的模型比较小，这里问了三次，上下文仅用了37%，没有精练之前一次问答使用了40%的占用量

将操作保存为脚本实现0 TOKEN 执行操作

将整过程保存为一键脚本，为以后使用

之后我们试下这个脚本即可实现全脚本操作刚刚的步骤啦

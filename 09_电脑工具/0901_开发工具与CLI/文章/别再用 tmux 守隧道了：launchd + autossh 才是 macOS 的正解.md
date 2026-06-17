---
title: 别再用 tmux 守隧道了：launchd + autossh 才是 macOS 的正解
author: AI灵感闪现
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490507&idx=1&sn=4dea8dd08d840b8bfda7f8b92de5c240&chksm=a7453082a93a4be30c69f75f789697da0310142988212414f4e4479e6d73a297d9b048eb5505&mpshare=1&scene=24&srcid=04025TolBW3fp4mSM7mrIkLh&sharer_shareinfo=3ef0f4d82edb729b5af81a34be382a05&sharer_shareinfo_first=3ef0f4d82edb729b5af81a34be382a05#rd
---

又断了。

每天早上打开电脑，第一件事就是跑去检查那条连服务器的 SSH 隧道还活着没有。十次里面八次是断的。`curl localhost:18800` 超时，服务挂了，一天的工作开了个烂头。

我烦透了这个循环，决定彻底查清楚原因。

## 到底是谁死了

第一反应是 SSH 连接断了。但实际排查之后发现，问题根本不在 SSH。

我先验证了 SSH 本身没问题：

```
ssh -o BatchMode=yes admin@192.168.1.85 "echo SSH_OK"
```

返回了 `SSH_OK`。密钥认证正常，服务器也活着。

然后我去找 tmux：

```
tmux attach -t tunnel
```

报错：

```
no server running on /private/tmp/tmux-501/default
```

不是 session 消失了，是 **tmux 服务进程本身没了**。整个 tmux server 都不存在了。

> **我是 AI灵感闪现，使用 OpenClaw 小龙虾 让 AI 自主管理工作和生活上的问题；使用 Claude Code + BMAD AI 驱动敏捷开发框架，让 AI 自主开发和交付软件来表达想法和灵感。****是 MoneyMind 省钱思维 App 和 HeartPetBond 心宠纽带 App 开发者。正在实践和分享让 AI 自主解决健康、生活、投资和等方面的问题。****我尽可能让 AI 自己完成从目标到交付以及演进的闭环，以最少的人为交互与监督，让 AI 自己跑流程。我只给 AI 想法或目标，全程不陪跑，让 AI 自主运行类似 Tesla FSD 自动驾驶。**

## macOS 在背后干了什么

这里有个很多人不知道的细节：macOS 在休眠或重启时，会清理 `/private/tmp/` 下的内容。tmux 的 socket 就放在那里。socket 没了，tmux server 自然起不来，session 自然也没了，里面跑着的 SSH 隧道当然一起凉凉。

更根本的问题是：tmux 是一个普通的用户空间进程，它没有任何机制让 macOS 在休眠后帮你重启它。你 `tmux new-session` 起来的东西，macOS 压根不知道它的存在，也不在乎它死没死。

所以这个问题不是"怎么让 tmux 更稳定"，而是"tmux 本来就不该干这个活"。

守护进程这件事，macOS 有专门的工具：**launchd**。

## 换个思路：launchd + autossh

launchd 是 macOS 的服务管理器，系统启动时最先跑起来的进程之一，PID 1 就是它。你告诉它要运行什么、什么时候运行、崩溃了怎么办，它帮你管着。macOS 休眠、重启、重新登录，launchd 都知道该干什么。

autossh 则解决了另一个问题：SSH 连接偶尔还是会断（网络抖动、NAT 超时）。autossh 会监控连接状态，断了自动重连。两者配合，隧道基本上就是永活的。

## 动手：十分钟搞定

### 第一步：装 autossh

```
brew install autossh
```

装完确认一下路径：

```
which autossh  
# /opt/homebrew/bin/autossh
```

记住这个路径，等下写 plist 会用到。Apple Silicon Mac 的 Homebrew 装在 `/opt/homebrew/`，Intel Mac 在 `/usr/local/`，别搞错了。

### 第二步：写 launchd plist

在 `~/Library/LaunchAgents/` 下创建一个 plist 文件。这个目录下的 plist 是用户级别的 agent，登录后自动加载，不需要 root 权限。

文件名我用 `com.openclaw.tunnel-s8.plist`，你可以换成自己喜欢的命名。

```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"  
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">  
<plist version="1.0">  
<dict>  
    <key>Label</key>  
    <string>com.openclaw.tunnel-s8</string>  
  
    <key>ProgramArguments</key>  
    <array>  
        <string>/opt/homebrew/bin/autossh</string>  
        <string>-M</string>  
        <string>0</string>  
        <string>-N</string>  
        <string>-o</string>  
        <string>ServerAliveInterval=30</string>  
        <string>-o</string>  
        <string>ServerAliveCountMax=3</string>  
        <string>-o</string>  
        <string>ExitOnForwardFailure=yes</string>  
        <string>-L</string>  
        <string>18800:127.0.0.1:18789</string>  
        <string>admin@192.168.1.85</string>  
    </array>  
  
    <key>RunAtLoad</key>  
    <true/>  
  
    <key>KeepAlive</key>  
    <true/>  
  
    <key>StandardOutPath</key>  
    <string>/tmp/openclaw-tunnel-s8.log</string>  
  
    <key>StandardErrorPath</key>  
    <string>/tmp/openclaw-tunnel-s8.log</string>  
  
    <key>EnvironmentVariables</key>  
    <dict>  
        <key>AUTOSSH_GATETIME</key>  
        <string>0</string>  
    </dict>  
</dict>  
</plist>
```

几个参数说明：

`-M 0` 禁用 autossh 自带的监控端口，改用 SSH 自己的 keepalive 机制，更简单可靠。

`ServerAliveInterval=30` 和 `ServerAliveCountMax=3` 是 SSH 的心跳配置，30 秒发一次，连续 3 次没响应就认为连接断了，触发 autossh 重连。

`ExitOnForwardFailure=yes` 端口转发失败时直接退出进程，让 launchd 来重启，比僵着更好。

`KeepAlive=true` 告诉 launchd：这个进程退出了就给我重启。

`AUTOSSH_GATETIME=0` 让 autossh 在首次连接失败时也尝试重连，而不是直接放弃退出。

### 第三步：检查 plist 格式

```
plutil -lint ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist
```

输出 `OK` 才继续。格式有问题 launchd 会静默失败，比较坑。

### 第四步：加载并启动

```
launchctl load ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist
```

或者用新版语法：

```
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist
```

### 第五步：验证

先看端口有没有监听：

```
lsof -i :18800
```

应该能看到 ssh 进程占着 18800 端口。

再测试实际连通：

```
curl -I http://localhost:18800
```

返回 HTTP 200 就对了。

查日志：

```
tail -f /tmp/openclaw-tunnel-s8.log
```

确认没有报错。

## 常用管理命令

停止隧道：

```
launchctl unload ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist
```

重启隧道（改了 plist 之后用这个）：

```
launchctl unload ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist  
launchctl load ~/Library/LaunchAgents/com.openclaw.tunnel-s8.plist
```

查看 launchd 认为它的状态如何：

```
launchctl list | grep openclaw
```

第一列是 PID（进程跑着是数字，没跑是 `-`），第二列是上次退出码（0 是正常）。

## 一个坑：autossh 要用绝对路径

plist 里的 `ProgramArguments` 必须写 autossh 的完整路径，不能写 `autossh`。launchd 启动时环境变量不完整，PATH 里不包含 Homebrew 的路径，写短命令会直接找不到。

同理，如果你的 SSH key 不在默认位置（`~/.ssh/id_rsa` 或 `~/.ssh/id_ed25519`），也要在 plist 里加 `-i /absolute/path/to/key`，不能用 `~` 开头，launchd 不展开 `~`。

## 搞定之后

从昨晚配好到现在，隧道跑了将近 18 小时，中间 Mac 睡了两次，全程稳定。日志里能看到它在定期发 keepalive，偶尔有一次网络抖动，autossh 自动重连了，30 秒内恢复。

再也不用早上第一件事去检查隧道了。

如果你有多条隧道，每条建一个 plist 文件就行，Label 和文件名改成不同的。管理起来比在一个 tmux window 里开一堆 SSH 命令清晰多了，哪条挂了 launchd 日志里也能单独看到。

tmux 还是好工具，但它的用途是交互式终端会话管理，不是守护进程。让对的工具干对的事。

[OpenClaw 小龙虾（点击跳转合集）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4421270045515841537#wechat_redirect)

[s?__biz=MjM5Mzk1NzA1NA==&mid=2247490360&idx=1&sn=8da4f1607ebfe90943a04624d32e9454&scene=21](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490360&idx=1&sn=8da4f1607ebfe90943a04624d32e9454&scene=21#wechat_redirect)

[一个机器人，多个 Agent：OpenClaw Discord 频道级路由配置](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490330&idx=1&sn=a9c02cd0f39923a1c83cc03543884b5b&scene=21#wechat_redirect)

[OpenClaw + 飞书 + Scrum：用 AI Agent 团队跑完整个敏捷研发闭环](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490213&idx=1&sn=a7c53052d27b7f8acc50be4ba159d91c&scene=21#wechat_redirect)

[CTO 视角：OpenClaw 企业级多项目 AI Agent 架构怎么搭](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490192&idx=1&sn=270a654532997b41f7376dba2603bba0&scene=21#wechat_redirect)

[给 OpenClaw 接飞书机器人，三个坑让我查了一小时](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490187&idx=1&sn=33e91b2008944de070b341e0a9f4350c&scene=21#wechat_redirect)

[QQ 小龙虾🦞：sliverp/qqbot 和 tencent-connect/openclaw-qqbot 到底选哪个](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490120&idx=1&sn=b33c92b1beb6b4c340a330dda5c38794&scene=21#wechat_redirect)

[给 QQ 装个小龙虾🦞：官方openclaw-qqbot 实测，2条命令搞定，群聊踩坑记录](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490116&idx=1&sn=b17b29cd9909d83dac6efef222832c29&scene=21#wechat_redirect)

[OpenClaw v2026.3.11: WebSocket 劫持已修复, Ollama 正式集成, 记忆搜索支持图片和音频](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490141&idx=1&sn=c50c8843dd4ef1c85b77802d264ccd18&scene=21#wechat_redirect)

[局域网两台电脑跑 OpenClaw，'Allow device to connect?' 弹个没完？四条命令治好它](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490140&idx=1&sn=730f77926c9702b4c4ff5220a2810be4&scene=21#wechat_redirect)

[Windows 11 原生装 OpenClaw：PowerShell 一行搞定 QQ 机器人](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490062&idx=1&sn=c9a06cd3628573b4777be0f7b57d1240&scene=21#wechat_redirect)

[macOS 原生装 OpenClaw：一条命令接上 QQ 机器人](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490053&idx=1&sn=70addb46c600fb0aeb161b39e13e620a&scene=21#wechat_redirect)

[用 Docker 装 OpenClaw：一条命令，三个坑，一个能用的 AI 智能体](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490027&idx=1&sn=c8e15d5f546148fa773935c0d6c94b22&scene=21#wechat_redirect)

[OpenClaw Telegram Topics: 一个群组运行多条并行任务流](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489773&idx=1&sn=ddf3ab39335ef6961df948de6d781078&scene=21#wechat_redirect)

[Claude Agent SDK 系列（点击跳转合集）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4074951080126136323#wechat_redirect)

[Claude Agent SDK 构建 AI Agent 实践：如何实现与上传文件的对话](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489575&idx=1&sn=31bd2ed68ba99d12c6ae16e3eea08f81&scene=21#wechat_redirect)

[Claude Agent SDK 构建 AI Agent 实践：服务端向 Claude Agent SDK 注入环境变量的实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489567&idx=1&sn=1b589f444911dcfb40e21b28b9ea2a14&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序：AI Agent 项目实践复盘 2026-01-21](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489493&idx=1&sn=59f8dd794de6447e713298ae1f305330&scene=21#wechat_redirect)

[Claude Agent SDK + 微信小程序为个人打造 AI 分身：vs-ai-agents 项目技术实践复盘](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489492&idx=1&sn=dda6a2df2490d4c2ab6eff9fd4c9aeba&scene=21#wechat_redirect)

[BMAD AI 驱动敏捷开发系](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)[列（点击跳转合集](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)[）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5Mzk1NzA1NA==&action=getalbum&album_id=4357253184793296905#wechat_redirect)

[BMAD 6.2.0：推荐使用 bmad-product-brief-preview 基于 Prompt 的多 Agent 编排](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490361&idx=1&sn=33bde27731c4e26b9b9d4965a057f31a&scene=21#wechat_redirect)

[如何用 BMAD Quick Dev 在 10 分钟内把客户的一句话需求变成完整的可行性评估](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490329&idx=1&sn=616acdd3988a18cc3b3d9b84bec4b686&scene=21#wechat_redirect)

[Claude Code + BMAD Quick Dev + YOLO：AI 自主修复缺陷的完整闭环实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490287&idx=1&sn=4824845eaba5daa36fe496caca0de5ec&scene=21#wechat_redirect)

[BMad v6.1.0 用统一的Skill技能架构替代了旧的工作流引擎](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490227&idx=1&sn=55ddd573b13fa93fb79abf8bf447356a&scene=21#wechat_redirect)

[当 BMAD 开发工作流遇上 PPT 周报生成：BMAD Quick Dev 的边界拓展](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490214&idx=1&sn=f13267797f593ed2e27739afb39a0271&scene=21#wechat_redirect)

[Claude Code + BMAD + YOLO 模式：一个 Session 搞定全栈功能开发](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247490152&idx=1&sn=62d93211cdb201ad7f4724ca2d10adb4&scene=21#wechat_redirect)

[BMad v6.0.4 + GDS v0.1.10：边缘用例猎手、多智能体测试和引擎知识库](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489806&idx=1&sn=0d00d1ff74b03e1c64b06d89463a3412&scene=21#wechat_redirect)

[BMAD v6.0.4：从 Beta 到正式版，两分钟搞定](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489793&idx=1&sn=2a3eb5bf21d03ea036161574985c3e5f&scene=21#wechat_redirect)

[BMAD v6.0.0-beta-8 安装实战：从零开始搭建你的 AI 开发团队](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489769&idx=1&sn=58e722d9efdc488203fd9c1efae87d62&scene=21#wechat_redirect)

[BMAD + Ralph 执行循环：Claude Code 的统一 AI 开发框架](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489763&idx=1&sn=3230665edbfcc6e4855c6f9f8e2687c7&scene=21#wechat_redirect)

[BMAD 最佳实践:AI 驱动的敏捷开发指南](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489751&idx=1&sn=8909904d4f14ba5ada08588952aeb2bc&scene=21#wechat_redirect)

[BMAD 突破性 AI 驱动敏捷开发框架：v6.0.0-alpha.23 升级体验：全新安装之旅](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489133&idx=1&sn=4973d5907ef08bb9ebd67a6eeda8f203&scene=21#wechat_redirect)

[BMAD Method 入门指南：用 Quick Dev 工作流更快、更稳地交付](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489101&idx=1&sn=f78aaea74ba3db553fbca71e83abe868&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

[BMAD V6 安装配置完全指南：项目目录安装最佳实践](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486789&idx=1&sn=7bbfe9ef92964ef9be4c722b8d357991&scene=21#wechat_redirect)

[BMAD v6 安装更新：模块化 + AgentVibes “会说话”的开发体验](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486758&idx=1&sn=5d88d95112b626015377effd19227384&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

[AI 时代的"文档屎山"？BMAD、Spec-Kit、OpenSpec 等面向文档AI编程的利弊](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486451&idx=1&sn=eeba9d266868b49759d855277e323562&scene=21#wechat_redirect)

[在 Codex 里像 Claude Code 一样用 BMAD：把多角色 AI 团队装进你的仓库](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486257&idx=1&sn=436d00899f6adcd45c962b47bdfad04e&scene=21#wechat_redirect)

[BMAD 突破性 AI 驱动敏捷开发框架：深度解析 26 个代理、68 个工作流和 655 个文件](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489141&idx=1&sn=1aa0e66547dcf30940b30b7132308271&scene=21#wechat_redirect)

[AI 自主开发 App 成功上架：历时 14 天审核，MoneyMind 省钱思维 App 今天发布了](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488720&idx=1&sn=e5b4a2165194be07f642e5cadf10dc28&scene=21#wechat_redirect)

[MoneyMind 省钱思维 App 审核又被拒：粗心提交错误版本的惨痛教训](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487799&idx=1&sn=0e00268ba1aea481407848d1ea0e494b&scene=21#wechat_redirect)

[被苹果审核拒绝不要怕：这次用 Google Antigravity AI 快速修复 App Store 审核问题](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487193&idx=1&sn=18e5dee00ad05d815f5106d8ac7ae0c4&scene=21#wechat_redirect)

[Claude Code 自主开发 MoneyMind（省钱思维）iOS 应用送审 App Store](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487054&idx=1&sn=3d609b27f0c79c67b752b3645f6addc0&scene=21#wechat_redirect)

[用 Claude Code + BMAD AI 驱动敏捷，把一个想法变成 省钱思维 (MoneyMind) App](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247486723&idx=1&sn=45dddd81a5d856ced875b4769cc54d09&scene=21#wechat_redirect)

[全网首发？第一款 GLM 4.7 + Claude Code AI 自主开发的心宠纽带 App 首次通过 App Store 审核并上架发布](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247489028&idx=1&sn=ade948ba611d1df9ae3c15cd9f692797&scene=21#wechat_redirect)

[智谱 GLM 4.7 模型 AI 自主开发 HeartBetBond 心宠纽带 App，从想法到提交 App Store 仅用 12 天](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247488932&idx=1&sn=8ace2b6e8d016d0628ca54c208e0e889&scene=21#wechat_redirect)

[实战测评：用 Claude Code + BMAD + GLM-4.7 打造 HeartPetBond App (心宠纽带)](https://mp.weixin.qq.com/s?__biz=MjM5Mzk1NzA1NA==&mid=2247487840&idx=1&sn=53acb782e5b9eb1df9744f31745e60ab&scene=21#wechat_redirect)

## 加入 AI灵感闪现 微信群

长按下图二维码进入 AI灵感闪现 微信群

长按下图二维码添加微信好友 VibeSparking 加群

## 关注 AI灵感闪现 微信公众号
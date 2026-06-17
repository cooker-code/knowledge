---
title: 用命令行查微信聊天记录，这个 Rust 工具做到了
author: 半步宇宙
date: 半步宇宙半步宇宙
url: https://mp.weixin.qq.com/s?__biz=MzkzMjY1NTA5Mw==&mid=2247485361&idx=1&sn=56cfeda271caa19954389d6a42ef0718&chksm=c378b9326033792e1070132f6073efe7ffd30e3f60b3d974f5224c81131c993f46ef5169f544&mpshare=1&scene=24&srcid=0530QcDIVQdQ40JlRGeyJGxY&sharer_shareinfo=3f36dc5bc39939a6e55775fe3dadb477&sharer_shareinfo_first=3f36dc5bc39939a6e55775fe3dadb477#rd
---

微信的本地数据一直是个黑箱。聊天记录存在哪、怎么加密的、能不能导出，这些问题官方从来不回答。直到有人用 Rust 写了个工具，直接从微信进程内存里把密钥捞了出来。

wx-cli 做的事情，一句话概括，在命令行里查询你自己的微信数据。聊天记录、联系人、群成员、朋友圈、收藏，全都能搜。关键是数据不出本机，实时解密，不需要提前导出。

技术原理不复杂但很巧妙。微信 4.x 用 SQLCipher 4 加密本地数据库，密钥缓存在进程内存里。wx-cli 通过 macOS 的 Mach VM API 或 Linux 的 /proc/pid/mem 扫描内存，匹配密钥格式，然后按需解密。Windows 也有支持，走的是不同的内存读取方式。

架构设计上有意思的是 daemon 模式。第一次解密后，数据库和 mtime 会持久化到 ~/.wx-cli/cache/。下次查询时如果 mtime 没变，直接复用缓存，不用重新解密。这就是为什么查询响应是毫秒级的。

命令设计得很 Unix 风格。查聊天记录就是 wx history "张三"，搜全库就是 wx search "关键词"，导出 markdown 就是 wx export "张三" --format markdown。默认输出 YAML，省 token 也方便人读，加 --json 可以接 jq 管道。

朋友圈功能是亮点。wx sns-feed 能拉时间线，wx sns-search 能搜朋友圈正文，wx sns-notifications 能看点赞评论通知。这些数据微信客户端本身都不提供搜索功能，wx-cli 直接给补上了。

最让我意外的是 AI Agent 集成。一行 npx skills add jackwener/wx-cli 就能让 Claude Code、Cursor 这些 AI 工具直接读你的微信数据。想象一下，让 AI 帮你总结上周的工作群讨论，或者找出所有包含「报销」的聊天记录。默认 YAML 输出就是为了省 AI 的 token。

安装零依赖。npm install -g @jackwener/wx-cli 走全平台，或者直接下载预编译二进制。macOS 需要先对微信做一次 ad-hoc 签名，Linux 和 Windows 以管理员运行 wx init 就行。

隐私问题上，工具声明只用于学习研究、解密自己的数据。但工具本身不做限制，理论上谁拿到你的电脑都能跑。密码安全是个永恒的话题。

你有多久没翻过自己的微信聊天记录了？有些信息可能比你以为的更有价值。

项目地址:https://github.com/jackwener/wx-cli（1.2k ⭐）
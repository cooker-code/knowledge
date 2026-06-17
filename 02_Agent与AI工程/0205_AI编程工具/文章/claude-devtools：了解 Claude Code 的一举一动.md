---
title: claude-devtools：了解 Claude Code 的一举一动
author: AgenticCoding 实验室
date: 
url: https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484992&idx=1&sn=7b89a1a0b7fc874d964325cb2fb8fe58&chksm=9a16b595b9b9a101ecc7feb3b02f01f6d6389cd1e080957ca1aec184594264815048ce6a1475&mpshare=1&scene=24&srcid=0408aPyZAal1AeKYc2u21C3b&sharer_shareinfo=30af7ea98f29f1bd91f84464ca3a9efa&sharer_shareinfo_first=30af7ea98f29f1bd91f84464ca3a9efa#rd
---

# claude-devtools：了解 Claude Code 的一举一动

Claude Code 新版本中不再命令行中展示具体信息，只输出一个模糊的 `Read 3 files.``Searched for 1 pattern.``Edited 2 files`，虽然可以精简界面，但却会让用户迷惑，Claude Code 到底在做什么？

于是claude-devtools[1]出现，这是一款桌面应用，专门解决 Claude Code 执行黑箱的问题。有了这个工具，你就可以清晰的了解 Claude Code 每次执行任务时到底读了哪些文件、调用了哪些工具、执行了哪些 SubAgent以及上下文为什么突然爆炸💥。

---

实现上也很直观，他没有 Wrapper Claude Code CLI，而是直接读取 `~/.claude/` 目录下的历史记录，通过分析这些会话将详细信息展示在 GUI 上，没有任何黑科技。

---

最后说一下，Claude Code 历史记录这点做的很好，将所有的信息都存储到了本地文件中（历史记录都是清晰的 jsonl 文件），对于查询信息和问题排查都很友好，如果愿意，你也可以基于此做一些自己的小工具，我们公众号中之前文章[Claude HUD: 全面掌握你的 Claude Code 运行状态](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484797&idx=1&sn=4b1db4564aeeb0d4f1276b5df0fac01c&scene=21#wechat_redirect) 其中部分信息也是基于本地历史信息获取的。

---

**更多 Claude Code 相关文章：**

[31 个实用的 Claude Code 使用技巧](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484780&idx=1&sn=400d72b55b4bc52bd82c8981cb8abbbf&scene=21#wechat_redirect)

[Claude HUD: 全面掌握你的 Claude Code 运行状态](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484797&idx=1&sn=4b1db4564aeeb0d4f1276b5df0fac01c&scene=21#wechat_redirect)

[使用 CC History 查看 Claude Code System Prompt 变更](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484940&idx=1&sn=8be376ac057a5be593ce5e7bc4db149b&scene=21#wechat_redirect)

[Superpowers Plugin：让你的 AI 编码智能体拥有超能力](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484887&idx=1&sn=1acad2c235623214faf087ef219db169&scene=21#wechat_redirect)

[使用 Claude Code insights 命令了解和优化你的使用情况](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484880&idx=1&sn=26df4b2cd58fca0397b99f92f358c694&scene=21#wechat_redirect)

[使用 Cladue Code Playground Skill 创建可视化的交互体验](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484832&idx=1&sn=2b40dc9a3fa23162d37aaffb17a2ff7b&scene=21#wechat_redirect)

[Claude Code 之父是如何使用 Claude Code 的](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484777&idx=1&sn=fc072f3e3e3560b34bc5557fdf18aeae&scene=21#wechat_redirect)

[Claude Code System Prompt 解析](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484707&idx=1&sn=41aa8064e0f373cb7b39089c34da395b&scene=21#wechat_redirect)

[如何编写一份优秀的 CLAUDE.md](https://mp.weixin.qq.com/s?__biz=MzAxMDA0MTE5Mw==&mid=2247484603&idx=1&sn=d017c9654ba69850a0bbe022e126531e&scene=21#wechat_redirect)

#### 引用链接

`[1]` claude-devtools: *https://github.com/matt1398/claude-devtools*
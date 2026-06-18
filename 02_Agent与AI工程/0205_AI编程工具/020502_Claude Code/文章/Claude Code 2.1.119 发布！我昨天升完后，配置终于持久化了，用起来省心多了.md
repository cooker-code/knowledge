---
title: Claude Code 2.1.119 发布！我昨天升完后，配置终于持久化了，用起来省心多了
author: 晓明兄
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIzMTE0MTQ5MQ==&mid=2247485501&idx=1&sn=ed508c386d997efea0bcf26fddbd107b&chksm=e99104084ee3fadd5796b7352b7d9ddab09e23728769396254e3c04d9586d8fcd80826ea3988&mpshare=1&scene=24&srcid=0424qVuVwV37x6j4iueOG8nE&sharer_shareinfo=c38e0ca3a3c850922e808060431b96b7&sharer_shareinfo_first=c38e0ca3a3c850922e808060431b96b7#rd
---

兄弟们，我昨天还在用 2.1.118，今天刷到 Claude Code 又悄悄推了 2.1.119，第一时间就敲了 `claude --update`。

升完重启后，我马上跑了几个长会话和多项目切换的任务，最直观的感受就是**配置管理和上下文稳定性明显提升**。以前老是折腾的设置丢失、滚动跳动、粘贴乱码这些小问题，这次被针对性干掉不少，用起来安心又顺手。

1. 1. **/config 设置终于持久化 + 优先级更聪明（最实用的一点）**  
   以前改主题、编辑模式、verbose 等配置，重启终端后经常莫名其妙丢掉，或者项目设置被本地覆盖。现在所有 /config 设置都会自动保存到 `~/.claude/settings.json`，同时严格遵守 project / local / policy 的优先级覆盖规则。

   我昨天试着在不同项目里切换，强制设置团队主题和 editor mode，再也不用每次重启后手动再配一遍。尤其是多项目协作或者经常换机器的朋友，这点升级直接把配置管理的麻烦砍掉大半，用着终于踏实了。
2. 2. **PR 链接和 footer 自定义更灵活**  
   新增了 `prUrlTemplate` 设置，可以把底部 PR badge 指向自己公司的 code-review 系统，而不是默认跳 GitHub。这对在企业内网或者用 GitLab/Bitbucket 的团队来说特别友好。我简单配了一下，footer 直接指向内部评审链接，工作流顺畅多了。
3. 3. **启动界面和滚动体验优化**  
   新增 `CLAUDE_CODE_HIDE_CWD` 环境变量，可以隐藏启动 logo 里的当前工作目录，避免信息泄露或者界面太乱。同时修复了 streaming 响应时 queued commands 闪烁、scrollback 莫名跳到顶部或跳回底部等问题。

   我升完后故意跑了一个长任务，滚动再也没以前那种突然乱跳的情况，眼睛舒服多了，尤其适合夜里高强度编码。

其他小改进也让我用着解气  
• --from-pr 现在支持 GitLab merge-request、Bitbucket pull-request 和 GitHub Enterprise URL，跨平台拉代码更方便。  
• --print 模式终于和交互模式一样尊重 agent 的 tools / disallowedTools 设置，输出行为一致了。  
• 修复了鼠标选中复制在窗口外释放不生效、列表溢出时的鬼影字符、SDK resume 时 session history 丢失等问题。  
• slash command 在消息处理中提交也不再被当成普通文本发送。

我的升级操作建议：

1. 1. 终端直接输入 `claude --update`
2. 2. 重启 CLI，用 `/version` 确认已经是 2.1.119
3. 3. 试试改几个 /config 设置（比如 theme 或 editor mode），重启看是否持久化
4. 4. 跑一个多项目切换或长会话的任务，感受配置稳定和滚动丝滑的区别
5. 5. 如果你经常多项目工作、注重配置一致性，或者被滚动跳动烦过，这波升级特别值得

我个人感觉，这次 2.1.119 把“配置持久化”“优先级可控”“界面稳定”这些重度用户日常最烦的点，处理得更扎实了。虽然没有新模型或大功能，但用完后的省心感是实打实的——工具越来越像“透明助手”，而不是时不时需要我去救火。

点赞、转发、收藏这篇，别让你的 Claude Code 还停留在旧版本的配置烦恼里！🚀
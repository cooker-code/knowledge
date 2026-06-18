---
title: 这样用 CCPM 管理Claude Code编程项目，爽到飞起！
author: 小石的AI智能体工坊
date: 
url: https://mp.weixin.qq.com/s?__biz=MzE5ODcxNzQxOA==&mid=2247484122&idx=1&sn=9211bf76442b64d009ac1a58dc960949&chksm=97ce5d448887ca2103c421d470ec9a960f36ec3c57c23c330ec0511539c87e7d16f8904fe557&mpshare=1&scene=24&srcid=1115G5Gq3QP5A5o3pAnmuaGq&sharer_shareinfo=da4d8f8733562a63d414f8e85d603a4f&sharer_shareinfo_first=da4d8f8733562a63d414f8e85d603a4f#rd
---

**你好，我是小石学长，00后AI创业**

**在AI时代，希望帮助更多人有专属AI智能体**

最近用了ccpm来对claude code开发的项目进行管理，爽到飞起，我前期跟他确认好了产品方案，他就自己咔咔干活，下面我这样一句提示词，直接解放双手，AI一直干到触发速率限制。

接下来我给大家分享一下关于使用ccpm去做claude code AI编程项目实战。

## 一、CCPM 是什么／核心原理

CCPM = Claude Code Project Management，一个以 Claude Code + GitHub Issues + Git worktrees 为基础的轻量项目管理系统。

核心目标是让 PRD（产品需求文档）→ Epic（大任务）→ Issues（子任务）→ 代码 路径可追踪，同时支持多个 AI agent /人并行工作。

项目的「源数据/事实源」是 GitHub Issues。状态、任务分解、评论都在那里可以查。

## 二、前置条件

要顺利安装 +用 CCPM，需要具备这些：

1. 1. Node.js 环境（通常最新版/兼容版本）。
2. 2. Claude Code 本体（命令行 / VS Code 插件）。由 npm install -g @anthropic-ai/claude-code 安装。
3. 3. Git + GitHub 帐号，以及仓库权限。因为要同步 Issue、给项目建多个 issue。
4. 4. GitHub CLI (gh) 辅助工具，以及可选的 gh-sub-issue 扩展来支持子 issue 模式。

## 三、安装步骤与常用命令

ccpm 安装过程中如果出问题可以去问ChatGPT或者直接在Claude Code窗口提问。

通过前面的安装，我们能成功使用上ccpm，看到右边有这样一个ccpm，那就说明安装成功了。

接下来开始设计需求，让他写产品文档。

产品项目文档开始建立

然后基本上就是一直按回车键或者是复制控制台的命令，就是开始安装门槛较高，后续基本上不需要怎么操作。

## 五、常见问题 &注意事项（避踩坑）

不过在使用过程中大家也可能遇到像我这次的一些问题，大家在项目中如果遇到问题的话可以参考下。

如果斜杠命令如 /pm:init 出现 Unknown slash command，通常是因为你不在正确目录（项目根含 .claude/commands/...），或没有装 Claude Code。确保路径和命令结构正确。

文档里旧的路径（如 install/ccpm.sh 或根目录下的脚本）有可能已经失效，官网/README 最近有用户反馈路径不一致的问题。最好看最新版仓库 /COMMANDS.md 或者 README.md。

VS Code 的 terminal vs 系统 terminal 的区别要搞清楚：斜杠命令是在 Claude Code 的 terminal /命令面板执行，不是在普通终端里直接执行。

如果 GitHub CLI (gh) 或 gh-sub-issue 未装好或没授权，同步到 GitHub Issue 时会失败或出错。

保持 .claude 目录不要嵌套套娃，否则命令路径会错（例如不要 .claude/.claude/...）。

用了ccpm在项目开发的时候token消耗很快，因为你给了skip权限之后，他就能快速开发，涉及多次发送prompt指令，如果你不想这么快，不要用 claude--dangerously-skip-permissions指令。

## 六、结语

最近很多AI公司都在疯狂发力AI编程产品，前有claude code，后有OpenAI的codex，国内腾讯阿里都在布局冲刺，毕竟这也确实算是AI目前能够有效落地的一个方向。

如果大家有时间的话，建议真的尝试开始用AI编程，有能力的话，还是先从Claude Code和codex开始，有些东西，你试了，发现还真不一样。

如果大家账号确实有问题的话，我们有办法，扫码，发送 "AI工具" 即可。

**觉得有收获可以一键三连，转发给需要的小伙伴~**

**\*\*\*\*相关阅读\*\*\*\***

[第一批不找工作的年轻人，靠AI半年赚几十万](http://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247529966&idx=1&sn=8526b93cfe8a4421977c0d03b9aa0496&chksm=c32070ebf457f9fd8798849bab23bf7b59f2874d8efa6a69021ede23be6912fdc7ae78e31d28&scene=21#wechat_redirect)

[2024总结——顺势破局，把副业兴趣干成了事业！](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247532384&idx=1&sn=c668a1fcb72886cc07d02926afc913ef&scene=21&token=1862652836&lang=zh_CN#wechat_redirect)

[2024总结——6个月，从大学生到年入百万公司老板](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247532386&idx=1&sn=c7334fb3b823834a4a30ce57988875f8&scene=21&token=1768923821&lang=zh_CN#wechat_redirect)

[2024复盘-做副业一不小心做成了公司](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247532390&idx=1&sn=07604b226c3d8fd0c47223419c409a4b&scene=21#wechat_redirect)

[2025，入局AI了！](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247532417&idx=1&sn=dbc27667b819e50b078d2cfa41e24361&scene=21#wechat_redirect)

[AI视频创业小作坊线下办公两个月，我们到底卷出了啥？](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247533389&idx=1&sn=b6be78fdf625e4fe2fccc260eb32cf2f&scene=21#wechat_redirect)

[西羊石使用说明书和产品清单](https://mp.weixin.qq.com/s?__biz=Mzk0NDU5MTk3OA==&mid=2247534563&idx=1&sn=4a515d64e3b058ad55e8eeaf16cb38a9&scene=21#wechat_redirect)
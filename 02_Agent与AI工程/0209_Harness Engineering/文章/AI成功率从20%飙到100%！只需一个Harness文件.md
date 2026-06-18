---
title: AI成功率从20%飙到100%！只需一个Harness文件
author: 新智元
date: ASI启示录ASI启示录
url: https://mp.weixin.qq.com/s?__biz=MzI3MTA0MTk1MA==&mid=2652702499&idx=2&sn=f37aa6ca155b5ac2b6c8c7db624c807e&chksm=f0e598e282551be2e5f852058e01a62ffd29267eeeb8682838b62c1a3384131a96a27a98efa2&mpshare=1&scene=24&srcid=0603lcbCZsLk98Hvc70cVYUS&sharer_shareinfo=b2b2957933cc2f1c0f57f4da53184186&sharer_shareinfo_first=b2b2957933cc2f1c0f57f4da53184186#rd
---

### **新智元报道**

##### **【新智元导读】**Anthropic实锤：Claude裸跑模型，9美元全废；但是套上Harness花200美元效果直接起飞。AI效果不好？别再纠结换模型了！OpenAI和Anthropic都在用的Harness工程，一文讲透。

最近，AI圈子里一个逃不开的话题就是Harness。

甚至，连DeepSeek最近也在开始招聘Harness工程师。

那么，到底什么是Harness？

**Harness，**围绕AI编程智能体搭建的一整套工程基础设施，由五个子系统组成：指令、工具、环境、状态、反馈。

为什么值得专门讲它？

因为2026年前后，Anthropic和OpenAI几乎同时在各自的工程实验里给出了同一个结论——**AI编程智能体频频失败，问题不在模型，在模型之外的Harness**。

两家分别用一组对照实验当证据。先看数据。

**两组数据对照**

**Anthropic对照实验**——同一个Opus 4.5模型，同一道编程题：

多花的191美元，全花在验证循环上——每写一段代码就跑测试，不通过就改，直到真正通过。

**OpenAI百万行实验，**Codex团队在真实仓库上验证：

实验只改了一件事——仓库根目录加了一个AGENTS.md文件，不到100行markdown。

**Harness是什么**

Harness不是工具，也不是提示词技巧，是围绕智能体的一整套工程基础设施，由五个子系统组成，每一个对应一种具体失败模式。

**指令子系统（Instructions）**

仓库根目录的一个markdown文件——OpenAI阵营叫AGENTS.md，Anthropic阵营叫CLAUDE.md。

Codex、Claude Code、Cursor启动时自动读取并注入「系统提示词」。

**解决：**智能体不知道项目约定，瞎写代码（风格不一致、用错包管理器、随手执行破坏性命令）。

不到15行，把项目约定从反复重申变成启动时自动注入。

**工具子系统（Tools）**

限定智能体能调用哪些命令。

Claude Code用.claude/settings.json，Codex用~/.codex/config.toml。

**解决：**越权操作（rm-rf误删、gitpush--force覆盖远端、不该联网时调外部API）。

允许的直接跑，禁止的直接拒，灰色地带的弹确认。

**环境子系统（Environment）**

锁定依赖版本、运行时配置、数据库状态。

实现：setup.sh/Dockerfile/devcontainer.json。

**解决：**这台机器上能跑的虚假环境（本地通过，CI一跑就废）。

关键一行--frozen-lockfile——智能体无法擅自升级任何依赖。

**状态子系统（State）**

把跨会话进度、断点、未完成任务持久化到PROGRESS.md，新会话第一件事读它。

**解决：**跨会话失忆（第二个会话从零开始，写出和第一个会话冲突的代码）。

在AGENTS.md固化约定：**新会话第一件事读PROGRESS.md；任务完成或断点变化，立即回写**。

**反馈子系统（Feedback）**

机器可执行的验证命令——测试、lint、类型检查、构建。

智能体宣布完成前必须跑通，退出码不为0就不算完成。

**解决：**过早宣布胜利（说Done!但一行跑不通）——Anthropic 9美元裸跑实验的核心死因。

**三大致命失败模式**

Anthropic和OpenAI的实验，不约而同指向了智能体最常见的三种致命失败模式。

**过早宣布胜利**

**场景：**智能体写完500行功能，输出已完成。合并代码——CI红屏，type check报12个错，单测一个没跑过。

**根因：**没有强制反馈循环。判定来自自我感觉，不来自机器可验证的事实。

**解法：**反馈子系统。把判定权移交给退出码——**退出码≠0，任务≠完成。**

**上下文焦虑（ContextAnxiety）**

**场景：**长任务做到70%，上下文Token数快撑满窗口。智能体开始赶进度——跳过测试、删边界处理、写stub收尾、宣布完成。

**根因：**没有断点续传。感知到上下文压力时，智能体会试图在这个会话内做完所有事，哪怕代价是质量崩塌。

**解法：**状态子系统+主动重启。每完成一个子任务立即回写PROGRESS.md；上下文Token用量超70%，主动停下、写完断点、开新会话。

**跨会话失忆（Cross-SessionAmnesia）**

**场景：**第一个会话写了用户模块，第二个会话写订单模块——智能体不知道用户模块已存在，又写了一遍getUserById，跟前一版接口签名冲突。

**根因：**没有持久化状态+没有首读约定。

**解法：**状态子系统+指令子系统组合。PROGRESS.md维护已完成功能清单；AGENTS.md写明开会话第一件事读PROGRESS.md；冲突时以代码为准——仓库本身是唯一事实来源。

**五步从零搭一个Harness**

搭建一个Harness，并不难。

下面五步用文本编辑器即可完成，加起来不超过200行配置。

**第1步·根目录建AGENTS.md**

touch AGENTS.md。至少三块：项目说明、禁止操作、完成定义。

**第2步·配permissions**

.claude/settings.json或~/.codex/config.toml。最小两条：

**第3步·写setup.sh锁环境**

已有Dockerfile/devcontainer.json可跳过。

否则写一个setup.sh，把所有版本写死。最关键一行：pnpminstall--frozen-lockfile。

**第4步·建PROGRESS.md**

touchPROGRESS.md，四块：已完成、进行中、待办、已知问题。提交进git，当成项目自身的一部分维护。

**第5步·在AGENTS.md末尾固化完成定义**

写明pnpm type check/test/lint/build四个命令，退出码不为0就不算完成。如果项目还没有这些命令，今天就配上。

**没有反馈循环，Harness等于没装**——这是Anthropic 9美元实验的核心教训：前四步全做对，第五步缺位，依然全废。

**两家殊途同归**

过去一年所有人都在追下一个更强的模型。

2026年，Anthropic和OpenAI用两组不同的实验给出了同一个答案——**别先换模型，先把Harness装好**。

模型能力决定上限，Harness决定你能用到上限的几成。

没有Harness，Opus 4.5跑出的代码连编译都过不去；有了Harness，小一档的模型也能稳定交付。

下一个更强的模型当然会再抬一截上限。但今天连Harness都没装，下一个模型来了，成功率依然停在20%。

与其等下一个模型，现在就安装Harness。

参考资料：

https://walkinglabs.github.io/learn-harness-engineering/en/

编辑：大卫

**秒追ASI**

**⭐点赞、转发、在看一键三连⭐**

**点亮星标，锁定新智元极速推送！**
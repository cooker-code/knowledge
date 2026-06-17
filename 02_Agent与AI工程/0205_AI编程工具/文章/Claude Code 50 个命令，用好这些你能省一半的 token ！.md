---
title: Claude Code 50 个命令，用好这些你能省一半的 token ！
author: 涛哥聊Python
date: 彭涛主创团队彭涛主创团队
url: https://mp.weixin.qq.com/s?__biz=MzA5MTkxNTMzNg==&mid=2650324988&idx=1&sn=69490b0cdba51b8759acbe75a3ac27f7&chksm=89197c3a58e1da3ee03d366bbb31e026d184620098f7f0fb7a56802bf3f5ab21754266805e6e&mpshare=1&scene=24&srcid=0509BIyKW79kB21MxJOgBtIz&sharer_shareinfo=61b3e59e932a37652c91b1c03184db27&sharer_shareinfo_first=61b3e59e932a37652c91b1c03184db27#rd
---

**点击上方卡片关注我**

**设置星标 学习更多AI出海知识**

用 Claude Code 的人越来越多，但到目前为止，大部分人的用法还是很原始，打开终端，输需求，等结果，上下文满了就重开一个 session，模型从头到尾只用默认的，更别提什么 subagent、分叉对话了。

这就像你买了辆车，每天只用来听收音机。

Claude Code 内置了 50 多个 slash 命令，但真正决定效率的不是记住多少命令，而是掌握几套组合打法。

这篇文章不列清单，直接讲 5 个实战场景，每个场景告诉你什么时候该用什么命令，以及为什么这么用。

## 场景一：上下文快爆了，怎么抢救

最常见的问题，跑了一个复杂的调试 session，来来回回十几轮，突然发现 Claude 的回答开始变蠢，它忘了前面的结论，开始重复已经试过的方案，大概率是上下文窗口快满了。

系统虽然会在 92% 的时候自动压缩，但自动压缩是“无差别删减”，它不知道哪些信息对你重要。

正确打法：

```
/context          ← 先看一眼，上下文占了多少  
/compact keep all api decisions and error patterns  
                  ← 带提示词手动压缩，告诉它保留什么  
/cost             ← 压缩后确认 token 消耗下来了没
```

关键是 /compact 后面跟的那句话。不带提示词的 /compact 和自动压缩没太大区别，但加上 keep all api decisions 这种指示，Claude 就知道什么该留什么该删。

正确习惯应该是：上下文到 50% 就手动 compact 一次，因为压缩本身也消耗 token，越早做越便宜，留下的有效信息也越多，等到 92% 再触发，该丢的和不该丢的都一起被削了。

## 场景二：一个 session 里同时处理不同难度的任务

很多人不知道 Claude Code 可以中途切换模型和推理深度，一个 session 从头到尾都用 Opus + max，简单任务也烧最贵的配置，纯浪费。

```
/model sonnet     ← 写简单 util 函数、改样式、跑格式化  
/effort low       ← 搭配低推理深度，快且便宜  
  
/model opus       ← 遇到架构决策、复杂 bug  
/effort max       ← 切回最强配置  
  
/fast on          ← 需要快速迭代时开启（Opus 4.6 跑 2.5 倍速）
```

切换模型不会丢上下文，这意味着可以在同一个 session 里灵活调整，写样板代码用 Sonnet low 省钱，遇到真正需要深度思考的问题再切 Opus max。

算一笔账：假设一个 session 里 60% 的对话是简单任务，如果全程 Opus，那 60% 的钱花在了不需要 Opus 的地方。

光这一个习惯，一个月能省 40% 的费用。

## 场景三：方向不确定，怎么低成本试错

有个需求，但不确定该用方案 A 还是方案 B，传统做法是选一个跑，跑完发现不对再回来跑另一个，前面的 token 全浪费了。

```
/plan implement caching with Redis vs in-memory LRU  
     ← 先不动手，让 Claude 出两个方案的对比  
  
（看完 plan 觉得都想试）  
/branch redis-approach    ← 分叉出去跑 Redis 方案  
/branch lru-approach      ← 再分叉一个跑 LRU 方案
```

/plan 是最被低估的命令，它让 Claude 进入“只想不做”模式——写出完整方案但不碰任何文件，方向如果不对，只浪费了一轮 plan 的 token（通常几百 token），而不是浪费整个实现过程的 token（可能几万）。

/branch 更进一步——直接分叉对话，两个方向同时跑，互不干扰，跑完比较结果，哪个好用哪个。

还有一个容易忽略的：/rewind，如果已经跑了一半发现方向错了，不需要新开 session，直接 /rewind 回到之前的 checkpoint，保留了前面正确的上下文，只回退错误的部分。

## 场景四：大项目里怎么不让上下文互相污染

在做一个大功能，涉及前端、后端、数据库三块，如果在一个对话里全部搞定，后端的代码细节会占满上下文，等做前端的时候 Claude 已经“忘了”前端相关的约定。

```
/agents           ← 把后端、前端、数据库拆成三个 subagent  
  
（或者手动管理范围）  
/add-dir ./frontend    ← 只让 Claude 看到前端目录  
/add-dir ./backend     ← 切到后端时再加  
  
/review           ← 每个模块写完后跑一次 code review  
/simplify         ← 最后精简一遍
```

/agents 的精髓不是并行（虽然确实能并行），而是上下文隔离，每个 subagent 在独立的空间里工作，消耗几万 token，但返回的只是一句两句的结论，主会话的上下文始终保持干净。

/review 和 /simplify 推荐养成习惯：写完一个模块就 /review 一次查问题，最后整体 /simplify 去掉 Claude 特有的“过度抽象”毛病。

## 场景五：让 Claude 融入你的日常工作流

大部分人把 Claude Code 当成一个独立工具在用，但它可以和 IDE、Git、CI/CD 深度打通。

**日常开发流：**

* /init — 新项目第一件事，生成 CLAUDE.md。
* /memory — 随时更新项目约定。
* /ide — 连上 VS Code / Cursor，获取当前打开的文件上下文。
* /hooks — 配置自动化：保存文件后自动跑 lint，提交前自动 review。

**PR 工作流：**

* /pr-comments #42 — 把 reviewer 的评论直接拉进来，Claude 读完直接改代码。
* /summary — 生成变更摘要，直接贴到 PR description。
* /diff — 最后确认一遍改了什么。

**团队协作：**

* /share — 把 session 分享给同事。
* /rc — 让同事从手机/网页远程控制你的 session。
* /export — 导出 session 做技术文档或 onboarding 材料。
* /hooks 是真正能改变工作方式的。

可以配置：每次 Claude 修改了文件就自动跑类型检查，每次完成一轮对话就自动跑测试，把那些每次都手动做的检查全部自动化。

## 写在最后

50 个命令看着多，但核心逻辑就三点：

省钱：别让贵的模型干便宜的活，/model 切模型，/effort 调深度，/compact 控制上下文——这三个加起来能砍掉一半的费用。

提效：别用一个对话做所有事，/plan 先想后做，/branch 并行试错，/agents 拆分任务——这三个让你从“线性工作”变成“并行工作”。

自动化：别手动重复，/hooks 绑事件，/loop 定时跑，/init + /memory 维护项目记忆——这三个让 Claude 从“被动响应”变成“主动协作”。

把这 9 个命令用起来，其他 41 个慢慢探索就行。

**延伸阅读：**

• Claude Code 官方文档：https://docs.anthropic.com/en/docs/claude-code 

• Claude Code 源码架构分析：https://github.com/VILA-Lab/Dive-into-Claude-Code

[我们出海社区终于有自己的网站了！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490919&idx=1&sn=975a80f7a8f8805296fc917b6c18b89c&scene=21#wechat_redirect)

**欢迎关注，这个账号还会持续分享更多AI编程、出海工具、实战经验、踩坑记录。**

**想了解更多可以加我 vx: 257735 聊。**

**[出海赚钱案例：一个人做了个开源UI库，不融资不投广告，45天30万美元](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490959&idx=1&sn=7899b49fc195dfbd1220f64a0d12d12b&scene=21#wechat_redirect)**

**[出海建站必备：一小时搞定自建邮件，免费！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490935&idx=1&sn=767ae180c54603f961cf1c4d49e18a55&scene=21#wechat_redirect)**

**[OpenClaw 真香！我让它每天帮我干这些活](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490327&idx=1&sn=84003d687dce8869dd36746a88657792&scene=21#wechat_redirect)**

**[出海赚钱案例：一个人用 PHP 做到月入 17 万美金，利润率 99%！](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490792&idx=1&sn=25aa972bc87fc276d52e217e64871594&scene=21#wechat_redirect)**

**[（2026年最新）Codex CLI 国内使用全攻略：终端 + VSCode + Cursor + Opencode 四种姿势全搞定](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247490089&idx=1&sn=0a7f51a79aea7e1f372ea9e705a1e7be&scene=21#wechat_redirect)**

**[从海外公司注册到 Stripe 收款，跑通了出海收付款全流程（实操分享）](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489551&idx=1&sn=08058b274add835f37b3374fa43b6757&scene=21#wechat_redirect)**

**[玩转 Claude Code Hooks：让自动化渗透到每个环节](https://mp.weixin.qq.com/s?__biz=Mzg3NzU2NjY3OQ==&mid=2247489503&idx=1&sn=b81277c62e501d497b7f621e3a726b34&scene=21#wechat_redirect)**
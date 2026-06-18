---
title: Async Vibe Coding ，AI 辅助编程新交互范式
author: 祝威廉
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484998&idx=1&sn=39164cf3c6ab264213826cd518b300b5&chksm=e999e1c4e3cdb19452ead47454745ffd749238a6e3e16c005b7cfa2e90de2e0e4e8289679db6&mpshare=1&scene=24&srcid=1110YLEXxdkmFq1bln8xTwJq&sharer_shareinfo=9adaa9d0ae48827ae408aa4eb7a4a02e&sharer_shareinfo_first=9adaa9d0ae48827ae408aa4eb7a4a02e#rd
---

前言

Claude Code 从刚开始出现到现在，我喜欢的创新其实就两个： subagent 和 内置 todo，再之后就没啥创新了，现在虽然每天更新都很勤快，但基本就是交互的一些小改进。 而其他工具，诸如 codex 以及后续一堆的跟进产品，几乎都毫无创新在追赶而已。

现在 Vibe Coding 实际卡在两个环节：

1. 并行度不足。现在需要用户开多个 tab 进行并行，或者自己通过worktree 来进行无冲突并行，繁琐，并且最后合并是个大难题。这就导致开发效率被极大的阻碍。

2. Review 负担太重。 Vibe Coding 写代码时有多happy，Review 就有多酸爽。本质还是AI的交付质量不够高。

解决方案：Async Vibe Coding

为此，auto-coder 社区推出了 async vibe coding。 具体提出三个创新性设计：

1. 首先我们把 CLI （主agent）退化成了一个任务管理系统。

2. 人类第一次可以精确的配置 subagent 的运行时间，确保交付质量。

3. 人类review 完后， 可以让 主agent 同时自动合并多个异步subagent的结果。

auto-coder.chat 通过创新的异步 subagent 完美解决实现了对这三者的支持。解决了之前的并行度不足，review 负担太重两个问题。

实战体验

```
# python 3.10 / 3.11 / 3.12 三个版本pip install -U auto-coderauto-coder.chat
```

现在，就可以开始提交任务：

```
/models /list
```

给你想要的模型添加 key:

```
/models volcengine/deepseek-v3-1-terminus <YOUR_API_KEY>
```

然后配置该模型：

```
/conf model:volcengine/deepseek-v3-1-terminus
```

现在可以开始工作了：

```
coding@auto-coder.chat:~$ /auto /async /time 10m /name async_drop_task  "在 @./src/autocoder/inner/async_command_handler.py 中添加一个新的子命令 drop, 然后删除相应的task"
```

其中:

1. /async 表示这是异步 subagent任务

2. /time 10m 告诉 subagent, 你要至少跑十分钟，仔仔细细检查了再给我review,而不是写完就给我review，加重我的review 负担。

3. /name async\_drop\_task 则表示当前的任务名称，需要保持唯一

回车提交后：

可以看到，系统已经提交异步任务，底部 status bar Async Tasks 那部分显示有一个任务在运行。你可以继续做其他的操作了。

你也通过其他指令查看任务列表：

也可以通过具体查看某个任务当前的状态：

还可以通过 cursor 来查看某个任务已经完成的修改，还可以查看实时日志：

```
coding@auto-coder.chat:~$ !cursor ~/.auto-coder/async_agent/tasks/async_drop_task
```

打开后，可以关注着两个主要区域：

你可以同时提交多个任务，给每一个任务指定运行时间，并且不用担心任务之间是否会冲突。

一旦任务完成，并且觉得合适，就可以进行合并操作。具体做法为：

```
coding@auto-coder.chat:~$ /auto /merge async_drop_task
```

指定任务名称，即可将任务的修改合并会主分支。注意，这里是 auto-coder.chat 唯一串行的地方。

如果你希望同时合并多个任务，则按逗号分割即可：

```
coding@auto-coder.chat:~$ /auto /merge task1,task2,task3
```

如果多个任务之间有冲突，系统会自动解决合并冲突。

Bonus: Best Of N

除了可以指定运行时间来提升AI的交付质量以外，我们还可以通过让多个模型同时运行相同的任务，比如：

```
/auto /async /model v3 /time 10m /name task1 "需求"/auto /async /model glm4-6 /time 10m /name task2 "需求"
```

上面的例子同时提交任务给到 v3, glm4-6 两个模型，然后他们会独立运行。最后运行结束后，你可以选择其中一个合并，也可以让主agent 自动选一个合并：

```
coding@auto-coder.chat:~$ /auto /merge task1,task2 -query "两个任务选择一个你认为最好的做合并"
```

这样就可以实现系统自动选择最好的实现来合并到主分支上。

总结

根据上面的示例，可以看到，我们只要填写任务，然后设置任务时间，系统即可异步运行。然后可以根据用户名无感合并。对用户来说，你可以设置同时运行10个并发（可能冲突的）任务，然后通过一次即可合并。review 工作因为我们通过延长任务时间从而使得代码质量得到很大提升，显著的降低的review 工作量。

让我们使用  auto-coder.chat 的 async vibe coding ，再次提升10倍效率！
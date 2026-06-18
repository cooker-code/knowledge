> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 一些使用 claude code 的小技巧
author: 独元殇
date: 独元殇独元殇
url: https://mp.weixin.qq.com/s?__biz=MzUxODQ3ODIzMw==&mid=2247487056&idx=1&sn=81f46f34760eb480c23f00c80391afd2&chksm=f888bf74b43adedd35a2ca7aa16cb243257b7f6cb978f60be347a12a76bc4db4096a777cd1c8&mpshare=1&scene=24&srcid=0427uuqsZTdvr1Q6gne5b53t&sharer_shareinfo=d8d80e1acb4c78cfe045f27062e0e13a&sharer_shareinfo_first=d8d80e1acb4c78cfe045f27062e0e13a#rd
---

> ⭐️ 博主我会**每天**在公众号发布一些关于建网站、编程、SEO、AI 类的**原创小文章**，纯手打，无 AI 辅助，篇篇力求值得收藏，欢迎大家关注本公众号！祝大家开心每一天！⭐️

今天我来讲一些使用 claude code 的小技巧吧，里面参考了 铁锤人 的一些经验，是一些笔记。

https://x.com/lxfater/status/2041448785516343592

首先是启动。

目前(2026-4-7)只要你使用的是官方的 opus 或者 kimi 2.5 、GLM-5 模型，大胆使用最高跳过权限的这个命令启动即可，这些模型足够聪明，几乎不给你乱搞：

```
claude --dangerously-skip-permissions   # YYSD 永远的神啊！
```

（当然，不喜欢命令行的，可以去官网下载 GUI 版本，只是用着有点点别扭，我不再赘述）。

你会发现，有时候 Ctrl + C 强制退出 CLI 界面后，再进入，好像是重开了一个会话。

其实这个时候，系统仍然保存着上一次对话，如果你想恢复，就一个斜杠命令下去：

```
/resume

# 或者

claude --dangerously-skip-permissions -c   # 后面有个 -c ，超级实用！！！
```

非常省劲儿！

## 自动化必看

著名的 AI 博主，铁锤人 lxfater，最近分享了一个很好的命令行命令，我的示例如下：

```
$ claude -p "100字介绍一下安阳扁粉菜" --output-format json
```

这样的话，它会直接返回 JSON 格式的一个智能结果，超级好用，自动化流程必备：

当然 如果你不加后面的 --output-format json 的话，它会直接给你文本，但是在自动化的时候，信息更多一点更好，是吧。

## 善用 ESC

我们程序员使用命令行，肯定特别喜欢使用 Ctrl + C 来关闭、终止进程。

但是！

即便在 claude code 虽然这个快捷键也有用，而且里面一闪一闪还在提醒你按这个.... 我们依然不要用。很容易退出窗口。我起码下意识退出它有 10 次有余了。

最好的办法是 ESC ，没错，ESC ，按一下，一般就起效。一直按，也不会退出窗口。

## 一个成熟的 vibe coding 指导 skill

即便你是个程序员，你使用 claude code 等工具来 vibe 时，也很难有什么方法论，就是不停的随心所欲而已。

然后国外有个老大佬，把一套成熟的 Vibe coding 方法论，打包成了一个 skill 。

这是一个很好的辅助，你在使用时，会频繁的输入信息，然后让你Vibe 的更严谨一些：

> 在您确认设计方案后，您的智能体会制定一份实施计划，这份计划足够清晰，即使是一位品味不佳、缺乏判断力、不了解项目背景且厌恶测试的初级工程师也能轻松遵循。它强调真正的红/绿测试驱动开发、YAGNI（您不会需要它）和 DRY 原则。 接下来，一旦你说"开始"，它就会启动一个*子代理驱动开发*流程，让代理们逐一处理每个工程任务，检查并评审他们的工作，然后继续推进。Claude 通常能够自主工作数小时而不偏离你制定的计划。

是不是很爽啊。

就是很废字，但这点代价微乎其微。

## 两个 Vibe 辅助软件

一个叫 Cmux ，专为同时跑多个 coding agent 而生，灵活分屏 。

另一个叫 Vibe Island ，这个比较新。跟上面那个差不多，不过，更加的丝滑。
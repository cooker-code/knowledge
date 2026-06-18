---
title: Code-Mode MCP：通过代码执行 MCP 工具，省下60%的 token
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475229&idx=1&sn=380dc78fab13543f20d3a4a03916ca85&chksm=8fe0b604e86f8fd9a6d106bfc6a9aac3736fecbbe7b15f676b40287863fb5508c8e208035d10&mpshare=1&scene=24&srcid=1202PWyEIFYGt6dlnFdNdV4O&sharer_shareinfo=d821fd1099dcc83b0ac40288fb715ac2&sharer_shareinfo_first=d821fd1099dcc83b0ac40288fb715ac2#rd
---

👀 有人实现了[Claude官方说的MCP优化](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475133&idx=1&sn=229f9f3911e17e4f29a86dff30ba50c1&scene=21#wechat_redirect)，一起看看

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式

---

 

### 写在前面

有没有觉得，现在的 AI Agent 调工具（Function Calling）有时候显得...挺“笨”的？

想象这么一个场景：你要让 AI 帮你在 GitHub 上查一个 Issue，然后根据 Issue 的内容去查相关的 PR，最后把这两个信息汇总发到 Slack 上。

在传统的模式下，这就像是你在跟一个反应迟钝的助手“挤牙膏”：

1. 1. 你：查一下 Issue #123。
2. 2. AI：（思考...调用 GitHub 工具...等待网络...）查到了，内容是...
3. 3. 你：好，那根据这个查一下 PR #456。
4. 4. AI：（思考...调用 GitHub 工具...等待网络...）查到了，状态是 Merged。
5. 5. 你：行，把这些发到 Slack 吧。
6. 6. AI：（思考...调用 Slack 工具...）发好了。

这中间的每一次来回，都是一次 **HTTP 请求**，都是一次 **LLM 推理**，更要命的是，都要消耗昂贵的 **Token** 来重复传输上下文。这简直就是对算力和时间的双重浪费（都是钱啊）。

那么，有没有一种办法，能让 AI 一次性把事情做完？

没错，**Code Mode (代码模式)** 就是为了解决这个问题诞生的。而今天我们要聊的这个仓库 `utcp-code-mode`，就是实现了Claude官方说的用代码执行代替MCP调用的“神器”。

---

### 什么是 Code Mode？

简单来说，Code Mode 就是把 AI 从“只会按按钮的操作员”变成了“会写脚本的程序员”。

以前我们给 AI 一堆工具，它只能一次调一个；现在，我们给它一个**沙盒环境**，告诉它：“嘿，这些工具都在这儿，你直接写一段代码，把它们组合起来用，跑完直接给我结果。”

这就像是上面的那个例子，在 Code Mode 下变成了这样：

1. 1. 你：帮我处理 Issue #123 和 PR #456 并发到 Slack。
2. 2. AI：收到。（反手写了一段 Python/JS 代码，里面并发调用了 GitHub 和 Slack 的接口，并在本地做好了数据拼接）
3. 3. 系统：执行代码 -> 完成。

一次请求，全部搞定。根据仓库里的 Benchmark 数据，这种方式能让复杂任务的执行速度提升 **80%** 以上，Token 消耗减少 **60%**。这哪里是优化，简直是降维打击。

---

### 它是怎么工作的？

翻看了一下 `utcp-code-mode` 的源码，我发现它的实现思路非常清晰，同时也非常大胆。

这个仓库主要包含了两大块核心实现：**Python 库** 和 **TypeScript 库**，以及一个 **MCP Bridge**。我们以 Python 版为例，来看看它是怎么把大象装进冰箱的。

#### 1. 给工具加上类型

AI 虽然聪明，但它怎么知道你的工具叫什么？参数是什么类型？返回值长啥样？

如果我们直接把原本的 JSON Schema 丢给 AI，它也能看懂，但写起代码来很容易出错。于是，这个库做了一件很聪明的事：**动态生成类型定义**。

在 `python-library/.../code_mode_utcp_client.py` 里，有一个 `tool_to_python_interface` 方法。它会把工具的 JSON 定义转换成 Python 的 `TypedDict`。

比如原本丑陋的 JSON Schema，会被转换成这样清爽的 Python 代码提示给 AI：

```
# Namespace: github  
class GetPullRequestInput(TypedDict):  
    owner: str  
    repo: str  
    pull_number: int  
  
# Access as: github.get_pull_request(args)
```

这样 AI 在写代码时，就知道：“哦，原来我要调用 `github.get_pull_request`，参数是这几个。” 这不仅提高了准确率，还利用了 LLM 强大的代码生成能力。

#### 2. 代码在安全沙盒里执行

这是最让人头秃的部分。允许 AI 执行代码？这也太危险了吧！万一它写了一句 `import os; os.system('rm -rf /')` 怎么办？

别慌，作者显然考虑到了这一点。在 Python 的实现中，它引入了 `RestrictedPython`。

在 `_run_with_restricted_python` 方法里，我们可以看到一个严密的“监狱”被建立了起来：

* • **禁止危险模块**：你只能用 `json`, `math`, `datetime` 这些人畜无害的标准库。想用 `os`？没门。
* • **重写 Builtins**：连 Python 原生的 `open`、`exec`、`eval` 都被禁用了。
* • **工具注入**：真正能对外交互的，只有我们注册进去的那些 UTCP 工具（比如前面提到的 `github` 模块）。

TypeScript 版本则是利用了 Node.js 的 `vm` 模块来做上下文隔离，思路是异曲同工的。

#### 3. 捕获输出和报错

代码执行过程中，AI 可能会用 `print` (Python) 或 `console.log` (TS) 来输出调试信息。Code Mode 会贴心地把这些标准输出全部捕获下来，和最终的返回值一起打包返回。

这对调试 Agent 的思维链（Chain of Thought）简直是神器。你可以清楚地看到 AI 是怎么一步步思考、计算、然后得出结论的。

---

### 怎么玩？

用法出乎意料的简单。如果你用 Python，大概是这样子：

```
from utcp_code_mode import CodeModeUtcpClient  
  
# 1. 创建客户端  
client = await CodeModeUtcpClient.create()  
  
# 2. 注册你的工具 (这里假设你已经有了一个 MCP 或 HTTP 的工具配置)  
await client.register_manual(github_tool_config)  
  
# 3. 见证奇迹的时刻  
# 注意：这里直接传入一段代码，而不是函数名  
result = await client.call_tool_chain('''  
    # AI 生成的代码  
    pr = await github.get_pull_request(owner='microsoft', repo='vscode', pull_number=1234)  
    comments = await github.get_pull_request_comments(owner='microsoft', repo='vscode', pull_number=1234)  
      
    # 在沙盒里直接处理数据，不用传回 LLM 处理  
    summary = {  
        "title": pr["title"],  
        "comment_count": len(comments)  
    }  
    print(f"分析完成: {summary}")  
    return summary  
''')  
  
print(result['result']) # 输出 summary 字典  
print(result['logs'])   # 输出 ['分析完成...']
```

看到没有？以前需要跑好几趟网络请求的逻辑，现在被封装成了一段字符串，一次 `call_tool_chain` 就搞定了。

**更有意思的是**，这个仓库还提供了一个 `code-mode-mcp` 包。这意味着什么？这意味着你可以把这个“代码执行器”直接作为一个 MCP Server 挂载到 **Claude Desktop** 上！

配置好之后，你在 Claude 里说：“帮我分析下我的代码库”，Claude 就会自动在后台调用这个 `call_tool_chain` 工具，自己写脚本去跑分析，而不是傻傻地一个个文件读。

---

### 一些思考

浏览完这个仓库，我有几点感触：

1、**从“调用”到“编排”**：传统的 Function Calling 是单点的调用，而 Code Mode 实际上是把“编排（Orchestration）”的权力下放给了 LLM。这不仅解放了人类开发者，也释放了 LLM 的逻辑潜能。

2、**计算的移动**：以前我们推崇“数据不动代码动”，Code Mode 其实也是这个逻辑。与其把海量的 API 数据（比如几千条评论）传回给 LLM 去数数，不如让 LLM 写一段几行的小代码，去数据所在的地方（或者代理层）运行。

3、**安全永远是达摩克利斯之剑**：虽然 `RestrictedPython` 和 `vm` 做了很多努力，但只要是执行任意代码，风险就永远存在。作者也在文档里很诚实地写道：这是一个“合作式沙盒（Cooperative Sandbox）”，适合用在 AI Agent 这种场景，但绝对不要拿去跑不可信的用户代码。

### 总结

`utcp-code-mode` 是一个非常棒的尝试，它用一种优雅的方式解决了 Agent 开发中的效率和成本痛点。

如果你正在开发复杂的 Agent，或者受够了 Function Calling 的慢速和昂贵，强烈建议你试一试这个库。毕竟，谁不喜欢又快又省钱的方案呢？

（P.S. 记得去给作者点个 Star，开源不易，且用且珍惜。）

#### 参考资料

* • UTCP Code Mode Repository
* • RestrictedPython Documentation
* • Model Context Protocol (MCP) Specs

 

---

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～
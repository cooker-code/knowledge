---
title: Claude-Mem：让 Claude 像人一样记住我说过的话
author: 知识药丸
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649475240&idx=1&sn=2861eb4996ce2fd73b52737e2470fdf9&chksm=8feedd8d45950782056b4288c5a2d0b0c0f28afcf7ed9776281d7ba4c8ddaf659308ba2fe0ca&mpshare=1&scene=24&srcid=1204BoTqiQAZlXQaVswYADri&sharer_shareinfo=b0120e13b127f63887e203d5dd7ef9bf&sharer_shareinfo_first=b0120e13b127f63887e203d5dd7ef9bf#rd
---

👀 自然语言查历史 + Endless Mode → 像问同事一样问“昨天那个bug怎么修的？”就能调出完整记录

[《贾杰的AI编程秘籍》](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MTM1NA==&mid=2649474437&idx=1&sn=4ae1516afc0ec71b7638dd79daaae2cb&scene=21#wechat_redirect)付费合集，共10篇，现已完结。30元交个朋友，学不到真东西找我退钱；）

以及我的墨问合集《100个思维碎片》，1块钱100篇，与你探讨一些有意思的话题（文末有订阅方式

---

 

### 写在前面

3 个月前，我在用 Claude Code 开发一个项目。

一切都很顺利，直到我关掉终端，第二天回来继续干活。我打开 Claude Code，输入："继续昨天的工作，把那个 API 优化一下。"

Claude 愣了："什么 API？我们用的什么技术栈？"

我：？？？

于是我花了 20 分钟，重新给 Claude 讲了一遍整个项目架构：Express 做后端，SQLite 存数据，PM2 管进程，文件结构是这样的，昨天我们刚实现了认证...

讲完继续干活，写了几十行代码，突然 context limit 了。

好吧，重启 Claude Code。

然后，**又是 20 分钟的项目介绍**。

那一刻我意识到一个问题：Claude 就像《海底总动员》里的多莉，每次见面都是初次见面。

---

### 问题的本质

我们来看看为什么会这样。

Claude Code 的工作原理其实很简单：每次你开启一个新 session，它的"记忆"就是一张白纸。你说什么，它就知道什么。

这在短期对话里没问题。但如果你在做一个真实的项目，跨越多天、多个 session，问题就来了：

* • 第一天：实现了用户认证
* • 第二天：Claude 不记得你用的是什么认证方案
* • 第三天：Claude 不记得你的数据库 schema
* • 第四天：Claude 不记得你为什么要这么设计 API

每次都要重新解释，**这简直是噩梦**。

更要命的是 token 限制。Claude Code 每个 session 大概能用 50 次工具调用（读文件、写文件、运行命令等），每次调用可能消耗 1k-10k tokens。用着用着就满了，然后你得重启，然后又是 20 分钟的项目介绍...

这不对劲。人类不是这样工作的。

---

### 人类是怎么记忆的？

我们来想想人类的记忆是怎么回事。

假设你昨天和同事讨论了一个技术方案，今天你们继续讨论。你不需要重新讲一遍昨天说了什么，因为：

1. 1. **你们都记得关键决策**（"我们决定用 Redis 做缓存"）
2. 2. **你们记得遇到的问题**（"昨天那个并发 bug 还没修"）
3. 3. **如果忘了细节，可以查笔记**（"等等，那个函数叫啥来着？"）

这就是"**渐进式记忆提取**"（Progressive Disclosure）的概念。

* • **第一层**：我知道有这件事（索引）
* • **第二层**：我记得大概内容（摘要）
* • **第三层**：我可以找到完整细节（原始数据）

人类不会把所有细节都塞进工作记忆里，那样大脑会爆炸。我们只记住**关键信息**，需要时再去"查阅"详细内容。

那么问题来了：我们能不能给 Claude 也弄一个这样的记忆系统？

---

### Claude-Mem 登场

没错，这就是我为什么要做 claude-mem。

claude-mem 是一个为 Claude Code 设计的**持久化记忆压缩系统**。它的核心思路非常简单：

> **监听 Claude 做的每一件事，用 AI 压缩成摘要，在下次 session 时自动注入。**

听起来很简单对吧？但魔鬼在细节里。

#### 它是怎么工作的？

整个流程可以用这张图来理解：

```
Session 开始 → 注入最近的观察记录作为上下文  
     ↓  
用户提问 → 创建 session，保存用户的 prompt  
     ↓  
工具执行 → 捕获观察（读文件、写文件、运行命令等）  
     ↓  
Worker 处理 → 用 Claude Agent SDK 提取学习成果  
     ↓  
Session 结束 → 生成总结，准备下次使用
```

让我们逐步拆解：

**1. Session 开始 - 自动注入上下文**

当你启动 Claude Code 时，claude-mem 的 `context-hook` 会自动触发。它会：

* • 从数据库里取出最近的 10 条观察记录
* • 把它们以**索引形式**注入到 Claude 的上下文里

注意，这里用的是"索引形式"，不是完整内容。每条观察大概长这样：

```
🔴 [Decision] 选择 Express 作为 API 框架 (token cost: ~500)  
🔵 [Feature] 实现用户认证中间件 (token cost: ~450)  
🟤 [Bugfix] 修复并发写入导致的数据竞争 (token cost: ~380)
```

这样 Claude 就知道："哦，我之前做过这些事"，但不会被大量细节淹没。

**2. 用户提问 - 创建 session**

当你输入第一个问题时，`user-message-hook` 会创建一个新的 session，记录你的 prompt。

这个 session 会持续到你关闭 Claude Code（或者手动 `/clear`）。

**3. 工具执行 - 捕获观察**

这是最关键的部分。

每当 Claude 使用一个工具（比如 `read_file`、`write_file`、`bash` 等），`new-hook` 和 `save-hook` 会捕获这次操作，包括：

* • 工具名称和参数
* • 工具的输出结果
* • 时间戳和 session ID

这些原始数据会先存到数据库里，等待后续处理。

**4. Worker 处理 - AI 提取学习成果**

这是 claude-mem 最"优雅"的地方。

它会启动一个后台 worker（用 PM2 管理），这个 worker 会：

* • 监听新的观察记录
* • 用 **Claude Agent SDK** 调用 AI（就是 Claude 自己！）
* • 让 AI 分析这次工具调用，提取**关键学习成果**

比如，如果 Claude 刚刚执行了：

```
npm install express
```

Worker 会让 AI 生成一条观察：

```
{  
  "type": "decision",  
  "concept": "framework-choice",  
  "narrative": "选择 Express 作为 Web 框架，因为它轻量且社区成熟",  
  "files": ["package.json"],  
  "tokens": 480  
}
```

这条观察只有 ~500 tokens，但包含了**最重要的信息**。

**5. Session 结束 - 生成总结**

当你关闭 Claude Code 时，`summary-hook` 会生成一个 session 级别的总结：

```
本次 session 主要完成了项目初始化，选择了 Express + SQLite 技术栈，  
实现了基础的用户认证中间件，并修复了一个并发写入的 bug。
```

下次你启动 Claude Code 时，这个总结会出现在上下文的最顶部。

---

### 为什么这样设计？

你可能会问：为什么要这么复杂？直接把所有历史对话塞进去不行吗？

不行，因为 **token 是有限的**。

Claude 的 context window 虽然很大（200k tokens），但 Claude Code 的每次工具调用会消耗大量 tokens。如果你把所有历史对话都塞进去，很快就会爆掉。

claude-mem 的设计哲学是：**用 AI 压缩 AI 的记忆**。

* • 原始工具输出可能有 10k tokens
* • 压缩后的观察只有 500 tokens
* • **压缩率高达 95%**

而且，这种压缩是"有损"的，但损失的是**不重要的细节**。就像人类记忆一样，我们记住关键决策，忘记无关紧要的东西。

---

### Progressive Disclosure：按需获取详情

但如果 Claude 真的需要某个观察的详细内容怎么办？

这就是"渐进式披露"（Progressive Disclosure）的妙处。

claude-mem 提供了一个 `mem-search` skill，Claude 可以随时调用它来查询历史：

```
Claude: "我需要查一下之前我们是怎么实现认证的"  
[自动调用 mem-search skill]  
→ 返回完整的观察记录，包括源代码和上下文
```

这样，Claude 可以：

1. 1. **先看索引**：知道有哪些观察（~10 tokens/条）
2. 2. **再看摘要**：了解大概内容（~500 tokens/条）
3. 3. **最后查详情**：如果需要，获取完整的工具输出（~10k tokens）

这就像人类查资料一样：

1. 1. 翻目录 → 找到相关章节
2. 2. 看摘要 → 了解大概内容
3. 3. 读全文 → 获取所有细节

**最重要的是**，每一层都有明确的 token cost 标注。Claude 可以自己决定："这个观察看起来不太相关，不值得花 5k tokens 去查，我直接读源代码吧（可能只要 1k tokens）。"

这就是真正的"智能"。

---

### mem-search：自然语言查询你的项目历史

说到 mem-search，这是 claude-mem 最"强大"的功能之一。

你不需要记住任何命令。直接用**自然语言**问 Claude：

```
"上个 session 我们做了什么？"  
"我们之前修过这个 bug 吗？"  
"认证是怎么实现的？"  
"最近对 worker-service.ts 做了哪些改动？"
```

Claude 会自动调用 mem-search skill，去数据库里搜索相关的观察记录，然后告诉你答案。

mem-search 支持 10 种搜索操作：

1. 1. **全文搜索观察** - 在所有观察里搜索关键词
2. 2. **搜索 session 总结** - 找到相关的历史 session
3. 3. **搜索用户 prompt** - 查找你之前问过的问题
4. 4. **按概念搜索** - 比如 "discovery"、"problem-solution"
5. 5. **按文件搜索** - 找到涉及某个文件的所有观察
6. 6. **按类型搜索** - decision、bugfix、feature、refactor 等
7. 7. **最近上下文** - 获取最近的 session 上下文
8. 8. **时间线查询** - 查看某个时间点前后发生了什么
9. 9. **带时间线的搜索** - 搜索并返回相关时间线
10. 10. **API 帮助** - 查看搜索 API 文档

而且，mem-search 比传统的 MCP（Model Context Protocol）工具节省了 **~2,250 tokens**。

为什么？因为 mem-search 是一个"skill"，不是一个"tool"。它会在 Claude 需要时自动触发，而不是每次都把所有搜索选项塞进 context。

---

### Hybrid Search：语义搜索 + 关键词搜索

claude-mem 的搜索不是简单的关键词匹配。

它用了**混合搜索**（Hybrid Search）：

* • **SQLite FTS5** - 全文索引，快速关键词搜索
* • **Chroma Vector Database** - 向量数据库，语义相似度搜索

什么意思？

假设你问："我们是怎么处理并发问题的？"

* • 关键词搜索会找到包含"并发"的观察
* • 语义搜索会找到包含"race condition"、"mutex"、"lock" 的观察（即使没有"并发"这个词）

两者结合，准确率大幅提升。

---

### Beta 功能：Endless Mode

说到这里，我得提一下 claude-mem 的一个实验性功能：**Endless Mode**。

这个功能的目标是：让 Claude Code 的 session **永远不会耗尽 context**。

怎么做到的？

传统 Claude Code 的问题是：每次工具调用，完整的输出都会留在 context 里。用着用着，context 就满了。

Endless Mode 的思路是：**实时压缩 transcript**。

```
Working Memory (Context):     压缩后的观察 (~500 tokens/条)  
Archive Memory (Disk):        完整的工具输出，随时可以调取
```

每次工具执行后，Endless Mode 会：

1. 1. 立即用 AI 生成观察摘要（~500 tokens）
2. 2. 把原始输出存到磁盘
3. 3. 用摘要替换 context 里的原始输出

这样，context 的增长速度从 O(N²) 变成了 O(N)。

理论上，你可以进行 **20 倍以上的工具调用**，而不会耗尽 context。

但有个代价：每次工具执行后，需要额外 60-90 秒来生成摘要。

这就是为什么它还在 beta 阶段。如果你愿意等，你可以获得"无限"的 context。如果你需要速度，可以用稳定版。

（P.S. 你可以在 http://localhost:37777 的 Web UI 里直接切换 beta 和稳定版，数据会保留。）

---

### Web Viewer UI：实时看到你的记忆流

claude-mem 还有一个很"漂亮"的功能：Web Viewer UI。

当 worker 启动后，你可以打开 http://localhost:37777，会看到一个实时更新的界面，展示：

* • 当前正在运行的 sessions
* • 所有观察记录的时间线
* • 每条观察的详细信息
* • Token 使用统计

而且支持亮色/暗色主题切换（v5.1.2 加的）。

这个 UI 的作用不仅仅是"好看"。它能让你**直观地看到 Claude 的记忆是怎么形成的**。

比如你会发现：

* • 某些工具调用生成了大量观察
* • 某些观察被标记为"critical"（🔴）
* • 某些 session 的 token 消耗特别高

这些信息可以帮你优化你的工作流。

---

### 实际使用体验

我自己用 claude-mem 开发了 3 个月，效果**远超预期**。

最明显的改变是：**我不再需要"教" Claude**。

以前每次启动 Claude Code，我得花 5-10 分钟解释项目结构。现在我直接说："继续昨天的工作"，Claude 就知道该干什么。

第二个改变是：**Claude 会主动参考历史决策**。

比如我问："我们应该用 Redis 还是 Memcached 做缓存？"

Claude 会说："根据上次 session 的讨论，我们决定用 Redis，因为它支持持久化。"

然后附上一个 `claude-mem://` 引用链接，指向那次决策的观察记录。

这种感觉就像是在和一个**真正记得你们对话的同事**交流。

第三个改变是：**Token 使用更高效**。

因为 Claude 不需要每次都重新读一遍整个 codebase，它可以直接查询 mem-search："之前我们是怎么处理这个问题的？"

节省的 tokens 意味着你可以进行更多次工具调用，完成更复杂的任务。

---

### 技术细节：6 个 Lifecycle Hooks

如果你对实现细节感兴趣，claude-mem 的核心是 **6 个生命周期 hooks**：

1. 1. **context-hook** - Session 开始时注入上下文
2. 2. **user-message-hook** - 捕获用户的 prompt
3. 3. **new-hook** - 工具执行前的准备
4. 4. **save-hook** - 保存工具执行结果
5. 5. **summary-hook** - Session 结束时生成总结
6. 6. **cleanup-hook** - 清理临时数据

还有一个 **pre-hook**（不是 lifecycle hook）：

* • **Smart Install** - 检查依赖是否已缓存，避免重复安装（从 2-5 秒降到 10 毫秒）

所有 hooks 都是异步执行的，不会阻塞 Claude Code 的正常使用。

Worker Service 用 PM2 管理，提供 HTTP API（端口 37777），支持：

* • 10 个搜索 endpoint
* • Web Viewer UI
* • 健康检查和日志

数据存储在 SQLite 数据库里（`~/.claude-mem/claude-mem.db`），用 FTS5 做全文索引。

（详细架构可以看 Architecture Overview。）

---

### 安装和使用

安装超级简单。在 Claude Code 的终端里输入：

```
/plugin marketplace add thedotmack/claude-mem  
/plugin install claude-mem
```

然后重启 Claude Code。就这样，**没有任何配置**。

第一次启动时，claude-mem 会：

1. 1. 初始化 SQLite 数据库
2. 2. 启动 PM2 worker
3. 3. 打开 Web Viewer UI（http://localhost:37777）

之后的每个 session，claude-mem 都会自动工作：

* • 注入上下文
* • 捕获观察
* • 生成总结

你唯一需要做的就是：**正常使用 Claude Code**。

---

### 系统要求

* • **Node.js**: 18.0.0 或更高
* • **Claude Code**: 最新版，支持 plugin 系统
* • **PM2**: 进程管理器（已内置，不需要全局安装）
* • **SQLite 3**: 持久化存储（已内置）

基本上，只要你能跑 Claude Code，就能跑 claude-mem。

---

### 配置（如果你想调整的话）

claude-mem 开箱即用，但如果你想自定义：

**选择 AI 模型：**

```
./claude-mem-settings.sh
```

可以选择用哪个 Claude 模型来处理观察（默认是 `claude-sonnet-4-5`）。

**环境变量：**

* • `CLAUDE_MEM_MODEL` - AI 模型名称
* • `CLAUDE_MEM_WORKER_PORT` - Worker 端口（默认 37777）
* • `CLAUDE_MEM_DATA_DIR` - 数据目录（仅开发用）

一般来说，默认配置就够用了。

---

### 常见问题 & 排查

**Q: Worker 没有启动怎么办？**

A: 运行 `npm run worker:restart`。

**Q: 为什么没有看到上下文注入？**

A: 运行 `npm run test:context` 检查。可能是第一次使用，数据库里还没有观察记录。

**Q: 搜索功能不工作？**

A: 检查 FTS5 表是否存在：

```
sqlite3 ~/.claude-mem/claude-mem.db "SELECT name FROM sqlite_master WHERE type='table';"
```

如果问题还在，直接问 Claude："帮我诊断 claude-mem"，troubleshoot skill 会自动激活并提供修复方案。

（详细的排查指南看 Troubleshooting Guide。）

---

### 总结

claude-mem 解决的是一个很"简单"的问题：**让 Claude 记住你们的对话**。

但它的实现并不简单。它涉及到：

* • AI 驱动的内容压缩
* • 渐进式记忆提取
* • 混合搜索（语义 + 关键词）
* • 生命周期 hooks
* • 后台 worker 和 PM2 管理
* • SQLite + Chroma 双数据库
* • 实时 Web UI

所有这些，最终的目标只有一个：**让你和 Claude 的协作更流畅**。

不需要每次都重新介绍项目。

不需要担心 context limit。

不需要手动保存笔记。

就像和一个真正的同事一起工作一样。

如果你也在用 Claude Code 做真实项目，如果你也被"goldfish memory"折磨过，不妨试试 claude-mem。

它可能会让你重新思考：**AI 协作的未来应该是什么样的**。

---

### 参考资料

* • **GitHub**: thedotmack/claude-mem
* • **完整文档**: docs.claude-mem.ai
* • **安装指南**: Installation Guide
* • **架构概览**: Architecture Overview
* • **Beta 功能**: Beta Features
* • **许可证**: AGPL-3.0

P.S. 这个项目是开源的（AGPL-3.0），欢迎贡献代码、提 issue、或者给个 star。

 

---

坚持创作不易，求个一键三连，谢谢你～❤️

以及「AI Coding技术交流群」，联系 ayqywx 我拉你进群，共同交流学习～
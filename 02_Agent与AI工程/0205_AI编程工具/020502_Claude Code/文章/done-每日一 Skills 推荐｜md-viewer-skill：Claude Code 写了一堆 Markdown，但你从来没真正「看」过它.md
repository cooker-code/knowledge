> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: 每日一 Skills 推荐｜md-viewer-skill：Claude Code 写了一堆 Markdown，但你从来没真正「看」过它
author: 大模型AI之旅
date:
url: https://mp.weixin.qq.com/s?__biz=MzU2NTUzOTUyMg==&mid=2247484686&idx=1&sn=7c7789f5476a5ff936a8747ebdf0b179&chksm=fdf2c78b6f33912f299176bf95cfc86143af7328bee0f97463ed08657d94ad832d20dfba7af5&mpshare=1&scene=24&srcid=04195jvkgl77UhfMGmwyys9b&sharer_shareinfo=06bd743127ba9e2c1ce6d720a82a5eff&sharer_shareinfo_first=06bd743127ba9e2c1ce6d720a82a5eff#rd
---

# 每日一 Skills 推荐｜md-viewer-skill：Claude Code 写了一堆 Markdown，但你从来没真正「看」过它

Claude Code 跑在终端里。

这句话的潜台词是：它生成的所有东西，你都是以纯文本的形式在看。Markdown 标题就是一行带 `#` 的字符串，表格就是一堆 `|` 和 `-` 对齐的格子，Mermaid 图就是一坨你根本不会去人肉解析的代码块。

你让它写一份分析报告，它在终端里刷出来三屏内容，滚动速度快过你的阅读速度。你用鼠标往回滚，找到想看的段落，盯着等宽字体下密密麻麻的纯文本，试图在脑子里把 `**加粗**` 还原成加粗、把 `- 列表项` 还原成列表。

这个过程中信息是有折损的。Markdown 的设计初衷就是渲染后阅读，不是在终端里肉眼解析。

## 一个本地预览器

md-viewer-skill 做的事很简单：让 Claude Code 把 Markdown 文件扔进一个本地浏览器窗口里，实时渲染，自动刷新。

你对 Claude 说「把 report.md 显示出来」或者「预览一下这个文件」，它会在本地启动一个 Express 服务（端口 3456），把文件加载到浏览器中，渲染成你习惯的样子。标题有层级，表格有边框，代码块有高亮，Mermaid 图变成真正的图。

作者是 GitHub 用户 Mxcks。

安装之后，使用方式很自然：

```
1

2

3

4

5

6

7

# 直接用斜杠命令
/md-viewer report.md
 
# 或者用自然语言
> 把这个文件显示出来
> 预览一下 analysis.md
> 显示输出的文档
```

三个核心操作：

| 命令 | 效果 |
| --- | --- |
| `show <文件>` | 替换当前预览内容 |
| `add <文件>` | 追加到现有预览 |
| `clear` | 清空预览器 |

实际操作一下感受更直观。比如你让 Claude 审查一个项目，想看渲染后的结果：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

> 帮我分析一下 src/auth 目录的设计，输出成 Markdown，然后用 md-viewer 打开给我看
 
Claude:
  1. 分析 src/auth 代码结构...
  2. 生成报告到 ./auth-analysis.md
  3. 检查 md-viewer 服务是否运行...
     → 端口 3456 未占用，启动服务
     → cd server && node viewer-server.js &
  4. 调用 show 命令加载文件
     → node viewer-helper.js show "E:/project/auth-analysis.md"
     ✅ Viewer updated with 1 file(s)
  5. 打开浏览器
     → start "http://localhost:3456/Markdown_Viewer_Enhanced.html"

  报告已在浏览器中打开。包含 3 个 Mermaid 图（依赖图、
  调用链、权限流），已自动渲染。
```

浏览器弹出来就是一份带层级标题、可交互图表、可搜索的文档，跟你在 Typora 里看自己写的 Markdown 差不多。Claude 后续再追加内容，3 秒后浏览器自己刷新。你不用回终端做任何操作。

如果你想让 Claude 边写边让你看进度：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

> 把 auth、api、cache 三个模块分别分析，结果追加到预览器里
 
Claude:
  1. 分析 auth → 生成 auth.md
     → add "E:/project/auth.md"
     ✅ Added 1 file(s)
  2. 分析 api → 生成 api.md
     → add "E:/project/api.md"
     ✅ Added 1 file(s)
  3. 分析 cache → 生成 cache.md
     → add "E:/project/cache.md"
     ✅ Added 1 file(s)

  三份报告已全部追加到预览器。右上角可切换全屏，
  侧边栏可以在三个文件间跳转。
```

这个用法有点像流式写作的看板：Claude 在终端里一个一个生成文件，浏览器里就一份一份出现。全部完成前你已经可以看前面的内容了。

show 替换 vs add 追加

## 架构特别简单

这个 Skill 的技术实现值得说一下，因为它的简单程度几乎是反直觉的。

没有 WebSocket。没有复杂的进程通信。整个架构就三个部分：

**一个 Express 服务**，跑在 `localhost:3456`，50 行代码。提供两个 API：`/api/manifest` 返回当前要显示哪些文件，`/api/file?path=xxx` 返回文件内容。

**一个 manifest.json 文件**，记录当前应该显示哪些 Markdown 文件的路径列表。Claude 每次调 `show` 或 `add` 命令，实际上就是往这个 JSON 文件里写路径。

**一个 HTML 页面**，每 3 秒轮询一次 manifest，发现有新文件就拉内容、渲染、显示。

Claude 写文件路径到 JSON，浏览器轮询 JSON，拉文件内容，渲染。整个链路没有任何魔法，甚至没有 WebSocket。

选择轮询而不是 WebSocket 是个有意思的决定。WebSocket 能做到实时推送，体验更好。但轮询的好处是：不用管连接状态、不用处理断线重连、不用引入额外依赖。3 秒一次的延迟对文档预览来说完全够用，因为你不是在看直播弹幕，你是在看一份报告。

这种「能用最笨的方案解决就不用聪明方案」的思路，跟上一篇推荐的 karpathy-skills 里 Simplicity First 原则不谋而合。

md-viewer 架构流程图

## 实际用起来

假设你在让 Claude Code 分析一份代码库，它输出了一份 Markdown 格式的架构分析报告。

在终端里你看到的是这样的：

```
1

2

3

4

5

6

7

8

9

10

11

12

13

14

## 模块依赖关系
 
| 模块 | 依赖 | 被依赖 | 复杂度 |
|------|------|--------|--------|
| auth | config, db | api, admin | 高 |
| api | auth, db, cache | frontend | 中 |
| cache | config | api | 低 |
 
```mermaid
graph LR<br>
    config --> auth<br>
    db --> auth<br>
    auth --> api<br>
    ...
```

一屏放不下，Mermaid 代码块根本看不出关系图长什么样，表格在等宽字体下勉强能看。

有了 md-viewer，你说一句「把这个报告显示出来」，浏览器弹出一个深色主题窗口：

* 表格有清晰的边框和行距
* Mermaid 自动渲染成可视化的依赖图
* 标题有层级缩进，侧边栏可以快速跳转
* 支持全文搜索，直接定位到你关心的模块

报告还在更新？没关系，3 秒自动刷新，你在浏览器里看着内容变化，不用手动刷新。

终端 vs 浏览器渲染效果

## 预览器本身的细节

这个 HTML 页面写了 2400 多行，在「预览器」这个定位下算是用心了。

深色主题，用的是 `#0d1117` 为底色（跟 GitHub 暗色主题一样），sage green（鼠尾草绿）作为强调色。视觉上不刺眼，长时间看报告不累。

几个功能点：

**自动滚动**：默认开启。Claude 在持续输出内容的时候，预览器会自动滚到最新位置，你不用一直手动翻。

**全屏模式**：右上角一个按钮切换。写报告的时候开全屏，不受其他窗口干扰。

**Mermaid 支持**：这是个大加分。Claude 经常用 Mermaid 画流程图、类图、时序图。在终端里这些就是一段文字。在预览器里它会被渲染成真正的图形。

**拖拽加载**：直接把 `.md` 文件拖进浏览器窗口，就能添加到预览里。不用回到终端操作。

**全文搜索**：跨所有已加载的文件搜索。报告太长找不到某个关键词？搜一下就行。

## 什么时候用得上

不是所有场景都需要预览器。如果 Claude 只是改了两行代码、回了一句话，开预览器纯属多余。

但下面这些场景就不一样了：

**Claude 输出长报告的时候。** 代码审查报告、架构分析、数据分析结论，终端里三屏滚完就找不到了。浏览器里能慢慢看、能搜索、能跳转。

**文档包含 Mermaid 图的时候。** 流程图、架构图、时序图，不渲染出来就是一堆字符。

**需要反复参考的文档。** Claude 生成了一份 API 文档或者使用说明，后续开发中要反复查看。放在预览器里保持打开，比在终端历史里翻来翻去强。

**要给别人看的时候。** 在浏览器里直接打印或截图，比贴一段纯文本体面得多。

## 判断

md-viewer-skill 解决的问题很小：终端里的 Markdown 看着不舒服。

但这个「小问题」背后有个更大的洞察：Claude Code 的输出界面太原始了。它生成的内容质量越来越高，能写报告、能画图、能排版，但展示方式还停留在等宽字体的纯文本上。你有一台高清打印机，永远在用传真模式。

md-viewer 的做法是在 Claude Code 旁边开一个「显示窗口」，让 AI 的输出有一个配得上内容质量的呈现方式。而且技术手段极其克制：一个本地服务器、一个 JSON 文件、一个 HTML 页面。没有云服务，没有账号，所有数据在本机，启动一次之后就不用管了。

局限也有。它只能预览 Markdown，不支持 HTML 或其他格式。没有导出 PDF 功能。目前 Star 数不多，社区还小，功能迭代全靠作者一个人。但作为一个「给 Claude Code 开一扇窗」的工具，已经够用了。

GitHub 仓库：https://github.com/Mxcks/md-viewer-skill

你有没有在终端里被大段 Markdown 输出搞晕过？或者你有什么更好的查看方式？评论区聊聊。

以上，既然看到这里了，

如果觉得不错，随手点个赞、在看、转发三连吧，

如果想第一时间收到推送，也可以给我个星标⭐～

谢谢你看我的文章

后台回复skill，也可以获取全套skill～

我创建了一个skill分享交流群，有兴趣的可以加入
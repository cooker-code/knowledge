---
title: 一个神奇的视频生成 Skills，实测，狂喜
author: Ai学习的老章
date: 老章很忙老章很忙
url: https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649013042&idx=1&sn=694960fbfd2ba2b45fa8b2b9a8e92b62&chksm=8625b2367394e07327e19896a391ad67df21d9bcf5cb8caa7bc57f9136dc1e7000fc1924715e&mpshare=1&scene=24&srcid=04277rglmiUNSG2McGd2mr4x&sharer_shareinfo=0083049a949314e9c915dc9747296e49&sharer_shareinfo_first=0083049a949314e9c915dc9747296e49#rd
---

这个 skills 我愿称之为老章狂喜Skills

我的视频号，都是文字转视频，用的基于Remotion自己手写的skills，目前我还是比较满意的

今天再聊一个更神奇的 Skills——HyperFrames，让 AI agent 用 HTML 代码直接写出视频

一句话定位是：**Write HTML. Render video. Built for agents.**

还是先看效果吧，我把前几天发的那篇 [Claude Opus 蒸馏 Qwen3.6-27B，GGUF 来了，消费级显卡轻松本地部署！](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649012980&idx=1&sn=36de8d3e45d1767f69d49c226fff64d9&scene=21#wechat_redirect) 喂给了 HyperFrames，让它压成一段可视化解读视频

结果如下：

然后我花了一些时间，把它融入到语音克隆、读稿、字幕处理、视频生产、BGM压制大工作流中

生成了我的声音播报、带精准字幕和BGM的视频

### 简介

HyperFrames 是一个开源的视频渲染框架，把视频组合（composition）写成一个 HTML 文件，浏览器里实时预览，命令行渲染成 MP4

最有意思的是 Skills 这条线

装上 HyperFrames 的 skills，Claude Code、Cursor、Gemini CLI、Codex 这些 agent 就学会怎么写合规的 composition 和 GSAP 动画

在 Claude Code 里，这套 skills 直接注册成斜杠命令：`/hyperframes` 写 composition，`/hyperframes-cli` 跑 CLI，`/gsap` 解决动画问题

技术栈是 headless Chrome + FFmpeg 的稳定组合，HTML 文件直接当源文件用，没有构建步骤

HyperFrames demo — HTML 代码在左，渲染出的视频在右

**核心特点：**

* **HTML 原生** — composition 就是带 `data-*` 属性的 HTML 文件，没有 React，没有专有 DSL，agent 本来就会写 HTML
* **Skills 驱动** — 一行 `npx skills add heygen-com/hyperframes` 把视频框架的肌肉记忆装进 agent 脑子里
* **确定性渲染** — 同样输入 = 同样输出，自动化流水线友好
* **Frame Adapter 模式** — 动画运行时可以换，GSAP、Lottie、CSS、Three.js 都能接
* **50+ 现成模块** — 社交平台覆盖层、shader 转场、数据图表、电影感特效，一行命令安装

### 安装

要求 Node.js >= 22，加 FFmpeg。

**配合 AI agent（官方推荐）：**

```
npx skills add heygen-com/hyperframes
```

装完之后直接在 agent 里描述需求即可

**手动起项目：**

```
npx hyperframes init my-video  
cd my-video  
npx hyperframes preview      # 浏览器实时预览，热更新  
npx hyperframes render       # 渲染成 MP4
```

`hyperframes init` 会自动把 skills 一并安装好，随时可以切回 agent 模式

Codex 用户有专门的 plugin 入口，稀疏安装：

```
codex plugin marketplace add heygen-com/hyperframes --sparse .codex-plugin --sparse skills --sparse assets
```

Cursor 也有对应 plugin，可以从 Cursor Marketplace 装，也可以本地 sideload

### 使用

一个 composition 长这样，就是一个 HTML 片段：

```
<div id="stage" data-composition-id="my-video" data-start="0" data-width="1920" data-height="1080">  
  <video  
    id="clip-1"  
    data-start="0"  
    data-duration="5"  
    data-track-index="0"  
    src="intro.mp4"  
    muted  
    playsinline  
  ></video>  
  <img  
    id="overlay"  
    class="clip"  
    data-start="2"  
    data-duration="3"  
    data-track-index="1"  
    src="logo.png"  
  />  
  <audio  
    id="bg-music"  
    data-start="0"  
    data-duration="9"  
    data-track-index="2"  
    data-volume="0.5"  
    src="music.wav"  
  ></audio>  
</div>
```

`data-start` 是开始时间，`data-duration` 是持续秒数，`data-track-index` 是轨道编号——视频、图片、音频在时间轴上的关系一目了然

这就是它能让 agent 准确生成视频的关键：HTML 大模型再熟不过了

调用 catalog 加现成组件：

```
npx hyperframes add flash-through-white   # shader 转场  
npx hyperframes add instagram-follow      # Ins 关注覆盖层  
npx hyperframes add data-chart            # 动态数据图表
```

跟 agent 配合的几个典型 prompt（直接抄）：

> ❝
>
> Using `/hyperframes`, create a 10-second product intro with a fade-in title, a background video, and background music.

> ❝
>
> Take a look at this GitHub repo https://github.com/heygen-com/hyperframes and explain its uses and architecture to me using `/hyperframes`.

> ❝
>
> Make a 9:16 TikTok-style hook video about [topic] using `/hyperframes`, with bouncy captions synced to a TTS narration.

> ❝
>
> Make the title 2x bigger, swap to dark mode, and add a fade-out at the end.

最后那条改稿 prompt 才是真正的杀器——把 agent 当视频剪辑师用，自然语言改样式、加下三分之一、加片尾淡出，全程不用碰代码

### HyperFrames 与 Remotion

视频渲染框架这块，绕不开 Remotion

HeyGen 自己也大方承认 HyperFrames 受 Remotion 启发，源码里还保留了向 Remotion 致敬的注释（Chrome 启动参数、image2pipe → FFmpeg 流式管道、帧缓冲那些模式）

两者底层都是 headless Chrome 驱动，都讲究确定性渲染

差别在一个核心决定上：**作者主要写什么**

Remotion 押注 React 组件，HyperFrames 押注 HTML

|  | **HyperFrames** | **Remotion** |
| --- | --- | --- |
| 作者写法 | HTML + CSS + GSAP | React 组件（TSX） |
| 构建步骤 | 无，`index.html` 直接跑 | 必须，要打包 |
| 库时钟动画（GSAP、Anime.js、Motion One） | 可 seek，帧级精准 | 渲染时按墙钟播放 |
| 任意 HTML / CSS 直通 | 粘贴即可动画 | 要重写成 JSX |
| 分布式渲染 | 目前单机 | Lambda，生产可用 |

许可证差别更直接：**HyperFrames 是 Apache 2.0 完全开源**，OSI 认证那种，商用任意规模、无按渲染计费、无座位上限、无公司体量阈值

**Remotion 是 source-available**，代码在 GitHub 上但用的是自定义 Remotion License，超过小团队规模需要付费授权

如果是给 agent 用，HTML 这条路比 React 那条路顺得多——大模型生成 HTML 的准确率远高于生成完整 React 项目，加上 HyperFrames 没有构建步骤，agent 写完文件就能预览，反馈链路特别短

### 生态包

仓库是个 monorepo，按职责拆得很清晰：

| Package | 作用 |
| --- | --- |
| `hyperframes` | CLI，create / preview / lint / render |
| `@hyperframes/core` | 类型、解析器、生成器、linter、运行时、frame adapter |
| `@hyperframes/engine` | 可 seek 的页面到视频捕获引擎（Puppeteer + FFmpeg） |
| `@hyperframes/producer` | 完整渲染流水线（捕获 + 编码 + 音频混合） |
| `@hyperframes/studio` | 浏览器端 composition 编辑器 UI |
| `@hyperframes/player` | 嵌入式 `<hyperframes-player>` web component |
| `@hyperframes/shader-transitions` | composition 用的 WebGL shader 转场 |

随框架一起出的 skills 一共 5 个：

| Skill | 教 agent 什么 |
| --- | --- |
| `hyperframes` | HTML composition 编写、字幕、TTS、音频反应动画、转场 |
| `hyperframes-cli` | CLI 命令：init、lint、preview、render、transcribe、tts、doctor |
| `hyperframes-registry` | 通过 `hyperframes add` 安装区块和组件 |
| `website-to-hyperframes` | 抓取一个 URL 把它变成视频——完整的 website-to-video 流水线 |
| `gsap` | GSAP 动画 API、时间轴、缓动、ScrollTrigger、插件、React/Vue/Svelte、性能 |

`website-to-hyperframes` 这个最骚——把网页直接变视频，这是把"内容素材搬运"的活儿都打包好了。

### 实战：把一篇公众号文章做成 22 秒解读视频

上周发的那篇 [Claude Opus 蒸馏 Qwen3.6-27B，GGUF 来了，消费级显卡轻松本地部署！](https://mp.weixin.qq.com/s?__biz=MzA4MjYwMTc5Nw==&mid=2649012980&idx=1&sn=36de8d3e45d1767f69d49c226fff64d9&scene=21#wechat_redirect) 喂给了 HyperFrames，让它压成一段可视化解读视频

全程 HTML + GSAP 写五个场景：标题 → Qwen + Opus 公式 → 关键参数卡片（12K SFT / 16 prompt 评测 / RTX 5090 / Apache 2.0）→ GGUF 量化档位列表 → 收尾

`npx hyperframes init` 起项目，`npx hyperframes lint` 校验，`npx hyperframes render` 渲染——总共 22 秒的 1920×1080 视频，渲染耗时 25 秒，输出 2 MB 的 MP4：

Qwopus3.6 文章解读 · HyperFrames 一把梭生成

又生成了一个适合视频号的3:4

整个 composition 的核心结构就是一个 HTML 文件，每个场景一个 `.scene` 块加 `data-start`、`data-duration` 控制时间轴，底下一段 GSAP timeline 控动画。改文案、调时长、换配色都是改 HTML，agent 改起来跟改普通网页没区别

中间有个小坑：渲染时碰到 `PingFang SC` 不在 HyperFrames 的确定性字体映射表里，有 warning，但兜底会走 Inter，不影响出片。要彻底干净的话，文档里说自己加 `@font-face` 引入字体文件就行

### 总结

HyperFrames 这套东西的精妙之处，在于它把**视频生成**这个传统上需要剪辑师/动画师的工种，重新定义成了**写 HTML**——而 agent 写 HTML 的能力本来就到位了

Skills 把框架的具体语法、动画模式、CLI 用法封装成 agent 可加载的上下文，等于给大模型现场培训了一个全栈视频开发能力

适合谁用：

* 做内容流水线的，想批量生成宣传短视频、产品演示、社交平台内容
* 已经在用 Claude Code / Cursor / Codex，希望把视频环节也接进 agent 工作流
* 需要确定性、可复现的渲染，做自动化测试、脚本化产出
* 不想被 Remotion 商业许可绑住，要纯开源协议的

局限也得说清楚：

* 分布式渲染目前还没有，Remotion 已经有 Lambda 方案
* HTML + GSAP 这条路对纯前端不熟的人有上手门槛
* 复杂三维特效还是要靠 Three.js 或外部工具，框架本身只是给一个 frame adapter 接口

最大的看点还是 Skills 这条线，也是本文测试等重点

把一个开源视频框架的全部知识塞进 agent，让"写视频"和"写文章"在 Claude Code 里变成同一种操作——这是让我觉得"AI 原生工具"这个词终于落到实处的一个例子

#HyperFrames #Skills #HeyGen #视频生成 #ClaudeCode

**制作不易，如果这篇文章觉得对你有用，可否点个关注。给我个三连击：点赞、转发和在看。若可以再给我加个🌟，谢谢你看我的文章，我们下篇再见！**
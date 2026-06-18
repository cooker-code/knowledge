---
title: Ironpress：纯 Rust 手搓 CSS 布局引擎，HTML 转 PDF 干进微秒级
author: 知识小零食
date: 知识小零食知识小零食
url: https://mp.weixin.qq.com/s?__biz=MzYyMjk5MTIzNg==&mid=2247484621&idx=1&sn=35f70f6419df2d617450450eaa403d2d&chksm=fe6526581a8fb3140adf0bfb362fbf4f584e4dd8c22268eda765ba409a957a0ef651cb23dcf1&mpshare=1&scene=24&srcid=06041hhGV9afWGnqEpoPzeTC&sharer_shareinfo=d5f59a0f7fe727ba908861f17599e216&sharer_shareinfo_first=d5f59a0f7fe727ba908861f17599e216#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090210_文档生成与排版/090210_核心知识点/文档生成与排版工具边界|文档生成与排版工具边界]]


我就是看不下去一大堆 Rust 的 PDF 方案非要套一个无头 Chrome 才搞得定 HTML 这事儿。浏览器抢了整个现代桌面软件层也就算了，现在连 wkhtmltopdf 这种包了十年前的 WebKit 的玩意儿都敢出来跑马圈地。这简直是把一个终端的活儿外包给了一个桌面渲染引擎再包一个终端接口——全是适配层，没有半点直击本质的酣畅感。直到撞上 Ironpress，我才觉得在 PDF 生成这个赛道上，Rust 第一次真正站直了没靠别人。

这东西的底牌就是“纯 Rust 硬写了一个 CSS 布局引擎”，不调用外部浏览器、不挂系统动态链接库、不用 JavaScript 引擎，只是把 HTML 和一个内置的布局引擎扔进 cargo build 出来的二进制里。他们甚至还在网页里放了个基于 WASM 的在线试玩，你直接在浏览器里试效果，但那个 Demo 本身跑的就是 Ironpress 编译到 WASM 的内核——一点不带糊弄人的。整个项目没有系统依赖，没有 Node 那摊东西，没有 Chromium，就是 Rust 源文件和它自己造的一整套排版引擎、CSS 解析器、字体子集化器、数学公式渲染器。

你先别急着喷“它还能比比 Chrome ？”，我们把硬数据摆出来：同样是转换一个简单的 <p>Hello</p>，Ironpress 耗时 16 微秒，每秒处理 62,500 页；换成一个带 CSS 样式、列表和链接的稍微像样的文档，耗时 71 微秒，每秒处理 14,000 页；Markdown 文档（带标题、代码块、列表）耗时 141 微秒，每秒 7,000 页；即便是有表头样式的五列表格，也不过 341 微秒，每秒还能出 2,900 页；完整报告（表格、flex 布局、进度条）耗时 587 微秒，每秒 1,700 页。对比一下，无头 Chrome 渲染一页通常要 2,500 毫秒——Ironpress 比它快四千倍。你没看错，四千倍。这不是优化，这是把整个开销模型从“启动一个浏览器进程”直接拉到“在一个预热的栈里走一次布局递归”。

那要怎么用这玩意儿？先给一条 CLI 让你三秒钟爽起来。用 cargo install ironpress 把二进制拽下来，然后：

```
ironpress input.html output.pdf
```

它连 --page-size、--landscape、--margin 都给你准备好了，甚至还支持 --header 和 --footer，页眉页脚里可以用 {page} 和 {pages} 占位符自动替换，这完全是奔着命令行报告生成那一套实打实的业务场景去的。如果你连文件都不想落盘，可以直接从 stdin 管道进去，echo '<h1>Hello</h1>' | ironpress --stdin output.pdf，管道一到，PDF 出来，中间没有临时文件、没有外部进程调用，干净得不像话。

CLI 用熟了，你早晚要把它揉进 Rust 项目里。Ironpress 的 API 设计很短小精悍，没有那种 Java 味儿冲天的抽象工厂加管道组合。要生成一份 PDF 的最短路径长这样：

```
use ironpress::html_to_pdf;
let pdf = html_to_pdf("<h1>Hello</h1><p>World</p>").unwrap();std::fs::write("output.pdf", pdf).unwrap();
```

同样，Markdown 也能直接上：

```
let pdf = ironpress::markdown_to_pdf("# Hello\n\nWorld").unwrap();
```

到了这一步，你已经可以在任何 API 服务里把用户提交的富文本直接塞进这个函数，然后在 71 微秒之内给 HTTP 响应里挂上 application/pdf。但是真正想发挥它全部功力，必须上 HtmlConverter 的 Builder API。这才是整个 crate 的完全形态：

```
use ironpress::{HtmlConverter, PageSize, Margin};
let pdf = HtmlConverter::new()    .page_size(PageSize::LETTER)    .margin(Margin::uniform(54.0))    .header("My Document")    .footer("Page {page} of {pages}")    .convert("<h1>Custom page</h1>")    .unwrap();
```

链式调用里依次设置纸张尺寸（支持 A4、LETTER、LEGAL、LANDSCAPE 甚至自定义尺寸如 210mm 297mm）、页边距（可以四边分别指定）、页眉页脚，然后把 HTML 喂进去，一口输出 PDF 字节数组。这种链式构造对于后面我们要聊到的字体注入、数学公式渲染以及整个布局引擎的深度定制来说，是必经的入口。

很多人一看到“内置 CSS 布局引擎”，第一反应是“它肯定只是支持几个基础选择器和 margin 之类吧”。我一开始也这么想，直到我翻开它的 CSS 支持列表，那种“一个 Rust 项目默默干翻半个浏览器引擎”的感觉直接冲上脑门。Ironpress 对 CSS 的支持不是那种挑几个属性意思一下的程度——Flexbox、Grid、多列布局（multi-column），calc() 表达式，CSS 变量，@media 查询，@page 规则，@font-face 字体声明，全部原生实现。这就意味着你能用写 Web 前端的方式去排版一份 PDF，用 display: flex; justify-content: space-between 来实现两端对齐，用 grid-template-columns: 1fr 2fr 来实现栅格化布局，甚至用 @page { size: A4; margin: 2cm } 直接在 CSS 里控制整页尺寸——所有浏览器引擎该有的布局能力，它在一个 PDF 生成工具里重写了一遍，而且不带任何 C 依赖。

布局引擎底下的字体系统同样值得单开一段。Ironpress 内建了 PDF 标准的 14 种 Base 字体（Helvetica、Times、Courier 那些），开箱即可渲染英文和西欧文字。如果你需要自定义字体——比如中文字体或者带有特殊符号的企业专属字体——它支持 TTF 文件嵌入，并且会自动做字体子集化（font subsetting），只把文档里真正用到的字符编码塞进 PDF，不浪费体积也不增加渲染负担。Unicode 和 CJK 回退链也做了系统字体发现，Linux 上会自动找 Noto 系列，macOS 上会走 CoreText，Windows 上也能顺着系统字体目录摸到微软雅黑之类的默认中文字体。不需要你在配置文件里指定字体路径，只需把带有中文的 HTML 扔进去，PDF 里就能正常显示。

真正让我决定单独为它写一篇的，是它的数学公式渲染模块。Ironpress 直接把 LaTeX 数学模式吃进去了——$...$ 做行内公式，$$...$$ 做块级公式，支持分数、根号、矩阵、希腊字母和常用运算符。背后是一套独立的数学引擎，在解析 LaTeX 语法后将每个符号映射到对应的 Unicode 数学字符和 PDF 内部的位置计算，生成的是矢量级排版而非图片嵌入。你完全可以把一篇带有大量数学公式的 Markdown 笔记直接用 markdown\_to\_pdf() 丢进去，连着代码块和表格一起出成 PDF，不需要在旁边开着 LaTeX 环境编译半个世纪。光是这一个能力，Ironpress 就够格进入科研写作和自动化报告生成的严肃工具箱。

SVG 和图片支持同样没有拖后腿。JPEG 和 PNG 可以直接嵌入，支持 data: URI 格式的内联图片，甚至可以通过开启 remote feature 来直接拉取远程 URL 对应的图像资源。SVG 则走了原生矢量渲染路径——path、基本形状、渐变、transform 变换、clip paths 和 viewBox 缩放全部被 Ironpress 自己的布局引擎接管，出来的结果是 PDF 内部的原生矢量图形，和文字一样可以无损缩放。这意味着你可以用 SVG 做图表，用 HTML + CSS 做布局，然后把整份东西一次性「打印」成 PDF，全程没有任何栅格化步骤——这是无头浏览器永远做不到的事情。

看到这里你大概已经意识到，Ironpress 的本质不是一个“HTML 转 PDF 工具”，而是一个完整的文档渲染平台，只不过它的输入格式恰好是 HTML 和 Markdown 而已。它输出的 PDF 是 1.4 版本，支持书签（bookmarks）自动生成、链接注解（link annotations）、页眉页脚、渐变色和流式输出（streaming output）。对于大文档，流式输出意味着你不需要在内存里把整个 PDF 建完再一次性写到磁盘，而是可以在生成第一页渲染数据的时候就开始往文件里写，内存占用不会随文档页数线性增长——这个特性对生产环境里的报表服务来说，重要性远高于那几个微秒级的性能数字。

PDF 赛道在 Rust 生态里其实有好几个玩家，但 Ironpress 的定位是唯一一个同时吃掉了“CSS 布局”和“PDF 生成”两个模块的项目。genpdf 是一个不错的纯 Rust PDF 生成器，但它面向的是程序式 API，你得手动设置每行文字的坐标，不解析 HTML；printpdf 更底层，给你画布级的控制，但完全不碰布局；oxidize-pdf 也是纯 Rust 方案，生成速度虽然比 PDFSharp 快两倍，但它同样不包含 HTML/CSS 解析。至于 fullbleed，虽然同样是 Rust 核、Python 壳的 HTML 转 PDF 工具，但 Ironpress 在数学引擎、字体子集化、SVG 原生渲染和 streaming output 这些点上显然多走了好几步。

怎么开始？如果你只是想快速给你的项目加一个报告导出功能，一行 cargo add ironpress，然后套那个 HtmlConverter 的 Builder 链，传进你的数据库查出来的 HTML 模板，几微秒之后 PDF 字节数组就已经在手上了。如果你的业务涉及高并发报表服务（比如每天要生成十万份对账单），把 Ironpress 嵌进 Actix-Web 或 Axum 的 handler 里，没有任何外部进程 fork 的开销，没有浏览器池子要管理，没有系统字体配置的跨环境一致性问题，内存占用只是一个栈上分配的布局树的大小。因为不依赖任何 C 库，编译出来的二进制可以在 scratch 容器里直接跑，Rust 编译器的平台交叉编译能力让你从 macOS 上 cargo build --target x86\_64-unknown-linux-musl，就能产出一个能在任何 Linux 容器里跑起来的静态二进制。

对于那些用 Ruby 的人，Ironpress 甚至还提供了 Ruby gem 版本——通过 Rust 的原生扩展直接把内核编译进 gem，Rails 的 wicked\_pdf 那一路方案如果让你觉得慢得想摔键盘，不妨拉过去看一眼。Python 用户则可以通过 PyO3 或者直接调 CLI 的 stdin/stdout 管道集成，性能损失几乎为零。

Ironpress 是不是完美的 PDF 生成器？不是。但是它让你在 RUST 里做文档排版的摩擦力降到了一个前所未有的低点，这本身就已经值回你读完这篇的功夫了。

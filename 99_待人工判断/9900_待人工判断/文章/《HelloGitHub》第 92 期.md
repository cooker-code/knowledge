---
title: 《HelloGitHub》第 92 期
author: HelloGitHub
date: 
url: http://mp.weixin.qq.com/s?__biz=MzA5MzYyNzQ0MQ==&mid=2247517137&idx=1&sn=e9774057bd8258e997a5f14d96e120e8&chksm=9058368fa72fbf998e539e88bdd9dafc3a0b783d1634464aaee8db6bc0b81cc4387fab5610e8&mpshare=1&scene=24&srcid=1128fcrLQRqTsEdrIsfTufqq&sharer_shareinfo=500afb86c20b0b52d92a316a803dd2c1&sharer_shareinfo_first=500afb86c20b0b52d92a316a803dd2c1#rd
---

> 兴趣是最好的老师，**HelloGitHub** 让你对编程感兴趣！

## 简介

**HelloGitHub** 分享 GitHub 上有趣、入门级的开源项目。

> https://github.com/521xueweihan/HelloGitHub

这里有实战项目、入门教程、黑科技、开源书籍、大厂开源项目等，涵盖多种编程语言 Python、Java、Go、C/C++、Swift...让你在短时间内感受到开源的魅力，对编程产生兴趣！

---

> 以下为本期内容｜每个月 **28** 号更新

### C 项目

1、activate-linux：将"Activate Windows"水印移植到 Linux 系统。这是一个可以在桌面系统的右下角，显示“激活 Windows” 字样的小工具，就是图一乐。

> 地址：https://github.com/MrGlockenspiel/activate-linux

2、kew：一款 C 语言写的命令行音乐播放器。适用于 Linux 系统的命令行音乐播放器，支持搜索音乐、播放列表、专辑封面等功能。

> 地址：https://github.com/ravachol/kew

### C# 项目

3、EGamePlay：一款基于 Unity 引擎的灵活战斗框架。这是一个灵活、通用、轻量的游戏战斗/技能框架，配置可选择 ScriptableObject 或 Excel 表格。内含 RPG、回合制、技能调试等示例，以及《如何实现一个战斗系统》的教程。

> 地址：https://github.com/m969/EGamePlay

4、FreeControl：在 Windows 电脑上控制 Android 设备的工具。该项目是基于 scrcpy、采用 C# 编写的控制 Android 设备的 PC 桌面工具，提供了更加简洁的交互界面。来自 @Pdone 的分享

> 地址：https://github.com/pdone/FreeControl

5、N\_m3u8DL-RE：适用于 MPD/M3U8/ISM 的流媒体下载器。该项目可以将常见的流媒体保存到本地，支持点播、录制直播、自动混流等功能，适用于 Windows、Linux、macOS 操作系统。

> 地址：https://github.com/nilaoda/N\_m3u8DL-RE

6、Squirrel-RIFE：中文自动补帧工具。该项目是基于 RIFE 算法的补帧软件，可用于去除动漫卡顿感。具有无需手动设置、高质量输出、速度快等特点，适用于 Windows 10 及以上操作系统。

> 地址：https://github.com/Justin62628/Squirrel-RIFE

### C++ 项目

7、olcNES：用 C++ 写一个 NES 模拟器。这是一份教你用 C++ 写 NES/FC 游戏模拟器的视频教程和源码，作者是油管大神 javidx9。

> 地址：https://github.com/OneLoneCoder/olcNES

8、olive：一款免费、开源的非线性视频剪辑工具。非线性视频剪辑是指将图片、音乐、特效等素材与视频进行混合编辑，虽然该项目完全免费，但目前还处于迭代中并不稳定，适用于 Windows、macOS 和 Linux 系统。

> 地址：https://github.com/olive-editor/olive

### Go 项目

9、algernon：小型、独立的 Go Web 服务器。该项目是用 Go 编写的“快餐” Web 服务器，采用 BoltDB、Redis、MySQL 或 PostgreSQL 作为数据库，内置 Lua 解释器。所有功能全在一个独立可执行文件中，支持 Markdown 渲染、Lua 脚本、请求限制、用户和权限等。

> 地址：https://github.com/xyproto/algernon

10、cheat：一款交互式的“小抄”命令行工具。该项目可以创建、编辑、查看 \*nix 系统命令的备忘录，比如常用命令的示例和解释。

```
cheat tar  
  
# To extract an uncompressed archive:  
tar -xvf '/path/to/foo.tar'  
  
# To extract a .gz archive:  
tar -xzvf '/path/to/foo.tgz'  
  
# To create a .gz archive:  
tar -czvf '/path/to/foo.tgz' '/path/to/foo/'  
  
# To extract a .bz2 archive:  
tar -xjvf '/path/to/foo.tgz'  
  
# To create a .bz2 archive:  
tar -cjvf '/path/to/foo.tgz' '/path/to/foo/'
```

> 地址：https://github.com/cheat/cheat

11、devbox：为应用程序创建隔离环境的命令行工具。该项目可以创建一个可移植、隔离、用于开发的独立 shell，无需 Docker 和虚拟机。比如你的项目使用 Python 和 Go 语言，用这个工具仅需一条命令就能初始化一个独立的开发环境。

```
# 安装  
curl -fsSL https://get.jetpack.io/devbox | bash  
# 初始化  
devbox init  
# 安装 Python 和 Go  
devbox add python2 go_1_18  
# 激活  
devbox shell
```

> 地址：https://github.com/jetpack-io/devbox

12、faas：一款高星的功能即服务框架。该项目用容器的方式运行 Serverless 函数，让功能即服务（FaaS）变得简单。它可以轻松地将函数和微服务部署到 Kubernetes，支持自动扩缩容、自带 Web 管理平台、Dockerfile 和多种编程语言。

> 地址：https://github.com/openfaas/faas

13、migrate：好用的数据库迁移/变更工具。该项目是用 Go 写的数据库迁移(migrate)工具，帮你自动创建 SQL 迁移文件并管理版本，支持 MySQL、MariaDB、PostgreSQL、SQLite、Neo4j、ClickHouse 等不同类型的数据库。

```
$ migrate -source file://path/to/migrations -database postgres://localhost:5432/database up 2
```

> 地址：https://github.com/golang-migrate/migrate

### Java 项目

14、graceful-response：SpringBoot 接口优雅响应处理器。该项目通过注解的方式，优化 Controller 层的代码，完成统一返回值封装、全局异常处理、异常与错误码映射等功能。

```
public class Controller {  
      
    @GetMapping("/query")  
    @ResponseBody  
    public Data query(Parameter params) {  
            Data data = service.query(params);  
           return data;  
    }  
}
```

> 地址：https://github.com/feiniaojin/graceful-response

### JavaScript 项目

15、Cronicle：一个简单的任务调度和运行平台。该项目是用 Node.js 写的 cron 替代品，它开箱即用、自带 Web 界面、无需数据库，提供了执行 shell 命令、实时统计、自动故障转移、自动重试、多时区等功能。

> 地址：https://github.com/jhuckaby/Cronicle

16、earth：一个可视化全球天气实况的项目。该项目以可视化的方式展示了全球的天气情况，提供了风、温度、相对湿度等多种天气数据，以及风、洋流和波浪的动画效果。

> 地址：https://github.com/cambecc/earth

17、javascript-testing-best-practices：JavaScript 和 Node.js 的测试最佳实践。这是一份提升 JavaScript & Node.js 项目稳定性的指南，包括前/后端测试、持续集成、工具等方面。

> 地址：https://github.com/goldbergyoni/javascript-testing-best-practices

18、MikuTools：一个轻量级的在线工具集合。该项目是用 Vue + Nuxt.js 构建的在线工具箱，开源版本仅保留了部分无需后端的功能。

> 地址：https://github.com/Ice-Hazymoon/MikuTools

19、page-spy-web：像使用谷歌控制台一样开始远程调试。这是一款用来调试远程 Web 项目的工具，提供了 Docker、NPM 等多种部署方案。

> 地址：https://github.com/HuolalaTech/page-spy-web

### Kotlin 项目

20、ponymusic：开源的 Android 在线音乐播放器。该项目是用 Kotlin 语言写的 Android 音乐播放器，支持添加和播放本地音乐、通知栏控制、同步网易云歌单、每日推荐、搜索歌曲和歌单等功能。

> 地址：https://github.com/wangchenyan/ponymusic

### Python 项目

21、example-code-2e：《流畅的 Python（第 2 版）》的示例代码。《流畅的 Python》是深受 Python 程序员喜爱的经典之作，该书可以帮助理解 Python 语言的核心特性和底层逻辑。但这里只有示例代码，书需要自行购买。

> 地址：https://github.com/fluentpython/example-code-2e

22、LaTeX-OCR：将数学公式转化成 LaTeX 代码。该项目可以将图片、剪贴板中的图片和屏幕截图，转化成对应的 LaTeX 代码，提供了命令行、库、GUI、Docker 多种使用方式。

```
from PIL import Image  
from pix2tex.cli import LatexOCR  
  
img = Image.open('path/to/image.png')  
model = LatexOCR()  
print(model(img))
```

> 地址：https://github.com/lukas-blecher/LaTeX-OCR

23、Rickrack：一款开源的调色板桌面应用。该项目是基于 PyQt5 的调色板应用程序，旨在帮助用户轻松实现色彩的协调与搭配。它免费、无需注册、没有任何限制，支持离线使用、提取颜色、调色等功能。开箱即用无论你是绘画爱好者还是专业用户，都可以轻松上手并发挥创意。

> 地址：https://github.com/eigenmiao/Rickrack

24、sqlmap：强大的 SQL 注入工具。这是一个 Python 写的渗透测试工具，可以自动检测和利用 SQL 注入漏洞，获得数据库服务器的权限。它提供了强大的检测引擎和多种特性，包括识别数据库类型和版本、枚举用户、提权、获取数据等。

> 地址：https://github.com/sqlmapproject/sqlmap

25、XHS-Downloader：小红书图文/视频采集工具。该项目是基于 Python Requests 库实现的小红书作品采集器，支持获取图文/视频信息、下载完整作品、批量下载等功能，提供了 Windows 可执行文件和源码运行两种方式。

> 地址：https://github.com/JoeanAmier/XHS-Downloader

### Rust 项目

26、git-cliff：自由可定制的变更日志生成器。该项目可以自定义解析规则，自动从 Git 历史记录中生成 Changelog 文件。

> 地址：https://github.com/orhun/git-cliff

27、proc-macro-workshop：学习如何编写 Rust 过程宏。Rust 的过程宏（procedural macros）是一种高级用法，可以理解为生成 Rust 代码的 Rust 代码。该项目包含 5 个示例项目，其中 3 个是作者在工作中实现的宏。

> 地址：https://github.com/dtolnay/proc-macro-workshop

28、ruff：非常快的 Python 代码风格检查和格式化工具。该项目采用 Rust 编写，比 Python 的 Flake8 和 Black 快 10-100 倍，支持通过 pip 安装、内置 700+ 规则、兼容 Python 3.12、自动纠错等功能。

> 地址：https://github.com/astral-sh/ruff

### Swift 项目

29、secretive：一款存储和管理 SSH 密钥的应用。该项目是可以将 SSH 密钥存储在苹果芯片安全隔离区（Secure Enclave）的工具。安全隔离区是指集成到 Apple 片上系统 (SoC) 的专用安全子系统，它独立于主处理器，可提供额外的安全保护。

> 地址：https://github.com/maxgoedjen/secretive

### 其它

30、Awesome-Love-Code：表白代码收藏馆。该项目收集了 50+ 个用于表白的代码和程序，涵盖 Web、Python、C/C++、C# 等编程语言。

> 地址：https://github.com/sun0225SUN/Awesome-Love-Code

31、dpoint：一款开源数字手写笔。该项目通过摄像头跟踪和惯性测量，实现了 6DoF 输入。触控笔可用于任何平面，仅需消费级的摄像头配合使用。

> 地址：https://github.com/Jcparkyn/dpoint

32、linux-router：将 Linux 作为路由器的脚本。这是一个 Linux 软路由器的 shell 脚本，它可以通过一条命令将 Linux 设备作为路由器，提供互联网共享、DNS 服务器、WiFi 热点等功能。来自 @GunVeda 的分享

> 地址：https://github.com/garywill/linux-router

33、nerd-fonts：解决字体缺失问题的项目。这是一个收集了 3600+ 图标的字体集合和补丁工具，该项目不是一个字体，而是一个可以将多种字体中的图标，作为补丁添加到目标字体中的工具。

> 地址：https://github.com/ryanoasis/nerd-fonts

34、RehabilitationGuide：程序员颈椎病腰突康复指南。该项目是作者从确诊颈椎病、腰椎间盘突出到康复的经验和方法分享。来自 @九旬UKDhO 的分享

> 地址：https://github.com/AnsonZnl/RehabilitationGuide

35、smhasher：测试 Hash 函数质量和速度的项目。该项目展示了 200+ 种非加密哈希函数，在分布、冲突和性能等方面的测试结果。

> 地址：https://github.com/rurban/smhasher

### 开源书籍

36、typescript-book：《简明的 TypeScript 书》。该书全面、精练地介绍了 TypeScript 语言，涵盖了 TypeScript 语言的入门、类型系统、基础语法和高级用法等知识。

> 地址：https://github.com/gibbok/typescript-book

### 机器学习

37、cleanlab：自动检测数据集中错误数据和标注的框架。该项目基于置信学习（confident learning，CL）算法，实现了自动检测出机器学习数据集中的各种问题，提高数据集质量训练出更好的模型，支持图像、文本、音频类型的数据。

> 地址：https://github.com/cleanlab/cleanlab

38、ComfyUI：一个基于节点流程的 AI 绘图操作界面。该项目将 Stable Diffusion 流程分成多个节点，通过拖拽各种节点构成图像生成到处理的工作流，支持 Stable Diffusion 1.x 和 2.x 版本、组合各种模型、根据 PNG 图片生成完整的工作流等功能。

> 地址：https://github.com/comfyanonymous/ComfyUI

39、dvc：一款针对 AI 项目的数据版本管理工具。基于 Git 的数据版本管理工具，版本化机器学习项目的数据和模型。可用于比较代码、数据、参数、模型或性能图，共享机器学习项目的数据或重现结果。

> 地址：https://github.com/iterative/dvc

40、ml-engineering：机器学习：LLM/VLM 训练与工程。该项目是作者训练开源 BLOOM-176B 大模型和 IDEFICS-80B 多模态模型的经验总结，还提供了大量可以直接拿来用的代码和脚本，希望能够帮助你成功训练大型语言模型和多模态模型。

> 地址：https://github.com/stas00/ml-engineering

41、screenshot-to-code：该项目可以将屏幕截图转化为 HTML/Tailwind CSS 代码，它使用 GPT-4 Vision 生成代码、DALL-E 3 生成相似的图片。

> 地址：https://github.com/abi/screenshot-to-code

## 最后

**感谢参与****分享开源项目的小伙伴**，欢迎更多的开源爱好者来 HelloGitHub 自荐/推荐开源项目。

本期有你感兴趣的开源项目吗？如果有的话就留言告诉我吧～还没看过瘾？[点击阅读](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA5MzYyNzQ0MQ==&action=getalbum&album_id=1331197538447310849&scene=173&from_msgid=2247511076&from_itemidx=1&count=3&nolastread=1#wechat_redirect) 往期内容。

- END -

********关注「HelloGitHub」******第一时间收到******更新********

点击**阅读原文**可按照编程语言查看项目
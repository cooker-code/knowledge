---
title: NotebookLM 开源平替，暴涨 12000+ GitHub Star！
author: GitHubDaily
date:
url: https://mp.weixin.qq.com/s?__biz=MzAxOTcxNTIwNQ==&mid=2457991035&idx=1&sn=5ea38fdcf543a1aa35fba3badc2a5e2c&chksm=8d23ae0f05eadeca6d6a22b5f5722160dcd8396b42db945a956e23d7c855b3087bf35c55cca6&mpshare=1&scene=24&srcid=1206MyR1UwDFodigqHhVaaHt&sharer_shareinfo=7f04fdf1d203756042866a274906d0d2&sharer_shareinfo_first=7f04fdf1d203756042866a274906d0d2#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090206_NotebookLM/090206_核心知识点/NotebookLM类AI阅读工具边界|NotebookLM 类 AI 阅读工具边界]]


这段时间，Google 在 AI 领域可以说是杀疯了，在发布一骑绝尘的 Gemini 3 Pro 模型后，还顺带推出了超强文生图工具 Nano Banana Pro。

随着大家不断探索深入，发现 Nano Banana Pro 搭配上 NotebookLM，便可轻松超出高质量、高保真、对汉字友好的 PPT 和各种产品演示图。

通过网上现有的种种示例，我们可以很明显感受到，Google 已不再将 NotebookLM 定义为普通的 AI 笔记助手，而是欲将其打造成普通人的「第二大脑」。

而要做成这件事，中间便免不了对接各种数据源，包括 PDF、公司文档、会议记录、学术论文、信息资讯等等。

对于一些数据敏感度要求较高的用户来说，更多时候还是希望能由自己来掌控数据权限。

为此，GitHub 上一个名为 **Open NoteBook** 的项目悄然问世，并在最近几天，暴涨到了 12000+ Star，成为近日较火的明星项目。

它的项目简介，把自己定义为一个开源的、注重隐私的 Google Notebook LM 替代方案。

搜索功能强大，数据完全私密，可做到 100%本地化，并且支持多模态。

相比于 Google 的全家桶策略，这个项目更像是一个主打隐私至上，把**「数据控制权」**交还给大家的专业笔记工具。

与 NotebookLM 最大的不同，是除了 Gemini 模型之外，它还支持 OpenAI、Claude 等 16 种 AI 模型。

如果有本地部署需求，也可以用 Ollama、DeepSeek 来实现，让用户在 AI 模型上拥有更多选择，应对不同生产环境。

此外，为了让用户可以整合更多内容，它提供了一套颇为严谨的知识管理系统，并内置了 PDF、Word、视频、网页等多种格式。

待内容整合完毕，我们便可基于 RAG（检索增强生成）来精准匹配关键词，更好理解问题，快速定位相关内容。

还有一个比较有意思的点，是它可以分配内容的开放权限：通过上下文控制，让部分内容对 AI 可见。

借助这种方式，便可以在数据隐私保护和答案内容质量中间，给 AI 提供一个缓冲地带，兼顾内容质量与数据安全。

之前有人用 Google 的 Notebook LM 来搜集各种名人的访谈记录，并基于其著作中的各种观点，生成跨行业、跨时代的虚拟语音播客。

但不足之处在于它最多只能拥有 2 个发言人，而 Open Notebook 虽说语音合成精度略逊一筹，但胜在灵活，可自定义更多发言人角色，拥有更丰富的应用场景。

这里我直接附张产品对比图，让大家可以更加直观的区分对比：

### 安装 & 部署

Open Notebook 提供了多种安装方式，最简单直接的方式，是通过 Docker Compose 一键部署，开箱即用。

在拥有 OPENAI\_API\_KEY 的前提下，我们可以在命令行终端，运行以下命令，即可快速用 Docker 实现部署安装：

```
mkdir open-notebook && cd open-notebookdocker run -d \  --name open-notebook \  -p8502:8502 -p5055:5055 \  -v ./notebook_data:/app/data \  -v ./surreal_data:/mydata \  -eOPENAI_API_KEY=your_key_here \  -eSURREAL_URL="ws://localhost:8000/rpc" \  -eSURREAL_USER="root" \  -eSURREAL_PASSWORD="root" \  -eSURREAL_NAMESPACE="open_notebook" \  -eSURREAL_DATABASE="production" \  lfnovo/open_notebook:v1-latest-single
```

访问地址： http://localhost:8502

如果你想设置远程服务器，或基于 Docker Compose 来简单管理项目，也可以到项目 README 文档中，查看具体操作。

### 写在最后

在 AI 技术日新月异的 2025 年末，Google 打了一套令人叹为观止的产品组合拳。

但 Open Notebook 的存在，也从侧面印证了一点，一个好产品，除了功能强大，还需兼顾数据安全隐私、功能高端制定化的需求。

如果你想简单高效，体验 AI 笔记本带来的便利，对数据敏感度没那么高，那建议直接使用 Notebook LM 即可。

但对于有离线部署需求，对定制化要求高，成本可控，并且在意数据安全的团队，那 Open NoteBook 无疑是目前首选的开源解决方案。

GitHub 项目地址：*https://github.com/lfnovo/open-notebook*

今天的分享到此结束，感谢大家抽空阅读，我们下期再见，Respect！

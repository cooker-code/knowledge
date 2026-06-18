---
title: 22K Star！GitHub上的PDF终结者，3秒把你的扫描件变成可编辑Markdown！
author: GHub开源精选库
date: 小果小果
url: https://mp.weixin.qq.com/s?__biz=MzkwOTY1MTI3Mw==&mid=2247488587&idx=1&sn=ceee1454cd5af898d173c496513f39ea&chksm=c0156eb6ac26ec7eba81f71ea2072bbd22e10614801f4bdf6d77057d96720ae191d880e43cde&mpshare=1&scene=24&srcid=0510zCVFSYHJcGiwycMkU9Iv&sharer_shareinfo=f8adbda5cd3a77db942bef2d7e231aae&sharer_shareinfo_first=f8adbda5cd3a77db942bef2d7e231aae#rd
---

---

各位打工人们，有没有遇到过这种崩溃时刻？收到一份几百页的PDF文档，里面有表格、有公式、还有图片，领导让你把它整理成Markdown格式方便后续处理。你打开文档，发现是扫描件，文字根本复制不了...

以前遇到这种情况，我可能直接打开某网站充个会员，或者干脆手打。

直到前段时间在GitHub上闲逛，发现了一个叫 **Marker** 的神器，真的让我惊了——原来PDF转Markdown这件事，可以做得这么漂亮！

### 这玩意儿到底强在哪？

Marker 是一个基于深度学习模型的文档转换工具，目前在GitHub上已经收获了超过22K的Star 。它不光能把PDF转成Markdown，还支持图片、PPTX、DOCX、XLSX、EPUB这些格式，输出也不限于Markdown，JSON、HTML都能搞定。

最让我惊喜的是它的几个细节：

**首先，它真的很懂中文**。不像有些工具遇到中文PDF就乱码，Marker支持所有语言，中文识别效果相当稳。

**其次，它会把公式转成LaTeX格式**。对于经常要看论文的朋友来说，这简直是救星，不用手动敲那些复杂的数学公式了。

**还有，它能自动去掉页眉页脚**。转换出来的文档干干净净，不会带着"第X页"或者公司Logo这些干扰信息。

**最离谱的是速度**。我之前用云端服务转一份50页的文档可能要等半分钟，Marker在本地跑，几秒钟就搞定了。官方数据显示，在H100显卡上批量处理能达到每秒25页，这速度比很多商业API还快 。

从这张图能看出来，Marker在准确率和速度上都比Llamaparse、Mathpix这些云服务表现更出色 。

### 安装其实没那么难

很多人看到深度学习模型就头大，觉得配环境很麻烦。其实Marker的安装比想象中简单很多。

**前置条件**：你需要Python 3.10或更高版本，还有PyTorch。如果你电脑有NVIDIA显卡最好，没有也没关系，CPU也能跑，就是慢一点。

**第一步，安装**：

```
pip install marker-pdf
```

如果你想处理PDF以外的格式（比如Word、PPT），需要装个完整版：

```
pip install marker-pdf[full]
```

对，就这一行命令，搞定！不需要你下载什么模型权重，Marker第一次运行的时候会自动下载需要的深度学习模型。

**第二步，验证安装**： 装完之后在终端输入：

```
marker_single --help
```

如果看到一堆参数说明，恭喜你，安装成功了。

### 手把手教你用起来

Marker提供了两种主要的使用方式：命令行和Python代码调用。咱们先从最简单的命令行开始。

**单个文件转换**： 假设你有一个叫`report.pdf`的文件，想转成Markdown，只需要：

```
marker_single report.pdf --output_format markdown --output_dir ./output
```

这条命令会在`output`文件夹里生成对应的Markdown文件，还有提取出来的图片。

**一些常用的参数**：

* `--page_range "0,5-10"`：只转换特定页面，比如第1页和第6到11页
* `--force_ocr`：强制进行OCR识别，适合扫描件PDF
* `--use_llm`：启用大模型增强，表格和公式识别会更准（需要配置Gemini或Ollama）
* `--strip_existing_ocr`：清除PDF里原有的OCR文字，重新识别
* `--languages "zh,en"`：指定文档语言，提升中文识别准确率

**批量处理**： 如果你有一整个文件夹的PDF要处理：

```
marker ./input_folder ./output_folder --workers 4
```

`--workers`参数可以设置同时处理几个文件，默认会自动根据你的显存调整。每个worker大概需要5GB显存峰值，3.5GB平均，如果你的显卡有16GB，跑3-4个worker没问题 。

**可视化界面（懒人福音）**： 如果你不喜欢敲命令，Marker还提供了Streamlit做的Web界面：

```
pip install streamlit streamlit-ace  
marker_gui
```

运行后会自动打开浏览器，拖进去PDF文件就能转换，操作简单直观。

网上还有大神做了Windows的一键启动包，完全不用配Python环境，下载解压就能用，对小白特别友好 。

### 进阶玩法：让AI帮你优化结果

Marker最香的功能之一，是可以接入大模型来提升转换质量。开启方式很简单，加个`--use_llm`参数：

```
marker_single document.pdf --use_llm
```

开启后，Marker会调用LLM来做一些聪明的事情，比如把跨页的表格合并、优化行内公式格式、提取表单数据等。

目前支持接入的服务包括：

* **Gemini**：默认用这个，需要设置`GOOGLE_API_KEY`
* **Ollama**：本地运行，隐私性更好，可以设置`--ollama_model`选择模型
* **Claude**、**OpenAI**、**Azure OpenAI**：也都支持，适合有API Key的朋友

根据官方测试，开启LLM模式后，表格识别的准确率能从0.816提升到0.907，效果非常明显 。

### 转换效果长啥样？

我随手拿了一份包含图表的技术文档测试，转换后的Markdown不仅文字准确，图片也自动提取到了单独文件夹，表格格式保留得很完整，公式都变成了标准的LaTeX语法。

如果你想对输出做二次开发，Marker还支持输出JSON格式，包含每个块的坐标、类型、层级信息，方便你做更精细的处理 。

### 一些踩坑经验

用了段时间，也总结了几点经验：

1. **路径别带中文和空格**：虽然Marker支持中文文档，但文件路径最好纯英文，不然偶尔会有奇怪问题。
2. **扫描件记得开OCR**：如果是图片型PDF，一定要加`--force_ocr`参数，不然识别不出来。
3. **显存不够就降worker**：如果爆显存了，手动设置`--workers 1`试试。
4. **LLM模式表格更准**：如果文档里有很多复杂表格，强烈建议开启`--use_llm`，准确率提升很明显。
5. **API服务**：如果你想做成服务给别人调用，Marker还自带了一个简单的FastAPI服务器：

```
pip install uvicorn fastapi python-multipart  
marker_server --port 8001
```

### 写在最后

Marker这个项目最让我感动的点在于，它把原本需要付费API或者复杂工具链才能做的事，变成了一个pip install就能搞定的小工具。对个人用户完全开源免费（Apache 2.0协议），而且性能还比很多商业服务强。

无论是做RAG项目需要处理大量文档，还是平时工作需要整理资料，Marker都能省下大量时间。22K的Star不是白来的，这玩意儿确实好用。

如果你也有PDF转Markdown的需求，不妨试试看。安装只要一分钟，但省下的时间，可能够你喝好几杯咖啡了。

GitHub地址：

https://github.com/VikParuchuri/marker

专注分享 GitHub知识，分享AI 资讯和AI搞米经验，分享OpenClaw使用经验。

想**领取完整版OpenClaw资料**，围观朋友圈，一起交流AI的，可加我VX，备注“gith**ub**"。
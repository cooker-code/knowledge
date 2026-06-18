---
title: 粉丝强烈推荐：MinerU——将PDF转化为机器可读格式的神器
author: 喵开发
date:
url: http://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484457&idx=1&sn=1e0a2111acb5afd2a6b39a53a9a306a8&chksm=fdc6e2b5253eadea12acc71b169de0b576751e22c61915c89d4279bea0398a4798a3c2e7bdc9&mpshare=1&scene=24&srcid=0410jeF6YCKrnRIXyObmtp09&sharer_shareinfo=8e5a646a2d94d795cf4760636355a86d&sharer_shareinfo_first=8e5a646a2d94d795cf4760636355a86d#rd
---
> 已吸收至：[[09_电脑工具/0902_文档与知识工具/090207_文档解析与OCR/090207_核心知识点/PDF解析与OCR工具验收边界|PDF 解析与 OCR 工具验收边界]]


👉 点击上方蓝字「喵开发」关注我，不错过每一篇推送！

各位铲屎官大家好！我是喵~

喵之前给大家介绍了一款微软开源的文档转换工具**[MarkItDown](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247483878&idx=1&sn=a4f1f741e92b0a0759514c78b3104b49&scene=21#wechat_redirect)**

但是MarkItDown有自身的局限性，不能保留格式，热心的粉丝朋友推荐了一个新的工具——**MinerU**[1]

这款工具能够将PDF文档转化为机器可读的格式（如Markdown、JSON等），极大地方便了文档内容的提取和处理。

在这里特别感谢之前粉丝的留言，让我们发现了这款宝藏工具！

    

## 项目简介

MinerU是一款专为PDF文档设计的开源工具，能够将PDF转化为机器可读的格式（如Markdown、JSON等）。它不仅可以提取文本内容，对比MarkItDown能够保留文档的结构（如标题、段落、列表等），并支持图像、表格、公式等元素的提取。可以说是相当的强大。

MinerU 官方宣传视频

## 主要功能

* • **删除冗余元素**：自动删除页眉、页脚、脚注、页码等元素，确保语义连贯, 保留正文图表。
* • **多元素提取**：支持提取图像、图片描述、表格、表格标题及脚注。
* • **公式识别**：自动识别并转换文档中的数学公式、超长公式并且转为LaTeX格式。
* • **表格识别**：自动识别并转换文档中的表格为HTML格式。
* • **保留文档结构**：提取标题、段落、列表等结构，输出符合人类阅读顺序的文本。
* • **OCR支持**：自动检测扫描版PDF和乱码PDF，并启用OCR功能，支持84种语言的检测与识别。
* • **多格式输出**：支持Markdown、JSON等多种输出格式。
* • **多平台支持**：兼容Windows、Linux和Mac平台，支持CPU、GPU、NPU加速。

## 快速开始

### 在线体验

如果你不想安装任何软件，可以直接通过以下在线Demo体验MinerU的功能：

* • OpenDataLab Demo[2]
* • ModelScope Demo[3]
* • HuggingFace Demo[4]

### 使用Docker快速部署

Docker 需设备gpu显存大于等于8GB，默认开启所有加速功能

```
wget https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/docker/china/Dockerfile -O Dockerfile
docker build -t mineru:latest .
docker run --rm -it --gpus=all mineru:latest /bin/bash -c "echo 'source /opt/mineru_venv/bin/activate' >> ~/.bashrc && exec bash"
magic-pdf --help
```

### 使用CPU快速体验

#### 1. 安装magic-pdf

```
conda create -n MinerU python=3.10
conda activate MinerU
pip install -U "magic-pdf[full]" --extra-index-url https://wheels.myhloli.com -i https://mirrors.aliyun.com/pypi/simple
```

#### 2. 使用python脚本 从ModelScope下载模型文件

python脚本会自动下载模型文件并配置好配置文件中的模型目录，配置文件可以在用户目录中找到，文件名为`magic-pdf.json`

```
pip install modelscope
wget https://gcore.jsdelivr.net/gh/opendatalab/MinerU@master/scripts/download_models.py -O download_models.py
python download_models.py
```

#### 3. 修改配置文件以进行额外配置

通过修改脚本自动生成的`magic-pdf.json`文件，可以配置默认模型路径。你可以根据需要修改配置文件中的功能开关，如表格识别功能：

```
{
    "layout-config":{
        "model":"doclayout_yolo"
    },
    "formula-config":{
        "enable":true// 公式识别功能默认开启，如需关闭请修改为false
    },
    "table-config":{
        "enable":true// 表格识别功能默认开启，如需关闭请修改为false
    }
}
```

## 使用方式

### 命令行

MinerU提供了命令行工具，方便用户快速提取PDF内容。具体的命令行参数参考如下：

#### 引用链接

`[1]` \*\*MinerU\*\*: *https://github.com/opendatalab/MinerU*
`[2]` OpenDataLab Demo: *https://mineru.net/OpenSourceTools/Extractor?source=github*
`[3]` ModelScope Demo: *https://www.modelscope.cn/studios/OpenDataLab/MinerU*
`[4]` HuggingFace Demo: *https://huggingface.co/spaces/opendatalab/MinerU*

---

📚 **推荐阅读：**

* • 《[star 8.3k！Deepseek + Easydict：macOS 上最好用的翻译软件！](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484424&idx=1&sn=15fb50866f539aa4a6eb5f5b0ab7431e&scene=21#wechat_redirect)》
* • 《[DeepSee嵌入WPS，实现办公自动化，太牛了！](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484044&idx=1&sn=2d0d140b2d145c1cd3da2274fe98251d&scene=21#wechat_redirect)》
* • 《[超强！通过 DeepSeek + n8n抓取《哪吒2》豆瓣影评并进行AI 情感分析一键搞定！](https://mp.weixin.qq.com/s?__biz=MzU2MDc5NTQxNQ==&mid=2247484010&idx=1&sn=ff2c4bf4d1908050630892924a587f09&scene=21#wechat_redirect)》

---

 

👍 如果你喜欢这篇文章，别忘了**点赞**和**在看**哦！

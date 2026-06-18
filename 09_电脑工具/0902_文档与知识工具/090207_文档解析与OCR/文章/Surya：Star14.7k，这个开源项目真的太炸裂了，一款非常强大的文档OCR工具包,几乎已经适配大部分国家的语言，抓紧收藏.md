---
title: Surya：Star14.7k，这个开源项目真的太炸裂了，一款非常强大的文档OCR工具包,几乎已经适配大部分国家的语言，抓紧收藏
author: 小华同学ai
date: 
url: http://mp.weixin.qq.com/s?__biz=Mzk0MjcxOTM2Nw==&mid=2247486897&idx=1&sn=b5033bb52390098349712e69f3eebbab&chksm=c2ff78dcf8b0c14e7876237ff59e7ca7a03011c28dee7cf402d3ce7e214197e03a42fcda8f4d&mpshare=1&scene=24&srcid=1224RgerPIHsaVnRTOhRg3r5&sharer_shareinfo=bf56719cdfe2653189c2224971b7202d&sharer_shareinfo_first=bf56719cdfe2653189c2224971b7202d#rd
---

# 嗨，大家好，我是小华同学，关注我们获得“**最新、最全、最优质**”开源项目和高效工作学习方法

> **Surya** 是一个功能强大的开源 OCR 文档处理工具包，支持 90 多种语言的 OCR 识别，并提供布局分析、阅读顺序检测和表格识别等功能。本文将详细介绍 Surya 的功能、应用场景和使用方法，帮助您快速掌握这款强大的工具。

## 主要功能

Surya以其全面的功能和高效性能脱颖而出，以下是它的一些核心功能：

* **多语言OCR**：支持90多种语言的OCR，与云服务相比具有竞争力的基准测试结果。
* **文本检测**：能够进行任何语言的行级文本检测。
* **布局分析**：包括表格、图像、标题等的检测。
* **阅读顺序检测**：帮助理解文档的逻辑阅读顺序。
* **表格识别**：能够识别表格的行和列。

### 实际应用场景

Surya适用于多种文档类型，包括PDF、图像和Word文档等。通过实际应用案例，我们可以更直观地了解Surya的强大功能。

| 检测 | OCR |
| --- | --- |
|  |  |
|  |  |

| 布局 | 阅读顺序 |
| --- | --- |
|  |  |

| 表格识别 |  |
| --- | --- |
|  |  |

### 使用示例

Surya支持多种语言的文档处理，以下是一些具体的使用示例：

| 名称 | 检测 | OCR | 布局 | 顺序 | 表格识别 |
| --- | --- | --- | --- | --- | --- |
| 日语 |  |  |  |  |  |
| 简体中文 |  |  |  |  |  |
|  |  |  |  |  |  |
| 印地语 |  |  |  |  |  |
| 阿拉伯语 |  |  |  |  |  |
|  |  |  |  |  |  |

### 安装和使用

Surya的安装和使用相对简单。你需要Python 3.10+和PyTorch。安装后，模型权重将在首次运行Surya时自动下载。

### 性能提示

在使用GPU时，正确设置`RECOGNITION_BATCH_SIZE`环境变量可以显著提高性能。每个批次项将使用`40MB`的VRAM，因此可以实现非常高的批量大小。默认批量大小为`512`，将使用约20GB的VRAM。

### 从Python使用

以下是如何从Python使用Surya进行OCR的示例代码：

```
from PIL import Image  
from surya.ocr import run_ocr  
from surya.model.detection.model import load_model as load_det_model, load_processor as load_det_processor  
from surya.model.recognition.model import load_model as load_rec_model  
from surya.model.recognition.processor import load_processor as load_rec_processor  
  
image = Image.open(IMAGE_PATH)  
langs = ["en"]  # 替换为你的语言 - 可选但推荐  
det_processor, det_model = load_det_processor(), load_det_model()  
rec_model, rec_processor = load_rec_model(), load_rec_processor()  
  
predictions = run_ocr([image], [langs], det_model, det_processor, rec_model, rec_processor)
```

### 编译

Surya支持模型的编译，你可以通过设置环境变量来启用编译：

* 识别：`COMPILE_RECOGNITION=true`
* 检测器：`COMPILE_DETECTOR=true`
* 布局：`COMPILE_LAYOUT=true`
* 表格识别：`COMPILE_TABLE_REC=true`

或者，你也可以设置`COMPILE_ALL=true`来编译所有模型。

### 文本行检测

Surya可以检测文本行并输出包含检测到的边界框的JSON文件。

### 布局和阅读顺序

Surya可以输出包含检测到的布局和阅读顺序的JSON文件。

### 表格识别

Surya可以输出包含检测到的表格单元格和行/列ID以及行/列边界框的JSON文件。

### 限制

Surya专门用于文档OCR，可能不适用于照片或其他图像。它适用于印刷文本，而不是手写文本（尽管它可能适用于某些手写文本）。

### 故障排除

如果OCR工作不正常，可以尝试以下方法：

* 增加图像的分辨率，使文本更大。如果分辨率已经很高，尝试将其降低到不超过`2048px`的宽度。
* 对图像进行预处理（二值化、去倾斜等）可以帮助处理非常老旧/模糊的图像。
* 可以调整`DETECTOR_BLANK_THRESHOLD`和`DETECTOR_TEXT_THRESHOLD`以获得更好的结果。

### 手动安装

如果你想开发Surya，可以手动安装它：

* `git clone https://github.com/VikParuchuri/surya.git`
* `cd surya`
* `poetry install` - 安装主要和开发依赖项
* `poetry shell` - 激活虚拟环境

### 基准测试

Surya在OCR、文本行检测、布局分析、阅读顺序和表格识别方面都有出色的表现。基准测试结果表明，Surya在多个方面都优于现有的解决方案。

### 同类项目

除了 Surya 之外，还有一些其他开源 OCR 项目，例如：

* **Tesseract OCR**: Tesseract OCR 是一个开源 OCR 引擎，支持多种语言，并且可以运行在多种平台上。
* **OCRopus**: OCRopus 是一个开源 OCR 平台，提供多种 OCR 功能，例如文本检测、字符识别等。
* **Kraken**: Kraken 是一个开源 OCR 引擎，支持多种语言，并且可以运行在多种平台上。

通过这篇文章，我们详细介绍了Surya的功能、应用场景和使用方法，希望能帮助读者更好地理解和使用这个强大的OCR工具包。随着技术的不断进步，我们相信Surya将在文档数字化处理领域发挥更大的作用。

## 项目地址

```
https://github.com/VikParuchuri/surya
```
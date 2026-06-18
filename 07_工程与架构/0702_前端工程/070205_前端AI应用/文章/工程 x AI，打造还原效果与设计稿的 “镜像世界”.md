---
title: 工程 x AI，打造还原效果与设计稿的 “镜像世界”
author: 数据可视化 AntV
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzg3NTU4OTc3OA==&mid=2247497796&idx=1&sn=4ef397b3694d6141a1ce027171c9ae5e&chksm=ce1ebc73f234d830df267270ecc813d0874a7bd4456e5cc2548d206985bfe687ef1d0d46c705&mpshare=1&scene=24&srcid=0611YVTM04u5y3RPUz6sunnE&sharer_shareinfo=fbbc451646ad8477a210efb7b52bacf6&sharer_shareinfo_first=fbbc451646ad8477a210efb7b52bacf6#rd
---

> ❝
>
> 上一章节中，我们介绍了基于 WeaveFox-VL[1] 大模型孵化 VIS 模型的过程。然而，模型输出仅包含图表类型及其位置信息，这只是迈向研发可用阶段的初始探索。为此，我们深度融合工程技术与 AI，通过不断优化技术路径，使生成图表与原图在视觉效果和细节呈现上相似。
>
> ❞

## 图表检测：开启 “镜像世界” 的钥匙

图表类型和位置检测，是打开 “镜像世界” 大门的第一把钥匙。我们构建的视觉语言联合模型，基于 Grounding DINO、Swin Transformer 等前沿技术，能够像敏锐的猎手一样，精准识别设计稿中的各类图表组件，准确输出图表类型与位置信息。

## 还原算法：雕琢 “镜像世界” 的工匠

但这仅仅是第一步，真正让 “镜像世界” 栩栩如生的，是工程侧一系列精妙绝伦的还原算法。这些算法如同技艺精湛的工匠，对图表的图例、数据、颜色、数据标注、圆角等配置进行深度检测和还原，不放过每一个细节。

数据解析算法提取设计稿隐含的数据逻辑，将其转化为可计算的数据；颜色匹配算法通过色彩空间转换和对比，确保生成图表的颜色与设计稿相近；细节还原算法则专注于处理数据标注、圆角、边框等细微之处，让生成图表与设计稿完美契合；组件还原算法则匹配图表图例、坐标轴、刻度文档等信息。

> ❝
>
> IR（**I**ntermediate **R**epresentation）：为了能够使一个图片资产，同时支持生产多端代码（Antd、Cube、小程序等等），我们设计了一套 IR 标准。IR 标准描述了整个代码模型中所支持的组件、属性、样式等。
>
> ❞

## 应用成果：“镜像世界” 的璀璨绽放

从实际数据成果分析，整体效果基本达标。多数图表在视觉还原方面表现良好，但当视觉稿包含辅助性标注、水印等非核心内容时，数据解析结果会出现偏差；此外，在处理复杂场景时也存在数据解析丢失的问题，如散点图大量数据点重叠。对比 v0.dev、claude 等同类竞品，发现其在相同场景下也存在类似技术瓶颈。

## 算法详解：解码 “镜像世界” 的技术密码

### 数据解析算法

设计稿中的数据往往以图形化形式存在，数据解析算法需完成从视觉符号到结构化数据的转换。该算法融合图像识别与自然语言处理技术，先通过 OCR 技术提取图表中的文本数据，再利用计算机视觉算法分析图形尺寸、比例关系等信息。例如，对于折线图，算法会识别折线上的标记点位置，结合坐标轴刻度计算出具体数值；对于饼图，则通过扇区角度占比推算数据比例。最终，算法将这些信息整合为可直接用于图表渲染的结构化数据。

以柱状图为例：

该方案依赖 ORC  返回的文本信息，特别是 0 和 200 的 bbox 信息，校准侧通过特征匹配出 [0, 50, 100, 150 , 200] 符合图表值域的特征， 取 0  和  200 之间的区域作为 Plot 区域（用于绘制 Interval Mark 的有效画布）。

data 还原只需要对 Element 进行顶点坐标投影，由公式 `1 - y / h = v / ( 200 - 0)` 推导即可得出相关数值。该方法依赖 0 和 200 相关的信息，当对应 ORC 丢失时，需要先进行补数操作。

### 颜色匹配算法

颜色匹配算法相对复杂一些，对于简单图表，只需获取主色的，推荐使用直方图统计法，按需结合 K-Means 对临近色进行合并即可。

```
from PIL import Image  
import numpy as np  
  
def get_dominant_color(image_path):  
    image = Image.open(image_path)  
    image = image.convert('RGB')  
    pixels = np.array(image).reshape(-1, 3)  
    colors, counts = np.unique(pixels, axis=0, return_counts=True)  
    dominant_color = colors[counts.argmax()]  
    return dominant_color
```

对于多色图表，由于没法确认簇的数量，K-Means 相关的聚类算法只能作为兜底方案，可以通过图像分割、轮廓检测等方案进行高效取色。

以饼图为例：

我们只需要通过霍夫圆检测获取饼图的外轮廓，依次缩小半径仅即可获相关配色信息。

```
    circles = cv2.HoughCircles(  
        blurred,  
        cv2.HOUGH_GRADIENT,  
        1,  
        minDist=min(resize_width, resize_height) / 5,  
        param1=50,  
        param2=30,  
        minRadius=int(minDist),  
        maxRadius=int(maxRadius),  
    )  
  
 circles = np.uint16(np.around(circles))  
    outer_circle = max(circles, key=lambda c: c[2])  
 cx, cy, radius = max_circle  
  
for offset in offset_list:  
for angle in range(0, 360, step):  
   x = cx + int(offset * math.cos(math.radians(angle)))  
            y = cy + int(offset * math.sin(math.radians(angle)))  
   colors.append(tuple(img[y, x]))
```

### 细节还原算法

细节还原算法聚焦图表的微小元素，采用多尺度特征融合技术实现精准还原。对于数据标注，算法利用目标检测模型定位标注文本位置，结合光学字符识别（OCR）和文本布局分析技术，确保标注与数据的对应关系；在处理圆角、边框等样式时，算法通过边缘检测和形态学运算提取形状特征，再基于参数化图形建模技术生成相应的图形元素。

### 组件还原算法

组件还原针对不同组件采用不同的检测机制，包括图例、坐标轴、刻度文档等组件信息，这里不一一展开。

以图例为例：

通过对不同库图例进行分析，我们不难发现其具备如下特征，我们需要做的仅仅是对图片元素进行特征匹配即可。

* Mark + Text 组成
* 颜色通道和主图相同
* 分类数量和主图相同
* 一般位于主图四周

例如检测对应 texts 前是否存在图例 Marker 特征：

```
def check_legend_marker(texts, colors, offset=MARKER_OFFSET, threshold=COLOR_THRESHOLD):  
    marker_colors = set()  
    for _, text_site in texts:  
        x1, y1, x2, y2 = text_site  
        y_diff = y2 - y1  
        step = 2  
        # Collect colors from the 3 y-coordinate positions  
        item_colors = set(  
            filter(  
                lambda color: not is_invalid_color(color),  
                [  
                    (  
                        img.getpixel((x1 - x, y1 + y))  
                        if x1 - x > box[1][0] and y1 + y < box[1][3]  
                        else (255, 255, 255)  
                    )  
                    for x in range(0, offset, step)  
                    for y in range(0, y_diff, step)  
                ],  
            )  
        )  
  
        if item_colors:  
            for c in colors:  
                if any(color_distance(color, c) < threshold for color in item_colors):  
                    marker_colors.add(tuple(c))
```

## 未来展望：拓展 “镜像世界” 的边界

展望未来，我们的探索之旅永不止步。我们计划拓展支持更多复杂图表类型和设计元素，让 “镜像世界” 的版图不断扩大；持续深化 AI 与工程的融合，赋予技术更强大的智能，实现更自动化、更智能的设计。

工程与 AI 的携手，正在为我们描绘一幅智能设计的宏伟蓝图。“镜像世界” 的打造，不仅是技术的突破，更是对未来设计开发模式的大胆探索。

不想看文字？那就上 B 吧：https://space.bilibili.com/3546816974948387[2]

### Reference

[1] 

WeaveFox-VL: *https://weavefox.alipay.com/*

[2] 

https://space.bilibili.com/3546816974948387: *https://space.bilibili.com/3546816974948387*
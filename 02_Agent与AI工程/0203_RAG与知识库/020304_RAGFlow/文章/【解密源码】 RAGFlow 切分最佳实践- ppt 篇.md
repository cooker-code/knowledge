---
title: 【解密源码】 RAGFlow 切分最佳实践- ppt 篇
author: AI慢想者
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483743&idx=1&sn=24f525106bc12042e0c127e7e1d64d91&chksm=c4491c6e0a3df7e6ff5370616aa1994ba0e975b7d96f5df31dad3d5fc1a62cba8e109ee5db56&mpshare=1&scene=24&srcid=1212YAEAtxeG8qPgRcwz8fuM&sharer_shareinfo=934eb12eb6d4927738ec4137c893ff9a&sharer_shareinfo_first=934eb12eb6d4927738ec4137c893ff9a#rd
---

引言

在 RAGFlow 的文档解析体系中，PPT 文件的解析流程相对独特。它不仅要提取页面中的文字、表格内容，还需要生成每一页的缩略图，保证多模态检索场景下的语义对齐。

本篇文章将深入拆解 RAGFlow 中的 PptParser 与 Ppt 实现逻辑，分析它如何在 文本抽取、层级保持、格式还原 与 图像生成 之间实现高效平衡。

省流版

 核心逻辑 

\_\_extract()：提取每个形状（shape）的文本框、表格和组合内容；\_\_get\_bulleted\_text()：保留项目符号、编号、图片标识等原始结构；\_\_call\_\_()：按“从上到下、从左到右”的顺序遍历页面，拼接文本；Ppt()：生成每页缩略图，并与提取文本对应绑定。

 设计亮点 

自动检测三类项目符号（字符、编号、图片），最大程度保留语义层次；

每页生成缩略图，天然支持后续视觉问答与多模态 RAG。

手撕版

1. PPT 文件解析

只支持 .pptx 格式的 ppt 文件解析，不兼容旧版本 .ppt 格式文件解析。

```
if re.search(r"\.pptx?$", filename, re.IGNORECASE):    ppt_parser = Ppt()    for pn, (txt, img) in enumerate(ppt_parser(            filename if not binary else binary, from_page, 1000000, callback)):        d = copy.deepcopy(doc)        pn += from_page        d["image"] = img        d["doc_type_kwd"] = "image"        d["page_num_int"] = [pn + 1]        d["top_int"] = [0]        d["position_int"] = [(pn + 1, 0, img.size[0], 0, img.size[1])]        tokenize(d, txt, eng)        res.append(d)    return res
```

Ppt 继承基类 PptParser

```
class Ppt(PptParser):    ...
```

2. PptParser 基类

\_\_extract：提取 ppt 文本内容，包括文本框文案和表格内容。

\_\_get\_bulleted\_text：提取文本框中文本内容，按照原文本样式输出。

2.1 \_\_extract

提取文本框中文字，换行符分隔返回。

```
if hasattr(shape, 'has_text_frame') and shape.has_text_frame:    text_frame = shape.text_frame    texts = []    for paragraph in text_frame.paragraphs:        if paragraph.text.strip():            texts.append(self.__get_bulleted_text(paragraph))    return "\n".join(texts)
```

提取表格内容，每行内容按照“"表头1: 值1; 表头2: 值2;”格式输出，内容行之间换行符分隔。

```
# Handle tableif shape_type == 19:    tb = shape.table    rows = []    for i in range(1, len(tb.rows)):        rows.append("; ".join([tb.cell(            0, j).text + ": " + tb.cell(i, j).text for j in range(len(tb.columns)) if tb.cell(i, j)]))    return "\n".join(rows)
```

递归提取组合图形内容。

```
# Handle group shapeif shape_type == 6:    texts = []    for p in sorted(shape.shapes, key=lambda x: (x.top // 10, x.left)):        t = self.__extract(p)        if t:            texts.append(t)    return "\n".join(texts)
```

2.2 \_\_get\_bulleted\_text

按照文档中文本的原始格式输出，保证信息的完整性。

```
def __get_bulleted_text(self, paragraph):    is_bulleted = bool(paragraph._p.xpath("./a:pPr/a:buChar")) or bool(paragraph._p.xpath("./a:pPr/a:buAutoNum")) or bool(paragraph._p.xpath("./a:pPr/a:buBlip"))    if is_bulleted:        return f"{'  '* paragraph.level}.{paragraph.text}"    else:        return paragraph.text
```

通过XPATH查询检查段落的XML结构，判断是否包含以下三种项目符号相关元素之一：

* buChar：自定义字符项目符号

+ 场景：PPT中使用特定字符（如圆点、方块、箭头等）作为项目符号的列表

+ 示例：常见的 "• 第一点"、"■ 第二点" 这类列表

* buAutoNum：自动编号

+ 场景：PPT中使用数字或字母顺序编号的列表

+ 示例："1. 第一步"、"2. 第二步" 或 "a. 首先"、"b. 其次" 这类有序列表

* buBlip：图片项目符号

+ 场景：PPT中使用小图片或图标作为项目符号的列表

+ 示例：使用公司logo、自定义图标等作为列表标记的情况

2.3 实例化方法 \_\_call\_\_

按照阅读习惯按照从上到下，从左到右的方式排列 ppt 页，按页解析获取文本。最终输出包含所有文本的列表。

```
for shape in sorted(slide.shapes, key=lambda x: ((x.top if x.top is not None else 0) // 10, x.left if x.left is not None else 0)):    txt = self.__extract(shape)    if txt:        texts.append(txt)
```

3. Ppt 类

将每页生成缩略图，

```
with slides.Presentation(BytesIO(fnm)) as presentation:    for i, slide in enumerate(presentation.slides[from_page: to_page]):        try:            with BytesIO() as buffered:                slide.get_thumbnail(                    0.1, 0.1).save(                    buffered, drawing.imaging.ImageFormat.jpeg)                buffered.seek(0)                imgs.append(Image.open(buffered).copy())
```

通过上方步骤获取 txt 文本信息与 image ppt 每页缩略图后，对 txt 文本信息进行分词得到最终 chunk，分词 tokenize 实现参考[《分词器原理》。](https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483693&idx=1&sn=294e58e479b8c9705a19a22ad9da20eb&scene=21#wechat_redirect)
---
title: 【解密源码】 RAGFlow 切分最佳实践 - General 模式语义切块（markdown 篇）
author: AI慢想者
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483714&idx=1&sn=b5fb49f32204550dbeeab7ef4c84f879&chksm=c450ffed1a267983d92fddeac3487f5d6bc4124592077527025e47eba098b27b48f1788ae62d&mpshare=1&scene=24&srcid=1123WjJ4A06WjVphxya7sYLT&sharer_shareinfo=832ec2f7dde28a061ae2bca108d2d59b&sharer_shareinfo_first=832ec2f7dde28a061ae2bca108d2d59b#rd
---

引言

在 RAGFlow 的语义切块体系中，markdown 是一种独特且复杂的输入类型。不同于 PDF 或 DOCX 等二进制格式，markdown 天然带有语义结构（标题、列表、引用、代码块、表格等），这让解析器不需要额外的 OCR 或排版分析即可精确还原内容层次。

然而，markdown 的灵活性也是一把双刃剑——不同语法风格（如无边框表格）、嵌套元素、图片链接、HTML 混写等情况都会使语义切块变得复杂。  

因此，RAGFlow 在 markdown 的解析中采用了专门的 MarkdownParser 与 MarkdownElementExtractor 双层架构，并引入了 VisionFigureParser 来增强图片语义，使文本与图像形成真正的多模态语义块。

本篇将带你深入源码层，解读 RAGFlow 如何一步步完成从 markdown 文档到结构化语义块的全过程。

省流版

Markdown 文档的语义切块在 RAGFlow 中由三个阶段完成：语法结构解析 → 图片提取与视觉增强 → 文本合并与分词。

 核心流程一览 

结构解析 

使用 MarkdownElementExtractor 提取标题、段落、列表、代码块、引用等语义单元。  

表格提取   

识别标准与无边框 markdown 表格，选择“分离模式”或“渲染模式”进行存储。  

图片处理 

从文本中提取图片链接，下载或加载本地图片；合并同一段落的图片，并交由视觉模型生成语义描述。  

多模态融合 

将文本与视觉描述拼接，形成具备上下文的语义段。  

分词与切块 

最终调用 naive\_merge\_with\_images 与 tokenize\_chunks\_with\_images，完成切分文本与图片的关联。

 设计亮点 

双模表格解析体系：同时兼容“标准表格（带边框）”与“无边框表格”，支持独立存储（结构化输入）与渲染替换（视觉还原），保证准确性与可读

手撕版

1. 解析器初始化

```
markdown_parser = Markdown(int(parser_config.get("chunk_token_num", 128)))
```

1.1 Markdown 类

继承MarkdownParser基类，解析表格；使用 MarkdownElementExtractor().extract\_elements()  解析文本内容。

```
class Markdown(MarkdownParser):    def __call__(self, filename, binary=None, separate_tables=True):        if binary:            encoding = find_codec(binary)            txt = binary.decode(encoding, errors="ignore")        else:            with open(filename, "r") as f:                txt = f.read()        remainder, tables = self.extract_tables_and_remainder(f'{txt}\n', separate_tables=separate_tables)        extractor = MarkdownElementExtractor(txt)        element_sections = extractor.extract_elements()        sections = [(element, "") for element in element_sections]        tbls = []        for table in tables:            tbls.append(((None, markdown(table, extensions=['markdown.extensions.tables'])), ""))        return sections, tbls
```

1.2 MarkdownParser 基类

extract\_tables\_and\_remainder：提取文档中的表格以及其他部分。

1.2.1 extract\_tables\_and\_remainder

提取文档中的表格内容，兼容标准语法和简化语法的表格解析，同时提供分离模式和渲染模式两种不同的解析方案，解析表格内容用于不同场景。

1）两种表格识别方案，兼容 markdown 中两种表格的语法形式。

```
if "|" in markdown_text:  # for optimize performance    # 标准 Markdown 表格    border_table_pattern = re.compile(        r"""        (?:\n|^)        (?:\|.*?\|.*?\|.*?\n)        (?:\|(?:\s*[:-]+[-| :]*\s*)\|.*?\n)        (?:\|.*?\|.*?\|.*?\n)+    """,        re.VERBOSE,    )    working_text = replace_tables_with_rendered_html(border_table_pattern, tables)    # 无边框 Markdown 表格    no_border_table_pattern = re.compile(        r"""        (?:\n|^)        (?:\S.*?\|.*?\n)        (?:(?:\s*[:-]+[-| :]*\s*).*?\n)        (?:\S.*?\|.*?\n)+        """,        re.VERBOSE,    )    working_text = replace_tables_with_rendered_html(no_border_table_pattern, tables)
```

标准 Markdown 表格

```
1. 使用管道符 | 包围所有单元格2. 必须有分隔线行（第二行）3. 单元格内容可以包含空格  
| 姓名 | 年龄 | 部门 | 薪资 ||------|------|------|------|| 张三 | 25   | 技术部 | 15000 || 李四 | 30   | 销售部 | 12000 || 王五 | 28   | 市场部 | 13000 |
```

无边框 Markdown 表格

```
1. 只有分隔线行使用 | 和 -2. 数据行的单元格不被 | 包围  
姓名 年龄 部门 薪资---- ---- ---- -----张三 25   技术部 15000李四 30   销售部 12000王五 28   市场部 13000
```

2）两种表格解析方案

```
if separate_tables:    # Skip this match (i.e., remove it)    new_text += working_text[last_end : match.start()] + "\n\n"else:    # Replace with rendered HTML    html_table = markdown(raw_table, extensions=["markdown.extensions.tables"]) if render else raw_table    new_text += working_text[last_end : match.start()] + html_table + "\n\n"
```

表格分离模式 (separate\_tables=True)

将整个表格转换成文本，保证语义完整性，作为后续切分和量化的数据基础。

```
输入 Markdown:正文...| 姓名 | 年龄 | 部门 ||------|------|------|| 张三 | 25   | 技术 |正文...  
输出:正文...正文...表格单独存储: ["| 姓名 | 年龄 | 部门 |\n|------|------|------|\n| 张三 | 25   | 技术 |"]
```

表格渲染模式 (separate\_tables=False)

保持原始格式和样式，满足重现文档外观需要。

```
输入 Markdown:正文...| 姓名 | 年龄 | 部门 ||------|------|------|| 张三 | 25   | 技术 |正文...  
输出:正文...<table>  <tr><th>姓名</th><th>年龄</th><th>部门</th></tr>  <tr><td>张三</td><td>25</td><td>技术</td></tr></table>正文...
```

1.3 MarkdownElementExtractor 类

建立针对 Markdown 语法规则，通过不同规则解析文本中的各个元素。

```
def extract_elements(self):    sections = []    i = 0    while i < len(self.lines):        line = self.lines[i]  
        # 优先级解析顺序        if re.match(r"^#{1,6}\s+.*$", line):           # 1. 标题            element = self._extract_header(i)        elif line.strip().startswith("```"):           # 2. 代码块            element = self._extract_code_block(i)        elif re.match(r"^\s*[-*+]\s+.*$", line) or \   # 3. 列表             re.match(r"^\s*\d+\.\s+.*$", line):            element = self._extract_list_block(i)        elif line.strip().startswith(">"):             # 4. 引用块            element = self._extract_blockquote(i)        elif line.strip():                             # 5. 文本块            element = self._extract_text_block(i)        else:                                          # 6. 空行            i += 1            continue  
        sections.append(element["content"])        i = element["end_line"] + 1  # 跳到下一个元素  
    return [section for section in sections if section.strip()]
```

2. 解析文本结构

通过解析器 markdown\_parser 中的方法解析出文本结构和表格内容。

```
sections, tables = markdown_parser(filename, binary, separate_tables=False)
```

3. 获取 section 中的图片

```
section_images = []for idx, (section_text, _) in enumerate(sections):    images = markdown_parser.get_pictures(section_text) if section_text else None
```

3.1 markdown\_parser.get\_pictures

1）获取图片链接

```
image_urls = self.get_picture_urls(text)def get_picture_urls(self, sections):    ...    from bs4 import BeautifulSoup    # 将 markdown 文档转换成 html    html_content = markdown(text)    soup = BeautifulSoup(html_content, 'html.parser')    # 提取图片链接    html_images = [img.get('src') for img in soup.find_all('img') if img.get('src')]    return html_images
```

2）获取图片（远程 or 本地）

```
if url.startswith(('http://', 'https://')):    # 远程 URL：下载图片    response = requests.get(url, stream=True, timeout=30)    if response.status_code == 200 and response.headers['Content-Type'].startswith('image/'):        img = Image.open(BytesIO(response.content)).convert('RGB')        images.append(img)else:    # 本地文件路径：直接打开    from pathlib import Path    local_path = Path(url)    if not local_path.exists():        logging.warning(f"Local image file not found: {url}")        continue    img = Image.open(url).convert('RGB')    images.append(img)
```

4. 合并图片

垂直合并同一个 section 中图片，使用视觉模型将图片解析成文本描述

```
combined_image = reduce(concat_img, images) if len(images) > 1 else images[0]section_images.append(combined_image)markdown_vision_parser = VisionFigureParser(vision_model=vision_model, figures_data= [((combined_image, ["markdown image"]), [(0, 0, 0, 0, 0)])], **kwargs)boosted_figures = markdown_vision_parser(callback=callback)sections[idx] = (section_text + "\n\n" + "\n\n".join([fig[0][1] for fig in boosted_figures]), sections[idx][1])
```

VisionFigureParser 的详细解析可参考[《General 模式语义切块（docx 篇）》](https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483697&idx=1&sn=cfbabc08f0fa0f8aeae9457a2f0ab579&scene=21#wechat_redirect)中的 VisionFigureParser 类模块。

5. 分词

对文本和表格内容进行分词

```
res = tokenize_table(tables, doc, is_english)
```

tokenize\_table 的详细解析可参考[《分词器原理》](https://mp.weixin.qq.com/s?__biz=Mzk4ODg5NTU4Mg==&mid=2247483693&idx=1&sn=294e58e479b8c9705a19a22ad9da20eb&scene=21#wechat_redirect)。

6. sections 后处理

```
chunks, images = naive_merge_with_images(sections, section_images,                                int(parser_config.get(                                    "chunk_token_num", 128)), parser_config.get(                                    "delimiter", "\n!?。；！？"))res.extend(tokenize_chunks_with_images(chunks, doc, is_english, images))
```

naive\_merge\_with\_images：将每个 section 与合并后的 section 图片进行关联。如果是跨页的 chunk，则需要将两页的图片合并后进行关联。

tokenize\_chunks\_with\_images：分词，返回统一格式的结构化文档块，作为后续向量化输入。

下期预告

在本期《【解密源码】RAGFlow 切分最佳实践 - General 模式语义切块（markdown 篇）》中，我们深入剖析了 .md 文档在 RAGFlow 中的完整解析流水线，区别于之前 docx，pdf 文档解析，通过根据 markdown 与生俱来的结构化语法进行文本解析后，将文档中的图片单独解析出来，最后进行关联。

在下一期中，我们将剖析 General 模式下最后的三个文件类型 html|json|doc 的语义切块方案。
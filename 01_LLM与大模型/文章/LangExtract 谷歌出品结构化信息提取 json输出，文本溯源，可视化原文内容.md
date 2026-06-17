---
title: LangExtract 谷歌出品结构化信息提取 json输出，文本溯源，可视化原文内容
author: AI大模型智能体
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMzczMjU1Ng==&mid=2247483940&idx=1&sn=a68fef2a49542f96050bd04a6735cec1&chksm=c37582f418c3d84577329f5822938faa498095a20d45c115de5131ead42caeae7a270b101435&mpshare=1&scene=24&srcid=0919GH4jlnZOVvmCuj09zVjC&sharer_shareinfo=d0548738e0baba667caaaef6f1d9f0bf&sharer_shareinfo_first=d0548738e0baba667caaaef6f1d9f0bf#rd
---

相信做东西的时候，一定会遇到输出为json格式，之前langchain有过一篇可以作为参考， 而现在的langextract更近一步，可以把结构化提取的内容， 在原文中找到， 这样可以看来提取内容的来源，同时还可以增加可信度。

[跟着木匠学大模型langchain系列（7）—结构化提取（信息抽取）](https://mp.weixin.qq.com/s?__biz=MzkzMzczMjU1Ng==&mid=2247483704&idx=1&sn=d62d7c48c9f2fbe89411465dee1bd30e&scene=21#wechat_redirect)

一 、提取

LangExtract的提取大家可以自行了解，我比较喜欢用langchain的提取， 同时对langextract 我比较感兴趣的是对齐，文本朔源。以下功能介绍：

1. **精确的源接地：**

   将每个提取映射到其在源文本中的确切位置，实现视觉突出显示，以便于追溯和验证。
2. **可靠的结构化输出：**

   根据您的少量示例强制执行一致的输出模式，利用 Gemini 等受支持模型中的受控生成来保证稳健、结构化的结果。
3. **针对长文档进行了优化：**

   通过使用文本分块、并行处理和多次传递的优化策略来克服大型文档提取的“大海捞针”挑战，以提高召回率。
4. **交互式可视化：**

   立即生成一个独立的交互式 HTML 文件，以在原始上下文中可视化和查看数千个提取的实体。
5. **灵活的 LLM 支持：**

   支持您喜欢的模型，从基于云的 LLM（如 Google Gemini 系列）到通过内置 Ollama 接口的本地开源模型。
6. **适用于任何领域：**

   仅使用几个示例即可定义任何域的提取任务。LangExtract 可以适应您的需求，无需任何模型微调。
7. **利用知识图谱：**

   利用精确的提示措辞和少量示例来影响提取任务如何利用 LLM 知识。任何推断信息的准确性及其对任务规范的遵守取决于所选的 LLM、任务的复杂性、提示指令的清晰度以及提示示例的性

二、对齐

当我们从文本中提取到内容，需要和原文进行对齐，就可以达到文本朔源的目的， 但是大模型提取的内容和原文会有各种差异，所以让我们详细看看对齐方法，这里对齐，有精确对齐， 模糊对齐， 超长对齐；都是以token来计算标准的

```
extraction_text = "关于开展2024年海尔招聘活动的通知"source_text = """  
# 关于开展2024年海尔招聘活动的通知各单位：此处省略很多很多的正文内容"""
```

2.1 文本分词处理（Tokenization）

## 

## 首先，系统会对文本进行分词处理：

```
extraction_tokens = ["关于", "开展", "2024", "年", "海尔", "招聘", "活动", "的", "通知"]
```

⚡关键点

- 所有token都会转换为小写

- 使用langextract内置的中文分词器

- 保留token的字符位置信息

2.2 ：精确对齐（Exact Alignment）

  使用Python的 `difflib.SequenceMatcher` 寻找完全匹配的token序列。设置序列匹配器

```
matcher = difflib.SequenceMatcher(autojunk=False)matcher.set_seqs(a=source_tokens, b=extraction_tokens)
```

获取匹配块

```
matching_blocks = matcher.get_matching_blocks()
```

```
# 结果：[(0, 0, 9), (18, 9, 0)]  # (源位置, 提取位置, 匹配长度)
```

匹配分析：

- 在源文本位置0开始，找到9个连续匹配的token

- source\_tokens[0:9] == extraction\_tokens[0:9]

- 完全匹配：["关于", "开展", "2024", "年", "海尔", "招聘", "活动", "的", "通知"]

精确匹配成功条件：

- ✅ 匹配长度 = 提取文本token数量 (9 == 9)

- ✅ 提取文本从位置0开始匹配 (j == 0)

- ✅ 结果：`MATCH\_EXACT` 状态

2.3：模糊对齐（Fuzzy Alignment）

当精确匹配失败时，启动模糊对齐机制。

设置一个 fuzzy\_threshold = 0.4  # 40% 相似度阈值

extraction\_tokens = ["关于", "开展", "2024", "年", "海尔", "招聘", "活动", "的", "通知"]

min\_overlap = int(9 \* 0.4) = 3  # 最少需要3个token重叠

2.3.1 Token标准化

# 处理复数、时态等变化

```
def normalize_token(token):    token = token.lower()    if len(token) > 3 and token.endswith("s"):        return token[:-1]  # 去除复数s    return token
```

extraction\_counts = Counter(["关于", "开展", "2024", "年", "海尔", "招聘", "活动", "的", "通知"])

# {'关于': 1, '开展': 1, '2024': 1, '年': 1, '海尔': 1, '招聘': 1, '活动': 1, '的': 1, '通知': 1}

2.3.2 滑动窗口匹配

窗口大小从9(与提取文本相同长度)开始，逐步增大：

```
for start_idx in range(len(source_tokens) - 9 + 1):    window = source_tokens[start_idx:start_idx + 9]    # 例如：start_idx = 0    window = ["关于", "开展", "2024", "年", "海尔", "招聘", "活动", "的", "通知"]    window_counts = Counter(window)    # 快速预检查：计算重叠    overlap = extraction_counts & window_counts    # 结果：{'关于': 1, '开展': 1, '2024': 1, '年': 1, '海尔': 1, '招聘': 1, '活动': 1, '的': 1, '通知': 1}    overlap_total = 9  # 完全重叠！    if overlap_total >= min_overlap:  # 9 >= 3 ✅        # 进行详细序列匹配        fuzzy_matcher.set_seq1(window)        matches = sum(size for _, _, size in fuzzy_matcher.get_matching_blocks())        ratio = matches / len(extraction_tokens) = 9 / 9 = 1.0        if ratio >= fuzzy_threshold:  # 1.0 >= 0.4 ✅            # 模糊匹配成功！
```

2.3.3  相似度计算详解

对于完全匹配的情况

matching\_blocks = [(0, 0, 9), (9, 9, 0)]# (窗口位置, 提取文本位置, 匹配长度)

matches = sum([9, 0]) = 9  # 匹配的token总数

ratio = 9 / 9 = 1.0  # 100% 匹配

对于模糊匹配的情况，最后ratio大于设置的参数阈值 

2.3.4：字符位置映射

     找到token匹配后，需要映射回原始文本的字符位置，其实就是文本朔源了：

```
# 获取匹配的token范围start_token = tokenized_source.tokens[0]  # "关于"end_token = tokenized_source.tokens[8]    # "通知"# 字符位置char_start = start_token.char_interval.start_pos  # 例如：3char_end = end_token.char_interval.end_pos        # 例如：18# 提取匹配文本matched_text = source_text[3:18]  # "关于开展2024年海尔招聘活动的通知"
```

例子会得到：

- 精确匹配：`MATCH\_EXACT`

- 字符位置：`CharInterval(start\_pos=3, end\_pos=18)`

- 匹配文本：`"关于开展2024年海尔招聘活动的通知"`

三 算法复杂度分析

- 精确匹配： O(n + m) - 非常快

- 模糊匹配：O(n × m × w) - 其中w是窗口数量

- Counter交集优化：\*将大部分O(n×m)操作降为O(k)，k是重叠token数

四  官方给的示例

提取内容 原文内容 都是不同颜色对应的
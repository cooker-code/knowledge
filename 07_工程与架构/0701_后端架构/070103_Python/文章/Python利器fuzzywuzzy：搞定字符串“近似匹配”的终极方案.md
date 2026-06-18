---
title: Python利器fuzzywuzzy：搞定字符串“近似匹配”的终极方案
author: Crossin的编程教室
date: 
url: https://mp.weixin.qq.com/s?__biz=MjM5MDEyMDk4Mw==&mid=2650194567&idx=1&sn=a1c0c21f481b78373345abbc0ed6772f&chksm=bf51f69272c13df10f6f54de3598ad9c0dc62b1929a40b70cda7e42a5745361548bc9dfe97a5&mpshare=1&scene=24&srcid=1001LB3EAkIdacpkaUtYElFD&sharer_shareinfo=3a96ffe0875704eafc0e8ed61c485498&sharer_shareinfo_first=3a96ffe0875704eafc0e8ed61c485498#rd
---

在数据清洗和分析任务中，我们经常遇到这样的难题：两个本该相同的文本数据，因为录入错误、空格、顺序颠倒或冗余信息等问题，导致传统的精确匹配（==）失效。

举个最常见的例子：地址数据分类与去重

你的数据表里可能存在以下几条地址记录，它们指的其实是同一个地方：

* 江苏南京鼓楼区
* 江苏省 南京市 鼓楼
* 南京市鼓楼区

如果用人工去识别和修正，效率低得可怕。这时，你需要一个工具来计算它们之间的“相似度”——这就是 Python fuzzywuzzy 库要解决的事。它能让你高效地进行模糊字符串匹配（Fuzzy String Matching）。

0. 核心原理：Levenshtein 距离

fuzzywuzzy 的底层基于 Levenshtein 距离（编辑距离），它计算将一个字符串转换成另一个字符串所需的最少单字符编辑操作（插入、删除或替换）次数。

fuzzywuzzy 将这个距离转化成一个 0 到 100 的相似度得分，100 代表完全匹配。

1. 安装

如果绝大多数 python 第三方模块，通过 pip 命令即可安装 fuzzywuzzy：

```
pip install fuzzywuzzy
```

推荐安装加速库，提升处理性能：

```
pip install python-levenshtein
```

如果未安装这个依赖，fuzzywuzzy 会回退到纯 Python 实现，性能下降约50%。

2. fuzzywuzzy 的三大匹配策略

fuzzywuzzy.fuzz 模块提供了多种计算比率的方法，以应对不同类型的“模糊”情况。

2.1. fuzz.ratio()：简单比率

这是最基础的计算方法，直接比较两个字符串的相似度。适用于大部分的轻微拼写错误或少量差异。

```
from fuzzywuzzy import fuzz  
s1 = "小米科技有限责任公司"s2 = "小米科技有限责任公司 (北京)" # 存在冗余信息print(f"简单比率: {fuzz.ratio(s1, s2)}") # 80  
s3 = "小木科技有限责任公司" # 存在一个错别字print(f"简单比率: {fuzz.ratio(s1, s3)}") # 90
```

2.2. fuzz.partial\_ratio()：部分比率

当一个短字符串是另一个长字符串的一部分时，这个方法特别有用。它会找到长字符串中的最佳子串与短字符串进行比较。

例如，在地址匹配中，我们只关心核心的街道信息。

```
s_long = "四川省 成都市 武侯区 天府大道中段 99号"s_short = "天府大道中段"  
# 简单比率较低，因为它比较的是整个字符串print(f"简单比率: {fuzz.ratio(s_long, s_short)}") # 43# 部分比率较高，因为它找到了精确匹配的子串print(f"部分比率: {fuzz.partial_ratio(s_long, s_short)}") # 100
```

2.3. fuzz.token\_sort\_ratio()：分词排序比率

这个方法用于解决词语顺序颠倒的问题。它首先将字符串分词，然后按字母顺序排序，最后再进行比率计算。

```
s_a = "张三 李四 王五" # 名字顺序s_b = "王五 李四 张三" # 顺序被打乱  
print(f"简单比率: {fuzz.ratio(s_a, s_b)}") # 50print(f"排序比率: {fuzz.token_sort_ratio(s_a, s_b)}") # 100
```

这种方法适合处理一组人名或复杂地址信息（如“南京市鼓楼区” vs “鼓楼区 南京市”）。

3. 进阶应用：标准化清洗

fuzzywuzzy.process 模块的核心价值在于：帮你从一个“标准答案列表”中，找到与“待清洗数据”最匹配的那一个。这在数据标准化、去重和分类时非常有用。

来看一个示例：假设你在数据库中收集了许多公司名称，但由于录入时不规范，它们都很“脏”。现在，你有一份确定的标准公司名列表，需要将所有脏数据清洗并映射到这些标准名上。

这时我们使用 process.extractOne()，它只返回得分最高的一个匹配结果，非常适合自动化决策。

```
from fuzzywuzzy import processimport pandas as pd  
# 1. 定义标准的公司名列表 (这是你的“标准答案”)STANDARD_NAMES = [    "华为技术有限公司",    "阿里巴巴（中国）有限公司",    "腾讯科技（深圳）有限公司",    "字节跳动有限公司"]  
# 2. 定义待清洗的“脏数据”dirty_data = [    "华微技术有限公斯",      # 拼写错误    "阿里 巴巴（中国）",     # 简写和空格问题    "深圳腾迅科技",          # 顺序颠倒 + 拼写错误    "字节跳動",              # 繁体字    "网易公司"               # 无法匹配项]  
def clean_company_name(name, choices, threshold=50):    """    清洗函数：从标准列表中找到最佳匹配项。    如果相似度低于阈值，则返回原始名称或标记为“待审核”。    """    # 使用 extractOne 找到得分最高的匹配    best_match, score = process.extractOne(name, choices)  
    if score >= threshold:        # 相似度达标，返回标准名称        return best_match    else:        # 相似度太低，保留原始数据，等待人工审核        return f"【待审核】{name}"  
# 3. 执行清洗操作并打印结果print("--- 批量清洗结果 ---")print("{:<20} | {}".format("原始名称", "清洗后名称/标记"))print("-" * 40)  
for dirty_name in dirty_data:    # 调用清洗函数，阈值设置为 80 分    clean_name = clean_company_name(dirty_name, STANDARD_NAMES)  
    # 使用格式化打印，让结果对齐    print("{:<20} | {}".format(dirty_name, clean_name))
```

返回结果：

```
原始名称                 | 清洗后名称/标记----------------------------------------华微技术有限公斯             | 华为技术有限公司阿里 巴巴（中国）            | 阿里巴巴（中国）有限公司深圳腾迅科技               | 腾讯科技（深圳）有限公司字节跳動                 | 字节跳动有限公司网易公司                 | 【待审核】网易公司
```

通过设置合理的相似度阈值（如本例中的 50 分），我们就能实现数据清洗的自动化决策。

总结

fuzzywuzzy 用简洁的 API 解决了字符串匹配中的老大难问题。它能帮你将看似混乱、难以归类的中文数据进行有效的标准化和去重，是 Python 数据清洗中不可或缺的利器。

如果本文对你有帮助，欢迎点赞、评论、转发。你们的支持是我更新的动力~

---

Crossin的新书《**码上行动：用ChatGPT学会Python编程**》已经上市了。本书以ChatGPT为辅助，系统全面地讲解了如何掌握Python编程，适合Python零基础入门的读者学习。**[【点此查看详细介绍】](http://mp.weixin.qq.com/s?__biz=MjM5MDEyMDk4Mw==&mid=2650192081&idx=2&sn=9a25b369cac504cb9662690c9e16da3b&chksm=be4bb5a9893c3cbfbe550634f5466f0b41eb8688e46706d87ab1789090d4671cd67948b6f881&scene=21#wechat_redirect)**

购买后可加入读者交流群，Crossin为你开启陪读模式，解答你在阅读本书时的一切疑问。

Crossin的其他书籍：

---

添加微信 **crossin123**，加入编程教室共同学习~

感谢**转发**和**点赞**的各位~
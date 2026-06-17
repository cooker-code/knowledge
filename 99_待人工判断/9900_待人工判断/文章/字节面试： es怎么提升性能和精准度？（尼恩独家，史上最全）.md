---
title: 字节面试： es怎么提升性能和精准度？（尼恩独家，史上最全）
author: 技术自由圈
date: 
url: http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504431&idx=1&sn=ce937939ab2f9dd64aaa7fe0c5d98f3f&chksm=c024001036c2b74bff9f6876280c0f89e1b7bb646c357c3c479567fc6959c7c84cefc7f1d3cd&mpshare=1&scene=24&srcid=0125noBS5XPfGVmor6FZa14M&sharer_shareinfo=63b77dd2af3d412054a81b421ca4447a&sharer_shareinfo_first=63b77dd2af3d412054a81b421ca4447a#rd
---

FSAC未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在40岁老架构师 尼恩的**读者交流群**(50+)中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的面试题：

* 字节面试： es怎么提升速度和精准度？
* 提升搜索精准度，有那些的实用技巧？
* 高性能的搜索系统如何设计，如何提高搜索精准度？

最近有小伙伴在面试 jd、字节，又遇到了相关的面试题。小伙伴懵了，因为没有遇到过，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩Java面试宝典PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V171版本，供后面的小伙伴参考，提升大家的 3高 架构、设计、开发水平。

> 最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩Java面试宝典》的PDF，请关注本公众号【技术自由圈】获取，回复：领电子书

## 本文目录

****-******尼恩说在前面**

****-**1 如何提升搜索的 性能？**

****-******精准度优化的四个层面**

****-******2 常用的ES四大查询**

 - 2.1 精确匹配查询（term 和 terms 查询）：

 - 2.2 短语查询（ match\_phrase 查询） ：

 - 2.3 布尔查询（bool 查询）：

 - 2.4 match模糊查询 （这个是最难的）

##### -2.4.1 mach模糊查询工作流程

##### -2.4.2 mach 模糊查询语法示例

 - 2.5 ES的  mach模糊查询  和match\_phrase 短语查询的区别

 - 2.6 match\_phrase 短语查询与    term  精确匹配查询  的区别

****-******3 查询构建 层面 的 精准度优化**

 - 3.1 选择合适的查询类型

 - 3.2 合理 设置查询参数

****-******4 索引设置层面 的 精准度优化**

 - 4.1 分词器选择：根据搜索场景，选择合适的内置分词器：

 - 4.2 细颗粒索引，粗颗粒搜索：

 - 4.3 自定义词库：定义一套 业务专用词库

 - 4.4 自定义停词：定义一套 业务专用 停用词

 - 4.5 合理设计字段类型：对于精确匹配的字段，定义为 keyword

 - 4.6 数据清洗 预处理，提前去除噪声数据：

 - 4.7 标准化数据格式：

 - 4.8 精细的字段映射,为同一个字段创建多个映射：

 - 4.9 通过动态模板 ，优化索引映射（mappings）

****-******5 搜索结果层面 的 精准度优化**

 - 5.1 使用`explain`分析相关性评分（\_score）

##### -什么是词频（Term Frequency, TF）？

##### -什么是逆向文档频率（Inverse Document Frequency, IDF）？

##### -什么是字段长度正则化？

 - 5.2 使用`boost` 参数调整相关性评分（\_score）

 - 5.3 使用 function\_score 调整相关性评分（\_score）

##### -functions解释：

 - 5.4 人工评估和用户反馈：

 - 5.5 搜索转化率的计算和分析：

 - 5.6 A/B 测试不同查询策略：

****-******6 应用代码层面 的 精准度优化**

 - 6.1 代码层面，使用 组合模式 实现 精准度优化

 - 6.2 代码层面  组合模式的 精准度优化的实例

 - 6.3 代码层面  组合模式的 精准度优化的流程图

****-******说在最后：有问题找老架构取经‍**

## 1 如何提升搜索的 性能？

es怎么提升速度和精准度？

首先要告诉面试官，这个压根不是一个问题。

**这个， 是两个问题。**

首先是 第一个小的问题： 如何提升搜索的 性能？

具体请参见尼恩前面的 塔尖、深度文章：  [极致 ElasticSearch 调优，让你的ES 狂飙100倍！](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504357&idx=1&sn=05b6e475016bf6c17696783ab6eaeecf&scene=21#wechat_redirect)

然后是 第二个小的问题。接下来的这篇 塔尖深度文章，重点介绍第二个小的问题：  **如何提升搜索的精准度？**

此文， 是尼恩干过足足一年的搜索架构的经验总结， 是尼恩干过足足一年的搜索架构的经验总结、爬坑经验， 大家一定要收藏起来， 看他个 10遍100遍， 吊打面试官、吊打你的技术leader。

## 精准度优化的四个层面

接下来，开始 精准度优化。分为 4个层面：

* 查询构建 层面 的 精准度优化
* 索引设置层面 的 精准度优化
* 搜索结果层面 的 精准度优化
* 应用代码层面 的 精准度优化

## 2 常用的ES四大查询

按照尼恩的 几十年来的规矩， 从最为基础的讲起。

首先，来看看尼恩给大家梳理的： 常用的ES四大查询。

### 2.1 精确匹配查询（term 和 terms 查询）：

当需要对 keyword 类型 字段，或者，对不被分词的字段进行精确匹配时，使用 term 或 terms 查询。

例如，查询 用户的姓名（假设姓名存储在`user_name`字段，为 keyword 类型），使用`term`查询可以精准找到该用户对应的文档。

```
{  
  "query": {  
    "term": {  
      "user_name": "架构师尼恩"  
    }  
  }  
}
```

`term`查询是 Elasticsearch 中一种“精确匹配” 查询方式。

而且，`term`查询 主要用于对`keyword`类型的字段， 或者不希望被分词的字段进行精确匹配。

例如，在处理用户 ID、产品编号、枚举类型的值等场景时非常有用。

`term`查询 两个特点：

**特点一：不分词**：`term`查询不会对 查询关键词 进行分词处理。

`term`查询 将查询关键词 作为一个整体，直接与索引中的字段值进行比较。

比如，查询词是 “架构师 尼恩“，它就会直接在索引中查找字段值为 “架构师 尼恩” 的文档，而不会把 “架构师 尼恩” 拆分成 “架构师 ” 和 “尼恩” 分别进行匹配。

**特点二：精确匹配规则**：

`term`查询 只有当 ”查询 关键词” 与索引中的 “字段值” 完全相等时，文档才会被匹配。

`term`查询 与 模糊查询（如`match`查询）形成对比，`match`查询会对查询词进行分词，并根据分词后的结果进行模糊匹配。

### 2.2 短语查询（ match\_phrase 查询） ：

如果要搜索一个确切的短语，使用 match\_phrase 查询。 它要求查询的短语在文档中的顺序和间隔都要符合要求。

例如，查询包含"架构师尼恩" 这个短语的文档，`match_phrase`查询会找到文档中短语完整，且顺序正确的内容，如

```
{  
  "query": {  
    "match_phrase": {  
      "content": {  
        "query": "架构师尼恩",  
      }  
    }  
  }  
}  
  
# 更细致一点就是：  
  
{  
  "query": {  
    "match_phrase": {  
      "content": {  
        "query": "架构师尼恩",  
        "slop": 1  
      }  
    }  
  }  
}
```

`slop` 参数允许在 词与词之间有一定的词间隔、或者 偏移偏移。

例如，对于查询短语 "架构师尼恩"，当 `slop` 为 `1` 时，文档中的 "架构师 导师 尼恩" 也可能被匹配，因为 `slop` 允许一个词的位置偏移，即允许一个词插入在短语的词之间。

### 2.3 布尔查询（bool 查询）：

用于组合多个查询条件。可以通过`must`（必须满足）、`should`（应该满足）和`must_not`（必须不满足）来构建复杂的查询逻辑。

例如，要查找同时包含 架构师尼恩 and 技术自由圈的doc 文档，可以使用`bool`查询，如

```
{  
  "query": {  
    "bool": {  
      "must": [  
        {  
          "match": {  
            "content": "架构师尼恩"  
          }  
        },  
        {  
          "match": {  
            "content": "技术自由圈"  
          }  
        }  
      ]  
    }  
  }  
}
```

布尔查询是一种复合查询，允许你将多个查询条件组合在一起，通过must、should和must\_not子句来实现复杂的查询逻辑。

* `must`：表示必须满足的条件，类似于逻辑运算符中的 "AND"，所有的 `must` 子句中的条件都必须满足，文档才会被匹配。
* `should`：表示应该满足的条件，类似于逻辑运算符中的 "OR"，至少满足一个 `should` 子句中的条件，文档就可能被匹配。
* `must_not`：表示必须不满足的条件，类似于逻辑运算符中的 "NOT"，满足 `must_not` 子句中的条件的文档将被排除。

假设使用 es 布尔查询（bool 查询），查询含有 架构师尼恩 ，但是不含 技术自由圈的doc

为了实现这个查询，将使用bool查询的must 和must\_not子句。

* `must` 子句用于添加必须满足的条件，在这里是包含 "架构师尼恩"。
* `must_not` 子句用于添加必须不满足的条件，即不包含 "技术自由圈"。

  具体如下：

```
{  
  "query": {  
    "bool": {  
      "must": [  
        {  
          "match": {  
            "content": "架构师尼恩"  
          }  
        }  
      ],  
      "must_not": [  
        {  
          "match": {  
            "content": "技术自由圈"  
          }  
        }  
      ]  
    }  
  }  
}
```

### 2.4 match模糊查询 （这个是最难的）

match 查询是 Elasticsearch 中最常用的全文查询类型之一， 主要用于在文本字段中进行模糊搜索，会对查询词进行分词处理，然后查找包含这些分词的文档。

match 查询方式很灵活，match 模糊搜索的关键含义：不要求文档中的分词，与查询关键词的分词 顺序完全一致。

### 2.4.1 mach模糊查询工作流程

尼恩给大家画了一个 mach模糊查询工作流程, 大致如下

**第一步：分词处理**：

当执行`match`查询时，ES 会首先对查询词使用相应的分词器进行分词。

例如，对于查询词 “数据挖掘技术”，如果使用`standard`分词器，可能会被分词为 “数据”“挖掘”“技术”。

**第2步：匹配文档**：

ES 会在索引的文本字段中查找包含这些分词的文档。

对于上述例子，只要文档的文本字段中包含 “数据”“挖掘” 和 “技术” 这三个词中的一个或多个，就有可能被匹配到。

**第3步：相关性计算**

ES 会根据文档中包含的分词数量、字段长度等因素来计算相关性得分（`_score`）。

例如，一个文档中同时包含 “数据”“挖掘” 和 “技术”，其相关性得分可能会比只包含其中一个词的文档更高。

**第4步：相关性排序**：

最后，搜索结果会按照相关性得分进行排序。

相关性得分越高的文档，在搜索结果中的排名越靠前。

### 2.4.2 mach 模糊查询语法示例

假设在一个名为`article_content`的字段中进行`match`查询，查询词为 “人工智能应用”，查询语句如下：

```
GET /forum/article/_search  
{  
    "query": {  
        "match": {  
            "content": "架构师尼恩"  
        }  
    }  
}
```

**查询第1步：分词处理**：

当执行`match`查询时，ES 会首先对查询词使用相应的分词器进行分词。 例如，对于查询词 “架构师尼恩”，如果使用`standard`分词器，可能会被分词为 2个词儿，“架构师”“尼恩”。

> 这里假设我们用了自定义词库， 架构师，尼恩在词库中，分别是一个词。自定义词库的专题，稍微晚点介绍。

**查询第2步：匹配文档**：

ES 会在索引的文本字段中查找包含这些分词（“架构师”、“尼恩”）的文档。

对于上述例子，只要文档的文本字段中包含“架构师” 和 “尼恩” 这2个词中的一个或多个，就有可能被匹配到。

**问题1：匹配文档过程中，如何 控制匹配的方式？**

另外，匹配过程中，可以使用 `operator`参数 控制匹配的方式 。

在 Elasticsearch（ES）的`match`查询中，`operator`是一个重要的参数。 它决定了在进行查询时，对于分词后的查询词，文档需要满足怎样的匹配条件才能被认为是匹配的。

**`operator`取值1：`or`（默认值）**

当`operator`设置为`or`时，只要文档包含查询词分词后的任意一个词，就会被认为是匹配的。

**`operator`取值2：`and`（默认值）**

当`operator`设置为`or`时，则要求文档必须包含查询词分词后的所有词。

默认情况下，`operator`的值为`or`，这意味着只要文档包含查询词分词后的任意一个词，就会被匹配。

假设要查找 “架构师”“尼恩” 两个词都包含的文章：

```
{  
  "query": {  
    "match": {  
      "content": {  
        "query": "架构师尼恩",  
        "operator": "and"  
      }  
    }  
  }  
}
```

上述查询要求文档的content 字段必须同时包含“架构师”、“尼恩”这两个词才会被匹配。

**问题2：匹配文档过程中，如何提升字段权重（****`boost`参数）？**

使用`boost`参数来提升某个字段在查询中的重要性。

例如，在一个包含`title`和`content`两个字段的索引中，如果希望`title`字段在`match`查询中更重要，可以这样设置：

```
{  
  "query": {  
    "match": {  
      "title": {  
        "query": "架构师尼恩",  
        "boost": 2  
      },  
      "content": "人工智能应用"  
    }  
  }  
}
```

这样，在计算相关性得分时，`title`字段匹配的文档得分会更高，更有可能排在搜索结果的前面。

**第3步：相关性计算**

ES 会根据文档中包含的分词数量、字段长度等因素来计算相关性得分（`_score`）。

例如，一个文档中同时包含“架构师”、“尼恩”，或者出现的次数更多的，其相关性得分`_score`可能会比只包含其中一个词的文档更高。

**第4步：相关性排序**：

最后，搜索结果会按照相关性得分进行排序。相关性得分越高的文档，在搜索结果中的排名越靠前。

### 2.5 ES的 mach模糊查询 和match\_phrase 短语查询的区别

**第一：match 会对查询关键词进行分词，但是，不要求这些分词在doc 中 顺序与关键词中的次序一致 。**

例如，对于查询词 “数据分析方法”，通过合适的分词器（如 standard 分词器）可能会被分词为 “数据”“分析”“方法”。然后，ES 会在索引中查找包含这些分词的文档 。也就是说，只要文档中包含了这些分词中的一个或多个，就有可能被匹配到。并且，ES 会根据文档中包含的分词数量、字段长度等因素来计算相关性得分（`_score`），并按照得分对结果进行排序。

**第二：match\_phrase 也会对 查询关键词进行分词，但是， 要求这些分词在doc 中 顺序与关键词中的次序一致 。**

match\_phrase 对于同样的 “数据分析方法” 短语，也会被分词为 “数据”“分析”“方法”。

match\_phrase 要求这些分词在文档中的顺序必须和查询短语中的顺序一致，并且分词之间的距离（可以通过`slop`参数来调整）也有一定限制。

默认情况下，分词之间的距离 `slop`为 0，这意味着分词后的词在文档中必须是连续出现的。比如， “数据分析方法” 需要在 doc中连续出现，表现出来，就是这些分词 是一个 完整的短语一样。

特殊情况下，分词之间的距离 `slop`为 >1，那么在一定程度上允许词与词之间插入其他词。例如，`slop`为 1 时，“数据相关的分析方法” 可能会被匹配到。

**第三： match 查询 匹配的严格程度 相对比较宽松**

match 查询 匹配的严格程度 相对比较宽松, 而不严格要求分词出现的 顺序。

例如，对于查询词 “架构师 尼恩””，如果文档中有 “尼恩 是一个45岁的 资深老 架构师 ”，也 会被匹配到。很明显

match 查询 不关注 分词出现的次序， 只关注分词后的词是否出现在文档中。

**第四：match\_phrase 非常严格，注重短语的完整性和顺序。**

只有当顺序和短语完全一致, 或者通过调整`slop`参数允许一定的 间隔时，才会被匹配。

例如，对于查询词 “架构师 尼恩””，如果文档中有 “尼恩 是一个45岁的 资深老 架构师 ”，就不 会被匹配到。

再如，对于查询词 “架构师 尼恩””，如果文档中有 “架构师 尼恩是一个45岁的 资深老 架构师 ”，就 会被匹配到。

再如，对于查询词 “架构师 尼恩””，如果文档中有 “架构师 老 尼恩是一个45岁的 资深老 架构师 ”，如果slop为1，也 会被匹配到。

相对来说， match 查询 不关注 分词出现的次序， 只关注分词后的词是否出现在文档中。 而 match\_phrase 非常严格，注重短语的完整性和顺序

**第五：match 、match\_phrase 查询应用场景**

* match 适用于用户以自然语言进行搜索，不太关注具体的词序的情况。

例如，在一个知识问答平台的文档搜索中，用户输入 “如何提高工作效率和时间管理”，match 查询可以找到包含 “提高工作效率” 和 “时间管理” 相关内容的文档，即使文档中这两个内容的顺序与用户输入不同。

* match\_phrase 查询应用场景

适合用于精确查找某个特定的短语。比如在学术文献搜索中，当用户想要查找包含特定研究方法相关短语（如 “实验设计方法”）的文献时，match\_phrase 查询能够精准地找到文档中出现这个完整短语，并且顺序正确的文献。

### 2.6 match\_phrase 短语查询与 term 精确匹配查询 的区别

短语查询（match\_phrase） 是用于查找包含确切短语的文档。

短语查询（match\_phrase） 会对查询关键词进行分词处理， 同时会要求查询的短语在文档中的顺序和间隔都要符合要求。

短语查询（match\_phrase）匹配的时候，还考虑词与词之间的距离。 词之间允许有一定的 “滑动窗口”， 通过距离参数（如`slop`参数）配置。`slop`参数用于控制词之间允许的最大距离，通过调整`slop`可以更灵活地控制短语匹配的严格程度。

精确匹配查询（term ） 用于对 keyword 类型的字段或者不希望被分词的字段进行精确匹配。

精确匹配查询（term ） 是完全按照给定的词条进行匹配，不会对查询关键词进行分词处理。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 3 查询构建 层面 的 精准度优化

### 3.1 选择合适的查询类型

**精确匹配查询（term 和 terms 查询）：**

当需要对 keyword 类型的字段或者不希望被分词的字段进行精确匹配时，使用 term 或 terms 查询。

例如，查询某个用户的姓名（假设姓名存储在`user_name`字段，为 keyword 类型），使用`term`查询可以精准找到该用户对应的文档。

如：

```
{  
  "term": {  
    "user_name": "张三"  
  }  
}
```

**短语查询（match\_phrase 查询）**：

如果要搜索一个确切的短语，使用 match\_phrase 查询。

它要求查询的短语在文档中的顺序和间隔都要符合要求。

例如，查询包含 “大数据分析” 这个短语的文档，`match_phrase`查询会找到文档中短语完整且顺序正确的内容，如

```
{  
  "match_phrase": {  
    "content": "大数据分析"  
  }  
}
```

**布尔查询（bool 查询）**：

用于组合多个查询条件， 可以通过`must`（必须满足）、`should`（应该满足）和`must_not`（必须不满足）来构建复杂的查询逻辑。

例如，要查找同时包含 “人工智能” 并且不包含 “深度学习框架” 的文档，可以使用`bool`查询，如

```
{  
  "bool": {  
    "must": [  
      {  
        "match": {  
          "content": "人工智能"  
        }  
      }  
    ],  
    "must_not": [  
      {  
        "match": {  
          "content": "深度学习框架"  
        }  
      }  
    ]  
  }  
}
```

### 3.2 合理 设置查询参数

**提升重要字段权重（boost）**：

对于某些在搜索结果中更重要的字段，可以通过设置`boost`参数来提高其权重。

例如，在一个商品搜索系统中，商品标题和品牌名称对于精准匹配可能更重要，在查询时可以给这些字段更高的权重。

如

```
{  
  "query": {  
    "multi_match": {  
      "query": "手机",  
      "fields": [  
        "product_title^3",  
        "brand^2",  
        "description"  
      ]  
    }  
  }  
}
```

这里`product_title`字段的权重是 3，`brand`字段权重是 2，`description`字段权重默认为 1。

**控制模糊度（fuzziness）**：

在允许模糊查询的情况下，合理设置模糊度参数。

如果模糊度过高，可能会返回大量不相关的结果；

模糊度过低，则可能错过一些合理的拼写错误等情况的匹配。

例如，对于一个允许用户输入拼写可能有误的搜索词的场景，设置`fuzziness`为 1 或 2（根据具体情况），如

```
{  
  "fuzzy": {  
    "product_name": {  
      "value": "aplle",  
      "fuzziness": 1  
    }  
  }  
}
```

来查找可能是 “apple” 的拼写错误的产品名称。

## 4. 索引设置层面 的 精准度优化

## 4.1 分词器选择：根据搜索场景，选择合适的内置分词器：

ES 提供了多种内置分词器，如whitespace 分词器、 standard 分词器等。

除此之外，还有很多 第三方的开源分词器插件，如尼恩用过的 IK 分词器、结巴分词器等等。

不同的搜索场景，，选择合适的内置分词器。

例如，对于简单的以空格分隔的文本搜索场景，whitespace 分词器可能就足够了，它在分词时仅以空格作为分隔符，不会进行一些如小写转换等其他操作，这样可以保持原始文本的一些特性用于精准搜索。

但是，如果不区分大小写，去除大部分标点符号，那么可以使用 standard 分词器。

standard 标准分词器是 ES 的默认分词器，该分词器会将文本转换为小写，去除大部分标点符号，同时会处理一些基本的字符过滤。

```
{  
  "query": {  
    "match": {  
      "content": "技术自由圈"  
    }  
  }  
}
```

standard 对于中文，它会将中文文本按字进行拆分。例如，对于文本 “技术自由圈”，它会拆分成 “技” “术” “自” “由” “圈” 5个词儿。

所以，中文场景，一般使用 第三方的开源分词器插件，如尼恩用过的 IK 分词器。

IK 分词器是一个开源的中文分词器，具有更强大的中文分词能力。

IK 分词器 支持两种分词模式：`ik_max_word` 和 `ik_smart`。

* `ik_max_word` 细颗粒分词： 会将文本尽可能拆分成最细粒度的字 和词，例如，对于 “我爱中国”，可能会拆分成 “我”“爱”“中国”“我爱” 等，以覆盖多种可能的词汇组合。
* `ik_smart` 粗颗粒拆分：会将文本拆分成粗粒度的，比较最合理的词，例如，对于 “我爱中国”，可能会拆分成 “我爱”“中国”。

```
{  
  "query": {  
    "match": {  
      "content": {  
        "query": "我爱中国",  
        "analyzer": "ik_max_word"  
      }  
    }  
  }  
}
```

在选择分词器时，可以使用\_analyze 这个 API 接口测试分词效果，例如：

```
{  
  "analyzer": "ik_max_word",  
  "text": "我爱中国"  
}
```

通过这个 API 接口可以查看不同分词器对一段话的分词结果，帮助大家更好地选择适合的分词器。

## 4.2 细颗粒索引，粗颗粒搜索：

尼恩团队曾经在维护一个 30个节点的大型ES集群过程中， 通过实践总结出来的一个 提高搜索的召回率和精准度的经验： 细颗粒索引，粗颗粒搜索。

**什么是 细颗粒索引？**：

在对文档进行索引时，将文档中的字词尽可能细颗粒拆分，尽可能的 分成更细小的单元。

例如，对于一篇文章，将其拆分成多个段落、句子，甚至对词语进行细致的分词处理。

例如，对于 “我爱中国”，可能会拆分成 “我”“爱”“中国”“我爱” 等，以覆盖多种可能的词汇组合。

这样做的目的是为了在搜索时能够更精确地匹配各种可能的查询词，提高搜索的召回率和精准度。

通常使用更细颗粒 的分词器（如 IK 分词器的 `ik_max_word` 模式）将doc 内容进行细致分词 。

**什么是 粗颗粒搜索?**：

指在查询时， 将搜索关键词尽可能粗颗粒拆分，尽可能的 分成更粗颗的单元。

例如，对于 “我爱中国”，可能会拆分成 “我爱”“中国” 两个词儿进行搜索，这样搜索的结果会 更高的精准度，不会出现很多低相关度的 doc。

反过来，如果拆分为 “我””爱”“中””国” 四个字儿去搜，那么会把 包含“我”的doc全部召回了， 查到上百万的doc，实际没啥用。

通常使用更粗颗粒 的分词器（如 IK 分词器的 `ik_smart`模式）将 搜索关键词进行粗颗粒分词 。

**如何实现 细颗粒索引？**：

根据这个原则，尼恩团队在设计索引的时候，使用具有细粒度 分词器， 设置为 IK 分词器为 `ik_max_word` 模式：

```
{  
  "settings": {  
    "analysis": {  
      "analyzer": {  
        "ik_max_word_analyzer": {  
          "type": "custom",  
          "tokenizer": "ik_max_word"  
        }  
      }  
    }  
  },  
  "mappings": {  
    "properties": {  
      "content": {  
        "type": "text",  
        "analyzer": "ik_max_word_analyzer"  
      }  
    }  
  }  
}
```

这里使用 `ik_max_word` 分词器对 `content` 字段进行细粒度分词，将文本拆分成尽可能多的词汇组合。

**如何实现 粗颗粒搜索？**：

进行查询时，也可以指定使用 `ik_smart` 模式进行分词，以下是一个使用 `match` 查询的示例：

```
{  
  "query": {  
    "match": {  
      "content": {  
        "query": "我爱中国",  
        "analyzer": "ik_smart"  
      }  
    }  
  }  
}
```

`ik_smart` 模式它产生的分词较少, 可以减少 搜索时的计算量，也提升搜索的精确度。

当然，可能会导致一些更细致的搜索需求无法满足，这就是需要再代码层面进行 组合模式 实现更高层次的 精准度优化。

## 4.3 自定义词库：定义一套 业务专用词库

**根据业务需求自定义词库, 可以大大提高分词的准确性和搜索的准确性。**

**什么是自定义词库？**

自定义词库是指在分词过程中，添加自定义的词汇列表，以满足特定领域或业务需求的分词要求。

这对于一些未被标准分词器或现有分词器覆盖的专业术语、新词汇或特定领域的词汇非常有用，可以提高分词的准确性和搜索的召回率。

对于特定领域的术语，可以创建一个包含该领域词汇表 ，以确保这些术语在索引和搜索时能被正确处理。

如果是医疗领域的文档搜索，需要将医学术语词典加入自定义词库，使医学术语能够精准地被索引和查询。

**如何 使用 IK 分词器的自定义词库？**

首先，确安装并配置了 IK 分词器插件。

可以在 IK 分词器的配置目录中添加自定义的词典文件，通常命名为 `my_dict.dic` 或其他自定义名称。

例如，将该文件放在 IK 分词器的 `config` 目录下。

**添加自定义词汇**：

在 `my_dict.dic` 文件中，添加自定义的词汇，每个词汇占一行。例如：

```
技术自由圈  
架构师尼恩  
大数据分析  
区块链技术  
人工智能算法
```

这些词汇将被 IK 分词器视为一个完整的词项，而不是被拆分成更小的词。

**配置 IK 分词器使用自定义词库**：

找到 IK 分词器的 `IKAnalyzer.cfg.xml` 文件，在 `<properties>` 部分添加自定义词典文件的配置，如下：

```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
    <comment>IK Analyzer 扩展配置</comment>  
    <entry key="ext_dict">my_dict.dic</entry>  
    <!-- 可以添加更多自定义词典文件，使用分号分隔 -->  
</properties>
```

这样，IK 分词器在进行分词时会将自定义词库中的词汇作为一个完整的词项处理。

通过自定义词库，实现 优化搜索结果的目的。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 4.4 自定义停词：定义一套 业务专用 停用词

**根据业务需求自定义停用词 可以大大提高分词的准确性和搜索的准确性。**

停用词是指在文本处理过程中，被认为是没有实际意义或 可以被忽略的词汇的词儿。

例如 “的”“是”“在”“和” 等。

需要 根据具体的业务场景和语言环境，指定哪些词汇应被视为停用词，从而优化搜索结果和索引性能。

**如何 使用 IK 分词器的自定义停用词词库？**

修改 IK 分词器的配置文件, 在 IK 分词器的配置文件（通常是 `IKAnalyzer.cfg.xml`）中，找到 `<entry key="ext_stopwords">`元素，添加自定义的停用词文件，例如：

```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
<properties>  
    <comment>IK Analyzer 扩展配置</comment>  
    <entry key="ext_stopwords">my_stopwords.dic</entry>  
</properties>
```

这里 `my_stopwords.dic` 是一个自定义的停用词文件。

**添加自定义停用词文件**：

在 IK 分词器的配置目录中创建 `my_stopwords.dic` 文件，添加自定义的停用词，每个停用词占一行，例如：

```
一些  
一个  
该
```

重启es可以测试自定义停用词，使用 \_analyze API 来测试自定义停用词的效果，以使用内置分词器自定义停用词的示例为例：

```
{  
  "analyzer": "my_custom_analyzer",  
  "text": "这是一个关于大数据分析的文章"  
}
```

发送上述请求， 会发现自定义的停用词（如 “是”“一个”“的”）在分词结果中被过滤掉。

通过自定义停用词，实现 优化搜索结果的目的。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 4.5 合理设计字段类型：对于精确匹配的字段，定义为 keyword

合理设计字段类型：精确匹配的字段，定义为 keyword 。

例如，如果一个字段主要用于精确匹配（如用户 ID、产品编号等），将其定义为 keyword 类型而不是 text 类型。

因为 keyword 类型不会进行分词，在匹配时是： **完全匹配**，更精准。

例如，一个产品文档中有`product_id`字段，定义为`keyword`类型后，搜索`product_id`为`12345`时就会精准匹配到该产品，而不会出现由于分词等情况导致的误匹配。

## 4.6 数据清洗 预处理，提前去除噪声数据：

在将数据导入 ES 之前，清理掉无用的、错误的或不相关的数据。

例如，对于文本数据，去除 HTML 标签、多余的空格和特殊字符等。

如果是网页内容作为文档存储，其中的广告代码部分（HTML 标签和相关内容）就可以在预处理阶段去除，避免这些无关内容干扰搜索结果。

## 4.7 标准化数据格式：

统一日期格式、数字格式等。

例如，所有日期字段都采用`yyyy - MM - dd`的格式，这样在进行日期范围搜索时就不会因为格式不一致而出现遗漏或错误匹配的情况。

## 4.8 精细的字段映射,为同一个字段创建多个映射：

对于同一个字段的内容，根据不同的业务处理，进行不同的字段映射。

有时，对于同一个字段的内容，由于不同的业务需求，我们希望以不同的方式对其进行索引和搜索。

例如，对于一个包含文本内容的字段，在某些情况下我们可能需要进行全文搜索，在另一些情况下可能需要进行精确匹配，或者在不同的搜索场景下使用不同的分析器进行分词处理。

通过为同一个字段创建多个映射，可以实现这些不同的业务需求。

多字段映射允许为同一个字段创建多个子字段，每个子字段具有不同的映射属性，以满足不同的业务处理需求。以下是一个示例：

```
{  
  "mappings": {  
    "properties": {  
      "content": {  
        "type": "text",  
        "fields": {  
          "keyword": {  
            "type": "keyword",  
            "ignore_above": 256  
          },  
          "ik_smart": {  
            "type": "text",  
            "analyzer": "ik_smart"  
          },  
          "ik_max_word": {  
            "type": "text",  
            "analyzer": "ik_max_word"  
          }  
        }  
      }  
    }  
  }  
}
```

在这个示例中：

* `content` 是原始字段，类型为 `text`，用于全文本搜索。
* `content.keyword` 是 `content` 字段的一个子字段，类型为 `keyword`，可用于精确匹配，并且 `ignore_above` 参数表示当字段长度超过 256 时不进行索引。
* `content.ik_smart` 是另一个子字段，使用 `ik_smart` 分析器进行分词，适用于粗颗粒的搜索。
* `content.ik_max_word` 是另一个子字段，使用 `ik_max_word` 分析器进行分词，适用于细颗粒的搜索。

查询时，可以根据具体的业务需求选择不同的子字段进行操作。

以下是几个示例：

**1 使用 keyword 子字段进行精确匹配**：

```
{  
  "query": {  
    "term": {  
      "content.keyword": "精确的关键词"  
    }  
  }  
}
```

这个查询使用 `content.keyword` 子字段进行精确匹配，适用于查找精确匹配的文档。

**2 使用 ik\_smart 子字段进行粗颗粒搜索**：

```
{  
  "query": {  
    "match": {  
      "content.ik_smart": "搜索关键词"  
    }  
  }  
}
```

这里使用 `content.ik_smart` 子字段进行粗颗粒搜索，使用 `ik_smart` 分析器对查询词进行分词。

**3 使用 ik\_max\_word 子字段进行细颗粒搜索**：

```
{  
  "query": {  
    "match": {  
      "content.ik_max_word": "搜索关键词"  
    }  
  }  
}
```

这里使用 `content.ik_max_word` 子字段进行细颗粒搜索，使用 `ik_max_word` 分析器对查询词进行分词。

**多字段映射的优势：**

通过为 `content` 字段创建多个子字段，可以灵活地根据不同的业务需求选择不同的搜索和索引方式。

例如，在一个新闻文章的搜索系统中：

* 当用户输入精确的文章标题进行查找时，可以使用 `content.keyword` 子字段进行精确匹配。
* 当用户进行模糊搜索时，可以使用 `content.ik_smart` 或 `content.ik_max_word` 子字段，根据不同的粒度需求进行搜索。

**多字段映射的不足**

多字段映射会增加索引的复杂性和大小，因为会存储多个不同映射的字段数据。

在创建索引时，要根据实际的业务需求和数据量来平衡性能和功能。

## 4.9 通过动态模板 ，优化索引映射（mappings）

**定义动态模板**：通过动态模板可以根据数据的类型自动设置字段的映射。

例如，对于新加入的字符串字段，如果是以数字为主的字符串（如产品编号等），可以自动将其映射为`keyword`类型，

如

```
{  
  "mappings": {  
    "dynamic_templates": [  
      {  
        "string_to_keyword_template": {  
          "match_mapping_type": "string",  
          "match": "^[0-9]+.*",  
          "mapping": {  
            "type": "keyword"  
          }  
        }  
      }  
    ]  
  }  
}
```

在上述示例中：

* `"string_to_keyword_template"` 是动态模板的名称，可以根据需要自行命名。
* `"match_mapping_type": "string"`表示匹配字符串类型的字段。
* `"match": "^[0-9]+.*"`是一个正则表达式，用于匹配以数字开头的字符串。这里的`^`表示字符串的开头，`[0-9]+`表示至少一个数字，`.*`表示任意数量的任意字符。`.*\\d+.` 也可以匹配以数字开头的字符串，和 `^[0-9]+.*` 效果差不多。
* `"mapping": {"type": "keyword"}`表示将匹配到的字段映射为`keyword`类型。

通过这种方式，当向 Elasticsearch 中插入新的文档时，如果遇到以数字开头的字符串字段，就会自动按照`keyword`类型进行映射，而不需要在每次插入文档时都手动指定字段的映射类型。

这有助于提高数据的索引效率和查询准确性，特别是在处理大量具有特定格式的字符串字段时，可以大大减少手动配置映射的工作量。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 5. 搜索结果层面 的 精准度优化

## 5.1 使用explain分析相关性评分（\_score）

ES 会为每个搜索结果计算一个相关性评分。分析这些评分，查看高评分和低评分的结果是否符合预期。

如果高评分结果中有不相关的文档，或者相关文档评分较低，就需要调整查询策略。

例如，通过查看搜索结果的`_score`字段，发现一个不太相关的文档得分很高，可能是因为某些字段的权重设置不合理，需要重新调整`boost`参数。

**查看评分明细**：

可以使用 ES 的`explain`参数来查看相关性得分的计算明细。

例如，在搜索请求中添加`"explain": true`，ES 会返回每个文档的详细评分解释，包括每个查询条件对评分的贡献等。

通过分析这些明细，可以准确找出导致评分异常的原因，如是否存在某个字段的权重设置过高或过低，或者某个词的词频计算不符合预期等。

以下是一个使用 Elasticsearch 的 `explain` 参数查看相关性得分计算明细的具体例子.

假设我们有一个简单的文章索引，索引中包含 `title`（文章标题）和 `content`（文章内容）两个字段，现在要搜索包含关键词 “人工智能” 的文章，并查看评分明细。

第一步：首先创建索引, 插入示例文档. 创建索引的请求（使用 `PUT` 方法）：

```
PUT /article_index  
{  
  "mappings": {  
    "properties": {  
      "title": {  
        "type": "text"  
      },  
      "content": {  
        "type": "text"  
      }  
    }  
  }  
}
```

插入两篇示例文章文档， 插入第一篇：

```
POST /article_index/_doc  
{  
  "title": "人工智能的发展趋势",  
  "content": "近年来，人工智能在诸多领域取得了显著进展，它正改变着我们的生活方式。"  
}
```

插入两篇示例文章文档， 插入第二篇：

```
POST /article_index/_doc  
{  
  "title": "科技前沿探索",  
  "content": "除了人工智能，还有很多其他前沿科技值得关注，比如量子计算等。"  
}
```

**第2步：发起带****`explain` 参数的搜索请求**

使用如下的 `GET` 请求来搜索包含 “人工智能” 的文章，并添加 `explain` 参数为 `true`：

```
GET /article_index/_search  
{  
  "query": {  
    "match": {  
      "content": "人工智能"  
    }  
  },  
  "explain": true  
}
```

注意，这里 `explain` 参数为 `true`.

**第3步：查看返回的评分明细**

返回的结果可能类似如下（为了便于理解，进行了简化和重点展示）。第一篇文章（标题为 “人工智能的发展趋势” 的那篇）的 explain 介绍：

```
{  
  "_index": "article_index",  
  "_type": "_doc",  
  "_id": "1",  //文档的实际ID  
  "_score": 0.57735026（这里是相关性得分，实际值根据具体计算情况而定）,  
  "_source": {  
    "title": "人工智能的发展趋势",  
    "content": "近年来，人工智能在诸多领域取得了显著进展，它正改变着我们的生活方式。"  
  },  
  "explain": {  
    "value": 0.57735026,  
    "description": "weight(content:人工智能 in 0) [PerFieldSimilarity], result of:",  
    "details": [  
      {  
        "value": 0.57735026,  
        "description": "fieldWeight in 0, product of:",  
        "details": [  
          {  
            "value": 1.4142135（词频相关值，此处‘人工智能’在内容中出现1次等情况对应的计算值）,  
            "description": "tf(freq=1.0), with freq of: 1.0"  
          },  
          {  
            "value": 1.7320508（逆向文档频率相关值，说明‘人工智能’这个词在整个索引文档中的独特性情况对应的计算值）,  
            "description": "idf(docFreq=1, maxDocs=2)"  
          },  
          {  
            "value": 0.25（字段长度正则化相关值，和content字段长度等因素有关的计算值）,  
            "description": "tfnorm, computed from:",  
            "details": [  
              {  
                "value": 2.0（比如可能和该字段包含的词的总数量等相关）,  
                "description": "len=2"  
              }  
            ]  
          }  
        ]  
      }  
    ]  
  }  
}
```

对于第二篇文章（标题为 “科技前沿探索” 的那篇）：

```
{  
  "_index": "article_index",  
  "_type": "_doc",  
  "_id": "文档的实际ID2",  
  "_score": 0.28867513（相关性得分）,  
  "_source": {  
    "title": "科技前沿探索",  
    "content": "除了人工智能，还有很多其他前沿科技值得关注，比如量子计算等。"  
  },  
  "explain": {  
    "value": 0.28867513,  
    "description": "weight(content:人工智能 in 0) [PerFieldSimilarity], result of:",  
    "details": [  
      {  
        "value": 0.28867513,  
        "description": "fieldWeight in 0, product of:",  
        "details": [  
          {  
            "value": 0.70710678（词频相关值，‘人工智能’在这篇文章内容里也出现了1次，但可能受其他因素影响计算不同）,  
            "description": "tf(freq=1.0)"  
          },  
          {  
            "value": 1.7320508（逆向文档频率相关值，和第一篇文章一样，因为整个索引中‘人工智能’的文档频率没变）,  
            "description": "idf(docFreq=1, maxDocs=2)"  
          },  
          {  
            "value": 0.25（字段长度正则化相关值，和content字段长度等因素有关的计算值）,  
            "description": "tfnorm, computed from:",  
            "details": [  
              {  
                "value": 8.0（比如这篇文章content字段包含词的总数量更多等情况导致不同）,  
                "description": "len=8"  
              }  
            ]  
          }  
        ]  
      }  
    ]  
  }  
}
```

从上述明细可以看出：

**词频影响**（TF）：两篇文章中 “人工智能” 在 `content` 字段出现次数都是 1 次，但由于不同文章 `content` 字段本身包含的总词数不同（第一篇 2 个词，第二篇 8 个词），导致词频计算出来的值对最终得分的影响有差异（第一篇 `tf` 值为 `1.4142135`，第二篇为 `0.70710678`）。

**逆向文档频率影响**：两篇文章中 “人工智能” 这个词的逆向文档频率值相同（`idf` 值都是 `1.7320508`），因为在整个索引的两篇文档里，包含 “人工智能” 这个词的文档只有 1 篇，所以其在索引中的独特性情况是一样的。

**字段长度正则化影响**：两篇文章 `content` 字段长度不同，使得最终在计算相关性得分时，通过 `tfnorm` 这个因素体现出了差异，进而影响了整体的相关性得分。

结论是，第二篇文章相关性得分应该更高一些，因为第二篇文章比较短。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

**深入理解相关性评分算法**：

Elasticsearch 默认在 5.0 版本及以后采用 BM25 算法计算相关性评分，它综合考虑了词频（TF）、逆向文档频率（IDF）、字段长度正则化、查询规范因子等多种因素。

接下来，尼恩不得不把当年困扰过自己的 几个名字，给大家稍微梳理一下，避免大家走尼恩的弯路。

#### 什么是词频（Term Frequency, TF）？

词频是指某个词在文档中出现的频率。

它表示一个词在单个文档中出现的次数。

通常情况下，一个词在文档中出现的次数越多，这个词对于该文档的重要性可能就越高。

词频（Term Frequency, TF） 计算方式：

简单的词频计算可以直接是某个词在文档中出现的次数。例如，在文档 "I love to love programming" 中，词 "love" 的词频是 2。

更常见的是进行归一化处理，以避免长文档中出现次数多的词被过度加权。

一种常见的归一化公式是：`TF(t, d) = (词 t 在文档 d 中出现的次数) / (文档 d 中词的总数)`。

在搜索中的应用：当用户搜索一个词时，包含该词多次的文档通常被认为与该词更相关。

例如，在一个搜索引擎中，用户搜索 "apple"，一个文档中多次提及 "apple" 可能比只提及一次的文档更可能是用户想要的结果，因为多次提及表明该文档可能更侧重于 "apple" 这个主题。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

#### 什么是逆向文档频率（Inverse Document Frequency, IDF）？

逆向文档频率衡量一个词在整个文档集合中的普遍程度。

如果一个词在很多文档中都出现，那么它对于区分不同的文档可能帮助不大，因此重要性较低；

反之，如果一个词仅在少数文档中出现，它可能是一个更具代表性的词，重要性较高。

计算方式：`IDF(t) = log( (总文档数) / (包含词 t 的文档数 + 1) )`，这里加 1 是为了避免除数为 0 的情况。

例如，如果总共有 100 个文档，词 "the" 出现在 90 个文档中，那么 `IDF("the") = log(100 / (90 + 1))`，结果是一个较小的值，说明 "the" 是一个很常见的词，重要性低。

而对于一个比较专业的词，如 "neural network"，如果只出现在 5 个文档中，`IDF("neural network") = log(100 / (5 + 1))`，结果会较大，表明其重要性较高。

在搜索中的应用：IDF 帮助搜索引擎区分重要和不重要的词。

在计算文档相关性时，对于一些通用词，由于其高出现频率，它们的 IDF 值较低，对得分的贡献相对较小；而对于罕见词，其 IDF 值较高，对得分的贡献更大。

#### 什么是字段长度正则化？

字段长度正则化考虑了文档中字段的长度对相关性的影响。

通常，在一个较短的字段中出现的词可能比在一个较长的字段中出现的相同词更重要，因为它在短字段中占的比重更大。

计算方式：不同的搜索引擎和信息检索系统可能有不同的计算方法，但一个常见的计算方式是：`fieldLengthNorm = 1 / sqrt(字段中的词数)`。

例如，对于一个包含 4 个词的字段，其字段长度正则化值为 `1 / sqrt(4) = 0.5`；对于一个包含 16 个词的字段，其字段长度正则化值为 `1 / sqrt(16) = 0.25`。

在搜索中的应用：当用户搜索一个词时，出现在短字段中的该词会被认为更重要。

例如，在搜索 "apple" 时，如果 "apple" 出现在一个简短的文档标题中，比出现在一个长篇大论的内容字段中的相同词可能更具相关性 。

所以， 从这个角度来说，大家写的博客越短，字段长度正则化的值越大， 搜索引擎会越排在前面。

看起来，这个规则对尼恩的文章是非常不利的， 尼恩的文章很多都是 5w字以上， 很长很长的文章。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 5.2 使用boost 参数调整相关性评分（\_score）

然而，我们发现， 第一篇才是我们想要的， 如何调整呢？

可以让 title也参与搜索，并且通过 `boost` 参数等方式，适当提高 `title` 字段的权重。

```
GET /article_index/_search  
{  
  "query": {  
    "bool": {  
      "should": [  
        {  
          "match": {  
            "title": {  
              "query": "人工智能",  
              "boost": 2.0  // 对 title 字段进行 2 倍加权  
            }  
          }  
        },  
        {  
          "match": {  
            "content": "人工智能"  
          }  
        }  
      ]  
    }  
  }  
}
```

这里包含两个 `match`

第一个 `match` 查询是针对 `title` 字段的，并且使用了 `boost` 参数将该字段的权重设置为 2.0。

这意味着如果文档的 `title` 字段中出现了 "人工智能" 这个词，其对 `_score` 的贡献会乘以 2.0。

第二个 `match` 查询是针对 `content` 字段的，没有设置 `boost` 参数，所以其对 `_score` 的贡献是默认权重。

在 Elasticsearch（ES）中，`boost` 是一个非常重要的参数，用于调整查询中不同部分的相对权重，从而影响搜索结果的相关性得分（`_score`）。

`boost` 可以应用于多种查询类型，如 `match`、`term`、`bool` 等，用于修改这些查询对最终文档得分的贡献程度。

`boost` 是一个浮点数，可以是大于 0 的任意值。

* 当 `boost` 值为 1.0 时，不会对查询的得分产生影响；
* 当 `boost` 值大于 1.0 时，会增加查询的权重，使匹配该查询的文档得分更高；
* 当 `boost` 值小于 1.0 且大于 0 时，会降低查询的权重，使匹配该查询的文档得分降低。

## 5.3 使用 function\_score 调整相关性评分（\_score）

如果发现  `boost` 参数不够用， 也就是高评分结果不相关、或相关文档评分低时，还可以尝试其他方法，进行 score 的调整。

其中 一种方法是使用`function_score`进行 score 的调整。

比如上面的例子，可以 通过 `function_score`等方式 进行 score 的调整

例如，对于多词查询， 使用`function_score`查询来根据特定的业务规则重新调整得分，如对最新发布的文档给予一定的加分，或者对热门文档进行加权等。

以下是通过 `function_score` 来调整 Elasticsearch 搜索结果 `_score`的示例：

1. 使用 `function_score` 查询，将多个评分函数组合起来，以调整搜索结果的得分。
2. 可以使用内置的评分函数，如 `field_value_factor` 来根据文档中的某个字段的值来调整得分，或者使用 `weight` 函数给查询条件分配不同的权重。
3. 也可以使用自定义的评分函数，根据自己的需求对得分进行调整。

假设上面的例子中， 两篇文档，第1个文章的 查看次数为 100

```
{  
  "title": "人工智能的发展趋势",  
  "content": "近年来，人工智能在诸多领域取得了显著进展，它正改变着我们的生活方式。",  
  "views": 100  
}
```

第二个文章的 查看次数为 50

```
{  
  "title": "科技前沿探索",  
  "content": "除了人工智能，还有很多其他前沿科技值得关注，比如量子计算等。",  
  "views": 50  
}
```

可以 把查看次数多的 ， 得分 高些， 通过  `function_score` 来加分：

```
GET /article_index/_search  
{  
  "query": {  
    "function_score": {  
      "query": {  
        "bool": {  
          "should": [  
            {  
              "match": {  
                "title": "人工智能"  
              }  
            },  
            {  
              "match": {  
                "content": "人工智能"  
              }  
            }  
          ]  
        }  
      },  
      "functions": [  
        {  
          "field_value_factor": {  
            "field": "views",  // 根据 views 字段的值来调整得分，假设 views 表示文章的浏览量  
            "modifier": "log1p",  // 使用 log1p 修饰符对 views 字段的值进行转换  
            "factor": 0.1  // 转换后的 views 字段值乘以 0.1 作为得分的一部分  
          }  
        },  
        {  
          "weight": 2.0  // 对查询条件整体加权，可根据需要调整  
        }  
      ],  
      "score_mode": "sum",  // 计算得分的模式，这里使用求和的方式  
      "boost_mode": "multiply"  // 最终得分的计算方式，这里使用乘法  
    }  
  }  
}
```

当执行上述 `function_score` 查询时：

对于第1个文章，首先根据 `bool` 查询计算原始得分，然后 `field_value_factor` 函数会根据 `views` 字段的值（这里是 100）计算一部分得分（约 0.461），`weight` 函数会将原始得分乘以 2.0，最后将这些得分根据 `score_mode` 相加，再根据 `boost_mode` 与原始得分相乘得到最终得分。

对于第2个文章，同样的逻辑，但由于 `views` 字段的值为 50，其 `field_value_factor` 函数计算的得分会不同，最终得分也会不同。

#### functions解释：

functions 可以包含多个评分函数，每一个评分函数包括下面的配置：

* `field`：根据目标字段（如 `views` 字段）的值调整得分。
* `modifier` 是对 `views` 字段的值进行转换的函数，这里使用 `log1p` 函数将 `views` 字段的值加 1 后取自然对数。
* `factor` 是一个乘数，将转换后的值乘以这个因子作为最终得分的一部分。

例如，如果 `views` 字段的值为 100，经过 `log1p` 转换后的值约为 4.61，乘以 `factor` 0.1 得到 0.461。

再看 functions 的参数设置：

* `weight`：为整个查询条件分配一个权重，这里设置为 2.0，意味着原始查询结果的得分会先乘以 2.0。
* `score_mode`：functions 多个函数之间的 得分的计算模式，这里是 `sum`，表示将所有函数的得分相加。
* `boost_mode`：functions 和 最终得分的计算方式，这里是 `multiply`，表示将 `functions` 计算的得分乘以 `query` 的得分。

> 备注：以上内容比较复杂， 如果看不懂的，后面可以看《尼恩面试宝典》 配套视频

## 5.4 人工评估和用户反馈：

人工检查搜索结果的准确性，同时收集用户反馈。用户可能会发现一些搜索结果不符合期望的情况，根据这些反馈来优化查询和索引。

**建立人工评估机制**：

组织专门的人员定期对搜索结果进行人工检查，制定明确的评估标准，如准确性、完整性、相关性等。评估人员可以根据这些标准对搜索结果进行打分或标注，记录下存在问题的搜索结果和具体问题描述。

例如，对于一个电商搜索系统，可以检查搜索结果中的商品是否与用户搜索词高度相关，商品信息是否完整准确等。

**多渠道收集用户反馈**：

除了用户主动反馈外，还可以通过多种渠道收集反馈。例如，在搜索页面设置反馈按钮，方便用户随时提交反馈；或者在用户完成搜索后，通过弹出问卷的方式询问用户对搜索结果的满意度和改进建议；还可以从用户的行为数据中挖掘潜在的问题，如用户频繁点击搜索结果中的某个无关文档后又重新搜索，可能意味着搜索结果不理想。

## 5.5 搜索转化率的计算和分析：

这也是尼恩团队之前干过的一个重点工作。

搜索转化率是指用户进行搜索操作后，完成了预期转化行为（如点击搜索结果、购买商品、提交表单等）的用户数量与总搜索用户数量的比值。

搜索转化率越高，说明搜索的效果越好，准确度越高。

搜索转化率的计算和分析：

* 需要收集用户的搜索行为数据，通常存储在一个索引中，包含用户进行搜索的信息，如用户 ID、搜索关键词、搜索时间等。
* 还需要收集用户的转化行为数据，存储在另一个索引或同一索引的不同部分，包含用户 ID、转化行为类型（如点击、购买等）、转化时间等。

**关联搜索行为和转化行为**：通过用户 ID 等信息将用户的搜索行为和转化行为关联起来，找出进行了搜索且发生转化的用户。

然后进行对比分析，这个和业务有关系了，尼恩在这里不做赘述了。 一般的电商搜索，肯定是天天进行**搜索转化率的计算和分析**的。

**根据反馈优化查询和索引**：

对收集到的用户反馈和人工评估结果进行分析和总结，找出共性问题和关键问题。

对于用户反馈的搜索结果不完整或不准确的问题，需要检查索引中的数据是否完整准确，以及查询语句是否能够准确地覆盖用户的需求。

## 5.6 A/B 测试不同查询策略：

对不同的查询构建方式、索引设置等进行 A/B 测试。

例如，对比使用`match`查询和`match_phrase`查询在同一搜索场景下的效果，根据测试结果选择更精准的查询策略并应用到实际系统中。

## 6 应用代码层面 的 精准度优化

**代码层面 的 精准度优化， 这也是尼恩团队之前干过的一个重点工作。**

用户侧看到的一个搜索，在后台，往往是多个搜索的 合并。

### 6.1 代码层面，使用 组合模式 实现 精准度优化

尼恩在 前面讲到， 一般来说，至少用两个查询，达到 搜索结果的 精准度优化：

* 一个term查询 ，精准查询
* 一个match查询， 模糊查询
* 然后 把结果组合起来。

下面是一个简单的例子， 使用 esclient 完成 一个term查询，一个match查询，然后再Java中的CompletableFuture来 执行这两个查询 结果合并 。

### 6.2 代码层面 组合模式的 精准度优化的实例

这里，使用 Java 的 `Elasticsearch` 客户端（ `RestHighLevelClient`）和 `CompletableFuture` 来并行执行 `term`查询和 `match` 查询，并将结果合并的示例代码。

引入maven 依赖先：

```
<dependency>  
    <groupId>org.elasticsearch.client</groupId>  
    <artifactId>elasticsearch-rest-high-level-client</artifactId>  
    <version>7.15.0</version>  <!-- 根据你的 Elasticsearch 版本选择合适的版本 -->  
</dependency>
```

以下是示例代码：

```
import org.elasticsearch.action.search.SearchRequest;  
import org.elasticsearch.action.search.SearchResponse;  
import org.elasticsearch.client.RequestOptions;  
import org.elasticsearch.client.RestHighLevelClient;  
import org.elasticsearch.index.query.BoolQueryBuilder;  
import org.elasticsearch.index.query.QueryBuilders;  
import org.elasticsearch.search.builder.SearchSourceBuilder;  
import java.io.IOException;  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
  
public class ElasticsearchQueryExample {  
    public static void main(String[] args) {  
        // 创建一个 Elasticsearch 客户端，这里假设你已经正确配置了 RestHighLevelClient  
        RestHighLevelClient esClient = createEsClient();    
  
        // 创建一个线程池，用于 CompletableFuture 的异步执行  
        ExecutorService executorService = Executors.newFixedThreadPool(2);  
  
        try {  
            // 构建 term 查询  
            SearchRequest termSearchRequest = new SearchRequest("your_index_name");  
            SearchSourceBuilder termSearchSourceBuilder = new SearchSourceBuilder();  
            termSearchSourceBuilder.query(QueryBuilders.termQuery("product_id", "P001"));  
            termSearchRequest.source(termSearchSourceBuilder);  
  
            // 构建 match 查询  
            SearchRequest matchSearchRequest = new SearchRequest("your_index_name");  
            SearchSourceBuilder matchSearchSourceBuilder = new SearchSourceBuilder();  
            matchSearchSourceBuilder.query(QueryBuilders.matchQuery("product_name", "手机"));  
            matchSearchRequest.source(matchSearchSourceBuilder);  
  
            // 使用 CompletableFuture 并行执行两个查询  
            CompletableFuture<SearchResponse> termFuture = CompletableFuture.supplyAsync(() -> {  
                try {  
                    return esClient.search(termSearchRequest, RequestOptions.DEFAULT);  
                } catch (IOException e) {  
                    throw new RuntimeException(e);  
                }  
            }, executorService);  
  
            CompletableFuture<SearchResponse> matchFuture = CompletableFuture.supplyAsync(() -> {  
                try {  
                    return esClient.search(matchSearchRequest, RequestOptions.DEFAULT);  
                } catch (IOException e) {  
                    throw new RuntimeException(e);  
                }  
            }, executorService);  
  
            // 合并两个查询的结果  
            CompletableFuture.allOf(termFuture, matchFuture).thenRun(() -> {  
                try {  
                    SearchResponse termResponse = termFuture.get();  
                    SearchResponse matchResponse = matchFuture.get();  
                    // 在这里可以处理和合并两个查询的结果  
                    // 例如，将两个响应中的 hits 合并  
                    // 以下是简单的打印结果示例，实际应用中可以进行更复杂的处理  
                    System.out.println("Term Query Results:");  
                    System.out.println(termResponse);  
                    System.out.println("Match Query Results:");  
                    System.out.println(matchResponse);  
                } catch (InterruptedException | ExecutionException e) {  
                    e.printStackTrace();  
                }  
            }).join();  
  
        } finally {  
            try {  
                esClient.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
            executorService.shutdown();  
        }  
    }  
  
    private static RestHighLevelClient createEsClient() {  
        // 这里应该根据你的 Elasticsearch 集群配置来创建 RestHighLevelClient  
        // 以下是一个简单的示例，实际使用时请根据实际情况修改  
        // return new RestHighLevelClient(RestClient.builder(new HttpHost("localhost", 9200, "http")));  
        return null;  // 你需要根据实际情况完成这个方法  
    }  
}
```

上面的代码，通过  `createEsClient` 首先创建 `RestHighLevelClient`。 示例中暂时返回 `null`，这里 需要实现该方法，根据实际情况配置 `HttpHost` 等信息。

**第一步：构建查询请求**：

这里 分别构建了 `term` 查询和 `match` 查询的 `SearchRequest` 和 `SearchSourceBuilder`。

`QueryBuilders.termQuery("product_id", "P001")` 用于创建 `term` 查询，精确查找 `product_id` 为 `P001` 的文档。

`QueryBuilders.matchQuery("product_name", "手机")` 用于创建 `match` 查询，查找 `product_name` 字段包含 “手机” 的文档。

**第二步：使用 CompletableFuture 并行执行**：

使用 `CompletableFuture.supplyAsync` 将两个查询的执行包装为异步任务，在 `executorService` 线程池中执行。

每个 `CompletableFuture` 会发送 `search` 请求到 `Elasticsearch`并返回 `SearchResponse`。

**第三步：结果合并和处理**：

使用 `CompletableFuture.allOf(termFuture, matchFuture).thenRun` 来确保两个查询都完成。

在 `thenRun` 中，通过 `termFuture.get()` 和 `matchFuture.get()`获取结果。

你可以根据具体需求对结果进行合并和处理，这里只是简单地打印结果，实际应用中可以进行更复杂的操作，例如将两个结果中的 `hits`合并，或者根据 `_score` 对结果排序等。

### 6.3 代码层面 组合模式的 精准度优化的流程图

## 说在最后：有问题找老架构取经‍

回到开始的时候的面试题：

> * es怎么提升速度和精准度？
> * 提升搜索精准度，有那些的实用技巧？
> * 高性能的搜索系统如何设计，如何提高搜索精准度？

按照此文的套路去回答，一定会 **吊打面试官，让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

在面试之前，建议大家系统化的刷一波 5000页《[尼恩Java面试宝典PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**前段时间，******[刚指导一个 40岁大龄，上岸： 转架构，收3个外企offer， 机会多了，不焦虑了，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486604&idx=1&sn=69e1b1b95f50947c2a1bf8601ac77b89&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer自由” 很容易的.

前段时间一个武汉的跟着尼恩卷了2年的小伙伴， 在极度严寒/痛苦被裁的环境下， offer拿到手软， 实现真正的 “offer自由” 。

## 空窗1年/空窗2年，如何通过一份绝世好简历，  起死回生  ？

[**空窗8月：中厂大龄34岁，被裁8月收一大厂offer， 年薪65W，转架构后逆天改命!**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486405&idx=1&sn=02336456cf4a09268c142c0b13327682&chksm=97b57a4da0c2f35be50007aee12b8f9e9aa7a74864f720846ae6dc58a5fa4b092e22d1805e1e&scene=21#wechat_redirect)

[**空窗2年：42岁被裁2年，天快塌了，急救1个月，拿到开发经理offer，起死回生**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486028&idx=1&sn=cf10ecda7a986b26e0359eba7a9cfeff&chksm=97b57bc4a0c2f2d2be01f2656ac65e9bd8854b91c64b01eb55f56e645f3c733525418808b06c&scene=21#wechat_redirect)

[**空窗半年：35岁被裁6个月， 职业绝望，转架构急救上岸，DDD和3高项目太重要了**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486098&idx=1&sn=bbc5732b071477573bfab8a259d208d3&chksm=97b57b1aa0c2f20c27dd74b490b6062c9eec262a3ac25548534ff70290b172dcc51c6eafe532&scene=21#wechat_redirect)

[**空窗1.5年：失业15个月，学习40天拿offer， 绝境翻盘，如何实现？**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486345&idx=1&sn=8978c2c98378e85efe9a089fa08fc9e8&chksm=97b57a01a0c2f317c29c7bedab1fa8a9f6101ab35cf005df447d662daa9c17d56f4e4c3a354a&scene=21#wechat_redirect)

## 100W 年薪  大逆袭,  如何实现  ？

# [**100W案例，100W年薪的底层逻辑是什么？** 如何实现年薪百万？ 如何远离  中年危机？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)

[**100W案例2****：**40岁小伙被裁6个月，猛卷3月拿100W年薪 ，秘诀：首席架构/总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[**环境太糟，如何升 P8级，年入100W？**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

## 如何  评价一份绝世好简历， 实现逆天改命，包含AI、大数据、golang、Java  等

## 

* ## [逆天大涨：暴涨200%，29岁/7年/双非一本 ， 从13K涨到 37K ，如何做到的？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&chksm=97b57a69a0c2f37fad55b164ba7afb963e0db0366901da97a1fadc599cb5cf1ef95dfac736f2&scene=21#wechat_redirect)
* ## [逆天改命：27岁被裁2月，转P6降维攻击，2个月提 JD/PDD 两大offer，时来运转，人生翻盘!!  大逆袭!!](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486421&idx=1&sn=02a88c23c8fe689a662214d5297f88a0&chksm=97b57a5da0c2f34b1f87ed0bb948ddfc9327533bdfc44b88ad25201949bbdb8ae7439904e675&scene=21#wechat_redirect)
* ## [急救上岸：29岁（golang）被裁3月，转架构降维打击，收3个大厂offer， 年薪60W，逆天改命](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486392&idx=1&sn=16c34d7f960805f659adfb27676ee2de&chksm=97b57a30a0c2f326092cd7f79e125fd8bd5a006ed7459f1a2a81a8d557388297a02acd2cd5ce&scene=21#wechat_redirect)

* [**绝地逢生：**9年经验](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486132&idx=1&sn=bb949c992b3a35935bf5fca57f19e2d8&chksm=97b57b3ca0c2f22aeb0e648191801aff933164dcf2b06eb4b6274df6499bffb0bba8e8169205&scene=21#wechat_redirect)[自考](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486132&idx=1&sn=bb949c992b3a35935bf5fca57f19e2d8&chksm=97b57b3ca0c2f22aeb0e648191801aff933164dcf2b06eb4b6274df6499bffb0bba8e8169205&scene=21#wechat_redirect)小伙伴，跟着尼恩狠卷3月硬核技术，面试机会爆表，2周后收3个offer ，满血复活

**职业救助站**

实现职业转型，极速上岸

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000页面试宝典、20个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的"在看"和"赞"，谢谢
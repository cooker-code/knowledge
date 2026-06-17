---
title: RAG 终于到第四代了，Agentic RAG SDK 来了
author: 祝威廉
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIyNzQyNzgxNQ==&mid=2247484953&idx=1&sn=426e965b2525c5ed3493abe498f275f1&chksm=e9f72eb337be2f8f6206f56b0431e59e826899081fce5607345d3d261719c96cebf167954f76&mpshare=1&scene=24&srcid=1027SaNSAGtYKjBUdC8J6v5k&sharer_shareinfo=c2a82ce69d2958c2a7800492fe7b1a6b&sharer_shareinfo_first=c2a82ce69d2958c2a7800492fe7b1a6b#rd
---

前三代 RAG 进化小史

第一代 RAG ,是以 embeding + 全文检索为基础做召回，以rerank模型为精排。这代其实RAG受限于当时的技术条件而诞生的一个技术，那个时候大模型窗口普遍4-8k，并发少，速度慢，模型效果也一般，所以只能采用传统技术来做召回，并且采用小模型(rerank)来做排序。 第一代RAG解决了从无到有的问题。下图是 一代 RAG 典型的流程。

第一代RAG 最大的问题就出在召回方案缺乏语义理解，受限于 embeding 对文本长度的支持，还长期受限于文本碎片化的困扰。后续RAG公司没有从根本上去解决这个问题，而是不断的在其上打补丁比如，加入实体图搜索，在数据预处理上进行大量场景定制，在召回策略上也要进行场景定制。一代RAG天花板太低，实际上效果很难再提升，唯一的出入就是定制，但这样很难控制成本。

随着大模型技术的发展，我们推出了第二代 RAG , 该RAG核心是使用大模型做召回，解决了海量数据召回速度和成本问题，同时高度优化窗口的使用率，彻底解决语义召回的难题。 效果也是出奇的好，并且实际成本相比定制化RAG 也更低，成本很透明：

但是第二代依然有明显的缺陷，对于需要复杂拆解的问题比较困难，下面图片是个例子：

于是我推出了第三代 RAG , Agentic RAG, 基于二代的基础上，我们引入了 Agentic 范式，并增加了联网以及规划能力，从而使用通用的方式解决了，下推演示了这种规划能力：

通过提前拆解规划问题，从而将复杂的问题拆解成一些简单的信息召回问题，然后结合多轮召回，最后给出答案。

这就是第三代 Agentic LLM Native RAG.  第三代 RAG 是已经 RAG 的顶级形态。 那么第四代 RAG 进化方向应该在哪呢？

第四代RAG

经过前三代RAG的积累，第四代 RAG 在使用方式开始进入新的阶段。传统方式将 RAG 作为一个服务（比如兼容 OpenAI 接口），比如这样：

```
 auto-coder.rag serve --lite --port 8102 \ --doc_dir ./byzer-llm \ --model v3_chat
```

然后就可以通过 OpenAI SDK 通过 127.0.0.1:8102 进行访问了。这种方式对于和其他产品结合并不方便。

为此，我们提供了命令行模式：

```
echo "byzer-llm prompt 函数是什么" | auto-coder.rag run  \--doc_dir ./byzer-llm --model v3_chat \--output_format text \--agentic
```

这意味着你可以随时随地的对任意一个目录进行 RAG 查询。命令行的方式几乎可以被任何Agent 调用，你可以很轻松的嵌入到 Cursor/Auto-Coder/Claude Code/Codex/Qoder 中，你只需要给这些 Code Agent 通过 Rule 添加一个简单的使用说明， Code Agent 就会在合适的时候使用知识库进行学习：

除了命令行，我们也提供了 Java，Go, NodeJS ,Python 的绑定，比如Java， 你可以添加如下依赖：

```
<dependency>    <groupId>tech.mlsql</groupId>    <artifactId>autocoder-rag-sdk</artifactId>    <version>0.0.2</version></dependency>
```

然后可以写类似的代码：

```
package com.example;import com.autocoder.rag.sdk.RAGClient;import com.autocoder.rag.sdk.RAGConfig;import java.util.stream.Stream;import java.util.stream.Collectors;public class Main {    public static void main(String[] args) {        RAGConfig config = new RAGConfig("./byzer-llm");        config.setModel("v3_chat");  
        RAGClient client = new RAGClient(config);  
        String query = "byzer-llm prompt function是什么";  
        try {            Stream<String> stream = client.queryStream(query);            String result = stream.collect(Collectors.joining("\n"));            System.out.println(result);        } catch (Exception e) {            System.err.println("Error: " + e.getMessage());            System.exit(1);        }    }}
```

就完成了流式的 RAG 查询，可以将 RAG 方便的各种集成到用户的应用中。

第四代RAG 无需部署即可实现开箱即用，并且能够让各种语言以库的方式被直接使用。所以我们将 第四代 RAG 称为 Agentic LLM Native RAG SDK.  你也可以简称为 RAG SDK.

RAG 是未来所有AI应用的基础设施，而 auto-coder.RAG 提供了更加便利友好方式和集成方式，以及最优秀的查询效果。期待大家使用和集成。

更多内容访问(或者点击原文链接)

https://uelng8wukz.feishu.cn/wiki/Nf7vwLNYiii0YXk7DbBcuCmLn4b
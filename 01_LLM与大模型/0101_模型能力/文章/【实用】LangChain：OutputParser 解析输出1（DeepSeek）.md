---
title: 【实用】LangChain：OutputParser 解析输出1（DeepSeek）
author: 网工手艺
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487506&idx=1&sn=0358787a70a039090d2f76da87be4145&chksm=c010328bf394f92f988fb387ec23d17c940a52a5bf5fac068068717e5ffc54c97fdd4d5a1c65&mpshare=1&scene=24&srcid=1013IW0Li6rmrFlk9QbYmVFS&sharer_shareinfo=eef1248f58b66a507eb13aab328499d3&sharer_shareinfo_first=eef1248f58b66a507eb13aab328499d3#rd
---

哈喽，大家好，我又来了。见字如面，每篇锤炼！这是本公众号第`191`篇原创文章。

长假已过，我们继续开展实验测试。此前有部分读者反馈，在国内使用 ChatGPT 的 API 存在不便，其实大家也可以选择国产 AI 大模型作为替代方案，例如 DeepSeek。为此，在之前的系列文章中，我从零开始，在 LangChain 框架中成功接入 DeepSeek，并引入了 Prompt、Chains 等核心概念。本次内容将聚焦 LangChain 的“输出解析”处理，逐步探讨 OutputParser 组件的具体应用。稍作梳理，分享于你。

特别说明：即使你没读过前面的文章也没关系，通常不会影响对本文实验的理解。本文设计的是实操小实验，你可以跟着一步步操作。实验代码可在本公众号后台回复wgsy获取，若能结合前面的实验一起做，效果会更佳。本文实验内容源自书籍《网络工程师的AI之路》的示例代码，并在此基础上进行了拓展。大家只需跟着教程把几个示例实验动手做一遍，上手 LangChain 框架后，就可考虑自主迁移到实际应用场景中。同时，我们也欢迎在社群里分享你的实战脱敏案例，让我们一起营造互助共创的氛围。

# 一、实验背景

我们先来回顾一下：Chains 是 LangChain 框架早期（<1.0 版本）最基础也最常用的链式组件，其核心职责可以总结为：将 PromptTemplate + LLM + OutputParser 串联成一条可调用、可复用的流水线。

* prompt：PromptTemplate，负责把用户输入变量渲染成最终文本，为与 LLM 交互作必要准备。
* LLM：任意 LangChain 兼容的 LLM（在线、本地 AI 大模型，如 DeepSeek 等）。
* output\_parser（可选项）：对模型返回的字符串做后处理。本文重点介绍。

顺便提一句：随着 LangChain 框架的迭代，Chains 正逐步被更灵活的 Runnable 替代，自 LangChain 0.1.17 版本起已被标记为“弃用”，1.0 版本后将正式移除。不过“链”的核心思想和设计逻辑仍是绕不开的基础，我们不妨先从 Chains 入手，打好底子再学新内容会更轻松。后续，官方推荐使用 Runnable 接口（| 管道符）或 prompt | llm | parser 的写法，更简洁、可组合、支持流式与并发。我们也将在后续文章继续展开介绍。

来，一步一步跟着做实验。

# 二、实验过程

## 2.1 模块升级

我们先把 LangChain 框架的模块升级到最新版本吧。如果你想让终端屏蔽升级过程的输出，可以将-U参数调整为-qU，这样升级过程就不会显示了。

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple langchain -U  
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple langchain-community -U  
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple langchain-openai -U  
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple langchain-deepseek -U
```

执行升级后，我们可以检查一下当前版本。

```
C:\Windows\System32>pip list | findstr langchain  
langchain                 0.3.27  
langchain-community       0.3.31  
langchain-core            0.3.79  
langchain-deepseek        0.1.4  
langchain-openai          0.3.35  
langchain-text-splitters  0.3.11  
  
C:\Windows\System32>
```

你会发现，这些 AI 相关的 Python 库迭代速度很快。我们学习时可以保持“求新探索”的心态，在实验前把相关模块升级到最新版本。

“求新探索”是一个获取超额 α 收益的有效途径。

[【思考】AI 助力网络工程师 α 收益](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487410&idx=1&sn=785467c7fe7f3bee30c8a720c9426040&scene=21#wechat_redirect)

## 2.2 实验脚本

接下来是咱们的实验代码，我同样尽量把它精简到最小可运行版本，这样方便新手快速上手和理解。

```
from langchain.chains import LLMChain  
from langchain.prompts import PromptTemplate  
from langchain.chat_models import init_chat_model  
from langchain.schema import BaseOutputParser  
  
# 定义一个OutputParser  
class OutputParser(BaseOutputParser):  
    def parse(self, text):  
        # 在这里对LLM的输出进行处理，比如去除多余字符  
        parsed_text = text.strip()  # 去掉前后多余的空格，空字符  
        # 也可以对LLM的输出做进一步格式化，比如将所有内容转换为大写英文字母  
        parsed_text = parsed_text.upper()  
        return parsed_text  
  
#设置 DeepSeek API Key  
api_key = "在此处填写个人的 DeepSeek API Key"  
  
#设置 PromptTemplate  
prompt_template = PromptTemplate(  
    input_variables=["question"],  
    template="你是一位资深网络工程师，请回答以下问题：{question}"  
    )  
  
#启用 DeepSeek 的 V3 模型作为 LLM  
llm = init_chat_model("deepseek-chat", model_provider="deepseek",api_key=api_key,temperature=0.7)  
  
# 创建链，指定PromptTemplate和OutputParser  
chain = LLMChain(llm=llm, prompt=prompt_template, output_parser=OutputParser())  
  
#执行链，将 DeepSeek 的回答赋值给变量 result 并打印出来  
result = chain.run(question="Does a network engineer need to learn English?")  
print(result)
```

这份代码用 LangChain 框架调用 DeepSeek 大语言模型，并通过自定义输出解析器处理模型的返回结果。除了 OutputParser 类外，其他内容与早前实验的逻辑一致，所以我们这里重点介绍这个类。

* 自定义了一个 OutputParser 类，继承自 LangChain 的 BaseOutputParser 基类
* 重写了 parse 方法，实现对模型输出的处理：

+ 使用 strip() 方法去除文本前后的空格
+ 使用 upper() 方法将所有英文字母转换为大写

* 返回处理后的文本

为了演示结果转换成大写的功能，我们给 LangChain 的 Chains 链安排一个英语问题：Does a network engineer need to learn English?——即你觉得网络工程师需要学习英语吗？

## 2.3 实验结果

运行脚本后，我们很快会看到下面这两个 Warning 提示。从这里能看到，LLMChain 类和 Chain.run 方法已经分别在 langchain 0.1.17 和 0.1.0 版本中被标记为废弃（deprecated），计划在 langchain 1.0 版本中正式移除（removed）。 Warning 里也贴心地给出了替代方案：RunnableSequence 类和 invoke 方法。不过 Warning 并不会影响我们这次实验，我们目前 LangChain 的版本是 0.3.x，相关组件还没移除。

DeepSeep 正常返回，输出格式符合预期。这说明我们的“输出解析”功能配置正确。实验成功。

接下来，你可以回头重读实验代码，结合 AI 聊天工具继续研究细节、调整需求、重新调试。

# 三、本文总结

我的文章通常分两类：一类是“思考”型，主要分享我对问题的看法；另一类是“实用”型，包含可直接跟着操作的实验——你可以按步骤动手实践，做出效果后再回头结合理论知识进行迁移思考。

我们从安装 LangChain 库开始，通过可执行实验一步步走进它的世界，同时把相关概念穿插其中。实验跑通后，划个重点请你细品：Chains（链）的底层逻辑，就是把 PromptTemplate + LLM + OutputParser 串成一条可调用、可复用的流水线。不过 Chains 后续会逐步被更灵活的 Runnable 替代，你可以借助 AI 工具，试着把咱们这次的实验代码改成 Runnable 的标准写法，这样能更好适配后续更新的 LangChain 框架。

[实验 1：LangChain 框架之 Prompt](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487463&idx=1&sn=c088b55245efea99764955d080a5d646&scene=21#wechat_redirect)

[实验 2：LangChain 框架之 Chains](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487479&idx=1&sn=0c735ab130f3dfdb150a5a60566edb1c&scene=21#wechat_redirect)

实验 3：LangChain 框架之 OutputParser（本文，后续还会继续梳理，如结构化解析 JSON / Pydantic 等。）

[【发布】《网络工程师的 AI 之路》勘误及优化全纪录](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487425&idx=1&sn=2dbe1200cbeb04d352363c7ead3872a9&scene=21#wechat_redirect)

好啦，话不多说，你赶紧动手做实验吧。先跟着引例“依葫芦画瓢”练起来， 掌握了这些基础套路和操作逻辑后，实际生产中编写具体的 Python 脚本反而不是重点，毕竟现在有 AI 工具可以辅助编码。

[【邀请】共享网络工程师 AI 时代红利](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247487402&idx=1&sn=4bda35526b99f4d3a388580eaeccfda4&scene=21#wechat_redirect)

[【思考】关于信息解析](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247485322&idx=1&sn=176c94145f189879d0e7e4b4496e14b5&scene=21#wechat_redirect)

[【思考】从 NetDevOps 到 NetAIOps](https://mp.weixin.qq.com/s?__biz=MzkyNTMwNTI1NQ==&mid=2247485568&idx=1&sn=7bad940e55bcbf386edd1feacf5d0e5b&scene=21#wechat_redirect)

附上内蒙古呼伦贝尔满洲里北湖公园清晨湖景作为题图。夏末初秋，楼宇与湖面倒影交相辉映，如诗如画。

今晚先分享这些，后续将继续梳理分享。

> 我知乎总目录   
> https://zhuanlan.zhihu.com/p/370526806   
>   
> 读者再创作目录   
> https://zhuanlan.zhihu.com/p/498090646

感谢阅读，欢迎关注点赞，转发分享。

觉得有帮助，特别认可，可打赏 1 元鼓励！

2025 年 10 月于广东汕头

（本人在家乡广东汕头工作和生活。汕头位于大陆海岸线与北回归线的交界处，是著名侨乡和“美食孤岛”，也是中国数字经济创新发展大会的永久会址。欢迎我的读者和同行朋友们有机会来汕头进行商务出差或旅游。如果你们有空来汕头，欢迎与我交流本地的风土人情和网络技术实践。我们的团队专注于大型数据中心和通信基础网的建设与运维，业务辐射华南乃至全国全球。欢迎交流洽谈合作。另，《网络工程师的Python之路》、《网络工程师的AI之路》已被多所院校、培训机构列为教材，也是 NetDevOps 生产工具书，欢迎购买支持。购书可加入读者交流群，享受更多增值服务。）
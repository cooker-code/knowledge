---
title: 你花8小时调研，AI用3秒钟搞定：Tavily API才是你的终极研究助理
author: 小妖同学学AI
date: 
url: https://mp.weixin.qq.com/s?__biz=Mzk0NDcyMjk4OA==&mid=2247493431&idx=1&sn=07a1cf8b9dc920397be77f981be63fd5&chksm=c223dec1c81b6c325bc219198a8035f816d7d2198ffc17cbd5f683670ebef20ca5fed036e3ab&mpshare=1&scene=24&srcid=1120QRLcBpIV3PAlj4wacV37&sharer_shareinfo=80143a7de754f659879a9df810b4a50a&sharer_shareinfo_first=80143a7de754f659879a9df810b4a50a#rd
---

```
```
```
```
点击上方“小妖同学学AI”，选择“星标”公众号
```
```

超级无敌干货，第一时间送达！！！

在信息爆炸的时代，你是否也曾被海量的网络信息淹没？当你需要快速获取准确答案时，是否厌倦了在无数网页中手动筛选的繁琐过程？

这正是我们今天要探讨的Tavily API想要解决的问题。作为一个专为AI智能体设计的搜索工具，Tavily正在重新定义我们获取信息的方式。它不像传统搜索引擎那样返回成千上万的网页链接，而是直接给出精准的答案和分析。

想象一下，你的应用程序能够像拥有私人研究助理一样，自动完成复杂的信息检索、数据整合和知识提炼。无论是市场调研、学术研究还是商业分析，Tavily都能将数小时的手动搜索工作压缩到几秒钟内完成。

网址：https://www.tavily.com/ 

账号登录

每个月有1000的免费调用额度

## 搜索 ：

### 

pip 安装 tavily-python

image-20251003215250824

python 测试代码

```
# To install: pip install tavily-python  
from tavily import TavilyClient  
client = TavilyClient("tvly-dev-***")  
response = client.search(  
    query=""  
)  
print(response)
```

image-20251003212632197

搜索结果：

image-20251003212754946

## URL数据提取 ：

### 

```
# To install: pip install tavily-python  
from tavily import TavilyClient  
client = TavilyClient("tvly-dev-***")  
response = client.extract(  
    urls=[""]  
)  
print(response)
```

image-20251003213111851

## 爬虫 ：

### 

```
# To install: pip install tavily-python  
from tavily import TavilyClient  
client = TavilyClient("tvly-dev-***")  
response = client.crawl(  
    url="",  
    extract_depth="advanced"  
)  
print(response)
```

```
// To install: npm i @tavily/core  
const { tavily } = require('@tavily/core');  
const client = tavily({ apiKey: "tvly-dev-***" });  
client.crawl("", {  
    extractDepth: "advanced"  
})  
.then(console.log);
```

感谢大家的点赞和关注，我们下期见！
```
```
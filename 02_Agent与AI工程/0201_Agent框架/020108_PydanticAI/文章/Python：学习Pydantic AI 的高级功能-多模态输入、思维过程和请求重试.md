---
title: Python：学习Pydantic AI 的高级功能-多模态输入、思维过程和请求重试
author: 猫咪不吃愚
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzNDY1NzE0NQ==&mid=2247487044&idx=1&sn=9003c8bcf7beef7d2c204ee1c2c30cd9&chksm=c3a6ee517d99af57abedad74f5c4629c9c1dd260ded216c7e451095facda7ded61b025a69051&mpshare=1&scene=24&srcid=1112avAoqKzNaoReksiJ4aOU&sharer_shareinfo=2ddca6328ae31c6dd1a8992b242eb2ab&sharer_shareinfo_first=2ddca6328ae31c6dd1a8992b242eb2ab#rd
---

> 字数 1212，阅读大约需 7 分钟

# Python：学习Pydantic AI 的高级功能-多模态输入、思维过程和请求重试

超级好用的AI Agent开发框架：`PydanticAI`。在Python构建人工智能驱动的应用程序的时候，往往会出现非结构化输出、类型不匹配和生产可靠性问题。`PydanticAI将帮助我们解决`：**将大型语言模型集成到Python应用程序缺乏生产系统所需的结构和验证的问题**。本系列文章持续分享pydantic ai的用法和最新更新内容！ #PydanticAI  

---

✨ 刚刷到的朋友注意啦！点击【关注】，从此升职加薪不迷路  
🌟 **若觉得内容有用，长按点赞**！你的每次互动，都是我深夜码字的星光

✅ 点击「**关注**」→ 持续收获成长能量  
✅ 点亮「**点赞**」→ 为干货内容打call  
✅ 设为「**星标**」⭐️→ 微信公众号算法优先推送，更新不错过

---

## 内容摘要

本文章将围绕以下内容介绍，多数内容来自官网文档：Pydantic AI[1]。`如果想要在agent开发过程中实战，建议仔细阅读pydantic ai的官网文档`，值得细细品味。

1. 1. 多模态信息输入有哪些呢？
2. 2. 如何获取模型的思考过程？
3. 3. 模型调用的重试机制配置？

---

## 多模态输入

多模态LLM 现在能够理解音频、视频、图像和文档内容，因此`pydantic ai也支持多种格式信息的输入`，不仅仅局限在文字。  
使用 ImageUrl、AudioUrl、VideoUrl 或 DocumentUrl 提供 URL 时，Pydantic AI 会下载文件内容，然后将其作为 API 请求的一部分发送。

### 图片

支持本地和URL

```
result = agent.run_sync(  
    [  
        'What company is this logo from?',  
        ImageUrl(url='https://iili.io/3Hs4FMg.png'),  
    ]  
)  
  
agent = Agent(model='openai:gpt-4o')  
result = agent.run_sync(  
    [  
        'What company is this logo from?',  
        BinaryContent(data=image_response.content, media_type='image/png'),    
    ]  
)
```

### 文档

也支持本地和URL

```
result = agent.run_sync(  
    [  
        'What is the main content of this document?',  
        DocumentUrl(url='https://storage.googleapis.com/cloud-samples-data/generative-ai/pdf/2403.05530.pdf'),  
    ]  
)  
  
result = agent.run_sync(  
    [  
        'What is the main content of this document?',  
        BinaryContent(data=pdf_path.read_bytes(), media_type='application/pdf'),  
    ]  
)
```

### 更多支持

阅读：图像、音频、视频和文档输入 - Pydantic AI 框架[2]

## 思维过程

### 什么是思考

什么是思考：`思考（或称推理）是模型在给出最终答案之前，一步一步解决问题的过程`。`模型一般是禁用此功能的`。如果在代码开发agent的过程中，需要了解每一步的思考过程，需要先确认模型服务是否支持此功能。主要模型供应商对思考过程的支持[3]

### 以Gemini为例：启用思考功能

```
from pydantic_ai import Agent  
from pydantic_ai.models.google import GoogleModel, GoogleModelSettings  
  
model = GoogleModel('gemini-2.5-pro-preview-03-25')  
settings = GoogleModelSettings(google_thinking_config={'include_thoughts': True})  
agent = Agent(model, model_settings=settings)  
...
```

---

## 请求重试

### 重试的价值

Pydantic AI 通过自定义 HTTP 传输为模型提供商发出的 HTTP 请求提供重试功能。这对于**处理瞬时故障（如速率限制、网络超时或临时服务器错误）特别有用**

### 智能重试

```
from httpx import AsyncClient, HTTPStatusError  
from tenacity import retry_if_exception_type, stop_after_attempt, wait_exponential  
  
from pydantic_ai import Agent  
from pydantic_ai.models.openai import OpenAIChatModel  
from pydantic_ai.providers.openai import OpenAIProvider  
from pydantic_ai.retries import AsyncTenacityTransport, RetryConfig, wait_retry_after  
  
  
def create_retrying_client():  
    """Create a client with smart retry handling for multiple error types."""  
  
    def should_retry_status(response):  
        """Raise exceptions for retryable HTTP status codes."""  
        if response.status_code in (429, 502, 503, 504):  
            response.raise_for_status()  # This will raise HTTPStatusError  
  
    transport = AsyncTenacityTransport(  
        config=RetryConfig(  
            # Retry on HTTP errors and connection issues  
            retry=retry_if_exception_type((HTTPStatusError, ConnectionError)),  
            # Smart waiting: respects Retry-After headers, falls back to exponential backoff  
            wait=wait_retry_after(  
                fallback_strategy=wait_exponential(multiplier=1, max=60),  
                max_wait=300  
            ),  
            # Stop after 5 attempts  
            stop=stop_after_attempt(5),  
            # Re-raise the last exception if all retries fail  
            reraise=True  
        ),  
        validate_response=should_retry_status  
    )  
    return AsyncClient(transport=transport)  
  
# Use the retrying client with a model  
client = create_retrying_client()  
model = OpenAIChatModel('gpt-4o', provider=OpenAIProvider(http_client=client))  
agent = Agent(model)
```

### 等待策略

* • 自动解析 HTTP 429 响应中的 Retry-After 响应头
* • 支持秒数格式（"30"）和 HTTP 日期格式（"Wed, 21 Oct 2015 07:28:00 GMT"）
* • 当响应头不存在时，回退到选择的策略
* • 遵守 max\_wait 限制以防止过度延迟

```
from tenacity import wait_exponential  
  
from pydantic_ai.retries import wait_retry_after  
  
# Basic usage - respects Retry-After headers, falls back to exponential backoff  
wait_strategy_1 = wait_retry_after()  
  
# Custom configuration  
wait_strategy_2 = wait_retry_after(  
    fallback_strategy=wait_exponential(multiplier=2, max=120),  
    max_wait=600  # Never wait more than 10 minutes  
)
```

### 异步http

```
from httpx import AsyncClient  
from tenacity import stop_after_attempt  
  
from pydantic_ai.retries import AsyncTenacityTransport, RetryConfig  
  
  
def validator(response):  
    """Treat responses with HTTP status 4xx/5xx as failures that need to be retried.  
    Without a response validator, only network errors and timeouts will result in a retry.  
    """  
    response.raise_for_status()  
  
# Create the transport  
transport = AsyncTenacityTransport(  
    config=RetryConfig(stop=stop_after_attempt(3), reraise=True),  
    validate_response=validator  
)  
  
# Create a client using the transport:  
client = AsyncClient(transport=transport)
```

### 更多重试机制

阅读：HTTP 请求重试 - Pydantic AI 框架[4]

---

## 彩蛋结尾

嘿，别滑了！手指停一停，听我说句悄悄话👇  
🌟 关注我：下次更新，系统会自动弹窗提醒你；  
📌 收藏本文：点个收藏，让它成为你的知识库，随时挖宝；  
❤️ 点赞在看：你的每个赞都是我熬夜写文的“鸡血”；

## 更多内容

访问我的专属博客：https://www.funkygod.vip/

#### 引用链接

`[1]` Pydantic AI: *https://ai.pydantic.dev/*  
`[2]` 图像、音频、视频和文档输入 - Pydantic AI 框架: *https://ai.pydantic.org.cn/input/#document-input*  
`[3]` 主要模型供应商对思考过程的支持: *https://ai.pydantic.org.cn/thinking/*  
`[4]` HTTP 请求重试 - Pydantic AI 框架: *https://ai.pydantic.org.cn/retries/#performance-considerations*
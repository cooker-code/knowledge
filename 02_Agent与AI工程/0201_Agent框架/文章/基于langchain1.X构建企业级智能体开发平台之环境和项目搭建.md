---
title: 基于langchain1.X构建企业级智能体开发平台之环境和项目搭建
author: 万智创界
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA4MTc1MjMxOA==&mid=2457539673&idx=1&sn=3c9f53c8273243b2d770e9dc2b737925&chksm=8975614e3b68fa01c7891cdbc0641ccdff9dc4b241bf7d93bfe4b66f1c3cebb541b4d474f5bb&mpshare=1&scene=24&srcid=1224l7dLwnngFfg28wFTZBJV&sharer_shareinfo=404acb9e434ca7c303597f6add371078&sharer_shareinfo_first=404acb9e434ca7c303597f6add371078#rd
---

> 前提说明：由于langchain1.0之前的版本和现在的1.0有非常大的调整；我这边的langchain指的是langchain1.0及以后的版本;
>
> 项目说明：我们这个教程并不是一步步从0开始教大家上手langchain框架，而是要求大家具备了一定的了解基于这个项目来开发企业组的智能体平台；本人也是一边学习一边完善；也会在日常学习到的知识放到examples中；
>
> 项目源码地址：https://github.com/wanrengang/langchain-agent-framework；

接下来会用一系列的文章介绍如何基于langchain1.X构建企业级智能体开发平台；

## 开发环境

```
## 创建虚拟环境：  
uv venv --python 3.13  # 或者你偏好的兼容版本，例如 3.10, 3.12  
## 启动虚拟环境：  
.venv\Scripts\Activate  
## 项目初始化  
uv init
```

项目目录结构

项目的目录结构如图；接下来安装项目依赖和配置a

```
pi key  
uv sync  
相关的apikey 目前主要用到的是deepseek模型，在.env 中配置自己key就行了；  
#模型的key  
DEEPSEEK_API_KEY=sk-d2d9a279278*****
```

本地启动：

```
python main.py
```

## 开发说明

1. 后端服务是集成了fastapi；
2. api完成openai格式适配；

---

到此环境已配置工作已经完成；接下来我们通过一个简单的例子来演示智能体的搭建；

## 简单Demo 演示

```
from langchain.agents import create_agent  
# 模型引入  
from core.llm import get_llm  
agent = create_agent(  
    model=get_llm(),  
    system_prompt="你是一名多才多艺的智能助手，可以调用工具帮助用户解决问题。"  
)  
def get_base_agent():  
    return agent
```

上面的代码就完成一个智能体开发，通过下面的代码完成智能体注册：

```
在*agents/registry.py*页面中进行注册：  
# agents/registry.py  
from typing import Dict, Callable  
from langchain_core.runnables import Runnable  
from agents.graph import get_base_agent  
# 具备联网搜索的deep agent  
  
AGENT_REGISTRY: Dict[str, Callable[[], Runnable]] = {  
     
    "base_agent":get_base_agent,  
      
}
```

## 智能体测试

我们可以直接在cherry studio进行测试，配置如下：

配置如图

开始对话

## 总结

到此，一个完整的智能体已经搭建完成；由于我们完成了open ai 格式的兼容，这个智能体可以直接在各种工具的直接使用也可以以api形式给前端页面调用；后面我们会一步一步丰富智能体内容；
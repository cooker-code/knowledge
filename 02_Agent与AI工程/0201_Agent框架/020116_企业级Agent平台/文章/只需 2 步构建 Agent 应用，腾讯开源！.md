---
title: 只需 2 步构建 Agent 应用，腾讯开源！
author: 渔夫 AIDaily
date: 
url: https://mp.weixin.qq.com/s?__biz=MzUyODgxNzM0Nw==&mid=2247488869&idx=1&sn=e97e145ebc47ba15b7afdd5d257dc95f&chksm=fb44ea3c3c8cbf8d91a324f2326309a96dc25c952902252f9bba2159614e6a19d87762d4ee10&mpshare=1&scene=24&srcid=1115h8JTEURWRYjLK7TXdeDF&sharer_shareinfo=f1895459b2abe960ba9077071e07d8e6&sharer_shareinfo_first=f1895459b2abe960ba9077071e07d8e6#rd
---

你好，我是渔夫。

腾讯开源了Agent手脚架，只需两步，轻松构建自己的Agent应用，这个 Agent 框架叫“Youtu-Agent”。

主打“开箱即用”，很灵活，不管是专业开发者，还是 AI 爱好者，都能轻松上手。

## 核心概念

* **智能体（Agent）**：一个配置了提示词、工具和环境的大语言模型。
* **工具包（Toolkit）**：智能体可以使用的封装工具集。
* **环境（Environment）**：agent 操作的世界（例如，浏览器、shell）。
* **上下文管理器（ContextManager）**：一个可配置模块，用于管理智能体的上下文窗口。
* **基准（Benchmark）**：一个针对特定数据集的封装工作流，包括预处理、执行和判断逻辑。

老规矩，直接上干货！

部署两种方式，你也可以用Docker部署。

第一步：github拉代码，安装到本地

```
git clone https://github.com/TencentCloudADP/youtu-agent.git  
cd youtu-agent  
uv sync  # or, `make sync`  
source ./.venv/bin/activate  
cp .env.example .env
```

第二步：修改 `.env` 配置文件

```
# LLM Configuration (required)  
UTU_LLM_TYPE=chat.completions  
UTU_LLM_MODEL=deepseek-chat  
UTU_LLM_BASE_URL=https://api.deepseek.com/v1  
UTU_LLM_API_KEY=sk-04datrhrjytrhrafa7f7f1ab024311 # Required  
  
# Serper API Configuration  
# Get your key from https://serper.dev/playground  
SERPER_API_KEY=a0e5c4trgrhrhhfhhtrhrt6jtracf5e8  
  
# Frontend Configuration  
# Note: IP must be 0.0.0.0 for Docker container port forwarding  
UTU_WEBUI_PORT=8848  
UTU_WEBUI_IP=0.0.0.0
```

第三步：启动

```
python scripts/cli_chat.py --stream --config default
```

然后你就可以 CLI 方式聊天了。效果

另外，你还可以借助 web-ui 以可视化方式预览 Agent 的运行情况，执行下面代码。

```
curl -LO https://github.com/Tencent/Youtu-agent/releases/download/frontend%2Fv0.1.5/utu_agent_ui-0.1.5-py3-none-any.whl  
  
uv pip install utu_agent_ui-0.1.5-py3-none-any.whl
```

启动：

```
python examples/svg_generator/main_web.py
```

官方给出的基准性能如下。

小框架，轻量级工具，很适合一些小规模小应用的场景，也容易折腾，用来练手不一而选，赶紧去试试吧。

GitHub仓库：

https://github.com/TencentCloudADP/youtu-agent

这是我的第272篇原创笔记记得点赞，在看，转发，打赏为我加油~👇👇点击下方卡片关注我👇👇
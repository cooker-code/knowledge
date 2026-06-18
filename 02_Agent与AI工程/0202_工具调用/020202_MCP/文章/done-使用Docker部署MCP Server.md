> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: 使用Docker部署MCP Server
author: ThinkInAI社区
date:
url: http://mp.weixin.qq.com/s?__biz=MzA4ODg0NDkzOA==&mid=2247510996&idx=1&sn=e947428f3de4f303a1a095b8f0836190&chksm=91b878c1c618e0f646dfeee086bb037366d24b3a9b14d2328c98071becc2c76f7434519f7092&mpshare=1&scene=24&srcid=0430Vo1IkKfolI1w0vjf4yMW&sharer_shareinfo=2a8f9bbd1d2e778cb6e414ed2978b672&sharer_shareinfo_first=2a8f9bbd1d2e778cb6e414ed2978b672#rd
---

AI Agent 正在快速发展——从实验室到实际应用。随着它们从生成文本到执行实际操作，模型上下文协议（MCP）已经成为了连接代理与工具的实际上标准。

前两天 Robin 以《模型的世界 应用的天下》为题发表演讲，带来了文心大模型 4.5 Turbo、文心大模型 X1 Turbo 两大新模型，连发高说服力数字人、通用超级智能体心响 APP、内容操作系统沧舟 OS 等多款 AI 应用，覆盖 AI 数字人、代码智能体、多智能体协作等热门赛道。同时，**宣布百度将帮助开发者全面拥抱 MCP**。

此外千帆还正式发布了国内首个企业级 MCP 服务，第一批已经有超过 1000 个 MCP Servers 供企业及开发者灵活选择。

MCP 很令人兴奋。它是简单且模块化的，并基于原生网络原则构建。我们相信它有可能像容器对应用程序部署所做的那样，标准化并简化一个复杂且碎片化的领域——即为代理式 AI 交互提供服务。

MCP 客户端和服务器具有巨大的潜力，但体验尚未达到生产就绪状态。发现是碎片化的。一般来说，你需要手工部署 MCP 服务，或者使用第三方部署的服务，这会带来一些问题：

* 手工部署依赖机器环境。部署环境可能缺乏类库、工具，你可能无法成功部署一个 MCP 服务器。尤其在一些大厂中，内网环境和软件版本无法让你正常安装和运行
* 手工部署的 MCP 服务器可能有安全问题，可以任意的访问你的服务器
* 第三方服务无法保证安全保证，有可能造成信息的泄露。相当于你把你的信息全部都上交给了第三方

如果回顾一下这些问题，你会发现历史总是相似的，云服务流行的时代我们也遇到了类似的问题，Docker 是一种优雅的解决云服务时代这类问题的解决方案之一。

在 AI Agent 时代， Docker 公司也与时俱进，提出了利用 Docker 技术部署 MCP 服务的技术： Docker MCP Catalog 。

通过 Docker Desktop 一键启动，您可以在几秒钟内启动 MCP 服务器，并将其连接到 Docker AI Agent、Claude、Cursor、VS Code、Windsurf、continue.dev 和 Goose 等客户端，无需复杂设置。它还包含内置凭证和 OAuth 管理功能，并与您的 Docker Hub 账户集成，确保顺畅的身份验证，并在必要时轻松撤销凭证。AGateway MCP 服务器动态将启用的工具暴露给兼容客户端，而新的 docker mcp CLI 可让您轻松构建、运行和管理这些工具。并且内置了内存、网络和磁盘隔离，每项工具默认安全运行，从第一天起即可用于生产环境。

Docker 公司的雄心是巨大的：未来的 Docker MCP Catalog 和 Toolkit 会是什么样子？想象一下：直接在 Docker Hub 上浏览数百个即用型 MCP 服务器，并像启动 Redis 或 Postgres 那样轻松地启动它们。只需几下点击即可将它们连接到代理。无需硬编码的密钥，无需通过 npx 或 uvx 以完整主机访问权限启动工具，也无需牺牲隔离或安全性。最好的是？运行一个 Docker 容器，MCP 工具就能自动工作。凭借熟悉的命令和工具，学习曲线几乎为零——而可能性却是巨大的。

你可以通过以下网址访问当前 Docker 所支持的 MCP Server:

https://hub.docker.com/catalogs/mcp

接下来我通过 fetch 这个 mcp 工具对比本地部署和 docker 部署的区别。

Fetch MCP 服务器是 Anthropic 官方开发的一款专注于网页内容抓取的高效数据采集工具。作为一个轻量级的网页爬虫服务器，它能够智能地将 HTML 内容转换为 Markdown 格式，特别适合与 LLM（大语言模型）配合使用。其核心功能和参数配置如下：

fetch 从互联网获取指定 URL 的内容并提取为 Markdown 格式,参数如下：

* `url`（字符串，必需）：要抓取的 URL
* `max_length`（整数，可选）：返回的最大字符数（默认：5000）
* `start_index`（整数，可选）：从该字符索引开始提取内容（默认：0）
* `raw`（布尔值，可选）：获取原始内容而不进行 Markdown 转换（默认：false）

## 本地 stdio 部署

你可以在你的 mcp host 配置 fetch mcp server:

```
"mcpServers": {
  "fetch": {
    "command": "uvx",
    "args": ["mcp-server-fetch"]
  }
}
```

比如我在 deepchat 中的配置如下：

然后在对话框调用这个工具：

## 使用 docker

可以看到，如果要运行 fetch mcp server, 你需要有 uvx 工具，它是由 Astral 公司开发的一个极其快速的 Python 包管理器。或者你通过 pip 安装和配置，这需要你先准备好这些前置的工具，而且你的机器环境不一定允许你安装。

Docker 目前提供了 fetch 工具，你可以先使用下面的命令安装：

```
docker run -i --rm mcp/fetch
```

你可以在 Agent Host 中这样部署：

```
"mcpServers": {
  "fetch": {
    "command": "docker",
    "args": ["run", "-i", "--rm", "mcp/fetch"]
  }
}
```



然后在对话框中测试：

使用 docker 部署 mcp 的好处还是那些：

1. **环境一致性 (Consistent Environments):**
2. **资源利用率高 (Resource Efficiency):**
3. **快速部署与扩展 (Rapid Deployment & Scalability):**
4. **可移植性 (Portability):**
5. **隔离性 (Isolation):**
6. **简化配置和管理 (Simplified Configuration & Management):**

解决了本地和第三方部署带来的环境问题、资源占用问题和安全问题。

#mcp #docker #agent #智能体 #deepchat

---

**加入ThinkInAI社区**

如果你也对AI工具充满兴趣，欢迎加入我们的ThinkInAI社区，在这里，你可以：

* 获取最新AI工具资讯
* 参与实战经验分享
* 结识志同道合的伙伴
* 共同探讨AI应用方向

扫描文末二维码，加入ThinkInAI社区，一起拥抱AI新时代！
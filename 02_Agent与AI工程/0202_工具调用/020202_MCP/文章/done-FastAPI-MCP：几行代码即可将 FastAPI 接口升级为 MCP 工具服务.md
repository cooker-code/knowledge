> 已吸收至：[[02_Agent与AI工程/0202_工具调用/020202_MCP/020202_核心知识点/MCP生产接入与治理边界|MCP生产接入与治理边界]]
---
title: FastAPI-MCP：几行代码即可将 FastAPI 接口升级为 MCP 工具服务
author: 数智脉动
date:
url: https://mp.weixin.qq.com/s?__biz=Mzk0Mjc0OTY1OA==&mid=2247485640&idx=1&sn=d6ebd944649f36117e3ccdc7c235bf32&chksm=c22da1ee8bdd57853f4f2c9e30052c1549eeccd012f167326d5b130ad4a32874b97795e545e0&mpshare=1&scene=24&srcid=101191I7NkyzhC0CnZ5LOaLU&sharer_shareinfo=321770e8e3ddded19a59202b5122df55&sharer_shareinfo_first=321770e8e3ddded19a59202b5122df55#rd
---

**点击蓝字 关注我们**

若你之前已经开发部署了大量 FastAPI 接口服务，打算让 LLM 调用这些已有的 API 服务来查询数据，执行任务。通常，需要将已有的 API 服务改写为支持 MCP 协议的服务：手写大量的“胶水代码”，定义工具（Tool）。工作繁琐，且容易出错。

**如何快速将已有的 API 服务转换成 MCP 服务呢？**

本文介绍一个开源项目：https://github.com/tadata-org/fastapi\_mcp，**几行代码就把现有 FastAPI 接口升级为支持 MCP 协议的工具服务，并保留原有的鉴权、数据校验和文档，极大降低了传统 API 与 AI 工作流之间的集成门槛，支持独立部署、SSE 流式传输、实时文档同步等功能。**

#### 什么是 FastAPI-MCP

* GitHub: https://github.com/tadata-org/fastapi\_mcp
* 官方文档: https://fastapi-mcp.tadata.com/

**FastAPI-MCP 是一个能将现有的 FastAPI 接口（Endpoints）自动转换并暴露为模型上下文协议（Model Context Protocol）工具服务的框架。**

这意味着，不需要为 LLM 重写任何 API 调用逻辑。FastAPI-MCP 会自动读取 FastAPI 应用的路由、Pydantic 模型、甚至是接口文档，然后生成完全符合 MCP 规范的工具集，让 LLM 可以直接理解和调用。

**FastAPI-MCP 特性**

* **🛡️自带认证**：直接复用 FastAPI 中已有的安全依赖（Depends()）。无论是 HTTPBearer 令牌认证还是复杂的 OAuth2 流程，都无需为 LLM 重复开发，安全性无缝继承。
* ✨**零/极简配置**：大多数情况下，需要两行代码就能完成所有设置，真正做到“即插即用”。
* 📚**保留结构与文档**：精确地将 Pydantic 模型和接口文档（docstrings）翻译成 LLM 能理解的工具输入/输出结构（Schema）和描述，保证了调用时的准确性。
* 🔧**灵活部署**：可以将 MCP 服务直接挂载到现有的 FastAPI 应用上，也可以将其作为独立服务分开部署。
* ⚡**高效通信**：通过 ASGI 接口直接与 FastAPI 应用通信，避免了从 MCP 服务到 API 服务的额外 HTTP 调用，延迟更低，效率更高。

#### 安装步骤

安装 fastapi-mcp 包，使用 uv 工具或 pip 进行安装。

使用 uv (推荐):

```
uv add fastapi-mcp
```

使用 pip：

```
pip install fastapi-mcp
```

#### 代码示例

**1、基础用法**

假设已经有了一个标准的 FastAPI 应用 app。现在，想让它所有的 API 都能被 LLM 调用。

* FastApiMCP(app)：自动完成了所有 API 的扫描和转换
* mcp.mount\_http()：创建了一个新的接口（默认为 /mcp），这个接口就是 LLM 和应用沟通的桥梁。

```
from fastapi import FastAPI
from fastapi_mcp import FastApiMCP

# 已有的 FastAPI app 
app = FastAPI()

# 1. 从你的 FastAPI app 创建一个 MCP 实例
mcp = FastApiMCP(app)
# 2. 将 MCP 服务挂载到你的 app 上
mcp.mount_http()

# 启动服务
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**2、无缝集成现有认证**

假设应用中有一个需要授权才能访问的接口。已有的FastAPI 应用代码如下：

```
from fastapi import Depends, FastAPI
from fastapi.security import HTTPBearer 

app = FastAPI()
token_auth_scheme = HTTPBearer()

# 一个受保护的接口，必须提供有效的 Bearer Token
@app.get("/private")
async def private(token: str = Depends(token_auth_scheme)):
    return {"message": f"Your token is: {token.credentials}"}
```

现在，如何让 LLM 在调用这个工具时也带上认证信息？使用 FastAPI-MCP，几乎不需要做任何额外的事：

```
from fastapi_mcp import FastApiMCP, AuthConfig

# ... (前面的 app 和 token_auth_scheme 定义)

# 创建 MCP 服务时，将 FastAPI 中已有的 Depends(token_auth_scheme) 传递给 AuthConfig
# 通过 AuthConfig 告诉 FastApiMCP 需要哪些认证依赖
mcp = FastApiMCP(
    app,
    name="Protected MCP",
    auth_config=AuthConfig(
        dependencies=[Depends(token_auth_scheme)], # 把认证依赖传进去
    ),
)

# 挂载服务
mcp.mount_http()
```

当 MCP 客户端尝试调用这个受保护的工具但未提供 Authorization 头时，FastAPI-MCP 会正确地返回 401 Unauthorized，从而触发客户端启动标准的 OAuth 流程，**引导用户完成登录并获取Token，然后用这个 Token 再次调用工具。整个过程完全符合现代安全实践，而几乎没写任何额外的认证代码！**

**3、选择性暴露的 API**

我们可能不想把所有的 API 都暴露给 LLM。FastAPI-MCP 同样提供了非常灵活的过滤机制，可以通过 operation\_id 或 tags 来精确控制。这种灵活性使得管理和维护面向 LLM 的工具集变得异常轻松和安全。

```
# 仅暴露 operation_id 为 "get_item" 和 "list_items" 的接口
include_operations_mcp = FastApiMCP(
    app,
    name="Item API MCP - Included Operations",
    include_operations=["get_item", "list_items"],
)

# 暴露除了带有 "search" 标签以外的所有接口
exclude_tags_mcp = FastApiMCP(
    app,
    name="Item API MCP - Excluded Tags",
    exclude_tags=["search"],
)

# 挂载不同的 MCP 服务到不同的路径
include_operations_mcp.mount_http(mount_path="/include-operations-mcp")
exclude_tags_mcp.mount_http(mount_path="/exclude-tags-mcp")
```

以上是 FastAPI-MCP 项目介绍。**该项目对 FastAPI 开发者十分友好，可以将已有的 API 服务快速高效地转换为 MCP 工具服务，从而将现有的业务能力赋能给大模型。**

***欢迎关注我，后续介绍更多 MCP 相关的内容。***
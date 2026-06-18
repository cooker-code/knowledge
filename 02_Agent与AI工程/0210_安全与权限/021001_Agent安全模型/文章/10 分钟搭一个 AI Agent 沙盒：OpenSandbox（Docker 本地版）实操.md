---
title: 10 分钟搭一个 AI Agent 沙盒：OpenSandbox（Docker 本地版）实操
author: 码农知道的事
date: 
url: https://mp.weixin.qq.com/s?__biz=MzIxNzY5NDk5MA==&mid=2247484967&idx=1&sn=4813461e0f2d05ca516d1cb7eb78845f&chksm=96bb7a91d297eac6dd08ad18d59c5ce87c1c5cf6a04a6b78f37ff9b318400767aa1c87126efc&mpshare=1&scene=24&srcid=0318xX7S9vdXAAlxYzDiy8pD&sharer_shareinfo=5c279afbedb6549669d67c12aeaf1eee&sharer_shareinfo_first=5c279afbedb6549669d67c12aeaf1eee#rd
---

# 10 分钟搭一个 AI Agent 沙盒：OpenSandbox（Docker 本地版）实操

> 目标：本地用 Docker 跑起 OpenSandbox Server，然后用 Python SDK 创建一个 sandbox，执行命令、读写文件，并运行 Code Interpreter。

## 1. OpenSandbox 是什么（30 秒版）

OpenSandbox 是一个**通用沙盒平台**，用“协议化的方式”提供：

* **生命周期管理**

  （创建/暂停/恢复/删除/续期…）
* **执行能力**

  （命令执行、文件系统操作、代码解释器、资源指标）

它的价值在于：当你在做 AI Coding/GUI Agent/评测平台时，你不需要为“执行环境”反复造轮子。

来源：

* https://github.com/alibaba/OpenSandbox
* https://github.com/alibaba/OpenSandbox/blob/main/README.md

## 2. 环境准备

* Docker（本地运行必需）
* Python 3.10+（官方示例推荐）

来源（Requirements）：

* https://github.com/alibaba/OpenSandbox/blob/main/README.md

## 3. 安装并启动 OpenSandbox Server（Docker Runtime）

### 3.1 安装 server

如果你用 `uv`：

* `uv pip install opensandbox-server`

### 3.2 生成示例配置（docker）

* `opensandbox-server init-config ~/.sandbox.toml --example docker`

### 3.3 启动服务

* `opensandbox-server`

> 小提示：查看参数

* `opensandbox-server -h`

来源：

* https://github.com/alibaba/OpenSandbox/blob/main/README.md

## 4. 用 Python SDK 创建 sandbox 并执行命令/文件操作

### 4.1 安装 Code Interpreter SDK

* `uv pip install opensandbox-code-interpreter`

### 4.2 最小可运行示例（从 README 提炼）

把下面脚本保存为 `demo.py` 后运行（注意：需要你本机已经启动 `opensandbox-server`）：

```
import asyncio  
from datetime import timedelta  
  
from code_interpreter import CodeInterpreter, SupportedLanguage  
from opensandbox import Sandbox  
from opensandbox.models import WriteEntry  
  
async def main() -> None:  
    # 1) 创建 sandbox（镜像 + entrypoint + 环境变量 + 超时）  
    sandbox = await Sandbox.create(  
        "opensandbox/code-interpreter:v1.0.1",  
        entrypoint=["/opt/opensandbox/code-interpreter.sh"],  
        env={"PYTHON_VERSION": "3.11"},  
        timeout=timedelta(minutes=10),  
    )  
  
    async with sandbox:  
        # 2) 执行命令  
        execution = await sandbox.commands.run("echo 'Hello OpenSandbox!'")  
        print(execution.logs.stdout[0].text)  
  
        # 3) 写文件  
        await sandbox.files.write_files([  
            WriteEntry(path="/tmp/hello.txt", data="Hello World", mode=644)  
        ])  
  
        # 4) 读文件  
        content = await sandbox.files.read_file("/tmp/hello.txt")  
        print(f"Content: {content}")  
  
        # 5) Code Interpreter（Python）  
        interpreter = await CodeInterpreter.create(sandbox)  
        result = await interpreter.codes.run(  
            """  
import sys  
print(sys.version)  
result = 2 + 2  
result  
""",  
            language=SupportedLanguage.PYTHON,  
        )  
        print(result.result[0].text)  
        print(result.logs.stdout[0].text)  
  
        # 6) 清理  
        await sandbox.kill()  
  
if __name__ == "__main__":  
    asyncio.run(main())
```

来源：

* https://github.com/alibaba/OpenSandbox/blob/main/README.md

## 5. 常见问题与避坑

### 5.1 “为什么我创建 sandbox 失败？”

优先检查：Docker 是否正常、server 是否启动、配置文件路径是否正确。

### 5.2 “跑着跑着 sandbox 没了？”

OpenSandbox 设计里有 **TTL 过期机制**；长任务应考虑续期（Lifecycle API 里有 `renew-expiration`）。

来源：

* https://github.com/alibaba/OpenSandbox/blob/main/docs/architecture.md

### 5.3 “我想限制它出网/只允许白名单”

OpenSandbox 设计了 per-sandbox 的 **egress controls**，适合做 Agent 安全边界。

来源：

* https://github.com/alibaba/OpenSandbox/blob/main/README.md

## 6. 扩展阅读：OpenSandbox 架构怎么工作的？

如果你想写成“平台级”而不是“工具级”，可以强调这三点：

1. **SDK 层**

   ：Python/Java/Kotlin/TS/C# 等多语言统一调用体验
2. **Specs 层（OpenAPI）**

   ：把生命周期与执行能力定义成稳定协议
3. **Runtime 层**

   ：本地 Docker 与高性能 K8s runtime
4. **Sandbox 实例层**

   ：容器内注入 `execd`，提供命令/文件/指标/代码执行的统一 API

来源：

* https://github.com/alibaba/OpenSandbox/blob/main/docs/architecture.md

---

## 参考链接

* OpenSandbox 仓库：https://github.com/alibaba/OpenSandbox
* README（安装/示例）：https://github.com/alibaba/OpenSandbox/blob/main/README.md
* 架构文档：https://github.com/alibaba/OpenSandbox/blob/main/docs/architecture.md
---
title: Jupyter AI 3.0：Agent驱动的探索性数据分析工具
author: 数据打工人的自我修养
date: 数你皮数你皮
url: https://mp.weixin.qq.com/s?__biz=MzE5MTU2NjM5NQ==&mid=2247484777&idx=1&sn=2a4024695550eb6ddc18249b443a7207&chksm=97a75fd4dd03329e73aeda6ace15dac0693029251c04bbe23b7f223b9e97fa45f699bf627777&mpshare=1&scene=24&srcid=0511cPGUU5DCgXyzBuZFuGvc&sharer_shareinfo=4991104d68a8cbff1763cd4b3087a59c&sharer_shareinfo_first=4991104d68a8cbff1763cd4b3087a59c#rd
---

## 一、Jupyter AI 3.0 版本概述

Jupyter AI 3.0 是 JupyterLab 的开源 AI 扩展，于 2026 年 4 月 1 日正式发布。它将 AI Agent 深度集成到 JupyterLab 中，支持你在Notebook 环境内直接与多种 AI 模型交互协作。

相比 v2.x 版本仅支持 API 调用式的聊天，v3.0 做了架构层面的全面重构，引入了 **Agent Client Protocol（ACP）** 和 **MCP Server**，实现了真正的 Agent 协作体验。

与单纯的 Claude Code 等智能体不同，Jupyter AI 在交互式探索分析上更加友好。Agent 通过 Jupyter MCP 工具直接操作  JupyterLab，自动插入并执行代码，无需手动复制粘贴。

更重要的是，分析过程中的中间变量始终保留在 Notebook  中，你可以随时基于已有结果继续深入探索，而不需要每次从头加载数据集、重新计算。

## 二、核心新特性

## 1. Agent 支持（通过 ACP 协议）

v3.0 最大的变化是引入了 ACP 协议，支持接入多种前沿 AI Agent：

* **Claude Code**
* **Codex CLI**
* **Gemini CLI**
* **GitHub Copilot CLI**
* **Goose**
* **Kiro CLI**
* **Mistral Vibe**
* **OpenCode**

Agent 通过 `@` 提及的方式在聊天中调用，类似 Slack 或 Teams 的交互方式。例如输入 `@Claude 如何优化这段代码？` 即可触发 Claude Agent 响应。

如果只配了一个智能体，比如claude code，则无须@使用，直接输入提示词即可。其中@file可以添加文件到提示词中：

## 2. 实时协作聊天界面

* 全新的原生聊天 UI，支持实时流式响应
* 支持 Tool Call 和 Diff 视图，可直接查看 Agent 对文件的修改
* 聊天以 `.chat` 文件形式保存在工作区，可随时重新打开。
* 支持同时创建多个聊天，管理不同工作线程

对话chat可侧边栏Jupyter Chat上方新建、切换

## 3. 工具调用权限控制

Agent 在执行写文件、运行命令等操作前会请求你授权，默认启用安全防护机制：

* 读取当前工作区文件：无需授权
* 写入文件、执行 Shell 命令、调用 MCP 工具：需要你确认

## 4. Jupyter MCP Server

内置 `jupyter_server_mcp`，Agent 可以：

* 读取和编辑 Notebook 单元格
* 执行 Notebook 代码
* 管理文件系统
* 通过 MCP 协议访问自定义工具和资源

## 5. 自定义 MCP Server 集成

你可通过 `.jupyter/mcp_settings.json` 配置文件添加自定义 MCP Server，为 Agent 提供领域特定的工具和资源。

以下是一个 Python MCP Server 的配置示例：

```
{  "mcpServers": {    "my-database": {      "command": "python",      "args": ["/home/your-username/mcp-servers/db_server.py"],      "env": {        "DB_HOST": "数据库ip",        "DB_PORT": "端口号",        "DB_USER": "用户名",        "DB_PASSWORD": "密码",        "DB_NAME": "数据库db_name"      }    }  }}
```

其中 `db_server.py` 是你自己编写的 MCP Server 脚本，通过 `env` 字段传入环境变量，避免将敏感信息硬编码在代码中。`command` 也可以替换为 `uvx`、`npx` 等包管理器直接运行已发布的 MCP 包。

配置完成后重启 Jupyter Lab，Agent 即可在聊天中调用这些工具。

## 6. 多聊天架构

* 支持无限数量的并发聊天
* 每个聊天独立保存为 `.chat` 文件
* 可拖拽文件或 Notebook 单元格作为上下文附件

## 三、安装教程

## 系统要求

* Python 3.10+
* JupyterLab 4.x
* Node.js 和 npm（用于 ACP 适配器）
* Linux 或 macOS（Windows当前测试兼容有些问题，可考虑使用 WSL2安装linux系统）

## 方式一：pip 安装

```
pip install jupyter-ai
```

国内网络环境建议使用镜像源加速下载：

```
pip install jupyter-ai -i https://pypi.tuna.tsinghua.edu.cn/simple
```

常用国内 pip 镜像源：

| 镜像 | 地址 |
| --- | --- |
| 清华大学 | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| 阿里云 | `https://mirrors.aliyun.com/pypi/simple` |
| 豆瓣 | `https://pypi.douban.com/simple` |

## 方式二：uv 安装

```
uv pip install jupyter-ai
```

## 方式三：conda 安装

```
conda install -c conda-forge jupyter-ai
```

## 安装 Agent

Jupyter AI 3.0 不再内置 Agent，需要单独安装。以 Claude Code 为例：

**第一步：配置 npm 国内镜像（可选，加速下载）**

```
npm config set registry https://registry.npmmirror.com
```

**第二步：安装 Claude Code CLI**

```
npm install -g @anthropic-ai/claude-code
```

**第三步：安装 ACP 适配器**

```
npm install -g @zed-industries/claude-agent-acp
```

注意：官方文档写的是 @agentclientprotocol/claude-agent-acp，但 jupyter-ai 3.0.0 代码实际检测的是 @zed-industries/claude-agent-acp。装错包名会导致 Claude 不出现在 @ 菜单中，以代码要求的包名为准。

**第四步：启动 JupyterLab**

```
jupyter lab
```

在 JupyterLab 中，点击 Launcher 页面的 **Chat** 卡片或侧边栏的 **+** 按钮创建新聊天，输入 `@` 即可看到可用的 Agent 列表。

## 四、WSL2 环境配置

## 笔者使用的是win电脑，不想打双系统。使用WSL2可以无缝访问win系统上的文件。

Windows 文件挂载在 `/mnt/c/` 下，例如：

 `C:\Users\用户名\Desktop` → `/mnt/c/Users/用户名/Desktop`

## 

## 什么是 WSL2

WSL2（Windows Subsystem for Linux 2）是微软提供的在 Windows 上运行 Linux 的方案。它内置完整的 Linux 内核，与 Windows 共享网络，且 Linux 进程可直接通过 `localhost` 被 Windows 访问。

安装方式（在 Windows PowerShell 管理员模式下执行）：

```
wsl --install
```

安装完成后重启电脑，按提示设置 Ubuntu 用户名和密码即可。

## WSL2 环境变量继承问题

WSL2 默认会自动继承 Windows 的系统环境变量。如果 Windows 上已配置了 `ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY` 等变量，它们会被带入 WSL2。

**不过不用担心**：Claude Code 的配置优先级为 `settings.json` 的 `env` 字段 > 系统环境变量。只要在 `~/.claude/settings.json` 中配置了模型参数，即使 Windows 侧有同名环境变量，也会被 settings.json 覆盖，不会产生冲突。

如果你没有使用 settings.json，而是通过系统环境变量配置模型，可以在 WSL2 的 `~/.bashrc` 末尾用 export 覆盖（~/.bashrc 在 Windows 变量导入之后执行）：

```
# 在 ~/.bashrc 末尾添加 
```

```
export ANTHROPIC_BASE_URL=https://你的API地址export ANTHROPIC_API_KEY=你的密钥export ANTHROPIC_MODEL=你的模型名
```

如果希望彻底禁用 Windows 环境变量继承，可编辑 `/etc/wsl.conf`：

```
[interop] enabled = false
```

然后在 Windows PowerShell 中执行 `wsl --shutdown` 重启 WSL2。

注意：禁用后 WSL2 中将无法直接调用 Windows 的 .exe 程序。

## 配置 Jupyter Lab 监听和免密登录

WSL2 中的Linux系统无图形界面， Jupyter Lab 默认只监听 localhost，在windows本地浏览器访问每次启动都需要输入 token。以下配置可以让局域网内的设备访问并免密登录。

如果是mac和图形linux系统，则无需次操作。

**1. 生成配置文件**

```
jupyter lab --generate-config
```

配置文件路径为 `~/.jupyter/jupyter_lab_config.py`。

**2. 编辑配置文件**

```
# 监听所有网络接口，允许局域网访问
```

```
c.ServerApp.ip = '0.0.0.0'
```

```
# 免密登录：不验证 token
```

```
c.ServerApp.token = ''
```

```
c.ServerApp.password = ''
```

```
# 禁止启动时自动打开浏览器（WSL2 中无需自动打开）
```

```
c.ServerApp.open_browser = False
```

```
# 指定默认工作目录（可选）
```

```
c.ServerApp.root_dir = '/home/你的用户名'
```

**3. 启动 Jupyter Lab**

```
jupyter lab
```

启动后在 Windows 浏览器中直接访问 `http://ip地址:8888` 即可，无需输入 token 或密码。

比如笔者直接输入：http://172.17.181.19:8888/

如需确认 WSL2 子系统的 IP 地址，在 WSL2 终端中执行：

```
hostname -I
```

输出类似 `172.20.196.1`，即为 WSL2 的内部 IP。Windows 浏览器中也可通过该 IP 访问：

```
http://172.20.196.1:8888
```

## 五、claude code配置国内模型

使用国内模型时，只需通过配置文件指定 API 端点和密钥即可。

**配置文件路径**

```
~/.claude/settings.json
```

**配置内容**

如果使用小米 MiMo 模型，完整配置参考如下：

```
{  "env": {    "ANTHROPIC_AUTH_TOKEN": "你的Token",    "ANTHROPIC_BASE_URL": "https://token-plan-cn.xiaomimimo.com/anthropic",    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "mimo-v2.5-pro",    "ANTHROPIC_DEFAULT_OPUS_MODEL": "mimo-v2.5-pro",    "ANTHROPIC_DEFAULT_SONNET_MODEL": "mimo-v2.5-pro",    "ANTHROPIC_MODEL": "mimo-v2.5-pro",    "ANTHROPIC_REASONING_MODEL": "mimo-v2.5-pro"  },  "includeCoAuthoredBy": false}
```

**验证配置**

```
claude -p "你好"
```

如果能正常回复，说明模型配置成功。

## 六、测试视频参考

## linux系统缺少中文字体，警告可按需关闭

```
import warningswarnings.filterwarnings("ignore")
```

---

参考资料：

* Jupyter AI 官方文档：https://jupyter-ai.readthedocs.io/en/v3/
* Jupyter AI GitHub 仓库：https://github.com/jupyterlab/jupyter-ai
* Agent Client Protocol：https://agentclientprotocol.com

---

### 感谢各位朋友的点赞支持~

本公众号聚焦Excel、VBA、SQL、Python、业务分析相关内容分享

点击下方链接关注，持续为您呈现更多内容~
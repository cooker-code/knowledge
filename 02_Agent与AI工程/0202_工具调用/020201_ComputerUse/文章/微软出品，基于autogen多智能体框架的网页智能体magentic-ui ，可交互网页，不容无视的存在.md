---
title: 微软出品，基于autogen多智能体框架的网页智能体magentic-ui ，可交互网页，不容无视的存在
author: AI大模型智能体
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzMzczMjU1Ng==&mid=2247484002&idx=1&sn=d249a1d327a2625359eda8e760435973&chksm=c392ab3af0109ddd6375bec41084d4d0058177838680d54d6f669ecc24089f2575f6ee401610&mpshare=1&scene=24&srcid=1030bBjTBs1xUBtamy7EFKaA&sharer_shareinfo=f5b089fcd262598e58253909ad21284c&sharer_shareinfo_first=f5b089fcd262598e58253909ad21284c#rd
---

Magentic-UI 是微软开发的一个研究原型，是一个以人为中心的多智能体系统界面，能够浏览和执行网页操作、生成和执行代码、生成和分析文件。这是一个基于 AutoGen 框架构建的智能体协作平台。

一 技术点

### 后端：

* 框架: FastAPI (异步Web框架)

* 数据库: SQLModel + PostgreSQL

* 浏览器自动化: Playwright

前端：

* 框架: React 18 + TypeScript

* UI库: Ant Design + Tailwind CSS

* 状态管理: Zustand

二、本地安装部署

使用uv来安装，相当方便，建议之前使用conda环境的可以切换uv；

国内的话， 我使用的本地的ollama比较方便一些，当然openai有的话效果更好

```
uv venv --python=3.12 .venv.venv/bin/activateuv pip install magentic-ui[ollama] --upgrade  
```

启动后端服务

```
magentic-ui --port 8081
```

注意使用docker images 查看下镜像是不是有

```
ghcr.io/microsoft/magentic-ui-python-env      0.0.1             155b0117a656   2 months ago   1.44GBghcr.io/microsoft/magentic-ui-browser         0.0.1             8db16d53c968   2 months ago   1.98GB
```

这是打开的界面

三 数据库

3.1 默认配置文件使用是sqlite数据库

```
class Settings(BaseSettings):    DATABASE_URI: str = "sqlite:///./magentic_ui.db"    API_DOCS: bool = True    CLEANUP_INTERVAL: int = 300  # 5 minutes    SESSION_TIMEOUT: int = 3600 * 100  # 24 hour    CONFIG_DIR: str = "configs"  # Default config directory relative to app_root    DEFAULT_USER_ID: str = "guestuser@gmail.com"    UPGRADE_DATABASE: bool = False    model_config = {"env_prefix": "MAGENTIC_UI_"}
```

SQLModel: 基于 Pydantic 的 ORM，提供类型安全和数据验证

['alembic\_version', 'gallery', 'message', 'plan', 'run', 'session', 'settings', 'team'] ，其中看一个 session的定义：

```
class Session(SQLModel, table=True):    id: Optional[int] = Field(primary_key=True)    created_at: datetime    updated_at: datetime    user_id: Optional[str]    team_id: Optional[int] = Field(ForeignKey("team.id"))  # 关联Team    name: Optional[str]
```

32 详细看看每个表

Team 表 (团队配置表)：用户可以创建不同的AI团队配置，每个团队有不同的代理组合和能力

component字段: 存储完整的团队组件配置，包括：

* 代理列表（agents）

* 模型配置（models）

* 工具配置（tools）

* 终止条件（terminations）

```
{"cooperative_planning": true, "autonomous_execution": false, "max_actions_per_step": 5, "multiple_tools_per_call": false, "max_turns": 20, "approval_policy": "auto-conservative", "allow_for_replans": true, "do_bing_search": false, "websurfer_loop": false, "retrieve_relevant_plans": "never", "server_url": "localhost", "mcp_agent_configs": [], "run_without_docker": false, "browser_headless": true, "model_client_configs":  "advanced_agent_settings": false}
```

### Session 表 (会话表)：管理用户与AI团队的交互会话， 一个Session对应一个Team配置

### Run 表 (运行实例表)：在Session内的单次任务执行，保存AI团队的执行结果和过程数据

* CREATED: 已创建，等待执行

* ACTIVE: 正在执行中

* COMPLETE: 执行完成

* ERROR: 执行出错

* STOPPED: 被停止

* AWAITING\_INPUT: 等待用户输入

* PAUSED: 暂停状态

### Message 表 (消息表)：存储对话过程中的所有消息，既属于Session也属于具体的Run

### Gallery 表 (组件库表)：管理可重用的AI组件: agents、models、tools、terminations、teams

### Settings 表 (用户设置表)：存储用户个性化设置包括主题、显示选项等

### Plan 表 (计划表)：存储AI生成的执行计划

```
User ├── Team (1:N) ──────┐ ├── Session (1:N) ───┼── Session.team_id ├── Settings (1:1)   │ ├── Gallery (1:N)    │ └── Plan (1:N)       │                      │Session ──────────────┘ ├── Run (1:N) └── Message (1:N) ───┐                      │Run ──────────────────┘ └── Message (1:N)
```

数据库的管理是封装了一个class DatabaseManager 来进行增删改查数据库， 而在web服务启动的时候，就会通过init\_managers 把数据库给初始化好。

四、浏览器

最大的特点就是浏览器， 首先他里面有三种浏览器； 可以根据不同情况进行选择使用

| 类型 | 界面 | 部署方式 | 适用场景 |
| --- | --- | --- | --- |
| HeadlessDockerPlaywrightBrowser | 无界面 | Docker 容器 | 生产环境、CI/CD、批量自动化 |
| VncDockerPlaywrightBrowser | VNC 远程桌面 | Docker 容器 + VNC 服务 | 调试、开发、需要可视化监控 |
| LocalPlaywrightBrowser | 可选界面 | 本地安装 | 开发测试、本地调试 |

#### 4.1 执行create\_container方法 创建docker容器

```
Docker容器内:├── Xvfb (:99)           # 虚拟显示服务器├── openbox              # 窗口管理器  ├── x11vnc               # VNC服务器 (端口5900)├── novnc                # Web VNC代理 (端口6080)└── playwright-server    # Playwright WebSocket服务器 (端口37367)    └── Chromium Browser # 实际的浏览器实例
```

#### 

#### 

4.2 启动Playwright客户端连接 来和上面的docker容器 通过ws进行连接， 例如

```
Connecting to browser at ws://127.0.0.1:9694/ad5d353419d43245bc75c81be6dd8c6a
```

```
self._browser = await connect_browser_with_retry(self._playwright, browser_address)
```

后面就可以进行通信了

```
Python WebSurfer代码    ↓ (方法调用)宿主机Playwright客户端 (async_playwright())    ↓ (WebSocket: ws://localhost:playwright_port/path)容器内Playwright服务器 (playwright-server.js)    ↓ (本地进程通信)容器内Chromium浏览器# 客户端通过WebSocket控制容器内浏览器page = await context.new_page()await page.goto("https://example.com")await page.click("button")
```

还有一个就是前端也是可以操作这个浏览器的

```
前端iframe (http://localhost:novnc_port/vnc.html)    ↓ (HTTP/WebSocket)容器内noVNC服务 (端口6080等)    ↓ (WebSocket代理)容器内x11vnc服务 (端口5900)    ↓ (VNC协议)容器内Xvfb显示服务器 (:99)    ↓ (X11显示)容器内Chromium浏览器
```

五、agent

内部使用的autogen框架，最让人想到的就是他的群聊机制多智能体， 开始干活， 发送其他几个agent，这里简单介绍下，因为autogen框架是一个很复杂很强大的框架。

#### Orchestrator (智能协调器)

```
    负责管理群聊的智能协调器，功能包括：    - 任务规划和进度跟踪    - 根据当前状态选择合适的代理    - 与LLM交互生成计划    - 管理消息历史和状态
```

参与者

```
  web_surfer,      # 网页浏览代理    user_proxy,      # 用户代理    coder_agent,     # 编程代理 (可选)    file_surfer,     # 文件操作代理 (可选)    *mcp_agents      # MCP协议代理
```

群聊启动流程：

1. 接收 GroupChatStart 事件 (@rpc handle\_start)

1. 检查终止条件

1. 向所有参与者广播初始消息

1. 选择第一个发言者

1. 发送 GroupChatRequestPublish 请求

代理响应处理：

1. 接收 GroupChatAgentResponse 事件 (@event handle\_agent\_response)

1. 将响应添加到消息线程

1. 检查终止条件和最大轮次

1. 选择下一个发言者并发送请求

六 实例运行效果
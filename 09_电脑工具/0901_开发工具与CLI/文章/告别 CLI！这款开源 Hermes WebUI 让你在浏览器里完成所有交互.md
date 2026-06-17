---
title: 告别 CLI！这款开源 Hermes WebUI 让你在浏览器里完成所有交互
author: 云枢智能
date: 
url: https://mp.weixin.qq.com/s?__biz=MzYzNDc5Mjc0OQ==&mid=2247484301&idx=1&sn=4cf6e5cb798fcd0d2358c98113e069f4&chksm=f13dc4fc83af8179f001aa700a864758b006940638610ac0050307ee55748cf191e1b552a5da&mpshare=1&scene=24&srcid=0421ZKep48v9y5NeI6dCpxRP&sharer_shareinfo=71f2901038fdca6ade9ec5a94dc35bc9&sharer_shareinfo_first=71f2901038fdca6ade9ec5a94dc35bc9#rd
---

> **摘要**：还在用终端敲命令？Hermes WebUI 让你告别 CLI，在浏览器里就能完全控制 AI 智能体。开源免费、支持手机访问、功能 1:1 还原 CLI，部署只需一条命令。

---

*Hermes WebUI 主界面：三栏布局，左侧会话、中央对话、右侧文件浏览器*

---

# 🎯 为什么需要 Hermes WebUI？

**痛点场景**：

* 🤔 想在手机上调用 AI 智能体，却发现只能用终端？
* 🤔 需要频繁切换窗口看代码和对话，效率低下？
* 🤔 想给团队成员开放访问权限，但不想配置复杂的 SSH？
* 🤔 喜欢图形化界面，但对命令行望而却步？

**Hermes WebUI 就是答案** — 一个专为 Hermes Agent 打造的开源 Web 界面，让你**在浏览器里完成所有 CLI 能做的操作**。

---

# 🚀 项目简介

**Hermes WebUI** 是一个轻量级、深色主题的 Web 应用，让你能在浏览器中使用 Hermes Agent。

* **开源协议**：MIT
* **技术栈**：Python + 原生 JavaScript（无框架、无构建步骤）
* **作者**：社区开发者 `@nesquena`
* **GitHub**：https://github.com/nesquena/hermes-webui[1]

**⚠️ 重要声明**：本项目为社区开源作品，非 Nous Research 官方出品。

---

# ✨ 三大核心优势

## 1️⃣ 1:1 还原 CLI 体验

终端能做的，浏览器里都能做：

* 发送消息、查看回复
* 编辑历史消息并重新生成
* 查看工具调用卡片
* 管理会话、文件、定时任务
* 使用斜杠命令（`/help`、`/model`、`/theme` 等）

## 2️⃣ 零配置启动

```
# 一键启动  
git clone https://github.com/nesquena/hermes-webui.git hermes-webui  
cd hermes-webui  
python3 bootstrap.py
```

Bootstrap 会自动：

1. 检测并安装 Hermes Agent
2. 创建 Python 环境
3. 启动服务器并打开浏览器
4. 引导完成首次配置

## 3️⃣ 复用现有环境

* 使用你已有的 Hermes Agent 设置
* 使用你已配置的模型（OpenAI、Claude、Gemini 等）
* 共享 `~/.hermes` 目录的配置、会话、技能、记忆
* **无需额外配置**

---

# 🎨 界面设计

## 三栏布局

| 区域 | 功能 |
| --- | --- |
| 左侧边栏 | 会话列表、导航菜单 |
| 中央区域 | 聊天对话、流式输出 |
| 右侧面板 | 工作区文件浏览 |

## 底部控制栏

模型、配置文件和工作区控制位于**底部输入框下方**，始终可见：

* 环形上下文指示器（实时显示 Token 使用情况）
* 模型切换下拉菜单
* 配置文件切换
* 发送/停止按钮

## 设置界面

*自定义设置：密码保护、主题切换、语言选择*

## 浅色主题

*浅色主题，完整的配置文件支持*

---

# 🔥 核心功能

## 📌 持久化记忆

**问题**：大多数 AI 工具每次会话都会重置，不记得你是谁、做过什么、项目遵循什么约定。你每次都要重新解释自己。

**Hermes 的解决方案**：

* ✅ 跨会话保留上下文
* ✅ 离线时执行定时任务，醒来后交付结果
* ✅ 使用越久，对你的环境越了解

## 🧠 自进化技能系统

* Hermes 会自动从经验中编写和保存技能
* 无需浏览市场，无需安装插件
* 技能可重复使用，越用越聪明

## 📱 多平台访问

* 支持 10+ 消息平台（Telegram、Discord、Slack、Signal 等）
* 手机浏览器完美适配，可添加到主屏幕
* 通过 Tailscale 实现零配置远程访问

## ⏰ 自托管调度

* 内置 cron 定时任务
* 离线时也能执行，结果推送到你的消息应用
* 无需依赖云端服务

## 🤖 多模型支持

* OpenAI、Anthropic、Google、DeepSeek、OpenRouter、MiniMax、Z.AI 等
* 动态模型下拉菜单，从配置的密钥自动填充
* Provider 无关，自由切换

---

# 📊 竞品对比

| 功能 | Hermes WebUI | OpenClaw | Claude Code | Codex CLI |
| --- | --- | --- | --- | --- |
| 持久化记忆（自动） | ✅ | ✅ | ⚠️ 部分 | ⚠️ 部分 |
| 自托管定时任务 | ✅ | ✅ | ❌ | ❌ |
| 多平台消息访问 | ✅ 10+ | ✅ 15+ | ⚠️ 预览版 | ❌ |
| Web 界面（自托管） | ✅ | ⚠️ 仅仪表板 | ❌ | ❌ |
| 自进化技能 | ✅ | ⚠️ 部分 | ❌ | ❌ |
| Python/ML 生态 | ✅ | ❌ Node.js | ❌ | ❌ |
| 多模型支持 | ✅ | ✅ | ❌ 仅 Claude | ✅ |
| 开源协议 | MIT | MIT | ❌ 闭源 | ✅ |

**最接近的竞品是 OpenClaw** — 两者都是全天候、自托管、开源智能体，具备记忆、定时任务和消息功能。

**关键差异**：

* Hermes 自动编写和保存技能作为核心行为（OpenClaw 依赖社区技能市场）
* Hermes 更新更稳定（OpenClaw 有 documented 回归和安全事件）
* Hermes 原生运行在 Python 生态中

---

# 🛠️ 快速开始

## 方式一：一键启动（推荐新手）

```
git clone https://github.com/nesquena/hermes-webui.git hermes-webui  
cd hermes-webui  
python3 bootstrap.py
```

## 方式二：Docker 部署（推荐生产环境）

```
docker pull ghcr.io/nesquena/hermes-webui:latest  
docker run -d \  
  -e WANTED_UID=`id -u` -e WANTED_GID=`id -g` \  
  -v ~/.hermes:/home/hermeswebui/.hermes \  
  -v ~/workspace:/workspace \  
  -p 8787:8787 ghcr.io/nesquena/hermes-webui:latest
```

## 方式三：Docker Compose（最灵活）

```
# 检查并调整 docker-compose.yml  
docker compose up -d
```

## 远程访问

### SSH 隧道（安全）

```
ssh -N -L 8787:127.0.0.1:8787 user@your.server.com
```

然后打开 http://localhost:8787[2]

### Tailscale（零配置）

1. 在服务器和手机安装 Tailscale
2. 启动 WebUI 并启用密码：

   ```
   HERMES_WEBUI_HOST=0.0.0.0 HERMES_WEBUI_PASSWORD=your-secret ./start.sh
   ```
3. 手机浏览器打开 `http://<Tailscale-IP>:8787`

---

# 🎨 主题系统

内置 7 种主题：

1. **Dark**（默认）
2. **Light**（浅色）
3. **Slate**（石板灰）
4. **Solarized Dark**（太阳化深色）
5. **Monokai**（经典编程主题）
6. **Nord**（北欧冷淡风）
7. **OLED**（纯黑背景，降低烧屏风险）

切换方式：

* 设置面板下拉菜单（即时预览）
* 输入 `/theme <主题名>` 命令

---

# 🧩 高级功能

## 会话管理

* 创建、重命名、复制、删除会话
* 按标题和内容搜索
* 置顶会话（金色标识）
* 归档会话（隐藏不删除）
* 会话项目 — 命名分组，颜色区分
* 会话标签 — `#tag` 标签，点击筛选
* 按日期分组（今天/昨天/更早）
* 导出为 Markdown 或 JSON
* **CLI 会话桥接** — CLI 会话出现在侧边栏，带金色"cli"标识

## 文件工作区

* 目录树展开/折叠
* 面包屑导航，可点击跳转
* 预览文本、代码、Markdown（渲染）、图片
* 编辑、创建、删除、重命名文件
* 创建文件夹
* Git 检测 — 分支名称和修改文件数徽章

## 语音输入

* 麦克风按钮（Web Speech API）
* 点击录音，再次点击或发送停止
* 实时转录显示在文本框
* 约 2 秒静音后自动停止

## 斜杠命令

输入 `/` 触发自动补全：

* `/help` — 帮助
* `/clear` — 清空对话
* `/compress [主题]` — 压缩上下文
* `/model <模型>` — 切换模型
* `/workspace <路径>` — 切换工作区
* `/new` — 新建会话
* `/usage` — 显示 Token 使用
* `/theme <主题>` — 切换主题

---

# 🔒 安全特性

* **可选密码认证** — 默认关闭，零摩擦本地使用
* **HMAC 签名 Cookie** — 24 小时有效期，HTTP-only
* **安全响应头** — X-Content-Type-Options、X-Frame-Options、Referrer-Policy
* **20MB POST 限制** — 防止大文件攻击
* **CDN 资源 SRI 校验** — 防止第三方资源篡改
* **API 密钥脱敏** — 所有响应中的密钥自动打码

---

# 📦 技术架构

```
hermes-webui/  
├── server.py           # HTTP 路由 + 认证 (~154 行)  
├── api/  
│   ├── auth.py         # 密码认证，签名 Cookie (~201 行)  
│   ├── config.py       # 配置发现、模型检测 (~1110 行)  
│   ├── routes.py       # 所有 GET/POST 路由 (~2250 行)  
│   ├── streaming.py    # SSE 引擎、取消支持 (~660 行)  
│   └── workspace.py    # 文件操作、Git 检测 (~288 行)  
├── static/  
│   ├── index.html      # HTML 模板 (~600 行)  
│   ├── style.css       # 全部 CSS，含响应式、主题 (~1050 行)  
│   └── ui.js           # DOM 工具、Markdown 渲染 (~1740 行)  
└── tests/              # 961 个测试，53 个测试文件
```

**测试覆盖**：961 个测试用例，覆盖所有核心功能

---

# 🌍 国际化

已支持语言：

* 🇺🇸 English（默认）
* 🇨🇳 简体中文
* 🇹🇼 繁体中文
* 🇩🇪 Deutsch
* 🇪🇸 Español

---

# 👥 社区贡献

核心贡献者：

* **@aronprins[3]** — v0.50.0 UI 全面重构（26 commits）
* **@iRonin[4]** — 安全加固冲刺（6 个连续 PR）
* **@andrewy-wizard[5]** — 中文本地化
* **@gabogabucho[6]** — 西班牙语 + 引导向导
* **@Jordan-SkyLF[7]** — 实时流式传输、会话恢复

---

# 📚 文档资源

* HERMES.md[8] — 为什么选择 Hermes，详细竞品对比
* ARCHITECTURE.md[9] — 系统设计、API 端点
* ROADMAP.md[10] — 功能路线图
* TESTING.md[11] — 测试计划
* THEMES.md[12] — 主题系统文档

---

# 🔗 相关链接

* **GitHub**: https://github.com/nesquena/hermes-webui[13]
* **Hermes Agent**: https://hermes-agent.nousresearch.com/[14]
* **Docker 镜像**: https://github.com/nesquena/hermes-webui/pkgs/container/hermes-webui[15]

---

**📬 商务合作**：lpwacer@outlook.com[16]

**📝 声明**：本文基于 Hermes WebUI GitHub 项目 README 整理，感谢 `@nesquena` 及所有社区贡献者的开源工作。

### 引用链接

[1]*https://github.com/nesquena/hermes-webui*

[2]*http://localhost:8787*

[3]@aronprins: *https://github.com/aronprins*

[4]@iRonin: *https://github.com/iRonin*

[5]@andrewy-wizard: *https://github.com/andrewy-wizard*

[6]@gabogabucho: *https://github.com/gabogabucho*

[7]@Jordan-SkyLF: *https://github.com/Jordan-SkyLF*

[8]HERMES.md: *https://github.com/nesquena/hermes-webui/blob/master/HERMES.md*

[9]ARCHITECTURE.md: *https://github.com/nesquena/hermes-webui/blob/master/ARCHITECTURE.md*

[10]ROADMAP.md: *https://github.com/nesquena/hermes-webui/blob/master/ROADMAP.md*

[11]TESTING.md: *https://github.com/nesquena/hermes-webui/blob/master/TESTING.md*

[12]THEMES.md: *https://github.com/nesquena/hermes-webui/blob/master/THEMES.md*

[13]*https://github.com/nesquena/hermes-webui*

[14]*https://hermes-agent.nousresearch.com/*

[15]*https://github.com/nesquena/hermes-webui/pkgs/container/hermes-webui*

[16]lpwacer@outlook.com: *mailto:lpwacer@outlook.com*